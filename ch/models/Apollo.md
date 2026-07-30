# Apollo（arXiv v2 起更名为 Klear；同一篇论文 arXiv:2601.04151，v1 标题《Apollo: Unified Multi-Task Audio-Video Joint Generation》，v2 标题《Klear: Unified Multi-Task Audio-Video Joint Generation》。注意与 Meta 的视频理解模型 Apollo、Meta 的 Apollo LMM 等同名工作无关，需以 arXiv 编号区分）

> 主题: 视频生成模型（含音视频同时生成）的数据处理：数据清洗流程、数据分布、打标方式

[← 返回首页](../index.md)

**目录**: [基本信息](#基本信息) · [数据规模与分布](#数据规模与分布) · [清洗流程](#清洗流程) · [打标方式](#打标方式) · [音视频对齐](#音视频对齐) · [训练配合](#训练配合) · [效果对比](#效果对比)

## 基本信息

### 名称

Apollo（arXiv v2 起更名为 Klear；同一篇论文 arXiv:2601.04151，v1 标题《Apollo: Unified Multi-Task Audio-Video Joint Generation》，v2 标题《Klear: Unified Multi-Task Audio-Video Joint Generation》。注意与 Meta 的视频理解模型 Apollo、Meta 的 Apollo LMM 等同名工作无关，需以 arXiv 编号区分）

### 发布机构/公司 ⚠️

快手科技（Kuaishou Technology）可灵团队（Kling Team）。作者：Jun Wang、Chunyu Qiang、Yuxin Guo、Yiran Wang、Xijuan Zeng、Feng Deng 等。该工作是快手可灵（Kling）系列音视频同步生成能力（Kling 2.6「音画同出」、Kling 3.0）背后的研究性技术报告，属工业界闭源模型的对外论文披露。[不确定]（论文本身未显式声明与 Kling 产品线的对应关系，此关联为 HuggingFace 论文页署名「Kling Team, Kuaishou Technology」与产品时间线的旁证推断）

### 发布时间（技术报告/论文/开源时间）

2026年1月7日首次提交 arXiv（v1，标题为 Apollo）；2026年1月13日更新 v2（标题改为 Klear）。HuggingFace 论文页收录日期为 2026年1月8日。无独立的开源发布时间（未开源）。

### 类型（模型/数据集/工具链/评测基准）

模型（统一多任务音视频联合生成基础模型，26B 参数）。附带产出：论文自称构建了「首个大规模带密集 caption 的音视频数据集」（81M 样本）及配套自动化数据构造 pipeline，但该数据集与 pipeline 均未发布，因此不构成可用的数据集产出。评测沿用第三方基准 Verse-Bench，非自建基准。

### 开源程度（权重/代码/数据/pipeline各自是否开源）

完全闭源，是本次调研样本中开放度最低的一档。
【权重】未开源，论文与 HuggingFace 论文页均无模型链接。
【代码】未开源，论文中未给出 GitHub 仓库或项目主页 URL。
【数据】81M 样本的音视频-caption 三元组数据集未发布，论文对其「首个大规模带密集 caption 音视频数据集」的定位仅作贡献声明，无任何公开承诺。
【pipeline】方法论层面做了框架级披露（四阶段漏斗、所用打标模型清单、对齐检测工具、27% 保留率），但缺少可复现的关键要素：无逐指标阈值表、无逐级输入/输出量、无 prompt 原文、无伪代码、无数据处理脚本。相较同期开源工作（如 MOVA 公开 Table 9 全阈值表与打标 prompt 原文），Apollo 的数据披露停留在「说了做什么、没说怎么做」的层次，唯一超出同行的是给出了 27% 的端到端定量保留率。

### 是否支持音视频同时生成，以及实现方式（原生联合/级联/MoE融合）

支持，且为原生联合生成（单塔单次前向同时产出音频与视频 latent，非级联）。实现方式：
- 单塔（Single-Tower）MultiModal Diffusion Transformer（MMDiT）架构，含 32 层联合扩散层（joint diffusion layers），音频与视频共享同一套 DiT block 参数，而非双塔 + cross-attention。
- 核心机制为 Omni-Full Attention：音频 token 与视频 token 在同一注意力窗口内做全连接自注意力，实现紧密的音视觉对齐与良好可扩展性。
- 位置编码为 MixD-RoPE（Mixed Dimension Rotary Position Embedding），统一处理视频的 3 维（t,h,w）与音频的 1 维时间轴索引。
- 训练目标为 flow matching（条件去噪）。
- 模型总参数 26B，flow-matching FFN 维度 4096。
- 输入为四路：视频、视频相关文本、音频相关文本、音频，各自独立编码后送入 MM-DiT。
【与双塔路线的对比证据】论文 Table 2 直接对比「Dual Tower（标准 cross-attention）」与「Single Tower（Omni-Full Attention）」，结论支持单塔全注意力方案，这与 MOVA/HunyuanVideo-Foley 等双塔+bridge 路线形成明确的架构分野。
【多任务统一】通过随机模态掩码（random modality masking）在同一模型内统一支持 T2A、T2V、T2AV、I2V、I2AV 五类任务，因此同一权重既能联合生成也能单模态生成。

### 调研信息来源列表（论文/技术报告/官方文档/新闻的URL，标注每条来源的性质：官方一手/同团队旁证/第三方报道）

- 【官方一手】arXiv:2601.04151《Apollo: Unified Multi-Task Audio-Video Joint Generation》（v1，2026-01-07）/《Klear: Unified Multi-Task Audio-Video Joint Generation》（v2，2026-01-13）：摘要页 https://arxiv.org/abs/2601.04151 、HTML 全文 https://arxiv.org/html/2601.04151v2 与 https://arxiv.org/html/2601.04151v1 、PDF https://arxiv.org/pdf/2601.04151 —— 本条目几乎全部字段的唯一直接来源，数据相关内容集中在第 4 节 Dataset Construction（4.1 Dataset Filtering、4.2 Audio-Guided Data Splitting、4.3 Dense Annotation and Integration）与 Figure 3 数据标注 pipeline 图。该节篇幅极短（约一页），是本条目大量字段标为不确定的根本原因。
- 【官方一手】HuggingFace 论文页 https://huggingface.co/papers/2601.04151 —— 确认署名单位为「Kling Team, Kuaishou Technology」，确认未挂载任何模型、数据集或代码仓库链接。
- 【同团队旁证】快手投资者关系公告《Kling AI Launches Video 2.6 Model with 「Simultaneous Audio-Visual Generation」 Capability》：https://ir.kuaishou.com/news-releases/news-release-details/kling-ai-launches-video-26-model-simultaneous-audio-visual —— 用于印证该研究与可灵产品线音画同出能力的对应关系（旁证，非论文明示）。
- 【第三方索引】NASA ADS 条目 https://ui.adsabs.harvard.edu/abs/2026arXiv260104151W/abstract 、ResearchGate 条目 https://www.researchgate.net/publication/399559825_Klear_Unified_Multi-Task_Audio-Video_Joint_Generation —— 用于交叉核对 Apollo→Klear 的改名事实与作者列表。

## 数据规模与分布

### 训练数据量级（视频条数/小时数/token数，预训练与SFT分开） ⚠️

【总量】8100 万（81 million）条带精确密集 caption 的音视频样本，这是论文给出的唯一规模数字，对应过滤后的最终训练集（论文原文：「81 million samples with accurate dense captions」）。
【口径说明】论文以「样本条数」为唯一口径，未给出总小时数、未给出 token 数、未给出单条样本的时长，因此无法换算为小时量级。若按同类工作常见的 5–10 秒 clip 粗估，81M 条约对应 11 万–22 万小时，但此为外推而非论文数据。
【阶段拆分】论文的三阶段训练（预训练 / 专项后训练 / 高质量后训练）均未给出各阶段的数据规模，也未给出预训练与 SFT 的分开量级。Stage III 使用的「manually-curated, high-quality dataset」规模完全未披露。
【类别拆分】81M 中单说话人语音、多说话人语音、歌唱、自然声四类各自的条数或占比未给出。
【结论】Apollo 在数据规模维度只公开了一个总数，粒度显著低于同期工作。[不确定]

### 数据来源构成（自有/公开数据集/网络爬取/授权采购/合成数据） ⚠️

论文对数据来源构成的披露基本为空白：未列举任何公开数据集名称，未说明自有数据、网络爬取、授权采购各自的占比，也未提及数据获取渠道。可作出的有限推断：
- 数据形态覆盖单说话人语音、多说话人语音、歌唱、自然声（natural sound）四类场景，且要求原生同步音轨（因为整个 pipeline 建立在对已有视频的音轨做质量与同步过滤之上，而非配音合成），说明数据来自真实带声视频而非无声视频后补音。
- 8100 万条的量级、以及快手作为短视频平台的属性，强烈暗示以平台自有/授权的短视频语料为主，但论文未作任何说明。
- 合成数据：论文未使用模型合成的音视频内容，唯一的「合成」成分是全部 caption 由 ASR/音频 captioner/视频专家模型自动生成（属合成标注而非合成内容）。
[不确定]

### 数据合规与溯源（授权数据占比、rights-cleared数据集、C2PA等） ⚠️

论文完全未涉及数据合规与溯源议题：未给出授权数据占比，未声明 rights-cleared 数据集，未提及 C2PA 或任何输出侧水印/溯源标记，未讨论版权、肖像权或数据法律状态，也无模型卡级的使用条款。数据过滤清单中出现的「safety」一词是唯一与合规沾边的表述，但无任何展开。作为闭源工业模型，其合规工作可能存在于内部流程但未对外披露。[不确定]

### 片段时长分布与切分策略 ⚠️

论文未披露任何片段时长信息：未给出训练 clip 的固定时长或时长分布，未给出帧率，未给出最短/最长时长阈值，也未说明是定长切分还是变长分桶。仅能从 VAE 规格间接约束：Video-VAE 输出 3 Hz 的时序 embedding（即每秒 3 个 latent 帧），Audio-VAE 输出 43 Hz 的 embedding；论文明确 Video-VAE「handles input videos with varying resolutions and frame rates」，说明输入侧帧率不统一，由 VAE 归一化到 3 Hz 表征。切分策略层面唯一可确认的是场景切分（scene splitting）保证每条样本只含单一场景，但切分后的窗口长度规则未说明。[不确定]

### 分辨率/宽高比分布与分桶策略 ⚠️

论文未披露分辨率与宽高比的分布或分桶策略：未给出训练分辨率、未给出宽高比枚举、未提及多分辨率分桶（bucketing）或渐进式升清课程。相关的仅两处：(1) 过滤阶段会「discard those videos with low resolution」（剔除低分辨率视频），但未给出分辨率下限阈值；(2) Video-VAE 采用 CogVideoX 的 3D causal visual encoder，对高、宽各做 16 倍压缩，并声明可处理「varying resolutions and frame rates」，即架构上支持变分辨率输入，但训练实际使用的分辨率配置未公开。[不确定]

### 类别/domain分布与配比策略（人物、动作、场景、风格等比例控制与概念均衡） ⚠️

论文对视觉 domain（人物、动作、场景、风格等）的分布与配比策略完全没有描述，也未提及概念均衡（concept balancing）机制。论文中唯一的「分布」概念是按音频类型划分的四类场景（单说话人语音 / 多说话人语音 / 歌唱 / 自然声），即 Apollo 的数据组织轴心是**音频类型而非视觉 domain**——这是本工作在数据设计上最显著的取向：它把「听到什么」而非「看到什么」作为数据切分的第一性维度（见 audio_category_distribution）。
可间接确认的配比意图来自训练策略而非数据统计：Stage II「专项后训练」会「adaptively rebalance data distributions across scenarios and tasks to strengthen underperforming capabilities while preserving overall competency」（依据评测指标自适应地在场景与任务间重新平衡数据分布，以补强表现不佳的能力同时不损伤整体能力），说明存在指标驱动的动态配比机制，但具体的场景清单、初始配比与调整后配比均未给出数字。[不确定]

### 音频类别分布与配比（语音/音效foley/音乐/环境音/静音的比例及控制策略）——AV模型独有维度 ⚠️

这是 Apollo 数据设计的核心维度，也是其区别于视觉优先同类工作的关键点——整个数据集按音频类型做树状分层切分（Section 4.2 Audio-Guided Data Splitting）：
【第一层：人声 vs 非人声】先将数据分为 vocal（含人声）与 non-vocal 两支，非人声支构成 sound split（自然声/音效子集）。
【第二层：人声内部三分】从 vocal 子集进一步划分为三类——singing（歌唱）、single-speaker speech（单说话人语音）、multi-speaker speech（多说话人语音）。
【最终四类】single-speaker speech / multi-speaker speech / singing / natural sound，论文原文：「The dataset contains single-speaker speech, multi-speaker speech, singing, and natural sound clips」。
【差异化标注】不同子集走不同标注路径——语音与歌唱子集额外抽取说话人属性（性别、年龄）并做逐字转写；sound split 只做音频 caption，不做转写。这是「按音频类别分流后再分别打标」的典型设计。
【与同类对比】值得注意的是 Apollo 显式保留了 singing（歌唱）这一类，而 MOVA 等工作因音频塔容量受限在歌声上表现退化并将其排除或弱化；Apollo 把歌唱作为一级子集，说明其在数据侧刻意覆盖了唱歌口型这一高难场景。同时它并未像 MOVA 那样「只保留语音」，而是让 natural sound（foley/环境音）与语音共存于同一训练集，配合多任务掩码训练同时习得 T2A 与唇同步能力。
【空白】四类各自的样本数或占比、音乐（instrumental music）是否单列、静音样本的处理（仅知静音占比 >20% 的样本被剔除）等均未披露。[不确定]（仅类别体系明确，比例数字缺失）

### 叙事结构分布（单镜头vs多镜头、平均clip时长、镜头数分布、是否含原生音轨） ⚠️

【单镜头 vs 多镜头】明确采取纯单镜头策略：论文原文「We then apply scene splitting to ensure each sample contains only one scene」（应用场景切分以确保每条样本只包含一个场景）。即训练数据全部为单镜头片段，不包含跨镜头转场样本。这与 MOVA 刻意构造「单场景/多场景」2×2 数据维度的做法形成对比，也意味着 Apollo 未在数据侧建模镜头转场能力。
【原生音轨】训练数据必须自带原生同步音轨——整个 pipeline 的音频过滤与音视频一致性检测环节均以原始音轨为对象，无音轨或音轨异常的样本在过滤阶段被剔除。
【平均 clip 时长 / 镜头数分布】未披露。[不确定]（时长数字缺失，单镜头结论确定）

### 语言/口音分布（多语种唇同步能力的数据基础） ⚠️

论文完全未讨论语言与口音分布：未列举支持语种，未给出各语种占比，未提及口音标注或多语种唇同步的数据基础。可作出的间接推断：转写环节同时使用 Whisper-Large-v3（多语种 ASR）、SenseVoice（阿里开源的中英日韩粤多语种语音理解模型，对中文与中国方言支持突出）与 Qwen2.5-Omni（中英双强），三者并用的组合强烈暗示语料至少覆盖中文与英文双语，且 SenseVoice 的引入指向中文/方言语料占相当比重；caption 文本编码器为 Qwen2.5-7B（中英双语模型）也支持这一推断。但这些均为工具链推断而非论文陈述。评测指标含 WER（词错误率），说明存在转写准确性评估，但评测语种未说明。[不确定]

## 清洗流程

### 清洗漏斗整体结构（几级过滤、各级顺序）

四阶段串行漏斗（论文第 4 节 + Figure 3「Overview of our Dataset Annotation Pipeline」）：
【阶段 1 — 视频过滤与场景切分（Video Filtering and Scene Splitting）】先从四个维度建模视频质量：动态质量（主体运动比例、镜头稳定性）、静态质量（清晰度、美学、色彩饱和度）、内容自然度（无过度特效/水印）、安全性；剔除低分辨率、低 SNR/MOS、静音占比超 20% 的视频；随后做场景切分，保证每条样本仅含单一场景。
【阶段 2 — 音频过滤与后处理（Audio Filtering and Post Processing）】剔除低 SNR、低 MOS、异常削波（clipping）、失真、噪声的样本，要求静音占比 <20%、高保真、格式统一；随后做音视频一致性检测——Synchformer 做时序对齐、ImageBind 做语义对齐，确保时序与语义两个维度均高度同步。
【阶段 3 — 音频引导的数据切分（Audio-Guided Data Splitting）】按音频类型把数据集分层：先分人声/非人声得到 sound split，再从人声子集分出歌唱、单说话人语音、多说话人语音三支。
【阶段 4 — 密集标注与融合（Dense Annotation and Integration）】按子集分别调用专用模型产出语音转写、音频 caption、视频 caption（含 meta 信息与详细内容），语音与歌唱子集额外抽取说话人属性（性别、年龄），sound split 只给音频 caption；最后把所有标注融合为统一的密集 caption（unified dense captions）。
【漏斗排序特征】与 MOVA 一致，把最昂贵的多模态打标放在漏斗最末端，只对通过质量与同步过滤的样本做标注；差异在于 Apollo 在打标前多插了一层「按音频类型分流」，使打标策略可按子集定制，这是其 pipeline 结构上的独特点。
【披露粒度局限】四个阶段均只有定性描述，无阈值表、无各级样本量、无伪代码。

### 漏斗定量保留率（每级过滤的输入/输出量与最终保留率，如Apollo 27%）

【核心数字】整体过滤后保留率 27%，论文原文：「with an overall post-filtering retention rate of 27%」。这是本次调研中少见的公开端到端定量漏斗指标，与 MOVA 的 26.39% 处于同一量级，两者相互印证了「音视频联合生成的数据漏斗大致只留下四分之一多一点」这一经验规律。
【口径】论文未说明 27% 是按样本条数、总时长还是原始视频条数计算，从上下文（紧接在「81 million samples」之后）推测为样本/片段条数口径，但未明示。若按条数口径反推，过滤前候选样本约 3 亿条（81M ÷ 27% ≈ 300M）——此为推算而非论文数据。
【逐级缺失】与 MOVA 公开逐级保留率表（100% → 84.57% → 58.75% → 26.39%）不同，Apollo 只给出端到端的单一总数，四个阶段各自的输入/输出量、各级淘汰占比、各子过滤（视频质量 / 音频质量 / 音视频一致性）各自砍掉多少均未披露，因此无法定位主要损耗环节。
【结论】27% 是一个有价值但孤立的锚点数字，可用于横向对标，不足以支撑 pipeline 逐级复现。

### 镜头切分方法（PySceneDetect/自研模型/shot-aware splitting） ⚠️

确认存在场景切分环节，目的明确但方法未公开：论文仅称「We then apply scene splitting to ensure each sample contains only one scene」（做场景切分以确保每条样本只含一个场景），置于视频质量过滤之后。未说明使用 PySceneDetect、TransNetV2 还是自研模型，未给出切点检测阈值，未说明切分后如何采样窗口，也未说明是否像 MOVA 那样把语音边界（VAD）与场景切点联合起来做 shot-aware + speech-aware 的双重感知切分。考虑到 Apollo 数据以语音/歌唱为主体，若切分不感知语音边界会导致语句被截断，但论文对此未作说明。[不确定]

### 质量过滤（美学评分、清晰度、OCR文字过滤、黑边/水印/logo检测） ⚠️

分视频侧与音频侧两条线，维度枚举清楚但阈值几乎全部缺失：
【视频侧四维质量建模】
- 动态质量（dynamic quality）：主体运动比例（subject motion ratio）、镜头稳定性（camera stability）
- 静态质量（static quality）：清晰度（sharpness）、美学（aesthetics）、色彩饱和度（color saturation）
- 内容自然度（content naturalness）：无过度特效与水印（no excessive effects/watermarks）
- 安全性（safety）
【视频侧硬性剔除条件】低分辨率、低 SNR/MOS、静音占比超过 20%（原文：「We discard those videos with low resolution, low SNR/MOS, or over 20% silence」）。注意此处 SNR/MOS 本属音频指标却出现在视频过滤条款中，说明音画质量在第一道闸门就被联合考量。
【唯一公开的数值阈值】静音占比 < 20%（silence ratio < 0.2）。视频质量各维度的打分模型（是否用 LAION-Aesthetic、DOVER、MUSIQ 等）与阈值数值均未给出；论文评测部分使用 MANIQA、aesthetic-predictor-v2-5、Musiq 计算美学分，但那是评测指标而非数据过滤器，不可混用。
【水印/logo】仅以「no excessive effects/watermarks」一句纳入内容自然度维度，未说明检测方法（是否 OCR、是否专用水印检测模型），也未说明是剔除还是裁剪。
【OCR/烧录字幕】完全未提及，不同于 MOVA 把「有无烧录字幕」做成显式可控属性的做法。[不确定]（维度明确，阈值与方法缺失）

### 运动过滤（光流/运动分数阈值、静态与抖动剔除） ⚠️

存在运动相关的过滤维度，但无方法与阈值：论文将「subject motion ratio（主体运动比例）」与「camera stability（镜头稳定性）」列为动态质量（dynamic quality）建模的两个子维度，纳入第一道视频过滤。这意味着静态呆板片段（运动比例过低）与手持抖动/剧烈晃动片段（镜头稳定性差）在设计上都会被处理，方向与常规做法一致。但论文未说明是用光流（RAFT/UniMatch）、帧差还是专用模型计算，也未给出运动分数的上下界阈值。评测环节使用 RAFT 光流计算 Motion Score（MS），但这属于评测指标，论文未说明数据过滤是否复用同一工具。[不确定]

### 去重方法（精确去重与基于embedding的语义去重分别记录） ⚠️

论文完全未提及去重环节：既无哈希/指纹级精确去重，也无基于 embedding 的语义去重。在 8100 万条、疑似来自短视频平台的语料规模下，重复内容（同一素材的多次搬运、模板化二创、同一视频的重叠切片）风险客观存在，且论文的场景切分会从同一长视频产出多个片段、天然带来近重复，但论文对此未作任何说明。[不确定]

### VLM/LLM作为质检员（多模态大模型质量打分与错配剔除，2026年从浅层打分器转向大模型语义判断的趋势） ⚠️

Apollo 的数据 pipeline 体现了明显的「大模型深度介入」特征，但其角色集中在**标注生成**侧，而非**质量裁决**侧，这与 2026 年「从浅层打分器转向大模型语义判断」的趋势只部分吻合：
【大模型承担标注】Qwen2.5-Omni（全模态大模型）同时用于语音转写与音频 caption；Gemini 2.5-Pro（闭源商用旗舰多模态大模型）用于音频 caption——引入商用 API 级大模型做大规模标注，在 8100 万条的量级下是相当高成本的选择，反映出快手对标注质量的投入优先级。视频侧使用「video expert model」（视频专家模型，未具名，推测为内部自研 VLM）。
【质量裁决仍由专用模型/信号处理指标承担】音频质量用 SNR、MOS、削波/失真/噪声检测等传统信号与感知指标；音视频对齐用 Synchformer（时序）与 ImageBind（语义）两个专用判别模型；视频质量用未具名的多维打分器。全流程中没有出现「用 VLM/LLM 打一个综合质量分」或「LLM-as-judge 做跨模态一致性裁决」的环节。
【关键缺失】与 MOVA 用 GPT-OSS-120B 显式做视觉-音频一致性冲突消解、并在 prompt 中内置反幻觉自审计字段的做法相比，Apollo 论文中没有任何关于标注幻觉抑制、跨模态串扰防御或标注结果二次校验的描述——多个模型（Whisper / SenseVoice / Qwen2.5-Omni 三个 ASR 并用）产出的结果如何仲裁与融合，论文只用「All annotations are merged into unified dense captions」一句带过，融合规则完全未公开。
【推断】三个 ASR 模型并用最可能的用途是交叉验证/投票以剔除转写不可靠的样本（这实质上是一种模型集成质检），但论文未证实此点。[不确定]

### 安全与合规过滤（NSFW、版权、人脸/隐私） ⚠️

论文对安全与合规过滤的披露仅有一个词：「safety」被列为视频质量建模的第四个维度（与动态质量、静态质量、内容自然度并列），置于第一道过滤闸门。没有任何展开——未说明 NSFW 检测方法或模型，未提及版权过滤，未提及人脸识别/隐私保护/名人肖像过滤，未给出安全类目体系，也无模型卡级的安全声明或使用限制。作为快手这类需承担内容监管责任的平台方，其内部必然存在成熟的安全审核体系，但论文选择不披露。[不确定]

## 打标方式

### 使用的caption模型（自研VLM/开源模型，模型规模）

采用「按子集分流 + 多模型并用 + 统一融合」的标注模型矩阵，全部为开源或商用 API 模型，视频侧为未具名自研模型：
【语音/歌唱转写（三模型并用）】Whisper-Large-v3（OpenAI，多语种 ASR，~1.55B）、SenseVoice（阿里 FunAudioLLM，多语种语音理解，支持中英粤日韩及情感/事件识别，~234M）、Qwen2.5-Omni（阿里全模态大模型，7B/3B 版本）。三者并用的组合在同类工作中少见，推测用于交叉验证或按语种/场景分派。
【音频 caption（双模型并用）】Qwen2.5-Omni（开源全模态）+ Gemini 2.5-Pro（Google 闭源旗舰多模态）。在 8100 万条量级上调用 Gemini 2.5-Pro 是显著的成本投入，也说明部分高价值子集走了更强模型。
【视频 caption】「a video expert model for detailed video labels」（视频专家模型，产出详细视频标签）——未具名、未给规模，推测为快手内部自研视频理解模型（与可灵体系配套）。
【说话人属性抽取】性别、年龄等属性由上述音频模型体系抽取，未单列专用模型。
【融合】「All annotations are merged into unified dense captions」，融合执行者未说明是规则拼接还是 LLM 改写——这是与 MOVA（明确用 GPT-OSS-120B 做融合与一致性校验）的显著差异点。
【下游文本编码】caption 送入 Qwen2.5-7B 作为文本编码器；另有 1024 维的 TTS 专用文本编码器处理待合成的语音文本；v1 中另提及 Qwen3-8B Embedding。

### caption密度与结构化程度（短/长/密集描述、结构化字段如镜头运动、风格标签） ⚠️

【总体定位】论文将其数据集定位为「the first large-scale audio–video dataset with dense captions」（首个带密集 caption 的大规模音视频数据集），密集（dense）与精确（accurate）是其自我标榜的两个关键属性，但 caption 的实际形态未给出任何样例。
【结构化程度】论文明确 caption 覆盖两个层次：「including both meta information and detailed content」（同时包含元信息与详细内容）。元信息层包括说话人属性（性别、年龄）等结构化字段；详细内容层为自然语言描述。这是一种「结构化 meta + 自然语言正文」的混合结构。
【分流条件化】caption 内容随音频子集而变：语音/歌唱子集含转写文本 + 说话人属性 + 音频 caption + 视频 caption；sound split 只含音频 caption + 视频 caption，不含转写与说话人属性。即 caption schema 是随数据类别动态裁剪的。
【最终态】「merged into unified dense captions」——所有标注最终融合为统一的密集 caption，不保留分离字段（与 MOVA 的「标注时分流、训练时融合」路径一致）。
【模型侧的条件通道】值得注意的是，模型架构层面输入是四路而非一路：视频、视频相关文本（video-related text）、音频、音频相关文本（audio-related text）——即视频 caption 与音频 caption 在**输入通道层面仍然是分离的两个文本条件**，并非合并为单一 prompt。这与「unified dense captions」的表述存在张力，可能的解释是数据侧生成统一 caption 后在训练时再拆回两路，或两种表述描述的是不同环节。
【关键缺失】caption 长度分布、词数统计、prompt 原文、完整 caption 示例、是否含镜头运动/风格标签等结构化字段，全部未披露。[不确定]

### 音视频联合caption结构（是否同时覆盖视觉+听觉轨道、是否分流为独立字段，如LTX-2全音景描述、Script-a-Video factorized streams、Foley-Omni三字段） ⚠️

Apollo 采用的是「按音频类型分流标注 → 各轨独立产出 → 融合为统一密集 caption」的方案，但在模型输入侧又保留了视觉/听觉双文本通道，属混合形态：
【标注轨道划分】三类标注并行产出——(1) 语音转写（speech transcripts，仅 vocal 子集）；(2) 音频 caption（audio captions，所有子集）；(3) 视频 caption（video captions，所有子集）。语音与歌唱子集额外附加说话人属性（性别、年龄）作为 meta 字段。
【与同类方案的坐标】
- 相比 Foley-Omni 的三字段并列长期保留，Apollo 在数据侧做了融合（merged into unified dense captions）。
- 相比 LTX-2 的单一全音景描述，Apollo 保留了「转写 / 音频描述 / 视频描述」的三轨来源结构与按子集裁剪的条件化 schema。
- 相比 Script-a-Video 的 factorized streams，Apollo 的分流依据是**音频内容类型（人声/非人声、单人/多人/歌唱）**而非叙事要素，这是其 schema 设计最独特之处：schema 本身随音频类别变化。
- 相比 MOVA 的「三轨严格互斥 prompt + 120B LLM 融合并做跨模态一致性裁决」，Apollo 未披露任何轨道间的互斥约束（如禁止视觉标注器参考音频）与融合规则，融合环节是黑箱。
【模型侧双文本通道】架构上音频相关文本与视频相关文本作为两个独立输入分别编码后送入 MM-DiT，因此推理时可对音、视两侧分别下达指令（这正是论文强调的「instruction following in both joint and unimodal settings」的基础），也支撑 T2A / T2V / T2AV 多任务统一。
【缺失】融合规则、字段名、schema 定义、示例均未公开。[不确定]（分轨结构确定，schema 细节缺失）

### 对白转写与说话人属性标注（ASR转写+说话人身份/语言/口音/情绪） ⚠️

这是 Apollo 数据体系中披露相对最实的一环，也是其相对同类工作的差异化投入：
【转写】对 vocal 子集（含歌唱、单说话人语音、多说话人语音）做语音转写，三模型并用：Whisper-Large-v3、SenseVoice、Qwen2.5-Omni。三者能力互补（Whisper 多语种通用、SenseVoice 中文/方言与情感事件、Qwen2.5-Omni 全模态上下文理解），推测用于交叉验证或按场景分派，但仲裁规则未披露。
【说话人属性】明确抽取说话人属性并举例为性别（gender）、年龄（age）：「For speech and singing, we extract speaker attributes (e.g., gender, age)」。「e.g.」说明属性集不止这两项，但完整清单未给出。相比 MOVA 以自然语言描述音色/口音/语速的做法，Apollo 的性别、年龄更接近离散结构化字段。
【说话人数量维度】通过数据切分而非标注字段来表达——单说话人语音与多说话人语音是两个独立子集，模型可据此区分单人/多人场景。但多说话人子集内部是否做了说话人分离（diarization）与说话人标签（[S01]/[S02]）标注，论文未说明；这恰是 MOVA 明确点出的多说话人场景瓶颈。
【口音/情绪】未提及口音标注；未提及情绪标注（尽管 SenseVoice 原生支持情感识别）。
【评测对应】评测指标含 WER（词错误率）与 SyncNet Confidence（唇同步置信度），对应验证转写内容的可生成性与唇形对齐。Table 3 显示全任务训练把 WER 从 0.044 降到 0.028。
[不确定]（属性集完整清单、diarization 处理、多 ASR 仲裁规则缺失）

### 几何与结构化标注（相机参数、深度、3D point tracks、动作标注、显式状态）

论文未使用任何几何或结构化的视觉标注：无相机参数（内外参、轨迹）、无深度图、无 3D point tracks、无骨架/关键点/动作标注、无显式物理状态标注、无 bounding box 或分割掩码。唯一接近结构化的标注是音频侧的说话人属性 meta 字段（性别、年龄）。位置信息完全由 MixD-RoPE 在模型内部以位置编码形式隐式处理（视频 3 维 + 音频 1 维），而非作为数据标注引入。「camera stability（镜头稳定性）」虽在过滤维度中出现，但那是一个用于筛选的标量质量分，不构成相机运动的结构化标注。

### 合成数据构造（受控扰动/编辑构造训练对，如InstructAV2AV）

论文未提及任何合成数据构造：无受控扰动/编辑构造的训练对，无 InstructAV2AV 式的指令编辑数据对，无 TTS 配音合成的伪音视频对，无模型自生成数据回流（self-distillation / rejection sampling）。全部 8100 万训练样本来自真实带原生音轨的视频，pipeline 的角色是过滤与标注而非生成。唯一的「合成」成分是全部 caption 由自动模型产出（合成标注，非合成内容）。多任务能力（T2A / T2V / T2AV / I2V / I2AV）不是通过构造不同形态的合成数据对得到的，而是通过**随机模态掩码（random modality masking）**在同一批真实数据上动态构造训练目标——这是一种「训练时合成任务」而非「数据侧合成样本」的思路，也是 Apollo 相对省数据工程量的关键设计。

### 人工介入程度（人工标注、人工质检、模型初筛+人工复核） ⚠️

人工介入程度极低且披露模糊，全流程唯一一处人工痕迹在训练末端而非数据 pipeline 中：
【数据 pipeline】完全自动化。论文明确定位为「a novel automated data-construction pipeline」（新颖的自动化数据构造 pipeline），81M 样本全部为「automatically annotated samples」（自动标注样本）。过滤阈值如何确定（是否像 MOVA 那样由人工抽检不同 cutoff 下的留存样本来标定）未说明；标注结果是否做人工抽检验证未说明；未提及任何人工标注、人工质检或模型初筛+人工复核环节。
【训练末端】Stage III 使用「the manually-curated, high-quality dataset」（人工精选的高质量数据集）做最终微调——这是全文唯一的人工介入表述，但该数据集的规模、人工筛选标准、参与人数、标注规范全部未披露。
【结论】Apollo 的定位是「自动化 pipeline 做大规模、人工精选做末端提纯」的两段式，人力集中投放在最后一公里，但这一公里的所有细节都是黑箱。[不确定]

## 音视频对齐

### 音视频同步检测方法（口型同步、事件对齐） ⚠️

音视频一致性检测是 Apollo 数据过滤中方法披露最明确的一环，采用**双工具、双维度**设计（置于音频过滤之后、数据分流之前）：
【时序对齐】使用 Synchformer 检测音视频的时间同步性。Synchformer 是基于 Transformer 的稀疏音视频同步检测模型，可估计音画时间偏移，也是 MMAudio、MOVA 等工作采用的同一工具（并对应评测指标 DeSync / AV-A）。
【语义对齐】使用 ImageBind 检测音视频的语义匹配度。ImageBind 将图像、音频等六种模态嵌入统一空间，通过跨模态余弦相似度衡量「画面里的东西和听到的声音是不是一回事」。
【论文原文】「We then assess audio–visual consistency, using Synchformer for temporal alignment and ImageBind for semantic alignment, ensuring high synchronization in both temporal and semantic dimensions.」
【唇同步专项】数据过滤环节未提及 SyncNet/LSE-D/LSE-C 等唇同步专用检测器——这是一个值得注意的缺口：Apollo 的核心卖点之一是解决「poor lip–speech alignment（唇语对齐差）」，但其数据侧的对齐过滤只用了通用的 Synchformer + ImageBind，未见针对人脸口型的专项过滤（对比 MOVA 明确用 LSE-D ≤ 9.5、LSE-C ≥ 4.5 筛出 2.5M 唇音高质量子集）。SyncNet Confidence 在 Apollo 中只作为**评测指标**（Table 3 中 Sync-conf 从 5.024 提升到 6.787）出现，而非数据过滤器。若属实，说明 Apollo 的唇同步能力主要靠架构（Omni-Full Attention）与多任务训练而非数据侧的唇同步筛选获得。[不确定]（唇同步是否用于数据过滤未明示）

### 同步检测具体指标与阈值（SyncNet/Synchformer/LSE/自研，阈值数值，如MOVA LSE-D≤9.5且LSE-C≥4.5、SkyReels-V4 SyncNet |offset|≤3∧conf>1.5） ⚠️

【指标】明确：时序同步用 Synchformer 输出的对齐分/偏移量，语义同步用 ImageBind 跨模态相似度。
【阈值】完全未披露——论文只说「ensuring high synchronization in both temporal and semantic dimensions」（确保时序与语义两个维度均高度同步），没有给出 Synchformer 的偏移量上限（如 |offset| < 0.2s）或分数下限，也没有给出 ImageBind 相似度的阈值（如 IB-score > 0.2），更没有说明两个条件是与关系还是加权组合。
【对比参照】同期工作中 MOVA 公开了完整阈值表（Audiobox PQ>5.0/CU>4.5/CE>2.5、DOVER-Aesthetic>0.85、LSE-D≤9.5、LSE-C≥4.5 等），UniTalking 公开 SyncNet conf>0.9，Apollo 在这一维度的披露显著落后。
【评测侧数值（非过滤阈值，不可混用）】Table 3 给出模型输出的同步指标：DeSync 0.650（越低越好）、Sync-conf 6.787（越高越好）、IB 0.316（越高越好）。
[不确定]

### 时序同步vs语义同步的分离处理（时间对齐与内容语义匹配作为两个独立过滤条件）

Apollo 明确地把时序同步与语义同步分离为两个独立的过滤条件，并各配一个专用模型——这是其数据 pipeline 中设计意图最清晰的一处：
- 时序维度：Synchformer，回答「声音和画面在时间轴上对不对得上」（口型是否跟得上语音、脚步声是否踩在落脚帧）。
- 语义维度：ImageBind，回答「声音和画面讲的是不是同一件事」（画面是海浪、声音却是键盘敲击，则时序上可能无冲突但语义完全错配）。
二者并列施加，共同构成「高度同步」的判定。这种「时序 ≠ 语义、需分别把关」的认识，正是为解决论文开篇指出的两类失败模式而设——asynchrony（不同步，时序问题）与 misalignment/lip–speech mismatch（错配，语义与口型问题）。
值得注意的是这一分离在评测侧被对称复用：AV-A（Synchformer 系，时序）与 IB score（ImageBind，语义）作为两个独立指标并列报告，数据过滤器与评测指标同源，形成「用什么筛就用什么评」的闭环——这既保证了训练目标与评测的一致性，也带来指标自证的潜在风险（用 Synchformer 筛出的数据训出的模型，在 Synchformer 系指标上天然占优），论文未讨论这一点。

### 音频质量过滤（SNR、静音检测与静音占比阈值、无音轨剔除、画外音源剔除、背景音乐分离） ⚠️

音频质量过滤是 Apollo 独立成段（Section 4.1 后半）的环节，维度枚举较全但阈值只公开一个：
【剔除条件】低 SNR（信噪比）、低 MOS（平均意见分，感知音质）、异常削波（abnormal clipping）、失真（distortion）、噪声（noise）。
【静音控制】要求静音占比低于 20%（「ensuring less than 20% silence」）——这是音频侧唯一公开的数值阈值，且在视频过滤条款中重复出现（「discard those videos with ... or over 20% silence」），说明静音占比是横跨音视频两道闸门的强约束，目的是避免模型学到大段无声输出。
【其他要求】高保真（high fidelity）、格式统一（consistent formatting，指采样率/声道/编码的规整化）。VAE 侧规格为 44.1 kHz 输入。
【未涉及的常见环节】论文未提及无音轨样本的剔除（虽然逻辑上被 pipeline 隐含要求）、未提及画外音/旁白源的剔除（即声源不在画面内的情形，这对唇同步训练是重要干扰源，MOVA 等工作会专门处理）、未提及背景音乐分离（BGM separation，如用 Demucs/UVR 剥离配乐）——对以歌唱为一级子集的 Apollo 而言，是否分离伴奏与人声是一个关键但未回答的问题。
【SNR/MOS 具体工具与阈值】未披露（如是否用 DNSMOS、UTMOS、Brouhaha 等）。[不确定]

### 语音/音效/音乐的分类与分别处理策略 ⚠️

Apollo 把「按音频类型分类并分别处理」提升为 pipeline 的一级阶段（Section 4.2 Audio-Guided Data Splitting），这是其数据方法论中最具辨识度的设计：
【两级树状分类】
- 第一级：vocal（含人声）vs non-vocal（不含人声）→ 后者构成 sound split（自然声/音效子集）。
- 第二级：从 vocal 子集再分为 singing（歌唱）/ single-speaker speech（单说话人语音）/ multi-speaker speech（多说话人语音）三支。
【分类器】论文未说明用什么模型做人声/非人声判别与说话人数判别（对比 MOVA 明确使用 EAT 自监督音频 Transformer + Silero VAD）。推测可用 VAD + 说话人分离（diarization）+ 歌唱检测组合实现，但未证实。[不确定]
【差异化处理策略】
- 语音与歌唱子集：做逐字转写 + 说话人属性抽取（性别、年龄）+ 音频 caption + 视频 caption。
- sound split：只做音频 caption，不做转写、不做说话人属性（「the sound split receives only audio captions」）。
即标注成本按子集的信息结构定制投放，避免对无人声片段做无意义的 ASR。
【音乐的位置】论文的四类枚举中没有单列「音乐（music/instrumental）」——纯器乐内容归入 sound split 还是被过滤掉，未说明；含唱的音乐归入 singing。这与 MOVA 在音频塔预训练阶段专门引入 JamendoMaxCaps 音乐语料的做法不同。
【下游用途】该分流直接服务于多任务训练：sound split 主要喂养通用 T2A/foley 能力，speech/singing split 主要喂养 TTS 与唇同步能力，评测也相应分为 audio、TTS、audio-video consistency 等组（Figure 5）。

## 训练配合

### 多阶段训练课程与数据课程调度（阶段划分依据：分辨率/时长/质量分/模态；低清→高清、图像→视频、短→长） ⚠️

采用「渐进式多任务训练（Progressive Multi-Task Training）」三阶段课程，但课程的划分依据是**任务与质量**，而非常见的分辨率/时长维度：
【Stage I — 预训练（Pre-training）】在大规模多场景语料上训练，目标是「acquire atomic generation capabilities across all tasks」——习得全部任务的原子能力，包括跨模态语义对齐、时序同步、高保真音频合成、精确视觉特征构建四项。
【Stage II — 专项后训练（Specialized Post-training）】「Guided by evaluation metrics, we adaptively rebalance data distributions across scenarios and tasks to strengthen underperforming capabilities while preserving overall competency」——由评测指标驱动，自适应地在场景与任务之间重新平衡数据配比，补强短板能力同时不牺牲整体能力。这是一种指标反馈驱动的动态数据课程，比固定配比更贴近工业实践。
【Stage III — 质量精修后训练（Quality-Refined Post-training）】在人工精选的高质量数据集上微调，提升生成保真度与复杂场景鲁棒性。
【正交的任务课程】与三阶段并行的是多任务掩码机制：通过随机模态掩码（random modality masking）在同一模型内联合优化 T2A、T2V、T2AV、I2V、I2AV 五类任务，论文称之为「from random modality masking to joint optimization across tasks」。
【显著缺失】未采用或未披露分辨率课程（低清→高清）、时长课程（短→长）、图像→视频的常规渐进策略；各阶段的训练步数、epoch 数、数据量、分辨率配置全部未给出。这是与 MOVA（360p→360p→720p，61.5k→37.6k→11k 小时，逐阶段耗时天数全列）等开源工作的最大披露差距。[不确定]

### 数据配比随训练阶段的变化（预训练/退火annealing/SFT高质量子集） ⚠️

【机制层面】明确存在阶段性数据配比调整，且是**指标驱动的自适应再平衡**：Stage II 的核心动作就是「adaptively rebalance data distributions across scenarios and tasks」——依据评测指标在场景（单说话人/多说话人/歌唱/自然声）与任务（T2A/T2V/T2AV/I2V/I2AV）两个轴上动态调整配比，哪个能力弱就加哪类数据，同时约束不损伤已有能力。这一「以评测为反馈信号闭环调数据配比」的做法，是 Apollo 训练配合部分最有价值的方法论表述。
【退火/精选层面】Stage III 相当于退火（annealing）阶段，把数据全面切换到人工精选的高质量子集。
【数字层面】全部缺失：各阶段的数据量、四类音频场景的配比数字、五类任务的采样权重、掩码策略的概率分布、再平衡的调整幅度与轮次，一概未披露。Figure 5「Ablations of different training stages」以图形展示了逐阶段在 video/audio/TTS/音视频一致性四组指标上的改善，但未标注对应的数据配比。[不确定]（机制明确，数字全缺）

### 后训练数据（SFT精选集规模与筛选标准、偏好对数量与标注方式、reward model训练数据） ⚠️

【两段式后训练】Apollo 的后训练分为 Stage II（专项后训练，指标驱动的数据再平衡）与 Stage III（质量精修后训练，人工精选高质量数据微调）。
【SFT 精选集】Stage III 使用「manually-curated, high-quality dataset」——论文对其规模、人工筛选标准、来源、与 81M 主集的关系（是子集还是独立采集）全部未说明。这是本条目最遗憾的空白之一：一个明确承认「小而精数据用于末端提纯」的设计，却没有给出任何可对标的数字。
【偏好数据 / RLHF】论文未使用也未提及任何基于偏好的优化：无 DPO、无 RLHF、无 reward model、无偏好对数据构造、无 rejection sampling。训练目标自始至终是 flow matching 加多任务掩码。这与同期部分工作（引入人类偏好对齐提升美学与指令遵循）路线不同，也意味着 Apollo 的对齐能力完全来自数据质量与架构，而非偏好优化。
[不确定]

### 数据处理基础设施与吞吐（NeMo Curator/Data-Juicer/自研，GPU加速比，处理规模，成本） ⚠️

论文完全未披露数据处理基础设施与吞吐信息：未提及 Ray、Spark、NeMo Curator、Data-Juicer 等分布式数据处理框架，未给出 GPU/NPU 数量与型号，未给出加速比、处理吞吐（样本/小时）、总处理耗时或成本估算。训练侧同样空白——未给出训练 GPU 数量、训练框架（是否 Megatron/FSDP/DeepSpeed）、并行策略（是否 context parallelism）、训练时长。考虑到 pipeline 中对 8100 万条最终样本（推算约 3 亿条候选）批量调用 Whisper-Large-v3、Qwen2.5-Omni、Gemini 2.5-Pro 等大模型，其标注算力开销必然极为可观，但论文对此毫无交代。作为闭源工业模型，基础设施细节属于典型的不披露项。[不确定]

## 效果对比

### 数据策略消融的量化影响（区分：过滤严格度ablation / caption密度风格ablation / 数据配比ablation，及对应评测指标） ⚠️

论文的消融实验集中在**架构与训练策略**，**没有任何针对数据策略的消融**——这是评估 Apollo 数据方法论有效性时的关键局限：
【已有消融 1 — 架构（Table 2）】「Comparison of different methods. The Dual Tower uses standard cross-attention, while the Single Tower utilizes our proposed Omni-Full Attention.」对比双塔 cross-attention 与单塔 Omni-Full Attention，支持单塔方案。
【已有消融 2 — 多任务掩码（Table 3）】「Ablation of multi-task masking」，三档递进：
  - 仅 T2V：Video ID 0.71，其余指标不适用
  - T2V + T2AV：Video ID 0.76、Audio MOS 88.181、CLAP 0.188、WER 0.044、DeSync 0.895、Sync-conf 5.024、IB 0.201
  - 全任务（Ours）：Video ID 0.80、Audio MOS 93.106、CLAP 0.232、WER 0.028、DeSync 0.650、Sync-conf 6.787、IB 0.316
  结论：任务越全，所有指标一致改善——视频身份一致性 +0.09、音频 MOS +4.9、CLAP +0.044、WER 相对下降 36%、DeSync 相对下降 27%、唇同步置信度 +1.76、跨模态对齐 +0.115。论文强调「for T2AV joint generation, our multi-task model significantly outperfords a counterpart trained solely on T2AV」，即多任务联合训练对联合生成本身有正向增益，反驳了「多任务会稀释单任务能力」的直觉。
【已有消融 3 — 训练阶段（Figure 5）】「Ablations of different training stages. Metrics include video, audio, TTS, and audio-video consistency」，展示三阶段课程的逐级改善。由于 Stage II/III 的差异本质上就是数据配比与数据质量的差异，这个消融**间接**构成了数据策略的证据链——它证明「指标驱动再平衡 + 人工精选精修」这条数据路径有效，但由于图中未标注配比数字、未做变量隔离（阶段推进同时叠加了更多训练步数），无法把增益归因到数据本身。
【完全缺失】过滤严格度消融（如 27% 保留率 vs 50% vs 80% 的效果对比，这本可以是 Apollo 最有价值的贡献）、caption 密度/风格消融（dense vs 短 caption）、数据配比消融（四类音频场景不同比例）、数据规模消融（81M vs 更小子集）——一个都没有。论文把「27% 保留率」作为事实陈述而非可验证的设计选择呈现。[不确定]

### 质量vs数量的证据（小而精数据超越大而杂的案例） ⚠️

论文未提供任何「小而精超越大而杂」的直接量化证据，但其整体设计隐含了强烈的质量优先倾向，可作为间接立场证据：
【立场证据 1】端到端只保留 27% 的激进过滤率——愿意丢弃近四分之三的候选数据以换取「high-quality, strictly aligned audio–video–caption triplets」（高质量、严格对齐的音视频-caption 三元组），这是一次明确的质量换数量的取舍。
【立场证据 2】Stage III 专设「质量精修后训练」，在末端用人工精选的小规模高质量集做微调，是典型的「大规模学能力、小规模提质感」金字塔结构。
【立场证据 3】在 81M 量级上仍不惜调用 Gemini 2.5-Pro 这类商用旗舰模型做标注，是标注质量优先于成本的信号。
【关键缺口】上述三点都是设计选择而非实验证据——论文没有做「同等算力下，用 27% 精选数据 vs 用全部 100% 数据」的对照实验，因此无法证明 27% 这个 cutoff 是最优的，也无法量化过滤带来的收益。相较之下，MOVA 至少给出了 360p→720p 课程的对照数值。Apollo 在这一维度提供的是**姿态**而非**证据**。[不确定]

### 训练数据domain分布与评测基准类目体系的对齐关系（如VABench七大类） ⚠️

【所用基准】沿用第三方基准 Verse-Bench 评测 T2AV 任务（论文原文：「Following Universe-1, we use Verse-Bench for T2AV tasks」），未自建评测基准。Verse-Bench 是 Universe-1 提出的音视频联合生成评测基准。
【评测指标体系】视频质量用 Motion Score（RAFT 光流）与 Aesthetic Score（MANIQA、aesthetic-predictor-v2-5、Musiq）；音频质量用 FD（PANNs mel 谱 Fréchet 距离）、KL 散度、MOS；同步用 AV-A / DeSync（Synchformer）与 SyncNet Confidence；跨模态语义用 CLAP 与 ImageBind score；语音内容用 WER。Table 1 为「Main comparisons of audio-visual joint generation」，对比级联方案与联合生成方案共 10 项指标，论文称达到与 Veo 3 可比的水平。
【与训练数据分布的对齐关系】存在清晰但未被论文点明的对应：Apollo 的四类训练数据切分（单说话人语音 / 多说话人语音 / 歌唱 / 自然声）与其评测的四组能力（video / audio / TTS / audio-video consistency，见 Figure 5）之间是「数据类目 → 能力类目 → 指标组」的映射链——sound split 对应 audio 组指标（FD/KL/CLAP），speech 与 singing split 对应 TTS 组指标（WER/Sync-conf），全体对应 consistency 组（DeSync/IB）。Stage II 的指标驱动再平衡机制正是建立在这条映射链之上：某组指标弱 → 加对应类目数据。这实际上是本调研中较少见的「数据类目体系与评测类目体系显式闭环」案例。
【缺口】论文未给出 Verse-Bench 的类目枚举（对比 VABench 七大类），也未逐类目报告分数，因此无法核验训练数据分布与基准类目在覆盖面上是否匹配、是否存在类目盲区。[不确定]

## 不确定字段

以下字段的调研信息部分不确定（⚠️ 标注来源）：

- organization
- data_scale
- data_sources
- provenance_licensing
- duration_distribution
- resolution_aspect_distribution
- domain_distribution
- audio_category_distribution
- narrative_structure_distribution
- language_accent_distribution
- shot_segmentation
- quality_filtering
- motion_filtering
- deduplication
- model_as_data_judge
- safety_filtering
- caption_structure
- joint_av_caption_schema
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
- data_ablation
- quality_over_quantity_evidence
- benchmark_taxonomy_alignment
