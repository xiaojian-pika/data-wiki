# CogVideoX（含 CogVideoX-2B / 5B、CogVideoX1.5-5B / 5B-I2V，以及配套的 CogVLM2-Caption 打标模型；产品侧对应「清影 Ying」，音效侧对应 CogSound）

> 主题: 视频生成模型（含音视频同时生成）的数据处理：数据清洗流程、数据分布、打标方式

[← 返回首页](../index.md)

**目录**: [基本信息](#基本信息) · [数据规模与分布](#数据规模与分布) · [清洗流程](#清洗流程) · [打标方式](#打标方式) · [音视频对齐](#音视频对齐) · [训练配合](#训练配合) · [效果对比](#效果对比)

## 基本信息

### 名称

CogVideoX（含 CogVideoX-2B / 5B、CogVideoX1.5-5B / 5B-I2V，以及配套的 CogVLM2-Caption 打标模型；产品侧对应「清影 Ying」，音效侧对应 CogSound）

### 发布机构/公司

智谱 AI（Zhipu AI）与清华大学 THUDM 联合团队，论文通讯作者唐杰；核心贡献者 Zhuoyi Yang、Jiayan Teng、Wendi Zheng、Ming Ding、Shiyu Huang、Xiaotao Gu 等

### 发布时间（技术报告/论文/开源时间）

2024年8月6日开源 CogVideoX-2B 权重与代码；2024年8月12日 arXiv 预印本 2408.06072 v1（《CogVideoX: Text-to-Video Diffusion Models with An Expert Transformer》）；2024年8月27日开源 CogVideoX-5B；2024年9月开源 CogVideoX-5B-I2V；2024年11月8日发布「新清影」并开源 CogVideoX1.5-5B / 1.5-5B-I2V（同期发布音效模型 CogSound）；论文被 ICLR 2025 接收，v3 版 2025年3月26日更新

### 类型（模型/数据集/工具链/评测基准）

模型（文生视频/图生视频扩散 Transformer 基础模型系列）+ 配套工具链（3D 因果 VAE、CogVLM2-Caption 视频打标模型、caption upsampler 提示词改写、微调与推理代码库 zai-org/CogVideo）

### 开源程度（权重/代码/数据/pipeline各自是否开源）

开源程度在同期闭源大厂中属于较高的一档，但数据本身未开源：
· 权重：开源。CogVideoX-2B（Apache 2.0）、CogVideoX-5B / 5B-I2V、CogVideoX1.5-5B / 1.5-5B-I2V 均在 Hugging Face（THUDM/zai-org 组织）与 SAT 格式下公开发布，允许商用。
· 代码：开源。推理、微调（LoRA/SFT）、SAT 与 diffusers 两套实现均在 GitHub https://github.com/zai-org/CogVideo （原 THUDM/CogVideo）。
· 数据处理 pipeline：以论文形式公开描述（第3.4节 Data + 附录 G「Dense Video Caption Data Generation」+ 附录 K「Data Filtering Details」），公开了负面标签体系、6 个 Video-LLaMA 分类器及其测试集准确率表、caption 生成链路与 GPT-4 摘要 prompt 全文；但清洗代码与阈值配置文件未开源，数据集本身未开放。
· 打标模型：开源。CogVLM2-Caption（huggingface.co/zai-org/cogvlm2-llama3-caption）与 3D VAE 权重均已发布，是「数据 pipeline 公开」的最实质体现——外部可直接复现其打标环节。
· 训练数据：未开源，35M clips 与 2B 图像的具体来源、清单均未公布。
· CogSound（音效模型）：闭源，仅在清影产品与 API 中提供，无技术报告。

### 是否支持音视频同时生成，以及实现方式（原生联合/级联/MoE融合）

CogVideoX 模型本体不支持音视频同时生成，为纯视觉的文生视频/图生视频模型，训练数据不使用音轨。
产品层（「新清影」，2024年11月）通过级联（cascade）方式实现「自带音效」的视频：先由 CogVideoX（1.5 版）生成 10 秒 4K/60 帧无声视频，再由独立的音效模型 CogSound 以视频为条件做 V2A（video-to-audio）配音。CogSound 的公开技术要点为：基于 GLM-4V 的视频理解能力提取语义/情感 → 潜空间扩散模型（Latent Diffusion）生成音频 → 用「分块时序对齐交叉注意力（Block-wise Temporal Alignment Cross-attention）」建立帧级视频特征与音频特征的对应关系 → 叠加旋转位置编码（RoPE）提升长序列时序一致性。可生成爆炸、水流、乐器、动物叫声、交通工具等复杂音效与节奏元素。
因此实现方式为「级联」，而非原生联合或 MoE 融合；且两个模型的数据管线彼此独立，视频侧论文完全不涉及音频维度。

### 调研信息来源列表（论文/技术报告/官方文档/新闻的URL，标注每条来源的性质：官方一手/同团队旁证/第三方报道）

- 官方一手: CogVideoX: Text-to-Video Diffusion Models with An Expert Transformer (arXiv:2408.06072v3, ICLR 2025, 30页含附录G/J/K) https://arxiv.org/abs/2408.06072 与 https://arxiv.org/pdf/2408.06072
- 官方一手: zai-org/CogVideo 开源仓库（代码、权重、caption 工具链） https://github.com/zai-org/CogVideo
- 官方一手: CogVLM2-Caption 打标模型权重（CogVideoX 训练中使用的视频 caption 模型） https://huggingface.co/zai-org/cogvlm2-llama3-caption
- 同团队旁证: 智谱「新清影」CogVideoX+CogSound 技术详解（智源社区转载智谱官方技术解读，含 CogSound 分块时序对齐交叉注意力与数据筛选框架描述） https://hub.baai.ac.cn/view/40956
- 第三方报道: 智谱视频生成大模型清影升级（新清影 10s/4K/60帧/自带音效，2024-11-08） https://cn.technode.com/post/2024-11-08/zhipu-qingying-new/
- 第三方报道: CogSound 音效模型能力介绍 https://ai-bot.cn/cogsound/
- 第三方整理: CogVideoX 论文文献综述（Moonlight，含数据过滤与 35M clips 说明） https://www.themoonlight.io/en/review/cogvideox-text-to-video-diffusion-models-with-an-expert-transformer

## 数据规模与分布

### 训练数据量级（视频条数/小时数/token数，预训练与SFT分开） ⚠️

· 视频预训练：经过滤后剩余约 3500 万（35M）条单镜头 clip，平均每条约 6 秒（论文原文「approximately 35M single-shot clips remain, with each clip averaging about 6 seconds」），据此估算总时长约 5.8 万小时量级。
· 图像辅助数据：额外使用 20 亿（2B）张图像，取自 LAION-5B 与 COYO-700M 并以美学分过滤（超参表给出 Lowest aesthetic-value = 4.5）。训练时图像被当作单帧视频与视频混合训练。
· 高质量微调（第4阶段 FT）：从上述数据中选出质量更高的子集，占总数据量的 20%，训练 10k 步。
· 打标模型训练数据：为微调 GPT-4 摘要的替代模型（LLaMA2），收集了 5 万（50,000）条摘要数据点。
· 过滤器训练数据：人工标注 20,000 条视频的正/负样本标签，其中 10% 随机划为测试集。
· CogSound 的数据规模未披露 [不确定]。CogVideoX1.5 相对 1.0 的数据增量亦未披露 [不确定]。

### 数据来源构成（自有/公开数据集/网络爬取/授权采购/合成数据） ⚠️

论文仅笼统表述为来自互联网的视频（「Videos from the Internet usually include a significant amount of low-resolution ones」），未披露具体渠道、采购或授权方式，判断以网络爬取 + 内部数据池为主 [具体构成不确定]。可确认的部分：
· 图像侧明确使用两个公开数据集：LAION-5B 与 COYO-700M（经美学分过滤后取 2B）。
· 打标环节借用了公开视频 caption 数据集/模型：Panda-70M 的 caption 模型用于产生短 caption（论文同时批评 Panda70M、COCO Caption、WebVid 的 caption 过短、描述不全面，因此不直接使用其文本）。
· caption 训练数据部分为合成数据：由 CogVLM 逐帧图像 caption + GPT-4 摘要产生，属于「模型生成的合成文本」。
· 未提及授权采购数据、影视素材库或合成视频数据。

### 数据合规与溯源（授权数据占比、rights-cleared数据集、C2PA等） ⚠️

[不确定] 论文与开源仓库均未讨论训练数据的版权授权比例、rights-cleared 数据集、内容溯源标准（C2PA）或生成内容水印。仅有的相关信息是间接的：模型权重以 Apache 2.0（2B）与自有开源协议（5B 系列，允许商用）发布；清洗阶段刻意剔除「带明显字幕、水印」的视频（高质量微调阶段明确「effectively removed generated subtitles and watermarks」），但该动作的动机是画质而非版权合规。图像侧使用的 LAION-5B / COYO-700M 本身为公开的 URL-文本对数据集，其版权状态属公开争议范畴，论文未作说明。

### 片段时长分布与切分策略 ⚠️

· 训练 clip 平均约 6 秒，且全部为单镜头（single-shot）片段；未给出时长直方图 [分布细节不确定]。
· 明确采用「混合时长训练（mixed-duration training）」：论文批评固定帧数训练必须「丢弃短视频、截断长视频」，导致数据利用不充分，因此把不同时长（同时也是不同分辨率）的视频放进同一 batch，通过 Multi-Resolution Frame Pack（受 Patch'n Pack 启发）保证 batch 内形状一致，再用 3D RoPE 建模不同形状的位置关系。这相当于用 packing 取代传统分桶（论文明确指出 SDXL 式 bucket 方案「让数据与训练 pipeline 更复杂」）。
· 训练阶段的时长上限逐级放宽：stage1/stage2 最大 6 秒，stage3/stage4(FT) 最大 10 秒（对应序列长度 25k → 75k → 700k）。
· 3D VAE 侧为节省显存，采用两阶段训练：先在 17 帧视频上训练，再用 context parallel 在 161 帧视频上微调。
· 切分策略见 shot_segmentation：论文未描述显式的场景切分工具，而是用「Lack of Motion Connectivity（运动不连贯）」分类器把含拼接/转场的片段整体剔除。

### 分辨率/宽高比分布与分桶策略 ⚠️

· 分辨率课程（progressive training）：先在 256px 训练学习语义与低频知识，再逐级提升到 512px、768px 学习高频细节。训练超参表给出各阶段最大分辨率：stage1 256×384 → stage2 480×720 → stage3 768×1360 → stage4(FT) 768×1360。CogVideoX-5B 最终输出 720×480，CogVideoX1.5-5B 输出 1360×768、10 秒、16fps；产品版新清影可达 4K/60 帧。
· 宽高比：明确保持原始宽高比不变，只把短边 resize 到目标分辨率（「we keep the aspect ratio unchanged and resize the short side to above resolutions」），从而保留生成任意比例视频的能力；CogVideoX1.5-5B-I2V 支持任意分辨率输出。未给出横屏/竖屏配比数字 [不确定]。
· 分桶 vs packing：不使用 SDXL 式 bucketing，改用 Multi-Resolution Frame Pack 把不同分辨率与时长的样本打包进同一 batch。
· RoPE 适配高分辨率时对比了插值（interpolation）与外推（extrapolation）两种方案，最终选择外推以保留局部细节与相对位置关系。

### 类别/domain分布与配比策略（人物、动作、场景、风格等比例控制与概念均衡） ⚠️

[不确定] 论文完全没有披露训练数据的类别/domain 分布、概念聚类或配比均衡策略——这是 CogVideoX 数据工程相对 Movie Gen、Seedance 等最明显的缺失环节：没有概念聚类、没有长尾重采样、没有人物占比目标、没有动词/动作 taxonomy。
可间接推断的 domain 倾向来自「负面标签」的反向约束：
· 「Lecture Type（讲座/直播口播类，画面基本静止只有人在说话）」被整类剔除 → 训练集中「静态人物说话」题材被系统性压低，这也解释了模型在唇动/口播场景上的弱势。
· 「Text Dominated（大量可见文字或以文本内容为主）」被整类剔除 → 图文类、字幕类、演示文稿类内容被排除。
· 「Noisy Screenshots（手机/电脑屏幕直录）」被整类剔除 → 录屏、游戏画面、UI 演示类被排除。
· 「Editing（含明显人工剪辑与特效）」被剔除 → 高度剪辑的 MV、预告片风格被排除。
因此数据集实际偏向「未经重剪辑的、有真实连续运动的实拍单镜头素材」。
· caption 侧的 GPT-4 摘要 prompt 显式要求覆盖「objects, scenery, animals, characters, and camera movements」五类要素，可视为团队关注的内容维度，但并未据此做配比控制。
· 评测端在 VBench 上强调 Human Action、Scene、Dynamic Degree、Multiple Objects、Appearance Style 等维度，属评测选择而非数据配比。

### 音频类别分布与配比（语音/音效foley/音乐/环境音/静音的比例及控制策略）——AV模型独有维度 ⚠️

不适用于 CogVideoX 本体：视频模型训练完全不使用音轨，数据 pipeline 中没有任何音频维度，也没有语音/音效/音乐/环境音/静音的分类与配比。
级联的 CogSound 侧存在音频数据，但其训练数据构成、音频类别配比、静音处理策略均未公开 [不确定]。公开信息仅说明其生成目标覆盖「爆炸、水流、乐器、动物叫声、交通工具声」等复杂音效以及节奏/音乐元素，暗示训练数据以 foley 音效为主并含一定音乐成分，但无量化配比。也未见其是否生成或建模语音对白的说明。

### 叙事结构分布（单镜头vs多镜头、平均clip时长、镜头数分布、是否含原生音轨） ⚠️

· 全部训练片段为单镜头（single-shot）：论文明确「approximately 35M single-shot clips」，平均 6 秒。多镜头叙事不在建模范围内，模型输出为单镜头连续视频。
· 保证单镜头的机制不是场景切分器，而是负面标签分类器：「Lack of Motion Connectivity」专门识别「转场处运动不连贯、常见于人工拼接的视频或由静态图剪成的视频」，把含镜头切换/拼接的片段整体判负剔除；「Editing」标签进一步剔除带转场特效的素材。
· 镜头数分布、平均镜头数等统计未给出 [不确定]。
· 是否含原生音轨：不适用，视频侧训练完全丢弃音轨。
· caption 层面包含镜头运动（camera movements）描述，但不含多镜头结构化字段。

### 语言/口音分布（多语种唇同步能力的数据基础） ⚠️

不适用且未建模。CogVideoX 不生成语音，数据 pipeline 无 ASR、无说话人属性、无语言/口音标注，不具备唇同步能力（相反，「Lecture Type」标签把口播类视频整类剔除，使模型在人物说话场景上的数据基础被主动削弱）。
文本条件侧：caption 与 T5 文本编码器主要面向英文（论文与 caption 生成 prompt 均为英文，文本长度上限 226 token）；官方仓库建议使用英文 prompt，中文 prompt 需先经改写。CogSound 侧是否涉及多语种语音未披露 [不确定]。

## 清洗流程

### 清洗漏斗整体结构（几级过滤、各级顺序）

CogVideoX 的数据 pipeline 是「负面标签分类器 + 连续分数阈值 + 稠密重打标」三段式，结构清晰但层级较浅（相比 Movie Gen 的十余级漏斗要简单得多）：
【第一级：负面标签过滤】人工标注 20,000 条视频的正/负样本 → 训练 6 个基于 Video-LLaMA 的二分类过滤器 → 对全量原始视频做批量筛除。6 类负面标签为：Editing（人工重剪辑/特效破坏视觉完整性）、Lack of Motion Connectivity（转场处运动不连贯，人工拼接或静态图剪辑）、Low Quality（拍摄差、画面不清、相机抖动严重）、Lecture Type（讲座/直播口播，有效运动极少）、Text Dominated（文字主导）、Noisy Screenshots（手机/电脑屏幕直录）。设计动机被论文归纳为两类内在噪声（视频创作时的人工剪辑扭曲真实动态信息；拍摄设备与抖动导致的画质问题）加一类训练适用性问题（动态信息过少或动态缺乏连通性）。
【第二级：连续分数动态阈值】对全部训练视频计算光流分数（optical flow score，表征运动强度）与图像美学分数（aesthetic score，表征画质），并在训练过程中动态调整这两个阈值（dynamically adjust their threshold during training），以保证生成视频的动态性与美学质量。超参表给出美学分下限 4.5。这一「随训练阶段动态收紧阈值」的做法是 CogVideoX 数据工程较有特色的一点。
【第三级：稠密重打标（recaption）】对通过过滤的视频运行 Dense Video Caption Data Generation pipeline 产出长 caption（详见 caption_model / caption_structure）。
【第四级：高质量子集精调】预训练数据仍残留字幕、水印、低码率等脏数据，故在最后一个训练阶段挑出质量更高的 20% 子集做微调。
【图像分支】LAION-5B 与 COYO-700M 仅用美学分过滤后取 2B 图像，与视频混合训练（每张图像视为单帧视频）。
【推理侧对齐】用微调过的 LLM（图生视频用 GPT-4V / CogVLM）做 prompt upsampling，把用户短 prompt 改写成与训练 caption 同分布的长描述。

### 漏斗定量保留率（每级过滤的输入/输出量与最终保留率，如Apollo 27%） ⚠️

[不确定] 论文未给出任何逐级定量漏斗（各级过滤的输入/输出量与保留率）。仅有的两个可量化点：
· 过滤后最终剩余约 35M 单镜头 clip，但未披露原始视频池规模，因此无法反推整体保留率。
· 高质量微调阶段的保留比例明确为「占总数据量的 20%」（a subset of higher quality video data, accounting for 20% of the total dataset）——这是全文唯一的显式比例数字。
· 附录 K 给出的是 6 个分类器在测试集（随机 10% 标注数据）上的混淆矩阵，可间接看出各类负样本在标注池中的占比：Editing 类 TP+FN≈0.89（负样本占比极高）、Low Quality 类 TP+FN≈0.89、Static（运动不连贯）类 TP+FN≈0.52、Lecture 类 0.53、Text 类 0.62、Screenshot 类 0.62；但这是人工标注采样池的分布而非真实数据分布，不能等同于漏斗保留率。

### 镜头切分方法（PySceneDetect/自研模型/shot-aware splitting） ⚠️

论文未提及使用 PySceneDetect、FFmpeg 场景检测或任何显式的镜头边界检测/切分工具 [切分工具不确定]，这是其 pipeline 描述中的明显空白。
实际达成单镜头的机制是「判别式剔除」而非「切分」：训练一个专门的分类器识别 Lack of Motion Connectivity（论文正文与附录表 14 中该分类器被命名为 Classifier-Static，测试准确率 0.92），其定义为「转场处缺乏连贯运动的视频片段，常见于人工拼接的视频或由静态图片剪辑而成的视频」；再叠加 Editing 分类器（准确率 0.91）剔除含明显剪辑与特效的素材。两者共同保证进入训练集的 35M 片段是单镜头且运动连续的。
换言之，CogVideoX 选择了「宁可整条丢弃也不做精细切分」的策略，代价是数据利用率较低，收益是实现简单且不引入切分器的误差。

### 质量过滤（美学评分、清晰度、OCR文字过滤、黑边/水印/logo检测）

质量过滤由「Video-LLaMA 判别器 + 两个连续分数」两条线构成：
【判别器线】6 个 Video-LLaMA 二分类器中有 4 个直接服务于画质与画面内容纯净度，附录 K 表 14 给出测试集（随机 10% 标注数据）性能：
· Classifier-Low Quality（拍摄质量差、画面模糊、相机剧烈抖动）：TP 0.80 / FP 0.02 / TN 0.09 / FN 0.09，Test Acc 0.89（6 个中最低）
· Classifier-Editing（人工重剪辑与特效）：TP 0.81 / FP 0.02 / TN 0.09 / FN 0.08，Acc 0.91
· Classifier-Text（文字主导，等价于 OCR 文字过滤的判别式替代）：TP 0.60 / FP 0.03 / TN 0.36 / FN 0.02，Acc 0.96
· Classifier-Screenshot（手机/电脑录屏）：TP 0.61 / FP 0.01 / TN 0.37 / FN 0.01，Acc 0.98
· Classifier-Lecture（口播讲座）：Acc 0.99（6 个中最高）
· Classifier-Static（运动不连贯）：Acc 0.92
【连续分数线】对全量视频计算图像美学分数，超参表给出 Lowest aesthetic-value = 4.5 作为下限；阈值在训练中随阶段动态调整。图像分支的 2B 图片同样用美学分过滤。
【最终阶段】高质量微调用的 20% 子集针对性解决预训练数据残留的「字幕（subtitles）、水印（watermarks）、低码率（low-bitrate）」三类脏数据，论文报告该步「有效去除了生成结果中的字幕与水印，并小幅提升视觉质量」。
值得注意：论文未描述黑边检测、logo 检测、独立 OCR 模型等 Movie Gen 式的专用检测器——文字过滤是用 Video-LLaMA 分类器端到端完成的，属于「用多模态大模型代替传统浅层检测器」的早期实践。

### 运动过滤（光流/运动分数阈值、静态与抖动剔除） ⚠️

两个手段并行：
· 光流分数（optical flow score）：对全部训练视频计算，并在训练过程中动态调整阈值，目的是「确保生成视频的动态性（ensure the dynamic ... quality of generated videos）」。具体阈值数值未公开 [不确定]。
· Video-LLaMA 静态/连通性分类器（Classifier-Static，对应负面标签 Lack of Motion Connectivity，测试准确率 0.92）：剔除动态信息极少、或转场处运动不连贯（人工拼接、静态图剪辑）的片段。
· 「Lecture Type」分类器（Acc 0.99）在效果上也承担剔除近静止画面的作用。
· 抖动处理走的是画质线而非运动线：相机剧烈抖动被归入 Low Quality 负面标签（「excessive camera shake」）由 Classifier-Low Quality 剔除，而非像 Movie Gen 那样用「每秒镜头数」做抖动代理指标。
· 论文未使用 motion vector、VMAF motion score 等 FFmpeg 侧指标。

### 去重方法（精确去重与基于embedding的语义去重分别记录） ⚠️

[不确定] 论文完全未提及去重环节——既没有精确去重（哈希/指纹），也没有基于 embedding 的语义去重或近重复检测（如 SSCD copy-detection embedding），更没有语义聚类与长尾重采样。这是 CogVideoX 数据 pipeline 相对同期工作最显著的缺失之一。
唯一与「冗余」间接相关的是负面标签中的 Editing 与 Lack of Motion Connectivity（可剔除由同一素材重复拼接而成的视频），但其设计目的是保证运动真实性而非去重。

### VLM/LLM作为质检员（多模态大模型质量打分与错配剔除，2026年从浅层打分器转向大模型语义判断的趋势）

CogVideoX 是「用多模态大模型充当数据质检员」这一范式的早期代表之一，且在打标与质检两端都依赖大模型：
【质检端】核心质检器是 6 个基于 Video-LLaMA 微调的视频二分类器，而非传统的浅层打分器（美学 CNN、OCR 检测器、静态检测器等）。用 20,000 条人工正负标注训练，直接对整段视频做端到端语义判断，一次性覆盖了「剪辑痕迹、运动连贯性、拍摄质量、题材类型（讲座）、文字占比、录屏来源」六个原本需要多个专用检测器才能覆盖的维度。附录 K 报告的测试准确率区间为 0.89–0.99（Low Quality 0.89、Editing 0.91、Static 0.92、Text 0.96、Screenshot 0.98、Lecture 0.99）。这种「一个 VLM 打六种语义标签」的做法，比同期依赖 LAION 美学分 + OCR + 静态检测器组合的方案更接近 2026 年的主流形态。
【打标端】GPT-4 作为 teacher 生成摘要式 caption，CogVLM 作为逐帧图像 recaption 模型，最终蒸馏到 LLaMA2（摘要加速）与 CogVLM2-Caption（端到端视频打标）。
【局限】大模型判断仍以「二分类过滤」为主，未用于打连续质量分、未做图文错配（caption-video mismatch）的语义一致性校验、也未用于生成结果的自动评审；连续分数仍由传统的光流与美学模型提供。因此可定位为「从浅层打分器转向大模型语义判断」的过渡形态。

### 安全与合规过滤（NSFW、版权、人脸/隐私） ⚠️

[不确定] 论文与开源仓库均未描述训练数据层面的 NSFW、暴力、版权、人脸/隐私过滤流程与工具，也未披露生成端的安全模型或水印方案。相关的仅有间接线索：负面标签中的 Text Dominated 与 Noisy Screenshots 会顺带剔除部分含敏感文本或屏幕内容的素材；开源协议（Apache 2.0 / 智谱开源协议）中包含常规的合规使用条款；清影产品侧作为国内公开服务需符合《生成式人工智能服务管理暂行办法》并完成算法备案，但技术细节未公开。

## 打标方式

### 使用的caption模型（自研VLM/开源模型，模型规模）

分两条路线，构成一个「teacher 生成 → student 蒸馏」的打标体系：
【离线 teacher pipeline（Dense Video Caption Data Generation，图 7）】
1) Panda-70M 的视频 caption 模型 → 先生成短 caption；
2) CogVLM（即 CogView3 中使用的图像 recaption 模型）→ 每隔 2 秒抽 1 帧，为每帧生成稠密图像 caption；
3) GPT-4 → 把带时间戳的 image_captions 字典摘要成最终的视频 caption（附录 G 给出完整 prompt，要求按时间顺序描述内容与变化，覆盖物体、景物、动物、人物与镜头运动，并明令禁止使用「The video presents / depicts / showcases」「throughout the video」等套话与换行符）；
4) 为加速，收集 5 万条 GPT-4 摘要数据微调一个 LLaMA2 作为摘要模型替代 GPT-4，实现大规模生产。
【在线 student 模型】CogVLM2-Caption：以 CogVLM2-Video + Llama3 为底座，用上述 pipeline 产出的稠密 caption 数据微调而成的端到端视频理解/打标模型，规模约 12B 级（Llama3-8B 系底座 + 视觉侧），用于进一步加速全量 recaption。该模型已开源（huggingface.co/zai-org/cogvlm2-llama3-caption），是本工作数据 pipeline 中唯一可被外部直接复用的组件。论文还发现把 CogVLM2-Caption 与 CogVideoX 串联即可实现 video-to-video 生成（附录 I），侧面证明其 caption 几乎捕捉了原视频的全部细节。
【推理侧】另有 caption upsampler：用微调 LLM（文生视频）或 GPT-4V / CogVLM（图生视频）把用户短 prompt 改写成训练 caption 风格的长描述，论文明确说明微调 LLM 优于零样本/少样本。

### caption密度与结构化程度（短/长/密集描述、结构化字段如镜头运动、风格标签）

· 密度：明确追求稠密长 caption。论文批评 Panda70M、COCO Caption、WebVid 等现有数据集的 caption「通常非常短、无法全面描述视频」，因此自建 pipeline 产出段落级长描述。附录 H 的对照示例极具代表性：同一视频 Panda-70M 输出「A crab is walking on the beach with a light bulb on its back.」（一句话），CogVLM2-Caption 输出近 80 词的段落，包含主体外观（dark glossy shell、reddish-brown legs）、动作、光照变化（from a soft glow to a more pronounced illumination）、场景（sandy terrain、tranquil sea backdrop）、时间（at night）与氛围评价（serene yet whimsical atmosphere）。
· 结构化程度：属于「非结构化的稠密自然语言段落」，没有 Movie Gen 式的显式字段（无独立的相机运动标签前缀、无 FPS token、无风格标签枚举）。但通过 GPT-4 摘要 prompt 强制约束了内容覆盖面与写作风格，形成事实上的隐式 schema：必须覆盖 objects / scenery / animals / characters / camera movements 五类要素；必须按时间顺序（in chronological order）描述内容及其变化；禁止使用固定套话开头以避免风格坍缩。
· 长度约束：文本编码器（T5）侧的 Text Length 上限为 226 token，是 caption 长度的实际天花板。
· 训练-推理分布对齐：因用户实际输入远短于训练 caption，专门设计 caption upsampler 在推理时把短 prompt 改写为长描述，与 DALL·E 3 的做法一致。

### 音视频联合caption结构（是否同时覆盖视觉+听觉轨道、是否分流为独立字段，如LTX-2全音景描述、Script-a-Video factorized streams、Foley-Omni三字段） ⚠️

不适用。CogVideoX 的 caption 为纯视觉描述，完全不涉及听觉轨道，训练数据不含音轨，因此不存在音视频联合 caption schema，也没有视觉/听觉分流字段。
级联的 CogSound 侧是否使用文本 caption 作为条件、其音频 caption 结构如何，均未公开 [不确定]；从公开描述看 CogSound 主要以视频特征（GLM-4V 理解 + 帧级特征交叉注意力）为条件，文本条件的角色未见说明。

### 对白转写与说话人属性标注（ASR转写+说话人身份/语言/口音/情绪） ⚠️

无。CogVideoX 数据 pipeline 中没有 ASR 转写、没有说话人身份/语言/口音/情绪标注，模型也不生成语音。相反，团队主动把「Lecture Type（以人物持续说话为主、有效运动极少）」列为负面标签并以 0.99 准确率的分类器整类剔除，等于在数据层面就排除了对白密集的素材。CogSound 侧是否处理语音未披露 [不确定]。

### 几何与结构化标注（相机参数、深度、3D point tracks、动作标注、显式状态）

基本没有显式的几何或结构化标注：论文未使用相机内外参、深度图、3D point tracks、光流场（光流仅作为标量分数用于过滤，不作为标注保留）、分割掩码或动作类别标签。
唯一的结构化要素是隐式的语言标注：GPT-4 摘要 prompt 明确要求 caption 中包含「camera movements（镜头运动）」，因此镜头运动信息以自然语言形式嵌在 caption 里，而非作为独立的分类字段或标签体系（对比 Movie Gen 的 16 类相机运动分类器 + 前缀拼接）。
模型侧的「结构化」主要体现在位置编码而非数据标注：3D RoPE 显式建模文本-时间-空间三个维度的位置关系，Multi-Resolution Frame Pack 在 batch 内携带各样本的形状信息。

### 合成数据构造（受控扰动/编辑构造训练对，如InstructAV2AV） ⚠️

· 视觉数据：全部为真实采集视频，未构造任何合成视频、未做受控扰动或编辑构造训练对。
· 文本数据：caption 全部为合成文本，且是多级模型合成链——Panda-70M 短 caption + CogVLM 逐帧图像 caption → GPT-4 摘要（5 万条）→ 蒸馏进 LLaMA2 → 再蒸馏进 CogVLM2-Caption。这是本工作中唯一体系化的合成数据构造，属于「合成标注」而非「合成样本」。
· 推理侧合成：caption upsampler 用 LLM 合成长 prompt，使推理输入分布向训练 caption 分布靠拢，本质也是一种合成数据对齐手段。
· 图生视频（I2V）模型的训练对构造细节见附录 D，论文正文未描述是否使用合成首帧 [不确定]。

### 人工介入程度（人工标注、人工质检、模型初筛+人工复核） ⚠️

人工介入集中在「训练过滤器的种子标注」这一个点上，整体属于「少量人工标注 → 训练模型 → 全自动大规模筛选」的经典范式：
· 过滤器种子标注：人工采样并标注 20,000 条视频的正/负质量标签（覆盖 6 类负面标签），其中随机 10% 作为测试集用于报告分类器准确率。这是全流程唯一明确的人工标注环节。
· 负面标签体系本身由人工设计（论文列出 6 类定义并在图 16 给出每类的样例视频）。
· 打标环节无人工：caption 全部由模型链自动生成，未提及人工精修或复核（对比 Movie Gen 在 SFT 阶段由标注员逐条重写 caption）。
· 高质量微调子集（20%）的挑选依据未说明是自动分数还是人工筛选 [不确定]，从行文推测为自动阈值筛选。
· 评测端有重度人工介入：100 条精心设计的 prompt，人工评审按 0–1 打细项分、0–5 打总分，并规定「若未遵循指令则总分不得超过 2」；评测维度为感官质量（Sensory Quality）、指令遵循（Instruction Following）、物理模拟（Physics Simulation）、封面图质量等，各维度均给出三档（1 / 0.5 / 0）的详细评分细则（附录 J）。致谢中专门感谢了 data annotators。

## 音视频对齐

### 音视频同步检测方法（口型同步、事件对齐） ⚠️

CogVideoX 视频侧无音视频对齐环节（不使用音轨）。
CogSound 侧的对齐是建模层面的机制而非数据过滤层面的检测：
· 分块时序对齐交叉注意力（Block-wise Temporal Alignment Cross-attention）：通过学习帧级视频特征与音频特征之间的关系，把视频与音频的特征精确连接，确保音频与视频在时序与语义两个层面的一致性。
· 旋转位置编码（RoPE）：为音频序列每个位置提供唯一标识并捕捉相对关系，提升长时序任务下的时序一致性与稳定性。
· 基于 GLM-4V 的视频理解：先准确识别视频背后的语义与情感，再据此生成匹配的音频，属于语义层对齐。
数据侧是否做过同步性检测与过滤（口型同步、事件对齐）完全未披露 [不确定]。CogSound 明确定位为音效（sound effects）生成而非对白配音，未见任何口型同步（lip-sync）相关能力或指标。

### 同步检测具体指标与阈值（SyncNet/Synchformer/LSE/自研，阈值数值，如MOVA LSE-D≤9.5且LSE-C≥4.5、SkyReels-V4 SyncNet |offset|≤3∧conf>1.5） ⚠️

[不确定] 无任何公开的同步检测指标或阈值。CogVideoX 视频侧不适用；CogSound 无技术报告，未公开 SyncNet / AV-align / ImageBind score 等指标的使用情况，也无置信度阈值数值。可确认的是全流程未使用 SyncNet 类唇同步指标（模型不生成语音）。

### 时序同步vs语义同步的分离处理（时间对齐与内容语义匹配作为两个独立过滤条件） ⚠️

视频侧不适用。CogSound 的公开描述在概念上区分了两个层面——分块时序对齐交叉注意力负责「时序层面的一致性」，基于 GLM-4V 的语义/情感理解负责「语义层面的匹配」，官方表述为「确保音频与视频在时序和语义层面的一致性」，可视为对时序对齐与内容语义匹配的分离处理，但这体现在模型结构设计上，而非作为两个独立的数据过滤条件 [数据侧是否分离处理不确定]。

### 音频质量过滤（SNR、静音检测与静音占比阈值、无音轨剔除、画外音源剔除、背景音乐分离） ⚠️

[不确定] CogVideoX 视频侧不涉及音频，训练时直接丢弃音轨，无 SNR、静音检测、无音轨剔除、画外音剔除或 BGM 分离等处理。CogSound 侧的音频质量过滤策略（信噪比阈值、静音占比、人声分离等）未公开。

### 语音/音效/音乐的分类与分别处理策略 ⚠️

[不确定] CogVideoX 视频侧不适用。CogSound 从公开描述看是「单一模型生成多类音频」的形态——可生成爆炸、水流、乐器、动物叫声、交通工具声等复杂音效，并可生成节奏等音乐元素，未见按语音/音效/音乐分模型或分数据流处理的说明，也未见各类别的分类器、配比与差异化处理策略。

## 训练配合

### 多阶段训练课程与数据课程调度（阶段划分依据：分辨率/时长/质量分/模态；低清→高清、图像→视频、短→长）

训练课程以「分辨率与时长渐进 + 最终高质量微调」为主线，四个阶段的超参在论文表 5 中完整给出（CogVideoX-2B 与 5B 共用）：
· stage1：最大分辨率 256×384，最大时长 6s，batch size 2000，序列长度 25k，训练 400k 步 —— 学习语义与低频知识；
· stage2：480×720，6s，batch 1000，序列 75k，220k 步；
· stage3：768×1360，10s，batch 250，序列 700k，120k 步 —— 学习高频细节；
· stage4（FT）：768×1360，10s，batch 100，序列 700k，10k 步 —— 高质量微调。
可见随分辨率提升，batch size 从 2000 降到 100、序列长度从 25k 涨到 700k、步数从 400k 降到 10k，算力向低分辨率阶段大幅倾斜。
其他课程要素：
· 图像与视频混合训练贯穿全程（每张图像视为单帧视频），解决了固定帧数联合训练时模型按 token 数分裂成两种生成模式的问题；
· 混合时长训练 + Multi-Resolution Frame Pack，使不同时长/分辨率样本同 batch，避免丢弃短视频与截断长视频；
· 数据过滤阈值（光流分数、美学分数）在训练过程中动态调整，即数据课程与训练进程耦合；
· 3D VAE 自身两阶段课程：先 17 帧训练，再 context parallel 微调到 161 帧；
· 扩散设置采用 v-prediction + zero SNR，并提出 Explicit Uniform Sampling（把 1..T 划分为 n 个区间、每个 rank 在各自区间内均匀采样）稳定 loss 曲线、加速收敛；
· RoPE 适配高分辨率时选择外推而非插值。

### 数据配比随训练阶段的变化（预训练/退火annealing/SFT高质量子集） ⚠️

· 分辨率维度：随阶段推进，训练数据从 256px 逐级切换到 512px、768px（保持宽高比、只 resize 短边），低分辨率数据在早期阶段占绝对主导。
· 时长维度：stage1/2 限定 ≤6s，stage3/4 放宽到 ≤10s，长片段在后期阶段才进入。
· 质量维度：光流分数与美学分数的阈值在训练过程中动态收紧（dynamically adjust their threshold during training），形成「质量随阶段单调提升」的隐式数据课程。
· 退火/精调阶段：最后一个阶段（stage4 FT，10k 步）只用质量最高的 20% 子集，针对性去除字幕、水印、低码率数据。论文诚实报告了代价：该步「有效去除了生成结果中的字幕与水印，并小幅提升视觉质量，但同时观察到模型语义能力的轻微退化（a slight degradation in the model's semantic ability）」——这是关于「高质量小数据精调会牺牲语义覆盖」的一个罕见的负面证据。
· 图文混合比例：2B 图像与 35M 视频 clip 混合训练，具体 batch 级配比未披露 [不确定]。

### 后训练数据（SFT精选集规模与筛选标准、偏好对数量与标注方式、reward model训练数据） ⚠️

· 后训练形式仅为高质量 SFT：从预训练数据中挑出质量更高的 20% 子集，在 stage4 训练 10k 步（batch 100，768×1360，10s）。筛选标准针对「字幕、水印、低码率」三类脏数据，具体阈值与是否有人工参与未披露 [不确定]。
· 未使用偏好数据：论文全文没有偏好对（preference pairs）标注、没有 reward model、没有 RLHF/DPO/RLAIF。人工评测（100 条 prompt、0–5 分制、多维评分细则）仅用于评估而非训练。这与同期及之后的 Seedance、Kling 等重度依赖 RLHF 的路线形成鲜明对比。
· I2V 模型为在 T2V 基座上继续训练得到（附录 D），其配对数据构造细节未在正文披露 [不确定]。
· CogVideoX1.5 相对 1.0 的后训练数据增量未公开 [不确定]。

### 数据处理基础设施与吞吐（NeMo Curator/Data-Juicer/自研，GPU加速比，处理规模，成本） ⚠️

[不确定] 论文未披露数据处理侧的基础设施、框架（未提及 NeMo Curator、Data-Juicer 等）、GPU 加速比、单位吞吐与成本。可推断的工具栈：Video-LLaMA（6 个过滤分类器）、光流计算模块、图像美学打分模型、Panda-70M caption 模型、CogVLM、GPT-4 API、LLaMA2、CogVLM2-Video/Llama3——其中「用 LLaMA2 蒸馏 GPT-4 摘要」与「用 CogVLM2-Caption 端到端取代三级 pipeline」两次蒸馏，动机都明确写为「加速大规模生成（to accelerate this process / to further accelerate video recaptioning）」，属于数据处理吞吐优化的直接证据，但没有给出加速倍数与处理规模数字。
训练侧基础设施同样披露有限：论文给出各阶段 batch size、序列长度与步数，序列长度最高 700k 需要 context parallel；VAE 训练使用 context parallel 扩展到 161 帧；推理侧给出 H800 上的时间与显存（5B 480×720×6s 为 113s / 26GB，5B 768×1360×5s 为 500s / 76GB），但未公布训练集群规模与 GPU 小时数 [不确定]。

## 效果对比

### 数据策略消融的量化影响（区分：过滤严格度ablation / caption密度风格ablation / 数据配比ablation，及对应评测指标） ⚠️

论文的消融实验重心在模型架构与训练技巧上，数据侧的量化消融很少，这是其相对 Movie Gen 的明显短板：
【架构/训练消融（非数据）】3D RoPE vs 正弦绝对位置编码（RoPE loss 收敛显著更快）；Expert AdaLN vs MMDiT vs 无 Expert AdaLN（按 FVD、CLIP4Clip Score 与 loss，Expert AdaLN 更优）；3D 全注意力 vs 2D+1D 分离注意力；Explicit Uniform Sampling（loss 曲线明显更稳定、收敛更快）；RoPE 外推 vs 插值（外推保留局部细节，插值产生模糊大图）。
【与数据直接相关的定性结论】
· caption 密度 ablation：论文主张「创新的视频打标模型显著提升了生成质量与语义对齐（Our innovative video captioning model significantly improves generation quality and semantic alignment）」，并在附录 H 用 Panda-70M 短 caption 与 CogVLM2-Caption 长 caption 的对照展示密度差异，但未给出 A/B 净胜率或指标增量数字 [量化结果不确定]。
· 高质量微调 ablation：明确报告了双向效果——去除字幕水印、视觉质量小幅提升，但语义能力轻微退化。这是关于「过滤严格度」权衡的一个定性但极有价值的结论。
· 混合时长训练 / Frame Pack：论证动机为固定帧数训练导致模型按 token 数分裂成两种生成模式且泛化差，但未给出对比数字 [量化结果不确定]。
· 数据配比 ablation：未做 [不确定]。

### 质量vs数量的证据（小而精数据超越大而杂的案例） ⚠️

证据存在但不如同期工作充分，且论文自身的立场偏向「规模与质量并重」：
· 支持质量优先的证据：（1）用 6 个 Video-LLaMA 分类器 + 光流/美学双阈值做严格前置过滤，宁可整条丢弃也不做精细切分与修复，最终只保留 35M 单镜头 clip；（2）最后阶段仅用 20% 高质量子集做 10k 步微调（相比 stage1 的 400k 步不足 1/40），即可有效去除生成结果中的字幕与水印并提升视觉质量；（3）caption 质量被论文列为提升生成质量与语义对齐的关键因素之一，即「同样的视频，更好的文本标注带来更好的模型」。
· 反向证据（论文诚实记录）：高质量微调在提升视觉质量的同时导致「模型语义能力轻微退化」，说明过度收紧数据质量会牺牲语义覆盖面与概念多样性——这是「小而精」路线代价的一个直接观测，在同类论文中较为少见。
· 论文的总体立场偏 scaling：结论明确写「CogVideoX 具有可扩展性，随着模型参数量、数据量与训练量的增加，未来性能会更好（As the size of model parameters, data volume, and training volume increase, the performance will get better in the future）」，并未主张小数据路线。
· 未给出「小而精数据超越大而杂数据」的对照实验 [不确定]。

### 训练数据domain分布与评测基准类目体系的对齐关系（如VABench七大类）

对齐关系较弱且是被动的——CogVideoX 采用外部通用基准而非自建类目体系，训练数据侧也没有 domain 配比策略可供对齐：
· 自动评测使用 VBench（剔除与质量无关的 color、bitrate 等子指标）以及 Devil 提出的 Dynamic Quality、CLIP4Clip 等；论文选取的关注维度为 Human Action、Scene、Dynamic Degree、Multiple Objects、Appearance Style 等「与人类感知一致」的类目，并用雷达图呈现。
· 训练数据没有对应的类别配比控制（无概念聚类、无长尾重采样、无人物占比目标），因此不存在 Movie Gen 那样「训练 taxonomy ↔ benchmark 类目」的一一对应关系。
· 唯一可辨识的对齐是「负向」的：数据侧通过 Lecture Type、Text Dominated、Noisy Screenshots 三个负面标签主动排除了静态口播、文字主导与录屏题材，与评测侧强调 Dynamic Degree / Human Action 的取向方向一致——即数据过滤在动态性维度上与评测目标同向。
· 人工评测自建了四维评分体系（感官质量、指令遵循、物理模拟、封面图质量，100 条 prompt，0–5 分制，指令未遵循则总分封顶 2 分），并与 Kling 做对比，CogVideoX-5B 在人工评测中胜出；该体系与训练数据的类目分布无直接映射。
· 未采用 VABench 等音视频联合类目体系（模型无音频维度）。

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
- motion_filtering
- deduplication
- safety_filtering
- joint_av_caption_schema
- dialogue_transcription_attributes
- synthetic_data_synthesis
- human_in_loop
- av_sync_detection
- sync_metric_and_threshold
- temporal_vs_semantic_sync
- audio_quality_filtering
- audio_type_handling
- stage_data_mixture
- post_training_data
- data_infra_throughput
- data_ablation
- quality_over_quantity_evidence
