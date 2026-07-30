# Script-a-Video（核心产出为 MTSS 表示范式，全称 Multi-Stream Scene Script，多流场景脚本）

> 主题: 视频生成模型（含音视频同时生成）的数据处理：数据清洗流程、数据分布、打标方式

[← 返回首页](../index.md)

**目录**: [基本信息](#基本信息) · [数据规模与分布](#数据规模与分布) · [清洗流程](#清洗流程) · [打标方式](#打标方式) · [音视频对齐](#音视频对齐) · [训练配合](#训练配合) · [效果对比](#效果对比)

## 基本信息

### 名称

Script-a-Video（核心产出为 MTSS 表示范式，全称 Multi-Stream Scene Script，多流场景脚本）

### 发布机构/公司 ⚠️

腾讯混元团队（Tencent Hunyuan Team）。论文署名为团队整体署名（作者栏仅写 Tencent Hunyuan Team），正文末尾设有 Project Contributors 章节但未在 arXiv HTML 版本中展开具体人名，故无法确认一作与通讯作者。[不确定]

### 发布时间（技术报告/论文/开源时间） ⚠️

2026年4月发布于 arXiv（arXiv:2604.11244）。当前可见最新版本为 v2，标注日期 2026年4月15日（cs.CV）。v1 提交时间在同月稍早（arXiv:2604.11244v1）。截至 2026年7月未见配套的 GitHub 仓库、项目主页或会议接收记录。[不确定]

### 类型（模型/数据集/工具链/评测基准）

方法/表示范式论文，兼具「打标 schema 定义 + 内部数据集构建 + 下游生成模型改造」三重属性：
【主体】提出 MTSS 结构化音视频 caption 表示范式（一套 JSON 式的四流 schema 定义），属于打标方法论而非模型或数据集。
【附带产出 1】一个 500K 视频片段的内部 MTSS 标注数据集（未开源）。
【附带产出 2】一个基于 Qwen3-Omni-Instruct 微调的专用 caption 模型 Qwen3-Omni-MTSS-FT（未开源）。
【附带产出 3】基于 LTX-2 改造的多镜头音视频联合生成模型（引入 Shot-Aware Structured Attention 与 Identity Customization 两处架构改进，未开源）。
【附带产出 4】一个内部评测集（125 条单镜头 + 100 条多镜头样本，未开源）。
不属于评测基准发布——评测使用的是 Video-SALMONN-2、UGC-VideoCap、Daily-Omni、WorldSense 等已有基准。

### 开源程度（权重/代码/数据/pipeline各自是否开源）

开源程度极低，属于「纯论文披露、零代码零权重零数据」类型：
【权重】未开源。Qwen3-Omni-MTSS-FT 与基于 LTX-2 改造的生成模型均未发布。
【代码】未开源。论文全文无 GitHub 链接、无项目主页 URL、无开源承诺声明。
【数据】未开源。500K MTSS 标注数据集明确表述为「internal dataset」（内部数据集）；生成侧的 400K identity-centric / 250K multi-shot / 870K cinematic pairs / 60K cinematic alignment pairs 四套数据同样为内部数据；125+100 条内部评测集也未发布。
【pipeline】方法论层面对 MTSS schema 的字段定义披露非常充分（Reference/Shot/Event/Global 四流的全部字段名与语义在正文第 3 节逐一说明，并配有 Figure 3 完整脚本示例），可复现性主要体现在「schema 可被他人重新实现」这一层；但数据清洗流程、Gemini-2.5-Pro 标注 prompt 原文、过滤阈值均未披露。
【许可】arXiv.org perpetual non-exclusive license（仅论文文本），无模型/代码许可。
【整体判断】其价值在于范式与 schema 设计本身可被直接借鉴复刻，而非在于可直接取用的资产。

### 是否支持音视频同时生成，以及实现方式（原生联合/级联/MoE融合）

支持音视频同时生成，但需区分两个层次：
【本工作的定位】Script-a-Video 本身不是一个新的音视频生成基座，MTSS 是「输入侧的结构化条件表示」。生成能力通过在已有基座上做条件替换与轻量架构改造来验证。
【生成基座】选用 LTX-2 作为生成框架，原因有二：(1) 其 Gemma 系 VLM 编码器天然擅长解析 MTSS 这类 JSON 式结构化语法，可从分流字段中抽取细粒度语义指令；(2) 其非对称双流 Diffusion Transformer（asymmetric dual-stream DiT）架构本身即为音视频联合合成设计，可把 MTSS 的 Shot 流与 Event 流分别映射到视频分支与音频分支的隐空间。
【实现方式归类】原生联合生成（单次生成过程同时产出同步的视频与音频），非级联、非 MoE 融合。
【两处架构改进】
1) Shot-Aware Structured Attention（镜头感知结构化注意力）：按 MTSS 的 shot 边界切分 Gemma-3 文本 embedding，再让每个镜头对应的视频 token 只与本镜头的语义片段做交叉注意力，实现镜头间上下文隔离，防止跨镜头语义串扰。
2) Identity Customization（身份定制）：通过 reference VAE 特征 + 可学习的 reference-learnable-tokens，把 Reference 流中的 ID 符号（如 "PERSON_1"）与对应参考图像显式对齐，作为视觉身份与语言指称之间的关系桥梁。
【多模态输入形态】多模态信息以图文交错（interleaved image-text）格式送入 Gemma-3，同时为视频与音频两个分支提供语义表示。
【任务形态】MTSS 三元组（S_ref, S_shot, S_eve）→ 同步的视频-音频对 (V, A)，目标是同时满足身份持久性与时序-听觉精确性。

### 调研信息来源列表（论文/技术报告/官方文档/新闻的URL，标注每条来源的性质：官方一手/同团队旁证/第三方报道）

- 【官方一手】arXiv 摘要页 https://arxiv.org/abs/2604.11244 —— 论文题目、机构（Tencent Hunyuan Team）、摘要核心数字（Video-SALMONN-2 总错误率平均降低 25%、Daily-Omni 平均提升 67%、多镜头生成人评身份一致性 +45% / 音视频对齐 +56% / 时序可控性 +71%）。
- 【官方一手】arXiv HTML 全文 v2 https://arxiv.org/html/2604.11244v2 （标注日期 2026-04-15，cs.CV）—— 本条目绝大多数字段的唯一直接来源：第 3 节 MTSS 四流 schema 完整字段定义、第 4.1 节 caption 数据集构建与 Table 1/Table 2、第 4.2 节生成实验与 Table 3/Table 4、第 4.2.2 节训练细节、Limitations 章节。
- 【官方一手】arXiv HTML 全文 v1 https://arxiv.org/html/2604.11244v1 —— 用于与 v2 交叉核对（v1 摘要称 Daily-Omni 提升 67%，v2 Figure 1 与正文分别给出 110% 与 127% 的口径，说明不同版本/不同统计口径下数字有差异）。
- 【官方一手】arXiv PDF https://arxiv.org/pdf/2604.11244 —— 图表版式与 Figure 3 的 MTSS 脚本示例。
- 【第三方旁证】腾讯混元 GitHub 组织 https://github.com/Tencent-Hunyuan —— 用于核实是否存在配套开源仓库；截至调研时该组织下有 HunyuanVideo、HunyuanVideo-1.5、HunyuanVideo-Avatar、HunyuanVideo-Foley、HunyuanVideo-I2V 等仓库，但无 Script-a-Video / MTSS 相关仓库，据此判定未开源。
- 【说明】未检索到任何第三方媒体报道、技术博客解读或社区复现讨论，该工作在调研时点尚属新发布且传播度有限。

## 数据规模与分布

### 训练数据量级（视频条数/小时数/token数，预训练与SFT分开）

论文披露的数据规模按用途分为「caption 侧」与「生成侧」两套，二者独立：
【caption 侧（用于训练 MTSS 标注模型）】500K 高质量视频片段（500,000 clips）。仅给出条数，未给出总小时数、平均时长、token 数。若按典型 5-10 秒片段粗估约 700-1400 小时，但论文无任何时长口径数据，此为推断而非披露。
【生成侧（用于训练 LTX-2 改造模型）】四套数据分阶段使用：
- ID Customization 阶段：400K identity-centric dataset（身份中心数据集），训练 3 epochs；
- Multi-shot Control 阶段：250K multi-shot sequences（多镜头序列），训练 1.5 epochs；
- Audio-Visual Synergy 阶段：870K cinematic pairs（电影级音视频对），训练 3 epochs；
- 最终联合微调阶段：60K high-fidelity cinematic alignment pairs（高保真电影级对齐对）+ 250K multi-shot sequences 交错混合，训练 15K steps。
生成侧数据合计规模约 1.5M 量级（含跨阶段复用），同样仅给条数不给小时数。
【评测侧】内部评测集 125 条单镜头样本 + 100 条多镜头样本，共 225 条。
【预训练 vs SFT 的拆分】caption 模型侧不存在预训练——直接在开源 Qwen3-Omni-Instruct 上做单阶段 SFT，500K 即全部 SFT 数据；生成侧同样是在已预训练的 LTX-2 上做多阶段微调，无从零预训练。

### 数据来源构成（自有/公开数据集/网络爬取/授权采购/合成数据）

披露极为简略，仅有领域级定性描述，无来源渠道构成比例：
【caption 侧 500K】明确表述为「internal dataset」（腾讯自有内部数据），领域覆盖三类：film（电影）、television（电视剧）、lifestyle（生活方式类内容）。未说明这些内容是自有版权库、采购授权、还是网络采集。
【生成侧】四套数据的来源同样未披露渠道，仅从命名可推断性质：identity-centric（以人物身份为中心，应为含明确人物主体的素材）、multi-shot sequences（多镜头序列，应来自影视剧等天然多镜头内容）、cinematic pairs（电影级音视频对）、cinematic alignment pairs（高保真电影级对齐对）。
【评测侧 225 条】覆盖 movie and TV drama clips（电影与电视剧片段）、short-form videos（短视频）、indoor scenes（室内场景）、outdoor scenes（室外场景）四类。
【公开数据集】未使用任何公开视频数据集训练（VGGSound、AudioSet、Panda-70M 等均未提及）；使用的公开资源仅限评测基准（Video-SALMONN-2 testset、UGC-VideoCap、Daily-Omni、WorldSense）。
【合成数据】未使用合成视频/音频；但 500K 数据集的「标注」全部由 Gemini-2.5-Pro 自动生成，属合成标注而非合成内容。
【采购授权】未提及。
整体来看，数据来源披露程度显著低于同类工作（如 UniVerse-1 逐项列举 YouTube 内容类型、MOVA 给出逐级构成），属于典型的大厂内部数据「只说领域不说渠道」披露风格。

### 数据合规与溯源（授权数据占比、rights-cleared数据集、C2PA等） ⚠️

论文完全未涉及数据合规与溯源议题：未给出授权数据占比，未声明 rights-cleared 数据集，未提及 C2PA 或任何输出侧水印/溯源标记，未讨论版权、肖像权或数据使用许可。
可识别的合规风险点：
1) caption 侧 500K 与生成侧数据均大量取自 film / television / TV drama 等影视内容，属版权高度敏感来源，论文未说明获取途径与授权状态；
2) Reference 流对人物做细粒度外观锚定（clothing、accessories、hairstyles 等属性），Identity Customization 模块支持从参考图注入真实人物身份，Event 流记录 verbatim 台词并绑定说话人 ID——这套组合在肖像权与深度伪造风险上比纯场景描述型 caption 更敏感，论文对此零讨论；
3) 所有资产均未公开发布，客观上降低了外部滥用风险，但也意味着无模型卡级的使用限制声明。
此字段在论文中属完全空白。[不确定]

### 片段时长分布与切分策略 ⚠️

论文未披露任何片段时长信息：未给出 500K 片段的平均时长、时长上下界、时长直方图，也未说明切分后是否做定长截断或分桶。
可间接推断的线索：
1) MTSS 的 Shot 流以 "time_range" 为每个镜头的锚定字段，Event 流同样带 "time_range"，并在 Shot 的 visual_description 内部嵌入 intra-description timestamps（描述内时间戳）以锚定微动作到全局时间轴——说明每个片段都有一条明确的全局时间轴，且时长足以容纳多个镜头与多个音频事件；
2) 生成侧区分单镜头（125 条评测）与多镜头（100 条评测）两类，多镜头样本必然长于单镜头；
3) 评测指标包含 Shot Boundary Deviation（镜头边界偏差，以帧为单位，最优 0.38 帧），说明生成侧的时长与帧率有确定配置，但具体数值未披露。
切分策略同样未披露——论文未说明 500K 片段是如何从长视频中切出的（是否用 PySceneDetect、是否按场景切、是否人工挑选）。这是本工作数据侧披露最薄弱的环节之一。[不确定]

### 分辨率/宽高比分布与分桶策略 ⚠️

论文完全未讨论分辨率与宽高比：未给出训练数据的分辨率门槛、分辨率分布、宽高比分布，未描述任何分桶（bucketing）或多分辨率训练策略，未说明生成模型的输出分辨率与帧率配置。
仅有的间接信息是生成基座 LTX-2 本身的分辨率能力，但论文未说明在其改造版本中实际使用的训练/推理分辨率档位。
评测指标中 Shot Boundary Deviation 以「帧」为单位报告（3.79 → 0.38 帧），说明存在固定帧率设定，但帧率数值未给出。[不确定]

### 类别/domain分布与配比策略（人物、动作、场景、风格等比例控制与概念均衡） ⚠️

domain 层面的披露停留在类目列举，无任何配比数字，也无概念均衡机制描述：
【caption 侧 500K 的领域构成】三类定性列举：film（电影）、television（电视）、lifestyle（生活方式）。三者各占多少比例未披露，也未说明是否做过配比调控。这一选择明显偏向「有叙事、有对白、有多镜头、有人物」的内容——与 MTSS schema 的设计目标高度耦合：Reference 流需要有反复出现的人物，Shot 流需要有真实的镜头切换，Event 流需要有对白与音效。换言之，数据领域是被 schema 需求反推选定的，而非追求通用覆盖。
【生成侧的隐含 domain 划分】四套数据实际上是按「能力维度」而非「题材维度」划分的：identity-centric（人物身份能力）、multi-shot（镜头结构能力）、cinematic AV pairs（音视频协同能力）、cinematic alignment pairs（高保真对齐能力）。这是一种「按目标能力组织数据」的思路，每套数据服务一个训练阶段的特定学习目标，而非按内容题材做均衡采样。
【评测侧的 domain 覆盖】225 条评测样本覆盖电影/电视剧片段、短视频、室内场景、室外场景四类，论文称「covers a diverse range of categories and scenarios」，但各类占比未给出。
【概念均衡】论文未描述任何概念均衡、长尾补采、类目配额或重采样机制。
【MTSS 内部的类目体系】值得注意的是，MTSS schema 自身内建了一套实体分类体系：Reference 流将实体划分为 person（人物）、object（物体）、animal（动物）、scene（场景）四类，且只保留与主线情节相关（integral to the main plot）的实体，边缘元素一律降级到 global scene description。这是一种「叙事重要性驱动的实体筛选」，构成了标注层面的隐式 domain 结构，但并非训练数据的采样配比。
整体而言，训练数据的题材配比数字属信息空白。[不确定]

### 音频类别分布与配比（语音/音效foley/音乐/环境音/静音的比例及控制策略）——AV模型独有维度 ⚠️

音频类别在 MTSS schema 层面有明确的三分类定义，但训练数据中各类别的比例完全未披露：
【schema 定义的三类音频事件】Event 流将所有音频事件严格分类为三种 type：
1) dialogue（对白/语音）——带 speaker（绑定 Reference ID）与 line（逐字台词文本）；
2) sfx（sound effect，音效）——要求必须由画面中可见的主体产生（"sound effects must be generated by a visible subject"）；
3) music（音乐）。
【第四类：全局音频】不构成独立事件的声音（环境底噪、背景音乐/氛围音）不进 Event 流，而是被归入 Global 流的 "global_audio" 字段。论文原文：「Irrelevant background noise is filtered into the global audio metadata」——即无关背景噪声被「过滤进」全局音频元数据，而非丢弃。这是一个四层分流设计：可见来源的音效 / 有说话人的对白 / 音乐 / 兜底的全局氛围音。
【筛选原则：严格音视频耦合】Event 流的准入条件是「strict audio-visual coupling principle」——只抽取具有直接视觉对应物或主题相关性的音频事件。这条原则实质上是在标注阶段就完成了「画外音源」的剔除与降级：无视觉对应的声音不会成为独立事件，从而保证 Event 流中的每一条都是可被生成模型学习的、有视觉线索的音频。这是本工作在音频侧最具方法论价值的设计。
【并发音源处理】同时发生的多个音源被「factorized into parallel event entries」（拆分为并行的事件条目），而非合并为一条混合描述。这保证了多声源场景下每条声音都有独立的 time_range、speaker 与 description，可被独立编辑与独立控制。
【比例数字】三类事件在 500K 数据集中各占多少、平均每个片段含多少条 event、dialogue 类事件占比多少，论文均未披露。[不确定]
【采样权重】未提及任何按音频类别调整训练采样权重的机制。

### 叙事结构分布（单镜头vs多镜头、平均clip时长、镜头数分布、是否含原生音轨） ⚠️

叙事结构是 MTSS 的一等公民，schema 层面处理充分，但分布统计缺失：
【单镜头 vs 多镜头】明确同时支持且显式建模。Shot 流本身即为「把视觉呈现分解为一串电影化片段（a sequence of cinematic segments）」，每个 shot 带独立 time_range。评测集按 125 条单镜头 / 100 条多镜头（约 5.6:4.4）划分，说明多镜头是一等评测场景而非附属。
【镜头数分布】500K 数据集中每个片段平均含多少个 shot、shot 数的分布区间，论文未披露。[不确定]
【平均 clip 时长】未披露。
【是否含原生音轨】必然含有——Event 流的构建依赖真实音轨中的对白与音效，Global 流的 global_audio 依赖真实环境音，整套 schema 若无原生音轨则大半字段为空。但论文未显式声明「无音轨视频被剔除」这类前置过滤规则。
【镜头语言的显式建模】Shot 流内设 camera 字段，专门记录专业电影语言：镜头运动（movements）、视角（perspectives）、景别（scales）。这使得镜头语言成为可被结构化检索与可控生成的字段，而非散落在自由文本中。
【叙事连续性的保证机制】跨镜头的叙事连贯由 Relational Grounding 保证：references_in_shot 数组把每个镜头中出现的主体映射到持久的 Reference ID，active_events 把镜头链接到并发的听觉事件。因此同一人物在多个镜头中不需重复描述外观，只需引用同一 ID——这直接解决了单体式 caption 中「重复描述导致的身份漂移」问题。
【实际生成效果的量化】Shot Boundary Deviation 从基线 LTX-2-AV 的 3.79 帧降至 0.38 帧（Ours w/o ID 配置），人评 MSC（多镜头一致性）从 1.00 提升至 2.49-2.62，证明 Shot 流提供的时序分段先验确实使多镜头可控性成为可能。

### 语言/口音分布（多语种唇同步能力的数据基础） ⚠️

论文未给出任何语言/口音分布数据：未列举支持语种、未给出各语种占比、未做口音标注、未讨论多语种唇同步。
可间接推断的线索：
1) 评测环节计算 WER 时明确使用「jieba-based tokenization for CJK text」（对中日韩文本采用 jieba 分词），说明生成与评测中确实存在中文（或更广义的 CJK）语音内容，训练数据非纯英文；
2) 数据来源以 film / television / TV drama 为主且为腾讯内部数据，中文影视内容占主导的可能性较高，但论文无任何佐证；
3) MTSS 的 Event 流为 dialogue 事件设有 "line" 字段记录逐字台词（verbatim text），schema 层面对语种无限制，但未设语种或口音字段；
4) Event 流的 "description" 字段用于捕捉细腻语义如情绪起伏（emotional shifts）与发声技巧（vocal techniques），这是最接近「说话人属性」的设计，但仍是自由文本而非结构化的口音/语种标签。
语种与口音属完全信息空白。[不确定]

## 清洗流程

### 清洗漏斗整体结构（几级过滤、各级顺序） ⚠️

需明确区分：本工作的「pipeline」重心在标注 schema 的构建逻辑，而非数据清洗漏斗。清洗流程几乎无披露。
【数据清洗漏斗】论文仅一句话带过：「we curated an internal dataset comprising 500K high-quality video clips from film, television, and lifestyle domains」（我们策划了一个包含 50 万条高质量视频片段的内部数据集，来自电影、电视和生活方式领域）。「high-quality」的判定标准、过滤级数、各级顺序、工具选型均未披露。这是本工作最显著的披露缺口。[不确定]
【标注 pipeline（有充分披露）】实际的处理流程为两步：
第 1 步：用 Gemini-2.5-Pro 对 500K 片段逐条生成 MTSS 格式标注（"Each clip was annotated using Gemini-2.5-Pro"）；
第 2 步：用这批数据对开源 Qwen3-Omni-Instruct 做监督微调（SFT），得到专用标注模型 Qwen3-Omni-MTSS-FT。
即典型的「强教师模型蒸馏到开源学生模型」范式，无中间的质检、复核、重采样环节披露。
【MTSS 标注内部的处理逻辑（本工作真正的方法核心）】可视为一个五步的「信息重组流水线」：
1) 实体抽取与筛选 → 按叙事重要性把实体分为 person/object/animal/scene 四类进入 Reference 流，边缘实体降级到 global scene description；
2) 视觉分段 → 按镜头切分为 Shot 流，每段带 time_range、visual_description、camera 三层信息；
3) 音频事件抽取与分流 → 按「严格音视频耦合」原则筛出 dialogue/sfx/music 三类事件进 Event 流，无视觉对应的背景噪声降级到 global_audio，并发音源拆为并行条目；
4) 全局兜底 → scene_description / global_style / global_audio 三字段承载不属于任何具体流的宏观信息；
5) 关系重连（Relational Grounding）→ 身份链接（references_in_shot 数组指向 Reference ID、Event 的 speaker 指向 Reference ID）+ 时间链接（active_events 把 shot 关联到并发事件、intra-description timestamps 把微动作锚定到全局时间轴）。
这套「先解耦、再重连」的结构是全文的方法论主张：解耦解决冗余与可编辑性，重连解决一致性。
【生成侧 pipeline】见 multi_stage_curriculum 字段（四阶段渐进训练）。

### 漏斗定量保留率（每级过滤的输入/输出量与最终保留率，如Apollo 27%） ⚠️

论文完全未给出任何漏斗定量数据：未披露原始采集视频总量（分子已知为 500K，分母完全缺失），未给出逐级过滤的输入/输出量，未给出各级或整体保留率，也未给出生成侧四套数据（400K/250K/870K/60K）各自的原始池规模与保留率。
无法计算类似 Apollo 27% 或 MOVA 26.39% 的整体保留率。
值得注意的一个「保留率」类比：MTSS 在标注层面存在一个隐性的信息保留决策——Event 流的严格音视频耦合原则会把大量音频信息「过滤」到 global_audio，Reference 流的叙事重要性筛选会把边缘实体「过滤」到 global scene description。但论文未量化这两个降级通道各自吸纳了多少比例的信息。[不确定]

### 镜头切分方法（PySceneDetect/自研模型/shot-aware splitting） ⚠️

本工作的镜头切分呈现一个关键的双重角色，需区分两个层面：
【数据预处理层面（未披露）】500K 片段是如何从长视频中切出的、是否用 PySceneDetect 或自研检测器、阈值如何设定，论文完全未说明。这是清洗侧的空白。[不确定]
【标注层面（充分披露且为核心创新）】MTSS 不把镜头切分当作预处理丢弃的中间产物，而是把镜头结构作为一等标注字段永久保留在 Shot 流中——每个 shot 带精确的 "time_range"，并携带 visual_description（客观按时序叙述核心动作）与 camera（专业电影语言：运动/视角/景别）两层描述，外加 references_in_shot 与 active_events 两个关系字段。因此单个训练样本内部可以包含多个镜头，而非被切碎成多个独立单镜头样本。这与 UniVerse-1 等「切分后只保留单镜头片段」的主流做法方向相反。
【下游收益】保留镜头结构使得生成模型可直接学习「在指定时间戳执行镜头转场」这一能力。论文引入的 Shot-Aware Structured Attention 正是利用 shot 边界把 Gemma-3 embedding 分区，让每段视频 token 只关注本镜头的语义，实现镜头间上下文隔离。
【切分精度的量化验证】Shot Boundary Deviation 指标（生成视频的镜头边界与脚本指定边界的帧级绝对偏差）：LTX-2-AV 基线 3.79 帧 → LTX-2-AV-MTSS（仅换 MTSS 提示词，不改架构）3.27 帧 → Ours w/o ID 0.38 帧 → Ours(Full) 1.36 帧。论文指出 Full 配置反而回升到 1.36 帧，是身份注入特征与时序镜头精度之间的潜在权衡（trade-off），归因于 VLM 编码器与 DiT 接口的设计，留作未来架构优化。

### 质量过滤（美学评分、清晰度、OCR文字过滤、黑边/水印/logo检测） ⚠️

论文未披露任何质量过滤细节：无美学评分器（LAION-Aesthetic / DOVER / VQA 类均未提及）、无清晰度或模糊度指标、无 OCR 烧录字幕检测、无黑边检测、无水印/logo 检测、无码率或压缩伪影检测，也未给出任何阈值数值。全部质量控制被压缩为「high-quality video clips」一个形容词。[不确定]
仅有的两条与「质量」相关的间接线索：
1) 数据领域选择本身构成隐式质量筛选——film / television / cinematic pairs 等均为专业制作内容，天然在画质、构图、收音上高于 UGC；
2) 生成侧最终阶段使用的是「60K high-fidelity cinematic alignment pairs」（高保真电影级对齐对），"high-fidelity" 的措辞暗示存在一个更严格的高质量子集筛选，但筛选标准未披露。
本字段的空白与该工作的定位有关：论文的贡献主张在标注范式而非清洗流程，因此把清洗部分整体略去。对于关注打标方式的调研而言这是可接受的取舍，但对关注清洗漏斗的调研而言本条目参考价值极低。

### 运动过滤（光流/运动分数阈值、静态与抖动剔除） ⚠️

论文未描述任何运动过滤环节：无光流计算、无运动分数阈值、无静态镜头剔除、无抖动/晃动检测。[不确定]
运动信息在本工作中不作为「过滤维度」而作为「标注维度」出现：
1) Shot 流的 camera 字段显式记录镜头运动（movements），使运镜成为可描述、可控制的语义字段；
2) Shot 流的 visual_description 要求对核心动作做客观的时序叙述（objective, chronological narrative of core actions），并通过 intra-description timestamps 把微动作锚定到全局时间轴——即运动被「结构化描述」而非「分数化过滤」。
评测侧存在一个与运动相关的观察：Intra-Shot Subject Consistency（镜头内主体一致性，DINOv2 [CLS] 特征的帧间平均余弦相似度）基线 LTX-2-AV 高达 0.87，但论文明确指出这是假象——该高分源于模型未能注入参考特征、生成了近乎静止的内容，帧间变化极小所致（"trivially static content with minimal frame variation"）。本工作的完整方案维持 0.59-0.62 的「较低」分数，反而对应真实的动态内容。这是一个关于「静态内容会虚高一致性指标」的重要警示，对数据侧的运动过滤设计有参考意义。

### 去重方法（精确去重与基于embedding的语义去重分别记录） ⚠️

论文完全未提及去重环节：既无哈希/指纹级精确去重，也无基于 embedding 的语义去重，未讨论跨数据源或跨阶段的重复问题。[不确定]
潜在重复风险客观存在：生成侧四套数据（400K identity-centric、250K multi-shot、870K cinematic pairs、60K cinematic alignment pairs）来源同为内部影视素材库，且最终联合微调阶段明确复用了前述的 250K multi-shot sequences，说明跨阶段数据复用是有意为之；但不同阶段数据集之间是否存在同源片段的重叠，论文无任何说明。
值得一提的是，MTSS 的设计在「描述层面」内建了一种去冗余机制——Reference 流的中心化实体库使得反复出现的人物/场景只需描述一次，后续 Shot 与 Event 只引用 ID，论文称之为消除「redundant re-description」。但这是文本冗余的消除，与数据集样本去重是两个不同层面的问题，不应混淆。

### VLM/LLM作为质检员（多模态大模型质量打分与错配剔除，2026年从浅层打分器转向大模型语义判断的趋势） ⚠️

本工作是「大模型作为标注器」的典型案例，但严格来说它并未把大模型用作质检员（judge of data quality），而是用作生成器（generator）与评测裁判（judge of output）两种角色：
【角色 1：Gemini-2.5-Pro 作为标注生成器】500K 片段的 MTSS 标注全部由 Gemini-2.5-Pro 自动生成，无其他模型参与、无多模型投票、无一致性交叉校验。选用 Gemini-2.5-Pro 的合理性由 Table 1 佐证：它在 UGC-VideoCap 上综合分 93.97（MTSS 版本 95.51），Video-SALMONN-2 总错误率 0.3959（MTSS 版本 0.2511），是全表最强模型，作为教师模型质量有保障。
【角色 2：Gemini-2.5-Pro 作为评测裁判】Table 1 与 Table 2 的所有 caption 与推理基准评测均以 Gemini-2.5-Pro 作为 judge model。此处存在一个方法论上的自指风险：训练数据的标注者与评测的裁判者是同一个模型，可能对 MTSS 格式存在系统性偏好，论文未讨论或缓解这一潜在偏差。
【角色 3：Gemma4-31B-it 作为生成质量裁判】生成侧的 Semantic Following 指标由 Gemma4-31B-it 视觉语言模型独立打分，从 Subject（身份与外观保真）、Action（动作与交互正确性）、Scene（背景/环境/空间布局准确性）、Style（视觉风格/配色/氛围的遵循度）四个子维度评分后取算术平均。这是一个 VLM-as-judge 的标准用法。
【缺失的环节：标注质检】论文未描述任何对 Gemini-2.5-Pro 标注结果的质量把关机制——无幻觉检测、无格式合法性校验（MTSS 是 JSON 式结构，理论上需要解析校验）、无 Reference ID 引用完整性校验（Shot 的 references_in_shot 与 Event 的 speaker 是否都指向存在的 ID）、无时间戳合法性校验（time_range 是否越界/重叠）、无人工抽检。对于一个强依赖交叉引用完整性的结构化 schema 而言，这是一个较明显的方法论缺口。[不确定]
【论文自陈的相关局限】Limitations 章节承认：生成准确的深度结构化脚本对基础模型的跨模态理解能力要求极高；当前开源 MLLM 在精确时序定位、鲁棒 ASR、准确的视听实体-事件关联三方面仍有局限；如何让更紧凑的开源架构达到 Gemini 级脚本能力并有效抑制幻觉，仍是未解难题。这一表述反向印证了标注质量高度依赖教师模型本身，而非依赖后置质检。

### 安全与合规过滤（NSFW、版权、人脸/隐私） ⚠️

论文完全未涉及安全与合规过滤：无 NSFW 检测、无暴力/敏感内容过滤、无版权过滤、无人脸隐私保护措施、无模型卡级安全声明或使用限制。[不确定]
风险落差值得注意：本工作的 Identity Customization 模块支持从参考图像注入真实人物身份并在多镜头中保持一致，Event 流记录逐字台词并绑定说话人，Reference 流对人物做细粒度外观锚定（服装、配饰、发型）——这套能力组合恰好构成高质量深度伪造的技术前提，而论文对滥用风险零讨论。缓解因素是所有模型权重与数据均未公开发布。

## 打标方式

### 使用的caption模型（自研VLM/开源模型，模型规模） ⚠️

标注侧涉及两类模型，角色分工明确：
【教师模型 / 数据标注模型】Gemini-2.5-Pro（Google 闭源）。承担 500K 片段的全部 MTSS 标注生成，是整个数据集的唯一标注来源。参数规模未公开（闭源模型）。
【学生模型 / 专用标注模型】Qwen3-Omni-Instruct（阿里通义千问全模态开源模型）。在 500K MTSS 数据上做监督微调（论文原文为 "supervised sine-tuning"，应为 supervised fine-tuning 笔误），得到 Qwen3-Omni-MTSS-FT。论文未披露使用的具体参数档位（Qwen3-Omni 有多个规模变体）。[不确定]
【蒸馏效果的量化】学生模型经 MTSS 数据微调后大幅逼近教师：
- Video-SALMONN-2 总错误率：Qwen3-Omni 原生 0.5853 → 零样本 MTSS 提示 0.5156 → MTSS 微调后 0.3913（教师 Gemini-2.5-Pro 原生为 0.3959，即微调后的 8B/30B 级开源模型在该指标上已略优于 Gemini-2.5-Pro 的原生 caption）；
- UGC-VideoCap 综合分：62.80 → 71.54 → 85.11（教师 93.97）；
- Daily-Omni：0.1806 → 0.4117 → 0.5945（教师 0.6825）；
- WorldSense：0.1569 → 0.3106 → 0.3875（教师 0.4332）。
【对比基线（均为 caption 模型）】AVoCaDO、ASID-Captioner-7B（7B 参数）、Qwen3.5-Omni-Flash（闭源）、Gemini-2.5-Pro（闭源）。
【关键发现：MTSS 是零成本增益】论文强调 MTSS 对所有测试模型都有效，包括未经任何微调的零样本提示场景——即仅把输出格式要求从「写一段话」改为「填 MTSS 四流结构」，Gemini-2.5-Pro 的总错误率就从 0.3959 降到 0.2511、Qwen3.5-Omni-Flash 从 0.5217 降到 0.3655。这说明结构化 schema 本身即为一种有效的提示工程手段，可被任何团队零成本复用。
【生成侧的文本编码器】LTX-2 内置的 Gemma-3 VLM 编码器，负责解析 MTSS 的 JSON 式语法并输出送入视频/音频双分支的语义表示；多模态输入以图文交错格式喂入。

### caption密度与结构化程度（短/长/密集描述、结构化字段如镜头运动、风格标签）

MTSS 是本次调研中结构化程度最高的 caption schema 之一，属于「深度结构化 + 显式关系图」类型，与主流的自由文本长描述形成范式级差异：
【总体形态】JSON 式的四流结构，而非自然语言段落。四流为 Reference（参考流）、Shot（镜头流）、Event（事件流）、Global（全局流）。
【流 1：Reference Stream（实体库，回答 WHO 与 WHERE）】
- 定位：持久实体银行（Entity Bank），为整个脚本提供身份锚点；
- 实体分类：person / object / animal / scene 四类；
- 筛选原则：只收录与主线情节相关（integral to the main plot）的实体，边缘元素降级到 Global 流的场景描述；
- 字段：semantic_description（实体整体状态描述）、timestamp（出现时间）、appearance_anchor（外观锚点）；
- appearance_anchor 内部：通用的 detail_description 适用于所有实体类型；针对 person 类别额外扩展细粒度属性——服装（clothing）、配饰（accessories）、发型（hairstyles）；
- 设计收益：后续所有流引用持久 Reference ID 而非重复描述，从根本上保证跨镜头身份绝对一致，同时消除文本冗余。
【流 2：Shot Stream（视觉分段，回答 WHAT-visual 与 HOW）】
- 每个 shot 由精确的 time_range 锚定；
- 视觉空间层：visual_description（对核心动作的客观、按时序的叙述）+ camera 字段（专业电影语言：镜头运动 movements、视角 perspectives、景别 scales）；
- 关系层：references_in_shot 数组（把画面中可见主体映射到 Reference ID）+ active_events（链接到本镜头内并发的听觉事件）；
- 时间层：在描述文本内部嵌入 intra-description timestamps（描述内时间戳），把微动作锚定到全局时间轴，论文称之为实现「surgical synchronization」（外科手术级同步）。
【流 3：Event Stream（音频事件，回答 WHAT-audio）】
- 准入原则：strict audio-visual coupling（严格音视频耦合）——只收录有直接视觉对应物或主题相关性的音频事件，音效必须由画面中可见主体产生；
- 事件类型：dialogue / sfx / music 三类；
- 字段：type、time_range、以及内容块含 speaker（关系绑定到 Reference ID）、line（逐字台词文本）、description（捕捉情绪起伏、发声技巧等细腻语义）；
- 并发处理：同时发生的多个音源拆分为并行事件条目而非合并；
- 降级通道：无关背景噪声过滤进 Global 流的 global_audio；
- 对齐：通过 micro-level timestamps（微观时间戳）与视觉轨精确对齐。
【流 4：Global Stream（宏观语境）】
- scene_description（视频事件的整体描述）、global_style（整体美学风格或类型/genre）、global_audio（不构成独立事件的环境音与背景音乐）。
【两大设计原则】
1) Stream Factorization（流分解）：把持久信息与时变信息分离，降低语义冗余，支持局部更新；
2) Relational Grounding（关系接地）：身份链接（中心化实体库 + ID 引用）+ 时间链接（共享锚点 + 描述内时间戳）把孤立的流重新编织为连贯脚本。
【核心主张：可编辑性】单体式 caption 的致命缺陷是局部修改触发全局重写（"local edits inevitably trigger global rewrites"）——想改一个运镜或一个音效，就得重写整段以维持叙事连贯。MTSS 使依赖关系可追溯，支持精确的局部更新，这是其相对于长密集 caption 的结构性优势，也是「scalability」论证的落点。
【核心主张：可学习性】MTSS 显著缩小了小模型与大模型之间的性能差距（Figure 1 明确以此为第二大卖点），论文解释为：单体式文本要求模型自行解开密集交织的关系，而 MTSS 已经把 WHO、WHERE、WHEN 预先消歧，下游推理模块可专注逻辑推断而非身份解析与时序去歧义。

### 音视频联合caption结构（是否同时覆盖视觉+听觉轨道、是否分流为独立字段，如LTX-2全音景描述、Script-a-Video factorized streams、Foley-Omni三字段）

本字段是 Script-a-Video 最具价值的贡献所在，其方案在同类工作中处于结构化光谱的最深端：
【定位对比】沿「视听信息如何组织」这条轴排列：LTX-2 的全音景单一描述（融合式）→ Foley-Omni 的三字段并列（浅分流）→ UniVerse-1 的三路独立字段（分流但无关系）→ MOVA 的分流后 LLM 融合为单段落（分流再融合）→ MTSS 的四流分解 + 显式关系图（分流再显式重连）。MTSS 是唯一在分流之后用「可机读的引用关系」而非「自然语言叙述」把各流重新连接起来的方案。
【是否同时覆盖视觉与听觉】是。Shot 流承载视觉轨，Event 流承载听觉轨（对白/音效/音乐三类），Global 流的 global_audio 承载环境底噪与背景音乐，Reference 流跨模态共用（既被 Shot 的 references_in_shot 引用，也被 Event 的 speaker 引用）。
【是否分流为独立字段】是，且是四流而非常见的二流或三流。关键差异在于多出了一条 Reference 流——这条流不属于任何单一模态，而是作为跨模态的共享身份锚点存在，是 MTSS 区别于其他 factorized 方案的结构性创新。
【跨模态连接的两种机制】
1) 身份链接：Event 的 speaker 字段与 Shot 的 references_in_shot 数组同时指向 Reference 流中的同一 ID。这意味着「谁在说话」这件事在数据结构上被显式表达为「音频事件的 speaker 指针」与「视觉镜头的可见主体指针」指向同一实体——说话人与画面人物的绑定是结构性的、可机读校验的，而非依赖自然语言描述的隐式对应。这是解决音视频身份错配的最直接手段。
2) 时间链接：Shot 的 active_events 字段把镜头链接到并发的音频事件；两侧共用同一条全局时间轴，Shot 与 Event 各自的 time_range 可直接做区间运算；更细的对应由双方的 intra-description / micro-level timestamps 承担。
【音频事件的准入过滤即为对齐保证】严格音视频耦合原则（音效必须由可见主体产生）实质上把「音画不匹配」的样本在标注阶段就化解掉了——不匹配的声音不会成为 Event，而是降级为 global_audio。这是一个用 schema 设计替代过滤器的巧思：不需要额外的音画一致性判别模型，标注规则本身就完成了分流。
【下游生成的直接收益（量化）】把单体式提示词换成 MTSS 脚本、架构完全不变（LTX-2-AV → LTX-2-AV-MTSS）时：人评身份一致性 1.22 → 1.77（+45%）、人评音视频对齐 1.18 → 1.85（+56%）、人评多镜头可控性 1.00 → 1.71（+71%）；WER 从 0.84 降至 0.13；Shot Boundary Deviation 3.79 → 3.27 帧。这组「同架构同训练范式、唯一变量是提示词结构」的对照，是本工作最有说服力的证据——论文明确指出两个基线共享完全相同的架构与训练范式，因此二者之间的任何性能差距可直接且完全归因于 MTSS 范式本身。
【局限】schema 未设置显式的音频质量/响度/声学环境字段（如录音环境、混响、音量），也无语种/口音结构化标签，音色描述仅隐含在 Event 的 description 自由文本中。

### 对白转写与说话人属性标注（ASR转写+说话人身份/语言/口音/情绪） ⚠️

对白转写在 MTSS 中有明确的结构化承载，但说话人属性标注偏薄：
【转写】Event 流中 type 为 dialogue 的事件设有 "line" 字段，记录 verbatim text（逐字台词文本）。论文未说明这些台词是由 ASR 转写还是由 Gemini-2.5-Pro 直接从音频中听写生成——考虑到整套标注均由 Gemini-2.5-Pro 一次性产出，后者可能性更高，即无独立的 Whisper 类 ASR 环节。[不确定]
【说话人身份】通过 "speaker" 字段实现，值为 Reference 流中的实体 ID（如 PERSON_1）。这是一种关系绑定而非属性描述——说话人身份直接指向实体库中带完整外观锚点（服装/配饰/发型）的条目，因此「声音」与「长相」在数据结构层面被硬绑定。这一设计对训练音视频联合生成模型辨识「谁在说话」尤为关键。
【说话人属性】未设独立的结构化字段。性别、年龄、音色、语种、口音、语速均无对应字段。最接近的是 Event 的 "description" 字段，论文描述其用于捕捉「nuanced semantics like emotional shifts or vocal techniques」（细腻语义如情绪起伏或发声技巧）——即情绪与发声方式以自由文本形式记录，但非枚举标签，不可直接用于配比统计或条件控制。
【多说话人处理】通过并发事件拆分机制天然支持：同时发生的多个音源被拆为并行的 event 条目，各自带独立的 speaker、time_range 与 line，因此多人对话场景可被完整表达而不会混作一团。这是相对于「一段话描述整个音景」类方案的明确优势。
【生成侧的转写准确性验证】WER 用 Whisper-large-v3 转写生成音频后、用 jiwer 对照真值文本计算（CJK 文本用 jieba 分词）。结果：LTX-2-AV 基线单镜头 1.64 / 多镜头 0.84 → LTX-2-AV-MTSS 单镜头 0.78 / 多镜头 0.13 → Ours(Full) 多镜头 0.19。论文指出 Event 流提供的显式音频事件描述把语音与音效生成从「近乎随机」（near-random）转变为「语义准确」。
【音质指标】Audio Quality 用 UTMOS（轻量语音质量 MOS 预测器）评估：基线 4.12-4.18，MTSS 版本 4.60，完整方案 4.68。

### 几何与结构化标注（相机参数、深度、3D point tracks、动作标注、显式状态）

MTSS 不包含任何数值型几何标注：无相机内外参、无深度图、无 3D point tracks、无光流场、无骨架/姿态关键点、无边界框坐标、无分割掩码。
但本工作在「非数值型结构化标注」维度做得相当深，是本条目的实质内容：
【镜头语言的语义化标注】Shot 流的 camera 字段用专业电影语言记录镜头运动（movements）、视角（perspectives）、景别（scales）。这是把相机信息以「导演语言」而非「参数矩阵」的形式结构化——牺牲了精确性，换取了可被 LLM 直接理解与生成的可控性。对于以文本为条件接口的生成模型而言，这是比数值相机参数更实用的形式。
【实体级的结构化外观锚定】Reference 流的 appearance_anchor 对 person 类实体展开为服装、配饰、发型等细粒度属性槽位。这是一种「以属性槽位替代像素级标注」的身份表示。
【时间轴的结构化】这是 MTSS 最硬核的结构化维度——三层时间标注：shot 级 time_range、event 级 time_range、以及嵌入描述文本内部的 intra-description / micro-level timestamps。三层共用同一条全局时间轴，构成一个可做区间运算的显式时序结构。论文称其目标是达成 sub-frame（亚帧级）的视听协调。
【关系图结构】references_in_shot（镜头→实体的多对多边）、active_events（镜头→事件的边）、speaker（事件→实体的边）三类引用共同构成一张显式的实体-镜头-事件三部图。这是整个 schema 中最接近「知识图谱」的部分，也是「Relational Grounding」一词的实质所指。
【动作标注】以自由文本形式存在于 visual_description 中，要求「客观、按时序」叙述核心动作，并由 intra-description timestamps 锚定，但无动作类别标签体系。
【显式状态】Reference 流的 semantic_description 记录实体的整体状态，属于弱化版的显式状态标注。

### 合成数据构造（受控扰动/编辑构造训练对，如InstructAV2AV）

论文未构造任何合成数据：无受控扰动构造训练对，无编辑式数据增广（不存在 InstructAV2AV 式的音视频编辑指令对），无 TTS 合成语音，无音效叠加混音，无生成模型自产数据的自蒸馏。全部训练数据均为真实采集的影视/生活类音视频片段。
【唯一的「合成」成分是标注而非内容】500K 数据集的全部 MTSS 标注由 Gemini-2.5-Pro 自动生成，属合成标注（synthetic annotation）。这构成一条标准的模型蒸馏链路：闭源强模型标注 → 开源模型 SFT → 得到成本更低的专用标注器。
【最接近数据构造的设计】MTSS 本身可视为一种「表示层的数据重构」——同一段视频在单体式 caption 与 MTSS 脚本两种表示下构成一组天然的对照数据对，论文正是用这组对照（LTX-2-AV vs LTX-2-AV-MTSS，同架构同训练、唯一变量是提示词结构）完成核心论证。但这是评测设计而非数据合成。
【可编辑性的潜在合成价值（论文未实践）】MTSS 的局部可编辑特性理论上非常适合构造受控扰动对——只改 Event 流中一条音效的 description、或只改某个 shot 的 camera 字段，即可得到一组语义差异受控的训练对，而无需重写全文。论文提出了这一结构优势但未将其用于合成数据构造，属于该 schema 尚未被挖掘的应用方向。

### 人工介入程度（人工标注、人工质检、模型初筛+人工复核） ⚠️

人工介入程度极低，且分布不均：
【标注环节：零人工】500K 片段的 MTSS 标注全部由 Gemini-2.5-Pro 自动生成，论文未提及任何人工标注、人工抽检、人工复核或标注规范培训环节，也未报告标注一致性指标。对于一个依赖交叉引用完整性的结构化 schema 而言，缺少人工或程序化的校验环节是明显缺口。[不确定]
【评测环节：有相当规模的人工投入】生成侧组织了 20 名专业评分员（20 professional raters）对 225 条生成视频（125 单镜头 + 100 多镜头）做人工评分，覆盖五个维度：Text Alignment（文本对齐）、Visual Quality（视觉质量）、Multi-Shot-Consistency（多镜头一致性）、Identity Consistency（身份一致性）、Audio-Video Synchronization（音视频同步），均为 1-3 分制。
【人工评测的方法论价值】论文明确指出人工评测揭示了自动指标的欺骗性，这是本工作一个有价值的观察：
1) Intra-Shot SC 指标上基线 0.87 高于完整方案 0.59，但这是基线生成近乎静止内容的假象；
2) A-V Sync（SyncNet）指标上基线 6.86「领先」于完整方案 9.72，但基线的人评 A-V 分仅 1.18——论文解释为「信息稀疏性伪影」：让平坦的环境噪声与画面同步是平凡的（trivially easy），而生成复杂对白后 SyncNet 分数会先升高（LTX-2-AV-MTSS 达 13.86，偏离越大越差），完整方案再把它拉回 9.72 同时人评达 2.26。这一分析对任何依赖 SyncNet 做数据过滤的团队都有直接警示价值：SyncNet 分数对无语音/平坦音频样本存在系统性偏置。
3) Reference ID Similarity 在多镜头场景降至约 0.22（因剧烈视角与光照变化），但人评 Cons. 稳定在 2.40 以上——自动 ID 相似度在跨镜头场景下同样不可靠。
【标注 prompt】未公开，也未提及是否有人工参与 prompt 迭代。

## 音视频对齐

### 音视频同步检测方法（口型同步、事件对齐） ⚠️

需区分数据侧与评测侧，二者差异显著：
【数据侧（标注阶段）：以 schema 规则替代检测模型】这是本工作最值得关注的思路。MTSS 没有使用 SyncNet 等同步检测模型来过滤数据，而是把音视频对应关系直接编码进标注规则：
1) 严格音视频耦合准入原则——Event 流只收录「有直接视觉对应物或主题相关性」的音频事件，音效必须由画面中可见的主体产生。不满足条件的声音不被剔除而是降级到 global_audio。这在标注层面就完成了「有视觉依据的声音」与「无视觉依据的声音」的分离。
2) 结构性说话人绑定——Event 的 speaker 字段指向 Reference ID，与 Shot 的 references_in_shot 指向同一实体库，使「说话人是否在画面中」成为可通过引用关系直接判定的问题，无需人脸检测 + 唇动分析。
3) 三层时间戳体系——shot 级 time_range、event 级 time_range、描述内 micro-level timestamps 共享全局时间轴，事件对齐由时间戳直接表达，论文称目标是达成 sub-frame（亚帧级）的视听协调与「surgical synchronization」。
即：对齐不是被「检测出来的过滤条件」，而是被「写进结构里的标注内容」。这与 UniVerse-1（SyncNet conf>2.0 硬阈值过滤）、MOVA 等以判别模型做闸门的路线是根本不同的方法论。
【评测侧：使用 SyncNet】生成结果的唇同步准确性用 SyncNet 评估，模型输出同步置信度分数与以帧为单位的时间偏移量。此外用 Shot Boundary Deviation 衡量生成镜头边界与脚本指定边界的帧级绝对偏差。
【论文对同步指标可靠性的重要发现】详见 human_in_loop 与 sync_metric_and_threshold 字段——SyncNet 分数在音频信息稀疏（平坦环境噪声）时会给出虚高的同步表现，是一个具有普遍参考价值的陷阱提示。
【数据侧是否另有同步过滤】论文未披露 500K 数据集或生成侧四套数据是否经过任何音画同步检测过滤，是清洗流程整体缺失的一部分。[不确定]

### 同步检测具体指标与阈值（SyncNet/Synchformer/LSE/自研，阈值数值，如MOVA LSE-D≤9.5且LSE-C≥4.5、SkyReels-V4 SyncNet |offset|≤3∧conf>1.5） ⚠️

【数据过滤阈值：无】论文未给出任何用于数据过滤的同步指标阈值——没有 SyncNet 置信度门槛、没有 AV-align 分数门槛、没有唇动检测门槛。这与 UniVerse-1（SyncNet conf > 2.0）、UniTalking（SyncNet conf > 0.9）等明确给出数值门槛的工作形成对比。本工作的对齐保证来自 schema 规则而非阈值过滤。[不确定]
【评测指标：SyncNet（chung2016out，Out of Time 原始版本）】输出同步置信度分数与帧级时间偏移。实测数值（多镜头场景）：
- LTX-2-AV（单体式提示基线）：6.86
- LTX-2-AV-MTSS（仅换提示词）：13.86
- Ours w/o ID：9.16
- Ours(Full)：9.72
【数值解读的关键警示】论文明确指出该自动指标具有欺骗性（"the automated A-V Sync metric can be deceptive"）：基线 6.86 看似最优，但其人评 A-V 分仅 1.18（1-3 分制最低档），原因是基线生成的是平坦的环境噪声——让近乎静止的音频包络与画面「同步」是平凡的（信息稀疏性伪影）。换用 MTSS 后模型开始尝试生成复杂对白，SyncNet 分数先升至 13.86（偏差扩大），完整方案再通过架构改进把时序缺口收窄到 9.72，同时人评 A-V 达 2.26（全表最优）。
这一发现对数据侧有直接的实践意义：若用 SyncNet 分数作为数据过滤闸门，会系统性地偏好无语音/弱音频内容的样本，反而滤掉信息丰富的对白样本。任何采用 SyncNet 阈值过滤的 pipeline 都应先做音频活跃度分层，再分层设阈值。
【其他相关指标与数值】
- Shot Boundary Deviation（帧）：LTX-2-AV 3.79 → LTX-2-AV-MTSS 3.27 → Ours w/o AV 1.28 → Ours w/o ID 0.38 → Ours(Full) 1.36；
- WER：LTX-2-AV 多镜头 0.84 → LTX-2-AV-MTSS 0.13 → Ours(Full) 0.19（单镜头基线 1.64 → 0.78 → 0.23）；
- Audio Quality（UTMOS）：4.12-4.18 → 4.60-4.79 → 4.68；
- Reference ID Similarity（ArcFace 余弦相似度）：单镜头 Ours w/o MS 0.62，多镜头 Ours w/o ID 无值、Ours(Full) 0.22、Ours w/o AV 0.20；
- Intra-Shot Subject Consistency（DINOv2 [CLS] 帧间平均余弦相似度）：LTX-2-AV 0.87（虚高）→ LTX-2-AV-MTSS 0.66 → Ours(Full) 0.59。

### 时序同步vs语义同步的分离处理（时间对齐与内容语义匹配作为两个独立过滤条件）

MTSS 在结构上把时序同步与语义同步彻底分开处理，这是 Relational Grounding 双链接设计的直接体现：
【时序对齐通道：时间链接（temporal links）】由三层时间戳承担——shot 级 time_range、event 级 time_range、描述内嵌的 intra-description / micro-level timestamps，加上 shared anchors（共享锚点）。这条通道只回答「什么时候」，全部信息为可做区间运算的数值区间。
【语义对齐通道：身份链接（identity links）】由中心化 Reference 实体库承担——Shot 的 references_in_shot、Event 的 speaker 均指向同一 ID 空间。这条通道只回答「是谁/是什么」，全部信息为符号引用。
【第三条隐性通道：因果对齐】Event 流的准入原则「音效必须由画面中可见的主体产生」实际上编码了声音与视觉的因果/来源关系，既非纯时序也非纯身份，而是「这个声音是画面里哪个东西发出的」。Shot 的 active_events 字段把镜头与并发事件关联，同样承载这层含义。
【三通道分离的价值】论文的核心论证正建立在此：单体式 caption 把 WHO、WHERE、WHEN 揉在同一段文本里，下游模型必须先做身份解析与时序去歧义才能推理；MTSS 把三者预先消歧，模型可直接专注逻辑推断。这解释了为什么 MTSS 在推理类基准（Daily-Omni、WorldSense）上的增益（Qwen3-Omni +127%）远大于在描述类基准上的增益。
【过滤条件层面】论文未把二者作为两个独立的数据过滤条件使用（因为整体上就没有披露过滤条件），二者的分离体现在表示层而非过滤层。这是与「时序同步与语义匹配作为两个独立过滤闸门」的传统做法不同的路径——MTSS 把这两个维度从「过滤时的判据」转化为「标注中的字段」。

### 音频质量过滤（SNR、静音检测与静音占比阈值、无音轨剔除、画外音源剔除、背景音乐分离） ⚠️

论文未披露任何数据侧的音频质量过滤：无 SNR 门槛、无静音检测与静音占比阈值、无「无音轨则剔除」规则、无背景音乐分离（如 Demucs 类源分离）、无响度/削波/失真检测。[不确定]
【schema 层面的等效机制】MTSS 用两条标注规则部分替代了传统的音频过滤：
1) 画外音源的处理——严格音视频耦合原则使无视觉对应的声音无法成为 Event 事件，实质上完成了「画外音源剔除」这一目标，但采用降级而非剔除：这些声音被归入 Global 流的 global_audio 字段继续保留。这是一个「不丢信息、只降层级」的设计，比直接剔除更节约数据。
2) 背景音乐的处理——不做信号级源分离，而是做语义级分层：构成独立叙事事件的音乐进 Event 流（type=music），仅作为氛围铺底的背景音乐进 Global 流的 global_audio。这是用标注层级替代音频信号处理的思路。
【评测侧的音频质量指标】UTMOS（轻量语音质量 MOS 预测器）用于评估生成音频的语音质量，非用于数据过滤。数值见 sync_metric_and_threshold 字段。
【隐含的音轨要求】整套 schema 强依赖真实音轨（Event 流与 global_audio 均需从原始音频中抽取），因此训练数据必然自带原生音轨，但论文未把这一点写成显式的过滤规则。

### 语音/音效/音乐的分类与分别处理策略 ⚠️

音频类型的分类与差异化处理是 MTSS 中定义最清晰的部分之一，采用「三类事件 + 一类全局」的四层划分：
【dialogue（对白/语音）】
- 归属：Event 流；
- 专属字段：speaker（关系绑定到 Reference ID，明确「谁在说」）、line（逐字台词文本）；
- description 字段额外承载情绪起伏（emotional shifts）与发声技巧（vocal techniques）等副语言信息；
- 是三类中字段最丰富的一类，反映其在音视频生成中的核心地位（唇同步与身份绑定均依赖它）。
【sfx（音效）】
- 归属：Event 流；
- 专属准入约束：必须由画面中可见的主体产生（"sound effects must be generated by a visible subject"）——这是三类中唯一带硬性视觉依据要求的类别，直接服务于 foley 类生成对「声音-动作因果关系」的需求；
- 无 speaker 与 line 字段，仅有 description。
【music（音乐）】
- 归属：Event 流（当音乐构成独立的叙事事件时）；
- 与 global_audio 中的背景音乐形成分层：前者是有明确起止、参与叙事的音乐（如画面中有人演奏、或作为情节点的配乐进入），后者是全片铺底、不构成独立事件的氛围配乐。
【global_audio（全局音频，第四层）】
- 归属：Global 流；
- 承载：不够格成为独立事件的环境音（ambient sounds）与背景音乐（background music），以及被过滤下来的无关背景噪声（irrelevant background noise）；
- 定位：兜底容器，保证音景信息不丢失的同时不污染 Event 流的信噪比。
【并发音源】拆分为并行的 event 条目，每条独立带 type、time_range 与内容块，不做混合描述。这使得「一个人说话的同时旁边有脚步声」这类场景可被完整且可分别控制地表达。
【生成侧的差异化验证】论文分别用 WER（对白内容准确性）、UTMOS（语音质量）、SyncNet（唇同步）三个指标评估语音相关能力，用定性的 RMS 包络分析评估音效——Figure 7/8 显示基线产生平坦的环境噪声型包络，MTSS 方案则呈现自然语音的节奏性起伏与动作驱动事件的周期性冲击（如脚步声的周期性 impact）。论文明确指出 Event 流把语音与音效生成「从近乎随机转变为语义准确」。
【数据侧配比】三类音频在训练数据中的比例未披露，也未提及按类别调整采样权重。[不确定]

## 训练配合

### 多阶段训练课程与数据课程调度（阶段划分依据：分辨率/时长/质量分/模态；低清→高清、图像→视频、短→长） ⚠️

caption 侧与生成侧的训练课程差异极大：
【caption 侧：单阶段，无课程】直接在开源 Qwen3-Omni-Instruct 上用 500K MTSS 数据做一次监督微调即得到 Qwen3-Omni-MTSS-FT。无多阶段、无分辨率/时长/质量分层的课程设计。论文用 "we simply performed supervised fine-tuning" 的措辞强调其简单性——这本身是论证的一部分：不需要复杂训练技巧，仅靠数据表示格式的改变就能带来大幅提升。
【生成侧：四阶段渐进课程，按「能力维度」而非「分辨率/时长」划分】这是本工作训练课程设计最有特点之处——阶段划分依据不是传统的低清→高清、短→长、图像→视频，而是 MTSS 四流各自对应的能力：
阶段 1 · ID Customization（身份定制）：400K identity-centric 数据集，3 epochs，128 GPU。目标是锚定持久的人物与环境特征，对应 Reference 流的能力。
阶段 2 · Multi-shot Control（多镜头控制）：250K 多镜头序列，1.5 epochs，32 GPU。目标是精调 shot-aware cross-attention 机制，对应 Shot 流的能力。注意此阶段 GPU 数最少（32 卡）且 epoch 数最少（1.5），是四阶段中投入最轻的。
阶段 3 · Audio-Visual Synergy（音视频协同）：870K cinematic pairs，3 epochs，400 GPU 集群。目标明确为 Gemma Connectors 的亚帧级对齐（sub-frame alignment），对应 Event 流的能力。此阶段数据量最大（870K）、算力投入最大（400 卡），说明音视频对齐被视为最难的能力。
阶段 4 · 联合微调（Joint Fine-tuning）：15K steps，64 GPU，使用 60K 高保真电影级对齐对 + 250K 多镜头序列的交错（interleaved）混合数据。目标是在单次生成过程中同时协调身份稳定性、叙事连贯性与时序同步，确保多重约束互不干扰（"without mutual interference"）。
【课程逻辑总结】前三阶段为「分能力单独优化」（staged optimization for individual tasks），第四阶段为「多能力联合协调」。这是一种与 MTSS 的流分解思想同构的训练设计——先分别掌握各流对应的能力，再重新协调，与 Stream Factorization + Relational Grounding 的表示设计形成呼应。
【未披露】各阶段的学习率、batch size、优化器、GPU 型号、分辨率/时长配置、总训练时长与 GPU-hours 均未给出。[不确定]

### 数据配比随训练阶段的变化（预训练/退火annealing/SFT高质量子集） ⚠️

生成侧的数据配比随阶段变化明确，但仅有条数无比例统计：
【阶段 1】纯 400K identity-centric 单一来源，无混合。
【阶段 2】纯 250K multi-shot sequences 单一来源，无混合。
【阶段 3】纯 870K cinematic pairs 单一来源，无混合。
即前三阶段均为单数据源的专项训练，不做混合——这是一种较为「干净」的课程设计，每阶段的学习信号高度纯粹，避免多目标互相稀释。
【阶段 4（唯一的混合阶段）】60K high-fidelity cinematic alignment pairs + 250K multi-shot sequences，以 interleaved（交错）方式混合。60K 高保真数据的引入是典型的「退火 / annealing」思路——在最后阶段引入规模小得多（60K vs 前期 870K，约 1/14.5）但质量更高的子集（"high-fidelity"），用于精修对齐质量。250K 多镜头序列则是从阶段 2 复用，用于防止多镜头能力在后续训练中遗忘。
【混合比例】60K : 250K 的名义条数比约为 1:4.2，但 interleaved 的具体采样权重、是否按能力做重加权，论文未说明。[不确定]
【caption 侧】无阶段划分，故无配比变化。
【值得注意的规模落差】阶段 3 用 870K 数据 + 400 卡训练 3 epochs，而阶段 4 仅 15K steps + 64 卡——最后的联合协调阶段算力投入仅为阶段 3 的约 1/6 且步数很少，说明其定位是轻量的「能力融合校准」而非再次的大规模学习。

### 后训练数据（SFT精选集规模与筛选标准、偏好对数量与标注方式、reward model训练数据） ⚠️

论文未涉及标准意义上的后训练环节：
【SFT】caption 侧的 500K MTSS 微调本身即是 SFT（且是唯一的训练阶段），筛选标准仅有「high-quality」与三个领域（film/television/lifestyle）的定性描述，无具体筛选指标。生成侧的四阶段全部为微调，不存在「预训练→SFT」的分层。
【偏好对齐】未使用任何偏好优化方法——无 DPO、无 RLHF、无 GRPO、无偏好对数据、无 reward model。论文的 Related Work 章节提及了 R1-Omni、EchoInk-R1、Omni-R1、HumanOmniV2、OmniVideo-R1 等一批用 RLVR/GRPO 做跨模态推理增强的工作，说明作者了解这条路线，但本工作明确不走强化学习路径，主张增益来自表示范式本身。
【reward model 训练数据】不适用。
【最接近「精选集」的成分】阶段 4 使用的 60K high-fidelity cinematic alignment pairs 可视为一个高质量精选子集（规模约为阶段 3 的 1/14.5），起到类似退火数据的作用，但论文未说明其筛选标准，也未称之为 SFT 集。[不确定]
【20 名专业评分员的人评数据】仅用于最终效果评估，未被回收用作偏好对或 reward model 训练数据。

### 数据处理基础设施与吞吐（NeMo Curator/Data-Juicer/自研，GPU加速比，处理规模，成本） ⚠️

论文完全未涉及数据处理基础设施：未提及 NeMo Curator、Data-Juicer 或任何自研数据处理框架，未给出 GPU 加速比、处理吞吐（clips/hour 或 hours/day）、处理耗时或成本估算。[不确定]
【可推断的算力成本量级】唯一可做粗略估算的是标注成本——500K 视频片段全部经 Gemini-2.5-Pro 逐条推理，且输出为包含四流多字段的长结构化 JSON，属于长输出的多模态 API 调用。以商用 API 计价，50 万次此类调用的成本量级不容忽视，但论文未披露费用、调用耗时、并发规模或是否使用内部部署。这也从侧面解释了为何要蒸馏出 Qwen3-Omni-MTSS-FT——用开源模型替代 Gemini 做大规模标注可显著降低边际成本，这是该工作在工程经济性上的实际价值所在。
【训练侧算力（有披露）】阶段 1 用 128 GPU、阶段 2 用 32 GPU、阶段 3 用 400 GPU 集群、阶段 4 用 64 GPU。GPU 型号、单阶段墙钟时间、总 GPU-hours 均未给出。400 卡集群的规模说明这是一个大厂级别的工程投入。
【数据存储与索引】MTSS 为 JSON 式结构化标注，理论上天然支持按字段检索（如「筛出所有含 3 个以上 shot 且含 dialogue 事件的片段」），对数据集的可检索性与可复用性有实际价值，但论文未讨论存储方案或检索基础设施。

## 效果对比

### 数据策略消融的量化影响（区分：过滤严格度ablation / caption密度风格ablation / 数据配比ablation，及对应评测指标）

本工作的消融设计集中在「表示格式」与「架构模块」两个维度，属于 caption 结构风格 ablation，而非过滤严格度或数据配比 ablation：
【维度一：caption 表示格式 ablation（本工作最核心的证据）】
设计极为干净：LTX-2-AV（单体式提示词）vs LTX-2-AV-MTSS（MTSS 结构化脚本），两者架构完全相同、训练范式完全相同，唯一变量是条件文本的组织结构。论文明确声明「any performance gap between them is directly and entirely attributable to the MTSS paradigm」。多镜头场景结果：
- 人评身份一致性（Cons.）：1.22 → 1.77（+45%）
- 人评音视频对齐（A-V）：1.18 → 1.85（+56%）
- 人评多镜头可控性（MSC）：1.00 → 1.71（+71%）
- WER：0.84 → 0.13（降低 85%）
- Shot Boundary Deviation：3.79 → 3.27 帧
- Audio Quality（UTMOS）：4.18 → 4.60
这组数据是「caption 结构本身即为性能杠杆」的强证据，且改造成本为零（不动架构、不加数据）。
【维度二：caption 侧的跨模型格式 ablation】在五个不同的 caption 模型上分别对比「原生单体式输出」与「MTSS 结构化输出」（零样本提示），全部为正增益：
- Gemini-2.5-Pro 总错误率 0.3959 → 0.2511（Miss 0.1902→0.1285，Incorrect 0.0848→0.0644，Hallucination 0.1209→0.0582，幻觉降低最显著达 52%）；UGC-VideoCap 综合 93.97 → 95.51；
- Qwen3.5-Omni-Flash 总错误率 0.5217 → 0.3655；UGC-VideoCap 76.07 → 83.28（其中 Audio Avg 从 65.92 大幅提升至 82.84）；
- Qwen3-Omni 总错误率 0.5853 → 0.5156；UGC-VideoCap 62.80 → 71.54（Audio Avg 52.81 → 68.79）。
特别值得注意：音频维度的增益普遍大于视觉维度，说明 Event 流的显式音频事件分解对听觉信息的捕捉改善最大。同时 Qwen3-Omni 在零样本 MTSS 下 Hallucination 反而从 0.1104 略升至 0.1355，是全表唯一的负向项，说明弱模型强行填结构化字段会诱发编造。
【维度三：SFT ablation】Qwen3-Omni-MTSS-FT（500K MTSS 数据微调）总错误率降至 0.3913、UGC-VideoCap 85.11、Daily-Omni 0.5945、WorldSense 0.3875，四项均大幅超越零样本 MTSS 版本，且总错误率已略优于 Gemini-2.5-Pro 的原生输出（0.3959）。证明 MTSS 数据的可学习性。
【维度四：架构模块 ablation（非数据侧）】Ours(Full) / w/o ID（去身份定制）/ w/o MS（去镜头感知注意力）/ w/o AV（去音频输入）四配置对比。关键发现：w/o ID 的 Shot Bnd. Dev. 最优（0.38 帧）而 Full 为 1.36 帧，揭示身份保持特征与时序镜头精度在 VLM 编码器-DiT 接口处存在权衡。
【缺失的 ablation】无过滤严格度 ablation（因为没有披露过滤流程）、无数据配比 ablation、无数据规模 ablation（未做 500K 的规模缩放实验）、无四流各自的消融（未验证去掉 Reference 流或 Global 流的影响）。最后一项是较可惜的缺口——论文在 Summary 中定性归因（Shot 流提供时序分段先验、Reference 流提供身份锚点、Event 流提供音频事件描述），但无逐流拆解的定量支撑。

### 质量vs数量的证据（小而精数据超越大而杂的案例） ⚠️

本工作提供的不是传统的「小数据超越大数据」证据，而是一个更本质的变体——「同样的数据，换一种组织结构，效果大幅提升」：
【核心证据：零成本的格式增益】LTX-2-AV 与 LTX-2-AV-MTSS 使用完全相同的架构、完全相同的训练数据与训练范式，唯一差异是条件文本的结构组织，即带来人评身份一致性 +45%、音视频对齐 +56%、多镜头可控性 +71%、WER 降低 85%。这说明在数据量与算力都不变的前提下，标注的「表示质量」本身即为一个独立且强效的性能维度。对于数据侧团队而言，这一结论的实践含义是：在考虑扩充数据规模之前，先审视现有标注的结构化程度可能是更高性价比的投入。
【第二重证据：结构化提升小模型的数据效率】论文将 Figure 1 的第二大卖点定为「Learnability」——MTSS 显著缩小了小模型与大模型的差距。Qwen3-Omni 在 Daily-Omni 上从 0.1806 提升到 0.4117（+127%），增幅远大于 Gemini-2.5-Pro 的 0.6825→0.7568（+11%）。即结构化 caption 对能力较弱的模型收益更大，机理是把「解开纠缠关系」这一负担从模型转移到了数据表示上。这实质上是「用标注结构换模型容量」，对算力受限的团队有直接价值。
【第三重证据：500K 蒸馏数据的效率】仅用 500K 条 Gemini 标注数据做一次简单 SFT，就使 Qwen3-Omni 的 Video-SALMONN-2 总错误率从 0.5853 降至 0.3913，超越教师模型 Gemini-2.5-Pro 的原生表现（0.3959）。学生在特定格式任务上超越教师，说明格式专精化的数据回报率极高。
【第四重证据：退火式高质量小集】生成侧最终阶段用 60K「high-fidelity」数据（规模仅为阶段 3 的 870K 的 1/14.5）+ 15K steps 完成最终协调，是典型的小规模高质量精修，但论文未做有无该阶段的对照实验。[不确定]
【局限】论文未做数据规模缩放实验（如 100K vs 500K 的对比），因此无法回答「MTSS 数据需要多少量才够」这一实践问题。

### 训练数据domain分布与评测基准类目体系的对齐关系（如VABench七大类）

本工作未构建自有评测基准，因此不存在「训练数据 domain 分布与自有基准类目体系对齐」的显式设计。但存在几层值得记录的对齐关系：
【训练数据领域与内部评测集的对齐】训练侧 500K 覆盖 film / television / lifestyle 三域；内部评测集 225 条覆盖 movie and TV drama clips、short-form videos、indoor scenes、outdoor scenes 四类。前两类（影视剧、短视频）与训练领域直接对应，后两类（室内/室外场景）是正交的场景维度补充。整体属同分布评测，无显式的分布外泛化测试。
【schema 设计与评测维度的对齐（本工作最清晰的对齐关系）】MTSS 四流与生成侧的五个人评维度存在近乎一一映射的关系：
- Reference 流 ↔ Identity Consistency（Cons.）+ Reference ID Similarity（ArcFace）
- Shot 流 ↔ Multi-Shot-Consistency（MSC）+ Shot Boundary Deviation + Intra-Shot Subject Consistency
- Event 流 ↔ Audio-Video Synchronization（A-V）+ WER + UTMOS + SyncNet
- Global 流 ↔ Semantic Following 中的 Scene 与 Style 两个子维度
这种「标注 schema 的每一条流都有对应评测指标」的设计，使消融分析可以定位到具体的结构成分，是本工作实验设计上的一个优点。同时 Semantic Following 的四个子维度（Subject / Action / Scene / Style）也与 MTSS 的信息组织高度对应——Subject 对应 Reference 流、Action 对应 Shot 流的 visual_description、Scene 与 Style 对应 Global 流。
【外部基准的选择逻辑】四个外部基准分工明确：Video-SALMONN-2 testset 与 UGC-VideoCap 测「描述保真度」（前者细分 Miss/Incorrect/Hallucination 三类错误，后者细分 Visual/Audio/Details 三个维度）；Daily-Omni 与 WorldSense 测「推理保真度」。UGC-VideoCap 的 Visual/Audio/Details 三维拆分恰好允许观察 MTSS 在视觉与听觉两侧的差异化增益（结果显示音频侧增益远大于视觉侧），这是基准选择与方法主张的良好对齐。
【无对应关系的部分】未与 VABench 等音视频专用基准的类目体系做对齐，也未报告训练数据在任何标准类目体系下的分布。

## 不确定字段

以下字段的调研信息部分不确定（⚠️ 标注来源）：

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
