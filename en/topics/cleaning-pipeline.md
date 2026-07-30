# Side-by-Side Comparison: Cleaning Pipeline

> Topic: Data processing for video generation models (including simultaneous audio-video generation): data cleaning pipeline, data distribution, annotation methods

[← Back to Home](../index.md)

This page compares all entries side by side, field by field. ⚠️ indicates that some information for this field in the given entry is uncertain.

**Fields**: [Overall filtering funnel structure (number of filtering stages, order of stages)](#overall-structure-of-the-filtering-funnel-number-of-filtering-stages-order-of-stages) · [Funnel quantitative retention rate (input/output volume at each filtering stage and final retention rate, e.g. Apollo 27%)](#quantitative-funnel-retention-rate-inputoutput-volume-at-each-filtering-stage-and-the-final-retention-rate-eg-apollo-27) · [Shot segmentation method (PySceneDetect / in-house model / shot-aware splitting)](#shot-segmentation-method-pyscenedetect--in-house-model--shot-aware-splitting) · [Quality filtering (aesthetic score, clarity, OCR text filtering, black bar/watermark/logo detection)](#quality-filtering-aesthetic-scoring-clarity-ocr-text-filtering-black-borderwatermarklogo-detection) · [Motion filtering (optical flow / motion score threshold, removal of static and shaky footage)](#motion-filtering-optical-flow--motion-score-thresholds-removal-of-static-and-jittery-content) · [Deduplication method (exact deduplication and embedding-based semantic deduplication recorded separately)](#deduplication-methods-exact-deduplication-and-embedding-based-semantic-deduplication-recorded-separately) · [VLM/LLM as quality inspector (multimodal large-model quality scoring and mismatch removal; the 2026 trend from shallow scorers toward large-model semantic judgment)](#vlmllm-as-data-quality-judge-multimodal-large-model-quality-scoring-and-mismatch-removal-—-the-2026-shift-from-shallow-scorers-to-large-model-semantic-judgment) · [Safety and compliance filtering (NSFW, copyright, faces/privacy)](#safety-and-compliance-filtering-nsfw-copyright-faceprivacy)
## Overall Structure of the Filtering Funnel (number of filtering stages, order of stages)

`pipeline_overview` · Level of detail: detailed

### [Allegro](../models/Allegro.md)

The paper discloses a complete chain consisting of a 7-stage serial filtering funnel + fine-grained annotation + quality stratification — one of the most granular disclosures among contemporaneous open-source work:
【Stage 1】Duration/frame-rate/resolution pre-screening: discard raw videos with duration <2s, FPS <23, or resolution <360p.
【Stage 2】Scene splitting: use PySceneDetect to cut long videos into single-scene clips, keeping only clips of 2–16 seconds; additionally discard the first and last 10 frames of each clip to eliminate false positives/transition residue from scene detection.
【Stage 3】Low-level metric filtering: brightness (evaluated on grayscale images, removing overly dark or overexposed frames), sharpness (DOVER video quality score), semantic/temporal consistency (LPIPS inter-frame perceptual distance), motion magnitude (UniMatch optical flow).
【Stage 4】Aesthetic filtering: LAION Aesthetics Predictor scores images and video middle frames.
【Stage 5】Content-irrelevant artifact filtering: CRAFT text detection + watermark detector, removing samples where black bars, text, or watermarks exceed an area-ratio threshold.
【Stage 6】Coarse-grained annotation: Tag2Text generates tag-style captions for middle frames, providing preliminary semantic information (also serving as the basis for subsequent category statistics).
【Stage 7】CLIP similarity filtering: compute CLIP cosine similarity between visual embeddings and the Stage-6 captions, removing samples with weak image-text correlation.
【Annotation】Samples that pass all filters are then given fine-grained spatiotemporal captions by a fine-tuned version of Aria.
【Stratification】The same corpus is split by threshold strictness into 4 nested subsets, fed respectively into the four stages: T2I pretraining / T2V 360p pretraining / T2V 720p pretraining / T2V fine-tuning.
Key design point: put "cheap coarse filtering (duration/resolution/splitting)" first and "expensive model inference (aesthetics, DOVER, CLIP, Aria captioning)" last, and place caption generation before CLIP filtering — using coarse captions as the image-text consistency criterion, avoiding running expensive fine-grained VLM inference over the full dataset.

### [Apollo](../models/Apollo.md)

A four-stage serial funnel (Section 4 of the paper + Figure 3 "Overview of our Dataset Annotation Pipeline"):
【Stage 1 — Video Filtering and Scene Splitting】First model video quality along four dimensions: dynamic quality (proportion of subject motion, shot stability), static quality (sharpness, aesthetics, color saturation), content naturalness (no excessive effects/watermarks), and safety; discard videos with low resolution, low SNR/MOS, or a silence ratio exceeding 20%; then perform scene splitting to ensure each sample contains only a single scene.
【Stage 2 — Audio Filtering and Post Processing】Discard samples with low SNR, low MOS, abnormal clipping, distortion, or noise, requiring a silence ratio <20%, high fidelity, and a unified format; then perform audio-visual consistency detection — Synchformer for temporal alignment, ImageBind for semantic alignment — to ensure high synchronization on both the temporal and semantic dimensions.
【Stage 3 — Audio-Guided Data Splitting】Stratify the dataset by audio type: first split into voice/non-voice to obtain the sound split, then further split the voice subset into singing, single-speaker speech, and multi-speaker speech.
【Stage 4 — Dense Annotation and Integration】For each subset, invoke dedicated models to produce speech transcription, audio captions, and video captions (including meta information and detailed content); the speech and singing subsets additionally extract speaker attributes (gender, age), while the sound split is only given audio captions; finally, all annotations are fused into unified dense captions.
【Funnel-ordering characteristic】Consistent with MOVA, the most expensive multimodal annotation is placed at the very end of the funnel, applied only to samples that have passed quality and synchronization filtering; the difference is that Apollo inserts an extra "audio-type routing" layer before annotation, allowing the annotation strategy to be customized per subset — this is the unique structural point of its pipeline.
【Limits of disclosure granularity】All four stages are described only qualitatively — no threshold table, no per-stage sample counts, no pseudocode.

### [CineDance / CineDance-1M](../models/CineDance.md)

A strict three-stage curation pipeline, which forms the methodological backbone of the entire paper:
【Stage One: Diverse Collection & Comprehensive Cleaning (Data Preparation & Quality Assessment)】
  1) Coarse-grained spatial cropping — EasyOCR locates burned-in subtitles, FFmpeg detects and crops out black bars/letterboxing;
  2) MLLM-guided temporal truncation — removes non-feature content such as opening/closing credits, with truncation amount t = max(5min, 0.1L);
  3) Full-volume quality metric computation — on the video side, DOVER (aesthetic score + technical score) and AMT (motion smoothness); on the audio side, DNSMOS (signal fidelity) and CLAP embedding temporal variance; on audio-visual alignment, ImageBind (global cross-modal alignment) and SyncNet (lip sync);
  4) Key design point: all quality scores are "retained in full as metadata" rather than used for hard pruning, allowing downstream users to construct task-specific subsets as needed;
  5) After shot splitting, a second OCR + black-bar fine check is run at the clip level.
【Stage Two: Bottom-Up Narrative Parsing】
  1) TransNetV2 splits atomic shots (25.89 million of them);
  2) Using Qwen3.5-27B as the backbone, atomic shots are grouped into narrative sequences according to four film-theory rules;
  3) A "bottom-up shot indexing" scheme is adopted rather than having the LLM directly output timestamps, significantly reducing timestamp hallucination;
  4) Context-aware sliding-window inference — roughly a 3-minute window, with window boundaries aligned to shot boundaries;
  5) A 20-second soft threshold to resist fragmentation.
【Stage Three: Configurable Dual-Modal Annotation】
  1) An anchor-token mechanism establishes a global character table and a global scene table;
  2) Visual annotation (Qwen3.5-35B-A3B) produces five-dimensional shot attributes + transitions + local character tables + active scenes + shot descriptions + transition descriptions;
  3) Audio annotation (Qwen3-Omni-30B-A3B) is split into three subtasks — sentence-level ASR, shot-level audio prompts, and character timbre description — to suppress hallucination;
  4) Windowed ASR-to-Character binding.
【Final】A three-person independent manual artifact audit on 500 randomly sampled clips serves as the final check.

### [CogVideoX](../models/CogVideoX.md)

CogVideoX's data pipeline is a three-stage design — "negative-label classifiers + continuous-score thresholds + dense re-captioning" — with a clear structure but shallower hierarchy (much simpler than Movie Gen's ten-plus-level funnel):
【Stage 1: Negative-label filtering】Human annotators label 20,000 videos as positive/negative samples → train 6 binary filters based on Video-LLaMA → apply them in batch to screen the full raw video pool. The 6 negative labels are: Editing (manual re-editing/effects that break visual integrity), Lack of Motion Connectivity (discontinuous motion at transitions, manually spliced or static-image edits), Low Quality (poor shooting, unclear picture, severe camera shake), Lecture Type (lecture/livestream talking-head content with minimal effective motion), Text Dominated, and Noisy Screenshots (direct phone/computer screen recordings). The paper attributes the design motivation to two categories of intrinsic noise (manual editing during video creation distorting real dynamic information; image-quality issues from filming equipment and shake) plus one training-suitability issue (too little dynamic information or lack of dynamic connectivity).
【Stage 2: Continuous-score dynamic thresholds】For all training videos, compute an optical flow score (representing motion intensity) and an image aesthetic score (representing image quality), and dynamically adjust these two thresholds during training to guarantee the dynamics and aesthetic quality of generated videos. The hyperparameter table gives a lower bound of 4.5 for the aesthetic score. This "dynamically tightening thresholds as training progresses" approach is one of the more distinctive aspects of CogVideoX's data engineering.
【Stage 3: Dense re-captioning】Videos that pass filtering are run through the Dense Video Caption Data Generation pipeline to produce long captions (see caption_model / caption_structure for details).
【Stage 4: High-quality subset fine-tuning】The pretraining data still retains dirty data such as subtitles, watermarks, and low bitrate, so in the final training stage a higher-quality 20% subset is selected out for fine-tuning.
【Image branch】LAION-5B and COYO-700M are filtered only by aesthetic score, taking 2B images, mixed with video for training (each image treated as a single-frame video).
【Inference-side alignment】A fine-tuned LLM (GPT-4V / CogVLM for image-to-video) performs prompt upsampling, rewriting the user's short prompt into a long description matching the distribution of the training captions.

### [Cosmos-Predict2.5](../models/Cosmos-Predict25.md)

The paper provides one of the most rigorous and complete descriptions of a video curation funnel currently in the public literature, explicitly defined as a seven-stage serial pipeline (Fig. 1):
【Stage 1 shot-aware video splitting】A high-precision shot-boundary detection model cuts long videos into shots, ensuring original shot transitions are excluded;
【Stage 2 GPU-based transcoding】Each clip undergoes GPU-accelerated transcoding (leveraging NVIDIA hardware decoders/encoders);
【Stage 3 video cropping】Black borders and spatial padding are cropped out; after this step, clips shorter than 5 seconds are discarded, leaving 6+ billion clips of 5–60 seconds;
【Stage 4 filtering】Internally split into seven further serial sub-stages, ordered strictly "cheap first, expensive last":
  4.1 aesthetic scoring filter
  4.2 motion filter (quantifying and removing based on degree of motion)
  4.3 OCR filter (removes clips with excessive text overlay)
  4.4 perceptual quality filter (DOVER-like, removes technical distortion and perceptual artifacts)
  4.5 semantic artifacts filter (removes video-in-video, poor transitions, and similar semantic artifacts, VTSS-like)
  4.6 VLM filter (Qwen2.5-VL vision-language model as the high-precision final check; the paper explicitly states it is "placed at the end of the filtering pipeline because it is more computationally expensive")
  4.7 content type classifier (26-class taxonomy) + physical-realism pruning (removes games, synthetic patterns, animation, cartoons)
【Stage 5 captioning】Each clip is cut into 5-second windows, and Qwen2.5-VL-7B generates short/medium/long captions window by window;
【Stage 6 semantic deduplication】Embedding clustering + pairwise comparison within clusters + retaining the highest-resolution version + online incremental deduplication;
【Stage 7 sharding】Sharded along four axes — content type / resolution / aspect ratio / length — to support efficient sampling, curriculum training, and fine-grained domain balancing.
【Domain-specific branches】Each of the five major Physical AI domains (Robotics / Autonomous Driving / Smart Spaces / Human Dynamics / Physics) goes through an isomorphic branch that differs in two respects: the VLM filter is omitted and replaced with a domain-specific filter subset and tuned parameters; and a larger VLM with domain-customized prompts is used for captioning. The output is merged into the general-purpose pretraining set.
【Design philosophy】The ordering of the entire funnel embodies "cascaded review with increasing compute" — lightweight scorers are used first to cut away the bulk in batch, then the most expensive multimodal large model performs a high-precision final review on the survivors, and only afterward is deduplication and sharding performed. Compared with Cosmos-Predict1, this is "a far stricter multi-stage filtering pipeline."

### [Data-Juicer 2.0](../models/Data-Juicer.md)

Data-Juicer's filtering funnel is not a fixed pipeline but a "configurable operator directed graph + an optimal operator combination searched via model feedback." This is the essential difference between it and self-built pipelines from model teams such as Apollo and UniVerse-1.
【Structural level — three-layer system architecture (DJ 2.0)】
  1. Capability Layer: expands DJ 1.0's roughly 50 pure-text pretraining operators to 150+ (reaching 229 as of v1.5.4) multimodal operators, covering text/image/audio/video/multimodal, and supporting post-training tasks. About 90% of operators are Mappers or Filters, roughly 75% involve multimodal processing, and about 50 are used for data synthesis and augmentation.
  2. Interface Layer: multi-level APIs — a low-level Python API, RESTful endpoints, a visual editor (an Alibaba Cloud PAI Designer component), and AgentScope-agent-based natural-language interaction (issuing data-processing tasks via conversational instructions).
  3. Runtime Layer: a unified Data-Juicer-Dataset abstraction, operator decoupling and adaptive optimization, fault-tolerant execution control, hardware-agnostic adapters (automatic switching among single-machine / Ray / Alibaba Cloud MaxCompute / PAI-DLC multi-backend).
【Process level — the Sandbox Probe-Analyze-Refine workflow】This is DJ's methodology for "designing the funnel," itself constituting a meta-process:
  · Probe: for each candidate operator, split the dataset into low/medium/high equal-sized data pools according to its statistic, plus a random-sampling control pool; train a reference model (small-scale, low-cost) with the same budget on each pool.
  · Analyze: compare the performance of models trained on each pool using a unified evaluation benchmark (16 metrics from VBench on the video side), and rank operators and their value ranges accordingly, identifying operators that genuinely provide gains.
  · Refine: cross-combine the top-ranked n operators into 2^n−1 data pools, forming a pyramid structure (the higher the layer, the more operator conditions are simultaneously satisfied, yielding higher quality but fewer samples); then validate the optimal combination at large scale, comparing two scaling routes — "repeating high-quality data" versus "progressively including suboptimal data layer by layer + deduplication."
【The Sandbox itself is also a three-layer design】The top layer is YAML-configuration-driven workflow orchestration (four stages: probe/refine/execute/evaluate); the middle layer provides Hook functions offering general development behaviors; the bottom layer is a factory class exposing 100+ operators and training/evaluation capabilities.
【The funnel finally adopted in the text-to-video case】After the above search, the final recipe settled on an extremely minimal two-stage filter: video_nsfw_filter (NSFW score filtering, underlying model Falconsai/nsfw_image_detection) → video_frames_text_similarity_filter (CLIP computes similarity between sampled frames and text, minimum threshold 0.306337). This filtered the pool from about 1.217 million down to 147,176. Notably, this final recipe is far simpler than the ten-plus-level funnels of most model teams — the Sandbox search concludes that "a few genuinely relevant operators + strict thresholds" outperforms "piling on a large number of filtering conditions."
【Engineering optimization — automatic funnel-order re-arrangement】DJ 2.0 introduces workload-aware OP reordering: under the constraint of operator commutativity, it automatically places computationally cheap operators earlier, reducing processing time by up to 70.22%. This turns "filter ordering" from manual experience into automatic system optimization, a rarely seen capability in this survey.

### [Foley-Omni](../models/Foley-Omni.md)

The filtering funnel is a three-stage serial structure; its core design philosophy is "filter first, annotate second, then use acoustic signals to reverse-validate the annotations." The input is weakly labeled audiovisual data, and the output is approximately 2.0M structured triples (v_i, a_i, Ŝ_i).
【Stage 1: Multi-dimensional Quality Filtering】Placed at the very front, cutting low-quality samples before expensive dense annotation — a typical cost-oriented ordering. The paper states: "removes clips with silence, low visual resolution, poor audio quality, weak audiovisual semantic consistency, or unreliable synchronization" — five removal conditions corresponding to three dimensions and six metrics (Table 7):
  · Visual dimension: resolution ≥480p, bitrate ≥1 Mbps, motion score ∈ [0.1, 3.2]
  · Audio dimension: Meta AudioBox Aesthetics audio-quality score ≥0.6 (covering both silence and poor audio quality issues at once)
  · Alignment dimension: ImageBind (IB) score ≥0.3 (semantic consistency), Synchformer sync score ≥0.2 (temporal synchronization reliability)
  Notably, "semantic consistency" and "temporal synchronization" are split into two independent metrics with two independent thresholds rather than merged into a single alignment score — a fine-grained design choice in this pipeline.
【Stage 2: Structured Annotation】Surviving clips are passed to Gemini 2.5 Pro, which takes both video frames and the audio track as input, and outputs according to a three-field [WORDS]/[AUDIO]/[MUSIC] template. The model is required to first perform component-existence detection, then description generation (detect-then-describe); the prompt template is given in Table 12 of the paper.
【Stage 3: Acoustic Post-Verification】A Bandit source-separation model splits the original audio track into three stems — speech/effects/music — and applies energy gating E(a_c) > −35 dB RMS field by field. Fields that fail are blanked out. This stage specifically corrects visual bias in the Stage-2 annotations — i.e., cases where the VLM fabricated audio components that are not actually audible, based on visual cues it saw.
【Overall assessment】The methodological highlight of this pipeline is "dual-path verification": the visual/multimodal path is responsible for generating annotations, while the pure-acoustic path is responsible for vetoing annotations; since the two paths draw on different information sources, their error modes are uncorrelated, making the cross-check effective. This idea is instructive for any work relying on a VLM for joint audio-visual annotation. The relative shortcomings are the lack of disclosure of standard steps such as deduplication, safety filtering, and manual review, and the absence of quantitative retention-rate reporting at each stage.

### [Goku](../models/Goku.md)

★A five-stage pipeline, the core methodological contribution of this entry, in the following order:
【Stage 1 Image and Video Collection】
  Raw material is aggregated from public datasets (LAION, Panda-70M, InternVid, OpenVid-1M, Pexels) and internal libraries.
【Stage 2 Video Extraction and Clipping】
  PySceneDetect detects shot boundaries → 1 frame per second is sampled and DINOv2 features extracted → adjacent-frame cosine similarity checks shot consistency (480p ≥0.85 / 720p ≥0.90) → each clip is truncated to a maximum of 10 seconds → clips from the same source are deduplicated using perceptual hashing.
【Stage 3 Image and Video Filtering】
  Multi-dimensional parallel filtering: basic attributes (duration ≥4s, short side ≥480, bitrate ≥500kbps, frame rate ≥24/23.976 FPS) + aesthetic score + OCR text-coverage ratio + RAFT optical-flow motion score, with each dimension's threshold set separately for the three resolution tiers 480p/720p/1080p — the higher-resolution tier being stricter.
【Stage 4 Captioning】
  InternVL2.0 generates key-frame captions + Tarsier2 generates full-clip video captions (including camera motion) → Qwen2 fuses the two caption streams → RAFT motion score is appended as a controllable condition.
【Stage 5 Data Distribution Balancing】
  An internal video-classification model tags 4 key frames → 9 major categories / 86 subcategories → over-represented categories are down-sampled, under-represented categories are augmented and over-sampled → human-related content is up-weighted overall.
【Structural characteristics】The first four stages form a bottom-up "per-sample purification" (funnel-style serial filtering), while the fifth stage is a top-down "dataset-level distribution shaping" — the two operate at different granularities. Making distribution balancing its own independent stage is Goku's key design differentiator from contemporaneous work.

### [Hailuo / MiniMax Video](../models/Hailuo.md) ⚠️

[Uncertain] (essentially no disclosure). MiniMax has never publicly disclosed the structure of Hailuo's video data cleaning pipeline — no funnel stage count, no per-stage filtering order, no flowchart. The closest thing to a data-processing statement in all public materials is a single sentence in the Hailuo 02 blog post: "training data expanded 4x, with improved quality and diversity," which implies the existence of quality filtering and diversity balancing steps, but says nothing whatsoever about how they are actually done.
This stands in sharp contrast to Vidu S1 (a six-stage funnel diagram), Seedance, Movie Gen, Cosmos, and other subjects in this survey that give complete pipeline descriptions. In terms of data-processing methodology, Hailuo sits at the "zero disclosure" tier, serving as a reference point for the lower bound of disclosure among commercial closed-source models, but it offers no reusable process detail whatsoever.
It should be noted that MiniMax's language-model side (the MiniMax-01/M1/M2/M3 technical reports) does describe data cleaning in more detail, but these are text-data pipelines that cannot be directly transferred to video data processing; this survey does not extrapolate across modalities.

### [HunyuanVideo-Foley](../models/HunyuanVideo-Foley.md) ⚠️

A seven-stage sequential funnel, overall following a "presence check → splitting → audio quality → cross-modal alignment → classification and annotation" narrowing structure. Its most distinctive feature: all quality filtering is concentrated on the audio side, with zero quality gating on the visual side; and the most expensive cross-modal alignment detection is placed after quality filtering but before annotation — a classic "cheap first, expensive later" cost-optimized ordering.
【Stage 1 Audio-track existence check】Videos lacking an audio stream are removed ("eliminate videos lacking audio streams"). The most front-loaded veto, with the lowest cost and potentially the largest elimination volume (a large fraction of web video is silent or has only background music).
【Stage 2 Scene detection + fixed-length splitting】A scene-detection algorithm performs shot splitting on the raw video, then regularizes clips into fixed-length 8-second blocks. This step performs no elimination — only a form transformation (converting variable-length long videos into fixed-length short clips) — and is a prerequisite for all subsequent per-clip judgments.
【Stage 3 Silence-ratio filtering】Compute the silence ratio for each 8-second clip; discard those exceeding 80%. A loose threshold that removes only near-silent clips.
【Stage 4 Bandwidth detection】Detect the audio's effective sampling rate (the upper bound of actual spectral energy rather than the sampling rate declared by the file), keeping only samples whose effective sampling rate exceeds 32 kHz. This is a fairly strict and technically sophisticated gate — it identifies "pseudo-high-definition" audio nominally labeled 48 kHz but actually upsampled from a low-bitrate source, sharing the same underlying idea as UniVerse-1's use of "bitrate-to-resolution ratio" to identify pseudo-high-definition video. An effective bandwidth of 32 kHz corresponds to an audible-frequency ceiling of 16 kHz, guaranteeing sufficient spectral information for high-frequency-rich Foley events such as cymbals, glass shattering, or metal friction. This threshold directly matches the model's 48 kHz output specification, an inevitable requirement of "data bandwidth determines output bandwidth."
【Stage 5 Audio aesthetics and SNR assessment】The primary metric uses the AudioBox-Aesthetics toolkit (from Meta, producing four dimensional scores: PQ production quality / PC production complexity / CE content enjoyment / CU content usefulness), supplemented by SNR as an auxiliary metric, to screen for high-quality audio clips. The paper gives no concrete threshold values — this is the biggest information gap at this stage, since which of AudioBox's four scores are used and what each is cut at is entirely unknown. [Uncertain]
【Stage 6 Audio-visual alignment verification】Two parallel paths: ImageBind performs semantic-alignment detection (whether the on-screen content and the sound content are semantically matched), and AV-align performs temporal-alignment detection (whether sound events and visual events correspond along the time axis). This is the most methodologically valuable step in the entire pipeline, and also where this work differs from most V2A work — treating semantics and timing as two orthogonal dimensions to be detected separately. No threshold is given here either. [Uncertain]
【Stage 7 Classification and distribution management】A speech-music detection model + an audio classification model tag every surviving clip with a category label, used to manage category distribution and achieve balanced representation. This is the only stage that does not perform elimination but rather "ratio adjustment," and is the final gate determining data composition.
【Step 8 (parallel, not a filter) Audio caption generation】A GenAU model generates a concise audio-content description for each clip, used as a text condition during training.
【Overall assessment】The pipeline design closely tracks the characteristics of the task, with clear trade-offs (strict on audio, loose on visuals); Stage 4's bandwidth detection and Stage 6's dual-dimension alignment are two highlights. But threshold disclosure is quite incomplete — of the seven stages, only three give numeric values (8 seconds, 80%, 32 kHz); the remaining four (aesthetics, SNR, ImageBind, AV-align) give no numbers at all, a noticeably lower transparency than the item-by-item threshold disclosure of UniVerse-1 or MOVA.

### [HunyuanVideo](../models/HunyuanVideo.md)

【HunyuanVideo original version (2024) — hierarchical data filtering pipeline】This is where the core reference value of this entry lies. The structure is "one filter chain + four progressively stricter threshold gates + one manual selection pass":
Step 0 Preprocessing: PySceneDetect cuts raw video into single-shot clips; an OpenCV Laplacian operator selects a sharp frame as the clip's starting frame;
Step 1 Deduplication and balancing: an internal VideoCLIP extracts embeddings, cosine-distance semantic deduplication is applied, and k-means with ~10K concept centers performs concept resampling and balancing;
Step 2 Filter chain (parallel multi-way scoring):
  · A Dover model scores from two perspectives, "aesthetic" and "technical quality";
  · An in-house blur-detection model removes unclear content;
  · Optical flow estimation gauges motion speed, removing static and slow-motion clips;
  · PySceneDetect + Transnet v2 provide scene-boundary information;
  · An OCR model removes clips with excessive text and crops out subtitle regions;
  · A YOLOX-like visual model detects and removes/crops watermarks, borders, logos, and sensitive information;
Step 3 Tiered dataset construction: the same set of filters, at increasingly strict thresholds, is used to build four training sets at 256p / 360p / 540p / 720p, each tier stricter than the last;
Step 4 SFT tier: on top of the strictest tier, manual annotation and selection is layered to obtain an SFT dataset of about 1 million samples.
Image data goes through a parallel tiered pipeline, reusing most of the filters except the motion-related ones, producing two image sets (on the order of billions and hundreds of millions respectively).

【HunyuanVideo 1.5 (2025) — three-stage filtering + spatial cropping】The structure was reorganized into a more engineering-oriented three stages:
Step 0 Basic deduplication and removal of corrupted files;
Step 1 Splitting: PySceneDetect + an in-house operator split into 2–10 second clips → a transition classifier removes clips containing transition effects;
Step 2 Spatial cropping: regions containing subtitles, logos, or watermarks are cropped out rather than the whole clip being discarded, but if the remaining area after cropping is <60% of the original frame, the whole clip is discarded (to preserve compositional integrity) — an important improvement over the original: shifting from "discard on sight of a watermark" to "crop where possible, discard only where cropping is not possible," improving data utilization;
Step 3 Three-tier quality filtering:
  (a) Basic filtering: removes clips with padding black bars, stitching artifacts, grid layouts/collages, or static/low-motion video;
  (b) Visual quality filtering: a composite quality-assessment operator scoring across four dimensions — sharpness, detail retention, noise & artifacts, and dynamic range;
  (c) Aesthetic filtering: an aesthetic-scoring operator removes low-scoring clips.
The output of about 800 million high-quality clips enters pretraining, and subsequent stages progressively shrink it under higher standards to 200M/100M/100M/1M.

### [InstructAV2AV](../models/InstructAV2AV.md)

The entire process can be divided into three major stages comprising roughly ten steps, with an overall architecture of "strictly filter real footage first → then generate controlled paired targets → finally doubly verify." Unlike the unidirectional funnels of "cleaning real data" seen in other work in this survey, the middle of this pipeline is a generation step, so its shape is an "hourglass" of "funnel + generator + secondary funnel."

【Stage One: Multi-stage filtering of real footage (multi-stage audio-visual filtering)】The goal is to filter out clean single-shot clips "suitable as an editing base."
  1. PySceneDetect — cuts long videos into single-shot clips, a prerequisite for all subsequent mask/tracking operations.
  2. CoTracker3 grid-point tracking — computes average motion magnitude; clips below a threshold are discarded (removing static footage).
  3. LAION Aesthetics Predictor — removes visually low-quality clips.
  4. PyDub silence detection — removes silent clips below -45 dBFS.
  5. Audiobox-Aesthetics — removes low-quality audio.
  6. Split-path processing: the speech path uses TalkNet to locate the active speaker + Scribe to extract speech timestamps, keeping only clips where "speech is temporally aligned with a visible on-screen speaker"; the non-speech path removes clips with ambiguous sound sources, and tags each clip with a unique semantic sound-event label.
  Note the ordering design: visual filtering first, audio filtering second, sound-source attribution judgment last — cost increasing from low to high, a reasonably cost-oriented ordering.

【Stage Two: Data Editing Engine — the core contribution of this work】The goal is to manufacture (source, instruction, target) triples from a single piece of real footage.
  7. Grounded-SAM-2 — obtains instance-level masks, locating editable visual entities.
  8. Qwen3-Omni — conditioned on the instance mask and the source audio-visual context, generates diverse textual editing instructions covering four categories of editing operations.
  9. Audio-side synthesis: SAM-Audio separates the target entity's sound from the original audio track → an ElevenLabs text-to-audio/speech model synthesizes a new target sound according to the instruction → this is seamlessly mixed with the preserved background sound to obtain the target audio track.
  10. Visual-side synthesis: a self-built mask-guided editing model (based on Wan2.2-5B, trained with a source-conditioned flow-matching objective) synthesizes the new target within the mask region; critically, frame-wise cross-attention is introduced to inject the already-synthesized target-audio features into the video-generation process, forcing the visual target to be strictly time-synchronized with the new audio track (e.g., new dialogue and new lip shapes aligned frame by frame).
  The ordering here is "synthesize audio first, then synthesize video conditioned on the audio," rather than parallel or reversed — because once speech duration and rhythm are fixed, lip shapes must follow it, whereas the reverse is difficult to constrain.

【Stage Three: Dual verification】The goal is to remove samples where synthesis failed.
  11. Automatic verification: a multimodal LLM (Qwen3-Omni) scores along five dimensions — (i) instruction fidelity, (ii) content preservation, (iii) perceptual quality, (iv) audio-visual synchronization, (v) safety — keeping only samples that pass all five simultaneously. Produces 79K training pairs.
  12. Manual verification (used only for the evaluation set): 20 volunteers are organized into 10 judge pairs; each candidate sample is evaluated by 5 independent judge pairs, each pair responsible for one evaluation dimension; a sample must pass all five dimensions to enter the evaluation set. Produces 1K manually curated evaluation pairs.

【Methodological assessment】
  Highlight 1: manufacturing "non-existent paired supervision data" through controlled generation, and, through the two mechanisms of SAM-Audio separation-mixing and mask-region confinement, guaranteeing by construction that "non-target regions remain strictly unchanged" — thus making source and target a clean differential pair, a precision that real data can never provide.
  Highlight 2: forcing audio-visual synchronization at synthesis time via frame-wise cross-attention, an instance of "guaranteeing quality at data-generation time" rather than "filtering after generation" — far more efficient than filtering after the fact using a sync score.
  Highlight 3: a clear division of labor between automatic and manual two-tier verification — automatic verification preserves scale, manual verification preserves the credibility of the evaluation set, and the manual verification adopts a "one judge pair per dimension" independent-review design, avoiding interference from a single person scoring multiple dimensions.
  Shortcomings: nowhere in the entire process is a quantitative retention rate disclosed; almost all threshold values are missing except -45 dBFS; there is no deduplication step; and there is no data-side ablation demonstrating the necessity of each pipeline component.

### [2026 Other Joint Audio-Video Generation Work](../models/JAVG_2026_misc.md) ⚠️

The data-pipeline complexity of the seven entries varies enormously, forming a spectrum from "no pipeline" to a "six-stage industrial pipeline":

【ALIVE — a six-stage industrial-grade pipeline (the most complete in this batch)】
Prefix step: first filter "videos with audio" from the raw data pool; samples without an audio track are eliminated outright.
Original text: "our pipeline begins by filtering videos with audio from the raw data pool and then proceeds through six core stages: video quality pre-processing, captioning, audio quality filtering, SubjectID correction, clarity filtering, and data balancing."
Stage 1 Video quality pre-processing: OCR measures the proportion of overlaid text and detects watermarks → a pretrained aesthetic model scores frame by frame → RAFT computes optical flow for motion analysis → a quality model classifies each sample as high-quality or into one of 13 low-quality dimensions.
Stage 2 Captioning: a two-round SFT-trained caption model + manually annotated sub-motion-unit timestamps → produces a three-part structured caption of Subjects / Visual / Narration, with speech marked inline as <W> and non-speech acoustic events as <I>.
Stage 3 Audio quality filtering: dual-criterion MLLM scoring — audio quality score (removing samples with significant background noise) + audio-visual relevance (separating strongly correlated samples, controlling the proportion of weakly correlated samples such as BGM).
Stage 4 SubjectID correction (speaker-to-subject attribution correction, a five-step sub-pipeline, the most distinctive design in this batch): ① TS-TalkNet performs active-speaker detection, generating a face-voice correspondence matrix; ② Qwen3-omni parses narration timestamps; ③ majority voting performs "sentence → TrackID" mapping; ④ CLIP / ArcFace embeddings perform visual-semantic matching; ⑤ the corrected labels are used to rewrite the text. Videos that fail to meet the high-confidence threshold are filtered out, yielding a "simplified but high-quality" initial dataset. This stage solves the misattribution problem of "who said this line" in multi-person scenes — a challenge that simply does not exist for pure-video models but that AV models must confront.
Stage 5 Clarity filtering: judged using six tiers of clarity reference images (from blurry to sharp).
Stage 6 Data balancing: a top-level speaking / non-speaking binary split → a three-level taxonomy of nine major Level-1 domains → category proportions are adjusted according to concept frequency and expected application scenarios.
There is also a parallel Character-driven data pipeline (cross-pair + in-pair augmentation, see synthetic_data_synthesis).

【NAVA — a large-scale multi-operator pipeline (both ends of the funnel disclosed)】
Raw pool ≈ 20M audio clips + 100M video clips → final ≈ 15M clips.
① Large-scale splitting via a Hadoop pipeline ("raw videos are first segmented at scale with a Hadoop-based pipeline")
② PaddleOCR performs OCR filtering and subtitle removal (note this is "removal," not merely "discarding")
③ VideoCLIP extracts video embeddings + large-scale k-means clustering removes redundant/near-duplicate clips
④ Multi-operator quality filtering: visual side (aesthetics / sharpness / brightness / motion score), audio side (AudioBox Aesthetic score), audio-visual alignment side (three alignment operators — SyncNet / SyncFormer / ImageBind)
⑤ VLM-based filtering and tagging, retaining clips with clear visual quality
⑥ YAMNet audio classification + an omni-modal tagger producing five-class audio tags
⑦ Captioning: Qwen3-VL generates video captions, Qwen3-Omni generates audio captions, then Gemini-3-Flash directly concatenates or rewrites and fuses them; the high-quality subset instead uses Gemini-3-Pro
⑧ Multi-operator collaborative filtering produces a 160K high-quality SFT subset

【OmniCustom — a lightweight pipeline of three rules】Starting from SpeakerVid-5M (5.2M clips / 8,000 hours): ① SyncNet dual thresholds (|offset|≤3 and confidence>1.5); ② discard if aesthetic score <0.3; ③ discard if duration <10 seconds → yielding OmniCustom-1M (about 1M clips / 2,500 hours). This is followed by format unification (480p/24fps, 16kHz audio) → GLM-ASR transcription → reference-training pair splitting (first 4 seconds as reference audio, remaining 5 seconds as training audio-video) → randomly sampling frames containing a face and cropping them as reference images.

【StreamChar — dataset combination, no self-built pipeline】Directly combines SpeakerVid-5M + TalkVid + OpenHumanVid, with the added constraint of "no video/line exceeding 20 seconds"; ASR timestamps are used as supervision labels for the progress-aware pointer. No explicit description of quality/synchronization filtering. [Uncertain]

【CCL / Baton — source list only, no pipeline disclosure】CCL consists of OpenHumanVid + in-house (interviews/short dramas/films) + WavCaps/VGGSound; Baton consists of OpenHuman-Vid + AudioCaps + WavCaps + internet video. Neither describes any filtering, annotation, or deduplication steps. [Uncertain]

【ITS-JAVG — moving the pipeline to inference time】No training-data pipeline, but its structure is isomorphic to a data-filtering pipeline: for each prompt, N candidates are generated (5 for JavisDiT, 10 for MMDisCo) → a suite of verifiers (VideoReward-TA / JavisScore / ImageBind-TA / AVHScore / VQAScore / ImageBind) scores them → ARW adaptive weighting aggregates the scores → EvoSearch performs evolutionary selection. This can be understood as "moving the joint multi-scorer filtering of a training-data pipeline to after generation, before output."

### [Joint Audio-Video Generation Baseline Collection](../models/JavisDiT_baselines.md) ⚠️

The filtering-funnel complexity of the five entries ranges from "almost none" to "three-stage filtering + large-model annotation":

【MM-Diffusion (simplest)】Crawl → split → no explicit filtering. Crawled 928 natural-scene videos from YouTube, mechanically split into 1,000 non-overlapping 10-second clips, organized under 9 scene categories. The paper describes no quality filtering, deduplication, or synchronization-detection steps. AIST++ is taken directly as an existing dataset (the authors additionally applied a crop, named AIST++_crop in the repository). This reflects the stage characteristics of joint-generation research in 2022 — "get the task working first, refine the data later rather than pursuing volume."

【AV-DiT (no self-built pipeline)】Directly reuses two public datasets, doing only formatting preprocessing: video sampled at 16 frames and cropped to 256×256, audio truncated or padded to 1.6 seconds @16kHz and converted to a mel spectrogram (40×16×8). No filtering funnel.

【JavisDiT / JavisDiT++ (the most complete and transparent funnel in this collection)】Split into two independent pipelines, training-side and evaluation-side:
《Training side — data preparation for each of three stages》(fully disclosed in the GitHub data.md)
- Stage 1 (audio pretraining, 780K samples): audio folder → generate metadata CSV → extract audio statistics → truncate to under 30 seconds → uniformly resample to 16kHz → add placeholder dummy video references → output train_audio.csv. It is explicitly stated that "no data filtering strategy is applied" in order to maximize T2A capability.
- Stage 2 (audio-visual SFT): TAVGBench's 1.1M triples → FunASR detects and removes most videos containing human speech → aesthetic scoring, optical-flow/motion scoring, and OCR text scoring are applied following Open-Sora's method → corrupted videos are removed and samples with fewer than 10 frames are filtered out → fps is uniformly normalized to 16Hz → aligned with TAVGBench's captions → multiple source CSVs are merged → output train_av_sft.csv (330K samples).
- Stage 3 (AV-DPO): from a 30K-prompt pool disjoint from the SFT data, a reference model generates N=3 audio-video candidates per prompt → metadata and audio are extracted from the generated samples → paired with 1 ground-truth sample to form a group of 4 candidates ("1 ground truth + 3 generated") → multiple reward models perform modality-aware scoring → normalized modality-aware ranking selects winning/losing pairs → output train_av_dpo.csv (about 25K preference pairs).
《Evaluation side — construction of JavisBench》About 30K candidate sound-producing videos are collected from the test sets of existing datasets (Landscape / AIST++ / FAVDBench) and YouTube videos uploaded between June and December 2024 → pre-filtering (removing noisy candidates based on quality) → the Qwen model family automatically generates captions and categorizes them under a 19-class taxonomy → post-filtering (ensuring diversity based on content) → strict manual legal and ethical review → final total of 10,140 clips; an additional random sample of 1,000 forms JavisBench-mini.

【Harmony】A relatively simple funnel structure, but introduces model-based quality control: on the speech side, data aggregated from Emilia + OpenHumanVid + SpeakerVid is filtered using an "audio-visual consistency scoring model," yielding 2 million high-quality clips (3–10 seconds); on the ambient-sound side, AudioCaps + Clotho + WavCaps + 2 million self-collected clips are aggregated → all data is automatically annotated with Google Gemini (ASR transcription + video-description captions + background-sound descriptions) → mixed into training at a 1:1 ratio.

【UniAVGen】Describes no data-cleaning process [Uncertain], noting only that stage 1 uses the Emilia English subset while stages 2 and 3 use internal real audio-visual data, plus two formatting steps: audio sampled at 24kHz and converted to a mel spectrogram, and video at 16fps subsequently VAE-encoded. This is the entry with the least data disclosure in this collection.

### [Kling 3.0 / Kling Video 3.0 Omni](../models/Kling_30_Omni.md) ⚠️

The Kling-Omni technical report discloses a "three-tier data processing system": Tier 1 basic filtering — resolution/duration threshold checks, frame-level and temporal fingerprint deduplication, audio-visual corruption detection, content safety and NSFW filtering; Tier 2 temporal quality assessment — blur/jitter/compression-noise scoring, removing abrupt scene cuts and incoherent transitions, and removing static clips with "action-semantic density too low"; Tier 3 multimodal alignment verification — semantic consistency between video caption and actual footage, fidelity consistency between reference image and target video, and identity-consistency checks for human-centric tasks. Upstream there is also an automated internet data-mining pipeline (an in-house embedding model for cross-modal semantic retrieval) and an expert-model-driven synthetic-data-construction pipeline. The complete funnel disclosed by the same team's Koala-36M is: raw video → temporal splitting (Color-Struct SVM) → structured captioning → manual threshold filtering → VTSS model filtering — which can serve as a public reference paradigm for the Kling family's data funnel. [Uncertain: where the audio-specific filtering tier is inserted within the three-tier system in Kling 3.0 Omni]

### [LTX-2](../models/LTX-2.md)

LTX-2 itself does not draw a new cleaning funnel; on the video side it fully reuses the LTX-Video pipeline, adding a layer of "audio-informativeness filtering" on top of its output.
【LTX-Video data-processing pipeline (paper Fig. 11, 9 stages total, in the following order)】
1. Input Shots (using shots as the input unit)
2. Crop Black Bars (removes black bars, normalizes aspect ratio)
3. Estimate Motion Level
4. Generate Thumbnails
5. Mid-Frame CLIP Image Embedding (compute a CLIP image embedding on the middle frame)
6. Cluster and De-Duplicate
7. Resize Shots (size normalization)
8. Estimate Aesthetics
9. Filter Shots → Final Shots (filtering by motion level/aesthetic score/other thresholds, producing the final shot set)
This is followed by an independent "Captioning and Metadata Enhancement" stage — an internal automatic captioner re-captions the entire training set.
【LTX-2's addition】On top of the Final Shots, an AV subset is extracted according to the criterion of "containing significant, information-rich audio content," and re-captioned with a newly developed dual-track audio-visual captioner.
【Structural characteristics】A typical five-stage design of "geometric/structural normalization first → representation extraction → deduplication → quality-score filtering → captioning last," dominated by shallow discriminators (CLIP embeddings + a Siamese aesthetic network + motion estimation), with no VLM/LLM semantic quality-check step in evidence.

### [LongCat-Video](../models/LongCat-Video.md)

Overall, a two-major-stage funnel structure of "preprocessing + annotation," with each stage further split into several sub-steps:
【Stage One: Preprocessing】
1) Collection and deduplication: raw video is collected from multiple sources and deduplicated using source video ID and MD5 hash;
2) Scene splitting: PySceneDetect combined with an internally trained TransNetV2 jointly perform scene detection, splitting long videos into content-consistent, training-friendly clips;
3) Black-bar cropping: black bars are cropped out with FFMPEG during transition splitting;
4) Compression and packing: processed clips are compressed and packed to support efficient data loading during training.
【Stage Two: Annotation】
5) Basic metadata annotation: duration, resolution, frame rate, bitrate, aesthetic score, blur score, text coverage, watermark detection;
6) Motion-information annotation: optical flow is extracted to assess video dynamics;
7) Caption annotation: a fine-tuned LLaVA-Video generates basic descriptions (+ Tarsier2 annotation reinforces temporal understanding);
8) Cinematic-language and style annotation: a camera-motion classifier + Qwen2.5VL annotate shot scale/shot type/realism/animation style/color tone;
9) Caption enhancement: Chinese-English mutual translation, generation of condensed summaries, and random combination of cinematic-language and style tags spliced into the enhanced caption;
10) Distribution analysis and allocation: clustering of caption text embeddings + LLM-based category induction, used for data supplementation/rebalancing, and for dynamically allocating data subsets to different training stages.
Notably, the report presents these annotations as a "annotate first, then threshold-filter" metadata-driven mode — all quality signals are annotated into the database in full first, and subsets are then extracted by combining thresholds as needed at different training stages, rather than a one-time hard deletion. This "annotate into the database, filter as needed" design is exactly what underpins the "dynamic allocation of data subsets across stages" mentioned above.
Avatar 1.5 upgrades this to a two-layer structure of "offline annotation + online clip-level verification": offline, it performs face detection and keypoints, audio-availability annotation, lip-sync verification, visual-quality estimation, camera and motion classification, and time-segmented captioning; when fetching data online during training, it applies a progressive filtering order of audio sync → camera applicability → text and visual quality → duration → visual defects → motion consistency → mask-area constraint.

### [MOVA](../models/MOVA.md)

A three-stage funnel (Figure 3), implemented on the Ray distributed framework, with mixed NVIDIA GPU and Huawei Ascend NPU compute:
【Stage 0 — Entry-level elimination】Discard samples with decoding failures or lacking a valid audio channel; container-format anomalies are remuxed, codec anomalies are transcoded.
【Stage 1 — Video Preprocessing and Standardization (Standardization → Detection → Segmentation)】
  1) Core-content normalization: FFmpeg cropdetect detects black bars and retains the core frame → centers it → scales to 720p → symmetric padding to 9:16 or 16:9; frame rate resampled to 24fps.
  2) Voice-activity detection: Silero VAD labels speech / non-speech intervals.
  3) Scene-transition analysis: PySceneDetect records the timestamps of all scene cuts across the video.
  4) Segmentation: VAD and scene-cut timing information are fused to generate fixed-length 8.05-second clips of four types (single/multi-scene × speech/non-speech), with speech-clip start points adaptively avoiding cutting off a sentence. Only speech clips are retained (69.47% of preprocessed clips).
【Stage 2 — Audio-Visual Quality Assessment (three-dimensional parallel filtering)】Audio quality (signal-level + aesthetic), video quality (technical + aesthetic), audio-visual alignment (temporal + semantic), with thresholds set empirically from manual spot-checks of retained videos under different cutoffs; an additional EAT classification model constructs dedicated speech/non-speech subsets.
【Stage 3 — Joint Audio-Visual Annotation】MiMo-VL-7B-RL produces visual descriptions, Qwen3-Omni-Instruct produces speech transcription, Qwen3-Omni-Captioner produces non-speech sound-effect/music descriptions, and finally GPT-OSS-120B performs a cross-modal consistency check and fuses everything into a unified natural-language caption.
【Secondary filtering within training stages】Before Phase 2, three additional orthogonal filters are layered on — OCR (no burned-in subtitles), LSE (lip-audio correspondence), and DOVER (technical score) — see quality_filtering and stage_data_mixture.
Overall characteristic: "standardize + split" first, then "score and filter," and finally "annotate" — placing the most expensive MLLM annotation at the very end of the funnel, applying it only to clips that have already passed quality and alignment screening, a reasonable cost-control ordering.

### [Merged Entry: ① Mochi 1](../models/Mochi_MAGI_Motif.md) ⚠️

① Mochi 1: No pipeline disclosure whatsoever. The official blog post discusses only the architecture (AsymmDiT, AsymmVAE, 44,520-token full 3D attention, extended 3D RoPE, SwiGLU, QK-Norm) and the evaluation protocol (following OpenAI DALL-E 3's prompt-adherence automatic evaluation paradigm, using Gemini-1.5-Pro-002 as the judge model); the data section is entirely absent. The only data-processing-related statement in the model card is that "NSFW content filtering has been applied." [Uncertain]
② MAGI-1 (a four-stage funnel that is method-level complete but parameter-level blank, Fig. 13):
【Stage 1】Shot splitting: PySceneDetect cuts long videos into single-shot short clips (this module is applied only to video data; image data skips it).
【Stage 2】Filter Actors parallel filtering: 11 categories of dedicated filters act on the split clips at once (see quality_filtering / motion_filtering for details) — Video Quality Assessment (DOVER), Aesthetics (LAION aesthetic model), Overexposed/Underexposed (average brightness in HSI color space), Motion Strength (RAFT optical flow + saliency detection split into foreground/background), Camera Movement Stability (adjacent-frame optical-flow consistency), Slides Movement (optical-flow divergence), Border Detection (edge detection + Hough transform), Text Detection (dedicated spatio-temporal pattern recognition for subtitles), Logo Detection (Florence-2 open-vocabulary detection), Corner Face Detection (face detection + corner-position confidence), Transition Detection (CLIP key-frame semantic similarity).
【Stage 3】Deduplication: pairwise similarity on dual CLIP and DINOv2 features; exceeding either threshold marks the pair as a duplicate for removal.
【Stage 4】Advanced MLLM filtering: the first three stages have already removed the vast majority of low-quality data and the remaining volume is greatly reduced; at this point a multimodal large model is introduced for one round of semantic-level re-screening, catching complex bad cases; the key engineering design is that "this step can be seamlessly folded into the subsequent captioning process, thereby reducing overall cost and improving efficiency" — i.e., scoring and annotation share a single MLLM forward pass.
【Stage 5】Annotation: data that passes all filters is given a highly descriptive caption and an auto-regressive caption by the same MLLM.
【Stage 6】Distribution adjustment: multi-stage adjustment (quality tightened stage by stage) + dynamic distribution adjustment (ratios adjusted online according to evaluation feedback).
Key design point: put cheap rule-based/small-model filters first, and put the expensive MLLM after the data volume has already shrunk significantly, letting the MLLM's "judge" and "captioner" roles reuse a single forward pass.
③ Motif-Video 2B (a branched funnel that is complete at both the tool level and the parameter level, Fig. 7 Sankey):
【Stage 0】Branch decomposition: the raw pool is explicitly split into four branches — Image Real / Image Synthetic / Video Real / Video Synthetic — sharing the same downstream controls.
【Stage 1】Sanitation (front-end decontamination): removes corrupted/undecodable files, abnormally small files (thumbnails or download corruption), SSCD near-duplicates, NSFW content, and watermarked content. NSFW and watermarks use a dual signal — first an OCR pre-screen inherited from the older crawling pipeline (using on-frame text detection to flag station logos, burned-in subtitles, etc. with high confidence), then surviving clips are re-checked by a VLM against structured labels (watermark / nsfw / padded / multi_scene / timelapse / quality), and any hit is dropped; the paper calls the second pass a "semantic-aware safety net layered on top of OCR."
【Stage 2】Black-bar detection: ffmpeg cropdetect estimates the maximum content rectangle based on brightness statistics.
【Stage 3】OCR detection: PaddleOCR-VL (served via vLLM) runs on N uniformly sampled frames per clip, clustering detection boxes across frames by spatial IoU, keeping only clusters appearing in ≥50% of frames (distinguishing fixed overlays from transient in-scene text); surviving OCR regions are merged with the black-bar crop box into a single final rectangle (excluding station logos in the top 20% of the content area and subtitles in the bottom 20%), applied in a single ffmpeg re-encode together with resolution scaling and frame-rate limiting.
【Stage 4】Scene splitting and length control: over-splitting with a conservative threshold + SigLIP-similarity-based stitch merging + discarding clips <2s.
【Stage 5】Multi-way visual quality filtering: Aesthetic Predictor V2.5 (aesthetics), brightness (OpenHumanVid formula), a Koala-36M-style modeled suitability score (conservative rejection), DOVER technical score (technical quality), UniMatch optical flow (motion quality, two-sided trimming). The paper emphasizes that these signals "are not synthesized into a single learned ranking, but each filter specializes in one failure mode."
【Stage 6】Three-stage SSCD deduplication (embedding / grouping / representative selection, see deduplication for details).
【Stage 7】VLM annotation as metadata: Qwen3-VL-30B-A3B produces both captions and structured labels in a single forward pass, with the labels fed directly back into sanitation and staged filtering.
【Stage 8】Staged progressive admission: 144p → 360p → 480p → 720p, with resolution/duration/motion/aesthetic filtering re-applied and made stricter at each transition; the 720p SFT tier additionally layers on domain-balancing and dynamic-motion admission; synthetic video is injected only at 720p.
Design point, isomorphic to MAGI-1 (VLM annotation and filtering sharing one forward pass), but Motif takes it further — "because these labels come from the same forward pass as the training caption, filtering and conditioning remain synchronized by construction," i.e., mechanically eliminating any drift between "the semantic judgment used for filtering" and "the text condition used for training."
</content>

### [Movie Gen](../models/Movie_Gen.md)

【Video side】A funnel of three-stage filtering + one-stage captioning (Figure 9 of the paper):
1) Visual filtering (6 filters): minimum aspect-ratio gate → aspect-ratio matching → video OCR removes multi-subtitle/text content → FFmpeg scene-boundary detection cuts 4–16 second clips → a lightweight visual model trained in-house predicts frame-level aesthetics/image quality/heavy letterboxing/visual effects and filters accordingly → following Panda-70M, the first few seconds of clips overlapping with the start of the source video are removed (openings often contain unstable camera work and transition effects).
2) Motion filtering: an internal static-video detection model removes videos with no motion → FFmpeg VMAF motion score and motion vectors determine "reasonable motion" → PySceneDetect's shots-per-second identifies and removes shakiness → removes slideshow-like effect motion.
3) Content filtering: copy-detection embeddings perform perceptual deduplication → joint video-text embedding clustering + merging of duplicate clusters + 1/sqrt(cluster size) resampling for concept balancing.
4) Captioning: LLaMa3-Video 8B/70B generates dense captions averaging 100 words + a 16-class camera-motion classifier's result is prepended + an FPS token.
There is also a "multi-stage curation": progressively stricter visual/motion/content thresholds carve out 3 subsets — a 720px low-resolution training set, a 768px high-resolution training set, and a newly augmented high-resolution set.
【Video SFT side】Four serial stages: ① automatic strict-threshold selection (strict aesthetics, motion, and scene-cut thresholds + Detic object detection removes videos where the subject is too small) yields several million candidates but with unbalanced concepts; ② concept balancing (a 600-verb taxonomy performs text k-NN, followed by video k-NN after human seed selection) shrinks it to a size manually reviewable; ③ humans select cinematic-feeling video (requiring angled or natural light or studio lighting, vivid but not oversaturated color, uncluttered composition, non-trivial motion, no camera shake, no editing effects or overlaid text), and annotators hand-crop the most compelling segments; ④ humans refine and complete the LLaMa3-Video captions.
【Audio side】AED tags according to the 527-class AudioSet taxonomy → discards silence-dominated samples → maps to three classes voice/music/sound → CAVTP cosine similarity buckets samples into diegetic/non-diegetic/mixed → visual-side quality filtering (OCR removes videos with text, removes static video, removes low-definition videos <480px) → duration limited to 4–120 seconds → copy-detection embeddings perform visual deduplication → multiple models synthesize structured captions. Fine-tuning data is additionally pre-screened automatically using a "cinematic audio-visual classifier + AED," followed by final human-annotated selection.

### [NVIDIA NeMo Curator](../models/NeMo_Curator.md)

This is the core value of this entry. Recorded at two levels.

【I. Cosmos WFM production pipeline: a seven-stage funnel (first-hand disclosure in arXiv:2501.03575)】
1. Shot-aware video splitting — TransNetV2 detects shot boundaries, cuts clips, and uses image-embedding similarity for stitching;
2. GPU-based transcoding — unified conversion to H.264 mp4, NVDEC/NVENC hardware acceleration;
3. Video cropping — unifies aspect ratio/removes black bars;
4. Filtering — motion filtering + visual quality filtering + text-overlay filtering + video-type taxonomy filtering, four discriminators in series;
5. Captioning — a VLM generates dense descriptions;
6. Semantic deduplication — embeddings + k-means clustering + pairwise distance within clusters;
7. Sharding — bucketed by resolution/aspect ratio/duration and packed into WebDataset.

【II. The two pipelines of the open-source NeMo Curator framework (arXiv:2503.12964 and official documentation)】
(A) Clipping Pipeline: decode raw video → split into contiguous short clips based on inter-frame color change → merge adjacent clips via image-embedding-similarity stitching → transcode to H.264 → extract frames (ClipFrameExtractionStage) → decode motion vectors and apply motion filtering → aesthetic filtering → video-embedding annotation (Cosmos-Embed1) → VLM captioning (optional LLM rewrite enhancement) → optional WebP preview image → semantic deduplication → write out.
(B) Sharding Pipeline: generate text embeddings for captions → produce WebDataset-format files (embeddings stored as Parquet, metadata retained), enabling multi-GPU parallel sequential-read access to petabyte-scale data during training.

【III. Architectural characteristics】
- All stages are uniformly abstracted as ProcessingStage (declaring their own CPU/GPU resource requirements and input/output data contracts); Pipeline chains stages together, and execution is handled by a pluggable executor;
- The video pipeline uses XennaExecutor (Cosmos-Xenna) by default, translating ProcessingStage into Xenna stage specs and executing on Ray in streaming or batch mode; there is also an experimental RayDataExecutor (promoted to stable in 26.04);
- Starting in 26.02, YAML declarative definition of the entire pipeline is supported, along with Pipeline.describe() for inspecting each stage's resource and data requirements during development;
- Starting in 26.07, pipeline resumability is supported: completed shards are recorded in LMDB, skipped on restart, with support for SLURM job arrays.

【IV. Structural assessment】A typical five-stage design of "geometric/encoding normalization first → discriminative filtering → semantic annotation → deduplication → finally bucketed and packed according to the training curriculum." Its biggest difference from self-built pipelines such as LTX-Video's is that it brings sharding/WebDataset generation into the scope of curation, forming a closed loop between data processing and training data loading; and it executes as streaming rather than batch — stages run concurrently, data flows continuously, avoiding full materialization of intermediate outputs.

### [OmniHuman Dataset + OHBench](../models/OmniHuman.md)

Overall, a fully automated pipeline of "four-stage filtering → structured perception → hierarchical annotation → verification," one of the core contributions of this work. Each stage's ordering follows a "light-to-heavy" cost principle (paper's original text: lightweight-to-heavy filtering strategy):
【Stage 1 Temporal-Spatial Cleaning】
- Uses TransNetV2 for shot splitting, cutting long videos into single-shot clips;
- Uses OCR and logo-detection algorithms to estimate "text-contaminated regions," and crops frames to remove subtitles and station logos;
- Normalizes to 30 FPS and 44.1 kHz.
【Stage 2 Audio Governance】
- Discards samples with missing audio track, abnormal duration, or low quality (silence ratio, volume threshold);
- Uses Demucs for four-source separation, with vocals as the target track and the rest mixed as the background track.
【Stage 3 Aesthetic Filtering】
- Frame-level CLIP aesthetic scoring quickly removes low-quality clips (fast, coarse);
- Video-level DOVER assesses composition and sharpness (slow, fine);
- OCR text-density analysis and watermark detection.
【Stage 4 Motion & Temporal Consistency】
- UniMatch optical flow analyzes motion quality, removing static, jittery, or abnormal-motion clips;
- Perceptual hashing + SigLIP cosine similarity filters out semantically inconsistent frames.
【Stage 5 Structured Perception (not filtering; builds the annotation foundation)】
- YOLOv11 detects human instances → NMS refinement → MOTRv2 models cross-frame association via query propagation, obtaining identity tracklets;
- DWPose-L extracts 134 full-skeleton keypoints across body, face, and feet, with dedicated hand detection and refinement;
- 3D-Speaker performs speaker diarization to obtain M speech intervals; SyncNet resolves audio-visual attribution, greedily assigning each audio segment to the visual ID track with the highest response; ArcFace extracts face embeddings to complete identity assignment.
【Stage 6 Hierarchical Annotation】A two-stage caption generation with Qwen3-Omni as the inference core + Gemini-3 for background-plausibility and interaction-semantics judgment (see caption_model / caption_structure for details).
【Stage 7 Consistency Verification】Multiple checks: the number of structured subject labels must match the count output by the tracking module; the number of speakers and the transcribed content must remain within an acceptable edit distance of the upstream ASR transcript; only if all checks pass is the video retained.
【Structural assessment】
- Strengths: reasonable stage ordering (cheap shot splitting and audio-track existence checks come first, expensive DOVER, optical flow, tracking, and MLLM annotation come later); a "perceive → annotate → reverse-verify" closed loop that is relatively rare — using the tracking module's output to verify the count consistency of MLLM annotations, and using ASR transcription to verify the speech content in captions, which is a low-cost form of automated hallucination suppression (see model_as_data_judge for details);
- Weaknesses: threshold values are almost entirely undisclosed throughout the process (CLIP aesthetic score, DOVER score, silence ratio, volume, optical flow, SyncNet score, edit-distance margin are all described qualitatively as "preset threshold," "acceptable margin," etc.); no input/output volume is given for any stage; no deduplication step; no safety filtering step.

### [Open-Sora Series](../models/Open-Sora.md)

The filtering funnels of the two projects are clearly structured and fully open-sourced/documented, making this the most reproducible sample under this topic.
【Open-Sora 2.0 (hierarchical data pyramid)】The design philosophy is "filtering thresholds loosen to strict progressively stage by stage, forming a pyramid-shaped data hierarchy that corresponds one-to-one with a progressive training curriculum from low resolution to high resolution" — a large amount of loosely filtered data at the base for low-definition pretraining, a small amount of strictly filtered data at the top for high-definition fine-tuning. Specific stages:
  Stage 0 Preprocessing: discards corrupted files and outlier samples (duration <2s, bpp <0.02, fps <16, aspect ratio outside [1/3, 3]);
  Stage 1 Shot cutting: FFmpeg libavfilter scene score detects shot boundaries and splits, output uniformly normalized to fps<30 / long side ≤1080px / H.264, segments >8s are cut into 8s pieces, segments <2s are discarded;
  Stage 2 Aesthetic score: a CLIP+MLP aesthetic scorer, averaging the first/middle/last three frames;
  Stage 3 Motion score: FFmpeg libavfilter's VMAF motion score, removing both extremes (too low = static, too high = chaotic/violent);
  Stage 4 Blur detection: OpenCV Laplacian-operator variance threshold, majority vote across five uniformly sampled frames per video;
  Stage 5 OCR: PaddleOCR detects text with confidence >0.7, videos with excessive text are discarded;
  Stage 6 Camera shake: PySceneDetect's Shot Boundary Detection, average inter-frame change above a threshold is judged as shake and removed;
  Stage 7 Captioning: LLaVA-Video for the low-definition stage, Qwen2.5-Max for the high-definition stage.
【Open-Sora 1.x (open-source-code-level pipeline, docs/data_processing.md)】Four stages: ① shot cutting (PySceneDetect scene_detect → cut); ② quality assessment and filtering (aesthetic → optical flow → OCR → datautil filtering); ③ captioning and alignment (PLLaVA/LLaVA captioning → CLIP matching score computes image-text consistency → caption cleaning); ④ camera-motion detection is performed on the remaining samples, with the result written back into the caption.
【Open-Sora Plan v1.3 (a seven-stage funnel, with retention rate fully disclosed at every stage)】ffmpeg 16s splitting → LPIPS jump-cut detection → LPIPS motion filtering → EasyOCR subtitle cropping → LAION aesthetic filtering → DOVER technical quality filtering → LPIPS motion re-check. A distinctive feature is that **motion filtering is done twice**: a coarse pass first, then a re-check after the subtitle region has been cropped out, since cropping changes the frame composition and a clip that originally "had motion" may in fact become static.

### [Ovi](../models/Ovi.md)

Ovi adopts a "two corpora, each with its own funnel" dual-track design: the audio-visual paired corpus goes through a four-stage serial funnel, while the pure-audio corpus goes through a simplified two-stage process.

【A. Audio-visual paired corpus — a four-step pipeline (explicitly numbered in Section 3.2 of the paper)】
Step 1 Splitting and filtering: scene detection cuts 121-frame @24fps clips → resolution must be >720×720 → RAFT optical flow removes static video and produces a motion score → the LAION Aesthetics Predictor removes low-quality data → an internal face-detection model controls the composition ratio of single-person/multi-person/no-person clips.
Step 2 Sync detection: SyncNet produces confidence and offset scalars → only samples with |offset| ≤ 3 and confidence > 1.5 are retained → an additional audio gate of average volume ≥ −60 dB is applied.
Step 3 Captioning: an MLLM takes 7 uniformly sampled frames + the full audio track as input → outputs a merged caption of "visual event narration + interleaved <S>dialogue<E> + a trailing <AUDCAP>audio description<ENDAUDCAP>."
Step 4 Packing: black bars removed → scaled to 518,400 pixels (equal area to 720×720) while preserving aspect ratio → frames extracted at 24fps and converted to byte arrays → audio converted to raw wave bytes.

【B. Pure-audio corpus — a simplified process】Split into two duration tiers (pretraining ≤12s, fine-tuning exactly 5.04s) → the same MLLM used on the audio-visual side produces an audio transcript (left blank if no speech) and an audio description → enters training. No sync/aesthetic/motion or other visual-side filtering.

【Design orientation】The overall funnel places "synchronization" at the highest priority (the paper explicitly states "even a small amount of unsynchronized data harms lip-sync ability, so we choose strict standards to minimize mismatch risk"), followed by basic quality gates on resolution and motion/aesthetics; deduplication, safety, watermark/OCR, and similar dimensions are not addressed.

### [Script-a-Video](../models/Script-a-Video.md) ⚠️

A clear distinction must be made: the "pipeline" focus of this work is on the construction logic of the annotation schema, not the data-cleaning funnel. The cleaning process is almost entirely undisclosed.
【Data-cleaning funnel】The paper devotes only a single sentence to this: "we curated an internal dataset comprising 500K high-quality video clips from film, television, and lifestyle domains." The criteria for "high-quality," the number of filtering stages, the ordering of stages, and the choice of tools are all undisclosed. This is the most notable disclosure gap in this work. [Uncertain]
【Annotation pipeline (fully disclosed)】The actual processing flow is two steps:
Step 1: Gemini-2.5-Pro generates MTSS-format annotations for each of the 500K clips ("Each clip was annotated using Gemini-2.5-Pro");
Step 2: this data is used to supervise fine-tune (SFT) the open-source Qwen3-Omni-Instruct, producing a dedicated annotation model, Qwen3-Omni-MTSS-FT.
This is a typical "strong teacher model distilled into an open-source student model" paradigm, with no intermediate quality check, review, or resampling steps disclosed.
【The internal processing logic of MTSS annotation (the true methodological core of this work)】Can be viewed as a five-step "information reorganization pipeline":
1) Entity extraction and screening → entities are classified by narrative importance into person/object/animal/scene categories entering the Reference stream, with marginal entities downgraded to the global scene description;
2) Visual segmentation → split by shot into the Shot stream, each segment carrying three layers of information: time_range, visual_description, and camera;
3) Audio-event extraction and routing → events are filtered under a "strict audio-visual coupling" principle into dialogue/sfx/music categories entering the Event stream, background noise without a visual correspondent is downgraded to global_audio, and concurrent sound sources are split into parallel entries;
4) Global fallback → the three fields scene_description / global_style / global_audio carry macro-level information that does not belong to any specific stream;
5) Relational Grounding → identity linking (the references_in_shot array points to a Reference ID, and an Event's speaker points to a Reference ID) + temporal linking (active_events associates a shot with concurrent events, and intra-description timestamps anchor micro-actions to the global timeline).
This "decouple first, then re-link" structure is the methodological thesis of the entire paper: decoupling solves redundancy and editability, re-linking solves consistency.
【Generation-side pipeline】See the multi_stage_curriculum field (four-stage progressive training).

### [Seedance 2.0 and Seedance 1.5 pro](../models/Seedance_20_Seedance_15_pro.md)

Seedance 1.5 pro gives only a high-level description: a holistic data framework composed of three parts — "a multi-stage curation pipeline + an advanced captioning system + scalable infrastructure" — with the pipeline prioritizing video-audio consistency, motion expressiveness, and curriculum-based data scheduling. The Seedance 2.0 report has no data section and discloses no funnel structure at all (the report contains only introduction, evaluation, and contributors sections). Seedance 1.0, usable as a longitudinal baseline, discloses a six-stage sequential funnel: (1) Diversity-Oriented Data Sourcing → (2) Shot-Aware Temporal Segmentation (up to 12 seconds) → (3) Visual Overlay Rectification (logo/watermark/subtitle/sticker detection + adaptive cropping) → (4) Quality and Safety Filtering → (5) Semantic Deduplication → (6) Distribution Rebalancing; the whole pipeline is deployed as an automated system for high-throughput processing of massive data. Note: the claim in the task background that "Seedance 2.0 has the most complete data-side disclosure" does not match the actual report content — the arXiv report for 2.0 is an evaluation-oriented model card, and its data disclosure is in fact lower than Seedance 1.0's.

### [SkyReels Series](../models/SkyReels.md)

【SkyReels-V2 (arXiv:2504.13074) data pipeline】Presents a chain of "collection → shot splitting → tiered quality filtering → post-processing artifact cleanup → dual-axis bucket normalization → structured captioning → concept-balanced sampling":
1. Data collection (open-source datasets Koala-36M/HumanVid + self-collected film/TV footage + art asset libraries, O(100M));
2. Shot-boundary detection and splitting (PyDetect + TransNet-V2) → single-shot clips;
3. Basic quality filtering (low resolution, low frame rate, black/white screens or static frames, camera shake);
4. Video-type filtering (removes surveillance footage, game recordings, animation, meaningless content);
5. Post-processing artifact filtering and cropping (subtitles, station logos, image-editing traces, split screens, black bars; subtitle/logo cropping is applied where cropping is possible to maximize usable frame area);
6. Multi-model scoring filtering (aesthetic filter, OCR filter, mosaic filter, effects/sticker filter, plus VQA, IQA, and VTTS models);
7. Normalization (dual-axis bucketing BT×BAR: duration axis × aspect-ratio axis, FPS standardization);
8. Structured captioning (a general MLLM + three specialized sub-expert models → distilled into SkyCaptioner-V1);
9. Concept-balanced sampling (balanced by subject category, halving data volume);
10. Full-pipeline human-in-the-loop spot-checking (0.01% for pretraining, 0.1% for post-training).
【SkyReels-V4 (arXiv:2602.21818) data pipeline】A structure of "collection (real + synthetic) → parallel video-branch / audio-branch processing → audio-visual sync filtering → three types of caption generation":
A. Collection: public datasets + licensed proprietary content + synthetic data (multilingual text/speech/edit pairs);
B. Video branch: preprocessing (VLM-enhanced intelligent shot splitting) → three-dimensional filtering (basic quality / content quality / motion quality) → balancing (concept diversity + motion diversity) → VideoCLIP-embedding deduplication;
C. Audio branch: category classification (Qwen3-Omni four-class) → quality filtering (SNR / MOS / clipping rate / audio bandwidth / VAD silence ratio) → duration control (cut into 15-second blocks, same-category blocks concatenated) → content recognition (Whisper transcription) → audio captioning (Qwen3-Omni);
D. Cross-modal: SyncNet lip-audio sync filtering (|offset|≤3 and confidence>1.5, and average volume ≥-60dB);
E. Captioning: short caption / long caption / structured caption with special tokens produced in three parallel tiers.
Overall characteristics: V2 centers on a "shallow discriminator matrix + structured captioning + concept balancing"; V4 elevates VLM/omni-modal LLMs to being the primary drivers of splitting, classification, and captioning, and adds an audio branch and cross-modal sync filtering as two new chains.

### [Sora 2](../models/Sora_2.md) ⚠️

The structure of the training-data cleaning funnel is entirely undisclosed. The System Card's entire statement about cleaning is just two sentences: "Our data processing pipeline includes rigorous filtering to maintain data quality and mitigate potential risks. We also employ a combination of safety classifiers to help prevent the use or generation of harmful or sensitive content, including explicit materials such as sexual content involving a minor." That is, it only acknowledges the existence of two broad categories — (1) quality filtering and (2) risk/safety classifiers — without specifying the number of stages, their order, or the criteria at each stage. By contrast, the System Card's description of the "inference-time safety stack" (input-prompt blocking → generation → output blocking, including a CSAM classifier and a custom-trained multimodal reasoning monitoring model) is far more detailed — this is a typical feature of this model's disclosure structure: detailed on the safety/deployment side, nearly blank on the training-data side. [Uncertain]

### [SpeakerVid-5M](../models/SpeakerVid-5M.md)

Overall, a funnel of "five-step structured processing → five-dimensional quality filtering → multimodal annotation → four-branch organization," with the entire pipeline's code open-sourced (Dorniwang/SpeakerVid-5M-Code, a six-part script suite):
【Stage One: Structured Processing (5 steps, executed in strict order)】
  1) Scene Splitting: PySceneDetect detects visually salient transitions based on color and brightness changes, splits, and trims clips to 3–14 seconds.
  2) Speaker Diarization: the 3D-Speaker tool performs voiceprint clustering, selecting the two primary speaker IDs by speaking duration/frequency (the two together accounting for over 80% of total speaking time); remaining speakers are discarded.
  3) Human Detection & Tracking: YOLO performs spatio-temporal tracking, temporally and spatially cropping around each person to obtain single-person-centered sub-clips.
  4) Lip Synchronization: SyncNet computes audio-visual overlap, using its confidence score to bind the speaker ID on the audio side to a human bounding box on the visual side — this step is the key decision of "which person in the frame the sound belongs to."
  5) ID Correction: ArcFace extracts facial features, computing cross-clip facial cosine similarity to verify speaker identity consistency; outlier samples are reassigned if they are more similar to another ID.
【Stage Two: Quality Filtering (five dimensions in parallel)】Brightness, video quality (DOVER), sharpness (clarity = bitrate density B/(W×H)), motion blur (Laplacian variance on the face/hand regions), audio quality (Whisper confidence/no-speech probability/compression ratio).
【Stage Three: Multimodal Annotation】Qwen2.5-VL structured captions, Qwen-3 topic classification, Whisper ASR, DWpose skeletons, SyncNet metrics, Laplacian blur score, Qwen2.5-VL multi-persona motion-intensity scoring.
【Stage Four: Branch Organization】Reorganized into four branches by interaction form — talking/listening/dialogue/multi-turn — with an HQ threshold used to carve out a 571K-clip SFT subset.
【Design characteristics】Unlike general-purpose video datasets, the core of this pipeline is not "filter out bad videos" but "parse interaction structure" — the three steps of diarization, SyncNet binding, and ArcFace correction all answer "who is speaking, who is listening, where are they in the frame, and is this the same person across clips" — quality filtering is instead a supporting role.

### [Step-Video-T2V](../models/Step-Video-T2V.md)

The core value of this entry. Step-Video-T2V's data pipeline consists of six serial stages, and the tool/model used at almost every stage is named explicitly, giving it a comparatively strong reproducibility description among contemporaneous work:
Stage 1 Video Segmentation: PySceneDetect's AdaptiveDetector detects scene changes → FFmpeg cuts into single-shot clips → the first and last 3 frames of each clip are discarded.
Stage 2 Video Quality Assessment: 7 quality labels are assigned to sampled frames (aesthetic score, NSFW score, watermark, subtitle, saturation, blur, black bars), see the quality_filtering field for details.
Stage 3 Video Motion Assessment: the Farneback optical-flow algorithm computes three motion quantities — Motion_Mean / Motion_Max / Motion_Min.
Stage 4 Video Captioning: an in-house VLM produces three types of text — Short Caption, Dense Caption, Original Title.
Stage 5 Video Concept Balancing: an internal VideoCLIP extracts embeddings → K-means clusters into 120K+ clusters → Cluster_Cnt and Center_Sim labels are assigned → resampling for balancing and outlier removal.
Stage 6 Video-Text Alignment: 8 frames are uniformly sampled from each clip, and the average cosine similarity between frame embeddings and text embeddings is computed to obtain a CLIP Score, used to remove image-text mismatched samples.

【Hierarchical Data Filtering — the organizational approach of this pipeline】The output of the above stages is a complete "label system" rather than a one-time discard: all clips are first fully labeled, then, by progressively raising the thresholds on each label, 6 subsets are carved out for the Step-2 T2VI pretraining stage (Figure 11 shows each filtering level as a bar chart: gray bars are data removed at that stage, colored bars are retained data). This "fully label first, then threshold-slice" design is more flexible than "filter and discard as you go," allowing the curriculum to be adjusted without rerunning the pipeline.
Two more stages are layered on top of this for post-training: automated scripts + heuristic rules + outlier removal by distance from cluster centers → manual review and curation, yielding a 30M SFT set.

### [UniTalking](../models/UniTalking.md) ⚠️

A three-stage serial funnel + one annotation stage + one derived-data synthesis stage, overall structured as "unimodal first, cross-modal second, annotation last" — the clearest presentation of the data section in this work (Figure 2 of the paper draws this pipeline specifically):
【Stage 1 Video Filtering (unimodal · visual)】Processes the visual stream, removing three categories of content: static video, video with text overlays, and video with generally low visual quality. Neither the model used nor any threshold is disclosed.
【Stage 2 Audio Filtering (unimodal · auditory)】Processes the audio stream, removing three categories of samples: muted, lacking speech, and low SNR. Tools are PANNs + SentenceASD, used to perform speech event detection. No threshold disclosed.
【Stage 3 Audio-Visual Cross-Modal Filtering】Assesses cross-modal relationships, removing two categories of samples: purely diegetic audio samples where the sound source is not on screen (tool: LightASD, active-speaker detection), and samples with poor lip-audio alignment (tool: LipSync). No threshold disclosed.
【Stage 4 Multi-level Joint Annotation】For audio-visual pairs that pass all filters, three granularities/forms of captions are generated (see caption_structure for details), using Qwen3-VL + Whisper-V3 + Qwen3-Omni-Captioner + Qwen3-Omni.
【Stage 5 Reference Data Synthesis】For each video, IndexTTS2 synthesizes 3 reference-timbre audio clips of 3–5 seconds, constructing training pairs for the TR2AV task.
【Structural characteristics and horizontal comparison】
- The "unimodal first, cross-modal as a backstop" ordering is a reasonable cost design: cheap unimodal detection eliminates most samples first, and expensive cross-modal detection is applied only to survivors. UniVerse-1's six-stage funnel is essentially the same ordering (hard visual metrics → audio activity → speech detection → SyncNet).
- Quite a few steps are missing: no shot splitting, no deduplication, no motion-score filtering (only a qualitative statement of "static removal"), no aesthetic scorer, no NSFW/safety filtering, no manual quality check, no watermark/logo/black-bar detection.
- The most prominent problem is zero threshold disclosure throughout the entire process. The paper lists 6 filtering rules and 4 tool names, but gives not a single numeric value — this makes the pipeline understandable at the "what was done" level but infeasible at the "can it be reproduced" level, a clear contradiction with its core claim of "open and reproducible."
【Verification finding on the claim "SyncNet confidence > 0.9"】This survey performed a full-text search of both the HTML and PDF (10 pages total) of arXiv:2603.01418v1; no SyncNet threshold value of any form exists in the paper, nor does the figure "0.9" appear anywhere in the paper (the only occurrence of 0.9 in the full text is β₁=0.9 for AdamW). The paper's lip-sync filtering is described only as "using LipSync to filter samples with poor lip-audio alignment," without specifying the exact model variant or giving a confidence threshold. The main text does mention an Appendix, but the v1 PDF actually contains no appendix at all. The claim "SyncNet conf > 0.9" therefore cannot be confirmed from the primary source. [Uncertain]

### [UniVerse-1](../models/UniVerse-1.md)

A six-stage sequential funnel (for self-collected data), with VGGSound/AudioSet taking a simplified bypass. Overall, a progressively narrowing structure of "hard metrics first, then shot splitting, then audio judgment, finally lip-sync verification":
【Stage 1 Audio-Visual Pre-screening】Videos lacking an audio track are discarded immediately — treating "must have a native synchronized audio track" as the most front-loaded veto condition.
【Stage 2 Quality Control (hard visual metrics)】Three parallel thresholds: resolution below 1080p is discarded; bitrate-to-resolution ratio below 600 is discarded; DOVER aesthetic-quality score below 0.6 is discarded.
【Stage 3 Temporal Coherence (shot splitting)】PySceneDetect performs scene splitting; clips shorter than 5 seconds after splitting are discarded.
【Stage 4 Audio Activity Detection】Analyzes three signal-level metrics — volume, energy, and zero-crossing rate — removing silent clips.
【Stage 5 Speech Content Verification (a routing point)】Whisper ASR detects the presence of speech — samples containing speech proceed to Stage 6 as the speech subset; samples without speech are not discarded but instead classified directly as "general audio-visual data." This is the only routing (rather than elimination) node in the funnel.
【Stage 6 Face and Lip-Sync Verification (speech subset only)】RetinaFace performs face detection; then SyncNet computes lip-audio sync confidence, requiring conf > 2.0 to be retained; samples that pass are explicitly tagged as "containing speech."
【Bypass: VGGSound/AudioSet】Only a minimum-duration constraint of 5 seconds is applied, skipping resolution, bitrate, DOVER, SyncNet, and all other quality and alignment filters — because these are introduced to supplement audio rather than visuals. The negative impact of their low visual quality is instead isolated on the training side through the LQLS loss strategy — a "if filtering can't solve it, solve it with the loss function" approach, a distinctive aspect of this work.
【Stage 7 (online, during training) annotation】Not part of the offline funnel, but an online annotation service run in parallel with training; see caption_model and model_as_data_judge.
【Overall characteristics】Compared with MOVA, UniVerse-1's funnel is shorter and shallower (6 stages vs. 3 major stages with a dozen-plus metrics), with no audio aesthetic scoring, no semantic-alignment filtering, no OCR subtitle filtering, and no deduplication; but moving the most expensive annotation step from offline to online is its biggest structural differentiator.

### [Unison](../models/Unison.md) ⚠️

This is the field with the weakest disclosure in this entry. The paper's entire description of the data-cleaning process consists of just two passages, both extremely brief:
【Passage 1: a one-sentence summary】At the end of Section 4.1, Training Corpora: "After refinement through our automated processing pipeline, the final dataset encompasses approximately 2 million synchronized audio-visual clips totaling over 3,000 hours, alongside 50 million high-quality audio segments exceeding 130,000 hours" — this only informs us that an "automated processing pipeline" exists and states its final output volume; the number of stages, their order, the methods and thresholds at each stage are all undisclosed.
【Passage 2: the only elaborated component — the lip-filtering operator】At the start of Section 3 (Method), described as the data-layer measure within a "three-layer safeguard for word-level lip sync":
Step a) Face detection — "first detects the number and location of faces," simultaneously determining both the number and location of faces. Note that detecting "number" implies the pipeline has the capability to recognize multi-face scenes (potentially usable to exclude multi-speaker ambiguity or select the primary speaker), but the paper does not explain how it handles the case of multiple detected faces;
Step b) In-box SyncNet verification — "SyncNet is then applied exclusively within these bounding boxes to verify alignment"; the key design is that SyncNet runs exclusively within the face bounding boxes rather than on the full frame. This is a practically meaningful engineering detail: running SyncNet on the full frame tends to fail in multi-person scenes or when the face occupies a small portion of the frame, whereas restricting it to the detection box significantly improves the reliability of sync judgment;
Step c) Dual removal targets — this operator "naturally filters out two types of samples": (i) clips where lip movement and speech are out of sync (unsynchronized speech and lip movements), and (ii) off-screen voice-over/narration clips. The removal of the second type is especially important — post-production voice-over is extremely common in vlogs, documentaries, and tutorial content, where the frame has no visible speaker yet the audio track has speech; such samples would badly contaminate lip-sync learning. UniVerse-1 achieves a similar effect indirectly through the Boolean condition "RetinaFace detects a face," whereas Unison lists it explicitly as a design goal.
【Passage 3: audio decoupling (not filtering, but preprocessing)】All audio is separated by Mel-RoFormer into speech and sound-effect streams, serving as ground truth for dual-stream supervision. This is the only explicitly named audio preprocessing step in the entire pipeline, applicable to both audio-visual data and pure-audio data.
【The minimal reconstructible pipeline skeleton】Aggregate five open-source datasets → automated refinement (details undisclosed) → lip-filtering (face detection + in-box SyncNet) → Mel-RoFormer speech/sound-effect decoupling → dual-stream latent encoding → training.
【Entirely missing components】No description of shot splitting, no aesthetic/sharpness filtering, no OCR/watermark/black-bar detection, no motion filtering, no deduplication, no safety filtering, no VLM quality check, no per-stage retention rates, no threshold values of any kind. Compared with contemporaneous work such as MOVA, UniVerse-1, and LTX-2, Unison's data-pipeline disclosure sits at the lowest tier — though it should be fairly noted that this follows from its positioning: since the video backbone is frozen, the marginal impact of visual-side data quality is minimal, and the team's attention is reasonably concentrated on the two dimensions actually being trained — audio and alignment. [Uncertain]

### [Veo 3 / Veo 3.1](../models/Veo_3_Veo_31.md) ⚠️

The officially disclosed cleaning funnel is extremely brief, reducible only to four confirmable components (order inferred): (1) multi-granularity caption annotation — multiple Gemini models generate text descriptions of varying detail for audio and video clips; (2) caption-side filtering — removes unsafe captions and descriptions containing personally identifiable information (PII); (3) video-side filtering — filters training videos by compliance, safety, and quality metrics, and applies safety filtering to pretraining data by risk domain; (4) semantic deduplication — cross-source semantic deduplication among all data sources, removing duplicate and highly similar-concept videos. There is also a non-filtering data-analysis step: harmful-content analysis and fairness review of population representation on the training data. [Uncertain] The exact number of filtering stages, execution order, judgment models, and thresholds for each stage are all undisclosed.

### [Vidu S1](../models/Vidu_S1.md)

A six-stage progressive filtering funnel (explicitly drawn in Figure 2 of the paper, from Raw Videos to the final training data):
Stage 1 Prefiltering: deduplication is performed first, then unreliable videos are removed by four technical metrics — frame rate, resolution, audio-video completeness, and audio-video synchronization → output: Pre-filtered;
Stage 2 Clipping: split along shot boundaries into single-shot clips, long shots are further subdivided with cut points avoiding the middle of speech, plus a Duration Filter retaining 3–60 seconds → output: Single-Shot Clips;
Stage 3 Subject Filtering: a Subject Filter ensures exactly one subject in frame with a reasonable proportion → output: Single-Person Clips;
Stage 4 Other Filtering: four parallel filters — Frame Content / Visual Quality Filter, Content Safety Filter, Shot Stability Filter, Interactivity Filter;
Stage 5 Diarization: VAD + ASD label speech segments and speakers, classified into onscreen/offscreen/overlap; an Overlap Filter removes overlapping-voice clips, a Speech Energy Filter removes clips where speech energy proportion is too low (singing/strong background music);
Stage 6 Caption + Embedding: generates dual-granularity structured captions at the clip level and the speech-aware chunk level; the selected clips and their captions are together embedded into training data.
The design goal is summarized as simultaneously improving visual sharpness, temporal stability, audio-visual consistency, and cross-modal interpretability.

### [Wan 2.5 / 2.6 / 2.7](../models/Wan.md) ⚠️

2.5/2.6/2.7 are undisclosed. Chapter 3 of the Wan 2.1 report gives a complete four-step funnel that is uncommonly thorough for the industry, making it the most reusable asset in this entry; Wan2.2-S2V additionally provides a hierarchical funnel dedicated to human-centric/audio-visual content.
【Wan 2.1 pretraining four-step data cleaning process】
Step 0 Candidate-pool construction: internal licensed data + publicly available data, first curated and deduplicated.
Step 1 "Fundamental dimensions" — efficient coarse screening targeting the source data's inherent properties, composed of 8+ lightweight detectors running in parallel: OCR text-coverage detection, LAION-5B aesthetic classifier pre-screening, an in-house NSFW safety model, watermark and logo detection, black-bar detection, overexposure detection, synthetic-image detection, blur detection, and duration/resolution gates. This step "successfully removes about 50% of the initial dataset."
Step 2 "Visual quality" — semantically driven fine screening, split into "clustering + scoring": first cluster into 100 clusters and sample by per-cluster quota to preserve the original distribution/prevent long-tail loss, then score the entire pool using an expert evaluation model trained on human 1–5 ratings.
Step 3 "Motion quality" — a six-tier motion-quality grading scheme, with retain/downsample/exclude actions applied per tier.
Parallel branch "visual text data" — two sub-tracks: rendering hundreds of millions of Chinese-character text images on a white background as synthetic data, and running real-world images containing text through multiple OCR models before feeding them to Qwen2-VL to generate natural descriptions containing the exact text content.
Step 4 Post-training curation (Section 3.2) — images go through "expert-model top-20% + manual curation," videos go through "visual-quality classifier top selections + motion-quality classifier splitting simple/complex motion + balancing across 12 major categories."
Finally, the captioning stage (Section 3.3): an in-house LLaVA-style dense-caption model re-captions the entire image and video corpus.
【Structural characteristics】A typical five-stage design of "cheap parallel coarse screening (cutting 50%) → clustered-quota + learned-scoring fine screening → motion-semantic-graded resampling → high-quality subset curation → full-corpus re-captioning"; discriminators are predominantly expert small models, with MLLMs appearing only in the captioning and evaluation stages.
【Wan2.2-S2V hierarchical human-centric funnel (paper Fig. 1)】
1) Data collection: automatic coarse screening of open-source datasets (captions containing person-related descriptions) + manual curation of videos with complex human activities such as speaking/singing/dancing → a million-scale human-centric video pool;
2) Pose tracking: VitPose extracts 2D pose and converts to DWPose, serving both as a control signal and a screening criterion;
3) Fine-grained screening: removes videos where the person occupies too small a proportion in the temporal or spatial dimension; retains only videos where the face remains continuously visible throughout the entire sequence (guaranteeing facial expressions can be learned from audio);
4) Five video-quality items: Dover sharpness, UniMatch optical-flow motion score, a Laplacian operator's sharpness check on face/hand regions, an improved aesthetic predictor, and OCR subtitle-occlusion detection;
5) Audio-visual alignment: Light-ASD active-speaker detection, removing videos where the audio does not sync with the active speaker, or where no active speaker is present on screen;
6) Dense captioning: QwenVL2.5-72B generates structured dense captions.

### [Audio-Visual Generation Benchmark Collection](../models/av_benchmarks.md)

The "cleaning process" of the five entries is actually an evaluation-data curation process, but their funnel structures have direct reference value for training-data pipelines:

【PhyAVBench five-stage curation process (the most complete)】① Audio physical-knowledge research — an LLM brainstorms candidate physical principles, human experts remove infeasible/redundant/irrelevant items; ② taxonomy construction — an LLM generates a hierarchical structure, experts review to disambiguate and remove redundancy; ③ physically-constrained prompt-pair design — an LLM generates candidate templates, experts manually verify and rewrite, ensuring each prompt pair differs in exactly one physical variable while all other non-target conditions remain unchanged, and avoiding subjective descriptions or explicit hints about the expected acoustic outcome; ④ real audio-visual collection — freshly recorded, under controlled conditions, sampled diversely across individuals/performers/devices; ⑤ iterative quality control and filtering — an LLM performs an initial screen for semantic ambiguity and unintended physical confounds, humans review physical consistency, three-way text-audio-video alignment, and real-world plausibility, and problematic samples are deleted, revised, or re-recorded. This is a typical closed loop of "LLM generation → expert revision → collection → LLM screening + manual review → iterative feedback."

【AV-SyncBench three-stage funnel】① Web collection of in-the-wild video; ② automatic filtering with Gemini 3 Flash — removing samples with off-screen sound sources and samples with clearly mismatched audio-visuals; ③ manual review — 5 annotators, each clip independently reviewed by at least 3 people, confirming the primary sound source is visible on screen and removing clips with poor audio quality, excessive noise, or semantic ambiguity. This is followed by a fourth step: programmatic perturbation generates temporal and semantic negative samples. This is the standard exemplar in this survey of "large-model pre-screening + multi-person cross-checked manual review."

【VABench dual-track curation】T2AV track: an LLM + expert templates batch-generate raw prompts → an LLM structurally decouples them into visual sub-prompts and auditory sub-prompts → generates VQA/AQA question-answer pairs → humans verify category correctness, element observability, and satisfaction of physical/commonsense constraints. I2AV track: humans curate and categorize high-quality images (including privacy screening) → a multimodal LLM generates a unified audio-visual description (visual objective + audio commonsense inference) → the description is used both to construct VQA/AQA and, decoupled by an LLM, into sub-prompts → humans review the plausibility of auditory inference and the discriminativeness of the questions. The paper explicitly states it "employed human workers and large language models to filter testing samples and adjust the distribution of test data" — i.e., humans and machines jointly share the two responsibilities of filtering and distribution adjustment.

【AVBench two pipelines】Evaluation-set track: 470 prompts of ≥720p are sampled from the prompt pool via Hard Quota-Based Greedy Sampling, with a quota constraint that any single attribute occupies ≤50%, then stratified into Normal/Hard subsets. Training-set track: 30K real clips are extracted from OpenHumanVid as positive examples → LLM-driven perturbations + algorithmic mismatches generate hard negative examples → each dimension is expanded to 100K pairs → 300K total across three dimensions.

【Omni-Judge】No data-cleaning process; this is meta-evaluation: 300 VidProM prompts → Sora 2 / Veo 3 each generate 1 clip → 6 PhD students score along 9 dimensions → the correlation between the Omni-LLM judge score and human scores is computed.

### [Video Caption Model Ecosystem](../models/caption_models.md)

The "cleaning process" for this entry has two distinct meanings that must be kept separate: (A) the cleaning of the captioner's own training data; (B) the position and filtering role the captioner occupies within a generative model's data pipeline.
【(A) the cleaning funnel for captioner training data】Generally simple, typically a four-part "teacher-model generation → LLM-score filtering → SFT → RL":
· AVoCaDO: Gemini-2.5-Pro separately generates visual captions and audio captions → the two captions plus the original video are fed back into Gemini-2.5-Pro to synthesize a temporally coherent multimodal caption → GPT-4.1 scores "synthesis completeness" on a 1–5 scale, keeping only scores ≥4 → SFT → GRPO.
· AVSCap: three stages — decoupled unimodal anchoring → cross-modal fusion → automated verification (tag-retention checks + semantic-consistency checks) → SFT → GRPO.
· CogVideoX's teacher chain: Panda-70M short captions → CogVLM generates dense image captions at a frame every 2 seconds → GPT-4 summarizes by timestamp → 50K samples fine-tune LLaMA2 to replace GPT-4 → further distilled into the end-to-end CogVLM2-Caption. This is a classic example of a "four-part teacher chain progressively cutting cost."
· SkyCaptioner-V1: Qwen2.5-VL-72B produces a general-purpose description + three sub-expert captioners (shot type/expression/camera motion) add film-specific dimensions → fused → distilled into a unified 7B model.
· Panda-70M: 31 candidate captioners generate in parallel → a user study performs greedy set-cover to select 8 → UMT-large fine-grained video-text retrieval picks the best. This is the only work that turns "captioner selection" itself into a filtering step.
【(B) the captioner's position within the generation pipeline】Three typical placements:
· A semantic gate mid-funnel: Allegro places Tag2Text at Stage 6, and its label output serves directly as the text-side input to the Stage-7 CLIP-similarity filter — the captioner is simultaneously an upstream dependency of the filter.
· Full captioning at the end of the funnel: the vast majority of work (HunyuanVideo, Step-Video, Movie Gen, Seedance) captions only after all filtering has passed, since captioning is the most expensive step.
· A tiered captioning pyramid: Open-Sora 2.0 uses the open-source LLaVA-Video to caption the massive low-resolution (256px) data, while the curated 5M high-resolution (768px) data is re-captioned with the more powerful Qwen 2.5 Max — "coarse captions at the base, fine captions at the top," strictly mirroring the data pyramid, a core piece of the low-cost strategy. Movie Gen's 70% 8B / 30% 70B mix is another implementation of the same idea.
· Online captioning (rare): UniVerse-1 places annotation inside the training loop, forcing it to choose lightweight models (Qwen2.5-Omni + Whisper), unable to bear the per-sample inference cost of a 120B-class model — the only public case of a trade-off between "annotation timing" and "annotation model capacity."

### [Geometric/Structured Annotation Dataset Collection](../models/geometric_datasets.md)

SceneScribe-1M (a three-stage funnel + annotation): ① specification filtering (resolution >1080p, frame rate ≥10fps, duration 5 seconds–1 minute) → ② Qwen2.5-VL-72B six-dimensional content-quality filtering (removing unknown motion intensity, visible watermarks, strong camera distortion, strong lighting artifacts, etc.) → ③ TransNetV2 temporal splitting (applied only to non-continuous video), with clips re-filtered after splitting → ④ geometric annotation (MegaSaM camera + depth, TAPIP3D 3D point trajectories) + semantic annotation (Qwen2.5-VL-72B captions); SpatialVID (manual pre-screening → splitting → four-dimensional quality filtering → geometric reconstruction → two-stage captioning): ① manual selection (removing corrupted content, titles containing inappropriate words or the phrase "panoramic camera," and content incompatible with MegaSaM reconstruction) → ② a modified PySceneDetect cuts 3–15 second clips and uniformly transcodes to 1280×720 H.265 → ③ four quality filters (aesthetics, brightness, OCR text ratio, motion/VMAF) → ④ MegaSaM camera-pose + depth reconstruction (with the depth module replaced by UniDepth v2 and Depth Anything v2) → ⑤ SAM2 dynamic masking and dynamic-ratio computation → ⑥ trajectory motion metrics (MoveDist/RotAngle/TrajTurns) and acceleration-anomaly detection → ⑦ a two-stage structured caption combining Gemini-2.0-Flash visual parsing + Qwen3-30B-A3B language refinement → ⑧ balanced sampling of the HQ subset; WildWorld (engine-native capture, no traditional cleaning funnel): a capture platform based on OBS Studio + Reshade partitions the display into sub-windows, synchronously timestamping and recording RGB (720p) and depth simultaneously, and exports skeletons, world state, and camera intrinsics/extrinsics from the game's memory/engine, producing 119 structured columns per frame, with a VLM then generating two tiers of captions; Action100M (an annotation-oriented pipeline that performs almost no video filtering): ① a V-JEPA 2 encoder converts video into a temporally dense visual embedding (1 frame sampled every 4, 64-frame overlapping windows, stride 8) → ② temporally constrained hierarchical agglomerative clustering (Ward linkage) generates a temporal tree → ③ Llama-3.2-Vision-11B frame-level captions + Perception-LM-3B clip-level captions → ④ GPT-OSS-120B performs evidence aggregation and structured extraction, three rounds of Self-Refine → ⑤ deduplication and semantic resampling

### [Video-Generation Post-Training Data Strategy](../models/post_training_data.md) ⚠️

【The anchor paper's four-stage post-training pipeline (the skeleton of this entry)】Phase 1 SFT (establishes a stable instruction-following baseline and reference policy using curated data) → Phase 2 RLHF (a GRPO-based trainer, aligning aesthetics, motion quality, and text alignment via multi-dimensional rewards) → Phase 3 PE prompt enhancement (an LLM is trained with the same reward loop via GRPO to rewrite user input) → Phase 4 AD autoregressive distillation (a self-forcing objective transfers capability to a causal architecture to improve inference efficiency).
The paper's core thesis is "SFT as the foundation for RLHF": the goal of SFT is not to solve alignment or optimize subjective quality, but to establish a stable and well-structured reference policy, eliminating the most severe and frequent failure modes, preventing subsequent RL from diverging into degenerate behavior, and, moreover, "SFT also enlarges the exploration for RLHF." The second thesis is "PE complements RLHF": RLHF optimizes the output-side generation policy while PE optimizes the input-side prompt, and the two are trained with the same reward (human preference, visual realism, semantic alignment), forming input-output dual-sided alignment.
【An important quality caveat】The specific wording in Section 3.1 on SFT is clearly migrated from LLM post-training text — the failure modes it lists, "refusal cascades, incoherent reasoning, unsafe outputs," are failure modes of language models rather than of video diffusion models (the video side should instead be things like hand errors, text errors, or fast-motion collapse, which are in fact mentioned in the introduction). This paragraph contains no video-specific data-construction detail, and readers should not treat it as a reproducible SFT data methodology.
【A cross-sectional spectrum of post-training pipeline shapes】
· Four-stage (SFT→RLHF→PE→distillation): the anchor paper, Seedance 1.5 pro (SFT → AV-customized RLHF), Kling 3.0 Omni (quality-tuning SFT → DPO);
· Three-stage CT→SFT→RLHF: HunyuanVideo 1.5 (with T2V and I2V kept entirely separate throughout);
· Two-stage SFT→DPO: Step-Video-T2V (Step-3 SFT → Step-4 Video-DPO), SkyReels-V2 (concept-balanced SFT → high-quality SFT → three-stage DPO);
· Two-stage SFT→GRPO: LongCat-Video, Cosmos-Predict 2.5 (five-domain SFT → model merging → GRPO);
· SFT only, no preference learning (the vast majority on the open-source/academic side): Movie Gen, CogVideoX, Allegro, Goku, Motif, MAGI-1, the Open-Sora series, NAVA, ALIVE, Apollo;
· No post-training at all: MOVA (folds SFT into the tail end of the pretraining curriculum), Unison, UniTalking, UniVerse-1, HunyuanVideo-Foley, Foley-Omni, CineDance, Mochi 1;
· Replacing post-training with inference-time search: ITS-JAVG (multi-verifier + Best-of-N/EvoSearch, JavisDiT 5 samples, MMDisCo 10 samples), arguing comparable results are achievable without any post-training.

### [Combined Survey of Mainstream Video Pretraining Datasets: Panda-70M, InternVid, Koala…](../models/pretraining_datasets.md)

**The structural differences among these seven funnels essentially come down to "what is used as the quality inspector,"** which can be grouped into three technological generations:
【First generation · pure heuristic scorers + thresholds】
- **InternVid (simplest)**: crawl constraints (duration 10s–30min, resolution 360P–720P, excluding existing public datasets) → an NSFW binary classifier → PySceneDetect (threshold 27) shot cutting → removes static and extreme-motion clips → multi-scale captioning → aesthetic-score-derived subsets applied post hoc via UMT-SIM. **Almost no thresholds are disclosed for filtering or captioning, making it the least reproducible.**
- **Panda-70M (splitting as the core, filtering very light)**: the weight of its funnel lies not in filtering but in **splitting** — PySceneDetect (threshold 25, min_scene_len=15 frames) → clips longer than 5 seconds are recursively cut to their first 5 seconds (changed to 7 seconds in the released code) → retained if the ImageBind first/last-frame feature distance ≤1.0 → stitched if the distance between adjacent clips ≤0.6 → clips <2 seconds or with too little motion (distance ≤0.15) are discarded → deduplicated if distance from the preceding clip >0.3 → the first and last 10% are trimmed → eight teacher models generate captions → a UMT retrieval model selects the best caption. **There is no aesthetic filtering, no sharpness filtering, no OCR filtering** — this is the root cause of the criticism it later drew from OpenVid/Koala/MiraData collectively.
- **OpenVid-1M (percentile-based filtering, four stages)**: source datasets aggregated → cascaded shot cutting → LAION aesthetic score (top **20%** taken from Panda-50M, top **90%** from the other three sources) → CLIP adjacent-frame cosine similarity two-sided removal (too high = near-static, too low = flickering) → UniMatch optical-flow two-sided removal → DOVER-Technical sharpness top **30%** retained → LLaVA-v1.6-34B re-captioning. Notable for **using percentiles rather than absolute thresholds throughout**, giving good portability but uncontrolled absolute quality.
【Second generation · MLLM as the semantic quality inspector】
- **LVD-2M (three stages, relying entirely on an MLLM as the semantic backstop)**: a >10-second duration gate → PySceneDetect **ContentDetector (threshold 50, min_scene_len=0, 0.5fps sampling)** removes shot-cut artifacts → RAFT optical flow (2fps, 520×960) mean **<20 discarded** → **PLLaVA-7B takes 8 frames and performs a GOOD/BAD binary classification**, with two separate prompts governing "degree of content change" and "visual diversity + screen text ratio" respectively. **No aesthetic scorer, no OCR model** — both are entirely outsourced to the MLLM's semantic judgment, making this the earliest and most thorough "VLM-as-judge" practice among the seven.
- **UltraVideo (four stages, statistical + model, two layers)**: source control (5,000 4K/8K clips, manually reviewed) → PySceneDetect AdaptiveDetector two passes + DINOv2 first/last 5-frame dissolve capture + routing by frame count into short/long → **statistical filtering** (PaddleOCR text area, black bars, overexposure, and grayscale — four items, uniformly applying a "discard the whole clip if the bad-frame rate >5%" pattern) → **model purification** (VTSS <0.01 removed, RAFT optical flow retained within 0.1–100, VideoCLIP-XL-v2 image-text similarity <0.2 removed, **Qwen2.5-VL-72B performs binary judgments on 16 low-quality attributes, any hit removed**) → Qwen2.5-VL-72B ten-dimensional captioning + Qwen3-4B summarization.
【Third generation · feeding multiple sub-metrics into a single learned scoring network】
- **Koala-36M (the only one opposing the multi-threshold paradigm)**: same-source data as Panda-70M → Color-Struct SVM transition detection re-splitting → LLaVA-family structured captioning → **a single VTSS score threshold of 2.5**. Its core argument is that **sub-metrics are not mutually orthogonal** (sharpness-aesthetics Pearson 0.3774, sharpness-motion −0.4028, motion-aesthetics −0.2515), and chaining multiple thresholds accumulates error (Appendix D Table 8: a 10% shift in the sharpness threshold alone mis-filters 250K/48M clips, and shifting all three thresholds simultaneously by 10% mis-filters 340K), so it instead uses a network that **takes "video pixels + sub-metrics" jointly as input and directly regresses a single scalar**.
【MiraData (the most distinctive structure: over-split → stitch → filter → caption)】
Collect → PySceneDetect (threshold **26**, deliberately low) over-splitting → **four-model voting to stitch** → four filters (frame color, LAION aesthetics, RAFT motion intensity, Stable Diffusion Safety Checker) computed at a **uniform 2fps**, producing four increasingly strict tiers of 330K/93K/42K/9K → Panda-70M short captions used as a hint + one round of GPT-4V dialogue produces 5 structured fields.
**Cross-sectional conclusion**: filtering strictness ranks roughly as UltraVideo > MiraData (9K tier) > Koala-36M ≈ LVD-2M > OpenVid-1M > InternVid > Panda-70M; while "quality-inspector intelligence level" ranks as UltraVideo (72B VLM) > LVD-2M (7B VLM) > Koala-36M (a learned scoring network) > OpenVid/MiraData (shallow scorers) > Panda-70M (almost none).
## Quantitative funnel retention rate (input/output volume at each filtering stage and the final retention rate, e.g. Apollo 27%)

`funnel_retention_rate` · Level of detail: detailed

### [Allegro](../models/Allegro.md) ⚠️

The paper provides a relatively rare staged quantitative retention rate (denominated against the original scale):
· Images: 412M raw images → 107M (about 26% retention), used for T2I pretraining.
· Video (360p tier, loosest threshold): 500M raw clips → 48M (retention 9.6%).
· Video (720p tier): 500M → 18M (retention 3.6%).
· Video (fine-tuning tier, strictest threshold): 500M → about 2M (retention 0.4%).
That is, from loosest to strictest, video retention spans 9.6% → 3.6% → 0.4%; the final high-quality curated set is only four-thousandths of the original volume.
Limitation: the paper only gives the "end-to-end retention rate for the entire funnel" and does not disclose the input/output volume of each individual filter stage (e.g. how many survived after PySceneDetect splitting, how many the aesthetic filter eliminated), so it is impossible to pinpoint which stage contributed the bulk of the elimination. [Partially uncertain: per-filter stage-by-stage retention rate]

### [Apollo](../models/Apollo.md)

【Core number】Overall post-filtering retention rate 27%, verbatim from the paper: "with an overall post-filtering retention rate of 27%." This is one of the rare publicly disclosed end-to-end quantitative funnel metrics in this survey, on the same order of magnitude as MOVA's 26.39% — the two corroborate each other in supporting the empirical rule that "the data funnel for joint audio-video generation generally retains a bit more than a quarter."
【Denominator convention】The paper does not specify whether the 27% is computed by sample count, total duration, or number of raw videos. From context (it immediately follows "81 million samples") it is inferred to be a sample/clip-count basis, but this is not stated explicitly. If computed on a count basis, the pre-filtering candidate pool would be about 300 million samples (81M ÷ 27% ≈ 300M) — this is an inference, not a number from the paper.
【Missing stage breakdown】Unlike MOVA's publicly disclosed stage-by-stage retention table (100% → 84.57% → 58.75% → 26.39%), Apollo only gives a single end-to-end total; the input/output volume of each of the four stages, the elimination share at each stage, and how much each sub-filter (video quality / audio quality / audio-video consistency) individually cuts are all undisclosed, so the main loss stage cannot be located.
【Conclusion】27% is a valuable but isolated anchor number, useful for cross-work benchmarking but insufficient to support stage-by-stage reproduction of the pipeline.

### [CineDance / CineDance-1M](../models/CineDance.md)

Paper Tab.3 gives a complete quantitative funnel, one of the most solidly disclosed parts of this work:
| Stage | Unit | Count | Duration |
| Raw collection | Videos | 45,181 | 32.8K hours |
| Spatiotemporal pre-filter | Videos | 44,579 | 32.5K hours |
| Shot detection | Shots | 25,899,474 | 32.5K hours |
| Narrative parsing | Sequences | 1,201,912 | 32.5K hours |
| Sequence pruning | Sequences | 1,079,382 | 28.6K hours |
| Post-verification | Sequences | 1,021,657 | 26.3K hours |
【Key retention rates】
  - Video-level pre-filter retention 98.7% (45,181 → 44,579), duration 32.8K → 32.5K hours, indicating the input material was already fairly high quality (since it comes from already-cleaned datasets such as MiraData/LVD-2M/Koala-36M);
  - Sequence pruning retention after narrative parsing 89.8% (1,201,912 → 1,079,382), duration 32.5K → 28.6K hours;
  - Post-verification retention 94.7% (1,079,382 → 1,021,657), duration 28.6K → 26.3K hours;
  - Total retention from parsed sequences to the final dataset is about 85.0% (1,201,912 → 1,021,657);
  - Total retention on the duration dimension is about 80.2% (32.8K → 26.3K hours).
【Compression ratio】25.89 million atomic shots are compressed into 1.2 million narrative sequences, i.e. an average of 21.5 shots merged per sequence.
【Artifact-audit comparison】In a manual audit of 500 random clips, CineDance-1M's non-compliance rate is 2.8%, versus 37.4% for Koala-36M — a 13.4× improvement.

### [CogVideoX](../models/CogVideoX.md) ⚠️

[Uncertain] The paper does not give any stage-by-stage quantitative funnel (input/output volume and retention rate for each filtering stage). The only two quantifiable points:
· After filtering, about 35M single-shot clips remain, but the original video pool size is not disclosed, so the overall retention rate cannot be inferred.
· The retention proportion for the high-quality fine-tuning stage is explicitly given as "a subset of higher quality video data, accounting for 20% of the total dataset" — the only explicit ratio number in the entire paper.
· Appendix K gives confusion matrices for six classifiers on the test set (a random 10% of labeled data), which indirectly reveal the share of each class of negative sample in the annotation pool: Editing class TP+FN≈0.89 (a very high negative-sample share), Low Quality class TP+FN≈0.89, Static (motion discontinuity) class TP+FN≈0.52, Lecture class 0.53, Text class 0.62, Screenshot class 0.62; but this is the distribution of the manually labeled sampling pool, not the true data distribution, and cannot be equated with a funnel retention rate.

### [Cosmos-Predict2.5](../models/Cosmos-Predict25.md) ⚠️

The paper gives an explicit end-to-end quantitative funnel and directly compares it with the previous generation — one of the most thoroughly disclosed quantitative cases in this survey:
【Cosmos-Predict2.5】35 million hours / 200M+ raw videos of original material → after splitting/cropping and removing clips <5 seconds, 6 billion+ candidate clips (5–60 seconds) → after seven-stage filtering + semantic deduplication, "Only about 4% of the initial clips pass all filters" → about 200 million trainable clips. That is, the clip-level total retention rate is about 4% (6B → 200M, roughly 1/30).
【Comparison with Cosmos-Predict1】20 million original hours, clip-level survival rate 30%. The paper explicitly states: "it achieves improved data quality control through a far stricter multi-stage filtering pipeline, which reduces survival from 30% of clips to only 4%" — i.e., while amplifying the original material volume by 1.75× (20M→35M hours), the survival rate was compressed to 1/7.5 of its former value, a deliberate, large-scale strategic shift toward "trading strictness for quality."
【Domain side】Smart Spaces is "keyword recall → VLM relevance verification → splitting → filtering → about 40K clips survive," but the input volume at the recall stage is not given, so the retention rate for this domain cannot be computed.
【Undisclosed】The input/output volume and stage-by-stage pass rate for each of the seven filtering sub-stages (only the end-to-end 4% total is given), and how much the deduplication stage alone removed. [Uncertain: the stage-by-stage retention rate of each filtering sub-stage]

### [Data-Juicer 2.0](../models/Data-Juicer.md) ⚠️

【The only officially disclosed quantitative retention rate — text-to-video case】
  · Input: InternVid 606k + Panda-70M 605k + MSR-VTT 6k ≈ 1,217,000 video-text pairs.
  · Output: 147,176 entries, about 227.5GB.
  · End-to-end retention rate: 12.09% (explicitly labeled on the HuggingFace dataset card).
  This number is on the same order of magnitude as Apollo's 27% and UniVerse-1's approximately 19%, but stricter, suggesting that the optimal strategy found by the Sandbox search leans toward "better too little than too much" — retaining only about 1/8 of the data.
  · There is also a 228k-entry data pool used for the model that ultimately topped VBench (retention rate about 18.7%); the paper records that this configuration corresponds to a training sample volume of 640k, i.e. about 2.8× repetition of the data was done on top of the 228k base.
【Stage-by-stage breakdown】[Uncertain] The elimination volume of the two operators individually is not disclosed: i.e. how much video_nsfw_filter alone eliminated, and how much video_frames_text_similarity_filter alone eliminated. Inferring from the nature of the operators, the NSFW filter should have a relatively low elimination rate (possibly single-digit percentage) on these three already lightly-cleaned public datasets, and the 12.09% retention rate is mainly contributed by the CLIP similarity threshold of 0.306337 — i.e., the vast majority of samples were removed because "the video imagery and the text description are not sufficiently aligned." This inference matches the threshold value itself: 0.306 is on the high side for a CLIP similarity threshold.
【Pool-partitioning convention during the probing stage】In the Probe stage, each single-operator pool is split into equal thirds by low/medium/high, i.e. each pool's retention rate is fixed at 33.3%; this is an equal-size constraint set for fair comparison, and does not represent the actual recommended retention rate.
【System-level throughput-oriented "retention rate"】The DJ 2.0 paper focuses on processing efficiency rather than retention rate, and does not give cross-project generic retention rate statistics.

### [Foley-Omni](../models/Foley-Omni.md) ⚠️

Quantitative disclosure is insufficient to construct a complete funnel table.
【Known endpoints】The pipeline output is about 2.0M video-audio-text triples; the full training corpus is about 2.7M pairs, about 4.9k hours.
【Missing】[Uncertain] The paper does not give the raw pool size before cleaning (in hours or count), does not give the input/output volume comparison for any single filtering stage, does not give the elimination share for each of the six metrics, and does not give the proportion of fields nulled out during the third-stage acoustic post-verification (this number could have directly quantified the severity of Gemini's visual hallucination, which would be the most persuasive evidence for this method — unfortunately it was not reported). Therefore an end-to-end retention rate similar to Apollo's 27% cannot be computed.
【A rough inference one can make】Among the six filtering metrics, Synchformer sync score ≥0.2 and ImageBind ≥0.3 are usually the main killers for in-the-wild audio-video data (a large amount of web video has added post-production soundtracks, narration voiceovers, or audio unrelated to the visuals). Referencing similar work (e.g. UniVerse-1 filtering from 40k+ hours down to 7,685 hours, about 19%; MMAudio-family work on VGGSound where the alignment filter elimination rate is often above 50%), it is speculated that this pipeline's original pool should be on the order of tens of thousands of hours, with an end-to-end retention rate roughly in the 20–50% range — but this is purely extrapolation, and the paper offers no data to support it.

### [Goku](../models/Goku.md) ⚠️

[Uncertain]. The paper gives the decision threshold for each filtering stage (Table 4) and the final data volume (160M images / 36M videos), but **does not disclose the input volume, output volume, or retention rate for any single stage**, nor does it give the original collection scale, so the overall funnel retention rate cannot be computed (compared with Apollo's practice of disclosing a 27% retention rate, Goku is less transparent on this point).
The only inferable quantitative shrinkage clue is the cascade of data volume by resolution tier: 480p tier 36M → 720p tier 24M (about 66.7% retention relative to 480p) → 1080p tier 7M (about 19.4% retention relative to 480p, about 29.2% relative to 720p). But this shrinkage is a composite result of "resolution + stricter thresholds" and is not equivalent to a single-stage filter retention rate.
In addition, I2V post-training uses 4.5M triples, about 12.5% of the 36M video pool, which can serve as an indirect reference for the proportion of the high-quality subset.

### [Hailuo / MiniMax Video](../models/Hailuo.md) ⚠️

[Uncertain] (no disclosure whatsoever). No input/output volume for any filtering stage, no final retention rate. Since even the funnel structure itself is not disclosed, quantitative retention rate is entirely out of reach.

### [HunyuanVideo-Foley](../models/HunyuanVideo-Foley.md) ⚠️

The paper gives no quantitative funnel data at all: it does not disclose the total volume of raw collected video (hours or count), does not give the input/output volume for any of the seven filtering stages, does not give stage-level or overall retention rates, and does not give the primary-reason distribution for elimination at each stage. The only known fact is that the final output is about 100,000 hours. The numerator is known but the denominator is entirely missing, so an overall retention rate comparable to Apollo's 27% or MOVA's 26.39% cannot be computed.
【Strength of qualitative inference】Among the seven stages chained in series, at least four have substantial elimination power: removal of tracks with no audio (broad elimination among web video), silence proportion >80% (fairly lenient, should eliminate a small amount), effective sample rate >32 kHz (fairly strict — a large amount of web video audio is nominally 44.1 kHz but has insufficient effective bandwidth after compression, so this may be the stage with the highest elimination rate), AudioBox aesthetics + SNR (threshold unknown, strength unclear), ImageBind + AV-align alignment (threshold unknown). Overall judgment: the total retention rate is probably in the single digits to low twenty percent range, meaning the original data volume could be on the order of hundreds of thousands to millions of hours. But this is pure speculation with no data from the paper to support it. [Uncertain]
【The significance of the disclosure gap】The paper lists "a scalable data pipeline" as the foremost of its three main contributions, yet provides no funnel data at all, making the claim of "scalability" impossible to verify quantitatively — readers know neither how much raw data the pipeline processed nor the compute-cost share of each stage, and therefore cannot judge this pipeline's efficiency advantage relative to comparable approaches. This is a substantive weak point in this work's argumentation.

### [HunyuanVideo](../models/HunyuanVideo.md)

【Original version】Only a relative convention, no absolute numbers: the paper states that at each stage "A large portion of data will be removed at each stage, ranging from half to one-fifth of the data from the previous stage" — i.e. each stage retains only 1/2 to 1/5 of the previous stage (converted to a retention rate of 20%–50%, elimination rate of 50%–80%). If four stages are chained in series and estimated using this range, the overall retention rate from 256p to SFT falls roughly in the wide range of 0.16%–6.25%; the paper does not give a final value.
【1.5】Absolute numbers are computable, a relatively rare quantitative funnel among comparable works:
· Original pool >10 million hours of video → after splitting+filtering, 800 million clips (at an average of 6 seconds this is roughly 1.33 million hours; a rough overall retention rate of about 13% — this conversion is a survey-derived estimate, not the paper's original figure);
· 800M (256p) → 200M (480p): 25% retention;
· 200M (480p) → 100M (720p/16fps): 50% retention;
· 100M → 100M (720p 16fps→24fps): 100% retention (same scale, frame-rate conversion only);
· 100M → 1M (CT/high-quality tier): 1% retention;
· Image side: >10 billion → 5 billion (retention <50%) → 1 billion (second-stage retention 20%).
The end-to-end retention rate from 800M to 1M is 0.125%, one of the most quantitatively complete funnels in this survey, directly comparable with Apollo's 27% (note the conventions differ: Apollo is a single-stage filter retention rate, whereas this is the terminal value after chaining multiple stages).

### [InstructAV2AV](../models/InstructAV2AV.md) ⚠️

[Uncertain] There is no quantitative disclosure whatsoever, the largest gap in this work's data methodology.
【The only known endpoint】Final output: 79K training pairs + 1K evaluation pairs (the HF release actually shows 87,074 + 1,000).
【All missing items】
  · The raw video pool size is unknown — it is not stated how many hours or how many videos were crawled from YouTube, nor how many were taken from each of MovieBench/Condensed Movies/Short-Films-20K/VGGSound.
  · The elimination rate for each of the six filters in stage one is unknown — how many clips PySceneDetect cut out, how many CoTracker3 motion filtering eliminated, how many the LAION aesthetic filter eliminated, how many the -45 dBFS silence filter eliminated, how many Audiobox eliminated, how many the TalkNet/sound-source attribution judgment eliminated — none of this has data.
  · The generation volume of stage two is unknown — how many candidate targets the data engine actually synthesized.
  · The elimination rate of stage-three verification is unknown — what proportion of synthesized samples the five-dimension MLLM verification rejected. This number is especially unfortunate to be missing: it directly quantifies the success rate of the data engine, a key metric for assessing this pipeline's engineering feasibility (how many attempts are needed to synthesize one usable sample, and at what cost), and is exactly the evidence most needed to support the self-positioning as a "scalable data synthesis pipeline." How many candidates the manual verification filtered 1K down from is likewise unreported.
【Cannot compute end-to-end retention rate】Since data on both ends is missing, an overall retention rate comparable to Apollo's 27% cannot be given.
【A rough extrapolation one can make】Referencing the experience of comparable synthetic data engines, the one-pass success rate for diffusion models doing mask-guided local edits is usually in the 30–60% range (failure modes include boundary artifacts, target deformation, temporal flicker); layered on top of the strict five-dimension verification (which must pass all five simultaneously), the stage-three retention rate is estimated to be roughly 20–50%; i.e. producing 79K training pairs might require synthesizing 150,000–400,000 candidates. This is purely extrapolation, unsupported by any data in the paper.

### [2026 miscellaneous joint audio-video generation works](../models/JAVG_2026_misc.md) ⚠️

Among this batch of works, only two give a computable retention rate; the rest are entirely blank:
【OmniCustom — the only funnel that can be computed precisely】
- Input: SpeakerVid-5M, "more than 5.2 million video clips" / "8,000 hours"
- Output: OmniCustom-1M, "about 1 million single-person video clips" / "2,500 hours"
- Clip-level retention rate ≈ 1.0M / 5.2M ≈ 19.2%
- Duration-level retention rate = 2,500 / 8,000 = 31.25%
- The inconsistency between the two retention rates is worth noting: the duration retention rate is significantly higher than the clip retention rate, meaning the retained clips are on average longer (retained clips average about 2500h/1M ≈ 9.0 seconds, vs. the original 8000h/5.2M ≈ 5.5 seconds) — this is the direct consequence of the "remove videos <10 seconds" rule, which cuts hard on clip count but only removes short clips.
- The pass rate of each of the three rules individually is not given separately [Uncertain].
【NAVA — only the two ends of the funnel, no middle layer】
- Input: "approximately 20M audio clips and 100M video clips"
- Output: "around 15M clips for large-scale training"
- Computed on the video side, retention is about 15M/100M = 15%; if computed jointly it would be even lower. The input/output volume and pass rate at each stage (OCR/deduplication/quality/alignment) are all undisclosed [Uncertain].
- A secondary convergence: 15M → 160K high-quality SFT subset, SFT retention rate about 1.07% (i.e. one in a thousand) — a number worth referencing, illustrating that there is a two-order-of-magnitude gap between "trainable" and "curated enough for SFT."
【ALIVE】No retention rate or original pool size is given at all; the trend of the data being progressively narrowed can only be inferred from the stage sample counts: 11M (joint training) → 4.3M (balanced, only 39% remaining after balancing) → 5M (SFT) → 0.7M (1080p high-definition subset, 6.4% of the 11M) → 0.8M (character pairing). The 39% retention rate in the "11M → 4.3M balanced" step is a reduction caused by data balancing (domain mixture adjustment), and is the only computable ratio in this batch besides OmniCustom [an inference, uncertain].
【Baton / CCL / StreamChar】No funnel information at all [Uncertain]. CCL's 4M is the final usage volume; the original candidate pool is unknown.
【ITS-JAVG】At inference time the concept of "retention rate" corresponds to the Best-of-N selection rate: JavisDiT is 1/5 = 20%, MMDisCo is 1/10 = 10% — coincidentally, this is on the same order of magnitude as the 10%–20% retention rate common in industrial data pipelines, incidentally suggesting that "oversampling then selecting the best" follows a similar economics on both the training side and the inference side.

### [Joint audio-video generation baseline collection](../models/JavisDiT_baselines.md) ⚠️

In this collection, only JavisDiT++ gives a computable funnel retention rate; the rest are entirely missing:
【JavisDiT++ (the only work with a quantitative funnel)】
- Video-side total retention rate: TAVGBench's original 1.1 million triples → 355K after filtering, overall retention rate about 32.3% (355K/1.1M). This number is on the same order of magnitude as publicly disclosed comparable funnels in the industry (e.g. Apollo's 27%), giving it cross-comparison value.
- Downstream allocation of retained data: 330K used for audio-video SFT (93% of retained volume), 25K used for AV-DPO (7%), the two sets strictly non-overlapping.
- Stage-by-stage retention rate is missing: the paper only gives the total, without breaking down how many entries were eliminated by "FunASR speech removal," "aesthetic scoring," "optical-flow scoring," and "OCR scoring" respectively, nor giving the specific threshold for each item [Uncertain]. Given that TAVGBench is fundamentally YouTube video with a high proportion containing speech, it can be speculated that the FunASR step is the largest elimination stage, but there is no data to support this.
- Audio-side retention rate is 100% (explicitly states "no data filtering strategy is applied" at all).
- Internal proportion within the DPO stage: of the final approximately 25,000 preference pairs, about 30% of the winning samples come from model generation (rather than ground truth); the authors use this to judge that "the baseline model already possesses fairly strong generative capability."
【JavisBench evaluation set funnel】About 30,000 candidates → pre-filtering for quality + Qwen automatic labeling and categorization + post-filtering for diversity + manual legal/ethical review → 10,140 entries, retention rate about 33.8% (10140/30000), coincidentally close to the training-side funnel retention rate. The elimination volume at each stage is not broken down [Uncertain].
【Harmony】On the speech side, 2 million entries were filtered by consistency scoring from the aggregated pool of Emilia + OpenHumanVid + SpeakerVid, but the original size of the aggregated pool is not given, so the retention rate cannot be computed [Uncertain].
【MM-Diffusion / AV-DiT / UniAVGen】No quantitative funnel data at all [Uncertain]. MM-Diffusion's 928 source videos → 1,000 clips is a splitting operation, not a filtering operation, and does not constitute a retention rate.

### [Kling 3.0 / Kling Video 3.0 Omni](../models/Kling_30_Omni.md) ⚠️

[Uncertain] Kling 3.0 Omni does not disclose the input/output volume or final retention rate for any filtering stage. Koala-36M, from the same team, publicly discloses a complete quantitative funnel that can serve as an order-of-magnitude reference: after splitting+labeling, Koala-all is 48 million entries → after manual threshold filtering, Koala-37M is 37 million entries (about 77% retention) → after VTSS filtering (threshold 2.5), Koala-36M is 36 million entries (about 97% further retention relative to the prior stage, about 75% end-to-end). On the Kling-Foley side no retention rate is disclosed, only the final scale (55 million clips / 122,000 hours).

### [LTX-2](../models/LTX-2.md) ⚠️

Entirely undisclosed. Neither the LTX-2 nor the LTX-Video paper gives the input sample volume, elimination rate, or final retention rate for any filtering stage, so it cannot be compared with quantitative funnels like Apollo's 27%. It is only known that the LTX-2 training set is "a subset" of the LTX-Video dataset, but the proportion of the subset relative to the parent set is likewise not disclosed. [Uncertain]

### [LongCat-Video](../models/LongCat-Video.md) ⚠️

[Uncertain]. The report gives no input/output volume or retention rate for any filtering stage at all, nor an overall final retention rate figure. This is one of the weakest points in this technical report's disclosure on the data side — it lists in detail which filters were used, but does not give a single threshold value or the pass rate of any single stage, so it cannot be compared horizontally with quantitative funnels like Apollo's 27%.

### [MOVA](../models/MOVA.md)

The paper gives, in Table 1 "Retention Ratio of Total Dataset Duration," the stage-by-stage retention rate computed by total duration (relative to the original video) — one of the rare publicly disclosed quantitative funnels in this survey:
- Raw (original video): 100%
- Stage 1 (after preprocessing, speech + non-speech segments): 84.57%
- Stage 1 (speech segments only retained): 58.75%
- Stage 2 (after quality and alignment filtering): 26.39%
That is, the final overall retention rate is 26.39%, on the same order of magnitude as Apollo's 27%. Breaking it down, there are two main sources of loss: (1) the "speech only" rule taking it from 84.57% → 58.75%, cutting 25.8 percentage points, the largest single elimination source (there is also a separate clip-count-based statement: speech clips account for 69.47% of all preprocessed clips); (2) the three-dimensional quality-and-alignment filter taking it from 58.75% → 26.39%, cutting a further 32.4 percentage points, a relative retention of about 44.9%.
In addition, the training stage applies an even stricter second-round narrowing (relative to Phase 1's 61,500 hours): Phase 2 drops to about 37,600 hours / 16.8M clips (about 61%), Phase 3 drops further to about 11,000 hours (about 18% of Phase 1). If the training curriculum is also counted as an extension of the funnel, the equivalent retention rate from the original material to the final 720p high-quality subset is far below 26.39%.
Public figures are also available for the output of Phase 2's three sub-filters: the OCR no-subtitle subset ~9.5M clips, the LSE lip-sync high-quality subset ~2.5M clips, the DOVER technical-score >0.15 subset ~2.4M clips, with the merged Phase 2 dataset being 16.8M clips.

### [Merged entry: ① Mochi 1](../models/Mochi_MAGI_Motif.md) ⚠️

① Mochi 1: None. [Uncertain]
② MAGI-1: No quantitative retention rate at all. Only qualitative statements: the initial filter "effectively discards most of the low-quality data," and by the time of MLLM filtering "the remaining data size has been significantly reduced," plus the directional statement in Table 5 that "the data volume decreases progressively across stages." Neither stage-by-stage input/output volumes nor an end-to-end retention rate is given, so the main elimination stage cannot be located. [Uncertain]
③ Motif-Video 2B: Provides a visualization of the funnel but no numbers. Fig.7 is a Sankey diagram; the paper text states it "visualizes how flows contract from the raw pool toward the curated training and SFT corpora," but the body text does not annotate any percentages or absolute volumes. The only anchorable endpoint is the final training set of "fewer than 10M clips"; the original pool size is not given, so the end-to-end retention rate cannot be computed. This is one of the few gaps in Motif's data disclosure.  [Uncertain]
Taken together, none of the three models in this group provide the kind of stage-by-stage quantitative retention rate seen in Apollo (27%) or Allegro (0.4%–26% tiered), a shared shortfall to note when making cross-comparisons.

### [Movie Gen](../models/Movie_Gen.md) ⚠️

Paper Table 44 gives a rare industry example of a stage-by-stage quantitative funnel (for the strictest high-resolution curation thresholds, the numbers are the remaining volume percentage relative to the original pool):
Duration 4s≤duration≤120s → 100%
Resolution width≥768 and height≥768 → 25% (a single step cuts 75%)
Aspect ratio width≥height → 7% (a single step cuts 72%, the steepest single cut)
No text (word-detection score × word-recognition score <0.6 for all sampled frames) → 1.94% (a single step cuts 72%)
No black borders → 1.87%
No scene cuts (sample 1 clip of 12–16 seconds from 1 scene) → 1.78%
Aesthetics (LAION aesthetic score of the clip's middle frame ≥4.0) → 1.57%
Non-slow-motion (motion score>2.0, motion-vector mean >0.5 and <7) → 1.32%
Non-shaky (shots per second <0.85) → 1.22%
No content repetition (embedding cosine similarity <0.99) → 1.15%
Concept resampling (retention per cluster 1/sqrt(cluster size)) → 0.94%
The paper explicitly states that under this set of thresholds "the data acceptance rate is less than 1%" — i.e. about one percent of the original video enters the high-resolution training set. The audio side gives no corresponding quantitative funnel [Uncertain].

### [NVIDIA NeMo Curator](../models/NeMo_Curator.md) ⚠️

Only single-stage quantitative numbers, no complete stage-by-stage funnel table.
【Disclosed quantitative points】(1) The semantic deduplication stage removes about 30% of the training data ("remove approximately 30% of training data"); (2) In visual-quality filtering, the DOVER-based distortion-assessment model removes the bottom 15% by score ("bottom 15%"); (3) The splitting stage discards all clips shorter than 2 seconds (proportion not disclosed); (4) Overall, from 20 million hours of original video, about 100 million 2–60-second clips are obtained — at a rough average clip length of 15 seconds this amounts to about 410,000 hours, a duration retention rate relative to the input of about 2%, but this estimate is not rigorous (a large amount of the content in the raw video is discarded during the splitting stage itself, and the paper does not confirm the average clip length).
【Missing】The elimination rates for motion filtering, text-overlay filtering, and video-type taxonomy filtering are individually not disclosed; input/output sample volume tables for each stage are not provided; an end-to-end retention rate comparable to Apollo's 27% cannot be constructed. [Uncertain]

### [OmniHuman dataset + OHBench](../models/OmniHuman.md) ⚠️

Entirely missing, the most severe gap in this work's data disclosure:
【Known】Final state: 1 million entries / 1,800 hours / 80,000 identities.
【Unknown】The original collection volume (how many YouTube videos crawled, how many hours); the input and output volume of each of the four filtering stages; the elimination proportion at each stage; the elimination volume during the tracking and audio-visual attribution stages; the elimination volume of the final consistency check; the overall end-to-end retention rate. The numerator is known but the denominator entirely absent, so a retention rate comparable to Apollo's 27% or MOVA's 26.39% cannot be computed, nor can the actual strictness of each filtering stage be judged.
【Qualitative inference (no basis in the paper)】Combining several strong constraints in the pipeline, one can judge that the overall elimination strength should be quite strong: (1) the hard HD-or-above resolution threshold would eliminate a large amount of old and low-bitrate content; (2) requiring an audio track that must be co-sourced with the on-screen person (SyncNet attribution + ±3-frame offset) would eliminate the vast majority of YouTube content with narration, post-dubbing, or pure BGM — this is likely the strongest gate; (3) the consistency check requiring the number of subjects labeled by the MLLM to exactly match the tracking result should have a low pass rate for chaotic multi-person scenes. Overall the total retention rate is likely on the order of single-digit percentages, but this is speculation, not a conclusion. [Uncertain]
【Impact on reproducibility】The paper emphasizes that one of its contributions is "a fully automated pipeline," yet provides neither thresholds nor a retention rate, so a third party rerunning the pipeline as described in the paper would end up with a dataset size and quality distribution that cannot be aligned with the original — the reproducibility of the pipeline is essentially not established. By comparison, UniVerse-1 discloses all thresholds and MOVA discloses a stage-by-stage retention table; OmniHuman lags behind on this point.

### [Open-Sora series](../models/Open-Sora.md) ⚠️

【Open-Sora Plan v1.3 — an industry-wide-rare stage-by-stage quantitative disclosure】The paper gives a complete seven-stage retention-rate table (cumulative retention relative to the original data):
  Video splitting (ffmpeg, 16s) → 100%
  Jump-cut detection (LPIPS, retain 32 ≤ frame count ≤ 512) → 97%
  Motion filtering (LPIPS, 0.001 ≤ score ≤ 0.3) → 89%
  OCR subtitle cropping (EasyOCR, edge ratio 0.20) → 89% (this stage only crops rather than discards, so retention is unchanged)
  Aesthetic filtering (LAION Aesthetic v2, ≥ 4.75) → 49% 【the single stage with the harshest elimination, cutting about 40 percentage points】
  Technical quality (DOVER technical score ≥ 0) → 44%
  Motion re-check (LPIPS, 0.001–0.3) → 42%
  **Final cumulative retention rate about 42%**.
According to another passage in the Open-Sora Plan v1.3.0 Report, the cleaned Panda70M **retains only 27% of the original data** (about 19 million entries out of an original approximately 70 million), which highly matches the order of magnitude of Apollo's reported 27%. The two numbers use different conventions (42% is the cumulative figure from the paper's funnel table; 27% is the final figure specifically for the whole Panda70M set); it is recommended to note the source version when citing either.
【Open-Sora 2.0】Explicitly states that a "layered data pyramid" was constructed to accompany progressive training, but **does not give the input/output volume or retention rate for any filtering stage**; only from the training-stage data volumes (70M → 10M → 5M) can the pyramid's tier ratio be indirectly seen to be about 14 : 2 : 1, i.e. the highest-quality tier is only about 7% of the base tier.
【Open-Sora 1.2】No stage-by-stage retention rate is given; a roughly inferable ratio is the original 30M clips (80k hours) → the final high-quality stage 2M clips (5k hours), about 6.7%. [Uncertain]

### [Ovi](../models/Ovi.md) ⚠️

[Uncertain]. The paper gives no input/output volume, pass rate, or overall retention rate for any filtering stage, and has no Apollo-style 27%-type quantitative funnel table. Only a qualitative judgment of high selection strength is possible: the SyncNet condition (|offset|≤3 and confidence>1.5) is self-described by the authors as "strict criteria," and layered together with the >720×720 resolution gate, RAFT static-shot removal, and aesthetic-score removal, the actual retention rate should be significantly lower than typical loose pipelines, but there is no data to support this. Ovi 1.1 only discloses that the final dataset size doubled, without disclosing the original candidate pool size.

### [Script-a-Video](../models/Script-a-Video.md) ⚠️

The paper gives no funnel quantitative data at all: the total volume of the original collected video is not disclosed (the numerator is known as 500K, the denominator entirely missing), the input/output volume of stage-by-stage filtering is not given, the stage-level or overall retention rate is not given, and the original pool size and retention rate for each of the four generation-side data sets (400K/250K/870K/60K) is not given either.
An overall retention rate comparable to Apollo's 27% or MOVA's 26.39% cannot be computed.
One "retention rate"-like analogy worth noting: at the annotation level, MTSS contains an implicit information-retention decision — the strict audio-visual coupling principle of the Event stream "filters" a large amount of audio information into global_audio, and the narrative-importance filtering of the Reference stream "filters" peripheral entities into the global scene description. But the paper does not quantify what proportion of information each of these two demotion channels absorbs. [Uncertain]

### [Seedance 2.0 and Seedance 1.5 pro](../models/Seedance_20_Seedance_15_pro.md) ⚠️

[Uncertain] None of the three generations of reports (1.0/1.5/2.0) give the input/output volume or retention rate for any filtering stage, so they cannot be benchmarked against disclosures like Apollo's 27%. This is the largest gap in the Seedance series' data disclosure.

### [SkyReels series](../models/SkyReels.md) ⚠️

The quantitative funnel disclosure is incomplete; only two usable ratios are available:
(1) SkyCaptioner-V1 training set: curated from "10 million original samples" down to "about 2 million concept-balanced videos," a retention rate of about 20% — this is the only clear "input→output" ratio in the SkyReels series, but it is the filtering rate for the captioning model's training set, not the funnel retention rate for the full training data of the generative model;
(2) Concept-balancing stage: doing concept balancing by subject category in the post-training stage "leads to a 50% reduction in data volume," i.e. this stage's retention rate is about 50%.
Also usable as a reference: the motion-recognition annotator's training data is "93,000 high-confidence manually annotated samples + 16,000 motion-axis-balanced synthetic samples," achieving an accuracy of 89% for single motion-type prediction on a 15,000-clip test set.
Neither generation of the paper gives a stage-by-stage input/output sample volume table, nor an end-to-end retention rate from an O(100M) original data to the final training set, so it cannot be directly compared with a complete funnel like Apollo's 27%. SkyReels-V4 discloses no filtering retention rate at all. [Uncertain]

### [Sora 2](../models/Sora_2.md) ⚠️

Entirely undisclosed. There is no input/output volume, retention rate, or elimination rate figure for any filtering stage. Cannot be compared with publicly disclosed quantitative funnels like Apollo's 27%. [Uncertain]

### [SpeakerVid-5M](../models/SpeakerVid-5M.md) ⚠️

The paper provides no stage-by-stage retention-rate table; an end-to-end estimate can only be made from the first and last conventions:
【Entry】153K videos / 64,386 hours of original material.
【Exit】more than 5.2M clips / 8,743 hours (including single-person branch 8.7K hours + dialogue branch 1.8K hours; the two overlap in samples, so the total is counted as 8,743 hours).
【End-to-end duration retention rate】8,743 / 64,386 ≈ 13.6%. This retention rate is notably lower than MOVA's 26.39% and Apollo's 27%, but the conventions are not fully comparable — the bulk of SpeakerVid-5M's loss is not quality filtering but structural loss: only retaining the two main speakers (other people discarded), only retaining segments with clear speaking/listening behavior, and the loss of frame area from YOLO single-person cropping — all of which substantially cut effective duration.
【HQ subset retention rate】1,368 / 8,743 ≈ 15.6% (relative to the final dataset); 1,368 / 64,386 ≈ 2.1% (relative to the original material). That is, the equivalent retention rate from the original material to high-quality dialogue data usable for SFT is about one-fiftieth.
【Undisclosed】The input/output volume and stage-by-stage elimination rate of each individual step (scene splitting, diarization, SyncNet binding, ArcFace correction, and each of the five dimensions in the quality filter) are not given; Figure 3 does show histograms of the DOVER-score and SyncNet-confidence distributions, but the elimination proportion cannot be inferred from these. This is a relatively weak link in this dataset's disclosure system. [Uncertain]

### [Step-Video-T2V](../models/Step-Video-T2V.md) ⚠️

The quantitative disclosure is incomplete; only partial estimation is possible:
【Known absolute volumes】Original pool → 2B video-text pairs (the total output of the pipeline) → about 1B clips available at the Step-2 low-resolution tier (actually seen 644M) → high-resolution tier seen 27.3M → SFT 30M → DPO/distillation about 95k.
【Computable retention rates】2B → 1B (192P tier) about 50%; 2B → 30M (SFT high-quality tier) about 1.5%, i.e. the end-to-end retention rate from pipeline output to SFT is about 1.5% (this conversion is a survey-derived estimate; the report does not give this ratio directly).
【Missing parts】(1) The absolute size of the original video pool (before splitting) is not disclosed, so the retention rate for the "original footage → 2B pairs" step cannot be computed; (2) The layered funnel diagram in Figure 11 only uses bar length to illustrate the retain/discard ratio of each stage, without annotating any absolute values or percentages; (3) The sample volume of each of the 6 pretraining subsets is not listed.
Therefore the completeness of this entry's quantitative funnel is lower than HunyuanVideo 1.5 (which gives the complete chain of >10 million hours → 800M → 200M → 100M → 1M); the only comparable point is the roughly 1.5% figure on the SFT side. [Uncertain]

### [UniTalking](../models/UniTalking.md) ⚠️

The paper gives no quantitative funnel data at all: the total volume of the original corpus (count or hours) is not disclosed, the input/output volume of each of the three filtering stages is not given, and the stage-level or overall retention rate is not given. The only known fact is that the final output is 2.3 million aligned samples; the numerator is known but the denominator entirely missing, so a retention rate comparable to Apollo's 27% or MOVA's 26.39% cannot be computed.
An indirect order-of-magnitude inference one can make (for reference only, with no basis in the paper): if the 2.3 million entries mainly come from OpenHumanVid's 13.2 million high-quality clips, the upper bound of the retention rate for that portion would be about 17.4%; considering that UniTalking additionally layers on the triple strong constraint of "must contain speech + sound source within frame + lip-sync," (OpenHumanVid, though it comes with its own audio track, was not filtered by speaker scene) the actual retention rate from the OpenHumanVid side should be significantly lower than this value, with the remainder made up by Huawei's internal data. But the specific mixing ratio of the two sources is unknown, so this estimate cannot be treated as a conclusion. [Uncertain]
Qualitative judgment: the elimination strength of this pipeline should be a fairly strong order of magnitude — each of the three chained stages is a hard boolean elimination (no soft weighting, no tiered retention), and the cross-modal-stage double condition of LightASD + LipSync inherently has a low pass rate for ordinary web video (a large amount of video has soundtrack, narration, non-synchronized sound, or too-small faces).

### [UniVerse-1](../models/UniVerse-1.md) ⚠️

The paper gives no quantitative funnel data at all: the total volume of raw collected video (hours or count) is not disclosed, the input/output volume of stage-by-stage filtering is not given, and the stage-level or overall retention rate is not given. The only known fact is that the final output is 7,685 hours (speech 1,187 + general 3,074 + public datasets 3,422). Since the numerator is known but the denominator entirely missing, an overall retention rate comparable to Apollo's 27% or MOVA's 26.39% cannot be computed.
What can be qualitatively inferred is that the retention rate should be fairly low: the three hard gates of resolution ≥1080p, bitrate ratio ≥600, and DOVER≥0.6 stacked together, plus the removal of clips <5 seconds after PySceneDetect splitting, and the additional SyncNet conf>2.0 for the speech subset, add up to considerable elimination strength; the relatively small size of the speech subset (only 1,187 hours) also indirectly reflects the stringency of the chained condition "has an audio track + has a face + lip sync meets standard." But all of these are inferences, unsupported by any data in the paper. [Uncertain]

### [Unison](../models/Unison.md) ⚠️

The paper gives no quantitative funnel data at all: the aggregated original candidate volume (denominator) is not disclosed, the input/output volume of stage-by-stage filtering is not given, and the stage-level or overall retention rate is not given. The only known fact is that the final output is about 2 million clips / 3,000+ hours (audio-video) and 50 million segments / 130,000+ hours (audio).
【A rough reference for the upstream scale — for order-of-magnitude sense only, not data from the paper】The publicly disclosed original scale of the five audio-video source datasets is roughly: OpenHumanVid tens of millions of clips, CelebV-Text about 70,000 entries, VFHQ about 16,000 entries, HDTF about a dozen-plus hours, VGGSound about 200,000 entries / 550 hours. Roughly computed on this basis, the combined original volume of the five is on the order of tens of millions of entries and above, while Unison's final retention is about 2 million entries, giving an overall retention rate possibly in the 10%–20% range — but this estimate is extremely unreliable: the actual usage proportion of each dataset is unknown, OpenHumanVid alone is large enough to sway the entire denominator, and it cannot be ruled out that the pipeline did a second round of splitting on long clips, thereby increasing the count. This estimate should not be cited as a conclusion. [Uncertain]
【Reason it cannot be computed】The numerator is known but the denominator is entirely missing, and the paper does not even state whether "the entirety of these datasets was aggregated, or only a subset." Compared with works like Apollo (27%) and MOVA (26.39%) that give a clear overall retention rate, Unison has no comparable data on this dimension.

### [Veo 3 / Veo 3.1](../models/Veo_3_Veo_31.md) ⚠️

[Uncertain] Entirely undisclosed. The official material gives no input/output data volume, retention rate, or final retention proportion for any filtering stage, so it cannot be compared with publicly disclosed quantitative funnels like Apollo's 27%. This is the most thorough gap in Veo 3's data disclosure.

### [Vidu S1](../models/Vidu_S1.md) ⚠️

[Uncertain]. Although the paper gives a complete six-stage funnel structure diagram (Figure 2), it provides no input/output data volume or retention rate figure for any stage, nor a final retention rate. This is one of the main gaps in this report's data disclosure (compared with the practice in Apollo-type works of giving a 27% retention rate).

### [Wan 2.5 / 2.6 / 2.7](../models/Wan.md) ⚠️

There are only two places with quantitative disclosure, insufficient to form a complete funnel ledger:
1) Wan 2.1 basic-dimension stage: "we successfully eliminated approximately 50% of the initial dataset" — the first-stage parallel coarse filter eliminates about 50%, with the remaining 50% moving on to semantic fine filtering. This is the only clear stage-level retention rate in the Wan series.
2) Wan 2.1 post-training image curation: takes the top 20% by expert-model predicted score.
The remaining stages (the specific cluster quota taken, the visual-quality score threshold, the retention ratio of each of the six motion tiers, the input/output volume ratio before/after V2A audio filtering, and the elimination rates at each S2V stage) all lack input/output volume and retention rate data. On the V2A side, only the result of O(1) thousand hours is known, with a parent set scale of "several billion videos"; if roughly converted, the retention rate would be far below 1%, but the report does not give a comparable convention, so this cannot be used as a funnel figure. Overall it cannot be aligned with a quantitative funnel like Apollo's 27%. [Uncertain]

### [Audio-video generation benchmark collection](../models/av_benchmarks.md) ⚠️

None of the five benchmarks disclose the input/output volume or retention-rate figures for stage-by-stage filtering [Uncertain]. Only indirect estimates are possible:
【AV-SyncBench】Final retention is 3,269 videos, but the original web-collected pool size is not disclosed, so the retention rate of the two stages (Gemini 3 Flash initial screening and 5-person manual review) cannot be computed [Uncertain].
【PhyAVBench】11,605 videos correspond to 337 paired prompt groups, an average of about 17 ground-truth videos per group; the paper's design target was N≥20 per group, and the actual mean of 17 indicates noticeable elimination during quality control (a rough estimate of about 85% retention, an inference rather than a disclosure from the paper [Uncertain]).
【AVBench】30K real clips → 300K training pairs is augmentation rather than filtering, so there is no concept of a retention rate; the original candidate pool size for the 470 evaluation prompts is not disclosed [Uncertain].
【VABench】The candidate generation volume corresponding to the 1,299 final samples is not disclosed [Uncertain].
Compared with training-side datasets (e.g. Apollo disclosing a 27% end-to-end retention rate, MOVA disclosing a stage-by-stage retention table), benchmark papers generally do not disclose quantitative funnel information — a common shortcoming of this category of literature.

### [Video caption model ecosystem](../models/caption_models.md) ⚠️

[Uncertain] The captioner ecosystem overall lacks quantitative retention-rate disclosure, with only scattered numbers:
· SkyCaptioner-V1: training data is curated from 10 million entries down to about 2 million concept-balanced videos, a retention rate of 20%. This is the only clearly stated captioner training-data retention rate in this entry.
· AVoCaDO: GPT-4.1 scores synthesis completeness from 1–5 and retains only ≥4, but the pass rate of this step is not disclosed.
· Tarsier2: recaptions 1 million videos from a public dataset, ultimately releasing 585K (Tarsier2-Recap-585K); if all 585K come from that 1M, the retention rate would be about 58.5%, but the paper does not confirm whether the two are the same batch (it could also be an actively sampled release rather than a filtered one). [Uncertain]
· Panda-70M's greedy set-cover gives "coverage" rather than a retention rate: the single best model covers 30.8%, 8 models cover 76.8%, all 31 models together cover 84.7%; at the same time the paper reports that human-to-human agreement is only 44.9%, showing that "the best caption" is itself highly subjective.
· Neither AVSCap-130K nor AVoCaDO-SFT-107K gives the stage-by-stage figures for "original candidate volume → final retained volume."
· The most complete end-to-end retention rate on the generation side comes from Apollo/Klear (27%), but that is for the whole data funnel, not the captioning stage.

### [Geometric / structured annotation dataset collection](../models/geometric_datasets.md) ⚠️

SpatialVID gives the clearest quantitative funnel: 21,789 hours of original YouTube video (33,443 entries) → final 7,089 hours of dynamic content (2.71 million clips), a duration-dimension retention rate of about 32.5%; the high-quality subset SpatialVID-HQ is 0.37M/2.71M clips, i.e. a further retention of about 13.7% on top of the already-filtered library (about 4.5% end-to-end relative to the original material). Key quantitative finding: more than 80% of videos in Panda-70M could not be successfully reconstructed by MegaSaM due to insufficient motion, constituting the main elimination source of the motion filter. SceneScribe-1M does not disclose the stage-by-stage input/output volume; only that from the hundred-million-scale source pool of HD-VILA-100M/Panda-70M/Koala-36M/Pexels, it converges to 1 million clips, 156.7M frames, 4000+ hours; the paper states "the source pool was substantially reduced during initial screening due to motion-diversity requirements," but the stage-by-stage retention-rate values are missing [Uncertain]; Action100M's quantification is on the deduplication side: it identifies 7.58M duplicate groups, 141.8 million duplicate instances, and removes them; of the source 1.2 million videos, 72% have usable ASR, and the final output is 147 million clips; WildWorld is directly captured by an engine, with no concept of a filtering funnel — it only mentions that a few thousand clips remain after filtering (the exact number is not given) [Uncertain]

### [Video generation post-training data strategy](../models/post_training_data.md) ⚠️

The anchor paper has no quantitative funnel data [Uncertain]. The horizontally comparable "post-training retention rate" (SFT curated set ÷ pretraining pool) is as follows — the most valuable set of comparable numbers in this topic:
· NAVA: 160K ÷ 15M ≈ 1.07% (the strictest in this survey, "taking ten out of a thousand");
· Allegro: 2M ÷ 500M = 0.4% (would be even stricter if computed on the original clip count, but its 2M is the direct output of the strictest threshold combination);
· Goku I2V: 4.5M ÷ 36M = 12.5% (and the selection criteria emphasize domain diversity rather than quality score);
· CogVideoX: top 20% (explicitly defined by percentile rather than a threshold);
· Open-Sora Plan T2V stage three: after filtering, Panda70M retains about 19M / 27%;
· NVIDIA Cosmos WFM: from about 10^8 pretraining clips, about 10^7 are extracted for fine-tuning (about 10%, selection criteria not disclosed);
· SkyReels-V4: 5 million → 1 million manually curated (second-stage SFT retention rate 20%).
【Empirical range】The retention rate of the SFT curated set relative to the pretraining pool is concentrated in the 0.4%–20% range, with a median of about 5%–10%; models that pursue aesthetics and a "cinematic feel" more aggressively tend to have lower retention rates (Allegro, NAVA), while models that pursue domain coverage or multi-task capability more aggressively tend to have higher retention rates (Goku, Open-Sora Plan, Cosmos).
【The "retention rate" on the preference-data side takes a different form】Its scale is determined by the annotation budget rather than a filtering threshold; SkyReels-V2's 30,000 manually annotated sample pairs, Step-Video's undisclosed count, and HunyuanVideo 1.5's O(10K) are all manifestations of "the ceiling of annotation capacity." JavisDiT++ gives a distinctive metric: about 30% of the winning samples in the final preference data come from model generation rather than ground truth; the authors use this to judge that "the baseline model already possesses fairly strong generative capability" — this ratio can serve as an empirical signal for judging whether a model has reached the stage where "self-generated data can be used for preference optimization."

### [Combined survey of mainstream video pretraining datasets: Panda-70M, InternVid, Koala…](../models/pretraining_datasets.md) ⚠️

**The degree of quantitative disclosure varies enormously; only three give usable funnel numbers**:
- **UltraVideo (the most complete disclosure)**: 5,000 original videos → after shot-cut and frame-count filtering, **62K short + 25K long** → after statistical filtering, **46K short + 19K long** (retention rate about **74% / 76%**) → after model-based purification, **42K short + 17K long** (retention rate about **91% / 89%**). **Computed from the post-split pool, the overall short-clip retention rate is about 68%**. But how many entries each check individually removed is not reported.
- **Koala-36M (gives two key figures)**: after re-splitting+labeling, the unfiltered pool is **48M clips** (this number comes from the denominator in Appendix D Table 8) → the manual multi-threshold filtering baseline is **37M** (77.1%) → **VTSS>2.5 final 36M (75.0%)**. That is, this quality-filtering stage has a retention rate of **75%**, and VTSS is **slightly stricter** than the manual multi-threshold approach (36M vs 37M) yet preserves more high-quality samples. **The production rate of the splitting step, from 103M source videos to 48M clips, is not reported.**
- **LVD-2M (only the first and last points)**: **220M input clips → 2M output, overall retention rate about 0.91%** — the lowest among the seven. **The stage-by-stage retention rate is entirely unreported** (the paper's Figure 2 is only a schematic diagram, with no counts). The only intermediate statistic is a diagnostic on InternVid: only 15% of clips exceed 10 seconds, and among those long videos about 52.5% contain a shot change.
- **MiraData (tiered but without thresholds)**: unfiltered pool **788K** → 330K (41.9%) → 93K (11.8%) → 42K (5.3%) → 9K (1.1%). There is also an extremely persuasive standalone number: **HD-VILA-100M has about 100 million input clips, with only 195,000 finally surviving (about 0.2%)**, which the authors use to argue that the quality of their own manually curated YouTube channels is far higher than generic crawling. **But the paper states that the specific thresholds between the four tiers are in the supplementary material, and that supplementary material in fact does not exist** (I checked the arXiv v1 appendix, the NeurIPS camera-ready version, and the supplementary ZIP link, and none contain the thresholds) — this is a genuine reproducibility gap.
- **OpenVid-1M**: only a qualitative description (Panda-50M candidate pool → filtered to Ours-0.4M → split and expanded to Ours-0.6M → merged with the remaining three sources to reach about 1M), **the absolute quantity at each stage and the overall retention rate are not disclosed**. [Uncertain]
- **Panda-70M**: 3.8 million source videos → 70.817 million clips, this is **an expansion ratio (about 18.7×) rather than a retention rate** — it does almost no quality filtering. The only filtering-related number is the published-release figure of 70,723,513 (about 94,000 fewer than the paper, attributed to harmful-content filtering, which the authors do not clearly state) and **89.6% of samples having matching_score>0.43**. There is also a desirability distribution added in October 2024 (share of the full set): desirable **80.5%**, low-desirability score 5.28%, static foreground 6.82%, minimal camera motion 1.20%, picture-in-picture 5.03%, screen recording 1.13% — **i.e. by their own assessment, about 19.5% of samples are still undesirable**.
- **InternVid**: **no funnel numbers disclosed at all**. [Uncertain]
**Citable empirical values**: Looking across datasets, the order of magnitude of the retention rate for "filtering from a generic web-video pool down to a usable training set" is — about 100% with no quality filtering (Panda), about 70–75% with routine quality filtering (the statistics-layer figures of Koala and UltraVideo), and about 1% with strict quality + long-shot filtering (LVD-2M's 0.91%, MiraData's 0.2% relative to HD-VILA). **The cost of the combined constraint "long shots + high dynamics + no scene changes" is a three-orders-of-magnitude data loss** — one of the strongest quantitative conclusions of this survey.

## Shot segmentation method (PySceneDetect / in-house model / shot-aware splitting)

`shot_segmentation` · Level of detail: brief

### [Allegro](../models/Allegro.md)

Uses the open-source tool PySceneDetect for shot/scene segmentation, splitting original long videos into single-scene clips; after splitting, only clips of 2–16 seconds are retained.
A notable engineering detail: the first and last 10 frames of each clip are discarded after splitting, to eliminate cross-shot contamination caused by false positives in scene detection (transition fade-ins/outs, residual frames at cuts).
Does not use an in-house shot-aware splitting model, nor does it do transition-type classification or multi-shot recomposition.

### [Apollo](../models/Apollo.md) ⚠️

A scene-splitting stage is confirmed to exist, with a clear purpose but undisclosed method: the paper only states, "We then apply scene splitting to ensure each sample contains only one scene," placed after video-quality filtering. It does not state whether PySceneDetect, TransNetV2, or an in-house model is used, does not give the cut-point detection threshold, does not describe how the sampling window is chosen after splitting, and does not state whether, like MOVA, speech boundaries (VAD) are jointly combined with scene cut points to do dual shot-aware + speech-aware splitting. Given that Apollo's data is predominantly speech/singing, if the splitting is not speech-boundary-aware, it would cause utterances to be truncated, but the paper does not address this. [Uncertain]

### [CineDance / CineDance-1M](../models/CineDance.md)

【Tool】TransNetV2, a mature deep-learning shot-boundary detection model in the industry (compared to traditional pixel-histogram-based methods like PySceneDetect, it is more robust to gradual transitions, dissolves, and wipes); it cuts 25,899,474 atomic shots from the 44,579 cleaned videos.
【Key differentiator】Splitting is only an intermediate step, not the endpoint. Traditional datasets treat "one shot = one sample"; this work adds a further layer of "bottom-up narrative grouping" on top of the shots, using Qwen3.5-27B, guided by four rules of film theory (multi-angle, parallel editing, causal action/ellipsis, montage), to regroup semantically coherent adjacent shots into 1,201,912 narrative sequences — the sequence, not the shot, is the final sample.
【Anti-hallucination design】Rather than letting the LLM output timestamps directly, the shots are first indexed by number, and the LLM outputs the grouping result over shot indices (bottom-up shot indexing), substantially reducing timestamp hallucination.
【Long-video inference】A context-aware sliding window, roughly 3 minutes wide, with the window boundaries forced to align to shot boundaries, avoiding splitting a single shot in half.
【Anti-fragmentation】A manual reference set showed the shortest narrative completeness needing 18.4 seconds, hence a 20-second soft threshold was set; ultimately only 3.1% of sequences are shorter than 20 seconds.
【Parsing quality】Qwen3.5-27B combined with the bottom-up strategy achieves F1 = 88.4% on the parsing task (Tab.4).

### [CogVideoX](../models/CogVideoX.md) ⚠️

The paper does not mention using PySceneDetect, FFmpeg scene detection, or any explicit shot-boundary detection/splitting tool [splitting tool uncertain] — a clear gap in its pipeline description.
The actual mechanism achieving single-shot status is "discriminative removal" rather than "splitting": a dedicated classifier is trained to identify Lack of Motion Connectivity (in the body text and Appendix Table 14 this classifier is named Classifier-Static, test accuracy 0.92), defined as "clips lacking coherent motion at transitions, often found in manually spliced videos or videos edited from static images"; layered on top is an Editing classifier (accuracy 0.91) that removes material with obvious editing and effects. Together these two ensure that the 35M clips entering the training set are single-shot and motion-continuous.
In other words, CogVideoX chose the strategy of "discard the whole clip rather than do fine-grained splitting," trading lower data utilization for implementation simplicity and avoiding introducing a splitter's error.

### [Cosmos-Predict2.5](../models/Cosmos-Predict25.md) ⚠️

Stage one is exactly "shot-aware video splitting": high-accuracy boundary detection models (plural, implying a combination of multiple models) are used to split long videos into shot-level clips, with the core quality requirement of "ensuring that raw shot transitions are excluded" — the cut clips must not retain any of the original transition frames, avoiding abrupt visual changes within a training sample. The paper does not name the specific model (does not mention PySceneDetect / TransNetV2, etc.), but NVIDIA's independently open-sourced Cosmos Curator provides both a pixel-threshold-based fast splitting mode and a TransNetV2-based neural-network splitting mode for the corresponding implementation, which can serve as engineering-level corroborating evidence.
Splitting is immediately followed by GPU transcoding and cropping (removing black borders and spatial padding), with a lower bound of 5 seconds used to discard clips that are too short, and an upper bound of 60 seconds.
The safety net for transition quality is in the filtering stage: the semantic artifacts filter (VTSS-like) specifically removes poor transitions and video-in-video, i.e. it provides a secondary remedy for splitting misses.
[Uncertain: the specific model used for shot-boundary detection]

### [Data-Juicer 2.0](../models/Data-Juicer.md)

Shot segmentation is one of the more fully supported areas among DJ's video operators, offering three strategies to choose from:
  · video_split_by_scene_mapper — "splits the video into scene clips based on detected scene changes," i.e. standard shot boundary detection splitting, built on top of the scenedetect (PySceneDetect) library, supporting its ContentDetector / ThresholdDetector / AdaptiveDetector detectors with configurable thresholds. This is fully in line with the mainstream industry technical route (Apollo, Movie Gen, Open-Sora, and others all widely use PySceneDetect).
  · video_split_by_duration_mapper — mechanically splits at a fixed duration, unaware of shot boundaries, suitable for material already known to be single-shot long video or scenarios insensitive to cut points.
  · video_split_by_key_frame_mapper — splits at video-encoding key frames (I-frames). Advantages: cut points align with the encoding structure, no re-encoding needed, very fast; disadvantage: I-frame positions are decided by the encoder and do not necessarily correspond to true shot boundaries.
【Auxiliary operator】video_clip_reassembly_mapper is used to re-stitch results computed separately on overlapping clips (e.g. hand-motion trajectories) back onto the long-video timeline, serving long-horizon annotation for embodied-AI scenarios.
【Engineering level】Starting in v1.4.6, video byte-stream I/O is supported, so splitting can be done entirely in memory without repeated disk writes — a substantive improvement to I/O overhead for large-scale splitting.
【Actual usage】The official T2V use case does not enable the splitting operator (the source dataset is already pre-split). So while DJ's splitting capability is complete, it lacks public real-world data on ultra-large-scale long-video corpora (e.g. splitting throughput, average clips produced per hour of video).

### [Foley-Omni](../models/Foley-Omni.md) ⚠️

[Uncertain] The paper does not mention any shot segmentation method at all. No mention of PySceneDetect, TransNetV2, an in-house shot-boundary-detection model, or any shot-aware splitting.
Inferring from the data composition, the need for splitting is itself weak: the training data is dominated by public datasets, and GRID, LRS2, VGGSound, AudioCaps are all already pre-split fixed-length short clips (VGGSound fixed 10 seconds, GRID about 3 seconds), requiring no further splitting; only the internal audio-video corpus and SpeakerVid/TalkVid may involve slicing from long videos, but the paper does not describe how they are handled. The motion-score upper bound of 3.2 may indirectly serve to remove clips containing hard cuts (a shot change would cause an abnormal spike in optical-flow/motion score), but this is a side effect rather than an explicit shot-segmentation design.

### [Goku](../models/Goku.md)

A two-stage shot segmentation, a relatively refined approach for its time:
(1) Stage one — PySceneDetect does the initial shot boundary detection, quickly locating transitions based on pixel/histogram differences.
(2) Stage two — DINOv2 semantic-feature verification: the split clips are sampled at 1 frame per second, DINOv2 visual features are extracted, and the cosine similarity between adjacent sampled frames is computed; the whole segment must maintain high similarity to be judged as "the same shot and visually coherent." The threshold tightens with resolution: the 480×864 tier requires ≥0.85, the 720×1280 tier requires ≥0.90. This step's role is to patch PySceneDetect's misses (e.g. gradual transitions, large content changes within the same scene), and functions as a "shot-aware splitting + semantic-consistency double insurance."
(3) Length constraint — each clip is at most 10 seconds; overlong shots are forcibly truncated.
Does not use an in-house end-to-end splitting model, nor does it mention alternatives like TransNetV2.

### [Hailuo / MiniMax Video](../models/Hailuo.md) ⚠️

[Uncertain] (no disclosure whatsoever). Not stated whether PySceneDetect, TransNetV2, or an in-house shot-boundary-detection model is used, nor whether shot-aware splitting is performed. It can only be inferred from "the output is 6–10 second single-shot clips" that the training data must have gone through some form of shot segmentation, but the method is entirely unknown.

### [HunyuanVideo-Foley](../models/HunyuanVideo-Foley.md) ⚠️

【Method】Uses "scene detection algorithms" to perform shot-boundary detection and splitting on original long videos, then regularizes the split results into fixed 8-second blocks. The paper uses a generic plural term without naming a specific tool — it does not state whether it is PySceneDetect, TransNetV2, AutoShot, or an in-house approach, nor does it give the detector type (content detection/adaptive detection) or threshold parameters. [Uncertain: the specific tool and parameters]
【The purpose of splitting and its special relevance to the V2A task】In video generation models, the purpose of shot segmentation is to ensure visual temporal coherence (avoiding abrupt image changes within a training sample). In the V2A task, shot segmentation has an even more critical significance on the audio side: shot changes are often accompanied by abrupt changes in the acoustic scene (indoor→outdoor, close-up→wide-shot reverberation changes); a cross-shot clip would present the model with a contradictory supervisory signal of "one continuous audio track corresponding to two unrelated acoustic environments." So for V2A, shot segmentation is essentially a means of ensuring acoustic-scene consistency, no less important than visual coherence. The paper does not articulate it from this angle, but the design is internally consistent in this respect.
【The relationship between splitting and fixed-length blocking】Splitting first by content boundary, then by fixed length — two separate steps. This differs from both UniVerse-1 (splits then discards short clips, retains variable length) and MOVA (VAD-aware adaptive windows) — this work performs no adaptation at all, purely mechanical fixed-length blocking, simple but potentially producing truncation at event boundaries.
【Splitting is not jointly decided with audio event boundaries】Entirely driven by visual scene cut points, without reference to the onset/offset of audio-side events. This means a complete acoustic event could be cut in the middle by an 8-second boundary, and the model would see samples that have "onset but no decay" or "no onset but only a tail." For a Foley task that needs to precisely model the acoustic envelope, this is a potential data flaw that the paper does not discuss.

### [HunyuanVideo](../models/HunyuanVideo.md)

【Original version】PySceneDetect does the main splitting (into single-shot clips); Transnet v2 and PySceneDetect provide scene-boundary information dual-path for cross-validation; after splitting, the OpenCV Laplacian operator is used to pick a sharp frame within the clip as the starting frame, avoiding starting with a blurry transition frame.
【1.5】PySceneDetect + custom operators jointly detect scene boundaries, uniformly splitting into 2–10 second clips; the key increment is adding a specially trained "transition classifier" after splitting for secondary cleaning, removing clips that still contain gradual/dissolve-type transition effects — indicating the team believes pure threshold-based shot detection has misses that need model-level correction.

### [InstructAV2AV](../models/InstructAV2AV.md) ⚠️

Uses PySceneDetect (Castellano, 2024) as the shot-splitting tool, placed as the first step of the entire pipeline, cutting original long videos into single-shot clips.
【Why it is placed first】In this pipeline, shot segmentation is not an optional cleaning step but a hard prerequisite: the downstream CoTracker3 point tracking, Grounded-SAM-2 mask propagation, TalkNet speaker localization, and mask-guided video synthesis all rely on continuous field of view within a clip, and any shot change would simultaneously break all four steps. In contrast, shot segmentation for text-to-video models is mainly to prevent the model from learning abrupt transitions, a much lower priority.
[Uncertain] The specific configuration is not disclosed: the detector type used (ContentDetector / AdaptiveDetector / ThresholdDetector), the threshold parameters, and the minimum shot-length setting are not stated. Nor is it mentioned whether a learning-based shot-boundary-detection model like TransNetV2 is layered on top as a secondary check — PySceneDetect is based on pixel statistics, and has a higher false-detection rate on gradual transitions (fade/dissolve) and fast camera movement, and the film material in the sources is precisely rich in such transitions, a potential quality risk the paper does not discuss.
【Fixed-length processing after splitting】Single-shot clips are further windowed at 5 seconds; the specific windowing rule is not described (see duration_distribution).

### [2026 miscellaneous joint audio-video generation works](../models/JAVG_2026_misc.md) ⚠️

Disclosure is minimal, a shared weak spot for this batch of works:
【NAVA — the only one to mention splitting engineering】"raw videos are first segmented at scale with a Hadoop-based pipeline" — a distributed pipeline based on Hadoop is used for large-scale splitting. It is worth noting that the paper only describes the engineering framework (Hadoop) without naming the splitting algorithm or tool (not stating whether it is PySceneDetect, TransNetV2, or an in-house shot-detection model) [Uncertain]. This "engineering framework mentioned, algorithm not" style of disclosure itself suggests that splitting is treated as an already-solved engineering problem. After splitting the average clip is about 7 seconds.
【ALIVE】There is no independent shot-segmentation stage among the six data-pipeline stages; the splitting action is implicit within the character-driven pipeline: "Extracts N clips (3–10 seconds) from long videos (10–30 minutes)" — extracting N clips of 3–10 seconds from 10–30 minute long videos, but the extraction criterion (whether based on shot boundaries, speaker activity, or random) is not stated [Uncertain]. Given that its identity-anchor selection uses "the 1.5-second sub-clip with the highest sync score," extraction is very likely driven by sync score rather than shot boundaries [inference, uncertain].
【OmniCustom】Does not do splitting — directly uses clips already split by the upstream SpeakerVid-5M, doing only the filtering of "remove clips <10 seconds" and the fixed-point cropping of "first 4 seconds / last 5 seconds." This is a typical example of "outsourcing splitting to the upstream dataset."
【StreamChar】Likewise does no splitting, entirely reusing the existing splitting granularity of the three datasets SpeakerVid-5M / TalkVid / OpenHumanVid, only applying a "≤20 seconds" length constraint.
【Baton / CCL】Splitting method not mentioned [Uncertain]. Both use already-split public datasets like OpenHumanVid, plus self-collected/in-house parts (Baton's internet video, CCL's short-drama films) — the latter would theoretically have to be split, but the paper does not describe this at all [Uncertain], a clear gap in CCL's data disclosure (film and short-drama content is the type of content with the most frequent shot changes, and splitting quality should have a substantial impact on their results).
【ITS-JAVG】Not applicable.
【Overall judgment】Among this batch of 2026 works, shot segmentation has become highly "infrastructure-ized" — either outsourced to public datasets, or only the engineering framework is mentioned without the algorithm. This is in contrast to the 2024–2025 trend of papers commonly naming PySceneDetect/TransNetV2, reflecting that this stage is no longer regarded as a technical contribution point.

### [Joint audio-video generation baseline collection](../models/JavisDiT_baselines.md) ⚠️

This collection is generally weak on shot segmentation, one of the areas showing the clearest gap versus industrial-grade pipelines:
【MM-Diffusion】Explicitly adopts "mechanical equal-length splitting" rather than shot-aware splitting: it splits 928 source videos "into 1,000 non-overlapping 10-second clips," without using PySceneDetect, TransNetV2, or any scene-detection tool. This means the resulting clips may cross shot boundaries and contain transition frames — a known rough edge of early data processing. The authors additionally apply cropping to AIST++ (the AIST++_crop in the repository), which is spatial cropping, not temporal splitting.
【AV-DiT】Does no splitting, directly sampling 16 frames from already-split dataset clips.
【JavisDiT / JavisDiT++】Relies on the segmentation already done by upstream TAVGBench; its own pipeline has no shot-detection stage; the video-processing steps in data.md are only "remove corrupted videos, filter out samples with fewer than 10 frames, normalize fps to 16Hz," with no scene detection [Uncertain: whether the TAVGBench upstream did shot detection].
【Harmony】Whether the 3–10 second clip division is based on shot detection is not stated [Uncertain]; the splitting method for the 2 million self-collected ambient-sound clips is likewise undisclosed.
【UniAVGen】No splitting method described at all [Uncertain].
【Impact】The lack of shot-aware splitting means the training data may include clips containing transitions, which in theory would harm temporal consistency; but since all these works use extremely short clips (1.6–10 seconds), the cross-shot risk is relatively controllable. This also explains why none of these models have multi-shot generation capability.

### [Kling 3.0 / Kling Video 3.0 Omni](../models/Kling_30_Omni.md) ⚠️

[Uncertain, though the same team has a public methodology paradigm] The Kling-Omni report only states that it "detects and removes abrupt scene changes and incoherent shot transitions," without naming a tool. Koala-36M, from the same team, proposes and open-sources an in-house shot-splitting method, Color-Struct SVM (CSS): using BGR-histogram correlation to measure color distance and Canny-brightness SSIM to measure structural distance, with a linear SVM classifier making the transition determination; it is further layered with temporal smoothing, assuming video change follows a Gaussian distribution, and only judges a real transition when it exceeds the 3σ confidence interval, thereby distinguishing "gradual transitions" from "fast motion scenes" — explicitly claimed to outperform threshold-based methods like PySceneDetect. Kling 3.0 Omni very likely continues an iterated version of this in-house splitter, but there is no official confirmation.

### [LTX-2](../models/LTX-2.md) ⚠️

The pipeline's input unit is directly labeled "Input Shots," indicating shot segmentation is already complete upstream, and the entire pipeline (motion estimation, deduplication, resizing, aesthetic scoring, filtering) treats the shot as its atomic unit throughout, with the final product being "Final Shots." But neither paper states the specific method used for shot segmentation — neither mentioning open-source tools like PySceneDetect/TransNetV2, nor stating whether an in-house shot-aware splitting model was used, and no threshold parameters. [Uncertain]

### [LongCat-Video](../models/LongCat-Video.md)

Adopts a dual-path joint scene-splitting scheme: the open-source tool PySceneDetect and an in-house trained TransNetV2 model are used together. TransNetV2 is a dedicated shot-boundary-detection neural network, significantly better than PySceneDetect's threshold-based method at recognizing gradual transitions (fade/dissolve), and the team further retrained it on their own data. The goal of splitting is to obtain "training-friendly clips while maintaining content consistency" — clips that are both suitable for training and content-consistent. During splitting, FFMPEG simultaneously performs black-border cropping (black border cropping during transition segmentation).

### [MOVA](../models/MOVA.md)

Uses PySceneDetect for scene-cut-point detection, but MOVA's key innovation is combining scene cut points with speech boundaries to jointly drive window sampling, rather than simply splitting by scene:
【Detection】PySceneDetect detects and records the timestamps of all scene changes across the full video; Silero VAD annotates speech intervals in parallel.
【Multi-shot window generation (Appendix A.3, Algorithm 1)】Iterate over VAD speech segments: the window start's upper bound is set to the current speech segment's start; the lower bound takes the maximum of three values — the end of the previous speech segment, the nearest scene cut point before the current speech segment, and the current speech segment's start minus half the window length (8.05/2); a window start is sampled uniformly at random within this range; the window length is fixed at 8.05 seconds; only when the window's time span contains at least one scene cut point is it retained as a multi-shot sample; the process then jumps to the first speech segment whose start is later than the window's end, avoiding overlapping windows.
【Single-shot window generation (Appendix A.3, Algorithm 2)】Iterate over the scene intervals formed by adjacent scene cut points, and within each scene look for a speech segment whose start satisfies "start + 8.05 seconds still falls entirely within this scene"; the start's lower bound takes the maximum of (the end of the previous speech segment, the scene's start, the current speech segment's start minus half the window length), sampled randomly; if the window's end goes past the scene's end, it is aborted; overlapping windows are likewise skipped.
【Design intent】(1) The lower-bound constraint of "no earlier than the end of the previous speech segment" avoids the window cutting into the previous utterance; (2) the lower-bound constraint of "no earlier than the nearest scene cut point" encourages natural transitions; (3) the "minus half window length" constraint avoids the window start being too far from the target speech, which would leave the first half empty; (4) random sampling of the start brings data augmentation and positional diversity. Overall this is a dual shot-aware + speech-aware splitting scheme, custom-tailored for the lip-sync task.

### [Merged entry: ① Mochi 1](../models/Mochi_MAGI_Motif.md) ⚠️

① Mochi 1: Not disclosed. [Uncertain]
② MAGI-1: Uses open-source PySceneDetect to split long videos into short clips, explicitly requiring that "each clip contains only a single shot." Because PySceneDetect handles complex transitions (gradual, dissolve, etc.) poorly and split clips may still retain multiple shots, the team added an independent Transition Detection filter as a backstop: sparsely sampled key frames, with CLIP computing the semantic similarity between adjacent key frames; if it falls below a predetermined threshold, the clip is judged to contain multiple shots and is discarded in its entirety. Note its backstop strategy is "deletion" rather than "re-splitting."
③ Motif-Video 2B: Adopts an "over-splitting + semantic re-merging" bidirectional strategy, the most elaborate of the three. Step one detects scene boundaries using a "conservative threshold biased toward over-splitting," i.e. deliberately allowing false positives (over-cutting) to outnumber false negatives (missed transitions); step two uses stitch detection based on SigLIP embedding similarity to re-merge adjacent clips, specifically restoring continuous shots that were mistakenly cut apart due to sudden intense motion or exposure changes; step three discards clips shorter than 2 seconds after merging. The third backstop comes from the VLM's multi_scene label — a hit means discard — which the paper explicitly calls "a secondary check on scene segmentation." Compared with MAGI-1, Motif's difference lies in acknowledging that the splitter errs on both sides and providing a remedy for each (merging to fix over-splitting, VLM labels to fix under-splitting), whereas MAGI-1 only addresses the under-splitting side.

### [Movie Gen](../models/Movie_Gen.md)

A dual-tool combination: ① FFmpeg's scene change detection handles the main splitting — first detecting scene boundaries, sampling 1–2 scenes longer than 16 seconds from each original video, then randomly extracting one 4–16 second clip from within each scene as the training clip; the paper explicitly states it does not do "random sampling that ignores scene boundaries," otherwise the generated video would show frequent, abrupt scene changes. ② PySceneDetect's Shot Boundary Detection is used as a shakiness detector rather than a splitter — because FFmpeg's motion score and motion vector have difficulty identifying frequent camera shake, and shaky video would be split by SBD into a large number of false-positive shots, so "shots detected per second" is used as a proxy metric for shakiness, with anything exceeding 0.85 shots/second removed. ③ Following Panda-70M's practice, if a clip's start coincides with the start of the whole video, the first few seconds are discarded (the beginning often contains unstable camera movement or transition effects).

### [NVIDIA NeMo Curator](../models/NeMo_Curator.md)

Well-disclosed with a rationale for the selection, the most solidly argued step in this pipeline.
【Selection】TransNetV2 neural network for shot-boundary detection. The Cosmos WFM paper explicitly documents a benchmark comparison against three alternatives — PySceneDetect, Panda70M's splitter, and AutoShot — with TransNetV2 selected for achieving the best F1 = 0.967 on the BBC dataset. This is a rare public example of treating "splitter selection" as a quantifiable decision, and its comparison method is worth borrowing directly.
【Framework implementation】NeMo Curator offers two splitting modes that can be combined: fixed-stride and scene-change detection (TransNetV2).
【Post-processing】A stitching stage follows splitting: image-embedding similarity is used to judge whether adjacent clips should be merged, suppressing over-fragmentation.
【Length constraint】<2 seconds discarded, >60 seconds re-split.
【Undisclosed】The specific threshold value used by TransNetV2, and the similarity threshold used for stitching.

### [OmniHuman dataset + OHBench](../models/OmniHuman.md) ⚠️

【Method】TransNetV2 — a dedicated deep-learning shot-boundary-detection model (compared to PySceneDetect's pixel/histogram-difference method, it has significantly better recall for gradual transitions, fade-ins/outs, and dissolves, making it a more suitable choice for YouTube content dense with manual editing artifacts). Positioned at the very front of the pipeline's first stage, a prerequisite for all subsequent processing.
【Assessment of the choice】This is a selection clearly superior to comparable works in the pipeline: UniTalking has no splitting stage at all, and most works use PySceneDetect. Talk-show, review, and film-clip content on YouTube commonly carries a large number of transition effects; using a threshold-based method for splitting would produce a large number of clips contaminated across transitions, and TransNetV2 is a fitting choice for this problem.
【Processing after splitting】The clips produced by splitting are directly used as sample units, with duration determined by the shot's own length (average about 6.5 seconds). The paper does not state whether overlong shots undergo secondary truncation, nor whether upper/lower duration bounds were set (though the audio-governance stage has a "duration anomaly" removal step, hinting bounds exist, but the values are not given). [Uncertain]
【Backstop for transition residue】The fourth-stage perceptual hashing + SigLIP cosine-similarity filter for "semantically inconsistent frames" can be viewed as a secondary backstop for splitting misses — if a clip still contains a cross-scene semantic jump internally, it would be caught here. This "splitting + semantic-consistency recheck" double-insurance design is more robust than relying on the splitter alone.
【Threshold】TransNetV2's decision threshold is not disclosed. [Uncertain]

### [Open-Sora series](../models/Open-Sora.md)

【Open-Sora 1.x】Uses **PySceneDetect**, with fully open-source code: `python tools/scene_cut/scene_detect.py meta.csv` outputs a list of scene timestamps for each video (in the form `[('00:00:01.234','00:00:02.345'), ...]`), which `tools/scene_cut/cut.py` then splits by, producing files named `{video_id}_scene-{scene_id}.mp4`; preceding this is `convert_id_to_path.py`, which does corrupted-file filtering (outputting an intact-flag column).
【Open-Sora 2.0】Switches to **FFmpeg libavfilter's scene score** for shot-boundary detection (faster than PySceneDetect and can be done in the same pipeline pass as transcoding), followed still by **PySceneDetect's Shot Boundary Detection** as a secondary detector for camera shake/abnormal jumps. After splitting, a fixed 8-second truncation is applied.
【Open-Sora Plan】Does not do content-based shot splitting; instead first mechanically splits into **16-second fixed-length clips using ffmpeg**, then uses **LPIPS perceptual distance** to detect whether a jump cut exists within the clip, discarding the entire clip if a change is detected (retention rate 97%). This is a different technical route from the previous approaches: "detect and discard" replaces "detect and split at the cut point," which is simpler to implement but loses some usable material.

### [Ovi](../models/Ovi.md) ⚠️

Uses "scene detection" to split long videos into intra-shot clips, then extracts a fixed-length clip of 121 frames @24fps from within them. The paper only writes the generic term "scene detection" without naming the specific tool (not stating whether it is PySceneDetect, TransNetV2, or an in-house model) [Uncertain]. Splitting and filtering are coupled in the same step: a split clip must simultaneously satisfy resolution, motion, aesthetics, and facial-composition conditions to be retained, so this is not a "split then filter" two-stage process but rather "splitting is filtering." The splitting granularity means all training samples are single-shot, with no shot-transition samples.

### [Script-a-Video](../models/Script-a-Video.md) ⚠️

This work's shot segmentation shows a key dual role that needs to be separated into two levels:
【Data preprocessing level (undisclosed)】How the 500K clips were cut from long videos, whether PySceneDetect or an in-house detector was used, and how the threshold was set — the paper says nothing about this at all. This is a gap on the cleaning side. [Uncertain]
【Annotation level (fully disclosed and the core innovation)】MTSS does not treat shot segmentation as an intermediate byproduct discarded during preprocessing, but instead treats shot structure as a first-class annotation field permanently retained in the Shot stream — each shot carries a precise "time_range," together with two layers of description: visual_description (objectively narrating the core action in chronological order) and camera (professional film language: movement/viewpoint/shot size), plus two relational fields, references_in_shot and active_events. As a result, a single training sample can contain multiple shots internally, rather than being chopped into multiple independent single-shot samples. This is the opposite direction from the mainstream approach of "splitting then keeping only single-shot clips" used by works like UniVerse-1.
【Downstream benefit】Retaining shot structure enables the generation model to directly learn the ability to "execute a shot transition at a specified timestamp." The Shot-Aware Structured Attention introduced in the paper exploits shot boundaries to partition Gemma-3 embeddings, so that each segment of video tokens attends only to the semantics of its own shot, achieving cross-shot context isolation.
【Quantitative verification of splitting precision】The Shot Boundary Deviation metric (the frame-level absolute deviation between the generated video's shot boundary and the boundary specified by the script): LTX-2-AV baseline 3.79 frames → LTX-2-AV-MTSS (only swapping the MTSS prompt, no architecture change) 3.27 frames → Ours w/o ID 0.38 frames → Ours(Full) 1.36 frames. The paper notes that the Full configuration rebounds to 1.36 frames instead, a potential trade-off between identity-injection features and temporal shot precision, attributed to the design of the VLM encoder and the DiT interface, left for future architectural optimization.

### [Seedance 2.0 and Seedance 1.5 pro](../models/Seedance_20_Seedance_15_pro.md) ⚠️

Seedance 1.5/2.0 do not describe this directly. Seedance 1.0's method: uses automated shot boundary detection, analyzing inter-frame visual dissimilarities or using a pretrained detector to identify natural scene changes; the video is then split into short clips of up to 12 seconds each, where each clip may contain one or more temporally coherent shots, in order to preserve local narrative flow while controlling input length. It is not stated whether an open-source tool like PySceneDetect is used; it leans toward being an in-house/internal solution. [Uncertain: whether 1.5 and 2.0 follow the same scheme, and whether audio-side boundary detection was introduced]

### [SkyReels series](../models/SkyReels.md)

【SkyReels-V2】A dual-tool combination: all original videos first undergo shot-boundary detection (explicitly naming PyDetect / PySceneDetect and TransNet-V2), split into single-shot video clips, which serve as the atomic unit for the entire subsequent pipeline. This is standard industry practice (threshold-based method + learning-based TransNet-V2 complementing each other).
【SkyReels-V4】Upgraded to VLM-involved "intelligent segmentation": explicitly points out that traditional scene-cutting methods are insufficient, and instead "combines TransNet's shot boundary predictions via VLM to extract semantically complete segments" — i.e. the VLM judges whether a cut point produces a semantically complete passage, avoiding chopping up one semantic unit or merging two semantic units together. This is the most valuable upgrade of V4 relative to V2 in this entry, and an instance of the 2026 trend of "large models moving down into the data-splitting stage." The TransNet version, the specific VLM model, and the threshold parameters are not disclosed.

### [Sora 2](../models/Sora_2.md) ⚠️

Entirely undisclosed. Neither open-source tools like PySceneDetect nor an in-house shot-aware splitting model is mentioned. Although the model has the ability to generate multiple shots and maintain cross-shot world-state consistency, hinting that shot-level splitting and shot-organization logic exist somewhere in the training pipeline, the method is completely undisclosed. [Uncertain]

### [SpeakerVid-5M](../models/SpeakerVid-5M.md)

Uses PySceneDetect (SceneDetect) as the sole shot-splitting tool, with no in-house splitting model:
【Detection principle】Analyzes changes in image color and brightness to locate "visually significant transitions," cutting at the transition points.
【Length constraint】After splitting, clips are trimmed to 3–14 seconds; those too short or too long are discarded/truncated.
【Coupling with speaker structure】Scene splitting is only the first step; what truly determines the final clip boundaries is the subsequent triple constraint: (1) 3D-Speaker's speaker time interval; (2) YOLO human-tracking's spatiotemporally continuous segment; (3) SyncNet's valid audio-visual binding interval. So SpeakerVid-5M's splitting is essentially a triple "shot-aware + speaker-aware + track-aware" splitting, rather than pure scene splitting.
【Comparison with MOVA】MOVA uses PySceneDetect scene cut points + Silero VAD speech boundaries jointly to generate an 8.05-second fixed-length window, deliberately retaining multi-shot samples; SpeakerVid-5M instead cuts with PySceneDetect and then crops by speaker trajectory, producing strictly single-shot, variable-length clips. Both are shot-aware, but serve different goals: the former serves "learning to do transitions," the latter serves "learning character interaction."
【Code availability】The code for scene detection and speaker separation is provided as step 6 in the open-source repository (marked optional, with some results precomputed).

### [Step-Video-T2V](../models/Step-Video-T2V.md)

PySceneDetect's AdaptiveDetector function (an adaptive-threshold detector, more robust to gradual transitions and camera movement than ContentDetector) detects scene-change points, and FFmpeg is then called to split into single-shot clips at the detected boundaries. After splitting, the first 3 and last 3 frames of each clip are uniformly removed, on the grounds that these frames often contain unstable camera motion (transition residue, shakiness).
Compared with concurrent works: does not use dual-path cross-validation with TransNetV2 (as HunyuanVideo did), and does not additionally train a "transition classifier" for secondary cleaning (as HunyuanVideo 1.5 did) — a relatively standard single-tool approach; but the compensatory design of "removing the first and last 3 frames" is a distinguishing feature.

### [UniTalking](../models/UniTalking.md) ⚠️

The paper does not mention shot segmentation at all: no PySceneDetect, no in-house splitting model, no shot-aware splitting, no transition detection, no post-splitting duration constraint. This is the most conspicuous structural gap in UniTalking's data pipeline relative to comparable works (UniVerse-1, MOVA, Apollo, etc. all treat shot segmentation as an independent funnel stage).
A possible reason for inference: its primary data source, OpenHumanVid, has already completed "decoding, cropping, segmentation" preprocessing upstream, delivering already-split clip units, so UniTalking can directly consume it without building its own splitting module. Whether the internal Huawei data is likewise pre-split cannot be judged — if the internal data is raw long video, the missing splitting stage would be a substantive problem; if it is pre-split, this gap can be explained as "handled upstream." The paper offers no clarification on this. [Uncertain]
An indirect guarantee of shot consistency: LightASD and LipSync in the cross-modal filtering both depend on continuous face tracks, and cross-shot samples would very likely fail to pass, so shot consistency is in effect backstopped by cross-modal filtering, though this is a side effect rather than a design choice.

### [UniVerse-1](../models/UniVerse-1.md)

Uses PySceneDetect (the paper gives the repository address https://github.com/Breakthrough/PySceneDetect in a footnote) to detect scene changes and split at the cut points — the most standard practice in the industry, with no in-house model and no special shot-aware design.
【The only post-hoc constraint】After splitting, clips shorter than 5 seconds are directly discarded, to ensure temporal coherence and meet the downstream 5-second training-window requirement.
【Relationship to speech boundaries】Splitting is entirely driven by visual scene cut points and is not jointly decided with speech boundaries (VAD) — this is considerably coarser than MOVA's approach of jointly using Silero VAD + PySceneDetect dual signals to generate windows and letting the window start adapt to avoid truncating an utterance, meaning UniVerse-1's training windows may start mid-sentence or cut a sentence off.
【Splitting parameters】The detector type used (ContentDetector / AdaptiveDetector) and threshold are not disclosed.
【VGGSound/AudioSet】No shot segmentation is done; only a 5-second minimum-duration constraint is applied.

### [Unison](../models/Unison.md) ⚠️

The paper does not describe shot segmentation at all: no mention of PySceneDetect or any scene-splitting tool, no mention of an in-house splitting model, no mention of shot-aware splitting or any splitting parameters.
【Most likely actual situation】Unison very likely does no shot segmentation at all, instead directly consuming already-split clips from upstream datasets. All five audio-video data sources are released as "pre-split clips" — HDTF, VFHQ, and CelebV-Text are already-cropped face-video clips, VGGSound is fixed-duration audio-visual event clips, and OpenHumanVid is likewise a collection of already-split clips. Under this "aggregating existing clip datasets" construction pattern, shot segmentation has already been completed by each upstream dataset individually, and Unison's pipeline only needs to filter rather than split. This is fundamentally different from works like UniVerse-1 and MOVA, which start from raw long video and must build their own splitting stage.
【Indirect evidence】The average clip duration of about 5.4 seconds matches the typical clip length of upstream datasets; the entire paper contains no shot/scene-change/transition-related vocabulary at all (shot, scene cut, transition).
【Risk】Relying on upstream splitting means the splitting standard is inconsistent across the five data sources (e.g. VGGSound's fixed 10-second splitting does not consider shot boundaries, whereas VFHQ's face-crop clips are strictly single-shot); the paper does not discuss the potential impact of this heterogeneity. [Uncertain]

### [Veo 3 / Veo 3.1](../models/Veo_3_Veo_31.md) ⚠️

[Uncertain] The shot-segmentation method is not disclosed; neither PySceneDetect nor any in-house shot-aware splitting scheme is mentioned. Counter-evidence: the model spontaneously produces shot changes within 8-second generations, suggesting the training samples were not strictly split down to single-shot granularity, or at least retained a fairly high proportion of multi-shot samples.

### [Vidu S1](../models/Vidu_S1.md) ⚠️

Splits original video into single-shot clips along shot boundaries; long shots are further split. The key constraint is that "cut points must not fall in the middle of speech" (speech-aware cutting), a splitting strategy custom-tailored for the lip-sync task. It is not stated whether PySceneDetect, TransNetV2, or an in-house shot-boundary-detection model is specifically used. [Uncertain]

### [Wan 2.5 / 2.6 / 2.7](../models/Wan.md) ⚠️

The entire series (2.1 through 2.7) has never disclosed its shot-segmentation method — neither mentioning open-source tools like PySceneDetect / TransNetV2, nor stating whether an in-house shot-aware splitting model or threshold parameters exist. Only the splitting granularity is confirmed: Wan 2.1's training samples are 5-second clips; Wan-Dancer explicitly splits the original video into 5-second clips with 50% overlap between adjacent clips (the only concrete splitting strategy the same team has written out, though this is for the dance vertical). The multi-shot narrative capability of 2.6/2.7 necessarily requires the training data to carry shot-boundary annotation and cross-shot consistency annotation; its splitting and shot-level alignment scheme is one of the most important unknowns in this entry. [Uncertain]

### [Audio-video generation benchmark collection](../models/av_benchmarks.md) ⚠️

None of the five involve traditional shot segmentation (PySceneDetect-type), since the material itself is already short single-shot clips. The relevant timeline-splitting strategies are:
【AV-SyncBench】Uniformly cut into 0.64-second non-overlapping chunks for segment-by-segment alignment computation of audio-visual embeddings, the basic granularity for its temporal evaluation (corresponding to the native window setting of models like CAV-MAE / Synchformer).
【VABench】The Desync metric only takes two windows — the first 4.8 seconds and the last 4.8 seconds of the generated video — and feeds them into Synchformer to predict the offset, avoiding the instability of long mid-sequence temporal regions.
【PhyAVBench】During recording, splitting is done by "a single physical event" as the unit, with each video corresponding to one complete sound-producing event (e.g. one impact, one instance of flowing water) — a semantic-event-level split, not a shot-level split.
【AVBench / Omni-Judge】No splitting involved [Uncertain].

### [Video caption model ecosystem](../models/caption_models.md)

Captioners themselves typically do not do shot segmentation (splitting is completed upstream), but there are three types of interaction:
【Captioner taking on narrative grouping】CineDance uses Qwen3.5-27B for shot grouping and narrative-boundary determination (bottom-up shot-index grouping is superior to having the LLM directly output timestamps), achieving F1 = 88.4%, with only 3.1% of sequences shorter than the 20-second soft threshold — a representative case of the captioning model directly taking on splitting decisions.
【Captioner describing transitions】ShareCaptioner-Video's DiffSW explicitly identifies scene transitions; MOVA's visual-annotation instructions explicitly require "focusing on video scene transitions"; AVoCaDO's Spatio-temporal & Cinematography dimension covers scene changes.
【Splitting quality in turn determines caption quality】One of Koala-36M's core contributions is precisely "a more accurate and efficient transition-detection method"; its ablation shows: training with Koala-all versus training with Panda-70M improves VBench subject consistency by +1.1% and background consistency by +2.4%, with the paper attributing this temporal-quality improvement to more accurate shot segmentation — the only quantitative evidence in the "splitting → caption consistency → generation quality" chain. CineDance's artifact audit shows a non-compliance rate of 2.8% for CineDance-1M vs. 37.4% for Koala-36M (13.4× improvement).
【Common upstream tool】PySceneDetect remains the industry default; the various in-house transition-detection models (Koala-36M, CineDance) mainly target the miss problem for gradual transitions and fast editing.

### [Geometric / structured annotation dataset collection](../models/geometric_datasets.md)

SceneScribe-1M: A TransNetV2 deep model detects hard cuts and gradual transitions, and only performs splitting on videos judged "discontinuous"; split clips are then re-filtered to ensure semantic coherence. SpatialVID: a modified version of PySceneDetect — adjusting the sensitivity threshold and introducing an interval-based multi-frame comparison strategy to make up for the original's misses on gradual transitions, outputting 3–15 second clips. WildWorld: no shot segmentation needed, continuously recorded by a game engine, samples organized by action-sequence boundaries (training takes an 81-frame window). Action100M: does not use traditional shot detection, instead using V-JEPA 2 semantic embeddings + Ward-linkage temporally constrained hierarchical agglomerative clustering to do semantic-level temporal segmentation, which can produce a cross-shot hierarchical clip tree, discarding clips <0.5 seconds.

### [Video generation post-training data strategy](../models/post_training_data.md) ⚠️

The anchor paper does not touch on this [Uncertain]. The post-training stage generally does not redo shot segmentation — the SFT curated set is a subset filtered from the already-split pretraining pool, so shot segmentation belongs to the responsibility of the upstream pretraining data pipeline. The only horizontally relevant point is that Step-Video-T2V's SFT manual review lists "whether the scene transition is smooth" as one of four criteria — i.e. in the post-training stage, humans manually catch residual splitting problems (a transition mixed into one clip) — one of the few pieces of evidence for "the post-training stage as the last line of defense for splitting quality." Motif-Video 2B's SFT admission criterion includes action=Dynamic, which indirectly excludes static transition remnants.

### [Combined survey of mainstream video pretraining datasets: Panda-70M, InternVid, Koala…](../models/pretraining_datasets.md) ⚠️

**This is the area of clearest technical divergence among the seven, falling into four routes**:
【Route 1 · Directly using PySceneDetect (parameters differ, and the parameter choice itself is the method)】
- **InternVid**: ContentDetector threshold **27**, with no other modification.
- **MiraData**: content-aware threshold **26** — deliberately lowered to **intentionally over-split** ("ensuring that all distinct clips are extracted"), leaving material for later stitching.
- **LVD-2M**: **the most clever parameterization**. Explicitly rejects AdaptiveDetector + rolling mean ("difficult to detect slow shot transitions with fade-in/fade-out"), instead using **ContentDetector, threshold 50, min_scene_len=0 frames, frame sampling rate 0.5fps**. The mechanism: at 0.5fps, adjacent sampled frames are 2 seconds apart, so **any significant change within a 2-second window is judged a cut, which also filters out fade-in/fade-out transitions that typically complete within 2 seconds**. Using sampling rate rather than detector complexity to solve the gradual-transition problem is a low-cost, high-payoff design.
- **UltraVideo**: AdaptiveDetector **two passes + rolling mean** to reduce false cuts caused by camera motion, then **DINOv2 on the first and last 5 frames of each clip** to do feature-similarity checks, catching dissolve transitions missed by AdaptiveDetector.
【Route 2 · PySceneDetect + secondary semantic-feature constraint and stitching (Panda-70M)】
PySceneDetect ContentDetector(threshold 25, min_scene_len=15 frames) → for clips longer than 5 seconds, **recursively cut out the first 5 seconds** (used to handle unedited continuous-shot material; the published code changed 5 seconds to 7 seconds "for better splitting results" — **to precisely reproduce the paper you need to modify lines 199/200 of event_stitching.py**) → take **ImageBind** features at **0.1×n and 0.9×n** of each n-frame clip, retaining only if the **head-tail distance ≤1.0** → adjacent clips are stitched if **‖f(tail of C¹)−f(head of C²)‖≤0.6**. The effect is measured by their own custom **Max Running LPIPS** metric (the maximum LPIPS between 1fps key frames): the caption-alignment method scores 0.408/11.8 seconds, plain PySceneDetect scores **0.247**/4.1 seconds, this method scores **0.256/7.9 seconds** — **i.e. trading a consistency cost of 0.009 for nearly double the clip length**, this paper's most elegant piece of quantitative argument.
【Route 3 · In-house learning-based transition detection (Koala-36M, the only one)】
**Color-Struct SVM (CSS) + temporal 3σ gating**, in two stages:
(a) Training data constructed via self-supervision — **frame pairs from the same video are negatives (no transition), frame pairs from different videos are positives (transition)**; two features: `d_color` = BGR-histogram correlation (frames resized to 256×256, 254 bins per channel, range [1,255]), `d_struct` = SSIM computed on a Canny-enhanced brightness map (E=max(Gray(I), Canny(Gray(I))), grayscale image resized to 128×128, Canny(100,200), data_range=255); a linear SVM, with **the exact fitted coefficients given in the published code**: `svm_score = 4.61480465×bgr_similarity + 3.75211168×canny_similarity − 5.485968377115124`.
(b) Temporal probability gating — the SVM scores undergo **a 3-tap moving average** to get conv_svm; `svm_score<0` is judged a hard cut directly; otherwise, for frame index ≥8 and conv_svm<0.75, **the preceding 8 frames** are used to estimate μ and σ (sorted **with the top and bottom 20% each dropped before computing**, a robust statistic), and a transition is judged when `conv_svm[i] < μ − 3×max(0.2, σ)` (**σ lower-bounded at 0.2** to prevent degeneration). Clips must be longer than 8 frames to be retained, and **the first and last 4 frames are eroded from each end**.
Performance (10,000 manually annotated clips, about half containing transitions): **accuracy 0.7741 / recall 0.9395 / precision 0.7547**, versus PySceneDetect(HSL) 0.4421/0.3096/0.5920, PySceneDetect(HSL+edge) 0.4574/0.4146/0.5854. **Converted F1: 0.838 vs. 0.407 / 0.485.** In speed, due to the fixed downsampling to 256/128 used in feature extraction, it overtakes at high resolution: 1080p 12.26ms vs. PySceneDetect-HSL 26.16ms, 4K 41.98ms vs. 102.55ms (6.4× faster than HSL+edge at 4K), but is **actually slower at 256 resolution** (1.42 vs. 0.68ms).
【Route 4 · Over-splitting followed by multi-model-voting stitching (MiraData, the only one)】
Adjacent short clips are re-merged using a "pairwise, within-group must agree" vote across four models: a vision-language-model group of **Qwen-VL-Chat + LLaVA** (judging "whether it is the same scene"), and an image-feature group of **ImageBind + DINOv2** (judging "whether the features are similar"). The rule, in the paper's own words: "A connection is made only if **both** vision language models **or both** image feature extraction models agree," i.e. (VLM₁∧VLM₂) ∨ (Feat₁∧Feat₂). Design rationale: VLMs are good at recognizing content-coherent transitions, while feature similarity is good at repairing clips that were mistakenly separated. **This is the only pipeline among the seven that treats "stitching" rather than "splitting" as the core action**, and is the direct reason MiraData achieves an average duration of 72 seconds.
【Route 5 · Undisclosed details】OpenVid-1M only states that it uses a "cascaded shot/cut-point detector," with the specific model and thresholds undisclosed. [Uncertain]
## Quality Filtering (Aesthetic Scoring, Clarity, OCR Text Filtering, Black-Border/Watermark/Logo Detection)

`quality_filtering` · Detail level: detailed

### [Allegro](../models/Allegro.md)

Quality filtering is split into three orthogonal dimensions, each with per-stage thresholds (Table 1):
[Brightness] Frames are converted to grayscale to evaluate mean brightness, keeping the range [20, 180] (on a 0–255 scale), consistent across all four stages, used to remove fully black/overexposed/solid-color frames.
[Clarity] DOVER (Disentangled Objective Video Quality Evaluator) video quality score, enabled only in the fine-tuning stage, threshold ≥0.07; no such gate is set in the pretraining stages.
[Semantic/Temporal Consistency] LPIPS inter-frame perceptual distance, enabled only in the fine-tuning stage, threshold ≥0.05 (a lower bound, used to remove samples that are almost completely static/repeated frames).
[Aesthetics] LAION Aesthetics Predictor scores images and video middle frames; this is the only metric that runs through all four stages and is tightened progressively: T2I pretraining ≥4.8 → T2V 360p pretraining ≥4.8 → T2V 720p pretraining ≥5.0 → fine-tuning ≥5.3.
[Text/Watermark/Black Border] CRAFT scene text detector + watermark detector, text-area proportion threshold ≤0.05% (consistent across all four stages, extremely strict, excluding almost any material with subtitles/station logos/decorative text); black borders and watermarks are both categorized as "content-irrelevant artifacts" and removed together.
[Image-Text Consistency] CLIP visual–text cosine similarity, ≥0.17 for 360p pretraining/T2I stages, ≥0.20 for 720p pretraining and fine-tuning stages.
Appendix A Fig. 11 shows the aesthetic score distribution shifting rightward overall as stages progress (360p pretraining → 720p pretraining → fine-tuning), validating the practical effect of the tiered tightening.

### [Apollo](../models/Apollo.md) ⚠️

Two tracks — video side and audio side — with clearly enumerated dimensions but almost all thresholds missing:
[Video-side four-dimensional quality modeling]
- Dynamic quality: subject motion ratio, camera stability
- Static quality: sharpness, aesthetics, color saturation
- Content naturalness: no excessive effects/watermarks
- Safety
[Video-side hard exclusion conditions] Low resolution, low SNR/MOS, silence ratio exceeding 20% (original text: "We discard those videos with low resolution, low SNR/MOS, or over 20% silence"). Note that SNR/MOS, which are properly audio metrics, appear here in the video filtering clause, indicating that audio and video quality are jointly considered at this first gate.
[The only publicly disclosed numeric threshold] Silence ratio < 20% (silence ratio < 0.2). The scoring models used for each video quality dimension (whether LAION-Aesthetic, DOVER, MUSIQ, etc. are used) and the threshold values are not given; the paper's evaluation section uses MANIQA, aesthetic-predictor-v2-5, and Musiq to compute aesthetic scores, but those are evaluation metrics, not data filters, and should not be conflated.
[Watermark/logo] Only mentioned via the single phrase "no excessive effects/watermarks" under the content-naturalness dimension, without specifying the detection method (whether OCR or a dedicated watermark-detection model), nor whether the outcome is removal or cropping.
[OCR/burned-in subtitles] Not mentioned at all, unlike MOVA's approach of making "presence of burned-in subtitles" an explicit, controllable attribute. [Uncertain] (dimensions are clear, but thresholds and methods are missing)

### [CineDance / CineDance-1M](../models/CineDance.md) ⚠️

Overall it adopts a hybrid strategy of "full-set scoring + metadata retention + hard rules used only for artifacts," a notable departure from the "single-threshold cutoff" approach used by most datasets:
[Hard exclusion (rule-based)]
  - Burned-in subtitles: detected and cropped out by EasyOCR, re-verified again after shot segmentation;
  - Black borders/letterboxing: FFmpeg black-border detection, likewise applied in two passes (coarse crop + clip-level fine verification);
  - Opening/closing credits/non-main-content: MLLM-guided temporal truncation, with truncation amount t = max(5min, 0.1L).
[Soft scoring (metadata-only, no hard cropping)]
  - DOVER: split into Aesthetic Quality and Technical Score (clarity/technical quality) components;
  - AMT: Motion Smoothness;
  - Audio-side DNSMOS and CLAP temporal variance;
  - Alignment-side ImageBind and SyncNet.
  The paper explicitly states "we store all quality scores as metadata, enabling users to flexibly construct task-specific subsets" — handing the thresholding decision to downstream users, so the paper itself does not publish any specific aesthetic-score/technical-score cutoff values.
[Manual artifact audit (final verification)] A random sample of 500 clips was independently reviewed by three annotators for residual artifacts, with disagreements resolved by joint deliberation. The target artifact checklist for review: burned-in subtitles, station logos, letterbox black borders, watermarks, TV-network overlays, title cards, end-credit text, screen recordings, transition effects, and freeze frames. Result: non-compliance rate of 2.8%, versus 37.4% for Koala-36M, a 13.4x improvement.
[Automated detection methods for watermarks/logos] The paper does not state whether there is an automated watermark/logo detection model; from the description, this relies mainly on upstream dataset cleaning plus final manual spot-checking. [Uncertain]

### [CogVideoX](../models/CogVideoX.md)

Quality filtering consists of two tracks: "Video-LLaMA classifiers + two continuous scores":
[Classifier track] Of the 6 Video-LLaMA binary classifiers, 4 directly serve image quality and content purity; Appendix K Table 14 gives their performance on the test set (a randomly annotated 10% of the data):
· Classifier-Low Quality (poor shooting quality, blurry footage, severe camera shake): TP 0.80 / FP 0.02 / TN 0.09 / FN 0.09, Test Acc 0.89 (lowest of the 6)
· Classifier-Editing (manual re-editing and effects): TP 0.81 / FP 0.02 / TN 0.09 / FN 0.08, Acc 0.91
· Classifier-Text (text-dominated, functionally a discriminative substitute for OCR text filtering): TP 0.60 / FP 0.03 / TN 0.36 / FN 0.02, Acc 0.96
· Classifier-Screenshot (phone/computer screen recording): TP 0.61 / FP 0.01 / TN 0.37 / FN 0.01, Acc 0.98
· Classifier-Lecture (talking-head lecture): Acc 0.99 (highest of the 6)
· Classifier-Static (incoherent motion): Acc 0.92
[Continuous-score track] An image aesthetic score is computed over the full video set; the hyperparameter table gives a lower bound of Lowest aesthetic-value = 4.5; the threshold is dynamically adjusted across stages during training. The 2B images in the image branch are likewise filtered by aesthetic score.
[Final stage] The 20% high-quality subset used for fine-tuning specifically addresses three categories of dirty data left over in the pretraining data — "subtitles, watermarks, low-bitrate" — with the paper reporting that this step "effectively removed subtitles and watermarks from generation results, and slightly improved visual quality."
Worth noting: the paper does not describe black-border detection, logo detection, or a standalone OCR model in the style of Movie Gen — text filtering is done end-to-end via the Video-LLaMA classifier, an early example of "using a multimodal large model in place of a traditional shallow detector."

### [Cosmos-Predict2.5](../models/Cosmos-Predict25.md) ⚠️

Quality filtering is the heaviest stage of the seven-stage funnel, composed of six independent filters run in series, covering the aesthetic, technical, and semantic levels; the paper explicitly states the ordering principle (cheap filters first, expensive ones last):
(1) Aesthetic scoring filter — scores and grades inputs by aesthetic quality; the scorer model and threshold are not disclosed.
(2) OCR filter — "attempts to remove clips with excessive text overlay," removing clips with excessive text overlay (subtitles, pasted-on text, station logos, etc.); the detector and text-area threshold are not disclosed.
(3) Perceptual quality filter — the paper explicitly states it is "akin to DOVER (Wu et al., 2023)," i.e., adopting a DOVER-style disentangled video quality assessment (DOVER splits VQA into technical and aesthetic branches), aiming to remove "technical distortions and perceptual artifacts" such as compression blocking, noise, and blur.
(4) Semantic artifacts filter — explicitly "akin to VTSS (Wang et al., 2025)," aiming to remove structural/semantic-level anomalies: video-in-video (picture-in-picture, screen-recording-within-screen-recording), poor transitions, etc. This is a stage newly added relative to Predict1, specifically targeting samples that are "technically normal but structurally unsuitable for training."
(5) VLM filter — uses a Qwen2.5-VL-family vision-language model "with higher precision" to remove a set of undesirable issues, acting as a final review; placed last due to its computational expense.
(6) Content type classifier + physical-realism pruning — an internally trained content-type classifier assigns labels from a 26-class taxonomy, and this stage also removes "physically unrealistic phenomena": video-game footage, synthetic visual patterns, animation, and cartoons, in order to maintain alignment with the distribution of the real physical world. This is a special quality standard rooted in the Physical AI positioning — "not physically realistic" is itself defined as a quality defect.
Black borders and spatial padding are not handled by filtering but are directly removed by cropping in the third stage (repair rather than discard). Watermark/logo detection is not listed separately (it may be folded into the OCR or VLM filter).
Threshold values for all filters are uniformly undisclosed; the domain-specific branch explicitly uses "a domain-specific subset of filters with adjusted hyperparameter values," indicating thresholds are tunable per domain. [Uncertain: the specific model versions and threshold values for each filter]

### [Data-Juicer 2.0](../models/Data-Juicer.md) ⚠️

Quality filtering is the most densely covered area among DJ's video operators, with 16 video-side Filters in total, which can be grouped by dimension as follows:
[Aesthetics and perceptual quality]
  · video_aesthetics_filter — computes an aesthetic score on specified sampled frames and filters by range, built on LAION/improved-aesthetic-predictor-family aesthetic scoring models, supporting configurable frame-sampling strategies (uniform sampling/keyframes) and any/all aggregation logic (pass if any frame qualifies vs. pass only if all frames qualify). This is one of the three candidate operators in the Sandbox probing experiments.
[Basic specifications]
  · video_resolution_filter — resolution upper/lower bounds.
  · video_aspect_ratio_filter — aspect-ratio range.
  · video_duration_filter — duration range.
[Text and overlays]
  · video_ocr_area_ratio_filter — "detects the text-area ratio of specified frames and filters by range"; the documentation explicitly notes that "videos with a low OCR text-area ratio tend to be higher quality," used to remove burned-in subtitles, danmaku (bullet comments), slideshow-style content, and text/poster-style samples. This is a standard step in video generation data cleaning.
  · video_watermark_filter — "keeps videos that are likely watermark-free," filtering by score from a watermark-detection model.
  · video_remove_watermark_mapper — a repair path complementary to the previous one: given a list of watermark-region coordinates (x1,y1,x2,y2 format), the watermark is directly erased, salvaging material that is watermarked but otherwise high quality, rather than discarding the whole clip.
[Content safety]
  · video_nsfw_filter — NSFW score filtering (see safety_filtering for details).
[Semantic alignment]
  · video_frames_text_similarity_filter — CLIP similarity filtering between sampled frames and text (see the related entries under model_as_data_judge and av_sync_detection).
[Person-related]
  · video_face_ratio_filter — filters by the range of the proportion of the frame occupied by a face, usable for distinguishing close-up/medium/wide shots, or for removing extreme samples with no people or a screen full of faces.
  · video_tagging_from_frames_filter — keeps or removes clips by content tag.
[Motion]
  · video_motion_score_filter / video_motion_score_raft_filter / video_motion_score_ptlflow_filter (see motion_filtering for details).
[Design-feature evaluation]
  1. "Filter + repair" dual paths: for watermarks, resolution, and aspect ratio, both a Filter (discard) and a Mapper (rewrite/repair) operator are provided in parallel, letting users choose a strategy based on data scarcity. This is relatively rare among comparable frameworks.
  2. All thresholds are externalized as YAML parameters, with no hard-coded default policy: DJ does not decide "how high the aesthetic-score cutoff should be" on the user's behalf, but instead relies on model feedback from Sandbox probing experiments to determine it. This is a methodological stance — thresholds should be searched for, not prescribed.
  3. [Uncertain] Not built in: black-border detection (letterbox/pillarbox), dedicated blur/clarity scoring (e.g., Laplacian variance, MUSIQ/DOVER-style video quality assessment models), encoding-bitrate filtering, and several other items common in model-team pipelines; nor is there a multi-metric weighted composite scoring mechanism — all filters are single-metric independent thresholds stacked in series.

### [Foley-Omni](../models/Foley-Omni.md) ⚠️

Quality filtering is the first stage of the cleaning funnel, using parallel visual and audio channels as gatekeepers, all as threshold-based hard filters with no weighted composite scoring.
[Visual quality]
  · Resolution ≥480p — removes low-clarity clips, ensuring reliable CLIP and Synchformer visual feature extraction. The 480p bar is relatively lenient (compared to the 720p/1080p bars common in text-to-video models), which is reasonable since video here only serves as a condition rather than the generation target.
  · Bitrate ≥1 Mbps — a second safeguard alongside resolution, specifically targeting samples with "nominally high resolution but heavy compression and actual loss of detail." This is a less common but very practical rule, since upsampled low-quality video can pass the resolution check but fail the bitrate check.
  · Motion score ∈ [0.1, 3.2] — a two-sided threshold. The lower bound removes near-static footage (lacking visual events, unable to provide temporal cues for sound-effect generation), and the upper bound removes overly violent motion (unreliable motion estimation, confused audio-visual correspondence). The paper does not specify the exact computation method for the motion score (mean optical flow / frame difference).
[Audio quality]
  · Meta AudioBox Aesthetics score ≥0.6 — uses Meta's open-source AudioBox Aesthetics reference-free audio aesthetic/quality assessment model. This single metric simultaneously serves multiple roles: naturally filtering out silent clips (content-free audio scores very low aesthetically), and removing tracks with heavy background noise/clipping distortion/low sample rate/encoding degradation. Compared to traditional SNR thresholds, AudioBox Aesthetics is a learned perceptual quality assessment closer to human auditory judgment, representing a new paradigm in audio data cleaning for 2025–2026.
[Common items not covered] [Uncertain] The paper does not mention aesthetic scoring (e.g., LAION-Aesthetics), OCR text filtering, black-border detection, watermark/logo detection, burned-in subtitle detection, or other standard steps in text-to-video cleaning. For this task these are indeed lower priority (the visual channel only needs to support semantic and synchronization judgment; whether the picture is aesthetically pleasing doesn't affect audio generation), but watermarks/subtitles occluding the mouth region could affect VisualTTS lip-sync learning — a risk the paper does not discuss.

### [Goku](../models/Goku.md)

Multi-dimensional threshold filtering, whose most notable feature is that **all thresholds are tiered by target resolution, with high-resolution tiers tightened across the board** (paper Table 4):
[Basic technical attributes (hard gates)]
  - Duration ≥ 4 seconds;
  - Resolution min{height, width} ≥ 480;
  - Bitrate ≥ 500 kbps (removes highly compressed material with severe blocking artifacts);
  - Frame rate ≥ 24 FPS (Film Standard) or 23.976 FPS (NTSC Standard), removing low-frame-rate/frame-dropped material.
[Aesthetic Score] An aesthetic model scores key frames:
  - 480p tier: score < 4.3 discarded;
  - 720p and above tier: score < 4.5 discarded.
  (The paper does not specify the exact aesthetic-model version; presumably a LAION-Aesthetic-family model or an in-house variant.)
[OCR text-coverage filter] Detects the proportion of the frame area occupied by text in key frames, discarding samples above the threshold (mainly removing subtitles, pasted-on ads, decorative text, and screen-recorded content):
  - 480p tier: text area ≤ 0.02 (2%);
  - 720p and above tier: text area ≤ 0.01 (1%).
  This threshold is extremely strict, indicating the team places high priority on preventing the model from learning to generate garbled text.
[Deduplication (linked to quality)] Same-source clips are compared via perceptual hashing; when hashes are close, **the one with the higher aesthetic score is kept** — deduplication decisions are directly arbitrated by the quality score.
[Common items not mentioned] The paper does not describe black-border detection, watermark/logo detection, blur/clarity (e.g., Laplacian variance, NIQE), abnormal brightness/saturation, or temporal flicker consistency filtering, nor does it mention any independent filtering standard for the image side (T2I data) beyond the aesthetic score. Compared with work from late 2025 to 2026, Goku's quality-filtering dimensions skew classical and shallow.

### [Hailuo / MiniMax Video](../models/Hailuo.md) ⚠️

[Uncertain] (essentially no disclosure). No aesthetic scorer, clarity/technical-quality scorer, OCR text detection, black-border cropping, watermark/logo detection, or any other specific method is mentioned, and no threshold is given at all. The only relevant official wording is Hailuo 02's "improved quality and diversity" and Hailuo 2.3's capability description, both of which are outcome claims rather than method disclosures.
Indirect evidence inferable from model behavior (speculative, not disclosed): Hailuo-series outputs perform relatively well in terms of frame cleanliness and cinematic feel, with subtitle/logo residue rarely appearing, suggesting effective frame-cleanliness and watermark/subtitle filtering exists; "native 1080p" suggests a resolution and clarity gate exists.

### [HunyuanVideo-Foley](../models/HunyuanVideo-Foley.md) ⚠️

This work's quality filtering shows extreme asymmetry — multiple strict layers on the audio side, and complete pass-through on the visual side:
[Audio quality filtering (three layers)]
1) Effective sample-rate / bandwidth detection: only samples with an effective sample rate >32 kHz are kept. Note this is the "effective" (as opposed to file-declared) value, i.e., spectral analysis confirms actual energy in the high-frequency band, allowing identification and removal of "fake HD" audio sources upsampled from low-bitrate sources. This is a fairly careful rule and corresponds directly to the model's 48 kHz output spec — if the training data lacks sufficient bandwidth, the model cannot learn to generate high-frequency detail.
2) AudioBox-Aesthetics audio aesthetic assessment: uses Meta's audiobox-aesthetics toolkit for perceptual quality scoring of audio. This tool outputs four dimensions: PQ (Production Quality), PC (Production Complexity), CE (Content Enjoyment), CU (Content Usefulness). The paper does not state which of these dimensions were used or what their thresholds were. As a point of comparison, MOVA explicitly gives a three-score threshold combination of PQ>5.0 / CU>4.5 / CE>2.5. [Uncertain]
3) SNR (signal-to-noise ratio): used as a "supplementary metric" to assist judgment, with no threshold given. [Uncertain]
[Visual quality filtering: none at all] There is no video-quality-related criterion anywhere in the entire pipeline — no resolution floor, no bitrate constraint, no aesthetic score (no DOVER, no LAION-Aesthetic), no clarity/blur detection, no black-border detection, no watermark/logo detection, no OCR subtitle filtering.
[Rationality analysis of this tradeoff] Seemingly aggressive, but actually consistent with the nature of the task: since video in this model serves only as conditioning input, encoded via SigLIP-2 into semantic features, factors like frame clarity, watermarks, and black borders barely affect the semantic judgment of "what acoustic event is happening in this frame." Spending compute on visual quality filtering has extremely low marginal benefit for output audio quality. By contrast, joint-generation models like UniVerse-1 must apply strict visual filtering (because they must generate the picture) — the difference between the two is a difference in task form, not engineering rigor.
[A noteworthy reverse risk] Zero filtering on the visual side means training data may include a large number of samples with extremely poor picture quality (heavy compression, extremely low resolution, strong watermark occlusion); the visual-semantic recognition of such samples is itself unreliable and could introduce noisy supervision into the model. An ideal approach would retain a low-cost visual-semantic-recognizability criterion (e.g., confidence or entropy of SigLIP features), but the paper does not mention any such design. [Uncertain]
[AudioBox's dual role] Worth noting is that AudioBox-Aesthetics is used both as a training-data filter and as a final evaluation metric (the PQ, PC, CE, CU scores in the main table). This constitutes a degree of circularity: using a metric to filter data, and then using the same metric to evaluate performance, will systematically overestimate the model's performance on that metric. The paper does not discuss this potential evaluation bias.

### [HunyuanVideo](../models/HunyuanVideo.md)

[Original version] Multiple parallel tracks, tightened progressively:
· Aesthetics and technical quality: Dover model dual-view scoring (aesthetic view + technical view);
· Clarity: an in-house, training-based blur-detection model removes visually blurry content; the Laplacian operator is used to select a clear starting frame;
· Text: an OCR model removes clips with excessive text coverage, and clips containing subtitles are cropped rather than discarded entirely;
· Watermark/black border/logo/sensitive information: a YOLOX-like detection model locates and removes or crops them out;
· Specific threshold values for each filter are not disclosed in the paper, which only states that "different training tiers use thresholds of different strictness."
[1.5] Restructured into "one composite quality assessment operator + one aesthetic operator + a set of basic rules":
· The composite quality operator has four dimensions: sharpness, detail retention, noise & artifacts, dynamic range — more finely subdivided than the single blur-detector of the original version, notably adding two "production quality"-leaning dimensions: noise/artifacts and dynamic range;
· Aesthetic operator: removes low-aesthetic-score samples;
· Basic rules: padding black borders, stitching seams, and grid collages are removed outright;
· Subtitles/logo/watermark: changed to spatial cropping, discarded only if the remaining area after cropping is <60%.
Neither generation discloses any threshold values.

### [InstructAV2AV](../models/InstructAV2AV.md) ⚠️

Quality filtering is distributed at the beginning and end of the pipeline, taking two forms — "source-material admission filtering" and "synthesized-output acceptance filtering" — the latter being a stage unique to editing-type datasets.

[A. Source-material admission filtering (Stage 1, applied to real material)]
  · Visual aesthetics: LAION Aesthetics Predictor (Romain and Christoph, 2022) removes visually low-quality clips. This is a standard tool in text-to-video cleaning, reused here. [Uncertain] The threshold value is not given (this model's output is typically on a 1–10 scale, with common industry thresholds in the 4.5–5.5 range).
  · Motion intensity: CoTracker3 grid-point tracking computes average motion magnitude, and clips below a preset threshold are discarded. [Uncertain] The threshold is not given.
  · Audio quality: Audiobox-Aesthetics (Tjandra et al., 2025, Meta's reference-free audio aesthetic assessment model) removes low-quality audio. [Uncertain] The threshold is not given. Note: Foley-Omni gave an explicit threshold of ≥0.6 when using the same model; this work does not.
  · Silence: PyDub removes clips below −45 dBFS. This is the only exact numeric threshold given in the entire paper.
  · Sound-source clarity: in non-speech clips, samples with ambiguous sound sources are removed, requiring each clip to have a single dominant sound source. This is a fairly distinctive filtering condition, serving the downstream need for "separable, directionally editable" audio.
  · Audio-visual attribution: speech clips are required to have speech temporally aligned with a visible on-screen speaker, effectively removing voice-over, narration, and post-dubbed audio.

[B. Synthesized-output acceptance filtering (Stage 3, applied to generated targets)]
  A multimodal LLM (Qwen3-Omni) scores along five dimensions, all five of which must be passed simultaneously:
  (i) instruction fidelity — whether the edit required by the instruction was strictly executed;
  (ii) content preservation — whether non-target regions/ambient sound remain untouched;
  (iii) perceptual quality — whether the synthesized result is realistic and free of visible artifacts;
  (iv) audio-video synchronization — whether the new audio track and the new picture are aligned;
  (v) safety — see safety_filtering.
  These five dimensions are not generic quality scores but correspond one-to-one to the four failure modes of editing tasks (what should have changed didn't / what shouldn't have changed did / the change looks unrealistic / the change is out of sync), a design well matched to the task itself, and echoing the 11 evaluation metrics used at the evaluation stage (TV-A/TA-A correspond to (i), SSIM/TC to (ii), FVD/FAD/LPAPS to (iii), AV-A/PEAVS/Sync-C/Sync-D to (iv)) — a consistent design of "evaluate the model the same way you filter the data."

[Common items not covered] [Uncertain] No mention of OCR text/subtitle filtering, black-border detection, watermark/logo detection, clarity (blur) detection, or compression-bitrate gates. The absence of subtitle filtering is particularly notable: the source material includes a large amount of film/TV footage, and burned-in subtitles cause obvious visible artifacts in editing tasks (when the edited region overlaps with subtitles, mask-guided synthesis will corrupt the text) — a risk the paper does not discuss.

### [2026 Miscellaneous Audio-Video Joint Generation Works](../models/JAVG_2026_misc.md) ⚠️

[ALIVE — four operator categories + an original six-tier clarity system (the most systematic in this batch)]
(1) OCR text and watermark detection: original text: "We employ Optical Character Recognition (OCR) to measure the proportion of overlaid text and detect watermarks" — note that its approach is to "measure the proportion of overlaid text" rather than a simple binary judgment, indicating thresholding by text-coverage-area ratio (threshold undisclosed [Uncertain]).
(2) Aesthetic scoring: "utilize a pre-trained aesthetics model for per-frame scoring" — per-frame rather than sampled-frame scoring, a finer granularity. The specific model and threshold are undisclosed [Uncertain].
(3) Motion analysis: RAFT computes optical flow (see motion_filtering).
(4) 13-dimensional low-quality classifier: "a quality model ... is used to classify samples as either 'high-quality' or belonging to one of 13 distinct low-quality dimensions" — a multi-label classifier that subdivides low quality into 13 dimensions. This is far more fine-grained than the common "single quality score + threshold" approach, and can support differentiated handling by dimension (e.g., some dimensions may be repairable, others must be discarded). But the paper never lists what these 13 dimensions actually are [Uncertain], the most regrettable omission in the entire paper.
(5) Independent Clarity filtering stage: judged against "reference images across six distinct clarity levels" (six clarity-tier reference images, from blurry to sharp) — i.e., turning clarity into a six-level ordinal label rather than a continuous score or binary judgment. The 0.7M "high-clarity samples" dedicated to the 1080p Refiner stage are drawn from the highest tier. This "reference-image-anchored ordinal grading" is rare among contemporaneous work; its advantage is high annotation consistency and cross-annotator comparability.
[NAVA — multiple parallel operators + VLM re-screening]
(1) PaddleOCR performs OCR filtering and subtitle erasure — notably, this is not simply "discard on subtitle detection" but rather performs "subtitle removal," repairing subtitled material and continuing to use it, significantly improving data utilization. This contrasts with ALIVE's approach of "measure text proportion, then filter": one repairs, one removes.
(2) Visual quality operator group: aesthetics, sharpness, brightness, motion score — four operators scoring in parallel. All thresholds undisclosed [Uncertain].
(3) VLM-based filtering and tagging: a vision-language model performs a secondary screening of clips that are "visually clear in quality" — i.e., adding large-model semantic judgment on top of traditional scorers (see model_as_data_judge for details).
[OmniCustom — a single aesthetic threshold (the only work in this batch to give a numeric aesthetic threshold)] Explicitly removes videos with "aesthetics scores below 0.3" — an aesthetic score <0.3 is eliminated. This is the only publicly given aesthetic-threshold number in this batch of work, though its score scale (0–1 normalized) and the specific aesthetic model are unspecified [Uncertain]. Also, videos shorter than 10 seconds are removed. No description of OCR / watermark / resolution filtering [Uncertain].
[StreamChar / CCL / Baton] No visual quality filtering described [Uncertain]. All three rely on the quality screening already built into the upstream public datasets they use (OpenHumanVid, SpeakerVid-5M, and TalkVid all come with their own quality filtering), i.e., "quality control outsourced."
[ITS-JAVG (inference-time quality gatekeeping)] In its verifier combination, VQAScore and VideoReward-TA are responsible for gatekeeping visual quality and text consistency, which can be seen as "inference-time quality filtering." The paper's key finding is that any single quality verifier will be exploited by the search algorithm ("the inference-time search algorithm exploits blind spots"), requiring a combination of multiple verifiers — this conclusion is also a cautionary point for training-side data filtering: relying on a single aesthetic score or a single quality model as the sole filtering criterion likewise has blind spots that can be "exploited in reverse" by the data distribution.

### [Audio-Video Joint Generation Baseline Collection](../models/JavisDiT_baselines.md) ⚠️

Only JavisDiT++ has systematic visual quality filtering; the rest is largely blank:
[JavisDiT++ (the only one in this collection)] Explicitly states it "follows the Open-Sora methodology" for three quality filters:
(1) Aesthetic scoring — removes clips with poor picture quality;
(2) Optical flow/motion scoring — see motion_filtering;
(3) OCR scoring — detects and removes clips with excessive on-screen text (subtitles, danmaku, title cards, etc.), preventing the model from learning to generate garbled text.
Specific thresholds for all three are undisclosed [Uncertain], and the amount eliminated by each is not broken out [Uncertain].
In addition, data.md lists two engineering-level cleanup steps: removing broken videos, and filtering samples with fewer than 10 frames.
[Common methods not mentioned] Even JavisDiT++ does not describe watermark detection, logo detection, black-border detection and removal, compression-artifact detection, blur detection, or brightness/contrast filtering [Uncertain].
[MM-Diffusion] When building its own Landscape dataset, it describes the dataset as "high-fidelity," but does not describe any automated quality-filtering pipeline; the small scale of 1,000 clips suggests manual curation may have occurred, but the paper does not state this [Uncertain].
[AV-DiT] No quality filtering; public datasets are used directly.
[Harmony] Visual quality filtering not mentioned [Uncertain]; the only screening is an "audio-video consistency scoring model," which is a cross-modal alignment filter rather than a visual quality filter.
[UniAVGen] Not mentioned at all [Uncertain].
[JavisBench evaluation set] Mentions a two-stage process of "pre-filtering to ensure quality and remove noisy candidates" and "post-filtering to ensure diversity," but the specific tools and thresholds used are undisclosed [Uncertain].

### [Kling 3.0 / Kling Video 3.0 Omni](../models/Kling_30_Omni.md) ⚠️

Multi-layer composite filtering: (1) Hard admission — resolution and duration thresholds, audio/video file corruption detection (Kling-Omni base level); on the Kling-Foley side, source video is required to be ≥720P. (2) Temporal-visual quality scoring — identification and removal of blur, jitter, and compression noise (Kling-Omni second level). (3) The same team's public Koala-36M work gives finer detail: instead of a single-metric hard threshold, they train a "training-suitability assessment network" that outputs a unified VTSS (Video Training Suitability Score), with inputs including sub-metrics such as clarity score, aesthetic score, and motion score; the network has a dynamic branch (3D Swin Transformer) + static branch (ConvNeXt) + feature/label branch, fused via a Weight Cross-Gating Block, with a threshold of 2.5 (determined based on the score's bimodal distribution); its core motivation is to avoid mistakenly discarding high-quality data due to a single-metric hard threshold. (4) Subtitles/text: the Kling-Foley data specification calls for "as few subtitles as possible." [Uncertain: whether Kling 3.0 Omni has a dedicated black-border/watermark/logo detection or OCR text-filtering module — not mentioned in the report]

### [LTX-2](../models/LTX-2.md) ⚠️

Video-side quality filtering is the most finely disclosed component in the LTX series, centered on an in-house paired-preference aesthetic scoring model:
(1) Annotation-pair sampling: a multi-labeling network first tags millions of samples, and pairs are sampled only among samples that "share at least one of the top-3 labels" — the goal being to make comparisons happen within similar content, minimizing the distribution shift introduced by aesthetics-based filtering. This design is the most worth borrowing from in this pipeline.
(2) Manual annotation: tens of thousands of image pairs are manually labeled for which one is more aesthetically superior.
(3) Model training: these preference pairs are used to train a Siamese Network that learns an aesthetic score preserving the labeled ordering (a ranking-style rather than an absolute-regression-style score), capable of evaluating both video and images.
(4) Application: an aesthetic score is computed for each sample, and those below the threshold are removed (the threshold value is not disclosed); the fine-tuning stage further keeps only the subset with the highest aesthetic scores.
(5) Geometric cleaning: explicit Crop Black Bars removes black borders.
[Not mentioned] OCR/subtitle text filtering, watermark and logo detection, clarity/blur criteria, and compression-artifact detection are all without any description. [Uncertain]

### [LongCat-Video](../models/LongCat-Video.md) ⚠️

Quality signals are fully annotated as metadata and then screened by threshold, with fairly complete coverage:
(1) Aesthetic score — the scoring model is not disclosed;
(2) Blur/clarity score — determines picture clarity and compression damage;
(3) Text coverage — detects the proportion of text/subtitles on screen, used to remove samples with excessive subtitles or heavy pasted-on text (corresponding to common OCR filtering);
(4) Watermark detection — removes clips with watermarks/station logos/logos;
(5) Black-border handling — not filtering but repair, cropped out directly during FFMPEG segmentation;
(6) Encoding quality — resolution, frame rate, and bitrate metadata can be used to remove low-resolution, low-bitrate degraded sources.
The curated set for the SFT stage is further re-screened on this basis by multiple metrics: "aesthetic score, video quality, motion quality."
Avatar 1.5's visual-quality estimation specifically targets three types of problematic content: "low-resolution, heavily compressed, subtitle-heavy," and the online stage additionally applies resolution, brightness-distribution, and black/white-pixel-ratio checks as well as frame-jump detection.
All specific threshold values are undisclosed. [Uncertain: thresholds for each quality metric]

### [MOVA](../models/MOVA.md)

Split into two layers — "Stage 2 general three-dimensional quality assessment" and "Phase 2 pre-training secondary specialized filtering" — with all thresholds publicly given (Appendix Table 9):
[Stage 2 audio quality]
- Silence Ratio < 0.8
- Bandwidth > 1,000 Hz
- Audiobox-aesthetics PQ (Production Quality) > 5.0
- Audiobox-aesthetics CU (Content Usefulness) > 4.5
- Audiobox-aesthetics CE (Content Enjoyment) > 2.5
[Stage 2 video quality] Uses the DOVER video quality assessment tool, scoring from both technical and aesthetic perspectives:
- DOVER-Aesthetic > 0.85
- DOVER-Technical > 0.05
[Threshold determination method] Not chosen arbitrarily or borrowed from the literature, but by "manually inspecting videos retained under different metric cutoffs and setting a reasonable threshold for each dimension," ensuring corpus quality while retaining sufficient diversity.
[Phase 2 secondary specialized filtering (three orthogonal filters)]
1) OCR subtitle filtering: OCR is used to detect burned-in subtitles, keeping about 9.5M subtitle-free clips, and appending the sentence "This video has no subtitles." to the end of the prompt — note that this is not a simple removal but rather makes "presence/absence of subtitles" an explicit controllable attribute taught to the model, a hybrid "filter + conditioning" approach.
2) Lip-audio correspondence filtering: keeps videos with LSE-D ≤ 9.5 and LSE-C ≥ 4.5, yielding a high-quality lip-audio correspondence subset of about 2.5M clips.
3) Visual fidelity filtering: DOVER technical score > 0.15, yielding about 2.4M clips.
After merging, the Phase 2 dataset is 16.8M clips / ~37,600 hours.
[Phase 3] Uses only the highest-quality 720p subset, with DOVER technical score > 0.14 (this threshold is re-calibrated for 720p).
[Black border/watermark/logo] Black borders are detected and cropped out via FFmpeg cropdetect during preprocessing and then repadded; watermarks and logos have no dedicated detector, but the visual-annotation prompt explicitly requires "Ignore all text, subtitle and watermark in the video," i.e., excluding watermarks/subtitles from the described content at the annotation level, avoiding the model learning to treat watermarks as describable elements.

### [Merged entry: ① Mochi 1](../models/Mochi_MAGI_Motif.md) ⚠️

① Mochi 1: only NSFW filtering is known to have been applied; the rest of the quality-filtering strategy, tools, and thresholds are entirely undisclosed. [Uncertain]
② MAGI-1 (11-class Filter Actor, methods detailed, thresholds all left blank):
· Video quality assessment: uses DOVER, whose output includes overall/aesthetic/technical scores; the team found empirically that "the technical score alone is the most effective metric," so only the technical score is used.
· Aesthetics: LAION aesthetic predictor; since this model is designed for images, the team uses the "aesthetic score of the first frame" to represent the quality of the whole clip (note this differs from Allegro's choice of using the middle frame).
· Over/under-exposure: each frame is converted to HSI color space, and the average brightness across the whole clip is computed, used to remove over- or under-exposed samples; the paper states such data will "adversely affect training stability" — attributing the motivation for exposure filtering to training stability rather than visual appearance, a notable framing.
· Border detection: frame-by-frame edge detection + Hough transform identify horizontal/vertical straight lines that persist across frames, treating these lines as candidate borders, with "the proportion of frames containing a border" used as the filtering confidence.
· Text detection: a clip is discarded entirely if any frame has too many characters or the text area occupies too large a proportion of the frame. Subtitles are handled as a special exception — subtitle characters are few and occupy a small area, making them hard to catch with the above rules, but they have a distinctive spatiotemporal pattern (fixed at the top or bottom of the frame, persisting across many consecutive frames), which is used to specifically identify and remove them.
· Logo detection: Florence-2 (which supports open-vocabulary object detection) is used, given a preset keyword set, to detect and locate logos in the frame and output a confidence score for filtering.
· Corner-face detection: in commentary-style videos the presenter often appears fixed in one corner of the frame; the team uses a face detection model, combining face position and detection confidence, averaging the confidence of a fixed-corner face across frames to estimate the likelihood of "a presenter being present" and remove such clips.
All of the above filters' thresholds are described in the report only as a "predefined threshold" without giving numeric values.
③ Motif-Video 2B (5 orthogonal quality signals, tools and some parameters named):
· Aesthetics: Aesthetic Predictor V2.5 (a SigLIP-based image-level aesthetic predictor); frames are uniformly sampled over time for each video, frame-level scores are computed, then averaged across sampled frames to obtain a video-level score; used as a staged filter — the low-aesthetic tail is removed, and "the cutoff becomes stricter at higher-resolution stages."
· Brightness: follows OpenHumanVid's formula L = 0.2126R + 0.7152G + 0.0722B (ITU-R BT.709 luminance weighting), computed on sampled frames and used to remove the extreme low/high brightness tail for the target stage, filtering severe under/overexposure and improving the visibility of subjects and scene content in the retained data.
· Model-based Suitability Score: borrowing from Koala-36M, aggregates multiple quality factors into a single estimate of "whether this video is suitable for training a video generation model"; used conservatively in practice, only as a rejection filter removing the lowest-suitability tail, with all remaining samples still needing to pass subsequent specialized filters.
· Technical quality: DOVER, taking its technical-quality-related output, filtering compression artifacts, noise, distortion, low sharpness, and other degradations.
· Text/watermark/black border: see pipeline_overview levels 1–3 (OCR pre-screening + VLM watermark tagging + cropdetect + PaddleOCR-VL persistent-region clustering).
· VLM quality tags: clips tagged quality=low are excluded starting from the 480p stage.
The paper specifically emphasizes that these signals are "not used as a single learned ranking," but rather each filter targets a specific failure mode (poor exposure, compression artifacts, static content, temporal instability), which stands in clear methodological contrast to approaches that fold multiple metrics into one composite score.

### [Movie Gen](../models/Movie_Gen.md)

Visual filtering comprises 6 filters in total, with item-by-item detail (Appendix J.1 gives the models and thresholds):
· Resolution: removed if min width or height <720px (the high-resolution stage mentions 768px).
· Aspect ratio: filtered to achieve a target mix of 60% landscape / 40% portrait (80%/20% for the HD set).
· OCR text filtering: an in-house OCR model adaptively samples frames → detects words → recognizes word text; only videos where, across all sampled frames, "word-detection score × word-recognition score are both <0.6" are kept, used to remove material with excessive subtitles/decorative text/pasted-on text. This is one of the harshest cuts in the funnel (7%→1.94%).
· Black-border/frame detection: an in-house simple detector first uses a first-order derivative to find positions with large pixel differences in the vertical/horizontal direction, then a scan-line algorithm to locate the border; the motivation is that training data with borders causes generated video to also have black borders, which is especially common for portrait-orientation material.
· Aesthetic scoring: the public LAION aesthetics image model is applied to the middle frame of each clip, and scores <4.0 are removed, used to remove clips with severe blur or compression artifacts; the paper specifically ran a comparison — computing the average aesthetic score across multiple frames did not significantly improve recall of low-quality clips, so only the middle frame is used (saving compute).
· Visual effects and image quality: several additional lightweight visual models were trained, outputting frame-level "visual aesthetics, visual quality, large border, visual effects" prediction signals used for filtering.
· Opening segment: removes the first few seconds of clips overlapping with the start of the video.
The SFT stage additionally: uses the Detic object-detection model to remove videos where the subject is too small; manual review checks whether the frame is cluttered, whether colors are over-saturated, and whether there is overlaid text or editing effects.
On the audio side, quality filtering of video is lighter: removing videos with text, removing static videos, and removing videos with resolution <480px.

### [NVIDIA NeMo Curator](../models/NeMo_Curator.md)

Visual quality filtering consists of four classes of discriminators (Cosmos WFM production version), of which the first two are already available in the open-source NeMo Curator:
[1. Aesthetic filtering (open-source, already provided)] Uses a CLIP-based aesthetic model to score sampled frames one by one, then aggregates per clip. Key parameters: reduction can be min (default) or mean — defaulting to min means a strategy of "if one frame fails, the whole clip is eliminated"; score_threshold defaults to 3.5; target_fps defaults to 1.0; num_gpus_per_worker defaults to 0.25. Requires upstream frame sampling to use the sequence strategy with a matching frame rate. The Cosmos WFM production environment likewise uses this 3.5 threshold.
[2. Distortion/image-quality assessment] A scoring model based on DOVER (a video quality assessment model), removing the lowest-scoring 15% of clips. This stage has no corresponding implementation in the open-source version.
[3. Text-overlay detection (a substitute for traditional OCR functionality)] Instead of traditional OCR, an MLP classifier is trained on top of InternVideo2 video embeddings to identify post-processed text overlays, removing videos with excessive text. This approach of "replacing heavy OCR with embeddings + a lightweight MLP" has significant cost advantages at petabyte scale and is worth borrowing.
[4. Video-type taxonomy classification] Likewise InternVideo2 embeddings + an MLP, used to exclude abstract patterns/game footage/animation (see domain_distribution).
[Not covered] Watermark/logo detection, black-border detection (cropping is a separate stage but it is not stated whether black-border detection is automated), compression-artifact detection, and a standalone blur criterion are all absent from the official documentation. Eliminated clips are moved into video.filtered_clips and their statistics updated, facilitating post-hoc auditing of the funnel — a good observability design.

### [OmniHuman Dataset + OHBench](../models/OmniHuman.md) ⚠️

Visual quality filtering is concentrated at levels one and three, the most tool-dense part of the pipeline, using a clear "light to heavy" two-stage approach:
[Aesthetics and image quality: light and heavy stages]
- Light stage: frame-level CLIP aesthetic scoring (a CLIP-feature-based aesthetic predictor, i.e., a LAION-Aesthetic-style method), scoring frame by frame quickly, used to "rapidly remove low-quality clips." Its advantage is extremely low cost, runnable at full scale; the drawback is it only looks at single frames, not temporal information;
- Heavy stage: video-level DOVER (Disentangled Objective Video Quality Evaluator, a dedicated VQA model that decouples video quality into aesthetic and technical branches), assessing composition and clarity. DOVER's computational cost is much higher than CLIP scoring, so it is placed afterward, processing only the surviving samples.
This "CLIP coarse screening + DOVER fine screening" two-stage structure is a fairly mature cost-effectiveness tradeoff.
[Text contamination: a triple treatment of detection + cropping + density filtering] This is a more detailed step than comparable work in this pipeline, spread across two levels:
- First level: OCR and logo-detection algorithms estimate "text-contaminated regions," followed by frame cropping to remove subtitles and station logos — note this is "cropping out" rather than "discarding the entire video," maximizing data retention;
- Third level: OCR-based text-density analysis eliminates samples that still have too much text after cropping, in their entirety.
The comparison across three strategies is quite illustrative: UniTalking is "discard on any sight of text" (most wasteful), MOVA is "don't process but label it as a controllable attribute" (most data-efficient, but leaves the noise to the model), and OmniHuman is "crop where possible, discard only when cropping fails" (a middle ground, and the most engineering-intensive). For a data source like YouTube where burned-in subtitles are extremely common, the data-preservation value of the cropping strategy is considerable.
[Watermark detection] The third level explicitly includes watermark detection, alongside OCR text-density analysis. The specific method and criteria are not described. [Uncertain]
[Resolution threshold] HD and above (see resolution_aspect_distribution).
[Face clarity: a standalone metric] Frame-level annotation includes a face-clarity score, defined as Cs = Var(ΔR), i.e., the variance of the Laplacian-operator response (Laplacian variance) — a classic image-blur metric. Notably, this score is retained as an annotation field rather than only being used for filtering, meaning downstream users can screen subsets by face clarity themselves, more flexible than a one-size-fits-all filter.
[All thresholds missing] The CLIP aesthetic-score threshold, DOVER-score threshold, text-density threshold, and face-clarity threshold are all given without numeric values. [Uncertain]
[Not done] Black-border detection and removal are not mentioned; exposure/color-cast detection is not mentioned.

### [Open-Sora Series](../models/Open-Sora.md)

[Aesthetics] Both use LAION-family CLIP+MLP aesthetic scorers (improved-aesthetic-predictor, trained on 176K SAC + 15K LAION-Logos + 250K AVA image-text pairs, output on a 1–10 scale). The Open-Sora documentation gives an empirical scale: 5.5 as an "acceptable" threshold, 6.5 as a "highly aesthetic" threshold, with good text-to-image models reaching 7.0+; the first/middle/last frame of the video are evaluated; the example filtering command is `python -m tools.datasets.datautil meta.csv --aesmin 5.0`. Open-Sora 1.2's filtering threshold for Panda-70M is **aesthetic score > 4.5** (yielding a 20M subset / 41k hours); the image-side LAION subset uses **aesthetic score > 6.5**. Open-Sora Plan uses **LAION Aesthetic v2 ≥ 4.75** (the paper explicitly explains that 4.75 was chosen because it also happens to filter out a large amount of footage with excessive text), and **for samples with aesthetic score > 6.25, the prefix "A high-aesthetic scene" is automatically added before the caption** — turning the aesthetic score into a controllable condition rather than just a filter. Open-Sora 2.0's aesthetic-score distribution statistics show that the bulk of the training data falls in the **4.5–5.5** range (i.e., a deliberate choice to retain a large amount of "medium-aesthetic" samples to preserve diversity, rather than keeping only high-scoring samples).
[Clarity/blur] Open-Sora 2.0 uses **the variance of the OpenCV Laplacian operator** for blur detection, with the video judged by majority vote across five evenly spaced sampled frames; Open-Sora Plan uses **DOVER's technical-branch predicted score (threshold ≥ 0)**, specifically targeting compression artifacts and low bitrate, closer to real image-quality degradation than a plain blur operator. Open-Sora 2.0 also removes low-bitrate sources directly during preprocessing using **bpp (bits per pixel) < 0.02**.
[OCR/text filtering] Three different approaches: Open-Sora 1.x uses **DBNet++ (MMOCR implementation)**, with the output column `ocr` recording the **number of text regions with detection confidence > 0.3**, used to remove densely-texted scenes such as news broadcasts and ads; Open-Sora 2.0 uses **PaddleOCR**, counting only text with confidence > 0.7, discarding when there is too much text; Open-Sora Plan uses **EasyOCR to detect only the bottom 20% region of the frame** (the typical location of subtitles) and takes a **crop-rather-than-discard** strategy (edge ratio 0.20); the paper also candidly acknowledges the method's limitation — it cannot handle ad text or speech captions located in the center of the frame.
[Black border/watermark/logo] None of the three has **implemented** a dedicated black-border detection, watermark detection, or logo detection module — a shared gap; the Shutterstock watermark issue in Webvid-10M therefore goes unaddressed.

### [Ovi](../models/Ovi.md) ⚠️

Visual quality filtering consists of three explicit modules, all in Step 1 of the pipeline:
(1) Resolution threshold: clip resolution must exceed 720×720 pixels, hard-removing low-clarity material. Ovi 1.1 raised this to native 960×960 data.
(2) Aesthetic scoring: uses LAION's aesthetic predictor (Schuhmann, 2022, i.e., the LAION-Aesthetics aesthetic scoring model) to remove low-quality data. The specific threshold is not disclosed [Uncertain].
(3) Motion quality: RAFT optical flow removes static clips (see motion_filtering for details).
(4) Structural cleanup (during the packing stage rather than the filtering stage): margins already present in the video (black bars/letterbox borders) are explicitly removed, then area-scaled to match. This treats "black borders" as a repairable flaw rather than grounds for discarding.
(5) Content-composition control: an internal face-detection model ensures a reasonable mix of single-person/multi-person/no-person categories (this is distribution control rather than quality filtering, but performed in the same step).
[Common methods not mentioned] OCR/subtitle text filtering, watermark detection, logo detection, compression-artifact/blur detection, brightness/contrast filtering, JPEG blocking-artifact detection, etc. — none of these are described in the paper [Uncertain]. This may be a potential risk point for the model on subtitled material.
[Audio-side quality filtering] See audio_quality_filtering (average volume ≥ −60 dB).

### [Script-a-Video](../models/Script-a-Video.md) ⚠️

The paper does not disclose any quality-filtering details: no aesthetic scorer (none of LAION-Aesthetic / DOVER / VQA-type mentioned), no clarity or blur metric, no OCR burned-in subtitle detection, no black-border detection, no watermark/logo detection, no bitrate or compression-artifact detection, and no threshold values given at all. All quality control is compressed into a single adjective: "high-quality video clips." [Uncertain]
There are only two indirect clues related to "quality":
1) The choice of data domain itself constitutes an implicit quality filter — film / television / cinematic pairs are all professionally produced content, naturally superior to UGC in image quality, composition, and audio recording;
2) The final generation-side stage uses "60K high-fidelity cinematic alignment pairs" (high-fidelity, cinematic-grade alignment pairs); the wording "high-fidelity" hints at a stricter high-quality subset selection, but the selection criteria are not disclosed.
The blankness of this field relates to this work's positioning: the paper's contribution claim lies in the annotation paradigm rather than the cleaning pipeline, so the cleaning section is omitted entirely. For research focused on annotation methods this is an acceptable tradeoff, but for research focused on the cleaning funnel, this entry has very low reference value.

### [Seedance 2.0 and Seedance 1.5 pro](../models/Seedance_20_Seedance_15_pro.md) ⚠️

Seedance 1.5 pro merely states that the pipeline includes "clip quality"-related screening (a third-party summary claims the second stage is automated filtering for audio-video sync and clip quality, but this wording does not appear in the original paper). Seedance 1.0's specific approach: (a) visual-occlusion correction — using a hybrid scheme of heuristic rule systems + dedicated object-detection models to identify occlusions such as logos, watermarks, subtitles, and on-screen graphics, then applying adaptive cropping to frames to maximize retention of the main visual content; (b) quality filtering — a dedicated "specialized visual quality model" systematically identifies and removes clips with blurriness, excessive jittering, low aesthetic quality, poor cinematographic composition, and predominantly static content. The Continue Training stage further uses "a series of specialized assessment models, including an aesthetic scorer," to select a subset of higher aesthetic quality. All thresholds, scorer scale, and tiering are undisclosed. [Uncertain: specific thresholds and whether OCR text filtering forms an independent stage]

### [SkyReels Series](../models/SkyReels.md) ⚠️

[SkyReels-V2] Quality issues are explicitly grouped into three categories, with a filter matrix configured for each:
(1) Basic quality issues: low-resolution sources, low frame rate, black/white screens/static frames, camera shake;
(2) Video-type issues: surveillance footage, game recordings, animation, meaningless content (removed as whole categories);
(3) Post-processing artifacts: subtitles, station logos/logos, image-editing traces, split screens, black borders.
Corresponding filters include: a black-screen filter, a static-frame filter, an aesthetic filter, an OCR filter (detecting on-screen text/subtitles), a mosaic filter, an effects-and-sticker filter; VQA (video quality assessment), IQA (image quality assessment), and VTTS-type models are also used for scoring. For salvageable samples, a "crop rather than discard" strategy is taken — subtitle and logo regions undergo subtitle/logo cropping to maximize usable frame area, a practical design for improving data utilization.
[SkyReels-V4] Filtering dimensions converge to three: basic quality (duration, resolution, aesthetic score, blur, contrast, exposure), content quality (watermark, logo, text overlay, synthetic artifacts), and motion quality. "Synthetic artifact detection," used to remove AI-generated or heavily post-processed content, is a filtering dimension that newly emerged in 2026. Neither generation discloses the source of the aesthetic scoring model (in-house or LAION-Aesthetics-family) or the threshold values for each filter. [Uncertain]

### [Sora 2](../models/Sora_2.md) ⚠️

Only a general statement: "rigorous filtering to maintain data quality." OpenAI does not mention any specific method such as an aesthetic scoring model, clarity/blur criteria, OCR text/subtitle filtering, black-border cropping, or watermark/logo detection — no thresholds or model names are given. [Uncertain]

### [SpeakerVid-5M](../models/SpeakerVid-5M.md) ⚠️

Five-dimensional threshold filtering, with all thresholds public, plus a stricter HQ threshold group:
[1. Luminance] Clips with a luminance score < 10 or > 210 are removed (the two ends of the 0–255 grayscale range), used to exclude overly dark and overexposed footage.
[2. Video quality (DOVER)] Uses DOVER's fused score for video quality assessment; those <0.25 are removed.
[3. Clarity Score] A custom metric, defined as clarity = B/(W×H), i.e., bitrate (B) divided by resolution area — measuring bit density per pixel, effective at identifying "high-resolution but actually upsampled-from-low-clarity/heavily-compressed" fake-HD video. Ranked by score, the lowest 5% are removed. This is a relatively distinctive custom quality metric in this dataset.
[4. Motion Blur] A 128×128 patch is cropped from the face and hand regions respectively, and Laplacian variance is computed as a clarity/blur score; clips where the average face or hand blur score is < 0.1 are removed. Face and hand are scored independently in two separate tracks, a design specifically for downstream gesture-generation tasks.
[5. Audio quality] Three parallel veto rules, removed if any one is met: Whisper transcription confidence < −1.5; no-speech probability > 0.8; compression ratio > 2.5 (used to identify ASR repetition degeneration/hallucinated output).
[HQ/SFT subset thresholds (stricter)] Hand blur score > 0.5, face blur score > 0.7, DOVER > 0.6, motion-intensity score > 2, ASR confidence > −1. All five must be satisfied for selection, yielding 571K clips / 1,368 hours.
[Filtering not performed] The paper does not describe a standalone aesthetic score, OCR burned-in subtitle detection and filtering, watermark/logo detection, or black-border detection. DOVER itself has an aesthetic-view branch, but the paper only uses its fused score without splitting it out. [Uncertain]

### [Step-Video-T2V](../models/Step-Video-T2V.md)

Stage 2 assigns 7 categories of quality labels to sampled frames of each clip, with the tools and methods almost all named (threshold values are entirely undisclosed, only stating that "thresholds are progressively raised" to construct 6 subsets):
1. Aesthetic Score: a LAION CLIP-based aesthetic predictor (the open-source LAION aesthetic predictor), evaluating the visual appeal of the frame;
2. NSFW Score: a LAION CLIP-based NSFW detector, a binary classifier based on CLIP ViT-L/14;
3. Watermark Detection: an EfficientNet-based image-classification model, determining whether a frame contains a watermark;
4. Subtitle Detection: PaddleOCR recognizes on-screen text, used to remove clips with excessive subtitles/text;
5. Saturation Score: statistics of the mean/max/min saturation in HSV color space, removing abnormal frames that are overexposed, overly gray, or oversaturated;
6. Blur Score: the Laplacian variance method measures picture sharpness, removing out-of-focus/blurry clips;
7. Black Border Detection: FFmpeg detects black-border size, and cropping is performed accordingly (cropping rather than discarding, consistent with HunyuanVideo 1.5's approach).
Methodological characteristics: this quality-inspection toolkit is entirely a combination of "traditional CV operators + small open-source classifiers," with none being large-model semantic judgment, representative of the typical "shallow multi-scorer" paradigm of early-2024–2025; its advantages are that all tools are named, cost is low, and reproducibility is well described; its drawback is an inability to capture semantic-level quality issues (such as physical implausibility or subject distortion).

### [UniTalking](../models/UniTalking.md) ⚠️

Visual quality filtering is concentrated at the first level of the funnel; the paper summarizes it in one sentence as removing three categories of video — "static, containing text overlays, and overall low visual quality" — with no model name or threshold given at all:
[Static-frame removal] Removes static videos. The judgment method (optical flow? frame difference? a motion-score model?) and threshold are not stated. This is the only motion-related filtering condition in the entire paper. [Uncertain]
[Text-overlay removal] Removes videos containing text overlays. This is a fairly targeted rule — burned-in subtitles, danmaku, and station logos are extremely common in talking-head videos, and failing to remove them would cause the model to generate garbled text on screen. But the paper does not state whether an OCR detector (PaddleOCR/CRAFT, etc.) or some other method is used, nor whether it is "removed if any text at all" or "removed only if text-area proportion exceeds a threshold." Worth comparing to MOVA's approach: instead of removal, it is labeled as a controllable attribute ("This video has no subtitles."), whereas UniTalking chooses direct elimination. [Uncertain]
[Overall low visual quality removal] "low overall visual quality" — an extremely vague description. It is not stated whether DOVER, LAION-Aesthetic, MUSIQ, or some other evaluator is used, nor whether sub-dimensions such as clarity, blur, compression artifacts, or brightness are included.
[Filtering not performed] No watermark/logo detection, no black-border detection and removal, no lower-bound constraint on face size/proportion (although the task is highly face-dependent, no explicit face-size gate is seen), no exposure/color-cast detection, no independent aesthetic score.
[Quality assurance inherited from upstream] The OpenHumanVid portion has already undergone multi-dimensional quality filtering in its own pipeline based on luminance, blur, aesthetics, motion, and technical metrics, and additionally performs person-quality filtering and caption-visual consistency filtering (structured captions generated by MiniCPM/CogVLM/Llama and refined by BLIP2 voting). Thus UniTalking's actual quality gatekeeping on the OpenHumanVid subset is a stack of "meticulous upstream + coarse at this level," whereas Huawei's internal data has only gone through this level's coarse filtering — a potential risk of mismatched filtering strength between the two sources that the paper does not discuss. [Uncertain]

### [UniVerse-1](../models/UniVerse-1.md) ⚠️

Visual-side quality filtering is concentrated at level 2 of the funnel, with three hard thresholds in total, all values publicly given:
[Resolution] Videos below 1080p are removed. This bar is relatively high among comparable work (MOVA uniformly normalizes to 720p for training, even starting as low as 360p), reflecting high selectivity given the small data scale.
[Bitrate-to-resolution ratio] Removed if the bitrate-to-resolution ratio is below 600. Used to identify material with "nominally high resolution but actually heavily compressed" — a fairly distinctive metric in this work, rare among comparable work.
[Aesthetic/technical quality] Removed if the DOVER composite quality score is below 0.6. Unlike MOVA, which splits DOVER-Aesthetic (>0.85) and DOVER-Technical (>0.05) into two scores, UniVerse-1 uses only a single overall DOVER score with a threshold of 0.6, at a coarser granularity.
[Threshold determination method] The paper does not explain how the threshold was calibrated (neither manual spot-checking nor ablation experiments are mentioned), suggesting an empirical setting. [Uncertain]
[Filtering not performed] No OCR burned-in subtitle detection (MOVA has this, and even made it a controllable attribute), no watermark/logo detection, no black-border detection and removal (no mention of cropdetect-type processing), no independent aesthetic scorer (such as LAION-Aesthetic), no dedicated clarity/blur metric.
[Alternative handling of low-quality data] For VGGSound/AudioSet, which cannot pass the above filters, rather than filtering them out, an LQLS (training-side) strategy is used for isolation — the Flow Matching loss is applied to these two datasets only when the diffusion timestep t > 800 (out of a total of 1000 steps), i.e., they participate only in learning the high-noise segment (coarse structure/semantic layer), not the low-noise segment (high-frequency detail layer), thereby avoiding the model overfitting to the high-frequency artifacts of their low-quality visuals. This is a typical case of shifting the data-quality problem from the "filtering dimension" to the "loss-weighting dimension."

### [Unison](../models/Unison.md) ⚠️

The paper does not describe any visual quality filtering step: no aesthetic-score gate, no clarity/blur metric, no OCR burned-in subtitle detection, no watermark/logo detection, no black-border detection and removal, no resolution or bitrate gate, no compression-artifact detection. All related information is blank.
[Why it's blank — a structural explanation rather than an oversight] During Stage 2 joint training, the video backbone Wan2.2-5B is entirely frozen (keeping the video backbone frozen), with only the audio branch and fusion modules (bidirectional cross-attention and layer normalization) being trained. This means the visual quality of the training data does not affect visual generation capability via gradients — low-quality visual samples can at most disturb cross-modal alignment learning, a far smaller hazard than in work that must train the visual branch. Unison therefore has good reason to put its entire filtering budget toward the audio and alignment dimensions rather than visual quality. This is particularly clear when contrasted with UniVerse-1: the latter likewise uses low-visual-quality VGGSound, but because it must train the visual branch, it had to specifically design the LQLS loss-isolation strategy; Unison uses the same VGGSound without needing any corresponding treatment.
[Quality assurance is effectively front-loaded into data-source selection] Unison performs no quality filtering, but its material selection itself is quality-oriented: VFHQ was built to a "high-fidelity" standard, HDTF is named for "High-Definition," and OpenHumanVid calls itself "high-quality." All three data sources had already completed their own quality screening at release time, and Unison directly inherits these screening results — a natural advantage of "aggregation-type data construction," and it also explains why the paper describes its audio corpus as "high-quality audio segments" without stating the screening basis.
[The only confirmed quality-related processing is on the audio side] The output of Mel-RoFormer separation is described as "high-fidelity speech and sound-effect components," but this refers to separation quality, not a filtering threshold.
[Quality models used in evaluation but not for filtering] LAION-Aesthetic Predictor V2.5 (video aesthetic score VA), DINOv3 (inter-frame identity consistency ID), Audiobox-Aesthetics (audio PQ/CU) — all three are used only for evaluation, with no evidence of use as training-data filters. This is a notable mismatch: the team clearly has access to these scoring tools, yet did not use them for data screening. [Uncertain]

### [Veo 3 / Veo 3.1](../models/Veo_3_Veo_31.md) ⚠️

The only official wording is "Training videos were also filtered for various compliance and safety metrics and for quality," which confirms that a quality-filtering step exists. [Uncertain] The specific methods are entirely undisclosed: whether an aesthetic scoring model, clarity/blur metric, OCR text-coverage filtering, black-border detection, watermark/logo detection, or compression-artifact detection is used — none is stated, and no threshold values are given. A weak inference can be drawn: the technical report acknowledges that Veo 3 "still performs relatively poorly at generating text"; if the data side had applied aggressive OCR text filtering, it would further weaken text-rendering ability — the two are directionally consistent, but this cannot serve as evidence.

### [Vidu S1](../models/Vidu_S1.md) ⚠️

Multi-dimensional quality filtering combining expert models and a large model, covering six focus areas:
(1) Subject detection: each clip contains exactly one subject occupying a reasonable proportion of the frame;
(2) Frame cleanliness: removes clips containing information irrelevant to visual content, explicitly listing watermarks, subtitles, and overlaid/pasted-on advertisements;
(3) Visual quality: uses dual scoring — aesthetic scoring and technical scoring — to select clips that are clear, complete, and visually pleasing, avoiding artifacts such as blur, jitter, and flicker;
(4) Content safety: removes NSFW and other inappropriate content;
(5) Shot stability: retains static or slow-motion shots, to reduce the risk of shot drift in long-duration streaming generation;
(6) Interactivity: requires the subject to have clear actions/behavior.
No specific scorer names or threshold values are disclosed [partially uncertain]. No mention of the specific implementation of OCR text detection, or of black-border detection.

### [Wan 2.5 / 2.6 / 2.7](../models/Wan.md) ⚠️

Wan 2.1's quality filtering is among the most completely enumerated in publicly available materials industry-wide (2.5+ is undisclosed, presumably inheriting and upgrading it).
[Basic-dimension parallel detection (faithfully reproduced item-by-item at the original text's level of detail)]
1) Text detection: a lightweight OCR detector quantifies "text coverage," removing video and images with excessive text elements to preserve visual clarity;
2) Aesthetic evaluation: uses an industry-standard LAION-5B aesthetic classifier for preliminary quality assessment, quickly removing low-quality data;
3) NSFW scoring: an internal safety-assessment model computes an NSFW score for all training data and filters accordingly;
4) Watermark and logo detection: detects whether watermarks/logos are present, and crops out these regions during training (note this is cropping, not discarding — a "recoverable" treatment);
5) Black-border detection: a heuristic method automatically crops out excess black borders, focusing on content-rich regions;
6) Overexposure detection: a self-trained expert classifier identifies and removes data with abnormal tone distribution;
7) Synthetic-image detection: a self-trained expert classifier removes AI-generated images. The report gives a highly valuable empirical finding — "even a very small proportion (<10%) of generated-image contamination significantly degrades model performance," a direct quantitative assertion of the importance of data purity;
8) Blur detection: an in-house model assigns a blur score to training material, systematically removing visually unclear content;
9) Duration and resolution: duration must be >4 seconds; the resolution threshold is dynamically raised across training stages.
[Learned visual-quality scoring] Samples drawn from each cluster are manually scored on a 1–5 scale (1 worst, 5 best); all annotated data are used to train an expert assessment model, which then scores the full dataset, used for data selection and processing at each stage.
[Wan2.2 addition] Introduces "carefully curated aesthetic data with fine-grained labels for lighting, composition, contrast, color tone, etc.," enabling the model to controllably generate in cinematic aesthetic styles — an important upgrade from "a single aesthetic total score" to "multi-dimensional controllable aesthetic labels," and the data foundation for the "cinema-grade" positioning of 2.5+.
[Wan2.2-S2V's five video-quality metrics] Dover clarity assessment, UniMatch optical-flow motion stability (removing subjects/backgrounds with overly violent motion), a Laplacian operator specifically checking whether the face and hand regions are blurry, an improved aesthetic predictor, and OCR detecting whether subtitles occlude the face or hands. Of these, "separately checking sharpness for face/hand regions" and "detecting subtitle occlusion of the face" are the two most worth borrowing, being designed specifically for human-centric scenarios.

### [Audio-Video Generation Evaluation Benchmark Collection](../models/av_benchmarks.md) ⚠️

Quality-gatekeeping methods across the various benchmarks:
[AVBench] The quality dimensions themselves are directly reusable filters: its 6 unimodal quality dimensions are each bound to a mature tool — audio quality via NISQA MOS, audio aesthetics via Audiobox, video technical quality via DOVER++, video aesthetics as a standalone dimension, plus two speech-specific dimensions, Speech Content Accuracy and Speech Realism. The paper explicitly states that its continuous, differentiable scores can be directly used as a data-filtering mechanism and as an RLHF reward signal — this is the benchmark in this survey most directly transferable as a training-data quality filter.
[VABench] Module 1's unimodal audio-quality trio: SpeechClarity uses DNSMOS to detect background noise, SpeechQual&Nat uses NISQAv2 to assess overall quality and naturalness, and AudioAesthetic uses Audiobox to assess pleasantness/usefulness/production complexity and quality. Visual-side quality is folded into MLLM scoring under Visual Realism and Artistry. In addition, a stereo-specific track provides 9 acoustic-quality metrics: spatial imaging quality (stereo width, imaging stability, level stability, inter-channel consistency) and signal integrity/compatibility (phase-coherence scores evaluated across low/mid/high frequency bands, mono-compatibility Mono Compat = 1 − normalized mono loss, directional consistency, and Mid/Side energy ratio measuring soundstage width).
[AV-SyncBench] The manual stage explicitly removes three categories of samples — "poor audio quality, excessive noise, semantically ambiguous" — i.e., a triple gate of audio SNR, clarity, and semantic discriminability (no quantitative thresholds given [Uncertain]).
[PhyAVBench] Quality control focuses on physical correctness rather than visual aesthetics: an LLM does initial screening for semantic ambiguity and unintended physical confounders, followed by manual review of physical consistency and realism; since the content is self-recorded in a controlled environment, issues like frame clarity, watermarks, and black borders are naturally absent.
[Omni-Judge] Performs no filtering; but its conclusion is cautionary for quality filtering — Omni-LLM shows extremely low correlation with human judgment on the video-quality dimension (Kendall τ_b ≈ 0.020), indicating that using an Omni-LLM directly in place of a traditional aesthetic/technical quality scorer for data filtering is unreliable; image-quality-type filtering still requires a dedicated model.
No mention of OCR text filtering, watermark/logo detection, or other common training-side methods [Uncertain].

### [Video Caption Model Ecosystem](../models/caption_models.md)

The captioner ecosystem's "quality filtering" has two directions:
[Direction 1: captioner as a component of quality filtering]
· Allegro: Tag2Text output serves as the text-side input for CLIP-similarity filtering, embedding the captioner directly into the filtering chain.
· Motif-Video 2B in the Mochi/MAGI/Motif batch: additionally equipped with PaddleOCR-VL (served via vLLM) for on-frame text detection, an OCR filtering branch within the captioning chain.
· InstructAV2AV uses Grounded-SAM-2 to provide instance-level mask anchors and TalkNet for active-speaker detection — structured annotation tools that double as quality gates.
· Traditional shallow filters (LAION aesthetic score, DOVER technical score, clarity, black-border/watermark/logo detection) run in parallel with, rather than in series with, the captioner, usually completed before captioning to save captioning compute.
[Direction 2: quality control of the captioner's own output] This is the ecosystem's core problem; the main approaches fall into four categories:
(1) LLM scoring filtering: AVoCaDO uses GPT-4.1 to score synthesis completeness on a 1–5 scale, keeping only ≥4;
(2) Automated consistency verification: AVSCap's automated verification (tag-retention checking + semantic-consistency checking);
(3) Retrieval-based selection: Panda-70M uses UMT-large (ViT-L/16 + BERT-large, VTC+VTM dual loss + hard-negative mining, with the un-selected 7 captions weighted at 1.0 and other in-batch negatives weighted at 0.01, 12 frames at 224×224, AdamW lr 2e-5, batch 32, 10 epochs, 8×A100-80G) to select the best caption from multiple candidates, achieving R@1 of 35.90% after fine-tuning (versus only 21.82% for pretrained UMT);
(4) Hallucination-suppressing post-training: Tencent Hunyuan 1.5 uses OPA-DPO (a preference optimization method targeting multimodal hallucination) for RL post-training of its three caption models; video-SALMONN 2's MrDPO jointly rewards completeness and factuality, reducing the caption error rate of the 7B model by 28% relative to baseline; Tarsier2 uses model-based sampling to construct preference pairs for DPO.
[A generation-side fallback] Step-Video-T2V does not apply dedicated hallucination-suppression post-training to its captioning model, and instead falls back on manual review at the SFT stage plus CLIP Score alignment filtering at Stage 6 — representing a pragmatic route of "if the captioner isn't good enough, fix it with downstream filtering."

### [Geometric/Structured Annotation Dataset Collection](../models/geometric_datasets.md)

SceneScribe-1M: centered on Qwen2.5-VL-72B's six-dimensional content quality assessment, explicitly removing clips with unclear motion intensity, visible watermarks, strong camera distortion, or strong lighting artifacts; layered on top are hard specification gates (>1080p, ≥10fps, 5s–60s). SpatialVID: four independent scorers form the quality layer — ① Aesthetics: CLIP+MLP aesthetic predictor, scores < 4.0 removed; ② Brightness: only clips with mean value falling in the [20, 140] range are kept, too-dark or overexposed removed; ③ OCR text: PaddleOCR detection, clips with text-area proportion > 30% are flagged for removal; ④ Motion: VMAF metric + subsequent reconstruction-feasibility judgment. WildWorld: engine-rendered footage is naturally free of watermarks, compression artifacts, and black borders, so quality filtering degenerates into scene-validity screening, mainly ensuring recording synchronization and lossless depth encoding. Action100M: applies almost no image-quality filtering (retaining HowTo100M's original image quality), with quality control shifted to the annotation side — ensuring annotation quality via multi-round LLM Self-Refine, and removing meaningless fragments via duration lower bounds (clip >0.5 seconds, tree node >4 seconds).

### [Video Generation Post-Training Data Strategy](../models/post_training_data.md) ⚠️

[Anchor paper] SFT data's quality-filtering standards are entirely undisclosed [Uncertain]. Quality control is handed entirely to four reward models in the RLHF stage (video aesthetics: lighting, composition, color harmony, temporal consistency, cross-frame cinematic feel; image aesthetics: frame-level perceptual quality, sharp detail, structurally pleasing; motion quality: motion realism, smoothness, coherence, suppressing jitter/motion discontinuity/temporally inconsistent object transitions; text-video alignment: semantic consistency). This reflects a paradigm shift — moving "what counts as high quality" from a data-filtering threshold into a learnable reward model.
[Cross-work SFT quality-screening standards (the few cases with publicly given thresholds)]
· Allegro (the most complete set of thresholds): duration 6–16 seconds, resolution ≥1280×720, brightness [20,180], DOVER ≥0.07, LPIPS ≥0.05, UniMatch motion [1.0,100], aesthetics ≥5.3, text area ≤0.05%, CLIP similarity ≥0.20;
· HunyuanVideo original version: 1 million manually curated samples, with standards of four aesthetic items (color harmony, lighting, subject prominence, spatial layout) + three motion items (motion speed, action completeness, motion blur) — this seven-dimensional manual standard is the most frequently cited SFT-screening rubric on this topic;
· HunyuanVideo 1.5 SFT: on top of CT data, re-screened more strictly by three items — aesthetic appeal, clarity, and motion smoothness — with the final set built via manual annotation;
· CogVideoX: targeting three types of dirty data — subtitles, watermarks, low bitrate — screening out the top 20%;
· Step-Video-T2V: a three-step process — automatic scoring and heuristic-rule filtering → VideoCLIP K-means removes outliers within clusters that exceed a distance threshold from the cluster center (threshold undisclosed) → manual item-by-item review (clarity, aesthetics, whether motion is appropriate, whether scene transitions are smooth) with caption refinement;
· LongCat-Video: a first layer of multiple metrics (aesthetic score, video quality, motion quality) + a second layer of inverse-density sampling in caption-embedding space;
· Motif-Video 2B: regular filtering + a stricter aesthetic cutoff + style/subject-tag-driven domain-balancing + action=Dynamic;
· NAVA: multi-operator collaborative filtering, with the standard being "accurate caption + strong audio-visual alignment," threshold undisclosed.
[Commonalities] SFT screening = pretraining thresholds + significantly raised aesthetic/clarity gates + concept balancing + manual final review. Manual final review appears in almost all industrial-grade work — HunyuanVideo (original and 1.5), Step-Video-T2V, Movie Gen, SkyReels-V4, Apollo Stage III, etc. — a hallmark step distinguishing SFT screening from pretraining filtering.

### [Merged Survey of Mainstream Video Pretraining Datasets: Panda-70M, InternVid, Koala…](../models/pretraining_datasets.md) ⚠️

**Horizontal comparison across four categories — "aesthetics / clarity-technical quality / OCR text / black-border-watermark"**:
[Aesthetic scoring]
- **Panda-70M: none.** This is its biggest gap.
- **InternVid**: LAION aesthetic predictor, **≥4** yields the InternVid-18M-AES subset (used only to derive a subset, not the main funnel).
- **MiraData**: LAION-Aesthetic predictor, computed at a uniform **2fps**, **threshold undisclosed** (the paper says it is in supplementary material, but the supplementary material does not exist). [Uncertain]
- **OpenVid-1M**: LAION aesthetic score, **the top 20% is taken from the Panda-50M source, and the top 90% from the other three sources** — using percentiles rather than absolute scores.
- **LVD-2M: does not use an aesthetic scorer**, instead using PLLaVA-7B's "visual diversity" prompt for semantic judgment (the word "aesthetic" appears in the whole paper only once, as the name of a VBench metric).
- **Koala-36M**: aesthetic score is one of the input sub-metrics of VTSS (in the CSV, aesthetic_score's actual range is **2.28–6.56**), but **no independent threshold is set**.
- **UltraVideo: uses neither the LAION aesthetic score nor DOVER**, directly reusing **Koala-36M's VTSS** (native score linearly rescaled to −0.0575~0.0728), **VTSS<0.01 removed**.
[Clarity / technical quality]
- **OpenVid-1M**: **top 30% by DOVER technical score**.
- **Koala-36M**: clarity_score is a VTSS sub-metric (actual CSV range 0–1); the paper uses DOVER and FastVQA as comparison baselines for the TSA network (TSA PLCC 0.8974 vs DOVER 0.8554 vs FastVQA 0.8684).
- **MiraData**: a **"frame color" check** — computes the average color as well as the color of the brightest/darkest 80% of frames, removing overly bright/dark material. This is the only explicit exposure/color-level filter among the seven (in the same category as UltraVideo's overexposure check).
- **Panda-70M / InternVid / LVD-2M: no dedicated clarity/technical-quality scorer.**
- **UltraVideo: four statistical checks, all following a unified pattern of "discard the whole video if the bad-frame rate >5%"** — (a) black border: a rectangular region extending inward **3%** from each edge with a mean value **<3** marks a bad frame; (b) overexposure: pixel values **>250 or <5** in a proportion **>12%** marks a bad frame; (c) grayscale: per-pixel RGB variance averaged over the whole image **<1.2** marks a bad frame. This is the **only statistical filtering scheme among the seven that gives all absolute threshold values**, directly reusable.
[OCR/text filtering]
- **UltraVideo**: **PaddleOCR**, the union of minimum bounding rectangles of text / total image area **>2%** marks a bad frame, discard whole video if bad-frame rate >5%.
- **LVD-2M**: **does not use an OCR model**, instead using PLLaVA-7B's second prompt to judge "Text Presence: whether text overlay dominates the frame to the point of harming visual appeal."
- **Panda-70M / InternVid / Koala-36M / MiraData / OpenVid-1M: none have OCR text filtering** (the Koala-36M paper mentions no OCR anywhere, contrary to common assumption; OpenVid is undisclosed).
[Black border / watermark / logo / transition effects]
- **UltraVideo (the only systematic treatment)**: black borders have a dedicated statistical check (see above); watermarks, transition effects, split screens, screen recordings, picture-in-picture, and **16 low-quality attributes** in total are judged item-by-item as binary by **Qwen2.5-VL-72B**, and the whole clip is removed if any one is hit. A round of manual re-inspection at the source also removed watermarks/blur/jitter.
- **Koala-36M**: subtitles, logos, effects, and transitions serve only as **guidance for manual annotators' scoring** (the "video naturalness" dimension), transmitted indirectly and lossily to VTSS through 200K annotations, with **no independent detector**.
- **The remaining five: none have black-border/watermark/logo detection.** Of these, LVD-2M used WebVid material (which carries Shutterstock watermarks and has since been taken down over copyright issues), a clear source of watermark contamination.
**One important counter-intuitive finding**: the combination of aesthetic scoring + OCR, widely considered "essential," is being replaced in the two most recent datasets (LVD-2M, UltraVideo) — LVD-2M replaces it wholesale with a 7B VLM, and UltraVideo replaces it with a combination of "precise statistical thresholds + 72B VLM attribute judgment," **neither relying on the LAION aesthetic score anymore**.
## Motion Filtering (Optical Flow / Motion Score Thresholds, Removal of Static and Jittery Content)

`motion_filtering` · Detail level: brief

### [Allegro](../models/Allegro.md)

Uses the UniMatch (Xu et al., 2023) optical flow model to estimate motion magnitude, enabled only during the T2V fine-tuning stage, with a retained interval of [1.0, 100]: the lower bound of 1.0 removes near-static frames (static images, slideshow-style material), and the upper bound of 100 removes samples with excessively intense/jittery motion or fast transitions. None of the three pretraining stages set a motion threshold (left blank in Table 1), indicating the team treats motion quality as a "late-stage refinement" rather than a "pretraining admission" criterion.
In addition, the LPIPS ≥0.05 lower bound complements motion filtering, likewise used to exclude samples with almost no inter-frame change.
No camera-motion classifier or dedicated jitter-detection model was used.

### [Apollo](../models/Apollo.md) ⚠️

A motion-related filtering dimension exists, but neither method nor threshold is given: the paper lists "subject motion ratio" and "camera stability" as two sub-dimensions of dynamic-quality modeling, incorporated into the first stage of video filtering. This implies that both static, lifeless clips (motion ratio too low) and handheld-jitter/violently-shaking clips (poor camera stability) are, by design, handled — consistent in direction with common practice. However, the paper does not state whether the computation uses optical flow (RAFT/UniMatch), frame differencing, or a dedicated model, nor does it give upper/lower thresholds for the motion score. The evaluation stage uses RAFT optical flow to compute a Motion Score (MS), but this is an evaluation metric, and the paper does not state whether data filtering reuses the same tool. [Uncertain]

### [CineDance / CineDance-1M](../models/CineDance.md) ⚠️

【Metric】Uses AMT to compute Motion Smoothness as a quantitative characterization of the motion dimension, stored in the metadata alongside the DOVER aesthetic/technical scores.
【Strategy】Also follows the "no hard pruning" principle — the motion score carries no filtering threshold and serves only as metadata for downstream ad hoc selection.
【Indirect static removal】The target checklist for manual artifact auditing includes "still-frame holds" and "transition effects," which is a qualitative check on static/anomalous-motion content rather than automatic filtering based on an optical-flow threshold.
【Not adopted】The paper does not mention optical flow computation, motion-magnitude thresholds, camera-shake detection, or any other common motion-filtering technique, nor does it give any distribution or threshold value for the motion score. This is a relatively weak link compared with other large-scale datasets. [Uncertain]

### [CogVideoX](../models/CogVideoX.md) ⚠️

Two mechanisms run in parallel:
· Optical flow score: computed for all training videos, with the threshold dynamically adjusted during training, aimed at "ensuring the dynamic ... quality of generated videos." The specific threshold value is not disclosed [uncertain].
· Video-LLaMA static/connectivity classifier (Classifier-Static, corresponding to the negative label Lack of Motion Connectivity, test accuracy 0.92): removes clips with extremely little dynamic information or with motion discontinuities at transitions (manual splicing, static-image editing).
· The "Lecture Type" classifier (Acc 0.99) also effectively removes near-static footage.
· Jitter handling runs along the quality line rather than the motion line: severe camera shake is classified under the Low Quality negative label ("excessive camera shake") and removed by Classifier-Low Quality, rather than using a "shots per second" proxy for jitter as Movie Gen does.
· The paper does not use FFmpeg-side metrics such as motion vector or VMAF motion score.

### [Cosmos-Predict2.5](../models/Cosmos-Predict25.md) ⚠️

Motion filtering is the second stage of the filtering chain: "we apply a motion filter, which quantifies and removes clips based on their degree of motion" — quantifying each clip's degree of motion and removing accordingly. The paper does not state the measurement method (does not specify optical flow, frame differencing, or a model-based score), does not give a threshold, and does not state whether it is a two-sided filter (removing both near-static and overly jittery clips simultaneously) or removes only the low-motion side.
Three pieces of confirmable evidence exist: (1) robotics-domain preprocessing explicitly "filtered out low-resolution and near-static videos" — removing low-resolution and near-static videos, indicating at least a low-motion-side removal exists; (2) the robotics domain also performs motion-cadence normalization — speeding up playback for videos where the robot arm moves too slowly (increased the playback speed) to ensure consistent motion cadence across datasets, an uncommon "actively rewrite the motion rate" approach rather than mere filtering; (3) in post-training SFT, high motion is listed separately as one of five major domains and retains 1.0M samples, indicating high-motion samples are deliberately preserved and reinforced targets rather than filter targets.
[Uncertain: the motion-measurement method and threshold, and whether high-frequency jitter is filtered]

### [Data-Juicer 2.0](../models/Data-Juicer.md) ⚠️

Motion filtering is one of the few areas where DJ provides three parallel implementations, reflecting the team's emphasis on this dimension:
  · video_motion_score_filter — computes a motion score based on OpenCV optical-flow estimation (Farneback dense optical flow), filtering by interval. Fast, requires no GPU; the default option.
  · video_motion_score_raft_filter — computes a motion score based on the RAFT (Recurrent All-Pairs Field Transforms) deep optical-flow model. Significantly more accurate than traditional optical flow, especially for large displacements and occlusion scenarios, at the cost of requiring GPU inference.
  · video_motion_score_ptlflow_filter — based on the ptlflow (PyTorch Lightning Optical Flow) library, which can plug in dozens of optical-flow models (FlowNet, PWC-Net, GMA, FlowFormer, etc.), offering the greatest model-selection flexibility.
【Significance of the three coexisting】Users can trade off between "OpenCV, fast but coarse → RAFT, accurate but slow → ptlflow, freely selectable" according to compute budget and accuracy needs. This design of multiple implementation tiers for the same function is a plus point in data-processing frameworks — coarse large-scale filtering with OpenCV and fine filtering with RAFT is a natural two-stage usage.
【Filtering semantics】All three are bilateral interval filters (min_score / max_score): the lower bound removes static or near-static footage (a video-generation model that learns from too many static samples will tend to generate "PPT-style" motionless video), and the upper bound removes clips with excessively violent motion, jitter, or rapid cuts (which distort optical-flow estimation and complicate temporal modeling).
【Actual usage】[Uncertain] The final recipe of the official T2V case study does not include a motion-score operator — in the operator ranking during the Probe stage, video_aesthetics_filter, video_nsfw_filter, and video_frames_text_similarity_filter entered the candidate pool, with the latter two ultimately winning out. The paper does not report the probe-stage ranking of the motion-score operator in this case, or the reason it was not selected, so it cannot be determined whether the motion dimension simply provides no notable gain on this dataset or was never included as a candidate at all. No recommended threshold value is disclosed either.

### [Foley-Omni](../models/Foley-Omni.md) ⚠️

Motion filtering is implemented via a bilateral motion-score threshold interval of [0.1, 3.2], one of three visual-dimension metrics in the filtering stage.
【Lower bound 0.1】Removes static or near-static footage. This step is especially critical for V2A/V2ST tasks: a static frame contains no visual event, so the model cannot learn the temporal mapping from "visual action → sound event" from it; such samples become pure noise supervision.
【Upper bound 3.2】Removes clips with excessively violent motion. Jitter, fast panning, and high-frequency cuts distort Synchformer's temporal features, and the audio-visual correspondence in such clips is often chaotic (multiple overlapping events, sound sources leaving the frame).
【Difference from text-to-video generation models】Motion filtering in text-to-video generation models mainly serves the goal of "generating videos with a sense of motion" (removing static content to avoid the model generating still footage); Foley-Omni's motion filtering instead serves "ensuring audio-visual events can be aligned" — a different starting point but a similar threshold shape (both bilateral intervals).
[Uncertain] The specific computation method for the motion score (RAFT/UniMatch optical flow, frame differencing, or something else) and the normalization scale are not described, so the values [0.1, 3.2] are difficult to transfer or reuse directly.

### [Goku](../models/Goku.md)

Uses the RAFT optical-flow model to compute a motion score, with **bilateral thresholds** set (removing both near-static footage and overly violent/jittery motion); the thresholds are likewise tiered into three levels by resolution, tightening at each level:
  - 480p: 0.3 ≤ motion score ≤ 20.0
  - 720p: 0.5 ≤ motion score ≤ 15.0
  - 1080p: 0.5 ≤ motion score ≤ 8.0
Design rationale: the lower bound removes slideshow-style static videos and dead footage (ensuring the model learns dynamic degree, corresponding to VBench's dynamic degree, on which Goku scores 76.11); the upper bound removes handheld jitter, fast panning, and residual violent-transition footage that would cause training instability and motion blur. The higher the resolution, the lower the upper bound is pushed (20 → 15 → 8), because high-definition material tends to be used more for fine image-quality learning than for large-motion learning.
In addition, the motion score is not discarded after filtering — it is **reused as part of the caption** (see the annotation fields), becoming an inference-time controllable motion-intensity condition. This is a clever design in Goku: the same signal serves both as a filtering gate and as a control condition.

### [Hailuo / MiniMax Video](../models/Hailuo.md) ⚠️

[Uncertain] (no disclosure whatsoever). No mention of optical-flow computation, motion-score thresholds, static-frame removal, or jitter removal of any kind. Indirect clue: both Hailuo 02 and 2.3 make "complex human motion, physical plausibility, large dynamics" a core selling point, and 2.3 further emphasizes "maintaining smooth, natural, and precise complex body movements even under dynamic camera work," which strongly suggests some targeted screening or weighting toward high-motion-magnitude samples exists on the data side (removing static clips, retaining high-dynamic samples), but there is no first-hand description of the method or thresholds.

### [HunyuanVideo-Foley](../models/HunyuanVideo-Foley.md) ⚠️

The paper describes no motion-filtering step at all: no optical-flow computation, no motion-score threshold, no static-shot removal, no camera-shake detection.
【Why this work can skip it】The role of motion filtering in video-generation models is to ensure the training data carries sufficient dynamic information (avoiding the model learning to generate static footage). In the V2A task, "how much the picture moves" does not directly determine audio quality — an almost static frame (e.g., a slowly burning fireplace) can equally correspond to meaningful, continuous sound effects. Motion intensity is therefore not an effective filtering dimension for this task.
【AV-align actually fulfills a similar role】This work uses AV-align for temporal-alignment detection, a method that inherently depends on the time-axis correspondence between visual events (sudden picture-energy changes) and audio events (audio onsets). A completely static, event-free frame will naturally have a low AV-align score (there is no visual event to correspond to an audio event), and will therefore be indirectly eliminated at level 6. In other words, AV-align functionally covers part of the need to "remove content lacking dynamics," and is more aligned with the task than a plain motion score — what it filters out is not "footage that doesn't move" but "samples where the visual events and sound events don't line up," and the latter is what V2A actually needs to exclude. This is a reasonable design that substitutes a cross-modal criterion for a single-modal one, though the paper does not frame it this way. [Uncertain: this is an inference — the paper does not state that AV-align has this side effect]

### [HunyuanVideo](../models/HunyuanVideo.md) ⚠️

【Original version】Uses optical-flow estimation to compute motion magnitude, removing static and slow-motion videos; the SFT manual-annotation stage additionally makes dedicated human judgments on three items — "motion speed, action integrity, motion blur" — meaning motion quality is gated both automatically and manually.
【1.5】Removes "static or low-motion scenes" at the basic filtering level, without stating whether optical flow is still used; one of the SFT-stage selection criteria is "motion smoothness." Neither generation gives threshold values. [Uncertain]

### [InstructAV2AV](../models/InstructAV2AV.md) ⚠️

Motion filtering is implemented via CoTracker3 (Karaev et al., Meta's point-tracking model), which samples points on a grid layout and tracks them, computing the average motion magnitude of a clip; clips below a preset threshold are discarded.
【A distinctive tool choice】Using a point-tracking model rather than traditional optical flow (RAFT/UniMatch) or frame differencing to measure motion is relatively uncommon among the works surveyed here. The advantage of point tracking is that it gives a long-horizon trajectory-consistency measure rather than a frame-by-frame instantaneous motion measure, better reflecting "whether the clip contains sustained, trackable motion," and it is highly aligned with the downstream need — the editing task requires the target to be stably trackable and replaceable throughout the full 5 seconds, so point-tracking success is itself a proxy for editability. By comparison, optical-flow magnitude cannot distinguish "sustained displacement" from "random jitter."
【One-sided threshold】Only a lower bound is set (removing static content); no upper bound is mentioned. This differs from Foley-Omni's bilateral interval [0.1, 3.2] — this work does not reject violent motion, because the editing task does not depend on gentle motion magnitude; but from the acknowledged limitation that "3D spatial consistency and object permanence are hard to maintain under large camera movements," the lack of a motion upper bound may well be one of the data-side causes of this failure mode — if samples with large camera movements were also removed during data construction, the model might not have exposed this weakness (at the cost of narrower capability coverage).
【Motivation for removing static content】Unlike the generation-model motivation of "avoiding generating static footage," here static content is removed because: editing a static frame is too trivial a task (equivalent to image editing), providing no supervision signal for temporal consistency and offering no way to test audio-visual synchronization ability.
[Uncertain] The specific threshold value, grid sampling density, and normalization method for motion magnitude (pixels/frame, or normalized to frame size) are all undisclosed, so this setup is difficult to reproduce or transfer directly.

### [2026 Miscellaneous Joint Audio-Video Generation](../models/JAVG_2026_misc.md) ⚠️

Disclosure is sparse; only two items are relevant:
【ALIVE】Uses RAFT to compute optical flow for motion analysis — the original text reads "compute optical flow using RAFT." RAFT is the only optical-flow model named in this batch, consistent with earlier works such as Ovi and HunyuanVideo, indicating RAFT has become the de facto standard for this step. However, ALIVE does not state: the motion-score threshold, whether static clips are removed, whether overly jittery/high-motion clips are removed, or whether the motion score serves as a downstream condition or a basis for curriculum scheduling [all uncertain].
【NAVA】Lists motion score as one of the visual-quality operators, alongside aesthetics / sharpness / brightness. The computation method (whether optical flow is used, which model) and the threshold are both undisclosed [uncertain].
【OmniCustom】No motion filtering — this is reasonable, since its data consists entirely of single-person talking heads, where motion magnitude is naturally small and concentrated on the face; applying motion filtering would instead mistakenly harm valid samples. This suggests the necessity of motion filtering is strongly domain-dependent: general video generation must do it, while talking-head customization need not.
【StreamChar / CCL / Baton】Not mentioned [uncertain].
【ITS-JAVG】Does not involve training-side motion filtering, but its verifier's VideoReward-TA implicitly evaluates motion quality.
【Overall judgment】Compared with 2024–2025 papers that generally detailed motion-score thresholds and dual-ended removal of static/jittery content in depth, this batch of 2026 works devotes noticeably less ink to motion filtering, and some no longer even mention thresholds — reflecting that this step is now regarded as a standardized "known operation," with the narrative focus of the field having shifted to audio-visual alignment and semantic annotation.

### [Joint Audio-Video Generation Baselines Collection](../models/JavisDiT_baselines.md) ⚠️

【JavisDiT++ (the only one that clearly uses it)】Adopts "flow scoring" as one of three quality metrics for TAVGBench data filtering, following the Open-Sora implementation (Open-Sora typically uses UniMatch or RAFT to compute optical-flow magnitude as the motion score). Its purpose is to remove static clips with too little motion. The specific threshold, whether it also removes overly-motion/jittery clips at the other end, and whether the motion score serves as a training condition or sampling weight, are all unstated by the paper [uncertain].
【MM-Diffusion】No motion filtering. Notably, however, its dataset selection itself implicitly enforces a motion constraint: Landscape's 9 scene categories (explosion, fire, rain, splashing water, thunder, waterfalls, wind, etc.) are all natural phenomena with sustained dynamics, and AIST++ consists entirely of dance movements — i.e., motion sufficiency was ensured through domain selection rather than a filter, a "selection in place of filtering" approach typical of the small-dataset era.
【AV-DiT】No motion filtering.
【Harmony / UniAVGen】Neither mentions any motion filtering or optical-flow scoring [uncertain].
【Common thread】Apart from JavisDiT++, none of them use an optical-flow tool, related to the small data scale of this collection and the feasibility of manual quality control; once data scale reaches the million level (JavisDiT++'s 1.1M TAVGBench entries), motion filtering becomes a necessity.

### [Kling 3.0 / Kling Video 3.0 Omni](../models/Kling_30_Omni.md) ⚠️

Kling-Omni explicitly removes clips with "excessively low action semantic density," i.e., static/near-static footage is filtered out; jitter is listed separately as a removal criterion in temporal-quality evaluation. The same team's Koala-36M uses motion score as one of the input sub-metrics of the VTSS network, and explicitly models temporal motion quality via a 3D Swin dynamic branch, rather than relying purely on an optical-flow threshold. [Uncertain: whether optical flow (RAFT, etc.) is used to compute the motion score and its specific thresholds]

### [LTX-2](../models/LTX-2.md) ⚠️

Stage 3 of the pipeline is "Estimate Motion Level," and filtering is applied accordingly in the Filter Shots step. The paper states explicitly: "we actively remove videos with insignificant motion to ensure the dataset focuses on dynamic content," the rationale being that dynamic content better matches the model's target capability. The specific algorithm for estimating motion level (whether optical flow, frame differencing, or something else) and the threshold value are not stated, nor is it mentioned whether jitter/handheld shake is also removed. [Uncertain]

### [LongCat-Video](../models/LongCat-Video.md) ⚠️

Optical flow is the sole motion metric: "Motion information is evaluated using extracted video optical flow to assess video dynamics, enabling us to filter out clips with minimal motion features" — extracting video optical flow to assess dynamics and filtering out clips with minimal motion features accordingly (i.e., removing near-static samples). The report only mentions removing the "low motion" side, without stating whether high-frequency jitter/violent-shake samples are also removed, nor does it give an optical-flow score threshold. The SFT stage has a separate motion-quality metric involved in fine selection (this metric is given by an internally fine-tuned VideoAlign model, distinct from optical-flow statistics, and is a model-based score). The Avatar 1.5 online filtering includes two checks — "motion intensity" and "motion consistency" — plus frame-skip detection. [Uncertain: the optical-flow threshold value, whether jitter is filtered]

### [MOVA](../models/MOVA.md) ⚠️

The paper describes no independent motion-filtering step: no optical-flow computation, no motion-score threshold, no static-shot removal or handheld-jitter removal. Motion-related quality control is implicit in two places: (1) the DOVER technical and aesthetic scores themselves include perceptual assessment of temporal distortion, stutter, and blur; (2) the visual-annotation prompt's "LAW OF VISUAL DYNAMICS" requires annotators to detect all transitions and precisely record motion trajectories, speed changes, and visual rhythm, and specifies that "when there is no visual change, visual_description outputs null" — in theory this signal could be used to identify static clips, but the paper does not state whether the null signal is used for filtering. Since MOVA's training data is dominated by speaker dialogue, on-screen motion magnitude is naturally small, so the necessity of motion filtering is also relatively lower. [Uncertain]

### [Merged Entry: ① Mochi 1](../models/Mochi_MAGI_Motif.md) ⚠️

① Mochi 1: Undisclosed. The official material only emphasizes Mochi 1's motion-quality advantages at the capability level (30 FPS, high temporal consistency, realistic motion dynamics), and the CEO also stated the team was "focusing heavily on improving motion quality," but no corresponding data-side motion-filtering strategy is described. [Uncertain]
② MAGI-1 (the most detailed of the three, and the only one that separates foreground/background motion):
· Motion intensity: uses the RAFT optical-flow model; to reduce compute cost, all videos are first downsampled to 8 FPS before computing optical flow between adjacent frames; optical flow is computed at the pixel level, and the flow magnitude is averaged over all pixels of the entire clip to obtain the overall motion intensity.
· Foreground/background separation (a key improvement): the paper points out that the above approach underestimates the amount of motion when "the background is static but the foreground moves substantially"; to address this, a saliency-detection model (Zhao & Wu, 2019) is additionally run on every frame, using the saliency map to distinguish foreground from background regions and computing the average optical flow for each separately. This ultimately yields three motion statistics — overall motion intensity, foreground motion intensity, and background motion intensity — with upper and lower thresholds set for each, prioritizing the retention of clips with moderate motion intensity while avoiding both the overly-static and overly-dynamic extremes, to balance data quality against training difficulty.
· Camera-motion stability: since a large amount of footage is handheld, with severe shake that cannot be screened out by motion intensity alone, the team estimates camera stability by evaluating the consistency of optical flow between adjacent frames, and removes clips with unstable camera motion.
· Slideshow-style motion: targeting floating photos/banners common in screen recordings or slide presentations, the divergence of the full-frame per-pixel optical flow is analyzed for each frame; if the divergence stays low over an extended period, it is judged as slide movement and removed.
· Transitions: see shot_segmentation.
No threshold values are given for any of the above.
③ Motif-Video 2B: uses UniMatch to estimate optical flow between sampled frame pairs and compute a motion score for each video, applying bilateral trimming — removing both extremely low motion (usually static or near-static) and extremely high motion (often containing cuts, jitter, or unstable camera motion) at the two ends, retaining the middle band to match the smooth temporal dynamics required by the main training stage. Motion filtering is reapplied, with a stricter threshold, at every resolution-stage transition. In addition, the 720p SFT stage introduces a semantic motion-admission criterion: the VLM label action=Dynamic is used as a "dynamic-motion criterion" — i.e., on top of the low-level optical-flow statistics, a further layer of "does a large model judge this video's action to count as dynamic" semantic gate is applied. This is a concrete case within this group of motion filtering moving from "pixel statistics" toward "semantic judgment."

### [Movie Gen](../models/Movie_Gen.md)

Four steps in series: ① an internal static-video-detection model directly removes videos with no motion whatsoever; ② FFmpeg's VMAF motion score and motion vector are used to judge "reasonable motion" — the quantitative thresholds given in the appendix are motion score > 2.0, motion vector mean > 0.5 and < 7 (the lower bound removes slow-motion/near-static content, the upper bound removes overly violent motion); ③ PySceneDetect's Shot Boundary Detection is used to identify frequently-jittery camerawork — jittery videos get erroneously split into a large number of shots, so a threshold of "≥0.85 detected shots per second" is used for removal, motivated by the fact that jitter in training data causes the generated video to shake as well; ④ videos with special motion effects are removed, e.g., slideshow-style videos. The overall approach follows the low-motion automated-filtering idea from Emu Video (Girdhar et al., 2024). The SFT stage applies a stricter threshold to motion, with humans confirming that motion is "non-trivial and free of camera shake."

### [NVIDIA NeMo Curator](../models/NeMo_Curator.md)

The most detailed and parameter-transparent filtering step among the open-source frameworks, implemented as two chained stages:
【MotionVectorDecodeStage】Does not perform full optical-flow computation; instead it directly decodes lightweight motion vectors from the compressed bitstream and computes the motion score from them — a notable cost optimization that avoids running optical-flow networks on PB-scale data. Parameters: target_fps defaults to 2.0 (sampled frames per second), target_duration_ratio defaults to 0.5 (only half of the clip's duration is analyzed to save compute), num_cpus_per_worker defaults to 4.0, num_gpus_per_worker optionally 0.5.
【MotionFilterStage】Applies threshold filtering. Produces two metrics: motion_score_global_mean (global average motion score, default threshold 0.00098) and motion_score_per_patch_min_256 (minimum motion score across 256 spatial patches, default threshold 0.000001). The intent of the dual-metric design: the global mean guards against "overall staticness," while the per-patch minimum guards against pseudo-dynamic samples where "most of the frame is static and only a small local object moves."
【Cosmos WFM production-version enhancement】There is also a motion classifier based on a ViT architecture trained on optical-flow sequences, which, besides removing static content, can also remove out-of-control shots with erratic camera motion, and labels the type of camera motion (pan / zoom / tilt) — i.e., motion filtering doubles as "camera-motion annotation," producing labels usable for camera-control training. This ViT classifier's weights are not provided in the open-source version.

### [OmniHuman Dataset + OHBench](../models/OmniHuman.md) ⚠️

【Tool】UniMatch — a unified optical-flow/stereo-matching/depth-estimation model, used here for optical-flow estimation to "analyze motion quality." Compared with the more traditional RAFT or frame-differencing approaches, UniMatch is a more recent choice.
【Filter targets: both upper and lower bounds set】Explicitly removes three categories: static, shaky, and abnormal clips. This is an improvement over UniTalking (which only removes static content, with no upper bound) — OmniHuman sets both a motion lower bound (preventing photo-like static frames or subjects that never move) and an upper bound (preventing handheld jitter and violent shaking that cause motion blur and unlearnable samples). For a dataset sourced from YouTube material, removing shaky samples is necessary — a large amount of vlog and outdoor content is handheld.
【Coordination with semantic-consistency filtering】The same stage also has a "semantically inconsistent frame" filter combining perceptual hashing with SigLIP cosine similarity. The two have a clear division of labor: optical flow governs "whether pixel-level motion is reasonable," while SigLIP governs "whether the semantic content is coherent" — for example, a clip with smooth optical flow but where the subject is entirely replaced midway cannot be caught by optical flow but can be caught by SigLIP. This dual "low-level motion + high-level semantics" temporal-consistency check is uncommon among comparable works.
【Thresholds】The upper/lower bound values for optical-flow magnitude, the SigLIP cosine-similarity threshold, and the perceptual-hash Hamming-distance threshold are all undisclosed. [Uncertain]
【Correspondence with evaluation】OHBench's global layer has a Dynamic Degree (DD) metric, which uses RAFT optical flow to measure the motion intensity of generated videos — i.e., the motion-filtering dimension of the training data corresponds to the motion-diagnosis dimension of the evaluation (even though one uses UniMatch and the other RAFT). In the LTX-2 fine-tuning experiment, DD improved by 10.7%, indirectly indicating that data after motion filtering does let the model learn more thorough motion.

### [Open-Sora Series](../models/Open-Sora.md)

【Open-Sora 1.x】Uses the **UniMatch (GMFlow) optical-flow model** to compute a dense optical-flow score (`tools/scoring/optical_flow/inference.py`, output column `flow`) — the higher the score, the greater the motion — used to remove static/near-static clips; the optical-flow result is also used for **camera-motion detection** (identifying pan left / zoom in, etc.), and high-confidence camera-motion labels are appended into the caption. The camera-motion detection module is separately open-sourced under tools.
【Open-Sora 2.0】Switches to **FFmpeg libavfilter's VMAF motion score** (much faster than an optical-flow model, and can be computed as a byproduct within the transcoding pipeline), with **bidirectional removal**: both extremely low motion scores (static footage) and extremely high ones (violent shake/chaotic cuts) are discarded. The motion score is also **appended to the caption text**, allowing the motion magnitude of the generated video to be controlled via the prompt at inference time.
【Open-Sora Plan】Uses **LPIPS frame-skip perceptual distance** in place of optical flow as the motion metric (far cheaper to compute than optical flow), retaining the interval **0.001 ≤ score ≤ 0.3**: below 0.001 is treated as essentially static, and above 0.3 is treated as "exhibiting noticeable jitter and flicker." The authors state that this threshold was verified for acceptable precision via **manual spot-checking of 2000 videos**. And as mentioned earlier, after caption cropping a **second pass of motion re-checking** is run (retention rate drops from 44% to 42%).
Common to all three: motion filtering is bidirectional (removing both static and jittery content), and the motion score is ultimately used as a controllable condition rather than merely a filter, in all three cases.

### [Ovi](../models/Ovi.md) ⚠️

Uses the RAFT optical-flow model (Teed & Deng, 2020) for two purposes: (1) filtering out static videos, i.e., clips lacking effective motion are directly removed; (2) computing and saving a motion score for the retained clips as an annotation/filtering signal.
【Threshold】The optical-flow-magnitude threshold for determining staticness, the motion-score value range, and the high/low truncation strategy are all undisclosed [uncertain].
【Jitter removal】The paper only mentions removing the "static" end; it does not state whether the overly-strong-motion end (severe camera shake/fast panning, etc.) is also removed [uncertain].
【Downstream use of the motion score】The paper does not state whether it is used as a training condition, sampling weight, or basis for curriculum scheduling [uncertain].

### [Script-a-Video](../models/Script-a-Video.md) ⚠️

The paper describes no motion-filtering step at all: no optical-flow computation, no motion-score threshold, no static-shot removal, no jitter/shake detection. [Uncertain]
In this work, motion information appears not as a "filtering dimension" but as an "annotation dimension":
1) The camera field of the Shot stream explicitly records camera movements, turning camerawork into a describable, controllable semantic field;
2) The visual_description field of the Shot stream requires an objective, chronological narrative of core actions, and anchors micro-actions to the global timeline via intra-description timestamps — i.e., motion is "structured description" rather than "scored filtering."
There is a motion-related observation on the evaluation side: Intra-Shot Subject Consistency (frame-to-frame average cosine similarity of DINOv2 [CLS] features), for which the baseline LTX-2-AV scores as high as 0.87, but the paper explicitly notes this is illusory — the high score stems from the model failing to inject the reference feature and generating near-static content, with minimal frame-to-frame variation ("trivially static content with minimal frame variation"). This work's full pipeline maintains "lower" scores of 0.59-0.62, which instead correspond to genuinely dynamic content. This is an important warning about "static content producing artificially inflated consistency scores," relevant for data-side motion-filtering design.

### [Seedance 2.0 and Seedance 1.5 pro](../models/Seedance_20_Seedance_15_pro.md) ⚠️

Seedance 1.0 removes overly jittery and predominantly-static clips during the pretraining filtering stage; in the Continue Training stage, "motion evaluators based on optical flow" together with the aesthetic scorer are used to select a subset with higher aesthetic quality and richer motion dynamics, and the report states that richer motion dynamics make the model's generations more natural and fluid. Seedance 1.5 pro lists "motion expressiveness" as one of the three priority goals of the data pipeline, and introduces a vividness metric on the evaluation side, explicitly opposing trading motion for stability via slow motion. The specific motion-score threshold is undisclosed. [Uncertain: threshold value]

### [SkyReels Series](../models/SkyReels.md) ⚠️

【SkyReels-V2】Motion is a core concern of this series (the paper argues that existing models compromise on "motion dynamics vs. image quality"). Data side: basic filtering removes static footage and camera-shake samples; on the annotation side, a dedicated "classification-based motion-recognition captioner" was trained — through classification-labeling of motion, producing "93K high-confidence samples + 16K motion-axis-balanced synthetic data" used to train this captioner, achieving 89% single-type motion-prediction accuracy on a 15K-video test set. The motion information then feeds into the camera-motion field of the structured caption, and serves as an optimization target (motion consistency and smoothness) during the reinforcement-learning stage.
【SkyReels-V4】Lists "motion quality" as one of three filtering dimensions, and introduces "motion diversity" in the balancing step: key motion patterns are defined for each subject or scene category, and quotas are balanced by pattern to prevent any given subject category from being represented by only a single action.
Neither generation discloses the specific optical-flow/motion-score algorithm or threshold values. [Uncertain]

### [Sora 2](../models/Sora_2.md) ⚠️

Completely undisclosed. No mention of optical-flow computation, motion-score thresholds, static-shot removal, or jitter/handheld-shake removal strategy. Given the heavy publicity around the model's performance on physical dynamics (gravity, momentum, collisions), it is speculated that some form of motion-quality screening exists, but there is no evidentiary basis for this. [Uncertain]

### [SpeakerVid-5M](../models/SpeakerVid-5M.md)

SpeakerVid-5M does not apply traditional optical-flow/motion-magnitude threshold filtering; instead it turns motion intensity into a retrievable annotation dimension, used as a filtering criterion only in the HQ subset step:
【Motion-intensity annotation method】Uses Qwen2.5-VL for multi-persona scoring — three prompts with different personas are designed, each scoring the same video on a 1–5 scale from a different viewpoint: (1) an expert viewpoint, assessing movement amplitude and the frequency of body movements; (2) an audience viewpoint, assessing the emotional swings and audience reactions reflected by body language; (3) an annotation-expert viewpoint, assessing gesture usage and interaction frequency. Outliers are removed and the average score is taken as the final motion-amplitude label. This is a typical case of using an MLLM in place of an optical-flow scorer, using a multi-persona ensemble to reduce the variance of any single score.
【Use as a filtering criterion】Enabled only when constructing the HQ / SFT subset: motion score > 2 (on the 5-point scale) is required for inclusion, i.e., excluding near-static, lifeless clips. No motion floor is applied to the full dataset.
【Indirect control over static content/jitter】Motion-blur filtering (face/hand Laplacian variance < 0.1 causes removal) removes blurred frames caused by violent shake or fast motion; YOLO-tracking-based spatiotemporal cropping keeps the subject stable within the frame, indirectly suppressing strong camera shake.
【Not used】No optical-flow computation, no RAFT/UniMatch-type motion score, no dedicated static-shot detector. Given that the corpus itself consists of interview/dialogue-style close-up footage of people, motion magnitude is naturally limited, making traditional motion filtering less necessary.

### [Step-Video-T2V](../models/Step-Video-T2V.md)

Stage 3 is dedicated to "Video Motion Assessment": the Farneback dense optical-flow algorithm (a classic OpenCV algorithm) is used to compute the optical-flow field of a clip, deriving three scalar metrics — Motion_Mean (average motion magnitude), Motion_Max (peak motion magnitude), and Motion_Min (minimum motion magnitude). The combination of the three values is used to identify and remove two categories of samples: nearly static clips (insufficient motion information, prone to causing the model to generate static footage) and clips with overly violent/jittery motion. The specific threshold is undisclosed.
The derivative model Step-Video-TI2V upgrades this motion score from a "filter" to a "controllable condition," with clearer methodological detail: the video is sampled once every 12 frames, converted to grayscale, and the optical-flow magnitude is computed; the batch with the highest magnitude values is then averaged to obtain the video's motion score. After extracting motion scores for all training data, on the one hand a threshold is set to filter out videos with excessively high or low motion, and on the other hand the motion score is injected as an explicit condition into the model, enabling "controllable motion magnitude" at inference time. This is a clear case of "a data label directly converted into a controllable condition," of reference value to this survey.

### [UniTalking](../models/UniTalking.md) ⚠️

Motion filtering exists only as the single phrase "filter out videos that are static," the first of the three rules in the video-filtering stage. The paper does not disclose: which motion metric is used (optical-flow magnitude / frame differencing / RAFT / a motion-score model), the threshold value, or whether a motion upper bound is set (i.e., whether overly violent motion or camera-shake clips are removed).
【Inference of a lower-bound-only, no-upper-bound design】From the wording, only "static" content is removed; there is no mention of removing jitter/violent shake, implying only a motion lower bound with no upper bound. For talking-head video this trade-off is reasonable — the target scenario itself has limited motion magnitude, and the main risk is "photo-like static footage" (the person never moves, or the video is effectively a static image + audio track); removing such samples avoids the model learning a mouth that doesn't move. But the lack of an upper bound means handheld-jitter and fast-camera-motion samples will not be removed.
【Upstream inheritance】OpenHumanVid's filtering dimensions already include a motion item, so this subset's motion quality has upstream assurance.
【Relationship to evaluation】None of the paper's evaluation metrics (Sync-C / Sync-D / subjective video quality / timbre similarity / WER) are motion-related, and no ablation of motion filtering is performed, so the actual contribution of this step cannot be assessed. [Uncertain]

### [UniVerse-1](../models/UniVerse-1.md) ⚠️

The paper describes no independent motion-filtering step: no optical-flow computation, no motion-score threshold, no static-shot removal, no handheld-jitter or camera-shake removal. Motion-related quality control is only implicit in the DOVER score (whose technical dimension includes perceptual assessment of temporal distortion, stutter, and blur).
Notably, on the evaluation side motion intensity is instead treated as a core observable: Motion Score (MS) is the first metric in the Verse-Bench main table, and the ablation results show this metric is highly sensitive to the training strategy (full model 0.20, w/o LQLS 0.38, w/o INSS 1.10) — here a large deviation in the MS value does not mean "richer motion"; it appears together with a simultaneous drop in aesthetic score and ID consistency, more likely reflecting spurious motion caused by picture instability or artifacts. This indirectly indicates that data-side problems from the lack of motion filtering ultimately transform into a need for compensation at the training-strategy level. [Uncertain]

### [Unison](../models/Unison.md) ⚠️

The paper describes no motion filtering whatsoever: no optical-flow computation, no motion-score threshold, no static-shot removal, no jitter/shake removal, no motion-magnitude bucketing.
【A clarification related to the actual meaning of "motion" in the paper】Unison's title and body text use the word "motion" frequently, but what it refers to is "the temporal correspondence between a character's motion and the audio in the generated output" (motion-audio synchronization), not "motion-intensity screening of the training data." What the paper cares about is whether note onsets during piano performance correspond to finger movements, and whether an impact sound corresponds to the instant of visual collision — an alignment problem, not a motion-quality problem. All motion-related mechanisms are therefore at the training-strategy level (bidirectional cross-modal forcing) and the architecture level (three-frame-window cross-attention); there is no corresponding facility at the data level.
【Indirect motion assurance】Among the five data sources, HDTF/VFHQ/CelebV-Text are all talking-head videos, naturally containing lip and head movement; OpenHumanVid centers on human activity; VGGSound's audio-visual events are mostly accompanied by visible motion. The material selection itself ensures the presence of motion, without needing extra filtering of static clips. This is an inference. [Uncertain]
【No motion metric on the evaluation side either】None of the eleven metrics in Table 1 is a Motion-Score-type motion-intensity metric (compare with UniVerse-1's Verse-Bench, which lists MS as the first item in the main table), indicating the team did not treat motion magnitude as a dimension requiring observation.

### [Veo 3 / Veo 3.1](../models/Veo_3_Veo_31.md) ⚠️

[Uncertain] Motion filtering is not mentioned at all. The official material does not disclose optical-flow computation, motion-score thresholds, static-shot removal, or jitter/handheld-shake removal strategy. One can only speculate, from capability descriptions such as "the model simulates real-world physics well" and "high-precision motion representation," that some form of motion-quality screening exists, but there is no direct evidence.

### [Vidu S1](../models/Vidu_S1.md) ⚠️

There are two motion constraints pulling in opposite directions: on one hand, "camera stability" filtering retains only static or slowly-moving shots, removing large camera movements/jitter (jitter is also removed under the visual-quality items); on the other hand, "interactivity" filtering requires the subject itself to have clearly discernible actions or behavior, removing samples that are entirely static with no action. In short: "the camera should be steady, and the subject should move." No mention of using optical flow or specific motion-score thresholds [uncertain about the specific method and values].

### [Wan 2.5 / 2.6 / 2.7](../models/Wan.md) ⚠️

Wan 2.1 elevates motion quality to an independent filtering-funnel level alongside visual quality, with the goal of "selecting videos that are natural, complete, and have significant motion, while avoiding staticness and jitter."
【Six-tier grading with differentiated handling】Optimal motion (retained, highest priority) / medium-quality motion (retained, to ensure motion diversity and help the model understand spatiotemporal relationships) / static video (chat/interview type, high image quality but little motion → identified separately and down-sampled) / camera-dominant motion (aerial footage, etc., near-static-image-like → sampling priority significantly reduced) / low-quality motion (too many subjects, severe occlusion, unclear subject → excluded) / jittery footage (amateur handheld, motion blur, foreground/background hard to distinguish → systematically excluded).
【Key point】Static video and camera-dominant motion are not deleted but "down-sampled," reflecting the consistent philosophy of "preserving distribution without dropping concepts"; only the low-quality-motion and jitter tiers are actually deleted.
【Method】Wan 2.1 does not state the specific algorithm used for the six-tier grading (whether optical flow, whether a trained classifier), nor does it give any threshold values. Corroborating evidence from the same family: Wan2.2-S2V uses UniMatch to predict optical flow and compute a motion score; Wan-Dancer uses SEA-RAFT to generate optical-flow masks (used for the loss function rather than for filtering); Wan-Bench uses RAFT optical-flow magnitude on the evaluation side to measure large-motion generation ability. [Uncertain: thresholds and algorithm]

### [Audio-Video Generation Evaluation Benchmarks Collection](../models/av_benchmarks.md) ⚠️

None of the five adopts optical-flow or motion-score thresholds for filtering [uncertain], but several functionally-equivalent alternative mechanisms exist:
【AV-SyncBench】The Local Jitter challenge task (local jitter perturbations at three severity tiers of 30–700 ms) and the Global Speed Change task (10 speed tiers spanning 0.8×–1.25×) are essentially artificially constructing negative samples with "a mismatched motion-audio time base," which can conversely serve as detection ideas for speed-change/frame-drop anomalies in training data.
【PhyAVBench】The "periodic motion / non-periodic motion rhythm consistency" test point under the Time and Causality dimension, and the Transient Synchronization (onset and offset of sound) test point, examine precisely the causal consistency between motion and sound — more semantic than a plain motion-magnitude threshold.
【VABench】No motion filtering; the static-image problem on the generation side, for the models under test, is indirectly penalized by the Visual Realism and Artistry scores.

### [Video Caption Model Ecosystem](../models/caption_models.md)

The captioner ecosystem intersects with motion filtering at three points:
【Captioner output providing motion information for use in filtering】Tarsier2 naturally describes camera-motion types (zoom in / pan right, etc.); the Goku paper explicitly states this is a key advantage of choosing it for video-level captioning — no need for an additional camera-motion annotation module to obtain camera-language labels.
【Division of labor between dedicated motion classifiers and captioners】The mainstream approach is to not hand motion recognition to a general VLM but to a lightweight classifier instead: Movie Gen trains a 16-class camera-motion classifier, with high-confidence predictions appended as a prefix to the caption; LongCat-Video's camera motion (pan/tilt/zoom) is handled by a separately trained lightweight classifier rather than a VLM (presumably for cost and accuracy reasons); HunyuanVideo has its own dedicated, in-house camera-motion classifier (upgraded in version 1.5 to dual granularity — clip-level plus temporal-level); SkyCaptioner-V1's camera-motion sub-expert is classification-based, trained on 93K high-confidence human-annotated samples plus 16K motion-axis-balanced synthetic data, achieving 89% single-type motion accuracy on a 15K test set.
【Motion filtering itself (optical-flow/motion-score thresholds, removal of static and jittery content) sits upstream of captioning】Foley-Omni uses a motion∈[0.1, 3.2] threshold; InstructAV2AV uses a CoTracker3 motion threshold; Open-Sora Plan uses an LPIPS upper bound of 0.3 (above which jitter and flicker appear, a conclusion verified via manual spot-checking of 2000 videos). All of these run before the captioner.
【Ecosystem judgment】"A dedicated classifier for camera-motion recognition, a VLM for content description" is a stable division-of-labor paradigm across 2024–2026, because a VLM's discrimination accuracy for camerawork is unstable and hard to quantitatively verify, whereas a classifier can give a confidence score usable for filtering. SkyCaptioner-V1's approach (distilling the classifier's results back into a unified 7B captioner) is an attempted fusion of the two routes, and its film-industry-specific fields achieve an average accuracy of 76.3% (shot type 93.7%, camera angle 89.8%, camera position 83.1%, camera motion 85.3%), significantly exceeding larger general models such as Qwen2.5-VL-72B.

### [Geometric/Structured Annotation Dataset Collection](../models/geometric_datasets.md)

Motion filtering is the key point that distinguishes these four datasets from conventional video-generation datasets — whereas the conventional pipeline removes excessive motion, geometric-annotation datasets instead remove those with "insufficient motion." SpatialVID: filters using VMAF combined with three camera-trajectory metrics — MoveDist (total displacement distance), RotAngle (cumulative rotation angle), TrajTurns (number of direction changes) — and uses an acceleration-based detector to identify and remove abrupt, non-physical motion jitter; quantitative evidence: 80%+ of Panda-70M videos cannot be reconstructed by MegaSaM due to insufficient parallax/motion, so it was necessary to actively retrieve high-motion material such as walk/tour/drone footage; ultimately 80% of clips in the library have curved or turning trajectories. SceneScribe-1M: uses "motion diversity" as the main axis of initial screening — clips whose motion intensity Qwen2.5-VL-72B judges as unknown are removed directly, and the source pool is substantially reduced due to motion requirements; the reason for choosing MegaSaM is precisely its robustness in scenes with limited camera parallax. WildWorld: actions are given directly by game inputs, requiring no inference of motion from pixels; static frames can be precisely identified via the action ID. Action100M: no optical-flow-level motion filtering; the variance-minimization clustering in the semantic-segmentation stage naturally merges static passages into long nodes.

### [Video Generation Post-Training Data Strategy](../models/post_training_data.md) ⚠️

The anchor paper describes no motion filtering on the data side, but has a dedicated Motion Quality reward model on the reward side (assessing the realism, smoothness, and coherence of motion dynamics, suppressing jitter, discontinuous motion, and temporally inconsistent object transitions), and reports in the results that motion quality is one of the two most significant gain dimensions from RLHF [data-side uncertain].
【Cross-reference】
· Allegro SFT: UniMatch optical-flow motion-score interval [1.0,100];
· HunyuanVideo original-version SFT manual criteria include motion speed, action integrity, and motion blur;
· Motif-Video 2B SFT explicitly requires action=Dynamic;
· SkyReels-V2's RL objective explicitly focuses on motion quality (dynamic consistency and smoothness) rather than general aesthetic preference, and the automatic side of its preference data consists precisely of corrupted samples generated by "applying controlled distortions to real videos";
· LongCat-Video's Motion Quality reward model has a design worth borrowing: it fine-tunes VideoAlign as a base model on internally annotated data, with grayscale video as input (color removed to force the model to evaluate only motion without being distracted by color/aesthetics) — a direct engineering means of decoupling the motion reward from the aesthetic reward;
· Cosmos-Predict 2.5 lists a "high motion" domain (1.0M samples) separately for dedicated SFT.

### [Merged Survey of Mainstream Video Pretraining Datasets: Panda-70M, InternVid, Koala…](../models/pretraining_datasets.md) ⚠️

**All seven apply motion filtering, and all are bidirectional (removing both static and jittery/violent content), but the measurement tools split into three camps**:
【Optical-flow camp】
- **LVD-2M**: **RAFT**, computed on adjacent sampled frames at a temporal rate of **2fps** and spatial resolution of **520×960**, taking the spatiotemporal average of the optical-flow magnitude, **discarded if <20** (the official 100-sample test set's minimum value of 20.19 perfectly corroborates this threshold; sample mean 51.91, maximum 201.33; overall dataset mean 47.8). **This is the only dataset among the seven that gives a verifiable absolute optical-flow threshold.** The goal is to remove static scenes and "talking heads in front of a static background." The authors also note a limitation of optical flow: **handheld videos with jitter have a high optical-flow score but no meaningful motion** — precisely the motivation for adding an extra layer of PLLaVA semantic judgment.
- **MiraData**: **RAFT** inter-frame optical flow used as a "motion intensity" screening item, computed at a uniform 2fps, with an **undisclosed threshold**. [Uncertain] On the evaluation side it additionally proposes **Tracking Strength (the average displacement of CoTracker tracking points relative to the first frame)** as a correction for optical-flow dynamic degree — Figure 5 of the paper gives a counterexample: for a pair of videos, optical-flow dynamic degree is 1.2 vs. 0.7 (wrong ranking), while Tracking Strength is 4.1 vs. 11.8 (correct ranking), showing that **optical flow can mistake camera shake and repetitive local motion for high dynamics**.
- **OpenVid-1M**: **UniMatch** optical flow, removing clips with the highest and lowest motion-difference scores at both ends, retaining "moderate" motion. Threshold undisclosed. [Uncertain]
- **UltraVideo**: **RAFT**, sampled at intervals then globally averaged, retaining **0.1 ≤ score ≤ 100**.
【Perceptual/feature-distance camp】
- **Panda-70M**: does not use optical flow, instead using **ImageBind first-to-last-frame feature distance** — a distance **≤0.15** is treated as too little motion and discarded (while a distance >1.0 is treated as excessive semantic change and also discarded), i.e., the same feature distance simultaneously serves both a "motion lower bound" and a "consistency upper bound" role. It also uses "distance from the preceding clip >0.3" for diversity deduplication.
【Learned-scoring camp】
- **Koala-36M**: motion_score is one of three input sub-metrics for VTSS (CSV-measured range **0.01–267**), **with no standalone threshold**; the VTSS network makes an overall judgment. Its manual annotation guideline's definition of motion is worth quoting: **"the motion region must cover more than 30% of the frame, otherwise the score is lowered for insufficient dynamics,"** and it distinguishes "amateur camera shake" from "professional camerawork," penalizing and rewarding them respectively.
- **InternVid**: only a qualitative description of "filtering out clips that are static or have extreme dynamics (such as photo-album browsing)," **with no model and no threshold**. [Uncertain]
- **LVD-2M's human-verification results** can serve as a cross-sectional yardstick for the effectiveness of motion filtering (40 clips/dataset, 3-tier rating scheme, "very dynamic / medium / not dynamic"): **LVD-2M 30.0%/62.5%/7.5%**, HD-VG 20.0%/37.5%/42.5%, InternVid 15.0%/60.0%/25.0%, Panda-70M 7.5%/67.5%/25.0%, WebVid 7.5%/42.5%/50.0%. **Roughly a quarter to a half of the clips in Panda-70M and WebVid were each judged by humans as "not dynamic,"** direct evidence that "large-scale datasets generally lack motion."
【Common gap】None of the seven **write the motion score into the caption or training condition as a controllable condition** (compare with Open-Sora, which appends the motion score into the caption, and Koala-36M, which injects three scores via AdaLN — note the latter is a design of its **generation model** rather than the dataset's own annotation strategy; but Koala-36M's CSV does release the three scores for downstream condition injection, making it the only one of the seven that provides a structured score column usable for condition injection).

## Deduplication Methods (Exact Deduplication and Embedding-Based Semantic Deduplication Recorded Separately)

`deduplication` · Detail level: brief

### [Allegro](../models/Allegro.md) ⚠️

The paper describes no deduplication step whatsoever — neither exact deduplication (hashing/pHash) nor embedding-based semantic deduplication or near-duplicate clustering. This is a clear gap in the pipeline relative to its otherwise detailed disclosure, given that the data sources include mutually overlapping public datasets such as WebVid, Panda-70M, HD-VILA, HD-VG, and OpenVid-1M — the risk of cross-dataset duplication genuinely exists but is not discussed in the paper. [Uncertain]

### [Apollo](../models/Apollo.md) ⚠️

The paper does not mention any deduplication step at all: neither hash/fingerprint-level exact deduplication nor embedding-based semantic deduplication. At a scale of 81 million entries, apparently sourced from short-video platforms, the risk of duplicate content (repeated reposting of the same material, templated re-creations, overlapping slices of the same video) objectively exists, and the paper's scene segmentation would naturally produce multiple clips from the same long video, inherently introducing near-duplicates — but the paper offers no discussion of this at all. [Uncertain]

### [CineDance / CineDance-1M](../models/CineDance.md) ⚠️

The paper describes no deduplication step — neither hash/fingerprint-based exact deduplication nor embedding-based (e.g., CLIP/ImageBind feature) semantic deduplication; the three-stage pipeline in the main text contains no deduplication step.
【Speculative mitigating factors】① The raw material comprises only 45,181 long videos, a not-large order of magnitude, so the probability of exact duplication between long films is relatively low; ② the upstream sources MiraData / LVD-2M / Koala-36M have each already undergone their own deduplication; ③ narrative sequences are composed of consecutive shots from within the same long film, so different sequences within the same film naturally do not overlap.
【Potential risk】No handling is described for overlap between cross-source datasets (MiraData and Koala-36M may both include the same film). This is overall a disclosure gap. [Uncertain]

### [CogVideoX](../models/CogVideoX.md) ⚠️

[Uncertain] The paper does not mention any deduplication step at all — neither exact deduplication (hashing/fingerprinting) nor embedding-based semantic deduplication or near-duplicate detection (such as the SSCD copy-detection embedding), let alone semantic clustering and long-tail resampling. This is one of the most notable gaps in the CogVideoX data pipeline relative to contemporaneous work.
The only thing indirectly related to "redundancy" is the negative labels Editing and Lack of Motion Connectivity (which can remove videos formed by repeatedly splicing the same material), but their design purpose is to ensure motion authenticity, not deduplication.

### [Cosmos-Predict2.5](../models/Cosmos-Predict25.md) ⚠️

Semantic deduplication is an independent sixth stage, with a fairly complete design that specifically accounts for incremental-scaling scenarios:
(1) Clustering: clips are first assigned to clusters using embedding-based similarity, reducing global pairwise comparison to within-cluster comparison — the key to scalability;
(2) Within-cluster comparison: clips within a cluster are compared pairwise to detect semantically similar content;
(3) Arbitration rule: within a duplicate group, the highest-resolution version is retained, the stated rationale being that a higher resolution preserves finer visual detail and provides a richer training signal — an explicit "pick the best among the same content" rather than "keep one at random" strategy;
(4) Online incremental deduplication: every newly ingested clip is compared against previously retained clips, with tie-breaking favoring older, higher-resolution clips. This design lets the dataset keep growing without needing a full rerun of deduplication, while maintaining semantic consistency across the whole corpus;
(5) Infrastructure support: the open-source Milvus vector database is used for embedding retrieval, supporting both semantic-similarity search and caption-level text-embedding retrieval.
The paper only describes embedding-level semantic deduplication and does not separately describe an exact-deduplication (MD5/byte-level hashing) step — presumably handled by upstream collection or the Cosmos Curator engineering implementation, which the paper does not describe. [Uncertain: the embedding model used, the similarity threshold, whether exact deduplication is also present]

### [Data-Juicer 2.0](../models/Data-Juicer.md) ⚠️

Deduplication capability is unevenly distributed: very strong on the text side, weaker on the video side.
【Video deduplication — exact deduplication only】
  · video_deduplicator — "uses exact matching on videos for document-level deduplication," i.e., exact deduplication based on video-file content hashing, which can only identify byte-identical files and cannot identify near-duplicates after transcoding, cropping, watermarking, or resolution changes.
  · ray_video_deduplicator — a Ray-distributed version of the above, with the same logic but parallelizable across nodes, suited to TB-scale data.
  [Uncertain] DJ does not provide a video-embedding-based semantic-deduplication operator (e.g., CLIP/VideoMAE features + ANN retrieval), nor does it provide perceptual-hash (pHash/dHash) near-duplicate detection. This is a clear shortcoming for video-generation data cleaning — near-duplicates in web-video corpora (the same material from different sources, different compression, added intros/outros) are typically far more common than byte-exact duplicates. When building large-scale video corpora with DJ, the semantic-deduplication step needs to be supplemented independently.
【Text-side deduplication — strong engineering capability】
  · MinHash locality-sensitive-hashing deduplication (Ray-distributed version): implemented using a "load-balanced union-find" approach, achieving a 3.3x speedup over the native Ray implementation; a real-world test with 8 Ray nodes processing 5TB of data took 2.8 hours. The paper compares this against NVIDIA NeMo Curator (which used 64 A100 GPUs to process 1.1TB in 1.8 hours), intending to demonstrate that DJ achieves comparable cost-effectiveness using pure CPU. Scalability data: when data volume increases 5x, time increases 4.02–5.62x (near-linear); when core count doubles, time drops to 58.9%–67.1% of the original.
  · v1.5.2 adds cross-document line deduplication, refining the granularity down to the line level.
  · There are also multiple other deduplication operators — SimHash, exact hashing, document-level/paragraph-level, etc. — with 10 Deduplicator classes in total.
【Image side】Provides an image-hash-based deduplication operator.
【Overall judgment】DJ's deduplication strength lies in "distributed engineering capability" rather than "video semantic-perception capability" — it solves "how to run deduplication efficiently across 10k cores," not "how to judge whether two videos are semantically duplicate."

### [Foley-Omni](../models/Foley-Omni.md) ⚠️

[Uncertain] The paper does not mention any deduplication step at all — neither exact deduplication (file hashing, perceptual hashing pHash) nor embedding-based semantic deduplication is described.
Potential risk points: training data overlaps across task groups — SpeakerVid appears in both VisualTTS (within the 1,980h group) and V2ST (within the 216h group); AudioSet is used for TTM, while VGGSound itself is a subset derivative of AudioSet (same-source YouTube videos), so there is a possibility of duplicate sample counting or even train-eval contamination. In addition, the paper states that V2ST-Bench's 300 clips are "drawn from curated audiovisual pool" (drawn from the same cleaned data pool); if a strict removal from the training set was not performed, there is a risk of evaluation-set leakage — the paper only states manual review was performed, without explicitly declaring no overlap with the training set. This is a notable gap in methodological rigor worth attention.

### [Goku](../models/Goku.md) ⚠️

Only one mechanism is described: **approximate deduplication based on perceptual hashing**. The approach computes a perceptual hash of the key frame of each clip and compares them pairwise; if two clips' hash values are close, they are judged duplicates, at which point **the one with the higher aesthetic score is kept** and the other discarded. The paper explicitly limits this deduplication to occur "between clips cut from the same source video" (clips from the same source video), mainly addressing the high content overlap between adjacent clips after splitting a long video, and duplicate/replay shots.
【Not covered】Cross-source/global exact deduplication (e.g., file-level MD5, full-corpus frame-hash comparison), embedding-based semantic-level deduplication (e.g., CLIP/DINOv2 feature clustering deduplication), and deduplication on the image side (the 100-million-sample LAION set) are not described in the paper. This is a relatively weak link in Goku's data pipeline — DINOv2 features are already extracted during the segmentation stage, yet are never used for semantic deduplication. [Uncertain] (whether an undisclosed global semantic-deduplication step exists)

### [Hailuo / MiniMax Video](../models/Hailuo.md) ⚠️

[Uncertain] (no disclosure whatsoever). Does not state whether exact deduplication (hash-level) or embedding-based semantic deduplication is performed, and gives no feature-extraction method or similarity threshold.

### [HunyuanVideo-Foley](../models/HunyuanVideo-Foley.md) ⚠️

The paper does not mention any deduplication step at all: neither file-hash/audio-fingerprint-level exact deduplication nor embedding-based (audio or visual) semantic deduplication, and cross-source duplication is not discussed.
【Duplication risk assessment for this task】The risk objectively exists and takes a distinctive form:
1) Same-source duplication — after a single long video is scene-segmented, multiple 8-second chunks are produced; if the video's content has high internal repetitiveness (e.g., a long fixed-shot recording), it will generate a large number of near-duplicate samples; the paper's segmentation strategy naturally amplifies this kind of duplication, yet no deduplication measure is described.
2) Sound-effect-library duplication — a large amount of web video reuses the same batch of commercial sound-effect-library assets (the same explosion sound, the same footstep sound, reused across tens of thousands of videos); this kind of audio-track-level duplication is easy to detect via audio fingerprinting, but if left unhandled, will cause the model to overfit to specific sound-effect samples. This is a duplication risk that is unique to and more severe in the V2A task than in video-generation tasks, because sound-effect-asset reuse rates far exceed those of visual footage.
3) Background-music duplication — popular BGM has an extremely high reuse rate in short videos.
【Relationship to scale】At the 100,000-hour scale, even a 10–20% duplication rate would not be easily noticeable, but it would substantively skew the category distribution (and category-distribution management is precisely a step this work emphasizes) — if a certain sound-effect category is systematically amplified due to asset-library reuse, the distribution derived from label statistics will deviate from the true acoustic-event distribution. This creates a methodological tension between the "no deduplication" design and the "category distribution management" design, on which the paper offers zero discussion. [Uncertain]

### [HunyuanVideo](../models/HunyuanVideo.md) ⚠️

【Original version】Performs semantic-level deduplication with a clear method: an internally developed, in-house VideoCLIP model extracts video embeddings, and duplicates are judged and removed based on cosine distance; the same set of embeddings is then run through k-means to obtain roughly 10,000 concept centers, used for concept resampling and balancing. No hash-level exact deduplication is mentioned.
【1.5】Described only as "basic deduplication and the removal of corrupted files" as the frontmost step, without stating whether this is hash-based or embedding-based semantic deduplication, and without stating whether the VideoCLIP approach is carried over. Version 1.5's deduplication description is noticeably weaker than the original's.[Uncertain]

### [InstructAV2AV](../models/InstructAV2AV.md) ⚠️

[Uncertain] The paper does not mention any deduplication step at all — neither exact deduplication (file hashing, pHash perceptual hashing) nor embedding-based semantic deduplication is described. The dataset card does not mention it either.
【Specific duplication risks under this pipeline, worth pointing out separately】
  1. Same-source multi-clip duplication: a single film, after being segmented by PySceneDetect, can produce dozens to hundreds of single-shot clips; if multiple clips from the same film all pass filtering into the dataset, they will be highly similar in scene, characters, lighting, and mix style, constituting semantic near-duplicates. Among the sources, MovieBench, Condensed Movies, and Short-Films-20K are all film-level datasets, so this risk is substantively real.
  2. Same-clip multiple-editing duplication: the same source clip can perfectly well be subjected to multiple different instructions, generating multiple targets and thus multiple training pairs. This is a natural and economical practice in data-engine design (reusing the expensive upstream filtering result), but it causes the source side to be highly repeated. The paper does not state whether a "maximum N instructions per source" limit was applied, nor does it state how many distinct source clips the 79K training pairs correspond to — if this number is far smaller than 79K, the actual diversity of the data would be significantly lower than the nominal scale. This is a key unknown for assessing the dataset's effective scale.
  3. Sliding-window segmentation duplication: if 5-second windows are extracted via sliding window from a long single-shot clip multiple times, adjacent windows would overlap substantially.
  4. Train-eval contamination: the 1K human-evaluated set and the 79K training set both come from the same pipeline and the same material pool; the paper does not explicitly declare that the two have no overlap at the source-clip level (it only states the evaluation set went through triple human vetting). If different clips from the same film end up on both the training and evaluation sides, the evaluation would be biased optimistic. This point is more serious in this work than in a genuine real dataset, because the style of the synthetic targets is determined by the same data engine, so what the model learns might be "imitating the output style of Wan2.2-5B + ElevenLabs" rather than general editing ability — while the evaluation set is produced by the very same engine — creating a systematic bias of "the evaluation favoring the in-house data engine's style" that the paper neither discusses nor substantiates with third-party data. Fortunately, the paper additionally conducts a zero-shot evaluation on the external AvED-Bench (FVD 227.82 / FAD 4.32 / AV-A 23.71, outperforming AVI-Edit's 372.37 / 7.65 / 23.21), which somewhat alleviates this concern.

### [2026 Miscellaneous Joint Audio-Video Generation](../models/JAVG_2026_misc.md) ⚠️

【NAVA — the only work that performs semantic deduplication with a clear method】Original text: "Redundant or near-duplicate clips [removed] by extracting video embeddings with VideoCLIP and performing large-scale k-means clustering" — extracting video embeddings with VideoCLIP, then performing large-scale k-means clustering, and removing redundant/near-duplicate clips accordingly. This is a typical case of embedding-based semantic deduplication (rather than hash-level exact deduplication); the k value, the within-cluster retention policy (how many kept per cluster, ranked by what), and the similarity threshold are all undisclosed [uncertain]. Choosing VideoCLIP over frame-by-frame CLIP indicates that temporal considerations were also taken into account. Exact deduplication (pHash/frame fingerprinting/file hashing) is not mentioned [uncertain].
【ALIVE】The six-stage pipeline contains no deduplication step at all; the full paper never mentions deduplication [uncertain]. Given its data source is an internal raw data pool at the scale of 11M samples, the lack of deduplication is a notable gap — unless its raw pool was already deduplicated prior to ingestion (the paper does not state this).
【OmniCustom / StreamChar】Relies on the deduplication already performed by upstream datasets (SpeakerVid-5M / TalkVid / OpenHumanVid) and has no deduplication step of its own [uncertain]. OmniCustom has an implicit "anti-deduplication" risk: it extracts "the first 4 seconds as reference + the following 5 seconds for training" pairs from the same long video, so the same speaker may appear multiple times in the dataset — a feature rather than a flaw for the identity-customization task (multiple samples of the same identity are needed), but it also means the distribution at the speaker level may be highly imbalanced [uncertain].
【CCL / Baton】Deduplication not mentioned [uncertain]. Baton mixes four sources — OpenHuman-Vid, AudioCaps, WavCaps, and internet videos — AudioCaps itself is a subset of AudioSet, and part of WavCaps's sources also overlap with AudioSet/FreeSound, so cross-dataset duplication risk objectively exists but is not addressed in the paper [uncertain].
【ITS-JAVG】Not applicable; however its Best-of-N / EvoSearch candidate-diversity-maintenance issue is mathematically the same problem that deduplication addresses — sample redundancy.
【Overall judgment】Of the seven works, only NAVA performs explicit deduplication, and it is also the most fully disclosed in this batch — this confirms a pattern: investment in deduplication correlates with data scale (NAVA processes 100M videos, while the rest mostly operate at the 1M–11M scale or directly reuse already-cleaned public datasets).

### [Joint Audio-Video Generation Baselines Collection](../models/JavisDiT_baselines.md) ⚠️

All five works in this collection describe no deduplication step at all [uncertain], whether exact deduplication (hash/pHash/frame fingerprinting) or embedding-based semantic deduplication (CLIP/video-embedding clustering) — a uniform gap across the collection.
Individual indirect situations:
- MM-Diffusion: Landscape is explicitly cut into "non-overlapped" 10-second clips — this is clip-level overlap avoidance, not cross-video deduplication; with only 928 source videos, the scale is small enough that duplication risk is naturally low.
- AV-DiT: uses ready-made public datasets; deduplication responsibility lies with the upstream dataset authors.
- JavisDiT / JavisDiT++: the only relevant design is that "the 30K prompt pool used for DPO is strictly non-overlapping with the SFT training data (apart from the SFT training data)," and the 330K SFT and 25K DPO samples do not overlap with each other — this is isolation at the train/eval split level, not data deduplication. Whether the upstream TAVGBench performed deduplication is unknown [uncertain]. JavisBench's "post-filtering to ensure diversity" during construction is functionally close to semantic deduplication, but the authors do not use the term "deduplication," nor do they describe what method is used to measure diversity [uncertain].
- Harmony: 4 million entries assembled from multiple public datasets (AudioCaps, Clotho, WavCaps have a known overlap among them, e.g., WavCaps intersects with AudioSet), but the paper does not mention cross-dataset deduplication [uncertain] — a potential duplication-risk point.
- UniAVGen: no description at all [uncertain].

### [Kling 3.0 / Kling Video 3.0 Omni](../models/Kling_30_Omni.md) ⚠️

The Kling-Omni report explicitly adopts a "frame-wise and temporal fingerprinting" deduplication mechanism, a form of perceptual-hash/fingerprint-based exact and near-duplicate deduplication. [Uncertain: whether embedding-based semantic-level deduplication is also present — not mentioned in the report; but its internet-mining step uses an in-house embedding model for cross-modal semantic retrieval, which provides the infrastructure basis for performing semantic deduplication]

### [LTX-2](../models/LTX-2.md) ⚠️

Adopts representation-based semantic deduplication: for each shot, the Mid-Frame's CLIP image embedding is computed, and "Cluster and De-Duplicate" is then performed in embedding space. This is embedding-level semantic deduplication rather than hash-level exact deduplication; the paper does not separately state whether an additional exact-deduplication step exists, nor does it disclose the clustering algorithm, similarity threshold, within-cluster retention policy, or deduplication ratio. [Uncertain]

### [LongCat-Video](../models/LongCat-Video.md) ⚠️

Performed only at the first preprocessing step, with a clear but fairly basic method: dual deduplication by source video ID and MD5 hash. The former removes duplicate collections of the same video by the source-side unique ID; the latter performs exact, byte-level-hash-based deduplication on files.
The report does not mention any embedding-based semantic/perceptual deduplication (e.g., CLIP/video-feature near-duplicate detection, pHash perceptual hashing), so there is no explicit handling mechanism for near-duplicates such as "different encoded versions, different crops, or reposts of the same material." However, caption-embedding-space clustering and "inverse-density sampling" during the SFT stage have an approximate semantic-deduplication effect — high-density redundant clusters are automatically down-sampled. [Uncertain: whether an undisclosed semantic-deduplication step exists elsewhere]

### [MOVA](../models/MOVA.md) ⚠️

The paper does not mention any deduplication step at all: neither hash/fingerprint-level exact deduplication nor embedding-based semantic deduplication. Given that the data sources simultaneously include multiple YouTube-sourced public datasets such as ACAV-100M, VGGSound, and OpenVid-1M, as well as in-house YouTube-scraped content, the risk of cross-dataset duplication objectively exists, but the paper offers no discussion of this at all. [Uncertain]

### [Merged Entry: ① Mochi 1](../models/Mochi_MAGI_Motif.md) ⚠️

① Mochi 1: undisclosed. [Uncertain]
② MAGI-1: the paper traces the importance of deduplication back to the LLM literature — citing Lee et al. (2021) and Hernandez et al. (2022) to note that "even a small amount of duplicated data can significantly harm performance," and on that basis "conduct rigorous de-duplication." The method is a dual-feature cross-criterion: CLIP and DINOv2 are both used to compute pairwise similarity between clips, and a clip is judged a duplicate and removed if either similarity exceeds a threshold (logical OR, a strict rule leaning toward over-removal). The intent of using the CLIP + DINOv2 dual-tower approach is to cover both semantic-similarity and visual/self-supervised-structural-similarity types of duplication. Threshold values and the amount of data removed are not given, nor is it explained how the O(N²) pairwise computation is avoided at scale. [Partially uncertain: thresholds, indexing scheme, and deduplication volume]
③ Motif-Video 2B (the only one of the three to give reproducible engineering detail): a three-stage SSCD pipeline —
· Embedding: uses the publicly released sscd_disc_mixup TorchScript model to encode each image/each video; frame images are first resized to 320×320 and ImageNet-normalized, producing a 512-dimensional descriptor. For video, the 10th frame is taken as the representative frame — the paper explicitly explains two reasons for this choice: avoiding the intro/logo bias present in the very first few frames, and avoiding full pairwise frame comparison, keeping matching computationally tractable. The reason for choosing SSCD is that it is purpose-built for copy detection, robust to re-encoding, cropping, and light editing — precisely the most common forms of duplication in web-crawled video.
· Grouping: uses NVIDIA cuVS's multi-GPU IVF-PQ index to retrieve under cosine distance, taking k=64 nearest neighbors per query with nprobe=16, retaining only pairs with cosine similarity above 0.9, then merging matched pairs into duplicate groups via Union-Find.
· Representative selection: within each duplicate group, a single sample is retained according to a weighted score s = 0.5·res^ + 0.3·fps^ + 0.2·filesize^ (all three min-max normalized within the group), with the remaining group members discarded. This rule favors copies with higher resolution, higher frame rate, and less heavy recompression.
Deduplication is placed at the very first stage of sanitation (rather than at the end of the funnel), meaning all subsequent expensive model inference is never wasted on duplicate samples.

### [Movie Gen](../models/Movie_Gen.md)

Split into two levels, roughly corresponding to the "exact/semantic" dichotomy required by this field:
· Perceptual-level/near-duplicate deduplication: uses similarity in the space of a copy-detection embedding (SSCD, Pizzi et al., 2022) to remove perceptually duplicate clips; the threshold given in Appendix Table 44 is "embedding cosine similarity < 0.99 retained," and this step in the funnel brings the figure from 1.22% down to 1.15% (roughly a 6% removal). The same method is also used for visual deduplication of the Movie Gen Audio pretraining data.
· Semantic-level deduplication and balancing: a joint video-text embedding model extracts semantic embeddings for clustering, producing fine-grained concept clusters; duplicate clusters are first merged, then sampling is performed per cluster proportional to 1/sqrt(cluster size). This step is both semantic de-redundancy and concept balancing, further bringing the figure from 1.15% down to 0.94%.
· Evaluation also pays attention to duplication: the paper notes that large amounts of duplicate or near-duplicate content (e.g., versions with static watermarks or text added) were found between the train/eval splits of some public datasets.

### [NVIDIA NeMo Curator](../models/NeMo_Curator.md) ⚠️

Primarily semantic deduplication, with a clear method chain:
【Embedding】Uses Cosmos-Embed1 to generate clip-level video embeddings (earlier and the Cosmos WFM production version used InternVideo2; InternVideo2 was removed from NeMo Curator in the 26.02 release). Embeddings are stored on disk in Parquet format, with the same set of embeddings serving both semantic retrieval and deduplication.
【Clustering】K-means clustering is applied to the embeddings, with the Cosmos WFM production environment using k = 10000.
【Within-cluster comparison】Pairwise distances are computed only within the same cluster to identify near-duplicates, avoiding full N² comparison — the key to feasibility at PB scale.
【Retention policy】Within a near-duplicate group, the highest-resolution version is retained.
【Effect】Removes approximately 30% of the training data.
【Framework implementation】The documentation describes this as "semantic clustering + pairwise similarity + k-means," organized into deduplication chunks of clips.
【Exact deduplication】No hash-based exact-deduplication stage is seen on the video side (the text side has a mature exact-dedup and fuzzy-dedup, with the latter officially claimed to be 16x faster and 40% lower TCO than a CPU-based approach on RedPajama v2 — but that figure is for the text modality and cannot be transplanted to video). The similarity-threshold value is not disclosed. [Uncertain]

### [OmniHuman Dataset + OHBench](../models/OmniHuman.md) ⚠️

【Explicit deduplication step: none】The paper describes no sample-level deduplication step: no video-fingerprint/pHash-level exact deduplication, no CLIP/SigLIP-embedding-level semantic deduplication, no cross-source duplication check.
【An approximate technique used for a different purpose】Stage 4 uses perceptual hashing + SigLIP cosine similarity, but its stated purpose is to "filter semantically inconsistent frames," i.e., a temporal-consistency check within an individual clip, not cross-sample duplicate detection. The technique closely resembles deduplication, but the purpose is different, so it cannot count as deduplication.
【Specific forms of duplication risk】For a dataset with YouTube as its sole source, duplication risk has particular characteristics: (1) multiple uploads of the same content (reposting, re-uploading, compilations) are extremely common on YouTube; (2) a given uploader's series of videos (same studio, same camera position, same person) produces large numbers of highly similar clips; (3) 80,000 identities / 1 million entries means an average of 12.5 clips per identity — if the identity distribution has a severe long tail, popular creators could contribute hundreds of near-homogeneous clips.
【Identity-level deduplication and quotas: also absent】The paper describes no sampling quota or long-tail suppression by identity. Although the figure of 80,000 identities is impressive, its distribution shape (whether it is even) is entirely unknown — if it follows a power law, actual effective identity diversity would be far lower than the nominal figure. [Uncertain]
【Partial fallback via the evaluation set】OHBench's 509 videos are drawn from OmniHuman but explicitly require "a domain gap from the training set," which avoids train-eval leakage at the evaluation level, but the redundancy problem within the training set itself is unaffected by this.

### [Open-Sora Series](../models/Open-Sora.md) ⚠️

**Neither project implements a substantive deduplication step**, a clear gap in their data pipelines relative to industry practice.
- Open-Sora's tools/ directory has no independent deduplication module; although the README feature list once mentioned deduplication, the four-stage flow in docs/data_processing.md (v1.2.0) contains no deduplication step, and tools/datasets/datautil mainly does score-column-based filtering and metadata merging, with no embedding-level semantic-deduplication implementation seen.
- Open-Sora Plan's seven-level funnel table likewise contains no deduplication level, and neither the paper nor any version's Report discusses exact deduplication (hashing) or semantic deduplication (embedding clustering).
- Potential risk: Panda-70M's 3.8 million long videos, sourced from HD-VILA-100M, are cut into 70.8 million clips, and adjacent clips from the same long video are highly similar; Open-Sora Plan additionally uses ShareGPT4Video (likewise sourced from Pexels/Pixabay), and neither source-level overlap nor intra-clip redundancy is handled. [Uncertain]

### [Ovi](../models/Ovi.md) ⚠️

[Uncertain]. None of the four steps in the paper's data pipeline includes a deduplication step, describing neither exact deduplication (hashing/pHash/frame fingerprinting) nor embedding-based semantic deduplication (CLIP/video-embedding clustering for de-redundancy). No deduplication is described for the pure-audio side either. Since the data comes from an internal corpus rather than large-scale web crawling, the duplication rate may naturally be lower, but this is only an inference with no textual basis whatsoever.

### [Script-a-Video](../models/Script-a-Video.md) ⚠️

The paper does not mention any deduplication step at all: neither hash/fingerprint-level exact deduplication nor embedding-based semantic deduplication, and cross-source or cross-stage duplication is not discussed. [Uncertain]
Potential duplication risk objectively exists: the four generation-side datasets (400K identity-centric, 250K multi-shot, 870K cinematic pairs, 60K cinematic alignment pairs) all originate from the same internal film/TV material library, and the final joint fine-tuning stage explicitly reuses the aforementioned 250K multi-shot sequences, indicating cross-stage data reuse is intentional; but whether there is overlap of same-source clips across the different-stage datasets is not stated anywhere in the paper.
Worth noting: MTSS's design has a built-in de-redundancy mechanism at the "description level" — the Reference stream's centralized entity library means recurring characters/scenes only need to be described once, with subsequent Shot and Event references pointing to the ID, which the paper calls eliminating "redundant re-description." But this is elimination of textual redundancy, a different level of problem from dataset sample deduplication, and the two should not be conflated.

### [Seedance 2.0 and Seedance 1.5 pro](../models/Seedance_20_Seedance_15_pro.md) ⚠️

Seedance 1.0 adopts Semantic Deduplication: an internally developed video-representation model extracts robust feature embeddings, clustering visually and semantically similar clips based on the embeddings; within each near-duplicate cluster, only the single instance with the highest overall quality score (from the preceding quality-filtering level) is retained. The report does not separately describe an exact-hash deduplication step. 1.5 and 2.0 do not disclose their deduplication schemes. [Uncertain: whether exact deduplication is included, the clustering threshold, the deduplication ratio]

### [SkyReels Series](../models/SkyReels.md) ⚠️

【SkyReels-V4】Explicitly adopts representation-based semantic deduplication: VideoCLIP embeddings are computed for the segmented clips and used for deduplication ("deduplication using VideoCLIP embeddings for segmented clips"). This is embedding-level semantic deduplication; the similarity threshold, clustering algorithm, and deduplication ratio are not disclosed, nor is it stated whether an additional hash-level exact-deduplication step exists.
【SkyReels-V2】The data section of the paper does not separately describe a deduplication step (nor was one extracted in third-party structured summaries), so it cannot be confirmed whether exact deduplication or semantic deduplication is present; given that the self-collected data includes 280,000 films and 800,000 TV episodes, the risk of duplicate footage from the same IP is relatively high, and some form of deduplication is presumed to exist, but there is no official basis for this. [Uncertain]

### [Sora 2](../models/Sora_2.md) ⚠️

Completely undisclosed. Neither exact deduplication (hash-level) nor embedding-based semantic deduplication is described. [Uncertain]

### [SpeakerVid-5M](../models/SpeakerVid-5M.md) ⚠️

Neither the paper nor the data card mentions any deduplication step: neither video-hash/fingerprint-based exact deduplication nor embedding-based (CLIP/DINO, etc.) semantic deduplication, and no description of near-duplicate detection across clips or across source videos.
【Objective risks】(1) A single YouTube video is segmented into multiple clips, and multiple clips of the same speaker in the same setting are highly similar, a structural form of near-duplication; (2) the data source spans YouTube content from 2006–2025, and reposts, edited versions, and multi-platform re-uploads of the same interview are extremely common on YouTube — if deduplication during collection is done only by video ID, content-level duplication cannot be identified; (3) 83K speaker IDs correspond to 5.2M clips, an average of about 63 clips per speaker; popular celebrities (a typical subject of interview/news-type content) likely contribute far more clips than average, indicating an identity-level long-tail imbalance.
【The only relevant mechanism】The ArcFace ID-correction step computes cross-clip facial cosine similarity, but its purpose is "ensuring consistent annotation of the same ID," not "removing duplicate content" — the direction is in fact the opposite, since it is performing identity aggregation, not deduplication.
【Conclusion】Deduplication is a clear gap in this dataset's disclosure. [Uncertain]

### [Step-Video-T2V](../models/Step-Video-T2V.md) ⚠️

The technical report describes no deduplication step at all — the six-stage pipeline includes no independent deduplication step, nor does it mention perceptual hashing (pHash) or other exact/near-duplicate deduplication, or embedding-based semantic deduplication.
The only thing that functionally partially serves a deduplication purpose is the concept balancing in Stage 5: K-means clustering (120,000+ clusters) on VideoCLIP embeddings, combined with Cluster_Cnt (number of samples in a cluster) for resampling, which can suppress the over-concentration of highly similar content, and Center_Sim removes outliers within a cluster; but this is fundamentally distribution balancing and outlier detection, not equivalent to duplicate-sample removal. This is a disclosure gap in this entry relative to HunyuanVideo (which explicitly uses VideoCLIP cosine distance for semantic deduplication). [Uncertain]

### [UniTalking](../models/UniTalking.md) ⚠️

The paper does not mention any deduplication step at all: neither hash/fingerprint-level exact deduplication (pHash, video fingerprinting) nor embedding-based semantic deduplication (CLIP/DINO feature clustering), and cross-source duplication is not discussed.
【Specific sources of duplication risk】This work's two-source structure raises duplication risk above that of single-source works: OpenHumanVid's video samples are themselves aggregated from "multiple publicly available datasets," so cross-subset duplication may already exist internally; and if the internally collected Huawei data also includes public-platform content, the probability of overlap with OpenHumanVid is not low. There is no deduplication mechanism at all between the two sources, which could cause some content within the 2.3 million entries to be counted more than once and oversampled during training.
【Identity-level deduplication is also absent】For speaker-generation tasks there is also a distinctive form of duplication — a large volume of clips of the same speaker (e.g., many episodes of the same host/actor) could be highly concentrated in the dataset, resulting in a severe long tail in identity distribution. The paper performs neither identity-level deduplication nor identity-level quota control, and does not report the number of unique speakers in the dataset. This has a direct impact on the model's identity generalization and voice-cloning ability, yet is a complete information gap. [Uncertain]

### [UniVerse-1](../models/UniVerse-1.md) ⚠️

The paper does not mention any deduplication step at all: neither hash/fingerprint-level exact deduplication nor embedding-based semantic deduplication, and cross-source duplication is not discussed. The risk objectively exists — the self-collected data is predominantly YouTube-sourced, while the concurrently introduced VGGSound and AudioSet are themselves collections of YouTube clips, so potential content overlap exists among the three; in addition, Pexels material is commonly and repeatedly included across multiple public datasets. The paper offers no discussion of this. [Uncertain]

### [Unison](../models/Unison.md) ⚠️

The paper does not mention deduplication at all: no hash/fingerprint-level exact deduplication, no embedding-based semantic deduplication, and cross-source duplication is not discussed.
【Duplication risk objectively exists and is not small】This is a typical hazard of aggregated data construction; Unison's specific risk points include:
1) Among the five audio-video sources, HDTF, VFHQ, and CelebV-Text are all primarily YouTube interview/speech footage of celebrities/public figures, so the likelihood of the same person, or even the same video, being collected by multiple datasets is high;
2) VGGSound and AudioSet are both collections of YouTube videos, with a known significant overlap between them, and Unison uses VGGSound on the audio-video side while using AudioSet on the pure-audio side, creating duplication between the cross-modal corpora;
3) The source pools of YouTube-8M, AudioSet, and VGGSound are entirely identical (YouTube), so overlap is unavoidable;
4) Whether there is track-level overlap between the music-side VidMuse and the singing-side YuE collection is also not stated.
【The paper's silence】No discussion of any of the above risks, nor any statement of whether the "automated processing pipeline" includes a deduplication step. Given that duplicate data would cause oversampling of specific individuals/specific sound events, subsequently affecting identity consistency and acoustic diversity, this is a substantive methodological gap. [Uncertain]

### [Veo 3 / Veo 3.1](../models/Veo_3_Veo_31.md) ⚠️

One of the few relatively specific and officially disclosed items. Model Card: "All training data was deduplicated semantically across various sources." Technical report: "All data is deduplicated semantically across sources to minimize the risk of outputs overfitting particular elements of training data." The Mitigations section additionally states: "removing duplicated and/or conceptually similar videos." It can be confirmed that: cross-data-source semantic-level deduplication (semantic dedup, presumably implemented via embedding similarity) is performed, removing not only exact duplicates but also videos that are "conceptually highly similar." The motivation for deduplication is explicitly stated as reducing the risk of outputs overfitting/memorizing specific training samples (i.e., mitigating verbatim reproduction and copyright risk). [Uncertain] The embedding model used, the similarity threshold, and the clustering method are undisclosed, and the respective contributions of exact deduplication (e.g., perceptual hashing) versus semantic deduplication are not distinguished.

### [Vidu S1](../models/Vidu_S1.md) ⚠️

At the very front of the pipeline (before formally entering the pipeline proper), all data is first deduplicated (all data are first deduplicated) before pre-filtering. The paper does not distinguish between exact deduplication and embedding-based semantic deduplication, nor does it state which features or similarity threshold are used [uncertain].

### [Wan 2.5 / 2.6 / 2.7](../models/Wan.md) ⚠️

Disclosure is extremely thin. The Wan 2.1 report only touches on it in passing, within the data-source sentence: "We curated and deduplicated a candidate dataset sourced from…" — confirming that a deduplication step exists and occurs at the very front of the funnel (during candidate-pool construction), but with no method description whatsoever: no distinction between exact deduplication (hashing/fingerprinting) and semantic deduplication (embedding clustering), and no similarity threshold, deduplication ratio, or within-cluster retention policy given.
One point that could be misread: the "clustering" (100 clusters) in Step 2's visual quality is for quota sampling to maintain distribution — the report explicitly states its purpose is to prevent long-tail loss, not deduplication, and should not be conflated with it.
2.2/2.5/2.6/2.7 all have no deduplication information whatsoever. [Uncertain]

### [Audio-Video Generation Evaluation Benchmarks Collection](../models/av_benchmarks.md) ⚠️

None of the five discloses an explicit exact-deduplication or embedding semantic-deduplication process [uncertain]. Related approximate mechanisms:
【PhyAVBench】Adopts a "reverse deduplication" idea — proactively ensuring zero overlap with existing training sets (all footage newly recorded), and deliberately sampling across different individuals, presenters, and recording devices during collection to increase intra-sample diversity — avoiding homogenization at the source rather than deduplicating after the fact.
【AVBench】Hard Quota-Based Greedy Sampling forces any single attribute to make up ≤50%, functionally equivalent to de-redundancy/balanced sampling at the attribute level.
【AV-SyncBench / VABench / Omni-Judge】Deduplication not mentioned [uncertain].

### [Video Caption Model Ecosystem](../models/caption_models.md) ⚠️

[Uncertain] This is one of the fields with the least disclosure in the captioner ecosystem.
【The small amount of known evidence】
· AVSCap-130K's sources include AVoCaDO-107K, and AVoCaDO-SFT's sources in turn include public datasets such as FineVideo and Shot2Story, which themselves already have video overlap among them — this "datasets built on datasets" snowballing reuse brings a significant but unquantified cross-dataset duplication risk, which neither paper discusses in terms of deduplication.
· Tarsier2-Recap-585K is produced from 1M videos in public datasets, whose sources (VATEX, TGIF, LSMDC, etc.) likewise have known overlap; the deduplication strategy is undisclosed.
· Panda-70M's greedy set cover is de-redundancy of "captioners" (reducing 31 down to 8), not deduplication of video samples.
· Goku uses Qwen2 as an LLM fuser to merge keyframe captions and video captions into a "unified, non-redundant, non-contradictory" final description — this is text-level redundancy elimination for captions, different from sample-level deduplication.
· LongCat-Video uses an LLM to name categories from clustering results on caption embeddings; caption-embedding clustering has the technical prerequisites for semantic deduplication, but the report does not state whether it is used for that purpose.
【Ecosystem judgment】Exact deduplication (hashing) and semantic deduplication (embedding) of video samples is generally completed upstream of the captioner, performed by the generation-side team, and rarely disclosed; deduplication of the caption text itself (avoiding homogeneous template sentences) is almost never discussed by anyone — given that CogVideoX's prompt explicitly bans stock phrases such as "The video presents / depicts / showcases" and "throughout the video," caption homogenization is evidently a real problem, but the industry addresses it via prompt constraints rather than after-the-fact deduplication.

### [Geometric/Structured Annotation Dataset Collection](../models/geometric_datasets.md) ⚠️

Action100M's deduplication is the most systematic and quantified: after annotation aggregation, 7.58M duplicate groups comprising a total of 141.8M duplicate instances were identified and deduplicated, followed by k-means (k=10³/10⁴/10⁵) semantic resampling to flatten the long tail — a combination of semantic-embedding-based deduplication plus rebalancing. SceneScribe-1M: merged from four libraries — HD-VILA-100M/Panda-70M/Koala-36M/Pexels — with cross-library overlap risk present; the paper describes no explicit deduplication step [uncertain]. SpatialVID: collected from YouTube by video ID, with natural near-duplication existing between clips cut from the same long video; the paper describes no embedding-level semantic deduplication [uncertain]. WildWorld: the action space has an obvious long tail (the top-150 actions account for 58.49% of samples); the paper presents this as an action-triplet distribution rather than deduplication, and no explicit deduplication process is seen [uncertain].

### [Video Generation Post-Training Data Strategy](../models/post_training_data.md) ⚠️

Not addressed by the anchor paper [uncertain]. The deduplication concern in the post-training stage differs from that in pretraining: it is not about removing duplicate footage but about removing duplicate/ambiguous prompts. Seedance 1.0 explicitly performs "data balancing and information filtering to remove duplicate and ambiguous prompts" after RLHF prompt collection; Step-Video-T2V has annotators synthesize supplementary prompts according to guidelines to ensure prompt diversity; JavisDiT++ ensures the 30K DPO prompt pool does not overlap with the SFT data; VideoReward reserves 13K prompt triples that never appeared in the training set as a validation set.
The most typical form of asset-level deduplication at the SFT stage is Step-Video-T2V's VideoCLIP K-means within-cluster outlier removal (formally clustering, functionally combining semantic deduplication with outlier removal) and LongCat-Video's caption-embedding inverse-density sampling (down-sampling high-density, i.e., near-duplicate, regions — a continuous, soft form of deduplication). These two represent the direction of "replacing hard deduplication thresholds with embedding-space density."

### [Merged Survey of Mainstream Video Pretraining Datasets: Panda-70M, InternVid, Koala…](../models/pretraining_datasets.md) ⚠️

**Deduplication is generally weak across the seven; only two did substantive work, and neither is embedding-level semantic deduplication**:
- **Panda-70M (the only one with a clip-level deduplication mechanism)**: uses the mean of each clip's first-stage ImageBind features as its representation, **retaining only clips with Euclidean distance >0.3 from the preceding clip** — this is intra-clip redundancy deduplication built into the segmentation flow. At the release level, source-level deduplication is also applied via a "maximum 3 clips per source video" rule (for the 10M subset), while the 2M subset takes exactly 3 clips per source video. But **across the full 70M, the average is about 18.7 clips per source video, and the data is not shuffled** (all clips from the same source video fall into the same shard, and the repository explicitly notes users must shuffle it themselves), so the redundancy risk is high.
- **UltraVideo**: in the re-filtering path from Koala-36M, a "deduplication" step is explicitly included (uniform per-category sampling followed by "plus dedup"), but **the specific method is undisclosed**. [Uncertain]
- **InternVid**: **has no deduplication process**; it only excludes, during crawling, videos that already existed in public datasets as of April 2023 (this is cross-dataset overlap prevention, not internal deduplication).
- **Koala-36M: has no deduplication at all.** The word "duplicat" appears only once in the entire paper, and that is within a reference-title citation (the D4 paper). Given that its 48M clips are cut from roughly 100 million HD-VILA YouTube videos, **near-duplicate content is an unaddressed, substantive risk**.
- **MiraData / OpenVid-1M / LVD-2M: none of the papers mentions any deduplication method.** [Uncertain] Among these, LVD-2M's risk is most worth flagging — it uses **HD-VG-130M, Panda-70M, and InternVid** simultaneously, three corpora all sourced from YouTube, with **unquantified source-level overlap among them** (Panda-70M derives from HD-VILA-100M, and HD-VG-130M is likewise crawled from YouTube), and the paper performs no cross-source deduplication whatsoever.
- **Exact deduplication (hashing) and semantic deduplication (embedding clustering)**: **none of the seven implements either.** This is the most consistent common technical gap among the seven datasets surveyed here, and also represents a notable generational gap relative to industry data pipelines.
## VLM/LLM as data quality judge (multimodal large-model quality scoring and mismatch removal — the 2026 shift from shallow scorers to large-model semantic judgment)

`model_as_data_judge` · Detail level: detailed

### [Allegro](../models/Allegro.md)

Belongs to the 2024-typical form of "shallow scorers as the mainstay, large models used only for annotation rather than judgment," and has not yet entered the VLM-as-judge stage:
· The models used as judgment criteria are all specialized small models/scorers: LAION Aesthetics Predictor (aesthetics), DOVER (video quality), LPIPS (perceptual consistency), UniMatch (optical flow), CRAFT (text detection), CLIP (image-text consistency).
· The role of multimodal large models is "generator" rather than "judge": Tag2Text is responsible for coarse-grained tags, and Aria (25.3B MoE) is responsible for fine-grained captions; Aria is not used to score video quality, make semantic judgments on image-text mismatch, or output structured quality-dimension scores.
· The only filter with any semantic character is the CLIP similarity threshold at level 7 (≥0.17 / ≥0.20), which in essence is still embedding cosine distance rather than large-model semantic judgment.
Therefore, judged by 2026 standards, Allegro belongs to the previous-generation paradigm of "model-based quality control."

### [Apollo](../models/Apollo.md) ⚠️

Apollo's data pipeline exhibits clear "deep large-model involvement," but this role is concentrated on the **annotation-generation** side rather than the **quality-adjudication** side, which only partially aligns with the 2026 trend of "shifting from shallow scorers to large-model semantic judgment":
[Large models handling annotation] Qwen2.5-Omni (an omni-modal large model) is used for both speech transcription and audio captioning; Gemini 2.5-Pro (a closed-source commercial flagship multimodal large model) is used for audio captioning — introducing a commercial API-tier large model for large-scale annotation at the 81-million-clip scale is a rather high-cost choice, reflecting Kuaishou's priority investment in annotation quality. The video side uses a "video expert model" (unnamed, presumably an in-house VLM).
[Quality adjudication is still handled by specialized models/signal-processing metrics] Audio quality uses traditional signal and perceptual metrics such as SNR, MOS, and clipping/distortion/noise detection; audio-video alignment uses two specialized discriminative models, Synchformer (temporal) and ImageBind (semantic); video quality uses an unnamed multi-dimensional scorer. Nowhere in the entire pipeline does a step appear where "a VLM/LLM produces a single composite quality score" or "an LLM-as-judge performs cross-modal consistency adjudication."
[Key gap] Compared with MOVA's explicit use of GPT-OSS-120B to resolve visual-audio consistency conflicts, with built-in anti-hallucination self-audit fields in the prompt, the Apollo paper contains no description of hallucination suppression for annotations, defense against cross-modal crosstalk, or secondary verification of annotation results — how the outputs of multiple models (three ASR systems used together: Whisper / SenseVoice / Qwen2.5-Omni) are arbitrated and merged is glossed over with a single sentence, "All annotations are merged into unified dense captions," and the merging rules are entirely undisclosed.
[Inference] The most likely purpose of using three ASR models together is cross-validation/voting to remove samples with unreliable transcription (which is in essence a form of model-ensemble quality control), but the paper does not confirm this. [Uncertain]

### [CineDance / CineDance-1M](../models/CineDance.md)

Deep large-model involvement is the core feature of this pipeline, and it is used not merely for scoring but for the heavy task of "structured semantic parsing," fully embodying the 2026 trend of shifting from shallow scorers to large-model semantic judgment:
[MLLM used for temporal truncation] Stage one uses a multimodal large model to judge the head/tail boundaries of long videos and decide where to truncate.
[LLM used for narrative parsing (the heaviest step)] Qwen3.5-27B judges, according to four film-theory rules, which adjacent shots constitute one complete narrative sequence — this already goes beyond "quality control" and treats the large model as a film-grammar parser. The key engineering measures are bottom-up shot indexing (to avoid timestamp hallucination) and a 3-minute sliding window aligned to shot boundaries. The Tab.4 ablation shows this combination reaches F1 = 88.4%, with only 3.1% of sequences shorter than the 20-second soft threshold.
[MLLM used for visual annotation] Qwen3.5-35B-A3B (MoE architecture, 35B total parameters / 3B active) produces the five-dimensional shot attributes and descriptions.
[Omni model used for audio annotation] Qwen3-Omni-30B-A3B (30B total parameters / 3B active) handles ASR, audio prompts, and character-timbre description, and performs ASR-to-Character binding.
[Three anti-hallucination engineering designs] (1) Audio annotation is split into three sub-tasks called separately rather than produced in one shot (reducing hallucination); (2) the ASR stage deliberately avoids speaker-to-character binding, with binding handled as an independent subsequent step; (3) binding adopts a windowed scheme (filtering non-speech intervals, preserving shot integrity and sentence completeness), raising Qwen3-Omni-30B-A3B's binding accuracy from 67.2% on whole-clip input to 95.4%.
[Comparison with traditional diarization (Tab.5/7)] Windowed Qwen3-Omni 95.4% > Gemini series 82.8%–87.4% > DiariZen 63.1% > Pyannote-3.1 62.7%, validating the advantage of the route "replacing dedicated diarization tools with an Omni large model."
[Quality-scoring side] Model scorers such as DOVER/DNSMOS/ImageBind/SyncNet run at full scale but are not used for hard cutoffs — they are stored only as metadata.

### [CogVideoX](../models/CogVideoX.md)

CogVideoX is one of the early representatives of the paradigm "using a multimodal large model as a data quality judge," relying on large models on both the annotation and quality-control sides:
[Quality-control side] The core quality-control component is a set of 6 video binary classifiers fine-tuned from Video-LLaMA, rather than traditional shallow scorers (aesthetic CNNs, OCR detectors, static-frame detectors, etc.). Trained with 20,000 manually annotated positive/negative labels, they make end-to-end semantic judgments directly on entire videos, covering in one pass six dimensions — editing traces, motion coherence, filming quality, genre type (lecture), text-coverage ratio, and screen-recording source — that would otherwise require multiple dedicated detectors. The test accuracy range reported in Appendix K is 0.89–0.99 (Low Quality 0.89, Editing 0.91, Static 0.92, Text 0.96, Screenshot 0.98, Lecture 0.99). This approach of "one VLM producing six semantic labels" is closer to the mainstream 2026 form than contemporaneous solutions relying on a combination of LAION aesthetic scores + OCR + static-frame detectors.
[Annotation side] GPT-4 acts as the teacher generating summary-style captions, CogVLM serves as the frame-by-frame image recaptioning model, and the results are ultimately distilled into LLaMA2 (summary acceleration) and CogVLM2-Caption (end-to-end video annotation).
[Limitations] Large-model judgment is still mainly "binary-classification filtering" — it is not used to produce continuous quality scores, has not been used to verify semantic consistency of caption-video mismatch, and is not used for automatic review of generated results; continuous scores are still provided by the traditional optical-flow and aesthetic models. It can therefore be positioned as a transitional form on the way "from shallow scorers to large-model semantic judgment."

### [Cosmos-Predict2.5](../models/Cosmos-Predict25.md)

This pipeline is a typical two-stage representative of "shallow scorers + large-model final review," and the paper rarely explains explicitly why it is ordered this way:
[VLM as the final-review quality judge] "Finally, we use a vision language model (VLM) (Bai et al., 2025) to further remove clips with a set of undesirable issues with higher precision. We apply the VLM at the very end of filtering because it is computationally more expensive." — that is, the VLM (Qwen2.5-VL series) does not replace the shallow scorers but is placed at the end of the chain to give a high-precision second pass over the survivors, the rationale being that compute is expensive. This is precisely the economic design of "use cheap filters first to remove the bulk, then let the large model refine borderline samples," and it is also a transitional form of the industry's shift, around 2026, from pure shallow scoring toward large-model semantic judgment.
[VLM as recall verifier] In the Smart Spaces domain: search keywords are first used to recall potentially relevant candidate videos, and then a "VLM (Bai et al., 2025) [is used] to verify its relevance" — the VLM verifies topical relevance clip by clip, treating the VLM as a semantic-level topic classifier/recall purifier.
[VLM as annotator] Captioning is handled entirely by Qwen2.5-VL-7B, while domain data switches to a larger-parameter VLM from the same family.
[Dedicated classifiers as domain sorters] A content-type classifier (internally trained, 26-class taxonomy) classifies each clip at the end of the filtering chain; the post-training side uses a multi-head classifier on InternVideo2 embeddings to split the data into five SFT domains. Both of these are discriminative small models rather than generative large models, reflecting the principle of "don't deploy a large model where a small model will do."
[Model-based scorers] The perceptual-quality filter uses a DOVER-style model and the semantic-artifacts filter uses a VTSS-style model — both are learned quality-assessment models rather than rule-based thresholds.
[Model judges on the RL side] Post-training uses VideoAlign (a VLM-based reward model) as the reward model, scoring along three dimensions — text alignment, motion quality, and visual quality — paired with an in-house Elastic Reward Service (supporting dynamic scale-up/down, compressed latent transport, pipeline-parallel decode and inference, zero-copy CUDA IPC, and an asynchronous UUID mechanism with Redis-backed results) — a rare case of engineering "model as judge" all the way down to the service level.
[Exception in the domain branches] The five major Physical AI domains explicitly omit the VLM filter, instead using a domain-specific filter subset with tuned parameters. This suggests the team judged that where the data source is already trustworthy and the domain is already narrow, the expensive VLM final review is not cost-effective — a valuable engineering judgment.

### [Data-Juicer 2.0](../models/Data-Juicer.md) ⚠️

Data-Juicer is a typical practitioner of the philosophy "processing data with foundation models" (the "with" in "for and with Foundation Models"), and its approach differs methodologically from the model teams: rather than having one large model assign a composite score to the data, it wraps a large number of specialized models as independent operators, and then uses feedback from downstream-model training to decide which operators to trust.
[List of model operators acting as quality judges (video side)]
  · CLIP — used in video_frames_text_similarity_filter, computing cross-modal similarity between sampled frames and the text description; this is the core criterion ultimately adopted in the T2V case (threshold 0.306337). In essence, CLIP acts as a "video-text alignment judge," removing samples where the caption does not match the footage.
  · Falconsai/nsfw_image_detection — the underlying model of video_nsfw_filter, acting as a safety judge.
  · Aesthetics-scoring model — video_aesthetics_filter, acting as a visual-taste judge.
  · RAM/image-tagging model — video_tagging_from_frames_mapper, acting as a content annotator.
  · Audio Spectrogram Transformer — video_tagging_from_audio_mapper / video_audio_ASR_mapper, an audio-side annotator.
  · Qwen-Audio — video_captioning_from_audio_mapper, audio understanding and description.
  · General-purpose VLM — video_captioning_from_vlm_mapper, "using a VLM that can directly accept video input to generate video descriptions"; this is the DJ operator that comes closest to the 2026 trend of "large-model semantic judgment," and supports plugging in any video VLM.
  · YOLOE + SAM2 — video_object_segmenting_mapper, object-level semantic evidence.
  · wav2vec2 — speaker age/gender detection.
[The meta-level design of "letting model feedback be the judge" — DJ's most distinctive feature] The Analyze stage of the Sandbox essentially "uses the evaluation scores of downstream generative models to score the quality judges themselves": each candidate operator first splits the data into low/medium/high pools, trains a reference model on each, ranks them via VBench evaluation, and thereby objectively determines "whether this quality-control operator is actually useful for final generation quality." This mechanism answers a question that model-team pipelines generally avoid — "is the filter I added actually useful." The conclusion of the T2V case was reached precisely this way: CLIP-alignment filtering and NSFW filtering were found effective and retained, while the other candidate operators were not selected. This "verify before adopting" stance is more rigorous than simply stacking VLM scorers, and is the point on which DJ's methodology offers the most value for this survey.
[Other forms of LLM involvement in data processing] Starting from v1.4.5, a Ray + vLLM pipeline is supported, enabling efficient batch calls to local large models for annotation/synthesis in a distributed environment; v1.4.6 introduces a "Q&A Copilot"; at the interface layer, AgentScope agents support issuing data-processing instructions in natural language; v1.5.2 adds an agent data-quality toolkit.
[Uncertain] DJ does not build in dedicated video-quality-assessment large-model operators similar to VideoScore or DOVER, nor does it provide an open-ended quality-control operator template of the "VLM gives a composite score with reasoning" style.

### [Foley-Omni](../models/Foley-Omni.md) ⚠️

In this pipeline, Gemini 2.5 Pro plays the dual role of "annotation generation + existence discrimination," a typical case of the 2026 trend of "large models as data quality judges/annotators" — but this work goes one step further: it does not trust the large model's judgment, and instead designs an independent acoustic verification channel to override the large model's output.
[Gemini 2.5 Pro's discrimination duties] It receives video frames together with the corresponding audio track (a genuine dual multimodal input, not vision alone), and is explicitly prompted to perform a two-step detect-then-describe process: first judge whether each of the three component types — Speech / Sound Effects / Music — is "physically present in the clip," and only generate a descriptive caption for components judged present. This step is equivalent to using the large model to perform three-way multi-label discrimination plus conditional description generation. The system prompt template appears in Table 12 of the paper.
[Distrust of and correction for the large model's judgment] The paper explicitly points out that this step has a visual bias — the multimodal model tends to be dominated by visual cues: seeing an instrument leads it to label music present, seeing a mouth open leads it to label speech present, seeing a vehicle leads it to label engine sound present — even when these components may in fact be absent or inaudible in the actual audio track. To address this, a third-level Bandit source separation + energy gating (E(a_c) > −35 dB) is introduced as a purely acoustic, independent criterion; any field that fails to reach the energy threshold is uniformly set to empty.
[Methodological value] This constitutes a "generate-then-verify" dual-path architecture: the verification path does not depend on any neural-network semantic understanding, but is based on physical energy measurement after source separation, and is fully decoupled from the error modes of the generation path. Compared with relying purely on VLM scoring (susceptible to shared-source bias) or purely on signal processing (lacking semantic understanding), this combination is more robust. The paper calls this a two-path validation typically missing in the construction of audio-video datasets.
[Limitations] [Uncertain] The proportion of Gemini-generated annotation fields actually overridden by the Bandit verification stage is not reported — this figure is key evidence for quantifying the severity of VLM hallucination, and its absence means the actual benefit of this design cannot be assessed; no ablation of "with vs. without acoustic post-verification" was performed to demonstrate its contribution to final model performance. It also does not mention using any model to perform a secondary scoring or removal pass on caption quality itself (descriptive accuracy, granularity).

### [Goku](../models/Goku.md)

Overall it still belongs to the "specialized small-model scorer" paradigm and has not yet reached the mainstream 2026 stage of "large-model semantic judge," though several rudiments of model-based quality control have appeared:
[Model-based judgments already in use]
  (1) DINOv2 self-supervised features — cosine similarity used to judge shot-level semantic consistency; a feature-level rather than a language-level judgment;
  (2) Aesthetics-scoring model — a shallow regression scorer, thresholds of 4.3/4.5;
  (3) RAFT optical-flow model — motion-quality judgment;
  (4) OCR model — text-coverage judgment;
  (5) In-house video classification model — semantic tagging across 9 major categories/86 subcategories, serving the distribution-balancing stage; this is the step closest to a "semantic-level model judge," but its role is **classification and mixture ratio** rather than **quality adjudication**;
  (6) Multimodal large models (InternVL2.0, Tarsier2) and LLMs (Qwen2) — but these are used **only for caption generation and fusion**; the paper does **not** use them for quality scoring, text-video misalignment detection, or semantic-plausibility removal.
[Explicit gaps] The paper has no VLM-scoring step, no back-check of caption-video consistency (e.g., CLIP score / VLM consistency judgment), and no LLM audit of caption quality.
[Assessment] Goku (early 2025) represents the "shallow scorer + model-based annotation" stage; compared with later systems such as Seedance and LTX-2, which introduce a VLM as a quality judge for semantic-level filtering and mismatch removal, Goku is clearly a sample from the earlier side of this era divide on this dimension, and can serve as a baseline for observing the trend's evolution.

### [Hailuo / MiniMax Video](../models/Hailuo.md) ⚠️

[Uncertain] (fully undisclosed). No mention of using a VLM/LLM as a data quality judge. Worth noting as indirect background: MiniMax itself has strong multimodal large-model capability (the MiniMax-01 series has VL capability, and M3 is a 427B image-text-input model), and open-sourced the VTP visual tokenizer in December 2025, so its tech stack is fully capable of using its own omni-modal model for data quality control and semantic tagging; but the company has never publicly confirmed whether this is actually used in its video data pipeline. This field therefore falls under "capability exists but is undisclosed," and cannot be counted as evidence for the 2026 trend of "shifting from shallow scorers to large-model semantic judgment."

### [HunyuanVideo-Foley](../models/HunyuanVideo-Foley.md) ⚠️

HunyuanVideo-Foley's approach sits in an intermediate state between "traditional discriminative small models" and "large-model semantic adjudication" — it makes extensive use of dedicated pretrained models for automatic judgment, but no general-purpose VLM/LLM participates in quality adjudication:
[List of models serving as "quality judges" (all dedicated models, not general large models)]
- AudioBox-Aesthetics: four-dimensional audio perceptual-quality scoring, the only "learned quality-assessment model," replacing traditional hand-crafted audio metrics;
- ImageBind: cross-modal semantic-consistency judgment (alignment between audio embedding and visual embedding);
- AV-align: temporal-alignment judgment;
- speech-music detection model + audio classification model: category judgment;
- SNR: a hand-crafted signal metric;
- bandwidth/effective-sample-rate detection: hand-crafted spectral analysis.
[Models serving as "annotators"] GenAU (audio-captioning model), generating audio-content descriptions.
[What's absent] No general-purpose multimodal large model (no Qwen-Omni, no GPT-4o, no Gemini) participates in data quality control, cross-modal consistency adjudication, or mismatch removal; no LLM performs caption fusion, rewriting, or hallucination self-checking; no model performs a secondary quality review of GenAU-generated captions. The entire pipeline is a "dedicated-model pipeline" rather than a "large-model pipeline."
[Relationship to the 2026 trend] Compared with approaches such as MOVA (using GPT-OSS-120B for cross-modal consistency adjudication and hallucination self-audit) that bring general-purpose large models into the quality-control step, HunyuanVideo-Foley (August 2025) still remains at the shallow-scorer stage. It should be noted, however, that its ImageBind semantic-alignment detection already functionally performs the duty of "cross-modal mismatch removal" — only via embedding similarity rather than language-based reasoning, which is far cheaper but much weaker in interpretability and fine-grained judgment. At a scale of 100,000 hours (roughly 45 million clips), having a large model adjudicate clip-by-clip is computationally unrealistic — a hard constraint that scale places on method.
[Degree of automation] The paper's abstract explicitly describes the pipeline as constructed "through automated annotation," i.e., the entire process involves no human intervention.
[Hallucination and misjudgment protection] No detection or constraint against GenAU caption hallucination is described, and no misjudgment rates or human-verification results for the various discriminative models are reported. [Uncertain]

### [HunyuanVideo](../models/HunyuanVideo.md) ⚠️

Both generations make extensive use of models as quality judges, but in different forms, and neither uses "a general-purpose large VLM for end-to-end semantic judgment":
[Original version] Uses a panel of dedicated small/discriminative models — Dover (combined aesthetic + technical view), an in-house blur-detection model, an OCR model, a YOLOX-like detection model, an optical-flow model, TransNet v2 scene detection, an in-house VideoCLIP (deduplication and concept balancing), and an in-house shot-motion classifier (14 classes). This is a typical 2024 "shallow multi-scorer" paradigm. The genuine large model is used only for annotation (an in-house VLM generating structured captions) and prompt rewriting (Hunyuan-Large).
[1.5] Evolves toward "model-based operators" but still does not publicly disclose the use of a general-purpose large VLM for judgment: newly added are a specially trained "transition classifier," a four-dimensional "comprehensive visual-quality assessment operator," an aesthetics-scoring operator, and shot-motion recognition models (clip-level + temporal-level). The clearest evidence of large-model involvement in data construction is on the caption side: the caption model itself is post-trained with OPA-DPO reinforcement learning, specifically optimizing the "descriptive richness vs. factual accuracy" trade-off to suppress hallucination — i.e., using RL to guarantee the reliability of the data annotator, treating the reliability of "the model as annotator" as a first-class citizen, a relatively advanced practice on this dimension. Neither report states whether a VLM is used to remove video-caption mismatches. [Uncertain]

### [InstructAV2AV](../models/InstructAV2AV.md) ⚠️

The multimodal large model in this pipeline plays a dual role — both "instruction generator" (stage two) and "quality judge" (stage three) — with Qwen3-Omni (Xu et al., 2025b) used in both places, a typical sample of the 2026 trend of "deep large-model involvement across the entire data-construction chain."

[Role 1: Instruction generator] Conditioned on the instance-level masks output by Grounded-SAM-2 and the source audio-video context, Qwen3-Omni generates diverse text editing instructions (formulate diverse text instructions for comprehensive editing operations). Choosing the Omni series (which natively supports text + image + video + audio multimodal input) rather than a purely visual VLM is necessary — the instructions must be grounded simultaneously in "what editable objects are in the frame" and "what sound this object makes on the audio track" to produce a coherent joint audio-video editing instruction (e.g., "replace the dog with a cat" implicitly requires that the barking be replaced with a meow).

[Role 2: Quality judge] For each synthesized target, it scores along five dimensions (instruction fidelity / content preservation / perceptual quality / audio-video synchronization / safety), applying "AND" logic — retaining only the samples that simultaneously pass all five criteria. This is a strict conjunctive filter rather than a weighted sum; failing any single dimension results in rejection, a conservative stance.

[Comparison with Foley-Omni's dual-path validation paradigm] Foley-Omni uses Gemini 2.5 Pro to generate annotations, then deliberately introduces an independent purely acoustic channel (Bandit source separation + energy gating) to override the large model's hallucinations, on the grounds that "the generation path and the verification path must have decoupled error modes." This work, by contrast, has the same Qwen3-Omni both generate the instructions and accept the results — the two steps share the same model's biases: if Qwen3-Omni generates an instruction whose execution quality it itself struggles to judge (e.g., a certain kind of fine-grained attribute change), it may equally make an incorrect judgment during acceptance testing, forming a self-consistent blind spot. This is a methodological weakness of this pipeline relative to Foley-Omni in terms of verification independence. Mitigating factors are that the evaluation set additionally undergoes a five-dimensional manual review by 20 people, plus zero-shot validation on the external AvED-Bench.

[Division of labor between human and machine verification] Automated verification covers all 79K training samples (preserving scale); human verification is applied only to the 1K evaluation samples (preserving trustworthiness). The organization of the human side is worth recording: 20 volunteers were organized into 10 judge pairs, with each candidate sample independently assessed by 5 different judge pairs, each pair responsible for only one of the five dimensions — i.e., "one pair of judges looks at only one dimension," a dimension-independent review that avoids the halo effect and cross-dimension contamination that occur when a single person evaluates multiple dimensions at once. This design is comparatively meticulous among peer works.

[Uncertain] Undisclosed: the scoring scale (binary pass/fail vs. continuous score + threshold), the passing threshold for each dimension, the prompt template, the actual rejection rate at the verification stage (a key figure for quantifying the success rate of the data engine, and its absence is notable), and any agreement check between Qwen3-Omni's judgments and human judgments (comparing automated vs. human conclusions on the 1K human-annotated set could have strongly supported the reliability of automated verification, but this was not done).

### [2026 miscellaneous joint audio-video generation works](../models/JAVG_2026_misc.md) ⚠️

This dimension is the part of this batch of work that best embodies the 2026 trend — both ALIVE and NAVA embed MLLM/VLM deeply into the data pipeline, using them not only for annotation but directly for quality control and semantic judgment:
[ALIVE — MLLM for audio quality control and speaker-attribution correction (the deepest application in this batch)]
(1) MLLM performs dual-criterion audio quality control: as the original text states, "We utilize an MLLM model ... to perform audio filtering based on two criteria. Firstly, for audio quality, the model assigns a score to each audio, allowing us to discard samples with significant background noise. Secondly, for audio-visual coherence, we assess the correlation ..." — that is, the multimodal large model simultaneously handles two things: scoring audio quality (removing samples with significant background noise) and judging audio-visual semantic relevance (separating strongly correlated samples and controlling the proportion of weakly correlated samples such as BGM). This is a landmark shift: audio-quality filtering has traditionally relied on signal-processing metrics such as SNR and silence detection, whereas ALIVE switches to a large model directly "listening" and scoring; audio-visual correlation has traditionally relied on contrastive-learning models such as CLAP/ImageBind computing similarity, whereas ALIVE switches to large-model semantic judgment.
(2) Qwen3-omni performs narration timestamp parsing: in the SubjectID correction pipeline, Qwen3-omni (an omni-modal large model) is used to parse narration timestamps — treating the omni-modal LLM as a temporal-alignment annotator.
(3) Gemini 2.5 participates in other parts of the pipeline (mentioned in the paper but its role is not fully specified [Uncertain]).
(4) Traditional scorers are still used alongside: OCR, a pretrained aesthetics model, RAFT, TS-TalkNet, CLIP, ArcFace, a 13-dimensional quality classifier — forming a hybrid system of "large-model semantic judgment + dedicated scorers," rather than complete replacement.
[NAVA — VLM filtering + three Gemini/Qwen models dividing the annotation labor]
(1) VLM-based filtering and tagging: a vision-language model is used to filter and tag clips with clear visual quality, an explicit case of "VLM as quality judge."
(2) An omni-modal tagger works together with YAMNet to perform five-category audio classification — a traditional audio classifier (YAMNet) working in tandem with an omni-modal large-model tagger, the former providing fast coarse classification and the latter providing semantic refinement.
(3) Three-model division of labor at the annotation step: Qwen3-VL (video captioning), Qwen3-Omni (audio captioning), Gemini-3-Flash (fusion/rewriting); the high-quality subset is upgraded to Gemini-3-Pro. Using models at different tiers to handle data at different quality tiers is a typical cost-quality trade-off engineering practice.
(4) "Multi-operator collaborative filtering" produces a 160K SFT subset — the word "collaborative" implies the scores from each operator are combined jointly rather than thresholded serially [the specific aggregation method is uncertain].
[OmniCustom] Uses GLM-ASR for transcription, a dedicated model rather than a general-purpose MLLM; caption construction follows Ovi's attribute system (speaker age, gender, accent, pitch, prosody, emotion, speech rate); the paper does not describe which model generates these attribute annotations [Uncertain], nor does it use a large model for quality control. Filtering relies entirely on three hard rules: SyncNet + aesthetic score + duration.
[CCL / Baton / StreamChar] None use a large model for data quality control [Uncertain]. Baton does have Qwen3-8B as the backbone of its VA-Planner, but that is a model component rather than a data-pipeline role.
[ITS-JAVG — formalizing the "model judge" paradigm and demonstrating its pitfalls (the contribution of greatest theoretical value to this dimension)]
Although it does not concern training data, its core research object is precisely the paradigm of "model as judge" itself, and its conclusions transfer directly to data quality control:
(1) Six verifiers each with their own role: VideoReward-TA (text-video consistency), JavisScore (fine-grained audio-video synchronization), ImageBind-TA (semantic coherence between text and generated audio), AVHScore (semantic consistency between audio events and visual events), VQAScore (text-video alignment), ImageBind (audio-video semantic similarity).
(2) Core finding one: a single verifier necessarily leads to asymmetric trade-offs — "single-verifier guidance effectively improves its intended evaluation metrics, yet fails to achieve a balanced improvement across all metrics," i.e., optimizing only for one judge's score sacrifices other dimensions.
(3) Core finding two: verifier hacking is real — search algorithms will "exploit blind spots" of the judge. This is an important warning for training-side data filtering: if a single model scorer is used for strict filtering, the retained data will be systematically skewed toward that scorer's preferences and blind spots, creating a hidden distortion of the data distribution.
(4) Solution ARW: treats reward aggregation as an online optimization problem, with formula R(i)=Σ_k w_k · r_k(i)/(σ_k+ε) (weighting after normalizing by each judge's score standard deviation), optimizing objective L_ARW=Σ_k (½exp(−s_k)·Var̂(r_k) + ½|s_k|) — i.e., adaptively adjusting weights by the variance of each judge's scores, giving higher weight to judges with higher variance (higher discriminability). This "adaptive normalized aggregation of multi-judge scores" methodology could be transplanted directly onto multi-operator collaborative filtering of training data (coincidentally, NAVA uses exactly "multi-operator collaborative filtering," but has not disclosed its aggregation algorithm).

### [Joint audio-video generation baseline collection](../models/JavisDiT_baselines.md) ⚠️

On this dimension, this collection happens to fully illustrate the three-generation evolution "no model judge → dedicated discriminative model → large-model semantic judge":
[Generation 1: no model judge (MM-Diffusion 2022 / AV-DiT 2024)] The data side uses no models for quality control or scoring at all, relying instead on dataset selection and small-scale manual control. Models are used only for evaluation (i3d computes FVD, AudioCLIP computes FAD), not for data filtering.
[Generation 2: dedicated discriminative models as filters (JavisDiT++ 2026)] Uses a set of shallow/dedicated models as filters and scorers, consistent with mainstream 2024–2025 practice:
- FunASR (Alibaba's open-source speech-recognition toolkit) as a "speech-presence detector," used to remove videos containing human speech — a clever use of an ASR model as a binary-classification filter.
- Aesthetic predictor as the quality judge.
- Optical-flow model as the motion judge.
- OCR model as the text-contamination judge.
[Generation 2.5: multi-reward-model ensemble as preference judge (JavisDiT++'s AV-DPO, the most valuable practice in this collection)] Uses six models, each responsible for a different score, to construct preference pairs — a typical case of "model-as-judge" used for data construction rather than filtering:
- Audio quality → AudioBox (AudioBox-Aesthetics)
- Text-audio alignment → ImageBind
- Video quality → VideoAlign
- Text-video alignment → ImageBind
- Audio-video cross-modal similarity → ImageBind
- Temporal synchronization → Synchformer
and it adopts "normalized modality-aware ranking" to select winning/losing pairs; the authors explicitly state this is done to "ensure consistency within each modality and avoid pairing high-quality audio with low-quality video" — a key piece of engineering wisdom for constructing preference data under multi-dimensional rewards.
[Generation 3: multimodal large models as annotator/judge (Harmony 2025 and JavisDiT's JavisBench)]
- Harmony: uses Google Gemini to automatically annotate all 4 million clips (ASR transcription + video description + background-sound description), and uses an "audio-video consistency scoring model" to filter speech data — the latter directly hands cross-modal consistency judgment to a model, though the identity, scoring dimensions, and threshold of this model are all undisclosed [Uncertain].
- JavisDiT's JavisBench: uses "advanced Qwen-family models" to simultaneously perform caption generation and 19-class scene categorization, i.e., the MLLM acts as both annotator and classification judge.
[UniAVGen] No mention of any model-based data judge [Uncertain].

### [Kling 3.0 / Kling Video 3.0 Omni](../models/Kling_30_Omni.md) ⚠️

Yes, and it is a core step in Kling 3.0's data system, embodying the 2026 trend of "shifting from shallow scorers to large-model semantic judgment." The third level, "multimodal alignment," in Kling-Omni is handled entirely by a model acting as judge: it determines the semantic consistency between the video caption and the actual visual content, the fidelity of the reference image to the target video, and identity consistency for human-centric tasks (cross-frame ID consistency checking). The synthetic-data side is likewise driven by "expert models" (an in-house image-editing model + a video-understanding model). Corroborating evidence from the same team: Koala-36M replaces manual threshold combinations with a trained VTSS neural network and uses a fine-tuned LLaVA to generate structured long captions; Kling-Foley uses CLAP to compute the consistency between audio and text tags and retains only highly consistent data, uses an audio-understanding large model to classify and generate descriptions, and uses an LLM to extract metadata and fuse the audio and visual descriptions. Kling's official materials also publicly state that its data solution covers "massive-scale video mining, multi-dimensional annotation and filtering, video-description enhancement, and data-driven quality assessment." [Uncertain: the specific name and parameter scale of the in-house VLM acting as judge]

### [LTX-2](../models/LTX-2.md) ⚠️

The LTX series does not use a VLM/LLM as a data quality judge — a notable point of divergence from the 2026 mainstream trend. Everything performing discrimination duties in the data pipeline is a shallow/dedicated model: a CLIP image encoder (for deduplication representations), a multi-label classification network (for sampling pairing), a Siamese aesthetics-ranking network (for quality scoring), and a motion-magnitude estimator. Large models in LTX-2 play a role concentrated in two places, neither of which is quality control: (1) an in-house captioner (dual-track video + audio annotation); (2) Gemma3-12B as a frozen text encoder participating in conditioning. The technical report makes no mention of using a multimodal large model for semantic quality scoring or image-text/audio-video mismatch removal. Whether it is used in an undisclosed internal process cannot be determined. [Uncertain]

### [LongCat-Video](../models/LongCat-Video.md)

This model makes extensive use of multimodal large models as annotators and quality judges, a core method in its data pipeline, but leans more toward "model-based annotation" than "model-based ranking/removal":
(1) Caption generation: a fine-tuned LLaVA-Video (fine-tuned with in-house synthesized pairs) serves as the primary captioner, with Tarsier2 annotations introduced to strengthen temporal understanding;
(2) Qwen2.5VL performs two kinds of structured discrimination — determining shot size and lens type, and determining visual style (realism, animation style, color tone);
(3) Camera motion (pan/tilt/zoom) uses a specially trained classifier rather than a VLM;
(4) after caption embeddings are clustered, an LLM performs category induction and naming — i.e., the LLM is used to define the data taxonomy;
(5) The reward model on the post-training side is likewise essentially a model quality judge: a Motion Quality model fine-tuned from the VideoAlign base (input grayscale video, forcing it to attend to motion only and ignore color) and a Text-Alignment model (input color video), plus HPSv3-general/HPSv3-percentile visual-quality scoring.
Avatar 1.5 further uses the VLM as both a hard filter and a consistency judge: for silent data, an initial pass by Qwen3-Omni is followed by a second pass from Qwen3-VL, and only samples on which the two models agree are retained; for emotion data, EmotiEffLib performs fine filtering with a confidence threshold s > 0.7, and hard exclusion rules are set (content containing synthetic material, more than two subjects, identity switches, or a subject occupying too small a fraction of the frame is all labeled null); on the caption side, Qwen3-Omni generates three-dimensional contextual descriptions — "spatial environment / character relationships / plot progression" — and is required to keep the descriptions focused on objectively observable physical presentation.
The report does not state whether the base version used a VLM to explicitly remove caption-video mismatches.

### [MOVA](../models/MOVA.md)

MOVA embodies the 2026 typical form of "deep large-model involvement in the data pipeline," but the division of labor is clearly split — **the large model handles semantic annotation and cross-modal consistency adjudication, while dedicated small models handle quality and alignment scoring**:
[Large model as semantic adjudicator (the core usage)] In the caption-merging step, GPT-OSS-120B does more than text concatenation — it performs an explicit cross-modal consistency check: "the model verifies the alignment between visual scenes and audio events to resolve potential conflicts," then synthesizes this into a unified natural-language description. This effectively uses a 120B-scale LLM as the judge of "whether the audio-visual content is semantically self-consistent," making it the component of this pipeline that comes closest to "LLM-as-judge."
[Anti-hallucination self-audit design] Both the visual- and speech-annotation prompts have a built-in final_verification_audit self-check field (hallucination_check_passed, visual_changes_verified / speech_dynamics_verified, and comment), requiring the MLLM to output a structured self-audit conclusion; and a strongly enforced "LAW" system is used to suppress cross-modal crosstalk — the visual annotator is explicitly instructed to "Ignore audio and inferred context entirely" and "Do not infer or hallucinate based on audio or context," while the speech transcriber is explicitly instructed to "Ignore non-speech sounds and music entirely." This is a defensive design against the most typical failure mode of multimodal annotators (inferring one modality from another).
[Quality filtering still handled by dedicated scorers] Audio uses Audiobox-aesthetics (PQ/CU/CE three-part scores), video uses DOVER (technical/aesthetic dual scores), alignment uses SynchFormer (temporal) and ImageBind (semantic), audio classification uses EAT, and lip-sync uses the SyncNet family's LSE-D/LSE-C. All of these are dedicated discriminative models rather than general-purpose VLMs.
[Conclusion] MOVA does not go so far as to "use a VLM to produce a single composite quality score replacing all shallow scorers"; instead it adopts a hybrid division of labor — "dedicated scorers handle quality and alignment + large model handles semantic annotation and consistency" — which can be seen as a pragmatic compromise on this trend.
[Limitations] The paper acknowledges in its Limitations section that annotation reliability is the bottleneck in multi-speaker scenes: errors in speaker diarization and imperfect active-speaker labels propagate into training, causing the model to confuse speakers or learn inconsistent supervision signals.

### [Merged entry: ① Mochi 1](../models/Mochi_MAGI_Motif.md) ⚠️

These three models happen to fully illustrate the evolution of "VLM as quality judge" from absence to becoming the pipeline's pivot point, making this the dimension of highest comparative value in this entry:
① Mochi 1 (2024.10): No disclosure at all, and no evidence that a large model was used for scoring. [Uncertain]
② MAGI-1 (2025.05, MLLM as an "advanced filter" positioned at the end of the funnel): The report devotes an independent subsection, 3.3 "MLLM as Advanced Filter," and the logic is very clear — the preceding Filter Actors and deduplication have already removed the vast majority of bad data, but, constrained by the capability of those filters, a small amount of low-quality data remains; at this point the data volume has shrunk considerably, so a multimodal large model is introduced for a further round of filtering, "enables us to detect more complex bad cases." The most important cost-related design choice is: "Notably, this step can be seamlessly integrated into the subsequent caption procedure, thereby reducing overall costs and improving efficiency" — i.e., scoring and annotation reuse the same MLLM call. However, the paper does not name the specific MLLM model or its scale, nor does it state which dimensions it judges or what structured output it produces. [Partially uncertain: MLLM model and judgment dimensions]
③ Motif-Video 2B (2026.04, VLM upgraded from "end-of-pipeline second pass" to "metadata source running throughout the process"): Its caption-as-metadata design pushes VLM scoring into a new form — a single forward pass of Qwen3-VL-30B-A3B simultaneously outputs, according to a fixed JSON schema, a free-text caption and structured labels (subject, style, action, camera_move, quality, watermark, nsfw, as well as padded, multi_scene, timelapse); these labels are then reused across four downstream purposes: (i) text conditioning during training; (ii) sanitation (nsfw / watermark / padded / multi_scene / timelapse / quality directly trigger hard deletion); (iii) domain and subject-balanced sampling (style / subject driving domain balancing for 720p and SFT); (iv) dynamic motion filtering for 720p SFT (action=Dynamic). The paper also states explicitly, "we do not apply a separate post-processing model to reinterpret them" — the labels are consumed directly by downstream stages as produced by the VLM. Its key methodological claim is: "because these labels and the training caption originate from the same forward pass, filtering and conditioning remain synchronized by construction" — an explicit design stance that fully unifies "model-based quality control" and "model-based annotation," eliminating semantic drift between the two, and represents the mature 2026 form of this paradigm.
Viewed together across the three: Mochi 1 (none) → MAGI-1 (MLLM end-of-pipeline second pass, reusing the annotation call to save cost) → Motif (VLM forward pass as the metadata source, filtering and conditioning synchronized by construction) — the direction of evolution is the VLM's position in the pipeline moving continuously earlier and its responsibilities continuously expanding.

### [Movie Gen](../models/Movie_Gen.md)

Movie Gen is representative of the "2024 paradigm": data quality control relies almost entirely on a set of dedicated lightweight discriminative models/classifiers and scorers rather than on a general-purpose VLM/LLM for overall semantic quality judgment and image-text mismatch removal; the large model here (LLaMa3-Video) plays the role of annotator (caption generation) rather than judge. The specific list of models:
Visual side — an in-house OCR model (dual scores for word detection + word recognition); an in-house black-border detector (non-learned); the LAION aesthetics image-aesthetics prediction model; several in-house-trained frame-level visual models (aesthetics/image quality/heavy borders/visual effects); an in-house static-video-detection model; PySceneDetect and FFmpeg (scene/motion); a copy-detection embedding model (SSCD); a video-text joint-embedding model (for clustering and k-NN retrieval); a 16-class camera-motion classifier; a Detic object-detection model (removing small subjects); an ArcFace face-recognition model (for PT2V identity consistency, threshold 0.5 across frames and 0.7 for synthetic reference images); face-detection and face-region-segmentation models.
Audio side — an AED audio-event-detection model (the AudioSet 527-class ontology); a CAVTP contrastive audio-video-text pretraining model (outputting an audio-video cosine similarity, used for diegetic determination and bucketing); an audio-quality prediction model (outputting a continuous 1–10 score, annotated in a manner mirroring the LAION aesthetic approach); a general-purpose audio-captioning model; a music-captioning model; a cinematic audio-video classifier.
Notably, the audio-quality score is not used as a hard filtering threshold, but is instead written into the caption to become a controllable condition (a quality of 7.0/8.0 can be specified at inference time) — a form of "converting the judge's score into a condition." Semantic-level judgment at the post-training stage, meanwhile, is handed to human annotators rather than a large model.

### [NVIDIA NeMo Curator](../models/NeMo_Curator.md)

Shows a clear generational evolution, making it a good sample for observing the 2026 trend "shallow scorers → large-model semantic judgment":
[25.09/early Cosmos production version: mainly shallow discriminators] Quality-control duties are handled by lightweight models — a CLIP-based aesthetics model, the DOVER quality model, InternVideo2 embeddings + an MLP (text-overlay detection, video-type classification), a ViT optical-flow motion classifier, and motion-vector statistics. All are small models or linear/MLP probes, motivated by per-unit cost at PB-scale.
[26.02–26.07: VLM/LLM gradually enter the pipeline] (1) The captioning backend expands from Qwen2.5-VL to Qwen3-VL, Nemotron Nano 12B V2 VLM, and Nemotron 3 Nano Omni, offering BF16/FP8/NVFP4 precision tiers to reduce large-model inference cost — FP8/NVFP4 quantization is the precondition for running a 12B-scale VLM at PB-scale; (2) 26.04 introduces vLLM as the default inference backend and builds in a Ray Serve inference server (an OpenAI-compatible interface), making "hanging an LLM on the pipeline to make judgments" a first-class citizen; (3) from 26.02, captioning supports an optional Qwen-LM second-pass rewrite enhancement; (4) in the 26.07 Nemotron OCR synthetic-data pipeline, Nemotron Nano Omni is explicitly used for optional quality scoring — the most direct official-documentation usage of "large model as data quality judge."
[Still missing] On the video side there is still no dedicated removal stage where "a VLM judges whether the caption and the footage mismatch," nor a VLM semantic quality-scoring stage; large models are currently used mainly to generate captions rather than to audit them. This step must be built by the user.

### [OmniHuman dataset + OHBench](../models/OmniHuman.md) ⚠️

OmniHuman is one of the cases in this survey that best fits the "2026 large model as quality judge" trend, and its usage of large models shows clear layering:
[Gemini-3: semantic adjudicator] Used for "background plausibility and interaction assessment" — that is, a closed-source flagship multimodal model judges the semantic plausibility of the scene and the naturalness of human interaction. This is a typical case of using a large model to replace shallow scorers for semantic-level judgment, covering a dimension that shallow metrics (aesthetic score, sharpness) simply cannot reach.
[Gemini-3 / Gemini-3-pro: primary judge on the evaluation side] In OHBench it handles four subjective dimensions scored on a 1–10 scale: Background Plausibility (global level), Interaction Naturalness (including eye contact, body gestures, and response accuracy — relational level), Object Consistency (temporal stability of appearance/position/state), and Contact Naturalness (spatial accuracy/temporal synchronization/force realism). This is the most aggressive step in this work — semantic/physical-plausibility dimensions that traditionally had to be scored by humans are handed over wholesale to Gemini-3, and this is used as the basis for a metric claimed to be "highly consistent with human perception."
[Qwen3-Omni: annotation generator, not adjudicator] Serves as the inference core for caption generation, responsible for extracting attributes and synthesizing narratives, without taking on quality adjudication.
[The most valuable design: a cross-validation closed loop] The consistency check at the end of the pipeline is this work's substantive design against "MLLM hallucination," and it does not rely on a bigger model but on redundancy across modalities:
- The number of structured subject labels must exactly match the person count output by the tracking module (YOLOv11 + MOTRv2) — a visual-geometry pipeline is used to check the MLLM's counting hallucinations;
- The number of speakers mentioned in the caption and the transcript content must stay within an acceptable edit-distance range compared with the upstream ASR (FunASR-Nano) transcription — the ASR result is used to check the MLLM's hallucinations about speech content;
- Only videos that pass all checks are retained.
This approach of "using the output of a deterministic dedicated model to constrain the output of a generative large model" is, in principle, more reliable than MOVA's use of GPT-OSS-120B for hallucination_check (i.e., using one large model to check another), because the error modes of tracking and ASR are independent of the MLLM's hallucination modes.
[Limitations] The numeric value of the "acceptable" edit-distance boundary is not given; Gemini-3's decision thresholds and prompts are undisclosed; heavy reliance on a closed-source API model (Gemini-3) makes the reproducibility and long-term stability of the entire pipeline dependent on a third-party service, and the paper does not mention the cost of calling a closed-source API across a million-scale sample set. [Uncertain]

### [Open-Sora series](../models/Open-Sora.md)

Both **remain at the "shallow scorer" stage and have not yet shifted to the "large-model semantic judgment" paradigm** — every filtering criterion (aesthetic score, optical-flow/VMAF/LPIPS motion score, Laplacian blur, DOVER technical score, OCR text count) is a scalar score plus a threshold, output by a lightweight dedicated model or a traditional CV operator; not a single filtering stage involves semantic-level judgment by a VLM/LLM.
The only mechanism approaching "model as judge" is the **matching score (image-text matching score) in Open-Sora 1.x**: **CLIP** is used to compute the cosine similarity between the video's middle frame and the caption (`tools/scoring/matching/inference.py`, output column `match`), used to remove image-text-mismatched samples. But CLIP's semantic discrimination power is far weaker than that of contemporary VLM judges, and it takes only a single frame, without regard to time.
In both projects, the VLM's role is that of an **annotator, not a judge** (PLLaVA-13B / LLaVA-Video / Qwen2-VL-7B / Qwen2.5-Max are responsible only for generating captions; their output is not used to score sample quality or make removal decisions). Open-Sora Plan's post-processing of Qwen2-VL captions is also just a rule table of 28 common opening phrases used to strip prefixes such as "The video shows" — rule-based cleaning rather than model-based judgment.
Comparative significance: this is precisely where the main generational gap lies between 2024–2025 open-source reproduction projects and 2026 frontier practice — the Open-Sora series pipeline can be seen as a complete technical snapshot of "before VLM-as-judge."

### [Ovi](../models/Ovi.md) ⚠️

Ovi is at the stage of "MLLM used for annotation rather than for quality scoring," which only partially overlaps with the mainstream 2026 route of "large model as semantic judge."
[Model-based judgments already used]
(1) MLLM as annotator: given 7 uniformly sampled frames of images plus the complete audio track as input, it produces a long caption interleaving visual events and dialogue lines, along with a structured audio description. The paper emphasizes that it "conducted extensive experiments to ensure the captioning included all relevant visual and audio events while respecting chronology" — i.e., extensive prompt experimentation was done to ensure completeness of events and correctness of chronological order, an indirect form of quality control over the MLLM's output, but it does not describe using a second model to verify whether the caption mismatches the content.
(2) Discriminative models as filters: SyncNet (synchronization judge), RAFT (motion judge), the LAION aesthetic predictor (aesthetic judge), and an in-house face-detection model (composition judge) — all shallow/dedicated scorers, consistent with typical mid-2025 practice.
[Means not used] No VLM/LLM is used to give an overall quality score for a clip, no large model is used for a secondary "caption vs. video semantic mismatch" removal pass, no large model is used for content-compliance judgment, and there is no model-as-judge scoring threshold. On this dimension overall, this is a relatively weak point for Ovi compared with strong contemporaneous pipelines (e.g., HunyuanVideo, which replaces T5 with an MLLM and pairs it with a robust data filter). [Uncertain: whether an undisclosed internal MLLM quality-control step exists]
[MLLM identity] Throughout, the paper writes only "an MLLM," without specifying whether it is GPT-4o/Gemini/an in-house model, or its scale. [Uncertain]

### [Script-a-Video](../models/Script-a-Video.md) ⚠️

This work is a typical case of "large model as annotator," but strictly speaking it does not use the large model as a judge of data quality — instead it is used as a generator and as a judge of output (evaluation judge):
[Role 1: Gemini-2.5-Pro as annotation generator] All 500K clips of MTSS annotation are generated automatically by Gemini-2.5-Pro alone, with no other model involved, no multi-model voting, and no consistency cross-checking. The reasonableness of choosing Gemini-2.5-Pro is supported by Table 1: it achieves a composite score of 93.97 on UGC-VideoCap (95.51 for the MTSS version), and a total error rate of 0.3959 on Video-SALMONN-2 (0.2511 for the MTSS version) — the strongest model in the entire table, giving assurance of quality as the teacher model.
[Role 2: Gemini-2.5-Pro as evaluation judge] All caption and reasoning benchmark evaluations in Table 1 and Table 2 use Gemini-2.5-Pro as the judge model. There is a methodological self-referential risk here: the annotator of the training data and the judge of the evaluation are the same model, which may carry a systematic preference for the MTSS format; the paper does not discuss or mitigate this potential bias.
[Role 3: Gemma4-31B-it as generation-quality judge] On the generation side, the Semantic Following metric is scored independently by the Gemma4-31B-it vision-language model, averaging four sub-dimensions after scoring: Subject (identity and appearance fidelity), Action (correctness of motion and interaction), Scene (accuracy of background/environment/spatial layout), and Style (adherence to visual style/color scheme/atmosphere). This is a standard use of VLM-as-judge.
[Missing step: annotation quality control] The paper describes no quality-control mechanism at all for the results annotated by Gemini-2.5-Pro — no hallucination detection, no format-validity check (MTSS is a JSON-like structure that in principle requires parsing validation), no Reference-ID reference-integrity check (whether a Shot's references_in_shot and an Event's speaker both point to existing IDs), no timestamp-validity check (whether time_range is out of bounds or overlapping), and no manual spot-checking. For a structured schema that heavily depends on cross-reference integrity, this is a fairly notable methodological gap. [Uncertain]
</content>

[Related limitation acknowledged by the paper itself] The Limitations section acknowledges: generating accurate, deeply structured scripts places extremely high demands on the cross-modal understanding ability of the foundation model; current open-source MLLMs still have limitations in precise temporal localization, robust ASR, and accurate audio-visual entity-event association; how to get a more compact open-source architecture to reach Gemini-level scripting ability while effectively suppressing hallucination remains an open problem. This statement, in reverse, confirms that annotation quality depends heavily on the teacher model itself rather than on post-hoc quality control.

### [Seedance 2.0 and Seedance 1.5 pro](../models/Seedance_20_Seedance_15_pro.md) ⚠️

The Seedance series has substantively moved toward "model as quality judge," but disclosure is limited: Seedance 1.0 uses a dedicated visual-quality model to determine defects, an in-house video-representation model for semantic clustering and deduplication, an in-house captioning model based on the Tarsier2 backbone for content understanding, and a Foundational Reward Model with a VLM-based architecture to assess image-text alignment and structural stability. Seedance 1.5 pro states that its captioning system can provide "rich, professional-grade descriptions" for both video and audio modalities, consistent with the large-model semantic-judgment route. The Seedance 2.0 report does not disclose details of its quality-control models, but the same team has access to the Seed-VL series of multimodal understanding models (listed in the introduction as used for cross-modal semantic understanding), suggesting these have likely already been used in data-discrimination steps. At the evaluation level, SeedVideoBench 2.0 explicitly separates "objective metrics via an automated pipeline" from "subjective metrics via blind expert review." [Uncertain: whether 1.5/2.0 introduced a VLM scorer and, if so, its scale, scoring dimensions, and removal threshold]

### [SkyReels series](../models/SkyReels.md) ⚠️

The SkyReels series is a typical representative of the trend "large models sinking down into the data stage," and the role positioning has evolved noticeably across the two generations:
[SkyReels-V2 (2025)] Large models mainly handle annotation rather than quality control: knowledge from a general-purpose MLLM (Qwen2.5-VL-72B-Instruct as the general description source) plus three sub-expert models (a shot captioner, an expression captioner, and a camera-motion captioner) is distilled into SkyCaptioner-V1 (based on Qwen2.5-VL-7B-Instruct). Quality-control duties remain with shallow/dedicated discriminators: an aesthetics filter, an OCR filter, a mosaic filter, a special-effects/sticker filter, and VQA/IQA models.
[SkyReels-V4 (2026)] An omni-modal large model is deeply involved at three key points in the data chain:
(1) The segmentation step — a VLM works together with TransNet shot-boundary prediction to judge the semantic completeness of a clip (using the VLM to decide "where to cut," rather than relying only on pixel-level scene-change detection);
(2) The audio classification and annotation step — Qwen3-Omni uniformly performs four-category audio discrimination and generates audio captions, replacing traditional audio-event classifiers;
(3) The transcription step — Whisper performs speech/singing content recognition.
This change corroborates the industry trend of "shifting from shallow scorers to large-model semantic judgment." However, the paper does not describe an explicit step in which a VLM produces a semantic quality score for samples or removes image-text/audio-video mismatches, nor does it give the accuracy or compute cost of the large-model discrimination. [Uncertain]

### [Sora 2](../models/Sora_2.md) ⚠️

Whether a VLM/LLM is used as a quality judge on the training-data side is undisclosed. But on the inference safety side, OpenAI explicitly uses a large model as judge: at the output-blocking stage it deploys "a safety-focused reasoning monitor... a multimodal reasoning model which is custom-trained to reason about content policies," making semantic-level judgments on generated video frames, scene-description text, and audio transcripts. This proves that OpenAI already possesses and deploys a "large-model semantic judgment" capability stack, and reusing similar capability for training-data quality control would be technically natural, though the System Card makes no such statement. In addition, in the safety evaluation process, adversarial prompts are fed to a "helpful-only version of the video model" whose outputs are then scored and converted into an automated evaluation, which likewise reflects a paradigm in which models participate in constructing and judging data. [Uncertain]

### [SpeakerVid-5M](../models/SpeakerVid-5M.md)

SpeakerVid-5M is at an intermediate stage of "deep large-model involvement in annotation, but quality adjudication still handled by dedicated models," similar in division of labor to MOVA:
[The MLLM's one actual use as a scorer] Qwen2.5-VL is used as a motion-intensity scorer, using a three-persona ensemble (expert/audience/annotation-expert perspectives, each on a 1–5 scale), removing outliers and taking the mean. This is a typical case of handing a subjective dimension hard to capture with shallow metrics — "the magnitude of motion and interaction liveliness" — to an MLLM for judgment, using multi-perspective ensembling to mitigate the instability of a single scoring pass — a more cautious approach than single-prompt scoring. This score subsequently becomes one of the hard filtering conditions for the HQ subset (> 2).
[LLM as classifier] Qwen-3 is used for classifying dialogue topic category.
[MLLM as describer] Qwen2.5-VL generates structured captions (camera motion, entity list, body orientation, half-body/full-body, expression, action description, etc.).
[Quality adjudication still handled by dedicated models] DOVER (video quality), SyncNet (lip-sync), ArcFace (face identity), Whisper (ASR confidence and speech-absence probability), Laplacian variance (blur), a custom clarity formula (sharpness) — all six of these hard filters are completed entirely by dedicated discriminative models or traditional signal metrics, with none replaced by a VLM composite quality score.
[Approaches not adopted] No LLM-as-judge-style cross-modal consistency verification (compare MOVA's use of GPT-OSS-120B for audio-visual semantic self-consistency adjudication); no caption-hallucination self-audit mechanism; no use of an MLLM to double-check filtering results.
[Overall positioning] The MLLM here is responsible for "annotation and subjective scoring," not "quality adjudication and mismatch removal" — an early/conservative form on this trend.

### [Step-Video-T2V](../models/Step-Video-T2V.md) ⚠️

On this dimension, this entry clearly leans toward "traditional scorer" rather than "large-model semantic judgment," making it a baseline sample for observing the paradigm shift from early 2025 to 2026:
[List of models handling quality control] All are dedicated small models or classic algorithms — the LAION CLIP aesthetic predictor, the CLIP ViT-L/14 NSFW binary classifier, an EfficientNet watermark classifier, PaddleOCR, Laplacian variance, HSV statistics, FFmpeg black-border detection, Farneback optical flow, an in-house VideoCLIP (clustering and outlier detection), and CLIP Score (image-text alignment). Not a single step uses a multimodal large model for end-to-end semantic quality judgment or physical-plausibility judgment on video.
[Where the large model appears] An in-house VLM appears only in the annotation step (generating short/dense captions), not in scoring or removal duties; no dedicated post-training for hallucination suppression is applied to this captioning VLM either (compare HunyuanVideo 1.5's use of OPA-DPO to govern annotator hallucination).
[The only model-level mismatch removal] The CLIP Score image-text alignment filtering at stage 6 is "using a discriminative contrastive model for coarse-grained semantic matching" — shallow semantic judgment rather than VLM-style reasoning-based judgment.
[Humans substituting for the large model's role] Semantic-level quality judgment at the SFT stage (sharpness, aesthetics, motion plausibility, smoothness of scene transitions, caption accuracy) is completed by human review rather than handed to a VLM — precisely the part the 2026 trend aims to replace. Step-Video-T2V can therefore serve as a reference sample from "before the large-model quality-judge paradigm." [Uncertain]

### [UniTalking](../models/UniTalking.md) ⚠️

UniTalking shows an orientation opposite to the mainstream 2026 trend — large models in its pipeline are used only for annotation, taking no part in quality adjudication at all, with quality control entirely handed to traditional dedicated discriminative models:
[The large model's role: annotation only] Qwen3-VL (video captioning), Whisper-V3 (speech transcription), Qwen3-Omni-Captioner (audio captioning), and Qwen3-Omni (audio-video fusion description) — all four models sit entirely in the annotation step, positioned after three-stage filtering. The paper describes no quality scoring, cross-modal semantic-consistency checking, caption-content mismatch detection, or hallucination self-audit mechanism performed by a VLM/LLM. This lags a generation behind MOVA's approach of using GPT-OSS-120B for cross-modal consistency adjudication with a built-in hallucination_check.
[Quality control entirely handled by dedicated discriminative models]
- PANNs: a CNN model for AudioSet 527-class audio-event classification, used for speech-event detection (determining whether speech is present in the audio track);
- SentenceASD: works together with PANNs to perform speech-event detection;
- LightASD: a lightweight Active Speaker Detection model, used to determine whether the speaking person is on screen — the most targeted tool choice in this pipeline, as ASD is precisely the standard solution for "audio-visual co-source" determination;
- LipSync: a lip-sync evaluation model, used to remove samples with poor lip-audio alignment. The paper does not specify whether the concrete implementation is SyncNet, Syncformer, or another variant. [Uncertain]
[Video-quality judgment models entirely undisclosed] The paper says not a word about what model underlies the three rules "static / text overlay / low visual quality." [Uncertain]
[Missing quality-assurance mechanism at the annotation step] Three captioning tracks are independently produced by three different models and then concatenated, and a fourth track is produced by fusing them with Qwen3-Omni, but there is no cross-track consistency check, no hallucination filtering, no manual spot-checking, and no caption quality scoring. Given that Qwen3-VL is prone to cross-modal hallucination when it can hear audio while describing video, and the paper states neither whether modal isolation was applied nor whether any protection exists, this is an uncontrolled risk point on the annotation side. [Uncertain]
[One point worth crediting] The tool selection itself fits the task well: the ASD + LipSync combination precisely maps onto the two core failure modes of the "talking-head video" domain (the sound source off-screen / the sound source on-screen but not synchronized), which is more targeted than general AV work relying on a single SyncNet threshold.

### [UniVerse-1](../models/UniVerse-1.md) ⚠️

UniVerse-1's mode of large-model involvement diverges noticeably from the 2026 mainstream trend — the large model handles only annotation, not quality control, with quality judgment still handed entirely to traditional discriminative small models:
[The large model's role: annotation only, no adjudication] QWen2.5-Omni, the only multimodal large model, is responsible for generating three-way aligned annotations online during training (speech content, video description, ambient-sound description). The paper describes no quality scoring, cross-modal consistency checking, or conflict resolution performed by an LLM/VLM — a clear gap compared with MOVA's use of GPT-OSS-120B for cross-modal consistency adjudication with a built-in hallucination_check self-audit block in the prompt.
[Quality control still handled by dedicated discriminative models] DOVER (video quality), Whisper (speech-presence determination + ASR), RetinaFace (face detection), SyncNet (lip-sync confidence), plus three signal-level statistics (volume/energy/zero-crossing rate). All are dedicated small models or hand-crafted features, none a general-purpose large model.
[The real innovation lies in "when to annotate" rather than "who annotates"] The paper's core claim is: traditional offline annotation (pre-generating one caption for an entire long video) causes a temporal mismatch between the annotation and the actual training window — the content of a randomly sampled 5-second window during training does not correspond to a global caption covering the whole video, and this "misalignment [in] text-based annotation" directly harms the generative model's performance. The solution is to move annotation inside the training loop: an independent service process running concurrently with training generates annotations on the fly for each actually-sampled fixed-length window, ensuring that "every frame, every segment of sound, and every sentence of text fed into the model" come strictly from the same source and the same window. This redefines the data-annotation alignment problem from a "quality problem" to a "timing problem," and is this work's most methodologically valuable contribution on the data side.
[Cost] Online annotation means every training step incurs an inference cost from QWen2.5-Omni + Whisper, requiring an independent annotation-service cluster running in parallel with the training cluster; the paper does not disclose this service's compute overhead, throughput, or whether it becomes a training bottleneck.
[Hallucination protection] The paper describes no constraint design against cross-modal hallucination in multimodal annotators (such as prohibiting the visual annotator from referencing audio), and the annotation prompt text itself is not disclosed. [Uncertain]

### [Unison](../models/Unison.md) ⚠️

Unison makes no use of a VLM/LLM as a quality judge on the training-data side at all — a notable departure from the 2026 mainstream trend of "large-model semantic judgment replacing shallow scorers." All judgment of training data is completed by traditional discriminative small models: a face detector (model unspecified), SyncNet (lip sync), and Mel-RoFormer (audio-source separation, a preprocessing step rather than a judgment step). There is no cross-modal consistency check, no semantic-matching adjudication, no hallucination detection, and no VLM quality scoring.
[The large model's involvement occurs entirely on the evaluation side, and is quite deep — this is the key finding of this field] Unison uses Gemini in two evaluation steps:
1) Generating test-set annotations — "a curated test set of 1,000 held-out samples, with ground-truth annotations provided by Gemini to ensure rigorous T2AV and TI2AV assessment," i.e., the ground-truth annotations (captions and transcriptions) for the 1,000 test samples are generated by Gemini, supporting evaluation of both the T2AV and TI2AV tasks;
2) LLM-as-judge subjective scoring — in addition to the ranking votes from 25 human participants in the user study, an extra "Gemini Score" column is added, in which Gemini scores the "Motion-Speech-SFX coherence" of the generated results. The results show that Gemini's scoring aligns closely with the human ranking (Unison 1.68 best, LTX-2 2.05, MOVA 2.48, UniAVGen 3.48), forming a cross-validation of human evaluation. This is a concrete instance of using a multimodal large model as an "audio-visual coherence judge," with reference value for audio-video generation evaluation.
[A notable asymmetry] The team trusts Gemini enough to use it for generating ground-truth annotations and as an evaluator, yet has not applied the same capability to quality control and annotation of the training data — the paper does not explain this asymmetry, which may be cost-related (calling Gemini on 2 million clips one by one is far more expensive than on 1,000 test samples), or may be because the training annotations directly reuse captions that come with the upstream datasets (CelebV-Text and OpenHumanVid both ship with their own text annotations). [Uncertain]
[Comparison with peer works] MOVA uses GPT-OSS-120B for cross-modal consistency adjudication with a built-in hallucination self-audit, and UniVerse-1 uses QWen2.5-Omni for online three-way annotation — Unison lags behind both on model-based quality control of training data, but goes further than both in its LLM-as-judge application on the evaluation side.

### [Veo 3 / Veo 3.1](../models/Veo_3_Veo_31.md) ⚠️

Veo 3 is a typical representative of the trend "deep large-model involvement in the data pipeline," but this involvement is mainly in the form of "annotation" rather than "scoring." Confirmed items: (1) multiple Gemini models are used to generate captions at different granularities for audio-video data — direct evidence of VLMs acting as data producers; (2) a multimodal classifier is used to detect content-policy violations, and the official documentation specifically emphasizes the necessity of being multimodal — "caption and video, viewed separately, may each be harmless, but the combination may produce a harmful result" (the example given: a text prompt "an image of a pig" paired with a video of a certain group of people would constitute a harmful representation) — this is in essence a cross-modal semantic-mismatch/harmful-combination discriminator, conceptually highly consistent with "VLM as data quality judge," though officially it is positioned within development-period safety monitoring (development evaluations) rather than an explicit training-data-cleaning step. [Uncertain] Whether Gemini/a VLM is used to score videos for aesthetics or semantic quality, whether a large model is used to remove caption-video semantic mismatches, and the scale and threshold of any scoring model are all undisclosed.

### [Vidu S1](../models/Vidu_S1.md) ⚠️

This is the report's most prominent methodological claim on the data side, embodying the 2026 trend "shifting from shallow scorers to large-model semantic judgment": the paper explicitly points out that evaluating video data with expert models only has clear limitations — for example, face-detection models struggle to generalize to exaggerated or highly stylized 2D animated subjects, and some expert models are image-based, able to judge only sampled frames, producing misjudgments due to insufficient global information. It therefore introduces an omni model (an omni-modal large model, citing references [24,25]) to perform global semantic understanding of the complete video as a supplement, producing semantic labels along nine quality dimensions: editing, subject, action, emotion, face, speech, scene, shot, and tone. The expert models and the omni model together form a joint filtering system that combines global contextual awareness with sensitivity to local detail. The specific identity and scale of the omni model are undisclosed [Uncertain]. In addition, caption annotation is likewise performed by an annotation model.

### [Wan 2.5 / 2.6 / 2.7](../models/Wan.md) ⚠️

The Wan series shows a clear division of labor — "expert small models for quality control, MLLMs for annotation and evaluation" — and there is no public evidence yet that its data funnel has shifted to large-model semantic judgment.
[The discriminators in Wan 2.1's data funnel are all dedicated/shallow models] A lightweight OCR detector, the LAION-5B aesthetic classifier, an in-house NSFW safety model, a watermark/logo detector, heuristic black-border detection, an overexposure expert classifier, a synthetic-image expert classifier, an in-house blur-scoring model, a manually annotated 1–5 scale visual-quality expert-assessment model, and a motion-quality grader.
[Three places MLLMs appear, none of them quality control]
1) Annotation: an in-house LLaVA-style dense-caption model (ViT + a two-layer MLP + a Qwen LLM) re-annotates the entire corpus; the text branch uses Qwen2-VL to turn OCR results into natural-language descriptions; V2A uses Qwen2-Audio to generate audio captions; Wan2.2-S2V uses QwenVL2.5-72B for annotation.
2) Verification during data construction: an LLM extracts (category, count) pairs from text, and Grounding DINO then actually counts them, retaining a sample only if the two agree — the step in the Wan series that comes closest to "model as judge," a weak form of quality control via cross-checking.
3) Evaluation: Wan-Bench makes extensive use of Qwen2-VL to judge complex tasks such as physical plausibility, motion smoothness, stylization, object/count/spatial relationships, complex camera moves, and action-instruction compliance, while simple tasks (such as object detection) still use traditional detectors. This shows the team already possesses the full engineering capability to use an MLLM for semantic judgment — it simply has not been applied to training-data filtering in the public materials.
Whether 2.5/2.6/2.7 have introduced VLM-based quality control is unknown; given the demands that capabilities such as "role-play" and "multi-shot consistency" place on semantic-level image-text/audio-video matching, it is quite plausible that such quality control has been introduced, but there is no evidence for this. [Uncertain]

### [Audio-video generation benchmark collection](../models/av_benchmarks.md)

This is the field of greatest training-side reference value among the five benchmarks in this survey; together the five happen to form a complete chain of evidence for the 2026 trend of "large model as quality judge":

[AV-SyncBench — empirical evidence of Gemini as a front-end quality judge] Explicitly uses Gemini 3 Flash as the first automated filtering pass, tasked with removing two categories of samples — "off-screen sound sources" and "clearly mismatched audio-visual content" — before handing the remainder to human review. This is one of the rare examples in the public literature of directly embedding a commercial closed-source multimodal large model into the first-stage filter of a data pipeline; its cost-effectiveness trade-off (using a lightweight Flash-tier model for high-recall coarse screening, then human effort for high-precision fine screening) is directly transferable to large-scale training-data cleaning.

[AVBench — a dedicated trained evaluator replacing a general-purpose large model] Takes a different route: rather than a general-purpose MLLM scoring directly, it trains a dedicated evaluator via preference learning. The VT and AV dimensions fine-tune only the LLM portion of Qwen2.5-Omni (7B); the AT dimension fine-tunes both the LLM and connector layers of Qwen2-Audio (7B). The training constraint is cleverly designed: the model outputs only a single token (Yes/No), and a continuous score is obtained by normalizing the token-probability ratio, turning a discrete judgment into a continuous, differentiable signal that can be used directly for data filtering and RLHF reward. This solves the core pain point of LLM-as-judge scoring — discreteness, high variance, and non-differentiability — and is the technical detail with the most reuse value on the training side.

[VABench — general-purpose MLLM for semantic-level evaluation] Module 2 uses Qwen2.5-Omni 7B to handle 5 macro-level scores (Alignment / Artistry / Expressiveness / Audio Realism / Visual Realism, on a 1–5 scale) and 2 micro-level QA tasks (3–7 audio QAs and 3–7 visual QAs per sample). The "macro scoring + micro QA, two-layer" structure is worth borrowing: fine-grained QA decomposes a fuzzy overall score into verifiable factual judgments, significantly reducing the subjective drift of MLLM scoring.

[Omni-Judge — a systematic mapping of capability boundaries] The most important source of negative conclusions. Using Qwen3-Omni (30B total parameters / 3B active) as its subject, comparing the instruct version against a reasoning-enhanced version, it performs correlation analysis against 6 PhD-student annotators across 9 dimensions. The conclusions divide clearly: semantic-type dimensions are usable — audio-text alignment reaches τ_b=0.292 / ρ=0.345 on the Sora 2 subset, and audio-video-text three-modal consistency reaches 0.139/0.151; perceptual-type dimensions are not usable — video quality reaches only τ_b≈0.020, and audio-video synchronization only 0.142, which the authors attribute to insufficient temporal resolution in Omni-LLMs. Direct guidance for training-data pipelines: an Omni-LLM is suited to "semantic matching/mismatch removal," but temporal-synchronization and image-quality judgment must be handed to dedicated models such as Synchformer and DOVER++. The paper also demonstrates an application of using Omni-Judge's interpretable feedback for "feedback-driven correction" of a generative model (correcting generated frames based on identified errors), but does not advocate using it for training-data filtering.

[PhyAVBench — LLM involvement at both the knowledge-construction and initial-screening ends] The LLM is used for physical-knowledge brainstorming, taxonomy generation, and prompt-template generation, as well as initial screening of semantic ambiguity and confounding factors at the quality-control stage; but all LLM output is reviewed and revised by human experts, forming a strict division of labor: "LLM for efficiency + expert gatekeeping."

Overall judgment: the 2026 consensus is a three-tier division of labor — "large models judge semantics, dedicated models judge perception, and humans judge physics and the final boundary" — rather than using a large model to do everything single-handedly.

### [Video captioning-model ecosystem](../models/caption_models.md)

This is the most salient trend in this ecosystem during 2025–2026, and the captioner ecosystem simultaneously plays a dual role — "being judged" and "judging others":
[Trend one: LLM/VLM as caption-quality judge (shallow scorer → large-model semantic judgment)]
· AVoCaDO: GPT-4.1 scores Gemini-synthesized captions on synthesis completeness from 1–5, retaining only ≥4; in the GRPO stage, GPT-4.1 is further used to judge five-dimension checklist coverage as reward ℛ_C.
· AVSCap: automated verification performs tag-retention checking + semantic-consistency checking; GRPO's hybrid reward includes an audio-visual consistency term.
· MOVA: GPT-OSS-120B (a 120B open-source model) handles caption fusion + cross-modal consistency checking — placing the largest model at the fusion/adjudication step rather than the perception step is a typical cost-effectiveness trade-off.
· AuroraCap proposes VDCScore: using a divide-and-conquer strategy to convert long-caption evaluation into multiple short QA pairs judged by an LLM — representing caption evaluation's migration from n-gram metrics like CIDEr/BLEU toward LLM semantic judgment.
· Omni-Cloze takes the opposite approach: using 2,000 clips / 70,000 fine-grained cloze questions to sidestep LLM-judge noise — a form of skepticism about and correction for LLM-as-judge reliability.
[Trend two: captioner as data quality judge]
· InstructAV2AV: Qwen3-Omni both generates editing instructions and performs the five-dimensional verification scoring — the paper itself acknowledges "the shared origin of generation and acceptance is a methodological hazard of this pipeline," a rare instance of self-criticism in this ecosystem.
· Vidu S1 uses an omni model for discrimination at the quality-filtering stage (details undisclosed).
· Foley-Omni uses Gemini 2.5 Pro for annotation, then uses Bandit (cinematic audio source separation) for acoustic post-verification to correct visual hallucination, with a threshold of −35 dB — the combination "large-model annotation + signal-level verification" is an effective paradigm for suppressing multimodal hallucination.
[Methodological hazards common across the ecosystem] (1) Same-source benchmark and model: AVSCap is both the author of AVSCapBench and its top scorer (60.44), carrying overfitting risk, so its score needs cross-validation with third-party benchmarks (UGC-VideoCap, Omni-Cloze); (2) teacher as judge: most works use the Gemini/GPT family as both teacher and judge, biasing scores toward their own output style; (3) a ceiling on human agreement: Panda-70M reports a human-to-human caption preference agreement rate of only 44.9%, meaning the "accuracy" ceiling of an LLM-judge is itself fuzzy.

### [Geometric/structured annotation dataset collection](../models/geometric_datasets.md)

This batch of datasets fully embodies the 2026 trend of "large model as quality judge." SceneScribe-1M is the most typical: it uses Qwen2.5-VL-72B directly as the sole content-quality judge, making a single-pass semantic judgment across six dimensions (motion intensity, watermarks, camera distortion, lighting artifacts, etc.), replacing the traditional combination of shallow aesthetic/sharpness scorers. SpatialVID adopts a hybrid strategy: shallow scorers (CLIP+MLP aesthetics, brightness statistics, PaddleOCR, VMAF) do coarse screening, reserving large-model compute for the annotation side — Gemini-2.0-Flash performs 1fps visual parsing, and Qwen3-30B-A3B refines and corrects motion-direction descriptions using camera-pose priors, essentially a reverse quality-control step of "using geometric ground truth to check the VLM's output," which can correct the left/right or push/pull direction hallucinations common in VLMs. Action100M pushes LLM-based quality control to the extreme: GPT-OSS-120B performs evidence aggregation and structured extraction over captions from multiple sources, and runs three rounds of Self-Refine self-correction — effectively using a reasoning model as an annotation-consistency auditor. WildWorld uses a VLM as the judge on the evaluation side: WildBench's Action Following metric is determined by a VLM judging whether a generated clip matches the ground-truth clip (a 0/1 score), reaching 85% agreement with human evaluation — an explicit validity check of "model as judge" on a geometric dataset.

### [Post-training data strategies for video generation](../models/post_training_data.md)

This is the core of this topic — in the post-training stage, "model as judge" is no longer just a data filter but directly becomes the training signal (a reward model), the most important paradigm shift of 2025–2026.
[The anchor paper's reward-model system (the relatively well-disclosed part)] Following the HPSv3 training paradigm, it uses Qwen3.5 (citation pointing to the Qwen3-VL technical report, arXiv:2511.21631) as the backbone to extract features from images and text, with an MLP outputting a scalar score; for training-image pairs (x1,x2), text c, and human-preference annotation (y1,y2), it computes r1 and r2, using an "uncertainty-aware ranking loss"; training has two stages — Stage 1 uses "data-aware orthogonal gradient projection" to incorporate the diverse aesthetic preferences from HPDv3++ while preserving the original human-preference knowledge already encoded in HPSv3; Stage 2 further leverages "unlabeled data produced by models of different capability tiers and different RL iterations." The end result is four reward metrics covering video aesthetics, text-video alignment, image aesthetics, and text-image alignment.
[Engineering difficulty of multi-reward fusion (explicitly pointed out by the paper)] Different reward signals vary in granularity, scale, and optimization tendency: emphasizing text-video alignment improves semantic fidelity but sometimes harms visual naturalness; over-prioritizing motion quality or video aesthetics leads to visually pleasing but semantically weaker footage. This requires carefully designed reward-aggregation strategies and weight-coefficient tuning, so the optimization process remains stable and is not dominated by any single objective. The paper states that it ultimately treats "overall visual quality" as the primary objective when making trade-offs.
[Cross-work reward-model inventory]
· VideoAlign / VideoReward (arXiv:2501.13918, Kuaishou Kling + CUHK): VLM-based, three dimensions (visual quality VQ / motion quality MQ / text alignment TA), trained on 16,000 prompts, 108,000 videos generated by 12 T2V models, and 182,000 annotated triplets, using a Bradley-Terry model (BTT) with tie support. Widely reused as the base by Cosmos-Predict 2.5, LongCat-Video, and JavisDiT++, making it the de facto standard for open-source video reward models;
· HPSv3 (ICCV 2025): HPDv3 contains 1.08 million text-image pairs and 1.17 million paired-comparison annotations, covering SOTA generative models and low-to-high-quality real images;
· LongCat-Video's three rewards: VQ = HPSv3-general (average over all frames) + HPSv3-percentile (taking the top 30% highest-scoring frames, combined with the video caption in computation, used to suppress dilution of the overall judgment by a minority of low-quality frames), two paths in parallel; both MQ and TA are fine-tuned from the VideoAlign base on internal annotation data;
· Seedance 1.0's three dedicated RMs: a Foundational RM (VLM architecture, for image-text alignment and structural stability), a Motion RM (suppressing artifacts, enhancing motion magnitude and liveliness), and an Aesthetic RM (image-space input, with the data source switched to video keyframes), with iterative multi-round learning;
· SkyReels-V2: a Bradley-Terry model trained on 30,000 sample pairs;
· JavisDiT++'s six-model division of labor: AudioBox-Aesthetics (audio quality), ImageBind (text-audio alignment / text-video alignment / cross-modal similarity), VideoAlign (video quality), Synchformer (temporal synchronization);
· Other cited RM lineages: ImageReward, Pick-a-Pic, VideoScore, VisionReward (AAAI 2026), Unified Reward Model, RewardDance.
[Counterexamples and cautionary notes] Step-Video-T2V explicitly does not train a reward model, and in its outlook section points out a limitation of current DPO — once the model can easily distinguish positive from negative samples, gains saturate — proposing that a future RM could dynamically score newly generated samples to keep providing an effective gradient. The verifier-hacking problem revealed by ITS-JAVG foreshadows that rashly using an automated verifier to construct preference data for RLHF is quite likely to train a model that merely learns to please the judge. Cosmos-Predict 2.5's response is to use diffusion loss on the fine-tuning dataset as a regularizer to mitigate reward hacking; LongCat-Video and the anchor paper's response is to use multiple rewards together to keep each other in check.

### [Combined survey of mainstream video pretraining datasets: Panda-70M, InternVid, Koala…](../models/pretraining_datasets.md)

**This is the axis along which these seven datasets best illustrate the "2024→2025 technology-generation shift," and three clear generations can be laid out**:
[Generation 0 · no model judge at all (Panda-70M, 2024.02)]
Filtering is done entirely by PySceneDetect and ImageBind feature distance, with no semantic-level quality judgment at all. The closest thing is the **matching_score output by the UMT retrieval model** (>0.43 counted as strong image-text relevance, with 89.6% of the full set meeting this), but its function is **to select the best caption among 8 candidate captions**, not to give a sample a quality score for removal — **the VLM here is only an annotator (8 teachers), not a judge**.
[Generation 1 · shallow dedicated scorer + threshold (InternVid 2023, OpenVid-1M 2024, MiraData 2024)]
The LAION aesthetic predictor, DOVER, RAFT/UniMatch optical flow, CLIP adjacent-frame similarity — all lightweight dedicated models that output a scalar, cut by threshold/percentile. MiraData's only use of a large model is in the **stitching step's Qwen-VL-Chat + LLaVA voting** (judging whether adjacent clips are the same scene) — using a VLM for **structural judgment** rather than quality judgment, a transitional form.
[Generation 2 · MLLM performing binary semantic adjudication directly (LVD-2M 2024.10, the earliest)]
**LVD-2M is the earliest and most thorough case in this survey of using an MLLM to replace scorers outright**: **PLLaVA-7B takes 8 uniformly sampled frames and directly outputs an uppercase GOOD/BAD**, and "only videos judged good on all defined metrics are retained." Two prompt excerpts are worth quoting — (1) content variation: "If the background, setting, and characters are in static states, the video lacks content variation… You must provide a capitalized either 'BAD' or 'GOOD' answer."; (2) visual diversity + text: "A visually diverse video should have rich content that is visually appealing. If the video is only some person talking to the camera with a static background, it is not diverse. And a video with only texts instead of objects is not diverse… Determine if text overlays dominate the video in a way that detracts from the visual experience." **In doing so it completely replaces both the aesthetics scorer and the OCR model**. The motivation is stated clearly: optical flow cannot catch problems like "handheld shake produces a high optical-flow value despite no meaningful motion." The authors also honestly list a limitation: "current MLLMs are not guaranteed to have video-quality assessment capability, and some models struggle to follow the relevant instructions," and call for building a benchmark for MLLM video-quality assessment.
[Generation 2.5 · large-model attribute-checklist-style adjudication (UltraVideo 2025.06, the largest in scale)]
**Qwen2.5-VL-72B makes a binary judgment on 16 low-quality attributes, and a hit on any one leads to removal** (transition effects, watermark, split-screen, screen recording, picture-in-picture, etc.). It also uses **VideoCLIP-XL-v2** to compute similarity between the video and its summary caption, **removing anything below 0.2** — the only explicit image-text-consistency removal step among the seven (Panda-70M's matching_score is used only for caption selection, not removal). A 72B-scale judge combined with attribute-checklist-style structured adjudication is the typical 2025 form.
[Generation 3 · feeding sub-metrics into a learned network that outputs a unified score (Koala-36M, a different route)]
Koala-36M does not take the route of "a bigger VLM" but rather "better fusion." Its **Training Suitability Assessment (TSA) network** has three branches: a dynamic branch (**3D Swin Transformer**), a static branch (**ConvNeXt**), and a label branch (feeding **traditional sub-metrics** such as sharpness/aesthetics/motion as additional input rather than as thresholds), with a **Weight Cross-Gating Block (WCGB)** injecting the label-branch features into the dynamic and static branches via learnable fusion weights. **VTSS is the scalar directly output by this network after consuming "pixels + sub-metrics"; it is not a regression over, or weighted sum of, sub-scores** — rejecting the weighted-threshold paradigm is precisely the paper's entire argument. Training labels come from **200,000 videos × 8 experts × 1–5 scale**, with two rounds of bias correction applied: personal-preference bias is corrected via **per-expert z-score normalization** then restored using the global mean and variance, and annotation-fluctuation bias is handled by taking the **mean across 8 annotators**. Ablation (PLCC/SRCC/KRCC/RMSE): dynamic branch alone 0.8684/0.8580/0.7027/0.4644 → + static 0.8730 → + label branch 0.8953 → + WCGB **0.8974/0.8868/0.7406/0.4099**; compared with FastVQA 0.8684 and DOVER 0.8554. **The choice of the VTSS threshold of 2.5 is also simple**: the overall distribution is bimodal (approximately two Gaussians), and the value is taken directly at the valley between the two peaks, 2.5 (in the released CSV, the minimum VTSS value happens to be 2.50, an independent corroboration).
**⚠️ Reproducibility warning**: The VTSS checkpoint Koala-36M **released is not the one used to actually build the data** — test.yml shows its configuration is `DiViDeAddEvaluator + swin_tiny_grpb + fragments-only + divide_head:false`, i.e., the pure fragments branch of FAST-VQA/DOVER, whose performance exactly matches the "dynamic branch only" row in the ablation table (PLCC 0.8684); **the ConvNeXt static branch, the label branch, and the WCGB are entirely absent from the repository**. Anyone reproducing the filtering from the open-source code will not obtain the paper's VTSS.
**Conclusion**: datasets from the first half of 2024 (Panda-70M, InternVid, OpenVid-1M, MiraData) still remained at "shallow scorer + threshold"; LVD-2M and Koala-36M, both from October 2024, made breakthroughs in two different directions — "switching the judge" and "switching the fusion method," respectively; and UltraVideo in 2025 pushed the judge's scale to 72B and structured it as an attribute checklist — **the evolution of "VLM-as-judge" across these seven datasets forms a very clear timeline**.
## Safety and Compliance Filtering (NSFW, Copyright, Face/Privacy)

`safety_filtering` · Detail level: brief

### [Allegro](../models/Allegro.md) ⚠️

The paper discloses no NSFW / violence / copyright / face-privacy safety compliance filtering step, and there is no corresponding metric in the Table 1 threshold table either. The team only gestures at compliant use at the level of the model license (Apache 2.0) and terms of use — the data-side safety filtering strategy is entirely absent from the public description. Given that the corpus comes from public datasets such as WebVid / HD-VILA, it may partially inherit filtering from the original datasets, but the paper does not state this. [Uncertain]

### [Apollo](../models/Apollo.md) ⚠️

The paper's disclosure of safety and compliance filtering amounts to a single word: "safety" is listed as the fourth dimension of video quality modeling (alongside dynamic quality, static quality, and content naturalness), positioned at the first filtering gate. There is no elaboration whatsoever — no explanation of the NSFW detection method or model, no mention of copyright filtering, no mention of face recognition/privacy protection/celebrity-likeness filtering, no safety taxonomy given, and no model-card-level safety statement or usage restriction. As a platform operator like Kuaishou that must bear content-moderation responsibility, a mature internal safety review system almost certainly exists, but the paper chooses not to disclose it. [Uncertain]

### [CineDance / CineDance-1M](../models/CineDance.md) ⚠️

Safety and compliance filtering is the weakest-disclosed dimension of this work:
【Paper level】The three-stage pipeline contains no description whatsoever of NSFW detection, violent-content filtering, face-privacy handling, or copyright-detection steps. The manually audited artifact checklist also consists entirely of image-quality issues (subtitles, watermarks, logos, black bars, screen recordings, etc.), with no safety category included.
【Substitute constraints at the release level】① Adopts gated access with human review of applicants; ② the license is limited to CC-BY-NC-SA-4.0 non-commercial use; ③ the dataset card explicitly disclaims — "automated and manual curation cannot guarantee removal of every low-quality, sensitive, or otherwise unsuitable sample," requiring users to perform their own quality checks according to their application scenario.
【Judgment】Safety responsibility is effectively shifted downstream to users via gating plus disclaimers, rather than being addressed within the pipeline. Given that the material is film/TV content (which may contain violence, adult plot elements), this is a gap that needs significant supplementing for practical use. [Uncertain]

### [CogVideoX](../models/CogVideoX.md) ⚠️

[Uncertain] Neither the paper nor the open-source repository describes an NSFW, violence, copyright, or face/privacy filtering workflow or tooling at the training-data level, nor does it disclose any safety model or watermarking scheme on the generation side. The only indirect clues are: the negative tags Text Dominated and Noisy Screenshots incidentally remove some material containing sensitive text or screen content; the open-source license (Apache 2.0 / Zhipu open-source license) contains standard compliant-use clauses; and the Qingying (CogVideo) product, as a publicly available service within China, must comply with the "Interim Measures for the Management of Generative Artificial Intelligence Services" and complete algorithm registration, but the technical details are not disclosed.

### [Cosmos-Predict2.5](../models/Cosmos-Predict25.md) ⚠️

[Uncertain]. At the paper level there is almost no description of safety and compliance filtering. The only phrase that touches on it is that the filtering objectives include "content that is unsuitable for training," with no elaboration of its definition, detection methods, or scale — presumably covering NSFW/violence etc., but there is no textual basis to confirm this. The paper nowhere mentions an NSFW classifier, violence detection, face privacy/de-identification, copyright filtering, or any other specific mechanism.
Indirectly related is the NVIDIA Cosmos platform-level Cosmos-Guardrail guardrail system (described in the prior-generation Cosmos-Predict1 and mentioned in the GitHub repository documentation as "robust guardrails" and "improved guardrails"), but that operates on the inference side — filtering input prompts and blocking output content — rather than in the training-data cleaning pipeline, and this paper does not elaborate on it.
Worth noting: this pipeline includes one content-trimming step that is similarly directional but unrelated to safety — removing non-physically-real content such as games/animation/cartoons, motivated by distribution alignment rather than safety.

### [Data-Juicer 2.0](../models/Data-Juicer.md) ⚠️

【Visual safety】video_nsfw_filter is the only video-side safety operator, "retaining samples whose NSFW score falls within a specified range," underpinned by the Falconsai/nsfw_image_detection image-classification model (scoring extracted frames one by one and then aggregating). In the T2V case study, this operator was validated as effective by the Probe stage and included in the final recipe; the official description states it "uses the VideoNSFWFilter operator to ensure high quality" — i.e., in that case study it simultaneously serves both a safety and a quality role (high-NSFW-score samples tend to also be low-quality, unnatural content).
【Privacy protection】video_face_blur_mapper detects and blurs faces in videos — a relatively rare built-in privacy de-identification operator at the data-processing-framework level. Combined with video_human_tracks_extraction_mapper (face/body track extraction), track-level continuous de-identification can be achieved. On the text side there are additionally operators for de-identifying sensitive information (phone numbers, email addresses, ID numbers, etc.).
【Copyright-related】video_watermark_filter and video_remove_watermark_mapper can identify and process watermarks, indirectly relating to copyright markers, but this does not constitute copyright-ownership determination capability.
[Uncertain] Missing pieces include: no copyright-content recognition/fingerprint-matching capability; no support for provenance authentication standards such as C2PA; no detection operators for fine-grained safety categories such as violence, hate, or minors (only binary NSFW); and no disclosure of the safety review system layered on top when used internally at Alibaba. As an open-source framework, DJ provides "assemblable safety parts" — a complete compliance system must be built by the user.

### [Foley-Omni](../models/Foley-Omni.md) ⚠️

[Uncertain] The pipeline describes no safety and compliance filtering step whatsoever: no NSFW detection, no copyright-content recognition, no face-privacy handling or de-identification measures.
The paper only acknowledges risk at the ethics-statement level: it explicitly notes the possibility of the model being misused to produce deepfakes (given that VisualTTS accounts for 40% of the training data and the model can generate matching speech from a speaker's video, this risk is indeed substantive), and states that the data was collected under "appropriate usage agreements." But these are after-the-fact statements rather than technical measures in the data-processing pipeline.
No mechanism for watermarking generated content, C2PA marking, or authorization confirmation of speaker identity is seen. Given that datasets such as GRID, LRS2, SpeakerVid, and TalkVid all contain large amounts of identifiable faces and voiceprints, and that the model directly learns a "face→voice" mapping, the absence of privacy-dimension handling is a rather conspicuous compliance shortfall in this work.

### [Goku](../models/Goku.md) ⚠️

[Uncertain]. The paper nowhere mentions any safety-compliance step such as NSFW/pornographic-or-violent-content filtering, copyright filtering, face and privacy protection, sensitive-person recognition, or harmful-content classifiers, nor does it have a Responsible AI / model-card section. Given that the publisher is ByteDance, an internal safety review process almost certainly exists in the actual production pipeline (ByteDance has a mature content-safety middle platform), but there is zero disclosure at the paper level. The only indirectly related item is OCR text filtering (removing subtitles/pasted advertisements), but its motivation is image quality and generation quality rather than safety compliance.

### [Hailuo / MiniMax Video](../models/Hailuo.md) ⚠️

[Uncertain] (no disclosure on the data side, only evidence on the inference side). NSFW filtering, copyright filtering, and face/privacy handling on the training-data side are all undisclosed. What can be observed is only the inference-side content safety policy: Hailuo AI and the open-platform API have explicit real-time review and interception for prompts and generated results involving politically sensitive content, pornography, violence, and public-figure likenesses (a compliance requirement under China's regulatory environment, the "Interim Measures for the Management of Generative Artificial Intelligence Services"); the Hailuo AI web front end displays the notice "content generated by AI — please use this feature lawfully and amicably." The existence of strict review on the inference side can indirectly indicate that the company possesses a mature content-safety model, but one cannot directly infer from this how it is applied — or with what intensity — at the training-data cleaning stage.

### [HunyuanVideo-Foley](../models/HunyuanVideo-Foley.md) ⚠️

Neither the paper nor the model card describes any safety and compliance filtering aimed at the training data: no NSFW content detection, no violent/gory content filtering, no copyright-content recognition, no face/voice privacy protection measures, and no description of excluded sensitive-content categories.
【Substitute risk control on the model side】Safety responsibility is shifted to the license and distribution layer rather than the data layer: it adopts the tencent-hunyuan-community community license (which includes usage-restriction clauses), sets extra_gated_eu_disallowed: true on HuggingFace to exclude the EU region, and requires a gated process to obtain the model. This is a posture of "substituting legal and distribution controls for technical filtering."
【Risk-surface assessment】Compared with video-generation or voice-cloning models, the misuse risk of Foley sound-effect generation is overall lower — it generates generic acoustic events rather than an identifiable voice or likeness, so the risk of deepfakes and identity impersonation is small. The main residual risks are: (1) if the training data does not remove violent/gory scene audio, the model could be used to reinforce the immersiveness of violent content; (2) when cascaded with a video-generation model, realistic sound effects would significantly increase the credibility of a forged video, constituting a capability gain for the overall forgery chain; (3) there is no output-side audio watermark, so generated audio is not traceable. The paper discusses none of these points.
【Degree of disclosure gap】Even accounting for the lower task risk, an open-source model released by a large commercial company that says nothing at all about data safety filtering still constitutes insufficient disclosure. [Uncertain]

### [HunyuanVideo](../models/HunyuanVideo.md) ⚠️

Disclosure is weak. The original version mentions only in a single sentence that a YOLOX-like vision model is used to remove "watermarks, borders, logos, and certain sensitive information," without elaborating on what counts as sensitive information, and without mentioning an NSFW classifier, face/privacy protection, or copyright filtering. The data section of the 1.5 technical report does not touch on safety filtering at all. As a product released within China, the actual production system almost certainly has content safety review (the model card and license contain standard compliant-use clauses, and the open-source repository requires compliance with local laws and regulations), but the safety filtering method on the training-data side has zero disclosure. [Uncertain]

### [InstructAV2AV](../models/InstructAV2AV.md) ⚠️

Safety filtering exists explicitly in this pipeline as one of five dimensions of automated verification — "(v) safety to filter inappropriate content," i.e., Qwen3-Omni judges whether the synthesized result contains inappropriate content, and it is discarded if it fails. Compared with the majority of the surveyed works, which omit safety filtering entirely, this work at least writes it into the pipeline and treats it as one of the hard pass conditions — a moderately above-average practice.
【Limitations and gaps】
  · Coarse granularity: only a single, general "safety" dimension, with no breakdown into sub-categories such as NSFW, violence, hateful content, copyright markers, or face privacy, and no statement of what safety criteria or policy is used.
  · No dedicated safety model observed: no mention of an NSFW classifier, copyright-content recognition, face detection/de-identification, or other specialized tools — everything relies on the judgment of a general-purpose MLLM.
  · Applied too late in the pipeline: the safety check operates only on the synthesized output; there is no upfront safety screening of the input material (content crawled from YouTube).
【The gap most worth flagging — unaddressed identity-misuse risk】The dataset includes three dedicated subsets — clone_id (identity cloning), clone_voice (voice cloning), and clone_id_voice (simultaneous identity + voice cloning) — and the model's direct capability is "face-swapping, voice-swapping, and line-rewriting on real people's videos while preserving lip sync" — which is precisely the technical definition of deepfakes. However:
  · The paper does not mention authorization or informed consent for speakers' likeness/voice rights;
  · No watermark, C2PA content credential, or other detectable marking is applied to generated content;
  · The paper's limitations discussion focuses entirely on technical shortcomings (physical realism, lighting consistency, 3D spatial consistency, object permanence) and contains no ethics statement or misuse-risk discussion — in contrast, Foley-Omni at least explicitly acknowledges deepfake risk, so this work is even more lacking on this point;
  · The dataset is fully open for download under the MIT license (including real people's footage), the weights are also fully open, and fine-tuning checkpoints specifically for identity/voice cloning are provided, which in practice lowers the barrier to misuse.
Taken together, safety is the dimension in this work that is most mismatched with its degree of technical openness.

### [2026 Other Joint Audio-Video Generation Works](../models/JAVG_2026_misc.md) ⚠️

[Uncertain] — none of the seven works describe safety-compliance steps such as NSFW filtering, violent/gory content recognition, copyright-content detection, face-privacy protection, or celebrity-likeness removal.
The only indirectly related content:
(1) It is unknown whether ALIVE's 13-dimensional low-quality classifier includes a safety dimension [Uncertain]; its OCR watermark detection can be viewed as an indirect recognition of copyright markers, but the purpose is image quality rather than compliance.
(2) NAVA's PaddleOCR subtitle erasure is likewise quality-oriented.
(3) OmniCustom and ALIVE both make heavy use of ArcFace face embeddings for identity matching/verification, which involves biometric processing but with no privacy statement; OmniCustom's evaluation set uses "30 persons who were not included in training data" plus 70 YouTube video clips, without stating any likeness authorization.
(4) Misuse risk of voice cloning: both OmniCustom (reference audio → voice imitation) and NAVA (Timbre-in-Context Conditioning, reference-timbre control) possess voiceprint-cloning capability — a clear deepfake risk point — yet neither paper contains a responsible-use statement, a watermarking scheme, or a discussion of misuse safeguards [Uncertain]. Although NAVA open-sources its weights and training code under Apache 2.0, no usage-restriction clause is found in the repository either [Uncertain].
(5) StreamChar's real-time streaming digital human likewise carries real-time forgery potential, with no related discussion.
【Judgment】Safety compliance is the most uniformly missing dimension across this batch of seven works — none of the seven has substantive disclosure, and three of them (OmniCustom, NAVA, StreamChar) happen to involve high-risk capabilities such as identity and voiceprint cloning. By contrast, contemporaneous closed-source industrial models such as Sora 2 and Veo 3 both have clear safety sections — this gap on the open-source/academic side is worth noting.

### [Joint Audio-Video Generation Baseline Collection](../models/JavisDiT_baselines.md) ⚠️

Overall disclosure is extremely sparse; only JavisDiT has a clear action item:
【JavisDiT / JavisDiT++ (the only one)】During construction of JavisBench, all content sourced from YouTube underwent "strict manual legal and ethical verification" — this is the only clearly defined safety-compliance step in this collection, and it is manual rather than automated. But this review targeted the evaluation set (10,140 clips); whether the 330K-clip TAVGBench training-side data underwent safety filtering is not stated [Uncertain]; nor is any automated mechanism such as an NSFW detector, violent-content filter, or face-privacy protection described.
【MM-Diffusion】No description of NSFW/copyright/privacy filtering [Uncertain]. Indirect compliance awareness is reflected in data selection: one reason for choosing AIST++ is that its soundtrack consists of copyright-cleared songs.
【AV-DiT】No safety filtering of any kind described [Uncertain].
【Harmony】No safety filtering of any kind described [Uncertain]. Its data contains large amounts of real-person video (OpenHumanVid, SpeakerVid, self-collected clips); face and voiceprint privacy risk is real in practice, but the paper makes no related statement.
【UniAVGen】No safety filtering of any kind described [Uncertain]. This is the highest-risk item — the training data is an "internally collected real-human audio-video dataset," and the model has audio-driven face-animation capability, a deepfake-sensitive capability — yet the paper has neither a data-side privacy statement nor any model-side misuse safeguard or usage-restriction statement.
【Common gaps】None of the five provides a model-card-level safety section, discusses deepfake-misuse prevention, or mentions removal of celebrity likenesses or handling of children's content.

### [Kling 3.0 / Kling Video 3.0 Omni](../models/Kling_30_Omni.md) ⚠️

The Kling-Omni base filtering tier explicitly includes "content safety / NSFW filtering." The product side additionally has strict input/output safety review (interception of face/celebrity likenesses, politically or violently sensitive content) and AI-generated-content labeling/watermarking, in line with Chinese regulatory requirements. [Uncertain: the specific strategy and model for face privacy, likeness rights, and copyright filtering on the training-data side]

### [LTX-2](../models/LTX-2.md) ⚠️

Safety and compliance filtering on the training-data side is entirely undisclosed: there is no explanation of NSFW filtering, face/privacy handling, or CSAM screening. The team's compliance strategy is mainly front-loaded at the data-acquisition stage — ensuring copyright cleanliness through a Shutterstock research license and Getty Images licensed procurement, rather than through downstream content classifiers. The "Social Impact" section of the technical report offers only a qualitative statement at the deployment level: acknowledging that synthetic media carries a risk of being used for deceptive content, requiring users to explicitly disclose the synthetic origin and comply with content-safety guidelines, acknowledging that the model will inherit visual and auditory biases present in the training data, and listing bias mitigation, authenticity verification, and traceability as future work. The Limitations section of the model card explicitly warns that the model "may generate inappropriate content or amplify social biases." No description of any automated safety filtering module, classifier, or red-teaming process is found. [Uncertain]

### [LongCat-Video](../models/LongCat-Video.md) ⚠️

[Uncertain]. The technical report does not mention any NSFW filtering, violent/sensitive-content filtering, copyright filtering, or face-privacy protection measures. Given that the model is intended for public open-source release and is published by a major domestic (Chinese) company, a content-safety review step almost certainly exists in actual production, but there is no textual basis in the report to confirm its method or intensity. The only indirectly related item is that in Avatar 1.5, "synthetic content" is listed as a hard exclusion for emotion data, but that is for annotation-reliability reasons rather than safety considerations.

### [MOVA](../models/MOVA.md) ⚠️

The paper does not touch on safety and compliance filtering at all: there is no description of NSFW detection, copyright filtering, or face/privacy protection measures, and no model-card-level safety statement or usage restriction. This stands in contrast to its Apache-2.0 fully open commercial-use license, and is a clear gap in MOVA's disclosure system (the exact inverse of Sora 2, which is "thorough on safety disclosure, blank on data disclosure": MOVA is "thorough on data-method disclosure, blank on safety disclosure"). [Uncertain]

### [Merged entry: ① Mochi 1](../models/Mochi_MAGI_Motif.md) ⚠️

① Mochi 1: The model card states that "NSFW content filtering has been applied," but does not explain the method, tools, or thresholds; it also explicitly notes that "Genmo's video models will reflect biases and preconceptions present in their training data," and recommends that organizations implement additional safety protocols before commercial deployment. Data-side filtering for copyright and face/privacy is not mentioned. [Partially uncertain: the specific method of NSFW filtering]
② MAGI-1: Section 3, DATA, sets up no safety-and-compliance-filtering step at all, with no mention of NSFW, violence, copyright, or face-privacy filtering. The only face-related filter is Corner Face Detection, but its motivation is to remove the fixed corner-anchorman picture-in-picture pattern common in commentary-style videos — a visual-pattern issue rather than privacy protection. [Uncertain]
③ Motif-Video 2B: Safety filtering is explicitly front-loaded into the first stage of sanitation, and this is the only one in the group with a dual-signal mechanism: first, an OCR pre-screen inherited from the older crawling pipeline flags high-confidence watermarks/station logos/burned-in subtitles via on-frame text detection; surviving clips are then passed to a VLM that outputs structured labels such as nsfw, watermark, and padded, and hitting any one triggers a hard delete. The paper describes the second VLM pass as "a semantically aware safety net on top of OCR." Still, this does not cover copyright review, face/privacy de-identification, minors' content, or other specialized categories, nor does it give the specific model or decision threshold for the NSFW classifier. [Partially uncertain: the specifics of NSFW determination and copyright/privacy handling]

### [Movie Gen](../models/Movie_Gen.md) ⚠️

[Uncertain] The technical report does not describe an NSFW, copyright, or face/privacy filtering workflow or tooling at the training-data level. The paper's statements about safety are concentrated in the "Safety considerations" paragraph of the conclusion: the model was developed for research purposes and requires numerous improvements before deployment; the model may learn unintended cross-modal associations; generative models learn biases present in each modality (e.g., visual biases in the video training data, biases in the prompt language); the research is limited to English text input; and in an actual deployment, a safety model would be connected to reject policy-violating input prompts or generated outputs, in order to prevent misuse. Data-side compliance filtering and generated-content watermarking/provenance marking are not disclosed.

### [NVIDIA NeMo Curator](../models/NeMo_Curator.md) ⚠️

NeMo Curator's safety-filtering capability is severely uneven across modalities:
【Image modality】Provides an NSFW-detection stage (officially listed in the README as one of the core image-side capabilities).
【Video modality】The video-filtering section of the official documentation contains only two categories, motion filtering and aesthetic filtering, and provides no stage for an NSFW classifier, face detection/blurring, personal-information removal, or copyright fingerprint matching. The curation section of the Cosmos WFM paper likewise does not describe data-side safety filtering — its safety capability is concentrated in the inference-side Cosmos Guardrails (input-prompt filtering + output-content classification + face blurring), which is a deployment guardrail rather than training-data cleaning.
【Text modality】Has a quality-filtering and classifier system that can indirectly take on part of the content-safety responsibility.
【Conclusion】When building video training data with this framework, the three compliance lines of NSFW, face privacy, and copyright all need to be implemented by the user separately. [Uncertain]

### [OmniHuman Dataset + OHBench](../models/OmniHuman.md) ⚠️

【Complete absence】The paper does not touch on any safety and compliance filtering: no NSFW/pornographic/violent-content detection, no minors'-content recognition, no copyright filtering, no face-privacy de-identification, no voiceprint-privacy handling, no hate-speech or harmful-speech-content filtering, and no model-card-level usage-restriction statement. There is also no ethics-statement section anywhere in the paper.
【Special risk exposure of this dataset】Compared with other entries, the consequences of this absence are more severe here:
1) The data source is undifferentiated YouTube scraping; although the YouTube platform itself has content moderation, its moderation standard (suitable for public playback) is completely different from the training-data standard (suitable for use in training generative models) — for example, a large amount of adult-adjacent, borderline, or violent gaming-livestream content is compliant on the platform but unsuitable as training data;
2) The released assets include ArcFace face embeddings, 134-point skeletons, SMPL/MANO 3D parameters, speech transcripts, and emotion labels for 80,000 real people — this is a complete kit of material directly usable for identity forgery, with no access review;
3) The content types explicitly include "film/TV," and the copyright status of this portion of content is the most sensitive — even distributing only via URL cannot fully avoid this;
4) Within the 8 content categories, the "education" and "gaming" categories contain a large amount of YouTube content involving children, and there is no mechanism for recognizing minors.
【The only indirect mitigation】Only releasing annotations and URLs, not the video itself, provides a buffer on the copyright dimension (see provenance_licensing for details), but provides no mitigation for content safety or likeness privacy.
【Comparison with the upstream dataset】OpenHumanVid has a download-approval mechanism; OmniHuman has no access control whatsoever. [Uncertain]

### [Open-Sora Series](../models/Open-Sora.md) ⚠️

**Neither project's technical report or documentation describes any NSFW detection, violent-content filtering, face/privacy protection, or copyright-content detection step** — safety filtering is essentially entirely absent, implicitly relying on the fact that upstream public datasets (Panda-70M, LAION, WebVid, etc.) have already undergone their own safety cleaning. The only indirectly related items are: Open-Sora Plan applied "high-quality" filtering when selecting portrait images from LAION-5B, but with no privacy/likeness-rights consideration; both projects' OCR filtering removes dense-text scenes, which incidentally filters out some material bearing copyright notices/station logos, but this is a side effect rather than a design goal. The model weights are released under permissive licenses such as Apache 2.0, without any accompanying output-side safety classifier or watermark. This is the largest compliance gap of the open-source reproduction projects relative to closed-source commercial models (Sora 2's CSAM source-level screening plus multi-tier safety classifiers). [Uncertain]

### [Ovi](../models/Ovi.md) ⚠️

[Uncertain]. Neither the paper nor the repository describes any safety-compliance step such as NSFW filtering, violent/gory-content filtering, copyright-content recognition, face-privacy protection, celebrity-likeness removal, or handling of children's content, nor is there a model-card-level safety section or usage-restriction statement (only an Apache 2.0 license). Given that Character AI, as a company behind a consumer-facing conversational product, typically has an internal content-moderation system, its internal audio-video corpus may have already undergone product-side review, but the paper makes no related statement whatsoever. This is a clear gap in this work's data disclosure.

### [Script-a-Video](../models/Script-a-Video.md) ⚠️

The paper does not touch on safety and compliance filtering at all: no NSFW detection, no violent/sensitive-content filtering, no copyright filtering, no face-privacy protection measures, and no model-card-level safety statement or usage restriction. [Uncertain]
Worth noting is the risk gap: this work's Identity Customization module supports injecting a real person's identity from a reference image and maintaining it consistently across multiple shots, the Event stream records verbatim dialogue and binds it to a speaker, and the Reference stream anchors fine-grained appearance details of a character (clothing, accessories, hairstyle) — this combination of capabilities happens to constitute the technical prerequisites for high-quality deepfakes, yet the paper contains zero discussion of misuse risk. The mitigating factor is that neither the model weights nor the data have been publicly released.

### [Seedance 2.0 and Seedance 1.5 pro](../models/Seedance_20_Seedance_15_pro.md) ⚠️

Seedance 1.0: deploys advanced classifiers to detect and remove pornography, explicit violence, child exploitation, explicit nudity, and other harmful or inappropriate content, in order to ensure ethical compliance and dataset safety. The Seedance 2.0 report states that a structured safety-evaluation framework is implemented across the full lifecycle of model iteration, with ongoing assessment and mitigation of potential risks. The specific mechanisms for face/privacy and copyright fingerprinting are not disclosed. [Uncertain: the specific methods for face and privacy filtering, and copyright detection]

### [SkyReels Series](../models/SkyReels.md) ⚠️

Neither generation's paper sets up a safety-and-compliance-filtering section: no explanation of an NSFW/violent-content classifier, no description of face/privacy handling (de-identification, likeness rights), no copyright-infringement detection, no CSAM screening process, and no red-teaming or Social Impact section either. The only indirectly related items are the content-type-level exclusion (surveillance-footage clips are filtered out as an entire category, which objectively reduces privacy-sensitive material) and the "licensed content" procurement standard. As a commercial product aimed at the Chinese market, the SkyReels product side should have content-safety review and synthetic-content labeling mechanisms that meet regulatory requirements, but the technical report does not touch on this at all, so its data-side implementation cannot be confirmed. [Uncertain]

### [Sora 2](../models/Sora_2.md)

This is the only cleaning step on the training-data side with substantive disclosure. Explicit content: (1) uses "a combination of safety classifiers" to prevent harmful or sensitive content from being used or generated, explicitly naming sexual content involving minors (CSAM); (2) on child safety, adopts "responsibly sourcing datasets to exclude CSAM" — i.e., screening out CSAM at the data source, in cooperation with NCMEC, and performing strong scanning on all input and output (including first-party products as well as API/enterprise third-party use), unless the customer meets strict exemption criteria; (3) has a dedicated CSAM safety stack that reuses system-level mitigations from other products while layering on Sora-specific protections. NSFW/copyright/face-privacy are not separately described in terms of filtering method on the training-data side, and are mainly handled through deployment-side policy: no support for video-to-video, no support for text-to-video of public figures, blocking generation involving real people except for users who have explicitly opted in via cameo authorization, and applying stricter thresholds to uploaded material suspected to involve minors.

### [SpeakerVid-5M](../models/SpeakerVid-5M.md) ⚠️

Neither the paper nor the data card describes any safety-and-compliance-filtering step: no NSFW/violent-content detection, no copyright-content recognition, no face blurring or privacy de-identification, no handling of minors' content, and no filtering of harmful speech content (content review based on ASR transcripts).
【Substitute, passive risk handling】The only compliance measure is after-the-fact and passive: restricting use to non-commercial research and education, stating that copyright belongs to the original authors, providing a takedown policy, and not hosting the original video, only publishing the YouTube ID. This is "shifting responsibility" rather than "active filtering."
【Privacy-sensitivity assessment】The privacy risk of this dataset is actually higher than that of a typical video dataset — it contains 83K unique speaker identities, per-clip ArcFace facial-feature associations, DWpose full-body skeletons, and complete Whisper speech transcripts (i.e., a searchable structure of "who said what, when"). This combination is sufficient to constitute a biometric database usable for identity tracking, yet the paper performs no privacy-impact assessment and does not state whether the consent of the people appearing on camera was obtained (the default answer for publicly available YouTube videos is no).
【Content political sensitivity】The data source explicitly includes the "news and politics" and "debates" categories; ASR transcription would fully preserve the political speech within them, yet no content-review explanation is seen.
【Conclusion】Safety and compliance filtering is the largest gap in this dataset's disclosure system. [Uncertain]

### [Step-Video-T2V](../models/Step-Video-T2V.md) ⚠️

Disclosure is weak, with only a single item: in stage 2 of the pipeline, NSFW scoring uses an LAION-provided binary NSFW detector based on CLIP ViT-L/14 to score sampled frames, filtering pornographic/inappropriate content on that basis. Beyond this, the report does not mention copyright filtering, face/privacy protection (face blurring, likeness-rights handling), violent/gory-content classification, politically-sensitive-content filtering, celebrity-likeness removal, or any other step, nor does it describe safety alignment or output review on the generation side. Although watermark detection can indirectly remove some material bearing obvious copyright marks, its design motivation is image cleanliness rather than compliance.
As a commercial product operating within China (launched on Yuewen, yuewen.cn), the actual production system almost certainly has a complete content-safety review chain, but the technical report has almost zero disclosure on data-side safety filtering. [Uncertain]

### [UniTalking](../models/UniTalking.md) ⚠️

The paper does not touch on safety and compliance filtering at all: no NSFW/violent-content detection, no copyright filtering, no face-privacy protection measures, no minors'-content recognition, no voiceprint-privacy protection, and no model-card-level safety statement or usage restriction.
【The responsibility gap is especially pronounced】Compared with similar work, UniTalking's absence on this dimension is more noteworthy for three reasons:
1) The model's capability itself is inherently a high-risk deepfake capability — given any person's image (TI2AV) + any voice reference (TR2AV) + any text, it can generate a video with audio of that person saying arbitrary content, which is precisely the complete technical path for forging a celebrity speech video;
2) The data side actively constructs voice-cloning training pairs — using IndexTTS2 to perform zero-shot cloning of real people's voices from real videos, batch-generating 6.9 million cloned speech clips for training, a step which itself warrants compliance scrutiny at the level of voiceprint rights, yet the paper contains no discussion of this;
3) The paper's core motivation is "open-source accessibility"; if the weights are indeed released, the above capabilities would spread with no barrier, and the entire paper does not contain a single sentence about misuse prevention, watermarking, content provenance, or usage terms.
【Partial backstop upstream】OpenHumanVid, in order to prevent the dataset from being misused, requires downloaders to submit user information and undergo approval — the upstream data provider has a clear awareness of misuse prevention, whereas the downstream model provider (UniTalking) fails to carry forward this awareness. [Uncertain]

### [UniVerse-1](../models/UniVerse-1.md) ⚠️

The paper does not touch on safety and compliance filtering at all: no NSFW detection, no copyright filtering, no face-privacy protection measures (although RetinaFace is used for face detection, its purpose is to select speaker shots rather than privacy de-identification), and no model-card-level safety statement or usage-restriction description. This stands in clear contrast to its Apache-2.0 fully open commercial-use license and the model's capability to generate lip-synced video for arbitrary faces (which can be used to produce forged speech videos). Given that the model's headline capability is "reference image + text → speaking-person video with audio," its risk of being misused for deepfakes is significantly higher than that of purely sound-effect-type models, yet the paper discusses this not at all. [Uncertain]

### [Unison](../models/Unison.md) ⚠️

The paper does not touch on safety and compliance filtering at all: no NSFW detection, no copyright filtering, no face-privacy protection measures, no harmful-content review, no model-card-level safety statement or usage restriction, and no output-side watermark.
【Risk assessment — the item with the largest risk exposure in this entry】Unison's capability profile is "given a person's image and text, generate a video with audio of that person speaking the specified content," and the paper explicitly pursues word-level lip-sync, achieving an LSE-C of 3.30 and LSE-D of 7.88, approaching LTX-2's level. This is precisely the core capability of deepfakes. At the same time, its training data — HDTF, VFHQ, CelebV-Text — contains large amounts of real celebrity and public-figure faces. Against this backdrop, the paper's zero discussion of misuse risk, zero discussion of likeness rights, and zero measures for output traceability constitute a clear responsibility gap.
【Notably inverted purpose of face detection】The pipeline does indeed use a face detector, but its purpose is to delimit the operating region for SyncNet in order to select qualifying lip-sync samples — the exact opposite of privacy de-identification or face blurring: faces are detected in order to learn faces better.
【Indirect risk mitigation】All the training corpus comes from academic open-source datasets (most restricted to non-commercial research use), and the model has not yet been open-sourced, so the near-term misuse surface is limited. But the paper promises to "release code and model publicly upon acceptance," and once that happens, without accompanying usage terms and safety measures, the risk would become substantive. The paper gives no forward-looking statement on this. [Uncertain]

### [Veo 3 / Veo 3.1](../models/Veo_3_Veo_31.md) ⚠️

One of the relatively thorough official disclosures. Pre-training mitigations include: applying safety filtering to the pretraining data by risk area; filtering captions for unsafe content and personally identifiable information (PII); filtering training videos by compliance and safety metrics; conducting harmful-content analysis and fairness review of demographic representation on the training data; and removing duplicate and conceptually similar videos. Risk areas covered include child sexual abuse and exploitation material (CSAM), hate speech, harassment, misinformation, deepfakes, sexual content, and violence/gore. Post-training mitigations include SynthID watermarking and production-environment output filtering. The safety policy is consistent with Google's cross-product generative-AI framework and the Gemini / Imagen 3 technical reports, and was approved before release by Google DeepMind's Responsibility and Safety Council (RSC). [Uncertain] The specific classifiers, decision thresholds, and proportion of data filtered are not disclosed; face/privacy is covered only at the caption-side PII level, and the video-side face-privacy handling strategy is not stated.

### [Vidu S1](../models/Vidu_S1.md) ⚠️

Tier 4 filtering includes a standalone Content Safety Filter that removes NSFW and other inappropriate content, motivated by preventing the model from learning harmful information. Copyright filtering, face-privacy/likeness-rights handling, and celebrity filtering are not mentioned [Uncertain].

### [Wan 2.5 / 2.6 / 2.7](../models/Wan.md) ⚠️

Disclosure is limited, and concentrated on a single action item on the training-data side and compliance labeling on the inference side.
【Training-data side】
- NSFW: an internal safety-evaluation model computes NSFW scores for "all training data" and systematically filters inappropriate content (a fixed item in the Wan 2.1 basic-dimension stage).
- Copyright/ownership: only through the acquisition-side constraint of "internal copyright sources," plus detection and training-time cropping of watermarks and logos — no downstream copyright classifier is described.
- Face/privacy: no general-purpose privacy de-identification is described. Face processing is concentrated in the construction of the personalization subset (1 FPS face detection, discarding any frame where multiple faces are detected, discarding clips where more than 10% of frames have no detected face, ArcFace inter-frame similarity for identity-consistency screening, face segmentation to remove background, facial-keypoint detection used for canvas alignment) — and small-area faces are deliberately not filtered out, because such videos typically contain full-body people. These are capability-building actions rather than privacy-protection actions.
- No mention of CSAM screening, red-teaming process, or celebrity-likeness restriction (on the contrary, the caption model was specifically trained on a dataset of thousands of identities for recognizing celebrities/landmarks/film-and-TV characters).
【Inference side】the API's watermark parameter (a fixed "AI-Generated" caption in the bottom-right corner); a negative-prompt mechanism; prompt_extend rewriting; and the commercial service is subject to China's Measures for the Management of Generative AI Services and the content-labeling measures.
The safety-filtering strategy for 2.5/2.6/2.7 is entirely undisclosed. [Uncertain]

### [Audio-Video Generation Evaluation Benchmark Collection](../models/av_benchmarks.md) ⚠️

Disclosure is limited:
【VABench】High-quality images on the I2AV track underwent privacy screening during the manual curation and classification stage — the only one of the five that explicitly mentions privacy filtering.
【AV-SyncBench】The manual stage removes semantically ambiguous samples; no mention of NSFW/copyright/face-privacy-specific filtering [Uncertain].
【PhyAVBench】The self-recorded data involves 184 participants appearing on camera; the likeness-rights and informed-consent process is not disclosed [Uncertain]; because the content consists of controlled physical demonstrations, NSFW risk is naturally avoided.
【AVBench】Based on the public dataset OpenHumanVid, inheriting its safety filtering; no additional safety-filtering description of its own [Uncertain].
【Omni-Judge】Generated using commercial model APIs; safety filtering is handled by Sora 2 / Veo 3's built-in content-safety mechanisms.
None of the five mentions the specific tools or thresholds for an NSFW classifier, copyright detection, or face anonymization [Uncertain].

### [Video Caption Model Ecosystem](../models/caption_models.md) ⚠️

[Uncertain] This field is almost entirely blank across the captioner ecosystem, with only indirect evidence:
· NSFW / copyright / face-privacy filtering is generally completed upstream by the generation-side team before reaching the captioner; captioner papers uniformly do not discuss it.
· The only relevant design is LTX-2's captioner principle of "comprehensive yet factual" — describing only what is seen and heard, explicitly prohibiting emotional interpretation. This is both an anti-hallucination measure and, incidentally, a reduction in the risk of the caption making potentially biased judgments about a person's subjective state — it is the closest thing to a public "safety by design" practice in this ecosystem.
· AVoCaDO-SFT contains the TikTok-10M and ShortVideo subsets, which involve real people's UGC content, with no discussion of face/privacy handling.
· Works involving speaker-identity annotation (AVoCaDO's (speaker, content) pairs, LTX-2's speaker/language/accent, CineDance's character anchor-token binding) all perform annotation highly related to generation and personal identity, yet none discuss privacy compliance.
· The safety filtering of generation-side teams (Movie Gen, Veo 3) is recorded under their respective entries and has no direct coupling with captioner selection.

### [Geometric/Structured Annotation Dataset Collection](../models/geometric_datasets.md) ⚠️

SpatialVID has explicit manual safety intervention: during the initial-screening stage, humans review and remove videos whose titles contain inappropriate terms, while also removing "panoramic camera" content incompatible with the pipeline; the license adopts CC-BY-NC-SA 4.0 to restrict commercial use. Action100M's privacy protection comes from upstream — it uses the face-blurred version of HowTo100M (1,199,096 clips), and only publishes annotations, not video, further reducing copyright and privacy exposure. WildWorld naturally carries no privacy risk, since all footage is virtual characters and monsters rendered by a game engine, with no real faces. SceneScribe-1M does not describe an independent NSFW/face/copyright filtering module; its safety relies on cleaning already performed by upstream datasets (HD-VILA, Panda-70M, Koala-36M) [Uncertain].

### [Post-Training Data Strategy for Video Generation](../models/post_training_data.md) ⚠️

The anchor paper's Section 3.1 lists "unsafe outputs" as one of the failure modes to be eliminated during the SFT stage, but gives no specific filtering method, classifier, or data-processing description [Uncertain] (and as noted earlier, this phrasing is suspected to be carried over from LLM post-training text). Section 6, Broader Impact, discusses commercial application value rather than risk mitigation.
【Cross-comparison】Sora 2 is the only entity that elaborates on safety in the post-training dimension, but its content is a safety-alignment evaluation rather than a capability post-training step: through targeted red-teaming, "thousands of adversarial prompts" were collected and categorized by use case and policy area; outputs were generated with a helpful-only version of the video model and then scored, converted into an automated evaluation set that measures the two metrics not_unsafe and not_overrefuse (adult nudity/sexual content not involving a likeness 96.04%/96.20%; involving a likeness 98.40%/97.60%; self-harm 99.70%/94.60%; violence and gore 95.10%/97.00%; prohibited political persuasion 95.52%/98.67%; extremism and hate 96.82%/99.11%). What Veo 3/3.1's official "post-training mitigations" actually refers to is SynthID watermarking and production-environment output filtering, which belong to the deployment side rather than the data side. InstructAV2AV's five-dimensional automated verification via Qwen3-Omni treats "safety" as one of the admission criteria for training samples — a rare instance on the academic side of folding safety into SFT admission.
【Overall judgment】In the public materials, post-training-stage safety work is almost entirely concentrated on "output-side interception + red-team evaluation," rather than "constructing safety preference pairs within the preference data." Using RLHF for safety alignment (a standard practice in the LLM field) has not yet been seen in public practice in the video-generation field. [Uncertain]

### [Merged Survey of Mainstream Video Pretraining Datasets: Panda-70M, InternVid, Koala…](../models/pretraining_datasets.md) ⚠️

**Safety and compliance filtering across the seven is overall weak, with only two having dedicated safety classifiers**:
- **InternVid (has a dedicated classifier)**: uses a **binary classifier to identify and exclude unethical/NSFW videos**, while also restricting search terms and channels to the SFW range at the crawling source. This is the earliest and clearest safety step among the seven.
- **MiraData (has a dedicated classifier)**: uses the **Stable Diffusion Safety Checker**, checking **8 uniformly sampled frames** per video frame by frame; NSFW clips are removed from **all versions, including the 788K unfiltered pool** — i.e., safety filtering is front-loaded and cannot be bypassed, which is more rigorous than the others.
- **Panda-70M (has content and privacy handling but no NSFW classifier)**: before release, an internal automated process filters out samples containing harmful/violent language and mentions of drugs or hate speech, and uses **NLTK to replace all person names in captions with "person"**. This is the only privacy de-identification measure among the seven. But there is **no visual NSFW detection**.
- **Koala-36M (relies only indirectly on manual annotation — a clear weak point)**: the only place safety is touched is in the "video naturalness" dimension of the manual annotation guidelines — requiring annotators to remove "political, terrorist, violent, gory, or otherwise uncomfortable content." The consequence: **safety is folded into a single scalar VTSS with a threshold of 2.5, fully entangled with quality** — an unsafe but aesthetically and dynamically excellent video could entirely pass. **There is no independent NSFW classifier and no explicit pornographic-content category**, making it impossible to audit or tune separately. This is the compliance shortcoming most deserving of being called out among the seven.
- **UltraVideo**: **no NSFW filtering, no face/privacy filtering**. Its 16 low-quality attribute judgments target image-quality defects rather than content safety. Safety implicitly relies on the relatively clean source of "YouTube 4K/8K curated pool + manual re-review."
- **OpenVid-1M**: The paper, repository, and HF card **mention no NSFW or safety filtering whatsoever**. [Uncertain]
- **LVD-2M**: A full-text grep confirms that **"NSFW" and "watermark" occur 0 times**, **no safety filtering of any kind**, implicitly relying on the individual cleaning of the four upstream datasets.
- **Copyright-content detection**: **none of the seven** has it.
- **Output side**: none of the seven has generated-content watermarking or detection tooling (they are datasets, but the accompanying models UltraWan, MVDiT, ViCLIP, etc., also come with no safety classifier).
**Overall judgment**: the ranking of safety-filtering completeness is MiraData ≈ InternVid > Panda-70M (privacy- and text-leaning) > UltraVideo (relies on the source) > Koala-36M (entangled within VTSS) > OpenVid-1M ≈ LVD-2M (none). This stands in stark contrast to "quality filtering has become ever more refined" — **over two years, the seven datasets took quality filtering from nothing to a 72B VLM, while safety filtering has essentially stood still or even regressed**.
