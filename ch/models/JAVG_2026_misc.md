# 2026 其他音视频联合生成（JAVG）工作集锦 —— 七项工作合并调研：(1) Baton《Baton: Explicit Semantic Blueprints for Joint Video-Audio Generation》arXiv:2605.25195；(2) OmniCustom《OmniCustom: Sync Audio-Video Customization Via Joint Audio-Video Generation Model》arXiv:2602.12304（含其自建数据集 OmniCustom-1M）；(3) StreamChar《StreamChar: Long-Horizon Streaming Character Audio-Video Generation with Decoupled Orchestration》arXiv:2605.25659；(4) ALIVE《ALIVE: Animate Your World with Lifelike Audio-Video Generation》arXiv:2602.08682（含 Alive-Bench 1.0）；(5) CCL《Improving Joint Audio-Video Generation with Cross-Modal Context Learning》arXiv:2603.18600；(6) NAVA《Native Audio-Visual Alignment for Generation》arXiv:2605.30073；(7) ITS-JAVG《Inference-Time Scaling for Joint Audio-Video Generation》arXiv:2606.03183。本条目只提取数据处理相关内容，七项工作在数据披露深度上差异极大：ALIVE 与 NAVA 属工业级重披露，OmniCustom 属数据集构建型中等披露，Baton/CCL/StreamChar 仅披露数据来源与规模，ITS-JAVG 为完全 training-free 无训练数据。

> 主题: 视频生成模型（含音视频同时生成）的数据处理：数据清洗流程、数据分布、打标方式

[← 返回首页](../index.md)

**目录**: [基本信息](#基本信息) · [数据规模与分布](#数据规模与分布) · [清洗流程](#清洗流程) · [打标方式](#打标方式) · [音视频对齐](#音视频对齐) · [训练配合](#训练配合) · [效果对比](#效果对比)

## 基本信息

### 名称

2026 其他音视频联合生成（JAVG）工作集锦 —— 七项工作合并调研：(1) Baton《Baton: Explicit Semantic Blueprints for Joint Video-Audio Generation》arXiv:2605.25195；(2) OmniCustom《OmniCustom: Sync Audio-Video Customization Via Joint Audio-Video Generation Model》arXiv:2602.12304（含其自建数据集 OmniCustom-1M）；(3) StreamChar《StreamChar: Long-Horizon Streaming Character Audio-Video Generation with Decoupled Orchestration》arXiv:2605.25659；(4) ALIVE《ALIVE: Animate Your World with Lifelike Audio-Video Generation》arXiv:2602.08682（含 Alive-Bench 1.0）；(5) CCL《Improving Joint Audio-Video Generation with Cross-Modal Context Learning》arXiv:2603.18600；(6) NAVA《Native Audio-Visual Alignment for Generation》arXiv:2605.30073；(7) ITS-JAVG《Inference-Time Scaling for Joint Audio-Video Generation》arXiv:2606.03183。本条目只提取数据处理相关内容，七项工作在数据披露深度上差异极大：ALIVE 与 NAVA 属工业级重披露，OmniCustom 属数据集构建型中等披露，Baton/CCL/StreamChar 仅披露数据来源与规模，ITS-JAVG 为完全 training-free 无训练数据。

### 发布机构/公司 ⚠️

七项工作分属不同机构：
(1) Baton：复旦大学（Shuyuan Tu、Zuxuan Wu、Yu-Gang Jiang，视觉与学习实验室）× 腾讯混元（Tencent Hunyuan，作者含 Weijie Kong、Jiangfeng Xiong、Zhao Zhong 等混元视频生成团队成员）联合，另有 Liefeng Bo（阿里通义/达摩院背景）与 Qi Tian、Xintong Han 参与[机构归属为按作者名单推断，不确定]。
(2) OmniCustom：香港大学（The University of Hong Kong，Guosheng Yin、Dong Xu）主导，联合 Shanda AI Research Tokyo（盛大 AI 研究院东京）、XIntelligence Technology Co., Limited，以及上海人工智能实验室（Kaipeng Zhang）；作者 Maomao Li、Zhifeng Li 有腾讯 AI Lab 背景。
(3) StreamChar：阿里巴巴通义实验室（Tongyi Lab, Alibaba Group），作者 Linrui Tian、Qi Wang、Bang Zhang，即 EMO / Wan-S2V 系列数字人团队，项目页托管于 HumanAIGC 组织。
(4) ALIVE：字节跳动 ALIVE 团队（Bytedance ALIVE Team），16 位作者，含 Xiang Yin（Seed 语音方向）、Bingyue Peng、Zehuan Yuan 等；仓库位于 FoundationVision 组织下。
(5) CCL：商汤科技（SenseTime）系，作者 Bingqi Ma、Guanglu Song、Yu Liu、Dailan He 等[机构为按作者推断，论文页未显式列出，不确定]。
(6) NAVA：百度 ERNIE Research（文心研究团队），作者 Longbin Ji、Guan Wang、Xuan Wei、Shuohuan Wang、Yu Sun 等，权重同时托管于 HuggingFace 的 baidu 与 ernie-research 组织。
(7) ITS-JAVG：KAIST（韩国科学技术院），作者 Jaemin Jung、Kyeongha Rho、Inkyu Shin、Joon Son Chung（Joon Son Chung 即 SyncNet 作者之一，音视频同步领域权威）。

### 发布时间（技术报告/论文/开源时间）

均为 2026 年上半年 arXiv 首发：
- ALIVE：2026年2月9日 v1，2月10日 v2（七项中最早）。
- OmniCustom：2026年2月12日 v1，持续修订至 2026年7月23日 v5（迭代最久）。
- CCL：2026年3月19日（arXiv 编号 2603.18600）。
- Baton：2026年5月24日 v1，2026年6月1日 v2。
- StreamChar：2026年5月25日。
- NAVA：2026年5月28日，随后开源代码与权重。
- ITS-JAVG：2026年6月2日（编号 2606.03183）。
时间上呈现清晰的季度节奏：2 月为大厂基座模型（ALIVE）与定制化任务（OmniCustom）先行，3 月出现「低成本高效路线」（CCL），5—6 月集中出现语义规划（Baton）、流式长时（StreamChar）、原生对齐（NAVA）与推理时缩放（ITS-JAVG）四条新分支，反映 JAVG 领域在 2026 年上半年从「能不能联合生成」转向「如何更省、更长、更准、更可控」。

### 类型（模型/数据集/工具链/评测基准）

七项均为学术论文形态，但性质分四类：
【基座模型类】ALIVE（VideoDiT 12B + AudioDiT 2B 的完整 T2VA/R2VA 基座模型，附 Alive-Bench 1.0 评测基准）、NAVA（6.3B 原生对齐联合生成模型，附开源权重与训练代码）。
【模块/方法类（在已有基座上做增强）】Baton（在联合 DiT 前加 VA-Planner 语义规划器 + RS-RoPE，属方法增强）、CCL（针对双流 Transformer 范式的四个模块级改进 TARP/LCT/DCR/UCG，主打省数据省算力）。
【任务+数据集类】OmniCustom（提出「同步音视频定制」新任务，核心贡献之一是自建数据集 OmniCustom-1M，并配 100 例评测集）。
【系统/工程类】StreamChar（长时流式生成系统，含两阶段蒸馏与实时推理管线）。
【推理时算法类】ITS-JAVG（完全 training-free 的推理时缩放算法 + 多验证器组合研究 + ARW 优化算法，无任何模型训练，本质上是「用评测模型做推理时数据筛选」）。
从数据调研角度：ALIVE、NAVA、OmniCustom 三者含实质数据处理 pipeline 披露；Baton、CCL、StreamChar 仅有数据来源清单；ITS-JAVG 无训练数据但其「验证器组合」思想与数据质检打分器高度同构。

### 开源程度（权重/代码/数据/pipeline各自是否开源） ⚠️

开放程度差异极大，从完全开源到仅有技术报告：
【最开放｜NAVA】Apache 2.0 源码许可。已发布：完整推理管线、训练代码（training code）、Gradio 交互 demo、模型权重 NAVA.safetensors（24GB bf16）与量化版 NAVA_fp8.safetensors（约 7GB）。依赖组件复用 Wan2.2-5B VAE、T5 编码器、LTX audio-VAE、ReDimNet 说话人嵌入器（各自遵循原许可）。未开源：训练数据本身与数据处理 pipeline 代码（仓库明确提到训练数据但无数据管线释出）。
【中等｜OmniCustom】提供 GitHub 仓库与项目主页，代码可得；其自建 OmniCustom-1M 派生自公开数据集 SpeakerVid-5M，因此数据构建过程可复现（筛选规则全部公开），但过滤后清单是否释出未明[不确定]。
【中等｜StreamChar】有项目主页 humanaigc.github.io/StreamChar_page/，训练数据全部来自三个公开数据集（SpeakerVid-5M / TalkVid / OpenHumanVid）+ Emilia，因此数据侧可复现度高；权重与代码释出情况未明[不确定]。
【偏封闭｜ALIVE】GitHub 仓库 FoundationVision/Alive 目前仅为技术报告发布页（含 assets 目录、arXiv 链接、项目页、Discord demo 入口），未见权重下载、训练代码、推理代码或数据释出；Alive-Bench 1.0（264 general prompts + 90 reference-character prompts）已在论文中定义但公开释出状态未明[不确定]。是七项中数据披露最详细、代码开放度最低的一项。
【封闭｜Baton / CCL】仅论文；训练数据含 in-house / 互联网自采部分，未开源；CCL 的 in-house 数据（访谈、短剧、电影）明确不可得。
【N/A｜ITS-JAVG】training-free，无权重需开源；论文称「Project materials and code are available online」，代码与 prompt 集应可得[不确定：具体仓库地址未验证]。所依赖的验证器（VideoReward、JavisScore、ImageBind、VQAScore、AVHScore）与基座生成器（JavisDiT、MMDisCo、LTX-2）均为已开源资产，可复现性最高。

### 是否支持音视频同时生成，以及实现方式（原生联合/级联/MoE融合）

七项全部围绕音视频同时生成，但实现路径分五种，恰好构成 2026 年 JAVG 技术路线的一个横切面：
(1) 【原生联合 + 语义前置规划】Baton：在联合 DiT 去噪之前，先由 VA-Planner（以 Qwen3-8B 为底的多模态 LLM，带 dual semantic alignment towers）生成一批「planned tokens」作为音视频共享的语义蓝图，再用 Relative Semantic RoPE（RS-RoPE）把蓝图注入去噪过程。其核心论点是「现有方法依赖 off-the-shelf 文本编码器的粗粒度 embedding，丢失细粒度语义且缺乏共享长时规划」。规划器不预测离散 token，而是回归连续特征（L2 回归到冻结 SigLip2 视频特征与 WavTokenizer 音频特征的倒数第二层），理由是「regressing continuous features preserves richer semantic structure」。
(2) 【原生联合 + 定制化 LoRA】OmniCustom：在已有联合音视频生成基座上，用两组独立 LoRA（reference identity LoRA 与 audio timbre LoRA）分别作用于 self-attention 层，实现「给一张参考图 + 一段参考音频，生成保持该身份、模仿该音色、说出 prompt 指定台词的同步音视频」。额外引入对比学习目标（有参考条件的预测流为正例、无参考条件的为负例）与 flow matching 并行。
(3) 【流式自回归 + 解耦编排】StreamChar：把「长时编排」与「短窗去噪」解耦——LLM 编排器（Orchestrator）读取台词全文与历史上下文，产出逐帧对齐的音频条件；联合音视频 DiT 在局部窗口内做双向去噪，配 reference frame 与 motion frame 条件。每 chunk 输出 33 帧 @24fps，历史音频上下文上限 15 秒，配 progress-aware pointer（PAP，用 ASR 时间戳的 ground-truth end indices 做 smooth L1 监督）与 persistent visual anchor（sink chunk）抑制长程漂移。单张 H100 实时，可连续生成 5 分钟流。
(4) 【双塔原生联合（大厂基座）】ALIVE：由预训练 T2V 模型改造而来的统一音视频合成模型，VideoDiT 12B + AudioDiT 2B，支持 Text-to-Video&Audio 与 Reference-to-Video&Audio（动画化）。关键架构件为 TA-CrossAttn（时序对齐的跨模态融合）与 UniTemp-RoPE（统一时序 RoPE 做精确音画对齐）。480p 基座 + 1080p refiner 两级。
(5) 【双流 Transformer 范式的修补】CCL：不改范式，专门诊断并修复双流 Transformer 的三类缺陷——门控机制引起的模型流形漂移、跨模态注意力在多模态背景区域引入的偏置、多模态 CFG 训练与推理不一致。对应四个模块：TARP（时序对齐 RoPE 与分区）、LCT + DCR（稳定的无条件锚点与跨模态信息的动态路由）、UCG（Unconditional Context Guidance，推理一致性）。最大卖点是用远少于同类的数据与算力达到 SOTA。
(6) 【Align-then-Fuse 原生对齐】NAVA：明确反对两种既有设计——双塔（dual-tower）削弱细粒度同步、统一三模态（unified tri-modal）把语义对齐与低层对齐混为一谈。改为 Align-then-Fuse MMDiT：先在专用对齐空间建立音视频对应关系，再把上下文（文本、说话人嵌入）作为外部条件通过 cross-attention 融入共享去噪。附 Timbre-in-Context Conditioning，把参考音色线索绑定到对应的语音片段（speech spans），支持多说话人参考音色控制。6.3B 参数，原生立体声输出，支持 T2AV / I2AV / T2A。
(7) 【推理时缩放（不改模型）】ITS-JAVG：把单模态领域的 Inference-Time Scaling 迁到联合音视频生成，完全 training-free。核心发现是单一验证器必然导致 verifier hacking 与指标间的非对称 trade-off，必须用多验证器组合；并提出 Adaptive Reward Weighting（ARW），把奖励聚合当作在线优化问题。可视为在推理阶段复刻了训练数据 pipeline 中的「多打分器联合过滤」思想。

### 调研信息来源列表（论文/技术报告/官方文档/新闻的URL，标注每条来源的性质：官方一手/同团队旁证/第三方报道）

【官方一手 · 论文】
1) Baton https://arxiv.org/abs/2605.25195 ／ https://arxiv.org/html/2605.25195v2 （数据信息集中在 Implementation Details）
2) OmniCustom https://arxiv.org/abs/2602.12304 ／ https://arxiv.org/html/2602.12304 （OmniCustom-1M 数据集构建章节为本条目最有价值的数据披露之一）
3) StreamChar https://arxiv.org/abs/2605.25659 ／ https://arxiv.org/html/2605.25659v1
4) ALIVE https://arxiv.org/abs/2602.08682 ／ https://arxiv.org/html/2602.08682v2 （Data 章节六阶段 pipeline，七项中数据披露最详尽）
5) CCL https://arxiv.org/abs/2603.18600 ／ https://arxiv.org/html/2603.18600v1 （Table 1 含训练数据量横向对比）
6) NAVA https://arxiv.org/abs/2605.30073 ／ https://arxiv.org/pdf/2605.30073 （Data 章节含 20M/100M→15M 漏斗与算子清单）
7) ITS-JAVG https://arxiv.org/abs/2606.03183 ／ https://arxiv.org/html/2606.03183v1
【官方一手 · 项目页/仓库】
8) NAVA 项目页 https://ernie-research.github.io/NAVA/ ；代码 https://github.com/ernie-research/NAVA ；权重 https://huggingface.co/baidu/NAVA 与 https://huggingface.co/ernie-research/NAVA （开源范围、许可、分辨率/时长、语言支持）
9) ALIVE 仓库 https://github.com/FoundationVision/Alive ；项目页 foundationvision.github.io（模型规模 VideoDiT 12B / AudioDiT 2B、Alive-Bench 1.0）
10) StreamChar 项目页 https://humanaigc.github.io/StreamChar_page/
11) OmniCustom 项目页与 GitHub（论文中给出）
【同团队旁证 / 聚合页】
12) HuggingFace Papers：https://huggingface.co/papers/2605.30073 （NAVA）、https://huggingface.co/papers/2605.25659 （StreamChar）
13) ResearchGate CCL 条目 https://www.researchgate.net/publication/402860562_Improving_Joint_Audio-Video_Generation_with_Cross-Modal_Context_Learning
【第三方】
14) ChatPaper StreamChar 解读 https://chatpaper.com/paper/286175 ；Takara TLDR CCL https://tldr.takara.ai/p/2603.18600 ；X/Twitter CS Vision Papers 对 NAVA 的推介（均为论文摘要复述，无新增一手信息）
【上游数据集一手来源（被这些工作复用）】
15) SpeakerVid-5M（OmniCustom 与 StreamChar 的共同上游）、TalkVid、OpenHumanVid（Baton/CCL/StreamChar 三方共用）、Koala-36M（NAVA 约占 20%）、AudioCaps、WavCaps、VGGSound、Emilia（StreamChar 语音预训练）

## 数据规模与分布

### 训练数据量级（视频条数/小时数/token数，预训练与SFT分开） ⚠️

各工作规模跨度超过两个数量级，且披露口径不一：
【ALIVE（字节，规模最大且分阶段披露最细）】按训练阶段给出样本数而非小时数：
- 音频 T2A Stage I：384M audio samples（对应 700k hours of transcribed speech，用于 TTS 预训练）
- 音频 T2A Stage II：19M samples（约 5k hours 高质量语音 + 111k hours 带声音标注的视频数据集）
- T2VA 联合训练：11M samples @480p/24fps（1.2 epoch）
- T2VA+I2VA：同一 11M samples（0.3 epoch）
- Continue-training：4.3M balanced samples（3 epoch）
- SFT：5M samples（0.5 epoch）
- 1080p Refiner：0.7M high-clarity samples（1 epoch）
- Character-driven：0.8M reference-paired samples
论文自述为「continue pretraining and finetuning on million-level high-quality data」。原始候选池总量未披露[不确定]。
【NAVA（百度）】唯一给出完整漏斗两端的工作：原始采集「approximately 20M audio clips and 100M video clips」，过滤后「around 15M clips for large-scale training」用于大规模训练；其中 Koala-36M 子集约占最终语料 20%。SFT 阶段进一步收敛到 160K high-quality samples。平均视频时长约 7 秒（按 15M×7s 估算约 2.9 万小时[此换算为推断，不确定]）。
【CCL（商汤系）】主打小数据：Table 1 明确列出自身训练数据量 4M，并与 Ovi 的 30.7M、MOVA 的 50M 并列对比——即用 Ovi 约 1/8、MOVA 约 1/12 的数据量。正文描述为「million-level audio-video pairs」。
【Baton】1.5 million video-audio clips，原文「Our training dataset (1.5 million video-audio clips) is aggregated from OpenHuman-Vid, AudioCaps, WavCaps, and videos collected from the internet.」——七项中训练规模最小的联合生成工作之一。
【OmniCustom】自建 OmniCustom-1M：约 1 million 单人视频片段，合计 2,500 hours，从 SpeakerVid-5M（原始「more than 5.2 million video clips」/「8,000 hours」）中筛得。即片段级保留率约 19%、时长级保留率约 31%（2500/8000）。
【StreamChar】未给总量数字，只给来源（SpeakerVid-5M + TalkVid + OpenHumanVid 组合）与训练步数（编排器 80k steps @batch 640；联合训练 100k steps @batch 128）。语音预训练用 Emilia 数据集。明确约束「training data contains no videos/transcripts longer than 20 seconds」[总规模不确定]。
【ITS-JAVG】无训练数据（training-free）；「数据量」体现在推理搜索预算：JavisDiT 每 prompt 采 5 samples（Best-of-N），MMDisCo 每 prompt 采 10 samples。

### 数据来源构成（自有/公开数据集/网络爬取/授权采购/合成数据） ⚠️

呈现明显的「公开数据集复用 + 自有/爬取补充」两段式，且 OpenHumanVid 与 SpeakerVid-5M 成为 2026 年 JAVG 领域的两个事实标准上游：
【Baton】OpenHuman-Vid + AudioCaps + WavCaps + 互联网自采视频（internet-collected videos）四路混合，无授权采购与合成数据描述。
【OmniCustom】单一上游：SpeakerVid-5M（公开的音视频双人交互人体生成数据集），经自建规则筛出 OmniCustom-1M。评测集另含 70 段 YouTube 视频 + 30 位未出现在训练集的真人。
【StreamChar】三个公开人体/说话视频数据集拼合：SpeakerVid-5M（大规模高质量音视双人交互人体生成）、TalkVid（大规模多样化音频驱动说话头合成）、OpenHumanVid（大规模高质量以人为中心视频生成）；语音侧用 Emilia 大规模多语种语音数据集。全部为公开数据，是七项中数据来源最「学术可复现」的一项。
【ALIVE】未点名具体数据集，描述为「raw data pool」中先筛出「videos with audio」的自有大规模语料；音频侧有 700k hours 转写语音（疑为字节内部 Seed 语音语料[不确定]）、5k hours 高质量语音、111k hours 带声音标注的视频数据集。另有大量合成/增强构造（见 synthetic_data_synthesis）。整体为内部语料主导。
【CCL】OpenHumanVid（公开）+ in-house collections，后者明确覆盖「interviews, short dramas, and films」（访谈、短剧、电影）三类——这是一个很具体的来源画像，说明其数据偏「有人说话的叙事内容」；音频预训练额外引入学术数据集 WavCaps 与 VGGSound。
【NAVA】三路构成：Koala-36M（公开大规模视频数据集，约占最终语料 20%）+ TED-style speech videos（TED 风格演讲视频，为高质量单人语音源）+ raw movie/TV footage（影视原片）。是七项中唯一显式说明「影视原片」作为来源的工作。
【ITS-JAVG】无训练数据；评测用 VGGSound test set 与 JavisBench-mini。
【共性观察】七项中无一披露授权采购数据、无一使用 C2PA 类溯源，且四项（Baton/CCL/NAVA/ALIVE）都含互联网自采或影视原片，合规风险集中于此。

### 数据合规与溯源（授权数据占比、rights-cleared数据集、C2PA等） ⚠️

[不确定]——七项工作全部未讨论数据授权比例、rights-cleared 数据集、C2PA 水印溯源、版权合规审查流程。可推断的间接事实：
(1) 依赖公开学术数据集的工作（OmniCustom 全量依赖 SpeakerVid-5M；StreamChar 依赖 SpeakerVid-5M/TalkVid/OpenHumanVid/Emilia；Baton 与 CCL 部分依赖 OpenHumanVid/AudioCaps/WavCaps/VGGSound；NAVA 依赖 Koala-36M）实际上把合规责任外包给了上游数据集——而这些上游多基于 YouTube 抓取，本身带有来源争议。
(2) 明确含高风险来源的：NAVA 的「raw movie/TV footage」（影视原片，版权敏感度最高）、CCL 的「short dramas and films」（短剧与电影）、Baton 的「videos collected from the internet」。三者均无任何版权说明。
(3) 隐私侧：OmniCustom 与 StreamChar 属人脸+音色定制任务，直接涉及生物特征（人脸身份 + 声纹音色）。OmniCustom 评测集刻意使用「30 persons who were not included in training data」以验证零样本泛化，但未提及肖像权/声音权授权；ALIVE 的 character-driven pipeline 大量使用 ArcFace 人脸嵌入做身份匹配，同样无隐私声明。这是本批工作在数据合规上的共同空白。
(4) 许可层面唯一明确的是 NAVA：源码 Apache 2.0，但仓库明确声明「模型权重、预训练骨干、tokenizer、audio VAE、说话人编码器、prompt 改写模型可能受各自原始提供方的不同许可约束」——即许可覆盖代码而非数据。

### 片段时长分布与切分策略 ⚠️

各工作的时长策略高度分化，反映其任务定位：
【定长短片｜OmniCustom】严格定长设计，且与「参考-训练配对」耦合：先过滤掉「videos shorter than 10 seconds」，再从每段 ≥10 秒的片段中取「the first 4 seconds as the reference audio」，「the last 5 seconds ... designated as both the training audio and video clips」——即一条 10 秒素材同时产出 4 秒参考音频与 5 秒训练样本，二者音色相同但语音内容不同（「each reference-training pair shares the same timbre but contains distinct speech content」）。这是一个非常干净的「同音色异内容」配对构造法。
【短片段+长源｜ALIVE】Character-driven 数据从「long videos (10–30 minutes)」中抽取 N 个「3–10 seconds」的片段；生成输出为 5 至 10 秒。身份锚点则取「1.5-second sub-clip with maximum sync score」——即用 1.5 秒的同步分最高子片段作为身份代表帧源。训练数据整体的时长分布未披露[不确定]。
【平均 7 秒｜NAVA】明确「The average video duration is about 7 seconds」，是七项中唯一给出平均时长的工作。推理侧默认 37 帧 @24fps（约 1.5 秒）可配置。
【流式分块｜StreamChar】时长策略最特殊——训练侧受限于「training data contains no videos/transcripts longer than 20 seconds」（训练数据中无超过 20 秒的视频/台词），推理侧却要生成 5 分钟连续流，靠 chunk 拼接实现（每 chunk 33 帧 @24fps ≈ 1.375 秒），历史音频上下文窗口上限 15 秒。这构成一个显著的「训练短、推理长」的泛化跨度，也是其 progress-aware pointer 与 persistent visual anchor 两个抗漂移机制存在的根本原因。评测中「150 clips generating 10s audio-video pairs」与「50 clips paired with randomly sampled transcripts (>300 words) to produce 5-minute continuous streams」。
【固定短时｜ITS-JAVG（评测侧）】所测基座各自固定：JavisDiT 生成 4 秒视频、MMDisCo 生成 2 秒视频。
【未披露｜Baton、CCL】Baton 的 Sem100 评测集为「100 unseen videos (10 seconds long)」，训练片段时长未说明[不确定]；CCL 完全未提时长分布[不确定]。
【切分策略】仅 NAVA 提到「raw videos are first segmented at scale with a Hadoop-based pipeline」（大规模 Hadoop 管线切分），其余均未描述切分策略[不确定]。

### 分辨率/宽高比分布与分桶策略 ⚠️

七项中仅少数披露，且普遍采用「低清训练 + 高清精修」的两级策略而非分桶：
【ALIVE】明确两级：基座在 480p / 24fps 上用 11M 样本训练，另设独立的 1080p Refiner 阶段用 0.7M「high-clarity samples」训练 1 epoch。这是一个典型的「大规模低清打底 + 小规模高清提纯」课程。宽高比分布与分桶策略未披露[不确定]。此外 ALIVE 有专门的 clarity filtering（清晰度过滤）阶段，用「reference images across six distinct clarity levels」（六档清晰度参考图，从模糊到锐利）作为评判基准，本质上是把分辨率/清晰度做成了六级序数标签而非连续分数——这一做法在同期工作中较少见。
【OmniCustom】完全统一归一化，不做分桶：「All videos are recorded in 480p at 24 FPS. We extract audio files from videos and unify them into 16kHZ.」——视频统一 480p/24fps，音频统一 16kHz。这与其单人说话头的窄任务定位一致。
【CCL】训练分辨率 352×640p（非标准分辨率，接近 9:16 竖屏），是七项中唯一给出精确像素尺寸的工作；单一分辨率训练，无分桶。
【NAVA】训练侧分辨率分布未披露[不确定]；推理侧默认 704×1280（9:16 竖屏），仓库称「flexible aspect ratios supported」，说明具备多宽高比能力，但训练侧的配比未说明。
【ITS-JAVG（评测基座）】JavisDiT 为 240p，MMDisCo 为 256×256——反映学术界开源 JAVG 基座的分辨率仍显著落后工业界。
【Baton、StreamChar】未披露分辨率/宽高比策略[不确定]。
【共性】七项中无一采用 Ovi 式的「等面积归一化」或 Seedance 式的多分辨率分桶调度，普遍是单一分辨率训练 + 可选高清精修，说明这批工作的算力预算与工程复杂度都偏节制。

### 类别/domain分布与配比策略（人物、动作、场景、风格等比例控制与概念均衡） ⚠️

本维度上 ALIVE 与 NAVA 各建立了一套显式的类目体系，是七项中最有价值的部分；其余五项基本无配比控制。
【ALIVE —— 三级标签体系 + 主动配比调整（本批工作中最完整的 domain 控制设计）】
(1) 顶层二分：先做「speaking（说话）」与「non-speaking（非说话）」两大核心场景的划分——原文「First, we make a top-level distinction between core scenarios: 'speaking' and 'non-speaking' scenario.」这一二分直接决定了模型在「对白驱动」与「音效驱动」两类能力上的配比。
(2) 三级层次标签：在此之上建立 three-level hierarchy，Level 1 为九大领域：Animals（动物）、Home Sounds（家居声）、Entertainment（娱乐）、Environment（环境）、Food（食物）、Nature（自然）、Sound Effects（音效）、Vehicles（载具）、Sports（运动）。注意这九类是「视听联合类目」而非纯视觉类目——Home Sounds 与 Sound Effects 本身就是声学类目，说明该体系从设计之初就是为音视频联合建模服务的。
(3) 音视关键词绑定：每个 Level 3 视觉标签都通过「joint audio-visual keyword retrieval」（联合音视关键词检索）配上对应的音频关键词，形成视觉概念与声音概念的成对索引——这是把 domain 分布与 audio 分布做联合而非独立控制的关键机制。
(4) 主动配比调整：原文「Guided by prior knowledge, such as concept frequency and projected application scenarios, we then adjust the data proportions for each category.」——依据概念频次与预期应用场景两条先验主动调整各类目占比。Continue-training 阶段用的 4.3M「balanced samples」即为该配比调整的产物。具体百分比未公开[不确定]。
【NAVA —— 五类音频驱动的内容分类】用 YAMNet 音频分类 + omni-modal tagger 把片段划为五类：single-speaker speech（单说话人语音）、multi-speaker speech（多说话人语音）、ambient sound（环境音）、music（音乐）、singing（歌唱）。注意其分类轴是声学的而非视觉的，与 ALIVE 的视听双轴不同。各类占比未公开[不确定]。此外还有 VLM-based filtering and tagging 保留视觉质量清晰的片段，标签体系未详述。
【CCL —— 隐式 domain 画像】未做显式配比，但其数据来源「interviews, short dramas, and films」本身构成了一个窄而深的 domain：以人为中心、以对白为主的叙事内容。这解释了它为何能用 4M 数据达到 SOTA——牺牲广度换深度。其多任务训练概率分布（见 stage_data_mixture）中「联合生成 0.6」的高占比也说明其目标 domain 高度聚焦。
【OmniCustom —— 单一 domain】完全聚焦「单人说话视频（single-person video clips）」，无 domain 多样性诉求，其 100 万片段全部为同一类目。评测集刻意控制性别比为「1:1」——这是七项中唯一显式的人口统计学配比控制。
【StreamChar —— 单一 domain】以人为中心的角色说话视频，三个上游数据集均为 human-centric，无跨 domain 配比。
【Baton】未提及任何 domain 配比策略；其数据来源（OpenHuman-Vid 人体 + AudioCaps/WavCaps 通用音频 + 互联网视频）的混合比例未公开[不确定]。
【ITS-JAVG】无训练数据；但其评测基准 JavisBench-mini 本身带有多类目结构（继承自 JavisBench 的场景/事件分类体系），用于检验各验证器在不同类目上的表现差异[类目细节不确定]。

### 音频类别分布与配比（语音/音效foley/音乐/环境音/静音的比例及控制策略）——AV模型独有维度 ⚠️

这是 AV 模型独有维度，本批工作中 ALIVE 与 NAVA 有实质设计，其余偏弱：
【ALIVE —— 分阶段的语音/音效两段式 + BGM 比例主动管控】
(1) 音频塔的两阶段配比切换极为鲜明：T2A Stage I 为纯语音预训练，用 700k hours of transcribed speech（384M samples，1 epoch，lr 5e-5），此阶段几乎全部是语音；T2A Stage II 切换为混合，「approx. 5k hours」高质量语音 + 「111k hours」带声音标注的视频数据（19M samples，10 epochs）——即从「70 万小时纯语音」骤降到「5 千小时精选语音」并大量掺入视频原生音轨。这是一个「大规模语音打底 → 小规模高质语音保活 + 大规模真实音景补充」的清晰调度。注意 Stage II 的 epoch 数（10）远高于 Stage I（1），说明高质量混合数据被反复利用。
(2) BGM 比例主动管控：原文「we assess the correlation and separate samples where the audio is highly correlated with the visual content, while also managing the proportion of weakly correlated data, such as background music (BGM), to optimize the dataset's composition.」——把音频按「与画面的相关性强弱」二分：强相关（diegetic，画内音）样本单独归集，弱相关（如 BGM，非画内音）样本则控制其占比而非全部剔除。这是一个比「一刀切剔除 BGM」更细腻的策略：保留一定比例弱相关音频可让模型学会生成配乐，但比例过高会破坏音画因果关系。具体比例未公开[不确定]。
(3) caption 层面的三类分流：<W> 标记语音（verbatim speech content），<I> 标记非语音声学事件（non-speech acoustic events），加上 Subjects 字段中的 acoustic profiles（声学画像），构成语音/音效/音色三条标注轨。
(4) 九大 Level-1 领域中 Home Sounds 与 Sound Effects 两类直接对应非语音音频类目，且每个 Level-3 视觉标签都绑定音频关键词——音频类别配比被嵌入到 domain 配比体系中统一调度。
【NAVA —— 五类音频标签驱动的语料组织】用 YAMNet 音频分类器配合 omni-modal tagger 将全部片段打上五类标签：single-speaker speech / multi-speaker speech / ambient sound / music / singing。这套五分法比常见的「语音/音效/音乐」三分更细，特别是把单人语音与多人语音分开（对应其多说话人 timbre 控制能力）、把 music 与 singing 分开（歌唱同时含语音与音乐属性，单独成类合理）。训练阶段的音频配比通过「audio-only : audio-visual」比例调度体现：Stage 1 为 3:1（音频侧主导，先把音频生成能力练强），Stage 2 反转为 1:2（音视频侧主导）。各音频类别的绝对占比未公开[不确定]。
【CCL】音频预训练引入 WavCaps 与 VGGSound 两个音效/通用音频数据集，联合训练数据（OpenHumanVid + 访谈/短剧/电影）则以对白语音为主——构成「通用音频预训练 + 语音为主联合训练」的结构。比例未公开[不确定]。
【Baton】数据源中 AudioCaps 与 WavCaps 提供通用音效/音景描述，OpenHuman-Vid 提供人声，混合比例未公开[不确定]。
【OmniCustom】音频几乎全部为单人语音（人声音色定制任务），无音效/音乐维度；音频统一重采样至 16kHz。属于音频类别最单一的一项。
【StreamChar】以角色语音为绝对主体，Emilia 为纯语音数据集；无音效/音乐类别设计[不确定]。
【ITS-JAVG】无训练数据；但其验证器组合覆盖了音频语义（ImageBind-TA、AVHScore）与同步（JavisScore）两类，评测基座之一 MMDisCo 基于 VGGSound（以音效为主），另一 JavisDiT 覆盖更广类目。

### 叙事结构分布（单镜头vs多镜头、平均clip时长、镜头数分布、是否含原生音轨） ⚠️

【单镜头为主，长时叙事仅 StreamChar 一家涉及】
(1) 单镜头短片：OmniCustom（5 秒定长单人说话，单镜头）、CCL（352×640 短片段）、Baton（短片段，评测为 10 秒）、NAVA（平均 7 秒，经 Hadoop 管线切分，应为单镜头片段）、ALIVE（3–10 秒片段，输出 5–10 秒）均为单镜头结构，无镜头切换与转场样本。
(2) 长源→短片的抽取：ALIVE 的 character-driven pipeline 明确从「10–30 分钟长视频」中抽取 N 个 3–10 秒片段——即长视频被当作「同一身份的多镜头素材池」使用，用于构造跨片段的身份配对（cross-pair），这实际上间接引入了「同一角色跨镜头一致性」的监督信号，虽然单个训练样本仍是单镜头。
(3) 长时叙事的唯一探索者：StreamChar 是本批中唯一正面处理长时序结构的工作，但其解法不是用长视频数据训练——恰恰相反，训练数据被限制在 20 秒以内（「training data contains no videos/transcripts longer than 20 seconds」），长时能力完全通过推理时的 chunk 自回归拼接（每 chunk 33 帧 @24fps）+ 两个显式抗漂移机制获得：progress-aware pointer（用 ASR 时间戳的 ground-truth end indices 做 smooth L1 监督，让模型知道台词念到哪了）与 persistent visual anchor / sink chunk（持久视觉锚点抑制身份漂移）。消融显示去掉 sink chunk 后 drift 从 0.0067 恶化到 0.0304（劣化约 4.5 倍），量化证明了视觉锚点对长程一致性的关键作用。评测中生成 5 分钟连续流（用 >300 words 的随机台词）。
(4) 原生音轨占比：七项全部使用带原生同步音轨的配对数据——ALIVE 明确「begins by filtering videos with audio from the raw data pool」（第一步就是从原始池中筛出带音轨的视频），OmniCustom/StreamChar 的上游数据集本身即音视频配对，NAVA 的音视频子集同理。无一采用「先剥离音轨再后期配音」的路线。
(5) 镜头数分布、平均镜头数等统计七项均未披露[不确定]。

### 语言/口音分布（多语种唇同步能力的数据基础） ⚠️

本批工作在语言维度上普遍薄弱，是共同短板：
【ALIVE】训练数据的语言分布未披露[不确定]；仅在评测环节提到评测 prompt 覆盖「multiple languages (Chinese and English)」（中英双语）——考虑到字节跳动的产品定位与 700k hours 转写语音语料，中英为主可推断但无正面证据[不确定]。caption 中 <W> 标签内为 verbatim speech content（逐字语音内容），说明转写保留原语言。
【NAVA】仓库明确「English speech generation supported」，并注明「limited other languages' support」（其他语言支持有限）——即以英语为主、多语能力弱。这与其数据来源含大量 TED-style speech videos（TED 演讲绝大多数为英语）一致。15M 语料的语种分布未公开[不确定]。架构上引入 ReDimNet 说话人嵌入器做音色控制，音色与语言解耦，但语言覆盖仍受数据限制。
【StreamChar】用 Emilia 做编排器语音预训练（80k steps）——Emilia 本身是大规模多语种语音数据集（含英/中/德/法/日/韩六语），这在原理上为多语种能力提供了基础，但论文未报告多语种评测，实际覆盖不确定[不确定]。
【OmniCustom】用 GLM-ASR 为每段 5 秒训练音频生成转写（「we used GLM-ASR to generate transcriptions for each 5s training audio clip」）——GLM-ASR 为中文优势模型，暗示语料含相当比例中文[推断，不确定]。caption 中显式标注 accent（口音）属性（沿用 Ovi 的属性集）。评测集控制性别 1:1，但未控制语言比例。
【CCL / Baton】未提及语言分布[不确定]。CCL 的 in-house 数据（访谈、短剧、电影）与 OpenHumanVid 混合，语种构成不明；CCL 报告 WER 指标说明其评估了语音可懂度，但语言未明。
【ITS-JAVG】无训练数据；评测基准 VGGSound 与 JavisBench-mini 以英语环境为主。
【总体判断】七项中无一给出定量的语种/口音分布表，无一报告多语种唇同步的分语种评测。OmniCustom 是唯一把 accent 作为显式标注字段的工作，NAVA 是唯一诚实声明「多语支持有限」的工作。

## 清洗流程

### 清洗漏斗整体结构（几级过滤、各级顺序） ⚠️

七项的数据管线复杂度差异极大，可排成一条从「无管线」到「六阶段工业管线」的谱系：

【ALIVE —— 六阶段工业级管线（本批最完整）】
前置：先从 raw data pool 中筛出「videos with audio」（带音轨的视频），无音轨样本直接出局。
原文：「our pipeline begins by filtering videos with audio from the raw data pool and then proceeds through six core stages: video quality pre-processing, captioning, audio quality filtering, SubjectID correction, clarity filtering, and data balancing.」
第1阶段 Video quality pre-processing（视频质量预处理）：OCR 测量叠加文字占比并检测水印 → 预训练美学模型逐帧打分 → RAFT 计算光流做运动分析 → 质量模型把样本判为 high-quality 或 13 个 low-quality 维度之一。
第2阶段 Captioning（打标）：两轮 SFT 训练的 caption 模型 + 人工标注 sub-motion units 时间戳 → 产出 Subjects / Visual / Narration 三段式结构化 caption，语音用 <W>、非语音声学事件用 <I> 内联标记。
第3阶段 Audio quality filtering（音频质量过滤）：MLLM 双准则打分——音频质量分（剔除显著背景噪声）+ 音画相关性（分离强相关样本，管控 BGM 类弱相关样本占比）。
第4阶段 SubjectID correction（说话人-主体归属校正，五步子管线，本批最独特的设计）：① TS-TalkNet 做主动说话人检测，生成人脸-声音对应矩阵；② Qwen3-omni 解析旁白时间戳；③ majority voting 做「句子→TrackID」映射；④ CLIP / ArcFace 嵌入做视觉-语义匹配；⑤ 用校正后的标签重写文本。未通过高置信阈值的视频被过滤，得到一个「simplified 但高质量」的初始数据集。这一阶段解决的是多人场景中「哪句话是谁说的」的错误归属问题——这是纯视频模型完全不存在、而 AV 模型必须面对的独有难题。
第5阶段 Clarity filtering（清晰度过滤）：用六档清晰度参考图（从模糊到锐利）做评判。
第6阶段 Data balancing（数据均衡）：speaking / non-speaking 顶层二分 → 九大 Level-1 领域三级标签 → 按概念频次与预期应用场景调整各类占比。
另有并行的 Character-driven data pipeline（cross-pair + in-pair augmentation，见 synthetic_data_synthesis）。

【NAVA —— 大规模多算子管线（漏斗两端公开）】
原始池 ≈ 20M audio clips + 100M video clips → 最终约 15M clips。
① Hadoop 管线大规模切分（「raw videos are first segmented at scale with a Hadoop-based pipeline」）
② PaddleOCR 做 OCR 过滤与字幕擦除（subtitle removal，注意是「擦除」而非仅「剔除」）
③ VideoCLIP 提取视频嵌入 + 大规模 k-means 聚类去除冗余/近重复片段
④ 多算子质量过滤：视觉侧（aesthetics 美学 / sharpness 锐度 / brightness 亮度 / motion score 运动分）、音频侧（AudioBox Aesthetic 分数）、音视对齐侧（SyncNet / SyncFormer / ImageBind 三个对齐算子）
⑤ VLM-based filtering and tagging，保留视觉质量清晰的片段
⑥ YAMNet 音频分类 + omni-modal tagger 做五类音频打标
⑦ 打标：Qwen3-VL 生成视频 caption、Qwen3-Omni 生成音频 caption，再由 Gemini-3-Flash 直接拼接或改写融合；高质量子集改用 Gemini-3-Pro
⑧ 多算子协同过滤（multi-operator collaborative filtering）产出 160K 高质量 SFT 子集

【OmniCustom —— 三条规则的轻量管线】从 SpeakerVid-5M（5.2M clips / 8,000 hours）出发：① SyncNet 双阈值（|offset|≤3 且 confidence>1.5）；② 美学分 <0.3 剔除；③ 时长 <10 秒剔除 → 得 OmniCustom-1M（约 1M clips / 2,500 hours）。之后统一格式（480p/24fps、音频 16kHz）→ GLM-ASR 转写 → 参考-训练配对切分（前 4 秒作参考音频、后 5 秒作训练音视频）→ 随机采样含人脸的帧并裁剪作参考图。

【StreamChar —— 数据集拼合，无自建管线】直接组合 SpeakerVid-5M + TalkVid + OpenHumanVid，加约束「无超过 20 秒的视频/台词」；ASR 时间戳用于 progress-aware pointer 的监督标签。无显式质量/同步过滤描述[不确定]。

【CCL / Baton —— 仅有来源清单，无管线披露】CCL 为 OpenHumanVid + in-house（访谈/短剧/电影）+ WavCaps/VGGSound；Baton 为 OpenHuman-Vid + AudioCaps + WavCaps + 互联网视频。二者均未描述任何过滤、打标或去重步骤[不确定]。

【ITS-JAVG —— 把管线搬到推理时】无训练数据管线，但其结构与数据过滤管线同构：每 prompt 生成 N 个候选（JavisDiT 5 个、MMDisCo 10 个）→ 用一组验证器（VideoReward-TA / JavisScore / ImageBind-TA / AVHScore / VQAScore / ImageBind）打分 → ARW 自适应加权聚合 → EvoSearch 进化式选优。可以理解为「把训练数据管线中的多打分器联合过滤，前移到了生成之后、输出之前」。

### 漏斗定量保留率（每级过滤的输入/输出量与最终保留率，如Apollo 27%） ⚠️

本批工作中仅两项给出可算的保留率，其余全为空白：
【OmniCustom —— 唯一可精确计算的漏斗】
- 输入：SpeakerVid-5M，「more than 5.2 million video clips」/「8,000 hours」
- 输出：OmniCustom-1M，「约 1 million single-person video clips」/「2,500 hours」
- 片段级保留率 ≈ 1.0M / 5.2M ≈ 19.2%
- 时长级保留率 = 2,500 / 8,000 = 31.25%
- 两个保留率不一致的含义值得注意：时长保留率显著高于片段保留率，说明被保留下来的片段平均更长（保留片段平均 2500h/1M ≈ 9.0 秒 vs 原始 8000h/5.2M ≈ 5.5 秒），这正是「剔除 <10 秒视频」这条规则的直接后果——该规则在片段数上砍得很狠，但砍掉的都是短片段。
- 三条规则各自的通过率未分开给出[不确定]。
【NAVA —— 只有漏斗两端，无中间层】
- 输入：「approximately 20M audio clips and 100M video clips」
- 输出：「around 15M clips for large-scale training」
- 若以视频侧计，保留率约 15M/100M = 15%；若合并计则更低。逐级（OCR/去重/质量/对齐）的输入输出量与通过率均未披露[不确定]。
- 二次收敛：15M → 160K 高质量 SFT 子集，SFT 保留率约 1.07%（即千分之十）——这个数字很有参考价值，说明「可训练」与「可做 SFT 的精品」之间存在两个数量级的落差。
【ALIVE】完全未给保留率或原始池规模，只能从阶段样本数反推数据被逐级收窄的趋势：11M（联合训练）→ 4.3M（balanced，均衡后仅剩 39%）→ 5M（SFT）→ 0.7M（1080p 高清子集，占 11M 的 6.4%）→ 0.8M（角色配对）。其中「11M → 4.3M balanced」这一步的 39% 保留率是数据均衡（domain 配比调整）造成的削减，是本批中除 OmniCustom 外唯一可推算的比例[属推断，不确定]。
【Baton / CCL / StreamChar】无任何漏斗信息[不确定]。CCL 的 4M 是最终使用量，原始候选池未知。
【ITS-JAVG】推理时「保留率」概念对应 Best-of-N 的选择率：JavisDiT 为 1/5 = 20%，MMDisCo 为 1/10 = 10%——巧合的是，这与工业数据管线常见的 10%–20% 保留率量级相当，侧面说明「过采样再择优」在训练侧与推理侧遵循相似的经济学。

### 镜头切分方法（PySceneDetect/自研模型/shot-aware splitting） ⚠️

披露极少，是本批工作的共同薄弱环节：
【NAVA —— 唯一提及切分工程】「raw videos are first segmented at scale with a Hadoop-based pipeline」——采用基于 Hadoop 的分布式管线做大规模切分。值得注意的是论文只说明了工程框架（Hadoop）而未点名切分算法或工具（未说明是 PySceneDetect、TransNetV2 还是自研镜头检测模型）[不确定]。这种「只说基础设施不说算法」的披露方式本身也说明切分被视为已解决的工程问题。切分后平均片段约 7 秒。
【ALIVE】数据管线六阶段中没有独立的 shot segmentation 环节，切分动作隐含在 character-driven pipeline 中：「Extracts N clips (3–10 seconds) from long videos (10–30 minutes)」——从 10–30 分钟长视频抽取 N 个 3–10 秒片段，但抽取依据（是否基于镜头边界、是否基于说话人活动、是否随机）未说明[不确定]。考虑到其身份锚点选取用的是「sync score 最高的 1.5 秒子片段」，抽取很可能是基于同步分而非镜头边界驱动的[推断，不确定]。
【OmniCustom】不做切分——直接使用上游 SpeakerVid-5M 已切好的片段，自身只做「时长 <10 秒剔除」的筛选与「前 4 秒 / 后 5 秒」的定点裁切。这是一种「切分外包给上游数据集」的典型做法。
【StreamChar】同样不做切分，完全复用 SpeakerVid-5M / TalkVid / OpenHumanVid 三个数据集的既有切分粒度，仅施加「≤20 秒」的长度约束。
【Baton / CCL】未提及切分方法[不确定]。二者均使用 OpenHumanVid 等已切分的公开数据集，加上自采/in-house 部分（Baton 的互联网视频、CCL 的短剧电影）——后者理论上必须做切分，但论文完全未描述[不确定]，这是 CCL 数据披露的明显缺口（影视与短剧是镜头切换最密集的内容类型，切分质量对其结果影响应该很大）。
【ITS-JAVG】不涉及。
【总体判断】2026 年这批工作中，shot segmentation 已高度「基础设施化」——要么外包给公开数据集，要么只提工程框架不提算法。这与 2024–2025 年论文普遍点名 PySceneDetect/TransNetV2 的风气形成对比，反映该环节已不再被视为技术贡献点。

### 质量过滤（美学评分、清晰度、OCR文字过滤、黑边/水印/logo检测） ⚠️

【ALIVE —— 四类算子 + 独创的六档清晰度体系（本批最系统）】
(1) OCR 文字与水印检测：原文「We employ Optical Character Recognition (OCR) to measure the proportion of overlaid text and detect watermarks」——注意其做法是「测量叠加文字的占比」而非简单二值判定，说明是按文字覆盖面积比例做阈值过滤（阈值未公开[不确定]）。
(2) 美学评分：「utilize a pre-trained aesthetics model for per-frame scoring」——逐帧（per-frame）打分而非抽帧打分，粒度较细。具体模型与阈值未公开[不确定]。
(3) 运动分析：RAFT 计算光流（见 motion_filtering）。
(4) 13 维低质量分类器：「a quality model ... is used to classify samples as either 'high-quality' or belonging to one of 13 distinct low-quality dimensions」——一个把低质量细分为 13 个维度的多标签分类器。这比常见的「单一质量分 + 阈值」精细得多，可支持按维度差异化处理（如某些维度可修复、某些必须丢弃）。但论文自始至终未列举这 13 个维度具体是什么[不确定]，是全文最令人遗憾的省略。
(5) 独立的 Clarity filtering 阶段：用「reference images across six distinct clarity levels」（六档清晰度参考图，从模糊到锐利）做评判基准——即把清晰度做成六级序数标签，而非连续分数或二值判定。1080p Refiner 阶段专用的 0.7M「high-clarity samples」即从最高档中筛出。这种「参考图锚定的序数分级」在同期工作中很少见，优点是标注一致性高、跨标注员可比。
【NAVA —— 多算子并行 + VLM 复筛】
(1) PaddleOCR 做 OCR 过滤与字幕擦除——特别之处在于不只是「检测到字幕就丢弃」，而是执行「subtitle removal」（字幕擦除/去除），把带字幕的素材修复后继续使用，显著提升数据利用率。这与 ALIVE「测量文字占比后过滤」的思路形成对比：一个修复、一个剔除。
(2) 视觉质量算子组：aesthetics（美学）、sharpness（锐度）、brightness（亮度）、motion score（运动分）四项并行打分。全部阈值未公开[不确定]。
(3) VLM-based filtering and tagging：用视觉语言模型二次筛选「视觉质量清晰」的片段——即在传统打分器之外叠加大模型语义判断（详见 model_as_data_judge）。
【OmniCustom —— 单一美学阈值（本批唯一给出美学阈值数值的工作）】明确剔除「aesthetics scores below 0.3」的视频——美学分 <0.3 即淘汰。这是本批工作中唯一公开的美学阈值数字，尽管其分数尺度（0–1 归一化）与具体美学模型未说明[不确定]。另有时长 <10 秒剔除。无 OCR / 水印 / 分辨率过滤描述[不确定]。
【StreamChar / CCL / Baton】未描述任何视觉质量过滤[不确定]。三者均依赖上游公开数据集自带的质量筛选（OpenHumanVid、SpeakerVid-5M、TalkVid 本身均带质量过滤），属于「质量控制外包」。
【ITS-JAVG（推理时的质量把关）】其验证器组合中 VQAScore 与 VideoReward-TA 承担视觉质量与文本一致性的把关，可视为「推理时的质量过滤」。论文的关键发现是：任何单一质量验证器都会被搜索算法钻空子（「the inference-time search algorithm exploits blind spots」），必须多验证器组合——这个结论对训练侧数据过滤同样有警示意义：单一美学分或单一质量模型作为唯一过滤依据，同样存在被数据分布「反向利用」的盲区。

### 运动过滤（光流/运动分数阈值、静态与抖动剔除） ⚠️

披露稀疏，仅两项涉及：
【ALIVE】使用 RAFT 计算光流做运动分析——原文「compute optical flow using RAFT」。RAFT 是本批中唯一被点名的光流模型，与 Ovi、HunyuanVideo 等前作的选择一致，说明 RAFT 已成为该环节的事实标准。但 ALIVE 未说明：运动分数的阈值、是否剔除静止片段、是否剔除抖动/运动过强片段、motion score 是否作为下游条件或课程调度依据[全部不确定]。
【NAVA】把 motion score（运动分）列为视觉质量算子之一，与 aesthetics / sharpness / brightness 并列。计算方法（是否用光流、用何模型）与阈值均未说明[不确定]。
【OmniCustom】无运动过滤——这是合理的，因为其数据全为单人说话头，运动幅度天然小且集中在面部，施加运动过滤反而会误伤有效样本。这提示运动过滤的必要性与 domain 强相关：通用视频生成必须做，talking-head 定制可以不做。
【StreamChar / CCL / Baton】未提及[不确定]。
【ITS-JAVG】不涉及训练侧运动过滤，但其验证器中 VideoReward-TA 隐含对运动质量的评价。
【总体判断】相比 2024–2025 年论文普遍详述运动分阈值与静止/抖动双端剔除，本批 2026 年工作对运动过滤的着墨明显减少，甚至连阈值都不再提及——反映该环节已被视为标准化的「已知操作」，技术叙事重心已转移到音视频对齐与语义打标上。

### 去重方法（精确去重与基于embedding的语义去重分别记录） ⚠️

【NAVA —— 唯一做语义去重且方法明确的工作】原文「Redundant or near-duplicate clips by extracting video embeddings with VideoCLIP and performing large-scale k-means clustering」——用 VideoCLIP 提取视频嵌入，再做大规模 k-means 聚类，据此剔除冗余与近重复片段。这是典型的 embedding-based 语义去重（而非哈希级精确去重），k 值、簇内保留策略（每簇保留几条、按什么排序）、相似度阈值均未公开[不确定]。选择 VideoCLIP 而非逐帧 CLIP 说明其在时序维度上也做了考量。未提及精确去重（pHash/帧指纹/文件哈希）环节[不确定]。
【ALIVE】六阶段管线中完全没有去重环节，论文全文未提 deduplication[不确定]。考虑到其数据源为内部 raw data pool 且量级达 11M 样本，缺少去重是一个值得注意的空白——除非其原始池在入库前已做过去重（论文未说明）。
【OmniCustom / StreamChar】依赖上游数据集（SpeakerVid-5M / TalkVid / OpenHumanVid）自带的去重，自身无去重步骤[不确定]。OmniCustom 有一个隐含的「反去重」风险：其从同一段长视频中切出「前 4 秒参考 + 后 5 秒训练」的配对，同一说话人可能在数据集中重复出现多次，这在身份定制任务中是特性而非缺陷（需要同一身份的多样本），但也意味着说话人层面的分布可能高度不均衡[不确定]。
【CCL / Baton】未提及去重[不确定]。Baton 混合了 OpenHuman-Vid、AudioCaps、WavCaps 与互联网视频四个来源——AudioCaps 本身是 AudioSet 的子集，而 WavCaps 部分来源也与 AudioSet/FreeSound 重叠，跨数据集的重复风险客观存在但论文未处理[不确定]。
【ITS-JAVG】不涉及；但其 Best-of-N / EvoSearch 中的候选多样性维持问题，与去重要解决的「样本冗余」在数学上同源。
【总体判断】七项中仅 NAVA 一家做了显式去重，且是本批数据披露最完整的一项——这印证了一个规律：去重投入与数据规模正相关（NAVA 处理 100M 视频，其余多在 1M–11M 量级或直接复用已清洗的公开集）。

### VLM/LLM作为质检员（多模态大模型质量打分与错配剔除，2026年从浅层打分器转向大模型语义判断的趋势） ⚠️

本维度是本批工作最能体现 2026 年趋势的部分——ALIVE 与 NAVA 都把 MLLM/VLM 深度嵌入数据管线，且不止用于打标，而是直接承担质检与语义判断职能：
【ALIVE —— MLLM 做音频质检与说话人归属校正（本批最深度的应用）】
(1) MLLM 做音频双准则质检：原文「We utilize an MLLM model ... to perform audio filtering based on two criteria. Firstly, for audio quality, the model assigns a score to each audio, allowing us to discard samples with significant background noise. Secondly, for audio-visual coherence, we assess the correlation ...」——即用多模态大模型同时承担两件事：给音频质量打分（剔除显著背景噪声）、判断音画语义相关性（分离强相关样本、管控 BGM 等弱相关样本占比）。这是一个标志性转变：音频质量过滤传统上靠 SNR、静音检测等信号处理指标，ALIVE 改用大模型直接「听」并打分；音画相关性传统上靠 CLAP/ImageBind 等对比学习模型算相似度，ALIVE 改用大模型做语义判断。
(2) Qwen3-omni 做旁白时间戳解析：在 SubjectID correction 管线中，用 Qwen3-omni（全模态大模型）解析 narration 的时间戳——这是把全模态 LLM 当作时序对齐标注器使用。
(3) Gemini 2.5 参与管线的其他环节（论文提及但角色未完全明确[不确定]）。
(4) 传统打分器仍在并用：OCR、预训练美学模型、RAFT、TS-TalkNet、CLIP、ArcFace、13 维质量分类器——构成「大模型语义判断 + 专用打分器」的混合体系，而非完全取代。
【NAVA —— VLM 过滤 + 三个 Gemini/Qwen 模型分工打标】
(1) VLM-based filtering and tagging：用视觉语言模型筛选并标注视觉质量清晰的片段，是显式的「VLM 作质检员」。
(2) omni-modal tagger 配合 YAMNet 做五类音频分类——传统音频分类器（YAMNet）与全模态大模型标注器协同，前者提供快速粗分类、后者提供语义细化。
(3) 打标环节三模型分工：Qwen3-VL（视频 caption）、Qwen3-Omni（音频 caption）、Gemini-3-Flash（融合改写）；高质量子集升级为 Gemini-3-Pro。用不同档位的模型处理不同质量档位的数据，是成本-质量权衡的典型工程实践。
(4) 「multi-operator collaborative filtering」（多算子协同过滤）产出 160K SFT 子集——「协同」一词暗示各算子分数被联合而非串行阈值化处理[具体聚合方式不确定]。
【OmniCustom】用 GLM-ASR 做转写，属专用模型而非通用 MLLM；caption 构造沿用 Ovi 的属性体系（说话人年龄、性别、口音、音高、韵律、情绪、语速），未描述用何模型生成这些属性标注[不确定]，也未用大模型做质检。过滤全靠 SyncNet + 美学分 + 时长三条硬规则。
【CCL / Baton / StreamChar】均未使用大模型做数据质检[不确定]。Baton 虽有 Qwen3-8B 作为 VA-Planner 底座，但那是模型组件而非数据管线角色。
【ITS-JAVG —— 把「模型判官」范式化并证明其陷阱（对本维度最有理论价值的贡献）】
虽不涉及训练数据，但其核心研究对象正是「模型作为判官」这一范式本身，结论对数据质检有直接迁移价值：
(1) 六个验证器各司其职：VideoReward-TA（文本-视频一致性）、JavisScore（细粒度音视频同步）、ImageBind-TA（文本与生成音频的语义连贯）、AVHScore（音频事件与视觉事件的语义一致性）、VQAScore（文本-视频对齐）、ImageBind（音视频语义相似度）。
(2) 核心发现一：单验证器必然导致非对称权衡——「single-verifier guidance effectively improves its intended evaluation metrics, yet fails to achieve a balanced improvement across all metrics」，即只优化某个判官的分数会牺牲其他维度。
(3) 核心发现二：verifier hacking（验证器钻空子）真实存在——搜索算法会「exploit blind spots」（利用判官的盲区）。这对训练侧数据过滤是重要警示：若用单一模型打分器做严格过滤，被保留的数据会系统性偏向该打分器的偏好与盲区，形成隐性的数据分布扭曲。
(4) 解法 ARW：把奖励聚合当作在线优化问题，公式 R(i)=Σ_k w_k · r_k(i)/(σ_k+ε)（按各判官分数的标准差归一化后加权），优化目标 L_ARW=Σ_k (½exp(−s_k)·Var̂(r_k) + ½|s_k|)——即自适应地按各判官分数的方差调整权重，方差大（区分度高）的判官权重高。这套「多判官分数的自适应归一化聚合」方法论完全可以移植到训练数据的多算子协同过滤上（恰好 NAVA 用的正是「multi-operator collaborative filtering」，但未公开聚合算法）。

### 安全与合规过滤（NSFW、版权、人脸/隐私） ⚠️

[不确定]——七项工作全部未描述 NSFW 过滤、暴力/血腥内容识别、版权内容检测、人脸隐私保护、名人肖像剔除等安全合规环节。
仅有的间接相关内容：
(1) ALIVE 的 13 维低质量分类器中是否包含安全维度未知[不确定]；其 OCR 水印检测可视为对版权标记的间接识别，但目的是画质而非合规。
(2) NAVA 的 PaddleOCR 字幕擦除同样是画质导向。
(3) OmniCustom 与 ALIVE 大量使用 ArcFace 人脸嵌入做身份匹配/校验，涉及生物特征处理但无隐私声明；OmniCustom 评测集使用「30 persons who were not included in training data」+ 70 段 YouTube 视频，未说明肖像授权。
(4) 音色克隆的滥用风险：OmniCustom（参考音频→音色模仿）与 NAVA（Timbre-in-Context Conditioning，参考音色控制）都具备声纹克隆能力，这是明确的深度伪造风险点，但两篇论文均未包含 responsible-use 声明、水印方案或滥用防护讨论[不确定]。NAVA 虽以 Apache 2.0 开源权重与训练代码，仓库亦未见使用限制条款[不确定]。
(5) StreamChar 的实时流式数字人同样具备实时伪造潜力，无相关讨论。
【判断】安全合规是本批七项工作最统一的空白维度——七项中零项有实质披露，且其中三项（OmniCustom、NAVA、StreamChar）恰恰涉及身份与声纹克隆这类高风险能力。相比之下，同期的 Sora 2、Veo 3 等闭源工业模型均有明确的安全章节，开源/学术侧的这一落差值得注意。

## 打标方式

### 使用的caption模型（自研VLM/开源模型，模型规模） ⚠️

本批工作在 caption 模型上呈现「专用小模型 → 通用大模型 → 多模型分工」的明显演进：
【ALIVE —— 自训 caption 模型 + 两轮 SFT + 人工修订（自建路线）】
(1) 视觉 caption 模型经过两轮 SFT（two-round supervised fine-tuning），训练数据为 MLLM 生成后再「manually revised caption data」（人工修订的 caption 数据）——即「大模型生成 → 人工修订 → SFT 蒸馏成专用 caption 模型」的经典自建路径，用一次性的人工成本换取大规模推理的低成本。
(2) 第二类人工标注：「decomposing sub-motion units to annotate the start and end timestamps of each complete small action」——把动作分解为 sub-motion units（子动作单元）并人工标注每个完整小动作的起止时间戳。这是极高成本的细粒度时序标注，目的是让 caption 具备精确的时间轴信息（服务于 Narration 字段的时序叙述与 UniTemp-RoPE 的时序对齐）。
(3) 音频信息融入：一个 MLLM 处理视觉描述以「incorporate the relevant audio information」（融入相关音频信息），实现视听合流。caption 主模型的基座名称论文未一致标明[不确定]；管线其他环节出现 Gemini 2.5 与 Qwen3-omni。
【NAVA —— 三模型分工 + 双档位策略（外部大模型路线）】
(1) 全量数据：Qwen3-VL 生成视频 caption、Qwen3-Omni 生成音频 caption，两者「are then fused by either direct concatenation or rewriting by Gemini-3-Flash」——由 Gemini-3-Flash 做直接拼接或改写融合。注意「direct concatenation or rewriting」二选一，说明存在成本分层：部分数据只做拼接（零成本），部分做改写（有成本）。
(2) 高质量子集：升级为 Gemini-3-Pro，「to produce more accurate, structured, and temporally grounded audio-visual captions」（更准确、更结构化、时序落地更强的音视频 caption）。「temporally grounded」（时序落地）是关键词，与 ALIVE 的 sub-motion 时间戳标注异曲同工，说明 2026 年 AV caption 的竞争焦点已在时序精度上。
(3) 这种「Flash 打全量 + Pro 打精品」的双档位策略是本批中最清晰的 caption 成本工程实践。
【OmniCustom】用 GLM-ASR 为每段 5 秒训练音频生成转写；音频 caption 的属性体系明确「Following the OVI model approach」（沿用 Ovi 的做法），强调说话人的 age、gender、accent 及 vocal characteristics（pitch、prosody、emotion、speaking rate）。生成这些属性标注所用的模型未说明[不确定]。视觉 caption 是否自建未说明[不确定]。
【StreamChar】使用 ASR 时间戳（「ground-truth end indices derived from ASR timestamps」）作为 progress-aware pointer 的监督信号，但未点名 ASR 工具[不确定]；视觉 caption 依赖上游数据集自带标注[不确定]。
【CCL / Baton】完全未描述 caption 模型或打标流程[不确定]。Baton 虽以 Qwen3-8B 为 VA-Planner 底座并回归 SigLip2 / WavTokenizer 特征，但这是模型架构而非数据标注环节；其训练数据的 caption 来自 OpenHuman-Vid / AudioCaps / WavCaps 自带标注，互联网自采部分如何打标未说明[不确定]。
【总体规律】数据量越大越倾向自建 caption 模型（ALIVE 11M 样本 → 自训+人工修订），中等规模倾向调用外部大模型（NAVA 15M 但用 Qwen3/Gemini 组合，且分档控成本），小规模则直接复用上游标注（Baton 1.5M、CCL 4M）。

### caption密度与结构化程度（短/长/密集描述、结构化字段如镜头运动、风格标签） ⚠️

【ALIVE —— 三段式结构化 + 内联时序标签（本批最精细）】
 caption schema 明确分三个字段：
- Subjects（主体）：包含「visual appearance + acoustic profiles」——即每个主体同时携带视觉外观描述与声学画像（acoustic profile）。把「这个人长什么样」与「这个人声音是什么样」绑在同一个主体条目下，是 AV 模型 caption 设计的关键创新：它让模型在生成时能把身份的视觉与听觉两面统一起来（这也是其 SubjectID correction 阶段要费五步管线校正说话人归属的原因——归属错了，声学画像就会挂到错误的人脸上）。
- Visual（视觉上下文）：场景设定（scene setting）。
- Narration（叙述）：动作的时序序列（temporal sequence of actions），内嵌两类特殊 token：
  · <W>：包裹 verbatim speech content（逐字语音内容）
  · <I>：标记 non-speech acoustic events（非语音声学事件）
即视觉动作叙述、台词、音效三者按时间顺序交织在同一条 Narration 中。这与 Ovi 的 <S>...<E> + <AUDCAP> 方案思路相近但更彻底——Ovi 把音频描述统一放在末尾，ALIVE 则让非语音事件 <I> 也内联到时间轴上，时序表达力更强。
配合人工标注的 sub-motion units 起止时间戳，Narration 的时序精度有 ground-truth 支撑。
【NAVA —— 统一模板的结构化密集 caption（四大板块）】
明确采用「structured dense captions rather than free-form short descriptions」（结构化密集描述而非自由形式的短描述），统一模板覆盖四个板块：
- 全局视觉语义：style（风格）、mood（氛围）、subject（主体）、appearance（外观）、background（背景）、lighting（光照）
- 时序动态：initial state（初始状态）、action sequences（动作序列）、motion details（运动细节）、ending state（结束状态）——这是一个明确的「起始态→过程→终止态」三段式时序骨架
- 镜头行为：camera angle（机位角度）、shot scale（景别）、framing（构图取景）、composition（构图）
- 音频描述：speech（语音）、SFX（音效）、music（音乐）、ambient sound（环境音）四类
合计约 18 个显式字段，是本批中字段最完整的 caption schema。特别值得注意的是它把「镜头行为」单列一个板块（4 个字段），这在 AV 生成工作中不常见，说明 NAVA 对相机可控性有明确诉求。
【OmniCustom】音频 caption 沿用 Ovi 体系：说话人 age、gender、accent + vocal characteristics（pitch、prosody、emotion、speaking rate）共 7 项属性；另有 GLM-ASR 产出的逐段转写。视觉 caption 结构未描述[不确定]。属于「音频侧结构化、视觉侧不详」的偏科设计，符合其身份+音色定制的窄任务。
【Baton —— 不用文本 caption，用连续特征做「语义蓝图」（本批最异类）】
Baton 的核心主张恰恰是「文本 caption 不够用」：现有方法依赖 off-the-shelf 文本编码器的粗粒度 embedding，「discards fine-grained semantics」（丢弃细粒度语义）且缺乏共享的长时规划。其替代方案是让 VA-Planner（Qwen3-8B 底座）产出 planned tokens，并通过 dual semantic alignment towers（双语义对齐塔，各带 learnable queries 做跨模态注意力）把 planned tokens 投影到预训练感知编码器的连续特征域——视频侧对齐冻结 SigLip2 的倒数第二层特征，音频侧对齐冻结 WavTokenizer 的倒数第二层特征，用 L2 回归监督。论文明确说明为何不用离散 token：「regressing continuous features preserves richer semantic structure」（回归连续特征能保留更丰富的语义结构）。
这在 caption 结构谱系中代表了一个新方向：从「越来越结构化的文本 caption」转向「绕过文本、直接用连续感知特征作为跨模态语义中介」。若这条路线成立，未来 AV 数据管线的重点可能从「怎么写好 caption」转向「怎么选好感知编码器」。
【CCL / StreamChar / ITS-JAVG】未描述 caption 结构[不确定]。StreamChar 使用的是 transcript（台词全文）而非描述性 caption，其编排器输入为「transcript 与历史上下文」，属于「脚本驱动」而非「描述驱动」的条件形式。

### 音视频联合caption结构（是否同时覆盖视觉+听觉轨道、是否分流为独立字段，如LTX-2全音景描述、Script-a-Video factorized streams、Foley-Omni三字段） ⚠️

本批工作在音视频联合 caption 上分化为四种范式，是 2026 年该维度最具信息量的横切面：
【范式一｜交织内联式（ALIVE）】三字段 + 双标签：Subjects[视觉外观 + 声学画像] / Visual[场景] / Narration[动作时序 + <W>逐字台词 + <I>非语音声学事件]。三条轨道（视觉动作、语音、音效）全部内联到同一条时序叙述中，靠标签而非字段分流。相比 Ovi 的「<S>台词<E> 内联 + <AUDCAP>音景块置尾」，ALIVE 把 <I>（非语音事件）也拉到时间轴上内联，时序表达更彻底。另一独到之处是 Subjects 字段把「视觉外观」与「声学画像」绑定在同一主体下——这是把音视频对齐从「时序层面」提升到「实体层面」的设计，也是其必须用五步 SubjectID correction 管线保证归属正确的根本原因。
【范式二｜分字段并列式（NAVA）】统一模板下音频作为独立板块与视觉板块并列，音频板块内再细分 speech / SFX / music / ambient sound 四类。生产方式是「双模型分产 + 第三模型融合」：Qwen3-VL 产视频 caption、Qwen3-Omni 产音频 caption，Gemini-3-Flash（或 Pro）做拼接或改写融合。这种「先分后合」的生产方式与「分字段并列」的最终结构互相印证，也带来一个隐患：两个模型独立生成时可能出现音视描述互相矛盾，融合模型需要承担一致性校验职责（论文未说明是否有此校验[不确定]）。
【范式三｜属性化音频 caption（OmniCustom）】明确「Following the OVI model approach」构造音频 caption，强调「speaker's age, gender, accent, and vocal characteristics (e.g., pitch, prosody, emotion, and speaking rate)」——即音频 caption 不描述「听到了什么」而描述「说话人是什么样的」，是一种说话人属性化的 schema。配合 GLM-ASR 的逐段转写形成「属性描述 + 台词内容」两轨。这套 schema 直接服务于其音色定制任务：属性可控 = 音色可控。
【范式四｜绕过文本 caption（Baton）】不构造联合 caption，改用 VA-Planner 输出的 planned tokens 作为音视频共享的语义蓝图，视频侧与音频侧分别通过 dual semantic alignment towers 对齐到 SigLip2 与 WavTokenizer 的连续特征域。这是对「联合 caption」这一范式本身的否定——其论点是文本 embedding 天然丢失细粒度语义，用连续感知特征作跨模态中介信息量更足。RS-RoPE（Relative Semantic RoPE）负责把蓝图注入去噪过程。
【无 schema｜CCL / StreamChar / ITS-JAVG】CCL 未描述联合 caption 设计[不确定]；StreamChar 用 transcript（台词）而非描述性 caption 作为主条件，属「脚本驱动」范式；ITS-JAVG 无训练数据，但其验证器 AVHScore 恰恰是在推理时度量「音频事件与视觉事件的语义一致性」——相当于把联合 caption 想要编码的音视语义绑定关系，改用判别式模型在输出端检验。
【与既有工作的谱系定位】把本批放入更大图景：Ovi 为「内联标签 + 末尾音景块」、LTX-2 为「全音景统一描述」、Script-a-Video 为「完全 factorized 独立流」、Foley-Omni 为「三字段拆分」。ALIVE 属于 Ovi 路线的加强版（内联更彻底 + 实体级绑定），NAVA 属于 factorized 路线（分板块并列），OmniCustom 属于属性化专用路线，Baton 则开辟了「非文本中介」的第四条路。2026 年上半年的态势是：内联时序化与分字段结构化两条主流并行，同时出现了绕过文本的探索。

### 对白转写与说话人属性标注（ASR转写+说话人身份/语言/口音/情绪） ⚠️

【OmniCustom —— 最完整（对白转写 + 七项说话人属性）】
(1) 转写：「we used GLM-ASR to generate transcriptions for each 5s training audio clip」——用 GLM-ASR 为每个 5 秒训练音频片段生成转写。GLM-ASR 为智谱系语音识别模型，中文能力较强。
(2) 说话人属性（沿用 Ovi 体系）：age（年龄）、gender（性别）、accent（口音）+ vocal characteristics 四项（pitch 音高、prosody 韵律、emotion 情绪、speaking rate 语速），合计 7 项。
(3) 音色的隐式标注：其「同音色异内容」配对构造（前 4 秒参考音频 + 后 5 秒训练音频来自同一段视频、同一说话人）本质上提供了一种无需显式声纹标注的音色监督——同一 pair 内音色恒定、内容不同，天然构成音色解耦的对比学习信号。这比显式标注声纹 ID 成本低得多，是很聪明的数据构造。
(4) 评测集控制性别比 1:1，且含 30 位训练集外真人以验证零样本音色迁移。
【ALIVE —— 说话人归属校正是其独有贡献】
(1) 转写标注：<W> 标签包裹 verbatim speech content（逐字语音内容），说明有精确 ASR 转写；<I> 标签标记非语音声学事件。
(2) 声学画像：Subjects 字段中每个主体带 acoustic profiles（声学画像），即说话人音色属性与视觉外观绑定。
(3) SubjectID correction 五步管线（针对多人场景「谁在说话」的错误归属，是本批唯一系统处理该问题的工作）：
  ① TS-TalkNet 做目标说话人主动说话人检测（Target-Speaker TalkNet），生成人脸-声音对应矩阵（face-voice correspondence matrix）
  ② Qwen3-omni 解析 narration 的时间戳
  ③ majority voting（多数投票）建立「句子 → TrackID（人脸轨迹 ID）」的映射
  ④ CLIP + ArcFace 嵌入做视觉-语义匹配
  ⑤ 用校正后的标签重写文本（text rewriting with corrected labels）
未通过高置信阈值的视频被直接过滤，得到「simplified」但高质量的初始数据集。这个管线解决的是纯视频模型完全不会遇到的问题：多人画面中把台词挂到错误的人脸上，会让模型学到错误的「脸-声」绑定，直接破坏多人对话场景的唇同步与音色一致性。阈值数值未公开[不确定]。
【StreamChar】使用 ASR 时间戳作为监督：「ground-truth end indices derived from ASR timestamps」用于 progress-aware pointer 的 smooth L1 损失——即把「台词念到第几个字」这一进度信息作为显式监督信号，是 transcript fidelity（台词保真）的数据基础。ASR 工具未点名[不确定]；说话人属性标注未描述[不确定]。
【NAVA】Timbre-in-Context Conditioning 用「reference utterance」（参考话语）经 timbre encoder（实现上为 ReDimNet 说话人嵌入器）编码，把参考音色线索关联到对应的 speech spans（语音片段）——即音色不是全局条件而是按语音段绑定，支持多说话人各自音色。但论文未说明标注协议（如何确定 speech span 边界、是否用 diarization）[不确定]；ASR 方法未详述[不确定]。其 YAMNet 五类标签中区分 single-speaker / multi-speaker speech，可视为一种粗粒度的说话人数标注。
【CCL】报告 WER 指标说明其评估了语音可懂度，隐含有转写-生成对照，但数据侧的转写流程未描述[不确定]。
【Baton】未描述[不确定]。

### 几何与结构化标注（相机参数、深度、3D point tracks、动作标注、显式状态） ⚠️

本批工作在几何标注上普遍薄弱，唯一的亮点在人脸/身份相关的结构化标注（这对 AV 模型比相机参数更关键）：
【ALIVE —— 人脸轨迹与身份的结构化标注（本批唯一有实质结构化标注的工作）】
(1) 人脸轨迹 ID（TrackID）：SubjectID correction 管线中建立「句子 → TrackID」映射，说明数据中存在人脸轨迹级的结构化标注（每个人脸在时间上被跟踪并分配 ID）。
(2) 人脸-声音对应矩阵：TS-TalkNet 产出的 face-voice correspondence matrix 是一个显式的结构化跨模态关联标注。
(3) 身份嵌入：ArcFace 人脸嵌入 + CLIP 视觉嵌入，用于跨片段身份匹配。Character-driven pipeline 中给出了明确的匹配阈值：face similarity >0.35、CLIP similarity >0.7、absolute proximity to 0.9——这是本批中唯一公开的身份匹配阈值组。
(4) 同步分：TS-TalkNet 的 sync score 用于选取身份锚点（「1.5-second sub-clip with maximum sync score」）。
(5) 人工的 sub-motion units 时间戳标注：把动作分解为子动作单元并标注起止时间——这是一种时序结构化标注（动作边界），虽非几何但属显式结构化。
(6) 未涉及：相机内外参、深度图、3D point tracks、姿态关键点、分割 mask[全部不确定]。
【NAVA】caption 模板中含「camera behavior」板块（camera angle 机位角度、shot scale 景别、framing 取景、composition 构图）——这是语言化的相机描述而非数值化的相机参数，属于「弱结构化」标注。无深度、3D track、姿态等[不确定]。说话人嵌入（ReDimNet）可视为声学侧的结构化标注。
【OmniCustom】参考图通过「randomly sampling and cropping a frame containing a face」（随机采样并裁剪含人脸的帧）获得——隐含人脸检测标注，但仅用于裁剪，未保留为结构化字段[不确定]。无几何标注。
【StreamChar】使用 reference frame 与 motion frame 作为条件——motion frame 是一种隐式的运动状态表示，非显式几何标注。ASR 时间戳属时序结构化标注。
【Baton】planned tokens 本质是「连续语义特征」，可视为一种非人类可读的隐式结构化中间表示——这是对传统结构化标注的另一种替代思路：不标注人类定义的结构（相机、深度、动作），而让模型自己学出一套语义结构。SigLip2 与 WavTokenizer 的倒数第二层特征即其「结构」的载体。
【CCL / ITS-JAVG】无[不确定]。
【总体判断】AV 生成模型的「结构化标注」重心与纯视频生成模型明显不同：后者关注相机参数、深度、3D 轨迹；前者关注人脸轨迹 ID、脸-声对应、说话人嵌入、语音时间戳。ALIVE 的 face-voice correspondence matrix 与 TrackID 体系是这一差异最典型的体现。

### 合成数据构造（受控扰动/编辑构造训练对，如InstructAV2AV） ⚠️

【ALIVE —— 本批唯一系统性构造合成/增强数据的工作，且方法值得细看】
Character-driven data pipeline 分两路：
(1) Cross-pair Pipeline（跨片段配对，用真实数据构造「同一身份不同场景」的训练对）：
  - 从 10–30 分钟长视频中抽取 N 个 3–10 秒片段
  - 选取「1.5-second sub-clip with maximum sync score」（同步分最高的 1.5 秒子片段）作为身份锚点（identity anchor）——用同步分而非画质分选锚点很有讲究：同步分高意味着此刻人物正在清晰说话、正脸朝向摄像机，是身份特征最充分暴露的时刻
  - 用 CLIP + ArcFace 嵌入做多主体特征匹配
  - 有效跨片段配对需同时满足：face similarity >0.35、CLIP similarity >0.7、absolute proximity to 0.9
  这一路是「用真实长视频挖掘同身份配对」，不是生成合成像素，但属于典型的配对数据构造（training pair construction）。
(2) In-pair Augmentation（片段内增强，真正的合成数据构造）：
  - pose/expression perturbation（姿态/表情扰动）
  - background augmentation（背景增强/替换）
  - semantic editing via Qwen-Image-Edit（用 Qwen-Image-Edit 做语义编辑）
  目的明确：「to decouple identity from appearance」（把身份与外观解耦）——即让模型学会「这个人换了姿势、换了背景、换了服装仍是同一个人」，而不是把背景和衣着记成身份的一部分。用图像编辑大模型做受控扰动来构造正样本对，是 2026 年很典型的做法（与 InstructAV2AV 类思路同源，但 ALIVE 用于身份解耦而非指令编辑）。
(3) Prompt 侧的合成：训练时以 0.3 概率 dropout 跨注意力信号（cross-attention signal），属条件 dropout 增强；推理侧用 FAISS 构建稠密向量索引，从用户 prompt 中抽取声音事件（sound events）并检索最近邻，相似度阈值 τ>0.85，做检索增强的 prompt 改写——这是推理期而非训练期的合成。
【OmniCustom —— 配对构造而非像素合成】其「前 4 秒作参考音频 / 后 5 秒作训练音视频」的切分策略，是一种零成本的配对数据构造：「each reference-training pair shares the same timbre but contains distinct speech content」（同音色、异内容）。这直接为音色解耦提供监督：模型必须学会从参考音频中只提取音色而不复制内容。参考图则从含人脸的帧中随机采样裁剪，同样是配对构造。无像素级合成或编辑。
【StreamChar —— 蒸馏数据即自生成数据】两阶段蒸馏中 Stage II 为「rollout consistency」（在线 chunk rollout 下微调 student），本质上是学生模型自己生成的 rollout 序列作为训练数据——属于 on-policy 的自生成数据，与传统合成数据构造不同但同属「模型产出数据回灌训练」。Stage I 600 steps 做步数压缩到 4 步生成器，Stage II 400 steps，student lr 2e-6、fake score network lr 4e-7。蒸馏数据源复用同一套音视频数据集，无额外构造。
【NAVA / CCL / Baton】未描述合成数据构造[不确定]。Baton 的三阶段训练中 Stage 2→Stage 3 的过渡涉及「从干净 ground-truth 特征切换到规划器预测特征」，这是一种课程式的输入扰动（用模型自身的不完美预测替代干净标签），思路上接近 scheduled sampling，可视为一种训练输入侧的合成，但不产生新数据样本。
【ITS-JAVG】其 Best-of-N 与 EvoSearch 在推理时大量生成候选再择优，若把择优结果回灌训练即构成合成数据管线——但论文明确 training-free，未做此扩展。这是一个显而易见的后续方向（用 ITS 的多验证器筛选产出高质量合成 SFT 数据），论文未涉及。

### 人工介入程度（人工标注、人工质检、模型初筛+人工复核） ⚠️

本批工作中人工介入程度差异显著，ALIVE 是唯一有实质人工标注投入的：
【ALIVE —— 两类人工标注 + 人工评测（本批投入最重）】
(1) 人工修订 caption：caption 模型的 SFT 训练数据为「manually revised caption data」（人工修订的 caption 数据），经两轮 SFT。这是「大模型初标 + 人工复核修订 + 蒸馏成专用模型」的标准范式，人工成本一次性投入、收益规模化。
(2) 人工标注 sub-motion units 时间戳：「decomposing sub-motion units to annotate the start and end timestamps of each complete small action」——把动作分解为子动作单元，人工标注每个完整小动作的起止时间戳。这是本批中成本最高的人工标注项（细粒度时序标注单位成本远高于打描述性标签），说明团队高度重视时序精度。
(3) 两类标注的规模（标注条数、标注员数量、成本）均未量化[不确定]。
(4) 人工评测：与 Veo 3.1、Kling 2.6、Wan 2.6、Sora 2、LTX-2 做盲测 side-by-side 对比；Alive-Bench 1.0 的 264 general prompts + 90 reference-character prompts 与 22 项细粒度指标（六大类）应含人工设计与评判成分[评判方式细节不确定]。
【StreamChar —— 人工评测明确量化】「24 participants, each presented with 50 randomly sampled cases」——24 名参与者、每人评 50 个随机抽样案例。这是本批中唯一给出人评参与者数与样本数的工作。数据侧无人工标注[不确定]。
【OmniCustom】数据管线全自动（SyncNet + 美学分 + 时长三条规则 + GLM-ASR），无人工标注环节。评测集构建有人工介入：精心挑选 100 例（30 位训练集外真人 + 70 段 YouTube 视频），并控制性别比 1:1——这个 1:1 的配比必然是人工筛选的结果。
【NAVA】数据管线高度自动化（全部依赖 OCR / VideoCLIP / 各类打分器 / VLM / LLM），无人工标注描述[不确定]。160K 高质量 SFT 子集由「multi-operator collaborative filtering」自动筛出而非人工挑选。评测含 user studies（用户研究），评估视频质量、音视频同步、音频保真度、音色可控性四个维度，参与者规模未说明[不确定]。
【CCL / Baton】无人工介入描述[不确定]。
【ITS-JAVG】无训练数据故无标注；其核心方法论恰恰是「用模型验证器替代人工判断」——多验证器组合 + ARW 自适应加权，本质上是在自动化地逼近人类偏好。但论文同时揭示了这条路的风险（verifier hacking），间接论证了人工评测不可完全替代。
【规律】人工投入量与数据披露详细度正相关：ALIVE 数据披露最详、人工投入最重；反之 CCL/Baton 披露最少、无人工描述。这可能反映了「重视数据的团队才会既投入人工又详细披露」的一致性。

## 音视频对齐

### 音视频同步检测方法（口型同步、事件对齐） ⚠️

本批工作在同步检测上分为三档，且出现了工具多样化的趋势：
【NAVA —— 三检测器并用（本批最全面）】
数据过滤阶段的音视对齐算子明确为三个并行：「audio-visual alignment, measured by SyncNet, SyncFormer, and ImageBind」——SyncNet（唇音时序同步，经典口型检测）、SyncFormer/Synchformer（Transformer 架构的通用音视同步检测，可处理非语音事件对齐）、ImageBind（跨模态语义相似度，度量音画内容是否讲同一件事）。这三者恰好覆盖三个不同层次：SyncNet 管唇音时序、Synchformer 管通用事件时序、ImageBind 管语义匹配——是一个层次分明的对齐检测体系，比单用 SyncNet 的做法（如 Ovi、OmniCustom）完整得多。全部阈值未公开[不确定]，这是 NAVA 数据披露最大的遗憾。
【OmniCustom —— 单一 SyncNet 双阈值（阈值完全公开）】
沿用 Ovi 的标准：「|offset|≤3」且「confidence >1.5」。这组数值与 Ovi 论文完全一致（|offset|≤3 帧、confidence>1.5），说明 Ovi 的阈值已成为该领域的事实参考标准并被后续工作直接继承。按 24fps 计，|offset|≤3 帧即时间容差 ≤125 毫秒。仅针对含语音片段的唇音同步，无非语音事件对齐检测[不确定]。
【ALIVE —— TS-TalkNet 做主动说话人检测（用途不同于纯同步过滤）】
使用 TS-TalkNet（Target-Speaker TalkNet）「to evaluate the active speaker」（评估主动说话人），产出人脸-声音对应矩阵。注意其主要用途不是「过滤不同步样本」而是「判断多人画面中谁在说话」——这是一个更难的任务（active speaker detection 而非 lip-sync verification）。sync score 的另一用途是选取身份锚点（同步分最高的 1.5 秒子片段）。阈值未公开，仅说明「Filters videos failing high-confidence thresholds」（过滤未达高置信阈值的视频）[阈值数值不确定]。
此外 ALIVE 的音画相关性判断改由 MLLM 承担（见 model_as_data_judge），与 TS-TalkNet 的说话人检测形成分工：MLLM 管语义相关性、TS-TalkNet 管说话人归属。
【StreamChar / CCL / Baton —— 数据侧无同步过滤】
三者均未描述数据侧的同步检测[不确定]，依赖上游数据集（SpeakerVid-5M / TalkVid / OpenHumanVid 本身带同步筛选）或模型架构保证。CCL 在评测中报告 Sync-C / Sync-D / DeSync 三个同步指标，说明重视同步性但把保障放在架构（TARP 时序对齐 RoPE）而非数据侧。
【ITS-JAVG —— 把同步检测搬到推理时】
用 JavisScore 作为细粒度音视频同步验证器（「focuses on fine-grained audio-video synchronization」），另有 AVHScore 度量音频事件与视觉事件的语义一致性、ImageBind 度量音视频语义相似度。关键发现是只用 JavisScore 引导搜索会导致其他指标退化（非对称权衡），且搜索会钻同步验证器的空子。这对数据侧的启示很直接：仅用 SyncNet 单一阈值过滤，保留下来的数据会系统性偏向 SyncNet 的偏好（例如偏向正脸、清晰口型的样本），造成分布扭曲——NAVA 的三检测器并用恰好可以缓解这一问题。作者 Joon Son Chung 正是 SyncNet 原作者之一，其对同步度量局限性的判断尤具权威性。

### 同步检测具体指标与阈值（SyncNet/Synchformer/LSE/自研，阈值数值，如MOVA LSE-D≤9.5且LSE-C≥4.5、SkyReels-V4 SyncNet |offset|≤3∧conf>1.5） ⚠️

【完全公开阈值的仅一项】
OmniCustom：SyncNet「|offset|≤3」且「confidence>1.5」（与 Ovi 完全一致，直接继承）；配合美学分 <0.3 剔除、时长 <10 秒剔除。三条规则为「与」关系。

【ALIVE 的相关阈值（非同步阈值但同属数据筛选阈值，本批唯一公开的一组数值）】
Character-driven cross-pair 的有效配对判定：
- face similarity（ArcFace 人脸相似度）> 0.35
- CLIP similarity（CLIP 视觉相似度）> 0.7
- absolute proximity to 0.9
身份锚点选取：sync score 最大的 1.5 秒子片段。
SubjectID correction：过滤未达「high-confidence thresholds」的视频，具体数值未公开[不确定]。
其他：训练时 cross-attention signal dropout 概率 0.3；推理时 FAISS 检索相似度阈值 τ>0.85。
TS-TalkNet 的同步分阈值未公开[不确定]。

【NAVA】检测器名单公开（SyncNet、SyncFormer、ImageBind）但「does not provide exact numerical thresholds」——全部阈值未公开[不确定]。同样地，视觉质量算子（aesthetics、sharpness、brightness、motion score）与 AudioBox Aesthetic 的阈值也全部未公开[不确定]。这是一篇数据流程描述完整但数值全部隐去的典型工业论文。

【StreamChar / CCL / Baton】数据侧无同步阈值[不确定]。

【评测侧指标（非过滤阈值，但反映各工作对同步的度量口径）】
- CCL：Table 2 报告 WER、Sync-C、Sync-D、DeSync、IB 五项指标做消融——Sync-C/Sync-D 为 SyncNet 系置信度与距离，DeSync 为 Synchformer 系去同步度，IB 为 ImageBind 分数。五指标并用说明其对同步的度量比多数工作更细。
- NAVA：在 Verse-Bench 上刷新 Sync-C / Sync-D / 视频质量 / 音频 WER 的 SOTA，且参数量比开源基线少 2–5 倍。
- ALIVE：Alive-Bench 1.0 含 22 项细粒度指标（六大类），同步应为其中之一[细节不确定]。
- ITS-JAVG：JavisScore（细粒度同步）、AVHScore（音视事件语义一致）、ImageBind（音视语义相似）三个与同步相关的验证器；ARW 归一化公式 R(i)=Σ_k w_k·r_k(i)/(σ_k+ε)。

【横向观察】2026 年该领域的阈值披露呈两极：学术/中小团队（OmniCustom）完整公开且直接沿用 Ovi 标准；工业团队（NAVA、ALIVE）流程公开、数值隐去。SyncNet 的 |offset|≤3 / conf>1.5 已固化为社区默认配置。

### 时序同步vs语义同步的分离处理（时间对齐与内容语义匹配作为两个独立过滤条件） ⚠️

本批工作中，「时序对齐」与「语义匹配」的分离处理意识明显增强，NAVA 与 ALIVE 各有代表性做法：
【NAVA —— 数据侧三层分离 + 架构侧显式分离（本批最清晰）】
(1) 数据侧：SyncNet（唇音时序）、SyncFormer（通用事件时序）、ImageBind（跨模态语义）三个算子并行——前两者管时序、后者管语义，是显式的双轨过滤。
(2) 架构侧：NAVA 的核心论点就建立在这一分离上——批评「unified tri-modal designs」（统一三模态设计）「conflate semantic and low-level alignment」（把语义对齐与低层对齐混为一谈）。其 Align-then-Fuse 设计正是为解耦二者：先在专用对齐空间建立音视频的低层/细粒度对应（native alignment），再把上下文（文本、说话人嵌入）作为外部条件通过 cross-attention 注入语义（context as external conditioning）。这是本批中把「时序 vs 语义分离」提升到架构第一性原理高度的唯一工作。
【ALIVE —— 音画相关性作为独立于同步的过滤维度】
音频质量过滤明确采用双准则：第一条是音频质量分（噪声），第二条是「audio-visual coherence」（音画一致性）——后者独立于 TS-TalkNet 的时序同步检测，专门判断音频内容与画面内容是否语义相关，并据此把强相关样本单独归集、把 BGM 等弱相关样本的占比加以管控。这构成一个非常明确的三维过滤空间：时序同步（TS-TalkNet）× 语义相关（MLLM 判断）× 音频质量（MLLM 打分）。特别值得注意的是它对弱相关样本不是一刀切剔除而是「控制比例」——承认非画内音（BGM、旁白）在真实内容中普遍存在且模型需要具备生成能力，只是不能占比过高以免破坏音画因果关系。这比 Ovi 等工作「只做时序过滤、语义靠 caption 隐式绑定」的做法成熟。
【OmniCustom】只做时序（SyncNet 双阈值），无语义匹配过滤——但其任务性质（单人说话头，音频必为画中人所说）使语义错配风险天然很低，属合理省略。
【ITS-JAVG —— 在推理侧把二者拆成不同验证器并证明其冲突】
验证器组合中，JavisScore 度量细粒度时序同步，AVHScore 与 ImageBind 度量语义一致性——论文的核心实证正是这两类目标之间存在非对称 trade-off：优化时序同步会牺牲语义一致性，反之亦然，单一验证器无法兼顾。这为「时序与语义必须分离处理并显式平衡」提供了直接的量化证据，也是本批工作中对该问题最有理论价值的贡献。其 ARW 算法本质上就是在时序类与语义类奖励之间做自适应权衡。
【CCL】在评测中同时报告 Sync-C/Sync-D（时序）、DeSync（时序）与 IB（ImageBind，语义）——指标层面区分了二者，但数据侧无对应的分离过滤[不确定]。
【StreamChar / Baton】未做此区分[不确定]。Baton 的 planned tokens 试图统一编码语义蓝图，RS-RoPE 负责时序注入——架构上有隐含分工（语义靠 planner、时序靠 RoPE），与 Ovi 的「RoPE 管时序、cross-attention 管语义」思路一脉相承。

### 音频质量过滤（SNR、静音检测与静音占比阈值、无音轨剔除、画外音源剔除、背景音乐分离） ⚠️

【ALIVE —— 用 MLLM 替代传统信号处理指标（本批最具方向性意义的做法）】
原文：「We utilize an MLLM model ... to perform audio filtering based on two criteria. Firstly, for audio quality, the model assigns a score to each audio, allowing us to discard samples with significant background noise. Secondly, for audio-visual coherence, we assess the correlation ...」
(1) 音频质量：由 MLLM 直接对每段音频打分，据此剔除背景噪声显著的样本——注意这里没有出现 SNR、PESQ、DNSMOS 等任何传统音频质量指标，完全交给多模态大模型「听」并判断。这是本批中最能代表 2026 年趋势的单点：音频质量评估从信号处理指标转向大模型语义判断。
(2) 音画一致性：评估相关性，把强相关样本单独归集，管控 BGM 等弱相关数据的占比「to optimize the dataset's composition」。
(3) 未提及：SNR 阈值、静音检测与静音占比、无音轨剔除（但管线第一步已筛选「videos with audio」，等价于剔除无音轨）、音源分离（source separation 完全未提及）、削波/失真检测[全部不确定]。
【NAVA —— AudioBox Aesthetic 作为音频质量算子】
音频质量过滤使用「AudioBox Aesthetic scores」（Meta 提出的音频美学评估模型，输出多维音频质量分）。这是本批中唯一点名使用专业音频质量模型的工作，比传统 SNR 更贴近人类感知。另有 YAMNet 音频分类做类别标注。阈值未公开[不确定]。未提及 SNR、静音、音源分离[不确定]。
【OmniCustom —— 仅格式统一，无质量过滤】
「We extract audio files from videos and unify them into 16kHZ」——音频统一重采样至 16kHz，无音质过滤描述[不确定]。其过滤全靠 SyncNet 双阈值（间接排除音轨异常样本：音轨损坏或静音会导致 SyncNet confidence 极低而被剔除）。这是一种「用同步检测隐式兼做音频有效性检测」的省事做法。
【StreamChar / CCL / Baton】未描述音频质量过滤[不确定]。CCL 使用 WavCaps 与 VGGSound 做音频预训练，二者自带质量筛选；Baton 使用 AudioCaps + WavCaps 同理。三者均把音频质量控制外包给上游。
【ITS-JAVG】推理侧用 ImageBind-TA 评估文本与生成音频的语义连贯性，非音质指标；无音质验证器——这本身是个缺口：其六个验证器中没有一个专门度量音频保真度/音质，可能导致搜索出语义对但音质差的样本[论文未讨论此点，属本调研观察]。
【总体判断】音频质量过滤在本批中最显著的变化是「去信号处理化」：ALIVE 用 MLLM 打分、NAVA 用 AudioBox Aesthetic（学习型感知模型），二者都绕开了 SNR 这类传统指标。同时七项中无一使用音源分离（Demucs 等）——反映主流共识是保留原生混合音轨，因为目标就是生成含语音+音效+音乐的完整音景。

### 语音/音效/音乐的分类与分别处理策略 ⚠️

【ALIVE —— 顶层 speaking/non-speaking 二分驱动全局（本批最系统）】
(1) 数据组织的最顶层就是音频类型二分：「First, we make a top-level distinction between core scenarios: 'speaking' and 'non-speaking' scenario.」——先按「有无人说话」把语料劈成两半，再在其上叠加九大领域的三级标签。这个顺序很关键：说明音频类型是比视觉领域更高优先级的组织维度，符合 AV 模型的本质需求（说话场景需要唇同步与音色一致，非说话场景需要事件级音画对齐，二者的建模难点完全不同）。
(2) caption 层面对应分流：<W> 标记逐字语音、<I> 标记非语音声学事件、Subjects 中的 acoustic profiles 描述音色属性。
(3) BGM 单独处理：作为「弱相关」音频的代表，其占比被主动管控而非剔除。
(4) 音频塔训练的两阶段类型切换：Stage I 纯语音（700k hours 转写语音，1 epoch）→ Stage II 语音+真实音景混合（5k hours 高质语音 + 111k hours 视频音轨，10 epochs）。
(5) Level-1 九大领域中 Home Sounds 与 Sound Effects 两类专属非语音音频，且每个 Level-3 视觉标签绑定音频关键词——视觉概念与声音概念成对索引。
【NAVA —— 五类音频标签（本批分类最细）】
YAMNet + omni-modal tagger 划分五类：single-speaker speech、multi-speaker speech、ambient sound、music、singing。相比常见的「语音/音效/音乐」三分，这套五分法的两处细化很有针对性：
- 单人语音 vs 多人语音分开：直接服务于其多说话人 timbre 控制能力（Timbre-in-Context Conditioning 需要知道有几个说话人、各自的 speech span 在哪）
- music 与 singing 分开：歌唱同时具备语音（有歌词、有音色）与音乐（有旋律、有节奏）双重属性，单列一类避免污染纯语音与纯音乐的建模
caption 音频板块也按 speech / SFX / music / ambient sound 四类描述。训练阶段用「audio-only : audio-visual」比例调度（Stage 1 为 3:1、Stage 2 为 1:2）间接控制音频类型的曝光。
【OmniCustom —— 单一类型（纯语音）】全部为单人说话语音，无音效/音乐处理。音频 caption 按说话人属性组织（age/gender/accent/pitch/prosody/emotion/speaking rate）。是本批中音频类型最单一的一项，但也因此其音色控制做得最深。
【StreamChar —— 语音为主】Emilia（纯语音多语种数据集）做编排器预训练，联合训练数据为角色说话视频。无音效/音乐类型设计[不确定]。
【CCL —— 预训练通用音频、联合训练偏语音】音频扩散预训练用 WavCaps（音频描述）+ VGGSound（视频音效），覆盖通用音效；联合训练数据（OpenHumanVid + 访谈/短剧/电影）以对白为主。形成「通用音频打底 + 语音场景微调」的结构。其多任务概率分布中 text-to-audio 占 0.1、video-to-audio 占 0.15，说明纯音频生成能力被显式保留。
【Baton】AudioCaps + WavCaps 提供通用音效/音景，OpenHuman-Vid 提供人声，无显式类型分流策略[不确定]。
【ITS-JAVG】所测两个基座音频类型侧重不同：MMDisCo 基于 VGGSound（音效为主）、JavisDiT 覆盖更广；其验证器中 ImageBind-TA 与 AVHScore 对语音与音效一视同仁，未做类型分流[不确定]。

## 训练配合

### 多阶段训练课程与数据课程调度（阶段划分依据：分辨率/时长/质量分/模态；低清→高清、图像→视频、短→长） ⚠️

【ALIVE —— 七阶段课程（本批最长最细，且音频与音视频两条线交织）】
按论文给出的阶段表：
① T2A Stage I（音频塔预训练）：700k hours 转写语音 / 384M audio samples，1 epoch，lr 5e-5 —— 纯语音打底
② T2A Stage II（音频塔混合微调）：19M samples（5k hours 高质语音 + 111k hours 视频音轨），10 epochs，lr 5e-5 —— 引入真实音景，注意 epoch 数从 1 跃升到 10，高质量数据被反复利用
③ T2VA Joint-Train（音视频联合训练）：11M samples @480p/24fps，1.2 epochs，lr 1e-4 —— 学习率最高的阶段，跨模态连接从零建立
④ T2VA+I2VA（加入图像条件）：同 11M samples，0.3 epochs，lr 5e-5 —— 短暂的能力扩展
⑤ Continue-Training（均衡数据继续训练）：4.3M balanced samples，3 epochs，lr 5e-5 —— 用经 domain 配比调整的均衡子集重训 3 轮，是纠正分布偏斜的关键阶段
⑥ SFT（监督微调）：5M samples，高美学数据与写实数据按 3:1 混合，0.5 epochs，非对称学习率（video 1e-5 / audio 1e-6，音频侧低一个数量级以保护已练好的音频能力）
⑦ 1080p Refiner（高清精修）：0.7M high-clarity samples，1 epoch，lr 5e-5
另有并行的 Character-driven 训练（0.8M reference-paired samples）。
课程划分依据横跨：模态（先音频后音视频）→ 条件类型（T2VA→+I2VA）→ 数据分布（原始→balanced）→ 质量（→高美学 SFT）→ 分辨率（480p→1080p）。是本批中维度最多的课程设计。非对称学习率（视频 1e-5 / 音频 1e-6）是一个值得注意的细节：反映联合训练中两模态的成熟度不同，需差异化保护。
【NAVA —— 三阶段，以模态配比为划分依据】
① Stage 1：audio-only : audio-visual = 3:1 —— 音频侧主导，先把音频生成能力练强
② Stage 2：比例反转为 1:2，「train on high-quality audio data together with the full audio-visual dataset」——高质量音频 + 全量音视频数据
③ Stage 3：在筛选出的 160K 高质量音视频数据上微调
总算力：「Approximately 107,520 H100 GPU-hours」（约 10.75 万 H100 卡时）跨三阶段——这是本批中唯一公开总算力的工作，极具参考价值。以 6.3B 模型 + 15M 片段的规模看，这个数字约合 1120 张 H100 训练 4 天，属中等工业规模。
划分依据纯粹是「模态配比 + 数据质量」，不涉及分辨率或时长课程。
【CCL —— 两阶段，极简】
① 音频扩散预训练：160,000 steps，batch 3200，lr 1e-4
② 联合音视频训练：cross-modal attention 参数用 lr 1e-4、其余参数用 lr 1e-5（差异化学习率，新增的跨模态模块学得快、已预训练部分学得慢）；最终设置 batch 320、12,000 steps；消融阶段 batch 128、6,000 steps
视频流由 Wan2.1-14B 预训练权重初始化，音频流从零构建（架构相似但通道维度缩减）。整个联合训练只有 12,000 步——这是其「省算力」主张的核心支撑：相比动辄数万步的同类工作，CCL 用一个数量级更少的训练量达到 SOTA。GPU 数与总卡时未公开[不确定]。
【StreamChar —— 两阶段预训练 + 两阶段蒸馏】
① Pretraining Stage 1（编排器）：Emilia 数据集，80k steps，batch 640，lr 6e-5
② Pretraining Stage 2（联合训练）：100k steps，batch 128，lr 1e-5
③ Distillation Stage I：600 steps，压缩为 4 步生成器
④ Distillation Stage II：400 steps，在线 chunk rollout 下做一致性微调，student lr 2e-6、fake score network lr 4e-7
蒸馏阶段仅 1000 步总量，相比预训练的 18 万步几乎可忽略，说明蒸馏是低成本的最后一公里工程。
【Baton —— 三阶段渐进式课程（围绕 planner 与 DiT 的解耦训练）】
① Stage 1（VA-Planner 预训练）：从冻结的 SigLip2 与 WavTokenizer 倒数第二层提取目标连续特征，用 L2 回归训练规划器
② Stage 2（DiT 适配）：DiT 接收经 Latent-MLPs 投影的 ground-truth 特征，用 flow matching 损失训练——刻意使用干净的编码器特征「to avoid confounding planner prediction noise」（避免规划器预测噪声的混淆）
③ Stage 3（联合微调）：VA-Planner 冻结、DiT 可训，DiT 改为接收规划器的预测特征——「bridging the gap between clean features and imperfect predictions」（弥合干净特征与不完美预测之间的鸿沟）
总计 10 epochs、batch 1 per GPU、lr 1e-5。这个「先用干净标签训、再切换到模型预测」的三段式，本质上是解决 exposure bias 的课程设计（与 scheduled sampling 同理），是本批中课程设计逻辑最精巧的一项。
【OmniCustom】训练阶段划分未详述[不确定]；采用 LoRA 微调（identity LoRA + timbre LoRA）+ 对比学习目标 + flow matching 的组合，属于在已有基座上的轻量适配而非多阶段预训练。
【ITS-JAVG】无训练课程（training-free）。

### 数据配比随训练阶段的变化（预训练/退火annealing/SFT高质量子集） ⚠️

本批工作中配比调度的显式程度差异极大，ALIVE 与 NAVA 给出了可直接借鉴的数字：
【ALIVE —— 三处显式配比】
(1) 音频阶段的语音/音景切换：Stage I 为 700k hours 纯转写语音（384M samples）；Stage II 骤降到 5k hours 高质语音 + 111k hours 视频音轨（19M samples）。语音时长从 70 万小时压到 5 千小时（1/140），同时引入 11.1 万小时视频原生音轨——这是一次剧烈的分布切换，从「海量语音学发音」转向「精选语音保能力 + 海量真实音景学场景」。Stage II 训 10 epochs 说明这批混合数据被高度重视。
(2) SFT 阶段的美学配比：「high-aesthetics data mixed at 3:1 ratio with realistic data」——高美学数据与写实数据 3:1 混合。这是本批中唯一明确公开的 SFT 数据配比数字，含义是：SFT 阶段刻意让高美学内容占主导（75%），但仍保留 25% 写实内容以防模型过度「网红滤镜化」而丧失真实感。
(3) Continue-training 的均衡配比：11M → 4.3M「balanced samples」，依据是九大领域三级标签体系与「概念频次 + 预期应用场景」两条先验。具体各类占比未公开[不确定]。
(4) 非对称学习率作为隐式配比：SFT 阶段 video lr 1e-5 / audio lr 1e-6，音频侧学习率低一个数量级——在损失权重之外，用学习率差异实现模态间的「更新配比」控制。
【NAVA —— 模态配比的显式反转】
Stage 1 audio-only : audio-visual = 3:1 → Stage 2 反转为 1:2 → Stage 3 纯高质量音视频（160K）。这条曲线的逻辑很清晰：先用大量纯音频数据把音频生成能力练扎实（音频数据便宜且量大：原始池有 20M audio clips），再逐步把重心移到昂贵的配对音视频数据上，最后在千分之一的精品子集上收尾。三阶段的绝对数据量与各来源（Koala-36M / TED / 影视）在各阶段的占比未拆分披露[不确定]。
【CCL —— 多任务概率配比（本批唯一给出任务级采样概率的工作）】
训练时按概率随机采样任务类型：
- text/image-to-video：0.1
- text-to-audio：0.1
- 联合生成（joint）：0.6
- audio-to-video：0.15
- video-to-audio：0.15
合计 1.0。联合生成占 60% 为绝对主体，两个跨模态转换任务（A2V、V2A）各占 15%——这两个任务的作用是强化跨模态条件依赖（模型必须真正「看懂」一个模态才能生成另一个），是防止双流退化为两个独立生成器的关键正则。单模态任务各占 10%，用于保活单模态能力。这套配比是 CCL 能用 4M 数据达到 SOTA 的重要原因之一：通过多任务混合，同一批数据被以五种不同方式反复利用，等效数据效率大幅提升。
【Baton】三阶段的数据配比未拆分[不确定]；1.5M 片段中 OpenHuman-Vid / AudioCaps / WavCaps / 互联网视频四路的占比未公开[不确定]。
【StreamChar】Emilia（纯语音）用于 Stage 1、音视频数据集用于 Stage 2，两阶段数据完全不同源，无混合配比[三个音视频数据集之间的配比不确定]。
【OmniCustom】单一数据集无配比问题；但其「前 4 秒参考 + 后 5 秒训练」的固定切分是一种时间维度上的配比设计。
【ITS-JAVG】无训练配比；但其 ARW 算法解决的正是「多个奖励信号如何加权聚合」——与训练数据配比在数学结构上同构（都是多来源信号的加权组合优化），其按方差自适应加权的思路理论上可迁移到数据配比调度（按各 domain 的梯度方差自适应调整采样权重），论文未做此引申。

### 后训练数据（SFT精选集规模与筛选标准、偏好对数量与标注方式、reward model训练数据） ⚠️

本批工作的后训练形态以「高质量子集 SFT」为主，无一采用偏好学习/RLHF：
【NAVA —— 千分之一精选 SFT（本批最典型）】
从 15M 训练语料中经「multi-operator collaborative filtering」（多算子协同过滤）筛出 160K high-quality samples，筛选标准为「accurate captions and strong audio-visual alignment」（caption 准确 + 音视对齐强）。SFT 保留率约 1.07%——即千条里取十条。且该子集的 caption 由更贵的 Gemini-3-Pro 重新生成（「to produce more accurate, structured, and temporally grounded audio-visual captions」），而非沿用 Flash 生成的原 caption。这个「精选子集 + 升级 caption」的双重提纯是很值得注意的做法：SFT 阶段不仅换数据，还换标注质量。具体阈值未公开[不确定]。无偏好对、无 reward model、无 RLHF[不确定]。
【ALIVE —— 多层后训练但无偏好学习】
(1) Continue-Training：4.3M balanced samples，3 epochs——用领域均衡后的子集纠正分布偏斜，介于预训练与 SFT 之间的过渡阶段。
(2) SFT：5M samples，高美学与写实数据 3:1 混合，0.5 epochs，非对称学习率（video 1e-5 / audio 1e-6）。注意只训 0.5 epoch——SFT 阶段刻意训得很浅，避免过拟合到高美学分布而损失多样性。
(3) 1080p Refiner：0.7M high-clarity samples，1 epoch——独立的高清精修模块，本质上也是一种针对性后训练。
(4) Character-driven：0.8M reference-paired samples（含 cross-pair 与 in-pair augmentation 构造的数据）——针对参考角色能力的专项后训练。
无 DPO/RLHF/偏好对/reward model 描述[不确定]。人工评测（vs Veo 3.1 / Kling 2.6 / Wan 2.6 / Sora 2 / LTX-2 的盲测）仅用于评估，未回收为训练信号[不确定]。
【StreamChar —— 蒸馏作为后训练】
两阶段蒸馏：Stage I 600 steps 做步数压缩（→4 步生成器）；Stage II 400 steps 做 rollout consistency（在线 chunk rollout 下微调 student），student lr 2e-6、fake score network lr 4e-7。Stage II 的训练数据是学生模型自己的在线 rollout——属于 on-policy 自生成数据，是本批中唯一涉及「模型产出回灌」的后训练形式。消融表明单阶段蒸馏效果劣于两阶段。
【OmniCustom —— LoRA 适配即后训练】
在联合音视频基座上训练 identity LoRA 与 timbre LoRA，配合对比学习目标（有参考条件的预测流为正例、无参考条件的为负例）与 flow matching。训练数据即 OmniCustom-1M 的配对样本。无独立的偏好数据或精选子集[不确定]。
【CCL / Baton】无后训练阶段[不确定]。CCL 的联合训练 12,000 步本身即最终阶段；Baton 的 Stage 3 联合微调可视为广义后训练（冻结 planner、微调 DiT），但无偏好数据。
【ITS-JAVG —— 用推理时搜索替代后训练（本批最有启发的一点）】
其核心主张恰恰是：不需要后训练也能提升质量——用多验证器 + Best-of-N/EvoSearch 在推理时择优（JavisDiT 5 samples、MMDisCo 10 samples），效果可与训练侧优化相比。这实际上提出了一条替代路径：与其花成本做 SFT/RLHF，不如在推理时多采样再用验证器筛。反过来看，其筛选出的高分样本天然是构造 SFT/偏好数据的现成来源（高分样本作正例、低分作负例），但论文未走这一步。
【共性缺口】七项中零项使用偏好对/RLHF/DPO——这与同期闭源工业模型（Sora 2、Seedance 等普遍有 RLHF 环节）形成鲜明对比，反映开源/学术侧 JAVG 在偏好对齐上仍是空白地带。ITS-JAVG 揭示的 verifier hacking 问题也预示：若贸然用自动验证器构造偏好数据做 RLHF，很可能训出一个只会讨好判官的模型。

### 数据处理基础设施与吞吐（NeMo Curator/Data-Juicer/自研，GPU加速比，处理规模，成本） ⚠️

披露稀疏但有两个高价值数据点：
【NAVA —— 唯一公开总算力与切分基础设施】
(1) 总算力：「Approximately 107,520 H100 GPU-hours」（约 107,520 H100 卡时）跨三阶段训练。这是本批唯一的算力数字，也是同期 JAVG 论文中少见的透明披露。折算：约合 1120 张 H100 连续训练 4 天，或 512 张 H100 训练约 8.75 天。对于 6.3B 模型 + 15M 片段的配置，属中等工业规模——作为对照，同期动辄数十万乃至百万卡时的大厂视频模型，NAVA 算是相当节制。
(2) 数据切分基础设施：「raw videos are first segmented at scale with a Hadoop-based pipeline」——用 Hadoop 分布式管线处理 100M 视频的切分。选择 Hadoop（而非 Spark、Ray 或 GPU 加速框架）说明其数据处理以 CPU 密集的 I/O 与转码为主。
(3) 去重基础设施：VideoCLIP 嵌入提取 + 「large-scale k-means clustering」（大规模 k-means 聚类）——在 1 亿量级视频上做嵌入聚类本身是显著的工程挑战，但具体实现（是否用 FAISS、分布式方案、耗时）未披露[不确定]。
(4) 推理侧：8 卡序列并行下生成约 1 分钟视频；权重 24GB（bf16）/ 约 7GB（fp8 量化）。
【ALIVE —— FAISS 检索基础设施（推理侧）】
构建稠密向量索引使用 FAISS：「We first construct a dense vector index ... using FAISS ... extract sound events from the user's prompt and query this database for the nearest neighbors based on cosine similarity」，检索相似度阈值 τ>0.85。这是推理时的 prompt 检索增强而非训练数据处理，但同样属数据基础设施。训练侧的集群规模、卡型、卡时、数据吞吐、处理成本全部未公开[不确定]。考虑到其 384M audio samples + 11M video samples 的规模，算力投入应远超 NAVA，但无数字。
【CCL —— 用训练规模间接体现「省」】
虽未公开 GPU 数与卡时[不确定]，但其核心卖点就是资源效率：训练数据 4M（vs Ovi 30.7M、MOVA 50M），联合训练仅 12,000 steps @batch 320。论文称「achieves state-of-the-art performance with substantially reduced computational resources compared to recent academic approaches」。以 batch 320 × 12,000 steps ≈ 384 万样本次计算，这个训练量确实极小。视频流由 Wan2.1-14B 初始化是其能省的前提——站在强基座上只需训跨模态连接。
【StreamChar —— 推理侧实时性是其核心工程指标】
单张 H100 上实时运行（real time on a single H100 GPU），每 chunk 33 帧 @24fps。这是本批中唯一以「实时性」为第一工程目标的工作，其两阶段蒸馏（4 步生成器）正是为此。数据处理侧无基础设施披露[不确定]。
【OmniCustom / Baton】无数据基础设施披露[不确定]。Baton 仅给出 batch size 1 per GPU、lr 1e-5、10 epochs——batch 1 per GPU 说明单卡显存吃紧（Qwen3-8B planner + DiT 同时在显存中），但 GPU 总数未说明[不确定]。
【ITS-JAVG —— 推理成本即其代价】
ITS 的本质是用推理算力换质量：JavisDiT 每 prompt 生成 5 个候选、MMDisCo 每 prompt 10 个，再加上 6 个验证器的评分开销——推理成本相比单次生成放大 5–10 倍以上。论文的价值主张是这笔开销远低于重新训练模型的成本。具体延迟/吞吐数字未提取到[不确定]。
【共性】七项中无一使用 NeMo Curator、Data-Juicer 等现成数据处理框架，均为自建或直接复用公开数据集；仅 NAVA 点名了 Hadoop 与 k-means 这类通用基础设施。

## 效果对比

### 数据策略消融的量化影响（区分：过滤严格度ablation / caption密度风格ablation / 数据配比ablation，及对应评测指标） ⚠️

本批工作的消融普遍集中在架构模块，真正的数据消融极少——这是七项共同的短板：
【无数据消融的六项】
- ALIVE：「The paper contains no ablation studies isolating the impact of individual pipeline stages」——六阶段数据管线中没有任何一级被单独消融验证。考虑到其管线是本批最复杂的（尤其 SubjectID correction 五步子管线成本很高），缺少「做/不做 SubjectID correction 的效果对比」是最令人遗憾的缺失。同理，13 维质量分类器、六档清晰度过滤、九大领域配比调整的收益均无量化。
- NAVA：「No ablation studies on data composition are presented」——消融集中在架构（Align-then-Fuse 层数）与引导机制，无数据侧消融。3:1 → 1:2 的模态配比调度、160K SFT 子集的收益均无对照实验。
- OmniCustom：无数据消融[不确定]；其三条过滤规则（SyncNet 双阈值、美学 <0.3、时长 <10s）的各自贡献未验证。
- StreamChar：消融见 Tables 1–2，全部为架构/系统侧——最有价值的一项是 sink chunk（持久视觉锚点）消融：去掉后 drift 从 0.0067 恶化到 0.0304，劣化约 4.5 倍，量化证明了视觉锚点对长程一致性的决定性作用。另有单阶段 vs 两阶段蒸馏的对比。无数据侧消融。
- Baton：Table 2 消融 VA-Planner 各组件、Table 3 消融 RS-RoPE 与 backbone 变体——全部架构侧。但有一项与数据/标注范式高度相关的设计选择被论证：连续特征回归 vs 离散 token 预测，论文主张「regressing continuous features preserves richer semantic structure」，若 Table 2 含该对照则构成对「语义中介表示形式」的实证[具体数值未提取到，不确定]。
- CCL：Table 2 报告消融结果，指标为 WER / Sync-C / Sync-D / DeSync / IB 五项，消融对象为 TARP、LCT、DCR、UCG 四个模块——架构侧。但 Table 1 的横向对比（CCL 4M vs Ovi 30.7M vs MOVA 50M）虽非严格消融，却是本批中最有力的「数据效率」证据（见 quality_over_quantity_evidence）。消融训练配置为 6,000 steps @batch 128，最终配置为 12,000 steps @batch 320。
【唯一系统性研究「打分器选择」影响的：ITS-JAVG】
虽不是训练数据消融，但其对「验证器组合」的系统性消融，在方法论上等价于「数据过滤打分器组合」的消融，是本批最接近数据消融的研究：
(1) 单验证器 vs 多验证器：证明「single-verifier guidance effectively improves its intended evaluation metrics, yet fails to achieve a balanced improvement across all metrics」——单一打分器只能提升它自己度量的指标，其余指标反而可能退化（非对称 trade-off）。
(2) verifier hacking 实证：搜索算法会「exploit blind spots」（利用打分器盲区），即在打分器看不见的维度上退化。
(3) 最优验证器组合的识别：论文的贡献之一就是找出能带来均衡提升的验证器组合。
(4) ARW 的收益：相比固定权重的奖励聚合，自适应加权（按各奖励的方差归一化）带来更均衡的提升。
这套结论对训练数据管线的直接启示：单一美学分/单一 SyncNet 阈值做过滤，会让保留数据系统性偏向该打分器的偏好并在其盲区（如音质、语义合理性）上劣化；多算子协同（如 NAVA 的 SyncNet+SyncFormer+ImageBind 三重对齐算子）在原理上更稳健。
【总体判断】2026 年上半年这批 JAVG 工作在数据消融上几乎是空白：无一项做了「过滤严格度 ablation」「caption 密度/风格 ablation」或「数据配比 ablation」。数据策略的有效性主要靠工程直觉与最终指标背书，缺少可追溯的因果证据。这与同期数据侧的工程复杂度（ALIVE 六阶段管线、NAVA 多算子过滤）形成强烈反差——管线越来越复杂，但没人验证每一环节值不值。

### 质量vs数量的证据（小而精数据超越大而杂的案例） ⚠️

【CCL —— 本批最直接、最量化的「小而精胜过大而杂」证据】
Table 1 明确并列三方训练数据量：CCL 4M vs Ovi 30.7M vs MOVA 50M——CCL 用 Ovi 约 1/8、MOVA 约 1/12 的数据量，配合仅 12,000 步的联合训练（batch 320），达到 SOTA 性能。论文自述「achieves state-of-the-art performance with substantially reduced computational resources compared to recent academic approaches, while requiring minimal training data by leveraging pre-trained diffusion models」。
需要客观指出的是，CCL 的高效并非纯粹来自「数据质量高」，而是三个因素叠加：
(1) 强基座迁移：视频流由 Wan2.1-14B 预训练权重初始化——大部分视觉先验是免费继承的，其 4M 数据只需学跨模态连接而非从头学视频生成。
(2) 窄而深的 domain：数据来源（OpenHumanVid + 访谈 + 短剧 + 电影）高度集中于「有人说话的叙事内容」，牺牲广度换取该 domain 内的密度。
(3) 多任务复用：五种任务按概率（joint 0.6、A2V 0.15、V2A 0.15、T2V/I2V 0.1、T2A 0.1）混合采样，同一批数据被以五种方式反复利用，等效数据效率成倍放大。
因此更准确的表述是：「强基座 + 窄 domain + 多任务复用」可以把数据需求压缩一个数量级——这对资源有限的团队是极有价值的路线参考。
【NAVA —— 参数效率侧的证据 + 千分之一 SFT 提纯】
(1) 参数效率：6.3B 参数在 Verse-Bench 上刷新 Sync-C / Sync-D / 视频质量 / 音频 WER 的 SOTA，同时「using 2× to 5× fewer parameters than open-source baselines」（参数量比开源基线少 2–5 倍）。
(2) 数据提纯的极端比例：15M → 160K SFT 子集（保留率约 1.07%），且该子集的 caption 由更贵的 Gemini-3-Pro 重新生成。这个「千分之十精选 + 标注升级」的做法本身就是质量优先的强表态——但论文未做「有/无 160K SFT 阶段」的对照实验[不确定]，收益无法量化。
(3) 严格过滤：100M 视频 → 15M（约 15% 保留率），说明团队愿意丢掉 85% 的数据。
【OmniCustom —— 严格筛选换定制质量】
从 SpeakerVid-5M 的 5.2M clips / 8,000 hours 筛到 1M clips / 2,500 hours（片段保留率 19%、时长保留率 31%），仅保留同步好（SyncNet |offset|≤3 且 conf>1.5）、美学分 ≥0.3、时长 ≥10 秒的片段。为身份+音色定制这一高精度任务准备高纯度数据，是典型的质量优先取向。但无「宽松 vs 严格过滤」的对照[不确定]。
【ALIVE —— 反向证据：规模仍是主要杠杆】
ALIVE 走的是相反路线：700k hours 语音、11M 联合样本、384M 音频样本——用绝对规模取胜，同时用极其复杂的六阶段管线保证质量。其「11M → 4.3M balanced」的均衡化削减（保留 39%）与「5M SFT 中高美学:写实 = 3:1」的配比，说明它在规模基础上做精细的分布控制，属于「规模 + 质量双管齐下」而非二选一。
【StreamChar —— 训练短、推理长的泛化证据】
训练数据全部 ≤20 秒，却能生成 5 分钟连续流——这是另一种形式的「以小博大」：不靠长视频数据，而靠架构机制（progress-aware pointer + persistent visual anchor）实现长程泛化。sink chunk 消融（drift 0.0067 vs 0.0304）证明这条路可行。对数据侧的启示是：某些能力（长程一致性）用架构解决可能比用数据（收集长视频）更经济。
【ITS-JAVG —— 最激进的「零训练数据」立场】
完全 training-free，用推理时多验证器搜索获得质量提升——把「质量」从数据侧和训练侧一起转移到了推理侧。这是本批中对「更多数据 = 更好模型」这一假设最彻底的挑战。但代价是推理成本放大 5–10 倍，且论文自己揭示了验证器可被钻空子的风险。
【Baton】1.5M clips 为本批联合生成工作中最小规模，其性能提升归因于 VA-Planner 的语义规划而非数据——同样属于「用方法弥补数据」的路线，但无数据量对照实验[不确定]。

### 训练数据domain分布与评测基准类目体系的对齐关系（如VABench七大类） ⚠️

【ALIVE —— 训练标签体系与评测类目的部分对齐（本批最接近闭环的一项）】
(1) 训练侧类目：speaking / non-speaking 顶层二分 + 九大 Level-1 领域（Animals、Home Sounds、Entertainment、Environment、Food、Nature、Sound Effects、Vehicles、Sports）的三级层次体系，且各类占比按「概念频次 + 预期应用场景」主动调整。
(2) 评测侧类目：Alive-Bench 1.0 含 264 general prompts，「covers a wide range of scenarios, including single-person speech, multi-people conversations, sports, daily activities, animals」——单人语音、多人对话、运动、日常活动、动物等；另有 90 reference-character prompts 专测参考角色能力。评测覆盖中英双语。
(3) 对齐关系：评测类目与训练 Level-1 领域有明显交集（sports↔Sports、animals↔Animals、daily activities↔Home Sounds/Food），且「单人语音 vs 多人对话」的区分直接对应训练侧的 speaking/non-speaking 二分与 SubjectID correction 所针对的多人场景。可以说 ALIVE 是本批中训练分布与评测类目对应关系最清晰的工作。
(4) 22 项细粒度指标分六大类[具体类别未提取到，不确定]；未见按 domain 分组报告的结果，因此对齐仍停留在「类目设计层面」而非「结果分组报告层面」[不确定]。
【NAVA —— 训练分类与评测基准不对齐】
训练侧用 YAMNet 五类音频标签（single-speaker speech / multi-speaker speech / ambient sound / music / singing）组织语料；评测则用 Verse-Bench（源自 UniVerse-1）与 Seed-TTS，二者均为「质量维度」评测而非「内容类目」评测。Verse-Bench 报 Sync-C / Sync-D / 视频质量 / 音频 WER，Seed-TTS 报语音指标。训练侧的五类音频标签在评测中无对应分组[不确定]。用户研究覆盖视频质量、音视频同步、音频保真度、音色可控性四维——其中「音色可控性」与训练侧的 single-/multi-speaker 区分及 Timbre-in-Context 设计直接呼应，是唯一的对齐点。
【OmniCustom —— 评测集设计与训练数据刻意「反向对齐」】
评测集 100 例的构成很讲究：30 位训练集外真人 + 70 段 YouTube 视频，性别比 1:1。刻意选取训练数据中不存在的身份，正是为了验证零样本音色/身份迁移——即评测集在身份维度上刻意与训练分布不重叠。这是一种「反向对齐」：不追求分布一致，而追求分布外泛化的验证。性别 1:1 的控制则是评测侧唯一的显式人口统计学配比。
【CCL —— 指标体系细但无类目对齐】
评测指标 WER / Sync-C / Sync-D / DeSync / IB 五项，覆盖语音可懂度、唇音同步（SyncNet 系两项）、通用音视同步（Synchformer 系）、跨模态语义（ImageBind）——指标维度划分很细且恰好对应「时序同步 vs 语义匹配」的分离思想。但训练数据的 domain（访谈/短剧/电影）在评测中无对应分组报告[不确定]。
【StreamChar —— 评测设计紧扣其长时流式定位】
EMTD 数据集上「150 clips generating 10s audio-video pairs」（短时能力）+「50 clips paired with randomly sampled transcripts (>300 words) to produce 5-minute continuous streams」（长时流式能力）——评测刻意设计了短/长两档，其中长档（5 分钟、>300 词台词）远超训练数据的 20 秒上限，是对泛化能力的针对性检验。人评 24 人 × 50 案例。评测类目与训练数据的 domain 分布无对应关系（均为人物说话，单一类目）。
【Baton —— 自建 Sem100 补充语义推理维度】
除 Verse-Bench 外自建 Sem100：「100 unseen videos (10 seconds long) with more complex prompts, selected from the internet to assess the semantic reasoning capability」——100 段未见过的 10 秒视频，配更复杂的 prompt，专门评估语义推理能力。这个基准是为验证 VA-Planner 的语义规划优势而量身定制的，即评测基准与方法论主张对齐（而非与训练数据分布对齐）。构建方法细节未披露[不确定]。
【ITS-JAVG —— 评测基准与验证器的循环风险（本批最值得警惕的一点）】
评测用 JavisBench-mini（测 JavisDiT）与 VGGSound test set（测 MMDisCo）。存在一个结构性问题：其验证器（JavisScore、ImageBind、AVHScore 等）与评测指标高度重叠甚至同源——用某指标当验证器引导搜索，再用该指标评估效果，必然「提升」。论文对此有清醒认识，这正是其提出 verifier hacking 概念与多验证器组合方案的动因：只有当被优化的验证器集合与被评估的指标集合不完全重合时，改进才是真实的。这一方法论警示对数据侧同样成立——若用与最终评测同源的模型做数据过滤（例如用 SyncNet 过滤数据、又用 Sync-C 评测），得到的提升有部分是循环论证。本批中 NAVA（用 SyncNet 过滤、用 Sync-C/Sync-D 评测）与 OmniCustom（同样情形）都存在这一潜在循环，论文均未讨论[不确定]。

## 不确定字段

以下字段的调研信息部分不确定（⚠️ 标注来源）：

- organization
- openness
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
