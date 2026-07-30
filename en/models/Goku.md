# Goku (*Goku: Flow Based Video Generative Foundation Models*, including the Goku-T2I / Goku-T2V / Goku-I2V series; arXiv:2502.04896, CVPR 2025 Highlight)

> Topic: Data processing for video generation models (including joint audio-video generation): data cleaning pipeline, data distribution, annotation methods

[← Back to home](../index.md)

**Table of Contents**: [Basic Information](#basic-information) · [Data Scale and Distribution](#data-scale-and-distribution) · [Cleaning Pipeline](#cleaning-pipeline) · [Annotation Methods](#annotation-methods) · [Audio-Video Alignment](#audio-video-alignment) · [Training Coordination](#training-coordination) · [Results Comparison](#results-comparison)

## Basic Information

### Name

Goku (*Goku: Flow Based Video Generative Foundation Models*, including the Goku-T2I / Goku-T2V / Goku-I2V series; arXiv:2502.04896, CVPR 2025 Highlight)

### Publishing Organization/Company

A joint effort between ByteDance and the University of Hong Kong (HKU). The paper credits 22 authors; the first author is Shoufa Chen (HKU), with corresponding/senior authors including Ping Luo (HKU), Yi Jiang, Zehuan Yuan, Bingyue Peng, and Xiaobing Liu (ByteDance). The team overlaps heavily with ByteDance's Seed / visual generation line; the project's codename derives from Goku of *Dragon Ball* (the organization name Saiyan-World).

### Release Date (technical report/paper/open-source date)

First submitted to arXiv (v1) on February 7, 2025; v2 updated on February 10, 2025. Later accepted by CVPR 2025 as a Highlight. The project homepage saiyan-world.github.io/goku and the GitHub repository Saiyan-World/goku went live around the same time.

### Type (model/dataset/toolchain/benchmark)

Model (a family of joint image-and-video generation foundation models based on a rectified-flow Transformer/DiT). The paper also discloses a five-stage data processing pipeline in detail (which can be regarded as a toolchain methodology description, though the pipeline code and the data itself are not open-sourced). Not a dataset, not a benchmark.

### Degree of Openness (whether weights/code/data/pipeline are each open-sourced)

Overall this falls into the "paper public, weights and data closed" category.
[Public] (1) The technical paper fully discloses the thresholds at each stage of the data pipeline, the model architecture (Goku-2B: 28 layers / dim 1792 / 28 heads; Goku-8B: 40 layers / dim 3072 / 48 heads), the rectified-flow training recipe, and the parallel training infrastructure; (2) the GitHub repository Saiyan-World/goku provides a code directory skeleton (configs/, goku/, tools/, etc.) and a requirements.txt; (3) the project homepage provides extensive visualizations of generated samples.
[Not open-sourced] Model weights (no official Goku weights are released on HuggingFace), training data, data processing pipeline code, the internal video classification model, and the internal aesthetic scoring model. No LICENSE file is present in the repository either.
[Conclusion] Weights: no; code: partial/incomplete; data: no; pipeline: only described in the paper's text, code not open-sourced.
Note: many third-party websites/products in the community named "Goku AI" are riding on the name and are not official releases.

### Whether Joint Audio-Video Generation Is Supported, and the Implementation Approach (native joint / cascaded / MoE fusion)

Not supported. Goku is a purely visual joint image + video generation model (T2I / T2V / I2V); the paper never touches on audio generation, audio-track modeling, speech, or sound effects anywhere in its text. Its "joint generation" refers to jointly modeling the two visual modalities of image and video within the same rectified-flow Transformer (an image is treated as a single-frame video; after encoding through a joint 3D image-video VAE it interacts uniformly with video tokens under full attention), not an audio-video joint. This entry is therefore not applicable along the audio-video dimension; its value lies mainly in its data-distribution-balancing methodology on the pure-video side.

### List of Research Information Sources (URLs of papers/technical reports/official documentation/news, with the nature of each source noted: first-party official / same-team corroboration / third-party report)

1. [First-party official] https://arxiv.org/abs/2502.04896 — the arXiv abstract page for the Goku paper (v1 2025-02-07, v2 2025-02-10).
2. [First-party official] https://arxiv.org/html/2502.04896v2 — full HTML text of the paper; Section 4, Data Curation Pipeline, is the main basis for this research, including Table 4 (per-resolution filtering thresholds) and Figure 3 (semantic category distribution charts).
3. [First-party official] https://github.com/Saiyan-World/goku — the official GitHub repository and README (full VBench comparison table, BibTeX, author and institutional credits: HKU, ByteDance).
4. [First-party official] https://saiyan-world.github.io/goku/ — the project homepage, with visualizations of generated samples.
5. [Third-party aggregator] https://huggingface.co/papers/2502.04896 — the HuggingFace Papers page, community discussion and voting.
6. [Third-party report] https://www.etcentric.org/bytedances-goku-video-model-is-latest-in-chinese-ai-streak/, https://stable-learn.com/en/goku-video-model-introduction/, https://www.analyticsvidhya.com/blog/2025/02/goku-ai/ — media coverage, used to cross-check model scale and positioning; not authoritative for data details.

## Data Scale and Distribution

### Training Data Volume (number of video clips/hours/tokens, pretraining vs. SFT separated)

[Total volume] The final training set comprises approximately 160 million image-text pairs (160M) + 36 million video-text pairs (36M).
[Text-to-Image T2I] 100 million public samples (from LAION) + 60 million internal high-quality samples, totaling 160M. Clear division of labor: public data is used for pre-training, internal data for fine-tuning.
[Text-to-Video T2V] 11 million public clips + 25 million in-house clips, totaling 36M clips.
[Layered by resolution] 36M clips are usable for 480p training; of these, 24M meet the 720p threshold; 7M meet the 1080p threshold. That is, the data volume shrinks progressively as resolution increases (36M → 24M → 7M), forming a natural resolution-curriculum data pyramid.
[Image-to-Video I2V post-training] Approximately 4.5 million (4.5M) "text-image-video" triplets.
[Not disclosed] Total video hours, token counts, raw collection volume (so the overall retention rate cannot be back-calculated).

### Composition of Data Sources (in-house/public datasets/web scraping/licensed acquisition/synthetic data)

A mix of four categories:
(1) Public datasets — on the image side, LAION (Schuhmann et al., 2022, 100 million samples); on the video side, explicitly listed as Panda-70M, InternVid, OpenVid-1M, and Pexels, totaling 11 million clips.
(2) In-house/internal data — 60 million "high-quality internal samples" on the image side, and 25 million in-house clips on the video side (about 69% of the video data, the primary source). The paper does not specify the exact channels behind the internal data (whether it comes from the Douyin/TikTok ecosystem, or is licensed acquisition).
(3) Web scraping — not explicitly stated, but the public datasets themselves (LAION, Panda-70M, InternVid) are all web-scraped sources.
(4) Synthetic data — the paper does not use synthetic video data; GPT-4o is used only in the evaluation stage to rewrite GenEval's short prompts (an evaluation-side operation, not training-data synthesis).
Overall this presents the classic ByteDance-style recipe of "public data as the base + internal high-quality data for fine polishing."

### Data Compliance and Provenance (proportion of licensed data, rights-cleared datasets, C2PA, etc.) ⚠️

[Uncertain]. The paper does not discuss data copyright, licensing proportions, rights-cleared datasets, C2PA / content-provenance labeling, watermark traceability, or any other compliance topics at all, nor does it mention a datasheet or terms of use. The only confirmable facts are: the public portion relies on existing academic/royalty-free datasets such as LAION, Panda-70M, InternVid, OpenVid-1M, and Pexels (Pexels is a royalty-free stock media site; LAION is a CC-scraped image-text pair index that is itself known to be subject to copyright disputes); the licensing status and provenance compliance of the internal 25 million video clips and 60 million images are not disclosed. This is fairly typical of Chinese-vendor papers from early 2025, in contrast to Movie Gen / Veo, which emphasize licensing and C2PA.

### Clip Duration Distribution and Segmentation Strategy

[Segmentation strategy] Raw long videos are first cut into semantically coherent short clips, with two core constraints:
(1) Duration lower bound — clips shorter than 4 seconds are discarded outright during preprocessing (duration ≥ 4s).
(2) Duration upper bound — a single clip is truncated to at most 10 seconds (maximum clip length 10 seconds); overly long continuous shots are forcibly cut.
Training clip durations therefore fall strictly within the [4s, 10s] range, which is also why Goku's generation capability is concentrated on short videos under 10 seconds.
[Other preprocessing constraints] Bitrate ≥ 500 kbps; frame rate ≥ 24 FPS (Film Standard) or 23.976 FPS (NTSC Standard); material with a lower frame rate is removed.
[Not disclosed] The specific duration histogram within the range, the average clip duration, and the sampling weights for each duration bucket.

### Resolution/Aspect-Ratio Distribution and Bucketing Strategy ⚠️

[Ingestion threshold] min{height, width} ≥ 480, i.e., the short side must be no less than 480 pixels.
[Resolution bucketing] Data is bucketed into three tiers — 480p / 720p / 1080p — each with its own independently tightened set of filtering thresholds (paper Table 4); higher-resolution tiers have stricter thresholds → the data volume shrinks tier by tier: 36M at the 480p tier, 24M at the 720p tier, 7M at the 1080p tier. This bucketing design of "the higher the resolution, the higher the quality bar, the less the data" directly serves a progressive resolution training curriculum.
[Training resolution sequence] 288×512 → 480×864 → 720×1280, all at a 16:9 landscape ratio; the I2V/high-resolution stage involves 1080p data.
[Aspect ratio] The paper organizes training around fixed 16:9 resolutions (288×512, 480×864, 720×1280); it does not describe a multi-aspect-ratio bucketing strategy, nor does it give the proportion of portrait/square data. [Uncertain] (the aspect-ratio distribution and whether arbitrary-ratio training is supported).

### Category/Domain Distribution and Mixture Strategy (proportional control and concept balancing across people, actions, scenes, styles, etc.)

★ This is Goku's most distinctive feature, and the highest-value part of this research: Data Distribution Balancing is explicitly listed as the fifth, independent stage of the five-stage pipeline, rather than being subordinate to the filtering step. Most contemporaneous work (HunyuanVideo, Open-Sora, CogVideoX, etc.) treats category mixture merely as a byproduct of filtering, or does not mention it at all; Goku elevates it to a first-class stage on par with "collection / segmentation / filtering / captioning," making it one of the most explicit pieces of modeling of the "data distribution" dimension.
[Taxonomy] An internal, in-house-developed video classification model is used to uniformly sample 4 keyframes per clip and produce labels across a two-level taxonomy: 9 top-level categories + 86 second-level subcategories.
  - Examples of top-level categories: human, scenery, animals, food, urban life, etc., of which human, scenery, food, urban life, and animals are the highest-proportion categories.
  - Examples of second-level subcategories: half-selfie, kid, dinner, wedding, etc. — fairly fine-grained.
[Mixture strategy] The paper explicitly states that it "emphasizes human-related content while ensuring equitable representation across subcategories" — that is, it deliberately overweights human-related content overall (since humans are the highest-frequency, most closely scrutinized demand in video generation, and also the aspect most prone to visible flaws), while ensuring relatively balanced representation across the 86 subcategories. Two specific mechanisms are used:
  (1) down-sampling overrepresented categories;
  (2) augmentation and oversampling for underrepresented categories.
[Visualization] Paper Figure 3 shows the balanced semantic distribution via two charts: (a) top-level categories, (b) second-level subcategories.
[Assessment] This stage embodies the idea of "concept balancing": filtering only guarantees the quality of individual samples, whereas distribution balancing guarantees the coverage and non-skewness of the entire dataset in concept space, directly corresponding to Goku's performance on VBench's multi-category metrics such as human action, object class, and scene (Goku scores 79.48 on human action and 85.72 on scene, both markedly higher than contemporaneous open-source models).
[Not disclosed] The specific percentage figures for each category (only a chart is given, no table), the specific down-sampling/oversampling ratios, the architecture and accuracy of the classification model, and whether a VLM is used instead of a dedicated classifier.

### Audio Category Distribution and Mixture (proportions and control strategy for speech/sound-effect foley/music/ambient sound/silence) — a dimension unique to AV models

Not applicable. Goku is a purely visual image + video generation model; its training data consists of "image-text pairs" and "video-text pairs" and contains no audio track. The paper does not touch on any classification, proportion, or control strategy for speech / sound-effect foley / music / ambient sound / silence anywhere in its text, and there is no audio channel in the data pipeline. Whether the audio track of the original video, if present, is kept or discarded is not stated in the paper (judging from the video-text pair data format, the audio track is presumably discarded outright).

### Narrative Structure Distribution (single-shot vs. multi-shot, average clip duration, shot-count distribution, whether a native audio track is included)

Single-shot clips form the overwhelming majority. The segmentation stage's design goal is precisely to "eliminate shot changes": PySceneDetect first performs shot-boundary detection, followed by a second verification pass using DINOv2 feature cosine similarity (the 480×864 tier requires adjacent-frame DINO similarity ≥ 0.85, the 720×1280 tier requires ≥ 0.90); the two-tier mechanism ensures each training clip is visually continuous internally and contains no transitions. The shot-count distribution can therefore be regarded as "almost entirely 1 shot," with no modeling of multi-shot narrative structure.
Average clip duration falls within the 4-10 second range (the exact mean is not disclosed).
Whether it includes a native audio track: no (the data format is pure video-text pairs).
The paper does not address structured narrative data design such as multi-shot narrative, storyboards, or long-video continuity.

### Language/Accent Distribution (data basis for multilingual lip-sync capability) ⚠️

Not applicable/not addressed. Goku does not generate speech or lip shapes, has no multilingual lip-sync capability, and its data pipeline has no ASR, no language identification, and no accent labeling. The only language-related aspect is the caption text language: the paper uses InternVL2.0, Tarsier2, and Qwen2 to generate and merge English captions, and the evaluation benchmarks (GenEval, DPG-Bench, VBench) are also English-prompt-based systems, from which it can be inferred that training captions are predominantly English. [Uncertain] (whether captions include Chinese or other languages, and the proportion of each language, are not stated in the paper).

## Cleaning Pipeline

### Overall Structure of the Cleaning Funnel (number of filtering stages, ordering)

★ The five-stage pipeline is this entry's most central methodological contribution, in the following order:
[Stage 1 — Image and Video Collection]
  Raw material is gathered from public datasets (LAION, Panda-70M, InternVid, OpenVid-1M, Pexels) and internal repositories.
[Stage 2 — Video Extraction and Clipping]
  PySceneDetect detects shot boundaries → 1 frame per second is sampled to extract DINOv2 features → adjacent-frame cosine similarity verifies shot consistency (480p ≥0.85 / 720p ≥0.90) → a single clip is truncated to at most 10 seconds → clips from the same source are deduplicated with perceptual hashing.
[Stage 3 — Image and Video Filtering]
  Multi-dimensional parallel filtering: basic attributes (duration ≥4s, short side ≥480, bitrate ≥500kbps, frame rate ≥24/23.976 FPS) + aesthetic score + OCR text coverage + RAFT optical-flow motion score, with thresholds along every dimension set separately for the 480p/720p/1080p resolution tiers, stricter at higher resolutions.
[Stage 4 — Captioning]
  InternVL2.0 generates keyframe captions + Tarsier2 generates whole-video captions (including camera motion) → Qwen2 merges the two caption streams → the RAFT motion score is appended as a controllable condition.
[Stage 5 — Data Distribution Balancing]
  The internal video classification model labels 4 keyframes → 9 top-level categories / 86 subcategories → overrepresented categories are down-sampled, underrepresented categories are augmented and oversampled → human-related content is overweighted overall.
[Structural characteristics] The first four stages are a bottom-up "per-sample purification" process (funnel-style serial filtering), while the fifth stage is a top-down "dataset-level distribution shaping" process — the two operate at different granularities. Making distribution balancing an independent stage is the key design that sets Goku apart from contemporaneous work.

### Quantitative Funnel Retention Rate (input/output volume and final retention rate at each filtering stage, e.g., Apollo's 27%) ⚠️

[Uncertain]. The paper gives the decision thresholds for each filtering stage (Table 4) and the final data volumes (160M images / 36M videos), but **does not disclose the input volume, output volume, or retention rate for any individual stage**, nor does it give the size of the raw collection, so the overall funnel retention rate cannot be computed (in contrast to Apollo, which discloses a 27% retention rate, Goku is less transparent on this point).
The only quantitative shrinkage clue that can be back-derived is the cascade of data volumes across resolution tiers: 36M at the 480p tier → 24M at the 720p tier (about 66.7% retained relative to 480p) → 7M at the 1080p tier (about 19.4% retained relative to 480p, about 29.2% relative to 720p). But this shrinkage is the compound result of "resolution + stricter thresholds" and is not equivalent to a single-stage filtering retention rate.
In addition, the I2V post-training uses 4.5M triplets, about 12.5% of the 36M video pool, which can serve as an indirect reference point for the proportion of the high-quality subset.

### Shot Segmentation Method (PySceneDetect / in-house model / shot-aware splitting)

A two-tier shot segmentation, a relatively refined approach for its time:
(1) Tier one — PySceneDetect performs initial shot boundary detection, quickly locating transitions based on pixel/histogram differences.
(2) Tier two — DINOv2 semantic feature verification: the extracted clip is sampled at 1 frame per second, DINOv2 visual features are extracted, and the cosine similarity between adjacent sampled frames is computed; the entire clip must maintain high similarity to be judged "the same shot and visually coherent." The threshold tightens with resolution: the 480×864 tier requires ≥0.85, the 720×1280 tier requires ≥0.90. This step compensates for PySceneDetect's missed detections (such as gradual transitions or large content changes within the same scene), constituting a "shot-aware splitting + semantic consistency double safeguard."
(3) Length constraint — a single clip is at most 10 seconds; overly long shots are forcibly truncated.
No in-house end-to-end segmentation model is used, nor is any alternative such as TransNetV2 mentioned.

### Quality Filtering (aesthetic score, sharpness, OCR text filtering, letterbox/watermark/logo detection)

Multi-dimensional threshold filtering; the most notable feature is that **all thresholds are set in tiers according to the target resolution, tightened across the board at higher resolutions** (paper Table 4):
[Basic technical attributes (hard gates)]
  - Duration ≥ 4 seconds;
  - Resolution min{height, width} ≥ 480;
  - Bitrate ≥ 500 kbps (removes heavily compressed, low-bitrate material with severe blocking artifacts);
  - Frame rate ≥ 24 FPS (Film Standard) or 23.976 FPS (NTSC Standard); material with a lower frame rate/frame-dropped material is removed.
[Aesthetic Score] An aesthetic model scores keyframes:
  - 480p tier: score < 4.3 is discarded;
  - 720p and above: score < 4.5 is discarded.
  (The paper does not specify the exact aesthetic model version; presumably from the LAION-Aesthetic family or an internal variant thereof.)
[OCR text coverage filtering] Detects the proportion of frame area occupied by text regions in keyframes, discarding clips above the threshold (mainly removing subtitles, overlay ads, stylized text, and screen-recorded content):
  - 480p tier: text area ≤ 0.02 (2%);
  - 720p and above: text area ≤ 0.01 (1%).
  This threshold is extremely strict, indicating the team places high priority on preventing the model from learning to generate garbled text.
[Deduplication (tied to quality)] Clips from the same source are compared via perceptual hashing; when hashes are close, **the clip with the higher aesthetic score is kept** — the deduplication decision is directly arbitrated by the quality score.
[Common items not mentioned] The paper does not describe letterbox/black-bar detection, watermark/logo detection, blur/sharpness (e.g., Laplacian variance, NIQE), abnormal brightness/saturation, or temporal flicker consistency as filtering dimensions, nor does it mention any independent filtering criteria for the image side (T2I data) beyond the aesthetic score. Compared with work from late 2025 through 2026, Goku's quality-filtering dimensions lean classical and shallow.

### Motion Filtering (optical-flow/motion-score thresholds, removing static and jittery content)

The RAFT optical-flow model is used to compute a motion score, with a **two-sided threshold** set (removing both near-static footage and footage with excessively violent motion/jitter); the thresholds are likewise divided into three resolution tiers, tightening at each tier:
  - 480p: 0.3 ≤ motion score ≤ 20.0
  - 720p: 0.5 ≤ motion score ≤ 15.0
  - 1080p: 0.5 ≤ motion score ≤ 8.0
Design logic: the lower bound removes slideshow-like static videos and dead frames (ensuring the model learns dynamic degree, corresponding to VBench's dynamic degree, on which Goku scores 76.11); the upper bound removes handheld jitter, fast pans, residual violent transitions, and other material that would cause training instability and motion blur. The higher the resolution, the lower the upper bound is pushed (20 → 15 → 8), because high-definition material is more geared toward fine-detail quality learning rather than large-motion learning.
Moreover, the motion score is not discarded after filtering — it is **reused as part of the caption** (see the captioning fields), becoming a controllable motion-intensity condition at inference time. This is one of Goku's clever design choices: the same signal serves both as a filtering gate and as a conditioning control.

### Deduplication Method (exact deduplication and embedding-based semantic deduplication recorded separately) ⚠️

Only one mechanism is described: **approximate deduplication based on perceptual hashing**. The method computes a perceptual hash for each clip's keyframe and compares them pairwise; if two clips' hashes are close, they are judged duplicates, and **the one with the higher aesthetic score is kept** while the other is discarded. The paper explicitly limits this deduplication to occurring "between clips cut from the same source video," mainly addressing the problems of highly overlapping adjacent clips after segmenting long videos, and of repeated/replayed shots.
[Not addressed] Cross-source/global exact deduplication (e.g., file-level MD5, whole-corpus frame-by-frame hash comparison), embedding-based semantic deduplication (e.g., CLIP/DINOv2 feature-cluster deduplication), and deduplication processing on the image side (the 100-million-sample LAION set) are all left unexplained in the paper. This is a relatively weak link in Goku's data pipeline — DINOv2 features have already been extracted at the segmentation stage, yet they are not used for semantic deduplication. [Uncertain] (whether undisclosed global semantic deduplication exists).

### VLM/LLM as Quality Inspector (multimodal large-model quality scoring and mismatch removal; the 2026 trend of shifting from shallow scorers to large-model semantic judgment)

Overall this still belongs to the "dedicated small-model scorer" paradigm, and has not yet reached the mainstream 2026 stage of "large-model semantic judge," but several early forms of model-based quality inspection have already emerged:
[Model-based judgments already in use]
  (1) DINOv2 self-supervised features — used for shot semantic-consistency judgment via cosine similarity, a feature-level rather than language-level judgment;
  (2) Aesthetic scoring model — a shallow regression scorer, thresholds 4.3/4.5;
  (3) RAFT optical-flow model — motion-quality judgment;
  (4) OCR model — text-coverage judgment;
  (5) Internal video classification model — semantic labeling across 9 top-level categories / 86 subcategories, serving the distribution-balancing stage; this is the step closest to a "semantic-level model judge," but its role is **classification and mixture control**, not **quality adjudication**;
  (6) Multimodal large models (InternVL2.0, Tarsier2) and an LLM (Qwen2) — but these are used only for **caption generation and merging**; the paper does **not** use them for quality scoring, text-video misalignment detection, or removal of semantically implausible samples.
[Clearly absent] The paper has no VLM scoring step, no verification of caption-video consistency (e.g., CLIP score / VLM consistency judgment), and no LLM review of caption quality.
[Assessment] Goku (early 2025) represents the "shallow scorer + model-based labeling" stage; compared with later work such as Seedance and LTX-2, which introduce a VLM as a quality inspector for semantic-level screening and mismatch removal, Goku sits clearly on the earlier side of this era's dividing line along this dimension, and can serve as a reference baseline for tracking this trend's evolution.

### Safety and Compliance Filtering (NSFW, copyright, face/privacy) ⚠️

[Uncertain]. The paper does not mention NSFW/pornography-violence filtering, copyright filtering, face and privacy protection, sensitive-person identification, harmful-content classifiers, or any other safety/compliance step anywhere in its text, nor does it have a Responsible AI / model-card section. Given that the publisher is ByteDance, internal safety review almost certainly exists in the actual production pipeline (ByteDance has a mature content-safety platform), but there is zero disclosure at the paper level. The only indirectly related item is OCR text filtering (removing subtitles/overlay ads), but its motivation is image quality and generation quality, not safety compliance.

## Annotation Methods

### Caption Model Used (in-house VLM / open-source model, model scale) ⚠️

★ A three-model collaborative captioning scheme of "dual complementary VLMs + LLM fusion" is adopted — a fairly advanced combination for its time:
(1) **InternVL2.0** — handles keyframe-level image captioning, strong at fine-grained content, objects, attributes, and scene description in static frames.
(2) **Tarsier2** — handles video-wide-level captioning. The paper specifically notes an inherent advantage of Tarsier2: it naturally describes **camera motion types** (e.g., zoom in, pan right), so no additional camera-motion annotation module is needed to obtain shot-language labels.
(3) **Qwen2** — acts as the LLM merger, combining the keyframe caption and the video caption into a single unified, non-redundant, non-contradictory final description. The paper's original text: "we utilize Qwen2 to merge the keyframe and video captions."
[Design motivation] A single image VLM lacks temporal and motion information, while a single video VLM has insufficient granularity on static details; the two complementary streams are disambiguated and fused by the LLM, achieving both spatial detail and temporal/shot semantics.
[Not disclosed] The specific parameter scale and version number of each model (InternVL2.0 has multiple tiers from 1B to 76B, Tarsier2 has a 7B tier), whether domain-specific fine-tuning was performed, the compute cost and throughput of captioning, and whether the image data (the 160M for T2I) uses the same captioning scheme. [Partially uncertain]

### Caption Density and Degree of Structuring (short/long/dense description, structured fields such as camera motion, style tags) ⚠️

This is a hybrid form of **dense long-form description + a small amount of appended structured tagging**:
[Main body] Dense natural-language description, generated by two streams — InternVL2.0 (keyframe details: objects, attributes, scene, composition) and Tarsier2 (temporal dynamics, action progression, camera motion such as zoom in / pan right) — then fused by Qwen2 into a single unified long-form text. A single caption therefore covers three levels at once: "frame content + temporal action + shot language."
[Structured field appended] The most distinctive point: **the motion score value computed by RAFT is appended directly to the end of the caption**, turning motion intensity into an explicit conditioning variable that can be manually specified at inference time. This lets users control the dynamic magnitude of the generated video by specifying a motion score in the prompt, converting a statistic originally used only for filtering into a controllable generation interface — a textbook example of "reusing a filtering signal as a conditioning signal."
[Not adopted] No layered multi-granularity captions (short/medium/long versions) are seen, no independent structured field system for style tags/lighting/image quality is seen, and no description of caption dropout or a mixed short/long caption training strategy is seen. [Uncertain] (the average caption length, and whether a short-caption version is retained).

### Joint Audio-Video Caption Structure (whether both visual + auditory tracks are covered simultaneously, whether they are split into separate fields, e.g. LTX-2's full soundscape description, Script-a-Video's factorized streams, Foley-Omni's three fields)

Not applicable. Goku has no audio modality; its caption covers only the visual track (frame content + action + camera motion + motion score), and there is no auditory-track description, no audio-video split field, and no full-soundscape description. It can serve as a baseline for the "pure-visual-era caption schema," forming a generational contrast with joint audio-video schemas such as LTX-2's full soundscape description, Script-a-Video's factorized streams, and Foley-Omni's three-field structure: Goku's "keyframe + whole-clip" dual-stream fusion approach is structurally kindred to the later audio-video models' "visual stream + auditory stream" factorize-then-fuse approach, just with the dimension swapped for temporal granularity rather than modality.

### Dialogue Transcription and Speaker Attribute Annotation (ASR transcription + speaker identity/language/accent/emotion)

Not applicable/not addressed. There is no ASR transcription step in the data pipeline, and no annotation of speaker identity, language, accent, emotion, or other attributes. On the contrary, Goku **actively removes** on-screen text (including subtitles) — a clip is discarded once OCR text area exceeds 1%-2% — indicating its data orientation deliberately avoids dialogue and subtitle scenes.

### Geometric and Structured Annotation (camera parameters, depth, 3D point tracks, action labeling, explicit state)

Structured annotation is very lightweight, comprising only two items:
(1) **Camera motion** — not obtained via geometric estimation, but implicitly described in natural-language form by Tarsier2 when generating the video caption (zoom in, pan right, etc.), i.e., a "languagized shot label" rather than a parametric camera trajectory.
(2) **Motion intensity** — the scalar motion score obtained from RAFT optical flow, serving both as a filtering threshold and as a numeric condition in the caption.
[Not addressed] Camera intrinsic/extrinsic calibration, depth maps, 3D point tracks, optical-flow fields used as conditioning input, human pose/skeleton, object bounding boxes and trajectories, segmentation masks, explicit state annotation, etc. Goku's conditioning-control dimensions consist only of "text + (in I2V scenarios) the first-frame image + motion score," a degree of structuring markedly lower than work around 2026 that emphasizes geometric controllability.

### Synthetic Data Construction (controlled perturbation/edited construction of training pairs, e.g. InstructAV2AV) ⚠️

No synthetic data construction is used in the training data. The paper contains no synthetic-data pipeline of any kind — no controlled perturbation, no edit-constructed training pairs, no rendering-engine synthesis, no data augmentation generating new samples.
The only two places that touch on "synthesis/rewriting" are both non-training-data:
(1) The "augmentation and oversampling" applied to underrepresented categories in the distribution-balancing stage — here "augmentation" presumably refers to traditional data augmentation/resampling; the paper does not state whether it includes generative expansion;
(2) In the evaluation stage, GPT-4o is used to expand GenEval's short prompts ("we expand the original short prompts in GenEval with ChatGPT-4o, preserving their semantics while enhancing descriptive detail"), which is a prompt-rewriting evaluation technique used to bridge the distribution gap between the dense captions used during training and the short prompts used during evaluation — this itself is important evidence that "the training caption style shapes the form of inference prompts." [Partially uncertain] (the specific meaning of "augmentation" in the balancing stage).

### Degree of Human Involvement (manual annotation, manual quality inspection, model pre-screening + manual review) ⚠️

The paper **does not mention any manual annotation, manual quality inspection, or manual review step**; the entire five-stage pipeline is presented as a fully automated design driven purely by models and thresholds (PySceneDetect + DINOv2 + aesthetic model + OCR + RAFT + classification model + VLM/LLM captioning).
Points of human involvement that likely exist but are not disclosed include: the design and training annotation of the internal video classification model's 9-category/86-subcategory taxonomy, the criteria for judging "high quality" among the 60 million internal high-quality images, and the process for determining the various thresholds (4.3/4.5, 0.85/0.90, 0.02/0.01, the motion-score bounds) — the selection of these thresholds itself almost certainly involved human trial-and-error and sample-based visual inspection, but the paper does not elaborate. In addition, the paper's subjective quality comparisons (Figure 5, etc.) involve human judgment, but this belongs to evaluation rather than the data pipeline. [Uncertain] (the actual extent of human involvement).

## Audio-Video Alignment

### Audio-Video Synchronization Detection Method (lip sync, event alignment)

Not applicable. Goku has no audio modality, and there is no audio-video synchronization detection step in the data pipeline (no lip-sync detection, no audio-visual event alignment, no SyncNet/AV-align-type methods). The only concept related to "alignment" in its data pipeline is **visual temporal consistency**: DINOv2 adjacent-frame cosine similarity (480p ≥0.85, 720p ≥0.90) is used to ensure visual coherence within a clip without jumps — this is a temporal-consistency constraint purely within the visual domain, methodologically distinct from cross-modal audio-video synchronization.

### Specific Synchronization Detection Metrics and Thresholds (SyncNet/Synchformer/LSE/in-house, threshold values, e.g. MOVA LSE-D≤9.5 and LSE-C≥4.5, SkyReels-V4 SyncNet |offset|≤3∧conf>1.5)

Not applicable. There are no audio-video synchronization metrics or thresholds. What can be recorded for comparison is its visual-side consistency threshold system (as a reference for its "threshold-setting style"): DINOv2 inter-frame cosine similarity ≥0.85 (480×864) / ≥0.90 (720×1280); aesthetic score ≥4.3 (480p) / ≥4.5 (720p+); OCR text area ≤0.02 (480p) / ≤0.01 (720p+); RAFT motion score [0.3, 20.0] (480p) / [0.5, 15.0] (720p) / [0.5, 8.0] (1080p). All of these thresholds are tiered by resolution and tighten as sharpness increases — a unified philosophy underlying Goku's threshold design.

### Separate Handling of Temporal Synchronization vs. Semantic Synchronization (time alignment and content-semantic matching treated as two independent filtering conditions)

Not applicable (no audio-video modality). If one forces an analogy to a "temporal vs. semantic" separation on Goku's visual side, an approximate structure can be observed: the shot-segmentation stage chains together two mutually independent conditions as gatekeepers — **PySceneDetect (temporal-abruptness detection at the pixel/histogram level)** and **DINOv2 feature cosine similarity (content-consistency judgment at the semantic level)** — the former governs "whether there is a hard cut," the latter governs "whether the content is still the same thing." This is structurally isomorphic to the approach audio-video models take in splitting "temporal alignment" and "content-semantic matching" into two independent filtering conditions, and can serve as an indirect methodological reference.

### Audio Quality Filtering (SNR, silence detection and silence-ratio thresholds, no-audio-track removal, off-screen-voice-source removal, background music separation)

Not applicable. There is no audio-quality filtering step of any kind in the data pipeline: no SNR judgment, no silence detection or silence-ratio threshold, no removal of clips lacking an audio track, no removal of off-screen-voice/narration sources, no background-music separation (e.g., Demucs/BS-RoFormer). The training data format is pure "video-text pairs," and the audio track is entirely ignored in data construction.

### Classification and Separate Handling Strategy for Speech/Sound Effects/Music

Not applicable. There is no classification or differentiated-handling strategy for speech/sound effects/music. Goku is entirely blank on the audio dimension; its value for this research is concentrated on the pure-visual-side data-distribution balancing and the resolution-tiered threshold system.

## Training Coordination

### Multi-Stage Training Curriculum and Data Curriculum Scheduling (basis for stage division: resolution/duration/quality score/modality; low-res→high-res, image→video, short→long)

★ A three-stage training curriculum, with stages divided by "task modality + resolution" — a typical progressive curriculum of image→video, low-resolution→high-resolution:
[Stage 1 — Text-Semantic Pairing]
  Pure text-to-image pretraining, using LAION's 100 million public image-text pairs to establish a basic correspondence between text and visual semantics. This step provides semantic priors for subsequent video training, avoiding the inefficiency of learning semantics from scratch directly on video.
[Stage 2 — Image-Video Joint Learning]
  Mixed training on image and video data, with resolution escalating in cascade: 288×512 → 480×864 → 720×1280. This curriculum corresponds strictly to the resolution bucketing on the data side — 36M videos are available at the 480p stage, shrinking to 24M at the 720p stage, and only 7M at the 1080p stage; data volume decreases as resolution rises, forming a pyramid-shaped curriculum. Image data continues to participate throughout this stage, ensuring image quality and semantic capability do not degrade (this is the core claim underlying Goku's joint modeling).
[Stage 3 — Modality-Specific Finetuning]
  Specialized optimization is performed separately for T2I and T2V; T2I fine-tuning uses the 60 million internal high-quality images; I2V is initialized from T2V and then fine-tuned using the 4.5 million text-image-video triplets.
[Related design] The resolution-tiered filtering thresholds in the data pipeline (Table 4) are tailor-made for this curriculum — every training resolution tier has a batch of data pre-screened to the corresponding strictness standing by, meaning the data side and training side are tightly coupled.

### Changes in Data Mixture Across Training Stages (pretraining/annealing/SFT high-quality subset) ⚠️

The qualitative pattern of data-mixture change between stages is clear, but the quantitative proportions are not disclosed:
(1) **Quality shift from public to internal**: the paper explicitly states "We use public data for pre-training and internal data for fine-tuning" — the 100 million public LAION images are used for pretraining, and the 60 million internal high-quality images for fine-tuning. This is the standard recipe of "large-and-messy base, small-and-refined finish."
(2) **Image proportion gradually decreasing, video proportion rising**: Stage 1 pure image → Stage 2 image-video joint → Stage 3 T2V specialization. Image data continues to be retained rather than completely exiting during the joint stage, in order to prevent video data (generally lower in per-frame quality than curated images) from dragging down frame quality.
(3) **Resolution-driven data-pool shrinkage**: 36M at the 480p stage → 24M at the 720p stage → 7M at the 1080p stage; the later the stage, the less data and the higher the quality bar — itself an implicit form of annealing, since training at the high-resolution end effectively takes place on a high-quality subset.
(4) **Timing of distribution balancing**: the paper does not state whether distribution balancing (Stage 5) is applied once to the full dataset, or with different mixtures at different training stages.
[Not disclosed] The specific image:video mixing ratio at each stage, the number of training steps and tokens seen per stage, and whether there is an explicit high-quality annealing stage. [Partially uncertain]

### Post-Training Data (SFT curated-set size and selection criteria, number of preference pairs and annotation method, reward-model training data)

The post-training step disclosed in the paper is mainly **I2V (image-to-video) fine-tuning**: initialized from the T2V checkpoint, using approximately **4.5 million (4.5M) "text-image-video" triplets**; the paper's original text emphasizes that this batch of data is "sourced from diverse domains to ensure robust generalization" — i.e., the selection criteria emphasize **domain diversity** rather than pure quality score. 4.5 million is about 12.5% of the 36-million video pool.
In addition, Stage 3's T2I-specific fine-tuning uses the 60 million internal high-quality images, which can be regarded as the fine-tuning set on the image side.
[Clearly absent] The paper **does not touch on any preference-learning step at all**: no RLHF/DPO, no preference-pair count or annotation method, no reward model or its training data, no human-feedback alignment process. This stands in sharp contrast to contemporaneous and later work such as Movie Gen, Seedance, and Kling, which emphasize post-training alignment, and represents a notable gap for Goku on the post-training dimension.
[Not disclosed] The specific selection rules for the 4.5M I2V triplets, the first-frame selection strategy (whether aesthetic score is used to pick the first frame), and the fine-tuning step count and hyperparameters.

### Data Processing Infrastructure and Throughput (NeMo Curator/Data-Juicer/in-house, GPU speedup ratio, processing scale, cost) ⚠️

[Uncertain] (on the data-processing side). Section 3 of the paper describes the **training** infrastructure in detail — including fault tolerance and fast failure recovery based on ByteCheckpoint, parallelism strategies (sequence parallelism/FSDP, etc.), stability optimization on large-scale clusters, activation recomputation and memory optimization, and so on, with the goal of "efficient and robust large-scale training" — but all of this serves model-training compute and **does not touch on the infrastructure of the data-processing pipeline**.
The paper does not disclose: the framework used for data processing (no mention of NeMo Curator, Data-Juicer, or any in-house system name), the GPU speedup ratio, the number of clips processed per unit time, the machine-hours and cost spent processing 36M videos, or the distributed scheduling scheme. It can only be inferred from the components used (PySceneDetect, DINOv2, RAFT, OCR, InternVL2.0, Tarsier2, Qwen2) that this is a heavy pipeline reliant on GPU inference — with Tarsier2's whole-video captioning of 36M clips being the largest compute expense item.

## Results Comparison

### Quantitative Impact of Data-Strategy Ablations (distinguishing: filtering-strictness ablation / caption-density-style ablation / data-mixture ablation, and corresponding evaluation metrics) ⚠️

The paper's ablation experiments center mainly on **model and training strategy**; **direct ablations of the data-processing strategy are largely absent** — this is its main shortcoming on the data dimension:
[Existing data-related ablations]
  (1) **Joint image-text training vs. video-only training (a data-mixture-type ablation)** — paper Figure 5(b) shows that a model trained jointly on image-video data with image data added produces markedly better generation results than a model trained on video data alone. This is the only explicit "data-mixture ablation" in the entire paper, supporting Goku's core claim of joint modeling (high-quality images compensate for video's image-quality shortfall and stabilize semantic learning).
  (2) **Model-scale ablation** — comparing Goku-T2V 2B against 8B, it is found that increasing model scale markedly alleviates distorted object structures. This belongs to the model side, not the data side.
  (3) **Effect of prompt rewriting (indirect evidence of a caption-style type)** — after using GPT-4o to expand GenEval's short prompts into semantically preserved, detailed descriptions, the GenEval score reaches 0.76. This indirectly demonstrates that the model's performance is highly dependent on long prompts consistent with the dense caption style used during training — empirical evidence that "caption density/style affects inference performance" — but the paper does not organize this as a formal caption ablation.
[Clearly absent] No ablation of filtering strictness (e.g., the effect of an aesthetic threshold of 4.3 vs. higher/lower on VBench), no controlled experiment on caption density and structure (short vs. long captions), no ablation of the distribution-balancing stage (the effect of balanced vs. unbalanced on per-category metrics) — the absence of the third item is especially regrettable, since distribution balancing is Goku's most distinctive design, yet its benefit is never quantified. [Partially uncertain]

### Evidence for Quality vs. Quantity (cases where small, refined data outperforms large, messy data) ⚠️

The paper does not provide a direct controlled experiment for "small, refined data outperforming large, messy data," but the design of the entire pipeline and several pieces of indirect evidence reflect this orientation:
(1) **Division of labor between public pretraining and internal fine-tuning** — 100 million LAION images (large and messy) are used as the pretraining base, while 60 million internal high-quality images (small and refined) are used to set the tone during fine-tuning, an explicit acknowledgment that public data alone is insufficient to support final quality.
(2) **The higher the resolution, the stricter the bar, the less the data** — the 1080p tier retains only 7M (about 19.4% of the 480p tier's 36M); high-resolution training is essentially conducted on a highly curated small subset, a classic case of "small, refined data setting the quality ceiling at the later stage."
(3) **Aesthetic-score arbitration during deduplication** — keeping the clip with the higher aesthetic score between two duplicates is a quality-first principle applied at the per-sample level.
(4) **Extremely strict OCR thresholds (only 1% text area allowed at the 720p tier)** — willing to sacrifice a substantial amount of data volume to guarantee frame cleanliness.
(5) **The joint-training ablation** (Figure 5b) — adding high-quality image data improves video generation quality, which can be viewed as side evidence that "the value of high-quality samples exceeds that of an equal quantity of homogeneous video samples."
All of the above is design-orientation evidence, not controlled-experiment evidence. [Partially uncertain]

### Alignment Between Training-Data Domain Distribution and Evaluation-Benchmark Taxonomy (e.g. VBench's seven major categories) ⚠️

A fairly strong implicit alignment exists, but the paper does not explicitly discuss the relationship between the two:
[Alignment relationship] Goku's data-side semantic system of 9 top-level categories/86 subcategories (human, scenery, animals, food, urban life, etc.) overlaps heavily in concept space with VBench's semantic-category dimensions (object class, human action, scene, multiple objects, etc.). The distribution-balancing stage's strategy of "overweighting human content + balancing subcategory representation" directly corresponds to Goku's semantic-score performance on VBench: Goku-T2V's overall score is 84.85 (Quality 85.60 / Semantic 81.87), with **human action 79.48**, **scene 85.72**, and **object class 94.40** all markedly higher than contemporaneous open-source models (e.g., HunyuanVideo's human action 68.55, scene 68.68; CogVideoX-5B's scene 66.35), with the lead being largest on the scene metric in particular — consistent with the data side's deliberate strategy of ensuring adequate representation of scene-type subcategories such as scenery/urban life. It can be argued that the distribution-balancing stage is an important source of Goku's advantage on VBench's semantic categories.
[Image side] GenEval (0.76) and DPG-Bench (83.65) examine compositional semantics such as object counting, attribute binding, and spatial relationships, corresponding to the fine-tuning on the 60 million internal high-quality images, rather than to category balancing on the video side.
[Limitations] The paper does not perform an item-by-item attribution analysis of "data category distribution → corresponding benchmark sub-item score"; the alignment described above is an inference made by this research based on the data and results. [Partially uncertain]

## Uncertain Fields

The research information for the following fields is partly uncertain (marked with the ⚠️ source annotation):

- provenance_licensing
- resolution_aspect_distribution
- language_accent_distribution
- funnel_retention_rate
- deduplication
- safety_filtering
- caption_model
- caption_structure
- synthetic_data_synthesis
- human_in_loop
- stage_data_mixture
- data_infra_throughput
- data_ablation
- quality_over_quantity_evidence
- benchmark_taxonomy_alignment
