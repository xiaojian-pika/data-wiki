# SpeakerVid-5M（论文全称：SpeakerVid-5M: A Large-Scale High-Quality Dataset for Audio-Visual Dyadic Interactive Human Generation）

> 主题: 视频生成模型（含音视频同时生成）的数据处理：数据清洗流程、数据分布、打标方式

[← 返回首页](../index.md)

**目录**: [基本信息](#基本信息) · [数据规模与分布](#数据规模与分布) · [清洗流程](#清洗流程) · [打标方式](#打标方式) · [音视频对齐](#音视频对齐) · [训练配合](#训练配合) · [效果对比](#效果对比)

## 基本信息

### 名称

SpeakerVid-5M（论文全称：SpeakerVid-5M: A Large-Scale High-Quality Dataset for Audio-Visual Dyadic Interactive Human Generation）

### 发布机构/公司

清华大学（深圳国际研究生院，Tsinghua University）联合 StepFun（阶跃星辰）与香港科技大学（HKUST，含广州校区）。作者列表：Youliang Zhang、Zhaoyang Li、Duomin Wang、Jiahe Zhang、Deyu Zhou、Zixin Yin、Xili Dai、Gang Yu、Xiu Li。项目负责人（Project Lead）为 Duomin Wang（王多民，StepFun），通讯作者为李秀（Xiu Li，清华大学）。Gang Yu（于刚）为 StepFun 视觉团队负责人。

### 发布时间（技术报告/论文/开源时间）

2025年7月14日首次提交 arXiv（arXiv:2507.09862，cs.CV）。配套资源随后陆续开放：HuggingFace 数据集仓库 dorni/SpeakerVid-5M-Dataset 创建于 2025年7月18日，最后更新于 2025年8月4日；GitHub 数据清洗代码库 Dorniwang/SpeakerVid-5M-Code 与项目主页 https://dorniwang.github.io/SpeakerVid-5M/ 同期上线。

### 类型（模型/数据集/工具链/评测基准）

数据集（大规模音视频双人交互人体生成数据集）为主体，同时附带两项衍生产出：(1) 一个自回归式 video chat 基线模型（0.8B 可训练参数）；(2) 一个配套评测基准 VidChatBench（500 组测试对，含六维指标体系）。此外开源了完整的数据清洗（data curation）代码库，因此也兼具工具链属性。它不是生成模型本身，而是 MOVA 等音视频联合生成模型的上游数据供给方之一。

### 开源程度（权重/代码/数据/pipeline各自是否开源） ⚠️

属于「元数据+标注全开、原始视频不托管」的典型学术数据集开放模式：
【数据（标注与索引）】开源。HuggingFace dorni/SpeakerVid-5M-Dataset 提供 all_data_list.json（YouTube 视频 ID + 切分后 clip 名称，clip 名即定位标注的唯一键）与 SFT_set.json（高质量子集清单），以及五类标注文件夹：merge_anno（clip 级元数据：时间戳、空间 bbox、clear/DOVER 质量分）、asr（Whisper 转写与置信度）、l_score（人脸/手部清晰度即模糊分）、anno（Qwen-VL 生成的 MLLM 结构化 caption）、dwpose（因体积过大未上传实际骨架序列，仅提供计算代码）。
【数据（原始视频）】不托管。需用户依据 YouTube video ID 自行用 yt-dlp 下载，存在链接失效（link rot）导致的可复现性衰减风险。
【数据处理 pipeline 代码】开源，这是本条目最有价值的部分之一。GitHub Dorniwang/SpeakerVid-5M-Code 发布了六段式完整清洗流程代码：base annotation（音视频同步抽取 + 单人检测）、DWpose 骨架标注、ASR 标注、blur score 计算、luminance 计算、scene detection + speaker diarization（部分预计算，可选）。
【基线模型权重】论文与代码库均未见权重开源的明确说明。[不确定]
【许可】明确限定为「non-commercial, scientific research, and educational purposes only」（仅限非商业的科研与教育用途），显式禁止商业使用；内容源自公开互联网，版权归原创作者所有，并提供 takedown 政策供版权方申请下架。未采用 Apache/CC 等标准 SPDX 许可证。

### 是否支持音视频同时生成，以及实现方式（原生联合/级联/MoE融合）

数据集层面：原生音视频成对（每个 clip 都自带与画面同步的原生音轨，无音轨或音视频不同步的样本在清洗阶段即被剔除），因此它是「音视频联合生成」训练的合格语料，也是当前少数以双人对话交互（dyadic interaction）为核心组织形态的音视频数据集。
基线模型层面：支持音视频同时生成，实现方式为原生联合的自回归框架，而非级联。具体构成为 Qwen2.5-Omni 作多模态理解主干 → 同时预测视频 token 与音频 token；视频侧用 3D VAE（时间 stride 4、空间 stride 8）编码，音频侧用 CosyVoice2 audio tokenizer；再接 spatial transformer 做逐帧精修、diffusion MLP 做视觉细节增强。可训练参数 0.8B。训练中引入 noise injection 策略缓解自回归的误差累积。
该数据集被下游多个音视频联合生成模型直接采用为训练语料，典型如 MOVA 将 SpeakerVid-5M 列为其 Phase 1 数据来源之一，并称其为唇同步（lip-sync）能力的核心来源。

### 调研信息来源列表（论文/技术报告/官方文档/新闻的URL，标注每条来源的性质：官方一手/同团队旁证/第三方报道）

- 【官方一手】arXiv:2507.09862《SpeakerVid-5M: A Large-Scale High-Quality Dataset for Audio-Visual Dyadic Interactive Human Generation》（2025年7月14日提交，cs.CV）：https://arxiv.org/abs/2507.09862 ，全文 HTML https://arxiv.org/html/2507.09862v1 —— 本条目绝大多数字段的直接来源，特别是第3节数据构建 pipeline、质量过滤阈值、四分支组织、Table 1 数据集对比与 Table 2 消融结果。
- 【官方一手】项目主页 https://dorniwang.github.io/SpeakerVid-5M/ —— 作者所属机构（清华大学、StepFun、香港科技大学/广州）、项目负责人 Duomin Wang 与通讯作者 Xiu Li、资源发布状态与各链接入口。
- 【官方一手】GitHub 数据清洗代码库 https://github.com/Dorniwang/SpeakerVid-5M-Code —— 六段式 curation pipeline 代码清单、依赖工具栈（3D-Speaker/YOLO/DWpose/Whisper/SceneDetect/SyncNet/ArcFace/Deepface/UniSpeech/Deep3DFaceRecon/yt-dlp）、以及「仅限非商业科研教育用途、版权归原作者、提供 takedown 政策」的许可条款。
- 【官方一手】GitHub 项目页仓库 https://github.com/Dorniwang/SpeakerVid-5M —— 项目主页源。
- 【官方一手】HuggingFace 数据集 https://huggingface.co/datasets/dorni/SpeakerVid-5M-Dataset （创建于 2025-07-18，最后更新 2025-08-04，约 1021 次下载，18 likes）—— 实际发布内容清单：all_data_list.json、SFT_set.json 与 merge_anno / dwpose / asr / l_score / anno 五类标注文件夹；确认原始视频不托管、需按 YouTube ID 自行下载；确认 dwpose 骨架数据因体积未上传。
- 【同团队旁证】arXiv:2602.08794《MOVA: Towards Scalable and Synchronized Video–Audio Generation》—— 非同团队，但为下游使用方，其第3节将 SpeakerVid-5M 列为 Phase 1 训练数据来源并明确定位为唇同步能力的核心来源，用于交叉印证本数据集的语种重心与音频类别属性（属旁证，非 SpeakerVid-5M 官方表述）。

## 数据规模与分布

### 训练数据量级（视频条数/小时数/token数，预训练与SFT分开）

论文以「clip 条数 + 小时数 + 说话人 ID 数」三口径给出，未给出 token 数：
【原始采集】153K 条音视频视频，共 64,386 小时原始素材。
【最终数据集总量】超过 5.2M（520万）视频 clip，超过 8,743 小时。
【单人分支（single branch，含 talking 与 listening）】5.2M clips / 8.7K 小时 / 83K 个唯一说话人 ID。
【对话分支（dialogue branch）】770K 组 clip pair / 1.8K 小时 / 16K 个唯一说话人 ID。
【高质量 SFT 子集】571K clips / 1,368 小时。
【预训练与 SFT 的拆分】基线模型预训练使用 7,375 小时（即 8,743 − 1,368，为剔除 HQ 子集后的剩余大规模数据），SFT/微调使用 571K clips / 1,368 小时的高质量子集。这是一个明确的「大规模预训练 + 小而精后训练」两段式规模设计，HQ 子集约占总时长的 15.6%。

### 数据来源构成（自有/公开数据集/网络爬取/授权采购/合成数据）

单一来源：YouTube 网络爬取，无自有拍摄数据、无授权采购、无合成数据。
【采集方式】人工筛选（manually collected）高质量的双人对话类 YouTube 视频，覆盖 2006 年至 2025 年 6 月（论文写作时的当下）近二十年跨度。
【体裁构成】访谈（interviews）、新闻报道（news reports）、研讨会/讲座（seminars）、电视节目（television programs）、综艺（variety shows）、辩论（debates）、教育类视频（educational videos）。
【YouTube 频道类目构成】娱乐（entertainment）、人物与博客（people and blogs）、喜剧（comedy）、新闻与政治（news and politics）、教育（education）、科学（science）。
【与公开数据集的关系】不复用 CelebV-HQ、HDTF、MultiTalk、OpenHumanVid 等已有数据集，是完全独立采集的新语料。论文 Table 1 将其与上述数据集对比：相比 OpenHumanVid 的 13.4M clips / 16.7K 小时，SpeakerVid-5M 在总量上不占优，但在「双人交互配对」「说话人 ID 规模」「1080P 占比」「结构化身体构图标注」等维度是独有的。
【合成数据】无。

### 数据合规与溯源（授权数据占比、rights-cleared数据集、C2PA等） ⚠️

披露不完整，属于典型的学术数据集合规姿态：
【已声明】(1) 明确限定非商业的科研与教育用途，显式禁止商业使用；(2) 声明内容来自公开互联网，版权归原始创作者所有；(3) 提供 takedown 政策，版权方可申请移除；(4) 不托管原始视频，仅发布 YouTube video ID 与标注，将版权风险转移给使用方自行下载——这是规避直接分发版权内容的常见做法。
【未涉及】未给出授权数据占比（实际为 0%，全部为爬取）；未使用任何 rights-cleared 商用数据集；未提及 C2PA、内容溯源水印或来源可信标记；未讨论出镜人物的肖像权与知情同意问题（数据全部为真人面部与身体，含 83K 个可识别说话人身份，且带 ArcFace 人脸特征做 ID 关联，隐私敏感度实际很高，但论文未作任何隐私影响评估）。
【下游影响】由于许可限定非商业，任何将 SpeakerVid-5M 用于商用模型训练的行为都与其条款冲突——这一点对将其列为训练数据来源的下游模型（如 MOVA 采用 Apache-2.0 允许商用）构成潜在的许可传导矛盾，但论文与下游工作均未讨论。[不确定]

### 片段时长分布与切分策略

采用变长切分，长度带被严格限定在一个窄区间：
【切分策略】先用 PySceneDetect 做场景切分，再将 clip 修剪（trim）到 3 至 14 秒之间；短于 3 秒的过碎片段与长于 14 秒的过长片段被排除或截断。
【平均时长推算】单人分支 8.7K 小时 / 5.2M clips ≈ 6.0 秒/clip；对话分支 1.8K 小时 / 770K pair ≈ 8.4 秒/pair（若按 pair 计双路则单路约 4.2 秒）。总体 8,743 小时 / 5.2M clips ≈ 6.05 秒，位于 3–14 秒区间偏下部。
【与固定时长方案的对比】不同于 MOVA 的 8.05 秒定长窗口，SpeakerVid-5M 保留原始语义边界带来的变长特性，切分点由场景变化与说话轮次共同决定，更贴近「一个自然对话轮次（turn）」的粒度。
【分布图】Figure 3 中给出了时长分布直方图，但论文正文未给出各时长桶的具体百分比。
【多轮分支的时间跨度】multi-turn 分支在当前轮次时间戳 x 之前取长度为 T 的历史窗口 [x−T, x] 聚合历史轮次；相邻 clip 时间间隔小于阈值 δt 的被判定为连续对话，可拼接成更长更自然的对话序列，因此多轮分支的有效上下文时长可远超单 clip 的 14 秒上限。

### 分辨率/宽高比分布与分桶策略 ⚠️

【原生分辨率】数据集保留源视频的原生分辨率，且分辨率档位极高：93% 的视频为 1080P 或更高，98% 超过 720P。这是其相对 CelebV-HQ、HDTF 等旧数据集的核心质量优势之一，Figure 3 给出了分辨率分布图。
【裁剪策略】不做统一的分辨率归一化或黑边补齐，而是基于 YOLO 的人体检测与跟踪结果对每个说话人做时空裁剪（temporal and spatial cropping），即从原始画面中裁出以单个说话人为中心的子区域。因此实际 clip 的宽高比由人体框决定，而非固定 16:9 或 9:16。
【身体构图分桶】论文以「身体构图」而非宽高比作为主要的画幅分桶维度，caption 中显式标注 half-body（半身）vs full-body（全身）、以及正面/侧面朝向（facing direction / body orientation），覆盖全身、半身、侧身多种取景，这是 Table 1 中相对以头部为主的 CelebV-HQ 等数据集的差异化标注项。
【训练侧】基线模型训练与推理时统一将帧标准化为 480×768 分辨率（竖向 5:8 近似比例），3D VAE 时间 stride 4、空间 stride 8。即数据侧高清多样、训练侧统一降采样。
【分桶策略】论文未描述基于宽高比的多桶训练（aspect-ratio bucketing）策略。[不确定]

### 类别/domain分布与配比策略（人物、动作、场景、风格等比例控制与概念均衡） ⚠️

论文给出定性类目与分布图，但未提供任何比例数字：
【交互形态维度（数据集的主组织轴）】这是 SpeakerVid-5M 最独特的 domain 划分方式——按交互角色而非视觉内容划分为四大分支：(1) dialogue 双人对话分支；(2) single 单人分支（monadic talking）；(3) listening 倾听分支（对话中的非说话侧反应行为）；(4) multi-turn 多轮分支。四分支同源于同一批原始视频，通过不同的组织与配对方式派生。这套「说 / 听 / 单向 / 多轮」的分类体系直接对应下游任务的能力拆分。
【内容体裁维度】访谈、新闻报道、研讨会、电视节目、综艺、辩论、教育视频。
【YouTube 频道类目维度】娱乐、人物与博客、喜剧、新闻与政治、教育、科学。Figure 3(c) 以图形展示了 topic 分布与年份分布（2006–2025），但正文未列百分比。
【概念均衡机制】论文未描述任何显式的类目配比控制或概念均衡（concept balancing）策略，采集侧只有「人工筛选高质量双人对话视频」这一定性准则。
【人物中心的强偏置】全部数据均为真人说话/倾听场景，不含物体运动、自然风景、动画等非人物 domain，是一个高度垂直的人物-对白数据集，而非通用视频生成数据集。[不确定]

### 音频类别分布与配比（语音/音效foley/音乐/环境音/静音的比例及控制策略）——AV模型独有维度 ⚠️

SpeakerVid-5M 是一个纯语音导向（speech-only）的数据集，音频类别配比是隐式且极端的：
【设计取向】所有分支均围绕人类说话与倾听构建，音轨内容以人声对白为绝对主体，不存在为音效（foley）、音乐、环境音单独设立的子集或配比。
【隐式的非语音剔除机制】音频质量过滤中设有 no-speech probability > 0.8 即剔除的规则（来自 Whisper 的无语音概率），这一条直接把纯音效、纯音乐、纯环境音、静音片段排除在数据集之外；ASR 置信度低于 −1.5 的样本同样被剔除。因此最终语料在音频类别上近似 100% 为语音主导片段。
【倾听分支的音频特殊性】listening 分支的画面主体是不说话的一方，但其配对音轨仍是对方的语音，因此音频侧依然是语音，只是与画面主体的口型无对应关系——这是数据集内部唯一的「音画角色错位」设计，且是刻意为之（用于训练倾听者的反应生成）。
【未做的处理】论文未描述背景音乐分离（BGM separation）、语音增强、SNR 估计、音效/音乐分类器（如 EAT）等任何音频类别细分处理，也未给出语音/非语音/静音的比例统计。相比 MOVA 明确统计「语音片段占预处理片段 69.47%」，SpeakerVid-5M 在此维度是信息空白。
【下游影响】正因其纯语音属性，MOVA 等下游模型将其定位为唇同步能力的专项数据源，而通用音效与音乐能力需从 VGGSound、WavCaps 等其他数据集获取。[不确定]

### 叙事结构分布（单镜头vs多镜头、平均clip时长、镜头数分布、是否含原生音轨） ⚠️

【单镜头 vs 多镜头】数据集是严格的单镜头（single-shot）语料。切分第一步即用 PySceneDetect 检测颜色与亮度的显著变化以定位转场，并在转场处切断，保证每个 clip 内部无镜头切换；随后再用 YOLO 跟踪做空间裁剪，进一步保证 clip 内主体连续。因此不存在多镜头样本，也不存在镜头数分布——这与 MOVA 刻意保留「多场景片段」类目形成明确对比。
【平均 clip 时长】约 6.0 秒（3–14 秒区间内），详见 duration_distribution。
【是否含原生音轨】全部含原生同步音轨。数据集从采集起就是 audio-visual 成对的（原始采集口径即为「153K audio-visual videos」），且经过 SyncNet 唇音同步校验与 Whisper 音频质量过滤，无音轨或音画失步的样本被剔除。
【更高层的叙事结构：对话轮次】SpeakerVid-5M 引入了视频数据集中罕见的「对话轮次（turn）」结构层：dialogue 分支把同一时段的两个说话人 clip 配成 pair；multi-turn 分支进一步在时间轴上串联多个轮次——contextual 型将前序轮次的 ASR 转写聚合为多轮对话上下文，sequential 型将时间间隔小于阈值 δt 的相邻 clip 判为连续对话并拼接。这使得数据集的叙事结构不是「镜头序列」而是「对话轮次序列」，是一种面向交互生成任务的独特叙事组织。
【各分支的轮次数分布】论文未给出多轮分支的轮次数分布统计。[不确定]

### 语言/口音分布（多语种唇同步能力的数据基础） ⚠️

披露极少，是本数据集最明显的统计空白之一：
【已知信息】数据源全部为 YouTube 公开视频，采集口径为「高质量双人对话视频」，体裁为访谈/新闻/研讨会/综艺/辩论/教育，未限定语种。使用 Whisper 做 ASR 转写，Whisper 本身具备语种识别能力，且清洗流程中提到会依据「detected language mismatches（检测到的语言不匹配）」做过滤，说明 pipeline 内部确实记录了语种标签。
【未披露】论文与数据卡均未给出支持语种清单、各语种的 clip 数或小时数占比、也未给出口音（accent）类别的任何标注或统计。caption 的结构化字段中不含语言或口音项，ASR 标注仅有转写文本与置信度。
【间接推断】考虑到源自 YouTube 的访谈/新闻/辩论/教育类内容、以及娱乐与新闻政治类频道占比，语料很可能以英语为主导，但论文明确未作此声明，不应当作事实引用。
【下游用途】MOVA 将 SpeakerVid-5M 与 OpenHumanVid、YouTube 内容一并作为其英文唇同步能力的数据基础，中文能力则另由 in-house 中文剧集提供，这从侧面印证 SpeakerVid-5M 的语种重心偏英文，但属旁证而非原文表述。[不确定]

## 清洗流程

### 清洗漏斗整体结构（几级过滤、各级顺序）

整体为「五步结构化处理 → 五维质量过滤 → 多模态标注 → 四分支组织」的漏斗，全流程代码已开源（Dorniwang/SpeakerVid-5M-Code，六段式脚本）：
【第一阶段：结构化处理（5 步，严格顺序执行）】
  1) 场景切分（Scene Splitting）：PySceneDetect 依据颜色与亮度变化检测视觉显著转场，切分并将 clip 修剪至 3–14 秒。
  2) 说话人分离（Speaker Diarization）：3D-Speaker 工具做声纹聚类，按说话时长/频次挑出两个主说话人 ID（二者合计占总说话时长 80% 以上），其余说话人丢弃。
  3) 人体检测与跟踪（Human Detection & Tracking）：YOLO 做时空跟踪，对每个人做时间与空间裁剪，得到以单人为中心的子片段。
  4) 唇音同步关联（Lip Synchronization）：SyncNet 计算音视频重合度，用 confidence score 把音频侧的说话人 ID 绑定到视觉侧的人体 bbox 上——这一步是「声音属于画面中哪个人」的关键裁决。
  5) ID 校正（ID Correction）：ArcFace 提取人脸特征，跨 clip 计算面部余弦相似度验证说话人身份一致性，离群样本若与其他 ID 相似度更高则重新分配。
【第二阶段：质量过滤（五维并行）】亮度、视频质量（DOVER）、清晰度（clarity = B/(W×H) 的比特率密度）、运动模糊（人脸/手部 Laplacian 方差）、音频质量（Whisper 置信度/无语音概率/压缩比）。
【第三阶段：多模态标注】Qwen2.5-VL 结构化 caption、Qwen-3 话题分类、Whisper ASR、DWpose 骨架、SyncNet 指标、Laplacian 模糊分、Qwen2.5-VL 多 persona 运动强度打分。
【第四阶段：分支组织】按 talking/listening/dialogue/multi-turn 四类交互形态重组为四个分支，并按 HQ 阈值切出 571K clips 的 SFT 子集。
【设计特征】与通用视频数据集不同，本 pipeline 的核心不是「筛掉坏视频」而是「解析交互结构」——diarization、SyncNet 绑定、ArcFace 校正三步都在回答「谁在说、谁在听、他在画面的哪里、跨片段是不是同一个人」，质量过滤反而是配角。

### 漏斗定量保留率（每级过滤的输入/输出量与最终保留率，如Apollo 27%） ⚠️

论文未提供逐级保留率表格，只能从首尾两个口径做端到端推算：
【入口】153K 条视频 / 64,386 小时原始素材。
【出口】超过 5.2M clips / 8,743 小时（含单人分支 8.7K 小时 + 对话分支 1.8K 小时，两者存在样本重叠，总量以 8,743 小时计）。
【端到端时长保留率】8,743 / 64,386 ≈ 13.6%。这一保留率显著低于 MOVA 的 26.39% 与 Apollo 的 27%，但口径不完全可比——SpeakerVid-5M 的损耗大头并非质量过滤，而是结构性损耗：只保留两个主说话人（其余人物丢弃）、只保留有明确说话/倾听行为的时段、以及 YOLO 单人裁剪后的画面区域损失，均会大量削减有效时长。
【HQ 子集保留率】1,368 / 8,743 ≈ 15.6%（相对最终数据集）；1,368 / 64,386 ≈ 2.1%（相对原始素材）。即从原始素材到可用于 SFT 的高质量对话数据，等效保留率约五十分之一。
【未披露】各单步（场景切分、diarization、SyncNet 绑定、ArcFace 校正、以及五维质量过滤中每一维）的输入/输出量与逐级淘汰率均未给出，Figure 3 中虽有 DOVER 分数与 SyncNet 置信度的分布直方图，但无法据此反推淘汰比例。这是本数据集披露体系中相对薄弱的一环。[不确定]

### 镜头切分方法（PySceneDetect/自研模型/shot-aware splitting）

使用 PySceneDetect（SceneDetect）作为唯一的镜头切分工具，无自研切分模型：
【检测原理】分析画面颜色与亮度的变化以定位「视觉显著转场（visually significant transitions）」，在转场处切断。
【长度约束】切分后的 clip 被修剪到 3 至 14 秒；不足或超长的做丢弃/截断处理。
【与说话人结构的耦合】场景切分只是第一步，真正决定最终 clip 边界的是后续的三重约束：(1) 3D-Speaker 的说话人时间区间；(2) YOLO 人体跟踪的时空连续段；(3) SyncNet 的音画绑定有效区间。因此 SpeakerVid-5M 的切分本质是「shot-aware + speaker-aware + track-aware」的三重感知切分，而非单纯的场景切分。
【与 MOVA 的对比】MOVA 用 PySceneDetect 场景切点 + Silero VAD 语音边界联合生成 8.05 秒定长窗口，刻意保留多镜头样本；SpeakerVid-5M 则用 PySceneDetect 切断后再按说话人轨迹裁剪，产出严格单镜头的变长片段。两者都是 shot-aware，但目标不同：前者服务于「学会转场」，后者服务于「学会人物交互」。
【代码可得性】场景检测与说话人分离的代码在开源仓库中作为第 6 步提供（标注为可选，部分结果已预计算）。

### 质量过滤（美学评分、清晰度、OCR文字过滤、黑边/水印/logo检测） ⚠️

五维阈值化过滤，全部阈值公开，另设更严的 HQ 阈值组：
【1. 亮度（Luminance）】luminance score < 10 或 > 210 的 clip 被剔除（灰度 0–255 区间的两端），用于排除过暗与过曝画面。
【2. 视频质量（DOVER）】使用 DOVER 视频质量评估工具的融合分（fused score），< 0.25 的剔除。
【3. 清晰度（Clarity Score）】自定义指标，定义为 clarity = B/(W×H)，即码率（bitrate）除以分辨率面积——衡量单位像素上的比特密度，可有效识别「高分辨率但实为低清上采样/重压缩」的伪高清视频。按分数排序后剔除最低 5%。这是本数据集较有特色的一个自定义质量指标。
【4. 运动模糊（Motion Blur）】对人脸与手部区域各裁 128×128 patch，计算 Laplacian 方差作为清晰度/模糊分；人脸或手部的平均模糊分 < 0.1 的剔除。分人脸与手部两路独立打分，是为下游手势生成任务专门设计的。
【5. 音频质量】三条并列否决规则，满足任一即剔除：Whisper 转写置信度 < −1.5；no-speech probability（无语音概率）> 0.8；compression ratio（压缩比）> 2.5（用于识别 ASR 重复退化/幻觉输出）。
【HQ / SFT 子集阈值（更严）】手部模糊分 > 0.5、人脸模糊分 > 0.7、DOVER > 0.6、运动强度分 > 2、ASR 置信度 > −1。五条全部满足才入选，得到 571K clips / 1,368 小时。
【未做的过滤】论文未描述美学评分（aesthetic score）单独打分、OCR 烧录字幕检测与过滤、水印/logo 检测、黑边检测。DOVER 本身含美学视角分支，但论文只使用其融合分而未拆分使用。[不确定]

### 运动过滤（光流/运动分数阈值、静态与抖动剔除）

SpeakerVid-5M 不做传统的光流/运动幅度阈值过滤，而是把运动强度做成一个可检索的标注维度，并仅在 HQ 子集环节用作筛选条件：
【运动强度标注方法】用 Qwen2.5-VL 做多 persona 打分——设计三个不同人格的 prompt，各自从不同视角对同一视频在 1–5 分制上评分：(1) 专家视角，评估动作幅度（movement amplitude）与身体动作频率；(2) 观众视角，评估身体语言反映出的情绪起伏与观众反应；(3) 标注专家视角，评估手势使用（gesture usage）与交互频率。剔除离群值后取平均分作为最终的运动幅度标注。这是用 MLLM 替代光流打分器的典型做法，且用多 persona 集成来降低单次打分的方差。
【作为过滤条件的使用】仅在 HQ / SFT 子集构造时启用：motion score > 2（5 分制）才入选，即排除近乎静止的呆板片段。全量数据集不施加运动下限。
【间接的静态/抖动控制】运动模糊过滤（人脸/手部 Laplacian 方差 < 0.1 剔除）会剔除因剧烈抖动或快速运动导致的糊帧；YOLO 跟踪的时空裁剪则保证主体在画面内稳定，间接抑制了强烈的镜头晃动。
【未使用】无光流（optical flow）计算、无 RAFT/UniMatch 类运动分数、无专门的静态镜头检测器。考虑到语料本身是访谈/对话类近景人物视频，运动幅度天然受限，传统运动过滤的必要性较低。

### 去重方法（精确去重与基于embedding的语义去重分别记录） ⚠️

论文与数据卡均未提及任何去重环节：既无基于视频哈希/指纹的精确去重，也无基于 embedding（CLIP/DINO 等）的语义去重，也未描述跨 clip 或跨源视频的近重复检测。
【客观风险】(1) 同一 YouTube 视频会被切分出多个 clip，同一说话人在同一场景下的多个片段高度相似，属结构性近重复；(2) 数据源覆盖 2006–2025 的 YouTube，同一访谈的转载、剪辑版、多平台重复上传在 YouTube 上极常见，采集时若仅按视频 ID 去重则无法识别内容级重复；(3) 83K 说话人 ID 对应 5.2M clips，平均每个说话人约 63 个 clip，头部名人（访谈/新闻类内容的典型主体）很可能贡献远高于平均的片段数，存在身份层面的长尾失衡。
【唯一相关机制】ArcFace 的 ID 校正步骤会跨 clip 计算人脸余弦相似度，但其目的是「保证同一 ID 的标注一致」而非「删除重复内容」，方向恰好相反——它是在做身份聚合而不是去重。
【结论】去重是本数据集披露体系中的明确空白。[不确定]

### VLM/LLM作为质检员（多模态大模型质量打分与错配剔除，2026年从浅层打分器转向大模型语义判断的趋势）

SpeakerVid-5M 处于「大模型深度参与标注、但质量裁决仍由专用模型承担」的中间形态，与 MOVA 的分工格局类似：
【MLLM 作为打分员的实际用法（唯一一处）】Qwen2.5-VL 被用作运动强度评分器，采用三 persona 集成打分（专家/观众/标注专家三种人格视角，各自 1–5 分制），剔除离群后取均值。这是把主观、难以用浅层指标刻画的「动作幅度与交互活跃度」交给 MLLM 判断的典型案例，且用多视角集成缓解单次打分的不稳定性——相比单 prompt 打分是更审慎的做法。该分数随后成为 HQ 子集的硬性筛选条件之一（> 2）。
【LLM 作为分类器】Qwen-3 被用于对话主题类目（dialogue topic category）的分类标注。
【MLLM 作为描述器】Qwen2.5-VL 生成结构化 caption（相机运动、实体列表、身体朝向、半身/全身、表情、动作描述等）。
【质量裁决仍由专用模型承担】DOVER（视频质量）、SyncNet（唇音同步）、ArcFace（人脸身份）、Whisper（ASR 置信度与无语音概率）、Laplacian 方差（模糊）、自定义 clarity 公式（清晰度）——这六类硬性过滤全部由专用判别模型或传统信号指标完成，没有任何一项由 VLM 出综合质量分替代。
【未采用的做法】没有 LLM-as-judge 式的跨模态一致性校验（对比 MOVA 用 GPT-OSS-120B 做视听语义自洽裁决）；没有 caption 幻觉自审计机制；没有用 MLLM 复核过滤结果。
【总体定位】MLLM 在此负责「标注与主观打分」，不负责「质量判决与错配剔除」，属该趋势的早期/保守形态。

### 安全与合规过滤（NSFW、版权、人脸/隐私） ⚠️

论文与数据卡均未描述任何安全与合规过滤环节：无 NSFW/暴力内容检测、无版权内容识别、无人脸模糊化或隐私脱敏、无未成年人内容处理、无有害言论（基于 ASR 转写文本的内容审核）过滤。
【替代性的风险处置】唯一的合规措施是事后的、被动的：限定非商业科研教育用途、声明版权归原作者、提供 takedown 下架政策、不托管原始视频而只发布 YouTube ID。这是「转移责任」而非「主动过滤」。
【隐私敏感度评估】该数据集的隐私风险实际高于一般视频数据集——它包含 83K 个唯一说话人身份、逐 clip 的 ArcFace 人脸特征关联、DWpose 全身骨架、以及完整的 Whisper 语音转写文本（即「谁在什么时候说了什么」的可检索结构）。这一组合足以构成可用于身份追踪的生物特征数据库，但论文未做任何隐私影响评估，也未说明是否获得出镜人物同意（YouTube 公开视频的默认答案是没有）。
【内容政治敏感性】数据源明确包含「新闻与政治（news and politics）」与「辩论（debates）」类目，ASR 转写会完整保留其中的政治言论，但未见任何内容审核说明。
【结论】安全与合规过滤是本数据集披露体系中最大的空白。[不确定]

## 打标方式

### 使用的caption模型（自研VLM/开源模型，模型规模） ⚠️

全部使用开源模型，未自研 captioner，形成「一个 MLLM 负责视觉描述 + 一个 LLM 负责话题分类 + 一个 ASR 模型负责转写」的三模型分工，且无融合环节：
【视觉结构化 caption】Qwen2.5-VL（阿里通义千问 2.5 视觉语言模型）。论文未明确说明使用的参数规模档位（Qwen2.5-VL 有 3B/7B/72B 等版本）。[不确定] HuggingFace 数据卡中该标注文件夹描述为「MLLM captions from qwen-vl」。
【运动强度打分】同样使用 Qwen2.5-VL，但以三 persona prompt 的方式独立调用，与 caption 生成是两条不同的调用链。
【话题分类】Qwen-3（阿里通义千问 3 系列 LLM），对对话主题做类目标注。
【语音转写】Whisper（OpenAI），输出转写文本、置信度分数、no-speech 概率与压缩比等诊断量。
【姿态标注】DWpose，输出人脸、手部、身体关键点。
【身份与同步】ArcFace（人脸识别）、3D-Speaker（声纹分离）、SyncNet（唇音同步）、YOLO（人体检测跟踪）、Deep3DFaceRecon 与 UniSpeech（用于 VidChatBench 评测侧的表情 FID 与音色相似度）。
【与 MOVA 的对比】MOVA 用 MiMo-VL-7B-RL + Qwen3-Omni-Instruct + Qwen3-Omni-Captioner + GPT-OSS-120B 四模型分工并融合；SpeakerVid-5M 的标注模型链更轻，且各路标注保持独立字段、不做 LLM 融合，成本更低但语义整合度也更低。

### caption密度与结构化程度（短/长/密集描述、结构化字段如镜头运动、风格标签） ⚠️

属于「结构化 JSON 字段」路线，而非长篇密集自然语言描述，是与 MOVA/LTX-2 等生成模型数据集的显著分野：
【最终形态】标注以 JSON 结构保存并随数据集一并发布（anno 文件夹），字段并列而非融合成段落。论文示例中出现的字段包括：
  - Clarity（清晰度）
  - Camera Motion（相机运动模式）
  - Motion Intensity Level（运动强度等级，1–5）
  - Entity List（画面实体列表）
  - Speech Presence（是否有语音）
  - Observed Actions（观察到的动作，详细身体动作描述）
  - Number of People（人数）
  - Upper-Body Only（是否仅上半身/半身 vs 全身）
  - Facing Direction（朝向：正面/侧面）
  - 面部表情概括（facial expression summary）
  - 对话主题类目（topic category，由 Qwen-3 标注）
【密度】以短字段值为主，仅 Observed Actions 一项为较长的自然语言描述。整体 caption 长度远短于 MOVA 的约 200 词密集融合段落。Figure 3 给出了 caption 词云与词频统计图。
【显式风格/运镜标签】有——Camera Motion 与 Upper-Body Only / Facing Direction 都是显式的离散化结构字段，这一点比纯自然语言 caption 更便于条件化控制与数据检索。
【设计取向】该 schema 明显是为「可检索、可按条件筛选子集」优化的（例如可直接筛出「全身 + 正面 + 高运动强度 + 有语音」的样本），而非为「直接作为扩散模型的文本条件」优化。下游模型若要用作生成条件，通常需自行改写为自然语言 prompt。
【空白】未给出 caption 的平均长度统计、未提供 caption 质量的人工评估或准确率数据。[不确定]

### 音视频联合caption结构（是否同时覆盖视觉+听觉轨道、是否分流为独立字段，如LTX-2全音景描述、Script-a-Video factorized streams、Foley-Omni三字段）

SpeakerVid-5M 采用的是「多轨道并列保留、不做融合」的 schema，是本次调研中融合度最低的一档：
【视觉轨】Qwen2.5-VL 生成的结构化 JSON caption（相机运动、实体、朝向、身体构图、动作、表情等），完全不描述听觉内容。
【语音轨】Whisper 的 ASR 转写文本 + 置信度，存于独立的 asr 文件夹。
【同步轨】SyncNet 的三项指标（offset 偏移、confidence 置信度、embedding distance 嵌入距离）作为独立数值标注保存。
【结构轨】DWpose 骨架关键点序列（代码开源，数据因体积未上传）。
【质量轨】l_score 文件夹的人脸/手部清晰度分、merge_anno 中的 clear/DOVER 分。
【关键缺失】没有非语音音频描述——不存在对音效、环境音、背景音乐、录音空间感、说话人音色/口音的任何文字描述。这与 MOVA 的 Qwen3-Omni-Captioner 非语音轨、LTX-2 的全音景描述、Foley-Omni 的三字段 schema 形成鲜明对比。SpeakerVid-5M 的音频侧标注仅有「说了什么（转写）」与「同步得多好（SyncNet 数值）」两类，没有「听起来是什么样」。
【也没有融合环节】不存在类似 MOVA 中 GPT-OSS-120B 的跨模态一致性校验与统一 caption 合成步骤。各轨道以文件夹形式并列发布，由使用方自行组合。
【评价】这套 schema 反映了 SpeakerVid-5M 的定位——它是一个「人体交互结构数据集」而非「音视频语义数据集」，其价值在于精确的说话人-画面-口型对应关系与丰富的结构化元数据，而非丰富的音视频联合语义描述。下游若需联合 caption，须自行补标。

### 对白转写与说话人属性标注（ASR转写+说话人身份/语言/口音/情绪） ⚠️

这是 SpeakerVid-5M 最强的维度，也是其区别于其他视频数据集的核心资产：
【转写】Whisper 执行 ASR，逐 clip 输出转写文本，并保留三项诊断量用于过滤：转写置信度（< −1.5 剔除，HQ 子集要求 > −1）、no-speech probability（> 0.8 剔除）、compression ratio（> 2.5 剔除，用于识别重复退化/幻觉输出）。pipeline 中还会检测「language mismatch（语言不匹配）」并据此过滤。
【说话人分离与身份标注】三重机制串联，构成本数据集的技术骨干：
  1) 3D-Speaker 做声纹 diarization，按说话时长/频次挑出两个主说话人（合计占总说话时长 80% 以上），其余说话人丢弃——这一「只要两个主角」的决策直接定义了数据集的 dyadic 属性。
  2) SyncNet confidence 做音画绑定，把音频侧的说话人 ID 关联到 YOLO 检出的视觉 bbox 上，回答「这段声音属于画面里哪个人」。
  3) ArcFace 做跨 clip 身份校正，计算人脸余弦相似度验证一致性，离群样本若与其他 ID 相似度更高则重新分配 ID。
  最终得到单人分支 83K 个、对话分支 16K 个唯一说话人 ID，且 ID 在整个数据集范围内跨 clip 可追溯。
【说话/倾听状态判定（数据集独有）】两套规则：
  - 共处同框（co-present）场景：若只有一人在说话，且两人的 SyncNet 分数差异大于预设阈值，则分数较低的一方被判定为「倾听中」。
  - 非共处（non-co-present）场景：若 ASR 结果有效、转写置信度高于阈值、但该人视频的 SyncNet 分数低于预设阈值，则判定其为倾听者。
  这套判定直接生成了 listening 分支，是把「谁在听」显式建模为数据标签的少见设计。
【多轮对话上下文】multi-turn 分支把前序轮次的 ASR 转写按时间窗口 [x−T, x] 聚合，形成多轮对话上下文标注；时间间隔小于 δt 的相邻 clip 判为连续对话可拼接。
【未标注的属性】语言/语种标签、口音、情绪类别、语速、音色描述——这些在 MOVA 的音频 caption 中被自然语言化覆盖的属性，在 SpeakerVid-5M 中均无对应字段（情绪信息仅通过视觉侧的 facial expression summary 间接体现）。[不确定]

### 几何与结构化标注（相机参数、深度、3D point tracks、动作标注、显式状态）

结构化标注是本数据集的强项，密度远高于多数生成类视频数据集：
【人体姿态】DWpose 输出人脸、手部、身体的完整关键点序列（skeletal sequences）。这是为下游手势生成、身体动作驱动任务准备的核心几何标注。注意：因数据体积过大，骨架序列本身未上传至 HuggingFace，仅开源了计算代码，使用方需自行运行。
【空间框】YOLO 人体检测与跟踪得到的逐帧时空 bounding box，存于 merge_anno，同时也是 clip 空间裁剪的依据。
【人脸身份特征】ArcFace 人脸特征与跨 clip 余弦相似度，用于说话人 ID 的一致性校验。
【画幅与朝向的离散化标注】caption 中的 Upper-Body Only（半身/全身）、Facing Direction（正面/侧面）、Number of People——这三项把身体构图显式离散化，是 Table 1 中相对 CelebV-HQ 等头部数据集的差异化标注项。
【相机运动】caption 中的 Camera Motion 字段以文字形式标注相机运动模式，但为自然语言/类目描述，非参数化的相机位姿（无外参矩阵、无轨迹坐标）。
【运动强度】1–5 分制的 Motion Intensity Level（Qwen2.5-VL 三 persona 集成打分）。
【模糊与质量分】人脸/手部的 Laplacian 方差模糊分、clarity 分、DOVER 分。
【音画同步的数值化】SyncNet 的 offset、confidence、embedding distance 三元组逐 clip 保存。
【未提供】相机内外参数、深度图、3D point tracks、光流场、3D 人体网格（SMPL 等）、显式物理状态标注。评测侧使用了 Deep3DFaceRecon 计算表情 FID，但那是 VidChatBench 的评测工具而非训练数据标注。

### 合成数据构造（受控扰动/编辑构造训练对，如InstructAV2AV） ⚠️

无。SpeakerVid-5M 全部由真实 YouTube 视频经切分、裁剪、过滤、标注得到，不含任何模型生成或受控扰动构造的合成样本，也没有构造「编辑前/编辑后」式的训练对（对比 InstructAV2AV 的受控扰动构造）。
【唯一的「构造」成分是数据重组而非数据合成】四分支的组织方式带有明显的构造性：dialogue 分支把同一时段两个说话人的独立 clip 配成 pair；listening 分支通过 SyncNet 分数差判定倾听者从而从原本的说话场景中派生出倾听样本；multi-turn 分支通过时间窗口聚合与 δt 间隔拼接构造多轮上下文。这些都是对真实素材的重新配对与串联（re-pairing / re-sequencing），不涉及像素或音频的生成、编辑或扰动。
【标注侧的合成成分】所有 caption、话题类目、运动强度分均由 MLLM/LLM 自动生成，属「合成标注」而非「合成内容」。
【数据增广】论文未描述任何训练时或数据构建时的增广策略（如时间抖动、颜色扰动、mixup 等）。[不确定]

### 人工介入程度（人工标注、人工质检、模型初筛+人工复核） ⚠️

人工介入程度低且集中在流程的最前端，后续环节完全自动化：
【采集端的人工筛选（唯一明确的人工环节）】论文明确表述为「manually collected high-quality videos featuring two-person dialogue from YouTube」——即由人工从 YouTube 上挑选高质量的双人对话视频作为源素材（153K 条 / 64,386 小时）。这一步决定了数据集的体裁构成与整体质量基线，是最关键的人工判断，但论文未说明筛选标准的具体条目、参与人数、以及人工筛选的通过率。
【阈值设定】五维质量过滤的阈值（亮度 10/210、DOVER 0.25、clarity 最低 5%、模糊分 0.1、ASR 置信度 −1.5 等）以及 HQ 子集阈值均为人工设定的经验值，但论文未像 MOVA 那样说明是否通过「人工抽检不同 cutoff 下的留存样本」来标定。[不确定]
【标注环节】完全自动化。caption、话题、运动强度、ASR、骨架、同步指标全部由模型产出，无人工标注、无人工复核、无「模型初筛 + 人工验收」的双阶段设计。
【质量评估】论文未报告任何针对自动标注结果的人工准确率评估（如 caption 幻觉率、diarization 错误率、说话/倾听判定准确率的人工抽检），这使得标注质量缺乏独立验证。多 persona 集成打分可视为对人工共识的一种模型化替代，但不等同于人工校验。
【评测环节】VidChatBench 采用全自动客观指标（FID、FVD、PSNR、SSIM、ArcFace 余弦距离、CLIP 距离、SyncNet 置信度、表情 FID、SIM-o），未见人类偏好评测（human preference / Arena）设计。

## 音视频对齐

### 音视频同步检测方法（口型同步、事件对齐）

音视频同步检测是 SpeakerVid-5M pipeline 的中枢，其作用超出了「质量过滤」的常规范畴，而是承担了三项结构性职能：
【职能一：音画身份绑定（最核心）】在 human detection 之后，用 SyncNet 计算每个候选人体框与音轨的唇音重合度，依据 confidence score 把音频侧的说话人 ID 绑定到视觉侧的 bounding box。这一步回答「这段声音是画面中哪个人发出的」，等价于 active speaker detection（主动说话人检测），是构建整个数据集的前提——没有它，diarization 得到的声纹 ID 无法与画面中的人对应。
【职能二：说话/倾听状态判定】
  - 共处同框场景：若仅一人在说话，且两人的 SyncNet 分数差异大于预设阈值，则较低分者被判为「倾听状态」。
  - 非共处场景：若 ASR 结果有效、转写置信度高于阈值、但该人的 SyncNet 分数低于预设阈值，则判为倾听者。
  由此派生出 listening 分支——这是把 SyncNet 从「过滤器」升格为「语义状态分类器」的用法。
【职能三：同步质量过滤与标注】SyncNet 的三项输出（offset 时间偏移、confidence 置信度、embedding distance 嵌入距离）作为逐 clip 的持久化标注保存于数据集中，供使用方自行按需筛选；Figure 3 给出了 SyncNet 置信度的分布直方图。
【事件级对齐】不涉及。数据集为纯语音场景，无音效-画面事件对齐（如脚步、碰撞）的检测需求，也未使用 Synchformer、ImageBind 等语义对齐工具。
【评测侧】VidChatBench 的六维指标之一即为音视频同步性，同样以 SyncNet confidence 度量。

### 同步检测具体指标与阈值（SyncNet/Synchformer/LSE/自研，阈值数值，如MOVA LSE-D≤9.5且LSE-C≥4.5、SkyReels-V4 SyncNet |offset|≤3∧conf>1.5） ⚠️

【使用的指标】SyncNet 系指标，逐 clip 记录三项：offset（音视频时间偏移帧数）、confidence（同步置信度）、embedding distance（音视频嵌入距离，即 LSE-D 的等价量）。数据集把这三项作为标注字段发布，而非仅用作内部过滤。
【阈值披露状况】论文在多处使用「pre-defined threshold（预设阈值）」的表述而未给出具体数值，这是本条目最遗憾的披露空白：
  - 说话/倾听判定中「两人 SyncNet 分数差异大于预设阈值」——差值阈值未公开。
  - 非共处倾听判定中「SyncNet 分数低于预设阈值」——该下限未公开。
  - HQ / SFT 子集的五条筛选条件（手部模糊 > 0.5、人脸模糊 > 0.7、DOVER > 0.6、运动分 > 2、ASR 置信度 > −1）中不含 SyncNet 项，说明 SyncNet 未被用作 HQ 子集的硬性门槛。
【与同类工作的对比】MOVA 明确给出 LSE-D ≤ 9.5 且 LSE-C ≥ 4.5 的双阈值并披露该过滤产出约 2.5M clips；SkyReels-V4 给出 SyncNet |offset| ≤ 3 且 conf > 1.5。SpeakerVid-5M 虽然对 SyncNet 的依赖程度最深（用它做身份绑定与状态分类，而不只是过滤），却是三者中唯一未公开具体阈值数值的，这在方法可复现性上是明显短板——不过其开源的 curation 代码库中可能包含实际阈值常量，可作为补充查证路径。
【自研指标】无，未提出新的同步度量。[不确定]

### 时序同步vs语义同步的分离处理（时间对齐与内容语义匹配作为两个独立过滤条件）

SpeakerVid-5M 只处理时序同步，不做语义同步，两者未被分离为独立过滤条件：
【时序同步】由 SyncNet 全面覆盖——offset 直接度量音画时间错位帧数，confidence 与 embedding distance 度量唇形与音素的帧级对应强度。这是数据集唯一的对齐维度，且贯穿身份绑定、状态判定、质量标注三个环节。
【语义同步（内容层面的音画语义匹配）】完全未涉及。没有使用 ImageBind、Synchformer 语义分支、CLAP 或任何跨模态语义相似度模型来判断「音频内容与画面内容在语义上是否匹配」。
【为何缺失是合理的】数据集是纯语音、纯人物近景场景，音频内容就是画面中人物的说话内容，语义匹配关系由 SyncNet 的唇音对应天然保证——只要口型对得上，语义就必然对得上。语义同步过滤的主要价值在于剔除「画面是海浪、音频是钢琴曲」这类通用音视频数据中的错配，而在 dyadic talking 场景中这类错配不存在。
【与 MOVA 的对比】MOVA 将时序对齐（SynchFormer）与语义对齐（ImageBind，IB-Score）显式分离为两个独立过滤条件，因其数据覆盖 foley、音乐、环境音等通用音视频内容；SpeakerVid-5M 因场景垂直而无此需要。这是数据集定位差异导致的方法论差异，而非疏漏。

### 音频质量过滤（SNR、静音检测与静音占比阈值、无音轨剔除、画外音源剔除、背景音乐分离） ⚠️

音频侧过滤全部依托 Whisper 的诊断输出，采用三条并列否决规则，满足任一即剔除：
【1. 转写置信度】Whisper confidence score < −1.5 的剔除。HQ / SFT 子集进一步收紧到 > −1。该指标间接反映音频的可懂度与信噪水平——嘈杂、混响重、远场录音会显著拉低置信度。
【2. 无语音概率】no-speech probability > 0.8 的剔除。这条同时承担两个职能：剔除静音片段与纯背景音片段，以及把数据集约束为纯语音语料（音效、音乐、环境音主导的片段会被此条淘汰）。
【3. 压缩比】compression ratio > 2.5 的剔除。这是 Whisper 内部用于检测转写退化的指标——当模型陷入重复循环输出（如「谢谢观看谢谢观看…」）时压缩比会异常升高。该条实质是在剔除 ASR 幻觉样本，属于对标注可靠性的防护，而非对音频信号质量的直接度量。
【4. 语言不匹配】pipeline 中还提到基于「detected language mismatches（检测到的语言不匹配）」的过滤，即 Whisper 检出的语种与预期不符时剔除。
【未做的处理】论文未描述任何显式的 SNR 信噪比估计与阈值、静音占比（silence ratio）统计与阈值、背景音乐分离（BGM separation / source separation）、语音增强或降噪、画外音/旁白源剔除、响度归一化。相比 MOVA 使用 Audiobox-aesthetics 的 PQ/CU/CE 三维美学分加静音占比与带宽阈值，SpeakerVid-5M 的音频质量把控明显更薄——它依赖 Whisper 一个模型的副产品指标完成全部音频筛选，是成本低但维度单一的方案。
【无音轨样本】原始采集口径即为 audio-visual 成对视频，且 no-speech 与置信度双重过滤保证保留样本均含有效语音，因此不存在无音轨或无语音样本。[不确定]

### 语音/音效/音乐的分类与分别处理策略

SpeakerVid-5M 不做音频类型的分类与分别处理——它通过过滤把非语音类型整体排除，而非分类后差异化对待：
【实际策略：单一类型收敛】no-speech probability > 0.8 即剔除的规则，配合 ASR 置信度下限，使得最终语料在音频类型上收敛为单一的「人声语音」。音效（foley）、音乐、环境音、静音片段没有被分类保留，而是被直接淘汰。因此数据集内不存在需要「分别处理」的音频类型分支。
【未使用的工具】没有音频类型分类器（对比 MOVA 使用 EAT 自监督音频 Transformer 构建 speech / non-speech 子集并按目标能力分流），没有音源分离，没有针对音乐或音效的专门标注或过滤路径。
【唯一的类型相关设计是角色维度而非声学维度】数据集的分支划分（talking / listening / dialogue / multi-turn）区分的是「说话人角色与交互形态」，全部四类的音轨都是语音，不构成声学类型的分流。listening 分支的画面主体不说话但音轨仍是对方语音——这是音画角色的错位，不是音频类型的差异。
【下游后果】使用该数据集训练的模型只能获得语音-唇同步能力，无法学习 foley、音乐或环境音生成。这与 MOVA 的实践一致：MOVA 将 SpeakerVid-5M 定位为唇同步能力来源，而通用音效与音乐能力另从 VGGSound、WavCaps、JamendoMaxCaps 等数据集的音频塔预训练阶段注入。这种「按数据集分工获取不同音频能力」的模式，正是 SpeakerVid-5M 作为垂直语料在生态中的定位。

## 训练配合

### 多阶段训练课程与数据课程调度（阶段划分依据：分辨率/时长/质量分/模态；低清→高清、图像→视频、短→长）

数据集本身以「四分支 + 双质量档（全量 / HQ）」的形式为多阶段课程提供了现成的调度结构，基线模型据此设计了三阶段训练：
【阶段一：预训练（单人 + 纯视频目标）】使用单人分支数据，以 ASR 转写文本与 caption 作为条件，生成目标仅为视频模态。此阶段建立文本/语音条件到人像视频的基础映射。
【阶段二：联合音视频训练】生成目标扩展到视频 + 音频双模态，使用经过滤的音频数据。此阶段建立跨模态联合生成能力。
【阶段三：微调 / SFT（双人对话 + 高质量）】使用高质量的双人对话音视频配对数据（HQ 子集），完成从单人生成到双人交互生成的能力迁移。
【课程调度依据】三个维度同时递进：模态（视频 → 视频+音频）、交互复杂度（单人 → 双人对话）、数据质量（全量 → HQ 子集）。与 MOVA 以「分辨率 + 质量」为主轴的课程不同，SpeakerVid-5M 基线的课程主轴是「模态与交互复杂度」——这正是其数据集四分支结构的直接映射。
【分辨率课程】无。基线全程使用固定的 480×768 分辨率，不做低清到高清的分辨率爬升。
【时长课程】无。clip 长度固定在 3–14 秒区间，不做短到长的时长爬升；多轮对话的长上下文由 multi-turn 分支的转写聚合提供，而非通过延长视频序列实现。
【算力】阶段一与阶段二合计在 128 张 NVIDIA L40S 上训练 15 天；阶段三微调在 32 张 NVIDIA A800 上训练 5 天。

### 数据配比随训练阶段的变化（预训练/退火annealing/SFT高质量子集） ⚠️

【规模层面的配比变化】呈典型的「大规模低门槛预训练 → 小规模高门槛后训练」金字塔：
  - 预训练阶段：7,375 小时（即总量 8,743 小时扣除 HQ 子集 1,368 小时后的剩余部分）。论文明确将这部分表述为 large-scale pretraining 数据。
  - 微调阶段：571K clips / 1,368 小时的高质量 SFT 子集，约占总时长的 15.6%。
  值得注意的是论文采取的是「HQ 子集与预训练集互斥」的划分方式（7,375 + 1,368 = 8,743），而非「HQ 子集是预训练集的子集」——即高质量数据被保留到后训练阶段专用，预训练不使用它们。这与多数工作「预训练用全量、SFT 用其中的精选子集」的做法不同，是一个较特别的选择。
【分支层面的配比变化】预训练用单人分支（single），微调用对话分支（dialogue）的高质量配对。listening 与 multi-turn 分支在基线训练中的具体使用比例未说明。[不确定]
【模态层面的配比变化】阶段一仅视频目标，阶段二起加入音频目标，即音频数据的参与是从零到有的阶跃式引入，而非渐进配比调整。
【未披露】各阶段内部不同数据源/分支/主题类目的采样权重、是否存在 annealing（退火）阶段、是否对长尾说话人做重采样均未说明。论文未提供 stage-wise 的数据配比表。

### 后训练数据（SFT精选集规模与筛选标准、偏好对数量与标注方式、reward model训练数据）

【SFT 精选集】571K clips / 1,368 小时，筛选标准为五条阈值同时满足：手部模糊分 > 0.5、人脸模糊分 > 0.7、DOVER 融合分 > 0.6、运动强度分 > 2（5 分制）、ASR 转写置信度 > −1。相比全量数据的过滤门槛（模糊分 > 0.1、DOVER > 0.25、ASR 置信度 > −1.5、无运动分要求），HQ 阈值在每一维上都显著收紧，尤其是模糊分从 0.1 提到 0.5/0.7（提升 5–7 倍）与新增的运动强度下限。这套阈值组合的意图很明确：要「人脸清晰、手部清晰、画质好、动作丰富、口齿清楚」的样本，直指手势与表情生成的质量瓶颈。
【使用方式】用于基线模型的第三阶段微调（32 张 A800 训练 5 天），数据为「premium dialogue audio-video pairs（优质双人对话音视频配对）」。
【偏好数据与奖励模型】无。论文未构建任何偏好对（preference pairs）、未训练 reward model、未使用 DPO/RLHF 类的偏好优化。基线的后训练仅为监督微调（SFT）一种形式。
【人类反馈】无。评测侧 VidChatBench 全部为自动客观指标，未包含人类偏好评测或 Arena 式对比，因此也不存在可复用为偏好数据的人类标注。
【对比】MOVA 同样无独立 SFT/RLHF 阶段而以三阶段渐进课程替代；SpeakerVid-5M 基线则有明确的 SFT 阶段但无偏好优化，处于「有 SFT、无 RL」的中间形态。

### 数据处理基础设施与吞吐（NeMo Curator/Data-Juicer/自研，GPU加速比，处理规模，成本） ⚠️

披露有限，但代码开源程度较高：
【处理框架】未使用 NeMo Curator、Data-Juicer 等成熟数据处理框架，也未提及 Ray、Spark 等分布式调度框架，为自研的脚本化 pipeline。GitHub Dorniwang/SpeakerVid-5M-Code 以六段独立脚本的形式组织：base annotation（音视频同步抽取 + 单人检测）、DWpose 骨架标注、ASR 标注、blur score 计算、luminance 计算、scene detection + speaker diarization（可选，部分结果已预计算）。这种分步脚本设计便于复用与断点续跑，但不含统一的调度层。
【依赖工具栈】3D-Speaker、YOLO、DWpose、Whisper、PySceneDetect、SyncNet、ArcFace、Deepface、UniSpeech、Deep3DFaceRecon、yt-dlp，均为开源组件。
【处理规模】输入 153K 条视频 / 64,386 小时，输出 5.2M+ clips / 8,743 小时，并对每个 clip 运行 YOLO 跟踪、DWpose 姿态、ArcFace 特征、SyncNet 同步、Whisper ASR、Qwen2.5-VL caption 与三次 persona 打分、Qwen-3 话题分类——按 clip 数计，MLLM 调用量在千万次量级（5.2M clips × 至少 4 次 VLM/LLM 调用），是相当可观的推理成本。
【未披露】数据处理所用的 GPU 型号与卡数、总处理耗时（wall-clock）、GPU-hours 或金钱成本、GPU 加速比、吞吐量（clips/hour）等全部缺失。论文仅披露了模型训练侧的算力（128×L40S 训练 15 天 + 32×A800 微调 5 天），未披露数据侧算力。
【骨架数据的体积问题】DWpose 骨架序列因体积过大未随数据集上传，仅提供计算代码——这从侧面说明该标注的数据量级相当可观（5.2M clips 的逐帧全身+手部+面部关键点）。[不确定]

## 效果对比

### 数据策略消融的量化影响（区分：过滤严格度ablation / caption密度风格ablation / 数据配比ablation，及对应评测指标） ⚠️

论文的消融实验集中在模型架构组件，未做数据策略层面的消融——这是本条目在「效果对比」维度的主要局限：
【已有的消融（Table 2，在 VidChatBench 上）】全部为架构组件消融：
  - 条件基线（conditioned baseline）：FID = 56.82，FVD = 55.06
  - 加入音频生成（配合 spatial transformer + noise injection）：FID = 38.53，FVD = 34.64
  - 双人（dyadic）设定但不含上述组件：FID = 49.97，FVD = 47.23
  - 完整双人配置（音频 + spatial transformer + noise injection）：FID = 32.35，FVD = 28.82
  结论指向两点：noise injection 训练策略显著抑制了自回归生成中的误差累积；spatial transformer 带来了视觉指标上的实质提升。从 56.82 到 32.35 的 FID 降幅约 43%。
【缺失的数据消融】以下三类数据策略消融均未开展：
  - 过滤严格度消融：未对比「使用全量 8,743 小时」vs「仅使用 HQ 571K clips」的效果差异，因此 HQ 阈值组合（模糊 0.5/0.7、DOVER 0.6、运动分 2、ASR −1）的收益缺乏量化验证。
  - caption 密度/风格消融：未对比结构化 JSON caption 与自然语言 caption、或不同 caption 字段组合对生成质量的影响。
  - 数据配比消融：未对比四分支（single/listening/dialogue/multi-turn）不同配比的效果，也未验证 listening 分支与 multi-turn 分支的实际增益。
【间接的规模证据】论文通过与 CelebV-HQ、HDTF、MultiTalk、OpenHumanVid 等既有数据集的对比（Table 1）论证其规模与标注优势，但这是数据集属性对比而非训练效果对比，不构成数据消融证据。[不确定]

### 质量vs数量的证据（小而精数据超越大而杂的案例） ⚠️

论文未提供「小而精超越大而杂」的直接量化证据，但其数据设计中隐含了对该理念的实践性采纳：
【隐含的实践】把 571K clips / 1,368 小时的 HQ 子集从 8,743 小时中单独切出，并专门用于第三阶段的对话微调，同时让预训练只使用剩余的 7,375 小时——这一「互斥划分」的设计本身就体现了「高质量数据应留在最接近最终能力的训练阶段使用」的判断。HQ 阈值在每个维度上都比全量门槛严格数倍（模糊分 0.1 → 0.5/0.7，DOVER 0.25 → 0.6，新增运动分下限），说明团队认为后训练阶段的数据质量比数量更关键。
【缺乏的验证】没有做「HQ 子集单独训练 vs 全量训练」的对照实验，因此无法回答「1,368 小时精选是否能超过 8,743 小时全量」这一问题。Table 2 的消融全部是架构组件消融，与数据量无关。
【端到端保留率的间接印证】从 64,386 小时原始素材筛到 8,743 小时（约 13.6%），再筛到 1,368 小时 HQ（约 2.1%），这个近五十分之一的收敛比例是同类数据集中相当激进的，反映出团队在质量取舍上的倾向性。但激进的过滤本身不等于「质量胜过数量」的证据。
【与同类的对比】MOVA 通过三阶段规模递减（61.5k → 37.6k → 11k 小时）配合各阶段指标提升，提供了课程层面的质量递增证据；SpeakerVid-5M 有相同形态的设计但未提供对应的量化验证。[不确定]

### 训练数据domain分布与评测基准类目体系的对齐关系（如VABench七大类） ⚠️

SpeakerVid-5M 是本次调研中训练数据类目体系与评测基准对齐度最高的样本之一——因为二者由同一团队同源构建，VidChatBench 直接从数据集中切出：
【构造同源性】VidChatBench 包含 500 组测试对（test pairs），使用未在训练中出现的说话人 ID（unseen speaker IDs），即测试集与训练集在身份维度上严格不重叠。它不是独立采集的外部基准，而是同一 pipeline 产出、同一标注体系下的留出集。
【六维指标体系与数据标注的一一对应】VidChatBench 的每一维指标都能在数据集的标注字段中找到对应的构建依据：
  1) 视频质量（FID、FVD、PSNR、SSIM）← 对应数据侧的 DOVER、clarity、luminance、blur 过滤。
  2) 身份保持（ArcFace 余弦距离）← 对应数据侧用 ArcFace 做 ID 校正的同一工具与同一特征空间。
  3) 对话连贯性（生成结果与 LLM 排序候选之间的 CLIP 距离）← 对应数据侧的 ASR 转写与 multi-turn 上下文聚合。
  4) 音视频同步（SyncNet confidence）← 对应数据侧用 SyncNet 做音画绑定与同步标注的同一指标。
  5) 情绪对齐（Deep3DFaceRecon 提取的表情 FID）← 对应数据侧 caption 中的面部表情概括字段。
  6) 说话人身份/音色（SIM-o 音色相似度，基于 UniSpeech）← 对应数据侧 3D-Speaker 的声纹 ID 体系。
【交互形态类目的对齐】数据集的四分支（talking / listening / dialogue / multi-turn）直接定义了下游任务的能力拆分，VidChatBench 的测试对以 dyadic 对话形态组织，与 dialogue 分支的组织方式一致。
【与 VABench 式外部基准的差异】VABench 等基准以七大内容类目（人物、动物、自然、乐器等）划分，考察的是内容覆盖广度；VidChatBench 则以「能力维度」而非「内容类目」划分，因为 SpeakerVid-5M 的内容 domain 高度单一（全部为真人对话），广度对齐无从谈起，转而在深度维度上做细粒度对齐。
【潜在风险】同源构造带来的高对齐度也意味着评测可能高估泛化能力——测试集与训练集共享同一套采集偏好、同一批清洗阈值、同一批标注模型的系统性偏差，跨数据集的外部有效性未被验证（论文未在 HDTF、CelebV-HQ 等外部基准上报告结果）。[不确定]

## 不确定字段

以下字段的调研信息部分不确定（⚠️ 标注来源）：

- openness
- provenance_licensing
- resolution_aspect_distribution
- domain_distribution
- audio_category_distribution
- narrative_structure_distribution
- language_accent_distribution
- funnel_retention_rate
- quality_filtering
- deduplication
- safety_filtering
- caption_model
- caption_structure
- dialogue_transcription_attributes
- synthetic_data_synthesis
- human_in_loop
- sync_metric_and_threshold
- audio_quality_filtering
- stage_data_mixture
- data_infra_throughput
- data_ablation
- quality_over_quantity_evidence
- benchmark_taxonomy_alignment
