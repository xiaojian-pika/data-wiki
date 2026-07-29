# Vidu S1（技术报告《Vidu S1: A Real-Time Interactive Video Generation Model》，arXiv:2607.03118v2；产品名 Vidu Stream，https://vidu.com/vidu-stream）

> 主题: 视频生成模型（含音视频同时生成）的数据处理：数据清洗流程、数据分布、打标方式

[← 返回首页](../index.md)

**目录**: [基本信息](#基本信息) · [数据规模与分布](#数据规模与分布) · [清洗流程](#清洗流程) · [打标方式](#打标方式) · [音视频对齐](#音视频对齐) · [训练配合](#训练配合) · [效果对比](#效果对比)

## 基本信息

### 名称

Vidu S1（技术报告《Vidu S1: A Real-Time Interactive Video Generation Model》，arXiv:2607.03118v2；产品名 Vidu Stream，https://vidu.com/vidu-stream）

### 发布机构/公司

生数科技（Shengshu Technology）联合清华大学（Tsinghua University）。作者含 Jintao Zhang、Kai Jiang、Jintao Chen 等27人，顾问为 Zhijie Deng、Fan Bao（鲍凡）、Jianfei Chen、Jun Zhu（朱军）

### 发布时间（技术报告/论文/开源时间）

2026年7月：2026年7月6日于全球数字经济大会由生数科技创始人朱军正式发布产品；arXiv 预印本 2607.03118，v2 版本时间为 2026年7月21日（cs.CV，13页）

### 类型（模型/数据集/工具链/评测基准）

模型（实时交互式流式音视频联合生成模型，语音驱动数字人/虚拟角色）。同时自建了配套评测基准 Vidu-StreamBench（500样本，内部基准，未见开源）

### 开源程度（权重/代码/数据/pipeline各自是否开源）

闭源。技术报告仅公开架构与训练框架思路，未开源模型权重、推理/训练代码、数据、数据处理 pipeline；自研加速与服务组件 TurboDiffusion、TurboServe 亦未开源。自建基准 Vidu-StreamBench 未公开发布。仅通过官网 https://vidu.com/vidu-stream 提供可交互在线 demo。论文以 CC-BY 4.0 许可发布。

### 是否支持音视频同时生成，以及实现方式（原生联合/级联/MoE融合）

支持，且为原生联合生成（native joint）。模型将第 i 帧的干净视频表示 v_0^i 与音频表示 a_0^i 沿模态维度拼接为联合状态 x_0^i = [v_0^i; a_0^i]，在同一扩散去噪模型中对视频-音频联合潜在序列统一去噪，非级联、非 MoE。统一条件接口 c 同时包含语音（speech）、文本提示、参考首帧图像——即语音既是条件控制信号（用户实时语音指令控制角色行为），联合状态中又包含音频轨道的生成。整体为自回归 + 扩散（AR+Diffusion）的因果流式生成范式，滑动窗口解码支持无限长生成。

### 调研信息来源列表（论文/技术报告/官方文档/新闻的URL，标注每条来源的性质：官方一手/同团队旁证/第三方报道）

1) 官方一手：arXiv:2607.03118v2《Vidu S1: A Real-Time Interactive Video Generation Model》https://arxiv.org/abs/2607.03118 （含第2.1节 Data Preparation 与 Figure 2 数据过滤流水线图，本调研数据侧信息几乎全部来自此节）
2) 官方一手：产品页与在线 demo https://vidu.com/vidu-stream
3) 第三方报道：中国日报网《生数科技发布 Vidu S1，推动视频生成迈向「实时交互」新时代》https://tech.chinadaily.com.cn/a/202607/06/WS6a4b12eea310d709c2fbbecb.html
4) 第三方报道：雷峰网 https://www.leiphone.com/category/industrynews/6GlFzI5hMwcfRoGZ.html ；爱范儿 https://www.ifanr.com/digest/1670950 （发布时间、AR+Diffusion 架构、540P/25-42FPS 等产品参数）
5) 第三方整理：AI 科技深度解读 Vidu S1 词条 https://www.ai-all.info/ai-models/vidu-s1 （TurboDiffusion/TurboServe、消费级 GPU RTX 3060 起、端到端延迟<200ms 等，属二手信息）

## 数据规模与分布

### 训练数据量级（视频条数/小时数/token数，预训练与SFT分开） ⚠️

[不确定]。技术报告完全未披露训练数据的绝对规模（未给出视频条数、小时数、token 数），也未区分预训练与后训练的数据量。仅定性描述为「高质量、多样、强交互性的单人单镜头视频语料（a corpus of high-quality, diverse, and highly interactive single-person, single-shot video）」。三阶段训练（双向教师训练 / 因果自回归适配 / DMD 蒸馏）各阶段用量同样未公开。

### 数据来源构成（自有/公开数据集/网络爬取/授权采购/合成数据）

明确为两大类原始视频来源（均未说明获取渠道是自有、采购还是爬取）：
(1) 直播与口播/说话人头部视频（livestream or talking-head videos）——主要用于学习面部表情、身体动作、唇形同步等细粒度特征；
(2) 影视剧高质量素材（high-quality footage from films and television dramas）——用于提升模型在不同镜头角度、场景与视觉风格上的泛化性与一致性。
未提及使用公开数据集、授权采购或合成数据。评测端使用了公开基准 HDTF 作为参考。

### 数据合规与溯源（授权数据占比、rights-cleared数据集、C2PA等） ⚠️

[不确定]。报告未提及任何数据授权、版权合规、rights-cleared 数据集比例、C2PA/水印溯源等信息。仅在清洗环节提到内容安全过滤（NSFW 及其他不当内容剔除）与画面干净度过滤（去除含水印、字幕、贴片广告的片段），后者更多是画质动机而非版权动机。考虑到数据来源包含影视剧素材，版权来源披露缺失是明显空白。

### 片段时长分布与切分策略

切分后训练片段时长为 3~60 秒的单镜头 clip。切分策略：先沿镜头边界（shot boundaries）分割保证单镜头连续性；长镜头进一步细分，且切点被约束为不得落在一句话语音的中间（cut points constrained so as not to fall in the middle of speech），以保护语音-唇形的完整性。未给出时长的具体分布直方图或平均时长。

### 分辨率/宽高比分布与分桶策略 ⚠️

[不确定]。训练侧仅在预过滤阶段以「帧率、分辨率」作为技术门槛剔除低帧率/低分辨率视频，未公布具体阈值、分辨率分布或分桶（bucketing）策略。推理/输出侧为 540p（960×540）、25 FPS 标准、最高 42 FPS（RTX 5090 实测），未提及多宽高比支持。

### 类别/domain分布与配比策略（人物、动作、场景、风格等比例控制与概念均衡） ⚠️

未给出定量配比，但存在很强的定性 domain 约束，本质上是一个高度垂直的数据分布：
(1) 主体约束：每个 clip 必须且仅含一个主体（exactly one subject），且主体在画面中占比合理——即严格的「单人」分布；
(2) 镜头约束：只保留静止镜头或慢速运动镜头（retain static shots or shots with slow motion），以降低长时生成中的镜头漂移风险——刻意压低了大运镜的比例；
(3) 交互性约束：要求画面中主体展现清晰的动作或行为（clear actions or behaviors），以保证模型学到有意义的运动信息；
(4) 风格覆盖：通过影视剧素材补充多角度、多场景、多视觉风格；产品端支持真人、动漫、宠物等多种自定义角色形象，暗示训练数据覆盖二次元与非人主体（论文提到人脸检测专家模型对夸张/高度风格化的2D动画主体泛化不佳，故引入 omni 模型补充）；
(5) 语义标签维度：由 omni 模型沿 editing（剪辑）、subject（主体）、action（动作）、emotion（情绪）、face（人脸）、speech（语音）、scene（场景）、shot（镜头）、tone（影调）九个质量维度打标，这九个维度构成了事实上的 domain 描述体系，但论文未公布各维度的配比数值。
两大来源（直播/口播 vs 影视剧）之间的比例同样未披露 [不确定]。

### 音频类别分布与配比（语音/音效foley/音乐/环境音/静音的比例及控制策略）——AV模型独有维度 ⚠️

未给出定量比例 [不确定]，但有明确的音频类别处理与筛选策略，方向是「纯化语音、剔除音乐」：
(1) 从原始音频中先提取语音成分（extract the speech component from the raw audio）；
(2) 通过 VAD（语音活动检测）+ ASD（主动说话人检测）标注每个语音段的时间戳与所属说话人；
(3) 将每段语音分为 onscreen（说话人即画面中人物）/ offscreen（画外音，说话人不在画面中）/ overlap（多人声重叠）三类；含 overlap 段的 clip 直接整条剔除；
(4) 针对唱歌与强背景音乐场景 diarization 模型不稳定（易把人声误判进音乐 stem、产生合成音色与失真伪影）的问题，引入启发式规则：若说话人在发声但语音能量占比过低（speech energy proportion too low），则丢弃该段——实际效果是系统性剔除歌唱与音乐主导片段。
(5) caption 层面仍标注 sound effects（音效）与 background music（背景音乐）字段，说明这两类音频信息被保留为可描述属性，但语音仍是训练分布的绝对主体。

### 叙事结构分布（单镜头vs多镜头、平均clip时长、镜头数分布、是否含原生音轨） ⚠️

全部为单镜头（single-shot）片段，明确排除多镜头叙事：原始视频沿镜头边界切分为单镜头 clip，长镜头再细分，最终均为 3~60 秒单镜头单人片段。全部训练数据均含原生音轨（预过滤阶段即以「音视频完整性 audio-visual integrity」剔除缺失音轨或音视频不完整的样本）。未给出平均 clip 时长与镜头数分布的数值 [不确定]。

### 语言/口音分布（多语种唇同步能力的数据基础） ⚠️

[不确定]。论文未披露训练数据的语种与口音分布，也未说明多语种唇同步能力的数据基础。仅在 caption 结构中包含 dialogue（对白）字段，并在 Vidu-StreamBench 基准中提到覆盖多样的「说话人属性（speaker attributes）」与情绪，但未展开语种维度。产品面向中文市场，推测以中文为主但无一手依据。

## 清洗流程

### 清洗漏斗整体结构（几级过滤、各级顺序）

六级渐进式过滤漏斗（论文 Figure 2 明确画出，从 Raw Videos 到最终训练数据）：
第1级 Prefiltering（预过滤）：先做去重，再按帧率、分辨率、音视频完整性、音视频同步性四项技术指标剔除不可靠视频 → 输出 Pre-filtered；
第2级 Clipping（切分）：沿镜头边界切为单镜头 clip，长镜头细分且切点避开语音中段，叠加时长过滤（Duration Filter）保留 3~60 秒 → 输出 Single-Shot Clips；
第3级 Subject Filtering（主体过滤）：Subject Filter 保证画面中恰好一个主体且占比合理 → 输出 Single-Person Clips；
第4级 Other Filtering（综合过滤）：并列四道 —— 画面内容/视觉质量过滤（Frame Content / Visual Quality Filter）、内容安全过滤（Content Safety Filter）、镜头稳定性过滤（Shot Stability Filter）、交互性过滤（Interactivity Filter）；
第5级 Diarization（说话人分离）：VAD + ASD 标注语音段与说话人，onscreen/offscreen/overlap 三分类，Overlap Filter 剔除重叠人声片段，Speech Energy Filter 剔除语音能量占比过低（唱歌/强背景音乐）片段；
第6级 Caption + Embedding：生成 clip 级与 speech-aware chunk 级双粒度结构化 caption，最终入选 clip 与其 caption 一起被 embedding 成训练数据。
设计目标被总结为同时提升视觉清晰度、时序稳定性、音视频一致性与跨模态可解释性。

### 漏斗定量保留率（每级过滤的输入/输出量与最终保留率，如Apollo 27%） ⚠️

[不确定]。论文虽给出完整的六级漏斗结构图（Figure 2），但未提供任何一级的输入/输出数据量或保留率数字，也没有最终保留率。这是该报告数据披露的主要缺口之一（对比 Apollo 类工作给出 27% 保留率的做法）。

### 镜头切分方法（PySceneDetect/自研模型/shot-aware splitting） ⚠️

沿镜头边界（shot boundaries）将原始视频切为单镜头 clip；长镜头进一步再切分。关键约束是「切点不得落在语音中间」（speech-aware cutting），这是为唇形同步任务定制的切分策略。未说明具体使用 PySceneDetect、TransNetV2 还是自研镜头边界检测模型 [不确定]。

### 质量过滤（美学评分、清晰度、OCR文字过滤、黑边/水印/logo检测） ⚠️

多维度、专家模型 + 大模型混合的质量过滤，共六个关注点：
(1) 主体检测（subject detection）：每个 clip 恰好含一个主体且在画面中占比合理；
(2) 画面干净度（frame cleanliness）：剔除含与视觉内容无关信息的片段，明确列举水印（watermarks）、字幕（subtitles）、贴片/叠加广告（overlaid advertisements）；
(3) 视觉质量（visual quality）：使用美学评分（aesthetic scoring）与技术评分（technical scoring）双评分，筛选清晰、完整、观感良好的片段，规避模糊（blur）、抖动（jitter）、闪烁（flicker）等伪影；
(4) 内容安全：剔除 NSFW 及其他不当内容；
(5) 镜头稳定性（shot stability）：保留静止镜头或慢速运动镜头，以降低长时流式生成中的镜头漂移（shot drift）风险；
(6) 交互性（interactivity）：要求主体有清晰的动作/行为。
未公开任何评分器的具体名称与阈值数值 [部分不确定]。未提及 OCR 文字检测的具体实现、黑边检测。

### 运动过滤（光流/运动分数阈值、静态与抖动剔除） ⚠️

有两条方向相反的运动约束：一方面通过「镜头稳定性」过滤只保留静止或慢速运动的镜头，剔除大幅运镜/抖动（jitter 亦在视觉质量项中被剔除）；另一方面通过「交互性」过滤要求主体本身具有清晰可辨的动作或行为，剔除完全静止无动作的样本。即「镜头要稳、主体要动」。未提及使用光流或具体运动分数阈值 [不确定具体方法与数值]。

### 去重方法（精确去重与基于embedding的语义去重分别记录） ⚠️

在流程最前端（正式进入 pipeline 之前）对所有数据先做去重（all data are first deduplicated），再进行预过滤。论文未区分精确去重与基于 embedding 的语义去重，也未说明所用特征或相似度阈值 [不确定]。

### VLM/LLM作为质检员（多模态大模型质量打分与错配剔除，2026年从浅层打分器转向大模型语义判断的趋势） ⚠️

这是该报告数据侧最突出的方法论主张，体现了2026年「从浅层打分器转向大模型语义判断」的趋势：论文明确指出仅用专家模型（expert models only）评估视频数据存在明显局限——例如人脸检测模型难以泛化到夸张或高度风格化的2D动画主体；部分专家模型是基于图像的、只能对抽帧做判断，因全局信息不足而产生误判。因此引入 omni model（全模态大模型，引用文献[24,25]）对完整视频进行全局语义理解作为补充，由其沿 editing、subject、action、emotion、face、speech、scene、shot、tone 九个质量维度生成语义标签。专家模型与 omni 模型共同构成兼具全局上下文感知与局部细节敏感度的联合过滤系统（joint filtering system）。omni 模型的具体身份与规模未披露 [不确定]。此外 caption 标注同样由标注模型（annotation model）完成。

### 安全与合规过滤（NSFW、版权、人脸/隐私） ⚠️

在第4级过滤中设有独立的内容安全过滤（Content Safety Filter），剔除 NSFW 及其他不当内容，动机是防止模型学到有害信息。未提及版权过滤、人脸隐私/肖像权处理、名人过滤等机制 [不确定]。

## 打标方式

### 使用的caption模型（自研VLM/开源模型，模型规模） ⚠️

使用了一个「标注模型（annotation model）」生成结构化自然语言描述，并在质量过滤阶段使用了 omni model（全模态大模型）。论文均未披露这些模型是自研还是开源模型、参数规模多大 [不确定]。

### caption密度与结构化程度（短/长/密集描述、结构化字段如镜头运动、风格标签）

双粒度 + 结构化，为流式交互式生成定制：
(1) full-clip caption（整段级）：为整条视频提供连贯的全局语义锚点；
(2) speech-aware chunk-level caption（语音感知的分块级）：将描述与其对应的时间区间对齐，为可控的交互式流式生成提供细粒度、时序局部化的条件信号——这是适配自回归流式范式的关键设计，使得用户可在生成过程中随时用语音指令改变后续内容。
结构化字段覆盖视觉、听觉、对白三类属性，明确列举：主体外观（subject appearance）、动作（actions）、运动（motion）、情绪（emotion）、场景上下文（scene context）、镜头语言（camera language）、影视化属性（cinematic properties）、光照（lighting）、画面文字（on-screen text）、对白（dialogue）、音效（sound effects）、背景音乐（background music）共12类。

### 音视频联合caption结构（是否同时覆盖视觉+听觉轨道、是否分流为独立字段，如LTX-2全音景描述、Script-a-Video factorized streams、Foley-Omni三字段）

是典型的「音视频联合 caption 但双路解耦」方案：caption 同时覆盖视觉轨道与听觉轨道（听觉侧含 dialogue、sound effects、background music 三类字段）。为提升标注保真度、减少跨模态幻觉（cross-modal hallucination），采用 dual-path strategy 解耦两个模态——视觉属性只从视频帧推断（inferred exclusively from video frames），听觉属性只从音轨推断（inferred exclusively from the audio track），二者互不交叉参考后再合并为结构化描述。论文称该结构化标注方案提升了多模态表征的质量与一致性，并使标注更可靠、生成更可控。与 LTX-2 的全音景描述、Foley-Omni 三字段属同一类思路，但 Vidu S1 的特色是叠加了 speech-aware 的时序分块对齐。

### 对白转写与说话人属性标注（ASR转写+说话人身份/语言/口音/情绪） ⚠️

有较完整的说话人属性标注链路：从原始音频中提取语音成分 → VAD（voice activity detection）+ ASD（active speaker detection）标注每个语音段的时间戳及其关联说话人 → 依据说话人与画面主体的对应关系将每段标为 onscreen / offscreen / overlap 三类。caption 中含 dialogue 字段。但论文未明确说明是否做了完整 ASR 文本转写，也未标注说话人的语种、口音、年龄等属性 [部分不确定]。评测基准 Vidu-StreamBench 中提到覆盖多样的说话人属性与情绪。

### 几何与结构化标注（相机参数、深度、3D point tracks、动作标注、显式状态） ⚠️

无显式几何标注。caption 中含「镜头语言（camera language）」与「影视化属性（cinematic properties）」「光照（lighting）」等半结构化文本标签，omni 模型另输出 shot、scene 等语义标签，但论文未提及相机参数估计、深度图、3D point tracks、姿态/骨骼关键点等几何或显式状态标注 [不确定]。

### 合成数据构造（受控扰动/编辑构造训练对，如InstructAV2AV） ⚠️

[不确定]。论文未提及使用合成数据或通过受控扰动/编辑构造训练对。训练侧的「人造监督」体现在训练策略而非数据构造上：第二阶段用 Teacher Forcing 与 Diffusion Forcing 混合策略做因果适配，第三阶段用 DMD（Distribution Matching Distillation）+ PCM（Phased Consistency Models）正则做少步蒸馏，由双向教师模型为学生提供分布匹配监督。

### 人工介入程度（人工标注、人工质检、模型初筛+人工复核） ⚠️

数据侧几乎全自动：过滤与标注均由专家模型 + omni 大模型 + 启发式规则（如语音能量占比规则）完成，论文未提及任何人工标注或人工复核环节 [数据侧人工介入程度不确定]。人工介入明确出现在评测端：在自建的 Vidu-StreamBench（500 样本）上组织人工偏好评测，与 HeyGen、LemonSlice、Kling Avatar 2.0 三家商用系统做成对 A/B 测试，评测维度含 Overall、Motion Dynamics、Identity Consistency、Audio-Video Sync、Subject Controllability。

## 音视频对齐

### 音视频同步检测方法（口型同步、事件对齐）

音视频同步在数据流程中被放在两个位置：
(1) 预过滤级的粗筛：入 pipeline 前即按「音视频同步性（audio-visual synchronization）」这一指标剔除不同步的原始视频，与帧率、分辨率、音视频完整性并列为四项技术门槛；
(2) Diarization 级的精细对齐：论文强调「只有当模型在训练中观察到与对应语音一致的视觉表现，才能学到高保真的唇形同步」，因此用 VAD + ASD 定位每段语音的时间戳与说话人，并通过说话人与画面主体的匹配关系判定 onscreen/offscreen/overlap；offscreen（画外音，声画主体不一致）与 overlap（多人声重叠）均被视为破坏声画对应关系而处理——含 overlap 的 clip 直接整条剔除。此外镜头切分时切点避开语音中段，也是为保护语音-唇形段的完整性。
评测侧使用 Sync-D 指标（基于 SyncNet/Wav2Lip 的 lip-sync expert，引用文献[59] Prajwal 等 「A lip sync expert is all you need」）衡量音视频同步，Vidu S1 在 HDTF 上取得 7.8470（越低越好），优于 OmniAvatar 9.242、StableAvatar-1.3B 11.18、Hallo3 8.660、Wan2.2-S2V-14B 8.255、LiveAvatar 8.447、LemonSlice 7.921、HeyGen 8.037、Kling Avatar 2.0 8.158。

### 同步检测具体指标与阈值（SyncNet/Synchformer/LSE/自研，阈值数值，如MOVA LSE-D≤9.5且LSE-C≥4.5、SkyReels-V4 SyncNet |offset|≤3∧conf>1.5） ⚠️

数据过滤阶段：论文只说明按「音视频同步性」预过滤，未给出所用同步检测模型（是否 SyncNet）与任何阈值数值 [不确定]。启发式规则中的「语音能量占比过低」同样未给出具体阈值 [不确定]。
评测阶段：明确使用 Sync-D（SyncNet 系 lip-sync expert 距离指标），Vidu S1 = 7.8470（HDTF，全场最优）；同期报告 CSIM = 0.9192（身份保持，最优）、DOVER = 0.5660（感知质量，最优）。

### 时序同步vs语义同步的分离处理（时间对齐与内容语义匹配作为两个独立过滤条件）

存在事实上的分离处理，但论文未用此术语表述：时序同步由预过滤的音视频同步指标 + VAD 语音段时间戳 + 切点避开语音中段共同保证；语义/身份层面的声画匹配则由 ASD 主动说话人检测与 onscreen/offscreen 分类保证（判断「这个声音是不是画面里这个人发出的」），二者是流程中两个独立的过滤条件。此外 caption 的 dual-path 策略（视觉属性只看画面、听觉属性只听音轨）也是在语义标注层面刻意解耦两模态以避免跨模态幻觉。

### 音频质量过滤（SNR、静音检测与静音占比阈值、无音轨剔除、画外音源剔除、背景音乐分离） ⚠️

(1) 无音轨/音视频不完整样本在预过滤阶段即被「音视频完整性」检查剔除；
(2) 画外音源剔除：ASD 判定为 offscreen 的语音段（说话人不在画面中）被单独标出并区别处理；overlap（多人声重叠）clip 直接整条剔除；
(3) 背景音乐/歌唱剔除：观察到 diarization 模型在唱歌或强背景音乐场景下不稳定（人声被误分到 music stem、产生合成音色与失真伪影），故引入启发式规则丢弃「说话人在发声但语音能量占比过低」的片段；这实际上也起到了人声与背景音乐分离质量的兜底作用；
(4) 语音成分提取：训练前从原始音频中显式抽取 speech component。
未提及 SNR 阈值、静音占比阈值等具体数值 [不确定]。

### 语音/音效/音乐的分类与分别处理策略 ⚠️

语音（speech）是唯一被作为训练与控制核心的音频类型：显式提取语音成分、做 VAD/ASD/说话人归属分类，并以语音作为流式生成的实时控制信号。音乐与歌唱被视为噪声源，通过语音能量占比启发式规则系统性剔除。音效与背景音乐不参与专门建模，但在结构化 caption 中保留为独立描述字段（sound effects、background music）。产品端支持用户选择不同音色（voice tones），推测音频侧接入了 TTS/音色控制，但论文未展开 [不确定]。

## 训练配合

### 多阶段训练课程与数据课程调度（阶段划分依据：分辨率/时长/质量分/模态；低清→高清、图像→视频、短→长） ⚠️

训练分三阶段，但阶段划分依据是生成范式与推理效率，而非数据课程（未见按分辨率/时长/质量分组织的低清→高清、短→长课程）：
阶段1 Bidirectional Teacher Training：在完整视频-音频序列上训练双向教师模型，条件为完整序列 c^{1:N}，对联合潜在状态 x_0^{1:N} 去噪，作为整个框架的基础；
阶段2 Causal Adaptation：从预训练双向模型初始化自回归模型，用统一 Teacher Forcing 与 Diffusion Forcing 的混合训练策略适配到因果生成设定，使模型获得稳定的多步自回归生成能力；
阶段3 DMD 蒸馏：采用 Distribution Matching Distillation 配合 Phased Consistency Models（PCM）正则，将自回归模型蒸馏为少步生成器，在保持生成质量的同时大幅提升推理效率。
各阶段所用数据是否有差异未说明 [不确定]。

### 数据配比随训练阶段的变化（预训练/退火annealing/SFT高质量子集） ⚠️

[不确定]。论文完全未披露三个训练阶段之间的数据配比变化，也未提及退火（annealing）阶段或高质量子集。数据侧只描述了一条统一的清洗流水线，未说明其产物如何在阶段间划分。

### 后训练数据（SFT精选集规模与筛选标准、偏好对数量与标注方式、reward model训练数据） ⚠️

[不确定/基本没有]。论文未提及任何 SFT 精选集、偏好对（DPO）、RLHF 或 reward model 训练数据。第三阶段的 DMD+PCM 蒸馏属于模型压缩/加速而非偏好后训练，其监督来自阶段1的双向教师模型分布而非人工偏好数据。人工偏好只用于评测（Vidu-StreamBench 500 样本 A/B 测试），未反哺训练。

### 数据处理基础设施与吞吐（NeMo Curator/Data-Juicer/自研，GPU加速比，处理规模，成本） ⚠️

[不确定]。论文未披露数据处理侧的基础设施、框架（未提 NeMo Curator / Data-Juicer / 自研系统）、GPU 加速比、处理规模或成本。报告的工程篇幅全部投向推理与服务侧：自研 TurboDiffusion（模型加速）与 TurboServe（高效流式调度），采用 Ulysses 式上下文并行（context parallelism）做多卡切分、算子融合减少 host launch 与同步开销、RoPE Repositioning 避免历史特征重复计算、TwinCache（阶段感知的噪声缓存 + 干净缓存双缓存）提升流式推理效率与时序稳定性；实测 RTX 5090 上 540p 42 FPS，超过 30 FPS 实时播放阈值，消费级 GPU 即可运行。

## 效果对比

### 数据策略消融的量化影响（区分：过滤严格度ablation / caption密度风格ablation / 数据配比ablation，及对应评测指标） ⚠️

[不确定]。论文未做任何数据侧消融实验——没有过滤严格度 ablation、没有 caption 密度/风格 ablation、没有数据配比 ablation。所有实验都是端到端的方法对比（Vidu-StreamBench 人工偏好 A/B + HDTF 上 CSIM/Sync-D/DOVER 三指标对比）与推理效率验证。数据处理各项设计（omni 模型辅助过滤、双粒度 caption、dual-path 解耦标注、overlap 剔除、语音能量规则）均只给出定性动机说明，无量化收益证据。

### 质量vs数量的证据（小而精数据超越大而杂的案例）

无直接量化证据，但整体设计路线明确是「质量与分布纯度优先于数量」：数据规模只字未提，而过滤条件极为严苛（单人、单镜头、镜头静止或慢速、必须有清晰动作、无水印字幕广告、无重叠人声、无低语音能量占比片段、3~60秒），最终数据分布高度垂直。论文的定性主张是「训练数据质量实质性地影响训练表现与模型泛化能力」，并强调严格过滤规则是保证训练数据质量的手段。间接旁证：Vidu S1 在 HDTF 上以 540p 实时模型的身份，CSIM(0.9192)、Sync-D(7.8470)、DOVER(0.5660) 三项全面超过 Wan2.2-S2V-14B、OmniAvatar、Hallo3、StableAvatar-1.3B 等离线开源模型及 HeyGen、LemonSlice、Kling Avatar 2.0 等商用系统。

### 训练数据domain分布与评测基准类目体系的对齐关系（如VABench七大类） ⚠️

存在较清晰的对齐：自建基准 Vidu-StreamBench 含 500 个样本，每个样本由「动作指令 + 参考首帧 + 音频片段」三元组构成，覆盖多样的动作指令、参考图风格、说话人属性、情绪与应用场景——这与训练数据的过滤维度（交互性/清晰动作、主体、情绪、多视觉风格、说话人语音）逐条呼应。人工评测的五个维度（Overall、Motion Dynamics、Identity Consistency、Audio-Video Sync、Subject Controllability）也分别对应训练数据侧的交互性过滤、单主体过滤+镜头稳定性、diarization 声画对应、以及 speech-aware chunk caption 带来的可控性。omni 模型打标的九个质量维度（editing/subject/action/emotion/face/speech/scene/shot/tone）可视为训练侧的类目体系，但论文未给出它与基准类目的显式映射表或各类目占比 [部分不确定]。公开基准侧只用了 HDTF（标准 audio-driven avatar 基准），论文指出 HDTF 等公开基准无法充分评估指令跟随与实时交互中的自然运动，这正是自建 Vidu-StreamBench 的动机。

## 不确定字段

以下字段的调研信息部分不确定（⚠️ 标注来源）：

- data_scale
- provenance_licensing
- resolution_aspect_distribution
- domain_distribution
- audio_category_distribution
- narrative_structure_distribution
- language_accent_distribution
- funnel_retention_rate
- shot_segmentation
- quality_filtering
- deduplication
- motion_filtering
- model_as_data_judge
- safety_filtering
- caption_model
- dialogue_transcription_attributes
- geometric_structured_annotation
- synthetic_data_synthesis
- human_in_loop
- sync_metric_and_threshold
- audio_quality_filtering
- audio_type_handling
- multi_stage_curriculum
- stage_data_mixture
- post_training_data
- data_infra_throughput
- data_ablation
- benchmark_taxonomy_alignment
