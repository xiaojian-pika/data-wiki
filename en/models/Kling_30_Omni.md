# Kling 3.0 / Kling Video 3.0 Omni (internally also known as Kling O3)

> Topic: Data processing for a video generation model (including simultaneous audio-video generation) — filtering pipeline, data distribution, and annotation approach

[← Back to Home](../index.md)

**Table of Contents**: [Basic Information](#basic-information) · [Data Scale and Distribution](#data-scale-and-distribution) · [Cleaning Pipeline](#cleaning-pipeline) · [Annotation Approach](#annotation-approach) · [Audio-Video Alignment](#audio-video-alignment) · [Training Coordination](#training-coordination) · [Results Comparison](#results-comparison)

## Basic Information

### Name

Kling 3.0 / Kling Video 3.0 Omni (internally also known as Kling O3)

### Publishing Organization/Company

Kuaishou Technology — Kling large-model team / Kuaishou Visual Generation and Interaction Center (KwaiVGI; the GitHub organization also appears as KlingTeam / KlingAIResearch)

### Release Date (Technical Report/Paper/Open-Source Date)

The Kling 3.0 series (Kling Video 3.0, Kling Video 3.0 Omni, Kling Image 3.0, Kling Image 3.0 Omni) officially launched globally on February 4-5, 2026. Its direct technical predecessor and the publicly citable official technical report is the "Kling-Omni Technical Report" (arXiv:2512.16776, submitted December 18, 2025, 67 authors, credited to the Kling Team); the native audio-visual co-generation capability was first introduced in Kling 2.6 in December 2025. Related reports from the same lineage include the "KlingAvatar 2.0 Technical Report" (arXiv:2512.13313, December 2025) and the "Kling-MotionControl Technical Report" (arXiv:2603.03160, March 2026).

### Type (Model/Dataset/Toolchain/Benchmark)

Closed-source commercial model (video generation/editing foundation model + unified multimodal generation model), served via the Kling AI web app, mobile app, and open-platform API (including third-party hosting such as Replicate/fal); not a dataset, not a toolchain, not a benchmark.

### Openness (Whether Weights/Code/Data/Pipeline Are Open) ⚠️

Weights: not open-sourced; code: not open-sourced; training data: not disclosed; data processing pipeline: only qualitatively described in the technical report, not open-sourced. This is a typical closed-source commercial API model. [uncertain: whether any component will be open-sourced later]. For reference, open-source assets from the same team serve as indirect corroboration: the Koala-36M video dataset and its cleaning/annotation pipeline (github.com/KwaiVGI/Koala-36M, arXiv:2410.08260), and the Kling-Foley video-to-audio model and Kling-Audio-Eval benchmark (arXiv:2506.19774), were both open-sourced by the Kuaishou Kling team, and can be regarded as public slices of the Kling series' data-processing methodology — but they are not equivalent to the actual training data of 3.0 Omni.

### Whether Simultaneous Audio-Video Generation Is Supported, and the Implementation Approach (Native Joint/Cascaded/MoE Fusion) ⚠️

Yes. It is officially positioned as "native audio-visual co-generation / native audio-visual synchronization": video, dialogue speech, ambient sound, and sound effects are jointly produced by a unified model within a single generation pass, without a cascaded post-hoc dubbing or V2A process (in contrast to the earlier Kling-Foley-style video-to-audio cascade). Based on a unified Multimodal Visual-Language (MVL) representation and 3D spatiotemporal joint attention, audio and vision are jointly modeled within a shared embedding space. It supports lip and emotion synchronization across five languages — Chinese, English, Japanese, Korean, and Spanish — and multiple regional accents; the Omni version additionally supports multi-image + timbre reference binding (character appearance and voice timbre locked together), multi-speaker co-reference and distinct timbres for 3+ characters, and audio continuity across multi-shot storyboards (the Director Memory context repository). Specifications: up to 15 seconds per generation, up to native 4K (3840×2160), up to 60fps, and up to 6 freely combinable shots within a single clip. [uncertain: whether the underlying architecture is a single DiT performing joint denoising, or an MoE-style fusion with separate video/audio branches plus cross-modal attention — the technical report does not disclose details of the audio branch]

### List of Research Information Sources (URLs of papers/technical reports/official documentation/news, with the nature of each source noted: primary official / same-team corroboration / third-party reporting)

- Primary official: Kling-Omni Technical Report (arXiv:2512.16776, three-tier data system/training stages/DPO) https://arxiv.org/abs/2512.16776
- Same-team corroboration: Koala-36M (arXiv:2410.08260, quantitative funnel/CSS segmentation/VTSS/structured captions) https://arxiv.org/abs/2410.08260 and https://github.com/KwaiVGI/Koala-36M
- Same-team corroboration: Kling-Foley (arXiv:2506.19774, audio data pipeline/VAD 0.2 threshold/CLAP filtering) https://arxiv.org/abs/2506.19774
- Same-team corroboration: KlingAvatar 2.0 (arXiv:2512.13313, multi-speaker data sources and automated annotation pipeline) https://arxiv.org/abs/2512.13313
- Same-team corroboration: Kling-MotionControl Technical Report (arXiv:2603.03160) https://arxiv.org/abs/2603.03160
- Third-party reporting: People's Daily Online / Economic Information Daily official release on the Kling 3.0 series launch, 2026-02-05 http://finance.people.com.cn/n1/2026/0205/c1004-40660255.html
- Primary official: Kling Video 3.0 Omni user guide https://kling.ai/quickstart/klingai-video-3-omni-model-user-guide
- Primary official: Replicate official hosted API spec page https://replicate.com/kwaivgi/kling-v3-video
- Third-party reporting: 302.AI Benchmark Lab hands-on test of Kling O3 https://zhuanlan.zhihu.com/p/2004988563100566106

## Data Scale and Distribution

### Training Data Scale (Number of Video Clips/Hours/Tokens, Pretraining vs. SFT Separately) ⚠️

[uncertain] The official party has never disclosed the training data scale of Kling 3.0 Omni (number of video clips/hours/tokens); neither the pretraining nor the SFT scale has been revealed. The Kling-Omni technical report only qualitatively states "large-scale real-world data collection + task-oriented synthetic data construction," with no figures given. For reference, same-team scale corroboration is available: the open-source Koala-36M dataset contains 36 million clips, averaging 13.75 seconds, at 720p (roughly 137,000 hours); Kling-Foley's TV2A training data comprises 122,000 hours / 55 million 8-second clips, totaling over 100 million video-audio-text triplet samples. The actual scale of Kling 3.0 Omni should be significantly larger than the above public datasets, but there is no public basis for this.

### Data Source Composition (Proprietary/Public Datasets/Web Crawling/Licensed Acquisition/Synthetic Data) ⚠️

Qualitatively disclosed as a mixture of three categories: (1) automated large-scale internet video/image mining — the report states an in-house embedding model is used to retrieve semantically relevant cross-modal samples from massive internet-scale data; (2) proprietary/acquired film-and-television-grade high-quality footage — both Kling-Omni and KlingAvatar 2.0 emphasize "cinematic-level video data," and KlingAvatar 2.0 explicitly mentions podcasts, interviews, and multi-character film/TV dramas as sources of multi-speaker dialogue data; (3) an expert-model-driven synthetic data pipeline — in-house image-editing models and video-understanding models are used to reverse-construct training pairs for editing and multi-image reference tasks. In addition, Kuaishou owns a proprietary short-video platform content pool, which in theory would be an important source, but the official party has not confirmed its use in Kling training. [uncertain: the specific proportions of proprietary platform data, licensed acquisition, and crawled data]

### Data Compliance and Provenance (Proportion of Licensed Data, Rights-Cleared Datasets, C2PA, etc.) ⚠️

[uncertain] The official party has not disclosed the proportion of licensed data, a list of rights-cleared datasets, or a data provenance mechanism. Product-side compliance measures are mainly on the output end rather than the training end: generated content carries explicit/implicit AI labels and watermarks per China's "Interim Measures for the Management of Generative AI Services" and the AI-Generated Synthetic Content Labeling Measures (mandatory watermark on the free tier, removable on paid tiers). [uncertain: whether C2PA Content Credentials are integrated — no official Kling statement of C2PA support was found in research, in contrast to the publicly announced support from OpenAI/Adobe/Google]. On the data side, only content-safety/NSFW filtering is mentioned in the cleaning stage; no copyright-filtering details are given. Kling-Foley's open-source benchmark annotation guidelines explicitly require "excluding copyrighted background music containing vocals," which can be regarded as circumstantial evidence of the team's copyright awareness.

### Clip Duration Distribution and Segmentation Strategy ⚠️

[uncertain] The training clip duration distribution has not been disclosed. Only the filtering-rule level can be confirmed: the Kling-Omni report states that basic filtering includes "resolution and duration thresholds" and removes clips with "abrupt scene changes and incoherent shot transitions" (i.e., favoring the retention of continuous single-shot clips). Public work from the same team can serve as reference: after temporal segmentation, Koala-36M clips average 13.75 seconds; Kling-Foley uniformly segments long videos into 8-10 second clips. Kling 3.0 Omni's target generation length is 15 seconds, so the training clip duration distribution likely covers and slightly exceeds this length to support 15-second/multi-shot generation. [uncertain: the specific bucketing and the ratio of long to short samples]

### Resolution/Aspect Ratio Distribution and Bucketing Strategy ⚠️

[uncertain] The resolution/aspect ratio distribution and bucketing strategy have not been disclosed. What is known: the model supports native 4K (3840×2160) 60fps output, and the official party emphasizes that this is pixel-level native generation during the diffusion process rather than post-hoc super-resolution, implying that the training set contains a substantial proportion of genuine 4K/high-frame-rate footage; it also supports multiple aspect ratios (16:9/9:16/1:1, etc.), and industry-standard multi-resolution/multi-aspect-ratio bucketing along with a progressive resolution curriculum are very likely adopted, though the report does not state this explicitly. For reference: Koala-36M is unified at 720p; Kling-Foley requires source videos ≥720P. KlingAvatar 2.0 uses a spatiotemporal cascade framework (spatial super-resolution + temporal frame interpolation) to achieve high resolution and high frame rate, suggesting that within the Kling system, 4K/60fps may be partly handled by a cascaded refinement module. [uncertain: the boundary between native 4K generation and cascaded refinement]

### Category/Domain Distribution and Mixture Strategy (Ratio Control and Concept Balancing across People, Actions, Scenes, Styles, etc.) ⚠️

[uncertain] Category/domain mixture figures have not been disclosed. Qualitatively, the Kling-Omni report states that data collection pursues "broad scenario coverage" spanning diverse subjects and stylistic variants; and in the multimodal alignment stage, a dedicated character identity-consistency check is applied to "human-centric" tasks, indicating that person-related data is a focus subdomain modeled and mixed separately. The product's capability emphasis (AI-director storyboarding, multi-character dialogue, cinematic-grade texture, physical laws and inertia/collision) reflects a training mixture clearly skewed toward: character performance and dialogue, cinematic camera movement, and physical interaction scenes. KlingAvatar 2.0 explicitly builds evaluation subsets across three categories — Chinese speech, English speech, and singing — suggesting that speech-type data is sub-mixed by use case. [uncertain: the specific ratio-control and concept-balancing methods]

### Audio Category Distribution and Mixture (Ratio and Control Strategy for Speech/Sound Effects (Foley)/Music/Ambient Sound/Silence) — A Dimension Unique to AV Models ⚠️

[uncertain] Kling 3.0 Omni has not disclosed the mixture ratio of speech/sound effects/music/ambient sound/silence. Capability coverage spans three complete categories: dialogue speech (5 languages + accents), ambient sound, and sound effects (foley); the official party does not emphasize music generation. The same team's Kling-Foley approach can serve as methodological corroboration: an audio classification model first sorts material into four classes — sound effects/music/speech/singing — and then an audio-understanding large model generates descriptions separately for each class; training material is retrieved and mixed by keyword search against the AudioSet three-tier ontology (roughly 1,000 tertiary categories, consolidated into 1,919 labels across nine major sound scenes: traffic, human voice, animal sounds, etc.); and VAD is used to filter out samples with a silence proportion ≥0.2, i.e., an explicit upper bound is set on the "silence ratio." Whether Kling 3.0 Omni follows the same taxonomy and silence threshold is undisclosed.

### Narrative Structure Distribution (Single-Shot vs. Multi-Shot, Average Clip Duration, Shot-Count Distribution, Whether Native Audio Tracks Are Included) ⚠️

[uncertain] The ratio of single-shot to multi-shot samples and the shot-count distribution have not been disclosed. Structural facts that can be inferred: (1) the basic cleaning stage removes clips with abrupt transitions/incoherent shot changes, indicating that the pretraining corpus is predominantly single-shot continuous clips; (2) the product supports "intelligent storyboarding" and up to 6 shots per clip with cross-shot character and timbre consistency (Director Memory), implying that the post-training stage must have introduced multi-shot/storyboard-level long-context samples (including shot-boundary annotations and cross-shot identity/timbre correspondence); (3) native audio-visual co-generation requires training samples to carry native audio tracks, so the high-quality subset should predominantly consist of clips "with native synchronized sound," with audio-less samples either removed or used only in a vision-only stage. Points (2) and (3) above are inferences based on product capabilities; the report does not explicitly document them.

### Language/Accent Distribution (The Data Basis for Multilingual Lip-Sync Capability) ⚠️

[uncertain] Language/accent proportions have not been disclosed. Capability coverage is clear: five languages — Chinese, English, Japanese, Korean, and Spanish — with support for multiple regional accents/dialects and regional accent control, with lip shapes synchronized to the corresponding language. KlingAvatar 2.0 performed data augmentation for multilingual, multi-character dialogue scenarios, and its evaluation set is built from 100 Chinese-speech examples, 100 English-speech examples, and 100 singing examples, suggesting that Chinese and English are the dominant languages, with the other three languages having smaller data volumes. [uncertain: the hour count per language, the dialect-coverage list, and the annotation method for accent labels]

## Cleaning Pipeline

### Overall Filtering Funnel Structure (Number of Filtering Tiers, Order) ⚠️

The Kling-Omni technical report discloses a "three-tier data processing system": Tier 1, basic filtering — resolution/duration threshold checks, frame-wise and temporal fingerprint deduplication, audio-visual corruption detection, content safety and NSFW filtering; Tier 2, temporal quality assessment — scoring for blur/jitter/compression noise, removal of abrupt scene changes and incoherent transitions, and removal of static clips with "excessively low action semantic density"; Tier 3, multimodal alignment verification — semantic consistency between video captions and actual footage, fidelity consistency between reference images and target videos, and character identity-consistency checks for human-centric tasks. Upstream, there is also an automated internet data-mining pipeline (an in-house embedding model performing cross-modal semantic retrieval) and an expert-model-driven synthetic data construction pipeline. The full funnel publicly disclosed by the same team's Koala-36M is: raw video → temporal segmentation (Color-Struct SVM) → structured annotation → manual threshold filtering → VTSS model filtering, which can serve as a public paradigm reference for the Kling system's data funnel. [uncertain: where an audio-specific filtering tier would be inserted within the three-tier system for Kling 3.0 Omni]

### Quantitative Funnel Retention Rate (Input/Output Volume and Final Retention Rate at Each Tier, e.g., Apollo's 27%) ⚠️

[uncertain] Kling 3.0 Omni has not disclosed the input/output volume or final retention rate for any filtering tier. The same team's Koala-36M publishes a complete quantitative funnel, which can serve as an order-of-magnitude reference: after segmentation + annotation, Koala-all totals 48 million clips → after manual threshold filtering, Koala-37M totals 37 million clips (roughly 77% retained) → after VTSS filtering (threshold 2.5), Koala-36M totals 36 million clips (roughly 97% further retention relative to the prior stage, ~75% end-to-end). Kling-Foley does not disclose a retention rate, giving only the final scale (55 million clips / 122,000 hours).

### Shot Segmentation Method (PySceneDetect/In-House Model/Shot-Aware Splitting) ⚠️

[uncertain, though a same-team public paradigm exists for the methodology] The Kling-Omni report only states that it "detects and removes abrupt scene changes and incoherent shot transitions," without specifying the tool. The same team's Koala-36M proposes and open-sources an in-house shot-segmentation method, Color-Struct SVM (CSS): color distance is measured via BGR histogram correlation, and structural distance is measured via Canny-edge-based luminance SSIM, with a linear SVM classifier determining transitions; temporal smoothing is then layered on top, assuming that video change follows a Gaussian distribution, with a transition judged genuine only if it exceeds the 3σ confidence interval — thereby distinguishing "gradual transitions" from "fast-motion scenes." The method explicitly claims to outperform threshold-based approaches such as PySceneDetect. Kling 3.0 Omni very likely uses an iterated version of this in-house segmenter, but there is no official confirmation.

### Quality Filtering (Aesthetic Score, Sharpness, OCR Text Filtering, Black-Bar/Watermark/Logo Detection) ⚠️

Multi-layered composite filtering: (1) Hard admission criteria — resolution and duration thresholds, audio-visual file corruption detection (Kling-Omni's basic tier); on the Kling-Foley side, source videos are required to be ≥720P. (2) Temporal visual quality scoring — identification and removal of blur, jitter, and compression noise (Kling-Omni's second tier). (3) The same team's Koala-36M offers a more fine-grained public approach: rather than a single-metric hard threshold, a "training-suitability assessment network" is trained to output a unified VTSS (Video Training Suitability Score), whose inputs include sub-metrics such as sharpness score, aesthetic score, and motion score; the network comprises a dynamic branch (3D Swin Transformer) + a static branch (ConvNeXt) + a feature/label branch, fused via a Weight Cross-Gating Block, with the threshold set at 2.5 (determined from the score's bimodal distribution); its core motivation is to avoid mistakenly discarding high-quality data via single-metric hard thresholds. (4) Subtitles/text: Kling-Foley's data specification calls for "as little subtitle content as possible." [uncertain: whether Kling 3.0 Omni has a dedicated black-bar/watermark/logo detection module and OCR text filtering — the report does not mention this]

### Motion Filtering (Optical Flow/Motion Score Thresholds, Removal of Static and Jittery Content) ⚠️

Kling-Omni explicitly removes clips with "excessively low action semantic density," i.e., static/near-static footage is filtered out; jitter is listed as a separate removal criterion within temporal quality assessment. The same team's Koala-36M treats motion score as one of the input sub-metrics for the VTSS network, and explicitly models temporal motion quality via the 3D Swin dynamic branch, rather than relying purely on an optical-flow threshold. [uncertain: whether optical flow (e.g., RAFT) is used to compute the motion score, and its specific threshold]

### Deduplication Method (Exact Deduplication and Embedding-Based Semantic Deduplication Recorded Separately) ⚠️

The Kling-Omni report explicitly adopts a "frame-wise and temporal fingerprinting" deduplication mechanism, which falls into the category of perceptual-hash/fingerprint-based exact and near-duplicate deduplication. [uncertain: whether there is additionally an embedding-based semantic-level deduplication step — the report does not mention one, though its internet-mining stage uses an in-house embedding model for cross-modal semantic retrieval, meaning the infrastructure for semantic deduplication is in place]

### VLM/LLM as Quality Inspector (Multimodal LLM Quality Scoring and Mismatch Removal — the 2026 Trend from Shallow Scorers to Large-Model Semantic Judgment) ⚠️

Yes, and this is a core component of Kling 3.0's data system, reflecting the 2026 trend of "shifting from shallow scorers to large-model semantic judgment." Kling-Omni's third tier, "multimodal alignment," is entirely judged by a model: it determines the semantic consistency between video captions and actual visual content, the fidelity of reference images to target videos, and character identity consistency for human-centric tasks (cross-frame ID consistency checks). The synthetic-data side is likewise driven by "expert models" (in-house image-editing models + video-understanding models). Same-team corroboration: Koala-36M replaces manual threshold combinations with a trained VTSS neural network, and uses a fine-tuned LLaVA to generate structured long captions; Kling-Foley uses CLAP to compute consistency between audio and text labels and retains only high-consistency data, uses an audio-understanding large model to classify and generate descriptions, and uses an LLM to extract metadata and fuse the visual and audio descriptions. Kling's official statements also publicly describe their data pipeline as covering "massive-scale video mining, multi-dimensional annotation and filtering, video description enhancement, and data-driven quality assessment." [uncertain: the specific name and parameter scale of the in-house VLM serving as judge]

### Safety and Compliance Filtering (NSFW, Copyright, Face/Privacy) ⚠️

Kling-Omni's basic filtering tier explicitly includes "content safety / NSFW filtering." On the product side, there is additionally strict input/output safety review (blocking of faces/celebrity likenesses, and politically or violently sensitive content) and AI-generated content labeling/watermarking, in compliance with Chinese regulatory requirements. [uncertain: the specific strategies and models used for face privacy, likeness rights, and copyright filtering on the training-data side]

## Annotation Approach

### Captioning Model Used (In-House VLM/Open-Source Model, Model Scale) ⚠️

[uncertain] The captioning model used by Kling 3.0 Omni is undisclosed; the official party only states that there is a "video description enhancement" step driven by an in-house video-understanding/multimodal model. Public practices from the same team can serve as reference: Koala-36M uses GPT-4V to generate seed captions and then fine-tunes LLaVA as the final annotation model (a high-resolution vision encoder + 2x2 spatial pooling, trained on a mix of images and videos); Kling-Foley uses a three-stage fusion of an audio classification model + an audio-understanding large model + an LLM to generate audio-visual descriptions. Kuaishou additionally has an in-house multimodal large model, Kwai Keye-VL (arXiv:2507.01949), capable of undertaking large-scale video annotation. [uncertain: model scale, and whether it has since been replaced by a newer in-house VLM]

### Caption Density and Degree of Structuring (Short/Long/Dense Descriptions, Structured Fields such as Camera Motion, Style Tags) ⚠️

[uncertain, though a same-team public paradigm exists] The caption structure for Kling 3.0 Omni is undisclosed, but the Kling-Omni report notes that pretraining text conditions span a wide range "from concise prompts to elaborate narratives" — that is, caption length/density diversity is deliberately mixed to ensure the model is robust to both long and short prompts (this is also the data foundation underpinning its ability to handle long vCoT instructions). The same team's Koala-36M publicly discloses an explicit six-field structured caption schema: subject, subject actions, environment, visual language (style/lighting/composition), camera language (camera movement/angle/shot scale), and world knowledge, averaging 202 words in length (versus 13.2 words for Panda-70M). Kling 3.0's powerful camera-motion control and "intelligent storyboarding" capabilities depend heavily on this kind of structured, dense captioning that includes camera-language fields.

### Joint Audio-Video Caption Structure (Whether Both Visual and Auditory Tracks Are Covered Simultaneously, Whether Split into Separate Fields — e.g., LTX-2's Full Soundscape Description, Script-a-Video's Factorized Streams, Foley-Omni's Three Fields) ⚠️

[uncertain] The joint audio-video caption schema for Kling 3.0 Omni is undisclosed. Native audio-visual co-generation necessarily requires training samples to carry both a visual and an auditory description track simultaneously. The public paradigm offered by the same team's Kling-Foley is "split-then-fuse": an audio classification model first splits the audio track into sound effects/music/speech/singing; each category then has an audio-understanding large model generate an audio description, while a video description is generated concurrently; finally, an LLM fuses the video description and audio description into a unified caption. Its open-source benchmark, Kling-Audio-Eval, explicitly requires video captions and audio captions to be annotated independently (to avoid cross-modal bias contaminating one another), and includes sound-event labels (1,919 labels across nine major sound scenes). This three-layer structure — "independent fields + event labels + fused caption" — is very likely the prototype for Kling 3.0 Omni's joint captioning, though there is no official confirmation.

### Dialogue Transcription and Speaker Attribute Annotation (ASR Transcription + Speaker Identity/Language/Accent/Emotion) ⚠️

[uncertain] The specific scheme for ASR transcription and speaker attribute annotation is undisclosed. Working backward from capabilities: the model supports lip synchronization across 5 languages plus accents, co-reference of distinct timbres for multiple characters (3+), and "uploading reference material to bind a specific subject's visual features and voice timbre together" — this requires training data to have sentence-level dialogue transcription, timestamp alignment, speaker diarization with speaker identity/timbre IDs, language and accent labels, and emotion labels as structured annotations. KlingAvatar 2.0 explicitly constructs multi-speaker dialogue data from podcasts, interviews, and multi-character TV dramas, and establishes an automated annotation pipeline for multi-character video (YOLO person detection + DWPose keypoints + SAM2 segmentation and temporal tracking, cross-validated against frame-level detection results) used to bind "who is speaking" to the audio track — this is public evidence of speaker-to-frame correspondence annotation. [uncertain: the ASR model used, and the emotion-label taxonomy]

### Geometric and Structured Annotation (Camera Parameters, Depth, 3D Point Tracks, Action Annotation, Explicit State) ⚠️

[uncertain] The Kling-Omni report does not disclose geometric annotations such as camera parameters, depth, or 3D point tracks. Indirect evidence: Kling 3.0 emphasizes "3D spatiotemporal joint attention," capable of modeling physical interactions such as inertia, weight transfer, and collision detection, and Kling has historically had strong camera-control capability, suggesting that the training data includes structured labels for camera motion/shot scale (Koala-36M's "camera language" field is exactly a text-form annotation of camera motion). Structured human-body annotation has clear evidence in KlingAvatar 2.0: DWPose keypoints, SAM2 instance segmentation, and temporal tracking. [uncertain: whether there is explicit camera extrinsic-parameter estimation, depth maps, or point-tracking annotation]

### Synthetic Data Construction (Controlled Perturbation/Editing to Construct Training Pairs, e.g., InstructAV2AV) ⚠️

Yes, and this is explicitly disclosed by the official party as a key focus. The Kling-Omni report states that data construction follows a dual track of "large-scale real-world data collection + task-oriented synthetic data construction"; the synthetic side is implemented via an "expert-model-driven synthesis pipeline," using in-house image-editing models and video-understanding models to construct training pairs for image/video editing tasks and multi-image reference tasks; it also proposes an "automatic reverse synthesis strategy" to construct reference-to-video training samples, i.e., reference images/conditions are reverse-extracted from finished footage and then paired as (condition, target video) training pairs. This directly underpins the Omni Edit capability (localized targeted editing without regenerating the entire clip) and the multi-image + timbre reference-locking capability. [uncertain: the proportion of synthetic data within the overall total; whether there is controlled-perturbation-style synthesis on the audio side, such as the construction of InstructAV2AV-style audio-video editing pairs]

### Degree of Human Involvement (Human Annotation, Human Quality Review, Model Pre-Screening + Human Review)

The basic form is "model-based automated filtering as the primary approach + human involvement at key junctures." Confirmed points of human involvement: in the post-training stage, DPO preference data is derived from humans evaluating multiple video variants generated under the same MVL condition and giving preference rankings (the Kling-Omni report explicitly describes this as human-evaluated); on the data side, there are additionally "manually curated resources" supplementing data sources. The same team's Kling-Audio-Eval (from Kling-Foley) is entirely human-annotated/proofread: 20,935 samples, with annotators correcting model-pre-generated captions, verifying whether labels belong to the predefined taxonomy, and verifying audio-visual quality and alignment (valid-sample rules: no vocals in the foreground sound, sound effects originate from objects/actions visible on screen, video ≥5 seconds, sound effect ≥2 seconds, no copyrighted background music containing vocals). [uncertain: whether large-scale training data has a human spot-check ratio, and the scale of human annotation for the SFT curated set]

## Audio-Video Alignment

### Audio-Video Synchronization Detection Method (Lip Sync, Event Alignment) ⚠️

[uncertain] The audio-visual synchronization detection method for Kling 3.0 Omni's training data is undisclosed. The Kling-Omni report only mentions "audio-visual corruption detection" at the basic tier; the third-tier multimodal alignment mainly targets caption-vision, reference image-video, and character identity consistency, without naming lip-sync detection specifically. KlingAvatar 2.0 states there is "an extensive filtering pipeline to ensure high visual fidelity and consistent audio-lip synchronization," confirming the existence of a lip-sync filtering step but not disclosing the method. On the Kling-Foley side, the model architecture built in a frame-level "audio-visual synchronization module" alongside a visual semantic representation module for conditional alignment, and the evaluation distinguishes between semantic alignment and temporal alignment as two separate metric categories — indicating that the team has a systematic measurement capability for "event-level audio-visual alignment." Overall assessment: Kling 3.0 Omni's data-side synchronization filtering should include at least lip-sync scoring (for speech) and event-level temporal alignment scoring (for sound effects), but no public details exist for either.

### Specific Synchronization Detection Metrics and Thresholds (SyncNet/Synchformer/LSE/In-House, Threshold Values, e.g., MOVA's LSE-D≤9.5 and LSE-C≥4.5, SkyReels-V4's SyncNet |offset|≤3 ∧ conf>1.5) ⚠️

[uncertain] No synchronization metric names or threshold values have been disclosed (e.g., specific gates for SyncNet confidence / LSE-C / LSE-D / AV-align). The only quantitative audio threshold in public materials comes from Kling-Foley's data specification: VAD silence proportion < 0.2; valid evaluation samples require video ≥5 seconds and sound effects ≥2 seconds. In the Kling-Foley paper, the team reports SOTA results across four metric categories — distribution matching, semantic alignment, temporal alignment, and audio quality — but those are model evaluation metrics, not data-filtering thresholds.

### Separate Handling of Temporal Sync vs. Semantic Sync (Temporal Alignment and Content-Semantic Matching as Two Independent Filtering Conditions) ⚠️

[uncertain, though strong indirect evidence exists] The team's methodology explicitly treats "temporal alignment" and "semantic alignment" as two independent dimensions: Kling-Foley's architecture separates a visual semantic representation module (handling semantic matching) from an audio-visual synchronization module (handling frame-level temporal alignment), and its evaluation likewise separately lists semantic alignment and temporal alignment; on the data side, CLAP is used to compute audio-to-text-label consistency (a semantic condition), VAD/event-duration rules constrain the temporal dimension (a temporal condition), and it is required that "sound effects must originate from objects or actions visible on screen" (a semantic-causal condition). Whether Kling 3.0 Omni continues this two-way split has not been directly disclosed.

### Audio Quality Filtering (SNR, Silence Detection and Silence-Ratio Thresholds, Removal of Tracks Without Audio, Removal of Off-Screen Audio Sources, Background Music Separation) ⚠️

[uncertain, though the same team has clear public specifications] Kling 3.0 Omni has not disclosed audio quality filtering details, with only Kling-Omni's "audio-visual corruption detection" available. Kling-Foley's public specification is quite complete and can serve as a paradigm: audio is uniformly standardized to WAV / 44kHz / 16bit / stereo; quality metrics include SNR (signal-to-noise ratio), MOS score, clipping ratio, and audio bandwidth; VAD is used for silence filtering, with samples whose silence proportion is ≥0.2 removed; evaluation samples require the foreground sound to contain no vocals, and exclude copyrighted background music containing vocals (i.e., involving vocal separation/removal from BGM). [uncertain: whether Kling 3.0 Omni removes off-screen/narration audio sources, and whether background music separation is performed]

### Classification and Separate Handling Strategy for Speech/Sound Effects/Music ⚠️

[uncertain, though a same-team public paradigm exists] Kling-Foley explicitly first uses an audio classification model to sort material into four classes — sound effects, music, speech, singing — and then, for each class, separately invokes an audio-understanding large model to generate descriptions and applies separate annotation and filtering rules, before finally fusing everything together. KlingAvatar 2.0 likewise builds data and evaluations separately by Chinese speech/English speech/singing. Kling 3.0 Omni's capability coverage of dialogue speech, ambient sound, and sound effects — each independently controllable (separately specifiable) — strongly suggests that the same classify-and-conquer processing exists on the training side, but the official party has not disclosed the specific classifier or mixture ratios.

## Training Coordination

### Multi-Stage Training Curriculum and Data Curriculum Scheduling (Basis for Stage Division: Resolution/Duration/Quality Score/Modality; Low-Res→High-Res, Image→Video, Short→Long) ⚠️

The stage division disclosed in the Kling-Omni report is: pretraining (large-scale text-video paired data + image-to-video tasks, with text-condition length mixed from short prompts to long narratives) → continue-training (introducing reference-to-video, image and video editing, and semantic tasks, using a highly interleaved multi-task mixture format) → quality-tuning (fine-tuning on a curated dataset with balanced task distribution and extremely high video quality) → post-training (DPO). [uncertain: whether there is an explicit curriculum schedule by resolution (low-res→high-res), duration (short→long), or modality (image→video→audio-video) — the report does not elaborate; but given the product's support for native 4K/60fps/15-second generation and the later-added native audio capability, a resolution- and modality-progressive curriculum almost certainly exists, and a "vision-first, audio-modality-added-later" route is consistent with the timeline in which native audio was only introduced with Kling 2.6]

### Changes in Data Mixture Across Training Stages (Pretraining/Annealing/SFT High-Quality Subset) ⚠️

Qualitatively, a three-stage mixture shift can be confirmed: pretraining relies mainly on large-scale text-video/image-video data of uneven quality but broad coverage, with caption density mixed between long and short; the continue-training stage shifts to an interleaved multi-task mixture (generation/editing/reference/semantic understanding mixed by task, with emphasis on inter-task balance); the quality-tuning stage (roughly equivalent to annealing) switches to a small-scale curated set with "balanced task distribution + extremely high video standards." [uncertain: the specific mixture figures for each stage, the size of the curated set, and the number of annealing steps]

### Post-Training Data (SFT Curated Set Size and Selection Criteria, Number of Preference Pairs and Annotation Method, Reward Model Training Data) ⚠️

Post-training uses DPO (Direct Preference Optimization): multiple video variants are sampled under the same Multimodal Visual-Language (MVL) condition, and human evaluators compare them to form preference pairs used for alignment. The "high-quality, task-distribution-balanced" SFT curated set used in the quality-tuning stage is the other pillar. [uncertain: the number of preference pairs, the scale of the annotator pool and the annotation criteria, and whether an explicit reward model was trained and used for RLHF or best-of-n sampling — the report does not mention a reward model. Preference annotation on the audio dimension (e.g., whether lip-sync quality or timbre consistency serve as independent scoring items) is also undisclosed]

### Data Processing Infrastructure and Throughput (NeMo Curator/Data-Juicer/In-House, GPU Acceleration Ratio, Processing Scale, Cost) ⚠️

[uncertain] No details of the data processing infrastructure (in-house platform / NeMo Curator / Data-Juicer), GPU acceleration ratio, processing scale, or cost have been disclosed. The Kling-Omni report only generally mentions "efficient large-scale pretraining strategies and inference infrastructure optimization," and this infrastructure description leans toward the training and inference side rather than the data-processing side. Official public statements describe having built an "efficient, large-scale automated data pipeline" during development, covering massive-scale video mining, multi-dimensional annotation and filtering, video description enhancement, and data-driven quality assessment — inferable as an end-to-end, in-house Kuaishou data platform, but with no throughput or cost figures whatsoever.

## Results Comparison

### Quantitative Impact of Data Strategy Ablations (Distinguishing: Filtering-Strictness Ablation / Caption-Density-Style Ablation / Data-Mixture Ablation, and Corresponding Evaluation Metrics) ⚠️

[uncertain] Kling 3.0 Omni itself has not published any data-strategy ablation experiments (the evaluations in the Kling-Omni technical report focus on overall comparisons of contextual generation, reasoning-driven editing, and multimodal instruction-following, without data-dimension ablations). The same team's Koala-36M paper provides public data-ablation evidence that can serve as methodological reference: (1) Data-source/cleaning ablation — training on Koala-all versus training on Panda-70M yields VBench subject consistency +1.1% and background consistency +2.4% (the temporal quality improvement stems from more accurate shot segmentation); (2) Filtering-strategy ablation — VTSS model-based filtering outperforms manual multi-metric hard-threshold combinations, retaining more high-quality data while achieving better results; (3) Caption-structure ablation — after introducing structured captions with "metric conditions," the VBench semantic score rose from 0.4504 to 0.5915 (+14.1 percentage points). [uncertain: the quantitative impact of audio-data mixture ratios on lip-sync/event-alignment metrics in joint audio-video training — no public data exists]

### Evidence for Quality vs. Quantity (Cases Where Small, High-Quality Data Outperforms Large, Noisy Data) ⚠️

[uncertain, with only indirect same-team evidence] Kling 3.0 Omni's official materials disclose no direct case of "small and refined outperforming large and messy," but the design of its quality-tuning stage (fine-tuning on a small-scale curated set that is "task-balanced + extremely high video quality") itself embodies a quality-first engineering philosophy. Koala-36M's public conclusions come closer to direct evidence: 36 million rigorously cleaned clips outperform the larger-scale (70 million) but more poorly segmented and captioned Panda-70M; at the same time, VTSS's core motivation runs in the opposite direction — avoiding the mistaken removal of high-quality data via hard thresholds in pursuit of "refinement," i.e., emphasizing "precise selection" rather than "indiscriminate strictness."

### Alignment Between Training-Data Domain Distribution and Evaluation-Benchmark Taxonomy (e.g., VABench's Seven Major Categories) ⚠️

[uncertain] Kling 3.0 Omni has not disclosed the alignment relationship between training-data domain distribution and evaluation-benchmark taxonomy. Observable alignment practice exists on the audio side: Kling-Foley directly uses the AudioSet three-tier ontology (roughly 1,000 tertiary categories) as the retrieval ontology for data collection, and consolidates it into 1,919 labels / nine major sound scenes (traffic, human voice, animal sounds, etc.) within the Kling-Audio-Eval benchmark, achieving strict alignment between "collection ontology" and "evaluation taxonomy" — structurally analogous to VABench's seven major categories. On the visual side, the team has long used VBench as its primary alignment reference (all of Koala-36M's data ablations are reported across VBench's various dimensions), and Kling 3.0's official marketing likewise cites third-party arena/leaderboard rankings. [uncertain: whether Kling 3.0 Omni has performed training-data taxonomy alignment against audio-visual-sync-type benchmarks (such as AV-align, PhyAVBench, etc.)]

## Uncertain Fields

Research information for the following fields is partially uncertain (sources marked with ⚠️):

- data_scale
- data_sources
- provenance_licensing
- duration_distribution
- resolution_aspect_distribution
- domain_distribution
- audio_category_distribution
- narrative_structure_distribution
- language_accent_distribution
- pipeline_overview
- funnel_retention_rate
- shot_segmentation
- quality_filtering
- motion_filtering
- deduplication
- model_as_data_judge
- safety_filtering
- caption_model
- caption_structure
- joint_av_caption_schema
- dialogue_transcription_attributes
- geometric_structured_annotation
- synthetic_data_synthesis
- av_sync_detection
- sync_metric_and_threshold
- temporal_vs_semantic_sync
- audio_quality_filtering
- audio_type_handling
- multi_stage_curriculum
- stage_data_mixture
- post_training_data
- data_infra_throughput
- data_ablation
- quality_over_quantity_evidence
- benchmark_taxonomy_alignment
- openness
- av_generation
