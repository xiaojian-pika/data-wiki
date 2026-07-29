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

### 训练数据量级（视频条数/小时数/token数，预训练与SFT分开） ⚠️

此处指「captioner 自身的训练数据规模」与「captioner 产出的标注规模」两个量，二者需严格区分：
【captioner 训练数据（预训练/SFT 分列）】
· Tarsier2-7B：预训练 40M video-text pairs（从 Tarsier1 的 11M 扩张 3.6 倍）；SFT 阶段做细粒度时序对齐（论文称 150K 级人工精标，含事件时间戳）[规模数字部分不确定]；再叠加 model-based sampling 自动构造的偏好对 + DPO。
· ShareCaptioner-Video：SFT 数据 = 40K 条 GPT-4V 标注的密集 caption（DiffSW 格式），基座 InternLM-XComposer2-4KHD。以极小的 SFT 量换取大规模推理能力，是「少量高质量教师数据蒸馏」的典型样本。
· AuroraCap：三阶段训练，累计超过 2000 万条高质量图/视频-文本对。
· SkyCaptioner-V1：约 200 万条概念均衡视频（从 1000 万条精选，保留率 20%），32 张 A800，全局 batch 512。
· AVoCaDO：AVoCaDO-SFT 107K（TikTok-10M 24K + ShortVideo 18K + Shot2Story 20K + FineVideo 29K + YouTube-Commons 11K + CinePile 5K），后接 GRPO。
· AVSCap：AVSCap-130K = 4 万条视频 × 3 份标注（visual / audio / synergistic omni-modal），后接 GRPO；论文明确论断「RL 带来的增益大于扩大 SFT 数据量」。
· CogVLM2-Caption：教师链产出的稠密 caption 数据（其中 GPT-4 摘要样本 5 万条用于微调 LLaMA2 摘要器）；student 端总量未披露[不确定]。
· video-SALMONN 2：未披露具体条数[不确定]，模型档位 3B/7B/72B。
【captioner 产出的标注规模（下游可用量）】
· ShareCaptioner-Video → 480 万条（4.8M）高质量美学视频标注，是开源社区最大的单模型产出之一。
· Tarsier2 → 对 100 万条公开数据集视频做 recaption，发布其中 585K（Tarsier2-Recap-585K）。
· Panda-70M → 7000 万条 clip 的 caption，但 caption 极短（均 13.2 词）。
· Koala-36M → 3600 万条 clip，caption 均长 202.3 词（比 Panda-70M 长约 15 倍）。
· 生成侧的隐性产出量级更大：Movie Gen 全量 clip 由 LLaMa3-Video 标注（70% 来自 8B、30% 来自 70B）；Apollo/Klear 对 8100 万条样本做多模型标注（含 Gemini-2.5-Pro 调用）；Harmony 用 Gemini 标注 400 万条音视频片段。
【生态级观察】caption 长度跨度达 60 倍——从 Panda-70M 的 13.2 词到最长的 824.2 词（详见 pretraining_datasets 条目），打标模型选择直接决定 caption 长度，进而决定下游 T2V 模型对 prompt 长度的敏感区间。

### 数据来源构成（自有/公开数据集/网络爬取/授权采购/合成数据）

captioner 训练/评测数据的来源构成：
【教师蒸馏为绝对主流（2024–2026 不变的范式）】GPT-4V → ShareGPT4Video-40K、Koala-36M 种子 caption、MiraData 结构化 caption；GPT-4 → CogVideoX 的摘要环节；Gemini-2.5-Pro → AVoCaDO-SFT、Script-a-Video 的 500K MTSS 标注、Harmony 的 400 万条；Gemini-3 系列 → 2026 年新工作的默认教师。学生模型普遍为 7B 级开源基座（InternLM-XComposer2、Qwen2.5-VL-7B、Qwen2.5-Omni-7B、LLaVA-Video、LLaMA2/LLaMA3）。
【公开数据集复用】Tarsier2-Recap-585K 全部来自公开数据集（VATEX、TGIF、LSMDC 等）；AVoCaDO-SFT 来自 Shot2Story、FineVideo、YouTube-Commons、CinePile 等公开源 + TikTok/短视频；AVSCap-130K 来自 AVoCaDO-107K、ASID-1M、FineVideo、TimeChatCap-40K、Movie101 —— 呈现明显的「数据集叠数据集」的滚雪球式复用。
【网络爬取】ShareGPT4Video 的 4.8M 美学视频、Koala-36M、Panda-70M（HD-VILA-100M 衍生）主要为 YouTube 等公开网络视频。
【人工精标（少量但关键）】SkyCaptioner-V1 的相机运动子专家用 9.3 万条高置信度人工标注 + 1.6 万条运动轴均衡合成数据；Movie Gen 后训练阶段 caption 由人工在模型输出上精修；ALIVE 的 caption 模型训练数据为「MLLM 生成后人工修订」。
【合成/自举】video-SALMONN 2 明确用自身产出的高质量 caption 语料再做后续 SFT（自举式数据生产）；Tarsier2 用 model-based sampling 自动构造偏好对。

### 数据合规与溯源（授权数据占比、rights-cleared数据集、C2PA等） ⚠️

该维度是本生态最薄弱的一环，绝大多数 captioner 论文完全不讨论：
【模型许可清晰】AVoCaDO（Apache-2.0）、video-SALMONN 2（Apache-2.0）、Tarsier2-Recap-7b、CogVLM2-Caption、SkyCaptioner-V1、Aria（Apache-2.0）均给出明确权重许可；ShareGPT4Video 论文 CC BY 4.0。
【数据来源合规几乎无披露】ShareGPT4Video 的 4.8M 网络视频、Koala-36M、Panda-70M 均未说明版权状态与授权来源；AVoCaDO-SFT 含 TikTok-10M 与 ShortVideo 子集，平台内容的再分发合规性未讨论；无任何 captioner 工作提及 C2PA、rights-cleared 数据集或授权采购占比。
【一个结构性风险】教师蒸馏范式普遍违反商用 API 的服务条款（OpenAI / Google 均禁止用输出训练竞品模型），ShareGPT4Video、Koala-36M、AVoCaDO、Script-a-Video 等大量工作依赖 GPT-4V / Gemini 蒸馏，学术发布无碍但商用落地存在法律不确定性——这一点在所有论文中均未被提及。
【生成侧相对谨慎】Movie Gen、Veo 3 等大厂技术报告在数据合规上着墨更多，但涉及 captioner 本身时只说「用了内部模型」，反而规避了这一问题。
[不确定] 无任何公开的定量授权占比数据。

### 片段时长分布与切分策略 ⚠️

captioner 的输入时长处理与适配能力（这直接决定它能标注什么样的 clip）：
【短片段（5–20 秒）为主流适配区间】绝大多数生成模型训练 clip 落在此区间，captioner 均按此优化。Koala-36M clip 均长 13.75 秒；Foley-Omni 固定 8 秒片段。
【抽帧策略是核心工程参数，各家差异极大】Open-Sora 的 PLLaVA 均匀抽 4 帧；Ovi 抽 7 帧关键帧 + 完整音轨；MAGI-1 经实证选定「每片段 4–12 帧（依时长而定）」为描述准确度与算力的最优权衡；CogVideoX 教师链每 2 秒抽 1 帧做稠密图像 caption；Panda-70M 的 BLIP-2 分支从 0.3N–0.7N 帧区间随机取单帧；Aria 官方称可在 10 秒内为 256 帧视频生成 caption（吞吐优势是 Allegro 选它的关键）。
【长视频专门方案】ShareCaptioner-Video 的 DiffSW 是唯一为任意长度视频设计的可扩展方案——先为首帧生成详细 caption，再以长度为 2 的滑窗按时间顺序处理后续帧，每步输入「前一帧 + 其差分 caption + 当前帧」，输出帧间变化（涵盖相机运动、物体运动、人物动作、场景转换）；其复杂度随帧数线性增长而非平方增长，并支持 clip summarization（对已处理片段快速摘要，无需重新处理帧）。AVoCaDO 输出长度最优区间为 2048–4096 token，超过 4096 由 ℛ_L 长度正则惩罚。AVSCapBench 视频时长为 30–120 秒，代表评测重心正向中长视频迁移。
[不确定] 各 captioner 训练数据的时长分布直方图均未公开。

### 分辨率/宽高比分布与分桶策略 ⚠️

[不确定] 该维度在 captioner 生态中普遍缺乏披露，仅有零散证据：
· ShareCaptioner-Video 官方模型卡明确声称「支持各种时长、宽高比与分辨率的视频」，其基座 InternLM-XComposer2-4KHD 具备 4K 高分辨率理解能力，这是它相对同期 captioner 的差异化能力。
· Koala-36M 的 clip 统一为 720p。
· AuroraCap 用 token merging 压缩视觉 token 以应对长序列开销，间接说明高分辨率/长视频输入是主要成本来源。
· Panda-70M 的 UMT 检索择优模型固定 12 帧 224×224。
· 主流做法是 captioner 在低分辨率抽帧上运行（打标只需语义正确，无需像素级保真），因此分辨率对打标质量的影响远小于对生成训练的影响——这解释了为何该维度普遍不被讨论。
· 未见任何 captioner 工作报告「分辨率/宽高比分桶策略」，这类分桶属于生成模型训练侧而非打标侧。

### 类别/domain分布与配比策略（人物、动作、场景、风格等比例控制与概念均衡） ⚠️

【概念均衡是打标器训练的显式设计目标（少数几个明确处理的工作）】
· SkyCaptioner-V1：训练用「约 200 万条概念均衡（concept-balanced）视频，从 1000 万条精选」，这是本生态中最明确的类目均衡声明；其相机运动子专家进一步用「1.6 万条运动轴均衡合成数据」补足长尾运镜类型。
· AVoCaDO-SFT-107K 的六源混合本身即是 domain 配比设计：TikTok-10M 24K（UGC 短视频）+ ShortVideo 18K + Shot2Story 20K（多镜头叙事）+ FineVideo 29K（最大宗，教育/生活类）+ YouTube-Commons 11K（长尾通用）+ CinePile 5K（影视）——即以 UGC 为主体、影视为少量高质量补充。
· AVSCap-130K 来源为 AVoCaDO-107K + ASID-1M + FineVideo + TimeChatCap-40K + Movie101，Movie101 的引入显著提高了影视叙事类占比。
· Panda-70M 的多教师贪心集合覆盖本质上是一种「按内容类型分派打标器」的隐式 domain 处理：单个最好的打标器仅覆盖 30.8% 的样本，31 个全用覆盖 84.7%，贪心选出的 8 个覆盖 76.8%——说明不同 domain 的视频需要不同的打标器，没有一个模型能通吃。这是本生态最重要的 domain 分布洞察。
【评测侧的 domain 类目体系】AVSCapBench 视频来自 YouTube / TikTok / Video-MME 三源；Omni-Cloze 覆盖 9 大领域 47 个子类（阿里唯一公开完整类目体系的工作）；UGC-VideoCap 专注 TikTok UGC（1000 条）；VDC 的结构化 caption 分 camera / short / background / main object / detailed 五类（这是字段维度而非 domain 维度）。
【生成侧对 captioner 的 domain 要求】Allegro 用 Tag2Text 的标签输出直接作为 people / objects / landscapes 三类目分布统计的依据——即打标器兼任数据分布统计工具；Goku 的分布均衡设计依赖 caption 语义聚类；LongCat-Video 用 LLM 对 caption 嵌入的聚类结果做类别命名。
【缺口】除 SkyCaptioner-V1 外，无任何 captioner 公开其训练数据的 domain 直方图或均衡策略的量化收益。[部分不确定]

### 音频类别分布与配比（语音/音效foley/音乐/环境音/静音的比例及控制策略）——AV模型独有维度 ⚠️

这是 2025Q4–2026 年音视频 captioner 生态最具区分度的维度，且 AVSCapBench 首次给出了完整的横向量化证据：
【三类音频的显式建模】AVSCap 的 Acoustic Completeness 准则明确要求 caption 同时覆盖 Speech（语音）/ SFX（音效）/ Music（音乐）三类。Foley-Omni 的三字段 schema 与 Bandit（cinematic audio source separation，天然按 speech / effects / music 三路分离）完全同构。MOVA 采用双模型分工：Qwen3-Omni-Instruct 负责语音（ASR）、Qwen3-Omni-Captioner 负责非语音声音与音乐。
【AVSCapBench event recall（%）横向对比表 —— 本生态最有价值的一张数据】
模型 / Visual / Speech / Music / SFX / Synergy / Overall：
· AVSCap-7B：59.33 / 69.45 / 40.36 / 30.82 / 57.70 / 60.44
· AVoCaDO-7B(9B)：50.59 / 70.42 / 38.71 / 19.25 / 29.13 / 49.31
· video-SALMONN-2-7B：39.05 / 46.76 / 13.76 / 8.71 / 12.43 / 32.02
· Qwen3-Omni-30B：41.85 / 49.08 / 9.34 / 8.68 / 16.19 / 35.29
· Qwen2.5-Omni-7B（裸基座）：34.78 / 13.92 / 4.02 / 7.22 / 7.00 / 21.53
· Gemini-3-Pro：60.43 / 79.81 / 39.52 / 27.77 / 48.88 / 60.97
· Gemini-3-Flash：58.14 / 79.78 / 39.46 / 32.34 / 48.94 / 60.54
【结论性观察】(1) 语音是所有模型最强的音频维度（40–80 分），音乐次之（4–40 分），SFX 最弱（7–32 分）——音频类别间的能力落差高达 8 倍，这直接限制了 Foley/环境音生成模型的训练数据质量；(2) 7B 开源模型（AVSCap 60.44）overall 已追平 Gemini-3-Pro（60.97），且 Synergy 反超（57.70 vs 48.88），但 Speech 仍落后 10 个点；(3) 裸 Qwen2.5-Omni 的 Speech 仅 13.92，证明 omni 基座不做 caption 专项训练完全不可用。
【生成侧的音频类别配比处理】Movie Gen 同时保留 AED 的音乐后验概率与音乐 caption 两路信号（因音乐 caption 模型在无音乐时易幻觉），实测这种冗余组合可控性最好——这是「音频类别分布」影响打标器设计的直接工程证据。LTX-2 的数据筛选标准是「包含显著且信息量大的音频成分」，以保证视觉与听觉内容分布均衡（具体比例未披露[不确定]）。

### 叙事结构分布（单镜头vs多镜头、平均clip时长、镜头数分布、是否含原生音轨） ⚠️

【单镜头 vs 多镜头的打标分野】主流生成模型训练 clip 为单镜头（经 PySceneDetect 等切分后），因此绝大多数 captioner 按单镜头优化。少数针对多镜头/叙事的例外：
· ShareCaptioner-Video 的 DiffSW 显式建模「场景转换（scene transitions）」，是少数原生支持跨镜头描述的开源 captioner。
· AVoCaDO 的五维 keypoint 中「Spatio-temporal & Cinematography」明确覆盖场景切换、时间推进与运镜。
· Shot2Story（AVoCaDO-SFT 中 20K 条）与 Movie101（AVSCap-130K 来源之一）本身即多镜头叙事数据集。
· CineDance 用 Qwen3.5-27B 做镜头分组与叙事边界判定，自底向上策略取得 F1 = 88.4%，仅 3.1% 序列短于 20 秒软阈值——这是本生态中唯一给出叙事结构解析定量指标的工作。
· MOVA 在视觉标注指令中明确要求「聚焦视频场景转场」。
【是否含原生音轨】纯视觉 captioner 完全忽略音轨（Panda-70M 甚至明确关闭 Video-LLaMA 的音频分支）；音视频 captioner 要求样本必须有有效音轨，LTX-2 的数据子集筛选条件即为「包含显著且信息量大的音频成分」。
【平均 clip 时长】Koala-36M 均 13.75 秒；AVSCapBench 30–120 秒；Foley-Omni 固定 8 秒。
[不确定] 无任何 captioner 公开其训练数据的镜头数分布直方图。

### 语言/口音分布（多语种唇同步能力的数据基础） ⚠️

【双语/多语打标能力（少数明确处理的工作）】
· Seedance 1.0 的 captioner（Tarsier2 基座）明确在中英双语数据上训练以获得双语打标能力，训练时冻结视觉编码器、对语言模型做全参微调——这是生成侧对 captioner 语言能力的最明确一手要求。
· LTX-2 的自研 captioner 是对语言/口音处理最精细的公开方案：对白转写不仅给文本，还同时标注 speaker（说话人）、language（语言）、accent（口音）三个属性。这一 schema 设计对多语种唇同步能力的数据基础至关重要。
· SkyCaptioner-V1 未明确双语声明；SkyReels-V4 的语音与歌唱内容由 Whisper 转写（多语种）。
【ASR 侧的多语支持】Whisper-large-v3（多语种，~1.55B）是全生态默认选择；阿里 SenseVoice（~234M，支持中英粤日韩 + 情感/事件识别）被 Apollo/Klear 与 Whisper、Qwen2.5-Omni 三模型并用；ElevenLabs Scribe 被 InstructAV2AV 用于精确时间戳；Qwen3-Omni 系列原生多语。
【口音处理的普遍空白】除 LTX-2 外，未见任何 captioner 显式标注口音；AVoCaDO 的对白 reward 只做 (speaker, spoken content) 二元组，不含语言/口音字段。
[不确定] 无任何 captioner 或其训练数据公开语种分布比例。

## 清洗流程

### 清洗漏斗整体结构（几级过滤、各级顺序）

本条目的「清洗流程」有两层含义，需分开：(A) captioner 自身训练数据的清洗；(B) captioner 在生成模型数据 pipeline 中所处的位置与承担的过滤职能。
【(A) captioner 训练数据的清洗漏斗】普遍简单，典型为「教师模型生成 → LLM 打分过滤 → SFT → RL」四段：
· AVoCaDO：Gemini-2.5-Pro 分别生成 visual caption 与 audio caption → 把两条 caption + 原视频再喂回 Gemini-2.5-Pro 合成时序连贯的多模态 caption → GPT-4.1 对「synthesis completeness」打 1–5 分，只保留 ≥4 分 → SFT → GRPO。
· AVSCap：三阶段 —— decoupled unimodal anchoring（解耦单模态锚定）→ cross-modal fusion（跨模态融合）→ automated verification（tag 保留检查 + 语义一致性检查）→ SFT → GRPO。
· CogVideoX 教师链：Panda-70M 短 caption → CogVLM 每 2 秒一帧的稠密图像 caption → GPT-4 按时间戳摘要 → 5 万条数据微调 LLaMA2 替代 GPT-4 → 再蒸馏为端到端的 CogVLM2-Caption。这是「四段式教师链逐级降本」的经典样本。
· SkyCaptioner-V1：Qwen2.5-VL-72B 产出通用描述 + 三个子专家 captioner（镜头/表情/相机运动）补充影视专业维度 → 融合 → 蒸馏进 7B 统一模型。
· Panda-70M：31 个候选打标器并行生成 → 用户研究做贪心集合覆盖选出 8 个 → UMT-large 细粒度视频-文本检索择优。这是唯一一个把「打标器选择」本身做成一道过滤工序的工作。
【(B) captioner 在生成 pipeline 中的位置】三种典型摆位：
· 漏斗中段的语义闸门：Allegro 把 Tag2Text 放在第 6 级，其标签输出直接作为第 7 级 CLIP 相似度过滤的文本侧输入 —— 打标器同时是过滤器的上游依赖。
· 漏斗末端的全量打标：绝大多数工作（HunyuanVideo、Step-Video、Movie Gen、Seedance）在所有过滤通过后才做打标，因为打标是最贵的一步。
· 分级打标金字塔：Open-Sora 2.0 低分辨率（256px）海量数据用开源 LLaVA-Video 打标、高分辨率（768px）精选 5M 数据改用 Qwen 2.5 Max 重标 —— 「粗标底层 + 精标顶层」与数据金字塔严格对应，是低成本策略的核心一环。Movie Gen 的 70% 8B / 30% 70B 混比是同一思路的另一实现。
· 在线打标（罕见）：UniVerse-1 把标注放进训练循环，因此被迫选择轻量模型（Qwen2.5-Omni + Whisper），无法承受 120B 级模型的逐样本推理开销 —— 这是「标注时机」与「标注模型容量」之间权衡的唯一公开案例。

### 漏斗定量保留率（每级过滤的输入/输出量与最终保留率，如Apollo 27%） ⚠️

[不确定] captioner 生态整体缺乏定量保留率披露，仅有零星数字：
· SkyCaptioner-V1：训练数据从 1000 万条精选出约 200 万条概念均衡视频，保留率 20%。这是本条目唯一明确的 captioner 训练数据保留率。
· AVoCaDO：GPT-4.1 对 synthesis completeness 打 1–5 分、只保留 ≥4 分，但未披露该步的通过率。
· Tarsier2：对 100 万条公开数据集视频做 recaption，最终发布 585K（Tarsier2-Recap-585K），若 585K 全部来自该 1M 则保留率约 58.5%，但论文未确认二者是同一批（也可能是主动抽样发布而非过滤）。[不确定]
· Panda-70M 的贪心集合覆盖给出的是「覆盖率」而非保留率：单个最好模型覆盖 30.8%、8 个覆盖 76.8%、31 个全上 84.7%；同时报告人类之间的一致率仅 44.9%，说明「最佳 caption」本身高度主观。
· AVSCap-130K、AVoCaDO-SFT-107K 均未给出「原始候选量 → 最终保留量」的逐级数字。
· 生成侧最完整的端到端保留率来自 Apollo/Klear（27%），但那是整条数据漏斗而非打标环节。

### 镜头切分方法（PySceneDetect/自研模型/shot-aware splitting）

captioner 本身通常不做镜头切分（切分在其上游完成），但有三类交互：
【打标器承担叙事分组】CineDance 用 Qwen3.5-27B 做镜头分组与叙事边界判定（自底向上镜头索引分组优于直接让 LLM 输出时间戳），F1 = 88.4%、仅 3.1% 序列短于 20 秒软阈值，是打标模型直接承担切分决策的代表。
【打标器描述转场】ShareCaptioner-Video 的 DiffSW 显式识别 scene transitions；MOVA 的视觉标注指令明确要求「聚焦视频场景转场」；AVoCaDO 的 Spatio-temporal & Cinematography 维度覆盖场景切换。
【切分质量反过来决定打标质量】Koala-36M 的核心贡献之一即「更准确高效的转场检测方法」，其消融显示：用 Koala-all 训练相较 Panda-70M 训练，VBench 主体一致性 +1.1%、背景一致性 +2.4%，论文将该时序质量提升归因于更准的镜头切分 —— 这是「切分 → caption 一致性 → 生成质量」链条的唯一量化证据。CineDance 的伪影审计显示 CineDance-1M 不合规率 2.8% vs Koala-36M 37.4%（13.4× 改善）。
【上游常用工具】PySceneDetect 仍是行业默认；各家自研的转场检测模型（Koala-36M、CineDance）主要针对渐变转场与快速剪辑的漏检问题。

### 质量过滤（美学评分、清晰度、OCR文字过滤、黑边/水印/logo检测）

captioner 生态的「质量过滤」有两个方向：
【方向一：captioner 作为质量过滤器的组成部分】
· Allegro：Tag2Text 输出作为 CLIP 相似度过滤的文本侧输入，打标器直接嵌入过滤链。
· Mochi/MAGI/Motif 批次中的 Motif-Video 2B：另配 PaddleOCR-VL（经 vLLM 服务化）做帧上文字检测，属于打标链中的 OCR 过滤分支。
· InstructAV2AV 用 Grounded-SAM-2 提供实例级 mask 锚点，TalkNet 做主动说话人检测 —— 结构化标注工具兼任质量闸门。
· 传统浅层过滤器（LAION 美学分、DOVER technical score、清晰度、黑边/水印/logo 检测）与 captioner 并行而非串行，通常在打标之前完成以节省打标算力。
【方向二：captioner 输出本身的质量控制】这是本生态的核心问题，主流手段有四：
(1) LLM 打分过滤：AVoCaDO 用 GPT-4.1 打 synthesis completeness 1–5 分、只保留 ≥4；
(2) 自动化一致性校验：AVSCap 的 automated verification（tag 保留检查 + 语义一致性检查）；
(3) 检索式择优：Panda-70M 用 UMT-large（ViT-L/16 + BERT-large，VTC+VTM 双损失 + 难负例挖掘，未选中的 7 条 caption 权重 1.0、batch 内其他负例权重 0.01，12 帧 224×224，AdamW lr 2e-5，batch 32，10 epoch，8×A100-80G）从多候选中选最佳 caption，微调后 R@1 达 35.90%（预训练 UMT 仅 21.82%）；
(4) 幻觉抑制的后训练：腾讯混元 1.5 对其三个 caption 模型用 OPA-DPO（针对多模态幻觉的偏好优化）做 RL 后训练；video-SALMONN 2 的 MrDPO 同时奖励完整性与事实性，7B 模型 caption 错误率相对基线降 28%；Tarsier2 用 model-based sampling 构造偏好对做 DPO。
【生成侧的兜底】Step-Video-T2V 不对 caption 模型做幻觉抑制专门后训练，兜底依赖 SFT 阶段人工复核 + 第 6 阶段 CLIP Score 对齐过滤 —— 代表了「打标器不够好就用下游过滤补」的务实路线。

### 运动过滤（光流/运动分数阈值、静态与抖动剔除）

captioner 生态与运动过滤的交互点有三：
【打标器输出运动信息供过滤使用】Tarsier2 天然会描述相机运动类型（zoom in / pan right 等），Goku 论文明确指出这是选它做视频级 caption 的关键优势 —— 无需额外的相机运动标注模块即可获得镜头语言标签。
【专用运动分类器与 captioner 分工】主流做法是运动识别不交给通用 VLM 而用轻量分类器：Movie Gen 训练了 16 类相机运动分类器，高置信度预测结果作为前缀拼到 caption 上；LongCat-Video 的相机运动（pan/tilt/zoom）由单独训练的轻量分类器负责而非 VLM（推测出于成本与精度考虑）；HunyuanVideo 有独立的自研 camera movement classifier（1.5 版升级为 clip 级 + 时序级双粒度）；SkyCaptioner-V1 的相机运动子专家基于分类，训练数据为 9.3 万条高置信度人工标注 + 1.6 万条运动轴均衡合成数据，1.5 万条测试集上单类型运动准确率 89%。
【运动过滤本身（光流/运动分数阈值、静态与抖动剔除）属于打标上游】Foley-Omni 用 motion∈[0.1, 3.2] 阈值；InstructAV2AV 用 CoTracker3 运动阈值；Open-Sora Plan 用 LPIPS 上界 0.3（超过即出现抖动闪烁，结论经 2000 条人工抽检验证）。这些均在 captioner 之前执行。
【生态判断】「运镜识别用专用分类器、内容描述用 VLM」是 2024–2026 年稳定的分工范式，原因是 VLM 对运镜的判别准确率不稳定且难以量化验证，而分类器可给出置信度用于过滤。SkyCaptioner-V1 的做法（把分类器结果蒸馏回 7B 统一 captioner）是两条路线的融合尝试，其影视专业字段平均准确率 76.3%（镜头类型 93.7%、机位角度 89.8%、机位位置 83.1%、相机运动 85.3%），显著超过 Qwen2.5-VL-72B 等更大的通用模型。

### 去重方法（精确去重与基于embedding的语义去重分别记录） ⚠️

[不确定] 这是 captioner 生态披露最少的字段之一。
【已知的少量证据】
· AVSCap-130K 的来源包含 AVoCaDO-107K，AVoCaDO-SFT 的来源又包含 FineVideo、Shot2Story 等公开数据集，而这些公开集之间本身存在视频重叠 —— 「数据集叠数据集」的滚雪球式复用带来了显著但未被量化的跨数据集重复风险，两篇论文均未讨论去重。
· Tarsier2-Recap-585K 从 1M 条公开数据集视频中产出，其来源（VATEX、TGIF、LSMDC 等）之间同样存在已知重叠，去重策略未披露。
· Panda-70M 的贪心集合覆盖是对「打标器」去冗余（31 个降到 8 个），不是对视频样本去重。
· Goku 用 Qwen2 作为 LLM 融合器把关键帧 caption 与视频 caption 合并为「统一、无冗余、无矛盾」的最终描述 —— 这是 caption 文本级的冗余消除，与样本级去重不同。
· LongCat-Video 用 LLM 对 caption 嵌入的聚类结果做类别命名，caption embedding 聚类具备做语义去重的技术条件，但报告未说明是否用于去重。
【生态判断】视频样本的精确去重（哈希）与语义去重（embedding）普遍在 captioner 上游完成、由生成侧团队执行且很少披露；caption 文本本身的去重（避免同质化模板句）几乎无人讨论 —— 考虑到 CogVideoX 的 prompt 明令禁止「The video presents / depicts / showcases」「throughout the video」等套话，说明 caption 同质化是真实存在的问题，但业界靠 prompt 约束而非事后去重解决。

### VLM/LLM作为质检员（多模态大模型质量打分与错配剔除，2026年从浅层打分器转向大模型语义判断的趋势）

这是 2025–2026 年该生态最显著的趋势，且 captioner 生态同时是「被判」与「判人」的双重角色：
【趋势一：LLM/VLM 作为 caption 质量裁判（浅层打分器 → 大模型语义判断）】
· AVoCaDO：GPT-4.1 对 Gemini 合成 caption 打 synthesis completeness 1–5 分，只保留 ≥4；GRPO 阶段进一步用 GPT-4.1 判定五维 checklist 覆盖度作为 reward ℛ_C。
· AVSCap：automated verification 做 tag 保留检查 + 语义一致性检查；GRPO 的 hybrid reward 含 audio-visual consistency 项。
· MOVA：GPT-OSS-120B（120B 开源模型）承担 caption 融合 + 跨模态一致性校验 —— 把最大的模型放在融合与裁决环节而非感知环节，是成本与效果权衡的典型选择。
· AuroraCap 提出 VDCScore：用分治策略把长 caption 评测转化为多个短问答对，由 LLM 判定 —— 代表 caption 评测从 CIDEr/BLEU 等 n-gram 指标向 LLM 语义判断的迁移。
· Omni-Cloze 反其道而行：用 2000 clip / 7 万道细粒度完形填空题规避 LLM-judge 噪声，是对 LLM-as-judge 可靠性的一种质疑与修正。
【趋势二：captioner 作为数据质检员】
· InstructAV2AV：Qwen3-Omni 既生成编辑指令又承担五维度验证打分 —— 论文自身承认「生成与验收同源是本 pipeline 的方法论隐患」，这是本生态少见的自我批评。
· Vidu S1 在质量过滤阶段使用 omni model 做判别（细节未披露）。
· Foley-Omni 用 Gemini 2.5 Pro 做标注，再用 Bandit（cinematic audio source separation）做声学后验证纠偏视觉幻觉，阈值 −35 dB —— 「大模型标注 + 信号级验证」的组合是抑制多模态幻觉的有效范式。
【方法论隐患（全生态共性）】(1) benchmark 与 model 同源：AVSCap 既是 AVSCapBench 的作者又是榜首（60.44），存在过拟合风险，其分数需用第三方基准（UGC-VideoCap、Omni-Cloze）交叉验证；(2) 教师即裁判：多数工作用 Gemini/GPT 家族既做教师又做裁判，评分偏向自身输出风格；(3) 人类一致性天花板：Panda-70M 报告的人类之间 caption 偏好一致率仅 44.9%，意味着 LLM-judge 的「准确率」上限本身就模糊。

### 安全与合规过滤（NSFW、版权、人脸/隐私） ⚠️

[不确定] 该字段在 captioner 生态中几乎完全空白，仅有间接证据：
· NSFW / 版权 / 人脸隐私过滤普遍在 captioner 上游由生成侧团队完成，captioner 论文一律不讨论。
· 唯一相关的设计是 LTX-2 captioner 的「comprehensive yet factual」原则 —— 只描述看到与听到的，显式禁止情绪解读（emotional interpretation）。这既是防幻觉措施，客观上也降低了 caption 对人物主观状态做出可能有偏见的判断的风险，是本生态中最接近「安全设计」的公开做法。
· AVoCaDO-SFT 含 TikTok-10M 与 ShortVideo 子集，涉及真实人物 UGC 内容，未讨论人脸/隐私处理。
· 涉及说话人身份标注的工作（AVoCaDO 的 (speaker, content) 二元组、LTX-2 的 speaker/language/accent、CineDance 的角色 anchor token 绑定）都在生成与人物身份高度相关的标注，但均未讨论隐私合规。
· 生成侧团队的安全过滤（Movie Gen、Veo 3）在其各自条目中记录，与 captioner 选型无直接耦合。

## 打标方式

### 使用的caption模型（自研VLM/开源模型，模型规模） ⚠️

本条目的核心字段。按「谁被用来打标」给出完整生态图谱：

【第一梯队：专为视频打标训练的开源 captioner（下游复用率最高）】
· Tarsier2-7B（字节跳动，2025-01）：预训练 11M→40M video-text pairs、SFT 做细粒度时序对齐、model-based sampling 构造偏好对 + DPO。DREAM-1K F1 超 GPT-4o 2.8%、超 Gemini-1.5-Pro 5.8%，人评 +8.6% / +24.9%，是首个 DREAM-1K overall recall 突破 40% 的模型，15 个公开基准 SOTA。天然描述相机运动类型是其独特优势。下游被 Seedance 1.0（作为基座，冻结视觉编码器 + LLM 全参微调 + 中英双语训练）、Goku（视频级 caption，与 InternVL2.0 关键帧 caption 互补，Qwen2 融合）、LongCat-Video（时序增强）复用 —— 是被生成侧采用最广的单一开源 captioner。
· ShareCaptioner-Video（中科大 + 上海 AI Lab + 港中文，2024-06）：基座 InternLM-XComposer2-4KHD，用 40K GPT-4V 密集 caption SFT 而成，产出 4.8M 高质量美学视频标注。核心方法 DiffSW（差分滑窗）—— 把「全帧到 caption」转化为差分描述任务：先为首帧生成详细 caption，再以长度 2 的滑窗顺序处理后续帧，每步输入「前一帧 + 其差分 caption + 当前帧」，输出相机运动/物体运动/人物动作/场景转换四类变化。支持任意时长、宽高比与分辨率，附带 clip summarization（无需重新处理帧）。Open-Sora Plan v1.3 的 stock footage 部分直接使用其标注。
· CogVLM2-Caption（智谱，2024-09-19 开源）：CogVLM2-Video + Llama3 底座，约 12B 级，由 Panda-70M→CogVLM→GPT-4→LLaMA2 教师链产出的稠密 caption 微调而成，是 CogVideoX 数据 pipeline 中唯一可复现的一环。
· SkyCaptioner-V1（昆仑万维，2025-04）：Qwen2.5-VL-7B-Instruct 基座（小模型以支撑亿级吞吐），知识蒸馏式融合 —— Qwen2.5-VL-72B 产出通用描述 + 三个子专家 captioner（镜头类型/机位角度/机位位置、面部表情与情绪强度、6DoF 相机运动）→ 融合 → 蒸馏进 7B。影视专业字段平均准确率 76.3%，显著超过更大的通用模型。
· AuroraCap（学术，2024-10）：token merging 压缩视觉 token，三阶段 2000 万图/视频-文本对训练，Flickr30k CIDEr 88.9（vs GPT-4V 55.3、Gemini-1.5-Pro 82.2），配套 VDC 基准与 VDCScore 指标。

【第二梯队：通用 VLM 直接当打标器（成本最低、最普遍）】
· Qwen 系列是 2025–2026 年事实标准：Qwen2-VL-7B-Instruct（Open-Sora Plan v1.3 主力）、Qwen2.5-VL（LongCat-Video 的结构化属性判别；SkyCaptioner 基座）、Qwen3-VL（UniTalking 视觉 caption；MOVA 推理侧 prompt 增强）、Qwen3-VL-30B-A3B（Motif-Video 2B 的全部 caption 与标签来源）、Qwen3.5-27B / Qwen3.5-35B-A3B（CineDance 的叙事解析与视觉标注）。选 MoE 稀疏架构（约 3B 激活）在吞吐与质量间取平衡是 2026 年的新趋势。
· LLaVA 家族：PLLaVA-13B（Open-Sora 1.x 主力，配置 2×2 空间池化、抽 4 帧；文档明确解释不用 GPT-4V 是因为「20 秒/样本太慢」）、LLaVA-Video（Open-Sora 2.0 低分辨率阶段、LongCat-Video 主 captioner）、LLaVA v1.6-Mistral-7B、Koala-36M 用 GPT-4V 种子 caption 微调的 LLaVA。
· InternVL2.0（Goku 的关键帧 caption）、Aria 25.3B MoE / 3.9B 激活（Allegro 的细粒度打标器，可在 10 秒内为 256 帧视频生成 caption）、MiMo-VL-7B-RL（MOVA 视觉标注）、Tag2Text（Allegro 的粗粒度标签器）。

【第三梯队：全模态/音视频联合 captioner（2025Q4 起爆发）】
· AVoCaDO（快手可灵 + 中科院自动化所等，2025-10）：Qwen2.5-Omni-7B 基座（HF 标 9B 全栈），AVoCaDO-SFT-107K → GRPO。Reward = ℛ_C（五维 checklist 覆盖，GPT-4.1 判定）+ ℛ_D（对白 F1，speaker 准确率与 content 相似度，编辑距离 + 动态规划找最优对齐子序列，阈值 0.6）+ ℛ_L（长度正则，>4096 token 惩罚）。Apache-2.0。
· AVSCap-7B（南京大学 NJU-LINK + 快手 Kling，2026-07）：同基座，AVSCap-130K → GRPO（长度控制 + speech preservation + AV consistency），AVSCapBench overall 60.44 追平 Gemini-3-Pro（60.97）。论文关键论断：RL 增益 > 扩 SFT 数据量。
· video-SALMONN 2 / 2+（清华电子系 + 字节，2025-06 起迭代至 2026-02）：LLaVA-OneVision + LoRA 加装音频能力，3B/7B/72B，MrDPO（多轮 DPO，周期性用新训轻量 adapter 重置 reference policy），caption 错误率相对基线降 28%，Apache-2.0。
· Qwen3-Omni-Captioner（阿里，2025-09，音频专用）与 Qwen3.5-Omni（2026-03，仅 API）：后者把「剧本级细粒度描述、自动切片、时间戳打标、人物与音频关系描述」直接定位为视频合成模型训练数据生成能力，Omni-Cloze 64.8 > Gemini-3.1 Pro 57.2。配套 Omni-Detective（agentic Query-Observation 循环标注框架）。
· Qwen2.5-Omni / Qwen3-Omni（被 UniVerse-1、UniTalking、MOVA、SkyReels-V4、InstructAV2AV、Apollo/Klear、LongCat Avatar 1.5 广泛用作音频侧或融合侧打标器）。

【第四梯队：闭源 API 作为教师或高价值子集打标器】
· GPT-4V / GPT-4 / GPT-4.1：ShareGPT4Video-40K、Koala-36M 种子 caption、MiraData 结构化 caption、CogVideoX 摘要、AVoCaDO 质检。
· Gemini 系列（2.5-Pro / 3-Pro / 3-Flash）：AVoCaDO 教师、Script-a-Video 的 500K MTSS 全部标注、Harmony 的 400 万条音视频标注、Foley-Omni 主标注（选它因原生支持视频+音频双模态输入，纯视觉 VLM 无法胜任音频存在性判别）、Apollo/Klear 高价值子集、Veo 3 的「多个 Gemini 模型」。
· Qwen 2.5 Max（Open-Sora 2.0 高分辨率精选 5M 数据重标）。

【第五梯队：生成侧自研且完全闭源】
· OpenAI Sora：「highly descriptive captioner model」（沿用 DALL·E 3 re-captioning 思路），Sora 2 零披露。
· Meta Movie Gen：LLaMa3-Video，8B 与 70B 两个变体分别做视频 captioning 微调，训练集 caption 中 70% 来自 8B、30% 来自 70B —— 唯一公开「大小模型混比」数字的工作。另有 16 类相机运动分类器。音频侧四模型协同（音频质量预测 1–10 分、AED 判定 voice/singing 存在性与 music 后验、通用音频 caption、音乐 caption）。
· 腾讯混元：原版为单一自研 VLM 输出 JSON 结构化 caption；1.5 升级为三模型分工（图像 caption、视频 caption、图生视频指令式 caption）+ 镜头运动识别模型，并用 OPA-DPO 抑制幻觉。
· Lightricks LTX-2：为音视频联合生成专门开发的新 captioning 系统，同时详尽描述视觉与听觉轨道。
· 阶跃星辰 Step-Video：in-house VLM，TI2V 版本专门微调强化「物体运动动态」与「镜头运动」描述。
· 快手可灵 3.0 Omni：官方仅称有「视频描述增强」环节，模型未公开[不确定]；同团队公开做法为 Koala-36M 的 LLaVA 微调路线与自研 Kwai Keye-VL（arXiv:2507.01949）。
· 字节 Seedance 1.5 pro / 2.0：称有「先进的字幕系统」为视频与音频提供专业级描述，未指明基座[不确定；推测已切换至自研 Seed-VL 系列]。

【关键选型规律（跨条目归纳）】
(1) 参数量普遍收敛到 7B：ShareCaptioner、Tarsier2、SkyCaptioner-V1、AVoCaDO、AVSCap、MiMo-VL 全部是 7B 级，原因是亿级样本打标必须控制单样本推理成本；把大模型（GPT-OSS-120B、Qwen2.5-72B）放在融合/裁决/教师环节而非感知环节，是全行业一致的成本-效果权衡。
(2) 多模型分工 > 单模型通吃：Panda-70M 的实证最有力 —— 单个最好打标器仅覆盖 30.8% 样本，8 个组合覆盖 76.8%。Goku（图像 VLM + 视频 VLM + LLM 融合）、MOVA（视觉 + ASR + 非语音 + 120B 融合）、SkyCaptioner（通用 + 三子专家）均是此思路的实现。
(3) 运镜识别用专用分类器而非 VLM：Movie Gen 16 类分类器、LongCat 轻量分类器、混元独立分类器、SkyCaptioner 的 6DoF 子专家（89% 准确率）—— 一致的行业选择。
(4) 蒸馏降本是标准动作：GPT-4V/Gemini 教师 → 7B 学生，几乎每个工作都做。Script-a-Video 给出了最完整的蒸馏收益量化（见 data_ablation）。
(5) 2026 年的分水岭是「能不能听」：纯视觉 captioner 无法服务音视频联合生成模型，omni captioner 成为 AV 生成模型的刚需组件。

### caption密度与结构化程度（短/长/密集描述、结构化字段如镜头运动、风格标签）

【长度谱系：从 13.2 词到 824.2 词，跨度 60 倍】Panda-70M 均 13.2 词（单句摘要，prompt 为「Please faithfully summarize the video in one sentence.」）→ Movie Gen caption 平均约 100 词 → Koala-36M 均 202.3 词 → AVoCaDO 最优区间 2048–4096 token。打标模型选择直接决定 caption 长度，进而决定下游 T2V 模型对 prompt 长度的敏感区间。

【结构化程度的四个档次】
· 档次一 · 纯自由文本单句：Panda-70M。适合大规模预训练的粗对齐，不足以支撑可控生成。
· 档次二 · 长密集散文 + 固定句式嵌入：主流做法。Allegro 覆盖主体属性、主体间交互、背景、环境、风格、氛围、镜头角度与运动、时序变化，其中镜头运动以固定句式「Camera [MOTION_PATTERN]」显式插入，其余以散文表达；Movie Gen 把 16 类相机运动分类器的高置信度预测作为前缀拼到 caption 上；Step-Video-TI2V 的示例形如「a flock of birds flying over a tree at sunset, camera pans left」。这种「散文主体 + 结构化片段」的混合形态是可控性与自然性的折中。
· 档次三 · 显式多字段结构化：VDC 定义 camera / short / background / main object / detailed 五字段；Koala-36M 六类结构化信息；SkyCaptioner-V1 输出镜头类型/机位角度/机位位置/相机运动/面部表情与情绪强度等影视专业字段；腾讯混元输出 JSON 结构化 caption；LongCat-Video 用 Qwen2.5VL 判别景别、镜头类型、写实度、动画风格、色调；CineDance 产出镜头级五维属性 + 转场类型 + 局部角色表 + 活跃场景 + 镜头描述 + 转场描述。
· 档次四 · 多粒度并存 + 随机采样：SkyReels-V4 生成短/长/结构化三档描述；Motif-Video 2B 的 caption 三变体按 0.5/0.3/0.2 概率采样（论文坦承这是「务实的配方选择而非经隔离验证的最优性主张」）；UniTalking 同时保有「分头标注后拼接」与「统一模型融合标注」两条路径的产物，训练时随机采样使用 —— 把标注策略的选择交给随机采样，让模型同时适应两种文本分布。

【DiffSW：唯一为可扩展性设计的结构】ShareCaptioner-Video 的差分滑窗把 caption 组织为「首帧详细描述 + 逐步差分描述」的链式结构，天然携带时序增量信息，且支持任意长度线性扩展与事后快速摘要。

【风格约束（被严重低估的工程细节）】CogVideoX 附录 G 的 prompt 明令禁止「The video presents / depicts / showcases」「throughout the video」等套话与换行符；LTX-2 的原则是「comprehensive yet factual」，只描述看到和听到的，显式禁止情绪解读（防止 T2V 训练时 caption 引入不可控主观信号）。这两条是全生态最有参考价值的 caption 风格设计经验。

【与推理侧 prompt 的分布对齐（caption 结构的下游约束）】Open-Sora Plan 明确论证训练 caption（密集长文）与用户 prompt（<10 词）的分布错配会损害生成效果，为此以 LLaMA-3.1-8B-Instruct 做 LoRA 微调（rank 64，batch 32，1 epoch，单张 H100 上 30 分钟）训练 prompt refiner；Seedance 1.0 的 PE 模型与 captioner 同源以保证结构对齐；MAGI-1 把大 MLLM 的增强 prompt 蒸馏到约 7B 小模型（约 200 万条语料）以降低推理延迟；Goku 用 GPT-4o 扩写 GenEval 短 prompt 后分数达 0.76，间接证明模型性能高度依赖与训练期密集 caption 一致的长 prompt。**caption 结构的选择实际上锁定了推理侧必须配套 prompt 改写器**，这是本生态最重要的系统性结论。

【文本编码器的上限约束】Allegro 用 T5（512 token 上限，论文定性论证 T5 优于 mT5）；Mochi 1 用单个 T5-XXL；Foley-Omni 与 UniVerse-1 用 UM-T5/umT5；HunyuanVideo-Foley 用 CLAP 文本编码器（约 77 token，因 CLAP 文本嵌入空间天然与音频语义对齐）；LTX-2 用 Gemma3-12B；Motif 第 4 阶段起换为 T5Gemma2；Apollo/Klear 用 Qwen2.5-7B。编码器的 token 上限直接截断了 caption 密度的可用上界。

### 音视频联合caption结构（是否同时覆盖视觉+听觉轨道、是否分流为独立字段，如LTX-2全音景描述、Script-a-Video factorized streams、Foley-Omni三字段）

音视频联合 caption 的 schema 设计是 2026 年该生态最活跃的创新点，已形成四种可辨识的范式：

【范式一 · 全音景融合式（单条 caption 同时叙述视听）】LTX-2 的自研 captioner 是标杆：视觉侧覆盖 camera motion（运镜）、lighting（打光）、subject behavior（主体行为）；音频侧覆盖 music、ambient sounds，以及精确的对白转写并附带 speaker / language / accent 三属性；设计原则「comprehensive yet factual」，显式禁止情绪解读。AVoCaDO 同属此类，输出自由形式长文本（非 JSON），结构性体现在 reward checklist 的五维分解上：Static Entity Description、Dynamic Action & Interaction、Auditory Elements（语音/音乐/环境音与叙事音效）、Spatio-temporal & Cinematography、**Cross-modal Narrative Logic**（视听互相解释/补充/引导，是其核心创新维度）。

【范式二 · factorized streams（分流为独立字段，各自成轨）】Script-a-Video 的 MTSS（Multi-Track Structured Script）是代表，由 Gemini-2.5-Pro 生成 500K 条标注，再蒸馏进 Qwen3-Omni-MTSS-FT。UniVerse-1 用单个 Qwen2.5-Omni 一次性并列输出「核验后的语音内容 / 视频描述性 caption / 环境音 caption」三路独立字段，不做融合。UniTalking 产出四种格式（Qwen3-VL 详细版 + 简短版视觉 caption、Whisper-V3 转写、Qwen3-Omni-Captioner 音频 caption、Qwen3-Omni 融合式统一描述），训练时随机采样。

【范式三 · 固定三字段 schema（对齐音频源分离的天然分类）】Foley-Omni 的三字段设计（speech / effects / music）与 Bandit（cinematic audio source separation）的输出结构完全同构 —— 若用通用音乐分离模型（如 Demucs 的 vocals/drums/bass/other）则无法直接对应。这是 schema 设计与验证工具协同的最佳案例。MOVA 的双模态分工（Qwen3-Omni-Instruct 处理语音、Qwen3-Omni-Captioner 处理非语音与音乐，GPT-OSS-120B 融合并做跨模态一致性检查）在结构上等价于三字段后融合。

【范式四 · 剧本级结构化（含时间戳与角色绑定）】Qwen3.5-Omni 提供「剧本级细粒度描述，含自动切片、时间戳打标、人物与音频关系描述」，是目前唯一原生支持此形态的通用模型。CineDance 走到了最细：镜头级五维属性 + 转场类型 + 局部角色表 + 句级 ASR + 镜头级音频 prompt（音乐/环境音/音效）+ 角色音色描述，并完成 ASR 句子到角色 anchor token 的绑定。

【质量准则的形式化（AVSCap 的贡献）】高质量 omni-modal caption 需同时满足三条：Acoustic Completeness（覆盖 Speech / SFX / Music 三类）、Visual Completeness（环境、人物、动作、物体交互、camera motion、OCR）、Audio-Visual Synergy（视听事件的显式绑定与时序对齐）。这是目前最可操作的 schema 验收标准。

【输入形态的隐含要求】Ovi 的做法揭示了关键约束：打标输入为「7 帧均匀采样关键帧 + 完整音轨」，即 caption 模型本身必须具备音频理解能力，7 帧的稀疏采样意味着视觉描述偏事件级/语义级而非逐帧密集 —— AV 联合打标天然是「视觉粗、音频细」的不对称形态。

### 对白转写与说话人属性标注（ASR转写+说话人身份/语言/口音/情绪）

【ASR 模型选择的三条路线】
· 外挂专用 ASR（最普遍）：Whisper-large-v3（OpenAI，~1.55B）是事实标准，被 SkyReels-V4、UniTalking、Unison（评测侧）、Apollo/Klear、UniVerse-1、CineDance（评测 WER/CER）广泛使用；阿里 SenseVoice（~234M，中英粤日韩 + 情感/事件识别）被 Apollo/Klear 与 Whisper、Qwen2.5-Omni 三模型并用；ElevenLabs Scribe 被 InstructAV2AV 用于精确语音时间戳。
· Omni 模型内生转写（2026 年趋势）：Ovi 明确不用独立 ASR，台词转写直接由打标 MLLM 从音轨产出；UniVerse-1 用 Qwen2.5-Omni 同时输出核验后的语音内容；CineDance 用 Qwen3-Omni-30B-A3B 统一承担句级 ASR、镜头级音频 prompt、角色音色描述三项子任务，理由是其在说话人归属任务上远超专用 diarization 工具（Pyannote-3.1 仅 62.7%、DiariZen 63.1%）。
· RL 逼出转写能力（AVoCaDO 的独创）：不走独立 ASR 模块，而是在 GRPO 阶段用 dialogue reward ℛ_D 强制模型学会转写 —— 对白被抽取为 (speaker, spoken content) 结构对，ℛ_D = speaker 识别准确率与 content 相似度的 F1，content 相似度用编辑距离 + 动态规划找最优对齐子序列，阈值 0.6。这是目前公开最实用的对白准确率优化方案。

【说话人属性标注的完整度谱系】
· 最完整：LTX-2 标注 speaker / language / accent 三属性 —— 是多语种唇同步能力的数据基础，也是全生态唯一显式标注口音的方案。
· 中等：CineDance 的角色音色描述 + ASR 句子到角色 anchor token 的绑定（窗口化方案把 Qwen3-Omni 从 67.2% 提升到 95.4%，对照 Gemini 系列 82.8%–87.4%）；Apollo/Klear 由音频模型体系抽取性别、年龄等属性；InstructAV2AV 用 TalkNet 做主动说话人检测，标注谁在说话及其时空位置。
· 最简：AVoCaDO 只做 (speaker, content) 二元组，无语言/口音字段。

【说话人分离的方法论对照（CineDance Tab.5，本生态唯一的定量对照）】100 条片段基准上，Qwen3-Omni-30B + 滑窗 prompt 达 83.1%，整片输入仅 56.4% —— 说明 prompt 的窗口化设计对说话人分离的影响远大于模型选择本身。

【语音质量的下游影响】AVSCapBench 显示 Speech 是所有 AV captioner 最强的维度（Gemini-3 系列 79.8、AVoCaDO 70.42、AVSCap 69.45），但裸 Qwen2.5-Omni 仅 13.92 —— 语音转写能力必须靠专项训练获得。

### 几何与结构化标注（相机参数、深度、3D point tracks、动作标注、显式状态） ⚠️

captioner 生态在几何/结构化标注上采取「专用模型外挂、结果拼进 caption」的一致策略，而非要求 VLM 直接输出几何量：
【相机参数与运镜（最成熟）】Movie Gen 的 16 类相机运动分类器（高置信度预测作前缀拼进 caption）；SkyCaptioner-V1 的 6DoF 相机运动子专家（分类式，9.3 万条人工标注 + 1.6 万条运动轴均衡合成数据，1.5 万条测试集上单类型运动准确率 89%，整体相机运动字段准确率 85.3%）；腾讯混元的独立 camera movement classifier（1.5 版为 clip 级 + 时序级双粒度）；LongCat-Video 的 pan/tilt/zoom 轻量分类器；Allegro 以固定句式「Camera [MOTION_PATTERN]」嵌入 caption。Tarsier2 是少数天然会描述相机运动类型的通用 captioner。
【实例级空间标注】InstructAV2AV 用 Grounded-SAM-2（开放词表检测 + 分割 + 视频跟踪）产出实例级 mask，作为「可编辑对象」的锚点与 mask-guided 合成的编辑区域 —— 是本生态中几何标注参与最深的案例。
【OCR / 文字检测】Motif-Video 2B 配 PaddleOCR-VL（经 vLLM 服务化）做帧上文字检测；AVSCap 的 Visual Completeness 准则显式包含 OCR。
【动作/时序标注】ALIVE 做「decomposing sub-motion units」—— 把动作分解为子动作单元并人工标注每个完整小动作的起止时间戳，是本生态成本最高的细粒度时序标注；Tarsier2 的 SFT 阶段做细粒度时序对齐；CineDance 的镜头分组与叙事边界判定（F1 88.4%）。
【深度 / 3D point tracks】[不确定] 未见任何主流 captioner 生态工作输出深度图或 3D point tracks 作为 caption 的组成部分 —— 这类标注在世界模型/机器人视频数据（如 Cosmos 系列）中出现，但不属于 T2V caption 生态的标准配置。
【显式状态标注】CineDance 的局部角色表 + 活跃场景是最接近「显式状态」的公开设计。

### 合成数据构造（受控扰动/编辑构造训练对，如InstructAV2AV）

【合成 caption 数据（本生态最普遍的合成形态）】
· 教师蒸馏本身即是合成：GPT-4V/Gemini 生成的 caption 全部属于合成标注，ShareGPT4Video-40K、Koala-36M 种子集、AVoCaDO-SFT-107K、Script-a-Video 的 500K MTSS 皆是。
· 两阶段合成（AVoCaDO）：Gemini-2.5-Pro 先分别生成 visual caption 与 audio caption，再把两条 caption + 原视频喂回同一模型合成时序连贯的多模态 caption —— 「先分解再合成」的两阶段设计显著优于一次性联合生成。
· 三份并列合成（AVSCap-130K）：每条视频产出 visual / audio / synergistic 三份标注，供不同训练目标使用。
· 自举式合成：video-SALMONN 2 用自身产出的高质量 caption 语料再做后续模型的 SFT。
· 偏好数据合成：Tarsier2 用 model-based sampling 自动构造偏好对做 DPO；AVoCaDO/AVSCap 的 GRPO reward 本身即在线合成偏好信号。
【合成训练样本（受控扰动/编辑构造训练对）】
· SkyCaptioner-V1 的相机运动子专家使用 1.6 万条「运动轴均衡合成数据」补足长尾运镜类型 —— 这是为纠正标注器自身的类别不均衡而做的受控合成，是本生态最明确的合成数据用例。
· InstructAV2AV 用 Qwen3-Omni 生成编辑指令 + Grounded-SAM-2 的 mask-guided 合成构造「源-指令-目标」三元组（InsAVE-80K），属于 captioner 被用于合成数据构造而非视频描述。
· Movie Gen 的 cross-paired 合成数据（PT2V）：略微降低 ArcFace 身份相似度，但换来显著更多样的头部姿态与更自然的表情。
【合成数据的方法论隐患】InstructAV2AV 用同一个 Qwen3-Omni 既生成指令又做五维度验证打分，论文自身承认「生成与验收同源是本 pipeline 的方法论隐患」；AVSCap 既是 benchmark 作者又是榜首，同源过拟合风险需用第三方基准交叉验证。

### 人工介入程度（人工标注、人工质检、模型初筛+人工复核）

人工介入在 captioner 生态中集中于三个位置，且呈「少量高价值人工 + 大规模模型推理」的一致格局：
【位置一 · 种子数据人工精标（成本最高、价值最大）】
· SkyCaptioner-V1 的相机运动子专家：9.3 万条高置信度人工标注 + 1.5 万条人工测试集。
· ALIVE：caption 模型的训练数据为「MLLM 生成 → 人工修订（manually revised caption data）→ 两轮 SFT」的经典自建路径，用一次性人工成本换大规模推理的低成本；另有 sub-motion units 的起止时间戳人工标注。
· Movie Gen：后训练阶段的 caption 由人工在模型输出基础上精修。
· Step-Video-T2V：SFT 阶段由人工「优化 caption」，是不做 captioner 幻觉抑制后训练时的兜底手段。
【位置二 · 打标器选型与评测中的人工研究】
· Panda-70M 用 1000 条 clip 的用户研究做贪心集合覆盖选出 8 个打标器；同时报告人类之间的一致率仅 44.9% —— 这个数字定义了整个 caption 质量评测的天花板，是本生态最重要的人工评测发现。
· Movie Gen 的 caption 方案对比全部靠人工 A/B：视频 caption 被偏好 67% vs 逐帧改写方案 15%。
· AVSCapBench 的 1226 条视频为人工标注；UGC-VideoCap 采用三阶段 human-in-the-loop（分别标 audio-only / visual-only / joint AV）。
· Open-Sora Plan 的阈值合理性经 2000 条人工抽检验证。
【位置三 · 人工被 LLM-judge 替代的趋势】2025–2026 年，caption 质量的逐条人工复核几乎被 GPT-4.1 / Gemini 打分完全替代（AVoCaDO 的 completeness 打分、AVSCap 的 automated verification、AuroraCap 的 VDCScore），人工只保留在基准构建与最终 A/B 环节。这一替代的代价尚未被系统评估 —— 考虑到人类一致率仅 44.9%，LLM-judge 与人类判断的偏差可能已被「人类自己也不一致」所掩盖。

## 音视频对齐

### 音视频同步检测方法（口型同步、事件对齐） ⚠️

captioner 生态与音视频同步检测的关系是「间接但关键」——captioner 不直接做同步检测，但它是同步质量的语义验收方：
【captioner 承担的同步职能】
· AVSCap 的第三准则 Audio-Visual Synergy 明确要求 caption 显式绑定视听事件并做时序对齐 —— 这是把「同步性」从信号级指标提升到语义级描述的尝试。AVSCapBench 的 Synergy 维度即在量化这一能力：AVSCap-7B 57.70、Gemini-3-Pro 48.88、AVoCaDO 29.13、video-SALMONN-2 12.43、Qwen2.5-Omni 裸基座 7.00。
· AVoCaDO 的第五维 Cross-modal Narrative Logic（视听互相解释/补充/引导）是同一思路的早期形态，但其 Synergy 得分仅 29.13，说明 checklist 里有这一维不等于模型真学会了。
· Ovi 明确要求 caption 覆盖所有相关视觉与听觉事件并「遵守时间顺序（respecting chronology）」，为此做了多轮 prompt 迭代实验。
【信号级同步检测由专用工具承担（在 captioner 上游/下游）】SyncNet、Synchformer 是主流：Foley-Omni 用 Synchformer 同时作为训练特征提供者与过滤阶段的同步性打分器（一模型两用，保证过滤标准与模型学习目标一致）；MOVA 用 LSE-D/LSE-C；SkyReels-V4 用 SyncNet。这些与 caption 生成解耦。
【口型同步的说话人归属问题】TalkNet（InstructAV2AV）做主动说话人检测以确定「谁在说」，CineDance 用 Qwen3-Omni + 滑窗 prompt 把 ASR-角色绑定从 67.2% 提升到 95.4% —— 说话人归属是 caption 侧最接近同步检测的任务。
[不确定] 无任何 captioner 工作把同步性检测结果作为 caption 字段直接输出（如「音画不同步 0.3 秒」），同步信息只以「事件绑定」的语义形式隐含在 caption 中。

### 同步检测具体指标与阈值（SyncNet/Synchformer/LSE/自研，阈值数值，如MOVA LSE-D≤9.5且LSE-C≥4.5、SkyReels-V4 SyncNet |offset|≤3∧conf>1.5） ⚠️

[不确定] captioner 本身不设同步指标阈值，本字段记录生态中与 caption 相关的可比阈值：
【caption 质量阈值（本生态自有的阈值）】
· AVoCaDO：GPT-4.1 对 synthesis completeness 打 1–5 分，保留阈值 ≥4；对白 content 相似度用编辑距离 + 动态规划对齐，阈值 0.6；caption 长度上限 4096 token（超出由 ℛ_L 惩罚），最优区间 2048–4096 token。
· Foley-Omni：Bandit 声学后验证的能量门控阈值 −35 dB（用于纠正 Gemini 标注中的视觉幻觉，即画面看到但实际无声的对象）。
· Panda-70M：UMT 检索择优的难负例权重设置（未选中的 7 条 caption 权重 1.0，batch 内其他负例权重 0.01）。
【下游生成模型的同步阈值（供对照，非 captioner 阈值）】MOVA 要求 LSE-D ≤ 9.5 且 LSE-C ≥ 4.5；SkyReels-V4 要求 SyncNet |offset| ≤ 3 且 conf > 1.5；Foley-Omni 用 Sync 分 ≥ 0.2、IB ≥ 0.3、AudioBox ≥ 0.6；OmniCustom 用 SyncNet 双阈值。这些均在 captioner 之外执行。
【生态缺口】没有任何工作发布「caption 与音视频同步性的联合阈值表」，即不存在「当同步性低于某阈值时该样本的 caption 应如何处理（丢弃/降级/标记）」的公开规程 —— 这是 AV 数据 pipeline 中一个明显的方法论空白。

### 时序同步vs语义同步的分离处理（时间对齐与内容语义匹配作为两个独立过滤条件）

本生态对「时序同步」与「语义匹配」的分离处理有明确的方法论演进：
【分离的三层证据】
(1) AVSCap 的评测维度设计本身即是分离：Visual / Speech / Music / SFX 四项衡量**语义完整性**（该说的内容有没有说到），Synergy 单独衡量**跨模态绑定与时序对齐**。数据显示两者可以严重脱节 —— AVoCaDO 的 Speech 高达 70.42（语义完整）但 Synergy 仅 29.13（绑定失败），证明「听得清」不等于「对得上」。
(2) Foley-Omni 的双特征通路：CLIP 提供场景语义特征、Synchformer 提供时序同步特征，两路独立注入。消融显示移除 Z_sync 同步通路后 IB_V2ST 从 0.26 降至 0.22、FD_VGG 从 1.57 升至 2.21（劣化 41%）、WER_V2ST 从 7.59 升至 12.40 —— 定量证明时序特征通路对语义一致性也有实质贡献，二者并非完全正交。
(3) MOVA 的过滤设计把「跨模态语义一致性检查」（GPT-OSS-120B）与「唇同步信号检测」（LSE-D/LSE-C 阈值）作为两个独立条件串联。
【caption 侧的具体分工】语义匹配靠 captioner 与 LLM 裁判（是否描述了正确的声音来源、是否有画外音幻觉）；时序对齐靠 Synchformer/SyncNet 信号级检测与 ASR 时间戳（ElevenLabs Scribe、CineDance 的句级 ASR）。
【一个反例警示】Foley-Omni 用 Bandit 做声学后验证的动机正是：Gemini 依据画面生成的音频描述可能描述了实际不存在的声音（视觉幻觉），这是纯语义判断无法自我纠正的错误，必须引入信号级验证 —— 说明语义与时序/信号两条链路互为兜底，不可偏废。

### 音频质量过滤（SNR、静音检测与静音占比阈值、无音轨剔除、画外音源剔除、背景音乐分离） ⚠️

captioner 生态中的音频质量处理主要体现为「打标前的分流闸门」与「打标后的声学验证」：
【打标前分流】UniVerse-1 在离线漏斗中用 Whisper 判定片段是否含语音，作为分流闸门决定走哪条标注路径；JavisDiT 系列的视频阶段用 FunASR 剔除含语音片段（而音频阶段明确「不做任何过滤以最大化 T2A 能力覆盖三类音频」——「音频不过滤、视频严过滤」的反差是团队对两种模态质量-数量权衡的实践判断）；LTX-2 的数据子集筛选标准为「包含显著且信息量大的音频成分」，以剔除静音与无信息音轨。
【SNR / 音频质量打分】Movie Gen 使用专门的音频质量预测模型（输出 1–10 分）；Foley-Omni 用 AudioBox 阈值 0.6；InstructAV2AV 用 Audiobox 阈值。这些均在 captioner 上游。
【背景音乐分离与声源分离】Bandit（Foley-Omni，按 speech/effects/music 三路分离，与三字段 schema 同构）；Mel-RoFormer（Unison，训练侧把混合音频解耦为语音与音效两路 ground-truth latent，评测侧做 WER 前的人声分离）；SAM-Audio（InstructAV2AV，按语义把目标实体的声音从混合音轨分离）。
【画外音源剔除】Foley-Omni 的 −35 dB 能量门控本质上是在剔除「画面中有但音轨里没有」的幻觉标注，反向操作则可识别画外音（音轨里有但画面中没有）。Movie Gen 的评测维度中「画内音正确性（on-screen sound correctness）」是唯一显式量化这一问题的指标，其 SFT 相对 PT 提升 +31.0±16.0。
[不确定] 没有任何 captioner 工作公开其训练数据的静音占比阈值或 SNR 阈值。

### 语音/音效/音乐的分类与分别处理策略

语音 / 音效 / 音乐三类音频的分类与分别处理，是 AV captioner 生态最成熟的设计模式：
【双模型分工（最普遍）】MOVA：Qwen3-Omni-Instruct 负责语音转写、Qwen3-Omni-Captioner 负责非语音声音与音乐描述，论文强调该分工「全面覆盖语言内容与声学特征，减少信息损失」。UniTalking：Whisper-V3 转写 + Qwen3-Omni-Captioner 描述声学环境 + Qwen3-Omni 做融合。SkyReels-V4：Qwen3-Omni 统一生成音频 caption，Whisper 转写语音与歌唱。
【三字段固定 schema】Foley-Omni 的 speech / effects / music 三字段与 Bandit 的分离输出同构；AVSCap 的 Acoustic Completeness 准则同样按 Speech / SFX / Music 三分。
【四模型协同（Movie Gen，最精细）】音频质量预测模型（1–10 分）+ AED 模型（判定 voice/singing 存在性与 music 后验概率）+ 通用音频 caption 模型（自由形式描述声音）+ 音乐 caption 模型（补充 mood 与 genre）。论文特别说明音乐 caption 模型主要在音乐样本上训练、无音乐时容易幻觉，因此**同时保留 AED 的音乐概率与音乐 caption 两路信号**，实测这种冗余组合可控性最好 —— 这是全生态关于「音频类型幻觉」的最有价值工程经验。
【单模型通吃（激进路线）】UniVerse-1 用单个 Qwen2.5-Omni 一次性并列输出三路；Ovi 用同一个 MLLM 处理音视频语料与纯音频语料；CineDance 用 Qwen3-Omni-30B-A3B 同时做句级 ASR、镜头级音频 prompt（音乐/环境音/音效）、角色音色描述三项子任务。
【能力现状（AVSCapBench 量化）】三类音频的描述能力差距极大且高度一致：Speech（40–80 分）≫ Music（4–40 分）> SFX（7–32 分）。SFX 是全行业最弱环节，最好的开源模型 AVSCap-7B 仅 30.82，Gemini-3-Flash 32.34 —— 这直接限制了 Foley 与环境音生成模型的训练数据上限，是 AV captioner 生态最亟待突破的方向。

## 训练配合

### 多阶段训练课程与数据课程调度（阶段划分依据：分辨率/时长/质量分/模态；低清→高清、图像→视频、短→长）

【captioner 自身的训练课程】
· AuroraCap：三阶段训练，累计超 2000 万条图/视频-文本对（图像→视频的经典课程）。
· Tarsier2：预训练（40M video-text pairs）→ 细粒度时序对齐 SFT → model-based sampling 构造偏好对 + DPO，三阶段。
· AVoCaDO / AVSCap：SFT → GRPO 两阶段。AVSCap 明确论断「RL 带来的增益大于扩大 SFT 数据量」，这是 caption 模型训练资源分配的关键结论。
· video-SALMONN 2：SFT → MrDPO（多轮 DPO，周期性用新训的轻量 adapter 重置 reference policy，避免 reference 陈旧）。
· Omni-Captioner（阿里）：两阶段 curriculum，audio → audio-visual（先学听、再学看+听）—— 与视觉侧「图像→视频」课程正好互补，是 omni captioner 特有的课程设计。
· SkyCaptioner-V1：Qwen2.5-VL-72B 通用描述 + 三子专家 → 融合 → 蒸馏进 7B，属于「多教师并行 → 单学生蒸馏」而非串行课程。
【captioner 在生成模型训练课程中的角色（分级打标）】
· Open-Sora 2.0：256px 阶段海量数据用 LLaVA-Video 打标、768px 阶段精选 5M 用 Qwen 2.5 Max 重标 —— 打标质量金字塔与数据金字塔严格对应。
· Movie Gen：70% caption 来自 8B、30% 来自 70B，是同一权衡的另一实现；后训练阶段 caption 由人工精修。
· Open-Sora Plan v1.3：阶段二用未过滤 Panda70M（约 100k 步接近收敛），阶段三换成过滤后数据（仅 30k 步、lr 降至 1e-5）实现质量跃升 —— 「大量粗数据学分布、少量精数据提质量」的间接证据。
· Motif-Video 2B：caption 三变体按 0.5/0.3/0.2 概率采样，第 1–3 阶段用 sentence-level embedding 文本编码器、第 4 阶段起换为 T5Gemma2（借鉴 PixArt-α 的「类条件到文本条件」课程思路）。
【生态结论】打标质量随训练阶段升级是 2025–2026 年的稳定实践，且「便宜模型标底层、贵模型标顶层」的成本结构在多家独立收敛。

### 数据配比随训练阶段的变化（预训练/退火annealing/SFT高质量子集） ⚠️

【captioner 训练的数据配比】
· AVoCaDO-SFT-107K 的六源配比即为显式的 mixture 设计：FineVideo 29K（27%）> TikTok-10M 24K（22%）> Shot2Story 20K（19%）> ShortVideo 18K（17%）> YouTube-Commons 11K（10%）> CinePile 5K（5%），以 UGC 为主体、影视为少量高质量补充。
· AVSCap-130K = 4 万视频 × 3 份标注（visual / audio / synergistic），三类标注 1:1:1 配比而非偏向融合式，说明单模态锚定标注对训练同样重要。
· SkyCaptioner-V1：1000 万条精选出 200 万条概念均衡数据（保留率 20%），配比目标是概念均衡而非源均衡。
【caption 在生成模型各阶段的配比变化】
· Motif-Video 2B：caption 三变体 0.5/0.3/0.2 的采样概率（论文坦承是「务实的配方选择而非经隔离验证的最优性主张」）。
· UniTalking：「分头标注后拼接」与「统一模型融合标注」两条路径的产物在训练时被随机采样使用 —— 把标注策略的选择交给随机采样，让模型同时适应两种文本分布。
· SkyReels-V4：短/长/结构化三档描述并存供混训。
· Movie Gen：8B/70B caption 的 70/30 混比贯穿全训练集，非阶段性变化。
· NAVA（JAVG 批次）：模态配比从 3:1 调度到 1:2，160K SFT 子集，但无对照实验验证收益。
【生态缺口】几乎没有工作报告「不同 caption 风格/长度的混合比例」的消融最优点，Motif 的坦诚说明（0.5/0.3/0.2 未经验证）代表了行业真实状态。[不确定]

### 后训练数据（SFT精选集规模与筛选标准、偏好对数量与标注方式、reward model训练数据） ⚠️

【captioner 的后训练数据（本生态 2025–2026 年的核心竞争点）】
· AVoCaDO 的 GRPO：reward 由三项构成 —— ℛ_C（五维 checklist 覆盖度，GPT-4.1 判定）、ℛ_D（对白 F1，speaker 准确率 + content 编辑距离 DP 对齐相似度，阈值 0.6）、ℛ_L（长度正则，>4096 token 惩罚）。偏好信号在线生成，无固定规模的偏好对数据集。
· AVSCap 的 GRPO：hybrid reward = 长度控制 + speech preservation（语音保真）+ audio-visual consistency。论文关键论断「RL 增益 > 扩 SFT 数据量」，是后训练投入优先于标注量扩张的直接依据。
· Tarsier2 的 DPO：用 model-based sampling 自动构造偏好数据（无需人工标注偏好对），规模未披露[不确定]。
· video-SALMONN 2 的 MrDPO：多轮 DPO，周期性用新训的轻量 adapter 重置 reference policy；caption 质量目标同时奖励完整性（completeness）与事实性（factual accuracy），7B 模型 caption 错误率相对基线降 28%。
· video-SALMONN-o1 的 pDPO：过程级 DPO，用对比式 step 选择做 step-level reward，配套 RivaBench（4000+ 专家标注 QA，覆盖脱口秀、学术演讲、合成视频检测）。
· 腾讯混元 1.5 的 caption 模型：用 OPA-DPO（针对多模态幻觉的偏好优化）做 RL 后训练以抑制幻觉 —— 是生成侧团队唯一公开 captioner 后训练方法的案例。
【SFT 精选集的筛选标准】AVoCaDO 只保留 GPT-4.1 打分 ≥4/5 的样本；SkyCaptioner 从 1000 万精选 200 万；ALIVE 的 caption 数据为 MLLM 生成后人工修订再做两轮 SFT。
【生态趋势】2025 年之前 captioner 后训练几乎只做 SFT，2025 下半年起 DPO/GRPO 成为标配，且 reward 设计（尤其对白准确率与跨模态一致性）取代数据量成为主要竞争维度。

### 数据处理基础设施与吞吐（NeMo Curator/Data-Juicer/自研，GPU加速比，处理规模，成本） ⚠️

[部分不确定] 该维度披露稀少，但吞吐考量深刻影响了打标器选型：
【明确的算力数字】
· SkyCaptioner-V1：32 张 A800、全局 batch size 512、约 200 万条训练数据。
· Panda-70M 的 UMT 择优模型：8×A100-80G，12 帧 224×224，AdamW lr 2e-5，batch 32，10 epoch。
· Open-Sora Plan v1.3 的 prompt refiner：LLaMA-3.1-8B-Instruct LoRA（rank 64，batch 32，1 epoch），单张 H100 上 30 分钟训完。
· MOVA：打标使用 NVIDIA GPU 与华为昇腾 Ascend NPU 混合执行 —— 少见的国产芯片参与大规模打标的公开记录。
· Motif-Video 2B：PaddleOCR-VL 经 vLLM 服务化；CineDance 推理框架致谢 vLLM。vLLM 是打标批量推理的事实标准。
【吞吐决定选型的直接证据（本生态最有价值的工程记录）】
· Open-Sora 官方文档：「GPT-4V 效果更好，但 20 秒/样本的速度对我们太慢」—— 因此改用 PLLaVA-13B。这是开源项目在打标成本上最坦诚的权衡记录。
· Allegro 选 Aria 的关键理由是官方称可在 10 秒内为 256 帧视频生成 caption。
· 全生态参数量收敛到 7B（ShareCaptioner、Tarsier2、SkyCaptioner、AVoCaDO、AVSCap、MiMo-VL）本身就是吞吐约束的产物；把 120B 级模型（GPT-OSS-120B）只放在融合裁决环节而非逐样本感知环节，同理。
· UniVerse-1 的在线标注架构强制要求轻量模型（Qwen2.5-Omni + Whisper），无法承受 120B 级模型的逐样本推理开销 —— 「标注时机」与「标注模型容量」的耦合。
· MAGI-1 把大 MLLM 的增强 prompt 蒸馏到约 7B 小模型（约 200 万条语料）以降低推理延迟，人评显示质量相当而延迟与算力开销大幅下降。
【成本】无任何工作公开打标的美元成本或 GPU 小时数[不确定]。Apollo/Klear 在 8100 万条样本上调用 Gemini-2.5-Pro 是已知最大规模的商用 API 打标投入，但金额未披露。
【专用框架】未见 captioner 生态使用 NeMo Curator / Data-Juicer 等数据工程框架的公开记录；打标环节普遍是自研脚本 + vLLM 批量推理。

## 效果对比

### 数据策略消融的量化影响（区分：过滤严格度ablation / caption密度风格ablation / 数据配比ablation，及对应评测指标） ⚠️

本字段汇总全生态关于「caption 质量/密度/风格如何影响下游生成」的量化证据，这是 captioner 生态存在价值的最终检验：

【最强证据 · Movie Gen 的 caption 形态 A/B（人工净胜率 win%−lose%）】原生视频 captioner（LLaMa3-Video 8B）vs 抽帧改写方案（对首/中/尾三帧做图像 caption 再用 LLaMa 改写合并）：
· caption 本身质量：人工 A/B 中视频 caption 被偏好 67%，逐帧改写方案仅 15%；
· 对生成模型的影响：整体 prompt 对齐 +10.8%，其中绝大部分增益来自运动对齐 +10.7%，在要求高运动的 prompt 上更是 +16.1%。
结论：原生视频 captioner 能准确描述更细粒度的运动细节，为训练提供更强的监督信号 —— 这是「为什么必须用视频 captioner 而非图像 captioner 拼接」的唯一定量答案。

【结构化 caption 的收益 · Koala-36M】引入「指标条件（metric conditions）」结构化 caption 后，VBench 语义得分从 0.4504 提升到 0.5915（+14.1 个百分点）；用 Koala-all 训练相较 Panda-70M 训练，VBench 主体一致性 +1.1%、背景一致性 +2.4%（归因于更准的镜头切分）。VTSS 模型过滤优于人工多指标硬阈值组合。

【蒸馏收益的完整量化 · Script-a-Video（本生态最完整的 caption 蒸馏 ablation）】Qwen3-Omni 原生 → 零样本 MTSS 提示 → MTSS 微调（教师为 Gemini-2.5-Pro）：
· video-SALMONN-2 总错误率：0.5853 → 0.5156 → 0.3913（教师原生 0.3959，即微调后的开源模型已略优于教师）；
· UGC-VideoCap 综合分：62.80 → 71.54 → 85.11（教师 93.97）；
· Daily-Omni：0.1806 → 0.4117 → 0.5945（教师 0.6825）；
· WorldSense：0.1569 → 0.3106 → 0.3875（教师 0.4332）。
关键发现：MTSS 结构化提示对所有测试模型都有效（包括未微调的），是零成本增益。

【打标器多样性的必要性 · Panda-70M】单个最好的打标器仅覆盖 30.8% 样本，31 个全上覆盖 84.7%，贪心选出的 8 个覆盖 76.8%；UMT 择优模型微调后 R@1 达 35.90%（预训练 UMT 仅 21.82%）；人类之间的 caption 偏好一致率仅 44.9%。

【音视频 caption 的横向量化 · AVSCapBench】详见 audio_category_distribution 的完整表格。核心结论：AVSCap-7B overall 60.44 ≈ Gemini-3-Pro 60.97，Synergy 反超（57.70 vs 48.88），Speech 落后 10 点；裸 Qwen2.5-Omni 仅 21.53。

【caption 模型自身能力的 ablation · Tarsier2】预训练数据 11M→40M + 细粒度时序对齐 SFT + DPO 三项升级，最终 DREAM-1K F1 超 GPT-4o 2.8%、超 Gemini-1.5-Pro 5.8%，人评 +8.6% / +24.9%，首个 DREAM-1K overall recall 突破 40%。AVSCap 的论断「RL 增益 > 扩 SFT 数据量」与之互补。

【定性但重要的结论】
· CogVideoX：主张「创新的视频打标模型显著提升了生成质量与语义对齐」，附录 H 用 Panda-70M 短 caption 与 CogVLM2-Caption 长 caption 对照展示密度差异，但未给 A/B 净胜率[量化缺失]。
· Goku：用 GPT-4o 扩写 GenEval 短 prompt 后分数达 0.76，间接证明模型性能高度依赖与训练期密集 caption 一致的长 prompt（未组织为正式 caption ablation）。
· Open-Sora Plan：明确论证训练 caption（密集长文）与用户 prompt（<10 词）分布错配会损害生成效果，据此构建 prompt refiner，但无开/关对照指标。
· Open-Sora 2.0：高清阶段把 caption 模型从 LLaVA-Video 换成 Qwen2.5-Max，理由是「更准确、语义对齐更好」，无对照实验。

【全生态最大缺口】绝大多数生成模型技术报告（HunyuanVideo、Allegro、LTX-2、LongCat-Video、MAGI-1、Mochi、Apollo、Foley-Omni、InstructAV2AV、CineDance）明确没有 caption 密度/风格 ablation。Movie Gen、Koala-36M、Script-a-Video 是仅有的三个提供扎实定量证据的工作 —— 这意味着行业对「caption 该多长、该多结构化」的选择基本靠直觉与成本，而非实证。[不确定]

### 质量vs数量的证据（小而精数据超越大而杂的案例）

【captioner 生态内部的质量胜量证据】
· ShareGPT4Video 是最经典的样本：仅用 40K 条 GPT-4V 密集 caption 做 SFT，就训出能标注 480 万条视频的 ShareCaptioner-Video —— 极小的高质量种子集撬动百倍规模的产出。
· AVSCap 的核心论断：「RL 带来的增益大于扩大 SFT 数据量」，即在 130K 数据规模下，投入 reward 设计比继续扩标注量更划算。
· Script-a-Video：500K 条 Gemini-2.5-Pro 精标数据微调后，8B/30B 级开源模型在 video-SALMONN-2 总错误率上（0.3913）已略优于教师 Gemini-2.5-Pro 原生输出（0.3959）。
· SkyCaptioner-V1：仅 200 万条概念均衡数据训出的 7B 模型，其影视专业字段准确率（平均 76.3%，镜头类型 93.7%）显著超过 Qwen2.5-VL-72B 等大 10 倍的通用模型 —— 「小模型 + 对口数据 > 大模型 + 通用数据」的直接证据。
【下游数据集层面的质量胜量证据】
· Koala-36M（3600 万条）vs Panda-70M（7000 万条）：规模仅一半，但 VBench 总分最高，主体一致性 +1.1%、背景一致性 +2.4%。核心差异在 caption 长度（202.3 词 vs 13.2 词）与切分准确度 —— 这是本生态最直接的「小而精超越大而杂」案例。
· CineDance-1M 伪影不合规率 2.8% vs Koala-36M 37.4%（13.4× 改善），代表更新一代对质量的进一步收紧。
· JavisDiT++ 的表述值得注意：不是简单的「质量优于数量」，而是「确保良好的数据质量是增加样本数量以提升训练效果的前提基础」——即质量不达标时单纯扩量无效甚至有害。
· Movie Gen 音频 SFT vs PT：整体 +41.7±15.3、专业度 +43.0±14.7，论文评语「凸显了微调阶段高质量数据 curation 的重要性」。
【反向警示】CogVideoX 报告了双向效果：去除字幕水印后视觉质量小幅提升，但语义能力轻微退化 —— 过度清洗会损失语义多样性，是「质量优先」的边界条件。

### 训练数据domain分布与评测基准类目体系的对齐关系（如VABench七大类）

【caption 专用基准的类目体系】
· VDC（AuroraCap 配套）：结构化 caption 分 camera / short / background / main object / detailed 五字段，配 VDCScore（分治式转化为多个短问答对由 LLM 判定）。这五字段实际上定义了后续大量工作的 caption schema。
· AVSCapBench（2026-07，最新最细）：1226 条人工标注视频（30–120 秒，来自 YouTube / TikTok / Video-MME），按 Visual / Speech / Music / SFX / Synergy 五维做细粒度 event recall，设计上防止「靠单模态刷分」。这五维与 AV 生成模型的训练数据 domain 需求高度对应。
· Omni-Cloze（阿里，2026-03）：2000 clip / 7 万道细粒度完形填空题 / 9 大领域 47 子类，用完形填空规避 LLM-judge 噪声。是类目体系最完整的公开基准。
· UGC-VideoCap（2025-07）：1000 条 TikTok 视频 + 4000 QA，三阶段 human-in-the-loop 分别标注 audio-only / visual-only / joint AV 语义 —— 类目按「模态可得性」划分而非内容主题。
· OmniCap-IF（2026-06）：首个音视频 caption 指令跟随基准，50 类约束覆盖纯视觉/纯音频/音视频，含 Temporal Grounding，双维度评分（format correctness + content correctness）。这一维度对生产 pipeline 至关重要 —— 打标器必须能按你的 schema 输出。
· DREAM-1K（Tarsier 配套）：以动态动作的 event recall 为核心，Tarsier2 是首个 overall recall 突破 40% 的模型。
· VidCapBench（2025-02）：明确面向可控 T2V 的视频 caption 评测，是少数把「caption 质量」与「生成可控性」显式挂钩的基准。
· video-SALMONN 2 testset：以 error rate（missing + hallucination）为导向，与前述 recall 导向基准互补。
【与生成侧评测体系的对齐关系】
· caption 基准的维度（camera / background / main object / motion）与 VBench 的评测维度（主体一致性、背景一致性、运动平滑度、语义得分）存在明显对应，Koala-36M 的结构化 caption 使 VBench 语义得分 +14.1 个百分点即是这一对应的直接体现。
· AVSCapBench 的 Speech / Music / SFX 三分与 VABench 等 AV 评测基准的音频类目、以及 Foley-Omni 的三字段 schema 完全同构 —— 说明「打标 schema、评测类目、生成能力维度」三者正在收敛为同一套本体。
【方法论警示】AVSCap 既是 AVSCapBench 作者又是榜首（60.44），benchmark-model 同源存在过拟合风险；论文发布仅两周（2026-07-14），无第三方复现，选型时须用 UGC-VideoCap、Omni-Cloze 等第三方基准交叉验证。Panda-70M 报告的人类一致率仅 44.9%，为所有 caption 基准的可靠性设定了天花板。

## 不确定字段

以下字段的调研信息部分不确定（⚠️ 标注来源）：

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
