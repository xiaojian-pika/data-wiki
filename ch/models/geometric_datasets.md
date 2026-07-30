# 几何/结构化标注数据集合集（SceneScribe-1M / SpatialVID / WildWorld / Action100M）

> 主题: 视频生成模型（含音视频同时生成）的数据处理：数据清洗流程、数据分布、打标方式

[← 返回首页](../index.md)

**目录**: [基本信息](#基本信息) · [数据规模与分布](#数据规模与分布) · [清洗流程](#清洗流程) · [打标方式](#打标方式) · [音视频对齐](#音视频对齐) · [训练配合](#训练配合) · [效果对比](#效果对比)

## 基本信息

### 名称

几何/结构化标注数据集合集（SceneScribe-1M / SpatialVID / WildWorld / Action100M）

### 发布机构/公司

多家机构：SceneScribe-1M——上海交通大学 + 蚂蚁集团 + 牛津大学视觉几何组(VGG) + 东方理工宁波数字孪生研究院（浙江省工业智能与数字孪生重点实验室）；SpatialVID——南京大学（NJU-3DV）+ 中国科学院自动化研究所；WildWorld——盛大AI（ShandaAI）+ 上海人工智能实验室 + 北京理工大学（作者含 Zhen Li、Kaipeng Zhang 等）；Action100M——Meta AI（FAIR，代码托管于 facebookresearch）+ 香港科技大学（Pascale Fung 团队）+ 阿姆斯特丹大学 + 索邦大学

### 发布时间（技术报告/论文/开源时间）

SceneScribe-1M：arXiv 2604.07990，2026年4月9日提交、4月26日修订，已被 CVPR 2026 接收；SpatialVID：arXiv 2509.09676，2025年9月11日 v1、2025年12月18日 v2；WildWorld：arXiv 2603.23497，2026年3月24日提交；Action100M：arXiv 2601.10592，2026年1月15日提交

### 类型（模型/数据集/工具链/评测基准）

数据集（四者均为大规模视频数据集，附带自建标注流水线；WildWorld 同时发布 WildBench 评测基准，Action100M 同时发布 VL-JEPA 预训练模型，SpatialVID 发布完整标注 pipeline 代码）

### 开源程度（权重/代码/数据/pipeline各自是否开源） ⚠️

均为学术开源数据集，权重非主要产物。SceneScribe-1M：论文明确表述开源，项目页 https://wangyunnan.github.io/SceneScribe-1M ，数据/标注对外发布，具体许可以官方仓库为准[不确定]；SpatialVID：开源程度最高，HuggingFace 发布 SpatialVID（271万clip）与 SpatialVID-HQ（37万clip）两个版本，约3.53TB，采用 CC-BY-NC-SA 4.0 许可（仅限非商用），标注流水线代码同步开源；WildWorld：GitHub 仓库 https://github.com/ShandaAI/WildWorld ，论文与基准开放，数据源自商业游戏《怪物猎人：荒野》，再分发条款未明确说明[不确定]；Action100M：GitHub 仓库 https://github.com/facebookresearch/Action100M ，标注体量约205GB（仅标注，视频依赖 HowTo100M 原始源），标注开源

### 是否支持音视频同时生成，以及实现方式（原生联合/级联/MoE融合）

均不支持音视频同时生成，也不涉及音频模态。四者是纯视觉几何/结构化标注范式：SceneScribe-1M 与 SpatialVID 面向相机可控视频生成与3D/4D感知，WildWorld 面向动作条件世界模型，Action100M 面向动作理解与视频-文本表征学习。Action100M 源自 HowTo100M（含旁白ASR），但ASR仅用于文本监督辅助，不做音频生成

### 调研信息来源列表（论文/技术报告/官方文档/新闻的URL，标注每条来源的性质：官方一手/同团队旁证/第三方报道）

- url: https://arxiv.org/abs/2604.07990 | title: SceneScribe-1M: A Large-Scale Video Dataset with Comprehensive Geometric and Semantic Annotations（CVPR 2026接收） | nature: 官方一手（论文摘要页）
- url: https://arxiv.org/html/2604.07990v2 | title: SceneScribe-1M 论文全文HTML版（含数据源、过滤规格、MegaSaM/TAPIP3D 标注方法、六项下游消融表） | nature: 官方一手（论文正文）
- url: https://wangyunnan.github.io/SceneScribe-1M | title: SceneScribe-1M 官方项目主页 | nature: 官方一手（项目页）
- url: https://arxiv.org/abs/2509.09676 | title: SpatialVID: A Large-Scale Video Dataset with Spatial Annotations（arXiv v1 2025-09-11, v2 2025-12-18） | nature: 官方一手（论文摘要页）
- url: https://arxiv.org/html/2509.09676v2 | title: SpatialVID 论文全文HTML版（含四维过滤阈值、MegaSaM+UniDepth v2+Depth Anything v2、SAM2动态掩膜、双阶段caption、HQ子集统计） | nature: 官方一手（论文正文）
- url: https://huggingface.co/datasets/SpatialVID/SpatialVID-HQ | title: SpatialVID-HQ HuggingFace 数据集页（CC-BY-NC-SA 4.0，3.53TB，74分组，标注字段清单） | nature: 官方一手（数据集发布页）
- url: https://arxiv.org/abs/2603.23497 | title: WildWorld: A Large-Scale Dataset for Dynamic World Modeling with Actions and Explicit State toward Generative ARPG | nature: 官方一手（论文摘要页）
- url: https://arxiv.org/html/2603.23497v1 | title: WildWorld 论文全文HTML版（含采集平台、119列逐帧标注、动作三元组统计、WildBench四维评测、五种条件方法消融） | nature: 官方一手（论文正文）
- url: https://github.com/ShandaAI/WildWorld | title: WildWorld 官方 GitHub 仓库（盛大AI） | nature: 官方一手（代码仓库）
- url: https://arxiv.org/abs/2601.10592 | title: Action100M: A Large-scale Video Action Dataset | nature: 官方一手（论文摘要页）
- url: https://arxiv.org/html/2601.10592v1 | title: Action100M 论文全文HTML版（含V-JEPA 2分割、Tree-of-Captions五字段、三模型caption链、GPT-OSS-120B Self-Refine、GPU小时成本、16个下游基准） | nature: 官方一手（论文正文）
- url: https://github.com/facebookresearch/Action100M | title: Action100M 官方 GitHub 仓库（Meta FAIR） | nature: 官方一手（代码仓库）

## 数据规模与分布

### 训练数据量级（视频条数/小时数/token数，预训练与SFT分开）

SceneScribe-1M：约100万条视频clip，156.7M帧，总时长4000+小时；SpatialVID：原始33,443条YouTube长视频（21,789小时），产出271万clip、127.60M帧、7,089小时动态内容，高质量子集 SpatialVID-HQ 为0.37M clip、20.63M帧；WildWorld：超过1.08亿帧，约1,800小时（30FPS折算），每帧119列标注；Action100M：源自120万条（1,199,096）教学视频、约14.6年时长（其中72%即10.6年有可用ASR），产出1.47亿个时序定位片段、约213亿英文词标注。四者均为单一数据集，不区分预训练与SFT划分（Action100M 提供语义重采样子集用于缓解长尾）

### 数据来源构成（自有/公开数据集/网络爬取/授权采购/合成数据）

SceneScribe-1M：混合复用已有公开数据集 + 网络素材——HD-VILA-100M（多类目大规模视频）、Panda-70M（视频-caption对）、Koala-36M（精确时序切分）、Pexels-Video（通过 OpenVideo 工具箱采集的66.8万条高质量Pexels视频）；SpatialVID：自行爬取 YouTube，以运动相关关键词检索（walk / tour / drone 等）以保证相机轨迹多样性；WildWorld：完全合成/引擎采集——自建游戏数据采集平台，从 AAA 写实动作角色扮演游戏《Monster Hunter: Wilds》引擎实时录制，非爬取非真人拍摄；Action100M：复用 HowTo100M（YouTube 教学视频，已做人脸模糊处理的1,199,096条版本），属公开数据集二次标注

### 数据合规与溯源（授权数据占比、rights-cleared数据集、C2PA等） ⚠️

SpatialVID 合规性最明确：整库采用 CC-BY-NC-SA 4.0 非商用许可，人工初筛阶段剔除标题含不当词汇的视频，但 YouTube 素材本身的版权授权状态未逐条清理[不确定]；SceneScribe-1M 依赖上游数据集（HD-VILA/Panda-70M/Koala-36M）各自的许可继承，Pexels 部分为可免费商用素材，整体授权占比未披露[不确定]；WildWorld 数据由游戏引擎生成，不涉及真人肖像与网络版权，但游戏内容本身的著作权归发行商，学术再分发条款论文未明确[不确定]；Action100M 只发布标注（约205GB），视频需用户自行从 HowTo100M 获取，规避了视频再分发风险，源视频已做人脸模糊以保护隐私。四者均未提及 C2PA 等内容溯源水印机制

### 片段时长分布与切分策略 ⚠️

SceneScribe-1M：规格过滤要求时长在5秒至1分钟之间，对判定为“非连续”的视频用 TransNetV2 做镜头切分后再次过滤，最终clip平均约14.4秒（156.7M帧/100万clip，按fps折算的近似值）[不确定]；SpatialVID：用改造版 PySceneDetect 切分为 3–15 秒 clip，平均约9.4秒（127.6M帧/2.71M clip @约30fps推算）；WildWorld：训练样本固定为81帧片段（30FPS录制、推理按16FPS），按动作序列组织；Action100M：分层时序分割，丢弃短于0.5秒的片段，LLM 聚合阶段进一步剔除短于4秒的节点，形成从秒级到分钟级的多层级时间树

### 分辨率/宽高比分布与分桶策略

SceneScribe-1M：规格过滤要求分辨率高于1080p、帧率≥10fps，以保留细粒度几何细节；SpatialVID：所有clip统一标准化为 1280×720、H.265 编码 MP4；WildWorld：2K全屏录制、子窗口720p，模型训练分辨率 544×960（约9:16竖向裁切比例的变体）；Action100M：沿用 HowTo100M 原始分辨率，无统一分桶策略，视觉编码走 V-JEPA 2 固定采样窗口。四者均未采用视频生成模型常见的多分辨率/宽高比分桶训练策略——因其定位是标注数据集而非训练配方

### 类别/domain分布与配比策略（人物、动作、场景、风格等比例控制与概念均衡）

SceneScribe-1M：源自 HD-VILA-100M/Panda-70M/Koala-36M 的开放域网络视频，覆盖多类目日常场景，未做显式类目配比控制，选择偏好由“运动多样性”驱动（初筛因运动多样性要求大幅缩减源池）；SpatialVID：以运动关键词主动构造 domain（步行第一视角、城市/自然游览、无人机航拍），并对 SpatialVID-HQ 做类别分布与运动多样性的显式均衡采样，结构化标签体系覆盖天气、时段、人群密度、光照、场景类型五个维度，实现可查询的 domain 控制；对比 Panda-70M 显示 SpatialVID-HQ 在美学、亮度、运动三项指标上分布更集中，80%的clip具备弯曲或转向轨迹；WildWorld：domain 高度受控但狭窄，全部为单一游戏世界的战斗/探索场景，测试集 WildBench 按场景类型显式配比——100条协作场景（玩家+3 NPC 对战怪物）与100条单挑场景，覆盖多角色、多武器、多怪物物种、多难度与多事件（技能释放、击倒、死亡、暴击）；Action100M：教学视频域（HowTo100M），动作类目开放词表、超长尾，作者用 k-means 语义重采样（k=10³/10⁴/10⁵）缓解长尾，并统计出7.58M个重复组、共1.418亿条重复实例需去重后再重采样

### 音频类别分布与配比（语音/音效foley/音乐/环境音/静音的比例及控制策略）——AV模型独有维度

不适用。四个数据集均不涉及音频模态标注，无语音/音效/音乐/环境音的类别配比。Action100M 唯一间接触及音频——其源自 HowTo100M 的旁白 ASR 转写（72%视频有可用ASR，覆盖10.6年内容），但仅作为文本弱监督线索参与，未按音频类别做分布统计或配比控制

### 叙事结构分布（单镜头vs多镜头、平均clip时长、镜头数分布、是否含原生音轨）

SceneScribe-1M：显式追求单镜头连续性——先用 TransNetV2 检测硬切与渐变转场，将“非连续”视频拆分为语义连贯的单镜头clip并重新过滤，最终库为纯单镜头；SpatialVID：改造 PySceneDetect（调整敏感度阈值 + 基于间隔的多帧比较策略）确保 3–15 秒clip内无转场，同样是单镜头库，但强调镜头内的相机轨迹复杂度（80%含弯曲/转向轨迹）；WildWorld：游戏内连续录制，天然单镜头，按动作序列（action-sequence）与样本两级组织，叙事单位是战斗回合；Action100M：不追求单镜头，而是构建“镜头之上”的层级叙事树——用 Ward linkage 层次聚类把长视频组织为多层级时间树（Tree-of-Captions），同时保留帧级、片段级与聚合级三层语义。四者均无原生音轨相关的叙事标注

### 语言/口音分布（多语种唇同步能力的数据基础）

SceneScribe-1M / SpatialVID / WildWorld：caption 均为单一英文，无多语种或口音维度，无说话人；Action100M：标注总量213亿词全部为英文，源自 HowTo100M 的英文教学视频子集（ASR为英文），未做语种与口音分布统计。四者均不具备多语种唇同步的数据基础

## 清洗流程

### 清洗漏斗整体结构（几级过滤、各级顺序）

SceneScribe-1M（三级漏斗+标注）：①规格过滤（分辨率>1080p、帧率≥10fps、时长5秒–1分钟）→②Qwen2.5-VL-72B 六维内容质量过滤（运动强度未知、可见水印、强相机畸变、强光照伪影等剔除）→③TransNetV2 时序切分（仅对非连续视频）并对切出的clip重新过滤→④几何标注（MegaSaM 相机+深度、TAPIP3D 3D点轨迹）+语义标注（Qwen2.5-VL-72B caption）；SpatialVID（人工初筛→切分→四维质量过滤→几何重建→双阶段caption）：①人工筛选（剔除损坏内容、标题含不当词或“全景相机”字样、与 MegaSaM 重建不兼容者）→②改造 PySceneDetect 切 3–15 秒clip并统一转码 1280×720 H.265→③四项质量过滤（美学、亮度、OCR文本占比、运动/VMAF）→④MegaSaM 相机位姿+深度重建（深度模块替换为 UniDepth v2 与 Depth Anything v2）→⑤SAM2 动态掩膜与动态比例计算→⑥轨迹运动指标（MoveDist/RotAngle/TrajTurns）与加速度异常检测→⑦Gemini-2.0-Flash 视觉解析 + Qwen3-30B-A3B 语言精修的双阶段结构化caption→⑧HQ子集均衡采样；WildWorld（引擎直采，无传统清洗漏斗）：基于 OBS Studio + Reshade 的采集平台将显示分区为子窗口，同步时间戳同时录制 RGB(720p) 与深度，并从游戏内存/引擎导出骨骼、世界状态、相机内外参，逐帧119列结构化标注，再用VLM生成两级caption；Action100M（标注型流水线，几乎不做视频过滤）：①V-JEPA 2 编码器把视频转为时序稠密视觉嵌入（每4帧采1帧，64帧重叠窗口、步长8）→②带时序约束的层次凝聚聚类（Ward linkage）生成时间树→③Llama-3.2-Vision-11B 帧级caption + Perception-LM-3B 片段级caption→④GPT-OSS-120B 做证据聚合与结构化抽取，三轮 Self-Refine→⑤去重与语义重采样

### 漏斗定量保留率（每级过滤的输入/输出量与最终保留率，如Apollo 27%） ⚠️

SpatialVID 给出最清晰的量化漏斗：21,789小时原始 YouTube 视频（33,443条）→ 最终 7,089 小时动态内容（271万clip），时长维度保留率约 32.5%；高质量子集 SpatialVID-HQ 为 0.37M/2.71M clip，即在已过滤库上再保留约 13.7%（相对原始素材端到端约 4.5%）。关键量化发现：Panda-70M 中超过 80% 的视频因运动不足而无法被 MegaSaM 成功重建，构成运动过滤的主要淘汰源。SceneScribe-1M 各级输入输出量未逐级披露，仅知从 HD-VILA-100M/Panda-70M/Koala-36M/Pexels 的亿级源池收敛至100万clip、156.7M帧、4000+小时，论文表述“因运动多样性要求初筛大幅缩减源池”，逐级保留率数值缺失[不确定]；Action100M 的量化在去重侧：识别出 7.58M 个重复组、1.418亿条重复实例并去除，源120万视频中72%有可用ASR，最终产出1.47亿片段；WildWorld 为引擎直采，无过滤漏斗概念，仅提及过滤后得到数千条clip（确切数量未给出）[不确定]

### 镜头切分方法（PySceneDetect/自研模型/shot-aware splitting）

SceneScribe-1M：TransNetV2 深度模型检测硬切与渐变转场，仅对判定为“非连续”的视频执行切分，切后clip重新过滤以保证语义连贯；SpatialVID：改造版 PySceneDetect——调整敏感度阈值并引入基于间隔的多帧比较策略（interval-based multi-frame comparison），弥补原版对渐变转场的漏检，输出 3–15 秒clip；WildWorld：无需镜头切分，游戏引擎连续录制，按动作序列边界组织样本（训练取81帧窗口）；Action100M：不用传统镜头检测，改用 V-JEPA 2 语义嵌入 + Ward linkage 带时序约束的层次凝聚聚类做语义级时序分割，可产出跨镜头的层级片段树，丢弃<0.5秒片段

### 质量过滤（美学评分、清晰度、OCR文字过滤、黑边/水印/logo检测）

SceneScribe-1M：核心是 Qwen2.5-VL-72B 的六维内容质量评估，显式剔除运动强度不明、可见水印、强相机畸变、强光照伪影等clip；叠加硬性规格门槛（>1080p、≥10fps、5s–60s）。SpatialVID：四项独立打分器构成质量层——①美学：CLIP+MLP aesthetic predictor，分数 < 4.0 剔除；②亮度：仅保留均值落在 [20, 140] 区间的clip，过暗过曝剔除；③OCR文字：PaddleOCR 检测，文字面积占比 > 30% 的clip被标记剔除；④运动：VMAF 指标 + 后续重建可行性判定。WildWorld：引擎渲染画面天然无水印、无压缩伪影、无黑边，质量过滤退化为场景有效性筛选，主要保证录制同步与深度无损编码。Action100M：几乎不做画质过滤（沿用 HowTo100M 原始画质），质量控制转移到标注侧——通过 LLM 多轮 Self-Refine 保证标注质量，并按时长下限（片段>0.5秒、树节点>4秒）剔除无意义碎片

### 运动过滤（光流/运动分数阈值、静态与抖动剔除）

运动过滤是这四个数据集区别于常规视频生成数据集的关键点——常规pipeline剔除过大运动，几何标注数据集反而剔除“运动不足”者。SpatialVID：以 VMAF 结合三项相机轨迹度量做筛选——MoveDist（总位移距离）、RotAngle（累计旋转角）、TrajTurns（方向变化次数），并用基于加速度的检测器识别突兀、非物理的运动抖动予以剔除；量化证据是 Panda-70M 中 80%+ 视频因视差/运动不足无法被 MegaSaM 重建，故必须主动检索 walk/tour/drone 类高运动素材；最终库中80%的clip具有弯曲或转向轨迹。SceneScribe-1M：以“运动多样性”为初筛主轴，Qwen2.5-VL-72B 判定运动强度未知者直接剔除，并因运动要求大幅缩减源池；选用 MegaSaM 的理由正是其在相机视差受限场景下仍稳健。WildWorld：动作由游戏输入直接给定，无需从像素反推运动，静态帧通过动作ID可精确识别。Action100M：无光流层面的运动过滤，语义分割阶段的方差最小化聚类天然把静止段落合并为长节点

### 去重方法（精确去重与基于embedding的语义去重分别记录） ⚠️

Action100M 的去重最系统且量化：在标注聚合后识别出 7.58M 个重复组、共 1.418亿条重复实例并执行去重，随后用 k-means（k=10³/10⁴/10⁵）做语义重采样以压平长尾——属于基于语义嵌入的去重+再平衡组合。SceneScribe-1M：源自 HD-VILA-100M/Panda-70M/Koala-36M/Pexels 四个库的合并，存在跨库重叠风险，论文未描述显式去重步骤[不确定]。SpatialVID：以视频ID为单位从 YouTube 采集，同一长视频切出的clip间存在天然近重复，论文未描述embedding级语义去重[不确定]。WildWorld：动作空间长尾明显（top-150 动作占 58.49% 样本），论文以动作三元组统计呈现分布而非去重，未见显式去重流程[不确定]

### VLM/LLM作为质检员（多模态大模型质量打分与错配剔除，2026年从浅层打分器转向大模型语义判断的趋势）

这批数据集充分体现了“大模型作质检员”的2026年趋势。SceneScribe-1M 最典型：直接用 Qwen2.5-VL-72B 作为唯一的内容质量判官，一次性完成六个维度（运动强度、水印、相机畸变、光照伪影等）的语义判断，取代了传统的浅层美学/清晰度打分器组合。SpatialVID 采取混合策略：浅层打分器（CLIP+MLP美学、亮度统计、PaddleOCR、VMAF）做粗筛，把大模型算力留给标注侧——Gemini-2.0-Flash 做1fps视觉解析、Qwen3-30B-A3B 结合相机位姿先验精修并纠正运动方向描述，本质是“用几何真值校验VLM输出”的反向质检，可修正VLM常见的左右/推拉方向幻觉。Action100M 把LLM质检推向极致：GPT-OSS-120B 对多来源caption做证据聚合与结构化抽取，并执行三轮 Self-Refine 自我修正，等于用推理模型充当标注一致性审核员。WildWorld 用VLM做评测端的judge：WildBench 的 Action Following 指标由VLM判定生成片段与真值片段是否一致（0/1打分），与人工评价达成85%一致率——是模型即评委在几何数据集上的显式效度验证

### 安全与合规过滤（NSFW、版权、人脸/隐私） ⚠️

SpatialVID 有显式的安全人工介入：初筛阶段人工审阅并剔除标题含不当词汇（inappropriate terms）的视频，同时剔除“全景相机”类不兼容内容；许可采用 CC-BY-NC-SA 4.0 限制商用。Action100M 的隐私保护来自上游——使用的是已做人脸模糊（face-blurred）处理的 HowTo100M 1,199,096 条版本，且只发布标注不发布视频，进一步降低版权与隐私暴露。WildWorld 天然无隐私风险，画面全部为游戏引擎渲染的虚拟角色与怪物，无真实人脸。SceneScribe-1M 未描述独立的 NSFW/人脸/版权过滤模块，安全性依赖上游数据集（HD-VILA、Panda-70M、Koala-36M）已有的清洗[不确定]

## 打标方式

### 使用的caption模型（自研VLM/开源模型，模型规模） ⚠️

SceneScribe-1M：Qwen2.5-VL-72B（开源VLM，72B规模），利用其动态分辨率处理与绝对时间编码能力处理长视频，同一模型同时承担质量过滤与caption生成双重角色；SpatialVID：双模型级联——Gemini-2.0-Flash（闭源商用API）对1fps采样帧做视觉解析产出初稿，Qwen3-30B-A3B（开源MoE，30B总参/3B激活）做语言精修，并注入相机位姿先验纠正运动方向；WildWorld：用视觉语言模型生成动作序列级与样本级两层caption，具体模型型号论文未指明[不确定]；Action100M：三模型分工——Llama-3.2-Vision-11B 负责帧级（中间帧）caption、Perception-LM-3B（PLM-3B）负责视频片段级caption、GPT-OSS-120B（120B推理模型）负责证据聚合与结构化字段抽取，总标注算力约130万 V100 GPU小时 + 30万 H100/H200 GPU小时

### caption密度与结构化程度（短/长/密集描述、结构化字段如镜头运动、风格标签）

SceneScribe-1M：单条综合性结构化场景描述，显式区分场景设定（scene settings）、主要主体/角色（primary subjects or characters）、关键动作（significant actions）三要素，属中等密度长caption；SpatialVID：多风格并行的结构化caption组，同一clip产出四类文本——运动导向描述（精修后平均50.3词）、场景摘要（平均28.6词）、沉浸式叙事（平均89.7词），外加天气/时段/人群密度/光照/场景类型五类结构化标签，实现了“一视频多粒度多风格”的密度谱；Action100M：Tree-of-Captions 层级结构，每节点五个结构化字段——简要动作描述（平均3.2词）、详细动作描述（平均27.8词）、动作主体识别（actor）、简要caption（平均19.2词）、详细caption（平均95.3词），从3词到95词覆盖完整密度梯度，全库213亿词；WildWorld：caption 分动作序列级与样本级两层，但caption并非主要监督信号，核心监督来自119列结构化数值标注

### 音视频联合caption结构（是否同时覆盖视觉+听觉轨道、是否分流为独立字段，如LTX-2全音景描述、Script-a-Video factorized streams、Foley-Omni三字段）

不适用。四个数据集均为纯视觉，无音轨描述、无视听双流caption schema。与 LTX-2 全音景描述、Script-a-Video 因子化流、Foley-Omni 三字段等音视频联合范式相比，这批数据集代表的是另一个正交方向——把caption之外的第二类监督放在几何与状态维度（相机、深度、3D轨迹、骨骼、世界状态、动作ID）而非听觉维度

### 对白转写与说话人属性标注（ASR转写+说话人身份/语言/口音/情绪）

基本不适用。SceneScribe-1M、SpatialVID、WildWorld 均无对白转写与说话人属性标注。Action100M 唯一相关：源自 HowTo100M 的旁白ASR转写可用于72%的视频（覆盖10.6年内容），但论文明确其标注范式是“不依赖ASR的纯视觉动作监督”，ASR仅作为对照/辅助线索，且不含说话人身份、语言、口音、情绪等属性字段

### 几何与结构化标注（相机参数、深度、3D point tracks、动作标注、显式状态）

本条目的核心字段。SceneScribe-1M（几何三件套+文本）：①相机参数——MegaSaM 估计，其基于 DROID-SLAM 相机跟踪骨架，在束调整（bundle adjustment）中注入 Depth Anything 与 UniDepth 的深度先验；论文对比评估过 DPVO、Fast3r、MonST3R、VGGT 后选定 MegaSaM，理由是特征点稀缺时更稳健、且对动态场景与低相机视差更有效；②稠密深度图——由 MegaSaM 的束调整层输出时序一致的连续视频深度；③3D点轨迹——TAPIP3D，将2D视频特征投影到3D世界空间以补偿相机运动，迭代精修得到长时程一致的3D point tracks，并可投影回像平面兼容2D跟踪任务；④文本描述。SpatialVID：逐帧相机位姿+内参、深度图（MegaSaM 的深度模块替换为 UniDepth v2 + Depth Anything v2）、SAM2 生成的动态物体掩膜与逐帧动态比例、三项轨迹度量（MoveDist/RotAngle/TrajTurns），以及序列化运动指令——由相邻帧间的相对平移与旋转推导，映射到受控的电影摄影术语词表（dolly in、pan left、truck right 等），把连续位姿离散为可被文本模型消费的指令序列，这是该数据集最独特的设计。WildWorld（引擎真值，精度最高）：逐帧119列标注，含①角色与怪物骨骼——3D关键点与关节树结构；②世界状态——实体位置、旋转、速度、动画ID、血量/耐力等玩法属性；③相机内参与外参；④无损编码深度图；⑤离散动作标签——角色侧5,960个唯一三元组（武器类型、bank ID、motion ID）、455个唯一角色动作，怪物侧2,132个唯一对、527个motion ID，覆盖移动、攻击、闪避、防御、道具使用、过渡动作。Action100M：结构化标注偏语义而非几何——时序定位边界 + 开放词表动作标签 + actor 识别 + 分层caption树，属于“结构化动作标注”而非3D几何标注

### 合成数据构造（受控扰动/编辑构造训练对，如InstructAV2AV）

WildWorld 是纯合成数据构造的代表：全部1.08亿帧由游戏引擎实时渲染产生，配套自建采集平台（OBS Studio + Reshade 分屏子窗口 + 多源时间戳同步）同时录取RGB与深度，动作与状态可被主动控制以构造受控扰动样本——同一场景下改变动作ID即可生成配对的反事实序列，这是真实视频无法提供的监督形式。其余三者均为真实视频的自动标注，不构造合成训练对：SceneScribe-1M 与 SpatialVID 通过几何重建模型生成伪真值（pseudo-GT）标注，属于“模型标注”而非“数据合成”；Action100M 通过VLM伪标注（并与PLM-3B直接伪标注做了对比消融）扩展监督，同样不涉及视频合成

### 人工介入程度（人工标注、人工质检、模型初筛+人工复核） ⚠️

整体人工介入极轻，以自动化为主。SpatialVID 人工介入最多：初筛阶段人工审阅33,443条源视频，剔除损坏内容、标题含不当词或“全景相机”字样、以及与 MegaSaM 重建管线不兼容者；后续标注全自动。WildWorld：人工用于构建评测集——WildBench 的200条样本由人工挑选以覆盖多角色/武器/怪物/难度/事件，并做了人工评价与VLM judge 的一致性验证（85%一致率）；数据采集端由游戏内自动录制。SceneScribe-1M：论文未描述任何人工验证环节，质量控制全部由 Qwen2.5-VL-72B 与 TransNetV2 自动完成[不确定]。Action100M：无显式人工验证，靠 GPT-OSS-120B 三轮 Self-Refine 替代人工复核

## 音视频对齐

### 音视频同步检测方法（口型同步、事件对齐）

不适用。四个数据集均无音轨，不涉及唇形同步或音画事件对齐检测。可类比的“对齐”概念在这批数据集中体现为跨模态几何对齐——WildWorld 用多源时间戳同步保证 RGB 帧、深度帧、骨骼与世界状态在同一时刻严格对齐（采集平台的核心工程挑战即此）；SpatialVID 用相机位姿先验校验并纠正VLM生成的运动方向描述，属于“文本-几何对齐”而非音视频对齐

### 同步检测具体指标与阈值（SyncNet/Synchformer/LSE/自研，阈值数值，如MOVA LSE-D≤9.5且LSE-C≥4.5、SkyReels-V4 SyncNet |offset|≤3∧conf>1.5）

不适用，四者均无 SyncNet / Synchformer / LSE-D / LSE-C 等音画同步指标与阈值。其对应位置的量化阈值出现在几何与画质维度：SpatialVID 的美学分 <4.0 剔除、亮度需落在 [20,140]、OCR文字面积占比 >30% 剔除、片段时长 3–15 秒；SceneScribe-1M 的分辨率 >1080p、帧率 ≥10fps、时长 5–60 秒；Action100M 的片段 >0.5 秒、树节点 >4 秒；WildWorld 的 State Alignment 骨骼关键点像素距离阈值 4/8/16/32 px

### 时序同步vs语义同步的分离处理（时间对齐与内容语义匹配作为两个独立过滤条件）

不适用于音视频语境。在几何标注语境下四者确有类似的“时序 vs 语义”分离思想：SpatialVID 把时序层面的相机轨迹（逐帧位姿、MoveDist/RotAngle/TrajTurns）与语义层面的场景描述（天气/时段/人群/光照/场景类型标签）作为两套独立标注并行维护，并在精修阶段令二者互相校验；Action100M 显式区分时序定位（何时发生，由V-JEPA 2嵌入聚类决定）与语义内容（发生了什么，由VLM caption决定），两条流水线解耦；WildWorld 的 WildBench 也把 Action Following（语义是否执行了正确动作）与 State Alignment（时空状态是否精确对齐）拆为两个独立指标

### 音频质量过滤（SNR、静音检测与静音占比阈值、无音轨剔除、画外音源剔除、背景音乐分离） ⚠️

不适用。四个数据集均不处理音轨，无SNR、静音占比、画外音、背景音乐分离等过滤环节。SpatialVID 转码为 H.265 MP4 时是否保留原始音轨论文未说明[不确定]；Action100M 只发布文本标注，音频由用户自行从 HowTo100M 源获取

### 语音/音效/音乐的分类与分别处理策略

不适用。四个数据集均无语音/音效/音乐的分类与差异化处理策略

## 训练配合

### 多阶段训练课程与数据课程调度（阶段划分依据：分辨率/时长/质量分/模态；低清→高清、图像→视频、短→长） ⚠️

作为数据集本身不定义训练课程，但均提供了支撑课程调度的分层结构。SpatialVID 提供最明确的两级数据课程基础：全量 2.71M clip 可用于大规模预训练，均衡采样的 SpatialVID-HQ（0.37M clip、20.63M帧）作为高质量退火/SFT子集；Action100M 提供语义重采样子集（k-means，k=10³/10⁴/10⁵ 三档粒度），可按需构造从粗到细的课程；SceneScribe-1M 在下游实验中采用“基线模型 + 引入 SceneScribe-1M 继续训练”的两段式配方，用于验证数据增益；WildWorld 用于对 Wan2.2-TI2V-5B 基线做条件微调，训练片段固定81帧、分辨率 544×960、推理16FPS，未描述分辨率/时长渐进课程[不确定]

### 数据配比随训练阶段的变化（预训练/退火annealing/SFT高质量子集） ⚠️

SpatialVID 的 HQ 子集是显式的配比产物——在美学、亮度、运动三项指标上做分布收紧，并对场景类别分布与运动多样性做均衡，相对 Panda-70M 展现出更集中的质量分布，可直接充当退火阶段的高质量配比源；Action100M 的 k-means 语义重采样即为对抗长尾的配比工具，配合7.58M重复组去重使用；SceneScribe-1M 的下游实验统一采用“原配方 vs 原配方+SceneScribe-1M”的加法式混合，未披露具体混合比例[不确定]；WildWorld 未涉及多阶段配比调度[不确定]

### 后训练数据（SFT精选集规模与筛选标准、偏好对数量与标注方式、reward model训练数据） ⚠️

四者均不提供偏好对、reward model 训练数据或RLHF素材。最接近后训练精选集概念的是 SpatialVID-HQ（0.37M clip，筛选标准为美学分、亮度区间、运动轨迹多样性与场景类别均衡）与 WildBench（200条人工挑选样本，但定位是评测集而非训练集，按协作/单挑各100条配比，覆盖多角色、武器、怪物、难度与关键事件）。Action100M 的高置信子集通过多轮 Self-Refine 收敛得到，但未单独发布为SFT集[不确定]

### 数据处理基础设施与吞吐（NeMo Curator/Data-Juicer/自研，GPU加速比，处理规模，成本） ⚠️

Action100M 披露最完整的算力账单：约130万 V100 GPU小时 + 30万 H100/H200 GPU小时完成1.47亿片段的标注，最终标注产物仅约205GB；未使用 NeMo Curator / Data-Juicer 等现成框架，为自研流水线（V-JEPA 2 编码 + 层次聚类 + 三模型caption + LLM聚合）。SpatialVID 自研标注流水线并开源，产物约3.53TB，组织为74个分组（每组约14GB视频 + 1.5GB标注），未披露GPU小时与成本[不确定]；其吞吐瓶颈明确在 MegaSaM 重建环节（论文称选它正是因为精度与效率的平衡）。SceneScribe-1M 的流水线成本未披露[不确定]，主要开销在 Qwen2.5-VL-72B 全库推理与 MegaSaM + TAPIP3D 逐clip重建。WildWorld 的基础设施是自建游戏数据采集平台（OBS Studio + Reshade 分屏、多源时间戳同步、深度无损编码），采集1.08亿帧约1,800小时游戏时长，为实时录制而非离线GPU处理

## 效果对比

### 数据策略消融的量化影响（区分：过滤严格度ablation / caption密度风格ablation / 数据配比ablation，及对应评测指标）

SceneScribe-1M（数据配比消融，统一采用“基线 vs 基线+本数据集”对照，覆盖六项下游任务）：①单目深度估计（MoGe，8个benchmark平均 Rel 指标提升幅度有限，约6.14量级）；②3D重建（VGGT，位姿 AUC15 由 83.4 提升至 83.8）；③4D重建（MonST3R，相机位姿 ATE 由 0.108 降至 0.099）；④2D点跟踪（CoTracker3，δ_avg^vis 由 76.6 提升至 77.4）；⑤3D点跟踪（SpatialTrackerV2，AJ 由 23.25 提升至 23.5）；⑥文本/位姿到视频生成（AC3D，FVD 由 38.20 降至 35.15、CLIP 由 28.62 升至 29.98）——生成任务的增益幅度显著大于感知任务，说明几何标注对相机可控视频生成的边际价值最高。SpatialVID（过滤严格度与分布消融）：与 Panda-70M 的分布对比构成核心证据——SpatialVID-HQ 在美学、亮度、运动三项上分布更紧凑，80%的clip具备弯曲/转向轨迹，而 Panda-70M 以静态内容为主且80%+无法被 MegaSaM 重建，量化说明面向几何用途时“运动过滤严格度”是决定数据可用性的第一因素。WildWorld（标注类型消融，基线 Wan2.2-TI2V-5B）：对比五种条件注入方式——CamCtrl（相机轨迹）、SkelCtrl（骨骼视频）、StateCtrl（离散+连续状态，Transformer实体建模）、StateCtrl-AR（自回归状态预测）；SkelCtrl 在 Action Following 与 State Alignment 上相对基线取得近100%的提升，但牺牲了美学质量，构成典型的“结构化条件 vs 画质”权衡；StateCtrl-AR 展示自回归潜力但存在误差累积。Action100M（caption密度与标注类型消融）：比较简要动作/详细动作/简要caption/详细caption 四类字段各自的监督效力，并与 PLM-3B 直接伪标注做对照，验证层级树+LLM聚合优于单模型伪标；另做 k-means 语义重采样（k=10³/10⁴/10⁵）的长尾缓解消融

### 质量vs数量的证据（小而精数据超越大而杂的案例）

SpatialVID 提供最直接的证据链：主动用运动关键词检索的 21,789 小时定向素材，其可用性远超被动爬取的通用大库——Panda-70M 规模大得多但 80%+ 无法完成几何重建，说明对几何标注任务而言“大而杂”几乎无效；进一步地，0.37M 的 SpatialVID-HQ 子集通过美学/亮度/运动三维收紧与类别均衡，被定位为优于 2.71M 全量的训练选择。SceneScribe-1M 的证据在规模对比表：其100万clip/156.7M帧的体量小于 Koala-36M（3600万视频）与 SpatialVID（约200万clip/123.6M帧），但凭借“相机+深度+3D点轨迹+文本”的完整标注组合，在六项下游任务上均带来增益，体现标注深度对样本数量的替代性。WildWorld 以约1,800小时的单一游戏域数据，凭借帧级119列精确真值，使 SkelCtrl 在动作遵循上近乎翻倍——用标注精度换数据广度的极端案例。Action100M 反向说明规模仍有价值，但其去重（1.418亿重复实例）与语义重采样恰恰是“把大数据变精”的操作

### 训练数据domain分布与评测基准类目体系的对齐关系（如VABench七大类） ⚠️

SceneScribe-1M 的对齐关系最系统：数据标注的四个维度（文本、相机参数、深度、3D点轨迹）与其建立的六项评测任务一一对应——深度标注对应单目深度估计基准、相机参数对应3D/4D重建（VGGT/MonST3R）基准、3D点轨迹对应2D/3D点跟踪（CoTracker3/SpatialTrackerV2）基准、文本+相机对应文本/位姿到视频生成（AC3D）基准，形成“标注即任务”的闭环设计。WildWorld 与 WildBench 的对齐是显式设计的：数据侧的动作ID对应基准的 Action Following 维度、骨骼与世界状态对应 State Alignment 维度（按4/8/16/32像素多阈值评测）、相机内外参对应 ATE/RPE 相机控制维度（用ViPE tracker做SfM）、画质由VBench四项（运动平滑度、动态程度、美学质量、图像质量）覆盖；作者明确指出VBench类通用指标在交互式任务上已“饱和”，正是数据标注体系倒逼评测体系扩展的例证。SpatialVID 的结构化标签体系（天气/时段/人群密度/光照/场景类型）为按类目切分评测提供了基础，但论文未建立与之对齐的固定评测类目体系[不确定]。Action100M 的动作层级标注对应其16个下游基准的两大族——动作识别（SSv2、EPIC-KITCHENS-100、EgoExo4D Keysteps、Kinetics-400、COIN、CrossTask 等8个）与文本-视频检索（MSR-VTT、ActivityNet、DiDeMo、MSVD、YouCook2、PVD-Bench、Dream-1K、VDC-1K 等8个），分别对应动作字段与caption字段的监督效力

## 不确定字段

以下字段的调研信息部分不确定（⚠️ 标注来源）：

- openness
- provenance_licensing
- duration_distribution
- funnel_retention_rate
- deduplication
- safety_filtering
- caption_model
- human_in_loop
- audio_quality_filtering
- multi_stage_curriculum
- stage_data_mixture
- post_training_data
- data_infra_throughput
- benchmark_taxonomy_alignment
