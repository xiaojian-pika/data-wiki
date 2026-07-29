# LongCat-Video（美团 LongCat 团队视频生成基础模型，技术报告 arXiv:2510.22200；同系列衍生 LongCat-Video-Avatar 与 LongCat-Video-Avatar 1.5，技术报告 arXiv:2605.26486）

> 主题: 视频生成模型（含音视频同时生成）的数据处理：数据清洗流程、数据分布、打标方式

[← 返回首页](../index.md)

**目录**: [基本信息](#基本信息) · [数据规模与分布](#数据规模与分布) · [清洗流程](#清洗流程) · [打标方式](#打标方式) · [音视频对齐](#音视频对齐) · [训练配合](#训练配合) · [效果对比](#效果对比)

## 基本信息

### 名称

LongCat-Video（美团 LongCat 团队视频生成基础模型，技术报告 arXiv:2510.22200；同系列衍生 LongCat-Video-Avatar 与 LongCat-Video-Avatar 1.5，技术报告 arXiv:2605.26486）

### 发布机构/公司

美团（Meituan）LongCat 团队。技术报告署名 Meituan LongCat Team，作者含 Xunliang Cai、Qilong Huang、Zhuoliang Kang、Hongyu Li、Shijun Liang、Liya Ma、Siyu Ren、Xiaoming Wei、Rixu Xie、Tong Zhang 等。

### 发布时间（技术报告/论文/开源时间）

LongCat-Video 基础模型：2025年10月25日 arXiv v1 提交（2510.22200），10月28日 v2；对外正式发布与开源约在 2025年10月27日。衍生版本：LongCat-Video-Avatar 于 2025年12月23日发布（美团技术团队博客同日发文）；LongCat-Video-Avatar 1.5 于 2026年5月发布（arXiv:2605.26486）。

### 类型（模型/数据集/工具链/评测基准）

模型（视频生成基础模型 + 配套开源推理代码/算子）。13.6B 参数的 Diffusion Transformer，单一模型统一支持 文生视频(T2V)、图生视频(I2V)、视频续写(Video-Continuation) 三类任务；并开源了 Block Sparse Attention 的前向/反向实现（可视为工具链组件）。非数据集、非评测基准（但报告内含自建内部评测集）。

### 开源程度（权重/代码/数据/pipeline各自是否开源）

权重与代码开源程度高，数据侧完全封闭。
【已开源】(1) 模型权重：Hugging Face meituan-longcat/LongCat-Video（13.6B），以及后续 LongCat-Video-Avatar（wav2vec2 音频编码器）、LongCat-Video-Avatar-1.5（Whisper-large-v3 音频编码器）；(2) 推理代码：GitHub meituan-longcat/LongCat-Video，含多种推理模式与 Streamlit 交互界面；(3) Block Sparse Attention 算子的 forward 与 backward 实现随基座模型一并开源；(4) 技术报告公开架构、训练阶段表与 RLHF 细节。
【许可】MIT License（权重与代码），可商用，是同规模视频生成模型中最宽松的许可之一。
【未开源】训练数据本身、数据处理 pipeline 代码、内部 captioner（微调版 LLaVA-Video）、内部美学/模糊/水印打分器、内部微调的 VideoAlign 奖励模型、以及全部数据规模统计数字。训练代码亦未开源（仅推理）。

### 是否支持音视频同时生成，以及实现方式（原生联合/级联/MoE融合）

基础版 LongCat-Video 不支持音视频同时生成——它是纯视觉生成模型（T2V/I2V/视频续写），输出无音轨。
同系列的 LongCat-Video-Avatar / Avatar 1.5 是「音频驱动视频生成」（Audio-Text-to-Video, AT2V 与 Audio-Text-Image-to-Video, ATI2V），属于音频作为条件输入、视频作为输出的单向驱动，而非音视频联合生成（不生成音频）。实现方式为在同一 13.6B DiT 基座上注入音频特征：Avatar 用 wav2vec2 音频编码器，Avatar 1.5 升级为 Whisper-Large 编码器以提升唇同步精度，支持单流与多流（多人）音频输入。因此按「原生联合/级联/MoE融合」分类，该系列均不属于任何一类，应归为「音频条件驱动的视频生成」。

### 调研信息来源列表（论文/技术报告/官方文档/新闻的URL，标注每条来源的性质：官方一手/同团队旁证/第三方报道）

1. arXiv:2510.22200 《LongCat-Video Technical Report》 https://arxiv.org/abs/2510.22200 与全文 https://arxiv.org/html/2510.22200v2 —— 官方一手（核心依据，数据处理章节、训练阶段表、RLHF 细节均出自此）。
2. GitHub 官方仓库 https://github.com/meituan-longcat/LongCat-Video —— 官方一手（开源范围、MIT 许可、模型变体与发布时间线）。
3. Hugging Face 模型卡 https://huggingface.co/meituan-longcat/LongCat-Video —— 官方一手（许可、能力描述、MOS 分数）。
4. arXiv:2605.26486 《LongCat-Video-Avatar 1.5 Technical Report》 https://arxiv.org/abs/2605.26486 —— 官方一手（同团队旁证，音视频对齐/音频过滤/情绪与静默数据 curation 细节主要来自此篇）。
5. 美团技术团队博客 https://tech.meituan.com/2025/12/23/longcat-video-avatar.html 与 https://tech.meituan.com/tags/longcat.html —— 官方一手（中文发布口径）。
6. 美团新闻稿《LongCat-Video 以长视频为锚点，构建世界模型技术底座》 https://www.meituan.com/news/NN251205166001020 —— 官方一手（定位与世界模型叙事）。
7. 36氪报道 https://www.36kr.com/p/3527169453464452 、知乎解读 https://zhuanlan.zhihu.com/p/1966806796392966062 —— 第三方报道（发布时间与推理加速倍数等佐证）。
8. HyperAI 论文页 https://hyper.ai/en/papers/2605.26486 —— 第三方索引。

## 数据规模与分布

### 训练数据量级（视频条数/小时数/token数，预训练与SFT分开） ⚠️

[不确定]。技术报告全文未披露任何训练数据量级——既没有视频条数、总时长（小时），也没有 token 数或图像张数，预训练与 SFT 的规模均未给出。可间接推算的只有训练迭代量与批次配置：预训练五阶段合计约 678k iterations（Stage1 T2I 256p 285k + Stage2 140k + Stage3 164k + Stage4 36k + Stage5 53k），SFT 阶段 7.5k iterations（480p+720p×93帧，lr 1e-5），RLHF（GRPO）阶段约 0.5k iterations（group size 4、每步 64 prompts）。Avatar 1.5 同样未披露数据总量（Stage1 256p×93帧 batch 64 共 130k iterations，Stage2 480p×93帧 batch 32 共 45k iterations）。评测集规模是唯一给出确切数字的数据：内部 T2V 1228 条（500 条人工评测 + 728 条自动评测，覆盖 48 个类别）、I2V 400 条（100 张首帧参考图 × 4 类 prompt）。

### 数据来源构成（自有/公开数据集/网络爬取/授权采购/合成数据） ⚠️

报告仅笼统表述为「We collect raw video data from a variety of sources」（从多种来源收集原始视频数据），未区分自有业务数据/公开数据集/网络爬取/授权采购的构成比例，也未点名任何具体数据集。可确认包含图像数据与视频数据两大类（Stage1 为纯 T2I 图像训练，Stage2 起图像与视频混合）。SFT 阶段额外「incorporated specialized datasets」以增强指令跟随能力，特别针对相机运动（camera motion）与视觉风格（visual style）两类专项数据集，来源未说明。
同系列 Avatar 1.5 披露了按功能划分的六类数据来源，可作为团队数据组织思路的旁证：(1) 近景人脸视频（用于面部建模与唇动）；(2) 访谈类视频（主体稳定、语音清晰）；(3) 表演类视频（提供镜头语言与姿态变化）；(4) 交互类视频（物体操持与手势）；(5) 音乐视频（歌唱与节奏性动作）；(6) 动画与风格化视频（用于非写实域泛化）。各来源占比未披露。[不确定：具体来源构成与配比]

### 数据合规与溯源（授权数据占比、rights-cleared数据集、C2PA等） ⚠️

[不确定]。技术报告完全未涉及数据合规与溯源议题：没有授权数据占比、没有 rights-cleared 数据集声明、没有 C2PA 或任何内容溯源/水印标识方案的说明，也没有版权许可来源的讨论。这与国内厂商技术报告的普遍做法一致（合规细节不写入论文）。需注意模型权重本身采用 MIT 许可开放商用，但这不构成对训练数据来源合法性的任何声明。

### 片段时长分布与切分策略 ⚠️

报告未给出片段时长的统计分布（无平均时长、无时长直方图）。切分策略明确：以场景检测将长视频切成「training-friendly clips while maintaining content consistency」（训练友好且内容一致的片段），工具为 PySceneDetect + 内部自训练的 TransNetV2 双路联合。切分后在元数据中记录 duration 字段并用于后续过滤，但过滤阈值未公布。训练时的目标时长在架构层面是固定的：全部五个预训练阶段与 SFT、RLHF 阶段均采用 93 帧（30fps 下约 3.1 秒）的视频长度；分钟级长视频不靠单次生成长片段，而靠 Video-Continuation（视频续写）任务链式外推实现——即以多个前序帧作为条件帧递归续写。Avatar 1.5 的在线过滤中「duration」是显式的逐级过滤条件之一。[不确定：时长分布数值与过滤阈值]

### 分辨率/宽高比分布与分桶策略 ⚠️

报告未给出原始数据的分辨率/宽高比分布统计，也未描述宽高比分桶（aspect-ratio bucketing）机制。元数据层面记录 resolution、frame rate、bitrate 三项并据此筛选。真正明确的是训练侧的分辨率课程：256p（Stage1、Stage2、Stage3）→ 480p（Stage4）→ 480p+720p 混合（Stage5、SFT、RLHF），即低清到高清的渐进式提分辨率，Stage5 起同一阶段内混合两档分辨率训练，使模型同时适配 480p 与 720p 输出。推理侧最终交付 720p / 30fps。此外模型采用「时空双轴 coarse-to-fine」生成策略：先生成低分辨率低帧率的粗视频再精化，这一策略要求训练数据能同时支撑多档分辨率。VAE 为 WAN2.1 VAE，时空压缩比 4×8×8，叠加 patchify 后整体为 4×16×16。[不确定：原始数据分辨率分布与分桶细节]

### 类别/domain分布与配比策略（人物、动作、场景、风格等比例控制与概念均衡） ⚠️

这是 LongCat-Video 数据工作中相对最有特色的一环，采用「caption 文本嵌入聚类 + LLM 语义命名」的无监督类目体系，而非人工预定义标签体系。具体做法：对全部视频 caption 做 text embedding，在嵌入空间做聚类，再用 LLM 对每个簇做类别归纳与命名，得到内容类型标签（报告举例：personal interactions 人际互动、artistic performances 艺术表演、natural landscapes 自然风光 等）。该类目体系的用途有三层：(1) 观测数据分布，识别过密/过稀类目；(2) 做「targeted data supplementation or rebalancing」——针对性补数据或重平衡配比；(3) 支撑「dynamic and precise allocation of data subsets tailored to specific requirements and objectives of different training phases」，即不同训练阶段动态、精确地调配数据子集。
更进一步，SFT 阶段把这套嵌入空间直接用作采样权重：样本被「selected inversely proportional to their density in the caption embedding space」（按其在 caption 嵌入空间中的密度的倒数采样），这是一种显式的概念均衡（concept balancing）机制——高密度的常见概念被降采样，长尾稀有概念被相对提升，从而让精选集的语义覆盖更均匀。
此外还有两类被单独强化的 domain：相机运动（camera motion）与视觉风格（visual style），SFT 阶段专门引入对应的专项数据集以增强这两个维度的指令跟随。
未披露的是：聚类簇数量、各类目的具体占比数字、重平衡后的目标配比。[不确定：各类目具体比例数值]

### 音频类别分布与配比（语音/音效foley/音乐/环境音/静音的比例及控制策略）——AV模型独有维度 ⚠️

基础版 LongCat-Video 无音频模态，该维度不适用。
同系列 LongCat-Video-Avatar 1.5 涉及音频，但其音频处理是「作为驱动条件」而非生成目标，因此没有语音/音效/音乐/环境音/静音的生成侧配比设计。可归纳出的音频相关数据分层为：(1) 语音类为绝对主体（近景人脸、访谈两类来源均以清晰语音为核心）；(2) 歌唱/音乐类单列（音乐视频来源，用于歌唱口型与节奏性动作）；(3) 静默（非说话）数据被专门构建为独立子集——用 Qwen3-Omni 初判 + Qwen3-VL 复判的双模型一致性机制，仅当两个模型都判定主体未在说话时才保留该片段，用于让模型学会「有音频但主体不说话」的情形；(4) 多人并发说话区间被显式剔除（excluding intervals with concurrent speaker activity）。各类占比均未披露。[不确定：音频类别的具体配比数值]

### 叙事结构分布（单镜头vs多镜头、平均clip时长、镜头数分布、是否含原生音轨）

训练数据经场景检测切分后为单镜头片段（single-shot clips）——PySceneDetect + TransNetV2 的作用正是消除镜头切换，保证每个训练片段内容连续一致，因此训练集不含多镜头叙事样本，也没有镜头数分布的概念。平均 clip 时长未披露，但训练时统一采样为 93 帧（30fps 约 3.1 秒）。原生音轨：基础版训练完全丢弃音轨（纯视觉）；Avatar 系列则依赖原生音轨（音视频成对且需通过唇同步校验）。
多镜头/长叙事能力不在数据层解决，而在推理层通过 Video-Continuation 递归续写实现：以多个前序帧为条件帧连续生成，配合针对跨帧时序一致性与物理运动合理性的优化，抑制色彩漂移、画质衰减与运动断裂，从而稳定输出分钟级长视频。Avatar 1.5 在 GRPO 阶段支持最多 5 个 clip 的多段 rollout（仅最后一个 clip 参与优化），侧面反映其长视频是多段拼接范式。

### 语言/口音分布（多语种唇同步能力的数据基础） ⚠️

[不确定]。基础版无语音模态，但文本侧明确支持中英双语：文本编码器采用 umT5（多语种编码器，报告明确说明同时支持英文与中文 caption），且 caption 增强环节包含中英互译（translating captions between Chinese and English），因此训练 prompt 分布覆盖中英两种语言，具体比例未披露。
Avatar 系列涉及真实语音驱动，但技术报告未披露任何语种、口音的分布统计，也未说明多语种唇同步的数据构成；未指定 ASR 转写模型。

## 清洗流程

### 清洗漏斗整体结构（几级过滤、各级顺序）

整体为「预处理 + 标注」两大阶段的漏斗结构，各阶段内部再分若干级：
【第一阶段 预处理 Preprocessing】
1) 收集与去重：从多来源收集原始视频，用 source video ID 与 MD5 哈希做去重；
2) 场景切分：PySceneDetect 与内部自训练的 TransNetV2 联合做场景检测，将长视频切为内容一致的训练友好片段；
3) 黑边裁剪：在转场切分过程中用 FFMPEG 裁掉黑边；
4) 压缩打包：将处理后的片段压缩打包，以支撑训练时的高效数据加载。
【第二阶段 标注 Annotation】
5) 基础元数据标注：duration、resolution、frame rate、bitrate、aesthetic score、blur score、text coverage、watermark detection；
6) 运动信息标注：提取光流评估视频动态性；
7) Caption 标注：微调 LLaVA-Video 生成基础描述（+ Tarsier2 标注补强时序理解）；
8) 电影语言与风格标注：相机运动分类器 + Qwen2.5VL 标注景别/镜头类型/写实度/动画风格/色调；
9) Caption 增强：中英互译、生成精简摘要、将电影语言与风格标签随机组合拼接进增强 caption；
10) 分布分析与调配：caption 文本嵌入聚类 + LLM 类目归纳，据此做数据补充/重平衡，并为不同训练阶段动态分配数据子集。
值得注意的是，报告把这些标注呈现为「打标 + 按阈值筛选」的元数据驱动模式——所有质量信号先全量标注入库，再在不同训练阶段按需组合阈值抽取子集，而非一次性硬删除。这种「标注入库、按需过滤」的设计正是支撑上述「不同阶段动态调配数据子集」的基础。
Avatar 1.5 则升级为「离线标注 + 在线 clip 级校验」两层结构：离线做人脸检测与关键点、音频可用性标注、唇同步校验、视觉质量估计、相机与运动分类、分时段 caption；在线训练取数时再按 音频同步 → 相机适用性 → 文本与视觉质量 → 时长 → 视觉缺陷 → 运动一致性 → mask 面积约束 的顺序做逐级渐进过滤。

### 漏斗定量保留率（每级过滤的输入/输出量与最终保留率，如Apollo 27%） ⚠️

[不确定]。报告完全未给出任何一级过滤的输入/输出量或保留率，也没有最终保留率的整体数字。这是该技术报告在数据侧披露最薄弱的部分之一——它详细列举了用了哪些过滤器，但没有给出任何一个阈值数值与任何一级的通过率，因此无法与 Apollo 27% 之类的定量漏斗做横向对比。

### 镜头切分方法（PySceneDetect/自研模型/shot-aware splitting）

采用双路联合的场景切分方案：开源工具 PySceneDetect 与内部自训练（in-house trained）的 TransNetV2 模型并用。TransNetV2 是专门的镜头边界检测神经网络，对渐变转场（fade/dissolve）的识别显著优于 PySceneDetect 的阈值法，团队还在自有数据上做了再训练。切分目标是得到「training-friendly clips while maintaining content consistency」——既适合训练又保持内容一致的片段。切分过程中同步用 FFMPEG 做黑边裁剪（black border cropping during transition segmentation）。

### 质量过滤（美学评分、清晰度、OCR文字过滤、黑边/水印/logo检测） ⚠️

质量信号以元数据形式全量标注，再按阈值筛选，覆盖面较完整：
(1) 美学评分 aesthetic score —— 打分器型号未披露；
(2) 模糊/清晰度 blur score —— 判定画面清晰度与压缩损伤；
(3) 文字覆盖率 text coverage —— 检测画面中的文字/字幕占比，用于剔除字幕过多、贴片文字过重的样本（对应常见的 OCR 过滤）；
(4) 水印检测 watermark detection —— 剔除带水印/台标/logo 的片段；
(5) 黑边处理 —— 不是过滤而是修复，在切分时直接用 FFMPEG 裁掉；
(6) 编码质量 —— resolution、frame rate、bitrate 三项元数据可用于剔除低分辨率、低码率的劣质源。
SFT 阶段的精选集在此基础上再按「aesthetic score、video quality、motion quality」多指标复筛。
Avatar 1.5 的视觉质量估计明确针对三类问题内容：「low-resolution, heavily compressed, subtitle-heavy」（低分辨率、重度压缩、字幕过多），在线阶段还追加分辨率、亮度分布、黑/白像素占比检查以及跳帧（frame jump）检测。
所有具体阈值数值均未披露。[不确定：各质量指标的阈值]

### 运动过滤（光流/运动分数阈值、静态与抖动剔除） ⚠️

以光流（optical flow）为唯一运动度量：「Motion information is evaluated using extracted video optical flow to assess video dynamics, enabling us to filter out clips with minimal motion features」——提取视频光流评估动态性，据此过滤掉运动特征极少的片段（即剔除近似静止的样本）。报告只提到剔除「低运动」一侧，未提及是否同时剔除高频抖动/剧烈晃动的样本，也未给出光流分数阈值。SFT 阶段另有独立的 motion quality 指标参与精选筛选（该指标由内部微调的 VideoAlign 模型给出，与光流统计量不同，属模型打分）。Avatar 1.5 在线过滤中包含「motion intensity」与「motion consistency」两项检查，以及跳帧检测。[不确定：光流阈值数值、是否过滤抖动]

### 去重方法（精确去重与基于embedding的语义去重分别记录） ⚠️

仅在预处理首步做，方法明确但较为基础：使用 source video ID（来源视频标识）与 MD5 哈希双重去重。前者按来源侧的唯一 ID 剔除同一视频的重复采集，后者按文件字节级哈希做精确去重。
报告未提及任何基于 embedding 的语义/感知去重（如 CLIP/视频特征近重复检测、pHash 感知哈希），因此对「同一素材的不同编码版本、不同裁切、转载搬运」这类近重复缺乏显式处理手段。不过 caption 嵌入空间聚类与「按密度倒数采样」在 SFT 阶段起到了近似语义去重的效果——高密度冗余簇被自动降采样。[不确定：是否另有未披露的语义去重]

### VLM/LLM作为质检员（多模态大模型质量打分与错配剔除，2026年从浅层打分器转向大模型语义判断的趋势）

该模型大量使用多模态大模型作为标注员与质检员，是其数据流程中的核心手段，但更偏「模型打标」而非「模型判优剔除」：
(1) Caption 生成：微调版 LLaVA-Video（用内部合成 pair 微调）为主 captioner，并引入 Tarsier2 的标注以增强时序理解；
(2) Qwen2.5VL 承担两类结构化判别——景别（shot size）与镜头类型（lens type）的判定，以及视觉风格判定（写实度 realism、动画风格 animation styles、色调 color tones）；
(3) 相机运动（pan/tilt/zoom）用专门训练的分类器而非 VLM；
(4) caption 嵌入聚类后由 LLM 做类目归纳与命名，即用 LLM 定义数据类目体系；
(5) 后训练侧的奖励模型本质也是模型质检员：VideoAlign 基座微调出的 Motion Quality 模型（输入灰度视频，强制其只看运动不看色彩）与 Text-Alignment 模型（输入彩色视频），以及 HPSv3-general/HPSv3-percentile 视觉质量打分。
Avatar 1.5 进一步把 VLM 用作硬过滤器与一致性裁判：静默数据用 Qwen3-Omni 初判 + Qwen3-VL 复判，两模型一致才保留；情绪数据用 EmotiEffLib 以置信度 s>0.7 为阈值精筛，并设硬性排除规则（含合成内容、超过两个主体、身份切换、主体在画面中占比过小者一律标为 null）；caption 侧用 Qwen3-Omni 生成「空间环境 / 人物关系 / 情节推进」三维上下文描述，并要求描述聚焦客观物理表现。
报告未说明基础版是否用 VLM 做过 caption-视频错配（mismatch）的显式剔除。

### 安全与合规过滤（NSFW、版权、人脸/隐私） ⚠️

[不确定]。技术报告未提及任何 NSFW 过滤、暴力/敏感内容过滤、版权过滤或人脸隐私保护措施。考虑到模型面向公开开源且由国内大厂发布，实际生产中几乎必然存在内容安全审核环节，但报告中无任何文字依据，无法确认其方法与强度。间接相关的仅有 Avatar 1.5 中把「synthetic content（合成内容）」列为情绪数据的硬性排除项，但那是出于标注可靠性而非安全考虑。

## 打标方式

### 使用的caption模型（自研VLM/开源模型，模型规模） ⚠️

采用「主 captioner + 蒸馏增强 + 多个专项判别模型」的组合，而非单一模型：
(1) 主 captioner：在内部合成 pair 数据上微调的 LLaVA-Video（fine-tuned LLaVA-Video model using in-house synthetic pairs），负责生成基础 caption，同时覆盖视觉与时序两个方面；
(2) 时序增强：引入 Tarsier2 的标注参与训练/补强，Tarsier2 是以细粒度时序事件描述见长的视频描述模型，用于弥补静态描述对动作演进刻画不足的问题；
(3) 结构化属性判别：Qwen2.5VL 负责景别、镜头类型、写实度、动画风格、色调；相机运动（pan/tilt/zoom）由单独训练的轻量分类器负责（而非 VLM，推测出于成本与精度考虑）；
(4) 类目归纳：LLM（型号未披露）对 caption 嵌入聚类结果做类别命名。
各模型的参数规模均未披露（LLaVA-Video 常见为 7B/72B 档，Qwen2.5VL 常见 7B/72B 档，报告未指明选用哪档）。
Avatar 1.5 的 captioner 换为 Qwen3-Omni（全模态，可同时看音频与视频）。[不确定：各 caption 模型的具体参数规模]

### caption密度与结构化程度（短/长/密集描述、结构化字段如镜头运动、风格标签） ⚠️

结构化程度较高，采用「基础描述 + 属性标签 + 随机组装」的合成式 caption 构造范式：
(1) 基础层：LLaVA-Video/Tarsier2 产出的自然语言描述，兼顾视觉内容与时序动作；
(2) 结构化标签层，分两组——
   · 电影语言（cinematography）：相机运动（pan/tilt/zoom）、景别（shot size）、镜头类型（lens type）；
   · 视觉风格（visual style）：写实度、动画风格、色调；
(3) 增强层：中英互译（生成双语 caption，配合 umT5 双语文本编码器）、生成精简摘要（concise summaries，形成长短两档 caption，使模型同时适配简短 prompt 与详尽 prompt）；
(4) 组装层：「randomly selecting elements from cinematography and visual style categories and integrating them with augmented captions」——从电影语言与视觉风格类目中随机抽取若干元素，拼接进增强 caption。这种随机组装是关键设计：它让模型在训练中见到「有时提及镜头运动、有时不提」的多样 prompt 形态，从而在推理时既能响应显式的镜头/风格指令，也不会在用户不提时强行施加某种镜头语言，是 SFT 阶段相机运动与视觉风格指令跟随能力的数据基础。
未披露 caption 的平均长度、token 数分布以及长短 caption 的混用比例。[不确定：caption 长度分布与长短配比]

### 音视频联合caption结构（是否同时覆盖视觉+听觉轨道、是否分流为独立字段，如LTX-2全音景描述、Script-a-Video factorized streams、Foley-Omni三字段）

基础版无音频模态，不适用。
Avatar 1.5 的 caption 方案虽用了全模态模型 Qwen3-Omni，但其产出是三个视觉/叙事维度的字段——「Spatial Environment（空间环境）、Interpersonal Relationships（人物关系）、Plot Progression（情节推进）」，并要求描述聚焦客观的物理表现（physical manifestation）而非主观推断。这是为音频驱动人物视频服务的叙事上下文 schema，并非覆盖听觉轨道的音景描述，与 LTX-2 的全音景描述、Foley-Omni 三字段等真正的音视频联合 caption 范式不同。此外 Avatar 1.5 离线阶段还生成「temporal-span caption」（分时段/时间跨度 caption），用于描述局部片段，属于时间维度的细粒度描述而非模态维度的联合描述。

### 对白转写与说话人属性标注（ASR转写+说话人身份/语言/口音/情绪） ⚠️

基础版不适用。
Avatar 1.5 涉及说话人处理，但未采用 ASR 转写路线——报告未指定任何转写模型，也无逐字文本标注。其说话人相关处理集中在「谁在说话」而非「说了什么」：使用 TalkNet 与 UniTalk 两个主动说话人检测（Active Speaker Detection）模型；多人场景用 YOLOv6 检测 + ByteTrack 做人物跟踪以关联身份，并显式剔除多人同时说话的时间区间（excluding intervals with concurrent speaker activity），避免音画归属歧义。说话人属性标注方面，做了情绪维度：定义 6 类情绪，用 EmotiEffLib 以置信度 s>0.7 筛选，并对含合成内容、主体多于两人、身份切换、主体占画面比例过小的样本一律标 null。语言/口音属性未做标注。[不确定：是否有未披露的 ASR 环节]

### 几何与结构化标注（相机参数、深度、3D point tracks、动作标注、显式状态） ⚠️

结构化标注集中在「镜头语言」层面而非几何层面：标注了相机运动类别（pan/tilt/zoom，由专用分类器判定）、景别与镜头类型（Qwen2.5VL），但这些是离散语义标签，不是连续的相机外参/内参（无相机位姿轨迹、无 camera pose 数值）。运动方面标注了光流统计量，属稠密二维运动场的聚合量。
报告未涉及深度图、3D point tracks、点云、动作骨架（pose/keypoint，基础版）等几何标注。
Avatar 1.5 因人物驱动需要，做了人脸检测与人脸关键点（landmark）提取以验证面部可见性，以及基于 ByteTrack 的人物跟踪框，并在训练取数时施加 mask-area 约束（对人物 mask 面积占比的限制），这些属于二维结构化标注，仍非 3D 几何。[不确定：是否有未披露的深度/位姿标注]

### 合成数据构造（受控扰动/编辑构造训练对，如InstructAV2AV） ⚠️

有限使用。明确的一处是主 captioner 的训练：LLaVA-Video 是「using in-house synthetic pairs」（用内部合成的图文/视频-文本 pair）微调的，即合成数据用于训练标注器本身，而非直接作为生成模型的训练样本。
视频生成侧未见受控扰动/编辑构造训练对的做法（如 InstructAV2AV 式的成对编辑数据构造）。Video-Continuation 任务的训练数据构造属于自监督式的条件切分——以片段的前若干帧作为条件帧、后续帧作为预测目标，用条件帧数量区分三类任务（T2V 为 0 帧条件、I2V 为 1 帧参考、VC 为多个前序帧），这是从真实视频自动派生监督信号，而非合成生成新内容。
此外 caption 的「随机组装电影语言与风格标签」可视为一种 prompt 侧的合成增强。[不确定：是否使用了生成模型自举的合成视频数据]

### 人工介入程度（人工标注、人工质检、模型初筛+人工复核） ⚠️

训练数据侧的人工介入几乎未被提及——数据 pipeline 通篇为自动化的模型打标 + 阈值过滤，无人工标注或人工复核环节的描述。
人工介入明确存在于两处下游环节：
(1) 评测：内部 T2V 评测集中 500 条 prompt 专供人工评测，采用 MOS（Mean Opinion Score）绝对打分与 GSB（Good-Same-Bad）相对比较双轨制，每条视频由 3 名独立标注员评价（three independent annotators）；I2V 评测集 400 条同样含人工评测。
(2) 奖励模型训练：Motion Quality 与 Text-Alignment 两个奖励模型均为「VideoAlign-based model fine-tuned on internal annotated datasets/internally annotated data」——即基于内部人工标注数据集微调，说明存在人工偏好/质量标注的投入，但标注量、标注规范、标注员规模与一致性指标均未披露。
可以概括为：人工不参与数据清洗，而是通过「标注奖励模型」的方式间接把人类偏好注入训练。[不确定：人工标注数据的规模]

## 音视频对齐

### 音视频同步检测方法（口型同步、事件对齐）

基础版不适用（无音频）。
Avatar 1.5 将唇同步校验作为离线标注阶段的独立一环与在线过滤的首道关卡：离线做 lip-sync verification，移除「samples with large audio-visual offsets」（音视频偏移过大的样本）；在线 clip 级校验中「audio synchronization」是渐进过滤链的第一级。配套的主动说话人检测使用 TalkNet 与 UniTalk 两个模型联合判定，确保音轨归属于画面中的目标人物而非画外音；多人场景以 YOLOv6 + ByteTrack 跟踪关联身份，并剔除多人并发说话区间。另有专门的静默数据分支（Qwen3-Omni + Qwen3-VL 双模型一致判定主体未说话）以覆盖「有音无口型」的合法情形。事件级（非唇动）的音视频对齐检测未涉及。

### 同步检测具体指标与阈值（SyncNet/Synchformer/LSE/自研，阈值数值，如MOVA LSE-D≤9.5且LSE-C≥4.5、SkyReels-V4 SyncNet |offset|≤3∧conf>1.5） ⚠️

[不确定]。Avatar 1.5 报告仅定性表述为移除「音视频偏移过大」的样本，未公布所用的同步度量指标（未提及 SyncNet conf / sync-c / sync-d / AV-align 等具体指标名），也未给出任何阈值数值。使用的 TalkNet、UniTalk 为主动说话人检测模型（输出说话/不说话及置信度），其判定阈值同样未披露。全流程中唯一公布了确切阈值的是情绪标注环节的 EmotiEffLib 置信度 s>0.7，与音视频同步无关。

### 时序同步vs语义同步的分离处理（时间对齐与内容语义匹配作为两个独立过滤条件） ⚠️

在 Avatar 1.5 的过滤链中，时序对齐与语义匹配确实被拆为不同环节，但并非作为「同一目标的两个正交条件」显式论述：时序侧为 lip-sync verification（音视频偏移检测）与主动说话人检测（音轨归属的时间区间判定）；语义/内容侧为 Qwen3-Omni + Qwen3-VL 对「主体是否在说话」的语义一致性双判，以及 caption 层面对场景/关系/情节的客观描述约束。在线过滤链把「audio synchronization」列为首级、把「文本与视觉质量」列为后续独立一级，结构上体现了二者分离。基础版无此维度。[不确定：团队是否有意作此区分]

### 音频质量过滤（SNR、静音检测与静音占比阈值、无音轨剔除、画外音源剔除、背景音乐分离） ⚠️

基础版不适用。
Avatar 1.5 的音频侧过滤描述较粗：离线阶段有专门的音频标注环节，「verifies whether a sample contains usable speech conditions」（校验样本是否含可用的语音条件），即以「是否存在可用语音」为准入门槛；数据来源选择上偏好访谈类等「主体稳定、语音清晰」的素材，属源头质量控制。报告提及使用了 vocal separation（人声分离），可用于把人声从背景音乐/环境音中剥离，但未说明所用模型与参数。画外音源（音轨主人不在画面内）通过主动说话人检测间接剔除。
未披露的有：SNR 阈值、静音检测与静音占比阈值、无音轨样本的处理规则、音频采样率/码率要求。[不确定：SNR、静音占比等具体阈值]

### 语音/音效/音乐的分类与分别处理策略

基础版不适用。
Avatar 1.5 按功能对音频做了粗粒度分流处理而非严格的语音/音效/音乐三分类：(1) 说话语音为主线，走完整的唇同步 + 主动说话人检测链路；(2) 歌唱/音乐单列一支（音乐视频来源），用于覆盖歌唱口型与节奏性肢体动作，与常规说话的口型动态特性不同；(3) 静默（无说话）单独构建子集，用双模型一致性判定保证「确实没在说话」，让模型学会音频存在但主体不发声的情形；(4) 多人并发说话区间被剔除，不作为一类保留。音效/foley 与环境音未作为独立类别处理——这与其定位一致：音频只是驱动条件，模型不生成音频，故无需覆盖完整音景类别。

## 训练配合

### 多阶段训练课程与数据课程调度（阶段划分依据：分辨率/时长/质量分/模态；低清→高清、图像→视频、短→长）

预训练为严格的五阶段渐进课程，调度依据同时包含「模态（图像→视频）、任务（单任务→多任务）、分辨率（低清→高清）」三条轴，时长轴固定为 93 帧不变：
· Stage 1：仅 T2I，256p，lr 1e-4，285k iterations —— 用纯图像先建立视觉先验与文本对齐，占总迭代量约 42%，投入最重；
· Stage 2：T2I + T2V，256p × 93 帧，lr 1e-4，140k iterations —— 引入视频，图像继续混入以稳住画质；
· Stage 3：T2I + T2V + I2V + VC 全任务，256p × 93 帧，lr 5e-5，164k iterations —— 在低分辨率下先补齐全部四类任务能力，学习率减半；
· Stage 4：全任务，480p × 93 帧，lr 5e-5，36k iterations —— 提分辨率，迭代量大幅收缩；
· Stage 5：全任务，480p + 720p × 93 帧混合，lr 2e-5，53k iterations —— 双分辨率混训，学习率再降。
后续为 SFT（480p+720p × 93 帧，lr 1e-5，7.5k iterations）与 RLHF/GRPO（480p+720p × 93 帧，约 0.5k iterations）。
整体呈现「大量低清图像打底 → 低清全任务铺开 → 少量高清精修 → 极少量 SFT/RL 对齐」的金字塔形迭代分配。任务的统一由「条件帧数量」实现：T2V 无条件帧、I2V 输入 1 帧参考、VC 输入多个前序帧，使四类任务能在同一批次内混合训练。此外生成侧采用时空双轴 coarse-to-fine 策略（先粗后精），配合 Block Sparse Attention 与蒸馏，推理提速约 10.1 倍。

### 数据配比随训练阶段的变化（预训练/退火annealing/SFT高质量子集） ⚠️

配比随阶段变化的机制明确但缺乏数值：
(1) 模态配比：Stage1 纯图像 → Stage2 起图像与视频混合，图像数据在全部含视频的阶段持续保留（T2I 始终在任务列表中），但图像与视频的具体比例未披露；
(2) 任务配比：Stage3 起 T2I/T2V/I2V/VC 四任务同批混训，四者采样比例未披露；
(3) 分辨率配比：Stage5 起 480p 与 720p 混合，两档比例未披露；
(4) 内容配比：由 caption 嵌入聚类得到的类目体系支撑「不同训练阶段动态、精确地分配数据子集」，并可据聚类结果做定向补数据或重平衡——这是配比调度的核心机制，但同样无数值；
(5) 质量门槛随阶段收紧：SFT 阶段改用高质量精选集（按 aesthetic score、video quality、motion quality 多指标筛选），并叠加「按 caption 嵌入空间密度倒数采样」的长尾均衡策略，同时引入相机运动与视觉风格两类专项数据集。
报告未描述独立的退火（annealing）阶段，Stage5 的双分辨率混训 + 逐级降学习率（1e-4→5e-5→2e-5→1e-5）在效果上承担了类似角色。[不确定：所有配比的具体数值]

### 后训练数据（SFT精选集规模与筛选标准、偏好对数量与标注方式、reward model训练数据） ⚠️

分 SFT 与 RLHF 两段：
【SFT】数据为「carefully curated, high-quality dataset」，筛选标准有二层——第一层多指标过滤（aesthetic score、video quality、motion quality）；第二层分布均衡：样本按其在 caption 嵌入空间中的密度的倒数被选取（inversely proportional to their density in the caption embedding space），实现长尾概念的相对提升。另额外并入专项数据集以强化相机运动与视觉风格的指令跟随。训练配置 480p+720p × 93 帧、lr 1e-5、7.5k iterations。精选集样本量未披露。
【RLHF】采用 GRPO（Group Relative Policy Optimization）多奖励在线强化，而非离线 DPO，因此不存在预先标注的偏好对数据集——偏好信号由奖励模型在 rollout 时在线给出。配置：group size 4、每步 64 prompts、约 0.5k iterations，任务为 T2V，分辨率 480p+720p × 93 帧。三个奖励模型：
· 视觉质量 VQ：HPSv3-general（全帧平均分）+ HPSv3-percentile（取分数最高的前 30% 帧、结合视频 caption 计算）双路，后者用于抑制少数低质帧对整体判断的稀释；
· 运动质量 MQ：以 VideoAlign 为基座、在内部标注数据上微调，输入灰度视频（去色以迫使模型只评估运动而不被色彩/美学干扰）；
· 文本对齐 TA：同样 VideoAlign 基座、内部标注数据微调，输入彩色视频。
多奖励并用的目的是抑制单一奖励导致的 reward hacking。用于训练 MQ/TA 的内部人工标注数据规模未披露。
Avatar 1.5 的后训练进一步把 GRPO 从视频级奖励扩展到逐帧奖励（per-frame reward modeling），首帧引入手部可见性检查以优先采样含手样本，多段 rollout 支持最多 5 个 clip 且仅末段参与优化；并用 DMD2 做步数蒸馏至 8 步（文本与音频 CFG 均设为 4.0）。[不确定：SFT 精选集规模、奖励模型标注数据量]

### 数据处理基础设施与吞吐（NeMo Curator/Data-Juicer/自研，GPU加速比，处理规模，成本） ⚠️

报告仅在预处理末端提到一句工程实现：处理后的片段被「compressed and packaged」（压缩打包），目的是支撑训练时的高效数据加载（efficient training data loading）——即采用打包式数据格式避免海量小文件的 IO 瓶颈，但未说明具体格式（如 webdataset/tar shard/自研格式）。
未披露的有：是否使用 NeMo Curator、Data-Juicer 等开源数据处理框架或自研平台、GPU 加速比、处理吞吐（视频小时/天）、集群规模与处理成本。训练侧的效率工作（Block Sparse Attention、条件 token 缓存、蒸馏，合计推理提速约 10.1 倍）披露充分，但那是模型推理侧而非数据处理侧的基础设施。[不确定：数据处理基础设施与吞吐]

## 效果对比

### 数据策略消融的量化影响（区分：过滤严格度ablation / caption密度风格ablation / 数据配比ablation，及对应评测指标） ⚠️

[不确定]。技术报告未包含任何数据策略的消融实验——既没有过滤严格度的 ablation，也没有 caption 密度/风格的 ablation，更没有数据配比的 ablation。报告中的消融实验全部集中在算法与架构侧：GRPO 的策略重加权与 KL 重加权、group 内标准差上限（max group std deviation）等 RL 技巧，以及 Block Sparse Attention、coarse-to-fine 策略、条件 token 缓存等推理效率组件。因此无法从该报告中量化任何数据处理环节对最终指标的贡献，这是其数据侧披露的主要短板。

### 质量vs数量的证据（小而精数据超越大而杂的案例） ⚠️

[不确定]。报告未提供「小而精超越大而杂」的直接量化证据（无对照实验）。仅能观察到策略层面的间接体现：SFT 阶段用「carefully curated, high-quality dataset」以仅 7.5k iterations（相较预训练 678k iterations 不足 1.1%）完成风格与指令跟随的对齐；以及「按 caption 嵌入密度倒数采样」这一显式偏向稀有样本、主动放弃冗余高密度样本的设计，隐含了团队对「分布均衡优先于样本数量」的判断。但这些均无消融数据支撑。

### 训练数据domain分布与评测基准类目体系的对齐关系（如VABench七大类） ⚠️

训练数据的类目体系与评测基准的类目体系之间存在方法论上的呼应，但报告未明确声明二者对齐：训练侧的 domain 划分来自 caption 文本嵌入聚类 + LLM 归纳的无监督类目（如人际互动、艺术表演、自然风光等）；评测侧的内部 T2V 基准覆盖 48 个不同类别（48 distinct categories），共 1228 条用例（500 条人工评测 + 728 条自动评测）；I2V 基准为 100 张首帧参考图 × 4 类 prompt 共 400 条。报告未说明这 48 个评测类别是否直接派生自训练数据的聚类类目，也未给出二者的映射关系或各类目在两侧的占比对照。人工评测采用 MOS 与 GSB 双指标、每条视频 3 名独立标注员。公开基准方面模型亦有对比（HF 模型卡给出 T2V 综合质量 MOS 3.38 的内部评测结果）。[不确定：48 类目与训练聚类类目的对应关系]

## 不确定字段

以下字段的调研信息部分不确定（⚠️ 标注来源）：

- data_scale
- data_sources
- provenance_licensing
- duration_distribution
- resolution_aspect_distribution
- domain_distribution
- audio_category_distribution
- language_accent_distribution
- funnel_retention_rate
- quality_filtering
- motion_filtering
- deduplication
- safety_filtering
- caption_model
- caption_structure
- dialogue_transcription_attributes
- geometric_structured_annotation
- synthetic_data_synthesis
- human_in_loop
- sync_metric_and_threshold
- temporal_vs_semantic_sync
- audio_quality_filtering
- stage_data_mixture
- post_training_data
- data_infra_throughput
- data_ablation
- quality_over_quantity_evidence
- benchmark_taxonomy_alignment
