# Hailuo / MiniMax Video (海螺AI视频). This is a product line rather than a single model; the model IDs across generations include: video-01 (September 2024, first generation, alias Hailuo), video-01-live (Live2D/anime-specialized), video-01-director (camera-movement control), S2V-01 (Subject-Reference), MiniMax-Hailuo-02 (June 2025), MiniMax-Hailuo-2.3 / 2.3-Fast (October 2025, still the latest online version in official documentation as of July 2026)

> Topic: Data processing for video generation models (including joint audio-video generation): data cleaning pipeline, data distribution, annotation approach

[← Back to home](../index.md)

**Contents**: [Basic Information](#basic-information) · [Data Scale and Distribution](#data-scale-and-distribution) · [Cleaning Pipeline](#cleaning-pipeline) · [Captioning Approach](#captioning-approach) · [Audio-Video Alignment](#audio-video-alignment) · [Training Coordination](#training-coordination) · [Effectiveness Comparison](#effectiveness-comparison)

## Basic Information

### Name

Hailuo / MiniMax Video (海螺AI视频). This is a product line rather than a single model; the model IDs across generations include: video-01 (September 2024, first generation, alias Hailuo), video-01-live (Live2D/anime-specialized), video-01-director (camera-movement control), S2V-01 (Subject-Reference), MiniMax-Hailuo-02 (June 2025), MiniMax-Hailuo-2.3 / 2.3-Fast (October 2025, still the latest online version in official documentation as of July 2026)

### Releasing institution/company

MiniMax (稀宇科技 / Shanghai MiniMax (Xiyu) Technology Co., Ltd.), Shanghai, China. The same company's other product lines include the MiniMax-01/M1/M2/M2.5/M2.7/M3 series of large language models, MiniMax Speech (speech), MiniMax Music (music), and MiniMax Code. The external brand for the video product is "海螺AI" (Hailuo AI, hailuoai.video / hailuoai.com)

### Release date (technical report/paper/open-source date)

There is no technical report or paper, only product-launch blog posts; the timeline is as follows:
- September 2024: video-01 (first-generation Hailuo video, 6 seconds, 1280×720, 25fps)
- January 2025: S2V-01 (Subject-Reference / character consistency)
- June 18, 2025: MiniMax Hailuo 02 (introduces the NCR architecture, native 1080p)
- October 28, 2025: MiniMax Hailuo 2.3 / 2.3 Fast, simultaneously upgrading Video Agent to Media Agent
- December 15, 2025: open-sourced the VTP visual tokenizer (arXiv:2512.13687), a generative foundational component from the same team, not the video generation model itself
As of the time of this research (July 29, 2026), the latest video model on both the MiniMax official news page and the platform documentation remains Hailuo 2.3, with no announcement observed of a new-generation video model released in 2026.

### Type (model/dataset/toolchain/evaluation benchmark)

Model (a closed-source commercial video generation model/product line, covering text-to-video (T2V), image-to-video (I2V), First-and-Last-Frame, Subject-Reference, and other modes). It is not a dataset, not a toolchain, and not an evaluation benchmark; MiniMax has also not released an accompanying video evaluation benchmark.

### Degree of openness (whether weights/code/data/pipeline are each open source)

Fully closed source — one of the least data-disclosed subjects in this research effort:
- Model weights: not open source, inference service is provided only via the Hailuo AI web/app, the MiniMax open platform API, and third-party hosting platforms such as Replicate and fal.ai;
- Training/inference code: not open source;
- Training data: not open source, not described;
- Data-processing pipeline: not open source, not described;
- Technical report/paper: none whatsoever. Every generation of the model has had only product-launch blog posts (minimax.io/news), whose content is mostly capability demos and pricing, containing almost no technical detail; the NCR architecture is likewise described with only a single naming phrase, with no supporting paper.
By contrast, MiniMax is highly open on the language-model side (MiniMax-01, M1, M2/M2.1/M2.5/M2.7, and M3 all release weights and technical reports on Hugging Face), whereas the video side is deliberately fully closed. The only open-source artifact related to visual generation is the VTP (Visual Tokenizer Pre-training) series of visual tokenizers released on December 15, 2025 (Small 0.2B / Base 0.3B / Large 0.7B, Modified MIT license, arXiv:2512.13687), but it is positioned as an image-tokenizer foundational component — its model card does not clarify any direct relationship to the Hailuo video model, and its training-data composition is likewise not explicitly listed in the model card.

### Whether joint audio-video generation is supported, and the implementation approach (native joint/cascade/MoE fusion)

Not supported (as of Hailuo 2.3 / July 2026).
- The MiniMax open platform's video generation API documentation contains no audio/sound parameters whatsoever; output is silent video;
- All 7 Hailuo/video-01 series models hosted on Replicate describe no capability for producing an audio track alongside the video;
- MiniMax's audio capability is handled by entirely separate product lines: MiniMax Speech (speech synthesis, version 2.8) and MiniMax Music (music generation, version 3.0); on the Hailuo AI website, "Audio" is a standalone entry point parallel to Video;
- From a product-experience standpoint, Hailuo AI's Media Agent (launched alongside 2.3 in October 2025) can chain text → video → speech/music in one click, which is a cascade at the product-orchestration layer, not native joint generation or MoE fusion at the model layer.
Accordingly, in the main line of this research on "joint audio-video generation," this entry serves as a counterexample/control group: it represents a leading commercial model that remains in a purely visual-generation paradigm after Veo 3, Sora 2, Kling 3.0 Omni, LTX-2, and others have already moved to native AV joint generation. Correspondingly, all AV-related fields in this research have no substantive content for this subject.

### List of research information sources (URLs of papers/technical reports/official documentation/news, with each source's nature annotated: official first-hand/same-team corroboration/third-party report)

Time of research: July 29, 2026. This subject has no paper and no technical report; all information comes from product blogs, API documentation, and third-party hosting platforms, with extremely limited disclosure on the data side.
1) Official first-hand: MiniMax Hailuo 02 launch announcement https://www.minimax.io/news/minimax-hailuo-02 (June 18, 2025; the only source in this research that mentions training-data scale — 3x the parameter count, 4x the training data, with improved quality and diversity; NCR architecture 2.5x efficiency; native 1080p; three tiers of 768p-6s/768p-10s/1080p-6s; ranked second globally on the Artificial Analysis Video Arena)
2) Official first-hand: MiniMax Hailuo 2.3 / 2.3 Fast launch announcement https://www.minimax.io/news/minimax-hailuo-23 (October 28, 2025; body movement, stylization including ink-wash painting and game CG, micro-expressions, responsiveness to motion commands; Video Agent upgraded to Media Agent; 2.3 Fast reduces batch cost by up to 50%; explicitly contains no resolution, duration, audio, training-data, or architecture details)
3) Official first-hand: MiniMax open platform video generation documentation https://platform.minimax.io/docs/guides/video-generation (model IDs: MiniMax-Hailuo-2.3 supports T2V/I2V, MiniMax-Hailuo-02 supports First-and-Last-Frame, S2V-01 supports Subject-Reference; 1080P/6-second examples; the API contains no audio parameters whatsoever)
4) Official first-hand: Hailuo AI product site https://hailuoai.video/ (video/image/audio are parallel standalone entry points, corroborating that audio and video are not the same model; template categories; AI-generated-content compliance notice)
5) Official first-hand/same-team corroboration: MiniMax Hugging Face organization page https://huggingface.co/MiniMaxAI (as of July 2026, the open-source models are language models such as MiniMax-M3 (427B), M3-MXFP8, M2.7/M2.5/M2.1/M2 (229B), M1-40k-hf (456B), and the VTP series of visual tokenizers; no video generation model is open source whatsoever — a key piece of corroboration that "the video line is fully closed source")
6) Same-team corroboration: VTP visual tokenizer model card https://huggingface.co/MiniMaxAI/VTP-Large-f16d64 and the paper "Towards Scalable Pre-training of Visual Tokenizers for Generation" https://arxiv.org/abs/2512.13687 (December 15, 2025, Modified MIT; a visual tokenizer jointly optimized with image-text contrastive + self-supervised + reconstruction triple losses; the model card does not clarify any direct relationship to the Hailuo video model, and training data is likewise not explicitly listed)
7) Third-party hosting platform: Replicate MiniMax model list https://replicate.com/minimax (lists 7 video models — hailuo-2.3, hailuo-2.3-fast, hailuo-02, hailuo-02-fast, video-01, video-01-live, video-01-director — along with run counts; key corroboration: none of the models describe a capability to produce an audio track alongside the video)
Note: The WebSearch quota for this research session had been exhausted; all of the above information was obtained by directly crawling (WebFetch) known official URLs, and did not cover unofficial leaks or reverse-engineering analyses from community sources such as Reddit, Zhihu, and CSDN, nor did it exhaustively search for possible new-version announcements in the first half of 2026. If supplementation is needed later, the recommended priority search directions are: whether MiniMax released a video model with native audio in 2026, whether a paper on the NCR architecture has been published, and Chinese-community (Zhihu/JIQIZHIXIN/QbitAI) reporting on the data sources for Hailuo video.

## Data Scale and Distribution

### Training data scale (number of video clips/hours/tokens, pretraining vs. SFT separated) ⚠️

[Uncertain] (there is only a single relative-scale disclosure, with no absolute figures whatsoever). The MiniMax official Hailuo 02 launch blog (June 18, 2025) is the only place in all public material that mentions training-data scale, stating: compared with the previous generation, the parameter count increased by about 3x (3x parameters), and the training data expanded by about 4x (4-fold expansion of training data), with quality and diversity improving in tandem. Beyond this:
- No number of video clips, total duration (hours), or token count has ever been disclosed;
- No distinction has ever been drawn between pretraining and SFT/post-training data volumes;
- The base figure for the previous-generation video-01 is likewise undisclosed, so the "4x" is a relative figure that cannot be anchored to an absolute value;
- The Hailuo 2.3 launch announcement makes no mention whatsoever of data scale, describing only capability improvements (body movement, stylization, micro-expressions, instruction following).
This is a textbook case of "marketing-style disclosure": a growth multiple is given to convey the level of investment, but no verifiable base figure is provided at all.

### Data source composition (in-house/public datasets/web crawling/licensed acquisition/synthetic data) ⚠️

[Uncertain]. MiniMax has never disclosed the composition of the Hailuo video model's data sources, and has not stated the respective proportions or presence/absence of in-house data, public datasets, web crawling, licensed acquisition, or synthetic data. Indirect corroboration that can be inferred (none of it an official statement, and of low reliability):
- The Hailuo 02 blog claims the training data improved in "quality and diversity," implying the existence of source expansion and filtering steps, but names no specific source;
- Hailuo 2.3 explicitly extends support for styles such as ink wash painting, game CG, anime, and illustration, and there was already the Live2D/anime-specialized video-01-live model early on, suggesting the training data contains a considerable proportion of anime/manga/game material — a pattern fairly common among Chinese vendors;
- The model's emphasis on realistic micro-expressions and live-action facial performance suggests a large amount of live-action film/TV and talking-head material.
The above is merely an inference of data distribution reverse-engineered from capabilities, with no first-hand basis.

### Data compliance and provenance (proportion of licensed data, rights-cleared datasets, C2PA, etc.) ⚠️

[Uncertain]. MiniMax has never issued any data-compliance statement for the Hailuo video model: it has not disclosed the proportion of licensed data, has not mentioned rights-cleared datasets, and has not mentioned C2PA or any content-provenance/source-attestation standard. The Hailuo AI web front end carries only a generic disclaimer, "Content is AI-generated; please use this feature legally and amicably," which is a user-side usage norm rather than a training-data provenance commitment. Whether output videos carry a visible or invisible watermark, or embed C2PA metadata, is likewise not stated in official documentation. This stands in clear contrast to the compliance-disclosure practices of Adobe Firefly, OpenAI Sora 2 (C2PA + visible watermark), and Google Veo (SynthID).

### Clip duration distribution and segmentation strategy ⚠️

[Uncertain] (only output-side specifications exist, with no training-data-side distribution).
Output side: video-01 is fixed at 6 seconds; Hailuo 02 offers three tiers — 768p-6s, 768p-10s, 1080p-6s; Hailuo 02 Fast offers 6s and 10s at 512p; the Hailuo 2.3 official documentation example is 1080p / 6 seconds.
Training side: the duration distribution of training clips, the average clip length, and the strategy for segmenting long videos into clips have never been disclosed officially. One can only infer, from the 6s/10s output tiers, that training clips roughly fall on the order of within 10 seconds, but this is an inference, not a disclosure.

### Resolution/aspect-ratio distribution and bucketing strategy ⚠️

[Uncertain] (again, only output-side specifications exist).
Output side: first-generation video-01 is 1280×720 / 25fps; Hailuo 02 claims "native 1080p" and offers two tiers, 768p and 1080p; Hailuo 02 Fast is a low-cost 512p tier; the Hailuo 2.3 documentation is marked 1080P.
Training side: the resolution distribution, aspect-ratio distribution, and whether a multi-resolution/multi-aspect-ratio bucketing training strategy is used are entirely undisclosed officially. The phrase "native 1080p" implies that high-resolution data occupies a substantial proportion of the training set (rather than relying solely on super-resolution post-processing), and there is likely a low-resolution → high-resolution multi-resolution curriculum, but there is no first-hand statement to this effect.

### Category/domain distribution and mixture strategy (proportional control and concept balancing across people, actions, scenes, styles, etc.) ⚠️

[Uncertain] (no quantitative mixture ratios disclosed whatsoever). MiniMax has never published proportions across categories such as people/actions/scenes/styles, and has not mentioned concept balancing or long-tail category supplementation strategies. Qualitative domain coverage that can be reverse-inferred from product capabilities (not an official data disclosure, for reference only):
(1) Live-action realism: Hailuo 2.3 emphasizes "live-action facial performances" and the naturalness of micro-expressions; its description on Replicate is "optimized for realistic human motion, cinematic VFX, and expressive characters," suggesting live-action + cinematic material is a core domain;
(2) Complex human motion and physics: Hailuo 02 emphasizes rendering of physical laws (e.g., high-difficulty movements such as gymnastics and diving were widely circulated demos), and 2.3 emphasizes "more complex character body movements," suggesting high-dynamic human motion is a deliberately reinforced domain;
(3) Anime/ACG: as early as video-01-live, the model was specialized for Live2D and general animation, and 2.3 explicitly extends support for anime and illustration;
(4) Chinese art styles: 2.3 specifically names ink wash painting and game CG, a targeted domain reinforcement aimed at the Chinese market;
(5) Camera language: video-01-director was separately trained for specific camera movements, suggesting a subset exists with camera-movement annotations.
The relative proportions of these five categories, and whether there is an explicit mixture-control mechanism, are entirely undisclosed.

### Audio category distribution and mixture (proportions and control strategy for speech/foley/music/ambient sound/silence) — a dimension unique to AV models ⚠️

Not applicable + [Uncertain]. The Hailuo video model does not generate audio; whether the original audio track is retained in the training data, and how the proportions of speech/sound effects/music/ambient sound/silence are controlled, is entirely unmentioned officially, and given the modeling objective it was very likely simply discarded during the data-processing stage. MiniMax's audio-data capability resides in the entirely separate MiniMax Speech (speech) and MiniMax Music (music) product lines, but the training data for these two lines is likewise undisclosed, and there is no evidence that they share a data-processing pipeline with the video line. This field therefore has no substantive content for this subject.

### Narrative structure distribution (single-shot vs. multi-shot, average clip duration, shot-count distribution, whether native audio tracks are included) ⚠️

[Uncertain]. The proportion of single-shot vs. multi-shot, the average clip duration, and the shot-count distribution are undisclosed, and it is also unstated whether training clips retain native audio tracks (given that the model does not generate audio, it is likely that none was retained). Indirect clue: output is a single-segment video of 6–10 seconds, and video-01-director specifically does camera-movement control (camera movement within a single shot, not multi-shot editing), suggesting that training data consists overwhelmingly of single-shot clips, with no observed modeling or data design for multi-shot narrative/cut coherence. Multi-shot long-video capability at the product level is achieved by the Media Agent stitching things together at the orchestration layer, rather than being natively possessed by the model.

### Language/accent distribution (data basis for multilingual lip-sync capability) ⚠️

Not applicable + [Uncertain]. The model does not generate speech and has no lip-sync task, so the question of a data basis for language/accent distribution does not arise. On the text side, prompts support both Chinese and English, but MiniMax has not disclosed the language composition of training captions or the Chinese-English ratio. The product targets both the Chinese and overseas markets (hailuoai.com and hailuoai.video / minimax.io as dual sites), leading to the presumption that captions are bilingual in Chinese and English, but there is no first-hand basis for this.

## Cleaning Pipeline

### Overall structure of the cleaning funnel (number of filtering stages, order of stages) ⚠️

[Uncertain] (no disclosure whatsoever). MiniMax has never disclosed the structure of the Hailuo video data-cleaning pipeline — no number of funnel stages, no order of filtering stages, no flow diagram. The closest thing to a statement about data processing in all public material is a single sentence in the Hailuo 02 blog: "training data expanded 4-fold, with improved quality and diversity," which implicitly suggests the existence of quality-filtering and diversity-balancing steps, but says nothing at all about the specific methods.
This stands in stark contrast to subjects in this research such as Vidu S1 (a six-stage funnel diagram), Seedance, Movie Gen, and Cosmos, which provide complete pipeline descriptions. In terms of data-processing methodology, Hailuo sits at a "zero disclosure" tier, and can serve as a reference point for the floor of disclosure among commercial closed-source models, but it offers no process detail whatsoever that could be drawn upon.
It should be noted that MiniMax provides a relatively detailed description of data cleaning on the language-model side (the MiniMax-01/M1/M2/M3 technical reports), but these concern the text-data pipeline, which cannot be directly transferred to video-data processing, and this research does not extrapolate across modalities.

### Quantitative funnel retention rates (input/output volume at each filtering stage and the final retention rate, e.g. Apollo's 27%) ⚠️

[Uncertain] (no disclosure whatsoever). There is no input/output volume for any filtering stage, and no final retention rate. Since even the funnel structure itself is undisclosed, quantitative retention rates are all the more out of the question.

### Shot segmentation method (PySceneDetect/in-house model/shot-aware splitting) ⚠️

[Uncertain] (no disclosure whatsoever). It is not stated whether PySceneDetect, TransNetV2, or an in-house shot-boundary detection model is used, nor whether shot-aware splitting is performed. One can only infer, from "output is a single-shot clip of 6–10 seconds," that the training data must have gone through some form of shot segmentation, but the method is entirely unknown.

### Quality filtering (aesthetic scoring, sharpness, OCR text filtering, letterbox/watermark/logo detection) ⚠️

[Uncertain] (no disclosure whatsoever). No mention is made of any specific means such as an aesthetic scorer, technical-quality/sharpness scoring, OCR text detection, letterbox cropping, or watermark/logo detection, and no threshold of any kind is given. The only relevant official wording is Hailuo 02's "improved quality and diversity" and Hailuo 2.3's capability descriptions, both of which are outcome claims rather than method disclosures.
Indirect evidence reverse-inferable from model behavior (speculative, not disclosed): Hailuo-series output performs well in terms of frame cleanliness and cinematic feel, with subtitle/logo residue rarely appearing, suggesting effective frame-cleanliness and watermark/subtitle filtering exists; "native 1080p" suggests the existence of a resolution/sharpness threshold.

### Motion filtering (optical-flow/motion-score thresholds, removal of static and jittery footage) ⚠️

[Uncertain] (no disclosure whatsoever). No detail is given regarding optical-flow computation, motion-score thresholds, static-frame removal, or jitter removal. Indirect clue: both Hailuo 02 and 2.3 position "complex human motion, physical plausibility, large dynamics" as a core selling point, with 2.3 further emphasizing "maintaining smooth, natural, and precise complex body movements even under dynamic camera work" — this strongly suggests the data side involves targeted filtering or weighting for high-motion samples (removing static clips, retaining high-dynamic samples), but there is no first-hand statement of the method or thresholds involved.

### Deduplication method (exact deduplication and embedding-based semantic deduplication recorded separately) ⚠️

[Uncertain] (no disclosure whatsoever). It is not stated whether exact (hash-level) deduplication or embedding-based semantic deduplication is performed, and no feature-extraction method or similarity threshold is given.

### VLM/LLM as quality inspector (multimodal large-model quality scoring and mismatch removal; the 2026 trend of moving from shallow scorers to large-model semantic judgment) ⚠️

[Uncertain] (no disclosure whatsoever). No mention is made of using a VLM/LLM as a data quality inspector. A noteworthy piece of indirect background: MiniMax itself possesses strong multimodal large-model capability (the MiniMax-01 series has VL capability, and M3 is a 427B image-text-input model), and open-sourced the VTP visual tokenizer in December 2025 — technically it fully possesses the conditions to use its own omni-modal models for data quality inspection and semantic annotation; but the company has never officially confirmed whether this approach is adopted in the video-data pipeline. This field therefore falls into "has the capability but no disclosure," and cannot be counted as evidence for the 2026 trend of "moving from shallow scorers to large-model semantic judgment."

### Safety and compliance filtering (NSFW, copyright, face/privacy) ⚠️

[Uncertain] (no disclosure on the data side, only inference-side evidence). NSFW filtering, copyright filtering, and face/privacy handling on the training-data end are all undisclosed. Only inference-side content-safety policy is observable: Hailuo AI and the open-platform API apply clear real-time review and interception to prompts and generated results touching on politics, pornography, violence, and the likenesses of public figures (a compliance requirement under China's regulatory environment, the "Interim Measures for the Administration of Generative Artificial Intelligence Services"), and the Hailuo AI web front end displays the notice "Content is AI-generated; please use this feature legally and amicably." The existence of strict inference-side review can indirectly indicate that the company possesses a mature content-safety model, but one cannot directly infer from it the manner and intensity of its application at the training-data-cleaning stage.

## Captioning Approach

### Captioning model used (in-house VLM/open-source model, model scale) ⚠️

[Uncertain] (no disclosure whatsoever). It is not stated whether an in-house VLM or an open-source model is used for video captioning, nor is any model scale given. Background corroboration (not a disclosure): MiniMax's in-house multimodal large models — MiniMax-01 (VL version) and MiniMax-M3 (427B, Image-Text-to-Text) — possess the capability to perform video captioning, and the company has a high degree of internal model self-sufficiency, leading to the presumption that captioning is done by an in-house model, but there is no first-hand confirmation.

### Caption density and degree of structuring (short/long/dense descriptions, structured fields such as camera movement, style tags) ⚠️

[Uncertain] (no disclosure whatsoever). The length and density of captions, whether they are structured, and whether they contain fields such as camera movement/style/lighting are all undisclosed. Indirect evidence that can be reverse-inferred (speculative):
(1) The Hailuo series performs notably well on instruction following for long, complex prompts (Hailuo 02 officially claims SOTA Instruction Following), which typically corresponds to long, dense training captions;
(2) video-01-director supports specifying concrete camera movements via bracketed instructions embedded in the prompt (such as [Push in], [Pan left], [Truck left], a vocabulary of roughly 15 camera-movement terms) — this is the only "structured tagging system" trace confirmable in public API documentation. It strongly suggests that the training data contains an explicit, vocabulary-based camera-movement annotation field, and that this field was designed to be directly callable by the user at inference time;
(3) 2.3's emphasis on "motion commands" suggests that action-type labels are likewise structured.
The above is an inference of the training annotation schema reverse-engineered from a controllable interface; the actual organizational form of the captions remains unknown.

### Joint audio-video caption structure (whether visual + auditory tracks are covered simultaneously, whether split into separate fields — e.g. LTX-2's full soundscape description, Script-a-Video's factorized streams, Foley-Omni's three fields)

Not applicable. The model does not generate audio; training captions do not involve description of the auditory track, and no joint audio-video caption structure exists. There is no basis for comparison with schemes such as LTX-2's full soundscape description, Script-a-Video's factorized streams, Foley-Omni's three fields, or Vidu S1's dual-path audio-visual decoupling. If MiniMax releases a video model with audio in the future, this item will need to be re-researched.

### Dialogue transcription and speaker-attribute annotation (ASR transcription + speaker identity/language/accent/emotion) ⚠️

Not applicable + [Uncertain]. There is no audio-generation task, so ASR transcription and speaker identity/language/accent/emotion annotation are not needed. It is undisclosed officially whether the original video's dialogue information was retained and utilized in the training data (e.g., as a semantic cue aiding captioning); presumed not used.

### Geometric and structured annotation (camera parameters, depth, 3D point tracks, action annotation, explicit state) ⚠️

[Uncertain] (no explicit disclosure, only strong indirect clues). No mention is made of camera-parameter calibration, depth maps, 3D point tracks, pose/skeletal keypoints, or explicit physical-state annotation. Indirect clues:
(1) video-01-director's camera-movement vocabulary (roughly 15 combinable camera-movement instructions) indicates the existence of at least discrete camera-movement labeling, but this is a semantic label rather than continuous camera extrinsics;
(2) Hailuo 02/2.3's emphasis on physical-law plausibility and complex human motion may correspond to pose- or kinematics-level annotation/filtering, but there is no statement of any kind;
(3) S2V-01's Subject-Reference capability requires data support for character-identity consistency, suggesting the existence of face/identity-embedding-level annotation and paired-data construction, which is likewise undisclosed.

### Synthetic data construction (controlled perturbation/edit-constructed training pairs, e.g. InstructAV2AV) ⚠️

[Uncertain] (no disclosure whatsoever). No mention is made of using synthetic data, controlled perturbation, or edit-constructed training pairs (such as InstructAV2AV-style pair construction). S2V-01's Subject-Reference task in principle typically requires "the same character across clips" paired data, which could plausibly be constructed automatically via identity clustering rather than manual collection, but this is a methodological speculation with no first-hand basis.

### Degree of human involvement (human annotation, human quality inspection, model pre-screening + human review) ⚠️

[Uncertain] (no disclosure whatsoever). It is not stated whether human annotation, human quality inspection, or a "model pre-screening + human review" step exists on the data side, and the scale of any annotation team is undisclosed. On the product side, there is a clear human-feedback channel: the Hailuo AI/Hailuo AI web front end and app provide user ratings and reporting entry points for generated results, and Media Agent supports manual human adjustment, which could in principle accumulate into preference data, but officially it is not stated whether this feedback flows back into training.

## Audio-Video Alignment

### Audio-video synchronization detection method (lip sync, event alignment)

Not applicable. The model does not generate audio, so no audio-video synchronization detection step (lip sync, event alignment) exists in the training pipeline. Even if original audio tracks were retained in the training data, there is no evidence they were used for synchronization-based filtering.

### Specific synchronization detection metrics and thresholds (SyncNet/Synchformer/LSE/in-house, threshold values, e.g. MOVA's LSE-D≤9.5 and LSE-C≥4.5, SkyReels-V4's SyncNet |offset|≤3 ∧ conf>1.5)

Not applicable. There is no SyncNet / Synchformer / LSE-D / LSE-C or any other synchronization metric or threshold of any kind. Not comparable to subjects such as MOVA (LSE-D≤9.5 and LSE-C≥4.5), SkyReels-V4 (SyncNet |offset|≤3 and conf>1.5), or Vidu S1 (Sync-D evaluation).

### Separate handling of temporal synchronization vs. semantic synchronization (temporal alignment and content-semantic matching as two independent filtering conditions)

Not applicable. There is no separate-handling issue between temporal synchronization and semantic synchronization.

### Audio quality filtering (SNR, silence detection and silence-ratio threshold, removal of tracks without audio, removal of off-screen sound sources, background-music separation) ⚠️

Not applicable + [Uncertain]. There is no audio-modeling objective, and it is undisclosed officially whether training data retains audio tracks; if the audio track is discarded at the segmentation stage, then no audio-quality-filtering steps of any kind — SNR, silence ratio, off-screen-sound removal, background-music separation — exist. No first-hand information whatsoever.

### Classification and separate handling strategy for speech/sound effects/music

Not applicable. There is no distinction among speech/sound effects/music, and no audio-type classification or separate-handling strategy. MiniMax's speech (MiniMax Speech 2.8) and music (MiniMax Music 3.0) capabilities are handled by separate models and separate data pipelines, with no observed evidence of any data-level integration with the video line.

## Training Coordination

### Multi-stage training curriculum and data-curriculum scheduling (basis for stage division: resolution/duration/quality score/modality; low-res→high-res, image→video, short→long) ⚠️

[Uncertain] (no disclosure, only architecture-level indirect clues). No training-stage division, data-curriculum scheduling (stage progression along resolution/duration/quality-score/modality dimensions) has been publicly disclosed, nor is any image→video, low-res→high-res, or short→long curriculum design described.
The only publicly known information related to a training mechanism is the NCR (Noise-aware Compute Redistribution) architecture introduced with Hailuo 02, which is officially claimed to improve training and inference efficiency by about 2.5x at equal parameter scale. Judging from the name, NCR is a mechanism that dynamically allocates compute according to the noise level (timestep) of the diffusion process — i.e., investing different amounts of compute at different denoising stages — which is a training/inference efficiency optimization rather than data-curriculum scheduling. MiniMax has not published a paper or technical details on NCR; this item is interpreted based on the name alone, as speculation.
Additionally, the phrase "native 1080p" suggests the high-resolution stage is a genuine training stage rather than pure post-hoc super-resolution, and possibly corresponds to a high-resolution fine-tuning stage, but again there is no first-hand basis.

### Change in data mixture across training stages (pretraining/annealing/SFT high-quality subset) ⚠️

[Uncertain] (no disclosure whatsoever). It is not stated how the data mixture changes across the pretraining, annealing, and SFT stages, nor is it mentioned whether a high-quality-subset fine-tuning stage exists. Hailuo 02's "4x the data volume + improved quality and diversity" is a blanket description of the training set as a whole, with no stage-level breakdown.

### Post-training data (scale and selection criteria of the SFT curated set, number of preference pairs and annotation method, reward-model training data) ⚠️

[Uncertain] (no disclosure whatsoever). The scale and selection criteria of the SFT curated set are unpublished, the number of preference pairs (DPO/RLHF) and their annotation method are unpublished, and no mention is made of a reward model or its training data.
Indirect clue (speculative): Hailuo 2.3's improvements over 02 are concentrated in subjective-feel dimensions such as "naturalness of micro-expressions, fluency and precision of body movement, stylization quality, and responsiveness to motion commands," and with no claimed change in parameter count or architecture, such improvements in the industry typically come from high-quality SFT-subset fine-tuning and human-preference alignment (aesthetic/motion-quality reward models); it is presumed that MiniMax built corresponding post-training data, but there is no official confirmation of any kind. The Hailuo AI product side's enormous volume of user-generated content and rating/reporting entry points could in principle accumulate preference data, but whether this is used for RLHF is unknown.

### Data-processing infrastructure and throughput (NeMo Curator/Data-Juicer/in-house, GPU acceleration ratio, processing scale, cost) ⚠️

[Uncertain] (no disclosure on the data side). No mention is made of NeMo Curator, Data-Juicer, or an in-house data-processing system, and no GPU acceleration ratio, processing scale, or cost is given.
The only officially disclosed efficiency-related information is on the model side rather than the data side: the NCR architecture improves training and inference efficiency by about 2.5x; both Hailuo 02 and 2.3 position themselves around "the industry's most open access and most cost-effective pricing" as selling points, with 2.3 Fast further reducing bulk creation cost by up to 50%, indicating that inference-cost optimization is a key area of investment — but none of this involves data-processing infrastructure.

## Effectiveness Comparison

### Quantified impact of data-strategy ablations (distinguishing: filtering-strictness ablation / caption-density-style ablation / data-mixture ablation, and corresponding evaluation metrics) ⚠️

[Uncertain] (entirely absent). MiniMax has published no paper or technical report, and consequently no data ablation experiments of any form exist — no filtering-strictness ablation, no caption-density/style ablation, no data-mixture ablation, and no corresponding evaluation-metric curves.
The only statement approaching "data impact" is the Hailuo 02 blog's juxtaposition of "3x parameters + 4x training data + improved quality and diversity" alongside the generational capability leap, but this is a product-iteration description with multiple variables changing simultaneously, from which the independent contribution of the data factor cannot be isolated; it does not constitute ablation evidence.

### Evidence on quality vs. quantity (cases where small, precise data beats large, messy data) ⚠️

[Uncertain] (no direct evidence, only directional wording). Hailuo 02's official wording, "training data expanded 4-fold, with improved quality and diversity," emphasizes both volume and quality simultaneously, which is a narrative of "bigger and better" rather than "small and precise surpassing large and messy," and therefore cannot serve as evidence that quality takes priority over quantity. MiniMax provides no comparative case of a small-scale, high-quality dataset surpassing a large-scale, low-quality dataset.
A weak piece of corroborating evidence is the positioning of the NCR architecture: MiniMax places the narrative weight of its generational improvement on architectural efficiency (2.5x training/inference efficiency) and compute cost-effectiveness rather than a data-scale race, indirectly reflecting a technical route that favors efficiency over stacking volume — but this pertains to compute efficiency rather than a data-quality argument.

### Alignment between training-data domain distribution and evaluation-benchmark taxonomy (e.g. VABench's seven major categories) ⚠️

[Uncertain] (no alignment relationship can be established). MiniMax has not released an accompanying evaluation benchmark, nor has it disclosed a domain-taxonomy for the training data, so there is no explicit alignment relationship between the training-data distribution and an evaluation-benchmark taxonomy (such as VABench's seven major categories) available for analysis.
The evaluation evidence cited officially is only a third-party leaderboard ranking: upon the launch of Hailuo 02, it was stated to rank second globally on the Artificial Analysis Video Arena (a third-party video-generation arena based on human pairwise-preference voting). That leaderboard is an overall preference Elo ranking with no fine-grained category breakdown, so no category alignment can be reverse-inferred from it either.
An observable product-side trace of categorization is the large number of template categories offered by Hailuo AI (dance videos, transformation effects, cinematic generation, etc.), which reflect the scenarios the product focuses on covering, and can be regarded as a de facto application taxonomy — but there is no publicly disclosed mapping relationship between this and the training data's domain mixture.

## Uncertain Fields

The research information for the following fields is partially uncertain (⚠️-marked sources):

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
- dialogue_transcription_attributes
- geometric_structured_annotation
- synthetic_data_synthesis
- human_in_loop
- audio_quality_filtering
- multi_stage_curriculum
- stage_data_mixture
- post_training_data
- data_infra_throughput
- data_ablation
- quality_over_quantity_evidence
- benchmark_taxonomy_alignment
</content>
