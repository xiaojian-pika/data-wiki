# Wan 2.5 / 2.6 / 2.7 (Tongyi Wan, Wan series closed-source commercial versions) — including its open-source predecessors Wan 2.1 (arXiv:2503.20314), Wan 2.2 (Apache 2.0), and same-team corroborating models Wan2.2-S2V (arXiv:2508.18621), Wan2.2-Animate, and Wan-Dancer (arXiv:2607.09581)

> Topic: Data processing for video-generation models (including simultaneous audio-video generation): data cleaning pipelines, data distribution, annotation methods

[← Back to Home](../index.md)

**Table of Contents**: [Basic Information](#basic-information) · [Data Scale and Distribution](#data-scale-and-distribution) · [Cleaning Pipeline](#cleaning-pipeline) · [Annotation Methods](#annotation-methods) · [Audio-Video Alignment](#audio-video-alignment) · [Training Integration](#training-integration) · [Performance Comparison](#performance-comparison)

## Basic Information

### Name

Wan 2.5 / 2.6 / 2.7 (Tongyi Wan, Wan series closed-source commercial versions) — including its open-source predecessors Wan 2.1 (arXiv:2503.20314), Wan 2.2 (Apache 2.0), and same-team corroborating models Wan2.2-S2V (arXiv:2508.18621), Wan2.2-Animate, and Wan-Dancer (arXiv:2607.09581)

### Publishing Organization/Company

Alibaba Group · Alibaba Cloud Tongyi Lab (the Wan / Wanxiang team; the human-centric models S2V, Animate, and Dancer are led by the HumanAIGC group). The service is delivered via Alibaba Cloud Bailian (Model Studio / DashScope) and the Wanxiang official site wan.video, tongyi.aliyun.com/wan.

### Release Date (Technical Report/Paper/Open-Source Release) ⚠️

Series timeline (all verifiable first-hand from official sources):
- Wan 2.1: open-sourced on February 25, 2025 (1.3B/14B); the technical report "Wan: Open and Advanced Large-Scale Video Generative Models," arXiv:2503.20314, was released on March 26, 2025, 60 pages with 33 figures, with Chapter 3 being a full data-processing chapter — this is the only detailed first-hand document across the entire Wan series disclosing data methodology.
- Wan 2.2: open-sourced on July 28, 2025 (T2V-A14B / I2V-A14B MoE, TI2V-5B), Apache 2.0.
- Wan2.2-S2V-14B (audio-driven human video): paper arXiv:2508.18621 submitted August 26, 2025; weights uploaded to Hugging Face on September 17, 2025.
- Wan2.2-Animate-14B: open-sourced in early November 2025.
- Wan 2.5-preview: launched as an API preview during late September 2025 (around the Apsara Conference), with model names wan2.5-t2v-preview / wan2.5-i2v-preview; official documentation example assets carry date stamps of 2025-09-23 / 2025-09-25, which can serve as corroborating evidence of timing. This was the series' first realization of "audio-visual sync." [Uncertain: exact day]
- Wan 2.6: released on the Wanxiang official site on December 16, 2025, with the official announcement on the Alibaba Cloud Developer Community on December 17; model names wan2.6-t2v / wan2.6-i2v / wan2.6-i2v-flash / wan2.6-t2v-us.
- Wan 2.7: June 2026, official API model names carry date stamps directly — wan2.7-t2v-2026-06-12 and wan2.7-i2v — currently the recommended flagship version.
- Wan-Dancer-14B (music-driven long-form dance video): open-sourced July 2026, paper arXiv:2607.09581.
Note: none of 2.5/2.6/2.7 has a technical report, paper, or released weights, so their data methodology can only be inferred backward from first-hand disclosures for 2.1/2.2/S2V and from API documentation capabilities.

### Type (Model/Dataset/Toolchain/Benchmark)

Model (a video-generation foundation-model series). 2.5/2.6/2.7 are closed-source commercial API models; the same family also includes open-source models (Wan2.1/2.2/S2V/Animate/Dancer), an open-source inference and fine-tuning toolchain (the Wan-Video GitHub organization, ComfyUI/Diffusers integration, the Wan-skills agent skill pack), and an in-house evaluation benchmark, Wan-Bench (3 major dimensions, 14 fine-grained metrics). Not a dataset entry.

### Open-Source Status (Weights/Code/Data/Pipeline)

Shows a "fully open predecessor, fully closed current generation" fault-line pattern, with the data side closed throughout.
**Wan 2.5 / 2.6 / 2.7**: fully closed-source — no weights, no code, no technical report, no paper. Service is offered solely via the Alibaba Cloud Bailian DashScope API (multi-region: Beijing/Singapore/Virginia) and the Wanxiang official site/Tongyi App, billed by resolution tier and duration. As of July 2026, neither the Hugging Face Wan-AI organization nor the GitHub Wan-Video organization contains any repository or weights for Wan2.5/2.6/2.7 — GitHub has only five repositories: Wan2.1, Wan2.2, Wan-Dancer, Wan-skills, and a diffusers fork.
**Wan 2.1**: weights + inference code open-sourced (Apache 2.0), along with a 60-page technical report containing a detailed data chapter; training data, cleaning-pipeline code, the in-house caption model, and the various expert classifiers are not open-sourced.
**Wan 2.2**: weights + inference/fine-tuning code open-sourced (Apache 2.0); the README discloses the data-increase ratios relative to 2.1 and the "cinematic aesthetic tagging system," but there is no standalone technical report, and data details are far sparser than in 2.1.
**Wan2.2-S2V / Animate / Dancer**: weights + code open-sourced, with papers containing data-processing chapters (S2V's Chapter 2 is the only document in the Wan family that explicitly spells out the data-filtering method for audio-visual sync).
**Conclusion**: to study the data methodology of Wan 2.5+, one can only use the Wan 2.1 report as the backbone, the Wan 2.2 README and the S2V/Dancer papers as incremental corroborating evidence, and infer the rest from API behavior.

### Whether Joint Audio-Video Generation Is Supported, and How It Is Implemented (Native Joint/Cascaded/MoE Fusion) ⚠️

Supported, and this is the watershed capability of the 2.5 release. The official line is "native audio-visual sync," but paper-level evidence is lacking.
**Capability evolution**
- Wan 2.1 (2025.02): audio is a separate, cascaded V2A module (Section 5.7 of the technical report) — video is generated first, then dubbed with audio; and it explicitly produces only ambient sound and background music, deliberately excluding speech/vocal singing.
- Wan2.2-S2V (2025.08): a dedicated model for audio-driven human animation (audio→video), which is conditional generation rather than joint generation.
- Wan 2.5-preview (2025.09): the official capability tags first list "audio-enabled video / audio-visual sync." Both T2V and I2V can automatically add audio (generating matching background music or sound effects) when no audio is supplied, or can be driven by audio passed via input.audio_url. Lip-sync capability becomes a selling point of the series from this point on.
- Wan 2.6 (2025.12): the capability tags are upgraded to "multi-shot narrative + audio-visual sync." Officially described as "native audio-visual sync, with picture perfectly matched to voice, sound effects, and BGM," and a new "voice-driven" mode is added (audio directly drives the character's lip movement and performance) along with "role-play" (uploading a personal video to replicate the person's likeness and voice). Officially claimed to be "the first domestic video-generation model to support role-play" and "the world's most feature-complete video-generation model."
- Wan 2.7 (2026.06): continues "multi-shot narrative + audio-visual sync," with audio as a first-class input modality (input.audio_url, wav/mp3, 2–30 seconds, ≤15MB); on the I2V side the input modality expands to "text + image + audio + video" (supporting first-frame, first-and-last-frame, video continuation, and continuation-plus-last-frame control). This is consistent with the research notes' statement that "2.7 adds native audio conditioning" — audio shifts from "output-side dubbing" to "input-side conditioning."
**Implementation**: the official marketing language is "Alibaba's in-house native multimodal architecture," pointing toward native joint generation rather than cascading; but no paper, architecture diagram, or parameter disclosure exists to confirm this, nor can one determine whether MoE fusion is used (Wan 2.2's MoE is a high-noise/low-noise dual-expert split by denoising stage, unrelated to modality). [Uncertain: the specific fusion mechanism]
**Output specs**: wan2.5-*-preview: 480P/720P/1080P, 5s or 10s; wan2.6-*: 720P/1080P, 2–15s in integer steps; wan2.7-*: 720P/1080P, 2–15s in integer steps, with ratio support for 16:9/9:16/1:1/4:3/3:4 (e.g., 1080P 16:9 = 1920×1080, 4:3 = 1648×1248); the entire series is fixed at 30fps, MP4/H.264.

### Research Source List (URLs of papers/technical reports/official docs/news, each annotated by nature: official first-hand/same-team corroborating/third-party report)

- **[Official first-hand · core]** Wan: Open and Advanced Large-Scale Video Generative Models (Wan Team, Alibaba Tongyi Lab), arXiv:2503.20314, 2025-03-26, 60 pages, 33 figures. Chapter 3, "Data Processing Pipeline," is the sole detailed first-hand source of data methodology for the whole series: 3.1 Pretraining data (four-step cleaning, 9 fundamental filters, ~50% elimination rate, 100-cluster quota sampling, six-tier motion grading, synthetic visual text); 3.2 Post-training data (top 20%, 12 major categories, million-scale); 3.3 Dense video captioning (LLaVA-style architecture, slow-fast encoding, 10-dimensional F1 comparison against Gemini 1.5 Pro); 4.2.2/4.2.3 training curriculum; 4.6 Wan-Bench; 4.7.2 architecture ablations; 5.7 audio generation (V2A's 1D-VAE, three-part audio-video caption, an O(1)-thousand-hour subset, exclusion of speech): https://arxiv.org/abs/2503.20314
- **[Official first-hand]** Wan 2.1 technical report PDF (all data-chapter passages cited in this research are quoted from this PDF): https://arxiv.org/pdf/2503.20314
- **[Official first-hand]** Wan-S2V: Audio-Driven Cinematic Video Generation, arXiv:2508.18621, 2025-08-26. Chapter 2, "Data Processing Pipeline," is the only document in the Wan family that spells out the data-filtering method for audio-visual sync: two exclusion rules from Light-ASD active-speaker detection, VitPose/DWPose pose tracking, five quality metrics (Dover/UniMatch/Laplacian/aesthetics/OCR), and the QwenVL2.5-72B annotation specification; Table 1 includes quantitative comparisons such as Sync-C: https://arxiv.org/abs/2508.18621
- **[Official first-hand]** Wan2.2 GitHub repository README (data-increase figures "images +65.6%, video +83.2%," the cinematic aesthetic tagging system covering lighting/composition/contrast/color tone, MoE dual-expert denoising, Wan2.2-VAE's 16×16×4 compression, S2V and Animate, Apache 2.0): https://github.com/Wan-Video/Wan2.2
- **[Official first-hand]** Wan2.1 GitHub repository: https://github.com/Wan-Video/Wan2.1
- **[Official first-hand]** Wan-Video GitHub organization page (as of 2026-07 contains only five repositories — Wan2.1/Wan2.2/Wan-Dancer/Wan-skills/diffusers — with no Wan2.5/2.6/2.7, direct evidence of closed-source status): https://github.com/Wan-Video
- **[Official first-hand]** Hugging Face Wan-AI organization page (the latest weights stop at the Wan2.2 series and Wan-Dancer-14B, with no 2.5/2.6/2.7): https://huggingface.co/Wan-AI
- **[Official first-hand · capability matrix]** Alibaba Cloud Bailian "Video Generation" model overview documentation (capability tags, input modalities, resolution tiers, duration, fps, and regional deployment for all versions wan2.7/2.6/2.5/2.2/2.1; explicitly states 2.5 = "audio-enabled video · audio-visual sync," 2.6/2.7 = "audio-enabled video · multi-shot narrative · audio-visual sync," and that 2.7-i2v input modalities include text/image/audio/video): https://help.aliyun.com/zh/model-studio/video-generation
- **[Official first-hand · API]** Wanxiang 2.7 text-to-video API reference (model name wan2.7-t2v-2026-06-12 carries a release-date stamp, audio_url audio-conditioned input, automatic dubbing, a timestamped shot-script example, duration [2,15], a ratio/resolution correspondence table, a 5000-character prompt limit, and a watermark parameter): https://help.aliyun.com/zh/model-studio/text-to-video-api-reference
- **[Official first-hand · API]** Wanxiang image-to-video (first-frame-based) API reference (2.1–2.6) (wan2.6's shot_type:"multi" multi-shot narrative, wan2.5/2.6's audio_url and automatic dubbing, the 3–30-second/wav/mp3/≤15MB audio constraint and truncation rule, per-version enumeration of resolutions and durations, and differences in prompt character limits): https://help.aliyun.com/zh/model-studio/image-to-video-api-reference
- **[Official first-hand · release announcement]** "The all-new Wanxiang 2.6 series models, officially released!," Alibaba Cloud Developer Community, 2025-12-17 (the first domestic video-generation model to support role-play, audio-visual sync, multi-shot generation, voice-driven mode, up to 15 seconds in a single generation, shot control and cross-shot consistency modeling, the Wanxiang family now supports 10+ visual creation capabilities): https://developer.aliyun.com/article/1693622
- **[Official first-hand · release announcement]** "Wanxiang Wan2.6 all-new upgrade released! The era where anyone can be a director has arrived," Alibaba Cloud Developer Community, 2025-12-16: https://developer.aliyun.com/article/1693451
- **[Official first-hand · same-team corroborating]** Wan-Dancer (music-driven minute-scale dance video generation), arXiv:2607.09581, 2026-07. Data side: a self-built proprietary dataset of roughly 200 hours at ≥720p/30fps, five dance genres in near-uniform distribution to mitigate class imbalance, a splitting strategy of 5-second clips with 50% overlap, Librosa audio features, and SEA-RAFT optical-flow masks fed into the loss: https://arxiv.org/abs/2607.09581
- **[Official first-hand]** Wan-Dancer GitHub repository README: https://github.com/Wan-Video/Wan-Dancer
- **[Official first-hand]** Wanxiang official site (product pages and online trial entry points for Wan 2.5/2.6/2.7): https://wan.video/
- **[Official first-hand]** Tongyi Wanxiang official site entry: https://tongyi.aliyun.com/wan/
- **[Third-party compilation]** Wanxiang 2.6 product description page ("major release on 2025.12.16," native audio-visual sync, multi-shot narrative, first video role-play feature, audio-driven mode, 15 seconds/1080P, an increase from Wan 2.5's 10 seconds): https://wan2.video/zh/wan2.6
- **[Third-party compilation]** Wanxiang 2.7 product description page (1080P/15-second cap, first-and-last-frame control, character cloning, instruction-based editing, single-image multi-shot narrative, native audio-visual sync): https://wan2.video/zh/wan2.7
- **[Third-party report]** "Tongyi Wanxiang Wan2.6 released: from 'random generation' to 'precise direction,'" Tencent News, 2025-12-17: https://news.qq.com/rain/a/20251217A031VN00
- **[Third-party report]** "Alibaba Tongyi Wanxiang Wan 2.6 released, from 'generating a video clip' to 'shooting the scene for you,'" Zhihu column: https://zhuanlan.zhihu.com/p/1984672026435294934
- **[Third-party report]** "Alibaba Tongyi Wanxiang Wan2.6 officially released! How to use it?," Zhihu column: https://zhuanlan.zhihu.com/p/1986838016199766019
- **[Related-work comparison]** LTX-2: Efficient Joint Audio-Visual Foundation Model, arXiv:2601.03233 (audio information-content filtering and full-soundscape dual-track captioning, providing a contrast with Wan's three-part V2A caption): https://arxiv.org/abs/2601.03233
- **[Method citation · third-party]** Light-ASD: A Light Weight Model for Active Speaker Detection, Liao et al., CVPR 2023 (the model used by Wan2.2-S2V for audio-visual-sync filtering): https://openaccess.thecvf.com/content/CVPR2023/html/Liao_A_Light_Weight_Model_for_Active_Speaker_Detection_CVPR_2023_paper.html

## Data Scale and Distribution

### Training Data Scale (Video Count/Hours/Token Count, Pretraining vs SFT Separated) ⚠️

Completely undisclosed for 2.5/2.6/2.7. All available scale information comes from predecessors and same-family models:
- Wan 2.1: the candidate pool is "billions of videos and images"; after fundamental-dimension filtering, roughly 50% is eliminated. No hour count or token count is given, nor is there a breakdown of absolute pretraining vs. SFT volumes.
- Wan 2.2: the README explicitly gives a relative increase — compared to Wan 2.1, "images +65.6%, video +83.2%" — the only cross-version quantitative comparison of data scale in the Wan series; officially this increase is said to significantly improve generalization in motion, semantics, and aesthetics.
- Wan 2.1 post-training set: curated images at the "millions" scale; on the video side, "millions of simple-motion clips" and "millions of complex-motion clips" were separately selected.
- Wan 2.1 V2A audio subset: after strict filtering from the video-generation dataset, only an O(1)-thousand-hour subset remains — the only audio-side scale figure in the Wan series, indicating that audio-visual-usable data is an extremely small subset relative to video data.
- Wan 2.1 personalization subset: an internal portrait classifier first screened out roughly O(100)M videos, from which roughly O(10)M personalized videos were ultimately constructed (each averaging 5 attached segmented face images), plus roughly O(1)M synthesized face-swap videos.
- Wan-Dancer (2026.07): a self-built set of roughly 200 hours of high-quality dance video (≥720p/30fps).
Starting from 2.5, new capabilities — speech, lip sync, multi-shot narrative, role-play — necessarily entail a substantial expansion of audio-visual and multi-shot data, but no figures are given. [Uncertain]

### Data Source Composition (In-House/Public Datasets/Web Scraping/Licensed Acquisition/Synthetic Data) ⚠️

Undisclosed for 2.5/2.6/2.7. The Wan 2.1 report's original wording is: the candidate dataset was "sourced from both internal copyrighted sources and publicly accessible data" — i.e., two legs, internal proprietary data plus publicly accessible data. Alibaba's own ecosystem (Taobao/Youku, etc.) is not named but is a reasonable inference. [Uncertain]
Breakdown (per the 2.1 report and same-family papers):
1) Internal copyrighted sources: proportion undisclosed;
2) Publicly accessible data: Wan2.2-S2V explicitly states that data was collected from open-source video datasets with a coarse caption-keyword screen, plus manually curated videos from publicly accessible sources containing complex human activities (speaking, singing, dancing);
3) Synthetic data: to improve Chinese glyph rendering, Wan 2.1 rendered Chinese characters on a plain white background, synthesizing "hundreds of millions" of text-containing images; for personalization data, InstantID was used to synthesize roughly O(1)M face-swap videos;
4) Licensed acquisition: the report does not mention any named stock-footage licensing partnership (unlike Lightricks' approach of licensing Shutterstock/Getty content).
No percentage figures for the sources are given.

### Data Compliance and Provenance (Licensed Data Share, Rights-Cleared Datasets, C2PA, etc.) ⚠️

Disclosure is minimal here — the weakest link in the Wan series.
- The report only glosses over copyright sourcing with the phrase "internal copyrighted sources," giving no licensed-data percentage, no list of rights-cleared datasets, and no mention of C2PA or any output-side content-provenance standard.
- The only visible provenance mechanism is on the inference side: the API offers a boolean watermark parameter that, when enabled, adds an explicit "AI-generated" watermark text in the bottom-right corner of the video (the default for wan2.7-t2v is false). This corresponds to China's "Measures for Labeling AI-Generated Synthetic Content" (effective September 1, 2025), which requires explicit/implicit labeling; whether implicit labeling (metadata) is written is not documented. [Uncertain]
- The only clear compliance action on the training side is that an internal NSFW safety-assessment model scores and filters the entire training dataset, along with watermark/logo detection and training-time cropping (both a quality action and an ownership action).
- No description of any face/portrait-rights, privacy de-identification, or data-subject opt-out mechanism is present.

### Clip Duration Distribution and Splitting Strategy ⚠️

On the training side (first-hand from Wan 2.1):
- Hard admission threshold: "video duration must exceed 4 seconds" — videos shorter than 4 seconds are removed at the fundamental-dimension stage.
- Training-sample duration: all three stages of joint image-text-video training use 5-second video clips (Stage 1: 192px/16fps; Stage 2: 480px; Stage 3: 720px), and post-training likewise runs on 480px/720px with 5-second clips. That is, the training granularity in the Wan 2.1 era was fixed at "5-second single-shot clips."
- Wan-Dancer's splitting strategy serves as same-team corroborating evidence: original long videos are cut into 5-second clips with 50% overlap between adjacent clips, to strengthen learning of short-term motion dynamics.
On the generation side (official specs for 2.5/2.6/2.7): wan2.5 offers a choice of 5s or 10s; wan2.6 and wan2.7 open this up to any integer in [2,15] seconds (default 5 seconds), with 15 seconds officially described as "the longest single generation available domestically." The shift from a fixed 5s to a continuous 2–15s range implies that training-data splitting has moved from fixed-length clips toward variable-length clips and long multi-shot clips, but the splitting method (PySceneDetect / an in-house shot-aware model) has never been disclosed across the series. [Uncertain]

### Resolution/Aspect Ratio Distribution and Bucketing Strategy ⚠️

No distribution statistics, but a staged threshold strategy is clearly described.
- In Wan 2.1's fundamental-dimension filtering, "resolution thresholds are applied at different training stages to filter out low-quality data" — the resolution bar is not a single cutoff but a dynamic threshold that rises with training stage; this is the core of its bucketing philosophy.
- Resolution curriculum: 256px text-to-image pretraining → joint training Stage 1 (images 256px + video 192px) → Stage 2 (both images and video raised to 480px) → Stage 3 (720px) → post-training jointly on 480px and 720px.
- Letterbox handling: heuristic black-bar detection with automatic cropping, to focus on content-rich regions — equivalent to an aspect-ratio normalization step.
- Generation-side aspect-ratio system (official wan2.7-t2v table): at 720P — 16:9=1280×720, 9:16=720×1280, 1:1=960×960, 4:3=1104×832, 3:4=832×1104; at 1080P — 16:9=1920×1080, 9:16=1080×1920, 1:1=1440×1440, 4:3=1648×1248, 3:4=1248×1648. On the I2V side, "the output aspect ratio tries to match the input image as closely as possible," indicating that training covered a continuous aspect-ratio distribution rather than a few fixed tiers.
The proportion of training samples at each resolution/aspect ratio is undisclosed. [Uncertain]

### Category/Domain Distribution and Mixture Strategy (Balancing People, Actions, Scenes, Styles, etc.) ⚠️

One of the most valuable disclosures in Wan 2.1; undisclosed for 2.5+.
**Cluster-preserved long-tail (core design)**: visual-quality screening is split into two steps — "clustering + scoring." The full dataset is first partitioned into 100 clusters, then a set quota is drawn from each cluster to proceed to the next stage. The report explicitly explains the purpose: to prevent "small but important data segments" from being entirely wiped out by a global threshold filter under a long-tail distribution, thereby preserving the original data distribution. This is a "cluster-unit quota sampling" approach to counteracting the distribution drift introduced by quality filtering — sharing the same motivation as LTX-2's multi-label-network-constrained pair sampling, but achieved through a different mechanism.
**Six-tier motion-quality grading with differentiated sampling rates**: videos are split into six motion-quality categories, each assigned its own sampling priority:
1) Optimal motion (significant motion in layout/perspective/amplitude, clean and smooth) — highest priority;
2) Medium-quality motion (visible motion but with multiple subjects or partial occlusion) — retained to preserve motion diversity and help the model understand spatiotemporal relationships;
3) Static video (mostly chat/interview-type footage, low motion information but high image quality) — identified separately and given "reduced sampling ratio";
4) Camera-dominated motion (aerial footage, etc., subject nearly stationary) — "greatly reduced sampling priority" since it approximates a static image;
5) Low-quality motion (too many subjects, severe occlusion, unclear subjects, e.g., crowded street scenes) — excluded outright;
6) Shaky footage (amateur handheld, motion blur, foreground/background hard to distinguish) — systematically excluded.
**Post-training category balancing**: video post-training explicitly "emphasizes category balance and high diversity," drawing from 12 major categories, with the report naming categories that include technology, animals, arts, humans, vehicles, etc.; for image post-training, beyond taking the top 20% by score, "style and category factors are additionally considered to ensure diversity in the data distribution," and humans manually fill in concepts missing from the dataset to enhance generalization.
**Staged three-axis mixture**: the report's Fig. 3, "Data provisioning across different training phases," explicitly states: for every training stage, the proportions along the three axes of motion, quality, and category are dynamically adjusted based on data throughput. This is the most transferable element of Wan's data methodology — the mixture ratio is not static but rolls with a schedule tied to stage.
**Same-family corroborating evidence**: Wan-Dancer covers five dance genres — Chinese classical dance, K-Pop, Latin, tap, and street dance — and "ensures a near-uniform duration distribution to mitigate class imbalance."
Specific percentages for each category, specific sampling weights for the six motion tiers, and specific quota values for the 100 clusters are all undisclosed. [Uncertain]

### Audio Category Distribution and Mixture (Speech/Foley/Music/Ambient/Silence Ratios and Control Strategy) — AV-Model-Specific Dimension ⚠️

The Wan 2.1 V2A subset provides the sole clear audio-category-screening criteria in the Wan series, while the policy from 2.5 onward has been completely reversed without disclosure — the biggest information gap in this area.
**Stage one (Wan 2.1, 2025.02) — three-way split with speech excluded**: the training data was drawn from the video-generation dataset through "strict filtering," per the rule: systematically remove (1) videos lacking an audio track, and (2) videos containing speech or vocal music, yielding a refined subset of O(1) thousand hours. That is: retained categories = ambient sound + background music; excluded categories = speech, vocals. The report states plainly in its limitations that the model cannot generate laughter, crying, or speech, "primarily attributed to the deliberate exclusion of speech-related data during our data preparation process," and explicitly lists "adding speech-generation capability in the future" as future work.
**Stage two (Wan2.2-S2V, 2025.08) — pivot to speech-specific**: the entire model is built around speaking/singing scenarios, with the data pipeline centered on active-speaker detection, representing a dedicated buildup of speech-related data capability.
**Stage three (Wan 2.5 onward, 2025.09+) — unified generation of all three categories**: officially, 2.6 offers "picture perfectly matched to voice, sound effects, and BGM," meaning speech (voice/dialogue), foley (sound effects), and background music are now generated together within a single model and synchronized with the picture; when no audio is provided, "matching background music or sound effects" are auto-generated, and when audio is provided, it drives lip movement and performance.
**Gap**: the proportions of the four audio categories in the 2.5+ training set, whether they are still processed as separate streams by type (e.g., separate filtering rules, loss weights, or expert routing), and the handling strategy for silent segments are all undisclosed. [Uncertain]
**Inference**: the shift from 2.1's "exclude speech" to 2.5's "lip sync as headline feature" is the largest strategic pivot on the data side in the Wan series, and it coincides precisely with the technical accumulation window of Wan2.2-S2V (2025.08, dedicated to processing speaking-person video) — S2V's human-centric + active-speaker-detection pipeline is very likely the foundation on which 2.5's speech data was built. [Uncertain]

### Narrative Structure Distribution (Single-Shot vs Multi-Shot, Average Clip Duration, Shot-Count Distribution, Native Audio Track Presence) ⚠️

No mixture figures on the training side, but the version-by-version evolution of capabilities lets us infer major changes in data structure.
- Wan 2.1/2.2 era: training samples were fixed 5-second clips, with model capability and evaluation (Wan-Bench) organized entirely around single shots — no multi-shot-narrative concept existed.
- Starting from Wan 2.6: official "multi-shot narrative / shot control" is newly added, explicitly enabled via the API through shot_type:"multi" + prompt_extend:true; officially described as: "using high-level semantic understanding to construct the raw input into a professional-grade multi-shot passage with a complete storyline and narrative tension, maintaining unified modeling of the core subject, scene layout, and environmental atmosphere across seamless shot transitions."
- Wan 2.7: removes the shot_type parameter (setting it has no effect), switching instead to full natural-language control of shot structure — users write "generate a single-shot video" / "generate a multi-shot video," or directly supply a timestamped shot script (official example: "Shot 1 [0-3s] Wide shot: a rainy night on a New York street... Shot 5 [12-15s] Close-up: ..."), and the model decides on its own when unspecified. The prompt-length limit also jumps from 1500 characters in 2.5/2.6 to 5000 characters in wan2.7-t2v.
**Data-side implication**: this interface design in 2.7 strongly suggests that the training captions include "timestamped shot-level structured descriptions" — i.e., training samples have shifted from single-shot clips to "multi-shot segments + per-shot timeline annotation," requiring cross-shot subject/scene-consistency annotation as well. This mirrors LTX-2's "training caption doubles as the inference interface."
**Native audio track**: from 2.5 onward, the entire series is labeled "audio-enabled video" (wan2.6-i2v-flash supports both with-audio and without-audio), indicating that training samples predominantly carry native synchronized audio tracks.
The proportion of single-shot vs. multi-shot, the average shot-count distribution, and the average clip-duration distribution are all undisclosed. [Uncertain]

### Language/Accent Distribution (Data Basis for Multilingual Lip-Sync) ⚠️

No language list or proportions are disclosed anywhere — a link that is clearly weaker for Wan than for LTX-2.
Indirect clues that can be established:
- Text encoder: after ablation, Wan 2.1 selected umT5 (5.3B, multilingual, bidirectional attention); the report's ablation clearly shows umT5 outperforming Qwen2.5-7B-Instruct and GLM-4-9B (even when the latter is given a HunyuanVideo-style bidirectional token refiner), with umT5's multilingual capability being one of the selection rationales.
- OCR and visual text: Wan 2.1's OCR-enhanced caption dataset "currently contains only English and Chinese text"; synthetic text images specifically render Chinese characters to improve Chinese glyph generation.
- Prompt language: the API series-wide "supports Chinese and English," and negative prompts are likewise bilingual.
- 2.6/2.7's official examples include both Chinese lyrics (classical poetry recitation) and English rap singing, indicating coverage of at least bilingual Chinese-English lip sync.
Accent has never appeared as an annotation field or capability dimension in any public Wan material — a stark contrast with LTX-2, which writes the three attributes speaker/language/accent into its caption schema. The data foundation for multilingual lip sync is entirely opaque. [Uncertain]

## Cleaning Pipeline

### Overall Cleaning Funnel Structure (Number of Filter Stages, Ordering) ⚠️

Undisclosed for 2.5/2.6/2.7. Chapter 3 of the Wan 2.1 report gives a complete four-step funnel that is rare in the industry, and is the single most reusable asset in this entry; Wan2.2-S2V provides a hierarchical funnel dedicated to human-centric/audio-visual data.
**Wan 2.1 pretraining four-step cleaning process**
Step 0, candidate-pool construction: internal copyrighted data + publicly accessible data, first curated and deduplicated.
Step 1, "fundamental dimensions" — efficient coarse screening on properties intrinsic to the source data, comprising 8+ lightweight detectors run in parallel: OCR text-coverage detection, LAION-5B aesthetic classifier pre-screening, an internal NSFW safety model, watermark and logo detection, black-bar detection, overexposure detection, synthetic-image detection, blur detection, and duration/resolution thresholds. This step "successfully eliminated approximately 50% of the initial dataset."
Step 2, "visual quality" — semantically driven fine screening, split into "clustering + scoring": first split into 100 clusters with per-cluster quota sampling to preserve the original distribution and avoid losing the long tail, then a full-dataset scoring pass by an expert evaluation model trained on 1–5 manual annotations.
Step 3, "motion quality" — six-tier motion-quality grading, with retain/downsample/exclude actions applied per tier.
Parallel branch, "visual text data" — two sub-paths: rendering Chinese characters on a white background to synthesize hundreds of millions of text-containing images, plus real-world text-containing images recognized by multiple OCR models and then fed to Qwen2-VL to generate natural-language descriptions containing the exact text content.
Step 4, post-training curation (Section 3.2): images go through "expert model top 20% + manual curation"; video goes through "visual-quality classifier top selection + motion-quality classifier splitting into simple/complex motion + 12-category class balancing."
Final, annotation stage (Section 3.3): an in-house LLaVA-style dense-caption model re-labels the entire dataset of images and video.
**Structural characteristics**: a typical five-stage design — "cheap parallel coarse screening (cuts ~50%) → cluster-quota + learned-scoring fine screening → motion-semantic-graded resampling → high-quality subset curation → full re-captioning"; discriminators are predominantly small expert models, with MLLMs appearing only in the captioning and evaluation stages.
**Wan2.2-S2V's hierarchical human-centric funnel (paper Fig. 1)**
1) Data collection: automatic coarse screening of open-source datasets (captions containing human-related descriptions) + manual curation of videos containing complex human activities such as speaking/singing/dancing → a human-centric video pool at the million scale;
2) Pose tracking: VitPose extracts 2D poses and converts them to DWPose, serving both as a control signal and as a screening basis;
3) Fine-grained filtering: videos are removed where the person's temporal or spatial footprint is too small; only videos where the face remains consistently visible throughout the full sequence are retained (to ensure facial expressions can be learned from audio);
4) Five video-quality metrics: Dover clarity, UniMatch optical-flow motion score, a Laplacian operator checking sharpness of the face/hand regions, an improved aesthetic predictor, and OCR-based caption-occlusion detection;
5) Audio-video alignment: Light-ASD active-speaker detection, removing videos where the audio is out of sync with the active speaker, or where no active speaker is present in the frame;
6) Dense annotation: QwenVL2.5-72B generates structured dense captions.

### Funnel Quantitative Retention Rates (Input/Output Volume per Stage and Final Retention, e.g. Apollo's 27%) ⚠️

Quantitative disclosure exists in only two places, insufficient to construct a complete funnel ledger:
1) Wan 2.1's fundamental-dimension stage: "we successfully eliminated approximately 50% of the initial dataset" — the first parallel coarse-screening stage eliminates roughly 50%, retaining 50% for semantic fine screening. This is the only clear inter-stage retention rate in the Wan series.
2) Wan 2.1's post-training image curation: the top 20% by expert-model predicted score is taken.
For all other stages (the exact quota drawn per cluster, the visual-quality score threshold, the retention ratio for each of the six motion tiers, the input/output volume ratio around V2A audio filtering, and S2V's elimination rate at each stage), no input/output volume or retention rate is given. On the V2A side, only the outcome (O(1) thousand hours) and the parent-pool scale ("billions of videos") are known; a rough conversion would put the retention rate far below 1%, but the report gives no comparable methodology, so this cannot be treated as an authoritative funnel figure. Overall, this cannot be aligned with quantitative-funnel figures like Apollo's 27%. [Uncertain]

### Shot Segmentation Method (PySceneDetect/In-House Model/Shot-Aware Splitting) ⚠️

Across the entire series (2.1 through 2.7), the shot-segmentation method has never been disclosed — neither open-source tools such as PySceneDetect / TransNetV2 nor whether an in-house shot-aware splitting model or threshold parameters exist are mentioned. Only the segmentation granularity is confirmed: Wan 2.1's training samples were 5-second clips; Wan-Dancer explicitly cuts original video into 5-second clips with 50% overlap between adjacent clips (the only concrete splitting strategy written out by the same team, though specific to the dance vertical). Wan 2.6/2.7's multi-shot-narrative capability necessarily requires training data with shot-boundary annotation and cross-shot-consistency annotation; its splitting and shot-level alignment scheme is one of the most important unknowns in this entry. [Uncertain]

### Quality Filtering (Aesthetic Scoring, Sharpness, OCR Text Filtering, Letterbox/Watermark/Logo Detection) ⚠️

Wan 2.1's quality filtering is among the most completely enumerated in publicly available industry material (undisclosed for 2.5+, presumably carried forward and upgraded).
**Fundamental-dimension parallel detection (reconstructed faithfully to the original text, item by item)**
1) Text detection: a lightweight OCR detector quantifies "text coverage," removing videos and images with excessive text elements to preserve visual clarity;
2) Aesthetic evaluation: uses the industry-standard LAION-5B aesthetic classifier for a preliminary quality assessment, quickly filtering out low-quality data;
3) NSFW scoring: an internal safety-assessment model computes an NSFW score for all training data and filters accordingly;
4) Watermark and logo detection: detects whether watermarks/logos are present and crops these regions out at training time (note: this is a crop, not a discard — a "recoverable" treatment);
5) Black-bar detection: heuristic methods automatically crop away excess black bars, focusing on content-rich regions;
6) Overexposure detection: a self-trained expert classifier identifies and removes data with abnormal tonal distributions;
7) Synthetic-image detection: a self-trained expert classifier removes AI-generated images. The report offers a highly valuable empirical finding here — "even a very small proportion (<10%) of generated-image contamination can significantly degrade model performance" — a direct quantitative claim about the importance of data purity;
8) Blur detection: an in-house model scores training material for blurriness, systematically removing visually unclear content;
9) Duration and resolution: duration must exceed 4 seconds; resolution thresholds rise dynamically with training stage.
**Learned visual-quality scoring**: samples drawn from each cluster are handed to human annotators to score on a 1–5 scale (1 worst, 5 best); the full set of annotated data is used to train an expert evaluation model, which then scores the entire dataset, used for data selection and processing at various stages.
**Wan2.2 increment**: introduces "carefully curated aesthetic data with fine-grained labels for lighting, composition, contrast, and color tone," enabling the model to generate controllably by cinematic aesthetic style — an important upgrade from "a single aesthetic score" to "multi-dimensional controllable aesthetic labels," and also the data foundation for the "cinematic-grade" positioning of 2.5+.
**Wan2.2-S2V's five video-quality metrics**: Dover clarity assessment, UniMatch optical-flow motion stability (filtering out samples where subject/background motion is excessive), a Laplacian operator specifically checking whether the face and hand regions are blurred, an improved aesthetic predictor, and OCR detection of whether captions occlude the face or hands. The two designs targeting human-centric scenes — "separately checking sharpness of face/hand regions" and "detecting caption occlusion of the face" — are especially worth borrowing.

### Motion Filtering (Optical Flow/Motion Score Thresholds, Static and Shaky Footage Removal) ⚠️

Wan 2.1 elevates motion quality to an independent funnel stage on par with visual quality, aiming to "select videos with natural, complete, and significant motion while avoiding static and shaky footage."
**Six-tier grading with differentiated handling**: optimal motion (retained, highest priority) / medium-quality motion (retained, to preserve motion diversity and help the model understand spatiotemporal relationships) / static video (chat/interview type, high image quality but low motion → identified separately and given reduced sampling ratio) / camera-dominated motion (aerial footage etc., near-static image → greatly reduced sampling priority) / low-quality motion (too many subjects, severe occlusion, unclear subjects → excluded) / shaky footage (amateur handheld, motion blur, foreground/background hard to distinguish → systematically excluded).
**Key point**: static and camera-dominated motion are not deleted but "downsampled," reflecting a consistent philosophy of "preserving distribution without losing concepts"; only low-quality motion and shaky footage are actually removed.
**Method**: Wan 2.1 does not specify the specific algorithm used for the six-tier grading (whether optical flow, or a trained classifier), nor does it give any threshold values. Same-family corroborating evidence: Wan2.2-S2V uses UniMatch to predict optical flow and compute a motion score; Wan-Dancer uses SEA-RAFT to generate optical-flow masks (used in the loss function, not for filtering); Wan-Bench's evaluation side uses RAFT optical-flow magnitude to measure large-motion generation capability. [Uncertain: thresholds and algorithm]

### Deduplication Method (Exact Deduplication and Embedding-Based Semantic Deduplication Recorded Separately) ⚠️

Disclosure is extremely thin. The Wan 2.1 report only mentions it in passing, in the data-sourcing sentence: "We curated and deduplicated a candidate dataset sourced from…" — confirming that deduplication exists and occurs at the very front of the funnel (candidate-pool construction), but the method is entirely unspecified: neither exact deduplication (hashing/fingerprinting) nor semantic deduplication (embedding clustering) is distinguished, and no similarity threshold, deduplication ratio, or within-cluster retention policy is given.
One point that could be misread: the "clustering" (100 clusters) in Step 2, visual quality, is for quota sampling to preserve distribution — the report explicitly states its purpose is to prevent long-tail loss, not deduplication, and should not be conflated with it.
2.2/2.5/2.6/2.7 have no deduplication information whatsoever. [Uncertain]

### VLM/LLM as Quality Judge (Multimodal LLM Quality Scoring and Mismatch Removal, the 2026 Shift from Shallow Scorers to LLM-Level Semantic Judgment) ⚠️

The Wan series shows a clear division of labor — "small expert models handle QC, MLLMs handle captioning and evaluation" — with no public evidence yet that its data funnel has shifted toward large-model semantic judgment.
**Discriminators within the Wan 2.1 data funnel are all dedicated/shallow models**: a lightweight OCR detector, the LAION-5B aesthetic classifier, an internal NSFW safety model, a watermark/logo detector, heuristic black-bar detection, an overexposure expert classifier, a synthetic-image expert classifier, an in-house blur-scoring model, a manually-annotated-and-trained 1–5 visual-quality expert evaluation model, and a motion-quality grading model.
**The three places MLLMs appear, none of which are QC**:
1) Annotation: an in-house LLaVA-style dense-caption model (ViT + two-layer MLP + Qwen LLM) re-labels the entire dataset; the visual-text branch uses Qwen2-VL to convert OCR results into natural-language descriptions; V2A uses Qwen2-Audio to generate audio captions; Wan2.2-S2V uses QwenVL2.5-72B for annotation.
2) Verification within data construction: an LLM extracts (category, count) pairs from text, then Grounding DINO performs an actual count, and only samples where the two agree are kept — this is the closest the Wan series comes to "model as judge," a cross-verification-style weak QC step.
3) Evaluation: Wan-Bench makes extensive use of Qwen2-VL to judge complex tasks such as physical plausibility, motion smoothness, stylization, object/count/spatial-relation, complex camera moves, and action-instruction adherence; only simple tasks (such as object detection) use traditional detectors. This shows the team already has the full engineering capability to use MLLMs for semantic judgment — it simply isn't shown being used for training-data filtering in public materials.
Whether 2.5/2.6/2.7 have introduced VLM-based QC is unknown; given the demands that capabilities such as "role-play" and "multi-shot consistency" place on semantic-level image-text/audio-visual matching, it is quite likely they have, but there is no evidence either way. [Uncertain]

### Safety and Compliance Filtering (NSFW, Copyright, Face/Privacy) ⚠️

Disclosure is limited, concentrated on a single action on the training-data side and compliance labeling on the inference side.
**Training-data side**
- NSFW: an internal safety-assessment model computes an NSFW score for "all training data" and systematically filters inappropriate content (a fixed step in Wan 2.1's fundamental-dimension stage).
- Copyright/ownership: only constrained on the acquisition end via "internal copyrighted sources," plus watermark/logo detection and training-time cropping — no downstream copyright-classifier description.
- Face/privacy: no general-purpose privacy de-identification is described. Face processing is concentrated in constructing the personalization subset (1FPS face detection, discarding any frame where multiple faces are detected, discarding samples where over 10% of frames have no face, ArcFace inter-frame similarity for identity-consistency screening, face segmentation to remove background, and facial-landmark detection used for canvas alignment) — these are capability-building actions rather than privacy-protection actions; small faces are deliberately not filtered out, because such videos typically contain full-body figures.
- No mention of CSAM removal, red-teaming processes, or celebrity-portrait restrictions (to the contrary, the caption model was specifically trained on a dataset for recognizing thousands of identities among celebrities/landmarks/film-and-TV characters).
**Inference side**: the API's watermark parameter (a fixed "AI-generated" text in the bottom-right corner); a negative-prompt mechanism; prompt_extend rewriting; commercial service is subject to China's Generative AI Service Management Measures and the content-labeling measures.
2.5/2.6/2.7's safety-filtering strategy is entirely undisclosed. [Uncertain]

## Annotation Methods

### Caption Model Used (In-House VLM/Open-Source Model, Model Scale) ⚠️

Wan 2.1 devotes an entire section (3.3, Dense Video Caption) to describing its in-house caption model, with a level of detail that ranks among the most complete of comparable reports; undisclosed for 2.5+.
**Motivation**: raw web-text descriptions are "too simplistic to convey detailed visual content." Citing the finding from DALL-E 3 that training with highly descriptive, generative captions significantly improves a generative model's prompt-following ability, the team built an in-house caption model to re-label every image and every video in the dataset.
**Architecture**: LLaVA-style — a ViT visual encoder extracts image and video-frame embeddings → a two-layer perceptron projects them → fed into a Qwen LLM.
- Image side: LLaVA-style dynamic high resolution, splitting an image into up to 7 patches, with each patch's visual embedding adaptively pooled to a 12×12 grid to reduce compute.
- Video side: 3 FPS sampling, capped at 129 frames; a slow-fast encoding strategy is used — every 4th frame retains full resolution while the remaining frames' embeddings undergo global average pooling. The report gives a quantitative benefit: under a limited visual-token budget, VideoMME (no subtitles) improved from 67.6% to 69.1%.
**Three training stages**: Stage 1 freezes the ViT and LLM, training only the MLP for vision-language spatial alignment (lr=1e-3); Stage 2 trains all parameters; Stage 3 trains end-to-end on a small amount of high-quality data (LLM and MLP lr=1e-5, ViT lr=1e-6).
**Training data**: open-source vision-language datasets (captions + visual question answering, covering actions/counting/OCR) + pure-text instruction data (enabling the caption model to produce captions in a specific style or content per user instruction) + a large amount of self-built data (see caption_structure).
**Effect**: using a self-built automated evaluation pipeline (based on CAPability), 1,000 videos were randomly sampled per dimension across 10 dimensions and compared against Gemini 1.5 Pro via F1: the in-house model performs better on event (88.0 vs 52.6), camera angle (89.1 vs 52.6, the largest margin), camera motion, style, and object color; Gemini performs better on action, OCR, scene, object category, and counting. Overall it is on par with Gemini 1.5 Pro.
**Parameter scale**: the specific sizes of the ViT and Qwen LLM are not disclosed. [Uncertain]
**Other captioning models in the same family**: Wan2.2-S2V uses QwenVL2.5-72B directly for annotation; Wan 2.1's V2A audio captions use Qwen2-Audio; the visual-text branch uses Qwen2-VL.

### Caption Density and Structuring (Short/Long/Dense Descriptions, Structured Fields such as Camera Motion, Style Tags) ⚠️

Follows a "dense caption + ten semantic dimensions" approach, with structured information folded into natural language rather than expressed as explicit JSON fields.
**Ten core dimensions**: reconstructed from the caption-evaluation system as inferred training targets — action, camera angle, camera motion, object category, object color, object counting, OCR text, scene, style, event. The team specifically notes that contemporary MLLMs (including GPT-4o and Gemini Pro) generally perform poorly at predicting camera angle and motion, so they manually annotated a dedicated camera-angle-and-motion dataset — used both to directly train the caption model and to train an expert model that batch-expanded labels for more videos used in earlier training stages. The report explicitly states this improved the downstream video-generation model's camera-motion controllability.
**Wan2.2-S2V's explicit annotation specification (the closest thing to an official "caption template")**: QwenVL2.5-72B is required to describe three aspects in detail: (1) camera angle — eye-level, high-angle, low-angle, wide shot, medium shot, close-up; (2) the character's appearance features (clothing, accessories) and actions, which must be decomposed into specific sub-actions; (3) the main features of the background environment, including architectural style, color scheme, greenery, etc. It also explicitly requires the model to "avoid subjective evaluations and emotional interpretations, as these are not useful for the target generated video" — closely matching LTX-2's "comprehensive yet factual" style guideline, suggesting this is an industry-wide consensus annotation norm.
**Capability-oriented data for the caption model (reflecting structuring demands)**: celebrity/landmark/film-and-TV-character recognition (thousands of identities, an LLM collects names → a TEAM CLIP-style model retrieves candidates → keyword matching provides double-checked denoising); object counting (numeric-keyword retrieval → an LLM extracts (category, count) pairs → Grounding DINO verifies); OCR-enhanced caption (OCR extracts text as a prior before the model generates a description; currently Chinese and English only); camera angle and motion; fine-grained categories (animals/plants/vehicles, millions of images); spatial-relationship understanding (left/right/up/down, leveraging object-detection datasets); re-captioning (expanding existing short labels/short captions into dense descriptions); edit-instruction captioning (describing the difference and transformation between two images); image-group description (writing shared traits first, then describing each image individually); and manually annotated dense captions for images and video (the highest quality, used for the final training stage).
**Length and density**: no distribution of caption word counts is given. Corroborating evidence from the generation side: the prompt-length limit rose progressively from 800 characters (wan2.2/wanx2.1) → 1500 characters (wan2.5/2.6) → 5000 characters (wan2.7-t2v), suggesting training captions have also grown steadily longer and denser.

### Joint Audio-Video Caption Structure (Whether Visual + Auditory Tracks Are Covered Jointly, Whether Split into Separate Fields, e.g. LTX-2's Full Soundscape Description, Script-a-Video's Factorized Streams, Foley-Omni's Three Fields) ⚠️

Wan 2.1's V2A module provides a clearly defined three-part joint audio-video caption structure, the most valuable first-hand design in this entry; but the schema used since the shift toward speech and lip sync from 2.5 onward is entirely undisclosed.
**Wan 2.1 V2A three-component structured caption**: per the report's original text, the final structured caption integrates three parts — (1) dense video description; (2) ambient sound characterization; (3) background music analysis, the latter characterized by four attributes: style, rhythm, melody, and instrumentation.
**Example (a real template from paper Fig. 32)**: "The video description: a horse is running. This audio contains ambient sound: the sound of clip-clop. This audio does not contain music." — this shows it is a natural-language string assembled from a fixed sentence template, and explicitly supports a negative statement of "no music," letting the model learn that "the presence or absence of music" is itself a controllable condition.
**Generation method**: the visual side reuses the in-house dense video-caption model; the audio side has Qwen2-Audio generate a dedicated audio caption, categorized into ambient sound and music. This is a split-stream design where "a visual captioner and an audio captioner each produce output, which is then concatenated per a fixed schema" — differing from LTX-2's "single captioner producing a fused, long, full-soundscape description," and closer to Foley-Omni's three-field approach.
**Random masking during training**: the implementation details explicitly state that during training, the ambient-sound caption and the music caption are each randomly omitted with a preset probability. The purpose is to force the model to build robust cross-modal associations purely from visual cues, while retaining controllability when text is present. This "randomly dropping caption components" technique is the most worth-borrowing training-side companion design in this schema.
**Gaps**: the length ratio among the three text types, the specific attribute vocabulary for the audio caption, and whether there is a quality-verification step are all undisclosed; how the schema expands after speech is introduced from 2.5/2.6/2.7 onward (whether a dialogue-transcription field or speaker field is added) is entirely unknown. [Uncertain]

### Dialogue Transcription and Speaker Attribute Annotation (ASR Transcription + Speaker Identity/Language/Accent/Emotion) ⚠️

There is a clear capability-versus-disclosure gap here — one of the largest information gaps in this entry.
**Wan 2.1**: speech is explicitly excluded — the V2A training set "systematically removes videos … containing speech/vocal music," so no ASR transcription or speaker-attribute annotation step exists at all; the report explicitly acknowledges this results in the model being unable to generate laughter, crying, or speech.
**Wan2.2-S2V**: processes speaking-person video, but the paper's data chapter only performs "active speaker detection" (Light-ASD) and face-visibility screening, with no mention of dialogue ASR transcription or speaker identity/language/accent/emotion annotation; its dense-caption specification (camera angle/appearance-and-action/background environment) also contains no auditory field.
**Wan 2.5/2.6/2.7**: capability-wise, these already support precise lip sync, Chinese/English dialogue recitation and singing, and audio-driven lip movement and performance; 2.6 further supports "role-play" (uploading a personal video to replicate the person's likeness and voice) — the latter effectively requires timbre/voiceprint-level speaker modeling. These capabilities necessarily correspond, on the data side, to a complete pipeline of ASR transcription + speaker diarization + voiceprint/timbre annotation, but Alibaba has never disclosed anything about it: no ASR model selection, no transcription-quality metrics, no speaker-attribute field definitions, no language/accent annotation.
By contrast, LTX-2 explicitly writes "precise transcription + speaker + language + accent" into its caption schema. On this dimension, the Wan series is a complete black box. [Uncertain]

### Geometric and Structured Annotation (Camera Parameters, Depth, 3D Point Tracks, Action Labels, Explicit State) ⚠️

The Wan series invests more heavily in structured annotation than most peers, and most of it serves "controllable generation" rather than pure QC.
**Pose**: Wan2.2-S2V — VitPose tracks each character's 2D pose and converts it to DWPose format, explicitly serving two purposes: (1) as an optional multimodal control signal fed into the generation model, achieving precise temporal alignment with human body motion; (2) as a fine-grained screening criterion (removing videos where the person's spatiotemporal footprint is too small).
**Camera**: Wan 2.1 — a batch of videos was manually annotated with camera angle and camera motion, used on one path to directly train the caption model at the final training stage, and on another to train an expert model that batch-expands labels over more videos (for use in earlier training stages). This is a representative example of the Wan series' "manual seed labeling → expert-model expansion → full-scale coverage" leverage-annotation pattern; the report explicitly attributes an improvement in the generation model's camera-motion controllability to it. Wan 2.1 also has a dedicated camera-motion-controllability extension module (Section 5.5).
**Face**: personalization data — 1FPS face detection, ArcFace inter-frame similarity, face segmentation (removing background interference), and facial-landmark detection (used for canvas alignment during training).
**Image-text consistency/temporal-consistency metrics**: SigLIP feature cosine similarity is used as a data-screening yardstick for three tasks — I2V data requires the similarity between the first frame's features and the mean of the remaining frames' features to exceed a threshold (value undisclosed); video-continuation data requires the SigLIP feature similarity between the preceding 1.5-second segment and the following 3.5-second segment to meet a bar; first-and-last-frame transition data does the reverse, raising the proportion of samples with large first/last-frame differences.
**Optical flow**: Wan-Dancer uses SEA-RAFT to generate optical-flow masks fed into the loss function; S2V uses UniMatch optical flow to compute a motion score.
**Spatial relations and counting**: object-detection datasets are used to construct left/right/up/down spatial-relation data; Grounding DINO performs count verification.
**Not seen**: camera intrinsics/extrinsics, depth maps, 3D point tracks, and explicit physical-state annotation are all absent. [Uncertain]

### Synthetic Data Construction (Controlled Perturbation/Edit-Constructed Training Pairs, e.g. InstructAV2AV) ⚠️

Wan 2.1 clearly uses synthetic data and provides two sizable concrete cases; however, there is nothing resembling InstructAV2AV's "controlled-perturbation-constructed audio-visual editing training pairs."
1) Visual-text synthesis (hundreds-of-millions scale): Chinese characters were rendered on a plain white background, synthesizing "hundreds of millions" of text-containing images. This is mixed during pretraining with real-world text-containing images (recognized for Chinese/English text by multiple OCR models, then fed to Qwen2-VL to generate descriptions with exact text content). The report states this combination lets the model generate rare characters in videos with accurate glyphs and high realism, giving it a significant lead in visual-text generation.
2) Face-diversity synthesis (O(1)M scale): from the O(10)M personalization videos, roughly O(1)M were randomly sampled, and Instant-ID was used to synthesize diverse faces for each; a text-prompt template library of 100+ prompts was built (covering anime, line-art, cinematic, Minecraft, and other styles), with each generation randomly drawing one prompt plus a random human pose estimated from another video, both fed as Instant-ID inputs; after generation, ArcFace similarity was used to filter out low-quality samples. The report states this substantially increased the personalization dataset's diversity in style, pose, lighting, and occlusion.
3) A reverse constraint: the synthetic-image-detection expert classifier actively removes AI-generated images of unknown origin, because <10% contamination significantly degrades performance — i.e., "self-built controllable synthetic data is usable, but wild AI-generated images must be removed" is a clear line in Wan's overall data philosophy.
Whether 2.5/2.6/2.7 use synthetic audio-video pairs (such as deliberately misaligned dubbing to construct negative samples) is unknown. [Uncertain]

### Degree of Human Involvement (Manual Annotation, Manual QC, Model Pre-Screening + Manual Review)

Human involvement follows a leverage-style design of "a small amount of high-value manual annotation → training an expert model → full-scale automation," appearing in six places:
1) Seed annotation for the visual-quality scorer: samples drawn from each cluster are scored 1–5 by human annotators (1 worst, 5 best); all annotated data is used to train an expert evaluation model, which then scores the entire dataset. This is the cornerstone of Stage 2 of the entire funnel.
2) Seed annotation for camera angle and motion: a batch of videos is manually annotated, used both to directly train the caption model and to train an expert model for batch label expansion.
3) Manual curation of post-training data: on the image side, beyond the expert-model top 20%, there is also "manual collection" — humans collect top-tier images from various categories and sources, and actively fill in concepts missing from the dataset to enhance generalization.
4) Manual dense annotation within caption training data: the report states that manually annotated dense captions for images and video are "our highest-quality caption data," used exclusively for the model's final training stage.
5) Wan2.2-S2V's manual data collection: humans curate videos containing deliberate and complex human activities (speaking, singing, dancing) from publicly accessible sources, running in parallel with automatic coarse screening to form the initial video pool.
6) Evaluation side: Wan-Bench trains a YOLOv3-type model to detect AI-generated artifacts, using 20,000 manually annotated AI-generated images as training data; the final composite score uses a "human feedback guided weighting strategy" to determine each dimension's weight.
No description exists of a per-sample manual review or spot-check process for captions; the scale of human involvement for 2.5+ is unknown.

## Audio-Video Alignment

### Audio-Video Sync Detection Method (Lip Sync, Event Alignment) ⚠️

The one place the Wan series clearly spells out an audio-visual-sync data-filtering method is the Wan2.2-S2V paper — the most valuable first-hand information in this entry for the research topic; undisclosed for 2.5/2.6/2.7 themselves.
**Wan2.2-S2V's approach (paper Section 2, end of "Pose Tracking and Fine-grained Filtering")**: original text: "to address audio-visual alignment challenges, we utilized Light-ASD to detect and exclude videos where (1) the audio is not synchronized with the active speaker, or (2) no active speaker exists in the scene."
That is, Light-ASD (a lightweight active-speaker-detection model from CVPR 2023, Liao et al. 2023) serves as the audio-visual-alignment discriminator, applying two exclusion rules:
(1) the audio is out of sync with the active speaker on screen → excluded (directly corresponding to dubbing, post-hoc audio addition, voiceover, and other asynchronous samples);
(2) no active speaker exists on screen at all → excluded (corresponding to narration, off-screen audio sources, and samples where audio and video have no causal relationship).
Together, these two rules effectively accomplish both "removing temporally out-of-sync samples" and "removing samples where the sound source is not on screen" — a fairly clean design.
**A companion prerequisite**: the same passage also requires "retaining videos where the face remains consistently visible throughout the full sequence," with the rationale explicitly stated as "to ensure the model can learn audio-driven facial expressions from the given audio signal" — that is, the precondition for learning lip sync is that the face is visible throughout, a data-admission condition complementary to sync detection.
**Wan 2.1 V2A's alignment handling (architecture side, not filtering side)**: there is no sync-detection filter at all; instead, temporal alignment is ensured by three design choices: (1) abandoning the "mel-spectrogram + image-style VAE" route, because DiT needs to slice the 2D latent into patches and reshape them into (Ha·Wa)×Ca, a process that would break temporal alignment with the video content; instead, a 1D-VAE is trained directly on the raw waveform, producing a Ta×Ca latent that explicitly preserves the time axis, which the report states is "necessary for precise synchronization." (2) CLIP is used to extract per-frame visual embeddings, then feature replication performs a temporal-rate adaptation to match the audio feature sampling rate, followed by linear projection and element-wise addition for fusion. (3) In implementation, input video is downsampled to a fixed ratio of "12 seconds corresponding to 48 frames," to ensure frame-level precise synchronization.
**2.5/2.6/2.7**: only marketing statements such as "native audio-visual sync" and "audio directly drives character lip movement and performance" exist — no detection method, model, or process is disclosed. [Uncertain]

### Specific Sync-Detection Metrics and Thresholds (SyncNet/Synchformer/LSE/In-House, Threshold Values, e.g. MOVA's LSE-D≤9.5 and LSE-C≥4.5, SkyReels-V4's SyncNet |offset|≤3 ∧ conf>1.5) ⚠️

Training-data-filtering thresholds are entirely undisclosed across the series; the evaluation side offers one comparable figure.
**Filtering thresholds**: Wan2.2-S2V states only that Light-ASD "detects and excludes" out-of-sync samples, giving no confidence threshold, time-offset tolerance window (such as |offset|≤3 frames), or any numeric value; Wan 2.1's V2A has only categorical rules ("exclude no-audio-track, exclude speech/vocal-containing"), with no numeric threshold; 2.5/2.6/2.7 give none. As a result, this cannot be aligned with works that give explicit thresholds, such as MOVA (LSE-D≤9.5 and LSE-C≥4.5), SkyReels-V4 (SyncNet |offset|≤3 ∧ conf>1.5), or UniTalking (SyncNet conf>0.9).
**Evaluation metric**: on the EMTD dataset, Wan2.2-S2V uses Sync-C (derived from the SyncNet confidence measure of Chung & Zisserman 2017) as a lip-audio sync-quality metric; measured results: Wan-S2V 4.51, compared with HY-Avatar 4.71 (higher), EMO2 4.58 (higher), EchoMimicV2 4.44, FantasyTalking 3.00, and MimicMotion 2.68. That is, on the lip-sync metric alone, Wan-S2V is not the top performer (slightly behind HunyuanVideo-Avatar and EMO2); its advantage instead shows up in image quality and identity consistency: FID 15.66 (best), FVD 129.57, SSIM 0.734 (best), PSNR 20.49 (best), CSIM 0.677 (best), HKC 0.435, HKV 0.142, EFID 0.283.
**No sync dimension in Wan-Bench**: none of the 14 fine-grained metrics across Wan-Bench's three major dimensions covers audio-visual sync; audio-visual-sync capability has never entered its main benchmark. 2.5/2.6/2.7 publish no sync-benchmark scores whatsoever. [Uncertain]

### Separation of Temporal Sync vs Semantic Sync (Time Alignment and Content-Semantic Matching as Two Independent Filter Conditions) ⚠️

The Wan series distinguishes "temporal alignment" from "semantic/emotional matching" at the design-intent level, but does not implement the two as two independent data-filtering conditions.
**Wan 2.1 V2A's explicit two-way framing**: the report opens by splitting the V2A objective into two parallel requirements: (1) ambient sound "must be temporally aligned with the visual content of the video"; (2) the accompanying music "should accurately reflect the emotional tone and contextual setting of the video." That is, ambient sound/sound effects correspond to temporal synchronization, while music corresponds to semantic and emotional matching — a fairly clear binary split, directly reflected in the three-part caption structure (the ambient-sound field carries temporal events, the music field carries stylistic/semantic attributes such as rhythm/melody/instrumentation).
**Division of labor at the mechanism level**: the temporal side is handled by the 1D-VAE preserving the time axis + CLIP frame-level feature replication for alignment + the fixed 12-second/48-frame ratio; the semantic side is handled by umT5 text conditioning and the music caption; the framework also lets users specify "on-screen / off-screen" sound and the music's style and presence via text — essentially exposing "whether the sound source is on screen," a semantic condition, explicitly to the user.
**On the filtering side**: Wan2.2-S2V's two Light-ASD rules implicitly carry the same binary split — rule (1), "audio out of sync with the active speaker," is a temporal condition, while rule (2), "no active speaker in the scene," is a semantic/sound-source-attribution condition; however, the paper does not frame them as two independent filtering dimensions.
Undisclosed for 2.5/2.6/2.7. [Uncertain]

### Audio Quality Filtering (SNR, Silence Detection and Silence-Ratio Thresholds, No-Soundtrack Removal, Off-Screen Sound-Source Removal, Background Music Separation) ⚠️

On the training side, only category-level rules exist, with no signal-level quantitative thresholds.
**Wan 2.1 V2A**: the only audio-side filtering consists of two exclusion rules — removing videos lacking a soundtrack, and removing videos containing speech or vocal music. The "no-soundtrack removal" rule serves the function of clearing silent samples. No SNR threshold, loudness/RMS threshold, silence-ratio threshold, or audio-event-density metric is given; it is not stated whether background-music/vocal source separation is performed, nor are there admission requirements for original audio-track sample rate or channel count.
**Wan2.2-S2V**: indirectly removes off-screen-audio/no-on-screen-sound-source samples via Light-ASD, but likewise gives no audio-signal quality metrics (no SNR, no silence detection).
**Corroborating technical specs**: Wan 2.1 V2A output is 44.1kHz stereo, up to 12 seconds; audio is compressed directly in the waveform domain via the 1D-VAE.
**Inference-side input-audio constraints (can reflect training-side spec preferences)**: wan2.5/2.6's audio_url requires wav/mp3, 3–30 seconds, ≤15MB; wan2.7 relaxes this to 2–30 seconds. Any portion beyond the duration is automatically truncated and discarded; when audio is shorter than video duration, the portion beyond the audio length outputs as a silent video (e.g., 3 seconds of audio paired with a 5-second video yields 3 seconds with sound followed by 2 seconds of silence) — this behavior implies the model is conditionally tolerant of "partial silence," possibly corresponding to the presence of samples with silent segments in the training data.
2.5/2.6/2.7's audio-quality-filtering strategy is undisclosed. [Uncertain]

### Classification and Differentiated Handling of Speech/Sound Effects/Music ⚠️

The Wan series has a clear divide-and-treat strategy for audio types, and it has undergone a significant strategic reversal.
**Stage one (Wan 2.1, 2025.02) — three-way split with speech excluded**
- Ambient sound: retained, given its own dedicated segment in the caption;
- Background music: retained, given its own dedicated segment in the caption, analyzed by four attributes — style/rhythm/melody/instrumentation;
- Speech and vocal singing: the entire video is removed from the V2A training set outright.
These three categories are generated and classified by Qwen2-Audio audio captioning, with random masking applied to the ambient-sound and music caption segments during training to strengthen the visual-to-audio association-learning. The report states outright that this causes the model to be unable to generate laughter, crying, or speech, and lists speech generation as future work.
**Stage two (Wan2.2-S2V, 2025.08) — pivot to speech-specific**: the entire model is built around speaking/singing scenarios, with the data pipeline centered on active-speaker detection — a dedicated buildup of speech-related data capability.
**Stage three (Wan 2.5 onward, 2025.09+) — unified co-generation of all three categories**: officially, 2.6 offers "picture perfectly matched to voice, sound effects, and BGM," meaning speech (voice/dialogue), sound effects (foley), and background music are generated together within one model, synchronized with the picture; if no audio is provided, "matching background music or sound effects" are auto-generated, and if audio is provided, it drives lip movement and performance.
**Gaps**: the proportions of the three audio categories in the 2.5+ training set, whether they are still processed by type as separate streams (e.g., separate filtering rules, loss weights, or expert routing), and the handling strategy for silent samples are all undisclosed. [Uncertain]

## Training Integration

### Multi-Stage Training Curriculum and Data Curriculum Scheduling (Stage Divisions by Resolution/Duration/Quality Score/Modality; Low-Res→High-Res, Image→Video, Short→Long) ⚠️

Wan 2.1 gives a complete resolution-modality progressive curriculum (undisclosed for 2.5+):
**Problem diagnosis**: the report first explains why direct high-resolution long-video joint training is infeasible: (1) overly long sequences (81 frames at 1280×720) severely lower training throughput, so under a fixed GPU-hour budget, data throughput becomes insufficient and convergence is impeded; (2) excessive memory usage forces smaller batch sizes, producing spikes in gradient variance that cause training instability.
**Stage divisions**
- Stage 0, image pretraining: the 14B model first undergoes 256px low-resolution text-to-image pretraining, aiming to establish cross-modal semantic-text alignment and geometric-structure fidelity before introducing the high-resolution video modality.
- Stage 1, joint image-text-video: 256px images + 5-second video (192px, 16fps).
- Stage 2: both images and video are raised simultaneously to 480px, with video duration remaining at 5 seconds.
- Stage 3: both images and video are raised simultaneously to 720px, video remaining at 5 seconds.
- Post-training: architecture and optimizer unchanged, initialized from the pretraining checkpoint, jointly trained on post-training datasets at 480px and 720px.
**Data-side companion to the curriculum**: the resolution filtering threshold itself changes by stage ("resolution thresholds are applied at different training stages") — i.e., the data-admission standard is coupled to the training stage rather than being screened once up front.
**Training configuration**: a flow-matching / Rectified Flow framework (xt = t·x1 + (1−t)·x0, predicting the velocity field vt = x1 − x0, with an MSE loss), with timestep drawn from a logit-normal distribution; bf16 mixed precision, AdamW, weight decay 1e−3, initial lr 1e−4, with the learning rate dynamically lowered when FID and CLIP Score plateau — using evaluation-metric plateaus to trigger lr decay is a notable engineering detail.
**Inferred dimension evolution**: duration in the curriculum is fixed at 5 seconds throughout, whereas 2.6/2.7 already support 2–15 seconds and multi-shot narrative, implying a "duration-extension stage" and a "multi-shot stage" must have been added after 2.5, and that audio modality must have been introduced at some stage — but the stage divisions and switching criteria are entirely unknown. [Uncertain]

### Data Mixture Changes Across Training Stages (Pretraining/Annealing/SFT High-Quality Subsets) ⚠️

Wan 2.1 clearly has a "three-axis mixture that rolls with training stage" — the most valuable element of its data methodology for borrowing — but this is only presented qualitatively, with an illustrative figure and no numeric values.
**Core statement**: the report's Fig. 3 is titled "Data provisioning across different training phases," with the caption stating: for every training stage, the proportions of data related to motion, quality, and category are dynamically adjusted based on data throughput. That is, mixture scheduling proceeds along three orthogonal axes, with throughput constraints driving the schedule (high-resolution stages have low throughput, so per-sample quality density must be raised).
**Coupling with the six-tier motion grading**: static video and camera-dominated motion are "given reduced sampling ratio/priority" rather than deleted — the sampling rate itself is a mixture-control lever, and it can change by stage.
**The pretraining → post-training switch**: post-training switches entirely to a curated high-quality subset (images: expert-score top 20% + manual curation; video: quality-classifier top selection, taking millions each of simple- and complex-motion clips, covering 12 major categories), while continuing joint training at 480px/720px. This is a clear instance of the "annealing/fine-tuning stage uses small, high-quality data" practice.
**Gaps**: the specific numeric values of the three-axis mixture at each stage, the sample ratio between images and video, and the proportion of audio data mixed into 2.5+'s joint training are all unavailable. Nor is it stated whether pure-video (no-soundtrack) data is still mixed in during audio-video joint training to protect visual quality. [Uncertain]

### Post-Training Data (SFT Curated Set Size and Selection Criteria, Preference Pair Count and Annotation Method, Reward Model Training Data) ⚠️

Wan 2.1's post-training (Section 3.2) discloses fairly fine-grained detail but covers only SFT-style high-quality data curation, with no preference-learning step.
**Overall principle**: the core goal of post-training is to simultaneously improve visual fidelity and motion dynamics through high-quality data; static and dynamic data are handled with different strategies — image data optimizes visual quality, video data specifically hones motion quality.
**Image side (two construction paths)**
1) Expert-model-driven: from the high-resolution image pool, take the top 20% by expert-model predicted score, and within this subset additionally consider style and category factors to ensure distributional diversity;
2) Manual collection: humans collect top-tier images from various categories and sources, and actively fill in concepts missing from the dataset to enhance model generalization.
Together these yield "millions" of curated images, with the selection standard explicitly being excellence in quality, composition, and detail.
**Video side**: a visual-quality classifier first screens out top-ranked videos, then a motion-quality classifier separately selects "millions of simple-motion videos" and "millions of complex-motion videos"; all selection follows a strategy emphasizing category balance and high diversity, drawing from the 12 major categories (technology, animals, arts, humans, vehicles, etc.) to strengthen generation ability on commonly used categories.
**Task-specific fine-tuning data (Section 5.1.2, also within post-training)**
- I2V data: it was found that an overly large difference between the first frame and the rest of the video content leads to unstable image-driven generation, so SigLIP features are used to compute the cosine similarity between the "first-frame feature" and the "mean of the remaining frames' features," retaining only samples above a preset threshold (value undisclosed);
- Video-continuation data: the same method computes SigLIP feature cosine similarity between the preceding 1.5-second segment and the following 3.5-second segment, screening for temporally consistent video;
- First-and-last-frame transition data: the opposite approach is used — raising the proportion of samples with pronounced first/last-frame differences, to serve community demand for smooth transitions.
**Gaps**: there is no RLHF/DPO chapter, no preference-pair count or annotation method, and no description of reward-model training data (Wan-Bench uses human-feedback weighting, but that is an evaluation weight, not a training signal). Post-training data for 2.5/2.6/2.7 is entirely undisclosed; given capabilities such as "role-play" and "shot control," a large-scale task-specific SFT set must exist, but nothing can be determined. [Uncertain]

### Data Processing Infrastructure and Throughput (NeMo Curator/Data-Juicer/In-House, GPU Acceleration Ratio, Processing Scale, Cost) ⚠️

Data-processing infrastructure and throughput are entirely undisclosed: the Wan 2.1 report does not mention NeMo Curator, Data-Juicer, or any in-house data-processing framework (notably, Data-Juicer is itself an Alibaba product, but the Wan report does not cite it), and gives no GPU acceleration ratio, no processing throughput, no processing scale or cost, and no description of a distributed implementation.
The report's substantial infrastructure sections (4.3 Model scaling and training efficiency, 4.4 Inference) are entirely on the training and inference side: parallelism strategies (DP/FSDP/context parallelism, etc.), memory optimization, cluster reliability, inference parallelism strategy, Diffusion Cache, and quantization. The only statement related to "throughput" that indirectly affects data strategy is the diagnosis within the training curriculum — long sequences reduce training throughput, leading to insufficient "data throughput" — plus Fig. 3's "mixture dynamically adjusted based on data throughput," which shows the team treats data throughput as a first-class constraint on mixture scheduling, but with no numeric values whatsoever.
None for 2.5/2.6/2.7. [Uncertain]

## Performance Comparison

### Quantitative Impact of Data-Strategy Ablations (Distinguishing: Filtering-Strictness Ablation / Caption-Density-Style Ablation / Data-Mixture Ablation, and Corresponding Evaluation Metrics) ⚠️

The Wan series has performed no data-strategy ablations of any kind — no filtering-strictness ablation, no caption-density/style ablation, and no data-mixture ablation. This is the most obvious shortfall in the data chapter of its 60-page report.
**Ablations that actually exist in the report (Section 4.7.2, all architecture/component-level, run on the 1.3B version for fast evaluation)**
1) AdaLN sharing scheme: comparing Full-shared-adaLN-1.3B, Half-shared-adaLN-1.5B, Full-shared-adaLN-1.5B (extended to 35 layers), and Non-shared-AdaLN-1.7B, all trained from scratch for text-to-image over 200K steps, global batch size 1536, measured by L2 training loss in latent space. Conclusion: at the same parameter count, Full-shared-1.5B (using extra parameters to deepen the network) achieves the lowest loss, while the larger Non-shared-1.7B actually underperforms — hence the choice of fully-shared AdaLN.
2) Text encoder: umT5(5.3B) vs. Qwen2.5-7B-Instruct vs. GLM-4-9B (the latter two taking second-to-last-layer features, with a bidirectional token refiner added per the HunyuanVideo approach), with umT5 best on both training loss and visualization (attributed to umT5's bidirectional attention being better suited to diffusion models, whereas decoder-only LLMs use causal attention). Also compared against Qwen-VL-7B on FID: umT5 43.01, Qwen-VL-7B last-layer 43.72, second-to-last-layer 42.91 — comparable performance but a larger model, hence umT5 was chosen.
3) Autoencoder: VAE vs. VAE-D (using a diffusion loss in place of the reconstruction loss), FID at 10K steps 42.60 vs. 44.21, at 15K steps 40.55 vs. 41.16 — VAE performs better.
**The only data-related quantitative claim**: the synthetic-image-contamination experiment — the report states empirical evidence shows that "even <10% generated-image contamination significantly degrades model performance," hence the dedicated classifier trained to remove it. This is the only conclusion in the Wan report that directly links data purity to model performance with a quantitative claim, though no comparison curve is given.
**Caption-side quantification**: slow-fast encoding raises the caption model's VideoMME score from 67.6% to 69.1%; the caption model's ten-dimensional F1 is compared item-by-item against Gemini 1.5 Pro. These measure the quality of the captioner itself, not "how caption style affects the generation model."
None for 2.5/2.6/2.7. [Uncertain]

### Quality-vs-Quantity Evidence (Cases Where Small, High-Quality Data Outperforms Large, Noisy Data) ⚠️

Several methodological manifestations exist along with one quasi-quantitative piece of evidence, but a direct "small-and-precise vs. large-and-noisy" controlled experiment is missing.
**Quasi-quantitative evidence (the strongest)**: synthetic-image contamination as low as <10% significantly degrades model performance — this directly demonstrates that the marginal impact of data purity far outweighs data volume, a textbook illustration of "a small amount of dirty data can destroy the gains from a large amount of good data," and the single most citable line in this entry.
**Methodological manifestations**
1) The fundamental-dimension stage eliminates roughly 50% of candidate data in one step — willing to cut the pool in half to secure baseline quality first;
2) Post-training uses only the expert-score top 20% of images and the top-ranked videos selected by the quality classifier, dropping the scale from billions to millions;
3) V2A is strictly filtered from a billions-scale video parent pool down to O(1) thousand hours — accepting extreme scale compression for the sake of audio-visual usability;
4) Personalization data is screened from O(100)M down to O(10)M;
5) The final stage of caption-model training is specifically end-to-end training on "a small amount of high-quality data," with manually annotated captions defined as the highest-quality data, used only in the final stage.
**Counterbalance**: it should be noted that Wan is simultaneously very careful "not to lose distribution in the pursuit of quality" — 100-cluster quota sampling to prevent long-tail loss, downsampling rather than deleting static/camera-motion categories, and post-training's emphasis on category balance plus manual filling of missing concepts are all hedges against "over-pursuing precision leading to distribution collapse." This dual stance of "high standards + preserved distribution" is more worth borrowing than a purely quality-first approach.
No "full dataset vs. curated subset" controlled experiment or quantified gain exists. [Uncertain]

### Alignment Between Training-Data Domain Distribution and Benchmark Taxonomy (e.g. VABench's Seven Major Categories) ⚠️

There is a clear but non-declarative correspondence between Wan's self-built Wan-Bench and its training-data taxonomy, with an evident gap on the audio-visual dimension.
**Wan-Bench structure**: three major dimensions, 14 fine-grained metrics, each dimension using a dedicated algorithm (traditional detectors for simple tasks, MLLMs for complex ones):
1) Dynamic quality: large-motion generation (RAFT optical-flow-magnitude normalized scoring), human-artifact detection (a YOLOv3-type model trained on 20,000 manually annotated AI-generated images, combining predicted likelihood, bounding box, and duration into a score), physical plausibility and smoothness (Qwen2-VL video Q&A, detecting object clipping, unrealistic collisions, anti-gravity motion, and other physics violations), pixel-level stability (using optical flow to identify static regions, then computing inter-frame differences), and ID consistency (three sub-dimensions — person/animal/object — via DINO frame-level feature similarity);
2) Image quality: overall image quality (the average of MANIQA fidelity, the LAION aesthetic predictor, and MUSIQ), scene-generation quality (inter-frame CLIP similarity for consistency + frame-text CLIP similarity for alignment), and stylization (frame-level Qwen2-VL Q&A);
3) Instruction following: single-object/multi-object/spatial-position (Qwen2-VL predicts category, count, and spatial relations, scored by per-frame hit rate), camera control (five types — pan, crane, dolly, aerial, tracking — with 130 custom prompts; the first three types analyzed via RAFT optical flow, the latter two via Qwen2-VL Q&A), and action-instruction adherence (three motion categories — person/animal/object — feeding Qwen2-VL key frames and querying action alignment, action completeness, and artifacts).
The final composite score is produced via a human-feedback-guided weighting strategy.
**Correspondence with training data**: dynamic quality ↔ the six-tier motion grading and optical-flow motion scoring; image quality ↔ the aesthetic scorer and the LAION classifier (the evaluation directly reuses the LAION aesthetic predictor, sharing its source with the training side); ID consistency ↔ ArcFace/SigLIP consistency screening; camera control ↔ the manually annotated camera-angle-and-motion dataset and its expert-model label expansion; object/counting/spatial-relations ↔ the caption model's self-built datasets for counting, fine-grained categories, and spatial relations. Wan-Bench's taxonomy is essentially a mirror of its data-construction dimensions — though the report never states this correspondence as a design principle.
**Key gap**: none of Wan-Bench's 14 metrics across its three major dimensions covers audio-visual sync, and the audio dimension is entirely absent overall; the three headline capabilities of 2.5 onward — "audio-visual sync," "multi-shot narrative," and "role-play" — have no corresponding category in Wan-Bench at all. Audio-visual-sync evaluation appears only in the Wan2.2-S2V paper, via the EMTD dataset plus the single Sync-C metric. External categorized audio-visual benchmarks such as VABench's seven major categories are not addressed. [Uncertain]

## Other Information

### summary_note

**Positioning**: the Wan series is a textbook two-phase case study of "an extremely detailed open-source predecessor report, followed by an extremely sparse closed-source commercial present." To study its data methodology, there is only one backbone document: Chapter 3 of the Wan 2.1 technical report (arXiv:2503.20314), whose level of detail ranks near the top among open-source video-generation models (enumerating 9 fundamental filters, cluster-quota sampling, six-tier motion grading, three-stage caption-model training, and 10-dimensional caption evaluation); while the actual research target — Wan 2.5/2.6/2.7 — has no technical report, no weights, and no paper at all, so its data methodology must be reverse-inferred from three corroborating sources: the incremental figures in the Wan 2.2 README, the audio-visual pipeline in the Wan2.2-S2V paper, and the capability matrix in the Bailian API documentation.
**The five most valuable points**
1) "Cluster-quota sampling" against the distribution drift caused by quality filtering: the full dataset is split into 100 clusters, with a quota drawn from each to proceed to the next stage, with the explicit goal of preventing "small but important" data from the long tail from being entirely wiped out by a global threshold. This shares its origin with, but differs mechanically from, LTX-2's "multi-label-network-constrained pair sampling," and is simpler to implement and reproduce.
2) Six-tier motion-quality grading with differentiated sampling rates: optimal/medium/static/camera-dominated/low-quality/shaky, where static and camera-dominated motion are "downsampled" rather than "deleted" — only low-quality motion and shaky footage are truly excluded. Treating motion as an independent funnel stage and regulating it via sampling rate rather than a binary threshold is Wan's most mature design.
3) Staged three-axis mixture scheduling (Fig. 3): along the three orthogonal axes of motion / quality / category, the mixture is dynamically adjusted at every training stage based on data throughput. The mixture is a rolling scheduled quantity rather than a static constant — a step further than most reports that only say "low-res first, then high-res."
4) V2A three-part joint audio-video caption + random masking: caption = dense video description + ambient-sound characterization + background-music analysis (style/rhythm/melody/instrumentation), supporting a negative "no music" statement; during training, the ambient-sound and music caption segments are each randomly dropped with a preset probability, forcing the model to build a purely visual-to-audio association while retaining text controllability when present. This "split-stream schema + random caption-component dropout" combination is a directly transferable design.
5) Wan2.2-S2V's Light-ASD audio-visual-sync filtering: active-speaker detection applies two exclusion rules — audio out of sync with the active speaker, or no active speaker present on screen — combined with a prerequisite that the face remain visible throughout the sequence. This is the only clearly documented audio-visual-sync data-filtering method in the Wan series, and the single most reusable finding under this research topic.
**Leverage-style annotation paradigm**: Wan repeatedly reuses the same pattern — manually label a small seed set (1–5 quality-score pairs, camera angle, 20,000 artifact annotations) → train an expert model → automatically score/expand labels at full scale. This is more economical than per-sample manual annotation, and the report applies it in all three of quality scoring, camera motion, and artifact detection.
**The biggest information gaps**
(a) Speech and lip-sync data: Wan 2.1 explicitly excludes speech, while lip sync becomes a headline feature from Wan 2.5 onward — this 180-degree pivot is entirely undisclosed on the ASR transcription, speaker diarization, language/accent annotation, and voiceprint-modeling fronts behind it — a stark contrast with LTX-2, which writes speaker/language/accent into its caption schema.
(b) Multi-shot data: the shot-control and timestamped shot-prompt interfaces of 2.6/2.7 suggest the training captions already contain shot-level timeline annotation and cross-shot-consistency annotation, but the shot-segmentation method (no shot-detection tool has ever been mentioned across the series) and multi-shot sample construction are entirely invisible.
(c) Deduplication method: mentioned in a single phrase, with no distinction between exact and semantic deduplication, no threshold, and no ratio.
(d) Data ablations: zero. All ablations are on the architecture side (AdaLN, text encoder, VAE).
(e) Quantitative funnel: only two figures exist — "fundamental dimensions eliminate ~50%" and "images take the top 20%."
**Comparison with peers**: compared with LTX-2 (more open overall, but with a data chapter of only two paragraphs, ~150 words), Wan 2.1's video-side cleaning is markedly more reproducible, especially its filter enumeration and motion grading; but on the joint audio-visual dimension, LTX-2's "audio information-content filtering + full-soundscape dual-track caption + the three attributes speaker/language/accent" is more complete than Wan's three-part V2A caption, and Wan's data methodology is completely absent for the speech era. Compared with closed-source models like Seedance/Sora 2/Veo 3, Wan has at least left behind a readable data-methodology blueprint via its open-source predecessor — its unique value — but one must stay clear-eyed: that blueprint describes an early-2025 pure-video model, separated from the 2026 Wan 2.7 by three major capability leaps — speech, multi-shot, and native audio conditioning — so directly applying it carries considerable risk.
**Compliance profile**: ownership is glossed over with the vague phrasing "internal copyrighted sources + public data," with no licensed-data percentage, no rights-cleared list, and no C2PA; compliance focus is on the output-side optional "AI-generated" watermark and China's content-labeling regulations, rather than a training-side provenance system.

## Uncertain Fields

The research information for the following fields is partly uncertain (sources marked ⚠️):

- release_date
- av_generation
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
- joint_av_caption_schema
- dialogue_transcription_attributes
- geometric_structured_annotation
- synthetic_data_synthesis
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
