# Allegro (including the subsequent Allegro-TI2V image-to-video version, and the 40×720P / 40×360P variants; the companion captioning model is the same team's multimodal MoE model Aria)

> Topic: Data processing for video generation models (including joint audio-video generation): data cleaning pipeline, data distribution, annotation approach

[← Back to home](../index.md)

**Contents**: [Basic Information](#basic-information) · [Data Scale and Distribution](#data-scale-and-distribution) · [Cleaning Pipeline](#cleaning-pipeline) · [Captioning Approach](#captioning-approach) · [Audio-Video Alignment](#audio-video-alignment) · [Training Coordination](#training-coordination) · [Effectiveness Comparison](#effectiveness-comparison)

## Basic Information

### Name

Allegro (including the subsequent Allegro-TI2V image-to-video version, and the 40×720P / 40×360P variants; the companion captioning model is the same team's multimodal MoE model Aria)

### Releasing institution/company

Rhymes AI (a Singapore–Hong Kong-background AI startup, from the same team as the Aria multimodal model). Paper authors: Yuan Zhou, Qiuyue Wang, Yuxuan Cai, Huan Yang (Huan Yang is the corresponding/lead author, formerly a researcher at Microsoft Research Asia)

### Release date (technical report/paper/open-source date)

arXiv preprint v1 submitted October 20, 2024 (arXiv:2410.15458, "Allegro: Open the Black Box of Commercial-Level Video Generation Model"); Allegro T2V weights and inference code open-sourced on GitHub/Hugging Face on October 22, 2024 (Apache 2.0); Allegro-TI2V (text+image to video) released November 25, 2024; Allegro T2V training code open-sourced December 10, 2024; 40×720P / 40×360P variants released December 26, 2024; Allegro-TI2V training code open-sourced January 2, 2025

### Type (model/dataset/toolchain/evaluation benchmark)

Model (a Diffusion Transformer base model for text-to-video / image-to-video) + companion toolchain (in-house VideoVAE 175M, VideoDiT 2.8B, a video captioning model fine-tuned from Aria, complete training and inference code). The paper's core selling point is precisely "publicly opening the black box of a commercial-level video generation model," so the granularity at which its data-processing pipeline is disclosed is among the finest of concurrent open-source works

### Degree of openness (whether weights/code/data/pipeline are each open source)

Overall openness is relatively high, but the data itself is not open:
· Weights: fully open source, Apache 2.0 license (permits commercial use), Hugging Face rhymes-ai/Allegro, rhymes-ai/Allegro-TI2V, including a 175M VideoVAE + 2.8B VideoDiT.
· Code: both inference code and training code are open source (GitHub rhymes-ai/Allegro); training requires the user to supply their own dataset in .parquet format.
· Pipeline: disclosed at high granularity in the paper form — the tool name at every step of the 7-level filtering funnel (PySceneDetect / DOVER / LPIPS / UniMatch / LAION Aesthetics Predictor / CRAFT / Tag2Text / CLIP) and the item-by-item threshold table for every training stage (Table 1) are all disclosed, with the retained volume at each stage given. However, the cleaning code and threshold configuration scripts were not open-sourced alongside the weights.
· Captioning model: Aria (rhymes-ai/Aria, 25.3B total parameters / 3.9B activated, a native multimodal MoE, Apache 2.0, arXiv:2410.05993) is open source, but the version fine-tuned specifically for video captioning was not separately released.
· Data: the training dataset itself is not open source; it is only stated that it was constructed based on public datasets such as WebVid / Panda-70M / HD-VILA / HD-VG / OpenVid-1M.

### Whether joint audio-video generation is supported, and the implementation approach (native joint/cascade/MoE fusion)

Not supported. Allegro is a purely visual text-to-video/image-to-video model, producing silent video (88 frames, 720×1280, 15 FPS, about 6 seconds, upsamplable to 30 FPS via EMA-VFI frame interpolation). The entire data pipeline does not involve extraction, filtering, or annotation of any audio track, nor is there a cascaded V2A module or an audio expert branch. All audio-related dimensions in this entry are not applicable.

### List of research information sources (URLs of papers/technical reports/official documentation/news, with each source's nature annotated: official first-hand/same-team corroboration/third-party report)

1. https://arxiv.org/abs/2410.15458 — official first-hand, the Allegro paper "Allegro: Open the Black Box of Commercial-Level Video Generation Model," 2024-10-20 v1; the data section (Sec. 2 Data: Data Filtering / Data Annotation / Data Stratification, the Table 1 threshold table, and Appendix A's data distribution figures Fig.11) is the primary source for all quantitative information in this entry.
2. https://arxiv.org/html/2410.15458v1 — official first-hand, full HTML text of the paper, including Table 1 (per-stage filtering thresholds), the training-stage configuration table (resolution/frame count/sample count/batch size/step count/GPU count), and Appendix A's distribution statistics.
3. https://github.com/rhymes-ai/Allegro — official first-hand, code repository and README, including the Apache 2.0 license, release timeline for each version, model specifications, and training code.
4. https://huggingface.co/rhymes-ai/Allegro — official first-hand, T2V weights card.
5. https://huggingface.co/rhymes-ai/Allegro-TI2V — official first-hand, TI2V weights card (2024-11-25).
6. https://huggingface.co/blog/RhymesAI/allegro — official first-hand, Rhymes AI's official blog post "Allegro: Advanced Video Generation Model."
7. https://arxiv.org/abs/2410.05993 — same-team corroboration, the Aria multimodal MoE model paper (the base model for the captioning model, 25.3B total parameters / 3.9B activated per visual token, supporting 64K multimodal input, captioning a 256-frame video within 10 seconds).
8. https://huggingface.co/papers/2410.15458 — third-party aggregation, Hugging Face paper page and community discussion.
9. https://www.marktechpost.com/2024/11/28/rhymes-ai-unveils-allegro-ti2v-a-breakthrough-in-visual-storytelling-with-open-source-ai-video-generation-technology/ — third-party report, coverage of the Allegro-TI2V release.

## Data Scale and Distribution

### Training data magnitude (video count/hour count/token count, pretraining and SFT listed separately)

Final curated training set: 106 million (106M) images + 48 million (48M) video clips, all with strongly associated text captions.
Original input scale: 412 million (412M) raw images + 500 million (500M) raw videos (here "500M" refers to the base count of video clips after splitting and initial filtering).
Available data volume by stage (the same data layered progressively by threshold strictness, narrowing at each layer, not independently collected):
· T2I pretraining: 107M images (filtered from 412M raw images)
· T2V pretraining, 360p: 48M video clips (from 500M)
· T2V pretraining, 720p: 18M video clips (from 500M)
· T2V fine-tuning (SFT): about 2M high-quality video clips (from 500M)
VideoVAE's standalone training set: 54.7K videos + 3.73M images (requiring short side ≥720 pixels).
Actual training samples consumed (including repeated sampling, distinct from the dataset item count): T2I pretraining 700M samples; T2V Pre-train-1 (368×640, 40 frames) 87M; Pre-train-2 (720×1280, 40 frames) 21M; Pre-train-3 (720×1280, 88 frames) 8M; Fine-tune 2.6M.

### Data source composition (proprietary/public datasets/web crawling/licensed acquisition/synthetic data) ⚠️

Primarily built on public datasets; the paper explicitly states that "existing public datasets such as WebVid, Panda-70M, HD-VILA, HD-VG, and OpenVid-1M provide a solid foundation for the data sources." Building on this, the team carried out its own complete re-splitting, filtering, and re-captioning to construct its own corpus of 106M images + 48M videos, rather than directly using the original datasets' original captions and splits.
No disclosure of the use of proprietary acquisition/licensed data, exclusive copyrighted libraries, or synthetic data; the specific mixture ratio across source datasets is also not given. The source of the 412M raw images on the image side is not named. [Uncertain] (the proportion from each source, and whether proprietary web-crawled data was mixed in)

### Data compliance and provenance (proportion of licensed data, rights-cleared datasets, C2PA, etc.) ⚠️

The paper contains no section on data compliance and provenance; it does not disclose the proportion of licensed data, the proportion of rights-cleared datasets, C2PA/content credentials, watermark provenance, or copyright review processes. The public datasets it relies on (WebVid, Panda-70M, HD-VILA, etc.) are themselves mostly under academic research licenses sourced from web video platforms; their commercial-use compliance is debatable but not discussed in the paper. The model weights are open-sourced under Apache 2.0, but the training data is not released and no source list is provided. [Uncertain]

### Clip duration distribution and splitting strategy

Splitting and duration constraints form the first gate of the funnel:
· Raw videos are first filtered to remove samples with duration <2 seconds or frame rate <23 FPS;
· After PySceneDetect scene splitting, only single-scene clips of 2–16 seconds are retained (clip-level duration bounds);
· The late 720p-pretraining stage and the fine-tuning stage further tighten this to 6–16 seconds (Table 1: T2V Pre-train 720p uses two tiers, [2s,16s] and [6s,16s]; Fine-tune uses [6s,16s]).
Distribution (Appendix A Fig.11): during the pretraining stages, the distribution across the three buckets of 2–6 seconds / 6–10 seconds / 10–16 seconds is relatively balanced; during the fine-tuning stage, the 6-second lower bound shifts the overall distribution rightward, with the share of longer clips significantly increased to match the 88-frame generation target (about 6 seconds @15FPS).
All clips are finally transcoded uniformly to H.264 at a fixed 30 FPS.

### Resolution/aspect-ratio distribution and bucketing strategy ⚠️

Resolution thresholds layered by training stage (Table 1):
· T2I pretraining and T2V 360p pretraining: width W≥640 and height H≥368 (corresponding to the 368×640 training resolution);
· T2V 720p pretraining and fine-tuning: width W≥1280 and height H≥720 (corresponding to the 720×1280 training resolution);
· The raw entry gate additionally requires a minimum of ≥360p.
Training resolution is not multi-bucket dynamic-resolution training but is fixed at two tiers (368×640 → 720×1280), following a "low resolution → high resolution" progressive curriculum. On aspect ratio, the paper does not disclose a bucketing strategy or multi-aspect-ratio training; training resolutions are all 720×1280 / 368×640 expressed as a portrait 16:9 layout. [Uncertain] (aspect-ratio distribution and bucketing strategy)

### Category/domain distribution and mixture strategy (proportional control and concept balancing for people, actions, scenes, styles, etc.)

In Appendix A, the paper gives category-distribution statistics based on coarse-grained Tag2Text labels (Fig.11): the three categories people, objects, and landscapes/natural scenery make up the vast majority of labels; the paper attributes this to "natural dataset composition," i.e., the distribution is determined by the source corpus rather than by deliberate mixture design.
Notably, Allegro's "stratification" is stratification by quality threshold (the same corpus is cut into 4 nested subsets from loose to strict by aesthetic/CLIP/sharpness thresholds, supplying the 4 training stages), rather than mixture balancing by content domain. The paper describes no concept balancing, no upsampling of long-tail categories, no proportional control by people/action/scene/style, and no category-resampling weight table. This is a clear gap in its pipeline relative to works like Movie Gen and Seedance.
There is one implicit constraint on content style: only single-shot material is retained, so multi-shot narrative-type domains are not included.

### Audio category distribution and mixture (proportion and control strategy for speech/sound effects/music/ambient sound/silence) — a dimension unique to AV models

Not applicable. Allegro is a purely visual video generation model; its training data does not retain or process audio tracks, and the paper makes no mention of the classification or mixture of speech/sound effects/music/ambient sound/silence.

### Narrative structure distribution (single-shot vs. multi-shot, average clip duration, shot-count distribution, presence of native audio track)

All clips are single-shot, with no multi-shot narrative structure: raw long videos are split by PySceneDetect into "single-scene clips" before entering the training set; the paper explicitly states the training material is single-shot camera footage.
Average clip duration falls in the 2–16 second range (6–16 seconds for the fine-tuning stage); training uniformly samples 40 or 88 frames (88 frames @15FPS ≈ 6 seconds).
No native audio track (audio is discarded).
Shot-count distribution is not applicable (always 1); the paper does not construct multi-shot/storyboard-type data.

### Language/accent distribution (data foundation for multilingual lip-sync capability) ⚠️

Not applicable to lip sync (no audio). On the text side, T5 (not mT5) is used as the text encoder, max 512 tokens; the paper argues qualitatively (Fig.7–8) that T5 outperforms mT5 for this task, so the training captions are effectively English monolingual; no multilingual caption or language-distribution statistics are disclosed. [Uncertain] (quantitative distribution of caption language composition)

## Cleaning Pipeline

### Overall structure of the cleaning funnel (number of levels, ordering)

The paper discloses a complete pipeline of a 7-level serial filtering funnel + fine-grained annotation + quality stratification, one of the finest-grained disclosures among concurrent open-source works:
【Level 1】Duration/frame-rate/resolution prescreen: remove raw videos with duration <2s, FPS <23, or resolution <360p.
【Level 2】Scene splitting: use PySceneDetect to split long videos into single-scene clips, retaining only clips of 2–16 seconds; additionally discard the first and last 10 frames of each clip to eliminate false positives/transition residue from scene detection.
【Level 3】Low-level metric filtering: brightness (grayscale-based assessment, removing over-dark/over-exposed clips), sharpness (DOVER video quality score), semantic/temporal consistency (LPIPS inter-frame perceptual distance), motion magnitude (UniMatch optical flow).
【Level 4】Aesthetic filtering: LAION Aesthetics Predictor, scoring both images and video middle frames.
【Level 5】Content-irrelevant artifact filtering: CRAFT text detection + watermark detector, removing samples where black bars, text, or watermarks exceed an area-ratio threshold.
【Level 6】Coarse-grained labeling: Tag2Text generates tag-style captions for middle frames, providing preliminary semantic information (also serving as the basis for later category statistics).
【Level 7】CLIP similarity filtering: compute the CLIP cosine similarity between the visual embedding and the Level 6 caption, removing samples with weak image-text association.
【Annotation】Samples that pass the funnel are then given fine-grained spatiotemporal captions by the fine-tuned Aria.
【Stratification】The same corpus is cut into 4 nested subsets by threshold strictness, fed respectively to the four stages: T2I pretraining / T2V 360p pretraining / T2V 720p pretraining / T2V fine-tuning.
Key design point: placing "cheap coarse filters (duration/resolution/splitting)" first and "expensive model inference (aesthetics, DOVER, CLIP, Aria captioning)" last, and placing caption generation before CLIP filtering — using the coarse caption as the basis for image-text consistency judgment, avoiding running an expensive fine-grained VLM over the full dataset.

### Quantitative funnel retention rate (input/output volume at each level and final retention rate, e.g. Apollo's 27%) ⚠️

The paper gives fairly rare stage-by-stage quantitative retention rates (with the original scale as denominator):
· Images: 412M raw images → 107M (about 26% retention), used for T2I pretraining.
· Video (360p tier, the loosest threshold): 500M raw clips → 48M (retention rate 9.6%).
· Video (720p tier): 500M → 18M (retention rate 3.6%).
· Video (fine-tuning tier, strictest threshold): 500M → about 2M (retention rate 0.4%).
That is, from loosest to strictest, video retention rates span 9.6% → 3.6% → 0.4%, with the final high-quality curated set amounting to only four-thousandths of the original volume.
Limitation: the paper only gives the "end-to-end retention rate of the whole funnel," without disclosing the input/output volume of each individual filter step (e.g., how many survived after PySceneDetect splitting, how many the aesthetic filter eliminated), so it is not possible to pinpoint which level contributed the bulk of the elimination. [Partially uncertain: per-filter, level-by-level retention rates]

### Shot-splitting method (PySceneDetect/in-house model/shot-aware splitting)

Uses the open-source tool PySceneDetect for shot/scene splitting, breaking raw long videos into single-scene clips, retaining only clips of 2–16 seconds after splitting.
A notable engineering detail: after splitting, the first and last 10 frames of each clip are discarded, used to eliminate false positives from scene detection (fade transitions, residual frames at cut points) that would otherwise contaminate clips across shot boundaries.
No in-house shot-aware splitting model is used, nor is transition-type classification or multi-shot reassembly performed.

### Quality filtering (aesthetic scoring, sharpness, OCR text filtering, black-bar/watermark/logo detection)

Quality filtering is split into three orthogonal dimensions, each with per-stage thresholds given (Table 1):
【Brightness】Convert frames to grayscale to assess average brightness, retaining the range [20, 180] (on a 0–255 scale), consistent across all four stages, used to remove fully black/overexposed/solid-color frames.
【Sharpness】DOVER (Disentangled Objective Video Quality Evaluator) video-quality score, enabled only at the fine-tuning stage, threshold ≥0.07; no such gate is set at the pretraining stages.
【Semantic/temporal consistency】LPIPS inter-frame perceptual distance, enabled only at the fine-tuning stage, threshold ≥0.05 (lower bound, used to remove samples that are nearly completely static/repeated frames).
【Aesthetics】LAION Aesthetics Predictor scoring images and video middle frames, the only metric applied across all four stages and progressively tightened: T2I pretraining ≥4.8 → T2V 360p pretraining ≥4.8 → T2V 720p pretraining ≥5.0 → fine-tuning ≥5.3.
【Text/watermark/black bars】CRAFT scene-text detector + watermark detector, text-area-ratio threshold ≤0.05% (consistent across all four stages, extremely strict, excluding almost any material with subtitles/logos/decorative text); black borders and watermarks are grouped together as "content-irrelevant artifacts" and removed together.
【Image-text consistency】CLIP visual-text cosine similarity, ≥0.17 at the 360p pretraining/T2I stages, ≥0.20 at the 720p pretraining and fine-tuning stages.
Appendix A Fig.11 shows the aesthetic-score distribution shifting rightward overall as stages progress (360p pretraining → 720p pretraining → fine-tuning), verifying the actual effect of the progressive tightening.

### Motion filtering (optical-flow/motion-score thresholds, removal of static and jittery content)

Uses the UniMatch (Xu et al., 2023) optical-flow model to estimate motion magnitude, enabled only at the T2V fine-tuning stage, retaining the range [1.0, 100]: the lower bound of 1.0 removes near-static footage (still images, slideshow-style material), the upper bound of 100 removes footage with excessively intense motion/jitter/rapid cuts. The three pretraining stages set no motion gate at all (blank in Table 1), indicating the team treats motion quality as a "late-stage refinement" rather than a "pretraining admission" condition.
In addition, the LPIPS ≥0.05 lower bound complements motion filtering, likewise used to exclude samples with almost no inter-frame change.
No camera-motion classifier or dedicated jitter-detection model is used.

### Deduplication method (exact deduplication and embedding-based semantic deduplication recorded separately) ⚠️

The paper describes no deduplication step at all — neither exact deduplication (hashing/pHash) nor embedding-based semantic deduplication or near-duplicate clustering. This is a clear gap in the pipeline given its otherwise fine disclosure granularity, considering the data sources include public datasets with mutual overlap such as WebVid, Panda-70M, HD-VILA, HD-VG, and OpenVid-1M — the risk of cross-dataset duplication genuinely exists but is not discussed in the paper. [Uncertain]

### VLM/LLM as quality inspector (multimodal-large-model quality scoring and mismatch removal; the 2026 trend of shifting from shallow scorers to large-model semantic judgment)

Falls into the typical 2024 form of "shallow scorers doing the bulk of the work, with large models used only for labeling, not for judgment" — not yet at the VLM-as-judge stage:
· The models used as judgment criteria are all dedicated small models/scorers: LAION Aesthetics Predictor (aesthetics), DOVER (video quality), LPIPS (perceptual consistency), UniMatch (optical flow), CRAFT (text detection), CLIP (image-text consistency).
· The multimodal large model's role is "generator" rather than "judge": Tag2Text handles coarse-grained tags, Aria (25.3B MoE) handles fine-grained captions; Aria is never used to score video quality, semantically judge image-text mismatch, or output structured quality-dimension scores.
· The only filter with any semantic character is the Level 7 CLIP similarity threshold (≥0.17 / ≥0.20), which is still fundamentally an embedding cosine distance rather than large-model semantic judgment.
So by 2026 standards, Allegro belongs to the previous generation of the "model quality inspection" paradigm.

### Safety and compliance filtering (NSFW, copyright, face/privacy) ⚠️

The paper discloses no NSFW/violence/copyright/facial-privacy safety-compliance filtering step whatsoever; the Table 1 threshold table has no corresponding metric either. The team only mentions compliant use at the level of the model license (Apache 2.0) and terms of use; the data-side safety filtering strategy is entirely absent from the public description. Given that the corpus comes from public datasets such as WebVid/HD-VILA, it may partially inherit filtering from the original datasets, but the paper makes no such statement. [Uncertain]

## Captioning Approach

### Caption models used (in-house VLM/open-source model, model scale)

Two-tier labeling with a clear division of labor:
【Coarse-grained】Tag2Text (Huang et al., 2023): serves as Level 6 of the filtering funnel, generating tag-style descriptions for video middle frames, providing preliminary semantic information; its output is used directly as the text side of the Level 7 CLIP similarity filter, and also serves as the basis for the category-distribution statistics (people / objects / landscapes). It was chosen for its lightweight nature, being runnable at full scale over 500M-tier clips.
【Fine-grained】A version of Aria (Li et al., 2024b, also developed in-house by Rhymes AI) fine-tuned for video captioning: Aria is an open-source native multimodal Mixture-of-Experts model with 25.3B total parameters, 3.9B activated per visual token (3.5B activated per text token), supporting up to 64K multimodal input, and officially claimed to be able to caption a 256-frame video within 10 seconds — this throughput characteristic is exactly why it was chosen as the large-scale video captioner. The base model has been open-sourced under Apache 2.0 (rhymes-ai/Aria), but the video-captioning fine-tuned weights were not separately released.
【Text encoder】T5 (not mT5) is used during training, max 512 tokens; the paper argues qualitatively (Fig.7–8) that T5 outperforms mT5 for this task.

### Caption density and degree of structuring (short/long/dense description, structured fields such as camera motion, style tags) ⚠️

Takes the form of "long dense description + semi-structured embedded fields":
· Coverage dimensions (as listed in the paper's original text): subject attributes, interactions among the subjects, background description, environment, style, atmosphere, camera angle and motion, and temporal changes — i.e., covering both spatial and temporal information simultaneously.
· Composition method: video caption = a static-frame caption of the middle frame + a dynamic caption of the whole video, concatenated together, balancing single-frame detail fidelity with whole-clip dynamic narration.
· Structured fields: camera motion is explicitly highlighted with a fixed sentence pattern, in the form of sentences like "Camera [MOTION_PATTERN]" inserted into the caption, letting the model learn a controllable response to camera-motion instructions; other dimensions are expressed as natural-language prose, not broken out into independent JSON fields.
· Dual granularity coexists: every sample retains both the coarse-grained (Tag2Text global semantic tags) and fine-grained (Aria spatiotemporal detail) annotation sets, the former mainly used for filtering and statistics, the latter for training.
· Caption length is constrained by the T5 encoder's 512-token limit.
· No disclosure of caption length distribution or caption diversity rewriting (recaptioning/prompt upsampling) strategy. [Partially uncertain: length distribution and short/long caption mixed-training ratio]

### Joint audio-video caption structure (whether both visual and auditory tracks are covered, whether split into independent fields, e.g. LTX-2's full soundscape description, Script-a-Video's factorized streams, Foley-Omni's three fields)

Not applicable. Allegro has no audio modality; captions cover only the visual track (including camera motion); there is no joint audio-video caption structure, independent auditory field, or full-soundscape description.

### Dialogue transcription and speaker attribute annotation (ASR transcription + speaker identity/language/accent/emotion)

Not applicable. No audio-processing pipeline exists, so no ASR transcription or speaker identity/language/accent/emotion annotation is performed.

### Geometric and structured annotation (camera parameters, depth, 3D point tracks, action annotation, explicit state)

Only lightweight semantic camera-motion annotation exists: captions use the explicit sentence pattern "Camera [MOTION_PATTERN]" to label camera-motion patterns (a semantic description generated by the fine-tuned Aria, not numeric parameters).
No camera intrinsics/extrinsics estimation, depth maps, 3D point tracks, action/skeletal annotation, or explicit object-state annotation is performed. Motion information is reflected on the filtering side only as UniMatch's scalar optical-flow magnitude ([1.0, 100]), and is not retained as a dense optical-flow condition or structured field.

### Synthetic data construction (controlled perturbation/editing to construct training pairs, e.g. InstructAV2AV)

The paper does not use synthetic data construction for training pairs; no controlled perturbation/editing construction is performed (e.g., InstructAV2AV-style synthetic paired data). All data consists of real videos obtained through splitting and filtering. The only "generative" step is that captions are generated by models (Tag2Text/Aria), which counts as synthetic annotation, not synthetic video.

### Degree of human involvement (human annotation, human quality inspection, model-prescreen + human review)

The data pipeline is fully automated, with no human annotation or human quality-inspection step: from splitting, the 7-level filtering, to caption generation, everything is done by tools and models; the paper mentions no human review, spot-checking, or annotation team at any point.
Human involvement only appears in the evaluation stage: the user study uses 46 prompts covering diverse scenarios and six evaluation dimensions (video-text relevance, appearance distortion, appearance aesthetics, motion naturalness, motion magnitude, overall quality), with each video pair rated by 2 annotators, totaling 5,448 ratings. In addition, VBench quantitative evaluation uses 946 prompts to generate 4,730 videos.

## Audio-Video Alignment

### Audio-video synchronization detection methods (lip sync, event alignment)

Not applicable. No audio modality exists, so there is no audio-video synchronization detection (lip sync or event alignment) step.

### Specific sync-detection metrics and thresholds (SyncNet/Synchformer/LSE/in-house, threshold values, e.g. MOVA's LSE-D≤9.5 ∧ LSE-C≥4.5, SkyReels-V4's SyncNet |offset|≤3 ∧ conf>1.5)

Not applicable. No SyncNet, AV-align, or any in-house sync metric is used, and there are no corresponding thresholds.

### Separate handling of temporal sync vs. semantic sync (temporal alignment and content-semantic matching treated as two independent filtering conditions)

Not applicable (no audio). If analogized to the purely visual side, the design in Allegro closest to this idea is splitting "temporal consistency" (LPIPS ≥0.05, UniMatch [1.0,100]) and "semantic matching" (CLIP similarity ≥0.17/0.20) into two independent filtering conditions applied at different pipeline levels — this approach of separating "temporal quality" filtering from "image-text semantic consistency" filtering is methodologically analogous to the temporal-sync/semantic-sync separation seen in AV models.

### Audio quality filtering (SNR, silence detection and silence-ratio thresholds, removal of no-audio-track clips, removal of off-screen sound sources, background-music separation)

Not applicable. The audio track of the raw video is discarded at the splitting stage; there is no SNR, silence detection, no-audio-track removal, off-screen sound source removal, or background-music separation.

### Classification and separate handling strategies for speech/sound effects/music

Not applicable. There is no classification or separate handling strategy for speech/sound effects/music.

## Training Coordination

### Multi-stage training curriculum and data-curriculum scheduling (basis for stage division: resolution/duration/quality score/modality; low-res→high-res, image→video, short→long)

A progressive curriculum of four (five including T2I) stages, whose scheduling basis simultaneously spans four axes — modality, resolution, duration/frame count, and quality score — the core of this work's data-stratification design:
· Stage 0 — T2I pretraining: 368×640, single frame, training samples 700M (dataset 107M images), batch 4096, 170K steps, 128× H100. Uses images to establish visual priors and text-image alignment.
· Stage 1 — T2V pretraining, 360p: 368×640, 40 frames, training samples 87M (dataset 48M videos), batch 1024, 85K steps, 256× H100. Image→video modality transfer, low resolution with large data volume.
· Stage 2 — T2V pretraining, 720p: 720×1280, 40 frames, training samples 21M (dataset 18M), batch 512, 41K steps, 256× H100. Low resolution → high resolution.
· Stage 3 — T2V pretraining, long frames: 720×1280, 88 frames, training samples 8M, batch 256, 31K steps, 256× H100. Short → long (40 frames → 88 frames).
· Stage 4 — T2V fine-tuning: 720×1280, 88 frames, training samples 2.6M (dataset about 2M), batch 256, 10K steps, 256× H100. Uses only the strictest-threshold curated set.
It can be seen that resolution, duration, and quality threshold all tighten in the same direction simultaneously, while data volume drops by orders of magnitude at each stage (107M→48M→18M→about 2M), with batch size correspondingly dropping from 4096 to 256. Paper Fig.9 shows the incremental benefit of multi-stage training via qualitative visualization.

### How data mixture changes across training stages (pretraining/annealing/SFT high-quality subset)

Allegro's "mixture" is not a weighted combination of multiple sources, but a nested quality stratification of a single corpus: the same batch of data, after 7-level filtering, is cut into 4 nested subsets from loose to strict by threshold strictness, and each training stage uses only the full set of its corresponding subset, with no cross-subset proportional mixing, nor any described mixing ratio between image and video data at the T2V stages.
Full comparison of how thresholds change by stage (Table 1):
· Duration: — / [2s,16s] / [2s,16s] and [6s,16s] / [6s,16s]
· Resolution: W≥640,H≥368 / same / W≥1280,H≥720 / same as 720p
· Brightness: [20,180] constant across all four stages
· DOVER sharpness: — / — / — / ≥0.07 (fine-tuning only)
· LPIPS: — / — / — / ≥0.05 (fine-tuning only)
· UniMatch motion: — / — / — / [1.0,100] (fine-tuning only)
· Aesthetic score: ≥4.8 / ≥4.8 / ≥5.0 / ≥5.3
· Text-area ratio: ≤0.05% constant across all four stages
· CLIP similarity: ≥0.17 / ≥0.17 / ≥0.20 / ≥0.20
That is, the pretraining stages rely mainly on the three gauges "aesthetics + resolution + image-text consistency," while the fine-tuning stage additionally stacks on three more expensive gauges — "sharpness + temporal consistency + motion magnitude" — a clear cost-quality trade-off design: expensive metrics are only computed after the data volume has already shrunk by two orders of magnitude.

### Post-training data (scale and selection criteria of the SFT curated set, number of preference pairs and annotation method, reward-model training data) ⚠️

Post-training is the T2V fine-tuning stage (SFT), using about 2M of the strictest-threshold curated videos (0.4% of the 500M raw clips), with selection criteria of: duration 6–16 seconds, resolution ≥1280×720, brightness [20,180], DOVER ≥0.07, LPIPS ≥0.05, UniMatch motion [1.0,100], aesthetics ≥5.3, text area ≤0.05%, CLIP similarity ≥0.20; training consumed 2.6M samples, batch 256, 10K steps, 256× H100.
No preference-alignment post-training is used: the paper does not involve DPO/RLHF, preference-pair data, or reward-model training data, nor is there any human preference-annotation set. [Uncertain] (whether any undisclosed preference data exists)

### Data-processing infrastructure and throughput (NeMo Curator/Data-Juicer/in-house, GPU acceleration ratio, processing scale, cost) ⚠️

The paper discloses no data-processing infrastructure details: it does not state whether NeMo Curator, Data-Juicer, or an in-house distributed cleaning framework is used, and gives no GPU acceleration ratio or the machine-time/cost consumed to clean 500M clips. An inferable order-of-magnitude clue is that PySceneDetect, DOVER, LPIPS, UniMatch, LAION Aesthetics, CRAFT, Tag2Text, and CLIP — 8 categories of model inference in total — must be run over 500M video clips and 412M images, and the 25.3B MoE Aria captioning model must be run over the samples that pass the funnel (officially claimed to caption a 256-frame video in 10 seconds) — placing Aria at the end of the funnel is precisely to control this cost.
Training-side infrastructure is disclosed: the entire pipeline uses 128–256 H100 GPUs, with a total of about 337K training steps. [Uncertain] (framework, throughput, and cost on the data-cleaning side)

## Effectiveness Comparison

### Quantified impact of data-strategy ablations (distinguishing: filtering-strictness ablation / caption-density-style ablation / data-mixture ablation, and corresponding evaluation metrics)

Quantitative data-strategy ablation is largely absent, the main shortcoming in this paper's disclosure granularity:
· Filtering-strictness ablation: none. The paper gives 4 tiers of threshold configuration, but does not perform a "how does loosening/tightening thresholds affect the final metrics" controlled experiment, nor does it report the VBench change from removing any single filter level.
· Caption-density/style ablation: none. No comparison of short caption vs. long dense caption, or Tag2Text caption vs. Aria caption, on training effectiveness.
· Data-mixture ablation: none.
· The comparisons that are done are all qualitative visualizations: T5 vs. mT5 text-encoder comparison (Fig.7–8, qualitative examples), qualitative visualization of the incremental effect of multi-stage progressive training (Fig.9, no isolated metric).
· End-to-end evaluation: quantitative VBench evaluation (946 prompts / 4,730 videos) and a user study (46 prompts, 6 dimensions, 5,448 ratings, 2 annotators per pair), with the conclusion that it surpasses all then-existing open-source models and most commercial models, ranking behind only Hailuo (MiniMax) and Kling.
So the effectiveness of its data design is mainly backed indirectly via "final model ranking," lacking quantified evidence attributable to any single data decision.

### Evidence on quality vs. quantity (cases where small, precise data beats large, messy data) ⚠️

Structural evidence exists but without an isolated experiment: the whole pipeline's design philosophy is "trade extremely low retention rate for quality" — video drops from 500M down to about 2M at the fine-tuning stage (0.4% retention rate), and the final 10K-step fine-tuning stage uses only this 2M curated set, i.e., completing the final quality shaping using less than four-thousandths of the data; Appendix A Fig.11 shows the aesthetic-score distribution shifting rightward as stages progress, corroborating that the curated set's quality does indeed rise progressively. The paper's abstract and introduction also explicitly list "striking a balance between data volume and quality" as key to achieving commercial-level results.
But the paper provides no "small and precise vs. large and messy" controlled experiment (e.g., using a loose-threshold dataset fine-tuned for the same number of steps as a baseline), so this evidence is design-level indirect evidence, not an empirical conclusion. [Partially uncertain]

### Alignment between training-data domain distribution and evaluation-benchmark taxonomy (e.g. VABench's seven major categories) ⚠️

No explicit alignment is established between the training-data domain distribution and any evaluation-benchmark taxonomy. Evaluation uses the general-purpose VBench (946 prompts, 4,730 generated videos) and a self-built 46-prompt user study (six dimensions: video-text relevance, appearance distortion, appearance aesthetics, motion naturalness, motion magnitude, overall quality).
There is no mapping or dimension-targeted data supplementation between the training-side category statistics (dominated by people / objects / landscapes under Tag2Text labels) and VBench's sixteen-dimension taxonomy; the paper explicitly attributes the category distribution to natural dataset composition rather than deliberate alignment design. There is a naive correspondence between the user study's six dimensions and the data-filtering metrics (aesthetics ↔ LAION Aesthetics, motion magnitude ↔ UniMatch, appearance distortion ↔ DOVER), but the paper does not formalize this correspondence as a design principle. [Partially uncertain]

## Uncertain Fields

The research information for the following fields is partially uncertain (⚠️-marked sources):

- data_sources
- provenance_licensing
- resolution_aspect_distribution
- language_accent_distribution
- funnel_retention_rate
- deduplication
- safety_filtering
- caption_structure
- post_training_data
- data_infra_throughput
- quality_over_quantity_evidence
- benchmark_taxonomy_alignment
</content>
