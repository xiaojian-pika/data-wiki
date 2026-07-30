# NVIDIA NeMo Curator (26.02 / 26.04 versions) + Cosmos-Xenna (the underlying distributed execution engine on the video side), together with the co-derived productized implementation Cosmos-Curate (the training data generation system for the Cosmos world foundation models)

> Topic: Data processing for video generation models (including joint audio-video generation): data cleaning pipelines, data distribution, and annotation methods

[← Back to home](../index.md)

**Table of Contents**: [Basic Information](#basic-information) · [Data Scale and Distribution](#data-scale-and-distribution) · [Cleaning Pipeline](#cleaning-pipeline) · [Annotation Methods](#annotation-methods) · [Audio-Video Alignment](#audio-video-alignment) · [Training Integration](#training-integration) · [Comparative Results](#comparative-results)

## Basic Information

### Name

NVIDIA NeMo Curator (26.02 / 26.04 versions) + Cosmos-Xenna (the underlying distributed execution engine on the video side), together with the co-derived productized implementation Cosmos-Curate (the training data generation system for the Cosmos world foundation models)

### Publishing organization/company

NVIDIA. The three repositories belong to different GitHub organizations: NeMo Curator lives under NVIDIA-NeMo/Curator, while Cosmos-Xenna and Cosmos-Curate live under the nvidia-cosmos organization (formerly NVIDIA/cosmos-curator). NeMo Curator is maintained by the NeMo Framework team; Cosmos-Xenna was developed by the Cosmos (Physical AI / world foundation model) team and later spun out as an independent open-source project.

### Release date (technical report/paper/open-source date)

【NeMo Curator version timeline (dual-tracked CalVer and SemVer; dates are PyPI upload times)】0.6.0 (2025-01-07, Dask architecture, text-only era) → 0.7.0 (2025-03-12) → 0.8.0 (2025-05-09) → 0.9.0 (2025-07-28) → 1.0.0 = version 25.09 (2025-10-01, milestone: backend fully switched from Dask to Ray, first introduction of video and audio modalities, forming a unified text/image/video/audio four-modality architecture) → 1.1.0 = version 26.02 (2026-02-23) → 1.2.0 = version 26.04 (2026-05-14) → 1.3.0 = version 26.07 (2026-07-27, released two days before this survey's cutoff date).
【Cosmos-Xenna】Split out from Cosmos-Curator and open-sourced independently (Apache 2.0) alongside the Cosmos platform in 2025; NeMo Curator 26.04 upgraded to Cosmos-Xenna 0.2.0. As of July 2026, the cosmos-xenna repository is marked "no longer under active development," with the official guidance pointing users to migrate to the Cosmos 3 / NeMo Curator ecosystem.
【Primary methodological literature】Cosmos World Foundation Model Platform for Physical AI (arXiv:2501.03575, 2025-01-07, Section 3 fully discloses the seven-stage video data curation pipeline); Training Video Foundation Models with NVIDIA NeMo (arXiv:2503.12964, 2025-03, specifically describes the dual clipping/sharding pipeline and GPU acceleration); the NVIDIA official blog first published the "89x acceleration" figure in 2025-01.

### Type (model/dataset/toolchain/benchmark)

Toolchain / data infrastructure framework (data curation toolkit). It is not a generative model, not a dataset, and not an evaluation benchmark. It is positioned as a "reproducible, GPU-accelerated framework for building large-scale data processing pipelines," covering the full load–filter–deduplicate–annotate–transform–write-out workflow across text, image, video, and audio modalities. Within the object system of this survey, it belongs to "upstream infrastructure used by other video generation/world model teams"; NVIDIA's own framing describes it as an open-source reference implementation for industrial-grade video data processing.

### Degree of openness (whether weights/code/data/pipeline are each open-sourced)

Among peers, its degree of openness is the highest, falling into the category of "pipeline fully open-sourced, data entirely absent."
【Code】All NeMo Curator source code is under the Apache License 2.0, publicly available on GitHub (NVIDIA-NeMo/Curator), with the PyPI package named nemo-curator; official installation is recommended via Docker container (video/audio workflows require a pre-configured FFmpeg 8.0.1 + NVENC).
【Underlying engine】Cosmos-Xenna is likewise independently open-sourced under Apache 2.0 and can be used standalone, decoupled from Curator.
【Productized implementation】Cosmos-Curate source code is Apache 2.0; the model weights it invokes are governed by the NVIDIA Open Model License, with custom commercial licenses available on request.
【Pipeline configuration】Starting from 26.02, YAML declarative definition of an entire curation pipeline is supported, further lowering the barrier to reproduction.
【Model weights】The framework itself does not train models; the discriminative/annotation models it invokes are mostly third-party open-source weights: Qwen2.5-VL / Qwen3-VL (captioning), Cosmos-Embed1 and InternVideo2 (video embeddings, with InternVideo2 removed in 26.02), a CLIP-based aesthetic model (aesthetic scoring), TransNetV2 (shot segmentation), the NVIDIA NeMo ASR model family (audio transcription), and the Nemotron Nano 12B V2 VLM and Nemotron 3 Nano Omni (new captioning backends added in 26.04/26.07, with BF16/FP8/NVFP4 precision variants).
【Not open-sourced】The training data itself (the framework ships with no dataset whatsoever); the actual list of data sources and the full-scale statistics NVIDIA uses internally for Cosmos are not disclosed.

### Whether joint audio-video generation is supported, and the implementation approach (native joint / cascaded / MoE fusion)

Not applicable — this entry is a data processing toolchain rather than a generative model, and it does not produce any audio-video content itself.
Regarding "joint audio-video processing capability," a current limitation must be explicitly noted: although NeMo Curator claims four modalities (text/image/video/audio), the video pipeline and the audio pipeline are architecturally independent and never cross paths. The video side only processes the visual track (segmentation/transcoding/frame extraction/motion and aesthetic filtering/captioning/embedding/deduplication/sharding); the audio side is an independent workflow aimed at ASR speech data (load → NeMo ASR transcription → WER/CER quality assessment → hand-off to text curation → export). Nowhere in the official documentation does the audio module mention any linkage with video, nor is there any stage that extracts an audio track from video, performs audio-video alignment, or does joint annotation. As a result, it currently cannot directly support the data construction needs of "jointly generated audio-video" models — this is the framework's biggest gap relative to the data requirements of AV joint-generation models such as LTX-2 / Ovi / Sora 2.

### List of research information sources (URLs of papers/technical reports/official documentation/news, with each source labeled as: official primary / same-team corroboration / third-party reporting)

- 【Official primary】NeMo Curator main GitHub repository (Apache 2.0, four-modality capability, Ray/Xenna architecture, performance claims, installation method): https://github.com/NVIDIA-NeMo/Curator
- 【Official primary】NeMo Curator Releases page (highlights of each version, including 26.02/26.04/26.07): https://github.com/NVIDIA-NeMo/Curator/releases
- 【Official primary】PyPI nemo-curator release history (precise release dates: 1.0.0=2025-10-01, 1.1.0/26.02=2026-02-23, 1.2.0/26.04=2026-05-14, 1.3.0/26.07=2026-07-27): https://pypi.org/project/nemo-curator/
- 【Official primary】NeMo Curator video curation overview documentation (each stage and the models used: TransNetV2, Qwen-VL, Cosmos-Embed1, CLIP aesthetic, NVENC/NVDEC, semantic deduplication, WebDataset): https://docs.nvidia.com/nemo/curator/curate-video
- 【Official primary】NeMo Curator video filtering documentation (algorithms for motion filtering and aesthetic filtering, stage names, all default thresholds and parameters): https://docs.nvidia.com/nemo/curator/curate-video/process-data/filtering
- 【Official primary】NeMo Curator video architecture documentation (Ray Actor management, Ray Object Store, Cosmos-Xenna executor, autoscaling, and all default configuration items): https://docs.nvidia.com/nemo/curator/about/concepts/video/architecture.md
- 【Official primary】NeMo Curator audio curation documentation (ASR transcription, WER/CER filtering, speaker diarization; confirms no linkage with video): https://docs.nvidia.com/nemo/curator/curate-audio
- 【Official primary】NeMo Curator 26.07 Release Notes / migration checklist (audio enhancement stage, Nemotron-CLIMB, captioning backend matrix, breaking changes): https://docs.nvidia.com/nemo/curator/about/release-notes
- 【Official primary】NeMo Curator 26.02 video quick-start documentation: https://docs.nvidia.com/nemo/curator/26.02/get-started/video.html
- 【Official primary】NeMo Curator 26.02 transcoding/Clip Encoding documentation: https://docs.nvidia.com/nemo/curator/26.02/curate-video/process-data/transcoding.html
- 【Official primary】Cosmos-Xenna GitHub repository README (Ray distributed data pipeline library, three modes — streaming/batch/serving, autoscaling and bin-packing, backpressure, SPMD, P2P distribution, Apache 2.0, note that the repository is no longer under active development): https://github.com/nvidia-cosmos/cosmos-xenna
- 【Official primary】Cosmos-Curate GitHub repository README (the Cosmos training data generation system, built on top of Cosmos-Xenna, code under Apache 2.0 / models under the NVIDIA Open Model License): https://github.com/nvidia-cosmos/cosmos-curate
- 【Official primary · core methodology】Cosmos World Foundation Model Platform for Physical AI, NVIDIA, arXiv:2501.03575 (Section 3 fully discloses the seven-stage curation pipeline: TransNetV2 selection benchmarked at F1=0.967, NVDEC/NVENC 6.5x, ViT optical-flow motion classifier, DOVER bottom 15%, aesthetic threshold 3.5, InternVideo2+MLP for text overlay and type classification, VILA 13B FP8 TensorRT-LLM 10x, caption 559 characters / 97 words, k-means k=10000 deduplication removing 30%, nine-category mixture ratios, 20M hours → 100M clips): https://arxiv.org/abs/2501.03575
- 【Official primary】Training Video Foundation Models with NVIDIA NeMo, arXiv:2503.12964 (the dual clipping/sharding pipeline structure, 100PB+ scale, NVDEC/NVENC 3x, captioning as the bottleneck stage, Ray automatic worker balancing): https://arxiv.org/abs/2503.12964
- 【Official primary · reprint】Accelerate Custom Video Foundation Model Pipelines with New NVIDIA NeMo Framework Capabilities (NVIDIA official blog, original source of the "89x acceleration" figure: "1K GPUs, ISO power, relative to an unoptimized CPU pipeline"; 20M hours; 100PB+; heterogeneous L40S/H100/GB200 clusters): https://www.edge-ai-vision.com/2025/01/accelerate-custom-video-foundation-model-pipelines-with-new-nvidia-nemo-framework-capabilities/
- 【Official primary】Advancing Physical AI with NVIDIA Cosmos World Foundation Model Platform (NVIDIA technical blog, 20 million hours in 40 days on Hopper / 14 days on Blackwell / 3.4 years on CPU): https://developer.nvidia.com/blog/advancing-physical-ai-with-nvidia-cosmos-world-foundation-model-platform/
- 【Official primary】World Simulation with Video Foundation Models for Physical AI (Cosmos follow-up paper, arXiv:2511.00062): https://arxiv.org/pdf/2511.00062
- 【Official primary】NVIDIA Nemotron 3 Nano Omni technical blog (background on the all-modal model introduced into Curator in 26.07): https://developer.nvidia.com/blog/nvidia-nemotron-3-nano-omni-powers-multimodal-agent-reasoning-in-a-single-efficient-open-model/
- 【Official primary】NeMo Curator video splitting example code, tutorials/video/getting-started/video_split_clip_example.py: https://github.com/NVIDIA-NeMo/Curator/blob/main/tutorials/video/getting-started/video_split_clip_example.py
- 【Official primary】Cosmos Curator video pipeline reference documentation, docs/curator/reference/video-pipelines.md: https://github.com/NVIDIA/cosmos-curator/blob/main/docs/curator/reference/video-pipelines.md
- 【Third-party reporting】Architecting Data Pipelines for Multimodal Datasets at Scale (Anyscale blog, a Ray-centric perspective on multimodal data pipeline architecture): https://www.anyscale.com/blog/architecting-multimodal-data-pipelines-that-scale-with-ray

## Data Scale and Distribution

### Training data volume (video count/hours/token count, pretraining and SFT reported separately)

The framework itself contains no data; recorded here are its "validated processing scale" and "designed capacity":
【Designed capacity】The officially and repeatedly stated figure is that it "can efficiently clip, annotate, and filter 100 PB or more of videos."
【Measured scale (Cosmos WFM production run)】Input of roughly 20 million hours (20M hours) of raw video, at resolutions from 720p–4K; output of roughly 100 million (100M) clips of 2–60 seconds each; of these, roughly 10^8-scale clips were used for pretraining and roughly 10^7-scale for fine-tuning.
【Processing time】For 20 million hours of video: a Hopper (H100) GPU cluster takes about 40 days; Blackwell takes about 14 days; an unoptimized CPU pipeline under equivalent conditions would take about 3.4 years. Another official figure states "using 1,000 GPUs, an 89x speedup over an unoptimized CPU pipeline at ISO power, compressing processing time from years to days."
Note: these numbers describe NVIDIA's own Cosmos project run at production scale — they are not general-purpose figures for NeMo Curator users.

### Data source composition (proprietary/public datasets/web crawling/licensed acquisition/synthetic data) ⚠️

As a toolchain, it is "source-agnostic" and does not itself dictate data provenance. Supported data ingestion methods include: local filesystem, S3-compatible object storage (26.04 added a CommonCrawl S3 transfer option), custom manifests, Hugging Face datasets (the audio-side example is FLEURS), and reading/writing in the WebDataset format.
The data source composition of its upstream production use case (Cosmos WFM) is: proprietary/in-house video datasets + public internet video, and it explicitly includes synthetically rendered video (accounting for roughly 4% of the final mixture ratio). NVIDIA has not disclosed the exact proportions, acquisition channels, or licensing arrangements for each source. [Uncertain]

### Data compliance and provenance (proportion of licensed data, rights-cleared datasets, C2PA, etc.) ⚠️

At the framework level, NeMo Curator provides no data provenance or licensing management capability: the official documentation contains no C2PA content-credential embedding, no copyright fingerprint detection, no rights-cleared dataset management module, and no standardized provenance-metadata field specification. The framework only guarantees that original metadata is passed through during sharding.
Compliance responsibility is left entirely to the user: under the Apache 2.0 license, NVIDIA makes no warranty regarding the legality of any data a user processes; the compliance narrative on the Cosmos model side (Guardrails, trustworthy-AI claims) is a separate matter from the data curation pipeline. The Cosmos WFM paper likewise does not disclose the proportion of its 20-million-hour video corpus that is licensed, nor any argument for the legality of its sourcing. This is a clear gap relative to strong-compliance-narrative solutions such as Adobe Firefly / Lightricks (which license from Shutterstock, Getty, etc.). [Uncertain]

### Clip duration distribution and segmentation strategy ⚠️

【Segmentation strategy (framework capability)】Two configurable clip-extraction modes are provided: (1) fixed-stride segmentation — hard-cut by a fixed number of seconds (clip length and stride are configurable); (2) scene-change detection segmentation — shot-aware, based on TransNetV2. The two can be stacked — first split by shot, then further subdivide any overlong shot by fixed stride.
【Duration thresholds in production practice (Cosmos WFM)】Clips shorter than 2 seconds are discarded after splitting; clips longer than 60 seconds are further subdivided; the final clip duration distribution falls in the range of 2–60 seconds. The granularity for captioning is "one caption generated per 256 frames," meaning an overlong clip is split into multiple captioned segments.
【Distribution figures】The specific sample proportion within each duration bucket has not been published. The framework itself does not mandate any duration distribution — this is left to user configuration. [Uncertain]

### Resolution/aspect-ratio distribution and bucketing strategy ⚠️

【Framework capability】Resolution and aspect ratio are not used as filtering criteria but rather as organizing dimensions at the sharding stage: the final WebDataset is packed into buckets along three dimensions — resolution × aspect ratio × duration — with the goal of aligning with the bucketing needs of the downstream training curriculum, so that training can sample by bucket and avoid excessive token-count variance within a single batch. This is the design in this pipeline most worth emulating in terms of "coupling data infrastructure with the training curriculum."
【Transcoding normalization】All clips are uniformly transcoded to H.264 mp4, with a choice of encoder — h264_nvenc (GPU) / libx264 / libopenh264 (CPU) — and a choice of decoder — nvdec (GPU) / ffmpeg. The frame-extraction stage (ClipFrameExtractionStage) allows configuration of target fps and extraction strategy (e.g., a "sequence" strategy for use by aesthetic scoring).
【Production practice】The input footage for Cosmos WFM ranges from 720p to 4K. The specific sample proportion for each resolution/aspect-ratio bucket has not been published. [Uncertain]

### Category/domain distribution and mixture strategy (proportional control and concept balancing across people, actions, scenes, styles, etc.)

This is one of the most valuable fields in this entry — through NeMo Curator/Cosmos-Curate, Cosmos WFM implements explicit, classifier-based domain mixture control, a rare publicly documented industrial case of "quantifiable data mixture ratios."
【Implementation mechanism】A lightweight MLP classifier is trained on top of InternVideo2 video embeddings, tagging each clip with a category label according to a predefined video-type taxonomy, and then filtering and resampling accordingly.
【Excluded categories】Abstract patterns, video game footage, and animation — these three categories are judged unhelpful for learning real-world physical dynamics and are excluded wholesale.
【Resampling strategy】Upsampling of human motion and human-object interaction categories; downsampling of landscape categories — because landscape video, while high in visual quality, is sparse in dynamic and physical-interaction information and is naturally overrepresented.
【Final nine-category mixture ratio (published in the Cosmos WFM paper)】Nature dynamics 20%, hand motion & object manipulation 16%, spatial awareness & navigation 16%, driving 11%, human motion & activity 10%, first-person POV 8%, dynamic camera 8%, other 7%, synthetically rendered 4%.
【Note】This taxonomy serves the goals of Physical AI / world models (learning physical laws) and differs from the mixture goals of general text-to-video models (which emphasize characters, cinematic feel, and stylistic diversity); the category system needs to be redefined for such a transfer. The open-source version of NeMo Curator does not ship the weights of this taxonomy classifier — users must train it themselves.

### Audio category distribution and mixture (proportions and control strategy for speech/foley/music/ambient sound/silence) — a dimension unique to AV models

None. This is the most critical missing dimension in this toolchain for the audio-video generation data scenario.
The video pipeline does not touch the audio track at all — there is no classification stage for speech/foley/music/ambient sound/silence, no statistics on the proportion of any audio type or any mixture control strategy, and no built-in stage for extracting an audio track from video. Although an audio pipeline does exist, its data model is "an independent audio file + transcript" (aimed at ASR/TTS speech data), covering only the speech category, and it is not linked to video at all. The audio enhancement stage added in version 26.07 (tagging, SQUIM quality metrics, bandwidth estimation, punctuation preparation, optional secondary ASR scoring) is likewise entirely centered on speech quality and does not touch foley/music/ambient sound classification.
Consequently, if one wants to use NeMo Curator to build training data for joint audio-video generation, the audio category distribution layer must be built entirely from scratch.

### Narrative structure distribution (single-shot vs. multi-shot, average clip duration, shot-count distribution, presence of native audio track) ⚠️

【Single-shot vs. multi-shot】The framework's default output is single-shot clips: TransNetV2 performs shot-aware splitting, and in principle no produced clip spans multiple shots. But there is a noteworthy reverse-direction design in the pipeline — the stitching stage: after splitting, image-embedding similarity is used to judge whether adjacent clips are content-continuous, and if similarity is high, they are re-merged, to avoid over-fragmenting the same continuous scene (e.g., from brief camera occlusion or flash-induced false cuts). This "split then re-stitch" two-stage design is more robust than relying on scene detection alone.
【Shot-count distribution】The final dataset predominantly consists of single-shot clips; multi-shot narrative samples are not a construction goal, and there is no statistic on shot-count distribution.
【Average clip duration】In the range of 2–60 seconds; the specific mean has not been published.
【Presence of native audio track】The pipeline does not retain or process the audio track; the transcoded output is H.264 video, so the final WebDataset contains no audio track information. [Uncertain]

### Language/accent distribution (data foundation for multilingual lip-sync capability) ⚠️

Not addressed at all on the video side (since the audio track is not processed, there is no language dimension).
On the audio side there is indirectly relevant capability but no distribution statistics: the audio pipeline uses NVIDIA NeMo ASR models for transcription, with FLEURS (a multilingual speech benchmark covering 100+ languages) as the example dataset, and there is a long-form audio cutting tutorial involving speaker diarization. Starting in 26.02, streaming Sortformer is supported for speaker diarization, VAD, and speaker segmentation. However, the framework provides no language-identification (LID) stage, no accent-annotation capability, and no control mechanism or statistics for language/accent mixture ratios. The data foundation required for multilingual lip-sync cannot be directly produced by this framework. [Uncertain]

## Cleaning Pipeline

### Overall structure of the filtering funnel (number of stages, ordering)

This is where the core value of this entry lies. It is recorded at two levels.

【I. Cosmos WFM production pipeline: a seven-stage funnel (first-hand disclosure in arXiv:2501.03575)】
1. Shot-aware video splitting — TransNetV2 detects shot boundaries and cuts clips, with image-embedding similarity used for stitching;
2. GPU-based transcoding — uniformly converted to H.264 mp4, with NVDEC/NVENC hardware acceleration;
3. Video cropping — normalizing aspect ratio / removing letterboxing;
4. Filtering — motion filtering + visual quality filtering + text-overlay filtering + video-type taxonomy filtering, four discriminators chained in series;
5. Captioning — dense descriptions generated by a VLM;
6. Semantic deduplication — embeddings + k-means clustering + intra-cluster pairwise distance;
7. Sharding — packed into WebDataset shards bucketed by resolution/aspect ratio/duration.

【II. The two pipelines of the open-source NeMo Curator framework (arXiv:2503.12964 and official documentation)】
(A) Clipping Pipeline: decode the raw video → split into contiguous short clips based on frame-to-frame color change → merge adjacent clips via image-embedding-similarity stitching → transcode to H.264 → frame extraction (ClipFrameExtractionStage) → motion-vector decoding and motion filtering → aesthetic filtering → video-embedding annotation (Cosmos-Embed1) → VLM captioning (with optional LLM rewriting enhancement) → optional WebP preview thumbnails → semantic deduplication → write-out.
(B) Sharding Pipeline: generate text embeddings for the captions → produce WebDataset-format files (embeddings stored as Parquet, metadata retained), enabling PB-scale sequential-read, multi-GPU parallel access during training.

【III. Architectural characteristics】
- All stages are uniformly abstracted as ProcessingStage (each declares its own CPU/GPU resource requirements and input/output data contract), a Pipeline chains stages together, and execution is handled by a pluggable executor;
- The video pipeline defaults to the XennaExecutor (Cosmos-Xenna), which translates ProcessingStage into Xenna stage specs and executes them on Ray in either streaming or batch mode; there is also an experimental RayDataExecutor (promoted to stable in 26.04);
- Starting from 26.02, YAML declarative definition of an entire pipeline is supported, along with Pipeline.describe() for inspecting each stage's resource and data requirements during development;
- Starting from 26.07, pipeline resumability is supported: completed shards are recorded in LMDB and skipped on restart, with support for SLURM job arrays.

【IV. Structural evaluation】A typical five-stage design of "geometric/encoding normalization first → then discriminative filtering → then semantic annotation → then deduplication → finally bucketed packing for the training curriculum." Its biggest difference from a model team's self-built pipeline (e.g., LTX-Video) is that it folds sharding/WebDataset generation into the scope of curation, forming a closed loop between data processing and training data loading; and it executes in streaming rather than batch mode — stages run concurrently and data flows continuously, avoiding fully materializing intermediate artifacts to disk.

### Quantitative funnel retention rate (input/output volume and final retention rate at each filtering stage, e.g. Apollo's 27%) ⚠️

Only isolated quantitative data points exist — there is no complete, stage-by-stage funnel table.
【Published quantitative points】(1) Semantic deduplication removes approximately 30% of training data; (2) In visual quality filtering, the DOVER-based distortion assessment model discards the bottom 15% of scores; (3) The splitting stage discards all clips shorter than 2 seconds (proportion not published); (4) Overall, roughly 100 million 2–60-second clips are obtained from 20 million hours of raw video — a rough back-of-envelope calculation assuming an average clip duration of 15 seconds gives about 410,000 hours, implying a duration retention rate of roughly 2% relative to input, but this estimate is not rigorous (a large amount of content in the raw video is discarded already at the splitting stage, and the paper does not confirm the average clip duration).
【Missing】The individual attrition rates for motion filtering, text-overlay filtering, and video-type taxonomy filtering are not published; input/output sample-count tables for each stage are not provided; it is not possible to construct an end-to-end retention figure comparable to Apollo's 27%. [Uncertain]

### Shot-splitting method (PySceneDetect/in-house model/shot-aware splitting)

Well-documented with a clear rationale for the choice — the most rigorously argued step in this pipeline.
【Selection】A TransNetV2 neural network performs shot-boundary detection. The Cosmos WFM paper explicitly records a benchmark comparison against three alternatives — PySceneDetect, the Panda70M splitter, and AutoShot — with TransNetV2 achieving the best result, F1 = 0.967, on the BBC dataset, and being selected on that basis. This is a rare public case of treating "shot-splitter selection" as a quantifiable decision, and its comparison methodology is directly worth emulating.
【Framework implementation】NeMo Curator provides two composable splitting modes: fixed-stride and scene-change detection (TransNetV2).
【Post-processing】After splitting there is a stitching stage: image-embedding similarity is used to judge whether adjacent clips should be merged, suppressing over-fragmentation.
【Duration constraints】Clips < 2 seconds are discarded; clips > 60 seconds are further split.
【Not disclosed】The threshold value used for TransNetV2's decision, and the similarity threshold used for stitching.

### Quality filtering (aesthetic scoring, sharpness, OCR text filtering, letterboxing/watermark/logo detection)

Visual quality filtering (in the Cosmos WFM production version) consists of four discriminators, the first two of which are already available in open-source NeMo Curator:
【1. Aesthetic filtering (available open-source)】A CLIP-based aesthetic model scores each extracted frame, then results are aggregated per clip. Key parameters: reduction can be min (default) or mean — the default of taking the minimum implies a strict "one bad frame fails the whole clip" policy; score_threshold defaults to 3.5; target_fps defaults to 1.0; num_gpus_per_worker defaults to 0.25. It requires the upstream frame extraction to use the sequence strategy with a matching frame rate. The Cosmos WFM production environment uses this same threshold of 3.5.
【2. Distortion/image-quality assessment】A scoring model based on DOVER (a video-quality-assessment model) discards the bottom 15% of clips by score. This stage has no corresponding implementation in the open-source version.
【3. Text-overlay detection (a substitute for traditional OCR functionality)】Rather than traditional OCR, an MLP classifier is trained on top of InternVideo2 video embeddings to identify post-processed text overlays, discarding videos with excessive text. This approach of "using embeddings + a lightweight MLP in place of heavyweight OCR" offers significant cost advantages at PB scale and is a worthwhile engineering trade-off to emulate.
【4. Video-type taxonomy classification】Likewise InternVideo2 embeddings + MLP, used to exclude abstract patterns / game footage / animation (see domain_distribution).
【Not covered】Watermark/logo detection, letterboxing detection (cropping is a separate stage, but it is not stated whether letterboxing is automatically detected), compression-artifact detection, and a standalone blur criterion are all absent from the official documentation as dedicated stages. Discarded clips are moved into video.filtered_clips and statistics are updated accordingly, enabling post-hoc auditing of the funnel — a good observability design.

### Motion filtering (optical flow/motion-score thresholds, removal of static and shaky footage)

The most finely implemented and parameter-transparent filtering step in the open-source framework, chaining two stages:
【MotionVectorDecodeStage】Rather than performing full optical-flow computation, it decodes lightweight motion vectors directly from the compressed bitstream and computes a motion score from them — a significant cost optimization that avoids running an optical-flow network at PB scale. Parameters: target_fps defaults to 2.0 (frames sampled per second), target_duration_ratio defaults to 0.5 (only half the clip's duration is analyzed to save compute), num_cpus_per_worker defaults to 4.0, num_gpus_per_worker can be set to 0.5.
【MotionFilterStage】Applies threshold filtering. It produces two metrics: motion_score_global_mean (the global mean motion score, default threshold 0.00098) and motion_score_per_patch_min_256 (the minimum motion score across 256 spatial patches, default threshold 0.000001). The two-metric design serves a purpose: the global mean guards against "overall stillness," while the per-patch minimum guards against pseudo-dynamic samples where "most of the frame is static and only a small local object moves."
【Enhancement in the Cosmos WFM production version】There is also a ViT-based motion classifier trained on optical-flow sequences that, beyond removing static content, can also remove clips with erratic camera motion, and can tag the type of camera motion (pan / zoom / tilt) — i.e., motion filtering doubles as "camera-motion annotation," producing labels usable for camera-control training. This ViT classifier's weights are not provided in the open-source version.

### Deduplication method (exact deduplication and embedding-based semantic deduplication recorded separately) ⚠️

Primarily semantic deduplication, with a clear methodological chain:
【Embedding】Cosmos-Embed1 is used to generate clip-level video embeddings (in earlier work and in the Cosmos WFM production version, InternVideo2 was used instead; InternVideo2 has been removed from NeMo Curator as of version 26.02). Embeddings are persisted in Parquet format, with the same embedding set serving both semantic retrieval and deduplication.
【Clustering】K-means clustering is applied to the embeddings; the Cosmos WFM production environment uses k = 10,000.
【Intra-cluster comparison】Pairwise distances are computed only within the same cluster to identify near-duplicates, avoiding full N² comparison — this is the key to making deduplication feasible at PB scale.
【Retention policy】Within a near-duplicate group, the version with the highest resolution is kept.
【Effect】Roughly 30% of training data is removed.
【Framework implementation】The documentation describes it as "semantic clustering + pairwise similarity + k-means," organized by processing clips in chunks.
【Exact deduplication】No hash-based exact deduplication stage is seen on the video side (the text side has a mature exact-dedup and fuzzy-dedup system, with the latter officially claimed to achieve a 16x speedup and a 40% TCO reduction over a CPU-based approach on RedPajama v2 — but that figure is for the text modality and should not be transposed onto video). The similarity threshold value is not published. [Uncertain]

### VLM/LLM as data quality judge (multimodal large-model quality scoring and mismatch removal; the 2026 trend of shifting from shallow scorers to large-model semantic judgment)

Shows a clear generational evolution, making this a good sample for observing the 2026 trend from "shallow scorers → large-model semantic judgment":
【25.09 / early Cosmos production version: dominated by shallow discriminators】Quality-control duties were handled by lightweight models — a CLIP-based aesthetic model, the DOVER image-quality model, InternVideo2 embeddings + MLP (text-overlay detection, video-type classification), a ViT optical-flow motion classifier, and motion-vector statistics. All are small models or linear/MLP probes, motivated by unit-cost considerations at PB scale.
【26.02–26.07: VLMs/LLMs gradually enter the pipeline】(1) The captioning backend expanded from Qwen2.5-VL to Qwen3-VL and the Nemotron Nano 12B V2 VLM and Nemotron 3 Nano Omni, offering BF16/FP8/NVFP4 precision tiers to compress large-model inference cost — FP8/NVFP4 quantization is the precondition for making a 12B-class VLM run at PB scale; (2) 26.04 introduced vLLM as the default inference backend and built in a Ray Serve inference server (OpenAI-compatible interface), making "attaching an LLM to judge things within the pipeline" a first-class citizen; (3) starting in 26.02, captioning supports optional Qwen-LM rewriting enhancement; (4) in the 26.07 Nemotron OCR synthetic-data pipeline, Nemotron Nano Omni is explicitly used for optional quality scoring — this is the most direct instance of "a large model as data quality judge" in the official documentation.
【Still missing】The video side still has no dedicated removal stage for "a VLM judging whether the caption mismatches the footage," and no VLM semantic quality-scoring stage; large models are currently used mainly for generating captions rather than auditing them. This layer must be self-built by users.

### Safety and compliance filtering (NSFW, copyright, face/privacy) ⚠️

NeMo Curator's safety-filtering capability is severely unbalanced across modalities:
【Image modality】An NSFW detection stage is provided (explicitly listed by the official README as one of the core capabilities on the image side).
【Video modality】The official video-filtering documentation section contains only motion filtering and aesthetic filtering; it provides no NSFW classifier, no face detection/blurring, no privacy-information removal, and no copyright-fingerprint matching stage. The curation section of the Cosmos WFM paper likewise does not describe any data-side safety filtering — its safety capability is concentrated on the model-inference side, in Cosmos Guardrails (input prompt filtering + output content classification + face blurring), which is a deployment guardrail rather than training-data cleaning.
【Text modality】There is a quality-filtering and classifier system that can indirectly shoulder some content-safety responsibility.
【Conclusion】When using this framework to build video training data, all three compliance lines — NSFW, facial privacy, and copyright — must be supplemented by the user. [Uncertain]

## Annotation Methods

### Captioning model used (in-house VLM/open-source model, model scale)

This field carries the richest information across version iterations and is a key reference for model selection:
【Cosmos WFM production version (2025-01)】An internally fine-tuned VILA 13B is used as the captioner, deployed as an FP8-quantized TensorRT-LLM engine, achieving roughly a 10x speedup over unoptimized inference. Input is 8 frames uniformly sampled from the clip.
【NeMo Curator 25.09 / 26.02】The open-source version's default captioning backend is the Qwen-VL family (Qwen2.5-VL), with optional Qwen-LM-based rewriting enhancement of the generated caption (caption enhancement / rewriting).
【New in 26.04】The Nemotron Nano 12B V2 VLM is added as a captioning backend alongside Qwen-VL, offering three precision variants: nemotron / nemotron-bf16 (default BF16, auto-downloaded), nemotron-fp8 (FP8-quantized, lower memory footprint), and nemotron-nvfp4 (an NVFP4 quantization-aware-distilled checkpoint).
【Expanded in 26.07】The captioning backend matrix expands to Qwen2.5-VL, Qwen3-VL, and multiple Nemotron variants (including the Nemotron 3 Nano Omni all-modal model), each optionally in BF16/FP8/NVFP4; the video preprocessing option was also removed in favor of a unified preprocessing path through either vLLM or HuggingFace.
【Engineering takeaway】Captioning is explicitly identified by the official team as the rate-limiting stage of the entire pipeline — its throughput is significantly lower than that of other stages such as embedding generation, so Xenna's autoscaling assigns the largest number of workers to this stage. This observation is directly useful for anyone building their own video data pipeline.

### Caption density and structuring level (short/long/dense descriptions; structured fields such as camera motion, style tags) ⚠️

【Density】Follows a dense, long-description route. The Cosmos WFM production version's measured statistics: an average of 559 characters / 97 words per caption.
【Temporal granularity】Annotation granularity is "one caption per 256 frames" — i.e., a long clip is split into multiple time segments each described separately, rather than one caption for the whole span — giving captions a coarse-grained temporal correspondence.
【Sampling method】VLM input is 8 uniformly sampled frames.
【Enhancement process】The open-source version supports a two-stage process: the VLM first generates a base caption, which is then rewritten and enhanced by an LLM (Qwen-LM), usable for style unification, adding detail, or generating variants of different lengths.
【Downstream use】Captions are further encoded by a text-embedding model (starting in 26.04, semantic deduplication defaults to the vLLM-backend google/embeddings-gemma-300m embedding model); text embeddings and video embeddings are written into the WebDataset together.
【Structuring level】The official documentation does not publish any caption template, slot definitions, or structured-field specification (e.g., whether subject/action/scene/camera work/lighting are mandatorily covered), nor the full prompt text. What can be confirmed is that camera-motion type (pan/zoom/tilt) is produced by a separate motion classifier as a label, rather than relying on caption text — i.e., structured labels and natural-language descriptions are two parallel annotation systems. [Uncertain]

### Joint audio-video caption structure (whether both visual and auditory tracks are covered simultaneously; whether split into independent fields, e.g. LTX-2's full-soundscape description, Script-a-Video's factorized streams, Foley-Omni's three-field design)

None. This is the core gap in this toolchain for the joint audio-video generation scenario.
Video captioning describes only the visual track and touches no auditory content whatsoever (no field for music/ambient sound/foley/dialogue description); the audio pipeline only produces ASR transcripts and is not linked to the video footage. There is no caption schema in the framework capable of covering both the visual and auditory tracks simultaneously — neither an LTX-2-style fused full-soundscape long description, nor Script-a-Video's factorized streams, nor Foley-Omni's three-field design.
Nemotron 3 Nano Omni, introduced in 26.07, is an all-modal model with audio-video understanding capability, and in theory has the potential to produce joint captions, but according to official documentation its role is quality scoring for OCR synthetic data and serving as a video-captioning backend — no stage or example for joint audio-video annotation is seen. If this framework is used to serve an AV joint-generation model, the joint-caption layer must be entirely self-built.

### Dialogue transcription and speaker-attribute annotation (ASR transcription + speaker identity/language/accent/emotion) ⚠️

There is no dialogue transcription in the video pipeline (the audio track is not processed).
The audio pipeline has partial capability but is disconnected from video: (1) ASR transcription — the NVIDIA NeMo ASR model family generates transcript text; (2) transcription quality assessment — WER (word error rate) and CER (character error rate) are used as quality metrics for filtering (WER-filtering), the framework's most clearly defined quality-control mechanism on the speech-data side; (3) speaker diarization — starting in 26.02, streaming Sortformer is supported for speaker diarization and VAD (voice activity detection), with a long-form audio cutting tutorial available; (4) new in 26.07, an audio enhancement stage: tagging, SQUIM objective quality metrics, bandwidth estimation, punctuation preparation, and optional secondary ASR scoring.
【Missing】There is no built-in stage for speaker identity-attribute annotation, language identification, accent annotation, or emotion annotation; default thresholds for WER/CER are not given in the documentation. These capabilities are necessary for building multilingual lip-sync data and must be self-built. [Uncertain]

### Geometric and structured annotation (camera parameters, depth, 3D point tracks, action annotation, explicit state) ⚠️

Structured-annotation capability is limited but present in two places:
【1. Camera-motion labels】The motion classifier (ViT + optical-flow sequences) tags camera-motion type — pan, zoom, tilt — while filtering. This is an explicit structured label directly usable for camera-control training, and is a rare non-text annotation output from this pipeline.
【2. Motion score】The two numeric metrics motion_score_global_mean and motion_score_per_patch_min_256 are retained alongside each clip and can serve as continuous conditioning signals for motion intensity.
【3. Embeddings】Both the Cosmos-Embed1 video embedding and the caption text embedding are stored in Parquet and packed into the WebDataset, usable for retrieval, clustering, and conditional modeling.
【Missing】There is no built-in stage for camera intrinsic/extrinsic parameter estimation, depth maps, 3D point tracks, persisted optical-flow fields, object bounding boxes/segmentation masks, action-category labels, or explicit physical state. Relative to the Cosmos platform as a whole (which has other components such as Cosmos Transfer involving depth/segmentation conditioning), geometric annotation is a weak spot on the curation side. [Uncertain]

### Synthetic data construction (controlled perturbation/editing to construct training pairs, e.g. InstructAV2AV)

The curation framework itself does not construct synthetic video data, but related capability is evolving:
【Video side】There is no stage for constructing controlled-perturbation/editing training pairs (no editing-pair construction analogous to InstructAV2AV). The Cosmos WFM training data includes roughly 4% "synthetically rendered" video, but that is externally supplied footage, not something the pipeline generates.
【Text/multimodal side】Starting in 26.04, NeMo Data Designer (a synthetic-data design tool) is integrated; 26.07 adds a Nemotron OCR synthetic-data pipeline that can generate OCR training records from existing datasets, with optional quality scoring by Nemotron Nano Omni; 26.07 also introduces the Nemotron-CLIMB data-mixture optimization workflow (embedding–clustering–mixture-search over documents, converging on a final mixture from 64 candidate ratios).
It is evident that NVIDIA is bringing "synthetic data generation" and "automatic mixture optimization" into Curator's scope, but currently this covers only text and OCR — the video modality has not yet caught up.

### Degree of human involvement (human annotation, human QC, model pre-screening + human review) ⚠️

The framework is positioned as a fully automated pipeline, with no built-in human-annotation/review interface or workflow; the official documentation describes no human-in-the-loop step (no annotation-platform integration, no spot-check sampling stage, no human-arbitration queue).
Human involvement actually occurs at two points outside the framework: (1) construction of the discriminators — the text-overlay-detection MLP, video-type taxonomy MLP, and motion-classification ViT in Cosmos WFM all require human-annotated training sets, but the annotation scale and process are not disclosed; (2) manual setting of thresholds — the aesthetic threshold of 3.5, the motion threshold of 0.00098, DOVER's bottom 15%, k=10000, and other parameters are set by engineering judgment, with the framework only providing default values.
The framework substitutes alternative observability designs to reduce reliance on human effort: filtered clips are moved into video.filtered_clips with statistics updated; automatic performance-metric logging has been available since 26.02; resource and bottleneck monitoring dashboards are provided for each stage; and Pipeline.describe() supports development-time review — allowing engineers to audit the funnel after the fact rather than manually inspecting samples one by one. [Uncertain]

## Audio-Video Alignment

### Audio-video synchronization detection method (lip sync, event alignment)

None. The framework has no audio-video synchronization-detection capability whatsoever: the video pipeline does not parse the audio track, so there is no lip-sync detection, event-alignment detection, or any stage for removing asynchronous samples; the audio pipeline processes independent audio files with no corresponding video frames to align against. Neither the official documentation, the Cosmos WFM paper, nor the NeMo video foundation model training paper mentions any synchronization-detection component such as SyncNet, Synchformer, or AV-align.
This constitutes the primary gap in NeMo Curator when it comes to serving joint audio-video generation models — although it is the de facto standard for "large-scale visual data processing," the core step of AV data construction (synchrony screening) must be entirely self-built.

### Specific synchronization-detection metrics and thresholds (SyncNet/Synchformer/LSE/in-house, threshold values, e.g. MOVA's LSE-D≤9.5 and LSE-C≥4.5, SkyReels-V4's SyncNet |offset|≤3∧conf>1.5)

None. There is no synchrony metric or threshold of any kind (no SyncNet confidence/offset, no LSE-D/LSE-C, no in-house sync score), and no related configuration item. For reference, one can point to this framework's other threshold systems, which are fully transparent in their parameters: aesthetic score_threshold = 3.5, motion motion_score_global_mean = 0.00098, motion_score_per_patch_min_256 = 0.000001, DOVER discarding the bottom 15%, clip duration 2–60 seconds, semantic deduplication k-means k = 10,000. This practice of "writing every default threshold into the documentation" is itself worth emulating, but none of it involves any audio-video dimension.

### Separate handling of temporal sync vs. semantic sync (temporal alignment and content-semantic matching as two independent filtering conditions)

Not applicable. The framework does not process the audio-video relationship, so there is no separation of temporal synchronization from semantic matching.
If forced into an analogy, there is in fact a division of labor on the video side between "structural indicators" and "semantic indicators": the motion-vector score and DOVER image quality belong to the structural/signal layer, handled by lightweight statistics and small models; the video-type taxonomy and text-overlay detection belong to the semantic layer, handled by embeddings + MLP; caption generation belongs to the semantic-description layer, handled by a VLM. This idea of "assigning different scales of model according to the nature of the criterion" could be transferred to the design of AV sync filtering (e.g., first coarsely screening temporal alignment with lightweight energy-envelope correlation, then using a large model for semantic-match review) — but the framework itself provides no such implementation.

### Audio quality filtering (SNR, silence detection and silence-ratio thresholds, removal of tracks with no audio, removal of off-screen sound sources, background-music separation) ⚠️

None on the video pipeline side (the audio track is not processed, so there is no removal of no-audio-track samples, no silence-ratio threshold, no off-screen-source removal, and no background-music separation).
The independent audio pipeline has quality filtering aimed at speech data: (1) WER/CER filtering — using ASR transcription error rate as a proxy quality metric to remove low-quality audio, the primary mechanism; (2) audio analysis — duration calculation and format validation; (3) VAD (voice activity detection), available starting in 26.02, usable for locating valid speech segments and removing silence; (4) new in 26.07, SQUIM objective quality metrics (which can estimate SNR, STOI, PESQ, and other speech-quality dimensions without a reference) and bandwidth estimation (identifying upsampled fake-high-sample-rate audio), with optional secondary ASR scoring for cross-validation.
【Missing】There is no built-in stage for source separation (vocal/accompaniment separation), background-music detection, sound-effect-event detection, or loudness normalization; default thresholds for each metric are not given in the documentation. [Uncertain]

### Classification and separate handling of speech/sound-effects/music ⚠️

No classification/routing mechanism. The audio pipeline's implicit assumption is "the input is speech data" — all stages (ASR transcription, WER/CER filtering, VAD, speaker diarization, SQUIM quality assessment, punctuation preparation) serve speech, with no distinction or separate handling of sound effects (foley), music, or ambient sound. The framework provides no audio-event classifier (e.g., an AudioSet-style label taxonomy), no music detector, and no source-separation module.
The audio tagging stage in 26.07 is the closest capability, but per the documentation its purpose is speech-attribute annotation rather than classification/routing across the four audio types. [Uncertain]

## Training Integration

### Multi-stage training curriculum and data-curriculum scheduling (basis for stage division: resolution/duration/quality score/modality; low-res→high-res, image→video, short→long)

As a data infrastructure component, it does not define the training curriculum, but its output format is designed to serve curriculum-based training — this is the design point where it is most tightly coupled with the training side:
【Bucketing as the curriculum interface】The sharding stage packs clips into different WebDataset shards along three dimensions — resolution × aspect ratio × duration — with the official documentation explicitly stating the goal is "to align with the training curriculum." Downstream training frameworks can thereby implement a progressive curriculum from low-res→high-res and short→long simply by switching which shard buckets are read, without reprocessing data.
【Quality scores persisted alongside data】Aesthetic score, motion score, type label, and other discriminative results are saved in metadata alongside each clip, letting the training side dynamically filter subsets by quality score (e.g., taking only high-aesthetic-score samples during an annealing phase) without rerunning the pipeline.
【Integration with NeMo training】NeMo Framework's video foundation model training stack (arXiv:2503.12964) connects directly with the WebDataset output by Curator, supporting multi-GPU sequential reads at PB scale.
The specific basis for dividing stages and the criteria for switching between them are decided by the user's training configuration — Curator does not prescribe these.

### Changes in data mixture ratio across training stages (pretraining/annealing/SFT high-quality subset) ⚠️

The framework does not prescribe mixture ratios, but it provides two types of infrastructure for mixture control:
【1. Static mixture control (already validated in production on the video side)】Cosmos WFM implements an explicit nine-category mixture ratio through a taxonomy classifier plus up/downsampling (nature dynamics 20%, hand manipulation 16%, spatial navigation 16%, driving 11%, etc. — see domain_distribution). This is the standard approach of "classifier tagging → resampling to a target ratio."
【2. Automatic mixture optimization (new in 26.07, currently text modality only)】The Nemotron-CLIMB workflow: embed documents → cluster → search the mixture space for an optimum → iteratively converge to a final mixture from 64 candidate mixture proposals. This is a systematic attempt to treat "data mixture ratio" itself as an optimizable object; if it is extended to the video modality in the future it would have direct value, but it does not currently support video.
【Missing】How the mixture ratio should change across pretraining/annealing/SFT stages is a matter of the user's training strategy — Curator neither addresses it nor offers a reference scheme. [Uncertain]

### Post-training data (scale and screening criteria of the curated SFT set, number of preference pairs and how they are annotated, reward-model training data) ⚠️

Not applicable / not covered. NeMo Curator's video side provides no post-training-data construction capability: no preference-pair construction stage, no reward-model training-data generation, and no automatic screening workflow for an SFT-curated set.
What can indirectly support SFT-subset screening is its metadata system — aesthetic score, motion score, type label, and deduplication-cluster information are persisted alongside the data, letting users cut out a high-quality subset via thresholds (Cosmos WFM did indeed cut roughly 10^7 clips for fine-tuning out of roughly 10^8 pretraining clips, but the screening criteria are not published).
On the text-modality side, the NeMo Data Designer integrated in 26.04 and the synthetic-data pipeline in 26.07 partially cover post-training data generation, but this has not extended to video. [Uncertain]

### Data-processing infrastructure and throughput (NeMo Curator/Data-Juicer/in-house, GPU acceleration ratio, processing scale, cost) ⚠️

This is the core field of this entry, and the basis on which this toolchain is called the "de facto standard for industrial-grade video data infrastructure."
【Overall speedup】Official figure: using 1,000 GPUs at ISO power (equal power draw), it achieves an 89x speedup over an unoptimized CPU pipeline, compressing processing time from "years" to "days." Specifically for 20 million hours of video: about 40 days on a Hopper cluster, about 14 days on Blackwell, versus about 3.4 years on an unoptimized CPU pipeline.
【Breakdown of speedup sources】(1) NVDEC hardware decoding + NVENC hardware encoding — decoding and transcoding achieve roughly a 3x speedup (per the NeMo paper); the Cosmos paper gives a transcoding throughput of 0.3702 videos/second on L40S, a 6.5x improvement over baseline; (2) FP8-quantized TensorRT-LLM engine for VLM captioning — roughly a 10x speedup; (3) Ray distributed scheduling and autoscaling eliminate GPU idle time; (4) streaming execution avoids fully materializing intermediate artifacts to disk.
【Autoscaling mechanism (the core of Cosmos-Xenna)】In streaming mode, the throughput (samples/second) of every stage is continuously measured, and the worker count of each stage is dynamically adjusted to balance the whole pipeline: workers are scaled down on fast stages and scaled up on the bottleneck stage. A sophisticated bin-packing algorithm optimally packs workers onto cluster nodes to maximize utilization. Default parameters: autoscale_interval_s = 180 seconds (autoscaling check interval), logging_interval = 60 seconds, cpu_allocation_percentage = 0.95, execution_mode can be set to streaming (recommended) or batch. There is also a serving mode supporting queue-based real-time input/output.
【Key engineering conclusion】Captioning is explicitly identified as the rate-limiting stage (its throughput is far lower than that of embedding generation, for instance), so autoscaling assigns it the most resources — any self-built video data pipeline should expect a similarly distributed bottleneck.
【Other mechanisms】Backpressure control prevents memory overflow; CPU-intensive stages do not block GPU stages; stateful actors ensure model weights are loaded only once per worker; SPMD mode supports distributed inference for large models requiring tensor parallelism across GPUs/nodes; P2P artifact distribution speeds up downloading model files across multiple nodes.
【Heterogeneous clusters】Supports heterogeneous clusters mixing GPU types such as L40S, H100, and GB200, able to take targeted advantage of specific GPUs' NVENC capability.
【Ray Object Store】Uses the Ray Object Store and object references to minimize data movement.
【Text-modality comparison figures】Fuzzy deduplication achieves a 16x speedup and a 40% TCO reduction over a CPU-based approach on RedPajama v2, with near-linear scaling from 1→4 nodes — note this is a text-side figure and should not be conflated with the video-side 89x.
【Operations】Starting in 26.07, LMDB records completed shards to support resumable runs, with SLURM job array support; starting in 26.02, automated stage- and pipeline-level benchmarking is provided for each modality.
【Cost】No per-unit-data dollar cost has been published. [Uncertain]

## Comparative Results

### Quantitative impact of data-strategy ablations (distinguishing: filtering-strictness ablation / caption-density-and-style ablation / data-mixture ablation, and the corresponding evaluation metrics) ⚠️

As a toolchain, it has no data-strategy ablation experiments of its own, but its upstream Cosmos WFM paper contains several quantitative comparisons that can be regarded as "selection ablations," which are the closest thing to ablation evidence in this entry:
【1. Shot-splitter selection comparison】TransNetV2 vs. PySceneDetect vs. the Panda70M splitter vs. AutoShot, benchmarked by F1 on the BBC dataset, with TransNetV2 winning at F1 = 0.967. This is direct quantitative evidence for the splitting-method selection.
【2. Transcoding-speedup comparison】On L40S, the NVDEC/NVENC hardware path achieves a 6.5x throughput improvement over baseline (0.3702 videos/second); the NeMo paper separately gives roughly a 3x speedup for decoding and transcoding.
【3. Captioning-inference speedup】The FP8-quantized TensorRT-LLM engine gives roughly a 10x speedup.
【4. Overall pipeline】An 89x speedup over an unoptimized CPU pipeline at ISO power with 1,000 GPUs.
【Missing】There is no ablation of "filtering strictness → generation quality" (e.g., the effect of an aesthetic threshold of 3.5 vs. 4.0 on the final model's FVD/human evaluation has not been measured); no ablation of caption density/style; no ablation of the nine-category mixture ratio (the mixture figures are given as-is, with no argument for why they are 20%/16%/16%…). That is, every quantitative conclusion is along the "processing efficiency" dimension, and none is along the "impact of data strategy on model quality" dimension. This follows from the toolchain's positioning as infrastructure, but it also means its default thresholds lack empirical support on the effectiveness side, and users should not adopt them uncritically. [Uncertain]

### Evidence for quality vs. quantity (cases where small, curated data outperforms large, messy data) ⚠️

There is no direct "small and curated beats large and messy" controlled experiment. But the design philosophy of the entire pipeline is itself an engineered expression of that claim, with a clear quantitative endpoint: starting from 20 million hours of raw video, and passing through seven gates — shot splitting (discarding < 2 seconds), motion filtering (removing static and out-of-control shaky footage), image-quality filtering (DOVER discarding the bottom 15%), aesthetic filtering (threshold 3.5, with the strict default of min aggregation), text-overlay filtering, taxonomy filtering (wholesale removal of animation/games/abstract patterns), and semantic deduplication (removing roughly 30%) — ultimately only roughly 10^7-scale clips enter the fine-tuning stage.
The Cosmos paper's own statement of the curation goal is itself a quality-first manifesto: the aim is "to locate clips within videos that possess rich dynamics and high visual quality, conducive to learning the physical laws encoded in visual content." Additionally, the "upsample human motion/object manipulation, downsample landscapes" strategy is also a classic case of judging "information density over material quantity." But these are all design intentions and experience-based judgments — no A/B controlled quantitative evidence is provided. [Uncertain]

### Alignment between training-data domain distribution and the evaluation-benchmark category system (e.g. VABench's seven major categories) ⚠️

The alignment between training-data domain distribution and the evaluation system is not explicitly argued, but the design motivation of its taxonomy clearly points toward downstream capability dimensions:
Cosmos's nine categories (nature dynamics 20%, hand manipulation 16%, spatial navigation 16%, driving 11%, human motion 10%, first-person POV 8%, dynamic camera 8%, other 7%, synthetically rendered 4%) are organized around the target application scenarios of Physical AI — robotics (hand manipulation, human motion), autonomous driving (driving), embodied navigation (spatial navigation, first-person POV) — i.e., the data mixture maps directly onto application domains rather than evaluation-benchmark categories.
【Comparison with this survey】This system does not map directly onto category systems such as VABench's seven major categories, which are oriented toward general text-to-video/audio-video generation evaluation: the Cosmos taxonomy explicitly excludes animation, games, and abstract style (which are precisely important dimensions in general video-generation evaluation), does not include categories such as close-up portraits/dialogue/cinematic feel that AV generation evaluation cares about, and has no audio dimension at all.
【Conclusion】What NeMo Curator provides is the reusable mechanism of "taxonomy-driven mixture control," while its built-in/example category system is custom-tailored for world models — when serving AV generation models, the category definitions need to be redesigned and the classifier retrained. NVIDIA has not provided any argument for benchmark-alignment. [Uncertain]

## Other Information

### summary_note

【Positioning judgment】NeMo Curator + Cosmos-Xenna is currently the only publicly available, production-validated-at-PB-scale open-source video data-processing infrastructure. Its irreplaceable value lies in three places: (1) the streaming autoscaling architecture of Ray + Cosmos-Xenna, which solves the load-balancing problem of a multi-stage, heterogeneous-compute pipeline (key insight: captioning is a perpetual bottleneck); (2) end-to-end GPU-ification via NVDEC/NVENC hardware codecs and FP8/NVFP4 quantized inference, which is the real source of the 89x speedup; (3) bringing sharding/WebDataset generation — bucketed by resolution × aspect ratio × duration — into the scope of curation, closing the loop between data processing and the training curriculum. In addition, all of its default thresholds are written into the documentation (aesthetic 3.5, motion 0.00098, DOVER bottom 15%, clip 2–60 seconds, k-means k=10000), and its F1 benchmark comparison for shot-splitter selection makes it the most transparent among comparable resources.
【Applicability to audio-video generation data】Its boundaries must be made explicit: the framework's four modalities are "four mutually disconnected pipelines" — the video side does not parse the audio track at all, and the audio side only serves ASR speech data. Everything needed for the joint audio-video scenario — audio-track extraction, AV sync detection (SyncNet-class), audio-category classification (speech/foley/music/ambient sound), a joint caption schema, and an audio-quality threshold system — is missing. The realistic use pattern, therefore, is to use NeMo Curator as the visual-side skeleton and distributed execution substrate, inserting audio-related stages oneself on top of its ProcessingStage abstraction — the framework's pluggable stage design, together with the built-in Ray Serve/vLLM inference server since 26.04, make this kind of extension engineeringly feasible, which is the correct positioning of it as "infrastructure" rather than a "solution."
【Main version-evolution thread】25.09 completed the Dask→Ray architectural refactor and introduced video/audio; 26.02 filled in engineering capability (YAML declarative pipelines, all-modality benchmarking, removal of InternVideo2); 26.04 turned toward large-model friendliness (vLLM by default, Ray Serve inference server, Nemotron VLM captioning backend, Cosmos-Xenna 0.2.0); 26.07 moved toward production operations and automated data science (LMDB resumable runs, SLURM job arrays, audio enhancement stage, Nemotron-CLIMB automatic mixture optimization, OCR synthetic data). The trend is clear: discriminative duties are migrating from shallow scorers (CLIP aesthetics, MLP probes) toward large, quantized models, and mixture-ratio decisions are migrating from manual setting toward automatic search — but both trends have so far only landed in the text modality, with the video modality yet to catch up.

## Uncertain Fields

Research information for the following fields is partially uncertain (sources marked with ⚠️):

- data_sources
- provenance_licensing
- duration_distribution
- resolution_aspect_distribution
- narrative_structure_distribution
- language_accent_distribution
- funnel_retention_rate
- deduplication
- safety_filtering
- caption_structure
- dialogue_transcription_attributes
- geometric_structured_annotation
- human_in_loop
- audio_quality_filtering
- audio_type_handling
- stage_data_mixture
- post_training_data
- data_infra_throughput
- data_ablation
- quality_over_quantity_evidence
- benchmark_taxonomy_alignment
