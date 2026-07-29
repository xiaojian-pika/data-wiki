# Step-Video-T2V（阶跃视频，30B 文生视频基础模型）及其衍生 Step-Video-T2V-Turbo、Step-Video-TI2V

> 主题: 视频生成模型（含音视频同时生成）的数据处理：数据清洗流程、数据分布、打标方式

[← 返回首页](../index.md)

**目录**: [基本信息](#基本信息) · [数据规模与分布](#数据规模与分布) · [清洗流程](#清洗流程) · [打标方式](#打标方式) · [音视频对齐](#音视频对齐) · [训练配合](#训练配合) · [效果对比](#效果对比)

## 基本信息

### 名称

Step-Video-T2V（阶跃视频，30B 文生视频基础模型）及其衍生 Step-Video-T2V-Turbo、Step-Video-TI2V

### 发布机构/公司

阶跃星辰（StepFun，Step-Video Team / 上海阶跃星辰智能科技有限公司）

### 发布时间（技术报告/论文/开源时间）

2025年2月17日同步开源推理代码与权重、2025年2月17日发布技术报告《Step-Video-T2V Technical Report: The Practice, Challenges, and Future of Video Foundation Model》（arXiv:2502.10248，后有 v2/v3 修订）；2025年2月18日官方对外正式宣布（与语音模型 Step-Audio 同批开源）。衍生模型 Step-Video-TI2V（图生视频）于 2025年3月17日开源，技术报告 arXiv:2503.11251。

### 类型（模型/数据集/工具链/评测基准）

模型（开源文生视频基础模型，30B 参数 DiT + 自研深度压缩 Video-VAE），同时附带评测基准 Step-Video-T2V-Eval（128 条中文 prompt、11 个类目）与开源推理代码。不是数据集、不是数据工具链——训练数据与数据处理代码均未发布。

### 开源程度（权重/代码/数据/pipeline各自是否开源）

属于「权重+代码+评测基准开源，数据与数据 pipeline 代码不开源」模式，但开源程度在同期国产模型中较高：
【权重】开源。Step-Video-T2V（30B）与蒸馏加速版 Step-Video-T2V-Turbo 权重均在 GitHub（stepfun-ai/Step-Video-T2V）、Hugging Face（stepfun-ai/stepvideo-t2v、stepvideo-t2v-turbo）与 ModelScope 发布；自研 Video-VAE（16×16 空间、8× 时间压缩）与双语文本编码器一并开放。
【代码】开源推理代码（含多 GPU 并行推理、xDiT/ComfyUI 集成），训练代码未开放。
【许可证】MIT License（相较腾讯混元社区协议、通义万相等自定义协议更宽松，是其开源度的一大亮点）。
【评测基准】开源 Step-Video-T2V-Eval，含 128 条 prompt 及多个开闭源引擎的生成结果视频，可直接复现对比。
【数据】不开源。2B 视频-文本对与 3.8B 图文对均为内部数据，未发布任何子集或清单。
【pipeline】方法论披露较完整（六阶段流水线、各阶段所用的具体开源工具与模型名称几乎全部点名，如 PySceneDetect AdaptiveDetector、LAION 美学预测器、LAION NSFW 检测器、EfficientNet 水印分类器、PaddleOCR、Laplacian 方差、FFmpeg cropdetect、Farneback 光流、内部 VideoCLIP），但过滤阈值数值、自研 caption VLM 与 VideoCLIP 权重、pipeline 代码均未公开，不可直接复现。

### 是否支持音视频同时生成，以及实现方式（原生联合/级联/MoE融合）

不支持。Step-Video-T2V 与 Step-Video-TI2V 均为纯视觉视频生成模型，输出无音轨；技术报告全文不涉及音频模态，数据 pipeline 中也不含任何音轨处理环节（切分后仅保留视觉帧）。
阶跃星辰的音频能力由完全独立的模型线承担：与 Step-Video-T2V 同日开源的 Step-Audio（产品级开源语音交互模型，支持情绪、方言、语种、歌声与个性化风格），以及后续的 Step-Audio 系列。二者之间没有联合训练、没有共享 latent、没有联合去噪，属于典型的「视频模型 + 语音模型」双线并行而非级联或原生联合的 AV 生成。
因此本条目在音视频联合生成维度上不构成参考样本，本调研中所有音频相关字段（audio_category_distribution、joint_av_caption_schema、dialogue_transcription_attributes、av_sync_detection、sync_metric_and_threshold、temporal_vs_semantic_sync、audio_quality_filtering、audio_type_handling）均不适用。

### 调研信息来源列表（论文/技术报告/官方文档/新闻的URL，标注每条来源的性质：官方一手/同团队旁证/第三方报道）

- 【官方一手】Step-Video-T2V Technical Report: The Practice, Challenges, and Future of Video Foundation Model, Step-Video Team（阶跃星辰）, arXiv:2502.10248, 2025-02（含 Section 7 Data：视频切分/质量评估/运动评估/打标/概念均衡/视频文本对齐六阶段，Figure 11 分层过滤示意，训练阶段配置表）: https://arxiv.org/abs/2502.10248
- 【官方一手】Step-Video-T2V 技术报告 ar5iv 全文（数据章节可检索，含 7 类质量标签、Farneback 光流三指标、12 万+ K-means 簇、8 帧 CLIP Score、30M SFT、95k 蒸馏样本、644M/27.3M seen samples 等数值）: https://ar5iv.labs.arxiv.org/html/2502.10248
- 【官方一手】Step-Video-T2V 技术报告 arXiv HTML v1: https://arxiv.org/html/2502.10248v1
- 【官方一手】Step-Video-T2V 技术报告 arXiv HTML v2（含分层过滤与 Figure 11 说明、Step-Video-T2V-Eval 128 prompt/11 类目、Video-DPO 人工偏好标注流程）: https://arxiv.org/html/2502.10248v2
- 【官方一手】Step-Video-T2V GitHub 开源仓库（权重、推理代码、Video-VAE、双语文本编码器、MIT 许可证、2025-02-17 发布）: https://github.com/stepfun-ai/Step-Video-T2V
- 【官方一手】Step-Video-T2V Hugging Face 模型卡: https://huggingface.co/stepfun-ai/stepvideo-t2v
- 【官方一手】Step-Video-T2V-Eval 评测基准数据集（128 条中文 prompt、11 类目、含多引擎对比生成结果）: https://huggingface.co/datasets/stepfun-ai/Step-Video-T2V-Eval
- 【同团队旁证】Step-Video-TI2V Technical Report, arXiv:2503.11251, 2025-03（含 5M 文本-图像-视频三元组、动漫数据占比 >80% 的配比失衡自述、光流运动分提取方法与阈值过滤、运动分作为可控条件、caption 模型微调强化运镜描述、Step-Video-TI2V-Eval 178+120 条）: https://arxiv.org/html/2503.11251v1
- 【同团队旁证】Step-Video-TI2V GitHub 开源仓库: https://github.com/stepfun-ai/Step-Video-TI2V
- 【第三方】The Moonlight 文献综述：Step-Video-T2V 技术报告解读（分层过滤与 6 个预训练子集的说明）: https://www.themoonlight.io/en/review/step-video-t2v-technical-report-the-practice-challenges-and-future-of-video-foundation-model
- 【第三方】Kingy AI：Step-Video-T2V 技术报告论文摘要（数据 pipeline 与 SFT/DPO 概述）: https://kingy.ai/blog/step-video-t2v-technical-report-paper-summary/
- 【第三方】Hugging Face Papers：Step-Video-T2V 论文页与社区讨论: https://huggingface.co/papers/2502.10248
- 【第三方】CSDN：Step-Video-T2V 阶跃星辰发布最强开源视频生成模型（论文详解，中文数据章节逐条拆解）: https://blog.csdn.net/sherlockMa/article/details/145706142
- 【第三方】CSDN：阶跃星辰的开源探索——Step-Video-T2V 与 Step-Audio 深度解析（说明二者为独立模型线，非联合 AV 生成）: https://blog.csdn.net/liaoqingjian/article/details/145820964
- 【第三方】知乎：阶跃星辰 30B 视频生成模型 Step-Video 简析: https://zhuanlan.zhihu.com/p/24619034131
- 【第三方】知乎：阶跃星辰开源 Step-Video-TI2V 图生视频模型介绍（102 帧/5 秒/540P、运动幅度可控与镜头运动可控）: https://zhuanlan.zhihu.com/p/31775732208
- 【第三方】智源社区：阶跃星辰首次开源 Step 系列多模态大模型（2025-02-18 官方发布报道）: https://hub.baai.ac.cn/view/43466
- 【第三方】NeuroHive：Step-Video-T2V 开源模型 16x 视频压缩突破解读: https://neurohive.io/en/state-of-the-art/step-video-t2v-text-to-video-open-source-model-achieves-16x-video-compression-breakthrough/

## 数据规模与分布

### 训练数据量级（视频条数/小时数/token数，预训练与SFT分开）

【预训练总量】技术报告明确给出：构建了包含 20 亿（2B）视频-文本对 与 38 亿（3.8B）图文对 的大规模数据集。这是本条目最硬的定量披露之一。
【各训练阶段实际消耗样本】
· Step-1 文生图（T2I）预训练：256px，约 3.8B 图像样本，253k 迭代；
· Step-2 文生图/视频（T2VI）联合预训练低分辨率档：192×320，约 10 亿（1B）视频片段可用，实际 seen 样本 6.44 亿（644M），430k 迭代；
· Step-2 高分辨率档：544×992，实际 seen 样本 2730 万（27.3M），46k 迭代；
· Step-3 文生视频（T2V）微调 / SFT：544×992，约 3000 万（30M）高质量视频；
· Step-4 Video-DPO：544×992，基于模型自生成视频构造偏好对；蒸馏（Turbo）数据集约 9.5 万（95k）样本。
【说明】原始视频池的小时数、切分前的原片条数、以及各级过滤的绝对淘汰量报告未给出（漏斗以 Figure 11 的相对条形图形式呈现，未标注绝对数值）。视频总时长（小时数）与 token 数均未披露。

### 数据来源构成（自有/公开数据集/网络爬取/授权采购/合成数据） ⚠️

报告未说明来源构成。仅表述为「将原始视频（raw videos）经完整 pipeline 转化为适合预训练的高质量视频-文本对」，未区分自有素材库、公开数据集、网络爬取、授权采购或合成数据的比例；也未列举任何具体数据源名称。从 pipeline 设计可反推的间接线索：(1) 明确保留并使用了视频的「原始标题（Original Title）」作为一路 caption 来源，说明大量数据来自带有标题元数据的网络视频平台内容；(2) 使用 EfficientNet 水印分类器与 PaddleOCR 字幕检测，说明原始池中含大量带台标/字幕的二次传播内容，符合网络爬取特征；(3) 衍生模型 Step-Video-TI2V 的训练数据中超过 80% 为动漫风格视频，说明团队有大规模动漫内容来源。图文侧 3.8B 的来源同样未披露。[不确定]

### 数据合规与溯源（授权数据占比、rights-cleared数据集、C2PA等） ⚠️

未披露。技术报告全文未涉及授权数据占比、rights-cleared 数据集、版权清算流程、C2PA 等内容溯源标准，也未提及生成内容水印/标识。数据合规维度上唯一可见的痕迹是 pipeline 中的 NSFW 打分（LAION CLIP-based NSFW 检测器）与水印检测（用于剔除带水印素材，动机更接近画面清洁与规避明显版权标记，而非系统性版权合规）。模型侧采用 MIT 协议开源，不含数据溯源承诺。作为中国境内发布的产品，实际生产必然存在内容安全与合规审核，但训练数据侧的合规方法零披露。[不确定]

### 片段时长分布与切分策略

未给出时长直方图，但切分与分桶策略明确：
【切分】用 PySceneDetect 的 AdaptiveDetector 函数检测场景变化，再由 FFmpeg 切分为单镜头（single-shot）片段；每个切出的片段还会额外丢弃首尾各 3 帧，以消除转场处不稳定的镜头运动与残留过渡帧——这是一个细颗粒但实用的工程细节，同期不少工作未做。
【时长表达】训练侧以帧数而非秒数表达，采用帧长分桶（frame-length bucketization）：1 帧、68 帧、136 帧、204 帧四档（1 帧档即图像，用于图像-视频联合训练）。模型最长单次生成 204 帧；按报告口径 204 帧对应约 8 秒级（衍生模型 Step-Video-TI2V 为 102 帧/5 秒，可反推约 20fps 左右口径）。
即课程设计上把「短→长」做成了显式的帧数分桶轴，而非固定 clip 时长。

### 分辨率/宽高比分布与分桶策略

【分辨率】作为课程主轴：图像阶段 256px → 视频阶段 192×320（192P）→ 544×992（540P）。最终发布模型输出 540P，未训练到 720P/1080P，这是其与同期混元/万相的显著取舍差异（团队把算力投在了 204 帧长序列与深度压缩 VAE 上）。
【宽高比】采用宽高比分桶（aspect-ratio bucketization），分为横屏（landscape）、竖屏（portrait）、方形（square）三组，与帧长分桶（1/68/136/204）组合形成二维 bucket 体系，支持多分辨率多宽高比混训。
【黑边处理】pipeline 中用 FFmpeg 检测黑边（black border）尺寸并据此裁剪，保证入桶前画面无 padding 边框。
各宽高比桶的具体占比未公布。

### 类别/domain分布与配比策略（人物、动作、场景、风格等比例控制与概念均衡） ⚠️

报告未给出 domain 占比数字，但提供了一个在同期工作中相当有辨识度的概念均衡机制——「Video Concept Balancing（视频概念均衡）」：
· 用内部自研 VideoCLIP 模型对每个片段计算视频 embedding；
· 在高维 embedding 空间做 K-means 聚类，聚成 超过 12 万（120,000+）个簇 —— 这个簇数远高于同期同类工作（如 HunyuanVideo 的约 1 万个概念中心），意味着概念划分粒度极细；
· 为每个片段打两个派生标签：Cluster_Cnt（所属簇的样本数，用于识别过密/长尾概念）与 Center_Sim（到簇中心的余弦距离，用于识别簇内离群样本）；
· 基于这两个标签实现两类操作：其一，按簇大小做重采样，保证覆盖广泛类别、抑制头部概念过度集中；其二，在后训练阶段按 Center_Sim 剔除距簇中心过远的离群片段（把聚类同时用作「概念均衡」与「离群质检」两用，是本条目较有方法论价值的一点）。
各具体类目（人物/动作/场景/风格）的最终配比、以及重采样的具体倍率均未公布。Step-Video-TI2V 侧则有明确的风格失衡披露：训练数据中动漫风格视频占比超过 80%，且早期阶段仅用动漫数据，导致该模型动漫表现强而真实场景表现受限——这是一个罕见的、由官方自述的 domain 配比失衡及其后果案例。[不确定]

### 音频类别分布与配比（语音/音效foley/音乐/环境音/静音的比例及控制策略）——AV模型独有维度 ⚠️

不适用。Step-Video-T2V 不生成音频，训练数据 pipeline 完全不涉及音轨的提取、分类与配比，报告中无语音/音效/音乐/环境音/静音的任何比例控制内容。若需阶跃星辰体系内的音频数据方法，应转向独立的 Step-Audio 系列（语音交互模型，含情绪、方言、语种、歌声等维度的数据构建），但其与视频模型无数据共用关系。[不确定]

### 叙事结构分布（单镜头vs多镜头、平均clip时长、镜头数分布、是否含原生音轨）

明确采用「单镜头（single-shot）」数据范式，不做多镜头叙事训练：PySceneDetect AdaptiveDetector 检测场景变化后由 FFmpeg 切分，每段仅含一个镜头，且首尾各去 3 帧进一步剔除转场残留。因此镜头数分布恒为 1，平均 clip 长度对应 68/136/204 帧三档（外加 1 帧图像档）。
值得注意的是，dense caption 中会显式描述「镜头运动（camera movements）」，SFT 环节的人工评审标准中也包含「场景转换是否平滑（smooth scene transitions）」一项，说明团队关注镜头语言，但这是在单镜头内部的运镜层面，而非跨镜头的多镜头叙事结构。
原生音轨：无（数据侧不保留、不处理音轨）。

### 语言/口音分布（多语种唇同步能力的数据基础） ⚠️

不适用于唇同步（无音频生成）。文本侧是本条目的一个特色：模型采用两个双语（中英）文本编码器（Hunyuan-CLIP 与自研 Step-LLM 的双编码器组合），原生支持中英双语 prompt 输入，官方将「原生中英双语输入」列为核心能力之一；配套的评测基准 Step-Video-T2V-Eval 全部 128 条 prompt 为中文，也印证其数据与评测的中文侧重。但训练 caption 的中英文语种构成比例、以及是否对中文 caption 做专门增强，报告均未公布。[不确定]

## 清洗流程

### 清洗漏斗整体结构（几级过滤、各级顺序）

核心价值所在。Step-Video-T2V 的数据 pipeline 由六个串行阶段构成，且每个阶段所用的工具/模型基本全部点名，可复现性描述在同期工作中属较好一档：
第1阶段 视频切分（Video Segmentation）：PySceneDetect 的 AdaptiveDetector 检测场景变化 → FFmpeg 切为单镜头片段 → 每片段丢弃首尾各 3 帧。
第2阶段 视频质量评估（Video Quality Assessment）：对抽样帧打 7 类质量标签（美学分、NSFW 分、水印、字幕、饱和度、模糊度、黑边），详见 quality_filtering 字段。
第3阶段 视频运动评估（Video Motion Assessment）：Farneback 光流算法计算 Motion_Mean / Motion_Max / Motion_Min 三个运动量。
第4阶段 视频打标（Video Captioning）：自研 VLM 产出三类文本——Short Caption、Dense Caption、Original Title。
第5阶段 概念均衡（Video Concept Balancing）：内部 VideoCLIP 抽 embedding → K-means 聚成 12 万+ 簇 → 打 Cluster_Cnt 与 Center_Sim 标签 → 重采样均衡与离群剔除。
第6阶段 视频-文本对齐（Video-Text Alignment）：从片段中均匀抽 8 帧，计算帧 embedding 与文本 embedding 的平均余弦相似度得到 CLIP Score，用于剔除图文错配样本。

【分层过滤（Hierarchical Data Filtering）——本 pipeline 的组织方式】上述阶段产出的是一整套「标签体系」而非一次性丢弃：所有片段先被打满标签，再通过逐级抬高各标签阈值，切出 6 个用于 Step-2 T2VI 预训练的子集（Figure 11 以条形图展示各级过滤：灰色条为被该级滤掉的数据，彩色条为留存数据）。这种「先全量打标、后按阈值切档」的设计相比「边过滤边丢弃」更灵活，可在不重跑 pipeline 的前提下调整课程。
后训练阶段在此之上再叠加两级：自动化脚本+启发式规则+按簇心距离剔离群 → 人工评审精选，得到 30M 的 SFT 集。

### 漏斗定量保留率（每级过滤的输入/输出量与最终保留率，如Apollo 27%） ⚠️

定量披露不完整，只能部分推算：
【已知绝对量】原始池 → 2B 视频-文本对（pipeline 产出的总量）→ Step-2 低分辨率档可用约 1B 片段（实际 seen 644M）→ 高分辨率档 seen 27.3M → SFT 30M → DPO/蒸馏约 95k。
【可推算的保留率】2B → 1B（192P 档）约 50%；2B → 30M（SFT 高质量档）约 1.5%，即端到端从 pipeline 产出到 SFT 的保留率约 1.5%（此换算为本调研推算，报告未直接给出该比值）。
【缺失部分】(1) 原始视频池（切分前）的绝对规模未公布，因此「原片 → 2B 对」这一段的保留率无从计算；(2) Figure 11 的分层漏斗图只以条形长度示意各级过滤的去留比例，未标注任何绝对数值或百分比；(3) 6 个预训练子集各自的样本量未列出。
因此本条目的漏斗定量完整度低于 HunyuanVideo 1.5（后者给出了 >1000万小时 → 8亿 → 2亿 → 1亿 → 100万 的完整链条），可比的只有 SFT 端的 1.5% 量级。[不确定]

### 镜头切分方法（PySceneDetect/自研模型/shot-aware splitting）

PySceneDetect 的 AdaptiveDetector 函数（自适应阈值检测器，相对 ContentDetector 对渐变与摄像机运动更鲁棒）检测场景变化点，再调用 FFmpeg 按检测到的边界切分为单镜头片段。切分后统一去掉每个片段的首 3 帧与尾 3 帧，理由是这些帧常包含不稳定的镜头运动（转场残留、抖动）。
与同期工作的对比：未使用 TransNetV2 等双路交叉验证（HunyuanVideo 用了），也未额外训练「转场分类器」做二次清洗（HunyuanVideo 1.5 用了），属于较标准的单工具方案；但「首尾各去 3 帧」这一补偿性设计是其特点。

### 质量过滤（美学评分、清晰度、OCR文字过滤、黑边/水印/logo检测）

第2阶段对每个片段的抽样帧打 7 类质量标签，工具与方法几乎全部点名（阈值数值一律未公布，只说明「逐级抬高阈值」构造 6 个子集）：
1. 美学分（Aesthetic Score）：LAION CLIP-based aesthetic predictor（开源 LAION 美学预测器），评估画面视觉吸引力；
2. NSFW 分（NSFW Score）：LAION CLIP-based NSFW 检测器，基于 CLIP ViT-L/14 的二分类器；
3. 水印检测（Watermark Detection）：基于 EfficientNet 的图像分类模型，判定帧内是否含水印；
4. 字幕/文字检测（Subtitle Detection）：PaddleOCR 识别帧内文字，用于剔除字幕/文字过多的片段；
5. 饱和度分（Saturation Score）：在 HSV 色彩空间统计饱和度的均值/最大值/最小值，剔除过曝、过灰或过饱和的异常画面；
6. 模糊分（Blur Score）：Laplacian 方差法（Laplacian variance）度量画面锐度，剔除失焦/模糊片段；
7. 黑边检测（Black Border Detection）：FFmpeg 检测黑边尺寸，据此做裁剪（裁剪而非丢弃，与 HunyuanVideo 1.5 的思路一致）。
方法论特点：这套质检器全部是「传统 CV 算子 + 开源小分类器」的组合，无一为大模型语义判断，属于典型的 2024–2025 年初「浅层多打分器」范式；优点是工具全部点名、成本低、可复现性描述好，缺点是无法捕捉语义层面的质量问题（如物理不合理、主体畸变）。

### 运动过滤（光流/运动分数阈值、静态与抖动剔除）

第3阶段专设「视频运动评估（Video Motion Assessment）」：使用 Farneback 稠密光流算法（OpenCV 经典算法）计算片段的光流场，导出三个标量指标——Motion_Mean（平均运动幅度）、Motion_Max（峰值运动幅度）、Motion_Min（最小运动幅度）。三值组合用于识别并剔除两类样本：几乎静止的片段（运动信息量不足、易导致模型生成静态画面）与运动过于剧烈/抖动的片段。具体阈值未公布。
衍生模型 Step-Video-TI2V 把这套运动打分从「过滤器」升级为「可控条件」，方法细节也更明确：每隔 12 帧采样一次，转灰度图后计算光流幅值，取幅值最高的一批数值再求均值作为该视频的运动分；对全部训练数据提取运动分后，一方面设阈值滤掉运动过高或过低的视频，另一方面把运动分作为显式条件注入模型，实现推理时「运动幅度可控」。这是「数据标签直接转化为可控条件」的一个清晰案例，对本调研有参考价值。

### 去重方法（精确去重与基于embedding的语义去重分别记录） ⚠️

技术报告未描述任何去重环节——六阶段 pipeline 中没有独立的去重步骤，也未提及感知哈希（pHash）等精确/近重复去重，或基于 embedding 的语义去重。
唯一在功能上部分承担去重效果的是第5阶段的概念均衡：VideoCLIP embedding 的 K-means（12 万+ 簇）配合 Cluster_Cnt（簇内样本数）做重采样，可以抑制高度相似内容的过度集中，并通过 Center_Sim 剔除簇内离群样本；但这本质是分布均衡与离群检测，不等价于重复样本剔除。这是本条目相对 HunyuanVideo（明确用 VideoCLIP 余弦距离做语义去重）的一处披露空白。[不确定]

### VLM/LLM作为质检员（多模态大模型质量打分与错配剔除，2026年从浅层打分器转向大模型语义判断的趋势） ⚠️

本条目在这一维度上明显偏「传统打分器」而非「大模型语义评判」，是观察 2025 年初到 2026 年范式迁移的一个基线样本：
【承担质检的模型清单】全部为专用小模型或经典算法——LAION CLIP 美学预测器、CLIP ViT-L/14 NSFW 二分类器、EfficientNet 水印分类器、PaddleOCR、Laplacian 方差、HSV 统计、FFmpeg 黑边检测、Farneback 光流、内部 VideoCLIP（聚类与离群检测）、CLIP Score（图文对齐）。没有任何一个环节使用多模态大模型对视频做端到端的语义质量判断或物理合理性判断。
【大模型的位置】自研 VLM 只出现在打标环节（生成 short/dense caption），不承担判分与剔除职责；对该 caption VLM 本身也未做幻觉抑制的专门后训练（对比 HunyuanVideo 1.5 用 OPA-DPO 治理标注器幻觉）。
【唯一的模型级错配剔除】第6阶段的 CLIP Score 图文对齐过滤，属于「用判别式对比模型做粗粒度语义匹配」，是浅层语义判断而非 VLM 推理式判断。
【人工替代了大模型的角色】SFT 阶段的语义级质量判断（清晰度、美感、运动合理性、场景转换平滑度、caption 准确性）由人工评审完成，而非交给 VLM——这正是 2026 年趋势要替换掉的部分。因此 Step-Video-T2V 可作为「大模型质检员范式之前」的对照样本。[不确定]

### 安全与合规过滤（NSFW、版权、人脸/隐私） ⚠️

披露很弱，只有一项：pipeline 第2阶段的 NSFW 打分，使用 LAION 提供的基于 CLIP ViT-L/14 的二分类 NSFW 检测器对抽样帧判分，据此过滤色情/不适宜内容。除此之外，报告未提及版权过滤、人脸/隐私保护（人脸打码、肖像权处理）、暴力血腥内容分类、政治敏感内容过滤、名人肖像剔除等任何环节，也未描述生成侧的安全对齐或输出审核。水印检测虽可间接剔除部分明显带版权标记的素材，但其设计动机是画面清洁而非合规。
作为中国境内商用产品（跃问 yuewen.cn 上线），实际生产系统必然存在完整的内容安全审核链路，但技术报告在数据侧的安全过滤上几乎零披露。[不确定]

## 打标方式

### 使用的caption模型（自研VLM/开源模型，模型规模） ⚠️

使用自研内部视觉语言模型（in-house Vision Language Model）作为 caption 模型，为每个视频片段生成描述。报告未披露该 VLM 的名称、参数规模、基座架构、训练数据或训练方式，也未说明是否基于阶跃星辰自家的 Step-1V 多模态模型改造。未对 caption 模型做幻觉抑制方面的专门后训练（无 DPO/RLHF 类描述），caption 质量的兜底依赖 SFT 阶段的人工复核（人工会「优化 caption」）与第6阶段的 CLIP Score 对齐过滤。
衍生模型 Step-Video-TI2V 披露了一步增量：对内部视频打标模型做了微调（fine-tuned an in-house video captioning model），专门强化对「物体运动动态」与「镜头运动」的描述能力，给出的示例 caption 形如「a flock of birds flying over a tree at sunset, camera pans left」——即让 caption 显式携带运镜指令，与 I2V 的可控运镜设计配套。[不确定]

### caption密度与结构化程度（短/长/密集描述、结构化字段如镜头运动、风格标签）

采用「三路并行 caption」而非单一密集描述，是本条目在打标维度上最值得记录的设计：
1. Short Caption（短描述）：极简，仅聚焦主体与动作（focusing solely on the main subject and action）；
2. Dense Caption（密集描述）：详述关键元素，强调主体、事件、环境与视觉表现，并明确包含镜头运动（camera movements）的描述；
3. Original Title（原始标题）：直接沿用视频自带的原始标题文本，团队给出的理由是引入「风格多样性（stylistic diversity）」——即保留人类真实撰写的、带口语化/标题党/风格化色彩的文本分布，防止模型只见过 VLM 生成的规整句式，从而在推理时更能适配真实用户的随意 prompt。这一点与 HunyuanVideo 用 caption 字段 dropout + 排列组合做增广的目的相同（对齐推理期 prompt 分布），但实现路径不同——阶跃用的是「引入真实人类文本源」，混元用的是「结构化字段的随机重组」。
【结构化程度评价】相比 HunyuanVideo 的七字段 JSON schema（含 Style、Shot Type、Lighting、Atmosphere 等独立字段），Step-Video-T2V 的 caption 是自然语言长短文本而非强字段化 schema，风格/光照/氛围等属性隐含在 dense caption 的自由文本中，未拆成独立可控字段。这限制了训练时按字段做条件控制的能力（其可控性主要通过 TI2V 的运动分标量实现）。
三路 caption 在训练时如何混用/采样（比例、随机切换策略）报告未说明。

### 音视频联合caption结构（是否同时覆盖视觉+听觉轨道、是否分流为独立字段，如LTX-2全音景描述、Script-a-Video factorized streams、Foley-Omni三字段） ⚠️

不适用。模型无音频模态，caption 只覆盖视觉轨道（主体、动作、环境、视觉表现、镜头运动），不存在视觉+听觉双轨 caption 结构，也不存在音景描述、说话人字段或音效字段。可作为「纯视觉 caption」的对照基线：其三路 caption 设计（短/密集/原始标题）中「保留原始标题以引入真实人类文本风格分布」的思路，理论上可正交迁移到 AV caption 体系（如保留视频原始标题与人工描述作为音景 caption 的风格多样性来源），但报告本身无任何 AV 相关内容。[不确定]

### 对白转写与说话人属性标注（ASR转写+说话人身份/语言/口音/情绪） ⚠️

不适用。数据 pipeline 不处理音轨，无 ASR 转写，无说话人身份/语言/口音/情绪标注。阶跃星辰体系内这类能力在独立的 Step-Audio 系列语音模型中（该模型支持情绪、方言、语种、歌声等属性的语音生成），但与 Step-Video-T2V 的训练数据无任何交集。[不确定]

### 几何与结构化标注（相机参数、深度、3D point tracks、动作标注、显式状态）

结构化标注较少，主要是两类标量与一类文本：
【运动标量】Farneback 光流导出的 Motion_Mean / Motion_Max / Motion_Min 三个数值标签（预训练用于过滤；在 Step-Video-TI2V 中升级为注入模型的显式可控条件，实现运动幅度可控）；
【聚类标量】VideoCLIP embedding 的 Cluster_Cnt 与 Center_Sim；
【镜头运动】以自然语言形式写入 dense caption（如「camera pans left」），而非离散标签枚举——这与 HunyuanVideo 训练了 14 类镜头运动分类器输出离散标签的做法不同；Step-Video-TI2V 通过微调 caption 模型强化运镜描述来实现「镜头运动可控」，仍走自然语言路径。
【完全缺失】未使用相机内外参估计、深度图、3D point tracks、光流场本身作为监督信号、动作骨架、显式物理状态等几何结构化标注。这与 Movie Gen、Seedance 等引入更强几何/结构监督的路线形成对比，也是团队在报告「Challenges and Future」章节中自陈的短板之一（复杂物理与因果建模能力不足）。

### 合成数据构造（受控扰动/编辑构造训练对，如InstructAV2AV） ⚠️

训练语料侧未使用合成数据构造（无受控扰动/编辑构造训练对，如 InstructAV2AV 那类做法）。带「构造」性质的只有后训练两处，均为模型自采样而非语料合成：
· Video-DPO 偏好数据：对每条 prompt 用不同随机种子让 Step-Video-T2V 生成多个视频，组成候选集供人工偏好排序——属于模型自生成的偏好对；
· 蒸馏数据：为训练加速版 Step-Video-T2V-Turbo，采样约 9.5 万（95k）条数据构成蒸馏数据集。
此外 prompt 侧有轻度合成：DPO 的 prompt 集除从训练数据中随机抽取外，还邀请人工标注员按精心设计的指引「合成（synthesize）」新 prompt，以扩大 prompt 分布覆盖。[不确定]

### 人工介入程度（人工标注、人工质检、模型初筛+人工复核）

人工被集中投放在漏斗最末端（预训练全自动、后训练重人工），分三处：
1. SFT 数据人工精选：30M 高质量视频集的构建为「自动化 + 人工」两段式——先用各项评估分数与启发式规则自动过滤，再按视频类目（簇）剔除距簇中心超阈值的离群样本，最后由人工标注员逐条评审，评审维度为：画面清晰度（clarity）、美学质量（aesthetics）、运动是否恰当（appropriate motion）、场景转换是否平滑（smooth scene transitions）；人工同时会对 caption 做优化/改写（caption refinement），即人不仅筛数据也修标注。
2. DPO 偏好标注：人工标注员按设计指引合成 prompt；对同一 prompt 不同种子生成的多个视频进行偏好打分排序；全流程由质控人员（quality control personnel）监督以保证标注的准确性与一致性。
3. 评测人工评审：在 Step-Video-T2V-Eval 上以人工评测为主要评估手段，对比多个开源与商用引擎。
未披露标注团队规模、人时成本或标注一致性指标（如 Kappa 值）。报告在展望部分明确指出 DPO 的收益会在「模型能轻易区分正负样本时饱和」，并提出未来改用 reward model 对新生成样本动态打分来替代纯人工标注——即团队已意识到人工偏好标注的可扩展性瓶颈。

## 音视频对齐

### 音视频同步检测方法（口型同步、事件对齐） ⚠️

不适用。模型无音频模态，训练数据不含音轨，pipeline 中不存在音视频同步检测环节，报告全文无 SyncNet、AV-align、唇同步、事件对齐等任何相关内容。[不确定]

### 同步检测具体指标与阈值（SyncNet/Synchformer/LSE/自研，阈值数值，如MOVA LSE-D≤9.5且LSE-C≥4.5、SkyReels-V4 SyncNet |offset|≤3∧conf>1.5） ⚠️

不适用。未使用任何音视频同步指标，无相应阈值。可作为对照说明的是，本条目在「跨模态对齐」上的唯一手段是视觉-文本对齐：从片段中均匀抽取 8 帧，分别计算帧 embedding 与 caption 文本 embedding 的余弦相似度并取平均，得到 CLIP Score 作为图文对齐度标签，用于过滤错配样本——但该阈值的具体数值同样未公布。[不确定]

### 时序同步vs语义同步的分离处理（时间对齐与内容语义匹配作为两个独立过滤条件） ⚠️

不适用（无音频）。在纯视觉侧存在一个结构上的类比：pipeline 把「时序/运动维度」（第3阶段 Farneback 光流的三项运动分）与「语义匹配维度」（第6阶段 CLIP Score 图文对齐）设为两个相互独立的过滤条件，分别处理「动得对不对」和「拍的是不是描述的东西」，这与 AV 模型将时序同步与语义匹配拆成两个独立门限的思路同构，可作为方法论层面的弱参考。[不确定]

### 音频质量过滤（SNR、静音检测与静音占比阈值、无音轨剔除、画外音源剔除、背景音乐分离） ⚠️

不适用。数据 pipeline 不提取、不保留、不处理音轨，无 SNR 计算、静音检测、无音轨剔除、画外音源剔除或背景音乐分离等任何环节。[不确定]

### 语音/音效/音乐的分类与分别处理策略 ⚠️

不适用。无语音/音效/音乐的分类与分别处理策略。阶跃星辰的语音侧能力封装在独立的 Step-Audio 模型中，与本条目的视频数据 pipeline 无交集。[不确定]

## 训练配合

### 多阶段训练课程与数据课程调度（阶段划分依据：分辨率/时长/质量分/模态；低清→高清、图像→视频、短→长）

采用明确的四步级联训练（cascaded training pipeline），课程轴同时覆盖模态、分辨率与帧长，且团队明确指出其目的是「加速收敛并充分利用不同质量的视频数据集」——即课程台阶与数据档位一一对应：
· Step-1 文生图（T2I）预训练：256px，约 3.8B 图像，253k 迭代。建立基础视觉先验与文本对齐；
· Step-2 文生图/视频（T2VI）联合预训练：分两档递进——先 192×320（192P），约 1B 视频片段可用、seen 644M、430k 迭代；再 544×992（540P），seen 27.3M、46k 迭代。全程图像与视频联合训练（T2VI 即 text-to-video-and-image），用 1 帧的帧长桶承载图像样本以防图像能力退化；
· Step-3 文生视频（T2V）微调 / SFT：544×992，30M 高质量视频；
· Step-4 Video-DPO：544×992，基于人工偏好数据做直接偏好优化。
【分桶】全程配合帧长分桶（1/68/136/204 帧）与宽高比分桶（横/竖/方），实现「短→长」的帧长课程与多宽高比混训。
【团队自述的关键经验】(1) 低分辨率阶段（192P）越稳定、用的数据越多样，后续向高分辨率扩展就越容易——即把「多样性」压在低清阶段、把「精度」压在高清阶段；(2) 观察到训练损失会随训练数据质量的提升出现「骤降（sudden drop in loss）」，团队据此把数据质量的阶跃直接当作课程切换的信号，这是本报告一个颇具实操价值的经验性发现；(3) 在 checkpoint 选择上，模型权重平均（checkpoint averaging）优于 EMA，且应选在梯度范数峰值之后、梯度范数与损失均已下降的 checkpoint。

### 数据配比随训练阶段的变化（预训练/退火annealing/SFT高质量子集） ⚠️

配比策略以「同一套标签、逐级抬高阈值」的方式表达，而非显式的类别配比数字：
· Step-2 T2VI 预训练阶段使用由分层过滤切出的 6 个子集，各子集的过滤阈值逐级收紧（从宽松到严格），对应从 192P 到 540P 的推进——即「早期用量大质杂、后期用量小质精」的退火式调度；
· 图像与视频全程联合训练（T2VI），图像以 1 帧桶参与，但图像:视频的具体混合比例未公布；
· Step-3 SFT 切换到 30M 高质量子集（相对 2B 总量约 1.5%）；
· Step-4 DPO 使用规模远小的人工偏好对；
· 蒸馏版 Turbo 使用约 95k 样本。
【定量缺口】6 个预训练子集各自的样本量、各阶段的图像/视频混合比、以及重采样后各概念簇的目标分布均未披露。[不确定]

### 后训练数据（SFT精选集规模与筛选标准、偏好对数量与标注方式、reward model训练数据）

后训练分两段，数据构建方式披露相对清晰：
【SFT（Step-3）】约 3000 万（30M）高质量视频，构建三步走：(1) 按 pipeline 各项评估分数与启发式规则自动过滤；(2) 按视频类目（VideoCLIP K-means 簇）剔除距簇中心距离超过阈值的离群样本（阈值数值未公布）；(3) 人工标注员逐条评审，标准为清晰度、美学、运动是否恰当、场景转换是否平滑，并对 caption 做优化。
【Video-DPO（Step-4）】(1) prompt 集构建：从训练数据中随机抽取一部分 prompt，再邀请人工标注员依照精心设计的指引合成补充 prompt，以保证 prompt 多样性；(2) 响应生成：对每条 prompt，Step-Video-T2V 用不同随机种子生成多个视频；(3) 偏好标注：人工标注员对这些样本做偏好评分，全程由质控人员监督一致性；(4) 用 DPO loss 更新模型以偏好被选中样本。prompt 条数、生成视频总数、标注员人数均未公布。
【reward model】未训练 reward model。报告在展望中明确指出当前 DPO 的局限——当模型能轻易区分正负样本时收益即饱和——并提出未来引入 reward model 对新生成样本动态打分以持续提供有效梯度信号。
【蒸馏】Step-Video-T2V-Turbo 使用约 9.5 万（95k）样本的蒸馏数据集。

### 数据处理基础设施与吞吐（NeMo Curator/Data-Juicer/自研，GPU加速比，处理规模，成本） ⚠️

数据处理基础设施与吞吐未披露。报告未提及 NeMo Curator、Data-Juicer 或任何自研数据处理框架的名称，未给出 GPU 加速比、处理吞吐（clip/小时）、集群规模或处理成本。可反推的工程压力：要产出 2B 视频-文本对，需要对更大规模的原始视频完成 PySceneDetect 切分 + 7 类质量打分 + Farneback 光流 + 自研 VLM 三路 caption 推理 + VideoCLIP embedding + 12 万簇 K-means + 8 帧 CLIP Score 计算，其中 VLM caption 与 VideoCLIP 推理是十亿级样本的多模型批量推理任务，工程量极大，但报告完全未着墨。
训练侧基础设施有较多披露（自研 Step Emulator 训练框架/StepRPC 通信库、显存优化、并行策略、多阶段训练稳定性经验、checkpoint averaging 优于 EMA 等），但不属于数据处理吞吐范畴。[不确定]

## 效果对比

### 数据策略消融的量化影响（区分：过滤严格度ablation / caption密度风格ablation / 数据配比ablation，及对应评测指标） ⚠️

技术报告未提供针对数据策略的受控消融实验——既无过滤严格度 ablation（如比较 6 个子集不同阈值档对最终指标的影响），也无 caption 密度/风格 ablation（如 short caption vs dense caption vs 原始标题三路的单独贡献），也无数据配比 ablation（如概念均衡开/关、图像视频混合比的对比）。这是本条目最主要的方法论短板：pipeline 描述工具级详尽，但每一步的必要性与收益缺乏量化归因。
报告提供的是两类替代性证据：
(1) 定性的训练曲线观察——「随着训练数据质量提升，损失出现骤降（sudden drop in loss）」。这是关于数据质量影响的唯一直接经验证据，但只是训练损失层面的定性现象，未换算为下游生成质量指标，也未做对照组；
(2) 端到端人工对比评测——在自建的 Step-Video-T2V-Eval（128 条中文 prompt、11 类目）上与多个开源及商用引擎人工对比，在指令遵循、运动流畅度、物理合理性、美感等维度综合胜出；另在 Movie Gen Bench 上做了对比。这些结果无法归因到具体数据策略。[不确定]

### 质量vs数量的证据（小而精数据超越大而杂的案例） ⚠️

无严格的受控对照实验，但存在一条较强的间接证据与一条明确的反面案例：
【正面间接证据】数据量从 pipeline 产出的 2B 单调收缩到 SFT 的 30M（约 1.5%），配合团队观察到的「训练数据质量提升时损失骤降」现象，构成「后期小而精数据带来显著增益」的经验性支持；四步级联训练的整体设计理由本身即是「充分利用不同质量的视频数据集」——把低质量大数据用在低分辨率早期阶段建立多样性先验，把高质量小数据用在高分辨率后期阶段抛光，属于「质与量各司其职」而非简单的质量优先。
【反面案例（更有价值）】衍生模型 Step-Video-TI2V 官方自述：训练数据中动漫风格视频占比超过 80%，且初期阶段仅用动漫数据，结果是动漫场景表现显著受益、而真实世界场景能力受限。这是一个由官方承认的「数据配比失衡直接导致能力偏科」的定性案例，从反面印证了数据分布（而非单纯数据量）对最终能力边界的决定性作用。
【缺口】没有「同模型、同算力、不同数据档」的受控对比实验，因此严格意义上不构成实验证明。[不确定]

### 训练数据domain分布与评测基准类目体系的对齐关系（如VABench七大类） ⚠️

训练数据的类目体系与评测基准类目体系之间未做显式对齐说明，但两侧的类目信息都相对具体，可作对照分析：
【评测侧】自建 Step-Video-T2V-Eval 含 128 条 prompt，覆盖 11 个类目：体育（Sports）、美食（Food）、风景（Scenery）、动物（Animals）、节日（Festivals）、组合概念（Combination Concepts）、超现实（Surreal）、人物（People）、3D 动画（3D Animation）、镜头语言/电影摄影（Cinematography）、风格（Style）。该类目体系既含内容主题类（体育/美食/风景/动物/节日/人物），也含能力压力测试类（组合概念、超现实、镜头语言），后者明显是为探测模型弱项而设计。衍生的 Step-Video-TI2V-Eval 则含 178 条真实风格与 120 条动漫风格的图文对，细分为 I2V 主体、背景、镜头运动、动作、风格与色彩等子类。
【训练侧】数据类目由 VideoCLIP embedding 的 12 万+ K-means 簇隐式定义，是无监督聚类得到的细粒度概念空间，与上述 11 个人工定义的评测类目之间没有映射关系，报告也未说明是否按评测类目反向调整训练数据配比。
【结论】两套类目体系各自独立、粒度差异极大（12 万无监督簇 vs 11 个人工类目），未见对齐设计。这与 VABench 七大类那种「训练分布与评测类目显式对齐」的思路不同。[不确定]

## 其他信息

### summary_note

核心结论：Step-Video-T2V 是本调研中「工具链点名最具体、可复现性描述最友好」的开源样本之一，其数据 pipeline 的每一步几乎都指明了所用的具体开源工具或模型；但它完全不涉及音频模态，对音视频联合生成议题无直接参考价值，且缺少任何数据策略的受控消融。

【最值得记录的四点】
(1) 六阶段流水线 + 全量打标后按阈值切档：切分（PySceneDetect AdaptiveDetector + FFmpeg，首尾各去 3 帧）→ 7 类质量标签（LAION 美学预测器 / CLIP ViT-L/14 NSFW / EfficientNet 水印 / PaddleOCR 字幕 / HSV 饱和度 / Laplacian 模糊 / FFmpeg 黑边）→ Farneback 光流三项运动分 → 自研 VLM 三路 caption → VideoCLIP + K-means 概念均衡 → 8 帧 CLIP Score 图文对齐。关键设计是「先全量打标、再逐级抬阈值切出 6 个预训练子集」，而非边过滤边丢弃，使课程调整无需重跑 pipeline。
(2) 12 万+ 簇的超细粒度概念均衡：用 Cluster_Cnt 与 Center_Sim 两个派生标签，同时实现概念重采样均衡与簇内离群剔除，聚类粒度比同期工作（如混元的 1 万概念中心）高一个数量级。
(3) 三路 caption 中保留「Original Title」：显式引入人类撰写的原始视频标题以获得风格多样性，这是与 HunyuanVideo「字段 dropout + 排列组合」不同的另一条「对齐真实 prompt 分布」的路径，思路简单但有效，值得借鉴。
(4) 「损失骤降」作为数据质量的信号：团队观察到训练损失会随数据质量的阶跃出现突降，并把这一现象用作课程切换的实操依据——这是少见的、把数据质量与训练动力学直接关联的经验披露。
此外，运动分从「过滤阈值」到「可控条件」的演化（在 Step-Video-TI2V 中把光流运动分注入模型实现运动幅度可控）是一个清晰的「数据标签复用为生成条件」的范例。

【主要缺陷】
· 无任何数据策略的受控消融，pipeline 各环节的收益无法量化归因；
· 漏斗定量不完整：只有 2B → 1B → 30M 这条主线，Figure 11 的分层过滤图未标注绝对数值，6 个子集规模未给出，原始视频池规模完全未知；
· 完全没有去重环节的描述（无哈希去重、无 embedding 语义去重）；
· 数据来源构成、版权与合规、隐私保护零披露；安全过滤仅有 NSFW 一项；
· 质检全部依赖传统 CV 算子与开源小分类器，无大模型语义判断，语义级判断由人工在 SFT 阶段兜底——可作为「大模型质检员范式之前」的对照基线；
· caption 为自然语言长短文本，未做强字段化 schema，限制了按字段做条件控制的能力；
· 数据处理基础设施与吞吐零披露。

若研究目标涉及音视频联合生成的数据构建，本条目仅可作为视觉侧切分/过滤/打标的工具级参考清单，音频侧需另行调研阶跃星辰的 Step-Audio 系列（与本模型无数据交集）。

## 不确定字段

以下字段的调研信息部分不确定（⚠️ 标注来源）：

- data_sources
- provenance_licensing
- domain_distribution
- audio_category_distribution
- language_accent_distribution
- funnel_retention_rate
- deduplication
- model_as_data_judge
- safety_filtering
- caption_model
- joint_av_caption_schema
- dialogue_transcription_attributes
- synthetic_data_synthesis
- av_sync_detection
- sync_metric_and_threshold
- temporal_vs_semantic_sync
- audio_quality_filtering
- audio_type_handling
- stage_data_mixture
- data_infra_throughput
- data_ablation
- quality_over_quantity_evidence
- benchmark_taxonomy_alignment
