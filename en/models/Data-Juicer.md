# Data-Juicer 2.0 (including the Data-Juicer Sandbox). Positioned as a one-stop data processing system and cloud-scale adaptive operator library "for foundation models and with foundation models," with the paper's full title being *Data-Juicer 2.0: Cloud-Scale Adaptive Data Processing for and with Foundation Models*. The companion Sandbox component's paper is titled *Data-Juicer Sandbox: A Feedback-Driven Suite for Multimodal Data-Model Co-development*, a data-model co-development middleware that closes the loop between "data recipes" and "model training/evaluation." In this survey of video-generation data processing, Data-Juicer's role is not that of a generative model, but rather that of underlying data cleaning/annotation operator infrastructure reused by multiple teams.

> Topic: Data processing for video-generation models (including joint audio-video generation): data cleaning pipelines, data distribution, annotation methods

[← Back to home](../index.md)

**Table of Contents**: [Basic Information](#basic-information) · [Data Scale and Distribution](#data-scale-and-distribution) · [Cleaning Pipeline](#cleaning-pipeline) · [Annotation Methods](#annotation-methods) · [Audio-Video Alignment](#audio-video-alignment) · [Training Integration](#training-integration) · [Performance Comparison](#performance-comparison)

## Basic Information

### Name

Data-Juicer 2.0 (including the Data-Juicer Sandbox). Positioned as a one-stop data processing system and cloud-scale adaptive operator library "for foundation models and with foundation models," with the paper's full title being *Data-Juicer 2.0: Cloud-Scale Adaptive Data Processing for and with Foundation Models*. The companion Sandbox component's paper is titled *Data-Juicer Sandbox: A Feedback-Driven Suite for Multimodal Data-Model Co-development*, a data-model co-development middleware that closes the loop between "data recipes" and "model training/evaluation." In this survey of video-generation data processing, Data-Juicer's role is not that of a generative model, but rather that of underlying data cleaning/annotation operator infrastructure reused by multiple teams.

### Publishing Organization/Company

Alibaba Group. The initiating team is the data team at Alibaba Tongyi Lab, co-built jointly with Alibaba Cloud PAI (Platform for AI), and developed in collaboration with external institutions including Anyscale (the Ray team), NVIDIA, and Sun Yat-sen University. Core authors: Daoyuan Chen (first/corresponding author, homepage yxdyc.github.io), Yilun Huang, Xuchen Pan, Nana Jiang, Haibin Wang, Yilei Zhang, Ce Ge, Yushuo Chen, Wenhao Zhang, Zhijian Ma, Jun Huang, Wei Lin, Yaliang Li, Bolin Ding, Jingren Zhou. The Sandbox paper's authors are Daoyuan Chen, Haibin Wang, Yilun Huang, Ce Ge, Yaliang Li, Bolin Ding, Jingren Zhou. The GitHub organization has migrated from modelscope/data-juicer to datajuicer/data-juicer.

### Release Date (Technical Report/Paper/Open-Source Date) ⚠️

Series timeline:
- 2023: Data-Juicer 1.0 first open-sourced, with the paper published at SIGMOD 2024 (approximately 50 text-only pretraining operators).
- July 16, 2024: The Data-Juicer Sandbox paper submitted to arXiv (arXiv:2407.11784 v1), with the latest v3 dated June 4, 2025, accepted as an ICML 2025 Spotlight. Around the same time it topped the VBench text-to-video leaderboard and was promoted in posts by the Alibaba Cloud developer community/ModelScope community.
- January 2025 (arXiv number 2501.14755; submission history shows v1 on December 23, 2024, v2 on June 4, 2025, v3 on October 29, 2025): The Data-Juicer 2.0 paper released, accepted as a NeurIPS 2025 Spotlight.
- Continued iteration up through the time of this survey: v1.4.4 (2025-12-01, coinciding with the NeurIPS Spotlight, adding 6 video/multimodal operators and S3 I/O), v1.4.5 (2026-01-15, 20+ new operators, Ray + vLLM pipeline, embodied-AI operators), v1.4.6 (2026-02-02, Q&A Copilot, video byte-stream I/O), v1.5.0 (2026-02-12, partitioned Ray executor, embodied-AI video processing, operator-level environment management), v1.5.1 (2026-03-17), v1.5.2 (2026-05-29), v1.5.3 (2026-06-26, VLA embodied operators, Ray repartition pipeline), v1.5.4 (2026-07-17, 9 new person-centric video understanding operators, batch-local stage fusion acceleration).
[Uncertain] The arXiv first-submission date of the Data-Juicer 2.0 paper appears differently across sources — as December 23, 2024, and as January 2025. Based on the arXiv number prefix 2501, the formal posting should have been in January 2025.

### Type (Model/Dataset/Toolchain/Benchmark)

A toolchain (data processing system / operator library / data-model co-development platform), not a generative model or a dataset. It can be broken down into three output tiers:
1. [Operator Library and Execution Engine] The Data-Juicer 2.0 core — as of v1.5.4, 229 operators in total (138 Mappers, 58 Filters, 10 Deduplicators, 8 Formatters, 5 Selectors, 4 Aggregators, 3 Groupers, 3 Pipelines), covering five modalities — text/image/audio/video/multimodal — paired with multiple adaptive execution backends including Ray, MaxCompute, and standalone mode.
2. [Data-Model Co-development Suite] Data-Juicer Sandbox — provides a "Probe-Analyze-Refine" workflow that chains data-recipe search and model training/evaluation into a closed loop, with built-in integration for training and evaluation frameworks such as EasyAnimate, T2V-Turbo, ModelScope, and VBench.
3. [Derived Datasets and Recipes] Taking the Sandbox's text-to-video practice as an example, it has open-sourced a curated optimal T2V data pool (HuggingFace datasets such as datajuicer/data-juicer-t2v-optimal-data-pool) along with the corresponding YAML recipes, an output that is a trinity of "method + tool + data recipe."
Accordingly, within the taxonomy of this survey, Data-Juicer falls under the "data infrastructure" category, and its most direct counterpart is NVIDIA NeMo Curator.

### Openness (Whether Weights/Code/Data/Pipeline Are Each Open-Sourced)

Extremely open-source — the most open object in this survey.
[Code] Fully open-source. GitHub datajuicer/data-juicer (formerly modelscope/data-juicer), Apache 2.0 license, approximately 6.8k stars, with continuous high-frequency iteration (7 versions released within 2026 alone, from v1.4.5 through v1.5.4). The implementations of all 229 operators are fully readable and modifiable; v1.5.3 added 409 new test cases in a single release, reflecting a relatively high engineering standard.
[Operators and Recipes] Open-source. config_all.yaml exposes all operators and their hyperparameters; the DJ-Cookbook maintains 20+ ready-to-use data recipes (including vertical-scenario recipes for video data synthesis, contrastive learning, curriculum learning, etc.).
[Data] Partially open-source. In the text-to-video case study, the resulting optimal data pool is publicly released as a HuggingFace dataset: datajuicer/data-juicer-t2v-optimal-data-pool, containing 147,176 samples, approximately 227.5GB, under an Apache 2.0 license; however, the underlying raw source material (InternVid, Panda-70M, MSR-VTT) must be obtained separately under each dataset's own license — what DJ publicly releases is the filtered sample index and metadata.
[Model Weights] The Sandbox paper states that the T2V model it trained (fine-tuned from T2V-Turbo/EasyAnimate) is open-sourced together with the code and data.
[Pipeline] Fully public. Not only is the method disclosed, but the executable YAML recipe and threshold values (such as the CLIP similarity threshold of 0.306337) are also public, giving it a lower reproduction barrier than the vast majority of model-side work.
[Documentation] A bilingual (Chinese/English) documentation site (datajuicer.github.io), including Operator Schemas with entry-by-entry descriptions.

### Whether Joint Audio-Video Generation Is Supported, and the Implementation Approach (Native Joint/Cascaded/MoE Fusion)

Does not support joint audio-video generation — Data-Juicer itself is not a generative model and produces no audio or video content. However, on the data side it provides operator capabilities that support joint audio-video tasks, which are substantively relevant to this survey's AV thread:
[Audio-side operators] audio_duration_filter (duration filtering), audio_nmf_snr_filter (NMF-based signal-to-noise ratio filtering), audio_size_filter (file-size filtering), audio_add_gaussian_noise_mapper (noise-augmentation), audio_ffmpeg_wrapped_mapper (FFmpeg audio-filter wrapper).
[Cross-modal audio-video operators] video_audio_ASR_mapper (speech recognition/annotation from the audio track), video_tagging_from_audio_mapper (generates video tags from the audio track based on an Audio Spectrogram Transformer), video_captioning_from_audio_mapper (generates video captions from the audio track based on Qwen-Audio), video_audio_detect_age_gender_mapper (detects speaker age and gender from the audio track based on wav2vec2), video_audio_speech_emotion_mapper (speech emotion recognition), video_active_speaker_detect_mapper (active speaker detection by jointly analyzing visual face tracks and the audio signal — the operator closest to "audio-video sync determination" in DJ).
[Positioning judgment] DJ's audio-video capability leans toward "understanding and annotation" rather than "sync quality gatekeeping": it has active speaker detection, but no SyncNet/Synchformer-style operators for sync-offset and confidence scoring, nor any operators for three-way speech/foley/music track separation and mixture control. Therefore, to reuse DJ for building AV joint-generation training sets, the sync-filtering and audio-track-classification steps still need to be extended by the user.
[Empirical case] Its official text-to-video case study (which topped VBench) is a pure text-to-video (T2V) case, with no audio processing involved anywhere in the pipeline.

### List of Research Information Sources (URLs of Papers/Technical Reports/Official Docs/News, with Each Source's Nature Labeled: Official Primary/Same-Team Corroboration/Third-Party Reporting)

1. [Official Primary] Data-Juicer 2.0 paper arXiv abstract page https://arxiv.org/abs/2501.14755 — authors, version history (v1 2024-12-23 / v2 2025-06-04 / v3 2025-10-29), NeurIPS 2025 Spotlight.
2. [Official Primary] Data-Juicer 2.0 full-text HTML https://arxiv.org/html/2501.14755v3 — three-layer architecture, 150+ multimodal operator taxonomy, TB-scale/10k+ CPU-core scale, MinHash Ray dedup performance, speedup data, Alibaba Cloud PAI deployment.
3. [Official Primary] Data-Juicer 2.0 PDF https://arxiv.org/pdf/2501.14755
4. [Official Primary] Data-Juicer Sandbox paper arXiv abstract page https://arxiv.org/abs/2407.11784 — ICML 2025 Spotlight, Probe-Analyze-Refine, topping VBench.
5. [Official Primary] Sandbox full-text HTML https://arxiv.org/html/2407.11784v3 — text-to-video case study, data-pool partitioning, operator selection, VBench score table, quality-vs-compute conclusions.
6. [Official Primary] GitHub main repository https://github.com/datajuicer/data-juicer — version timeline, operator statistics, adopter list, list of related papers, Apache 2.0 license.
7. [Official Primary] Operator schema documentation https://datajuicer.github.io/data-juicer/en/main/docs/Operators.html and https://raw.githubusercontent.com/datajuicer/data-juicer/main/docs/Operators.md — entry-by-entry names and descriptions of the 229 operators, the authoritative source for the list of video/audio operators.
8. [Official Primary] HuggingFace dataset https://huggingface.co/datasets/datajuicer/data-juicer-t2v-optimal-data-pool — 147,176 samples, 12.09% retention rate, 227.5GB, source-dataset composition (InternVid 606k / Panda-70M 605k / MSR-VTT 6k), the two filtering operators and the CLIP threshold of 0.306337.
9. [Official Primary] OpenReview review pages https://openreview.net/forum?id=NiL5U1DrRN (DJ 2.0) and https://openreview.net/forum?id=zIGIvysR1H (Sandbox).
10. [Same-Team Corroboration] Alibaba Cloud developer community article "New No. 1 on VBench for Video Generation! Data-Juicer Sandbox Powers Multimodal Data-Model Co-development" https://developer.aliyun.com/article/1570605 and the same article on the ModelScope community https://community.modelscope.cn/669f1a7b76e87a79e35ada49.html — Chinese-language explanation of the case study.
11. [Same-Team Corroboration] HumanVBench paper https://arxiv.org/html/2412.17574v2 (CVPR 2026) — uses 20+ DJ operators to build a person-centric video annotation pipeline, a real-world use case of DJ's video operators in dataset construction.
12. [Official Primary] ICML 2025 Poster page https://icml.cc/virtual/2025/poster/43484 and the PMLR official version https://proceedings.mlr.press/v267/chen25bm.html
13. [Third-Party Index] ResearchGate https://www.researchgate.net/publication/388421863_Data-Juicer_20_Cloud-Scale_Adaptive_Data_Processing_for_Foundation_Models
14. [Third-Party Reporting] CSDN blog post "Data-Juicer: A Foundation-Model Data Cleaning Framework from Alibaba" https://blog.csdn.net/qq_41895747/article/details/140150556 — Chinese community interpretation, no new primary information.

## Data Scale and Distribution

### Training Data Scale (Number of Videos/Hours/Tokens, Pretraining and SFT Separated)

Data-Juicer does not itself train models, so there is no "training data scale" per se; the relevant figures fall under two separate framings.
[System processing-throughput framing] The Data-Juicer 2.0 paper reports the ability to process TB-scale data and to schedule 10k+ CPU cores; experiments span dataset scales from 560,000 (560K) samples up to 70 billion (70B) samples; one production deployment has run stably for over 5 months, cumulatively processing more than TB-scale data. In distributed benchmarking, processing a 500x-scale dataset on 3,200 Ray-DLC cores took 1,780 seconds, and a 2,500x-scale dataset took 7,083 seconds.
[Text-to-video case-study framing] The Sandbox's T2V case study is the data most directly relevant to this survey. The original candidate pool is composed of three public datasets, totaling approximately 1,217,000 video-text pairs: InternVid 606k + Panda-70M 605k + MSR-VTT 6k. After filtering, the open-sourced optimal data pool contains 147,176 items (about 227.5GB); there is also a larger 228k-item data pool used for the final model that topped VBench (the paper records this configuration as corresponding to 640k training samples, i.e., involving data repetition). Small-scale probing experiments uniformly use a data pool of about 40k samples to control for variables.
[No pretraining/SFT split] As a toolchain, there is no division between pretraining and SFT data volumes.

### Data Source Composition (Proprietary/Public Datasets/Web Crawling/Licensed Acquisition/Synthetic Data)

As a tool, Data-Juicer itself does not hold data; the question of data sourcing manifests in two places.
[Data sources in the official case study] The text-to-video case study uses exclusively public datasets, with no proprietary data, no web crawling, no licensed acquisition, and no synthetic data: InternVid (Shanghai AI Lab, a large-scale YouTube-sourced video-text dataset, contributing 606k), Panda-70M (Snap/UCM, a high-quality video-caption dataset derived from HD-VILA-100M, contributing 605k), MSR-VTT (Microsoft's classic small-scale video description dataset, contributing 6k). The three together total about 1.217 million, showing that in terms of mixture, InternVid and Panda-70M are nearly equal, with MSR-VTT serving only as a minor supplement (0.5%).
[Data-access surface supported by the system] Natively compatible with HuggingFace Datasets, ModelScope datasets, the local filesystem, Alibaba Cloud OSS/NAS/CPFS, AWS S3 (from v1.4.4 onward), and other data sources; supports compressed dataset formats (v1.5.1) and video byte-stream I/O (v1.4.6, facilitating processing of video without writing to disk).
[Synthetic data capability] Of the 229 operators, roughly 50 are dedicated to data synthesis and augmentation; the DJ-Cookbook contains a "video data synthesis" YAML recipe, meaning DJ can not only filter real data but also supports constructing synthetic data — a point of difference from NeMo Curator's cleaning-focused positioning.

### Data Compliance and Provenance (Share of Licensed Data, Rights-Cleared Datasets, C2PA, etc.) ⚠️

[Tool's own compliance] Apache 2.0 license, commercially friendly. The public T2V data pool is likewise labeled Apache 2.0, but what is actually released is the filtered sample metadata/index — the raw video material remains subject to the respective licenses of InternVid, Panda-70M, and MSR-VTT — a common practice for academic video datasets to avoid redistributing copyrighted content.
[Operator capabilities relevant to compliance] DJ provides several operators usable for privacy and compliance processing: video_face_blur_mapper (detects and blurs faces in video, directly serving face-privacy de-identification), video_nsfw_filter (inappropriate-content filtering), video_watermark_filter and video_remove_watermark_mapper (watermark detection and removal, indirectly related to processing copyright markers), and on the text side there are additional sensitive-information de-identification operators. This makes DJ one of the few open-source systems with built-in privacy de-identification operators at the data-processing-framework level.
[Uncertain] Neither the paper nor the documentation mentions support for content-provenance/authentication standards such as C2PA; no governance-oriented features are provided such as data authorization-status tracking, rights-cleared dataset labeling, or license-compatibility checking; nor is the share of authorized data used internally at Alibaba disclosed. Overall, DJ provides "tools for compliant processing" rather than a "framework for compliance governance."

### Clip Duration Distribution and Splitting Strategy ⚠️

[Splitting operators — one of DJ's strengths] Provides three complementary video-splitting operators that can be combined as needed:
  · video_split_by_scene_mapper — splits video into shot-level clips based on scene-change detection (underlying implementation based on the PySceneDetect family of methods), the splitting approach most aligned with building video-generation training data.
  · video_split_by_duration_mapper — splits into equal-length segments by a fixed duration, suitable for quickly constructing fixed-length training samples.
  · video_split_by_key_frame_mapper — splits at keyframe (I-frame) boundaries, with cut points aligned to the coding structure, incurring low decoding overhead.
  There is also video_clip_reassembly_mapper, used to reassemble processing results from overlapping clips back into a long video (aimed at embodied-AI hand-motion scenarios).
[Duration filtering] video_duration_filter supports setting an upper/lower duration bound interval, removing clips that are too short (insufficient information content) or too long (imperfectly split).
[Uncertain] The official T2V case study does not disclose a duration-distribution histogram or average duration for the final data pool; InternVid/Panda-70M/MSR-VTT are themselves already pre-split short-clip datasets (Panda-70M averages about 8.5 seconds, MSR-VTT about 10-30 seconds, InternVid averages about 11 seconds), and no additional secondary splitting was performed in the case study, so the splitting operators were not actually enabled in that case. DJ also does not offer any officially recommended duration-bucketing strategy.

### Resolution/Aspect Ratio Distribution and Bucketing Strategy ⚠️

[Relevant operators] Provides both filtering- and rewriting-type operators, with fairly complete coverage:
  · video_resolution_filter — filters by an upper/lower resolution bound, removing low-definition material.
  · video_aspect_ratio_filter — filters by an aspect-ratio interval, removing extreme long-strip/portrait or anomalous-ratio samples.
  · video_resize_resolution_mapper — rewrites resolution to satisfy width/height constraints (rather than discarding), usable for unifying mixed-resolution data to a target spec.
  · video_resize_aspect_ratio_mapper — resamples video to fall within a specified aspect-ratio interval.
  The "filter + rewrite" dual-path design is a characteristic of DJ relative to purely filter-based frameworks: non-conforming samples can be repaired rather than simply discarded, which is friendlier for scenarios where data volume is precious.
[Uncertain] DJ does not have a built-in "resolution bucketing" feature commonly used in video-generation training — i.e., grouping samples by resolution/aspect ratio for batching within the same training batch — this is considered the responsibility of the training framework side (e.g., EasyAnimate, Diffusers); DJ is only responsible for arranging data into a state ready for bucketing. The official T2V case study also does not disclose resolution or aspect-ratio distribution statistics for the data pool.

### Category/Domain Distribution and Mixture Strategy (Proportional Control and Concept Balancing for People, Actions, Scenes, Styles, etc.) ⚠️

Data-Juicer's approach to category/domain mixture is fundamentally different from that of generative-model teams: it does not presuppose a fixed domain taxonomy, but instead provides mechanisms that make domain distribution measurable, splittable, and searchable, handing mixture decisions off to Sandbox's empirical feedback loop.
[Domain-annotation capabilities]
  · video_tagging_from_frames_mapper — generates video content tags from sampled frames (based on image-tagging models such as RAM), can produce open-vocabulary semantic tags, and is the primary means of profiling domain distribution.
  · video_tagging_from_audio_mapper — generates tags from the audio track based on an Audio Spectrogram Transformer, providing auditory-side domain information (complementary to visual tags).
  · video_tagging_from_frames_filter — retains/removes samples by tag, i.e., targeted filtering based on domain.
  · video_object_segmenting_mapper — text-guided semantic segmentation based on YOLOE + SAM2, can provide object-level domain evidence.
[Dedicated support for person-centric domains] v1.5.4 added 9 person-centric video-understanding operators in a single release, forming a fairly complete person-domain processing chain: video_human_tracks_extraction_mapper (extracts face and body bounding-box tracks), video_face_ratio_filter (filters by the proportion of face within the frame, usable to select close-ups/medium shots/wide shots), video_active_speaker_detect_mapper (active speaker detection), video_captioning_from_human_tracks_mapper (generates descriptions based on person tracks), video_captioning_face_attribute_emotion_mapper (face attribute and emotion descriptions), video_human_tracks_face_demographic_mapper (demographic attributes), video_whole_body_pose_estimation_mapper (2D full-body keypoints for body/hands/feet/face). This chain corresponds directly to the "people, actions" domain — the most important domain in video generation.
[Mixture strategy — empirical search rather than prior design] The Sandbox's Probe-Analyze-Refine workflow is essentially an automated search for data mixture: for each candidate operator i, the data pool is evenly split by its statistic into three equal-sized sub-pools — low/middle/high (P_i,low / P_i,middle / P_i,high) — plus a randomly sampled control pool; reference models are trained on each and evaluated with VBench, and operators and their value ranges are ranked accordingly; the top-ranked operators are then combined into 2^n−1 cross-combined data pools (a pyramid structure, where higher layers have higher quality but fewer samples), continuing the search for the optimal combination. This process turns "which segment of the distribution should be retained on which dimension" into an experimental question that can be answered directly by model feedback, rather than relying on manually set proportions.
[Mixture-related recipes in the DJ-Cookbook] Maintains a "curriculum learning based on data difficulty" recipe and a contrastive-learning recipe, both ready-made templates for domain/difficulty mixture scheduling.
[Uncertain] DJ has not officially published any recommended domain taxonomy for video generation (e.g., target ratios for people/actions/scenes/styles), nor did it report category-distribution statistics for the final data pool in the T2V case study; no dedicated operator support is documented for strategies such as concept balancing or long-tail category resampling.

### Audio Category Distribution and Mixture (Proportions and Control Strategy for Speech/Foley/Music/Ambient Sound/Silence) — A Dimension Unique to AV Models ⚠️

[Uncertain] Data-Juicer does not provide classification or mixture-control capability for audio categories (speech/sound-effect foley/music/ambient sound/silence), which is a clear gap relative to AV-generation data pipelines.
[Existing audio-side capabilities are limited to]
  · Quality dimension: audio_nmf_snr_filter (estimates signal-to-noise ratio via non-negative matrix factorization and filters by interval), audio_duration_filter (duration), audio_size_filter (file size).
  · Semantic-annotation dimension: both video_tagging_from_audio_mapper and video_audio_ASR_mapper are based on the Audio Spectrogram Transformer (AST, trained on AudioSet) to produce audio-event tags — the AudioSet ontology contains 527 categories including speech/music/ambient sound and others, so in theory an approximate three-way distinction could be derived, but DJ does not package this as an explicit three-class field or mixture-control operator.
  · Speaker-attribute dimension: video_audio_detect_age_gender_mapper (wav2vec2-based age/gender detection), video_audio_speech_emotion_mapper (speech emotion).
[Missing key capabilities] No audio-track source-separation operator (e.g., Demucs/Bandit-style speech/effects/music three-way separation), so it cannot do Foley-Omni-style field-level energy gating and track-category mixture control; no silence detection or silence-ratio-threshold operator; no dedicated "remove samples without an audio track" operator (must be achieved indirectly via the audio_duration/size filter); no music/vocal separation for background-music stripping.
[Empirical evidence] Its only official video case study (the VBench T2V topper) is a purely visual task, with no audio operator enabled throughout, indicating that its audio-side capability has not yet been battle-tested at scale in video-generation scenarios. To use DJ to build training data for joint audio-video generation, the audio-category taxonomy would need to be extended with custom operators — fortunately DJ's operator-interface design (Mapper/Filter base classes + YAML registration) makes extension relatively low-cost, which is also one of the programmability selling points the team claims.

### Narrative Structure Distribution (Single-Shot vs. Multi-Shot, Average Clip Duration, Shot-Count Distribution, Whether Native Audio Tracks Are Included) ⚠️

[Capability for handling single-shot vs. multi-shot] DJ provides, via video_split_by_scene_mapper (scene-change-detection splitting), a standard means of decomposing multi-shot long videos into single-shot clips — the most direct lever for controlling narrative structure: enabling this operator means the training data becomes entirely single-shot; not enabling it preserves the original multi-shot structure. DJ leaves this choice to the user, with no default orientation.
[Actual structure in the official case study] The three source datasets in the T2V case study (InternVid, Panda-70M, MSR-VTT) are themselves already pre-split into single-shot or near-single-shot short clips, and no additional scene splitting was performed in the case study, so the final data pool is predominantly single-shot short clips.
[Whether native audio tracks are included] The case study neither requires nor reports statistics on this; DJ supports processing video both with and without audio tracks, and splitting operators such as video_split_by_scene_mapper handle the audio track synchronously.
[Uncertain] No shot-count distribution, average clip duration, or proportion-of-samples-with-audio-track statistics are reported anywhere; nor does DJ provide "shot-count statistics" or "narrative-structure profiling" analysis operators (its data-analysis reports are mainly histograms of single-value statistics rather than structured narrative attributes).

### Language/Accent Distribution (Data Foundation for Multilingual Lip-Sync Capability) ⚠️

[Uncertain] Data-Juicer provides no analysis or control capability for language/accent distribution in video/audio, and the official T2V case study does not address the language dimension either.
[Indirectly relevant capabilities]
  · Text side: DJ has included language_id_score_filter (fastText-based language identification and confidence filtering) since 1.0, which can filter caption text by language, but this is text language rather than spoken language.
  · Speech side: video_audio_ASR_mapper can perform speech recognition on the audio track, and the multilingual capability of the underlying ASR model determines the range of transcribable languages, but DJ does not package "the recognized language" as a structured, filterable field, nor does it have an accent-annotation operator.
  · Speaker attributes: existing operators cover age, gender, and emotion, but not language or accent.
[Assessment] For multilingual lip-sync — a key capability for AV generation — DJ currently offers no data-side support, requiring the user to extend it independently. This is related to its operator-evolution trajectory — the new operators added in 2026 have concentrated on two directions, embodied AI (camera calibration, pose, hand reconstruction) and person-centric video understanding, with the speech-language dimension not yet prioritized.

## Cleaning Pipeline

### Overall Structure of the Filtering Funnel (Number of Filtering Stages, Stage Ordering)

Data-Juicer's filtering funnel is not a fixed pipeline, but rather "a configurable directed graph of operators + an optimal operator combination searched out via model feedback." This is the essential difference between it and pipelines self-built by model teams such as Apollo or UniVerse-1.
[Structural level — three-layer system architecture (DJ 2.0)]
  1. Capability Layer: expands DJ 1.0's roughly 50 text-only pretraining operators to 150+ (reaching 229 by v1.5.4) multimodal operators, covering text/image/audio/video/multimodal, and supporting post-training tasks. About 90% of the operators are Mappers and Filters, about 75% involve multimodal processing, and about 50 are for data synthesis and augmentation.
  2. Interface Layer: multi-tier APIs — a low-level Python API, RESTful endpoints, a visual editor (an Alibaba Cloud PAI Designer component), and natural-language interaction based on AgentScope agents (issuing data-processing tasks via conversational instructions).
  3. Runtime Layer: a unified Data-Juicer-Dataset abstraction, operator decoupling and adaptive optimization, fault-tolerant execution control, hardware-agnostic adapters (automatic switching among standalone / Ray / Alibaba Cloud MaxCompute / PAI-DLC and other backends).
[Process level — the Sandbox's Probe-Analyze-Refine workflow] This is the methodology DJ uses to "design the funnel," itself constituting a meta-process:
  · Probe: for each candidate operator, split the dataset by its statistic into three equal-sized pools — low/middle/high — plus a randomly sampled control pool; on each pool, train a reference model (small-scale, low-cost) under the same budget.
  · Analyze: compare the performance of models trained on each pool using a unified evaluation benchmark (16 metrics from VBench on the video side), ranking operators and their value ranges accordingly, identifying which operators genuinely bring gains.
  · Refine: cross-combine the top n ranked operators into 2^n−1 data pools (a pyramid structure — higher layers simultaneously satisfy more operator conditions, yielding higher quality but fewer samples); then validate the optimal combination at scale, comparing two scale-up routes — "repeating high-quality data" versus "progressively descending the pyramid to include suboptimal data + deduplication."
[The Sandbox is itself a three-layer design] The top layer is YAML-configuration-driven workflow orchestration (four stages: probe/refine/execute/evaluate); the middle layer provides Hook functions offering common development behaviors; the bottom layer is factory classes exposing 100+ operators and training/evaluation capabilities.
[The final funnel adopted in the text-to-video case study] After the above search, an extremely minimal two-stage filter was settled on: video_nsfw_filter (NSFW score filtering, underlying model Falconsai/nsfw_image_detection) → video_frames_text_similarity_filter (computes CLIP similarity between sampled frames and text, minimum threshold 0.306337). This filtered approximately 1.217 million items down to 147,176. Notably, this final recipe is far simpler than the dozen-plus-stage funnels used by most model teams — the Sandbox's search concluded that "a few genuinely relevant operators + strict thresholds" outperforms "piling on a large number of filter conditions."
[Engineering optimization — automatic reordering of the funnel] DJ 2.0 introduces workload-aware OP reordering: under the constraint of operator commutativity, computationally cheap operators are automatically moved earlier in the pipeline, reducing processing time by up to 70.22%. This turns "filter ordering" from manual experience into automated system optimization — a rare capability in this survey.

### Quantitative Funnel Retention Rate (Input/Output Volume per Filtering Stage and Final Retention Rate, e.g., Apollo's 27%) ⚠️

[The only officially published quantitative retention rate — the text-to-video case study]
  · Input: InternVid 606k + Panda-70M 605k + MSR-VTT 6k ≈ 1,217,000 video-text pairs.
  · Output: 147,176 items, approximately 227.5GB.
  · End-to-end retention rate: 12.09% (explicitly labeled on the HuggingFace dataset card).
  This figure is in the same order of magnitude as Apollo's 27% and UniVerse-1's roughly 19%, but stricter, indicating that the optimal strategy found by the Sandbox search tends toward "better to have less than too much" — retaining only about 1/8 of the data.
  · There is also a 228k-item data pool used for the final model that topped VBench (retention rate about 18.7%); the paper records this configuration as corresponding to 640k training samples, i.e., about 2.8x data repetition on top of the 228k base.
[Stage-by-stage breakdown] [Uncertain] The elimination amounts of the two operators individually are not disclosed — i.e., how much video_nsfw_filter alone eliminates, and how much video_frames_text_similarity_filter alone eliminates. Inferring from the nature of the operators, the NSFW filter's elimination rate on these three already lightly-cleaned public datasets should be relatively low (possibly a single-digit percentage), with the 12.09% retention rate mainly attributable to the CLIP-similarity threshold of 0.306337 — i.e., the vast majority of samples were removed for "insufficient alignment between the video imagery and the text description." This inference is consistent with the threshold value itself: 0.306 is a relatively high bar for CLIP similarity.
[Pool-partitioning convention during the probe stage] During Probe, each single-operator pool is split into three equal thirds by low/middle/high, i.e., each pool's retention rate is fixed at 33.3%; this is an equal-size constraint set for fair comparison and does not represent an actual recommended retention rate.
[System-level throughput framing of "retention rate"] The DJ 2.0 paper focuses on processing efficiency rather than retention rate, and does not give a cross-project general retention-rate statistic.

### Shot Splitting Method (PySceneDetect/In-House Model/Shot-Aware Splitting)

Shot splitting is one of the more fully supported aspects of DJ's video operators, offering three strategies to choose from:
  · video_split_by_scene_mapper — "splits video into scene clips based on detected scene changes," i.e., standard shot-boundary-detection splitting, built on the scenedetect (PySceneDetect) library, supporting its ContentDetector / ThresholdDetector / AdaptiveDetector and other detectors with configurable thresholds. This is fully consistent with the mainstream industry approach (Apollo, Movie Gen, Open-Sora, and others generally use PySceneDetect).
  · video_split_by_duration_mapper — mechanical splitting at a fixed duration, agnostic to shot boundaries, suitable for material already known to be single-shot long video, or scenarios insensitive to exact cut points.
  · video_split_by_key_frame_mapper — splits at the video encoding's keyframes (I-frames). Advantage: cut points align with the coding structure, no re-encoding needed, very fast; disadvantage: I-frame placement is determined by the encoder and may not correspond to actual shot boundaries.
[Auxiliary operator] video_clip_reassembly_mapper is used to stitch results computed separately on overlapping clips (such as hand-motion trajectories) back onto the long-video timeline, serving embodied-AI long-horizon annotation scenarios.
[Engineering aspect] From v1.4.6 onward, video byte-stream I/O is supported, allowing the splitting process to complete in-memory without repeatedly writing to disk, substantively improving I/O overhead for large-scale splitting.
[Actual usage] The official T2V case study did not enable the splitting operators (the source datasets were already pre-split). Thus, while DJ's splitting capability is comprehensive, it lacks public real-world data on ultra-large-scale, long-video-corpus operation (such as splitting throughput or average clips produced per hour of video).

### Quality Filtering (Aesthetic Scoring, Sharpness, OCR Text Filtering, Letterbox/Watermark/Logo Detection) ⚠️

Quality filtering is the aspect of DJ's video operators with the densest coverage — 16 video-side Filters in total, which can be categorized by dimension as follows:
[Aesthetics and perceptual quality]
  · video_aesthetics_filter — computes an aesthetic score on specified sampled frames and filters by interval, based on an aesthetic-scoring model from the LAION/improved-aesthetic-predictor family, with configurable frame-sampling strategy (uniform sampling/keyframe) and any/all aggregation logic (any single frame qualifying vs. all frames qualifying). This is one of the three major candidate operators in the Sandbox's probing experiments.
[Basic specifications]
  · video_resolution_filter — resolution upper/lower bound.
  · video_aspect_ratio_filter — aspect-ratio interval.
  · video_duration_filter — duration interval.
[Text and overlays]
  · video_ocr_area_ratio_filter — "detects the text-area proportion of specified frames and filters by interval"; the documentation explicitly states that "video with a low OCR text-area proportion tends to be higher quality," used to remove burned-in subtitles, on-screen comments, slides, and poster/text-image samples. This is a standard component of video-generation data cleaning.
  · video_watermark_filter — "retains videos with a high probability of being watermark-free," based on a watermark-detection model score for filtering.
  · video_remove_watermark_mapper — a complementary repair path: given a list of watermark-region coordinates (x1,y1,x2,y2 format), directly erases the watermark, thereby salvaging otherwise high-quality material with a watermark rather than discarding the entire item.
[Content safety]
  · video_nsfw_filter — NSFW score filtering (see safety_filtering for details).
[Semantic alignment]
  · video_frames_text_similarity_filter — CLIP similarity filtering between sampled frames and text (see model_as_data_judge and av_sync_detection entries for details).
[Person-related]
  · video_face_ratio_filter — filters by the interval of the proportion of frame occupied by faces, usable to distinguish close-up/medium/wide shots, or to remove extreme samples with no person or a screen full of faces.
  · video_tagging_from_frames_filter — retains or removes samples by content tags.
[Motion]
  · video_motion_score_filter / video_motion_score_raft_filter / video_motion_score_ptlflow_filter (see motion_filtering for details).
[Design-feature assessment]
  1. "Filter + repair" dual paths: for watermarks, resolution, and aspect ratio, both a Filter (discard) and a Mapper (rewrite/repair) operator are provided in parallel, letting the user choose a strategy based on how scarce the data is. This is relatively rare among comparable frameworks.
  2. All thresholds are externalized as YAML parameters with no hard-coded default strategy: DJ does not decide "what the aesthetic-score cutoff should be" on the user's behalf, but instead uses model feedback from the Sandbox's probing experiments to determine it. This is its methodological stance — thresholds should be searched for, not prescribed.
  3. [Uncertain] Not built in are letterbox/pillarbox detection, dedicated blur/sharpness scoring (such as Laplacian variance or video quality assessment models like MUSIQ/DOVER), encoding-bitrate filtering, and other items common in model teams' pipelines; nor is a multi-metric weighted composite scoring mechanism provided — all filtering is a serial stack of single-metric independent thresholds.

### Motion Filtering (Optical Flow/Motion Score Thresholds, Removal of Static and Jittery Content) ⚠️

Motion filtering is one of the few aspects where DJ provides three parallel implementations, reflecting the team's emphasis on this dimension:
  · video_motion_score_filter — computes a motion score based on OpenCV optical-flow estimation (Farneback dense optical flow), filtered by interval. Fast, no GPU required, the default option.
  · video_motion_score_raft_filter — computes a motion score based on the RAFT (Recurrent All-Pairs Field Transforms) deep optical-flow model. Significantly more accurate than traditional optical flow, especially in large-displacement and occlusion scenarios, at the cost of requiring GPU inference.
  · video_motion_score_ptlflow_filter — based on the ptlflow (PyTorch Lightning Optical Flow) library, which can connect to dozens of optical-flow models within it (FlowNet, PWC-Net, GMA, FlowFormer, etc.), offering the greatest model-selection flexibility.
[Significance of the three coexisting] Users can trade off between "OpenCV fast but coarse → RAFT accurate but slow → ptlflow arbitrary choice" according to compute budget and precision needs; this design of multiple implementation tiers for the same function is a bonus point in data-processing frameworks — using OpenCV for large-scale coarse screening and RAFT for fine screening is a natural two-stage usage pattern.
[Filtering semantics] All three are bilateral interval filters (min_score / max_score): the lower bound removes static or near-static footage (if a video-generation model learns from too many static samples it will tend to generate "PPT-like" motionless video); the upper bound removes clips with excessively intense motion, jitter, or rapid cuts (which distort optical-flow estimation and make temporal modeling difficult).
[Actual usage] [Uncertain] The official T2V case study's final recipe does not include a motion-score operator — in the Probe-stage operator ranking, video_aesthetics_filter, video_nsfw_filter, and video_frames_text_similarity_filter were the three candidates, and the latter two ultimately won out. The paper does not report the probing rank of the motion-score operators in this case study or the reason they were not selected, so it cannot be determined whether the motion dimension simply produced no significant gain on this dataset, or was never entered as a candidate at all. No recommended threshold values are published either.

### Deduplication Method (Exact Deduplication and Embedding-Based Semantic Deduplication Recorded Separately) ⚠️

Deduplication capability is unevenly distributed: very strong on the text side, weaker on the video side.
[Video deduplication — only exact deduplication supported]
  · video_deduplicator — "document-level deduplication using exact matching of videos," i.e., exact deduplication based on a content hash of the video file, only able to identify completely identical files, unable to identify near-duplicate samples that have been transcoded, cropped, watermarked, or resized.
  · ray_video_deduplicator — the Ray distributed version of the above, with identical logic but able to parallelize across nodes, suited to TB-scale operation.
  [Uncertain] DJ provides no video-embedding-based semantic-deduplication operator (e.g., CLIP/VideoMAE features + ANN retrieval), nor any perceptual-hash (pHash/dHash) style near-duplicate detection. This is a clear shortfall for video-generation data cleaning — in web-scraped video corpora, near-duplicates (the same material from different sources, different compression, with different intros/outros added) typically far outnumber byte-level exact duplicates. When using DJ to build large-scale video corpora, semantic deduplication would need to be added by the user.
[Text-side deduplication — a strong engineering capability]
  · MinHash locality-sensitive-hashing deduplication (Ray distributed version): uses a "load-balanced union-find" implementation, achieving a 3.3x speedup over the native Ray implementation; measured to process 5TB of data in 2.8 hours on 8 Ray nodes. The paper compares this with NVIDIA NeMo Curator (which uses 64 A100 GPUs to process 1.1TB in 1.8 hours), intended to show that DJ achieves comparable cost-effectiveness using pure CPU. Scalability data: when data volume increases 5x, time increases 4.02x-5.62x (near-linear); when core count doubles, time drops to 58.9%-67.1% of the original.
  · v1.5.2 added cross-document line-level deduplication, refining the granularity down to the line level.
  · There are also SimHash, exact-hash, document-level/paragraph-level, and other deduplication operators, with 10 Deduplicator classes in total.
[Image side] Provides image-hash-based deduplication operators.
[Overall assessment] DJ's deduplication strength lies in "distributed engineering capability" rather than "video semantic-perception capability" — it solves "how to run deduplication efficiently across 10k cores," not "how to judge whether two video clips are semantically duplicate."

### VLM/LLM as Quality Inspector (Multimodal LLM Quality Scoring and Mismatch Removal, the 2026 Trend from Shallow Scorers Toward LLM-Based Semantic Judgment) ⚠️

Data-Juicer is a typical practitioner of the philosophy "using foundation models to process data" (the "with" in "for and with Foundation Models"), and its approach differs methodologically from model teams: rather than using one large model to give data a composite score, it packages a large number of specialized models as independent operators, then uses downstream training feedback to decide which operators to trust.
[List of model operators serving as quality inspectors (video side)]
  · CLIP — video_frames_text_similarity_filter, computes cross-modal similarity between sampled frames and the text description, the core criterion ultimately adopted in the T2V case study (threshold 0.306337). Essentially uses CLIP as a "video-text alignment inspector," removing samples where the caption does not match the imagery.
  · Falconsai/nsfw_image_detection — the underlying model of video_nsfw_filter, serving as a safety inspector.
  · Aesthetic-scoring model — video_aesthetics_filter, serving as a visual-taste inspector.
  · RAM/image-tagging model — video_tagging_from_frames_mapper, serving as a content annotator.
  · Audio Spectrogram Transformer — video_tagging_from_audio_mapper / video_audio_ASR_mapper, audio-side annotators.
  · Qwen-Audio — video_captioning_from_audio_mapper, audio understanding and description.
  · General-purpose VLM — video_captioning_from_vlm_mapper, "uses a VLM that can directly accept video input to generate video descriptions"; this is the operator in DJ that comes closest to the 2026 trend of "LLM-based semantic judgment," and supports connecting to any video VLM.
  · YOLOE + SAM2 — video_object_segmenting_mapper, object-level semantic evidence.
  · wav2vec2 — speaker age/gender detection.
["Model feedback as judge" meta-level design — DJ's most distinctive feature] The Sandbox's Analyze stage essentially "uses the downstream generative model's evaluation scores to score the quality inspectors themselves": each candidate operator first splits the data into low/middle/high pools, a reference model is trained on each, and VBench evaluation results are used to rank them, thereby objectively determining "whether this particular inspector actually helps the final generation quality." This mechanism answers a question that model-team pipelines commonly avoid — "does the filter I added actually help?" The T2V case study's conclusion was reached exactly this way: CLIP-alignment filtering and NSFW filtering were found effective and retained, while the other candidate operators were not selected. This "verify-then-adopt" stance is more rigorous than simply piling on VLM scorers, and is the point in DJ's methodology of the greatest reference value for this survey.
[Other forms of LLM involvement in data processing] From v1.4.5 onward, a Ray + vLLM pipeline is supported, enabling efficient batch invocation of local large models for annotation/synthesis in a distributed environment; v1.4.6 introduced a "Q&A Copilot"; the interface layer supports issuing data-processing instructions in natural language via AgentScope agents; v1.5.2 added an agent data-quality toolkit.
[Uncertain] DJ does not include operators based on dedicated video-quality-assessment large models like VideoScore or DOVER, nor does it offer an open-ended quality-inspection operator template in the style of "VLM composite scoring with rationale."

### Safety and Compliance Filtering (NSFW, Copyright, Face/Privacy) ⚠️

[Visual safety] video_nsfw_filter is the only video-side safety operator, "retains samples whose NSFW score falls within a specified interval," based on the Falconsai/nsfw_image_detection image classification model (scoring sampled frames individually and then aggregating). This operator was validated as effective during the Probe stage in the T2V case study and incorporated into the final recipe; the official description is "uses the VideoNSFWFilter operator to ensure high quality" — i.e., in this case study it served both a safety and a quality role simultaneously (samples with high NSFW scores are often also low-quality, unnatural content).
[Privacy protection] video_face_blur_mapper detects and blurs faces in video, a relatively rare built-in privacy de-identification operator at the data-processing-framework level. Combined with video_human_tracks_extraction_mapper (face/body track extraction), track-level continuous de-identification can be achieved. On the text side there are also operators for de-identifying sensitive information (phone numbers, emails, ID numbers, etc.).
[Copyright-related] video_watermark_filter and video_remove_watermark_mapper can identify and process watermarks, indirectly related to copyright markers, but do not constitute a copyright-ownership determination capability.
[Uncertain] Missing components include: no copyright-content recognition/fingerprint-matching capability; no support for provenance-authentication standards such as C2PA; no detection operators for finer-grained safety categories such as violence, hate, or minors (only a binary NSFW score); the additional safety-review system layered on top for internal Alibaba use is not disclosed. As an open-source framework, DJ provides "assemblable safety components"; a complete compliance system must be built by the user themselves.

## Annotation Methods

### Caption Model Used (In-House VLM/Open-Source Model, Model Scale) ⚠️

Data-Juicer does not develop its own caption model in-house; instead it packages multiple caption models as pluggable operators, constituting one of the richest open-source frameworks in this survey for video-captioning operators. There are 6 video-captioning-related Mappers in total:
  · video_captioning_from_vlm_mapper — "uses a VLM that can accept video input to generate video descriptions." The most general-purpose interface, connectable to any video VLM (Qwen2.5-VL, InternVL, Video-LLaVA, etc.), the operator corresponding to the mainstream 2026 approach. Model scale is chosen by the user.
  · video_captioning_from_video_mapper — uses a HuggingFace video-to-text model to generate descriptions (such as the VideoBLIP family).
  · video_captioning_from_frames_mapper — samples frames and uses an image-to-text model (the BLIP/BLIP-2 family) to generate a description per frame, then aggregates. Lowest cost, but lacks temporal understanding.
  · video_captioning_from_audio_mapper — "generates video descriptions from the audio track based on Qwen-Audio." An auditory-side caption, a key piece for building joint audio-video annotation.
  · video_captioning_from_summarizer_mapper — "generates video descriptions by summarizing multiple already-generated pieces of text." This is DJ's most well-designed captioning operator: it aggregates outputs from the various preceding paths (frame descriptions, audio descriptions, OCR text, content tags, ASR transcripts) and hands them to an LLM for summary fusion, producing a single comprehensive, dense description. This "multi-source separate outputs → LLM fusion" path is consistent with the multi-model fusion captioning approach used by teams such as Movie Gen and Panda-70M.
  · video_captioning_from_human_tracks_mapper and video_captioning_face_attribute_emotion_mapper — person-centric targeted descriptions (added in v1.5.4); the former generates descriptions based on person tracks, the latter generates facial-attribute and emotion descriptions for each tracked person.
[Companion annotation-type operators] video_tagging_from_frames_mapper (visual tags), video_tagging_from_audio_mapper (audio tags, AST), video_audio_ASR_mapper (speech recognition) — these do not produce natural-language descriptions but do produce structured tags, and are the input sources for the summarizer operator.
[Actual usage] [Uncertain] The official T2V case study did not enable any captioning operator — the three source datasets already come with their own captions (InternVid generated with BLIP-2 + Tag2Text, Panda-70M generated with a multi-model caption fusion + retrieval-based selection, MSR-VTT with human annotation); the case study only performed CLIP-alignment filtering without recaptioning. Thus, while DJ's captioning-operator chain is complete, it lacks public validated effectiveness and a recommended configuration at large-scale video-generation-corpus scale. DJ has also not officially published a recommended model choice or prompt template for video captioning.

### Caption Density and Degree of Structuring (Short/Long/Dense Descriptions, Structured Fields such as Camera Motion, Style Tags) ⚠️

[DJ's stance: provide assemblable components, do not prescribe a caption paradigm] DJ does not presuppose how long a caption should be or which fields it should contain; instead it provides a full spectrum of operators from "short tags" to "long dense descriptions," to be combined by the user as needed.
[Spectrum of caption density achievable]
  · Sparsest: video_tagging_from_frames_mapper / video_tagging_from_audio_mapper produce a tag vocabulary (such as the 527 AudioSet classes or the open-vocabulary RAM tags), not sentences.
  · Medium: video_captioning_from_frames_mapper / from_video_mapper produce single-sentence or multi-sentence conventional descriptions.
  · Densest: video_captioning_from_summarizer_mapper produces a comprehensive long description by fusing multiple sources; video_captioning_from_vlm_mapper can produce descriptions of arbitrary length and structure via prompt control.
[Support for structured fields] DJ's data model is "sample = multimodal fields + an arbitrary number of stats/tags fields," with each operator writing its output into its own independent field rather than merging into caption text. This means structured information is naturally stored field by field: shot-level aesthetic score, motion score, OCR-area ratio, face ratio, content tags, speaker age/gender/emotion, camera intrinsics and pose, depth, full-body keypoints — all are parallel structured fields that can be concatenated into a condition text or used as separate conditioning channels by the downstream training framework as needed. This "fields-first" design is actually more highly structured than most model teams' pipelines, which produce only a single long block of text.
[Uncertain] DJ does not provide an officially recommended video-generation caption template (e.g., whether it should include fixed fields such as camera motion, lighting, style, subject action), nor does any public case study report average caption length, field composition, or comparative results across different caption structures. This is an inevitable consequence of its "tool-neutral" positioning, but it also means users cannot obtain caption-design best-practice guidance from DJ.

### Joint Audio-Video Caption Structure (Whether Both Visual and Auditory Tracks Are Covered, Whether Split into Separate Fields, e.g., LTX-2's Full Soundscape Description, Script-a-Video's Factorized Streams, Foley-Omni's Three Fields) ⚠️

[Uncertain] Data-Juicer does not define any joint audio-video caption schema, which is a notable gap relative to dedicated works like LTX-2 (full soundscape description), Script-a-Video (factorized streams), and Foley-Omni ([WORDS]/[AUDIO]/[MUSIC] three-field structure).
[But it does have components from which a joint schema could be assembled] DJ's fields-based data model allows a user to construct a similar schema themselves:
  · Auditory-side content description: video_captioning_from_audio_mapper (Qwen-Audio-based description generated from the audio track) → usable as a "soundscape description" field.
  · Auditory-side tags: video_tagging_from_audio_mapper (AST, 527 AudioSet classes) → can approximately distinguish speech/music/ambient sound, usable as an audio-category field.
  · Speech content: video_audio_ASR_mapper → usable as a dialogue-transcript field.
  · Speaker attributes: video_audio_detect_age_gender_mapper, video_audio_speech_emotion_mapper → usable as a speaker-attribute field.
  · Visual-side description: video_captioning_from_vlm_mapper / from_frames_mapper → visual description field.
  · Fusion: video_captioning_from_summarizer_mapper can hand the above fields to an LLM to fuse into a single unified description — this is in effect a feasible implementation of the "unified full-soundscape + visual" description route.
[Key gaps] No audio-track source-separation operator, so it cannot, like Foley-Omni, split the audio track into speech/effects/music stems and describe and validate them by field; no acoustic energy-gating mechanism to correct for VLM visual hallucination (e.g., writing "there is music" upon seeing an instrument in frame). These two points are key steps for building high-quality AV joint annotation, which DJ currently does not provide.
[Conclusion] DJ provides most of the raw material for building AV joint captions, but does not provide the schema design or cross-modal cross-validation methodology; the user must independently fill in the remaining roughly 30% of key components (source separation + energy verification + schema definition).

### Dialogue Transcription and Speaker Attribute Annotation (ASR Transcription + Speaker Identity/Language/Accent/Emotion) ⚠️

[Transcription capability] video_audio_ASR_mapper — performs speech recognition on the video's audio track, described in the documentation as "generates video tags from the audio stream based on the Audio Spectrogram Transformer" (this operator's documented description overlaps with the tagging-type operators, but it actually handles both ASR and audio-event-tagging duties). Can produce a speech transcript as an independent field.
[Speaker attributes — a set of capabilities DJ covers relatively well]
  · video_audio_detect_age_gender_mapper — "detects age and gender from the video's audio signal based on a pretrained wav2vec2 model." Produces the speaker's age bracket and gender attribute, relatively rare among open-source data-processing frameworks.
  · video_audio_speech_emotion_mapper — speech emotion recognition, producing an emotion-label field.
  · video_active_speaker_detect_mapper — "detects the active speaker in a video by analyzing visual face tracks and the audio signal." This is a key operator for binding "the speaker" from the audio track to a specific person on screen, used in multi-speaker scenarios to determine "who is speaking at this moment."
  · video_captioning_face_attribute_emotion_mapper — generates a natural-language description of facial attributes and emotion for each tracked person, providing visual-side emotion evidence that can be cross-validated against speech emotion.
  · video_human_tracks_face_demographic_mapper — demographic attributes of person tracks.
  · video_human_tracks_extraction_mapper — extracts face and body bounding-box tracks, a common prerequisite for the above person-centric operators.
[Attribute-coverage checklist] Age ✓, gender ✓, emotion ✓ (both audio and visual paths), speaker identity (track-level ID) ✓, speaker-to-frame binding ✓; language ✗, accent ✗.
[Application evidence] HumanVBench (CVPR 2026, same team) is explicitly built on Data-Juicer, using 20+ SOTA operators to construct a "Human-Centric Video Annotation Pipeline," with the benchmark covering four dimensions — emotion perception, person identification, behavior analysis, and speech-visual alignment. This is a complete real-world landing case for the above operator chain in constructing an actual dataset, and indirectly indicates that this set of operators already has production-level usability.
[Uncertain] The specific model choice and multilingual coverage of the underlying ASR are not disclosed; no transcription-quality assessment or confidence-based filtering mechanism is provided.

### Geometric and Structured Annotation (Camera Parameters, Depth, 3D Point Tracks, Action Annotation, Explicit State) ⚠️

This is the direction in which Data-Juicer has invested most heavily in 2026, and also the most distinctive relative to other data-processing frameworks — driven by embodied-AI (Embodied AI / VLA) requirements, with v1.4.5, v1.5.0, and v1.5.3 successively adding a large number of geometric and structured-annotation operators across three versions. For video generation, this set of operators corresponds precisely to a "second annotation paradigm beyond text captions," in the vein of SceneScribe-1M and SpatialVID.
[Camera-parameter annotation — three implementations provided]
  · video_camera_calibration_deepcalib_mapper — computes the intrinsics and field of view (FOV) of a static camera based on DeepCalib.
  · video_camera_calibration_droidcalib_mapper — extracts camera intrinsics from video based on DroidCalib.
  · video_camera_calibration_moge_mapper — computes intrinsics and FOV based on MoGe-2.
  · video_undistort_mapper — undistorts the raw video using previously computed intrinsics and distortion coefficients.
[Camera pose/trajectory]
  · video_camera_pose_megasam_mapper — "extracts camera pose by combining MegaSaM and MoGe-2." MegaSaM is a 2025 dynamic-scene SLAM method capable of estimating camera trajectories on in-the-wild video containing moving objects — precisely the core capability needed for "camera motion" conditioning annotation in video generation.
[Depth]
  · video_depth_estimation_mapper — performs depth estimation on video, producing a dense depth-map sequence.
[Body and hands — an action-annotation chain]
  · video_whole_body_pose_estimation_mapper — extracts 2D full-body pose estimation covering body, hand, foot, and face keypoints.
  · video_hand_reconstruction_hawor_mapper — hand reconstruction based on HaWoR + MoGe-2.
  · video_hand_reconstruction_mapper — hand localization and reconstruction based on the WiLoR model.
  · video_hand_motion_smooth_mapper — smooths hand motion in world coordinates and removes outliers.
  · video_hand_action_compute_mapper — "computes 7-DoF actions and 8-dimensional state from hand-reconstruction results and camera pose." This is explicit "state" annotation — converting video into a robot-usable action-state sequence, the standard format for VLA training data.
  · video_atomic_action_segment_mapper — segments unified hand trajectories into atomic action clips, i.e., action-level temporal segmentation annotation.
  · video_trajectory_overlay_mapper — "samples frames and overlays hand trajectories, preparing frames readable by a VLM." Overlays a visualization of the geometric trajectory onto the frame before feeding it to a VLM — a bridging technique between geometric annotation and semantic annotation.
  · video_clip_reassembly_mapper — reassembles hand-action results computed on overlapping clips back onto the long-video timeline.
[Object level]
  · video_object_segmenting_mapper — text-guided semantic segmentation based on YOLOE + SAM2, producing a mask sequence for objects across the whole video.
[Assessment] This operator chain makes DJ currently the most complete open-source framework for geometric annotation capability: camera intrinsics, distortion, pose trajectory, depth, full-body/hand pose, 7-DoF action, 8-dimensional state, atomic action segmentation, and object segmentation are all present, with heavy use of state-of-the-art 2025-2026 models (MegaSaM, MoGe-2, HaWoR, WiLoR, SAM2, YOLOE). Although designed for embodied AI, these operators are directly reusable for controllable video generation requiring camera control, depth conditioning, or action conditioning.
[Uncertain] No dedicated operator is seen for 3D point tracks (long-horizon point trajectories such as CoTracker/SpatialTracker-style tracking); the throughput, cost, and real-world effectiveness of this set of operators on large-scale video-generation corpora are not disclosed.

### Synthetic Data Construction (Controlled Perturbation/Editing to Construct Training Pairs, e.g., InstructAV2AV) ⚠️

[Scale of synthesis capability] Of the 229 operators, about 50 are dedicated to data synthesis and augmentation, one of DJ's four major capability areas (analysis, synthesis, annotation, post-training). This proportion is noticeably higher than comparable cleaning-focused frameworks.
[Video-side synthesis/construction operators]
  · video_ffmpeg_wrapped_mapper — wraps FFmpeg video filters, able to apply arbitrary video transformations (cropping, color grading, noise addition, speed change, transitions, etc.), a general-purpose tool for constructing controlled-perturbation samples.
  · audio_ffmpeg_wrapped_mapper — the audio-side counterpart.
  · audio_add_gaussian_noise_mapper — Gaussian-noise augmentation for audio.
  · video_resize_resolution_mapper / video_resize_aspect_ratio_mapper — spec rewriting, usable to construct multi-resolution training pairs.
  · video_remove_watermark_mapper — watermark removal, can be paired with the original watermarked sample to form a "watermark-removal" editing training pair.
  · video_object_segmenting_mapper — produces object masks, a prerequisite for constructing object-level editing/replacement training pairs.
[Synthesis recipes in the DJ-Cookbook] Maintains a "video data synthesis" YAML recipe, a contrastive-learning data-synthesis recipe (corresponding to the CVPR 2025 work ImgDiff, arXiv:2408.04594, which improves VLLMs via contrastive data synthesis), a persona-oriented dialogue-synthesis recipe, and a curriculum-learning recipe based on data difficulty.
[Related research] The same team's ImgDiff (CVPR 2025) and MindGYM (NeurIPS 2025, question synthesis) are both methodological works on "using large models to synthesize training data," whose capabilities have flowed back into DJ operators.
[Uncertain] DJ does not provide an InstructAV2AV-style capability for constructing audio-video editing instruction pairs, nor a ready-made recipe specifically for video generation's "constructing positive/negative pairs via controlled perturbation" (such as constructing motion-intensity contrast pairs or audio-visual misalignment negative samples). The specific content and actual output scale of the video-synthesis recipe are not disclosed.

### Degree of Human Involvement (Manual Annotation, Manual Quality Inspection, Model Pre-Screening + Manual Review) ⚠️

Data-Juicer's design orientation is to "minimize human involvement as much as possible, replacing human judgment with model feedback," with the human role confined to two areas.
[Human-involvement point one: final determination of recipe design and threshold decisions] DJ presets no default thresholds, with all operator hyperparameters externalized to YAML. But the Sandbox's core contribution is precisely transforming this step from "manually setting thresholds by experience" into "automatic ranking via small-scale reference-model training + benchmark evaluation feedback" — humans only need to specify the candidate operator set and the evaluation benchmark; which segment of the distribution to retain is determined by experimental data.  This is the only work in this survey that systematizes and automates the threshold-determination process.
[Human-involvement point two: interactive operation and visual review] The interface layer provides multi-tier human-machine interaction channels: a low-level Python API (for engineers writing code), RESTful endpoints (for service-based invocation), a visual editor (the Alibaba Cloud PAI Designer component, a drag-and-drop data-processing-flow configurator), and conversational natural-language instruction via AgentScope agents (describing a data-processing need in a single sentence). The "Q&A Copilot" introduced in v1.4.6 further lowers the barrier to use. In addition, DJ provides a data analysis report and a Tracer (with Ray-mode support from v1.4.6) that lets a human see exactly which samples each operator actually removed — supporting infrastructure for manual sample-based quality inspection, though the quality inspection itself is carried out by the human.
[Large-scale group participation] The DJ 2.0 paper mentions supporting a data-filtering/synthesis competition involving 3,000+ teams, a form of crowdsourced recipe exploration, but not sample-by-sample manual annotation.
[No manual-annotation step] All of DJ's annotation capabilities are provided by model operators, with no manual-annotation workflow, annotation-platform integration, or annotation quality-spot-check mechanism included.
[Uncertain] It is not disclosed whether the official T2V case study included any manual-review step; DJ also does not provide annotation-consistency checking or manual-vs-model annotation comparison quality-assurance tools.

## Audio-Video Alignment

### Audio-Video Sync Detection Method (Lip Sync, Event Alignment) ⚠️

[Uncertain] Data-Juicer provides no dedicated audio-video sync-detection operator, which is its most critical capability gap in the AV-generation data-processing chain.
[Closest operator to sync detection] video_active_speaker_detect_mapper — "detects the active speaker in a video by analyzing visual face tracks and the audio signal." The underlying principle of this operator (such as TalkNet/Light-ASD-style active speaker detection models) does indeed rely on the temporal correlation between lip motion and audio, implying a form of lip-sync judgment; however, its output is a determination of "who is speaking" identity/time segments, rather than a continuous score of "how many frames of audio-visual offset" or "how confident is the sync" that could be used for threshold-based filtering. Users cannot directly use it to perform LSE-D/LSE-C-style sync-quality filtering.
[Missing capabilities]
  · No SyncNet operator — cannot output an audio-visual offset value and confidence score, and thus cannot reproduce SkyReels-V4's "|offset|≤3 and conf>1.5" style filtering.
  · No Synchformer operator — cannot perform temporal-alignment scoring on general audio-video events (non-speech), and thus cannot reproduce Foley-Omni's sync score ≥0.2 style filtering.
  · No AV-HuBERT, Perceiver, or similar audio-video alignment representation operators.
  · No ImageBind operator — cannot perform audio-video cross-modal semantic-consistency scoring.
[Visual-text alignment does exist] video_frames_text_similarity_filter performs "video frame-text" cross-modal consistency filtering based on CLIP, DJ's only mature, real-world-validated cross-modal alignment operator (the core criterion of the T2V case study, threshold 0.306337). But it operates on the visual-text axis, not the visual-auditory axis.
[Conclusion] To use Data-Juicer to build training data for joint audio-video generation, the sync-detection step must be extended by the user with custom operators. Fortunately, DJ's operator-registration mechanism (inheriting the Filter base class + YAML declaration + writing to stats fields) makes the engineering cost of wrapping SyncNet/Synchformer as custom operators relatively low, and can directly reuse its Ray distributed execution, adaptive GPU allocation, batch-processing optimization, and other underlying infrastructure — exactly where the value of DJ as an "operator library" rather than a "fixed pipeline" lies.

### Specific Sync Detection Metrics and Thresholds (SyncNet/Synchformer/LSE/In-House, Threshold Values, e.g., MOVA's LSE-D≤9.5 and LSE-C≥4.5, SkyReels-V4's SyncNet |offset|≤3∧conf>1.5) ⚠️

[Uncertain] Since it provides no sync-detection operator, Data-Juicer also has no recommended values for any sync metric or threshold.
[The only cross-modal threshold DJ publishes — for reference] The minimum threshold for video_frames_text_similarity_filter in the official optimal T2V data pool is 0.306337 (CLIP visual-text cosine similarity). How this value was arrived at is worth noting: it is not a manually chosen round number (like the commonly used 0.3), but rather a boundary value that naturally fell out of a quantile split of the data pool during the Probe-Analyze-Refine process — i.e., the specific value corresponding to that quantile point when "retaining the highest-similarity portion of samples" was applied. This practice of "deriving the threshold by working backward from a target retention proportion" contrasts with the "manually set threshold by experience" commonly seen in model teams.
[Threshold status for other operators] The thresholds of all DJ operators are externalized YAML parameters; the official config_all.yaml provides placeholder default values rather than recommended values. The documentation explicitly states that it does not provide generally recommended thresholds across scenarios; its position is that thresholds should be determined through the Sandbox's search on the specific dataset and specific downstream model.
[Methodological value] For this survey, what DJ provides is not "threshold values" but "the method for determining thresholds": split the data by some statistic into thirds → train a reference model on each → evaluate using a unified benchmark → select the segment with the greatest gain. This process could in theory be directly applied to sync metrics (e.g., splitting SyncNet confidence into thirds for probing), it's just that DJ officially has not yet run such an experiment on the audio-video sync dimension.

### Separate Handling of Temporal Sync vs. Semantic Sync (Temporal Alignment and Content-Semantic Matching as Two Independent Filtering Conditions) ⚠️

[Uncertain] Data-Juicer does not address the separate handling of temporal sync versus semantic sync, since it lacks audio-video sync-detection capability altogether (see av_sync_detection for details).
[An analogous design philosophy does exist] DJ's overall architectural stance is precisely "one independent operator per dimension, one independent threshold, no weighted merging" — all Filter operators make an independent single-metric determination and then stack them in series, never providing a multi-metric composite score. This is methodologically homologous to Foley-Omni's approach of splitting ImageBind (semantic) and Synchformer (temporal) into two independent thresholds: both hold that errors of different natures should be gate-kept independently by different criteria, avoiding a high score on one dimension masking a low score on another. Thus, were audio-video sync capability to be extended within DJ, it would naturally take the form of "separate semantic operator + temporal operator," with no architectural obstacle.
[An existing instance of this separation] On the visual-text axis, DJ does indeed split "semantic matching" (video_frames_text_similarity_filter, CLIP similarity) and "content attributes" (video_tagging_from_frames_filter, tag matching) into two independent operators rather than merging them into one composite alignment score — this can be viewed as the same design principle manifesting on a different modality axis.

### Audio Quality Filtering (SNR, Silence Detection and Silence-Ratio Thresholds, Removal of Tracks Without Audio, Off-Screen Sound Source Removal, Background Music Separation) ⚠️

Audio quality filtering is the most substantive part of DJ's audio-side capability, but its coverage is noticeably narrower than that of the video side.
[Existing operators]
  · audio_nmf_snr_filter — "retains samples whose audio signal-to-noise ratio (SNR) falls within a specified interval." Uses an NMF (non-negative matrix factorization) method to estimate SNR, decomposing the audio into a "signal component" and a "noise component" and computing their ratio. Compared to simple energy statistics, NMF estimation is more robust for scenarios containing structured noise (wind noise, hum, background noise). This is DJ's only core audio-quality operator.
  · audio_duration_filter — duration-interval filtering. Can indirectly remove samples with no audio track (zero duration) and audio clips that are too short.
  · audio_size_filter — filters by audio-file size, allowing rough identification of anomalously low-bitrate (too-small file size) or anomalously formatted samples.
  · audio_ffmpeg_wrapped_mapper — wraps FFmpeg audio filters, able to apply arbitrary repair operations such as noise reduction, loudness normalization, resampling, and channel processing. This is a "repair" path rather than a "filter" path, consistent with the dual-path design on the video side.
[Missing key capabilities] [Uncertain]
  · No silence-detection operator — cannot set a threshold such as "silence proportion must not exceed X%," nor locate and remove long silent segments. Can only indirectly exclude fully-silent samples via audio_duration_filter.
  · No audio-track-presence-detection operator — "whether the video has an audio track" must be determined by the user during data preparation.
  · No learned audio-quality-assessment operator — no integration of no-reference perceptual quality models such as Meta AudioBox Aesthetics, NISQA, or DNSMOS, which is a gap relative to the 2025-2026 trend of audio cleaning shifting from signal metrics toward perceptual models (compare Foley-Omni's use of AudioBox Aesthetics ≥0.6).
  · No source-separation operator — cannot perform background-music separation or vocal extraction, and thus cannot achieve "strip BGM while retaining vocals" or "quality-check by audio-track category."
  · No off-screen-voice/narration recognition — video_active_speaker_detect_mapper can determine whether anyone visible on screen is speaking, which could theoretically help identify cases of "speech present but the speaker is not on screen," but DJ has not packaged this usage into a ready-made operator or recipe.
[Overall assessment] DJ's audio-quality filtering remains at the traditional signal-processing level (SNR, duration, file size), sufficient to support ASR/TTS corpus cleaning, but insufficiently covers the fine-grained audio quality inspection needed for joint audio-video generation (silence ratio, perceptual quality, audio-track classification, in-frame-vs-off-frame source determination).

### Classification and Separate Handling Strategy for Speech/Sound Effects/Music ⚠️

[Uncertain] Data-Juicer does not provide an explicit classification taxonomy or separate handling strategy for speech/sound-effects/music, which is the core shortfall in its audio capabilities.
[Indirect means usable for approximate classification]
  · video_tagging_from_audio_mapper — produces audio-event tags based on the Audio Spectrogram Transformer (AST, trained on AudioSet). The AudioSet ontology is hierarchical with 527 classes, and its top-level categories happen to include Human sounds (including Speech), Music, Sounds of things, Natural sounds, Environment and background, etc. — in theory this top-level taxonomy could be used to roughly divide audio into four categories: speech/music/sound-effects/ambient sound. But DJ does not package this mapping as an explicit category field, nor does it provide an operator for category-based mixture sampling — the user would need to parse the tags and write the mapping logic independently.
  · video_audio_ASR_mapper — if it succeeds in transcribing meaningful text, this can serve as strong evidence of "speech present."
  · video_audio_speech_emotion_mapper / video_audio_detect_age_gender_mapper — these two operators are only meaningful when a human voice is present, so their applicability itself constitutes an indirect indicator of speech presence.
[Key gaps]
  · No audio-track source-separation operator (Demucs, Bandit, HTDemucs, etc. are all not integrated), so it is impossible to split a mixed audio track into speech/effects/music stems for separate processing and validation — this directly means it cannot reproduce Foley-Omni-style "field-level energy gating" correction mechanisms.
  · No mixture-control or stratified-sampling operator by audio category.
  · No music-detection/BGM-identification dedicated operator.
[Significance for this survey] Data-Juicer's current operator system has evolved along the main line of "text → image → video → embodied-AI geometric annotation," with audio consistently playing a supporting role (of the 229 operators, only 3 Filters + 2 Mappers are purely audio-focused). If it is to serve as the foundation for a joint audio-video generation data pipeline, roughly a dozen operators' worth of audio-side capability would need to be added (source separation, silence detection, perceptual quality assessment, audio-track classification, sync detection, audio-track-presence detection, loudness statistics, etc.) — a substantial but architecturally entirely feasible amount of work.

## Training Integration

### Multi-Stage Training Curriculum and Data Curriculum Scheduling (Basis for Stage Division: Resolution/Duration/Quality Score/Modality; Low-Res→High-Res, Image→Video, Short→Long) ⚠️

Data-Juicer does not train models, but it does provide two kinds of support for "data curriculum scheduling," and its T2V case study itself contains an implicit two-stage curriculum.
[Curriculum-learning recipe in the DJ-Cookbook] Officially maintains a "curriculum learning based on data difficulty" YAML recipe: operators first assign a difficulty score to samples (any statistic — aesthetic score, motion score, CLIP similarity, text complexity, etc. — can serve as a difficulty proxy), then the dataset is output in difficulty-stratified layers to be fed in stages. This is a ready-made template that productizes curriculum scheduling.
[The Sandbox's pyramid data pools = a natural curriculum structure] The 2^n−1 cross-combined data pools constructed during Refine form a pyramid: the top layer simultaneously satisfies all operator conditions strictly (highest quality, fewest samples), with conditions progressively relaxed further down (quality decreasing, sample count increasing). This structure maps directly onto a training curriculum: the large-sample lower layers can be used early in training to build a foundation, while the small, high-quality top layer can be used later for annealing/fine-tuning. In its comparison of scale-up routes, the paper actually tested the route of "progressively descending the pyramid to include suboptimal data + deduplication," equivalent to a reverse curriculum.
[The T2V case study's two-stage practice] The optimal data pool (147k) was first used for distillation training, then expanded to the 228k data pool (corresponding to 640k training samples, including about 2.8x repetition) for the final model — constituting a "small and precise → scale up" two-stage flow.
[Engineering-level stage support] v1.5.0 introduced operator-level environment management (OP-level environment management), allowing different stages to use different combinations of operators with different dependencies; the partitioned Ray executor and repartition pipeline (v1.5.3) facilitate producing data shards of different specs by stage.
[Uncertain] DJ does not give an officially recommended multi-stage curriculum scheme for video generation (e.g., the specific division basis and switching timing for "low-res→high-res," "image→video," "short→long"), nor does it report any curriculum experiments along the resolution or duration dimensions in any public case study. This falls under the responsibility of the training-framework side; DJ only provides the raw material for data stratification.

### Changes in Data Mixture Across Training Stages (Pretraining/Annealing/SFT High-Quality Subset) ⚠️

[DJ's capability positioning] Provides tools for measuring, splitting, and searching data mixture, but does not itself provide a mixture plan.
[Mixture-search mechanism] The Sandbox's Refine stage directly answers "how should data of different quality tiers be combined": cross-combining the top-n ranked operators into 2^n−1 data pools, with each pool corresponding to a mixture scheme, each trained on a small-scale reference model and evaluated, from which the optimal combination is selected. This is a concrete implementation that turns data mixture from a manual-design problem into a searchable one.
[Two contrasted scale-up routes — the most valuable mixture conclusion in this case] The paper explicitly contrasts two routes for scaling from a small pool to a large pool:
  Route A: repeating high-quality data (duplicating the top-layer small pool multiple times).
  Route B: progressively descending the pyramid to include suboptimal data and deduplicating (expanding sample diversity).
  The experimental conclusion favors Route A as superior for text-to-video — "the performance obtained by repeating the highest-quality data 6x is higher than that from using 8x the compute with suboptimal data." This means that in the T2V setting, the marginal value of data quality exceeds that of data diversity and compute investment.
[Related mixture research] The same team's BiMix (arXiv:2405.14908) proposes a Bivariate Data Mixing Law for language-model pretraining, modeling the effect of "data mixture × data volume" on loss as an extrapolatable law; Diversity as Reward (arXiv:2502.04380, NeurIPS 2025) studies diversity-driven fine-tuning data selection on domain-uncertain data. These two works represent the DJ team's exploration of mixture theory, but both target language/multimodal understanding rather than covering video generation.
[Uncertain] DJ does not provide an official mixture template for the three stages of pretraining/annealing/SFT; the T2V case study does not report the specific mixing method between the 147k and 228k pools, nor the changes in the proportion of each source dataset (InternVid/Panda-70M/MSR-VTT) in the final pool.

### Post-Training Data (Size and Selection Criteria of the Curated SFT Set, Number and Annotation Method of Preference Pairs, Reward Model Training Data) ⚠️

[System capability] One of the clear expansion directions of Data-Juicer 2.0 relative to 1.0 is "supporting foundation-model post-training tasks" (foundation model post-training), listed in the paper's abstract alongside data analysis, synthesis, and annotation as one of four supported task types. Roughly 50 operators serve data synthesis and augmentation, many geared toward post-training scenarios.
[Specific support]
  · Post-training data synthesis: the DJ-Cookbook contains persona-oriented dialogue-processing recipes and contrastive-learning data-synthesis recipes; the same team's MindGYM (NeurIPS 2025, arXiv:2503.09499) studies question synthesis oriented toward fine-tuning reasoning ability, whose methods have flowed back into operators.
  · Data selection: Diversity as Reward (NeurIPS 2025, arXiv:2502.04380) studies diversity-driven fine-tuning data selection on domain-uncertain data, i.e., a methodology for constructing curated SFT sets.
  · Reinforcement fine-tuning support: the DJ 2.0 paper mentions the system "facilitates studies on data scaling laws and reinforced fine-tuning," i.e., it has already been used to prepare data for reinforcement fine-tuning (RFT) related research.
  · Agent data quality: v1.5.2 added an agent data-quality toolkit.
[Situation on the video-generation post-training side] [Uncertain] DJ has not officially published any preference-pair construction operator, reward-model training-data construction recipe, or SFT curated-set selection criteria oriented toward video generation. The T2V case study follows a supervised distillation training route (based on T2V-Turbo), with no RLHF/DPO stage involved. Building preference data for video generation with DJ would require independently extending operators for "same-prompt multi-sample generation + scoring/ranking + pairing."

### Data Processing Infrastructure and Throughput (NeMo Curator/Data-Juicer/In-House, GPU Speedup Ratio, Processing Scale, Cost)

This is the core contribution dimension of Data-Juicer 2.0, and also the object with the most thoroughly disclosed data-infrastructure information in this survey — most model teams stay tight-lipped on this, while the entirety of DJ's paper is dedicated to answering exactly this question.
[Scale capability] Processes TB-scale data, scheduling 10k+ CPU cores; experiments span dataset scales from 560,000 to 70 billion samples; one production deployment has run stably for over 5 months, cumulatively processing more than TB-scale data.
[Multi-backend adaptive execution] The runtime layer provides hardware-agnostic adapters, automatically or manually selecting backends according to scale:
  · Small scale (about 560,000 samples): standalone mode offers the best cost-effectiveness.
  · Medium scale (5.6 million-56 million samples): Ray mode is recommended, with a 4-node speedup of 148%-226%.
  · Large scale (70 billion samples): for text-only data, Alibaba Cloud MaxCompute is recommended (at 10k+ core scale, taking about 1/4 the time of Ray); for multimodal data, Ray-DLC (PAI's containerized deep-compute cluster) is recommended.
  · Distributed measurements: processing a 500x-scale dataset on 3,200 Ray-DLC cores took 1,780 seconds, and a 2,500x-scale dataset took 7,083 seconds (near-linear scaling).
[Key optimization techniques and quantified gains]
  · Adaptive data splitting: 2-3x speedup on large datasets.
  · Workload-aware OP reordering: under the constraint of operator commutativity, moving lightweight operators earlier, reducing processing time by up to 70.22%.
  · Adaptive GPU resource allocation: model-type operators automatically use GPU and quantization (integrating vLLM), saving up to 99% of processing time.
  · Batched processing: reduces time by up to 84%.
  · Operator fusion (FusedOP): batch-level operator fusion using a Strategy/Decorator pattern, substantially reducing network I/O — peak throughput drops from about 160MB/s to 60MB/s (i.e., reducing roughly two-thirds of wasted data movement).
  · I/O-intensive operators use a hierarchical parallelism of batch processing, multiprocessing, and multithreading.
  · v1.5.4 added batch-local stage fusion for further speedup.
[Deduplication performance benchmarked against NeMo Curator] MinHash deduplication uses a load-balanced union-find implementation, achieving a 3.3x speedup over the native Ray implementation; 8 Ray nodes process 5TB in 2.8 hours. The paper uses NVIDIA NeMo Curator (1.1TB in 1.8 hours on 64 A100 GPUs) as a comparison point, highlighting the cost advantage of DJ's pure-CPU approach. Scalability: a 5x increase in data volume leads to a 4.02x-5.62x increase in time; doubling core count reduces time to 58.9%-67.1% of the original.
[Empirical insights on cost and storage]
  · Alibaba Cloud's dedicated cluster-network optimizations save 24.8% in cost compared to standard ECS.
  · Storage choice has a significant effect: using NAS/OSS instead of CPFS increases cost by 20-30%.
  · A 3x increase in network bandwidth yields a 2.7x processing speedup — the paper explicitly notes that large-scale multimodal data processing is I/O-bound rather than compute-bound, a highly valuable engineering conclusion for this survey.
[Ecosystem and adoption] Natively compatible with HuggingFace datasets and the Ray compute engine; integrates with ModelScope, NVIDIA NeMo, Alibaba Cloud PAI (including the visual Designer component), PAI-DLC, and 20+ other frameworks and platforms. The adopter list covers enterprises including Alibaba Group, Ant Group, BYD, ByteDance, DTSTACK, JD.com, NVIDIA, OPPO, Xiaohongshu, Xiaomi, and Ximalaya, as well as academic institutions including the Chinese Academy of Sciences, Nanjing University, Peking University, Renmin University of China, Tsinghua University, UCAS, and Zhejiang University. The paper specifically mentions that it supports Alibaba Tongyi's enterprise-scale foundation-model training, particularly TB-token-scale pretraining and the high-overhead video/image processing needed for the "spatial intelligence" direction.

## Performance Comparison

### Quantitative Impact of Data Strategy Ablations (Distinguishing: Filtering Strictness Ablation / Caption Density-Style Ablation / Data Mixture Ablation, and Corresponding Evaluation Metrics) ⚠️

Data-Juicer Sandbox's entire methodology is organized around data ablation — it elevates data ablation from "a validation experiment in a paper's final section" to "a search mechanism running throughout the entire data-recipe design process." This is its most distinctive contribution to this survey.
[Ablation type one: filtering-strictness ablation (DJ's main battleground)] During Probe, for each candidate operator, the dataset is evenly split by its statistic into three equal-sized pools — low/middle/high (P_i,low / P_i,middle / P_i,high) — plus a randomly sampled control pool, with a reference model trained on each pool under the same budget and evaluated with VBench (16 metrics). This is equivalent to running a complete "which segment of the distribution is most beneficial to retain" ablation for each filtering dimension, with strict variable control (equal pool sizes, identical training budgets). The candidate operators on the video side were video_aesthetics_filter, video_nsfw_filter, and video_frames_text_similarity_filter; NSFW and frame-text similarity ultimately won out and entered the final recipe, while the aesthetic score was not selected.
[Ablation type two: data mixture/combination ablation] During Refine, the winning operators are cross-combined into 2^n−1 data pools (pyramid structure), each validated one by one — i.e., an exhaustive ablation over multi-operator mixtures.
[Ablation type three: data-volume vs. quality vs. compute three-way ablation — the most valuable quantitative conclusion] The paper compares three scale-up routes and gives a clear ranking: performance from repeating the highest-quality data 6x exceeds that from using 8x the compute with suboptimal data. This is a joint ablation spanning three variables — data quality, data repetition count, and compute budget — with a conclusion pointing directly at "in text-to-video scenarios, improving data quality has a higher cost-effectiveness than increasing compute."
[Quantified final results]
  · T2V-Turbo, trained on the DJ-refined data pool, topped VBench: Board Average 82.53%, Uniform Average 81.26%, surpassing second-place Gen-3's 82.32% and third-place VEnhancer's 81.97%.
  · Improvement over the T2V-Turbo baseline: Board Average +1.53%, Uniform Average +2.59%.
  · The HuggingFace dataset card separately records the 147k data pool as bringing "a 1.09% improvement relative to T2V-Turbo."
  · The paper emphasizes that this result was achieved at "at least 22x lower compute cost" — the most direct evidence for "data optimization substituting for compute stacking."
  · The small-scale probing experiments on EasyAnimate likewise obtained a considerable improvement via the data recipe (specific values not given in the accessible content).
[Cross-model transferability verification] The optimal recipe searched for at small scale (EasyAnimate) remained effective when directly transferred to the architecturally different T2V-Turbo (based on VideoCrafter-2.0), indicating that the data recipe's benefits are transferable across architectures — a precondition for the Sandbox methodology to hold, and one of its most important experimental findings.
[Total experiments] The paper reports completing 100+ groups of experiments, covering three task types: image-text pretraining (CLIP), image-to-text generation (LLaVA-type), and text-to-video (DiT-type).
[Uncertain] Not disclosed: the ablation along the caption density/style dimension (since no recaptioning was performed in the case study); the detailed probing rank and score for each of the three candidate operators; whether other video operators such as the motion score participated in probing at all.

### Evidence for Quality vs. Quantity (Cases Where Small, High-Quality Data Outperforms Large, Mixed Data)

The Data-Juicer Sandbox's T2V case study is the most complete and most reproducible example in this survey of "quality over quantity" evidence, because its data, recipe, model, and evaluation are all open-sourced.
[Evidence one: an extreme 12.09% retention rate still achieves SOTA] Out of about 1.217 million candidates, only 147,176 (12.09%) were retained, and the model trained on this 1/8 of the data topped VBench. If quantity were paramount, using the full 1.217 million directly should have been better; the experimental conclusion is the opposite.
[Evidence two: data repetition beats compute scaling — the sharpest finding] The paper's explicit conclusion: the performance from repeating the highest-quality data 6x exceeds that from using 8x the compute with suboptimal data. This is methodologically potent because it pushes the "quality vs. quantity" comparison forward into "quality vs. compute" — even giving suboptimal data 8x the compute cannot catch up to a simple repetition of high-quality data. This directly challenges the common assumption that "data diversity is necessarily better than data repetition," suggesting that in a task as sensitive to video-text alignment as text-to-video, the negative impact of one low-quality sample outweighs the diminishing marginal benefit of one repeated sample.
[Evidence three: winning under a 22x compute gap] The model that topped VBench did so at "at least 22x lower compute cost" than competitors such as Gen-3 and VEnhancer. This is the manifestation of "small, high-quality data outperforming large, mixed data" in a cross-team horizontal comparison.
[Evidence four: a minimal recipe outperforms a complex funnel] The final effective recipe contains only two operators (NSFW filtering + CLIP frame-text similarity filtering), far simpler than the dozen-plus-stage funnels used by most model teams, and it was rigorously ablation-validated. This suggests that "number of filters" and "data quality" are not positively correlated — strict gatekeeping on a small number of truly relevant dimensions beats loose filtering across a large number of weakly relevant dimensions.
[Evidence five: cross-architecture transferability] The recipe searched for at small scale (EasyAnimate) remained effective when transferred to T2V-Turbo, indicating that "what constitutes high-quality data" has some degree of model-independence, not an artifact overfit to a specific model.
[Methodological significance] What DJ provides is not merely a conclusion, but a reusable mechanism (Probe-Analyze-Refine) for reaching that conclusion. For video-generation teams, the most practical takeaway is: rather than designing a long funnel by experience, first use a small model to run a tertile-based probe on each candidate filtering dimension, and retain only the few dimensions that genuinely bring evaluation gains.

### Alignment Between Training Data Domain Distribution and Benchmark Taxonomy (e.g., VABench's Seven Major Categories) ⚠️

[DJ's approach to benchmark alignment: embedding the benchmark directly into the data-recipe search loop] This is the most fundamental difference between the Sandbox and other work — most teams first fix a data distribution, then run the finished model against a benchmark, whereas DJ "uses benchmark scores to determine the data distribution in reverse." On the video side, VBench's 16 metrics are used as the ranking basis during the Analyze stage, meaning the data recipe is directly shaped by VBench's taxonomy.
[Benefits and costs of this strong alignment]
  · Benefit: the data distribution and the evaluation criteria are naturally consistent, maximizing evaluation gains — this is the direct reason it topped VBench.
  · Cost: there is a risk of overfitting to the benchmark. If VBench's 16 metrics fail to cover certain real needs (such as long-term temporal consistency, multi-shot narrative, or audio-visual coordination), the data recipe searched via its feedback will systematically neglect those dimensions. The paper does not discuss this risk, a point worth noting methodologically. DJ's mitigating evidence is its cross-architecture transferability (the recipe is effective on both EasyAnimate and T2V-Turbo), but this only demonstrates cross-model generalization, not cross-benchmark generalization.
[Relationship to other benchmarks in this survey]
  · The Sandbox's video evaluation uses only VBench, without connecting to audio-video benchmarks such as VABench (the seven-major-category taxonomy), AVBench, or AV-SyncBench — consistent with its purely visual case-study positioning.
  · If the Sandbox methodology were transferred to joint audio-video generation, it would only require swapping the evaluator in the Analyze stage for VABench/AV-SyncBench, etc., requiring no architectural change; however, as noted earlier, the audio-video-side candidate operators (sync detection, audio-track classification, audio perceptual quality) would need to be added first, otherwise there would be no candidates to rank during the probing stage.
[Benchmark-building by the same team] HumanVBench (CVPR 2026, arXiv:2412.17574), built by this team on Data-Juicer, covers four major dimensions — emotion perception, person identification, behavior analysis, and speech-visual alignment — using 20+ operators to construct both a person-centric video annotation pipeline and a distractor-containing QA synthesis pipeline. Its "speech-visual alignment" category is the only part of the DJ ecosystem that touches on audio-video alignment evaluation, and can be seen as an interface toward DJ's future expansion in the AV direction. The team also has DetailMaster (ICML 2026, arXiv:2505.16915), which focuses on long-prompt processing for text-to-image.
[Measurability of data distribution] A hidden advantage of DJ is that, since every operator writes its statistic to the sample's stats field, the distribution of any dataset along any dimension can be quantified and plotted as a histogram, making the analysis of "training data distribution vs. benchmark taxonomy" alignment technically executable — it's just that DJ has not officially published such an analysis report for video generation.
[Uncertain] Not disclosed: a distribution profile of the final data pool across VBench's 16 metric-corresponding dimensions; any cross-alignment analysis against other benchmark taxonomies such as VABench.

## Uncertain Fields

The research findings for the following fields are partially uncertain (sources marked with ⚠️):

- release_date
- duration_distribution
- resolution_aspect_distribution
- domain_distribution
- audio_category_distribution
- narrative_structure_distribution
- language_accent_distribution
- provenance_licensing
- funnel_retention_rate
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
- human_in_loop
- av_sync_detection
- sync_metric_and_threshold
- temporal_vs_semantic_sync
- audio_quality_filtering
- audio_type_handling
- multi_stage_curriculum
- stage_data_mixture
- post_training_data
- data_ablation
- benchmark_taxonomy_alignment
