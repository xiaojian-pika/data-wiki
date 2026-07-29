# 合并条目：① Mochi 1（含 mochi-1-preview 与 AsymmVAE）② MAGI-1（含 24B / 4.5B 变体、Distill 蒸馏版，以及 2026 年的 MAGI-1.1 24B）③ Motif-Video 2B（含 T2V 与 I2V 扩展）。三者均为开源纯视觉视频生成基础模型，放在一起对照的核心价值是：数据披露粒度从「几乎为零」（Mochi 1）→「方法级详尽但无数值」（MAGI-1）→「工具级+参数级详尽且明确以小数据取胜」（Motif-Video 2B）呈现出 2024→2026 的清晰演进曲线。

> 主题: 视频生成模型（含音视频同时生成）的数据处理：数据清洗流程、数据分布、打标方式

[← 返回首页](../index.md)

**目录**: [基本信息](#基本信息) · [数据规模与分布](#数据规模与分布) · [清洗流程](#清洗流程) · [打标方式](#打标方式) · [音视频对齐](#音视频对齐) · [训练配合](#训练配合) · [效果对比](#效果对比)

## 基本信息

### 名称

合并条目：① Mochi 1（含 mochi-1-preview 与 AsymmVAE）② MAGI-1（含 24B / 4.5B 变体、Distill 蒸馏版，以及 2026 年的 MAGI-1.1 24B）③ Motif-Video 2B（含 T2V 与 I2V 扩展）。三者均为开源纯视觉视频生成基础模型，放在一起对照的核心价值是：数据披露粒度从「几乎为零」（Mochi 1）→「方法级详尽但无数值」（MAGI-1）→「工具级+参数级详尽且明确以小数据取胜」（Motif-Video 2B）呈现出 2024→2026 的清晰演进曲线。

### 发布机构/公司

① Mochi 1：Genmo（Genmo AI，美国旧金山创业公司，CEO/联合创始人 Paras Jain，联合创始人 Ajay Jain，均为 UC Berkeley 博士背景；2024 年完成约 2840 万美元 A 轮融资，NEA 领投）。
② MAGI-1：Sand AI（三体智能 / sand.ai，中国北京，创始人曹越 Yue Cao，清华大学博士、原微软亚洲研究院研究员，Swin Transformer 作者之一；技术报告署名作者 39 人）。
③ Motif-Video 2B：Motif Technologies（韩国 AI 公司，与 AI 芯片/编译器公司 Moreh 同源，此前发布过 Motif-2.6B / Motif-2-12.7B 语言模型系列）。

### 发布时间（技术报告/论文/开源时间）

① Mochi 1：2024 年 10 月 22 日发布并开源 mochi-1-preview（权重 + 推理代码，Apache 2.0），同日开放 720p HD 版本的预告（后续未按期完整开源）；2024 年 11 月前后陆续开源 AsymmVAE 与微调脚本。无正式论文或 arXiv 技术报告，仅有官方博客。
② MAGI-1：2025 年 4 月 21 日开源 24B 权重与推理代码（GitHub SandAI-org/MAGI-1）；2025 年 5 月 19 日发布 arXiv 技术报告 arXiv:2505.13211《MAGI-1: Autoregressive Video Generation at Scale》（61 页）；2025 年 4 月 30 日发布 4.5B 版本；2025 年 5 月 26 日发布 4.5B Distill；2025 年 5 月 30 日支持 ComfyUI；2026 年 6 月 17 日开源 MAGI-1.1 24B。
③ Motif-Video 2B：2026 年 4 月 14 日在 Hugging Face 发布权重（Motif-Technologies/Motif-Video-2B，Apache 2.0）；arXiv:2604.16503《Motif-Video 2B: Technical Report》，v2 版本更新于 2026 年 5 月 19 日；同期提供 Diffusers 官方集成与社区 GGUF 量化版本。

### 类型（模型/数据集/工具链/评测基准）

三者均为「模型 + 配套工具链」：
① Mochi 1：10B 参数的 Asymmetric Diffusion Transformer（AsymmDiT，48 层，视觉维度 3072 / 文本维度 1536）文生视频模型 + 自研 AsymmVAE（362M 参数，8×8 空间 + 6× 时间压缩至 12 通道潜空间，总压缩率 96×）。输出 848×480、30 FPS、最长 5.4 秒（84 帧）。
② MAGI-1：块级自回归扩散（chunk-wise autoregressive denoising）视频世界模型，最大 24B 参数，每 chunk 24 帧，上下文最长 400 万 token，支持 T2V / I2V / V2V 视频续写统一建模与流式生成；配套自研 Transformer VAE（8× 空间 / 4× 时间下采样，16 通道）、MagiAttention 分布式注意力库、Shortcut Model 蒸馏。
③ Motif-Video 2B：2B 参数的 rectified flow matching 文生视频模型 + 图生视频扩展，核心创新为 Shared Cross-Attention 与三段式骨干功能分解（早期融合 / 联合表征 / 细节解码），训练侧使用 TREAD token 路由与 REPA 表征对齐；额外贡献一套「离线 bucket 均衡采样器」数据基础设施。定位明确为「micro-budget（微预算）」路线的对照实验。

### 开源程度（权重/代码/数据/pipeline各自是否开源）

① Mochi 1（部分开放，数据侧完全封闭）：权重与推理代码 Apache 2.0 开源（GitHub genmoai/mochi、HF genmo/mochi-1-preview），AsymmVAE 一并开源，允许商用。但无技术报告/论文，无任何数据 pipeline 披露，训练数据、清洗流程、打标方法均未公开，官方对数据来源明确采取「因竞争原因不透露」的态度。可视为「开权重、闭方法、闭数据」。
② MAGI-1（高度开放，数据方法开放但数值封闭）：Apache 2.0 开源 24B / 4.5B 基座与蒸馏权重、FP8 量化版、推理代码，另开源 MagiAttention 独立库；61 页技术报告完整披露了 Section 3 DATA 的全部方法论（切分工具、11 类过滤 actor、去重双模型、MLLM 二次过滤、两类 caption schema、三阶段数据配置表）。但所有过滤阈值均以「predefined threshold / lower and upper thresholds」表述而不给数值，数据集规模仅给「tens of petabytes 原始素材」这一量级词，清洗代码与数据本体不开放。可视为「开权重、开方法、闭参数、闭数据」。
③ Motif-Video 2B（三者中数据披露最彻底）：权重 Apache 2.0 开源，技术报告把数据管线写成可复现的工程文档——具名工具（NeMo Curator、ffmpeg cropdetect、PaddleOCR-VL、SigLIP、Aesthetic Predictor V2.5、DOVER、UniMatch、SSCD sscd_disc_mixup、NVIDIA cuVS、Qwen3-VL-30B-A3B）、具名超参（SSCD 512 维描述子 / 320×320 / 第 10 帧 / cosine≥0.9 / k=64 / nprobe=16、OCR 聚类需出现于 ≥50% 帧、上下各 20% 区域排除、caption 采样概率 0.5/0.3/0.2、SA 优化 30000 次迭代、rolling shuffle 窗口 4096）、以及完整 10 阶段课程表。但数据集本体与清洗代码同样不开源，Sankey 图（Fig.7）只给流向不给绝对数值。可视为「开权重、开方法、开参数、闭数据」。

### 是否支持音视频同时生成，以及实现方式（原生联合/级联/MoE融合）

三者均不支持音视频同时生成，全部为纯视觉（无声）视频生成模型。
① Mochi 1：输出 848×480 / 30fps / 5.4s 无声视频，文本编码器为单个 T5-XXL，架构中无音频分支，官方博客与模型卡均未提及音频。
② MAGI-1：技术报告全文（含 Section 3 DATA 与 Section 4 Infrastructure）不涉及音轨抽取、音频过滤或音画对齐；数据管线在 PySceneDetect 切分后仅保留视觉帧序列。MAGI-1.1 亦未加入音频能力。
③ Motif-Video 2B：数据管线明确只处理 Image Real / Image Synthetic / Video Real / Video Synthetic 四个视觉分支，caption 的 JSON schema 中不含任何听觉字段，全文无 audio / speech / music 相关处理。
因此本条目中所有音频相关维度（audio_category_distribution、joint_av_caption_schema、dialogue_transcription_attributes、av_sync_detection、sync_metric_and_threshold、audio_quality_filtering、audio_type_handling）均为「不适用」。

### 调研信息来源列表（论文/技术报告/官方文档/新闻的URL，标注每条来源的性质：官方一手/同团队旁证/第三方报道）

1. https://arxiv.org/abs/2604.16503 —— 官方一手，《Motif-Video 2B: Technical Report》，Motif Technologies，v2 于 2026-05-19 更新。本条目 Motif 部分的全部定量信息来源（Section 4 训练课程 Table 1、Section 5 Data 完整数据管线、Table 2 采样器利用率、Table 3 VBench 16 维结果）。
2. https://arxiv.org/html/2604.16503 —— 官方一手，Motif 技术报告 HTML 全文（本次调研抓取全文本地解析，Section 5.1 数据处理管线、5.2 视频打标、5.3 离线 bucket 均衡采样器逐段核对）。
3. https://huggingface.co/Motif-Technologies/Motif-Video-2B —— 官方一手，Motif-Video 2B 权重卡片，Apache 2.0，2026-04-14 发布，标注约 1000 万视频片段、<100,000 H200 GPU 小时、≈90% 数据利用率。
4. https://arxiv.org/abs/2505.13211 —— 官方一手，《MAGI-1: Autoregressive Video Generation at Scale》，Sand AI，2025-05-19，61 页。
5. https://arxiv.org/html/2505.13211v1 —— 官方一手，MAGI-1 技术报告 HTML 全文（本次调研抓取全文本地解析，Section 3 DATA 的 3.1 Filter Actors / 3.2 De-duplication / 3.3 MLLM as Advanced Filter / 3.4 Caption / 3.5 Data Adjustment、Table 3 caption 属性表、Table 4 caption 示例、Table 5 三阶段数据配置逐段核对）。
6. https://github.com/SandAI-org/MAGI-1 —— 官方一手，MAGI-1 代码仓库与 README，含 Apache 2.0 许可、24B/4.5B/Distill/FP8 变体清单与发布时间线（2025-04-21 至 2026-06-17 MAGI-1.1）。
7. https://www.genmo.ai/blog/mochi-1-a-new-sota-in-open-text-to-video —— 官方一手，Genmo 官方博客《Mochi 1: A new SOTA in open text-to-video》，AsymmDiT 架构、AsymmVAE 96× 压缩、44,520 token 上下文、Gemini-1.5-Pro-002 作为 prompt adherence 自动评测员等；全文无训练数据章节。
8. https://huggingface.co/genmo/mochi-1-preview —— 官方一手，Mochi 1 模型卡（48 层 AsymmDiT、362M AsymmVAE、480p/84 帧、Apache 2.0、约 60GB 显存、已做 NSFW 过滤、承认模型反映训练数据偏见）。
9. https://github.com/genmoai/mochi —— 官方一手，Mochi 1 代码仓库。
10. https://venturebeat.com/ai/video-ai-startup-genmo-launches-mochi-1-an-open-source-model-to-rival-runway-kling-and-others —— 第三方报道（VentureBeat，2024-10-22），含 Paras Jain 关于训练数据的直接引语「Generally, we use publicly available data and sometimes work with a variety of data partners」以及其拒绝细说的表态，是 Mochi 1 数据来源唯一的一手表述。
11. https://siliconangle.com/2024/10/22/genmo-introduces-mochi-1-open-source-text-video-generation-model/ —— 第三方报道，Mochi 1 发布报道与规格核对。
12. https://www.oschina.net/news/346129/sand-ai-magi1 —— 第三方报道（OSCHINA 中文），Sand AI 与曹越团队背景、MAGI-1 发布信息旁证。
13. https://huggingface.co/papers/2505.13211 —— 第三方聚合，MAGI-1 论文页与社区讨论。

## 数据规模与分布

### 训练数据量级（视频条数/小时数/token数，预训练与SFT分开） ⚠️

① Mochi 1：完全未披露。官方从未给出视频条数、小时数或 token 数，仅称模型「trained entirely from scratch」且为「当时公开发布的最大视频生成模型」。[不确定]
② MAGI-1：只给出原始素材量级——「from tens of petabytes of raw videos and images collected from a wide range of sources」（数十 PB 原始视频与图像）。清洗后的训练集条数、小时数、各阶段样本量均未披露；Table 5 只写明「数据量随阶段递减」而不给数字。预训练与 SFT 也未分开计量（报告中无独立 SFT 阶段，见 post_training_data）。[不确定]
③ Motif-Video 2B：给出明确且刻意压低的上界——「fewer than 10M clips」（少于 1000 万视频片段）与「less than 100,000 H200 GPU hours」（少于 10 万 H200 GPU 小时）。HF 卡片表述为「约 1000 万视频片段」。论文将此对照同期开源模型的「hundreds of millions of curated clips」（Wan2.1、HunyuanVideo、Seedance 等为数亿片段、5B–14B 参数），自称数据量低一个数量级、参数量少 7 倍。SFT 语料规模未给绝对数值，仅说明为「curated high-quality subset」并按 subject 类目迭代补充（Fig.8 给的是分布比例而非绝对量）。[部分不确定：SFT 精选集绝对规模]

### 数据来源构成（自有/公开数据集/网络爬取/授权采购/合成数据） ⚠️

① Mochi 1：官方仅有一句口径——Paras Jain 对 VentureBeat 表示「Generally, we use publicly available data and sometimes work with a variety of data partners」（一般使用公开可得数据，有时与各类数据合作方合作），并以竞争原因拒绝细说。既未点名任何公开数据集，也未说明爬取/授权比例。[不确定]
② MAGI-1：仅表述为「collected from a wide range of sources」（来源广泛），未区分自有爬取、公开数据集、授权采购或合成数据，未点名任何来源数据集。从「数十 PB 原始素材 + 自建可扩展数据处理系统 + PySceneDetect 从长视频切分」可推断以自有大规模网络爬取为主，但论文未明说。[不确定]
③ Motif-Video 2B：明确为两路来源——「an internal web-scale video collection（自有网络级视频爬取）」与「a set of publicly available image and video datasets（一批公开图像与视频数据集）」，两路共用同一套下游 sanitation / 过滤 / 去重 / 分阶段质量控制流程，以保证最终语料由单一标准治理。此外把原始池显式拆为 Image Real / Image Synthetic / Video Real / Video Synthetic 四个分支，即承认使用合成数据，且规定「Synthetic video is injected only at 720p（合成视频只在 720p 阶段注入）」，理由是其可控质量最契合该阶段的准入标准。但未点名具体公开数据集名称，也未给各来源配比。[部分不确定：具体公开数据集名单与配比]

### 数据合规与溯源（授权数据占比、rights-cleared数据集、C2PA等） ⚠️

三者均无实质性的数据合规与溯源披露，这是本组三个模型共同的最大缺口：
① Mochi 1：数据来源刻意模糊化，恰恰成为发布时的争议焦点——VentureBeat 报道直接指出「训练数据集是 AI 创作工具最具争议的方面之一，已有证据表明许多工具未经许可、未付报酬地使用了大量网络上的人类创作成果，其中部分为受版权保护作品」，而 Jain 对此「was coy（含糊其辞）」。未披露授权数据占比、rights-cleared 数据集、C2PA 内容凭证或水印溯源。模型卡仅在下游层面提示「Genmo 视频模型会反映其训练数据中的偏见与成见」，建议机构在商用部署前追加安全协议。[不确定]
② MAGI-1：技术报告无数据合规章节，未讨论版权、授权比例、C2PA 或来源清单，仅在权重层面给 Apache 2.0。[不确定]
③ Motif-Video 2B：同样无合规章节，未披露授权占比、rights-cleared 数据或内容凭证。相对的间接合规动作是把 NSFW 与水印过滤前置到 sanitation 首关（水印/台标本身也是版权信号的代理），并在 caption 阶段用 VLM 复核 watermark 标签做二次剔除——可以理解为「以工程手段规避带权利标记的素材」，但论文并未把它表述为合规措施。[不确定]

### 片段时长分布与切分策略 ⚠️

① Mochi 1：训练侧时长分布与切分策略完全未披露；仅知生成侧上限为 84 帧 / 30 FPS / 5.4 秒。[不确定]
② MAGI-1：给出分阶段的时长上界（Table 5）——stage-1 与 stage-2 均为「≤ 8 秒」，stage-3 放宽到「≤ 16 秒」，论文说明最后阶段延长时长是为让模型捕捉更丰富的时序动态。切分策略为 PySceneDetect 切成单镜头 clip（详见 shot_segmentation）。未给下界、未给分布直方图。[部分不确定：时长下界与实际分布]
③ Motif-Video 2B：切分后设有明确下界——「Clips shorter than two seconds after merging are discarded（合并后短于 2 秒的片段丢弃）」，目的是保证每个训练片段覆盖有意义的时序跨度。训练侧的时长实际由帧数桶决定：帧数桶为单帧图像、33 帧、65 帧、121 帧四档，并在每个阶段转换时重新施加 clip-length 过滤（阈值随分辨率提升而收紧）。未给时长直方图。[部分不确定：具体分布]

### 分辨率/宽高比分布与分桶策略 ⚠️

① Mochi 1：未披露训练数据的分辨率与宽高比分布及分桶策略；输出固定 848×480（约 16:9）。[不确定]
② MAGI-1：训练分辨率按三阶段推进——stage-1 为 256p/360p，stage-2 为 480p，stage-3 为 720p（Table 5）。VAE 推理侧采用滑窗支持任意分辨率（空间窗口 256×256、步长 192、25% 重叠；时间维不重叠），模型宣称支持任意分辨率，4.5B 版本默认 720×720。但训练数据的宽高比分布与分桶权重未披露。[部分不确定：宽高比分桶]
③ Motif-Video 2B：分辨率是整条管线最主要的分层轴——课程为 144p → 360p → 480p → 720p 四档十阶段，每次阶段跃迁都会重新施加 resolution / clip-length / motion / aesthetic 四类过滤且阈值更严。宽高比处理上有两条具体工程措施：一是用 ffmpeg cropdetect 基于亮度统计估计最大内容矩形，剔除 letterbox/pillarbox 黑边填充；二是把 OCR 检出区域与黑边裁剪框合成为单一最终矩形（排除内容区上方 20% 的台标与下方 20% 的字幕），在一次 ffmpeg 重编码中与分辨率缩放、帧率限制一并施加。训练时按「帧数桶 × 分辨率桶」联合分桶（帧数桶：1 / 33 / 65 / 121，每档再细分多个空间分辨率），并由离线 bucket 均衡采样器保证各 rank 上各桶样本数的变异系数最小。未给宽高比的定量分布。[部分不确定：宽高比定量分布]

### 类别/domain分布与配比策略（人物、动作、场景、风格等比例控制与概念均衡） ⚠️

这是三者差异最大的维度，恰好构成一条「无 → 动态自适应 → 显式类目均衡」的演进线：
① Mochi 1：无任何披露。既未给类目统计，也未提及概念均衡或配比控制。[不确定]
② MAGI-1：不做静态配比表，而是提出「动态分布调整（Dynamic Distribution Adjustment）」——论文明确指出「合适的数据分布对训练高性能模型至关重要，但事先确定最优分布极其困难」，并给出实际观察：「训练过程中我们发现风景类场景对模型而言相对容易学习，而人物表情则显著更难」，这类洞察事前无法预判。因此其做法是在训练全程持续监控模型在各语义概念上的表现，据评测结果自适应地调高欠拟合子集的采样比例，以针对性强化模型短板。这是一种「以评测反馈闭环驱动的在线数据配比」，比静态配比表更贴近课程学习的本意，但论文未公开任何类目清单、初始配比或调整幅度。同时 MAGI-1 还提到一项任务级配比：因产品场景以图生视频为主，「训练中分配了更大比例的 I2V 任务」。[部分不确定：类目清单与配比数值]
③ Motif-Video 2B：做显式的类目均衡，且把它接在 caption 元数据上形成闭环。VLM 在同一次前向中产出 subject 与 style 结构化标签，这两个标签「drive domain balancing for the 720p stage and SFT（驱动 720p 阶段与 SFT 的领域均衡）」。更关键的是 SFT 语料的组装方式是迭代式补短板：「我们在最新 checkpoint 上跑中间评测，识别出生成质量最弱的 subject 类目，然后针对性地补充这些类目的片段」。Fig.8 给出最终结果——图像侧 People（人物）占主导，反映以角色为中心的使用场景；视频侧则向 Transportation（交通工具）、Sports（运动）、Animals（动物）倾斜，因为这三类涉及剧烈动态、在中间评测中被识别为弱项。这与 MAGI-1 的动态调整思路同构，但 Motif 把它落到了可读的类目分布图上，并且额外用 action=Dynamic 标签作为 720p SFT 的动态运动准入条件。

### 音频类别分布与配比（语音/音效foley/音乐/环境音/静音的比例及控制策略）——AV模型独有维度

不适用。三个模型均为纯视觉视频生成模型，训练数据不保留音轨，技术报告/博客全文均未涉及语音、音效 foley、音乐、环境音、静音的分类与配比控制。

### 叙事结构分布（单镜头vs多镜头、平均clip时长、镜头数分布、是否含原生音轨） ⚠️

① Mochi 1：未披露。生成侧固定输出单段 5.4 秒视频，推断训练素材为单镜头短片段，但无官方说明。[不确定]
② MAGI-1：训练片段严格为单镜头——PySceneDetect 切分「ensuring that each clip contains only a single shot」，并额外用 Transition Detection actor 兜底（PySceneDetect 难以处理复杂转场，导致切出的 clip 仍可能含多镜头；因此稀疏采关键帧、用 CLIP 计算相邻关键帧语义相似度，低于阈值即判为多镜头并整段剔除）。平均 clip 时长按阶段为 ≤8s / ≤8s / ≤16s。不含原生音轨。
值得单列的是：MAGI-1 虽然训练素材是单镜头，却通过自回归 caption（见 caption_structure）和 chunk-wise 文本条件，在推理侧获得了多镜头叙事能力——技术报告 Fig.10 展示了一个近 30 秒、由 (a)–(g) 七段不同 prompt 逐段驱动的复杂动作与叙事结构生成示例，Fig.12 还展示了通过在不同去噪阶段调制 KV range 来实现「保持身份的镜头切换」与「保持场景布局但改变物体细节的转场」两类镜头转换。也就是说，多镜头叙事在 MAGI-1 中是被「架构 + caption schema」而非「多镜头训练数据」实现的，这一点在数据侧对照上很有意思。
③ Motif-Video 2B：同样严格单镜头，且切分策略更精细：先用「偏向过切分（宁可假阳性，不可漏检转场）」的保守阈值检测场景边界，再用 SigLIP embedding 相似度做 stitch detection 把因瞬时运动或曝光变化而被误切的连续镜头重新合并，最后丢弃合并后 <2 秒的片段。此外 VLM 产出的 multi_scene 标签被用作「对场景切分的二次检查」，命中即丢弃。不含原生音轨。镜头数恒为 1，无多镜头叙事数据构造。

### 语言/口音分布（多语种唇同步能力的数据基础） ⚠️

① Mochi 1：文本编码器为单个 T5-XXL（英文为主），caption 语言构成未披露。[不确定]
② MAGI-1：caption 由 MLLM 生成，示例（Table 4）均为英文；语言分布未披露。有一处间接线索：为缓解训练 caption 与真实用户输入的分布错配，MAGI-1 在推理侧设计了 Prompt Enhancement（PE）策略，并把大 MLLM 的改写能力蒸馏到约 7B 的小模型上，构建了约 200 万条训练语料——但未说明该语料的语言构成。[不确定]
③ Motif-Video 2B：caption 由 Qwen3-VL-30B-A3B 生成，schema 与示例均为英文，caption_long 定义为 150–250 words、caption_short 为 15–25 words，明确以词（word）计量，实为英文单语；文本编码器为 T5Gemma2（早期阶段为 sentence-level embedding 模型）。未披露多语种 caption 或语言分布统计。
唇同步/口音维度对三者均不适用（无音频）。[不确定]

## 清洗流程

### 清洗漏斗整体结构（几级过滤、各级顺序） ⚠️

① Mochi 1：无任何 pipeline 披露。官方博客只讲架构（AsymmDiT、AsymmVAE、44,520 token 全 3D 注意力、扩展的 3D RoPE、SwiGLU、QK-Norm）与评测协议（沿用 OpenAI DALL-E 3 的 prompt adherence 自动评测范式、以 Gemini-1.5-Pro-002 为评判模型），数据章节完全缺席。模型卡唯一与数据处理相关的表述是「已施加 NSFW 内容过滤」。[不确定]
② MAGI-1（方法级完整、参数级留白的四段式漏斗，Fig.13）：
【第 1 级】镜头切分：PySceneDetect 把长视频切为单镜头短片（仅对视频数据施加，图像跳过该模块）。
【第 2 级】Filter Actors 并行过滤：11 类专用过滤器一次性作用于切好的 clip（详见 quality_filtering / motion_filtering）——Video Quality Assessment（DOVER）、Aesthetics（LAION 美学模型）、Overexposed/Underexposed（HSI 色彩空间平均亮度）、Motion Strength（RAFT 光流 + 显著性检测分前景/背景）、Camera Movement Stability（相邻帧光流一致性）、Slides Movement（光流散度）、Border Detection（边缘检测 + Hough 变换）、Text Detection（含字幕的时空模式专项识别）、Logo Detection（Florence-2 开放词表检测）、Corner Face Detection（人脸检测 + 角落位置置信度）、Transition Detection（CLIP 关键帧语义相似度）。
【第 3 级】去重：CLIP 与 DINOv2 双特征成对相似度，任一超阈值即判重删除。
【第 4 级】MLLM 高级过滤：前三级已滤掉绝大部分劣质数据、剩余量已大幅缩小，此时引入多模态大模型做一轮语义级复筛，捕捉复杂 bad case；关键工程设计是「该步骤可无缝并入后续 caption 流程，从而降低总成本、提升效率」——即判分与打标共用同一次 MLLM 调用。
【第 5 级】打标：通过筛的数据由同一 MLLM 生成 highly descriptive caption 与 auto-regressive caption。
【第 6 级】分布调整：多阶段调整（质量随阶段收紧）+ 动态分布调整（按评测反馈在线改配比）。
设计要点：把便宜的规则/小模型过滤放前、把昂贵的 MLLM 放在数据量已显著收缩之后，并让 MLLM 的「判官」与「打标员」两个角色复用同一次前向。
③ Motif-Video 2B（工具级 + 参数级完整的分支式漏斗，Fig.7 Sankey）：
【第 0 级】分支拆解：原始池显式拆为 Image Real / Image Synthetic / Video Real / Video Synthetic 四条支路，共用同一套下游控制。
【第 1 级】Sanitation（前置消毒）：剔除损坏/不可解码文件、异常小文件（缩略图或下载损坏）、SSCD 近重复、NSFW、带水印内容。其中 NSFW 与水印用双信号——先是继承自旧爬取管线的 OCR 初筛（用帧上文字检测标出台标、烧录字幕等高置信水印），存活片段再由 VLM 复核结构化标签（watermark / nsfw / padded / multi_scene / timelapse / quality），命中即丢；论文称第二遍是「架在 OCR 之上的语义感知安全网」。
【第 2 级】黑边检测：ffmpeg cropdetect 基于亮度统计估计最大内容矩形。
【第 3 级】OCR 检测：PaddleOCR-VL（经 vLLM 服务化）在每个 clip 均匀采样 N 帧上运行，按空间 IoU 跨帧聚类检测框，只保留出现在 ≥50% 帧中的簇（以此区分固定叠加物与场景内瞬时文字）；存活的 OCR 区域与黑边裁剪框合成为单一最终矩形（排除内容区上 20% 的台标与下 20% 的字幕），在一次 ffmpeg 重编码中连同分辨率缩放与帧率限制一并落地。
【第 4 级】场景切分与长度控制：保守阈值过切分 + SigLIP 相似度 stitch 合并 + 丢弃 <2s。
【第 5 级】视觉质量多路过滤：Aesthetic Predictor V2.5（美学）、亮度（OpenHumanVid 公式）、Koala-36M 式模型化 suitability 分（保守拒绝）、DOVER technical 分（技术质量）、UniMatch 光流（运动质量，双侧截尾）。论文强调这些信号「不被合成为单一学习排序，而是每个过滤器专治一种失效模式」。
【第 6 级】SSCD 三阶段去重（嵌入 / 分组 / 代表选择，详见 deduplication）。
【第 7 级】VLM 打标即元数据：Qwen3-VL-30B-A3B 单次前向同时产出 caption 与结构化标签，标签直接回灌 sanitation 与分阶段过滤。
【第 8 级】分阶段渐进准入：144p → 360p → 480p → 720p，每次跃迁重新施加分辨率/时长/运动/美学过滤且阈值更严；720p SFT 额外叠加 domain-balancing 与 dynamic-motion 准入；合成视频只在 720p 注入。
设计要点与 MAGI-1 同构（VLM 打标与过滤共用一次前向），但 Motif 更进一步——「因为这些标签与训练 caption 出自同一次前向，过滤与条件化在构造上始终保持同步（synchronized by construction）」，即从机制上消除了「过滤用的语义判断」与「训练用的文本条件」之间的漂移。

### 漏斗定量保留率（每级过滤的输入/输出量与最终保留率，如Apollo 27%） ⚠️

① Mochi 1：无。[不确定]
② MAGI-1：无任何定量保留率。仅有定性表述：初次过滤「effectively discards most of the low-quality data（有效剔除了大部分低质数据）」、到 MLLM 过滤时「the remaining data size has been significantly reduced（剩余数据量已显著缩小）」，以及 Table 5 中「数据量随阶段递减」的方向性说明。既无逐级输入/输出量，也无端到端保留率，无法定位主要淘汰级。[不确定]
③ Motif-Video 2B：给了漏斗的可视化但没给数字。Fig.7 是一张 Sankey 图，论文原文称其「visualizes how flows contract from the raw pool toward the curated training and SFT corpora（可视化了数据流如何从原始池向精选训练集与 SFT 集收缩）」，但正文未标注任何百分比或绝对量。唯一可锚定的端点是最终训练集「fewer than 10M clips」，原始池规模未给，因此端到端保留率无法计算。这是 Motif 数据披露中少数的空白之一。[不确定]
综合看，本组三个模型都没有提供 Apollo（27%）或 Allegro（0.4%–26% 分档）那种逐级定量保留率，是横向对比时需要注意的共同短板。

### 镜头切分方法（PySceneDetect/自研模型/shot-aware splitting） ⚠️

① Mochi 1：未披露。[不确定]
② MAGI-1：使用开源 PySceneDetect 把长视频切为短片，明确要求「每个 clip 只含单个镜头」。因 PySceneDetect 对复杂转场（渐变、叠化等）处理不佳、切出的 clip 仍可能残留多镜头，团队追加了一个独立的 Transition Detection 过滤器兜底：稀疏采样关键帧，用 CLIP 计算相邻关键帧的语义相似度，若低于预定阈值则判定该 clip 含多镜头并整段剔除。注意其兜底策略是「删除」而非「二次切分」。
③ Motif-Video 2B：采用「过切分 + 语义再合并」的双向策略，是三者中最讲究的一套。第一步用「偏向过切分的保守阈值」检测场景边界，即刻意让假阳性（多切）多于假阴性（漏切转场）；第二步用基于 SigLIP embedding 相似度的 stitch detection 把相邻片段重新合并，专门恢复那些因瞬时剧烈运动或曝光变化而被误切开的连续镜头；第三步丢弃合并后不足 2 秒的片段。第三重兜底来自 VLM 的 multi_scene 标签——命中即丢，论文明确称其为「对场景切分的二次检查（a secondary check on scene segmentation）」。与 MAGI-1 相比，Motif 的差异在于承认切分器会两头出错并分别给出补救（合并纠过切、VLM 标签纠漏切），而 MAGI-1 只处理了漏切一侧。

### 质量过滤（美学评分、清晰度、OCR文字过滤、黑边/水印/logo检测） ⚠️

① Mochi 1：仅知施加了 NSFW 过滤，其余质量过滤策略、工具与阈值一律未披露。[不确定]
② MAGI-1（11 类 Filter Actor，方法详尽、阈值全部留白）：
· 视频质量评估：采用 DOVER，其输出 overall / aesthetic / technical 三个分数；团队经实证发现「technical score 单独使用是最有效的指标」，故只用技术分。
· 美学：LAION 美学预测器；因该模型本为图像设计，团队用「首帧的美学分」代表整段 clip 的质量（注意与 Allegro 用中间帧的选择不同）。
· 过曝/欠曝：把每一帧转到 HSI 色彩空间、计算全片平均亮度，据此剔除过曝或欠曝样本；论文指出这类数据会「adversely affect training stability（损害训练稳定性）」——把曝光过滤的动机归到训练稳定性而非观感，是一个值得注意的表述。
· 边框检测：逐帧边缘检测 + Hough 变换识别跨帧持续存在的水平/垂直直线，把这些线视为候选边框，以「含边框帧的占比」作为过滤置信度。
· 文字检测：若任一帧字符数过多或文字区域占画面比例过大则整段丢弃。特别处理字幕这一例外——字幕字符少、占面积小，不易被上述规则捕获，但具有鲜明时空模式（固定出现在画面上方或下方、跨多个连续帧持续存在），据此专项识别并剔除。
· Logo 检测：用 Florence-2（支持开放词表目标检测），给定预设关键词集合来检测定位画面中的 logo 并输出置信度用于过滤。
· 角落人脸检测：解说类视频中主播常固定出现在画面某个角落，团队用人脸检测模型、结合人脸位置与检测置信度，对固定角落人脸的置信度做跨帧平均，以估计「存在解说员」的可能性并剔除。
所有上述过滤器的阈值在报告中均以「predefined threshold」表述而不给数值。
③ Motif-Video 2B（5 路正交质量信号，工具与部分参数具名）：
· 美学：Aesthetic Predictor V2.5（基于 SigLIP 的图像级美学预测器）；对每个视频沿时间均匀采样帧、算帧级分、再跨采样帧平均得到视频级分；用作分阶段过滤器——低美学尾部被剔除，且「cutoff becomes stricter at higher-resolution stages（分辨率越高截断越严）」。
· 亮度：沿用 OpenHumanVid 的公式 L = 0.2126R + 0.7152G + 0.0722B（ITU-R BT.709 亮度权重），在采样帧上统计并剔除目标阶段的极低/极高亮度尾部，以过滤严重欠曝/过曝、提升保留数据中主体与场景内容的可见性。
· 模型化训练适配度分（Model-based Suitability Score）：借鉴 Koala-36M，用一个把多种质量因子汇总成「该视频是否适合用于训练视频生成模型」的单一估计值；实践中保守使用，只作为拒绝过滤器剔除最低适配度尾部，其余样本仍需通过后续各专项过滤。
· 技术质量：DOVER，取其技术质量相关输出，过滤压缩伪影、噪声、失真、低锐度等退化。
· 文字/水印/黑边：见 pipeline_overview 第 1–3 级（OCR 初筛 + VLM watermark 标签 + cropdetect + PaddleOCR-VL 持久区域聚类）。
· VLM 质量标签：quality=low 的片段自 480p 阶段起被排除。
论文特别强调这些信号「不被合成为单一学习排序（not used as a single learned ranking）」，而是每个过滤器针对一种具体失效模式（曝光差、压缩伪影、静态、时序不稳），这与把多指标塞进一个综合分的做法形成明确的方法论对照。

### 运动过滤（光流/运动分数阈值、静态与抖动剔除） ⚠️

① Mochi 1：未披露。官方仅在能力层面强调 Mochi 1 在运动质量上的优势（30 FPS、高时序一致性、真实运动动力学），CEO 也表示团队「focusing heavily on improving motion quality」，但对应的数据侧运动过滤策略无任何说明。[不确定]
② MAGI-1（三者中最细致，且是唯一做前景/背景运动分离的）：
· 运动强度：用 RAFT 光流模型；为降低计算开销先把所有视频下采样到 8 FPS 再计算相邻帧光流；在像素级计算光流，对全片所有像素的流幅值取平均得到整体运动强度。
· 前景/背景分离（关键改进）：论文指出上述做法在「背景静止但前景大幅运动」时会低估运动量；为此额外对每帧运行显著性检测模型（Zhao & Wu, 2019），用显著性图区分前景与背景区域，分别计算两者的平均光流。最终得到三个运动统计量：整体运动强度、前景运动强度、背景运动强度，并对三者各自设定上下阈值，优先保留中等运动强度的片段，同时避开过静与过动两端，以平衡数据质量与训练难度。
· 相机运动稳定性：因大量素材为手持拍摄、抖动剧烈且难以被运动强度单独筛出，团队通过评估相邻帧之间光流的一致性来估计相机稳定性，剔除相机运动不稳的片段。
· 幻灯片式运动：针对屏幕录制或幻灯片演示中常见的浮动照片/横幅，分析每帧全体像素光流的散度（divergence），若散度长期保持低值则判定为 slide movement 并剔除。
· 转场：见 shot_segmentation。
所有阈值未给数值。
③ Motif-Video 2B：用 UniMatch 在采样帧对之间估计光流并计算每个视频的运动分，双侧截尾——剔除极低运动（通常是静态或近静态）与极高运动（往往含剪切、抖动或不稳定相机运动）两端，保留中间带以匹配主训练阶段所要的平滑时序动态。运动过滤在每次分辨率阶段跃迁时重新施加且阈值更严。此外 720p SFT 阶段引入一个语义化的运动准入条件：VLM 标签 action=Dynamic 被用作「dynamic-motion criterion」——即在低层光流统计之外，再叠加一层「大模型判断这段视频的动作是否算动态」的语义门槛，这是本组中运动过滤从「像素统计」走向「语义判断」的一个具体案例。

### 去重方法（精确去重与基于embedding的语义去重分别记录） ⚠️

① Mochi 1：未披露。[不确定]
② MAGI-1：论文把去重的重要性上溯到 LLM 文献——引用 Lee et al. (2021) 与 Hernandez et al. (2022) 指出「即便少量重复数据也会显著损害性能」，据此「conduct rigorous de-duplication（执行严格去重）」。方法为双特征交叉判定：同时用 CLIP 与 DINOv2 计算片段间成对相似度，只要任一相似度超过阈值即判为重复并删除（逻辑或，偏严）。用 CLIP + DINOv2 双塔的用意在于兼顾语义相似与视觉/自监督结构相似两类重复模式。阈值数值与去重规模未给，也未说明如何在大规模上规避 O(N²) 的成对计算。[部分不确定：阈值、索引方案与去重量]
③ Motif-Video 2B（三者中唯一给出可复现工程细节的）：三阶段 SSCD 管线——
· 嵌入（Embedding）：用公开发布的 sscd_disc_mixup TorchScript 模型对每张图/每个视频编码，帧图先 resize 到 320×320 并做 ImageNet 归一化，产出 512 维描述子。视频取第 10 帧作为代表帧——论文明确解释这一选择的两个理由：避开最前几帧的片头与 logo 偏置，以及避免全帧两两比较、保持匹配可计算。选 SSCD 的理由是它专为 copy detection 设计，对重编码、裁剪和轻度编辑鲁棒，而这些正是网络爬取视频中最常见的重复形态。
· 分组（Grouping）：用 NVIDIA cuVS 的多 GPU IVF-PQ 索引在余弦距离下检索，每个 query 取 k=64 个近邻、nprobe=16，仅保留余弦相似度超过 0.9 的配对，再用 Union-Find 合并配对形成重复组。
· 代表选择（Representative selection）：每个重复组内按加权分 s = 0.5·res^ + 0.3·fps^ + 0.2·filesize^ 保留单一样本（三项均在组内做 min-max 归一化），组内其余成员丢弃。该规则偏好更高分辨率、更高帧率、重压缩程度更低的副本。
去重被放在 sanitation 首关（而非漏斗末端），意味着后续所有昂贵的模型推理都不会浪费在重复样本上。

### VLM/LLM作为质检员（多模态大模型质量打分与错配剔除，2026年从浅层打分器转向大模型语义判断的趋势） ⚠️

本组三个模型恰好完整呈现了「VLM 作为质检员」从缺席到成为管线枢纽的演进，是本条目对照价值最高的一个维度：
① Mochi 1（2024.10）：无任何披露，也无证据表明使用了大模型判分。[不确定]
② MAGI-1（2025.05，MLLM 作为「高级过滤器」，位置在漏斗末段）：报告设有独立小节 3.3「MLLM as Advanced Filter」，逻辑非常清晰——前面的 Filter Actors 与去重已滤掉绝大部分不良数据，但受限于这些过滤器的能力，仍残留少量低质数据；此时数据量已显著缩小，因此引入多模态大模型再做一轮过滤，「enables us to detect more complex bad cases（使我们能够检出更复杂的坏样本）」。最关键的成本设计是：「Notably, this step can be seamlessly integrated into the subsequent caption procedure, thereby reducing overall costs and improving efficiency（值得注意的是，该步骤可无缝并入后续的打标流程，从而降低总成本并提升效率）」——即判分与打标复用同一次 MLLM 调用。但论文未点名所用 MLLM 的具体型号与规模，也未说明它判断哪些维度、输出何种结构。[部分不确定：MLLM 型号与判定维度]
③ Motif-Video 2B（2026.04，VLM 从「末段复筛」升级为「贯穿全程的元数据源」）：其 caption-as-metadata 设计把 VLM 判分推到了新的形态——Qwen3-VL-30B-A3B 的单次前向按固定 JSON schema 同时输出自由文本 caption 与结构化标签（subject、style、action、camera_move、quality、watermark、nsfw，以及 padded、multi_scene、timelapse），这些标签随后被四路复用：(i) 训练时的文本条件；(ii) sanitation（nsfw / watermark / padded / multi_scene / timelapse / quality 直接触发硬删除）；(iii) 领域与主体均衡采样（style / subject 驱动 720p 与 SFT 的 domain balancing）；(iv) 720p SFT 的动态运动过滤（action=Dynamic）。且论文明确「我们不再用一个单独的后处理模型去重新解释这些标签（we do not apply a separate post-processing model to reinterpret them）」，标签由 VLM 直接消费。其方法论上的关键论断是：「因为这些标签与训练 caption 产自同一次前向，过滤与条件化在构造上始终同步（filtering and conditioning remain synchronized by construction）」——这是把「模型质检」与「模型打标」彻底合一、并以此消除两者语义漂移的明确设计主张，代表了 2026 年该范式的成熟形态。
三者连起来看：Mochi 1（无）→ MAGI-1（MLLM 末段复筛、与打标复用调用以省成本）→ Motif（VLM 前向即元数据源、过滤与条件化构造性同步），演进方向是 VLM 在管线中的位置不断前移、职能不断扩张。

### 安全与合规过滤（NSFW、版权、人脸/隐私） ⚠️

① Mochi 1：模型卡表明「NSFW content filtering has been applied（已施加 NSFW 内容过滤）」，但未说明方法、工具或阈值；同时明确提示「Genmo 视频模型会反映其训练数据中的偏见与成见」，建议机构在商用部署前实施额外安全协议。版权、人脸/隐私相关的数据侧过滤未提及。[部分不确定：NSFW 过滤的具体方法]
② MAGI-1：Section 3 DATA 全文未设安全与合规过滤环节，未提及 NSFW、暴力、版权或人脸隐私过滤。唯一与人脸相关的过滤是 Corner Face Detection，但其动机是「剔除解说类视频的固定角落主播画中画」这一视觉模式问题，而非隐私保护。[不确定]
③ Motif-Video 2B：安全过滤被显式前置到 sanitation 首关，且是本组中唯一给出双信号机制的：先由继承自旧爬取管线的 OCR 初筛以帧上文字检测标出高置信水印/台标/烧录字幕，存活片段再由 VLM 输出 nsfw、watermark、padded 等结构化标签，命中任一即硬删除；论文称第二遍 VLM 是「架在 OCR 之上的语义感知安全网（a semantically aware safety net on top of OCR）」。但仍未涉及版权审查、人脸/隐私脱敏、未成年人内容等专项，也未给 NSFW 分类器的具体型号与判定阈值。[部分不确定：NSFW 判定细则与版权/隐私处理]

## 打标方式

### 使用的caption模型（自研VLM/开源模型，模型规模） ⚠️

① Mochi 1：打标模型完全未披露。已知的只有训练时的文本编码器——单个 T5-XXL（官方强调「a single T5-XXL language model rather than multiple pretrained models」，即刻意不做多编码器融合），文本维度 1536、视觉维度 3072，视觉流参数量约为文本流的 4 倍。是否使用 VLM 重打标、caption 长度与结构一概不明。[不确定]
② MAGI-1：使用「一个 MLLM」为数据打标，但技术报告始终未点名具体模型与规模（既未说是自研还是开源微调）。有一个明确的工程参数：因主流 MLLM 多为图像设计，团队把每个视频抽取一组关键帧构成图像序列送入；经实证分析，「每个视频片段用 4 到 12 帧（依时长而定）在描述准确度与计算效率之间达到最佳权衡」。另有一个规模数字属于推理侧而非打标侧：为轻量化部署，团队把大 MLLM 生成的增强 prompt 蒸馏到约 7B 的小模型，训练语料约 200 万条（并过滤掉目标文本过长的样本以控制输出长度），人评显示蒸馏模型的生成质量与大模型相当而延迟与算力开销大幅下降。[部分不确定：打标 MLLM 型号与规模]
③ Motif-Video 2B：明确点名——所有 caption 与标签均来自 Qwen3-VL-30B-A3B（30B 总参 / 3A B 激活的 MoE 视觉语言模型）。视频输入为从片段中均匀采样的 N 帧，图像则直接输入。另有一个独立的 OCR 专用模型 PaddleOCR-VL（经 vLLM 服务化）用于帧上文字检测。文本编码器方面：第 1–3 阶段用 sentence-level embedding 模型，第 4 阶段起换为 T5Gemma2——论文借鉴 PixArt-α 的「类条件到文本条件」课程思路，假设低维条件空间能在早期加速收敛，待需要细粒度组合控制时再换用更强的编码器。

### caption密度与结构化程度（短/长/密集描述、结构化字段如镜头运动、风格标签） ⚠️

① Mochi 1：未披露。[不确定]
② MAGI-1（两类 caption 并存，是其最有特色的数据设计）：
【Highly Descriptive Caption（高描述性 caption）】沿用 DALL-E 3（Betker et al., 2023）的结论，用 MLLM 生成高密度描述。针对视频比图像多出的时序信息（动作、相机运动、场景变化），采用两阶段 prompt 结构：第一阶段先用一组定向问题引导模型对预定义属性作结构化分析，属性清单（Table 3）共 8 项——Scene Count（识别视频中不同场景的数量）、Camera Transitions（记录镜头间可察觉的转场）、Camera Shot Type（指明所用景别）、Camera Movement（描述相机运动）、Main Subject Identification（确定视频的核心焦点是谁/什么）、Subject Attributes（描述主体外观）、Subject Position（指明主体在画面中的位置）、Subject Action（说明主体在做什么）；第二阶段才生成最终描述性 caption，可吸收第一阶段分析中识别出的显著观察。对图像数据则不做属性预分析、直接生成 caption。注意这些属性是「引导分析用的中间产物」而非独立保留的 JSON 字段，最终落到训练的仍是自然语言散文。
【Auto-Regressive Caption（自回归 caption）】这是 MAGI-1 独有、与其块级自回归架构强耦合的设计。因模型逐块生成、可为视频不同部分提供不同文本条件，团队为每个片段提供「逐秒（second-by-second）」的细粒度描述：第 1 秒的 caption 被要求生成详细描述，后续每秒的 caption 则聚焦于「相对上一秒的变化」。Table 4 的示例清楚展现了这一增量式结构（如「2nd second: 女子头部微倾，表情从中性转为浅笑，口红仍在手中，镜头保持静止」）。这套 caption 使 chunk-wise 文本控制在训练侧有据可依，也是 MAGI-1 能用单镜头素材实现推理侧多镜头叙事的数据基础。AR caption 在训练中按阶段配比使用：stage-1 为 0%，stage-2 与 stage-3 各为 10%（Table 5）。
【推理侧补丁】因模型以结构化高描述性 caption 训练、而真实用户输入从极简到冗长差异极大，团队在推理侧加了 Prompt Enhancement 策略并蒸馏到约 7B 小模型（语料约 200 万条），以弥合训练分布与用户输入分布的错配。
③ Motif-Video 2B（三变体 caption + 固定 JSON schema，直接针对训练/推理分布错配）：
· schema：要求每个响应遵循固定 JSON 格式，同时含自由文本 caption 字段与结构化字段（style、subject、action、camera_move、quality 等），格式非法的响应会被重采样。
· 双 prompt：视频 prompt 与图像 prompt 共享同一 JSON schema、仅在时序字段上不同。视频 prompt 把采样帧视作单一描述对象，并按固定顺序索取——相机属性（景别、机位角度、运动）、主体、动作、环境、光照与色彩、任何屏幕上的文字。图像 prompt 去掉时序字段，改为索取构图、取景与文字的逐字转录。
· 反幻觉约束：两个 prompt 都明令禁止三件事——做出画面中无依据的断言、逐帧式叙述（frame-by-frame narration）、以及对质量/流畅度/氛围的主观评论；目的是减少幻觉标签与描述漂移。
· 三变体与采样概率（针对训练/推理错配的核心设计）：每个片段保留同一次 VLM 响应派生的三个 caption 变体——caption_long（150–250 词的详细描述）、caption_short（15–25 词的单句）、caption_truncated（只保留 caption_long 的首句）；训练时按固定概率 (p_long, p_short, p_truncated) = (0.5, 0.3, 0.2) 采样。论文说明其意图是缩小「长合成 caption」与「用户实际给的短 prompt」之间的训练—测试错配，并指出短/截断变体同时起到温和的 caption dropout 作用、降低对 VLM 特定措辞的过拟合。论文诚实标注这是「务实的配方选择而非经隔离验证的最优性主张（a pragmatic recipe choice rather than an isolated claim of optimality）」。
对照要点：MAGI-1 与 Motif 面对的是同一个问题（训练用长密集 caption vs 用户给短 prompt），但解法方向相反——MAGI-1 在推理侧加 Prompt Enhancement 把短 prompt 补长，Motif 在训练侧混入短 caption 让模型直接适应短输入。这是本条目中一个非常干净的方法论对照。

### 音视频联合caption结构（是否同时覆盖视觉+听觉轨道、是否分流为独立字段，如LTX-2全音景描述、Script-a-Video factorized streams、Foley-Omni三字段）

不适用。三者均无音频模态，caption 仅覆盖视觉轨道。Motif-Video 2B 虽有严格的 JSON schema（camera / subject / action / environment / lighting / on-screen text 等字段），但全部为视觉字段，不含任何听觉轨道描述；MAGI-1 的 8 项预定义属性同样纯视觉。不存在全音景描述、factorized audio/visual streams 或独立音频字段。

### 对白转写与说话人属性标注（ASR转写+说话人身份/语言/口音/情绪）

不适用。三者均无音频处理链路，未做 ASR 转写、说话人身份/语言/口音/情绪标注。需要注意的是，Motif-Video 2B 的 caption schema 中确有「verbatim text transcription（文字逐字转录）」与「on-screen text（屏幕文字）」字段，MAGI-1 也有专门的字幕检测逻辑，但这些指的都是画面上的视觉文字（OCR），与语音转写无关；而且在 Motif 的管线中，检出持久文字区域的后果是被裁剪或整段剔除，并非作为标注保留。

### 几何与结构化标注（相机参数、深度、3D point tracks、动作标注、显式状态） ⚠️

① Mochi 1：无披露。架构侧使用扩展到三维的可学习 RoPE 位置编码，但这属于模型设计而非数据标注。[不确定]
② MAGI-1：仅有语义级的相机结构化标注，无数值化几何标注。caption 属性表中包含 Camera Shot Type（景别）、Camera Movement（相机运动）、Camera Transitions（转场）、Subject Position（主体在画面中的位置）、Subject Action（主体动作）五项与结构/几何相关的属性，但均以自然语言描述形式经由 caption 传递，未做相机内外参估计、深度图、3D point tracks 或骨骼动作标注。过滤侧的几何相关量是 RAFT 光流的标量统计（整体/前景/背景三个运动强度）与光流散度、光流一致性，这些只用于判别不作为条件保留。
③ Motif-Video 2B：同样是语义级而非数值级。VLM 结构化标签中的 camera_move 字段以及 video prompt 索取的「shot type、angle、motion」三项相机属性构成了可用于条件化的相机描述，但仍是文本形式；未做相机参数回归、深度、3D 追踪或显式状态标注。几何相关的数值处理只出现在预处理（cropdetect 的内容矩形、OCR 检测框的 IoU 聚类、SSCD 512 维描述子）与过滤（UniMatch 光流分）中。
三者均未采用 2025–2026 年部分工作（如引入相机轨迹参数、点追踪、深度条件）的稠密几何标注路线。

### 合成数据构造（受控扰动/编辑构造训练对，如InstructAV2AV） ⚠️

① Mochi 1：无披露。[不确定]
② MAGI-1：数据全部为真实素材经切分筛选而来，未构造受控扰动/编辑训练对。唯一的「合成」环节是文本侧——caption 由 MLLM 生成（合成标注），以及约 200 万条用于蒸馏 Prompt Enhancement 小模型的合成 prompt 改写语料。
③ Motif-Video 2B：本组中唯一在视觉侧显式使用合成数据的。原始池被拆为 Image Real / Image Synthetic / Video Real / Video Synthetic 四个分支，即图像与视频两侧都设有合成支路。关键的调度约束是：「Synthetic video is injected only at 720p, where its controlled quality is most compatible with the admission criteria（合成视频只在 720p 阶段注入，因为其可控的质量最契合该阶段的准入标准）」——也就是把合成数据当作高质量阶段的补充而非预训练的填充料。但论文未说明合成视频由何模型生成、占比多少、如何避免自蒸馏式的分布塌缩，也未做合成 vs 真实的消融。同样未构造 InstructAV2AV 式的成对编辑数据。[部分不确定：合成数据的生成方式、占比与影响]

### 人工介入程度（人工标注、人工质检、模型初筛+人工复核） ⚠️

① Mochi 1：数据侧无任何人工介入的披露。评测侧使用了自动化指标——官方沿用 OpenAI DALL-E 3 的 prompt adherence 评测范式，以 Gemini-1.5-Pro-002 作为评判模型（即用大模型代替人评）。[部分不确定：数据侧是否有人工质检]
② MAGI-1：数据管线全自动（PySceneDetect + 11 类 Filter Actor + 双模型去重 + MLLM 过滤与打标），未提及人工标注或人工质检。人工介入集中在评测环节且做得相当重：团队自建了一套内部人评基准——设计分层的指标体系（强调完备性优先于简洁性，并要求指标之间正交以避免冗余），组织为 Overall / Motion Quality / Instruction Following / Visual Quality 四个主维度并各自细分子指标；基准数据集由 100 组精选的图像—prompt 对构成，来源刻意多元化（现有视频生成平台的用户真实输入、FLUX 生成的合成图像、公开图库的真实照片、专业影视素材四类），每个样本按指标框架标注具体评测目标；评测采用严格双盲的成对比较（Win/Tie/Lose），评委为「具备良好审美训练的专家」。结论是整体优于开源的 Wan-2.1、明显优于 Hailuo(i2v-01) 与 HunyuanVideo、略逊于商业模型 Kling1.6(HD)，其中指令跟随与运动质量为强项、视觉质量仍有差距。
③ Motif-Video 2B：数据管线全自动，无人工标注/质检。但有一处「人在环」体现在训练调度而非数据标注上——SFT 语料的组装是迭代式的：跑中间评测 → 由人判断哪些 subject 类目最弱 → 针对性补数据 → 再评测。论文还把整个训练描述为「a diagnostic loop rather than a single forward pass through a predefined schedule（一个诊断循环而非一次性走完预定义课程）」，Stage 9 的 Shared Cross-Attention 中途插入正是因为人观察到 720p 阶段出现语义对齐回退。评测侧主要依赖 VBench 自动指标（Table 3，16 维），未报告人评。

## 音视频对齐

### 音视频同步检测方法（口型同步、事件对齐）

不适用。三个模型均为无声视频生成，训练数据不含音轨，不存在口型同步或音画事件对齐的检测环节。

### 同步检测具体指标与阈值（SyncNet/Synchformer/LSE/自研，阈值数值，如MOVA LSE-D≤9.5且LSE-C≥4.5、SkyReels-V4 SyncNet |offset|≤3∧conf>1.5）

不适用。三者均未使用 SyncNet、AV-align 或任何自研音画同步指标，无相应阈值。

### 时序同步vs语义同步的分离处理（时间对齐与内容语义匹配作为两个独立过滤条件）

不适用（无音频）。若把该方法论映射到纯视觉侧，本组中确有明确的「时序质量」与「语义匹配」分离处理：
· MAGI-1 把时序侧（RAFT 光流的整体/前景/背景三档运动强度上下阈值、相机稳定性的光流一致性、幻灯片运动的光流散度）与语义侧（Transition Detection 用 CLIP 关键帧语义相似度判多镜头、MLLM 高级过滤做语义级复筛）拆成流水线上不同层级的独立条件。
· Motif-Video 2B 的分离更彻底且互为补充：底层时序信号是 UniMatch 光流的双侧截尾，语义信号是 VLM 的 action=Dynamic 标签，两者在 720p SFT 阶段被同时施加——前者管「像素层面动没动」，后者管「语义层面算不算动态动作」。同理，SigLIP 相似度用于时序连续性判定（stitch 合并），VLM 的 multi_scene 标签用于语义层面的多镜头判定，二者亦是「时序 vs 语义」的双轨兜底。
这一「低层统计 + 大模型语义」双轨过滤的结构，与 AV 模型中时序同步与语义同步分离处理在方法论上同构。

### 音频质量过滤（SNR、静音检测与静音占比阈值、无音轨剔除、画外音源剔除、背景音乐分离）

不适用。三者的数据管线在切分阶段即丢弃音轨（MAGI-1 与 Motif 的管线图中均只有视觉分支），无 SNR、静音检测与静音占比阈值、无音轨剔除、画外音源剔除或背景音乐分离等处理。

### 语音/音效/音乐的分类与分别处理策略

不适用。无语音/音效/音乐的分类与分别处理策略。

## 训练配合

### 多阶段训练课程与数据课程调度（阶段划分依据：分辨率/时长/质量分/模态；低清→高清、图像→视频、短→长） ⚠️

① Mochi 1：未披露训练课程。仅知模型「从零训练」、全 3D 注意力覆盖 44,520 视频 token 的上下文窗口，输出 480p；官方称 720p HD 版本在后续推出（实际未按期完整开源）。[不确定]
② MAGI-1（三阶段，调度轴为分辨率 + 时长 + 数据质量 + caption 类型，Table 5）：
· stage-1：分辨率 256p/360p，视频时长 ≤8 秒，图像:视频 = 4:1，AR caption 占比 0%。
· stage-2：分辨率 480p，时长 ≤8 秒，图像:视频 = 4:1，AR caption 占比 10%。
· stage-3：分辨率 720p，时长 ≤16 秒，图像:视频 = 4:1，AR caption 占比 10%。
论文明确「随分辨率提升，数据量逐步减少，并采用更严格的过滤策略以保证更高的数据质量」；最后阶段把时长上限从 8 秒放宽到 16 秒，让模型捕捉更丰富的时序动态。值得注意的是 MAGI-1 的图像:视频比例在三个阶段恒为 4:1（不随阶段调整），而 AR caption 从第二阶段起才引入且固定 10%——即「自回归可控性」是作为一个恒定的小比例辅助任务贯穿中后期，而非渐进加码。
③ Motif-Video 2B（十阶段，调度轴为模态 + 分辨率 + 帧数 + 文本编码器 + REPA 教师 + 架构变更，Table 1，是本组中最复杂也最「非线性」的课程）：
· Stage 1：T2I，144p，1 帧，句向量编码器，REPA 教师为 DINOv2 —— 空间自举（spatial bootstrap），在引入时序复杂度前先初始化空间生成通路。
· Stage 2：T2IV，144p，33 帧，句向量，REPA 教师换为 V-JEPA。
· Stage 3：T2IV，144p，65 帧，句向量，V-JEPA。
· Stage 4：T2IV，360p，65 帧，文本编码器换为 T5Gemma2，REPA 关闭（依据是早期收敛后表征对齐会变得适得其反）。
· Stage 5：T2IV，480p，65 帧 —— 先走「分辨率桥」：在引入 480p 视频之前，先用 360p 视频与 480p 图像联合训练，让模型从更便宜的图像上获得高分辨率空间特征，再让时序通路适应增大的 token 量。
· Stage 6：T2IV，480p，121 帧。
· Stage 7：T2IV，480p，121 帧 —— 第一次 SFT。
· Stage 8：T2IV，720p，121 帧 —— 关键的非常规决策：从 480p 的 SFT checkpoint（而非 480p 预训练 checkpoint）起步。
· Stage 9：T2IV，360p，121 帧 —— 回落分辨率，引入 Shared Cross-Attention 做交叉注意力精修（因 720p 阶段观察到语义对齐回退）。
· Stage 10：T2IV，720p，121 帧 —— 最终预训练 + 第二次 SFT。
另有两条贯穿性设计：所有视频阶段均为图文联合训练（图像样本稳定逐帧视觉质量并强化语义接地，视频样本驱动时序建模）；classifier-free guidance 训练的 prompt dropout 概率固定 p=0.1，噪声调度与损失加权不做任何修改。
 Motif 课程最值得注意的是它承认自己不是一次跑完的直线——「把训练当作诊断循环而非一次性走完预定义课程」，Stage 9 的分辨率回落与架构中途插入即为证据；以及 Stage 8 从 SFT checkpoint 起跑的假设（SFT 已把密度集中到高质量流形上，720p 阶段因而继承了更干净的起始分布、可把容量分配给分辨率适配而非同时补救预训练损失的质量），论文诚实标注该决策「未做与从预训练 checkpoint 起步的对照消融」，只作为务实的配方决策报告。

### 数据配比随训练阶段的变化（预训练/退火annealing/SFT高质量子集） ⚠️

① Mochi 1：未披露。[不确定]
② MAGI-1：配比的显式部分见 Table 5——图像:视频恒定 4:1（三阶段不变）、AR caption 占比 0%/10%/10%、时长上限 8s/8s/16s、分辨率 256p·360p/480p/720p；数据量随阶段递减、过滤阈值随阶段收紧（无数值）。配比的隐式部分是「动态分布调整」：训练全程按评测反馈自适应改变各语义子集的采样比例以强化短板（如人物表情难学则加码），这使得 MAGI-1 的实际数据配比是一条随训练演化的曲线而非一张静态表。另有一项任务级配比：因产品以 I2V 为主，训练中给 I2V 任务分配了更大比例。无独立的 annealing / SFT 阶段划分。[部分不确定：动态调整的具体幅度与语义子集划分]
③ Motif-Video 2B：混合策略同时体现在三个层次——
· 模态层：所有视频阶段均图文联合训练；训练时用「固定的图像—视频交错调度」（示例模式为 I-V-V，完整调度由规划出的各桶步数决定），以保证图文混合比例在训练全程稳定。
· 质量层：每次阶段跃迁重新施加分辨率/时长/运动/美学过滤且阈值更严；quality=low 自 480p 起排除；720p SFT 额外叠加 domain-balancing 与 action=Dynamic 动态运动准入。
· 来源层：合成视频只在 720p 阶段注入。
此外还有一层工程性的「配比」：帧数桶（1/33/65/121）× 分辨率桶的联合分桶必须在所有 rank 上均衡，否则 FSDP2 同步训练下某个桶的更新会被最慢 rank 拖住；离线 bucket 均衡采样器通过模拟退火最小化各 rank 上各桶片段数的变异系数来解决这一问题（见 data_infra_throughput）。SFT 语料的类目配比则由中间评测反馈迭代决定（Fig.8）。

### 后训练数据（SFT精选集规模与筛选标准、偏好对数量与标注方式、reward model训练数据） ⚠️

① Mochi 1：未披露任何后训练数据。既无 SFT 精选集说明，也无 DPO/RLHF、偏好对或 reward model 的信息。[不确定]
② MAGI-1：技术报告未设独立的 SFT 章节，也未描述偏好对齐后训练（无 DPO/RLHF、无偏好对数量、无 reward model 训练数据）。其「后训练」实质由两部分承担：(i) 多阶段训练的最后阶段本身即以更严过滤的高质量数据、更长时长（≤16s）承担精修职能；(ii) Shortcut Model 蒸馏——采用 shortcut model 作为蒸馏目标，最小步长为 1/64（对应标准 flow-matching 的 64 次函数求值），蒸馏步长从集合 [1/64]×8 ∪ [1/32, 1/16, 1/8] 中循环采样，使单个蒸馏模型可在 64/32/16/8 步等不同算力预算下推理；最小步长训练时并入 classifier-free guidance 蒸馏。此外推理侧的 Prompt Enhancement 小模型（约 7B）用约 200 万条 MLLM 生成的增强 prompt 语料蒸馏而成，并过滤掉目标文本过长的样本以控制输出长度。[部分不确定：是否存在未披露的偏好数据]
③ Motif-Video 2B：明确设有两次 SFT——Stage 7（480p）与 Stage 10（720p），各自在「Section 5 所述的精选高质量子集」上进行。论文对 SFT 的目的表述得很直接：把模型输出分布推向训练数据的高质量尾部，改善美学质量、运动平滑度与 prompt 遵从度，这是「在松散过滤数据上做广泛预训练所无法实现的」；并指出这已是视频生成的标准做法（列举 HunyuanVideo 1.5、Wan2.1、SANA-Video、Seedance 1.5、SkyReels-V2 均报告了专门的 SFT 阶段）。SFT 数据的筛选标准为在常规过滤基础上叠加更严的美学截断、domain-balancing（由 style/subject 标签驱动）以及视频侧的 action=Dynamic 动态运动准入；语料组装为迭代式补短板（见 domain_distribution）。非常规之处在于 720p 预训练从 480p SFT checkpoint 起步而非从预训练 checkpoint 起步（详见 multi_stage_curriculum）。同样未使用偏好对齐类后训练（无 DPO/RLHF、无 reward model）。SFT 精选集的绝对规模未给。[部分不确定：SFT 精选集规模与美学阈值数值]

### 数据处理基础设施与吞吐（NeMo Curator/Data-Juicer/自研，GPU加速比，处理规模，成本） ⚠️

① Mochi 1：未披露数据处理基础设施。训练侧规模亦未公开；仅知推理侧单卡需约 60GB 显存。[不确定]
② MAGI-1：报告称构建了「a scalable data processing system（可扩展的数据处理系统）」以从数十 PB 原始素材中构造训练集，但未点名框架（未提 NeMo Curator / Data-Juicer 等）、未给 GPU 加速比、处理机时或成本。可推断的成本量级线索是：需在 PB 级素材上跑 PySceneDetect、DOVER、LAION 美学、RAFT（已下采样到 8 FPS 以省算力）、显著性检测、Hough 边框检测、文字检测、Florence-2、人脸检测、CLIP、DINOv2 共 10 余类模型推理，再对幸存数据跑 MLLM——把 MLLM 过滤与打标合并为一次调用正是为控制这部分成本。
训练侧基础设施披露则非常充分（占技术报告 Section 4 大篇幅）：采用 DP + CP + TP 三维并行；针对变长视频序列导致的 DP 负载不均与短序列 GPU 利用率不足，设计了在线分布式 Packing and Padding（PnP）——把 bin-packing 问题重述为「从 M 个候选样本中打包进 N 个容量为 max_length 的 bin，M ≫ N，N 须被 DP_SIZE 整除，max_length 须被 TP_SIZE × CP_SIZE 整除」，用扩展的 First-Fit Decreasing 贪心加自定义启发式在线求解，达到 99% 的容量利用率且各 DP 组间差异可忽略。之所以必须在线而非离线打包，正是因为「训练期间数据构成会被频繁调整」（呼应 3.5 的动态分布调整）——这是数据策略反过来约束基础设施设计的一个典型案例。另贡献 MagiAttention 分布式注意力（Flex-Flash-Attention + dispatch solver + Group-Cast/Group-Reduce 通信原语 + 多级重叠），面向超长序列与异构 mask 实现线性可扩展。[部分不确定：数据清洗侧的框架、吞吐与成本]
③ Motif-Video 2B（本组中数据基础设施披露最完整，且给出了量化收益）：
· 清洗框架：「We made extensive use of NeMo Curator（我们大量使用了 NeMo Curator）」，称其可扩展的数据治理工具包与对大规模视频处理管线的支持显著简化了预处理工作流。
· 去重加速：NVIDIA cuVS 多 GPU IVF-PQ 索引（余弦距离，k=64，nprobe=16）。
· OCR 服务化：PaddleOCR-VL 经 vLLM 部署。
· 预处理落地：黑边裁剪、OCR 区域排除、分辨率缩放、帧率限制在单次 ffmpeg 重编码中一并完成，避免多次转码。
· 存储与采样（独立贡献，Section 5.3 + Table 2）：语料以 WebDataset tar 分片存储。基线做法（全局 clip 索引 + shuffle_block_size=1 随机打散 + round-robin 按 index mod W 分配）虽保留随机性，却破坏了归档局部性、牺牲了 WebDataset 顺序读的核心优势，并造成跨 rank 的桶构成严重失衡——在 FSDP2/HSDP 同步训练下，某个桶的全局步数取决于该桶上最慢的 rank，导致基线每 epoch 只有 N 步、利用率约 20%，dataloader 延迟 0.05 s/step。团队的离线 bucket 均衡采样器把过滤、分桶与 rank 分配全部移到离线规划阶段：先从 clip 级 Parquet 元数据施加过滤规则并分配「帧数×分辨率」联合桶，再贪心地把每个 tar 分片放到最能降低当前跨 rank 桶失衡的 rank 上，然后以模拟退火跑 30,000 次迭代（每次提议交换两个 tar 分片）优化分片到 rank 的映射，目标是最小化 1f/33f/65f/121f 各桶片段数在各 rank 间的变异系数；最终方案序列化为每 rank 的 rank{r}.npz 计划文件。运行时每个 rank 只按分片顺序读取自己被分配的 tar，配合 4096 样本窗口的滚动打散在不破坏读局部性的前提下保留桶内随机性。
· 量化收益（Table 2）：基线 N 步/epoch、约 20% 利用率、0.05 s/step；离线采样器 + 贪心为约 4.6N 步、约 76% 利用率、<0.001 s/step；离线采样器 + 模拟退火（最终方案）为约 5.4N 步、利用率接近 90%、<0.001 s/step。即相对基线吞吐提升约 5.4 倍、利用率从 20% 升至约 90%、dataloader 延迟降低 50 倍以上；SA 相对贪心再提升约 18%。
· 训练硬件：8 个 Azure 节点 × 8 张 H200 = 共 64 卡，Kubernetes 编排、SkyPilot 启动（负责调度、故障恢复与云资源供给），通过 Accelerate 使用 FSDP2；在 2B 规模下单个节点内分片组即可容纳 720p/121 帧配置的完整模型状态，因此不需要张量并行或序列并行，从而简化通信模式、减少同步开销。混合精度为 bfloat16 计算与激活、float32 保留归约敏感通信与优化器状态。
这一节是本组三个模型里唯一把「数据基础设施的改进 → 可量化的训练吞吐收益」完整打通的，对「小预算做视频模型」的实践参考价值最高。

## 效果对比

### 数据策略消融的量化影响（区分：过滤严格度ablation / caption密度风格ablation / 数据配比ablation，及对应评测指标） ⚠️

① Mochi 1：无任何数据策略消融，亦无数据披露。评测只有 prompt adherence（沿用 DALL-E 3 协议，以 Gemini-1.5-Pro-002 为评判）与运动质量的定性/自动指标。[不确定]
② MAGI-1：数据侧的消融基本缺失——没有过滤严格度消融、没有 caption 密度/风格消融、没有数据配比消融的量化结果。报告中与数据决策相关的实证性表述均为「经验发现」而非对照实验，例如「经实证评估我们发现 DOVER 的 technical score 单独使用是最有效的指标」、「经实证分析每片段用 4–12 帧在描述准确度与计算效率间达到最佳权衡」、「训练中观察到风景易学、人物表情难学」——这些是选型依据，但未给出被比较项的指标数值。端到端评测则相当充分：自建 100 组图像—prompt 对的双盲专家人评（四主维度 + 子指标）、VBench-I2V（含 1× 解码器 1280×720 与 2× VAE 上采样 2560×1440 两个变体）、以及 Physics-IQ 物理理解基准（8 秒真实物理现象视频，给前 3 秒预测后 5 秒；MAGI 的 V2V 变体使用完整 24 FPS 的 96 帧输入）。因此其数据设计的有效性同样靠最终排名间接背书。
③ Motif-Video 2B：数据过滤各组件本身「未做隔离消融，通过端到端模型表现评估」，论文对此态度坦诚，多处主动标注局限——caption 三变体的 0.5/0.3/0.2 采样概率被明说是「务实的配方选择而非经隔离验证的最优性主张」；从 480p SFT checkpoint 起跑 720p 的决策「未与从预训练 checkpoint 起跑做对照」，只作为可复现性记录报告；早期训练效率的检查（在 6.5×10²⁰ FLOPs 下本模型 FID 15.5 vs 该 scaling law 对 vanilla DiT 预测的 FID ≈ 30）也被明确标注「与并行的 REPA 及架构差异混淆，只当作一致性检查而非对早期课程设计的独立验证」。
真正做了量化对照的是数据基础设施侧（Table 2，见 data_infra_throughput）：基线 vs 离线采样器+贪心 vs 离线采样器+SA 三档，给出步数/epoch（N / 4.6N / 5.4N）、利用率（20% / 76% / 90%）与 dataloader 延迟（0.05 / <0.001 / <0.001 s/step）的完整对照，这是本组唯一一组严格的数据侧量化消融——只不过它衡量的是吞吐而非生成质量。
端到端结果（Table 3，VBench 全 16 维）：Total 83.76%，超过 Wan2.1-T2V-14B（83.69%）、HunyuanVideo（83.24%）、Step-Video-T2V-30B（81.83%）；Wan2.2-T2V 的 84.23% 因使用 prompt 优化被单独归类不作同口径比较。强项在语义侧——Spatial Relationship 83.02%（在有完整分维结果的开源模型中领先）、Object Class 92.93%、Multiple Objects 77.29%、Semantic Score 80.44%；弱项在质量侧——Subject Consistency 95.38%、Background Consistency 95.74% 低于最强的 Wan 系列，Temporal Flickering 98.16% 亦落后于 Wan2.1 家族（最高 99.55%）。论文把这解读为「特定的取舍而非全面胜出」。

### 质量vs数量的证据（小而精数据超越大而杂的案例） ⚠️

这正是本条目合并调研的核心看点，三者提供了强度递增的证据链：
① Mochi 1：无证据（无数据披露）。[不确定]
② MAGI-1：提供的是结构性证据——从数十 PB 原始素材出发，经 11 类 Filter Actor + CLIP/DINOv2 双模型严格去重 + MLLM 语义复筛的层层收缩，并明确「随分辨率提升数据量逐步减少、过滤策略更严格以保证更高质量」；引用 LLM 文献论证「即便少量重复数据也会显著损害性能」作为严格去重的动机。但既无保留率数字，也无「小而精 vs 大而杂」的对照实验，属于设计哲学层面的间接证据。[部分不确定]
③ Motif-Video 2B：本组中最强、也是整个调研中较为罕见的一类证据——它把「质量胜过数量」直接做成了论文的中心命题并给出可比较的结果。论文开篇即设问：「训练强视频生成模型通常需要海量数据、巨大参数量与大量算力。本工作追问：在小得多的预算下——少于 1000 万片段、少于 10 万 H200 GPU 小时——是否也能达到强文生视频质量？」，并指出当前最强开源模型（Wan2.1、HunyuanVideo、Seedance）「训练于数亿条精选片段、参数量 5B–14B」，「这种资源集中虽产出了亮眼结果，却也收窄了参与面：实际上能训练有竞争力的视频生成模型的团队极少」。结论是：2B 参数、少于 1000 万片段、10 万 H200 小时内训练，VBench 达 83.76%，「超过 Wan2.1-14B，参数少 7 倍、训练数据少一个数量级」。
方法论上支撑这一结论的核心表述是：「Rather than maximizing raw scale, we prioritize curation quality to support resource-efficient training（我们不追求最大化原始规模，而是优先保证治理质量以支撑资源高效训练）」。
需要保持的审慎：Motif 的胜出同时来自架构（Shared Cross-Attention、三段式骨干功能分解）与训练技术（TREAD token 路由、REPA/V-JEPA 早期表征对齐、两次 SFT 与从 SFT checkpoint 起跑 720p），论文并未把「数据治理质量」的贡献从这些因素中隔离出来做消融；且其优势集中在语义维度而质量维度（主体/背景一致性、时序闪烁）仍落后于 Wan 系列，论文自己也把这读作「特定取舍而非全面胜出」，并明确指出「长时程时序稳定性与外观一致性仍是进一步扩大规模与改进数据的主要目标」。因此这是一个强但非隔离的证据。

### 训练数据domain分布与评测基准类目体系的对齐关系（如VABench七大类） ⚠️

① Mochi 1：无对齐关系可述（数据分布未披露）。评测用自动 prompt adherence 协议（DALL-E 3 范式 + Gemini-1.5-Pro-002 评判）与运动质量定性展示。[不确定]
② MAGI-1：不做静态类目对齐，但通过「动态分布调整」建立了一条评测→数据的反馈通路：持续监控训练中的评测表现，对模型表现薄弱的语义概念子集调高采样比例。这实质上是把「评测类目」当作数据配比的目标函数，比静态映射更强，但论文既未公开语义概念的类目体系，也未说明所用评测与 VBench 十六维或自建人评四维（Overall / Motion Quality / Instruction Following / Visual Quality）之间的对应关系。另有一处对齐是任务级而非类目级：因产品与评测均以 I2V 为主（团队明确表示「I2V 更贴近真实使用模式：用户通常从图像而非文本生成视频」），训练中就给 I2V 分配了更大比例，评测也主要用 VBench-I2V 而非 T2V track——这是「训练配比对齐评测/产品形态」的清晰案例。[部分不确定：语义类目体系与评测维度的具体映射]
③ Motif-Video 2B：建立了本组中最显式的对齐闭环——VLM 产出的 subject / style 标签既用于 720p 与 SFT 的领域均衡，又是 Fig.8 类目分布图的统计口径；而 SFT 语料的补充方向直接由「中间评测中生成质量最弱的 subject 类目」决定，最终形成的视频侧分布向 Transportation / Sports / Animals 倾斜，正是因为这三类涉及剧烈动态、在评测中被识别为弱项。这条「评测发现弱项 → 按 subject 标签补数据 → 再评测」的闭环，使训练数据的类目分布在事实上被评测结果塑形。
不过与 VBench 十六维类目体系之间并无形式化映射：Motif 的 subject 类目（People、Transportation、Sports、Animals 等）是内容主体分类，而 VBench 的维度是能力分类（Object Class、Multiple Objects、Spatial Relationship、Human Action、Subject Consistency、Temporal Flickering 等），两者只在 Human Action 等少数维度上朴素对应。论文也未按 VBench 弱项维度（如 Subject Consistency、Temporal Flickering）反向补数据，而是承认这些是「进一步扩大规模与改进数据的主要目标」。[部分不确定：与 VBench 类目体系的形式化映射]

## 不确定字段

以下字段的调研信息部分不确定（⚠️ 标注来源）：

- data_scale
- data_sources
- provenance_licensing
- duration_distribution
- resolution_aspect_distribution
- domain_distribution
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
- geometric_structured_annotation
- synthetic_data_synthesis
- human_in_loop
- multi_stage_curriculum
- stage_data_mixture
- post_training_data
- data_infra_throughput
- data_ablation
- quality_over_quantity_evidence
- benchmark_taxonomy_alignment
