# Veo 3 / Veo 3.1 (including Veo 3.1 Lite)

> Topic: Data processing for video-generation models (including simultaneous audio-video generation): data filtering pipelines, data distribution, annotation methods

[← Back to home](../index.md)

**Table of Contents**: [Basic Information](#basic-information) · [Data Scale and Distribution](#data-scale-and-distribution) · [Filtering Pipeline](#filtering-pipeline) · [Annotation Methods](#annotation-methods) · [Audio-Video Alignment](#audio-video-alignment) · [Training Coordination](#training-coordination) · [Comparative Results](#comparative-results)

## Basic Information

### Name

Veo 3 / Veo 3.1 (including Veo 3.1 Lite)

### Publishing Institution/Company

Google DeepMind (Google)

### Publication Date (technical report/paper/open-source date)

Veo 3 was released on May 20, 2025 at Google I/O. The official Model Card was first published on 2025-05-23 and updated on 2026-01-13; the accompanying technical report "Veo: a text-to-video generation system" was released at the same time. Veo 3.1 launched in mid-October 2025 (live on Flow / Gemini API / Vertex AI). The Veo 3.1 Lite Model Card was published on 2026-04-08. The official statement is that "this Model Card covers Veo 3 and its successor versions," i.e., the Veo 3.1 series follows the same data-and-training disclosures as Veo 3.

### Type (model/dataset/toolchain/evaluation benchmark)

Model (a closed-source commercial video+audio joint-generation foundation model, served via Gemini App, Flow, Google Vids, Google AI Studio, Gemini API, and Vertex AI)

### Degree of Open-Sourcing (whether weights/code/data/pipeline are each open-sourced)

Completely closed-source. Weights are not open-sourced, code is not open-sourced, training data is not public, the data-processing pipeline is not public, and the data mixture and scale are not disclosed. Only a roughly 7-page technical report (Veo-3-Tech-Report.pdf) and a roughly 6-page Model Card are publicly available, and the data-related body text within them totals fewer than 200 words — the vast majority of the space is devoted to responsibility and safety evaluation. Inference access is available only through paid API/product forms (Veo 3, Veo 3 Fast, Veo 3.1, Veo 3.1 Fast, Veo 3.1 Lite, and other variants).

### Whether Simultaneous Audio-Video Generation Is Supported, and the Implementation Approach (native joint / cascaded / MoE fusion)

Supported, and implemented as native joint generation, not a cascaded pipeline. The official technical report states verbatim: "Veo 3 uses latent diffusion, in which the diffusion process is applied jointly to the temporal audio latents, and the spatio-temporal video latents." That is, video and audio are each encoded by their own autoencoder into compressed latent representations (spatio-temporal latents for video, temporal latents for audio), after which a single Transformer-based denoising network jointly denoises both types of latents within the same diffusion process — each denoising step processes audio and video tokens simultaneously, thereby naturally guaranteeing synchronization at the generation stage. The generated content spans dialogue (with lip sync), sound effects/foley, ambient sound, and background music. No MoE fusion or video-first-then-dub two-stage cascaded scheme is used.

### List of Research Information Sources (URLs of papers/technical reports/official documentation/news, with each source labeled by nature: official primary source/same-team corroboration/third-party reporting)

- Official primary source: Veo 3 Tech Report (PDF, Model & Data / Mitigations sections) https://storage.googleapis.com/deepmind-media/veo/Veo-3-Tech-Report.pdf
- Official primary source: Veo 3 Model Card (PDF, Training Dataset/Processing/Evaluation) https://storage.googleapis.com/deepmind-media/Model-Cards/Veo-3-Model-Card.pdf
- Official primary source: Veo 3.1 Lite Model Card (confirms the 3.1 series follows the Veo 3 data disclosures) https://deepmind.google/models/model-cards/veo-3-1-lite/
- Third-party reporting: CNBC — Google used a subset of YouTube videos to train Gemini and Veo 3 https://www.cnbc.com/2025/06/19/google-youtube-ai-training-veo-3.html
- Official primary source: Video models are zero-shot learners and reasoners (arXiv:2509.20328, inferring the absence of explicit geometric annotation) https://arxiv.org/abs/2509.20328
- Official primary source: Veo product page (output specifications and MovieGenBench results) https://deepmind.google/models/veo/

## Data Scale and Distribution

### Training Data Volume (number of video clips/hours/token count, pretraining vs. SFT separated) ⚠️

[Uncertain] Google has never disclosed any training-data volume — no video clip count, no hour count, no token count, and no breakdown between pretraining and SFT scale. The technical report states only: "We train on a large dataset comprising images, videos, and associated annotations." Claims circulating in third-party blogs, such as "billions of audio-video pairs" or "millions of hours of paired audio-video," have no official source and should not be trusted. A weak corroborating clue is the use of Google TPU Pod clusters, JAX, and ML Pathways for training, hinting that the scale is among the largest of its era.

### Data Source Composition (proprietary/public datasets/web scraping/licensed procurement/synthetic data) ⚠️

Officially, the training data is stated only to consist of three types — "audio, video, and images" (Model Card: "Veo 3 was trained on audio, video, and image data") — with no disclosed proportions for each source. Confirmed source clues: (1) YouTube videos — reported by Reuters/CNBC in June 2025 and confirmed by YouTube, Google used a subset (not the entire corpus) of the YouTube corpus to train Gemini and Veo 3, with the legal basis being the "worldwide, non-exclusive, royalty-free" license in YouTube's Terms of Service, of which most creators were unaware; (2) synthetic data — officially confirmed to generate synthetic captions to increase concept diversity, but it is not stated whether synthetic video is used; (3) the specific composition and proportion of licensed/procured data versus public datasets [Uncertain]. The overall source-composition proportions are [Uncertain].

### Data Compliance and Provenance (proportion of licensed data, rights-cleared datasets, C2PA, etc.) ⚠️

Officially confirmed compliance measures: training videos are filtered against "various compliance and safety metrics"; on the caption side, unsafe descriptions and personally identifiable information (PII) are filtered out; the training data is analyzed to identify potentially harmful data and reviewed for fairness of demographic representation; on the output side, the SynthID invisible watermark is embedded uniformly, combined with production-environment filtering to reduce information-integrity risk. Across 2025-2026, the Google ecosystem applies SynthID uniformly to Imagen / Veo / Lyria outputs, and is progressively aligning with the C2PA Content Credentials metadata standard (whether Veo 3 output carries a C2PA manifest by default depends on the product line) [Uncertain]. Undisclosed key items: the proportion of licensed data, a list of rights-cleared datasets, and any revenue-sharing or opt-out mechanism for rights holders. The use of YouTube data has raised creator-knowledge-and-consent and IP disputes, which are a matter of public controversy.

### Clip Duration Distribution and Segmentation Strategy ⚠️

[Uncertain] The training-side clip-duration distribution and segmentation strategy are entirely undisclosed. Only inferable from the product side: Veo 3 generates a fixed 8 seconds per call; Veo 3.1 supports base clips of 4/6/8 seconds, and can be extended via the Extend feature (up to roughly 60 seconds or even beyond 140 seconds in Flow). The base generation unit being fixed at the 4-8-second scale hints indirectly that the main body of the training data consists of shot-segmented second-scale short clips rather than long videos.

### Resolution/Aspect-Ratio Distribution and Bucketing Strategy ⚠️

[Uncertain] The training data's resolution/aspect-ratio distribution and bucketing strategy are undisclosed. Product-side output specs: Veo 3 supports 720p / 1080p, aspect ratios 16:9 and 9:16; Veo 3.1 further supports up to 4K, likewise covering both 16:9 landscape and 9:16 portrait, at 24fps. Supporting both landscape and portrait ratios simultaneously suggests the training data includes multi-aspect-ratio samples and very likely employs some form of bucketing or multi-resolution mixed training, but the specific proportions and implementation have no official basis.

### Category/Domain Distribution and Mixture Strategy (proportional control and concept balancing across people, actions, scenes, styles, etc.) ⚠️

[Uncertain] The category/domain mixture strategy is undisclosed. Official clues that can be used for reverse inference: (1) the Model Card explicitly states that synthetic captions are generated "to improve the variety and diversity of concepts associated with videos in the training data" — a strategy of achieving concept balancing via caption-side rewriting rather than direct sample resampling; (2) the technical report, in the "Dangerous Capabilities" section, notes that Veo 3 "has a bias for generating cinematic footage, with frequent camera cuts and dramatic camera angles," and consequently struggles to generate low-production-value realistic coercion-type videos — this strongly suggests the training data is significantly skewed toward cinematic/professional film and high-production-value material, with low-production-value domains such as UGC, surveillance footage, or casual phone footage underrepresented; (3) fairness evaluation shows the model clearly skews toward lighter skin tones when race is unspecified, and exhibits semantic bias between specific vocabulary and specific demographic groups, reflecting an inherent imbalance in the person-representation distribution of the training data; (4) the official documentation acknowledges the model remains relatively weak at text rendering, hinting at insufficient coverage or annotation of text/OCR-containing scenes in the data. All of the above are inferences drawn from public reports, not an official mixture statement.

### Audio Category Distribution and Mixture (proportion and control strategy for speech/sound-effect foley/music/ambient sound/silence) — a dimension unique to AV models ⚠️

[Uncertain] The audio category mixture and control strategy are entirely undisclosed — this is one of the largest gaps in Veo 3's data disclosure. Inferable clues: (1) the model's capability spans four major categories — dialogue, sound effects/foley, ambient noise, and background music — indicating the paired audio-video training data has substantial coverage across all four; (2) the official documentation emphasizes that Veo 3 solved the previous generation of video models' "silent film" problem, meaning that samples with no audio track or with an audio track unrelated to the frame must have been explicitly excluded during training, but the exclusion criteria and the proportion of silent samples are undisclosed; (3) the technical report, in its deepfake assessment, notes that Veo 3's generated deepfakes are "much less controllable — particularly with respect to speech," hinting that speech data was not finely annotated or conditioned along the dimension of speaker identity/timbre; (4) the Model Card makes no mention of any source separation (e.g., separating BGM from vocals), SNR threshold, or silence-proportion threshold. Specific proportion figures for speech/sound effects/music/ambient sound have no public basis whatsoever.

### Narrative Structure Distribution (single-shot vs. multi-shot, average clip duration, shot-count distribution, presence of native audio tracks) ⚠️

[Uncertain] The proportion of single-shot vs. multi-shot in the training data, the average clip duration, and the shot-count distribution are all undisclosed. Reverse-inference clue: the technical report explicitly records the model's cinematic bias toward "frequent camera cuts and dramatic camera angles," indicating the training data contains a large amount of multi-shot (multi-shot) edited footage that has not been fully segmented into single shots — otherwise the model would not spontaneously produce edit points within an 8-second generation. This is an important piece of indirect evidence about Veo 3's data composition. Another certain item: the training data must predominantly have "native audio tracks," since the audio-video latents must enter the joint diffusion process as pairs.

### Language/Accent Distribution (data foundation for multilingual lip-sync capability) ⚠️

[Uncertain] Language and accent distribution are entirely undisclosed. On the product side, the model can generate multilingual dialogue with automatically matched accent and lip shape (English, Spanish, Mandarin, etc.), but the official documentation does not list a supported-language roster, does not give per-language data proportions, and does not describe an accent-annotation scheme. Third-party evaluations widely report that lip-sync and pronunciation quality for non-English dialogue are noticeably weaker than for English, indirectly indicating the training data is English-dominant. The official documentation also acknowledges the model "does not provide fine-grained control over accent, timbre, etc.," indicating speaker attributes were not incorporated as a conditioning dimension into the training-data schema.

## Filtering Pipeline

### Overall Structure of the Filtering Funnel (number of stages, order of each stage) ⚠️

The officially disclosed filtering funnel is extremely brief, reconstructible into only four confirmable stages (the order is inferred): (1) multi-granularity caption annotation — using multiple Gemini models to generate text descriptions at varying levels of detail for audio and video clips; (2) caption-side filtering — removing unsafe captions and descriptions containing personally identifiable information (PII); (3) video-side filtering — filtering training videos along three categories of metrics: compliance, safety, and quality, and applying safety filtering to pretraining data by risk area; (4) semantic deduplication — performing cross-source semantic deduplication across all data sources, removing both exact duplicates and conceptually highly similar videos. There is also a non-filtering data-analysis step: analyzing training data for harmful-content and reviewing fairness of demographic representation. [Uncertain] The specific number of stages, execution order, judgment models, and thresholds for each level of filtering are all undisclosed.

### Quantitative Funnel Retention Rates (input/output volume at each filtering stage and final retention rate, e.g., Apollo's 27%) ⚠️

[Uncertain] Entirely undisclosed. Google gives no input/output data volume, retention rate, or final retention proportion for any single stage of filtering, making it impossible to compare against works with explicit, quantified funnels such as Apollo (27%). This is the most thorough blank in Veo 3's data disclosure.

### Shot Segmentation Method (PySceneDetect/in-house model/shot-aware splitting) ⚠️

[Uncertain] The shot-segmentation method is undisclosed, with no mention of PySceneDetect or any in-house shot-aware splitting scheme. Counter-evidence is that the model spontaneously produces shot cuts within its 8-second generations, indicating that the training samples were not strictly segmented down to single-shot granularity, or at least retained a considerable proportion of multi-shot samples.

### Quality Filtering (aesthetic score, clarity, OCR text filtering, letterbox/watermark/logo detection) ⚠️

The only official statement is "Training videos were also filtered for various compliance and safety metrics and for quality," confirming the existence of a quality-filtering stage. [Uncertain] The specific methods are entirely undisclosed: whether an aesthetic-scoring model is used, clarity/blur metrics, OCR text-coverage filtering, letterbox detection, watermark/logo detection, compression-artifact detection, etc. are all unstated, and no threshold values of any kind are given. A weak reverse-inference: the technical report acknowledges Veo 3 "remains poor at generating text," which is directionally consistent with (though not proof of) the data side having performed aggressive OCR text filtering — if it had, this would further weaken text-rendering ability, but the two cannot be used as evidence for each other.

### Motion Filtering (optical-flow/motion-score thresholds, removal of static or shaky footage) ⚠️

[Uncertain] Motion filtering is not mentioned at all. Google discloses no optical-flow computation, motion-score threshold, static-shot removal, or shake/handheld-jitter removal strategy. It can only be weakly speculated from capability descriptions such as "the model simulates real-world physics well" and "high-precision motion representation" that some form of motion-quality screening exists, but there is no direct evidence.

### Deduplication Method (exact deduplication and embedding-based semantic deduplication recorded separately) ⚠️

One of the few officially disclosed items that is relatively specific. Model Card: "All training data was deduplicated semantically across various sources." Technical report: "All data is deduplicated semantically across sources to minimize the risk of outputs overfitting particular elements of training data." The Mitigations section additionally states: "removing duplicated and/or conceptually similar videos." What can be confirmed: cross-data-source semantic-level deduplication (semantic dedup, presumably implemented via embedding similarity), removing not only exact duplicates but also videos that are "conceptually highly similar." The motivation for deduplication is explicitly stated as reducing the risk of outputs overfitting/memorizing particular elements of the training data (i.e., mitigating verbatim reproduction and copyright risk). [Uncertain] The embedding model used, the similarity threshold, and the clustering method are undisclosed, and the respective contributions of exact deduplication (e.g., perceptual hashing) and semantic deduplication are not distinguished.

### VLM/LLM as Quality Inspector (multimodal large-model quality scoring and mismatch removal; the 2026 trend of shifting from shallow scorers to large-model semantic judgment) ⚠️

Veo 3 is a typical representative of the trend of "large models deeply involved in the data pipeline," but its form of involvement is primarily "annotation" rather than "scoring." Confirmed items: (1) multiple Gemini models are used to generate captions at varying levels of granularity for audio-video data — direct evidence of a VLM serving as a data producer; (2) a multimodal classifier is used to detect content-policy violations, with the official documentation specifically emphasizing the necessity of multimodality — "captions and video may each be harmless in isolation, but harmful when combined" (example given: a text prompt of "an image of a pig" paired with a video of a certain demographic group would constitute harmful representation) — this is essentially a cross-modal semantic-mismatch/harmful-combination discriminator, conceptually very similar to "VLM as data quality inspector," though officially positioned within development-stage safety monitoring (development evaluations) rather than an explicit training-data-cleaning stage. [Uncertain] Whether Gemini/a VLM is used to score video aesthetically or semantically, whether a large model is used to remove caption-video semantic-mismatch samples, and the scale and threshold of any scoring model are all undisclosed.

### Safety and Compliance Filtering (NSFW, copyright, face/privacy) ⚠️

One of the more thoroughly disclosed items. Pre-training-stage mitigations include: safety filtering of pretraining data by risk area; filtering captions for unsafe content and personally identifiable information (PII); filtering training videos by compliance and safety metrics; analyzing training data for harmful content and reviewing fairness of demographic representation; and removing duplicated and conceptually similar videos. Risk areas covered include child sexual abuse and exploitation material (CSAM), hate speech, harassment, misinformation, deepfakes, sexual content, and graphic violence. Post-training-stage mitigations include SynthID watermarking and production-environment output filtering. The safety strategy is consistent with Google's cross-product generative-AI framework and the Gemini / Imagen 3 technical reports, and was approved before release by the Google DeepMind Responsibility and Safety Council (RSC). [Uncertain] The specific classifiers, decision thresholds, and the proportion of data filtered out are undisclosed; face/privacy handling covers only PII on the caption side — video-side facial-privacy handling strategy is unstated.

## Annotation Methods

### Caption Model Used (in-house VLM/open-source model, model scale) ⚠️

"Multiple Gemini models" are used to text-caption-annotate audio and video clips — a core data-processing measure the official documentation repeatedly emphasizes in both the technical report and the Model Card. Using multiple models rather than a single one corresponds to the multi-granularity annotation requirement (presumably, a larger model produces long, dense descriptions while a smaller model produces short descriptions to control cost). [Uncertain] The specific Gemini version (1.5 / 2.0 / 2.5 series), parameter scale, whether it is an internal variant fine-tuned specifically for video annotation, and annotation throughput and cost are all undisclosed.

### Caption Density and Degree of Structuring (short/long/dense descriptions, structured fields such as camera motion, style tags) ⚠️

Two points can be confirmed: (1) multi-granularity — "annotated with text captions at different levels of detail," i.e., each piece of data is paired with multiple captions ranging from short to long / coarse to fine, a standard practice for improving robustness to both short and long prompts; (2) synthetic captions for concept expansion — "We generate synthetic captions to improve the variety and diversity of concepts associated with videos in the training data," i.e., rather than relying on original alt-text/titles, captions are rewritten and generated by Gemini, thereby covering concept dimensions such as photographic technique, action, style, and scene context that the original metadata does not include. This corroborates the model's ability to precisely respond to combinations of "style, camera angle, and camerawork." [Uncertain] Whether a structured field schema exists (e.g., explicit tag slots for camera motion / shot size / lighting / style), the average caption length, the number of captions per data item, and the sampling proportion of multi-granularity captions during training are all undisclosed.

### Joint Audio-Video Caption Structure (whether both visual and auditory tracks are covered, whether split into independent fields, e.g., LTX-2's full soundscape description, Script-a-Video's factorized streams, Foley-Omni's three fields) ⚠️

[Uncertain] The official statement is "Audio and video clips were annotated with text captions at different levels of detail," i.e., both audio and video are text-annotated, indicating the captions cover both the auditory and visual tracks; the model can respond separately to dialogue in quotation marks, sound-effect descriptions, and ambient-sound descriptions within a prompt, which also suggests these three types of auditory information are explicitly described in the training captions. But the official documentation never states the organizational form — whether merged into a single "full soundscape" description as in LTX-2, split into factorized streams as in Script-a-Video, or split into three independent fields as in Foley-Omni; whether the audio and video captions share the same text, and whether they are injected into different cross-attention branches separately, are both undisclosed.

### Dialogue Transcription and Speaker-Attribute Annotation (ASR transcription + speaker identity/language/accent/emotion) ⚠️

[Uncertain] No ASR transcription workflow or speaker-attribute annotation system is disclosed. Reverse-inference clue: the model can precisely generate corresponding dialogue matching quoted text in the prompt while maintaining lip sync, indicating the training data must contain verbatim dialogue transcription (most likely produced via Gemini's audio-understanding capability or an internal Google ASR system), aligned to the video timeline. But the official documentation, when assessing deepfake risk, explicitly notes the model "is much less controllable — particularly with respect to speech" and does not offer fine-grained control over accent or timbre, strongly suggesting that speaker identity, timbre, accent, and emotion were not systematically annotated as independent conditioning fields.

### Geometric and Structured Annotation (camera parameters, depth, 3D point tracks, action annotation, explicit state) ⚠️

[Uncertain] No geometric or structured annotation of any kind is disclosed. There is no mention of camera parameters (intrinsics/extrinsics/pose), depth maps, 3D point tracks, optical flow, action labels, or explicit physical-state annotation. Two points of reverse inference: (1) Veo 3.1 offers explicit "camera controls (precise framing and camerawork control)" as a product capability, suggesting some form of camera-motion annotation exists in the training data (most likely still implemented via natural-language camera-motion descriptions within captions, rather than explicit camera parameters); (2) DeepMind's own paper "Video models are zero-shot learners and reasoners" (arXiv:2509.20328) tests Veo 3 across 62 visual tasks and finds it zero-shot capable of segmentation, edge detection, keypoint localization, super-resolution, deblurring, denoising, and more, and proposes the concept of "chain-of-frames" — the authors emphasize that none of these tasks was explicitly trained for, instead suggesting these geometric/structural capabilities emerge from large-scale natural video rather than from explicit geometric annotation.

### Synthetic Data Construction (controlled perturbation/editing to build training pairs, e.g., InstructAV2AV) ⚠️

The only confirmed synthesis is on the text side: synthetic captions generated by Gemini, used to expand the diversity of concepts associated with videos. [Uncertain] No visual- or audio-side synthetic-data construction is disclosed; there is no mention of constructing training pairs via controlled perturbation, or of instruction-style audio-video edit pairs (as in InstructAV2AV-style construction). Veo 3.1 has editing capabilities such as Insert/Remove object insertion and deletion, first/last-frame transitions, outpainting, and style reference, which typically require paired before/after-editing training samples — a synthetic construction pipeline very likely exists, but there is no official statement on this.

### Degree of Human Involvement (human annotation, human quality inspection, model pre-screening + human review) ⚠️

Confirmed human involvement is concentrated on the evaluation and safety side rather than the data-annotation side: (1) model performance is evaluated by human raters doing head-to-head preference comparisons (MovieGenBench, VBench-I2V); (2) red-teaming is conducted by a mix of an internal expert team and externally recruited participants, spanning the entire model-development process; (3) assurance evaluations are developed and executed by a team dedicated and independent of the model-development team, with evaluation datasets strictly held out; (4) bias evaluation involves human analysis — generating 16 videos for each of 140 occupations, then categorizing by perceived skin tone (Monk Skin Tone Scale), perceived age, and perceived gender; (5) pre-release approval is manually reviewed by the Google DeepMind Responsibility and Safety Council (RSC). [Uncertain] The scale of human annotation applied to the training data itself, the human spot-check proportion for quality inspection, and whether a "model pre-screening + human review" process exists are all undisclosed — from the wording, captioning and filtering appear to be essentially fully automated and model-driven.

## Audio-Video Alignment

### Audio-Video Synchronization Detection Method (lip sync, event alignment) ⚠️

[Uncertain] The data-side audio-video synchronization detection method is entirely undisclosed. Officially, synchronization is explained only at the architectural level: video and audio latents are jointly denoised within the same diffusion process, so synchronization is guaranteed by the architecture rather than by post-hoc alignment. This implies that the training data must itself already be strictly synchronized, native paired audio-video, and therefore some form of synchronization-quality screening (lip sync, event alignment) must exist, but Google discloses no detection method for it. Officially, only at the effect level does it mention that Veo 3.1 performs best on the audio-video synchronization dimension.

### Specific Synchronization Metrics and Thresholds (SyncNet/Synchformer/LSE/in-house, threshold values, e.g., MOVA's LSE-D≤9.5 and LSE-C≥4.5, SkyReels-V4's SyncNet |offset|≤3∧conf>1.5) ⚠️

[Uncertain] Entirely undisclosed. No mention of SyncNet, AV-align, or any in-house synchronization metric, and no confidence threshold values whatsoever (compare with the publicly disclosed SyncNet conf>0.9-type threshold from UniTalking).

### Separate Treatment of Temporal vs. Semantic Synchronization (temporal alignment and content-semantic matching as two independent filtering conditions) ⚠️

[Uncertain] It is undisclosed whether "temporal alignment" and "content-semantic matching" are split into two independent filtering conditions. An indirect clue can be drawn from the design thinking behind the safety-side multimodal classifier — the official documentation explicitly notes the need to judge the semantic outcome of a caption-video combination, showing Google does indeed attend to cross-modal semantic-matching issues on the data side, but this mechanism serves safety rather than synchronization quality.

### Audio Quality Filtering (SNR, silence detection and silence-proportion thresholds, removal of tracks with no audio, removal of off-screen audio sources, background-music separation) ⚠️

[Uncertain] Entirely undisclosed. No SNR threshold, silence detection or silence-proportion threshold, rule for removing samples with no audio track, removal of off-screen/narration audio sources, or background-music separation (BGM separation) of any kind is mentioned. It can only be inferred from the general statement that "training videos are filtered for quality" that audio quality is presumably included.

### Classification and Separate Treatment Strategy for Speech/Sound Effects/Music ⚠️

[Uncertain] The classification and separate-treatment strategy for speech/sound effects/music is undisclosed. At the output level, the model clearly distinguishes dialogue, sound effects, ambient sound, and background music, each separately controllable via prompt, suggesting these categories are distinguished in the training-data captions; but whether an independent audio classifier exists, and whether different filtering thresholds or training weights are applied to each category, is entirely undisclosed.

## Training Coordination

### Multi-Stage Training Curriculum and Data-Curriculum Scheduling (basis for stage division: resolution/duration/quality score/modality; low-res→high-res, image→video, short→long) ⚠️

[Uncertain] The training curriculum and data-curriculum scheduling are undisclosed. Officially, only "pre-training" and "post-training" are distinguished, in a safety context, with no statement of whether stages are divided by resolution (low-res→high-res), duration (short→long), modality (image→video→audio-video), or quality score. A weak reverse-inference: the training data includes images, and the model supports multiple output tiers of 720p/1080p/4K and 4/6/8-second durations, consistent with the industry-common "image pretraining → low-resolution video → high-resolution video" progressive curriculum, but there is no official basis for this.

### Change in Data Mixture Across Training Stages (pretraining/annealing/high-quality SFT subset) ⚠️

[Uncertain] Entirely undisclosed. There is no data-mixture information for pretraining/annealing/SFT stages, nor any statement of whether a high-quality-subset annealing stage exists.

### Post-Training Data (SFT curated-set scale and screening criteria, number of preference pairs and annotation method, reward-model training data) ⚠️

[Uncertain] Post-training data is entirely undisclosed. The "post-training mitigations" mentioned officially refer only to SynthID watermarking and production-environment output filtering — deployment-side interventions, not data-side post-training. The SFT curated-set scale and screening criteria, the number of preference pairs and their annotation method, and reward-model training data are all undisclosed, and whether RLHF/DPO-style preference optimization is used is not confirmed. It is only known that one of the model's development goals is to "maximize adherence to a user's request," following Gemini's desiderata-optimization approach, hinting that some form of preference-alignment process exists.

### Data-Processing Infrastructure and Throughput (NeMo Curator/Data-Juicer/in-house, GPU acceleration ratio, processing scale, cost) ⚠️

The training infrastructure is confirmed: Google TPUs (TPU Pod clusters) are used, with a software stack of JAX and ML Pathways; the official documentation emphasizes that TPUs' high-bandwidth memory supports large models and large batches, and that training can be distributed across multiple TPU devices. [Uncertain] The data-processing-side infrastructure is entirely undisclosed: no mention of NeMo Curator, Data-Juicer, or an in-house data-processing framework, no GPU/TPU acceleration ratio, no processing throughput, and no processing scale or cost data. It can be speculated that caption annotation is performed via large-scale batch inference with Gemini, and that this is likely the primary cost of the data pipeline, but there is no quantitative information of any kind.

## Comparative Results

### Quantitative Impact of Data-Strategy Ablations (distinguishing filtering-strictness ablation / caption-density-and-style ablation / data-mixture ablation, and corresponding evaluation metrics) ⚠️

[Uncertain] Google has published no data-strategy ablation experiments. The technical report and Model Card contain no filtering-strictness ablation, no caption-density/style ablation, no data-mixture ablation, and no corresponding evaluation metrics for any of these. This is the largest disclosure gap between Veo 3 and open-source/semi-open models (such as Movie Gen, Seedance, and the Wan series). The only statements that could be classified as "evidence of data-strategy effectiveness" are qualitative: synthetic captions are used to improve concept diversity, and semantic deduplication is used to reduce the risk of outputs overfitting particular elements of the training data — but neither is accompanied by any quantitative comparison.

### Evidence for Quality vs. Quantity (cases where small, curated data outperforms large, messy data) ⚠️

[Uncertain] No quality-vs-quantity comparative evidence exists. Google states only in general terms that the training data is "large" and has undergone quality filtering, without providing any case or ablation conclusion showing small, curated data outperforming large, messy data.

### Alignment Between Training-Data Domain Distribution and Evaluation-Benchmark Taxonomy (e.g., VABench's seven major categories) ⚠️

[Uncertain] The alignment between training-data domain distribution and evaluation-benchmark taxonomy is undisclosed, and Google has not built its own systematic taxonomy-based evaluation benchmark (compare VABench's seven major categories). The confirmed evaluation systems are: (1) Meta's MovieGenBench — 1,003 prompts on the video side, 527 prompts on the video+audio side, compared against MovieGen, Kling 2.0, MiniMax T2V-01, Sora Turbo/OpenAI Sora, Runway Gen-3, WAN 2.1, Kling 2.0 + MMAudio, and others; (2) VBench-I2V — 355 image-text pairs, compared against Runway Gen-4, Kling 2.0 Pro, WAN 2.1, and MiniMax I2V-01. Evaluation uses human raters doing head-to-head preference comparisons; Veo 3 achieves SOTA in overall preference and prompt adherence, and Veo 3.1 is officially stated to perform best in overall preference, text alignment, and visual quality. In addition, the safety evaluation uses a standardized bias-evaluation set of 140 occupations × 16 videos, plus several adversarial safety-prompt datasets; the correspondence between these evaluation categories and the training-data mixture is not stated.

## Uncertain Fields

The research findings for the following fields are partially uncertain (sources marked ⚠️):

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
- model_as_data_judge
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
- data_infra_throughput
- data_ablation
- quality_over_quantity_evidence
- benchmark_taxonomy_alignment
