# NVIDIA NeMo Curator（26.02 / 26.04 版）+ Cosmos-Xenna（视频侧底层分布式执行引擎），并附带同源的产品化实现 Cosmos-Curate（Cosmos 世界基础模型的训练数据生成系统）

> 主题: 视频生成模型（含音视频同时生成）的数据处理：数据清洗流程、数据分布、打标方式

[← 返回首页](../index.md)

**目录**: [基本信息](#基本信息) · [数据规模与分布](#数据规模与分布) · [清洗流程](#清洗流程) · [打标方式](#打标方式) · [音视频对齐](#音视频对齐) · [训练配合](#训练配合) · [效果对比](#效果对比)

## 基本信息

### 名称

NVIDIA NeMo Curator（26.02 / 26.04 版）+ Cosmos-Xenna（视频侧底层分布式执行引擎），并附带同源的产品化实现 Cosmos-Curate（Cosmos 世界基础模型的训练数据生成系统）

### 发布机构/公司

NVIDIA。三个仓库分属不同 GitHub 组织：NeMo Curator 在 NVIDIA-NeMo/Curator，Cosmos-Xenna 与 Cosmos-Curate 在 nvidia-cosmos 组织（早期为 NVIDIA/cosmos-curator）。NeMo Curator 由 NeMo Framework 团队维护，Cosmos-Xenna 由 Cosmos（Physical AI / 世界基础模型）团队开发后独立开源。

### 发布时间（技术报告/论文/开源时间）

【NeMo Curator 版本线（CalVer 与 SemVer 双轨，PyPI 上传时间为准）】0.6.0（2025-01-07，Dask 架构，纯文本时代）→ 0.7.0（2025-03-12）→ 0.8.0（2025-05-09）→ 0.9.0（2025-07-28）→ 1.0.0 = 25.09 版（2025-10-01，里程碑：后端由 Dask 全面切换为 Ray，首次引入视频与音频模态，形成文本/图像/视频/音频四模态统一架构）→ 1.1.0 = 26.02 版（2026-02-23）→ 1.2.0 = 26.04 版（2026-05-14）→ 1.3.0 = 26.07 版（2026-07-27，本调研截止日前两天发布）。
【Cosmos-Xenna】随 Cosmos 平台于 2025 年从 Cosmos-Curator 中拆分独立开源（Apache 2.0）；26.04 版 NeMo Curator 升级至 Cosmos-Xenna 0.2.0。截至 2026 年 7 月，cosmos-xenna 仓库标注「不再积极开发」，官方引导迁移至 Cosmos 3 / NeMo Curator 体系。
【方法论一手文献】Cosmos World Foundation Model Platform for Physical AI（arXiv:2501.03575，2025-01-07，第3节完整披露七级视频数据 curation pipeline）；Training Video Foundation Models with NVIDIA NeMo（arXiv:2503.12964，2025-03，专门描述 clipping/sharding 双 pipeline 与 GPU 加速）；NVIDIA 官方博客首次公布「89x 加速」为 2025-01。

### 类型（模型/数据集/工具链/评测基准）

工具链 / 数据基础设施框架（data curation toolkit）。不是生成模型，不是数据集，也不是评测基准。定位为「可复现的、GPU 加速的大规模数据处理流水线构建框架」，覆盖文本、图像、视频、音频四种模态的加载—过滤—去重—标注—变换—写出全流程。在本调研的对象体系中，它属于「被其他视频生成/世界模型团队使用的上游基建」，NVIDIA 官方口径称其为工业级视频数据处理的开源参考实现。

### 开源程度（权重/代码/数据/pipeline各自是否开源）

开源程度在同类中最高，属于「pipeline 完全开源、数据完全不含」的类型。
【代码】NeMo Curator 全部源码 Apache License 2.0，GitHub 公开（NVIDIA-NeMo/Curator），PyPI 包名 nemo-curator，官方推荐 Docker 容器安装（视频/音频 workflow 需要预配置的 FFmpeg 8.0.1 + NVENC）。
【底层引擎】Cosmos-Xenna 同为 Apache 2.0 独立开源，可脱离 Curator 单独使用。
【产品化实现】Cosmos-Curate 源码 Apache 2.0；其调用的模型权重走 NVIDIA Open Model License，可申请自定义商业许可。
【pipeline 配置】26.02 起支持 YAML 声明式定义整条 curation pipeline，进一步降低复现门槛。
【模型权重】自身不训练模型，调用的判别/标注模型多为第三方开源权重：Qwen2.5-VL / Qwen3-VL（captioning）、Cosmos-Embed1 与 InternVideo2（视频嵌入，InternVideo2 在 26.02 被移除）、CLIP-based aesthetic model（美学打分）、TransNetV2（镜头切分）、NVIDIA NeMo ASR 系列（音频转写）、Nemotron Nano 12B V2 VLM 与 Nemotron 3 Nano Omni（26.04/26.07 新增的 captioning 后端，含 BF16/FP8/NVFP4 三种精度变体）。
【不开源】训练数据本身（框架本身不附带任何数据集）；NVIDIA 内部用于 Cosmos 的实际数据源清单与全量统计不公开。

### 是否支持音视频同时生成，以及实现方式（原生联合/级联/MoE融合）

不适用——本条目是数据处理工具链而非生成模型，不产出任何音视频内容。
就「音视频联合处理能力」而言需明确指出其当前局限：NeMo Curator 虽然号称四模态（文本/图像/视频/音频），但视频 pipeline 与音频 pipeline 在架构上相互独立、不交叉：视频侧只处理视觉轨道（切分/转码/抽帧/运动与美学过滤/captioning/嵌入/去重/分片），音频侧是面向 ASR 语音数据的独立工作流（加载→NeMo ASR 转写→WER/CER 质量评估→与文本 curation 衔接→导出）。官方文档中音频模块未提及任何与视频的联动，也没有从视频中抽取音轨、做音视频对齐或联合打标的 stage。因此它目前不能直接支撑「音视频同时生成」模型的数据构建，是该框架相对于 LTX-2 / Ovi / Sora 2 等 AV 联合生成模型数据需求的最大缺口。

### 调研信息来源列表（论文/技术报告/官方文档/新闻的URL，标注每条来源的性质：官方一手/同团队旁证/第三方报道）

- 【官方一手】NeMo Curator GitHub 主仓库（Apache 2.0，四模态能力、Ray/Xenna 架构、性能声明、安装方式）: https://github.com/NVIDIA-NeMo/Curator
- 【官方一手】NeMo Curator Releases 页面（各版本 highlights，含 26.02/26.04/26.07）: https://github.com/NVIDIA-NeMo/Curator/releases
- 【官方一手】PyPI nemo-curator 发布历史（精确发布时间：1.0.0=2025-10-01、1.1.0/26.02=2026-02-23、1.2.0/26.04=2026-05-14、1.3.0/26.07=2026-07-27）: https://pypi.org/project/nemo-curator/
- 【官方一手】NeMo Curator 视频 curation 总览文档（各 stage 与所用模型：TransNetV2、Qwen-VL、Cosmos-Embed1、CLIP aesthetic、NVENC/NVDEC、语义去重、WebDataset）: https://docs.nvidia.com/nemo/curator/curate-video
- 【官方一手】NeMo Curator 视频过滤文档（运动过滤与美学过滤的算法、stage 名称、全部默认阈值与参数）: https://docs.nvidia.com/nemo/curator/curate-video/process-data/filtering
- 【官方一手】NeMo Curator 视频架构文档（Ray Actor 管理、Ray Object Store、Cosmos-Xenna executor、自动扩缩容与全部默认配置项）: https://docs.nvidia.com/nemo/curator/about/concepts/video/architecture.md
- 【官方一手】NeMo Curator 音频 curation 文档（ASR 转写、WER/CER 过滤、说话人分离；确认与视频无联动）: https://docs.nvidia.com/nemo/curator/curate-audio
- 【官方一手】NeMo Curator 26.07 Release Notes / 迁移清单（音频增强 stage、Nemotron-CLIMB、captioning 后端矩阵、breaking changes）: https://docs.nvidia.com/nemo/curator/about/release-notes
- 【官方一手】NeMo Curator 26.02 视频快速上手文档: https://docs.nvidia.com/nemo/curator/26.02/get-started/video.html
- 【官方一手】NeMo Curator 26.02 转码/Clip Encoding 文档: https://docs.nvidia.com/nemo/curator/26.02/curate-video/process-data/transcoding.html
- 【官方一手】Cosmos-Xenna GitHub 仓库 README（Ray 分布式数据流水线库、streaming/batch/serving 三模式、自动扩缩容与 bin-packing、backpressure、SPMD、P2P 分发、Apache 2.0、仓库已停止积极开发的说明）: https://github.com/nvidia-cosmos/cosmos-xenna
- 【官方一手】Cosmos-Curate GitHub 仓库 README（Cosmos 训练数据生成系统，构建于 Cosmos-Xenna 之上，代码 Apache 2.0 / 模型 NVIDIA Open Model License）: https://github.com/nvidia-cosmos/cosmos-curate
- 【官方一手·方法论核心】Cosmos World Foundation Model Platform for Physical AI, NVIDIA, arXiv:2501.03575（第3节完整披露七级 curation pipeline：TransNetV2 选型 F1=0.967 横评、NVDEC/NVENC 6.5x、ViT 光流运动分类器、DOVER 底部15%、美学阈值3.5、InternVideo2+MLP 文字叠加与类型分类、VILA 13B FP8 TensorRT-LLM 10x、caption 559字符/97词、k-means k=10000 去重移除30%、九大类配比数字、20M小时→100M clip）: https://arxiv.org/abs/2501.03575
- 【官方一手】Training Video Foundation Models with NVIDIA NeMo, arXiv:2503.12964（clipping/sharding 双 pipeline 结构、100PB+ 口径、NVDEC/NVENC 3x、captioning 为瓶颈 stage、Ray 自动均衡 worker）: https://arxiv.org/abs/2503.12964
- 【官方一手·转载】Accelerate Custom Video Foundation Model Pipelines with New NVIDIA NeMo Framework Capabilities（NVIDIA 官方博客，89x 加速原始出处：「1K GPUs、ISO 功耗、相较未优化 CPU pipeline」；20M 小时；100PB+；L40S/H100/GB200 异构集群）: https://www.edge-ai-vision.com/2025/01/accelerate-custom-video-foundation-model-pipelines-with-new-nvidia-nemo-framework-capabilities/
- 【官方一手】Advancing Physical AI with NVIDIA Cosmos World Foundation Model Platform（NVIDIA 技术博客，2000万小时在 Hopper 40天 / Blackwell 14天 / CPU 3.4年）: https://developer.nvidia.com/blog/advancing-physical-ai-with-nvidia-cosmos-world-foundation-model-platform/
- 【官方一手】World Simulation with Video Foundation Models for Physical AI（Cosmos 后续论文 arXiv:2511.00062）: https://arxiv.org/pdf/2511.00062
- 【官方一手】NVIDIA Nemotron 3 Nano Omni 技术博客（26.07 引入 Curator 的全模态模型背景）: https://developer.nvidia.com/blog/nvidia-nemotron-3-nano-omni-powers-multimodal-agent-reasoning-in-a-single-efficient-open-model/
- 【官方一手】NeMo Curator 视频切分示例代码 tutorials/video/getting-started/video_split_clip_example.py: https://github.com/NVIDIA-NeMo/Curator/blob/main/tutorials/video/getting-started/video_split_clip_example.py
- 【官方一手】Cosmos Curator 视频 pipeline 参考文档 docs/curator/reference/video-pipelines.md: https://github.com/NVIDIA/cosmos-curator/blob/main/docs/curator/reference/video-pipelines.md
- 【第三方报道】Architecting Data Pipelines for Multimodal Datasets at Scale（Anyscale 博客，Ray 侧视角的多模态数据流水线架构）: https://www.anyscale.com/blog/architecting-multimodal-data-pipelines-that-scale-with-ray

## 数据规模与分布

### 训练数据量级（视频条数/小时数/token数，预训练与SFT分开）

框架本身不含数据，此处记录其「已验证处理规模」与「设计容量」：
【设计容量】官方反复给出的口径是「可高效地对 100 PB 或以上量级的视频进行切分、标注与过滤」（clip, annotate, and filter 100 PB or more of videos）。
【实测规模（Cosmos WFM 生产运行）】输入约 2000 万小时（20M hours）原始视频，分辨率 720p–4K；输出约 1 亿（100M）个 2–60 秒的 clip；其中约 10^8 量级用于预训练、约 10^7 量级用于微调（fine-tuning）。
【处理耗时】2000 万小时视频：Hopper（H100）GPU 集群约 40 天完成；Blackwell 约 14 天；同等条件下未优化的 CPU pipeline 需约 3.4 年。另一组官方口径为「用 1000 张 GPU 在 ISO 功耗下相较未优化 CPU pipeline 达 89 倍加速，把处理时间从数年压缩到数天」。
注意：这些数字是 NVIDIA 自家 Cosmos 项目的运行规模，不是 NeMo Curator 用户的通用数字。

### 数据来源构成（自有/公开数据集/网络爬取/授权采购/合成数据） ⚠️

作为工具链，它是「数据源无关（source-agnostic）」的，本身不规定数据来源。已支持的数据接入方式包括：本地文件系统、S3 兼容对象存储（26.04 新增 CommonCrawl S3 传输选项）、自定义 manifest、Hugging Face 数据集（音频侧示例为 FLEURS）、以及 WebDataset 格式的读写。
其上游生产用例（Cosmos WFM）的数据来源构成为：专有/自有视频数据集 + 公开互联网视频，并明确包含合成渲染视频（synthetically rendered，占最终配比约 4%）。NVIDIA 未披露各来源的具体占比、采购渠道或授权方式。[不确定]

### 数据合规与溯源（授权数据占比、rights-cleared数据集、C2PA等） ⚠️

NeMo Curator 框架层面未提供数据溯源与授权管理能力：官方文档中没有 C2PA 内容凭证写入、没有版权指纹检测、没有 rights-cleared 数据集管理模块，也没有 provenance metadata 的标准化字段规范。框架只保证在分片（sharding）时保留原始 metadata 透传。
合规责任被完全交给使用者：Apache 2.0 许可下 NVIDIA 不对用户所处理数据的合法性做任何保证；Cosmos 模型侧的合规叙事（Guardrails、可信 AI 声明）与数据 curation pipeline 是分离的两套东西。Cosmos WFM 论文亦未披露其 2000 万小时视频的授权比例与来源合法性论证。这是该工具链相对于 Adobe Firefly / Lightricks（Shutterstock、Getty 授权）等强合规叙事方案的明显空白。[不确定]

### 片段时长分布与切分策略 ⚠️

【切分策略（框架能力）】提供两种 clip 抽取模式，均可配置：(1) fixed-stride 固定步长切分（按固定秒数硬切，配置 clip 长度与步长）；(2) scene-change detection 场景变化检测切分（基于 TransNetV2，shot-aware）。二者可叠加使用——先按镜头切，再对超长镜头按固定步长细分。
【生产实践中的时长阈值（Cosmos WFM）】切分后丢弃短于 2 秒的 clip；长于 60 秒的 clip 进一步细分；最终 clip 时长分布区间为 2–60 秒。captioning 时的处理粒度为「每 256 帧生成一条 caption」，即超长 clip 会被切成多个 caption 段。
【分布数值】各时长区间的具体样本占比未公布。框架本身不强制任何时长分布，由用户配置决定。[不确定]

### 分辨率/宽高比分布与分桶策略 ⚠️

【框架能力】分辨率与宽高比不作为过滤条件，而是作为分片（sharding）阶段的组织维度：最终 WebDataset 按「分辨率 × 宽高比 × 时长」三维分桶打包，目的是与下游训练课程（training curriculum）的分桶需求对齐，使训练时能按桶取样、避免同 batch 内 token 数差异过大。这是该 pipeline 在「数据基建与训练课程耦合」上最值得借鉴的设计。
【转码归一】所有 clip 统一转码为 H.264 mp4，编码器可选 h264_nvenc（GPU）/ libx264 / libopenh264（CPU）；解码器可选 nvdec（GPU）/ ffmpeg。抽帧阶段（ClipFrameExtractionStage）可配置目标 fps 与抽帧策略（如 sequence 策略供美学打分使用）。
【生产实践】Cosmos WFM 的输入素材为 720p 至 4K。各分辨率/宽高比桶的具体样本占比未公布。[不确定]

### 类别/domain分布与配比策略（人物、动作、场景、风格等比例控制与概念均衡）

这是本条目最有参考价值的字段之一——Cosmos WFM 通过 NeMo Curator/Cosmos-Curate 实现了显式的、基于分类器的 domain 配比控制，是公开资料中少见的「数据配比可量化」的工业案例。
【实现机制】在 InternVideo2 视频嵌入之上训练一个轻量 MLP 分类器，按预定义的视频类型 taxonomy 对每个 clip 打类别标签，然后据此做过滤与重采样。
【排除类别】抽象图案（abstract patterns）、电子游戏画面（video game footage）、动画（animation）——这三类被判定为不利于学习真实世界物理规律而整体剔除。
【重采样策略】上采样（upsample）人类动作与人-物交互类目；下采样（downsample）风景类目——因为风景视频虽然画质高但动态信息与物理交互信息稀薄，天然过量。
【最终九大类配比（Cosmos WFM 论文公布）】自然动态（nature dynamics）20%、手部动作/物体操控（hand motion & object manipulation）16%、空间感知与导航（spatial awareness & navigation）16%、驾驶（driving）11%、人体动作/活动（human motion & activity）10%、第一人称视角（first-person POV）8%、动态镜头（dynamic camera）8%、其他（other）7%、合成渲染（synthetically rendered）4%。
【说明】这套 taxonomy 服务于 Physical AI / 世界模型的目标（学习物理规律），与通用文生视频模型（重人物、影视感、风格多样性）的配比目标并不相同，迁移时需重新定义类目体系。NeMo Curator 开源版本中未内置该 taxonomy 分类器权重，需用户自行训练。

### 音频类别分布与配比（语音/音效foley/音乐/环境音/静音的比例及控制策略）——AV模型独有维度

无。这是该工具链在音视频生成数据场景下最关键的缺失维度。
视频 pipeline 完全不处理音轨——没有语音/音效 foley/音乐/环境音/静音的分类 stage，没有各类音频的占比统计或配比控制策略，也没有从视频中抽取音轨的内置 stage。音频 pipeline 虽存在但其数据模型是「独立的音频文件 + 转写文本」（面向 ASR/TTS 语音数据），仅涵盖语音一类，且不与视频关联。26.07 版新增的音频增强 stage（tagging、SQUIM 质量指标、带宽估计、标点准备、可选二次 ASR 打分）也全部围绕语音质量，不涉及 foley/音乐/环境音分类。
因此若要用 NeMo Curator 构建音视频联合生成的训练数据，音频类别分布这一层需完全自建。

### 叙事结构分布（单镜头vs多镜头、平均clip时长、镜头数分布、是否含原生音轨） ⚠️

【单镜头 vs 多镜头】框架的默认产物是单镜头 clip：TransNetV2 做 shot-aware splitting，产出的每个 clip 原则上不跨镜头。但 pipeline 中有一个值得注意的反向设计——stitching（缝合）阶段：在切分之后，用图像嵌入相似度判断相邻 clip 是否内容连续，若相似度高则将其重新合并，以避免把同一连续场景过度碎片化（如摄影机短暂遮挡、闪光导致的误切）。这是「切分—再缝合」的两段式设计，比单纯依赖场景检测更稳健。
【镜头数分布】最终数据集以单镜头 clip 为主，多镜头叙事样本不是构建目标；镜头数分布无统计数字。
【平均 clip 时长】区间 2–60 秒，具体均值未公布。
【是否含原生音轨】pipeline 不保留、不处理音轨，转码输出为 H.264 视频，故最终 WebDataset 中不含音频轨道信息。[不确定]

### 语言/口音分布（多语种唇同步能力的数据基础） ⚠️

视频侧完全不涉及（不处理音轨即无语言维度）。
音频侧有间接相关能力但无分布统计：音频 pipeline 使用 NVIDIA NeMo ASR 模型做转写，示例数据集为 FLEURS（多语种语音基准，覆盖 100+ 语种），并有 long-form audio cutting 教程涉及说话人分离（speaker diarization）。26.02 起支持 streaming Sortformer 做说话人分离、VAD 与说话人切分。但框架未提供语种识别（LID）stage、口音标注能力，也未给出任何语言/口音配比的控制机制或统计。多语种唇同步所需的数据基础无法由该框架直接产出。[不确定]

## 清洗流程

### 清洗漏斗整体结构（几级过滤、各级顺序）

这是本条目的核心价值所在。分两个层次记录。

【一、Cosmos WFM 生产 pipeline：七级漏斗（arXiv:2501.03575 一手披露）】
1. Shot-aware video splitting（镜头感知切分）——TransNetV2 检测镜头边界，切出 clip，并用图像嵌入相似度做 stitching 缝合；
2. GPU-based transcoding（GPU 转码）——统一转为 H.264 mp4，NVDEC/NVENC 硬件加速；
3. Video cropping（裁剪）——统一宽高比/去黑边；
4. Filtering（过滤）——运动过滤 + 视觉质量过滤 + 文字叠加过滤 + 视频类型 taxonomy 过滤，四类判别器串联；
5. Captioning（标注）——VLM 生成密集描述；
6. Semantic deduplication（语义去重）——嵌入 + k-means 聚类 + 簇内成对距离；
7. Sharding（分片）——按分辨率/宽高比/时长分桶打包为 WebDataset。

【二、NeMo Curator 开源框架的两条 pipeline（arXiv:2503.12964 与官方文档）】
(A) Clipping Pipeline（切片流水线）：解码原始视频 → 按帧间颜色变化切分为连续短 clip → 基于图像嵌入相似度 stitching 合并相邻 clip → 转码为 H.264 → 抽帧（ClipFrameExtractionStage）→ 运动向量解码与运动过滤 → 美学过滤 → 视频嵌入标注（Cosmos-Embed1）→ VLM captioning（可选 LLM 改写增强）→ 可选 WebP 预览图 → 语义去重 → 写出。
(B) Sharding Pipeline（分片流水线）：为 caption 生成文本嵌入 → 生成 WebDataset 格式文件（嵌入以 Parquet 存储，metadata 保留），使训练时可对 PB 级数据做顺序读取的多 GPU 并行访问。

【三、架构特征】
- 所有 stage 统一抽象为 ProcessingStage（声明自身的 CPU/GPU 资源需求与输入输出数据契约），Pipeline 将 stage 串联，执行由可插拔的 executor 承担；
- 视频 pipeline 默认使用 XennaExecutor（Cosmos-Xenna），将 ProcessingStage 翻译为 Xenna stage spec 并在 Ray 上以 streaming 或 batch 模式执行；另有实验性的 RayDataExecutor（26.04 转正）；
- 26.02 起支持 YAML 声明式定义整条 pipeline，并提供 Pipeline.describe() 用于开发期检查各 stage 资源与数据需求；
- 26.07 起支持 pipeline 可恢复性（resumability）：已完成分片记录在 LMDB 中，重启时跳过，并支持 SLURM job array。

【四、结构特点评价】典型的「先几何/编码归一 → 再判别式过滤 → 再语义标注 → 再去重 → 最后按训练课程分桶打包」五段式。与 LTX-Video 等模型自研 pipeline 的最大差异在于：它把 sharding/WebDataset 生成也纳入 curation 范畴，使数据处理与训练数据加载形成闭环；并且是 streaming 而非 batch 执行——各 stage 并发运行、数据持续流动，避免中间产物全量落盘。

### 漏斗定量保留率（每级过滤的输入/输出量与最终保留率，如Apollo 27%） ⚠️

只有单级定量数字，没有完整逐级漏斗表。
【已公布的定量点】(1) 语义去重环节移除约 30% 的训练数据（remove approximately 30% of training data）；(2) 视觉质量过滤中，基于 DOVER 的失真评估模型剔除得分最低的 15%（bottom 15%）；(3) 切分阶段丢弃全部短于 2 秒的 clip（占比未公布）；(4) 总体从 2000 万小时原始视频得到约 1 亿个 2–60 秒 clip——若按平均 clip 时长 15 秒粗算约合 41 万小时，相对输入的时长保留率约 2%，但该推算不严谨（原始视频中大量内容在切分阶段即被丢弃，且论文未确认 clip 平均时长）。
【缺失】运动过滤、文字叠加过滤、视频类型 taxonomy 过滤各自的淘汰率未公布；各级的输入/输出样本量表格未提供；无法构成类似 Apollo 27% 那样的端到端保留率数字。[不确定]

### 镜头切分方法（PySceneDetect/自研模型/shot-aware splitting）

披露充分且有选型依据，是该 pipeline 中论证最扎实的一环。
【选型】TransNetV2 神经网络做镜头边界检测。Cosmos WFM 论文明确记录了与 PySceneDetect、Panda70M 的切分器、AutoShot 三个替代方案的基准对比，TransNetV2 在 BBC 数据集上取得 F1 = 0.967 的最佳成绩而被选用。这是公开资料中罕见的、把「镜头切分器选型」当作可量化决策来做的案例，值得直接借鉴其对比方法。
【框架实现】NeMo Curator 提供两种切分模式并可组合：fixed-stride（固定步长）与 scene-change detection（TransNetV2）。
【后处理】切分后有 stitching 阶段：用图像嵌入相似度判断相邻 clip 是否应合并，抑制过度碎片化。
【时长约束】< 2 秒丢弃，> 60 秒再切分。
【未披露】TransNetV2 的判定阈值取值、stitching 的相似度阈值。

### 质量过滤（美学评分、清晰度、OCR文字过滤、黑边/水印/logo检测）

视觉质量过滤由四类判别器组成（Cosmos WFM 生产版），其中前两类已在开源 NeMo Curator 中提供：
【1. 美学过滤（开源已提供）】使用 CLIP-based aesthetic model 对抽帧逐帧打分，再按 clip 聚合。关键参数：reduction 可选 min（默认）或 mean——默认取最小值意味着「一帧不合格则整个 clip 淘汰」的严格策略；score_threshold 默认 3.5；target_fps 默认 1.0；num_gpus_per_worker 默认 0.25。要求上游抽帧使用 sequence 策略且帧率匹配。Cosmos WFM 生产环境同样使用 3.5 这一阈值。
【2. 失真/画质评估】基于 DOVER（视频质量评估模型）的打分模型，剔除得分最低的 15% clip。该 stage 在开源版中未见对应实现。
【3. 文字叠加检测（OCR 类功能的替代方案）】不使用传统 OCR，而是在 InternVideo2 视频嵌入之上训练 MLP 分类器，识别后期添加的文字叠加（post-processed text overlay），剔除文字过多的视频。这一「用嵌入 + 轻量 MLP 替代重型 OCR」的做法在 PB 级规模下成本优势显著，是值得借鉴的工程取舍。
【4. 视频类型 taxonomy 分类】同样是 InternVideo2 嵌入 + MLP，用于排除抽象图案/游戏画面/动画（见 domain_distribution）。
【未覆盖】水印/logo 检测、黑边检测（cropping 为独立 stage 但未说明是否自动检测黑边）、压缩伪影检测、模糊度单独判据，官方文档均未提供对应 stage。被淘汰的 clip 会被移入 video.filtered_clips 并更新统计，便于事后审计漏斗——这是良好的可观测性设计。

### 运动过滤（光流/运动分数阈值、静态与抖动剔除）

开源框架中实现最细、参数最透明的过滤环节，两级 stage 串联：
【MotionVectorDecodeStage】不做完整光流计算，而是直接从压缩码流中解码轻量运动向量（lightweight motion vectors）并据此计算运动分数——这是显著的成本优化，避免了在 PB 级数据上跑光流网络。参数：target_fps 默认 2.0（每秒采样帧数）、target_duration_ratio 默认 0.5（只分析 clip 的一半时长以省算力）、num_cpus_per_worker 默认 4.0、num_gpus_per_worker 可选 0.5。
【MotionFilterStage】施加阈值过滤。产出两个指标：motion_score_global_mean（全局平均运动分，默认阈值 0.00098）与 motion_score_per_patch_min_256（256 个空间 patch 上的最小运动分，默认阈值 0.000001）。双指标设计的用意：全局均值防「整体静止」，patch 最小值防「画面大部分静止、仅局部小物体动」的伪动态样本。
【Cosmos WFM 生产版增强】另有一个基于 ViT 架构、在光流序列上训练的运动分类器，除剔除静止内容外还能剔除剧烈抖动的失控镜头（erratic camera motion），并对镜头运动类型打标（pan 平移 / zoom 推拉 / tilt 俯仰）——即运动过滤同时兼任「镜头运动标注」的职能，产出可用于运镜控制训练的标签。该 ViT 分类器在开源版中未提供权重。

### 去重方法（精确去重与基于embedding的语义去重分别记录） ⚠️

以语义去重为主，方法链路清晰：
【嵌入】用 Cosmos-Embed1 生成 clip 级视频嵌入（早期与 Cosmos WFM 生产版用 InternVideo2；InternVideo2 已在 26.02 版中从 NeMo Curator 移除）。嵌入以 Parquet 格式落盘，同一份嵌入同时服务于语义检索与去重两个用途。
【聚类】对嵌入做 k-means 聚类，Cosmos WFM 生产环境取 k = 10000。
【簇内比对】只在同一簇内计算成对距离（pairwise distance）识别近重复，避免全量 N² 比对，这是 PB 级去重可行性的关键。
【保留策略】近重复组内保留分辨率最高的版本。
【效果】约移除 30% 的训练数据。
【框架实现】文档表述为「semantic clustering + pairwise similarity + k-means」，按 clip 分块（chunks of clips）组织去重。
【精确去重】视频侧未见基于哈希的精确去重 stage（文本侧另有成熟的 exact dedup 与 fuzzy dedup，后者官方称在 RedPajama v2 上相较 CPU 方案有 16 倍加速、TCO 降低 40%，但那是文本模态的数字，不可挪用到视频）。相似度阈值取值未公布。[不确定]

### VLM/LLM作为质检员（多模态大模型质量打分与错配剔除，2026年从浅层打分器转向大模型语义判断的趋势）

呈现出明确的代际演进，是观察「浅层打分器 → 大模型语义判断」这一 2026 趋势的好样本：
【25.09/早期 Cosmos 生产版：浅层判别器为主】质检职责由轻量模型承担——CLIP-based 美学模型、DOVER 画质模型、InternVideo2 嵌入 + MLP（文字叠加检测、视频类型分类）、ViT 光流运动分类器、运动向量统计。全部是小模型或线性/MLP 探针，设计动机是 PB 级规模下的单位成本。
【26.02–26.07：VLM/LLM 逐步进入流水线】(1) captioning 后端从 Qwen2.5-VL 扩展到 Qwen3-VL 与 Nemotron Nano 12B V2 VLM、Nemotron 3 Nano Omni，并提供 BF16/FP8/NVFP4 三档精度以压低大模型推理成本——FP8/NVFP4 量化是让 12B 级 VLM 在 PB 级数据上跑得起的前提；(2) 26.04 引入 vLLM 作为默认推理后端并内置 Ray Serve 推理服务器（OpenAI 兼容接口），使「在 pipeline 中挂一个 LLM 做判断」成为一等公民；(3) 26.02 起 captioning 支持可选的 Qwen-LM 二次改写增强；(4) 26.07 的 Nemotron OCR 合成数据流水线中，明确使用 Nemotron Nano Omni 做可选的质量打分（quality scoring）——这是官方文档中最直接的「大模型作为数据质检员」用法。
【仍然缺失】视频侧尚无「VLM 判断 caption 与画面是否错配」的专用剔除 stage，也无 VLM 语义质量打分 stage；大模型目前主要用于生成 caption 而非审核 caption。这一环需用户自建。

### 安全与合规过滤（NSFW、版权、人脸/隐私） ⚠️

NeMo Curator 的安全过滤能力在模态间严重不均衡：
【图像模态】提供 NSFW 检测 stage（官方 README 明确列为图像侧核心能力之一）。
【视频模态】官方文档的视频过滤章节只包含运动过滤与美学过滤两类，未提供 NSFW 分类器、人脸检测/打码、隐私信息移除、版权指纹比对等 stage。Cosmos WFM 论文的 curation 章节亦未描述数据侧的安全过滤——其安全能力集中在模型推理侧的 Cosmos Guardrails（输入 prompt 过滤 + 输出内容分类 + 人脸模糊化），属于部署护栏而非训练数据清洗。
【文本模态】有质量过滤与分类器体系，可间接承担部分内容安全职责。
【结论】用该框架构建视频训练数据时，NSFW、人脸隐私、版权三条合规线均需用户自行补充实现。[不确定]

## 打标方式

### 使用的caption模型（自研VLM/开源模型，模型规模）

该字段在版本演进上信息最丰富，是选型参考的重点：
【Cosmos WFM 生产版（2025-01）】使用 NVIDIA 内部微调的 VILA 13B 作为 captioner；部署为 FP8 量化的 TensorRT-LLM engine，相较未优化推理获得约 10 倍加速。输入为从 clip 中均匀采样的 8 帧。
【NeMo Curator 25.09 / 26.02】开源版默认 captioning 后端为 Qwen-VL 系列（Qwen2.5-VL），并支持可选的 Qwen-LM 对生成的 caption 做二次改写增强（caption enhancement / rewriting）。
【26.04 新增】Nemotron Nano 12B V2 VLM 作为与 Qwen-VL 并列的 captioning 后端，提供三种精度变体：nemotron / nemotron-bf16（默认 BF16，自动下载）、nemotron-fp8（FP8 量化，显存占用更低）、nemotron-nvfp4（NVFP4 量化感知蒸馏 checkpoint）。
【26.07 扩展】captioning 后端矩阵扩展为 Qwen2.5-VL、Qwen3-VL 与多个 Nemotron 变体（含 Nemotron 3 Nano Omni 全模态模型），各自可选 BF16/FP8/NVFP4；同时移除了视频预处理选项，统一改用 vLLM 或 HuggingFace 的预处理路径。
【工程要点】captioning 被官方明确指认为整条 pipeline 的速率瓶颈（rate-limiting stage）——其吞吐显著低于嵌入生成等其他 stage，因此 Xenna 的自动扩缩容会把最多的 worker 分配给该 stage。这一观察对任何自建视频数据 pipeline 都有直接参考价值。

### caption密度与结构化程度（短/长/密集描述、结构化字段如镜头运动、风格标签） ⚠️

【密度】走密集长描述路线。Cosmos WFM 生产版的实测统计：单条 caption 平均 559 个字符 / 97 个词。
【时间粒度】以「每 256 帧一条 caption」为标注粒度，即长 clip 会被切成多个时间段分别描述，而非整段一条——这使 caption 具备粗粒度的时序对应关系。
【采样方式】VLM 输入为均匀采样的 8 帧。
【增强流程】开源版支持两段式：VLM 先生成基础 caption，再由 LLM（Qwen-LM）做改写增强，可用于风格统一、补充细节或生成多种长度变体。
【下游用途】caption 会进一步经文本嵌入模型编码（26.04 起语义去重默认改用 vLLM 后端的 google/embeddings-gemma-300m 嵌入模型），文本嵌入与视频嵌入一并写入 WebDataset。
【结构化程度】官方未公布 caption 的模板、槽位定义或结构化字段规范（如是否强制覆盖主体/动作/场景/运镜/光照等），也未公布 prompt 全文。可确定的是运镜类型（pan/zoom/tilt）由独立的运动分类器以标签形式产出，而非依赖 caption 文本——即结构化标签与自然语言描述是两套并行的标注体系。[不确定]

### 音视频联合caption结构（是否同时覆盖视觉+听觉轨道、是否分流为独立字段，如LTX-2全音景描述、Script-a-Video factorized streams、Foley-Omni三字段）

无。这是该工具链面向音视频联合生成场景的核心空白。
视频 captioning 只描述视觉轨道，不涉及任何听觉内容（无音乐/环境音/foley/对白的描述字段）；音频 pipeline 只产出 ASR 转写文本，不与视频画面关联。框架中不存在能同时覆盖视觉与听觉两条轨道的 caption schema，既没有 LTX-2 式的「融合式全音景长描述」，也没有 Script-a-Video 的 factorized streams 或 Foley-Omni 的三字段设计。
26.07 引入的 Nemotron 3 Nano Omni 是一个具备音视频理解能力的全模态模型，理论上具备产出联合 caption 的潜力，但官方文档中它的用途是 OCR 合成数据的质量打分与视频 captioning 后端，未见音视频联合标注的 stage 或示例。若用该框架服务 AV 联合生成模型，joint caption 层需完全自建。

### 对白转写与说话人属性标注（ASR转写+说话人身份/语言/口音/情绪） ⚠️

视频 pipeline 中无对白转写（不处理音轨）。
音频 pipeline 具备部分能力但与视频割裂：(1) ASR 转写——使用 NVIDIA NeMo ASR 模型系列生成转写文本；(2) 转写质量评估——以 WER（词错误率）与 CER（字错误率）作为质量指标并据此过滤（WER-filtering），这是该框架在语音数据侧最明确的质量把控手段；(3) 说话人分离——26.02 起支持 streaming Sortformer 做 speaker diarization 与 VAD（语音活动检测），并有 long-form audio cutting 教程；(4) 26.07 新增音频增强 stage：tagging、SQUIM 客观质量指标、带宽估计（bandwidth estimation）、标点准备（punctuation preparation）、可选的二次 ASR 打分。
【缺失】说话人身份属性标注、语种识别、口音标注、情绪标注均无内置 stage；WER/CER 的默认阈值未在文档中给出。这些能力对多语种唇同步数据构建是必需的，需自建。[不确定]

### 几何与结构化标注（相机参数、深度、3D point tracks、动作标注、显式状态） ⚠️

结构化标注能力有限但存在两处：
【1. 镜头运动标签】运动分类器（ViT + 光流序列）在过滤的同时对镜头运动类型打标：pan（平移）、zoom（推拉）、tilt（俯仰）。这是可直接用于运镜控制训练的显式结构化标签，是该 pipeline 少见的非文本标注产出。
【2. 运动分数】motion_score_global_mean 与 motion_score_per_patch_min_256 两个数值指标随 clip 一并保留，可作为运动强度的连续型条件信号。
【3. 嵌入】Cosmos-Embed1 视频嵌入与 caption 文本嵌入均以 Parquet 存储并打包进 WebDataset，可用于检索、聚类与条件建模。
【缺失】相机内外参估计、深度图、3D point tracks、光流场落盘、物体框/分割掩码、动作类别标签、显式物理状态等均无内置 stage。相较 Cosmos 平台整体（另有 Cosmos Transfer 等涉及深度/分割条件的组件），curation 侧的几何标注是薄弱环节。[不确定]

### 合成数据构造（受控扰动/编辑构造训练对，如InstructAV2AV）

curation 框架本身不做视频合成数据构造，但相关能力在演进：
【视频侧】无受控扰动/编辑构造训练对的 stage（无类似 InstructAV2AV 的编辑配对构造）。Cosmos WFM 的训练数据中包含约 4% 的「合成渲染（synthetically rendered）」视频，但那是外部输入的素材，不是 pipeline 生成的。
【文本/多模态侧】26.04 起集成 NeMo Data Designer（合成数据设计工具）；26.07 新增 Nemotron OCR 合成数据流水线，可从已有数据集生成 OCR 训练记录，并用 Nemotron Nano Omni 做可选质量打分；26.07 还引入 Nemotron-CLIMB 数据配比优化工作流（对文档做嵌入—聚类—配比搜索，从 64 个候选配比中筛选出最终方案）。
可见 NVIDIA 正把「合成数据生成」与「数据配比自动优化」纳入 Curator 版图，但目前仅覆盖文本与 OCR，视频模态尚未跟进。

### 人工介入程度（人工标注、人工质检、模型初筛+人工复核） ⚠️

框架定位为全自动流水线，无内置的人工标注/复核界面或工作流，官方文档未描述任何 human-in-the-loop 环节（无标注平台集成、无抽检采样 stage、无人工仲裁队列）。
人工介入实际发生在框架之外的两处：(1) 判别器的构建——Cosmos WFM 中的文字叠加检测 MLP、视频类型 taxonomy MLP、运动分类 ViT 都需要人工标注的训练集，但标注规模与流程未披露；(2) 阈值的人工设定——美学 3.5、运动 0.00098、DOVER 底部 15%、k=10000 等参数由工程师经验设定，框架只提供默认值。
框架在可观测性上做了替代性设计以降低对人工的依赖：被过滤的 clip 移入 video.filtered_clips 并更新统计、26.02 起自动记录性能指标、提供各 stage 的资源与瓶颈监控面板、Pipeline.describe() 供开发期审查——这些让工程师可以事后审计漏斗而非逐样本人工看。[不确定]

## 音视频对齐

### 音视频同步检测方法（口型同步、事件对齐）

无。框架完全不具备音视频同步检测能力：视频 pipeline 不解析音轨，因此不存在唇同步检测、事件对齐检测或异步样本剔除的任何 stage；音频 pipeline 处理的是独立音频文件，无对应视频帧可供对齐。官方文档、Cosmos WFM 论文、NeMo 视频基础模型训练论文中均未出现 SyncNet、Synchformer、AV-align 等任何同步检测组件。
这构成 NeMo Curator 服务于音视频联合生成模型时的首要缺口——尽管它在「大规模视觉数据处理」上是事实标准，但 AV 数据构建的核心环节（同步性筛选）需完全自建。

### 同步检测具体指标与阈值（SyncNet/Synchformer/LSE/自研，阈值数值，如MOVA LSE-D≤9.5且LSE-C≥4.5、SkyReels-V4 SyncNet |offset|≤3∧conf>1.5）

无。不存在任何同步性指标或阈值（无 SyncNet 置信度/offset、无 LSE-D/LSE-C、无自研同步分），也无相关配置项。可作为对照参考的是该框架已有的、参数完全透明的其他阈值体系：美学 score_threshold = 3.5、运动 motion_score_global_mean = 0.00098、motion_score_per_patch_min_256 = 0.000001、DOVER 剔除底部 15%、clip 时长 2–60 秒、语义去重 k-means k = 10000。这套「默认阈值全部写进文档」的做法本身值得借鉴，但其中不含任何音视频维度。

### 时序同步vs语义同步的分离处理（时间对齐与内容语义匹配作为两个独立过滤条件）

不适用。框架不处理音视频关系，因而不存在时序同步与语义匹配的分离处理。
若强行类比，其视频侧确实存在「结构性指标」与「语义性指标」的分工设计：运动向量分数、DOVER 画质分属结构/信号层判据，由轻量统计与小模型承担；视频类型 taxonomy、文字叠加检测属语义层判据，由嵌入 + MLP 承担；caption 生成属语义描述层，由 VLM 承担。这种「按判据性质分配不同量级模型」的成本分层思路可迁移到 AV 同步过滤的设计中（如先用轻量能量包络相关性粗筛时序对齐，再用大模型做语义匹配复核），但框架本身未提供此类实现。

### 音频质量过滤（SNR、静音检测与静音占比阈值、无音轨剔除、画外音源剔除、背景音乐分离） ⚠️

视频 pipeline 中无（不处理音轨，故不存在无音轨剔除、静音占比阈值、画外音源剔除、背景音乐分离等）。
独立音频 pipeline 中有面向语音数据的质量过滤：(1) WER/CER 过滤——以 ASR 转写错误率为代理指标剔除低质量音频，是主力手段；(2) 音频分析——时长计算（duration-calculation）与格式校验（format-validation）；(3) 26.02 起的 VAD（语音活动检测）可用于定位有效语音段、剔除静音；(4) 26.07 新增 SQUIM 客观质量指标（可无参考地估计 SNR、STOI、PESQ 等语音质量维度）与带宽估计（识别上采样的伪高采样率音频），并支持可选的二次 ASR 打分作为交叉验证。
【缺失】源分离（人声/伴奏分离）、背景音乐检测、音效事件检测、响度归一化等均无内置 stage；各指标的默认阈值文档中未给出。[不确定]

### 语音/音效/音乐的分类与分别处理策略 ⚠️

无分类分流机制。音频 pipeline 的隐含假设是「输入即语音数据」，全部 stage（ASR 转写、WER/CER 过滤、VAD、说话人分离、SQUIM 质量评估、标点准备）均为语音服务，不区分也不分别处理音效（foley）、音乐与环境音。框架未提供音频事件分类器（如 AudioSet 标签体系）、音乐检测器或源分离模块。
26.07 的音频 tagging stage 是最接近的能力，但文档中其定位是语音属性标注而非四类音频的分类分流。[不确定]

## 训练配合

### 多阶段训练课程与数据课程调度（阶段划分依据：分辨率/时长/质量分/模态；低清→高清、图像→视频、短→长）

作为数据基建，它不定义训练课程，但其输出格式是为课程化训练服务的，这是其与训练侧耦合最紧密的设计点：
【分桶即课程接口】sharding 阶段按「分辨率 × 宽高比 × 时长」三维把 clip 打包成不同的 WebDataset 分片，官方明确说明目的是「与训练课程（training curriculum）对齐」——下游训练框架据此实现低清→高清、短→长的渐进式课程，只需切换读取的分片桶即可，无需重新处理数据。
【质量分随数据落盘】美学分、运动分、类型标签等判别结果随 clip 一并保存在 metadata 中，使训练侧可以按质量分动态筛选子集（如退火阶段只取高美学分样本），而不必重跑 pipeline。
【NeMo 训练侧配合】NeMo Framework 的视频基础模型训练栈（arXiv:2503.12964）与 Curator 输出的 WebDataset 直接对接，支持多 GPU 顺序读取 PB 级数据。
各阶段的具体划分依据、切换判据由使用者的训练配置决定，Curator 不做规定。

### 数据配比随训练阶段的变化（预训练/退火annealing/SFT高质量子集） ⚠️

框架不规定配比，但提供了配比控制的两类基础设施：
【1. 静态配比控制（已在视频侧生产验证）】Cosmos WFM 通过 taxonomy 分类器 + 上/下采样实现九大类的显式配比（自然动态 20%、手部操控 16%、空间导航 16%、驾驶 11% 等，详见 domain_distribution）。这是「分类器打标 → 按目标比例重采样」的标准做法。
【2. 自动配比优化（26.07 新增，目前仅文本模态）】Nemotron-CLIMB 工作流：对文档做嵌入 → 聚类 → 在配比空间中搜索优化 → 从 64 个候选混合方案中迭代收敛到最终配比。这是把「数据配比」本身当作可优化对象的系统化尝试，若未来扩展到视频模态将有直接价值，但当前不支持视频。
【缺失】预训练/退火/SFT 各阶段配比如何变化，属于使用者的训练策略，Curator 不涉及也未给出参考方案。[不确定]

### 后训练数据（SFT精选集规模与筛选标准、偏好对数量与标注方式、reward model训练数据） ⚠️

不适用/未覆盖。NeMo Curator 视频侧不提供后训练数据构建能力：无偏好对（preference pair）构造 stage、无 reward model 训练数据生成、无 SFT 精选集的自动筛选工作流。
可间接支撑 SFT 子集筛选的是其 metadata 体系——美学分、运动分、类型标签、去重簇信息随数据落盘，使用者可据此用阈值切出高质量子集（Cosmos WFM 即从约 10^8 预训练 clip 中切出约 10^7 用于微调，但未公布筛选标准）。
文本模态方面，26.04 集成的 NeMo Data Designer 与 26.07 的合成数据流水线部分覆盖了后训练数据生成，但未延伸到视频。[不确定]

### 数据处理基础设施与吞吐（NeMo Curator/Data-Juicer/自研，GPU加速比，处理规模，成本） ⚠️

这是本条目的核心字段，也是该工具链被称为「工业级视频数据基建事实标准」的依据所在。
【总体加速比】官方口径：使用 1000 张 GPU，在 ISO 功耗（等功耗）条件下相较未优化的 CPU pipeline 达到 89 倍加速，把处理时间从「数年」压缩到「数天」。具体到 2000 万小时视频：Hopper 集群约 40 天、Blackwell 约 14 天，而未优化 CPU pipeline 需约 3.4 年。
【加速来源拆解】(1) NVDEC 硬件解码 + NVENC 硬件编码——解码与转码环节获得约 3 倍加速（NeMo 论文口径）；Cosmos 论文给出 L40S 上转码吞吐 0.3702 videos/second，相较基线 6.5 倍提升；(2) VLM captioning 的 FP8 量化 TensorRT-LLM 引擎——约 10 倍加速；(3) Ray 分布式调度与自动扩缩容消除 GPU 空闲；(4) streaming 执行避免中间产物全量落盘。
【自动扩缩容机制（Cosmos-Xenna 核心）】streaming 模式下持续测量每个 stage 的吞吐（samples/second），动态调整各 stage 的 worker 数以平衡整条流水线：缩减快速 stage 的 worker、扩容瓶颈 stage。通过复杂的装箱（bin-packing）算法把 worker 最优地打包到集群节点上以最大化利用率。默认参数：autoscale_interval_s = 180 秒（扩缩容检查间隔）、logging_interval = 60 秒、cpu_allocation_percentage = 0.95、execution_mode 可选 streaming（推荐）/ batch。另有 serving 模式支持基于队列的实时输入输出。
【关键工程结论】captioning 被明确指认为速率瓶颈 stage（吞吐远低于嵌入生成），自动扩缩容因此会把最多资源分配给它——任何自建视频数据 pipeline 都应预期同样的瓶颈分布。
【其他机制】backpressure 反压控制防止内存溢出；CPU 密集 stage 不阻塞 GPU stage；stateful actor 使模型权重每个 worker 只加载一次；SPMD 模式支持需要张量并行的大模型跨 GPU/跨节点分布式推理；P2P artifact 分发加速多节点模型文件下载。
【异构集群】支持 L40S、H100、GB200 等混合 GPU 类型的异构集群，可针对性利用特定 GPU 的 NVENC 能力。
【Ray 对象存储】利用 Ray Object Store 与对象引用最小化数据搬运。
【文本模态对照数字】模糊去重（fuzzy dedup）在 RedPajama v2 上相较 CPU 方案 16 倍加速、TCO 降低 40%，1→4 节点近线性扩展——注意这是文本侧数字，不可与视频侧的 89x 混用。
【运维】26.07 起支持 LMDB 记录已完成分片实现断点续跑，并支持 SLURM job array；26.02 起提供各模态的自动化 stage 与 pipeline 基准测试。
【成本】未公布单位数据的美元成本。[不确定]

## 效果对比

### 数据策略消融的量化影响（区分：过滤严格度ablation / caption密度风格ablation / 数据配比ablation，及对应评测指标） ⚠️

作为工具链本身没有数据策略消融实验，但其上游 Cosmos WFM 论文中有若干可视为「选型消融」的定量对比，是该条目最接近 ablation 的证据：
【1. 镜头切分器选型对比】TransNetV2 vs PySceneDetect vs Panda70M 切分器 vs AutoShot，在 BBC 数据集上以 F1 为指标横评，TransNetV2 以 F1 = 0.967 胜出。这是切分方法选型的直接量化依据。
【2. 转码加速对比】L40S 上 NVDEC/NVENC 硬件路径相较基线 6.5 倍吞吐提升（0.3702 videos/second）；NeMo 论文另给出解码转码环节约 3 倍加速。
【3. captioning 推理加速】FP8 量化 TensorRT-LLM 引擎约 10 倍加速。
【4. 整体 pipeline】1000 GPU ISO 功耗下 89 倍于未优化 CPU pipeline。
【缺失】没有「过滤严格度 → 生成质量」的消融（如美学阈值取 3.5 vs 4.0 对最终模型 FVD/人评的影响未测）；没有 caption 密度/风格的消融；没有九大类配比的消融（配比数字是给定的，未论证为何是 20%/16%/16%…）。即所有量化结论都在「处理效率」维度，无一在「数据策略对模型效果的影响」维度。这是该工具链作为基建的定位使然，但也意味着其默认阈值缺乏效果侧的实证支撑，使用者不应直接照搬。[不确定]

### 质量vs数量的证据（小而精数据超越大而杂的案例） ⚠️

没有直接的「小而精超越大而杂」对照实验。但整条 pipeline 的设计哲学本身即是该主张的工程化表达，且有明确的量化落点：从 2000 万小时原始视频出发，经镜头切分（丢弃 < 2 秒）、运动过滤（剔除静止与失控抖动）、画质过滤（DOVER 剔除底部 15%）、美学过滤（阈值 3.5，且默认 min 聚合的严格策略）、文字叠加过滤、类型 taxonomy 过滤（整体剔除动画/游戏/抽象图案）、语义去重（移除约 30%）七道关卡，最终仅约 10^7 量级 clip 进入微调阶段。
Cosmos 论文对 curation 目标的表述本身即是质量优先的宣言：目的是「定位视频中具有丰富动态与高视觉质量、有利于学习视觉内容所编码的物理规律的片段」。此外「上采样人类动作/物体操控、下采样风景」的策略也是典型的「信息密度优先于素材数量」判断。但这些都是设计意图与经验判断，未提供 A/B 对照的量化证据。[不确定]

### 训练数据domain分布与评测基准类目体系的对齐关系（如VABench七大类） ⚠️

训练数据 domain 分布与评测体系的对齐关系未被显式论证，但其 taxonomy 的设计动机明确指向下游能力维度：
Cosmos 的九大类（自然动态 20%、手部操控 16%、空间导航 16%、驾驶 11%、人体动作 10%、第一人称 POV 8%、动态镜头 8%、其他 7%、合成渲染 4%）是围绕 Physical AI 的目标应用场景组织的——机器人（手部操控、人体动作）、自动驾驶（驾驶）、具身导航（空间导航、第一人称 POV）——即数据配比直接映射到应用领域而非评测基准类目。
【与本调研的对照】这套体系与 VABench 七大类等面向通用文生视频/音视频生成的评测类目体系不可直接对应：Cosmos taxonomy 明确排除动画、游戏、抽象风格（而这些恰是通用视频生成评测的重要维度），也不包含人物特写/对白/影视感等音视频生成关注的类目，更完全没有音频维度。
【结论】NeMo Curator 提供的是「taxonomy 驱动配比控制」这一可复用的机制，而其内置/示例的类目体系是为世界模型定制的，服务音视频生成模型时需重新设计类目定义并自训分类器。官方未提供任何评测基准对齐性的论证。[不确定]

## 其他信息

### summary_note

【定位判断】NeMo Curator + Cosmos-Xenna 是当前公开可得的、唯一在 PB 级规模上被生产验证过的开源视频数据处理基建，其不可替代的价值在三处：(1) Ray + Cosmos-Xenna 的 streaming 自动扩缩容架构，解决了多 stage 异构算力流水线的负载均衡问题（关键洞察：captioning 恒为瓶颈）；(2) NVDEC/NVENC 硬件编解码与 FP8/NVFP4 量化推理的端到端 GPU 化，构成 89x 加速的实质来源；(3) 把 sharding/WebDataset 按「分辨率×宽高比×时长」分桶纳入 curation 范畴，使数据处理与训练课程形成闭环。此外其默认阈值全部写进文档（美学 3.5、运动 0.00098、DOVER 底部 15%、clip 2–60 秒、k-means k=10000）与镜头切分器的 F1 横评选型，是同类资料中透明度最高的。
【对音视频生成数据的适用性】需明确其边界：该框架的四模态是「四条互不相通的流水线」，视频侧完全不解析音轨，音频侧只服务 ASR 语音数据。音视频联合场景所需的音轨抽取、AV 同步检测（SyncNet 类）、音频类别分类（语音/foley/音乐/环境音）、联合 caption schema、音频质量阈值体系，全部缺失。因此现实的用法是：把 NeMo Curator 作为视觉侧的骨架与分布式执行底座，在其 ProcessingStage 抽象上自行插入音频相关的 stage——框架的可插拔 stage 设计与 26.04 起内置的 Ray Serve/vLLM 推理服务器使这种扩展在工程上可行，这也是它作为「基建」而非「方案」的正确定位。
【版本演进主线】25.09 完成 Dask→Ray 的架构重构并引入视频/音频；26.02 补齐工程化能力（YAML 声明式 pipeline、全模态基准测试、移除 InternVideo2）；26.04 转向大模型友好（vLLM 默认、Ray Serve 推理服务器、Nemotron VLM captioning 后端、Cosmos-Xenna 0.2.0）；26.07 走向生产运维与自动化数据科学（LMDB 断点续跑、SLURM job array、音频增强 stage、Nemotron-CLIMB 自动配比优化、OCR 合成数据）。趋势清晰：判别职责正从浅层打分器（CLIP 美学、MLP 探针）向量化后的大模型迁移，配比决策正从人工设定向自动搜索迁移——但两条趋势目前都只在文本模态落地，视频模态尚未跟进。

## 不确定字段

以下字段的调研信息部分不确定（⚠️ 标注来源）：

- data_sources
- provenance_licensing
- duration_distribution
- resolution_aspect_distribution
- narrative_structure_distribution
- language_accent_distribution
- funnel_retention_rate
- deduplication
- safety_filtering
- caption_structure
- dialogue_transcription_attributes
- geometric_structured_annotation
- human_in_loop
- audio_quality_filtering
- audio_type_handling
- stage_data_mixture
- post_training_data
- data_infra_throughput
- data_ablation
- quality_over_quantity_evidence
- benchmark_taxonomy_alignment
