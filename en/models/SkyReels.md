# SkyReels Series (this entry covers SkyReels-V2 "SkyReels-V2: Infinite-length Film Generative Model" arXiv:2504.13074, and SkyReels-V4 "SkyReels-V4: Multi-modal Video-Audio Generation, Inpainting and Editing model" arXiv:2602.21818; the intermediate version SkyReels-V3 was open-sourced on January 29, 2026, featuring three major capabilities — reference-image-to-video R2V, video extension V2V, and audio-driven digital human Talking Avatar — as a transition between V2 and V4)

> Topic: Data processing for video generation models (including simultaneous audio-video generation): data cleaning pipeline, data distribution, annotation methods

[← Back to Home](../index.md)

**Table of Contents**: [Basic Information](#basic-information) · [Data Scale and Distribution](#data-scale-and-distribution) · [Cleaning Pipeline](#cleaning-pipeline) · [Annotation Methods](#annotation-methods) · [Audio-Video Alignment](#audio-video-alignment) · [Training Coordination](#training-coordination) · [Performance Comparison](#performance-comparison)

## Basic Information

### Name

SkyReels Series (this entry covers SkyReels-V2 "SkyReels-V2: Infinite-length Film Generative Model" arXiv:2504.13074, and SkyReels-V4 "SkyReels-V4: Multi-modal Video-Audio Generation, Inpainting and Editing model" arXiv:2602.21818; the intermediate version SkyReels-V3 was open-sourced on January 29, 2026, featuring three major capabilities — reference-image-to-video R2V, video extension V2V, and audio-driven digital human Talking Avatar — as a transition between V2 and V4)

### Publishing Organization/Company

The SkyReels team under Kunlun Tech (Skywork AI, Tiangong AI). The V4 paper has 50+ authors; project lead Guibin Chen (guibin.chen@kunlun-inc.com), project initiator Yahui Zhou (周亚辉); the team is organized into infrastructure, data processing, video model training, audio model training, multi-modal training, and evaluation groups.

### Release Date (technical report/paper/open-source date)

SkyReels-V1: open-sourced February 18, 2025 (a character-centric video foundation model). SkyReels-V2: paper submitted to arXiv:2504.13074 on April 17, 2025 (v3 revised April 21), with inference code and SkyCaptioner-V1 open-sourced on April 21, 2025, and the 720P version released as a supplement on April 24. SkyReels-V3: open-sourced January 29, 2026. SkyReels-V4: paper submitted to arXiv:2602.21818 on February 24/25, 2026 (v3 version dated March 18, 2026), officially launched as a product and API at the Zhongguancun Forum on March 27, 2026; as of March 18, 2026, it ranked first on both the "text-to-video (with audio)" and "image-to-video (with audio)" leaderboards, and second on "text-to-video (without audio)" on the Artificial Analysis Video Arena.

### Type (model/dataset/toolchain/evaluation benchmark)

Primarily models, with toolchain and evaluation benchmark outputs as well: (1) Models — the SkyReels-V2 video generation foundation model (1.3B/5B/14B, 540P/720P, including T2V, I2V, Diffusion Forcing long video, and Camera Director variants) and the SkyReels-V4 unified multi-modal audio-video foundation model (generation + inpainting + editing); (2) Toolchain — SkyCaptioner-V1, a structured video annotation model (based on Qwen2.5-VL-7B-Instruct, open-sourced) with its inference code, and the SkyReels-V2 training/inference framework; (3) Evaluation benchmarks — SkyReels-Bench (V2, a human evaluation benchmark) and SkyReels-VABench (V4, a set of 2000+ audio-video joint evaluation prompts).

### Openness (whether weights/code/data/pipeline are each open source) ⚠️

Exhibits a tiered strategy of "fully open-source early versions, closed-source productized latest flagship."
[SkyReels-V1/V2/V3: weights + code open source] V2 released the full 1.3B/5B/14B weight series on Hugging Face and ModelScope (DF long video, T2V, I2V, 540P/720P), along with open-sourced inference code and SkyCaptioner-V1 annotation model weights; V3 (R2V/V2V/Talking Avatar, 14B) was open-sourced in January 2026. The open-source models in the SkyReels series have accumulated nearly 300,000 downloads on HuggingFace and over 10,000 GitHub stars.
[SkyReels-V4: no weight release observed as of July 2026] The paper makes no commitment to any code/weight release and includes no GitHub link; it is offered externally as the skyreels.ai product (limited preview + free quota) and an open API, covering text-to-video, image-to-video, multi-modal reference generation, video editing/repair, and joint audio-video generation. Some Chinese media state that "V1–V4 are all open source," but no SkyReels-V4 weight repository can be found on HuggingFace/GitHub, so it should be understood as "weights not yet open-sourced." [Uncertain]
[Data side] The training data itself, the data cleaning pipeline code, and internal quality-inspection models for all three versions are unreleased; only the model weights for the SkyCaptioner-V1 annotation stage are open source — this is the most valuable open asset on the data side for this series.

### Whether it supports simultaneous audio-video generation, and the implementation approach (native joint / cascaded / MoE fusion)

SkyReels-V2 does not support audio generation (a pure video model). SkyReels-V4 supports native joint audio-video generation, implemented as a "dual-stream MMDiT (dual-stream Multimodal Diffusion Transformer)," which is native joint generation rather than cascaded or MoE:
(1) The video branch and audio branch form a symmetric twin backbone; the video branch is initialized from an existing text-to-video model, while the audio branch is trained from scratch, with both aligned in specification;
(2) They share a frozen multi-modal large language model (MLLM) as the text encoder, which directly processes descriptive text that combines "visual + auditory" content, allowing a single prompt to drive both modalities simultaneously;
(3) Each transformer block has bidirectional audio-video cross-attention internally (audio→video and video→audio condition each other), with dual-stream MM layers in the early stages transitioning to single-stream hybrid blocks later;
(4) Temporal alignment between the two modalities is achieved via RoPE temporal-dimension scaling (the audio/video token ratio is approximately 0.0963; on the audio side, 44.1kHz and 5 seconds correspond to 218 latent tokens, aligned to the temporal resolution of 21 video frames);
(5) Generation capability: up to 1080p, 32FPS, 15 seconds, supporting multi-shot, film-level content with synchronized audio tracks;
(6) Efficiency strategy: the base model first generates a "low-resolution full sequence + high-resolution keyframes," which are then finalized by a Refiner module (joint super-resolution and frame interpolation based on Video Sparse Attention);
(7) Unification: through a channel-concatenation inpainting framework plus different mask configurations, generation, inpainting, and editing are unified into a single model, accepting text, image, video clip, mask, and audio reference as inputs.

### List of research information sources (URLs of papers/technical reports/official documentation/news, with each source's nature noted: primary official / same-team corroboration / third-party report)

- Primary official | https://arxiv.org/abs/2602.21818 — SkyReels-V4 technical report (submitted 2026-02-24, v3 2026-03-18), including data collection/processing/annotation, a six-stage training table, SyncNet thresholds, and SkyReels-VABench
- Primary official | https://arxiv.org/html/2602.21818v3 — full HTML text of the V4 paper (data section, Table 1 training stage table)
- Primary official | https://arxiv.org/abs/2504.13074 — SkyReels-V2 technical report (2025-04-17), including the data pipeline, SkyCaptioner-V1, multi-stage training, and RL
- Primary official | https://github.com/SkyworkAI/SkyReels-V2 — official V2 repository (weight list, SkyCaptioner-V1, release timeline)
- Primary official | https://huggingface.co/Skywork/SkyCaptioner-V1 — SkyCaptioner-V1 model card (Qwen2.5-VL-7B-Instruct base, 10 million → 2 million concept-balanced data, 32×A800, accuracy)
- Primary official | https://github.com/SkyworkAI/SkyReels-V3 — official V3 repository (open-sourced 2026-01-29)
- Third-party analysis | https://blog.csdn.net/Together_CZ/article/details/148583114 — a detailed CSDN Chinese summary of the V2 paper's data section (O(100M) scale, 280,000 movies/800,000 TV episodes/6.2 million hours, filter list, manual spot-check rate, DPO data volume)
- Third-party analysis | https://www.alphaxiv.org/overview/2504.13074 — alphaXiv's structured analysis of V2 (100 million video samples, PyDetect+TransNet-V2, Qwen2.5-VL-7B, 30k sample pairs)
- Third-party analysis | https://www.alphaxiv.org/overview/2602.21818 — alphaXiv's analysis of V4 (five VABench dimensions, bidirectional cross-attention, Refiner)
- Third-party analysis | https://www.emergentmind.com/papers/2504.13074 — a summary of V2's data pipeline and training curriculum (dual-axis bucketing, four-stage post-training)
- Third-party report | https://finance.sina.com.cn/tech/roll/2026-01-30/doc-inhkakfi6974616.shtml — Sina Finance: report on SkyReels-V3 open-sourcing
- Third-party report | https://tech.cnr.cn/techgd/20260328/t20260328_527565454.shtml — CNR: Kunlun Tech launches SkyReels-V4 at the 2026 Zhongguancun Forum
- Third-party report | https://wavespeed.ai/blog/posts/what-is-skyreels-v4/ — V4 capabilities and description of the "weights not yet released, limited preview" status
- Third-party report | https://comfyui-wiki.com/en/news/2025-04-21-skyreels-v2-infinite-length-film-generative-model — V2 release and VBench 83.9% overall score

## Data Scale and Distribution

### Training data volume (number of videos/hours/tokens, pretraining and SFT separated)

[SkyReels-V2] The raw dataset scale is O(100M) (approximately 100 million video samples), accompanied by O(100M)-scale concept-balanced image data used in joint training. In-house media is estimated to total "6.2M+ hours (over 6.2 million hours)," sourced from "280,000+ movies" and "800,000+ TV episodes" from more than 120 countries. The training set for the annotation model SkyCaptioner-V1 is "approximately 2 million concept-balanced videos curated from an initial 10 million samples." During post-training, concept balancing "reduced the data volume by 50%."
[SkyReels-V4] Data volumes are given by training stage (Table 1): Stage1 text-to-image, 3 billion images (3 epochs); Stage2 image-text + video, 1 billion images + 400 million videos (3 epochs); Stage3 inpainting tasks, likewise 1 billion images + 400 million videos (2 epochs); Stage4 mixed resolution, 100 million each (2 epochs); Stage5 high resolution 480/720/1080p, 50 million each (2 epochs); Stage6 multi-modal conditioning, 20 million images + 50 million videos (2 epochs); audio backbone pretraining uses "hundreds of thousands of hours," with each clip at most 15 seconds (3 epochs); joint audio-video training uses "50% video data + T2A data" (2 epochs); SFT stage one uses 5 million videos (20% with multi-modal conditions), SFT stage two uses 1 million manually curated high-quality videos. Total video hours and token counts are not given.

### Data source composition (proprietary/public datasets/web crawling/licensed procurement/synthetic data)

[SkyReels-V2] Three clearly identified sources: (1) general open-source datasets, naming Koala-36M and HumanVid; (2) in-house media — 280,000+ movies and 800,000+ TV episodes (estimated 6.2 million+ hours) from more than 120 countries, i.e., primarily high-quality footage from mainstream film and television; (3) artistic repositories — high-quality video assets sourced from the internet.
[SkyReels-V4] Split into "real data + synthetic data." Real data = public datasets + licensed proprietary content: named image datasets are LAION and Flickr; named video datasets are WebVid-10M, Koala-36M, and OpenHumanVid; named audio datasets are Emilia, AudioSet, and VGGSound (some materials also list SoundNet); the proprietary side consists of "licensed movies, TV series, short videos, web series" — forming a data loop with Kunlun Tech's short-drama business (80 million monthly active users in the short-drama scenario). Synthetic data is used to fill three gaps: multilingual on-screen text generation, multilingual text-to-speech (TTS), and paired data for inpainting/editing. The proportions of each source are not disclosed.

### Data compliance and provenance (proportion of licensed data, rights-cleared datasets, C2PA, etc.) ⚠️

Neither paper generation provides quantitative compliance disclosure. Only qualitative statements exist: V4 states that its proprietary data is "licensed" film, TV, short-video, and web-drama content, while V2 states that its self-collected media and "artistic repositories" are used, but neither gives the proportion of licensed data, a list of rights-cleared datasets, a list of rights holders, or the procurement method. Neither paper mentions C2PA, invisible watermarking, output provenance, or content-source identification mechanisms, nor is there any NSFW/privacy compliance section. On the public dataset side, LAION, WebVid-10M, and other corpora known to have copyright disputes are used, without any explanation from the papers. As a generative service operating within China, the SkyReels product should be subject to the "Interim Measures for the Management of Generative Artificial Intelligence Services" and deep-synthesis labeling requirements, but the papers do not mention filing or labeling implementation. [Uncertain]

### Clip duration distribution and segmentation strategy ⚠️

[SkyReels-V2] No duration histogram is given, but "dual-axis bucketing" is explicitly used: samples are organized into a BT×BAR bucket matrix along the two dimensions of temporal length and spatial aspect ratio, with FPS normalization applied; the segmentation granularity is single-shot clips (all raw video is first segmented into single-shot clips via shot boundary detection). The Diffusion Forcing stage further supports variable-length, even theoretically infinite-length generation (the paper markets "infinite duration" as a selling point, with 30-second/60-second-class clips demonstrated to be stably generated in practice). The specific bucket boundaries and per-bucket proportions are not published.
[SkyReels-V4] The unified duration cap for training and generation is 15 seconds; Stage3 explicitly states a training clip duration range of "2–15 seconds." The audio side applies explicit duration control: long audio is cut into 15-second chunks, and overly short audio is concatenated by category to fill up to 15 seconds — this is the most concrete duration-handling strategy in this entry. The proportion of video samples in each duration range is not disclosed. [Uncertain]

### Resolution/aspect-ratio distribution and bucketing strategy ⚠️

[SkyReels-V2] Aspect ratio is explicitly managed via the BAR axis of "dual-axis bucketing" (duration × aspect-ratio bucket matrix), with FPS normalization; the resolution dimension is managed via a progressive curriculum: pretraining stage 1 is 256p, stage 2 increases to 360p, stage 3 increases to 540p, and post-training's final SFT raises it to 720p. Basic quality filtering removes low-resolution and low-frame-rate sources and crops out black bars and split-screen content. The specific proportions of each resolution/aspect-ratio bucket are not published.
[SkyReels-V4] Resolution and data volume are assigned by stage: Stage1/2/3 use 256px (16fps), Stage4 mixes 256/480px (100 million each), Stage5 mixes 480/720/1080px (50 million each), and Stage6 and SFT use 720/1080px. The generation cap is 1080p/32FPS. Aspect-ratio bucketing strategy is not separately described. The proportion of samples at each resolution can be indirectly inferred as roughly equal, based on the "equal amount per tier" phrasing above. [Uncertain]

### Category/domain distribution and mixture strategy (proportional control and concept balancing for people, actions, scenes, styles, etc.) ⚠️

This is the most prominent methodology on the data side across the SkyReels series — "concept balance" runs through both generations.
[SkyReels-V2] (1) Annotation first: the "subject category" field in the structured output of SkyCaptioner-V1 serves as the classification basis for balancing; (2) balancing is executed during post-training — "detailed concept balancing is performed based on the subject category from the captioner, which reduced the data volume by 50%," i.e., the team was willing to cut half the data to eliminate head-category bias, a direct manifestation of "quality over quantity"; (3) the pretraining stage also uses "concept-balanced image data" jointly trained with video; (4) at the video-type level, a blacklist-style exclusion is applied: surveillance footage, game recordings, animation, and meaningless content are excluded as entire categories, indicating the target domain is locked onto genuinely filmed movie/TV/life content; (5) SkyCaptioner-V1's own training set is likewise "2 million concept-balanced videos curated from 10 million." The paper does not disclose the specific category taxonomy or per-category proportion figures.
[SkyReels-V4] Continued and expanded into two orthogonal dimensions: (1) conceptual diversity — matching-based balancing according to a taxonomy; (2) motion diversity — defining key motion patterns for each subject or scene category, then balancing by motion pattern, to avoid a single subject category appearing with only one type of action. This "taxonomy × motion pattern" two-dimensional balancing is the main increment of V4 relative to V2. The specific category tree and proportion figures are not disclosed. [Uncertain]

### Audio category distribution and mixture (proportions and control strategy for speech/sound-effects foley/music/ambient sound/silence) — a dimension unique to AV models ⚠️

Only SkyReels-V4 addresses this; the strategy is clear but no proportion figures are given.
[Taxonomy] Audio is explicitly divided into four categories: sound effects (SFX), music, speech, and singing — "singing" being listed as a separate category is a refinement relative to most audio-video models (which typically split into only speech/music/sound-effects), directly serving lip-sync for short-drama and music scenarios.
[Classification tool] The all-modality large model Qwen3-Omni is used to uniformly classify audio categories and also to uniformly generate audio captions.
[Category-driven processing differences] (1) During duration padding, clips are "concatenated by category" — i.e., only same-category audio may be concatenated, avoiding cross-category mixing; (2) speech and singing additionally go through Whisper transcription, while sound effects and music are not transcribed; (3) captions use four special tokens — <sfx>, <dialogue>, <singing>, <bgm> — to carry each category respectively, corresponding one-to-one with the four classes.
[Undisclosed] The sample counts and proportions for each of the four categories, the retained proportion of silent samples, and target mixture ratios across categories. It is known that the audio backbone's pretraining data — hundreds of thousands of hours — is "primarily speech data," indicating speech is the overwhelming majority class, with sound effects/music/singing being minority classes. [Uncertain]

### Narrative structure distribution (single-shot vs multi-shot, average clip duration, shot-count distribution, whether native audio tracks are included) ⚠️

[SkyReels-V2] The data-processing granularity is single-shot: all raw video is first segmented into single-shot clips via shot boundary detection before entering the rest of the pipeline, so training samples are predominantly single-shot. Multi-shot/long-narrative capability is achieved on the generation side by the Diffusion Forcing framework through autoregressive continuation (a non-decreasing noise schedule supports infinite extension), rather than by training on multi-shot samples. One of the paper's core selling points is "shot-aware generation," but this is reflected in the caption's shot-type/camera-position/camera-movement fields rather than in a multi-shot sample structure.
[SkyReels-V4] Explicitly supports and evaluates "multi-shot" film-level narrative; the SkyReels-VABench prompt set covers a "complexity gradient from single-shot to multi-shot," and the model provides video extension and shot-transition capabilities. Regarding native audio tracks: V4's joint audio-video training data is required to carry synchronized audio tracks and pass SyncNet validation, but the paper does not give figures for "proportion of samples with native audio tracks," "single-shot vs multi-shot sample ratio," or "average shot count/average clip duration." [Uncertain]

### Language/accent distribution (data basis for multilingual lip-sync capability) ⚠️

[SkyReels-V2] A pure video model, with no language dimension; annotation and prompts are primarily in Chinese and English.
[SkyReels-V4] Language is an explicit object of data construction, but no proportion figures are given: (1) synthetic data is specifically used for "multilingual on-screen text generation," covering Chinese, English, Japanese, Korean, German, French, etc.; (2) synthetic data is also used for "multilingual speech synthesis," i.e., using TTS to fill language-coverage gaps — a direct means of addressing insufficient long-tail-language lip-sync data; (3) the audio backbone is pretrained on hundreds of thousands of hours of primarily speech data, with language coverage extended via TTS; (4) the evaluation benchmark SkyReels-VABench's prompts cover multiple languages, "with particular emphasis on Chinese and English." Per-language sample proportions are not given, nor is there an accent annotation field, nor is there any distributional control along a dialect/accent dimension — accent is not an independent field in the caption schema (unlike LTX-2, which explicitly labels accent). [Uncertain]

## Cleaning Pipeline

### Overall funnel structure (number of filtering stages, order of stages)

[SkyReels-V2 (arXiv:2504.13074) data pipeline] Follows the chain "collection → shot segmentation → tiered quality filtering → post-processing artifact cleanup → dual-axis bucket normalization → structured annotation → concept-balanced sampling":
1. Data collection (open-source datasets Koala-36M/HumanVid + in-house film/TV + artistic repositories, O(100M));
2. Shot boundary detection and segmentation (PyDetect + TransNet-V2) → single-shot clips;
3. Basic quality filtering (low resolution, low frame rate, black/white screen or static frames, camera shake);
4. Video-type filtering (removing surveillance footage, game recordings, animation, meaningless content);
5. Post-processing artifact filtering and cropping (subtitles, station logos, image-editing traces, split screens, black bars; croppable cases undergo subtitle/logo cropping to maximize usable frame area);
6. Multi-model scoring filters (aesthetic filter, OCR filter, mosaic filter, special-effects/sticker filter, plus VQA, IQA, VTTS and other scoring models);
7. Normalization (dual-axis bucketing BT×BAR: duration axis × aspect-ratio axis, FPS standardization);
8. Structured annotation (general MLLM + three sub-expert models → distilled into SkyCaptioner-V1);
9. Concept-balanced sampling (balanced by subject category, halving the data volume);
10. Full-pipeline human-in-the-loop spot checks (0.01% for pretraining, 0.1% for post-training).
[SkyReels-V4 (arXiv:2602.21818) data pipeline] Structured as "collection (real + synthetic) → parallel video-branch / audio-branch processing → audio-video synchronization filtering → three-tier caption generation":
A. Collection: public datasets + licensed proprietary content + synthetic data (multilingual text/speech/editing pairs);
B. Video branch: preprocessing (VLM-enhanced intelligent shot segmentation) → three-dimensional filtering (basic quality / content quality / motion quality) → balancing (conceptual diversity + motion diversity) → VideoCLIP embedding-based deduplication;
C. Audio branch: category classification (Qwen3-Omni four-way classification) → quality filtering (SNR / MOS / clipping ratio / audio bandwidth / VAD silence ratio) → duration control (cut into 15-second chunks, same-category concatenation) → content recognition (Whisper transcription) → audio captioning (Qwen3-Omni);
D. Cross-modal: SyncNet lip-audio sync filtering (|offset|≤3 and confidence>1.5, and average volume ≥ -60dB);
E. Captioning: short caption / long caption / structured caption with special tokens produced in parallel across three tiers.
Overall characteristics: V2 centers on a "shallow discriminator matrix + structured annotation + concept balancing," while V4 elevates VLMs/all-modality LLMs to be the primary drivers of segmentation, classification, and annotation, and adds an audio branch and a cross-modal synchronization filtering chain.

### Quantitative funnel retention rates (input/output volume and final retention rate at each filtering stage, e.g., Apollo 27%) ⚠️

Quantitative funnel disclosure is incomplete; only two usable ratios are available:
(1) SkyCaptioner-V1 training set: curated from "10 million raw samples" down to "approximately 2 million concept-balanced videos," a retention rate of about 20% — this is the only clear "input→output" ratio in the SkyReels series, but it is the selection rate for the annotation model's training set, not the funnel retention rate for the generative model's full training data;
(2) Concept balancing stage: balancing by subject category during post-training "reduced the data volume by 50%," i.e., a retention rate of about 50% for this stage.
Also available as a reference point: the motion-recognition annotator's training data consists of "93,000 high-confidence manually annotated samples + 16,000 motion-axis-balanced synthetic samples," achieving 89% accuracy for single motion-type prediction on a 15,000-video test set.
Neither paper gives a stage-by-stage table of input/output sample counts, nor an end-to-end retention rate from the O(100M) raw data to the final training set, so it cannot be directly compared with a complete funnel such as Apollo's 27%. SkyReels-V4 discloses no filtering retention rates at all. [Uncertain]

### Shot segmentation method (PySceneDetect/in-house model/shot-aware splitting)

[SkyReels-V2] A two-tool combination: all raw video is first passed through shot boundary detection (explicitly naming PyDetect / PySceneDetect and TransNet-V2) and segmented into single-shot clips, serving as the atomic unit for the rest of the pipeline. This is an industry-standard approach (traditional threshold-based methods complemented by the learned TransNet-V2).
[SkyReels-V4] Upgraded to VLM-assisted "intelligent segmentation": the paper explicitly states that traditional scene-cut methods are insufficient, and instead "combines TransNet's shot boundary predictions via VLM to extract semantically complete segments" — i.e., a VLM judges whether a cut point produces semantically complete segments, avoiding fragmenting a single semantic unit or merging two semantic units together. This is the most instructive upgrade in this entry relative to V2, and an instance of the 2026 trend of "pushing large models down into the data-segmentation stage." The TransNet version, specific VLM model, and threshold parameters are not disclosed.

### Quality filtering (aesthetic scoring, sharpness, OCR text filtering, black-bar/watermark/logo detection) ⚠️

[SkyReels-V2] Quality issues are explicitly grouped into three major categories, each with its own filter matrix:
(1) Basic quality issues: low-resolution sources, low frame rate, black/white screen or static frames, camera shake;
(2) Video-type issues: surveillance footage, game recordings, animation, meaningless content (excluded as entire categories);
(3) Post-processing artifacts: subtitles, station logos, image-editing traces, split screens, black borders.
Corresponding filters include: black-screen filter, static-frame filter, aesthetic filter, OCR filter (detecting on-screen text/subtitles), mosaic filter, special-effects and sticker filter; VQA (video quality assessment), IQA (image quality assessment), and VTTS models are also used for scoring. For salvageable samples, a "crop rather than discard" strategy is applied — subtitle and logo regions undergo subtitle/logo cropping to maximize usable frame area, a practical design for improving data utilization.
[SkyReels-V4] Filtering dimensions converge to three: basic quality (duration, resolution, aesthetic score, blur, contrast, exposure), content quality (watermark, logo, text overlay, synthetic artifacts), and motion quality. "Synthetic artifact detection" is used to remove AI-generated or heavily post-processed content, a newly emerging filtering dimension in 2026. Neither generation discloses the source of the aesthetic scoring model (in-house or a LAION-Aesthetics-type model) nor the specific threshold values for each filter. [Uncertain]

### Motion filtering (optical-flow/motion-score thresholds, removal of static and shaky footage) ⚠️

[SkyReels-V2] Motion is a core concern for this series (the paper argues that existing models compromise on "motion dynamics vs. image quality"). On the data side: basic filtering removes static frames and camera-shake samples; on the annotation side, a dedicated "classification-based motion-recognition captioner" was trained — by classifying and labeling motion, it produced "93,000 high-confidence samples + 16,000 motion-axis-balanced synthetic samples" for training this captioner, achieving 89% accuracy for single motion-type prediction on a 15,000-video test set. Motion information subsequently feeds into the camera-motion field of the structured caption, and serves as an optimization target during the reinforcement-learning stage (motion consistency and smoothness).
[SkyReels-V4] Lists "motion quality" as one of the three major filtering dimensions, and introduces "motion diversity" during balancing: key motion patterns are defined for each subject or scene category, and mixture is balanced by pattern, preventing a given subject category from appearing with only a single type of action.
Neither generation discloses the specific optical-flow/motion-score algorithm or threshold values. [Uncertain]

### Deduplication methods (exact deduplication and embedding-based semantic deduplication recorded separately) ⚠️

[SkyReels-V4] Explicitly adopts representation-based semantic deduplication: VideoCLIP embeddings are computed for segmented clips and used for deduplication ("deduplication using VideoCLIP embeddings for segmented clips"). This is embedding-level semantic deduplication; the similarity threshold, clustering algorithm, and deduplication ratio are not disclosed, nor is it stated whether an additional hash-level exact deduplication step exists.
[SkyReels-V2] The paper's data section does not separately describe a deduplication step (nor is a deduplication item extracted in third-party structured analyses), so it cannot be confirmed whether exact or semantic deduplication is present; given that the in-house data includes 280,000 movies and 800,000 TV episodes, the risk of repeated footage from the same IP is relatively high, so some form of deduplication is presumed but there is no official basis. [Uncertain]

### VLM/LLM as data quality judge (multi-modal large model quality scoring and mismatch removal — the 2026 trend of shifting from shallow scorers to large-model semantic judgment) ⚠️

The SkyReels series is a representative example of the trend of "pushing large models down into the data pipeline," and the role each generation plays has clearly evolved:
[SkyReels-V2 (2025)] Large models are mainly responsible for annotation rather than quality inspection: the knowledge of a general MLLM (Qwen2.5-VL-72B-Instruct as the source of general descriptions) plus three sub-expert models (shot captioner, expression captioner, camera-motion captioner) is distilled into SkyCaptioner-V1 (based on Qwen2.5-VL-7B-Instruct). Quality-inspection duties are still handled by shallow/dedicated discriminators: aesthetic filter, OCR filter, mosaic filter, special-effects/sticker filter, VQA/IQA models, etc.
[SkyReels-V4 (2026)] All-modality large models are deeply involved at three key points in the data chain:
(1) Segmentation stage — VLM combined with TransNet shot-boundary prediction to judge segment semantic completeness (using the VLM to decide "where to cut," rather than relying solely on pixel-level scene changes);
(2) Audio classification and captioning stage — Qwen3-Omni uniformly performs four-way audio classification and audio caption generation, replacing traditional audio-event classifiers;
(3) Transcription stage — Whisper performs speech/singing content recognition.
This change confirms the industry trend of "shifting from shallow scorers to large-model semantic judgment." However, the paper does not describe an explicit step where a VLM performs quality/semantic scoring on samples or removes text-image/audio-video mismatches, nor does it give accuracy or compute-cost figures for large-model judgment. [Uncertain]

### Safety and compliance filtering (NSFW, copyright, face/privacy) ⚠️

Neither generation's paper includes a safety and compliance filtering section: there is no description of an NSFW/violence content classifier, no description of face/privacy handling (de-identification, portrait rights), no copyright infringement detection, no CSAM screening process, and no red-teaming or Social Impact section. The only indirectly related item is exclusion at the content-type level (surveillance footage is filtered out as an entire category, objectively reducing privacy-sensitive material) and the procurement framing of "licensed content." As a commercial product targeting the Chinese market, the SkyReels product should have content-safety review and deep-synthesis labeling mechanisms that comply with regulatory requirements, but the technical report does not address this at all, so its data-side implementation cannot be confirmed. [Uncertain]

## Annotation Methods

### Caption model used (in-house VLM/open-source model, model scale) ⚠️

[SkyReels-V2 → SkyCaptioner-V1 (open source, the most valuable data-side output in this series)]
- Base model: Qwen2.5-VL-7B-Instruct (7B; a small model deployed to support annotation throughput at the scale of hundreds of millions of samples);
- Construction method: knowledge-distillation fusion — a general large MLLM (Qwen2.5-VL-72B-Instruct) first produces general descriptions, then three "sub-expert captioners" supplement film/TV-specific dimensions; the two results are fused and then distilled into a unified 7B model;
- Three sub-expert models: ① shot captioner (identifies shot type/camera angle/camera position); ② expression captioner (facial expression and emotion intensity); ③ camera-motion captioner (6-DoF camera-movement recognition, classification-based, trained on 93,000 high-confidence manually annotated samples + 16,000 motion-axis-balanced synthetic samples, achieving 89% accuracy for single-motion-type prediction on a 15,000-sample test set);
- Training configuration: about 2 million concept-balanced videos (curated from 10 million), global batch size 512, 32 A800 GPUs;
- Results: 76.3% average accuracy across film/TV-specific fields, with 93.7% for shot type, 89.8% for camera angle, 83.1% for camera position, and 85.3% for camera motion — significantly outperforming larger models such as Qwen2.5-VL-72B and specialized competitors.
[SkyReels-V4] Does not carry over the public description of SkyCaptioner-V1, instead using a combination of multiple models: on the visual side, captions are generated by an (unnamed) VLM producing short/long/structured tiers of description; on the audio side, captions are generated uniformly by Qwen3-Omni; speech and singing content are transcribed by Whisper. In addition, a frozen MLLM is used as the text encoder during training (not as a captioner). V4 does not disclose the model or scale of the visual captioner. [Uncertain]

### Caption density and structuredness (short/long/dense descriptions, structured fields such as camera motion, style tags)

[SkyReels-V2] Follows a "structured fields + film/TV shot-language" approach, a notable divergence from most vendors' "natural-language long-description" approach, and the methodological core of this series. The structured representation consists of two groups:
- Subject dimension: subject type/category, subject appearance, action, expression, position;
- Shot dimension (provided by sub-experts): shot type (e.g., close-up/medium shot/wide shot), camera angle, camera position, camera motion (6DoF), environment, lighting.
The paper argues that the combination of "general descriptions from the MLLM + fine-grained shot-language from sub-expert models" provides the data foundation for shot-aware (shot-language-controllable) generation; the "subject category" field is also reused as the classification basis for concept balancing — the annotation schema directly serves the data mixture, a closed-loop design well worth emulating.
[SkyReels-V4] Changed to "three tiers of caption produced in parallel":
- Short caption: brief description;
- Long caption: comprehensive description of environment, subjects, lighting, atmosphere;
- Structured caption: follows a standardized description order, with special tokens marking five types of elements — on-screen text, sound effects, dialogue, singing, and background music.
The coexistence of three tiers lets the model respond to both brief and dense prompts; on the inference side, a matching "prompt enhancement" module rewrites free-form user input into a form consistent with the trained structured representation — a classic closed loop where the training caption structure is also the inference interface. The length distribution and mixture ratio of the three caption tiers are not disclosed.

### Joint audio-video caption structure (whether visual and auditory tracks are both covered, whether split into independent fields — e.g., LTX-2's full soundscape description, Script-a-Video's factorized streams, Foley-Omni's three-field scheme)

Only SkyReels-V4 has this, using a "single structured caption + special-token partitioning" scheme that sits between LTX-2's fused long description and Foley-Omni's multi-field approach:
[Tokenized schema] The structured caption uses five paired special tokens to explicitly partition and label content:
- <text></text>: text appearing on screen (in-video text, serving multilingual text-rendering ability);
- <sfx></sfx>: sound effects/foley;
- <dialogue></dialogue>: dialogue content (Whisper transcription results are filled in directly);
- <singing></singing>: singing content (including lyric transcription);
- <bgm></bgm>: background music.
[Design points] (1) Visual and auditory information coexist in the same text sequence, jointly encoded by the shared frozen MLLM text encoder to drive both the video stream and the audio stream simultaneously — this "shared text encoder + token partitioning" is the key to achieving audio-video semantic alignment; (2) the four auditory tokens correspond strictly one-to-one with the four audio classes (sound effects/music/speech/singing), and the classification result can directly determine which token is filled; (3) the visual token <text> is placed alongside the auditory tokens, indicating that on-screen text is treated as "symbolic content requiring precise reproduction" on the same level as the audio track; (4) special tokens allow precise inference-time control of a particular modality component (e.g., specifying only <sfx> while leaving <dialogue> empty to generate a dialogue-free scene).
[Undisclosed] Whether there are finer attribute slots within tokens (e.g., speaker, emotion, timbre), the length ratio between visual and auditory descriptions, and the caption quality-verification method.

### Dialogue transcription and speaker attribute annotation (ASR transcription + speaker identity/language/accent/emotion) ⚠️

[Transcription] SkyReels-V4 uses Whisper to transcribe content for the "speech" and "singing" categories among the four audio classes; the transcribed text is filled into the <dialogue> and <singing> tokens of the structured caption; sound-effects and music classes are not transcribed, and instead receive descriptive captions generated by Qwen3-Omni.
[Speaker attributes] The paper does not mention speaker diarization, speaker identity annotation, emotion annotation, accent annotation, or timbre-description fields — a clear gap compared to LTX-2 (which explicitly labels speaker/language/accent). How line-to-character binding is handled in multi-speaker scenes is not explained.
[Language] Multilingual capability is mainly filled in via synthetic TTS data (Chinese/English/Japanese/Korean/German/French, etc.) rather than by annotating and balancing language on real data. Whisper has language-identification capability, but the paper does not state whether language is stored as an explicit field. [Uncertain]

### Geometric and structured annotation (camera parameters, depth, 3D point tracks, action labels, explicit state) ⚠️

[SkyReels-V2] There is clear structured annotation at the level of "cinematographic geometry," but as classification labels rather than continuous parameters: shot type, camera angle, camera position, and camera motion (classification-based recognition of 6-DoF camera movement, output by the dedicated motion-recognition captioner, backed by 93,000 manually annotated + 16,000 synthetic samples). These fields enter the caption as discrete labels, supporting the controllable camera-movement capability of the SkyReels-V2-Camera-Director variant. No continuous geometric annotations such as camera intrinsic/extrinsic parameter values, depth maps, 3D point tracks, or optical-flow fields are used.
[SkyReels-V4] The focus of structured annotation shifts toward "masks and conditions" rather than geometry: to support the unified inpainting/editing framework, training data must carry conditioning channels such as region masks, reference images, reference video clips, and reference audio (in Stage6, image reference and video reference each account for 20%); a visual segmentation model is used to construct masks. The paper does not mention camera parameters, depth, point tracks, or explicit physical state annotations. [Uncertain]

### Synthetic data construction (controlled perturbation/editing to construct training pairs, e.g., InstructAV2AV)

The SkyReels series makes heavy use of synthetic data across both generations, with distinct purposes — an information-rich item in this entry:
[SkyReels-V2] Synthetic data serves motion quality: (1) On the annotation side — 16,000 "motion-axis-balanced" synthetic samples are generated to fill gaps in motion direction/type that are scarce in real data, used to train the motion-recognition captioner; (2) On the reinforcement-learning side — negative samples are automatically constructed by applying "controlled distortions" to real videos, together with manually annotated preference pairs forming the DPO training data; the distorted samples cover the V2V, I2V, and T2V variant forms, forming a "semi-automatic data production pipeline" — a typical practice of "constructing training pairs via controllable perturbation."
[SkyReels-V4] Synthetic data fills three real-data gaps: (1) multilingual on-screen text — synthesizing images/videos containing Chinese/English/Japanese/Korean/German/French, etc., text to address the shortage of multilingual glyph-rendering data; (2) multilingual speech — TTS-synthesized speech fills language-coverage gaps; (3) inpainting/editing paired data — the paper explicitly states that "paired training data is inherently unavailable in real-world datasets," so an "edited-before/edited-after" paired-sample construction pipeline combining a visual segmentation model, image/video editing models, and controllable generation techniques is used — this is the data prerequisite for V4's unified generation-inpainting-editing capability. The specific segmentation/editing model versions used, the number of samples constructed, and the quality-verification method are not disclosed.

### Degree of human involvement (manual annotation, manual quality inspection, model pre-screening + manual review)

The SkyReels series has a relatively clear human-involvement design, adopting a leveraged strategy of "human-defined criteria + low-ratio spot checks":
[SkyReels-V2]
(1) Tiered spot-check rates (the most valuable quantitative disclosure in this entry): a 0.01% manual spot-check rate during pretraining (1 in every 10,000 samples checked), rising to 0.1% during post-training (1 in every 1,000 checked) — quality-inspection intensity is configured differently by data stage;
(2) Human-in-the-loop validation covers every stage of the pipeline;
(3) Seed annotation for annotation models: the motion-recognition captioner relies on 93,000 high-confidence manually annotated samples; SkyCaptioner-V1's capability evaluation relies on human evaluation (76.3% average accuracy);
(4) Reinforcement-learning preference data: manual annotation produced "30,000 sample pairs (30k sample pairs)" for Bradley-Terry reward modeling, mixed with automatically generated distorted samples;
(5) The taxonomy for concept balancing is manually defined.
[SkyReels-V4]
(1) SFT stage two uses "1 million manually curated high-quality videos" — human involvement is directly reflected in constructing the final fine-tuning set;
(2) On the evaluation side, "50 professional evaluators with backgrounds in film/TV production, audio engineering, and content creation" are organized to conduct both 5-point absolute scoring and GSB pairwise comparison on 2000+ prompts;
(3) The manual spot-check rate and annotation labor hours for the data-cleaning stage are not disclosed.

## Audio-Video Alignment

### Audio-video synchronization detection method (lip sync, event alignment)

Only SkyReels-V4 addresses this, and it is the only cross-modal filtering step in its data pipeline; the method is clearly described:
[Method] SyncNet is used as the lip-audio synchronization discriminator — the paper describes it as "adopting a ConvNet architecture to learn joint embeddings between sound and mouth images," sliding along the time axis to match via embedding distance, and outputting two quantities: the audio-video time offset and a confidence score, used to remove samples with post-dubbing, audio-video misalignment, or mismatched lip sync. The paper notes that this SyncNet process was adapted for distributed processing at the scale of "millions of video samples."
[Placement] Synchronization filtering occurs after video-branch filtering and balancing, and before entry into joint audio-video training, serving as a cross-modal admission gate.
[Limitations] (1) It only covers lip-audio synchronization (speaker scenes); no event-level alignment detection is seen for non-speech events (impact-sound effect, footstep-visual), such as AV-align-type metrics; how temporal alignment is ensured for sound-effect samples is not explained; (2) it is not stated how face-less/speaker-less clips are handled (such samples cannot have a SyncNet score computed); (3) it is not stated whether voiceover, narration, or post-production score (non-diegetic audio) is excluded. Synchronization at the architecture level is handled by the per-layer bidirectional audio-video cross-attention and RoPE temporal-dimension scaling alignment.

### Specific synchronization detection metrics and thresholds (SyncNet/Synchformer/LSE/in-house, threshold values, e.g., MOVA LSE-D≤9.5 and LSE-C≥4.5, SkyReels-V4 SyncNet |offset|≤3∧conf>1.5)

SkyReels-V4 gives explicit threshold values, disclosed more thoroughly than most comparable models:
(1) Synchronization admission condition: |offset| ≤ 3 ∧ confidence > 1.5 — i.e., the absolute value of the audio-video time offset estimated by SyncNet must not exceed 3 frames, and the synchronization confidence must exceed 1.5, with both conditions (a conjunction) required to retain the clip;
(2) Volume threshold: requires a minimum mean volume of no less than -60 dB, used to exclude near-silent clips where SyncNet results would be unreliable;
(3) Silence threshold: VAD (voice activity detection) requires a silence ratio below 0.2.
[Comparative reference] This threshold combination (offset≤3 frames, conf>1.5) is stricter on the confidence dimension than UniTalking's SyncNet conf>0.9, while additionally adding an offset-dimension constraint — an "offset + confidence" dual-condition formulation.
[Undisclosed] The specific SyncNet weight version (original SyncNet or a self-trained version), whether the absolute time tolerance corresponding to a 3-frame offset at 32FPS (about 94ms) was a deliberate design choice, and what proportion of samples this filter removed. SkyReels-V2 has no such step.

### Separate handling of temporal vs. semantic synchronization (time alignment and content-semantic matching treated as two independent filtering conditions) ⚠️

SkyReels-V4 performs a de facto division of labor between the two on the data side, though the paper does not frame them as two parallel filtering conditions:
(1) Temporal synchronization: handled by SyncNet's |offset|≤3 condition, purely judging time-axis alignment;
(2) Semantic/content matching: partially handled by SyncNet's confidence>1.5 condition (low confidence implies the audio-visual content itself does not match, e.g., voiceover or background-music scenes), and separately handled by the four audio-category tokens in the structured caption — the caption writes "what is happening on screen" and "what is being heard" into the same text, so semantic matching is ensured via text conditioning rather than an independent filter;
(3) At the architecture level: per-block bidirectional cross-attention handles continuous alignment, while RoPE temporal-dimension scaling handles unifying the time scale across the two modalities' token rates.
The paper does not set up an independent "audio-visual semantic consistency" discriminator model (e.g., using an all-modality LLM to judge whether the audio track and the visuals depict the same thing), nor does it explain how semantic-matching verification is done for sound-effect/music samples (which lack lip information and for which SyncNet is inapplicable) — a visible gap in this pipeline. [Uncertain]

### Audio quality filtering (SNR, silence detection and silence-ratio threshold, no-audio-track removal, off-screen-source removal, background-music separation) ⚠️

SkyReels-V4's audio quality filtering is the most metric-complete part of this entry, using four objective metrics plus one activity-detection metric:
(1) SNR (signal-to-noise ratio) — removes samples with excessive background noise;
(2) MOS score (Mean Opinion Score, a subjective quality estimate given by an automatic MOS-prediction model) — removes samples with poor perceived quality;
(3) Clipping ratio — removes overload-distorted samples;
(4) Audio bandwidth — identifies and removes pseudo-high-sample-rate audio that has undergone low-bitrate compression with missing high frequencies;
(5) VAD (voice activity detection) — requires a silence ratio below 0.2, i.e., silence must not exceed 20%;
(6) Mean volume threshold of -60dB (used together with synchronization filtering).
Duration is also normalized: long audio is cut into 15-second chunks, short audio is concatenated by category up to 15 seconds.
[Undisclosed] The specific threshold values for each metric (other than the 0.2 silence ratio and -60dB, the thresholds for SNR/MOS/clipping ratio/bandwidth are not given); background-music separation (source separation) or vocal-accompaniment separation is not mentioned; it is not stated whether voiceover/narration (non-diegetic audio) is excluded; sample-rate and channel-count admission requirements are not stated (it is known that audio is encoded at a ratio of 44.1kHz, 5 seconds → 218 latent tokens). SkyReels-V2 has no audio processing. [Uncertain]

### Classification and separate handling strategy for speech/sound effects/music ⚠️

SkyReels-V4 adopts a clear "classify first, then route processing by category" strategy, the backbone of its audio-side methodology:
(1) Classification: Qwen3-Omni classifies audio into four categories — sound effects, music, speech, singing;
(2) Divergent processing: speech and singing → go through Whisper transcription, with transcribed text entering the <dialogue>/<singing> tokens and required to pass SyncNet lip-audio synchronization filtering; sound effects and music → not transcribed, receiving descriptive captions generated by Qwen3-Omni, entering the <sfx>/<bgm> tokens;
(3) Duration concatenation is done by category — only same-category audio may be concatenated to fill up to 15 seconds, avoiding cross-category mixing that would cause semantic contamination;
(4) The four auditory tokens in the caption schema correspond strictly to the four classes, allowing the model to control each category independently during generation.
[Undisclosed] The sample count and training mixture ratio for each of the four categories, whether different quality thresholds are set per category, and whether loss weighting is applied per category. It is known that the audio backbone's pretraining data is "primarily speech," so the four categories are naturally imbalanced. SkyReels-V2 does not involve audio. [Uncertain]

## Training Coordination

### Multi-stage training curriculum and data-curriculum scheduling (basis for stage division: resolution/duration/quality score/modality; low-res→high-res, image→video, short→long)

[SkyReels-V2] The curriculum advances along a dual axis of increasing resolution and increasing filtering strictness, a clear "low-res→high-res, lenient→strict" paradigm:
- Pretraining stage 1: 256p low resolution, jointly trained with concept-balanced image data;
- Pretraining stage 2: raised to 360p;
- Pretraining stage 3: raised to 540p; the paper states that "data filtering tightens in step with increasing resolution, from 256p to 720p" — i.e., higher-resolution stages are fed only cleaner data;
- Four post-training stages: ① concept-balanced SFT (540p) → ② reinforcement learning targeting motion quality (DPO, human-annotated + synthetic distorted data) → ③ Diffusion Forcing training (supporting variable-length/infinite-length generation) → ④ high-quality, high-resolution SFT (720p).
[SkyReels-V4] A six-stage progressive curriculum + audio pretraining + joint training + two-stage SFT (Table 1):
- Stage1 text-to-image: 256px, 3 billion images, 3 epochs (establishing visual priors);
- Stage2 text-to-image + text-to-video: 256px/16fps, 1 billion images + 400 million videos, 3 epochs (introducing temporal dynamics);
- Stage3 inpainting tasks: 256px/16fps/2–15 seconds, same data volume, 2 epochs, with each inpainting sub-task accounting for 5% (introducing mask conditioning);
- Stage4 mixed resolution: 256/480px, 100 million each, 2 epochs;
- Stage5 high resolution: 480/720/1080px, 50 million each, 2 epochs;
- Stage6 multi-modal conditioning: 480/720/1080px, 20 million images + 50 million videos, 2 epochs, with image reference and video reference each accounting for 20%;
- Audio backbone pretraining: hundreds of thousands of hours (primarily speech), up to 15 seconds, 3 epochs;
- Joint audio-video training: 720/1080px, 50% video data + text-to-audio data, 2 epochs;
- SFT stage 1: 720/1080px, 5 million videos (20% with multi-modal conditions), 3 epochs; SFT stage 2: 1 million manually curated high-quality videos, 3 epochs.
The curriculum logic is clear: image → video → task conditioning → resolution climb → multi-modal conditioning → cross-modal joint training → fine-tuning, with modality and task capabilities layered progressively rather than mixed in a single pass.

### Data mixture changes across training stages (pretraining/annealing/SFT high-quality subset)

Both generations show a pattern of "stage progression = decreasing data volume + increasing data quality":
[SkyReels-V2] (1) The pretraining stage has the largest data volume (O(100M)) and the loosest filtering, tightening progressively as resolution increases; (2) upon entering post-training, concept balancing by subject category is executed, directly halving the data volume (-50%); (3) the final SFT stage uses the highest-quality subset and raises resolution to 720p; (4) concept-balanced image data (O(100M) scale) is mixed in throughout as conceptual supplementation.
[SkyReels-V4] Table 1 shows a clear order-of-magnitude decay: 3 billion images → 1 billion images + 400 million videos → 100 million each → 50 million each → 50 million videos → 5 million SFT → 1 million manually curated — spanning three orders of magnitude, a textbook "pyramid-shaped data curriculum." Mixture details: in Stage3 each inpainting sub-task accounts for 5%; in Stage6, image reference and video reference conditions each account for 20%; in the joint audio-video training stage, "50% video data + text-to-audio (T2A) data" is mixed — i.e., half pure video data is retained during joint training to prevent visual capability degradation, a common practice for preventing cross-modal training from harming single-modality quality, and the most practically useful mixture clue in this entry; in SFT stage 1, 20% of samples carry multi-modal conditions.
Not disclosed is how the mixture of domains/categories changes across stages.

### Post-training data (SFT curated-set scale and selection criteria, number of preference pairs and annotation method, reward-model training data) ⚠️

[SkyReels-V2] Post-training data is disclosed relatively thoroughly:
(1) SFT: first concept-balanced SFT (540p), then high-quality SFT (720p), using the highest-quality subset; the specific sample count is not published;
(2) Preference data (RL/DPO): a "semi-automatic data production pipeline" is used — manual annotation produces "30,000 sample pairs (30k sample pairs)," while the automatic side generates corrupted samples by applying controlled distortion to real videos, covering the V2V, I2V, and T2V variant forms;
(3) Reward model: a Bradley-Terry model is trained on the 30,000 sample pairs;
(4) DPO training proceeds in stages — "each stage requires 20,000 training samples, across a total of 3 stages" (i.e., about 60,000 preference samples used across three rounds of DPO);
(5) The RL objective focuses on motion quality (dynamic consistency and smoothness) rather than general aesthetic preference.
[SkyReels-V4] (1) Two SFT stages: 5 million videos with multi-modal conditions (3 epochs) → 1 million manually curated high-quality videos (3 epochs); (2) the paper's main text does not describe an RLHF/DPO method or preference data, but official release materials mention that V4 has a "full-modality reinforcement learning system" — a discrepancy between the two accounts — so the scale and annotation method of preference data cannot be confirmed. [Uncertain]

### Data-processing infrastructure and throughput (NeMo Curator/Data-Juicer/in-house, GPU acceleration ratio, processing scale, cost) ⚠️

Neither paper discloses details of the data-processing infrastructure: no mention of open-source frameworks such as NeMo Curator or Data-Juicer, or an in-house data platform; no figures for GPU acceleration ratio, per-sample processing cost, or throughput (clips/hour). The only available indirect clues are: (1) SkyReels-V4 mentions SyncNet being adapted for processing "millions of video samples," implying the existence of distributed batch-processing capability; (2) the V4 paper's author grouping includes a dedicated "data processing" team, indicating data engineering is treated as a first-class function; (3) SkyCaptioner-V1's choice of a 7B small model rather than directly using a 72B model for annotation is itself a cost-engineering decision made to support annotation throughput at the scale of hundreds of millions of samples (its training used 32 A800 GPUs, global batch size 512); (4) SkyReels-V2's training infrastructure and cluster scale are also not disclosed. [Uncertain]

## Performance Comparison

### Quantitative impact of data-strategy ablations (distinguishing: filtering-strictness ablation / caption-density-style ablation / data-mixture ablation, and corresponding evaluation metrics) ⚠️

Quantitative data-strategy ablations are scarce; both papers focus mainly on overall system comparisons:
[SkyReels-V2] (1) Caption quality ablation (indirect): the field-accuracy comparison between SkyCaptioner-V1 and baseline captioners constitutes quantitative evidence of annotation quality — an average of 76.3%, with 93.7% for shot type, 89.8% for camera angle, 83.1% for camera position, and 85.3% for camera motion, surpassing Qwen2.5-VL-72B, a model with 10x the parameters, indicating that the "small model + sub-expert distillation" annotation route outperforms general large models on specialized fields — the most persuasive piece of data-side evidence in this series; (2) the motion-recognition captioner achieves 89% accuracy for single motion-type prediction on a 15,000-sample test set; (3) the effect of concept balancing is only stated qualitatively (reducing head-category bias, improving generalization), with no on/off comparison scores given; (4) training-stage ablation: the paper narratively attributes gains to each of the four post-training stages in sequence (concept-balanced SFT improves baseline quality, motion RL fixes dynamic artifacts, Diffusion Forcing enables long video, high-resolution SFT improves fidelity), but does not provide a stage-by-stage quantitative gain table; (5) overall performance: VBench overall score 83.9%, quality score 84.7%, outperforming HunyuanVideo-13B and Wan2.1-14B.
[SkyReels-V4] The paper does not report any ablation experiments targeting the data pipeline (no on/off comparisons for filtering strictness, caption density, or data mixture). Evidence of effect is end-to-end: first place overall average human-evaluation score on SkyReels-VABench, and first place on Artificial Analysis Video Arena's "text-to-video (with audio)." [Uncertain]

### Evidence for quality vs. quantity (cases where small, refined data outperforms large, messy data) ⚠️

The SkyReels series provides two relatively clear pieces of "better to have less but better" evidence:
(1) Halving via concept balancing: SkyReels-V2 performs concept balancing by subject category during post-training, explicitly stating it "reduced the data volume by 50%," and the team still chose to execute it, reasoning that it eliminates head-category bias and improves generalization — trading half the data for distributional balance, a typical instance of prioritizing quality/distribution over quantity;
(2) 20% retention for the annotation set: SkyCaptioner-V1's training set was curated from 10 million down to about 2 million (roughly 20% retained) to ensure concept balance and annotation quality; and the fact that the 7B small model outperforms the 72B general model on specialized fields indicates that data quality (sub-expert annotation) matters more than model scale;
(3) SkyReels-V4's pyramid curriculum: converges progressively from 3 billion images down to a final 1 million manually curated videos for the final SFT — the final image quality is determined by these 1 million clips, likewise reflecting the idea of "using large scale to build priors, small scale to set the ceiling."
However, neither paper conducts a controlled "small-and-refined vs. large-and-messy" experiment; the above is all strategic evidence rather than ablation evidence. [Uncertain]

### Alignment between training-data domain distribution and evaluation-benchmark taxonomy (e.g., VABench's seven major categories)

[SkyReels-V2] Built its own SkyReels-Bench human evaluation benchmark (covering instruction following, motion quality, consistency, visual quality, and other dimensions), and was evaluated on the public VBench (overall score 83.9%, quality score 84.7%). The film/TV fields in its annotation schema — shot type, camera angle, camera motion, etc. — correspond directly to the evaluation concerns of "shot-aware generation," but the paper does not explicitly map the training data's domain distribution to the benchmark taxonomy.
[SkyReels-V4] Built its own SkyReels-VABench, with a relatively clear alignment to training data:
(1) Content categories: advertising, social media content, narrative storytelling, educational content, and entertainment — five major categories, with 2000+ carefully constructed prompts — these five categories correspond closely to its training data's "licensed movies, TV series, short videos, web series" and Kunlun Tech's short-drama business scenario, an instance of "designing evaluation categories around business scenarios";
(2) Five evaluation dimensions: instruction following (split into video and audio sub-items), audio-video synchronization, visual quality, motion quality, and audio quality — where the "audio-video synchronization" dimension corresponds one-to-one with the data-side SyncNet filtering, and the "motion quality" dimension corresponds to the data-side motion-quality filtering and motion-diversity balancing, a clear example of alignment between training-data processing and the evaluation system;
(3) Language: prompts cover multiple languages, with particular emphasis on Chinese and English, aligned with the construction goals of synthetic multilingual text/speech data;
(4) Complexity gradient: from single-shot to multi-shot, corresponding to V4's multi-shot narrative capability;
(5) Evaluation method: 50 professional evaluators with film/TV/audio/content-creation backgrounds, using both 5-point absolute scoring and GSB pairwise comparison.
The paper does not give a quantitative correspondence between the proportions of each training-data domain and the benchmark's five major categories.

## Uncertain Fields

The research information for the following fields is partially uncertain (sources marked ⚠️):

- openness
- provenance_licensing
- duration_distribution
- resolution_aspect_distribution
- domain_distribution
- audio_category_distribution
- narrative_structure_distribution
- language_accent_distribution
- funnel_retention_rate
- quality_filtering
- motion_filtering
- deduplication
- model_as_data_judge
- safety_filtering
- caption_model
- dialogue_transcription_attributes
- geometric_structured_annotation
- temporal_vs_semantic_sync
- audio_quality_filtering
- audio_type_handling
- post_training_data
- data_infra_throughput
- data_ablation
- quality_over_quantity_evidence
