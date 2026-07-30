# Script-a-Video (the core deliverable is the MTSS representation paradigm, full name Multi-Stream Scene Script)

> Topic: Data processing for video generation models (including joint audio-video generation): data cleaning pipeline, data distribution, annotation approach

[← Back to home](../index.md)

**Contents**: [Basic Information](#basic-information) · [Data Scale and Distribution](#data-scale-and-distribution) · [Cleaning Pipeline](#cleaning-pipeline) · [Captioning Approach](#captioning-approach) · [Audio-Video Alignment](#audio-video-alignment) · [Training Coordination](#training-coordination) · [Effectiveness Comparison](#effectiveness-comparison)

## Basic Information

### Name

Script-a-Video (the core deliverable is the MTSS representation paradigm, full name Multi-Stream Scene Script)

### Releasing institution/company ⚠️

Tencent Hunyuan Team. The paper is attributed collectively to the team (the author list only states "Tencent Hunyuan Team"); the end of the main text has a Project Contributors section, but it is not expanded with specific names in the arXiv HTML version, so the first author and corresponding author cannot be confirmed. [Uncertain]

### Release date (technical report/paper/open-source date) ⚠️

Published on arXiv in April 2026 (arXiv:2604.11244). The latest visible version is currently v2, dated April 15, 2026 (cs.CV). v1 was submitted slightly earlier in the same month (arXiv:2604.11244v1). As of July 2026, no accompanying GitHub repository, project homepage, or conference acceptance record has been found. [Uncertain]

### Type (model/dataset/toolchain/evaluation benchmark)

A methodology/representation-paradigm paper that simultaneously carries three properties — "annotation schema definition + internal dataset construction + downstream generative model modification":
[Main body] Proposes the MTSS structured audio-video caption representation paradigm (a JSON-style four-stream schema definition), which belongs to annotation methodology rather than a model or a dataset.
[Byproduct 1] An internal MTSS-annotated dataset of 500K video clips (not open-sourced).
[Byproduct 2] A dedicated captioning model, Qwen3-Omni-MTSS-FT, fine-tuned from Qwen3-Omni-Instruct (not open-sourced).
[Byproduct 3] A multi-shot joint audio-video generation model built on top of LTX-2 (introducing two architectural improvements, Shot-Aware Structured Attention and Identity Customization; not open-sourced).
[Byproduct 4] An internal evaluation set (125 single-shot + 100 multi-shot samples, not open-sourced).
This is not the release of an evaluation benchmark — the benchmarks used for evaluation are existing ones: Video-SALMONN-2, UGC-VideoCap, Daily-Omni, and WorldSense.

### Degree of openness (whether weights/code/data/pipeline are each open source)

Extremely low degree of openness, of the "paper disclosure only — zero code, zero weights, zero data" type:
[Weights] Not open source. Neither Qwen3-Omni-MTSS-FT nor the LTX-2-based generation model has been released.
[Code] Not open source. The full paper contains no GitHub link, no project homepage URL, and no open-source commitment statement.
[Data] Not open source. The 500K MTSS-annotated dataset is explicitly described as an "internal dataset"; the four generation-side datasets (400K identity-centric / 250K multi-shot / 870K cinematic pairs / 60K cinematic alignment pairs) are likewise internal data; the 125+100-sample internal evaluation set has also not been released.
[Pipeline] At the methodological level, disclosure of the MTSS schema field definitions is quite thorough (all field names and semantics for the four streams — Reference/Shot/Event/Global — are explained one by one in Section 3 of the main text, accompanied by a complete script example in Figure 3); reproducibility is mainly reflected in the fact that "the schema can be re-implemented by others"; however, the data cleaning pipeline, the verbatim Gemini-2.5-Pro annotation prompt, and the filtering thresholds are not disclosed.
[License] arXiv.org perpetual non-exclusive license (paper text only), no model/code license.
[Overall assessment] Its value lies in the paradigm and schema design itself being directly reusable, rather than in any directly usable asset.

### Whether joint audio-video generation is supported, and the implementation approach (native joint/cascade/MoE fusion)

Joint audio-video generation is supported, but two levels must be distinguished:
[Positioning of this work] Script-a-Video itself is not a new audio-video generation foundation model — MTSS is a "structured conditioning representation on the input side." The generation capability is validated by performing condition replacement and lightweight architectural modification on an existing foundation model.
[Generation foundation model] LTX-2 is chosen as the generation framework, for two reasons: (1) its Gemma-series VLM encoder is naturally adept at parsing JSON-style structured syntax such as MTSS, and can extract fine-grained semantic instructions from the factorized-stream fields; (2) its asymmetric dual-stream Diffusion Transformer (asymmetric dual-stream DiT) architecture is itself designed for joint audio-video synthesis, allowing MTSS's Shot stream and Event stream to be mapped respectively into the latent spaces of the video branch and the audio branch.
[Implementation category] Native joint generation (a single generation process simultaneously produces synchronized video and audio), neither cascaded nor MoE fusion.
[Two architectural improvements]
1) Shot-Aware Structured Attention: splits the Gemma-3 text embedding along MTSS shot boundaries, then lets the video tokens corresponding to each shot perform cross-attention only with the semantic segment of that shot, achieving inter-shot context isolation and preventing cross-shot semantic bleed-through.
2) Identity Customization: through reference VAE features plus learnable reference-learnable-tokens, explicitly aligns the ID symbols in the Reference stream (e.g., "PERSON_1") with the corresponding reference image, serving as a relational bridge between visual identity and linguistic reference.
[Multimodal input form] Multimodal information is fed into Gemma-3 in an interleaved image-text format, providing semantic representations for both the video and audio branches simultaneously.
[Task form] The MTSS triplet (S_ref, S_shot, S_eve) → a synchronized video-audio pair (V, A), with the goal of simultaneously satisfying identity persistence and temporal-auditory precision.

### List of research information sources (paper/technical report/official documentation/news URLs, with each source's nature annotated: first-party official/same-team corroboration/third-party reporting)

- [First-party official] arXiv abstract page https://arxiv.org/abs/2604.11244 — paper title, institution (Tencent Hunyuan Team), core abstract figures (Video-SALMONN-2 average total error rate reduced by 25%, Daily-Omni average improvement of 67%, multi-shot generation human-rated identity consistency +45% / audio-video alignment +56% / temporal controllability +71%).
- [First-party official] arXiv full-text HTML v2 https://arxiv.org/html/2604.11244v2 (dated 2026-04-15, cs.CV) — the sole direct source for the great majority of fields in this entry: Section 3's complete field definitions for the MTSS four-stream schema, Section 4.1's captioning-dataset construction and Table 1/Table 2, Section 4.2's generation experiments and Table 3/Table 4, Section 4.2.2's training details, and the Limitations section.
- [First-party official] arXiv full-text HTML v1 https://arxiv.org/html/2604.11244v1 — used for cross-checking against v2 (the v1 abstract states Daily-Omni improved by 67%, while v2's Figure 1 and main text separately give figures of 110% and 127%, indicating that the numbers differ across versions/different accounting conventions).
- [First-party official] arXiv PDF https://arxiv.org/pdf/2604.11244 — figure layout and the MTSS script example in Figure 3.
- [Third-party corroboration] Tencent Hunyuan GitHub organization https://github.com/Tencent-Hunyuan — used to verify whether an accompanying open-source repository exists; as of the time of research, this organization contains repositories such as HunyuanVideo, HunyuanVideo-1.5, HunyuanVideo-Avatar, HunyuanVideo-Foley, and HunyuanVideo-I2V, but no Script-a-Video / MTSS-related repository, from which it is judged that this work has not been open-sourced.
- [Note] No third-party media coverage, technical blog analysis, or community reproduction discussion was found; at the time of this research the work was still newly released with limited dissemination.

## Data Scale and Distribution

### Training data volume (number of videos/hours/tokens, pretraining and SFT separated)

The data scale disclosed in the paper is divided by use into two independent sets — "captioning side" and "generation side":
[Captioning side (used to train the MTSS annotation model)] 500K high-quality video clips (500,000 clips). Only the count is given; no total hours, average duration, or token count is given. If roughly estimated at a typical 5-10 second clip length, this would be approximately 700-1400 hours, but the paper gives no duration figures of any kind — this is an inference, not a disclosure.
[Generation side (used to train the LTX-2-modified model)] Four datasets are used across stages:
- ID Customization stage: 400K identity-centric dataset, trained for 3 epochs;
- Multi-shot Control stage: 250K multi-shot sequences, trained for 1.5 epochs;
- Audio-Visual Synergy stage: 870K cinematic pairs, trained for 3 epochs;
- Final joint fine-tuning stage: 60K high-fidelity cinematic alignment pairs interleaved with 250K multi-shot sequences, trained for 15K steps.
The generation-side data totals roughly 1.5M in scale (including cross-stage reuse), again given only in counts, not hours.
[Evaluation side] Internal evaluation set of 125 single-shot samples + 100 multi-shot samples, 225 in total.
[Pretraining vs. SFT split] On the captioning-model side there is no pretraining — supervised fine-tuning (SFT) is performed directly on the open-source Qwen3-Omni-Instruct in a single stage, and the 500K clips constitute the entire SFT data; on the generation side, likewise, multi-stage fine-tuning is performed on the already-pretrained LTX-2, with no training from scratch.

### Data source composition (proprietary/public datasets/web scraping/licensed procurement/synthetic data)

Disclosure is extremely brief, consisting only of domain-level qualitative description, with no source-channel composition ratios:
[Captioning side, 500K] Explicitly described as an "internal dataset" (Tencent's own internal data), covering three domains: film, television, and lifestyle content. Whether this content comes from an owned-copyright library, licensed procurement, or web collection is not stated.
[Generation side] The provenance of the four datasets is likewise not disclosed by channel; only the nature can be inferred from the naming: identity-centric (centered on character identity, presumably material with clearly defined human subjects), multi-shot sequences (multi-shot sequences, presumably from naturally multi-shot content such as film/TV drama), cinematic pairs (cinematic-grade audio-video pairs), cinematic alignment pairs (high-fidelity cinematic-grade alignment pairs).
[Evaluation side, 225 clips] Covers four categories: movie and TV drama clips, short-form videos, indoor scenes, and outdoor scenes.
[Public datasets] No public video dataset was used for training (VGGSound, AudioSet, Panda-70M, etc. are not mentioned); the only public resources used are the evaluation benchmarks (Video-SALMONN-2 testset, UGC-VideoCap, Daily-Omni, WorldSense).
[Synthetic data] No synthetic video/audio content was used; however, the "annotations" of the 500K dataset are entirely automatically generated by Gemini-2.5-Pro, which counts as synthetic annotation rather than synthetic content.
[Licensed procurement] Not mentioned.
Overall, the level of data-source disclosure is markedly lower than comparable works (e.g., UniVerse-1 itemizes YouTube content types, MOVA gives a tiered composition breakdown) — this is a typical "domain stated, channel not stated" disclosure style for a large company's internal data.

### Data compliance and provenance (proportion of licensed data, rights-cleared datasets, C2PA, etc.) ⚠️

The paper does not touch on data compliance and provenance at all: no proportion of licensed data is given, no rights-cleared dataset is declared, no C2PA or any output-side watermark/provenance marker is mentioned, and copyright, portrait rights, or data-use licensing are not discussed.
Identifiable compliance risk points:
1) Both the captioning-side 500K set and the generation-side data draw heavily on film / television / TV drama content, a copyright-highly-sensitive source category; the paper does not explain how it was obtained or its licensing status;
2) The Reference stream anchors people at a fine-grained appearance level (clothing, accessories, hairstyles, etc.); the Identity Customization module supports injecting a real person's identity from a reference image; the Event stream records verbatim dialogue lines bound to a speaker ID — this combination is more sensitive with respect to portrait rights and deepfake risk than purely scene-descriptive captioning, and the paper gives zero discussion of it;
3) None of the assets have been publicly released, which objectively lowers the risk of external misuse, but also means there is no model-card-level usage-restriction statement.
This field is entirely blank in the paper. [Uncertain]

### Clip duration distribution and segmentation strategy ⚠️

The paper discloses no clip duration information at all: no average duration, duration bounds, or duration histogram is given for the 500K clips, and it is not stated whether fixed-length truncation or bucketing was applied after segmentation.
Clues that can be indirectly inferred:
1) The Shot stream in MTSS uses "time_range" as the anchoring field for each shot, and the Event stream likewise carries a "time_range," with intra-description timestamps embedded within the Shot's visual_description to anchor micro-actions to the global timeline — indicating that each clip has an explicit global timeline long enough to accommodate multiple shots and multiple audio events;
2) The generation side distinguishes single-shot (125 evaluation samples) from multi-shot (100 evaluation samples), and multi-shot samples must necessarily be longer than single-shot ones;
3) The evaluation metrics include Shot Boundary Deviation (shot boundary deviation, measured in frames, with an optimum of 0.38 frames), indicating that the generation side has a fixed duration and frame-rate configuration, but the specific value is not disclosed.
The segmentation strategy is likewise not disclosed — the paper does not explain how the 500K clips were cut from long videos (whether PySceneDetect was used, whether cutting followed scene boundaries, or whether selection was manual). This is one of the weakest links in this work's data-side disclosure. [Uncertain]

### Resolution/aspect-ratio distribution and bucketing strategy ⚠️

The paper does not discuss resolution and aspect ratio at all: no resolution threshold, resolution distribution, or aspect-ratio distribution is given for the training data; no bucketing or multi-resolution training strategy is described; the output resolution and frame-rate configuration of the generation model is not stated.
The only indirect information is the resolution capability of the generation foundation model LTX-2 itself, but the paper does not state the training/inference resolution tier actually used in its modified version.
Among the evaluation metrics, Shot Boundary Deviation is reported in units of "frames" (3.79 → 0.38 frames), indicating a fixed frame-rate setting exists, but the frame-rate value is not given. [Uncertain]

### Category/domain distribution and mixture strategy (proportional control and concept balancing across people, actions, scenes, styles, etc.) ⚠️

Disclosure at the domain level stays at listing categories, with no mixture-ratio numbers and no description of any concept-balancing mechanism:
[Domain composition of the captioning-side 500K] Three qualitative categories are listed: film, television, and lifestyle. The proportion of each is not disclosed, nor is it stated whether any mixture control was applied. This selection clearly leans toward content that is "narrative, has dialogue, has multiple shots, and has characters" — highly coupled with the design goals of the MTSS schema: the Reference stream needs recurring characters, the Shot stream needs genuine shot changes, and the Event stream needs dialogue and sound effects. In other words, the data domain was selected in reverse from the schema's requirements, rather than pursuing general coverage.
[Implicit domain division on the generation side] The four datasets are actually divided by "capability dimension" rather than "subject-matter dimension": identity-centric (character-identity capability), multi-shot (shot-structure capability), cinematic AV pairs (audio-video synergy capability), cinematic alignment pairs (high-fidelity alignment capability). This is an "organize data by target capability" approach, where each dataset serves the specific learning objective of one training stage, rather than balanced sampling by content subject matter.
[Domain coverage on the evaluation side] The 225 evaluation samples cover four categories — movie/TV drama clips, short-form videos, indoor scenes, and outdoor scenes; the paper states this "covers a diverse range of categories and scenarios," but the proportion of each is not given.
[Concept balancing] The paper describes no concept-balancing, long-tail supplementation, category-quota, or resampling mechanism.
[The category system internal to MTSS] Notably, the MTSS schema itself has a built-in entity-classification system: the Reference stream divides entities into four categories — person, object, animal, scene — and only retains entities that are integral to the main plot, with peripheral elements uniformly demoted to the global scene description. This is an "entity filtering driven by narrative importance," constituting an implicit domain structure at the annotation level, but it is not a sampling ratio for training data.
Overall, the mixture-ratio numbers for the training data's subject matter are an information gap. [Uncertain]

### Audio-category distribution and mixture (proportions and control strategy for speech/sound-effect foley/music/ambient sound/silence) — a dimension unique to AV models ⚠️

Audio categories have a clear three-way classification definition at the MTSS schema level, but the proportion of each category in the training data is entirely undisclosed:
[The three schema-defined audio event types] The Event stream strictly classifies all audio events into three types:
1) dialogue — carries speaker (bound to a Reference ID) and line (verbatim dialogue text);
2) sfx (sound effect) — must be produced by a visible subject in the frame ("sound effects must be generated by a visible subject");
3) music.
[A fourth category: global audio] Sounds that do not constitute an independent event (ambient background noise, background music/atmosphere) do not enter the Event stream, but are instead placed into the "global_audio" field of the Global stream. In the paper's own words: "Irrelevant background noise is filtered into the global audio metadata" — i.e., irrelevant background noise is "filtered into" the global audio metadata rather than discarded. This is a four-tier stream design: visually-sourced sound effects / dialogue with a speaker / music / a fallback global ambience.
[Admission principle: strict audio-visual coupling] The admission criterion for the Event stream is the "strict audio-visual coupling principle" — only audio events with a direct visual correspondent or thematic relevance are extracted. This principle essentially completes the removal and demotion of "off-screen sound sources" already at the annotation stage: sounds without visual correspondence never become independent events, thereby ensuring that every entry in the Event stream is a visually-grounded audio event learnable by a generative model. This is this work's most methodologically valuable design on the audio side.
[Handling of concurrent sound sources] Multiple sound sources occurring simultaneously are "factorized into parallel event entries" rather than merged into a single mixed description. This ensures that in multi-source scenes, each sound has an independent time_range, speaker, and description, and can be edited and controlled independently.
[Proportion figures] The paper does not disclose what proportion each of the three event types accounts for in the 500K dataset, the average number of events per clip, or the proportion of dialogue-type events. [Uncertain]
[Sampling weights] No mechanism for adjusting training sampling weights by audio category is mentioned.

### Narrative-structure distribution (single-shot vs. multi-shot, average clip duration, shot-count distribution, whether native audio tracks are included) ⚠️

Narrative structure is a first-class citizen of MTSS, thoroughly handled at the schema level, but distribution statistics are missing:
[Single-shot vs. multi-shot] Both are explicitly and simultaneously supported and explicitly modeled. The Shot stream itself is "a decomposition of the visual presentation into a sequence of cinematic segments," with each shot carrying an independent time_range. The evaluation set is divided into 125 single-shot / 100 multi-shot samples (roughly 5.6:4.4), indicating that multi-shot is a first-class evaluation scenario rather than a secondary one.
[Shot-count distribution] The average number of shots per clip in the 500K dataset, and the distribution range of shot counts, are not disclosed. [Uncertain]
[Average clip duration] Not disclosed.
[Whether native audio tracks are included] Necessarily yes — construction of the Event stream depends on dialogue and sound effects in the real audio track, and the Global stream's global_audio depends on real ambient sound; without a native audio track, the majority of the fields in the entire schema would be empty. However, the paper does not explicitly state a pre-filtering rule such as "videos without an audio track are excluded."
[Explicit modeling of shot language] The Shot stream has a built-in camera field, dedicated to recording professional film-language attributes: camera movements, perspectives, and scales (shot sizes). This makes shot language a field that can be structurally retrieved and controllably generated, rather than scattered within free text.
[Mechanism guaranteeing narrative continuity] Cross-shot narrative coherence is guaranteed by Relational Grounding: the references_in_shot array maps the subjects appearing in each shot to persistent Reference IDs, and active_events links a shot to concurrent auditory events. Consequently, the same character appearing across multiple shots need not be re-described in appearance — only the same ID need be referenced — which directly resolves the "identity drift caused by repeated description" problem found in monolithic captions.
[Quantified real generation effect] Shot Boundary Deviation drops from the LTX-2-AV baseline's 3.79 frames to 0.38 frames (in the Ours w/o ID configuration); human-rated MSC (multi-shot consistency) rises from 1.00 to 2.49-2.62, demonstrating that the temporal-segmentation prior provided by the Shot stream genuinely makes multi-shot controllability possible.

### Language/accent distribution (data foundation for multilingual lip-sync capability) ⚠️

The paper gives no language/accent distribution data at all: no supported languages are listed, no per-language proportions are given, no accent annotation is done, and multilingual lip sync is not discussed.
Clues that can be indirectly inferred:
1) The evaluation stage explicitly uses "jieba-based tokenization for CJK text" when computing WER, indicating that Chinese (or CJK more broadly) speech content genuinely exists in generation and evaluation, and that the training data is not purely English;
2) The data source is predominantly film / television / TV drama and is Tencent's internal data, so there is a relatively high likelihood that Chinese-language film/TV content dominates, but the paper offers no corroborating evidence;
3) The MTSS Event stream provides a "line" field for dialogue events recording verbatim text; the schema places no restriction on language at the schema level, but no language or accent field is provided;
4) The Event stream's "description" field is used to capture nuanced semantics such as emotional shifts and vocal techniques — the design closest to a "speaker attribute" — but it remains free text rather than a structured accent/language label.
Language and accent are an entirely blank field of information. [Uncertain]

## Cleaning Pipeline

### Overall structure of the cleaning funnel (number of filtering tiers, order of each tier) ⚠️

A clear distinction must be made: this work's "pipeline" focus is on the construction logic of the annotation schema, not on the data-cleaning funnel. The cleaning process is almost entirely undisclosed.
[Data cleaning funnel] The paper covers this in just one sentence: "we curated an internal dataset comprising 500K high-quality video clips from film, television, and lifestyle domains." The criterion for judging "high-quality," the number of filtering tiers, the order of each tier, and the tool selection are all undisclosed. This is this work's most conspicuous disclosure gap. [Uncertain]
[Annotation pipeline (thoroughly disclosed)] The actual processing flow is two steps:
Step 1: Gemini-2.5-Pro is used to generate an MTSS-format annotation for each of the 500K clips one by one ("Each clip was annotated using Gemini-2.5-Pro");
Step 2: this batch of data is used to perform supervised fine-tuning (SFT) on the open-source Qwen3-Omni-Instruct, yielding the dedicated annotation model Qwen3-Omni-MTSS-FT.
This is the classic "strong-teacher-model distillation into an open-source student model" paradigm, with no intermediate quality-control, review, or resampling step disclosed.
[The internal processing logic of MTSS annotation (this work's true methodological core)] This can be viewed as a five-step "information-reorganization pipeline":
1) Entity extraction and filtering → entities are classified into person/object/animal/scene based on narrative importance and placed into the Reference stream, with peripheral entities demoted to the global scene description;
2) Visual segmentation → segmentation by shot into the Shot stream, with each segment carrying three layers of information: time_range, visual_description, and camera;
3) Audio-event extraction and routing → events of type dialogue/sfx/music are filtered out into the Event stream according to the "strict audio-visual coupling" principle, background noise without visual correspondence is demoted to global_audio, and concurrent sound sources are split into parallel entries;
4) Global fallback → the three fields scene_description / global_style / global_audio carry macro-level information not belonging to any specific stream;
5) Relational reconnection (Relational Grounding) → identity linking (the references_in_shot array points to Reference IDs, and an Event's speaker points to a Reference ID) + temporal linking (active_events associates a shot with its concurrent events, and intra-description timestamps anchor micro-actions to the global timeline).
This "decouple first, then reconnect" structure is the methodological thesis of the entire paper: decoupling solves redundancy and editability, reconnection solves consistency.
[Generation-side pipeline] See the multi_stage_curriculum field (a four-stage progressive training curriculum).

### Quantitative funnel retention rate (input/output volume at each filtering tier and the final retention rate, e.g., Apollo's 27%) ⚠️

The paper gives no quantitative funnel data whatsoever: the total volume of originally collected video is not disclosed (the numerator, 500K, is known, but the denominator is entirely missing), no per-tier input/output volumes are given, no per-tier or overall retention rate is given, and no original pool size or retention rate is given for any of the four generation-side datasets (400K/250K/870K/60K).
It is impossible to compute an overall retention rate comparable to Apollo's 27% or MOVA's 26.39%.
One "retention-rate" analogy worth noting: at the annotation level, MTSS has an implicit information-retention decision — the Event stream's strict audio-visual-coupling principle "filters" a large amount of audio information into global_audio, and the Reference stream's narrative-importance filtering "filters" peripheral entities into the global scene description. But the paper does not quantify what proportion of information each of these two demotion channels absorbs. [Uncertain]

### Shot-segmentation method (PySceneDetect/in-house model/shot-aware splitting) ⚠️

This work's shot segmentation presents a key dual role, requiring two levels to be distinguished:
[Data-preprocessing level (undisclosed)] How the 500K clips were cut from long videos, whether PySceneDetect or an in-house detector was used, and how thresholds were set are entirely unexplained in the paper. This is a gap on the cleaning side. [Uncertain]
[Annotation level (thoroughly disclosed and the core innovation)] MTSS does not treat shot segmentation as an intermediate byproduct discarded during preprocessing, but instead permanently retains shot structure as a first-class annotation field within the Shot stream — each shot carries a precise "time_range," along with two layers of description: visual_description (an objective, chronological narration of the core action) and camera (professional film language: movements/perspectives/scales), plus two relational fields, references_in_shot and active_events. Consequently, a single training sample can internally contain multiple shots, rather than being chopped into multiple independent single-shot samples. This runs opposite to the mainstream practice of works such as UniVerse-1, which "retain only single-shot clips after segmentation."
[Downstream benefit] Retaining shot structure allows the generation model to directly learn the capability of "executing a shot transition at a specified timestamp." The Shot-Aware Structured Attention introduced in the paper does exactly this — it uses shot boundaries to partition the Gemma-3 embedding, so that each segment of video tokens attends only to the semantics of its own shot, achieving inter-shot context isolation.
[Quantitative validation of segmentation precision] The Shot Boundary Deviation metric (the frame-level absolute deviation between the shot boundaries of the generated video and those specified by the script): LTX-2-AV baseline 3.79 frames → LTX-2-AV-MTSS (only the prompt is swapped to MTSS, architecture unchanged) 3.27 frames → Ours w/o ID 0.38 frames → Ours(Full) 1.36 frames. The paper points out that the Full configuration actually rises back to 1.36 frames — a potential trade-off between identity-injection features and temporal shot precision, attributed to the design of the interface between the VLM encoder and the DiT, left as future architectural optimization.

### Quality filtering (aesthetic scoring, sharpness, OCR text filtering, letterbox/watermark/logo detection) ⚠️

The paper discloses no quality-filtering details of any kind: no aesthetic scorer (LAION-Aesthetic / DOVER / VQA-type models are not mentioned), no sharpness or blur metric, no burned-in subtitle (OCR) detection, no letterbox detection, no watermark/logo detection, no bitrate or compression-artifact detection, and no threshold values of any kind are given. All quality control is compressed into a single adjective, "high-quality video clips." [Uncertain]
The only two indirect clues related to "quality":
1) The choice of data domain itself constitutes an implicit quality filter — film / television / cinematic pairs are all professionally produced content, naturally superior to UGC in image quality, composition, and audio capture;
2) The final generation-side stage uses "60K high-fidelity cinematic alignment pairs," and the wording "high-fidelity" implies the existence of a stricter high-quality subset filter, but the filtering criteria are not disclosed.
The blankness of this field relates to this work's positioning: the paper's contribution claim rests on the annotation paradigm rather than the cleaning pipeline, so the cleaning portion is omitted entirely. This is an acceptable trade-off for research focused on annotation approach, but for research focused on the cleaning funnel, this entry has very little reference value.

### Motion filtering (optical-flow/motion-score thresholds, removal of static and jittery content) ⚠️

The paper describes no motion-filtering step at all: no optical-flow computation, no motion-score threshold, no static-shot removal, no jitter/shake detection. [Uncertain]
Motion information in this work does not appear as a "filtering dimension" but as an "annotation dimension":
1) The Shot stream's camera field explicitly records camera movements, turning camerawork into a describable, controllable semantic field;
2) The Shot stream's visual_description requires an objective, chronological narration of the core action, and anchors micro-actions to the global timeline through intra-description timestamps — i.e., motion is "structurally described" rather than "score-filtered."
An observation related to motion exists on the evaluation side: Intra-Shot Subject Consistency (the frame-to-frame average cosine similarity of DINOv2 [CLS] features) reaches as high as 0.87 for the LTX-2-AV baseline, but the paper explicitly points out this is an illusion — the high score results from the model failing to inject reference features and generating near-static content, with minimal frame-to-frame variation ("trivially static content with minimal frame variation"). This work's full solution maintains a "lower" score of 0.59-0.62, which instead corresponds to genuinely dynamic content. This is an important warning about "static content inflating consistency metrics," with reference value for motion-filtering design on the data side.

### Deduplication method (exact dedup and embedding-based semantic dedup recorded separately) ⚠️

The paper makes no mention of any deduplication step: neither hash/fingerprint-level exact deduplication nor embedding-based semantic deduplication is present, and cross-source or cross-stage duplication is not discussed. [Uncertain]
Potential duplication risk objectively exists: the four generation-side datasets (400K identity-centric, 250K multi-shot, 870K cinematic pairs, 60K cinematic alignment pairs) all draw from the same internal film/TV material library, and the final joint fine-tuning stage explicitly reuses the aforementioned 250K multi-shot sequences, indicating that cross-stage data reuse is intentional; but whether there is overlap of same-origin clips across the different stage datasets is not explained at all.
Worth mentioning: MTSS's design does build a redundancy-reduction mechanism in at the "description level" — the Reference stream's centralized entity bank means recurring characters/scenes need only be described once, with subsequent Shot and Event entries referencing the same ID only; the paper calls this eliminating "redundant re-description." But this is elimination of textual redundancy, a different matter from dataset-sample deduplication, and the two should not be conflated.

### VLM/LLM as a quality-control judge (multimodal large-model quality scoring and mismatch removal, the 2026 trend from shallow scorers toward large-model semantic judgment) ⚠️

This work is a typical case of "large model as annotator," but strictly speaking it does not use a large model as a data-quality judge; rather, it uses it in two roles — generator and judge of the output:
[Role 1: Gemini-2.5-Pro as annotation generator] All MTSS annotations for the 500K clips are automatically generated by Gemini-2.5-Pro, with no other model involved, no multi-model voting, and no consistency cross-validation. The reasonableness of choosing Gemini-2.5-Pro is corroborated by Table 1: it achieves a composite score of 93.97 on UGC-VideoCap (95.51 in the MTSS version) and a total error rate of 0.3959 on Video-SALMONN-2 (0.2511 in the MTSS version), making it the strongest model in the whole table — giving quality assurance as a teacher model.
[Role 2: Gemini-2.5-Pro as evaluation judge] All caption and reasoning-benchmark evaluations in Table 1 and Table 2 use Gemini-2.5-Pro as the judge model. There is a methodological self-referencing risk here: the annotator of the training data and the judge of the evaluation are the same model, which may carry a systematic preference for the MTSS format; the paper does not discuss or mitigate this potential bias.
[Role 3: Gemma4-31B-it as generation-quality judge] The Semantic Following metric on the generation side is scored independently by the Gemma4-31B-it vision-language model, across four sub-dimensions — Subject (identity and appearance fidelity), Action (correctness of motion and interaction), Scene (accuracy of background/environment/spatial layout), Style (adherence to visual style/color palette/atmosphere) — and then arithmetically averaged. This is a standard VLM-as-judge usage.
[Missing step: annotation quality control] The paper describes no quality-control mechanism for Gemini-2.5-Pro's annotation output — no hallucination detection, no format-validity checking (MTSS is a JSON-style structure that in principle requires parse validation), no Reference-ID reference-integrity checking (whether Shot's references_in_shot and Event's speaker all point to existing IDs), no timestamp-validity checking (whether time_range is out of bounds/overlapping), and no manual spot-checking. For a structured schema so heavily dependent on cross-reference integrity, this is a fairly notable methodological gap. [Uncertain]
[Related limitation admitted by the paper itself] The Limitations section acknowledges: generating accurate, deeply structured scripts places extremely high demands on a foundation model's cross-modal understanding capability; current open-source MLLMs still have limitations in three respects — precise temporal localization, robust ASR, and accurate audio-visual entity-event association; how to enable a more compact open-source architecture to reach Gemini-level scripting ability while effectively suppressing hallucination remains an open problem. This statement conversely confirms that annotation quality is highly dependent on the teacher model itself, rather than on post-hoc quality control.

### Safety and compliance filtering (NSFW, copyright, face/privacy) ⚠️

The paper does not touch on safety and compliance filtering at all: no NSFW detection, no violence/sensitive-content filtering, no copyright filtering, no face/privacy protection measures, and no model-card-level safety statement or usage restrictions. [Uncertain]
A notable risk gap: this work's Identity Customization module supports injecting a real person's identity from a reference image and maintaining it consistently across multiple shots; the Event stream records verbatim dialogue lines bound to a speaker; the Reference stream anchors people at a fine-grained appearance level (clothing, accessories, hairstyle) — this combination of capabilities constitutes precisely the technical precondition for high-quality deepfakes, and the paper gives zero discussion of misuse risk. The mitigating factor is that none of the model weights or data have been publicly released.

## Captioning Approach

### Captioning model used (in-house VLM/open-source model, model scale) ⚠️

Two types of models are involved on the annotation side, with a clear division of roles:
[Teacher model / data annotation model] Gemini-2.5-Pro (Google, closed-source). Responsible for all MTSS annotation generation for the 500K clips — the sole annotation source for the entire dataset. Parameter scale is not public (closed-source model).
[Student model / dedicated annotation model] Qwen3-Omni-Instruct (Alibaba Tongyi Qianwen's open-source omni-modal model). Supervised fine-tuning is performed on the 500K MTSS data (the paper's original text reads "supervised sine-tuning," presumably a typo for "supervised fine-tuning") to obtain Qwen3-Omni-MTSS-FT. The specific parameter tier used is not disclosed (Qwen3-Omni has multiple scale variants). [Uncertain]
[Quantified distillation effect] After fine-tuning on MTSS data, the student model closes in dramatically on the teacher:
- Video-SALMONN-2 total error rate: Qwen3-Omni native 0.5853 → zero-shot MTSS prompting 0.5156 → post-MTSS-fine-tuning 0.3913 (the teacher Gemini-2.5-Pro's native figure is 0.3959, i.e., after fine-tuning, this 8B/30B-class open-source model has already slightly surpassed Gemini-2.5-Pro's native captioning on this metric);
- UGC-VideoCap composite score: 62.80 → 71.54 → 85.11 (teacher: 93.97);
- Daily-Omni: 0.1806 → 0.4117 → 0.5945 (teacher: 0.6825);
- WorldSense: 0.1569 → 0.3106 → 0.3875 (teacher: 0.4332).
[Comparison baselines (all captioning models)] AVoCaDO, ASID-Captioner-7B (7B parameters), Qwen3.5-Omni-Flash (closed-source), Gemini-2.5-Pro (closed-source).
[Key finding: MTSS is a zero-cost gain] The paper emphasizes that MTSS is effective for all tested models, including in the zero-shot prompting scenario without any fine-tuning at all — i.e., merely changing the output-format requirement from "write a paragraph" to "fill in the MTSS four-stream structure" drops Gemini-2.5-Pro's total error rate from 0.3959 to 0.2511, and Qwen3.5-Omni-Flash's from 0.5217 to 0.3655. This indicates that the structured schema itself is an effective prompt-engineering technique that any team can reuse at zero cost.
[Text encoder on the generation side] LTX-2's built-in Gemma-3 VLM encoder is responsible for parsing MTSS's JSON-style syntax and outputting the semantic representation fed into the video/audio dual branches; multimodal input is fed in interleaved image-text format.

### Caption density and degree of structuring (short/long/dense description, structured fields such as camera movement, style tags)

MTSS is one of the most highly structured captioning schemas in this research — of the "deeply structured + explicit relational graph" type, forming a paradigm-level contrast with mainstream free-text long descriptions:
[Overall form] A JSON-style four-stream structure, rather than a natural-language paragraph. The four streams are Reference, Shot, Event, and Global.
[Stream 1: Reference Stream (entity bank, answers WHO and WHERE)]
- Positioning: a persistent Entity Bank, providing identity anchors for the entire script;
- Entity classification: person / object / animal / scene;
- Filtering principle: only entities integral to the main plot are included, with peripheral elements demoted to the Global stream's scene description;
- Fields: semantic_description (overall state description of the entity), timestamp (time of appearance), appearance_anchor (appearance anchor);
- Inside appearance_anchor: a general-purpose detail_description applicable to all entity types; for the person category, additional fine-grained attributes are expanded — clothing, accessories, hairstyles;
- Design benefit: subsequent streams reference the persistent Reference ID rather than repeating the description, fundamentally guaranteeing absolute cross-shot identity consistency while eliminating textual redundancy.
[Stream 2: Shot Stream (visual segmentation, answers WHAT-visual and HOW)]
- Each shot is anchored by a precise time_range;
- Visual-spatial layer: visual_description (an objective, chronological narration of the core action) + the camera field (professional film language: camera movements, perspectives, scales);
- Relational layer: the references_in_shot array (mapping subjects visible in the frame to Reference IDs) + active_events (linking to concurrent auditory events within this shot);
- Temporal layer: intra-description timestamps embedded within the description text, anchoring micro-actions to the global timeline — the paper calls this achieving "surgical synchronization."
[Stream 3: Event Stream (audio events, answers WHAT-audio)]
- Admission principle: strict audio-visual coupling — only audio events with a direct visual correspondent or thematic relevance are included; sound effects must be produced by a visible subject in the frame;
- Event types: dialogue / sfx / music;
- Fields: type, time_range, and a content block containing speaker (relationally bound to a Reference ID), line (verbatim dialogue text), description (capturing nuanced semantics such as emotional shifts and vocal techniques);
- Concurrency handling: multiple simultaneous sound sources are split into parallel event entries rather than merged;
- Demotion channel: irrelevant background noise is filtered into the Global stream's global_audio;
- Alignment: precisely aligned with the visual track through micro-level timestamps.
[Stream 4: Global Stream (macro context)]
- scene_description (overall description of the video event), global_style (overall aesthetic style or genre), global_audio (ambient sound and background music that do not constitute an independent event).
[Two overarching design principles]
1) Stream Factorization: separates persistent information from time-varying information, reducing semantic redundancy and supporting local updates;
2) Relational Grounding: identity linking (centralized entity bank + ID references) + temporal linking (shared anchors + intra-description timestamps) re-weave the isolated streams back into a coherent script.
[Core claim: editability] The fatal flaw of monolithic captions is that local edits inevitably trigger global rewrites — to change a single camera move or a single sound effect, the entire passage must be rewritten to preserve narrative coherence. MTSS makes dependency relationships traceable and supports precise local updates, which is its structural advantage over long dense captions and the basis of its "scalability" argument.
[Core claim: learnability] MTSS significantly narrows the performance gap between small and large models (Figure 1 explicitly positions this as its second major selling point); the paper explains this as follows: monolithic text requires the model to untangle densely interwoven relationships on its own, whereas MTSS has already pre-disambiguated WHO, WHERE, and WHEN, so downstream reasoning modules can focus on logical inference rather than identity resolution and temporal disambiguation.

### Joint audio-video caption structure (whether visual + auditory tracks are covered simultaneously, whether it is factorized into independent fields, e.g., LTX-2's full soundscape description, Script-a-Video's factorized streams, Foley-Omni's three fields)

This field is Script-a-Video's most valuable contribution, and its approach sits at the deepest end of the structuring spectrum among comparable works:
[Positioning comparison] Along the axis of "how audio-visual information is organized": LTX-2's single full-soundscape description (fused) → Foley-Omni's three parallel fields (shallow factorization) → UniVerse-1's three independent fields (factorized but unrelated) → MOVA's factorize-then-LLM-fuse-into-a-single-paragraph (factorize then fuse) → MTSS's four-stream decomposition + explicit relational graph (factorize then explicitly reconnect). MTSS is the only approach that, after factorization, reconnects the streams using "machine-readable reference relationships" rather than "natural-language narration."
[Whether visual and auditory are both covered simultaneously] Yes. The Shot stream carries the visual track, the Event stream carries the auditory track (dialogue/sound effects/music), the Global stream's global_audio carries ambient background noise and background music, and the Reference stream is shared cross-modally (referenced both by Shot's references_in_shot and by Event's speaker).
[Whether it is factorized into independent fields] Yes, and into four streams rather than the more common two or three. The key difference is the extra Reference stream — this stream does not belong to any single modality but instead exists as a cross-modal shared identity anchor, a structural innovation that sets MTSS apart from other factorized approaches.
[Two mechanisms of cross-modal connection]
1) Identity linking: the Event's speaker field and the Shot's references_in_shot array simultaneously point to the same ID within the Reference stream. This means that "who is speaking" is explicitly expressed at the data-structure level as "the audio event's speaker pointer" and "the visual shot's visible-subject pointer" pointing to the same entity — the binding between speaker and on-screen person is structural and machine-verifiable, rather than an implicit correspondence relying on natural-language description. This is the most direct means of resolving audio-video identity mismatch.
2) Temporal linking: the Shot's active_events field links a shot to its concurrent audio events; both sides share the same global timeline, and Shot's and Event's respective time_ranges can be directly interval-computed; finer correspondence is handled by both sides' intra-description / micro-level timestamps.
[The admission filter for audio events is itself the alignment guarantee] The strict audio-visual coupling principle (sound effects must be produced by a visible subject) essentially dissolves "audio-visual mismatch" samples already at the annotation stage — mismatched sounds never become an Event, but are instead demoted to global_audio. This is a clever trick of replacing a filter with schema design: no additional audio-visual consistency discriminator model is needed — the annotation rule itself accomplishes the routing.
[Quantified direct downstream generation benefit] When a monolithic prompt is swapped for an MTSS script while the architecture is entirely unchanged (LTX-2-AV → LTX-2-AV-MTSS): human-rated identity consistency 1.22 → 1.77 (+45%), human-rated audio-video alignment 1.18 → 1.85 (+56%), human-rated multi-shot controllability 1.00 → 1.71 (+71%); WER drops from 0.84 to 0.13; Shot Boundary Deviation 3.79 → 3.27 frames. This "same architecture, same training paradigm, the only variable is prompt structure" comparison is this work's most persuasive evidence — the paper explicitly states that the two baselines share exactly the same architecture and training paradigm, so any performance gap between them can be directly and entirely attributed to the MTSS paradigm itself.
[Limitations] The schema sets no explicit audio quality/loudness/acoustic-environment fields (such as recording environment, reverberation, volume), nor any language/accent structured labels; timbre description is only implicit in the Event's free-text description.

### Dialogue transcription and speaker-attribute annotation (ASR transcription + speaker identity/language/accent/emotion) ⚠️

Dialogue transcription has a clear structured carrier in MTSS, but speaker-attribute annotation is comparatively thin:
[Transcription] Events of type dialogue in the Event stream carry a "line" field recording verbatim text. The paper does not state whether these lines are produced by ASR transcription or dictated directly from the audio by Gemini-2.5-Pro — given that the entire annotation set is produced by Gemini-2.5-Pro in one pass, the latter seems more likely, i.e., no independent Whisper-type ASR step. [Uncertain]
[Speaker identity] Implemented through the "speaker" field, whose value is an entity ID from the Reference stream (e.g., PERSON_1). This is a relational binding rather than an attribute description — speaker identity points directly to an entry in the entity bank that carries a complete appearance anchor (clothing/accessories/hairstyle), so "voice" and "appearance" are hard-bound at the data-structure level. This design is especially critical for training an audio-video joint generation model to identify "who is speaking."
[Speaker attributes] No independent structured field is set. Gender, age, timbre, language, accent, and speaking rate have no corresponding field. The closest is the Event's "description" field, which the paper describes as capturing "nuanced semantics like emotional shifts or vocal techniques" — i.e., emotion and vocal delivery are recorded as free text, not as enumerated labels, and cannot be directly used for mixture statistics or conditional control.
[Multi-speaker handling] Naturally supported through the concurrent-event-splitting mechanism: multiple simultaneous sound sources are split into parallel event entries, each carrying an independent speaker, time_range, and line, so multi-person dialogue scenes can be fully expressed without being blended together. This is a clear advantage over approaches that use "a single passage describing the entire soundscape."
[Transcription-accuracy validation on the generation side] WER is computed by transcribing the generated audio with Whisper-large-v3 and comparing it with the ground-truth text using jiwer (CJK text is tokenized with jieba). Results: LTX-2-AV baseline single-shot 1.64 / multi-shot 0.84 → LTX-2-AV-MTSS single-shot 0.78 / multi-shot 0.13 → Ours(Full) multi-shot 0.19. The paper states that the explicit audio-event descriptions provided by the Event stream turn speech and sound-effect generation from "near-random" into "semantically accurate."
[Audio-quality metric] Audio Quality is evaluated with UTMOS (a lightweight speech-quality MOS predictor): baseline 4.12-4.18, MTSS version 4.60, full solution 4.68.

### Geometric and structured annotation (camera parameters, depth, 3D point tracks, action labels, explicit state)

MTSS contains no numerical geometric annotation of any kind: no camera intrinsics/extrinsics, no depth maps, no 3D point tracks, no optical-flow fields, no skeleton/pose keypoints, no bounding-box coordinates, no segmentation masks.
However, this work goes quite deep in the "non-numerical structured annotation" dimension, which is the substantive content of this entry:
[Semanticized annotation of shot language] The Shot stream's camera field records camera movements, perspectives, and scales (shot sizes) in professional film language. This structures camera information in "director's language" rather than "parameter matrices" — sacrificing precision in exchange for controllability that can be directly understood and generated by an LLM. For a generation model with a text-based conditioning interface, this is a more practical form than numerical camera parameters.
[Entity-level structured appearance anchoring] The Reference stream's appearance_anchor expands into fine-grained attribute slots — clothing, accessories, hairstyle — for person-category entities. This is an "attribute-slot representation replacing pixel-level annotation" for identity.
[Structuring of the timeline] This is MTSS's most rigorous structuring dimension — a three-tier temporal annotation: shot-level time_range, event-level time_range, and intra-description / micro-level timestamps embedded within the description text. All three tiers share the same global timeline, forming an explicit temporal structure that supports interval computation. The paper states its goal as achieving sub-frame audio-visual coordination.
[Relational-graph structure] The three types of references — references_in_shot (a many-to-many edge from shot to entities), active_events (an edge from shot to events), and speaker (an edge from event to entities) — together constitute an explicit tripartite entity-shot-event graph. This is the part of the schema closest to a "knowledge graph," and the substantive referent of the term "Relational Grounding."
[Action annotation] Present as free text within visual_description, which is required to narrate the core action "objectively and chronologically" and is anchored by intra-description timestamps, but there is no action-category label system.
[Explicit state] The Reference stream's semantic_description records an entity's overall state, a weakened form of explicit state annotation.

### Synthetic data construction (controlled perturbation/edited construction of training pairs, e.g., InstructAV2AV)

The paper constructs no synthetic data of any kind: no controlled-perturbation-constructed training pairs, no edit-based data augmentation (no InstructAV2AV-style audio-video editing instruction pairs), no TTS-synthesized speech, no sound-effect overlay/mixing, and no self-distillation from a generative model's own output. All training data is genuinely collected film/TV or lifestyle audio-video content.
[The only "synthetic" component is annotation, not content] All MTSS annotations for the 500K dataset are automatically generated by Gemini-2.5-Pro — synthetic annotation. This constitutes a standard model-distillation chain: closed-source strong-model annotation → open-source model SFT → a cheaper dedicated annotator.
[The design closest to data construction] MTSS itself can be viewed as a "representation-layer data reconstruction" — the same video, represented under a monolithic caption versus an MTSS script, forms a naturally paired contrastive data pair; the paper uses exactly this pairing (LTX-2-AV vs. LTX-2-AV-MTSS, same architecture, same training, the only variable being prompt structure) to make its core argument. But this is an evaluation design, not data synthesis.
[Potential synthetic value of editability, not exercised by the paper] MTSS's local editability is in principle very well suited to constructing controlled-perturbation pairs — changing only the description of one sound effect in the Event stream, or only the camera field of one shot, would yield a semantically controlled-difference training pair without rewriting the entire text. The paper raises this structural advantage but does not use it for synthetic-data construction — an as-yet-unexplored application direction for this schema.

### Degree of human involvement (manual annotation, manual quality control, model pre-screening + manual review) ⚠️

The degree of human involvement is extremely low, and unevenly distributed:
[Annotation step: zero manual involvement] All MTSS annotations for the 500K clips are automatically generated by Gemini-2.5-Pro; the paper mentions no manual annotation, manual spot-checking, manual review, or annotation-guideline training step, and reports no inter-annotator-agreement metric. For a structured schema that depends on cross-reference integrity, the absence of a manual or programmatic validation step is a clear gap. [Uncertain]
[Evaluation step: a fairly substantial manual investment] On the generation side, 20 professional raters were organized to manually score 225 generated videos (125 single-shot + 100 multi-shot), covering five dimensions: Text Alignment, Visual Quality, Multi-Shot-Consistency, Identity Consistency, and Audio-Video Synchronization, all on a 1-3 point scale.
[Methodological value of human evaluation] The paper explicitly points out that human evaluation reveals the deceptiveness of automated metrics — a valuable observation in this work:
1) On the Intra-Shot SC metric, the baseline's 0.87 is higher than the full solution's 0.59, but this is an illusion arising from the baseline generating near-static content;
2) On A-V Sync (SyncNet), the baseline's 6.86 "leads" the full solution's 9.72, but the baseline's human-rated A-V score is only 1.18 — the paper explains this as an "information-sparsity artifact": it is trivially easy to make flat ambient noise "synchronize" with the picture, whereas after generating complex dialogue, the SyncNet score first rises (LTX-2-AV-MTSS reaches 13.86, with larger deviation being worse), and the full solution then pulls it back to 9.72 while achieving a human rating of 2.26. This analysis carries a direct warning for any team relying on SyncNet for data filtering: SyncNet scores carry a systematic bias against speech-free/flat-audio samples.
3) Reference ID Similarity drops to about 0.22 in multi-shot scenes (due to drastic viewpoint and lighting changes), but the human-rated Cons. score remains stable above 2.40 — automatic ID similarity is likewise unreliable in cross-shot scenarios.
[Annotation prompt] Not made public, and it is not stated whether there was human involvement in iterating on the prompt.

## Audio-Video Alignment

### Audio-video synchronization detection method (lip sync, event alignment) ⚠️

The data side and the evaluation side must be distinguished, as they differ significantly:
[Data side (annotation stage): schema rules replace a detection model] This is the most notable idea in this work. MTSS does not use a synchronization-detection model such as SyncNet to filter data; instead, it directly encodes the audio-video correspondence into the annotation rules:
1) The strict audio-visual coupling admission principle — the Event stream only admits audio events with "a direct visual correspondent or thematic relevance," and sound effects must be produced by a visible subject in the frame. Sounds that fail to meet this criterion are not discarded but demoted to global_audio. This accomplishes, already at the annotation level, the separation of "visually grounded sound" from "visually ungrounded sound."
2) Structural speaker binding — the Event's speaker field points to a Reference ID, the same entity bank pointed to by the Shot's references_in_shot, making "whether the speaker is on screen" a question directly decidable through reference relationships, without needing face detection + lip-motion analysis.
3) A three-tier timestamp system — shot-level time_range, event-level time_range, and intra-description micro-level timestamps share a global timeline, so event alignment is directly expressed by timestamps; the paper states the goal as achieving sub-frame audio-visual coordination and "surgical synchronization."
That is: alignment is not "a filtering condition detected after the fact," but "annotation content written directly into the structure." This is a fundamentally different methodology from routes such as UniVerse-1 (hard SyncNet conf>2.0 threshold filtering) or MOVA that use a discriminator model as a gate.
[Evaluation side: SyncNet is used] The lip-sync accuracy of generated results is evaluated with SyncNet, whose model outputs a synchronization confidence score and a frame-level time offset. In addition, Shot Boundary Deviation is used to measure the frame-level absolute deviation between generated shot boundaries and the boundaries specified by the script.
[The paper's important finding on sync-metric reliability] See the human_in_loop and sync_metric_and_threshold fields for details — SyncNet scores give an inflated appearance of synchronization when audio information is sparse (flat ambient noise), a trap warning of general reference value.
[Whether the data side has additional sync filtering] The paper does not disclose whether the 500K dataset or the four generation-side datasets underwent any audio-visual synchronization detection filtering — part of the overall absence of a disclosed cleaning pipeline. [Uncertain]

### Specific sync-detection metrics and thresholds (SyncNet/Synchformer/LSE/in-house, threshold values, e.g., MOVA's LSE-D≤9.5 and LSE-C≥4.5, SkyReels-V4's SyncNet |offset|≤3∧conf>1.5) ⚠️

[Data-filtering threshold: none] The paper gives no synchronization-metric threshold used for data filtering — no SyncNet confidence gate, no AV-align score gate, no lip-motion-detection gate. This contrasts with works such as UniVerse-1 (SyncNet conf > 2.0) and UniTalking (SyncNet conf > 0.9), which explicitly give numerical gates. This work's alignment guarantee comes from schema rules, not threshold filtering. [Uncertain]
[Evaluation metric: SyncNet (chung2016out, the original "Out of Time" version)] Outputs a synchronization confidence score and a frame-level time offset. Measured values (multi-shot scenario):
- LTX-2-AV (monolithic-prompt baseline): 6.86
- LTX-2-AV-MTSS (prompt swap only): 13.86
- Ours w/o ID: 9.16
- Ours(Full): 9.72
[Key warning on interpreting the numbers] The paper explicitly states that this automated metric can be deceptive ("the automated A-V Sync metric can be deceptive"): the baseline's 6.86 appears optimal, but its human-rated A-V score is only 1.18 (the lowest tier on a 1-3 scale), because the baseline generates flat ambient noise — making near-static audio envelope "synchronize" with the picture is trivial (an information-sparsity artifact). After switching to MTSS, the model starts attempting to generate complex dialogue, and the SyncNet score first rises to 13.86 (deviation widens); the full solution then narrows the temporal gap to 9.72 through architectural improvements while reaching a human-rated A-V score of 2.26 (the best in the whole table).
This finding has direct practical implications for the data side: if SyncNet score were used as a data-filtering gate, it would systematically favor speech-free/weak-audio-content samples, filtering out information-rich dialogue samples instead. Any pipeline adopting SyncNet threshold filtering should first stratify by audio activity level and then set thresholds per stratum.
[Other related metrics and values]
- Shot Boundary Deviation (frames): LTX-2-AV 3.79 → LTX-2-AV-MTSS 3.27 → Ours w/o AV 1.28 → Ours w/o ID 0.38 → Ours(Full) 1.36;
- WER: LTX-2-AV multi-shot 0.84 → LTX-2-AV-MTSS 0.13 → Ours(Full) 0.19 (single-shot baseline 1.64 → 0.78 → 0.23);
- Audio Quality (UTMOS): 4.12-4.18 → 4.60-4.79 → 4.68;
- Reference ID Similarity (ArcFace cosine similarity): single-shot Ours w/o MS 0.62; multi-shot: Ours w/o ID no value, Ours(Full) 0.22, Ours w/o AV 0.20;
- Intra-Shot Subject Consistency (frame-to-frame average cosine similarity of DINOv2 [CLS] features): LTX-2-AV 0.87 (inflated) → LTX-2-AV-MTSS 0.66 → Ours(Full) 0.59.

### Separation of temporal sync vs. semantic sync (temporal alignment and content-semantic matching as two independent filtering conditions)

MTSS structurally separates temporal synchronization and semantic synchronization completely, a direct manifestation of the dual-linkage design of Relational Grounding:
[Temporal-alignment channel: temporal links] Carried by three tiers of timestamps — shot-level time_range, event-level time_range, and intra-description / micro-level timestamps embedded in the description, plus shared anchors. This channel answers only "when," with all information being numerical intervals that support interval computation.
[Semantic-alignment channel: identity links] Carried by the centralized Reference entity bank — Shot's references_in_shot and Event's speaker both point to the same ID space. This channel answers only "who/what," with all information being symbolic references.
[A third, implicit channel: causal alignment] The Event stream's admission principle "sound effects must be produced by a visible subject in the frame" in fact encodes the causal/source relationship between sound and visuals — neither purely temporal nor purely identity-based, but "which thing in the frame produced this sound." Shot's active_events field, associating a shot with concurrent events, likewise carries this meaning.
[The value of separating the three channels] The paper's core argument rests precisely on this: a monolithic caption blends WHO, WHERE, and WHEN into the same passage of text, forcing the downstream model to perform identity resolution and temporal disambiguation before it can reason; MTSS pre-disambiguates all three, letting the model focus directly on logical inference. This explains why MTSS's gains on reasoning-type benchmarks (Daily-Omni, WorldSense) — Qwen3-Omni +127% — are far larger than its gains on description-type benchmarks.
[At the filtering-condition level] The paper does not use the two as two independent data-filtering conditions (because no filtering conditions are disclosed overall); their separation is manifested at the representation layer rather than the filtering layer. This is a path different from the traditional practice of using "temporal sync and semantic matching as two independent filtering gates" — MTSS converts these two dimensions from "filtering criteria" into "annotation fields."

### Audio quality filtering (SNR, silence detection and silence-ratio thresholds, no-audio-track exclusion, off-screen source exclusion, background-music separation) ⚠️

The paper discloses no data-side audio quality filtering of any kind: no SNR gate, no silence detection or silence-ratio threshold, no "exclude if no audio track" rule, no background-music separation (e.g., Demucs-type source separation), and no loudness/clipping/distortion detection. [Uncertain]
[Equivalent mechanism at the schema level] MTSS partially substitutes two annotation rules for traditional audio filtering:
1) Handling of off-screen sound sources — the strict audio-visual coupling principle prevents sounds without visual correspondence from becoming an Event, in effect achieving the goal of "off-screen source exclusion," but through demotion rather than removal: these sounds are placed into the Global stream's global_audio field and retained. This is a "lose no information, only demote it" design, more data-economical than outright removal.
2) Handling of background music — no signal-level source separation is performed; instead, semantic-level layering is done: music that constitutes an independent narrative event goes into the Event stream (type=music), while music serving only as atmospheric bedding goes into the Global stream's global_audio. This is an approach of substituting annotation tiers for audio signal processing.
[Audio-quality metric on the evaluation side] UTMOS (a lightweight speech-quality MOS predictor) is used to evaluate the speech quality of generated audio, not for data filtering. Values are given in the sync_metric_and_threshold field.
[Implicit audio-track requirement] The entire schema is heavily dependent on a real audio track (both the Event stream and global_audio need to be extracted from the original audio), so the training data necessarily comes with a native audio track — but the paper does not write this into an explicit filtering rule.

### Classification and differentiated handling of speech/sound-effects/music ⚠️

The classification and differentiated handling of audio types is one of the most clearly defined parts of MTSS, using a "three event types + one global tier" four-layer division:
[dialogue]
- Belongs to: Event stream;
- Dedicated fields: speaker (relationally bound to a Reference ID, explicitly stating "who is speaking"), line (verbatim dialogue text);
- The description field additionally carries paralinguistic information such as emotional shifts and vocal techniques;
- The richest field set among the three types, reflecting its central role in audio-video generation (both lip sync and identity binding depend on it).
[sfx (sound effect)]
- Belongs to: Event stream;
- Dedicated admission constraint: must be produced by a visible subject in the frame ("sound effects must be generated by a visible subject") — the only one of the three types with a hard visual-grounding requirement, directly serving foley-type generation's need for "sound-action causal relationships";
- No speaker or line field, only description.
[music]
- Belongs to: Event stream (when the music constitutes an independent narrative event);
- Forms a layered distinction against background music in global_audio: the former has a clear start/end and participates in the narrative (e.g., someone visibly performing on screen, or scoring that enters as a plot point), while the latter is film-wide underlying atmosphere music that does not constitute an independent event.
[global_audio (global audio, the fourth layer)]
- Belongs to: Global stream;
- Carries: ambient sounds and background music that do not qualify as independent events, as well as irrelevant background noise that has been filtered down into it;
- Role: a fallback container, ensuring soundscape information is not lost while not polluting the signal-to-noise ratio of the Event stream.
[Concurrent sound sources] Split into parallel event entries, each independently carrying type, time_range, and content block, with no mixed description. This allows a scene such as "a person speaking while footsteps sound nearby" to be fully and separately controllably expressed.
[Differentiated validation on the generation side] The paper evaluates speech-related capability separately with three metrics — WER (dialogue-content accuracy), UTMOS (speech quality), SyncNet (lip sync) — and evaluates sound effects qualitatively through RMS envelope analysis: Figures 7/8 show the baseline producing a flat, ambient-noise-type envelope, whereas the MTSS solution exhibits the rhythmic undulation of natural speech and the periodic impact of action-driven events (such as the periodic impact of footsteps). The paper explicitly states that the Event stream turns speech and sound-effect generation "from near-random to semantically accurate."
[Data-side mixture ratio] The proportion of the three audio types in the training data is not disclosed, nor is any adjustment of sampling weights by category mentioned. [Uncertain]

## Training Coordination

### Multi-stage training curriculum and data-curriculum scheduling (basis for stage division: resolution/duration/quality score/modality; low-res→high-res, image→video, short→long) ⚠️

The training curricula for the captioning side and the generation side differ enormously:
[Captioning side: a single stage, no curriculum] Supervised fine-tuning is performed once on the open-source Qwen3-Omni-Instruct using the 500K MTSS data, directly yielding Qwen3-Omni-MTSS-FT. There is no multi-stage design and no curriculum layered by resolution/duration/quality score. The paper uses the phrasing "we simply performed supervised fine-tuning" to emphasize its simplicity — this is itself part of the argument: no elaborate training tricks are needed; the change in data representation format alone delivers a large gain.
[Generation side: a four-stage progressive curriculum, divided by "capability dimension" rather than "resolution/duration"] This is the most distinctive aspect of this work's training-curriculum design — the stage division is not based on the traditional low-res→high-res, short→long, image→video progression, but on the capabilities corresponding to each of MTSS's four streams:
Stage 1 · ID Customization: 400K identity-centric dataset, 3 epochs, 128 GPUs. The goal is to anchor persistent character and environment features, corresponding to the Reference stream's capability.
Stage 2 · Multi-shot Control: 250K multi-shot sequences, 1.5 epochs, 32 GPUs. The goal is to fine-tune the shot-aware cross-attention mechanism, corresponding to the Shot stream's capability. Note this stage has the fewest GPUs (32) and fewest epochs (1.5), making it the lightest-investment stage of the four.
Stage 3 · Audio-Visual Synergy: 870K cinematic pairs, 3 epochs, a 400-GPU cluster. The goal is explicitly the Gemma Connectors' sub-frame alignment, corresponding to the Event stream's capability. This stage has the largest data volume (870K) and largest compute investment (400 GPUs), indicating that audio-video alignment is regarded as the hardest capability.
Stage 4 · Joint Fine-tuning: 15K steps, 64 GPUs, using an interleaved mixture of 60K high-fidelity cinematic alignment pairs + 250K multi-shot sequences. The goal is to simultaneously coordinate identity stability, narrative coherence, and temporal synchronization within a single generation pass, ensuring multiple constraints do not mutually interfere ("without mutual interference").
[Curriculum-logic summary] The first three stages are "staged optimization for individual tasks"; the fourth stage is "joint coordination of multiple capabilities." This is a training design isomorphic to MTSS's own stream-decomposition idea — first master each stream's corresponding capability separately, then re-coordinate them, echoing the Stream Factorization + Relational Grounding representation design.
[Undisclosed] The learning rate, batch size, optimizer, GPU model, resolution/duration configuration, total training duration, and total GPU-hours for each stage are all undisclosed. [Uncertain]

### Data mixture changes across training stages (pretraining/annealing/SFT high-quality subset) ⚠️

The generation side's data mixture clearly changes across stages, but only counts are given, not ratio statistics:
[Stage 1] Purely the 400K identity-centric single source, no mixing.
[Stage 2] Purely the 250K multi-shot sequences single source, no mixing.
[Stage 3] Purely the 870K cinematic pairs single source, no mixing.
That is, the first three stages are all single-source dedicated training with no mixing — a relatively "clean" curriculum design where each stage's learning signal is highly pure, avoiding multi-objective dilution.
[Stage 4 (the only mixing stage)] 60K high-fidelity cinematic alignment pairs + 250K multi-shot sequences, mixed in an interleaved manner. Introducing the 60K high-fidelity dataset is a classic "annealing" idea — at the final stage, a much smaller (60K vs. the earlier 870K, about 1/14.5) but higher-quality ("high-fidelity") subset is introduced to refine alignment quality. The 250K multi-shot sequences are reused from Stage 2, used to prevent forgetting of the multi-shot capability during subsequent training.
[Mixing ratio] The nominal count ratio of 60K : 250K is about 1:4.2, but the specific interleaved sampling weights, and whether any capability-based reweighting was applied, are not stated. [Uncertain]
[Captioning side] No stage division, hence no mixture change.
[A notable scale drop worth noting] Stage 3 uses 870K data + 400 GPUs training for 3 epochs, while Stage 4 uses only 15K steps + 64 GPUs — the final joint-coordination stage's compute investment is only about 1/6 of Stage 3's, with very few steps, indicating its role is a lightweight "capability-fusion calibration" rather than another round of large-scale learning.

### Post-training data (SFT curated-set scale and filtering criteria, number of preference pairs and annotation method, reward-model training data) ⚠️

The paper does not involve a standard-sense post-training step:
[SFT] The captioning side's 500K MTSS fine-tuning is itself the SFT (and the only training stage); the filtering criterion is only the qualitative description "high-quality" plus the three domains (film/television/lifestyle), with no specific filtering metric. All four generation-side stages are fine-tuning; there is no "pretraining → SFT" layering.
[Preference alignment] No preference-optimization method is used at all — no DPO, no RLHF, no GRPO, no preference-pair data, no reward model. The paper's Related Work section mentions a batch of works using RLVR/GRPO for cross-modal reasoning enhancement — R1-Omni, EchoInk-R1, Omni-R1, HumanOmniV2, OmniVideo-R1 — indicating the authors are aware of this route, but this work explicitly does not take the reinforcement-learning path, arguing instead that the gains come from the representation paradigm itself.
[Reward-model training data] Not applicable.
[The component closest to a "curated set"] The 60K high-fidelity cinematic alignment pairs used in Stage 4 can be viewed as a high-quality curated subset (about 1/14.5 the scale of Stage 3), playing a role similar to annealing data, but the paper does not state its filtering criteria, nor does it call it an SFT set. [Uncertain]
[The 20 professional raters' human-evaluation data] Used only for final effectiveness evaluation, not recycled as preference pairs or reward-model training data.

### Data-processing infrastructure and throughput (NeMo Curator/Data-Juicer/in-house, GPU speedup, processing scale, cost) ⚠️

The paper does not touch on data-processing infrastructure at all: no mention of NeMo Curator, Data-Juicer, or any in-house data-processing framework; no GPU speedup ratio, processing throughput (clips/hour or hours/day), processing time, or cost estimate is given. [Uncertain]
[An order-of-magnitude estimate of compute cost that can be inferred] The only thing that can be roughly estimated is annotation cost — all 500K video clips underwent per-clip inference by Gemini-2.5-Pro, with output being long structured JSON containing multiple fields across four streams, i.e., a long-output multimodal API call. Priced at commercial API rates, the cost magnitude of 500,000 such calls is not negligible, but the paper discloses no fee figure, call latency, concurrency scale, or whether an internal deployment was used. This also indirectly explains why Qwen3-Omni-MTSS-FT was distilled — using an open-source model to replace Gemini for large-scale annotation can significantly lower marginal cost, which is this work's actual value in engineering economics.
[Compute on the training side (disclosed)] Stage 1 uses 128 GPUs, Stage 2 uses 32 GPUs, Stage 3 uses a 400-GPU cluster, Stage 4 uses 64 GPUs. GPU model, per-stage wall-clock time, and total GPU-hours are all not given. The scale of the 400-GPU cluster indicates a major-company-level engineering investment.
[Data storage and indexing] MTSS, being JSON-style structured annotation, in principle naturally supports field-based retrieval (e.g., "find all clips with more than 3 shots and containing a dialogue event"), of real practical value for the dataset's retrievability and reusability, but the paper does not discuss the storage scheme or retrieval infrastructure.

## Effectiveness Comparison

### Quantified impact of data-strategy ablations (distinguish: filtering-strictness ablation / caption-density-style ablation / data-mixture ablation, and corresponding evaluation metrics)

This work's ablation design concentrates on two dimensions — "representation format" and "architectural module" — belonging to caption-structure-style ablation, rather than filtering-strictness or data-mixture ablation:
[Dimension 1: caption representation-format ablation (this work's most central piece of evidence)]
The design is extremely clean: LTX-2-AV (monolithic prompt) vs. LTX-2-AV-MTSS (MTSS structured script), with completely identical architecture and completely identical training paradigm — the only variable is the organizational structure of the conditioning text. The paper explicitly states "any performance gap between them is directly and entirely attributable to the MTSS paradigm." Multi-shot scenario results:
- Human-rated identity consistency (Cons.): 1.22 → 1.77 (+45%)
- Human-rated audio-video alignment (A-V): 1.18 → 1.85 (+56%)
- Human-rated multi-shot controllability (MSC): 1.00 → 1.71 (+71%)
- WER: 0.84 → 0.13 (an 85% reduction)
- Shot Boundary Deviation: 3.79 → 3.27 frames
- Audio Quality (UTMOS): 4.18 → 4.60
This set of data is strong evidence that "the caption structure itself is a performance lever," with a modification cost of zero (no architecture changes, no added data).
[Dimension 2: cross-model format ablation on the caption side] Comparing "native monolithic output" against "MTSS structured output" (zero-shot prompting) across five different captioning models, all showing positive gains:
- Gemini-2.5-Pro total error rate 0.3959 → 0.2511 (Miss 0.1902→0.1285, Incorrect 0.0848→0.0644, Hallucination 0.1209→0.0582, with hallucination showing the largest reduction, 52%); UGC-VideoCap composite 93.97 → 95.51;
- Qwen3.5-Omni-Flash total error rate 0.5217 → 0.3655; UGC-VideoCap 76.07 → 83.28 (with the Audio Avg going from 65.92 to a substantial 82.84);
- Qwen3-Omni total error rate 0.5853 → 0.5156; UGC-VideoCap 62.80 → 71.54 (Audio Avg 52.81 → 68.79).
Particularly notable: gains on the audio dimension are generally larger than on the visual dimension, indicating that the Event stream's explicit audio-event decomposition provides the greatest improvement in capturing auditory information. At the same time, Qwen3-Omni's Hallucination under zero-shot MTSS actually rose slightly, from 0.1104 to 0.1355 — the only negative entry in the whole table, indicating that a weaker model forced to fill in structured fields may be induced to fabricate.
[Dimension 3: SFT ablation] Qwen3-Omni-MTSS-FT (fine-tuned on 500K MTSS data) reaches a total error rate of 0.3913, UGC-VideoCap 85.11, Daily-Omni 0.5945, WorldSense 0.3875 — all four figures substantially exceed the zero-shot MTSS version, with the total error rate already slightly better than Gemini-2.5-Pro's native output (0.3959). This proves the learnability of MTSS data.
[Dimension 4: architectural-module ablation (not data-side)] Comparing four configurations — Ours(Full) / w/o ID (identity customization removed) / w/o MS (shot-aware attention removed) / w/o AV (audio input removed). Key finding: w/o ID achieves the best Shot Bnd. Dev. (0.38 frames) while Full is 1.36 frames, revealing a trade-off between identity-preservation features and temporal shot precision at the VLM-encoder-to-DiT interface.
[Missing ablations] No filtering-strictness ablation (because the filtering pipeline is not disclosed), no data-mixture ablation, no data-scale ablation (no scaling experiment on the 500K figure was done), and no per-stream ablation (the effect of removing the Reference stream or the Global stream was not verified). This last item is a somewhat regrettable gap — in the Summary the paper gives qualitative attributions (the Shot stream provides a temporal-segmentation prior, the Reference stream provides an identity anchor, the Event stream provides audio-event descriptions), but there is no quantitative support from a stream-by-stream breakdown.

### Evidence for quality vs. quantity (cases where small, refined data outperforms large, mixed data) ⚠️

What this work provides is not the traditional "small data beats big data" evidence, but a more fundamental variant — "the same data, reorganized into a different structure, delivers a large performance gain":
[Core evidence: zero-cost format gain] LTX-2-AV and LTX-2-AV-MTSS use exactly the same architecture, exactly the same training data, and exactly the same training paradigm; the only difference is the structural organization of the conditioning text, yet this yields human-rated identity consistency +45%, audio-video alignment +56%, multi-shot controllability +71%, and an 85% reduction in WER. This shows that, with data volume and compute held fixed, the "representation quality" of the annotation is itself an independent and powerfully effective performance dimension. The practical implication for data-side teams is that, before considering expanding data scale, examining the structuring of existing annotations may be a higher-return investment.
[Second layer of evidence: structuring improves small models' data efficiency] The paper positions Figure 1's second major selling point as "Learnability" — MTSS significantly narrows the gap between small and large models. Qwen3-Omni improves from 0.1806 to 0.4117 on Daily-Omni (+127%), a far larger gain than Gemini-2.5-Pro's 0.6825→0.7568 (+11%). That is, structured captions benefit weaker-capability models more, the mechanism being that the burden of "untangling entangled relationships" is shifted from the model onto the data representation. This is in essence "trading annotation structure for model capacity," of direct value to compute-constrained teams.
[Third layer of evidence: the efficiency of the 500K distilled dataset] Using only 500K Gemini-annotated data for a single simple SFT run drops Qwen3-Omni's Video-SALMONN-2 total error rate from 0.5853 to 0.3913, surpassing the teacher model Gemini-2.5-Pro's native performance (0.3959). The student surpassing the teacher on a format-specific task indicates a very high return rate for format-specialized data.
[Fourth layer of evidence: an annealing-style high-quality small set] The final generation-side stage uses 60K "high-fidelity" data (only 1/14.5 the scale of Stage 3's 870K) + 15K steps to complete the final coordination — a classic small-scale, high-quality refinement, but the paper does not run a with/without-this-stage ablation. [Uncertain]
[Limitation] The paper runs no data-scale scaling experiment (e.g., 100K vs. 500K comparison), so it cannot answer the practical question of "how much MTSS data is enough."

### Alignment relationship between training-data domain distribution and the evaluation-benchmark category system (e.g., VABench's seven major categories)

This work does not build its own evaluation benchmark, so there is no explicit design of "aligning training-data domain distribution with a self-built benchmark's category system." However, several alignment relationships worth recording do exist:
[Alignment between training-data domain and the internal evaluation set] The training side's 500K covers three domains — film / television / lifestyle; the internal evaluation set of 225 samples covers four categories — movie and TV drama clips, short-form videos, indoor scenes, outdoor scenes. The first two categories (film/TV, short-form video) correspond directly to the training domains, while the latter two (indoor/outdoor scenes) are an orthogonal scene-dimension supplement. Overall this is an in-distribution evaluation, with no explicit out-of-distribution generalization test.
[Alignment between schema design and evaluation dimensions (this work's clearest alignment relationship)] The MTSS four streams map almost one-to-one onto the five human-evaluation dimensions on the generation side:
- Reference stream ↔ Identity Consistency (Cons.) + Reference ID Similarity (ArcFace)
- Shot stream ↔ Multi-Shot-Consistency (MSC) + Shot Boundary Deviation + Intra-Shot Subject Consistency
- Event stream ↔ Audio-Video Synchronization (A-V) + WER + UTMOS + SyncNet
- Global stream ↔ the Scene and Style sub-dimensions of Semantic Following
This design, in which "every stream of the annotation schema has a corresponding evaluation metric," lets ablation analysis be localized to a specific structural component — a strength of this work's experimental design. The four sub-dimensions of Semantic Following (Subject / Action / Scene / Style) likewise correspond closely to MTSS's information organization — Subject corresponds to the Reference stream, Action to the Shot stream's visual_description, and Scene and Style to the Global stream.
[Selection logic of the external benchmarks] The four external benchmarks have a clear division of labor: Video-SALMONN-2 testset and UGC-VideoCap test "description fidelity" (the former breaking down into Miss/Incorrect/Hallucination error types, the latter into Visual/Audio/Details dimensions); Daily-Omni and WorldSense test "reasoning fidelity." UGC-VideoCap's Visual/Audio/Details three-way breakdown conveniently allows observation of MTSS's differentiated gains on the visual versus auditory sides (the results show the audio-side gain is far larger than the visual side) — a good alignment between benchmark selection and the method's claims.
[Parts with no corresponding relationship] No alignment was made with the category system of audio-video-dedicated benchmarks such as VABench, and no distribution of the training data under any standard category system is reported.

## Uncertain Fields

The research information for the following fields is partly uncertain (⚠️ marked sources):

- organization
- release_date
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
- quality_over_quantity_evidence
