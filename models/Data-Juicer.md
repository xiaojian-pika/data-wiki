# Data-Juicer 2.0（含 Data-Juicer Sandbox 沙盒实验室）。定位为「面向基础模型、并借助基础模型」的一站式数据处理系统与云规模自适应算子库，论文全称《Data-Juicer 2.0: Cloud-Scale Adaptive Data Processing for and with Foundation Models》。配套的 Sandbox 组件论文全称《Data-Juicer Sandbox: A Feedback-Driven Suite for Multimodal Data-Model Co-development》，是把「数据配方」与「模型训练/评测」闭环起来的数据-模型协同开发中间件。在本次视频生成数据处理调研中，Data-Juicer 的角色不是生成模型，而是被多个团队复用的底层数据清洗/标注算子基础设施。

> 主题: 视频生成模型（含音视频同时生成）的数据处理：数据清洗流程、数据分布、打标方式

[← 返回首页](../index.md)

**目录**: [基本信息](#基本信息) · [数据规模与分布](#数据规模与分布) · [清洗流程](#清洗流程) · [打标方式](#打标方式) · [音视频对齐](#音视频对齐) · [训练配合](#训练配合) · [效果对比](#效果对比)

## 基本信息

### 名称

Data-Juicer 2.0（含 Data-Juicer Sandbox 沙盒实验室）。定位为「面向基础模型、并借助基础模型」的一站式数据处理系统与云规模自适应算子库，论文全称《Data-Juicer 2.0: Cloud-Scale Adaptive Data Processing for and with Foundation Models》。配套的 Sandbox 组件论文全称《Data-Juicer Sandbox: A Feedback-Driven Suite for Multimodal Data-Model Co-development》，是把「数据配方」与「模型训练/评测」闭环起来的数据-模型协同开发中间件。在本次视频生成数据处理调研中，Data-Juicer 的角色不是生成模型，而是被多个团队复用的底层数据清洗/标注算子基础设施。

### 发布机构/公司

阿里巴巴集团。发起团队为阿里巴巴通义实验室（Alibaba Tongyi Lab）数据方向团队，与阿里云 PAI（Platform for AI）联合共建，并与 Anyscale（Ray 团队）、NVIDIA、中山大学等外部单位协作。核心作者：Daoyuan Chen（陈道源，一作/通讯，主页 yxdyc.github.io）、Yilun Huang、Xuchen Pan、Nana Jiang、Haibin Wang、Yilei Zhang、Ce Ge、Yushuo Chen、Wenhao Zhang、Zhijian Ma、Jun Huang、Wei Lin、Yaliang Li（李雅亮）、Bolin Ding（丁博麟）、Jingren Zhou（周靖人）。Sandbox 论文作者为 Daoyuan Chen、Haibin Wang、Yilun Huang、Ce Ge、Yaliang Li、Bolin Ding、Jingren Zhou。GitHub 组织已从 modelscope/data-juicer 迁移至 datajuicer/data-juicer。

### 发布时间（技术报告/论文/开源时间） ⚠️

系列时间线：
- 2023年：Data-Juicer 1.0 首次开源，论文发表于 SIGMOD 2024（约50个纯文本预训练算子）。
- 2024年7月16日：Data-Juicer Sandbox 论文提交 arXiv（arXiv:2407.11784 v1），最新 v3 为2025年6月4日，录用于 ICML 2025 Spotlight。同期在 VBench 文生视频榜单登顶并由阿里云开发者社区/魔搭社区发文宣传。
- 2025年1月（arXiv 编号 2501.14755，提交历史显示 v1 为2024年12月23日、v2 为2025年6月4日、v3 为2025年10月29日）：Data-Juicer 2.0 论文发布，录用于 NeurIPS 2025 Spotlight。
- 持续迭代至调研时点：v1.4.4（2025-12-01，NeurIPS Spotlight 同期，新增6个视频/多模态算子、S3 I/O）、v1.4.5（2026-01-15，20+新算子、Ray vLLM 流水线、具身智能算子）、v1.4.6（2026-02-02，Q&A Copilot、视频字节流 I/O）、v1.5.0（2026-02-12，分区 Ray 执行器、具身智能视频处理、算子级环境管理）、v1.5.1（2026-03-17）、v1.5.2（2026-05-29）、v1.5.3（2026-06-26，VLA 具身算子、Ray repartition 流水线）、v1.5.4（2026-07-17，新增9个人物中心视频理解算子、batch-local stage fusion 加速）。
[不确定] Data-Juicer 2.0 论文 arXiv 首次提交日期在不同来源上呈现为2024年12月23日与2025年1月两种说法，以 arXiv 编号 2501 推断正式挂出应为2025年1月。

### 类型（模型/数据集/工具链/评测基准）

工具链（数据处理系统 / 算子库 / 数据-模型协同开发平台），不是生成模型也不是数据集。可细分为三个产出层次：
1. 【算子库与执行引擎】Data-Juicer 2.0 本体——截至 v1.5.4 共 229 个算子（Mapper 138、Filter 58、Deduplicator 10、Formatter 8、Selector 5、Aggregator 4、Grouper 3、Pipeline 3），覆盖文本/图像/音频/视频/多模态五类模态，配套 Ray、MaxCompute、单机等多种自适应执行后端。
2. 【数据-模型协同开发套件】Data-Juicer Sandbox——提供「探测-分析-精炼」（Probe-Analyze-Refine）工作流，把数据配方搜索与模型训练/评测串成闭环，内置对 EasyAnimate、T2V-Turbo、ModelScope、VBench 等训练与评测框架的接入。
3. 【衍生数据集与配方】以 Sandbox 在文生视频上的实践为例，开源了经筛选的 T2V 最优数据池（datajuicer/data-juicer-t2v-optimal-data-pool 等 HuggingFace 数据集）与对应 YAML 配方，属于「方法+工具+数据配方」三位一体的产出。
因此在本调研的谱系中，Data-Juicer 属于「数据基础设施」类目，与 NVIDIA NeMo Curator 是最直接的对标关系。

### 开源程度（权重/代码/数据/pipeline各自是否开源）

开源程度极高，是本调研中开放度最高的一类对象。
【代码】完全开源。GitHub datajuicer/data-juicer（原 modelscope/data-juicer），Apache 2.0 许可证，约 6.8k stars，持续高频迭代（2026年内已发布 v1.4.5 至 v1.5.4 共7个版本）。229个算子的实现全部可读可改，v1.5.3 一次性新增409个测试用例，工程规范度较高。
【算子与配方】开源。config_all.yaml 暴露全部算子及其超参；DJ-Cookbook 维护20+条即用型数据配方（含视频数据合成、对比学习、课程学习等垂直场景配方）。
【数据】部分开源。文生视频案例中筛选得到的最优数据池以 HuggingFace 数据集形式公开：datajuicer/data-juicer-t2v-optimal-data-pool 共 147,176 条样本、约 227.5GB，Apache 2.0 许可；但底层原始素材（InternVid、Panda-70M、MSR-VTT）需按各自原始协议自行获取，DJ 公开的是筛选后的样本索引与元数据。
【模型权重】Sandbox 论文声明所训练的 T2V 模型（基于 T2V-Turbo/EasyAnimate 微调）连同代码、数据一并开源。
【pipeline】完全公开。不仅公开方法，还公开可执行的 YAML 配方与阈值数值（如 CLIP 相似度阈值 0.306337），复现门槛低于绝大多数模型侧工作。
【文档】中英双语文档站（datajuicer.github.io），含算子提要（Operator Schemas）逐条说明。

### 是否支持音视频同时生成，以及实现方式（原生联合/级联/MoE融合）

不支持音视频同时生成——Data-Juicer 本身不是生成模型，不产出任何音频或视频内容。但它在数据侧提供了支撑音视频联合任务的算子能力，与本调研的 AV 主线有实质关联：
【音频侧算子】audio_duration_filter（时长过滤）、audio_nmf_snr_filter（基于 NMF 的信噪比过滤）、audio_size_filter（文件大小过滤）、audio_add_gaussian_noise_mapper（加噪增广）、audio_ffmpeg_wrapped_mapper（FFmpeg 音频滤镜封装）。
【音视频跨模态算子】video_audio_ASR_mapper（从音轨做语音识别/打标）、video_tagging_from_audio_mapper（基于 Audio Spectrogram Transformer 从音轨生成视频标签）、video_captioning_from_audio_mapper（基于 Qwen-Audio 根据音轨为视频生成 caption）、video_audio_detect_age_gender_mapper（基于 wav2vec2 从音轨检测说话人年龄与性别）、video_audio_speech_emotion_mapper（语音情绪识别）、video_active_speaker_detect_mapper（联合视觉人脸轨迹与音频信号做主动说话人检测——这是 DJ 中最接近「音视频同步判定」的算子）。
【定位判断】DJ 的音视频能力偏向「理解与标注」而非「同步质量把关」：它有主动说话人检测，但没有 SyncNet/Synchformer 式的同步偏移量与置信度打分算子，也没有 speech/foley/music 三类音轨分离与配比控制算子。因此若要复用 DJ 构建 AV 联合生成训练集，同步过滤与音轨分类环节仍需自行扩展算子。
【实证案例】其官方文生视频案例（VBench 登顶）为纯文生视频（T2V），全流程未涉及音频处理。

### 调研信息来源列表（论文/技术报告/官方文档/新闻的URL，标注每条来源的性质：官方一手/同团队旁证/第三方报道）

1. 【官方一手】Data-Juicer 2.0 论文 arXiv 摘要页 https://arxiv.org/abs/2501.14755 —— 作者、版本历史（v1 2024-12-23 / v2 2025-06-04 / v3 2025-10-29）、NeurIPS 2025 Spotlight。
2. 【官方一手】Data-Juicer 2.0 全文 HTML https://arxiv.org/html/2501.14755v3 —— 三层架构、150+ 多模态算子分类、TB级/10k+ CPU核规模、MinHash Ray 去重性能、加速比数据、阿里云 PAI 落地。
3. 【官方一手】Data-Juicer 2.0 PDF https://arxiv.org/pdf/2501.14755
4. 【官方一手】Data-Juicer Sandbox 论文 arXiv 摘要页 https://arxiv.org/abs/2407.11784 —— ICML 2025 Spotlight、Probe-Analyze-Refine、VBench 登顶。
5. 【官方一手】Sandbox 全文 HTML https://arxiv.org/html/2407.11784v3 —— 文生视频案例、数据池切分、算子选择、VBench 分数表、质量vs算力结论。
6. 【官方一手】GitHub 主仓库 https://github.com/datajuicer/data-juicer —— 版本时间线、算子统计、采用方名单、关联论文列表、Apache 2.0 许可。
7. 【官方一手】算子提要文档 https://datajuicer.github.io/data-juicer/en/main/docs/Operators.html 与 https://raw.githubusercontent.com/datajuicer/data-juicer/main/docs/Operators.md —— 229个算子的逐条名称与说明，视频/音频算子清单的权威来源。
8. 【官方一手】HuggingFace 数据集 https://huggingface.co/datasets/datajuicer/data-juicer-t2v-optimal-data-pool —— 147,176条样本、12.09%保留率、227.5GB、来源数据集构成（InternVid 606k / Panda-70M 605k / MSR-VTT 6k）、两个过滤算子及 CLIP 阈值 0.306337。
9. 【官方一手】OpenReview 评审页 https://openreview.net/forum?id=NiL5U1DrRN （DJ 2.0）与 https://openreview.net/forum?id=zIGIvysR1H （Sandbox）。
10. 【同团队旁证】阿里云开发者社区文章《VBench 视频生成新榜首！Data-Juicer 沙盒实验室助力多模态数据与模型协同开发》 https://developer.aliyun.com/article/1570605 及魔搭社区同文 https://community.modelscope.cn/669f1a7b76e87a79e35ada49.html —— 中文版案例说明。
11. 【同团队旁证】HumanVBench 论文 https://arxiv.org/html/2412.17574v2 （CVPR 2026）—— 使用 DJ 20+算子构建人物中心视频标注pipeline，是 DJ 视频算子在真实数据集构建中的用例。
12. 【官方一手】ICML 2025 Poster 页 https://icml.cc/virtual/2025/poster/43484 与 PMLR 正式版 https://proceedings.mlr.press/v267/chen25bm.html
13. 【第三方索引】ResearchGate https://www.researchgate.net/publication/388421863_Data-Juicer_20_Cloud-Scale_Adaptive_Data_Processing_for_Foundation_Models
14. 【第三方报道】CSDN 博客《Data-Juicer：阿里巴巴荣誉出品的大模型数据清洗框架》 https://blog.csdn.net/qq_41895747/article/details/140150556 —— 中文社区解读，无新增一手数据。

## 数据规模与分布

### 训练数据量级（视频条数/小时数/token数，预训练与SFT分开）

Data-Juicer 本身不训练模型，因此无「训练数据量级」；相关量级分两个口径。
【系统处理吞吐口径】Data-Juicer 2.0 论文报告可处理 TB 级数据、调度 10k+ CPU 核心；实验覆盖从 56万（560K）样本到 700亿（70B）样本的数据集规模区间；一个生产部署已稳定运行5个月以上、累计处理规模超过 TB 级。分布式测评中在 3200 Ray-DLC 核心上处理 500 倍数据集耗时 1780 秒、2500 倍数据集耗时 7083 秒。
【文生视频案例口径】Sandbox 的 T2V 案例是本调研最直接相关的数据。原始候选池由三个公开数据集构成，合计约 121.7 万条视频-文本对：InternVid 606k + Panda-70M 605k + MSR-VTT 6k。经筛选后开源的最优数据池为 147,176 条（约 227.5GB），另有一个 228k 条的更大数据池用于登顶 VBench 的最终模型（论文记该配置对应 640k 训练样本量，即含数据重复）。小规模探测实验统一使用约 40k 样本的数据池以控制变量。
【无预训练/SFT之分】作为工具链，不存在预训练与 SFT 数据量的划分。

### 数据来源构成（自有/公开数据集/网络爬取/授权采购/合成数据）

作为工具，Data-Juicer 本身不持有数据，其数据来源问题体现在两处。
【官方案例的数据来源】文生视频案例完全使用公开数据集，无自有数据、无网络爬取、无授权采购、无合成数据：InternVid（上海AI Lab，YouTube来源大规模视频-文本数据集，贡献 606k）、Panda-70M（Snap/UCM，HD-VILA-100M 衍生的高质量视频-caption 数据集，贡献 605k）、MSR-VTT（微软经典小规模视频描述数据集，贡献 6k）。三者合计约 121.7 万条，可见配比上 InternVid 与 Panda-70M 近乎各半、MSR-VTT 仅作少量补充（0.5%）。
【系统支持的数据接入面】原生兼容 HuggingFace Datasets、ModelScope 数据集、本地文件系统、阿里云 OSS/NAS/CPFS、AWS S3（v1.4.4 起）等多种数据源；支持压缩数据集格式（v1.5.1）与视频字节流 I/O（v1.4.6，便于不落盘处理视频）。
【合成数据能力】229个算子中约50个专门用于数据合成与增广，DJ-Cookbook 内含「视频数据合成」（video-data-synthesis）YAML 配方，即 DJ 不仅能筛选真实数据，也支持构造合成数据——这与 NeMo Curator 侧重清洗的定位有所差异。

### 数据合规与溯源（授权数据占比、rights-cleared数据集、C2PA等） ⚠️

【工具自身合规】Apache 2.0 许可，商用友好。公开的 T2V 数据池同样标注 Apache 2.0，但实际发布的是筛选后的样本元数据/索引，原始视频素材仍受 InternVid、Panda-70M、MSR-VTT 各自协议约束——这是学术视频数据集规避版权再分发的通行做法。
【面向合规的算子能力】DJ 提供若干可用于隐私与合规处理的算子：video_face_blur_mapper（检测并模糊视频中的人脸，直接服务于人脸隐私脱敏）、video_nsfw_filter（不良内容过滤）、video_watermark_filter 与 video_remove_watermark_mapper（水印检测与去除，间接关联版权标识处理）、文本侧另有敏感信息脱敏类算子。这使 DJ 成为少数在数据处理框架层面内置隐私脱敏算子的开源系统。
[不确定] 论文与文档未提及 C2PA 等内容溯源/来源认证标准的支持，未提供数据授权状态追踪、rights-cleared 数据集标记、许可证兼容性检查等治理型功能，也未披露阿里内部使用时的授权数据占比。整体上 DJ 提供的是「合规处理的工具」，而非「合规治理的框架」。

### 片段时长分布与切分策略 ⚠️

【切分算子——DJ 的强项之一】提供三种互补的视频切分算子，可按需组合：
  · video_split_by_scene_mapper —— 基于场景变化检测把视频切成镜头级 clip（底层为 PySceneDetect 系方案），是最贴合视频生成训练数据构建的切分方式。
  · video_split_by_duration_mapper —— 按固定时长切分为等长片段，适合快速构建定长训练样本。
  · video_split_by_key_frame_mapper —— 按关键帧（I帧）边界切分，切点与编码结构对齐，解码开销低。
  另有 video_clip_reassembly_mapper 用于把重叠片段的处理结果重新组装回长视频（面向具身智能手部动作场景）。
【时长过滤】video_duration_filter 支持设定时长上下界区间，剔除过短（信息量不足）与过长（切分不干净）的片段。
[不确定] 官方 T2V 案例未披露最终数据池的片段时长分布直方图或平均时长；InternVid/Panda-70M/MSR-VTT 本身已是预切好的短片段数据集（Panda-70M 平均约8.5秒、MSR-VTT 约10-30秒、InternVid 平均约11秒），案例中未额外做二次切分，因此切分算子在该案例中未被实际启用。DJ 也未提供官方推荐的时长分桶策略。

### 分辨率/宽高比分布与分桶策略 ⚠️

【相关算子】提供过滤与改写两类算子，覆盖较完整：
  · video_resolution_filter —— 按分辨率上下界过滤，剔除低清素材。
  · video_aspect_ratio_filter —— 按宽高比区间过滤，剔除极端长条/竖屏或异常比例样本。
  · video_resize_resolution_mapper —— 按宽高约束改写分辨率（而非丢弃），可用于把混合分辨率数据统一到目标规格。
  · video_resize_aspect_ratio_mapper —— 把视频重采样到指定宽高比区间内。
  「过滤 + 改写」双路设计是 DJ 相对纯过滤型框架的特点：不合规样本可以被修复而非直接丢弃，对数据量宝贵的场景更友好。
[不确定] DJ 未内置视频生成训练常用的「分辨率分桶」（resolution bucketing）功能——即按分辨率/宽高比把样本分组以便同批次训练，这属于训练框架侧（如 EasyAnimate、Diffusers）的职责，DJ 只负责把数据整理到可分桶的状态。官方 T2V 案例也未披露数据池的分辨率与宽高比分布统计。

### 类别/domain分布与配比策略（人物、动作、场景、风格等比例控制与概念均衡） ⚠️

Data-Juicer 在类别/domain 配比上的思路与生成模型团队根本不同：它不预设一套 domain 类目体系，而是提供「让 domain 分布可度量、可切分、可搜索」的机制，把配比决策交给 Sandbox 的反馈闭环去经验性确定。
【domain 标注能力】
  · video_tagging_from_frames_mapper —— 从抽帧生成视频内容标签（基于 RAM 等图像打标模型），可产出开放词表的语义标签，是构建 domain 分布画像的主要手段。
  · video_tagging_from_audio_mapper —— 基于 Audio Spectrogram Transformer 从音轨生成标签，提供听觉侧的 domain 信息（与视觉标签互补）。
  · video_tagging_from_frames_filter —— 按标签保留/剔除样本，即基于 domain 的定向筛选。
  · video_object_segmenting_mapper —— 基于 YOLOE + SAM2 的文本引导语义分割，可给出物体级的 domain 证据。
【人物中心 domain 的专项支持】v1.5.4 一次性新增9个人物中心视频理解算子，构成一条较完整的人物 domain 处理链：video_human_tracks_extraction_mapper（提取人脸与人体框轨迹）、video_face_ratio_filter（人脸占画面比例过滤，可用于筛选特写/中景/远景）、video_active_speaker_detect_mapper（主动说话人检测）、video_captioning_from_human_tracks_mapper（基于人物轨迹生成描述）、video_captioning_face_attribute_emotion_mapper（人脸属性与情绪描述）、video_human_tracks_face_demographic_mapper（人口统计属性）、video_whole_body_pose_estimation_mapper（身体/手/脚/面部2D全身关键点）。这条链直接对应视频生成中「人物、动作」这一最重要的 domain。
【配比策略——经验搜索而非先验设计】Sandbox 的 Probe-Analyze-Refine 工作流本质上是一种数据配比的自动搜索：对每个候选算子 i，按其统计量把数据池均分为低/中/高三个等大子池 P_i,low / P_i,middle / P_i,high，外加一个随机采样的对照池，分别训练参考模型并用 VBench 评测，据此对算子及其取值区间排序；再把排名靠前的若干算子组合成 2^n−1 个交叉数据池（金字塔结构，越上层质量越高但样本越少），继续搜索最优组合。这套流程把「哪个维度上该保留哪一段分布」变成了可被模型反馈直接回答的实验问题，而不是靠人工设定比例。
【DJ-Cookbook 中的配比相关配方】维护有「基于数据难度的课程学习」（curriculum learning）配方与对比学习配方，属于 domain/难度配比调度的现成模板。
[不确定] DJ 官方未发布任何视频生成的推荐 domain 类目体系（如人物/动作/场景/风格的目标比例），也未在 T2V 案例中报告最终数据池的类别分布统计；概念均衡（concept balancing）、长尾类别重采样等策略在文档中未见专门算子支持。

### 音频类别分布与配比（语音/音效foley/音乐/环境音/静音的比例及控制策略）——AV模型独有维度 ⚠️

[不确定] Data-Juicer 未提供音频类别（语音/音效foley/音乐/环境音/静音）的分类与配比控制能力，这是其相对于 AV 生成数据 pipeline 的明显缺口。
【现有的音频侧能力仅限于】
  · 质量维度：audio_nmf_snr_filter（基于非负矩阵分解估计信噪比并按区间过滤）、audio_duration_filter（时长）、audio_size_filter（文件体积）。
  · 语义标注维度：video_tagging_from_audio_mapper 与 video_audio_ASR_mapper 均基于 Audio Spectrogram Transformer（AST，在 AudioSet 上训练）产出音频事件标签——AudioSet 本体包含语音/音乐/环境音等527类，理论上可据此近似区分三类音频，但 DJ 未把它封装为显式的三分类字段或配比控制算子。
  · 说话人属性维度：video_audio_detect_age_gender_mapper（wav2vec2 检测年龄性别）、video_audio_speech_emotion_mapper（语音情绪）。
【缺失的关键能力】无音轨源分离算子（如 Demucs/Bandit 式的 speech/effects/music 三分离），因此无法做 Foley-Omni 那样的字段级能量门控与音轨类别配比；无静音检测与静音占比阈值算子；无「无音轨样本剔除」的专用算子（需借 audio_duration/size filter 间接实现）；无音乐/人声分离用于背景音乐剥离。
【实践佐证】其唯一的官方视频案例（VBench T2V）为纯视觉任务，全程未启用任何音频算子，说明音频侧能力尚未经过大规模视频生成场景的实战检验。若要用 DJ 构建音视频联合生成的训练数据，音频类别体系需自行以自定义算子扩展——好在 DJ 的算子接口设计（Mapper/Filter 基类 + YAML 注册）使扩展成本较低，这也是团队宣称的可编程性（programmability）卖点之一。

### 叙事结构分布（单镜头vs多镜头、平均clip时长、镜头数分布、是否含原生音轨） ⚠️

【单镜头 vs 多镜头的处理能力】DJ 通过 video_split_by_scene_mapper（场景变化检测切分）提供了把多镜头长视频拆解为单镜头 clip 的标准手段，这是控制叙事结构最直接的杠杆——启用该算子即意味着训练数据全部为单镜头；不启用则保留原始的多镜头结构。DJ 把这一选择开放给使用者，不做默认取向。
【官方案例的实际结构】T2V 案例的三个源数据集（InternVid、Panda-70M、MSR-VTT）本身均已是切好的单镜头或近单镜头短片段，案例中未再做场景切分，因此最终数据池以单镜头短 clip 为主。
【是否含原生音轨】案例未做要求也未做统计；DJ 支持处理带音轨与不带音轨的视频，video_split_by_scene_mapper 等切分算子会同步处理音轨。
[不确定] 未报告任何镜头数分布、平均 clip 时长、含音轨样本占比的统计；DJ 也未提供「镜头数统计」或「叙事结构画像」类的分析算子（其数据分析报告主要围绕单值统计量的直方图，而非结构化叙事属性）。

### 语言/口音分布（多语种唇同步能力的数据基础） ⚠️

[不确定] Data-Juicer 未提供针对视频/音频的语言与口音分布分析或控制能力，官方 T2V 案例也未涉及语言维度。
【间接相关的能力】
  · 文本侧：DJ 1.0 起即内置 language_id_score_filter（基于 fastText 的语种识别与置信度过滤），可对 caption 文本做语种筛选，但这是文本语种而非语音语种。
  · 语音侧：video_audio_ASR_mapper 可对音轨做语音识别，其底层 ASR 模型的多语种能力决定了可转写的语言范围，但 DJ 未把「识别出的语言」封装为可过滤的结构化字段，也无口音标注算子。
  · 说话人属性：现有算子覆盖年龄、性别、情绪，唯独不覆盖语言与口音。
【评价】对多语种唇同步这一 AV 生成的关键能力，DJ 目前不提供数据侧支撑，需自行扩展。这与其算子演进路径有关——2026年的新增算子集中在具身智能（相机标定、位姿、手部重建）与人物中心视频理解两个方向，语音语言维度尚未进入优先级。

## 清洗流程

### 清洗漏斗整体结构（几级过滤、各级顺序）

Data-Juicer 的清洗漏斗不是一条固定流水线，而是「可配置的算子有向图 + 由模型反馈搜索出的最优算子组合」。这是它与 Apollo、UniVerse-1 等模型团队自建 pipeline 的本质区别。
【结构层面——三层系统架构（DJ 2.0）】
  1. 能力层（Capability Layer）：把 DJ 1.0 的约50个纯文本预训练算子扩展到 150+（截至 v1.5.4 已达229个）多模态算子，覆盖文本/图像/音频/视频/多模态，并支持后训练任务。约90%的算子为 Mapper 与 Filter 两类，约75%涉及多模态处理，约50个用于数据合成与增广。
  2. 接口层（Interface Layer）：多级 API——底层 Python API、RESTful 端点、可视化编辑器（阿里云 PAI Designer 组件）、以及基于 AgentScope 智能体的自然语言交互（用对话指令下发数据处理任务）。
  3. 运行时层（Runtime Layer）：统一的 Data-Juicer-Dataset 抽象、算子解耦与自适应优化、容错执行控制、硬件无关适配器（单机 / Ray / 阿里云 MaxCompute / PAI-DLC 多后端自动切换）。
【流程层面——Sandbox 的 Probe-Analyze-Refine 工作流】这是 DJ 用于「设计漏斗」的方法论，本身构成一条元流程：
  · Probe（探测）：对每个候选算子，按其统计量把数据集均分为低/中/高三个等大数据池，再加一个随机采样对照池；在每个池上用相同预算训练一个参考模型（小规模、低成本）。
  · Analyze（分析）：用统一评测基准（视频侧为 VBench 的16项指标）比较各池训练出的模型表现，据此对算子及其取值区间排序，识别出真正带来增益的算子。
  · Refine（精炼）：把排名靠前的 n 个算子交叉组合成 2^n−1 个数据池，形成金字塔结构（越上层同时满足的算子条件越多、质量越高但样本越少）；再在大规模上验证最优组合，并对比「高质量数据重复」与「逐层下探纳入次优数据 + 去重」两种扩量路线。
【Sandbox 自身也是三层设计】顶层为 YAML 配置驱动的工作流编排（probe/refine/execute/evaluate 四个阶段）；中层为提供通用开发行为的 Hook 函数；底层为暴露100+算子及训练/评测能力的工厂类。
【文生视频案例中最终采用的漏斗】经上述搜索后确定为极简的两级过滤：video_nsfw_filter（NSFW 打分过滤，底层模型 Falconsai/nsfw_image_detection）→ video_frames_text_similarity_filter（CLIP 计算抽帧与文本的相似度，最小阈值 0.306337）。从约121.7万条筛至147,176条。值得注意的是这个最终配方远比多数模型团队的十几级漏斗简单——Sandbox 的搜索结论是「少数几个真正相关的算子 + 严格阈值」优于「堆砌大量过滤条件」。
【工程优化——漏斗顺序的自动重排】DJ 2.0 引入 workload-aware OP reordering：在满足算子可交换性约束的前提下，自动把计算开销小的算子排在前面，最多可减少 70.22% 的处理时间。这把「过滤器排序」从人工经验变成了系统自动优化，是本调研中少见的能力。

### 漏斗定量保留率（每级过滤的输入/输出量与最终保留率，如Apollo 27%） ⚠️

【官方唯一公开的定量保留率——文生视频案例】
  · 输入：InternVid 606k + Panda-70M 605k + MSR-VTT 6k ≈ 1,217,000 条视频-文本对。
  · 输出：147,176 条，约 227.5GB。
  · 端到端保留率：12.09%（HuggingFace 数据集卡片明确标注）。
  这个数字与 Apollo 的 27%、UniVerse-1 的约19% 同处一个量级但更严格，说明 Sandbox 搜索出的最优策略倾向于「宁缺毋滥」——仅保留约1/8的数据。
  · 另有一个 228k 条的数据池用于最终登顶 VBench 的模型（保留率约18.7%），论文记该配置对应640k训练样本量，即在228k基础上做了约2.8倍的数据重复。
【逐级拆解】[不确定] 未公开两个算子各自的淘汰量：即 video_nsfw_filter 单独淘汰多少、video_frames_text_similarity_filter 单独淘汰多少。从算子性质推断，NSFW 过滤在这三个已经过初步清洗的公开数据集上淘汰率应较低（可能个位数百分比），12.09% 的保留率主要由 CLIP 相似度阈值 0.306337 贡献——即绝大多数样本因「视频画面与文本描述不够对齐」被剔除。这一推断与阈值数值本身相符：0.306 对 CLIP 相似度而言属于偏高的门槛。
【探测阶段的池划分口径】Probe 阶段各单算子池按低/中/高三等分，即每池保留率固定为33.3%；这是为了公平比较而设的等大约束，不代表实际推荐保留率。
【系统级吞吐口径的"保留率"】DJ 2.0 论文侧重处理效率而非保留率，未给出跨项目的通用保留率统计。

### 镜头切分方法（PySceneDetect/自研模型/shot-aware splitting）

镜头切分是 DJ 视频算子中支持较完整的环节，提供三种策略供选择：
  · video_split_by_scene_mapper —— 「基于检测到的场景变化把视频切成场景 clip」，即标准的 shot boundary detection 切分，底层为 scenedetect（PySceneDetect）库，支持其 ContentDetector / ThresholdDetector / AdaptiveDetector 等检测器与阈值配置。这是与业界主流（Apollo、Movie Gen、Open-Sora 等普遍使用 PySceneDetect）完全一致的技术路线。
  · video_split_by_duration_mapper —— 按固定时长机械切分，不感知镜头边界，适合已知素材为单镜头长视频或对切点不敏感的场景。
  · video_split_by_key_frame_mapper —— 按视频编码的关键帧（I帧）切分。优点是切点与编码结构对齐、无需重编码、速度极快；缺点是 I帧位置由编码器决定，未必对应真实镜头边界。
【辅助算子】video_clip_reassembly_mapper 用于把在重叠 clip 上分别计算的结果（如手部动作轨迹）重新拼接回长视频时间轴，服务于具身智能场景的长时序标注。
【工程层面】v1.4.6 起支持视频字节流 I/O，切分过程可在内存中完成而不必反复落盘，对大规模切分的 I/O 开销有实质改善。
【实际使用情况】官方 T2V 案例未启用切分算子（源数据集已预切分）。因此 DJ 的切分能力虽然齐备，但缺乏在超大规模长视频语料上的公开实战数据（如切分吞吐、平均每小时视频切出多少 clip）。

### 质量过滤（美学评分、清晰度、OCR文字过滤、黑边/水印/logo检测） ⚠️

质量过滤是 DJ 视频算子覆盖最密集的环节，共16个视频侧 Filter，可按维度归类如下：
【美学与感知质量】
  · video_aesthetics_filter —— 对指定抽帧计算美学评分并按区间过滤，底层为 LAION/improved-aesthetic-predictor 系的美学打分模型，支持配置抽帧策略（均匀抽帧/关键帧）与 any/all 聚合逻辑（任一帧达标 vs 所有帧达标）。这是 Sandbox 探测实验中的三大候选算子之一。
【基础规格】
  · video_resolution_filter —— 分辨率上下界。
  · video_aspect_ratio_filter —— 宽高比区间。
  · video_duration_filter —— 时长区间。
【文字与叠加物】
  · video_ocr_area_ratio_filter —— 「检测指定帧的文字区域占比并按区间过滤」，文档明确指出「OCR 文字面积占比低的视频往往质量更高」，用于剔除字幕烧录、弹幕、幻灯片、图文海报类样本。这是视频生成数据清洗的标配环节。
  · video_watermark_filter —— 「保留大概率无水印的视频」，基于水印检测模型打分过滤。
  · video_remove_watermark_mapper —— 与前者互补的修复路径：给定水印区域坐标列表（x1,y1,x2,y2 格式）直接把水印抹除，从而挽救带水印但内容优质的素材，而非整条丢弃。
【内容安全】
  · video_nsfw_filter —— NSFW 打分过滤（详见 safety_filtering）。
【语义对齐】
  · video_frames_text_similarity_filter —— 抽帧与文本的 CLIP 相似度过滤（详见 model_as_data_judge 与 av_sync_detection 相关条目）。
【人物相关】
  · video_face_ratio_filter —— 人脸占画面比例区间过滤，可用于区分特写/中景/远景，或剔除无人/满屏人脸的极端样本。
  · video_tagging_from_frames_filter —— 按内容标签保留或剔除。
【运动】
  · video_motion_score_filter / video_motion_score_raft_filter / video_motion_score_ptlflow_filter（详见 motion_filtering）。
【设计特点评价】
  1. 「过滤 + 修复」双路：水印、分辨率、宽高比三处都同时提供 Filter（丢弃）与 Mapper（改写修复）两种算子，使用者可按数据稀缺程度选择策略。这在同类框架中较为少见。
  2. 阈值全部外置为 YAML 参数、无硬编码默认策略：DJ 不替使用者决定「美学分该卡多少」，而是通过 Sandbox 的探测实验用模型反馈来定。这是其方法论立场——阈值应当是被搜索出来的，不是被规定的。
  3. [不确定] 未内置黑边检测（letterbox/pillarbox）、模糊度/清晰度专项打分（如拉普拉斯方差、MUSIQ/DOVER 等视频质量评估模型）、编码码率过滤等若干在模型团队 pipeline 中常见的项目；也未提供多指标加权综合评分机制，所有过滤均为单指标独立阈值的串行叠加。

### 运动过滤（光流/运动分数阈值、静态与抖动剔除） ⚠️

运动过滤是 DJ 少数提供了三种并列实现的环节，反映团队对该维度的重视：
  · video_motion_score_filter —— 基于 OpenCV 的光流估计（Farneback 稠密光流）计算运动分数，按区间过滤。速度快、无需 GPU，是默认选项。
  · video_motion_score_raft_filter —— 基于 RAFT（Recurrent All-Pairs Field Transforms）深度光流模型计算运动分数。精度显著高于传统光流，尤其在大位移与遮挡场景，代价是需要 GPU 推理。
  · video_motion_score_ptlflow_filter —— 基于 ptlflow（PyTorch Lightning Optical Flow）库，可接入其中数十种光流模型（FlowNet、PWC-Net、GMA、FlowFormer 等），提供最大的模型选择灵活性。
【三者并存的意义】使用者可按算力预算与精度需求在「OpenCV 快但粗 → RAFT 准但慢 → ptlflow 可任选」之间权衡，这种同一功能多档实现的设计在数据处理框架中是加分项——大规模粗筛用 OpenCV、精筛用 RAFT 是自然的两阶段用法。
【过滤语义】三者均为双边区间过滤（min_score / max_score），下界剔除静止或近静止画面（视频生成模型若学到大量静态样本会倾向生成「PPT 式」无动感视频），上界剔除运动过于剧烈、抖动、快速剪辑的片段（光流估计失真、时序建模困难）。
【实际使用情况】[不确定] 官方 T2V 案例的最终配方中未包含运动分数算子——在 Probe 阶段的算子排序中，video_aesthetics_filter、video_nsfw_filter、video_frames_text_similarity_filter 三者进入候选，最终胜出的是后两者。论文未报告运动分数算子在该案例中的探测名次或未被选中的原因，因此无法判断是运动维度在该数据集上增益不显著，还是根本未纳入候选。也未公开任何推荐阈值数值。

### 去重方法（精确去重与基于embedding的语义去重分别记录） ⚠️

去重能力分布不均：文本侧极强，视频侧较弱。
【视频去重——仅支持精确去重】
  · video_deduplicator —— 「使用视频的精确匹配在文档级去重」，即基于视频文件内容哈希的精确去重，只能识别完全相同的文件，无法识别转码、裁剪、加水印、变分辨率后的近重复样本。
  · ray_video_deduplicator —— 上者的 Ray 分布式版本，逻辑相同但可跨节点并行，适配 TB 级规模。
  [不确定] DJ 未提供基于视频 embedding（如 CLIP/VideoMAE 特征 + ANN 检索）的语义去重算子，也未提供感知哈希（pHash/dHash）类的近重复检测。这在视频生成数据清洗中是明显短板——网络视频语料中的近重复（同一素材不同来源、不同压缩、加片头片尾）比例通常远高于字节级完全重复。使用 DJ 构建大规模视频语料时，语义去重环节需自行补齐。
【文本侧去重——工程能力突出】
  · MinHash 局部敏感哈希去重（Ray 分布式版）：采用「负载均衡的并查集」（load-balanced union-find）实现，相对原生 Ray 实现取得 3.3 倍加速；实测 8 个 Ray 节点处理 5TB 数据耗时 2.8 小时。论文将其与 NVIDIA NeMo Curator 对比（后者用 64 张 A100 处理 1.1TB 耗时 1.8 小时），意在说明 DJ 用纯 CPU 达到了可比的性价比。扩展性数据：数据量增至5倍时耗时增至4.02–5.62倍（近线性）；核心数翻倍时耗时降至原来的58.9%–67.1%。
  · v1.5.2 新增跨文档行级去重（cross-document line deduplication），处理粒度细化到行。
  · 另有 SimHash、精确哈希、文档级/段落级等多种去重算子，Deduplicator 类共10个。
【图像侧】提供基于图像哈希的去重算子。
【总体判断】DJ 的去重强在「分布式工程能力」而非「视频语义感知能力」——它解决的是「如何在10k核上高效跑去重」，而非「如何判断两段视频语义重复」。

### VLM/LLM作为质检员（多模态大模型质量打分与错配剔除，2026年从浅层打分器转向大模型语义判断的趋势） ⚠️

Data-Juicer 是「用基础模型处理数据」（for and with Foundation Models 中的 with）这一理念的典型实践者，且其做法与模型团队有方法论差异：它不是用一个大模型给数据打综合分，而是把大量专用模型封装为独立算子，再用下游模型的训练反馈来选择该信任哪些算子。
【作为质检员的模型算子清单（视频侧）】
  · CLIP —— video_frames_text_similarity_filter，计算抽帧与文本描述的跨模态相似度，是 T2V 案例最终采用的核心判据（阈值0.306337）。本质是用 CLIP 作为「视频-文本对齐质检员」，剔除 caption 与画面不符的样本。
  · Falconsai/nsfw_image_detection —— video_nsfw_filter 的底层模型，作为安全质检员。
  · 美学评分模型 —— video_aesthetics_filter，作为视觉品味质检员。
  · RAM/图像打标模型 —— video_tagging_from_frames_mapper，作为内容标注员。
  · Audio Spectrogram Transformer —— video_tagging_from_audio_mapper / video_audio_ASR_mapper，音频侧标注员。
  · Qwen-Audio —— video_captioning_from_audio_mapper，音频理解与描述。
  · 通用 VLM —— video_captioning_from_vlm_mapper，「使用可直接接收视频输入的 VLM 生成视频描述」，这是 DJ 中最贴近2026年「大模型语义判断」趋势的算子，支持接入任意视频 VLM。
  · YOLOE + SAM2 —— video_object_segmenting_mapper，物体级语义证据。
  · wav2vec2 —— 说话人年龄性别检测。
【"以模型反馈为裁判"的元层设计——DJ 最有特色之处】Sandbox 的 Analyze 阶段实质上是「用下游生成模型的评测分数来给质检员本身打分」：每个候选算子先切出低/中/高三个池，各训一个参考模型，用 VBench 评测后排序，从而客观地判断「这个质检员对最终生成质量到底有没有用」。这套机制回答了模型团队 pipeline 中普遍回避的问题——「我加的这个过滤器真的有用吗」。T2V 案例的结论正是靠它得出的：CLIP 对齐过滤与 NSFW 过滤有效并被保留，其余候选算子未入选。这种「先验证再采用」的立场，比直接堆砌 VLM 打分器的做法更严谨，是 DJ 在方法论上对本调研最有借鉴价值的一点。
【LLM 参与数据处理的其他形式】v1.4.5 起支持 Ray + vLLM 流水线，可在分布式环境高效批量调用本地大模型做标注/合成；v1.4.6 引入「Q&A Copilot」；接口层通过 AgentScope 智能体支持自然语言下发数据处理指令；v1.5.2 新增 agent 数据质量工具包。
[不确定] DJ 未内置类似 VideoScore、DOVER 等专门的视频质量评估大模型算子，也未提供「VLM 综合打分并给出理由」式的开放式质检算子模板。

### 安全与合规过滤（NSFW、版权、人脸/隐私） ⚠️

【视觉安全】video_nsfw_filter 是唯一的视频侧安全算子，「保留 NSFW 分数在指定区间内的样本」，底层为 Falconsai/nsfw_image_detection 图像分类模型（对抽帧逐帧打分后聚合）。该算子在 T2V 案例中被 Probe 阶段验证为有效并纳入最终配方，官方表述为「使用 VideoNSFWFilter 算子以保证高质量」——即在该案例中它同时承担了安全与质量双重职责（NSFW 高分样本往往也伴随低质量、非自然内容）。
【隐私保护】video_face_blur_mapper 检测并模糊视频中的人脸，是数据处理框架层面较少见的内置隐私脱敏算子。配合 video_human_tracks_extraction_mapper（人脸/人体轨迹提取）可实现轨迹级的持续脱敏。文本侧另有敏感信息（手机号、邮箱、身份证等）脱敏类算子。
【版权相关】video_watermark_filter 与 video_remove_watermark_mapper 可识别与处理水印，间接关联版权标识，但不构成版权归属判定能力。
[不确定] 缺失的环节包括：无版权内容识别/指纹比对能力；无 C2PA 等来源认证标准支持；无暴力、仇恨、未成年人等细分安全类目的检测算子（仅有二元 NSFW）；未披露阿里内部使用时叠加的安全审核体系。作为开源框架，DJ 提供的是「可组装的安全零件」，完整的合规体系需使用方自行构建。

## 打标方式

### 使用的caption模型（自研VLM/开源模型，模型规模） ⚠️

Data-Juicer 不自研 caption 模型，而是把多种 caption 模型封装为可插拔算子，构成本调研中视频 captioning 算子最丰富的开源框架之一。共6个视频 captioning 相关 Mapper：
  · video_captioning_from_vlm_mapper —— 「使用可接收视频输入的 VLM 生成视频描述」。最通用的接口，可接入任意视频 VLM（Qwen2.5-VL、InternVL、Video-LLaVA 等），是2026年主流做法的对应算子。模型规模由使用者选择。
  · video_captioning_from_video_mapper —— 使用 HuggingFace video-to-text 模型生成描述（如 VideoBLIP 系）。
  · video_captioning_from_frames_mapper —— 抽帧后用 image-to-text 模型（BLIP/BLIP-2 系）逐帧生成描述再聚合。成本最低但缺乏时序理解。
  · video_captioning_from_audio_mapper —— 「基于 Qwen-Audio 根据音轨为视频生成描述」。听觉侧 caption，是构建音视频联合标注的关键零件。
  · video_captioning_from_summarizer_mapper —— 「通过对多种已生成文本做摘要来生成视频描述」。这是 DJ 最有设计感的 captioning 算子：把前述各路（帧描述、音频描述、OCR 文字、内容标签、ASR 转写）的产出汇总后交给 LLM 做摘要融合，产出一条综合性的密集描述。这条「多源分述 → LLM 融合」的路径与 Movie Gen、Panda-70M 等团队的多模型融合 captioning 思路一致。
  · video_captioning_from_human_tracks_mapper 与 video_captioning_face_attribute_emotion_mapper —— 人物中心的定向描述（v1.5.4 新增），前者基于人物轨迹生成描述，后者为每个被追踪的人生成面部属性与情绪描述。
【配套的标注型算子】video_tagging_from_frames_mapper（视觉标签）、video_tagging_from_audio_mapper（音频标签，AST）、video_audio_ASR_mapper（语音识别）——这些不产出自然语言描述但产出结构化标签，是 summarizer 算子的输入源。
【实际使用情况】[不确定] 官方 T2V 案例未启用任何 captioning 算子——三个源数据集自带 caption（InternVid 用 BLIP-2 + Tag2Text 生成、Panda-70M 用多模型 caption 融合 + 检索式挑选、MSR-VTT 为人工标注），案例只做了 CLIP 对齐过滤而未重新打标。因此 DJ 的 captioning 算子链虽然完备，但缺乏在大规模视频生成语料上的公开效果验证与推荐配置。DJ 官方也未发布视频 captioning 的推荐模型选型与 prompt 模板。

### caption密度与结构化程度（短/长/密集描述、结构化字段如镜头运动、风格标签） ⚠️

【DJ 的立场：提供组装零件，不规定 caption 范式】DJ 不预设 caption 应该多长、包含哪些字段，而是提供从「短标签」到「长密集描述」的全谱系算子，由使用者按需组合。
【可产出的 caption 密度谱系】
  · 最稀疏：video_tagging_from_frames_mapper / video_tagging_from_audio_mapper 产出的是标签词表（如 AudioSet 527类标签、RAM 开放词表标签），不是句子。
  · 中等：video_captioning_from_frames_mapper / from_video_mapper 产出单句或数句的常规描述。
  · 最密集：video_captioning_from_summarizer_mapper 通过融合多源信息产出综合长描述；video_captioning_from_vlm_mapper 可通过 prompt 控制产出任意长度与结构的描述。
【结构化字段的支持情况】DJ 的数据模型是「样本 = 多模态字段 + 任意数量的 stats/tags 字段」，每个算子把自己的产出写入独立的字段而非合并进 caption 文本。这意味着结构化信息天然是分字段存储的：镜头级的美学分、运动分、OCR 占比、人脸比例、内容标签、说话人年龄性别情绪、相机内参与位姿、深度、全身关键点等，都是并列的结构化字段，可被下游训练框架按需拼接进条件文本或作为独立条件通道。这种「字段化优先」的设计在结构化程度上实际高于多数只产出一段长文本的模型团队 pipeline。
[不确定] DJ 未提供官方推荐的视频生成 caption 模板（如是否应包含镜头运动、光照、风格、主体动作等固定字段），也未在任何公开案例中报告 caption 平均长度、字段构成或不同 caption 结构的效果对比。这属于其「工具中立」定位的必然结果，但也意味着使用者无法从 DJ 获得 caption 设计的最佳实践指引。

### 音视频联合caption结构（是否同时覆盖视觉+听觉轨道、是否分流为独立字段，如LTX-2全音景描述、Script-a-Video factorized streams、Foley-Omni三字段） ⚠️

[不确定] Data-Juicer 未定义任何音视频联合 caption 的 schema，这是其相对于 LTX-2（全音景描述）、Script-a-Video（factorized streams）、Foley-Omni（[WORDS]/[AUDIO]/[MUSIC] 三字段）等专门工作的显著空白。
【但具备拼装出联合 schema 的零件】DJ 的字段化数据模型使使用者可以自行构造类似 schema：
  · 听觉侧内容描述：video_captioning_from_audio_mapper（Qwen-Audio 根据音轨生成描述）→ 可作为「音景描述」字段。
  · 听觉侧标签：video_tagging_from_audio_mapper（AST，AudioSet 527类）→ 可近似区分语音/音乐/环境音，作为音频类别字段。
  · 语音内容：video_audio_ASR_mapper → 可作为对白转写字段。
  · 说话人属性：video_audio_detect_age_gender_mapper、video_audio_speech_emotion_mapper → 可作为说话人属性字段。
  · 视觉侧描述：video_captioning_from_vlm_mapper / from_frames_mapper → 视觉描述字段。
  · 融合：video_captioning_from_summarizer_mapper 可把上述各字段交给 LLM 融合为一条统一描述——这实际上就是「全音景 + 视觉」统一描述路线的可行实现。
【关键缺失】无音轨源分离算子，因此无法像 Foley-Omni 那样把音轨拆成 speech/effects/music 三个 stem 并分字段描述与验证；无声学能量门控机制来纠正 VLM 的视觉幻觉（看到乐器就写有音乐）。这两点是构建高质量 AV 联合标注的关键环节，DJ 目前不提供。
【结论】DJ 提供了构建 AV 联合 caption 的大部分原材料，但没有提供 schema 设计与跨模态交叉验证的方法论，需使用方自行补齐约30%的关键环节（源分离 + 能量验证 + schema 定义）。

### 对白转写与说话人属性标注（ASR转写+说话人身份/语言/口音/情绪） ⚠️

【转写能力】video_audio_ASR_mapper —— 从视频的音轨做语音识别，文档描述为「基于 Audio Spectrogram Transformer 从音频流生成视频标签」（该算子在文档中与 tagging 类算子描述重合，实际承担 ASR 与音频事件标注职责）。可产出语音转写文本作为独立字段。
【说话人属性——DJ 覆盖较好的一组能力】
  · video_audio_detect_age_gender_mapper —— 「基于预训练 wav2vec2 模型从视频音频信号中检测年龄与性别」。产出说话人的年龄段与性别属性，这在开源数据处理框架中较少见。
  · video_audio_speech_emotion_mapper —— 语音情绪识别，产出情绪标签字段。
  · video_active_speaker_detect_mapper —— 「通过分析视觉人脸轨迹与音频信号检测视频中的主动说话人」。这是把「说话人」从音轨绑定到画面中具体人物的关键算子，多说话人场景下用于确定「此刻是谁在说话」。
  · video_captioning_face_attribute_emotion_mapper —— 为每个被追踪的人生成面部属性与情绪的自然语言描述，是视觉侧的情绪证据，可与语音情绪交叉验证。
  · video_human_tracks_face_demographic_mapper —— 人物轨迹的人口统计属性。
  · video_human_tracks_extraction_mapper —— 提取人脸与人体边界框轨迹，是上述人物算子的公共前置。
【属性覆盖对照】年龄✓、性别✓、情绪✓（音+视双路）、说话人身份（轨迹级 ID）✓、说话人-画面绑定✓；语言✗、口音✗。
【应用佐证】HumanVBench（CVPR 2026，同团队）明确基于 Data-Juicer 构建，使用20+个 SOTA 算子搭建了「人物中心视频标注 pipeline」（Human-Centric Video Annotation Pipeline），基准覆盖情绪感知、人物识别、行为分析、语音-视觉对齐四个维度——这是上述算子链在真实数据集构建中的完整落地案例，也间接说明这组算子已具备生产可用性。
[不确定] 未披露 ASR 底层模型的具体选型与多语种支持范围；未提供转写质量评估或置信度过滤机制。

### 几何与结构化标注（相机参数、深度、3D point tracks、动作标注、显式状态） ⚠️

这是 Data-Juicer 在2026年投入最集中、也是其相对其他数据处理框架最独特的方向——由具身智能（Embodied AI / VLA）需求驱动，v1.4.5、v1.5.0、v1.5.3 连续三个版本大量新增几何与结构化标注算子。对视频生成而言，这批算子恰好对应 SceneScribe-1M、SpatialVID 等「文本 caption 之外的第二类标注范式」。
【相机参数标注——提供三种实现】
  · video_camera_calibration_deepcalib_mapper —— 基于 DeepCalib 计算静态相机的内参与视场角（FOV）。
  · video_camera_calibration_droidcalib_mapper —— 基于 DroidCalib 从视频提取相机内参。
  · video_camera_calibration_moge_mapper —— 基于 MoGe-2 计算内参与 FOV。
  · video_undistort_mapper —— 用已求得的内参与畸变系数对原始视频做去畸变校正。
【相机位姿/轨迹】
  · video_camera_pose_megasam_mapper —— 「结合 MegaSaM 与 MoGe-2 提取相机位姿」。MegaSaM 是2025年的动态场景 SLAM 方法，可在含运动物体的野外视频上估计相机轨迹——这正是视频生成中「镜头运动」条件标注所需的核心能力。
【深度】
  · video_depth_estimation_mapper —— 对视频做深度估计，产出稠密深度图序列。
【人体与手部——动作标注链】
  · video_whole_body_pose_estimation_mapper —— 提取身体、手部、脚部、面部关键点的2D全身姿态估计。
  · video_hand_reconstruction_hawor_mapper —— 基于 HaWoR + MoGe-2 的手部重建。
  · video_hand_reconstruction_mapper —— 基于 WiLoR 模型的手部定位与重建。
  · video_hand_motion_smooth_mapper —— 对世界坐标系下的手部运动做平滑并剔除离群点。
  · video_hand_action_compute_mapper —— 「从手部重建结果与相机位姿计算 7-DoF 动作与 8 维状态」。这是明确的「显式状态」标注——把视频转化为机器人可用的动作-状态序列，是 VLA 训练数据的标准格式。
  · video_atomic_action_segment_mapper —— 把统一的手部轨迹切分为原子动作片段，即动作级的时序分割标注。
  · video_trajectory_overlay_mapper —— 「采样帧并叠加手部轨迹，准备 VLM 可读的帧」。把几何轨迹可视化叠加到画面上再喂给 VLM，是几何标注与语义标注的桥接技巧。
  · video_clip_reassembly_mapper —— 把重叠 clip 上的手部动作结果重组回长视频时间轴。
【物体级】
  · video_object_segmenting_mapper —— 基于 YOLOE + SAM2 的文本引导语义分割，产出全视频的物体掩码序列。
【评价】这套算子链使 DJ 成为目前开源框架中几何标注能力最完整的一个：相机内参、畸变、位姿轨迹、深度、全身/手部姿态、7-DoF 动作、8维状态、原子动作分割、物体分割全部具备，且大量采用2025–2026年的最新模型（MegaSaM、MoGe-2、HaWoR、WiLoR、SAM2、YOLOE）。虽然设计初衷是具身智能，但对需要相机控制、深度条件、动作条件的可控视频生成而言可直接复用。
[不确定] 未见 3D point tracks（如 CoTracker/SpatialTracker 式的长时序点轨迹）专用算子；这批算子的处理吞吐、成本与在大规模视频生成语料上的实战效果均未公开。

### 合成数据构造（受控扰动/编辑构造训练对，如InstructAV2AV） ⚠️

【合成能力的规模】229个算子中约50个专门服务于数据合成与增广，是 DJ 的四大能力板块之一（分析、合成、标注、后训练）。这一比例明显高于以清洗为主的同类框架。
【视频侧的合成/构造算子】
  · video_ffmpeg_wrapped_mapper —— 封装 FFmpeg 视频滤镜，可施加任意视频变换（裁剪、调色、加噪、变速、转场等），是构造受控扰动样本的通用工具。
  · audio_ffmpeg_wrapped_mapper —— 音频侧对应物。
  · audio_add_gaussian_noise_mapper —— 音频加高斯噪声增广。
  · video_resize_resolution_mapper / video_resize_aspect_ratio_mapper —— 规格改写，可用于构造多分辨率训练对。
  · video_remove_watermark_mapper —— 水印去除，可与原始带水印样本构成「去水印」编辑训练对。
  · video_object_segmenting_mapper —— 产出物体掩码，是构造对象级编辑/替换训练对的前置条件。
【DJ-Cookbook 中的合成配方】维护有「视频数据合成」（video-data-synthesis）YAML 配方、对比学习数据合成配方（对应 CVPR 2025 的 ImgDiff 工作，arXiv:2408.04594，通过对比式数据合成提升 VLLM）、角色导向对话合成、以及基于数据难度的课程学习配方。
【关联研究】同团队的 ImgDiff（CVPR 2025）与 MindGYM（NeurIPS 2025，问题合成）都是「用大模型合成训练数据」的方法论工作，其能力已回流为 DJ 算子。
[不确定] DJ 未提供类似 InstructAV2AV 的音视频编辑指令对构造能力，也未提供针对视频生成的「受控扰动构造正负样本对」的现成配方（如构造运动强度对比对、音画错位负样本对）。视频合成配方的具体内容与实际产出规模未公开。

### 人工介入程度（人工标注、人工质检、模型初筛+人工复核） ⚠️

Data-Juicer 的设计取向是「尽可能减少人工介入，用模型反馈替代人工判断」，人工的角色被限定在两个环节。
【人工介入点一：配方设计与阈值决策的最终裁定】DJ 不预设任何默认阈值，所有算子超参外置于 YAML。但 Sandbox 的核心贡献恰恰是把这个环节从「人工凭经验设阈值」改造为「用小规模参考模型训练 + 基准评测的反馈自动排序」——人只需要指定候选算子集合与评测基准，具体保留哪一段分布由实验数据决定。这是本调研中唯一把阈值确定过程系统化、自动化的工作。
【人工介入点二：交互式操作与可视化审查】接口层提供多级人机交互通道：低层 Python API（工程师编程）、RESTful 端点（服务化调用）、可视化编辑器（阿里云 PAI Designer 组件，拖拽式配置数据处理流程）、基于 AgentScope 智能体的自然语言对话式指令（用一句话描述数据处理需求）。v1.4.6 引入的「Q&A Copilot」进一步降低使用门槛。此外 DJ 提供数据分析报告与追踪器（Tracer，v1.4.6 支持 Ray 模式），可让人查看每个算子实际剔除了哪些样本——这是人工抽样质检的支撑设施，但质检本身由人自行进行。
【规模化的群体参与】DJ 2.0 论文提到其支撑了3000+团队参与的数据过滤/合成竞赛，属于一种众包式的配方探索，但不是逐样本的人工标注。
【无人工标注环节】DJ 全部标注能力均由模型算子提供，不含任何人工标注工作流、标注平台对接或标注质量抽检机制。
[不确定] 官方 T2V 案例中未披露是否有人工复核环节；DJ 也未提供标注一致性检验、人工-模型标注对比等质量保障工具。

## 音视频对齐

### 音视频同步检测方法（口型同步、事件对齐） ⚠️

[不确定] Data-Juicer 未提供专门的音视频同步检测算子，这是其在 AV 生成数据处理链上最关键的能力缺口。
【最接近同步检测的算子】video_active_speaker_detect_mapper —— 「通过分析视觉人脸轨迹与音频信号检测视频中的主动说话人」。该算子的底层原理（如 TalkNet/Light-ASD 类主动说话人检测模型）确实依赖唇部运动与音频的时序相关性，因此隐含了唇同步判断；但它的输出是「谁在说话」的身份/时段判定，而不是「音画偏移量是多少帧」「同步置信度多高」的可用于阈值过滤的连续分数。使用者无法直接用它做 LSE-D/LSE-C 式的同步质量筛选。
【缺失的能力】
  · 无 SyncNet 算子——无法输出音画偏移量（offset）与置信度（confidence），因而无法复现 SkyReels-V4 的「|offset|≤3 且 conf>1.5」式过滤。
  · 无 Synchformer 算子——无法对通用音视频事件（非语音）做时序对齐打分，因而无法复现 Foley-Omni 的 sync score ≥0.2 式过滤。
  · 无 AV-HuBERT、Perceiver 类音视频对齐表征算子。
  · 无 ImageBind 算子——无法做音视频跨模态语义一致性打分。
【视觉-文本对齐是有的】video_frames_text_similarity_filter 基于 CLIP 做「视频帧-文本」的跨模态一致性过滤，这是 DJ 唯一成熟且经实战验证的跨模态对齐算子（T2V 案例的核心判据，阈值0.306337）。但它处理的是视觉-文本轴，不是视觉-听觉轴。
【结论】若要用 Data-Juicer 构建音视频联合生成的训练数据，同步检测环节必须自行扩展算子。好在 DJ 的算子注册机制（继承 Filter 基类 + YAML 声明 + stats 字段写入）使得封装 SyncNet/Synchformer 为自定义算子的工程成本较低，且可直接复用其 Ray 分布式执行、GPU 自适应分配、批处理优化等基础设施——这正是 DJ 作为「算子库」而非「固定 pipeline」的价值所在。

### 同步检测具体指标与阈值（SyncNet/Synchformer/LSE/自研，阈值数值，如MOVA LSE-D≤9.5且LSE-C≥4.5、SkyReels-V4 SyncNet |offset|≤3∧conf>1.5） ⚠️

[不确定] 由于不提供同步检测算子，Data-Juicer 也不存在任何同步指标与阈值的推荐值。
【DJ 公开的唯一跨模态阈值——供参照】video_frames_text_similarity_filter 在官方 T2V 最优数据池中的最小阈值为 0.306337（CLIP 视觉-文本余弦相似度）。这个数值的产生方式值得注意：它不是人工设定的整数（如常见的0.3），而是 Probe-Analyze-Refine 流程中按数据池分位切分自然落到的边界值——即「保留相似度最高的那一部分样本」时该分位点对应的具体数值。这种「阈值由目标保留比例反推」的做法与模型团队常见的「人工经验定阈值」形成对比。
【其他算子的阈值状况】DJ 全部算子的阈值均为 YAML 外置参数，官方 config_all.yaml 中给出的是占位默认值而非推荐值。文档明确不提供跨场景通用的推荐阈值，其立场是阈值应通过 Sandbox 在具体数据集与具体下游模型上搜索确定。
【方法论价值】对本调研而言，DJ 提供的不是「阈值数值」而是「确定阈值的方法」：把数据按某统计量三等分 → 各训一个参考模型 → 用统一基准评测 → 选出增益最大的分段。这套流程理论上可直接套用于同步指标（如把 SyncNet confidence 三等分做探测），只是 DJ 官方尚未在音视频同步维度做过此类实验。

### 时序同步vs语义同步的分离处理（时间对齐与内容语义匹配作为两个独立过滤条件） ⚠️

[不确定] Data-Juicer 未涉及时序同步与语义同步的分离处理，因其不具备音视频同步检测能力（详见 av_sync_detection）。
【类比性的设计思想是存在的】DJ 的整体架构立场恰恰是「每个维度一个独立算子、一条独立阈值，不做加权合并」——所有 Filter 算子都是单指标独立判定后串行叠加，从不提供多指标综合评分。这在方法论上与 Foley-Omni 把 ImageBind（语义）与 Synchformer（时序）拆为两条独立阈值的做法同源：都主张不同性质的错误应由不同的判据独立把关，避免高分维度掩盖低分维度。因此若在 DJ 上扩展音视频同步能力，天然会落到「语义算子 + 时序算子分列」的形态，架构上无障碍。
【现有的一个分离实例】在视觉-文本轴上，DJ 确实把「语义匹配」（video_frames_text_similarity_filter，CLIP 相似度）与「内容属性」（video_tagging_from_frames_filter，标签匹配）分作两个独立算子，而非合并为一个综合对齐分——可视为同一设计原则在另一模态轴上的体现。

### 音频质量过滤（SNR、静音检测与静音占比阈值、无音轨剔除、画外音源剔除、背景音乐分离） ⚠️

音频质量过滤是 DJ 音频侧能力最实在的部分，但覆盖面明显窄于视频侧。
【现有算子】
  · audio_nmf_snr_filter —— 「保留音频信噪比（SNR）在指定区间内的样本」。底层采用 NMF（非负矩阵分解）方法估计信噪比，把音频分解为「信号成分」与「噪声成分」后计算比值。相比简单的能量统计，NMF 估计对含结构性噪声（风噪、底噪、嗡鸣）的场景更稳健。这是 DJ 唯一的音频质量核心算子。
  · audio_duration_filter —— 时长区间过滤。可间接用于剔除无音轨样本（时长为0）与过短音频片段。
  · audio_size_filter —— 按音频文件大小过滤，可粗略识别异常低码率（体积过小）或异常格式的样本。
  · audio_ffmpeg_wrapped_mapper —— 封装 FFmpeg 音频滤镜，可施加降噪、响度归一化、重采样、声道处理等任意修复操作。这是「修复」而非「过滤」的路径，与视频侧的双路设计一致。
【缺失的关键能力】[不确定]
  · 无静音检测算子——无法设定「静音占比不得超过X%」类阈值，也无法定位并剔除长静音段。仅能通过 audio_duration_filter 间接排除全静音。
  · 无音轨存在性检测算子——「视频是否有音轨」需在数据准备阶段自行判断。
  · 无学习型音频质量评估算子——未集成 Meta AudioBox Aesthetics、NISQA、DNSMOS 等无参考感知质量模型，这与2025–2026年音频清洗从信号指标转向感知模型的趋势有落差（对照 Foley-Omni 使用 AudioBox Aesthetics ≥0.6）。
  · 无源分离算子——无法做背景音乐分离、人声提取，因而无法实现「剥离BGM保留人声」或「按音轨类别分别质检」。
  · 无画外音/旁白识别——video_active_speaker_detect_mapper 可判断画面中是否有人在说话，理论上可辅助识别「有语音但画面无说话人」的画外音情形，但 DJ 未把这一用法封装为现成算子或配方。
【总体判断】DJ 的音频质量过滤停留在传统信号处理层面（SNR、时长、体积），足以支撑 ASR/TTS 类语料清洗，但对音视频联合生成所需的精细音频质检（静音占比、感知质量、音轨分类、声源在画内外判定）覆盖不足。

### 语音/音效/音乐的分类与分别处理策略 ⚠️

[不确定] Data-Juicer 不提供语音/音效/音乐的显式分类体系与分别处理策略，这是其音频能力的核心短板。
【可用于近似分类的间接手段】
  · video_tagging_from_audio_mapper —— 基于 Audio Spectrogram Transformer（AST，在 AudioSet 上训练）产出音频事件标签。AudioSet 本体（ontology）为层次化的527类，顶层类目正好包含 Human sounds（含 Speech）、Music、Sounds of things、Natural sounds、Environment and background 等——理论上可据顶层类目把音频粗分为语音/音乐/音效/环境音四类。但 DJ 未把这一映射封装为显式的类别字段，也未提供按类别配比采样的算子，需使用者自行解析标签并编写映射逻辑。
  · video_audio_ASR_mapper —— 若能成功转写出有意义的文本，可作为「含语音」的强证据。
  · video_audio_speech_emotion_mapper / video_audio_detect_age_gender_mapper —— 这两个算子只在有人声时有意义，其可用性本身也构成语音存在性的间接判据。
【关键缺失】
  · 无音轨源分离算子（Demucs、Bandit、HTDemucs 等均未集成），因此无法把混合音轨拆成 speech/effects/music 三个 stem 分别处理与验证——这直接导致无法复现 Foley-Omni 式的「字段级能量门控」纠偏机制。
  · 无按音频类别的配比控制或分层采样算子。
  · 无音乐检测/BGM 识别专用算子。
【对本调研的意义】Data-Juicer 目前的算子体系是围绕「文本 → 图像 → 视频 → 具身智能几何标注」这条主线演进的，音频始终是配角（229个算子中纯音频算子仅3个 Filter + 2个 Mapper）。若把它作为音视频联合生成数据 pipeline 的底座，音频侧需要补齐的算子数量约在10个量级（源分离、静音检测、感知质量评估、音轨分类、同步检测、音轨存在性、响度统计等），工作量不小但架构上完全可行。

## 训练配合

### 多阶段训练课程与数据课程调度（阶段划分依据：分辨率/时长/质量分/模态；低清→高清、图像→视频、短→长） ⚠️

Data-Juicer 不训练模型，但在「数据课程调度」上提供了两类支撑，且其 T2V 案例本身就包含一个隐式的两阶段课程。
【DJ-Cookbook 中的课程学习配方】官方维护有「基于数据难度的课程学习」（curriculum learning based on data difficulty）YAML 配方，即先用算子给样本打上难度分（可用美学分、运动分、CLIP 相似度、文本复杂度等任意统计量作为难度代理），再按难度分层输出数据集供分阶段投喂。这是把课程调度产品化的现成模板。
【Sandbox 金字塔数据池 = 天然的课程结构】Refine 阶段构造的 2^n−1 个交叉数据池呈金字塔形：顶层同时满足所有算子的严格条件（质量最高、样本最少），逐层向下放宽条件（质量递减、样本递增）。这个结构可直接映射为训练课程——底层大样本量用于前期预训练打基础，顶层小样本高质量用于后期退火/微调。论文在扩量路线的对比中实际测试了「逐层下探纳入次优数据 + 去重」这一路径，等价于反向课程。
【T2V 案例的两阶段实践】最优数据池（147k）先用于蒸馏训练，再扩展至 228k 数据池（对应640k训练样本量，含约2.8倍重复）用于最终模型——这构成一个「小而精 → 扩量」的两阶段流程。
【工程层面的阶段支撑】v1.5.0 引入算子级环境管理（OP-level environment management），不同阶段可用不同依赖的算子组合；分区 Ray 执行器与 repartition 流水线（v1.5.3）便于按阶段产出不同规格的数据分片。
[不确定] DJ 未针对视频生成给出官方推荐的多阶段课程方案（如「低清→高清」「图像→视频」「短→长」的具体划分依据与切换时机），也未在公开案例中报告分辨率或时长维度的课程实验。这属于训练框架侧职责，DJ 只提供数据分层的原材料。

### 数据配比随训练阶段的变化（预训练/退火annealing/SFT高质量子集） ⚠️

【DJ 的能力定位】提供数据配比的度量、切分与搜索工具，不提供配比方案本身。
【配比搜索机制】Sandbox 的 Refine 阶段直接回答「不同质量层数据该如何组合」：把排名靠前的 n 个算子交叉组合成 2^n−1 个数据池，每个池对应一种配比方案，逐一训练小规模参考模型并评测，从中选出最优组合。这是把数据配比从人工设计变为可搜索问题的具体实现。
【扩量路线的两条对比——本案例最有价值的配比结论】论文明确对比了两种从小池扩到大池的路线：
  路线A：重复高质量数据（把顶层小池复制多份）。
  路线B：逐层下探纳入次优数据并去重（扩大样本多样性）。
  实验结论支持路线A在文生视频上更优——「把最优质数据重复6倍所得性能，高于使用8倍算力配合次优数据」。这意味着在 T2V 场景下，数据质量的边际价值高于数据多样性与算力投入。
【相关配比研究】同团队的 BiMix（arXiv:2405.14908）提出语言模型预训练的双变量数据混合律（Bivariate Data Mixing Law），把「数据配比 × 数据量」对损失的影响建模为可外推的定律；Diversity as Reward（arXiv:2502.04380，NeurIPS 2025）研究领域不确定数据上的多样性驱动微调数据选择。这两项工作代表了 DJ 团队在配比理论上的探索，但均以语言/多模态理解为对象，未覆盖视频生成。
[不确定] DJ 未提供预训练/退火/SFT 三阶段的官方配比模板；T2V 案例未报告 147k 与 228k 两个池之间的具体混合方式与各源数据集（InternVid/Panda-70M/MSR-VTT）在最终池中的占比变化。

### 后训练数据（SFT精选集规模与筛选标准、偏好对数量与标注方式、reward model训练数据） ⚠️

【系统能力】Data-Juicer 2.0 相对 1.0 的一个明确扩展方向就是「支持基础模型的后训练任务」（foundation model post-training），论文摘要中与数据分析、合成、标注并列为四大支持任务。约50个算子服务于数据合成与增广，大量面向后训练场景。
【具体支撑】
  · 后训练数据合成：DJ-Cookbook 含角色导向对话（persona-oriented dialog）处理配方、对比学习数据合成配方；同团队的 MindGYM（NeurIPS 2025，arXiv:2503.09499）研究面向思维能力微调的问题合成，其方法已回流为算子。
  · 数据选择：Diversity as Reward（NeurIPS 2025，arXiv:2502.04380）研究在领域不确定数据上做多样性驱动的微调数据筛选，即 SFT 精选集的构建方法论。
  · 强化微调支撑：DJ 2.0 论文提到系统「facilitates studies on data scaling laws and reinforced fine-tuning」，即已被用于强化微调（RFT）相关研究的数据准备。
  · Agent 数据质量：v1.5.2 新增 agent 数据质量工具包。
【视频生成后训练侧的情况】[不确定] DJ 官方未发布任何面向视频生成的偏好对（preference pairs）构造算子、reward model 训练数据构建配方，或 SFT 精选集筛选标准。T2V 案例走的是监督式蒸馏训练路线（基于 T2V-Turbo），不含 RLHF/DPO 环节。若要用 DJ 构建视频生成的偏好数据，需自行扩展「同 prompt 多样本生成 + 打分排序 + 配对」类算子。

### 数据处理基础设施与吞吐（NeMo Curator/Data-Juicer/自研，GPU加速比，处理规模，成本）

这是 Data-Juicer 2.0 的核心贡献维度，也是本调研中数据基础设施披露最充分的对象——绝大多数模型团队对此讳莫如深，而 DJ 的整篇论文都在回答这个问题。
【规模能力】处理 TB 级数据、调度 10k+ CPU 核心；实验覆盖 56万 至 700亿 样本的数据集规模；一个生产部署稳定运行5个月以上、累计处理规模超 TB 级。
【多后端自适应执行】运行时层提供硬件无关适配器，按规模自动/手动选择后端：
  · 小规模（约56万样本）：单机 standalone 模式最具性价比。
  · 中等规模（560万–5600万样本）：推荐 Ray 模式，4节点加速比为 148%–226%。
  · 大规模（700亿样本）：纯文本推荐阿里云 MaxCompute（在10k+核规模下耗时约为 Ray 的1/4）；多模态推荐 Ray-DLC（PAI 容器化深度计算集群）。
  · 分布式实测：3200 Ray-DLC 核心处理 500 倍数据集耗时 1780 秒、2500 倍数据集耗时 7083 秒（近线性扩展）。
【关键优化技术与量化收益】
  · 自适应数据切分（adaptive splitting）：大数据集上 2–3 倍加速。
  · 负载感知的算子重排序（workload-aware OP reordering）：在满足算子可交换性的前提下把轻量算子前置，最多减少 70.22% 处理时间。
  · GPU 资源自适应分配：模型类算子自动使用 GPU 与量化（集成 vLLM），最多节省 99% 处理时间。
  · 批处理（batched processing）：最多减少 84% 时间。
  · 算子融合（FusedOP）：以 Strategy/Decorator 模式做批级算子融合，显著降低网络 I/O——峰值吞吐从约 160MB/s 降至 60MB/s（即减少约2/3的无谓数据搬运）。
  · I/O 密集算子采用批处理、多进程、多线程的层次化并行。
  · v1.5.4 新增 batch-local stage fusion 进一步提速。
【去重性能对标 NeMo Curator】MinHash 去重采用负载均衡并查集，相对原生 Ray 实现取得 3.3 倍加速；8 个 Ray 节点处理 5TB 耗时 2.8 小时。论文以 NVIDIA NeMo Curator 处理 1.1TB 耗时 1.8 小时（64 张 A100）作为对照，突出 DJ 纯 CPU 方案的成本优势。扩展性：数据量增5倍耗时增4.02–5.62倍；核心数翻倍耗时降至58.9%–67.1%。
【成本与存储的实测洞察】
  · 阿里云针对集群网络的专项优化相比标准 ECS 节省 24.8% 成本。
  · 存储选型影响显著：使用 NAS/OSS 相比 CPFS 会使成本上升 20–30%。
  · 网络带宽提升3倍可带来 2.7 倍处理加速——论文明确指出大规模多模态数据处理是 I/O 瓶颈而非算力瓶颈，这是对本调研极有价值的工程结论。
【生态与落地】原生兼容 HuggingFace 数据集与 Ray 计算引擎；集成 ModelScope、NVIDIA NeMo、Alibaba Cloud PAI（含可视化 Designer 组件）、PAI-DLC 等20+框架与平台。采用方名单覆盖阿里巴巴集团、蚂蚁集团、比亚迪、字节跳动、DTSTACK、京东、NVIDIA、OPPO、小红书、小米、喜马拉雅等企业，以及中科院、南京大学、北京大学、中国人民大学、清华大学、国科大、浙江大学等高校。论文特别提到其支撑了阿里通义的企业级基础模型训练，尤其是 TB-token 级预训练与「空间智能」方向高开销的视频/图像处理。

## 效果对比

### 数据策略消融的量化影响（区分：过滤严格度ablation / caption密度风格ablation / 数据配比ablation，及对应评测指标） ⚠️

Data-Juicer Sandbox 的整个方法论就是围绕数据消融组织的——它把数据消融从「论文最后一节的验证实验」升级为「贯穿数据配方设计全程的搜索机制」。这是它对本调研最独特的贡献。
【消融类型一：过滤严格度消融（DJ 的主战场）】Probe 阶段对每个候选算子把数据集按统计量均分为低/中/高三个等大池 P_i,low / P_i,middle / P_i,high，外加随机采样对照池，在每个池上以相同预算训练参考模型并用 VBench（16项指标）评测。这等价于对每个过滤维度做完整的「保留哪一段分布最有利」的消融，且做到了严格控制变量（池大小相等、训练预算相同）。视频侧的候选算子为 video_aesthetics_filter、video_nsfw_filter、video_frames_text_similarity_filter 三者，最终 NSFW 与帧-文本相似度两项胜出并进入最终配方，美学分未入选。
【消融类型二：数据配比/组合消融】Refine 阶段把胜出算子交叉组合成 2^n−1 个数据池（金字塔结构），逐一验证组合效果，即多算子配比的穷举消融。
【消融类型三：数据量 vs 质量 vs 算力的三方消融——最有价值的定量结论】论文对比了三条扩量路线并给出明确排序：把最优质数据重复6倍所得性能，高于使用8倍算力配合次优数据。这是一个跨越数据质量、数据重复次数与算力预算三个变量的联合消融，结论直指「在文生视频场景，提升数据质量的性价比高于增加算力」。
【最终效果的量化】
  · T2V-Turbo 在 DJ 精炼数据池上训练后登顶 VBench：Board Average 82.53%、Uniform Average 81.26%，超过第二名 Gen-3 的 82.32% 与第三名 VEnhancer 的 81.97%。
  · 相对基线 T2V-Turbo 的提升：Board Average +1.53%、Uniform Average +2.59%。
  · HuggingFace 数据集卡片另记 147k 数据池带来「相对 T2V-Turbo 提升 1.09%」。
  · 论文强调该成绩是在「至少低 22 倍的算力成本」下取得的——这是「数据优化替代算力堆砌」最直接的证据。
  · EasyAnimate 上的小规模探测实验同样通过数据配方获得可观提升（具体数值未在可获取内容中给出）。
【跨模型可迁移性验证】小规模上（EasyAnimate）搜索出的最优配方，直接迁移到架构不同的 T2V-Turbo（基于 VideoCrafter-2.0）后仍然有效，说明数据配方的收益具备跨架构可迁移性——这是 Sandbox 方法论成立的前提，也是其最重要的实验发现之一。
【实验总量】论文报告完成了100+组实验，覆盖图文预训练（CLIP）、图生文（LLaVA 类）、文生视频（DiT 类）三类任务。
[不确定] 未公开 caption 密度/风格维度的消融（因案例中未重新打标）；未公开三个候选算子各自的探测名次与分数明细；未公开运动分数等其他视频算子是否参与过探测。

### 质量vs数量的证据（小而精数据超越大而杂的案例）

Data-Juicer Sandbox 的 T2V 案例是本调研中「质量优于数量」证据链最完整、最可复现的一例，因为其数据、配方、模型、评测全部开源。
【证据一：12.09% 的极端保留率仍取得 SOTA】从约121.7万条候选中仅保留147,176条（12.09%），用这1/8的数据训练出的模型登顶 VBench。若数量至上，直接用全量121.7万条应更优，实验结论相反。
【证据二：数据重复优于算力扩张——最锋利的一条】论文明确结论：把最优质数据重复6倍所得性能，高于使用8倍算力配合次优数据。这在方法论上很有杀伤力，因为它把「质量 vs 数量」的对比推进到了「质量 vs 算力」——即便给次优数据配上8倍算力也追不上高质量数据的简单重复。这直接挑战了「数据多样性必然优于数据重复」的常识，说明在文生视频这类对视频-文本对齐高度敏感的任务上，一条低质样本的负面影响大于一条重复样本的边际收益递减。
【证据三：22倍算力差距下的胜出】登顶 VBench 的模型相对 Gen-3、VEnhancer 等竞争者「至少低22倍算力成本」。这是「小而精数据超越大而杂」在跨团队横向对比上的体现。
【证据四：极简配方胜过复杂漏斗】最终有效配方仅含两个算子（NSFW 过滤 + CLIP 帧文相似度过滤），远比多数模型团队的十几级漏斗简单，且经过了严格的消融验证。这提示「过滤器数量」与「数据质量」之间并非正相关——真正相关的少数维度上严格把关，胜过在大量弱相关维度上宽松筛选。
【证据五：跨架构可迁移】小规模（EasyAnimate）搜索出的配方迁移到 T2V-Turbo 仍有效，说明「什么是高质量数据」具有一定的模型无关性，不是某个模型的过拟合产物。
【方法论意义】DJ 提供的不只是结论，还有得出结论的可复用机制（Probe-Analyze-Refine）。对视频生成团队而言，最实用的启示是：与其凭经验设计长漏斗，不如先用小模型对每个候选过滤维度做三分位探测，只保留真正带来评测增益的那几个维度。

### 训练数据domain分布与评测基准类目体系的对齐关系（如VABench七大类） ⚠️

【DJ 的评测对齐方式：把评测基准直接嵌入数据配方的搜索回路】这是 Sandbox 与其他工作最根本的差异——大多数团队是「先定数据分布，训完再上基准测」，DJ 是「用基准分数反过来决定数据分布」。视频侧使用 VBench 的16项指标作为 Analyze 阶段的排序依据，即数据配方是被 VBench 的类目体系直接塑造出来的。
【这种强对齐的收益与代价】
  · 收益：数据分布与评测口径天然一致，评测增益最大化，这是其登顶 VBench 的直接原因。
  · 代价：存在向基准过拟合的风险。若 VBench 的16项指标未能覆盖某些真实需求（如长时序一致性、多镜头叙事、音画协同），基于其反馈搜索出的数据配方就会系统性忽略这些维度。论文未讨论这一风险，是方法论上值得关注的一点。DJ 的缓解证据是跨架构可迁移性（配方在 EasyAnimate 与 T2V-Turbo 上均有效），但这只证明了跨模型泛化，未证明跨基准泛化。
【与本调研其他基准的关系】
  · Sandbox 的视频评测仅用 VBench，未接入 VABench（七大类目体系）、AVBench、AV-SyncBench 等音视频类基准——这与其纯视觉的案例定位一致。
  · 若要把 Sandbox 方法论迁移到音视频联合生成，只需把 Analyze 阶段的评测器换成 VABench/AV-SyncBench 等，架构上无需改动；但如前所述，音视频侧的候选算子（同步检测、音轨分类、音频感知质量）需先行补齐，否则探测阶段无候选可排序。
【同团队的基准建设】HumanVBench（CVPR 2026，arXiv:2412.17574）由该团队基于 Data-Juicer 构建，覆盖情绪感知、人物识别、行为分析、语音-视觉对齐四大维度，使用20+算子搭建了人物中心视频标注 pipeline 与含干扰项的 QA 合成 pipeline。其中「语音-视觉对齐」（speech-visual alignment）类目是 DJ 生态中唯一触及音视频对齐评测的部分，可视为未来 DJ 向 AV 方向扩展的接口。此外团队还有 DetailMaster（ICML 2026，arXiv:2505.16915）关注文生图长 prompt 处理。
【数据分布可度量性】DJ 的一个隐性优势是：由于所有算子都会把统计量写入样本的 stats 字段，任何数据集在任何维度上的分布都可被量化并绘制直方图，这使得「训练数据分布 vs 评测基准类目」的对齐分析在技术上是可执行的——只是 DJ 官方未针对视频生成公开过此类分析报告。
[不确定] 未公开最终数据池在 VBench 16项指标对应维度上的分布画像；未做与 VABench 等其他基准类目体系的交叉对齐分析。

## 不确定字段

以下字段的调研信息部分不确定（⚠️ 标注来源）：

- release_date
- duration_distribution
- resolution_aspect_distribution
- domain_distribution
- audio_category_distribution
- narrative_structure_distribution
- language_accent_distribution
- provenance_licensing
- funnel_retention_rate
- quality_filtering
- motion_filtering
- deduplication
- model_as_data_judge
- safety_filtering
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
- data_ablation
- benchmark_taxonomy_alignment
