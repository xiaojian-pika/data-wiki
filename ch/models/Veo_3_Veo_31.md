# Veo 3 / Veo 3.1（含 Veo 3.1 Lite）

> 主题: 视频生成模型（含音视频同时生成）的数据处理：数据清洗流程、数据分布、打标方式

[← 返回首页](../index.md)

**目录**: [基本信息](#基本信息) · [数据规模与分布](#数据规模与分布) · [清洗流程](#清洗流程) · [打标方式](#打标方式) · [音视频对齐](#音视频对齐) · [训练配合](#训练配合) · [效果对比](#效果对比)

## 基本信息

### 名称

Veo 3 / Veo 3.1（含 Veo 3.1 Lite）

### 发布机构/公司

Google DeepMind（Google）

### 发布时间（技术报告/论文/开源时间）

Veo 3 于 2025 年 5 月 20 日在 Google I/O 发布，官方 Model Card 首次发布于 2025-05-23，并于 2026-01-13 更新；配套的《Veo: a text-to-video generation system》技术报告同期发布。Veo 3.1 于 2025 年 10 月中旬发布（Flow / Gemini API / Vertex AI 上线）。Veo 3.1 Lite 的 Model Card 发布于 2026-04-08。官方声明「本 Model Card 覆盖 Veo 3 及其后续版本」，即 Veo 3.1 系列沿用 Veo 3 的数据与训练披露。

### 类型（模型/数据集/工具链/评测基准）

模型（闭源商业视频+音频联合生成基础模型，通过 Gemini App、Flow、Google Vids、Google AI Studio、Gemini API、Vertex AI 提供服务）

### 开源程度（权重/代码/数据/pipeline各自是否开源）

完全闭源。权重不开源、代码不开源、训练数据不公开、数据处理 pipeline 不公开、数据配比与规模不披露。仅公开一份约 7 页的技术报告（Veo-3-Tech-Report.pdf）和一份约 6 页的 Model Card，其中与数据相关的正文不足 200 词，绝大部分篇幅是责任与安全评测。仅通过付费 API / 产品形态开放推理调用（Veo 3、Veo 3 Fast、Veo 3.1、Veo 3.1 Fast、Veo 3.1 Lite 等变体）。

### 是否支持音视频同时生成，以及实现方式（原生联合/级联/MoE融合）

支持，且为原生联合生成（native joint generation），不是级联 pipeline。官方技术报告原文：「Veo 3 uses latent diffusion, in which the diffusion process is applied jointly to the temporal audio latents, and the spatio-temporal video latents.」即视频与音频分别由各自的 autoencoder 编码为压缩潜表示（视频为时空潜变量 spatio-temporal latents，音频为时序潜变量 temporal latents），随后一个基于 Transformer 的去噪网络在同一个扩散过程中对两类 latent 联合去噪，每一步 denoising 同时处理音视频 token，从而在生成阶段天然保证同步。生成内容涵盖对白（带唇形同步）、音效/foley、环境音与背景音乐。未采用 MoE 融合或先视频后配音的两阶段级联方案。

### 调研信息来源列表（论文/技术报告/官方文档/新闻的URL，标注每条来源的性质：官方一手/同团队旁证/第三方报道）

- 官方一手: Veo 3 Tech Report (PDF, Model & Data / Mitigations章节) https://storage.googleapis.com/deepmind-media/veo/Veo-3-Tech-Report.pdf
- 官方一手: Veo 3 Model Card (PDF, Training Dataset/Processing/Evaluation) https://storage.googleapis.com/deepmind-media/Model-Cards/Veo-3-Model-Card.pdf
- 官方一手: Veo 3.1 Lite Model Card (确认3.1系列沿用Veo 3数据披露) https://deepmind.google/models/model-cards/veo-3-1-lite/
- 第三方报道: CNBC - Google使用YouTube视频子集训练Gemini与Veo 3 https://www.cnbc.com/2025/06/19/google-youtube-ai-training-veo-3.html
- 官方一手: Video models are zero-shot learners and reasoners (arXiv:2509.20328, 反推无显式几何标注) https://arxiv.org/abs/2509.20328
- 官方一手: Veo产品页 (输出规格与MovieGenBench结果) https://deepmind.google/models/veo/

## 数据规模与分布

### 训练数据量级（视频条数/小时数/token数，预训练与SFT分开） ⚠️

[不确定] 官方从未披露任何训练数据量级——无视频条数、无小时数、无 token 数，也未区分预训练与 SFT 规模。技术报告仅表述为「We train on a large dataset comprising images, videos, and associated annotations」。第三方博客流传的「数十亿音视频对」「数百万小时配对音视频」等说法均无官方出处，不可采信。可作弱旁证的是：训练使用 Google TPU Pod 集群、JAX 与 ML Pathways，暗示规模处于同期最大级别之一。

### 数据来源构成（自有/公开数据集/网络爬取/授权采购/合成数据） ⚠️

官方仅说明训练数据由「音频、视频、图像」三类构成（Model Card: 「Veo 3 was trained on audio, video, and image data」），未给出各来源占比。可确认的来源线索：(1) YouTube 视频——2025 年 6 月 Reuters/CNBC 报道并经 YouTube 证实，Google 使用 YouTube 语料库的一个子集（非全量）训练 Gemini 与 Veo 3，法律依据是 YouTube 服务条款中「全球、非独占、免版税」许可，多数创作者对此不知情；(2) 合成数据——官方确认生成合成 caption（synthetic captions）以提升概念多样性，但未说明是否使用合成视频；(3) 授权/采购与公开数据集的具体构成与比例[不确定]。整体来源构成比例[不确定]。

### 数据合规与溯源（授权数据占比、rights-cleared数据集、C2PA等） ⚠️

官方可确认的合规措施：训练视频按「various compliance and safety metrics」过滤；caption 侧过滤掉不安全描述与个人可识别信息（PII）；训练数据被分析以识别潜在有害数据并审查人群表征的公平性；输出端统一嵌入 SynthID 隐形水印，并配合生产环境过滤降低信息完整性风险。Google 生态在 2025-2026 年同时对 Imagen / Veo / Lyria 输出采用 SynthID，并逐步对齐 C2PA Content Credentials 元数据标准（Veo 3 输出是否默认携带 C2PA 清单需以产品线为准）[不确定]。未披露的关键项：授权数据占比、rights-cleared 数据集清单、版权方分成或退出（opt-out）机制。YouTube 数据使用引发创作者知情权与 IP 争议，属公开争议点。

### 片段时长分布与切分策略 ⚠️

[不确定] 训练侧片段时长分布与切分策略完全未披露。仅可从产品侧反推：Veo 3 单次生成固定 8 秒；Veo 3.1 支持 4 / 6 / 8 秒基础片段，并通过 Extend 功能级联延长（Flow 中可延长至约 60 秒乃至 140 秒以上）。基础生成单元固定在 4-8 秒量级，间接暗示训练数据主体为经镜头切分后的秒级短片段而非长视频。

### 分辨率/宽高比分布与分桶策略 ⚠️

[不确定] 训练数据的分辨率/宽高比分布与分桶（bucketing）策略未披露。产品侧输出规格：Veo 3 支持 720p / 1080p，宽高比 16:9 与 9:16；Veo 3.1 进一步支持至 4K，同样覆盖 16:9 横屏与 9:16 竖屏，帧率 24fps。同时支持横竖两种比例说明训练数据包含多宽高比样本并很可能采用了分桶或多分辨率混合训练，但具体比例与实现方式无官方依据。

### 类别/domain分布与配比策略（人物、动作、场景、风格等比例控制与概念均衡） ⚠️

[不确定] 类别/domain 配比策略未披露。可作反推的官方线索：(1) Model Card 明确说明「生成合成 caption 以提升训练数据中与视频关联的概念的多样性与丰富度」（synthetic captions to improve the variety and diversity of concepts），这是一种以 caption 侧改写实现概念均衡的策略，而非直接的样本重采样；(2) 技术报告在「Dangerous Capabilities」一节指出 Veo 3「有生成电影化素材的偏好，频繁出现镜头切换与戏剧化机位」（bias for generating cinematic footage, with frequent camera cuts and dramatic camera angles），并因此难以生成低制作水准的写实胁迫类视频——这强烈暗示训练数据显著偏向电影/专业影视与高制作水准素材，UGC/监控/手机随手拍等低制作水准 domain 占比偏低；(3) 公平性评测显示模型在未指定人种时明显偏向浅肤色（skews towards lighter skin tones），并存在特定词汇与特定人群的语义偏差（semantic bias），反映训练数据的人物表征分布本身不均衡；(4) 官方承认在文字渲染上仍然较弱，暗示含文字/OCR 场景的数据覆盖或标注不足。以上均为从公开报告反推，非官方配比说明。

### 音频类别分布与配比（语音/音效foley/音乐/环境音/静音的比例及控制策略）——AV模型独有维度 ⚠️

[不确定] 音频类别配比与控制策略完全未披露，这是 Veo 3 数据披露中最大的空白之一。可反推的线索：(1) 模型能力覆盖对白（dialogue）、音效/foley（sound effects）、环境音（ambient noise）与背景音乐（music）四大类，说明训练配对音视频数据在这四类上均有相当覆盖；(2) 官方强调 Veo 3 解决了此前视频模型的「无声电影」问题，说明训练时必须显式排除无音轨或音轨与画面无关的样本，但排除标准与静音样本占比未披露；(3) 技术报告在 deepfake 评估中提到 Veo 3 生成的深度伪造「在语音方面尤其难以控制」（much less controllable - particularly with respect to speech），暗示语音数据在说话人身份/音色维度上未做细粒度标注或条件化；(4) Model Card 未提及任何音源分离（如 BGM 与人声分离）、SNR 阈值或静音占比阈值。语音/音效/音乐/环境音的具体配比数值无任何公开依据。

### 叙事结构分布（单镜头vs多镜头、平均clip时长、镜头数分布、是否含原生音轨） ⚠️

[不确定] 训练数据中单镜头 vs 多镜头的比例、平均 clip 时长、镜头数分布均未披露。反推线索：技术报告明确记载模型存在「频繁镜头切换与戏剧化机位」的电影化偏好，说明训练数据中包含大量多镜头（multi-shot）剪辑成片，且未被完全切分为单镜头片段，否则模型不会在 8 秒内自发产生剪辑点——这是 Veo 3 数据构成的一个重要间接证据。另一确定项：训练数据必然以「带原生音轨」为主，因为音视频 latent 需成对进入联合扩散过程。

### 语言/口音分布（多语种唇同步能力的数据基础） ⚠️

[不确定] 语言与口音分布完全未披露。产品侧表现为可生成多语种对白并自动匹配口音与唇形（英语、西班牙语、普通话等），但官方文档未列出支持语种清单、未给出各语种数据占比、未说明口音标注体系。第三方评测普遍反映非英语对白的唇同步与发音质量明显弱于英语，间接说明训练数据以英语为主导。官方亦承认模型「不提供口音、音色等细粒度声音控制」，说明说话人属性未作为条件维度进入训练数据 schema。

## 清洗流程

### 清洗漏斗整体结构（几级过滤、各级顺序） ⚠️

官方披露的清洗漏斗极其简略，仅可还原为四个可确认环节（顺序为推测）：(1) 多粒度 caption 标注——用多个 Gemini 模型对音频与视频片段生成不同详细程度的文本描述；(2) caption 侧过滤——剔除不安全 caption 与含个人可识别信息（PII）的描述；(3) 视频侧过滤——按合规性（compliance）、安全性（safety）与质量（quality）三类指标过滤训练视频，并按风险领域对预训练数据做安全过滤；(4) 语义去重——在所有数据源之间做跨源语义去重，同时移除重复及概念高度相似的视频。此外还有一项非过滤性的数据分析环节：对训练数据做有害内容分析与人群表征公平性审查。[不确定] 各级过滤的具体层级数、执行顺序、判定模型与阈值均未公开。

### 漏斗定量保留率（每级过滤的输入/输出量与最终保留率，如Apollo 27%） ⚠️

[不确定] 完全未披露。官方未给出任何一级过滤的输入/输出数据量、保留率或最终留存比例，无法与 Apollo 27% 之类的公开量化漏斗做对比。这是 Veo 3 数据披露中最彻底的空白。

### 镜头切分方法（PySceneDetect/自研模型/shot-aware splitting） ⚠️

[不确定] 镜头切分方法未披露，未提及 PySceneDetect 或任何自研 shot-aware splitting 方案。反向证据是模型在 8 秒生成中会自发产生镜头切换，说明训练样本并非严格切分至单镜头粒度，或至少保留了相当比例的多镜头样本。

### 质量过滤（美学评分、清晰度、OCR文字过滤、黑边/水印/logo检测） ⚠️

官方唯一表述为「Training videos were also filtered for various compliance and safety metrics and for quality」，即确认存在质量过滤这一环节。[不确定] 具体手段完全未披露：是否使用美学评分模型（aesthetic scorer）、清晰度/模糊度指标、OCR 文字覆盖率过滤、黑边检测、水印/logo 检测、压缩伪影检测等均无说明，也无任何阈值数值。可作弱反推：技术报告承认 Veo 3「在生成文字上仍然较差」，若数据端做过激进的 OCR 文字过滤，会进一步削弱文字渲染能力，二者在方向上是一致的但不能作为证据。

### 运动过滤（光流/运动分数阈值、静态与抖动剔除） ⚠️

[不确定] 完全未提及运动过滤。官方未披露光流（optical flow）计算、运动分数阈值、静态镜头剔除或抖动/手持晃动剔除策略。仅可从「模型对真实世界物理有较好模拟」「运动表示精度高」等能力描述侧面推测存在某种运动质量筛选，但无直接依据。

### 去重方法（精确去重与基于embedding的语义去重分别记录） ⚠️

官方明确披露且相对具体的少数几项之一。Model Card：「All training data was deduplicated semantically across various sources.」技术报告：「All data is deduplicated semantically across sources to minimize the risk of outputs overfitting particular elements of training data.」Mitigations 一节另有：「removing duplicated and/or conceptually similar videos」。可确认为：跨数据源的语义级去重（semantic dedup，实现上应基于 embedding 相似度），且不止剔除完全重复项，还剔除「概念上高度相似」的视频。去重的动机被明确表述为降低输出对特定训练样本的过拟合/记忆化风险（即缓解逐字复现与版权风险）。[不确定] 未披露所用 embedding 模型、相似度阈值、聚类方式，也未区分精确去重（如感知哈希）与语义去重的各自贡献量。

### VLM/LLM作为质检员（多模态大模型质量打分与错配剔除，2026年从浅层打分器转向大模型语义判断的趋势） ⚠️

Veo 3 是「大模型深度介入数据流程」这一趋势的典型代表，但介入形态以「标注」为主而非「打分」。可确认项：(1) 使用多个 Gemini 模型（multiple Gemini models）为音视频数据生成不同粒度的 caption，这是 VLM 作为数据生产者的直接证据；(2) 使用多模态分类器（multimodal classifier）检测内容策略违规，官方特别强调多模态的必要性——「caption 与 video 单独看均无害，组合后可能产生有害结果」（举例：文本 prompt「一头猪的图像」与某一人群视频配对会构成有害表征），这本质上是一个跨模态语义错配/有害组合的判别器，思路上与「VLM 作为数据质检员」高度一致，但官方将其定位在开发期安全监测（development evaluations），而非明确的训练数据清洗环节。[不确定] 是否存在用 Gemini/VLM 对视频做美学或语义质量打分、是否用大模型剔除 caption-video 语义错配样本、打分模型规模与阈值，均未披露。

### 安全与合规过滤（NSFW、版权、人脸/隐私） ⚠️

官方披露相对充分的一项。预训练阶段（pre-training mitigations）措施包括：按风险领域（risk areas）对预训练数据做安全过滤；对 caption 过滤不安全内容与个人可识别信息（PII）；训练视频按合规与安全指标过滤；对训练数据做有害内容分析与人群表征的公平性审查；移除重复及概念相似视频。风险领域覆盖儿童性虐待与剥削材料（CSAM）、仇恨言论、骚扰、错误信息、深度伪造、色情内容、暴力血腥等。后训练阶段（post-training mitigations）包括 SynthID 水印与生产环境输出过滤。安全策略与 Google 跨产品生成式 AI 框架及 Gemini / Imagen 3 技术报告一致，发布前由 Google DeepMind 责任与安全委员会（RSC）审批。[不确定] 未披露具体分类器、判定阈值与被过滤数据比例；人脸/隐私仅覆盖到 caption 侧 PII，视频侧人脸隐私处理策略未说明。

## 打标方式

### 使用的caption模型（自研VLM/开源模型，模型规模） ⚠️

使用「多个 Gemini 模型」（multiple Gemini models）对音频与视频片段进行文本 caption 标注，这是官方在技术报告与 Model Card 中重复强调的核心数据处理手段。采用多个模型而非单一模型，对应「不同详细程度」的多粒度标注需求（可推测为大模型产出长密集描述、小模型产出短描述以控制成本）。[不确定] 未披露具体 Gemini 版本（1.5 / 2.0 / 2.5 系列）、参数规模、是否为针对视频标注微调的内部专用变体、标注吞吐与成本。

### caption密度与结构化程度（短/长/密集描述、结构化字段如镜头运动、风格标签） ⚠️

可确认两点：(1) 多粒度——「annotated with text captions at different levels of detail」，即同一条数据配有从短到长/从粗到细的多个层级 caption，这是提升 prompt 长度鲁棒性（短 prompt 与长 prompt 均可用）的标准做法；(2) 合成 caption 用于概念扩展——「We generate synthetic captions to improve the variety and diversity of concepts associated with videos in the training data」，即不依赖原始 alt-text/标题，而由 Gemini 重写生成，从而覆盖摄影手法、动作、风格、场景语境等原始元数据不包含的概念维度。这与模型能精确响应「风格、机位、运镜及其组合」的能力描述相互印证。[不确定] 未披露是否存在结构化字段 schema（如显式的 camera motion / shot size / lighting / style 标签位）、caption 平均长度、每条数据的 caption 数量、多粒度 caption 在训练中的采样比例。

### 音视频联合caption结构（是否同时覆盖视觉+听觉轨道、是否分流为独立字段，如LTX-2全音景描述、Script-a-Video factorized streams、Foley-Omni三字段） ⚠️

[不确定] 官方表述为「Audio and video clips were annotated with text captions at different levels of detail」，即音频与视频均被文本化标注，说明 caption 覆盖听觉与视觉两条轨道；模型能按 prompt 中的对白引号、音效描述、环境音描述分别响应，也说明训练 caption 中这三类听觉信息是被显式描述的。但官方从未说明其组织形式：是像 LTX-2 那样融合为单条「全音景」描述，还是像 Script-a-Video 那样分流为 factorized streams，抑或像 Foley-Omni 那样拆为独立三字段；音视频 caption 是否共享同一条文本、是否分别注入不同 cross-attention 分支，均无公开信息。

### 对白转写与说话人属性标注（ASR转写+说话人身份/语言/口音/情绪） ⚠️

[不确定] 未披露任何 ASR 转写流程或说话人属性标注体系。反推线索：模型可按 prompt 中引号内文本精确生成对应对白并保持唇同步，说明训练数据必然包含逐字对白转写（很可能由 Gemini 的音频理解能力或 Google 内部 ASR 完成），且转写与视频时间轴对齐。但官方在评估 deepfake 风险时明确指出模型「在语音方面可控性差」、不提供口音与音色的细粒度控制，强烈暗示说话人身份、音色、口音、情绪等属性未被系统性标注为独立条件字段。

### 几何与结构化标注（相机参数、深度、3D point tracks、动作标注、显式状态） ⚠️

[不确定] 未披露任何几何或结构化标注。未提及相机参数（内外参/位姿）、深度图、3D point tracks、光流、动作标签或显式物理状态标注。可作反推的两点：(1) Veo 3.1 提供了明确的「camera controls（精确取景与运镜控制）」产品能力，暗示训练数据中存在某种形式的运镜标注（最可能仍是通过 caption 中的自然语言运镜描述实现，而非显式相机参数）；(2) DeepMind 自家论文《Video models are zero-shot learners and reasoners》(arXiv:2509.20328) 用 62 项视觉任务测试 Veo 3，发现其零样本具备分割、边缘检测、关键点定位、超分、去模糊、去噪等能力，并提出「chain-of-frames」概念——作者强调这些任务模型均未被显式训练，反而说明这些几何/结构能力是从大规模自然视频中涌现的，而非来自显式几何标注。

### 合成数据构造（受控扰动/编辑构造训练对，如InstructAV2AV） ⚠️

可确认的合成仅限于文本侧：合成 caption（synthetic captions）由 Gemini 生成，用于扩展与视频关联的概念多样性。[不确定] 未披露任何视觉/音频侧的合成数据构造，未提及通过受控扰动、编辑对、指令式音视频编辑对（如 InstructAV2AV 式构造）来生成训练对；Veo 3.1 具备 Insert/Remove 物体插入删除、首尾帧过渡、outpainting、风格参考等编辑类能力，这类能力通常需要成对的编辑前后训练样本，很可能存在合成构造流程，但官方无任何说明。

### 人工介入程度（人工标注、人工质检、模型初筛+人工复核） ⚠️

可确认的人工介入集中在评测与安全侧而非数据标注侧：(1) 模型效果评测由人类评分员（human raters）做 head-to-head 偏好对比（MovieGenBench、VBench-I2V）；(2) 红队测试由内部专家团队与外部招募参与者混合执行，贯穿模型开发全过程；(3) 保障性评测（assurance evaluations）由独立于模型开发团队的专门团队开发与执行，评测数据集严格保留（held out）；(4) 偏见评测涉及人工分析——用 140 种职业各生成 16 条视频，再按感知肤度（Monk 肤色量表）、感知年龄、感知性别归类；(5) 发布前由 Google DeepMind 责任与安全委员会（RSC）人工审批。[不确定] 训练数据本身的人工标注规模、人工质检抽检比例、「模型初筛+人工复核」是否存在，均未披露——从表述看 caption 与过滤基本为全自动模型驱动。

## 音视频对齐

### 音视频同步检测方法（口型同步、事件对齐） ⚠️

[不确定] 数据侧的音视频同步检测方法完全未披露。官方仅在架构层面解释同步来源：视频与音频 latent 在同一扩散过程中被联合去噪，同步性由架构保证而非后处理对齐。这意味着训练数据必须本身就是严格同步的原生配对音视频，因此必然存在某种同步质量筛选（唇形同步、事件对齐），但 Google 未披露任何检测手段。官方仅在效果层面提到 Veo 3.1 在音视频同步（audio-video synchronization）维度上表现最佳。

### 同步检测具体指标与阈值（SyncNet/Synchformer/LSE/自研，阈值数值，如MOVA LSE-D≤9.5且LSE-C≥4.5、SkyReels-V4 SyncNet |offset|≤3∧conf>1.5） ⚠️

[不确定] 完全未披露。未提及 SyncNet、AV-align 或任何自研同步指标，更无置信度阈值数值（对比 UniTalking 公开的 SyncNet conf>0.9 之类）。

### 时序同步vs语义同步的分离处理（时间对齐与内容语义匹配作为两个独立过滤条件） ⚠️

[不确定] 未披露是否将「时序对齐」与「内容语义匹配」拆分为两个独立过滤条件。可作间接线索的是安全侧的多模态分类器设计思路——官方明确指出需要判断 caption 与 video 组合后的语义结果，体现了 Google 在数据侧确实关注跨模态语义匹配问题，但该机制服务于安全而非同步质量。

### 音频质量过滤（SNR、静音检测与静音占比阈值、无音轨剔除、画外音源剔除、背景音乐分离） ⚠️

[不确定] 完全未披露。未提及 SNR 阈值、静音检测与静音占比阈值、无音轨样本剔除规则、画外音/旁白音源剔除、背景音乐分离（BGM separation）等任何具体手段。仅可从「训练视频按 quality 过滤」这一笼统表述推断音频质量应包含在内。

### 语音/音效/音乐的分类与分别处理策略 ⚠️

[不确定] 未披露语音/音效/音乐的分类与分别处理策略。模型输出层面明确区分对白、音效、环境音、背景音乐四类且可分别通过 prompt 控制，说明训练数据的 caption 中这些类别是被区分描述的，但是否存在独立的音频分类器、是否对各类音频施加不同的过滤阈值或训练权重，均无公开信息。

## 训练配合

### 多阶段训练课程与数据课程调度（阶段划分依据：分辨率/时长/质量分/模态；低清→高清、图像→视频、短→长） ⚠️

[不确定] 训练课程与数据课程调度未披露。官方仅在安全语境下区分「pre-training」与「post-training」两个阶段，未说明是否按分辨率（低清→高清）、时长（短→长）、模态（图像→视频→音视频）或质量分做阶段划分。可作弱反推：训练数据包含图像（images），且模型同时支持 720p/1080p/4K 与 4/6/8 秒多档输出，符合业界通行的「图像预训练→低分辨率视频→高分辨率视频」渐进式课程，但无官方依据。

### 数据配比随训练阶段的变化（预训练/退火annealing/SFT高质量子集） ⚠️

[不确定] 完全未披露。无预训练/退火（annealing）/SFT 各阶段的数据配比信息，也未说明是否存在高质量子集退火阶段。

### 后训练数据（SFT精选集规模与筛选标准、偏好对数量与标注方式、reward model训练数据） ⚠️

[不确定] 后训练数据完全未披露。官方提及的「post-training mitigations」仅指 SynthID 水印与生产环境输出过滤，属于部署侧干预而非数据侧后训练。未披露 SFT 精选集规模与筛选标准、偏好对（preference pairs）数量与标注方式、reward model 训练数据，也未确认是否使用 RLHF/DPO 类偏好优化。仅知模型开发目标之一是「最大化对用户 prompt 的遵循度」（maximize adherence to a user's request），沿用 Gemini 的 desiderata 优化思路，暗示存在某种偏好对齐流程。

### 数据处理基础设施与吞吐（NeMo Curator/Data-Juicer/自研，GPU加速比，处理规模，成本） ⚠️

训练基础设施可确认：使用 Google TPU（TPU Pods 集群），软件栈为 JAX 与 ML Pathways；官方强调 TPU 的高带宽内存支持大模型与大 batch，并可跨多 TPU 设备分布式训练。[不确定] 数据处理侧的基础设施完全未披露：未提及 NeMo Curator、Data-Juicer 或自研数据处理框架，无 GPU/TPU 加速比、无处理吞吐量、无处理规模与成本数据。可推测 caption 标注由 Gemini 大规模批量推理完成，成本应为数据流水线的主要开销，但无任何量化信息。

## 效果对比

### 数据策略消融的量化影响（区分：过滤严格度ablation / caption密度风格ablation / 数据配比ablation，及对应评测指标） ⚠️

[不确定] 官方未发布任何数据策略消融实验。技术报告与 Model Card 中不含过滤严格度 ablation、caption 密度/风格 ablation、数据配比 ablation 及其对应评测指标。这是 Veo 3 与开源/半开放模型（如 Movie Gen、Seedance、Wan 系列）在披露程度上的最大差距。唯一可归为「数据策略有效性」的定性表述是：合成 caption 用于提升概念多样性、语义去重用于降低输出过拟合训练数据特定元素的风险——但均无量化对照。

### 质量vs数量的证据（小而精数据超越大而杂的案例） ⚠️

[不确定] 无任何质量 vs 数量的对照证据。官方仅笼统表述训练数据「large」且经过质量过滤，未提供小而精数据超越大而杂的案例或消融结论。

### 训练数据domain分布与评测基准类目体系的对齐关系（如VABench七大类） ⚠️

[不确定] 训练数据 domain 分布与评测基准类目体系的对齐关系未披露，官方也未构建自有的类目体系化评测基准（对比 VABench 七大类）。可确认的评测体系为：(1) Meta 发布的 MovieGenBench——视频侧 1003 条 prompt、视频+音频侧 527 条 prompt，对比对象包括 MovieGen、Kling 2.0、MiniMax T2V-01、Sora Turbo/OpenAI Sora、Runway Gen-3、WAN 2.1、Kling 2.0 + MMAudio 等；(2) VBench-I2V——355 组图文对，对比 Runway Gen-4、Kling 2.0 Pro、WAN 2.1、MiniMax I2V-01。评测采用人类评分员 head-to-head 偏好对比，Veo 3 在整体偏好与 prompt 遵循度上取得 SOTA；Veo 3.1 官方称在整体偏好、文本对齐与视觉质量上表现最佳。此外安全评测使用 140 职业 × 16 视频的标准化偏见评测集，以及若干对抗性安全 prompt 数据集，这些评测类目与训练数据配比之间的对应关系未被说明。

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
- pipeline_overview
- funnel_retention_rate
- shot_segmentation
- quality_filtering
- motion_filtering
- model_as_data_judge
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
