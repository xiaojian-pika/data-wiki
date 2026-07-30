# CineDance / CineDance-1M (paper title: CineDance: Towards Next-Generation Multi-Shot Long-Form Cinematic Audio-Video Generation). This work comprises three deliverables: the CineDance-1M dataset (1 million multi-shot, long-form audio-video narrative sequences), the CineBench evaluation benchmark (1,000 test samples + a six-dimensional metric system), and the CineDance generation model (an open-source baseline adapted from LTX-2.3).

> Topic: Data processing for video generation models (including joint audio-video generation): data cleaning pipeline, data distribution, annotation approach

[← Back to home](../index.md)

**Contents**: [Basic Information](#basic-information) · [Data Scale and Distribution](#data-scale-and-distribution) · [Cleaning Pipeline](#cleaning-pipeline) · [Captioning Approach](#captioning-approach) · [Audio-Video Alignment](#audio-video-alignment) · [Training Coordination](#training-coordination) · [Effectiveness Comparison](#effectiveness-comparison)

## Basic Information

### Name

CineDance / CineDance-1M (paper title: CineDance: Towards Next-Generation Multi-Shot Long-Form Cinematic Audio-Video Generation). This work comprises three deliverables: the CineDance-1M dataset (1 million multi-shot, long-form audio-video narrative sequences), the CineBench evaluation benchmark (1,000 test samples + a six-dimensional metric system), and the CineDance generation model (an open-source baseline adapted from LTX-2.3).

### Releasing institution/company ⚠️

A multi-institution academic collaboration, with no corporate-lab attribution. Participating institutions include: Shanghai Jiao Tong University, the University of Electronic Science and Technology of China, Zhejiang University, The University of Tokyo, and Nanyang Technological University. Author list: Yuheng Chen, Teng Hu, Yuji Wang, Qingdong He, Zhucun Xue, Qianyu Zhou, Jason Li, Lizhuang Ma, Jiangning Zhang, Dacheng Tao. The project homepage is maintained by the first author, Yuheng Chen (github.com/AliothChen). Which institution each author is specifically affiliated with is not explicitly listed on the page. [Uncertain]

### Release date (technical report/paper/open-source date)

First submitted to arXiv on June 8, 2026 (arXiv:2606.09639 v1), updated to v2 on June 11, 2026. The GitHub repository and project homepage went live at the same time. The first shard of the dataset (CineDance_01) was released on HuggingFace via gated access around July 2026. As of July 2026, the code (curation pipeline, inference, training) and model weights are still marked as "pending release" and have not yet been published.

### Type (model/dataset/toolchain/evaluation benchmark)

A composite, data-set-centric work with three components:
[Primary deliverable] Dataset — CineDance-1M, the first large-scale open research dataset at 1080p for multi-shot, long-form audio-video joint generation, comprising 1,021,657 narrative sequences / approximately 26.3K hours.
[Secondary deliverable 1] Evaluation benchmark — CineBench, 1,000 stratified test prompts + a six-dimensional human-aligned metric system.
[Secondary deliverable 2] Model — CineDance, an open-source baseline adapted from LTX-2.3 (13B video + 3B audio + 3B cross-modal attention), used to validate the dataset's effectiveness.
[Secondary deliverable 3] Toolchain — a three-stage data-curation pipeline (cleaning / narrative parsing / dual-modal annotation), which the paper pledges to open-source but has not yet released.

### Degree of openness (whether weights/code/data/pipeline are each open source) ⚠️

Openness is medium-to-high overall, but as of July 2026 it remains in a "released in batches" state:
[Data] Partially open. Released on HuggingFace via gated (application-required) access; the first batch, CineDance_01, is the first of four shards, containing approximately 240,488 video clips across 150 TAR archives, totaling 5.83 TB; it currently contains video only (the native audio track is retained inside the video container), and structured annotation files have not yet been released with this first batch. The license is CC-BY-NC-SA-4.0 (Attribution-NonCommercial-ShareAlike), explicitly restricting use to non-commercial research and education. Downloading requires a HuggingFace token and manual application review.
[Code] Not open source. The GitHub repository github.com/AliothChen/CineDance has been created, but the curation-pipeline code, inference toolkit, inference code, and training code are all listed as pending.
[Weights] Not open source. The CineDance model checkpoint is listed as pending release.
[Pipeline] Disclosed thoroughly at the methodological level — the tool selection at every stage of the three-stage process (EasyOCR, FFmpeg black-border detection, TransNetV2, Qwen3.5-27B, Qwen3.5-35B-A3B, Qwen3-Omni-30B-A3B), the quantitative funnel figures at each level, the annotation schema fields, and the ablation comparison tables are all given in the main text and could be re-implemented by a third party; however, the verbatim prompts and filtering threshold values are not fully disclosed.
[Dependency statement] The README credits LTX-2, the Qwen series, and vLLM in its acknowledgments. The repository itself has no stated code license. [Uncertain]

### Whether joint audio-video generation is supported, and the implementation approach (native joint/cascade/MoE fusion)

Supports joint audio-video generation, belonging to the "native joint audio-video generation" route, rather than a cascaded scheme that generates video first and dubs it afterward.
[Data-side positioning] One of the dataset's core selling points is precisely that it "retains native audio tracks" — the paper lists "the absence of the acoustic modality" as one of four major flaws in existing datasets; all sequences carry original synchronized audio, and the annotation covers both the visual and auditory tracks.
[Model-side implementation] The CineDance model is based on LTX-2.3, with an architecture of a 13B video DiT + a 3B audio branch + a 3B cross-modal cross-attention module; video and audio are coupled through cross-modal attention within the same diffusion process, making it a native joint architecture with dual-tower + cross-attention fusion, neither MoE fusion nor cascaded post-dubbing.
[Task definition] The paper formulates the task as T2AV (Text-to-Audio-Video), i.e., generating, from a single text prompt, a multi-shot long video with a synchronized audio track in one pass.

### List of research information sources (URLs of papers/technical reports/official documentation/news, with each source's nature annotated: official first-hand/same-team corroboration/third-party report)

1. [Official first-hand] Paper arXiv abstract page https://arxiv.org/abs/2606.09639 — title, authors, v1/v2 submission dates, abstract.
2. [Official first-hand] Full paper HTML https://arxiv.org/html/2606.09639v2 — the three-stage pipeline, the funnel table (Tab.3), the annotation schema, the CineBench six dimensions, the ablation tables (Tab.4/5/7), the dataset comparison table (Tab.6).
3. [Official first-hand] Paper PDF https://arxiv.org/pdf/2606.09639
4. [Official first-hand] Project homepage https://aliothchen.github.io/projects/CineDance/ — the five participating institutions, an overview of dataset scale, and links to each resource.
5. [Official first-hand] GitHub repository https://github.com/AliothChen/CineDance — the open-source progress checklist (dataset released in gated batches, code and weights pending).
6. [Official first-hand] HuggingFace dataset card https://huggingface.co/datasets/CineDance/CineDance_01 — the scale of the first shard (5.83TB / 240,488 clips / 150 TAR), the CC-BY-NC-SA-4.0 license, gated access, the video-only status, and a statement of limitations.
7. [Third-party index] The awesome-video-generation list https://github.com/kongzhecn/awesome-video-generation — included as corroborating evidence.

## Data Scale and Distribution

### Training data scale (video count/hours/token count, pretraining vs SFT split separately) ⚠️

As this is a dataset release, there is no traditional pretraining/SFT split; the scale figures are counted at the endpoint of the curation funnel:
[Final scale] 1,021,657 narrative sequences, totaling approximately 26.3K hours (26,300 hours).
[Raw collection] 45,181 long videos, totaling 32.8K hours.
[Intermediate products] TransNetV2 cut 25,899,474 atomic shots; narrative grouping produced 1,201,912 narrative sequences.
[Per-sequence characteristics] Average duration 92.8 seconds, averaging 24.2 consecutive shots, minimum spatial resolution 1080p.
[Annotation volume] An average of 6,496.3 words of structured dual-modal annotation per video (Tab.6), an annotation density that leads its class by an order of magnitude.
[First-batch open volume] The HuggingFace CineDance_01 batch contains approximately 240,488 clips / 5.83 TB, one quarter of the total shards.
[Model training usage] The sample counts, batch size, epoch count, learning rate, and total token count used in each of CineDance's two training stages are all undisclosed. [Uncertain]

### Data source composition (proprietary/public datasets/web crawling/licensed procurement/synthetic data) ⚠️

Primarily secondary curation of public datasets plus self-collection; no licensed procurement or synthetic data is mentioned:
[Public dataset sources] Explicitly lists reuse of three existing large-scale video datasets — MiraData, LVD-2M, and Koala-36M — as part of the material pool.
[Collection process reference] States that the collection pipeline norms of SkyReels-V2 and OpenHumanVid are followed for web-material collection.
[Content type] Predominantly cinematic / film / narrative long-form video content, emphasizing "feature-film-style" multi-shot narrative content, so the raw material is primarily long-form films rather than UGC short video — the fact that just 45,181 videos support 32.8K hours (roughly 43 minutes per video on average) confirms that the material is feature-film-level content.
[Hard admission criteria] Minimum 1080p spatial resolution, and must carry a native audio track.
[Synthetic data] None. All material is real, filmed footage; no synthetic or edited samples were constructed.
[Respective proportions of each source] The respective contribution shares of MiraData / LVD-2M / Koala-36M / self-collected material are not given. [Uncertain]

### Data compliance and provenance (proportion of licensed data, rights-cleared datasets, C2PA, etc.) ⚠️

Compliance disclosure is relatively weak, a notable shortcoming of this work:
[Release license] The dataset is released on HuggingFace under CC-BY-NC-SA-4.0, restricted to non-commercial research and educational use, and requiring share-alike.
[Access control] Adopts a gated-access mechanism, requiring a HuggingFace account token and manual review approval by the authors before download — a form of admission constraint against downstream misuse.
[Upstream copyright] The material is primarily sourced from web-collected film/TV content and existing public datasets (MiraData / LVD-2M / Koala-36M); the paper does not discuss the copyright licensing status of the upstream film/TV material, the proportion that is rights-cleared, nor does it mention C2PA or any other content-provenance authentication standard.
[Risk disclaimer] The dataset card explicitly states that "automatic and manual curation cannot guarantee removal of every low-quality, sensitive, or otherwise undesirable sample," partially shifting compliance responsibility to users, who are required to supplement quality inspection for their own use case.
[Proportion of licensed/procured data] No disclosure; inferred to be zero. [Uncertain]

### Clip duration distribution and segmentation strategy ⚠️

Duration distribution is the most core differentiating dimension of this dataset, and is paired with a dedicated "anti-fragmentation" strategy:
[Average duration] 92.8 seconds/sequence, several times that of comparable multi-shot datasets (compare MiraData 72.1s, LVD-2M 20.2s, SpeakerVid-5M 8.3s).
[Segmentation strategy] Does not adopt the traditional "one shot = one sample" segmentation; instead uses two levels: first cutting 25.89 million atomic shots with TransNetV2, then recombining consecutive shots "bottom-up" according to film-theory rules into narrative sequences — the sequence, not the single shot, is the final sample unit.
[Minimum-duration constraint] From empirical measurement on a manually curated reference set, the minimum duration for narrative completeness was found to be 18.4 seconds, from which a soft threshold of 20 seconds was set to prevent parsing out meaningless overly-short clips. The ablation shows that only 3.1% of sequences in the final data are shorter than 20 seconds.
[Duration trimming] MLLM-guided temporal truncation removes leading/trailing content (opening/closing credits, etc.) from long videos, with the truncation-length formula t = max(5 minutes, 0.1L), where L is the total duration of the original video.
[Distribution figure] Paper Fig.5 gives a joint histogram of duration and shot count, but specific percentiles (P50/P90, min/max) are not listed in text. [Uncertain]

### Resolution/aspect-ratio distribution and bucketing strategy ⚠️

[Resolution lower bound] A hard requirement of a minimum of 1080p, one of this dataset's key selling points relative to peer works — in the paper's comparison table (Tab.6), CineDance-1M is the only entry that simultaneously satisfies "1080p + native audio track + shot-level dual-modal dense annotation."
[Spatial preprocessing] Uses a "coarse crop + fine verification" two-pass approach: in the coarse-crop stage, EasyOCR locates and crops out burned-in-subtitle regions, and FFmpeg black-border detection crops out letterboxing; after shot segmentation is complete, OCR and black-border detection are re-run once more at the clip level for fine-grained re-verification.
[Aspect-ratio distribution / bucketing strategy] The paper gives no statistical distribution of aspect ratio, nor does it describe a resolution-bucketing strategy for training; given that the material is predominantly theatrical-grade film content, it is presumed to be predominantly widescreen aspect ratios, but there is no data to support this. [Uncertain]

### Category/domain distribution and mixture strategy (proportional control and concept balancing across people, actions, scenes, styles, etc.) ⚠️

An eight-dimensional taxonomy is used to safeguard diversity, but no quantitative proportion is published for any category:
[Eight dimensions] Genre, Format, Region, Modality, Story Logic, Era, Tone, Audience.
[Design intent] This system is used during collection and filtering to proactively broaden coverage and avoid the corpus concentrating in a single genre or era, serving the core scenario of "cinematic narrative."
[Relationship to evaluation] CineBench's 1,000 test prompts are also stratified-sampled along three axes — Theme/Style × Duration/Shot Count × Difficulty — where the theme/style axis echoes the training-side taxonomy.
[Mixture control strategy] The paper does not describe any explicit mixture control, concept balancing, or resampling mechanism based on this taxonomy; the eight-dimensional system reads more as a post-hoc characterization of diversity than a prior sampling constraint.
[Category lists and percentages under each dimension] Not published. Fine-grained proportions for people/actions/scenes/styles are likewise absent. [Uncertain]

### Audio category distribution and mixture (proportions and control strategy for speech/sound-effects-foley/music/ambient sound/silence — a dimension unique to AV models) ⚠️

Audio is explicitly decomposed into three categories and annotated separately, but no quantity proportion is published for any category:
[Three-way split] In the shot-level audio prompt, audio content is decomposed and described separately into three categories — music, ambient sound, and effects; dialogue/speech is handled through an independent ASR channel, together forming a de facto "speech / music / ambient sound / effects" four-category parallel annotation structure.
[Relationship to speaker] A separate character voice description field characterizes each character's vocal timbre.
[Quality-side characterization] Audio quality is quantified with two metrics: DNSMOS (signal fidelity) and the temporal variance of CLAP embeddings (measuring how much audio content varies over time, which can indirectly reflect silence/monotonic tracks); both are retained as metadata.
[Proportion and mixture control strategy] The paper gives no duration or sample proportions for speech / music / effects / ambient sound / silence respectively, nor does it describe any proactive audio-category mixture control strategy. This is an area of insufficient disclosure for this dataset on the AV dimension. [Uncertain]

### Narrative structure distribution (single-shot vs. multi-shot, average clip duration, shot-count distribution, whether native audio track is included) ⚠️

Narrative structure is the foundation of this dataset, and the section the paper devotes the most ink to:
[Multi-shot] An average of 24.2 consecutive shots per sequence, all samples being multi-shot, marking a generational gap from datasets predominantly single-shot such as LVD-2M (1.86 shots) and SpeakerVid-5M (1.27 shots).
[Average sequence duration] 92.8 seconds; compare MiraData 72.1s / 7.15 shots, LVD-2M 20.2s / 1.86 shots.
[Native audio track] The original synchronized audio track is fully retained across the board, making it the only dataset in the comparison table that simultaneously possesses 1080p + native audio track + shot-level dual-modal dense annotation.
[Definition of narrative sequence] "a continuous flow of diegetic time and causality" — i.e., a combination of shots that is continuous in dramatic time and causal chain, with character and environmental state maintained continuously, allowing spatial jumps but not narrative breaks.
[Four film-theory parsing rules] ① Multi-angle shots: only camera-angle changes while the event remains unified; ② Cross-cutting: alternating rapidly between different spaces but bound together by a unified causal tension; ③ Causal action / ellipsis: spatiotemporal jumps exist but the subsequent event directly and interpretably follows from the preceding event; ④ Montage: shots are individually disjointed but unified by an overarching theme or emotional arc.
[Shot-level attributes] Each shot is annotated with five dimensions: scale, angle, movement, narrative function, and duration category, plus shot transition type.
[Distribution detail] Fig.5 gives the joint duration–shot-count distribution, but specific percentiles for shot count are not given in text. [Uncertain]

### Language/accent distribution (data foundation for multilingual lip-sync capability) ⚠️

[Taxonomy level] The eight-dimensional taxonomy includes a Region dimension, indirectly covering the diversity demand across languages and cultural regions.
[Technical pipeline] ASR uses Qwen3-Omni-30B-A3B, itself a multilingual-capable model; on the evaluation side, CineBench's WER/CER use Whisper-large-v3, likewise a multilingual model, indicating that neither the data nor the evaluation is a monolingual setup.
[Speaker attributes] Provides a character voice description field and ASR-to-Character binding (binding speech sentences to a specific character anchor token), but the annotation schema shows no explicit language-tag, accent-tag, or emotion-tag fields.
[Quantitative distribution] The paper gives no statistics whatsoever on language proportions, accent distribution, or the Chinese/English ratio. [Uncertain]

## Cleaning Pipeline

### Overall structure of the cleaning funnel (number of filtering stages, ordering)

A strict three-stage curation pipeline, the methodological backbone of the paper:
[Stage 1: Data Preparation & Quality Assessment]
  1) Coarse-grained spatial cropping — EasyOCR locates burned-in subtitles, FFmpeg detects and crops out black borders/letterboxing;
  2) MLLM-guided temporal truncation — removes non-main-content such as opening/closing credits, truncation amount t = max(5min, 0.1L);
  3) Full-batch quality-metric computation — video side: DOVER (aesthetic score + technical score), AMT (motion smoothness); audio side: DNSMOS (signal fidelity), CLAP embedding temporal variance; audio-video alignment side: ImageBind (global cross-modal alignment), SyncNet (lip sync);
  4) Key design: all quality scores are "stored entirely as metadata" rather than used for hard pruning, allowing downstream users to flexibly construct task-specific subsets as needed;
  5) After shot segmentation, another round of OCR + black-border fine verification is performed at the clip level.
[Stage 2: Bottom-Up Narrative Parsing]
  1) TransNetV2 segments atomic shots (25.89 million);
  2) Using Qwen3.5-27B as the backbone, groups atomic shots into narrative sequences according to the four film-theory rules;
  3) Adopts "bottom-up shot indexing" rather than having the LLM directly output timestamps, significantly reducing timestamp hallucination;
  4) Context-aware sliding-window inference — a window of about 3 minutes, with window boundaries aligned to shot boundaries;
  5) A 20-second soft threshold against fragmentation.
[Stage 3: Configurable Dual-Modal Annotation]
  1) An anchor-token mechanism establishes a global character table and a global scene table;
  2) Visual annotation (Qwen3.5-35B-A3B) produces the five shot-level attributes + transitions + a localized character list + active scene + shot description + transition description;
  3) Audio annotation (Qwen3-Omni-30B-A3B) is split into three sub-tasks — sentence-level ASR, shot-level audio prompt, and character voice description — to suppress hallucination;
  4) Windowed ASR-to-Character binding.
[Final step] A final human check consisting of a three-person independent manual artifact audit of 500 randomly sampled clips.

### Funnel quantitative retention rate (input/output volume at each filtering stage and the final retention rate, e.g., Apollo 27%)

Paper Tab.3 gives a complete quantitative funnel, one of the most solidly disclosed parts of this work:

| Stage | Unit | Count | Duration |
|---|---|---|---|
| Raw collection | Videos | 45,181 | 32.8K hours |
| Spatiotemporal pre-filter | Videos | 44,579 | 32.5K hours |
| Shot detection | Shots | 25,899,474 | 32.5K hours |
| Narrative parsing | Sequences | 1,201,912 | 32.5K hours |
| Sequence pruning | Sequences | 1,079,382 | 28.6K hours |
| Post-verification | Sequences | 1,021,657 | 26.3K hours |

[Key retention rates]
  - Video-level pre-filtering retention rate 98.7% (45,181 → 44,579), duration 32.8K → 32.5K hours, indicating the entry-level material already had relatively high quality (since it derives from already-cleaned datasets such as MiraData/LVD-2M/Koala-36M);
  - Post-narrative-parsing sequence-pruning retention rate 89.8% (1,201,912 → 1,079,382), duration 32.5K → 28.6K hours;
  - Post-verification retention rate 94.7% (1,079,382 → 1,021,657), duration 28.6K → 26.3K hours;
  - Total retention rate from parsed sequences to the final dataset is approximately 85.0% (1,201,912 → 1,021,657);
  - Total retention rate by duration is approximately 80.2% (32.8K → 26.3K hours).
[Compression ratio] 25.89 million atomic shots are compressed into 1.2 million narrative sequences, averaging about 21.5 shots combined into each sequence.
[Artifact-audit comparison] In the manual audit of 500 random clips, CineDance-1M's non-compliance rate is 2.8%, versus 37.4% for Koala-36M, a 13.4× improvement.

### Shot-segmentation method (PySceneDetect/in-house model/shot-aware splitting)

[Tool] TransNetV2, a mature deep-learning shot-boundary detection model in the industry (more robust to gradual transitions, dissolves, and wipes than pixel-histogram-based traditional methods such as PySceneDetect), cutting 25,899,474 atomic shots from 44,579 cleaned videos.
[Key differentiator] Segmentation is only an intermediate step, not the endpoint. Traditional datasets treat "one shot = one sample," while this work adds a further layer of "bottom-up narrative grouping" on top of shots, using Qwen3.5-27B to regroup semantically coherent adjacent shots according to the four film-theory rules (multi-angle, cross-cutting, causal action/ellipsis, montage) into 1,201,912 narrative sequences — the sequence, not the shot, is the final sample.
[Anti-hallucination design] Rather than letting the LLM directly output timestamps, shots are first indexed numerically, and the LLM is made to output shot-number-based grouping results (bottom-up shot indexing), substantially reducing timestamp hallucination.
[Long-video inference] A context-aware sliding window, roughly 3 minutes wide, with the window's cut points forcibly aligned to shot boundaries, avoiding splitting a single shot in half.
[Anti-fragmentation] Empirical measurement on a manually curated reference set found narrative completeness requires a minimum of 18.4 seconds, hence the 20-second soft threshold; ultimately only 3.1% of sequences are shorter than 20 seconds.
[Parsing quality] Qwen3.5-27B combined with the bottom-up strategy achieves F1 = 88.4% on the parsing task (Tab.4).

### Quality filtering (aesthetic scoring, sharpness, OCR text filtering, black-border/watermark/logo detection) ⚠️

Overall adopts a hybrid strategy of "full-batch scoring + metadata retention + hard rules used only for artifacts," a notable departure from the "one-size-fits-all threshold" approach used by most datasets:
[Hard exclusion (rule-based)]
  - Burned-in subtitles: detected and cropped out by EasyOCR, re-verified once more after shot segmentation;
  - Black borders/letterboxing: FFmpeg black-border detection, likewise applied in two passes (coarse crop + clip-level fine verification);
  - Opening/closing credits/non-main content: MLLM-guided temporal truncation, with truncation amount t = max(5min, 0.1L).
[Soft scoring (metadata-type, no hard cropping)]
  - DOVER: split into two components, Aesthetic Quality and Technical Score (sharpness/technical quality);
  - AMT: Motion Smoothness;
  - Audio side: DNSMOS and CLAP temporal variance;
  - Alignment side: ImageBind and SyncNet.
  The paper's original text explicitly states, "we store all quality scores as metadata, enabling users to flexibly construct task-specific subsets" — handing the threshold decision over to downstream users, so the paper itself does not publish any specific aesthetic-score/technical-score cutoff value.
[Manual artifact audit (final verification)] 500 random clips were sampled, with three independent annotators each reviewing residual artifacts, with disagreements resolved by joint review. Target artifact checklist: burned-in subtitles, station logos, letterbox black borders, watermarks, TV-network overlays, title cards, end credits, screen-recording footage, transition effects, still-frame holds. Result: non-compliance rate 2.8%, versus 37.4% for Koala-36M, a 13.4× improvement.
[Automated watermark/logo detection methods] The paper does not state whether there is an automated watermark/logo detection model; from the description, this seems to rely primarily on upstream dataset cleaning plus a final manual spot-check. [Uncertain]

### Motion filtering (optical-flow/motion-score thresholds, exclusion of static and shaky footage) ⚠️

[Metric] Uses AMT to compute Motion Smoothness as a quantitative characterization of the motion dimension, stored alongside DOVER's aesthetic/technical scores.
[Strategy] Likewise follows the "no hard pruning" principle — no filtering threshold is set on the motion score, which serves only as metadata for downstream selection as needed.
[Indirect static exclusion] The target checklist for the manual artifact audit includes "still-frame holds" and "transition effects," representing a qualitative check against static/anomalous-motion content, but this is not an automated filter based on an optical-flow threshold.
[Not adopted] The paper does not mention optical-flow computation, motion-magnitude thresholds, camera-shake detection, or other common motion-filtering means, nor does it give any distribution or threshold value for motion scores. This is a relatively weak link compared with other large-scale datasets. [Uncertain]

### Deduplication method (exact deduplication and embedding-based semantic deduplication, recorded separately) ⚠️

The paper describes no deduplication step at all — neither hash/fingerprint-based exact deduplication nor embedding-based (e.g., CLIP/ImageBind feature) semantic deduplication; the main-text three-stage pipeline contains no deduplication step.
[Presumed mitigating factors] ① The raw material is only 45,181 long videos, a moderate order of magnitude, so the probability of full duplication between long-form films is relatively low; ② the upstream MiraData / LVD-2M / Koala-36M datasets have each already undergone their own deduplication; ③ narrative sequences are composed of consecutive shots from the same long film, so different sequences within the same film naturally do not overlap.
[Potential risk] No handling is described for overlap between datasets from different sources (MiraData and Koala-36M may both include the same film). Overall, this is a disclosure gap. [Uncertain]

### VLM/LLM as quality inspector (multimodal large-model quality scoring and mismatch removal — the 2026 trend of shifting from shallow scorers to large-model semantic judgment)

Deep large-model involvement is the core characteristic of this pipeline, and it goes beyond scoring alone — it also shoulders the heavy task of "structured semantic parsing," fully reflecting the 2026 trend of shifting from shallow scorers to large-model semantic judgment:
[MLLM for temporal truncation] In Stage 1, a multimodal large model determines the opening/closing boundaries of long videos to decide the truncation position.
[LLM for narrative parsing (the heaviest step)] Qwen3.5-27B determines, according to the four film-theory rules, which adjacent shots constitute one complete narrative sequence — this already goes beyond "quality inspection" in scope, using the large model as a film-grammar parser. The key engineering measures are bottom-up shot indexing (to avoid timestamp hallucination) and a 3-minute sliding window aligned to shot boundaries. Tab.4's ablation shows this combination achieves F1 = 88.4%, with only 3.1% of sequences below the 20-second soft threshold.
[MLLM for visual annotation] Qwen3.5-35B-A3B (an MoE architecture, 35B total parameters / 3B activated) produces the five-dimensional shot attributes and descriptions.
[Omni model for audio annotation] Qwen3-Omni-30B-A3B (30B total parameters / 3B activated) handles ASR, audio prompts, and character voice description as three tasks, and performs ASR-to-Character binding.
[Three anti-hallucination engineering designs] ① Audio annotation is split into three sub-tasks called separately, rather than output in one pass (reducing hallucination); ② the ASR stage deliberately does not perform speaker-to-character binding, with binding kept as a separate subsequent step; ③ the binding uses a windowed scheme (filtering non-speech intervals, preserving shot and sentence integrity), raising Qwen3-Omni-30B-A3B's binding accuracy from 67.2% on whole-clip input to 95.4%.
[Comparison with traditional diarization (Tab.5/7)] Windowed Qwen3-Omni 95.4% > the Gemini series 82.8%~87.4% > DiariZen 63.1% > Pyannote-3.1 62.7%, validating the advantage of the route of "replacing dedicated diarization tools with an Omni large model."
[Quality-scoring side] Scoring models such as DOVER/DNSMOS/ImageBind/SyncNet all run at full scale but do not perform hard pruning, and are stored only as metadata.

### Safety and compliance filtering (NSFW, copyright, faces/privacy) ⚠️

Safety and compliance filtering is the most weakly disclosed dimension of this work:
[Paper level] The three-stage pipeline contains no description of any NSFW detection, violent-content filtering, facial-privacy handling, or copyright-detection step. The manual audit's artifact checklist is also entirely image-quality issues (subtitles, watermarks, logos, black borders, screen recordings, etc.), with no safety category included.
[Release-level substitute constraints] ① Gated access is used for release, requiring manual review of applicants; ② the license is limited to CC-BY-NC-SA-4.0 non-commercial use; ③ the dataset card explicitly disclaims — "automatic and manual curation cannot guarantee removal of every low-quality, sensitive, or otherwise undesirable sample" — requiring users to perform their own quality inspection according to their application scenario.
[Judgment] Safety responsibility is effectively shifted to downstream users through gating plus a disclaimer, rather than being resolved within the pipeline. Given that the material is film/TV content (which may contain violence or adult plot elements), this is an area that needs significant supplementation for practical use. [Uncertain]

## Captioning Approach

### Captioning model(s) used (in-house VLM/open-source model, model scale) ⚠️

The entire chain uses open-source models from the Qwen series, with no in-house captioning model; the inference framework credits vLLM in its acknowledgments:
[Narrative-parsing backbone] Qwen3.5-27B — handles shot grouping and narrative-boundary determination; Tab.4's ablation compares different Qwen backbone scales, with the 27B version paired with the bottom-up strategy achieving the best F1 = 88.4%.
[Visual annotation model] Qwen3.5-35B-A3B — an MoE architecture (approximately 35B total parameters, 3B activated), producing shot-level five-dimensional attributes, transition type, localized character list, active scene, shot description, and transition description.
[Audio annotation model] Qwen3-Omni-30B-A3B — an omni-modal MoE model (approximately 30B total, 3B activated), simultaneously handling sentence-level ASR extraction, shot-level audio prompt (music/ambient sound/effects) generation, and character voice description as three sub-tasks, and completing the binding of ASR sentences to character anchor tokens.
[Selection rationale] Visual and audio annotation are each assigned to a specialized model, and both choose sparse MoE architectures (3B activated), balancing annotation throughput and quality at the million-sequence scale; ASR and audio description are not delegated to a dedicated model such as Whisper, but are instead handled uniformly by the Omni model, on the grounds that Tab.5/7 show it far outperforms dedicated diarization tools such as Pyannote-3.1 (62.7%) and DiariZen (63.1%) on the speaker-attribution task.
[Separately used on the evaluation side] CineBench's WER/CER computation uses Whisper-large-v3.
[Undisclosed] The verbatim annotation prompts, the per-model inference cost, and whether task-specific fine-tuning was performed on the Qwen models. [Uncertain]

### Caption density and structuring level (short/long/dense descriptions, structured fields such as camera movement, style tags)

Caption density and structuring level represent this dataset's largest generational advantage over existing work, which the paper terms "Configurable / Hierarchical Dual-Modal Annotation":
[Density] An average of 6,496.3 words of structured annotation per video (Tab.6), far exceeding datasets such as MiraData that have only video-level captions, and the only one in its class providing shot-level dense annotation.
[Two-level structure]
  · Global level: a global character table [⟨char₁⟩,…,⟨charₙ⟩] and a global scene table [⟨scene₁⟩,…,⟨sceneₘ⟩], defined as anchor tokens;
  · Shot level: each shot's description explicitly references the above anchor tokens, achieving cross-shot identity and scene binding.
[The value of the Anchor Token mechanism] This is the key design for solving multi-shot consistency — a generation model can thereby know that "⟨char₂⟩ in shot 3 and ⟨char₂⟩ in shot 17 are the same person"; on the evaluation side CineBench also uses this token directly as the anchor for judging identity continuity (ArcFace clustering) and scene continuity (DINOv2 cosine similarity).
[Shot-level structured fields] Five-dimensional shot attributes: scale, angle, movement, narrative function, duration category; plus shot transition type, localized character list (characters appearing in the shot), active scene (the shot's scene), shot description, and transition description.
[Meaning of "configurable"] All fields and quality scores can be combined and filtered, letting users assemble task-specific subsets as needed (e.g., only clips with camera-movement annotation, or only those with a high aesthetic score).
[Model-side usage format] Organized during training as: a global header (character/scene definitions) + per-shot blocks [SHOT i | scene sᵢ | camera κᵢ] ⊕ transition description div ⊕ dialogue dia ⊕ {(speaker spkᵢ,ℓ, line speechᵢ,ℓ)}.

### Joint audio-video caption structure (whether both visual and auditory tracks are covered simultaneously, whether split into separate fields — e.g., LTX-2's full-soundscape description, Script-a-Video's factorized streams, Foley-Omni's three fields)

The joint audio-video caption adopts a "dual-track parallel, field-split" design, in which the visual and auditory tracks each form their own system but are aligned via shared anchor tokens and shot indices:
[Visual track (produced by Qwen3.5-35B-A3B)] The five-dimensional shot attributes (scale/angle/movement/narrative function/duration category), transition type, localized character list, active scene, shot description, transition description.
[Auditory track (produced by Qwen3-Omni-30B-A3B, split into three fields)]
  ① Sentence-level ASR transcript segments — at this stage, speaker identity is deliberately not bound;
  ② Shot-level audio prompt — natural-language description covering music, ambient sound, and effects, i.e., non-speech soundscape description;
  ③ Character voice description — characterizes each character's vocal timbre.
[Coupling points between the two tracks] ① Shared global character anchor token ⟨charₖ⟩: in the subsequent binding step, ASR sentences are attached to a specific character token, thereby aligning with the shots in the visual track in which that character appears; ② shared shot index: the audio prompt is shot-level, corresponding one-to-one with the visual description shot by shot.
[Comparison with peer schemes] Compared with LTX-2's "single full-soundscape description," this is more structured; it belongs to the same design lineage as Script-a-Video's factorized streams and Foley-Omni's three-field design, but CineDance's distinguishing feature is introducing the anchor-token consistency mechanism into cross-shot long-sequence scenarios, and treating "speaker-to-character binding" as an independent, separately evaluated sub-task (95.4% accuracy).
[Design motivation] The paper explicitly states that the three audio sub-tasks are called separately rather than output in one pass in order to reduce hallucination.

### Dialogue transcription and speaker attribute annotation (ASR transcription + speaker identity/language/accent/emotion) ⚠️

Dialogue processing is the most finely worked part of this dataset on the AV dimension:
[ASR transcription] Sentence-level transcription is performed by Qwen3-Omni-30B-A3B, rather than a streaming output over the whole clip, with the segmentation granularity aligned to sentence boundaries.
[Speaker-identity binding (a core contribution)] Designed as a step independent of and subsequent to ASR, binding each ASR sentence to a global character anchor token ⟨charₖ⟩. A windowed scheme is adopted: first filtering out non-speech intervals, with window segmentation ensuring both that shots are not cut and that sentences are not truncated.
[Quantitative effect (Tab.5 / Tab.7)] On a benchmark of 100 manually annotated clips:
  · Qwen3-Omni-30B-A3B + windowing: 95.4% (whole-clip input only 67.2%, sliding-window prompt 83.1%, whole-clip 56.4% — the exact figures differ slightly across tables depending on the setting);
  · The Gemini series of models: 82.8% ~ 87.4%;
  · DiariZen: 63.1%;
  · Pyannote-3.1: 62.7%.
  The conclusion is that the Omni large model plus windowing engineering is significantly better than traditional dedicated speaker-separation tools.
[Timbre attribute] Provides a character voice description field describing each character's vocal characteristics.
[Missing dimensions] No explicit language tag, accent tag, or emotion tag field is seen, and structured attributes such as speaker gender/age are not annotated. [Uncertain]

### Geometric and structured annotation (camera parameters, depth, 3D point tracks, action annotation, explicit state)

[Existing structured annotation] The five-dimensional shot-level attributes contain fairly strong cinematographic-grammar information: scale (e.g., close-up/medium/long shot), angle, movement (e.g., dolly/pan/tilt), narrative function, and duration category; as well as shot transition type (e.g., hard cut/dissolve). At training time, the camera parameter is injected as an independent conditioning term in the form [SHOT i | scene sᵢ | camera κᵢ].
[Structuring of scene and identity] The global scene table ⟨sceneₘ⟩ and the global character table ⟨charₙ⟩ constitute an explicit state-annotation system that allows cross-shot identity/scene continuity to be explicitly modeled and evaluated.
[Missing dimensions] No numeric camera extrinsics/intrinsics (camera pose, focal length, etc.), depth maps, 3D point tracks, optical-flow fields, human pose, or action skeletons are provided. Camera information is a linguistic qualitative description (κᵢ) rather than numeric parameters.
[Evaluation-side geometric substitute] CineBench uses ArcFace (facial clustering) and DINOv2 (visual-feature cosine similarity) for consistency judgment, which operates at the representation level rather than the geometric level.

### Synthetic data construction (controlled perturbation/edits constructing training pairs, e.g., InstructAV2AV)

None. CineDance-1M is entirely real, filmed film/TV material; the pipeline contains no controlled perturbation, edit-based construction, paired sample synthesis (such as InstructAV2AV-style before/after editing pairs), or augmentation-style synthesis step.
[Closest operation] The only "construction" is regrouping atomic shots into narrative sequences, which is a re-grouping of real material rather than synthesis; also, the CineDance model's training uses a DARC curriculum (continuous noise added to reference frames ρ(rₖ; ηᵥ(u)) = ηᵥ(u)·rₖ + (1−ηᵥ(u))·εₖ, stochastic index switching, and reference-frame dropping with probability p_drop(u)), which is an input-perturbation strategy applied at training time, not offline synthetic-data construction.

### Degree of human involvement (manual annotation, manual quality inspection, model pre-screening + manual review) ⚠️

Human involvement is positioned as "threshold calibration + quality spot-checking + benchmark construction," and does not participate in large-scale annotation production (all annotation is performed by large models):
[① Threshold calibration] Empirical measurement on a manually curated reference set found the minimum duration for narrative completeness to be 18.4 seconds, from which the anti-fragmentation soft threshold of 20 seconds was set — a typical case of human priors directly determining a pipeline parameter.
[② Manual artifact audit] 500 random clips were sampled, with three independent annotators each reviewing residual artifacts (burned-in subtitles, logos, watermarks, TV-network overlays, title cards, end credits, screen recordings, transition effects, still-frame holds, etc.), with disagreements resolved by joint review, arriving at a 2.8% non-compliance rate compared against Koala-36M's 37.4%.
[③ Manual annotation benchmark] A small benchmark of 100 manually annotated clips was constructed, used to evaluate ASR-to-Character binding accuracy (95.4%) against various diarization baselines.
[④ CineBench human evaluation] Every video is scored by 10 independent evaluators using a 5-point Likert scale (1 = unusable, 5 = excellent), double-blind with randomized presentation order, with Spearman rank correlation then used to verify the consistency between automatic metrics and human judgment.
[Pattern summary] This belongs to the modern paradigm of "full-scale large-model annotation + small-sample manual verification and alignment," with human cost concentrated on the verification side rather than the production side. The recruitment method, headcount, and compensation of the manual annotators are undisclosed. [Uncertain]

## Audio-Video Alignment

### Audio-video synchronization detection method (lip sync, event alignment)

Audio-video synchronization in this work follows two lines: computed as a full-batch metadata metric on the data side, and reported as an independent dimension on the evaluation side (CineBench):
[Data-curation side (Stage 1)]
  · SyncNet — used for lip-sync detection, measuring the temporal alignment between lip shape and speech in speaking shots;
  · ImageBind — used for global cross-modal semantic alignment, measuring how well the visual content matches the audio content in a semantic space, covering non-speech scenarios (such as the correspondence between sound effects and events).
  Both are computed and stored as metadata, without hard filtering.
[Evaluation side (CineBench's AV Sync dimension)]
  · Sync-C / Sync-D (SyncNet's confidence and offset-distance components);
  · IB-Score (ImageBind cross-modal similarity).
[Design orientation] The paper deliberately distinguishes "lip sync" (SyncNet, targeting frame-level temporal alignment between speech and lip shape) from "global semantic alignment" (ImageBind, targeting overall audio-visual content matching), forming two complementary detection criteria.
[Limitation] No event-level synchronization detection method such as Synchformer or AV-align is used; no dedicated detection means for sound-effect-event (foley event) temporal alignment is seen.

### Specific synchronization detection metrics and thresholds (SyncNet/Synchformer/LSE/in-house, threshold values, e.g., MOVA's LSE-D≤9.5 and LSE-C≥4.5, SkyReels-V4's SyncNet |offset|≤3∧conf>1.5) ⚠️

[Metric selection] Data side: SyncNet (lip sync) + ImageBind (global cross-modal alignment); evaluation side: Sync-C, Sync-D, IB-Score.
[Thresholds] No filtering threshold value for any synchronization metric is published in the paper — this is the fundamental methodological difference between this work and works such as MOVA (LSE-D ≤ 9.5 and LSE-C ≥ 4.5) and SkyReels-V4 (SyncNet |offset| ≤ 3 and conf > 1.5) that adopt hard thresholds. CineDance explicitly adopts a "score at full scale → store as metadata → no hard pruning" strategy; the paper's original text states, "we store all quality scores as metadata, enabling users to flexibly construct task-specific subsets," delegating the threshold decision to downstream users.
[Trade-off evaluation] The advantage is preserving data completeness and downstream flexibility, avoiding losing the long tail from over-filtering; the disadvantage is that the dataset itself does not guarantee a lower bound on synchronization quality, requiring users to set their own filtering thresholds, and the paper provides no recommended threshold reference.
[Specific SyncNet version, ImageBind version, and score-distribution range] All undisclosed. [Uncertain]

### Separate handling of temporal synchronization vs. semantic synchronization (time alignment and content semantic matching treated as two independent filtering conditions)

This work makes a clear separation between temporal synchronization and semantic synchronization at the metric-design level, a relatively clear-headed point in its methodology:
[Temporal synchronization] Handled by SyncNet, producing Sync-C (confidence) and Sync-D (offset distance), characterizing the frame-level temporal alignment precision between lip shape and speech, meaningful only in shots where a character is speaking.
[Semantic synchronization] Handled by ImageBind (IB-Score / IB-A Score), characterizing the semantic matching degree between visual content and audio content in a cross-modal embedding space, applicable to music, ambient sound, effects, and other non-speech scenarios, without requiring frame-level alignment.
[The two are parallel, not merged] The two are stored separately in the data metadata, and reported separately in CineBench's AV Sync dimension, constituting two independent criteria. In addition, the Prompt Alignment dimension's IB-A Score further splits out "the semantic consistency between audio and the text prompt" as a separate item.
[What is not done] Neither has a filtering threshold set, so in practice they are not used as "two independent filtering conditions" during dataset construction — the separation is manifested at the measurement and evaluation level, not at the filtering level.

### Audio quality filtering (SNR, silence detection and silence-ratio threshold, exclusion of tracks with no audio, exclusion of off-screen sound sources, background-music separation) ⚠️

[Metrics used]
  · DNSMOS — a no-reference MOS estimate of speech-signal fidelity, measuring degradation such as noise and distortion;
  · CLAP embedding temporal variance — measures how much audio content varies over time, and can indirectly identify monotonic, constant, or near-silent tracks (extremely low variance often means an uninformative audio track).
[Strategy] Likewise follows the global "no hard pruning" principle — both metrics are stored as metadata for downstream filtering; the paper gives no DNSMOS score floor or variance threshold.
[Admission constraint] Material admission requires "carrying a native audio track," so samples with no audio track are excluded at the collection stage.
[Non-speech interval handling] In the windowed ASR-to-Character binding scheme, non-speech intervals are filtered out first before binding — a functional treatment of silence/non-speech segments, but its purpose is to improve binding accuracy rather than data filtering.
[Not addressed] No mention of SNR computation, silence-ratio threshold, exclusion of off-screen voice sources, or background-music separation (source separation, e.g., using Demucs/BS-RoFormer) — as film/TV material, background music mixed with dialogue is a common phenomenon, and the paper does not explain how this is handled. [Uncertain]

### Classification and separate processing strategy for speech/sound effects/music

[Classification method] In the shot-level audio prompt, non-speech audio is explicitly split into three categories described separately — music, ambient sound, and effects; speech/dialogue is routed through an independent ASR channel and further bound to a character anchor token; character vocal timbre has a separate character voice description field. Overall this forms a "speech / music / ambient sound / effects" four-category parallel annotation structure.
[Separate processing strategies]
  · Speech: sentence-level ASR transcription → windowed speaker-to-character binding (95.4% accuracy) → character voice description; at training time injected into the prompt as (spkᵢ,ℓ, speechᵢ,ℓ) tuples; evaluated using Whisper-large-v3-computed WER/CER.
  · Music/ambient sound/effects: uniformly handled via natural-language soundscape description (audio prompt), with no further category labeling or track separation; evaluated using AudioBox-Aesthetics' three components — PQ (Production Quality), CE (Content Enjoyment), CU (Content Usefulness).
[Design orientation] Speech follows a "precise transcription + identity binding" structured route, while non-speech follows a "natural-language description" soft route, with the two processing paradigms clearly distinguished.
[Not done] No audio source separation is performed (splitting dialogue, music, and effects into separate tracks), and no differentiated quality threshold or sampling mixture is set for the three audio categories.

## Training Coordination

### Multi-stage training curriculum and data curriculum scheduling (basis for stage division: resolution/duration/quality score/modality; low-res→high-res, image→video, short→long) ⚠️

The CineDance model (adapted from LTX-2.3) adopts a dual-curriculum design — a data curriculum and a reference-frame curriculum advancing in parallel:
[Data-Driven Curriculum (two stages)]
  · Stage 1: 2–3 adjacent shots, 10–12 seconds, with the training target being local shot-switching capability;
  · Stage 2: up to 8 shots, approximately 30 seconds, with the training target being cross-shot consistency.
  The stage division is based on the "shot count + duration" narrative-complexity axis, rather than the traditional resolution or quality-score axis — an orientation different from most video-generation models (low-res→high-res, image→video), reflecting that this work's core challenge lies in narrative length rather than image quality.
[Dual-Axis Reference Curriculum (DARC)] Progressively removes the reference-frame scaffold as training progresses u:
  · Visual scaffold: continuous noise added to reference frames ρ(rₖ; ηᵥ(u)) = ηᵥ(u)·rₖ + (1−ηᵥ(u))·εₖ, with the noise proportion increasing over training;
  · Temporal scaffold: stochastic index switching with probability q(ηₜ);
  · Reference-frame dropping: reference tokens are removed entirely with probability p_drop(u).
  Together, the three achieve a smooth transition from "strong reference guidance" to "autonomous generation."
[Resolution curriculum] Material is uniformly 1080p from the start; the paper does not describe a low-res-to-high-res resolution progression strategy.
[Undisclosed] The training steps, sample counts, batch size, learning rate, and total compute for each stage. [Uncertain]

### Data mixture changes across training stages (pretraining/annealing/high-quality SFT subset) ⚠️

[Known mixture changes] The data-mixture change is primarily expressed as a progression along the "narrative complexity" dimension, rather than modality or quality mixture: Stage 1 uses a short-sequence subset of 2–3 shots / 10–12 seconds, and Stage 2 switches to a longer-sequence subset of up to 8 shots / about 30 seconds. Since CineDance-1M averages 24.2 shots / 92.8 seconds per sequence, both training stages in practice extract sub-clips from the complete sequences rather than using the full length.
[Supporting mechanism for subset construction] The dataset stores all quality scores (DOVER aesthetic/technical, AMT motion, DNSMOS, CLAP variance, ImageBind, SyncNet) and structured fields as full-batch metadata; the paper emphasizes this allows users to "flexibly construct task-specific subsets," i.e., the infrastructure for mixture adjustment is fully in place.
[Undisclosed] The specific sample counts and proportions for each stage, whether an annealing stage exists, whether there is a high-quality-subset SFT stage, the mixing ratio of image-text data to video data, and any supplementary audio-only data. The paper gives almost no quantitative description of the model's training data mixture. [Uncertain]

### Post-training data (SFT curated-set scale and selection criteria, number of preference pairs and annotation method, reward-model training data) ⚠️

The paper describes no post-training step of any kind — no SFT curated set, no preference-pair annotation, no reward-model training data, and nothing related to RLHF/DPO. The CineDance model's training description ends at the two-stage data curriculum plus the DARC reference curriculum, positioned as a "robust open baseline" rather than a fully productized model pursuing SOTA, so post-training is out of scope for this paper.
[The only relevant human-preference data] The human Likert scores collected in the CineBench evaluation (10 evaluators per video, 5-point scale, double-blind and randomized), but their purpose is to verify the Spearman rank correlation between automatic metrics and human judgment, serving as evaluation-calibration data, not used to train a reward model. [Uncertain]

### Data-processing infrastructure and throughput (NeMo Curator/Data-Juicer/in-house, GPU acceleration ratio, processing scale, cost) ⚠️

The paper discloses no information whatsoever about data-processing infrastructure and throughput — no GPU model or count, no GPU-hour figures, no end-to-end pipeline processing time, no annotation cost, no parallelism framework (such as Ray/Spark) description, and no mention of ready-made data-processing frameworks such as NeMo Curator or Data-Juicer.
[Only clue] The GitHub README's acknowledgments mention vLLM, from which it can be inferred that large-scale LLM/MLLM annotation inference is carried on vLLM to improve throughput.
[Scale-estimate reference (not paper-reported data)] The processing scale is: 32.8K hours of video passed through TransNetV2 to cut 25.89 million shots, then Qwen3.5-27B performs parsing to produce 1.2 million sequences, Qwen3.5-35B-A3B performs full-batch visual annotation, and Qwen3-Omni-30B-A3B performs full-batch audio annotation, with an average of 6,496.3 words of annotation output per video — an output volume at the scale of million-level sequences × 6.5K words implies an enormous inference cost, yet the paper gives no quantitative account of this whatsoever, making it the primary obstacle to reproducing this pipeline. [Uncertain]

## Effectiveness Comparison

### Quantified impact of data-strategy ablations (distinguishing: filtering-strictness ablation / caption-density-style ablation / data-mixture ablation, and their corresponding evaluation metrics) ⚠️

The paper's ablations concentrate on "method selection within the annotation pipeline," rather than the traditional sense of "the impact of data strategy on generation effectiveness" — a distinction that needs to be made explicit:
[Pipeline ablations performed]
  · Tab.4 (narrative-parsing strategy ablation): compares "letting the LLM directly output timestamps" versus "bottom-up shot-index grouping," and horizontally compares different Qwen backbone scales. The best combination, Qwen3.5-27B + the bottom-up strategy, achieves F1 = 88.4%, with only 3.1% of sequences below the 20-second soft threshold (a fragmentation rate significantly lower than the control group).
  · Tab.5 (speaker-separation ablation): on a benchmark of 100 clips, Qwen3-Omni-30B + sliding-window prompt reaches 83.1%, versus only 56.4% for whole-clip input.
  · Tab.7 (ASR-to-character-binding ablation): the windowed scheme raises Qwen3-Omni from 67.2% to 95.4%; compared against the Gemini series 82.8%–87.4%, DiariZen 63.1%, Pyannote-3.1 62.7%.
  · Artifact-audit comparison: CineDance-1M non-compliance rate 2.8% versus Koala-36M 37.4% (13.4× improvement), which can be regarded as qualitative evidence of "cleaning strictness."
[Data ablations not performed] The paper reports no quantified impact on downstream generation quality (CineBench scores across dimensions) from "removing a given filtering stage / changing the shot-grouping threshold / adjusting caption density or style / changing data mixture." That is, the three classic categories of data ablation — filtering-strictness ablation, caption-density/style ablation, and data-mixture ablation — are all absent.
[Judgment] This work's empirical emphasis is on "annotation quality itself being measurable and optimizable," rather than "how data strategy changes generation-model performance," and the latter is a clear evidence gap. [Uncertain]

### Evidence for quality vs. quantity (cases where small, refined data outperforms large, messy data) ⚠️

The paper provides no direct controlled experiment demonstrating "small and refined outperforming large and messy" — there is no comparison of training the same model on a CineDance-1M subset versus a larger, noisier dataset.
[Indirect evidence]
  · Artifact-audit comparison: CineDance-1M non-compliance rate 2.8% versus Koala-36M 37.4%, a 13.4× improvement. Koala-36M is far larger in scale than CineDance-1M, and the paper uses this to argue that "this dataset, though smaller, is much cleaner," but this cleanliness difference is not converted into a comparison of downstream generation metrics.
  · The dataset comparison table (Tab.6): argues for comprehensive leadership on the quality dimension via the combination of "1080p + native audio track + shot-level dual-modal dense annotation + 92.8s/24.2 shots," which is an attribute comparison rather than an effectiveness comparison.
[The paper's overall stance] Actually leans more toward "having both quality and scale" rather than "trading quantity for quality" — deliberately retaining the scale of 1 million sequences, while using the design of "no hard pruning, store all as metadata" to hand the quality-quantity trade-off decision to downstream users, which is itself a way of avoiding the route of "one-size-fits-all pursuit of a high-quality subset."
[Conclusion] This dimension lacks direct experimental support. [Uncertain]

### Alignment between training-data domain distribution and the evaluation-benchmark taxonomy (e.g., VABench's seven major categories)

There is partial correspondence, but not a strict one-to-one alignment, between the training-data taxonomy and the CineBench taxonomy:
[Training-side taxonomy (8 dimensions)] Genre, Format, Region, Modality, Story Logic, Era, Tone, Audience.
[CineBench's stratification axes (3 axes)]
  · Theme/Style — echoes the training side's Genre/Tone/Era and other dimensions;
  · Duration/Shot Count — split into three tiers: 10s / 2–3 shots, 30s / 4–9 shots, 60s / 10–20 shots — directly corresponding to the training data's core differentiating dimension, and also strictly aligned with the model's two-stage curriculum (10–12s/2–3 shots → 30s/8 shots);
  · Difficulty — within each tier, split into Easy/Medium/Hard by tertile, with the difficulty formula D = n_char + 1.5·n_scene + 0.4·log(1 + n_spk·L_ASR), where n_char is the character count, n_scene the scene count, n_spk the speaker count, and L_ASR the dialogue length — every term in this formula is taken directly from fields in the training data's annotation schema (the global character table, the global scene table, the ASR binding result), making this the point of tightest coupling between the data schema and the evaluation system.
[Correspondence between the six evaluation dimensions and the data dimensions]
  1) Video Quality (Aesthetic Quality, MUSIQ imaging quality, AMT motion smoothness) — corresponds to the data-side DOVER/AMT;
  2) Audio Quality (AudioBox-Aesthetics' PQ/CE/CU, Whisper-large-v3's WER/CER) — corresponds to the data-side DNSMOS and ASR annotation;
  3) AV Sync (Sync-C/Sync-D, ImageBind IB-Score) — corresponds to the data-side SyncNet/ImageBind;
  4) Prompt Alignment (shot-level ViCLIP, video-level VideoScore-v1.1, audio-level IB-A Score);
  5) Narrative Continuity (identity continuity uses ArcFace clustering matched against the ⟨charₖ⟩ token; scene continuity uses DINOv2 cosine similarity matched against shots sharing a ⟨sceneₖ⟩) — directly consumes the anchor tokens in the data annotation;
  6) Shot Structure Response, SSR = S_cnt^0.35 · S_seg^0.65, where S_cnt = min(N,M)/max(N,M) (shot-count matching degree) and S_seg is the bidirectional temporal IoU (shot-cut-position matching degree) — tests whether the model follows the shot structure specified by the prompt.
[Human-alignment verification] Every video is scored by 10 independent evaluators using a 5-point Likert scale, double-blind and randomized in presentation order, with Spearman rank correlation used to verify the consistency between automatic metrics and human judgment.
[Overall judgment] CineBench is reverse-engineered around CineDance-1M's annotation schema, and the two are extremely tightly coupled — data-side designs such as anchor tokens, shot attributes, and ASR binding directly become the evaluation-side judgment basis. This is a standout feature of this work's "data-evaluation co-design," but it also means CineBench may not be entirely fair when applied to models trained on other datasets.

## Uncertain Fields

The research information for the following fields is partially uncertain (sources marked with ⚠️):

- organization
- openness
- data_scale
- data_sources
- provenance_licensing
- duration_distribution
- resolution_aspect_distribution
- domain_distribution
- audio_category_distribution
- language_accent_distribution
- quality_filtering
- motion_filtering
- deduplication
- safety_filtering
- caption_model
- dialogue_transcription_attributes
- sync_metric_and_threshold
- audio_quality_filtering
- multi_stage_curriculum
- stage_data_mixture
- post_training_data
- data_infra_throughput
- data_ablation
- quality_over_quantity_evidence
- human_in_loop
