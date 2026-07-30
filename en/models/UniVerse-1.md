# UniVerse-1 (with companion benchmark Verse-Bench)

> Topic: Data processing for video generation models (including simultaneous audio-video generation): data cleaning pipeline, data distribution, annotation methods

[← Back to Home](../index.md)

**Table of Contents**: [Basic Information](#basic-information) · [Data Scale and Distribution](#data-scale-and-distribution) · [Cleaning Pipeline](#cleaning-pipeline) · [Annotation Method](#annotation-method) · [Audio-Video Alignment](#audio-video-alignment) · [Training Coordination](#training-coordination) · [Effectiveness Comparison](#effectiveness-comparison)

## Basic Information

### Name

UniVerse-1 (with companion benchmark Verse-Bench)

### Releasing Institution/Company

StepFun (阶跃星辰) is the lead/dominant institution, in collaboration with the Hong Kong University of Science and Technology (Guangzhou), the Hong Kong University of Science and Technology, and Tsinghua University. Author list: Duomin Wang (王多民, first author, maintainer of the project homepage and repository Dorniwang), Wei Zuo, Aojie Li, Ling-Hao Chen (陈凌灏, Tsinghua), Xinyao Liao, Deyu Zhou, Zixin Yin, Xili Dai (戴习理, HKUST-GZ), Daxin Jiang (姜大昕, StepFun founder/CEO), Gang Yu (于刚, StepFun's head of vision). Positioned as StepFun's first open-source exploration in joint audio-video generation.

### Release Date (technical report/paper/open-source timeline)

Project homepage went live on September 3, 2025; arXiv submission on September 7, 2025 (arXiv:2509.06155v1, cs.CV, UTC 17:55:03); model weights, inference code, and Verse-Bench dataset open-sourced on September 8, 2025; technical report officially released on September 9, 2025; Verse-Bench evaluation metric tooling additionally released on September 28, 2025. There is also an OpenReview submission record (forum id 8aFYx2mDyE).

### Type (model/dataset/toolchain/evaluation benchmark)

A combined output of model + evaluation benchmark. The main body is a joint audio-video generation model (UniVerse-1-Base, 7B parameters); it is released together with the Verse-Bench evaluation benchmark (600 image-text prompt pairs, with companion evaluation metric tooling). This is not a dataset release — the 7,685 hours of training data itself is not open-sourced. Complete inference code is included, but training code and data-cleaning scripts are not.

### Degree of Openness (whether weights/code/data/pipeline are each open-sourced)

Moderately-high degree of openness, under the Apache-2.0 license.
【Weights】Open. dorni/UniVerse-1-Base is released on HuggingFace (7B parameters, F32/safetensors, bfloat16 inference), with only this one variant and no multi-resolution version.
【Code】Partially open. The GitHub repository Dorniwang/UniVerse-1-code releases inference code (built on diffusers, Python≥3.10, PyTorch≥2.5.0-cu121); training code and the server-side implementation of the online annotation pipeline are not open-sourced.
【Evaluation benchmark】Open. Both the Verse-Bench dataset and its evaluation metric tooling are released on HuggingFace.
【Data-processing pipeline】At the methodological level, Section 3 of the paper discloses stage-by-stage thresholds (resolution, bitrate ratio, DOVER, clip duration, and SyncNet thresholds are all given as concrete numbers), so reproducibility is fair; however, no cleaning scripts or original annotation prompts have been released, and the disclosure granularity is clearly lower than MOVA's (no stage-by-stage retention-rate table, no full prompt text, no pseudocode).
【Data】Training data is not open-sourced. The public-dataset portion (VGGSound, AudioSet) is freely obtainable, but the self-collected YouTube/Pexels/movie-clip portion is not disclosed, nor is the list of the filtered subset.

### Whether Simultaneous Audio-Video Generation Is Supported, and the Implementation Approach (native joint / cascaded / MoE fusion)

Supported, as native joint generation (a single inference pass produces video and audio simultaneously), but the implementation path is the core innovation of this work — rather than training a joint model from scratch, it "stitches" two already-pretrained single-modality experts together (Stitching of Experts, SoE):
【Expert sources】Video expert Wan2.1 (1.3B DiT + 3D VAE + umT5 text encoder); audio expert Ace-step (3.5B music-generation model, including Music-DCAE + umT5 + lyric encoder + speaker encoder + DiT). The merged model totals roughly 7B parameters.
【Fusion method】Deep fusion occurs at the level of corresponding blocks across the two DiTs, rather than only concatenating at the input/output ends. The fusion component is a lightweight cross-modal MLP connector: two-layer linear adapters align the feature spaces, and dedicated key (kproj) and value (vproj) projection layers are configured for cross-modal attention.
【Depth-alignment problem】The two experts have different numbers of transformer layers, resolved via layer interpolation — new blocks are inserted at uniform intervals, with their weights initialized by linear interpolation of the weights of adjacent layers, so that the two towers' depths can be stitched one-to-one.
【Structural modification】The speaker encoder originally in Ace-step is removed, with the goal of freeing the model from speaker-specific generation constraints and generalizing to arbitrary speakers (timbre is implicitly determined jointly by the reference image and the text).
【Temporal-grid alignment】The audio sample rate is adjusted from the original 44.1 kHz to 25.6 kHz, so that the audio latent's time grid strictly aligns with the video's 25 fps — this is an engineering trade-off that adapts the sample rate to fit the video frame rate, differing from MOVA's approach of using Aligned RoPE for index rescaling.
【VAE】The video 3D VAE compresses (3,T,H,W) into (16,T/4,H/8,W/8); the audio Music-DCAE compresses the mel spectrogram (8,T,F) into (8,T/8,F/8).
【Noise sampling】An Independent Noise Sampling Strategy (INSS) is proposed, avoiding spurious cross-modal noise correlations that would arise if the two modalities shared PRNG state.
【Task form】Reference image + text → video + audio (IT2VA); Verse-Bench simultaneously supports evaluation across four task types: joint generation, audio-to-video, video-to-audio, and TTS.

### List of Research Information Sources (URLs of papers/technical reports/official documents/news, with each source's nature labeled: official primary / same-team corroboration / third-party report)

- 【Official primary】arXiv:2509.06155v1 "UniVerse-1: Unified Audio-Video Generation via Stitching of Experts" (submitted 2025-09-07): https://arxiv.org/abs/2509.06155 , HTML full text https://arxiv.org/html/2509.06155v1 — the sole direct source for the vast majority of fields in this entry, especially Section 3's data construction and online annotation pipeline, Section 4's experiments and ablations, and the appendix's Verse-Bench category table.
- 【Official primary】GitHub repository Dorniwang/UniVerse-1-code: https://github.com/Dorniwang/UniVerse-1-code/ — open-source checklist, Apache-2.0 license, release timeline (2025-09-03 project page / 09-08 weights and Verse-Bench / 09-09 technical report / 09-28 evaluation tooling).
- 【Official primary】HuggingFace model page dorni/UniVerse-1-Base: https://huggingface.co/dorni/UniVerse-1-Base — 7B parameter scale, bfloat16 inference configuration, Apache-2.0 declaration, the "7,600 hours of training data" statement.
- 【Official primary】Project homepage: https://dorniwang.github.io/UniVerse-1/ — institutional affiliation (StepFun / HKUST-GZ / HKUST / Tsinghua), demo samples, description of the layer-interpolation method, composition of the Verse-Bench three subsets.
- 【Official primary】HuggingFace Papers page: https://huggingface.co/papers/2509.06155 — cross-checking the abstract and core contributions.
- 【Same-team corroboration】OpenReview submission page https://openreview.net/forum?id=8aFYx2mDyE — used to confirm the paper's conference submission status.
- 【Third-party corroboration】ResearchGate paper entry: https://www.researchgate.net/publication/395356081_UniVerse-1_Unified_Audio-Video_Generation_via_Stitching_of_Experts
- 【Third-party corroboration】Subsequent same-direction works citing and comparing against UniVerse-1, such as UniAVGen (arXiv:2511.03334), JavisDiT++ (arXiv:2602.19163), MMControl (arXiv:2604.19679) — used to confirm that UniVerse-1 has become one of the standard reference baselines for joint audio-video generation.

## Data Scale and Distribution

### Training Data Volume (video count/hours/token count, pretraining and SFT separated)

A total of 7,685 hours of audio-video data (the paper body and model card frequently give the approximate figure of "about 7,600 hours"), all used in a single fine-tuning stage — there is no pretraining/SFT scale split, because UniVerse-1's "pretraining" directly inherits the weights of the two existing experts, Wan2.1 and Ace-step, and only performs one round of joint fine-tuning itself.
【Composition of the three parts and their respective hours】
- Rigorously verified speech-centric data: 1,187 hours;
- General-purpose audio-video data: 3,074 hours;
- Public datasets VGGSound + AudioSet: 3,422 hours.
【Notes on scope】Only hour counts are given; the number of clips and token count are not disclosed, nor is the original collection volume given (so the overall retention rate cannot be back-calculated).
【Training volume】Effective batch size 128, gradient accumulation 4 steps, 50k steps total, AdamW, learning rate 5e-6, FSDP multi-node distributed training. Number of GPUs and GPU-days are not disclosed.
【Scale positioning】7.6k hours is small-to-medium scale among comparable works — about 1/8 of MOVA Phase 1 (61.5k hours), and close to the scale of MOVA's Phase 3 high-quality subset (11k hours). In the Limitations, the paper explicitly cites compute constraints as the main reason, and lists "larger-scale, more refined data curation" as future work.

### Data Source Composition (proprietary / public datasets / web crawling / licensed procurement / synthetic data)

Three source categories, with self-collected data and public datasets each accounting for roughly half:
【Self-collected web data (approx. 4,261 hours, i.e., 1,187 + 3,074)】
- YouTube is the primary source, with content types explicitly enumerated as: music variety shows, classical music performances, cooking tutorials, public speeches, interviews, vlogs, tool usage demonstrations;
- Cinematic movie clips;
- Pexels stock footage videos.
【Public datasets (3,422 hours)】VGGSound and AudioSet, with the paper explicitly stating the purpose of introducing them is to "further bolster the audio modality" — i.e., to supplement acoustic diversity in sound effects/ambient audio rather than to improve visual quality — this is also why the LQLS low-quality loss strategy is later applied to them separately.
【Licensed procurement】No paid, licensed procurement data is mentioned.
【Synthetic data】No model-synthesized video or audio training data is used; the only "synthetic" component is that all annotations are automatically generated by QWen2.5-Omni and Whisper (synthetic annotation rather than synthetic content).
【Intent behind source selection】The listed content types closely match the model's three major target capabilities: music variety/classical performances → instrument sound generation; speeches/interviews/vlogs → speech and lip sync; cooking/tool demos/Pexels → ambient sound and foley.

### Data Compliance and Provenance (proportion of licensed data, rights-cleared datasets, C2PA, etc.) ⚠️

The paper does not discuss data compliance and provenance issues at all: no proportion of licensed data is given, no rights-cleared datasets are declared, no C2PA or any output-side watermark/provenance marker is mentioned, and no copyright risk or portrait-rights issue is discussed. An indirectly inferable risk point: the training corpus explicitly includes scraped YouTube content and cinematic movie clips, which are copyright-sensitive sources; the Verse-Bench evaluation set is likewise collected from YouTube and Bilibili, and uses TED Talks material from September 2025. The Pexels portion falls under a relatively permissive free-stock license. The referenced VGGSound and AudioSet are themselves collections of YouTube links, and their copyright status is governed by the original datasets' terms. The model weights and code use the fully open, commercially permissive Apache-2.0 license, but there is zero explanation of training-data compliance — presenting the same "methodology disclosed, compliance not disclosed" asymmetry as MOVA. [Uncertain]

### Clip Duration Distribution and Splitting Strategy

【Duration of split output】Based on PySceneDetect scene splitting, clips shorter than 5 seconds after splitting are uniformly discarded, so the retained clips have a duration floor of 5 seconds. The paper does not give an upper bound, mean, or duration histogram, nor does it state whether longer clips receive secondary truncation, so what is actually retained are variable-length clips rather than fixed-length ones.
【Duration used in training】The full clip is not consumed directly; instead, the online annotation pipeline randomly samples a fixed-length window (the paper gives 5 seconds as the example) from the clip at each sampling step and feeds it into training. That is, a two-tier design of "variable length on ingestion (≥5s), fixed length on consumption (~5s)," where the random window position itself provides data augmentation.
【VGGSound/AudioSet】Follows a simplified processing flow, with the same 5-second minimum-duration constraint applied.
【Relationship to video frame rate】Video is standardized to 25 fps, so 5 seconds corresponds to 125 frames; the audio sample rate is specifically lowered to 25.6 kHz to align with the 25 fps time grid.
【Comparison】Compared to MOVA's strict fixed duration of 8.05 seconds (193 frames @24fps) plus VAD-aware windowing, UniVerse-1's splitting strategy is noticeably coarser: there is no VAD-driven adaptive window start, and there is no guarantee that a window won't cut off an utterance.

### Resolution/Aspect-Ratio Distribution and Bucketing Strategy ⚠️

【Resolution admission threshold】Source videos with resolution below 1080p are directly discarded — a fairly aggressive, high-bar filter that excludes a large amount of 720p material, reflecting a "better fewer but better" orientation. A bitrate-to-resolution ratio below 600 is simultaneously applied as an exclusion constraint, used to exclude "falsely high-definition" heavily-recompressed material that is high-resolution but low-bitrate — this metric is relatively rare among comparable works.
【Exception】VGGSound and AudioSet follow the simplified processing path and are not subject to the 1080p or bitrate-ratio constraints (these two datasets are inherently low visual quality), and instead the LQLS loss strategy on the training side is used to isolate the effect of their low-quality visual signal.
【Frame rate】Standardized to 25 fps.
【Aspect ratio】The paper describes no aspect-ratio standardization, bucketing, or padding strategy, nor does it give an aspect-ratio distribution.
【Training/inference resolution】The paper does not disclose the actual video resolution and frame-count configuration used at training and inference time; neither the model card nor the code repository give resolution specifications. Constrained by the capabilities of the base Wan2.1-1.3B, it is presumed to be around 480p, but there is no official data to support this. [Uncertain]

### Category/Domain Distribution and Mixture Strategy (proportional control and concept balancing across people, actions, scenes, styles, etc.) ⚠️

Domain coverage of the training data is only qualitatively enumerated, with no proportion numbers and no description of any concept-balancing mechanism:
【Content categories】Music variety shows, classical music performances, cooking tutorials, public speeches, interviews, vlogs, tool usage demonstrations, cinematic movie clips, Pexels stock clips.
【The only quantifiable mixture split】A three-way split by data "nature" rather than "subject matter": speech-centric 1,187 hours (15.4%), general-purpose audio-video 3,074 hours (40.0%), VGGSound+AudioSet 3,422 hours (44.5%). Notably, speech data accounts for only 15%, while sound-effect/ambient-audio-type data accounts for the vast majority — this is exactly the opposite of MOVA's extreme orientation of "keeping only speech clips"; UniVerse-1 prioritizes general soundscape over speech in its mixture.
【Indirect evidence of the intent behind the subject-matter mixture】The chosen subject matter corresponds one-to-one with the three target capabilities (music category → instrument sound, speeches/interviews category → speech lip sync, cooking/tool category → foley and ambient sound), indicating the mixture was collected by working backward from "target sound type" rather than by balancing visual subject matter.
【The only percentage-based category table】The nine-category audio-category breakdown given in the paper's appendix (natural environment 36.1%, etc.) describes the sample distribution of the Verse-Bench evaluation benchmark, not the training-data distribution, and the two should not be conflated.
The subject-matter domain mixture numbers for the training data are an information gap. [Uncertain]

### Audio Category Distribution and Mixture (proportion and control strategy for speech / sound-effects-foley / music / ambient sound / silence) — a dimension unique to AV models

Audio category mixture is the key dimension on which UniVerse-1 diverges from MOVA — UniVerse-1 deliberately preserves the dominance of non-speech data:
【Three-way mixture (by hours)】
- Speech (speech-centric, verified via triple check: Whisper speech detection + RetinaFace face detection + SyncNet conf>2.0): 1,187 hours, 15.4%;
- General-purpose audio-video (general, i.e., clips Whisper judges to contain no speech, covering ambient sound, sound effects, instrumental music, etc.): 3,074 hours, 40.0%;
- VGGSound + AudioSet (dedicated sound-effect/event-audio supplement): 3,422 hours, 44.5%.
That is, non-speech data totals 84.6%, while speech accounts for only 15.4%.
【Classification mechanism】Whisper ASR is used as a binary gate: clips from which valid speech content can be detected enter the speech subset and undergo further face and lip-sync verification; clips with no detectable speech are not discarded but instead downgraded and retained as "general-purpose audio-video data." This is a "nothing wasted" design — a single filtering rule simultaneously performs routing and retention, rather than MOVA's direct elimination.
【Silence handling】Audio activity detection is performed via three signal-level metrics — volume, energy, and zero-crossing rate — to discard silent clips. The training set therefore contains no silent samples.
【Music category】Music capability is mainly carried by the base model Ace-step (itself a music-generation model), while on the training-data side it is reinforced by music-variety and classical-performance material, but no separate hour count for music is given.
【Undisclosed】The internal breakdown of the general audio-video subset (ambient sound/sound effects/instrumental music/background music) is not given, nor is it stated whether per-category sampling weights are adjusted during training.
【Comparison with Verse-Bench】The evaluation side gives an explicit nine-category audio breakdown: natural environment 36.1%, music and instruments 19.3%, daily life 10.2%, human voice 7.6%, transportation 4.5%, industrial and urban 3.9%, weapons and explosions 2.5%, special effects sound 2.3%, unclassified 15.8%. The evaluation set is likewise dominated by natural ambient sound with human voice at only 7.6%, consistent with the orientation of the training mixture.

### Narrative Structure Distribution (single-shot vs. multi-shot, average clip duration, shot-count distribution, presence of native audio track)

【Single-shot vs. multi-shot】UniVerse-1 adopts a "purely single-shot" strategy without explicitly discussing it: after PySceneDetect detects scene cut points and splits accordingly, only clips with post-split length ≥5 seconds are retained, so every training clip naturally falls within a single shot, with no cross-shot samples. The paper does not treat "multi-shot" as an independent data dimension, nor does it generate any multi-shot training samples — this contrasts sharply with MOVA's explicit construction of a 2×2 taxonomy of "single/multi-shot × speech/non-speech" clips, and also means UniVerse-1 lacks the ability to generate shot transitions.
【Average clip duration】Only the lower bound of 5 seconds is known, and the training consumption window is about 5 seconds; the mean and distribution are not disclosed.
【Presence of native audio track】Mandatory. The very first step of the cleaning pipeline is "videos lacking an audio track were immediately discarded," so all training data comes with a native, synchronized audio track — there are no post-dubbed or audio-track-less samples.
【Shot-count distribution】Not applicable (all single-shot).
【Narrative complexity】Constrained by the roughly 5-second window length, the training data consists of short-duration single-shot clips that do not carry narrative structure information.

### Language/Accent Distribution (data basis for multilingual lip-sync capability) ⚠️

The paper does not discuss language or accent distribution at all: no list of supported languages, no per-language hour counts or proportions, no accent annotation, and no discussion of multilingual phoneme-viseme mapping. Indirectly inferable clues include:
- The training side uses Whisper for ASR, which is itself multilingual, but the paper does not state whether languages are restricted or whether a language distribution was tallied;
- The data sources are YouTube speeches/interviews/vlogs, so English content is very likely the largest proportion;
- On the evaluation side, Verse-Bench's Set2-V is explicitly collected from both YouTube and Bilibili, and the inclusion of Bilibili indicates the evaluation set covers Chinese-language content; Set3-Ted is drawn from TED Talks from September 2025, which are English-language speeches. So at least at the evaluation level, both Chinese and English are covered;
- Architecturally, the Ace-step speaker encoder was removed, with the goal of not tying the model to a specific speaker, but this does not concern the language dimension.
Language and accent distribution constitute a complete information gap. [Uncertain]

## Cleaning Pipeline

### Overall Structure of the Cleaning Funnel (number of filtering stages, order of stages)

A six-stage sequential funnel (for self-collected data), with VGGSound/AudioSet following a simplified bypass. The overall structure is "hard-threshold check first, then shot splitting, then audio judgment, then lip-sync verification last" — a progressively narrowing sequence:
【Stage 1: Audio-video pre-screening】Videos lacking an audio track are immediately discarded — treating "must have a native synchronized audio track" as the most front-loaded veto condition.
【Stage 2: Quality control (visual hard metrics)】Three parallel thresholds: resolution below 1080p discarded; bitrate-to-resolution ratio below 600 discarded; DOVER aesthetic-quality score below 0.6 discarded.
【Stage 3: Temporal coherence (shot splitting)】PySceneDetect performs scene splitting; clips shorter than 5 seconds after splitting are discarded.
【Stage 4: Audio activity detection】Volume, energy, and zero-crossing rate — three signal-level metrics — are analyzed to discard silent clips.
【Stage 5: Speech-content verification (routing point)】Whisper ASR detects whether speech is present — clips containing speech enter the speech subset and proceed to Stage 6; clips without speech are not eliminated but are directly routed into "general-purpose audio-video data." This is the only routing (rather than elimination) node in the funnel.
【Stage 6: Face and lip-sync verification (speech subset only)】RetinaFace performs face detection; then SyncNet computes a lip-audio-sync confidence score, requiring conf > 2.0 for retention; clips that pass are explicitly tagged as "containing speech."
【Bypass: VGGSound/AudioSet】Only the 5-second minimum-duration constraint is applied, skipping all quality and alignment filters — resolution, bitrate, DOVER, SyncNet, etc. — because the purpose of introducing them is to bolster audio, not visuals. The negative impact of their low visual quality is instead isolated on the training side via the LQLS loss strategy — a "if filtering can't solve it, solve it with the loss function" approach, one of the more distinctive aspects of this work.
【Stage 7 (online, during training): annotation】Not part of the offline funnel, but an online annotation service running concurrently with training — see caption_model and model_as_data_judge.
【Overall characteristics】Compared with MOVA, UniVerse-1's funnel is shorter and shallower (6 stages vs. MOVA's three major stages with a dozen-plus metrics); it has no audio aesthetic scoring, no semantic-alignment filtering, no OCR subtitle filtering, and no deduplication — but moving the most expensive annotation step from offline to online is its biggest structural differentiator.

### Quantitative Funnel Retention Rate (input/output volume at each stage and overall retention rate, e.g. Apollo 27%) ⚠️

The paper gives no quantitative funnel data at all: the total volume of originally collected video (in hours or clip count) is not disclosed, nor are the input/output volumes at each filtering stage, nor is any per-stage or overall retention rate given. All that can be confirmed is that the final output is 7,685 hours (speech 1,187 + general 3,074 + public datasets 3,422). Since the numerator is known but the denominator is entirely missing, an overall retention rate comparable to Apollo's 27% or MOVA's 26.39% cannot be computed.
What can be qualitatively inferred is that the retention rate should be fairly low: the three hard thresholds — resolution ≥1080p, bitrate ratio ≥600, DOVER ≥0.6 — stacked together, plus the <5-second exclusion after PySceneDetect splitting, plus the speech subset's additional SyncNet conf>2.0 requirement, together represent a considerable amount of cumulative elimination; the relatively small scale of the speech subset (only 1,187 hours) also indirectly reflects the strictness of this chained condition of "has audio track + has face + meets lip-sync standard." But all of this is inference, unsupported by any figures in the paper. [Uncertain]

### Shot-Splitting Method (PySceneDetect / in-house model / shot-aware splitting)

PySceneDetect is used (the paper gives the repository address https://github.com/Breakthrough/PySceneDetect in a footnote) for scene-change detection and splitting at the cut points — a fairly standard industry practice, with no in-house model and no shot-aware special design.
【The only post-hoc constraint】Clips shorter than 5 seconds after splitting are directly discarded, to ensure temporal coherence and to satisfy the downstream 5-second training-window requirement.
【Relationship to speech boundaries】Splitting is driven entirely by visual scene cut points and is not jointly decided together with speech boundaries (VAD) — this is considerably coarser than MOVA's approach of jointly using Silero VAD + PySceneDetect signals to generate windows with an adaptive window start that avoids cutting off utterances, meaning UniVerse-1's training windows may start mid-sentence or cut a sentence short.
【Splitting parameters】The detector type (ContentDetector / AdaptiveDetector) and threshold are not disclosed.
【VGGSound/AudioSet】No shot splitting is performed; only the 5-second minimum-duration constraint is applied.

### Quality Filtering (aesthetic scoring, sharpness, OCR text filtering, letterbox/watermark/logo detection) ⚠️

Visual-side quality filtering is concentrated in Stage 2 of the funnel, comprising three hard thresholds, all with disclosed numeric values:
【Resolution】Videos below 1080p are discarded. The bar is relatively high among comparable works (MOVA uniformly regularizes to 720p for training and even starts as low as 360p), reflecting high selectivity given the small data scale.
【Bitrate-to-resolution ratio】Below 600 is discarded. Used to identify material that is "falsely high-resolution but actually heavily compressed" — a relatively distinctive metric, rare in comparable works.
【Aesthetic/technical quality】DOVER overall quality score below 0.6 is discarded. Unlike MOVA, which splits DOVER-Aesthetic (>0.85) and DOVER-Technical (>0.05) into two separate scores, UniVerse-1 uses only a single overall DOVER score with a threshold of 0.6, a coarser granularity.
【Threshold-determination method】The paper does not explain how the thresholds were calibrated (neither manual spot-checking nor ablation experiments are mentioned); they appear to be empirically set. [Uncertain]
【Filtering not performed】No OCR burned-in subtitle detection (MOVA has this, and even turns it into a controllable attribute), no watermark/logo detection, no letterbox detection and cropping (no mention of cropdetect-type processing), no independent aesthetic scorer (such as LAION-Aesthetic), no dedicated sharpness/blur metric.
【Alternative handling of low-quality data】For VGGSound/AudioSet, which cannot pass the above filters, no filtering is applied; instead the training-side LQLS strategy is used for isolation — Flow Matching loss is applied to these two datasets only when the diffusion timestep t > 800 (out of 1000 total steps), meaning they only participate in learning the high-noise segment (coarse structure/semantic layer) and not the low-noise segment (high-frequency detail layer), thereby avoiding overfitting the model to their low-quality visual high-frequency artifacts. This is a typical case of shifting the data-quality problem from the "filtering dimension" to the "loss-weighting dimension."

### Motion Filtering (optical-flow/motion-score thresholds, removal of static or jittery footage) ⚠️

The paper describes no independent motion-filtering step at all: no optical-flow computation, no motion-score threshold, no removal of static shots, no removal of handheld shake or camera jitter. Motion-related quality control is only implicitly present within the DOVER score (whose technical dimension includes perceptual assessment of temporal distortion, stutter, and blur).
It is worth noting that on the evaluation side, motion intensity is instead treated as a core observed quantity: Motion Score (MS) is the first metric in the Verse-Bench main table, and the ablation results show this metric is highly sensitive to training strategy (full model 0.20, w/o LQLS 0.38, w/o INSS 1.10) — here, a large deviation in MS value does not equate to "richer motion," but rather co-occurs with a simultaneous drop in aesthetic score and ID consistency, more likely reflecting visual instability or spurious motion caused by artifacts. This indirectly shows that the lack of motion filtering on the data side ultimately translates into a compensation need at the training-strategy level. [Uncertain]

### Deduplication Method (exact deduplication and embedding-based semantic deduplication recorded separately) ⚠️

The paper does not mention deduplication at all: there is neither hash/fingerprint-level exact deduplication, nor embedding-based semantic deduplication, and cross-source duplication is not discussed. The risk is objectively present — self-collected data is predominantly YouTube-sourced, while the concurrently introduced VGGSound and AudioSet are themselves collections of YouTube clips, so there is potential content overlap among the three; furthermore, Pexels material being repeatedly included across multiple public datasets is also common. The paper provides no explanation of this whatsoever. [Uncertain]

### VLM/LLM as Quality Judge (multimodal large-model quality scoring and mismatch removal, the 2026 trend shifting from shallow scorers toward large-model semantic judgment) ⚠️

UniVerse-1's approach to model involvement differs markedly from the mainstream 2026 trend — the large model handles only annotation, not quality judgment; quality determination is still handled entirely by traditional discriminative small models:
【The large model's role: annotation only, no adjudication】QWen2.5-Omni, the only multimodal large model used, is responsible for generating three-way aligned annotations online during training (speech content, video description, ambient-sound description). The paper describes no quality scoring, cross-modal consistency verification, or conflict-resolution step performed by an LLM/VLM — this stands in clear contrast to MOVA's use of GPT-OSS-120B for cross-modal consistency adjudication, with a built-in hallucination_check self-audit block in the prompt.
【Quality checking is still handled by dedicated discriminative models】DOVER (video quality), Whisper (speech presence judgment + ASR), RetinaFace (face detection), SyncNet (lip-audio-sync confidence), and the three signal-level statistics of volume/energy/zero-crossing rate. All of these are dedicated small models or hand-crafted features, none is a general-purpose large model.
【The real innovation lies in "when to annotate," not "who annotates"】The paper's core claim is: traditional offline annotation (pre-generating a single caption for an entire long video) causes a temporal mismatch between the annotation and the actual training window — the content of a randomly sampled 5-second window during training does not correspond to a global caption covering the whole video, and this "misalignment text-based annotation" directly harms the generative model's performance. The solution is to move annotation into the training loop: an independent service process running concurrently with training generates an annotation on the fly for each actually sampled fixed-length window, ensuring that "every frame of imagery, every segment of audio, and every sentence of text fed into the model" strictly share the same source and window. This reframes the data-annotation alignment problem from an "annotation quality problem" into an "annotation timing problem," and is this work's most methodologically valuable contribution on the data side.
【The cost】Online annotation means every training step incurs an inference cost from QWen2.5-Omni + Whisper, requiring an independent annotation-service cluster running in parallel with the training cluster; the paper does not disclose the compute overhead, throughput capability, or whether this service becomes a training bottleneck.
【Hallucination protection】The paper describes no constraint design targeting cross-modal hallucination in the multimodal annotator (such as prohibiting the visual annotator from referencing audio), and the original annotation prompt text is not disclosed either. [Uncertain]

### Safety and Compliance Filtering (NSFW, copyright, face/privacy) ⚠️

The paper does not touch on safety and compliance filtering at all: no NSFW detection, no copyright filtering, no face-privacy protection measures (although RetinaFace is used for face detection, its purpose is to select speaker shots rather than privacy de-identification), and no model-card-level safety statement or usage restriction. This stands in clear tension with its fully open, commercially permissive Apache-2.0 license and the model's capability of generating lip-synced video for arbitrary faces (which could be used to fabricate spoken-word videos of real people). Given that this model's headline capability is "reference image + text → speaking-person video with audio," its risk of being misused for deepfakes is significantly higher than that of purely sound-effect-oriented models, and the paper offers zero discussion of this. [Uncertain]

## Annotation Method

### Caption Model Used (in-house VLM / open-source model, model scale) ⚠️

Entirely open-source models are used, with no in-house captioner, and both the number and scale of models are far smaller than comparable works:
【Multimodal annotation】QWen2.5-Omni (Alibaba's Tongyi Qianwen omni-modal model) handles all three annotation streams — simultaneously outputting verified speech content, a descriptive video caption, and an ambient-audio caption. This is a single model with multiple outputs, rather than MOVA's division of labor across three models. The paper does not disclose the specific parameter-scale tier used (3B / 7B). [Uncertain]
【Speech transcription】Whisper (OpenAI) performs ASR. In the offline funnel it is used to determine whether a clip contains speech (the routing gate); in online annotation it is used to transcribe the audio of the sampled window. The paper does not disclose the specific Whisper version (base/large-v2/large-v3). [Uncertain]
【No fusion model】There is no MOVA-style large-parameter LLM fusion and consistency-adjudication step (MOVA uses GPT-OSS-120B). The three annotation streams are produced simultaneously and in parallel by QWen2.5-Omni in one pass, remaining as independent fields.
【Text encoder】Text-condition encoding on both the training and inference side uses umT5 (both base experts share the same text-encoder family, which is also one of the preconditions for the SoE stitching to work).
【Annotation model on the evaluation side】The Verse-Bench Set2-V subset uses an LLM to generate captions + Whisper for ASR, subsequently verified by humans.
【Scale comparison】UniVerse-1's total annotation-model scale (QWen2.5-Omni + Whisper) is far smaller than MOVA's (MiMo-VL-7B + Qwen3-Omni×2 + GPT-OSS-120B), which is directly related to its choice of online annotation architecture — online annotation requires the annotation model to be light enough to keep pace with training throughput, and cannot afford the per-sample inference overhead of a 120B-class model. This is a trade-off between "annotation timing" and "annotation-model capacity."

### Caption Density and Structuring Degree (short/long/dense descriptions, structured fields such as camera motion, style tags) ⚠️

A structured annotation of three parallel fields is used, without fusion — a typical factorized schema:
【Three outputs】QWen2.5-Omni is instructed to output three mutually aligned, independent annotations:
1) verified speech content — verified speech content (the Whisper ASR transcription result verified/regularized by the multimodal model);
2) descriptive video caption — a descriptive caption of the visual imagery;
3) ambient audio caption — a caption of the ambient/non-speech audio.
【Degree of structuring】The three-field division itself is structured, but the content within each field is free-form natural-language text, with no further enumerable fields (no camera-motion tags, no style tags, no shot-scale/composition fields, no on-screen text fields).
【Density and length】The paper gives no caption-length statistics, provides no complete caption examples, and does not disclose the original annotation prompt. The density attribute cannot be assessed. [Uncertain]
【Temporality】The caption corresponds to the roughly 5-second window actually sampled during training, so its temporal granularity is "window-level" rather than "whole-clip-level" — this is precisely the core problem online annotation is designed to solve. Whether fine-grained event ordering within the window (e.g., "first…then…") is performed on the time axis, however, is not stated.
【Relationship to training conditioning】The three fields are fed into the model as parallel text conditions (encoded via umT5), remaining separate rather than being merged into a single paragraph — this differs in path from MOVA's approach of "routing at annotation time, fusing into a single caption at training time": UniVerse-1 "routes at annotation time and also routes at training time."
【Controllable attributes】No explicit controllable marker is designed (such as MOVA's "This video has no subtitles.").

### Joint Audio-Video Caption Structure (whether it covers both visual and auditory tracks simultaneously, whether it is routed into independent fields, as in LTX-2's full-soundscape description, Script-a-Video's factorized streams, Foley-Omni's three fields)

UniVerse-1 adopts a purely factorized-streams scheme, and is among the most "thoroughly separated" samples of joint audio-video caption structure in this survey:
【Three tracks kept in parallel, not fused】The visual track (video caption), speech track (verified speech content), and non-speech audio track (ambient audio caption) are produced in one pass by QWen2.5-Omni and retained as three independent fields all the way to the training stage, never merged.
【Design orientation】Compared with LTX-2's "single unified soundscape description" or MOVA's "route then fuse into a single natural-language paragraph via a 120B LLM," UniVerse-1's scheme is closer to Script-a-Video's factorized streams and Foley-Omni's three-field parallel design. Its advantage is that the conditions of the three modalities can be independently controlled at inference time (users can specify dialogue, imagery, and ambient sound separately); its downside is the lack of a cross-modal consistency-verification step — although the three annotation streams are produced simultaneously by the same model and theoretically share context, the paper does not state whether there is a mechanism to prevent or detect semantic conflicts among the three.
【Explicit split of speech and non-speech】Splitting "what was said" from "what other sounds are audible" into two fields is a natural choice aligned with the data mixture (speech only 15.4%, non-speech 84.6%): for most samples the speech field is empty and the ambient field dominates; the small number of speech samples have both.
【Correspondence with the generation task】The three-field structure directly supports Verse-Bench's four evaluation task types (joint generation, audio-to-video, video-to-audio, TTS) — because the conditions are separated, any subset of them can be masked out to construct different task forms.
【Key differentiator: online alignment】This scheme's real distinctiveness lies not in the field division itself but in the fact that the annotation for these three tracks is generated on the fly, at training time, for the actually sampled window; therefore the three annotation streams — and the audio-video latents fed into the model — inherently share the same time window, fundamentally eliminating the temporal mismatch of offline annotation. This is an alignment guarantee for the "joint caption schema" along the time dimension, not merely along the field dimension.

### Dialogue Transcription and Speaker-Attribute Annotation (ASR transcription + speaker identity/language/accent/emotion) ⚠️

【Transcription】Whisper is used for ASR to produce a verbatim transcript of speech content; QWen2.5-Omni then outputs "verified speech content" (verified speech content), i.e., the multimodal model performs a layer of verification/regularization on the ASR result. The paper does not explain the specific verification rules, whether unintelligible segments are handled, or whether the original language is preserved without translation. [Uncertain]
【Speaker-attribute annotation】Essentially absent. The paper annotates no speaker identity, gender, age, timbre, accent, emotion, speech rate, or any other speaker attribute, and performs no speaker diarization.
【Architectural-level decision on speaker modeling】More importantly, UniVerse-1 actively removes the speaker encoder originally in Ace-step, with the explicit goal of "generalizing the model beyond speaker-specific generation." This means timbre is not controlled by an explicit speaker embedding but is instead implicitly determined by the reference image and text conditions — the model card describes this capability as "generating speech with a character-matched voice timbre." This is a technical route entirely different from MOVA's (using natural language to describe timbre, accent, tone, and recording environment in detail, and binding this to the on-screen person during the fusion stage): UniVerse-1 relies on architectural simplification + implicit visual-conditioning, while MOVA relies on explicit annotation.
【Multi-speaker scenarios】The paper does not discuss the handling of multiple speakers, speaker switching, or dialogue turns; the training data's single-shot, roughly 5-second windows also largely do not carry multi-turn dialogue.
【Evaluation】WER (word error rate, 0.18 for UniVerse-1) is used to measure the intelligibility and text fidelity of generated speech, and LSE-C (1.34) is used to measure lip sync — but there is no MOVA-style cpCER (concatenated minimum-permutation character error rate that distinguishes speakers), since speaker-identity evaluation is not involved.

### Geometric and Structured Annotation (camera parameters, depth, 3D point tracks, action annotation, explicit state) ⚠️

The paper uses no geometric or structured annotation at all: no camera parameters, no depth maps, no 3D point tracks, no skeleton/pose/action annotation, no explicit physical-state annotation, no optical-flow field.
The only model involving spatial structure is the RetinaFace face detector, but its output is used solely as a boolean criterion within the funnel (whether a face is present in the frame, to decide whether the clip can enter the speech subset and undergo SyncNet verification); the detection-box coordinates and facial landmarks are not retained as annotations or used as training conditions.
Camera movement, shot scale, composition, and similar information are also not parameterized or tagged, and may only exist scattered in natural-language form within the video captions generated by QWen2.5-Omni, but since the paper discloses no caption examples this cannot be confirmed. [Uncertain]

### Synthetic Data Construction (controlled perturbation/editing to construct training pairs, e.g. InstructAV2AV)

The paper constructs no synthetic data whatsoever: no controlled-perturbation training pairs, no editing-based data augmentation (such as InstructAV2AV-style audio-video editing instruction pairs), no TTS-synthesized speech, no sound-effect overlay/mixing synthesis, and no self-distillation using data self-produced by a generative model. All training data consists of genuinely collected native audio-video clips (and mandatorily requires a native synchronized audio track).
【The two designs closest to "construction," though neither counts as data synthesis】
1) Random time-window sampling in online annotation — the same source clip may be sampled into different 5-second windows across different epochs/steps, each paired with a freshly generated annotation, which objectively provides data augmentation, but what is augmented is "window position + annotation," not synthesized new content;
2) Independent Noise Sampling Strategy (INSS) — independently sampling diffusion noise for the video and audio modalities, avoiding spurious cross-modal correlation from shared PRNG state. This is a training-side noise-construction technique and does not involve content synthesis. The ablation shows a significant effect (removing INSS worsens FD from 1.25 to 1.43, KL from 2.70 to 3.51, CLAP from 0.16 down to 0.11, LSE-C from 1.34 down to 0.99), indicating that cross-modal noise decorrelation is critical to both audio quality and lip sync.

### Degree of Human Involvement (manual annotation, manual quality inspection, model pre-screening + manual review)

Human involvement on the training-data side is essentially zero: the entire cleaning funnel (audio-track detection → resolution/bitrate/DOVER → PySceneDetect → audio activity detection → Whisper speech judgment → RetinaFace + SyncNet) is fully automated threshold-based judgment, and the annotation step is entirely automatically generated online by QWen2.5-Omni and Whisper. The paper mentions no human annotation, human spot-checking, or human review, nor does it state whether any thresholds were calibrated with human verification — this contrasts with MOVA, which explicitly states that "human spot-checking of the videos retained at different cutoffs" was used to set thresholds.
【The only human involvement is on the evaluation side】Verse-Bench's Set2-V subset (295 items, from YouTube and Bilibili) is constructed via: LLM-generated caption + Whisper ASR, followed by human verification. That is, human effort is concentrated on quality-assuring the evaluation benchmark, not the training data.
【Design implication】This is a resource-allocation orientation of "fully automated training data, semi-manual evaluation data," consistent with its overall positioning as compute/labor constrained (the Limitations explicitly acknowledge being constrained by computational resources).

## Audio-Video Alignment

### Audio-Video Synchronization Detection Method (lip sync, event alignment)

Audio-video synchronization detection in UniVerse-1 is the last and hardest checkpoint in the cleaning funnel, but it applies only to the speech subset, and only performs lip sync — not general event alignment:
【Lip-sync detection (used for offline filtering)】Two steps in series:
1) RetinaFace face detection — first confirming that a face suitable for lip-sync verification is present in the frame; speech clips with no face cannot enter the speech subset;
2) SyncNet lip-audio-sync confidence — for speech clips containing a face, a confidence score is computed, requiring > 2.0 for retention; those that pass are explicitly tagged as "containing speech."
Together, these two steps filter the 1,187 hours of high-confidence speech data out of a larger candidate pool.
【Event alignment (non-speech)】The paper performs no temporal-alignment detection whatsoever on the general non-speech audio-video data — no AV-align, no Synchformer-style temporal-alignment scoring as a filter condition, and no semantic-matching filter. That is, the 3,074 hours of general data + 3,422 hours of VGGSound/AudioSet undergo no audio-video alignment verification at all, relying solely on the natural assumption of a "native synchronized audio track" to guarantee alignment. This is a considerable simplification, and it also explains why the model performs adequately at coarse-grained coordination of ambient sound but lacks precise event-level alignment capability.
【Architecture-level alignment guarantee】The weakness of alignment on the data side is compensated for by the architecture and engineering side: (1) the audio sample rate is lowered from 44.1 kHz to 25.6 kHz, so the audio latent's time grid strictly aligns with the 25 fps video; (2) the two towers undergo deep fusion at corresponding block levels + cross-modal attention, so alignment happens continuously at the feature level; (3) online annotation ensures the text condition and the audio-video window share the same source.
【Evaluation side】Synchformer is used to compute AV-A (Audio-Video Alignment) temporal-sync score (lower is better, UniVerse-1 scores 0.23), and LSE-C is used to measure lip sync (higher is better, 1.34). Note that the Synchformer used for evaluation was not used as a filter for the training data.

### Specific Sync-Detection Metrics and Thresholds (SyncNet/Synchformer/LSE/in-house, threshold values, e.g. MOVA's LSE-D≤9.5 and LSE-C≥4.5, SkyReels-V4's SyncNet |offset|≤3∧conf>1.5) ⚠️

【The only hard threshold】SyncNet confidence score > 2.0, applied to speech clips already confirmed by RetinaFace to contain a face — the only quantitative sync-admission threshold in the training data.
【Assessment of threshold strictness】2.0 is a fairly lenient tier. As a cross-reference: MOVA adopts a bidirectional constraint of LSE-D ≤ 9.5 and LSE-C ≥ 4.5 (LSE-C is the same metric family as SyncNet conf, and its threshold of 4.5 is markedly stricter than 2.0); UniTalking, mentioned elsewhere in this survey, uses SyncNet conf > 0.9 (though the scope may differ). UniVerse-1 uses only a one-sided conf threshold, with no accompanying LSE-D distance constraint, and sets the bar at 2.0, indicating that in the trade-off between "preserving data volume" and "guaranteeing sync quality" it leans toward the former — given that the speech subset ends up with only 1,187 hours, a higher threshold would have made it difficult to support training with sufficient data.
【Threshold-calibration method】The paper does not explain how 2.0 was determined, with neither a description of manual spot-checking nor a threshold ablation experiment. [Uncertain]
【No sync threshold for non-speech data】The 3,074 hours of general data and 3,422 hours of VGGSound/AudioSet are subject to no sync threshold whatsoever.
【Mismatch between evaluation metrics and training filter metrics】Evaluation uses LSE-C (1.34) and Synchformer AV-A (0.23), while training filtering uses only SyncNet conf — the absolute value of LSE-C 1.34 is on the low side (MOVA-720p reaches 6.593), which can be partly attributed to the lenient sync threshold on the training side, and partly to the base model being only Wan2.1-1.3B.

### Separate Handling of Temporal Sync vs. Semantic Sync (temporal alignment and content semantic matching as two independent filter conditions) ⚠️

UniVerse-1 does not treat temporal sync and semantic sync as two independent filter conditions; in practice it only covers one special case of temporal sync (lip movement), and the semantic-sync dimension is entirely absent:
【Temporal sync】Only covered via SyncNet conf > 2.0 for the lip-audio temporal correspondence of speech clips; the non-speech data's event-level temporal alignment (e.g., "the instant a door closes corresponding to the impact sound") has no detection or filtering whatsoever.
【Semantic sync】Entirely absent — there is no ImageBind-style cross-modal embedding-similarity filter, no judgment of "whether the visual content and audio content are semantically matched," and no use of a VLM/LLM to adjudicate audio-visual semantic self-consistency. This means samples with semantic mismatches — such as "the frame shows a kitchen but the audio track is background music/voice-over narration" — cannot be identified and removed. Given that the data sources include large amounts of vlog and tutorial-type content (where post-production music and narration are extremely common), this is a substantive data risk.
【Comparative reference】MOVA explicitly splits Stage 2 alignment evaluation into two orthogonal dimensions — temporal (SynchFormer) and semantic (ImageBind) — with separate thresholds for each; UniVerse-1 has no independent facility for either, and the semantic dimension is completely blank.
【Implicit substitute mechanism】The paper's implicit claim is: as long as "native synchronized audio track + online-annotated window sharing the same source" is guaranteed, temporal and semantic consistency are already assured at the data source and require no post-hoc filtering. This holds when the native audio track genuinely corresponds to the imagery, but does not hold for content with post-production music/narration/sound effects. The paper does not discuss the boundaries of this assumption. [Uncertain]

### Audio Quality Filtering (SNR, silence detection and silence-ratio threshold, no-audio-track removal, off-screen audio-source removal, background-music separation) ⚠️

Audio-side filtering occupies only one stage in the funnel, and uses only signal-level, hand-crafted features, with no learned audio-quality assessment at all:
【Audio activity detection (the only audio-filtering step)】Three signal-level statistics are analyzed — volume, energy, and zero-crossing rate — used to identify and discard silent segments. The paper gives no specific threshold values. [Uncertain]
【No-audio-track removal】The very first step of the funnel discards videos lacking an audio track, the most front-loaded veto.
【Audio filtering not performed】
- No SNR/signal-to-noise-ratio threshold;
- No audio aesthetic scoring (MOVA uses Audiobox-aesthetics' triple scores of PQ>5.0 / CU>4.5 / CE>2.5; UniVerse-1 has no corresponding facility at all);
- No bandwidth lower-bound constraint;
- No silence-ratio proportional threshold — only a binary "is it silent" exclusion is performed, not a continuous "excessive silence ratio" judgment;
- No off-screen audio/narration-source removal;
- No background-music separation (source separation) — background music in speech clips is not stripped out, and is trained together with the voice as-is;
- No detection of clipping, distortion, popping, or other recording defects.
【Sample-rate handling】Uniformly resampled to 25.6 kHz (downgraded from Ace-step's native 44.1 kHz), for the purpose of aligning with the 25 fps video rather than for quality reasons — but this substantively lowers the upper bound of audio bandwidth (Nyquist frequency 12.8 kHz), causing loss of high-frequency detail (such as cymbals, sibilance), an audio-quality cost paid for the sake of cross-modal alignment. The paper does not discuss the impact of this trade-off.
【Overall assessment】Audio quality gatekeeping is the weakest link in UniVerse-1's data pipeline, consistent with its overall approach in which "the audio expert base comes from the music-generation model Ace-step, and audio capability is mainly inherited from the base model" — the data side performs no deep audio-quality screening, relying instead on the audio prior already present in the base model.

### Classification and Separate Handling Strategies for Speech/Sound-Effects/Music

The classification and handling strategy for the three audio categories of speech, sound effects, and music is as follows:
【Classification mechanism】Only a single binary gate — Whisper ASR detecting whether recognizable speech is present in a clip:
- Speech detected → enters the speech subset, undergoes further RetinaFace face detection and SyncNet conf>2.0 verification, and upon passing is explicitly tagged "contains speech," ending up at 1,187 hours;
- No speech detected → not eliminated, downgraded into "general-purpose audio-video data," ending up at 3,074 hours.
No dedicated audio-classification model is used (compare MOVA's use of the self-supervised audio Transformer EAT for speech/non-speech classification), and no further distinction is made within non-speech content among sound effects, music, and ambient sound — these three categories are merged into a single "general/ambient" bucket in UniVerse-1's data taxonomy.
【Sources of the three capability divisions】
- Speech and lip-sync capability: derived from the 1,187-hour, triple-verified speech subset, and the architectural decision to remove the speaker encoder for generalization;
- Sound-effect/ambient-sound capability: derived from the 3,074 hours of general data + 3,422 hours of VGGSound/AudioSet (the latter being an event-audio annotation dataset introduced specifically to bolster sound effects);
- Music/instrument-sound capability: mainly inherited from the base model Ace-step (itself a music-generation model), reinforced on the data side by music-variety and classical-performance material, but with no separate hour count given.
This is a three-part division of labor of "music via base model, sound effects via public dataset, speech via self-collected fine screening" — similar in approach to MOVA's two-part division ("sound effects/music injected during audio-tower pretraining, speech reinforced during joint training"), but UniVerse-1 entirely outsources the acquisition of music capability to the base model and performs no dedicated curation of music data itself.
【Category separation on the annotation side】In the caption schema, speech content and ambient-audio description are two independent fields, so the model can distinguish "what was said" from "other sounds" at the text-condition level, but the ambient-audio field is not further subdivided into sound effects/music/ambient sound internally.
【Differentiated loss】VGGSound/AudioSet, due to their low visual quality, are subjected to LQLS (Flow Matching loss computed only in the high-noise segment where t>800) — the only training treatment differentiated by data source, essentially "letting low-quality data teach only semantics, not detail."

## Training Coordination

### Multi-Stage Training Curriculum and Data-Curriculum Scheduling (basis for stage division: resolution/duration/quality score/modality; low-res→high-res, image→video, short→long)

UniVerse-1 has no multi-stage training curriculum — this is one of the most notable structural differences from comparable works:
【Single training stage】All 7,685 hours of data are mixed and trained in a single stage, for a total of 50k steps, effective batch size 128, gradient accumulation 4 steps, AdamW optimizer, learning rate 5e-6 (constant, with no warmup or decay schedule mentioned), FSDP multi-node distributed training. There is no resolution curriculum (no low-res→high-res), no duration curriculum (no short→long), no quality curriculum (no coarse-screen→fine-screen), and no modality curriculum (no image→video).
【Curriculum replaced by "expert inheritance"】The fundamental reason for having no curriculum is the SoE paradigm itself: single-modality capability for video and audio has already been thoroughly established by Wan2.1 and Ace-step in their respective pretraining, and UniVerse-1 only needs to learn "cross-modal alignment," so it has no need to build up visual capability starting from low resolution. This can be seen as front-loading the cost of curriculum learning onto the two base models — the core selling point of SoE is precisely to "bypass training from scratch" for improved training efficiency.
【The only "grading" occurs not on the time axis but on the noise axis】The LQLS strategy applies differentiated loss by diffusion timestep, based on data source: VGGSound/AudioSet only participate in Flow Matching loss computation when t>800 (out of 1000 total steps). This effectively resembles a "noise-level curriculum": low-quality data contributes supervision only during the coarse-structure stage, while high-quality data covers the full noise range. This is an interesting design that substitutes the noise dimension for the time dimension in data grading, but it holds constant throughout the entire training process and does not vary as training progresses, so it does not constitute curriculum scheduling in the traditional sense.
【Comparison with MOVA】MOVA achieves a "decreasing scale, increasing quality" pyramid curriculum via three stages (61.5k→37.6k→11k hours, 360p→360p→720p); UniVerse-1 instead uses flat single-stage training + data grading on the noise axis. The former suits gradual ascent from a relatively weak base model, while the latter suits a scenario where a strong base model already exists and only needs alignment.

### Data-Mixture Change Across Training Stages (pretraining/annealing/SFT high-quality subset) ⚠️

Since there is only a single training stage, the dimension of "mixture changing with stage" does not exist. The data mixture is static and constant throughout:
【Fixed mixture (by hours)】Speech-centric data 1,187 hours (15.4%) : general-purpose audio-video 3,074 hours (40.0%) : VGGSound+AudioSet 3,422 hours (44.5%). The paper does not state whether training sampled according to this natural ratio, or applied up-/down-sampling weighting to some subset. [Uncertain]
【No annealing stage】The paper describes no operation of switching to a high-quality subset at the end of training.
【No SFT stage】There is no independent supervised fine-tuning stage — the entirety of UniVerse-1's training is itself a single fine-tuning of the two base models, with no second round of refinement afterward.
【The only mixture-adjustment mechanism is LQLS】Rather than adjusting the sampling ratio, the influence of the 44.5% low-quality data (VGGSound/AudioSet) on visual detail is reduced by restricting it to contribute loss only when t>800. This is equivalent to applying a piecewise weighting function to this subset — zero weight in the low-noise segment, weight one in the high-noise segment — a kind of "soft mixture by noise segment" rather than a "hard mixture by sample count." The ablation proves this mechanism is effective: removing LQLS drops the aesthetic score from 0.47 to 0.44 and ID consistency drops sharply from 0.89 to 0.78, showing that visual artifacts in low-quality data do indeed contaminate generation quality when not isolated.

### Post-Training Data (SFT curated-set size and selection criteria, number and annotation method of preference pairs, reward-model training data)

No post-training data. UniVerse-1 performs no post-training after the 50k-step joint fine-tuning:
- No SFT curated set (the entire training run is a single fine-tuning, with no second-round refinement);
- No RLHF / DPO preference alignment — no preference pairs were constructed, and no reward model was trained;
- No human preference-annotation data, and no Arena-style subjective evaluation (compare MOVA, which constructed a 732-item bilingual Chinese-English Arena evaluation set);
- No distillation or acceleration-oriented post-training (no mention of step distillation, CFG distillation, etc.).
Evaluation relies entirely on objective automatic metrics on Verse-Bench (MS/AS/ID/FD/KL/CLAP/LSE-C/AV-A/WER), with no subjective human-evaluation step. This is consistent with the paper's self-positioning as "an initial exploration" of unified audio-video generation and with its compute-constrained reality.

### Data-Processing Infrastructure and Throughput (NeMo Curator/Data-Juicer/in-house, GPU acceleration ratio, processing scale, cost) ⚠️

The paper discloses very little about data infrastructure, but its online annotation architecture itself is an infrastructure design worth recording:
【Online annotation service】A dedicated server process running concurrently with training, responsible for completing, on the fly at each sampling step: fixed-length window time sampling → QWen2.5-Omni multimodal annotation + Whisper ASR → text encoding (umT5) → video 3D VAE encoding → audio mel-spectrogram + Music-DCAE latent encoding, finally feeding the aligned "data-annotation pair" to the training process. This effectively merges annotation, encoding, and data loading into a single online pipeline, with the training side directly consuming latents.
【Engineering implications】This architecture requires the throughput of annotation and encoding to match the training consumption rate, or it becomes a bottleneck; it also eliminates the storage overhead of offline-precomputed latents (the latent cache for 7,685 hours of video would be considerable), but pays the cost of annotation and encoding repeatedly every epoch. The paper does not disclose this service's deployment scale, number of GPUs, per-sample latency, throughput ceiling, or whether there is a caching mechanism to reuse windows that have already been annotated. [Uncertain]
【Training infrastructure】FSDP distributed training across multiple nodes; GPU model, GPU count, total training duration, and GPU-days are not disclosed, and no mention is made of context parallelism or other long-sequence parallelism strategies (constrained by the 1.3B base model and short clips, sequence-length pressure is smaller than MOVA's).

## Effectiveness Comparison

### Quantitative Impact of Data-Strategy Ablations (distinguish: filtering-strictness ablation / caption-density-and-style ablation / data-mixture ablation, and corresponding evaluation metrics) ⚠️

The paper's ablation experiments (Table 2) target two training strategies rather than data composition itself, so in the strict sense there is only one "data-strategy ablation" (LQLS, which can be viewed as a mixture/grading ablation by data quality), with no filtering-strictness ablation and no caption-density/style ablation:
【Ablation 1: LQLS (low-quality-data loss strategy) — data mixture/grading ablation】
For the 44.5% of low-visual-quality data (VGGSound/AudioSet), Flow Matching loss is applied only when the diffusion timestep t>800 (out of 1000 total steps).
- Without LQLS: aesthetic score AS 0.47→0.44 (decrease), ID consistency 0.89→0.78 (sharp decrease of 11 points), Motion Score 0.20→0.38, KL 2.70→2.84, CLAP 0.16→0.15, AV-A 0.23→0.28 (worse).
- Conclusion: if low-quality data is not isolated by noise segment, it significantly contaminates visual fidelity and identity consistency; the drop in ID from 0.89 to 0.78 is the single largest degradation across all ablations. This provides quantitative support for the alternative route of "large-scale low-quality data + loss isolation" in place of "direct filtering" — that is, when data quantity is insufficient, restricting the scope of low-quality data's supervisory role is preferable to discarding it.
【Ablation 2: INSS (Independent Noise Sampling Strategy) — training-strategy ablation, not a data ablation】
- Without INSS: FD 1.25→1.43, KL 2.70→3.51 (sharply worse), CLAP 0.16→0.11 (sharp drop), LSE-C 1.34→0.99 (significant lip-sync degradation), Motion Score 0.20→1.10 (anomalous spike, co-occurring with AS 0.47→0.43 and ID 0.89→0.75 also dropping, indicating visual instability).
- Conclusion: the spurious cross-modal noise correlation caused by video and audio sharing PRNG state severely harms both audio quality (FD/KL) and cross-modal alignment (CLAP/LSE-C), and is the single most broadly impactful of the three configurations.
【Missing data ablations】No ablation validates the contribution of any of the following: the strictness of the 1080p resolution threshold, the tightness of the DOVER 0.6 threshold, the tightness of the SyncNet conf 2.0 threshold, whether the three-way data mixture (15.4/40.0/44.5) is optimal, or a controlled comparison of online vs. offline annotation (this is the paper's core claim, yet it is not quantitatively ablated, only qualitatively argued) — the absence of an online-annotation ablation is the most obvious weakness in this work's argumentation. [Uncertain]

### Evidence for Quality vs. Quantity (cases of small-but-refined data surpassing large-but-messy data) ⚠️

UniVerse-1 provides evidence pointing in the opposite direction — what it demonstrates is that "a small base model + moderate-scale data can still produce a usable joint-generation system," but its internal evidence actually supports "low-quality data need not be discarded, isolated use adds value," which is not consistent with the classic "small-but-refined beats large-but-messy" narrative:
【Evidence supporting quality-first】
- The admission-filtering bar is set aggressively (≥1080p, bitrate ratio ≥600, DOVER≥0.6), and this high bar is maintained even though the final output is only 7.6k hours, indicating the team leans toward quality in the quality-vs-quantity trade-off;
- After triple verification ("has audio track + face + SyncNet"), the speech subset ends up at only 1,187 hours, 15.4% of the total — a typical case of high-cost fine screening;
- The SoE paradigm itself is a case of "using high-quality pretrained experts in place of large-scale training from scratch" — at 7,685 hours, this is two orders of magnitude below the million-hour-scale data that training a video-generation model from scratch would require, relying on base-model quality rather than data quantity.
【Evidence pointing the other way (more noteworthy)】The LQLS ablation precisely demonstrates that the 44.5% of low-quality data (VGGSound/AudioSet) should not be discarded, but rather safely utilized by restricting its role within the diffusion process — retaining it and applying LQLS raises AS from 0.44 to 0.47 and ID from 0.78 to 0.89. This suggests a data-utilization paradigm different from "filtering means discarding": converting the binary decision of "use it or not" into the continuous decision of "which noise segment to use it in." In scenarios where total data volume is limited, this may be more valuable than aggressive filtering.
【Evidence not provided】The paper does not run a direct comparison of "small-scale finely-screened data vs. large-scale coarsely-screened data," nor does it provide a data-scaling curve, so no quantitative conclusion about the quality-quantity trade-off can be given. The Limitations instead list "larger-scale, more refined data curation" as future work, i.e., they consider there is still room for improvement on both fronts. [Uncertain]

### Alignment Between Training-Data Domain Distribution and the Evaluation-Benchmark Category Taxonomy (e.g. VABench's seven major categories)

UniVerse-1 releases both training data and an evaluation benchmark simultaneously, and the taxonomies of the two show partial alignment but not a strictly designed correspondence:
【Verse-Bench structure】600 image-text prompt pairs, in three subsets:
- Set1-I: 205 items, image sources being AI-generated images, web-scraped images, and media screenshots — testing generalization to synthetic/diverse visual inputs;
- Set2-V: 295 items, real video clips from YouTube and Bilibili, with captions generated by an LLM + Whisper ASR, subsequently human-verified — the only portion with human quality control, covering bilingual Chinese-English platforms;
- Set3-Ted: 100 items, drawn from TED Talks from September 2025 — specifically testing speech generation and lip sync in speaking scenarios.
【Verse-Bench audio category distribution (nine categories)】Natural environment 36.1%, music and instruments 19.3%, daily life 10.2%, human voice 7.6%, transportation 4.5%, industrial and urban 3.9%, weapons and explosions 2.5%, special effects sound 2.3%, unclassified 15.8% (the appendix additionally has a more fine-grained sub-category table).
【Alignment relationship with training data】
- Points of strong alignment: the evaluation set's human voice accounts for only 7.6%, and the training set's speech data accounts for 15.4% — both are "non-speech-dominated," a consistent orientation; music and instruments account for 19.3% in the evaluation set, and the training side has corresponding music-variety and classical-performance material plus the Ace-step base model, also corresponding;
- Points of weak or no alignment: the training data has never been classified, tallied, or mixture-controlled according to these nine audio categories — the training-side classification is only the binary "speech/non-speech," whereas the evaluation side has a nine-category breakdown. This nine-category taxonomy was therefore independently designed for evaluation, rather than derived from the training-data mixture. Categories such as weapons and explosions (2.5%) and special effects sound (2.3%) have no corresponding source explanation whatsoever in the description of the training data;
- Set3-Ted directly corresponds to the "public speeches, interviews" subject matter in the training data, making it the clearest of the three subsets in terms of training-evaluation correspondence.
【Evaluation task coverage】Verse-Bench supports four task types: joint audio-video generation, audio-to-video, video-to-audio, and text-to-speech. This multi-task design directly benefits from its factorized three-field caption schema (any subset of conditions can be masked to construct different task forms), representing a genuine structural alignment between the data schema and the evaluation design.
【Conclusion】UniVerse-1's training-data distribution and evaluation-category taxonomy are "same-origin but not deliberately aligned" — the evaluation taxonomy is finer-grained and more complete, while the training side remains at a coarse-grained binary classification, lacking a mixture-control mechanism organized by category.

## Uncertain Fields

The research information for the following fields is partly uncertain (source of the ⚠️ marker):

- provenance_licensing
- resolution_aspect_distribution
- domain_distribution
- language_accent_distribution
- funnel_retention_rate
- quality_filtering
- motion_filtering
- deduplication
- model_as_data_judge
- safety_filtering
- caption_model
- caption_structure
- dialogue_transcription_attributes
- geometric_structured_annotation
- sync_metric_and_threshold
- temporal_vs_semantic_sync
- audio_quality_filtering
- stage_data_mixture
- data_infra_throughput
- data_ablation
- quality_over_quantity_evidence
