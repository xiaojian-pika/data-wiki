# Apollo (renamed to Klear starting from arXiv v2; same paper arXiv:2601.04151, v1 titled "Apollo: Unified Multi-Task Audio-Video Joint Generation", v2 titled "Klear: Unified Multi-Task Audio-Video Joint Generation". Note: unrelated to Meta's video understanding model Apollo, Meta's Apollo LMM, or other same-named works — distinguish by arXiv number)

> Topic: Data processing for video generation models (including joint audio-video generation): data cleaning pipeline, data distribution, annotation approach

[← Back to home](../index.md)

**Contents**: [Basic Information](#basic-information) · [Data Scale and Distribution](#data-scale-and-distribution) · [Cleaning Pipeline](#cleaning-pipeline) · [Captioning Approach](#captioning-approach) · [Audio-Video Alignment](#audio-video-alignment) · [Training Coordination](#training-coordination) · [Effectiveness Comparison](#effectiveness-comparison)

## Basic Information

### Name

Apollo (renamed to Klear starting from arXiv v2; same paper arXiv:2601.04151, v1 titled "Apollo: Unified Multi-Task Audio-Video Joint Generation", v2 titled "Klear: Unified Multi-Task Audio-Video Joint Generation". Note: unrelated to Meta's video understanding model Apollo, Meta's Apollo LMM, or other same-named works — distinguish by arXiv number)

### Releasing institution/company ⚠️

Kuaishou Technology's Kling Team. Authors: Jun Wang, Chunyu Qiang, Yuxin Guo, Yiran Wang, Xijuan Zeng, Feng Deng, et al. This work is the research-oriented technical report behind Kuaishou Kling's series of synchronized audio-video generation capability (Kling 2.6 "simultaneous audio-visual generation", Kling 3.0), representing an industry closed-source model's external paper disclosure. [Uncertain] (the paper itself does not explicitly state the correspondence with the Kling product line; this association is a circumstantial inference from the HuggingFace paper page's authorship attribution "Kling Team, Kuaishou Technology" and the product timeline)

### Release date (technical report/paper/open-source date)

First submitted to arXiv on January 7, 2026 (v1, titled Apollo); updated to v2 on January 13, 2026 (title changed to Klear). The HuggingFace paper page indexing date is January 8, 2026. There is no independent open-source release date (not open-sourced).

### Type (model/dataset/toolchain/evaluation benchmark)

Model (a unified multi-task audio-video joint generation foundation model, 26B parameters). Accompanying output: the paper claims to have constructed "the first large-scale audio-video dataset with dense captions" (81M samples) and a companion automated data-construction pipeline, but neither the dataset nor the pipeline has been released, so it does not constitute a usable dataset deliverable. Evaluation follows the third-party benchmark Verse-Bench, not a self-built benchmark.

### Degree of openness (whether weights/code/data/pipeline are each open source)

Completely closed-source — the lowest degree of openness among the samples surveyed in this research.
[Weights] Not open source; neither the paper nor the HuggingFace paper page contains a model link.
[Code] Not open source; the paper gives no GitHub repository or project homepage URL.
[Data] The 81M-sample audio-video-caption triplet dataset has not been released; the paper's positioning of it as "the first large-scale audio-video dataset with dense captions" is only a contribution claim, with no public commitment of any kind.
[Pipeline] Framework-level disclosure was made at the methodological level (the four-stage funnel, the list of captioning models used, the alignment-detection tools, the 27% retention rate), but the key elements needed for reproducibility are missing: no per-metric threshold table, no per-stage input/output volumes, no verbatim prompts, no pseudocode, no data-processing scripts. Compared with concurrent open-source works (e.g., MOVA publishing the full Table 9 threshold table and the verbatim captioning prompts), Apollo's data disclosure stays at the level of "stating what was done, without saying how it was done"; the only respect in which it exceeds its peers is providing the 27% end-to-end quantitative retention rate.

### Whether joint audio-video generation is supported, and the implementation approach (native joint/cascade/MoE fusion)

Supported, and it is native joint generation (a single-tower single forward pass simultaneously produces audio and video latents, not cascaded). Implementation approach:
- A Single-Tower MultiModal Diffusion Transformer (MMDiT) architecture, containing 32 joint diffusion layers, in which audio and video share the same set of DiT block parameters, rather than a dual-tower + cross-attention design.
- The core mechanism is Omni-Full Attention: audio tokens and video tokens undergo fully connected self-attention within the same attention window, achieving tight audio-visual alignment and good scalability.
- Positional encoding uses MixD-RoPE (Mixed Dimension Rotary Position Embedding), uniformly handling the video's 3 dimensions (t, h, w) and the audio's 1-dimensional temporal-axis index.
- The training objective is flow matching (conditional denoising).
- The model has 26B total parameters, with a flow-matching FFN dimension of 4096.
- Input consists of four streams: video, video-related text, audio-related text, and audio, each independently encoded before being fed into the MM-DiT.
[Evidence versus the dual-tower route] Paper Table 2 directly compares "Dual Tower (standard cross-attention)" against "Single Tower (Omni-Full Attention)"; the conclusion supports the single-tower full-attention scheme, marking a clear architectural divergence from dual-tower + bridge routes such as MOVA/HunyuanVideo-Foley.
[Multi-task unification] Random modality masking is used to uniformly support five task types — T2A, T2V, T2AV, I2V, I2AV — within the same model, so the same set of weights can perform both joint generation and single-modality generation.

### List of research information sources (URLs of papers/technical reports/official documentation/news, with each source's nature annotated: official first-hand/same-team corroboration/third-party report)

- [Official first-hand] arXiv:2601.04151 "Apollo: Unified Multi-Task Audio-Video Joint Generation" (v1, 2026-01-07) / "Klear: Unified Multi-Task Audio-Video Joint Generation" (v2, 2026-01-13): abstract page https://arxiv.org/abs/2601.04151 , HTML full text https://arxiv.org/html/2601.04151v2 and https://arxiv.org/html/2601.04151v1 , PDF https://arxiv.org/pdf/2601.04151 — the sole direct source for almost all fields in this entry; data-related content is concentrated in Section 4 Dataset Construction (4.1 Dataset Filtering, 4.2 Audio-Guided Data Splitting, 4.3 Dense Annotation and Integration) and Figure 3, the data-annotation pipeline diagram. This section is extremely short (about one page), which is the root cause of a large number of fields in this entry being marked uncertain.
- [Official first-hand] HuggingFace paper page https://huggingface.co/papers/2601.04151 — confirms the attributed unit as "Kling Team, Kuaishou Technology," confirms that no model, dataset, or code repository link is attached.
- [Same-team corroboration] Kuaishou investor-relations announcement "Kling AI Launches Video 2.6 Model with 'Simultaneous Audio-Visual Generation' Capability": https://ir.kuaishou.com/news-releases/news-release-details/kling-ai-launches-video-26-model-simultaneous-audio-visual — used to corroborate the correspondence between this research and the Kling product line's simultaneous audio-visual generation capability (circumstantial, not explicitly stated by the paper).
- [Third-party index] NASA ADS entry https://ui.adsabs.harvard.edu/abs/2026arXiv260104151W/abstract , ResearchGate entry https://www.researchgate.net/publication/399559825_Klear_Unified_Multi-Task_Audio-Video_Joint_Generation — used to cross-check the fact of the Apollo→Klear renaming and the author list.

## Data Scale and Distribution

### Training data scale (video count/hours/token count, pretraining vs SFT split separately) ⚠️

[Total volume] 81 million audio-video samples with accurate dense captions — this is the only scale figure the paper provides, corresponding to the final filtered training set (paper's original text: "81 million samples with accurate dense captions").
[Unit note] The paper uses "sample count" as its sole unit, giving no total hour count, no token count, and no per-sample duration, so it cannot be converted into an hour-level figure. If roughly estimated using the 5–10 second clip length common in similar works, 81M samples would correspond to roughly 110,000–220,000 hours, but this is an extrapolation, not paper-reported data.
[Stage breakdown] None of the paper's three training stages (pretraining / specialized post-training / high-quality post-training) is given a per-stage data scale, nor is a separate figure given for pretraining versus SFT. The scale of the "manually-curated, high-quality dataset" used in Stage III is entirely undisclosed.
[Category breakdown] The respective counts or proportions of the four categories within the 81M — single-speaker speech, multi-speaker speech, singing, and natural sound — are not given.
[Conclusion] Apollo discloses only one total figure along the data-scale dimension, at a granularity notably lower than that of concurrent works. [Uncertain]

### Data source composition (proprietary/public datasets/web crawling/licensed procurement/synthetic data) ⚠️

The paper's disclosure of data source composition is essentially blank: no public dataset names are listed, no breakdown is given of the respective shares of proprietary data, web crawling, and licensed procurement, and no data-acquisition channel is mentioned. Limited inferences can be drawn:
- Data forms cover four scenario types — single-speaker speech, multi-speaker speech, singing, and natural sound — and require a native synchronized audio track (since the entire pipeline is built on quality and synchronization filtering of the audio tracks of existing videos, rather than dubbing added after the fact to silent video), indicating the data comes from real videos with sound rather than silent video with audio added later.
- The 81-million-sample scale, together with Kuaishou's identity as a short-video platform, strongly suggests that the data is predominantly the platform's own/licensed short-video corpus, but the paper makes no statement to this effect.
- Synthetic data: the paper does not use model-synthesized audio-video content; the only "synthetic" component is that all captions are automatically generated by ASR/audio captioners/video expert models (synthetic annotation, not synthetic content).
[Uncertain]

### Data compliance and provenance (proportion of licensed data, rights-cleared datasets, C2PA, etc.) ⚠️

The paper does not address data compliance and provenance at all: no proportion of licensed data is given, no rights-cleared dataset is declared, no C2PA or any output-side watermarking/provenance marker is mentioned, no discussion of copyright, portrait rights, or the legal status of the data, and no model-card-level terms of use. The word "safety," which appears in the data-filtering list, is the only expression that touches on compliance, but it is not elaborated at all. As a closed-source industrial model, its compliance work may exist internally but is not disclosed externally. [Uncertain]

### Clip duration distribution and segmentation strategy ⚠️

The paper discloses no clip-duration information at all: no fixed duration or duration distribution for training clips is given, no frame rate is given, no minimum/maximum duration threshold is given, and it is not stated whether segmentation is fixed-length or variable-length bucketing. The only indirect constraint comes from the VAE specification: the Video-VAE outputs a 3 Hz temporal embedding (i.e., 3 latent frames per second), and the Audio-VAE outputs a 43 Hz embedding; the paper explicitly states the Video-VAE "handles input videos with varying resolutions and frame rates," indicating the input-side frame rate is not uniform and is normalized to a 3 Hz representation by the VAE. The only confirmable point at the segmentation-strategy level is scene splitting, which ensures each sample contains only a single scene, but the window-length rule after splitting is not described. [Uncertain]

### Resolution/aspect-ratio distribution and bucketing strategy ⚠️

The paper discloses no distribution or bucketing strategy for resolution and aspect ratio: no training resolution is given, no aspect-ratio enumeration is given, and no multi-resolution bucketing or progressive upscaling curriculum is mentioned. Only two related points exist: (1) the filtering stage will "discard those videos with low resolution," but no lower resolution bound is given; (2) the Video-VAE adopts CogVideoX's 3D causal visual encoder, compressing both height and width by a factor of 16, and states it can handle "varying resolutions and frame rates," i.e., architecturally it supports variable-resolution input, but the resolution configuration actually used in training is not disclosed. [Uncertain]

### Category/domain distribution and mixture strategy (proportional control and concept balancing across people, actions, scenes, styles, etc.) ⚠️

The paper gives no description whatsoever of the distribution or mixture strategy across visual domains (people, actions, scenes, styles, etc.), nor does it mention any concept-balancing mechanism. The only "distribution" concept in the paper is the four scenario categories split by audio type (single-speaker speech / multi-speaker speech / singing / natural sound) — that is, the axis around which Apollo organizes its data is **audio type, not visual domain**. This is the most salient orientation of this work's data design: it treats "what is heard" rather than "what is seen" as the first-order dimension for data partitioning (see audio_category_distribution).
The indirectly confirmable mixture intent comes from the training strategy rather than data statistics: Stage II, "specialized post-training," "adaptively rebalance[s] data distributions across scenarios and tasks to strengthen underperforming capabilities while preserving overall competency" (dynamically rebalancing data distribution across scenarios and tasks based on evaluation metrics, to shore up weak capabilities while not damaging overall competency) — indicating a metric-driven dynamic mixture mechanism exists, but the specific scenario list, initial mixture, and post-adjustment mixture are all given no numbers. [Uncertain]

### Audio category distribution and mixture (proportions and control strategy for speech/sound-effects-foley/music/ambient sound/silence — a dimension unique to AV models) ⚠️

This is the core dimension of Apollo's data design and also the key point that distinguishes it from vision-first peer works — the entire dataset is split hierarchically, tree-like, by audio type (Section 4.2 Audio-Guided Data Splitting):
[First tier: vocal vs. non-vocal] Data is first divided into vocal (containing human voice) and non-vocal branches; the non-vocal branch forms the sound split (the natural-sound/sound-effects subset).
[Second tier: three-way split within vocal] The vocal subset is further divided into three categories — singing, single-speaker speech, and multi-speaker speech.
[Final four categories] single-speaker speech / multi-speaker speech / singing / natural sound; paper's original text: "The dataset contains single-speaker speech, multi-speaker speech, singing, and natural sound clips."
[Differentiated annotation] Different subsets follow different annotation paths — the speech and singing subsets additionally extract speaker attributes (gender, age) and undergo word-by-word transcription; the sound split only receives audio captions, without transcription. This is a typical design of "splitting by audio category, then annotating each branch separately."
[Comparison with peers] Notably, Apollo explicitly retains singing as a category, whereas MOVA and similar works, limited by audio-tower capacity, degrade on singing voice and exclude or downweight it; Apollo makes singing a first-tier subset, indicating that on the data side it deliberately covers the high-difficulty scenario of singing lip movement. At the same time, unlike MOVA, it does not "keep only speech" — instead it lets natural sound (foley/ambient sound) coexist with speech in the same training set, jointly acquiring T2A capability and lip-sync capability alongside multi-task masked training.
[Gaps] The respective sample counts or proportions of the four categories, whether music (instrumental music) is listed separately, and the handling of silent samples (only known: samples with a silence ratio > 20% are excluded) are all undisclosed. [Uncertain] (only the category taxonomy is clear; proportion figures are missing)

### Narrative structure distribution (single-shot vs. multi-shot, average clip duration, shot-count distribution, whether native audio track is included) ⚠️

[Single-shot vs. multi-shot] A pure single-shot strategy is explicitly adopted: the paper's original text states, "We then apply scene splitting to ensure each sample contains only one scene." That is, all training data consists of single-shot clips, with no cross-shot transition samples included. This contrasts with MOVA's approach of deliberately constructing a "single-scene/multi-scene" 2×2 data dimension, and also means Apollo does not model shot-transition capability on the data side.
[Native audio track] Training data must carry a native synchronized audio track — both the audio-filtering and audio-video consistency detection stages of the entire pipeline operate on the original audio track, and samples with no audio track or an abnormal audio track are excluded during filtering.
[Average clip duration / shot-count distribution] Undisclosed. [Uncertain] (the duration figures are missing; the single-shot conclusion is certain)

### Language/accent distribution (data foundation for multilingual lip-sync capability) ⚠️

The paper does not discuss language and accent distribution at all: no supported languages are listed, no per-language proportions are given, and neither accent annotation nor the data foundation for multilingual lip sync is mentioned. Indirect inferences that can be drawn: the transcription step uses Whisper-Large-v3 (multilingual ASR), SenseVoice (Alibaba's open-source multilingual speech-understanding model, spanning Chinese, English, Japanese, Korean, and Cantonese, with notably strong support for Chinese and Chinese dialects), and Qwen2.5-Omni (strong in both Chinese and English) together — this combination strongly suggests the corpus covers at least Chinese and English bilingually, and the introduction of SenseVoice points to a substantial proportion of Chinese/dialect material; the caption text encoder being Qwen2.5-7B (a Chinese-English bilingual model) also supports this inference. But all of this is a tool-chain-based inference rather than a statement in the paper. The evaluation metrics include WER (word error rate), indicating a transcription-accuracy evaluation exists, but the evaluation language is not stated. [Uncertain]

## Cleaning Pipeline

### Overall structure of the cleaning funnel (number of filtering stages, ordering)

A four-stage serial funnel (paper Section 4 + Figure 3, "Overview of our Dataset Annotation Pipeline"):
[Stage 1 — Video Filtering and Scene Splitting] First models video quality along four dimensions: dynamic quality (subject motion ratio, camera stability), static quality (sharpness, aesthetics, color saturation), content naturalness (no excessive effects/watermarks), and safety; excludes videos with low resolution, low SNR/MOS, or a silence ratio exceeding 20%; then performs scene splitting to ensure each sample contains only a single scene.
[Stage 2 — Audio Filtering and Post Processing] Excludes samples with low SNR, low MOS, abnormal clipping, distortion, or noise; requires a silence ratio < 20%, high fidelity, and consistent formatting; then performs audio-video consistency detection — Synchformer for temporal alignment, ImageBind for semantic alignment — ensuring high synchronization along both the temporal and semantic dimensions.
[Stage 3 — Audio-Guided Data Splitting] Stratifies the dataset by audio type: first splits into vocal/non-vocal to obtain the sound split, then splits the vocal subset into singing, single-speaker speech, and multi-speaker speech.
[Stage 4 — Dense Annotation and Integration] Invokes dedicated models per subset to produce speech transcripts, audio captions, and video captions (including meta information and detailed content); the speech and singing subsets additionally extract speaker attributes (gender, age); the sound split receives only audio captions; finally, all annotations are merged into unified dense captions.
[Funnel ordering characteristic] Consistent with MOVA, the most expensive multimodal captioning is placed at the very end of the funnel, applied only to samples that pass quality and synchronization filtering; the difference is that Apollo inserts an extra "split by audio type" layer before captioning, allowing the captioning strategy to be customized per subset — this is a distinctive point of its pipeline structure.
[Limitation of disclosure granularity] All four stages have only qualitative descriptions, with no threshold table, no per-stage sample volumes, and no pseudocode.

### Funnel quantitative retention rate (input/output volume at each filtering stage and the final retention rate, e.g., Apollo 27%)

[Core figure] Overall post-filtering retention rate of 27%, paper's original text: "with an overall post-filtering retention rate of 27%." This is one of the rare publicly disclosed end-to-end quantitative funnel metrics found in this research, on the same order of magnitude as MOVA's 26.39% — the two figures mutually corroborate the empirical pattern that "the data funnel for joint audio-video generation generally retains a bit over a quarter of the candidates."
[Unit basis] The paper does not state whether the 27% is computed by sample count, total duration, or original video count; given the context (immediately following "81 million samples"), it is inferred to be a sample/clip-count basis, but this is not made explicit. If computed by count, the number of candidate samples before filtering would be roughly 300 million (81M ÷ 27% ≈ 300M) — this is a back-calculation, not paper-reported data.
[Missing per-stage figures] Unlike MOVA, which discloses a per-stage retention-rate table (100% → 84.57% → 58.75% → 26.39%), Apollo gives only a single end-to-end total; the respective input/output volumes of the four stages, the elimination proportion at each stage, and how much each sub-filter (video quality / audio quality / audio-video consistency) removes are all undisclosed, making it impossible to locate the primary source of loss.
[Conclusion] 27% is a valuable but isolated anchor figure, useful for cross-work benchmarking but insufficient to support stage-by-stage reproduction of the pipeline.

### Shot-segmentation method (PySceneDetect/in-house model/shot-aware splitting) ⚠️

Confirms the existence of a scene-splitting step with a clear purpose but an undisclosed method: the paper states only, "We then apply scene splitting to ensure each sample contains only one scene," placed after video quality filtering. It does not state whether PySceneDetect, TransNetV2, or an in-house model is used, gives no cut-point detection threshold, does not describe how windows are sampled after splitting, and does not state whether, like MOVA, it jointly performs shot-aware and speech-aware dual-perception splitting by combining speech boundaries (VAD) with scene cut points. Given that Apollo's data is predominantly speech/singing, if splitting is not speech-boundary-aware it could cause utterances to be cut off mid-sentence, but the paper makes no statement on this. [Uncertain]

### Quality filtering (aesthetic scoring, sharpness, OCR text filtering, black-border/watermark/logo detection) ⚠️

Split into a video-side line and an audio-side line, with dimensions clearly enumerated but nearly all thresholds missing:
[Video-side four-dimensional quality modeling]
- Dynamic quality: subject motion ratio, camera stability
- Static quality: sharpness, aesthetics, color saturation
- Content naturalness: no excessive effects/watermarks
- Safety
[Video-side hard exclusion conditions] Low resolution, low SNR/MOS, silence ratio exceeding 20% (original text: "We discard those videos with low resolution, low SNR/MOS, or over 20% silence"). Note that SNR/MOS, which properly belongs to audio metrics, appears in the video-filtering clause here, indicating that audio-visual quality is jointly considered at the very first gate.
[The only publicly disclosed numeric threshold] Silence ratio < 20% (silence ratio < 0.2). The scoring model used for each dimension of video quality (whether LAION-Aesthetic, DOVER, MUSIQ, etc.) and the threshold values are all undisclosed; the paper's evaluation section uses MANIQA, aesthetic-predictor-v2-5, and Musiq to compute aesthetic scores, but those are evaluation metrics rather than data filters, and must not be conflated.
[Watermark/logo] Included in the content-naturalness dimension via only the phrase "no excessive effects/watermarks," with no description of the detection method (whether OCR-based, whether a dedicated watermark-detection model) and no statement of whether it results in exclusion or cropping.
[OCR/burned-in subtitles] Not mentioned at all, unlike MOVA's approach of making "presence of burned-in subtitles" an explicit, controllable attribute. [Uncertain] (dimensions are clear; thresholds and methods are missing)

### Motion filtering (optical-flow/motion-score thresholds, exclusion of static and shaky footage) ⚠️

A motion-related filtering dimension exists but with no method or threshold: the paper lists "subject motion ratio" and "camera stability" as two sub-dimensions of dynamic-quality modeling, incorporated into the first video-filtering gate. This means static/lifeless clips (motion ratio too low) and handheld-shake/violently-jittery clips (poor camera stability) are both, by design, addressed, in a direction consistent with common practice. But the paper does not state whether optical flow (RAFT/UniMatch), frame differencing, or a dedicated model is used to compute this, nor does it give upper/lower motion-score bounds. The evaluation stage uses RAFT optical flow to compute Motion Score (MS), but this is an evaluation metric, and the paper does not state whether data filtering reuses the same tool. [Uncertain]

### Deduplication method (exact deduplication and embedding-based semantic deduplication, recorded separately) ⚠️

The paper does not mention a deduplication step at all: neither hash/fingerprint-level exact deduplication nor embedding-based semantic deduplication. At the scale of 81 million entries, from a corpus suspected to derive from a short-video platform, the risk of duplicate content (the same material re-uploaded multiple times, templated re-edits, overlapping clips from the same video) is objectively present, and the paper's scene splitting produces multiple clips from the same long video, naturally introducing near-duplicates — but the paper makes no statement on this. [Uncertain]

### VLM/LLM as quality inspector (multimodal large-model quality scoring and mismatch removal — the 2026 trend of shifting from shallow scorers to large-model semantic judgment) ⚠️

Apollo's data pipeline exhibits an evident "deep large-model involvement" characteristic, but its role is concentrated on the **annotation-generation** side rather than the **quality-adjudication** side, which only partially aligns with the 2026 trend of "shifting from shallow scorers to large-model semantic judgment":
[Large models handle annotation] Qwen2.5-Omni (an omni-modal large model) is used for both speech transcription and audio captioning; Gemini 2.5-Pro (a closed-source commercial flagship multimodal large model) is used for audio captioning — introducing a commercial API-tier large model for large-scale annotation is a rather high-cost choice at the 81-million scale, reflecting Kuaishou's prioritization of annotation quality. On the video side, a "video expert model" (unnamed, presumed to be an internally developed VLM) is used.
[Quality adjudication still handled by dedicated models/signal-processing metrics] Audio quality uses traditional signal and perceptual metrics such as SNR, MOS, and clipping/distortion/noise detection; audio-video alignment uses two dedicated discriminative models, Synchformer (temporal) and ImageBind (semantic); video quality uses an unnamed multi-dimensional scorer. Nowhere in the pipeline does a "use a VLM/LLM to assign a single composite quality score" or "LLM-as-judge for cross-modal consistency adjudication" step appear.
[Key gap] Compared with MOVA's approach of explicitly using GPT-OSS-120B to resolve visual-audio consistency conflicts and building anti-hallucination self-audit fields into its prompts, Apollo's paper contains no description of annotation hallucination suppression, cross-modal crosstalk defense, or secondary verification of annotation results — how the outputs of multiple models (three ASR systems used in parallel: Whisper / SenseVoice / Qwen2.5-Omni) are arbitrated and merged is covered by the paper only with the single sentence "All annotations are merged into unified dense captions," with the merging rules entirely undisclosed.
[Inference] The most likely purpose of using three ASR models in parallel is cross-validation/voting to exclude samples with unreliable transcription (which is in essence a form of model-ensemble quality inspection), but the paper does not confirm this. [Uncertain]

### Safety and compliance filtering (NSFW, copyright, faces/privacy) ⚠️

The paper's disclosure of safety and compliance filtering amounts to a single word: "safety" is listed as the fourth dimension of video-quality modeling (alongside dynamic quality, static quality, and content naturalness), placed at the first filtering gate. There is no elaboration whatsoever — no NSFW detection method or model is described, no copyright filtering is mentioned, no face recognition/privacy protection/celebrity-portrait filtering is mentioned, no safety taxonomy is given, and there is no model-card-level safety statement or usage restriction. As a platform like Kuaishou that must bear content-regulation responsibility, a mature internal safety-review system surely exists, but the paper chooses not to disclose it. [Uncertain]

## Captioning Approach

### Captioning model(s) used (in-house VLM/open-source model, model scale)

Adopts a "split by subset + multiple models used in parallel + unified fusion" annotation-model matrix, entirely composed of open-source or commercial API models, with the video side using an unnamed in-house model:
[Speech/singing transcription (three models used in parallel)] Whisper-Large-v3 (OpenAI, multilingual ASR, ~1.55B), SenseVoice (Alibaba FunAudioLLM, multilingual speech understanding, supporting Chinese, English, Cantonese, Japanese, Korean plus emotion/event recognition, ~234M), Qwen2.5-Omni (Alibaba's omni-modal large model, 7B/3B versions). This combination of three models used together is rare among peer works, and is presumed to serve cross-validation or dispatch by language/scenario.
[Audio captioning (two models used in parallel)] Qwen2.5-Omni (open-source omni-modal) + Gemini 2.5-Pro (Google's closed-source flagship multimodal). Invoking Gemini 2.5-Pro at the 81-million scale represents a significant cost investment, also indicating that some high-value subsets were routed to the stronger model.
[Video captioning] "a video expert model for detailed video labels" — unnamed, no scale given, presumed to be Kuaishou's internally developed video-understanding model (paired with the Kling system).
[Speaker attribute extraction] Attributes such as gender and age are extracted by the above audio-model stack; no dedicated model is separately listed.
[Fusion] "All annotations are merged into unified dense captions" — the fusion executor is not stated to be rule-based concatenation or LLM rewriting — this is a notable difference from MOVA (which explicitly uses GPT-OSS-120B for fusion and consistency verification).
[Downstream text encoding] Captions are fed into Qwen2.5-7B as the text encoder; there is also a dedicated 1024-dimensional TTS text encoder that handles the speech text to be synthesized; v1 additionally mentions Qwen3-8B Embedding.

### Caption density and structuring level (short/long/dense descriptions, structured fields such as camera movement, style tags) ⚠️

[Overall positioning] The paper positions its dataset as "the first large-scale audio–video dataset with dense captions" — density (dense) and accuracy (accurate) are the two key attributes it claims for itself, but no example of the caption's actual form is given.
[Structuring level] The paper explicitly states captions cover two levels: "including both meta information and detailed content." The meta-information layer includes structured fields such as speaker attributes (gender, age); the detailed-content layer is natural-language description. This is a hybrid structure of "structured meta + natural-language body text."
[Conditional branching] Caption content varies by audio subset: the speech/singing subsets include transcript text + speaker attributes + audio caption + video caption; the sound split includes only audio caption + video caption, without transcript or speaker attributes. That is, the caption schema is dynamically pruned depending on the data category.
[Final form] "merged into unified dense captions" — all annotations are ultimately merged into a unified dense caption, with no separated fields retained (consistent with MOVA's path of "split during annotation, fused during training").
[Model-side conditioning channels] Notably, at the model-architecture level, the input consists of four streams rather than one: video, video-related text, audio, and audio-related text — i.e., the video caption and the audio caption remain **two separate text conditions at the input-channel level**, not merged into a single prompt. This is in tension with the "unified dense captions" phrasing; a possible explanation is that a unified caption is generated on the data side and then split back into two streams during training, or the two phrasings describe different stages.
[Key gaps] Caption length distribution, word-count statistics, verbatim prompts, complete caption examples, and whether structured fields such as camera movement/style tags are included — all undisclosed. [Uncertain]

### Joint audio-video caption structure (whether both visual and auditory tracks are covered simultaneously, whether split into separate fields — e.g., LTX-2's full-soundscape description, Script-a-Video's factorized streams, Foley-Omni's three fields) ⚠️

Apollo adopts a scheme of "split annotation by audio type → each track produced independently → merged into unified dense captions," but at the model-input side it also retains separate visual/auditory text channels, making it a hybrid form:
[Annotation track division] Three types of annotation are produced in parallel — (1) speech transcripts (vocal subset only); (2) audio captions (all subsets); (3) video captions (all subsets). The speech and singing subsets additionally attach speaker attributes (gender, age) as meta fields.
[Coordinates relative to peer schemes]
- Compared with Foley-Omni's three fields, which are kept permanently parallel and separate, Apollo performs a fusion on the data side (merged into unified dense captions).
- Compared with LTX-2's single full-soundscape description, Apollo retains the three-track source structure of "transcript / audio description / video description" and a conditional schema pruned by subset.
- Compared with Script-a-Video's factorized streams, Apollo's splitting basis is **audio content type (vocal/non-vocal, single-speaker/multi-speaker/singing)** rather than narrative elements — this is the most distinctive aspect of its schema design: the schema itself changes with the audio category.
- Compared with MOVA's "three strictly mutually exclusive-prompt tracks + 120B-LLM fusion with cross-modal consistency adjudication," Apollo discloses no mutual-exclusion constraints between tracks (such as prohibiting the visual annotator from referencing audio) or fusion rules — the fusion step is a black box.
[Model-side dual text channels] Architecturally, audio-related text and video-related text are two separate inputs, each encoded independently before being fed into the MM-DiT, so at inference time instructions can be issued separately to the audio and video sides (this is precisely the basis for the paper's emphasis on "instruction following in both joint and unimodal settings"), and it also supports the unification of T2A / T2V / T2AV multi-task capability.
[Gaps] Fusion rules, field names, schema definitions, and examples are all undisclosed. [Uncertain] (the track-split structure is confirmed; schema details are missing)

### Dialogue transcription and speaker attribute annotation (ASR transcription + speaker identity/language/accent/emotion) ⚠️

This is the segment of Apollo's data system with relatively the most substantive disclosure, and also a differentiated investment relative to peer works:
[Transcription] Speech transcription is performed on the vocal subset (including singing, single-speaker speech, and multi-speaker speech), with three models used in parallel: Whisper-Large-v3, SenseVoice, Qwen2.5-Omni. The three have complementary capabilities (Whisper: general multilingual coverage; SenseVoice: Chinese/dialects and emotion-event recognition; Qwen2.5-Omni: omni-modal contextual understanding), presumed to be used for cross-validation or dispatch by scenario, but the arbitration rule is undisclosed.
[Speaker attributes] Speaker attributes are explicitly extracted, exemplified as gender and age: "For speech and singing, we extract speaker attributes (e.g., gender, age)." The "e.g." indicates the attribute set is not limited to these two, but the complete list is not given. Compared with MOVA's approach of describing timbre/accent/speech rate in natural language, Apollo's gender and age are closer to discrete structured fields.
[Speaker-count dimension] Expressed through data splitting rather than an annotation field — single-speaker speech and multi-speaker speech are two separate subsets, from which the model can distinguish single-person versus multi-person scenarios. But whether the multi-speaker subset internally performs speaker separation (diarization) and speaker labeling ([S01]/[S02]) is not stated by the paper; this is precisely the multi-speaker-scenario bottleneck that MOVA explicitly points out.
[Accent/emotion] No accent annotation is mentioned; no emotion annotation is mentioned (even though SenseVoice natively supports emotion recognition).
[Evaluation correspondence] Evaluation metrics include WER (word error rate) and SyncNet Confidence (lip-sync confidence), corresponding to verifying the generatability of transcribed content and lip-shape alignment. Table 3 shows that full-task training reduces WER from 0.044 to 0.028.
[Uncertain] (the complete attribute-set list, diarization handling, and multi-ASR arbitration rules are missing)

### Geometric and structured annotation (camera parameters, depth, 3D point tracks, action annotation, explicit state)

The paper uses no geometric or structured visual annotation of any kind: no camera parameters (intrinsics/extrinsics, trajectory), no depth maps, no 3D point tracks, no skeleton/keypoint/action annotation, no explicit physical-state annotation, no bounding boxes or segmentation masks. The only annotation approaching structure is the audio-side speaker-attribute meta field (gender, age). Positional information is handled entirely implicitly inside the model via MixD-RoPE's positional encoding (3 dimensions for video + 1 dimension for audio), rather than being introduced as data annotation. "Camera stability," although it appears among the filtering dimensions, is a scalar quality score used for screening, and does not constitute structured annotation of camera motion.

### Synthetic data construction (controlled perturbation/edits constructing training pairs, e.g., InstructAV2AV)

The paper mentions no synthetic data construction of any kind: no training pairs constructed via controlled perturbation/editing, no InstructAV2AV-style instruction-editing data pairs, no pseudo audio-video pairs from TTS dubbing synthesis, no model self-generated data feedback loop (self-distillation / rejection sampling). All 81 million training samples come from real videos with native audio tracks; the pipeline's role is filtering and annotation, not generation. The only "synthetic" component is that all captions are produced by automated models (synthetic annotation, not synthetic content). Multi-task capability (T2A / T2V / T2AV / I2V / I2AV) is not obtained by constructing synthetic data pairs of different modality forms, but rather through **random modality masking**, which dynamically constructs training targets on the same batch of real data — this is a "synthesizing tasks at training time" rather than a "synthesizing samples on the data side" approach, and is also a key design that keeps Apollo relatively economical in data-engineering effort.

### Degree of human involvement (manual annotation, manual quality inspection, model pre-screening + manual review) ⚠️

The degree of human involvement is extremely low and its disclosure is vague; the entire pipeline's only trace of human involvement is at the tail end of training rather than in the data pipeline:
[Data pipeline] Fully automated. The paper explicitly positions it as "a novel automated data-construction pipeline," with all 81M samples being "automatically annotated samples." How the filtering thresholds were determined (whether, like MOVA, humans spot-checked the retained samples under different cutoffs to calibrate them) is not stated; whether the annotation results underwent manual spot-check verification is not stated; no manual annotation, manual quality inspection, or model-pre-screening-plus-manual-review step is mentioned.
[Tail end of training] Stage III uses "the manually-curated, high-quality dataset" for final fine-tuning — this is the only mention of human involvement in the entire paper, but this dataset's scale, the manual selection criteria, the number of participants, and the annotation guidelines are all entirely undisclosed.
[Conclusion] Apollo's positioning is a two-stage approach of "automated pipeline for large scale, manual curation for tail-end refinement," with human effort concentrated on the last mile — but every detail of that last mile is a black box. [Uncertain]

## Audio-Video Alignment

### Audio-video synchronization detection method (lip sync, event alignment) ⚠️

Audio-video consistency detection is the most clearly disclosed methodological segment of Apollo's data filtering, employing a **dual-tool, dual-dimension** design (placed after audio filtering, before data splitting):
[Temporal alignment] Uses Synchformer to detect the temporal synchronization of audio and video. Synchformer is a Transformer-based sparse audio-video synchronization detection model that estimates the audio-visual time offset, and is the same tool adopted by works such as MMAudio and MOVA (and corresponds to the evaluation metrics DeSync / AV-A).
[Semantic alignment] Uses ImageBind to detect the semantic matching degree of audio and video. ImageBind embeds six modalities, including image and audio, into a unified space, measuring "whether what's on screen and what's heard are the same thing" via cross-modal cosine similarity.
[Paper's original text] "We then assess audio–visual consistency, using Synchformer for temporal alignment and ImageBind for semantic alignment, ensuring high synchronization in both temporal and semantic dimensions."
[Dedicated lip-sync check] The data-filtering stage does not mention any dedicated lip-sync detector such as SyncNet/LSE-D/LSE-C — a notable gap worth noting: one of Apollo's core selling points is solving "poor lip–speech alignment," but its data-side alignment filtering uses only the general-purpose Synchformer + ImageBind, with no dedicated filtering targeting facial lip shape observed (compare MOVA's explicit use of LSE-D ≤ 9.5 and LSE-C ≥ 4.5 to screen out a 2.5M lip-audio high-quality subset). SyncNet Confidence in Apollo appears only as an **evaluation metric** (Sync-conf improving from 5.024 to 6.787 in Table 3), not as a data filter. If accurate, this indicates Apollo's lip-sync capability is obtained primarily through its architecture (Omni-Full Attention) and multi-task training rather than through data-side lip-sync screening. [Uncertain] (whether lip sync is used for data filtering is not made explicit)

### Specific synchronization detection metrics and thresholds (SyncNet/Synchformer/LSE/in-house, threshold values, e.g., MOVA's LSE-D≤9.5 and LSE-C≥4.5, SkyReels-V4's SyncNet |offset|≤3∧conf>1.5) ⚠️

[Metrics] Clear: temporal synchronization uses Synchformer's alignment score/offset; semantic synchronization uses ImageBind cross-modal similarity.
[Thresholds] Entirely undisclosed — the paper says only "ensuring high synchronization in both temporal and semantic dimensions," giving no upper bound on Synchformer's offset (e.g., |offset| < 0.2s) or lower bound on the score, and no threshold for ImageBind similarity (e.g., IB-score > 0.2), nor does it state whether the two conditions are combined by AND or by weighted combination.
[Comparison reference] Among concurrent works, MOVA discloses a full threshold table (Audiobox PQ>5.0/CU>4.5/CE>2.5, DOVER-Aesthetic>0.85, LSE-D≤9.5, LSE-C≥4.5, etc.), and UniTalking discloses SyncNet conf>0.9; Apollo's disclosure on this dimension is notably behind.
[Evaluation-side values (not filtering thresholds, must not be conflated)] Table 3 gives the model's output synchronization metrics: DeSync 0.650 (lower is better), Sync-conf 6.787 (higher is better), IB 0.316 (higher is better).
[Uncertain]

### Separate handling of temporal synchronization vs. semantic synchronization (time alignment and content semantic matching treated as two independent filtering conditions)

Apollo explicitly separates temporal synchronization and semantic synchronization into two independent filtering conditions, each paired with a dedicated model — this is the point in its data pipeline with the clearest design intent:
- Temporal dimension: Synchformer, answering "do the sound and the picture line up on the time axis" (does the mouth shape keep pace with the speech, do the footstep sounds land on the correct frame).
- Semantic dimension: ImageBind, answering "are the sound and the picture about the same thing" (if the scene shows ocean waves but the sound is keyboard typing, there may be no conflict temporally, yet a complete semantic mismatch exists).
The two are applied in parallel, jointly constituting the determination of "high synchronization." This recognition that "temporal ≠ semantic, each must be checked separately" is precisely designed to address the two failure modes identified at the paper's outset — asynchrony (a temporal problem) and misalignment/lip–speech mismatch (a semantic and lip-shape problem).
Notably, this separation is symmetrically reused on the evaluation side: AV-A (Synchformer-family, temporal) and IB score (ImageBind, semantic) are reported as two independent metrics in parallel — the data filter and the evaluation metric share the same origin, forming a closed loop of "evaluate with what you filtered with." This both ensures consistency between the training objective and the evaluation, and also carries the potential risk of metric self-validation (a model trained on data screened by Synchformer will naturally have an advantage on Synchformer-family metrics), a point the paper does not discuss.

### Audio quality filtering (SNR, silence detection and silence-ratio threshold, exclusion of tracks with no audio, exclusion of off-screen sound sources, background-music separation) ⚠️

Audio quality filtering is a segment Apollo treats independently (the latter half of Section 4.1), with fairly complete dimension enumeration but only one publicly disclosed threshold:
[Exclusion conditions] Low SNR (signal-to-noise ratio), low MOS (mean opinion score, perceptual audio quality), abnormal clipping, distortion, noise.
[Silence control] Requires a silence ratio below 20% ("ensuring less than 20% silence") — this is the only publicly disclosed numeric threshold on the audio side, and it also reappears in the video-filtering clause ("discard those videos with low resolution, low SNR/MOS, or over 20% silence"), indicating that silence ratio is a strong constraint spanning both the audio and video gates, aimed at preventing the model from learning to produce long stretches of silent output.
[Other requirements] High fidelity, consistent formatting (referring to standardization of sample rate/channel count/encoding). The VAE-side specification is a 44.1 kHz input.
[Commonly seen steps not addressed] The paper does not mention the exclusion of tracks with no audio (although this is logically implied by the pipeline), does not mention the exclusion of off-screen voice/narration sources (i.e., cases where the sound source is not visible on screen, which is an important source of interference for lip-sync training and is specifically handled by works such as MOVA), and does not mention background-music separation (BGM separation, e.g., stripping the accompaniment using Demucs/UVR) — for Apollo, which treats singing as a first-tier subset, whether the accompaniment is separated from the vocal is a key but unanswered question.
[Specific tools and thresholds for SNR/MOS] Undisclosed (e.g., whether DNSMOS, UTMOS, Brouhaha, etc. are used). [Uncertain]

### Classification and separate processing strategy for speech/sound effects/music ⚠️

Apollo elevates "classifying by audio type and processing each type separately" to a first-class stage of the pipeline (Section 4.2 Audio-Guided Data Splitting), which is the most distinctive design in its data methodology:
[Two-level tree classification]
- First level: vocal (containing human voice) vs. non-vocal (not containing human voice) → the latter forms the sound split (natural-sound/sound-effects subset).
- Second level: the vocal subset is further split into singing / single-speaker speech / multi-speaker speech.
[Classifier] The paper does not state which model is used for the vocal/non-vocal determination or the speaker-count determination (compare MOVA's explicit use of the EAT self-supervised audio Transformer + Silero VAD). It is presumed that a combination of VAD + speaker separation (diarization) + singing detection could achieve this, but this is unconfirmed. [Uncertain]
[Differentiated processing strategy]
- Speech and singing subsets: word-by-word transcription + speaker attribute extraction (gender, age) + audio caption + video caption.
- Sound split: only audio caption, no transcription, no speaker attributes ("the sound split receives only audio captions").
That is, annotation cost is allocated according to each subset's information structure, avoiding meaningless ASR on clips without human voice.
[Position of music] The paper's four-category enumeration does not separately list "music (music/instrumental)" — whether purely instrumental content falls into the sound split or is filtered out is unstated; content with singing falls into the singing category. This differs from MOVA's approach of specifically introducing the JamendoMaxCaps music corpus during audio-tower pretraining.
[Downstream use] This splitting directly serves multi-task training: the sound split primarily feeds general T2A/foley capability, and the speech/singing split primarily feeds TTS and lip-sync capability; evaluation is correspondingly split into groups such as audio, TTS, and audio-video consistency (Figure 5).

## Training Coordination

### Multi-stage training curriculum and data curriculum scheduling (basis for stage division: resolution/duration/quality score/modality; low-res→high-res, image→video, short→long) ⚠️

Adopts a three-stage "Progressive Multi-Task Training" curriculum, but the basis for dividing stages is **task and quality**, rather than the more common resolution/duration dimension:
[Stage I — Pre-training] Trains on a large-scale, multi-scenario corpus, with the goal to "acquire atomic generation capabilities across all tasks" — acquiring atomic capabilities for all tasks, including cross-modal semantic alignment, temporal synchronization, high-fidelity audio synthesis, and precise visual feature construction.
[Stage II — Specialized Post-training] "Guided by evaluation metrics, we adaptively rebalance data distributions across scenarios and tasks to strengthen underperforming capabilities while preserving overall competency" — driven by evaluation metrics, adaptively rebalancing data mixture across scenarios and tasks, shoring up weak capabilities without sacrificing overall competency. This is a metric-feedback-driven dynamic data curriculum, closer to industrial practice than a fixed mixture.
[Stage III — Quality-Refined Post-training] Fine-tunes on a manually curated, high-quality dataset to improve generation fidelity and robustness in complex scenarios.
[Orthogonal task curriculum] Running in parallel to the three stages is the multi-task masking mechanism: random modality masking is used to jointly optimize five task types — T2A, T2V, T2AV, I2V, I2AV — within the same model; the paper calls this "from random modality masking to joint optimization across tasks."
[Notable gaps] No resolution curriculum (low-res→high-res), duration curriculum (short→long), or the common image-to-video progressive strategy is adopted or disclosed; the training steps, epoch counts, data volume, and resolution configuration for each stage are all not given. This is the largest disclosure gap relative to open-source works such as MOVA (360p→360p→720p, 61.5k→37.6k→11k hours, with per-stage elapsed days all listed). [Uncertain]

### Data mixture changes across training stages (pretraining/annealing/high-quality SFT subset) ⚠️

[Mechanism level] Stage-wise data mixture adjustment clearly exists, and is a **metric-driven adaptive rebalancing**: the core action of Stage II is precisely "adaptively rebalance data distributions across scenarios and tasks" — dynamically adjusting the mixture across two axes, scenario (single-speaker/multi-speaker/singing/natural sound) and task (T2A/T2V/T2AV/I2V/I2AV), based on evaluation metrics — whichever capability is weak gets more of that data, while ensuring existing capabilities are not damaged. This "closing the loop between evaluation feedback and data mixture" approach is the most valuable methodological statement in Apollo's Training Coordination section.
[Annealing/curation level] Stage III is effectively an annealing stage, switching the data entirely to a manually curated, high-quality subset.
[Numeric level] All entirely missing: the data volume for each stage, the mixture figures for the four audio scenarios, the sampling weights for the five tasks, the probability distribution of the masking strategy, and the adjustment magnitude and frequency of rebalancing are all undisclosed. Figure 5 ("Ablations of different training stages") graphically shows stage-by-stage improvement across four metric groups — video/audio/TTS/audio-video consistency — but does not annotate the corresponding data mixture. [Uncertain] (mechanism is clear; numbers are entirely missing)

### Post-training data (SFT curated-set scale and selection criteria, number of preference pairs and annotation method, reward-model training data) ⚠️

[Two-stage post-training] Apollo's post-training is divided into Stage II (specialized post-training, metric-driven data rebalancing) and Stage III (quality-refined post-training, fine-tuning on manually curated, high-quality data).
[SFT curated set] Stage III uses a "manually-curated, high-quality dataset" — the paper says nothing at all about its scale, the manual selection criteria, its source, or its relationship to the 81M main set (whether it is a subset or independently collected). This is one of the most regrettable gaps in this entry: a design that explicitly acknowledges "small, refined data used for tail-end curation" yet provides no benchmarkable numbers whatsoever.
[Preference data / RLHF] The paper neither uses nor mentions any preference-based optimization: no DPO, no RLHF, no reward model, no preference-pair data construction, no rejection sampling. The training objective throughout is flow matching plus multi-task masking. This differs from the route taken by some concurrent works (introducing human-preference alignment to improve aesthetics and instruction-following), and also means Apollo's alignment capability comes entirely from data quality and architecture, not from preference optimization.
[Uncertain]

### Data-processing infrastructure and throughput (NeMo Curator/Data-Juicer/in-house, GPU acceleration ratio, processing scale, cost) ⚠️

The paper discloses no information at all about data-processing infrastructure and throughput: no distributed data-processing framework such as Ray, Spark, NeMo Curator, or Data-Juicer is mentioned, no GPU/NPU count or model is given, and no acceleration ratio, processing throughput (samples/hour), total processing time, or cost estimate is given. The training side is equally blank — no training GPU count, training framework (whether Megatron/FSDP/DeepSpeed), parallelism strategy (whether context parallelism), or training duration is given. Given that the pipeline makes batch calls to large models such as Whisper-Large-v3, Qwen2.5-Omni, and Gemini 2.5-Pro against the 81-million final samples (an estimated ~300 million candidates), its annotation compute expenditure must be substantial, yet the paper gives no account of this whatsoever. As a closed-source industrial model, infrastructure details are a typically undisclosed item. [Uncertain]

## Effectiveness Comparison

### Quantified impact of data-strategy ablations (distinguishing: filtering-strictness ablation / caption-density-style ablation / data-mixture ablation, and their corresponding evaluation metrics) ⚠️

The paper's ablation experiments concentrate on **architecture and training strategy**, with **no ablation targeting data strategy whatsoever** — this is a key limitation when assessing the effectiveness of Apollo's data methodology:
[Existing ablation 1 — Architecture (Table 2)] "Comparison of different methods. The Dual Tower uses standard cross-attention, while the Single Tower utilizes our proposed Omni-Full Attention." Compares dual-tower cross-attention against single-tower Omni-Full Attention, supporting the single-tower scheme.
[Existing ablation 2 — Multi-task masking (Table 3)] "Ablation of multi-task masking," three progressive tiers:
  - T2V only: Video ID 0.71, other metrics not applicable
  - T2V + T2AV: Video ID 0.76, Audio MOS 88.181, CLAP 0.188, WER 0.044, DeSync 0.895, Sync-conf 5.024, IB 0.201
  - All tasks (Ours): Video ID 0.80, Audio MOS 93.106, CLAP 0.232, WER 0.028, DeSync 0.650, Sync-conf 6.787, IB 0.316
  Conclusion: the more complete the task set, the more consistently all metrics improve — video identity consistency +0.09, audio MOS +4.9, CLAP +0.044, WER relatively down 36%, DeSync relatively down 27%, lip-sync confidence +1.76, cross-modal alignment +0.115. The paper emphasizes that "for T2AV joint generation, our multi-task model significantly outperforms a counterpart trained solely on T2AV," i.e., multi-task joint training produces a positive gain for joint generation itself, refuting the intuition that "multi-task training dilutes single-task capability."
[Existing ablation 3 — Training stages (Figure 5)] "Ablations of different training stages. Metrics include video, audio, TTS, and audio-video consistency," showing stage-by-stage improvement of the three-stage curriculum. Since the difference between Stage II/III is essentially a difference in data mixture and data quality, this ablation **indirectly** constitutes evidence for the data strategy — it demonstrates that the data path of "metric-driven rebalancing + manual curation refinement" is effective, but since the figure does not annotate mixture numbers and does not isolate variables (stage progression is confounded with more training steps), the gain cannot be attributed to the data itself.
[Entirely missing] Filtering-strictness ablation (e.g., comparing the effect of a 27% retention rate versus 50% versus 80%, which could have been Apollo's most valuable contribution), caption-density/style ablation (dense vs. short captions), data-mixture ablation (different proportions of the four audio scenarios), and data-scale ablation (81M versus a smaller subset) — none of these exist. The paper presents "27% retention rate" as a stated fact rather than a verifiable design choice. [Uncertain]

### Evidence for quality vs. quantity (cases where small, refined data outperforms large, messy data) ⚠️

The paper provides no direct quantified evidence of "small and refined outperforming large and messy," but its overall design implies a strong quality-first orientation, which can serve as indirect evidence of its stance:
[Stance evidence 1] An aggressive end-to-end retention rate of only 27% — willing to discard nearly three-quarters of the candidate data in exchange for "high-quality, strictly aligned audio–video–caption triplets," a clear quality-for-quantity trade-off.
[Stance evidence 2] Stage III sets up a dedicated "quality-refined post-training" stage, fine-tuning at the tail end on a small, manually curated, high-quality set — a typical pyramid structure of "large scale learns capability, small scale improves perceived quality."
[Stance evidence 3] Even at the 81M scale, the paper still spares no expense to invoke commercial flagship models such as Gemini 2.5-Pro for annotation, a signal that annotation quality is prioritized over cost.
[Key gap] The above three points are all design choices rather than experimental evidence — the paper does not run a controlled experiment of "under equal compute, using the 27% curated data versus using the full 100% of the data," so it cannot prove that the 27% cutoff is optimal, nor can it quantify the benefit brought by filtering. By comparison, MOVA at least gives contrastive figures for its 360p→720p curriculum. What Apollo provides on this dimension is a **posture** rather than **evidence**. [Uncertain]

### Alignment between training-data domain distribution and the evaluation-benchmark taxonomy (e.g., VABench's seven major categories) ⚠️

[Benchmark used] Follows the third-party benchmark Verse-Bench to evaluate the T2AV task (paper's original text: "Following Universe-1, we use Verse-Bench for T2AV tasks"), with no self-built evaluation benchmark. Verse-Bench is the audio-video joint generation evaluation benchmark proposed by Universe-1.
[Evaluation metric system] Video quality uses Motion Score (RAFT optical flow) and Aesthetic Score (MANIQA, aesthetic-predictor-v2-5, Musiq); audio quality uses FD (PANNs mel-spectrogram Fréchet distance), KL divergence, and MOS; synchronization uses AV-A / DeSync (Synchformer) and SyncNet Confidence; cross-modal semantics uses CLAP and ImageBind score; speech content uses WER. Table 1, "Main comparisons of audio-visual joint generation," compares cascaded schemes against joint-generation schemes across 10 metrics, with the paper claiming a level comparable to Veo 3.
[Alignment relationship with training-data distribution] A clear correspondence exists but is not pointed out by the paper: between Apollo's four-way training-data split (single-speaker speech / multi-speaker speech / singing / natural sound) and its four evaluated capability groups (video / audio / TTS / audio-video consistency, see Figure 5) there is a mapping chain of "data category → capability category → metric group" — the sound split corresponds to the audio group of metrics (FD/KL/CLAP), the speech and singing splits correspond to the TTS group of metrics (WER/Sync-conf), and all correspond to the consistency group (DeSync/IB). Stage II's metric-driven rebalancing mechanism is built precisely on top of this mapping chain: when a metric group is weak, add the corresponding category of data. This is actually one of the relatively rare cases in this research where "the data-taxonomy system and the evaluation-taxonomy system form an explicit closed loop."
[Gaps] The paper gives no enumeration of Verse-Bench's categories (contrast with VABench's seven major categories), nor does it report scores per category, so it is impossible to verify whether the training-data distribution matches the benchmark taxonomy in coverage, or whether there are blind spots in the taxonomy. [Uncertain]

## Uncertain Fields

The research information for the following fields is partially uncertain (sources marked with ⚠️):

- organization
- data_scale
- data_sources
- provenance_licensing
- duration_distribution
- resolution_aspect_distribution
- domain_distribution
- audio_category_distribution
- narrative_structure_distribution
- language_accent_distribution
- shot_segmentation
- quality_filtering
- motion_filtering
- deduplication
- model_as_data_judge
- safety_filtering
- caption_structure
- joint_av_caption_schema
- dialogue_transcription_attributes
- human_in_loop
- av_sync_detection
- sync_metric_and_threshold
- audio_quality_filtering
- audio_type_handling
- multi_stage_curriculum
- stage_data_mixture
- post_training_data
- data_infra_throughput
- data_ablation
- quality_over_quantity_evidence
- benchmark_taxonomy_alignment
