# Seedance 2.0 与 Seedance 1.5 pro（含 Seedance 1.0 作为数据管线纵向基线）

> 主题: 视频生成模型（含音视频同时生成）的数据处理：数据清洗流程、数据分布、打标方式

[← 返回首页](../index.md)

**目录**: [基本信息](#基本信息) · [数据规模与分布](#数据规模与分布) · [清洗流程](#清洗流程) · [打标方式](#打标方式) · [音视频对齐](#音视频对齐) · [训练配合](#训练配合) · [效果对比](#效果对比)

## 基本信息

### 名称

Seedance 2.0 与 Seedance 1.5 pro（含 Seedance 1.0 作为数据管线纵向基线）

### 发布机构/公司

字节跳动 Seed 团队（ByteDance Seed / Team Seedance）

### 发布时间（技术报告/论文/开源时间）

Seedance 1.5 pro：技术报告 arXiv:2512.13507，v1 于 2025-12-15 提交、v3 于 2025-12-23 更新，产品于 2025 年 12 月上线豆包/即梦/火山方舟（模型 ID：Doubao-Seedance-1.5-pro）。Seedance 2.0：2026 年 2 月初在中国正式发布（模型 ID：doubao-seedance-2-0-260128），技术报告 arXiv:2604.14148 于 2026-04-15 提交，火山引擎 API 于 2026-04-14 全面开放。作为纵向对比基线的 Seedance 1.0 技术报告为 arXiv:2506.09113（2025-06）。

### 类型（模型/数据集/工具链/评测基准）

闭源商用模型（视频/音视频联合生成基础模型）+ 配套自研评测基准（SeedVideoBench 1.0/1.5/2.0）。两份技术报告本身更接近 model card + 评测报告，而非完整方法论论文。

### 开源程度（权重/代码/数据/pipeline各自是否开源）

权重：不开源；代码：不开源；训练数据：不公开；数据处理 pipeline：仅 Seedance 1.0 报告给出较完整的文字描述（无代码、无阈值、无开源工具），1.5 pro 仅一段要点式描述，2.0 报告完全未披露数据章节。对外仅提供 API/产品访问（火山方舟 Ark、豆包 Doubao、即梦 Jimeng，以及 Replicate 等第三方托管）。

### 是否支持音视频同时生成，以及实现方式（原生联合/级联/MoE融合） ⚠️

支持。Seedance 1.5 pro 为原生音视频联合生成（native joint audio-video generation）：采用双分支 Diffusion Transformer（报告摘要表述为 dual-branch DiT，正文表述为基于 MMDiT 的统一框架）+ 跨模态联合模块（cross-modal joint module），在大规模混合模态数据上做多任务预训练，同时支持 T2VA、I2VA 以及纯视觉的 T2V/I2V，因此属于「原生联合」而非级联式先视频后配音。Seedance 2.0 进一步升级为统一的多模态音视频联合生成架构（unified multi-modal audio-video joint generation architecture，官方博客提及采用稀疏架构 sparse architecture），支持文本/图像/音频/视频四种输入模态；音频侧新增双声道（binaural/双通道立体声）能力，并支持背景音乐、环境音效、角色人声旁白的多轨并行输出，与画面节奏精确时间对齐。[不确定：2.0 的具体融合方式（是否为 MoE 融合）未在报告中说明]

### 调研信息来源列表（论文/技术报告/官方文档/新闻的URL，标注每条来源的性质：官方一手/同团队旁证/第三方报道）

- 官方一手: Seedance 2.0 报告 (arXiv:2604.14148, 26页, 仅引言+评测+贡献者, 无数据章节) https://arxiv.org/abs/2604.14148
- 官方一手: Seedance 1.5 pro 报告 (arXiv:2512.13507, v1-v3全部核对, 无数据章节) https://arxiv.org/abs/2512.13507
- 官方一手: Seedance 1.0 技术报告 (arXiv:2506.09113, 含完整Data章节, 本条目数据管线的可考证基线) https://arxiv.org/abs/2506.09113
- 辨伪说明: emergentmind等AI摘要站流传的'1亿分钟训练数据/四阶段pipeline'经三版PDF全文检索零命中, 判定为幻觉, 不采信

## 数据规模与分布

### 训练数据量级（视频条数/小时数/token数，预训练与SFT分开） ⚠️

[不确定] 两份报告均未披露任何训练数据量级数字（视频条数、小时数、token 数，预训练与 SFT 均未给出）。经全文检索，Seedance 1.5 pro（v1/v3）与 Seedance 2.0 报告中均不含 million/hours/minutes 等规模表述。网络上流传的「约 1 亿分钟（~100 million minutes）野外音视频片段」说法来自 emergentmind 等 AI 自动摘要站点并声称引自该论文，但原文并无此数据，属未经证实的二手推断，不应采信。Seedance 1.0 报告同样未给出绝对数据量。

### 数据来源构成（自有/公开数据集/网络爬取/授权采购/合成数据） ⚠️

Seedance 1.5 pro 仅表述为「大规模混合模态数据集」，未拆解来源构成。Seedance 2.0 未披露。可参照同团队 Seedance 1.0 报告的表述：以「合乎伦理与法律的方式获取」（ethically and legally sourced），来自多样化的公开仓库与已授权仓库（diverse public and licensed repositories），并与图像数据联合训练（图像数据准备沿用 Seedream 的方法论）。自有/爬取/采购/合成各自占比均未披露。[不确定：1.5 与 2.0 的具体来源构成与配比]

### 数据合规与溯源（授权数据占比、rights-cleared数据集、C2PA等） ⚠️

仅有原则性表述，无量化披露。Seedance 1.0 报告称数据获取「优先选择伦理与法律上合规的内容，来自多样化的公开与授权仓库」，并在过滤环节部署分类器剔除色情、暴力、儿童剥削、露骨裸露等内容以保证「伦理合规与数据集安全」。Seedance 2.0 报告在引言末尾声明「安全是我们工作的核心考量」，在模型迭代全生命周期实施了结构化的安全评估框架并持续评估与缓解潜在风险，以支持负责任、合规、符合伦理的开发。[不确定：授权数据占比、rights-cleared 数据集清单、是否采用 C2PA/水印溯源等均未披露]

### 片段时长分布与切分策略 ⚠️

[不确定] 1.5 与 2.0 均未披露训练片段时长分布。可参照 Seedance 1.0：原始长视频经镜头感知时序切分后，切片最大时长为 12 秒；预训练第一阶段使用 3–12 秒（12 fps）的视频片段与 256px 图像联合训练。生成侧可反推数据具备的时长覆盖：1.5 pro 支持约 10 秒级生成，Seedance 2.0 支持 4–15 秒的直出音视频生成，官方博客提及 15 秒高质量多镜头音视频输出，暗示训练数据已覆盖 15 秒量级的多镜头片段。

### 分辨率/宽高比分布与分桶策略 ⚠️

[不确定] 未披露分辨率/宽高比的分布统计与分桶（bucketing）策略。可参照 Seedance 1.0 的渐进式分辨率课程：先以 256px 文生图充分训练 → 256px 图文视频联合训练 → 提升至 640px → 最后提升到 24 fps。Seedance 1.0 在数据分布再平衡（Distribution Rebalancing）时明确把「分辨率」列为需要统计频次并做上/下采样的属性维度之一。Seedance 2.0 的原生输出分辨率为 480p 与 720p（报告特别指出其在 720p 下的 Arena Elo 已超过对手的 1080p 模型）。

### 类别/domain分布与配比策略（人物、动作、场景、风格等比例控制与概念均衡） ⚠️

1.5 与 2.0 的训练数据类目分布未披露，但可从两条侧证还原其配比意图：(1) Seedance 1.0 报告的「多样性导向数据采集」明确列出需最大化覆盖的维度：片段时长、分辨率、主体（人/动物/物体）、场景类型（自然风光/城市环境）、主体动作、体裁（纪录片/动画）、艺术风格、相机运动学、摄影技法；并在「分布再平衡」环节按主体类别、场景类型、主导动作、体裁、视觉风格、片段时长、分辨率、运动特性等维度统计频次，对过度表征的头部类目做下采样（downsampling），对长尾类目提高训练时采样概率（increase sampling probability）并发起定向数据补采（targeted data acquisition），目标是对视觉世界形成「更公平、更全面的表征」；SFT 阶段则定义了「数百个类别」（several hundred categories，按视觉风格、运动类型等关键属性划分）做定向采集。(2) Seedance 1.5 pro 报告称其数据管线优先保障「视频-音频一致性、运动表现力（motion expressiveness）与基于课程的数据调度」，说明配比上显式向高运动表现力样本倾斜——这与其在 SeedVideoBench 1.5 中新增「视频生动性（Video Vividness）」指标（拆分为动作、运镜、氛围、情绪四个维度）互为呼应，报告并批评业界普遍用慢动作换取稳定性的做法。Seedance 2.0 侧则强化了复杂交互与人体运动、多主体交互、多镜头叙事、中文方言/戏曲/唱段等类目。[不确定：所有具体比例数值]

### 音频类别分布与配比（语音/音效foley/音乐/环境音/静音的比例及控制策略）——AV模型独有维度 ⚠️

[不确定] 训练侧的音频类别配比未披露。但 Seedance 1.5 pro/2.0 的评测标注体系高度细化，可视为其数据类目体系的镜像：SeedVideoBench 1.5 的音频标签主类为 —— 人声类型（Human Voice Types：语音 speech、歌唱 singing、非言语发声 non-verbal vocalizations 如笑声，并含细粒度子维度）；人声属性（Human Voice Attributes：音色 timbre、口音 accent、情绪基调 emotional tone）；非语音音频（Non-Speech Audio，含音效与音乐，按声源如动物/机械工具、声学属性、音乐流派、技术参数分类）。SeedVideoBench 2.0 的音频细粒度类目扩展为 17 类：中文方言/口音、中文多人对话、中文综艺人声、中文戏曲、英语、少数民族语言、歌唱/说唱、空间场景音、画外音（off-screen voice）、非言语人声、人声+动作交互音、物体交互音、动物叫声、环境/背景音、特殊音效（含 ASMR）、乐器与音频、双声道音频。Seedance 2.0 明确支持背景音乐 / 环境音效 / 角色配音三类音轨的多轨并行输出。静音样本比例、语音:音效:音乐的具体配比均未披露。

### 叙事结构分布（单镜头vs多镜头、平均clip时长、镜头数分布、是否含原生音轨） ⚠️

Seedance 1.0 的切分策略明确允许「每个切片可包含一个或多个时序连贯的镜头」（one or multiple temporally coherent shots），以保留局部叙事流，这是其多镜头生成能力的数据基础。Seedance 1.5 pro 强调「叙事连贯性（narrative coherence）」与多镜头视频生成工作流的应用潜力，并具备连续长镜头、希区柯克变焦（dolly zoom）等运镜与电影级转场能力。Seedance 2.0 主打「原生专业级多镜头叙事能力」，可自主规划镜头序列（shot sequencing）并设计视觉呈现模板，SeedVideoBench 2.0 为此新增叙事质量（narrative quality）指标，含摄影语言（镜头逻辑与表现力，检查冗余覆盖、越轴/180 度法则违规、景别错配、节奏不均）、情节设计、风格化美学三个子维度。训练数据原生音轨占比、平均 clip 时长、镜头数分布等具体统计均未披露。[不确定：定量分布]

### 语言/口音分布（多语种唇同步能力的数据基础） ⚠️

训练数据的语种/口音比例未披露，但能力与评测覆盖可反推：Seedance 1.5 pro 原生支持多语种与多地区方言的唇形同步，可捕捉各自独特的语音韵律与情绪张力；Seedance 2.0 在中文方言、传统戏曲、歌唱场景上的指令跟随准确率显著提升。SeedVideoBench 2.0 音频评测显式区分中文方言/口音、中文多人对话、中文综艺人声、中文戏曲、英语、少数民族语言等语种类目（2.0 在音频质量上英语 4.17、少数民族语言 3.82、中文方言 2.82；音画同步上英语 4.17、少数民族语言 3.88、中文方言 3.64），说明训练语料以中英为主并覆盖多种中文方言与少数民族语言。此外 Seedance 1.0 的 caption 模型即在中英双语数据上训练。[不确定：各语种具体占比]

## 清洗流程

### 清洗漏斗整体结构（几级过滤、各级顺序）

Seedance 1.5 pro 只给出高层描述：一个整体性的数据框架（holistic data framework），由「多阶段筛选管线（multi-stage curation pipeline）+ 先进的字幕/描述系统（advanced captioning system）+ 可扩展基础设施（scalable infrastructure）」三部分构成，管线优先保障视频-音频一致性、运动表现力与基于课程的数据调度。Seedance 2.0 报告未设数据章节，完全未披露漏斗结构（该报告仅含引言、评测、贡献者三部分）。可作为纵向基线的 Seedance 1.0 报告给出了六级顺序漏斗：(1) 多样性导向数据采集 Diversity-Oriented Data Sourcing → (2) 镜头感知时序切分 Shot-Aware Temporal Segmentation（最长 12 秒）→ (3) 视觉覆盖物矫正 Visual Overlay Rectification（logo/水印/字幕/贴图检测 + 自适应裁剪）→ (4) 质量与安全过滤 Quality and Safety Filtering → (5) 语义去重 Semantic Deduplication → (6) 分布再平衡 Distribution Rebalancing；整套流程部署为面向海量数据高吞吐的自动化系统。注意：任务背景中「Seedance 2.0 数据侧披露最完整」的说法与实际报告内容不符——2.0 的 arXiv 报告是评测导向的 model card，数据披露程度反而低于 Seedance 1.0。

### 漏斗定量保留率（每级过滤的输入/输出量与最终保留率，如Apollo 27%） ⚠️

[不确定] 三代报告（1.0/1.5/2.0）均未给出任何一级过滤的输入/输出量或保留率数字，无法与 Apollo 27% 这类披露对标。这是 Seedance 系列数据披露的最大缺口。

### 镜头切分方法（PySceneDetect/自研模型/shot-aware splitting） ⚠️

Seedance 1.5/2.0 未直接描述。Seedance 1.0 的方法：采用自动镜头边界检测（automated shot boundary detection），通过分析帧间视觉差异（inter-frame visual dissimilarities）或使用预训练检测器识别自然场景切换；随后将视频切成最长 12 秒的短片段，每个片段可包含一个或多个时序连贯的镜头，以在保留局部叙事流的同时控制输入长度。未指明是否使用 PySceneDetect 等开源工具，倾向为自研/内部方案。[不确定：1.5 与 2.0 是否沿用同一方案、以及是否引入音频侧边界检测]

### 质量过滤（美学评分、清晰度、OCR文字过滤、黑边/水印/logo检测） ⚠️

Seedance 1.5 pro 仅称管线包含「片段质量」相关的筛选（第三方摘要称第二阶段为音画同步与片段质量的自动化过滤，但该表述不见于论文原文）。Seedance 1.0 的具体做法：(a) 视觉覆盖物矫正——用启发式规则系统 + 专用目标检测模型的混合方案识别 logo、水印、字幕、屏幕图形等遮挡，再对帧做自适应裁剪（adaptive cropping）以最大化保留主体视觉内容；(b) 质量过滤——由专用的「视觉质量模型」（specialized visual quality model）系统性识别并剔除模糊（blurriness）、剧烈抖动（excessive jittering）、低美学质量（low aesthetic quality）、构图/摄影质量差（poor cinematographic composition）、以及以静态内容为主（predominantly static content）的片段。Continue Training 阶段进一步用「一系列专用评估模型，包括美学打分器（aesthetic scorer）」筛出更高美学质量子集。所有阈值、打分器规模与分档均未披露。[不确定：具体阈值与 OCR 文字过滤是否独立成级]

### 运动过滤（光流/运动分数阈值、静态与抖动剔除） ⚠️

Seedance 1.0 在预训练过滤阶段剔除抖动过大与以静态内容为主的片段；在 Continue Training 阶段使用「基于光流的运动评估器（motion evaluators based on optical flow）」与美学打分器共同筛选出美学质量更高、运动动态更丰富的子集，报告称更丰富的运动动态使模型生成更自然流畅。Seedance 1.5 pro 把「运动表现力（motion expressiveness）」列为数据管线的三大优先目标之一，并在评测端引入生动性指标，明确反对以慢动作换稳定性。具体运动分数阈值未披露。[不确定：阈值数值]

### 去重方法（精确去重与基于embedding的语义去重分别记录） ⚠️

Seedance 1.0 采用语义去重（Semantic Deduplication）：用内部自研的视频表征模型（internally developed video representation model）提取鲁棒特征嵌入，基于嵌入对视觉与语义相近的片段做聚类；在每个近重复簇内仅保留质量总分（来自前一级质量过滤）最高的单个实例。报告未单独描述精确哈希去重（exact dedup）环节。1.5 与 2.0 未披露去重方案。[不确定：是否含精确去重、聚类阈值、去重比例]

### VLM/LLM作为质检员（多模态大模型质量打分与错配剔除，2026年从浅层打分器转向大模型语义判断的趋势） ⚠️

Seedance 系列已实质走向「模型即质检员」，但披露粒度有限：Seedance 1.0 使用专用视觉质量模型做缺陷判定、内部视频表征模型做语义聚类去重、Tarsier2 基座的自研 caption 模型做内容理解、以及以 VLM 为架构的 Foundational Reward Model 做图文对齐与结构稳定性评估。Seedance 1.5 pro 称其字幕系统能为视频与音频两种模态提供「丰富的专业级描述（rich, professional-grade descriptions）」，属于大模型语义判断路线。Seedance 2.0 报告未披露质检模型细节，但同团队具备 Seed-VL 系列多模态理解模型（引言列出用于跨模态语义理解），推测已用于数据判别环节。SeedVideoBench 2.0 评测层面明确区分「客观指标走自动化流水线、主观指标走盲审专家评审」。[不确定：1.5/2.0 是否引入 VLM 打分器及其规模、打分维度与剔除阈值]

### 安全与合规过滤（NSFW、版权、人脸/隐私） ⚠️

Seedance 1.0：部署先进分类器检测并剔除色情（pornography）、露骨暴力（explicit violence）、儿童剥削（child exploitation）、露骨裸露（explicit nudity）等有害或不当内容，以确保伦理合规与数据集安全。Seedance 2.0 报告声明在模型迭代全生命周期实施结构化安全评估框架并持续评估与缓解潜在风险。人脸/隐私、版权指纹等具体机制未披露。[不确定：人脸与隐私过滤、版权检测的具体方法]

## 打标方式

### 使用的caption模型（自研VLM/开源模型，模型规模） ⚠️

Seedance 1.0：caption 模型以 Tarsier2（强视频理解能力的开源模型）为基座，在人工高质量标注数据上训练，训练时冻结视觉编码器（visual encoder frozen）、对语言模型做全参微调（fully fine-tuned），并在中英双语数据上训练以获得双语能力；推理时用同源的 Prompt Engineering（PE）模型把用户 prompt 改写成与训练 caption 在内容和结构上对齐的详细视频描述。Seedance 1.5 pro 称其「先进的字幕系统」为视频与音频两种模态提供丰富的专业级描述，但未指明基座模型与参数规模。Seedance 2.0 未披露。[不确定：1.5/2.0 的 caption 模型身份与规模；推测已切换至自研 Seed-VL 系列]

### caption密度与结构化程度（短/长/密集描述、结构化字段如镜头运动、风格标签） ⚠️

Seedance 1.0 采用「密集描述（dense caption）」风格，融合动态特征与静态特征两类：动态特征细致描述片段中的动作与相机运动，突出变化元素，覆盖运动、主体或场景的变化、镜头运动三个类目；静态特征描述核心人物或场景的特征，覆盖外观、美学、风格等类目。团队为这些类目采集多样数据并进行高质量人工标注用于训练 caption 模型。Continue Training 阶段针对 I2V 任务设计了两种 caption：(1) 同时含动静态细节的原始长 caption；(2) 去除首帧对应静态描述、只聚焦运动动态的短 caption，以强化与训练目标的语义对齐。Seedance 1.5 pro 表述为「显著提升视频描述的丰富度与专业性，并引入音频描述」。[不确定：1.5/2.0 是否新增结构化字段 schema 及字段清单]

### 音视频联合caption结构（是否同时覆盖视觉+听觉轨道、是否分流为独立字段，如LTX-2全音景描述、Script-a-Video factorized streams、Foley-Omni三字段） ⚠️

Seedance 1.5 pro 是该系列首次引入音频侧描述的版本：其字幕系统「为视频与音频两种模态提供丰富的专业级描述」，即在传统「视频-caption」数据之外系统性引入结构化的音频描述（structured audio descriptions），使模型内化更丰富的音视频联合语义空间。但报告未公开具体 schema——未说明是单条全音景描述（如 LTX-2 风格）还是拆分为独立字段流（如 Script-a-Video 的 factorized streams 或 Foley-Omni 的三字段）。从 SeedVideoBench 1.5/2.0 的音频标签体系（人声类型 / 人声属性 / 非语音音频三大主类，2.0 细化到 17 个类目）以及 Seedance 2.0 支持背景音乐、环境音效、角色人声三轨并行输出来看，训练侧极可能采用按音轨分流的多字段标注结构，但这属于推断而非论文明示。[不确定：schema 的确切形态与字段名]

### 对白转写与说话人属性标注（ASR转写+说话人身份/语言/口音/情绪） ⚠️

[不确定] 报告未明示 ASR 转写与说话人属性标注流程。间接证据强烈支持其存在：Seedance 1.5 pro 具备精准的多语种与方言唇形同步、能捕捉各语言独特的语音韵律与情绪张力；SeedVideoBench 1.5 的「人声属性」标签体系明确包含音色（timbre）、口音（accent）、情绪基调（emotional tone）三类；SeedVideoBench 2.0 覆盖中文方言/口音、多人对话、少数民族语言、画外音等 17 类。这些能力必须以对白转写 + 说话人身份/语言/口音/情绪的标注为数据基础，但论文未描述具体做法（是否用自研 ASR、说话人分离 diarization 等）。

### 几何与结构化标注（相机参数、深度、3D point tracks、动作标注、显式状态） ⚠️

[不确定] 未披露相机参数、深度、3D point tracks 等几何标注。已知的结构化标注仅停留在文本 caption 层面：Seedance 1.0 的 caption 显式包含相机运动（camera movements）类目作为动态特征之一；Seedance 1.5 pro 具备自主运镜调度能力（连续长镜头、dolly zoom/希区柯克变焦、电影级转场与专业调色）；Seedance 2.0 具备基本的导演与摄影推理能力，可自主规划镜头序列。这些提示运镜以语言标签而非数值相机参数的形式进入训练。

### 合成数据构造（受控扰动/编辑构造训练对，如InstructAV2AV） ⚠️

[不确定] 未披露合成数据构造流程。唯一相关的是 RLHF 环节：Seedance 1.0 明确将「模型不同训练阶段生成的合成视频（synthetic videos generated by different stages of our model）」纳入人类偏好标注的数据对来源，实验表明多来源视觉素材能提升 RM 的领域容量、扩展偏好上界并增强泛化。这属于偏好数据合成而非受控扰动式的编辑训练对构造。Seedance 2.0 具备视频编辑与续写能力（针对指定片段/角色/动作/情节的定向修改、前后向时间轴续写），推理其训练需要成对的编辑前后数据，但构造方式未披露。

### 人工介入程度（人工标注、人工质检、模型初筛+人工复核） ⚠️

人工介入贯穿多个环节且被明确提及：(1) Seedance 1.0 为训练 caption 模型采集多样类目数据并进行「高质量人工标注（high-quality manual annotations）」；(2) SFT 阶段使用「人工核验过 caption」的高质量视频-文本对（manually verified captions），并按数百个类别定向采集，构成人工策展数据集（Human-Curated Dataset）；(3) RLHF 阶段采用多维度人工标注协议——在特定维度下选出最好与最差视频，同时确保最好者在其他维度上不劣于最差者；(4) 数据处理基础设施的统一平台层显式支持「自动化的 human-in-the-loop 工作流（automating human-in-the-loop workflows）」、任务管理、数据可视化与流水线监控；(5) Seedance 2.0 在评测端引入广告与游戏制作领域的专家评审做主观打分与盲审，并做真实性辨别研究（评审区分生成视频与真实片段），结果反哺美学调优流程。整体模式为「模型初筛 + 人工复核/精标」。[不确定：人工标注规模与人力投入]

## 音视频对齐

### 音视频同步检测方法（口型同步、事件对齐） ⚠️

[不确定] 数据侧的音画同步检测方法未在报告中披露。Seedance 1.5 pro 仅在原则层面声明数据管线「优先保障视频-音频一致性（video-audio coherence）」，第三方摘要称第二阶段为「针对音画同步与片段质量的自动化过滤」，但该表述不见于论文原文，属未证实推断。评测侧则有完整定义：SeedVideoBench 1.5 的「音视频同步」指标衡量听觉流与视觉流的时间对齐，评估语音-唇动同步（缓解「腹语术」感知效应 ventriloquism effect）、音效与视觉事件的对齐、以及显著画面动作是否有对应听觉线索；SeedVideoBench 2.0 沿用并按 17 个细粒度类目打分（Seedance 2.0 的 T2V 音画同步总分 3.75，17 类中 16 类第一，可用率 93.30%、满意率 68.30%，远超竞品最高 25.45%）。

### 同步检测具体指标与阈值（SyncNet/Synchformer/LSE/自研，阈值数值，如MOVA LSE-D≤9.5且LSE-C≥4.5、SkyReels-V4 SyncNet |offset|≤3∧conf>1.5） ⚠️

[不确定] 完全未披露。报告中未出现 SyncNet、AV-align、LSE-C/LSE-D 等任何自动同步指标名称，也未给出任何过滤阈值数值。Seedance 系列的音画同步评估全部采用 1–5 分的人工/专家主观评分（SeedVideoBench），而非自动化客观指标——报告在 Arena 章节甚至专门指出其评估路线不依赖 FVD、CLIPScore 这类自动指标。因此无法与 UniTalking「SyncNet conf > 0.9」这类披露做对标。

### 时序同步vs语义同步的分离处理（时间对齐与内容语义匹配作为两个独立过滤条件） ⚠️

报告未在数据过滤层面明确拆分这两个条件，但评测体系已完成该拆分，可视为其数据质量观的映射：SeedVideoBench 1.5/2.0 将「音视频同步（Audio-Visual Sync）」定义为纯时间对齐维度（唇动-语音、音效-视觉事件的时序对齐），而将内容语义匹配另设为「音频指令跟随（Audio Prompt Following）」维度（评估人声、对白、音效是否忠实于用户指令与预期语义、有无语义漂移，典型失败模式含指定音效缺失、语言或方言不准确、以及「有语音但无对应唇动」的音画错配）；此外还独立设「音频质量（Audio Quality）」与「音频表现力（Audio Expressiveness）」两个维度。这种四维拆分强烈暗示数据侧亦按时序对齐与语义匹配两条独立条件把关。[不确定：数据过滤是否真按此拆分]

### 音频质量过滤（SNR、静音检测与静音占比阈值、无音轨剔除、画外音源剔除、背景音乐分离） ⚠️

[不确定] 未披露 SNR 阈值、静音占比阈值、无音轨样本剔除策略、画外音源剔除、背景音乐分离（source separation）等任何具体做法。仅有评测侧定义可参考：SeedVideoBench 1.5 的「音频质量」指标衡量输出的内在声学质量（含人声与非人声成分），关键判据包括是否存在削波（clipping）、截断（truncation）等伪影、空间声场渲染、音色真实感与整体信号清晰度——这些判据很可能同时被用作训练数据的音频质量准入标准。Seedance 2.0 报告在局限性中承认仍存在「偶发音频失真」。

### 语音/音效/音乐的分类与分别处理策略 ⚠️

训练侧的分类与分别处理策略未披露，但产品与评测两端均体现出清晰的三类划分：Seedance 2.0 支持背景音乐（BGM）、环境音效（ambient sound effects）、角色人声旁白（character narration/voiceover）三类音轨的多轨并行输出并各自与画面节奏精确对齐，这在架构上要求训练数据具备按音轨类型分离或分别标注的能力。评测标签体系亦按此划分：SeedVideoBench 1.5 将音频分为人声类型、人声属性、非语音音频（音效+音乐，再按声源、声学属性、音乐流派、技术参数细分）；SeedVideoBench 2.0 的 17 类中语音类（方言/多人对话/综艺/戏曲/英语/少数民族语言/歌唱说唱/画外音/非言语人声）、音效类（人声+动作交互音/物体交互音/动物叫声/环境背景音/特效 ASMR）、音乐类（乐器与音频）、空间类（空间场景音/双声道音频）界限分明。[不确定：训练数据侧的实际分类器与处理分支]

## 训练配合

### 多阶段训练课程与数据课程调度（阶段划分依据：分辨率/时长/质量分/模态；低清→高清、图像→视频、短→长） ⚠️

Seedance 1.5 pro 明确把「基于课程的数据调度（curriculum-based data scheduling）」列为数据管线的三大优先目标之一，并采用大规模混合模态数据集上的多任务预训练（覆盖 T2VA、I2VA、T2V、I2V），但未公开阶段划分依据与切换点。Seedance 2.0 报告未披露训练课程。可参照 Seedance 1.0 的渐进式训练（Progressive Training）课程：先用充分的 256px 低分辨率文生图训练初始化模型 → 阶段一以 256px 图像 + 3–12 秒（12 fps）视频片段做图文视频联合训练 → 阶段二分辨率提升至 640px、时长不变 → 阶段三提升到 24 fps 以改善视频流畅度；整体遵循低清→高清、图像→视频、低帧率→高帧率的递进。扩散调度上还采用「分辨率感知的时间步偏移（resolution-aware shift）」，对更高分辨率、更长时长的视频施加更强噪声扰动。整体训练分为 预训练 → 继续训练 CT → SFT → RLHF 四段。[不确定：1.5/2.0 的阶段数、各阶段依据（是否含按模态或按质量分分档）]

### 数据配比随训练阶段的变化（预训练/退火annealing/SFT高质量子集） ⚠️

Seedance 1.0 给出了少数可查的配比数字：视频预训练期间保留一小部分文生图任务以维持语义对齐，并将图生视频（I2V）任务比例设为 20% 以激活视觉提示跟随能力；Continue Training 阶段将 I2V 比例从 20% 提升到 40%，同时用美学打分器与光流运动评估器进一步精炼训练集（更高美学质量 + 更丰富运动动态），并使用略少于预训练的 GPU 数与退火（annealed）学习率调度；SFT 阶段切换到人工核验的高质量视频-文本对精选集。Seedance 1.5 pro 与 2.0 未披露任何阶段配比数字。[不确定：1.5/2.0 的各阶段配比、退火阶段设置]

### 后训练数据（SFT精选集规模与筛选标准、偏好对数量与标注方式、reward model训练数据） ⚠️

Seedance 1.5 pro：使用高质量音视频数据集做 SFT，随后采用专为音视频场景定制的 RLHF 算法，并使用多维度奖励模型（multi-dimensional reward model）提升 T2V/I2V 的运动质量、视觉美学与音频保真度；针对 RLHF 流水线的基础设施优化带来近 3 倍训练加速。规模与筛选标准均未给出。Seedance 1.0 披露更细：SFT 使用人工核验 caption 的高质量视频-文本对精选集，按视觉风格/运动类型等属性定义数百个类别做定向采集，并训练多个覆盖不同风格/运动/场景的子模型再做模型融合（model merging），使用比预训练更小的学习率、有限 GPU 数并配合早停以防过拟合与保持文本可控性；RLHF 侧从训练集与线上用户收集 prompt，做数据均衡与信息过滤以剔除重复与含糊 prompt，偏好数据涵盖模型不同阶段生成的合成视频等多来源素材，采用多维度标注协议（在指定维度选最优/最差，同时保证最优者在其他维度不劣于最差者）；奖励系统含 Foundational RM（VLM 架构，图文对齐与结构稳定性）、Motion RM（抑制伪影、增强运动幅度与生动性）、Aesthetic RM（图像空间输入，数据源改为视频关键帧）三个专用模型，并做多轮迭代式学习。[不确定：SFT 精选集规模、偏好对数量]

### 数据处理基础设施与吞吐（NeMo Curator/Data-Juicer/自研，GPU加速比，处理规模，成本） ⚠️

Seedance 1.5 pro 称其数据框架由「针对海量多模态数据处理优化的高效工程基础设施」支撑，但未给出细节。Seedance 1.0 给出完整三层架构：顶层为统一平台层（Unified Platform / Data Platform），负责自动化 human-in-the-loop 工作流、任务管理、数据可视化与流水线监控；中层为计算框架层，使用 BMF（Babit Multimedia Framework，字节自研）与 Ray（开源）实现跨 CPU/GPU/NPU 的异构计算，并对稳定算力与弹性算力分别做资源分配优化；底层为资源层，依托 ByteCloud（内部）与火山引擎 Volcengine（外部）云基础设施，存储为对象存储 + HDFS。数据任务流为 数据接入 → 多阶段数据筛选 → 数据编码与打包。异构计算上按算子分配最优硬件（CPU 解码、GPU 深度模型推理），计算单元间异步通信以缓解硬件性能差导致的瓶颈；针对弹性资源不稳定性引入自适应弹性伸缩与抢占任务失败重试，定制版 BMF 与 Ray 实现「近线性扩展与极高吞吐」。未使用 NeMo Curator / Data-Juicer 等外部方案。[不确定：GPU 加速比、处理规模、成本等量化指标均未披露]

## 效果对比

### 数据策略消融的量化影响（区分：过滤严格度ablation / caption密度风格ablation / 数据配比ablation，及对应评测指标） ⚠️

[不确定] 三份报告均未提供任何数据策略消融实验（无过滤严格度 ablation、无 caption 密度/风格 ablation、无数据配比 ablation 的量化对照）。仅有定性归因表述：Seedance 1.0 称 CT 阶段更丰富的运动动态与多样化 caption 使模型生成更自然流畅、更高美学质量的训练数据带来 T2V 视觉保真度的显著提升、SFT 的模型融合步骤显著改善视觉保真度与运动质量；RLHF 侧唯一的算法级对照是奖励最大化方法对比 DPO/PPO/GRPO 被称为最高效有效。跨版本的端到端提升可作为粗粒度替代证据：SeedVideoBench 2.0 上 Seedance 2.0 相对 Seedance 1.5 六维平均提升 0.86 分，运动质量提升最大（+1.36，2.39→3.75），音画同步 2.91→3.75，音频质量 2.88→3.63，音频指令跟随 2.69→3.56；T2V 可用率（≥3 分）运动质量 46.93%→97.55%、音画同步 69.64%→93.30%。但这些是模型整体迭代结果，无法归因到单一数据策略。

### 质量vs数量的证据（小而精数据超越大而杂的案例） ⚠️

无直接的「小而精超越大而杂」对照实验。间接体现为其整体方法论取向：Seedance 1.0 的漏斗以严格质量与安全过滤 + 语义去重（每个近重复簇仅保留质量分最高的单个实例）+ 分布再平衡为核心，CT 阶段主动收缩为更高美学、更丰富运动的子集，SFT 阶段进一步收敛到人工核验的数百类精选集，并强调早停以防过拟合、保持文本可控性——整条链路呈现逐级「以质换量」的设计。Seedance 1.5 pro 则把数据管线的优先目标定为一致性、运动表现力与课程调度而非规模。[不确定：缺乏量化证据]

### 训练数据domain分布与评测基准类目体系的对齐关系（如VABench七大类）

对齐关系明确但为单向推断（数据类目未公开，只能由评测类目反推）。SeedVideoBench 随模型迭代同步扩展：1.0 版覆盖运动动态、提示遵循、美学质量、主体一致性；1.5 版在视频维度给出主体、运动动态、交互、镜头运动等核心方面的细粒度案例分类与属性标签，并新增广告、社交媒体内容、短篇叙事等应用场景标签，同时因 1.5 支持音视频联合生成而新增完整音频维度（人声类型 / 人声属性 / 非语音音频三大主类），视频指标新增「视频生动性」（动作、运镜、氛围、情绪四维），音频指标含音频指令跟随、音频质量、音视频同步、音频表现力四项；2.0 版新增多模态任务评测体系（参考类：主体/运动/特效/风格参考；编辑类：主体/风格/场景/音频内容编辑；扩展类：情节续写与无缝前后向延展；组合类：如「参考图换视频主体」等贴近真实工作流的组合任务）与一致性指标（参考对齐 reference alignment、编辑一致性 editing consistency），并新增叙事质量三子维（摄影语言、情节设计、风格化美学），音频细化到 17 个类目，评测拆分为客观自动化流水线与主观专家盲审两条轨道。报告明确称「我们构建了覆盖主体、运动、场景、风格、音频的专用数据集，样本分布经过调优以在小评测预算下最小化方差」——这句话指的是评测数据集构建，但其类目体系与训练侧「分布再平衡」所用的属性维度（主体类别、场景类型、主导动作、体裁、视觉风格、时长、分辨率、运动特性）高度同构。

## 不确定字段

以下字段的调研信息部分不确定（⚠️ 标注来源）：

- data_scale
- data_sources
- provenance_licensing
- duration_distribution
- resolution_aspect_distribution
- domain_distribution
- audio_category_distribution
- narrative_structure_distribution
- language_accent_distribution
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
- av_generation
