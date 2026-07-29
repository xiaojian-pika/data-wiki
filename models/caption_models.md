# 视频 Caption 模型生态（Video Captioner Ecosystem）——合并调研条目，覆盖 ShareGPT4Video / ShareCaptioner-Video、Tarsier & Tarsier2 系列、CogVLM2-Caption、SkyCaptioner-V1、AVoCaDO、AVSCap、video-SALMONN 2、Qwen3-Omni-Captioner / Qwen3.5-Omni、AuroraCap、Panda-70M 多教师 captioner、LLaVA-Video / PLLaVA、Aria、Tag2Text 等，并归纳它们在视频生成数据 pipeline 中的实际应用方式。本条目不是单一模型/数据集，而是「打标器」这一 pipeline 组件的横向生态图谱。

> 主题: 视频生成模型（含音视频同时生成）的数据处理：数据清洗流程、数据分布、打标方式

[← 返回首页](../index.md)

**目录**: [基本信息](#基本信息) · [数据规模与分布](#数据规模与分布) · [清洗流程](#清洗流程) · [打标方式](#打标方式) · [音视频对齐](#音视频对齐) · [训练配合](#训练配合) · [效果对比](#效果对比)

## 基本信息

### 名称

视频 Caption 模型生态（Video Captioner Ecosystem）——合并调研条目，覆盖 ShareGPT4Video / ShareCaptioner-Video、Tarsier & Tarsier2 系列、CogVLM2-Caption、SkyCaptioner-V1、AVoCaDO、AVSCap、video-SALMONN 2、Qwen3-Omni-Captioner / Qwen3.5-Omni、AuroraCap、Panda-70M 多教师 captioner、LLaVA-Video / PLLaVA、Aria、Tag2Text 等，并归纳它们在视频生成数据 pipeline 中的实际应用方式。本条目不是单一模型/数据集，而是「打标器」这一 pipeline 组件的横向生态图谱。

### 发布机构/公司

多家，按阵营归类：
【学术/开源社区】ShareGPT4Video & ShareCaptioner-Video（中科大 + 上海 AI Lab + 香港中文大学，Lin Chen、Xilin Wei、Jinsong Li 等 15 位作者，NeurIPS 2024 Datasets & Benchmarks Track）；AuroraCap（UC Santa Barbara / 多校合作，ICLR 2025）；Panda-70M（Snap Research + UC Merced，CVPR 2024）。
【中国大厂】字节跳动 Tarsier / Tarsier2 / Tarsier2-Recap（bytedance/tarsier），并与清华电子系合作 video-SALMONN 2 / video-SALMONN-o1；智谱 AI（zai-org / 原 THUDM）CogVLM2-Caption；阿里通义 Qwen 系列（Qwen2-VL / Qwen2.5-VL / Qwen3-VL / Qwen2.5-Omni / Qwen3-Omni-Captioner / Qwen3.5-Omni，配套 Omni-Captioner + Omni-Detective + Omni-Cloze，ICLR 2026）；昆仑万维 SkyWork SkyCaptioner-V1；快手可灵团队 AVoCaDO（联合中科院自动化所、国科大、北大、南大）与 Koala-36M 的 LLaVA 微调 captioner；南京大学 NJU-LINK + 快手 Kling AVSCap；小米 MiMo-VL（被 MOVA 用作视觉标注器）。
【海外大厂/闭源】OpenAI（Sora 的 highly descriptive captioner、Whisper ASR）；Meta（Movie Gen 的 LLaMa3-Video 8B/70B）；Google（Gemini 系列作为通用打标器，Veo 3 使用「多个 Gemini 模型」）；Lightricks（LTX-Video / LTX-2 的内部自研音视频 captioner）；Rhymes AI（Aria，被 Allegro 用作细粒度打标器）。

### 发布时间（技术报告/论文/开源时间）

生态演进时间线（以公开发布/arXiv 提交为准）：
· 2023：Tag2Text（标签式轻量打标器，被 Allegro 用作粗筛）。
· 2024-02：Panda-70M（arXiv 2402.19479，CVPR 2024），首个系统化的「多教师 + 检索择优」打标范式。
· 2024-06-06：ShareGPT4Video / ShareCaptioner-Video（arXiv 2406.04325），提出 DiffSW 差分滑窗打标。
· 2024-08：CogVideoX（arXiv 2408.06072）披露 Panda-70M→CogVLM→GPT-4→LLaMA2 的四段式打标链；2024-09-19 开源 CogVLM2-Caption 权重。
· 2024-10：AuroraCap + VDC benchmark（arXiv 2410.03051）；Koala-36M（arXiv 2410.08260）。
· 2024-10：Movie Gen 技术报告披露 LLaMa3-Video 8B/70B 打标方案。
· 2025-01-14：Tarsier2-7B（arXiv 2501.07888）；配套 Tarsier2-Recap-585K 数据集。
· 2025-02：video-SALMONN-o1（arXiv 2502.11775）。
· 2025-04：SkyCaptioner-V1（随 SkyReels-V2 开源）。
· 2025-06-18：video-SALMONN 2（arXiv 2506.15220，3B/7B/72B）。
· 2025-09：Qwen3-Omni-Captioner（音频 captioner）开源；2025-10 Omni-Detective / Omni-Captioner 开源。
· 2025-10-12：AVoCaDO（arXiv 2510.10395），首个成体系的音视频联合 captioner。
· 2026-01：LTX-2（arXiv 2601.03233）披露自研音视频联合 captioner。
· 2026-03：Qwen3.5-Omni（arXiv 2604.15804）把「剧本级结构化音视频 caption」列为一等能力，Omni-Cloze benchmark 发布。
· 2026-06：OmniCap-IF（arXiv 2606.08572），首个音视频 caption 指令跟随基准。
· 2026-07-14：AVSCap + AVSCapBench（arXiv 2607.12820），最新的音视频联合 caption 模型与专用基准。

### 类型（模型/数据集/工具链/评测基准）

模型 + 工具链 + 评测基准的复合生态。三层结构：
(1) 打标模型（captioner）本体——通用 VLM（Qwen-VL / InternVL / LLaVA 系）、专用 captioner（ShareCaptioner-Video、Tarsier2、CogVLM2-Caption、SkyCaptioner-V1、AuroraCap、AVoCaDO、AVSCap）、全模态 captioner（Qwen3-Omni-Captioner、video-SALMONN 2、Qwen3.5-Omni）；
(2) 由打标器产出的数据集（ShareGPT4Video-40K / 4.8M、Tarsier2-Recap-585K、Panda-70M、Koala-36M、AVoCaDO-SFT-107K、AVSCap-130K）——打标模型与数据集互为因果，是本生态的核心特征；
(3) 评测基准（DREAM-1K、VDC/VDCScore、VidCapBench、AVSCapBench、UGC-VideoCap、Omni-Cloze、OmniCap-IF、video-SALMONN 2 testset）。

### 开源程度（权重/代码/数据/pipeline各自是否开源） ⚠️

开源度呈明显的「学术全开 / 大厂半开 / 生成侧闭源」三档分化：
【全开（权重+代码+数据）】ShareGPT4Video：40K GPT-4V 密集 caption 与 4.8M ShareCaptioner-Video 标注全部公开，论文 CC BY 4.0，ShareCaptioner-Video 权重在 HF（Lin-Chen/ShareCaptioner-Video，基座 InternLM-XComposer2-4KHD）。Tarsier 系列：bytedance/tarsier 代码 + omni-research/Tarsier2-Recap-7b 权重 + Tarsier2-Recap-585K 数据全开，是目前被下游生成模型复用最多的开源 captioner。CogVLM2-Caption：权重开源（zai-org/cogvlm2-llama3-caption），是 CogVideoX 数据 pipeline 中唯一可复现的一环。SkyCaptioner-V1：权重 + 训练细节（Qwen2.5-VL-7B 基座、32×A800、200 万条概念均衡数据）全披露。AVoCaDO：Apache-2.0 权重 + 代码 + 项目页，但 AVoCaDO-SFT-107K 未单独发布。video-SALMONN 2：Apache-2.0，代码/权重/testset 全开。AuroraCap + VDC 基准开源。
【半开】AVSCap：代码 + AVSCapBench 已开放，AVSCap-130K 训练集 README 明示「will release as soon as possible」，截至调研时未发布；AVSCap-7B 权重可用性存疑（GitHub 同时出现 HF 链接与「待发布」表述）[不确定]。Qwen3-Omni-Captioner（音频版）开源，但 Qwen3.5-Omni 的音视频 caption 能力仅 API 可用，未作为独立打标工具开源。
【闭源】生成侧团队自研 captioner 几乎全部不开源：OpenAI Sora 的 highly descriptive captioner、Meta 的 LLaMa3-Video captioning 微调版、Google 用于 Veo 3 的 Gemini 打标变体、Lightricks LTX-2 的音视频 captioner、腾讯混元的三个自研 caption 模型、阶跃星辰 Step-Video 的 in-house VLM、字节 Seedance 1.5/2.0 的字幕系统、快手 Kling 3.0 Omni 的视频描述增强模块——均只披露「用了什么」，不披露参数量、基座与权重。
【一个反常识的结构性事实】打标器的开源度显著高于生成模型的数据 pipeline 开源度：多数闭源生成模型的技术报告愿意点名自己用了 Tarsier2 / Qwen3-Omni / LLaVA-Video，因为打标器被视为「工具」而非「壁垒」；真正被视为壁垒的是打标 prompt 原文、字段 schema 与阈值表，这部分几乎全行业不公开。

### 是否支持音视频同时生成，以及实现方式（原生联合/级联/MoE融合）

captioner 本身不做生成，此字段在本条目按「是否支持音视频双模态输入并输出联合描述」重新诠释，这是 2025–2026 年该生态最关键的能力分水岭：
【纯视觉 captioner（不听声音）】ShareCaptioner-Video、Tarsier / Tarsier2、CogVLM2-Caption、SkyCaptioner-V1、AuroraCap、LLaVA-Video、PLLaVA、Aria、Qwen2.5-VL / Qwen3-VL、Panda-70M 的全部 31 个候选打标器（Panda-70M 论文明确说明 Video-LLaMA 分支「仅用视觉分支、音频分支明确关闭」）。此档占据 2024–2025 上半年的绝对主流。
【级联式音视频标注（多模型分工后由 LLM 融合）】主流工程做法，代表：MOVA = MiMo-VL-7B-RL（视觉）+ Qwen3-Omni-Instruct（ASR）+ Qwen3-Omni-Captioner（非语音）+ GPT-OSS-120B（融合与一致性校验）；Movie Gen 音频侧 = 音频质量预测 + AED + 通用音频 caption + 音乐 caption 四模型协同；UniTalking = Qwen3-VL + Whisper-V3 + Qwen3-Omni-Captioner + Qwen3-Omni；Kling-Foley = 音频分类 + 音频理解大模型 + LLM 融合三段式。
【原生联合音视频 captioner（单模型同时看+听）】2025Q4 起成型的新范式：AVoCaDO（Qwen2.5-Omni-7B 基座，~9B 全栈）、AVSCap-7B（同基座）、video-SALMONN 2（LLaVA-OneVision + 音频 LoRA，3B/7B/72B）、Qwen3-Omni / Qwen3.5-Omni、以及 Lightricks 为 LTX-2 自研的未具名系统。UniVerse-1 更极端，用单个 Qwen2.5-Omni 一次性并列输出语音内容/视频 caption/环境音 caption 三路。
【关键实证】裸 Qwen2.5-Omni 的零样本打标能力很差（AVSCapBench overall 仅 21.53、Speech 13.92），必须经 caption 专项 SFT+RL 才能当打标器用（AVoCaDO 49.31、AVSCap 60.44）——「有 omni 基座 ≠ 能当 omni 打标器」是本生态最重要的工程教训。

### 调研信息来源列表（论文/技术报告/官方文档/新闻的URL，标注每条来源的性质：官方一手/同团队旁证/第三方报道）

- https://arxiv.org/abs/2406.04325 — ShareGPT4Video 论文（官方一手，NeurIPS 2024 D&B Track，CC BY 4.0；40K GPT4V 密集 caption + 4.8M ShareCaptioner-Video 标注 + DiffSW 方法）
- https://huggingface.co/Lin-Chen/ShareCaptioner-Video — ShareCaptioner-Video 模型卡（官方一手，确认基座为 InternLM-XComposer2-4KHD、支持流式差分滑窗与 clip summarization）
- https://arxiv.org/abs/2501.07888 — Tarsier2 论文（官方一手；7B、预训练 11M→40M video-text pairs、细粒度时序对齐 SFT、model-based sampling + DPO；DREAM-1K F1 超 GPT-4o 2.8%、超 Gemini-1.5-Pro 5.8%；人评 +8.6% / +24.9%；15 个公开基准 SOTA）
- https://github.com/bytedance/tarsier — Tarsier 官方仓库（官方一手；Tarsier2-Recap-7b 权重与 Tarsier2-Recap-585K 数据集）
- https://huggingface.co/omni-research/Tarsier2-Recap-7b — Tarsier2-Recap 模型卡（官方一手；585K clip 来自 VATEX/TGIF/LSMDC 等公开数据集，由 Tarsier2-7B 标注；上游为 1M 视频的 recaption）
- https://github.com/zai-org/CogVideo/blob/main/tools/caption/README.md — CogVideoX 打标工具文档（官方一手；CogVLM2-Caption 用途与 video→caption→video 闭环）
- https://arxiv.org/pdf/2408.06072 — CogVideoX 论文（官方一手；Panda-70M 短 caption → CogVLM 逐帧稠密 caption → GPT-4 摘要 → 5 万条数据微调 LLaMA2 摘要模型 → 蒸馏为 CogVLM2-Caption 的四段式打标链，附录 G 给出完整 prompt）
- https://arxiv.org/abs/2410.03051 — AuroraCap + VDC benchmark（官方一手；token merging 降视觉 token、2000 万图/视频-文本对三阶段训练、VDC 千余条结构化 caption 含 camera/short/background/main object/detailed 五字段、VDCScore 分治式 LLM 评测；Flickr30k CIDEr 88.9 > GPT-4V 55.3 > Gemini-1.5-Pro 82.2）
- https://arxiv.org/abs/2410.08260 — Koala-36M（官方一手；36M clip、均长 13.75s、720p、caption 均长 202 词、GPT-4V 生成种子 caption 微调 LLaVA、六类结构化信息、VTSS 训练适用性评分）
- https://openaccess.thecvf.com/content/CVPR2024/papers/Chen_Panda-70M_Captioning_70M_Videos_with_Multiple_Cross-Modality_Teachers_CVPR_2024_paper.pdf — Panda-70M（官方一手；31 个候选打标器 = 6 基座 × 权重/模态变体、贪心集合覆盖选出 8 个覆盖 76.8%、UMT-large 细粒度检索择优、人类一致率仅 44.9%）
- https://arxiv.org/abs/2510.10395 — AVoCaDO（官方一手；Qwen2.5-Omni-7B 基座、AVoCaDO-SFT-107K 六源构成、五维 keypoint checklist、GRPO 三项 reward 含对白编辑距离 DP 对齐阈值 0.6）
- https://github.com/AVoCaDO-Captioner/AVoCaDO — AVoCaDO 官方仓库与 Apache-2.0 权重（官方一手）
- https://arxiv.org/abs/2607.12820 — AVSCap + AVSCapBench（官方一手，2026-07-14；三准则 Acoustic/Visual Completeness + AV Synergy、AVSCap-130K = 4 万视频 × 3 份标注、SFT→GRPO、AVSCapBench 1226 视频细粒度 event recall 横向对比表）
- https://github.com/NJU-LINK/AVSCap — AVSCap 官方仓库（官方一手；训练集尚未释放）
- https://arxiv.org/pdf/2506.15220 — video-SALMONN 2（官方一手，清华电子系 + 字节；LLaVA-OneVision + 音频 LoRA、3B/7B/72B、MrDPO 多轮 DPO、caption 错误率相对基线降 28%）
- https://github.com/ddlBoJack/Omni-Captioner — Omni-Detective / Omni-Captioner / Omni-Cloze（官方一手，ICLR 2026；agentic Query-Observation 循环标注框架，音频版即 Qwen3-Omni-Captioner，音视频版并入 Qwen3.5-Omni）
- https://arxiv.org/html/2604.15804v2 — Qwen3.5-Omni 技术报告（官方一手；明确把「剧本级细粒度描述、自动切片、时间戳打标、人物与音频关系描述」定位为视频合成模型训练数据生成能力；Omni-Cloze Plus 64.8 > Gemini-3.1 Pro 57.2）
- https://arxiv.org/html/2601.03233v1 — LTX-2（官方一手；第 5.1 节自研音视频 captioner，字段含 camera motion / lighting / subject behavior + music / ambient / 对白转写含 speaker-language-accent，原则为 comprehensive yet factual、显式禁止情绪解读）
- https://arxiv.org/html/2502.12782v1 — VidCapBench（第三方基准；面向可控 T2V 的视频 caption 评测）
- https://arxiv.org/abs/2606.08572 — OmniCap-IF（第三方/学术；首个音视频 caption 指令跟随基准，50 类约束 + format/content 双维评分）
- https://arxiv.org/html/2507.11336 — UGC-VideoCaptioner + UGC-VideoCap（第三方/学术；1000 条 TikTok 视频 + 4000 QA，三阶段 human-in-the-loop 分别标 audio-only / visual-only / joint AV）
- https://arxiv.org/abs/2502.11775 — video-SALMONN-o1（官方一手；pDPO 过程级偏好优化 + RivaBench，定位 QA/reasoning 而非 captioner）
- https://huggingface.co/AVoCaDO-Captioner/AVoCaDO — AVoCaDO 权重（官方一手）
- https://arxiv.org/html/2412.09283v1 — InstanceCap（第三方/学术；实例感知结构化 caption 提升 T2V）
- https://www.marktechpost.com/2025/01/15/bytedance-researchers-introduce-tarsier2-a-large-vision-language-model-lvlm-with-7b-parameters-designed-to-address-the-core-challenges-of-video-understanding/ — Tarsier2 报道（第三方报道）
- 本仓库同批调研的 30 份生成模型条目（Movie_Gen.json / HunyuanVideo.json / Seedance_20_Seedance_15_pro.json / Open-Sora.json / SkyReels.json / MOVA.json / LTX-2.json / CogVideoX.json / Allegro.json / Goku.json / LongCat-Video.json / Step-Video-T2V.json / Sora_2.json / Veo_3_Veo_31.json / pretraining_datasets.json 等）的 caption_model 与 data_ablation 字段 — 生成侧对 captioner 选型的一手技术报告转述（同项目旁证）

## 数据规模与分布

## 清洗流程

## 打标方式

## 音视频对齐

## 训练配合

## 效果对比
