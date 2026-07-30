# LongCat-Video (Meituan LongCat team's video generation foundation model; technical report arXiv:2510.22200; derivative models in the same family, LongCat-Video-Avatar and LongCat-Video-Avatar 1.5, technical report arXiv:2605.26486)

> Topic: Data processing for video generation models (including joint audio-video generation): data cleaning pipeline, data distribution, annotation approach

[← Back to home](../index.md)

**Contents**: [Basic Information](#basic-information) · [Data Scale and Distribution](#data-scale-and-distribution) · [Cleaning Pipeline](#cleaning-pipeline) · [Captioning Approach](#captioning-approach) · [Audio-Video Alignment](#audio-video-alignment) · [Training Coordination](#training-coordination) · [Effectiveness Comparison](#effectiveness-comparison)

## Basic Information

### Name

LongCat-Video (Meituan LongCat team's video generation foundation model; technical report arXiv:2510.22200; derivative models in the same family, LongCat-Video-Avatar and LongCat-Video-Avatar 1.5, technical report arXiv:2605.26486)

### Releasing institution/company

Meituan LongCat team. The technical report is authored under the name "Meituan LongCat Team," with contributors including Xunliang Cai, Qilong Huang, Zhuoliang Kang, Hongyu Li, Shijun Liang, Liya Ma, Siyu Ren, Xiaoming Wei, Rixu Xie, Tong Zhang, and others.

### Release date (technical report/paper/open-source date)

LongCat-Video base model: arXiv v1 submitted October 25, 2025 (2510.22200), v2 on October 28; official public release and open-sourcing occurred around October 27, 2025. Derivative versions: LongCat-Video-Avatar released December 23, 2025 (Meituan's technical team blog published a post the same day); LongCat-Video-Avatar 1.5 released May 2026 (arXiv:2605.26486).

### Type (model/dataset/toolchain/evaluation benchmark)

Model (a video generation foundation model + companion open-source inference code/operators). A 13.6B-parameter Diffusion Transformer; a single model uniformly supports three task types — text-to-video (T2V), image-to-video (I2V), and video continuation — and it also open-sources forward/backward implementations of Block Sparse Attention (which can be regarded as a toolchain component). Not a dataset, not an evaluation benchmark (though the report does include a self-built internal evaluation set).

### Degree of openness (whether weights/code/data/pipeline are each open source)

High degree of openness for weights and code; the data side is fully closed.
【Open】(1) Model weights: Hugging Face meituan-longcat/LongCat-Video (13.6B), plus the subsequent LongCat-Video-Avatar (wav2vec2 audio encoder) and LongCat-Video-Avatar-1.5 (Whisper-large-v3 audio encoder); (2) Inference code: GitHub meituan-longcat/LongCat-Video, including multiple inference modes and a Streamlit interactive interface; (3) The forward and backward implementations of the Block Sparse Attention operator are open-sourced alongside the base model; (4) The technical report publicly discloses the architecture, the training-stage table, and RLHF details.
【License】MIT License (weights and code), permitting commercial use — one of the most permissive licenses among video generation models of comparable scale.
【Not open】The training data itself, the data-processing pipeline code, the internal captioner (a fine-tuned LLaVA-Video), the internal aesthetic/blur/watermark scoring models, the internally fine-tuned VideoAlign reward model, and all data-scale statistics. Training code is also not open-sourced (only inference code is).

### Whether joint audio-video generation is supported, and the implementation approach (native joint/cascade/MoE fusion)

The base version of LongCat-Video does not support joint audio-video generation — it is a purely visual generation model (T2V/I2V/video continuation), producing output with no audio track.
The same-family LongCat-Video-Avatar / Avatar 1.5 are "audio-driven video generation" models (Audio-Text-to-Video, AT2V, and Audio-Text-Image-to-Video, ATI2V), where audio is a conditioning input and video is the output — a one-directional drive rather than joint audio-video generation (no audio is generated). The implementation approach injects audio features into the same 13.6B DiT backbone: Avatar uses a wav2vec2 audio encoder, while Avatar 1.5 upgrades to a Whisper-Large encoder to improve lip-sync precision, and supports both single-stream and multi-stream (multi-person) audio input. Therefore, under the "native joint/cascade/MoE fusion" taxonomy, this family does not belong to any of these categories and should instead be classified as "audio-conditioned video generation."

### List of research information sources (URLs of papers/technical reports/official documentation/news, with each source's nature annotated: official first-hand/same-team corroboration/third-party report)

1. arXiv:2510.22200 "LongCat-Video Technical Report" https://arxiv.org/abs/2510.22200 and full text https://arxiv.org/html/2510.22200v2 — official first-hand (the core basis; the data-processing section, training-stage table, and RLHF details all come from this source).
2. Official GitHub repository https://github.com/meituan-longcat/LongCat-Video — official first-hand (scope of open-sourcing, MIT license, model variants, and release timeline).
3. Hugging Face model card https://huggingface.co/meituan-longcat/LongCat-Video — official first-hand (license, capability description, MOS scores).
4. arXiv:2605.26486 "LongCat-Video-Avatar 1.5 Technical Report" https://arxiv.org/abs/2605.26486 — official first-hand (same-team corroboration; the details on audio-video alignment/audio filtering/emotion and silence data curation are drawn primarily from this paper).
5. Meituan technical team blog https://tech.meituan.com/2025/12/23/longcat-video-avatar.html and https://tech.meituan.com/tags/longcat.html — official first-hand (Chinese-language release framing).
6. Meituan press release "LongCat-Video Anchors on Long Video, Building a World-Model Technology Foundation" https://www.meituan.com/news/NN251205166001020 — official first-hand (positioning and world-model narrative).
7. 36Kr report https://www.36kr.com/p/3527169453464452, Zhihu analysis https://zhuanlan.zhihu.com/p/1966806796392966062 — third-party reports (corroborating release timing and the inference speedup factor, etc.).
8. HyperAI paper page https://hyper.ai/en/papers/2605.26486 — third-party index.

## Data Scale and Distribution

### Training data scale (video count/hours/token count, pretraining vs. SFT broken out separately) ⚠️

[Uncertain]. The technical report does not disclose any training data scale anywhere in the full text — no video count, no total duration (hours), no token count or image count, and the scale of pretraining vs. SFT is not given either. The only things that can be indirectly inferred are training iteration counts and batch configurations: the five pretraining stages total approximately 678k iterations (Stage 1 T2I 256p 285k + Stage 2 140k + Stage 3 164k + Stage 4 36k + Stage 5 53k); the SFT stage is 7.5k iterations (480p+720p × 93 frames, lr 1e-5); the RLHF (GRPO) stage is approximately 0.5k iterations (group size 4, 64 prompts per step). Avatar 1.5 likewise does not disclose total data volume (Stage 1 256p × 93 frames, batch 64, 130k iterations total; Stage 2 480p × 93 frames, batch 32, 45k iterations total). The evaluation-set size is the only data point given with exact numbers: the internal T2V set has 1228 items (500 for human evaluation + 728 for automatic evaluation, covering 48 categories), and the I2V set has 400 items (100 first-frame reference images × 4 prompt categories).

### Data source composition (proprietary/public datasets/web crawling/licensed procurement/synthetic data) ⚠️

The report states only in general terms that "We collect raw video data from a variety of sources," without breaking down the proportions of proprietary business data / public datasets / web crawling / licensed procurement, and without naming any specific dataset. It is confirmed to include both image data and video data (Stage 1 is pure T2I image training; from Stage 2 onward images and video are mixed). During the SFT stage, "specialized datasets" were additionally incorporated to strengthen instruction-following, targeting two specific categories — camera motion and visual style — with the sources unspecified.
The same-family Avatar 1.5 discloses six functionally divided categories of data sources, which can serve as corroborating evidence for the team's data-organization approach: (1) close-up facial videos (for facial modeling and lip motion); (2) interview-style videos (stable subject, clear speech); (3) performance videos (providing cinematographic language and pose variation); (4) interaction videos (object manipulation and gestures); (5) music videos (singing and rhythmic motion); (6) animation and stylized videos (for generalization to non-photorealistic domains). The proportion of each source is not disclosed. [Uncertain: specific source composition and mixture ratios]

### Data compliance and provenance (proportion of licensed data, rights-cleared datasets, C2PA, etc.) ⚠️

[Uncertain]. The technical report does not touch on data compliance and provenance issues at all: there is no statement of the proportion of licensed data, no declaration of rights-cleared datasets, no description of C2PA or any content-provenance/watermarking scheme, and no discussion of copyright licensing sources. This is consistent with the general practice among domestic (China-based) vendors' technical reports (compliance details are not written into the paper). Note that the model weights themselves are released under the MIT license permitting commercial use, but this does not constitute any claim about the legality of the training data's provenance.

### Clip duration distribution and splitting strategy ⚠️

The report does not give a statistical distribution of clip durations (no average duration, no duration histogram). The splitting strategy is clearly stated: scene detection is used to cut long videos into "training-friendly clips while maintaining content consistency," using a combination of PySceneDetect and an in-house-trained TransNetV2. After splitting, a duration field is recorded in the metadata and used for subsequent filtering, but the filtering threshold is not disclosed. The target duration is fixed at the architecture level: all five pretraining stages, plus SFT and RLHF, use a video length of 93 frames (about 3.1 seconds at 30fps). Minute-scale long videos are not achieved by generating a long segment in a single pass, but rather via the Video-Continuation task chaining outward extrapolation — i.e., recursively continuing generation using multiple preceding frames as conditioning frames. In Avatar 1.5's online filtering, "duration" is one of the explicit stage-by-stage filter conditions. [Uncertain: duration distribution values and filtering thresholds]

### Resolution/aspect-ratio distribution and bucketing strategy ⚠️

The report does not give a statistical distribution of the raw data's resolution/aspect ratio, nor does it describe an aspect-ratio bucketing mechanism. At the metadata level, resolution, frame rate, and bitrate are recorded and used for filtering. What is clearly established is the resolution curriculum on the training side: 256p (Stage 1, Stage 2, Stage 3) → 480p (Stage 4) → mixed 480p+720p (Stage 5, SFT, RLHF) — i.e., a progressive low-to-high resolution ramp-up, with Stage 5 onward mixing two resolution tiers within the same stage so the model adapts to both 480p and 720p output simultaneously. The final delivered inference resolution is 720p/30fps. In addition, the model adopts a "spatiotemporal dual-axis coarse-to-fine" generation strategy: a coarse video at low resolution and low frame rate is generated first, then refined — a strategy that requires the training data to support multiple resolution tiers simultaneously. The VAE is the WAN2.1 VAE, with a spatiotemporal compression ratio of 4×8×8, which combined with patchify yields an overall ratio of 4×16×16. [Uncertain: raw data resolution distribution and bucketing details]

### Category/domain distribution and mixture strategy (proportional control and concept balancing across people, actions, scenes, styles, etc.)

This is the relatively most distinctive aspect of LongCat-Video's data work, employing an unsupervised taxonomy of "caption text-embedding clustering + LLM semantic naming" rather than a manually predefined label system. Specifically: text embeddings are computed for all video captions, clustering is performed in the embedding space, and an LLM is then used to induce and name a category for each cluster, yielding content-type labels (the report gives examples such as personal interactions, artistic performances, natural landscapes, etc.). This taxonomy serves three purposes: (1) observing the data distribution to identify over- or under-represented categories; (2) performing "targeted data supplementation or rebalancing"; (3) supporting "dynamic and precise allocation of data subsets tailored to specific requirements and objectives of different training phases."
Going further, the SFT stage directly uses this embedding space as a sampling weight: samples are "selected inversely proportional to their density in the caption embedding space" — an explicit concept-balancing mechanism whereby high-density common concepts are downsampled and long-tail rare concepts are relatively upweighted, so that the curated set's semantic coverage becomes more uniform.
In addition, two domains are singled out for special reinforcement: camera motion and visual style, with the SFT stage specifically introducing corresponding specialized datasets to strengthen instruction-following in these two dimensions.
Not disclosed: the number of clustering clusters, the specific proportion figures for each category, and the target mixture ratios after rebalancing. [Uncertain: specific proportion values for each category]

### Audio category distribution and mixture (proportions and control strategy for speech/sound-effects/foley/music/ambient sound/silence) — a dimension unique to AV models ⚠️

The base version of LongCat-Video has no audio modality, so this dimension is not applicable.
The same-family LongCat-Video-Avatar 1.5 does involve audio, but its audio processing serves "as a driving condition" rather than as a generation target, so there is no generation-side mixture design for speech/sound-effects/music/ambient-sound/silence. The audio-related data stratification that can be summarized is: (1) speech is the overwhelming majority category (both the close-up-face and interview source categories center on clear speech); (2) singing/music is listed as a separate category (from the music-video source, used to cover singing lip motion and rhythmic movement); (3) silent (non-speaking) data is specifically constructed as an independent subset — using a dual-model consistency mechanism of Qwen3-Omni for an initial judgment plus Qwen3-VL for a secondary review, retaining a clip only when both models judge that the subject is not speaking, so that the model learns the scenario of "audio present but subject not speaking"; (4) intervals with concurrent multi-speaker activity are explicitly excluded. Proportions for each category are not disclosed. [Uncertain: specific proportion values for audio categories]

### Narrative-structure distribution (single-shot vs. multi-shot, average clip duration, shot-count distribution, whether native audio tracks are included)

After scene-detection splitting, the training data consists of single-shot clips — the purpose of the PySceneDetect + TransNetV2 combination is precisely to eliminate shot transitions, ensuring each training clip is internally continuous and consistent; the training set therefore does not include multi-shot narrative samples, and there is no concept of a shot-count distribution. Average clip duration is not disclosed, but at training time clips are uniformly sampled as 93 frames (about 3.1 seconds at 30fps). Native audio track: the base version completely discards the audio track during training (purely visual); the Avatar family, by contrast, relies on native audio tracks (audio and video must be paired and pass lip-sync verification).
Multi-shot/long-narrative capability is not solved at the data level but rather at the inference level via Video-Continuation recursive continuation: generation continues using multiple preceding frames as conditioning frames, combined with optimization targeting cross-frame temporal consistency and physical-motion plausibility, suppressing color drift, quality degradation, and motion discontinuity, thereby stably producing minute-scale long videos. In its GRPO stage, Avatar 1.5 supports multi-segment rollouts of up to 5 clips (only the final clip participates in optimization), which indirectly reflects that its long videos follow a multi-segment-stitching paradigm.

### Language/accent distribution (the data basis for multilingual lip-sync capability) ⚠️

[Uncertain]. The base version has no speech modality, but on the text side it clearly supports bilingual Chinese-English: the text encoder is umT5 (a multilingual encoder; the report explicitly states it supports both English and Chinese captions), and the caption-augmentation step includes Chinese-English mutual translation, so the distribution of training prompts covers both Chinese and English, with the specific ratio not disclosed.
The Avatar family involves genuine speech-driven input, but the technical report does not disclose any statistics on language or accent distribution, nor does it describe the data composition underlying multilingual lip sync; no ASR transcription model is specified.

## Cleaning Pipeline

### Overall structure of the filtering funnel (number of filtering stages, ordering of each stage)

The overall structure is a two-major-phase funnel of "preprocessing + annotation," with each phase further subdivided into several stages:
【Phase 1: Preprocessing】
1) Collection and deduplication: raw video is collected from multiple sources, deduplicated using source video ID and MD5 hashing;
2) Scene splitting: PySceneDetect combined with an in-house-trained TransNetV2 jointly perform scene detection, cutting long videos into content-consistent, training-friendly clips;
3) Black-border cropping: black borders are cropped out with FFMPEG during the transition-splitting process;
4) Compression and packaging: processed clips are compressed and packaged to support efficient data loading during training.
【Phase 2: Annotation】
5) Basic metadata annotation: duration, resolution, frame rate, bitrate, aesthetic score, blur score, text coverage, watermark detection;
6) Motion information annotation: optical flow is extracted to assess video dynamics;
7) Caption annotation: a fine-tuned LLaVA-Video generates the base description (+ Tarsier2 annotation to strengthen temporal understanding);
8) Cinematography and style annotation: a camera-motion classifier + Qwen2.5VL annotate shot size/lens type/realism/animation style/color tone;
9) Caption augmentation: Chinese-English mutual translation, generation of concise summaries, and random combination of cinematography and style labels spliced into the augmented caption;
10) Distribution analysis and allocation: caption text-embedding clustering + LLM category induction, used as the basis for data supplementation/rebalancing and for dynamically allocating data subsets across different training stages.
Notably, the report frames these annotations as an "annotate first, then threshold-filter" metadata-driven pattern — all quality signals are fully annotated into the database first, and subsets are then extracted by combining thresholds as needed for different training stages, rather than being hard-deleted in one pass. This "annotate into the database, filter on demand" design is precisely what underpins the "dynamic allocation of data subsets across different stages" mentioned above.
Avatar 1.5 upgrades this to a two-layer structure of "offline annotation + online clip-level verification": offline, it performs face detection and keypoints, audio-usability annotation, lip-sync verification, visual quality estimation, camera/motion classification, and time-segmented captioning; online, at training data-fetch time, it applies progressive stage-by-stage filtering in the order of audio synchronization → camera applicability → text and visual quality → duration → visual defects → motion consistency → mask-area constraint.

### Quantitative retention rate of the funnel (input/output volume and final retention rate at each filtering stage, e.g. Apollo's 27%) ⚠️

[Uncertain]. The report gives no input/output volume or retention rate for any single filtering stage, and no overall final retention-rate figure. This is one of the weakest points of data-side disclosure in this technical report — it enumerates in detail which filters were used, but gives no threshold value and no pass-rate figure for any single stage, making it impossible to make a horizontal quantitative comparison against something like Apollo's 27% funnel.

### Shot-splitting method (PySceneDetect/in-house model/shot-aware splitting)

A dual-path joint scene-splitting scheme is used: the open-source tool PySceneDetect together with an in-house-trained TransNetV2 model. TransNetV2 is a dedicated shot-boundary-detection neural network whose recognition of gradual transitions (fade/dissolve) is markedly better than PySceneDetect's threshold-based method; the team additionally retrained it on their own data. The splitting goal is to obtain "training-friendly clips while maintaining content consistency" — clips that are both suitable for training and internally consistent. During splitting, black-border cropping is performed simultaneously with FFMPEG (black border cropping during transition segmentation).

### Quality filtering (aesthetic scoring, clarity, OCR text filtering, black-border/watermark/logo detection) ⚠️

Quality signals are fully annotated as metadata and then filtered by threshold, with fairly comprehensive coverage:
(1) Aesthetic score — the scoring-model identity is not disclosed;
(2) Blur/clarity score — determines image sharpness and compression damage;
(3) Text coverage — detects the proportion of on-screen text/subtitles, used to eliminate samples with excessive subtitles or overlaid text (corresponding to common OCR filtering);
(4) Watermark detection — eliminates clips with watermarks/station logos/logos;
(5) Black-border handling — not a filter but a repair, cropped out directly with FFMPEG during splitting;
(6) Encoding quality — resolution, frame rate, and bitrate metadata can be used to eliminate low-resolution, low-bitrate poor-quality sources.
The SFT-stage curated set is further re-filtered on top of this using multiple metrics — aesthetic score, video quality, motion quality.
Avatar 1.5's visual-quality estimation explicitly targets three types of problematic content: "low-resolution, heavily compressed, subtitle-heavy," and the online stage additionally checks resolution, brightness distribution, black/white pixel proportion, and frame-jump detection.
All specific threshold values are undisclosed. [Uncertain: thresholds for each quality metric]

### Motion filtering (optical-flow/motion-score thresholds, removal of static and jittery content) ⚠️

Optical flow is used as the sole motion metric: "Motion information is evaluated using extracted video optical flow to assess video dynamics, enabling us to filter out clips with minimal motion features" — optical flow is extracted to assess dynamics, and clips with minimal motion features are filtered out accordingly (i.e., near-static samples are removed). The report only mentions removing the "low motion" side and does not mention whether high-frequency jitter/violent shaking samples are also removed, nor does it give an optical-flow score threshold. The SFT stage has an independent motion-quality metric involved in curated-set filtering (this metric is given by an internally fine-tuned VideoAlign model, distinct from the optical-flow statistic — it is a model-based score). Avatar 1.5's online filtering includes two checks — "motion intensity" and "motion consistency" — plus frame-jump detection. [Uncertain: optical-flow threshold value, whether jitter is filtered]

### Deduplication method (exact deduplication and embedding-based semantic deduplication recorded separately) ⚠️

Performed only at the first step of preprocessing, using a method that is clear but relatively basic: dual deduplication using source video ID (a source-side unique identifier) and MD5 hash. The former removes duplicate collections of the same video by source-side unique ID; the latter performs byte-level exact deduplication by file hash.
The report does not mention any embedding-based semantic/perceptual deduplication (e.g., CLIP/video-feature near-duplicate detection, pHash perceptual hashing), so there appears to be no explicit handling mechanism for near-duplicates such as "different encoded versions of the same material, different crops, or reposted/re-uploaded copies." However, the caption-embedding-space clustering and the "inverse-density sampling" used in the SFT stage do achieve an approximate semantic-deduplication effect — high-density redundant clusters are automatically downsampled. [Uncertain: whether there is additional undisclosed semantic deduplication]

### VLM/LLM as data quality judge (multimodal large-model quality scoring and mismatch removal; the 2026 trend of shifting from shallow scorers toward large-model semantic judgment)

This model makes extensive use of multimodal large models as annotators and quality judges, a core mechanism in its data pipeline, though it leans more toward "model annotation" than "model-based accept/reject judgment":
(1) Caption generation: a fine-tuned LLaVA-Video (fine-tuned using in-house synthetic pairs) serves as the primary captioner, with Tarsier2 annotation introduced to strengthen temporal understanding;
(2) Qwen2.5VL handles two categories of structured discrimination — shot-size and lens-type determination, and visual-style determination (realism, animation styles, color tones);
(3) Camera motion (pan/tilt/zoom) is handled by a dedicated trained classifier rather than a VLM;
(4) After caption-embedding clustering, an LLM performs category induction and naming, i.e., an LLM is used to define the data taxonomy;
(5) On the post-training side, the reward models are also, in essence, model-based quality judges: a Motion Quality model fine-tuned from the VideoAlign base (input is a grayscale video, forcing it to attend only to motion and not color) and a Text-Alignment model (input is a color video), plus HPSv3-general/HPSv3-percentile visual-quality scoring.
Avatar 1.5 further uses VLMs as hard filters and consistency judges: silence data uses Qwen3-Omni for an initial judgment plus Qwen3-VL for secondary review, retained only when both models agree; emotion data uses EmotiEffLib with a confidence threshold of s>0.7 for fine filtering, plus hard exclusion rules (samples containing synthetic content, more than two subjects, identity switching, or a subject occupying too small a proportion of the frame are all labeled null); the caption side uses Qwen3-Omni to generate a three-dimensional contextual description of "Spatial Environment / Interpersonal Relationships / Plot Progression," requiring the description to focus on objective physical manifestation.
The report does not state whether the base version used a VLM for explicit removal of caption-video mismatches.

### Safety and compliance filtering (NSFW, copyright, face/privacy) ⚠️

[Uncertain]. The technical report does not mention any NSFW filtering, violence/sensitive-content filtering, copyright filtering, or facial-privacy protection measures. Given that the model is open-sourced publicly and released by a major domestic (China-based) company, a content-safety review process almost certainly exists in actual production, but the report offers no textual basis for it, so its method and rigor cannot be confirmed. The only tangentially related item is that Avatar 1.5 lists "synthetic content" as a hard exclusion criterion for emotion data, but that is for annotation-reliability reasons rather than safety reasons.

## Captioning Approach

### Captioning model(s) used (in-house VLM/open-source model, model scale) ⚠️

A combination of "primary captioner + distillation-based enhancement + multiple specialized discriminative models" is used, rather than a single model:
(1) Primary captioner: LLaVA-Video fine-tuned on in-house synthetic pair data (fine-tuned LLaVA-Video model using in-house synthetic pairs), responsible for generating the base caption, covering both visual content and temporal aspects;
(2) Temporal enhancement: Tarsier2 annotations are incorporated into training/enhancement; Tarsier2 is a video-description model known for fine-grained temporal event description, used to compensate for static descriptions' insufficient depiction of action progression;
(3) Structured-attribute discrimination: Qwen2.5VL handles shot size, lens type, realism, animation style, and color tone; camera motion (pan/tilt/zoom) is handled by a separately trained lightweight classifier (rather than a VLM, presumably for cost and precision reasons);
(4) Category induction: an LLM (model unspecified) performs category naming on the caption-embedding clustering results.
The parameter scale of each model is not disclosed (LLaVA-Video commonly comes in 7B/72B tiers, Qwen2.5VL commonly in 7B/72B tiers; the report does not specify which tier is used).
Avatar 1.5's captioner is switched to Qwen3-Omni (fully multimodal, able to see both audio and video simultaneously). [Uncertain: specific parameter scale of each captioning model]

### Caption density and degree of structuring (short/long/dense descriptions, structured fields such as camera motion, style tags) ⚠️

A relatively high degree of structuring, using a synthesized "base description + attribute tags + random assembly" caption-construction paradigm:
(1) Base layer: natural-language description produced by LLaVA-Video/Tarsier2, covering both visual content and temporal actions;
(2) Structured-tag layer, in two groups —
   · Cinematography: camera motion (pan/tilt/zoom), shot size, lens type;
   · Visual style: realism, animation style, color tone;
(3) Augmentation layer: Chinese-English mutual translation (producing bilingual captions, paired with the bilingual umT5 text encoder), and generation of concise summaries (forming long and short caption tiers, so the model adapts to both brief and detailed prompts);
(4) Assembly layer: "randomly selecting elements from cinematography and visual style categories and integrating them with augmented captions" — randomly drawing several elements from the cinematography and visual-style categories and splicing them into the augmented caption. This random assembly is a key design: it lets the model see, during training, diverse prompt forms that "sometimes mention camera motion, sometimes don't," so that at inference time it can both respond to explicit camera/style instructions and avoid forcibly imposing a particular cinematographic style when the user doesn't mention one — this is the data basis for the SFT stage's instruction-following ability for camera motion and visual style.
The average caption length, token-count distribution, and the mixing ratio of long vs. short captions are not disclosed. [Uncertain: caption length distribution and long/short mixing ratio]

### Joint audio-video caption structure (whether visual + auditory tracks are covered simultaneously, whether they are split into separate fields, e.g. LTX-2's full-soundscape description, Script-a-Video's factorized streams, Foley-Omni's three fields)

Not applicable to the base version, which has no audio modality.
Avatar 1.5's caption scheme does use the fully multimodal model Qwen3-Omni, but its output is a set of three visual/narrative-dimension fields — "Spatial Environment, Interpersonal Relationships, Plot Progression" — requiring the description to focus on objective physical manifestation rather than subjective inference. This is a narrative-context schema serving audio-driven character video, not a genuine auditory-track soundscape description, and differs from true joint audio-video caption paradigms such as LTX-2's full-soundscape description or Foley-Omni's three fields. In addition, Avatar 1.5's offline stage also generates a "temporal-span caption" (a time-segmented caption) to describe local segments — this is fine-grained description along the time dimension, not joint description along the modality dimension.

### Dialogue transcription and speaker-attribute annotation (ASR transcription + speaker identity/language/accent/emotion) ⚠️

Not applicable to the base version.
Avatar 1.5 involves speaker processing, but does not use an ASR transcription route — the report does not specify any transcription model, and there is no verbatim text annotation. Its speaker-related processing focuses on "who is speaking" rather than "what is said": it uses two Active Speaker Detection models, TalkNet and UniTalk; for multi-person scenes, YOLOv6 detection + ByteTrack tracking is used to associate identity, and intervals with concurrent multi-speaker activity are explicitly excluded (excluding intervals with concurrent speaker activity), avoiding audio-visual attribution ambiguity. On speaker-attribute annotation, emotion is handled: 6 emotion categories are defined, filtered using EmotiEffLib with a confidence threshold of s>0.7, and samples containing synthetic content, more than two subjects, identity switching, or a subject occupying too small a proportion of the frame are uniformly labeled null. Language/accent attributes are not annotated. [Uncertain: whether there is an undisclosed ASR step]

### Geometric and structured annotation (camera parameters, depth, 3D point tracks, action labels, explicit state) ⚠️

Structured annotation is concentrated at the "cinematographic language" level rather than the geometric level: camera-motion categories (pan/tilt/zoom, determined by a dedicated classifier), shot size, and lens type (Qwen2.5VL) are annotated, but these are discrete semantic labels, not continuous camera extrinsics/intrinsics (no camera pose trajectories, no camera-pose numerical values). On the motion side, optical-flow statistics are annotated, which are aggregated quantities from a dense 2D motion field.
The report does not cover depth maps, 3D point tracks, point clouds, or action skeletons (pose/keypoints, for the base version) as geometric annotation.
Due to character-driven requirements, Avatar 1.5 performs face detection and facial landmark extraction to verify facial visibility, plus ByteTrack-based person-tracking boxes, and imposes a mask-area constraint (limiting the proportion of the frame occupied by the person mask) during training data-fetch — these are 2D structured annotations, still not 3D geometry. [Uncertain: whether there is undisclosed depth/pose annotation]

### Synthetic data construction (controlled perturbation/editing to construct training pairs, e.g. InstructAV2AV) ⚠️

Limited use. One clear instance is in training the primary captioner: LLaVA-Video is fine-tuned "using in-house synthetic pairs" — i.e., synthetic data is used to train the annotator model itself, not directly as training samples for the generative model.
On the video-generation side, there is no observed use of controlled perturbation/editing to construct training pairs (e.g., InstructAV2AV-style paired editing data construction). The training-data construction for the Video-Continuation task is a self-supervised style of conditional splitting — using the preceding N frames of a clip as conditioning frames and the subsequent frames as the prediction target, with the number of conditioning frames distinguishing the three task types (T2V has 0 conditioning frames, I2V has 1 reference frame, VC has multiple preceding frames) — this is a supervision signal automatically derived from real video, not synthetically generated new content.
In addition, the caption's "random assembly of cinematography and style tags" can be regarded as a form of prompt-side synthetic augmentation. [Uncertain: whether generative-model self-bootstrapped synthetic video data was used]

### Degree of human involvement (human annotation, human quality review, model pre-screening + human review) ⚠️

Human involvement on the training-data side is almost never mentioned — the data pipeline is described throughout as fully automated model-based annotation + threshold filtering, with no description of any human annotation or human review step.
Human involvement clearly exists at two downstream points:
(1) Evaluation: 500 prompts in the internal T2V evaluation set are reserved specifically for human evaluation, using a dual-track scheme of MOS (Mean Opinion Score) absolute scoring and GSB (Good-Same-Bad) relative comparison, with each video rated by three independent annotators; the I2V evaluation set of 400 items likewise includes human evaluation.
(2) Reward-model training: the Motion Quality and Text-Alignment reward models are both "VideoAlign-based model fine-tuned on internal annotated datasets/internally annotated data" — i.e., fine-tuned on internal human-annotated data, indicating an investment in human preference/quality annotation, but the annotation volume, annotation guidelines, annotator pool size, and consistency metrics are all undisclosed.
This can be summarized as: humans do not participate in data cleaning but instead inject human preference into training indirectly through "annotating the reward model." [Uncertain: scale of human-annotated data]

## Audio-Video Alignment

### Audio-video sync detection method (lip sync, event alignment)

Not applicable to the base version (no audio).
Avatar 1.5 treats lip-sync verification as an independent step in the offline annotation stage and as the first gate in online filtering: offline, lip-sync verification is performed, removing "samples with large audio-visual offsets"; in the online clip-level verification, "audio synchronization" is the first stage of the progressive filtering chain. The accompanying active-speaker-detection setup uses TalkNet and UniTalk jointly to ensure the audio track belongs to the target person on screen rather than an off-screen voice; multi-person scenes use YOLOv6 + ByteTrack tracking to associate identity, and intervals with concurrent multi-speaker activity are excluded. There is also a dedicated silence-data branch (dual-model consistency judgment by Qwen3-Omni + Qwen3-VL that the subject is not speaking) to cover the legitimate scenario of "audio present but no lip motion." Event-level (non-lip) audio-video alignment detection is not covered.

### Specific sync-detection metrics and thresholds (SyncNet/Synchformer/LSE/in-house, threshold values, e.g. MOVA's LSE-D≤9.5 and LSE-C≥4.5, SkyReels-V4's SyncNet |offset|≤3 ∧ conf>1.5) ⚠️

[Uncertain]. The Avatar 1.5 report only states qualitatively that it removes samples with "excessive audio-visual offset," without disclosing the sync metric used (no specific metric name such as SyncNet conf / sync-c / sync-d / AV-align is mentioned), and without giving any threshold value. TalkNet and UniTalk, used as active-speaker-detection models (outputting speaking/not-speaking plus confidence), likewise have undisclosed decision thresholds. The only place in the entire pipeline where an exact threshold is disclosed is the emotion-annotation step's EmotiEffLib confidence of s>0.7, which is unrelated to audio-video sync.

### Separate handling of temporal sync vs. semantic sync (temporal alignment and content-semantic matching as two independent filtering conditions) ⚠️

In Avatar 1.5's filtering chain, temporal alignment and semantic matching are indeed split into different steps, though this is not explicitly framed as "two orthogonal conditions serving the same goal": the temporal side comprises lip-sync verification (audio-visual offset detection) and active-speaker detection (determining the time interval to which an audio track belongs); the semantic/content side comprises the Qwen3-Omni + Qwen3-VL dual judgment of "whether the subject is speaking," plus the caption-level constraint of objective description for scene/relationship/plot. In the online filtering chain, "audio synchronization" is listed as the first stage and "text and visual quality" as a subsequent, separate stage — structurally reflecting this separation. Not applicable to the base version. [Uncertain: whether the team intentionally made this distinction]

### Audio quality filtering (SNR, silence detection and silence-proportion thresholds, no-audio-track removal, off-screen-voice-source removal, background-music separation) ⚠️

Not applicable to the base version.
Avatar 1.5's audio-side filtering description is relatively sparse: the offline stage has a dedicated audio-annotation step that "verifies whether a sample contains usable speech conditions," i.e., using "presence of usable speech" as the admission threshold; in source selection, material such as interview videos with "a stable subject, clear speech" is preferred — a form of source-level quality control. The report mentions the use of vocal separation, which can be used to separate the human voice from background music/ambient sound, but the model and parameters used are not specified. Off-screen audio sources (where the audio's owner is not visible on screen) are removed indirectly via active-speaker detection.
Not disclosed: SNR threshold, silence-detection and silence-proportion thresholds, the handling rule for audio-track-free samples, and audio sample-rate/bitrate requirements. [Uncertain: specific thresholds such as SNR, silence proportion, etc.]

### Classification and separate handling strategy for speech/sound-effects/music

Not applicable to the base version.
Avatar 1.5 applies coarse-grained functional stream separation to audio rather than a strict three-way classification of speech/sound-effects/music: (1) speech is the primary track, going through the full lip-sync + active-speaker-detection pipeline; (2) singing/music is listed as a separate stream (from the music-video source), used to cover singing lip motion and rhythmic body movement, which differs in lip dynamics from ordinary speech; (3) silence (non-speaking) is built as a separate subset, using dual-model consistency judgment to ensure the subject "truly is not speaking," so the model learns the scenario where audio is present but the subject does not vocalize; (4) intervals of concurrent multi-speaker overlap are excluded rather than retained as a category. Sound effects/foley and ambient sound are not treated as independent categories — consistent with its positioning: audio serves only as a driving condition, and the model does not generate audio, so there is no need to cover the full soundscape category set.

## Training Coordination

### Multi-stage training curriculum and data curriculum scheduling (basis for stage division: resolution/duration/quality score/modality; low-res→high-res, image→video, short→long)

Pretraining follows a strict five-stage progressive curriculum, scheduled along three simultaneous axes — modality (image→video), task (single-task→multi-task), and resolution (low-res→high-res) — while the duration axis is fixed at 93 frames throughout:
· Stage 1: T2I only, 256p, lr 1e-4, 285k iterations — using pure images first to establish visual priors and text alignment; accounts for about 42% of total iterations, the heaviest investment;
· Stage 2: T2I + T2V, 256p × 93 frames, lr 1e-4, 140k iterations — video is introduced, with images continuing to be mixed in to keep image quality stable;
· Stage 3: T2I + T2V + I2V + VC, all four tasks, 256p × 93 frames, lr 5e-5, 164k iterations — filling out all four task capabilities at low resolution first, with the learning rate halved;
· Stage 4: all tasks, 480p × 93 frames, lr 5e-5, 36k iterations — resolution is raised, iteration volume shrinks sharply;
· Stage 5: all tasks, mixed 480p + 720p × 93 frames, lr 2e-5, 53k iterations — dual-resolution mixed training, learning rate lowered further.
This is followed by SFT (480p+720p × 93 frames, lr 1e-5, 7.5k iterations) and RLHF/GRPO (480p+720p × 93 frames, approximately 0.5k iterations).
Overall, this presents a pyramid-shaped iteration allocation of "a large amount of low-res image groundwork → low-res all-task rollout → a small amount of high-res refinement → a very small amount of SFT/RL alignment." Task unification is achieved via "number of conditioning frames": T2V has no conditioning frames, I2V takes 1 reference frame, and VC takes multiple preceding frames, allowing all four task types to be trained together within the same batch. In addition, the generation side adopts a spatiotemporal dual-axis coarse-to-fine strategy (coarse first, then refine), which, combined with Block Sparse Attention and distillation, achieves roughly a 10.1× inference speedup.

### Changes in data mixture ratio across training stages (pretraining/annealing/SFT high-quality subset) ⚠️

The mechanism by which the mixture ratio changes across stages is clear, but numerical values are lacking:
(1) Modality mixture: pure images in Stage 1 → images and video mixed from Stage 2 onward, with image data continuing to be retained throughout all subsequent video-containing stages (T2I remains in the task list throughout), but the specific ratio of image to video data is not disclosed;
(2) Task mixture: from Stage 3 onward, T2I/T2V/I2V/VC are trained together in the same batch, with the sampling ratio among the four not disclosed;
(3) Resolution mixture: from Stage 5 onward, 480p and 720p are mixed, with the ratio between the two tiers not disclosed;
(4) Content mixture: the taxonomy derived from caption-embedding clustering supports "dynamic, precise allocation of data subsets across different training stages," and can be used to guide targeted data supplementation or rebalancing based on the clustering results — this is the core mechanism of mixture scheduling, but again without numerical values;
(5) Quality threshold tightening across stages: the SFT stage switches to a high-quality curated set (filtered by multiple metrics — aesthetic score, video quality, motion quality), layered with a "inverse-density sampling in caption embedding space" long-tail balancing strategy, while also introducing specialized datasets for camera motion and visual style.
The report does not describe a separate annealing stage; Stage 5's dual-resolution mixed training combined with progressively decreasing learning rates (1e-4→5e-5→2e-5→1e-5) plays an analogous role in effect. [Uncertain: specific numerical values for all mixture ratios]

### Post-training data (scale and filtering criteria of the SFT curated set, number of preference pairs and annotation method, reward-model training data) ⚠️

Divided into an SFT stage and an RLHF stage:
【SFT】The data is a "carefully curated, high-quality dataset," with filtering criteria at two layers — the first layer is multi-metric filtering (aesthetic score, video quality, motion quality); the second layer is distribution balancing: samples are selected inversely proportional to their density in the caption embedding space, achieving a relative upweighting of long-tail concepts. Specialized datasets are additionally merged in to strengthen instruction-following for camera motion and visual style. Training configuration: 480p+720p × 93 frames, lr 1e-5, 7.5k iterations. The size of the curated set is not disclosed.
【RLHF】GRPO (Group Relative Policy Optimization) online multi-reward reinforcement is used, rather than offline DPO, so there is no pre-annotated preference-pair dataset — preference signals are given online by the reward models during rollout. Configuration: group size 4, 64 prompts per step, approximately 0.5k iterations, task is T2V, resolution 480p+720p × 93 frames. Three reward models are used:
· Visual quality (VQ): HPSv3-general (average score over all frames) + HPSv3-percentile (taking the top 30% highest-scoring frames, combined with the video caption for computation) in a dual-path setup, with the latter used to suppress dilution of the overall judgment by a minority of low-quality frames;
· Motion quality (MQ): based on the VideoAlign backbone, fine-tuned on internally annotated data, taking grayscale video as input (color removed to force the model to evaluate only motion, uninfluenced by color/aesthetics);
· Text alignment (TA): also VideoAlign-based, fine-tuned on internally annotated data, taking color video as input.
The purpose of using multiple rewards together is to suppress reward hacking that a single reward would cause. The scale of the internal human-annotated data used to train MQ/TA is not disclosed.
Avatar 1.5's post-training further extends GRPO from video-level reward to per-frame reward modeling, introducing a hand-visibility check on the first frame to prioritize sampling of hand-containing samples, and multi-segment rollout supporting up to 5 clips with only the final segment participating in optimization; it also uses DMD2 to distill down to 8 steps (with both text and audio CFG set to 4.0). [Uncertain: SFT curated-set scale, reward-model annotation data volume]

### Data-processing infrastructure and throughput (NeMo Curator/Data-Juicer/in-house, GPU acceleration ratio, processing scale, cost) ⚠️

The report mentions only one sentence of engineering implementation at the tail end of preprocessing: processed clips are "compressed and packaged," for the purpose of supporting efficient training data loading — i.e., a packaged data format is adopted to avoid the I/O bottleneck of massive numbers of small files, but the specific format (e.g., webdataset/tar shard/an in-house format) is not stated.
Not disclosed: whether an open-source data-processing framework such as NeMo Curator or Data-Juicer, or an in-house platform, is used; GPU acceleration ratio; processing throughput (video-hours/day); cluster scale; and processing cost. Efficiency work on the training side (Block Sparse Attention, conditioning-token caching, distillation, together achieving roughly a 10.1× inference speedup) is disclosed in full detail, but that pertains to model-inference-side rather than data-processing-side infrastructure. [Uncertain: data-processing infrastructure and throughput]

## Effectiveness Comparison

### Quantitative impact of data-strategy ablations (distinguishing: filtering-strictness ablation / caption density-style ablation / data-mixture ablation, and corresponding evaluation metrics) ⚠️

[Uncertain]. The technical report contains no ablation experiments on data strategy whatsoever — no ablation of filtering strictness, no ablation of caption density/style, and no ablation of data mixture ratio. The ablation experiments in the report are all concentrated on the algorithmic and architectural side: GRPO's policy reweighting and KL reweighting, the max-group-std-deviation trick, plus efficiency components on the inference side such as Block Sparse Attention, the coarse-to-fine strategy, and conditioning-token caching. As a result, the report offers no way to quantify the contribution of any data-processing step to the final metrics — this is the main weak point of its data-side disclosure.

### Evidence of quality vs. quantity (cases where small, refined data outperforms large, mixed data) ⚠️

[Uncertain]. The report offers no direct quantitative evidence of "small and refined outperforming large and mixed" (no controlled experiment). Only strategy-level indirect evidence can be observed: the SFT stage uses a "carefully curated, high-quality dataset" to complete style and instruction-following alignment in just 7.5k iterations (less than 1.1% of the pretraining total of 678k iterations); and the design of "inverse-density sampling in caption embedding space," which explicitly favors rare samples while proactively discarding redundant high-density samples, implicitly reflects the team's judgment that "distribution balance takes priority over sample count." However, none of this is backed by ablation data.

### Alignment between the domain distribution of training data and the category taxonomy of evaluation benchmarks (e.g. VABench's seven major categories) ⚠️

There is a methodological correspondence between the taxonomy of the training data and the taxonomy of the evaluation benchmarks, though the report does not explicitly declare that the two are aligned: the domain division on the training side comes from an unsupervised taxonomy via caption text-embedding clustering + LLM induction (e.g., personal interactions, artistic performances, natural landscapes, etc.); on the evaluation side, the internal T2V benchmark covers 48 distinct categories, totaling 1228 test cases (500 for human evaluation + 728 for automatic evaluation); the I2V benchmark is 100 first-frame reference images × 4 prompt categories, totaling 400. The report does not state whether these 48 evaluation categories are directly derived from the training data's clustering categories, nor does it give a mapping between the two or the proportion of each category on both sides. Human evaluation uses both MOS and GSB metrics, with each video rated by 3 independent annotators. On public benchmarks, the model is also compared (the HF model card gives an internal-evaluation result of MOS 3.38 for overall T2V quality). [Uncertain: the correspondence between the 48 categories and the training clustering categories]

## Uncertain Fields

The research information for the following fields is partly uncertain (sources marked with ⚠️):

- data_scale
- data_sources
- provenance_licensing
- duration_distribution
- resolution_aspect_distribution
- domain_distribution
- audio_category_distribution
- language_accent_distribution
- funnel_retention_rate
- quality_filtering
- motion_filtering
- deduplication
- safety_filtering
- caption_model
- caption_structure
- dialogue_transcription_attributes
- geometric_structured_annotation
- synthetic_data_synthesis
- human_in_loop
- sync_metric_and_threshold
- temporal_vs_semantic_sync
- audio_quality_filtering
- stage_data_mixture
- post_training_data
- data_infra_throughput
- data_ablation
- quality_over_quantity_evidence
- benchmark_taxonomy_alignment
</content>
