# Step-Video-T2V (StepFun Video, 30B text-to-video foundation model) and its derivatives Step-Video-T2V-Turbo, Step-Video-TI2V

> Topic: Data processing for video generation models (including simultaneous audio-video generation): data cleaning pipeline, data distribution, annotation methods

[← Back to home](../index.md)

**Table of Contents**: [Basic Information](#basic-information) · [Data Scale and Distribution](#data-scale-and-distribution) · [Cleaning Pipeline](#cleaning-pipeline) · [Annotation Methods](#annotation-methods) · [Audio-Video Alignment](#audio-video-alignment) · [Training Integration](#training-integration) · [Performance Comparison](#performance-comparison)

## Basic Information

### Name

Step-Video-T2V (StepFun Video, 30B text-to-video foundation model) and its derivatives Step-Video-T2V-Turbo, Step-Video-TI2V

### Publishing Organization/Company

StepFun (阶跃星辰, Step-Video Team / Shanghai StepFun Intelligent Technology Co., Ltd.)

### Release Date (technical report/paper/open-source date)

Inference code and weights were open-sourced simultaneously on February 17, 2025; the technical report "Step-Video-T2V Technical Report: The Practice, Challenges, and Future of Video Foundation Model" was published the same day (arXiv:2502.10248, later revised as v2/v3). Official public announcement came on February 18, 2025 (open-sourced in the same batch as the speech model Step-Audio). The derivative model Step-Video-TI2V (image-to-video) was open-sourced on March 17, 2025, with technical report arXiv:2503.11251.

### Type (model/dataset/toolchain/evaluation benchmark)

Model (an open-source text-to-video foundation model, a 30B-parameter DiT + an in-house deeply compressed Video-VAE), accompanied by the evaluation benchmark Step-Video-T2V-Eval (128 Chinese prompts across 11 categories) and open-source inference code. Not a dataset, not a data toolchain — neither the training data nor the data-processing code has been released.

### Degree of openness (whether weights/code/data/pipeline are each open-source)

Falls into a "weights + code + evaluation benchmark open, data and data-pipeline code not open" pattern, but its degree of openness is relatively high among contemporaneous Chinese domestic models:
[Weights] Open source. Step-Video-T2V (30B) and its distilled acceleration version Step-Video-T2V-Turbo weights are both released on GitHub (stepfun-ai/Step-Video-T2V), Hugging Face (stepfun-ai/stepvideo-t2v, stepvideo-t2v-turbo), and ModelScope; the in-house Video-VAE (16×16 spatial, 8× temporal compression) and bilingual text encoders are released together.
[Code] Open-source inference code (including multi-GPU parallel inference, xDiT/ComfyUI integration); training code is not released.
[License] MIT License (notably more permissive than Tencent Hunyuan's community agreement, Tongyi Wanxiang's custom agreement, etc. — a highlight of its openness).
[Evaluation benchmark] Step-Video-T2V-Eval is open-sourced, containing 128 prompts along with generated result videos from multiple open- and closed-source engines, allowing direct reproduction of comparisons.
[Data] Not open. Both the 2B video-text pairs and 3.8B image-text pairs are internal data; no subset or manifest has been released.
[Pipeline] Methodology disclosure is relatively complete (a six-stage pipeline, with the specific open-source tools and model names used at nearly every stage named — e.g., PySceneDetect AdaptiveDetector, the LAION aesthetic predictor, the LAION NSFW detector, an EfficientNet watermark classifier, PaddleOCR, Laplacian variance, FFmpeg cropdetect, Farneback optical flow, an in-house VideoCLIP), but the specific filtering threshold values, the in-house caption VLM, and the VideoCLIP weights, along with the pipeline code, are all undisclosed, so it cannot be directly reproduced.

### Whether it supports simultaneous audio-video generation, and how it is implemented (native joint/cascaded/MoE fusion)

Not supported. Both Step-Video-T2V and Step-Video-TI2V are purely visual video-generation models with no audio track in their output; the technical report does not address the audio modality anywhere, and the data pipeline contains no audio-track processing step at all (only visual frames are retained after segmentation).
StepFun's audio capability is handled by a completely independent model line: Step-Audio, a production-grade open-source speech-interaction model that supports emotion, dialect, language, singing, and personalized style, open-sourced the same day as Step-Video-T2V, along with subsequent Step-Audio series models. There is no joint training, no shared latent, and no joint denoising between the two — a typical "video model + speech model" dual-track parallel arrangement, not cascaded or natively joint AV generation.
Consequently, this entry does not constitute a reference sample on the audio-video joint generation dimension; all audio-related fields in this research round (audio_category_distribution, joint_av_caption_schema, dialogue_transcription_attributes, av_sync_detection, sync_metric_and_threshold, temporal_vs_semantic_sync, audio_quality_filtering, audio_type_handling) are not applicable.

### List of research information sources (URLs of papers/technical reports/official documents/news, each labeled by source type: official primary/same-team corroborating/third-party report)

- [Official primary] Step-Video-T2V Technical Report: The Practice, Challenges, and Future of Video Foundation Model, Step-Video Team (StepFun), arXiv:2502.10248, 2025-02 (includes Section 7 Data: the six-stage pipeline of video segmentation/quality assessment/motion assessment/captioning/concept balancing/video-text alignment, the Figure 11 hierarchical-filtering illustration, and the training-stage configuration table): https://arxiv.org/abs/2502.10248
- [Official primary] Step-Video-T2V technical report, full ar5iv text (the data section is searchable, containing the 7 quality-label types, the Farneback optical-flow triple metrics, the 120,000+ K-means clusters, the 8-frame CLIP Score, the 30M SFT figure, the 95k distillation samples, the 644M/27.3M seen-samples figures, and other values): https://ar5iv.labs.arxiv.org/html/2502.10248
- [Official primary] Step-Video-T2V technical report, arXiv HTML v1: https://arxiv.org/html/2502.10248v1
- [Official primary] Step-Video-T2V technical report, arXiv HTML v2 (includes hierarchical filtering and the Figure 11 explanation, Step-Video-T2V-Eval's 128 prompts/11 categories, and the Video-DPO human-preference annotation workflow): https://arxiv.org/html/2502.10248v2
- [Official primary] Step-Video-T2V GitHub repository (weights, inference code, Video-VAE, bilingual text encoders, MIT license, released 2025-02-17): https://github.com/stepfun-ai/Step-Video-T2V
- [Official primary] Step-Video-T2V Hugging Face model card: https://huggingface.co/stepfun-ai/stepvideo-t2v
- [Official primary] Step-Video-T2V-Eval evaluation benchmark dataset (128 Chinese prompts, 11 categories, including multi-engine comparison generation results): https://huggingface.co/datasets/stepfun-ai/Step-Video-T2V-Eval
- [Same-team corroborating] Step-Video-TI2V Technical Report, arXiv:2503.11251, 2025-03 (includes 5M text-image-video triplets, a self-disclosed mixture imbalance of anime-style data exceeding 80%, an optical-flow motion-score extraction method with threshold filtering, motion score used as a controllable condition, caption-model fine-tuning to strengthen camera-motion description, and Step-Video-TI2V-Eval's 178+120 prompts): https://arxiv.org/html/2503.11251v1
- [Same-team corroborating] Step-Video-TI2V GitHub repository: https://github.com/stepfun-ai/Step-Video-TI2V
- [Third-party] The Moonlight literature review: an interpretation of the Step-Video-T2V technical report (explaining the hierarchical filtering and the 6 pretraining subsets): https://www.themoonlight.io/en/review/step-video-t2v-technical-report-the-practice-challenges-and-future-of-video-foundation-model
- [Third-party] Kingy AI: a summary of the Step-Video-T2V technical report paper (overview of the data pipeline and SFT/DPO): https://kingy.ai/blog/step-video-t2v-technical-report-paper-summary/
- [Third-party] Hugging Face Papers: the Step-Video-T2V paper page and community discussion: https://huggingface.co/papers/2502.10248
- [Third-party] CSDN: StepFun releases the strongest open-source video-generation model, Step-Video-T2V (a detailed paper walkthrough, with the Chinese data section broken down item by item): https://blog.csdn.net/sherlockMa/article/details/145706142
- [Third-party] CSDN: StepFun's open-source exploration — an in-depth analysis of Step-Video-T2V and Step-Audio (explaining that the two are independent model lines, not joint AV generation): https://blog.csdn.net/liaoqingjian/article/details/145820964
- [Third-party] Zhihu: a brief analysis of StepFun's 30B video-generation model Step-Video: https://zhuanlan.zhihu.com/p/24619034131
- [Third-party] Zhihu: an introduction to StepFun's open-sourced image-to-video model Step-Video-TI2V (102 frames/5 seconds/540P, controllable motion amplitude and camera motion): https://zhuanlan.zhihu.com/p/31775732208
- [Third-party] BAAI Hub: StepFun's first open-sourcing of the Step-series multimodal large models (official release report, 2025-02-18): https://hub.baai.ac.cn/view/43466
- [Third-party] NeuroHive: an explanation of Step-Video-T2V's open-source model achieving a 16x video-compression breakthrough: https://neurohive.io/en/state-of-the-art/step-video-t2v-text-to-video-open-source-model-achieves-16x-video-compression-breakthrough/

## Data Scale and Distribution

### Training data magnitude (number of videos/hours/tokens, pretraining vs. SFT separated)

[Pretraining total] The technical report explicitly states: a large-scale dataset was constructed containing 2 billion (2B) video-text pairs and 3.8 billion (3.8B) image-text pairs. This is one of the hardest quantitative disclosures in this entry.
[Actual samples consumed at each training stage]
· Step-1 text-to-image (T2I) pretraining: 256px, about 3.8B image samples, 253k iterations;
· Step-2 text-to-image/video (T2VI) joint pretraining, low-resolution tier: 192×320, about 1 billion (1B) video clips available, actual seen samples 644 million (644M), 430k iterations;
· Step-2 high-resolution tier: 544×992, actual seen samples 27.3 million (27.3M), 46k iterations;
· Step-3 text-to-video (T2V) fine-tuning / SFT: 544×992, about 30 million (30M) high-quality videos;
· Step-4 Video-DPO: 544×992, constructing preference pairs based on the model's own self-generated videos; the distillation (Turbo) dataset is about 95,000 (95k) samples.
[Note] The report does not give the hours of the raw video pool, the number of original clips before segmentation, or the absolute elimination amounts at each filtering stage (the funnel is presented only as a relative bar chart in Figure 11, with no absolute values annotated). Neither total video duration in hours nor token counts are disclosed.

### Data source composition (in-house/public datasets/web crawling/licensed acquisition/synthetic data) ⚠️

The report does not describe the source composition. It only states that "raw videos" were transformed through the complete pipeline into high-quality video-text pairs suitable for pretraining, without distinguishing the proportions of in-house footage libraries, public datasets, web crawling, licensed acquisition, or synthetic data; nor does it list any specific data-source names. Indirect clues that can be inferred from the pipeline design: (1) the explicit retention and use of a video's "Original Title" as one caption source indicates a large amount of the data comes from network video platform content with title metadata; (2) the use of an EfficientNet watermark classifier and PaddleOCR subtitle detection indicates the raw pool contains a large amount of secondarily distributed content with station logos/subtitles, consistent with characteristics of web crawling; (3) in the derivative model Step-Video-TI2V, training data being more than 80% anime-style video indicates the team has a large-scale source of anime content. The source of the image-text side's 3.8B is likewise undisclosed. [Uncertain]

### Data compliance and provenance (proportion of licensed data, rights-cleared datasets, C2PA, etc.) ⚠️

Not disclosed. The technical report does not address the proportion of licensed data, rights-cleared datasets, copyright-clearance workflows, C2PA, or any other content-provenance standards, nor does it mention generated-content watermarking/labeling. The only visible trace on the data-compliance dimension in the pipeline is NSFW scoring (a LAION CLIP-based NSFW detector) and watermark detection (used to remove watermarked footage, with a motivation closer to visual cleanliness and avoiding obvious copyright markers, rather than systematic copyright compliance). On the model side, the MIT license is adopted for open-sourcing, with no data-provenance commitments included. As a product released within China, the actual production process necessarily involves content-safety and compliance review, but the training-data side's compliance methodology is entirely undisclosed. [Uncertain]

### Clip-duration distribution and segmentation strategy

No duration histogram is given, but the segmentation and bucketing strategy is clear:
[Segmentation] The PySceneDetect AdaptiveDetector function is used to detect scene changes, followed by FFmpeg segmentation into single-shot clips; each segmented clip additionally has its first and last 3 frames discarded to eliminate unstable camera motion and residual transition frames at cut points — a fine-grained but practical engineering detail that many contemporaneous works did not implement.
[Duration expression] The training side expresses duration in frame counts rather than seconds, adopting frame-length bucketization: 1 frame, 68 frames, 136 frames, and 204 frames as four tiers (the 1-frame tier is images, used for joint image-video training). The model's maximum single-generation length is 204 frames; per the report's framing, 204 frames corresponds to roughly the 8-second level (the derivative model Step-Video-TI2V is 102 frames/5 seconds, from which a frame rate of roughly 20fps can be inferred).
That is, in curriculum design, "short → long" was made an explicit frame-count bucketing axis, rather than a fixed clip duration.

### Resolution/aspect-ratio distribution and bucketing strategy

[Resolution] Serves as the primary curriculum axis: image stage at 256px → video stage at 192×320 (192P) → 544×992 (540P). The final released model outputs at 540P, not trained to 720P/1080P — a notable trade-off difference from contemporaneous Hunyuan/Wanxiang (the team invested its compute in long 204-frame sequences and a deeply compressed VAE instead).
[Aspect ratio] Aspect-ratio bucketization is used, divided into landscape, portrait, and square groups, combined with the frame-length buckets (1/68/136/204) to form a two-dimensional bucket system, supporting mixed multi-resolution, multi-aspect-ratio training.
[Black-border handling] The pipeline uses FFmpeg to detect black-border size and crop accordingly, ensuring frames have no padding borders before entering a bucket.
The specific proportions of each aspect-ratio bucket are not disclosed.

### Category/domain distribution and mixture strategy (proportional control and concept balancing across people, actions, scenes, styles, etc.) ⚠️

The report does not give domain proportion figures, but it does provide a concept-balancing mechanism with considerable distinctiveness among contemporaneous work — "Video Concept Balancing":
· The in-house VideoCLIP model computes a video embedding for each clip;
· K-means clustering is performed in the high-dimensional embedding space, forming over 120,000 (120,000+) clusters — a cluster count far higher than contemporaneous comparable work (e.g., HunyuanVideo's approximately 10,000 concept centers), implying an extremely fine-grained concept division;
· Two derived labels are assigned to each clip: Cluster_Cnt (the number of samples in its cluster, used to identify overly dense or long-tail concepts) and Center_Sim (cosine distance to the cluster center, used to identify within-cluster outlier samples);
· Based on these two labels, two types of operations are implemented: first, resampling by cluster size, ensuring broad category coverage and suppressing over-concentration of head-tier concepts; second, at the post-training stage, removing clips whose Center_Sim is too far from the cluster center (using clustering for both "concept balancing" and "outlier quality control," a point of considerable methodological value in this entry).
The final proportions of specific categories (people/actions/scenes/styles) and the specific resampling multipliers are not disclosed. On the Step-Video-TI2V side, there is an explicit disclosure of style imbalance: over 80% of the training data is anime-style video, and the early stage used only anime data, resulting in the model performing strongly on anime scenes while its real-world scene performance is limited — a rare, officially self-disclosed case of "domain-mixture imbalance directly leading to skewed capability." [Uncertain]

### Audio-category distribution and mixture (proportions and control strategy for speech/foley/music/ambient sound/silence) — a dimension unique to AV models ⚠️

Not applicable. Step-Video-T2V does not generate audio, and the training-data pipeline does not involve the extraction, classification, or mixture of audio tracks at all; the report contains no content on proportional control of speech/foley/music/ambient sound/silence. For audio-data methodology within the StepFun ecosystem, one would need to turn to the independent Step-Audio series (a speech-interaction model, with data construction involving dimensions such as emotion, dialect, language, and singing), but it has no data-sharing relationship with the video model. [Uncertain]

### Narrative-structure distribution (single-shot vs. multi-shot, average clip duration, shot-count distribution, whether native audio tracks are present)

Explicitly adopts a "single-shot" data paradigm, with no multi-shot narrative training: PySceneDetect's AdaptiveDetector detects scene changes, after which FFmpeg segments the video, with each segment containing only one shot, and the first and last 3 frames of each further removed to eliminate transition residue. As a result, the shot-count distribution is invariably 1, and the average clip length corresponds to the three tiers of 68/136/204 frames (plus the 1-frame image tier).
Notably, the dense caption explicitly describes "camera movements," and the human-review criteria at the SFT stage also include "smooth scene transitions" — indicating the team is attentive to cinematography, but this is at the level of camera movement within a single shot, not cross-shot multi-shot narrative structure.
Native audio track: none (the data side does not retain or process any audio track).

### Language/accent distribution (the data basis for multilingual lip-sync capability) ⚠️

Not applicable to lip sync (no audio generation). On the text side, however, this entry has a distinctive feature: the model employs two bilingual (Chinese-English) text encoders (a combination of Hunyuan-CLIP and an in-house Step-LLM dual encoder), natively supporting bilingual Chinese-English prompt input; the official team lists "native Chinese-English bilingual input" as a core capability. The companion evaluation benchmark Step-Video-T2V-Eval's full set of 128 prompts is entirely in Chinese, also corroborating a Chinese-side emphasis in its data and evaluation. However, the language composition proportion of Chinese vs. English training captions, and whether any dedicated enhancement was applied to Chinese captions, are not disclosed in the report. [Uncertain]

## Cleaning Pipeline

### Overall structure of the cleaning funnel (number of filtering stages, order of stages)

The core value of this entry. Step-Video-T2V's data pipeline consists of six sequential stages, and the tools/models used at nearly every stage are named — its reproducibility description is on the better side among contemporaneous work:
Stage 1, Video Segmentation: PySceneDetect's AdaptiveDetector detects scene changes → FFmpeg cuts into single-shot clips → the first and last 3 frames of each clip are discarded.
Stage 2, Video Quality Assessment: 7 categories of quality labels are assigned to sampled frames (aesthetic score, NSFW score, watermark, subtitle, saturation, blur, black border); see the quality_filtering field for details.
Stage 3, Video Motion Assessment: the Farneback optical-flow algorithm computes three motion quantities — Motion_Mean / Motion_Max / Motion_Min.
Stage 4, Video Captioning: an in-house VLM produces three types of text — Short Caption, Dense Caption, and Original Title.
Stage 5, Video Concept Balancing: the internal VideoCLIP extracts embeddings → K-means clustering into 120,000+ clusters → Cluster_Cnt and Center_Sim labels are assigned → resampling for balance and outlier removal.
Stage 6, Video-Text Alignment: 8 frames are uniformly sampled from each clip, and the average cosine similarity between frame embeddings and text embeddings is computed to obtain a CLIP Score, used to remove image-text-mismatched samples.

[Hierarchical Data Filtering — this pipeline's organizational approach] What the above stages produce is a complete "label system" rather than a one-shot discard: all clips are first fully labeled, and then, by progressively raising each label's threshold, 6 subsets are cut out for Step-2 T2VI pretraining (Figure 11 illustrates each level of filtering as a bar chart: gray bars represent data filtered out at that level, colored bars represent retained data). This "label everything first, then cut by threshold" design is more flexible than "filter and discard as you go," allowing curriculum adjustments without rerunning the pipeline.
The post-training stage layers two additional levels on top of this: automated scripts + heuristic rules + removal by cluster-center distance for outliers → manual review curation, yielding the 30M SFT set.

### Quantitative funnel retention rates (input/output volume at each filtering stage and final retention rate, e.g., Apollo's 27%) ⚠️

Quantitative disclosure is incomplete, allowing only partial estimation:
[Known absolute figures] Raw pool → 2B video-text pairs (the pipeline's total output) → about 1B clips available at the low-resolution (192P) tier (actual seen 644M) → high-resolution tier seen 27.3M → SFT 30M → DPO/distillation about 95k.
[Estimable retention rates] 2B → 1B (192P tier) about 50%; 2B → 30M (SFT high-quality tier) about 1.5%, i.e., the end-to-end retention rate from pipeline output to SFT is about 1.5% (this conversion is calculated for this research entry; the report does not directly give this ratio).
[Missing portions] (1) The absolute scale of the raw video pool (before segmentation) is not disclosed, so the retention rate for the segment "raw footage → 2B pairs" cannot be calculated; (2) Figure 11's hierarchical funnel chart only illustrates the relative retain/discard ratio at each level via bar length, without annotating any absolute values or percentages; (3) the sample sizes of each of the 6 pretraining subsets are not listed.
Consequently, the quantitative completeness of this entry's funnel is lower than that of HunyuanVideo 1.5 (which gives a complete chain of >10 million hours → 800 million → 200 million → 100 million → 1 million); the only comparable figure is the SFT-side ~1.5% order of magnitude. [Uncertain]

### Shot-segmentation method (PySceneDetect/in-house model/shot-aware splitting)

PySceneDetect's AdaptiveDetector function (an adaptive-threshold detector, more robust to gradual transitions and camera motion than ContentDetector) detects scene-change points, after which FFmpeg is invoked to segment into single-shot clips at the detected boundaries. After segmentation, the first 3 and last 3 frames of each clip are uniformly removed, on the grounds that these frames often contain unstable camera motion (transition residue, shaking).
Comparison with contemporaneous work: no use of dual-path cross-validation with tools such as TransNetV2 (which HunyuanVideo used), and no additionally trained "transition classifier" for secondary cleaning (which HunyuanVideo 1.5 used) — a relatively standard single-tool approach; but the "discard first and last 3 frames" compensatory design is a distinguishing feature.

### Quality filtering (aesthetic scoring, clarity, OCR text filtering, black-border/watermark/logo detection)

At Stage 2, each clip's sampled frames are assigned 7 categories of quality labels, with tools and methods almost entirely named (specific threshold values are uniformly undisclosed; only "progressively raising thresholds" to construct 6 subsets is described):
1. Aesthetic Score: the LAION CLIP-based aesthetic predictor (the open-source LAION aesthetic predictor), assessing the visual appeal of the frame;
2. NSFW Score: a LAION CLIP-based NSFW detector, a binary classifier based on CLIP ViT-L/14;
3. Watermark Detection: an EfficientNet-based image classification model, determining whether a frame contains a watermark;
4. Subtitle Detection: PaddleOCR identifies text within a frame, used to remove clips with excessive subtitles/text;
5. Saturation Score: statistics on the mean/max/min of saturation in HSV color space, removing anomalous frames that are overexposed, overly gray, or oversaturated;
6. Blur Score: the Laplacian variance method, measuring image sharpness to remove out-of-focus/blurry clips;
7. Black Border Detection: FFmpeg detects black-border size and crops accordingly (cropping rather than discarding, the same approach as HunyuanVideo 1.5).
Methodological characteristics: this quality-inspection suite is entirely a combination of "traditional CV operators + open-source small classifiers," with none being large-model semantic judgment — a typical example of the "shallow multi-scorer" paradigm of early 2024–2025. Its advantages are that all tools are named, cost is low, and reproducibility description is good; its drawback is an inability to capture semantic-level quality problems (such as physical implausibility or subject distortion).

### Motion filtering (optical-flow/motion-score thresholds, removal of static and shaky footage)

Stage 3 sets up a dedicated "Video Motion Assessment": the classic OpenCV Farneback dense optical-flow algorithm computes the optical-flow field for each clip, deriving three scalar metrics — Motion_Mean (average motion magnitude), Motion_Max (peak motion magnitude), and Motion_Min (minimum motion magnitude). This combination of three values is used to identify and remove two types of samples: nearly static clips (insufficient motion information, prone to causing the model to generate static imagery) and clips that are overly intensely moving/shaky. Specific thresholds are not disclosed.
The derivative model Step-Video-TI2V upgrades this motion score from a "filter" to a "controllable condition," with more clearly described method details: sampling once every 12 frames, converting to grayscale and computing optical-flow magnitude, taking the mean of the highest-magnitude batch of values as the video's motion score; after extracting motion scores for the entire training set, one side sets a threshold to filter out videos with motion that is too high or too low, and the other side injects the motion score as an explicit condition into the model, achieving controllable motion amplitude at inference time. This is a clear example of "data labels directly converted into controllable conditions," of reference value to this research.

### Deduplication method (exact deduplication and embedding-based semantic deduplication recorded separately) ⚠️

The technical report describes no deduplication step at all — the six-stage pipeline contains no independent deduplication step, and there is no mention of exact/near-duplicate deduplication such as perceptual hashing (pHash), nor of embedding-based semantic deduplication.
The only mechanism that functionally partially achieves a deduplication effect is Stage 5's concept balancing: K-means (120,000+ clusters) on VideoCLIP embeddings, combined with Cluster_Cnt (within-cluster sample count) for resampling, can suppress the over-concentration of highly similar content, and Center_Sim can remove within-cluster outlier samples; but this is fundamentally distribution balancing and outlier detection, not equivalent to duplicate-sample removal. This is a disclosure gap relative to HunyuanVideo (which explicitly uses VideoCLIP cosine distance for semantic deduplication). [Uncertain]

### VLM/LLM as quality inspector (multimodal large-model quality scoring and mismatch removal; the 2026 trend from shallow scorers toward large-model semantic judgment) ⚠️

This entry clearly leans toward "traditional scorers" rather than "large-model semantic judgment" on this dimension, serving as a baseline sample for observing the paradigm shift from early 2025 to 2026:
[List of models responsible for quality inspection] All are specialized small models or classic algorithms — the LAION CLIP aesthetic predictor, the CLIP ViT-L/14 NSFW binary classifier, the EfficientNet watermark classifier, PaddleOCR, Laplacian variance, HSV statistics, FFmpeg black-border detection, Farneback optical flow, the internal VideoCLIP (clustering and outlier detection), and the CLIP Score (image-text alignment). No step uses a multimodal large model to perform end-to-end semantic quality judgment or physical-plausibility judgment on video.
[Position of the large model] The in-house VLM appears only in the captioning step (generating short/dense captions), and does not bear judgment or removal responsibilities; no dedicated post-training was done for hallucination suppression on this caption VLM either (compared with HunyuanVideo 1.5's use of OPA-DPO to govern annotator hallucination).
[The only model-level mismatch removal] Stage 6's CLIP Score image-text-alignment filter, which falls under "using a discriminative contrastive model for coarse-grained semantic matching" — a shallow semantic judgment rather than a VLM-reasoning-style judgment.
[Humans substitute for the large model's role] The semantic-level quality judgments at the SFT stage (clarity, aesthetics, motion plausibility, smoothness of scene transitions, caption accuracy) are completed by human reviewers rather than handed to a VLM — precisely the part that the 2026 trend is meant to replace. Step-Video-T2V can thus serve as a control sample from "before the large-model quality-inspector paradigm." [Uncertain]

### Safety and compliance filtering (NSFW, copyright, faces/privacy) ⚠️

Disclosure is very weak, with only one item: the NSFW scoring in Stage 2 of the pipeline, using a LAION-provided CLIP ViT-L/14-based binary NSFW detector to score sampled frames and filter out pornographic/inappropriate content accordingly. Beyond this, the report does not mention copyright filtering, face/privacy protection (face mosaicing, portrait-rights handling), violent/gory content classification, politically sensitive content filtering, celebrity-likeness removal, or any generation-side safety alignment or output review. Watermark detection can indirectly remove some footage with obvious copyright markers, but its design motivation is visual cleanliness rather than compliance.
As a commercial product operating within China (launched on 跃问 yuewen.cn), the actual production system necessarily has a complete content-safety review pipeline, but the technical report discloses almost nothing about safety filtering on the data side. [Uncertain]

## Annotation Methods

### Caption model used (in-house VLM/open-source model, model scale) ⚠️

An in-house Vision Language Model is used as the caption model, generating a description for each video clip. The report does not disclose this VLM's name, parameter scale, base architecture, training data, or training method, nor whether it is adapted from StepFun's own Step-1V multimodal model. No dedicated post-training was done for hallucination suppression on the caption model (no DPO/RLHF-type description); caption quality is backstopped by manual review at the SFT stage (humans "refine captions") and by the Stage-6 CLIP Score alignment filter.
The derivative model Step-Video-TI2V discloses one incremental step: an in-house video-captioning model was fine-tuned specifically to strengthen its ability to describe "object-motion dynamics" and "camera motion," giving an example caption of the form "a flock of birds flying over a tree at sunset, camera pans left" — i.e., allowing the caption to explicitly carry camera-motion instructions, paired with I2V's controllable-camera-motion design. [Uncertain]

### Caption density and structuring level (short/long/dense description, structured fields such as camera motion, style tags)

Adopts a "three-way parallel caption" rather than a single dense description — the most notable design in this entry's annotation dimension:
1. Short Caption: extremely minimal, focusing solely on the main subject and action;
2. Dense Caption: detailing key elements, emphasizing subject, event, environment, and visual presentation, and explicitly including a description of camera movements;
3. Original Title: directly reuses the video's own original title text; the team's stated rationale is to introduce "stylistic diversity" — i.e., preserving the distribution of genuinely human-written text, with its colloquial/clickbait/stylized coloring, preventing the model from only ever seeing the regular, standardized sentence forms produced by a VLM, so that it can better adapt to real users' casual prompts at inference time. This shares the same purpose as HunyuanVideo's use of caption-field dropout + permutation/combination augmentation (aligning to the inference-time prompt distribution), but via a different path — StepFun's approach is "introducing a genuinely human text source," whereas Hunyuan's is "random recombination of structured fields."
[Assessment of structuring level] Compared with HunyuanVideo's seven-field JSON schema (containing independent fields such as Style, Shot Type, Lighting, Atmosphere), Step-Video-T2V's caption is natural-language long/short text rather than a strongly field-structured schema — attributes such as style/lighting/atmosphere are implicit within the dense caption's free text, not broken out into independent controllable fields. This limits the ability to perform field-based conditional control during training (its controllability is achieved mainly through TI2V's scalar motion score).
How the three captions are mixed/sampled during training (proportions, random-switching strategy) is not stated in the report.

### Joint audio-video caption structure (whether visual + auditory tracks are covered simultaneously, whether split into independent fields, e.g., LTX-2's full soundscape description, Script-a-Video's factorized streams, Foley-Omni's three-field schema) ⚠️

Not applicable. The model has no audio modality; the caption covers only the visual track (subject, action, environment, visual presentation, camera motion), with no dual visual+auditory caption structure, and no soundscape description, speaker field, or sound-effect field. It can serve as a "pure-visual caption" control baseline: its three-way caption design (short/dense/original title) — with the idea of "retaining the original title to introduce a genuinely human text-style distribution" — could in principle be transplanted orthogonally to an AV caption system (e.g., retaining a video's original title and human-written description as a source of stylistic diversity for soundscape captions), but the report itself contains nothing AV-related. [Uncertain]

### Dialogue transcription and speaker-attribute annotation (ASR transcription + speaker identity/language/accent/emotion) ⚠️

Not applicable. The data pipeline does not process audio tracks; there is no ASR transcription, and no speaker identity/language/accent/emotion annotation. Within the StepFun ecosystem, this class of capability resides in the independent Step-Audio series of speech models (which support attributes such as emotion, dialect, language, and singing in speech generation), but it has no intersection with Step-Video-T2V's training data. [Uncertain]

### Geometric and structured annotation (camera parameters, depth, 3D point tracks, action annotation, explicit state)

Structured annotation is relatively sparse, mainly consisting of two types of scalars and one type of text:
[Motion scalars] The three numeric labels Motion_Mean / Motion_Max / Motion_Min derived from Farneback optical flow (used for filtering during pretraining; upgraded in Step-Video-TI2V into an explicit controllable condition injected into the model, enabling controllable motion amplitude);
[Clustering scalars] The Cluster_Cnt and Center_Sim from VideoCLIP embeddings;
[Camera motion] Written into the dense caption as natural language (e.g., "camera pans left"), rather than as discrete label enumeration — different from HunyuanVideo's approach of training a 14-class camera-motion classifier that outputs discrete labels; Step-Video-TI2V achieves "controllable camera motion" by fine-tuning the caption model to strengthen camera-motion description, still following the natural-language path.
[Entirely absent] No camera intrinsic/extrinsic parameter estimation, depth maps, 3D point tracks, optical-flow fields used as a supervisory signal in their own right, action skeletons, or explicit physical-state annotation are used. This contrasts with routes such as Movie Gen and Seedance that introduce stronger geometric/structural supervision, and is also a shortcoming the team self-acknowledges in the report's "Challenges and Future" section (insufficient capability for complex physics and causal modeling).

### Synthetic data construction (controlled perturbation/edit-constructed training pairs, e.g., InstructAV2AV) ⚠️

No synthetic data construction was used on the training-corpus side (no controlled-perturbation/edit-constructed training pairs, of the InstructAV2AV kind). The only "constructive" elements occur in two places within post-training, both being model self-sampling rather than corpus synthesis:
· Video-DPO preference data: for each prompt, Step-Video-T2V generates multiple videos with different random seeds, forming a candidate set for human-preference ranking — a preference pair from model self-generation;
· Distillation data: to train the accelerated version Step-Video-T2V-Turbo, approximately 95,000 (95k) samples were sampled to form the distillation dataset.
Additionally, there is light synthesis on the prompt side: the DPO prompt set, besides being randomly sampled from training data, also invited human annotators to "synthesize" new prompts following carefully designed guidelines, to expand prompt-distribution coverage. [Uncertain]

### Degree of human involvement (manual annotation, manual quality control, model pre-screening + manual review)

Human effort is concentrated at the very end of the funnel (pretraining fully automated, post-training heavily manual), across three points:
1. Manual curation of SFT data: the construction of the 30M high-quality video set is a two-stage process of "automation + human": first, various evaluation scores and heuristic rules automatically filter the data; then, outlier samples exceeding a distance threshold from the cluster center are removed by video category (cluster); finally, human annotators review each clip item by item, along dimensions of clarity, aesthetic quality, appropriate motion, and smooth scene transitions; humans also refine/rewrite captions at the same time — i.e., people not only filter the data but also correct the annotations.
2. DPO preference annotation: human annotators synthesize prompts following design guidelines; multiple videos generated for the same prompt with different seeds are given preference rankings by human annotators; the whole process is supervised by quality-control personnel to ensure annotation accuracy and consistency.
3. Human review for evaluation: on Step-Video-T2V-Eval, human evaluation is the primary assessment method, comparing against multiple open-source and commercial engines.
The size of the annotation team, labor-hour cost, or annotation-consistency metrics (such as a Kappa value) are not disclosed. In the outlook section, the report explicitly notes that DPO's gains saturate "once the model can easily distinguish positive from negative samples," and proposes that future work introduce a reward model to dynamically score newly generated samples in place of pure human annotation — i.e., the team already recognizes the scalability bottleneck of purely manual preference annotation.

## Audio-Video Alignment

### Audio-video synchronization detection method (lip sync, event alignment) ⚠️

Not applicable. The model has no audio modality, the training data contains no audio track, and there is no audio-video synchronization detection step in the pipeline; the report contains no content whatsoever related to SyncNet, AV-align, lip sync, or event alignment. [Uncertain]

### Specific synchronization metrics and thresholds (SyncNet/Synchformer/LSE/in-house, threshold values, e.g., MOVA's LSE-D ≤ 9.5 and LSE-C ≥ 4.5, SkyReels-V4's SyncNet |offset| ≤ 3 ∧ conf > 1.5) ⚠️

Not applicable. No audio-video synchronization metric is used, and no corresponding threshold exists. What can be noted as a point of comparison is that this entry's only cross-modal alignment mechanism is visual-text alignment: 8 frames are uniformly sampled from a clip, and the cosine similarity between each frame's embedding and the caption text embedding is computed and averaged, yielding a CLIP Score as an image-text-alignment label used to filter mismatched samples — but this threshold's specific value is likewise undisclosed. [Uncertain]

### Separation of temporal synchronization vs. semantic synchronization (temporal alignment and content-semantic matching handled as two independent filtering conditions) ⚠️

Not applicable (no audio). There is a structural analogy on the purely visual side, however: the pipeline sets the "temporal/motion dimension" (Stage 3's three Farneback optical-flow motion scores) and the "semantic-matching dimension" (Stage 6's CLIP Score image-text alignment) as two mutually independent filtering conditions, separately handling "is the motion right" and "does it depict what the description says" — this is structurally isomorphic with the idea of AV models splitting temporal synchronization and semantic matching into two independent gates, and can serve as a weak methodological point of reference. [Uncertain]

### Audio quality filtering (SNR, silence detection and silence-ratio thresholds, no-audio-track removal, voice-over-source removal, background-music separation) ⚠️

Not applicable. The data pipeline does not extract, retain, or process audio tracks at all — there is no SNR computation, silence detection, no-audio-track removal, voice-over-source removal, or background-music separation of any kind. [Uncertain]

### Classification and differential handling strategy for speech/sound-effects/music ⚠️

Not applicable. There is no classification or differential handling strategy for speech/sound-effects/music. StepFun's speech-side capability is encapsulated within the independent Step-Audio model, which has no intersection with this entry's video data pipeline. [Uncertain]

## Training Integration

### Multi-stage training curriculum and data-curriculum scheduling (basis for stage division: resolution/duration/quality score/modality; low-res→high-res, image→video, short→long)

Adopts an explicit four-step cascaded training pipeline, with a curriculum axis spanning modality, resolution, and frame length simultaneously, and the team explicitly states its purpose is to "accelerate convergence and fully leverage video datasets of varying quality" — i.e., each curriculum step corresponds to a specific data tier:
· Step-1 text-to-image (T2I) pretraining: 256px, about 3.8B images, 253k iterations. Establishes a basic visual prior and text alignment;
· Step-2 text-to-image/video (T2VI) joint pretraining: progressed in two tiers — first 192×320 (192P), with about 1B video clips available, seen 644M, 430k iterations; then 544×992 (540P), seen 27.3M, 46k iterations. Images and videos are trained jointly throughout (T2VI, i.e., text-to-video-and-image), with the 1-frame frame-length bucket carrying image samples to prevent image capability from degrading;
· Step-3 text-to-video (T2V) fine-tuning / SFT: 544×992, 30M high-quality videos;
· Step-4 Video-DPO: 544×992, direct preference optimization based on human preference data.
[Bucketing] Throughout, this is paired with frame-length bucketing (1/68/136/204 frames) and aspect-ratio bucketing (landscape/portrait/square), achieving a "short → long" frame-length curriculum along with mixed multi-aspect-ratio training.
[Key experience self-reported by the team] (1) the more stable the low-resolution stage (192P) is, and the more diverse the data used, the easier subsequent scaling to high resolution becomes — i.e., "diversity" is loaded onto the low-resolution stage and "precision" onto the high-resolution stage; (2) they observed that training loss exhibits a "sudden drop in loss" as training-data quality improves, and the team used this data-quality step as a direct signal for curriculum switching — a quite practically valuable empirical finding in this report; (3) for checkpoint selection, weight averaging (checkpoint averaging) outperforms EMA, and one should choose a checkpoint after the gradient-norm peak, once both the gradient norm and the loss have declined.

### Data mixture changes across training stages (pretraining/annealing/SFT high-quality subset) ⚠️

The mixture strategy is expressed via "the same label set, with progressively raised thresholds" rather than explicit categorical mixture figures:
· The Step-2 T2VI pretraining stage uses the 6 subsets cut out by hierarchical filtering, with each subset's filtering threshold progressively tightened (from loose to strict), corresponding to the progression from 192P to 540P — i.e., an annealing-style schedule of "large, mixed-quality data used early, small, refined data used later";
· Images and videos are trained jointly throughout (T2VI), with images participating via the 1-frame bucket, but the specific image:video mixture ratio is not disclosed;
· Step-3 SFT switches to the 30M high-quality subset (about 1.5% of the 2B total);
· Step-4 DPO uses a much smaller-scale set of human preference pairs;
· The distilled Turbo version uses about 95k samples.
[Quantitative gaps] The sample sizes of each of the 6 pretraining subsets, the image/video mixture ratio at each stage, and the target distribution of each concept cluster after resampling are all undisclosed. [Uncertain]

### Post-training data (SFT curated-set scale and selection criteria, number of preference pairs and annotation method, reward-model training data)

Post-training is divided into two stages, with the data-construction methodology relatively clearly disclosed:
[SFT (Step-3)] About 30 million (30M) high-quality videos, constructed in three steps: (1) automatic filtering by each of the pipeline's evaluation scores and heuristic rules; (2) removing outlier samples by video category (VideoCLIP K-means clusters) whose distance to the cluster center exceeds a threshold (the threshold value is undisclosed); (3) human annotators review each item, with criteria of clarity, aesthetics, appropriateness of motion, and smoothness of scene transitions, along with caption refinement.
[Video-DPO (Step-4)] (1) Prompt-set construction: a portion of prompts is randomly sampled from the training data, with human annotators invited to synthesize supplementary prompts following carefully designed guidelines to ensure prompt diversity; (2) response generation: for each prompt, Step-Video-T2V generates multiple videos with different random seeds; (3) preference annotation: human annotators give preference scores/rankings on these samples, with the whole process supervised by quality-control personnel to ensure consistency; (4) the DPO loss is used to update the model toward the preferred samples. The number of prompts, the total number of generated videos, and the number of annotators are all undisclosed.
[Reward model] No reward model was trained. In its outlook, the report explicitly notes the current limitation of DPO — its benefit saturates once the model can easily distinguish positive from negative samples — and proposes that future work introduce a reward model to dynamically score newly generated samples in order to continue providing effective gradient signal.
[Distillation] Step-Video-T2V-Turbo uses a distillation dataset of about 95,000 (95k) samples.

### Data-processing infrastructure and throughput (NeMo Curator/Data-Juicer/in-house, GPU acceleration ratio, processing scale, cost) ⚠️

Data-processing infrastructure and throughput are not disclosed. The report does not mention the name of NeMo Curator, Data-Juicer, or any in-house data-processing framework, and gives no GPU acceleration ratio, processing throughput (clips/hour), cluster scale, or processing cost. The engineering pressure can be inferred indirectly: to produce 2B video-text pairs, an even larger volume of raw video must go through PySceneDetect segmentation + 7-category quality scoring + Farneback optical flow + three-way in-house VLM caption inference + VideoCLIP embedding + 120,000-cluster K-means + 8-frame CLIP Score computation — of which the VLM captioning and VideoCLIP inference are billion-scale-sample, multi-model batch-inference tasks representing enormous engineering effort, but the report devotes essentially no attention to it.
The training side has considerably more disclosure regarding infrastructure (the in-house Step Emulator training framework/StepRPC communication library, memory optimization, parallelism strategy, multi-stage training-stability experience, checkpoint averaging outperforming EMA, etc.), but this does not fall under the category of data-processing throughput. [Uncertain]

## Performance Comparison

### Quantitative impact of data-strategy ablations (distinguish: filtering-strictness ablation / caption-density-style ablation / data-mixture ablation, and corresponding evaluation metrics) ⚠️

The technical report provides no controlled ablation experiments for data strategy — no filtering-strictness ablation (e.g., comparing the impact of the 6 subsets' different threshold tiers on final metrics), no caption density/style ablation (e.g., the individual contribution of short caption vs. dense caption vs. the original title, among the three tracks), and no data-mixture ablation (e.g., concept balancing on/off, or a comparison of image-video mixture ratios). This is the main methodological shortcoming of this entry: the pipeline is described in exhaustive tool-level detail, but the necessity and benefit of each step lack quantitative attribution.
The report offers two categories of substitute evidence instead:
(1) A qualitative training-curve observation — "as training-data quality improves, a sudden drop in loss occurs." This is the only direct empirical evidence regarding the impact of data quality, but it is only a qualitative phenomenon at the training-loss level, never converted into downstream generation-quality metrics, nor accompanied by a control group;
(2) End-to-end human comparative evaluation — on the self-built Step-Video-T2V-Eval (128 Chinese prompts across 11 categories), it outperforms multiple open-source and commercial engines in a comprehensive human comparison across dimensions such as instruction-following, motion smoothness, physical plausibility, and aesthetics; it is also compared on Movie Gen Bench. These results cannot be attributed to any specific data strategy. [Uncertain]

### Evidence for quality vs. quantity (cases where small, refined data surpasses large, noisy data) ⚠️

There is no strictly controlled comparative experiment, but there exists one fairly strong piece of indirect evidence and one clear counter-example:
[Positive indirect evidence] The data volume monotonically shrinks from the pipeline's output of 2B down to SFT's 30M (about 1.5%), together with the team's observed phenomenon of "sudden drop in loss when training-data quality improves," constituting empirical support for "small, refined data yielding significant gains later in training"; the rationale behind the overall four-step cascaded training design is itself "fully leveraging video datasets of varying quality" — using low-quality, large-scale data at the early, low-resolution stage to build a diversity prior, and using high-quality, small-scale data at the later, high-resolution stage for polishing, which is a case of "quality and quantity each playing their own role" rather than simple quality-first prioritization.
[Counter-example (more valuable)] The derivative model Step-Video-TI2V's official self-disclosure: over 80% of its training data is anime-style video, and only anime data was used in the early stage, resulting in significantly better performance on anime scenes and limited capability on real-world scenes. This is an officially acknowledged, qualitative case where "data-mixture imbalance directly leads to skewed capability," confirming from the opposite direction that data distribution (rather than sheer data volume) is decisive for the final capability boundary.
[Gap] There is no controlled experiment of "same model, same compute, different data tiers," so this does not, strictly speaking, constitute experimental proof. [Uncertain]

### Alignment relationship between training-data domain distribution and evaluation-benchmark category taxonomy (e.g., VABench's seven major categories) ⚠️

No explicit alignment description is given between the training-data taxonomy and the evaluation-benchmark taxonomy, but both sides' category information is relatively specific, allowing for a comparative analysis:
[Evaluation side] The self-built Step-Video-T2V-Eval contains 128 prompts across 11 categories: Sports, Food, Scenery, Animals, Festivals, Combination Concepts, Surreal, People, 3D Animation, Cinematography, and Style. This taxonomy includes both content-theme categories (sports/food/scenery/animals/festivals/people) and capability-stress-test categories (combination concepts, surreal, cinematography), the latter clearly designed to probe the model's weaknesses. The derivative Step-Video-TI2V-Eval contains 178 real-style and 120 anime-style image-text pairs, further subdivided into subject, background, camera motion, action, style, and color subcategories for I2V.
[Training side] The training-data taxonomy is implicitly defined by the 120,000+ K-means clusters of VideoCLIP embeddings — an unsupervised, fine-grained concept space with no mapping relationship to the 11 human-defined evaluation categories above, and the report does not state whether training-data mixture was adjusted in reverse to match evaluation categories.
[Conclusion] The two taxonomies are each independent, with vastly different granularity (120,000 unsupervised clusters vs. 11 human-defined categories), showing no evidence of alignment design. This differs from the approach of benchmarks like VABench's seven major categories, where the training distribution and evaluation categories are explicitly aligned. [Uncertain]

## Other Information

### summary_note

Core conclusion: Step-Video-T2V is one of the open-source samples in this research round with the most specifically named toolchain and the most reproducibility-friendly description; almost every step of its data pipeline names the specific open-source tool or model used. However, it does not involve the audio modality at all, offering no direct reference value for joint audio-video generation topics, and it lacks any controlled ablation of data strategy.

[The four most noteworthy points]
(1) A six-stage pipeline + full labeling followed by threshold-based tiering: segmentation (PySceneDetect AdaptiveDetector + FFmpeg, discarding the first and last 3 frames) → 7 categories of quality labels (LAION aesthetic predictor / CLIP ViT-L/14 NSFW / EfficientNet watermark / PaddleOCR subtitle / HSV saturation / Laplacian blur / FFmpeg black border) → Farneback optical-flow's three motion scores → three-way in-house VLM captions → VideoCLIP + K-means concept balancing → 8-frame CLIP Score image-text alignment. The key design is "fully label everything first, then progressively raise thresholds to cut out 6 pretraining subsets," rather than filtering and discarding as you go — letting curriculum adjustments happen without rerunning the pipeline.
(2) Ultra-fine-grained concept balancing with 120,000+ clusters: using the two derived labels Cluster_Cnt and Center_Sim, it simultaneously achieves concept-resampling balance and within-cluster outlier removal, with clustering granularity an order of magnitude higher than contemporaneous work (e.g., Hunyuan's 10,000 concept centers).
(3) Retaining "Original Title" among the three caption tracks: explicitly introducing human-written original video titles to gain stylistic diversity — a different path from HunyuanVideo's "field dropout + permutation/combination" for the same goal of "aligning to the real prompt distribution," a simple but effective idea worth borrowing.
(4) "Sudden drop in loss" as a signal of data quality: the team observed that training loss drops sharply as data quality steps up, and used this phenomenon as a practical basis for curriculum switching — a rare, directly disclosed empirical link between data quality and training dynamics.
In addition, the evolution of the motion score from a "filtering threshold" to a "controllable condition" (injecting the optical-flow motion score into the model in Step-Video-TI2V to enable controllable motion amplitude) is a clear example of "reusing a data label as a generation condition."

[Main shortcomings]
· No controlled ablation of any data strategy; the benefit of each pipeline component cannot be quantitatively attributed;
· Incomplete funnel quantification: only the main line of 2B → 1B → 30M is given, Figure 11's hierarchical filtering chart does not annotate absolute values, the sizes of the 6 subsets are not given, and the scale of the raw video pool is entirely unknown;
· No description whatsoever of a deduplication step (no hash deduplication, no embedding-based semantic deduplication);
· Zero disclosure on data-source composition, copyright and compliance, and privacy protection; safety filtering has only the single NSFW item;
· Quality inspection relies entirely on traditional CV operators and open-source small classifiers, with no large-model semantic judgment; semantic-level judgment is backstopped by humans at the SFT stage — this can serve as a control baseline from "before the large-model quality-inspector paradigm";
· Captions are natural-language long/short text, without a strongly field-structured schema, limiting the ability to perform field-based conditional control;
· Zero disclosure of data-processing infrastructure and throughput.

If the research goal involves data construction for joint audio-video generation, this entry can only serve as a tool-level reference checklist for visual-side segmentation/filtering/captioning; the audio side would need separate research into StepFun's Step-Audio series (which has no data intersection with this model).

## Uncertain Fields

The research information for the following fields is partially uncertain (⚠️-marked sources):

- data_sources
- provenance_licensing
- domain_distribution
- audio_category_distribution
- language_accent_distribution
- funnel_retention_rate
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
- stage_data_mixture
- data_infra_throughput
- data_ablation
- quality_over_quantity_evidence
- benchmark_taxonomy_alignment
</content>
