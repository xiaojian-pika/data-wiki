# Video Caption Model Ecosystem — A Consolidated Research Entry Covering ShareGPT4Video / ShareCaptioner-Video, the Tarsier & Tarsier2 Series, CogVLM2-Caption, SkyCaptioner-V1, AVoCaDO, AVSCap, video-SALMONN 2, Qwen3-Omni-Captioner / Qwen3.5-Omni, AuroraCap, the Panda-70M Multi-Teacher Captioner, LLaVA-Video / PLLaVA, Aria, Tag2Text, etc.

Covering ShareGPT4Video / ShareCaptioner-Video, the Tarsier & Tarsier2 series, CogVLM2-Caption, SkyCaptioner-V1, AVoCaDO, AVSCap, video-SALMONN 2, Qwen3-Omni-Captioner / Qwen3.5-Omni, AuroraCap, the Panda-70M multi-teacher captioner, LLaVA-Video / PLLaVA, Aria, Tag2Text, and more, and summarizing how they are actually used in video-generation data pipelines. This entry is not a single model/dataset but a cross-sectional ecosystem map of the "annotation/captioning tool" pipeline component.

> Topic: Data processing for video-generation models (including simultaneous audio-video generation): data filtering pipelines, data distribution, and captioning/annotation approaches

[← Back to Home](../index.md)

**Table of Contents**: [Basic Information](#basic-information) · [Data Scale and Distribution](#data-scale-and-distribution) · [Filtering Pipeline](#filtering-pipeline) · [Captioning Approach](#captioning-approach) · [Audio-Video Alignment](#audio-video-alignment) · [Training Integration](#training-integration) · [Results Comparison](#results-comparison)

## Basic Information

### Name

Video Caption Model Ecosystem — a consolidated research entry covering ShareGPT4Video / ShareCaptioner-Video, the Tarsier & Tarsier2 series, CogVLM2-Caption, SkyCaptioner-V1, AVoCaDO, AVSCap, video-SALMONN 2, Qwen3-Omni-Captioner / Qwen3.5-Omni, AuroraCap, the Panda-70M multi-teacher captioner, LLaVA-Video / PLLaVA, Aria, Tag2Text, and more, summarizing how they are actually used within video-generation data pipelines. This entry is not a single model/dataset, but a cross-sectional ecosystem map of the "annotation/captioning tool" pipeline component.

### Publishing Institution/Company

Multiple organizations, grouped by camp:
[Academic/open-source community] ShareGPT4Video & ShareCaptioner-Video (USTC + Shanghai AI Lab + CUHK, 15 authors including Lin Chen, Xilin Wei, Jinsong Li, NeurIPS 2024 Datasets & Benchmarks Track); AuroraCap (UC Santa Barbara / multi-institution collaboration, ICLR 2025); Panda-70M (Snap Research + UC Merced, CVPR 2024).
[Chinese big-tech] ByteDance Tarsier / Tarsier2 / Tarsier2-Recap (bytedance/tarsier), and collaboration with Tsinghua's Department of Electronic Engineering on video-SALMONN 2 / video-SALMONN-o1; Zhipu AI (zai-org / formerly THUDM) CogVLM2-Caption; Alibaba Tongyi's Qwen series (Qwen2-VL / Qwen2.5-VL / Qwen3-VL / Qwen2.5-Omni / Qwen3-Omni-Captioner / Qwen3.5-Omni, paired with Omni-Captioner + Omni-Detective + Omni-Cloze, ICLR 2026); Kunlun Wanwei's SkyWork SkyCaptioner-V1; Kuaishou's Kling team AVoCaDO (jointly with CASIA, UCAS, PKU, and NJU) and the Koala-36M LLaVA-fine-tuned captioner; Nanjing University's NJU-LINK + Kuaishou Kling's AVSCap; Xiaomi's MiMo-VL (used by MOVA as a visual annotator).
[Overseas big-tech/closed-source] OpenAI (Sora's highly descriptive captioner, Whisper ASR); Meta (Movie Gen's LLaMa3-Video 8B/70B); Google (the Gemini series as a general-purpose captioner; Veo 3 uses "multiple Gemini models"); Lightricks (LTX-Video / LTX-2's in-house audio-video captioner); Rhymes AI (Aria, used by Allegro as a fine-grained captioner).

### Release Date (Technical Report/Paper/Open-Source Date)

Ecosystem evolution timeline (based on public release/arXiv submission dates):
· 2023: Tag2Text (a tag-based lightweight captioner, used by Allegro for coarse filtering).
· 2024-02: Panda-70M (arXiv 2402.19479, CVPR 2024), the first systematic "multi-teacher + retrieval-based selection" captioning paradigm.
· 2024-06-06: ShareGPT4Video / ShareCaptioner-Video (arXiv 2406.04325), proposing DiffSW differential sliding-window captioning.
· 2024-08: CogVideoX (arXiv 2408.06072) discloses the four-stage Panda-70M→CogVLM→GPT-4→LLaMA2 captioning chain; 2024-09-19 open-sources CogVLM2-Caption weights.
· 2024-10: AuroraCap + the VDC benchmark (arXiv 2410.03051); Koala-36M (arXiv 2410.08260).
· 2024-10: The Movie Gen technical report discloses the LLaMa3-Video 8B/70B captioning approach.
· 2025-01-14: Tarsier2-7B (arXiv 2501.07888); the accompanying Tarsier2-Recap-585K dataset.
· 2025-02: video-SALMONN-o1 (arXiv 2502.11775).
· 2025-04: SkyCaptioner-V1 (open-sourced alongside SkyReels-V2).
· 2025-06-18: video-SALMONN 2 (arXiv 2506.15220, 3B/7B/72B).
· 2025-09: Qwen3-Omni-Captioner (audio captioner) open-sourced; 2025-10 Omni-Detective / Omni-Captioner open-sourced.
· 2025-10-12: AVoCaDO (arXiv 2510.10395), the first fully systematic joint audio-video captioner.
· 2026-01: LTX-2 (arXiv 2601.03233) discloses its in-house joint audio-video captioner.
· 2026-03: Qwen3.5-Omni (arXiv 2604.15804) lists "script-level structured audio-video captioning" as a first-class capability; the Omni-Cloze benchmark is released.
· 2026-06: OmniCap-IF (arXiv 2606.08572), the first audio-video caption instruction-following benchmark.
· 2026-07-14: AVSCap + AVSCapBench (arXiv 2607.12820), the newest joint audio-video captioning model and dedicated benchmark.

### Type (Model/Dataset/Toolchain/Benchmark)

A composite ecosystem of models + toolchains + evaluation benchmarks. Three-layer structure:
(1) The captioning models themselves — general-purpose VLMs (Qwen-VL / InternVL / LLaVA family), specialized captioners (ShareCaptioner-Video, Tarsier2, CogVLM2-Caption, SkyCaptioner-V1, AuroraCap, AVoCaDO, AVSCap), and omni-modal captioners (Qwen3-Omni-Captioner, video-SALMONN 2, Qwen3.5-Omni);
(2) Datasets produced by captioners (ShareGPT4Video-40K / 4.8M, Tarsier2-Recap-585K, Panda-70M, Koala-36M, AVoCaDO-SFT-107K, AVSCap-130K) — captioning models and the datasets they produce are mutually causal, a core feature of this ecosystem;
(3) Evaluation benchmarks (DREAM-1K, VDC/VDCScore, VidCapBench, AVSCapBench, UGC-VideoCap, Omni-Cloze, OmniCap-IF, the video-SALMONN 2 test set).

### Openness (Whether Weights/Code/Data/Pipeline Are Each Open-Sourced) ⚠️

Openness shows a clear three-tier split: "fully open in academia / half-open at big tech / closed on the generation side":
[Fully open (weights + code + data)] ShareGPT4Video: 40K GPT-4V dense captions and 4.8M ShareCaptioner-Video annotations are all public; the paper is CC BY 4.0; ShareCaptioner-Video weights are on HF (Lin-Chen/ShareCaptioner-Video, base model InternLM-XComposer2-4KHD). The Tarsier series: bytedance/tarsier code + omni-research/Tarsier2-Recap-7b weights + the Tarsier2-Recap-585K dataset are all fully open, making it currently the most widely reused open-source captioner among downstream generation models. CogVLM2-Caption: weights open-sourced (zai-org/cogvlm2-llama3-caption), the only reproducible link in the CogVideoX data pipeline. SkyCaptioner-V1: weights + training details (Qwen2.5-VL-7B base model, 32×A800, 2 million concept-balanced samples) are fully disclosed. AVoCaDO: Apache-2.0 weights + code + project page, but AVoCaDO-SFT-107K was not separately released. video-SALMONN 2: Apache-2.0, code/weights/test set are all open. AuroraCap + the VDC benchmark are open-sourced.
[Half-open] AVSCap: code + AVSCapBench are open, and the AVSCap-130K training set README explicitly states it "will release as soon as possible" but had not been released as of research time; the availability of AVSCap-7B weights is questionable (the GitHub repo shows both an HF link and a "to be released" note) [uncertain]. Qwen3-Omni-Captioner (the audio version) is open-sourced, but Qwen3.5-Omni's audio-video captioning capability is only available via API and has not been open-sourced as a standalone captioning tool.
[Closed-source] Nearly all in-house captioners built by generation-side teams are closed: OpenAI Sora's highly descriptive captioner, Meta's fine-tuned LLaMa3-Video captioning version, Google's Gemini captioning variant used for Veo 3, Lightricks LTX-2's audio-video captioner, Tencent Hunyuan's three in-house caption models, StepFun's in-house VLM for Step-Video, ByteDance Seedance 1.5/2.0's captioning system, and Kuaishou Kling 3.0 Omni's video-description enhancement module — all disclose only "what was used," never parameter count, base model, or weights.
[A counterintuitive structural fact] Captioners have markedly higher openness than the data-pipeline openness of the generation models themselves: most closed-source generation models' technical reports are willing to name-drop Tarsier2 / Qwen3-Omni / LLaVA-Video, because the captioner is treated as a "tool" rather than a "moat." What is actually treated as a moat is the original captioning prompt text, field schemas, and threshold tables — and this part is nearly universally undisclosed across the industry.

### Whether Simultaneous Audio-Video Generation Is Supported, and Implementation Approach (Native Joint/Cascaded/MoE Fusion)

Captioners themselves do not perform generation; this field is reinterpreted here as "whether it supports joint audio-video input and outputs a joint description," which is the single most important capability watershed for this ecosystem in 2025–2026:
[Vision-only captioners (deaf to audio)] ShareCaptioner-Video, Tarsier / Tarsier2, CogVLM2-Caption, SkyCaptioner-V1, AuroraCap, LLaVA-Video, PLLaVA, Aria, Qwen2.5-VL / Qwen3-VL, and all 31 candidate captioners from the Panda-70M paper (which explicitly states that the Video-LLaMA branch "uses only the visual branch, with the audio branch explicitly disabled"). This tier dominated the first half of 2024–2025.
[Cascaded audio-video annotation (multiple models divide labor, then an LLM fuses the outputs)] The mainstream engineering approach, exemplified by: MOVA = MiMo-VL-7B-RL (vision) + Qwen3-Omni-Instruct (ASR) + Qwen3-Omni-Captioner (non-speech) + GPT-OSS-120B (fusion and consistency checking); Movie Gen's audio side = four models working together — audio-quality prediction, AED, general audio captioning, and music captioning; UniTalking = Qwen3-VL + Whisper-V3 + Qwen3-Omni-Captioner + Qwen3-Omni; Kling-Foley = a three-stage pipeline of audio classification + an audio-understanding LLM + LLM fusion.
[Native joint audio-video captioners (single model that both sees and hears)] A new paradigm that took shape starting 2025 Q4: AVoCaDO (Qwen2.5-Omni-7B base, ~9B end to end), AVSCap-7B (same base model), video-SALMONN 2 (LLaVA-OneVision + audio LoRA, 3B/7B/72B), Qwen3-Omni / Qwen3.5-Omni, and Lightricks's unnamed system developed in-house for LTX-2. UniVerse-1 is more extreme, using a single Qwen2.5-Omni to output speech content / video caption / ambient-sound caption in parallel in one pass.
[Key empirical finding] Bare Qwen2.5-Omni's zero-shot captioning ability is very poor (AVSCapBench overall only 21.53, Speech 13.92); it must undergo caption-specific SFT+RL before it can be used as a captioner (AVoCaDO 49.31, AVSCap 60.44) — "having an omni base model ≠ being able to serve as an omni captioner" is the single most important engineering lesson in this ecosystem.

### Research Source List (URLs to papers/technical reports/official documentation/news, with each source's nature labeled: primary/official, corroborating from the same team, or third-party reporting)

- https://arxiv.org/abs/2406.04325 — ShareGPT4Video paper (primary/official, NeurIPS 2024 D&B Track, CC BY 4.0; 40K GPT4V dense captions + 4.8M ShareCaptioner-Video annotations + the DiffSW method)
- https://huggingface.co/Lin-Chen/ShareCaptioner-Video — ShareCaptioner-Video model card (primary/official, confirms the base model is InternLM-XComposer2-4KHD, supports streaming differential sliding windows and clip summarization)
- https://arxiv.org/abs/2501.07888 — Tarsier2 paper (primary/official; 7B, pretraining 11M→40M video-text pairs, fine-grained temporal-alignment SFT, model-based sampling + DPO; DREAM-1K F1 exceeds GPT-4o by 2.8% and Gemini-1.5-Pro by 5.8%; human evaluation +8.6% / +24.9%; SOTA on 15 public benchmarks)
- https://github.com/bytedance/tarsier — Tarsier official repository (primary/official; Tarsier2-Recap-7b weights and the Tarsier2-Recap-585K dataset)
- https://huggingface.co/omni-research/Tarsier2-Recap-7b — Tarsier2-Recap model card (primary/official; 585K clips drawn from public datasets such as VATEX/TGIF/LSMDC, annotated by Tarsier2-7B; the upstream source is a 1M-video recaptioning effort)
- https://github.com/zai-org/CogVideo/blob/main/tools/caption/README.md — CogVideoX captioning tool documentation (primary/official; the purpose of CogVLM2-Caption and the video→caption→video closed loop)
- https://arxiv.org/pdf/2408.06072 — CogVideoX paper (primary/official; the four-stage captioning chain of Panda-70M short captions → CogVLM per-2-seconds dense frame captioning → GPT-4 summarization → fine-tuning LLaMA2 on 50K summarization samples → distillation into CogVLM2-Caption, with the full prompt given in Appendix G)
- https://arxiv.org/abs/2410.03051 — AuroraCap + the VDC benchmark (primary/official; token merging to reduce visual token count, three-stage training on 20 million image/video-text pairs, over a thousand structured VDC captions with five fields — camera/short/background/main object/detailed — and VDCScore, a divide-and-conquer LLM-based evaluation metric; Flickr30k CIDEr 88.9 > GPT-4V 55.3 > Gemini-1.5-Pro 82.2)
- https://arxiv.org/abs/2410.08260 — Koala-36M (primary/official; 36M clips, average length 13.75s, 720p, average caption length 202 words, GPT-4V-generated seed captions used to fine-tune LLaVA, six categories of structured information, the VTSS training-suitability score)
- https://openaccess.thecvf.com/content/CVPR2024/papers/Chen_Panda-70M_Captioning_70M_Videos_with_Multiple_Cross-Modality_Teachers_CVPR_2024_paper.pdf — Panda-70M (primary/official; 31 candidate captioners = 6 base models × weight/modality variants; greedy set-cover selects 8 covering 76.8%; UMT-large fine-grained video-text retrieval for best-caption selection; human agreement rate only 44.9%)
- https://arxiv.org/abs/2510.10395 — AVoCaDO (primary/official; Qwen2.5-Omni-7B base model, six-source composition of AVoCaDO-SFT-107K, a five-dimensional keypoint checklist, GRPO with three reward terms including a dialogue edit-distance DP-alignment threshold of 0.6)
- https://github.com/AVoCaDO-Captioner/AVoCaDO — AVoCaDO official repository and Apache-2.0 weights (primary/official)
- https://arxiv.org/abs/2607.12820 — AVSCap + AVSCapBench (primary/official, 2026-07-14; three criteria — Acoustic/Visual Completeness + AV Synergy; AVSCap-130K = 40K videos × 3 annotations each; SFT→GRPO; a horizontal comparison table of fine-grained event recall on 1,226 AVSCapBench videos)
- https://github.com/NJU-LINK/AVSCap — AVSCap official repository (primary/official; the training set has not yet been released)
- https://arxiv.org/pdf/2506.15220 — video-SALMONN 2 (primary/official, Tsinghua Dept. of Electronic Engineering + ByteDance; LLaVA-OneVision + audio LoRA, 3B/7B/72B, MrDPO multi-round DPO, caption error rate reduced 28% relative to baseline)
- https://github.com/ddlBoJack/Omni-Captioner — Omni-Detective / Omni-Captioner / Omni-Cloze (primary/official, ICLR 2026; an agentic Query-Observation loop annotation framework; the audio version is Qwen3-Omni-Captioner, the audio-video version is folded into Qwen3.5-Omni)
- https://arxiv.org/html/2604.15804v2 — Qwen3.5-Omni technical report (primary/official; explicitly positions "script-level fine-grained description, automatic segmentation, timestamp annotation, and description of character-audio relationships" as a data-generation capability for training video synthesis models; Omni-Cloze Plus 64.8 > Gemini-3.1 Pro 57.2)
- https://arxiv.org/html/2601.03233v1 — LTX-2 (primary/official; Section 5.1 describes its in-house audio-video captioner, whose fields include camera motion / lighting / subject behavior + music / ambient sound / dialogue transcription with speaker-language-accent; the guiding principle is comprehensive yet factual, explicitly banning emotional interpretation)
- https://arxiv.org/html/2502.12782v1 — VidCapBench (third-party benchmark; a video-caption evaluation oriented toward controllable T2V)
- https://arxiv.org/abs/2606.08572 — OmniCap-IF (third-party/academic; the first audio-video caption instruction-following benchmark, with 50 constraint types + a dual format/content scoring scheme)
- https://arxiv.org/html/2507.11336 — UGC-VideoCaptioner + UGC-VideoCap (third-party/academic; 1,000 TikTok videos + 4,000 QA pairs; three-stage human-in-the-loop annotation covering audio-only / visual-only / joint AV separately)
- https://arxiv.org/abs/2502.11775 — video-SALMONN-o1 (primary/official; process-level pDPO preference optimization + RivaBench, positioned toward QA/reasoning rather than captioning)
- https://huggingface.co/AVoCaDO-Captioner/AVoCaDO — AVoCaDO weights (primary/official)
- https://arxiv.org/html/2412.09283v1 — InstanceCap (third-party/academic; instance-aware structured captioning to improve T2V)
- https://www.marktechpost.com/2025/01/15/bytedance-researchers-introduce-tarsier2-a-large-vision-language-model-lvlm-with-7b-parameters-designed-to-address-the-core-challenges-of-video-understanding/ — Tarsier2 coverage (third-party reporting)
- 30 generation-model entries from the same research batch in this repository (Movie_Gen.json / HunyuanVideo.json / Seedance_20_Seedance_15_pro.json / Open-Sora.json / SkyReels.json / MOVA.json / LTX-2.json / CogVideoX.json / Allegro.json / Goku.json / LongCat-Video.json / Step-Video-T2V.json / Sora_2.json / Veo_3_Veo_31.json / pretraining_datasets.json, etc.) — the `caption_model` and `data_ablation` fields, being first-hand technical-report accounts of generation-side captioner selection (corroborating, same-project)

## Data Scale and Distribution

### Training Data Scale (Video Count/Hours/Token Count, Pretraining and SFT Listed Separately) ⚠️

Here we distinguish two strictly separate quantities: "the training-data scale of the captioner itself" and "the scale of annotations produced by the captioner":
[Captioner training data (pretraining/SFT listed separately)]
· Tarsier2-7B: pretraining on 40M video-text pairs (a 3.6× expansion from Tarsier1's 11M); the SFT stage performs fine-grained temporal alignment (the paper cites roughly 150K human-curated annotations, including event timestamps) [the scale figure is partly uncertain]; followed by DPO on preference pairs automatically constructed via model-based sampling.
· ShareCaptioner-Video: SFT data = 40K GPT-4V-annotated dense captions (in DiffSW format), base model InternLM-XComposer2-4KHD. It trades an extremely small amount of SFT data for large-scale inference capability, a textbook case of "distilling from a small amount of high-quality teacher data."
· AuroraCap: three-stage training, cumulatively over 20 million high-quality image/video-text pairs.
· SkyCaptioner-V1: about 2 million concept-balanced videos (curated from 10 million, a 20% retention rate), 32 A800 GPUs, global batch size 512.
· AVoCaDO: AVoCaDO-SFT 107K (TikTok-10M 24K + ShortVideo 18K + Shot2Story 20K + FineVideo 29K + YouTube-Commons 11K + CinePile 5K), followed by GRPO.
· AVSCap: AVSCap-130K = 40K videos × 3 annotations (visual / audio / synergistic omni-modal), followed by GRPO; the paper explicitly states "the gains from RL exceed those from scaling up SFT data."
· CogVLM2-Caption: dense caption data produced by the teacher chain (of which 50K GPT-4-summarized samples were used to fine-tune the LLaMA2 summarizer); the total volume on the student side is undisclosed [uncertain].
· video-SALMONN 2: exact sample counts undisclosed [uncertain]; model tiers of 3B/7B/72B.
[Annotation scale produced by captioners (usable volume downstream)]
· ShareCaptioner-Video → 4.8 million high-quality aesthetic video annotations, one of the largest single-model outputs in the open-source community.
· Tarsier2 → recaptioned 1 million public-dataset videos, releasing 585K of them (Tarsier2-Recap-585K).
· Panda-70M → captions for 70 million clips, but the captions are extremely short (average 13.2 words).
· Koala-36M → 36 million clips, average caption length 202.3 words (about 15× longer than Panda-70M).
· The implicit output scale on the generation side is even larger: Movie Gen's entire clip corpus is annotated by LLaMa3-Video (70% from the 8B model, 30% from the 70B model); Apollo/Klear multi-model-annotates 81 million samples (including Gemini-2.5-Pro calls); Harmony uses Gemini to annotate 4 million audio-video clips.
[Ecosystem-level observation] Caption length spans a 60× range — from Panda-70M's 13.2 words to the longest at 824.2 words (see the pretraining_datasets entry for details) — the choice of captioning model directly determines caption length, which in turn determines the prompt-length sensitivity range of downstream T2V models.

### Data Source Composition (In-House/Public Datasets/Web Scraping/Licensed Procurement/Synthetic Data)

The source composition of captioner training/evaluation data:
[Teacher distillation as the unwavering dominant paradigm (2024–2026)] GPT-4V → ShareGPT4Video-40K, Koala-36M seed captions, MiraData structured captions; GPT-4 → the summarization stage of CogVideoX; Gemini-2.5-Pro → AVoCaDO-SFT, the 500K MTSS annotations of Script-a-Video, Harmony's 4 million; the Gemini-3 series → the default teacher for new 2026 work. Student models are overwhelmingly 7B-class open-source base models (InternLM-XComposer2, Qwen2.5-VL-7B, Qwen2.5-Omni-7B, LLaVA-Video, LLaMA2/LLaMA3).
[Reuse of public datasets] Tarsier2-Recap-585K is drawn entirely from public datasets (VATEX, TGIF, LSMDC, etc.); AVoCaDO-SFT comes from public sources such as Shot2Story, FineVideo, YouTube-Commons, and CinePile, plus TikTok/short-video content; AVSCap-130K comes from AVoCaDO-107K, ASID-1M, FineVideo, TimeChatCap-40K, and Movie101 — a clear "dataset-stacked-on-dataset" snowball pattern of reuse.
[Web scraping] ShareGPT4Video's 4.8M aesthetic videos, Koala-36M, and Panda-70M (derived from HD-VILA-100M) are mainly public web videos such as YouTube.
[Human curation (small in quantity but critical)] SkyCaptioner-V1's camera-motion sub-expert uses 93K high-confidence human annotations + 16K motion-axis-balanced synthetic data; captions in Movie Gen's post-training stage are refined by humans on top of model output; ALIVE's caption-model training data is "MLLM-generated then human-revised."
[Synthetic/bootstrapped] video-SALMONN 2 explicitly uses its own high-quality caption corpus for subsequent SFT (self-bootstrapped data production); Tarsier2 uses model-based sampling to automatically construct preference pairs.

### Data Compliance and Provenance (Proportion of Licensed Data, Rights-Cleared Datasets, C2PA, etc.) ⚠️

This dimension is the weakest link in this ecosystem — the vast majority of captioner papers do not discuss it at all:
[Clear model licensing] AVoCaDO (Apache-2.0), video-SALMONN 2 (Apache-2.0), Tarsier2-Recap-7b, CogVLM2-Caption, SkyCaptioner-V1, and Aria (Apache-2.0) all give explicit weight licenses; the ShareGPT4Video paper is CC BY 4.0.
[Data-source compliance is almost entirely undisclosed] ShareGPT4Video's 4.8M web videos, Koala-36M, and Panda-70M do not state copyright status or licensing provenance; AVoCaDO-SFT includes TikTok-10M and ShortVideo subsets, and the re-distribution compliance of platform content is not discussed; no captioner work mentions C2PA, rights-cleared datasets, or the proportion of licensed procurement.
[A structural risk] The teacher-distillation paradigm generally violates the terms of service of commercial APIs (both OpenAI and Google prohibit using outputs to train competing models); ShareGPT4Video, Koala-36M, AVoCaDO, Script-a-Video, and many other works rely on distillation from GPT-4V / Gemini — unproblematic for academic release, but legally uncertain for commercial deployment. This point is never mentioned in any of the papers.
[The generation side is relatively more cautious] Technical reports from large generation-side players such as Movie Gen and Veo 3 devote more space to data compliance, but when it comes to the captioner itself they only say "we used an internal model," which actually sidesteps the issue.
[Uncertain] No publicly available quantitative data on licensing proportions exists.

### Clip Duration Distribution and Splitting Strategy ⚠️

The captioner's handling of and adaptability to input duration (which directly determines what kind of clip it can annotate):
[Short clips (5–20 seconds) are the mainstream adaptation range] The vast majority of generation-model training clips fall in this range, and captioners are optimized accordingly. Koala-36M's average clip length is 13.75 seconds; Foley-Omni uses fixed 8-second clips.
[Frame-sampling strategy is a core engineering parameter, varying widely across teams] Open-Sora's PLLaVA uniformly samples 4 frames; Ovi samples 7 keyframes + the full audio track; MAGI-1 empirically settled on "4–12 frames per clip depending on duration" as the optimal trade-off between description accuracy and compute; the CogVideoX teacher chain samples 1 frame every 2 seconds for dense image captioning; Panda-70M's BLIP-2 branch randomly samples a single frame from the 0.3N–0.7N frame range; Aria officially claims it can generate a caption for a 256-frame video within 10 seconds (this throughput advantage was the key reason Allegro selected it).
[Dedicated approaches for long videos] ShareCaptioner-Video's DiffSW is the only scalable scheme designed for videos of arbitrary length — it first generates a detailed caption for the first frame, then processes subsequent frames in chronological order via a sliding window of length 2, with each step taking "the previous frame + its differential caption + the current frame" as input and outputting inter-frame changes (covering camera motion, object motion, character actions, and scene transitions); its complexity grows linearly rather than quadratically with frame count, and it supports clip summarization (rapidly summarizing already-processed segments without reprocessing frames). AVoCaDO's optimal output-length range is 2048–4096 tokens, with an ℛ_L length-regularization penalty applied beyond 4096. AVSCapBench videos run 30–120 seconds, representing a shift in evaluation focus toward medium-to-long videos.
[Uncertain] No captioner has published a duration-distribution histogram of its training data.

### Resolution/Aspect Ratio Distribution and Bucketing Strategy ⚠️

[Uncertain] This dimension is generally undisclosed in the captioner ecosystem, with only scattered evidence:
· ShareCaptioner-Video's official model card explicitly claims "support for videos of various durations, aspect ratios, and resolutions"; its base model InternLM-XComposer2-4KHD has 4K high-resolution understanding capability, a differentiating capability relative to contemporaneous captioners.
· Koala-36M's clips are uniformly 720p.
· AuroraCap uses token merging to compress visual tokens to handle long-sequence overhead, indirectly indicating that high resolution/long video input is a major cost driver.
· Panda-70M's UMT retrieval-based selection model uses a fixed 12 frames at 224×224.
· The mainstream practice is for captioners to operate on low-resolution sampled frames (captioning only needs semantic correctness, not pixel-level fidelity), so resolution has far less effect on captioning quality than on generation training — this explains why the dimension is generally not discussed.
· No captioner work reports a "resolution/aspect-ratio bucketing strategy" — this type of bucketing belongs to the generation-model training side, not the captioning side.

### Category/Domain Distribution and Mixture Strategy (Balance Control and Ratios for People, Actions, Scenes, Styles, etc.) ⚠️

[Concept balance is an explicit design goal for captioner training (one of the few works that address it explicitly)]
· SkyCaptioner-V1: trained on "about 2 million concept-balanced videos, curated from 10 million," the clearest category-balance statement in this ecosystem; its camera-motion sub-expert further supplements long-tail camera-movement types with "16K motion-axis-balanced synthetic data."
· AVoCaDO-SFT-107K's six-source mixture is itself a domain-mixture design: TikTok-10M 24K (UGC short video) + ShortVideo 18K + Shot2Story 20K (multi-shot narrative) + FineVideo 29K (the largest, education/lifestyle content) + YouTube-Commons 11K (long-tail general content) + CinePile 5K (film) — i.e., UGC as the main body, with film as a small high-quality supplement.
· AVSCap-130K's sources are AVoCaDO-107K + ASID-1M + FineVideo + TimeChatCap-40K + Movie101; the inclusion of Movie101 significantly increases the proportion of film-narrative content.
· Panda-70M's multi-teacher greedy set-cover is essentially an implicit form of "dispatching captioners by content type": the single best captioner covers only 30.8% of samples, all 31 combined cover 84.7%, and the greedily selected 8 cover 76.8% — indicating that videos of different domains require different captioners, and no single model can handle everything. This is the ecosystem's most important insight into domain distribution.
[Evaluation-side domain taxonomies] AVSCapBench videos come from three sources — YouTube / TikTok / Video-MME; Omni-Cloze covers 9 major domains and 47 sub-categories (the only work with a fully public taxonomy in this ecosystem); UGC-VideoCap focuses on TikTok UGC (1,000 clips); VDC's structured captions are divided into five categories — camera / short / background / main object / detailed (this is a field dimension, not a domain dimension).
[Generation-side domain requirements for captioners] Allegro directly uses Tag2Text's tag output as the basis for its people/objects/landscapes three-category distribution statistics — i.e., the captioner doubles as a data-distribution statistics tool; Goku's distribution-balancing design relies on semantic clustering of caption embeddings; LongCat-Video uses an LLM to name categories from clustering results on caption embeddings.
[Gap] Aside from SkyCaptioner-V1, no captioner publishes a domain histogram of its training data or the quantified benefit of its balancing strategy. [partly uncertain]

### Audio Category Distribution and Mixture (Proportions and Control Strategy for Speech/SFX/Music/Ambient Sound/Silence) — AV-Model-Specific Dimension ⚠️

This is the most differentiating dimension in the 2025 Q4–2026 audio-video captioner ecosystem, and AVSCapBench provides, for the first time, complete quantitative cross-model evidence:
[Explicit modeling of three audio categories] AVSCap's Acoustic Completeness criterion explicitly requires captions to cover Speech / SFX / Music simultaneously. Foley-Omni's three-field schema is structurally identical to Bandit (cinematic audio source separation, which naturally splits into speech / effects / music). MOVA uses a two-model division of labor: Qwen3-Omni-Instruct handles speech (ASR), Qwen3-Omni-Captioner handles non-speech sound and music.
[AVSCapBench event-recall (%) horizontal comparison table — the single most valuable data point in this ecosystem]
Model / Visual / Speech / Music / SFX / Synergy / Overall:
· AVSCap-7B: 59.33 / 69.45 / 40.36 / 30.82 / 57.70 / 60.44
· AVoCaDO-7B(9B): 50.59 / 70.42 / 38.71 / 19.25 / 29.13 / 49.31
· video-SALMONN-2-7B: 39.05 / 46.76 / 13.76 / 8.71 / 12.43 / 32.02
· Qwen3-Omni-30B: 41.85 / 49.08 / 9.34 / 8.68 / 16.19 / 35.29
· Qwen2.5-Omni-7B (bare base model): 34.78 / 13.92 / 4.02 / 7.22 / 7.00 / 21.53
· Gemini-3-Pro: 60.43 / 79.81 / 39.52 / 27.77 / 48.88 / 60.97
· Gemini-3-Flash: 58.14 / 79.78 / 39.46 / 32.34 / 48.94 / 60.54
[Conclusive observations] (1) Speech is the strongest audio dimension for every model (40–80 points), followed by music (4–40 points), with SFX the weakest (7–32 points) — the capability gap across audio categories is as large as 8×, directly limiting the training-data quality available for Foley/ambient-sound generation models; (2) a 7B open-source model (AVSCap 60.44) has now caught up to Gemini-3-Pro (60.97) on overall score, and even surpasses it on Synergy (57.70 vs 48.88), but still trails by 10 points on Speech; (3) bare Qwen2.5-Omni scores only 13.92 on Speech, proving that an omni base model without caption-specific training is entirely unusable.
[How the generation side handles audio-category mixture] Movie Gen retains both the AED's music-posterior probability and a music-caption signal simultaneously (because the music-caption model tends to hallucinate when no music is present); empirically this redundant combination gives the best controllability — direct engineering evidence of "audio category distribution" shaping captioner design. LTX-2's data-selection criterion is "contains significant and information-rich audio content," to ensure balance between visual and auditory content distribution (the exact ratio is undisclosed [uncertain]).

### Narrative Structure Distribution (Single-Shot vs. Multi-Shot, Average Clip Length, Shot-Count Distribution, Whether Native Audio Track Is Present) ⚠️

[The single-shot vs. multi-shot captioning divide] Mainstream generation-model training clips are single-shot (after splitting with tools such as PySceneDetect), so the vast majority of captioners are optimized for single shots. A few exceptions target multi-shot/narrative content:
· ShareCaptioner-Video's DiffSW explicitly models "scene transitions," one of the few open-source captioners that natively support cross-shot description.
· AVoCaDO's five-dimensional keypoint set includes "Spatio-temporal & Cinematography," which explicitly covers scene changes, temporal progression, and camera work.
· Shot2Story (20K of the AVoCaDO-SFT samples) and Movie101 (one of the sources of AVSCap-130K) are themselves multi-shot narrative datasets.
· CineDance uses Qwen3.5-27B for shot grouping and narrative-boundary determination; a bottom-up strategy achieves an F1 of 88.4%, with only 3.1% of sequences shorter than the 20-second soft threshold — the only work in this ecosystem to give a quantitative metric for narrative-structure parsing.
· MOVA's visual-annotation instructions explicitly require "focusing on scene transitions in the video."
[Whether native audio track is present] Vision-only captioners completely ignore the audio track (Panda-70M even explicitly disables Video-LLaMA's audio branch); audio-video captioners require samples to have a valid audio track — LTX-2's data-subset filtering condition is precisely "contains significant and information-rich audio content."
[Average clip length] Koala-36M averages 13.75 seconds; AVSCapBench spans 30–120 seconds; Foley-Omni uses fixed 8-second clips.
[Uncertain] No captioner has published a shot-count distribution histogram for its training data.

### Language/Accent Distribution (Data Foundation for Multilingual Lip-Sync Capability) ⚠️

[Bilingual/multilingual captioning capability (a few works that explicitly address it)]
· Seedance 1.0's captioner (Tarsier2 base) is explicitly trained on bilingual Chinese-English data to acquire bilingual captioning capability, freezing the visual encoder while doing full-parameter fine-tuning on the language model during training — the clearest first-hand requirement from the generation side on a captioner's language capability.
· LTX-2's in-house captioner is the most fine-grained publicly disclosed scheme for handling language/accent: dialogue transcription is annotated not only with text but also with the three attributes speaker, language, and accent. This schema design is critical to the data foundation for multilingual lip-sync capability.
· SkyCaptioner-V1 makes no explicit bilingual claim; SkyReels-V4's speech and singing content is transcribed by Whisper (multilingual).
[Multilingual support on the ASR side] Whisper-large-v3 (multilingual, ~1.55B) is the ecosystem-wide default choice; Alibaba's SenseVoice (~234M, supports Chinese/English/Cantonese/Japanese/Korean + emotion/event recognition) is used alongside Whisper and Qwen2.5-Omni by Apollo/Klear; ElevenLabs Scribe is used by InstructAV2AV for precise timestamps; the Qwen3-Omni series is natively multilingual.
[A widespread gap in accent handling] Aside from LTX-2, no captioner explicitly annotates accent; AVoCaDO's dialogue reward uses only (speaker, spoken content) pairs, with no language/accent field.
[Uncertain] No captioner or its training data publishes a language-distribution ratio.

## Filtering Pipeline

### Overall Filtering Funnel Structure (Number of Stages, Order)

The "filtering pipeline" for this entry has two distinct meanings that must be kept separate: (A) the cleaning of the captioner's own training data; (B) the position the captioner occupies within a generation model's data pipeline, and the filtering role it plays there.
[(A) The filtering funnel for captioner training data] Generally simple, typically a four-stage pattern of "teacher-model generation → LLM-score filtering → SFT → RL":
· AVoCaDO: Gemini-2.5-Pro separately generates a visual caption and an audio caption → the two captions plus the original video are fed back into Gemini-2.5-Pro to synthesize a temporally coherent multimodal caption → GPT-4.1 scores "synthesis completeness" from 1–5, keeping only scores ≥4 → SFT → GRPO.
· AVSCap: three stages — decoupled unimodal anchoring → cross-modal fusion → automated verification (tag-retention check + semantic-consistency check) → SFT → GRPO.
· The CogVideoX teacher chain: Panda-70M short captions → CogVLM dense image captions at one frame per 2 seconds → GPT-4 summarization by timestamp → fine-tuning LLaMA2 on 50K samples to replace GPT-4 → further distillation into the end-to-end CogVLM2-Caption. A classic example of "progressively cost-reducing a four-stage teacher chain."
· SkyCaptioner-V1: Qwen2.5-VL-72B produces a general description + three sub-expert captioners (shot/expression/camera motion) supply cinematography-specific dimensions → fusion → distillation into a unified 7B model.
· Panda-70M: 31 candidate captioners generate in parallel → a user study performs greedy set-cover to select 8 → UMT-large fine-grained video-text retrieval selects the best. The only work to turn "captioner selection" itself into a filtering stage.
[(B) The captioner's position within the generation pipeline] Three typical placements:
· A semantic gate in the middle of the funnel: Allegro places Tag2Text at stage 6, with its tag output feeding directly into stage 7's CLIP-similarity filtering as the text-side input — the captioner is simultaneously an upstream dependency of the filter.
· Full-volume captioning at the end of the funnel: most works (HunyuanVideo, Step-Video, Movie Gen, Seedance) caption only after all filtering stages have passed, because captioning is the most expensive step.
· A tiered captioning pyramid: Open-Sora 2.0 uses the open-source LLaVA-Video for massive low-resolution (256px) data, then switches to Qwen 2.5 Max to re-caption a curated 5M high-resolution (768px) subset — a "coarse labels for the base layer, fine labels for the top layer" scheme strictly matched to the data pyramid, a core piece of the low-cost strategy. Movie Gen's 70% 8B / 30% 70B mix is another implementation of the same idea.
· Online captioning (rare): UniVerse-1 places annotation inside the training loop, forcing it to choose a lightweight model (Qwen2.5-Omni + Whisper) since it cannot afford the per-sample inference cost of a 120B-class model — the only public case of the trade-off between "annotation timing" and "annotation model capacity."

### Quantitative Funnel Retention Rates (Input/Output Volume and Final Retention Rate at Each Filtering Stage, e.g., Apollo's 27%) ⚠️

[Uncertain] The captioner ecosystem generally lacks quantitative retention-rate disclosure; only scattered figures exist:
· SkyCaptioner-V1: training data curated from 10 million down to about 2 million concept-balanced videos, a 20% retention rate — the only clearly stated captioner training-data retention rate in this entry.
· AVoCaDO: GPT-4.1 scores synthesis completeness 1–5, keeping only ≥4, but the pass rate at this step is undisclosed.
· Tarsier2: recaptions 1 million public-dataset videos, ultimately releasing 585K (Tarsier2-Recap-585K); if all 585K came from that same 1M, the retention rate would be about 58.5%, but the paper does not confirm the two batches are identical (it could instead be active sampling for release rather than filtering) [uncertain].
· Panda-70M's greedy set-cover reports "coverage" rather than "retention": the single best model covers 30.8%, 8 models cover 76.8%, all 31 cover 84.7%; it also reports a human-to-human agreement rate of only 44.9%, indicating that "the best caption" is itself highly subjective.
· Neither AVSCap-130K nor AVoCaDO-SFT-107K gives stage-by-stage "original candidate volume → final retained volume" figures.
· The most complete end-to-end retention rate on the generation side comes from Apollo/Klear (27%), but that covers the entire data funnel, not just the captioning step.

### Shot Splitting Method (PySceneDetect/In-House Model/Shot-Aware Splitting)

Captioners themselves typically do not perform shot splitting (splitting is done upstream), but there are three types of interaction:
[The captioner takes on narrative grouping] CineDance uses Qwen3.5-27B for shot grouping and narrative-boundary determination (a bottom-up shot-index grouping strategy outperforms having the LLM output timestamps directly), achieving F1 = 88.4% with only 3.1% of sequences shorter than the 20-second soft threshold — a representative case of the captioning model directly taking on splitting decisions.
[The captioner describes transitions] ShareCaptioner-Video's DiffSW explicitly identifies scene transitions; MOVA's visual-annotation instructions explicitly require "focusing on scene transitions in the video"; AVoCaDO's Spatio-temporal & Cinematography dimension covers scene changes.
[Splitting quality in turn determines captioning quality] One of Koala-36M's core contributions is precisely a "more accurate and efficient transition-detection method"; its ablation shows that training on Koala-all versus Panda-70M improves VBench subject consistency by +1.1% and background consistency by +2.4%, with the paper attributing this temporal-quality improvement to more accurate shot splitting — the single quantified piece of evidence for the "splitting → caption consistency → generation quality" chain. CineDance's artifact audit shows a non-compliance rate of 2.8% for CineDance-1M versus 37.4% for Koala-36M (a 13.4× improvement).
[Common upstream tools] PySceneDetect remains the industry default; in-house transition-detection models (Koala-36M, CineDance) mainly target missed detections of gradual transitions and rapid cuts.

### Quality Filtering (Aesthetic Score, Sharpness, OCR Text Filtering, Black-Bar/Watermark/Logo Detection)

The captioner ecosystem's "quality filtering" runs in two directions:
[Direction one: the captioner as a component of quality filtering]
· Allegro: Tag2Text's output serves as the text-side input for CLIP-similarity filtering — the captioner is embedded directly into the filtering chain.
· The Motif-Video 2B batch (alongside Mochi/MAGI/Motif) also deploys PaddleOCR-VL (served via vLLM) for on-frame text detection, an OCR-filtering branch within the captioning chain.
· InstructAV2AV uses Grounded-SAM-2 to provide instance-level mask anchors, and TalkNet for active-speaker detection — structured annotation tools that double as quality gates.
· Traditional shallow filters (LAION aesthetic score, DOVER technical score, sharpness, black-bar/watermark/logo detection) run in parallel with, not in series after, the captioner, typically completing before captioning to save on captioning compute.
[Direction two: quality control of the captioner's own output] This is the core problem in this ecosystem, with four mainstream approaches:
(1) LLM score-based filtering: AVoCaDO uses GPT-4.1 to score synthesis completeness from 1–5, keeping only ≥4;
(2) Automated consistency verification: AVSCap's automated verification (tag-retention check + semantic-consistency check);
(3) Retrieval-based selection: Panda-70M uses UMT-large (ViT-L/16 + BERT-large, VTC+VTM dual loss + hard negative mining; unselected candidate captions get a weight of 1.0, other in-batch negatives get 0.01; 12 frames at 224×224; AdamW lr 2e-5; batch size 32; 10 epochs; 8×A100-80G) to select the best caption from multiple candidates, achieving R@1 of 35.90% after fine-tuning (versus only 21.82% for the pretrained UMT);
(4) Post-training for hallucination suppression: Tencent Hunyuan 1.5 applies OPA-DPO (a preference-optimization method for multimodal hallucination) as RL post-training for its three caption models; video-SALMONN 2's MrDPO rewards both completeness and factuality simultaneously, reducing the 7B model's caption error rate by 28% relative to baseline; Tarsier2 uses model-based sampling to construct preference pairs for DPO.
[The generation side's fallback] Step-Video-T2V does not apply dedicated hallucination-suppression post-training to its caption model, instead falling back on human review during the SFT stage plus CLIP Score alignment filtering at stage 6 — representative of the pragmatic "if the captioner isn't good enough, patch it with downstream filtering" approach.

### Motion Filtering (Optical-Flow/Motion-Score Thresholds, Removing Static and Shaky Footage)

There are three points of interaction between captioners and motion filtering:
[The captioner's output supplies motion information used for filtering] Tarsier2 naturally describes camera-motion types (zoom in / pan right, etc.); Goku's paper explicitly cites this as the key advantage of selecting it for video-level captioning — no separate camera-motion annotation module is needed to obtain cinematography labels.
[Dedicated motion classifiers divide labor with the captioner] The mainstream approach is to hand motion recognition to a lightweight classifier rather than a general-purpose VLM: Movie Gen trains a 16-class camera-motion classifier, prefixing high-confidence predictions onto the caption; LongCat-Video's camera motion (pan/tilt/zoom) is handled by a separately trained lightweight classifier rather than a VLM (presumably for cost and precision reasons); HunyuanVideo has its own in-house camera-movement classifier (upgraded in version 1.5 to dual granularity — clip-level and temporal-level); SkyCaptioner-V1's camera-motion sub-expert is classification-based, trained on 93K high-confidence human annotations plus 16K motion-axis-balanced synthetic data, achieving 89% single-type motion accuracy on a 15K-sample test set.
[Motion filtering itself (optical-flow/motion-score thresholds, removing static and shaky footage) belongs upstream of the captioner] Foley-Omni uses a motion∈[0.1, 3.2] threshold; InstructAV2AV uses a CoTracker3 motion threshold; Open-Sora Plan uses an LPIPS upper bound of 0.3 (above which shaking/flickering appears, a conclusion validated via 2,000 manually spot-checked samples). All of these run before the captioner.
[Ecosystem-wide judgment] "Camera-motion recognition uses dedicated classifiers, content description uses VLMs" is a stable division of labor from 2024–2026, because VLMs' judgment of camera motion is unreliable and hard to quantitatively verify, whereas classifiers can output a confidence score usable for filtering. SkyCaptioner-V1's approach (distilling classifier results back into the unified 7B captioner) is a fusion attempt between the two routes, with its cinematography-field average accuracy of 76.3% (shot type 93.7%, camera angle 89.8%, camera position 83.1%, camera motion 85.3%) significantly exceeding larger general-purpose models such as Qwen2.5-VL-72B.

### Deduplication Method (Exact Deduplication and Embedding-Based Semantic Deduplication Recorded Separately) ⚠️

[Uncertain] This is one of the least-disclosed fields in the captioner ecosystem.
[Known scattered evidence]
· AVSCap-130K's sources include AVoCaDO-107K, whose own sources (FineVideo, Shot2Story, etc.) are public datasets that themselves overlap with one another — the snowball reuse of "dataset stacked on dataset" introduces significant but unquantified cross-dataset duplication risk, and neither paper discusses deduplication.
· Tarsier2-Recap-585K is produced from 1M public-dataset videos, whose sources (VATEX, TGIF, LSMDC, etc.) are known to overlap with one another as well; the deduplication strategy is undisclosed.
· Panda-70M's greedy set-cover deduplicates "captioners" (reducing 31 down to 8), not video samples.
· Goku uses Qwen2 as an LLM fuser to merge keyframe captions and video captions into a "unified, non-redundant, non-contradictory" final description — this is textual redundancy elimination at the caption level, distinct from sample-level deduplication.
· LongCat-Video uses an LLM to name categories from clustering results on caption embeddings; caption-embedding clustering has the technical prerequisites for semantic deduplication, but the report does not state whether it is used for that purpose.
[Ecosystem-wide judgment] Exact deduplication (hashing) and semantic deduplication (embedding) of video samples are generally performed upstream of the captioner, executed by the generation-side team, and rarely disclosed; deduplication of caption text itself (avoiding homogenized template sentences) is almost never discussed — given that CogVideoX's Appendix G prompt explicitly bans clichés such as "The video presents / depicts / showcases" and "throughout the video," caption homogenization is clearly a real problem, but the industry relies on prompt constraints rather than post-hoc deduplication to address it.

### VLM/LLM as Quality Inspector (Multimodal-LLM Quality Scoring and Mismatch Removal; the 2026 Trend From Shallow Scorers to Large-Model Semantic Judgment)

This is the most prominent trend in this ecosystem for 2025–2026, and the captioner ecosystem plays both the "judged" and "judging" roles simultaneously:
[Trend one: LLM/VLM as caption-quality judge (shallow scorers → large-model semantic judgment)]
· AVoCaDO: GPT-4.1 scores Gemini-synthesized captions on synthesis completeness from 1–5, keeping only ≥4; in the GRPO stage, GPT-4.1 further judges five-dimensional checklist coverage as the reward ℛ_C.
· AVSCap: automated verification performs tag-retention checks and semantic-consistency checks; the hybrid reward for GRPO includes an audio-visual consistency term.
· MOVA: GPT-OSS-120B (a 120B open-source model) handles caption fusion + cross-modal consistency checking — placing the largest model in the fusion/adjudication stage rather than the perception stage, a typical cost-effectiveness trade-off.
· AuroraCap proposes VDCScore: a divide-and-conquer strategy that converts long-caption evaluation into multiple short QA pairs judged by an LLM — representative of caption evaluation shifting from n-gram metrics like CIDEr/BLEU toward LLM semantic judgment.
· Omni-Cloze takes the opposite approach: it uses 2,000 clips / 70,000 fine-grained cloze questions to sidestep LLM-judge noise, representing a challenge to and correction of LLM-as-judge reliability.
[Trend two: the captioner as data quality inspector]
· InstructAV2AV: Qwen3-Omni both generates editing instructions and performs five-dimensional verification scoring — the paper itself acknowledges that "generation and acceptance sharing the same source is a methodological hazard of this pipeline," a rare instance of self-criticism in this ecosystem.
· Vidu S1 uses an omni model for discrimination during the quality-filtering stage (details undisclosed).
· Foley-Omni uses Gemini 2.5 Pro for annotation, then uses Bandit (cinematic audio source separation) for acoustic post-hoc verification to correct visual hallucinations, with a threshold of −35 dB — "large-model annotation + signal-level verification" is an effective paradigm for suppressing multimodal hallucination.
[Methodological hazards common across the ecosystem] (1) benchmark and model share the same source: AVSCap is both the author of AVSCapBench and its top scorer (60.44), posing overfitting risk, so its scores need cross-validation against third-party benchmarks (UGC-VideoCap, Omni-Cloze); (2) the teacher is also the judge: most works use the Gemini/GPT family as both teacher and judge, biasing scores toward their own output style; (3) a ceiling on human agreement: Panda-70M reports a human-to-human caption-preference agreement rate of only 44.9%, meaning the "accuracy" ceiling of LLM-judge itself is inherently blurry.

### Safety and Compliance Filtering (NSFW, Copyright, Face/Privacy) ⚠️

[Uncertain] This field is almost entirely blank in the captioner ecosystem, with only indirect evidence:
· NSFW/copyright/facial-privacy filtering is generally performed upstream of the captioner by the generation-side team; captioner papers uniformly do not discuss it.
· The only relevant design is LTX-2 captioner's "comprehensive yet factual" principle — describing only what is seen and heard, explicitly banning emotional interpretation. This is both an anti-hallucination measure and, objectively, reduces the risk of captions making potentially biased judgments about people's subjective states — the closest thing to a "safety design" publicly documented in this ecosystem.
· AVoCaDO-SFT includes TikTok-10M and ShortVideo subsets involving real-person UGC content, without discussing face/privacy handling.
· Works involving speaker-identity annotation (AVoCaDO's (speaker, content) pairs, LTX-2's speaker/language/accent, CineDance's character anchor-token binding) all annotate content highly related to personal identity in the context of generation, but none discuss privacy compliance.
· Safety filtering by generation-side teams (Movie Gen, Veo 3) is documented in their respective entries and has no direct coupling to captioner selection.

## Captioning Approach

### Caption Models Used (In-House VLM/Open-Source Model, Model Scale) ⚠️

The core field of this entry. A complete ecosystem map organized by "who is used for captioning":

[First tier: open-source captioners trained specifically for video captioning (highest downstream reuse rate)]
· Tarsier2-7B (ByteDance, 2025-01): pretraining 11M→40M video-text pairs, SFT for fine-grained temporal alignment, model-based sampling to construct preference pairs + DPO. DREAM-1K F1 exceeds GPT-4o by 2.8% and Gemini-1.5-Pro by 5.8%; human evaluation +8.6% / +24.9%; the first model to break 40% overall recall on DREAM-1K; SOTA on 15 public benchmarks. Its natural ability to describe camera-motion type is a unique advantage. Reused downstream by Seedance 1.0 (as a base model, with a frozen visual encoder + full-parameter LLM fine-tuning + bilingual Chinese-English training), Goku (video-level captioning, complementing InternVL2.0's keyframe captioning, fused via Qwen2), and LongCat-Video (temporal enhancement) — the single most widely adopted open-source captioner on the generation side.
· ShareCaptioner-Video (USTC + Shanghai AI Lab + CUHK, 2024-06): base model InternLM-XComposer2-4KHD, SFT-trained on 40K GPT-4V dense captions, producing 4.8M high-quality aesthetic video annotations. Its core method, DiffSW (differential sliding window), converts "full-frame-to-caption" into a differential-description task: it first generates a detailed caption for the first frame, then processes subsequent frames in chronological order via a length-2 sliding window, with each step taking "the previous frame + its differential caption + the current frame" as input and outputting four categories of change — camera motion/object motion/character action/scene transition. It supports arbitrary duration, aspect ratio, and resolution, and comes with clip summarization (no need to reprocess frames). Open-Sora Plan v1.3 uses its annotations directly for the stock-footage portion.
· CogVLM2-Caption (Zhipu, open-sourced 2024-09-19): CogVLM2-Video + Llama3 base, roughly 12B scale, fine-tuned from dense captions produced by the Panda-70M→CogVLM→GPT-4→LLaMA2 teacher chain — the only reproducible link in the CogVideoX data pipeline.
· SkyCaptioner-V1 (Kunlun Wanwei, 2025-04): Qwen2.5-VL-7B-Instruct base (a small model to support hundred-million-scale throughput), a knowledge-distillation fusion approach — Qwen2.5-VL-72B produces a general description + three sub-expert captioners (shot type/camera angle/camera position, facial expression and emotion intensity, 6DoF camera motion) → fusion → distillation into a 7B model. Average accuracy on cinematography-specific fields is 76.3%, significantly exceeding larger general-purpose models.
· AuroraCap (academic, 2024-10): token merging to compress visual tokens, three-stage training on 20 million image/video-text pairs, Flickr30k CIDEr 88.9 (vs. GPT-4V 55.3, Gemini-1.5-Pro 82.2), paired with the VDC benchmark and the VDCScore metric.

[Second tier: general-purpose VLMs used directly as captioners (lowest cost, most common)]
· The Qwen series is the de facto standard for 2025–2026: Qwen2-VL-7B-Instruct (the workhorse of Open-Sora Plan v1.3), Qwen2.5-VL (LongCat-Video's structured-attribute discrimination; SkyCaptioner's base model), Qwen3-VL (UniTalking's visual captioning; MOVA's inference-side prompt enhancement), Qwen3-VL-30B-A3B (the sole source of all captions and tags in Motif-Video 2B), Qwen3.5-27B / Qwen3.5-35B-A3B (CineDance's narrative parsing and visual annotation). Choosing a sparse MoE architecture (about 3B active parameters) to balance throughput and quality is a new trend for 2026.
· The LLaVA family: PLLaVA-13B (the workhorse for Open-Sora 1.x, configured with 2×2 spatial pooling, sampling 4 frames; the documentation explicitly explains that GPT-4V was not used because "20 seconds/sample is too slow"), LLaVA-Video (used in Open-Sora 2.0's low-resolution stage and as LongCat-Video's main captioner), LLaVA v1.6-Mistral-7B, and the LLaVA fine-tuned on GPT-4V seed captions used by Koala-36M.
· InternVL2.0 (Goku's keyframe captioning), Aria 25.3B MoE / 3.9B active (Allegro's fine-grained captioner, capable of generating a caption for a 256-frame video within 10 seconds), MiMo-VL-7B-RL (MOVA's visual annotation), Tag2Text (Allegro's coarse-grained tagger).

[Third tier: omni-modal/joint audio-video captioners (an explosion starting 2025 Q4)]
· AVoCaDO (Kuaishou Kling + CASIA et al., 2025-10): Qwen2.5-Omni-7B base (marked 9B end to end on HF), AVoCaDO-SFT-107K → GRPO. Reward = ℛ_C (five-dimensional checklist coverage, judged by GPT-4.1) + ℛ_D (dialogue F1, speaker accuracy and content similarity, with edit distance + dynamic programming finding the optimal aligned subsequence, threshold 0.6) + ℛ_L (length regularization, penalizing >4096 tokens). Apache-2.0.
· AVSCap-7B (NJU-LINK + Kuaishou Kling, 2026-07): same base model, AVSCap-130K → GRPO (length control + speech preservation + AV consistency); AVSCapBench overall score of 60.44 catches up to Gemini-3-Pro (60.97). The paper's key conclusion: RL gains > gains from scaling SFT data.
· video-SALMONN 2 / 2+ (Tsinghua Dept. of Electronic Engineering + ByteDance, iterating from 2025-06 through 2026-02): LLaVA-OneVision + LoRA-added audio capability, 3B/7B/72B, MrDPO (multi-round DPO that periodically resets the reference policy with a newly trained lightweight adapter), caption error rate reduced 28% relative to baseline, Apache-2.0.
· Qwen3-Omni-Captioner (Alibaba, 2025-09, audio-specific) and Qwen3.5-Omni (2026-03, API-only): the latter directly positions "script-level fine-grained description, automatic segmentation, timestamp annotation, and description of character-audio relationships" as a data-generation capability for video synthesis models, with Omni-Cloze score 64.8 > Gemini-3.1 Pro's 57.2. Paired with Omni-Detective (an agentic Query-Observation loop annotation framework).
· Qwen2.5-Omni / Qwen3-Omni (widely used as audio-side or fusion-side captioners by UniVerse-1, UniTalking, MOVA, SkyReels-V4, InstructAV2AV, Apollo/Klear, LongCat Avatar 1.5).

[Fourth tier: closed-source APIs as teachers or captioners for high-value subsets]
· GPT-4V / GPT-4 / GPT-4.1: ShareGPT4Video-40K, Koala-36M seed captions, MiraData structured captions, CogVideoX summarization, AVoCaDO quality checking.
· The Gemini series (2.5-Pro / 3-Pro / 3-Flash): AVoCaDO's teacher, all 500K MTSS annotations in Script-a-Video, Harmony's 4 million audio-video annotations, Foley-Omni's primary annotator (chosen for its native support of joint video+audio input, since vision-only VLMs cannot judge audio presence), Apollo/Klear's high-value subset, and the "multiple Gemini models" used by Veo 3.
· Qwen 2.5 Max (used by Open-Sora 2.0 to re-caption a curated 5M high-resolution subset).

[Fifth tier: fully closed-source, in-house on the generation side]
· OpenAI Sora: a "highly descriptive captioner model" (following the DALL·E 3 re-captioning approach), with Sora 2 disclosing nothing at all.
· Meta Movie Gen: LLaMa3-Video, with 8B and 70B variants each fine-tuned separately for video captioning; 70% of captions in the training set come from the 8B model and 30% from the 70B model — the only work to publicly disclose an exact "small/large model mix ratio." It also has a 16-class camera-motion classifier. Its audio side uses four models working together (audio-quality prediction on a 1–10 scale, AED for judging voice/singing presence and music posterior probability, a general audio-caption model, and a music-caption model).
· Tencent Hunyuan: the original version used a single in-house VLM to output structured JSON captions; version 1.5 upgraded to a three-model division of labor (image captioning, video captioning, image-to-video instruction-style captioning) + a camera-motion recognition model, using OPA-DPO to suppress hallucination.
· Lightricks LTX-2: a newly developed captioning system built specifically for joint audio-video generation, describing both visual and auditory tracks in detail.
· StepFun's Step-Video: an in-house VLM, with the TI2V version specially fine-tuned to strengthen descriptions of "object motion dynamics" and "camera motion."
· Kuaishou Kling 3.0 Omni: officially states only that it has a "video-description enhancement" step, with the model itself undisclosed [uncertain]; the same team's publicly documented approach is the Koala-36M LLaVA fine-tuning route and its in-house Kwai Keye-VL (arXiv:2507.01949).
· ByteDance Seedance 1.5 pro / 2.0: claimed to have an "advanced captioning system" providing professional-grade descriptions for both video and audio, with the base model unspecified [uncertain; presumably switched to the in-house Seed-VL series].

[Key selection patterns (synthesized across entries)]
(1) Parameter counts generally converge to 7B: ShareCaptioner, Tarsier2, SkyCaptioner-V1, AVoCaDO, AVSCap, and MiMo-VL are all 7B-class, because hundred-million-sample captioning must keep per-sample inference cost under control; placing large models (GPT-OSS-120B, Qwen2.5-72B) in the fusion/adjudication/teacher stages rather than the perception stage is a consistent cost-effectiveness trade-off across the industry.
(2) Multi-model division of labor beats a single model doing everything: Panda-70M's evidence is the strongest — the single best captioner covers only 30.8% of samples, while a combination of 8 covers 76.8%. Goku (image VLM + video VLM + LLM fusion), MOVA (vision + ASR + non-speech + 120B fusion), and SkyCaptioner (general + three sub-experts) are all implementations of this idea.
(3) Camera-motion recognition uses dedicated classifiers rather than VLMs: Movie Gen's 16-class classifier, LongCat's lightweight classifier, Hunyuan's independent classifier, SkyCaptioner's 6DoF sub-expert (89% accuracy) — a consistent industry choice.
(4) Distillation for cost reduction is standard practice: GPT-4V/Gemini teacher → 7B student, done in almost every work. Script-a-Video provides the most complete quantification of distillation gains (see data_ablation).
(5) The 2026 watershed is "can it hear": vision-only captioners cannot serve joint audio-video generation models, and omni captioners have become a hard requirement for AV generation models.

### Caption Density and Structuring Level (Short/Long/Dense Descriptions, Structured Fields Such as Camera Motion, Style Tags)

[Length spectrum: from 13.2 words to 824.2 words, a 60× range] Panda-70M averages 13.2 words (a single-sentence summary, with the prompt "Please faithfully summarize the video in one sentence.") → Movie Gen captions average about 100 words → Koala-36M averages 202.3 words → AVoCaDO's optimal range is 2048–4096 tokens. The choice of captioning model directly determines caption length, which in turn determines the prompt-length sensitivity range of downstream T2V models.

[Four tiers of structuring level]
· Tier one · Pure free-text single sentence: Panda-70M. Suitable for coarse alignment in large-scale pretraining, insufficient to support controllable generation.
· Tier two · Long dense prose with fixed-pattern insertions: the mainstream approach. Allegro covers subject attributes, inter-subject interaction, background, environment, style, mood, camera angle and motion, and temporal change, with camera motion explicitly inserted via the fixed pattern "Camera [MOTION_PATTERN]" while the rest is expressed in prose; Movie Gen prefixes high-confidence predictions from its 16-class camera-motion classifier onto the caption; a Step-Video-TI2V example reads "a flock of birds flying over a tree at sunset, camera pans left." This "prose body + structured fragment" hybrid form is a compromise between controllability and naturalness.
· Tier three · Explicit multi-field structuring: VDC defines five fields — camera / short / background / main object / detailed; Koala-36M has six categories of structured information; SkyCaptioner-V1 outputs cinematography-specific fields such as shot type/camera angle/camera position/camera motion/facial expression and emotion intensity; Tencent Hunyuan outputs structured JSON captions; LongCat-Video uses Qwen2.5VL to determine shot scale, shot type, realism, animation style, and color tone; CineDance produces shot-level five-dimensional attributes + transition type + local character list + active scene + shot description + transition description.
· Tier four · Multi-granularity coexistence + random sampling: SkyReels-V4 generates short/long/structured descriptions across three tiers; Motif-Video 2B's three caption variants are sampled with probabilities 0.5/0.3/0.2 (the paper candidly admits this is "a pragmatic recipe choice rather than an isolation-validated optimality claim"); UniTalking retains outputs from both the "annotate separately then concatenate" path and the "unified model fusion annotation" path, randomly sampling between them during training — leaving the choice of annotation strategy to random sampling so the model adapts to both text distributions simultaneously.

[DiffSW: the only structure designed for scalability] ShareCaptioner-Video's differential sliding window organizes captions into a chained structure of "detailed first-frame description + step-by-step differential descriptions," naturally carrying incremental temporal information, and supporting linear extension to arbitrary length as well as rapid post-hoc summarization.

[Style constraints (a severely underrated engineering detail)] CogVideoX's Appendix G prompt explicitly bans clichés such as "The video presents / depicts / showcases" and "throughout the video," as well as line breaks; LTX-2's principle is "comprehensive yet factual," describing only what is seen and heard and explicitly banning emotional interpretation (to prevent captions from introducing uncontrollable subjective signals during T2V training). These two are the most valuable reference points for caption style design across the entire ecosystem.

[Alignment with inference-side prompts (a downstream constraint on caption structure)] Open-Sora Plan explicitly demonstrates that a distribution mismatch between training captions (dense, long text) and user prompts (<10 words) harms generation quality; to address this it LoRA-fine-tunes LLaMA-3.1-8B-Instruct (rank 64, batch 32, 1 epoch, 30 minutes on a single H100) to train a prompt refiner; Seedance 1.0's PE model shares the same origin as its captioner to ensure structural alignment; MAGI-1 distills a large MLLM's enhanced prompts into a roughly 7B small model (about 2 million samples of corpus) to reduce inference latency; Goku uses GPT-4o to expand short GenEval prompts, reaching a score of 0.76, indirectly proving that model performance is highly dependent on long prompts consistent with the dense captions used during training. **The choice of caption structure effectively locks in the requirement for a matching prompt rewriter on the inference side** — this is the single most important systemic conclusion in this ecosystem.

[The upper bound imposed by the text encoder] Allegro uses T5 (a 512-token cap, with the paper qualitatively arguing T5 is superior to mT5); Mochi 1 uses a single T5-XXL; Foley-Omni and UniVerse-1 use UM-T5/umT5; HunyuanVideo-Foley uses the CLAP text encoder (about 77 tokens, because CLAP's text-embedding space is naturally aligned with audio semantics); LTX-2 uses Gemma3-12B; Motif switches to T5Gemma2 starting at stage 4; Apollo/Klear uses Qwen2.5-7B. The encoder's token cap directly truncates the usable upper bound of caption density.

### Joint Audio-Video Caption Structure (Whether It Covers Both Visual and Auditory Tracks Simultaneously, Whether Split Into Separate Fields, e.g., LTX-2's Full Soundscape Description, Script-a-Video's Factorized Streams, Foley-Omni's Three Fields)

The design of joint audio-video caption schemas is the most active area of innovation in this ecosystem for 2026, having crystallized into four identifiable paradigms:

[Paradigm one · fully fused soundscape (a single caption narrating vision and audio together)] LTX-2's in-house captioner is the benchmark: on the visual side it covers camera motion, lighting, and subject behavior; on the audio side it covers music, ambient sounds, and precise dialogue transcription with the three attributes speaker / language / accent attached; its design principle is "comprehensive yet factual," explicitly banning emotional interpretation. AVoCaDO belongs to the same category, outputting free-form long text (not JSON), with its structuring embodied in the five-dimensional breakdown of the reward checklist: Static Entity Description, Dynamic Action & Interaction, Auditory Elements (speech/music/ambient sound and narrative sound effects), Spatio-temporal & Cinematography, and **Cross-modal Narrative Logic** (audio and visual mutually explaining/complementing/guiding each other — its core innovation dimension).

[Paradigm two · factorized streams (split into independent fields, each its own track)] Script-a-Video's MTSS (Multi-Track Structured Script) is the representative example, with 500K annotations generated by Gemini-2.5-Pro and then distilled into Qwen3-Omni-MTSS-FT. UniVerse-1 uses a single Qwen2.5-Omni to output three independent fields in parallel in one pass — "verified speech content / descriptive video caption / ambient-sound caption" — without fusing them. UniTalking produces four formats (detailed and short visual captions from Qwen3-VL, a Whisper-V3 transcription, an audio caption from Qwen3-Omni-Captioner, and a fused unified description from Qwen3-Omni), randomly sampled during training.

[Paradigm three · fixed three-field schema (matched to the natural categories of audio-source separation)] Foley-Omni's three-field design (speech / effects / music) is structurally identical to the output of Bandit (cinematic audio source separation) — using a general-purpose music-separation model (such as Demucs's vocals/drums/bass/other) would not map directly. This is the best example of schema design co-developed with its verification tool. MOVA's bimodal division of labor (Qwen3-Omni-Instruct handles speech, Qwen3-Omni-Captioner handles non-speech and music, GPT-OSS-120B fuses and performs cross-modal consistency checking) is structurally equivalent to a three-field post-fusion scheme.

[Paradigm four · script-level structuring (with timestamps and character binding)] Qwen3.5-Omni offers "script-level fine-grained description, including automatic segmentation, timestamp annotation, and description of character-audio relationships," currently the only general-purpose model natively supporting this form. CineDance goes the furthest: shot-level five-dimensional attributes + transition type + local character list + sentence-level ASR + shot-level audio prompts (music/ambient sound/sound effects) + character timbre description, plus binding ASR sentences to character anchor tokens.

[Formalization of quality criteria (AVSCap's contribution)] A high-quality omni-modal caption must satisfy three criteria simultaneously: Acoustic Completeness (covering Speech / SFX / Music), Visual Completeness (environment, people, actions, object interaction, camera motion, OCR), and Audio-Visual Synergy (explicit binding and temporal alignment of audio-visual events). This is currently the most actionable schema acceptance standard.

[Implicit requirements on input form] Ovi's approach reveals a key constraint: its captioning input is "7 uniformly sampled keyframes + the full audio track," meaning the caption model itself must have audio-understanding ability, and the sparse 7-frame sampling implies the visual description is event/semantic-level rather than densely per-frame — joint AV captioning is naturally "coarse on vision, fine on audio" in asymmetric form.

### Dialogue Transcription and Speaker Attribute Annotation (ASR Transcription + Speaker Identity/Language/Accent/Emotion)

[Three approaches to ASR model selection]
· External dedicated ASR (most common): Whisper-large-v3 (OpenAI, ~1.55B) is the de facto standard, used broadly by SkyReels-V4, UniTalking, Unison (evaluation side), Apollo/Klear, UniVerse-1, and CineDance (for evaluating WER/CER); Alibaba's SenseVoice (~234M, Chinese/English/Cantonese/Japanese/Korean + emotion/event recognition) is used by Apollo/Klear alongside Whisper and Qwen2.5-Omni; ElevenLabs Scribe is used by InstructAV2AV for precise speech timestamps.
· Native transcription from omni models (a 2026 trend): Ovi explicitly forgoes an independent ASR module, with dialogue transcription produced directly by the captioning MLLM from the audio track; UniVerse-1 uses Qwen2.5-Omni to simultaneously output verified speech content; CineDance uses Qwen3-Omni-30B-A3B to handle sentence-level ASR, shot-level audio prompts, and character timbre description as three sub-tasks in one unified model, citing its far superior performance on speaker-attribution tasks compared to dedicated diarization tools (Pyannote-3.1 only 62.7%, DiariZen 63.1%).
· RL-elicited transcription ability (AVoCaDO's innovation): rather than using a separate ASR module, it forces the model to learn transcription during the GRPO stage via the dialogue reward ℛ_D — dialogue is extracted into (speaker, spoken content) structured pairs, with ℛ_D = the F1 of speaker-identification accuracy and content similarity, where content similarity uses edit distance + dynamic programming to find the optimal aligned subsequence, threshold 0.6. This is currently the most practical publicly documented scheme for optimizing dialogue-transcription accuracy.

[Spectrum of completeness in speaker-attribute annotation]
· Most complete: LTX-2 annotates the three attributes speaker / language / accent — the data foundation for multilingual lip-sync capability, and the only scheme in the entire ecosystem that explicitly annotates accent.
· Moderate: CineDance's character timbre description + binding of ASR sentences to character anchor tokens (a windowed approach lifts Qwen3-Omni from 67.2% to 95.4%, versus 82.8%–87.4% for the Gemini series); Apollo/Klear extracts attributes such as gender and age via its audio-model stack; InstructAV2AV uses TalkNet for active-speaker detection, annotating who is speaking and their spatiotemporal position.
· Simplest: AVoCaDO does only (speaker, content) pairs, with no language/accent field.

[Methodological comparison of speaker diarization (CineDance Tab. 5, the only quantitative comparison in this ecosystem)] On a 100-clip benchmark, Qwen3-Omni-30B + a sliding-window prompt achieves 83.1%, versus only 56.4% for whole-clip input — showing that the windowing design of the prompt has a far greater impact on speaker diarization than model choice itself.

[Downstream impact of speech quality] AVSCapBench shows Speech is the strongest dimension for all AV captioners (Gemini-3 series 79.8, AVoCaDO 70.42, AVSCap 69.45), but bare Qwen2.5-Omni scores only 13.92 — speech-transcription ability must be acquired through dedicated training.

### Geometric and Structured Annotation (Camera Parameters, Depth, 3D Point Tracks, Action Annotation, Explicit State) ⚠️

The captioner ecosystem takes a consistent strategy for geometric/structured annotation — "bolt on a dedicated model and splice the results into the caption" — rather than requiring the VLM to output geometric quantities directly:
[Camera parameters and camera motion (most mature)] Movie Gen's 16-class camera-motion classifier (high-confidence predictions prefixed onto the caption); SkyCaptioner-V1's 6DoF camera-motion sub-expert (classification-based, 93K human annotations + 16K motion-axis-balanced synthetic data, achieving 89% single-type motion accuracy on a 15K-sample test set, with overall camera-motion-field accuracy of 85.3%); Tencent Hunyuan's independent camera-movement classifier (version 1.5 has dual granularity — clip-level and temporal-level); LongCat-Video's lightweight pan/tilt/zoom classifier; Allegro embeds the fixed pattern "Camera [MOTION_PATTERN]" into the caption. Tarsier2 is one of the few general-purpose captioners that naturally describes camera-motion type.
[Instance-level spatial annotation] InstructAV2AV uses Grounded-SAM-2 (open-vocabulary detection + segmentation + video tracking) to produce instance-level masks, serving as anchors for "editable objects" and the editing region for mask-guided synthesis — the deepest involvement of geometric annotation in this ecosystem.
[OCR/text detection] Motif-Video 2B pairs with PaddleOCR-VL (served via vLLM) for on-frame text detection; AVSCap's Visual Completeness criterion explicitly includes OCR.
[Action/temporal annotation] ALIVE performs "decomposing sub-motion units" — breaking actions into sub-motion units and manually annotating the start/end timestamps of each complete micro-action, the most expensive fine-grained temporal annotation in this ecosystem; Tarsier2's SFT stage performs fine-grained temporal alignment; CineDance's shot grouping and narrative-boundary determination (F1 88.4%).
[Depth / 3D point tracks] [Uncertain] No mainstream captioner-ecosystem work has been found that outputs depth maps or 3D point tracks as a component of the caption — this type of annotation appears in world-model/robotics video data (such as the Cosmos series) but is not part of the standard T2V caption ecosystem.
[Explicit state annotation] CineDance's local character list + active scene is the design closest to "explicit state" that has been publicly documented.

### Synthetic Data Construction (Controlled Perturbation/Edit-Constructed Training Pairs, e.g., InstructAV2AV)

[Synthetic caption data (the most common form of synthesis in this ecosystem)]
· Teacher distillation is itself synthesis: captions generated by GPT-4V/Gemini all count as synthetic annotation — ShareGPT4Video-40K, the Koala-36M seed set, AVoCaDO-SFT-107K, and Script-a-Video's 500K MTSS are all examples.
· Two-stage synthesis (AVoCaDO): Gemini-2.5-Pro first generates a visual caption and an audio caption separately, then the two captions plus the original video are fed back into the same model to synthesize a temporally coherent multimodal caption — this "decompose first, then synthesize" two-stage design significantly outperforms one-shot joint generation.
· Three-way parallel synthesis (AVSCap-130K): each video is annotated three times — visual / audio / synergistic — to serve different training objectives.
· Bootstrapped synthesis: video-SALMONN 2 uses its own high-quality caption corpus for the SFT of subsequent models.
· Preference-data synthesis: Tarsier2 uses model-based sampling to automatically construct preference pairs for DPO; the GRPO rewards of AVoCaDO/AVSCap are themselves online-synthesized preference signals.
[Synthetic training samples (controlled perturbation/edit-constructed training pairs)]
· SkyCaptioner-V1's camera-motion sub-expert uses 16K "motion-axis-balanced synthetic data" to supplement long-tail camera-movement types — a controlled synthesis performed specifically to correct the annotator's own category imbalance, the clearest use case of synthetic data in this ecosystem.
· InstructAV2AV uses Qwen3-Omni to generate editing instructions + Grounded-SAM-2's mask-guided synthesis to construct "source-instruction-target" triples (InsAVE-80K), an example of a captioner being used for synthetic-data construction rather than video description.
· Movie Gen's cross-paired synthetic data (PT2V) slightly lowers ArcFace identity similarity in exchange for significantly more diverse head poses and more natural expressions.
[Methodological hazards of synthetic data] InstructAV2AV uses the same Qwen3-Omni to both generate instructions and perform five-dimensional verification scoring, with the paper itself acknowledging that "generation and acceptance sharing the same source is a methodological hazard of this pipeline"; AVSCap is both the benchmark's author and its top scorer, so the same-source overfitting risk requires cross-validation against third-party benchmarks.

### Human Involvement Level (Human Annotation, Human Quality Checking, Model Pre-Screening + Human Review)

Human involvement in the captioner ecosystem is concentrated at three points, consistently following a pattern of "a small amount of high-value human effort + large-scale model inference":
[Location one · human curation of seed data (highest cost, highest value)]
· SkyCaptioner-V1's camera-motion sub-expert: 93K high-confidence human annotations + a 15K human-annotated test set.
· ALIVE: the caption model's training data follows the classic self-built path of "MLLM-generated → manually revised caption data → two rounds of SFT," trading one-time human cost for large-scale low-cost inference; it also has human-annotated start/end timestamps for sub-motion units.
· Movie Gen: captions in the post-training stage are refined by humans on top of model output.
· Step-Video-T2V: humans "optimize captions" during the SFT stage, a fallback measure used when no dedicated hallucination-suppression post-training is applied to the captioner.
[Location two · human studies for captioner selection and evaluation]
· Panda-70M uses a 1,000-clip user study to perform greedy set-cover selection of 8 captioners; it also reports a human-to-human agreement rate of only 44.9% — this figure defines the ceiling for the entire ecosystem's caption-quality evaluation, the most important human-evaluation finding in this ecosystem.
· Movie Gen's caption-scheme comparisons rely entirely on human A/B testing: video captioning is preferred 67% of the time versus 15% for the per-frame rewriting scheme.
· AVSCapBench's 1,226 videos are human-annotated; UGC-VideoCap uses a three-stage human-in-the-loop process (annotating audio-only / visual-only / joint AV separately).
· Open-Sora Plan's threshold reasonableness is validated via 2,000 manually spot-checked samples.
[Location three · the trend of humans being replaced by LLM-judge] From 2025–2026, per-sample human review of caption quality has been almost entirely replaced by GPT-4.1 / Gemini scoring (AVoCaDO's completeness scoring, AVSCap's automated verification, AuroraCap's VDCScore), with humans retained only for benchmark construction and final A/B testing. The cost of this substitution has not yet been systematically evaluated — given a human agreement rate of only 44.9%, the discrepancy between LLM-judge and human judgment may already be masked by the fact that "humans themselves are inconsistent."

## Audio-Video Alignment

### Audio-Video Sync Detection Method (Lip Sync, Event Alignment) ⚠️

The captioner ecosystem's relationship with audio-video sync detection is "indirect but critical" — captioners do not perform sync detection directly, but they serve as the semantic acceptance check on sync quality:
[Sync-related roles taken on by captioners]
· AVSCap's third criterion, Audio-Visual Synergy, explicitly requires captions to explicitly bind audio-visual events and align them temporally — an attempt to elevate "synchronization" from a signal-level metric to a semantic-level description. AVSCapBench's Synergy dimension quantifies this exact capability: AVSCap-7B 57.70, Gemini-3-Pro 48.88, AVoCaDO 29.13, video-SALMONN-2 12.43, bare Qwen2.5-Omni base model 7.00.
· AVoCaDO's fifth dimension, Cross-modal Narrative Logic (audio and visual mutually explaining/complementing/guiding each other), is an earlier form of the same idea, but its Synergy score of only 29.13 shows that having this dimension in the checklist does not mean the model has truly learned it.
· Ovi explicitly requires captions to cover all relevant visual and auditory events while "respecting chronology," conducting multiple rounds of prompt-iteration experiments toward this end.
[Signal-level sync detection is handled by dedicated tools (upstream/downstream of the captioner)] SyncNet and Synchformer are the mainstream choices: Foley-Omni uses Synchformer both as a training feature provider and as the sync-scoring tool in the filtering stage (one model, two uses, ensuring the filtering standard is consistent with the model's learning objective); MOVA uses LSE-D/LSE-C; SkyReels-V4 uses SyncNet. These are decoupled from caption generation.
[The speaker-attribution problem in lip sync] TalkNet (InstructAV2AV) performs active-speaker detection to determine "who is speaking"; CineDance uses Qwen3-Omni + a sliding-window prompt to lift ASR-to-character binding from 67.2% to 95.4% — speaker attribution is the task on the caption side that comes closest to sync detection.
[Uncertain] No captioner work outputs sync-detection results directly as a caption field (e.g., "audio-visual offset of 0.3 seconds"); synchronization information is only implicitly embedded in the caption in the semantic form of "event binding."

### Specific Sync Detection Metrics and Thresholds (SyncNet/Synchformer/LSE/In-House, Threshold Values, e.g., MOVA's LSE-D≤9.5 and LSE-C≥4.5, SkyReels-V4's SyncNet |offset|≤3∧conf>1.5) ⚠️

[Uncertain] Captioners themselves do not set sync-metric thresholds; this field records the comparable thresholds related to captioning found in this ecosystem:
[Caption-quality thresholds (native to this ecosystem)]
· AVoCaDO: GPT-4.1 scores synthesis completeness from 1–5, with a retention threshold of ≥4; dialogue content similarity uses edit distance + dynamic programming alignment, threshold 0.6; caption length is capped at 4096 tokens (penalized beyond that by ℛ_L), with an optimal range of 2048–4096 tokens.
· Foley-Omni: Bandit's acoustic post-hoc-verification energy-gating threshold of −35 dB (used to correct visual hallucinations in Gemini annotations, i.e., objects seen on screen but with no actual sound).
· Panda-70M: the hard-negative weighting scheme for UMT retrieval-based selection (unselected candidate captions get a weight of 1.0, other in-batch negatives get 0.01).
[Sync thresholds for downstream generation models (shown for comparison, not captioner thresholds)] MOVA requires LSE-D ≤ 9.5 and LSE-C ≥ 4.5; SkyReels-V4 requires SyncNet |offset| ≤ 3 and conf > 1.5; Foley-Omni uses a Sync score ≥ 0.2, IB ≥ 0.3, AudioBox ≥ 0.6; OmniCustom uses dual SyncNet thresholds. All of these run outside the captioner.
[Ecosystem gap] No work publishes a "joint threshold table for caption quality and audio-video synchronization," i.e., there is no public protocol for "how a sample's caption should be handled (discarded/downgraded/flagged) when its synchronization falls below some threshold" — a clear methodological gap in AV data pipelines.

### Separate Handling of Temporal Sync vs. Semantic Sync (Time Alignment and Content-Semantic Matching as Two Independent Filtering Conditions)

This ecosystem shows a clear methodological evolution in separately handling "temporal synchronization" and "semantic matching":
[Three layers of evidence for separation]
(1) AVSCap's evaluation-dimension design is itself a separation: the four dimensions Visual / Speech / Music / SFX measure **semantic completeness** (was the content that should have been said, said), while Synergy separately measures **cross-modal binding and temporal alignment**. The data shows these two can severely diverge — AVoCaDO's Speech score is as high as 70.42 (semantically complete) but its Synergy score is only 29.13 (binding failure), proving that "clearly audible" does not equal "properly matched."
(2) Foley-Omni's dual-feature pathway: CLIP provides scene-semantic features, Synchformer provides temporal-sync features, injected via two independent channels. An ablation shows that removing the Z_sync sync channel drops IB_V2ST from 0.26 to 0.22, raises FD_VGG from 1.57 to 2.21 (a 41% degradation), and raises WER_V2ST from 7.59 to 12.40 — quantitatively proving that the temporal-feature channel also makes a substantive contribution to semantic consistency, i.e., the two are not fully orthogonal.
(3) MOVA's filtering design chains "cross-modal semantic-consistency checking" (GPT-OSS-120B) and "lip-sync signal detection" (LSE-D/LSE-C thresholds) as two independent conditions.
[The specific division of labor on the caption side] Semantic matching relies on the captioner and an LLM judge (whether the correct sound source was described, whether there is off-screen-sound hallucination); temporal alignment relies on signal-level detection via Synchformer/SyncNet and ASR timestamps (ElevenLabs Scribe, CineDance's sentence-level ASR).
[A cautionary counterexample] Foley-Omni's motivation for using Bandit for acoustic post-hoc verification is precisely that: audio descriptions Gemini generates based on the picture may describe sounds that do not actually exist (visual hallucination), an error that pure semantic judgment cannot self-correct, requiring signal-level verification to be introduced — showing that the semantic and temporal/signal pathways serve as mutual backstops, neither of which can be neglected.

### Audio Quality Filtering (SNR, Silence Detection and Silence-Ratio Thresholds, No-Audio-Track Removal, Off-Screen-Sound Removal, Background-Music Separation) ⚠️

Audio-quality handling in the captioner ecosystem mainly manifests as "a pre-captioning routing gate" and "post-captioning acoustic verification":
[Pre-captioning routing] UniVerse-1's offline funnel uses Whisper to determine whether a clip contains speech, as a routing gate deciding which annotation path to take; the video stage of the JavisDiT series uses FunASR to remove clips containing speech (while the audio stage explicitly performs "no filtering at all, to maximize T2A capability coverage across all three audio categories" — this "no filtering for audio, strict filtering for video" contrast reflects the team's practical judgment on the quality-quantity trade-off for the two modalities); LTX-2's data-subset filtering criterion is "contains significant and information-rich audio content," used to remove silence and uninformative audio tracks.
[SNR/audio-quality scoring] Movie Gen uses a dedicated audio-quality prediction model (outputting a 1–10 score); Foley-Omni uses an AudioBox threshold of 0.6; InstructAV2AV uses an Audiobox threshold. These all run upstream of the captioner.
[Background-music separation and source separation] Bandit (Foley-Omni, separating into speech/effects/music, structurally identical to the three-field schema); Mel-RoFormer (Unison, decoupling mixed audio into separate speech and sound-effect ground-truth latents on the training side, and performing vocal separation before WER computation on the evaluation side); SAM-Audio (InstructAV2AV, semantically separating the sound of a target entity from the mixed audio track).
[Off-screen-sound removal] Foley-Omni's −35 dB energy gate is essentially removing annotations hallucinating "present in the picture but absent from the audio track"; the reverse operation can identify off-screen sound (present in the audio track but absent from the picture). Movie Gen's evaluation dimension "on-screen sound correctness" is the only metric that explicitly quantifies this issue, with SFT improving +31.0±16.0 relative to PT.
[Uncertain] No captioner work publicly discloses the silence-ratio threshold or SNR threshold of its training data.

### Classification and Separate Handling Strategy for Speech/SFX/Music

The classification and separate handling of the three audio categories — speech / sound effects / music — is the most mature design pattern in the AV captioner ecosystem:
[Two-model division of labor (most common)] MOVA: Qwen3-Omni-Instruct handles speech transcription, Qwen3-Omni-Captioner handles non-speech sound and music description; the paper emphasizes this division "comprehensively covers linguistic content and acoustic characteristics, reducing information loss." UniTalking: Whisper-V3 transcription + Qwen3-Omni-Captioner describing the acoustic environment + Qwen3-Omni for fusion. SkyReels-V4: Qwen3-Omni uniformly generates audio captions, while Whisper transcribes speech and singing.
[Fixed three-field schema] Foley-Omni's speech / effects / music fields are structurally identical to Bandit's separated outputs; AVSCap's Acoustic Completeness criterion likewise splits into Speech / SFX / Music.
[Four-model collaboration (Movie Gen, the most fine-grained)] An audio-quality prediction model (1–10 scale) + an AED model (judging voice/singing presence and music posterior probability) + a general audio-caption model (freeform sound description) + a music-caption model (supplying mood and genre). The paper specifically notes that the music-caption model is trained mainly on music samples and tends to hallucinate when there is no music, so it **retains both the AED's music-probability signal and the music-caption signal simultaneously**; empirically this redundant combination gives the best controllability — the most valuable engineering lesson in the entire ecosystem regarding "audio-type hallucination."
[Single-model-does-everything (an aggressive route)] UniVerse-1 uses a single Qwen2.5-Omni to output all three streams in parallel in one pass; Ovi uses the same MLLM to process both audio-video corpora and audio-only corpora; CineDance uses Qwen3-Omni-30B-A3B to simultaneously handle sentence-level ASR, shot-level audio prompts (music/ambient sound/sound effects), and character timbre description as three sub-tasks.
[Current state of capability (quantified by AVSCapBench)] The descriptive-ability gap across the three audio categories is large and highly consistent: Speech (40–80 points) ≫ Music (4–40 points) > SFX (7–32 points). SFX is the weakest link industry-wide, with the best open-source model AVSCap-7B scoring only 30.82, and Gemini-3-Flash 32.34 — this directly limits the training-data ceiling for Foley and ambient-sound generation models, making it the most urgent direction to break through in the AV captioner ecosystem.

## Training Integration

### Multi-Stage Training Curriculum and Data Curriculum Scheduling (Basis for Stage Division: Resolution/Duration/Quality Score/Modality; Low-Res→High-Res, Image→Video, Short→Long)

[The captioner's own training curriculum]
· AuroraCap: three-stage training, cumulatively over 20 million image/video-text pairs (a classic image-to-video curriculum).
· Tarsier2: pretraining (40M video-text pairs) → fine-grained temporal-alignment SFT → model-based-sampling preference-pair construction + DPO, three stages.
· AVoCaDO / AVSCap: SFT → GRPO, two stages. AVSCap explicitly concludes that "the gains from RL exceed those from scaling up SFT data" — a key conclusion for allocating training resources in caption-model development.
· video-SALMONN 2: SFT → MrDPO (multi-round DPO, periodically resetting the reference policy with a newly trained lightweight adapter to avoid a stale reference).
· Omni-Captioner (Alibaba): a two-stage curriculum, audio → audio-visual (learning to hear first, then to see+hear) — nicely complementing the visual side's "image→video" curriculum, a curriculum design unique to omni captioners.
· SkyCaptioner-V1: Qwen2.5-VL-72B general description + three sub-experts → fusion → distillation into a 7B model, a "multi-teacher parallel → single-student distillation" pattern rather than a sequential curriculum.
[The captioner's role in the generation model's training curriculum (tiered captioning)]
· Open-Sora 2.0: massive data at the 256px stage is captioned with LLaVA-Video, while a curated 5M subset at the 768px stage is re-captioned with Qwen 2.5 Max — a captioning-quality pyramid strictly matched to the data pyramid.
· Movie Gen: 70% of captions from the 8B model and 30% from the 70B model is another implementation of the same trade-off; captions in the post-training stage are refined by humans.
· Open-Sora Plan v1.3: stage two uses unfiltered Panda70M (converging at roughly 100K steps), stage three switches to filtered data (only 30K steps, with the learning rate lowered to 1e-5) to achieve a quality jump — indirect evidence for "large volumes of coarse data to learn the distribution, a small amount of refined data to raise quality."
· Motif-Video 2B: its three caption variants are sampled with probabilities 0.5/0.3/0.2; stages 1–3 use a sentence-level-embedding text encoder, switching to T5Gemma2 starting at stage 4 (borrowing PixArt-α's "class-conditioning to text-conditioning" curriculum idea).
[Ecosystem conclusion] Upgrading caption quality as training stages progress is a stable practice for 2025–2026, and the cost structure of "cheap models label the base layer, expensive models label the top layer" has independently converged across multiple teams.

### Data Mixture Changes Across Training Stages (Pretraining/Annealing/High-Quality SFT Subsets) ⚠️

[Data mixture in captioner training]
· AVoCaDO-SFT-107K's six-source mix is itself an explicit mixture design: FineVideo 29K (27%) > TikTok-10M 24K (22%) > Shot2Story 20K (19%) > ShortVideo 18K (17%) > YouTube-Commons 11K (10%) > CinePile 5K (5%), with UGC as the main body and film as a small high-quality supplement.
· AVSCap-130K = 40K videos × 3 annotations (visual / audio / synergistic), with the three annotation types mixed 1:1:1 rather than skewed toward the fused type — showing that unimodal-anchored annotation is equally important for training.
· SkyCaptioner-V1: curated from 10 million down to 2 million concept-balanced samples (20% retention), with the mixture goal being concept balance rather than source balance.
[Caption mixture changes across generation-model training stages]
· Motif-Video 2B: sampling probabilities of 0.5/0.3/0.2 for its three caption variants (the paper candidly admits this is "a pragmatic recipe choice rather than an isolation-validated optimality claim").
· UniTalking: outputs from both the "annotate separately then concatenate" path and the "unified model fusion annotation" path are randomly sampled during training — leaving the choice of annotation strategy to random sampling so the model adapts to both text distributions simultaneously.
· SkyReels-V4: short/long/structured descriptions coexist for mixed training.
· Movie Gen: the 70/30 mix of 8B/70B captions runs throughout the whole training set, not as a staged change.
· NAVA (in the JAVG batch): the modality mixture ratio is scheduled from 3:1 to 1:2, with a 160K SFT subset, but with no controlled experiment validating the benefit.
[Ecosystem gap] Almost no work reports the ablation-optimal point for "the mixture ratio of different caption styles/lengths"; Motif's candid admission (0.5/0.3/0.2, unvalidated) represents the industry's actual state of affairs. [uncertain]

### Post-Training Data (SFT Curated-Set Scale and Selection Criteria, Preference-Pair Count and Annotation Method, Reward-Model Training Data) ⚠️

[Captioner post-training data (the core competitive focus of this ecosystem for 2025–2026)]
· AVoCaDO's GRPO: the reward comprises three terms — ℛ_C (five-dimensional checklist coverage, judged by GPT-4.1), ℛ_D (dialogue F1, speaker accuracy + content edit-distance DP-aligned similarity, threshold 0.6), and ℛ_L (length regularization, penalizing >4096 tokens). Preference signals are generated online, with no fixed-scale preference-pair dataset.
· AVSCap's GRPO: hybrid reward = length control + speech preservation + audio-visual consistency. The paper's key conclusion "RL gains > gains from scaling up SFT data" is direct evidence for prioritizing post-training investment over expanding annotation volume.
· Tarsier2's DPO: constructs preference data automatically via model-based sampling (no human preference-pair annotation needed); the scale is undisclosed [uncertain].
· video-SALMONN 2's MrDPO: multi-round DPO, periodically resetting the reference policy with a newly trained lightweight adapter; the caption-quality objective simultaneously rewards completeness and factual accuracy, reducing the 7B model's caption error rate by 28% relative to baseline.
· video-SALMONN-o1's pDPO: process-level DPO, using contrastive step selection for step-level reward, paired with RivaBench (4,000+ expert-annotated QA pairs covering stand-up comedy, academic lectures, and synthetic-video detection).
· Tencent Hunyuan 1.5's caption model: uses OPA-DPO (a preference-optimization method targeting multimodal hallucination) for RL post-training to suppress hallucination — the only case where a generation-side team publicly discloses a captioner post-training method.
[SFT curated-set selection criteria] AVoCaDO retains only samples scoring ≥4/5 from GPT-4.1; SkyCaptioner curates 2 million from 10 million; ALIVE's caption data is MLLM-generated, then human-revised, then used for two rounds of SFT.
[Ecosystem trend] Before 2025, captioner post-training was almost SFT-only; starting in the second half of 2025, DPO/GRPO became standard, and reward design (especially dialogue accuracy and cross-modal consistency) replaced data volume as the main axis of competition.

### Data Processing Infrastructure and Throughput (NeMo Curator/Data-Juicer/In-House, GPU Speedup Ratios, Processing Scale, Cost) ⚠️

[Partly uncertain] This dimension is sparsely disclosed, but throughput considerations profoundly shape captioner selection:
[Concrete compute figures]
· SkyCaptioner-V1: 32 A800 GPUs, global batch size 512, roughly 2 million training samples.
· Panda-70M's UMT selection model: 8×A100-80G, 12 frames at 224×224, AdamW lr 2e-5, batch 32, 10 epochs.
· Open-Sora Plan v1.3's prompt refiner: LLaMA-3.1-8B-Instruct LoRA (rank 64, batch 32, 1 epoch), trained in 30 minutes on a single H100.
· MOVA: captioning runs on a mix of NVIDIA GPUs and Huawei Ascend NPUs — a rare public record of domestic chips participating in large-scale captioning.
· Motif-Video 2B: PaddleOCR-VL served via vLLM; CineDance's inference framework credits vLLM. vLLM is the de facto standard for batch captioning inference.
[Direct evidence of throughput determining model selection (the most valuable engineering record in this ecosystem)]
· Open-Sora's official documentation: "GPT-4V is better, but 20 seconds/sample is too slow for us" — hence the switch to PLLaVA-13B. This is the most candid public trade-off record on captioning cost among open-source projects.
· Allegro's key reason for choosing Aria is its official claim of generating a caption for a 256-frame video within 10 seconds.
· The ecosystem-wide convergence of parameter counts to 7B (ShareCaptioner, Tarsier2, SkyCaptioner, AVoCaDO, AVSCap, MiMo-VL) is itself a product of throughput constraints; likewise for placing 120B-class models (GPT-OSS-120B) only in the fusion/adjudication stage rather than the per-sample perception stage.
· UniVerse-1's online-annotation architecture forces the use of lightweight models (Qwen2.5-Omni + Whisper), unable to bear the per-sample inference cost of a 120B-class model — a coupling between "annotation timing" and "annotation model capacity."
· MAGI-1 distills a large MLLM's enhanced prompts into a roughly 7B small model (about 2 million samples) to reduce inference latency, with human evaluation showing comparable quality while latency and compute overhead drop substantially.
[Cost] No work publicly discloses the dollar cost or GPU-hour count of captioning [uncertain]. Apollo/Klear's call to Gemini-2.5-Pro over 81 million samples is the largest known commercial-API captioning investment, but the dollar amount is undisclosed.
[Dedicated frameworks] No public record was found of the captioner ecosystem using data-engineering frameworks such as NeMo Curator / Data-Juicer; the captioning step is generally an in-house script + vLLM batch inference.

## Results Comparison

### Quantified Impact of Data Strategy Ablations (Distinguishing: Filter-Strictness Ablation / Caption-Density-Style Ablation / Data-Mixture Ablation, and Corresponding Evaluation Metrics) ⚠️

This field consolidates the ecosystem's quantitative evidence for "how caption quality/density/style affects downstream generation" — the ultimate test of a captioner's value:

[Strongest evidence · Movie Gen's caption-form A/B test (human net-preference win% − lose%)] A native video captioner (LLaMa3-Video 8B) vs. a frame-sampling-and-rewriting scheme (image-captioning the first/middle/last frames, then rewriting and merging with LLaMa):
· Caption quality itself: in human A/B testing, the video caption is preferred 67% of the time versus only 15% for the frame-rewriting scheme;
· Impact on the generation model: overall prompt alignment +10.8%, with the vast majority of the gain coming from motion alignment +10.7%, rising to +16.1% on high-motion prompts.
Conclusion: a native video captioner can accurately describe finer-grained motion detail, providing a stronger supervisory signal for training — the only quantitative answer to "why a video captioner must be used rather than stitched-together image captions."

[The benefit of structured captions · Koala-36M] Introducing structured "metric conditions" captions raises VBench semantic score from 0.4504 to 0.5915 (+14.1 percentage points); training on Koala-all versus Panda-70M improves VBench subject consistency by +1.1% and background consistency by +2.4% (attributed to more accurate shot splitting). The VTSS model-based filter outperforms manual multi-metric hard-threshold combinations.

[Complete quantification of distillation gains · Script-a-Video (the most complete caption-distillation ablation in this ecosystem)] Native Qwen3-Omni → zero-shot MTSS prompting → MTSS fine-tuning (teacher: Gemini-2.5-Pro):
· video-SALMONN-2 overall error rate: 0.5853 → 0.5156 → 0.3913 (the teacher's native score is 0.3959, i.e., the fine-tuned open-source model already slightly outperforms the teacher);
· UGC-VideoCap composite score: 62.80 → 71.54 → 85.11 (teacher: 93.97);
· Daily-Omni: 0.1806 → 0.4117 → 0.5945 (teacher: 0.6825);
· WorldSense: 0.1569 → 0.3106 → 0.3875 (teacher: 0.4332).
Key finding: structured MTSS prompting benefits all tested models (including non-fine-tuned ones) — a zero-cost gain.

[The necessity of captioner diversity · Panda-70M] The single best captioner covers only 30.8% of samples; all 31 combined cover 84.7%; the greedily selected 8 cover 76.8%; the fine-tuned UMT selection model achieves R@1 of 35.90% (versus only 21.82% for the pretrained UMT); the human-to-human caption-preference agreement rate is only 44.9%.

[Horizontal quantification of audio-video captioning · AVSCapBench] See the full table in audio_category_distribution. Core conclusion: AVSCap-7B overall 60.44 ≈ Gemini-3-Pro 60.97, with Synergy surpassing it (57.70 vs. 48.88), while Speech trails by 10 points; bare Qwen2.5-Omni scores only 21.53.

[Ablation of the caption model's own capability · Tarsier2] Three upgrades — pretraining data 11M→40M + fine-grained temporal-alignment SFT + DPO — result in DREAM-1K F1 exceeding GPT-4o by 2.8% and Gemini-1.5-Pro by 5.8%, human evaluation +8.6% / +24.9%, and the first DREAM-1K overall recall to break 40%. AVSCap's conclusion "RL gains > gains from scaling SFT data" complements this.

[Qualitative but important conclusions]
· CogVideoX: argues that "an innovative video captioning model significantly improves generation quality and semantic alignment," with Appendix H showing side-by-side examples of Panda-70M short captions versus CogVLM2-Caption long captions to demonstrate density differences, but without giving a net A/B preference score [quantification missing].
· Goku: expanding short GenEval prompts with GPT-4o raises the score to 0.76, indirectly proving that model performance is highly dependent on long prompts consistent with the dense captions used during training (not organized as a formal caption ablation).
· Open-Sora Plan: explicitly demonstrates that a distribution mismatch between training captions (dense, long text) and user prompts (<10 words) harms generation quality, and builds a prompt refiner accordingly, but with no on/off comparison metric.
· Open-Sora 2.0: switches the captioning model from LLaVA-Video to Qwen2.5-Max in the high-resolution stage, reasoning it is "more accurate, with better semantic alignment," with no controlled experiment.

[The single biggest gap across the ecosystem] The vast majority of generation-model technical reports (HunyuanVideo, Allegro, LTX-2, LongCat-Video, MAGI-1, Mochi, Apollo, Foley-Omni, InstructAV2AV, CineDance) explicitly lack a caption-density/style ablation. Movie Gen, Koala-36M, and Script-a-Video are the only three works providing solid quantitative evidence — meaning the industry's choice of "how long should a caption be, how structured should it be" is largely driven by intuition and cost rather than empirical evidence. [uncertain]

### Evidence for Quality vs. Quantity (Cases Where Small-but-Refined Data Outperforms Large-but-Messy Data)

[Evidence for quality beating quantity within the captioner ecosystem]
· ShareGPT4Video is the most classic example: SFT on just 40K GPT-4V dense captions produced ShareCaptioner-Video, which was then able to annotate 4.8 million videos — an extremely small high-quality seed set unlocking a hundredfold-larger output.
· AVSCap's core thesis: "the gains from RL exceed those from scaling up SFT data," i.e., at a scale of 130K samples, investing in reward design pays off more than continuing to expand annotation volume.
· Script-a-Video: after fine-tuning on 500K Gemini-2.5-Pro-curated samples, an 8B/30B-class open-source model's overall error rate on video-SALMONN-2 (0.3913) already slightly outperforms the teacher Gemini-2.5-Pro's native output (0.3959).
· SkyCaptioner-V1: a 7B model trained on just 2 million concept-balanced samples achieves cinematography-field accuracy (average 76.3%, shot type 93.7%) significantly exceeding general-purpose models 10× its size, such as Qwen2.5-VL-72B — direct evidence that "a small model + matched data > a large model + generic data."
[Evidence for quality beating quantity at the downstream-dataset level]
· Koala-36M (36 million clips) vs. Panda-70M (70 million clips): only half the scale, yet the highest total VBench score, with subject consistency +1.1% and background consistency +2.4%. The core difference lies in caption length (202.3 words vs. 13.2 words) and splitting accuracy — the most direct "small-but-refined beats large-but-messy" case in this ecosystem.
· CineDance-1M's artifact non-compliance rate of 2.8% versus Koala-36M's 37.4% (a 13.4× improvement) represents a further tightening by a newer generation.
· JavisDiT++'s framing is worth noting — it is not simply "quality over quantity," but "ensuring good data quality is the prerequisite for increasing sample count to improve training outcomes" — i.e., simply scaling up when quality is substandard is ineffective or even harmful.
· Movie Gen audio SFT vs. PT: overall +41.7±15.3, professionalism +43.0±14.7, with the paper commenting that this "highlights the importance of high-quality data curation in the fine-tuning stage."
[A cautionary counter-case] CogVideoX reports a two-sided effect: removing subtitle watermarks slightly improves visual quality but slightly degrades semantic capability — over-aggressive cleaning can lose semantic diversity, a boundary condition on the "quality-first" principle.

### Alignment Between Training Data Domain Distribution and Benchmark Taxonomy (e.g., VABench's Seven Major Categories)

[Taxonomies specific to caption benchmarks]
· VDC (paired with AuroraCap): structured captions divided into five fields — camera / short / background / main object / detailed — paired with VDCScore (a divide-and-conquer conversion into multiple short QA pairs judged by an LLM). These five fields effectively define the caption schema used by much subsequent work.
· AVSCapBench (2026-07, the newest and most fine-grained): 1,226 human-annotated videos (30–120 seconds, from YouTube / TikTok / Video-MME), measuring fine-grained event recall along five dimensions — Visual / Speech / Music / SFX / Synergy — designed to prevent "gaming the score via a single modality." These five dimensions correspond closely to the training-data domain needs of AV generation models.
· Omni-Cloze (Alibaba, 2026-03): 2,000 clips / 70,000 fine-grained cloze questions / 9 major domains / 47 sub-categories, using cloze questions to avoid LLM-judge noise. The most complete publicly available taxonomy.
· UGC-VideoCap (2025-07): 1,000 TikTok videos + 4,000 QA pairs, with a three-stage human-in-the-loop process annotating audio-only / visual-only / joint AV semantics separately — categorized by "modality availability" rather than content topic.
· OmniCap-IF (2026-06): the first audio-video caption instruction-following benchmark, with 50 constraint categories covering pure-vision/pure-audio/audio-video, including Temporal Grounding, with dual-dimension scoring (format correctness + content correctness). This dimension matters greatly for production pipelines — a captioner must be able to output according to your schema.
· DREAM-1K (paired with Tarsier): centers on event recall for dynamic actions, with Tarsier2 the first model to break 40% overall recall.
· VidCapBench (2025-02): explicitly oriented toward evaluating video captions for controllable T2V, one of the few benchmarks to explicitly tie "caption quality" to "generation controllability."
· The video-SALMONN 2 test set: oriented toward error rate (missing + hallucination), complementing the recall-oriented benchmarks above.
[Alignment with the generation-side evaluation system]
· The dimensions of caption benchmarks (camera / background / main object / motion) correspond clearly to VBench's evaluation dimensions (subject consistency, background consistency, motion smoothness, semantic score); Koala-36M's structured captions raising the VBench semantic score by 14.1 percentage points is a direct manifestation of this correspondence.
· AVSCapBench's Speech / Music / SFX split is structurally identical to the audio taxonomies used by AV evaluation benchmarks such as VABench and Foley-Omni's three-field schema — indicating that "captioning schema, evaluation taxonomy, and generation-capability dimensions" are converging toward a single shared ontology.
[Methodological caution] AVSCap is both the author of AVSCapBench and its top scorer (60.44), posing a same-source overfitting risk; the paper has been out for only two weeks (2026-07-14) with no third-party reproduction, so selection decisions should cross-validate against third-party benchmarks such as UGC-VideoCap and Omni-Cloze. Panda-70M's reported human agreement rate of only 44.9% sets a ceiling on the reliability of all caption benchmarks.

## Uncertain Fields

The research findings for the following fields are partly uncertain (⚠️-marked sources):

- openness
- data_scale
- provenance_licensing
- duration_distribution
- resolution_aspect_distribution
- domain_distribution
- audio_category_distribution
- narrative_structure_distribution
- language_accent_distribution
- funnel_retention_rate
- deduplication
- safety_filtering
- caption_model
- geometric_structured_annotation
- av_sync_detection
- sync_metric_and_threshold
- audio_quality_filtering
- stage_data_mixture
- post_training_data
- data_infra_throughput
- data_ablation
