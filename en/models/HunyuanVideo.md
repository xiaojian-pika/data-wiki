# HunyuanVideo (Tencent Hunyuan Video, original 2024 13B) + HunyuanVideo 1.5 (Tencent Hunyuan Video 1.5, 8.3B)

> Topic: Data processing for video generation models (including joint audio-video generation): data cleaning pipeline, data distribution, annotation methods

[← Back to home](../index.md)

**Table of contents**: [Basic Information](#basic-information) · [Data Scale and Distribution](#data-scale-and-distribution) · [Cleaning Pipeline](#cleaning-pipeline) · [Annotation Methods](#annotation-methods) · [Audio-Video Alignment](#audio-video-alignment) · [Training Coordination](#training-coordination) · [Performance Comparison](#performance-comparison)

## Basic Information

### Name

HunyuanVideo (Tencent Hunyuan Video, original 2024 13B) + HunyuanVideo 1.5 (Tencent Hunyuan Video 1.5, 8.3B)

### Publishing Organization/Company

Tencent Hunyuan Foundation Model Team

### Release Date (technical report/paper/open-source date)

HunyuanVideo: technical report released December 3, 2024 (arXiv:2412.03603, later revised as v2), with weights and inference code open-sourced simultaneously; the HunyuanVideo-I2V image-to-video version was released in March 2025. HunyuanVideo 1.5: open-sourced on November 21, 2025, technical report arXiv:2511.18870 (November 2025).

### Type (model/dataset/toolchain/benchmark)

A model (an open-source foundation video-generation model), accompanied by complete open-source code and an inference framework. The original 13B version was, at the time, the largest-parameter open-source video-generation model; 1.5 is a lightweight 8.3B version, aimed at running on consumer-grade GPUs (about 14GB VRAM). Both are pure video-generation models, not a dataset, and not an evaluation benchmark.

### Degree of Openness (whether weights/code/data/pipeline are each open-sourced)

A typical pattern of "open weights + code, closed data and data pipeline."
【Weights】Open source. HunyuanVideo 13B (DiT backbone + 3D VAE + text encoders) is released on Hugging Face (tencent/HunyuanVideo, tencent/HunyuanVideo-I2V); HunyuanVideo 1.5 (8.3B, including T2V/I2V/super-resolution modules) is released on GitHub at Tencent-Hunyuan/HunyuanVideo-1.5 and on Hugging Face.
【Code】Open source. Inference code, parallel inference, quantization, LoRA, and ComfyUI/Diffusers integration are all provided; training code is not fully open.
【License】Tencent Hunyuan Community License, not a standard OSI open-source license, with usage restrictions for regions such as the EU and for users above a certain monthly-active-user count.
【Data】Not open source. Neither the training dataset itself, nor the manifest after each level of filtering, nor the caption data is disclosed.
【Pipeline】Disclosed in fair methodological detail (in particular, the original version's description of the hierarchical filtering funnel, the structured caption schema, and the camera-motion classifier is among the most detailed of any contemporaneous closed-source model), but the filter code and model weights (such as the in-house VideoCLIP, the blur-detection model, the YOLOX-like detector, and the captioning VLM) are not open-sourced, so it cannot be directly reproduced.

### Whether Simultaneous Audio-Video Generation Is Supported, and How (native joint/cascaded/MoE fusion)

Not supported. Both HunyuanVideo and HunyuanVideo 1.5 are pure visual video-generation models, whose output has no audio track. The full text of the 1.5 technical report contains no content related to audio generation whatsoever, and the data side likewise involves no audio-track processing.
Tencent's audio capability is handled by independent models rather than joint generation: HunyuanVideo-Foley (August 2025, video-to-audio/Foley generation, based on a roughly 100,000-hour TV2A dataset), HunyuanVideo-Avatar (audio-driven digital humans), etc., belonging to the "cascaded/bolt-on" form — HunyuanVideo generates the picture first, and then a Foley model adds the sound, rather than native joint denoising. Therefore, this entry does not constitute a reference sample on the joint audio-video generation dimension, and all audio-related fields in this survey (audio_category_distribution, av_sync_detection, sync_metric_and_threshold, temporal_vs_semantic_sync, audio_quality_filtering, audio_type_handling, joint_av_caption_schema, dialogue_transcription_attributes) do not apply.

### List of Research Sources (URLs for papers/technical reports/official documentation/news, with the nature of each source labeled: official primary source/same-team corroboration/third-party reporting)

- 【Official primary source】HunyuanVideo: A Systematic Framework For Large Video Generative Models, Tencent Hunyuan, arXiv:2412.03603 (includes Section 3 data preprocessing/hierarchical filtering funnel, structured captions, a 14-class camera-motion classifier, ~1M manually annotated SFT samples): https://arxiv.org/abs/2412.03603
- 【Official primary source】HunyuanVideo paper HTML v1 full text (data section searchable): https://arxiv.org/html/2412.03603v1
- 【Official primary source】HunyuanVideo paper HTML v2 full text: https://arxiv.org/html/2412.03603v2
- 【Official primary source】HunyuanVideo 1.5 Technical Report, Tencent Hunyuan, arXiv:2511.18870, 2025-11 (includes >10M hours of raw video, 800M clips, Table 2's eight-stage training data table, three captioning models, OPA-DPO, CT/SFT/RLHF): https://arxiv.org/abs/2511.18870
- 【Official primary source】HunyuanVideo 1.5 technical report HTML full text: https://arxiv.org/html/2511.18870v1
- 【Official primary source】HunyuanVideo 1.5 technical report PDF: https://arxiv.org/pdf/2511.18870
- 【Official primary source】HunyuanVideo GitHub repository (weights, inference code, license): https://github.com/Tencent-Hunyuan/HunyuanVideo
- 【Official primary source】HunyuanVideo Hugging Face model card: https://huggingface.co/tencent/HunyuanVideo
- 【Official primary source】HunyuanVideo-I2V Hugging Face model card (2025-03 image-to-video version): https://huggingface.co/tencent/HunyuanVideo-I2V
- 【Same-team corroboration】HunyuanVideo-Foley (Tencent's video-to-audio/Foley generation model, roughly 100,000-hour TV2A dataset, demonstrating that Tencent's audio capability is an independent cascaded model rather than jointly generated with video): https://github.com/Tencent-Hunyuan/HunyuanVideo-Foley
- 【Third party】alphaXiv HunyuanVideo 1.5 paper analysis page: https://www.alphaxiv.org/overview/2511.18870v1
- 【Third party】Emergent Mind: overview of the HunyuanVideo 1.5 open-source video-synthesis model: https://www.emergentmind.com/topics/hunyuanvideo-1-5
- 【Third party】Hugging Face Papers: HunyuanVideo paper page and community discussion: https://huggingface.co/papers/2412.03603
- 【Third party】ResearchGate: HunyuanVideo 1.5 Technical Report entry: https://www.researchgate.net/publication/397934115_HunyuanVideo_15_Technical_Report

## Data Scale and Distribution

### Training Data Scale (number of video clips/hours/tokens, pretraining vs. SFT split)

【HunyuanVideo original (2024)】The disclosure of the overall scale accounting is incomplete: the paper explicitly gives only the SFT stage's approximately 1 million (~1M) manually curated samples, plus an image-side "billions" scale used for the first-stage T2I pretraining and a "hundreds of millions" scale used for the second stage. The size of the raw video-side pool and the absolute number of clips/hours at each resolution tier (256p/360p/540p/720p) are not published — only the relative retention ratio is given (each stage retains 1/2 to 1/5 of the previous stage).
【HunyuanVideo 1.5 (2025)】The accounting is markedly more complete, and is this entry's most valuable quantitative disclosure:
- Raw video pool: more than 10 million hours (>10M hours) of raw video;
- After segmentation and filtering: about 800 million (800M) high-quality video clips enter pretraining;
- Subsequent stages shrink progressively: 200 million (200M) at the 480p stage, 100 million (100M) at the 720p/16fps stage, 100 million (100M) at the 720p/24fps stage;
- CT (continued training) stage: 1 million (1M) high-quality clips each for T2V and I2V;
- Image side: from an image pool of more than 10 billion (>10B), 5 billion (5B) are screened for the first-stage 256p T2I pretraining, and 1 billion (1B) for the second stage at 512p;
- Super-resolution module training data: 1 million high-quality video clips (1K-4K resolution) + high-resolution images.
The 1.5 report gives no exact sample counts for the SFT stage or the RLHF stage (only the screening criteria are described).

### Data Source Composition (proprietary/public datasets/web crawling/licensed procurement/synthetic data) ⚠️

Both generations only give a qualitative description, with no proportion breakdown of sources given. The original version states the raw data pool covers multiple domains such as people, animals, plants, landscapes, vehicles, objects, buildings, and animation, without specifying the sources (implying a mix of web crawling + proprietary/licensed content libraries). The 1.5 report states videos come "from a variety of channels, ensuring comprehensive coverage in content, filming techniques, camera movement, style, and scenes," likewise without distinguishing between proprietary/public datasets/crawled/procured/synthetic. Neither generation uses synthetic video data as primary training corpus (not mentioned). On the image side, 1.5 explicitly reuses the data acquisition and processing pipeline of HunyuanImage-3.0. [Uncertain]

### Data Compliance and Provenance (proportion of licensed data, rights-cleared datasets, C2PA, etc.) ⚠️

Not disclosed. Neither technical report addresses the proportion of licensed data, a list of rights-cleared datasets, a copyright-handling strategy, or provenance standards such as C2PA. The original version only mentions, in the filtering stage, using a YOLOX-like vision model to remove "watermarks, borders, logos, and certain sensitive information," which is closer to image cleanliness than to copyright compliance. The compliance constraints on the open-source side are reflected in the model license (the Tencent Hunyuan Community License restricting use in regions such as the EU, and restricting commercial use for services with more than 100 million monthly active users), not in training-data provenance. [Uncertain]

### Clip Duration Distribution and Segmentation Strategy

【Original version】No duration histogram is given. The segmentation strategy is explicit: PySceneDetect is used to cut raw video into single-shot clips, after which an OpenCV Laplacian operator selects a clear frame within the clip as the starting frame. Training duration is expressed in frame count — from 65 frames (256×256×65) to 129 frames (720×1280×129), supporting multi-frame-count bucketing from 1-129 frames.
【1.5】Much more explicit: all training clips are uniformly segmented to 2-10 seconds; pretraining stages III-V run at 16fps, 2-10s, and from stage VI onward the frame rate rises to 24fps, 2-10s; the CT/SFT stages maintain 24fps, 2-10s. That is, 1.5 follows a route of "predominantly short clips, with curriculum progression driven by frame rate rather than duration" — no clip longer than 10 seconds is trained on.

### Resolution/Aspect-Ratio Distribution and Bucketing Strategy

【Original version】Resolution itself is the layering axis of the hierarchical filtering funnel: four progressively stricter tiers of datasets are constructed — 256p → 360p → 540p → 720p — plus a multi-scale SFT dataset tier, with each higher tier filtered more strictly. The training side uses a bucketing strategy to simultaneously support multiple resolutions and aspect ratios (also supporting a variable 1-129 frame count), enabling generation of video at arbitrary aspect ratios. The specific proportion of each aspect ratio is not published.
【1.5】Staging is clearer: images go 256p → 512p; video goes 256p → 480p → 720p, with the CT/SFT stages training both 480p and 720p simultaneously; there is also an independent super-resolution module trained with 1K-4K data, lifting the output of the 480p/720p base model to the 1080p tier. Details of aspect-ratio bucket proportions are not disclosed.

### Category/Domain Distribution and Mixture Strategy (proportional control and concept balancing across people, actions, scenes, styles, etc.) ⚠️

This is the most methodologically valuable aspect of the original HunyuanVideo, and one of few open-source works of its period to explicitly perform "concept balancing":
【Original version】(1) The raw data pool is divided by domain, explicitly listing categories such as people, animals, plants, landscapes, vehicles, objects, buildings, and animation, with coverage breadth stated as a data-construction goal; (2) an in-house VideoCLIP model is used to extract embeddings from clips, first performing semantic deduplication by cosine distance, then running k-means on the embeddings to obtain about 10,000 (~10K) concept centroids, and performing concept resampling and balancing based on these centroids — i.e., using cluster centers as concept proxies, downsampling over-dense concepts and upsampling sparse ones to suppress long-tail imbalance. This "VideoCLIP embedding + 10K cluster centroids + resampling" combination is a signature design of the original version's data processing. (3) The final proportion figures for each domain are not published.
【1.5】The report does not repeat a description of the concept-balancing mechanism, only emphasizing channel-diversity coverage of "content, filming techniques, camera movement, style, and scenes"; whether the 10K concept-centroid resampling is continued is not stated. There are traces of categorization in 1.5's RLHF stage: I2V RLHF prompts cover "100+ categories," and T2V DPO uses a prompt set balanced across the three dimensions of motion/scene/subject. [Uncertain]

### Audio Category Distribution and Mixture (proportions and control strategy for speech/Foley/music/ambient sound/silence) — a dimension unique to AV models ⚠️

Not applicable. Neither HunyuanVideo nor HunyuanVideo 1.5 generates audio; the training data involves no audio-track processing, and neither report contains any content on audio-category mixture. For Tencent's in-house audio-data methodology, one should look instead to independent works such as HunyuanVideo-Foley (video to audio, a roughly 100,000-hour TV2A dataset, with a multi-label balancing strategy for speech/sound-effects/music). [Uncertain]

### Narrative Structure Distribution (single-shot vs. multi-shot, average clip duration, shot-count distribution, whether native audio track is present)

Both generations explicitly adopt a "single-shot" data paradigm and do not train on multi-shot narrative:
【Original version】PySceneDetect is used to segment into single-shot clips; Transnet v2 + PySceneDetect are also used together to provide scene-boundary information to improve segmentation reliability; the dense description field records scene transitions and camera movement.
【1.5】Goes further: after segmentation via PySceneDetect and an in-house operator, an additional dedicated "transition classifier" is applied to entirely discard clips still retaining transition effects (fades, dissolves, etc.), ensuring every training clip is a clean single shot. The average clip duration is simply the 2-10 second range; shot-count distribution is meaningless (always 1). Native audio track: none (the audio track is not retained on the data side).

### Language/Accent Distribution (the data basis for multilingual lip-sync capability) ⚠️

Not applicable to lip sync (no audio generation). On the text side: the original version uses an MLLM as a text encoder and considers multiple languages during caption generation and prompt rewriting — a Hunyuan-Large large language model is used to rewrite user prompts (prompt rewrite), with functions including "standardization of prompt structure, simplification of complex terminology, and multilingual adaptation" to align with the distribution of the training captions; it actually supports both Chinese and English prompts. The proportion of language composition in the training captions is not published. The 1.5 report does not describe prompt rewriting or language distribution. [Uncertain]

## Cleaning Pipeline

### Overall Structure of the Cleaning Funnel (number of filtering stages, order)

【HunyuanVideo original (2024) — hierarchical data filtering pipeline】This is the core reference value of this entry. The structure is "one filter chain + four progressively stricter threshold gates + one round of manual curation":
Step 0, preprocessing: PySceneDetect segments raw video into single-shot clips; an OpenCV Laplacian operator selects a clear frame as the clip's starting frame;
Step 1, deduplication and balancing: the in-house VideoCLIP extracts embeddings, semantic deduplication is performed by cosine distance, and k-means on the embeddings yields ~10K concept centroids for concept resampling and balancing;
Step 2, filter chain (multiple parallel scorers):
  · a Dover model scores from both an "aesthetic" and a "technical quality" perspective;
  · an in-house blur-detection model removes unclear content;
  · optical-flow estimation estimates motion speed, removing static and slow-motion clips;
  · PySceneDetect + Transnet v2 provide scene-boundary information;
  · an OCR model removes clips with excessive text and crops out subtitle regions;
  · a YOLOX-like vision model detects and removes/crops watermarks, borders, logos, and sensitive information;
Step 3, hierarchical dataset construction: the same set of filters, at different strictness thresholds, is used to construct 256p / 360p / 540p / 720p tiered training sets, each progressively stricter;
Step 4, the SFT tier: manual annotation and curation is layered on top of the strictest tier, yielding an SFT dataset of about 1 million samples.
The image data goes through a parallel hierarchical pipeline, reusing most of the filters except the motion-related ones, producing two image sets (a billions-scale one and a hundreds-of-millions-scale one).

【HunyuanVideo 1.5 (2025) — three-stage filtering + spatial cropping】The structure is reorganized into a more engineering-oriented three stages:
Step 0, basic deduplication and removal of corrupted files;
Step 1, segmentation: PySceneDetect + an in-house operator segment into 2-10 second clips → a transition classifier removes clips containing transition effects;
Step 2, spatial cropping: regions containing subtitles, logos, and watermarks are cropped rather than the whole clip being discarded, but if the retained area after cropping is <60% of the original frame, the whole clip is discarded (to preserve compositional integrity) — this is an important improvement relative to the original version: shifting from "discard on sight of a watermark" to "crop where possible, discard only where cropping is not possible," raising data utilization;
Step 3, three-tier quality filtering:
  (a) basic filtering: removes clips with padding black bars, stitching artifacts, grid layouts/collages, and static or low-motion video;
  (b) visual quality filtering: a comprehensive quality-assessment operator scoring across four dimensions — sharpness, detail retention, noise & artifacts, dynamic range;
  (c) aesthetic filtering: an aesthetic-scoring operator removes low-scoring clips.
The output is about 800 million high-quality clips entering pretraining, which subsequent stages then shrink progressively to 200 million/100 million/100 million/1 million against increasingly higher standards.

### Quantitative Funnel Retention Rates (input/output volume at each filtering stage and final retention rate, e.g. Apollo 27%)

【Original version】Only relative accounting is given, no absolute numbers: the paper states each filtering stage removes "a large portion of data... ranging from half to one-fifth of the data from the previous stage" — i.e., each stage retains only 1/2 to 1/5 of the previous stage (translating to a retention rate of 20%-50%, elimination of 50%-80%). If the four stages are chained together using this range, the overall retention rate from 256p to SFT would fall roughly in the wide range of 0.16%-6.25%; the paper gives no final value.
【1.5】Absolute numbers are available for computation, a rare quantitative funnel among comparable works:
· Raw pool >10 million hours of video → after segmentation + filtering, 800 million clips (at an average of about 6 seconds each, roughly 1.33 million hours, giving a rough estimated overall retention rate of about 13%; this conversion is this survey's own estimate, not the paper's original figure);
· 800 million (256p) → 200 million (480p): 25% retained;
· 200 million (480p) → 100 million (720p/16fps): 50% retained;
· 100 million → 100 million (720p 16fps→24fps): 100% retained (same scale, just a frame-rate change);
· 100 million → 1 million (CT/high-quality tier): 1% retained;
· Image side: >10 billion → 5 billion (retention <50%) → 1 billion (second-stage retention 20%).
The end-to-end retention rate from 800 million to 1 million is 0.125%, one of the most quantitatively complete funnels in this survey, and directly comparable to accounting such as Apollo's 27% (noting a difference in accounting basis: Apollo's is a single-stage filter retention rate, whereas this is a multi-stage chained final value).

### Shot Segmentation Method (PySceneDetect/in-house model/shot-aware splitting)

【Original version】PySceneDetect performs the main segmentation (segmenting into single-shot clips); Transnet v2 and PySceneDetect together provide scene-boundary information for cross-validation; after segmentation, an OpenCV Laplacian operator picks a clear frame within the clip as the clip's starting frame, avoiding starting with a blurry transition frame.
【1.5】PySceneDetect + a custom operator jointly detect scene boundaries, uniformly segmenting into 2-10 second clips; a key addition is a dedicated, purpose-trained "transition classifier" applied after segmentation as a second-pass cleaning step, removing clips that still contain transition effects such as fades and dissolves — indicating the team judged that pure threshold-based shot detection misses soft transitions, requiring a model-level follow-up fix.

### Quality Filtering (aesthetic scoring, clarity, OCR text filtering, black-bar/watermark/logo detection)

【Original version】Multiple parallel paths, progressively stricter:
· Aesthetics and technical quality: a Dover model scores from two views (aesthetic view + technical view);
· Clarity: an in-house, training-based blur-detection model removes visually blurry content; the Laplacian operator picks clear starting frames;
· Text: an OCR model removes clips with excessive text overlay, and crops (rather than fully discards) clips containing subtitles;
· Watermarks/black bars/logos/sensitive information: a YOLOX-like detection model locates and removes or crops them out;
· The specific threshold values for each filter are not published by the paper — it only states that "different strictness thresholds are used for different training tiers."
【1.5】Restructured into "one comprehensive quality-assessment operator + one aesthetic operator + a set of basic rules":
· The comprehensive quality operator has four dimensions: sharpness, detail retention, noise & artifacts, dynamic range — more fine-grained than the original version's single blur-detection model, notably adding two dimensions leaning toward "production quality," namely noise/artifacts and dynamic range;
· Aesthetic operator: removes low-aesthetic-score clips;
· Basic rules: padding black bars, stitching artifacts, and grid layouts/collages are directly removed;
· Subtitles/logos/watermarks: changed to spatial cropping, with the clip discarded only if the retained area after cropping is <60%.
Neither generation publishes any threshold values.

### Motion Filtering (optical-flow/motion-score thresholds, removal of static and shaky footage) ⚠️

【Original version】Optical-flow estimation is used to compute motion magnitude, removing static and slow-motion video; in the SFT manual-annotation stage, "motion speed," "action integrity," and "motion blur" are also specifically judged by hand — i.e., motion quality has both an automatic and a manual gate.
【1.5】In the basic-filtering tier, "static or low-motion scenes" are removed, but it is not stated whether optical flow is still used; one of the SFT-stage screening criteria is "motion smoothness." Neither generation gives threshold values. [Uncertain]

### Deduplication Method (exact deduplication and embedding-based semantic deduplication recorded separately) ⚠️

【Original version】Semantic-level deduplication is performed, with a clear method: an in-house VideoCLIP model extracts video embeddings, and duplication is determined and removed by cosine distance; the same set of embeddings is then run through k-means to obtain about 10,000 concept centroids used for concept resampling and balancing. Hash-level exact deduplication is not mentioned.
【1.5】Only described as "basic deduplication and the removal of corrupted files" as the frontmost step, without stating whether it is hash-based deduplication or embedding-based semantic deduplication, and without stating whether the VideoCLIP approach is continued. 1.5's description of deduplication is markedly weaker than the original version's. [Uncertain]

### VLM/LLM as Data-Quality Judge (large multimodal model scoring and mismatch removal, the 2026 trend from shallow scorers toward large-model semantic judgment) ⚠️

Both generations make extensive use of models as data-quality judges, but in different forms, and neither uses "a general large VLM for end-to-end semantic judgment":
【Original version】Uses a panel of specialized small/discriminative models — Dover (dual aesthetic + technical view), an in-house blur-detection model, an OCR model, a YOLOX-like detection model, an optical-flow model, Transnet v2 scene detection, in-house VideoCLIP (deduplication and concept balancing), and an in-house camera-motion classifier (14 classes). This is a typical "shallow multi-scorer" paradigm characteristic of 2024. True large models are only used in the annotation stage (an in-house VLM generating structured captions) and in prompt rewriting (Hunyuan-Large).
【1.5】Evolves toward "model-based operators" but still discloses no use of a general large VLM for scoring: it adds a purpose-trained "transition classifier," a four-dimensional "comprehensive visual quality assessment operator," an aesthetic-scoring operator, and a camera-motion recognition model (clip-level + temporal-level). The clearest evidence of large-model involvement in data construction is on the captioning side: the captioning model itself is post-trained with reinforcement learning via OPA-DPO, specifically optimizing the trade-off between "description richness vs. factual accuracy" to suppress hallucination — i.e., using RL to guarantee the reliability of the data annotator, treating the "model as annotator" reliability problem as a first-class concern, a fairly advanced practice on this dimension. Whether a VLM is used to remove video-caption mismatches is not stated by either report. [Uncertain]

### Safety and Compliance Filtering (NSFW, copyright, face/privacy) ⚠️

Disclosure is very weak. The original version only mentions in one sentence using a YOLOX-like vision model to remove "watermarks, borders, logos, and certain sensitive information," without elaborating on what "sensitive information" means, and without mentioning an NSFW classifier, face/privacy protection, or copyright filtering. The data section of the 1.5 technical report does not touch on safety filtering at all. As a product released within China, the actual production system necessarily has content-safety review (the model card and license contain compliant-use clauses, and the open-source repository requires compliance with local laws and regulations), but the safety-filtering method on the training-data side is entirely undisclosed. [Uncertain]

## Annotation Methods

### Captioning Model Used (in-house VLM/open-source model, model scale) ⚠️

【Original version】An in-house Vision Language Model (VLM) serves as the captioning model, used to generate structured JSON captions; the parameter scale, architecture, and training details are not published. There is also a separate in-house camera-movement classifier. On the prompt side, the Hunyuan-Large large language model is used to rewrite user prompts.
【1.5】Upgraded to a "three-model division of labor" annotation system, the biggest structural improvement relative to the original version:
1. Image captioning model — follows the method of HunyuanImage-3.0;
2. Video captioning model — produces a highly structured multi-component description;
3. Image-to-Video Instructional Captioning model — a newly added module; instead of describing the content of the whole picture, it specifically describes the "temporal evolution/change relative to the first frame," covering changes in both foreground subject and background environment, converting the text condition used for I2V training from "descriptive" to "instructional," aligning it with the distribution of what users actually input at I2V inference time (giving an image + saying how they want it to move);
plus an additional camera-motion recognition model (dual granularity: clip-level + temporal-level).
The names, parameter counts, and base models of the three captioning models are not disclosed; only the training method is disclosed: OPA-DPO (a preference-optimization method targeting multimodal hallucination) is used for RL post-training to suppress hallucination. [Uncertain]

### Caption Density and Degree of Structuring (short/long/dense descriptions, structured fields such as camera motion, style tags)

【Original version】Dense and strongly structured, one of the most complete caption schemas among open-source video models in 2024. Adopts a JSON format containing seven dimension fields:
1. Short Description (a summary of the main content);
2. Dense Description (a detailed description of scene elements, explicitly including narration of scene transitions and camera movement);
3. Background (background/environment);
4. Style (style: documentary/cinematic/realistic/sci-fi, etc.);
5. Shot Type (shot type and camera position: aerial/close-up/medium shot/long shot, etc.);
6. Lighting (lighting conditions);
7. Atmosphere (mood: warm, tense, mysterious, etc.).
There are also derived metadata fields (source tags, quality tags). High-confidence predictions from the camera-motion classifier are also merged into the JSON caption, granting the model controllable camera-motion capability.
During training, a "dropout mechanism + permutation and combination strategy" is used, randomly dropping/recombining fields from this set of JSON fields to synthesize captions of varied length and phrasing, avoiding the model overfitting to a single caption template — this is a crucial point, directly determining why short prompts also work at inference time.
【1.5】Continues and refines the structure of "multi-level textual narrative + a set of cinematic aesthetic attributes": the cinematic/aesthetic attributes explicitly list shot type, shot angle, composition, lighting, style, color palette, and atmosphere — adding two fields, shot angle and color palette, relative to the original version; camera motion is written into the caption in natural-language form, distinguishing between clip-level (the dominant camera motion for the whole segment) and sequential-level (a sequence of camera motions changing over time). 1.5 does not restate the dropout+permutation-and-combination strategy.

### Joint Audio-Video Caption Structure (whether both visual + auditory tracks are covered simultaneously, whether split into separate fields, e.g. LTX-2's full soundscape description, Script-a-Video's factorized streams, Foley-Omni's three-field scheme) ⚠️

Not applicable. Neither generation of models has an audio modality; captions cover only the visual track, and there is no dual-track visual+auditory caption structure. This can serve as a "pure visual structured caption" reference baseline, contrasted with AV schemes such as LTX-2's full soundscape description, Script-a-Video's factorized streams, and Foley-Omni's three-field scheme: HunyuanVideo's seven-field/multi-attribute schema already has a high degree of field structuring on the visual side, and the audio-track fields of an AV model can be viewed as an orthogonal extension on top of it. [Uncertain]

### Dialogue Transcription and Speaker Attribute Annotation (ASR transcription + speaker identity/language/accent/emotion) ⚠️

Not applicable. There is no audio-track processing, no ASR transcription, and no annotation of speaker identity/language/accent/emotion. The relevant capability within Tencent's system resides in independent models such as HunyuanVideo-Avatar (audio-driven digital humans), not within the scope of this entry. [Uncertain]

### Geometric and Structured Annotation (camera parameters, depth, 3D point tracks, action annotation, explicit state)

One limited but clear item: camera-movement annotation.
【Original version】A camera-motion classifier is trained, predicting 14 types of camera movement: zoom in, zoom out, pan up, pan down, pan left, pan right, tilt up, tilt down, tilt left, tilt right, around left, around right, static shot, handheld shot. Only high-confidence predictions are written into the JSON caption.
【1.5】The camera-motion recognition model is no longer restricted to a fixed 14-class enumeration (the report states it recognizes "various types of camera movement" without enumerating them), instead producing dual-granularity output (clip-level + temporal-level), transcribed into natural language and merged into the caption.
Neither generation uses explicit camera intrinsics/extrinsics, depth maps, 3D point tracks, explicit physical state, or action/skeleton annotation — camera movement is expressed as discrete labels/natural language rather than geometric parameters. This differs from routes such as Seedance and Movie Gen, which introduce stronger geometric supervision.

### Synthetic Data Construction (controlled perturbation/editing to construct training pairs, e.g. InstructAV2AV) ⚠️

Neither generation uses synthetic data to construct training pairs; the reports describe no controlled-perturbation/editing-style paired data (such as InstructAV2AV-type constructions). The only thing with a "constructed" nature is 1.5's T2V DPO stage: for the same prompt, the model itself samples N candidate videos, forming non-repetitive preference pairs, which are then judged win/lose via manual GSB annotation — this is model self-sampled construction of preference data, not synthesis of training corpus. In addition, the original version's caption dropout+permutation-and-combination can be viewed as a form of text-side data augmentation. [Uncertain]

### Degree of Human Involvement (manual annotation, manual quality checking, model pre-screening + human review)

Both generations place human involvement at the tail end of the funnel (model pre-screening + human curation) rather than annotating throughout:
【Original version】The roughly 1 million-sample SFT dataset is entirely manually annotated and screened, with annotators judging across two major categories comprising seven dimensions: aesthetic side — color harmony, lighting quality, object emphasis, spatial layout; motion side — motion speed, action integrity, motion blur. The goal is to select clips that are "visually appealing and rich in motion detail."
【1.5】Human involvement is distributed across three places: (1) the SFT dataset is ultimately built via manual annotation; (2) I2V RLHF prompts and their paired images undergo manual verification of text-image consistency; (3) T2V DPO preference pairs are annotated by humans via GSB (Good/Same/Bad); (4) in the evaluation stage, "100+ professional evaluators" perform GSB human evaluation on 300 text prompts and 300 image samples. Neither the annotation team size nor the person-hour cost is disclosed.

## Audio-Video Alignment

### Audio-Video Synchronization Detection Method (lip sync, event alignment) ⚠️

Not applicable. There is no audio modality, and the training data contains no audio track, so no audio-video synchronization detection stage exists. [Uncertain]

### Synchronization Detection: Specific Metrics and Thresholds (SyncNet/Synchformer/LSE/in-house, threshold values, e.g. MOVA LSE-D≤9.5 and LSE-C≥4.5, SkyReels-V4 SyncNet |offset|≤3 ∧ conf>1.5) ⚠️

Not applicable. No SyncNet, AV-align, or any synchronization metric is used, and there is no threshold. [Uncertain]

### Separate Handling of Temporal vs. Semantic Synchronization (time alignment and content-semantic matching as two independent filtering conditions) ⚠️

Not applicable. With no audio, there is no separate handling of temporal synchronization and semantic synchronization. A visual-side analogy: 1.5 treats "basic rule-based filtering (structural defects)" and "visual quality/aesthetic scoring (perceptual quality)" as two independent conditions, an instance of the same idea on a single modality. [Uncertain]

### Audio Quality Filtering (SNR, silence detection and silence-ratio threshold, no-audio-track removal, off-screen source removal, background-music separation) ⚠️

Not applicable. There is no audio-track processing, and no SNR/silence detection/background-music separation, etc. [Uncertain]

### Classification and Separate Handling Strategies for Speech/Sound-Effects/Music ⚠️

Not applicable. There is no classification and separate handling of speech/sound-effects/music. [Uncertain]

## Training Coordination

### Multi-Stage Training Curriculum and Data-Curriculum Scheduling (basis for stage division: resolution/duration/quality score/modality; low-res→high-res, image→video, short→long)

Both generations follow a textbook "image→video, low-res→high-res, low-frame-rate→high-frame-rate" progressive curriculum, with the data tiers corresponding one-to-one with the training stages (the data tier is the curriculum step):
【Original version】
· Two-stage image pretraining: Stage 1 — 256px multi-aspect-ratio training; Stage 2 — mixed-scale training at 256px and 512px;
· Three-stage joint video-image training: low-resolution short video (256×256×65 frames) → low-resolution long video → high-resolution long video (720×1280×129 frames); video and image are trained jointly throughout to prevent degradation of image capability; the video:image mix ratio in VAE training is 4:1;
· Finally layering on SFT (about 1 million manually curated samples, multi-scale).
【1.5】Eight stages total (Table 2), with the curriculum axis simultaneously spanning resolution, frame rate, and task type:
· Stage I: 256p images, 5 billion, T2I;
· Stage II: 512p images, 1 billion, T2I;
· Stage III: 256p / 16fps / 2-10s, 800 million, mixed T2V+I2V+T2I training, with task ratio 1:6:3 (I2V has the largest share, reaching 60%);
· Stage IV: 480p / 16fps / 2-10s, 200 million;
· Stage V: 720p / 16fps / 2-10s, 100 million;
· Stage VI: 720p / 24fps / 2-10s, 100 million (this stage only raises frame rate, not resolution);
· Stage VII: CT continued training, 480p/720p / 24fps, 1 million, focused on T2V;
· Stage VIII: CT continued training, 480p/720p / 24fps, 1 million, focused on I2V;
· Followed by SFT → RLHF/DPO. There is also an independent super-resolution module trained separately with 1K-4K data.
A noteworthy difference: the original version treats long duration as a curriculum axis (65 frames → 129 frames), while 1.5 keeps a fixed 2-10 seconds throughout with no duration progression, instead using frame rate (16fps→24fps) as a new axis, and outsources the task of raising resolution to an independent super-resolution module — this is a key trade-off 1.5 makes to keep training cost down for the 8.3B model.

### Data Mixture Changes Across Training Stages (pretraining/annealing/high-quality SFT subset)

【Original version】Video and image are trained jointly throughout (joint image-video training) to prevent degradation of the image prior; VAE training explicitly uses video:image = 4:1; the specific video/image mixture at each DiT stage is not published. The SFT stage switches to a manually curated high-quality subset of about 1 million samples.
【1.5】The mixture disclosure is more specific: pretraining stages III-VI use a task mixture ratio of T2V : I2V : T2I = 1 : 6 : 3 (I2V dominates absolutely, reflecting that image-to-video is the primary product use case, and also explaining why an I2V instructional caption is a matching design); data volume shrinks monotonically across stages, 800M→200M→100M→100M→1M, i.e., a typical "shrinking volume, rising quality" annealing-style schedule: large, coarse, low-resolution data is used early to establish a world prior, and small, refined, high-resolution data is used later for polishing. The CT stage splits into two independent branches by task (T2V and I2V, 1 million each), and SFT and RLHF are also executed separately by task.

### Post-Training Data (scale and screening criteria of the SFT curated set, number of preference pairs and annotation method, reward-model training data)

【Original version】Only SFT: about 1 million manually curated samples, screened against aesthetic criteria (color harmony, lighting, object emphasis, spatial layout) plus motion criteria (motion speed, action integrity, motion blur). The original version does not perform RLHF/DPO.
【1.5】Post-training is a three-stage CT → SFT → RLHF, with T2V and I2V kept fully separate throughout:
· CT (continued training): 1 million high-quality clips per task, at 480p/720p, 24fps;
· SFT: further strict screening on top of the CT data, against criteria of aesthetic appeal, clarity, and motion smoothness; the final dataset is built via manual annotation; the specific sample count and thresholds are not published;
· RLHF/DPO: on the I2V side, a prompt set covering 100+ categories is constructed, with paired images carefully selected from high-aesthetic images and manually verified for text-image consistency; on the T2V side, a balanced prompt set on the order of O(10K) (tens of thousands) is constructed, sourced from a mix of LLM-generated prompts and training captions, category-balanced across the three dimensions of motion/scene/subject; for each prompt, N candidate videos are sampled to form non-repetitive preference pairs, with preference data obtained via manual GSB annotation;
· Super-resolution module: trained independently using 1 million 1K-4K high-quality video clips + high-resolution images.
The scale of reward-model training data is not disclosed.

### Data Processing Infrastructure and Throughput (NeMo Curator/Data-Juicer/in-house, GPU speedup, processing scale, cost) ⚠️

Neither generation discloses details of the data-processing infrastructure. There is no mention of NeMo Curator, Data-Juicer, or the name of an in-house data-processing framework, nor GPU speedup ratio, throughput, or processing cost. An inferable scale pressure: 1.5 needed to run segmentation + transition classification + four-dimensional quality scoring + aesthetic scoring + three captioning-model inferences over more than 10 million hours of raw video — a batch-inference task across multiple models at billion-clip scale, an enormous engineering undertaking, yet the paper devotes no space to it. Training-side infrastructure is disclosed to a limited degree (1.5 uses efficiency techniques such as SSTA sparse attention, and the 8.3B model can run inference on a consumer GPU with about 14GB VRAM), but this does not concern data-processing throughput. [Uncertain]

## Performance Comparison

### Quantitative Impact of Data-Strategy Ablations (distinguish: filter-strictness ablation / caption-density-style ablation / data-mixture ablation, and the corresponding evaluation metrics) ⚠️

Neither technical report provides any controlled ablation experiment targeting data strategy — there is no filter-strictness ablation (such as the impact of loosening/tightening an aesthetic threshold on the metrics), no caption density/style ablation (such as short caption vs. dense caption comparison), and no data-mixture ablation (such as comparing T2V:I2V:T2I = 1:6:3 against other ratios). This is the entry's main methodological shortcoming — the pipeline is described in detail, but the necessity of each step lacks quantitative evidentiary support.
What both generations provide instead is end-to-end human comparative evaluation rather than ablation: the original version had 60 professional evaluators compare against five closed-source models such as Gen-3 and Luma 1.6 across 1,533 prompts, coming out ahead comprehensively on text alignment, motion quality, and visual quality; 1.5 had 100+ professional evaluators compare against Wan2.2, Kling 2.1, Seedance Pro, and Veo3 across 300 text prompts and 300 images, using Rating and GSB methods. These results cannot be attributed to any specific data strategy. [Uncertain]

### Evidence for Quality vs. Quantity (cases of small, refined data outperforming large, messy data) ⚠️

There is no direct controlled-experiment evidence, but the funnel design of both generations itself is a strong expression of a "quality over quantity" stance, and 1.5 provides quantifiable indirect evidence:
· 1.5's data volume shrinks monotonically from 800 million to 1 million (end-to-end retention rate 0.125%), yet with only 8.3B parameters (about 64% of the original version's 13B) matches larger, more expensive models such as Kling 2.1, Seedance Pro, and Veo3 in human evaluation; the team attributes this to "meticulous data curation" being one of several key factors — this is a case-level piece of evidence that "a small model + refined data can rival large models," but it lacks a controlled comparison of "same model, different data," so strictly speaking it does not constitute experimental proof.
· The original version explicitly states each filtering stage eliminates 50%-80% of the data, and reserves the most expensive step, manual annotation, only for the final 1 million samples, reflecting a resource-allocation philosophy of "the further along, the more refined, and the more concentrated the investment."
· A noteworthy piece of reverse signal: 1.5, relative to the original version, expands the raw pool to more than 10 million hours (a significant scale-up), indicating the team's actual practice is "first expand the candidate pool, then filter more strictly," i.e., quality improvement comes from a higher elimination rate rather than a smaller starting pool — quantity and quality are not a trade-off relationship. [Uncertain]

### The Alignment Relationship Between Training-Data Domain Distribution and Evaluation-Benchmark Taxonomy (e.g. VABench's seven major categories) ⚠️

No explicit alignment statement exists between the training-data domain taxonomy and the evaluation-benchmark taxonomy. There is no mapping relationship between the original version's listed data domains (people/animals/plants/landscapes/vehicles/objects/buildings/animation) and its human-evaluation dimensions (text alignment, motion quality, visual quality). 1.5's category system in the RLHF stage (I2V's 100+ categories, T2V's motion/scene/subject three-dimension balancing) is closer to an evaluation-oriented category division, but the report does not state its correspondence to the category systems of public benchmarks such as VBench or VABench. Both generations rely predominantly on manual GSB evaluation and do not use public benchmark taxonomies to inversely inform training-data mixture. [Uncertain]

## Other Information

### summary_note

Core conclusion: the HunyuanVideo series is one of the open-source samples in this survey with the most complete disclosure of pure-visual-side data-processing methodology, but it involves no audio at all, so it has no direct reference value on the topic of joint audio-video generation.
The value contributed by the two generations is clearly divided:
(1) HunyuanVideo original (2024-12) — contributes a "hierarchical filtering funnel + structured caption schema" paradigm template: four resolution-tiered datasets, each progressively stricter (each stage retains 1/2 to 1/5 of the previous), VideoCLIP-embedding semantic deduplication + 10K-concept-centroid k-means resampling for concept balancing, Dover/blur/optical-flow/OCR/YOLOX-like multi-scorer combination, a seven-field JSON caption + a 14-class camera-motion label + caption dropout and permutation-combination augmentation, and a tail-end 1-million manually curated SFT set. This design has been widely borrowed by later open-source works.
(2) HunyuanVideo 1.5 (2025-11) — contributes "quantitative funnel + annotator-reliability governance": it gives a complete quantitative chain from >10 million raw hours → 800 million clips → 200 million → 100 million → 1 million (end-to-end retention rate 0.125%), changes watermark/subtitle handling from "discard the whole clip" to "spatial crop + a 60% area floor," uses a dedicated transition classifier to catch what shot detection misses, splits visual quality into four operator dimensions — sharpness/detail retention/noise-artifacts/dynamic range — and pioneers an "instructional caption" for I2V (describing temporal evolution relative to the first frame rather than static content), using OPA-DPO to post-train the captioning model via RL to suppress annotation hallucination. The training-side mixture ratio T2V:I2V:T2I = 1:6:3 is also a rare instance of explicit disclosure.
Main shortcomings: neither generation has any controlled ablation experiment on data strategy, so the benefit of each pipeline component cannot be quantitatively attributed; safety/copyright/privacy filtering disclosure is nearly blank; data-processing infrastructure and throughput are entirely undisclosed; and 1.5's description of deduplication and concept balancing is actually weaker than the original version's (unclear whether this was simplified or simply left unwritten).
If a research goal involves data construction for joint audio-video generation, this entry can only serve as a baseline reference for visual-side filtering and annotation; the audio side requires separate research into Tencent's HunyuanVideo-Foley (a roughly 100,000-hour TV2A dataset) and HunyuanVideo-Avatar.

## Uncertain Fields

The information for the following fields is partially uncertain (source labeled ⚠️):

- data_sources
- provenance_licensing
- domain_distribution
- audio_category_distribution
- language_accent_distribution
- motion_filtering
- deduplication
- model_as_data_judge
- safety_filtering
- caption_model
- joint_av_caption_schema
- dialogue_transcription_attributes
- synthetic_data_synthesis
- av_sync_detection
- sync_metric_and_threshold
- temporal_vs_semantic_sync
- audio_quality_filtering
- audio_type_handling
- data_infra_throughput
- data_ablation
- quality_over_quantity_evidence
- benchmark_taxonomy_alignment
