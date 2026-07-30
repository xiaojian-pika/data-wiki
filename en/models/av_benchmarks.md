# Audio-Video Generation Evaluation Benchmark Collection (VABench / AVBench / AV-SyncBench / PhyAVBench / Omni-Judge)

> Topic: Data processing for video generation models (including joint audio-video generation): data cleaning pipeline, data distribution, annotation methods

[← Back to home](../index.md)

**Table of Contents**: [Basic Information](#basic-information) · [Data Scale and Distribution](#data-scale-and-distribution) · [Cleaning Pipeline](#cleaning-pipeline) · [Annotation Methods](#annotation-methods) · [Audio-Video Alignment](#audio-video-alignment) · [Training Support](#training-support) · [Comparative Results](#comparative-results)

## Basic Information

### Name

Audio-Video Generation Evaluation Benchmark Collection (VABench / AVBench / AV-SyncBench / PhyAVBench / Omni-Judge)

### Publishing Institution/Company

Multiple institutions; the five benchmarks belong to different teams:
【VABench】Peking University (Wentao Zhang's group, including Bohan Zeng, Hao Liang, Junbo Niu, etc.) + Ant Group (Quanqing Xu) + Institute of Automation, Chinese Academy of Sciences + Huazhong University of Science and Technology. First authors Daili Hua, Xizhi Wang.
【AVBench】Tsinghua University (Wenming Yang's group, first author Jialiang Yang, including Bin Xia, Ruihang Chu, etc.) + The Chinese University of Hong Kong (Dingdong Wang, etc.).
【AV-SyncBench】Alibaba Group (Jun Song, Bo Zheng, etc., corresponding author jsong.sj@alibaba-inc.com) + Tsinghua University (first author Tianhong Zhou, zth24@mails.tsinghua.edu.cn) + Fudan University.
【PhyAVBench】The Hong Kong University of Science and Technology (Guangzhou) HKUST(GZ) (corresponding author Li Liu, first author Tianxin Xie) + Tencent + Shanghai Jiao Tong University + Technical University of Munich (TUM); 29+ authors, 4 core contributors.
【Omni-Judge】University of Rochester (first author Susan Liang, corresponding author Chenliang Xu) + University of Michigan, Ann Arbor (Filippos Bellos, Jason J. Corso).

### Publication Date (technical report/paper/open-source release date) ⚠️

【VABench】arXiv:2512.09299, v1 submitted December 10, 2025, v2 updated April 6, 2026 (24 pages/25 figures, cs.CV + cs.SD, CC BY 4.0). The survey task describes it as a CVPR 2026 paper, but the arXiv comments field does not indicate acceptance information [uncertain].
【AVBench】arXiv:2605.24652, published May 2026; the project page's figure naming points to an ECCV submission [uncertain].
【AV-SyncBench】arXiv:2607.00726, published July 2026; already accepted by Interspeech 2026.
【PhyAVBench】arXiv:2512.23994, v1 published December 2025 (at that time a pure benchmark design report, with model evaluation left for later), subsequent versions filled in complete evaluation results for 17 models.
【Omni-Judge】arXiv:2602.01623, published February 2026.

### Type (model/dataset/toolchain/evaluation benchmark)

Evaluation benchmark. All five are evaluation systems for the Audio-Video Generation direction, but with different focuses:
- VABench: a comprehensive, all-dimension benchmark (three task types: T2AV / I2AV / stereo AV, seven content categories + 15 evaluation dimensions);
- AVBench: a human-aligned automated evaluation benchmark + a trainable dedicated evaluator (10 dimensions, accompanied by 300K preference training samples; the evaluator itself can be reused as a data filter and RLHF reward);
- AV-SyncBench: a dedicated synchronization benchmark (temporal synchronization and semantic synchronization decoupled), also a dataset with perturbation annotations;
- PhyAVBench: a dedicated physical-commonsense benchmark, accompanied by a self-recorded real dataset PhyAV-Sound-11K (11,605 videos / 25.5 hours);
- Omni-Judge: an evaluation methodology study (investigating whether Omni-LLMs can serve as human-aligned judges), belonging to meta-evaluation.
Among these, AVBench, AV-SyncBench, and PhyAVBench also have "dataset" attributes.

### Degree of Open-Sourcing (whether weights/code/data/pipeline are each open-sourced) ⚠️

【VABench】Paper under CC BY 4.0; code repository https://github.com/tanABCC/VABench; no explicit dataset license stated. Prompts, VQA/AQA question-answer pairs, and evaluation scripts are the main open-source outputs; generated videos rely on each provider's own API for reproduction.
【AVBench】Highest degree of openness: GitHub https://github.com/YaJialiang/AVBench, evaluator weights released on HuggingFace (iiiiii123/AVBench_model), and hosts a HuggingFace Leaderboard (spaces/iiiiii123/AVBenchLB). Data, code, and model are all released; specific license not stated [uncertain].
【AV-SyncBench】The dataset is live on ModelScope (coming245/AVSyncBench) and HuggingFace (coming245/AV-SyncBench), code repository https://github.com/fgt7t6g/AV-SyncBench (as of the survey time, the README marks the evaluation code as coming soon); the paper uses the arXiv perpetual non-exclusive license.
【PhyAVBench】Project pages https://imxtx.github.io/PhyAVBench/ and https://phyavbench.pages.dev/; publicly releases prompts, self-recorded ground-truth videos, and generated samples from each model, with a commitment of zero overlap with training sets; paper under an arXiv license.
【Omni-Judge】Only a project page liangsusan-git.github.io/project/omni_judge/; the paper does not explicitly state code/data open-sourcing [uncertain].

### Whether Joint Audio-Video Generation Is Supported, and the Implementation Approach (native joint generation/cascaded/MoE fusion)

None of the five benchmarks themselves generate audio-video content; rather, they evaluate joint audio-video generation capability. The systems under test that they cover happen to constitute the three current technical routes of AV generation, and the evaluation design explicitly distinguishes them:
1) Native joint generation (end-to-end T2AV/I2AV): Sora 2, Veo 3 / Veo 3.1 / Veo3-Fast, Wan 2.5 Preview / Wan 2.6, Kling 2.5 Turbo / Kling v2.6, Seedance 1.5 Pro; on the open-source side Ovi, LTX, MOVA, UniVerse-1, JavisDiT / JavisDiT++;
2) Cascaded composition (V+A, video generated first, then dubbed): video side Seedance-1.0-Lite / Wan2.2-TI2V / Kling2.5 Turbo, audio side MMAudio, ThinkSound(-Light), HunyuanVideo-Foley, FoleyCrafter;
3) Representation/discriminative models (the objects under test in AV-SyncBench): Synchformer, SparseSync, ImageBind, CAV-MAE, CAV-MAE-Sync.
VABench additionally introduces the "stereo audio-video generation" route, using 116 prompts that explicitly specify left/right channel positioning to examine spatial audio generation capability — this is currently a rare stereo AV evaluation dimension.

### List of Research Information Sources (URLs of papers/technical reports/official documentation/news, with the nature of each source noted: official primary source/same-team corroboration/third-party report)

- https://arxiv.org/abs/2512.09299 —— VABench: A Comprehensive Benchmark for Audio-Video Generation, arXiv abstract page (official primary source). Authors Daili Hua, Xizhi Wang, Bohan Zeng, Xinyi Huang, Hao Liang, Junbo Niu, Xinlong Chen, Quanqing Xu, Wentao Zhang; v1 2025-12-10, v2 2026-04-06; 24 pages, 25 figures; cs.CV + cs.SD; CC BY 4.0.
- https://arxiv.org/html/2512.09299v1 —— VABench paper full HTML text (official primary source). Extracted the seven-category system, 15 evaluation dimensions and two major modules, the dual-path T2AV/I2AV data curation pipeline, sample sizes of 778/521/116, the 9 stereo acoustic metrics, the list of tested models, and author institutions (Peking University / Ant Group / Institute of Automation, CAS / Huazhong University of Science and Technology).
- https://github.com/tanABCC/VABench —— VABench official code repository (official primary source).
- https://arxiv.org/abs/2605.24652 —— AVBench: Human-Aligned and Automated Evaluation Benchmark for Audio-Video Generative Models, arXiv abstract page (official primary source). Authors Jialiang Yang, Bin Xia, Ruihang Chu, Dingdong Wang, Wanke Xia, Zhun Mou, Tianyang Zhong, Yiting Zhao, Wenming Yang.
- https://arxiv.org/html/2605.24652v1 —— AVBench paper full HTML text (official primary source). Extracted the 10 evaluation dimensions, 470 evaluation prompts with Normal/Hard stratification, the 30K→300K hard-negative synthesis recipe, the Qwen2.5-Omni 7B and Qwen2-Audio 7B evaluator architectures, the 4-expert 2AFC annotation protocol, the discussion of usability as a data filter and RLHF reward, and author institutions (Tsinghua University / The Chinese University of Hong Kong).
- https://yajialiang.github.io/AVBench-site/ —— AVBench official project page (official primary source). Contains links to GitHub, HuggingFace models, and the Leaderboard.
- https://github.com/YaJialiang/AVBench —— AVBench official code repository (official primary source).
- https://huggingface.co/iiiiii123/AVBench_model —— AVBench evaluator weights (official primary source).
- https://huggingface.co/spaces/iiiiii123/AVBenchLB —— AVBench online leaderboard (official primary source).
- https://arxiv.org/abs/2607.00726 —— AV-SyncBench: Decoupled Benchmarking of Temporal and Semantic Audio-Visual Synchronization, arXiv abstract page (official primary source). Authors Tianhong Zhou, Mingyang Han, Boyu Li, Yuxuan Jiang, Jiaxin Ye, Dongxiao Wang, Haoxiang Shi, Kunpeng Wang, Jun Song, Cheng Yu, Bo Zheng; already accepted by Interspeech 2026; 3,269 videos / 38,390 samples.
- https://arxiv.org/html/2607.00726v1 —— AV-SyncBench paper full HTML text (official primary source). Extracted the list of 10 scenes and 5 challenge tasks, the 0.64-second chunk diagonal similarity evaluation protocol, the Gemini 3 Flash automatic filtering + 5-annotator cross-review process with ≥3 people, the perturbation parameter spectrum (offset 50–500ms in five levels / jitter 30–700ms in three levels / speed change 0.8×–1.25× in ten levels), the OpenVoice V2 and DDSP semantic perturbation tools, the empirical results for 5 baseline models, and author institutions (Alibaba Group / Tsinghua University / Fudan University).
- https://fgt7t6g.github.io/AV-SyncBench —— AV-SyncBench official project page (official primary source). Contains links to ModelScope / HuggingFace / GitHub datasets and code.
- https://github.com/fgt7t6g/AV-SyncBench —— AV-SyncBench official code repository (official primary source); at the time of the survey the evaluation code was marked coming soon.
- https://huggingface.co/datasets/coming245/AV-SyncBench —— AV-SyncBench dataset (official primary source).
- https://modelscope.cn/datasets/coming245/AVSyncBench —— AV-SyncBench dataset ModelScope mirror (official primary source).
- https://arxiv.org/abs/2512.23994 —— PhyAVBench: A Challenging Audio Physics-Sensitivity Benchmark for Physically Grounded Text-to-Audio-Video Generation, arXiv abstract page (official primary source). First author Tianxin Xie, 29+ co-authors; 25.5 hours / 11,605 videos / 337 paired prompt sets / 6 dimensions with 41 test points / evaluation of 17 models.
- https://arxiv.org/html/2512.23994v1 —— PhyAVBench v1 paper HTML (official primary source). Extracted the five-stage curation pipeline, the details of physical dimensions and test points, the definitions of the CPRS and FGAS metrics, and author institutions (HKUST(GZ) / Tencent / Shanghai Jiao Tong University / TU Munich, corresponding author Li Liu). Note that v1 is a pure benchmark design report (describing 1,000 prompt sets, 50 test points, with model evaluation left for later), which differs from the 337-set / 41-test-point data of subsequent versions; the body text follows the subsequent versions.
- https://arxiv.org/html/2512.23994v3 —— PhyAVBench latest version paper HTML (official primary source). Extracted the PhyAV-Sound-11K dataset specifications (11,605 clips / 25.5 hours / 184 participants / average 17 GT clips per set), CPRS switching to CLAP embeddings, the 74-rater PVR-MOS, the complete list of 17 evaluated models with results such as Sora 2's CPRS of 0.4512, and the CPRS-to-human-judgment Pearson correlation of 0.92.
- https://imxtx.github.io/PhyAVBench/ —— PhyAVBench official project page (official primary source).
- https://phyavbench.pages.dev/ —— PhyAVBench official project page mirror (official primary source).
- https://arxiv.org/abs/2602.01623 —— Omni-Judge: Can Omni-LLMs Serve as Human-Aligned Judges for Text-Conditioned Audio-Video Generation?, arXiv abstract page (official primary source). First author Susan Liang, with 9 co-authors including Jason J. Corso, Chenliang Xu, etc.
- https://arxiv.org/html/2602.01623v1 —— Omni-Judge paper full HTML text (official primary source). Extracted the Qwen3-Omni (30B/3B activated) judge model, 9 evaluation dimensions, 300 VidProM prompts + 600 videos each generated by Sora 2/Veo 3, the 6-PhD-student 1–5 scale annotation protocol, per-dimension Kendall τ_b and Spearman ρ correlation results, and author institutions (University of Rochester / University of Michigan, Ann Arbor).
- https://liangsusan-git.github.io/project/omni_judge/ —— Omni-Judge official project page (official primary source).

## Data Scale and Distribution

### Training Data Volume (video count/hours/token count, pretraining and SFT separated)

All five are at evaluation/meta-evaluation scale, far smaller than training sets, but AVBench and PhyAVBench come with substantial data assets:
【VABench】1,299+ test cases in total: 778 T2AV prompts, 521 I2AV prompts (with paired images), of which 116 are dedicated to stereo. Each sample is accompanied by 3–7 audio question-answer pairs (AQA) and 3–7 visual question-answer pairs (VQA).
【AVBench】Evaluation set: 470 prompts at ≥720p HD (Normal subset 350, 1–2 speakers; Hard subset 120, 3–4 speakers + overlapping speech + noisy background). The evaluator training set is significantly larger: 30K real clips extracted from OpenHumanVid, expanded per dimension to 100K pairs each, totaling 300K supervised samples across the three consistency dimensions (all preference pairs with hard negatives).
【AV-SyncBench】3,269 in-the-wild videos, expanded to 38,390 evaluation samples (temporal challenge 37,569 samples / 2,717 videos; semantic challenge 821 samples / 552 videos; of which human voice timbre replacement 592, instrument timbre transfer 229).
【PhyAVBench】Dataset PhyAV-Sound-11K: 11,605 newly recorded videos, totaling 25.5 hours; 337 controlled paired prompt sets; 184 participants appearing on camera/performing actions; each paired prompt set is accompanied by an average of about 17 ground-truth real recorded videos (the paper requires N≥20 for mean noise reduction, actual average is 17).
【Omni-Judge】300 prompts (taken from the VidProM real-user prompt library), with 1 video each generated by Sora 2 and Veo 3, totaling 600 generated videos, accompanied by 600×9-dimension human ratings.

### Data Source Composition (proprietary/public datasets/web scraping/licensed acquisition/synthetic data)

【VABench】The T2AV side consists of prompts synthesized in bulk by LLM + expert templates (no real video source); the I2AV side consists of manually curated, high-quality images that have undergone privacy screening, from which a multimodal LLM generates a unified audio-visual description. This constitutes "synthetic prompts + manually curated images."
【AVBench】Evaluation prompts come from an independent prompt pool (≥720p HD real human scenes); evaluator training data comes from the public dataset OpenHumanVid (real human videos), with negatives synthesized via LLM-driven perturbation and algorithmic mismatching — i.e., "real positives from a public dataset + programmatically synthesized hard negatives."
【AV-SyncBench】All videos scraped from public web platforms in the wild (specific platforms not specified); perturbed samples are synthesized by algorithms and speech/instrument conversion models.
【PhyAVBench】The most distinctive — all videos are "newly recorded or captured," explicitly aimed at avoiding data leakage (zero training-set overlap), filmed in a controlled environment, and sampled across different individuals, performers, and recording devices for diversity. No existing public datasets are used.
【Omni-Judge】Prompts come from VidProM (a real-user prompt gallery), videos are generated live by Sora 2 and Veo 3.

### Data Compliance and Provenance (proportion of licensed data, rights-cleared datasets, C2PA, etc.) ⚠️

Overall disclosure is weak; only PhyAVBench has a systematic design:
【PhyAVBench】By fundamentally "recording everything in-house + guaranteeing zero training-set overlap," it avoids copyright and data leakage issues at the root, making it the cleanest in provenance among the five; it involves 184 on-camera participants, but the paper does not disclose the specific process for portrait licensing and informed consent [uncertain].
【VABench】I2AV images explicitly went through privacy screening before inclusion; the copyright source of the images is not disclosed.
【AV-SyncBench】Data comes from public web platforms; the paper does not discuss copyright licensing or provenance mechanisms such as C2PA [uncertain].
【AVBench】Training data is based on the public dataset OpenHumanVid, inheriting its license; the source of the evaluation prompt pool is not detailed.
【Omni-Judge】Uses VidProM prompts (this dataset itself has a CC BY-NC type license); copyright of generated videos is governed by the terms of service of each commercial model.
None of the five mentions C2PA / content watermarking provenance standards.

### Clip Duration Distribution and Segmentation Strategy ⚠️

【VABench】The duration of tested videos is constrained by each model's default settings rather than mandated by the benchmark: Sora2 is 10 seconds @30FPS; Veo3-fast, Wan2.5, Seedance, Wan2.2, Kling are 5–8 seconds @24FPS. Synchformer's Desync metric is computed only over the first and last 4.8-second windows — this 4.8s window setting is the key segmentation strategy for its temporal evaluation.
【AV-SyncBench】The most fine-grained segmentation strategy: clip duration 3–13 seconds; during evaluation, video and audio are uniformly cut into non-overlapping 0.64-second chunks, and the diagonal similarity mean is taken after extracting audio-visual embeddings chunk by chunk — 0.64s is the basic unit of its temporal resolution.
【PhyAVBench】11,605 videos / 25.5 hours, averaging about 7.9 seconds per clip.
【AVBench】The generation duration corresponding to evaluation prompts is not separately specified [uncertain]; the segmentation strategy for the OpenHumanVid clips used in training is not disclosed [uncertain].
【Omni-Judge】Default output duration of Sora 2 / Veo 3 (about 8–10 seconds) [uncertain].

### Resolution/Aspect Ratio Distribution and Bucketing Strategy ⚠️

【VABench】Uniformly requires 720P (or the closest aspect-ratio tier); audio uniformly extracted and retained at 48kHz stereo; frame rate follows each model's default of 24–30 FPS, without forced normalization.
【AVBench】Evaluation prompts explicitly require ≥720p HD (HD prompts), which is the prerequisite threshold for its "real human scene" positioning.
【AV-SyncBench】Normalized before evaluation: video decoded at 25 FPS, audio resampled to 16 kHz; the original resolution and aspect-ratio distribution are not disclosed [uncertain].
【PhyAVBench】The specific distribution of recording resolution and device models is not given in the paper [uncertain]; it only states that data was captured across multiple recording devices to increase diversity.
【Omni-Judge】Follows the default output specifications of Sora 2 / Veo 3 [uncertain].
A bucketing insight that can be borrowed in reverse: AV evaluation generally converges on two spec sets — 720P + 5–10s + 48kHz (on the generation side) / 16kHz + 25FPS (on the discriminative side) — training data bucketing can align to these two anchor points.

### Category/Domain Distribution and Ratio Strategy (proportional control and concept balancing across people, actions, scenes, styles, etc.) ⚠️

This item is the field of the most reverse-guidance value to training data in this survey — the category systems of the five benchmarks can directly serve as a standard coordinate system for training data domain ratios.

【VABench's seven-category system (the most complete content-side taxonomy)】
1. Animals: species-specific vocalization and audio-visual behavioral consistency;
2. Human Sounds: subdivided into linguistic (containing semantic content, involving lip sync) and non-linguistic (physiological/action-based, such as coughing, clapping, footsteps);
3. Music: structured audio across genres, examining melodic and rhythmic coherence;
4. Environmental Sounds: further divided into natural, urban, and indoor soundscapes;
5. Synchronous Physical Sounds: instantaneous/rhythmic physical interaction sounds, requiring strict adherence to material properties;
6. Complex Scenes: higher-order scenes, containing five sub-dimensions — complex soundscapes, subjective feelings, world knowledge, symbolic association, off-screen (invisible) sound sources;
7. Virtual Worlds: unrealistic scenes that transcend physical laws, used only for the T2AV task.
Data volume allocation: 778 T2AV + 521 I2AV distributed according to the above category hierarchy (the paper presents this via a sunburst chart, without giving exact counts per category [uncertain]). Note that category 7, "Virtual Worlds," is excluded from the two physical-plausibility scoring items, Audio Realism and Visual Realism — this design of "separating realistic categories from stylized categories in scoring" is likewise applicable to setting domain-specific quality gates for training data.

【AV-SyncBench's ten-scene system (an orthogonal partition from the synchronization perspective)】
Action, Animal Sound, Object Sound, Ambient Sound, Group Vocalization, Single Speaker, Dialogue, Singing, Single Instrument, Ensemble. These are grouped upward into three major audio classes: Voice / Music / Sound. The value of this system lies in partitioning by "synchronization difficulty" rather than "content topic": the distinction between single speaker and dialogue, single instrument and ensemble, corresponds directly to the ratio needs for multi-source overlapping scenes in training data.

【PhyAVBench's six physical dimensions + 41 fine-grained test points (a physical-correctness perspective)】
① Sound source mechanics (material hardness/damping, density, surface texture, object geometric size/shape/thickness, contact dynamics such as impact velocity/contact area/excitation continuity); ② Fluid and aerodynamics (flow velocity, liquid impact/splashing, Helmholtz resonance, container material, viscosity, bubbles, aerodynamic whistling); ③ Sound propagation environment (reverberation as a function of spatial volume and surface absorption, echo, spatial transitions, diffraction, scattering, underwater acoustics, solid-borne sound transmission, vacuum, occlusion and sound insulation); ④ Observer physics (inverse-square law with distance, air absorption, the Doppler effect for approaching/receding and rotating sound sources, binaural horizontal/vertical localization); ⑤ Time and causality (far-field delay and near-field synchronization caused by the speed difference between light and sound, transient synchronization of onset and offset, rhythmic consistency of periodic/aperiodic motion); ⑥ Complex coupling and extreme physics (phase changes such as boiling/freezing and fracturing, supersonic and nonlinear distortion in explosions and shockwaves). This system can directly serve as a checklist for the "physical acoustic coverage" of training data.

【AVBench's scene stratification】Centered on real human scenes, stratified by number of speakers and acoustic difficulty: Normal (1–2 speakers) 350 items / Hard (3–4 speakers + overlapping speech + noisy background) 120 items, with a Hard Quota-Based Greedy Sampling constraint applied, forcing any single attribute's proportion to be ≤50% — this is an explicit concept-balancing mechanism, directly transplantable as an attribute-quota control strategy for sampling training data.

【Omni-Judge】No content categories are set; its 300 prompts are taken directly from the real VidProM user distribution, representing an alternative domain benchmark of "real user intent distribution."

### Audio Category Distribution and Ratio (proportion and control strategy for speech/foley/music/environmental sound/silence — a dimension unique to AV models) ⚠️

On this AV-unique dimension, the five benchmarks provide three complementary audio category divisions:

【AV-SyncBench's tripartite division】Voice / Music / Sound is the simplest and most operable top-level audio classification. Its mapping to the ten scenes is: Voice ← Single Speaker, Dialogue, Group Vocalization, Singing; Music ← Single Instrument, Ensemble, Singing (cross-category); Sound ← Action, Animal Sound, Object Sound, Ambient Sound. The semantic challenge tasks are also routed by this division: human voice goes through timbre replacement (OpenVoice V2), instruments go through timbre transfer (pretrained DDSP) — indicating that the semantic perturbation mechanisms for the two audio types are fundamentally different, and semantic-alignment filtering of training data should likewise be routed separately. In terms of sample volume, the temporal challenge (37,569) far exceeds the semantic challenge (821), reflecting that semantically mismatched samples are harder to construct in real data.

【VABench's content-driven division】Animal sounds, human sounds (linguistic/non-linguistic), music, environmental sounds (natural/urban/indoor), synchronous physical sounds, complex soundscapes (including off-screen sound sources), virtual sound effects. Finer than the tripartite division, and it explicitly introduces the category of "off-screen/invisible sound sources" — training data usually has such samples mistakenly killed by on-screen filters, but the evaluation system acknowledges their existence, suggesting the training side should retain a certain proportion of off-screen sound samples to support complex-soundscape generation.

【PhyAVBench's physics-oriented Foley division】Classified by sound-production mechanism rather than content topic (solid impact/mechanical structure/fluid aerodynamics/propagation environment/observer position/temporal causality), essentially a deep expansion of the Foley sound-effect category.

【AVBench】Speech is the absolute focus (3 of the 10 dimensions — Speech Content Accuracy, Speech Realism, Lip Sync — directly target speech), corresponding to the OpenHumanVid human-video data source; music and sound-effect categories are basically not covered.

None of the five gives precise percentage ratios for their respective audio categories [uncertain], but the union of categories (speech / music / physical sound effects Foley / environmental soundscape / animal sounds / off-screen sound / silence and virtual sound) can directly serve as row labels for a training-data audio-domain ratio table.

### Narrative Structure Distribution (single-shot vs. multi-shot, average clip duration, shot-count distribution, whether native audio tracks are included)

All five focus on single-shot short-clip evaluation, without constructing multi-shot narrative evaluation:
【VABench】5–10 second single-shot generation, but the "Complex Scenes" category indirectly examines narrative expressiveness through sub-dimensions such as complex soundscapes, world knowledge, and symbolic association; the Expressiveness (narrative effectiveness and emotional alignment) and Artistry (aesthetic quality of audio-visual fusion) items in Module 2 are the only narrative-level metrics.
【AV-SyncBench】3–13 second in-the-wild clips, all containing native audio tracks (this is a prerequisite for inclusion), predominantly single-shot; Dialogue and Ensemble scenes implicitly contain multiple sound sources but not multiple shots.
【PhyAVBench】Controlled, recorded single-shot short clips, all with native synchronized sound.
【AVBench】Real human-scene single shots; the Hard subset's 3–4 overlapping speakers is its setting closest to multi-subject narrative.
【Omni-Judge】Sora 2 / Veo 3 default single-prompt generation, not involving multiple shots.
Overall, long-form, multi-shot, and cross-shot audio-track continuity remain blank areas in current AV evaluation systems; if the training data side is already producing multi-shot narrative data, there is currently no corresponding benchmark to validate against.

### Language/Accent Distribution (the data foundation for multilingual lip-sync capability) ⚠️

Disclosure is generally weak, a common shortcoming across all five benchmarks:
【AVBench】Lip sync and Speech Content Accuracy are core dimensions, but the paper does not disclose the language and accent composition of the evaluation prompts and OpenHumanVid training clips [uncertain].
【VABench】The lip-sync metric is applied only to samples in the "human sounds - linguistic" subset where a speaking head is detected; multilingual coverage is not disclosed [uncertain].
【AV-SyncBench】For human-voice scenes (single speaker/dialogue/group vocalization/singing), semantic perturbation uses OpenVoice V2 for timbre replacement (this tool itself supports cross-lingual timbre cloning), but the benchmark does not report statistics stratified by language/accent [uncertain].
【PhyAVBench】Dominated by non-speech physical sound effects; speech is only indirectly involved via the WER metric (Whisper-Large V3); the language background of the 184 participants is not disclosed [uncertain].
【Omni-Judge】Does not involve a language dimension [uncertain].
Conclusion: there is currently no public benchmark to reference for multilingual/multi-accent lip sync, so the language ratio of training data cannot be reverse-validated with existing benchmarks.

## Cleaning Pipeline

### Overall Filtering Funnel Structure (number of filtering stages, order of stages)

The "cleaning pipeline" of the five benchmarks is really an evaluation-data curation process, but their funnel structure has direct reference value for training-data pipelines:

【PhyAVBench's five-stage curation pipeline (most complete)】① Audio physics knowledge survey — LLM brainstorms candidate physical principles, human experts eliminate infeasible/redundant/irrelevant entries; ② Taxonomy construction — LLM generates a hierarchical structure, experts review to disambiguate and de-duplicate; ③ Physically constrained prompt-pair design — LLM generates candidate templates, experts manually verify and rewrite to ensure each prompt pair differs in only a single physical variable while all other non-target conditions remain unchanged, avoiding subjective descriptions and explicit hints about the expected acoustic outcome; ④ Real audio-video capture — newly recorded, in a controlled environment, sampled diversely across individuals/performers/devices; ⑤ Iterative quality control and filtering — LLM does an initial pass for semantic ambiguity and unintended physical confounds, humans review physical consistency, three-way text-audio-video alignment, and realism; problematic samples are deleted, revised, or re-captured. This is a typical closed loop of "LLM generation → expert revision → capture → LLM pre-screening + human review → iterative feedback."

【AV-SyncBench's three-tier funnel】① Web collection of in-the-wild videos; ② Gemini 3 Flash automatic filtering — eliminating samples with off-screen sound sources and clearly mismatched audio-visual content; ③ Human review — 5 annotators, each clip independently reviewed by at least 3 people, confirming the primary sound source is visible on screen, and eliminating clips with poor audio quality, excessive noise, or semantic ambiguity. This is followed by a fourth step: programmatically generating temporal and semantic negative samples through perturbation. This is a standard example of "large-model pre-screening + multi-person cross-review human verification" in this survey.

【VABench's dual-path curation】T2AV path: LLM + expert templates batch-generate raw prompts → an LLM structurally decouples them into visual sub-prompts and auditory sub-prompts → generates VQA/AQA question-answer pairs → humans verify category correctness, element observability, and satisfaction of physical/commonsense constraints. I2AV path: humans curate and categorize high-quality images (including privacy screening) → a multimodal LLM generates a unified audio-visual description (visual objective + audio commonsense inference) → the description is used to construct VQA/AQA and decoupled by an LLM into sub-prompts → humans review the plausibility of the auditory inference and the discriminability of the questions. The paper explicitly states that it "employed human workers and large language models to filter testing samples and adjust the distribution of test data" — i.e., human-machine collaboration jointly handles both filtering and distribution adjustment.

【AVBench's two pipelines】Evaluation-set path: sample 470 ≥720p prompts from the prompt pool via Hard Quota-Based Greedy Sampling, with a quota constraint that any single attribute is ≤50%, then stratify into Normal/Hard subsets. Training-set path: extract 30K real clips from OpenHumanVid as positives → LLM-driven perturbation + algorithmic mismatching generates hard negatives → expand to 100K pairs per dimension → 300K total across three dimensions.

【Omni-Judge】No data-cleaning pipeline; a meta-evaluation: 300 VidProM prompts → 1 video each generated by Sora 2 / Veo 3 → 6 PhD students score on 9 dimensions → compute the correlation between Omni-LLM scoring and human scoring.

### Funnel Quantitative Retention Rate (input/output volume for each filtering stage and the final retention rate, e.g., Apollo's 27%) ⚠️

None of the five benchmarks publishes input/output volume or retention-rate figures for each filtering stage [uncertain]. The only indirectly derivable figures are:
【AV-SyncBench】Final retention of 3,269 videos, but the size of the original web-collected pool is not disclosed, so the retention rates of the two stages — Gemini 3 Flash pre-screening and 5-person review — cannot be calculated [uncertain].
【PhyAVBench】11,605 videos correspond to 337 paired prompt sets, averaging about 17 ground-truth videos per set; the paper's design target was N≥20 per set, and the actual average of 17 indicates noticeable attrition at the quality-control stage (a rough estimated retention rate of about 85%, an inference rather than a figure disclosed in the paper [uncertain]).
【AVBench】30K real clips → 300K training pairs is expansion rather than filtering, with no concept of retention rate; the size of the original candidate pool for the 470 evaluation prompts is not disclosed [uncertain].
【VABench】The candidate generation volume corresponding to the final 1,299 samples is not disclosed [uncertain].
Compared to training-side datasets (e.g., Apollo publishing a 27% end-to-end retention rate, MOVA publishing a stage-by-stage retention table), evaluation-benchmark papers generally do not disclose quantitative funnel information — this is a common shortcoming of this class of literature.

### Shot Segmentation Method (PySceneDetect/in-house model/shot-aware splitting) ⚠️

None of the five involves shot segmentation in the traditional sense (of the PySceneDetect variety), because the source material is itself already short single-shot clips. The related timeline segmentation strategies are:
【AV-SyncBench】Uniformly cut into non-overlapping 0.64-second chunks for chunk-by-chunk audio-visual embedding alignment computation — this is the basic granularity of its temporal evaluation (corresponding to the native window setting of models like CAV-MAE / Synchformer).
【VABench】The Desync metric only feeds the first 4.8 seconds and last 4.8 seconds of the generated video into Synchformer to predict the offset, avoiding the instability of long mid-sequence segments.
【PhyAVBench】During recording, segmentation is by "a single physical event" — each video corresponds to one complete sound-producing event (such as one impact, one stretch of running water) — this is a semantic-event-level segmentation rather than a shot-level one.
【AVBench / Omni-Judge】Not involved [uncertain].

### Quality Filtering (aesthetic score, sharpness, OCR text filtering, black-bar/watermark/logo detection) ⚠️

The quality-control methods of each benchmark:
【AVBench】The quality dimensions themselves are reusable filters; its 6 unimodal quality dimensions are each bound to mature tools: audio quality via NISQA MOS, audio aesthetics via Audiobox, video technical quality via DOVER++, video aesthetics as a separate dimension, plus Speech Content Accuracy and Speech Realism as two speech-specific items. The paper explicitly states that its continuous differentiable scores can be directly used as a data filtering mechanism and RLHF reward signal — this is the most directly transplantable-to-training-data quality filter benchmark in this survey.
【VABench】Module 1's unimodal audio quality triple: SpeechClarity uses DNSMOS to detect background noise, SpeechQual&Nat uses NISQAv2 to rate overall quality and naturalness, AudioAesthetic uses Audiobox to rate pleasantness/usefulness/production complexity and quality. Visual-side quality is folded into the MLLM-scored Visual Realism and Artistry. In addition, the stereo module provides 9 acoustic quality metrics: spatial imaging quality (stereo width, imaging stability, level stability, inter-channel consistency) and signal integrity/compatibility (phase coherence scored across low/mid/high frequency bands, mono compatibility Mono Compat = 1 − normalized mono loss, directional consistency, Mid/Side energy ratio measuring soundstage width).
【AV-SyncBench】The manual stage explicitly eliminates three types of samples: "poor audio quality, excessive noise, semantic ambiguity" — i.e., a triple threshold of audio SNR, clarity, and semantic discriminability (no quantitative thresholds given [uncertain]).
【PhyAVBench】Quality control focuses on physical correctness rather than visual aesthetics: LLM does initial screening of semantic ambiguity and unintended physical confounds, humans review physical consistency and realism; since the data is self-recorded in a controlled environment, issues like poor sharpness, watermarks, and black bars naturally do not exist.
【Omni-Judge】No filtering; but its conclusions carry a cautionary implication for quality filtering — Omni-LLM has extremely low correlation with humans on the video quality dimension (Kendall τ_b ≈ 0.020), indicating that directly substituting Omni-LLM for traditional aesthetic/technical quality scorers in data filtering is unreliable; image-quality filtering still requires dedicated models.
None mention common training-side techniques such as OCR text filtering or watermark/logo detection [uncertain].

### Motion Filtering (optical-flow/motion-score thresholds, static and jitter removal) ⚠️

None of the five uses optical flow or motion-score thresholds for filtering [uncertain], but several functionally equivalent alternative mechanisms exist:
【AV-SyncBench】The Local Jitter challenge task (local jitter perturbation at three severity levels, 30–700 ms) and Global Speed Change (0.8×–1.25×, 10 levels in total) are essentially manufactured negative samples with "motion-audio time-base inconsistency," which can be used in reverse as a detection idea for speed-change/frame-drop anomalies in training data.
【PhyAVBench】Under the Time and Causality dimension, the test points "periodic/aperiodic motion rhythm consistency" and Transient Synchronization (onset and offset) examine precisely the causal consistency between motion and sound, which is more semantic than a plain motion-magnitude threshold.
【VABench】No motion filtering is applied; static-image issues in the tested models on the generation side are indirectly penalized by the Visual Realism and Artistry scores.

### Deduplication Method (exact deduplication and embedding-based semantic deduplication recorded separately) ⚠️

None of the five discloses an explicit exact-deduplication or embedding-based semantic-deduplication pipeline [uncertain]. Related approximate mechanisms are:
【PhyAVBench】Adopts a "reverse deduplication" approach — proactively guaranteeing zero overlap with existing training sets (all newly recorded), and deliberately sampling across different individuals, performers, and recording devices during capture to increase intra-sample diversity — avoiding homogeneity from the source rather than after the fact.
【AVBench】Hard Quota-Based Greedy Sampling forcing any single attribute's proportion to be ≤50% is functionally equivalent to attribute-level de-redundancy/balanced sampling.
【AV-SyncBench / VABench / Omni-Judge】Deduplication not mentioned [uncertain].

### VLM/LLM as Quality Inspector (multimodal LLM quality scoring and mismatch removal; the 2026 trend of moving from shallow scorers to large-model semantic judgment)

This is the field of greatest training-side reference value among the five benchmarks in this survey; together they constitute a complete chain of evidence for the 2026 trend of "large models as quality inspectors":

【AV-SyncBench — empirical evidence of Gemini as a front-line quality inspector】Explicitly uses Gemini 3 Flash as the first stage of automatic filtering, tasked with removing two types of samples: "off-screen sound sources" and "clearly mismatched audio-visual content," before handing off to human review. This is a rarely seen practice in the public literature of embedding a commercial closed-source multimodal LLM directly into the first-tier filtering of a data pipeline; its cost-effectiveness tradeoff (using a Flash-tier lightweight model for high-recall coarse screening, then using human labor for high-precision fine screening) is directly transplantable to large-scale training-data cleaning.

【AVBench — a dedicated trained evaluator replacing a general-purpose large model】Takes a different route: instead of using a general-purpose MLLM to score directly, it trains a dedicated evaluator via preference learning. The VT and AV dimensions are based on Qwen2.5-Omni (7B), fine-tuning only the LLM part; the AT dimension is based on Qwen2-Audio (7B), fine-tuning both the LLM and the connector layer. The training constraint design is clever: the model only outputs a single token (Yes/No), and a continuous score is obtained by normalizing the token probability ratio, converting a discrete judgment into a continuous differentiable signal that can be directly used for data filtering and RLHF reward. This solves the core pain point of LLM-as-judge scoring — discreteness, high variance, non-differentiability — and is the most reusable technical detail on the training side.

【VABench — general-purpose MLLM for semantic-level evaluation】Module 2 uses Qwen2.5-Omni 7B to handle 5 macro scoring items (Alignment / Artistry / Expressiveness / Audio Realism / Visual Realism, 1–5 scale) and 2 micro question-answering items (3–7 audio QA and 3–7 visual QA per sample). The "macro scoring + micro QA" two-layer structure is worth borrowing: fine-grained QA breaks down an ambiguous overall score into verifiable factual judgments, significantly reducing subjective drift in MLLM scoring.

【Omni-Judge — systematic mapping of capability boundaries】The most important source of negative findings. Using Qwen3-Omni (30B total parameters / 3B activated) as the subject, comparing the instruct version and the reasoning-enhanced version, correlation analysis was run against 6 PhD student annotators across 9 dimensions. The conclusion is clearly stratified: semantic dimensions are usable — audio-text alignment reaches τ_b=0.292 / ρ=0.345 on the Sora 2 subset, audio-video-text tri-modal consistency 0.139/0.151; perceptual dimensions are not usable — video quality τ_b≈0.020, audio-video synchronization only 0.142, with the authors attributing this to Omni-LLM's insufficient temporal resolution. Direct guidance for training-data pipelines: Omni-LLM is suited to handling "semantic matching/mismatch removal," but temporal synchronization and image-quality judgment must be handed to dedicated models like Synchformer and DOVER++. The paper also demonstrates an application of using Omni-Judge's interpretable feedback for "feedback-driven correction" of generative models (correcting frames based on identified errors in generation), but does not advocate its use for training-data filtering.

【PhyAVBench — LLM involvement at both the knowledge-construction and pre-screening ends】LLMs are used for physics knowledge brainstorming, taxonomy generation, prompt-template generation, and initial screening of semantic ambiguity and confounding factors during quality control; but all LLM outputs are reviewed and revised by human experts, forming a strict division of labor of "LLM for efficiency + expert gatekeeping."

Overall judgment: the 2026 consensus is a three-tier division of labor — "large models judge semantics, dedicated models judge perception, humans judge physics and final boundary cases" — rather than using a large model to do everything.

### Safety and Compliance Filtering (NSFW, copyright, face/privacy) ⚠️

Limited disclosure:
【VABench】High-quality images on the I2AV path underwent privacy screening during manual curation and categorization — the only one of the five to explicitly mention privacy filtering.
【AV-SyncBench】The manual stage eliminates semantically ambiguous samples; NSFW/copyright/face-privacy specific filtering is not mentioned [uncertain].
【PhyAVBench】Self-recorded data involves 184 on-camera participants; the process for portrait rights and informed consent is not disclosed [uncertain]; since the content consists of controlled physical demonstrations, it naturally avoids NSFW risk.
【AVBench】Based on the public dataset OpenHumanVid, inheriting its safety filtering; no additional safety-filtering description of its own [uncertain].
【Omni-Judge】Generated using commercial model APIs; safety filtering is handled by the built-in content safety mechanisms of Sora 2 / Veo 3.
None of the five mentions specific tools or thresholds for NSFW classifiers, copyright detection, or face anonymization [uncertain].

## Annotation Methods

### Captioning Models Used (in-house VLM/open-source models, model scale) ⚠️

On the evaluation-benchmark side, "annotation" mainly refers to prompt construction and question-answer-pair generation. The models used are as follows:
【VABench】Prompt generation uses a general-purpose LLM (specific model not named [uncertain]) together with expert templates; the I2AV path uses a multimodal LLM to generate a unified audio-visual description from images (model not named [uncertain]); the MLLM scorer at evaluation time is explicitly Qwen2.5-Omni 7B.
【AVBench】The evaluator is a self-trained model: Qwen2.5-Omni 7B (VT, AV dimensions, only the LLM part fine-tuned) and Qwen2-Audio 7B (AT dimension, LLM + connector fine-tuned); hard-negative construction uses LLM-driven perturbation (model not named [uncertain]).
【AV-SyncBench】Data quality control uses Gemini 3 Flash; semantic perturbation uses OpenVoice V2 (human voice timbre replacement) and pretrained DDSP (instrument timbre transfer); no caption generation is involved.
【PhyAVBench】LLM is used for knowledge survey, taxonomy construction, prompt-template generation, and initial quality-control screening (model not named [uncertain]); speech-task transcription uses Whisper-Large V3 to compute WER.
【Omni-Judge】The judge model under test is Qwen3-Omni (30B total parameters / 3B activated), comparing the instruct and reasoning-enhanced variants.

### Caption Density and Degree of Structuring (short/long/dense descriptions, structured fields such as camera motion, style tags) ⚠️

The degree of structuring of prompts/annotations varies significantly across benchmarks, with direct mapping value for training-side caption design:
【VABench has the most thorough structural decoupling】The original prompt is structurally decoupled by an LLM into a "visual sub-prompt" and an "auditory sub-prompt," two independent streams, from which fine-grained VQA (3–7 items) and AQA (3–7 items) question-answer pairs are then generated respectively. That is, one sample carries four types of structured fields: the full prompt + visual sub-prompt + auditory sub-prompt + per-modality question-answer pairs. This three-layer structure of "overall description → per-modality decoupling → verifiable QA" can be directly migrated as a layered caption schema for training data.
【PhyAVBench's single-variable controlled prompts】337 paired prompt sets, each pair differing in only a single physical variable, deliberately avoiding subjective descriptions and explicit hints of the expected acoustic outcome — this is an extremely structured form of "controlled-variable captioning," essentially writing physical attributes into the text as an enumerable label dimension.
【AVBench】470 ≥720p real human-scene prompts, annotated in strata by number of speakers and acoustic complexity; sampling controlled by attribute quota, implying that prompts have already been tagged with enumerable attribute labels (number of speakers, overlapping speech or not, background noise level, etc.) [the specific tagging system is not fully disclosed, uncertain].
【AV-SyncBench】Does not generate captions, but each sample carries structured perturbation metadata: perturbation type (global offset/local jitter/global speed change/human voice timbre replacement/instrument timbre transfer) + perturbation intensity level + scene category + audio major class. This "perturbation label" structure is highly valuable for constructing synchronization negative samples on the training side.
【Omni-Judge】Uses VidProM real-user prompts directly, without structural rewriting — representing the natural distribution of real-user captions (typically short, vague, lacking auditory description).

### Joint Audio-Video Caption Structure (whether it covers both visual and auditory tracks simultaneously, whether it is split into independent fields, e.g., LTX-2's full soundscape description, Script-a-Video's factorized streams, Foley-Omni's three-field scheme)

This field provides the most direct reverse guidance from the evaluation systems to training-side caption schemas:
【VABench's factorized dual-stream scheme】Explicitly decouples a unified audio-visual description into a visual sub-prompt and an auditory sub-prompt field stream via an LLM, and generates VQA / AQA question-answer pairs for cross-verification. This shares the same design philosophy as the training-side Script-a-Video factorized streams and Foley-Omni three-field scheme. Especially notable is the I2AV path's approach: in the description generated by the multimodal LLM from a static image, the visual part is required to be objectively observable, while the audio part is required to be commonsense-inferred, with humans subsequently reviewing the plausibility of the auditory inference — this offers an actionable paradigm for "how to fill in audio captions for audio-track-free video/images."
【VABench's observability constraint】The manual verification step explicitly checks "element observability" and "satisfaction of physical/commonsense constraints," i.e., auditory descriptions in the caption must be reasonably inferable from the visuals and must not be fabricated out of nothing. This constraint can directly serve as a quality-control standard for audio captions in training data.
【AVBench's three-dimensional consistency split】Splits the audio-visual-text relationship into three independent consistency dimensions — AT (audio-text), VT (video-text), AV (audio-video) — each with its own dedicated trained evaluator — indicating that joint-caption quality should be measured along these three edges separately rather than combined into a single score.
【AV-SyncBench's orthogonal split】Further splits the AV edge into two orthogonal sub-dimensions — "temporal alignment" and "semantic matching" — meaning that a complete quality check of joint AV captions requires at least 4 edges: AT, VT, AV-temporal, AV-semantic.
【PhyAVBench】Audio attributes in prompts are expressed implicitly through physical causality (describing physical conditions rather than describing the sound directly) — another schema approach: letting the model derive the acoustic outcome from physical conditions rather than directly giving sound labels.
【Omni-Judge】Among its 9 evaluation dimensions is audio-video-text tri-modal joint consistency (tri-modal coherence), the only benchmark to explicitly define "joint tri-modal" as a caption-quality metric.

### Dialogue Transcription and Speaker Attribute Annotation (ASR transcription + speaker identity/language/accent/emotion) ⚠️

【AVBench has the most complete coverage】Of the 10 dimensions, Speech Content Accuracy (implying ASR transcription comparison), Speech Realism, and Lip Sync Consistency directly target dialogue; hard-negative construction covers speaker identity mismatches and emotional polarity reversals — indicating that its data annotation includes speaker identity and emotion labels. The Hard subset is annotated with number of speakers (3–4) and overlapping-speech attributes.
【PhyAVBench】The speech task uses Whisper-Large V3 to compute WER as a content-accuracy metric.
【VABench】The lip-sync metric draws on the LatentSync method to compute alignment confidence, applied only to the "human sounds-linguistic" subset with a detected talking head.
【AV-SyncBench】Human-voice scenes use OpenVoice V2 for timbre replacement to construct semantic negatives, implying a controllable editing capability over speaker timbre/identity attributes; scene labels distinguish four categories: single speaker, dialogue, group vocalization, singing.
【Omni-Judge】No dedicated dialogue dimension [uncertain].
The annotation status of accent and language labels is not disclosed by any of the five [uncertain].

### Geometric and Structured Annotation (camera parameters, depth, 3D point tracks, action annotation, explicit states) ⚠️

【PhyAVBench is the richest】Its 41 fine-grained test points are essentially a structured physical-attribute annotation system: material hardness/damping, density, surface texture, object size/shape/thickness, impact velocity, contact area/sharpness, rotation speed, tightness, tension, flow velocity, viscosity, spatial volume, surface absorption, propagation medium, source distance, orientation (horizontal/vertical localization), etc. — all enumerable explicit-state annotations. In addition, the Observer Physics dimension implicitly includes camera/listener position and distance parameters, and the Binaural Effect test point involves binaural spatial-orientation annotation.
【VABench's stereo subset】116 prompts explicitly specify left/right channel spatial orientation, equivalent to structured annotation of sound-source orientation; evaluation computes spatial geometric quantities such as Mid/Side energy ratio, phase coherence, and inter-channel consistency.
【AV-SyncBench】Annotates precise timing parameters of perturbations (offset in five levels of 50–500ms, jitter in three levels of 30–700ms, speed change in ten levels of 0.8×–1.25×), which is structured annotation along the time axis.
【AVBench】Hard negatives contain precisely annotated time offsets of 0.2–3.0s.
None of the five involves visual geometric annotation such as camera parameters, depth maps, or 3D point tracks [uncertain].

### Synthetic Data Construction (controlled perturbation/editing to construct training pairs, e.g., InstructAV2AV) ⚠️

This field is the core methodology of AVBench and AV-SyncBench, and is extremely valuable for constructing controlled data pairs on the training side:
【AVBench's hard-negative synthesis】30K real clips extracted from OpenHumanVid serve as positives, expanded via LLM-driven perturbation + algorithmic mismatching to 100K pairs per dimension, totaling 300K supervised samples. The hard-negative types are explicitly enumerated in five categories: ① Temporal offset (0.2–3.0 seconds); ② Acoustic corruption (pitch shifting, speed change); ③ Speaker identity mismatch; ④ Emotional polarity reversal; ⑤ State transitions. This recipe of "real positives + five types of controlled degraded negatives" can be directly reused for training evaluators or data filters.
【AV-SyncBench's dual-axis perturbation synthesis】Three temporal-axis perturbations: global offset (50–500 ms, five levels), local jitter (30–700 ms, three severities), global speed change (0.8×–1.25×, ten levels); two semantic-axis perturbations: human voice timbre replacement (OpenVoice V2) and instrument timbre transfer (pretrained DDSP), with the key design being that semantic perturbation is forced to maintain temporal invariance — the edited audio has exactly the same timing as the original, changing only semantic attributes — thereby achieving strict decoupling of the two axes. This is a textbook approach of "orthogonal perturbation construction of orthogonal evaluation dimensions."
【PhyAVBench's single-variable pairing】337 paired prompt sets, each pair differing in only a single physical variable with all other conditions held constant, is controlled-variable synthesis on the text side; the corresponding real videos are actually filmed rather than synthesized.
【VABench】Prompts are LLM-synthesized, but all audio-visual content is generated entirely by the tested models, without manual degradation construction [uncertain whether it includes negative-sample design].
【Omni-Judge】Does not construct synthetic data.

### Degree of Human Involvement (manual annotation, manual quality control, model pre-screening + human review) ⚠️

The degree of human involvement differs markedly across the five, forming a complete spectrum from "human-heavy" to "human-light":
【PhyAVBench has the heaviest human involvement】Human experts run through all five stages: eliminating infeasible/redundant physical principles generated by the LLM, reviewing the taxonomy for disambiguation and de-duplication, verifying and rewriting each paired prompt one by one to ensure single-variable difference, personally organizing 184 participants to record 11,605 videos in a controlled environment, and at the quality-control stage reviewing physical consistency and tri-modal alignment, deleting/revising/re-capturing problematic samples. In the subjective evaluation stage, PVR-MOS deployed 74 raters to assess the correspondence between audio differences and physical changes on a 1–5 scale.
【AV-SyncBench has the most rigorous cross-review mechanism】5 human annotators, each video clip independently reviewed by at least 3 people, verifying the visibility of the primary sound source and eliminating low-quality samples — "a pool of 5 annotators, ≥3-person cross-review per sample" is a directly reusable QC configuration. It is preceded by a coarse screening from Gemini 3 Flash, forming a standard two-tier "model pre-screening + human review" structure.
【Omni-Judge】6 PhD students with experience in audio-video generation research rated 600 videos on 4 aspects (quality, semantic alignment, temporal alignment, aesthetics) with integer scores of 1–5, serving as the human baseline for measuring Omni-LLM judging capability.
【AVBench】4 domain experts perform pairwise comparison (2AFC), selecting the better model per dimension with ties allowed, with a win-rate formula of (W + 0.5T)/(W + T + L), then using the Pearson correlation coefficient to validate consistency between automatic scoring and human preference.
【VABench】Explicitly uses human workers together with LLMs to filter test samples and adjust the test-data distribution; humans verify category correctness, element observability, physical/commonsense constraints, plausibility of auditory inference, and question discriminability; but the number of annotators and consistency metrics are not disclosed [uncertain].

## Audio-Video Alignment

### Audio-Video Synchronization Detection Methods (lip sync, event alignment)

The five benchmarks together constitute the most complete methodology collection for AV synchronization detection currently available:

【AV-SyncBench — decoupled synchronization detection (the core method of this survey)】Unified protocol: video and audio are cut into non-overlapping 0.64-second chunks, visual and audio embeddings are extracted independently, the diagonal elements of the temporal similarity matrix (cosine similarity) are computed and averaged: S = (1/N)Σ sim(v_i, a_i); judgment uses binary accuracy — whether the score of the original pairing is higher than that of the perturbed pairing. On the temporal side, it examines the separability of three perturbation types (global offset, local jitter, global speed change); on the semantic side, under strict temporal invariance, it changes only timbre/sound source, examining semantic separability. Five representation models are tested: Synchformer, SparseSync, ImageBind, CAV-MAE, CAV-MAE-Sync.

【VABench】Two synchronization tracks: ① Desync — uses Synchformer to predict the audio-video offset, computed only over the first/last 4.8-second windows; ② Lip-Sync — draws on the LatentSync method to compute alignment confidence, applied only to the human sounds-linguistic subset with a detected talking head. Cross-modal semantic alignment additionally uses a set of three: ViCLIP (text-video, with temporal understanding), CLAP (text-audio cosine similarity), ImageBind (joint audio-video embedding space).

【AVBench】Turns AV consistency into a trainable dedicated evaluator (Qwen2.5-Omni 7B fine-tuned), rather than relying on a fixed SyncNet-type model; also sets up an independent Lip Sync Consistency dimension.

【PhyAVBench】Proposes FGAS (Fine-Grained Alignment Score) — frame-level cosine similarity between visual and audio tokens, taking the mean of the diagonal elements of the temporal similarity matrix; the core metric CPRS uses the CAV-MAE Sync encoder (later versions switch to CLAP embeddings) to measure the consistency between the direction of acoustic change across paired samples and the true reference vector.

【Omni-Judge】Treats audio-video synchronization as one of 9 dimensions handed to Qwen3-Omni for judgment, with a negative conclusion: τ_b only 0.142, explicitly noting that Omni-LLM's insufficient temporal resolution makes it unsuited for synchronization detection.

### Specific Synchronization Detection Metrics and Thresholds (SyncNet/Synchformer/LSE/in-house, threshold values, e.g., MOVA's LSE-D≤9.5 and LSE-C≥4.5, SkyReels-V4's SyncNet |offset|≤3∧conf>1.5) ⚠️

Specific metrics and quantitative parameters (directly usable as reference parameters for training-data synchronization filtering):
</content>

【AV-SyncBench's perturbation intensity spectrum — the most valuable threshold calibration data】
- Global offset: 50–500 ms, five levels;
- Local jitter: 30–700 ms, three severities;
- Global speed change: 0.8×–1.25×, ten levels;
- Temporal granularity: 0.64-second chunks; video uniformly at 25 FPS, audio uniformly at 16 kHz.
Key empirical finding: at the 50 ms offset level, the discrimination accuracy of all SOTA synchronization models is only about 0.51 (close to random guessing). This means that when using models like Synchformer/SparseSync for training-data synchronization filtering, their effective resolution floor is around 50 ms or above — mismatches below this magnitude cannot be reliably detected — this has direct guidance value for setting synchronization thresholds.
Empirical model-capability divergence: ImageBind performs best overall on timbre-editing (semantic) tasks at 0.859, but is weaker on temporal alignment; SparseSync shows the opposite trend; Synchformer and SparseSync excel at detecting temporal offsets; CAV-MAE is stronger on local jitter and speed change. Conclusion: no single model can handle both temporal and semantic filtering simultaneously; the training pipeline should combine multiple models.

【AVBench's hard-negative parameters】Temporal offset 0.2–3.0 seconds (covering the coarse-grained mismatch range); acoustic corruption includes pitch shifting and speed change; the evaluator's output is normalized from Yes/No single-token probability ratios into a continuous score (0–1), naturally allowing thresholds to be set.

【VABench】Desync is based on Synchformer-predicted offset, computed over the first/last 4.8-second windows; the MLLM macro score is on a 1–5 scale; the stereo Mono Compat = 1 − normalized mono loss. The absolute thresholds used for screening are not published [uncertain].

【PhyAVBench】CPRS = ½(cosine_similarity + 1), normalized to [0,1], where 1.0 is perfect alignment, 0.5 is orthogonal, and 0.0 is the reverse direction; requires at least N≥20 ground-truth samples per set to be averaged to suppress noise (actual observed average 17).

None of the five uses the classic SyncNet/LSE-D/LSE-C metric suite [uncertain why not adopted], instead generally shifting toward more modern representation models such as Synchformer / CAV-MAE-Sync / ImageBind — this itself is a signal of the 2026 shift in the synchronization-detection technology stack.

### Separate Treatment of Temporal Synchronization vs. Semantic Synchronization (temporal alignment and content-semantic matching as two independent filtering conditions)

AV-SyncBench is the first systematic implementation of this concept; the paper explicitly claims to be "the first benchmark to fully separate the temporal and semantic evaluation of audio-video synchronization," and its decoupled design is directly transplantable to training-data filtering:

【Mechanism of decoupling】The key lies in the orthogonality of perturbation construction — temporal perturbation changes only the time axis, not the audio content (global offset, local jitter, global speed change), while semantic perturbation is forced to impose temporal invariance, replacing only timbre/sound source while keeping the temporal structure completely unchanged (OpenVoice V2 for human voice timbre replacement, pretrained DDSP for instrument timbre transfer). Because the two types of perturbation are constructed orthogonally, model performance along the two axes can be observed independently.

【Empirical value of the decoupling】Empirical results show the two axes indeed measure different capabilities: ImageBind is strong at semantic discrimination (0.859) but weak at temporal alignment, while SparseSync shows exactly the opposite. This directly proves that "one synchronization score fits all" would mask a model's capability gaps, and also means that if training-data filtering uses only a single synchronization score, it will miss one of two types of bad samples — "temporally correct but content-wrong" and "content-correct but temporally wrong."

【Implications for training-data pipelines】Synchronization filtering should be set up as two independent, serial conditions: ① Temporal alignment condition — use Synchformer / SparseSync-type models to detect offset, noting unreliability below 50ms; ② Semantic matching condition — use ImageBind / CLAP-type cross-modal embeddings to detect audio-visual content mismatches (such as mismatched sound effects, post-added background music, off-screen narration).

【Corresponding designs in other benchmarks】VABench likewise makes this split: Desync (Synchformer temporal offset) and Audio-Visual Align (ImageBind semantic alignment) are listed as two separate independent metrics, while Module 2's Alignment dimension is explicitly defined as a combined judgment of "temporal synchronization + semantic correspondence." AVBench's AV Consistency dimension merges the two without splitting them; its hard negatives include temporal offset (temporal) and speaker identity mismatch/emotion reversal (semantic) as two distinct types but judged by the same evaluator. PhyAVBench's Time and Causality dimension focuses specifically on temporal causality (far-field delay from the speed-of-light/speed-of-sound difference, transient onset/offset synchronization, periodic/aperiodic rhythm consistency), while its other five dimensions pertain to acoustic-semantic correctness — this also constitutes an implicit temporal/semantic separation.

### Audio Quality Filtering (SNR, silence detection and silence-ratio thresholds, no-audio-track removal, off-screen sound source removal, background music separation) ⚠️

【AV-SyncBench's dual checkpoint】The first checkpoint, Gemini 3 Flash, automatically removes samples with "off-screen sound sources" — this is the most critical and also hardest-to-do filtering item for AV training data (post-dubbing, off-screen narration, and overlaid background music all fall into this category); the second checkpoint is 5-person cross human review, confirming the primary sound source is visible on screen and eliminating clips with poor audio quality, excessive noise, or semantic ambiguity. Together they guarantee "what you see is what you hear," making this the best-practice example of on-screen sound source filtering in this survey (no quantitative thresholds such as SNR published [uncertain]).
【VABench】The audio quality triple can be directly reused as filters: DNSMOS measures background noise/speech clarity, NISQAv2 measures overall quality and naturalness, Audiobox measures audio aesthetics (pleasantness, usefulness, production complexity and quality); on the generation side, a uniform 48kHz stereo track is extracted.
【AVBench】NISQA MOS measures audio quality, Audiobox measures audio aesthetics, both producing continuous scores; the paper explicitly states these can be used as a data filtering mechanism. The Hard subset deliberately retains noisy-background and overlapping-speech samples, indicating its positioning is "difficult but not dirty" — training-data filtering should likewise distinguish between "noisy but genuine" and "poor quality that needs removal."
【PhyAVBench】Controlled-environment recording guarantees audio quality from the source; the quality-control stage's human review eliminates samples containing unintended physical confounds (equivalent to removing mixed-in irrelevant sound sources).
【Omni-Judge】Sets up audio quality and audio aesthetic as two dimensions handed to Omni-LLM for judgment, but the overall conclusion suggests LLM scoring of perceptual dimensions is unreliable.
None of the five mentions specific approaches for silence-ratio thresholds, no-audio-track removal, or background music separation (source separation) [uncertain].

### Classification and Separate Handling Strategy for Speech/Sound Effects/Music ⚠️

【AV-SyncBench has the clearest classify-and-treat-separately approach】Routes processing by the three major classes Voice / Music / Sound: for semantic perturbation, human-voice classes go through OpenVoice V2 timbre replacement, instrument classes go through DDSP timbre transfer, and general sound-effect classes are not semantically perturbed (hence semantic-challenge samples number only 821, far fewer than the 37,569 temporal-challenge samples). This indicates that the difficulty of controllable editing differs greatly across audio types — speech and music have mature timbre-decoupling tools, while general Foley sound effects lack them — this reality also constrains the feasibility of constructing Foley negative samples on the training-data side. The ten-scene labels further subdivide into single-source and multi-source (single speaker vs. dialogue/group vocalization, single instrument vs. ensemble).
【VABench applies differentiated metrics by category】The lip-sync metric is applied only to the human sounds-linguistic subset with a detected talking head; speech-quality metrics (DNSMOS, NISQAv2) are meaningful only for samples containing speech; the Audio Realism physical-plausibility score explicitly excludes the Virtual Worlds category. This mechanism of "conditionally applying metrics by audio type" corresponds, on the training-data side, to setting different domain-specific quality gates.
【PhyAVBench focuses on Foley physical sound effects】Its six dimensions all revolve around non-speech physical sound-production mechanisms; speech is only indirectly involved through WER — this fills the evaluation gap for physical correctness in Foley-type sound effects.
【AVBench focuses on speech】Of its 10 dimensions, three directly target speech (content accuracy, realism, lip sync); music and general sound effects are basically not covered, making it a pure human-voice-scene benchmark.
【Omni-Judge】No layered analysis by audio type [uncertain].
Overall: speech / music / Foley sound effects are highly heterogeneous in evaluation metrics, controllable editing tools, and data availability; training pipelines must classify and treat them separately rather than use a uniform threshold.

## Training Support

### Multi-Stage Training Curriculum and Data Curriculum Scheduling (basis for stage division: resolution/duration/quality score/modality; low-res→high-res, image→video, short→long) ⚠️

The five benchmarks themselves, being evaluation benchmarks, do not involve the training curriculum of generative models [not applicable]. The only one involving training is AVBench's evaluator training: the VT and AV evaluators adopt a partial-parameter strategy of "fine-tuning only the LLM part, freezing the rest" based on Qwen2.5-Omni 7B; the AT evaluator adopts a strategy of "fine-tuning the LLM + connector layer" based on Qwen2-Audio 7B. Both are single-stage preference-learning fine-tuning, without a multi-stage curriculum; specific hyperparameters such as learning rate, number of epochs, and batch size are not disclosed in the accessible content [uncertain].
Curriculum-design signals that can be borrowed in reverse: AVBench's Normal (1–2 speakers) / Hard (3–4 speakers + overlapping speech + noise) two-subset stratification, and AV-SyncBench's multi-level perturbation-intensity design (5 offset levels, 3 jitter levels, 10 speed-change levels), are essentially difficulty-grading systems that can be directly mapped to the easy-to-hard curriculum arrangement basis for training data.

### Data Ratio Changes Across Training Stages (pretraining/annealing/SFT high-quality subsets) ⚠️

None of the five involves the staged ratios of generative-model training [not applicable/uncertain]. Extractable ratio-related designs are:
【AVBench】Evaluator training data is ratio-balanced uniformly by dimension — 100K pairs per consistency dimension, totaling 300K across AT/VT/AV, i.e., a strict 1:1:1 balanced ratio; the positive-to-negative sample ratio is not disclosed [uncertain]. The evaluation set is split 350:120 (about 74.5% : 25.5%) between the Normal and Hard subsets — this roughly 3:1 "normal:hard" ratio can serve as a reference anchor for the proportion of hard cases in an SFT high-quality subset.
【AV-SyncBench】Temporal challenge 37,569 samples vs. semantic challenge 821 samples, a ratio of about 46:1, reflecting construction cost rather than an ideal ratio.
【VABench】T2AV 778 : I2AV 521 : stereo 116, about 6.7 : 4.5 : 1.
【PhyAVBench】337 paired prompt sets span 6 dimensions with 41 test points, averaging about 8 sets per test point; the distribution of set counts across dimensions is not disclosed item by item [uncertain].

### Post-Training Data (SFT curated-set scale and selection criteria, number of preference pairs and annotation method, reward-model training data) ⚠️

Strictly speaking, none of the five produces post-training data for generative models [not applicable], but the outputs of AVBench and AV-SyncBench can directly serve as post-training assets:
【AVBench has the most post-training value】Its 300K preference pairs with hard negatives are themselves in a standard preference-data format; the paper explicitly states that the evaluator's continuous differentiable score can be used as an RLHF reward signal and a data filtering mechanism, i.e., this evaluator can directly serve as a reward model for post-training AV generation models. This is the only benchmark in this survey explicitly positioned as a "reusable reward model." Its human preference annotation uses 4 experts doing 2AFC pairwise comparison, allowing ties, with a win-rate formula of (W + 0.5T)/(W + T + L) — a standard preference-annotation protocol.
【AV-SyncBench】Its 38,390 samples with precise perturbation labels can directly serve as training/calibration data for synchronization discriminative models or a sync reward model.
【Omni-Judge】600 videos × 9 dimensions × 6 PhD students of human ratings constitute a small but high-quality human preference calibration set, usable for calibrating the consistency between automatic reward models and humans.
【VABench / PhyAVBench】Positioned purely as evaluation, providing no preference pairs or reward-training data [not applicable].

### Data Processing Infrastructure and Throughput (NeMo Curator/Data-Juicer/in-house, GPU acceleration ratio, processing scale, cost) ⚠️

None of the five discloses data-processing infrastructure, GPU acceleration ratios, processing throughput, or cost [uncertain] — a common gap in evaluation-benchmark papers. Indirectly observable engineering scale includes:
【PhyAVBench】11,605 videos / 25.5 hours, all newly recorded, involving 184 participants, multiple recording devices, and controlled environment setups, plus 74 PVR-MOS raters and 29+ authors collaborating — the largest human-engineering investment among the five (cost data not disclosed [uncertain]).
【AV-SyncBench】3,269 videos underwent full-volume calls to Gemini 3 Flash for pre-screening, followed by ≥3-person cross-review from 5 annotators (about 9,800+ person-reviews), then programmatic generation of 38,390 perturbed samples; Gemini API call costs are not disclosed [uncertain]. Choosing the Flash-tier lightweight model rather than the Pro tier is itself a manifestation of the cost-throughput tradeoff.
【AVBench】The compute required for synthesizing 300K training samples and fine-tuning the 7B model is not disclosed [uncertain]; a HuggingFace Leaderboard is provided, indicating ongoing online evaluation-service infrastructure.
【VABench】1,299 samples generated by multiple commercial model APIs (Veo3, Wan2.5, Sora2, Seedance, Kling via official APIs, ThinkSound and MMAudio deployed locally); API call costs are not disclosed [uncertain].
No mention of dedicated data-processing frameworks such as NeMo Curator or Data-Juicer [uncertain].

## Comparative Results

### Quantitative Impact of Data-Strategy Ablations (distinguishing: filtering-strictness ablation / caption-density-style ablation / data-ratio ablation, and corresponding evaluation metrics) ⚠️

The five benchmarks, being evaluation benchmarks, do not perform training-data-strategy ablations [not applicable], but they provide numerous diagnostic conclusions useful for guiding data strategy:
【PhyAVBench's model capability profile】A full evaluation of 17 SOTA models found that even the strongest, Sora 2, has a CPRS of only 0.4512 (out of a maximum of 1.0, where 0.5 is the random-orthogonal level), indicating that all current models are essentially in a state of "near-zero physical sensitivity" for audio physical correctness; among V2A routes, MMAudio is best but reaches only 0.4003. The consistently weak points across models are the two dimensions of fluid dynamics and sound propagation environment — this directly points to a domain missing from training data: flowing water/bubbles/viscosity-type fluid sounds, and reverberation/echo/occlusion/underwater/solid-borne-transmission-type propagation-environment samples. This is the most actionable data-supplementation checklist.
【PhyAVBench's metric-credibility validation】CPRS achieves a Pearson correlation of 0.92 with human judgment, indicating the automatic metric is reliable and can be used as a proxy metric for data filtering.
【AV-SyncBench's detector capability profile】ImageBind is strong semantically (0.859) but weak temporally, SparseSync is strong temporally but weak semantically, CAV-MAE is strong on jitter and speed change, and Synchformer and SparseSync excel at offset detection; at the fine-grained 50 ms offset level, all models' accuracy drops to about 0.51. This is equivalent to a "filter-selection ablation," concluding that multiple models must be combined and divided by task for filtering.
【AVBench's alignment validation】Uses the Pearson correlation coefficient to validate the consistency between automatic scoring and 4-expert 2AFC preference; its hard-negative design (temporal offset 0.2–3.0s, pitch/speed-change corruption, speaker mismatch, emotion reversal, state transitions in five categories) can, after preference learning, detect subtle cross-modal inconsistencies, indirectly proving the effectiveness of the "synthetic negatives with controlled degradation" data-construction strategy.
【Omni-Judge's judge-capability ablation】Measures Omni-LLM's correlation with humans dimension by dimension, yielding a clear capability boundary: semantic dimensions are usable (audio-text τ_b=0.292/ρ=0.345, audio-video-text 0.139/0.151), perceptual dimensions are not usable (video quality τ_b≈0.020, AV sync 0.142); it also compares the differences between the instruct version and the reasoning-enhanced version. This is the most direct quantitative ablation of the strategy of "using large models for data quality control."
【VABench】A horizontal comparison across 15 dimensions × 8 models/combinations, from which the systematic differences between end-to-end native joint generation (Sora2/Veo3/Wan2.5) and cascaded V+A combinations across dimensions could be read out [specific figures not fully disclosed in the accessible content, uncertain].

### Evidence of Quality vs. Quantity (cases where small, precise data outperforms large, messy data)

【PhyAVBench is the strongest evidence】Using only 337 paired prompt sets and 11,605 carefully recorded videos (25.5 hours, an extremely small scale), it drove the CPRS of all 17 SOTA models below 0.45, exposing a systemic failure in basic audio physics for commercial large models trained on massive web data. Its core methodological claim is "record everything new to guarantee zero training-set overlap" — emphasizing data purity and controllability over scale, and explicitly requiring at least N≥20 real samples per set to be averaged to suppress noise, embodying an approach of "small and precise + repeated sampling."
【AVBench's 30K→300K expansion approach】Rather than collecting more data, it applies controlled degradation expansion to 30K high-quality real clips to reach 300K training pairs, substituting the diversity of synthetic negatives for the scale of real data — a "small seed + controllable synthesis" quality-first route.
【AV-SyncBench】3,269 clean videos, after Gemini pre-screening + 5-person cross-review, expand into 38,390 evaluation samples; its strict on-screen sound-source requirement means a large amount of original data is discarded in exchange for absolute label reliability.
【VABench】1,299 samples covering 15 evaluation dimensions, winning through structured category-system design rather than sample volume.
【Omni-Judge】Only 300 prompts / 600 videos, yet arrives at conclusions substantial enough to change data-pipeline design (Omni-LLM is unusable for perceptual-dimension scoring).
Shared insight: the "small and precise" paradigm has been validated on the evaluation side; mapped to the training side, investment in domain coverage and label reliability may yield a higher return than simply expanding data volume.

### Alignment Between Training-Data Domain Distribution and Evaluation-Benchmark Category Systems (e.g., VABench's seven categories)

This field is the core output of this survey — the category systems of the five benchmarks can directly serve, in reverse, as a standard coordinate system for training-data domain distribution; it is recommended to combine them along three orthogonal axes:

【Axis 1: Content domain axis — adopting VABench's seven categories】Animals / Human Sounds (linguistic·non-linguistic) / Music / Environmental Sounds (natural·urban·indoor) / Synchronous Physical Sounds / Complex Scenes (complex soundscapes·subjective feelings·world knowledge·symbolic association·off-screen sound sources) / Virtual Worlds. This is the most complete content-side taxonomy, directly usable as the primary row label for a training-data ratio table. Two key design elements should be migrated together: ① Human sounds must be split into linguistic and non-linguistic (the former needs lip-sync and ASR annotation, the latter does not); ② The Virtual Worlds category should be exempted from the physical-plausibility quality gate, with filtering standards set separately from realistic categories.

【Axis 2: Audio type and synchronization-difficulty axis — adopting AV-SyncBench's tripartite division + ten scenes】Voice / Music / Sound as three major classes, each containing scenes among: action, animal sound, object sound, ambient sound, group vocalization, single speaker, dialogue, singing, single instrument, ensemble. This system partitions by synchronization difficulty rather than topic; its distinction between single-source and multi-source (single speaker vs. dialogue/group vocalization, single instrument vs. ensemble) directly corresponds to the ratio needs for multi-source overlapping samples in training data, and also determines the selection of synchronization filters (reliability of Synchformer-type models decreases in multi-source scenes).

【Axis 3: Physical-correctness coverage axis — adopting PhyAVBench's six dimensions with 41 test points】Sound source mechanics / mechanical structure / fluid and aerodynamics / sound propagation environment / observer physics / time and causality (and complex coupling and extreme physics). This is a checkable checklist for auditing blind spots in training data's physical-acoustic coverage. Empirical evaluation has already identified fluid dynamics and sound propagation environment as the industry's common blind spots, which can serve as priority directions for data supplementation.

【Orthogonal constraint layer — adopting AVBench's quota-balancing mechanism】Hard Quota-Based Greedy Sampling forcing any single attribute's proportion to be ≤50%, and the Normal : Hard ≈ 3:1 difficulty stratification. Both can be directly migrated as concept-balancing strategies and hard-case ratio anchors for sampling training data.

【Real-distribution calibration — adopting Omni-Judge's VidProM prompt distribution】The preceding four axes are all expert-designed idealized classifications, prone to detaching from real user needs; Omni-Judge directly samples the real-user VidProM prompt distribution, providing an empirical distribution of "what users actually want to generate," usable for calibrating the weights of the aforementioned category systems and avoiding over-investment in long-tail categories.

【Deployment recommendation】Build a three-dimensional ratio matrix (content domain × audio type × physical test-point coverage), setting weights with the real VidProM distribution, constraining single-attribute proportions with AVBench's quota rule, auditing blind spots with the PhyAVBench checklist, and driving domain-specific selection of synchronization filters with AV-SyncBench's scene labels. Also note the shared gaps across the five benchmarks — multi-shot long-form narrative, multilingual accents, and cross-shot audio-track continuity currently have no benchmark to reference; investment in training data for these domains cannot currently be validated against a public benchmark.

## Uncertain Fields

The research information for the following fields is partially uncertain (as marked by the ⚠️ source):

- release_date
- openness
- provenance_licensing
- duration_distribution
- resolution_aspect_distribution
- domain_distribution
- audio_category_distribution
- language_accent_distribution
- funnel_retention_rate
- shot_segmentation
- quality_filtering
- motion_filtering
- deduplication
- safety_filtering
- caption_model
- caption_structure
- dialogue_transcription_attributes
- geometric_structured_annotation
- synthetic_data_synthesis
- human_in_loop
- sync_metric_and_threshold
- audio_quality_filtering
- audio_type_handling
- multi_stage_curriculum
- stage_data_mixture
- post_training_data
- data_infra_throughput
- data_ablation
