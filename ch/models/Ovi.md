# Ovi（论文《Ovi: Twin Backbone Cross-Modal Fusion for Audio-Video Generation》，arXiv:2510.01284；后续迭代版本 Ovi 1.1；音频塔单独命名为 Ovi-Aud）

> 主题: 视频生成模型（含音视频同时生成）的数据处理：数据清洗流程、数据分布、打标方式

[← 返回首页](../index.md)

**目录**: [基本信息](#基本信息) · [数据规模与分布](#数据规模与分布) · [清洗流程](#清洗流程) · [打标方式](#打标方式) · [音视频对齐](#音视频对齐) · [训练配合](#训练配合) · [效果对比](#效果对比)

## 基本信息

### 名称

Ovi（论文《Ovi: Twin Backbone Cross-Modal Fusion for Audio-Video Generation》，arXiv:2510.01284；后续迭代版本 Ovi 1.1；音频塔单独命名为 Ovi-Aud）

### 发布机构/公司

Character AI（主体，作者 Chetwin Low、Weimin Wang，Weimin Wang 为 Project Lead）与 Yale University（耶鲁大学，作者 Calder Katyal）联合发布。致谢中提到 Yi Cui、Manav Shah、Diego De La Torre 参与数据准备工作。

### 发布时间（技术报告/论文/开源时间）

论文成稿日期 2025年9月29日，2025年10月1日在 arXiv 公开（arXiv:2510.01284，v1），HuggingFace 模型页标注论文发布日 2025年9月30日；同期在 GitHub（character-ai/Ovi）开源 11B 模型权重与推理代码。2025年11月10日发布 Ovi 1.1 更新（原生 960×960 训练、10 秒生成、数据集扩容一倍）。截至 2026年7月未见 Ovi 2.0 或训练代码发布。

### 类型（模型/数据集/工具链/评测基准）

模型（开源的文本/文本+图像到「音频+视频」一次性联合生成模型，T2AV / I2AV），并附带开源推理工具链（推理脚本、Gradio 应用、多卡序列并行推理、fp8/qint8 量化、ComfyUI 集成）。不是数据集，也不是评测基准（评测借用他人提出的 Verse-Bench）。

### 开源程度（权重/代码/数据/pipeline各自是否开源）

属于「权重+推理代码开源、数据与数据 pipeline 代码闭源」的典型模式。
【已开源】(1) 模型权重：11B（HuggingFace 页面标注约 12B 参数、BF16）完整 checkpoint，含三个版本 720x720_5s、960x960_5s、960x960_10s，托管于 HuggingFace chetwinlow1/Ovi；(2) 推理代码：文本/文本+图像输入、Gradio App、多 GPU（含序列并行）推理、权重下载脚本；(3) 社区贡献的 fp8（@rkfg）与 qint8（@gluttony-10）量化权重，24GB 显存可跑；(4) 论文完整披露了数据处理 pipeline 的方法与关键阈值（这一点比多数同类模型开放）。
【未开源】训练脚本（README 的 Todo List 中「Training scripts」仍未勾选）、训练数据本身（内部音视频语料与内部音频语料）、数据处理 pipeline 代码、所用 MLLM 打标模型的身份、各级过滤的定量保留率、RL 后训练细节。
【许可】Apache 2.0（相对宽松，可商用）。依赖组件：视频分支权重来自 Wan2.2-TI2V-5B，文本编码器 T5 与视频 VAE 解码器来自 Wan，音频 VAE 来自 MMAudio（均为开源模型）。

### 是否支持音视频同时生成，以及实现方式（原生联合/级联/MoE融合）

支持，且是原生联合生成（one-pass joint AV generation），不是级联也不是 MoE 融合，作者称之为「twin backbone（孪生/双骨干）blockwise cross-modal fusion」。
(1) 对称孪生 DiT：音频塔与视频塔架构完全相同（Model Dim 3072、FFN Dim 14336、24 heads、head dim 128、30 个 block，每个 block 各有 30 层 Self-Attn / Text Cross-Attn / AV Cross-Attn）。视频分支由 Wan2.2 5B 初始化，音频分支为同架构从零训练的 5B，合计约 11B。
(2) 逐 block 双向跨模态注意力：每个 transformer block 内音频流 attend 视频流、视频流反向 attend 音频流，同步线索贯穿全网络。因两塔隐维一致，无需任何投影层（对比 UniVerse-1 需要插块+投影+辅助语义对齐损失）。
(3) 时序对齐靠 scaled-RoPE：视频 latent 31 帧、音频 latent 157 token（16kHz×5s/512），将音频分支 RoPE 频率按 31/157≈0.197 缩放，使跨模态 RoPE 亲和矩阵对角线对齐。
(4) 单一冻结 T5 编码器编码「合并 prompt」（视觉描述 + <S>台词<E> + 音频描述），同一份文本嵌入分别与音频塔、视频塔做 cross-attention，统一跨模态语义控制。
(5) 训练目标为 flow matching，两模态共享同一 timestep t、各自独立噪声，总损失 L=0.85·L_video+0.15·L_audio；无任何显式同步损失、无人脸 mask、无 post-hoc 对齐、无辅助同步模块。推理时两分支共用同一 ODE 求解器（UniPC）。
(6) 能力：Ovi 初版 5 秒 720×720@24fps；Ovi 1.1 为 10 秒 960×960@24fps，支持 9:16、16:9、1:1 等多种宽高比，同时输出对白、音效与背景音乐。

### 调研信息来源列表（论文/技术报告/官方文档/新闻的URL，标注每条来源的性质：官方一手/同团队旁证/第三方报道）

1) 官方一手｜论文全文 https://arxiv.org/abs/2510.01284 与 https://arxiv.org/pdf/2510.01284 （《Ovi: Twin Backbone Cross-Modal Fusion for Audio-Video Generation》，第3节 Data Processing Pipeline 为数据侧唯一权威披露）
2) 官方一手｜GitHub 仓库与 README https://github.com/character-ai/Ovi （开源范围、Todo List、Ovi 1.1 更新说明、prompt 格式）
3) 官方一手｜HuggingFace 模型卡 https://huggingface.co/chetwinlow1/Ovi （参数量、基座模型、许可、分辨率/时长版本）
4) 官方一手｜项目主页 https://aaxwaz.github.io/Ovi/ （demo 与代码入口）
5) 同团队旁证｜HuggingFace Papers 页 https://huggingface.co/papers/2510.01284
6) 第三方报道｜CSDN 中文解读 https://blog.csdn.net/SuaniCommunity/article/details/154737163 与腾讯新闻 https://view.inews.qq.com/a/20251113A02WA900 （复述论文数据 pipeline 四步与 720×720 门槛，无新增一手信息）
7) 第三方旁证｜腾讯云开发者社区技术详解 https://cloud.tencent.cn/developer/article/2584843
8) 第三方平台｜WaveSpeed AI 托管页 https://wavespeed.ai/models/character-ai/ovi/text-to-video （商用可得性）

## 数据规模与分布

### 训练数据量级（视频条数/小时数/token数，预训练与SFT分开） ⚠️

论文只给出量级描述，未给精确数字。
【音视频配对语料】「millions of videos」（数百万条视频），全部为内部（in-house）音视频语料；按每条 121 帧 @24fps ≈ 5.04 秒估算，若为 300 万条则约 4200 小时量级，但论文未确认条数，此换算为推断[不确定]。
【纯音频语料】预训练阶段「hundreds of thousands of hours of raw audio」（数十万小时原始音频），以人声语音为主，波形长度最长 12 秒；微调阶段为 5.04 秒定长波形，混入 VGGSound / AudioSet / WavCaps 公开音效数据 + 从内部音视频语料中抽取的音轨。
【训练步数侧推算】音频预训练 50k 步 × batch 2880 ≈ 1.44 亿样本次；音视频融合训练 40k 步 × batch 768 ≈ 3072 万样本次（为 epoch 换算参考，非数据条数）。
【Ovi 1.1】README 明确「Dataset includes 100% more videos」，即音视频数据集规模较初版扩大一倍，且改为原生 960×960 分辨率数据训练；绝对数值未公布[不确定]。
预训练与 SFT 的严格拆分数字未公开[不确定]。

### 数据来源构成（自有/公开数据集/网络爬取/授权采购/合成数据） ⚠️

三类构成，界限清晰：
(1) 内部自有音视频配对语料（internal audio-video corpus）——占核心地位，论文称「composed of human and nonhuman data from diverse contexts」（涵盖有人与无人的多样场景），来源渠道未披露[不确定]。
(2) 内部纯音频语料（internal collections）——用于音频塔从零预训练，以人类语音为主，强调语言多样性、韵律与音色变化。README 称「high quality in-house audio datasets」。Character AI 作为对话陪伴产品公司，推测其内部语音资源与产品语音数据相关，但论文未说明[不确定]。
(3) 公开数据集——仅用于音频微调阶段补充音效能力：VGGSound（Chen et al., 2020）、AudioSet（Gemmeke et al., 2017）、WavCaps（Mei et al., 2024）。
(4) 无授权采购数据的说明，无合成数据构造环节的描述。
另外模型层面复用了大量开源资产（Wan2.2 5B 视频权重、Wan 的 T5 与视频 VAE、MMAudio 的 16kHz 音频 1D VAE、BigVGAN 声码器），可视为「间接继承了 Wan2.2 预训练语料的分布」。

### 数据合规与溯源（授权数据占比、rights-cleared数据集、C2PA等） ⚠️

[不确定]。论文与仓库均未讨论数据授权比例、rights-cleared 数据集、C2PA/水印溯源、版权合规审查等任何内容，也无 Model Card 层面的数据来源声明。可确认的仅有：微调期使用的三个公开音频数据集（VGGSound、AudioSet、WavCaps）均为学术研究许可数据集，其中 AudioSet/VGGSound 基于 YouTube 视频，本身带有来源合规争议；模型权重以 Apache 2.0 发布，但许可覆盖的是权重而非训练数据。内部语料的授权状况完全未披露。

### 片段时长分布与切分策略 ⚠️

策略是「统一定长、不做多时长分桶」，这是 Ovi 数据设计上最鲜明的特点之一。
【音视频数据】经场景检测切分后统一取 121 帧 @24fps 的定长片段（= 5.04 秒），初版训练与推理均为该长度；Ovi 1.1 扩展到 10 秒（对应约 241 帧 @24fps）[10秒对应的帧数为推断，不确定]。
【纯音频数据】双时长设计：预训练用最长 12 秒的变长波形（论文明确「use variable-length audio to maximize coverage of diverse acoustics」，靠长波形学到说话人音高、情绪的长程一致性）；微调用 padding 到精确 5.04 秒的定长波形，以对齐 121 帧视频的时长，避免进入音视频融合阶段时再做时序重适配。
【设计动机】论文说明对所有注意力层统一施加 scaled RoPE，就是为了「避免转入音视频微调阶段时的重适配、免于维护多套音频 RoPE 尺度」。
【局限自述】论文 Limitations 明确承认模型被限定在短 5 秒片段，分钟级叙事、镜头间转场、全局故事一致性均不在范围内，并提出未来用 chunk-wise causal 音频模型 + 以上一段末帧为条件的因果视频骨干来拼接多个 5 秒 chunk。

### 分辨率/宽高比分布与分桶策略 ⚠️

【筛选门槛】切分阶段要求片段分辨率严格大于 720×720 像素（论文原文「clips are greater than 720x720 pixel resolution」），低于该门槛直接丢弃。
【归一化策略】不做分辨率分桶，而是统一归一：打包（packing）前先去除视频中已有的黑边/margin，再在保持宽高比（maintaining aspect ratio）的前提下将帧缩放到固定像素总数 518400（= 720×720）。即约束的是「面积」而非「边长」，因而不同宽高比的样本能落到同一 token 数，天然支持 9:16、16:9、1:1 等多宽高比而无需分桶调度。
【Ovi 1.1】改为原生 960×960 分辨率数据训练（像素总数升至 921600），推理支持 960×960 及其等面积的各种宽高比。
【各宽高比的实际配比数字未公开】[不确定]。
【打包】最终视频按 24fps 抽帧后转为字节数组，音频转为原始 wave 字节，供训练侧高吞吐读取。

### 类别/domain分布与配比策略（人物、动作、场景、风格等比例控制与概念均衡） ⚠️

论文披露了一个明确且少见的「人物构成配比控制」机制，但未给出具体比例数字。
【显式配比控制】使用内部人脸检测模型（internal face detection model）将片段分为三类并「ensure an adequate mix」：单人视频（single-person）、多人视频（multi-person）、无人视频（person-free）。论文给出的动机是「让模型学会在多样上下文中生成视频，而不过拟合到某个特定子任务」——即避免像多数 A2V 模型那样退化为纯 talking-head 模型。这三类的具体百分比未公开[不确定]。
【语料定性构成】内部音视频语料被描述为「human and nonhuman data from diverse contexts」（人类与非人类内容、多样上下文）。
【间接的 domain 覆盖证据】论文 5.1 节的跨模态注意力可视化按内容类别组织，覆盖乐器演奏、鸟叫、火箭、动物、语音（多例）、直升机、体育等类别，可反推训练分布至少涵盖：人物对白、乐器/音乐、动物、载具/机械、体育运动、自然环境音等。
【音效侧 domain】通过引入 VGGSound / AudioSet / WavCaps，音效 domain 覆盖由这三个公开数据集的类目体系（AudioSet 632 类本体等）间接决定。
【未涉及】风格（写实/动画/CG）比例、场景类别比例、动作类别比例、概念均衡（concept balancing）等策略论文均未提及[不确定]。

### 音频类别分布与配比（语音/音效foley/音乐/环境音/静音的比例及控制策略）——AV模型独有维度 ⚠️

音频类别配比是 Ovi 数据设计的核心矛盾点，论文给出的是「分阶段偏置」而非静态比例。
【预训练阶段】「predominantly human speech」（以人类语音为压倒性主体），来自内部语料，强调语言多样性（linguistic diversity）、韵律（prosody）与音色变化（timbral variation）。此阶段几乎不含音效，目的是先把 TTS/说话人建模能力打牢。
【音频微调阶段】显式转向音效：「we emphasize modeling sound effects」，通过引入 VGGSound / AudioSet / WavCaps 补足 SFX；同时「为保持 TTS 能力并更好对齐下游目标」，额外掺入从内部音视频语料中抽取的音轨。这形成了一个「语音重→音效补充+语音保活」的两段式配比调度。三个公开音效集与内部语音音轨的混合比例未公开[不确定]。
【音视频融合阶段】音频侧完全来自配对视频的原生音轨，其语音/音效/音乐/环境音的自然比例由内部语料决定，未做人为再平衡的描述[不确定]。
【打标层面的类别分流】caption 阶段按「有语音 / 无语音」二分处理：含语音片段的音频描述强调说话人声学属性；非语音片段的音频描述则描述音效（sound effects）、背景音（background audio）、音乐元素（musical elements）——这实际是一个三类音频（语音/音效/音乐）的分类标注体系。
【静音处理】通过平均音量 ≥ −60 dB 的门槛剔除近似静音片段（见 audio_quality_filtering）。
【统一音频模型的必要性论证】论文在结果部分强调：真实世界视频常同时包含复杂音效与连贯语音，专用模型（纯 T2A 或纯 TTS）无法支持，因此必须训练一个统一的 T2A+TTS 音频塔——这是其音频类别混合训练策略的核心动机。
【音乐】BGM 生成能力被列为特性，但音乐数据是否单独引入未说明[不确定]。

### 叙事结构分布（单镜头vs多镜头、平均clip时长、镜头数分布、是否含原生音轨） ⚠️

【单镜头为主】通过场景检测（scene detection）切分，切分粒度本身保证每个 121 帧片段位于单一镜头内，因此训练样本几乎全为单镜头（single-shot）片段，不含镜头切换。这与论文 Limitations 中「inter-shot transitions 与全局故事一致性不在范围内」互为印证。
【平均 clip 时长】固定 5.04 秒（Ovi 1.1 为 10 秒），无长度分布——所有样本等长。
【镜头数分布】每样本恒为 1 个镜头。
【原生音轨】100% 含原生音轨——音视频配对语料的全部价值就在于原生同步音轨，且经过 SyncNet 与音量门槛过滤，无音轨或近似静音的片段已被剔除。这与部分模型「先剥离音轨再后期配音」的做法相反。
【叙事能力的来源】论文称模型能做「cinematic storytelling」，但叙事性来自 caption 中按时序交织的视觉事件与台词标注（chronology 被显式要求），而非多镜头数据。

### 语言/口音分布（多语种唇同步能力的数据基础） ⚠️

未给出定量分布。可确认的定性事实：
(1) 音频预训练数据「emphasize linguistic diversity, prosody, and timbral variation」，即语料在语言/口音层面有意做了多样性覆盖，但覆盖了哪些语种、各占多少未公开[不确定]。
(2) 打标阶段对含语音片段显式标注 accent（口音）属性，与 age、gender、pitch、prosody、emotion、speaking rate 并列写入音频描述，意味着口音是可控条件而非隐含变量。
(3) 评测层面仅报告英文 WER：在 Seed-TTS test-en 上 WER=0.035，未报告任何非英语指标，说明其可验证的语音能力以英语为主；社区实践与 demo 亦以英语对白为主[不确定]。
(4) 唇同步能力完全由数据驱动（无人脸 bbox、无 mask），因此多语种唇同步能力直接受限于内部语料的语种覆盖。

## 清洗流程

### 清洗漏斗整体结构（几级过滤、各级顺序）

Ovi 采用「双语料、各自成漏斗」的双轨设计，音视频配对语料是四级串行漏斗，纯音频语料是简化两级流程。

【A. 音视频配对语料 —— 四步 pipeline（论文 3.2 节明确编号）】
第1步 Splitting and filtering（切分与过滤）：场景检测切出 121 帧 @24fps 片段 → 分辨率必须 > 720×720 → RAFT 光流剔除静态视频并产出 motion score → LAION 美学预测器剔除低质数据 → 内部人脸检测模型做单人/多人/无人的构成配比控制。
第2步 Sync detection（同步检测）：SyncNet 产出 confidence 与 offset 标量 → 仅保留 |offset| ≤ 3 且 confidence > 1.5 → 并叠加平均音量 ≥ −60 dB 的音频门槛。
第3步 Captioning（打标）：MLLM 输入 7 帧均匀采样图像 + 完整音轨 → 输出「视觉事件叙述 + <S>台词<E> 交织 + 末尾 <AUDCAP>音频描述<ENDAUDCAP>」的合并 caption。
第4步 Packing（打包）：去黑边 → 保持宽高比缩放到 518400 像素（720×720 等面积）→ 按 24fps 抽帧转为字节数组 → 音频转 raw wave 字节。

【B. 纯音频语料 —— 简化流程】
按时长切两档（预训练 ≤12s、微调精确 5.04s）→ 用与音视频侧同一个 MLLM 产出音频转写（无语音则留空）与音频描述 → 进入训练。无 sync/美学/运动等视觉侧过滤。

【设计取向】整体漏斗把「同步性」置于最高优先级（论文明言「即使少量不同步数据也会损害唇同步能力，因此选择严格标准以最小化错配风险」），其次是分辨率与运动/美学的基础质量门槛，对去重、安全、水印/OCR 等维度则未着墨。

### 漏斗定量保留率（每级过滤的输入/输出量与最终保留率，如Apollo 27%） ⚠️

[不确定]。论文未给出任何一级过滤的输入/输出量、通过率或整体保留率，也没有 Apollo 式的 27% 类定量漏斗表。仅能定性判断筛选强度很高：SyncNet 条件（|offset|≤3 且 confidence>1.5）被作者自述为「strict criteria」（严格标准），叠加 >720×720 分辨率门槛、RAFT 静态剔除、美学评分剔除，实际保留率应显著低于常见的宽松 pipeline，但无数据支撑。Ovi 1.1 只披露最终数据集规模翻倍，未披露原始候选池规模。

### 镜头切分方法（PySceneDetect/自研模型/shot-aware splitting） ⚠️

使用「场景检测（scene detection）」将长视频切为镜头内片段，再从中截取 121 帧 @24fps 的定长 clip。论文只写了 scene detection 这一通用表述，未点名具体工具（未说明是 PySceneDetect、TransNetV2 还是自研模型）[不确定]。切分与筛选耦合在同一步：切出的片段必须同时满足分辨率、运动、美学、人脸构成等条件才被保留，因此不是「先切后筛」的两阶段，而是切分即过滤。切分粒度决定了训练样本全部为单镜头，无镜头转场样本。

### 质量过滤（美学评分、清晰度、OCR文字过滤、黑边/水印/logo检测） ⚠️

视觉质量过滤由三个显式模块构成，均在 pipeline 第 1 步：
(1) 分辨率门槛：片段分辨率必须大于 720×720 像素，硬性剔除低清素材。Ovi 1.1 提升到原生 960×960 数据。
(2) 美学评分：使用 LAION 的 aesthetic predictor（Schuhmann, 2022，即 LAION-Aesthetics 美学打分模型）剔除低质量数据（remove low-quality data）。具体阈值未公开[不确定]。
(3) 运动质量：RAFT 光流剔除静态片段（详见 motion_filtering）。
(4) 结构性清理（在 packing 阶段而非过滤阶段）：显式去除视频已有的 margins（黑边/信箱边框），再等面积缩放。这是把「黑边」当作可修复瑕疵而非丢弃理由的处理方式。
(5) 内容构成控制：内部人脸检测模型保证单人/多人/无人三类的合理配比（属于分布控制，非质量过滤，但在同一步执行）。
【未提及的常见手段】OCR/字幕文字过滤、水印检测、logo 检测、压缩伪影/模糊检测、亮度对比度过滤、JPEG 块效应检测等，论文均无描述[不确定]。这可能是模型在带字幕素材上的潜在风险点。
【音频侧质量过滤】见 audio_quality_filtering（平均音量 ≥ −60 dB）。

### 运动过滤（光流/运动分数阈值、静态与抖动剔除） ⚠️

使用光流模型 RAFT（Teed & Deng, 2020）承担两个作用：(1) 过滤掉静态视频（filter out static videos），即缺乏有效运动的片段直接剔除；(2) 为保留下来的片段计算并保存 motion score（运动分数）作为标注/筛选信号。
【阈值】静态判定的光流幅值阈值、motion score 的取值范围与高低两端截断策略均未公开[不确定]。
【抖动剔除】论文只提到剔除「静态」一端，未提及是否剔除相机剧烈抖动/快速摇镜等运动过强的一端[不确定]。
【motion score 的下游用途】论文未说明是否作为训练条件、采样权重或课程调度依据[不确定]。

### 去重方法（精确去重与基于embedding的语义去重分别记录） ⚠️

[不确定]。论文的数据 pipeline 四步中完全没有去重环节，既未描述精确去重（哈希/pHash/帧指纹），也未描述基于 embedding 的语义去重（CLIP/视频 embedding 聚类去冗余）。纯音频侧同样无去重描述。由于数据来自内部语料而非大规模网络爬取，重复度可能天然较低，但这只是推断，无任何文本依据。

### VLM/LLM作为质检员（多模态大模型质量打分与错配剔除，2026年从浅层打分器转向大模型语义判断的趋势） ⚠️

Ovi 处于「MLLM 用于打标而非用于质检打分」的阶段，这与 2026 年主流的「大模型语义判官」路线只有部分重合。
【已使用的模型化判断】
(1) MLLM 作为标注器：输入 7 帧均匀采样图像 + 完整音轨，产出交织了视觉事件与台词的长 caption 及结构化音频描述。论文强调「conducted extensive experiments to ensure the captioning included all relevant visual and audio events while respecting chronology」——即通过大量 prompt 实验来保证事件完整性与时间顺序正确性，这是对 MLLM 输出质量的间接管控，但未描述用第二个模型去校验 caption 与内容是否错配。
(2) 判别式模型作为过滤器：SyncNet（同步性判官）、RAFT（运动判官）、LAION aesthetic predictor（美学判官）、内部人脸检测模型（构成判官）——全部是浅层/专用打分器，符合 2025 年中期的典型做法。
【未使用的手段】未使用 VLM/LLM 对片段做整体质量打分、未用大模型做「caption 与视频语义错配」的二次剔除、未用大模型做内容合规判断、未有 model-as-judge 的评分阈值。这一维度整体上是 Ovi 相对同期强 pipeline（如 HunyuanVideo 用 MLLM 替代 T5 并配 robust data filter）的薄弱环节[不确定：是否存在未公开的内部 MLLM 质检环节]。
【MLLM 身份】论文全程只写「an MLLM」，未指明是 GPT-4o/Gemini/自研模型或其规模[不确定]。

### 安全与合规过滤（NSFW、版权、人脸/隐私） ⚠️

[不确定]。论文与仓库均未描述任何 NSFW 过滤、暴力/血腥内容过滤、版权内容识别、人脸隐私保护、名人肖像剔除、儿童内容处理等安全合规环节，也没有 Model Card 层面的 safety 章节或使用限制说明（仅有 Apache 2.0 许可）。考虑到 Character AI 作为面向消费者的对话产品公司通常有内部内容审核体系，其内部音视频语料可能已经过产品侧审核，但论文无任何相关表述。这是该工作在数据披露上的明显空白项。

## 打标方式

### 使用的caption模型（自研VLM/开源模型，模型规模） ⚠️

使用「一个 MLLM（多模态大语言模型）」统一承担音视频语料与纯音频语料的打标工作，音视频侧与纯音频侧明确使用同一个模型（「the same MLLM as used in our audio-video data」）。
【模型身份与规模】论文未点名，未说明是商用 API（如 GPT-4o、Gemini）还是内部自研模型，也未给参数规模[不确定]。
【输入形式（关键设计）】不是纯视觉输入，而是「7 帧均匀采样的关键帧 + 完整音轨」同时喂入——即 caption 模型本身必须具备音频理解能力（audio-capable MLLM），这是 AV 联合生成模型打标区别于纯视频模型的核心要求。7 帧的稀疏采样意味着视觉侧描述偏事件级/语义级，而非逐帧密集。
【prompt 工程投入】论文明确「conducted extensive experiments」以确保 caption 覆盖所有相关视觉与听觉事件并遵守时间顺序（respecting chronology），说明 caption 模板经过多轮迭代。
【无独立 ASR】台词转写直接由该 MLLM 从音轨产出，未提及使用 Whisper 等独立 ASR 模型[不确定]。
【推理侧对偶】用户 prompt 需遵循与训练 caption 相同的格式，仓库提供 GPT 生成的示例 prompt CSV（gpt_examples_t2v.csv / gpt_examples_i2v.csv），即推荐用 LLM 按模板扩写 prompt。

### caption密度与结构化程度（短/长/密集描述、结构化字段如镜头运动、风格标签） ⚠️

属于「单条长 caption、内嵌结构化标签」的设计，而非多字段 JSON。
【总体形态】一条 verbose（冗长/密集）的自然语言 caption，视觉事件按时间顺序叙述，台词以内联标签嵌入其中，音频描述统一放在末尾。
【标签体系】
- 语音：<S> 台词内容 <E>（start-of-speech / end-of-speech 标签），可在 caption 中多次出现，与视觉事件按时序交织（interleaved），从而隐式编码「谁在何时说什么」的时间信息。
- 音频描述：<AUDCAP> 音频描述 <ENDAUDCAP>，置于 caption 末尾。Ovi 1.1 起该格式简化为纯文本前缀「Audio: ...」，放在 prompt 结尾。
【密度】视觉侧为 verbose 长描述；因输入仅 7 帧，密度偏事件级而非逐帧密集描述。
【结构化字段】未见镜头运动（camera movement）、景别、风格标签、光照等显式结构化字段[不确定]。已知的结构化维度集中在音频侧（说话人属性）与隐式的时序顺序上。
【单一 prompt 条件化】caption 不拆成视频 prompt 与音频 prompt 两路，而是合并成一条送入单个冻结 T5 编码器，同一嵌入分别与视频塔、音频塔做 cross-attention。论文的直觉解释是：视觉上下文细节提升音频的具体性与多样性，声学上下文细节则引导面部动作与肢体动作。消融实验（5.5 节）证实了这一合并设计优于分离编码方案。

### 音视频联合caption结构（是否同时覆盖视觉+听觉轨道、是否分流为独立字段，如LTX-2全音景描述、Script-a-Video factorized streams、Foley-Omni三字段）

Ovi 的联合 caption schema 是「交织式内联 + 末尾音景块」的混合结构，同时覆盖视觉、语音、非语音听觉三条轨道：
(1) 视觉轨道：自然语言叙述视觉事件，按时间顺序展开，构成 caption 主干。
(2) 语音轨道：以 <S>...<E> 标签内联嵌入 caption 主干的对应时间位置，实现「视觉事件与台词的时序交织」，是其对齐语义与时序的关键设计（相当于把 script 与 shot description 编织在一起）。
(3) 非语音听觉轨道：以 <AUDCAP>...<ENDAUDCAP>（1.1 版为「Audio: ...」）封装在末尾，是一个完整的音景（soundscape）描述块。其内容按片段是否含语音自适应切换：
   - 含语音片段：强调说话人声学属性——年龄（age）、性别（gender）、口音（accent）、音高（pitch）、韵律（prosody）、情绪（emotion）、语速（speaking rate）；
   - 无语音片段：描述音效（sound effects）、背景音（background audio）、音乐元素（musical elements）。
【与其他方案的对比定位】不同于 LTX-2 的全音景统一描述、Script-a-Video 的完全 factorized 独立字段流、Foley-Omni 的三字段拆分，Ovi 选择「不分流为独立条件」——三条轨道全部塞进同一条文本、同一个 T5 嵌入，靠标签而非字段做区分。
【消融验证】5.5 节的唯一消融就是针对这一点：初版设计用 CLAP 编码器编码非语音音频描述、T5 编码语音转写，试图解耦 T2A 与 TTS 任务；结果发现该分离设置限制了模型生成连贯输出的能力——能单独做音效或单独做语音，但难以把二者融合成统一连贯的音轨。改为合并单一 T5 嵌入后，WER 基本持平（0.033→0.035）而 FD_PANNs 从 20.78 降到 18.03、FD_VGG 从 7.13 降到 5.02、IS 从 8.34 升到 11.20、CLAP 从 0.190 升到 0.224。这为「联合 AV caption 应统一编码而非分流」提供了直接的量化证据。
【纯音频数据的对齐】纯音频语料同样用同一 MLLM 产出「转写 + 音频描述」两部分（无语音则转写留空），schema 与音视频侧保持一致，保证两阶段训练的条件分布不漂移。

### 对白转写与说话人属性标注（ASR转写+说话人身份/语言/口音/情绪） ⚠️

【转写】由音频理解型 MLLM 直接从完整音轨产出台词文本，写入 <S>...<E> 标签，并按时序穿插在视觉描述中；纯音频数据同样产出转写，无语音时该字段留空。未提及使用独立 ASR（Whisper 等）或做转写质量校验/WER 过滤[不确定]。
【说话人属性】对含语音片段，音频描述被明确要求强调说话人相关的声学属性：年龄、性别、口音、音高、韵律、情绪、语速（age, gender, accent, pitch, prosody, emotion, speaking rate）。这套属性集使模型在推理时可通过 prompt 直接调控音色与情绪，Ovi 1.1 进一步强化为「基于 prompt 的情绪指令标签」。
【说话人身份/分离】未提及说话人日志（diarization）、说话人 ID 标注、多说话人分离标注，也未提及说话人 embedding 或参考音色条件（README Todo 中「Reference voice condition」仍未实现），因此多人对话场景中「哪句话属于哪个人」缺乏显式监督[不确定]。
【语言标注】口音被标注，但语种是否单独标注未说明[不确定]。

### 几何与结构化标注（相机参数、深度、3D point tracks、动作标注、显式状态） ⚠️

[不确定]。Ovi 没有任何几何/结构化标注环节：无相机内外参、无相机运动标签、无深度图、无 3D point tracks、无姿态/关键点、无动作分类标签、无分割 mask、无显式状态标注。
唯一接近的两项是：(1) RAFT 产出的 motion score（标量运动强度，非结构化几何信息）；(2) 内部人脸检测模型输出的人脸存在性/人数（仅用于数据构成配比，未作为条件注入模型）。
论文特别强调其方法「不需要人脸 bounding box、不需要 face mask」等启发式几何先验（对比 HunyuanVideo-Avatar 用 mask 限制音频特征作用于面部区域），把不依赖几何标注作为一项设计优势——唇同步完全由数据与跨模态注意力自发学得，5.1 节的注意力可视化显示语音 token 自动聚焦于嘴部、鼓声聚焦于鼓、动物叫声聚焦于对应身体部位。

### 合成数据构造（受控扰动/编辑构造训练对，如InstructAV2AV） ⚠️

[不确定]。论文未描述任何合成数据构造环节：无受控扰动构造训练对、无编辑式配对数据（如 InstructAV2AV 类）、无 TTS 合成语音回灌、无音画错配负样本构造、无音轨替换/移位的数据增强。所有音视频训练样本均为真实采集的原生配对数据。
唯一的「人工构造」是数据格式层面的处理：纯音频微调数据 padding 到精确 5.04 秒以匹配 121 帧视频时长；以及推理侧推荐用 GPT 按训练 caption 模板扩写用户 prompt（属于推理期而非训练期的合成）。

### 人工介入程度（人工标注、人工质检、模型初筛+人工复核） ⚠️

[不确定：数据侧几乎无描述]。可确认的人工介入分布在两端：
(1) 数据侧：致谢中提到 Yi Cui、Manav Shah、Diego De La Torre「for their contributions to data preparation」，说明有专人负责数据准备工作，但其职责是人工标注、人工质检还是工程管线搭建未说明。论文正文的数据 pipeline 四步全部由模型/规则自动化完成，未出现人工复核环节，可判断整体为「全自动 pipeline，无逐条人工审核」。caption prompt 的「extensive experiments」是人工在模板设计层面的介入。
(2) 评测侧：人工介入充分且是主要评测手段——组织 50 名参与者做盲测成对偏好研究（blind pairwise preference study），在音频质量、视频质量、音视频同步三个维度对比 Ovi 与 JavisDiT、UniVerse-1，报告 Pairwise Win Rate（PWR）。
未见 RLHF 偏好标注、人工数据清洗抽检率等描述。

## 音视频对齐

### 音视频同步检测方法（口型同步、事件对齐） ⚠️

Ovi 把音视频同步过滤当作整个数据 pipeline 中优先级最高的一环，并且是唯一被作者用「strict」形容的过滤。
【方法】采用经典的 SyncNet（Chung & Zisserman, 2016）——基于 ConvNet 学习声音与嘴部图像之间的联合嵌入，输出两个标量：confidence（置信度，衡量音画相关性强度）与 offset（时间偏移，衡量音轨相对画面的超前/滞后帧数）。作者明确表示对该模型做了工程改造以处理「百万量级（on the scale of millions）」的视频数据，即把 SyncNet 从研究级脚本改造成大规模批处理组件。
【判定逻辑】仅保留同时满足 |offset| ≤ 3（帧）且 confidence > 1.5 的片段，并叠加音频平均音量 ≥ −60 dB 的条件。
【作用范围】主要针对「speech videos」（含语音的片段），用于剔除唇音不同步的数据；非语音片段的事件级音画对齐（如撞击声与撞击画面）未使用专门的检测器，也未使用 AV-align 类事件对齐指标[不确定]。
【设计理由（重要经验）】论文明确写道：「We have experimentally determined that even a small quantity of out-of-sync data can impede lip-sync abilities and chose these strict criteria to minimize the risk of misaligned data.」——即实验证明极少量不同步数据就足以损害模型的唇同步能力，因此宁可牺牲数据量也要用严格阈值把错配风险降到最低。这是全文关于「数据质量优先于数量」最直接的表述。
【无训练期同步损失】数据侧严格过滤之后，训练阶段不再使用任何显式同步损失、辅助同步模块或人脸 mask，同步完全依靠配对采样 + 共享 timestep + 双向跨模态注意力 + scaled RoPE 自发涌现。可以说 Ovi 把同步性的保证几乎完全押在了数据过滤上。

### 同步检测具体指标与阈值（SyncNet/Synchformer/LSE/自研，阈值数值，如MOVA LSE-D≤9.5且LSE-C≥4.5、SkyReels-V4 SyncNet |offset|≤3∧conf>1.5） ⚠️

指标与阈值全部明确公开，是本工作数据侧披露最具体的部分：
- 检测器：SyncNet（Chung & Zisserman, 2016），经内部改造以支持百万级视频规模批处理。
- 指标 1：offset（音视频时间偏移，单位为帧）。阈值：|offset| ≤ 3。按 24fps 计算即容许误差 ≤ 125 毫秒。
- 指标 2：confidence（SyncNet 置信度）。阈值：confidence > 1.5。
- 指标 3（音频侧联合门槛）：mean volume ≥ −60 dB（平均音量不低于 −60 分贝），用于排除近似静音/音轨极弱的片段。
- 组合逻辑：三者为「与」关系，须同时满足才保留。
【横向对照】该 confidence>1.5 门槛属于中等偏严（SyncNet conf 常见取值范围约 0~10+，部分工作如 UniTalking 用 >0.9，talking-head 数据集常用 >3），而 |offset|≤3 帧属于较严格的时间容差。作者自评为 strict criteria。
【未公开】通过率、各阈值的消融曲线、阈值选取的实验依据数值均未给出[不确定]。

### 时序同步vs语义同步的分离处理（时间对齐与内容语义匹配作为两个独立过滤条件） ⚠️

Ovi 在数据过滤层面基本只处理「时序同步」，语义匹配主要交给 caption 与训练机制，两者未被设计成两个并列的独立过滤条件。
【时序同步（有显式过滤）】SyncNet 的 offset 与 confidence 双阈值，专门约束语音与嘴部动作的时间对齐；音频平均音量门槛排除无效音轨。
【语义同步（无显式过滤）】没有使用 CLAP/AV-CLIP 类跨模态语义相似度打分来剔除「音画内容不匹配」的样本（例如画面是海滩、音轨是无关旁白/后期配乐的情形）。语义一致性依赖两条间接路径：(1) caption 阶段 MLLM 同时看画面与听音轨，把双方内容写进同一条描述，从而在文本条件层面强制语义绑定；(2) 训练时双向跨模态注意力让两塔互为语义上下文。
【潜在缺口】画外音（voice-over）、后期配乐、配音译制等「时序可能对齐但语义与画面无因果关系」的样本，仅靠 SyncNet 难以完全剔除（SyncNet 对无嘴部画面的片段本身置信度低，可部分间接过滤），论文未讨论此风险[不确定]。
【架构层面的时序/语义分工】值得注意的是，Ovi 在模型侧对二者做了明确分工：时序由 scaled-RoPE（31/157≈0.197 频率缩放）负责，语义由双向 cross-attention 负责——论文摘要即表述为「blockwise exchange of timing (via scaled-RoPE embeddings) and semantics (through bidirectional cross-attention)」。这种「时序与语义分离处理」的思想体现在架构而非数据过滤上。

### 音频质量过滤（SNR、静音检测与静音占比阈值、无音轨剔除、画外音源剔除、背景音乐分离） ⚠️

音频侧过滤相对简洁，仅一条硬性门槛加若干隐式约束：
(1) 音量门槛：平均音量必须 ≥ −60 dB，与 SyncNet 双阈值同时施加。作用是剔除静音、近似静音、音轨损坏或音量极低的片段——即隐式完成了「无有效音轨剔除」。
(2) 无显式 SNR/噪声过滤：论文未描述信噪比阈值、噪声抑制、混响过滤、削波/失真检测[不确定]。
(3) 无背景音乐分离：未使用音源分离（如 Demucs）把 BGM 与人声拆开——事实上 Ovi 有意保留原生混合音轨，因为其目标就是同时生成语音+音效+BGM 的完整音景。
(4) 无画外音源剔除的显式模块：仅靠 SyncNet 低置信度间接过滤[不确定]。
(5) 静音占比阈值：只有「平均音量」这一全局指标，未见逐段静音占比统计[不确定]。
(6) 采样率与带宽约束（工程层面）：音频统一走 MMAudio 的 16kHz 编码器变体（STFT → mel 频谱 → 1D VAE latent），论文 Limitations 承认这条 16kHz 固定 1D-VAE 路径限制了带宽与空间感，高保真音乐、空间线索与细微音色会被压平，未来可换更高带宽 latent 或做后处理带宽扩展。推理时由 BigVGAN 声码器还原波形。

### 语音/音效/音乐的分类与分别处理策略 ⚠️

Ovi 的核心主张之一是「必须用统一音频模型同时覆盖语音与音效」，而非分而治之，但在数据组织上仍按类型分别处理：
【数据来源分型】语音主要来自内部语料（预训练主体）；音效来自 VGGSound / AudioSet / WavCaps（微调期引入）；音乐未见独立来源，主要以视频原生音轨中的 BGM 形式出现[不确定]。
【训练阶段分型】音频预训练以语音为主（学说话人身份、音高、情绪、韵律）；音频微调阶段引入多样音效，同时掺回内部音视频的音轨以保活 TTS 能力，使 Ovi-Aud 成为可同时做 T2A 与 TTS 的统一音频基座。
【打标分型】按「含语音 / 不含语音」二分：含语音者音频描述聚焦说话人声学属性；不含语音者聚焦 sound effects / background audio / musical elements 三类。纯音频数据中若无语音则转写字段留空。
【条件编码上不分型（关键决策）】早期版本曾用 CLAP 编码非语音描述、T5 编码语音转写以「解耦 T2A 与 TTS」，消融证明该分离方案会导致模型无法把音效与语音融合成连贯音轨，最终改为全部合并进单一 T5 嵌入。
【评测分型】T2A 侧按 MMAudio 协议报 FD_PANNs 18.03 / FD_VGG 5.02 / IS 11.20 / CLAP 0.224；TTS 侧在 Seed-TTS test-en 上报 WER 0.035。二者均接近各自领域专用模型（如 MMAudio-L 的 FD_PANNs 15.04、F5-TTS 的 WER 0.018），论文认为统一模型不必超越专用模型，能同时胜任才是音视频融合的前提。

## 训练配合

### 多阶段训练课程与数据课程调度（阶段划分依据：分辨率/时长/质量分/模态；低清→高清、图像→视频、短→长） ⚠️

三阶段课程，划分依据是「模态 → 时长 → 模态融合」，而非常见的分辨率或质量分：
【阶段 1｜音频预训练（Ovi-Aud pretraining）】从零训练音频塔（架构照搬 Wan2.2 5B），数据为数十万小时、最长 12 秒的变长原始音频，以人类语音为主。目的是打牢说话人身份、音高、情绪、韵律的长程建模。50k 步，batch 2880，lr 1e-4，AdamW(β1=0.9, β2=0.999, ε=1e-8)。关键工程细节：从预训练起就对所有注意力层施加 scaled RoPE，从而避免进入音视频阶段时的重适配、免于维护多套音频 RoPE 尺度。
【阶段 2｜音频微调】改用 padding 到精确 5.04 秒的定长波形（对齐 121 帧 @24fps），混入 VGGSound/AudioSet/WavCaps 音效数据与内部音视频抽取的音轨，使音频分布与下游融合训练一致，同时补足音效能力、保活 TTS 能力。
【阶段 3｜音视频融合微调】拼接已预训练的音频塔与 Wan2.2 5B 视频塔，跨模态注意力层从零初始化，冻结全部 FFN 以省显存，11B 中 5.7B 参数可训（仅训练单模态 self-attention 与两类 cross-attention：text-to-modality、modality-to-modality）。40k 步，batch 768，lr 5e-5，AdamW(β1=0.9, β2=0.95, ε=1e-8)。损失为两模态 flow matching 加权和 λ_v=0.85、λ_a=0.15，共享 timestep、独立噪声。
【后续（Ovi 1.1）】README Todo 中已勾选「Finetune model with higher resolution data, and RL for performance improvement」与「Longer video generation (10s)」，即追加了高分辨率（960×960 原生）微调、RL 优化与长时长扩展阶段，构成事实上的「低清短片段 → 高清长片段」课程；该阶段的数据与训练细节未公开[不确定]。
【未采用】图像→视频的渐进课程、低清→高清的多分辨率课程（初版全程 720×720 等面积定长）、按质量分分层的课程调度均未出现在论文中。

### 数据配比随训练阶段的变化（预训练/退火annealing/SFT高质量子集） ⚠️

配比随阶段的变化是显式且有明确动机的，但缺少数值：
(1) 音频预训练 → 音频微调：从「以人类语音为压倒性主体、最长 12 秒变长」切换为「5.04 秒定长 + 大量音效（VGGSound/AudioSet/WavCaps）+ 内部视频音轨」。两处调整同时发生：时长分布收窄以对齐下游、音频类别从语音单一转向语音+音效混合。掺回内部视频音轨的目的被明确表述为「maintain TTS abilities and better align with the downstream goal」（保持 TTS 能力并更好对齐下游目标）——即典型的防遗忘回放配比。混合比例数值未公开[不确定]。
(2) 音视频融合阶段：数据全部换为经四步 pipeline 清洗的内部音视频配对语料，纯音频数据不再参与[不确定：是否有少量音频数据混合]。
(3) 损失权重层面的「模态配比」：λ_v=0.85 / λ_a=0.15，视频损失权重远高于音频——可视为在训练目标上对视频保真度的倾斜，以缓解融合训练导致的视频质量退化。
(4) Ovi 1.1：数据量翻倍且改用 960×960 原生数据，配比结构变化未公开[不确定]。
(5) 无退火（annealing）阶段、无高质量 SFT 子集的描述[不确定]。

### 后训练数据（SFT精选集规模与筛选标准、偏好对数量与标注方式、reward model训练数据） ⚠️

初版论文中没有传统意义的后训练：无 SFT 精选集、无偏好对（preference pairs）、无 reward model、无 DPO/RLHF 数据描述。论文中所称的「audio post-training」实为音频塔的 5 秒定长微调阶段（见 multi_stage_curriculum 阶段 2），属于分布适配而非偏好优化。
【Ovi 1.1 的 RL】README Todo List 中「Finetune model with higher resolution data, and RL for performance improvement」已勾选完成，表明 1.1 版引入了强化学习优化，但 RL 的算法、奖励信号（是否基于同步性/美学/人类偏好）、偏好数据规模与标注方式全部未公开，也无对应技术报告[不确定]。
【人类偏好数据的唯一出现场景】50 人盲测成对偏好研究仅用于评测（PWR 指标），未被回收用于训练 reward model[不确定]。
【蒸馏】README Todo 中「Distilled model for faster inference」仍未完成；论文 Limitations 提出可用 DMD2 框架做步数蒸馏，属未来工作。

### 数据处理基础设施与吞吐（NeMo Curator/Data-Juicer/自研，GPU加速比，处理规模，成本） ⚠️

披露极少。可确认的工程细节：
(1) SyncNet 的规模化改造：论文明确「We adapt the model to handle video data on the scale of millions」——把研究级 SyncNet 改造为可处理百万量级视频的批处理组件，是数据基础设施上唯一被点名的改造工作。
(2) 数据打包为字节：packing 阶段把视频按 24fps 抽帧后转为 byte array、音频转为 raw wave bytes，以字节形式喂给训练侧，属于面向高吞吐读取的存储格式设计。
(3) 训练侧基础设施：全程 bf16 精度，使用 DeepSpeed（Rasley et al., 2020）做分片式分布式数据并行（sharded DDP）；音频预训练 batch 2880、融合训练 batch 768，可推断使用了较大规模 GPU 集群，但卡数、卡型、训练时长、总算力成本均未公开[不确定]。
(4) 未使用/未提及 NeMo Curator、Data-Juicer 等现成数据处理框架，也未给出 GPU 加速比、处理吞吐（clip/小时）、处理成本等任何定量指标[不确定]。
(5) 推理侧（非数据侧）披露较全：单卡峰值显存约 80GB（FlashAttention-3），标准 32GB、fp8 量化 24GB 可跑，支持多卡序列并行；Todo 中 FSDP 分片推理与序列并行效率优化尚未完成。

## 效果对比

### 数据策略消融的量化影响（区分：过滤严格度ablation / caption密度风格ablation / 数据配比ablation，及对应评测指标） ⚠️

论文的消融实验只有一项，且属于「caption/条件编码结构」类而非「过滤严格度」或「数据配比」类，但它恰好是 AV 联合 caption 设计的关键证据。
【消融 1（唯一正式消融，5.5 节 + 表 3）：音频条件的分离编码 vs 合并编码】
- 变体 A「Ovi with CLAP」：用 CLAP 文本编码器编码非语音音频描述、T5 编码语音转写，试图解耦 T2A 与 TTS 两个任务、避免相互干扰。
- 变体 B「Ovi」（最终方案）：把语音转写与音频描述融合进一条文本，用单一 T5 嵌入。
- 结果：FD_PANNs 20.78 → 18.03（↓2.75）、FD_VGG 7.13 → 5.02（↓2.11）、IS 8.34 → 11.20（↑2.86）、CLAP 0.190 → 0.224（↑0.034）、WER 0.033 → 0.035（基本持平，语言正确性未受损）。
- 结论：分离编码虽能各自处理音效或语音，却无法把二者整合成统一连贯的音轨；合并编码在保持语言正确性的同时显著提升音频保真度与文本-音频对齐，并让音视频两塔可共用同一 T5 表示，简化跨模态建模、增强多模态连贯性。
【消融 2（架构侧，图 2 定性）：scaled RoPE】不缩放时跨模态 RoPE 亲和矩阵对角线错位、阻碍同步；按 31/157≈0.197 缩放后对角线锐利对齐。仅有定性可视化，无定量指标。
【过滤严格度的「实验性结论」（无表格）】论文声称「experimentally determined that even a small quantity of out-of-sync data can impede lip-sync abilities」，即做过不同步数据比例的实验并据此选定严格阈值，但未给出实验设置与量化结果[不确定]。这是全文最值得追问却未披露的数据消融。
【视频质量的代价（非正式消融但为重要观测）】与基座 Wan2.2 相比，Ovi 的视频质量有轻微退化，论文归因于「联合训练所用的音视频数据集比 Wan2.2 预训练语料窄得多（a narrower audio-video dataset）」——这是数据覆盖面收窄带来的量化可感代价，也侧面印证严格同步过滤对数据多样性的削减。
【缺失】caption 密度/风格的消融、数据配比的消融、过滤阈值扫描（SyncNet conf 或 offset 取不同值的效果曲线）均未提供[不确定]。

### 质量vs数量的证据（小而精数据超越大而杂的案例） ⚠️

有明确的定性主张，但缺少「小而精数据集超越大而杂数据集」的对照实验数字。
【正向证据（作者自述）】唯一直接表述在同步过滤处：「即使少量不同步数据也会阻碍唇同步能力（even a small quantity of out-of-sync data can impede lip-sync abilities），因此选择严格标准以最小化错配风险」。这句话表达的是典型的质量优先原则——宁可大幅缩减可用数据量，也不容忍少量脏数据污染同步能力。整套 pipeline 的设计取向（>720×720 硬门槛、RAFT 静态剔除、美学剔除、SyncNet 严格双阈值）与之一致。
【反向证据（代价）】质量优先并非无代价：论文承认联合训练后视频质量相对 Wan2.2 基座有轻微退化，原因正是「音视频配对数据集比 Wan2.2 的大规模预训练语料窄」——即严格筛选换来的高纯度数据在覆盖广度上不足。作者认为这一权衡是边际的、可接受的。
【规模仍被追求的证据】Ovi 1.1 的主要改进之一就是「Dataset includes 100% more videos for greater diversity」（数据量翻倍以获得更大多样性），说明团队在保持质量门槛的前提下仍以扩量为主要提升手段——质量是门槛、数量是杠杆。
【无量化对照】未提供「严格过滤子集 vs 宽松大集」的并列训练结果[不确定]。

### 训练数据domain分布与评测基准类目体系的对齐关系（如VABench七大类） ⚠️

对齐关系较弱，训练数据 domain 分布与评测类目体系之间没有刻意设计的映射。
【所用评测】(1) 联合 AV 生成：在 Verse-Bench（由 UniVerse-1 的作者 Wang et al., 2025 提出，非 Ovi 自建）上做 50 人盲测成对偏好研究，评测维度仅三项——音频质量、视频质量、音视频同步，Ovi 对 JavisDiT 与 UniVerse-1 在三个维度上均以显著优势胜出。这是一个「质量维度」而非「内容类目」的评测体系，与训练数据的 domain 分布无类目级对应关系。(2) 音频塔：T2A 沿用 MMAudio 的评测协议（FD_PANNs / FD_VGG / IS / CLAP），其数据分布与训练期引入的 VGGSound/AudioSet 音效类目体系天然同源，可视为音频侧唯一存在的类目对齐。(3) TTS：Seed-TTS test-en 报 WER，仅覆盖英语。(4) 视频侧：与基座 Wan2.2 做相对比较以确认未因联合训练而退化。
【无 VABench 式类目对齐】论文未构建或对齐任何七大类/多类目的评测分类体系，也未按内容类别分组报告指标；训练侧唯一的显式配比控制（单人/多人/无人）在评测中并无对应的分组结果[不确定]。
【缺口】未报告 VBench、VideoScore 等标准视频基准分数，未报告 AV-Align、SyncNet conf 等客观同步指标（同步性仅靠人评），使其数据策略与评测之间缺少可追溯的量化闭环。

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
