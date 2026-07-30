# Foley-Omni（配套评测基准 V2ST-Bench）。论文全称《Foley-Omni: A Unified Multimodal Generation Model from Task-Level Audio Synthesis to Complete Video Soundtrack Generation》，即从任务级音频合成迈向完整视频配乐（Video-to-Soundtrack, V2ST）生成的统一多模态生成模型。核心定位是在同一个隐空间生成过程中联合建模语音（speech）、音效（sound effects/foley）与音乐（music）三类音轨。

> 主题: 视频生成模型（含音视频同时生成）的数据处理：数据清洗流程、数据分布、打标方式

[← 返回首页](../index.md)

**目录**: [基本信息](#基本信息) · [数据规模与分布](#数据规模与分布) · [清洗流程](#清洗流程) · [打标方式](#打标方式) · [音视频对齐](#音视频对齐) · [训练配合](#训练配合) · [效果对比](#效果对比)

## 基本信息

### 名称

Foley-Omni（配套评测基准 V2ST-Bench）。论文全称《Foley-Omni: A Unified Multimodal Generation Model from Task-Level Audio Synthesis to Complete Video Soundtrack Generation》，即从任务级音频合成迈向完整视频配乐（Video-to-Soundtrack, V2ST）生成的统一多模态生成模型。核心定位是在同一个隐空间生成过程中联合建模语音（speech）、音效（sound effects/foley）与音乐（music）三类音轨。

### 发布机构/公司

以南京大学智能科学与技术学院（School of Intelligence Science and Technology, Nanjing University）为主导，联合 Video Rebirth（工业界合作方）、上海交通大学、北京交通大学、上海人工智能实验室（Shanghai AI Laboratory）。作者列表：Ye Tao（陶烨，一作，联系邮箱 taoye0402@gmail.com，项目主页 ty0402.github.io 维护者）、Lupeng Liu、Xuenan Xu（徐雪男，音频描述/音频caption方向研究者）、Jiasun Feng、Jiarui Wang、Ying Qin、Shuiyang Mao、Wei Liu、Shuai Wang（王帅，通讯作者方向，南大语音组）。GitHub 组织名为 NJU-Speech，HuggingFace 账号为 CocoBro。

### 发布时间（技术报告/论文/开源时间）

2026年6月2日提交至 arXiv（arXiv:2606.03672，cs.SD/cs.CV 方向，v1版本）。同期上线项目主页 https://ty0402.github.io/Foley-omni-Web/ 与 GitHub 仓库 NJU-Speech/Foley-Omni，HuggingFace 上发布 CocoBro/Foley-Omni 推理权重。ResearchGate 于同期收录（publication 405852241）。V2ST-Bench 评测集截至调研时（2026年7月）仓库标注为 Coming soon，尚未正式放出。

### 类型（模型/数据集/工具链/评测基准）

模型 + 评测基准 + 数据处理pipeline方法论 的组合产出。主体是视频到完整配乐（V2ST）的统一多模态音频生成模型（约5.5B参数量级，基于 Diffusion Transformer）；同时提出 V2ST-Bench 评测基准（300条视频-音频-文本三元组）；论文第3.1节完整描述了一条音视频数据清洗与结构化标注pipeline。注意：本项目生成的是音频/配乐，视频侧为条件输入而非生成目标，因此属于「音视频生成」谱系中的V2A/V2ST分支，而非文生视频模型。

### 开源程度（权重/代码/数据/pipeline各自是否开源）

开源程度中等偏上。
【权重】开源。HuggingFace CocoBro/Foley-Omni 发布 inference-only 权重（v2st.pth），同包分发音频VAE、BigVGAN vocoder、视觉特征抽取模型等依赖组件，声明为 MIT 许可证。
【代码】部分开源。GitHub NJU-Speech/Foley-Omni 提供推理代码与视觉特征预处理脚本（CLIP特征、Synchformer sync特征抽取），训练代码未见发布。仓库自身许可证未在页面明确标注，并声明重分发了 Wan2.2-TI2V-5B、MMAudio、Ovi 等上游项目组件，需分别遵循上游许可。
【数据】训练数据不开源。约4.9k小时、2.7M对训练语料中自采/内部部分（internal）不公开；公开数据集部分（LJSpeech、LibriTTS、AudioCaps、Freesound、MusicCaps、MusicBench、AudioSet、VGGSound、GRID、LRS2、Chem、SpeakerVid、TalkVid、Kling-Foley）可自行获取。
【评测基准】承诺开源但尚未释出。论文称将发布 V2ST-Bench 的 annotations、metadata 与 processing scripts；因版权限制不直接分发原始视频文件，改为提供 URL + metadata（类似 VGGSound/HowTo100M 的做法）。
【pipeline】方法论层面披露较充分：Table 7 给出六项过滤指标的具体阈值数值，Table 12 给出 Gemini 2.5 Pro 标注 prompt 模板，声学后验证给出 -35 dB 显式阈值公式。但未发布完整清洗脚本，也未给出逐级保留率表。

### 是否支持音视频同时生成，以及实现方式（原生联合/级联/MoE融合）

不是「音视频同时生成」模型，而是「视频条件下的多类音频联合生成」。实现方式为原生联合生成（native joint generation）而非级联：语音、音效、音乐三类音轨在同一个共享隐空间的扩散生成过程中一次性联合产出，而非分别用TTS/Foley/音乐模型生成后混音。这是相对基线（MMAudio + CosyVoice 3 + AudioX 级联管线、MMAudio + LipVoicer + AudioX 级联管线）的核心差异点。
架构上为 Diffusion Transformer（DiT），文本侧用共享的 UM-T5 encoder 编码三字段结构化文本，音频侧用 frozen Mel VAE（沿用 MMAudio）+ BigVGAN vocoder，视频侧双路条件：CLIP 特征提供场景语义、Synchformer 特征提供时序同步线索（后者以 additive path Z_sync 注入）。
实证支持联合生成优于级联：V2ST-Bench 上 Foley-Omni 的 WER 7.59 优于 MMAudio+CosyVoice 3+AudioX 的 10.57 与 MMAudio+LipVoicer+AudioX 的 37.84，DeSync 0.16 亦优于 0.85/0.26，且三项主观分 A-MOS 3.92 / S-MOS 4.13 / T-MOS 4.14 全面领先级联基线（对照 Ground Truth 为 4.33/4.37/4.42）。

### 调研信息来源列表（论文/技术报告/官方文档/新闻的URL，标注每条来源的性质：官方一手/同团队旁证/第三方报道）

1. 【官方一手】arXiv 论文摘要页 https://arxiv.org/abs/2606.03672 —— 标题、作者、摘要、2026年6月2日提交日期。
2. 【官方一手】arXiv HTML 全文 https://arxiv.org/html/2606.03672v1 —— 第3.1节数据清洗pipeline、Table 7 过滤阈值、Table 9 训练数据构成、Table 6 消融、V2ST-Bench 主结果表、附录 B/C.1。核心信息来源。
3. 【官方一手】arXiv PDF https://arxiv.org/pdf/2606.03672
4. 【官方一手】HuggingFace 模型页 https://huggingface.co/CocoBro/Foley-Omni —— 5.5B 参数量、MIT 许可证、inference-only 权重构成、上游依赖声明。
5. 【官方一手】GitHub 仓库 https://github.com/NJU-Speech/Foley-Omni —— 推理代码、特征抽取脚本、V2ST-Bench「Coming soon」状态。
6. 【官方一手】项目主页 https://ty0402.github.io/Foley-omni-Web/ —— demo 样例与项目介绍。
7. 【第三方索引】ResearchGate 收录页 https://www.researchgate.net/publication/405852241_Foley-Omni_A_Unified_Multimodal_Generation_Model_from_Task-Level_Audio_Synthesis_to_Complete_Video_Soundtrack_Generation
8. 【第三方报道】AI Films Studio 博客 https://studio.aifilms.ai/blog/foley-omni-video-soundtrack-generation —— 面向创作者的通俗解读，无新增一手数据。

## 数据规模与分布

### 训练数据量级（视频条数/小时数/token数，预训练与SFT分开） ⚠️

总训练规模约 2.7M 音频/视频-文本配对样本，累计约 4.9k 小时音频经过清洗pipeline处理。其中经数据清洗pipeline产出的统一视频-音频-文本三元组 (v_i, a_i, Ŝ_i) 约 2.0M 条（这部分是本文pipeline的直接产物，其余为直接取用的公开TTS/TTA/TTM语料）。
按六个任务组拆分（Table 9）：
- TTS（文本转语音）：LJSpeech + LibriTTS + internal，1,253 小时
- TTA（文本转音频/音效）：AudioCaps + Freesound，912 小时
- TTM（文本转音乐）：MusicCaps + MusicBench + AudioSet，139 小时
- VisualTTS（视觉条件语音合成）：Chem + GRID + LRS2 + SpeakerVid + TalkVid，1,980 小时（占比最大）
- V2A（视频转音效）：VGGSound + Kling-Foley + internal，403 小时
- V2ST（视频转完整配乐，本文核心任务）：internal + SpeakerVid，216 小时（规模最小，作为第三阶段微调数据）
阶段视角：Stage 1 使用约 0.7M 文本-音频对（TTA/TTS/TTM）；Stage 2 扩展至视频条件（V2A/VisualTTS）；Stage 3 用 216 小时 V2ST 数据 + 每个前序单任务域各 100 小时回放数据做联合微调。
评测侧 V2ST-Bench 为 300 条 5–10 秒片段。
[不确定] 论文未披露清洗前的原始视频池总量（小时数或条数），因此无法反推整体保留率。

### 数据来源构成（自有/公开数据集/网络爬取/授权采购/合成数据）

混合来源，公开数据集 + 内部自有数据两大类，未提及网络大规模爬取或授权采购。
【公开数据集】语音类：LJSpeech、LibriTTS（TTS基础）；GRID、LRS2、Chem（经典唇读/视觉语音数据集）；SpeakerVid、TalkVid（较新的大规模说话人视频数据集，同时用于 VisualTTS 与 V2ST）。音效类：AudioCaps、Freesound、VGGSound、Kling-Foley（快手可灵团队发布的foley数据集）。音乐类：MusicCaps、MusicBench、AudioSet。
【内部数据（internal）】在 TTS（内部语音库）、V2A（内部音视频语料）、V2ST（内部数据，构成216小时中的主体）三处出现，是本文数据清洗pipeline的主要作用对象——论文将这部分描述为「weakly labeled audiovisual data」（弱标注音视频对），即只有原始视频+原始音轨、缺乏组件级文本标注的原始素材。
【合成/构造数据】无。三字段标注由 Gemini 2.5 Pro 生成属于「模型标注」而非「合成音视频数据」，pipeline中不涉及人工扰动构造训练对。
值得注意的是数据来源的任务导向性很强：VisualTTS 占 1,980 小时（约40%）说明模型对「说话人视频→语音」这一支路投入最大，与其主打的 speech intelligibility（WER 7.59，甚至优于 Ground Truth 的 8.03 在部分对比中接近）优势直接对应。

### 数据合规与溯源（授权数据占比、rights-cleared数据集、C2PA等） ⚠️

披露非常薄弱。论文仅在 Ethics/Limitations 类表述中笼统声明数据「collected under appropriate usage agreements」（在适当的使用协议下收集），并承认模型存在被用于制作 deepfake 的潜在滥用风险。
间接可见的合规意识：V2ST-Bench 发布策略上明确因版权可再分发性限制（redistributable content constraints）而不直接分发原始视频文件，改为提供 URL 与 metadata，由使用者自行下载——这是学术音视频数据集的常见规避做法（同 VGGSound、AudioSet）。
模型权重侧声明 MIT 许可，并明确标注重分发了 Wan2.2-TI2V-5B 与 MMAudio 的组件、指引用户查阅上游原始许可，属于较规范的上游合规声明。
[不确定] 未披露授权数据占比、未提及 rights-cleared 数据集采购、未提及 C2PA 或任何内容溯源/水印机制、未说明 internal 数据的具体获取渠道与授权形式、公开数据集（如 Freesound、AudioSet、VGGSound）各自许可证的兼容性也未讨论。

### 片段时长分布与切分策略 ⚠️

披露有限。
【评测集】V2ST-Bench 片段时长明确为 5–10 秒。
【训练集】论文未给出训练片段的时长分布直方图或平均时长。可从数据规模反推粗略量级：约 4.9k 小时对应约 2.7M 样本，平均每样本约 6.5 秒，与 V2ST-Bench 的 5–10 秒区间一致，说明整体走的是短片段（约10秒以内）路线——这与其对标的 MMAudio、VGGSound（10秒片段）等 V2A 主流设定吻合。
【切分策略】[不确定] 论文未描述如何将长视频切分为训练片段，未提及镜头边界检测、固定窗口滑窗或其他切分方法。过滤阶段的指标（motion score、Synchformer sync score、ImageBind score）均是片段级计算，暗示切分发生在过滤之前，但具体切分算法未交代。

### 分辨率/宽高比分布与分桶策略 ⚠️

仅有过滤下界，无分布统计。过滤阶段设置视觉分辨率硬门槛 ≥480p，并配合码率门槛 ≥1 Mbps（双重把关，防止低码率高分辨率的「虚高清」样本混入——这一码率约束在同类工作中较少见，UniVerse-1 使用的是码率比 bitrate ratio，思路类似）。
[不确定] 未披露最终数据集的分辨率分布直方图、未披露宽高比（aspect ratio）分布、未提及任何分辨率或宽高比分桶（bucketing）策略。这一点在本项目中影响相对有限：由于视频仅作为条件输入且经 CLIP 与 Synchformer 编码为固定维度特征，分辨率主要影响特征抽取质量而非生成分辨率，因此不需要像文生视频模型那样做严格的分桶训练。

### 类别/domain分布与配比策略（人物、动作、场景、风格等比例控制与概念均衡） ⚠️

论文没有以「视觉domain/类别」维度做配比控制，而是以「任务组」为一级配比单位——这是 Foley-Omni 数据组织方式与文生视频模型的根本差异。
六个任务组的小时数配比（总4.9k小时）：VisualTTS 1,980h（40.4%）> TTS 1,253h（25.6%）> TTA 912h（18.6%）> V2A 403h（8.2%）> V2ST 216h（4.4%）> TTM 139h（2.8%）。可见语音相关（VisualTTS + TTS）合计占66%，音效相关（TTA + V2A）占26.8%，音乐相关（TTM）仅2.8%——严重向语音倾斜，这解释了模型在 WER 指标上的突出优势（7.59，低于两个级联基线的10.57与37.84），同时也可能是音乐生成能力相对薄弱的隐忧。
视觉内容domain可从数据集来源间接推断：GRID、LRS2、Chem 为受控/半受控的说话人正面视频（实验室朗读、BBC新闻、化学讲座），SpeakerVid、TalkVid 为大规模野外说话人视频，VGGSound 为10秒野外事件视频（310类声源），Kling-Foley 为foley专用视频。整体domain高度集中在「有人说话的场景」+「有明确声源事件的场景」两类，缺乏纯风景、抽象艺术、动画等视觉domain。
【运动强度作为隐式domain控制】motion score 限定在 [0.1, 3.2] 区间——下界剔除近静止画面（无视觉事件可对齐，V2A监督信号空洞），上界剔除剧烈运动/快速剪辑（光流估计不可靠、声画对应关系混乱），实际上是通过运动强度把数据domain收窄到「中等动态、单一连续场景」。
[不确定] 未给出显式的视觉类别标签分布统计、未提及概念均衡（concept balancing）或长尾类别重采样策略、未给出人物/动作/场景/风格的比例控制。

### 音频类别分布与配比（语音/音效foley/音乐/环境音/静音的比例及控制策略）——AV模型独有维度 ⚠️

这是 Foley-Omni 最具特色的维度，也是其区别于同期 AV 工作的核心贡献——它把「音频类别」从隐式属性提升为显式的一等结构化字段。
【三分类体系】语音 [WORDS] / 音效 [AUDIO] / 音乐 [MUSIC]，三者构成结构化标注 Ŝ 的三个并列字段。关键设计是「字段可为空」（Each field can be left empty when the corresponding audio component is absent）——空字段本身携带信息，等价于「该片段不含此类音频」的显式负标注。这使得同一套条件接口既能表达单任务（仅填一个字段=TTS/TTA/TTM）也能表达完整配乐（多字段共存=V2ST），实现了任务的统一形式化。
【比例控制策略——双阶段判定】
第一阶段（视觉+听觉联合检测）：Gemini 2.5 Pro 同时接收视频帧与音轨，被显式指令「先判断某类音频组件（Speech/Sound Effects/Music）是否physically present，若存在再生成对应描述」——即先做存在性二分类，再做描述生成，而非一股脑生成三段描述。
第二阶段（声学能量门控纠偏）：用 Bandit 源分离模型（Watcharasupat et al., 2024，专为 cinematic audio source separation 设计的speech/effects/music三分离模型）把原始音轨分离为三个 stem，仅当对应 stem 的 RMS 能量 E(a_c) > −35 dB 时才保留该字段标注。此步骤专门用于消除「视觉幻觉」——即 Gemini 看到画面中有钢琴/有人张嘴/有汽车，就臆断存在音乐/语音/音效，但音轨里实际听不到。这是本文明确点名的方法论创新：视觉路径标注 + 声学路径验证的双路交叉校验（two-path validation），论文称在典型音视频数据集构建中缺失。−35 dB 阈值由「对小规模验证子集的人工检视」（manual inspection of a small validation subset）确定。
【静音处理】过滤阶段第一条即剔除含静音（silence）的片段，无音轨/静音样本不进入后续流程。
【实际分布】训练集三类音频的最终占比未直接给出，但可由任务组小时数近似：语音类66%、音效类27%、音乐类3%。
【评测集显式配比】V2ST-Bench 300条严格要求「≥2个音频组件共存」，配比为 语音+音效 150条（50%）、语音+音乐 120条（40%）、语音+音效+音乐 30条（10%）。三类组合中语音100%出现，可见基准设计也以语音为轴心；三组件齐全的最难场景仅占10%。
[不确定] 训练集中三字段各自的实际非空比例、各组合模式（单字段/双字段/三字段）的样本数分布未披露。

### 叙事结构分布（单镜头vs多镜头、平均clip时长、镜头数分布、是否含原生音轨） ⚠️

整体属于单镜头短片段范式。从 5–10 秒的片段时长、motion score 上界 3.2 对剧烈运动/快切的抑制、以及数据来源（GRID/LRS2/Chem/SpeakerVid/TalkVid 为连续说话人镜头，VGGSound 为单事件10秒片段）综合判断，训练与评测数据基本都是单镜头、无剪辑点的连续片段。这与其任务性质匹配：跨镜头的配乐连贯性建模不是本文目标。
【是否含原生音轨】全部含原生音轨，且原生音轨是唯一的监督目标（生成目标即还原/重建该音轨的三类成分）。这一点与文生视频模型「音轨可有可无」形成对比——对 Foley-Omni 而言无音轨样本直接被过滤掉，原生音轨质量（AudioBox Aesthetics ≥0.6）是硬门槛。
[不确定] 未给出镜头数分布统计、未给出平均clip时长的精确数值、未说明是否做过跨镜头检测与剔除。

### 语言/口音分布（多语种唇同步能力的数据基础） ⚠️

披露缺失，但可从数据集来源强推断为以英语为绝对主体。TTS 侧 LJSpeech（美式英语单说话人）、LibriTTS（英语有声书多说话人）；VisualTTS 侧 GRID（英语受控语料，34名英式说话人）、LRS2（BBC 英式英语）、Chem（英语讲座）均为纯英语数据集；SpeakerVid、TalkVid 为较新的大规模说话人视频集，可能含少量多语种但比例不明。
评测侧使用 WER 作为语音可懂度指标（V2ST-Bench WER 7.59，GRID 子集 WER 15.3），WER 计算通常依赖英语 ASR 模型，亦佐证评测为英语场景。
[不确定] 论文未给出任何语言分布表、未提及口音标注、未提及多语种唇同步能力、未说明 internal 数据的语言构成。这是该工作相对于工业级 AV 模型（如 Veo 3、Sora 2、Kling Omni）在多语种唇同步维度的明显短板。

## 清洗流程

### 清洗漏斗整体结构（几级过滤、各级顺序）

清洗漏斗为三阶段串行结构，核心设计理念是「先过滤后标注、再用声学信号反向校验标注」，输入是弱标注音视频对（weakly labeled audiovisual data），输出是约 2.0M 条结构化三元组 (v_i, a_i, Ŝ_i)。
【第一级：多维质量过滤（Filtering）】置于最前，在昂贵的密集标注之前先砍掉低质样本，是典型的成本导向排序。论文原文表述为「removes clips with silence, low visual resolution, poor audio quality, weak audiovisual semantic consistency, or unreliable synchronization」——五个剔除条件对应三个维度、六项指标（Table 7）：
  · 视觉维度：分辨率 ≥480p、码率 ≥1 Mbps、motion score ∈ [0.1, 3.2]
  · 音频维度：Meta AudioBox Aesthetics 音频质量分 ≥0.6（同时覆盖静音与音质差两类问题）
  · 对齐维度：ImageBind (IB) score ≥0.3（语义一致性）、Synchformer sync score ≥0.2（时序同步可靠性）
  值得注意的是「语义一致性」与「时序同步」被拆成两个独立指标、两条独立阈值，而非合并为单一对齐分——这是本pipeline的一个精细设计。
【第二级：结构化标注（Annotation）】对留存片段调用 Gemini 2.5 Pro，同时输入视频帧与音轨，按 [WORDS]/[AUDIO]/[MUSIC] 三字段模板输出。模型被要求先做组件存在性检测、再做描述生成（detect-then-describe），prompt 模板见论文 Table 12。
【第三级：声学后验证（Acoustic Post-Verification）】用 Bandit 源分离模型把原音轨拆成 speech/effects/music 三个 stem，逐字段做能量门控 E(a_c) > −35 dB RMS；未通过的字段被置空。此级专门修正第二级的视觉幻觉偏差（visual bias），即 VLM 因看到视觉线索而虚构了实际不可闻的音频成分。
【整体评价】该pipeline的方法论亮点在于「双路验证」：视觉/多模态路径负责生成标注，纯声学路径负责否决标注，两条路径信息源不同因而错误模式不相关，交叉校验有效。这一思路对任何依赖 VLM 做音视频联合标注的工作都有借鉴价值。相对短板是缺乏去重、安全过滤、人工复核等常规环节的披露，也没有逐级保留率的定量报告。

### 漏斗定量保留率（每级过滤的输入/输出量与最终保留率，如Apollo 27%） ⚠️

定量披露不足，无法构建完整漏斗表。
【已知端点】pipeline 输出约 2.0M 条视频-音频-文本三元组；全部训练语料约 2.7M 对、约 4.9k 小时。
【缺失】[不确定] 论文未给出清洗前的原始池规模（小时数或条数），未给出任何单级过滤的输入/输出量对比，未给出六项指标各自的淘汰占比，也未给出第三级声学后验证中被置空的字段比例（这个数字本可直接量化 Gemini 视觉幻觉的严重程度，是该方法最有说服力的证据，可惜未报告）。因此无法计算类似 Apollo 27% 那样的端到端保留率。
【可做的粗略推断】六项过滤指标中，Synchformer sync score ≥0.2 与 ImageBind ≥0.3 通常是野外音视频数据的主要杀手（大量网络视频存在配乐后期添加、旁白画外音、声画无关等情况），若参照同类工作（如 UniVerse-1 从 40k+ 小时筛至 7,685 小时约19%、MMAudio 系工作在 VGGSound 上的对齐过滤淘汰率常在50%以上），推测本pipeline原始池应在万小时量级，端到端保留率大致在20–50%区间——但这纯属外推，论文无任何数据支撑。

### 镜头切分方法（PySceneDetect/自研模型/shot-aware splitting） ⚠️

[不确定] 论文完全未提及镜头切分方法。未见 PySceneDetect、TransNetV2、自研 shot boundary detection 或任何 shot-aware splitting 的描述。
从数据构成推断，切分需求本身较弱：训练数据以公开数据集为主，而 GRID、LRS2、VGGSound、AudioCaps 等均已是预先切好的定长短片段（VGGSound 固定10秒、GRID 约3秒），无需再切分；仅 internal 音视频语料与 SpeakerVid/TalkVid 可能涉及从长视频切片，但论文未交代其处理方式。motion score 上界 3.2 可能间接起到剔除含硬切镜头片段的作用（镜头切换会造成光流/运动分数异常尖峰），但这只是副作用而非显式的镜头切分设计。

### 质量过滤（美学评分、清晰度、OCR文字过滤、黑边/水印/logo检测） ⚠️

质量过滤是清洗漏斗的第一级，采用视觉、音频双通道并行把关，全部为阈值型硬过滤，无加权综合评分。
【视觉质量】
  · 分辨率 ≥480p —— 剔除低清片段，保证 CLIP 与 Synchformer 视觉特征抽取的可靠性。480p 门槛相对宽松（对比文生视频模型常见的720p/1080p门槛），合理之处在于视频仅作条件而非生成目标。
  · 码率 ≥1 Mbps —— 与分辨率形成双保险，专门针对「分辨率标称高但压缩严重、实际细节丢失」的样本。这是较少见但很实用的一条，因为上采样过的低质视频能通过分辨率检查却通不过码率检查。
  · motion score ∈ [0.1, 3.2] —— 双边阈值。下界剔除近静止画面（缺乏视觉事件，无法为音效生成提供时序线索），上界剔除过度剧烈运动（运动估计不可靠、声画对应混乱）。motion score 的具体计算方法（光流均值/帧差）论文未指明。
【音频质量】
  · Meta AudioBox Aesthetics 分数 ≥0.6 —— 使用 Meta 开源的 AudioBox Aesthetics 无参考音频美学/质量评估模型。这一个指标同时承担了多重职责：低分自然过滤掉静音片段（无内容则美学分极低）、剔除底噪大/削波失真/低采样率/编码劣化的音轨。相比传统 SNR 阈值，AudioBox Aesthetics 是学习型的感知质量评估，更贴近人耳判断，属于2025–2026年音频数据清洗的新范式。
【未涉及的常规项】[不确定] 论文未提及美学评分（如 LAION-Aesthetics）、OCR 文字过滤、黑边检测、水印/logo 检测、字幕烧录检测等文生视频清洗中的标配环节。对本任务而言这些确实优先级较低（视觉只需支撑语义与同步判断，画面美观与否不影响音频生成），但水印/字幕遮挡嘴部区域可能影响 VisualTTS 的唇同步学习，这一风险论文未讨论。

### 运动过滤（光流/运动分数阈值、静态与抖动剔除） ⚠️

运动过滤通过 motion score 的双边阈值区间 [0.1, 3.2] 实现，是过滤阶段视觉维度三项指标之一。
【下界 0.1】剔除静态或近静态画面。对 V2A/V2ST 任务这一步尤为关键：静止画面无视觉事件，模型无法从中学到「视觉动作→声音事件」的时序映射，这类样本会成为纯噪声监督。
【上界 3.2】剔除运动过于剧烈的片段。抖动、快速摇镜、高频剪辑会使 Synchformer 的时序特征失真，且此类片段的声画对应往往是混乱的（多个事件叠加、声源出画）。
【与文生视频模型的差异】文生视频模型的运动过滤主要服务于「生成有动感的视频」这一目标（剔除静态是为了避免模型生成静止画面）；Foley-Omni 的运动过滤则服务于「保证视听事件可对齐」，出发点不同但阈值形态相似（都是双边区间）。
[不确定] motion score 的具体计算方式（RAFT/UniMatch 光流、帧间差分、还是其他）与归一化尺度均未说明，因此 [0.1, 3.2] 这组数值难以直接迁移复用。

### 去重方法（精确去重与基于embedding的语义去重分别记录） ⚠️

[不确定] 论文完全未提及去重环节，既无精确去重（文件哈希、感知哈希 pHash）也无基于 embedding 的语义去重描述。
潜在风险点：训练数据跨任务组存在数据集重叠——SpeakerVid 同时出现在 VisualTTS（1,980h组内）与 V2ST（216h组内）两处，AudioSet 用于 TTM 而 VGGSound 本身是 AudioSet 的子集衍生（同源 YouTube 视频），存在样本重复计入甚至训练-评测污染的可能。此外 V2ST-Bench 的300条片段论文称「drawn from curated audiovisual pool」（取自同一清洗后的数据池），若未做严格的训练集剔除，存在评测集泄漏的隐患——论文仅说明经过人工复核，未明确声明与训练集无交集。这是该工作在方法论严谨性上一个值得关注的空白。

### VLM/LLM作为质检员（多模态大模型质量打分与错配剔除，2026年从浅层打分器转向大模型语义判断的趋势） ⚠️

Gemini 2.5 Pro 在本pipeline中承担的是「标注生成 + 存在性判别」双重角色，是2026年「大模型作为数据质检员/标注员」趋势的典型案例，但本文更进一步——它不信任大模型的判断，而是设计了独立的声学验证通道来否决大模型的输出。
【Gemini 2.5 Pro 的判别职责】接收视频帧与对应音轨（多模态双输入，而非仅视觉），被 prompt 显式指令执行 detect-then-describe 两步：先判断 Speech / Sound Effects / Music 三类组件各自是否「physically present in the clip」，仅对判定为存在的组件生成描述性 caption。这一步等价于用大模型做三分类的多标签判别 + 条件式描述生成。系统 prompt 模板见论文 Table 12。
【对大模型判断的不信任与纠偏】论文明确指出该环节存在 visual bias（视觉偏置）——多模态模型倾向于被视觉线索主导，看到乐器就标注有音乐、看到人张嘴就标注有语音、看到车辆就标注有引擎声，而实际音轨中这些成分可能根本不存在或不可闻。为此引入第三级 Bandit 源分离 + 能量门控（E(a_c) > −35 dB）作为纯声学的独立判据，凡未达能量阈值的字段一律置空。
【方法论价值】这构成一个「生成-验证」（generate-then-verify）的双路架构：验证路径不依赖任何神经网络语义理解，而是基于源分离后的物理能量测量，与生成路径的错误模式完全解耦。相比单纯依赖 VLM 打分（易受同源偏置影响）或单纯依赖信号处理（无语义理解能力），这种组合更稳健。论文称此为典型音视频数据集构建中缺失的two-path validation。
【局限】[不确定] 未报告 Bandit 验证阶段实际否决了多少比例的 Gemini 标注字段——这个数字是量化 VLM 幻觉严重程度的关键证据，缺失使得该设计的实际收益无法评估；也未做「有无声学后验证」的数据消融来证明其对最终模型性能的贡献。另外未提及使用任何模型对 caption 质量本身（描述准确性、细粒度）做二次打分或剔除。

### 安全与合规过滤（NSFW、版权、人脸/隐私） ⚠️

[不确定] pipeline 中未描述任何安全与合规过滤环节：无 NSFW 检测、无版权内容识别、无人脸隐私处理或去标识化措施。
论文仅在伦理声明层面承认风险：明确提到模型存在被滥用于制作 deepfake 的可能（考虑到 VisualTTS 占训练数据40%、模型能根据说话人视频生成匹配语音，这一风险确实实质存在），并声明数据在「appropriate usage agreements」下收集。但这些属于事后声明而非数据处理环节的技术措施。
未见任何生成内容水印、C2PA 标记、或对说话人身份的授权确认机制。考虑到 GRID、LRS2、SpeakerVid、TalkVid 等数据集均包含大量可识别人脸与声纹，且模型直接学习「人脸→声音」映射，隐私维度的处理缺位是该工作较明显的合规短板。

## 打标方式

### 使用的caption模型（自研VLM/开源模型，模型规模） ⚠️

【主标注模型】Gemini 2.5 Pro（Google 闭源商用多模态大模型）。选型特点是原生支持视频+音频双模态输入——这对本任务是硬需求，因为标注必须同时依据「看到什么」和「听到什么」，纯视觉 VLM（如 Qwen2.5-VL、InternVL）无法胜任音频存在性判别。使用闭源 API 意味着标注成本较高且不可完全复现，但论文未披露调用量与成本。参数规模不公开。
【辅助模型 —— 声学验证】Bandit（Watcharasupat et al., 2024），一个面向 cinematic audio source separation（电影音频源分离）的模型，天然按 speech / effects / music 三路分离，与本文三字段schema完全同构——这是一个非常契合的选型，若用通用音乐源分离模型（如 Demucs 的 vocals/drums/bass/other）则无法直接对应三字段。
【文本编码器】UM-T5 encoder（多语言版 T5），用于把三字段结构化文本编码到共享语义空间，供 DiT 条件注入。所有任务共享同一编码器，是「统一接口」设计的实现基础。
【视觉特征模型】CLIP（场景语义特征）+ Synchformer（时序同步特征），后者同时兼任过滤阶段的同步性打分器——一模型两用，训练与清洗使用同一同步表征，保证了过滤标准与模型学习目标的一致性。
[不确定] 未使用独立 ASR 模型做语音转写（见下条）；未提及任何自研 captioner 或对 Gemini 输出做蒸馏以降低成本。

### caption密度与结构化程度（短/长/密集描述、结构化字段如镜头运动、风格标签） ⚠️

结构化程度高，但单字段描述密度不高——属于「结构优先于密度」的设计取向，与文生视频模型追求长密集caption的路线明显不同。
【格式】统一模板 `[WORDS] <spoken content>; [AUDIO] <sound event descriptions>; [MUSIC] <music specification>`，三个字段固定顺序、显式标签分隔，字段内容可为空字符串。
【结构化的三层价值】
  1. 可组合的条件接口：单字段填充=单任务（TTS/TTA/TTM），多字段填充=完整配乐（V2ST），一套模板覆盖六类任务，避免了为每个任务设计独立条件通路。
  2. 显式负标注：空字段明确告知模型「此片段无该类音频」，而非让模型从缺失中猜测。这对抑制模型在不该生成音乐时凭空加入配乐（V2A 模型的常见失败模式）有直接作用。
  3. 可验证性：字段化使得每个标注单元可被独立的声学证据（对应 stem 的能量）校验，若是自由形式的整段描述则无法做这种细粒度纠错。
【与同期工作的谱系关系】按论文对比语境，这属于「音视频联合caption分流化」这一2026年趋势的一支：LTX-2 走「全音景统一描述」路线（一段文本描述整个声景），Script-a-Video 走 factorized streams（分解为独立流），Foley-Omni 则是最简洁的固定三字段方案。三字段的优势是schema极简、与源分离模型天然对应、易于验证；代价是表达力受限——无法描述字段内多个声源的相对位置、响度层次、时间先后。
[不确定] 未给出 caption 的平均长度/词数统计，未说明 [WORDS] 字段是逐字转写还是内容概述（从 WER 作为评测指标推断应为逐字转写文本），未说明是否包含镜头运动、视觉风格等视觉侧结构化标签（从模板看不包含——视觉信息完全由 CLIP/Synchformer 特征承载，不进入文本条件）。

### 音视频联合caption结构（是否同时覆盖视觉+听觉轨道、是否分流为独立字段，如LTX-2全音景描述、Script-a-Video factorized streams、Foley-Omni三字段）

这是本工作最具代表性的贡献，也是 note 中特别标注的「三字段」所指。
【schema 定义】音视频联合标注 Ŝ = ([WORDS] 语音内容, [AUDIO] 音效事件描述, [MUSIC] 音乐描述)。三字段并列、独立可空，共同描述同一片段的完整听觉内容。
【是否覆盖视觉轨道】不覆盖。这是一个重要的设计选择：schema 只结构化「听觉」侧，视觉侧不做文本描述，而是直接以 CLIP（语义）+ Synchformer（时序）双路特征注入模型。理由是视频在本任务中是条件而非生成目标，无需用文本作为中介表示；但标注生成过程是「视听联合」的——Gemini 同时看视频帧和听音轨来产出这三个听觉字段，视觉信息以隐式方式参与了标注决策（例如通过画面判断某个声音是否属于画内声源）。
【是否分流为独立字段】是，且这正是核心。与「全音景单段描述」相比，分流带来三个可操作的好处：(a) 训练时可对不同字段施加不同的条件强度/dropout；(b) 推理时用户可只填某几个字段做定向控制（如只要音效不要音乐）；(c) 清洗时可对每个字段做独立的声学验证并单独置空，而整段描述只能整体保留或整体丢弃，粒度粗得多。
【与源分离的同构性】三字段 ↔ Bandit 的三个 stem ↔ 三类生成任务（TTS/TTA/TTM）形成严格一一对应，这种跨环节的结构一致性是该设计优雅之处：标注schema、验证工具、任务定义、评测维度全部对齐在同一套三分类上。
【评测端的体现】V2ST-Bench 按字段共存模式组织：语音+音效150、语音+音乐120、三者齐全30，直接测试模型在多字段同时非空时的联合生成能力。

### 对白转写与说话人属性标注（ASR转写+说话人身份/语言/口音/情绪） ⚠️

对白转写以 [WORDS] 字段承载，但实现路径不常规——不使用独立 ASR，而是由 Gemini 2.5 Pro 在多模态标注中直接产出语音内容文本。这样做的好处是转写与存在性判别、与其他两字段的描述在同一次调用中完成，成本与流程都更简洁；代价是转写精度不如专用 ASR（Whisper 等），且论文未做转写质量评估。
【质量保障】[WORDS] 字段同样受 Bandit speech stem 的 −35 dB 能量门控约束——若分离出的语音轨能量不达标，说明画面中人物张嘴但实际无可闻语音（或语音被淹没），该字段被置空，可有效防止 ASR/VLM 在无语音处产生幻觉转写。
【说话人属性标注】[不确定] 论文未描述任何说话人身份 ID、性别、年龄、语言、口音、情绪的结构化标注。三字段 schema 中 [WORDS] 只承载语音内容文本，不含说话风格/音色描述。这意味着模型的音色控制能力主要依赖视觉条件（从说话人面部推断音色），而非文本属性标注——这与 VisualTTS 的任务设定一致，但也限制了对语音风格的显式可控性。
【间接证据】评测使用 WER 作为核心指标（V2ST-Bench 7.59、GRID 15.3、对照 Ground Truth 8.03），说明 [WORDS] 字段是逐字转写文本而非内容摘要，否则 WER 无法计算。

### 几何与结构化标注（相机参数、深度、3D point tracks、动作标注、显式状态） ⚠️

[不确定] 论文未涉及任何几何或3D结构化标注：无相机参数（内外参、轨迹）、无深度图、无3D point tracks、无人体姿态或动作标注、无显式状态标注。
最接近「结构化视觉信息」的是 Synchformer 抽取的时序同步特征与 CLIP 的场景语义特征，但这些是隐式的神经网络表征而非可解释的几何标注。motion score 是唯一的显式运动量化，且仅用于过滤阈值判断，不作为训练标注保留。
这一缺位对本任务影响不大：音频生成不需要精确的3D几何理解，声画对齐主要依赖2D时序事件线索。但对更精细的空间音频生成（如双耳/环绕声、声源方位随镜头运动变化）而言，缺乏相机与3D标注会成为瓶颈——本文生成的是单声道/立体声混合音轨，未涉及空间音频。

### 合成数据构造（受控扰动/编辑构造训练对，如InstructAV2AV） ⚠️

[不确定] 论文未描述任何合成数据构造环节。pipeline 中不存在受控扰动、音轨替换、时间偏移构造负样本、或类似 InstructAV2AV 的编辑指令对构造。
唯一涉及「音频拆解重组」的是 Bandit 源分离，但其用途是验证标注（能量门控）而非构造训练样本——论文未提及把分离得到的单类 stem 反过来当作 TTA/TTM 的合成训练数据（这本是一个自然的数据增广思路：从 V2ST 数据中分离出纯音效轨可扩充 V2A 训练集，但文中未采用或未提及）。
数据全部为真实采集的音视频对 + 模型生成的文本标注，属于「真实数据 + 模型标注」范式，不含生成式数据合成。

### 人工介入程度（人工标注、人工质检、模型初筛+人工复核） ⚠️

人工介入程度低，且集中在阈值标定与评测集把关两处，训练数据标注全自动。
【阈值标定环节】声学后验证的 −35 dB RMS 能量门限通过「manual inspection of a small validation subset」（对小规模验证子集的人工检视）确定。这是典型的「人工定标、机器执行」模式——人只参与超参数选择，不参与逐样本判断。Table 7 中其余六项阈值（480p、1 Mbps、[0.1,3.2]、0.6、0.3、0.2）的确定依据论文未说明，推测同样有经验性的人工调参成分。
【评测集人工复核】V2ST-Bench 的300条片段经人工审核，审核维度明确为三项：audiovisual consistency（音视频一致性）、annotation accuracy（标注准确性）、suitability for mixed soundtrack evaluation（是否适合作混合配乐评测）。这属于「模型初筛 + 人工复核」模式，但仅施加于300条评测样本，规模极小。
【主观评测】使用 A-MOS / S-MOS / T-MOS 三项主观意见分，需要人类听测评分。[不确定] 未说明评测者人数、招募方式、评分细则与一致性检验。
【训练数据】约2.0M条训练标注全部由 Gemini + Bandit 自动产出，无人工复核环节，也未提及抽样质检的准确率报告。

## 音视频对齐

### 音视频同步检测方法（口型同步、事件对齐） ⚠️

同步检测是过滤阶段「对齐维度」的一半，且同一技术栈贯穿清洗、建模、评测三个环节，一致性很强。
【清洗阶段】使用 Synchformer 计算 sync score，阈值 ≥0.2，剔除「unreliable synchronization」（同步不可靠）的片段。Synchformer 是基于 Transformer 的音视频同步检测模型，相比经典 SyncNet 的优势是不局限于口型同步，能处理通用音视频事件的时序对齐（如撞击声与撞击画面），这对以音效为重要生成目标的本任务是必要的——若用 SyncNet，非人脸场景的音效数据会被大量误杀。
【建模阶段】Synchformer 特征被直接用作模型的时序条件输入，通过一条 additive path（记为 Z_sync）注入 DiT。这意味着「用于过滤的同步表征」与「用于生成的同步条件」是同一个，过滤标准与模型能力天然对齐——通过过滤的样本正是模型能有效利用其同步信号的样本。这个设计在同类工作中值得注意。
【评测阶段】使用 DeSync 指标（越低越好）衡量生成音频与视频的时序对齐质量。Foley-Omni 在 V2ST-Bench 上 DeSync 0.16，非常接近 Ground Truth 的 0.14，显著优于级联基线 MMAudio+CosyVoice 3+AudioX 的 0.85 与 MMAudio+LipVoicer+AudioX 的 0.26。
【消融证据】Table 6 显示移除 Z_sync 同步附加通路后，IB 从 0.26 降至 0.22、V2ST WER 从 7.59 升至 12.40、FD_VGG 从 1.57 升至 2.21，三项指标同时劣化，证实同步特征通路对音视频一致性与语音质量均有实质贡献。
【口型同步】[不确定] 论文未在清洗阶段部署专门的唇同步检测器（如 SyncNet confidence），VisualTTS 支路的唇同步质量依赖数据集本身（GRID/LRS2 天然是唇读数据集，同步性有保证）与 Synchformer 的通用同步判据。这在纯语音场景下可能不如专用唇同步指标精确。

### 同步检测具体指标与阈值（SyncNet/Synchformer/LSE/自研，阈值数值，如MOVA LSE-D≤9.5且LSE-C≥4.5、SkyReels-V4 SyncNet |offset|≤3∧conf>1.5） ⚠️

【清洗端具体数值】
  · Synchformer sync score ≥ 0.2 —— 时序同步可靠性门槛
  · ImageBind (IB) score ≥ 0.3 —— 音视频语义一致性门槛
  两项并列于 Table 7 的 Alignment 维度，均为独立硬阈值，通过条件为「同时满足」。
【评测端指标】
  · DeSync（越低越好）—— 时序对齐误差。Foley-Omni 0.16 vs Ground Truth 0.14 vs 级联基线 0.85/0.26。
  · IB score（越高越好）—— 音视频语义一致性。Foley-Omni 0.26 vs Ground Truth 0.36 vs 基线 0.25/0.16。注意生成结果的 IB 0.26 与清洗阈值 0.3 处于同一量级，说明 0.3 这个门槛在真实数据上已属中等偏严。
  · CLAP（越高越好）—— 音频与文本描述的语义相关性。Foley-Omni 0.27 vs GT 0.30。
  · WER（越低越好）—— 语音可懂度。V2ST-Bench 上 Foley-Omni 7.59，甚至低于 Ground Truth 的 8.03（因生成语音比真实录音更清晰、无环境干扰）；GRID 子集 15.3。
  · 主观 MOS 三项：A-MOS 3.92（音质）、S-MOS 4.13（语义一致）、T-MOS 4.14（时序同步），对照 GT 的 4.33/4.37/4.42，差距最小的是 S-MOS 与 T-MOS。
【阈值宽严评估】Synchformer 0.2 与 ImageBind 0.3 相比业界同类（如 UniVerse-1 的 SyncNet 严格阈值、MMAudio 系工作的对齐筛选）属于中等偏宽松的设定，倾向于保留数据量。论文未提供阈值敏感性分析。
[不确定] 阈值的确定依据未说明（是否经过网格搜索、是否有保留率-性能权衡曲线均无报告）。Synchformer/ImageBind 分数的具体计算配置（窗口长度、特征聚合方式）也未交代，影响复现精度。

### 时序同步vs语义同步的分离处理（时间对齐与内容语义匹配作为两个独立过滤条件）

明确做了分离处理，这是本pipeline在对齐维度上的一个精细设计点。
 Table 7 的 Alignment 维度并列两项独立指标、两条独立阈值：
  · ImageBind score ≥0.3 负责【语义一致性】—— 判断音频内容与视觉内容是否讲的是同一件事（画面是狗、音轨是狗叫 vs 画面是狗、音轨是无关背景音乐）。ImageBind 通过统一的跨模态 embedding 空间计算余弦相似度，捕捉的是「内容匹配」。
  · Synchformer sync score ≥0.2 负责【时序同步】—— 判断音频事件与视觉事件在时间轴上是否对齐（撞击画面与撞击声是否同一帧）。Synchformer 专为时序对齐检测训练，捕捉的是「时间对齐」。
【为何必须分开】两类错误在真实数据中独立发生且性质不同：后期配乐的视频语义可能一致（悲伤画面配悲伤音乐）但时序完全不对齐；画外音/旁白的时序可能对齐（配合剪辑节奏）但语义与画面不匹配；音画不同步的错帧素材则是语义对但时序错。用单一综合分会让这三类问题相互掩盖——语义分高可以补偿时序分低，导致坏样本蒙混过关。设置两条独立硬门槛则要求样本在两个维度上都合格。
【论文原文佐证】过滤条件的表述把二者并列为两个独立剔除理由：「weak audiovisual semantic consistency, or unreliable synchronization」——用 or 连接，确认是两个独立的淘汰条件。
【延伸到建模与评测】这种分离贯穿全流程：模型侧 CLIP 特征承载语义、Synchformer 特征（Z_sync 通路）承载时序，两路分开注入；评测侧 IB 分数测语义一致、DeSync 测时序对齐，两个指标分开报告。清洗、建模、评测三处保持同一套语义/时序二分，方法论上相当自洽。

### 音频质量过滤（SNR、静音检测与静音占比阈值、无音轨剔除、画外音源剔除、背景音乐分离） ⚠️

音频质量过滤集中于单一指标 + 一道能量门控，设计简洁。
【主过滤器】Meta AudioBox Aesthetics 分数 ≥0.6。这是 Meta 开源的无参考（no-reference）音频美学/质量评估模型，输出感知层面的综合质量分。选用学习型感知评估而非传统信号指标（SNR、THD、频谱平坦度）是2025–2026年音频数据清洗的方法演进——传统 SNR 无法识别过度压缩、削波、编码劣化、混音失衡等问题，而 AudioBox Aesthetics 与人耳判断相关性更高。
【静音处理】过滤阶段第一条剔除条件即为 silence（静音）。实现上很可能由 AudioBox Aesthetics 低分自然覆盖（静音片段美学分极低），论文未描述独立的静音检测器或静音占比阈值。无音轨样本自然被排除在外（本任务音轨是必需的监督目标）。[不确定] 未给出静音判定的能量阈值、静音时长占比上限等具体参数。
【能量门控（字段级）】声学后验证阶段对分离后的每个 stem 施加 E(a_c) > −35 dB 的 RMS 能量门槛。这不是片段级过滤而是字段级置空——不达标只删除该字段标注，片段本身仍保留用于其他字段的训练。这个粒度设计很关键：一个只有语音没有音乐的片段仍是合格的 VisualTTS/V2ST 训练样本，不应整条丢弃。
【背景音乐分离】使用 Bandit（cinematic audio source separation 模型）做 speech/effects/music 三路分离，但用途是验证标注而非清洗音轨——分离结果不作为训练目标，模型仍学习生成完整混合音轨。这与一些 V2A 工作「分离掉背景音乐再训练」的做法不同，Foley-Omni 反而要求模型能生成含音乐的完整配乐。
【画外音源剔除】[不确定] 论文未描述专门的画外音（off-screen sound）、旁白、解说的识别与剔除机制。ImageBind ≥0.3 的语义一致性门槛可以部分过滤掉与画面无关的旁白，但不是针对性设计。

### 语音/音效/音乐的分类与分别处理策略

语音/音效/音乐的三分类是贯穿整个系统的组织主线，不只是清洗环节的分类，而是任务定义、标注schema、验证工具、训练课程、评测基准五个层面共用的同一套划分。
【分类工具双路】
  · 语义判别路径：Gemini 2.5 Pro 多模态判断三类组件各自是否 physically present（detect-then-describe）。
  · 声学验证路径：Bandit 源分离得到三个 stem，按 −35 dB RMS 能量门控做二次确认与否决。
  Bandit 的选型与三分类严格同构（cinematic audio source separation 天然分 speech/effects/music），避免了通用源分离模型类别不对应的问题。
【分别处理策略】
  · 标注层：三类各占一个独立字段 [WORDS]/[AUDIO]/[MUSIC]，可空。
  · 数据层：三类对应三个单任务组 TTS(1,253h) / TTA(912h) / TTM(139h)，并有对应的视频条件版本 VisualTTS(1,980h) / V2A(403h)（音乐无视频条件单任务组）。
  · 训练层：Stage 1 先在三类单任务上分别打基础（约0.7M文本-音频对，5 epochs），Stage 2 引入视觉条件（V2A + VisualTTS，3 epochs），Stage 3 才做三类联合生成（V2ST，2 epochs）。即先分后合的课程设计。
  · 生成层：三类不是分别生成后混音，而是在共享隐空间联合生成——分类用于条件控制与数据组织，不用于推理时的模块拆分。
  · 评测层：V2ST-Bench 按共存组合分三档（语音+音效、语音+音乐、三者齐全），并用不同指标侧重不同类别（WER 测语音、CLAP/IB 测音效语义、A-MOS 综合）。
【配比失衡】三类投入极不均衡：语音相关约66%、音效相关约27%、音乐仅2.8%（139小时TTM，且无独立的视频条件音乐任务组）。这是明显的取舍——论文的核心卖点是「speech intelligibility」的提升，音乐能力相对是配角。

## 训练配合

### 多阶段训练课程与数据课程调度（阶段划分依据：分辨率/时长/质量分/模态；低清→高清、图像→视频、短→长） ⚠️

采用三阶段渐进式课程，划分依据是【模态复杂度 + 任务复合度】，而非文生视频常见的分辨率/时长维度。整体路线为「文本条件 → 加入视觉条件 → 多组件联合」。
【Stage 1：文本驱动奠基】任务 TTA + TTS + TTM，5 epochs，约0.7M文本-音频对。目的是先让模型在纯文本条件下掌握三类音频各自的生成能力，建立音频先验。此阶段无视频输入。epochs 最多（5轮），说明基础音频生成能力的建立被给予最多训练预算。
【Stage 2：引入视觉条件】任务 V2A + VisualTTS，3 epochs。加入视频条件通路（CLIP 语义特征 + Synchformer 同步特征 Z_sync），学习「视觉→音频」的语义映射与时序对齐。此阶段仍是单类音频任务，只是条件模态变复杂。
【Stage 3：联合配乐微调】任务 V2ST，2 epochs，使用216小时V2ST数据。学习在同一片段内联合生成多类音轨并保持相互协调。epochs 最少，属于轻量微调，符合「小规模高质量数据做最终对齐」的常规做法。
【课程有效性的量化证据】Table 6 消融显示，若跳过课程直接单阶段联合训练，性能全面崩塌：V2ST WER 从 7.59 恶化到 29.29（近4倍劣化）、GRID WER 从 15.3 到 27.4、IB 从 0.26 降到 0.24、FD_VGG 从 1.57 升到 1.73。语音可懂度受损最严重，说明语音这类结构性最强、最不容错的模态尤其依赖前置的单任务预训练——直接混合训练时语音能力被音效/音乐任务的梯度淹没。这是本文关于数据/训练课程最有力的一条实证。
[不确定] 各阶段的学习率、batch size、总训练步数、硬件配置与训练时长均未在可获取内容中披露。

### 数据配比随训练阶段的变化（预训练/退火annealing/SFT高质量子集） ⚠️

配比随阶段的核心变化是「任务集合的扩张 + 回放机制的引入」。
【Stage 1 → Stage 2】数据从纯文本-音频对（TTA 912h + TTS 1,253h + TTM 139h，约0.7M对）切换为视频-音频对（VisualTTS 1,980h + V2A 403h）。任务集合完全替换而非叠加。
【Stage 2 → Stage 3】切换到 V2ST 数据（216小时，internal + SpeakerVid），但关键是引入了显式的经验回放（replay）机制以缓解灾难性遗忘：在 V2ST 微调数据之外，混入前序每个单任务域各 100 小时的数据。按此计算，Stage 3 的混合配比约为 V2ST 216小时 + 五个前序任务域各100小时（合计约500小时），即目标任务约占30%、回放数据约占70%——回放比例相当高，反映出作者对遗忘问题的重视程度。
【回放的必要性论证】这与 Table 6 的单阶段训练消融互为印证：多任务音频生成中不同任务相互干扰严重（尤其语音 vs 非语音），既不能一开始就混合训练（Stage 消融证明），也不能后期完全脱离前序数据（故需回放），只能用「先分后合 + 高比例回放」的方式平衡。
【退火/annealing】论文未使用 annealing 术语，但 Stage 3 用小规模（216h）高质量 V2ST 数据 + 递减的 epochs（5→3→2）做最终微调，功能上等价于退火阶段。
[不确定] 各阶段内部不同数据集之间的采样权重（是按小时数自然比例还是加权采样）未披露；回放数据是随机抽取100小时还是按质量分挑选也未说明。

### 后训练数据（SFT精选集规模与筛选标准、偏好对数量与标注方式、reward model训练数据） ⚠️

本文的训练流程止于三阶段监督式课程（Stage 3 的 V2ST 微调即为最终阶段），未包含独立的 RLHF/DPO 后训练环节。
【最接近SFT的环节】Stage 3 可视为SFT性质的阶段：使用规模最小（216小时）、任务最贴合最终目标（V2ST完整配乐）的数据做2 epochs轻量微调，并配以每域100小时回放防遗忘。筛选标准即为通过完整清洗pipeline（六项过滤阈值 + Gemini标注 + Bandit能量验证）且含多个音频组件的样本。
【偏好数据】[不确定] 未见任何偏好对（preference pairs）构造、未训练 reward model、未使用 DPO/PPO 类偏好优化。A-MOS/S-MOS/T-MOS 主观评分仅用于最终评测，未回流为训练信号。
【人工精选集】V2ST-Bench 的300条经人工三维度复核的样本是全流程中唯一的人工精选集，但明确用于评测而非训练。
对照工业级模型（Veo 3、Sora 2 等）普遍有的偏好对齐阶段，Foley-Omni 作为学术工作在后训练维度是空白的，这也意味着其在音频美学、创作意图贴合等主观维度上仍有提升空间（A-MOS 3.92 vs GT 4.33 是三项主观分中差距最大的一项）。

### 数据处理基础设施与吞吐（NeMo Curator/Data-Juicer/自研，GPU加速比，处理规模，成本） ⚠️

[不确定] 论文未披露任何数据处理基础设施信息：未提及 NeMo Curator、Data-Juicer 或自研分布式清洗框架，未给出 GPU 加速比、处理吞吐（小时/GPU天）、集群规模或清洗成本。
可推断的成本结构：pipeline 中最昂贵的环节应是 Gemini 2.5 Pro 的 API 调用——对约2.0M条视频片段做多模态（视频帧+音轨）标注，按商用API定价属于显著开销，但论文完全未讨论调用量、成本控制策略（如是否先小规模验证prompt、是否有降级模型分流）。这也是使用闭源API做大规模标注的工作普遍回避的话题。
其余环节（分辨率/码率解析、AudioBox Aesthetics 打分、ImageBind/Synchformer 推理、Bandit 源分离）均为可批量GPU推理的轻量模型，总算力需求相对Gemini标注应低一个量级，但同样无数据披露。
开源侧 GitHub 仓库提供了视觉特征（CLIP + Synchformer）的预处理抽取脚本，属于最小可用的工具支持，但不构成完整的清洗基础设施。

## 效果对比

### 数据策略消融的量化影响（区分：过滤严格度ablation / caption密度风格ablation / 数据配比ablation，及对应评测指标） ⚠️

论文的消融实验（Table 6）聚焦于【训练课程】与【模型结构】两类，未做纯数据策略消融。
【已做的消融 —— 定量结果】
  指标口径：FD_VGG↓（VGGSound上的Fréchet Distance，音频保真度）、WER_GRID↓（GRID上语音可懂度）、IB_V2ST↑（V2ST-Bench音视频语义一致性）、WER_V2ST↓（V2ST-Bench语音可懂度）。
  · 完整模型：FD_VGG 1.57 / WER_GRID 15.3 / IB_V2ST 0.26 / WER_V2ST 7.59
  · 单阶段训练（去掉三阶段课程）：1.73 / 27.4 / 0.24 / 29.29 —— WER_V2ST 恶化3.9倍，WER_GRID 恶化79%，四项全面劣化。属于训练课程/数据调度消融，可归入广义的数据配比消融：证明「数据分阶段投喂」相对「一次性混合」有决定性影响，且语音任务受损最烈。
  · 移除 Z_sync 同步附加通路：2.21 / 18.9 / 0.22 / 12.40 —— IB 从0.26降至0.22（音视频一致性劣化最直接）、FD_VGG 从1.57升至2.21（音频保真度劣化41%）、WER_V2ST 从7.59升至12.40。证明 Synchformer 时序特征通路对三个维度均有实质贡献。
【未做的消融 —— 明显空白】[不确定]
  · 过滤严格度消融：六项阈值（480p / 1Mbps / motion[0.1,3.2] / AudioBox 0.6 / IB 0.3 / Sync 0.2）无一做过阈值敏感性分析，无「宽松过滤 vs 严格过滤」的性能对比曲线。
  · caption 密度/风格消融：未对比三字段结构化标注 vs 单段自由描述（如 LTX-2 式全音景描述）的效果差异——这本是验证其核心贡献「三字段schema」价值的最直接实验，缺失较可惜。
  · 声学后验证消融：未做「有/无 Bandit 能量门控」的对比，因而无法量化视觉幻觉纠偏带来的实际收益，也无法验证 −35 dB 阈值的合理性。
  · 数据配比消融：未对比不同任务组小时数配比（如提高音乐占比）的影响。
  综合看，本文对模型结构与训练课程的消融是充分的，但对其自称的核心贡献之一「数据清洗pipeline」几乎没有做消融验证，pipeline 的有效性主要靠方法论叙述而非实验证据支撑。

### 质量vs数量的证据（小而精数据超越大而杂的案例） ⚠️

无直接的「小而精超越大而杂」对照实验，但有若干间接体现该理念的设计。
【间接证据一：Stage 3 的小数据高效微调】最终的 V2ST 能力仅由216小时数据（占总量4.4%）在2个epoch内习得，且达到 WER 7.59（优于 Ground Truth 的 8.03）、DeSync 0.16（接近 GT 的 0.14）。这说明在前序大规模阶段打好基础后，目标任务只需极小规模但高度匹配的数据即可对齐——是「小而精」的一种体现，但缺乏「用更大规模低质V2ST数据训练」的对照组。
【间接证据二：pipeline 的严过滤取向】六项过滤阈值 + 字段级能量门控的设计本身体现了质量优先的取向，尤其声学后验证宁可置空字段（减少监督信号）也不保留可能错误的标注，是明确的「宁缺毋滥」。
【间接证据三：评测集的严格人工复核】300条评测样本经三维度人工把关，规模远小于常见基准但质量可控。
【反向证据】数据构成上又体现了「量」的追求：VisualTTS 投入1,980小时（40%）以支撑语音能力，说明在语音这一关键能力上作者选择了大规模数据路线。可以说本文是「基础能力靠量、任务对齐靠质」的混合策略。
[不确定] 论文未做任何数据规模缩放实验（如用50%数据训练的性能对比），因此无法定量论证质量与数量的权衡关系。

### 训练数据domain分布与评测基准类目体系的对齐关系（如VABench七大类） ⚠️

训练数据的组织维度与评测基准的类目体系高度对齐，且对齐轴是「音频组件类型」而非视觉domain——这是 Foley-Omni 与文生视频模型（如对齐 VBench/VABench 视觉七大类）的根本差异。
【对齐关系】
  · 训练侧任务组：TTS / TTA / TTM（三类单任务）→ VisualTTS / V2A（视频条件单任务）→ V2ST（多组件联合）
  · 标注侧字段：[WORDS] / [AUDIO] / [MUSIC]
  · 验证侧 stem：Bandit 的 speech / effects / music 三路分离
  · 评测侧 V2ST-Bench 分档：语音+音效 150条 / 语音+音乐 120条 / 三者齐全 30条
  四者严格同构于同一套 speech-SFX-music 三分类，形成闭环。这种全链路对齐使得「训练数据里怎么标的，评测就怎么测」，一致性很强。
【评测指标与类目的对应】WER 对应 [WORDS] 语音字段、CLAP/IB 对应 [AUDIO] 音效的语义匹配、DeSync 对应跨类的时序对齐、A/S/T-MOS 做整体主观补充。
【对齐中的失衡】V2ST-Bench 三档中语音100%出现（150+120+30 全含语音），而纯音效+音乐无语音的组合完全缺席——评测基准的语音中心倾向与训练数据66%语音占比的失衡一致，二者互相强化。这意味着基准在「非语音场景的完整配乐」（如自然纪录片、动作场面的音效+配乐）上缺乏覆盖，模型在该场景的能力未被检验。
【外部基准对齐】单任务性能在通用基准上评测：VGGSound（FD_VGG，V2A标准基准）、GRID（WER，唇同步语音标准基准），保证与专家系统可比。论文称在单任务上「achieves competitive performance with expert systems」。
[不确定] 未与 VABench、AudioCaps 官方榜、Kling-Foley 评测集等其他基准做交叉对齐分析；V2ST-Bench 与训练集是否严格无重叠也未明确声明。

## 不确定字段

以下字段的调研信息部分不确定（⚠️ 标注来源）：

- data_scale
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
- dialogue_transcription_attributes
- geometric_structured_annotation
- synthetic_data_synthesis
- human_in_loop
- av_sync_detection
- sync_metric_and_threshold
- audio_quality_filtering
- multi_stage_curriculum
- stage_data_mixture
- post_training_data
- data_infra_throughput
- data_ablation
- quality_over_quantity_evidence
- benchmark_taxonomy_alignment
