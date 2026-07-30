# Sora 2（含 Sora 2 Pro）

> 主题: 视频生成模型（含音视频同时生成）的数据处理：数据清洗流程、数据分布、打标方式

[← 返回首页](../index.md)

**目录**: [基本信息](#基本信息) · [数据规模与分布](#数据规模与分布) · [清洗流程](#清洗流程) · [打标方式](#打标方式) · [音视频对齐](#音视频对齐) · [训练配合](#训练配合) · [效果对比](#效果对比)

## 基本信息

### 名称

Sora 2（含 Sora 2 Pro）

### 发布机构/公司

OpenAI

### 发布时间（技术报告/论文/开源时间）

2025年9月30日发布模型与 System Card（同日上线 sora.com 与独立 iOS Sora 应用）；2025年10月上旬开放 API（sora-2 / sora-2-pro）并将时长上限从10秒扩展到15秒（Pro 网页版25秒）；2025年12月与迪士尼达成三年授权协议；2026年3月OpenAI宣布关停Sora消费级应用（应用于2026年4月26日下线，API于2026年9月24日停用），迪士尼10亿美元投资与授权协议随之终止。注意：OpenAI 从未发布 Sora 2 的技术报告或论文，仅有一份7页的 System Card。

### 类型（模型/数据集/工具链/评测基准）

闭源商业模型（原生音视频联合生成的视频生成基础模型 + 消费级社交应用 + API 服务）。非数据集、非工具链、非评测基准。

### 开源程度（权重/代码/数据/pipeline各自是否开源）

全封闭。权重不开源、代码不开源、训练数据不开源、数据处理pipeline不开源。唯一公开材料为2025年9月30日的《Sora 2 System Card》（共7页），其中关于数据的内容仅第2节「Model Data & Data Filtering」一个自然段（约5句话）。没有技术报告、没有论文、没有架构细节、没有任何数据统计数字。相比之下前代 Sora 1 至少有一篇技术博客《Video generation models as world simulators》披露了时空patch、原生分辨率训练、重打标等方法论。曾以API形式（sora-2、sora-2-pro）商业开放，2026年9月API也已停用。

### 是否支持音视频同时生成，以及实现方式（原生联合/级联/MoE融合） ⚠️

支持，且为原生联合生成（native joint generation），这是 Sora 2 相对 Sora 1 的核心升级点。System Card 明确将其定位为「video and audio generation model」，新增能力包括「synchronized audio」。音频不是后处理级联的video-to-audio模块，而是与视频在同一生成管线内联合去噪产出：视频与音频分别经各自编码器压缩到latent，再由同一个transformer扩散主干对两路latent同时去噪。可生成对白（含唇形同步）、音效/foley、环境音与背景音乐，音量与空间定位随物体与镜头距离变化。注意：上述架构描述（双编码器+共享扩散主干、3D RoPE、音频backbone与GPT-4o多模态系统同源）均来自第三方技术解读与二手报道，OpenAI 官方从未确认，属于推测性信息。[不确定]

### 调研信息来源列表（论文/技术报告/官方文档/新闻的URL，标注每条来源的性质：官方一手/同团队旁证/第三方报道）

- Sora 2 System Card, OpenAI, 2025-09-30 (PDF): https://cdn.openai.com/pdf/50d5973c-c4ff-4c2d-986f-c72b5d0ff069/sora_2_system_card.pdf
- Sora 2 System Card 索引页: https://openai.com/index/sora-2-system-card/
- OpenAI Deployment Safety Hub - Sora 2: https://deploymentsafety.openai.com/sora-2
- Sora System Card (Sora 1, 含CSAM安全栈细节): https://openai.com/index/sora-system-card/
- Video generation models as world simulators (Sora 1 技术博客，含时空patch/原生分辨率/重打标): https://openai.com/index/video-generation-models-as-world-simulators/
- Sora 2 is here (发布公告): https://openai.com/index/sora-2/
- Disney 与 OpenAI 授权协议公告: https://openai.com/index/disney-sora-agreement/
- Sora, Not Sorry: OpenAI Backtracks on Opt-Out Copyright Policy, Copyright Lately: https://copyrightlately.com/openai-backtracks-sora-opt-out-copyright-policy/
- Sora 2 Does A Copyright Somersault Upon Launch, Forbes, 2025-10-17: https://www.forbes.com/sites/legalentertainment/2025/10/17/sora-2-does-a-copyright-somersault-upon-launch/
- MPA 要求 Sora 2 停止侵权, CNBC, 2025-10-07: https://www.cnbc.com/2025/10/07/openais-sora-2-must-stop-allowing-copyright-infringement-mpa-says.html
- 日本政府就 Sora 2 侵权向 OpenAI 提出要求, EC IP Helpdesk: https://intellectual-property-helpdesk.ec.europa.eu/news-events/news/japanese-government-requests-openai-avoid-copyright-infringement-sora-2-us-federal-judge-dismisses-2025-10-23_en
- Public Citizen 要求 OpenAI 撤回 Sora 2 的公开信: https://www.citizen.org/news/public-citizen-letter-calls-on-open-ai-to-withdraw-sora-2-video-generation/
- OpenAI Will Shut Down Sora Video App; Disney Drops $1B Investment, Variety, 2026-03: https://variety.com/2026/digital/news/openai-shutting-down-sora-video-disney-1236698277/
- Sora Shutting Down, Disney Investment Dead, Deadline, 2026-03: https://deadline.com/2026/03/sora-shut-down-disney-investment-1236764689/
- OpenAI Adds Longer Clips and Storyboarding to Sora 2, eWeek: https://www.eweek.com/openai/openai-adds-longer-clips-sora-2/
- How OpenAI Built Sora 2: Training, Data, and Model Design (第三方技术解读，非官方): https://skywork.ai/blog/openai-sora-2-2025-ultimate-guide-training-model-design/
- How to Uncover Sora 2's Training Datasets (第三方，非官方): https://skywork.ai/blog/how-to-uncover-sora-2s-training-datasets/
- Sora 2 API on Replicate (规格与定价): https://replicate.com/openai/sora-2
- Getty Images/OpenAI 授权合作报道 (2026-06): https://finance.yahoo.com/markets/stocks/articles/getty-images-openai-deal-gives-154500732.html

## 数据规模与分布

### 训练数据量级（视频条数/小时数/token数，预训练与SFT分开） ⚠️

完全未披露。System Card 未给出任何视频条数、总小时数、token/patch 数量，未区分预训练与SFT规模，也未披露算力预算。OpenAI 对外亦无任何官方数据规模口径。[不确定]

### 数据来源构成（自有/公开数据集/网络爬取/授权采购/合成数据） ⚠️

仅有一句极高层级的定性描述，System Card 原文：「Sora 2 was trained on diverse datasets, including information that is publicly available on the internet, information that we partner with third parties to access, and information that our users or human trainers and researchers provide or generate.」即三类来源：(1) 互联网公开可得数据（网络爬取）；(2) 通过第三方合作/授权获取的数据；(3) 用户、人类训练师与研究员提供或生成的数据。未给出任何来源的占比、具体数据集名称、爬取范围、合作方名单。合成数据是否使用未明确说明（「generate」一词可能暗示包含人类训练师生成内容，但不等于模型合成数据）。[不确定]

### 数据合规与溯源（授权数据占比、rights-cleared数据集、C2PA等）

训练侧溯源披露极弱，输出侧溯源披露较强，二者需严格区分。
【训练侧】未披露授权数据占比，未公布任何 rights-cleared 数据集清单，未说明授权采购方。唯一明确的合规声明是儿童安全相关：「responsibly sourcing datasets to exclude CSAM」（负责任地筛选数据源以排除儿童性虐待材料），并与美国失踪与受虐儿童中心（NCMEC）合作。版权方面：Sora 2 上线时采取「opt-out」（版权方需主动要求排除）策略，引发SpongeBob、South Park、Scooby-Doo 等IP大规模被生成的争议；2025年10月3日（上线仅3天后）Sam Altman 发博客改为「opt-in」并承诺给版权方更细粒度控制与收益分成。美国电影协会（MPA）公开施压，日本政府亦正式要求 OpenAI 避免侵权。关键点：opt-in 政策仅约束「生成」环节，OpenAI 从未澄清该政策是否回溯适用于「训练数据」，即已被训练进模型的受版权内容并未被移除。后续 OpenAI 与迪士尼（2025年12月，三年授权、覆盖200+迪士尼/漫威/皮克斯/星战角色）、与 Getty Images（2026年6月，Getty 与 Shutterstock 37亿美元合并后的多年期合作）达成授权协议，但这些均为「生成侧IP授权/展示合作」，未明确为 Sora 2 的训练语料授权。
【输出侧】所有一方产品资产带 C2PA 元数据（行业标准可验证来源）；从 sora.com 与 Sora App 下载的视频带可见移动水印；OpenAI 保有内部检测工具判定某视频/音频是否由其产品生成。OpenAI 自承「provenance 不存在单一解法」。

### 片段时长分布与切分策略 ⚠️

训练数据的片段时长分布与切分策略完全未披露。仅知推理侧输出规格：初始10秒；2025年10月更新后所有用户15秒，ChatGPT Pro 网页版最长25秒；Sora 2 Pro API 支持 10s/15s/25s 档位。训练片段时长如何分桶、如何从长视频切分为训练clip，无任何信息。（前代 Sora 1 曾说明按原生时长训练、不做统一裁剪到固定帧数，Sora 2 是否延续未确认。）[不确定]

### 分辨率/宽高比分布与分桶策略 ⚠️

训练侧分布与分桶策略未披露。推理侧规格：Sora 2 标准版 720p，Sora 2 Pro 支持 1024p 与原生 1080p（1920x1080），同时支持竖屏（1080x1920）与横屏输出。前代 Sora 1 技术博客明确采用「native size training」——不做resize/crop/trim到固定尺寸，因而可原生采样任意宽高比，推测 Sora 2 延续该策略（可变分辨率/宽高比patch打包），但 Sora 2 官方从未确认，亦无任何分桶比例数字。[不确定]

### 类别/domain分布与配比策略（人物、动作、场景、风格等比例控制与概念均衡） ⚠️

完全未披露。System Card 未给出人物/动作/场景/风格等任何类别配比，未说明概念均衡（concept balancing）策略，未披露长尾概念处理方式。唯一可间接推断的线索来自能力描述：模型被强调在物理规律上表现更好（重力、动量、浮力、材料形变、碰撞动力学、物体恒存性），第三方解读称训练数据带有「物理标注」（physical annotations）覆盖这些概念，暗示可能存在面向物理交互的定向数据配比或标注体系——但该说法来自二手技术解读，非 OpenAI 官方表述，且无任何比例数字。此外从产品形态（cameo 真人出镜、社交feed）可推断人物/人脸类数据占比不低，但同样无官方依据。[不确定]

### 音频类别分布与配比（语音/音效foley/音乐/环境音/静音的比例及控制策略）——AV模型独有维度 ⚠️

完全未披露。作为原生音视频模型，Sora 2 在能力上明确覆盖四类音频——对白（dialogue）、音效/foley（sound effects tied to on-screen actions）、背景音乐（background music matched to scene tone）、环境音/氛围声（context-aware ambient sounds / background soundscapes），且宣称声音的音量与空间定位随物体与镜头的距离变化。但训练数据中这四类音频各占多少比例、无音轨/静音片段如何处理与保留、是否对语音-音效-音乐做显式配比控制，OpenAI 未给出任何信息。这是本次调研中信息缺口最大的维度之一：模型明显具备该能力，但数据侧构造方式零披露。[不确定]

### 叙事结构分布（单镜头vs多镜头、平均clip时长、镜头数分布、是否含原生音轨） ⚠️

未披露训练数据的镜头数分布与单/多镜头配比。能力侧披露：Sora 2 可「follow intricate instructions spanning multiple shots while accurately persisting world state」，即支持跨多镜头的指令遵循并在镜头切换间保持角色、环境、光照一致，说明训练数据中必然包含多镜头叙事样本且带有跨镜头一致性监督信号。2025年10月推出的 Storyboard（故事板）工具进一步允许用户逐段规划多场景视频。是否含原生音轨：从原生音视频联合生成能力可反推训练数据以「自带同步原声的视频」为主体，但平均clip时长、镜头数直方图、有声/无声样本比例均无数字。[不确定]

### 语言/口音分布（多语种唇同步能力的数据基础） ⚠️

完全未披露。模型具备对白生成与唇形同步能力，实际使用中可生成多语种语音，但 OpenAI 未公布支持语种列表、各语种数据占比、口音分布，也未说明多语种唇同步的数据基础。安全侧仅提及会对音频转写文本（audio transcripts）过安全分类器，间接说明存在ASR能力，但未涉及语种覆盖。[不确定]

## 清洗流程

### 清洗漏斗整体结构（几级过滤、各级顺序） ⚠️

训练数据清洗漏斗结构完全未披露。System Card 关于清洗的全部表述仅两句：「Our data processing pipeline includes rigorous filtering to maintain data quality and mitigate potential risks. We also employ a combination of safety classifiers to help prevent the use or generation of harmful or sensitive content, including explicit materials such as sexual content involving a minor.」即只承认存在(1)质量过滤与(2)风险/安全分类器两个大类，未说明级数、顺序、各级判据。相比之下，System Card 对「推理时安全栈」（输入prompt阻断 → 生成 → 输出阻断，含CSAM分类器与自定义训练的多模态推理监控模型）的描述详尽得多——这是本模型披露结构的典型特征：安全与部署侧详尽、训练数据侧近乎空白。[不确定]

### 漏斗定量保留率（每级过滤的输入/输出量与最终保留率，如Apollo 27%） ⚠️

完全未披露。没有任何一级过滤的输入/输出量、保留率或淘汰率数字。无法与 Apollo 27% 之类的公开定量漏斗做对比。[不确定]

### 镜头切分方法（PySceneDetect/自研模型/shot-aware splitting） ⚠️

完全未披露。既未提及 PySceneDetect 等开源工具，也未提及自研 shot-aware splitting 模型。虽然模型具备多镜头生成与跨镜头世界状态保持能力，暗示训练pipeline中存在镜头级切分与镜头组织逻辑，但方法零披露。[不确定]

### 质量过滤（美学评分、清晰度、OCR文字过滤、黑边/水印/logo检测） ⚠️

仅有「rigorous filtering to maintain data quality」一句笼统表述。美学评分模型、清晰度/模糊度判据、OCR文字/字幕过滤、黑边裁剪、水印与logo检测等具体手段，OpenAI 均未提及，无任何阈值或模型名称。[不确定]

### 运动过滤（光流/运动分数阈值、静态与抖动剔除） ⚠️

完全未披露。未提及光流计算、运动分数阈值、静态镜头剔除或抖动/手持晃动剔除策略。考虑到模型在物理动力学（重力、动量、碰撞）上的表现被重点宣传，推测存在面向运动质量的筛选，但无任何依据。[不确定]

### 去重方法（精确去重与基于embedding的语义去重分别记录） ⚠️

完全未披露。既未说明精确去重（哈希级），也未说明基于embedding的语义去重。[不确定]

### VLM/LLM作为质检员（多模态大模型质量打分与错配剔除，2026年从浅层打分器转向大模型语义判断的趋势） ⚠️

训练数据侧是否使用 VLM/LLM 作为质检员，未披露。但在推理安全侧，OpenAI 明确使用了大模型作为判官：输出阻断环节部署「a safety-focused reasoning monitor... a multimodal reasoning model which is custom-trained to reason about content policies」（一个为内容政策推理而定制训练的多模态推理模型），对生成视频帧、场景描述文本、音频转写进行语义级判断。这证明 OpenAI 已具备并部署了「大模型语义判断」能力栈，将同类能力复用于训练数据质检在技术上顺理成章，但 System Card 未做此陈述。此外安全评测流程中，对抗prompt由「helpful-only 版本视频模型」生成输出后再打分并转化为自动化评测，也体现了模型参与数据构造与评判的范式。[不确定]

### 安全与合规过滤（NSFW、版权、人脸/隐私）

这是训练数据侧唯一有实质披露的清洗环节。明确内容：(1) 使用「a combination of safety classifiers」防止有害或敏感内容被使用或生成，明确点名涉未成年人的性内容（CSAM）；(2) 儿童安全上采取「responsibly sourcing datasets to exclude CSAM」——即在数据源头筛除CSAM，并与NCMEC合作，对所有输入输出（含一方产品与API/企业版三方使用）执行强扫描，除非客户满足严格豁免标准；(3) 拥有专门的CSAM安全栈，复用其他产品的系统级缓解措施并叠加Sora专属保护。NSFW/版权/人脸隐私在训练数据侧未单独说明过滤方式，主要通过部署侧策略处理：不支持video-to-video、不支持公众人物的文生视频、除通过cameo显式opt-in授权的用户外阻断包含真人的生成、对疑似未成年人的上传素材施加更严阈值。

## 打标方式

### 使用的caption模型（自研VLM/开源模型，模型规模） ⚠️

完全未披露。Sora 2 未公布任何caption模型信息（自研VLM名称、规模、是否使用GPT-4o/GPT-5系列打标）。可参考前代：Sora 1 技术博客称训练了一个「highly descriptive captioner model」为训练视频生成高描述性文本caption（沿用 DALL·E 3 的 re-captioning 思路），并在推理时用 GPT 将用户短prompt扩写为长详细caption。Sora 2 极可能延续并升级该方案，但无官方确认，也无模型规模数字。[不确定]

### caption密度与结构化程度（短/长/密集描述、结构化字段如镜头运动、风格标签） ⚠️

完全未披露。无caption长度分布、密集程度、是否结构化字段（镜头运动、构图、风格标签等）的信息。前代 Sora 1 强调「highly descriptive captions」提升文本忠实度与视频质量，可推断 Sora 2 仍走长密caption路线，且从其「enhanced steerability（增强可控性）」「expanded stylistic range（扩展风格范围）」「follows intricate instructions spanning multiple shots」等能力宣称可反推caption中很可能包含风格标签、镜头/运镜描述与多镜头分段结构，但均属推测。[不确定]

### 音视频联合caption结构（是否同时覆盖视觉+听觉轨道、是否分流为独立字段，如LTX-2全音景描述、Script-a-Video factorized streams、Foley-Omni三字段） ⚠️

完全未披露。作为原生音视频模型，Sora 2 必然需要覆盖听觉轨道的标注（否则无法响应对白/音效/音乐类文本指令），实际使用中用户可通过prompt中的引号对白直接指定台词内容，说明caption schema中存在明确的对白字段或等价机制。但 OpenAI 未说明是采用单一融合caption、还是像 LTX-2（全音景描述）、Script-a-Video（factorized streams）、Foley-Omni（三字段）那样分流为视觉/对白/音效独立字段。这是与开源同类工作对比时最大的信息空白。[不确定]

### 对白转写与说话人属性标注（ASR转写+说话人身份/语言/口音/情绪） ⚠️

训练数据侧未披露。间接证据：安全栈明确将「audio transcripts（音频转写）」作为独立通道送入安全分类器，说明 OpenAI 具备并常规使用视频音轨的ASR转写能力，该能力用于训练数据打标是自然延伸。说话人身份/语言/口音/情绪等属性标注，无任何披露。产品侧的 cameo 功能要求用户录制一次性的视频+音频样本以完成身份验证并绑定其形象与声音，说明存在「说话人身份-音色-形象」的绑定表征，但这属于个性化/条件注入机制而非训练数据标注schema。[不确定]

### 几何与结构化标注（相机参数、深度、3D point tracks、动作标注、显式状态） ⚠️

完全未披露。未提及相机参数、深度图、3D point tracks、动作标注或显式物理状态标注。第三方技术解读称训练数据带有覆盖重力、动量、浮力、材料形变、碰撞动力学的「物理标注（physical annotations）」，若属实则构成显式结构化状态标注，但该说法非 OpenAI 官方表述，无法证实。OpenAI 官方仅在能力层面宣称模型「understands real-world physics」并将 Sora 2 定位为迈向物理世界模拟器（world simulator）的一步。[不确定]

### 合成数据构造（受控扰动/编辑构造训练对，如InstructAV2AV） ⚠️

完全未披露。System Card 中「information that our users or human trainers and researchers provide or generate」的「generate」一词可能涵盖人类训练师生成的内容，但是否包含模型合成数据、是否通过受控扰动/编辑构造训练对（如 InstructAV2AV 式的编辑配对），完全无信息。安全评测环节使用了「helpful-only 版本的视频模型」批量生成对抗样本输出用于构建自动化评测集，这是模型生成数据用于评测/安全对齐的明确案例，但不属于主训练数据合成。[不确定]

### 人工介入程度（人工标注、人工质检、模型初筛+人工复核） ⚠️

训练数据标注侧：仅知存在「human trainers and researchers provide or generate」的数据，说明有人工参与数据提供与生成，但规模、流程、是否人工质检复核均未披露。部署侧人工介入有明确说明：与外部 OpenAI Red Team Network 红队成员合作测试（覆盖性内容、裸露、极端主义、自残、违法行为、暴力血腥、政治说服，以及青少年安全与肖像使用等专项政策），红队反馈用于调整prompt过滤器、屏蔽词表与分类器阈值；内容审核采用「自动化 + 人工复核」组合以识别滥用模式，并提供应用内举报通道与申诉机制。[不确定]

## 音视频对齐

### 音视频同步检测方法（口型同步、事件对齐） ⚠️

训练数据侧的音视频同步检测方法完全未披露——这是本次调研最核心的关切点，而 OpenAI 恰恰零披露。System Card 中没有任何关于唇形同步检测、事件对齐检测、异步样本剔除的表述。能力侧仅宣称音频「properly synchronized with on-screen action, including accurate lip-sync for speaking characters」，以及音效与画面事件绑定、音乐节奏匹配场景节奏。可以确定训练pipeline中必然存在某种AV同步质量控制（否则无法学得唇同步），但方法、模块、判据全部未知。[不确定]

### 同步检测具体指标与阈值（SyncNet/Synchformer/LSE/自研，阈值数值，如MOVA LSE-D≤9.5且LSE-C≥4.5、SkyReels-V4 SyncNet |offset|≤3∧conf>1.5） ⚠️

完全未披露。无 SyncNet、AV-align 或任何自研同步指标的名称，无任何置信度阈值数值（无法与 UniTalking 的 SyncNet conf>0.9 之类做对比）。OpenAI 亦未公布 Sora 2 在任何第三方AV同步评测基准上的分数。[不确定]

### 时序同步vs语义同步的分离处理（时间对齐与内容语义匹配作为两个独立过滤条件） ⚠️

完全未披露。是否将「时间轴对齐」与「内容语义匹配」拆分为两个独立过滤条件，无任何信息。从能力描述可看出模型同时具备时序对齐（唇形与语音逐帧对齐、音效与碰撞时刻对齐）与语义匹配（环境音符合场景语境、音乐情绪匹配画面基调）两类能力，暗示数据侧可能有分离处理，但纯属推断。[不确定]

### 音频质量过滤（SNR、静音检测与静音占比阈值、无音轨剔除、画外音源剔除、背景音乐分离） ⚠️

完全未披露。未提及信噪比阈值、静音检测与静音占比阈值、无音轨样本剔除策略、画外音/旁白源剔除、背景音乐分离（source separation）等任何手段。[不确定]

### 语音/音效/音乐的分类与分别处理策略 ⚠️

完全未披露。模型输出侧明确区分对白、音效/foley、音乐、环境音四类，但训练数据侧是否对这四类做显式分类并分别设计过滤/配比/标注策略，OpenAI 未做任何说明。[不确定]

## 训练配合

### 多阶段训练课程与数据课程调度（阶段划分依据：分辨率/时长/质量分/模态；低清→高清、图像→视频、短→长） ⚠️

完全未披露。无阶段划分信息（低清→高清、图像→视频、短→长、静音→有声等），无任何按分辨率/时长/质量分/模态调度数据课程的描述。可间接推断的线索：产品分为 Sora 2（720p）与 Sora 2 Pro（1080p）两档，且时长上限从10秒逐步放开到15/25秒，符合分辨率与时长渐进式课程的一般规律，但这属于产品分层与推理配置，不能直接等同于训练课程。[不确定]

### 数据配比随训练阶段的变化（预训练/退火annealing/SFT高质量子集） ⚠️

完全未披露。预训练/退火（annealing）/SFT 各阶段的数据配比变化，无任何信息。[不确定]

### 后训练数据（SFT精选集规模与筛选标准、偏好对数量与标注方式、reward model训练数据） ⚠️

完全未披露训练用后训练数据。无SFT精选集规模与筛选标准，无偏好对数量与标注方式，无reward model训练数据说明。唯一相关的公开信息属于安全对齐评测而非模型能力后训练：OpenAI 通过定向红队收集了「数千条对抗性prompt」，按用例与政策领域分类，用 helpful-only 版本视频模型生成输出后打分，并转化为自动化评测集，用于测量生产安全栈的 not_unsafe（拦截召回）与 not_overrefuse（避免误拦）两项指标。公布结果：成人裸露/性内容（不涉肖像）96.04%/96.20%；成人裸露/性内容（涉肖像）98.40%/97.60%；自残 99.70%/94.60%；暴力血腥 95.10%/97.00%；违规政治说服 95.52%/98.67%；极端主义/仇恨 96.82%/99.11%。[不确定]

### 数据处理基础设施与吞吐（NeMo Curator/Data-Juicer/自研，GPU加速比，处理规模，成本） ⚠️

完全未披露。未提及 NeMo Curator、Data-Juicer 或自研数据处理框架，无GPU加速比、处理规模或成本数据。间接的经济性线索来自媒体报道：Sora 服务运行成本被报道为约每日1500万美元而收入仅约210万美元，活跃用户在2026年初跌破50万，被认为是OpenAI于2026年3月关停Sora消费应用的主因——这反映的是推理侧算力经济性而非数据处理吞吐。[不确定]

## 效果对比

### 数据策略消融的量化影响（区分：过滤严格度ablation / caption密度风格ablation / 数据配比ablation，及对应评测指标） ⚠️

完全未披露。System Card 不含任何消融实验，无过滤严格度ablation、无caption密度/风格ablation、无数据配比ablation，也没有对应的评测指标对比表。OpenAI 未发布 Sora 2 在 VBench 等任何公开视频生成基准上的官方成绩。System Card 中唯一的量化表格是上述六类安全指标。[不确定]

### 质量vs数量的证据（小而精数据超越大而杂的案例） ⚠️

完全未披露。无任何「小而精数据超越大而杂」的实验证据或论述。System Card 仅有「rigorous filtering to maintain data quality」这一句定性表态，隐含重视数据质量的立场，但无任何支撑证据。[不确定]

### 训练数据domain分布与评测基准类目体系的对齐关系（如VABench七大类） ⚠️

完全未披露。OpenAI 未公布 Sora 2 训练数据的domain类目体系，也未将其与任何评测基准（如 VABench 七大类、VBench 等）的类目做对齐说明。OpenAI 自建的唯一类目体系是安全政策类目（成人性内容/自残/暴力血腥/政治说服/极端主义仇恨，及是否涉及肖像的二分），该体系服务于安全评测，与训练数据domain分布无对应关系。[不确定]

## 其他信息

### summary_note

核心结论：Sora 2 是本主题下「数据披露程度最低」的代表性样本。其 System Card 全长7页，涉及训练数据的内容仅第2节一个自然段（约5句、无一个数字），而安全栈、使用政策、溯源工具、红队与安全评测占据了全文约85%的篇幅。全部42个调研字段中，仅 name/organization/release_date/type/openness/av_generation/provenance_licensing/safety_filtering 8项可基于官方材料给出实质回答，其余34项（涵盖数据规模、清洗漏斗、打标schema、音视频对齐检测、训练课程、消融证据等全部技术性维度）均无官方信息。OpenAI 亦从未发布 Sora 2 技术报告或论文，延续了自 GPT-4 起「不披露模型与实现细节」的做法。因此本条目在数据处理方法论层面的参考价值极低，其主要研究价值在于：(1) 作为原生音视频联合生成的能力标杆与产品形态样本；(2) 作为「输出侧溯源治理（C2PA+水印+内部检测）详尽 vs 训练侧数据溯源空白」这一不对称披露模式的典型案例；(3) 其版权 opt-out→opt-in 政策急转、以及 opt-in 不回溯适用于训练数据的争议，是训练数据合规议题的重要案例。如需可复现的音视频数据处理方法，应转向 LTX-2、Ovi、MMAudio、Veo 3 技术材料及相关开源工作。

## 不确定字段

以下字段的调研信息部分不确定（⚠️ 标注来源）：

- av_generation
- data_scale
- data_sources
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
