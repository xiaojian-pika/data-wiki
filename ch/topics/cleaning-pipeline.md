# 横向对比：清洗流程

> 主题: 视频生成模型（含音视频同时生成）的数据处理：数据清洗流程、数据分布、打标方式

[← 返回首页](../index.md)

本页按字段横向对比所有条目。⚠️ 表示该条目此字段部分信息不确定。

**字段**: [清洗漏斗整体结构（几级过滤、各级顺序）](#清洗漏斗整体结构几级过滤各级顺序) · [漏斗定量保留率（每级过滤的输入/输出量与最终保留率，如Apollo 27%）](#漏斗定量保留率每级过滤的输入输出量与最终保留率如apollo-27) · [镜头切分方法（PySceneDetect/自研模型/shot-aware splitting）](#镜头切分方法pyscenedetect自研模型shot-aware-splitting) · [质量过滤（美学评分、清晰度、OCR文字过滤、黑边/水印/logo检测）](#质量过滤美学评分清晰度ocr文字过滤黑边水印logo检测) · [运动过滤（光流/运动分数阈值、静态与抖动剔除）](#运动过滤光流运动分数阈值静态与抖动剔除) · [去重方法（精确去重与基于embedding的语义去重分别记录）](#去重方法精确去重与基于embedding的语义去重分别记录) · [VLM/LLM作为质检员（多模态大模型质量打分与错配剔除，2026年从浅层打分器转向大模型语义判断的趋势）](#vlmllm作为质检员多模态大模型质量打分与错配剔除2026年从浅层打分器转向大模型语义判断的趋势) · [安全与合规过滤（NSFW、版权、人脸/隐私）](#安全与合规过滤nsfw版权人脸隐私)

## 清洗漏斗整体结构（几级过滤、各级顺序）

`pipeline_overview` · 详细程度: detailed

### [Allegro](../models/Allegro.md)

论文披露了一条 7 级串行过滤漏斗 + 细粒度标注 + 质量分层的完整链路，是同期开源工作中粒度最细的披露之一：
【第 1 级】时长/帧率/分辨率初筛：剔除时长 <2s、FPS <23、分辨率 <360p 的原始视频。
【第 2 级】场景切分：用 PySceneDetect 将长视频切为单场景 clip，仅保留 2–16 秒的片段；并额外丢弃每个 clip 的首尾各 10 帧，以消除场景检测的误判/转场残留。
【第 3 级】low-level 指标过滤：亮度（灰度图评估，剔除过暗/过曝）、清晰度（DOVER 视频质量评分）、语义/时序一致性（LPIPS 帧间感知距离）、运动幅度（UniMatch 光流）。
【第 4 级】美学过滤：LAION Aesthetics Predictor，对图像与视频中间帧打分。
【第 5 级】内容无关伪影过滤：CRAFT 文字检测 + 水印检测器，剔除黑边、文字、水印面积占比超阈值的样本。
【第 6 级】粗粒度打标：Tag2Text 对中间帧生成标签式 caption，提供初步语义信息（同时也是后续类目统计的依据）。
【第 7 级】CLIP 相似度过滤：计算视觉 embedding 与第 6 级 caption 的 CLIP 余弦相似度，剔除图文弱关联样本。
【标注】通过筛的样本再由微调版 Aria 生成细粒度时空 caption。
【分层 stratification】按阈值严格度将同一语料切为 4 个嵌套子集，分别喂给 T2I 预训练 / T2V 360p 预训练 / T2V 720p 预训练 / T2V 微调四个阶段。
关键设计点：把「便宜的粗筛（时长/分辨率/切分）」放最前、「昂贵的模型推理（美学、DOVER、CLIP、Aria caption）」放后，并且把 caption 生成放在 CLIP 过滤之前——用粗 caption 做图文一致性判据，避免对全量数据跑昂贵的细粒度 VLM。

### [Apollo](../models/Apollo.md)

四阶段串行漏斗（论文第 4 节 + Figure 3「Overview of our Dataset Annotation Pipeline」）：
【阶段 1 — 视频过滤与场景切分（Video Filtering and Scene Splitting）】先从四个维度建模视频质量：动态质量（主体运动比例、镜头稳定性）、静态质量（清晰度、美学、色彩饱和度）、内容自然度（无过度特效/水印）、安全性；剔除低分辨率、低 SNR/MOS、静音占比超 20% 的视频；随后做场景切分，保证每条样本仅含单一场景。
【阶段 2 — 音频过滤与后处理（Audio Filtering and Post Processing）】剔除低 SNR、低 MOS、异常削波（clipping）、失真、噪声的样本，要求静音占比 <20%、高保真、格式统一；随后做音视频一致性检测——Synchformer 做时序对齐、ImageBind 做语义对齐，确保时序与语义两个维度均高度同步。
【阶段 3 — 音频引导的数据切分（Audio-Guided Data Splitting）】按音频类型把数据集分层：先分人声/非人声得到 sound split，再从人声子集分出歌唱、单说话人语音、多说话人语音三支。
【阶段 4 — 密集标注与融合（Dense Annotation and Integration）】按子集分别调用专用模型产出语音转写、音频 caption、视频 caption（含 meta 信息与详细内容），语音与歌唱子集额外抽取说话人属性（性别、年龄），sound split 只给音频 caption；最后把所有标注融合为统一的密集 caption（unified dense captions）。
【漏斗排序特征】与 MOVA 一致，把最昂贵的多模态打标放在漏斗最末端，只对通过质量与同步过滤的样本做标注；差异在于 Apollo 在打标前多插了一层「按音频类型分流」，使打标策略可按子集定制，这是其 pipeline 结构上的独特点。
【披露粒度局限】四个阶段均只有定性描述，无阈值表、无各级样本量、无伪代码。

### [CineDance / CineDance-1M](../models/CineDance.md)

严格的三阶段策展流水线（three-stage curation pipeline），是全文方法论主干：
【阶段一：多样化采集与全面清洗（Data Preparation & Quality Assessment）】
  1) 粗粒度空间裁剪——EasyOCR 定位烧录字幕、FFmpeg 检测并裁除黑边/信箱框；
  2) MLLM 引导的时序截断——去除片头片尾等非正片内容，截断量 t = max(5min, 0.1L)；
  3) 全量质量指标计算——视频侧 DOVER（美学分 + 技术分）、AMT（运动平滑度）；音频侧 DNSMOS（信号保真）、CLAP embedding 时序方差；音视频对齐侧 ImageBind（全局跨模态对齐）、SyncNet（唇同步）；
  4) 关键设计：所有质量分「以元数据形式全量保留」而非硬性裁剪（not hard-pruning），使下游用户可按需自行构造任务特定子集；
  5) 镜头切分后再对 clip 级做一次 OCR + 黑边细验。
【阶段二：自底向上的叙事解析（Bottom-Up Narrative Parsing）】
  1) TransNetV2 切分原子镜头（2589 万个）；
  2) 以 Qwen3.5-27B 为骨干，按四条电影理论规则将原子镜头分组为叙事序列；
  3) 采用「自底向上镜头索引」而非直接让 LLM 输出时间戳，显著降低时间戳幻觉；
  4) 上下文感知滑窗推理——约 3 分钟窗口，且窗口边界对齐镜头边界；
  5) 20 秒软阈值抗碎片化。
【阶段三：可配置的双模态标注（Configurable Dual-Modal Annotation）】
  1) Anchor token 机制建立全局角色表与全局场景表；
  2) 视觉标注（Qwen3.5-35B-A3B）产出镜头五维属性 + 转场 + 局部角色表 + 活跃场景 + 镜头描述 + 转场描述；
  3) 音频标注（Qwen3-Omni-30B-A3B）拆成句级 ASR、镜头级音频 prompt、角色音色描述三个子任务以抑制幻觉；
  4) 窗口化 ASR-to-Character 绑定。
【最后】500 条随机片段的三人独立人工伪影审计做终验。

### [CogVideoX](../models/CogVideoX.md)

CogVideoX 的数据 pipeline 是「负面标签分类器 + 连续分数阈值 + 稠密重打标」三段式，结构清晰但层级较浅（相比 Movie Gen 的十余级漏斗要简单得多）：
【第一级：负面标签过滤】人工标注 20,000 条视频的正/负样本 → 训练 6 个基于 Video-LLaMA 的二分类过滤器 → 对全量原始视频做批量筛除。6 类负面标签为：Editing（人工重剪辑/特效破坏视觉完整性）、Lack of Motion Connectivity（转场处运动不连贯，人工拼接或静态图剪辑）、Low Quality（拍摄差、画面不清、相机抖动严重）、Lecture Type（讲座/直播口播，有效运动极少）、Text Dominated（文字主导）、Noisy Screenshots（手机/电脑屏幕直录）。设计动机被论文归纳为两类内在噪声（视频创作时的人工剪辑扭曲真实动态信息；拍摄设备与抖动导致的画质问题）加一类训练适用性问题（动态信息过少或动态缺乏连通性）。
【第二级：连续分数动态阈值】对全部训练视频计算光流分数（optical flow score，表征运动强度）与图像美学分数（aesthetic score，表征画质），并在训练过程中动态调整这两个阈值（dynamically adjust their threshold during training），以保证生成视频的动态性与美学质量。超参表给出美学分下限 4.5。这一「随训练阶段动态收紧阈值」的做法是 CogVideoX 数据工程较有特色的一点。
【第三级：稠密重打标（recaption）】对通过过滤的视频运行 Dense Video Caption Data Generation pipeline 产出长 caption（详见 caption_model / caption_structure）。
【第四级：高质量子集精调】预训练数据仍残留字幕、水印、低码率等脏数据，故在最后一个训练阶段挑出质量更高的 20% 子集做微调。
【图像分支】LAION-5B 与 COYO-700M 仅用美学分过滤后取 2B 图像，与视频混合训练（每张图像视为单帧视频）。
【推理侧对齐】用微调过的 LLM（图生视频用 GPT-4V / CogVLM）做 prompt upsampling，把用户短 prompt 改写成与训练 caption 同分布的长描述。

### [Cosmos-Predict2.5](../models/Cosmos-Predict25.md)

论文给出的是当前公开文献中最规范、最完整的视频 curation 漏斗描述之一，明确为七阶段串行流水线（Fig.1）：
【Stage 1 shot-aware video splitting 镜头感知切分】用高精度镜头边界检测模型将长视频切成 shot，确保原始镜头转场被排除；
【Stage 2 GPU-based transcoding GPU 转码】每个片段做 GPU 加速转码（利用 NVIDIA 硬件解码器/编码器）；
【Stage 3 video cropping 视频裁剪】裁除黑边（black borders）与空间 padding；此步后丢弃时长不足 5 秒的片段，剩余 5–60 秒片段共 60 亿+ 条；
【Stage 4 filtering 多级过滤】内部再分七个串行子级，顺序严格按「便宜的在前、昂贵的在后」排列：
  4.1 aesthetic scoring filter（美学打分过滤）
  4.2 motion filter（运动过滤，按运动程度量化并剔除）
  4.3 OCR filter（剔除文字叠加过多的 clip）
  4.4 perceptual quality filter（感知质量过滤，类 DOVER，剔除技术失真与感知伪影）
  4.5 semantic artifacts filter（语义伪影过滤，类 VTSS，剔除画中画 video-in-video、劣质转场等）
  4.6 VLM filter（Qwen2.5-VL 视觉语言模型高精度终审，论文明确说明「放在过滤最末端是因为它计算开销更大」）
  4.7 content type classifier（内容类型分类，26 类 taxonomy）+ 物理真实性裁剪（剔除游戏、合成图案、动画、卡通）
【Stage 5 captioning 打标】每条 clip 切为 5 秒窗口，用 Qwen2.5-VL-7B 逐窗口生成 short/medium/long 三档 caption；
【Stage 6 semantic deduplication 语义去重】embedding 聚类 + 簇内两两比对 + 保留最高分辨率版本 + 在线增量去重；
【Stage 7 sharding 分片】沿 content type / resolution / aspect ratio / length 四轴分片，支撑高效采样、课程式训练与细粒度域均衡。
【领域专用支线】五大 Physical AI 域（Robotics / Autonomous Driving / Smart Spaces / Human Dynamics / Physics）各走一条同构但两点不同的支线：省略 VLM filter、改用领域特定过滤器子集与调参；改用更大 VLM 与领域定制 prompt 做 caption。产出并入通用预训练集。
【设计哲学】整条漏斗的排序体现「算力递增的级联审查」——先用轻量打分器批量砍掉大头，再让最贵的多模态大模型对幸存者做高精度终审，最后才做去重与分片。这与 Cosmos-Predict1 相比是「a far stricter multi-stage filtering pipeline」。

### [Data-Juicer 2.0](../models/Data-Juicer.md)

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

### [Foley-Omni](../models/Foley-Omni.md)

清洗漏斗为三阶段串行结构，核心设计理念是「先过滤后标注、再用声学信号反向校验标注」，输入是弱标注音视频对（weakly labeled audiovisual data），输出是约 2.0M 条结构化三元组 (v_i, a_i, Ŝ_i)。
【第一级：多维质量过滤（Filtering）】置于最前，在昂贵的密集标注之前先砍掉低质样本，是典型的成本导向排序。论文原文表述为「removes clips with silence, low visual resolution, poor audio quality, weak audiovisual semantic consistency, or unreliable synchronization」——五个剔除条件对应三个维度、六项指标（Table 7）：
  · 视觉维度：分辨率 ≥480p、码率 ≥1 Mbps、motion score ∈ [0.1, 3.2]
  · 音频维度：Meta AudioBox Aesthetics 音频质量分 ≥0.6（同时覆盖静音与音质差两类问题）
  · 对齐维度：ImageBind (IB) score ≥0.3（语义一致性）、Synchformer sync score ≥0.2（时序同步可靠性）
  值得注意的是「语义一致性」与「时序同步」被拆成两个独立指标、两条独立阈值，而非合并为单一对齐分——这是本pipeline的一个精细设计。
【第二级：结构化标注（Annotation）】对留存片段调用 Gemini 2.5 Pro，同时输入视频帧与音轨，按 [WORDS]/[AUDIO]/[MUSIC] 三字段模板输出。模型被要求先做组件存在性检测、再做描述生成（detect-then-describe），prompt 模板见论文 Table 12。
【第三级：声学后验证（Acoustic Post-Verification）】用 Bandit 源分离模型把原音轨拆成 speech/effects/music 三个 stem，逐字段做能量门控 E(a_c) > −35 dB RMS；未通过的字段被置空。此级专门修正第二级的视觉幻觉偏差（visual bias），即 VLM 因看到视觉线索而虚构了实际不可闻的音频成分。
【整体评价】该pipeline的方法论亮点在于「双路验证」：视觉/多模态路径负责生成标注，纯声学路径负责否决标注，两条路径信息源不同因而错误模式不相关，交叉校验有效。这一思路对任何依赖 VLM 做音视频联合标注的工作都有借鉴价值。相对短板是缺乏去重、安全过滤、人工复核等常规环节的披露，也没有逐级保留率的定量报告。

### [Goku](../models/Goku.md)

★五阶段流水线，是本条目最核心的方法论贡献，顺序为：
【阶段1 图像与视频采集（Image and Video Collection）】
  从公开数据集（LAION、Panda-70M、InternVid、OpenVid-1M、Pexels）与内部库汇聚原始素材。
【阶段2 视频抽取与切分（Video Extraction and Clipping）】
  PySceneDetect 检测镜头边界 → 每秒采样 1 帧提取 DINOv2 特征 → 相邻帧余弦相似度校验镜头一致性（480p ≥0.85 / 720p ≥0.90）→ 单片段最长截断 10 秒 → 同源片段间用感知哈希（perceptual hashing）去重。
【阶段3 图像与视频过滤（Image and Video Filtering）】
  多维并行过滤：基础属性（时长 ≥4s、短边 ≥480、码率 ≥500kbps、帧率 ≥24/23.976 FPS）+ 美学评分 + OCR 文字覆盖率 + RAFT 光流运动分数，且每一维阈值按 480p/720p/1080p 三档分辨率分别设定，高分辨率档更严。
【阶段4 打标（Captioning）】
  InternVL2.0 生成关键帧 caption + Tarsier2 生成整段视频 caption（含相机运动）→ Qwen2 融合两路 caption → 追加 RAFT 运动分数作为可控条件。
【阶段5 数据分布均衡（Data Distribution Balancing）】
  内部视频分类模型对 4 帧关键帧打标 → 9 大类 / 86 子类 → 过度表征类降采样、不足表征类增强与过采样 → 整体加权人物内容。
【结构特点】前四阶段是自下而上的「逐条样本净化」（漏斗式串行过滤），第五阶段是自上而下的「数据集级分布整形」，二者关注粒度不同。将分布均衡独立成阶段，是 Goku 区别于同期工作的关键设计。

### [Hailuo / MiniMax Video](../models/Hailuo.md) ⚠️

[不确定]（完全无披露）。MiniMax 从未公开 Hailuo 视频数据的清洗流水线结构——没有漏斗级数、没有各级过滤顺序、没有流程图。全部公开材料中与数据处理最接近的表述，仅是 Hailuo 02 博客中「训练数据扩大4倍，且质量与多样性提升（with improved quality and diversity）」这一句，隐含存在质量筛选与多样性配平环节，但对具体做法只字未提。
这与本调研中 Vidu S1（六级漏斗图）、Seedance、Movie Gen、Cosmos 等给出完整流水线描述的对象形成鲜明对比。就数据处理方法论而言，Hailuo 属于「零披露」档位，可作为商用闭源模型披露下限的参照点，但无法提供任何可借鉴的流程细节。
需要说明的是：MiniMax 在语言模型侧（MiniMax-01/M1/M2/M3 技术报告）对数据清洗有较详细描述，但这些是文本数据流水线，与视频数据处理不可直接迁移，本调研不做跨模态外推。

### [HunyuanVideo-Foley](../models/HunyuanVideo-Foley.md) ⚠️

七级顺序漏斗，整体呈「有无 → 切分 → 音频质量 → 跨模态对齐 → 分类标注」的收窄结构。最鲜明的特征是：全部质量过滤集中在音频侧，视觉侧零质量门槛；且把最贵的跨模态对齐检测放在质量过滤之后、标注之前，属于典型的「先便宜后昂贵」的成本优化排序。
【第 1 级 音轨存在性检查】剔除无音频流的视频（eliminate videos lacking audio streams）。最前置的一票否决，成本最低、淘汰量可能最大（大量网络视频为无声或纯配乐）。
【第 2 级 场景检测 + 定长切分】用场景检测算法对原始视频做镜头切分，再规整为 8 秒定长块。这一步不做淘汰，只做形态转换（把变长长视频转为定长短片段），是后续所有逐片段判定的前提。
【第 3 级 静音占比过滤】计算每个 8 秒片段的静音占比，超过 80% 的丢弃。宽松阈值，只剔近乎无声者。
【第 4 级 带宽检测】检测音频的有效采样率（effective sampling rate，即实际频谱能量的上限而非文件声明的采样率），仅保留有效采样率超过 32 kHz 的样本。这是一条相当严格且技术上讲究的门槛——它能识别出「文件标称 48 kHz 但实际是低码率音源上采样而来」的伪高清音频，与 UniVerse-1 用「码率-分辨率比」识别伪高清视频的思路同源。32 kHz 有效带宽对应 16 kHz 的可听频率上限，保证了镲片、玻璃碎裂、金属摩擦等高频丰富的 Foley 事件音有足够的频谱信息。这一门槛与模型 48 kHz 的输出规格直接匹配，是「数据带宽决定输出带宽」的必然要求。
【第 5 级 音频美学与信噪比评估】主指标用 AudioBox-Aesthetics 工具包（Meta 出品，输出 PQ 制作质量 / PC 制作复杂度 / CE 内容愉悦度 / CU 内容有用性四个维度分数），辅以 SNR 信噪比作为补充指标，筛出高质量音频片段。论文未给出任何具体阈值数值——这是本级最大的信息缺口，因为 AudioBox 四个分数用了哪几个、各自卡在多少完全未知。[不确定]
【第 6 级 音视频对齐验证】双路并行：ImageBind 做语义对齐检测（画面内容与声音内容是否语义匹配），AV-align 做时序对齐检测（声音事件与画面事件在时间轴上是否对应）。这是整条流水线中最具方法论价值的一环，也是本工作区别于多数 V2A 工作的地方——把语义与时序作为两个正交维度分别检测。同样未给出阈值。[不确定]
【第 7 级 分类标注与分布管理】用语音-音乐检测模型 + 音频分类模型为每个存活片段打类别标签，据此管理类别分布、实现均衡表征。这是唯一一个不做淘汰而做「配比调节」的环节，也是最终决定数据构成的闸门。
【第 8 步（并行，非过滤）音频 caption 生成】用 GenAU 模型为每个片段生成简洁的音频内容描述，作为训练时的文本条件。
【整体评价】流程设计紧扣任务特性、取舍清晰（音频严、视觉松），第 4 级带宽检测与第 6 级双维度对齐是两处亮点。但阈值披露极不完整——七级中只有三级给出了数值（8 秒、80%、32 kHz），其余四级（美学、SNR、ImageBind、AV-align）全部无数值，与 UniVerse-1、MOVA 的逐项阈值披露相比透明度明显偏低。

### [HunyuanVideo](../models/HunyuanVideo.md)

【HunyuanVideo 原版（2024）——分层漏斗（hierarchical data filtering pipeline）】这是本条目的核心参考价值所在。结构为「一条过滤器链 + 四道逐级加严的阈值门 + 一道人工精选」：
第0步 预处理：PySceneDetect 将原始视频切为单镜头片段；OpenCV Laplacian 算子选取清晰帧作为片段起始帧；
第1步 去重与均衡：内部 VideoCLIP 抽 embedding，余弦距离语义去重，k-means ~10K 概念中心做概念重采样均衡；
第2步 过滤器链（并行多路打分）：
  · Dover 模型从「美学（aesthetic）」与「技术质量（technical）」两个视角打分；
  · 自研 blur 检测模型剔除不清晰内容；
  · 光流估计（optical flow）估算运动速度，剔除静止与慢动作片段；
  · PySceneDetect + Transnet v2 提供场景边界信息；
  · OCR 模型剔除文字过多的片段，并裁掉字幕区域；
  · YOLOX-like 视觉模型检测并剔除/裁除水印、边框、logo 与敏感信息；
第3步 分层构档：用同一套过滤器、不同严格度阈值构造 256p / 360p / 540p / 720p 四档训练集，逐档加严；
第4步 SFT 档：在最严档之上叠加人工标注精选，得到约 100 万样本的 SFT 数据集。
图像数据走一条平行的分层pipeline，复用除运动相关外的大部分过滤器，产出两个图像集（数十亿级 + 数亿级）。

【HunyuanVideo 1.5（2025）——三级过滤 + 空间裁剪】结构被重组为更工程化的三级：
第0步 基础去重与损坏文件剔除；
第1步 切分：PySceneDetect + 自研算子切为 2–10 秒片段 → 转场分类器剔除含转场特效的片段；
第2步 空间裁剪：对字幕、logo、水印所在区域做裁剪而非整片丢弃，但若裁剪后保留面积 < 原帧 60% 则整片丢弃（保证构图完整性）——这是相对原版的重要改进：从「见水印即弃」变为「能裁则裁、裁不动才弃」，提高了数据利用率；
第3步 三级质量过滤：
  (a) 基础过滤：剔除带 padding 黑边、拼接痕迹（stitching artifacts）、宫格拼贴（grid layouts/collages）、静止或低运动的视频；
  (b) 视觉质量过滤：一个综合质量评估算子，从四个维度打分——清晰度（sharpness）、细节保留（detail retention）、噪声与伪影（noise & artifacts）、动态范围（dynamic range）；
  (c) 美学过滤：美学打分算子剔除低分片段。
输出约 8 亿高质量片段进入预训练，后续阶段再按更高标准逐级收缩到 2亿/1亿/1亿/100万。

### [InstructAV2AV](../models/InstructAV2AV.md)

整条流程可划分为三大阶段、约十个环节，整体架构是「先严筛真实素材 → 再受控合成配对目标 → 最后双重验证」。与调研中其他工作「清洗真实数据」的单向漏斗不同，本pipeline的中段是一个生成环节，因此形态是「漏斗 + 生成器 + 二次漏斗」的沙漏结构。

【阶段一：真实素材多级过滤（multi-stage audio-visual filtering）】目的是筛出「适合作为编辑基底」的干净单镜头片段。
  1. PySceneDetect —— 长视频切为单镜头片段，为后续所有 mask/跟踪操作提供前提。
  2. CoTracker3 网格点跟踪 —— 计算平均运动幅度，低于阈值的片段丢弃（剔除静态画面）。
  3. LAION Aesthetics Predictor —— 剔除视觉低质片段。
  4. PyDub 静音检测 —— 剔除低于 -45 dBFS 的静音片段。
  5. Audiobox-Aesthetics —— 剔除低质音频。
  6. 分流处理：语音路用 TalkNet 定位主动说话人 + Scribe 提取语音时间戳，仅保留「语音与画内可见说话人时间对齐」的片段；非语音路剔除声源模糊片段，为每个片段打唯一语义声音事件标签。
  注意顺序设计：视觉过滤在前、音频过滤在后、声源归属判定最后，成本从低到高递增，是合理的成本导向排序。

【阶段二：数据编辑引擎（data editing engine）—— 本工作的核心贡献】目的是从单份真实素材制造出 (source, instruction, target) 三元组。
  7. Grounded-SAM-2 —— 获取实例级 mask，定位可编辑的视觉实体。
  8. Qwen3-Omni —— 以实例 mask 与源音视频上下文为条件，生成多样化的文本编辑指令，覆盖四类编辑操作。
  9. 音频侧合成：SAM-Audio 把目标实体的声音从原音轨分离出来 → ElevenLabs 文本转音频/语音模型按指令合成新的目标声 → 与保留的背景音无缝混合，得到 target 音轨。
  10. 视觉侧合成：自建的 mask-guided 编辑模型（基于 Wan2.2-5B，以 source-conditioned flow matching 目标训练）在 mask 区域内合成新目标；关键是引入 frame-wise cross-attention 把已合成的目标音频特征注入视频生成过程，强制视觉目标与新音轨严格时序同步（如新台词与新口型逐帧对齐）。
  这里的顺序是「先合成音频、再以音频为条件合成视频」，而非并行或反向——因为语音的时长与节奏一旦确定，口型必须服从它，反过来则难以约束。

【阶段三：双重验证（verification）】目的是剔除合成失败的样本。
  11. 自动验证：多模态 LLM（Qwen3-Omni）沿五个维度打分——(i) 指令保真度、(ii) 内容保持度、(iii) 感知质量、(iv) 音视频同步、(v) 安全性，仅保留同时通过全部五项的样本。产出 79K 训练对。
  12. 人工验证（仅用于评测集）：20 名志愿者分为 10 个评审对（judge pairs），每个候选样本由 5 个独立评审对评估，每对负责一个评价维度；样本须通过全部五项才能进入评测集。产出 1K 人工精选评测对。

【方法论评价】
  亮点一：把「不存在的配对监督数据」用受控生成制造出来，且通过 SAM-Audio 分离-混合、mask 区域限定这两个机制，从构造上就保证了「非目标区域严格未变」，从而使 source 与 target 构成干净的差分对——这是真实数据永远给不了的精确性。
  亮点二：合成阶段就用 frame-wise cross-attention 强制音视频同步，属于「在数据生成时保证质量」而非「生成后过滤」，比事后用同步分数筛选效率高得多。
  亮点三：自动 + 人工的两级验证分工明确，自动验证保规模、人工验证保评测集可信度，且人工验证采用「一对评委负责一个维度」的分维度独立评审设计，避免单人对多维度打分的相互干扰。
  短板：全流程无一处披露定量保留率；除 -45 dBFS 外几乎所有阈值数值缺失；无去重环节；无任何数据侧消融来证明pipeline各环节的必要性。

### [2026 其他音视频联合生成](../models/JAVG_2026_misc.md) ⚠️

七项的数据管线复杂度差异极大，可排成一条从「无管线」到「六阶段工业管线」的谱系：

【ALIVE —— 六阶段工业级管线（本批最完整）】
前置：先从 raw data pool 中筛出「videos with audio」（带音轨的视频），无音轨样本直接出局。
原文：「our pipeline begins by filtering videos with audio from the raw data pool and then proceeds through six core stages: video quality pre-processing, captioning, audio quality filtering, SubjectID correction, clarity filtering, and data balancing.」
第1阶段 Video quality pre-processing（视频质量预处理）：OCR 测量叠加文字占比并检测水印 → 预训练美学模型逐帧打分 → RAFT 计算光流做运动分析 → 质量模型把样本判为 high-quality 或 13 个 low-quality 维度之一。
第2阶段 Captioning（打标）：两轮 SFT 训练的 caption 模型 + 人工标注 sub-motion units 时间戳 → 产出 Subjects / Visual / Narration 三段式结构化 caption，语音用 <W>、非语音声学事件用 <I> 内联标记。
第3阶段 Audio quality filtering（音频质量过滤）：MLLM 双准则打分——音频质量分（剔除显著背景噪声）+ 音画相关性（分离强相关样本，管控 BGM 类弱相关样本占比）。
第4阶段 SubjectID correction（说话人-主体归属校正，五步子管线，本批最独特的设计）：① TS-TalkNet 做主动说话人检测，生成人脸-声音对应矩阵；② Qwen3-omni 解析旁白时间戳；③ majority voting 做「句子→TrackID」映射；④ CLIP / ArcFace 嵌入做视觉-语义匹配；⑤ 用校正后的标签重写文本。未通过高置信阈值的视频被过滤，得到一个「simplified 但高质量」的初始数据集。这一阶段解决的是多人场景中「哪句话是谁说的」的错误归属问题——这是纯视频模型完全不存在、而 AV 模型必须面对的独有难题。
第5阶段 Clarity filtering（清晰度过滤）：用六档清晰度参考图（从模糊到锐利）做评判。
第6阶段 Data balancing（数据均衡）：speaking / non-speaking 顶层二分 → 九大 Level-1 领域三级标签 → 按概念频次与预期应用场景调整各类占比。
另有并行的 Character-driven data pipeline（cross-pair + in-pair augmentation，见 synthetic_data_synthesis）。

【NAVA —— 大规模多算子管线（漏斗两端公开）】
原始池 ≈ 20M audio clips + 100M video clips → 最终约 15M clips。
① Hadoop 管线大规模切分（「raw videos are first segmented at scale with a Hadoop-based pipeline」）
② PaddleOCR 做 OCR 过滤与字幕擦除（subtitle removal，注意是「擦除」而非仅「剔除」）
③ VideoCLIP 提取视频嵌入 + 大规模 k-means 聚类去除冗余/近重复片段
④ 多算子质量过滤：视觉侧（aesthetics 美学 / sharpness 锐度 / brightness 亮度 / motion score 运动分）、音频侧（AudioBox Aesthetic 分数）、音视对齐侧（SyncNet / SyncFormer / ImageBind 三个对齐算子）
⑤ VLM-based filtering and tagging，保留视觉质量清晰的片段
⑥ YAMNet 音频分类 + omni-modal tagger 做五类音频打标
⑦ 打标：Qwen3-VL 生成视频 caption、Qwen3-Omni 生成音频 caption，再由 Gemini-3-Flash 直接拼接或改写融合；高质量子集改用 Gemini-3-Pro
⑧ 多算子协同过滤（multi-operator collaborative filtering）产出 160K 高质量 SFT 子集

【OmniCustom —— 三条规则的轻量管线】从 SpeakerVid-5M（5.2M clips / 8,000 hours）出发：① SyncNet 双阈值（|offset|≤3 且 confidence>1.5）；② 美学分 <0.3 剔除；③ 时长 <10 秒剔除 → 得 OmniCustom-1M（约 1M clips / 2,500 hours）。之后统一格式（480p/24fps、音频 16kHz）→ GLM-ASR 转写 → 参考-训练配对切分（前 4 秒作参考音频、后 5 秒作训练音视频）→ 随机采样含人脸的帧并裁剪作参考图。

【StreamChar —— 数据集拼合，无自建管线】直接组合 SpeakerVid-5M + TalkVid + OpenHumanVid，加约束「无超过 20 秒的视频/台词」；ASR 时间戳用于 progress-aware pointer 的监督标签。无显式质量/同步过滤描述[不确定]。

【CCL / Baton —— 仅有来源清单，无管线披露】CCL 为 OpenHumanVid + in-house（访谈/短剧/电影）+ WavCaps/VGGSound；Baton 为 OpenHuman-Vid + AudioCaps + WavCaps + 互联网视频。二者均未描述任何过滤、打标或去重步骤[不确定]。

【ITS-JAVG —— 把管线搬到推理时】无训练数据管线，但其结构与数据过滤管线同构：每 prompt 生成 N 个候选（JavisDiT 5 个、MMDisCo 10 个）→ 用一组验证器（VideoReward-TA / JavisScore / ImageBind-TA / AVHScore / VQAScore / ImageBind）打分 → ARW 自适应加权聚合 → EvoSearch 进化式选优。可以理解为「把训练数据管线中的多打分器联合过滤，前移到了生成之后、输出之前」。

### [音视频联合生成基线合集](../models/JavisDiT_baselines.md) ⚠️

五者的清洗漏斗复杂度从「几乎没有」到「三级过滤 + 大模型标注」不等：

【MM-Diffusion（最简）】爬取 → 切分 → 无显式过滤。从 YouTube 爬取 928 条自然场景视频，机械切分为 1,000 条 10 秒不重叠片段，按 9 个场景类目组织。论文未描述任何质量过滤、去重、同步检测环节。AIST++ 直接取用现成数据集（作者另做了 crop 处理，仓库中命名为 AIST++_crop）。这反映 2022 年联合生成研究「先把任务做通、数据求精不求多」的阶段特征。

【AV-DiT（无自建 pipeline）】直接复用两个公开数据集，只做格式化预处理：视频采样 16 帧并裁剪到 256×256、音频截断或补齐到 1.6 秒 @16kHz 并转 mel 频谱（40×16×8）。无过滤漏斗。

【JavisDiT / JavisDiT++（本合集中最完整、最透明的漏斗）】分训练侧与评测侧两条独立管线：
《训练侧 —— 三阶段各自的数据准备》（GitHub data.md 完整公开）
- 阶段一（音频预训练，78 万条）：音频文件夹 → 生成元数据 CSV → 提取音频统计信息 → 截断到 30 秒以内 → 统一重采样到 16kHz → 添加占位 dummy 视频引用 → 输出 train_audio.csv。明确声明「不采用任何数据过滤策略」以最大化 T2A 能力。
- 阶段二（音视频 SFT）：TAVGBench 110 万条三元组 → 用 FunASR 检测并剔除大部分含人类语音的视频 → 按 Open-Sora 的方法做美学评分（aesthetic scoring）、光流/运动评分（flow scoring）、OCR 文字评分（OCR scoring）三项过滤 → 剔除损坏视频、过滤掉少于 10 帧的样本 → fps 统一归一到 16Hz → 与 TAVGBench 的 caption 对齐 → 合并多来源 CSV → 输出 train_av_sft.csv（33 万条）。
- 阶段三（AV-DPO）：从与 SFT 不重叠的 3 万条 prompt 池中，用参考模型对每条 prompt 生成 N=3 个音视频候选 → 提取生成样本的元数据与音频 → 与 1 条真值配成「1 真值 + 3 生成」的 4 候选组 → 用多个奖励模型做模态感知打分 → 归一化模态感知排序，选出 winning/losing 对 → 输出 train_av_dpo.csv（约 2.5 万条偏好对）。
《评测侧 —— JavisBench 构建》：从已有数据集测试集（Landscape / AIST++ / FAVDBench）与 2024 年 6–12 月上传的 YouTube 视频中采集约 3 万条候选发声视频 → 前置过滤（pre-filtering，基于质量剔除噪声候选）→ 用 Qwen 系列模型自动生成 caption 并按 19 类体系归类 → 后置过滤（post-filtering，基于内容保证多样性）→ 严格人工法律与伦理审核 → 最终 10,140 条；另随机抽 1,000 条构成 JavisBench-mini。

【Harmony】漏斗结构较简，但引入了模型化质检：语音侧从 Emilia + OpenHumanVid + SpeakerVid 汇总 → 用「音视频一致性打分模型（audio-visual consistency scoring model）」筛选 → 得到 200 万条高质量片段（3–10 秒）；环境音侧汇总 AudioCaps + Clotho + WavCaps + 自采 200 万条 → 全部数据用 Google Gemini 做自动标注（ASR 转写 + 视频描述 caption + 背景音描述）→ 按 1:1 混合进入训练。

【UniAVGen】未描述任何数据清洗流程[不确定]，只说明阶段一用 Emilia 英文子集、阶段二三用内部真人音视频数据，以及音频 24kHz 采样转 mel 频谱、视频 16fps 后经 VAE 编码这两条格式化处理。是本合集中数据披露最少的一家。

### [可灵 3.0 / 可灵视频 3.0 Omni](../models/Kling_30_Omni.md) ⚠️

Kling-Omni 技术报告披露为“三级数据处理体系”（three-tier）：第一级基础过滤（basic filtering）——分辨率/时长阈值校验、逐帧与时序指纹去重、音视频损坏检测、内容安全与NSFW过滤；第二级时序质量评估（temporal quality assessment）——模糊/抖动/压缩噪声打分，剔除突兀场景切换与不连贯转场，剔除“动作语义密度过低”的静态片段；第三级多模态对齐校验（multimodal alignment）——视频caption与实际画面的语义一致性、参考图与目标视频的保真一致性、以人为中心任务的角色身份一致性检查。上游还有一条自动化互联网数据挖掘 pipeline（自研embedding模型做跨模态语义检索）与一条专家模型驱动的合成数据构造 pipeline。同团队 Koala-36M 公开的完整漏斗为：原始视频 → 时序切分（Color-Struct SVM）→ 结构化打标 → 人工阈值过滤 → VTSS 模型过滤，可作为可灵体系数据漏斗的公开范式参考。[不确定：可灵3.0 Omni 中音频专属过滤级在三级体系中的插入位置]

### [LTX-2](../models/LTX-2.md)

LTX-2 本身未画新的清洗漏斗图，其视频侧完全沿用 LTX-Video 的 pipeline，在其输出上再叠加一层「音频信息量筛选」。
【LTX-Video 数据处理 pipeline（论文 Fig.11，共9级，顺序如下）】
1. Input Shots（以镜头为输入单元）
2. Crop Black Bars（裁除黑边、标准化宽高比）
3. Estimate Motion Level（估计运动量级）
4. Generate Thumbnails（生成缩略图）
5. Mid-Frame CLIP Image Embedding（取中间帧计算 CLIP 图像嵌入）
6. Cluster and De-Duplicate（聚类与去重）
7. Resize Shots（尺寸归一）
8. Estimate Aesthetics（美学评分）
9. Filter Shots → Final Shots（按运动量/美学分等阈值过滤，产出最终镜头集）
随后是独立的「Captioning and Metadata Enhancement」阶段——用内部自动 captioner 对全量训练集重新打标（re-caption）。
【LTX-2 增量】在 Final Shots 之上以「含显著且信息量丰富的音频成分」为准则抽取 AV 子集，并用新开发的音视频双轨 captioner 重新打标。
【结构特点】属于「先几何/结构归一 → 再表征提取 → 再去重 → 再质量评分过滤 → 最后打标」的典型五段式，浅层判别器（CLIP embedding + 美学 Siamese 网络 + 运动估计）为主，未见 VLM/LLM 语义质检环节。

### [LongCat-Video](../models/LongCat-Video.md)

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

### [MOVA](../models/MOVA.md)

三阶段漏斗（Figure 3），基于 Ray 分布式框架实现，NVIDIA GPU 与昇腾 Ascend NPU 混合算力：
【Stage 0 — 入口筛除】剔除解码失败或缺少有效音频通道的样本；对容器格式异常的做 remux、对编码格式异常的做转码。
【Stage 1 — 视频预处理与标准化（Standardization → Detection → Segmentation）】
  1) 核心内容归一化：FFmpeg cropdetect 检测黑边并保留核心画面 → 居中 → 缩放到 720p → 对称补边成 9:16 或 16:9；帧率重采样到 24fps。
  2) 语音活动检测：Silero VAD 标注 speech / non-speech 区间。
  3) 场景转换分析：PySceneDetect 记录全片场景切点时间戳。
  4) 分段：融合 VAD 与场景切点的时间信息，生成 8.05 秒定长的四类片段（单/多场景 × 语音/非语音），语音片段起点自适应避免截断语句。仅保留语音片段（占预处理片段的 69.47%）。
【Stage 2 — 音视频质量评估（三维并行过滤）】音频质量（信号级 + 美学）、视频质量（技术 + 美学）、音视频对齐（时序 + 语义），阈值由人工抽检不同 cutoff 下的留存视频后经验设定；另用 EAT 分类模型构建 speech/non-speech 专用子集。
【Stage 3 — 音视频联合打标】MiMo-VL-7B-RL 做视觉描述、Qwen3-Omni-Instruct 做语音转写、Qwen3-Omni-Captioner 做非语音音效/音乐描述，最后由 GPT-OSS-120B 做跨模态一致性校验并融合为统一自然语言 caption。
【训练阶段内的二次过滤】Phase 2 之前还额外叠加了三道正交过滤（OCR 无烧录字幕、LSE 唇音对应、DOVER 技术分），见 quality_filtering 与 stage_data_mixture。
整体特征：先“标准化+切分”再“打分过滤”最后“打标”，把最贵的 MLLM 打标放在漏斗最末端，只对已通过质量与对齐筛选的片段做标注，是成本控制上的合理排序。

### [合并条目：① Mochi 1](../models/Mochi_MAGI_Motif.md) ⚠️

① Mochi 1：无任何 pipeline 披露。官方博客只讲架构（AsymmDiT、AsymmVAE、44,520 token 全 3D 注意力、扩展的 3D RoPE、SwiGLU、QK-Norm）与评测协议（沿用 OpenAI DALL-E 3 的 prompt adherence 自动评测范式、以 Gemini-1.5-Pro-002 为评判模型），数据章节完全缺席。模型卡唯一与数据处理相关的表述是「已施加 NSFW 内容过滤」。[不确定]
② MAGI-1（方法级完整、参数级留白的四段式漏斗，Fig.13）：
【第 1 级】镜头切分：PySceneDetect 把长视频切为单镜头短片（仅对视频数据施加，图像跳过该模块）。
【第 2 级】Filter Actors 并行过滤：11 类专用过滤器一次性作用于切好的 clip（详见 quality_filtering / motion_filtering）——Video Quality Assessment（DOVER）、Aesthetics（LAION 美学模型）、Overexposed/Underexposed（HSI 色彩空间平均亮度）、Motion Strength（RAFT 光流 + 显著性检测分前景/背景）、Camera Movement Stability（相邻帧光流一致性）、Slides Movement（光流散度）、Border Detection（边缘检测 + Hough 变换）、Text Detection（含字幕的时空模式专项识别）、Logo Detection（Florence-2 开放词表检测）、Corner Face Detection（人脸检测 + 角落位置置信度）、Transition Detection（CLIP 关键帧语义相似度）。
【第 3 级】去重：CLIP 与 DINOv2 双特征成对相似度，任一超阈值即判重删除。
【第 4 级】MLLM 高级过滤：前三级已滤掉绝大部分劣质数据、剩余量已大幅缩小，此时引入多模态大模型做一轮语义级复筛，捕捉复杂 bad case；关键工程设计是「该步骤可无缝并入后续 caption 流程，从而降低总成本、提升效率」——即判分与打标共用同一次 MLLM 调用。
【第 5 级】打标：通过筛的数据由同一 MLLM 生成 highly descriptive caption 与 auto-regressive caption。
【第 6 级】分布调整：多阶段调整（质量随阶段收紧）+ 动态分布调整（按评测反馈在线改配比）。
设计要点：把便宜的规则/小模型过滤放前、把昂贵的 MLLM 放在数据量已显著收缩之后，并让 MLLM 的「判官」与「打标员」两个角色复用同一次前向。
③ Motif-Video 2B（工具级 + 参数级完整的分支式漏斗，Fig.7 Sankey）：
【第 0 级】分支拆解：原始池显式拆为 Image Real / Image Synthetic / Video Real / Video Synthetic 四条支路，共用同一套下游控制。
【第 1 级】Sanitation（前置消毒）：剔除损坏/不可解码文件、异常小文件（缩略图或下载损坏）、SSCD 近重复、NSFW、带水印内容。其中 NSFW 与水印用双信号——先是继承自旧爬取管线的 OCR 初筛（用帧上文字检测标出台标、烧录字幕等高置信水印），存活片段再由 VLM 复核结构化标签（watermark / nsfw / padded / multi_scene / timelapse / quality），命中即丢；论文称第二遍是「架在 OCR 之上的语义感知安全网」。
【第 2 级】黑边检测：ffmpeg cropdetect 基于亮度统计估计最大内容矩形。
【第 3 级】OCR 检测：PaddleOCR-VL（经 vLLM 服务化）在每个 clip 均匀采样 N 帧上运行，按空间 IoU 跨帧聚类检测框，只保留出现在 ≥50% 帧中的簇（以此区分固定叠加物与场景内瞬时文字）；存活的 OCR 区域与黑边裁剪框合成为单一最终矩形（排除内容区上 20% 的台标与下 20% 的字幕），在一次 ffmpeg 重编码中连同分辨率缩放与帧率限制一并落地。
【第 4 级】场景切分与长度控制：保守阈值过切分 + SigLIP 相似度 stitch 合并 + 丢弃 <2s。
【第 5 级】视觉质量多路过滤：Aesthetic Predictor V2.5（美学）、亮度（OpenHumanVid 公式）、Koala-36M 式模型化 suitability 分（保守拒绝）、DOVER technical 分（技术质量）、UniMatch 光流（运动质量，双侧截尾）。论文强调这些信号「不被合成为单一学习排序，而是每个过滤器专治一种失效模式」。
【第 6 级】SSCD 三阶段去重（嵌入 / 分组 / 代表选择，详见 deduplication）。
【第 7 级】VLM 打标即元数据：Qwen3-VL-30B-A3B 单次前向同时产出 caption 与结构化标签，标签直接回灌 sanitation 与分阶段过滤。
【第 8 级】分阶段渐进准入：144p → 360p → 480p → 720p，每次跃迁重新施加分辨率/时长/运动/美学过滤且阈值更严；720p SFT 额外叠加 domain-balancing 与 dynamic-motion 准入；合成视频只在 720p 注入。
设计要点与 MAGI-1 同构（VLM 打标与过滤共用一次前向），但 Motif 更进一步——「因为这些标签与训练 caption 出自同一次前向，过滤与条件化在构造上始终保持同步（synchronized by construction）」，即从机制上消除了「过滤用的语义判断」与「训练用的文本条件」之间的漂移。

### [Movie Gen](../models/Movie_Gen.md)

【视频侧】三级过滤 + 一级打标的漏斗（论文图9）：
1) 视觉过滤（Visual filtering，6个filter）：最小宽高门槛 → 宽高比配比 → 视频OCR去多字幕/文字 → FFmpeg场景边界检测切4~16秒clip → 训练轻量视觉模型预测帧级美学/画质/大边框/视觉特效并过滤 → 参照 Panda-70M 移除与视频开头重合的clip的前几秒（开头常有不稳定运镜和转场特效）。
2) 运动过滤（Motion filtering）：内部静态视频检测模型去无运动 → FFmpeg VMAF motion score 与 motion vector 判定「合理运动」→ PySceneDetect 每秒镜头数识别并剔除抖动 → 剔除幻灯片等特效运动。
3) 内容过滤（Content filtering）：copy-detection embedding 感知去重 → 视频-文本联合embedding聚类 + 合并重复簇 + 1/sqrt(簇大小) 重采样做概念均衡。
4) 打标（Captioning）：LLaMa3-Video 8B/70B 生成平均100词的稠密caption + 16类相机运动分类器结果前缀拼接 + FPS token。
另有「多阶段curation」：用逐级更严的视觉/运动/内容阈值切出3个子集——720px低分辨率训练集、768px高分辨率训练集、以及新增数据扩充后的高分辨率集。
【视频SFT侧】四阶段串行：①自动严阈值筛选（美学、运动、场景切换严阈值 + 用 Detic 物体检测剔除主体过小的视频）得到几百万条但概念不均衡的候选；②概念均衡（600动词taxonomy做 text k-NN + 人工挑种子后做 video k-NN）缩到可人工审核的规模；③人工挑选影视感视频（需有角度光/自然光或棚拍光、色彩鲜艳但不过饱和、画面不杂乱、有非平凡运动、无相机抖动、无编辑特效与叠加文字），并由标注员亲手裁剪出最精彩的片段；④人工在 LLaMa3-Video caption 基础上精修与补全。
【音频侧】：AED 按 AudioSet 527类打标 → 丢弃 silence 主导样本 → 映射为 voice/music/sound 三类 → 用 CAVTP 余弦相似度分入 diegetic/non-diegetic/mixed 桶 → 视觉侧质量过滤（OCR去带文字视频、去静态视频、去<480px低清视频）→ 时长限制4~120秒 → copy-detection embedding 视觉去重 → 多模型合成结构化caption。微调数据额外用「影视感音视频分类器 + AED」自动初筛，再由人工标注做最终选择。

### [NVIDIA NeMo Curator](../models/NeMo_Curator.md)

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

### [OmniHuman 数据集 + OHBench](../models/OmniHuman.md)

整体为「四级过滤 → 结构化感知 → 层次化标注 → 校验」的全自动流水线（fully automated pipeline），是本工作的核心贡献之一。各级顺序遵循「由轻到重」的成本原则（论文原文：lightweight-to-heavy filtering strategy）：
【第一级 时空清洁（Temporal-Spatial Cleaning）】
- 用 TransNetV2 做镜头切分，把长视频切为单镜头片段；
- 用 OCR 与 logo 检测算法估计「文字污染区域」（text-contaminated regions），并对帧做裁剪以去除字幕与台标；
- 标准化到 30 FPS 与 44.1 kHz。
【第二级 音频治理（Audio Governance）】
- 剔除音轨缺失、时长异常、质量低下（静音占比、音量阈值）的样本；
- 用 Demucs 做四源分离，人声为目标轨、其余混合为背景轨。
【第三级 美学过滤（Aesthetic Filtering）】
- 帧级 CLIP 美学打分快速剔除低质片段（快、粗）；
- 视频级 DOVER 评估构图与清晰度（慢、细）；
- OCR 文字密度分析与水印检测。
【第四级 运动与时序一致性（Motion & Temporal Consistency）】
- UniMatch 光流分析运动质量，剔除静止、抖动、异常运动的片段；
- 感知哈希（perceptual hashing）+ SigLIP 余弦相似度过滤语义不一致的帧。
【第五级 结构化感知（非过滤，构建标注基础）】
- YOLOv11 检测人体实例 → NMS 精修 → MOTRv2 通过 query propagation 建模跨帧关联，得到身份轨迹（tracklet）；
- DWPose-L 提取身体、脸部、足部共 134 个全骨架关键点，并配以专门的手部检测与优化；
- 3D-Speaker 做说话人分离得到 M 个语音区间；SyncNet 解析音视觉归属，贪心匹配把每段音频指派给响应最高的视觉 ID 轨迹；ArcFace 提取人脸嵌入完成身份指派。
【第六级 层次化标注】Qwen3-Omni 为推理核心的两阶段 caption 生成 + Gemini-3 做背景合理性与交互语义判定（详见 caption_model / caption_structure）。
【第七级 一致性校验】多重校验：结构化 subject 标签数必须与跟踪模块输出的数量一致；说话人数量与转写内容必须与上游 ASR 转写保持在可接受的编辑距离范围内；只有全部通过才保留该视频。
【结构评价】
- 优点：级序设计合理（廉价的镜头切分与音轨存在性检查在前，昂贵的 DOVER、光流、跟踪、MLLM 标注在后）；「感知 → 标注 → 反向校验」的闭环较少见——用跟踪模块的输出去校验 MLLM 标注的数量一致性、用 ASR 转写去校验 caption 中的语音内容，这是一种低成本的自动化幻觉抑制（详见 model_as_data_judge）；
- 缺点：几乎全流程无阈值数值披露（CLIP 美学分、DOVER 分、静音占比、音量、光流、SyncNet 分数、编辑距离边界全部为「preset threshold」「acceptable margin」之类的定性表述）；无任何一级的输入/输出量；无去重环节；无安全过滤环节。

### [Open-Sora 系列](../models/Open-Sora.md)

两个项目的清洗漏斗结构清晰且完整开源/开文档，是本主题下最具可复现价值的样本。
【Open-Sora 2.0（分层数据金字塔，hierarchical data pyramid）】设计理念是「过滤阈值由松到严逐级收紧，形成金字塔状的数据分层，与由低分辨率到高分辨率的渐进式训练课程一一对应」——底层大量宽松过滤数据供低清预训练，顶层少量严格过滤数据供高清精调。具体级次：
  第0级 预处理：剔除损坏文件与离群样本（时长<2s、bpp<0.02、fps<16、宽高比不在 [1/3,3]）；
  第1级 切镜：FFmpeg libavfilter scene score 检测镜头边界并切分，输出统一为 fps<30 / 长边≤1080px / H.264，>8s 切成 8s 段，<2s 丢弃；
  第2级 美学分：CLIP+MLP 美学打分器，取首/中/尾三帧平均；
  第3级 运动分：FFmpeg libavfilter 的 VMAF motion score，剔除过低（静止）与过高（剧烈/混乱）两端；
  第4级 模糊检测：OpenCV Laplacian 算子方差阈值，视频取五个均匀采样帧多数表决；
  第5级 OCR：PaddleOCR 检测置信度>0.7 的文字，文字过多者丢弃；
  第6级 相机抖动：PySceneDetect 的 Shot Boundary Detection，帧间平均变化超阈值判为抖动并剔除；
  第7级 打标：低清阶段 LLaVA-Video、高清阶段 Qwen2.5-Max。
【Open-Sora 1.x（开源代码级 pipeline，docs/data_processing.md）】四阶段：①切镜（PySceneDetect scene_detect → cut）；②质量评估与过滤（aesthetic → optical flow → OCR → datautil 过滤）；③打标与对齐（PLLaVA/LLaVA 打标 → CLIP matching score 计算图文一致性 → caption 清洗）；④在剩余样本上做相机运动检测并把结果写回 caption。
【Open-Sora Plan v1.3（七级漏斗，逐级保留率全公开）】ffmpeg 16s 切分 → LPIPS 跳切检测 → LPIPS 运动过滤 → EasyOCR 字幕裁剪 → LAION 美学过滤 → DOVER 技术质量过滤 → LPIPS 运动复检。特别之处在于**运动过滤做了两次**：先粗筛，在裁掉字幕区域之后再复检一次，因为裁剪改变了画面构成、可能使原先「有运动」的片段实际变成静止。

### [Ovi](../models/Ovi.md)

Ovi 采用「双语料、各自成漏斗」的双轨设计，音视频配对语料是四级串行漏斗，纯音频语料是简化两级流程。

【A. 音视频配对语料 —— 四步 pipeline（论文 3.2 节明确编号）】
第1步 Splitting and filtering（切分与过滤）：场景检测切出 121 帧 @24fps 片段 → 分辨率必须 > 720×720 → RAFT 光流剔除静态视频并产出 motion score → LAION 美学预测器剔除低质数据 → 内部人脸检测模型做单人/多人/无人的构成配比控制。
第2步 Sync detection（同步检测）：SyncNet 产出 confidence 与 offset 标量 → 仅保留 |offset| ≤ 3 且 confidence > 1.5 → 并叠加平均音量 ≥ −60 dB 的音频门槛。
第3步 Captioning（打标）：MLLM 输入 7 帧均匀采样图像 + 完整音轨 → 输出「视觉事件叙述 + <S>台词<E> 交织 + 末尾 <AUDCAP>音频描述<ENDAUDCAP>」的合并 caption。
第4步 Packing（打包）：去黑边 → 保持宽高比缩放到 518400 像素（720×720 等面积）→ 按 24fps 抽帧转为字节数组 → 音频转 raw wave 字节。

【B. 纯音频语料 —— 简化流程】
按时长切两档（预训练 ≤12s、微调精确 5.04s）→ 用与音视频侧同一个 MLLM 产出音频转写（无语音则留空）与音频描述 → 进入训练。无 sync/美学/运动等视觉侧过滤。

【设计取向】整体漏斗把「同步性」置于最高优先级（论文明言「即使少量不同步数据也会损害唇同步能力，因此选择严格标准以最小化错配风险」），其次是分辨率与运动/美学的基础质量门槛，对去重、安全、水印/OCR 等维度则未着墨。

### [Script-a-Video](../models/Script-a-Video.md) ⚠️

需明确区分：本工作的「pipeline」重心在标注 schema 的构建逻辑，而非数据清洗漏斗。清洗流程几乎无披露。
【数据清洗漏斗】论文仅一句话带过：「we curated an internal dataset comprising 500K high-quality video clips from film, television, and lifestyle domains」（我们策划了一个包含 50 万条高质量视频片段的内部数据集，来自电影、电视和生活方式领域）。「high-quality」的判定标准、过滤级数、各级顺序、工具选型均未披露。这是本工作最显著的披露缺口。[不确定]
【标注 pipeline（有充分披露）】实际的处理流程为两步：
第 1 步：用 Gemini-2.5-Pro 对 500K 片段逐条生成 MTSS 格式标注（"Each clip was annotated using Gemini-2.5-Pro"）；
第 2 步：用这批数据对开源 Qwen3-Omni-Instruct 做监督微调（SFT），得到专用标注模型 Qwen3-Omni-MTSS-FT。
即典型的「强教师模型蒸馏到开源学生模型」范式，无中间的质检、复核、重采样环节披露。
【MTSS 标注内部的处理逻辑（本工作真正的方法核心）】可视为一个五步的「信息重组流水线」：
1) 实体抽取与筛选 → 按叙事重要性把实体分为 person/object/animal/scene 四类进入 Reference 流，边缘实体降级到 global scene description；
2) 视觉分段 → 按镜头切分为 Shot 流，每段带 time_range、visual_description、camera 三层信息；
3) 音频事件抽取与分流 → 按「严格音视频耦合」原则筛出 dialogue/sfx/music 三类事件进 Event 流，无视觉对应的背景噪声降级到 global_audio，并发音源拆为并行条目；
4) 全局兜底 → scene_description / global_style / global_audio 三字段承载不属于任何具体流的宏观信息；
5) 关系重连（Relational Grounding）→ 身份链接（references_in_shot 数组指向 Reference ID、Event 的 speaker 指向 Reference ID）+ 时间链接（active_events 把 shot 关联到并发事件、intra-description timestamps 把微动作锚定到全局时间轴）。
这套「先解耦、再重连」的结构是全文的方法论主张：解耦解决冗余与可编辑性，重连解决一致性。
【生成侧 pipeline】见 multi_stage_curriculum 字段（四阶段渐进训练）。

### [Seedance 2.0 与 Seedance 1.5 pro](../models/Seedance_20_Seedance_15_pro.md)

Seedance 1.5 pro 只给出高层描述：一个整体性的数据框架（holistic data framework），由「多阶段筛选管线（multi-stage curation pipeline）+ 先进的字幕/描述系统（advanced captioning system）+ 可扩展基础设施（scalable infrastructure）」三部分构成，管线优先保障视频-音频一致性、运动表现力与基于课程的数据调度。Seedance 2.0 报告未设数据章节，完全未披露漏斗结构（该报告仅含引言、评测、贡献者三部分）。可作为纵向基线的 Seedance 1.0 报告给出了六级顺序漏斗：(1) 多样性导向数据采集 Diversity-Oriented Data Sourcing → (2) 镜头感知时序切分 Shot-Aware Temporal Segmentation（最长 12 秒）→ (3) 视觉覆盖物矫正 Visual Overlay Rectification（logo/水印/字幕/贴图检测 + 自适应裁剪）→ (4) 质量与安全过滤 Quality and Safety Filtering → (5) 语义去重 Semantic Deduplication → (6) 分布再平衡 Distribution Rebalancing；整套流程部署为面向海量数据高吞吐的自动化系统。注意：任务背景中「Seedance 2.0 数据侧披露最完整」的说法与实际报告内容不符——2.0 的 arXiv 报告是评测导向的 model card，数据披露程度反而低于 Seedance 1.0。

### [SkyReels 系列](../models/SkyReels.md)

【SkyReels-V2（arXiv:2504.13074）数据 pipeline】呈「采集 → 镜头切分 → 分级质量过滤 → 后处理伪影清理 → 双轴分桶归一 → 结构化打标 → 概念均衡采样」的链路：
1. 数据采集（开源数据集 Koala-36M/HumanVid + 自采影视 + 艺术素材库，O(100M)）；
2. 镜头边界检测与切分（PyDetect + TransNet-V2）→ 单镜头 clip；
3. 基础质量过滤（低分辨率、低帧率、黑屏/白屏/静止画面、相机抖动）；
4. 视频类型过滤（剔除监控录像、游戏录屏、动画、无意义内容）；
5. 后处理伪影过滤与裁剪（字幕、台标/logo、图像编辑痕迹、分屏、黑边；对可裁剪者做 subtitle/logo cropping 以最大化可用画面）；
6. 多模型打分过滤（美学过滤器、OCR 过滤器、马赛克过滤器、特效/贴纸过滤器，以及 VQA、IQA、VTTS 等模型）；
7. 归一化（双轴分桶 BT×BAR：时长轴 × 宽高比轴，FPS 标准化）；
8. 结构化打标（通用 MLLM + 三个子专家模型 → 蒸馏为 SkyCaptioner-V1）；
9. 概念均衡采样（按主体类别均衡，数据量减半）；
10. 全流程人工在环抽检（预训练0.01%、后训练0.1%）。
【SkyReels-V4（arXiv:2602.21818）数据 pipeline】结构为「采集（真实+合成）→ 视频支路 / 音频支路并行处理 → 音视频同步过滤 → 三类 caption 生成」：
A. 采集：公开数据集 + 授权自有内容 + 合成数据（多语种文字/语音/编辑配对）；
B. 视频支路：预处理（VLM 增强的智能镜头切分）→ 三维度过滤（基础质量 / 内容质量 / 运动质量）→ 均衡（概念多样性 + 运动多样性）→ VideoCLIP 嵌入去重；
C. 音频支路：类别分类（Qwen3-Omni 四分类）→ 质量过滤（SNR / MOS / 削波率 / 音频带宽 / VAD 静音比）→ 时长控制（切15秒块、同类拼接）→ 内容识别（Whisper 转写）→ 音频打标（Qwen3-Omni）；
D. 跨模态：SyncNet 唇音同步过滤（|offset|≤3 且 confidence>1.5，且平均音量≥-60dB）；
E. 打标：短 caption / 长 caption / 带特殊 token 的结构化 caption 三档并行产出。
整体特点：V2 以「浅层判别器矩阵 + 结构化打标 + 概念均衡」为核心，V4 把 VLM/全模态 LLM 提升为切分、分类与打标的主力，并新增音频支路与跨模态同步过滤两条链路。

### [Sora 2](../models/Sora_2.md) ⚠️

训练数据清洗漏斗结构完全未披露。System Card 关于清洗的全部表述仅两句：「Our data processing pipeline includes rigorous filtering to maintain data quality and mitigate potential risks. We also employ a combination of safety classifiers to help prevent the use or generation of harmful or sensitive content, including explicit materials such as sexual content involving a minor.」即只承认存在(1)质量过滤与(2)风险/安全分类器两个大类，未说明级数、顺序、各级判据。相比之下，System Card 对「推理时安全栈」（输入prompt阻断 → 生成 → 输出阻断，含CSAM分类器与自定义训练的多模态推理监控模型）的描述详尽得多——这是本模型披露结构的典型特征：安全与部署侧详尽、训练数据侧近乎空白。[不确定]

### [SpeakerVid-5M](../models/SpeakerVid-5M.md)

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

### [Step-Video-T2V](../models/Step-Video-T2V.md)

核心价值所在。Step-Video-T2V 的数据 pipeline 由六个串行阶段构成，且每个阶段所用的工具/模型基本全部点名，可复现性描述在同期工作中属较好一档：
第1阶段 视频切分（Video Segmentation）：PySceneDetect 的 AdaptiveDetector 检测场景变化 → FFmpeg 切为单镜头片段 → 每片段丢弃首尾各 3 帧。
第2阶段 视频质量评估（Video Quality Assessment）：对抽样帧打 7 类质量标签（美学分、NSFW 分、水印、字幕、饱和度、模糊度、黑边），详见 quality_filtering 字段。
第3阶段 视频运动评估（Video Motion Assessment）：Farneback 光流算法计算 Motion_Mean / Motion_Max / Motion_Min 三个运动量。
第4阶段 视频打标（Video Captioning）：自研 VLM 产出三类文本——Short Caption、Dense Caption、Original Title。
第5阶段 概念均衡（Video Concept Balancing）：内部 VideoCLIP 抽 embedding → K-means 聚成 12 万+ 簇 → 打 Cluster_Cnt 与 Center_Sim 标签 → 重采样均衡与离群剔除。
第6阶段 视频-文本对齐（Video-Text Alignment）：从片段中均匀抽 8 帧，计算帧 embedding 与文本 embedding 的平均余弦相似度得到 CLIP Score，用于剔除图文错配样本。

【分层过滤（Hierarchical Data Filtering）——本 pipeline 的组织方式】上述阶段产出的是一整套「标签体系」而非一次性丢弃：所有片段先被打满标签，再通过逐级抬高各标签阈值，切出 6 个用于 Step-2 T2VI 预训练的子集（Figure 11 以条形图展示各级过滤：灰色条为被该级滤掉的数据，彩色条为留存数据）。这种「先全量打标、后按阈值切档」的设计相比「边过滤边丢弃」更灵活，可在不重跑 pipeline 的前提下调整课程。
后训练阶段在此之上再叠加两级：自动化脚本+启发式规则+按簇心距离剔离群 → 人工评审精选，得到 30M 的 SFT 集。

### [UniTalking](../models/UniTalking.md) ⚠️

三级串行漏斗 + 一级标注 + 一级衍生数据合成，整体结构为「先单模态、后跨模态、再标注」，是本工作数据部分最清晰的贡献表述（论文 Figure 2 专门绘制了该流水线）：
【第 1 级 视频过滤（单模态·视觉）】处理视觉流，剔除三类内容：静态视频（static）、含文字叠加的视频（text overlays）、整体视觉质量低的视频（low overall visual quality）。未披露所用模型与任何阈值。
【第 2 级 音频过滤（单模态·听觉）】处理音频流，剔除三类样本：静音（muted）、不含语音（lack speech）、低信噪比（low SNR）。工具为 PANNs + SentenceASD，用于执行语音事件检测（speech event detection）。未披露阈值。
【第 3 级 音视频跨模态过滤】评估跨模态关系，剔除两类样本：声源不在画面内的纯 diegetic 音频样本（工具 LightASD，主动说话人检测）、唇音对齐差的样本（工具 LipSync）。未披露阈值。
【第 4 级 多层次联合标注】对通过全部过滤的音视频对生成三种粒度/形态的 caption（详见 caption_structure），工具为 Qwen3-VL + Whisper-V3 + Qwen3-Omni-Captioner + Qwen3-Omni。
【第 5 级 参考数据合成】为每条视频用 IndexTTS2 合成 3 条 3–5 秒参考音色音频，构造 TR2AV 任务的训练对。
【结构特征与横向对比】
- 「单模态优先、跨模态兜底」的排序是合理的成本设计：先用廉价的单模态检测淘汰大部分样本，再对幸存者施加昂贵的跨模态检测。UniVerse-1 的六级漏斗本质上也是这个顺序（视觉硬指标 → 音频活动 → 语音检测 → SyncNet）。
- 缺失的环节相当多：无镜头切分、无去重、无运动分数过滤（仅有「静态剔除」的定性表述）、无美学评分器、无 NSFW/安全过滤、无人工质检、无水印/logo/黑边检测。
- 最突出的问题是全流程零阈值披露。论文列出了 6 条过滤规则与 4 个工具名，但没有一个数值——这使得该流水线在「知道做了什么」层面可理解、在「能否复现」层面不可行，与其「开放可复现」的核心主张构成明显矛盾。
【关于「SyncNet 置信度 > 0.9」的核实结论】本次调研对 arXiv:2603.01418v1 的 HTML 与 PDF 全文（共10页）做了全文检索，论文中不存在任何形式的 SyncNet 阈值数值，也不存在「0.9」这一数字（全文出现的 0.9 仅为 AdamW 的 β₁=0.9）。论文唇同步过滤仅表述为「使用 LipSync 过滤唇音对齐差的样本」，未指明具体模型变体、未给出置信度阈值。正文虽提及 Appendix，但 v1 版 PDF 实际未附任何附录。因此「SyncNet conf > 0.9」这一说法无法在一手文献中得到确认。[不确定]

### [UniVerse-1](../models/UniVerse-1.md)

六级顺序漏斗（针对自采数据），VGGSound/AudioSet 走简化旁路。整体是「先卡硬指标、再切镜头、再判音频、最后核唇同步」的逐级收窄结构：
【第 1 级 音视频预筛】缺少音轨的视频立即丢弃——把「必须有原生同步音轨」作为最前置的一票否决条件。
【第 2 级 质量控制（视觉硬指标）】三条并列阈值：分辨率低于 1080p 剔除；码率-分辨率比低于 600 剔除；DOVER 美学质量分低于 0.6 剔除。
【第 3 级 时序连贯性（镜头切分）】PySceneDetect 做场景切分，切分后长度短于 5 秒的片段剔除。
【第 4 级 音频活动检测】分析音量、能量、过零率三项信号级指标，剔除静音片段。
【第 5 级 语音内容核验（分流点）】Whisper ASR 检测是否含语音——含语音者进入语音子集继续走第 6 级；不含语音者不淘汰，直接归入「通用音视频数据」。这是漏斗中唯一的分流而非淘汰节点。
【第 6 级 人脸与唇同步核验（仅语音子集）】RetinaFace 做人脸检测；再用 SyncNet 计算唇音同步置信度，要求 conf > 2.0 方可保留；通过者被显式打上「含语音」标签。
【旁路 VGGSound/AudioSet】仅施加 5 秒最短时长约束，跳过分辨率、码率、DOVER、SyncNet 等全部质量与对齐过滤——因为引入目的是补强音频而非视觉。其低视觉质量的负面影响改由训练侧的 LQLS 损失策略隔离，这是「过滤解决不了就用损失函数解决」的思路，是本工作较有特色的一点。
【第 7 级（在线，训练时）标注】不属于离线漏斗，而是与训练并行的在线标注服务，见 caption_model 与 model_as_data_judge。
【整体特征】与 MOVA 相比，UniVerse-1 的漏斗更短更浅（6 级 vs 3 大阶段十余项指标），无音频美学评分、无语义对齐过滤、无 OCR 字幕过滤、无去重；但把最贵的标注环节从离线搬到在线，是其结构上最大的差异化。

### [Unison](../models/Unison.md) ⚠️

这是本条目披露最薄弱的字段。论文对数据清洗流程的全部描述仅有两处，且极其简略：
【处 1：一句话概括】4.1 节 Training Corpora 末尾：「After refinement through our automated processing pipeline, the final dataset encompasses approximately 2 million synchronized audio-visual clips totaling over 3,000 hours, alongside 50 million high-quality audio segments exceeding 130,000 hours」——仅告知存在一条「自动化处理 pipeline」及其最终产出量，pipeline 的级数、各级顺序、各级方法与阈值均未披露。
【处 2：唯一展开的环节——lip-filtering 算子】位于第 3 节方法开头，作为「词级唇同步三层保障」中的数据层措施描述：
步骤 a) 人脸检测——「first detects the number and location of faces」，同时确定人脸的数量与位置。注意「数量」的检测意味着 pipeline 具备识别多人脸场景的能力（可用于排除多说话人歧义或选择主说话人），但论文未说明检测到多张人脸时如何处理；
步骤 b) 框内 SyncNet 核验——「SyncNet is then applied exclusively within these bounding boxes to verify alignment」，关键设计是 SyncNet 仅在人脸框内运行（exclusively within these bounding boxes），而非在整帧上运行。这是一个有实际意义的工程细节：全帧运行 SyncNet 在多人或人脸占比小的画面中容易失效，限定在检测框内可显著提升同步判定的可靠性；
步骤 c) 双重剔除目标——该算子「自然地过滤掉两类样本」：(i) 唇动与语音不同步的片段（unsynchronized speech and lip movements），(ii) 画外音/旁白配音片段（off-screen voice-overs）。第二类的剔除尤为重要——vlog、纪录片、教程类内容中后期配音极为常见，画面无说话人而音轨有语音，这类样本会严重污染唇同步学习。UniVerse-1 用「RetinaFace 检测到人脸」这一布尔条件间接实现类似效果，Unison 则把它显式列为设计目标。
【处 3：音频解耦（不属于过滤，属于预处理）】所有音频经 Mel-RoFormer 分离为语音与音效两路，作为双流监督的 ground-truth。这是整条流水线中唯一被明确命名的音频预处理步骤，适用于音视频数据与纯音频数据两者。
【可重建的最简 pipeline 骨架】聚合五个开源数据集 → 自动化精炼（细节未披露）→ lip-filtering（人脸检测 + 框内 SyncNet）→ Mel-RoFormer 语音/音效解耦 → 双流 latent 编码 → 训练。
【完全缺失的环节】无镜头切分描述、无美学/清晰度过滤、无 OCR/水印/黑边检测、无运动过滤、无去重、无安全过滤、无 VLM 质检、无逐级保留率、无任何阈值数值。与 MOVA、UniVerse-1、LTX-2 等同期工作相比，Unison 的数据 pipeline 披露度处于最低档——但需公允指出，这是由其定位决定的：视频骨干冻结意味着视觉侧数据质量的边际影响极小，团队的注意力合理地集中在音频与对齐两个真正被训练的维度上。[不确定]

### [Veo 3 / Veo 3.1](../models/Veo_3_Veo_31.md) ⚠️

官方披露的清洗漏斗极其简略，仅可还原为四个可确认环节（顺序为推测）：(1) 多粒度 caption 标注——用多个 Gemini 模型对音频与视频片段生成不同详细程度的文本描述；(2) caption 侧过滤——剔除不安全 caption 与含个人可识别信息（PII）的描述；(3) 视频侧过滤——按合规性（compliance）、安全性（safety）与质量（quality）三类指标过滤训练视频，并按风险领域对预训练数据做安全过滤；(4) 语义去重——在所有数据源之间做跨源语义去重，同时移除重复及概念高度相似的视频。此外还有一项非过滤性的数据分析环节：对训练数据做有害内容分析与人群表征公平性审查。[不确定] 各级过滤的具体层级数、执行顺序、判定模型与阈值均未公开。

### [Vidu S1](../models/Vidu_S1.md)

六级渐进式过滤漏斗（论文 Figure 2 明确画出，从 Raw Videos 到最终训练数据）：
第1级 Prefiltering（预过滤）：先做去重，再按帧率、分辨率、音视频完整性、音视频同步性四项技术指标剔除不可靠视频 → 输出 Pre-filtered；
第2级 Clipping（切分）：沿镜头边界切为单镜头 clip，长镜头细分且切点避开语音中段，叠加时长过滤（Duration Filter）保留 3~60 秒 → 输出 Single-Shot Clips；
第3级 Subject Filtering（主体过滤）：Subject Filter 保证画面中恰好一个主体且占比合理 → 输出 Single-Person Clips；
第4级 Other Filtering（综合过滤）：并列四道 —— 画面内容/视觉质量过滤（Frame Content / Visual Quality Filter）、内容安全过滤（Content Safety Filter）、镜头稳定性过滤（Shot Stability Filter）、交互性过滤（Interactivity Filter）；
第5级 Diarization（说话人分离）：VAD + ASD 标注语音段与说话人，onscreen/offscreen/overlap 三分类，Overlap Filter 剔除重叠人声片段，Speech Energy Filter 剔除语音能量占比过低（唱歌/强背景音乐）片段；
第6级 Caption + Embedding：生成 clip 级与 speech-aware chunk 级双粒度结构化 caption，最终入选 clip 与其 caption 一起被 embedding 成训练数据。
设计目标被总结为同时提升视觉清晰度、时序稳定性、音视频一致性与跨模态可解释性。

### [Wan 2.5 / 2.6 / 2.7](../models/Wan.md) ⚠️

2.5/2.6/2.7 未披露。Wan 2.1 报告第3章给出了业界少见的完整四步漏斗，是本条目最核心的可复用资产；Wan2.2-S2V 则给出了人本/音画专用的层级化漏斗。
【Wan 2.1 预训练四步清洗（four-step data cleaning process）】
第0步 候选池构建：内部版权数据 + 公开可获取数据，先做去重（curated and deduplicated）。
第1步「基础维度」（fundamental dimensions）——针对源数据固有属性的高效粗筛，由8+项轻量检测并联组成：OCR 文字覆盖率检测、LAION-5B 美学分类器初筛、内部 NSFW 安全模型、水印与 logo 检测、黑边检测、过曝检测、合成图检测、模糊检测、时长与分辨率门槛。此步「成功淘汰约50%的初始数据集」。
第2步「视觉质量」（visual quality）——语义驱动的精筛，拆为「聚类 + 打分」：先分100簇按簇配额采样以保持原始分布/防长尾丢失，再用人工1–5分标注训练的专家评估模型对全量打分。
第3步「运动质量」（motion quality）——六档运动质量分级，按档位分别执行保留/降采样/排除。
并行分支「视觉文本数据」（visual text data）——两条支路：白底渲染中文字符合成数亿张文字图 + 真实世界含文字图像经多 OCR 模型识别后送入 Qwen2-VL 生成含精确文字内容的自然描述。
第4步 后训练精选（3.2节）——图像走「专家模型 top20% + 人工精选」，视频走「视觉质量分类器 top + 运动质量分类器分选简单/复杂运动 + 12大类类别均衡」。
最后 打标阶段（3.3节）：自研 LLaVA 式 dense caption 模型对全量图像与视频重新打标。
【结构特点】典型的「廉价并联粗筛（砍掉50%）→ 聚类配额 + 学习式打分精筛 → 运动语义分级调采样 → 高质量子集精选 → 全量重打标」五段式；判别器以专家小模型为主，MLLM 只在打标与评测环节出现。
【Wan2.2-S2V 层级化人本漏斗（论文 Fig.1）】
1) 数据收集：开源数据集自动粗筛（caption 中含人物相关描述）+ 人工精选含说话/唱歌/跳舞等复杂人体活动的视频 → 百万级人本视频池；
2) 姿态跟踪：VitPose 提取2D姿态并转 DWPose，兼作控制信号与筛选依据；
3) 细粒度筛选：剔除人物在时间或空间维度占比过小的视频；只保留全序列中人脸持续一致可见的视频（保证能从音频学到面部表情）；
4) 视频质量五项：Dover 清晰度、UniMatch 光流运动分、Laplacian 算子对人脸/手部区域的锐度检验、改进版美学预测器、OCR 字幕遮挡检测；
5) 音画对齐：Light-ASD 主动说话人检测，剔除音频与活跃说话人不同步、或画面中不存在活跃说话人的视频；
6) 密集打标：QwenVL2.5-72B 生成结构化 dense caption。

### [音视频生成评测基准合集](../models/av_benchmarks.md)

五者的「清洗流程」实为评测数据策展流程，但其漏斗结构对训练数据 pipeline 有直接借鉴价值：

【PhyAVBench 五阶段策展流程（最完整）】① 音频物理知识调研 —— LLM 头脑风暴候选物理原理，人类专家剔除不可行/冗余/无关条目；② 分类体系构建 —— LLM 生成层级结构，专家审核消歧去冗；③ 物理约束提示词对设计 —— LLM 生成候选模板，专家人工核验并改写，保证每对提示词仅在单一物理变量上有差异、其余非目标条件全部不变，并避免主观描述与对预期声学结果的显式提示；④ 真实音视频采集 —— 全新录制，可控环境，跨个体/演示者/设备多样采样；⑤ 迭代式质控与过滤 —— LLM 先做语义歧义与非预期物理混杂因素的初筛，人类复核物理一致性、文本-音频-视频三方对齐与现实合理性，问题样本删除、修订或重新采集。这是一条典型的「LLM 生成 → 专家修订 → 采集 → LLM 初筛 + 人工复核 → 迭代回流」闭环。

【AV-SyncBench 三级漏斗】① 网络采集 in-the-wild 视频；② Gemini 3 Flash 自动过滤 —— 剔除画外声源（off-screen sound sources）样本与明显视听错配样本；③ 人工复核 —— 5 名标注员，每条 clip 至少由 3 人独立审核，确认主声源在画面内可见，并剔除音质差、噪声过大、语义模糊的片段。之后进入第四步：程序化扰动生成时序与语义负样本。这是本次调研中「大模型初筛 + 多人交叉人工复核」的标准范式样板。

【VABench 双路策展】T2AV 路：LLM + 专家模板批量生成原始提示词 → LLM 结构化解耦为视觉子提示与听觉子提示 → 生成 VQA/AQA 问答对 → 人工核验类目归属正确性、要素可观测性、物理与常识约束满足性。I2AV 路：人工精选并分类高质量图像（含隐私筛查）→ 多模态 LLM 生成统一视听描述（视觉客观 + 音频常识推断）→ 描述同时用于构造 VQA/AQA 并由 LLM 解耦为子提示 → 人工复核听觉推断的合理性与问题的区分度。论文明确表述「employed human workers and large language models to filter testing samples and adjust the distribution of test data」——即人机协同同时承担过滤与分布调整两项职责。

【AVBench 两条流水线】评测集路：从提示词池按 Hard Quota-Based Greedy Sampling 采样 470 条 ≥720p 提示词，配额约束单属性占比 ≤50%，再分层为 Normal/Hard 子集。训练集路：OpenHumanVid 抽取 30K 真实片段作正例 → LLM 驱动扰动 + 算法性错配生成硬负例 → 每维度扩展至 100K 对 → 三维度合计 300K。

【Omni-Judge】无数据清洗流程，属元评测：300 条 VidProM 提示词 → Sora 2 / Veo 3 各生成 1 条 → 6 名博士生按 9 维度打分 → 计算 Omni-LLM 判分与人类判分的相关性。

### [视频 Caption 模型生态](../models/caption_models.md)

本条目的「清洗流程」有两层含义，需分开：(A) captioner 自身训练数据的清洗；(B) captioner 在生成模型数据 pipeline 中所处的位置与承担的过滤职能。
【(A) captioner 训练数据的清洗漏斗】普遍简单，典型为「教师模型生成 → LLM 打分过滤 → SFT → RL」四段：
· AVoCaDO：Gemini-2.5-Pro 分别生成 visual caption 与 audio caption → 把两条 caption + 原视频再喂回 Gemini-2.5-Pro 合成时序连贯的多模态 caption → GPT-4.1 对「synthesis completeness」打 1–5 分，只保留 ≥4 分 → SFT → GRPO。
· AVSCap：三阶段 —— decoupled unimodal anchoring（解耦单模态锚定）→ cross-modal fusion（跨模态融合）→ automated verification（tag 保留检查 + 语义一致性检查）→ SFT → GRPO。
· CogVideoX 教师链：Panda-70M 短 caption → CogVLM 每 2 秒一帧的稠密图像 caption → GPT-4 按时间戳摘要 → 5 万条数据微调 LLaMA2 替代 GPT-4 → 再蒸馏为端到端的 CogVLM2-Caption。这是「四段式教师链逐级降本」的经典样本。
· SkyCaptioner-V1：Qwen2.5-VL-72B 产出通用描述 + 三个子专家 captioner（镜头/表情/相机运动）补充影视专业维度 → 融合 → 蒸馏进 7B 统一模型。
· Panda-70M：31 个候选打标器并行生成 → 用户研究做贪心集合覆盖选出 8 个 → UMT-large 细粒度视频-文本检索择优。这是唯一一个把「打标器选择」本身做成一道过滤工序的工作。
【(B) captioner 在生成 pipeline 中的位置】三种典型摆位：
· 漏斗中段的语义闸门：Allegro 把 Tag2Text 放在第 6 级，其标签输出直接作为第 7 级 CLIP 相似度过滤的文本侧输入 —— 打标器同时是过滤器的上游依赖。
· 漏斗末端的全量打标：绝大多数工作（HunyuanVideo、Step-Video、Movie Gen、Seedance）在所有过滤通过后才做打标，因为打标是最贵的一步。
· 分级打标金字塔：Open-Sora 2.0 低分辨率（256px）海量数据用开源 LLaVA-Video 打标、高分辨率（768px）精选 5M 数据改用 Qwen 2.5 Max 重标 —— 「粗标底层 + 精标顶层」与数据金字塔严格对应，是低成本策略的核心一环。Movie Gen 的 70% 8B / 30% 70B 混比是同一思路的另一实现。
· 在线打标（罕见）：UniVerse-1 把标注放进训练循环，因此被迫选择轻量模型（Qwen2.5-Omni + Whisper），无法承受 120B 级模型的逐样本推理开销 —— 这是「标注时机」与「标注模型容量」之间权衡的唯一公开案例。

### [几何/结构化标注数据集合集](../models/geometric_datasets.md)

SceneScribe-1M（三级漏斗+标注）：①规格过滤（分辨率>1080p、帧率≥10fps、时长5秒–1分钟）→②Qwen2.5-VL-72B 六维内容质量过滤（运动强度未知、可见水印、强相机畸变、强光照伪影等剔除）→③TransNetV2 时序切分（仅对非连续视频）并对切出的clip重新过滤→④几何标注（MegaSaM 相机+深度、TAPIP3D 3D点轨迹）+语义标注（Qwen2.5-VL-72B caption）；SpatialVID（人工初筛→切分→四维质量过滤→几何重建→双阶段caption）：①人工筛选（剔除损坏内容、标题含不当词或“全景相机”字样、与 MegaSaM 重建不兼容者）→②改造 PySceneDetect 切 3–15 秒clip并统一转码 1280×720 H.265→③四项质量过滤（美学、亮度、OCR文本占比、运动/VMAF）→④MegaSaM 相机位姿+深度重建（深度模块替换为 UniDepth v2 与 Depth Anything v2）→⑤SAM2 动态掩膜与动态比例计算→⑥轨迹运动指标（MoveDist/RotAngle/TrajTurns）与加速度异常检测→⑦Gemini-2.0-Flash 视觉解析 + Qwen3-30B-A3B 语言精修的双阶段结构化caption→⑧HQ子集均衡采样；WildWorld（引擎直采，无传统清洗漏斗）：基于 OBS Studio + Reshade 的采集平台将显示分区为子窗口，同步时间戳同时录制 RGB(720p) 与深度，并从游戏内存/引擎导出骨骼、世界状态、相机内外参，逐帧119列结构化标注，再用VLM生成两级caption；Action100M（标注型流水线，几乎不做视频过滤）：①V-JEPA 2 编码器把视频转为时序稠密视觉嵌入（每4帧采1帧，64帧重叠窗口、步长8）→②带时序约束的层次凝聚聚类（Ward linkage）生成时间树→③Llama-3.2-Vision-11B 帧级caption + Perception-LM-3B 片段级caption→④GPT-OSS-120B 做证据聚合与结构化抽取，三轮 Self-Refine→⑤去重与语义重采样

### [视频生成后训练数据策略](../models/post_training_data.md) ⚠️

【锚论文的四阶段后训练流水线（本条目的骨架）】Phase 1 SFT（用精选数据建立稳定的指令跟随基线与参考策略）→ Phase 2 RLHF（基于 GRPO 的训练器，用多维奖励对齐美学、运动质量、文本对齐）→ Phase 3 PE 提示增强（用同一套奖励回路 GRPO 训练 LLM 改写用户输入）→ Phase 4 AD 自回归蒸馏（self-forcing 目标，把能力迁移到因果架构以提升推理效率）。
论文的核心论断是「SFT as the foundation for RLHF」：SFT 的目标不是解决对齐或优化主观质量，而是建立一个稳定、结构良好的参考策略（stable and well-structured reference policy），消除最严重最频繁的失败模式，防止后续 RL 发散到退化行为，并且「SFT also enlarges the exploration for RLHF」（SFT 还扩大了 RLHF 的探索空间）。第二条论断是「PE complements RLHF」：RLHF 优化输出侧生成策略，PE 优化输入侧 prompt，二者用同一套奖励（人类偏好、视觉真实感、语义对齐）训练，形成输入-输出双侧对齐。
【重要质量提示】3.1 节 SFT 的具体表述明显是从 LLM 后训练文本迁移而来——所列举的失败模式为「refusal cascades（拒答级联）、incoherent reasoning（推理不连贯）、unsafe outputs」，这些是语言模型的失效模式而非视频扩散模型的失效模式（视频侧应为手部错误、文字错误、快速运动崩坏等，正文引言中恰好提到过）。该段落不含任何视频特有的数据构造细节，读者不应把它当作可复现的 SFT 数据方法论。
【横向的后训练流水线形态谱系】
· 四段式（SFT→RLHF→PE→蒸馏）：锚论文、Seedance 1.5 pro（SFT→AV 定制 RLHF）、Kling 3.0 Omni（quality-tuning SFT→DPO）；
· 三段式 CT→SFT→RLHF：HunyuanVideo 1.5（且 T2V 与 I2V 全程分开）；
· 两段式 SFT→DPO：Step-Video-T2V（Step-3 SFT → Step-4 Video-DPO）、SkyReels-V2（概念均衡 SFT→高质量 SFT→三阶段 DPO）；
· 两段式 SFT→GRPO：LongCat-Video、Cosmos-Predict 2.5（五域 SFT→模型合并→GRPO）；
· 只有 SFT 无偏好学习（开源/学术侧的绝大多数）：Movie Gen、CogVideoX、Allegro、Goku、Motif、MAGI-1、Open-Sora 系列、NAVA、ALIVE、Apollo；
· 完全无后训练：MOVA（把 SFT 融进预训练课程末端）、Unison、UniTalking、UniVerse-1、HunyuanVideo-Foley、Foley-Omni、CineDance、Mochi 1；
· 用推理时搜索替代后训练：ITS-JAVG（多验证器 + Best-of-N/EvoSearch，JavisDiT 5 samples、MMDisCo 10 samples），主张不做后训练也能达到可比效果。

### [主流视频预训练数据集合并调研：Panda-70M、InternVid、Koala…](../models/pretraining_datasets.md)

**七条漏斗的结构差异，本质是「用什么当质检员」的差异**，可归纳为三代技术水位：
【第一代·纯启发式打分器 + 阈值】
- **InternVid（最简）**：爬取限定（时长10s–30min、分辨率360P–720P、排除已有公开数据集）→ NSFW二分类器 → PySceneDetect(阈值27)切镜 → 剔除静止与极端动态片段 → 多尺度打标 → 事后用 UMT-SIM 与美学分派生子集。**过滤与打标基本无阈值披露，可复现性最低。**
- **Panda-70M（切分为核心，过滤极轻）**：其漏斗重心不在过滤而在**切分**——PySceneDetect(阈值25, min_scene_len=15帧) → 长于5秒的递归切出前5秒（发布代码改为7秒） → ImageBind首尾特征距离≤1.0保留 → 相邻片段距离≤0.6则缝合 → 丢弃<2秒及运动过小(距离≤0.15)者 → 与前序片段距离>0.3去重 → 首尾各裁10% → 八教师打标 → UMT检索模型选caption。**没有美学过滤、没有清晰度过滤、没有OCR过滤**——这是它后来被 OpenVid/Koala/MiraData 集体批评的根源。
- **OpenVid-1M（比例式过滤，四道）**：源数据集汇集 → 级联切镜 → LAION美学分（Panda-50M取**前20%**，其余三源取**前90%**）→ CLIP相邻帧余弦相似度双向剔除（过高=近静止、过低=闪烁）→ UniMatch光流双向剔除 → DOVER-Technical清晰度取**前30%** → LLaVA-v1.6-34B重打标。特点是**全部用百分位而非绝对阈值**，可移植性好但绝对质量不可控。
【第二代·MLLM 作为语义质检员】
- **LVD-2M（三级，全靠MLLM兜底语义）**：>10秒长度门 → PySceneDetect **ContentDetector(阈值50, min_scene_len=0, 0.5fps采样)** 剔切镜 → RAFT光流(2fps, 520×960)均值**<20则丢弃** → **PLLaVA-7B 取8帧做 GOOD/BAD 二分类**，两个prompt分别管「内容变化度」与「视觉多样性+文字占屏」。**没有美学打分器、没有OCR模型**——把这两件事全部外包给了MLLM的语义判断，是七者中最早、最彻底的「VLM-as-judge」实践。
- **UltraVideo（四阶段，统计+模型双层）**：源头控制（5,000条4K/8K，人工复检）→ PySceneDetect AdaptiveDetector两轮 + DINOv2首尾5帧补捉溶解 + 按帧数分流short/long → **统计过滤**（PaddleOCR文字面积、黑边、过曝、灰度四项，统一采用「坏帧率>5%则丢弃整条」的模式）→ **模型净化**（VTSS<0.01剔除、RAFT光流保留0.1–100、VideoCLIP-XL-v2图文相似度<0.2剔除、**Qwen2.5-VL-72B 对16种低质属性做二元判断，命中任一即删**）→ Qwen2.5-VL-72B 十维打标 + Qwen3-4B 汇总。
【第三代·把多个子指标喂给一个学习出来的打分网络】
- **Koala-36M（唯一一家反对多阈值范式）**：与 Panda-70M 同源数据 → Color-Struct SVM 转场检测重切分 → LLaVA系结构化打标 → **单一 VTSS 打分门限2.5**。其核心论证是**子指标彼此不正交**（清晰度-美学 Pearson 0.3774、清晰度-运动 −0.4028、运动-美学 −0.2515），多阈值串联会累积误差（附录D表8：仅清晰度阈值偏移10%误滤25万条/4800万，三个阈值同时偏移10%误滤34万条），因此改用一个**吃「视频像素 + 子指标」联合输入的网络直接回归出一个标量**。
【MiraData（结构最特殊：过切→缝合→筛选→打标）】
收集 → PySceneDetect(阈值**26**，刻意低)过度切分 → **四模型投票缝合** → 四项筛选（画面色彩、LAION美学、RAFT运动强度、Stable Diffusion Safety Checker）在**统一2fps**下计算，产出330K/93K/42K/9K四档递增严格度 → Panda-70M短caption做hint + GPT-4V一轮对话产出5个结构化字段。
**横向结论**：过滤严格度排序大致为 UltraVideo > MiraData(9K档) > Koala-36M ≈ LVD-2M > OpenVid-1M > InternVid > Panda-70M；而「质检员智能程度」排序为 UltraVideo(72B VLM) > LVD-2M(7B VLM) > Koala-36M(学习式打分网络) > OpenVid/MiraData(浅层打分器) > Panda-70M(几乎无)。

## 漏斗定量保留率（每级过滤的输入/输出量与最终保留率，如Apollo 27%）

`funnel_retention_rate` · 详细程度: detailed

### [Allegro](../models/Allegro.md) ⚠️

论文给出了较为罕见的分阶段定量保留率（以原始规模为分母）：
· 图像：412M 原始图像 → 107M（约 26% 保留率），用于 T2I 预训练。
· 视频（360p 档，阈值最宽）：500M 原始片段 → 48M（保留率 9.6%）。
· 视频（720p 档）：500M → 18M（保留率 3.6%）。
· 视频（微调档，阈值最严）：500M → 约 2M（保留率 0.4%）。
即从最宽到最严，视频保留率跨越 9.6% → 3.6% → 0.4%，最终高质量精选集仅为原始量的千分之四。
局限：论文只给出「整条漏斗的端到端保留率」，未逐级披露每一道过滤器单独的输入/输出量（例如 PySceneDetect 切分后有多少、美学过滤淘汰了多少），因此无法定位是哪一级贡献了主要淘汰量。[部分不确定：逐级 per-filter 保留率]

### [Apollo](../models/Apollo.md)

【核心数字】整体过滤后保留率 27%，论文原文：「with an overall post-filtering retention rate of 27%」。这是本次调研中少见的公开端到端定量漏斗指标，与 MOVA 的 26.39% 处于同一量级，两者相互印证了「音视频联合生成的数据漏斗大致只留下四分之一多一点」这一经验规律。
【口径】论文未说明 27% 是按样本条数、总时长还是原始视频条数计算，从上下文（紧接在「81 million samples」之后）推测为样本/片段条数口径，但未明示。若按条数口径反推，过滤前候选样本约 3 亿条（81M ÷ 27% ≈ 300M）——此为推算而非论文数据。
【逐级缺失】与 MOVA 公开逐级保留率表（100% → 84.57% → 58.75% → 26.39%）不同，Apollo 只给出端到端的单一总数，四个阶段各自的输入/输出量、各级淘汰占比、各子过滤（视频质量 / 音频质量 / 音视频一致性）各自砍掉多少均未披露，因此无法定位主要损耗环节。
【结论】27% 是一个有价值但孤立的锚点数字，可用于横向对标，不足以支撑 pipeline 逐级复现。

### [CineDance / CineDance-1M](../models/CineDance.md)

论文 Tab.3 给出完整定量漏斗，是该工作披露最扎实的部分之一：
| 阶段 | 计量单位 | 数量 | 时长 |
| 原始采集 Raw collection | 视频 | 45,181 | 32.8K 小时 |
| 时空预过滤 Spatiotemporal pre-filter | 视频 | 44,579 | 32.5K 小时 |
| 镜头检测 Shot detection | 镜头 | 25,899,474 | 32.5K 小时 |
| 叙事解析 Narrative parsing | 序列 | 1,201,912 | 32.5K 小时 |
| 序列剪枝 Sequence pruning | 序列 | 1,079,382 | 28.6K 小时 |
| 后验证 Post-verification | 序列 | 1,021,657 | 26.3K 小时 |
【关键保留率】
  - 视频级预过滤保留率 98.7%（45,181 → 44,579），时长 32.8K → 32.5K 小时，说明入口素材本身质量已较高（因来自 MiraData/LVD-2M/Koala-36M 等已清洗数据集）；
  - 叙事解析后的序列剪枝保留率 89.8%（1,201,912 → 1,079,382 条），时长 32.5K → 28.6K 小时；
  - 后验证保留率 94.7%（1,079,382 → 1,021,657 条），时长 28.6K → 26.3K 小时；
  - 从解析出的序列到最终数据集的总保留率约 85.0%（1,201,912 → 1,021,657）；
  - 时长维度总保留率约 80.2%（32.8K → 26.3K 小时）。
【压缩比】2589 万原子镜头压缩为 120 万叙事序列，平均每 21.5 个镜头合成一条序列。
【伪影审计对比】人工审计 500 条随机片段，CineDance-1M 的不合规率 2.8%，而 Koala-36M 为 37.4%，改善 13.4 倍。

### [CogVideoX](../models/CogVideoX.md) ⚠️

[不确定] 论文未给出任何逐级定量漏斗（各级过滤的输入/输出量与保留率）。仅有的两个可量化点：
· 过滤后最终剩余约 35M 单镜头 clip，但未披露原始视频池规模，因此无法反推整体保留率。
· 高质量微调阶段的保留比例明确为「占总数据量的 20%」（a subset of higher quality video data, accounting for 20% of the total dataset）——这是全文唯一的显式比例数字。
· 附录 K 给出的是 6 个分类器在测试集（随机 10% 标注数据）上的混淆矩阵，可间接看出各类负样本在标注池中的占比：Editing 类 TP+FN≈0.89（负样本占比极高）、Low Quality 类 TP+FN≈0.89、Static（运动不连贯）类 TP+FN≈0.52、Lecture 类 0.53、Text 类 0.62、Screenshot 类 0.62；但这是人工标注采样池的分布而非真实数据分布，不能等同于漏斗保留率。

### [Cosmos-Predict2.5](../models/Cosmos-Predict25.md) ⚠️

论文给出了明确的端到端定量漏斗，且与前代做了直接对比，是本调研中定量披露最充分的案例之一：
【Cosmos-Predict2.5】原始素材 3500 万小时 / 2 亿+ 条原始视频 → 切分裁剪并剔除 <5 秒片段后得到 60 亿+ 条候选 clip（5–60 秒）→ 经七级过滤 + 语义去重后「Only about 4% of the initial clips pass all filters」→ 约 2 亿条可训练 clip。即 clip 级总保留率约 4%（6B → 200M，约 1/30）。
【Cosmos-Predict1 对比】原始 2000 万小时，clip 级存活率 30%。论文明确表述：「it achieves improved data quality control through a far stricter multi-stage filtering pipeline, which reduces survival from 30% of clips to only 4%」——即在原始素材量放大 1.75 倍（20M→35M 小时）的同时，把存活率压缩到原先的 1/7.5，是一次刻意的、大幅度的「以严格换质量」策略切换。
【领域侧】Smart Spaces 为「关键词召回 → VLM 相关性核验 → 切分 → 过滤 → 约 40K clip 存活」，但未给出召回端的输入量，无法计算该域保留率。
【未披露】七个过滤子级各自的输入/输出量与逐级通过率（只有端到端的 4% 总数），以及去重环节单独去掉了多少。[不确定：各过滤子级的分级保留率]

### [Data-Juicer 2.0](../models/Data-Juicer.md) ⚠️

【官方唯一公开的定量保留率——文生视频案例】
  · 输入：InternVid 606k + Panda-70M 605k + MSR-VTT 6k ≈ 1,217,000 条视频-文本对。
  · 输出：147,176 条，约 227.5GB。
  · 端到端保留率：12.09%（HuggingFace 数据集卡片明确标注）。
  这个数字与 Apollo 的 27%、UniVerse-1 的约19% 同处一个量级但更严格，说明 Sandbox 搜索出的最优策略倾向于「宁缺毋滥」——仅保留约1/8的数据。
  · 另有一个 228k 条的数据池用于最终登顶 VBench 的模型（保留率约18.7%），论文记该配置对应640k训练样本量，即在228k基础上做了约2.8倍的数据重复。
【逐级拆解】[不确定] 未公开两个算子各自的淘汰量：即 video_nsfw_filter 单独淘汰多少、video_frames_text_similarity_filter 单独淘汰多少。从算子性质推断，NSFW 过滤在这三个已经过初步清洗的公开数据集上淘汰率应较低（可能个位数百分比），12.09% 的保留率主要由 CLIP 相似度阈值 0.306337 贡献——即绝大多数样本因「视频画面与文本描述不够对齐」被剔除。这一推断与阈值数值本身相符：0.306 对 CLIP 相似度而言属于偏高的门槛。
【探测阶段的池划分口径】Probe 阶段各单算子池按低/中/高三等分，即每池保留率固定为33.3%；这是为了公平比较而设的等大约束，不代表实际推荐保留率。
【系统级吞吐口径的"保留率"】DJ 2.0 论文侧重处理效率而非保留率，未给出跨项目的通用保留率统计。

### [Foley-Omni](../models/Foley-Omni.md) ⚠️

定量披露不足，无法构建完整漏斗表。
【已知端点】pipeline 输出约 2.0M 条视频-音频-文本三元组；全部训练语料约 2.7M 对、约 4.9k 小时。
【缺失】[不确定] 论文未给出清洗前的原始池规模（小时数或条数），未给出任何单级过滤的输入/输出量对比，未给出六项指标各自的淘汰占比，也未给出第三级声学后验证中被置空的字段比例（这个数字本可直接量化 Gemini 视觉幻觉的严重程度，是该方法最有说服力的证据，可惜未报告）。因此无法计算类似 Apollo 27% 那样的端到端保留率。
【可做的粗略推断】六项过滤指标中，Synchformer sync score ≥0.2 与 ImageBind ≥0.3 通常是野外音视频数据的主要杀手（大量网络视频存在配乐后期添加、旁白画外音、声画无关等情况），若参照同类工作（如 UniVerse-1 从 40k+ 小时筛至 7,685 小时约19%、MMAudio 系工作在 VGGSound 上的对齐过滤淘汰率常在50%以上），推测本pipeline原始池应在万小时量级，端到端保留率大致在20–50%区间——但这纯属外推，论文无任何数据支撑。

### [Goku](../models/Goku.md) ⚠️

[不确定]。论文给出了每一级过滤的判定阈值（Table 4）与最终数据量（160M 图 / 36M 视频），但**未披露任何一级的输入量、输出量或保留率**，也未给出原始采集规模，因此无法计算整体漏斗保留率（对比 Apollo 披露 27% 保留率的做法，Goku 在这一点上透明度较低）。
唯一可反推的定量收缩线索是按分辨率档位的数据量级联：480p 档 36M → 720p 档 24M（相对 480p 保留约 66.7%）→ 1080p 档 7M（相对 480p 保留约 19.4%，相对 720p 约 29.2%）。但这一收缩是「分辨率+更严阈值」的复合结果，不等同于单级过滤保留率。
此外 I2V 后训练用 4.5M 三元组，相对 36M 视频池约占 12.5%，可视作高质量子集比例的一个间接参考。

### [Hailuo / MiniMax Video](../models/Hailuo.md) ⚠️

[不确定]（完全无披露）。无任何一级过滤的输入/输出量、无最终保留率。由于连漏斗结构本身都未公开，定量保留率更无从谈起。

### [HunyuanVideo-Foley](../models/HunyuanVideo-Foley.md) ⚠️

论文未给出任何定量漏斗数据：未披露原始采集视频的总量（小时数或条数），未给出七级过滤中任何一级的输入/输出量，未给出各级或整体保留率，也未给出各级淘汰的主要原因分布。仅能确知最终产出为约 10 万小时。分子已知、分母完全缺失，无法计算类似 Apollo 27%、MOVA 26.39% 的整体保留率。
【可定性推断的强度】七级串联中至少有四道具备实质淘汰力的关卡：无音轨剔除（在网络视频中淘汰面广）、静音占比 >80%（较宽松，淘汰量应较小）、有效采样率 >32 kHz（相当严格——大量网络视频音轨为 44.1 kHz 但经压缩后有效带宽不足，此条可能是淘汰率最高的一级）、AudioBox 美学 + SNR（阈值未知，力度不明）、ImageBind + AV-align 对齐（阈值未知）。综合判断整体保留率应在个位数到二十几个百分点之间，即原始数据量级可能在数十万至数百万小时。但这纯属推断，无任何论文数据支撑。[不确定]
【披露缺失的意义】论文把「可扩展的数据 pipeline」列为三大贡献之首，却不给任何漏斗数据，使得「可扩展性」这一主张无法被量化验证——读者既不知道 pipeline 处理了多少原始数据，也不知道各级的计算成本占比，因此无从判断该 pipeline 相比同类方案的效率优势。这是本工作论证上的一处实质性短板。

### [HunyuanVideo](../models/HunyuanVideo.md)

【原版】只有相对口径，无绝对数字：论文原文称各级过滤「A large portion of data will be removed at each stage, ranging from half to one-fifth of the data from the previous stage」——即每一级只保留上一级的 1/2 到 1/5（换算为保留率 20%–50%，淘汰率 50%–80%）。若四级串联按此区间估算，256p→SFT 的整体保留率大致落在 0.16%–6.25% 的宽区间，论文未给出终值。
【1.5】有绝对数字可算，是同类工作中较罕见的定量漏斗：
· 原始池 >1000万小时视频 → 切分+过滤后 8 亿片段（按平均6秒计约133万小时，粗估整体保留率约 13%，此换算为本调研推算而非论文原文）；
· 8亿（256p）→ 2亿（480p）：保留 25%；
· 2亿（480p）→ 1亿（720p/16fps）：保留 50%；
· 1亿 → 1亿（720p 16fps→24fps）：保留 100%（同规模换帧率）；
· 1亿 → 100万（CT/高质量档）：保留 1%；
· 图像侧：>100亿 → 50亿（保留 <50%）→ 10亿（二阶段保留 20%）。
从 8 亿到 100 万的端到端保留率为 0.125%，是本调研中定量最完整的漏斗之一，可直接与 Apollo 的 27% 等口径对照（注意口径不同：Apollo 为单级过滤保留率，此处为多级串联终值）。

### [InstructAV2AV](../models/InstructAV2AV.md) ⚠️

[不确定] 完全无定量披露，这是本工作在数据方法论上最大的缺口。
【已知的唯一端点】最终产出 79K 训练对 + 1K 评测对（HF 实际发布 87,074 + 1,000）。
【全部缺失项】
  · 原始视频池规模未知——未说明从 YouTube 爬取了多少、从 MovieBench/Condensed Movies/Short-Films-20K/VGGSound 各取了多少小时或多少条。
  · 阶段一六道过滤各自的淘汰率未知——PySceneDetect 切出多少片段、CoTracker3 运动过滤淘汰多少、LAION 美学淘汰多少、-45 dBFS 静音淘汰多少、Audiobox 淘汰多少、TalkNet/声源归属判定淘汰多少，均无数据。
  · 阶段二生成量未知——data engine 实际合成了多少候选 target。
  · 阶段三验证淘汰率未知——五维度 MLLM 验证否决了多少比例的合成样本。这个数字尤其可惜：它直接量化了数据引擎的成功率，是评估该pipeline工程可行性（合成一条可用样本需要尝试几次、成本几何）的关键指标，也是「可扩展数据合成pipeline」这一自我定位最需要的证据。人工验证从多少候选中筛出 1K 同样未报告。
【无法计算端到端保留率】由于两端数据均缺失，无法给出类似 Apollo 27% 的整体保留率。
【可做的粗略外推】参考同类合成数据引擎的经验，扩散模型做 mask-guided 局部编辑的一次通过率通常在 30–60% 区间（失败模式包括边界穿帮、目标形变、时序闪烁），叠加五维度严格验证（须同时通过全部五项）后，估计阶段三保留率大致在 20–50%；即产出 79K 训练对可能需要合成 15–40 万条候选。这纯属外推，论文无任何数据支撑。

### [2026 其他音视频联合生成](../models/JAVG_2026_misc.md) ⚠️

本批工作中仅两项给出可算的保留率，其余全为空白：
【OmniCustom —— 唯一可精确计算的漏斗】
- 输入：SpeakerVid-5M，「more than 5.2 million video clips」/「8,000 hours」
- 输出：OmniCustom-1M，「约 1 million single-person video clips」/「2,500 hours」
- 片段级保留率 ≈ 1.0M / 5.2M ≈ 19.2%
- 时长级保留率 = 2,500 / 8,000 = 31.25%
- 两个保留率不一致的含义值得注意：时长保留率显著高于片段保留率，说明被保留下来的片段平均更长（保留片段平均 2500h/1M ≈ 9.0 秒 vs 原始 8000h/5.2M ≈ 5.5 秒），这正是「剔除 <10 秒视频」这条规则的直接后果——该规则在片段数上砍得很狠，但砍掉的都是短片段。
- 三条规则各自的通过率未分开给出[不确定]。
【NAVA —— 只有漏斗两端，无中间层】
- 输入：「approximately 20M audio clips and 100M video clips」
- 输出：「around 15M clips for large-scale training」
- 若以视频侧计，保留率约 15M/100M = 15%；若合并计则更低。逐级（OCR/去重/质量/对齐）的输入输出量与通过率均未披露[不确定]。
- 二次收敛：15M → 160K 高质量 SFT 子集，SFT 保留率约 1.07%（即千分之十）——这个数字很有参考价值，说明「可训练」与「可做 SFT 的精品」之间存在两个数量级的落差。
【ALIVE】完全未给保留率或原始池规模，只能从阶段样本数反推数据被逐级收窄的趋势：11M（联合训练）→ 4.3M（balanced，均衡后仅剩 39%）→ 5M（SFT）→ 0.7M（1080p 高清子集，占 11M 的 6.4%）→ 0.8M（角色配对）。其中「11M → 4.3M balanced」这一步的 39% 保留率是数据均衡（domain 配比调整）造成的削减，是本批中除 OmniCustom 外唯一可推算的比例[属推断，不确定]。
【Baton / CCL / StreamChar】无任何漏斗信息[不确定]。CCL 的 4M 是最终使用量，原始候选池未知。
【ITS-JAVG】推理时「保留率」概念对应 Best-of-N 的选择率：JavisDiT 为 1/5 = 20%，MMDisCo 为 1/10 = 10%——巧合的是，这与工业数据管线常见的 10%–20% 保留率量级相当，侧面说明「过采样再择优」在训练侧与推理侧遵循相似的经济学。

### [音视频联合生成基线合集](../models/JavisDiT_baselines.md) ⚠️

本合集中只有 JavisDiT++ 给出了可计算的漏斗保留率，其余全部缺失：
【JavisDiT++（唯一有定量漏斗的工作）】
- 视频侧总保留率：TAVGBench 原始 110 万条三元组 → 过滤后 35.5 万条，整体保留率约 32.3%（355K/1.1M）。这一数字与业界公开的同类漏斗（如 Apollo 的 27%）处于同一量级，具有横向可比价值。
- 保留数据的下游分配：33 万条用于音视频 SFT（占保留量 93%）、2.5 万条用于 AV-DPO（占 7%），两者严格不重叠。
- 分级保留率缺失：论文只给总数，未拆分「FunASR 语音剔除」「美学评分」「光流评分」「OCR 评分」各自淘汰了多少条，也未给出各项的具体阈值[不确定]。考虑到 TAVGBench 底层是 YouTube 视频、含语音比例很高，可推测 FunASR 这一步是最大的淘汰环节，但无数据支撑。
- 音频侧保留率为 100%（明确「不采用任何数据过滤策略」）。
- DPO 环节的内部比例：最终约 2.5 万条偏好对中，约 30% 的 winning 样本来自模型生成（而非真值），作者据此判断「基线模型本身已具备相当强的生成能力」。
【JavisBench 评测集漏斗】约 3 万条候选 → 前置质量过滤 + Qwen 自动打标归类 + 后置多样性过滤 + 人工法律伦理审核 → 10,140 条，保留率约 33.8%（10140/30000），与训练侧漏斗保留率巧合地接近。各级淘汰量未拆分[不确定]。
【Harmony】语音侧从 Emilia + OpenHumanVid + SpeakerVid 的汇总池经一致性打分筛出 200 万条，但汇总池原始规模未给出，无法计算保留率[不确定]。
【MM-Diffusion / AV-DiT / UniAVGen】完全无漏斗定量数据[不确定]。MM-Diffusion 的 928 条源视频 → 1,000 条片段是切分而非过滤，不构成保留率。

### [可灵 3.0 / 可灵视频 3.0 Omni](../models/Kling_30_Omni.md) ⚠️

[不确定] 可灵3.0 Omni 未公布任何一级过滤的输入/输出量与最终保留率。同团队 Koala-36M 公开了完整的量化漏斗，可作数量级参考：切分+打标后 Koala-all 4800万条 → 人工阈值过滤后 Koala-37M 3700万条（保留约77%）→ VTSS（阈值2.5）过滤后 Koala-36M 3600万条（相对再保留约97%，端到端约75%）。Kling-Foley 侧未公布保留率，仅给出最终规模（5500万clip/12.2万小时）。

### [LTX-2](../models/LTX-2.md) ⚠️

完全未披露。LTX-2 与 LTX-Video 两篇论文均未给出任何一级过滤的输入/输出样本量、淘汰率或最终保留率，无法与 Apollo 27% 之类的定量漏斗对比。仅知 LTX-2 训练集是 LTX-Video 数据集的「一个子集」，但子集占母集的比例同样未公布。[不确定]

### [LongCat-Video](../models/LongCat-Video.md) ⚠️

[不确定]。报告完全未给出任何一级过滤的输入/输出量或保留率，也没有最终保留率的整体数字。这是该技术报告在数据侧披露最薄弱的部分之一——它详细列举了用了哪些过滤器，但没有给出任何一个阈值数值与任何一级的通过率，因此无法与 Apollo 27% 之类的定量漏斗做横向对比。

### [MOVA](../models/MOVA.md)

论文以 Table 1《Retention Ratio of Total Dataset Duration》给出了按总时长计的逐级保留率（相对原始视频），这是本次调研中少见的公开定量漏斗：
- Raw（原始视频）：100%
- Stage 1（预处理后，语音 + 非语音片段）：84.57%
- Stage 1（仅保留语音片段）：58.75%
- Stage 2（质量与对齐过滤后）：26.39%
即最终整体保留率 26.39%，与 Apollo 的 27% 处于同一量级。拆解看，两次主要损耗是：(1) “只要语音”这一条从 84.57% → 58.75%，砍掉 25.8 个百分点，是最大的单一淘汰源（另有一处片段数口径的表述：语音片段占全部预处理片段的 69.47%）；(2) 三维质量与对齐过滤从 58.75% → 26.39%，再砍掉 32.4 个百分点，相对留存率约 44.9%。
此外训练阶段还有更严的二次收窄（相对 Phase 1 的 61,500 小时）：Phase 2 降至约 37,600 小时 / 16.8M clips（约 61%），Phase 3 再降至约 11,000 小时（约为 Phase 1 的 18%）。若把训练课程也算作漏斗的延伸，从原始素材到最终 720p 高质量子集的等效保留率远低于 26.39%。
Phase 2 三道子过滤各自的产出量也有公开数字：OCR 无字幕子集 ~9.5M clips、LSE 唇音高质量子集 ~2.5M clips、DOVER 技术分 >0.15 子集 ~2.4M clips，合并后的 Phase 2 数据集为 16.8M clips。

### [合并条目：① Mochi 1](../models/Mochi_MAGI_Motif.md) ⚠️

① Mochi 1：无。[不确定]
② MAGI-1：无任何定量保留率。仅有定性表述：初次过滤「effectively discards most of the low-quality data（有效剔除了大部分低质数据）」、到 MLLM 过滤时「the remaining data size has been significantly reduced（剩余数据量已显著缩小）」，以及 Table 5 中「数据量随阶段递减」的方向性说明。既无逐级输入/输出量，也无端到端保留率，无法定位主要淘汰级。[不确定]
③ Motif-Video 2B：给了漏斗的可视化但没给数字。Fig.7 是一张 Sankey 图，论文原文称其「visualizes how flows contract from the raw pool toward the curated training and SFT corpora（可视化了数据流如何从原始池向精选训练集与 SFT 集收缩）」，但正文未标注任何百分比或绝对量。唯一可锚定的端点是最终训练集「fewer than 10M clips」，原始池规模未给，因此端到端保留率无法计算。这是 Motif 数据披露中少数的空白之一。[不确定]
综合看，本组三个模型都没有提供 Apollo（27%）或 Allegro（0.4%–26% 分档）那种逐级定量保留率，是横向对比时需要注意的共同短板。

### [Movie Gen](../models/Movie_Gen.md) ⚠️

论文表44给出了业界罕见的逐级定量漏斗（针对最严格的高分辨率集curation阈值，数字为相对原始池的剩余体量百分比）：
时长 4s≤duration≤120s → 100%
分辨率 width≥768 且 height≥768 → 25%（单步砍掉75%）
宽高比 width≥height → 7%（单步砍掉72%，最陡的一刀）
无文字（所有采样帧的 词检测分×词识别分 均<0.6）→ 1.94%（单步砍掉72%）
无黑边 → 1.87%
无场景切换（从1个场景中采1个12~16秒clip）→ 1.78%
美学（clip中间帧 LAION 美学分 ≥4.0）→ 1.57%
非慢动作（motion score>2.0，motion vector均值>0.5 且 <7）→ 1.32%
非抖动（每秒镜头数<0.85）→ 1.22%
无内容重复（embedding余弦相似度<0.99）→ 1.15%
概念重采样（每簇保留量 1/sqrt(簇大小)）→ 0.94%
论文原文明确指出：在这套阈值下「数据接受率低于1%」（less than 1%），即约百分之一的原始视频进入高分辨率训练集。音频侧未给出对应的定量漏斗[不确定]。

### [NVIDIA NeMo Curator](../models/NeMo_Curator.md) ⚠️

只有单级定量数字，没有完整逐级漏斗表。
【已公布的定量点】(1) 语义去重环节移除约 30% 的训练数据（remove approximately 30% of training data）；(2) 视觉质量过滤中，基于 DOVER 的失真评估模型剔除得分最低的 15%（bottom 15%）；(3) 切分阶段丢弃全部短于 2 秒的 clip（占比未公布）；(4) 总体从 2000 万小时原始视频得到约 1 亿个 2–60 秒 clip——若按平均 clip 时长 15 秒粗算约合 41 万小时，相对输入的时长保留率约 2%，但该推算不严谨（原始视频中大量内容在切分阶段即被丢弃，且论文未确认 clip 平均时长）。
【缺失】运动过滤、文字叠加过滤、视频类型 taxonomy 过滤各自的淘汰率未公布；各级的输入/输出样本量表格未提供；无法构成类似 Apollo 27% 那样的端到端保留率数字。[不确定]

### [OmniHuman 数据集 + OHBench](../models/OmniHuman.md) ⚠️

完全缺失，是本工作数据披露中最严重的空白：
【已知】终态 100 万条 / 1,800 小时 / 8 万身份。
【未知】原始采集量（爬取了多少条 YouTube 视频、多少小时）；四级过滤每一级的输入量与输出量；每一级的淘汰比例；跟踪与音视觉归属阶段的淘汰量；最后一致性校验的淘汰量；全流程整体保留率。分子已知而分母全无，无法计算类似 Apollo 27%、MOVA 26.39% 的保留率，也无法判断各级过滤的实际严格程度。
【定性推断（无论文依据）】结合流水线中若干道强约束可判断整体淘汰力度应属很强：(1) 高清及以上的分辨率硬门槛会淘汰大量老旧与低码率内容；(2) 必须含音轨且音轨须与画面中人物同源（SyncNet 归属 + ±3 帧偏移）会淘汰绝大多数带旁白、后期配音、纯 BGM 的 YouTube 内容——这是最强的一道；(3) 一致性校验要求 MLLM 标注的主体数与跟踪结果精确一致，对多人杂乱场景的通过率应当很低。综合看整体保留率大概率在个位数百分比量级，但这是推测而非结论。[不确定]
【对可复现性的影响】论文强调其贡献之一是「全自动 pipeline」，但既无阈值也无保留率，第三方按论文描述复跑该流水线，得到的数据集规模与质量分布将与原作无从对齐，pipeline 的可复现性实质上不成立。相比之下 UniVerse-1 给出了全部阈值、MOVA 给出了逐级保留率，OmniHuman 在这一点上落后。

### [Open-Sora 系列](../models/Open-Sora.md) ⚠️

【Open-Sora Plan v1.3 —— 全行业罕见的逐级定量披露】论文给出完整的七级保留率表（相对原始数据的累计保留率）：
  视频切分（ffmpeg，16s）→ 100%
  跳切检测（LPIPS，保留 32 ≤ 帧数 ≤ 512）→ 97%
  运动过滤（LPIPS，0.001 ≤ 分数 ≤ 0.3）→ 89%
  OCR 字幕裁剪（EasyOCR，边缘比例 0.20）→ 89%（该级只裁剪不丢弃，故保留率不变）
  美学过滤（LAION Aesthetic v2，≥ 4.75）→ 49%【单级淘汰最狠，砍掉约 40 个百分点】
  技术质量（DOVER technical score ≥ 0）→ 44%
  运动复检（LPIPS，0.001–0.3）→ 42%
  **最终累计保留率约 42%**。
另据 Open-Sora Plan v1.3.0 Report 的另一处口径，清洗后的 Panda70M **只保留了原始数据的 27%**（约 1900 万条 / 原约 7000 万条），这与 Apollo 报告的 27% 数量级高度一致。两个数字口径不同（42% 是论文漏斗表的累计口径，27% 是针对 Panda70M 全集的最终口径），建议引用时注明来源版本。
【Open-Sora 2.0】明确说明构建了「分层数据金字塔」以配合渐进式训练，但**未给出任何一级过滤的输入/输出量或保留率数字**，只能从训练阶段数据量（70M → 10M → 5M）间接看出金字塔的层级比例约为 14 : 2 : 1，即最高质量层仅占最底层的约 7%。
【Open-Sora 1.2】未给出逐级保留率；可推的粗略比例是原始 30M 片段（80k 小时）→ 最终高质量阶段 2M 片段（5k 小时），约 6.7%。[不确定]

### [Ovi](../models/Ovi.md) ⚠️

[不确定]。论文未给出任何一级过滤的输入/输出量、通过率或整体保留率，也没有 Apollo 式的 27% 类定量漏斗表。仅能定性判断筛选强度很高：SyncNet 条件（|offset|≤3 且 confidence>1.5）被作者自述为「strict criteria」（严格标准），叠加 >720×720 分辨率门槛、RAFT 静态剔除、美学评分剔除，实际保留率应显著低于常见的宽松 pipeline，但无数据支撑。Ovi 1.1 只披露最终数据集规模翻倍，未披露原始候选池规模。

### [Script-a-Video](../models/Script-a-Video.md) ⚠️

论文完全未给出任何漏斗定量数据：未披露原始采集视频总量（分子已知为 500K，分母完全缺失），未给出逐级过滤的输入/输出量，未给出各级或整体保留率，也未给出生成侧四套数据（400K/250K/870K/60K）各自的原始池规模与保留率。
无法计算类似 Apollo 27% 或 MOVA 26.39% 的整体保留率。
值得注意的一个「保留率」类比：MTSS 在标注层面存在一个隐性的信息保留决策——Event 流的严格音视频耦合原则会把大量音频信息「过滤」到 global_audio，Reference 流的叙事重要性筛选会把边缘实体「过滤」到 global scene description。但论文未量化这两个降级通道各自吸纳了多少比例的信息。[不确定]

### [Seedance 2.0 与 Seedance 1.5 pro](../models/Seedance_20_Seedance_15_pro.md) ⚠️

[不确定] 三代报告（1.0/1.5/2.0）均未给出任何一级过滤的输入/输出量或保留率数字，无法与 Apollo 27% 这类披露对标。这是 Seedance 系列数据披露的最大缺口。

### [SkyReels 系列](../models/SkyReels.md) ⚠️

定量漏斗披露不完整，只有两个可用比值：
(1) SkyCaptioner-V1 训练集：从「1000万条原始样本」精选出「约200万条概念均衡视频」，保留率约 20%——这是 SkyReels 系列唯一明确的「输入→输出」比值，但它是打标模型训练集的筛选率，而非生成模型全量训练数据的漏斗保留率；
(2) 概念均衡环节：后训练阶段按主体类别做概念均衡「导致数据量减少了50%」，即该级保留率约 50%。
此外可作参照的还有：运动识别打标器的训练数据为「9.3万条高置信度人工标注样本 + 1.6万条运动轴均衡的合成数据」，在1.5万段视频测试集上单一运动类型预测准确率89%。
两代论文均未给出逐级过滤的输入/输出样本量表，也未给出从 O(100M) 原始数据到最终训练集的端到端保留率，无法与 Apollo 27% 一类的完整漏斗直接对比。SkyReels-V4 完全未披露任何过滤保留率。[不确定]

### [Sora 2](../models/Sora_2.md) ⚠️

完全未披露。没有任何一级过滤的输入/输出量、保留率或淘汰率数字。无法与 Apollo 27% 之类的公开定量漏斗做对比。[不确定]

### [SpeakerVid-5M](../models/SpeakerVid-5M.md) ⚠️

论文未提供逐级保留率表格，只能从首尾两个口径做端到端推算：
【入口】153K 条视频 / 64,386 小时原始素材。
【出口】超过 5.2M clips / 8,743 小时（含单人分支 8.7K 小时 + 对话分支 1.8K 小时，两者存在样本重叠，总量以 8,743 小时计）。
【端到端时长保留率】8,743 / 64,386 ≈ 13.6%。这一保留率显著低于 MOVA 的 26.39% 与 Apollo 的 27%，但口径不完全可比——SpeakerVid-5M 的损耗大头并非质量过滤，而是结构性损耗：只保留两个主说话人（其余人物丢弃）、只保留有明确说话/倾听行为的时段、以及 YOLO 单人裁剪后的画面区域损失，均会大量削减有效时长。
【HQ 子集保留率】1,368 / 8,743 ≈ 15.6%（相对最终数据集）；1,368 / 64,386 ≈ 2.1%（相对原始素材）。即从原始素材到可用于 SFT 的高质量对话数据，等效保留率约五十分之一。
【未披露】各单步（场景切分、diarization、SyncNet 绑定、ArcFace 校正、以及五维质量过滤中每一维）的输入/输出量与逐级淘汰率均未给出，Figure 3 中虽有 DOVER 分数与 SyncNet 置信度的分布直方图，但无法据此反推淘汰比例。这是本数据集披露体系中相对薄弱的一环。[不确定]

### [Step-Video-T2V](../models/Step-Video-T2V.md) ⚠️

定量披露不完整，只能部分推算：
【已知绝对量】原始池 → 2B 视频-文本对（pipeline 产出的总量）→ Step-2 低分辨率档可用约 1B 片段（实际 seen 644M）→ 高分辨率档 seen 27.3M → SFT 30M → DPO/蒸馏约 95k。
【可推算的保留率】2B → 1B（192P 档）约 50%；2B → 30M（SFT 高质量档）约 1.5%，即端到端从 pipeline 产出到 SFT 的保留率约 1.5%（此换算为本调研推算，报告未直接给出该比值）。
【缺失部分】(1) 原始视频池（切分前）的绝对规模未公布，因此「原片 → 2B 对」这一段的保留率无从计算；(2) Figure 11 的分层漏斗图只以条形长度示意各级过滤的去留比例，未标注任何绝对数值或百分比；(3) 6 个预训练子集各自的样本量未列出。
因此本条目的漏斗定量完整度低于 HunyuanVideo 1.5（后者给出了 >1000万小时 → 8亿 → 2亿 → 1亿 → 100万 的完整链条），可比的只有 SFT 端的 1.5% 量级。[不确定]

### [UniTalking](../models/UniTalking.md) ⚠️

论文未给出任何定量漏斗数据：未披露原始语料的总量（条数或小时数）、未给出三级过滤各自的输入/输出量、未给出各级或整体保留率。仅已知最终产出为 230 万条对齐样本，分子已知而分母完全缺失，无法计算类似 Apollo 27% 或 MOVA 26.39% 的保留率。
可做的间接量级推断（仅供参考，无论文依据）：若 230 万条主要来自 OpenHumanVid 的 1,320 万条高质量片段，则该部分保留率上限约为 17.4%；考虑到 UniTalking 还额外叠加了「必须含语音 + 声源在画面内 + 唇音同步」三重强约束（OpenHumanVid 虽自带语音音轨但并未按说话人场景筛选），实际从 OpenHumanVid 侧的保留率应显著低于此值，剩余部分由华为内部数据补足。但两个来源的具体配比未知，故此推算不可作为结论。[不确定]
定性判断：本流水线的淘汰力度应属较强量级——三级串联中的每一级都是硬性布尔淘汰（无软加权、无分层保留），且跨模态级的 LightASD + LipSync 双重条件对普通网络视频而言通过率天然很低（大量视频存在配乐、旁白、非同期声或人脸过小）。

### [UniVerse-1](../models/UniVerse-1.md) ⚠️

论文未给出任何定量漏斗数据：未披露原始采集视频的总量（小时数或条数），未给出逐级过滤的输入/输出量，未给出各级或整体保留率。仅能确知最终产出为 7,685 小时（语音 1,187 + 通用 3,074 + 公开数据集 3,422）。由于分子已知而分母完全缺失，无法计算类似 Apollo 27% 或 MOVA 26.39% 的整体保留率。
可定性推断的是保留率应当相当低：分辨率≥1080p、码率比≥600、DOVER≥0.6 三条硬门槛叠加，再加 PySceneDetect 切分后 <5 秒剔除，以及语音子集额外的 SyncNet conf>2.0，累积淘汰力度不小；语音子集仅 1,187 小时的偏小规模，也侧面反映了「有音轨 + 有人脸 + 唇同步达标」这一串联条件的严苛。但这些均属推断，无论文数据支撑。[不确定]

### [Unison](../models/Unison.md) ⚠️

论文未给出任何定量漏斗数据：未披露聚合后的原始候选量（分母），未给出逐级过滤的输入/输出量，未给出各级或整体保留率。仅已知最终产出为约 200 万 clips / 3,000+ 小时（音视频）与 5,000 万段 / 130,000+ 小时（音频）。
【可作粗略参照的上游规模——仅供量级感知，非论文数据】五个音视频源数据集的公开原始规模大致为：OpenHumanVid 千万级 clips、CelebV-Text 约 7 万条、VFHQ 约 1.6 万条、HDTF 约十余小时、VGGSound 约 20 万条 / 550 小时。若按此粗算，五者原始总量的数量级在千万条以上，而 Unison 最终保留约 200 万条，整体保留率可能在 10%~20% 量级——但这一估算极不可靠：各数据集的实际使用比例未知，OpenHumanVid 一家的体量就足以左右整个分母，且不排除 pipeline 对长片段做了二次切分从而增加条数。此推算不应作为结论引用。[不确定]
【无法计算的原因】分子已知而分母完全缺失，且论文连「聚合了这些数据集的全部还是子集」都未说明。与 Apollo（27%）、MOVA（26.39%）等给出明确整体保留率的工作相比，Unison 在这一维度上无任何可比数据。

### [Veo 3 / Veo 3.1](../models/Veo_3_Veo_31.md) ⚠️

[不确定] 完全未披露。官方未给出任何一级过滤的输入/输出数据量、保留率或最终留存比例，无法与 Apollo 27% 之类的公开量化漏斗做对比。这是 Veo 3 数据披露中最彻底的空白。

### [Vidu S1](../models/Vidu_S1.md) ⚠️

[不确定]。论文虽给出完整的六级漏斗结构图（Figure 2），但未提供任何一级的输入/输出数据量或保留率数字，也没有最终保留率。这是该报告数据披露的主要缺口之一（对比 Apollo 类工作给出 27% 保留率的做法）。

### [Wan 2.5 / 2.6 / 2.7](../models/Wan.md) ⚠️

定量披露仅有两处，无法构成完整漏斗账本：
1) Wan 2.1 基础维度阶段：「we successfully eliminated approximately 50% of the initial dataset」——第一级并联粗筛淘汰约50%，保留50%进入语义精筛。这是 Wan 系唯一明确的级间保留率。
2) Wan 2.1 后训练图像精选：取专家模型预测分数的 top 20%。
其余各级（聚类配额具体取多少、视觉质量分阈值、六档运动各自保留比例、V2A 音频过滤前后的量比、S2V 各级淘汰率）均无输入/输出量与保留率。V2A 侧只知结果为 O(1) 千小时，母集规模为「数十亿视频」，若粗略折算保留率远低于1%，但报告未给出可比口径，不能作为漏斗数字使用。整体上无法与 Apollo 27% 之类的定量漏斗对齐。[不确定]

### [音视频生成评测基准合集](../models/av_benchmarks.md) ⚠️

五个基准均未公布逐级过滤的输入/输出量与保留率数字[不确定]。可间接推算的仅有：
【AV-SyncBench】最终保留 3,269 条视频，但网络采集的原始池规模未披露，故 Gemini 3 Flash 初筛与 5 人复核两级的保留率无法计算[不确定]。
【PhyAVBench】11,605 条视频对应 337 组配对提示词，平均每组约 17 条 ground-truth 视频；论文设计目标为每组 N≥20 条，实际均值 17 说明质控阶段存在明显淘汰（粗估保留率约 85%，属推算而非论文披露[不确定]）。
【AVBench】30K 真实片段 → 300K 训练对属扩增而非过滤，无保留率概念；470 条评测提示词的原始候选池规模未披露[不确定]。
【VABench】1,299 条最终样本对应的候选生成量未披露[不确定]。
相较于训练侧数据集（如 Apollo 公布 27% 端到端保留率、MOVA 公布逐级保留率表），评测基准论文普遍不披露漏斗定量信息，这是本类文献的通病。

### [视频 Caption 模型生态](../models/caption_models.md) ⚠️

[不确定] captioner 生态整体缺乏定量保留率披露，仅有零星数字：
· SkyCaptioner-V1：训练数据从 1000 万条精选出约 200 万条概念均衡视频，保留率 20%。这是本条目唯一明确的 captioner 训练数据保留率。
· AVoCaDO：GPT-4.1 对 synthesis completeness 打 1–5 分、只保留 ≥4 分，但未披露该步的通过率。
· Tarsier2：对 100 万条公开数据集视频做 recaption，最终发布 585K（Tarsier2-Recap-585K），若 585K 全部来自该 1M 则保留率约 58.5%，但论文未确认二者是同一批（也可能是主动抽样发布而非过滤）。[不确定]
· Panda-70M 的贪心集合覆盖给出的是「覆盖率」而非保留率：单个最好模型覆盖 30.8%、8 个覆盖 76.8%、31 个全上 84.7%；同时报告人类之间的一致率仅 44.9%，说明「最佳 caption」本身高度主观。
· AVSCap-130K、AVoCaDO-SFT-107K 均未给出「原始候选量 → 最终保留量」的逐级数字。
· 生成侧最完整的端到端保留率来自 Apollo/Klear（27%），但那是整条数据漏斗而非打标环节。

### [几何/结构化标注数据集合集](../models/geometric_datasets.md) ⚠️

SpatialVID 给出最清晰的量化漏斗：21,789小时原始 YouTube 视频（33,443条）→ 最终 7,089 小时动态内容（271万clip），时长维度保留率约 32.5%；高质量子集 SpatialVID-HQ 为 0.37M/2.71M clip，即在已过滤库上再保留约 13.7%（相对原始素材端到端约 4.5%）。关键量化发现：Panda-70M 中超过 80% 的视频因运动不足而无法被 MegaSaM 成功重建，构成运动过滤的主要淘汰源。SceneScribe-1M 各级输入输出量未逐级披露，仅知从 HD-VILA-100M/Panda-70M/Koala-36M/Pexels 的亿级源池收敛至100万clip、156.7M帧、4000+小时，论文表述“因运动多样性要求初筛大幅缩减源池”，逐级保留率数值缺失[不确定]；Action100M 的量化在去重侧：识别出 7.58M 个重复组、1.418亿条重复实例并去除，源120万视频中72%有可用ASR，最终产出1.47亿片段；WildWorld 为引擎直采，无过滤漏斗概念，仅提及过滤后得到数千条clip（确切数量未给出）[不确定]

### [视频生成后训练数据策略](../models/post_training_data.md) ⚠️

锚论文无任何漏斗定量数据 [不确定]。横向可对标的「后训练保留率」（SFT 精选集 ÷ 预训练池）如下，这是本专题最有价值的一组可比数字：
· NAVA：160K ÷ 15M ≈ 1.07%（本次调研中最严，「千条里取十条」）；
· Allegro：2M ÷ 500M = 0.4%（若以原始片段计则更严，但其 2M 是最严阈值组合的直接产出）；
· Goku I2V：4.5M ÷ 36M = 12.5%（且筛选标准侧重领域多样性而非质量分）；
· CogVideoX：top 20%（明确以百分位而非阈值定义）；
· Open-Sora Plan T2V 阶段三：过滤后 Panda70M 约 19M / 27% 保留；
· NVIDIA Cosmos WFM：从约 10^8 预训练 clip 中切出约 10^7 用于微调（约 10%，筛选标准未公布）；
· SkyReels-V4：500 万 → 100 万人工精选（第二段 SFT 保留率 20%）。
【经验区间】SFT 精选集相对预训练池的保留率集中在 0.4%–20%，中位数约 5%–10%；越是追求美学与「影视感」的模型保留率越低（Allegro、NAVA），越是追求领域覆盖或多任务能力的保留率越高（Goku、Open-Sora Plan、Cosmos）。
【偏好数据侧的「保留率」形态不同】其规模由标注预算而非过滤阈值决定，SkyReels-V2 的 3 万人工样本对、Step-Video 的未公开条数、HunyuanVideo 1.5 的 O(10K) 均是「标注能力上限」的体现。JavisDiT++ 给出的一个独特指标：最终偏好数据中约 30% 的 winning 样本来自模型生成而非真值，作者据此判断「基线模型本身已具备相当强的生成能力」——这个比例可作为判断模型是否已到达「可用自生成数据做偏好优化」阶段的经验信号。

### [主流视频预训练数据集合并调研：Panda-70M、InternVid、Koala…](../models/pretraining_datasets.md) ⚠️

**定量披露程度差异巨大，只有三家给出了可用的漏斗数字**：
- **UltraVideo（披露最完整）**：5,000条原始视频 → 切镜与帧数过滤后 **62K short + 25K long** → 统计过滤后 **46K short + 19K long**（保留率约**74% / 76%**）→ 模型净化后 **42K short + 17K long**（保留率约**91% / 89%**）。**从切分后池子算起，short 整体保留率约68%**。但每一项检查各自剔除多少条未报告。
- **Koala-36M（给出了关键的两点）**：重切分+打标后未过滤池 **48M clips**（该数字出自附录D表8的分母）→ 人工多阈值过滤基线 **37M**（77.1%）→ **VTSS>2.5 最终 36M（75.0%）**。即质量过滤这一级保留率为**75%**，且 VTSS 比人工多阈值**略更严格**（36M vs 37M）却保住了更多高质样本。**从103M源视频到48M clips 这一段的切分产出率未报告。**
- **LVD-2M（只有首尾）**：**220M clips 输入 → 2M 输出，整体保留率约 0.91%**——七者中最低。**逐级保留率完全未报告**（论文图2只是示意图，无计数）。唯一的中间统计是对 InternVid 的诊断：仅15%的clip超过10秒，而这些长视频中约52.5%含镜头切换。
- **MiraData（有分档但无阈值）**：未过滤池 **788K** → 330K（41.9%）→ 93K（11.8%）→ 42K（5.3%）→ 9K（1.1%）。另有一个极具说服力的单点数字：**HD-VILA-100M 约1亿clips输入，最终仅19.5万条幸存（约0.2%）**，作者以此论证自家人工挑选的YouTube频道质量远高于通用爬取。**但四档之间的具体阈值论文称在补充材料中，而补充材料实际不存在**（我核查了 arXiv v1 附录、NeurIPS camera-ready 与补充ZIP链接，均无阈值），这是一处真实的可复现性缺口。
- **OpenVid-1M**：只有定性描述（Panda-50M候选池 → 过滤得 Ours-0.4M → 切分扩展为 Ours-0.6M → 合并其余三源达到约1M），**各级绝对数量与整体保留率未披露**。[不确定]
- **Panda-70M**：380万源视频 → 7,081.7万clips，这是**扩张比（约18.7倍）而非保留率**——它几乎不做质量过滤。唯一的过滤性数字是发布口径的 70,723,513（较论文少约9.4万条，系有害内容过滤，作者未明说）与 **89.6% 的样本 matching_score>0.43**。另有2024年10月追加的 desirability 分布（全量占比）：desirable **80.5%**、低期望分5.28%、静止前景6.82%、微小相机运动1.20%、画中画5.03%、屏幕录制1.13%——**即按其自评仍有约19.5%的样本是不理想的**。
- **InternVid**：**完全未披露任何漏斗数字**。[不确定]
**可引用的经验值**：跨数据集看，「从通用网络视频池筛到可用训练集」的保留率量级为——不做质量过滤约100%（Panda）、做常规质量过滤约70–75%（Koala、UltraVideo统计层）、做严格质量+长镜头筛选约1%（LVD-2M 0.91%、MiraData对HD-VILA的0.2%）。**「长镜头+高动态+无切换」这一组约束的代价是三个数量级的数据损耗**，这是本次调研最强的定量结论之一。

## 镜头切分方法（PySceneDetect/自研模型/shot-aware splitting）

`shot_segmentation` · 详细程度: brief

### [Allegro](../models/Allegro.md)

使用开源工具 PySceneDetect 做镜头/场景切分，将原始长视频拆为单场景（single-scene）clip，切分后仅保留 2–16 秒的片段。
一个值得注意的工程细节：切分后丢弃每个 clip 的首尾各 10 帧，用于消除场景检测的假阳性（转场淡入淡出、切换残帧）造成的跨镜头污染。
未使用自研 shot-aware 切分模型，也未做转场类型分类或多镜头重组。

### [Apollo](../models/Apollo.md) ⚠️

确认存在场景切分环节，目的明确但方法未公开：论文仅称「We then apply scene splitting to ensure each sample contains only one scene」（做场景切分以确保每条样本只含一个场景），置于视频质量过滤之后。未说明使用 PySceneDetect、TransNetV2 还是自研模型，未给出切点检测阈值，未说明切分后如何采样窗口，也未说明是否像 MOVA 那样把语音边界（VAD）与场景切点联合起来做 shot-aware + speech-aware 的双重感知切分。考虑到 Apollo 数据以语音/歌唱为主体，若切分不感知语音边界会导致语句被截断，但论文对此未作说明。[不确定]

### [CineDance / CineDance-1M](../models/CineDance.md)

【工具】TransNetV2，业界成熟的深度学习镜头边界检测模型（相较 PySceneDetect 这类基于像素直方图的传统方法，对渐变转场、溶解、划像更鲁棒），从 44,579 条清洗后视频中切出 25,899,474 个原子镜头。
【关键差异化】切分只是中间步骤而非终点。传统数据集把「一个镜头 = 一个样本」，本工作在镜头之上再做一层「自底向上的叙事分组」，用 Qwen3.5-27B 依据四条电影理论规则（多角度、平行剪辑、因果动作/省略、蒙太奇）把语义连贯的相邻镜头重组为 1,201,912 条叙事序列，序列才是最终样本。
【抗幻觉设计】不让 LLM 直接输出时间戳，而是先把镜头编号索引化，让 LLM 输出镜头编号的分组结果（bottom-up shot indexing），大幅降低时间戳幻觉。
【长视频推理】上下文感知滑动窗口，窗口约 3 分钟，且窗口切分点强制对齐到镜头边界，避免把一个镜头劈成两半。
【抗碎片化】人工参考集实测叙事完整性最短需 18.4 秒，故设 20 秒软阈值；最终仅 3.1% 序列短于 20 秒。
【解析质量】Qwen3.5-27B + 自底向上策略在解析任务上取得 F1 = 88.4%（Tab.4）。

### [CogVideoX](../models/CogVideoX.md) ⚠️

论文未提及使用 PySceneDetect、FFmpeg 场景检测或任何显式的镜头边界检测/切分工具 [切分工具不确定]，这是其 pipeline 描述中的明显空白。
实际达成单镜头的机制是「判别式剔除」而非「切分」：训练一个专门的分类器识别 Lack of Motion Connectivity（论文正文与附录表 14 中该分类器被命名为 Classifier-Static，测试准确率 0.92），其定义为「转场处缺乏连贯运动的视频片段，常见于人工拼接的视频或由静态图片剪辑而成的视频」；再叠加 Editing 分类器（准确率 0.91）剔除含明显剪辑与特效的素材。两者共同保证进入训练集的 35M 片段是单镜头且运动连续的。
换言之，CogVideoX 选择了「宁可整条丢弃也不做精细切分」的策略，代价是数据利用率较低，收益是实现简单且不引入切分器的误差。

### [Cosmos-Predict2.5](../models/Cosmos-Predict25.md) ⚠️

第一阶段即为「shot-aware video splitting」（镜头感知切分）：用高精度镜头边界检测模型（high-accuracy boundary detection models，复数，暗示多模型组合）将长视频切分为镜头级片段，核心质量要求是「ensuring that raw shot transitions are excluded」——切出的片段中不能残留原始转场帧，避免训练样本内部出现画面突变。论文未点名具体模型（未提 PySceneDetect / TransNetV2 等），但 NVIDIA 独立开源的 Cosmos Curator 中对应实现同时提供基于像素阈值的快速切分与基于 TransNetV2 的神经网络切分两种模式，可作为工程旁证。
切分之后紧接 GPU 转码与裁剪（去黑边与空间 padding），并以 5 秒为下限丢弃过短片段，上限 60 秒。
转场质量的兜底在过滤阶段：semantic artifacts filter（类 VTSS）专门剔除 poor transitions（劣质转场）与 video-in-video（画中画），即对切分漏检做二次补救。
[不确定：镜头边界检测模型的具体型号]

### [Data-Juicer 2.0](../models/Data-Juicer.md)

镜头切分是 DJ 视频算子中支持较完整的环节，提供三种策略供选择：
  · video_split_by_scene_mapper —— 「基于检测到的场景变化把视频切成场景 clip」，即标准的 shot boundary detection 切分，底层为 scenedetect（PySceneDetect）库，支持其 ContentDetector / ThresholdDetector / AdaptiveDetector 等检测器与阈值配置。这是与业界主流（Apollo、Movie Gen、Open-Sora 等普遍使用 PySceneDetect）完全一致的技术路线。
  · video_split_by_duration_mapper —— 按固定时长机械切分，不感知镜头边界，适合已知素材为单镜头长视频或对切点不敏感的场景。
  · video_split_by_key_frame_mapper —— 按视频编码的关键帧（I帧）切分。优点是切点与编码结构对齐、无需重编码、速度极快；缺点是 I帧位置由编码器决定，未必对应真实镜头边界。
【辅助算子】video_clip_reassembly_mapper 用于把在重叠 clip 上分别计算的结果（如手部动作轨迹）重新拼接回长视频时间轴，服务于具身智能场景的长时序标注。
【工程层面】v1.4.6 起支持视频字节流 I/O，切分过程可在内存中完成而不必反复落盘，对大规模切分的 I/O 开销有实质改善。
【实际使用情况】官方 T2V 案例未启用切分算子（源数据集已预切分）。因此 DJ 的切分能力虽然齐备，但缺乏在超大规模长视频语料上的公开实战数据（如切分吞吐、平均每小时视频切出多少 clip）。

### [Foley-Omni](../models/Foley-Omni.md) ⚠️

[不确定] 论文完全未提及镜头切分方法。未见 PySceneDetect、TransNetV2、自研 shot boundary detection 或任何 shot-aware splitting 的描述。
从数据构成推断，切分需求本身较弱：训练数据以公开数据集为主，而 GRID、LRS2、VGGSound、AudioCaps 等均已是预先切好的定长短片段（VGGSound 固定10秒、GRID 约3秒），无需再切分；仅 internal 音视频语料与 SpeakerVid/TalkVid 可能涉及从长视频切片，但论文未交代其处理方式。motion score 上界 3.2 可能间接起到剔除含硬切镜头片段的作用（镜头切换会造成光流/运动分数异常尖峰），但这只是副作用而非显式的镜头切分设计。

### [Goku](../models/Goku.md)

两级镜头切分，属于当时较精细的做法：
(1) 第一级——PySceneDetect 做初始镜头边界检测（shot boundary detection），基于像素/直方图差异快速定位转场。
(2) 第二级——DINOv2 语义特征校验：对切出的片段按每秒 1 帧采样，提取 DINOv2 视觉特征，计算相邻采样帧的余弦相似度，要求整段维持高相似度才判为「同一镜头且视觉连贯」。阈值随分辨率收紧：480×864 档要求 ≥0.85，720×1280 档要求 ≥0.90。该步骤的作用是补 PySceneDetect 的漏检（如渐变转场、同场景大幅内容变化），属于「shot-aware splitting + 语义一致性双保险」。
(3) 长度约束——单片段最长 10 秒，超长镜头强制截断。
未使用自研端到端切分模型，也未提及 TransNetV2 等替代方案。

### [Hailuo / MiniMax Video](../models/Hailuo.md) ⚠️

[不确定]（完全无披露）。未说明是否使用 PySceneDetect、TransNetV2 或自研镜头边界检测模型，也未说明是否做 shot-aware splitting。仅能从「输出为 6~10 秒单镜头片段」推断训练数据必然经过某种镜头切分，但方法完全未知。

### [HunyuanVideo-Foley](../models/HunyuanVideo-Foley.md) ⚠️

【方法】使用「场景检测算法」（scene detection algorithms）对原始长视频做镜头边界检测并切分，随后把切分结果规整为 8 秒定长块。论文用的是复数泛称，未点名具体工具——未说明是 PySceneDetect、TransNetV2、AutoShot 还是自研方案，也未给出检测器类型（内容检测/自适应检测）与阈值参数。[不确定：具体工具与参数]
【切分的目的与 V2A 任务的特殊关联】在视频生成模型中，镜头切分的目的是保证视觉时序连贯（避免训练样本中出现画面跳变）。在 V2A 任务中，镜头切分还有一层更关键的音频侧意义：镜头切换处往往伴随声学场景的突变（室内→室外、近景→远景的混响变化），跨镜头的片段会让模型面对「同一段音频对应两个不相关声学环境」的矛盾监督信号。因此对 V2A 而言，镜头切分实质上是一种声学场景一致性的保证手段，其重要性不亚于视觉连贯性。论文未从这个角度阐述，但设计上是自洽的。
【切分与定长的关系】先按内容边界切、再按固定长度切，两步分离。这与 UniVerse-1（切分后剔除短片段、保留变长）和 MOVA（VAD 感知的自适应窗口）都不同——本工作不做任何自适应，纯粹的机械定长化，简单但可能在事件边界处产生截断。
【切分不与音频事件边界联合决策】完全由视觉场景切点驱动，不参考音频侧的事件起止（onset/offset）。这意味着一次完整的声学事件可能被 8 秒边界从中间切断，模型会见到「有起音无衰减」或「无起音只有尾音」的残缺样本。对需要精确建模声学包络的 Foley 任务，这是一处潜在的数据缺陷，论文未讨论。

### [HunyuanVideo](../models/HunyuanVideo.md)

【原版】PySceneDetect 做主切分（切为单镜头 clip）；Transnet v2 与 PySceneDetect 双路提供场景边界信息用于交叉验证；切分后用 OpenCV Laplacian 算子在片段内挑选清晰帧作为 clip 起始帧，避免以转场模糊帧开头。
【1.5】PySceneDetect + 自研算子（custom operators）联合检测场景边界，统一切为 2–10 秒 clip；关键增量是在切分之后再接一个专门训练的「转场分类器（transition classifier）」做二次清洗，剔除仍含渐变/叠化等转场特效的片段——说明团队认为纯阈值型 shot detection 会漏检软转场，需要模型级补刀。

### [InstructAV2AV](../models/InstructAV2AV.md) ⚠️

使用 PySceneDetect（Castellano, 2024）作为镜头切分工具，置于整条pipeline的第一步，把原始长视频切为单镜头片段（single-shot clips）。
【为何前置】镜头切分在本pipeline中不是可选的清洗步骤而是硬性前提：后续的 CoTracker3 点跟踪、Grounded-SAM-2 mask 传播、TalkNet 说话人定位、mask-guided 视频合成全部依赖片段内视野连续，任何镜头切换都会同时破坏这四个环节。相比之下，文生视频模型的镜头切分主要是为了避免模型学到突兀转场，优先级低得多。
[不确定] 未披露具体配置：使用的检测器类型（ContentDetector / AdaptiveDetector / ThresholdDetector）、阈值参数、最小镜头长度设定均未说明。也未提及是否叠加了 TransNetV2 等基于学习的镜头边界检测模型做二次校验——PySceneDetect 基于像素统计，在渐变转场（fade/dissolve）与快速运镜上误检率较高，而来源中的影视素材恰恰富含此类转场，这是潜在的质量隐患，论文未讨论。
【切分后的定长处理】单镜头片段进一步取 5 秒窗口，具体取窗规则未说明（见 duration_distribution）。

### [2026 其他音视频联合生成](../models/JAVG_2026_misc.md) ⚠️

披露极少，是本批工作的共同薄弱环节：
【NAVA —— 唯一提及切分工程】「raw videos are first segmented at scale with a Hadoop-based pipeline」——采用基于 Hadoop 的分布式管线做大规模切分。值得注意的是论文只说明了工程框架（Hadoop）而未点名切分算法或工具（未说明是 PySceneDetect、TransNetV2 还是自研镜头检测模型）[不确定]。这种「只说基础设施不说算法」的披露方式本身也说明切分被视为已解决的工程问题。切分后平均片段约 7 秒。
【ALIVE】数据管线六阶段中没有独立的 shot segmentation 环节，切分动作隐含在 character-driven pipeline 中：「Extracts N clips (3–10 seconds) from long videos (10–30 minutes)」——从 10–30 分钟长视频抽取 N 个 3–10 秒片段，但抽取依据（是否基于镜头边界、是否基于说话人活动、是否随机）未说明[不确定]。考虑到其身份锚点选取用的是「sync score 最高的 1.5 秒子片段」，抽取很可能是基于同步分而非镜头边界驱动的[推断，不确定]。
【OmniCustom】不做切分——直接使用上游 SpeakerVid-5M 已切好的片段，自身只做「时长 <10 秒剔除」的筛选与「前 4 秒 / 后 5 秒」的定点裁切。这是一种「切分外包给上游数据集」的典型做法。
【StreamChar】同样不做切分，完全复用 SpeakerVid-5M / TalkVid / OpenHumanVid 三个数据集的既有切分粒度，仅施加「≤20 秒」的长度约束。
【Baton / CCL】未提及切分方法[不确定]。二者均使用 OpenHumanVid 等已切分的公开数据集，加上自采/in-house 部分（Baton 的互联网视频、CCL 的短剧电影）——后者理论上必须做切分，但论文完全未描述[不确定]，这是 CCL 数据披露的明显缺口（影视与短剧是镜头切换最密集的内容类型，切分质量对其结果影响应该很大）。
【ITS-JAVG】不涉及。
【总体判断】2026 年这批工作中，shot segmentation 已高度「基础设施化」——要么外包给公开数据集，要么只提工程框架不提算法。这与 2024–2025 年论文普遍点名 PySceneDetect/TransNetV2 的风气形成对比，反映该环节已不再被视为技术贡献点。

### [音视频联合生成基线合集](../models/JavisDiT_baselines.md) ⚠️

本合集在镜头切分上普遍薄弱，是与工业级 pipeline 差距最明显的环节之一：
【MM-Diffusion】明确采用「机械等长切分」而非镜头感知切分：把 928 条源视频「分成 1,000 条不重叠的 10 秒片段」，未使用 PySceneDetect、TransNetV2 或任何场景检测工具。这意味着切出的片段可能跨越镜头边界，含转场画面——是早期数据处理的已知粗糙点。作者对 AIST++ 另做了 crop 处理（仓库中的 AIST++_crop），属于空间裁剪而非时序切分。
【AV-DiT】不做切分，直接从已切好的数据集片段中采样 16 帧。
【JavisDiT / JavisDiT++】依赖上游 TAVGBench 已完成的片段划分，自身 pipeline 中无镜头检测环节；data.md 中的视频处理步骤只有「剔除损坏视频、过滤少于 10 帧的样本、fps 归一到 16Hz」，无 scene detection[不确定：TAVGBench 上游是否做过镜头检测]。
【Harmony】3–10 秒的片段划分方式未说明是否基于镜头检测[不确定]；自采集的 200 万条环境音片段的切分方法同样未披露。
【UniAVGen】未描述任何切分方法[不确定]。
【影响】缺乏镜头感知切分意味着训练数据中可能混入含转场的片段，理论上会损害时序一致性；但由于所有工作的片段都极短（1.6–10 秒），跨镜头风险相对可控。这也解释了为何这批模型全部不具备多镜头生成能力。

### [可灵 3.0 / 可灵视频 3.0 Omni](../models/Kling_30_Omni.md) ⚠️

[不确定，方法论有同团队公开范式] Kling-Omni 报告仅说明会“检测并移除突兀场景变化与不连贯镜头转场”，未指明工具。同团队 Koala-36M 提出并开源了自研镜头切分方法 Color-Struct SVM（CSS）：以 BGR 直方图相关性度量颜色距离、以 Canny 亮度 SSIM 度量结构距离，用线性SVM分类器判定转场；再叠加时序平滑，假设视频变化服从高斯分布，超过3σ置信区间才判为真实转场，从而把“渐变转场”与“快速运动场景”区分开——明确宣称优于 PySceneDetect 等阈值法。可灵3.0 Omni 很可能沿用该自研切分器的迭代版本，但无官方确认。

### [LTX-2](../models/LTX-2.md) ⚠️

pipeline 的输入单元直接标注为「Input Shots」，说明上游已完成镜头切分，且全流程（估运动、去重、resize、美学评分、过滤）均以 shot 为原子单位，最终产物为「Final Shots」。但两篇论文均未说明镜头切分所用的具体方法——既未提及 PySceneDetect/TransNetV2 等开源工具，也未说明是否为自研 shot-aware splitting 模型，无阈值参数。[不确定]

### [LongCat-Video](../models/LongCat-Video.md)

采用双路联合的场景切分方案：开源工具 PySceneDetect 与内部自训练（in-house trained）的 TransNetV2 模型并用。TransNetV2 是专门的镜头边界检测神经网络，对渐变转场（fade/dissolve）的识别显著优于 PySceneDetect 的阈值法，团队还在自有数据上做了再训练。切分目标是得到「training-friendly clips while maintaining content consistency」——既适合训练又保持内容一致的片段。切分过程中同步用 FFMPEG 做黑边裁剪（black border cropping during transition segmentation）。

### [MOVA](../models/MOVA.md)

使用 PySceneDetect 做场景切点检测，但 MOVA 的关键创新在于把场景切点与语音边界联合起来做窗口采样，而不是简单按场景切分：
【检测】PySceneDetect 检测并记录全片所有场景变化点时间戳；Silero VAD 并行标注语音区间。
【多镜头窗口生成（附录 A.3 Algorithm 1）】遍历 VAD 语音段：窗口起点上界设为当前语音段起点；起点下界取三者最大值——上一个语音段的结束时刻、当前语音段之前最近的场景切点、当前语音段起点减去半个窗长（8.05/2）；在该区间内均匀随机采样窗口起点；窗口长度固定 8.05 秒；只有当窗口时间跨度内至少包含一个场景切点时才保留为多镜头样本；随后跳到第一个起点晚于窗口结束时刻的语音段，避免窗口重叠。
【单镜头窗口生成（附录 A.3 Algorithm 2）】遍历相邻场景切点构成的场景区间，在每个场景内寻找起点满足“起点 + 8.05 秒仍完整落在该场景内”的语音段，起点下界取（上一语音段结束、场景起点、当前语音段起点减半窗长）三者最大值，随机采样起点；若窗口末端越过场景结束点则中止；同样跳过重叠窗口。
【设计意图】(1) 通过下界约束“不早于上一段语音结束”避免窗口切进上一句话；(2) 通过下界约束“不早于最近场景切点”鼓励自然转场；(3) 通过“减半窗长”约束避免窗口起点离目标语音过远导致前半段空白；(4) 随机采样起点带来数据增广与位置多样性。整体是一套 shot-aware + speech-aware 的双重感知切分，专为唇同步任务定制。

### [合并条目：① Mochi 1](../models/Mochi_MAGI_Motif.md) ⚠️

① Mochi 1：未披露。[不确定]
② MAGI-1：使用开源 PySceneDetect 把长视频切为短片，明确要求「每个 clip 只含单个镜头」。因 PySceneDetect 对复杂转场（渐变、叠化等）处理不佳、切出的 clip 仍可能残留多镜头，团队追加了一个独立的 Transition Detection 过滤器兜底：稀疏采样关键帧，用 CLIP 计算相邻关键帧的语义相似度，若低于预定阈值则判定该 clip 含多镜头并整段剔除。注意其兜底策略是「删除」而非「二次切分」。
③ Motif-Video 2B：采用「过切分 + 语义再合并」的双向策略，是三者中最讲究的一套。第一步用「偏向过切分的保守阈值」检测场景边界，即刻意让假阳性（多切）多于假阴性（漏切转场）；第二步用基于 SigLIP embedding 相似度的 stitch detection 把相邻片段重新合并，专门恢复那些因瞬时剧烈运动或曝光变化而被误切开的连续镜头；第三步丢弃合并后不足 2 秒的片段。第三重兜底来自 VLM 的 multi_scene 标签——命中即丢，论文明确称其为「对场景切分的二次检查（a secondary check on scene segmentation）」。与 MAGI-1 相比，Motif 的差异在于承认切分器会两头出错并分别给出补救（合并纠过切、VLM 标签纠漏切），而 MAGI-1 只处理了漏切一侧。

### [Movie Gen](../models/Movie_Gen.md)

双工具组合：① FFmpeg 的场景变化检测（scene change detection）负责主切分——先检出场景边界，从每条原视频中采样1~2个时长超过16秒的场景，再从每个场景内随机抽取1个4~16秒片段作为训练clip；论文明确说明不做「不考虑场景边界的随机采样」，否则生成视频会出现频繁而突兀的场景切换。② PySceneDetect 的 Shot Boundary Detection 用作抖动检测器而非切分器——因为FFmpeg的motion score和motion vector难以识别频繁抖动的运镜，而抖动视频会被SBD拆成大量假阳性镜头，故用「每秒检出镜头数」作为抖动代理指标，超过0.85 shots/秒即剔除。③ 参照 Panda-70M 的做法，若clip起点与整段视频开头重合，则丢弃开头数秒（开头常含不稳定运镜或转场特效）。

### [NVIDIA NeMo Curator](../models/NeMo_Curator.md)

披露充分且有选型依据，是该 pipeline 中论证最扎实的一环。
【选型】TransNetV2 神经网络做镜头边界检测。Cosmos WFM 论文明确记录了与 PySceneDetect、Panda70M 的切分器、AutoShot 三个替代方案的基准对比，TransNetV2 在 BBC 数据集上取得 F1 = 0.967 的最佳成绩而被选用。这是公开资料中罕见的、把「镜头切分器选型」当作可量化决策来做的案例，值得直接借鉴其对比方法。
【框架实现】NeMo Curator 提供两种切分模式并可组合：fixed-stride（固定步长）与 scene-change detection（TransNetV2）。
【后处理】切分后有 stitching 阶段：用图像嵌入相似度判断相邻 clip 是否应合并，抑制过度碎片化。
【时长约束】< 2 秒丢弃，> 60 秒再切分。
【未披露】TransNetV2 的判定阈值取值、stitching 的相似度阈值。

### [OmniHuman 数据集 + OHBench](../models/OmniHuman.md) ⚠️

【方法】TransNetV2——专用的深度学习镜头边界检测模型（相比 PySceneDetect 的像素/直方图差分法，对渐变转场、淡入淡出、叠化等软切换的召回显著更高，在人工剪辑痕迹密集的 YouTube 内容上是更合适的选择）。位于流水线第一级的最前端，是所有后续处理的前提。
【选型评价】这是流水线中一个明显优于同类工作的选择：UniTalking 完全无切分环节、多数工作使用 PySceneDetect。YouTube 上的脱口秀、测评、影视剪辑类内容普遍带有大量转场特效，用阈值法切分会产生大量跨转场的脏片段，TransNetV2 是对症的选型。
【切分后的处理】切分产出的片段直接作为样本单元，时长由镜头本身长度决定（平均约 6.5 秒）。论文未说明是否对过长镜头做二次截断、是否设置了时长上下界（虽然音频治理阶段有「时长异常」剔除，暗示存在界限但数值未给）。[不确定]
【转场残留的兜底】第四级的感知哈希 + SigLIP 余弦相似度过滤「语义不一致的帧」，可视为对切分漏检的二次兜底——若一个片段内部仍存在跨场景的语义跳变，会在此被捕获。这种「切分 + 语义一致性复查」的双保险设计比单纯依赖切分器更稳健。
【阈值】TransNetV2 的判定阈值未披露。[不确定]

### [Open-Sora 系列](../models/Open-Sora.md)

【Open-Sora 1.x】使用 **PySceneDetect**，代码完全开源：`python tools/scene_cut/scene_detect.py meta.csv` 输出每个视频的场景时间戳列表（形如 `[('00:00:01.234','00:00:02.345'), ...]`），再由 `tools/scene_cut/cut.py` 按时间戳切分，产物命名 `{video_id}_scene-{scene_id}.mp4`；前置还有 `convert_id_to_path.py` 做损坏文件过滤（输出 intact 标记列）。
【Open-Sora 2.0】改用 **FFmpeg libavfilter 的 scene score** 做镜头边界检测（比 PySceneDetect 更快、可与转码同流水线完成），随后仍用 **PySceneDetect 的 Shot Boundary Detection** 作为相机抖动/异常跳变的二次检测器。切分后做固定 8 秒截断。
【Open-Sora Plan】不做基于内容的切镜，而是先用 **ffmpeg 机械切成 16 秒定长片段**，再用 **LPIPS 感知距离**检测片段内是否存在跳切（jump cut），把含切换的片段整条丢弃（保留率 97%）。这是与前者不同的技术路线：以「检测并丢弃」代替「检测并在切点处切分」，实现更简单但会损失部分可用素材。

### [Ovi](../models/Ovi.md) ⚠️

使用「场景检测（scene detection）」将长视频切为镜头内片段，再从中截取 121 帧 @24fps 的定长 clip。论文只写了 scene detection 这一通用表述，未点名具体工具（未说明是 PySceneDetect、TransNetV2 还是自研模型）[不确定]。切分与筛选耦合在同一步：切出的片段必须同时满足分辨率、运动、美学、人脸构成等条件才被保留，因此不是「先切后筛」的两阶段，而是切分即过滤。切分粒度决定了训练样本全部为单镜头，无镜头转场样本。

### [Script-a-Video](../models/Script-a-Video.md) ⚠️

本工作的镜头切分呈现一个关键的双重角色，需区分两个层面：
【数据预处理层面（未披露）】500K 片段是如何从长视频中切出的、是否用 PySceneDetect 或自研检测器、阈值如何设定，论文完全未说明。这是清洗侧的空白。[不确定]
【标注层面（充分披露且为核心创新）】MTSS 不把镜头切分当作预处理丢弃的中间产物，而是把镜头结构作为一等标注字段永久保留在 Shot 流中——每个 shot 带精确的 "time_range"，并携带 visual_description（客观按时序叙述核心动作）与 camera（专业电影语言：运动/视角/景别）两层描述，外加 references_in_shot 与 active_events 两个关系字段。因此单个训练样本内部可以包含多个镜头，而非被切碎成多个独立单镜头样本。这与 UniVerse-1 等「切分后只保留单镜头片段」的主流做法方向相反。
【下游收益】保留镜头结构使得生成模型可直接学习「在指定时间戳执行镜头转场」这一能力。论文引入的 Shot-Aware Structured Attention 正是利用 shot 边界把 Gemma-3 embedding 分区，让每段视频 token 只关注本镜头的语义，实现镜头间上下文隔离。
【切分精度的量化验证】Shot Boundary Deviation 指标（生成视频的镜头边界与脚本指定边界的帧级绝对偏差）：LTX-2-AV 基线 3.79 帧 → LTX-2-AV-MTSS（仅换 MTSS 提示词，不改架构）3.27 帧 → Ours w/o ID 0.38 帧 → Ours(Full) 1.36 帧。论文指出 Full 配置反而回升到 1.36 帧，是身份注入特征与时序镜头精度之间的潜在权衡（trade-off），归因于 VLM 编码器与 DiT 接口的设计，留作未来架构优化。

### [Seedance 2.0 与 Seedance 1.5 pro](../models/Seedance_20_Seedance_15_pro.md) ⚠️

Seedance 1.5/2.0 未直接描述。Seedance 1.0 的方法：采用自动镜头边界检测（automated shot boundary detection），通过分析帧间视觉差异（inter-frame visual dissimilarities）或使用预训练检测器识别自然场景切换；随后将视频切成最长 12 秒的短片段，每个片段可包含一个或多个时序连贯的镜头，以在保留局部叙事流的同时控制输入长度。未指明是否使用 PySceneDetect 等开源工具，倾向为自研/内部方案。[不确定：1.5 与 2.0 是否沿用同一方案、以及是否引入音频侧边界检测]

### [SkyReels 系列](../models/SkyReels.md)

【SkyReels-V2】双工具组合：所有原始视频先经镜头边界检测（明确点名 PyDetect / PySceneDetect 与 TransNet-V2），切分为单镜头视频片段，作为后续全流程的原子单位。属于业界标准做法（传统阈值法 + 学习式 TransNet-V2 互补）。
【SkyReels-V4】升级为 VLM 参与的「智能切分（intelligent segmentation）」：显式指出传统场景切割方法不足，改为「将 TransNet 的镜头边界预测与 VLM 结合，抽取语义完整的片段（combines TransNet's shot boundary predictions via VLM to extract semantically complete segments）」——即由 VLM 判断切分点是否产生语义完整的段落，避免把一个语义单元切碎或把两个语义单元并到一起。这是本条目中 V4 相对 V2 最具借鉴价值的升级，也是2026年「大模型下沉到数据切分环节」趋势的实例。未公布 TransNet 版本、VLM 具体型号与阈值参数。

### [Sora 2](../models/Sora_2.md) ⚠️

完全未披露。既未提及 PySceneDetect 等开源工具，也未提及自研 shot-aware splitting 模型。虽然模型具备多镜头生成与跨镜头世界状态保持能力，暗示训练pipeline中存在镜头级切分与镜头组织逻辑，但方法零披露。[不确定]

### [SpeakerVid-5M](../models/SpeakerVid-5M.md)

使用 PySceneDetect（SceneDetect）作为唯一的镜头切分工具，无自研切分模型：
【检测原理】分析画面颜色与亮度的变化以定位「视觉显著转场（visually significant transitions）」，在转场处切断。
【长度约束】切分后的 clip 被修剪到 3 至 14 秒；不足或超长的做丢弃/截断处理。
【与说话人结构的耦合】场景切分只是第一步，真正决定最终 clip 边界的是后续的三重约束：(1) 3D-Speaker 的说话人时间区间；(2) YOLO 人体跟踪的时空连续段；(3) SyncNet 的音画绑定有效区间。因此 SpeakerVid-5M 的切分本质是「shot-aware + speaker-aware + track-aware」的三重感知切分，而非单纯的场景切分。
【与 MOVA 的对比】MOVA 用 PySceneDetect 场景切点 + Silero VAD 语音边界联合生成 8.05 秒定长窗口，刻意保留多镜头样本；SpeakerVid-5M 则用 PySceneDetect 切断后再按说话人轨迹裁剪，产出严格单镜头的变长片段。两者都是 shot-aware，但目标不同：前者服务于「学会转场」，后者服务于「学会人物交互」。
【代码可得性】场景检测与说话人分离的代码在开源仓库中作为第 6 步提供（标注为可选，部分结果已预计算）。

### [Step-Video-T2V](../models/Step-Video-T2V.md)

PySceneDetect 的 AdaptiveDetector 函数（自适应阈值检测器，相对 ContentDetector 对渐变与摄像机运动更鲁棒）检测场景变化点，再调用 FFmpeg 按检测到的边界切分为单镜头片段。切分后统一去掉每个片段的首 3 帧与尾 3 帧，理由是这些帧常包含不稳定的镜头运动（转场残留、抖动）。
与同期工作的对比：未使用 TransNetV2 等双路交叉验证（HunyuanVideo 用了），也未额外训练「转场分类器」做二次清洗（HunyuanVideo 1.5 用了），属于较标准的单工具方案；但「首尾各去 3 帧」这一补偿性设计是其特点。

### [UniTalking](../models/UniTalking.md) ⚠️

论文完全未提及镜头切分：无 PySceneDetect、无自研切分模型、无 shot-aware splitting、无转场检测、无切分后的时长约束。这是 UniTalking 数据流水线相较同类工作最显眼的一处结构性缺失（UniVerse-1、MOVA、Apollo 等均把镜头切分作为漏斗的独立一级）。
可能的原因推断：其主要数据来源 OpenHumanVid 在上游已完成「解码、裁剪、切分」（decoding, cropping, segmentation）预处理，交付的即是切分后的片段单元，因此 UniTalking 可直接消费而无需自建切分模块。华为内部数据是否同样已预切分则无从判断——若内部数据为原始长视频，则缺少切分环节会是一个实质性问题；若已预切分，则该缺失可解释为「上游承担」。论文对此无任何说明。[不确定]
间接的镜头一致性保障：跨模态过滤中的 LightASD 与 LipSync 均依赖连续人脸轨迹，跨镜头样本大概率无法通过，因此镜头一致性在事实上由跨模态过滤兜底，但这是副作用而非设计。

### [UniVerse-1](../models/UniVerse-1.md)

使用 PySceneDetect（论文以脚注给出仓库地址 https://github.com/Breakthrough/PySceneDetect）做场景切换检测并按切点切分，属于业界最常规的做法，无自研模型、无 shot-aware 的特殊设计。
【唯一的后置约束】切分后长度短于 5 秒的片段直接剔除，以保证时序连贯性（temporal coherence）并满足下游 5 秒训练窗口的需求。
【与语音边界的关系】切分完全由视觉场景切点驱动，不与语音边界（VAD）联合决策——这与 MOVA 用 Silero VAD + PySceneDetect 双信号联合生成窗口、并让窗口起点自适应避免截断语句的做法相比要粗放得多，意味着 UniVerse-1 的训练窗口可能从句子中间切入或切断。
【切分参数】未披露检测器类型（ContentDetector / AdaptiveDetector）与阈值。
【VGGSound/AudioSet】未做镜头切分，仅施加 5 秒最短时长约束。

### [Unison](../models/Unison.md) ⚠️

论文完全未描述镜头切分：未提及 PySceneDetect 或任何场景切分工具，未提及自研切分模型，未提及 shot-aware splitting 或任何切分参数。
【最可能的实际情况】Unison 很可能根本不做镜头切分，而是直接消费上游数据集已切分好的 clip。五个音视频数据源均以「预切分片段」形式发布——HDTF、VFHQ、CelebV-Text 是已裁剪的人脸视频片段，VGGSound 是固定时长的视听事件片段，OpenHumanVid 亦为切分后的 clip 集合。在这种「聚合既有 clip 数据集」的构建模式下，镜头切分工作已由上游各数据集各自完成，Unison 的 pipeline 只需做筛选而非切分。这与 UniVerse-1、MOVA 等从原始长视频出发、必须自建切分环节的工作有本质区别。
【间接证据】平均 clip 时长约 5.4 秒，与上游数据集的典型片段长度吻合；论文全文未出现任何镜头/场景切换相关词汇（shot、scene cut、transition）。
【风险】沿用上游切分意味着切分标准在五个数据源之间不统一（例如 VGGSound 的固定 10 秒切分不考虑镜头边界，而 VFHQ 的人脸裁剪片段是严格单镜头），论文未讨论这一异质性可能带来的影响。[不确定]

### [Veo 3 / Veo 3.1](../models/Veo_3_Veo_31.md) ⚠️

[不确定] 镜头切分方法未披露，未提及 PySceneDetect 或任何自研 shot-aware splitting 方案。反向证据是模型在 8 秒生成中会自发产生镜头切换，说明训练样本并非严格切分至单镜头粒度，或至少保留了相当比例的多镜头样本。

### [Vidu S1](../models/Vidu_S1.md) ⚠️

沿镜头边界（shot boundaries）将原始视频切为单镜头 clip；长镜头进一步再切分。关键约束是「切点不得落在语音中间」（speech-aware cutting），这是为唇形同步任务定制的切分策略。未说明具体使用 PySceneDetect、TransNetV2 还是自研镜头边界检测模型 [不确定]。

### [Wan 2.5 / 2.6 / 2.7](../models/Wan.md) ⚠️

全系（2.1 至 2.7）从未披露镜头切分方法——既未提及 PySceneDetect / TransNetV2 等开源工具，也未说明是否有自研 shot-aware splitting 模型或阈值参数。可确认的只有切分粒度：Wan 2.1 训练样本为5秒 clip；Wan-Dancer 明确将原始视频切为5秒 clip 且相邻重叠50%（这是同团队唯一写出的具体切分策略，但属舞蹈垂类）。2.6/2.7 的多镜头叙事能力必然要求训练数据具备镜头边界标注与跨镜头一致性标注，其切分与镜头级对齐方案是本条目最重要的未知项之一。[不确定]

### [音视频生成评测基准合集](../models/av_benchmarks.md) ⚠️

五者均不涉及传统意义上的镜头切分（PySceneDetect 类），因为素材本身即为短单镜头片段。相关的时间轴切分策略为：
【AV-SyncBench】统一切成 0.64 秒互不重叠 chunk 用于视听嵌入的逐段对齐计算，这是其时序评测的基本粒度（对应 CAV-MAE / Synchformer 类模型的原生窗口设定）。
【VABench】Desync 指标仅取生成视频的首 4.8 秒与末 4.8 秒两个窗口送入 Synchformer 预测偏移，规避中段长时序不稳定问题。
【PhyAVBench】录制阶段按「单一物理事件」为单位切分，每条视频对应一次完整的发声事件（如一次撞击、一段流水），属语义事件级切分而非镜头级。
【AVBench / Omni-Judge】未涉及切分[不确定]。

### [视频 Caption 模型生态](../models/caption_models.md)

captioner 本身通常不做镜头切分（切分在其上游完成），但有三类交互：
【打标器承担叙事分组】CineDance 用 Qwen3.5-27B 做镜头分组与叙事边界判定（自底向上镜头索引分组优于直接让 LLM 输出时间戳），F1 = 88.4%、仅 3.1% 序列短于 20 秒软阈值，是打标模型直接承担切分决策的代表。
【打标器描述转场】ShareCaptioner-Video 的 DiffSW 显式识别 scene transitions；MOVA 的视觉标注指令明确要求「聚焦视频场景转场」；AVoCaDO 的 Spatio-temporal & Cinematography 维度覆盖场景切换。
【切分质量反过来决定打标质量】Koala-36M 的核心贡献之一即「更准确高效的转场检测方法」，其消融显示：用 Koala-all 训练相较 Panda-70M 训练，VBench 主体一致性 +1.1%、背景一致性 +2.4%，论文将该时序质量提升归因于更准的镜头切分 —— 这是「切分 → caption 一致性 → 生成质量」链条的唯一量化证据。CineDance 的伪影审计显示 CineDance-1M 不合规率 2.8% vs Koala-36M 37.4%（13.4× 改善）。
【上游常用工具】PySceneDetect 仍是行业默认；各家自研的转场检测模型（Koala-36M、CineDance）主要针对渐变转场与快速剪辑的漏检问题。

### [几何/结构化标注数据集合集](../models/geometric_datasets.md)

SceneScribe-1M：TransNetV2 深度模型检测硬切与渐变转场，仅对判定为“非连续”的视频执行切分，切后clip重新过滤以保证语义连贯；SpatialVID：改造版 PySceneDetect——调整敏感度阈值并引入基于间隔的多帧比较策略（interval-based multi-frame comparison），弥补原版对渐变转场的漏检，输出 3–15 秒clip；WildWorld：无需镜头切分，游戏引擎连续录制，按动作序列边界组织样本（训练取81帧窗口）；Action100M：不用传统镜头检测，改用 V-JEPA 2 语义嵌入 + Ward linkage 带时序约束的层次凝聚聚类做语义级时序分割，可产出跨镜头的层级片段树，丢弃<0.5秒片段

### [视频生成后训练数据策略](../models/post_training_data.md) ⚠️

锚论文未涉及 [不确定]。后训练阶段一般不再重做镜头切分——SFT 精选集是从已完成切分的预训练池中筛选的子集，因此镜头切分属于上游预训练数据 pipeline 的职责。横向唯一相关的是 Step-Video-T2V 的 SFT 人工评审把「场景转换是否平滑」列为四项标准之一，即在后训练阶段用人工把关切分残留问题（一个 clip 内混入转场）——这是「后训练阶段作为切分质量最后一道防线」的少数明证。Motif-Video 2B 的 SFT 准入含 action=Dynamic，间接排除了静止的转场残片。

### [主流视频预训练数据集合并调研：Panda-70M、InternVid、Koala…](../models/pretraining_datasets.md) ⚠️

**这是七者技术分化最明显的一环，可分为四种路线**：
【路线1·PySceneDetect 直接用（参数各异，且参数选择本身即方法）】
- **InternVid**：ContentDetector 阈值 **27**，无其他修饰。
- **MiraData**：content-aware 阈值 **26**——刻意调低以**故意过度切分**（「ensuring that all distinct clips are extracted」），为后续缝合留料。
- **LVD-2M**：**最巧妙的参数化**。明确弃用 AdaptiveDetector + 滚动均值（「难以检测带淡入淡出的缓慢镜头切换」），改用 **ContentDetector，阈值50，min_scene_len=0帧，帧采样率 0.5fps**。机理是：0.5fps下相邻采样帧相隔2秒，**任何2秒内的显著变化都会被判为切换，从而把通常在2秒内完成的淡入淡出也一并滤掉**。用采样率而非检测器复杂度来解决渐变转场，是低成本高收益的设计。
- **UltraVideo**：AdaptiveDetector **两轮 + 滚动均值**以降低相机运动造成的误切，再用 **DINOv2 对每个clip的首尾各5帧**做特征相似度，补捉 AdaptiveDetector 漏掉的溶解转场。
【路线2·PySceneDetect + 语义特征二次约束与缝合（Panda-70M）】
PySceneDetect ContentDetector(阈值25, min_scene_len=15帧) → 对超过5秒的clip**递归切出前5秒**（用于处理无剪辑的连续拍摄素材；发布代码把5秒改成了7秒「以获得更好的切分效果」，**要精确复现论文需改 event_stitching.py 第199/200行**）→ 取每个n帧clip在 **0.1×n 与 0.9×n** 处的 **ImageBind** 特征，**首尾距离≤1.0** 才保留 → 相邻clip的 **‖f(C¹尾)−f(C²首)‖≤0.6** 则缝合。效果由其自创的 **Max Running LPIPS** 指标衡量（1fps关键帧间LPIPS的最大值）：字幕对齐法0.408/11.8秒、纯PySceneDetect **0.247**/4.1秒、本方法 **0.256/7.9秒**——**即以0.009的一致性代价换来近2倍的片段长度**，这是该论文最漂亮的一条量化论证。
【路线3·自研学习式转场检测（Koala-36M，唯一）】
**Color-Struct SVM（CSS）+ 时序3σ门限**，两阶段：
(a) 训练数据自监督构造——**同一视频内的帧对为负例（无转场）、不同视频的帧对为正例（有转场）**；两个特征：`d_color` = BGR直方图相关性（帧resize到256×256，每通道254 bins，范围[1,255]），`d_struct` = 在Canny增强的亮度图上算SSIM（E=max(Gray(I), Canny(Gray(I)))，灰度图resize到128×128，Canny(100,200)，data_range=255）；线性SVM，**发布代码中给出了拟合好的精确系数**：`svm_score = 4.61480465×bgr_similarity + 3.75211168×canny_similarity − 5.485968377115124`。
(b) 时序概率门限——对SVM分做**3抽头滑动平均**得 conv_svm；`svm_score<0` 直接判硬切；否则对帧号≥8且 conv_svm<0.75 者，用**前8帧**估计 μ、σ（先排序**去掉首尾各20%再算**，鲁棒统计），当 `conv_svm[i] < μ − 3×max(0.2, σ)` 判为转场（**σ下限0.2**防止退化）。片段需长于8帧才保留，且**首尾各腐蚀4帧**。
性能（1万条人工标注clip，约半数含转场）：**准确率0.7741 / 召回0.9395 / 精确率0.7547**，对比 PySceneDetect(HSL) 0.4421/0.3096/0.5920、PySceneDetect(HSL+edge) 0.4574/0.4146/0.5854。**换算F1：0.838 vs 0.407 / 0.485。** 速度上因特征提取固定降采样到256/128，在高分辨率上反超：1080p 12.26ms vs PySceneDetect-HSL 26.16ms、4K 41.98ms vs 102.55ms（对HSL+edge在4K快6.4倍），但**在256分辨率上反而更慢**（1.42 vs 0.68ms）。
【路线4·过切后用多模型投票缝合（MiraData，唯一）】
相邻短片段用**四模型「两两成对、组内需一致」的表决**重新合并：视觉语言模型组 **Qwen-VL-Chat + LLaVA**（判断「是否同一场景」），图像特征组 **ImageBind + DINOv2**（判断「特征是否相似」）。规则原文：「A connection is made only if **both** vision language models **or both** image feature extraction models agree」，即 (VLM₁∧VLM₂) ∨ (Feat₁∧Feat₂)。设计理由：VLM 擅长识别内容连贯的转场，特征相似度擅长修复被错误分离的片段。**这是七者中唯一把「缝合」而非「切分」当作核心动作的 pipeline**，也是 MiraData 能拿到72秒平均时长的直接原因。
【路线5·未披露细节】OpenVid-1M 仅称使用「级联的镜头/切点检测器」，具体模型与阈值未披露。[不确定]

## 质量过滤（美学评分、清晰度、OCR文字过滤、黑边/水印/logo检测）

`quality_filtering` · 详细程度: detailed

### [Allegro](../models/Allegro.md)

质量过滤分为三个正交维度，均给出了逐阶段阈值（Table 1）：
【亮度】将帧转灰度评估平均亮度，保留区间 [20, 180]（0–255 尺度），四个阶段一致，用于剔除全黑/过曝/纯色画面。
【清晰度】DOVER（Disentangled Objective Video Quality Evaluator）视频质量评分，仅在微调阶段启用，阈值 ≥0.07；预训练阶段不设该门槛。
【语义/时序一致性】LPIPS 帧间感知距离，仅在微调阶段启用，阈值 ≥0.05（下界，用于剔除几乎完全静止/重复帧的样本）。
【美学】LAION Aesthetics Predictor 对图像与视频中间帧打分，是唯一贯穿全部四个阶段且逐级收紧的指标：T2I 预训练 ≥4.8 → T2V 360p 预训练 ≥4.8 → T2V 720p 预训练 ≥5.0 → 微调 ≥5.3。
【文字/水印/黑边】CRAFT 场景文字检测器 + 水印检测器，文字区域面积占比阈值 ≤0.05%（四阶段一致，极其严格，几乎排除任何带字幕/台标/花字的素材）；黑边（black borders）与水印同属「content-irrelevant artifacts」一并剔除。
【图文一致性】CLIP 视觉—文本余弦相似度，预训练 360p / T2I 阶段 ≥0.17，720p 预训练与微调阶段 ≥0.20。
Appendix A Fig.11 展示了美学分数分布随阶段推进整体右移（360p 预训练 → 720p 预训练 → 微调），验证了分层收紧的实际效果。

### [Apollo](../models/Apollo.md) ⚠️

分视频侧与音频侧两条线，维度枚举清楚但阈值几乎全部缺失：
【视频侧四维质量建模】
- 动态质量（dynamic quality）：主体运动比例（subject motion ratio）、镜头稳定性（camera stability）
- 静态质量（static quality）：清晰度（sharpness）、美学（aesthetics）、色彩饱和度（color saturation）
- 内容自然度（content naturalness）：无过度特效与水印（no excessive effects/watermarks）
- 安全性（safety）
【视频侧硬性剔除条件】低分辨率、低 SNR/MOS、静音占比超过 20%（原文：「We discard those videos with low resolution, low SNR/MOS, or over 20% silence」）。注意此处 SNR/MOS 本属音频指标却出现在视频过滤条款中，说明音画质量在第一道闸门就被联合考量。
【唯一公开的数值阈值】静音占比 < 20%（silence ratio < 0.2）。视频质量各维度的打分模型（是否用 LAION-Aesthetic、DOVER、MUSIQ 等）与阈值数值均未给出；论文评测部分使用 MANIQA、aesthetic-predictor-v2-5、Musiq 计算美学分，但那是评测指标而非数据过滤器，不可混用。
【水印/logo】仅以「no excessive effects/watermarks」一句纳入内容自然度维度，未说明检测方法（是否 OCR、是否专用水印检测模型），也未说明是剔除还是裁剪。
【OCR/烧录字幕】完全未提及，不同于 MOVA 把「有无烧录字幕」做成显式可控属性的做法。[不确定]（维度明确，阈值与方法缺失）

### [CineDance / CineDance-1M](../models/CineDance.md) ⚠️

整体采取「全量打分 + 元数据保留 + 硬规则只用于伪影」的混合策略，这是与多数数据集「阈值一刀切」做法的显著区别：
【硬性剔除（规则型）】
  - 烧录字幕：EasyOCR 检测并裁除，镜头切分后再复验一遍；
  - 黑边/信箱框：FFmpeg black-border detection，同样两道（粗裁 + clip 级细验）；
  - 片头/片尾/非正片内容：MLLM 引导的时序截断，截断量 t = max(5min, 0.1L)。
【软性打分（元数据型，不做硬裁剪）】
  - DOVER：拆为 Aesthetic Quality（美学质量）与 Technical Score（技术质量/清晰度）两个分量；
  - AMT：运动平滑度（Motion Smoothness）；
  - 音频侧 DNSMOS 与 CLAP 时序方差；
  - 对齐侧 ImageBind 与 SyncNet。
  论文原文明确表述「we store all quality scores as metadata, enabling users to flexibly construct task-specific subsets」——将阈值决策权交给下游使用者，因此论文本身未公布任何具体的美学分/技术分截断数值。
【人工伪影审计（终验）】随机抽 500 条片段，三名独立标注员分别审查残留伪影，分歧由联合复议裁定。审查目标伪影清单：烧录字幕、台标 logo、信箱黑边、水印、电视网叠加层、片名卡、片尾字幕、录屏画面、转场特效、静帧停留。结果：不合规率 2.8%，对照 Koala-36M 的 37.4%，改善 13.4 倍。
【水印/logo 的自动检测手段】论文未说明是否有自动水印/logo 检测模型，从描述看主要依赖上游数据集清洗 + 最终人工抽检。[不确定]

### [CogVideoX](../models/CogVideoX.md)

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

### [Cosmos-Predict2.5](../models/Cosmos-Predict25.md) ⚠️

质量过滤是七级漏斗中最重的一环，由六个独立过滤器串行构成，覆盖美学、技术、语义三个层次，且论文明确了排序原则（廉价在前、昂贵在后）：
(1) 美学打分 aesthetic scoring filter —— 按美学质量给输入打分并分级；打分器型号与阈值未披露。
(2) OCR filter —— 「attempts to remove clips with excessive text overlay」，剔除文字叠加过多的片段（字幕、贴片文字、台标类）；检测器与文字面积阈值未披露。
(3) perceptual quality filter 感知质量过滤 —— 论文明确「akin to DOVER (Wu et al., 2023)」，即采用 DOVER 式的分离视角视频质量评估（DOVER 将 VQA 拆为 technical 与 aesthetic 两支），目标是剔除「technical distortions and perceptual artifacts」（技术失真与感知伪影），如压缩块效应、噪声、模糊。
(4) semantic artifacts filter 语义伪影过滤 —— 明确「akin to VTSS (Wang et al., 2025)」，目标是剔除结构/语义层面的异常：video-in-video（画中画、录屏套娃）、poor transitions（劣质转场）等。这是相对 Predict1 新增的一级，专门对付「技术指标正常但结构不适合训练」的样本。
(5) VLM filter —— 用 Qwen2.5-VL 系视觉语言模型「with higher precision」剔除一组不良问题（a set of undesirable issues），作为终审；因计算昂贵而置于最末。
(6) content type classifier + 物理真实性裁剪 —— 内部训练的内容类型分类器给出 26 类 taxonomy 标签，并在此阶段剔除「physically unrealistic phenomena」：视频游戏画面、合成视觉图案、动画、卡通，以维持与真实物理世界分布的对齐。这是 Physical AI 定位下的特殊质量标准——「非物理真实」本身被定义为一种质量缺陷。
黑边与空间 padding 不靠过滤而靠第三阶段的 cropping 直接裁除（修复而非丢弃）。水印/logo 检测未被单独列出（可能并入 OCR 或 VLM filter）。
所有过滤器的阈值数值一律未公布；领域支线则明确使用「a domain-specific subset of filters with adjusted hyperparameter values」（领域特定的过滤器子集 + 调整过的超参），说明阈值是按域可调的。[不确定：各过滤器的具体模型型号与阈值数值]

### [Data-Juicer 2.0](../models/Data-Juicer.md) ⚠️

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

### [Foley-Omni](../models/Foley-Omni.md) ⚠️

质量过滤是清洗漏斗的第一级，采用视觉、音频双通道并行把关，全部为阈值型硬过滤，无加权综合评分。
【视觉质量】
  · 分辨率 ≥480p —— 剔除低清片段，保证 CLIP 与 Synchformer 视觉特征抽取的可靠性。480p 门槛相对宽松（对比文生视频模型常见的720p/1080p门槛），合理之处在于视频仅作条件而非生成目标。
  · 码率 ≥1 Mbps —— 与分辨率形成双保险，专门针对「分辨率标称高但压缩严重、实际细节丢失」的样本。这是较少见但很实用的一条，因为上采样过的低质视频能通过分辨率检查却通不过码率检查。
  · motion score ∈ [0.1, 3.2] —— 双边阈值。下界剔除近静止画面（缺乏视觉事件，无法为音效生成提供时序线索），上界剔除过度剧烈运动（运动估计不可靠、声画对应混乱）。motion score 的具体计算方法（光流均值/帧差）论文未指明。
【音频质量】
  · Meta AudioBox Aesthetics 分数 ≥0.6 —— 使用 Meta 开源的 AudioBox Aesthetics 无参考音频美学/质量评估模型。这一个指标同时承担了多重职责：低分自然过滤掉静音片段（无内容则美学分极低）、剔除底噪大/削波失真/低采样率/编码劣化的音轨。相比传统 SNR 阈值，AudioBox Aesthetics 是学习型的感知质量评估，更贴近人耳判断，属于2025–2026年音频数据清洗的新范式。
【未涉及的常规项】[不确定] 论文未提及美学评分（如 LAION-Aesthetics）、OCR 文字过滤、黑边检测、水印/logo 检测、字幕烧录检测等文生视频清洗中的标配环节。对本任务而言这些确实优先级较低（视觉只需支撑语义与同步判断，画面美观与否不影响音频生成），但水印/字幕遮挡嘴部区域可能影响 VisualTTS 的唇同步学习，这一风险论文未讨论。

### [Goku](../models/Goku.md)

多维阈值过滤，最显著特点是**所有阈值按目标分辨率分档设定，高分辨率档全面收紧**（论文 Table 4）：
【基础技术属性（硬门槛）】
  - 时长 ≥ 4 秒；
  - 分辨率 min{高, 宽} ≥ 480；
  - 码率 ≥ 500 kbps（剔除高压缩、块效应严重的低码率素材）；
  - 帧率 ≥ 24 FPS（Film Standard）或 23.976 FPS（NTSC Standard），剔除低帧率/抽帧素材。
【美学评分（Aesthetic Score）】用美学模型对关键帧打分：
  - 480p 档：分数 < 4.3 丢弃；
  - 720p 及以上档：分数 < 4.5 丢弃。
  （论文未指明具体美学模型版本，推测为 LAION-Aesthetic 系列或其内部改版。）
【OCR 文字覆盖率过滤】检测关键帧中的文字区域占画面面积比例，超阈值丢弃（主要剔除字幕、贴片广告、花字、屏摄内容）：
  - 480p 档：文字面积 ≤ 0.02（2%）；
  - 720p 及以上档：文字面积 ≤ 0.01（1%）。
  这一项阈值极为严格，说明团队高度重视避免模型学会生成乱码文字。
【去重（与质量联动）】同源片段用感知哈希比对，哈希相近时**保留美学分更高的那条**——去重决策直接以质量分为仲裁依据。
【未提及的常见项】论文未描述黑边检测、水印/logo 检测、模糊度/清晰度（如拉普拉斯方差、NIQE）、亮度/饱和度异常、时序闪烁一致性等过滤维度，也未提及图像侧（T2I 数据）除美学分之外的独立过滤标准。相较 2025 年底至 2026 年的工作，Goku 的质量过滤维度偏经典、偏浅层。

### [Hailuo / MiniMax Video](../models/Hailuo.md) ⚠️

[不确定]（完全无披露）。未提及美学评分器、清晰度/技术质量评分、OCR 文字检测、黑边裁切、水印/logo 检测等任何一项具体手段，也未给出任何阈值。唯一相关的官方措辞是 Hailuo 02 的「improved quality and diversity（质量与多样性提升）」以及 Hailuo 2.3 的能力描述，均为结果性宣称而非方法披露。
从模型表现可反推的间接证据（推测，非披露）：Hailuo 系列输出在画面干净度与电影感上表现较好、极少出现字幕/台标残留，暗示存在有效的画面干净度与水印/字幕过滤；「原生1080p」暗示存在分辨率与清晰度门槛。

### [HunyuanVideo-Foley](../models/HunyuanVideo-Foley.md) ⚠️

本工作的质量过滤呈现极强的非对称性——音频侧多层严格把关，视觉侧完全放行：
【音频质量过滤（三层）】
1) 有效采样率 / 带宽检测：仅保留有效采样率 >32 kHz 的样本。注意是「有效」（effective）而非文件声明值，即通过频谱分析确认高频段确有能量，能识别并剔除低码率音源上采样伪装的「伪高清」。这是相当讲究的一条，且与模型 48 kHz 的输出规格直接对应——若训练数据带宽不足，模型无法学会生成高频细节。
2) AudioBox-Aesthetics 音频美学评估：使用 Meta 的 audiobox-aesthetics 工具包对音频做感知质量打分。该工具输出四个维度：PQ（Production Quality，制作质量）、PC（Production Complexity，制作复杂度）、CE（Content Enjoyment，内容愉悦度）、CU（Content Usefulness，内容有用性）。论文未说明使用了其中哪几个维度、阈值各为多少。作为对照，MOVA 明确给出 PQ>5.0 / CU>4.5 / CE>2.5 的三分数阈值组合。[不确定]
3) SNR 信噪比：作为「补充指标」（supplementary metrics）辅助判定，未给阈值。[不确定]
【视觉质量过滤：完全没有】整条流水线中不存在任何视频画质相关的判据——无分辨率下限、无码率约束、无美学评分（无 DOVER、无 LAION-Aesthetic）、无清晰度/模糊检测、无黑边检测、无水印/logo 检测、无 OCR 字幕过滤。
【这一取舍的合理性分析】看似激进，实则符合任务本质：视频在本模型中只作为条件输入，经 SigLIP-2 编码为语义特征后，画面的清晰度、水印、黑边等因素几乎不影响「这个画面里在发生什么声学事件」的语义判断。花费算力做视觉质量过滤对输出音频质量的边际收益极低。相比之下，UniVerse-1 这类联合生成模型必须做严格视觉过滤（因为要生成画面），两者的差异是任务形态而非工程水准的差异。
【值得注意的反向风险】视觉侧零过滤意味着训练数据中可能混入大量画面质量极差的样本（严重压缩、极低分辨率、强水印遮挡），这类样本的视觉语义识别本身就不可靠，可能给模型带来噪声监督。理想做法是保留一条低成本的视觉语义可识别性判据（如 SigLIP 特征的置信度或熵），但论文未提及任何此类设计。[不确定]
【AudioBox 的双重角色】值得留意的是 AudioBox-Aesthetics 既被用作训练数据的过滤器，又被用作最终评测的指标（主表中的 PQ、PC、CE、CU 分数）。这构成一定程度的循环：用某指标筛数据、再用同一指标评效果，会系统性地高估模型在该指标上的表现。论文未讨论这一潜在的评测偏置。

### [HunyuanVideo](../models/HunyuanVideo.md)

【原版】多路并行、逐级加严：
· 美学与技术质量：Dover 模型双视角打分（aesthetic view + technical view）；
· 清晰度：自研 blur 检测模型（training-based），剔除视觉模糊内容；起始帧用 Laplacian 算子选清晰帧；
· 文字：OCR 模型剔除文字覆盖过多的片段，并对含字幕的片段做裁切（crop）而非整片丢弃；
· 水印/黑边/logo/敏感信息：YOLOX-like 检测模型定位并剔除或裁除；
· 各过滤器的具体阈值数值论文未公布，只说明「不同训练档使用不同严格度的阈值」。
【1.5】改为「一个综合质量评估算子 + 一个美学算子 + 一组基础规则」的结构：
· 综合质量算子四维度：sharpness（清晰度）、detail retention（细节保留）、noise & artifacts（噪声与伪影）、dynamic range（动态范围）——比原版单一模糊检测更细分，尤其新增了噪声/伪影与动态范围两个偏「摄制质量」的维度；
· 美学算子：低美学分剔除；
· 基础规则：padding 黑边、拼接痕迹、宫格拼贴（collage）直接剔除；
· 字幕/logo/水印：改为空间裁剪，裁后保留面积 <60% 才丢弃。
两代均未公布任何阈值数值。

### [InstructAV2AV](../models/InstructAV2AV.md) ⚠️

质量过滤分布在pipeline的首尾两处，形态上分为「素材准入过滤」与「合成产物验收过滤」两类，后者是编辑类数据集特有的环节。

【A. 素材准入过滤（阶段一，作用于真实素材）】
  · 视觉美学：LAION Aesthetics Predictor（Romain and Christoph, 2022）剔除视觉低质片段。这是文生视频清洗中的标配工具，此处沿用。[不确定] 阈值数值未给出（该模型输出通常为 1–10 分，业界常用门槛在 4.5–5.5 区间）。
  · 运动强度：CoTracker3 网格点跟踪计算平均运动幅度，低于预设阈值的片段丢弃。[不确定] 阈值未给出。
  · 音频质量：Audiobox-Aesthetics（Tjandra et al., 2025，Meta 的无参考音频美学评估模型）剔除低质音频。[不确定] 阈值未给出。注：Foley-Omni 用同一模型时给出了 ≥0.6 的明确阈值，本工作未给。
  · 静音：PyDub 剔除低于 -45 dBFS 的片段。这是全文唯一给出确切数值的阈值。
  · 声源清晰度：非语音片段中剔除声源模糊（ambiguous sound sources）的样本，要求每个片段有唯一主导声源。这是一条较独特的过滤条件，服务于「可分离、可定向编辑」的下游需求。
  · 声画归属：语音片段要求语音与画内可见说话人时间对齐，实质剔除画外音、旁白、后期配音。

【B. 合成产物验收过滤（阶段三，作用于生成的 target）】
  多模态 LLM（Qwen3-Omni）五维度打分，须同时通过全部五项：
  (i) instruction fidelity 指令保真度 —— 是否严格执行了指令要求的编辑；
  (ii) content preservation 内容保持度 —— 非目标区域/环境声是否原封未动；
  (iii) perceptual quality 感知质量 —— 合成结果是否真实无穿帮；
  (iv) audio-video synchronization 音视频同步 —— 新音轨与新画面是否对齐；
  (v) safety 安全性 —— 见 safety_filtering。
  这五个维度并非泛泛的质量分，而是与编辑任务的四个失败模式一一对应（该改的没改 / 不该改的改了 / 改得不真实 / 改得不同步），设计上相当贴合任务本身，且与评测阶段的 11 项指标形成呼应（TV-A/TA-A 对应 i，SSIM/TC 对应 ii，FVD/FAD/LPAPS 对应 iii，AV-A/PEAVS/Sync-C/Sync-D 对应 iv）——「怎么筛数据就怎么评模型」的一致性设计。

【未涉及的常规项】[不确定] 未提及 OCR 文字/字幕过滤、黑边检测、水印/logo 检测、清晰度（模糊）检测、压缩码率门槛。其中字幕过滤的缺失值得注意：来源含大量影视片段，烧录字幕在编辑任务中会造成明显穿帮（编辑区域与字幕重叠时 mask-guided 合成会破坏文字），论文未讨论此风险。

### [2026 其他音视频联合生成](../models/JAVG_2026_misc.md) ⚠️

【ALIVE —— 四类算子 + 独创的六档清晰度体系（本批最系统）】
(1) OCR 文字与水印检测：原文「We employ Optical Character Recognition (OCR) to measure the proportion of overlaid text and detect watermarks」——注意其做法是「测量叠加文字的占比」而非简单二值判定，说明是按文字覆盖面积比例做阈值过滤（阈值未公开[不确定]）。
(2) 美学评分：「utilize a pre-trained aesthetics model for per-frame scoring」——逐帧（per-frame）打分而非抽帧打分，粒度较细。具体模型与阈值未公开[不确定]。
(3) 运动分析：RAFT 计算光流（见 motion_filtering）。
(4) 13 维低质量分类器：「a quality model ... is used to classify samples as either 'high-quality' or belonging to one of 13 distinct low-quality dimensions」——一个把低质量细分为 13 个维度的多标签分类器。这比常见的「单一质量分 + 阈值」精细得多，可支持按维度差异化处理（如某些维度可修复、某些必须丢弃）。但论文自始至终未列举这 13 个维度具体是什么[不确定]，是全文最令人遗憾的省略。
(5) 独立的 Clarity filtering 阶段：用「reference images across six distinct clarity levels」（六档清晰度参考图，从模糊到锐利）做评判基准——即把清晰度做成六级序数标签，而非连续分数或二值判定。1080p Refiner 阶段专用的 0.7M「high-clarity samples」即从最高档中筛出。这种「参考图锚定的序数分级」在同期工作中很少见，优点是标注一致性高、跨标注员可比。
【NAVA —— 多算子并行 + VLM 复筛】
(1) PaddleOCR 做 OCR 过滤与字幕擦除——特别之处在于不只是「检测到字幕就丢弃」，而是执行「subtitle removal」（字幕擦除/去除），把带字幕的素材修复后继续使用，显著提升数据利用率。这与 ALIVE「测量文字占比后过滤」的思路形成对比：一个修复、一个剔除。
(2) 视觉质量算子组：aesthetics（美学）、sharpness（锐度）、brightness（亮度）、motion score（运动分）四项并行打分。全部阈值未公开[不确定]。
(3) VLM-based filtering and tagging：用视觉语言模型二次筛选「视觉质量清晰」的片段——即在传统打分器之外叠加大模型语义判断（详见 model_as_data_judge）。
【OmniCustom —— 单一美学阈值（本批唯一给出美学阈值数值的工作）】明确剔除「aesthetics scores below 0.3」的视频——美学分 <0.3 即淘汰。这是本批工作中唯一公开的美学阈值数字，尽管其分数尺度（0–1 归一化）与具体美学模型未说明[不确定]。另有时长 <10 秒剔除。无 OCR / 水印 / 分辨率过滤描述[不确定]。
【StreamChar / CCL / Baton】未描述任何视觉质量过滤[不确定]。三者均依赖上游公开数据集自带的质量筛选（OpenHumanVid、SpeakerVid-5M、TalkVid 本身均带质量过滤），属于「质量控制外包」。
【ITS-JAVG（推理时的质量把关）】其验证器组合中 VQAScore 与 VideoReward-TA 承担视觉质量与文本一致性的把关，可视为「推理时的质量过滤」。论文的关键发现是：任何单一质量验证器都会被搜索算法钻空子（「the inference-time search algorithm exploits blind spots」），必须多验证器组合——这个结论对训练侧数据过滤同样有警示意义：单一美学分或单一质量模型作为唯一过滤依据，同样存在被数据分布「反向利用」的盲区。

### [音视频联合生成基线合集](../models/JavisDiT_baselines.md) ⚠️

只有 JavisDiT++ 有系统的视觉质量过滤，其余基本空白：
【JavisDiT++（本合集唯一）】明确「遵循 Open-Sora 的方法」做三项质量过滤：
(1) 美学评分（aesthetic scoring）——剔除画面质量差的片段；
(2) 光流/运动评分（flow scoring）——见 motion_filtering；
(3) OCR 评分（OCR scoring）——检测并剔除画面中文字过多的片段（字幕、弹幕、标题卡等），避免模型学会生成乱码文字。
三项的具体阈值全部未公开[不确定]，各自淘汰量也未拆分[不确定]。
此外 data.md 中还有两项工程性清理：剔除损坏视频（remove broken videos）、过滤少于 10 帧的样本。
【未提及的常见手段】即使是 JavisDiT++，也未描述水印检测、logo 检测、黑边检测与去除、压缩伪影检测、模糊检测、亮度/对比度过滤等[不确定]。
【MM-Diffusion】自建 Landscape 时提到数据集为「high-fidelity（高保真）」，但未描述任何自动化质量过滤流程；1,000 条的小规模意味着可能存在人工筛选但论文未说明[不确定]。
【AV-DiT】无质量过滤，直接使用公开数据集。
【Harmony】视觉质量过滤未提及[不确定]；唯一的筛选是「音视频一致性打分模型」，属于跨模态对齐过滤而非视觉质量过滤。
【UniAVGen】完全未提及[不确定]。
【JavisBench 评测集】提到「前置过滤（pre-filtering）以保证质量、剔除噪声候选」与「后置过滤（post-filtering）以保证多样性」的两段式，但使用的具体工具与阈值未披露[不确定]。

### [可灵 3.0 / 可灵视频 3.0 Omni](../models/Kling_30_Omni.md) ⚠️

多层复合过滤：(1) 硬性准入——分辨率与时长阈值、音视频文件损坏检测（Kling-Omni 基础级）；Kling-Foley 侧要求源视频≥720P。(2) 时序视觉质量打分——模糊（blur）、抖动（jitter）、压缩噪声（compression noise）识别与剔除（Kling-Omni 第二级）。(3) 同团队 Koala-36M 的公开做法给出更细颗粒：不再用单指标硬阈值，而是训练一个“训练适宜性评估网络”输出统一的 VTSS（Video Training Suitability Score），输入包含清晰度分、美学分、运动分等子指标，网络含动态分支（3D Swin Transformer）+静态分支（ConvNeXt）+特征/标签分支，经 Weight Cross-Gating Block 融合，阈值取2.5（依据分数双峰分布确定）；其核心动机是避免单指标硬阈值误删高质量数据。(4) 字幕/文字：Kling-Foley 数据规范为“尽量少含字幕”。[不确定：可灵3.0 Omni 是否有专门的黑边/水印/logo检测与OCR文字过滤模块，报告未提及]

### [LTX-2](../models/LTX-2.md) ⚠️

视频侧质量过滤是 LTX 系列披露最细的环节，核心是自研的成对偏好美学评分模型：
(1) 标注对采样：先用一个多标签网络（multi-labeling network）为数百万样本打标签，只在「至少共享 top-3 标签之一」的样本之间采样配对——目的是让比较发生在同类内容之间，从而最小化按美学过滤时引入的分布漂移（distribution shift）。这一设计是该 pipeline 中最值得借鉴的细节。
(2) 人工标注：数万（tens of thousands）图像对由人工标注哪一张美学更优。
(3) 模型训练：用这些偏好对训练一个 Siamese Network，学习保持标注序关系的美学分数（排序式而非绝对回归式打分），可同时评估视频与图像。
(4) 应用：为每个样本计算美学分，低于阈值者剔除（阈值数值未公布）；微调阶段进一步只取美学分最高的子集。
(5) 几何清洗：显式 Crop Black Bars 裁除黑边。
【未提及】OCR/字幕文字过滤、水印与 logo 检测、清晰度/模糊度判据、压缩伪影检测均无任何说明。[不确定]

### [LongCat-Video](../models/LongCat-Video.md) ⚠️

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

### [MOVA](../models/MOVA.md)

分为“Stage 2 通用三维质量评估”与“Phase 2 训练前二次专项过滤”两层，全部阈值均公开（附录 Table 9）：
【Stage 2 音频质量】
- 静音占比 Silence Ratio < 0.8
- 带宽 Bandwidth > 1,000 Hz
- Audiobox-aesthetics PQ（Production Quality，制作质量）> 5.0
- Audiobox-aesthetics CU（Content Usefulness，内容有用性）> 4.5
- Audiobox-aesthetics CE（Content Enjoyment，内容愉悦度）> 2.5
【Stage 2 视频质量】使用 DOVER 视频质量评估工具，从技术与美学两个视角打分：
- DOVER-Aesthetic（美学分）> 0.85
- DOVER-Technical（技术分）> 0.05
【阈值确定方法】不是拍脑袋或沿用文献值，而是“人工抽检不同 metric cutoff 下留存的视频，逐维度设定合理阈值”，在保证语料质量的同时保留足够多样性。
【Phase 2 二次专项过滤（三道正交过滤）】
1) OCR 字幕过滤：用 OCR 识别是否有烧录字幕（burned-in subtitles），保留无字幕视频约 9.5M clips，并在其 prompt 末尾追加一句 “This video has no subtitles.” ——注意这不是单纯剔除，而是把“有无字幕”作为可控属性显式教给模型，是一种“过滤 + 条件化”的混合做法。
2) 唇音对应过滤：保留 LSE-D ≤ 9.5 且 LSE-C ≥ 4.5 的视频，得到约 2.5M clips 的高质量唇音对应子集。
3) 视觉保真度过滤：DOVER 技术分 > 0.15，得到约 2.4M clips。
合并后 Phase 2 数据集为 16.8M clips / ~37,600 小时。
【Phase 3】进一步只用 720p 最高质量子集，DOVER 技术分 > 0.14（该阈值针对 720p 重新标定）。
【黑边/水印/logo】黑边通过 FFmpeg cropdetect 在预处理阶段检测并裁除后重新补边；水印与 logo 未做专门检测，但视觉打标 prompt 明确要求“Ignore all text, subtitle and watermark in the video”，即在标注层面把水印/字幕排除出描述内容，避免模型学习到把水印当作可描述元素。

### [合并条目：① Mochi 1](../models/Mochi_MAGI_Motif.md) ⚠️

① Mochi 1：仅知施加了 NSFW 过滤，其余质量过滤策略、工具与阈值一律未披露。[不确定]
② MAGI-1（11 类 Filter Actor，方法详尽、阈值全部留白）：
· 视频质量评估：采用 DOVER，其输出 overall / aesthetic / technical 三个分数；团队经实证发现「technical score 单独使用是最有效的指标」，故只用技术分。
· 美学：LAION 美学预测器；因该模型本为图像设计，团队用「首帧的美学分」代表整段 clip 的质量（注意与 Allegro 用中间帧的选择不同）。
· 过曝/欠曝：把每一帧转到 HSI 色彩空间、计算全片平均亮度，据此剔除过曝或欠曝样本；论文指出这类数据会「adversely affect training stability（损害训练稳定性）」——把曝光过滤的动机归到训练稳定性而非观感，是一个值得注意的表述。
· 边框检测：逐帧边缘检测 + Hough 变换识别跨帧持续存在的水平/垂直直线，把这些线视为候选边框，以「含边框帧的占比」作为过滤置信度。
· 文字检测：若任一帧字符数过多或文字区域占画面比例过大则整段丢弃。特别处理字幕这一例外——字幕字符少、占面积小，不易被上述规则捕获，但具有鲜明时空模式（固定出现在画面上方或下方、跨多个连续帧持续存在），据此专项识别并剔除。
· Logo 检测：用 Florence-2（支持开放词表目标检测），给定预设关键词集合来检测定位画面中的 logo 并输出置信度用于过滤。
· 角落人脸检测：解说类视频中主播常固定出现在画面某个角落，团队用人脸检测模型、结合人脸位置与检测置信度，对固定角落人脸的置信度做跨帧平均，以估计「存在解说员」的可能性并剔除。
所有上述过滤器的阈值在报告中均以「predefined threshold」表述而不给数值。
③ Motif-Video 2B（5 路正交质量信号，工具与部分参数具名）：
· 美学：Aesthetic Predictor V2.5（基于 SigLIP 的图像级美学预测器）；对每个视频沿时间均匀采样帧、算帧级分、再跨采样帧平均得到视频级分；用作分阶段过滤器——低美学尾部被剔除，且「cutoff becomes stricter at higher-resolution stages（分辨率越高截断越严）」。
· 亮度：沿用 OpenHumanVid 的公式 L = 0.2126R + 0.7152G + 0.0722B（ITU-R BT.709 亮度权重），在采样帧上统计并剔除目标阶段的极低/极高亮度尾部，以过滤严重欠曝/过曝、提升保留数据中主体与场景内容的可见性。
· 模型化训练适配度分（Model-based Suitability Score）：借鉴 Koala-36M，用一个把多种质量因子汇总成「该视频是否适合用于训练视频生成模型」的单一估计值；实践中保守使用，只作为拒绝过滤器剔除最低适配度尾部，其余样本仍需通过后续各专项过滤。
· 技术质量：DOVER，取其技术质量相关输出，过滤压缩伪影、噪声、失真、低锐度等退化。
· 文字/水印/黑边：见 pipeline_overview 第 1–3 级（OCR 初筛 + VLM watermark 标签 + cropdetect + PaddleOCR-VL 持久区域聚类）。
· VLM 质量标签：quality=low 的片段自 480p 阶段起被排除。
论文特别强调这些信号「不被合成为单一学习排序（not used as a single learned ranking）」，而是每个过滤器针对一种具体失效模式（曝光差、压缩伪影、静态、时序不稳），这与把多指标塞进一个综合分的做法形成明确的方法论对照。

### [Movie Gen](../models/Movie_Gen.md)

视觉过滤共6个filter，逐项细节（附录J.1给出模型与阈值）：
· 分辨率：最小宽或高 <720px 剔除（高分辨率阶段提到768px）。
· 宽高比：过滤以达成 60%横屏/40%竖屏（高清集 80%/20%）的目标配比。
· OCR文字过滤：内部OCR模型自适应采样帧→检测词→识别词文本；只保留「所有采样帧上 词检测分 × 词识别分 均 <0.6」的视频，用于剔除字幕/花字/贴片文字过多的素材。这一步在漏斗中是最狠的几刀之一（7%→1.94%）。
· 黑边/边框检测：自研简易检测器，先用一阶导数找出垂直/水平方向像素差值大的位置，再用扫描线算法定位边框；动机是训练数据带边框会导致生成视频也带黑边，竖屏素材尤其常见。
· 美学评分：对每个clip的中间帧应用公开的 LAION aesthetics 图像模型，分数 <4.0 剔除，用于去掉模糊、压缩伪影严重的片段；论文特别做过对比——计算多帧平均美学分并未显著提升对低质片段的召回，故只用中间帧（省算力）。
· 视觉特效与画质：另外训练了若干轻量视觉模型，输出帧级的「视觉美学、视觉质量、大边框、视觉特效」预测信号用于过滤。
· 起始片段：移除与视频开头重合的clip的前几秒。
SFT阶段追加：用 Detic 物体检测模型剔除主体过小的视频；人工复核画面是否杂乱、色彩是否过饱和、有无叠加文字与编辑特效。
音频侧对视频的质量过滤更轻量：OCR去带文字视频、去静态视频、去分辨率<480px的视频。

### [NVIDIA NeMo Curator](../models/NeMo_Curator.md)

视觉质量过滤由四类判别器组成（Cosmos WFM 生产版），其中前两类已在开源 NeMo Curator 中提供：
【1. 美学过滤（开源已提供）】使用 CLIP-based aesthetic model 对抽帧逐帧打分，再按 clip 聚合。关键参数：reduction 可选 min（默认）或 mean——默认取最小值意味着「一帧不合格则整个 clip 淘汰」的严格策略；score_threshold 默认 3.5；target_fps 默认 1.0；num_gpus_per_worker 默认 0.25。要求上游抽帧使用 sequence 策略且帧率匹配。Cosmos WFM 生产环境同样使用 3.5 这一阈值。
【2. 失真/画质评估】基于 DOVER（视频质量评估模型）的打分模型，剔除得分最低的 15% clip。该 stage 在开源版中未见对应实现。
【3. 文字叠加检测（OCR 类功能的替代方案）】不使用传统 OCR，而是在 InternVideo2 视频嵌入之上训练 MLP 分类器，识别后期添加的文字叠加（post-processed text overlay），剔除文字过多的视频。这一「用嵌入 + 轻量 MLP 替代重型 OCR」的做法在 PB 级规模下成本优势显著，是值得借鉴的工程取舍。
【4. 视频类型 taxonomy 分类】同样是 InternVideo2 嵌入 + MLP，用于排除抽象图案/游戏画面/动画（见 domain_distribution）。
【未覆盖】水印/logo 检测、黑边检测（cropping 为独立 stage 但未说明是否自动检测黑边）、压缩伪影检测、模糊度单独判据，官方文档均未提供对应 stage。被淘汰的 clip 会被移入 video.filtered_clips 并更新统计，便于事后审计漏斗——这是良好的可观测性设计。

### [OmniHuman 数据集 + OHBench](../models/OmniHuman.md) ⚠️

视觉质量过滤集中在第一、三级，是流水线中工具最密集的部分，采用明确的「由轻到重」两段式：
【美学与画质：轻重两段】
- 轻量段：帧级 CLIP aesthetic scoring（基于 CLIP 特征的美学预测器，即 LAION-Aesthetic 类方法），逐帧快速打分，用于「快速剔除低质片段」。优点是成本极低可全量跑，缺点是只看单帧不看时序；
- 重量段：视频级 DOVER（Disentangled Objective Video Quality Evaluator，把视频质量解耦为美学分支与技术分支的专用 VQA 模型），评估构图（composition）与清晰度（clarity）。DOVER 计算成本远高于 CLIP 打分，故置于其后只处理幸存样本。
这一「CLIP 粗筛 + DOVER 精筛」的两段结构是成本-效果权衡上较为成熟的做法。
【文字污染：检测 + 裁剪 + 密度过滤三重处理】这是本流水线相较同类工作最细致的一环，分布在两级：
- 第一级：用 OCR 与 logo 检测算法估计「文字污染区域」（text-contaminated regions），随后对帧做裁剪（frame cropping）以去除字幕与台标——注意这是「裁掉」而非「丢弃整条视频」，最大限度保留了数据；
- 第三级：基于 OCR 的文字密度分析（text-density analysis），对裁剪后仍然文字过多的样本做整体淘汰。
三种策略的对比很有代表性：UniTalking 是「见文字即丢弃」（最浪费）、MOVA 是「不处理但标注为可控属性」（最省数据但把噪声留给模型）、OmniHuman 是「能裁则裁、裁不掉再丢」（居中且工程量最大）。对 YouTube 这种字幕烧录极普遍的数据源，裁剪策略的数据保全价值相当可观。
【水印检测】第三级中明确包含 watermark detection，与 OCR 文字密度分析并列。具体方法与判定标准未说明。[不确定]
【分辨率门槛】高清及以上（见 resolution_aspect_distribution）。
【人脸清晰度：单列指标】帧级标注中包含人脸清晰度分数，定义为 Cs = Var(ΔR)，即基于拉普拉斯算子响应的方差（Laplacian variance）——这是经典的图像模糊度度量。值得注意的是，该分数被作为标注字段保留而非仅用于过滤，意味着下游可按人脸清晰度自行筛选子集，比一刀切过滤更灵活。
【全部阈值缺失】CLIP 美学分阈值、DOVER 分数阈值、文字密度阈值、人脸清晰度阈值全部未给出数值。[不确定]
【未做】黑边检测与裁除未提及；曝光/色偏检测未提及。

### [Open-Sora 系列](../models/Open-Sora.md)

【美学】两者都用 LAION 系的 CLIP+MLP 美学打分器（improved-aesthetic-predictor，在 176K SAC + 15K LAION-Logos + 250K AVA 图文对上训练，输出 1–10 分）。Open-Sora 文档给出的经验刻度：5.5 为「尚可」阈值，6.5 为「高美学」阈值，好的文生图模型可达 7.0+；视频取首/中/尾三帧评估；代码示例过滤命令为 `python -m tools.datasets.datautil meta.csv --aesmin 5.0`。Open-Sora 1.2 对 Panda-70M 的筛选阈值为**美学分 > 4.5**（得到 20M 子集 / 41k 小时）；图像侧 LAION 子集用**美学分 > 6.5**。Open-Sora Plan 用 **LAION Aesthetic v2 ≥ 4.75**（论文明确说明选 4.75 是因为它同时能顺带滤掉大量含过多文字的画面），且对**美学分 > 6.25 的样本在 caption 前自动加上 「A high-aesthetic scene」 前缀**——把美学分转化为可控条件而非仅作过滤。Open-Sora 2.0 的美学分分布统计显示训练数据主体落在 **4.5–5.5**（即刻意保留了大量「中等美学」样本以保证多样性，而非只留高分样本）。
【清晰度/模糊】Open-Sora 2.0 用 **OpenCV Laplacian 算子的方差**做模糊检测，视频取五个均匀间隔帧分别判定后多数表决；Open-Sora Plan 用 **DOVER 的 technical 分支预测分（阈值 ≥ 0）**，专门针对压缩伪影与低码率片源，比单纯的模糊算子更贴近真实画质退化。Open-Sora 2.0 另在预处理阶段用 **bpp（bits per pixel）< 0.02** 直接剔除低码率片源。
【OCR / 文字过滤】三条不同路线：Open-Sora 1.x 用 **DBNet++（MMOCR 实现）**，输出列 `ocr` 记录**检测置信度 > 0.3 的文字区域数量**，用于剔除新闻播报、广告等密集文字场景；Open-Sora 2.0 用 **PaddleOCR**，只统计置信度 > 0.7 的文字，文字过多则丢弃；Open-Sora Plan 用 **EasyOCR 只检测画面底部 20% 区域**（字幕的典型位置）并采取**裁剪而非丢弃**的策略（edge ratio 0.20），论文同时坦承该方法的局限——无法处理画面中央的广告字、演讲字幕等。
【黑边/水印/logo】三者均**未实现**专门的黑边检测、水印检测或 logo 检测模块，这是共同缺口；Webvid-10M 的 Shutterstock 水印问题因此未被处理。

### [Ovi](../models/Ovi.md) ⚠️

视觉质量过滤由三个显式模块构成，均在 pipeline 第 1 步：
(1) 分辨率门槛：片段分辨率必须大于 720×720 像素，硬性剔除低清素材。Ovi 1.1 提升到原生 960×960 数据。
(2) 美学评分：使用 LAION 的 aesthetic predictor（Schuhmann, 2022，即 LAION-Aesthetics 美学打分模型）剔除低质量数据（remove low-quality data）。具体阈值未公开[不确定]。
(3) 运动质量：RAFT 光流剔除静态片段（详见 motion_filtering）。
(4) 结构性清理（在 packing 阶段而非过滤阶段）：显式去除视频已有的 margins（黑边/信箱边框），再等面积缩放。这是把「黑边」当作可修复瑕疵而非丢弃理由的处理方式。
(5) 内容构成控制：内部人脸检测模型保证单人/多人/无人三类的合理配比（属于分布控制，非质量过滤，但在同一步执行）。
【未提及的常见手段】OCR/字幕文字过滤、水印检测、logo 检测、压缩伪影/模糊检测、亮度对比度过滤、JPEG 块效应检测等，论文均无描述[不确定]。这可能是模型在带字幕素材上的潜在风险点。
【音频侧质量过滤】见 audio_quality_filtering（平均音量 ≥ −60 dB）。

### [Script-a-Video](../models/Script-a-Video.md) ⚠️

论文未披露任何质量过滤细节：无美学评分器（LAION-Aesthetic / DOVER / VQA 类均未提及）、无清晰度或模糊度指标、无 OCR 烧录字幕检测、无黑边检测、无水印/logo 检测、无码率或压缩伪影检测，也未给出任何阈值数值。全部质量控制被压缩为「high-quality video clips」一个形容词。[不确定]
仅有的两条与「质量」相关的间接线索：
1) 数据领域选择本身构成隐式质量筛选——film / television / cinematic pairs 等均为专业制作内容，天然在画质、构图、收音上高于 UGC；
2) 生成侧最终阶段使用的是「60K high-fidelity cinematic alignment pairs」（高保真电影级对齐对），"high-fidelity" 的措辞暗示存在一个更严格的高质量子集筛选，但筛选标准未披露。
本字段的空白与该工作的定位有关：论文的贡献主张在标注范式而非清洗流程，因此把清洗部分整体略去。对于关注打标方式的调研而言这是可接受的取舍，但对关注清洗漏斗的调研而言本条目参考价值极低。

### [Seedance 2.0 与 Seedance 1.5 pro](../models/Seedance_20_Seedance_15_pro.md) ⚠️

Seedance 1.5 pro 仅称管线包含「片段质量」相关的筛选（第三方摘要称第二阶段为音画同步与片段质量的自动化过滤，但该表述不见于论文原文）。Seedance 1.0 的具体做法：(a) 视觉覆盖物矫正——用启发式规则系统 + 专用目标检测模型的混合方案识别 logo、水印、字幕、屏幕图形等遮挡，再对帧做自适应裁剪（adaptive cropping）以最大化保留主体视觉内容；(b) 质量过滤——由专用的「视觉质量模型」（specialized visual quality model）系统性识别并剔除模糊（blurriness）、剧烈抖动（excessive jittering）、低美学质量（low aesthetic quality）、构图/摄影质量差（poor cinematographic composition）、以及以静态内容为主（predominantly static content）的片段。Continue Training 阶段进一步用「一系列专用评估模型，包括美学打分器（aesthetic scorer）」筛出更高美学质量子集。所有阈值、打分器规模与分档均未披露。[不确定：具体阈值与 OCR 文字过滤是否独立成级]

### [SkyReels 系列](../models/SkyReels.md) ⚠️

【SkyReels-V2】质量问题被显式归纳为三大类，并为每类配置过滤器矩阵：
(1) 基础质量问题：低分辨率源、低帧率、黑屏/白屏/静止画面、相机抖动；
(2) 视频类型问题：监控录像、游戏录制、动画、无意义内容（整类剔除）；
(3) 后处理伪影：字幕、台标/logo、图像编辑痕迹、分屏（split screen）、黑边（black border）。
对应过滤器包括：黑屏过滤器、静止画面过滤器、美学过滤器（aesthetic filter）、OCR 过滤器（检测画面文字/字幕）、马赛克过滤器、特效与贴纸过滤器；另使用 VQA（视频质量评估）模型、IQA（图像质量评估）模型与 VTTS 等模型打分。对可挽救样本采取「裁剪而非丢弃」的策略——对字幕与台标区域做 subtitle/logo cropping，以最大化可用画幅面积，这是提高数据利用率的实用设计。
【SkyReels-V4】过滤维度收敛为三个：基础质量（时长、分辨率、美学分、模糊度、对比度、曝光）、内容质量（水印、logo、文字叠加、合成伪影 synthetic artifacts）、运动质量。其中「合成伪影检测」用于剔除由 AI 生成或经过重度后期的内容，是2026年新出现的过滤维度。两代均未公布美学评分模型的来源（自研或 LAION-Aesthetics 类）与各过滤器的阈值数值。[不确定]

### [Sora 2](../models/Sora_2.md) ⚠️

仅有「rigorous filtering to maintain data quality」一句笼统表述。美学评分模型、清晰度/模糊度判据、OCR文字/字幕过滤、黑边裁剪、水印与logo检测等具体手段，OpenAI 均未提及，无任何阈值或模型名称。[不确定]

### [SpeakerVid-5M](../models/SpeakerVid-5M.md) ⚠️

五维阈值化过滤，全部阈值公开，另设更严的 HQ 阈值组：
【1. 亮度（Luminance）】luminance score < 10 或 > 210 的 clip 被剔除（灰度 0–255 区间的两端），用于排除过暗与过曝画面。
【2. 视频质量（DOVER）】使用 DOVER 视频质量评估工具的融合分（fused score），< 0.25 的剔除。
【3. 清晰度（Clarity Score）】自定义指标，定义为 clarity = B/(W×H)，即码率（bitrate）除以分辨率面积——衡量单位像素上的比特密度，可有效识别「高分辨率但实为低清上采样/重压缩」的伪高清视频。按分数排序后剔除最低 5%。这是本数据集较有特色的一个自定义质量指标。
【4. 运动模糊（Motion Blur）】对人脸与手部区域各裁 128×128 patch，计算 Laplacian 方差作为清晰度/模糊分；人脸或手部的平均模糊分 < 0.1 的剔除。分人脸与手部两路独立打分，是为下游手势生成任务专门设计的。
【5. 音频质量】三条并列否决规则，满足任一即剔除：Whisper 转写置信度 < −1.5；no-speech probability（无语音概率）> 0.8；compression ratio（压缩比）> 2.5（用于识别 ASR 重复退化/幻觉输出）。
【HQ / SFT 子集阈值（更严）】手部模糊分 > 0.5、人脸模糊分 > 0.7、DOVER > 0.6、运动强度分 > 2、ASR 置信度 > −1。五条全部满足才入选，得到 571K clips / 1,368 小时。
【未做的过滤】论文未描述美学评分（aesthetic score）单独打分、OCR 烧录字幕检测与过滤、水印/logo 检测、黑边检测。DOVER 本身含美学视角分支，但论文只使用其融合分而未拆分使用。[不确定]

### [Step-Video-T2V](../models/Step-Video-T2V.md)

第2阶段对每个片段的抽样帧打 7 类质量标签，工具与方法几乎全部点名（阈值数值一律未公布，只说明「逐级抬高阈值」构造 6 个子集）：
1. 美学分（Aesthetic Score）：LAION CLIP-based aesthetic predictor（开源 LAION 美学预测器），评估画面视觉吸引力；
2. NSFW 分（NSFW Score）：LAION CLIP-based NSFW 检测器，基于 CLIP ViT-L/14 的二分类器；
3. 水印检测（Watermark Detection）：基于 EfficientNet 的图像分类模型，判定帧内是否含水印；
4. 字幕/文字检测（Subtitle Detection）：PaddleOCR 识别帧内文字，用于剔除字幕/文字过多的片段；
5. 饱和度分（Saturation Score）：在 HSV 色彩空间统计饱和度的均值/最大值/最小值，剔除过曝、过灰或过饱和的异常画面；
6. 模糊分（Blur Score）：Laplacian 方差法（Laplacian variance）度量画面锐度，剔除失焦/模糊片段；
7. 黑边检测（Black Border Detection）：FFmpeg 检测黑边尺寸，据此做裁剪（裁剪而非丢弃，与 HunyuanVideo 1.5 的思路一致）。
方法论特点：这套质检器全部是「传统 CV 算子 + 开源小分类器」的组合，无一为大模型语义判断，属于典型的 2024–2025 年初「浅层多打分器」范式；优点是工具全部点名、成本低、可复现性描述好，缺点是无法捕捉语义层面的质量问题（如物理不合理、主体畸变）。

### [UniTalking](../models/UniTalking.md) ⚠️

视觉质量过滤集中在漏斗第一级，论文用一句话概括为剔除「静态、含文字叠加、整体视觉质量低」三类视频，无任何模型名称与阈值数值：
【静态画面剔除】剔除 static 视频。未说明判定方法（光流？帧差？运动分数模型？）与阈值。这是全文唯一涉及运动的过滤条件。[不确定]
【文字叠加剔除】剔除含 text overlays 的视频。这是一条较有针对性的规则——说话人视频中烧录字幕、弹幕、台标极常见，若不剔除会导致模型生成带乱码文字的画面。但论文未说明用 OCR 检测器（PaddleOCR/CRAFT 等）还是其他方式，也未说明是「有任何文字即剔除」还是「文字面积占比超阈值才剔除」。值得对比的是 MOVA 的做法：不剔除而是标注为可控属性（「This video has no subtitles.」），UniTalking 则选择直接淘汰。[不确定]
【整体视觉质量低剔除】low overall visual quality，表述极其笼统。未说明是用 DOVER、LAION-Aesthetic、MUSIQ 还是其他评估器，也未说明是否包含清晰度、模糊度、压缩伪影、亮度等子维度。[不确定]
【未做的过滤】无水印/logo 检测、无黑边检测与裁除、无人脸尺寸/占比下界约束（虽然任务高度依赖人脸，但未见显式的人脸大小门槛）、无曝光/色偏检测、无独立的美学评分。
【上游继承的质量保障】OpenHumanVid 部分已在其自身流水线中经过基于亮度（luminance）、模糊度（blur）、美学（aesthetics）、运动（motion）与技术指标（technical）的多维质量过滤，且额外做了人物质量过滤与 caption-视觉一致性过滤（用 MiniCPM/CogVLM/Llama 生成结构化 caption 并经 BLIP2 投票精修）。因此 UniTalking 在 OpenHumanVid 子集上的实际质量把关是「上游细致 + 本级粗放」的叠加；而华为内部数据只经过本级的粗放过滤，两个来源之间存在质量把关强度不对等的潜在风险，论文未讨论。[不确定]

### [UniVerse-1](../models/UniVerse-1.md) ⚠️

视觉侧质量过滤集中在漏斗第 2 级，共三条硬阈值，全部数值公开：
【分辨率】低于 1080p 的视频剔除。门槛在同类工作中偏高（MOVA 统一规整到 720p 后训练甚至在 360p 起步），体现小规模数据下的高选择性。
【码率-分辨率比】bitrate-to-resolution ratio 低于 600 的剔除。用于识别「分辨率虚高但实际压缩严重」的素材，是本工作较有辨识度的一条指标，同类工作中少见。
【美学/技术质量】DOVER 综合质量分低于 0.6 的剔除。与 MOVA 拆分 DOVER-Aesthetic（>0.85）与 DOVER-Technical（>0.05）双分数不同，UniVerse-1 只用单一 DOVER 总分且阈值为 0.6，粒度更粗。
【阈值确定方法】论文未说明阈值是如何标定的（既未提人工抽检，也未提消融实验），属经验设定。[不确定]
【未做的过滤】没有 OCR 烧录字幕检测（MOVA 有，且做成了可控属性）、没有水印/logo 检测、没有黑边检测与裁除（未提及 cropdetect 类处理）、没有独立的美学评分器（如 LAION-Aesthetic）、没有清晰度/模糊度专项指标。
【低质量数据的替代处理】对无法通过上述过滤的 VGGSound/AudioSet，不做过滤而改用训练侧的 LQLS 策略隔离——仅当扩散 timestep t > 800（总步数 1000）时才对这两个数据集施加 Flow Matching 损失，即只让它们参与高噪声段（粗结构/语义层）的学习，不参与低噪声段（高频细节层）的学习，从而避免模型过拟合到其低质量视觉的高频伪影。这是把数据质量问题从「过滤维度」转移到「损失加权维度」的典型做法。

### [Unison](../models/Unison.md) ⚠️

论文未描述任何视觉质量过滤环节：无美学评分门槛、无清晰度/模糊度指标、无 OCR 烧录字幕检测、无水印/logo 检测、无黑边检测与裁除、无分辨率或码率门槛、无压缩伪影检测。全部相关信息为空白。
【为何空白——一个结构性解释而非疏漏】Stage 2 联合训练时视频骨干 Wan2.2-5B 完全冻结（keeping the video backbone frozen），仅训练音频分支与融合模块（双向交叉注意力与 layer normalization）。这意味着训练数据的视觉质量不会通过梯度影响视觉生成能力——低质视觉样本最多只会干扰跨模态对齐的学习，其危害远小于需要训练视觉分支的工作。因此 Unison 有理由把过滤预算全部投向音频与对齐维度，而非视觉质量。这一点在与 UniVerse-1 的对比中尤为清楚：后者同样使用低视觉质量的 VGGSound，但因需训练视觉分支，不得不专门设计 LQLS 损失隔离策略；Unison 用了同样的 VGGSound 却无需任何对应处理。
【质量保障实际上被前置到了数据源选择上】Unison 不做质量过滤，但其选材本身即高质量导向：VFHQ 以「high-fidelity（高保真）」为构建标准，HDTF 以「High-Definition（高清）」命名，OpenHumanVid 自称「high-quality」。三个数据源在发布时就已完成各自的质量筛选，Unison 直接继承了这些筛选结果——这是「聚合型数据构建」的一个天然优势，也解释了为何论文用「high-quality audio segments」形容其音频语料却不说明筛选依据。
【唯一可确认的质量相关处理在音频侧】Mel-RoFormer 分离产出被形容为「high-fidelity speech and sound-effect components」，但这是分离质量而非筛选门槛。
【评测侧使用但未用于过滤的质量模型】LAION-Aesthetic Predictor V2.5（视频美学分 VA）、DINOv3（帧间身份一致性 ID）、Audiobox-Aesthetics（音频 PQ/CU）——这三者均仅用于评测，未见用作训练数据过滤器。这是一个值得注意的错位：团队显然掌握这些评分工具，却未将其用于数据筛选。[不确定]

### [Veo 3 / Veo 3.1](../models/Veo_3_Veo_31.md) ⚠️

官方唯一表述为「Training videos were also filtered for various compliance and safety metrics and for quality」，即确认存在质量过滤这一环节。[不确定] 具体手段完全未披露：是否使用美学评分模型（aesthetic scorer）、清晰度/模糊度指标、OCR 文字覆盖率过滤、黑边检测、水印/logo 检测、压缩伪影检测等均无说明，也无任何阈值数值。可作弱反推：技术报告承认 Veo 3「在生成文字上仍然较差」，若数据端做过激进的 OCR 文字过滤，会进一步削弱文字渲染能力，二者在方向上是一致的但不能作为证据。

### [Vidu S1](../models/Vidu_S1.md) ⚠️

多维度、专家模型 + 大模型混合的质量过滤，共六个关注点：
(1) 主体检测（subject detection）：每个 clip 恰好含一个主体且在画面中占比合理；
(2) 画面干净度（frame cleanliness）：剔除含与视觉内容无关信息的片段，明确列举水印（watermarks）、字幕（subtitles）、贴片/叠加广告（overlaid advertisements）；
(3) 视觉质量（visual quality）：使用美学评分（aesthetic scoring）与技术评分（technical scoring）双评分，筛选清晰、完整、观感良好的片段，规避模糊（blur）、抖动（jitter）、闪烁（flicker）等伪影；
(4) 内容安全：剔除 NSFW 及其他不当内容；
(5) 镜头稳定性（shot stability）：保留静止镜头或慢速运动镜头，以降低长时流式生成中的镜头漂移（shot drift）风险；
(6) 交互性（interactivity）：要求主体有清晰的动作/行为。
未公开任何评分器的具体名称与阈值数值 [部分不确定]。未提及 OCR 文字检测的具体实现、黑边检测。

### [Wan 2.5 / 2.6 / 2.7](../models/Wan.md) ⚠️

Wan 2.1 的质量过滤是全行业公开材料中枚举最完整的之一（2.5+ 未披露，推测沿用并升级）。
【基础维度并联检测（逐项原文级还原）】
1) 文字检测（Text detection）：轻量 OCR 检测器量化「文字覆盖率」，剔除文字元素过多的视频与图像以保持视觉清晰度；
2) 美学评估（Aesthetic evaluation）：使用业界通用的 LAION-5B 美学分类器做初步质量评估，快速滤除低质数据；
3) NSFW 打分：内部安全评估模型对全部训练数据算 NSFW 分并过滤；
4) 水印与 logo 检测：检测是否含水印/logo，并在训练时对这些区域做裁剪（注意是 crop 而非丢弃，属「可回收」处理）；
5) 黑边检测：启发式方法自动裁掉多余黑边，聚焦内容富集区；
6) 过曝检测（Overexposure detection）：自训练专家分类器识别并剔除色调分布异常的数据；
7) 合成图检测（Synthetic image detection）：自训练专家分类器剔除 AI 生成图像。报告给出了一条极有价值的经验性结论——「即便极小比例（<10%）的生成图污染，也会显著劣化模型性能」，这是数据纯净度重要性的直接量化断言；
8) 模糊检测（Blur detection）：自研模型给训练素材打模糊分，系统性移除视觉不清晰内容；
9) 时长与分辨率：时长必须 >4 秒；分辨率阈值随训练阶段动态提高。
【视觉质量学习式打分】从每个簇中抽样交人工按 1–5 分打分（1最差5最好），用全部标注数据训练一个专家评估模型，再对全量数据打分，用于各阶段的数据选择与处理。
【Wan2.2 增量】引入「精心策划的美学数据，带有光影（lighting）、构图（composition）、对比度（contrast）、色调（color tone）等细粒度标签」，使模型可按电影美学风格可控生成——这是从「一个美学总分」到「多维可控美学标签」的重要升级，也是 2.5+「电影级」定位的数据基础。
【Wan2.2-S2V 五项视频质量指标】Dover 清晰度评估、UniMatch 光流运动稳定性（滤除主体/背景运动过剧者）、Laplacian 算子专门检验人脸与手部区域是否模糊、改进版美学预测器、OCR 检测字幕是否遮挡人脸或手部。其中「对人脸/手部单独做锐度检验」与「字幕遮挡人脸检测」两项针对人本场景的设计最值得借鉴。

### [音视频生成评测基准合集](../models/av_benchmarks.md) ⚠️

各基准的质量把关手段：
【AVBench】质量维度本身即是可复用的过滤器，其 6 项单模态质量维度分别绑定成熟工具：音频质量用 NISQA MOS、音频美学用 Audiobox、视频技术质量用 DOVER++、视频美学单列一维，另加 Speech Content Accuracy 与 Speech Realism 两项语音专项。论文明确指出其连续可微分打分可直接用作数据过滤机制与 RLHF 奖励信号 —— 这是本次调研中最直接可移植为训练数据质量过滤器的基准。
【VABench】Module 1 的单模态音频质量三件套：SpeechClarity 用 DNSMOS 检测背景噪声、SpeechQual&Nat 用 NISQAv2 评整体质量与自然度、AudioAesthetic 用 Audiobox 评愉悦度/有用性/制作复杂度与质量。视觉侧质量并入 MLLM 打分的 Visual Realism 与 Artistry。此外立体声专项提供 9 项声学质量指标：空间成像质量（立体声宽度、成像稳定性、电平稳定性、声道间一致性）与信号完整性/兼容性（相位相干性分低/中/高三频段评估、单声道兼容性 Mono Compat = 1 − 归一化单声道损失、方向一致性、Mid/Side 能量比测声场宽度）。
【AV-SyncBench】人工阶段明确剔除「音质差、噪声过大、语义模糊」三类样本，即音频 SNR、清晰度与语义可判别性三重门槛（未给出量化阈值[不确定]）。
【PhyAVBench】质控聚焦物理正确性而非画面美学：LLM 初筛语义歧义与非预期物理混杂因素，人工复核物理一致性与现实合理性；因是可控环境自录，画面清晰度、水印、黑边等问题天然不存在。
【Omni-Judge】不做过滤；但其结论对质量过滤有警示意义 —— Omni-LLM 在 video quality 维度上与人类的相关性极低（Kendall τ_b ≈ 0.020），说明用 Omni-LLM 直接替代传统美学/技术质量打分器做数据过滤是不可靠的，画质类过滤仍需专用模型。
未提及 OCR 文字过滤、水印/logo 检测等训练侧常见手段[不确定]。

### [视频 Caption 模型生态](../models/caption_models.md)

captioner 生态的「质量过滤」有两个方向：
【方向一：captioner 作为质量过滤器的组成部分】
· Allegro：Tag2Text 输出作为 CLIP 相似度过滤的文本侧输入，打标器直接嵌入过滤链。
· Mochi/MAGI/Motif 批次中的 Motif-Video 2B：另配 PaddleOCR-VL（经 vLLM 服务化）做帧上文字检测，属于打标链中的 OCR 过滤分支。
· InstructAV2AV 用 Grounded-SAM-2 提供实例级 mask 锚点，TalkNet 做主动说话人检测 —— 结构化标注工具兼任质量闸门。
· 传统浅层过滤器（LAION 美学分、DOVER technical score、清晰度、黑边/水印/logo 检测）与 captioner 并行而非串行，通常在打标之前完成以节省打标算力。
【方向二：captioner 输出本身的质量控制】这是本生态的核心问题，主流手段有四：
(1) LLM 打分过滤：AVoCaDO 用 GPT-4.1 打 synthesis completeness 1–5 分、只保留 ≥4；
(2) 自动化一致性校验：AVSCap 的 automated verification（tag 保留检查 + 语义一致性检查）；
(3) 检索式择优：Panda-70M 用 UMT-large（ViT-L/16 + BERT-large，VTC+VTM 双损失 + 难负例挖掘，未选中的 7 条 caption 权重 1.0、batch 内其他负例权重 0.01，12 帧 224×224，AdamW lr 2e-5，batch 32，10 epoch，8×A100-80G）从多候选中选最佳 caption，微调后 R@1 达 35.90%（预训练 UMT 仅 21.82%）；
(4) 幻觉抑制的后训练：腾讯混元 1.5 对其三个 caption 模型用 OPA-DPO（针对多模态幻觉的偏好优化）做 RL 后训练；video-SALMONN 2 的 MrDPO 同时奖励完整性与事实性，7B 模型 caption 错误率相对基线降 28%；Tarsier2 用 model-based sampling 构造偏好对做 DPO。
【生成侧的兜底】Step-Video-T2V 不对 caption 模型做幻觉抑制专门后训练，兜底依赖 SFT 阶段人工复核 + 第 6 阶段 CLIP Score 对齐过滤 —— 代表了「打标器不够好就用下游过滤补」的务实路线。

### [几何/结构化标注数据集合集](../models/geometric_datasets.md)

SceneScribe-1M：核心是 Qwen2.5-VL-72B 的六维内容质量评估，显式剔除运动强度不明、可见水印、强相机畸变、强光照伪影等clip；叠加硬性规格门槛（>1080p、≥10fps、5s–60s）。SpatialVID：四项独立打分器构成质量层——①美学：CLIP+MLP aesthetic predictor，分数 < 4.0 剔除；②亮度：仅保留均值落在 [20, 140] 区间的clip，过暗过曝剔除；③OCR文字：PaddleOCR 检测，文字面积占比 > 30% 的clip被标记剔除；④运动：VMAF 指标 + 后续重建可行性判定。WildWorld：引擎渲染画面天然无水印、无压缩伪影、无黑边，质量过滤退化为场景有效性筛选，主要保证录制同步与深度无损编码。Action100M：几乎不做画质过滤（沿用 HowTo100M 原始画质），质量控制转移到标注侧——通过 LLM 多轮 Self-Refine 保证标注质量，并按时长下限（片段>0.5秒、树节点>4秒）剔除无意义碎片

### [视频生成后训练数据策略](../models/post_training_data.md) ⚠️

【锚论文】SFT 数据的质量过滤标准完全未披露 [不确定]。质量控制被完全推给了 RLHF 阶段的四个奖励模型（视频美学：光照、构图、色彩和谐、时序一致性、跨帧影视感；图像美学：帧级感知质量、锐利细节、结构悦目；运动质量：运动真实性、平滑性、连贯性，抑制抖动/运动不连续/时序不一致的物体转换；文本-视频对齐：语义一致性）。这体现了一种范式转移——把「什么是高质量」从数据过滤阈值搬到可学习的奖励模型里。
【横向的 SFT 质量筛选标准（阈值级公开的少数案例）】
· Allegro（阈值最完整）：时长 6–16 秒、分辨率 ≥1280×720、亮度 [20,180]、DOVER ≥0.07、LPIPS ≥0.05、UniMatch 运动 [1.0,100]、美学 ≥5.3、文字面积 ≤0.05%、CLIP 相似度 ≥0.20；
· HunyuanVideo 原版：人工精选 100 万，标准为美学四项（色彩和谐、光照、主体突出、空间布局）+ 运动三项（运动速度、动作完整性、运动模糊）——这套七维人工标准是本主题被引用最多的 SFT 筛选 rubric；
· HunyuanVideo 1.5 SFT：在 CT 数据之上按美学吸引力、清晰度、运动流畅性三项再严筛，最终经人工标注构建；
· CogVideoX：针对「字幕、水印、低码率」三类脏数据筛出 top 20%；
· Step-Video-T2V：三步走——自动分数与启发式规则过滤 → VideoCLIP K-means 簇内剔除距簇中心超阈值的离群样本（阈值未公布）→ 人工逐条评审（清晰度、美学、运动是否恰当、场景转换是否平滑）并优化 caption；
· LongCat-Video：第一层多指标（美学分、视频质量、运动质量）+ 第二层 caption 嵌入空间密度倒数采样；
· Motif-Video 2B：常规过滤 + 更严美学截断 + style/subject 标签驱动的 domain-balancing + action=Dynamic；
· NAVA：多算子协同过滤，标准为「caption 准确 + 音视对齐强」，阈值未公开。
【共性】SFT 筛选 = 预训练阈值 + 显著上调的美学/清晰度门槛 + 概念均衡 + 人工终审。人工终审出现在 HunyuanVideo（原版与 1.5）、Step-Video-T2V、Movie Gen、SkyReels-V4、Apollo Stage III 等几乎所有工业级工作中，是 SFT 区别于预训练过滤的标志性环节。

### [主流视频预训练数据集合并调研：Panda-70M、InternVid、Koala…](../models/pretraining_datasets.md) ⚠️

**按「美学 / 清晰度技术质量 / 文字OCR / 黑边水印」四类横向对比**：
【美学评分】
- **Panda-70M：无。** 这是其最大缺口。
- **InternVid**：LAION 美学预测器，**≥4** 得到 InternVid-18M-AES 子集（仅用于派生子集，非主漏斗）。
- **MiraData**：LAION-Aesthetic 预测器，在统一 **2fps** 下计算，**阈值未公开**（论文称在补充材料，补充材料不存在）。[不确定]
- **OpenVid-1M**：LAION 美学分，**Panda-50M 源取前20%，其余三源取前90%**——用百分位而非绝对分。
- **LVD-2M：不用美学打分器**，改由 PLLaVA-7B 的「视觉多样性」prompt 做语义判断（全文「aesthetic」仅作为VBench指标名出现一次）。
- **Koala-36M**：美学分是 VTSS 的输入子指标之一（CSV中 aesthetic_score 实测范围 **2.28–6.56**），但**不单独设阈值**。
- **UltraVideo：不用 LAION 美学分也不用 DOVER**，直接复用 **Koala-36M 的 VTSS**（原生分线性缩放到 −0.0575~0.0728），**VTSS<0.01 剔除**。
【清晰度 / 技术质量】
- **OpenVid-1M**：**DOVER technical 分取前30%**。
- **Koala-36M**：clarity_score 作为 VTSS 子指标（CSV实测范围 0–1）；论文用 DOVER 与 FastVQA 作为 TSA 网络的对比基线（TSA PLCC 0.8974 vs DOVER 0.8554 vs FastVQA 0.8684）。
- **MiraData**：**「画面色彩」检查**——计算平均色彩以及最亮/最暗80%帧的色彩，剔除过亮/过暗素材。这是七者中唯一显式的曝光/色彩层面过滤（与UltraVideo的过曝检查同类）。
- **Panda-70M / InternVid / LVD-2M：无专门的清晰度/技术质量打分器。**
- **UltraVideo：四项统计检查，全部采用「坏帧率>5%则丢弃整条视频」的统一模式**——(a) 黑边：从四边各向内延伸**3%**的矩形区域均值**<3** 判坏帧；(b) 过曝：像素值 **>250 或 <5** 的比例**>12%** 判坏帧；(c) 灰度化：逐像素RGB方差的全图均值 **<1.2** 判坏帧。这是七者中**唯一给出全部绝对阈值数值的统计过滤方案**，可直接复用。
【OCR / 文字过滤】
- **UltraVideo**：**PaddleOCR**，文字最小外接矩形并集面积 / 图像总面积 **>2%** 判坏帧，坏帧率>5%丢弃。
- **LVD-2M**：**不用OCR模型**，由 PLLaVA-7B 的第二个prompt判断「Text Presence: 文字叠加是否主导画面以致损害观感」。
- **Panda-70M / InternVid / Koala-36M / MiraData / OpenVid-1M：均无OCR文字过滤**（Koala 论文全文无 OCR，与常见猜测相反；OpenVid 未披露）。
【黑边 / 水印 / logo / 转场特效】
- **UltraVideo（唯一系统处理）**：黑边有专门的统计检查（见上）；水印、转场特效、分屏、屏幕录制、画中画等**16种低质属性**交由 **Qwen2.5-VL-72B 逐项二元判断，命中任意一项即删除整条**。另在源头做过一次人工复检剔除水印/模糊/抖动。
- **Koala-36M**：字幕、logo、特效、转场仅作为**人工标注员的评分指引**（「视频自然度」维度），通过200K标注间接、有损地传递给 VTSS，**没有独立的检测器**。
- **其余五家：均无黑边/水印/logo检测。** 其中 LVD-2M 使用了 WebVid 素材（带 Shutterstock 水印且该数据集已因版权下架），是明确的水印污染风险来源。
**一条重要的反直觉发现**：被广泛认为「必备」的美学打分+OCR组合，在最新的两个数据集（LVD-2M、UltraVideo）中正在被替代——LVD-2M 用7B VLM 全面替代，UltraVideo 用「精确统计阈值 + 72B VLM 属性判断」的组合替代，**都不再依赖 LAION 美学分**。

## 运动过滤（光流/运动分数阈值、静态与抖动剔除）

`motion_filtering` · 详细程度: brief

### [Allegro](../models/Allegro.md)

使用 UniMatch（Xu et al., 2023）光流模型估计运动幅度，仅在 T2V 微调阶段启用，保留区间 [1.0, 100]：下界 1.0 剔除近乎静止的画面（静态图、幻灯片式素材），上界 100 剔除运动过于剧烈/抖动/快速转场的样本。预训练三个阶段不设运动门槛（Table 1 中为空），说明团队将运动质量视为「后期精修」而非「预训练准入」条件。
此外 LPIPS ≥0.05 的下界与运动过滤形成互补，同样用于排除帧间几乎无变化的样本。
未使用相机运动分类器或抖动专项检测模型。

### [Apollo](../models/Apollo.md) ⚠️

存在运动相关的过滤维度，但无方法与阈值：论文将「subject motion ratio（主体运动比例）」与「camera stability（镜头稳定性）」列为动态质量（dynamic quality）建模的两个子维度，纳入第一道视频过滤。这意味着静态呆板片段（运动比例过低）与手持抖动/剧烈晃动片段（镜头稳定性差）在设计上都会被处理，方向与常规做法一致。但论文未说明是用光流（RAFT/UniMatch）、帧差还是专用模型计算，也未给出运动分数的上下界阈值。评测环节使用 RAFT 光流计算 Motion Score（MS），但这属于评测指标，论文未说明数据过滤是否复用同一工具。[不确定]

### [CineDance / CineDance-1M](../models/CineDance.md) ⚠️

【指标】使用 AMT 计算 Motion Smoothness（运动平滑度）作为运动维度的量化刻画，与 DOVER 的美学/技术分并列存入元数据。
【策略】同样遵循「不硬剪枝」原则，运动分数不设过滤阈值，仅作为元数据供下游按需筛选。
【间接的静态剔除】人工伪影审计的目标清单中包含「静帧停留（still-frame holds）」与「转场特效（transition effects）」，属于对静止/异常运动内容的定性把关，但非基于光流阈值的自动过滤。
【未采用】论文未提及光流（optical flow）计算、运动幅度阈值、抖动（camera shake）检测等常见运动过滤手段，也未给出任何运动分数的分布或阈值数值。这是相对其他大规模数据集较弱的一环。[不确定]

### [CogVideoX](../models/CogVideoX.md) ⚠️

两个手段并行：
· 光流分数（optical flow score）：对全部训练视频计算，并在训练过程中动态调整阈值，目的是「确保生成视频的动态性（ensure the dynamic ... quality of generated videos）」。具体阈值数值未公开 [不确定]。
· Video-LLaMA 静态/连通性分类器（Classifier-Static，对应负面标签 Lack of Motion Connectivity，测试准确率 0.92）：剔除动态信息极少、或转场处运动不连贯（人工拼接、静态图剪辑）的片段。
· 「Lecture Type」分类器（Acc 0.99）在效果上也承担剔除近静止画面的作用。
· 抖动处理走的是画质线而非运动线：相机剧烈抖动被归入 Low Quality 负面标签（「excessive camera shake」）由 Classifier-Low Quality 剔除，而非像 Movie Gen 那样用「每秒镜头数」做抖动代理指标。
· 论文未使用 motion vector、VMAF motion score 等 FFmpeg 侧指标。

### [Cosmos-Predict2.5](../models/Cosmos-Predict25.md) ⚠️

运动过滤是过滤链的第二级：「we apply a motion filter, which quantifies and removes clips based on their degree of motion」——量化每条 clip 的运动程度并据此剔除。论文未说明度量方式（未指明光流、帧差还是模型打分），也未给出阈值，更未说明是双侧过滤（同时剔除近静止与剧烈抖动）还是仅剔除低运动一侧。
可确认的相关证据有三处：(1) 机器人域预处理明确「filtered out low-resolution and near-static videos」——剔除低分辨率与近乎静止的视频，说明至少存在低运动侧的剔除；(2) 机器人域还做了运动节奏归一化——对机械臂动作过慢的视频提高播放速度（increased the playback speed），以保证跨数据集的动作节奏一致，这是一种少见的「主动改写运动速率」而非单纯过滤的做法；(3) 后训练 SFT 把 high motion（高运动）单独列为五大域之一并保留 1.0M 条，说明高运动样本是被刻意保留和强化的目标，而非被过滤对象。
[不确定：运动度量方法与阈值、是否过滤高频抖动]

### [Data-Juicer 2.0](../models/Data-Juicer.md) ⚠️

运动过滤是 DJ 少数提供了三种并列实现的环节，反映团队对该维度的重视：
  · video_motion_score_filter —— 基于 OpenCV 的光流估计（Farneback 稠密光流）计算运动分数，按区间过滤。速度快、无需 GPU，是默认选项。
  · video_motion_score_raft_filter —— 基于 RAFT（Recurrent All-Pairs Field Transforms）深度光流模型计算运动分数。精度显著高于传统光流，尤其在大位移与遮挡场景，代价是需要 GPU 推理。
  · video_motion_score_ptlflow_filter —— 基于 ptlflow（PyTorch Lightning Optical Flow）库，可接入其中数十种光流模型（FlowNet、PWC-Net、GMA、FlowFormer 等），提供最大的模型选择灵活性。
【三者并存的意义】使用者可按算力预算与精度需求在「OpenCV 快但粗 → RAFT 准但慢 → ptlflow 可任选」之间权衡，这种同一功能多档实现的设计在数据处理框架中是加分项——大规模粗筛用 OpenCV、精筛用 RAFT 是自然的两阶段用法。
【过滤语义】三者均为双边区间过滤（min_score / max_score），下界剔除静止或近静止画面（视频生成模型若学到大量静态样本会倾向生成「PPT 式」无动感视频），上界剔除运动过于剧烈、抖动、快速剪辑的片段（光流估计失真、时序建模困难）。
【实际使用情况】[不确定] 官方 T2V 案例的最终配方中未包含运动分数算子——在 Probe 阶段的算子排序中，video_aesthetics_filter、video_nsfw_filter、video_frames_text_similarity_filter 三者进入候选，最终胜出的是后两者。论文未报告运动分数算子在该案例中的探测名次或未被选中的原因，因此无法判断是运动维度在该数据集上增益不显著，还是根本未纳入候选。也未公开任何推荐阈值数值。

### [Foley-Omni](../models/Foley-Omni.md) ⚠️

运动过滤通过 motion score 的双边阈值区间 [0.1, 3.2] 实现，是过滤阶段视觉维度三项指标之一。
【下界 0.1】剔除静态或近静态画面。对 V2A/V2ST 任务这一步尤为关键：静止画面无视觉事件，模型无法从中学到「视觉动作→声音事件」的时序映射，这类样本会成为纯噪声监督。
【上界 3.2】剔除运动过于剧烈的片段。抖动、快速摇镜、高频剪辑会使 Synchformer 的时序特征失真，且此类片段的声画对应往往是混乱的（多个事件叠加、声源出画）。
【与文生视频模型的差异】文生视频模型的运动过滤主要服务于「生成有动感的视频」这一目标（剔除静态是为了避免模型生成静止画面）；Foley-Omni 的运动过滤则服务于「保证视听事件可对齐」，出发点不同但阈值形态相似（都是双边区间）。
[不确定] motion score 的具体计算方式（RAFT/UniMatch 光流、帧间差分、还是其他）与归一化尺度均未说明，因此 [0.1, 3.2] 这组数值难以直接迁移复用。

### [Goku](../models/Goku.md)

使用 RAFT 光流模型计算运动分数（motion score），并设置**双边阈值**（既剔除近静止画面，也剔除运动过剧烈/抖动画面），阈值同样按分辨率分三档且逐档收紧：
  - 480p：0.3 ≤ motion score ≤ 20.0
  - 720p：0.5 ≤ motion score ≤ 15.0
  - 1080p：0.5 ≤ motion score ≤ 8.0
设计逻辑：下限剔除幻灯片式静态视频与死画面（保证模型学到动态度，对应 VBench dynamic degree，Goku 得 76.11）；上限剔除手持抖动、快速摇镜、剧烈转场残留等会导致训练不稳定与运动模糊的素材。分辨率越高，上限压得越低（20 → 15 → 8），因为高清素材更倾向于用于精细画质学习而非大幅运动学习。
此外运动分数并未在过滤后被丢弃——它被**复用为 caption 的一部分**（见打标字段），成为推理期可控的运动强度条件，这是 Goku 一个巧妙的设计：同一信号既做过滤门控又做条件控制。

### [Hailuo / MiniMax Video](../models/Hailuo.md) ⚠️

[不确定]（完全无披露）。未提及光流计算、运动分数阈值、静态帧剔除或抖动剔除的任何细节。间接线索：Hailuo 02 与 2.3 都把「复杂人体运动、物理合理性、大动态」作为核心卖点，2.3 进一步强调「即使在动态运镜下也能保持流畅自然精确的复杂肢体动作」，这强烈暗示数据侧存在针对高运动幅度样本的定向筛选或加权（剔除静止片段、保留高动态样本），但没有任何一手说明其方法与阈值。

### [HunyuanVideo-Foley](../models/HunyuanVideo-Foley.md) ⚠️

论文未描述任何运动过滤环节：无光流计算、无运动分数阈值、无静态镜头剔除、无镜头抖动检测。
【为何本工作可以不做】运动过滤在视频生成模型中的作用是保证训练数据有足够的动态信息（避免模型学会生成静止画面）。在 V2A 任务中，「画面动得多不多」并不直接决定音频质量——一段几乎静止的画面（如缓慢燃烧的壁炉）同样对应有意义的持续音效。因此运动强度不是本任务的有效筛选维度。
【真正承担类似职能的是 AV-align】本工作用 AV-align 做时序对齐检测，该方法本身依赖于视觉事件（画面能量突变）与音频事件（音频 onset）的时间轴对应关系。一个完全静止无事件的画面，其 AV-align 分数天然偏低（无视觉事件可与音频事件对应），因此会在第 6 级被间接淘汰。换言之，AV-align 在功能上部分覆盖了「剔除无动态内容」的需求，且比单纯的运动分数更贴合任务——它筛掉的不是「不动的画面」，而是「画面事件与声音事件对不上的样本」，后者才是 V2A 真正需要排除的。这是一个用跨模态判据替代单模态判据的合理设计，虽然论文未如此表述。[不确定：此为推断，论文未说明 AV-align 具有此副作用]

### [HunyuanVideo](../models/HunyuanVideo.md) ⚠️

【原版】用光流估计（optical flow estimation）计算运动幅度，剔除静止（static）与慢动作（slow-motion）视频；SFT 人工标注环节还专门对「运动速度（motion speed）、动作完整性（action integrity）、运动模糊（motion blur）」三项做人工判定，即运动质量同时有自动与人工两道关。
【1.5】在基础过滤级中剔除「静止或低运动场景（static or low-motion scenes）」，未说明是否仍用光流；SFT 阶段的筛选标准之一为「运动流畅性（motion smoothness）」。两代均无阈值数值。[不确定]

### [InstructAV2AV](../models/InstructAV2AV.md) ⚠️

运动过滤通过 CoTracker3（Karaev et al.，Meta 的点跟踪模型）实现，采用网格布局采样点并跟踪（sample points on a grid layout and track them），计算片段的平均运动幅度（average motion magnitude），低于预设阈值的片段被丢弃。
【工具选型的特点】使用点跟踪模型而非传统光流（RAFT/UniMatch）或帧差法来度量运动，这在本调研的对象中较为少见。点跟踪的优势在于给出的是长时程的轨迹一致性度量而非逐帧瞬时运动，更能反映「片段内是否存在持续、可跟踪的运动」，且与下游需求高度契合——编辑任务需要目标在整个 5 秒内可被稳定跟踪与替换，点跟踪的成功与否本身就是可编辑性的代理指标。相比之下光流大小无法区分「持续位移」与「随机抖动」。
【单边阈值】只设下界（剔除静态），未提及上界。这与 Foley-Omni 的双边区间 [0.1, 3.2] 不同——本工作不排斥剧烈运动，因为编辑任务并不依赖运动幅度的温和性；但从局限性中承认的「大幅相机运动下 3D 空间一致性与物体恒常性难以保持」看，缺乏运动上界可能正是该失败模式的数据侧诱因之一，若在数据构造时也剔除大幅运镜样本，模型或许不会暴露这一缺陷（但代价是能力覆盖变窄）。
【剔除静态的动机】与生成模型「避免生成静止画面」的动机不同，此处剔除静态是因为：静止画面的编辑任务过于平凡（等价于图像编辑），无法为模型提供时序一致性的监督信号，也无法检验音视频同步能力。
[不确定] 阈值的具体数值、网格采样密度、运动幅度的归一化方式（像素/帧、还是归一化到画面尺寸）均未披露，因此该设定难以直接复现或迁移。

### [2026 其他音视频联合生成](../models/JAVG_2026_misc.md) ⚠️

披露稀疏，仅两项涉及：
【ALIVE】使用 RAFT 计算光流做运动分析——原文「compute optical flow using RAFT」。RAFT 是本批中唯一被点名的光流模型，与 Ovi、HunyuanVideo 等前作的选择一致，说明 RAFT 已成为该环节的事实标准。但 ALIVE 未说明：运动分数的阈值、是否剔除静止片段、是否剔除抖动/运动过强片段、motion score 是否作为下游条件或课程调度依据[全部不确定]。
【NAVA】把 motion score（运动分）列为视觉质量算子之一，与 aesthetics / sharpness / brightness 并列。计算方法（是否用光流、用何模型）与阈值均未说明[不确定]。
【OmniCustom】无运动过滤——这是合理的，因为其数据全为单人说话头，运动幅度天然小且集中在面部，施加运动过滤反而会误伤有效样本。这提示运动过滤的必要性与 domain 强相关：通用视频生成必须做，talking-head 定制可以不做。
【StreamChar / CCL / Baton】未提及[不确定]。
【ITS-JAVG】不涉及训练侧运动过滤，但其验证器中 VideoReward-TA 隐含对运动质量的评价。
【总体判断】相比 2024–2025 年论文普遍详述运动分阈值与静止/抖动双端剔除，本批 2026 年工作对运动过滤的着墨明显减少，甚至连阈值都不再提及——反映该环节已被视为标准化的「已知操作」，技术叙事重心已转移到音视频对齐与语义打标上。

### [音视频联合生成基线合集](../models/JavisDiT_baselines.md) ⚠️

【JavisDiT++（唯一明确使用）】采用「光流评分（flow scoring）」作为 TAVGBench 数据过滤的三项质量指标之一，遵循 Open-Sora 的实现（Open-Sora 通常用 UniMatch 或 RAFT 计算光流幅值作为 motion score）。用途是剔除运动过少的静态片段。具体阈值、是否也剔除运动过强/抖动的一端、motion score 是否作为训练条件或采样权重，论文均未说明[不确定]。
【MM-Diffusion】无运动过滤。但值得注意的是其数据集选型本身隐含了运动约束：Landscape 的 9 类场景（爆炸、火焰、下雨、水花、雷、瀑布、风等）全部是有持续动态的自然现象，AIST++ 全是舞蹈动作——即通过 domain 选择而非过滤器保证了运动充分性，这是小数据集时代的「以选型代替过滤」思路。
【AV-DiT】无运动过滤。
【Harmony / UniAVGen】均未提及任何运动过滤或光流打分[不确定]。
【共性】除 JavisDiT++ 外无一使用光流工具，这与本合集数据规模小、可人工把控质量有关；一旦数据规模上到百万级（JavisDiT++ 的 110 万条 TAVGBench），运动过滤就成为必需项。

### [可灵 3.0 / 可灵视频 3.0 Omni](../models/Kling_30_Omni.md) ⚠️

Kling-Omni 明确剔除“动作语义密度过低（excessively low action semantic density）”的片段，即静态/近静止画面被过滤；抖动（jitter）在时序质量评估中被单列为剔除项。同团队 Koala-36M 将运动分（motion score）作为 VTSS 网络的输入子指标之一，并以3D Swin 动态分支显式建模时序运动质量，而非单纯依赖光流阈值。[不确定：是否使用光流（RAFT等）计算运动分及其具体阈值]

### [LTX-2](../models/LTX-2.md) ⚠️

pipeline 第3级为「Estimate Motion Level」，并在 Filter Shots 环节据此过滤。论文明确：「我们主动移除运动量不显著（insignificant motion）的视频，以确保数据集聚焦于动态内容」，理由是动态内容更贴合模型的目标能力。未说明运动量估计的具体算法（是否用光流、帧差或其他）、阈值数值，也未提及是否剔除抖动/手持晃动等劣质运动。[不确定]

### [LongCat-Video](../models/LongCat-Video.md) ⚠️

以光流（optical flow）为唯一运动度量：「Motion information is evaluated using extracted video optical flow to assess video dynamics, enabling us to filter out clips with minimal motion features」——提取视频光流评估动态性，据此过滤掉运动特征极少的片段（即剔除近似静止的样本）。报告只提到剔除「低运动」一侧，未提及是否同时剔除高频抖动/剧烈晃动的样本，也未给出光流分数阈值。SFT 阶段另有独立的 motion quality 指标参与精选筛选（该指标由内部微调的 VideoAlign 模型给出，与光流统计量不同，属模型打分）。Avatar 1.5 在线过滤中包含「motion intensity」与「motion consistency」两项检查，以及跳帧检测。[不确定：光流阈值数值、是否过滤抖动]

### [MOVA](../models/MOVA.md) ⚠️

论文未描述任何独立的运动过滤环节：没有光流计算、没有运动分数阈值、没有静态镜头剔除或手持抖动剔除。运动相关的质量控制隐含在两处：(1) DOVER 技术分与美学分本身包含对时域失真、卡顿、模糊等的感知评估；(2) 视觉打标 prompt 中的“LAW OF VISUAL DYNAMICS”要求标注器检测所有转场、精确记录运动轨迹、速度变化与视觉节奏，并规定“当无视觉变化时 visual_description 输出 null”——理论上可据此识别静态片段，但论文未说明是否用该 null 信号做过滤。由于 MOVA 的训练数据以说话人对白为主体，画面运动幅度天然偏小，运动过滤的必要性也相对较低。[不确定]

### [合并条目：① Mochi 1](../models/Mochi_MAGI_Motif.md) ⚠️

① Mochi 1：未披露。官方仅在能力层面强调 Mochi 1 在运动质量上的优势（30 FPS、高时序一致性、真实运动动力学），CEO 也表示团队「focusing heavily on improving motion quality」，但对应的数据侧运动过滤策略无任何说明。[不确定]
② MAGI-1（三者中最细致，且是唯一做前景/背景运动分离的）：
· 运动强度：用 RAFT 光流模型；为降低计算开销先把所有视频下采样到 8 FPS 再计算相邻帧光流；在像素级计算光流，对全片所有像素的流幅值取平均得到整体运动强度。
· 前景/背景分离（关键改进）：论文指出上述做法在「背景静止但前景大幅运动」时会低估运动量；为此额外对每帧运行显著性检测模型（Zhao & Wu, 2019），用显著性图区分前景与背景区域，分别计算两者的平均光流。最终得到三个运动统计量：整体运动强度、前景运动强度、背景运动强度，并对三者各自设定上下阈值，优先保留中等运动强度的片段，同时避开过静与过动两端，以平衡数据质量与训练难度。
· 相机运动稳定性：因大量素材为手持拍摄、抖动剧烈且难以被运动强度单独筛出，团队通过评估相邻帧之间光流的一致性来估计相机稳定性，剔除相机运动不稳的片段。
· 幻灯片式运动：针对屏幕录制或幻灯片演示中常见的浮动照片/横幅，分析每帧全体像素光流的散度（divergence），若散度长期保持低值则判定为 slide movement 并剔除。
· 转场：见 shot_segmentation。
所有阈值未给数值。
③ Motif-Video 2B：用 UniMatch 在采样帧对之间估计光流并计算每个视频的运动分，双侧截尾——剔除极低运动（通常是静态或近静态）与极高运动（往往含剪切、抖动或不稳定相机运动）两端，保留中间带以匹配主训练阶段所要的平滑时序动态。运动过滤在每次分辨率阶段跃迁时重新施加且阈值更严。此外 720p SFT 阶段引入一个语义化的运动准入条件：VLM 标签 action=Dynamic 被用作「dynamic-motion criterion」——即在低层光流统计之外，再叠加一层「大模型判断这段视频的动作是否算动态」的语义门槛，这是本组中运动过滤从「像素统计」走向「语义判断」的一个具体案例。

### [Movie Gen](../models/Movie_Gen.md)

四步串联：① 用内部的静态视频检测模型（internal static video detection model）直接剔除完全无运动的视频；② 用 FFmpeg 的 VMAF motion score 与 motion vector 判定「合理运动」，附录给出的定量阈值为 motion score > 2.0、motion vector 均值 > 0.5 且 < 7（下界剔除慢动作/近静止，上界剔除运动过剧烈的素材）；③ 用 PySceneDetect 的 Shot Boundary Detection 识别频繁抖动运镜——抖动视频会被误拆成大量镜头，故以「每秒检出镜头数 ≥ 0.85」为阈值剔除，动机是训练数据抖动会导致生成视频画面发抖；④ 剔除带特殊运动特效的视频，例如幻灯片式（slideshow）视频。整体做法沿用 Emu Video（Girdhar et al., 2024）的低运动自动过滤思路。SFT阶段对运动使用更严格的阈值，并由人工确认「有非平凡运动且无相机抖动」。

### [NVIDIA NeMo Curator](../models/NeMo_Curator.md)

开源框架中实现最细、参数最透明的过滤环节，两级 stage 串联：
【MotionVectorDecodeStage】不做完整光流计算，而是直接从压缩码流中解码轻量运动向量（lightweight motion vectors）并据此计算运动分数——这是显著的成本优化，避免了在 PB 级数据上跑光流网络。参数：target_fps 默认 2.0（每秒采样帧数）、target_duration_ratio 默认 0.5（只分析 clip 的一半时长以省算力）、num_cpus_per_worker 默认 4.0、num_gpus_per_worker 可选 0.5。
【MotionFilterStage】施加阈值过滤。产出两个指标：motion_score_global_mean（全局平均运动分，默认阈值 0.00098）与 motion_score_per_patch_min_256（256 个空间 patch 上的最小运动分，默认阈值 0.000001）。双指标设计的用意：全局均值防「整体静止」，patch 最小值防「画面大部分静止、仅局部小物体动」的伪动态样本。
【Cosmos WFM 生产版增强】另有一个基于 ViT 架构、在光流序列上训练的运动分类器，除剔除静止内容外还能剔除剧烈抖动的失控镜头（erratic camera motion），并对镜头运动类型打标（pan 平移 / zoom 推拉 / tilt 俯仰）——即运动过滤同时兼任「镜头运动标注」的职能，产出可用于运镜控制训练的标签。该 ViT 分类器在开源版中未提供权重。

### [OmniHuman 数据集 + OHBench](../models/OmniHuman.md) ⚠️

【工具】UniMatch——统一的光流/立体匹配/深度估计模型，此处用于光流估计以「分析运动质量」（analyze motion quality）。相比传统的 RAFT 或帧差法，UniMatch 是较新的选型。
【过滤对象：设了上下界】明确剔除三类：静止（static）、抖动（shaky）、异常（abnormal）片段。这一点优于 UniTalking（后者只剔除静止、无上界），即 OmniHuman 同时设置了运动下界（防止照片式静止画面或人物一动不动）与上界（防止手持抖动、剧烈晃动导致的运动模糊与不可学习的样本）。对以 YouTube 素材为源的数据集而言，剔除抖动样本是必要的——大量 vlog、户外内容为手持拍摄。
【与语义一致性过滤的配合】同一级中还有感知哈希 + SigLIP 余弦相似度的「语义不一致帧」过滤。二者分工明确：光流管的是「像素级运动是否合理」，SigLIP 管的是「语义内容是否连贯」——例如一段光流平稳但中途主体被完全替换的片段，光流查不出来而 SigLIP 能查出来。这种「低层运动 + 高层语义」的双重时序一致性检查在同类工作中不多见。
【阈值】光流幅值的上下界数值、SigLIP 余弦相似度阈值、感知哈希汉明距离阈值全部未披露。[不确定]
【与评测的呼应】OHBench 的全局层设有 Dynamic Degree（DD）指标，用 RAFT 光流衡量生成视频的运动强度——即训练数据的运动过滤维度与评测的运动诊断维度是对应的（尽管一个用 UniMatch 一个用 RAFT）。LTX-2 微调实验中 DD 提升 10.7%，间接说明运动过滤后的数据确实让模型学到了更充分的运动。

### [Open-Sora 系列](../models/Open-Sora.md)

【Open-Sora 1.x】使用 **UniMatch（GMFlow）光流模型**计算稠密光流分数（`tools/scoring/optical_flow/inference.py`，输出列 `flow`），分数越高运动越大，用于剔除静止/近静止片段；同时把光流结果用于**相机运动检测**（识别 pan left / zoom in 等），高置信度的相机运动标签会被拼接进 caption。相机运动检测模块单独开源在 tools 下。
【Open-Sora 2.0】改用 **FFmpeg libavfilter 的 VMAF motion score**（比光流模型快得多，可在转码流水线内顺带算出），**双向剔除**：运动分极低（静止画面）与极高（剧烈晃动/混乱切换）者都丢弃。运动分数还会被**追加到 caption 文本中**，使推理时可通过 prompt 控制生成视频的运动幅度。
【Open-Sora Plan】用 **LPIPS 跳帧感知距离**代替光流作为运动度量（计算成本远低于光流），保留区间 **0.001 ≤ score ≤ 0.3**：低于 0.001 视为几乎静止，高于 0.3 则「存在明显抖动与闪烁」。作者说明该阈值经**人工抽检 2000 条视频**验证精度可接受。且如前所述，在字幕裁剪之后会**再跑一遍运动复检**（保留率从 44% 降到 42%）。
三者共同点：运动过滤都是双向的（既剔静止也剔抖动），且运动分数最终都被利用为可控条件而非仅作过滤器。

### [Ovi](../models/Ovi.md) ⚠️

使用光流模型 RAFT（Teed & Deng, 2020）承担两个作用：(1) 过滤掉静态视频（filter out static videos），即缺乏有效运动的片段直接剔除；(2) 为保留下来的片段计算并保存 motion score（运动分数）作为标注/筛选信号。
【阈值】静态判定的光流幅值阈值、motion score 的取值范围与高低两端截断策略均未公开[不确定]。
【抖动剔除】论文只提到剔除「静态」一端，未提及是否剔除相机剧烈抖动/快速摇镜等运动过强的一端[不确定]。
【motion score 的下游用途】论文未说明是否作为训练条件、采样权重或课程调度依据[不确定]。

### [Script-a-Video](../models/Script-a-Video.md) ⚠️

论文未描述任何运动过滤环节：无光流计算、无运动分数阈值、无静态镜头剔除、无抖动/晃动检测。[不确定]
运动信息在本工作中不作为「过滤维度」而作为「标注维度」出现：
1) Shot 流的 camera 字段显式记录镜头运动（movements），使运镜成为可描述、可控制的语义字段；
2) Shot 流的 visual_description 要求对核心动作做客观的时序叙述（objective, chronological narrative of core actions），并通过 intra-description timestamps 把微动作锚定到全局时间轴——即运动被「结构化描述」而非「分数化过滤」。
评测侧存在一个与运动相关的观察：Intra-Shot Subject Consistency（镜头内主体一致性，DINOv2 [CLS] 特征的帧间平均余弦相似度）基线 LTX-2-AV 高达 0.87，但论文明确指出这是假象——该高分源于模型未能注入参考特征、生成了近乎静止的内容，帧间变化极小所致（"trivially static content with minimal frame variation"）。本工作的完整方案维持 0.59-0.62 的「较低」分数，反而对应真实的动态内容。这是一个关于「静态内容会虚高一致性指标」的重要警示，对数据侧的运动过滤设计有参考意义。

### [Seedance 2.0 与 Seedance 1.5 pro](../models/Seedance_20_Seedance_15_pro.md) ⚠️

Seedance 1.0 在预训练过滤阶段剔除抖动过大与以静态内容为主的片段；在 Continue Training 阶段使用「基于光流的运动评估器（motion evaluators based on optical flow）」与美学打分器共同筛选出美学质量更高、运动动态更丰富的子集，报告称更丰富的运动动态使模型生成更自然流畅。Seedance 1.5 pro 把「运动表现力（motion expressiveness）」列为数据管线的三大优先目标之一，并在评测端引入生动性指标，明确反对以慢动作换稳定性。具体运动分数阈值未披露。[不确定：阈值数值]

### [SkyReels 系列](../models/SkyReels.md) ⚠️

【SkyReels-V2】运动是该系列的核心关切（论文主张现有模型在「运动动态性 vs 画质」上妥协）。数据侧：基础过滤剔除静止画面与相机抖动样本；打标侧训练了专门的「基于分类的运动识别 captioner」——通过对运动做分类标注，产出「9.3万条高置信度样本 + 1.6万条运动轴（motion axis）均衡的合成数据」用于训练该 captioner，在1.5万段视频测试集上单一类型运动预测准确率89%。运动信息随后进入结构化 caption 的相机运动字段，并在强化学习阶段作为优化目标（运动一致性与流畅度）。
【SkyReels-V4】把「运动质量」列为三大过滤维度之一，并在均衡环节引入「运动多样性」：为每个主体类别或场景类别定义关键运动模式（key motion patterns），按模式做配比均衡，防止某类主体只出现单一动作。
两代均未公布光流/运动分数的具体算法与阈值数值。[不确定]

### [Sora 2](../models/Sora_2.md) ⚠️

完全未披露。未提及光流计算、运动分数阈值、静态镜头剔除或抖动/手持晃动剔除策略。考虑到模型在物理动力学（重力、动量、碰撞）上的表现被重点宣传，推测存在面向运动质量的筛选，但无任何依据。[不确定]

### [SpeakerVid-5M](../models/SpeakerVid-5M.md)

SpeakerVid-5M 不做传统的光流/运动幅度阈值过滤，而是把运动强度做成一个可检索的标注维度，并仅在 HQ 子集环节用作筛选条件：
【运动强度标注方法】用 Qwen2.5-VL 做多 persona 打分——设计三个不同人格的 prompt，各自从不同视角对同一视频在 1–5 分制上评分：(1) 专家视角，评估动作幅度（movement amplitude）与身体动作频率；(2) 观众视角，评估身体语言反映出的情绪起伏与观众反应；(3) 标注专家视角，评估手势使用（gesture usage）与交互频率。剔除离群值后取平均分作为最终的运动幅度标注。这是用 MLLM 替代光流打分器的典型做法，且用多 persona 集成来降低单次打分的方差。
【作为过滤条件的使用】仅在 HQ / SFT 子集构造时启用：motion score > 2（5 分制）才入选，即排除近乎静止的呆板片段。全量数据集不施加运动下限。
【间接的静态/抖动控制】运动模糊过滤（人脸/手部 Laplacian 方差 < 0.1 剔除）会剔除因剧烈抖动或快速运动导致的糊帧；YOLO 跟踪的时空裁剪则保证主体在画面内稳定，间接抑制了强烈的镜头晃动。
【未使用】无光流（optical flow）计算、无 RAFT/UniMatch 类运动分数、无专门的静态镜头检测器。考虑到语料本身是访谈/对话类近景人物视频，运动幅度天然受限，传统运动过滤的必要性较低。

### [Step-Video-T2V](../models/Step-Video-T2V.md)

第3阶段专设「视频运动评估（Video Motion Assessment）」：使用 Farneback 稠密光流算法（OpenCV 经典算法）计算片段的光流场，导出三个标量指标——Motion_Mean（平均运动幅度）、Motion_Max（峰值运动幅度）、Motion_Min（最小运动幅度）。三值组合用于识别并剔除两类样本：几乎静止的片段（运动信息量不足、易导致模型生成静态画面）与运动过于剧烈/抖动的片段。具体阈值未公布。
衍生模型 Step-Video-TI2V 把这套运动打分从「过滤器」升级为「可控条件」，方法细节也更明确：每隔 12 帧采样一次，转灰度图后计算光流幅值，取幅值最高的一批数值再求均值作为该视频的运动分；对全部训练数据提取运动分后，一方面设阈值滤掉运动过高或过低的视频，另一方面把运动分作为显式条件注入模型，实现推理时「运动幅度可控」。这是「数据标签直接转化为可控条件」的一个清晰案例，对本调研有参考价值。

### [UniTalking](../models/UniTalking.md) ⚠️

运动过滤仅以「剔除静态视频」（filter out videos that are static）一句表述存在，是视频过滤级三条规则中的第一条。论文未披露：使用何种运动度量（光流幅值 / 帧间差分 / RAFT / 运动分数模型）、阈值数值、是否设运动上界（即是否剔除运动过于剧烈或镜头抖动的片段）。
【只设下界、不设上界的推断】从表述看仅剔除「静态」，未提及剔除抖动/剧烈晃动，即只有运动下界没有上界。对说话人视频而言这一取舍是合理的——目标场景本身运动幅度有限，主要风险是「照片式静止画面」（人物一动不动或视频实为静态图 + 音轨），剔除这类样本可避免模型学到不动的嘴。但缺少上界意味着手持抖动、快速镜头运动的样本不会被剔除。
【上游继承】OpenHumanVid 的过滤维度中已包含 motion 一项，故该子集的运动质量有上游保障。
【与评测的关系】论文的评测指标（Sync-C / Sync-D / 主观视频质量 / 音色相似度 / WER）中没有任何运动相关指标，也未做运动过滤的消融，因此该环节的实际贡献无从评估。[不确定]

### [UniVerse-1](../models/UniVerse-1.md) ⚠️

论文未描述任何独立的运动过滤环节：没有光流计算、没有运动分数阈值、没有静态镜头剔除、没有手持抖动或镜头晃动剔除。运动相关的质量控制仅隐含于 DOVER 分数（其技术维度包含对时域失真、卡顿、模糊的感知评估）。
值得注意的是，评测侧反而把运动强度作为核心观测量：Motion Score（MS）是 Verse-Bench 主表的第一项指标，且消融结果显示该指标对训练策略高度敏感（完整模型 0.20、w/o LQLS 0.38、w/o INSS 1.10）——这里 MS 数值大幅偏离并不等同于「运动更丰富」，而是伴随美学分与 ID 一致性同步下降出现，更可能反映画面不稳定或伪影导致的虚假运动。这从侧面说明缺乏运动过滤的数据侧问题最终转化为了训练策略上的补偿需求。[不确定]

### [Unison](../models/Unison.md) ⚠️

论文完全未描述运动过滤：无光流计算、无运动分数阈值、无静态镜头剔除、无抖动/晃动剔除、无运动幅度分桶。
【与「motion」在论文中的实际含义有关的澄清】Unison 标题与全文高频使用「motion」一词，但其所指是「生成结果中人物运动与音频的时序对应关系」（motion-audio synchronization），而非「训练数据的运动强度筛选」。论文关心的是钢琴演奏时音符起音是否对应手指动作、撞击声是否对应画面冲击瞬间——这是对齐问题，不是运动质量问题。因此 motion 相关的全部机制都在训练策略层（双向跨模态 forcing）与架构层（三帧窗口交叉注意力），数据层无对应设施。
【间接的运动保障】五个数据源中 HDTF/VFHQ/CelebV-Text 均为说话人视频，天然含唇部与头部运动；OpenHumanVid 以人物活动为主题；VGGSound 的视听事件多伴随可见运动。选材本身保证了运动的存在性，无需额外过滤静态片段。此为推断。[不确定]
【评测侧亦无运动指标】Table 1 的十一项指标中无 Motion Score 类的运动强度度量（对比 UniVerse-1 的 Verse-Bench 把 MS 列为主表第一项），说明团队并未把运动幅度视为需要观测的维度。

### [Veo 3 / Veo 3.1](../models/Veo_3_Veo_31.md) ⚠️

[不确定] 完全未提及运动过滤。官方未披露光流（optical flow）计算、运动分数阈值、静态镜头剔除或抖动/手持晃动剔除策略。仅可从「模型对真实世界物理有较好模拟」「运动表示精度高」等能力描述侧面推测存在某种运动质量筛选，但无直接依据。

### [Vidu S1](../models/Vidu_S1.md) ⚠️

有两条方向相反的运动约束：一方面通过「镜头稳定性」过滤只保留静止或慢速运动的镜头，剔除大幅运镜/抖动（jitter 亦在视觉质量项中被剔除）；另一方面通过「交互性」过滤要求主体本身具有清晰可辨的动作或行为，剔除完全静止无动作的样本。即「镜头要稳、主体要动」。未提及使用光流或具体运动分数阈值 [不确定具体方法与数值]。

### [Wan 2.5 / 2.6 / 2.7](../models/Wan.md) ⚠️

Wan 2.1 把运动质量提升为与视觉质量并列的独立漏斗层级，目标是「选出自然、完整、运动显著的视频，同时规避静态与抖动」。
【六档分级与差异化处置】最优运动（保留，最高优先）／中等质量运动（保留，保证运动多样性并帮助模型理解时空关系）／静态视频（聊天访谈类，画质高但运动少 → 单独识别并降低采样比例）／相机主导运动（航拍等，近似静态图 → 大幅降低采样优先级）／低质量运动（主体过多、严重遮挡、主体不清 → 排除）／抖动镜头（业余手持、运动模糊、前后景难分 → 系统性排除）。
【要点】静态与相机主导运动不是被删除而是被「降采样」，体现了「保分布不丢概念」的一贯思路；真正被删除的只有低质运动与抖动两档。
【方法】Wan 2.1 未说明六档分级所用的具体算法（是否光流、是否训练分类器），也未给出任何阈值数值。同族旁证：Wan2.2-S2V 用 UniMatch 预测光流并计算运动分；Wan-Dancer 用 SEA-RAFT 生成光流 mask（用途是损失函数而非过滤）；Wan-Bench 评测侧用 RAFT 光流幅值度量大运动生成能力。[不确定：阈值与算法]

### [音视频生成评测基准合集](../models/av_benchmarks.md) ⚠️

五者均未采用光流或运动分数阈值做过滤[不确定]，但存在若干功能等价的替代机制：
【AV-SyncBench】Local Jitter 挑战任务（30–700 ms 三档严重度的局部抖动扰动）与 Global Speed Change（0.8×–1.25× 共 10 档变速）本质是在人为构造「运动-音频时间基不一致」的负样本，可反向用作训练数据中变速/抽帧异常样本的检测思路。
【PhyAVBench】Time and Causality 维度下的「周期运动 / 非周期运动节奏一致性」测试点，以及 Transient Synchronization（起振与止声）测试点，考察的正是运动与声音的因果一致性，比单纯运动幅度阈值更语义化。
【VABench】未做运动过滤；生成侧被测模型的静态画面问题由 Visual Realism 与 Artistry 打分间接惩罚。

### [视频 Caption 模型生态](../models/caption_models.md)

captioner 生态与运动过滤的交互点有三：
【打标器输出运动信息供过滤使用】Tarsier2 天然会描述相机运动类型（zoom in / pan right 等），Goku 论文明确指出这是选它做视频级 caption 的关键优势 —— 无需额外的相机运动标注模块即可获得镜头语言标签。
【专用运动分类器与 captioner 分工】主流做法是运动识别不交给通用 VLM 而用轻量分类器：Movie Gen 训练了 16 类相机运动分类器，高置信度预测结果作为前缀拼到 caption 上；LongCat-Video 的相机运动（pan/tilt/zoom）由单独训练的轻量分类器负责而非 VLM（推测出于成本与精度考虑）；HunyuanVideo 有独立的自研 camera movement classifier（1.5 版升级为 clip 级 + 时序级双粒度）；SkyCaptioner-V1 的相机运动子专家基于分类，训练数据为 9.3 万条高置信度人工标注 + 1.6 万条运动轴均衡合成数据，1.5 万条测试集上单类型运动准确率 89%。
【运动过滤本身（光流/运动分数阈值、静态与抖动剔除）属于打标上游】Foley-Omni 用 motion∈[0.1, 3.2] 阈值；InstructAV2AV 用 CoTracker3 运动阈值；Open-Sora Plan 用 LPIPS 上界 0.3（超过即出现抖动闪烁，结论经 2000 条人工抽检验证）。这些均在 captioner 之前执行。
【生态判断】「运镜识别用专用分类器、内容描述用 VLM」是 2024–2026 年稳定的分工范式，原因是 VLM 对运镜的判别准确率不稳定且难以量化验证，而分类器可给出置信度用于过滤。SkyCaptioner-V1 的做法（把分类器结果蒸馏回 7B 统一 captioner）是两条路线的融合尝试，其影视专业字段平均准确率 76.3%（镜头类型 93.7%、机位角度 89.8%、机位位置 83.1%、相机运动 85.3%），显著超过 Qwen2.5-VL-72B 等更大的通用模型。

### [几何/结构化标注数据集合集](../models/geometric_datasets.md)

运动过滤是这四个数据集区别于常规视频生成数据集的关键点——常规pipeline剔除过大运动，几何标注数据集反而剔除“运动不足”者。SpatialVID：以 VMAF 结合三项相机轨迹度量做筛选——MoveDist（总位移距离）、RotAngle（累计旋转角）、TrajTurns（方向变化次数），并用基于加速度的检测器识别突兀、非物理的运动抖动予以剔除；量化证据是 Panda-70M 中 80%+ 视频因视差/运动不足无法被 MegaSaM 重建，故必须主动检索 walk/tour/drone 类高运动素材；最终库中80%的clip具有弯曲或转向轨迹。SceneScribe-1M：以“运动多样性”为初筛主轴，Qwen2.5-VL-72B 判定运动强度未知者直接剔除，并因运动要求大幅缩减源池；选用 MegaSaM 的理由正是其在相机视差受限场景下仍稳健。WildWorld：动作由游戏输入直接给定，无需从像素反推运动，静态帧通过动作ID可精确识别。Action100M：无光流层面的运动过滤，语义分割阶段的方差最小化聚类天然把静止段落合并为长节点

### [视频生成后训练数据策略](../models/post_training_data.md) ⚠️

锚论文在数据侧无运动过滤描述，但在奖励侧设有专用的 Motion Quality 奖励模型（评估运动动态的真实性、平滑性与连贯性，抑制抖动、不连续运动、时序不一致的物体转换），并在结果中报告运动质量是 RLHF 增益最显著的两个维度之一 [数据侧不确定]。
【横向】
· Allegro SFT：UniMatch 光流运动分数区间 [1.0,100]；
· HunyuanVideo 原版 SFT 人工标准含运动速度、动作完整性、运动模糊三项；
· Motif-Video 2B SFT 显式要求 action=Dynamic；
· SkyReels-V2 的 RL 目标明确聚焦运动质量（动态一致性与流畅度）而非通用美学偏好，其偏好数据的自动侧样本正是「对真实视频施加受控失真」生成的损坏样本；
· LongCat-Video 的 Motion Quality 奖励模型有一个值得借鉴的设计：以 VideoAlign 为基座在内部标注数据上微调，输入灰度视频（去色以迫使模型只评估运动而不被色彩/美学干扰）——这是解耦运动奖励与美学奖励的直接工程手段；
· Cosmos-Predict 2.5 单列「high motion」域（1.0M 条）做专项 SFT。

### [主流视频预训练数据集合并调研：Panda-70M、InternVid、Koala…](../models/pretraining_datasets.md) ⚠️

**七者全部做运动过滤，且全部是双向的（既剔静止也剔抖动/剧烈），但度量工具分三派**：
【光流派】
- **LVD-2M**：**RAFT**，在时间 **2fps**、空间 **520×960** 下对相邻采样帧计算，取时空平均的光流幅值，**<20 则丢弃**（实测官方100条样本最小值20.19，完美印证该阈值；样本均值51.91、最大201.33；数据集整体均值47.8）。**这是七者中唯一给出可验证绝对光流阈值的数据集。** 目的是剔除静态场景与「静止背景前的口播」。作者同时指出光流的局限：**手持拍摄的抖动视频光流分很高但没有有意义的运动**——这正是他们额外加一层 PLLaVA 语义判断的动因。
- **MiraData**：**RAFT** 帧间光流作为「运动强度」筛选项，在统一2fps下计算，**阈值未公开**。[不确定] 另在评测端提出 **Tracking Strength（CoTracker 追踪点相对首帧的平均位移）** 作为光流动态度的修正——论文图5给出反例：某对视频光流动态度为1.2 vs 0.7（排序错误），而 Tracking Strength 为4.1 vs 11.8（排序正确），说明**光流会把相机抖动和往复局部运动误判为高动态**。
- **OpenVid-1M**：**UniMatch** 光流，剔除运动差异分最高与最低的两端，保留「适中」运动。阈值未披露。[不确定]
- **UltraVideo**：**RAFT** 按间隔采样后取全局平均，保留 **0.1 ≤ 分数 ≤ 100**。
【感知/特征距离派】
- **Panda-70M**：不用光流，用 **ImageBind 首尾特征距离**——距离 **≤0.15** 视为运动过小而丢弃（同时距离 >1.0 视为语义变化过大也丢弃），即用同一个特征距离同时承担「运动下限」与「一致性上限」两个职能。此外还用「与前序片段距离 >0.3」做多样性去重。
【学习式打分派】
- **Koala-36M**：motion_score 作为 VTSS 的三个输入子指标之一（CSV实测范围 **0.01–267**），**不单独设阈值**，由 VTSS 网络综合判断。其人工标注准则中对运动的定义值得引用：**「运动区域需覆盖画面30%以上，否则因动态不足而降分」**，并区分「业余相机抖动」与「专业运镜」分别惩罚/奖励。
- **InternVid**：仅定性描述「过滤掉静止或极端动态（如相册浏览）的片段」，**无模型无阈值**。[不确定]
- **LVD-2M 的人工验证结果**可作为运动过滤有效性的横向标尺（40条/数据集，3档评分，「非常动态/中等/不动态」）：**LVD-2M 30.0%/62.5%/7.5%**、HD-VG 20.0%/37.5%/42.5%、InternVid 15.0%/60.0%/25.0%、Panda-70M 7.5%/67.5%/25.0%、WebVid 7.5%/42.5%/50.0%。**Panda-70M 与 WebVid 各有约四分之一到一半的片段被人类判为「不动态」**，这是「大规模数据集普遍缺乏运动」的直接证据。
【共同缺口】七者**均未把运动分作为可控条件写入caption或训练条件**（对比 Open-Sora 把 motion score 拼进caption、Koala-36M 把三个分数经AdaLN注入——注意后者是其**生成模型**的设计而非数据集本身的标注策略，但 Koala-36M 的 CSV 确实发布了三个分数供下游做条件控制，这是七者中唯一提供了可用于条件注入的结构化分数列的数据集）。

## 去重方法（精确去重与基于embedding的语义去重分别记录）

`deduplication` · 详细程度: brief

### [Allegro](../models/Allegro.md) ⚠️

论文未描述任何去重步骤——既没有精确去重（哈希/pHash），也没有基于 embedding 的语义去重或近重复聚类。这是该 pipeline 相对其披露粒度而言的一个明显空白，考虑到数据来源包含 WebVid、Panda-70M、HD-VILA、HD-VG、OpenVid-1M 等相互存在重叠的公开数据集，跨数据集重复的风险实际存在但未被论文讨论。[不确定]

### [Apollo](../models/Apollo.md) ⚠️

论文完全未提及去重环节：既无哈希/指纹级精确去重，也无基于 embedding 的语义去重。在 8100 万条、疑似来自短视频平台的语料规模下，重复内容（同一素材的多次搬运、模板化二创、同一视频的重叠切片）风险客观存在，且论文的场景切分会从同一长视频产出多个片段、天然带来近重复，但论文对此未作任何说明。[不确定]

### [CineDance / CineDance-1M](../models/CineDance.md) ⚠️

论文未描述任何去重环节——既无基于哈希/指纹的精确去重，也无基于 embedding（如 CLIP/ImageBind 特征）的语义去重，正文的三阶段 pipeline 中不含去重步骤。
【推测性缓解因素】① 原始素材仅 45,181 条长视频，条目数量级不大，长片之间的完全重复概率相对较低；② 上游 MiraData / LVD-2M / Koala-36M 各自已做过去重；③ 叙事序列由同一部长片内的连续镜头组成，同片内不同序列天然不重叠。
【潜在风险】跨来源数据集之间（MiraData 与 Koala-36M 可能都收录同一部影片）的重叠未见处理说明。整体属于披露缺失。[不确定]

### [CogVideoX](../models/CogVideoX.md) ⚠️

[不确定] 论文完全未提及去重环节——既没有精确去重（哈希/指纹），也没有基于 embedding 的语义去重或近重复检测（如 SSCD copy-detection embedding），更没有语义聚类与长尾重采样。这是 CogVideoX 数据 pipeline 相对同期工作最显著的缺失之一。
唯一与「冗余」间接相关的是负面标签中的 Editing 与 Lack of Motion Connectivity（可剔除由同一素材重复拼接而成的视频），但其设计目的是保证运动真实性而非去重。

### [Cosmos-Predict2.5](../models/Cosmos-Predict25.md) ⚠️

语义去重为独立的第六阶段，方案设计得比较完整，且专门考虑了增量扩容场景：
(1) 聚类：先用 embedding 相似度把 clip 分配到簇（assign video clips to clusters using embedding-based similarity），把全局两两比对降为簇内比对，这是可扩展性的关键；
(2) 簇内比对：簇内 clip 两两比较以检出语义相似内容（compared pairwise to detect semantically similar content）；
(3) 仲裁规则：重复组内保留分辨率最高的版本（the highest-resolution version is retained），论文给出的理由是高分辨率保留更细的视觉细节、提供更丰富的训练信号——这是一条明确的「同内容选优」而非「随机保一」策略；
(4) 在线增量去重（online deduplication strategy）：每条新入库 clip 与此前已保留的 clip 比对，tie-breaking 时优先保留更早的、分辨率更高的 clip。此设计使数据集可持续增长而不必全量重跑去重，同时保持全语料的语义一致性；
(5) 基础设施支撑：接入 Milvus 开源向量数据库做 embedding 检索，支撑语义相似度搜索与 caption 级文本 embedding 检索。
论文只描述了语义/embedding 级去重，未单独描述精确去重（MD5/字节级哈希）环节——推测由上游采集或 Cosmos Curator 工程实现承担，论文未提。[不确定：embedding 模型型号、相似度阈值、是否另有精确去重]

### [Data-Juicer 2.0](../models/Data-Juicer.md) ⚠️

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

### [Foley-Omni](../models/Foley-Omni.md) ⚠️

[不确定] 论文完全未提及去重环节，既无精确去重（文件哈希、感知哈希 pHash）也无基于 embedding 的语义去重描述。
潜在风险点：训练数据跨任务组存在数据集重叠——SpeakerVid 同时出现在 VisualTTS（1,980h组内）与 V2ST（216h组内）两处，AudioSet 用于 TTM 而 VGGSound 本身是 AudioSet 的子集衍生（同源 YouTube 视频），存在样本重复计入甚至训练-评测污染的可能。此外 V2ST-Bench 的300条片段论文称「drawn from curated audiovisual pool」（取自同一清洗后的数据池），若未做严格的训练集剔除，存在评测集泄漏的隐患——论文仅说明经过人工复核，未明确声明与训练集无交集。这是该工作在方法论严谨性上一个值得关注的空白。

### [Goku](../models/Goku.md) ⚠️

仅描述了一种机制：**基于感知哈希（perceptual hashing）的近似去重**。做法是对每个片段的关键帧计算感知哈希值并两两比对，若两个片段哈希值相近则判为重复，此时**保留美学评分更高的那条**，丢弃另一条。论文明确限定该去重发生在「同一源视频切出的片段之间」（clips from the same source video），主要解决长视频切分后相邻片段内容高度重叠、以及重复镜头/回放镜头的问题。
【未涉及】跨源/全局的精确去重（如文件级 MD5、逐帧哈希全库比对）、基于 embedding 的语义级去重（如 CLIP/DINOv2 特征聚类去重）、图像侧（LAION 1亿样本）的去重处理，均未在论文中说明。这是 Goku 数据流水线相对薄弱的一环——DINOv2 特征已经在切分阶段被提取，却未被用于语义去重。[不确定]（是否存在未披露的全局语义去重）

### [Hailuo / MiniMax Video](../models/Hailuo.md) ⚠️

[不确定]（完全无披露）。未说明是否做精确去重（哈希级）或基于 embedding 的语义去重，未给出特征提取方式或相似度阈值。

### [HunyuanVideo-Foley](../models/HunyuanVideo-Foley.md) ⚠️

论文完全未提及去重环节：既无文件哈希/音频指纹级的精确去重，也无基于 embedding（音频或视觉）的语义去重，未讨论跨来源重复问题。
【本任务下的重复风险评估】风险客观存在且形态特殊：
1) 同源重复——同一条长视频经场景切分后产生多个 8 秒块，若该视频内容重复性高（如固定机位的长时间录制），会产生大量近似样本；论文的切分策略天然会放大这类重复，却无任何去重措施。
2) 音效素材库重复——网络视频中大量使用同一批商业音效库素材（同一段爆炸声、同一段脚步声被成千上万个视频复用），这类音轨级重复在音频指纹层面极易检出，但若不处理会让模型过拟合到特定音效样本。这是 V2A 任务特有的、比视频生成任务更严重的重复风险，因为音效素材的复用率远高于画面素材。
3) 背景音乐重复——热门 BGM 在短视频中的复用率极高。
【与规模的关系】10 万小时的量级下，即便存在 10-20% 的重复也不易被察觉，但会实质性扭曲类别分布（而类别分布管理恰恰是本工作强调的环节）——若某类音效因素材库复用而被系统性放大，标签统计出的分布将偏离真实的声学事件分布。这使得「无去重」与「做类别分布管理」两项设计之间存在方法论上的张力，论文对此零讨论。[不确定]

### [HunyuanVideo](../models/HunyuanVideo.md) ⚠️

【原版】做了语义级去重且方法明确：用内部自研 VideoCLIP 模型抽取视频 embedding，按余弦距离（cosine distance）判定重复并去重；同一套 embedding 随后跑 k-means 得到约 1 万个概念中心用于概念重采样与均衡。未提及哈希级精确去重。
【1.5】仅表述为「basic deduplication and the removal of corrupted files（基础去重与损坏文件剔除）」作为最前置步骤，未说明是哈希去重还是 embedding 语义去重，也未说明是否延续 VideoCLIP 方案。1.5 在去重描述上明显弱于原版。[不确定]

### [InstructAV2AV](../models/InstructAV2AV.md) ⚠️

[不确定] 论文完全未提及去重环节，既无精确去重（文件哈希、pHash 感知哈希）也无基于 embedding 的语义去重描述。数据集卡亦未提及。
【本pipeline下的具体重复风险，值得单独指出】
  1. 同源多片段重复：一部电影经 PySceneDetect 切分后可产出数十至上百个单镜头片段，若同一部影片的多个片段均通过过滤进入数据集，则场景、人物、光照、混音风格高度相似，构成语义近重复。来源中 MovieBench、Condensed Movies、Short-Films-20K 都是影片级数据集，此风险实质存在。
  2. 同片段多次编辑重复：同一个 source 片段完全可以被施加多条不同指令、生成多个 target，从而产出多条训练对。这在数据引擎设计中是自然且经济的做法（复用昂贵的前置过滤成果），但会造成 source 侧高度重复。论文未说明是否做了「每个 source 最多生成 N 条指令」的限制，也未说明 79K 训练对对应多少个不同的 source 片段——若这个数字远小于 79K，则数据的实际多样性会显著低于名义规模。这是评估该数据集有效规模的关键未知数。
  3. 滑窗切分重复：若 5 秒窗口以滑窗方式从长单镜头片段中多次截取，相邻窗口会大幅重叠。
  4. 训练-评测污染：1K 人工评测集与 79K 训练集均出自同一条pipeline、同一素材池，论文未明确声明二者在 source 素材层面无交集（仅说明评测集经人工三重把关）。若同一部影片的不同片段分别落入训练与评测两侧，会造成评测偏乐观。这一点在本工作中比在真实数据集中更严重，因为合成 target 的风格由同一个 data engine 决定，模型学到的可能是「模仿 Wan2.2-5B + ElevenLabs 的输出风格」而非通用编辑能力，而评测集恰恰也由同一引擎产出——存在「评测偏向自家数据引擎风格」的系统性偏置，论文未讨论也未用第三方数据佐证。所幸论文额外在外部 AvED-Bench 上做了零样本评测（FVD 227.82 / FAD 4.32 / AV-A 23.71，优于 AVI-Edit 的 372.37 / 7.65 / 23.21），一定程度缓解了这一质疑。

### [2026 其他音视频联合生成](../models/JAVG_2026_misc.md) ⚠️

【NAVA —— 唯一做语义去重且方法明确的工作】原文「Redundant or near-duplicate clips by extracting video embeddings with VideoCLIP and performing large-scale k-means clustering」——用 VideoCLIP 提取视频嵌入，再做大规模 k-means 聚类，据此剔除冗余与近重复片段。这是典型的 embedding-based 语义去重（而非哈希级精确去重），k 值、簇内保留策略（每簇保留几条、按什么排序）、相似度阈值均未公开[不确定]。选择 VideoCLIP 而非逐帧 CLIP 说明其在时序维度上也做了考量。未提及精确去重（pHash/帧指纹/文件哈希）环节[不确定]。
【ALIVE】六阶段管线中完全没有去重环节，论文全文未提 deduplication[不确定]。考虑到其数据源为内部 raw data pool 且量级达 11M 样本，缺少去重是一个值得注意的空白——除非其原始池在入库前已做过去重（论文未说明）。
【OmniCustom / StreamChar】依赖上游数据集（SpeakerVid-5M / TalkVid / OpenHumanVid）自带的去重，自身无去重步骤[不确定]。OmniCustom 有一个隐含的「反去重」风险：其从同一段长视频中切出「前 4 秒参考 + 后 5 秒训练」的配对，同一说话人可能在数据集中重复出现多次，这在身份定制任务中是特性而非缺陷（需要同一身份的多样本），但也意味着说话人层面的分布可能高度不均衡[不确定]。
【CCL / Baton】未提及去重[不确定]。Baton 混合了 OpenHuman-Vid、AudioCaps、WavCaps 与互联网视频四个来源——AudioCaps 本身是 AudioSet 的子集，而 WavCaps 部分来源也与 AudioSet/FreeSound 重叠，跨数据集的重复风险客观存在但论文未处理[不确定]。
【ITS-JAVG】不涉及；但其 Best-of-N / EvoSearch 中的候选多样性维持问题，与去重要解决的「样本冗余」在数学上同源。
【总体判断】七项中仅 NAVA 一家做了显式去重，且是本批数据披露最完整的一项——这印证了一个规律：去重投入与数据规模正相关（NAVA 处理 100M 视频，其余多在 1M–11M 量级或直接复用已清洗的公开集）。

### [音视频联合生成基线合集](../models/JavisDiT_baselines.md) ⚠️

本合集五项工作全部未描述任何去重环节[不确定]，无论是精确去重（哈希/pHash/帧指纹）还是基于 embedding 的语义去重（CLIP/视频 embedding 聚类），这是一个统一的空白维度。
各自的间接情况：
- MM-Diffusion：Landscape 明确切成「不重叠（non-overlapped）」的 10 秒片段，这是片段级的重叠避免而非跨视频去重；928 条源视频规模极小，重复风险天然较低。
- AV-DiT：使用现成公开数据集，去重责任在上游数据集作者。
- JavisDiT / JavisDiT++：唯一相关的设计是「DPO 用的 3 万条 prompt 池与 SFT 训练数据严格不重叠（apart from the SFT training data）」，以及 33 万 SFT 与 2.5 万 DPO 样本互不重叠——这是训练/评测集划分层面的隔离，不是数据去重。TAVGBench 上游是否做过去重未知[不确定]。JavisBench 构建中的「后置过滤以保证多样性（ensure diversity）」在功能上接近语义去重，但作者未使用去重的表述、也未说明用什么方法度量多样性[不确定]。
- Harmony：400 万条来自多个公开数据集拼装（AudioCaps、Clotho、WavCaps 之间存在已知的音频重叠，例如 WavCaps 与 AudioSet 有交集），但论文未提及跨数据集去重[不确定]，这是一个潜在的重复风险点。
- UniAVGen：无任何描述[不确定]。

### [可灵 3.0 / 可灵视频 3.0 Omni](../models/Kling_30_Omni.md) ⚠️

Kling-Omni 报告明确采用“逐帧与时序指纹（frame-wise and temporal fingerprinting）”的去重机制，属于感知哈希/指纹类的精确与近重复去重。[不确定：是否另有基于 embedding 的语义级去重——报告未提；但其互联网挖掘环节使用自研 embedding 模型做跨模态语义检索，具备做语义去重的基础设施条件]

### [LTX-2](../models/LTX-2.md) ⚠️

采用基于表征的语义去重：对每个镜头取中间帧（Mid-Frame）计算 CLIP 图像嵌入，然后在嵌入空间做「Cluster and De-Duplicate（聚类并去重）」。属于 embedding 级语义去重，而非哈希级精确去重；论文未单独说明是否另有精确去重步骤，也未公布聚类算法、相似度阈值、簇内保留策略与去重比例。[不确定]

### [LongCat-Video](../models/LongCat-Video.md) ⚠️

仅在预处理首步做，方法明确但较为基础：使用 source video ID（来源视频标识）与 MD5 哈希双重去重。前者按来源侧的唯一 ID 剔除同一视频的重复采集，后者按文件字节级哈希做精确去重。
报告未提及任何基于 embedding 的语义/感知去重（如 CLIP/视频特征近重复检测、pHash 感知哈希），因此对「同一素材的不同编码版本、不同裁切、转载搬运」这类近重复缺乏显式处理手段。不过 caption 嵌入空间聚类与「按密度倒数采样」在 SFT 阶段起到了近似语义去重的效果——高密度冗余簇被自动降采样。[不确定：是否另有未披露的语义去重]

### [MOVA](../models/MOVA.md) ⚠️

论文完全未提及去重环节：既无哈希/指纹级精确去重，也无基于 embedding 的语义去重。考虑到数据来源同时包含 ACAV-100M、VGGSound、OpenVid-1M 等多个源自 YouTube 的公开数据集，以及 in-house 的 YouTube 抓取内容，跨数据集重复的风险客观存在，但论文对此未作任何说明。[不确定]

### [合并条目：① Mochi 1](../models/Mochi_MAGI_Motif.md) ⚠️

① Mochi 1：未披露。[不确定]
② MAGI-1：论文把去重的重要性上溯到 LLM 文献——引用 Lee et al. (2021) 与 Hernandez et al. (2022) 指出「即便少量重复数据也会显著损害性能」，据此「conduct rigorous de-duplication（执行严格去重）」。方法为双特征交叉判定：同时用 CLIP 与 DINOv2 计算片段间成对相似度，只要任一相似度超过阈值即判为重复并删除（逻辑或，偏严）。用 CLIP + DINOv2 双塔的用意在于兼顾语义相似与视觉/自监督结构相似两类重复模式。阈值数值与去重规模未给，也未说明如何在大规模上规避 O(N²) 的成对计算。[部分不确定：阈值、索引方案与去重量]
③ Motif-Video 2B（三者中唯一给出可复现工程细节的）：三阶段 SSCD 管线——
· 嵌入（Embedding）：用公开发布的 sscd_disc_mixup TorchScript 模型对每张图/每个视频编码，帧图先 resize 到 320×320 并做 ImageNet 归一化，产出 512 维描述子。视频取第 10 帧作为代表帧——论文明确解释这一选择的两个理由：避开最前几帧的片头与 logo 偏置，以及避免全帧两两比较、保持匹配可计算。选 SSCD 的理由是它专为 copy detection 设计，对重编码、裁剪和轻度编辑鲁棒，而这些正是网络爬取视频中最常见的重复形态。
· 分组（Grouping）：用 NVIDIA cuVS 的多 GPU IVF-PQ 索引在余弦距离下检索，每个 query 取 k=64 个近邻、nprobe=16，仅保留余弦相似度超过 0.9 的配对，再用 Union-Find 合并配对形成重复组。
· 代表选择（Representative selection）：每个重复组内按加权分 s = 0.5·res^ + 0.3·fps^ + 0.2·filesize^ 保留单一样本（三项均在组内做 min-max 归一化），组内其余成员丢弃。该规则偏好更高分辨率、更高帧率、重压缩程度更低的副本。
去重被放在 sanitation 首关（而非漏斗末端），意味着后续所有昂贵的模型推理都不会浪费在重复样本上。

### [Movie Gen](../models/Movie_Gen.md)

分两个层次，与字段要求的「精确/语义」二分基本对应：
· 感知级/近重复去重：使用 copy-detection embedding（SSCD，Pizzi et al., 2022）空间中的相似度剔除感知上重复的clip，附录表44给出的阈值为「embedding 余弦相似度 < 0.99 保留」，该步在漏斗中把1.22%降到1.15%（约剔除6%）。这一方法同样被用于 Movie Gen Audio 预训练数据的视觉去重。
· 语义级去重与均衡：用视频-文本联合embedding模型抽取语义embedding做聚类得到细粒度概念簇，先合并重复的簇（merge duplicate clusters），再按 1/sqrt(簇大小) 从每簇采样。该步既是语义去冗余也是概念均衡，把1.15%进一步降到0.94%。
· 评测侧亦关注重复：论文提到发现某些公开数据的train/eval split之间存在大量重复或近重复（如加了静态水印或文字的版本）。

### [NVIDIA NeMo Curator](../models/NeMo_Curator.md) ⚠️

以语义去重为主，方法链路清晰：
【嵌入】用 Cosmos-Embed1 生成 clip 级视频嵌入（早期与 Cosmos WFM 生产版用 InternVideo2；InternVideo2 已在 26.02 版中从 NeMo Curator 移除）。嵌入以 Parquet 格式落盘，同一份嵌入同时服务于语义检索与去重两个用途。
【聚类】对嵌入做 k-means 聚类，Cosmos WFM 生产环境取 k = 10000。
【簇内比对】只在同一簇内计算成对距离（pairwise distance）识别近重复，避免全量 N² 比对，这是 PB 级去重可行性的关键。
【保留策略】近重复组内保留分辨率最高的版本。
【效果】约移除 30% 的训练数据。
【框架实现】文档表述为「semantic clustering + pairwise similarity + k-means」，按 clip 分块（chunks of clips）组织去重。
【精确去重】视频侧未见基于哈希的精确去重 stage（文本侧另有成熟的 exact dedup 与 fuzzy dedup，后者官方称在 RedPajama v2 上相较 CPU 方案有 16 倍加速、TCO 降低 40%，但那是文本模态的数字，不可挪用到视频）。相似度阈值取值未公布。[不确定]

### [OmniHuman 数据集 + OHBench](../models/OmniHuman.md) ⚠️

【显式去重环节：无】论文未描述任何面向样本级的去重步骤——无视频指纹/pHash 级的精确去重、无 CLIP/SigLIP embedding 级的语义去重、无跨源重复检查。
【一处被用于其他目的的近似技术】第四级中使用了感知哈希（perceptual hashing）+ SigLIP 余弦相似度，但其用途被明确表述为「过滤语义不一致的帧」（filter semantically inconsistent frames），即用于单个片段内部的时序一致性检查，而非跨样本的重复检测。技术手段与去重高度相似但目的不同，不能算作去重。
【重复风险的具体形态】以 YouTube 为唯一来源的数据集，重复风险有其特殊性：(1) 同一内容的多次上传（搬运、转载、合集）在 YouTube 上极为普遍；(2) 同一 UP 主的系列视频（同一演播室、同一机位、同一人物）会产生大量高度相似的片段；(3) 8 万身份 / 100 万条意味着平均每身份 12.5 条，若身份分布长尾严重，头部博主可能贡献数百条近乎同质的片段。
【身份级去重与配额：亦无】论文未描述任何按身份的采样配额或长尾抑制。8 万身份的数字虽然亮眼，但其分布形态（是否均匀）完全未知——若呈幂律分布，实际有效身份多样性会远低于名义值。[不确定]
【评测集的部分兜底】OHBench 的 509 条视频选自 OmniHuman 但明确要求「与训练集之间存在 domain gap」，这在评测层面避免了训练-评测泄漏，但训练集内部的冗余问题不受此影响。

### [Open-Sora 系列](../models/Open-Sora.md) ⚠️

两个项目**均未实现实质性的去重环节**，这是其数据 pipeline 相对工业界的明显缺口。
- Open-Sora 的 tools/ 目录下没有独立的去重模块，README 特性列表虽提及过 deduplication，但 v1.2.0 的 docs/data_processing.md 四阶段流程中并无去重步骤，tools/datasets/datautil 主要做基于分数列的过滤与 meta 合并，未见 embedding 级语义去重实现。
- Open-Sora Plan 的七级漏斗表中同样不含去重级，论文与各版本 Report 均未讨论精确去重（哈希）或语义去重（embedding 聚类）。
- 潜在风险：Panda-70M 源自 HD-VILA-100M 的 380 万条长视频切成 7080 万片段，同一长视频的相邻片段高度相似；Open-Sora Plan 又叠加使用 ShareGPT4Video（同样源自 Pexels/Pixabay），来源重叠与片段内冗余都未被处理。[不确定]

### [Ovi](../models/Ovi.md) ⚠️

[不确定]。论文的数据 pipeline 四步中完全没有去重环节，既未描述精确去重（哈希/pHash/帧指纹），也未描述基于 embedding 的语义去重（CLIP/视频 embedding 聚类去冗余）。纯音频侧同样无去重描述。由于数据来自内部语料而非大规模网络爬取，重复度可能天然较低，但这只是推断，无任何文本依据。

### [Script-a-Video](../models/Script-a-Video.md) ⚠️

论文完全未提及去重环节：既无哈希/指纹级精确去重，也无基于 embedding 的语义去重，未讨论跨数据源或跨阶段的重复问题。[不确定]
潜在重复风险客观存在：生成侧四套数据（400K identity-centric、250K multi-shot、870K cinematic pairs、60K cinematic alignment pairs）来源同为内部影视素材库，且最终联合微调阶段明确复用了前述的 250K multi-shot sequences，说明跨阶段数据复用是有意为之；但不同阶段数据集之间是否存在同源片段的重叠，论文无任何说明。
值得一提的是，MTSS 的设计在「描述层面」内建了一种去冗余机制——Reference 流的中心化实体库使得反复出现的人物/场景只需描述一次，后续 Shot 与 Event 只引用 ID，论文称之为消除「redundant re-description」。但这是文本冗余的消除，与数据集样本去重是两个不同层面的问题，不应混淆。

### [Seedance 2.0 与 Seedance 1.5 pro](../models/Seedance_20_Seedance_15_pro.md) ⚠️

Seedance 1.0 采用语义去重（Semantic Deduplication）：用内部自研的视频表征模型（internally developed video representation model）提取鲁棒特征嵌入，基于嵌入对视觉与语义相近的片段做聚类；在每个近重复簇内仅保留质量总分（来自前一级质量过滤）最高的单个实例。报告未单独描述精确哈希去重（exact dedup）环节。1.5 与 2.0 未披露去重方案。[不确定：是否含精确去重、聚类阈值、去重比例]

### [SkyReels 系列](../models/SkyReels.md) ⚠️

【SkyReels-V4】明确采用基于表征的语义去重：对切分后的片段计算 VideoCLIP 嵌入并据此去重（「deduplication using VideoCLIP embeddings for segmented clips」）。属于 embedding 级语义去重，未公布相似度阈值、聚类算法与去重比例，也未说明是否另有哈希级精确去重。
【SkyReels-V2】论文数据章节未单独描述去重步骤（第三方结构化解读中亦未提取到去重条目），无法确认是否存在精确去重或语义去重环节；考虑到自采数据含28万部电影与80万集剧集，同一 IP 的重复镜头风险较高，推测存在某种去重但无官方依据。[不确定]

### [Sora 2](../models/Sora_2.md) ⚠️

完全未披露。既未说明精确去重（哈希级），也未说明基于embedding的语义去重。[不确定]

### [SpeakerVid-5M](../models/SpeakerVid-5M.md) ⚠️

论文与数据卡均未提及任何去重环节：既无基于视频哈希/指纹的精确去重，也无基于 embedding（CLIP/DINO 等）的语义去重，也未描述跨 clip 或跨源视频的近重复检测。
【客观风险】(1) 同一 YouTube 视频会被切分出多个 clip，同一说话人在同一场景下的多个片段高度相似，属结构性近重复；(2) 数据源覆盖 2006–2025 的 YouTube，同一访谈的转载、剪辑版、多平台重复上传在 YouTube 上极常见，采集时若仅按视频 ID 去重则无法识别内容级重复；(3) 83K 说话人 ID 对应 5.2M clips，平均每个说话人约 63 个 clip，头部名人（访谈/新闻类内容的典型主体）很可能贡献远高于平均的片段数，存在身份层面的长尾失衡。
【唯一相关机制】ArcFace 的 ID 校正步骤会跨 clip 计算人脸余弦相似度，但其目的是「保证同一 ID 的标注一致」而非「删除重复内容」，方向恰好相反——它是在做身份聚合而不是去重。
【结论】去重是本数据集披露体系中的明确空白。[不确定]

### [Step-Video-T2V](../models/Step-Video-T2V.md) ⚠️

技术报告未描述任何去重环节——六阶段 pipeline 中没有独立的去重步骤，也未提及感知哈希（pHash）等精确/近重复去重，或基于 embedding 的语义去重。
唯一在功能上部分承担去重效果的是第5阶段的概念均衡：VideoCLIP embedding 的 K-means（12 万+ 簇）配合 Cluster_Cnt（簇内样本数）做重采样，可以抑制高度相似内容的过度集中，并通过 Center_Sim 剔除簇内离群样本；但这本质是分布均衡与离群检测，不等价于重复样本剔除。这是本条目相对 HunyuanVideo（明确用 VideoCLIP 余弦距离做语义去重）的一处披露空白。[不确定]

### [UniTalking](../models/UniTalking.md) ⚠️

论文完全未提及去重环节：既无哈希/指纹级的精确去重（pHash、视频指纹），也无基于 embedding 的语义去重（CLIP/DINO 特征聚类），未讨论跨数据源的重复问题。
【重复风险的具体来源】本工作的两源结构使重复风险高于单源工作：OpenHumanVid 的视频样本本身采集自「多个公开可得的数据集」的聚合，其内部即可能存在跨子集重复；而华为内部采集数据若同样含有公开平台内容，则与 OpenHumanVid 存在重叠的概率不低。两源之间无任何去重机制，可能导致部分内容在 230 万条中被重复计入并在训练中被过采样。
【人物身份维度的去重也缺失】对说话人生成任务而言，还存在一个特有的重复维度——同一说话人的大量片段（如同一位主播/演员的多期节目）可能在数据集中高度集中，导致身份分布长尾严重。论文既未做身份级去重，也未做身份级配额控制，更未报告数据集中的唯一说话人数量。这对模型的身份泛化与音色克隆能力有直接影响，却是完全的信息空白。[不确定]

### [UniVerse-1](../models/UniVerse-1.md) ⚠️

论文完全未提及去重环节：既无哈希/指纹级的精确去重，也无基于 embedding 的语义去重，未讨论跨数据源重复问题。风险客观存在——自采数据以 YouTube 为主，而同时引入的 VGGSound 与 AudioSet 本身就是 YouTube 片段集合，三者之间存在潜在的内容重叠；此外 Pexels 素材在多个公开数据集中被反复收录也是常见情况。论文对此无任何说明。[不确定]

### [Unison](../models/Unison.md) ⚠️

论文完全未提及去重：无哈希/指纹级精确去重，无基于 embedding 的语义去重，未讨论跨数据源重复问题。
【重复风险客观存在且不小】这是聚合型数据构建的典型隐患，Unison 的具体风险点包括：
1) 音视频侧五个数据源中，HDTF、VFHQ、CelebV-Text 均以 YouTube 名人/公众人物访谈与演讲为主要素材，同一人物甚至同一视频被多个数据集收录的可能性很高；
2) VGGSound 与 AudioSet 均为 YouTube 视频集合，二者本身即存在已知的显著重叠，而 Unison 在音视频侧用 VGGSound、在纯音频侧用 AudioSet，跨模态语料之间存在重复；
3) YouTube-8M 与 AudioSet、VGGSound 三者的来源池完全相同（YouTube），重叠不可避免；
4) 音乐侧 VidMuse 与歌唱侧 YuE collection 之间是否存在曲目重复亦无说明。
【论文的沉默】对上述任一风险均无讨论，也未说明「自动化处理 pipeline」是否包含去重步骤。考虑到重复数据会导致特定人物/特定声音事件的过采样，进而影响身份一致性与声学多样性，这是一个实质性的方法学缺口。[不确定]

### [Veo 3 / Veo 3.1](../models/Veo_3_Veo_31.md) ⚠️

官方明确披露且相对具体的少数几项之一。Model Card：「All training data was deduplicated semantically across various sources.」技术报告：「All data is deduplicated semantically across sources to minimize the risk of outputs overfitting particular elements of training data.」Mitigations 一节另有：「removing duplicated and/or conceptually similar videos」。可确认为：跨数据源的语义级去重（semantic dedup，实现上应基于 embedding 相似度），且不止剔除完全重复项，还剔除「概念上高度相似」的视频。去重的动机被明确表述为降低输出对特定训练样本的过拟合/记忆化风险（即缓解逐字复现与版权风险）。[不确定] 未披露所用 embedding 模型、相似度阈值、聚类方式，也未区分精确去重（如感知哈希）与语义去重的各自贡献量。

### [Vidu S1](../models/Vidu_S1.md) ⚠️

在流程最前端（正式进入 pipeline 之前）对所有数据先做去重（all data are first deduplicated），再进行预过滤。论文未区分精确去重与基于 embedding 的语义去重，也未说明所用特征或相似度阈值 [不确定]。

### [Wan 2.5 / 2.6 / 2.7](../models/Wan.md) ⚠️

披露极其薄弱。Wan 2.1 报告仅在数据来源句中一笔带过：「We curated and deduplicated a candidate dataset sourced from…」——确认存在去重环节且发生在漏斗最前端（候选池构建阶段），但完全未说明方法：既未区分精确去重（哈希/指纹）与语义去重（embedding 聚类），也未给出相似度阈值、去重比例或簇内保留策略。
可能被误读的一点：第2步视觉质量中的「clustering」（100簇）目的是配额采样以保持分布，报告明确其作用是防长尾丢失，并非去重手段，不应混淆。
2.2/2.5/2.6/2.7 均无任何去重信息。[不确定]

### [音视频生成评测基准合集](../models/av_benchmarks.md) ⚠️

五者均未披露显式的精确去重或 embedding 语义去重流程[不确定]。相关的近似机制为：
【PhyAVBench】采用「反向去重」思路 —— 主动保证与现有训练集零重叠（全部新录制），并在采集时刻意跨不同个体、演示者与录制设备取样以提升样本内多样性，这是从源头避免同质化而非事后去重。
【AVBench】Hard Quota-Based Greedy Sampling 强制任一单属性占比 ≤50%，功能上等价于属性层面的去冗余/均衡采样。
【AV-SyncBench / VABench / Omni-Judge】未提及去重[不确定]。

### [视频 Caption 模型生态](../models/caption_models.md) ⚠️

[不确定] 这是 captioner 生态披露最少的字段之一。
【已知的少量证据】
· AVSCap-130K 的来源包含 AVoCaDO-107K，AVoCaDO-SFT 的来源又包含 FineVideo、Shot2Story 等公开数据集，而这些公开集之间本身存在视频重叠 —— 「数据集叠数据集」的滚雪球式复用带来了显著但未被量化的跨数据集重复风险，两篇论文均未讨论去重。
· Tarsier2-Recap-585K 从 1M 条公开数据集视频中产出，其来源（VATEX、TGIF、LSMDC 等）之间同样存在已知重叠，去重策略未披露。
· Panda-70M 的贪心集合覆盖是对「打标器」去冗余（31 个降到 8 个），不是对视频样本去重。
· Goku 用 Qwen2 作为 LLM 融合器把关键帧 caption 与视频 caption 合并为「统一、无冗余、无矛盾」的最终描述 —— 这是 caption 文本级的冗余消除，与样本级去重不同。
· LongCat-Video 用 LLM 对 caption 嵌入的聚类结果做类别命名，caption embedding 聚类具备做语义去重的技术条件，但报告未说明是否用于去重。
【生态判断】视频样本的精确去重（哈希）与语义去重（embedding）普遍在 captioner 上游完成、由生成侧团队执行且很少披露；caption 文本本身的去重（避免同质化模板句）几乎无人讨论 —— 考虑到 CogVideoX 的 prompt 明令禁止「The video presents / depicts / showcases」「throughout the video」等套话，说明 caption 同质化是真实存在的问题，但业界靠 prompt 约束而非事后去重解决。

### [几何/结构化标注数据集合集](../models/geometric_datasets.md) ⚠️

Action100M 的去重最系统且量化：在标注聚合后识别出 7.58M 个重复组、共 1.418亿条重复实例并执行去重，随后用 k-means（k=10³/10⁴/10⁵）做语义重采样以压平长尾——属于基于语义嵌入的去重+再平衡组合。SceneScribe-1M：源自 HD-VILA-100M/Panda-70M/Koala-36M/Pexels 四个库的合并，存在跨库重叠风险，论文未描述显式去重步骤[不确定]。SpatialVID：以视频ID为单位从 YouTube 采集，同一长视频切出的clip间存在天然近重复，论文未描述embedding级语义去重[不确定]。WildWorld：动作空间长尾明显（top-150 动作占 58.49% 样本），论文以动作三元组统计呈现分布而非去重，未见显式去重流程[不确定]

### [视频生成后训练数据策略](../models/post_training_data.md) ⚠️

锚论文未涉及 [不确定]。后训练阶段的去重关注点与预训练不同：不是去除重复素材，而是去除重复/含糊的 prompt。Seedance 1.0 明确在 RLHF prompt 收集后做「数据均衡与信息过滤以剔除重复与含糊 prompt」；Step-Video-T2V 邀请标注员依指引合成补充 prompt 以保证 prompt 多样性；JavisDiT++ 保证 3 万条 DPO prompt 池与 SFT 数据不重叠；VideoReward 保留 1.3 万 prompt 从未出现在训练集的三元组作验证集。
素材级去重在 SFT 阶段最典型的形态是 Step-Video-T2V 的 VideoCLIP K-means 簇内离群剔除（形式上是聚类，作用上兼顾了语义去重与离群剔除）与 LongCat-Video 的 caption 嵌入密度倒数采样（对高密度即近重复区域降采样，是一种连续化的软去重）。这两者代表了「用 embedding 空间密度替代硬去重阈值」的方向。

### [主流视频预训练数据集合并调研：Panda-70M、InternVid、Koala…](../models/pretraining_datasets.md) ⚠️

**七者的去重普遍薄弱，只有两家做了实质性工作，且都不是embedding级语义去重**：
- **Panda-70M（唯一有片段级去重机制）**：以每个clip第一阶段 ImageBind 特征的均值作为表示，**只保留与前序片段欧氏距离 >0.3 的片段**——这是切分流程内嵌的片段内冗余去重。另外在发布层面用「**每个源视频最多3条clip**」的规则做源级去重（10M子集），2M子集则是每源视频恰好3条。但**全量70M平均每源视频约18.7条clip，且数据未打散**（同一源视频的所有clip落在同一分片，仓库明确提示用户需自行shuffle），冗余风险很高。
- **UltraVideo**：在从 Koala-36M 复筛的路径中明确包含「去重」步骤（uniform per-category sampling 后 plus dedup），但**具体方法未披露**。[不确定]
- **InternVid**：**没有去重流程**，仅在爬取时排除了截至2023年4月已存在于公开数据集中的视频（这是跨数据集防重叠，不是内部去重）。
- **Koala-36M：完全没有去重。** 全文「duplicat」一词仅出现1次且在参考文献标题中（D4论文）。考虑到其48M clips 切自约1亿条 HD-VILA YouTube 视频，**近重复内容是未经处理的实质风险**。
- **MiraData / OpenVid-1M / LVD-2M：论文均未提及任何去重方法。**[不确定] 其中 LVD-2M 的风险最值得警惕——它同时使用了 **HD-VG-130M、Panda-70M、InternVid** 三个都源自 YouTube 的语料，**三者之间存在未经量化的源级重叠**（Panda-70M 来自 HD-VILA-100M，HD-VG-130M 亦为 YouTube 爬取），论文未做任何跨源去重。
- **精确去重（哈希）与语义去重（embedding聚类）**：**七者无一实现**。这是本次调研中七个数据集共同的、最一致的技术缺口，也是与工业界数据 pipeline 的显著代差。

## VLM/LLM作为质检员（多模态大模型质量打分与错配剔除，2026年从浅层打分器转向大模型语义判断的趋势）

`model_as_data_judge` · 详细程度: detailed

### [Allegro](../models/Allegro.md)

属于「浅层打分器为主、大模型仅用于打标不用于判分」的 2024 年典型形态，尚未进入 VLM-as-judge 阶段：
· 用作判据的模型均为专用小模型/打分器：LAION Aesthetics Predictor（美学）、DOVER（视频质量）、LPIPS（感知一致性）、UniMatch（光流）、CRAFT（文字检测）、CLIP（图文一致性）。
· 多模态大模型的角色是「生成者」而非「判官」：Tag2Text 负责粗粒度标签、Aria（25.3B MoE）负责细粒度 caption；Aria 并未被用来对视频质量打分、对图文错配做语义判定，或输出结构化质量维度评分。
· 唯一带语义性质的过滤是第 7 级的 CLIP 相似度阈值（≥0.17 / ≥0.20），本质仍是 embedding 余弦距离而非大模型语义判断。
因此按 2026 年的标准衡量，Allegro 属于「模型质检」的前一代范式。

### [Apollo](../models/Apollo.md) ⚠️

Apollo 的数据 pipeline 体现了明显的「大模型深度介入」特征，但其角色集中在**标注生成**侧，而非**质量裁决**侧，这与 2026 年「从浅层打分器转向大模型语义判断」的趋势只部分吻合：
【大模型承担标注】Qwen2.5-Omni（全模态大模型）同时用于语音转写与音频 caption；Gemini 2.5-Pro（闭源商用旗舰多模态大模型）用于音频 caption——引入商用 API 级大模型做大规模标注，在 8100 万条的量级下是相当高成本的选择，反映出快手对标注质量的投入优先级。视频侧使用「video expert model」（视频专家模型，未具名，推测为内部自研 VLM）。
【质量裁决仍由专用模型/信号处理指标承担】音频质量用 SNR、MOS、削波/失真/噪声检测等传统信号与感知指标；音视频对齐用 Synchformer（时序）与 ImageBind（语义）两个专用判别模型；视频质量用未具名的多维打分器。全流程中没有出现「用 VLM/LLM 打一个综合质量分」或「LLM-as-judge 做跨模态一致性裁决」的环节。
【关键缺失】与 MOVA 用 GPT-OSS-120B 显式做视觉-音频一致性冲突消解、并在 prompt 中内置反幻觉自审计字段的做法相比，Apollo 论文中没有任何关于标注幻觉抑制、跨模态串扰防御或标注结果二次校验的描述——多个模型（Whisper / SenseVoice / Qwen2.5-Omni 三个 ASR 并用）产出的结果如何仲裁与融合，论文只用「All annotations are merged into unified dense captions」一句带过，融合规则完全未公开。
【推断】三个 ASR 模型并用最可能的用途是交叉验证/投票以剔除转写不可靠的样本（这实质上是一种模型集成质检），但论文未证实此点。[不确定]

### [CineDance / CineDance-1M](../models/CineDance.md)

大模型深度介入是该 pipeline 的核心特征，且不止用于打分，更承担了「结构化语义解析」这一重任务，充分体现 2026 年从浅层打分器转向大模型语义判断的趋势：
【MLLM 用于时序截断】阶段一用多模态大模型判断长视频的片头/片尾边界，决定截断位置。
【LLM 用于叙事解析（最重的一环）】Qwen3.5-27B 依据四条电影理论规则判断哪些相邻镜头构成一条完整叙事序列——这已超出「质检」范畴，是把大模型当作电影语法解析器使用。关键工程手段是自底向上镜头索引（避免时间戳幻觉）与镜头边界对齐的 3 分钟滑窗。Tab.4 消融显示该组合达 F1 = 88.4%，且仅 3.1% 序列短于 20 秒软阈值。
【MLLM 用于视觉标注】Qwen3.5-35B-A3B（MoE 架构，35B 总参 / 3B 激活）产出镜头五维属性与描述。
【Omni 模型用于音频标注】Qwen3-Omni-30B-A3B（30B 总参 / 3B 激活）承担 ASR、音频 prompt、角色音色描述三项，并做 ASR-to-Character 绑定。
【抗幻觉的三项工程设计】① 音频标注拆成三个子任务分别调用，而非一次性输出（降低幻觉）；② ASR 阶段刻意不做说话人-角色绑定，绑定作为独立后续步骤；③ 绑定采用窗口化方案（过滤非语音区间、保持镜头完整与句子完整），使 Qwen3-Omni-30B-A3B 的绑定准确率从整片输入的 67.2% 提升到 95.4%。
【与传统 diarization 的对比（Tab.5/7）】窗口化 Qwen3-Omni 95.4% > Gemini 系列 82.8%~87.4% > DiariZen 63.1% > Pyannote-3.1 62.7%，验证了「用 Omni 大模型替代专用 diarization 工具」的路线优势。
【质量打分侧】DOVER/DNSMOS/ImageBind/SyncNet 等模型打分器全量运行但不做硬裁剪，仅存元数据。

### [CogVideoX](../models/CogVideoX.md)

CogVideoX 是「用多模态大模型充当数据质检员」这一范式的早期代表之一，且在打标与质检两端都依赖大模型：
【质检端】核心质检器是 6 个基于 Video-LLaMA 微调的视频二分类器，而非传统的浅层打分器（美学 CNN、OCR 检测器、静态检测器等）。用 20,000 条人工正负标注训练，直接对整段视频做端到端语义判断，一次性覆盖了「剪辑痕迹、运动连贯性、拍摄质量、题材类型（讲座）、文字占比、录屏来源」六个原本需要多个专用检测器才能覆盖的维度。附录 K 报告的测试准确率区间为 0.89–0.99（Low Quality 0.89、Editing 0.91、Static 0.92、Text 0.96、Screenshot 0.98、Lecture 0.99）。这种「一个 VLM 打六种语义标签」的做法，比同期依赖 LAION 美学分 + OCR + 静态检测器组合的方案更接近 2026 年的主流形态。
【打标端】GPT-4 作为 teacher 生成摘要式 caption，CogVLM 作为逐帧图像 recaption 模型，最终蒸馏到 LLaMA2（摘要加速）与 CogVLM2-Caption（端到端视频打标）。
【局限】大模型判断仍以「二分类过滤」为主，未用于打连续质量分、未做图文错配（caption-video mismatch）的语义一致性校验、也未用于生成结果的自动评审；连续分数仍由传统的光流与美学模型提供。因此可定位为「从浅层打分器转向大模型语义判断」的过渡形态。

### [Cosmos-Predict2.5](../models/Cosmos-Predict25.md)

该 pipeline 是「浅层打分器 + 大模型终审」两段式的典型代表，且论文罕见地明确说明了为什么这么排：
【VLM 作为终审质检员】「Finally, we use a vision language model (VLM) (Bai et al., 2025) to further remove clips with a set of undesirable issues with higher precision. We apply the VLM at the very end of filtering because it is computationally more expensive.」——即 VLM（Qwen2.5-VL 系）不是替代浅层打分器，而是放在链末对幸存者做高精度复审，理由是算力昂贵。这正是「先用便宜过滤器砍掉大头、再让大模型精修边界样本」的经济学设计，也是 2026 年前后行业从纯浅层打分转向大模型语义判断的过渡形态。
【VLM 作为召回核验员】Smart Spaces 域：先用搜索关键词召回可能相关的候选视频，再「used a VLM (Bai et al., 2025) to verify its relevance」——用 VLM 逐条核验主题相关性，这是把 VLM 当作语义级的主题分类/召回净化器。
【VLM 作为标注员】captioning 全部由 Qwen2.5-VL-7B 承担，领域数据改用更大参数量的同族 VLM。
【专用分类器作为分域员】内容类型分类器（内部训练，26 类 taxonomy）在过滤末端为每条 clip 分类；后训练侧则用 InternVideo2 embedding 上的多头分类器把数据切成五个 SFT 域。这两处是判别式小模型而非生成式大模型，体现「能用小模型的地方不上大模型」。
【类模型打分器】perceptual quality filter 用 DOVER 式模型、semantic artifacts filter 用 VTSS 式模型，都是学习型质量评估模型而非规则阈值。
【RL 侧的模型裁判】后训练用 VideoAlign（VLM-based reward model）作为奖励模型，从文本对齐、运动质量、视觉质量三维打分，并配套自研的 Elastic Reward Service 弹性奖励服务（支持动态扩缩容、latent 传输压缩、decode 与 inference 流水线并行、CUDA IPC 零拷贝、Redis 存结果的异步 UUID 机制）——这是把「模型当裁判」工程化到服务级的少见案例。
【领域支线的例外】五大 Physical AI 域明确省略 VLM filter（omit the VLM filter），改用领域特定过滤器子集加调参。这说明团队认为在数据来源已经可信、域已收窄的情况下，昂贵的 VLM 终审性价比不足——是一个有价值的工程判断。

### [Data-Juicer 2.0](../models/Data-Juicer.md) ⚠️

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

### [Foley-Omni](../models/Foley-Omni.md) ⚠️

Gemini 2.5 Pro 在本pipeline中承担的是「标注生成 + 存在性判别」双重角色，是2026年「大模型作为数据质检员/标注员」趋势的典型案例，但本文更进一步——它不信任大模型的判断，而是设计了独立的声学验证通道来否决大模型的输出。
【Gemini 2.5 Pro 的判别职责】接收视频帧与对应音轨（多模态双输入，而非仅视觉），被 prompt 显式指令执行 detect-then-describe 两步：先判断 Speech / Sound Effects / Music 三类组件各自是否「physically present in the clip」，仅对判定为存在的组件生成描述性 caption。这一步等价于用大模型做三分类的多标签判别 + 条件式描述生成。系统 prompt 模板见论文 Table 12。
【对大模型判断的不信任与纠偏】论文明确指出该环节存在 visual bias（视觉偏置）——多模态模型倾向于被视觉线索主导，看到乐器就标注有音乐、看到人张嘴就标注有语音、看到车辆就标注有引擎声，而实际音轨中这些成分可能根本不存在或不可闻。为此引入第三级 Bandit 源分离 + 能量门控（E(a_c) > −35 dB）作为纯声学的独立判据，凡未达能量阈值的字段一律置空。
【方法论价值】这构成一个「生成-验证」（generate-then-verify）的双路架构：验证路径不依赖任何神经网络语义理解，而是基于源分离后的物理能量测量，与生成路径的错误模式完全解耦。相比单纯依赖 VLM 打分（易受同源偏置影响）或单纯依赖信号处理（无语义理解能力），这种组合更稳健。论文称此为典型音视频数据集构建中缺失的two-path validation。
【局限】[不确定] 未报告 Bandit 验证阶段实际否决了多少比例的 Gemini 标注字段——这个数字是量化 VLM 幻觉严重程度的关键证据，缺失使得该设计的实际收益无法评估；也未做「有无声学后验证」的数据消融来证明其对最终模型性能的贡献。另外未提及使用任何模型对 caption 质量本身（描述准确性、细粒度）做二次打分或剔除。

### [Goku](../models/Goku.md)

整体仍属「专用小模型打分器」范式，尚未走到 2026 年主流的「大模型语义裁判」阶段，但已出现若干模型化质检的雏形：
【已使用的模型化判定】
  (1) DINOv2 自监督特征 —— 用余弦相似度做镜头语义一致性判定，属特征级而非语言级判断；
  (2) 美学评分模型 —— 浅层回归打分器，阈值 4.3/4.5；
  (3) RAFT 光流模型 —— 运动质量判定；
  (4) OCR 模型 —— 文字覆盖率判定；
  (5) 内部视频分类模型 —— 9 大类/86 子类语义打标，服务于分布均衡阶段，这是最接近「语义级模型裁判」的一环，但它的角色是**分类与配比**而非**质量判决**；
  (6) 多模态大模型（InternVL2.0、Tarsier2）与 LLM（Qwen2）—— 但它们仅被用于 **caption 生成与融合**，论文**未**将其用于质量打分、图文错配（text-video misalignment）检测或语义合理性剔除。
【明确缺失】论文没有 VLM 打分环节、没有 caption 与视频一致性的回验（如 CLIP score / VLM 一致性判定）、没有 LLM 对 caption 质量的审核。
【评价】Goku（2025年初）代表的是「浅层打分器 + 模型打标」阶段；相较后续 Seedance、LTX-2 等引入 VLM 作为质检员做语义级筛选与错配剔除，Goku 在这一维度是明显的时代分界前侧样本，可作为趋势演进的对照基线。

### [Hailuo / MiniMax Video](../models/Hailuo.md) ⚠️

[不确定]（完全无披露）。未提及使用 VLM/LLM 作为数据质检员。值得注意的间接背景：MiniMax 自身拥有强多模态大模型能力（MiniMax-01 系列具备 VL 能力，M3 为 427B 图文输入模型），且在 2025年12月开源了 VTP 视觉 tokenizer，技术栈上完全具备用自家全模态模型做数据质检与语义打标的条件；但公司从未公开确认视频数据流水线中是否采用了这一做法。因此本字段属于「有能力但无披露」，无法归入 2026 年「从浅层打分器转向大模型语义判断」这一趋势的证据集。

### [HunyuanVideo-Foley](../models/HunyuanVideo-Foley.md) ⚠️

HunyuanVideo-Foley 的做法处在「传统判别式小模型」与「大模型语义裁决」之间的中间状态——大量使用专用预训练模型做自动判定，但没有任何通用 VLM/LLM 参与质量裁决：
【担任「质检员」的模型清单（全部为专用模型，非通用大模型）】
- AudioBox-Aesthetics：音频感知质量四维打分，是唯一的「学习型质量评估器」，替代了传统的手工音频指标；
- ImageBind：跨模态语义一致性判定（音频 embedding 与视觉 embedding 的对齐程度）；
- AV-align：时序对齐判定；
- 语音-音乐检测模型 + 音频分类模型：类别判定；
- SNR：手工信号指标；
- 带宽/有效采样率检测：手工频谱分析。
【担任「标注员」的模型】GenAU（音频 captioning 模型），生成音频内容描述。
【没有的东西】没有任何通用多模态大模型（无 Qwen-Omni、无 GPT-4o、无 Gemini）参与数据质检、跨模态一致性裁决或错配剔除；没有 LLM 做 caption 融合、改写或幻觉自检；没有模型对 GenAU 生成的 caption 做质量复核。整条流水线是「专用模型流水线」而非「大模型流水线」。
【与 2026 年趋势的关系】相比 MOVA（用 GPT-OSS-120B 做跨模态一致性裁决与幻觉自审计）这类把通用大模型引入质检环节的做法，HunyuanVideo-Foley（2025 年 8 月）仍停留在浅层打分器阶段。但需要指出的是，其 ImageBind 语义对齐检测在功能上已经承担了「跨模态错配剔除」的职责——只是用 embedding 相似度而非语言化推理来实现，成本低得多但可解释性与细粒度判断能力弱得多。在 10 万小时（约 4,500 万片段）的规模下，用大模型逐片段裁决在算力上不现实，这是规模对方法的硬约束。
【自动化程度】论文摘要明确把 pipeline 描述为「通过自动化标注」（through automated annotation）构建，即全流程无人工介入。
【幻觉与误判防护】未描述任何针对 GenAU caption 幻觉的检测或约束，未说明各判别模型的误判率或人工校验结果。[不确定]

### [HunyuanVideo](../models/HunyuanVideo.md) ⚠️

两代都大量使用模型作为质检员，但形态不同，且都不是「用通用大 VLM 做端到端语义评判」：
【原版】使用的是一组专用小模型/判别模型组成的评审团——Dover（美学+技术双视角）、自研 blur 检测模型、OCR 模型、YOLOX-like 检测模型、光流模型、Transnet v2 场景检测、自研 VideoCLIP（去重与概念均衡）、自研镜头运动分类器（14类）。属于典型的 2024 年「浅层多打分器」范式。真正的大模型只用在打标环节（自研 VLM 生成结构化 caption）与 prompt 改写（Hunyuan-Large）。
【1.5】向「模型化算子」演进但仍未公开使用通用大 VLM 判分：新增了专门训练的「转场分类器」、四维度「综合视觉质量评估算子」、美学打分算子、镜头运动识别模型（clip级+时序级）。最能体现大模型参与数据构建的是 caption 侧：对 caption 模型本身用 OPA-DPO 强化学习做后训练，专门优化「描述丰富度 vs 事实准确性」的权衡以压制幻觉——即用 RL 保证数据标注器的可信度，这是把「模型作为标注员」的可靠性问题当作一等公民来处理，属于该维度上较前沿的做法。是否用 VLM 对 video-caption 错配做剔除，两份报告均未说明。[不确定]

### [InstructAV2AV](../models/InstructAV2AV.md) ⚠️

多模态大模型在本pipeline中承担双重角色——既是「指令生成器」（阶段二）又是「质量裁判」（阶段三），两处均使用 Qwen3-Omni（Xu et al., 2025b），是2026年「大模型深度参与数据构造全链路」趋势的典型样本。

【角色一：指令生成器】Qwen3-Omni 以 Grounded-SAM-2 输出的实例级 mask 与源音视频上下文为条件，生成多样化的文本编辑指令（formulate diverse text instructions for comprehensive editing operations）。选用 Omni 系列（原生支持文本+图像+视频+音频全模态输入）而非纯视觉 VLM 是必要的——指令必须同时基于「画面里有什么可编辑对象」和「音轨里这个对象发什么声」两方面信息才能生成合理的音视频联合编辑指令（如「把狗换成猫」隐含着「狗叫要换成猫叫」）。

【角色二：质量裁判】对每条合成的 target 沿五维度打分（指令保真度 / 内容保持度 / 感知质量 / 音视频同步 / 安全性），采取「与」逻辑——仅保留同时通过全部五项的样本（retain only the samples that simultaneously pass all five criteria）。这是严格的合取式筛选而非加权求和，任一维度不合格即淘汰，取向偏保守。

【与 Foley-Omni 双路验证范式的对比】Foley-Omni 用 Gemini 2.5 Pro 生成标注后，特意引入独立的纯声学通道（Bandit 源分离 + 能量门控）来否决大模型的幻觉，理由是「生成路径与验证路径必须错误模式解耦」。本工作则是同一个 Qwen3-Omni 既生成指令又验收结果，两个环节共享同一模型的偏置——若 Qwen3-Omni 生成了一条它自己难以判别执行质量的指令（例如某类细粒度属性变更），它在验收时同样可能给出错误判断，形成自洽的盲区。这是本pipeline在验证独立性上相对 Foley-Omni 的一个方法论弱点。缓解因素是评测集额外经过 20 人的人工五维度复核，以及外部 AvED-Bench 的零样本验证。

【人机验证的分工设计】自动验证覆盖全部 79K 训练样本（保规模），人工验证只施加于 1K 评测样本（保可信度）。人工侧的组织方式值得记录：20 名志愿者编为 10 个评审对（judge pair），每个候选样本由 5 个独立评审对分别评估，每对只负责五个维度中的一个——即「一对评委只看一个维度」的分维度独立评审，避免了单人同时评多维度时的光环效应与维度间相互污染。这一设计在同类工作中较为细致。

[不确定] 未披露：打分的分值尺度（是二分通过/不通过还是连续分数+阈值）、各维度的通过门槛、prompt 模板、验证阶段的实际淘汰率（这是量化数据引擎成功率的关键，缺失）、Qwen3-Omni 判断与人工判断的一致性检验（若在 1K 人工集上对比自动与人工结论的吻合率，本可有力佐证自动验证的可靠性，但未做）。

### [2026 其他音视频联合生成](../models/JAVG_2026_misc.md) ⚠️

本维度是本批工作最能体现 2026 年趋势的部分——ALIVE 与 NAVA 都把 MLLM/VLM 深度嵌入数据管线，且不止用于打标，而是直接承担质检与语义判断职能：
【ALIVE —— MLLM 做音频质检与说话人归属校正（本批最深度的应用）】
(1) MLLM 做音频双准则质检：原文「We utilize an MLLM model ... to perform audio filtering based on two criteria. Firstly, for audio quality, the model assigns a score to each audio, allowing us to discard samples with significant background noise. Secondly, for audio-visual coherence, we assess the correlation ...」——即用多模态大模型同时承担两件事：给音频质量打分（剔除显著背景噪声）、判断音画语义相关性（分离强相关样本、管控 BGM 等弱相关样本占比）。这是一个标志性转变：音频质量过滤传统上靠 SNR、静音检测等信号处理指标，ALIVE 改用大模型直接「听」并打分；音画相关性传统上靠 CLAP/ImageBind 等对比学习模型算相似度，ALIVE 改用大模型做语义判断。
(2) Qwen3-omni 做旁白时间戳解析：在 SubjectID correction 管线中，用 Qwen3-omni（全模态大模型）解析 narration 的时间戳——这是把全模态 LLM 当作时序对齐标注器使用。
(3) Gemini 2.5 参与管线的其他环节（论文提及但角色未完全明确[不确定]）。
(4) 传统打分器仍在并用：OCR、预训练美学模型、RAFT、TS-TalkNet、CLIP、ArcFace、13 维质量分类器——构成「大模型语义判断 + 专用打分器」的混合体系，而非完全取代。
【NAVA —— VLM 过滤 + 三个 Gemini/Qwen 模型分工打标】
(1) VLM-based filtering and tagging：用视觉语言模型筛选并标注视觉质量清晰的片段，是显式的「VLM 作质检员」。
(2) omni-modal tagger 配合 YAMNet 做五类音频分类——传统音频分类器（YAMNet）与全模态大模型标注器协同，前者提供快速粗分类、后者提供语义细化。
(3) 打标环节三模型分工：Qwen3-VL（视频 caption）、Qwen3-Omni（音频 caption）、Gemini-3-Flash（融合改写）；高质量子集升级为 Gemini-3-Pro。用不同档位的模型处理不同质量档位的数据，是成本-质量权衡的典型工程实践。
(4) 「multi-operator collaborative filtering」（多算子协同过滤）产出 160K SFT 子集——「协同」一词暗示各算子分数被联合而非串行阈值化处理[具体聚合方式不确定]。
【OmniCustom】用 GLM-ASR 做转写，属专用模型而非通用 MLLM；caption 构造沿用 Ovi 的属性体系（说话人年龄、性别、口音、音高、韵律、情绪、语速），未描述用何模型生成这些属性标注[不确定]，也未用大模型做质检。过滤全靠 SyncNet + 美学分 + 时长三条硬规则。
【CCL / Baton / StreamChar】均未使用大模型做数据质检[不确定]。Baton 虽有 Qwen3-8B 作为 VA-Planner 底座，但那是模型组件而非数据管线角色。
【ITS-JAVG —— 把「模型判官」范式化并证明其陷阱（对本维度最有理论价值的贡献）】
虽不涉及训练数据，但其核心研究对象正是「模型作为判官」这一范式本身，结论对数据质检有直接迁移价值：
(1) 六个验证器各司其职：VideoReward-TA（文本-视频一致性）、JavisScore（细粒度音视频同步）、ImageBind-TA（文本与生成音频的语义连贯）、AVHScore（音频事件与视觉事件的语义一致性）、VQAScore（文本-视频对齐）、ImageBind（音视频语义相似度）。
(2) 核心发现一：单验证器必然导致非对称权衡——「single-verifier guidance effectively improves its intended evaluation metrics, yet fails to achieve a balanced improvement across all metrics」，即只优化某个判官的分数会牺牲其他维度。
(3) 核心发现二：verifier hacking（验证器钻空子）真实存在——搜索算法会「exploit blind spots」（利用判官的盲区）。这对训练侧数据过滤是重要警示：若用单一模型打分器做严格过滤，被保留的数据会系统性偏向该打分器的偏好与盲区，形成隐性的数据分布扭曲。
(4) 解法 ARW：把奖励聚合当作在线优化问题，公式 R(i)=Σ_k w_k · r_k(i)/(σ_k+ε)（按各判官分数的标准差归一化后加权），优化目标 L_ARW=Σ_k (½exp(−s_k)·Var̂(r_k) + ½|s_k|)——即自适应地按各判官分数的方差调整权重，方差大（区分度高）的判官权重高。这套「多判官分数的自适应归一化聚合」方法论完全可以移植到训练数据的多算子协同过滤上（恰好 NAVA 用的正是「multi-operator collaborative filtering」，但未公开聚合算法）。

### [音视频联合生成基线合集](../models/JavisDiT_baselines.md) ⚠️

这一维度上，本合集恰好完整呈现了「无模型判官 → 专用判别模型 → 大模型语义判官」的三代演进：
【第一代：无模型判官（MM-Diffusion 2022 / AV-DiT 2024）】数据侧完全不使用模型做质检或打分，靠数据集选型与小规模人工把控质量。模型仅用于评测（i3d 算 FVD、AudioCLIP 算 FAD），不参与数据筛选。
【第二代：专用判别模型作为过滤器（JavisDiT++ 2026）】使用一组浅层/专用模型作为过滤器与打分器，符合 2024–2025 年主流做法：
- FunASR（阿里开源的语音识别工具包）作为「语音存在性检测器」，用于剔除含人类语音的视频——这是一个把 ASR 模型当作二分类过滤器的巧妙用法。
- 美学评分模型（aesthetic predictor）作为质量判官。
- 光流模型作为运动判官。
- OCR 模型作为文字污染判官。
【第二代半：多奖励模型集成作为偏好判官（JavisDiT++ 的 AV-DPO，本合集最有价值的实践）】用六个模型分工打分构造偏好对，是「model-as-judge」用于数据构造而非过滤的典型案例：
- 音频质量 → AudioBox（AudioBox-Aesthetics）
- 文本-音频对齐 → ImageBind
- 视频质量 → VideoAlign
- 文本-视频对齐 → ImageBind
- 音视频跨模态相似度 → ImageBind
- 时序同步性 → Synchformer
并采用「归一化的模态感知排序（normalized modality-aware ranking）」选取 winning/losing 对，作者明确说明这样做是为了「保证每个模态内部的一致性，避免把优质音频与劣质视频混搭配对」——这是多维奖励下构造偏好数据的关键工程经验。
【第三代：多模态大模型作为标注/判官（Harmony 2025 与 JavisDiT 的 JavisBench）】
- Harmony：用 Google Gemini 对全部 400 万条片段做自动标注（ASR 转写 + 视频描述 + 背景音描述），并用一个「音视频一致性打分模型」筛选语音数据——后者是把跨模态一致性判断交给模型的直接实践，但该模型的身份、打分维度与阈值均未披露[不确定]。
- JavisDiT 的 JavisBench：用「先进的 Qwen 系列模型（advanced Qwen-family models）」同时完成 caption 生成与 19 类场景归类，即 MLLM 既做标注器又做分类判官。
【UniAVGen】未提及任何模型化数据判官[不确定]。

### [可灵 3.0 / 可灵视频 3.0 Omni](../models/Kling_30_Omni.md) ⚠️

是，且是可灵3.0数据体系的核心环节，体现2026年“从浅层打分器转向大模型语义判断”的趋势。Kling-Omni 第三级“多模态对齐”完全由模型担任judge：判定视频caption与实际视觉内容的语义一致性、参考图对目标视频的保真度、以及以人为中心任务的角色身份一致性（跨帧ID一致性校验）。合成数据侧也由“专家模型”驱动（自研图像编辑模型 + 视频理解模型）。同团队旁证：Koala-36M 用训练好的 VTSS 神经网络替代人工阈值组合、用微调 LLaVA 生成结构化长caption；Kling-Foley 用 CLAP 计算音频与文本标签的一致性并只保留高一致性数据、用音频理解大模型分类生成描述、用LLM抽取元数据并融合视听双描述。可灵官方亦公开表述其数据方案覆盖“海量视频挖掘、多维标注与筛选、视频描述增强、数据驱动的质量评估”。[不确定：担任judge的自研VLM的具体名称与参数规模]

### [LTX-2](../models/LTX-2.md) ⚠️

LTX 系列未使用 VLM/LLM 作为数据质检员，这是其与2026年主流趋势的显著差异点。数据 pipeline 中承担判别职责的全部是浅层/专用模型：CLIP 图像编码器（用于去重表征）、多标签分类网络（用于采样配对）、Siamese 美学排序网络（用于质量打分）、运动量估计器。大模型在 LTX-2 中的角色集中在两处，但都不是质检：(1) 内部自研 captioner（视频+音频双轨打标）；(2) Gemma3-12B 作为冻结文本编码器参与条件建模。技术报告未提及用多模态大模型做质量语义打分或图文/音画错配剔除。是否在未公开的内部流程中使用，无从判断。[不确定]

### [LongCat-Video](../models/LongCat-Video.md)

该模型大量使用多模态大模型作为标注员与质检员，是其数据流程中的核心手段，但更偏「模型打标」而非「模型判优剔除」：
(1) Caption 生成：微调版 LLaVA-Video（用内部合成 pair 微调）为主 captioner，并引入 Tarsier2 的标注以增强时序理解；
(2) Qwen2.5VL 承担两类结构化判别——景别（shot size）与镜头类型（lens type）的判定，以及视觉风格判定（写实度 realism、动画风格 animation styles、色调 color tones）；
(3) 相机运动（pan/tilt/zoom）用专门训练的分类器而非 VLM；
(4) caption 嵌入聚类后由 LLM 做类目归纳与命名，即用 LLM 定义数据类目体系；
(5) 后训练侧的奖励模型本质也是模型质检员：VideoAlign 基座微调出的 Motion Quality 模型（输入灰度视频，强制其只看运动不看色彩）与 Text-Alignment 模型（输入彩色视频），以及 HPSv3-general/HPSv3-percentile 视觉质量打分。
Avatar 1.5 进一步把 VLM 用作硬过滤器与一致性裁判：静默数据用 Qwen3-Omni 初判 + Qwen3-VL 复判，两模型一致才保留；情绪数据用 EmotiEffLib 以置信度 s>0.7 为阈值精筛，并设硬性排除规则（含合成内容、超过两个主体、身份切换、主体在画面中占比过小者一律标为 null）；caption 侧用 Qwen3-Omni 生成「空间环境 / 人物关系 / 情节推进」三维上下文描述，并要求描述聚焦客观物理表现。
报告未说明基础版是否用 VLM 做过 caption-视频错配（mismatch）的显式剔除。

### [MOVA](../models/MOVA.md)

MOVA 体现了 2026 年“大模型深度介入数据环节”的典型形态，但其分工是清晰切分的——**大模型负责语义标注与跨模态一致性裁决，专用小模型负责质量与对齐打分**：
【大模型作为语义裁决者（核心用法）】GPT-OSS-120B 在 caption 合并环节承担的不只是文本拼接，而是显式的跨模态一致性检查：“the model verifies the alignment between visual scenes and audio events to resolve potential conflicts”（校验视觉场景与音频事件的一致性，消解潜在冲突），再综合为统一自然语言描述。这实质上是用 120B 级 LLM 做“视听内容语义是否自洽”的判官，是本 pipeline 中最接近“LLM-as-judge”的组件。
【反幻觉的自审计设计】视觉与语音标注 prompt 都内置了 final_verification_audit 自检字段（hallucination_check_passed、visual_changes_verified / speech_dynamics_verified 及 comment），要求 MLLM 输出结构化的自我审计结论；并用强约束“LAW”体系抑制跨模态串扰——视觉标注器被明令“Ignore audio and inferred context entirely”“Do not infer or hallucinate based on audio or context”，语音转写器被明令“Ignore non-speech sounds and music entirely”。这是针对多模态标注器最典型失效模式（用一个模态臆测另一个模态）的防御性设计。
【质量过滤仍由专用打分器承担】音频用 Audiobox-aesthetics（PQ/CU/CE 三分数），视频用 DOVER（技术/美学双分数），对齐用 SynchFormer（时序）与 ImageBind（语义），音频分类用 EAT，唇音用 SyncNet 系的 LSE-D/LSE-C。这些均为专用判别模型而非通用 VLM。
【结论】MOVA 并未走向“用 VLM 打一个综合质量分替代所有浅层打分器”的路线，而是采取“专用打分器管质量与对齐 + 大模型管语义标注与一致性”的混合分工，可视为该趋势的一种务实折中。
【局限】论文在 Limitations 中承认，标注可靠性是多说话人场景的瓶颈：说话人分离（diarization）错误与不完善的 active-speaker 标签会传播到训练，导致模型混淆说话人或学到不一致的监督信号。

### [合并条目：① Mochi 1](../models/Mochi_MAGI_Motif.md) ⚠️

本组三个模型恰好完整呈现了「VLM 作为质检员」从缺席到成为管线枢纽的演进，是本条目对照价值最高的一个维度：
① Mochi 1（2024.10）：无任何披露，也无证据表明使用了大模型判分。[不确定]
② MAGI-1（2025.05，MLLM 作为「高级过滤器」，位置在漏斗末段）：报告设有独立小节 3.3「MLLM as Advanced Filter」，逻辑非常清晰——前面的 Filter Actors 与去重已滤掉绝大部分不良数据，但受限于这些过滤器的能力，仍残留少量低质数据；此时数据量已显著缩小，因此引入多模态大模型再做一轮过滤，「enables us to detect more complex bad cases（使我们能够检出更复杂的坏样本）」。最关键的成本设计是：「Notably, this step can be seamlessly integrated into the subsequent caption procedure, thereby reducing overall costs and improving efficiency（值得注意的是，该步骤可无缝并入后续的打标流程，从而降低总成本并提升效率）」——即判分与打标复用同一次 MLLM 调用。但论文未点名所用 MLLM 的具体型号与规模，也未说明它判断哪些维度、输出何种结构。[部分不确定：MLLM 型号与判定维度]
③ Motif-Video 2B（2026.04，VLM 从「末段复筛」升级为「贯穿全程的元数据源」）：其 caption-as-metadata 设计把 VLM 判分推到了新的形态——Qwen3-VL-30B-A3B 的单次前向按固定 JSON schema 同时输出自由文本 caption 与结构化标签（subject、style、action、camera_move、quality、watermark、nsfw，以及 padded、multi_scene、timelapse），这些标签随后被四路复用：(i) 训练时的文本条件；(ii) sanitation（nsfw / watermark / padded / multi_scene / timelapse / quality 直接触发硬删除）；(iii) 领域与主体均衡采样（style / subject 驱动 720p 与 SFT 的 domain balancing）；(iv) 720p SFT 的动态运动过滤（action=Dynamic）。且论文明确「我们不再用一个单独的后处理模型去重新解释这些标签（we do not apply a separate post-processing model to reinterpret them）」，标签由 VLM 直接消费。其方法论上的关键论断是：「因为这些标签与训练 caption 产自同一次前向，过滤与条件化在构造上始终同步（filtering and conditioning remain synchronized by construction）」——这是把「模型质检」与「模型打标」彻底合一、并以此消除两者语义漂移的明确设计主张，代表了 2026 年该范式的成熟形态。
三者连起来看：Mochi 1（无）→ MAGI-1（MLLM 末段复筛、与打标复用调用以省成本）→ Motif（VLM 前向即元数据源、过滤与条件化构造性同步），演进方向是 VLM 在管线中的位置不断前移、职能不断扩张。

### [Movie Gen](../models/Movie_Gen.md)

Movie Gen 属于「2024年范式」的代表：数据质检几乎完全依赖一组专用的轻量判别模型/分类器与打分器，而非用通用VLM/LLM做整体语义质量判断与图文错配剔除；大模型（LLaMa3-Video）在这里承担的是打标（caption生成）角色而非judge角色。具体的模型清单：
视觉侧——内部OCR模型（词检测+词识别双分数）；自研黑边检测器（非学习式）；LAION aesthetics 图像美学预测模型；若干自训的帧级视觉模型（美学/画质/大边框/视觉特效）；内部静态视频检测模型；PySceneDetect 与 FFmpeg（场景/运动）；copy-detection embedding 模型（SSCD）；视频-文本联合embedding模型（聚类与k-NN检索）；16类相机运动分类器；Detic 物体检测模型（剔小主体）；ArcFace 人脸识别模型（PT2V身份一致性，帧间阈值0.5、合成参考图阈值0.7）；人脸检测与人脸区域分割模型。
音频侧——AED 音频事件检测模型（AudioSet 527类本体）；CAVTP 对比式音-视-文预训练模型（输出音视频余弦相似度，用于diegetic判定与分桶）；音频质量预测模型（输出1~10连续分，标注方式仿照 LAION aesthetic）；通用音频caption模型；音乐caption模型；影视感（cinematic）音视频分类器。
值得注意的是，音频质量分并非用作硬过滤阈值，而是被写进caption成为可控条件（推理时指定质量7.0/8.0），这是一种「把judge分数转成条件」的做法。而后训练阶段的语义级判断则交给人类标注员，而非大模型。

### [NVIDIA NeMo Curator](../models/NeMo_Curator.md)

呈现出明确的代际演进，是观察「浅层打分器 → 大模型语义判断」这一 2026 趋势的好样本：
【25.09/早期 Cosmos 生产版：浅层判别器为主】质检职责由轻量模型承担——CLIP-based 美学模型、DOVER 画质模型、InternVideo2 嵌入 + MLP（文字叠加检测、视频类型分类）、ViT 光流运动分类器、运动向量统计。全部是小模型或线性/MLP 探针，设计动机是 PB 级规模下的单位成本。
【26.02–26.07：VLM/LLM 逐步进入流水线】(1) captioning 后端从 Qwen2.5-VL 扩展到 Qwen3-VL 与 Nemotron Nano 12B V2 VLM、Nemotron 3 Nano Omni，并提供 BF16/FP8/NVFP4 三档精度以压低大模型推理成本——FP8/NVFP4 量化是让 12B 级 VLM 在 PB 级数据上跑得起的前提；(2) 26.04 引入 vLLM 作为默认推理后端并内置 Ray Serve 推理服务器（OpenAI 兼容接口），使「在 pipeline 中挂一个 LLM 做判断」成为一等公民；(3) 26.02 起 captioning 支持可选的 Qwen-LM 二次改写增强；(4) 26.07 的 Nemotron OCR 合成数据流水线中，明确使用 Nemotron Nano Omni 做可选的质量打分（quality scoring）——这是官方文档中最直接的「大模型作为数据质检员」用法。
【仍然缺失】视频侧尚无「VLM 判断 caption 与画面是否错配」的专用剔除 stage，也无 VLM 语义质量打分 stage；大模型目前主要用于生成 caption 而非审核 caption。这一环需用户自建。

### [OmniHuman 数据集 + OHBench](../models/OmniHuman.md) ⚠️

OmniHuman 是本次调研样本中最贴合「2026 年大模型作质检员」趋势的案例之一，且其大模型的用法有明显的层次划分：
【Gemini-3：语义裁决者】用于「背景合理性与交互评估」（background plausibility and interaction assessment）——即由闭源顶级多模态模型对场景的语义合理性与人物交互的自然度做判断。这是典型的用大模型替代浅层打分器做语义层判断的做法，也是浅层指标（美学分、清晰度）根本无法覆盖的维度。
【Gemini-3 / Gemini-3-pro：评测端的主力裁判】在 OHBench 中承担四项 1–10 分制的主观维度打分：Background Plausibility（背景合理性，全局层）、Interaction Naturalness（交互自然度，含眼神接触、肢体手势、回应准确性，关系层）、Object Consistency（对象一致性，外观/位置/状态的时序稳定性）、Contact Naturalness（接触自然度，空间准确性/时序同步/受力真实性）、Attribute Consistency（属性一致性，个体层）。这是本工作最激进的一步——把传统上必须靠人工评分的语义/物理合理性维度，整体交给 Gemini-3 打分，并以此作为「与人类感知高度一致」的指标基础。
【Qwen3-Omni：标注生成者而非裁决者】作为 caption 生成的推理核心，负责提取属性与合成叙事，不承担质量裁决。
【最有价值的设计：交叉校验闭环】流水线末端的一致性校验是本工作在「防 MLLM 幻觉」上的实质性设计，且不依赖更大的模型而是依赖模态间的冗余：
- 结构化 subject 标签的数量必须与跟踪模块（YOLOv11 + MOTRv2）输出的人数精确一致——用视觉几何管线校验 MLLM 的计数幻觉；
- caption 中的说话人数量与转写内容必须与上游 ASR（FunASR-Nano）转写保持在可接受的编辑距离范围内——用 ASR 结果校验 MLLM 的语音内容幻觉；
- 只有全部通过校验的视频才被保留。
这种「用确定性专用模型的输出去约束生成式大模型的输出」的做法，比 MOVA 用 GPT-OSS-120B 做 hallucination_check（即用大模型查大模型）在原理上更可靠，因为跟踪与 ASR 的错误模式与 MLLM 的幻觉模式是独立的。
【局限】编辑距离的「可接受边界」数值未给出；Gemini-3 的判定阈值与 prompt 未公开；对闭源 API 模型（Gemini-3）的强依赖使得整条流水线的可复现性与长期稳定性受制于第三方服务，且百万级样本调用闭源 API 的成本论文未提及。[不确定]

### [Open-Sora 系列](../models/Open-Sora.md)

两者都**停留在「浅层打分器」阶段，尚未转向「大模型语义判断」范式**——所有过滤判据（美学分、光流/VMAF/LPIPS 运动分、Laplacian 模糊、DOVER 技术分、OCR 文字数）都是轻量专用模型或传统 CV 算子输出的标量分数 + 阈值，没有任何一级过滤是由 VLM/LLM 做语义级判断。
唯一接近「模型作判官」的机制是 **Open-Sora 1.x 的 matching score（图文匹配分）**：用 **CLIP** 计算视频中间帧与 caption 的余弦相似度（`tools/scoring/matching/inference.py`，输出列 `match`），用于剔除图文错配样本。但 CLIP 的语义判别力远弱于当代 VLM 判官，且只取单帧、不看时序。
VLM 在两个项目中的角色是**打标器而非判官**（PLLaVA-13B / LLaVA-Video / Qwen2-VL-7B / Qwen2.5-Max 只负责生成 caption，其输出不被用来给样本打质量分或做剔除决策）。Open-Sora Plan 的 Qwen2-VL 打标后处理也只是用 28 条常见开场短语的规则表去掉 「The video shows」 之类前缀，属规则清洗而非模型判断。
对比意义：这正是 2024–2025 年开源复现项目与 2026 年前沿实践之间的主要代差所在——Open-Sora 系列的 pipeline 可视为「VLM-as-judge 之前」的完整技术快照。

### [Ovi](../models/Ovi.md) ⚠️

Ovi 处于「MLLM 用于打标而非用于质检打分」的阶段，这与 2026 年主流的「大模型语义判官」路线只有部分重合。
【已使用的模型化判断】
(1) MLLM 作为标注器：输入 7 帧均匀采样图像 + 完整音轨，产出交织了视觉事件与台词的长 caption 及结构化音频描述。论文强调「conducted extensive experiments to ensure the captioning included all relevant visual and audio events while respecting chronology」——即通过大量 prompt 实验来保证事件完整性与时间顺序正确性，这是对 MLLM 输出质量的间接管控，但未描述用第二个模型去校验 caption 与内容是否错配。
(2) 判别式模型作为过滤器：SyncNet（同步性判官）、RAFT（运动判官）、LAION aesthetic predictor（美学判官）、内部人脸检测模型（构成判官）——全部是浅层/专用打分器，符合 2025 年中期的典型做法。
【未使用的手段】未使用 VLM/LLM 对片段做整体质量打分、未用大模型做「caption 与视频语义错配」的二次剔除、未用大模型做内容合规判断、未有 model-as-judge 的评分阈值。这一维度整体上是 Ovi 相对同期强 pipeline（如 HunyuanVideo 用 MLLM 替代 T5 并配 robust data filter）的薄弱环节[不确定：是否存在未公开的内部 MLLM 质检环节]。
【MLLM 身份】论文全程只写「an MLLM」，未指明是 GPT-4o/Gemini/自研模型或其规模[不确定]。

### [Script-a-Video](../models/Script-a-Video.md) ⚠️

本工作是「大模型作为标注器」的典型案例，但严格来说它并未把大模型用作质检员（judge of data quality），而是用作生成器（generator）与评测裁判（judge of output）两种角色：
【角色 1：Gemini-2.5-Pro 作为标注生成器】500K 片段的 MTSS 标注全部由 Gemini-2.5-Pro 自动生成，无其他模型参与、无多模型投票、无一致性交叉校验。选用 Gemini-2.5-Pro 的合理性由 Table 1 佐证：它在 UGC-VideoCap 上综合分 93.97（MTSS 版本 95.51），Video-SALMONN-2 总错误率 0.3959（MTSS 版本 0.2511），是全表最强模型，作为教师模型质量有保障。
【角色 2：Gemini-2.5-Pro 作为评测裁判】Table 1 与 Table 2 的所有 caption 与推理基准评测均以 Gemini-2.5-Pro 作为 judge model。此处存在一个方法论上的自指风险：训练数据的标注者与评测的裁判者是同一个模型，可能对 MTSS 格式存在系统性偏好，论文未讨论或缓解这一潜在偏差。
【角色 3：Gemma4-31B-it 作为生成质量裁判】生成侧的 Semantic Following 指标由 Gemma4-31B-it 视觉语言模型独立打分，从 Subject（身份与外观保真）、Action（动作与交互正确性）、Scene（背景/环境/空间布局准确性）、Style（视觉风格/配色/氛围的遵循度）四个子维度评分后取算术平均。这是一个 VLM-as-judge 的标准用法。
【缺失的环节：标注质检】论文未描述任何对 Gemini-2.5-Pro 标注结果的质量把关机制——无幻觉检测、无格式合法性校验（MTSS 是 JSON 式结构，理论上需要解析校验）、无 Reference ID 引用完整性校验（Shot 的 references_in_shot 与 Event 的 speaker 是否都指向存在的 ID）、无时间戳合法性校验（time_range 是否越界/重叠）、无人工抽检。对于一个强依赖交叉引用完整性的结构化 schema 而言，这是一个较明显的方法论缺口。[不确定]
【论文自陈的相关局限】Limitations 章节承认：生成准确的深度结构化脚本对基础模型的跨模态理解能力要求极高；当前开源 MLLM 在精确时序定位、鲁棒 ASR、准确的视听实体-事件关联三方面仍有局限；如何让更紧凑的开源架构达到 Gemini 级脚本能力并有效抑制幻觉，仍是未解难题。这一表述反向印证了标注质量高度依赖教师模型本身，而非依赖后置质检。

### [Seedance 2.0 与 Seedance 1.5 pro](../models/Seedance_20_Seedance_15_pro.md) ⚠️

Seedance 系列已实质走向「模型即质检员」，但披露粒度有限：Seedance 1.0 使用专用视觉质量模型做缺陷判定、内部视频表征模型做语义聚类去重、Tarsier2 基座的自研 caption 模型做内容理解、以及以 VLM 为架构的 Foundational Reward Model 做图文对齐与结构稳定性评估。Seedance 1.5 pro 称其字幕系统能为视频与音频两种模态提供「丰富的专业级描述（rich, professional-grade descriptions）」，属于大模型语义判断路线。Seedance 2.0 报告未披露质检模型细节，但同团队具备 Seed-VL 系列多模态理解模型（引言列出用于跨模态语义理解），推测已用于数据判别环节。SeedVideoBench 2.0 评测层面明确区分「客观指标走自动化流水线、主观指标走盲审专家评审」。[不确定：1.5/2.0 是否引入 VLM 打分器及其规模、打分维度与剔除阈值]

### [SkyReels 系列](../models/SkyReels.md) ⚠️

SkyReels 系列是「大模型下沉到数据环节」趋势的典型代表，且两代的角色定位有明显演进：
【SkyReels-V2（2025）】大模型主要承担打标而非质检：通用 MLLM（Qwen2.5-VL-72B-Instruct 作为通用描述来源）+ 三个子专家模型（镜头 captioner、表情 captioner、相机运动 captioner）的知识被蒸馏进 SkyCaptioner-V1（Qwen2.5-VL-7B-Instruct 基座）。质检职责仍由浅层/专用判别器承担：美学过滤器、OCR 过滤器、马赛克过滤器、特效贴纸过滤器、VQA/IQA 模型等。
【SkyReels-V4（2026）】全模态大模型深度介入数据链路的三个关键位置：
(1) 切分环节——VLM 与 TransNet 镜头边界预测结合，判断片段的语义完整性（用 VLM 决定「切在哪里」，而非仅靠像素级场景切换）；
(2) 音频分类与打标环节——Qwen3-Omni 统一完成四类音频判别与音频 caption 生成，取代传统音频事件分类器；
(3) 转写环节——Whisper 完成语音/歌唱内容识别。
这一变化印证了「从浅层打分器转向大模型语义判断」的行业趋势。但论文未描述用 VLM 对样本做质量语义打分或图文/音画错配剔除的显式环节，也未给出大模型判别的准确率与算力成本。[不确定]

### [Sora 2](../models/Sora_2.md) ⚠️

训练数据侧是否使用 VLM/LLM 作为质检员，未披露。但在推理安全侧，OpenAI 明确使用了大模型作为判官：输出阻断环节部署「a safety-focused reasoning monitor... a multimodal reasoning model which is custom-trained to reason about content policies」（一个为内容政策推理而定制训练的多模态推理模型），对生成视频帧、场景描述文本、音频转写进行语义级判断。这证明 OpenAI 已具备并部署了「大模型语义判断」能力栈，将同类能力复用于训练数据质检在技术上顺理成章，但 System Card 未做此陈述。此外安全评测流程中，对抗prompt由「helpful-only 版本视频模型」生成输出后再打分并转化为自动化评测，也体现了模型参与数据构造与评判的范式。[不确定]

### [SpeakerVid-5M](../models/SpeakerVid-5M.md)

SpeakerVid-5M 处于「大模型深度参与标注、但质量裁决仍由专用模型承担」的中间形态，与 MOVA 的分工格局类似：
【MLLM 作为打分员的实际用法（唯一一处）】Qwen2.5-VL 被用作运动强度评分器，采用三 persona 集成打分（专家/观众/标注专家三种人格视角，各自 1–5 分制），剔除离群后取均值。这是把主观、难以用浅层指标刻画的「动作幅度与交互活跃度」交给 MLLM 判断的典型案例，且用多视角集成缓解单次打分的不稳定性——相比单 prompt 打分是更审慎的做法。该分数随后成为 HQ 子集的硬性筛选条件之一（> 2）。
【LLM 作为分类器】Qwen-3 被用于对话主题类目（dialogue topic category）的分类标注。
【MLLM 作为描述器】Qwen2.5-VL 生成结构化 caption（相机运动、实体列表、身体朝向、半身/全身、表情、动作描述等）。
【质量裁决仍由专用模型承担】DOVER（视频质量）、SyncNet（唇音同步）、ArcFace（人脸身份）、Whisper（ASR 置信度与无语音概率）、Laplacian 方差（模糊）、自定义 clarity 公式（清晰度）——这六类硬性过滤全部由专用判别模型或传统信号指标完成，没有任何一项由 VLM 出综合质量分替代。
【未采用的做法】没有 LLM-as-judge 式的跨模态一致性校验（对比 MOVA 用 GPT-OSS-120B 做视听语义自洽裁决）；没有 caption 幻觉自审计机制；没有用 MLLM 复核过滤结果。
【总体定位】MLLM 在此负责「标注与主观打分」，不负责「质量判决与错配剔除」，属该趋势的早期/保守形态。

### [Step-Video-T2V](../models/Step-Video-T2V.md) ⚠️

本条目在这一维度上明显偏「传统打分器」而非「大模型语义评判」，是观察 2025 年初到 2026 年范式迁移的一个基线样本：
【承担质检的模型清单】全部为专用小模型或经典算法——LAION CLIP 美学预测器、CLIP ViT-L/14 NSFW 二分类器、EfficientNet 水印分类器、PaddleOCR、Laplacian 方差、HSV 统计、FFmpeg 黑边检测、Farneback 光流、内部 VideoCLIP（聚类与离群检测）、CLIP Score（图文对齐）。没有任何一个环节使用多模态大模型对视频做端到端的语义质量判断或物理合理性判断。
【大模型的位置】自研 VLM 只出现在打标环节（生成 short/dense caption），不承担判分与剔除职责；对该 caption VLM 本身也未做幻觉抑制的专门后训练（对比 HunyuanVideo 1.5 用 OPA-DPO 治理标注器幻觉）。
【唯一的模型级错配剔除】第6阶段的 CLIP Score 图文对齐过滤，属于「用判别式对比模型做粗粒度语义匹配」，是浅层语义判断而非 VLM 推理式判断。
【人工替代了大模型的角色】SFT 阶段的语义级质量判断（清晰度、美感、运动合理性、场景转换平滑度、caption 准确性）由人工评审完成，而非交给 VLM——这正是 2026 年趋势要替换掉的部分。因此 Step-Video-T2V 可作为「大模型质检员范式之前」的对照样本。[不确定]

### [UniTalking](../models/UniTalking.md) ⚠️

UniTalking 呈现出与 2026 年主流趋势相反的取向——大模型在其流水线中只负责标注，完全不参与质量裁决，质检全部交给传统专用判别模型：
【大模型的角色：仅标注】Qwen3-VL（视频 caption）、Whisper-V3（语音转写）、Qwen3-Omni-Captioner（音频 caption）、Qwen3-Omni（音视频融合描述）四个模型全部处于标注环节，位于三级过滤之后。论文未描述任何由 VLM/LLM 执行的质量打分、跨模态语义一致性校验、caption-内容错配检测或幻觉自审计机制。这与 MOVA 用 GPT-OSS-120B 做跨模态一致性裁决并内置 hallucination_check 的做法相比落后一代。
【质检全部由专用判别模型承担】
- PANNs：AudioSet 527 类音频事件分类的 CNN 模型，用于语音事件检测（判定音轨中是否存在语音）；
- SentenceASD：与 PANNs 配合执行语音事件检测；
- LightASD：轻量级主动说话人检测（Active Speaker Detection）模型，用于判定发声者是否在画面中——这是本流水线中最有针对性的一个工具选型，ASD 恰好是「声画同源」判定的标准解法；
- LipSync：唇音同步评估模型，用于剔除唇音对齐差的样本。论文未指明具体是 SyncNet、Syncformer 还是其他实现变体。[不确定]
【视频质量判定的模型完全未披露】「静态/文字叠加/低视觉质量」三条规则背后用的是什么模型，论文一个字都没提。[不确定]
【标注环节的质量保障机制缺失】三路 caption 由三个不同模型独立产出后拼接，以及第四路由 Qwen3-Omni 融合产出，但没有任何跨路一致性检查、没有幻觉过滤、没有人工抽检、没有 caption 质量评分。考虑到 Qwen3-VL 描述视频时若能听到音频容易产生跨模态幻觉，而论文既未说明是否做了模态隔离也未说明是否有防护，这是标注侧的一个未受控风险点。[不确定]
【值得肯定的一点】工具选型本身是贴合任务的：ASD + LipSync 的组合精确对应「说话人视频」这一 domain 的两个核心失效模式（声源不在画面 / 声源在画面但不同步），比通用 AV 工作只用单一 SyncNet 阈值要更有针对性。

### [UniVerse-1](../models/UniVerse-1.md) ⚠️

UniVerse-1 的模型介入方式与 2026 年主流趋势有明显差异——大模型只做标注、不做质检，质量判定仍完全交给传统判别式小模型：
【大模型的角色：仅标注，不裁决】QWen2.5-Omni 作为唯一的多模态大模型，职责是在训练过程中在线生成三路对齐标注（语音内容、视频描述、环境音描述）。论文未描述任何由 LLM/VLM 执行的质量打分、跨模态一致性校验或冲突消解环节——这一点与 MOVA 用 GPT-OSS-120B 做跨模态一致性裁决、并在 prompt 中内置 hallucination_check 自审计块的做法形成明显落差。
【质检仍由专用判别模型承担】DOVER（视频质量）、Whisper（语音有无判定 + ASR）、RetinaFace（人脸检测）、SyncNet（唇音同步置信度）、以及音量/能量/过零率三项信号级统计。全部为专用小模型或手工特征，无一为通用大模型。
【真正的创新在「何时标注」而非「谁来标注」】论文的核心主张是：传统离线标注（为整段长视频预先生成一份 caption）会导致标注与实际训练窗口之间的时序错配——训练时随机抽取的 5 秒窗口内容，与覆盖整段视频的全局 caption 并不对应，这种「misalignment text-based annotation」会直接损害生成模型的性能。解决方案是把标注搬到训练循环内：一个与训练并发运行的独立服务进程，对每次实际采样到的定长窗口即时生成标注，保证「送进模型的每一帧画面、每一段声音、每一句文本」三者严格同源同窗。这是把数据-标注对齐问题从「标注质量问题」重新定义为「标注时机问题」，是本工作在数据侧最具方法论价值的贡献。
【代价】在线标注意味着每个训练步都要付出一次 QWen2.5-Omni + Whisper 的推理成本，需要独立的标注服务集群与训练集群并行；论文未披露该服务的算力开销、吞吐能力或是否成为训练瓶颈。
【幻觉防护】论文未描述任何针对多模态标注器跨模态幻觉的约束设计（如禁止视觉标注器参考音频等），标注 prompt 原文亦未公开。[不确定]

### [Unison](../models/Unison.md) ⚠️

Unison 在训练数据侧完全没有使用 VLM/LLM 作为质检员——这与 2026 年「大模型语义判断替代浅层打分器」的主流趋势明显背离。训练数据的全部判定由传统判别式小模型完成：人脸检测器（型号未披露）、SyncNet（唇同步）、Mel-RoFormer（音源分离，属预处理非判定）。无跨模态一致性校验、无语义匹配裁决、无幻觉检测、无 VLM 质量打分。
【大模型的介入全部发生在评测侧，且相当深入——这是本字段的关键发现】Unison 把 Gemini 用在了两个评测环节：
1) 测试集标注生成——「a curated test set of 1,000 held-out samples, with ground-truth annotations provided by Gemini to ensure rigorous T2AV and TI2AV assessment」，即 1,000 条测试样本的 ground-truth 标注（caption 与 transcription）由 Gemini 生成，用于支撑 T2AV 与 TI2AV 两类任务的评测；
2) LLM-as-judge 主观评分——用户研究中除 25 名人类参与者的排序投票外，额外增设一列「Gemini Score」，由 Gemini 对生成结果的「运动-语音-音效连贯性」（Motion-Speech-SFX coherence）打分。结果显示 Gemini 评分与人类排序高度一致（Unison 1.68 最优，LTX-2 2.05，MOVA 2.48，UniAVGen 3.48），构成了对人类评价的交叉验证。这是把多模态大模型用作「视听连贯性裁判」的一个具体实例，在音视频生成评测中具有参考价值。
【显著的不对称】团队既然信任 Gemini 到可以用它生成 ground-truth 标注并作为评判者，却未把同等能力用于训练数据的质检与标注——这一不对称论文未作解释，可能与成本有关（200 万条 clips 逐条调用 Gemini 的开销远高于 1,000 条测试样本），也可能因为训练标注直接沿用了上游数据集自带的 caption（CelebV-Text、OpenHumanVid 均自带文本标注）。[不确定]
【与同类工作对照】MOVA 用 GPT-OSS-120B 做跨模态一致性裁决并内置幻觉自审计，UniVerse-1 用 QWen2.5-Omni 做在线三路标注——Unison 在训练数据的模型化质检上落后于两者，但在评测侧的 LLM-as-judge 应用上走得更远。

### [Veo 3 / Veo 3.1](../models/Veo_3_Veo_31.md) ⚠️

Veo 3 是「大模型深度介入数据流程」这一趋势的典型代表，但介入形态以「标注」为主而非「打分」。可确认项：(1) 使用多个 Gemini 模型（multiple Gemini models）为音视频数据生成不同粒度的 caption，这是 VLM 作为数据生产者的直接证据；(2) 使用多模态分类器（multimodal classifier）检测内容策略违规，官方特别强调多模态的必要性——「caption 与 video 单独看均无害，组合后可能产生有害结果」（举例：文本 prompt「一头猪的图像」与某一人群视频配对会构成有害表征），这本质上是一个跨模态语义错配/有害组合的判别器，思路上与「VLM 作为数据质检员」高度一致，但官方将其定位在开发期安全监测（development evaluations），而非明确的训练数据清洗环节。[不确定] 是否存在用 Gemini/VLM 对视频做美学或语义质量打分、是否用大模型剔除 caption-video 语义错配样本、打分模型规模与阈值，均未披露。

### [Vidu S1](../models/Vidu_S1.md) ⚠️

这是该报告数据侧最突出的方法论主张，体现了2026年「从浅层打分器转向大模型语义判断」的趋势：论文明确指出仅用专家模型（expert models only）评估视频数据存在明显局限——例如人脸检测模型难以泛化到夸张或高度风格化的2D动画主体；部分专家模型是基于图像的、只能对抽帧做判断，因全局信息不足而产生误判。因此引入 omni model（全模态大模型，引用文献[24,25]）对完整视频进行全局语义理解作为补充，由其沿 editing、subject、action、emotion、face、speech、scene、shot、tone 九个质量维度生成语义标签。专家模型与 omni 模型共同构成兼具全局上下文感知与局部细节敏感度的联合过滤系统（joint filtering system）。omni 模型的具体身份与规模未披露 [不确定]。此外 caption 标注同样由标注模型（annotation model）完成。

### [Wan 2.5 / 2.6 / 2.7](../models/Wan.md) ⚠️

Wan 系呈现清晰的「专家小模型做质检、MLLM 做打标与评测」的分工，尚未见到公开证据表明其数据漏斗已转向大模型语义判断。
【Wan 2.1 数据漏斗中的判别器全部是专用/浅层模型】轻量 OCR 检测器、LAION-5B 美学分类器、内部 NSFW 安全模型、水印/logo 检测器、启发式黑边检测、过曝专家分类器、合成图专家分类器、自研模糊打分模型、人工标注训练的1–5分视觉质量专家评估模型、运动质量分级器。
【MLLM 出现的三个位置，均非质检】
1) 打标：自研 LLaVA 式 dense caption 模型（ViT + 两层 MLP + Qwen LLM）对全量重打标；视觉文字分支用 Qwen2-VL 把 OCR 结果转成自然描述；V2A 用 Qwen2-Audio 生成音频 caption；Wan2.2-S2V 用 QwenVL2.5-72B 打标。
2) 数据构造中的校验：用 LLM 从文本中抽取 (类别, 数量) 对，再用 Grounding DINO 实际计数，只有两者一致才保留——这是 Wan 系最接近「模型做裁判」的一处，属交叉校验式弱质检。
3) 评测：Wan-Bench 大量使用 Qwen2-VL 做物理合理性、运动平滑度、风格化、物体/数量/空间关系、复杂运镜、动作指令遵循等复杂任务的判定，简单任务（如物体检测）才用传统检测器。这说明团队已具备用 MLLM 做语义判定的完整工程能力，只是公开材料中未把它用于训练数据过滤。
2.5/2.6/2.7 是否已引入 VLM 质检不可知；考虑到其「角色扮演」「多镜头一致性」等能力对语义级图文/音画匹配的要求，很可能已引入，但无任何依据。[不确定]

### [音视频生成评测基准合集](../models/av_benchmarks.md)

这是本次调研中五个基准最具训练侧参考价值的字段，五者恰好构成「大模型作质检员」这条 2026 年趋势的完整证据链：

【AV-SyncBench —— Gemini 作前置质检员的实证】明确使用 Gemini 3 Flash 作为自动过滤第一关，任务是剔除「画外声源」与「明显视听错配」两类样本，随后才交人工复核。这是目前公开文献中少见的、把商业闭源多模态大模型直接嵌入数据 pipeline 首级过滤的做法，其成本-效果权衡（用 Flash 级轻量模型做高召回粗筛，再用人力做高精度精筛）对大规模训练数据清洗有直接可移植性。

【AVBench —— 训练专用评测器替代通用大模型】走了另一条路：不用通用 MLLM 直接打分，而是通过偏好学习训练专用评测器。VT 与 AV 维度基于 Qwen2.5-Omni (7B) 仅微调 LLM 部分；AT 维度基于 Qwen2-Audio (7B) 微调 LLM 与 connector 层。训练约束设计巧妙：模型仅输出单 token（Yes/No），通过 token 概率比归一化得到连续分数，从而把离散判断转成连续可微信号，可直接用于数据过滤与 RLHF reward。这解决了 LLM-as-judge 打分离散、方差大、不可微的核心痛点，是训练侧最值得复用的技术细节。

【VABench —— 通用 MLLM 做语义层评测】Module 2 使用 Qwen2.5-Omni 7B 承担 5 项宏观打分（Alignment / Artistry / Expressiveness / Audio Realism / Visual Realism，1–5 分）与 2 项微观问答（每样本 3–7 条音频 QA 与 3–7 条视觉 QA）。「宏观打分 + 微观 QA 双层」结构值得借鉴：细粒度 QA 把模糊的整体评分拆解为可验证的事实判断，显著降低 MLLM 打分的主观漂移。

【Omni-Judge —— 能力边界的系统性测绘】最重要的负面结论提供者。以 Qwen3-Omni（30B 总参 / 3B 激活）为对象，对比 instruct 版与 reasoning 增强版，在 9 个维度上与 6 名博士生标注做相关性分析。结论清晰分层：语义类维度可用 —— audio-text alignment 在 Sora 2 子集上达 τ_b=0.292 / ρ=0.345，audio-video-text 三模态一致性 0.139/0.151；感知类维度不可用 —— video quality τ_b≈0.020，audio-video synchronization 仅 0.142，作者归因于 Omni-LLM 的时间分辨率不足。对训练数据 pipeline 的直接指导：Omni-LLM 适合承担「语义匹配/错配剔除」，但时序同步与画质判定必须交给 Synchformer、DOVER++ 这类专用模型。此外论文展示了利用 Omni-Judge 可解释反馈做生成模型「反馈式修正」的应用（基于识别出的错误生成修正帧），但未主张用于训练数据过滤。

【PhyAVBench —— LLM 在知识构建与初筛两端介入】LLM 用于物理知识头脑风暴、分类体系生成、提示词模板生成，以及质控阶段的语义歧义与混杂因素初筛；但所有 LLM 产出均经人类专家审核修订，形成严格的「LLM 提效 + 专家把关」分工。

综合判断：2026 年的共识是「大模型判语义、专用模型判感知、人类判物理与最终边界」三层分工，而非用大模型一把梭。

### [视频 Caption 模型生态](../models/caption_models.md)

这是 2025–2026 年该生态最显著的趋势，且 captioner 生态同时是「被判」与「判人」的双重角色：
【趋势一：LLM/VLM 作为 caption 质量裁判（浅层打分器 → 大模型语义判断）】
· AVoCaDO：GPT-4.1 对 Gemini 合成 caption 打 synthesis completeness 1–5 分，只保留 ≥4；GRPO 阶段进一步用 GPT-4.1 判定五维 checklist 覆盖度作为 reward ℛ_C。
· AVSCap：automated verification 做 tag 保留检查 + 语义一致性检查；GRPO 的 hybrid reward 含 audio-visual consistency 项。
· MOVA：GPT-OSS-120B（120B 开源模型）承担 caption 融合 + 跨模态一致性校验 —— 把最大的模型放在融合与裁决环节而非感知环节，是成本与效果权衡的典型选择。
· AuroraCap 提出 VDCScore：用分治策略把长 caption 评测转化为多个短问答对，由 LLM 判定 —— 代表 caption 评测从 CIDEr/BLEU 等 n-gram 指标向 LLM 语义判断的迁移。
· Omni-Cloze 反其道而行：用 2000 clip / 7 万道细粒度完形填空题规避 LLM-judge 噪声，是对 LLM-as-judge 可靠性的一种质疑与修正。
【趋势二：captioner 作为数据质检员】
· InstructAV2AV：Qwen3-Omni 既生成编辑指令又承担五维度验证打分 —— 论文自身承认「生成与验收同源是本 pipeline 的方法论隐患」，这是本生态少见的自我批评。
· Vidu S1 在质量过滤阶段使用 omni model 做判别（细节未披露）。
· Foley-Omni 用 Gemini 2.5 Pro 做标注，再用 Bandit（cinematic audio source separation）做声学后验证纠偏视觉幻觉，阈值 −35 dB —— 「大模型标注 + 信号级验证」的组合是抑制多模态幻觉的有效范式。
【方法论隐患（全生态共性）】(1) benchmark 与 model 同源：AVSCap 既是 AVSCapBench 的作者又是榜首（60.44），存在过拟合风险，其分数需用第三方基准（UGC-VideoCap、Omni-Cloze）交叉验证；(2) 教师即裁判：多数工作用 Gemini/GPT 家族既做教师又做裁判，评分偏向自身输出风格；(3) 人类一致性天花板：Panda-70M 报告的人类之间 caption 偏好一致率仅 44.9%，意味着 LLM-judge 的「准确率」上限本身就模糊。

### [几何/结构化标注数据集合集](../models/geometric_datasets.md)

这批数据集充分体现了“大模型作质检员”的2026年趋势。SceneScribe-1M 最典型：直接用 Qwen2.5-VL-72B 作为唯一的内容质量判官，一次性完成六个维度（运动强度、水印、相机畸变、光照伪影等）的语义判断，取代了传统的浅层美学/清晰度打分器组合。SpatialVID 采取混合策略：浅层打分器（CLIP+MLP美学、亮度统计、PaddleOCR、VMAF）做粗筛，把大模型算力留给标注侧——Gemini-2.0-Flash 做1fps视觉解析、Qwen3-30B-A3B 结合相机位姿先验精修并纠正运动方向描述，本质是“用几何真值校验VLM输出”的反向质检，可修正VLM常见的左右/推拉方向幻觉。Action100M 把LLM质检推向极致：GPT-OSS-120B 对多来源caption做证据聚合与结构化抽取，并执行三轮 Self-Refine 自我修正，等于用推理模型充当标注一致性审核员。WildWorld 用VLM做评测端的judge：WildBench 的 Action Following 指标由VLM判定生成片段与真值片段是否一致（0/1打分），与人工评价达成85%一致率——是模型即评委在几何数据集上的显式效度验证

### [视频生成后训练数据策略](../models/post_training_data.md)

这是本专题的核心——后训练阶段「模型即判官」不再只是数据过滤器，而是直接成为训练信号（奖励模型），这是 2025–2026 年最重要的范式转移。
【锚论文的奖励模型体系（披露相对充分的部分）】遵循 HPSv3 训练范式，用 Qwen3.5（引文指向 Qwen3-VL 技术报告 arXiv:2511.21631）作为主干从图像与文本抽特征，经 MLP 输出标量分数；对训练图像对 (x1,x2) 与文本 c 及人类偏好标注 (y1,y2) 计算 r1、r2，采用「不确定性感知的排序损失（uncertainty-aware ranking loss）」；训练分两阶段——Stage 1 用「数据感知的正交梯度投影（data-aware orthogonal gradient projection）」把来自 HPDv3++ 的多样美学偏好整合进来，同时保留 HPSv3 中已编码的原始人类偏好知识；Stage 2 进一步利用「由不同能力等级模型与不同 RL 迭代产生的无标注数据」。最终得到覆盖视频美学、文本-视频对齐、图像美学、文本-图像对齐的四个奖励指标。
【多奖励融合的工程难点（论文明确点出）】不同奖励信号的粒度、尺度与优化倾向各异：强调文本-视频对齐会提升语义保真但有时损害视觉自然度；过度优先运动质量或视频美学则导致画面好看但语义变弱。因此需精心设计奖励聚合策略并调节权重系数，使优化过程稳定且不被任一目标主导。论文自陈其最终以「整体视觉质量」为首要目标做取舍。
【横向奖励模型清单】
· VideoAlign / VideoReward（arXiv:2501.13918，快手可灵 + CUHK）：VLM-based，三维（视觉质量 VQ / 运动质量 MQ / 文本对齐 TA），训练自 1.6 万 prompt、12 个 T2V 模型生成的 10.8 万视频、18.2 万标注三元组，采用带平局的 Bradley-Terry 模型（BTT）。被 Cosmos-Predict 2.5、LongCat-Video、JavisDiT++ 广泛复用为基座，是开源视频奖励模型的事实标准；
· HPSv3（ICCV 2025）：HPDv3 含 108 万文本-图像对与 117 万成对比较标注，覆盖 SOTA 生成模型与低到高质量真实图像；
· LongCat-Video 三奖励：VQ = HPSv3-general（全帧平均）+ HPSv3-percentile（取分数最高前 30% 帧、结合视频 caption 计算，用于抑制少数低质帧对整体判断的稀释）双路；MQ 与 TA 均以 VideoAlign 为基座在内部标注数据上微调；
· Seedance 1.0 三个专用 RM：Foundational RM（VLM 架构，图文对齐与结构稳定性）、Motion RM（抑制伪影、增强运动幅度与生动性）、Aesthetic RM（图像空间输入，数据源改为视频关键帧），并做多轮迭代式学习；
· SkyReels-V2：Bradley-Terry 模型在 3 万样本对上训练；
· JavisDiT++ 六模型分工：AudioBox-Aesthetics（音频质量）、ImageBind（文本-音频对齐 / 文本-视频对齐 / 跨模态相似度）、VideoAlign（视频质量）、Synchformer（时序同步）；
· 其他被引用的 RM 谱系：ImageReward、Pick-a-Pic、VideoScore、VisionReward（AAAI 2026）、Unified Reward Model、RewardDance。
【反例与警示】Step-Video-T2V 明确不训练奖励模型，并在展望中指出当前 DPO 的局限——当模型能轻易区分正负样本时收益即饱和——提出未来引入 RM 对新生成样本动态打分以持续提供有效梯度。ITS-JAVG 揭示的 verifier hacking 问题预示：若贸然用自动验证器构造偏好数据做 RLHF，很可能训出只会讨好判官的模型。Cosmos-Predict 2.5 的应对是用微调数据集上的 diffusion loss 做正则以缓解 reward hacking；LongCat-Video 与锚论文的应对是多奖励并用互相牵制。

### [主流视频预训练数据集合并调研：Panda-70M、InternVid、Koala…](../models/pretraining_datasets.md)

**这是七者最能体现「2024→2025 技术代际迁移」的一条轴线，可清晰排出三代**：
【第0代·完全无模型判官（Panda-70M, 2024.02）】
过滤全部由 PySceneDetect 与 ImageBind 特征距离完成，没有任何语义级质量判断。唯一接近的是 **UMT 检索模型输出的 matching_score**（>0.43 视为图文关联强，全量89.6%达标），但它的职能是**从8条候选caption中选最优**，而非给样本打质量分做剔除——**VLM 在此仅是打标器（8个教师）而非判官**。
【第1代·浅层专用打分器 + 阈值（InternVid 2023, OpenVid-1M 2024, MiraData 2024）】
LAION美学预测器、DOVER、RAFT/UniMatch光流、CLIP相邻帧相似度——全部是输出标量的轻量专用模型，靠阈值/百分位裁切。MiraData 唯一用到大模型的地方是**缝合环节的 Qwen-VL-Chat + LLaVA 投票**（判断相邻片段是否同一场景），这是把 VLM 用于**结构判断**而非质量判断，属于过渡形态。
【第2代·MLLM 直接做二元语义裁决（LVD-2M 2024.10，最早）】
**LVD-2M 是本次调研中最早、最彻底地用 MLLM 替代打分器的案例**：**PLLaVA-7B 取8帧均匀采样，直接输出大写的 GOOD/BAD**，「只有在所有定义的指标上都被判为good的视频才保留」。两条prompt原文值得引用——(1) 内容变化度：「If the background, setting, and characters are in static states, the video lacks content variation… You must provide a capitalized either 'BAD' or 'GOOD' answer.」；(2) 视觉多样性+文字：「A visually diverse video should have rich content that is visually appealing. If the video is only some person talking to the camera with a static background, it is not diverse. And a video with only texts instead of objects is not diverse… Determine if text overlays dominate the video in a way that detracts from the visual experience.」**它以此完全取代了美学打分器与OCR模型**。动因写得很明确：光流抓不到「手持抖动虽然光流高但无有意义运动」这类问题。作者同时诚实地列为局限：「当前MLLM不保证具备视频质量评估能力，部分模型难以遵循相关指令」，并呼吁建立MLLM视频质量评估的基准。
【第2.5代·大模型属性清单式裁决（UltraVideo 2025.06，规模最大）】
**Qwen2.5-VL-72B 对16种低质属性做二元判断，命中任意一项即删除**（转场特效、水印、分屏、屏幕录制、画中画等）。同时用 **VideoCLIP-XL-v2** 计算视频与摘要caption的相似度，**<0.2 剔除**——这是七者中唯一显式的图文一致性剔除环节（Panda-70M 的 matching_score 只用于选caption不用于剔除）。72B 规模的判官 + 属性清单式的结构化裁决，是2025年的典型形态。
【第3代·把子指标喂进学习式网络输出统一分（Koala-36M，另一条路线）】
Koala-36M 走的不是「更大的VLM」而是「更好的融合」。其 **Training Suitability Assessment (TSA) 网络**有三个分支：动态分支（**3D Swin Transformer**）、静态分支（**ConvNeXt**）、标签分支（把清晰度/美学/运动等**传统子指标作为额外输入**而非阈值），再用 **Weight Cross-Gating Block (WCGB)** 把标签分支特征以可学习的融合权重注入动态与静态分支。**VTSS 是这个网络吃「像素+子指标」后直接输出的标量，不是对子分数做回归或加权求和**——否定加权阈值范式正是该论文的全部论点。训练标签来自 **20万条视频 × 8名专家 × 1–5分**，并做了两重偏差校正：个人偏好偏差用**逐专家 z-score 标准化**后按全局均值方差还原，标注波动偏差取**8人均值**。消融（PLCC/SRCC/KRCC/RMSE）：仅动态分支 0.8684/0.8580/0.7027/0.4644 → +静态 0.8730 → +标签分支 0.8953 → +WCGB **0.8974/0.8868/0.7406/0.4099**；对比 FastVQA 0.8684、DOVER 0.8554。**VTSS 阈值2.5 的选法也很朴素**：全量分布呈双峰（近似两个高斯），直接取两峰之间的谷值2.5（发布CSV中VTSS最小值恰为2.50，独立印证）。
**⚠️ 复现警告**：Koala-36M **发布的 VTSS 检查点并非造数据用的那个**——test.yml 显示其配置为 `DiViDeAddEvaluator + swin_tiny_grpb + fragments-only + divide_head:false`，即 FAST-VQA/DOVER 的纯 fragments 分支，性能恰好等于消融表中「仅动态分支」那一行（PLCC 0.8684），**ConvNeXt静态分支、标签分支与WCGB 在仓库中完全缺失**。任何人按其开源代码复现过滤，得不到论文的 VTSS。
**结论**：2024年上半年的数据集（Panda-70M、InternVid、OpenVid-1M、MiraData）尚停留在「浅层打分器+阈值」，2024年10月的 LVD-2M 与 Koala-36M 分别从「换判官」和「换融合方式」两个方向突破，2025年的 UltraVideo 则把判官规模推到 72B 并结构化为属性清单——**「VLM-as-judge」在这七个数据集上的演进时间线非常清晰**。

## 安全与合规过滤（NSFW、版权、人脸/隐私）

`safety_filtering` · 详细程度: brief

### [Allegro](../models/Allegro.md) ⚠️

论文未披露任何 NSFW / 暴力 / 版权 / 人脸隐私相关的安全合规过滤环节，Table 1 的阈值表中也无相应指标。团队仅在模型许可（Apache 2.0）与使用条款层面提示合规使用，数据侧的安全过滤策略完全缺失于公开描述。考虑到语料来自 WebVid / HD-VILA 等公开数据集，可能部分继承了原数据集的过滤，但论文未做说明。[不确定]

### [Apollo](../models/Apollo.md) ⚠️

论文对安全与合规过滤的披露仅有一个词：「safety」被列为视频质量建模的第四个维度（与动态质量、静态质量、内容自然度并列），置于第一道过滤闸门。没有任何展开——未说明 NSFW 检测方法或模型，未提及版权过滤，未提及人脸识别/隐私保护/名人肖像过滤，未给出安全类目体系，也无模型卡级的安全声明或使用限制。作为快手这类需承担内容监管责任的平台方，其内部必然存在成熟的安全审核体系，但论文选择不披露。[不确定]

### [CineDance / CineDance-1M](../models/CineDance.md) ⚠️

安全与合规过滤是该工作披露最薄弱的维度：
【论文层面】三阶段 pipeline 中不含 NSFW 检测、暴力内容过滤、人脸隐私处理或版权检测环节的任何描述。人工审计的伪影清单也全部是画质类问题（字幕、水印、logo、黑边、录屏等），不含安全类目。
【发布层面的替代性约束】① 采用 gated access 门控发布，需人工审核申请者；② 许可限定为 CC-BY-NC-SA-4.0 非商业用途；③ 数据集卡明确免责——「自动与人工策展无法保证移除每一条低质、敏感或其他不良样本」，要求使用者按自身应用场景自行做质检。
【判断】安全责任事实上通过门控 + 免责声明转移给了下游使用者，而非在 pipeline 内解决。考虑到素材为影视内容（可能含暴力、成人情节），这是实际使用时需重点补足的环节。[不确定]

### [CogVideoX](../models/CogVideoX.md) ⚠️

[不确定] 论文与开源仓库均未描述训练数据层面的 NSFW、暴力、版权、人脸/隐私过滤流程与工具，也未披露生成端的安全模型或水印方案。相关的仅有间接线索：负面标签中的 Text Dominated 与 Noisy Screenshots 会顺带剔除部分含敏感文本或屏幕内容的素材；开源协议（Apache 2.0 / 智谱开源协议）中包含常规的合规使用条款；清影产品侧作为国内公开服务需符合《生成式人工智能服务管理暂行办法》并完成算法备案，但技术细节未公开。

### [Cosmos-Predict2.5](../models/Cosmos-Predict25.md) ⚠️

[不确定]。论文层面几乎没有安全合规过滤的描述。唯一沾边的表述是过滤目标中包含「content that is unsuitable for training」（不适合训练的内容），未展开说明其定义、检测手段与规模，推测涵盖 NSFW / 暴力等，但无文字依据可确认。全文未提 NSFW 分类器、暴力检测、人脸隐私/去标识化、版权过滤等任何具体机制。
间接相关的是 NVIDIA Cosmos 平台层面的 Cosmos-Guardrail 护栏体系（前代 Cosmos-Predict1 中描述过、GitHub 仓库文档提及「robust guardrails」与「improved guardrails」），但那是推理侧的输入 prompt 与输出内容拦截，并非训练数据清洗环节，且本篇未展开。
值得注意的是本 pipeline 有一条与安全无关但方向相似的内容裁剪：剔除游戏/动画/卡通等非物理真实内容，出发点是分布对齐而非安全。

### [Data-Juicer 2.0](../models/Data-Juicer.md) ⚠️

【视觉安全】video_nsfw_filter 是唯一的视频侧安全算子，「保留 NSFW 分数在指定区间内的样本」，底层为 Falconsai/nsfw_image_detection 图像分类模型（对抽帧逐帧打分后聚合）。该算子在 T2V 案例中被 Probe 阶段验证为有效并纳入最终配方，官方表述为「使用 VideoNSFWFilter 算子以保证高质量」——即在该案例中它同时承担了安全与质量双重职责（NSFW 高分样本往往也伴随低质量、非自然内容）。
【隐私保护】video_face_blur_mapper 检测并模糊视频中的人脸，是数据处理框架层面较少见的内置隐私脱敏算子。配合 video_human_tracks_extraction_mapper（人脸/人体轨迹提取）可实现轨迹级的持续脱敏。文本侧另有敏感信息（手机号、邮箱、身份证等）脱敏类算子。
【版权相关】video_watermark_filter 与 video_remove_watermark_mapper 可识别与处理水印，间接关联版权标识，但不构成版权归属判定能力。
[不确定] 缺失的环节包括：无版权内容识别/指纹比对能力；无 C2PA 等来源认证标准支持；无暴力、仇恨、未成年人等细分安全类目的检测算子（仅有二元 NSFW）；未披露阿里内部使用时叠加的安全审核体系。作为开源框架，DJ 提供的是「可组装的安全零件」，完整的合规体系需使用方自行构建。

### [Foley-Omni](../models/Foley-Omni.md) ⚠️

[不确定] pipeline 中未描述任何安全与合规过滤环节：无 NSFW 检测、无版权内容识别、无人脸隐私处理或去标识化措施。
论文仅在伦理声明层面承认风险：明确提到模型存在被滥用于制作 deepfake 的可能（考虑到 VisualTTS 占训练数据40%、模型能根据说话人视频生成匹配语音，这一风险确实实质存在），并声明数据在「appropriate usage agreements」下收集。但这些属于事后声明而非数据处理环节的技术措施。
未见任何生成内容水印、C2PA 标记、或对说话人身份的授权确认机制。考虑到 GRID、LRS2、SpeakerVid、TalkVid 等数据集均包含大量可识别人脸与声纹，且模型直接学习「人脸→声音」映射，隐私维度的处理缺位是该工作较明显的合规短板。

### [Goku](../models/Goku.md) ⚠️

[不确定]。论文全文未提及 NSFW/涉黄涉暴过滤、版权过滤、人脸与隐私保护、敏感人物识别、有害内容分类器等任何安全合规环节，也没有 Responsible AI / 模型卡章节。考虑到发布方为字节跳动，实际生产流程中几乎必然存在内部安全审核（字节有成熟的内容安全中台），但论文层面零披露。唯一间接相关的是 OCR 文字过滤（剔除字幕/贴片广告），但其动机是画质与生成质量而非安全合规。

### [Hailuo / MiniMax Video](../models/Hailuo.md) ⚠️

[不确定]（数据侧无披露，仅有推理侧证据）。训练数据端的 NSFW 过滤、版权过滤、人脸/隐私处理均未披露。可观测的只有推理侧的内容安全策略：海螺AI 与开放平台 API 对涉政、色情、暴力及公众人物肖像的提示词与生成结果有明确的实时审核与拦截（中国监管环境下的《生成式人工智能服务管理暂行办法》合规要求），海螺AI 网页端展示「内容由 AI 生成，请合法、友好地使用该功能」提示。推理侧存在严格审核可间接说明公司具备成熟的内容安全模型，但不能直接推断其在训练数据清洗阶段的应用方式与强度。

### [HunyuanVideo-Foley](../models/HunyuanVideo-Foley.md) ⚠️

论文与模型卡均未描述任何面向训练数据的安全与合规过滤：没有 NSFW 内容检测、没有暴力/血腥内容过滤、没有版权内容识别、没有人脸/人声隐私保护措施、没有敏感内容的类目排除说明。
【模型侧的替代性风险控制】安全责任被转移到许可证与分发层面而非数据层面：采用 tencent-hunyuan-community 社区许可（含使用限制条款），HuggingFace 上设置 extra_gated_eu_disallowed: true 排除欧盟地区，模型获取需通过 gated 流程。这是一种「用法律与分发控制替代技术过滤」的姿态。
【风险面评估】相比视频生成或语音克隆模型，Foley 音效生成的滥用风险总体较低——生成的是通用声学事件而非可辨识的人声或人像，深度伪造与身份冒用风险小。主要残余风险是：(1) 若训练数据未剔除暴力/血腥场景音，模型可能被用于强化暴力内容的沉浸感；(2) 与视频生成模型级联使用时，逼真音效会显著提升伪造视频的可信度，构成对整体伪造链条的能力增益；(3) 无输出侧音频水印，生成音频不可溯源。论文对上述任一点均无讨论。
【披露缺失的程度】即便考虑到任务风险较低，一个由大型商业公司发布的开源模型完全不提数据安全过滤，仍属披露不足。[不确定]

### [HunyuanVideo](../models/HunyuanVideo.md) ⚠️

披露很弱。原版仅一句提及用 YOLOX-like 视觉模型剔除「水印、边框、logo 及某些敏感信息（sensitive information）」，未展开何为敏感信息，未提 NSFW 分类器、人脸/隐私保护、版权过滤。1.5 技术报告的数据章节完全未涉及安全过滤。作为中国境内发布的产品，实际生产系统必然存在内容安全审核（模型卡与许可证中有合规使用条款、开源仓库要求遵守当地法律法规），但训练数据侧的安全过滤方法零披露。[不确定]

### [InstructAV2AV](../models/InstructAV2AV.md) ⚠️

安全过滤在本pipeline中作为自动验证五维度中的一维显式存在——「(v) safety to filter inappropriate content」，即由 Qwen3-Omni 判断合成结果是否含不当内容，不通过则丢弃。相比调研中多数工作把安全过滤完全省略，本工作至少把它写入了流程并作为硬性通过条件之一，属于中等偏上的做法。
【局限与空白】
  · 粒度粗：仅一个笼统的 safety 维度，未细分 NSFW、暴力、仇恨内容、版权标识、人脸隐私等子类，也未说明使用了何种安全判据或 policy。
  · 未见专用安全模型：未提及 NSFW 分类器、版权内容识别、人脸检测/去标识化等专门工具，全部依赖通用 MLLM 的判断。
  · 施加位置偏后：安全检查只作用于合成产物，未见对输入素材（YouTube 爬取内容）的前置安全筛查。
【最值得关注的空白——身份滥用风险未处理】数据集包含 clone_id（身份克隆）、clone_voice（音色克隆）、clone_id_voice（身份+音色同时克隆）三个专门子集，模型的直接能力就是「在真实人物视频上换脸、换声、改台词并保持唇同步」——这正是 deepfake 的技术定义。然而：
  · 论文未提及说话人肖像权/声音权的授权或知情同意；
  · 未对生成内容施加任何水印、C2PA 内容凭证或可检测标记；
  · 论文的局限性讨论完全聚焦于技术缺陷（物理真实感、光照一致性、3D 空间一致性、物体恒常性），未包含伦理声明或滥用风险讨论——对照 Foley-Omni 至少明确承认了 deepfake 风险，本工作在这一点上更为缺失；
  · 数据集以 MIT 许可完整开放下载（含真实人物影像），权重亦完整开放，且专门提供了身份/音色克隆的微调 checkpoint，实际降低了滥用门槛。
综合看，安全维度是本工作与其技术开放度最不匹配的一环。

### [2026 其他音视频联合生成](../models/JAVG_2026_misc.md) ⚠️

[不确定]——七项工作全部未描述 NSFW 过滤、暴力/血腥内容识别、版权内容检测、人脸隐私保护、名人肖像剔除等安全合规环节。
仅有的间接相关内容：
(1) ALIVE 的 13 维低质量分类器中是否包含安全维度未知[不确定]；其 OCR 水印检测可视为对版权标记的间接识别，但目的是画质而非合规。
(2) NAVA 的 PaddleOCR 字幕擦除同样是画质导向。
(3) OmniCustom 与 ALIVE 大量使用 ArcFace 人脸嵌入做身份匹配/校验，涉及生物特征处理但无隐私声明；OmniCustom 评测集使用「30 persons who were not included in training data」+ 70 段 YouTube 视频，未说明肖像授权。
(4) 音色克隆的滥用风险：OmniCustom（参考音频→音色模仿）与 NAVA（Timbre-in-Context Conditioning，参考音色控制）都具备声纹克隆能力，这是明确的深度伪造风险点，但两篇论文均未包含 responsible-use 声明、水印方案或滥用防护讨论[不确定]。NAVA 虽以 Apache 2.0 开源权重与训练代码，仓库亦未见使用限制条款[不确定]。
(5) StreamChar 的实时流式数字人同样具备实时伪造潜力，无相关讨论。
【判断】安全合规是本批七项工作最统一的空白维度——七项中零项有实质披露，且其中三项（OmniCustom、NAVA、StreamChar）恰恰涉及身份与声纹克隆这类高风险能力。相比之下，同期的 Sora 2、Veo 3 等闭源工业模型均有明确的安全章节，开源/学术侧的这一落差值得注意。

### [音视频联合生成基线合集](../models/JavisDiT_baselines.md) ⚠️

整体披露极少，仅 JavisDiT 一家有明确动作：
【JavisDiT / JavisDiT++（唯一）】JavisBench 构建过程中，对全部来自 YouTube 的内容进行了「严格的人工法律与伦理审核（strict manual legal and ethical verification）」——这是本合集中唯一明确的安全合规环节，且是人工而非自动化的。但该审核针对的是评测集（10,140 条），训练侧的 33 万条 TAVGBench 数据是否做过安全过滤未说明[不确定]；也未描述 NSFW 检测器、暴力内容过滤、人脸隐私保护等自动化手段。
【MM-Diffusion】未描述 NSFW/版权/隐私过滤[不确定]。间接的合规意识体现在数据选型上：选用 AIST++ 的理由之一是其配乐为版权已清理歌曲。
【AV-DiT】未描述任何安全过滤[不确定]。
【Harmony】未描述任何安全过滤[不确定]。其数据含大量真人视频（OpenHumanVid、SpeakerVid、自采集片段），人脸与声纹隐私风险实际存在但论文无相关表述。
【UniAVGen】未描述任何安全过滤[不确定]。风险最高——训练数据为「内部采集的真人（real human）音视频数据集」，且模型具备音频驱动人脸动画（audio-driven animation）能力，属于深度伪造敏感能力，但论文既无数据侧隐私说明，也无模型侧滥用防护或使用限制声明。
【共性缺口】五者均未提供 Model Card 级别的 safety 章节、未讨论深度伪造滥用防范、未提及名人肖像剔除或儿童内容处理。

### [可灵 3.0 / 可灵视频 3.0 Omni](../models/Kling_30_Omni.md) ⚠️

Kling-Omni 基础过滤级明确包含“内容安全 / NSFW 过滤”。产品侧另有严格的输入输出安全审核（人脸/名人肖像、涉政涉暴内容拦截）与AI生成内容标识/水印，符合中国监管要求。[不确定：训练数据侧的人脸隐私、肖像权、版权过滤的具体策略与模型]

### [LTX-2](../models/LTX-2.md) ⚠️

训练数据侧的安全与合规过滤完全未披露：NSFW 过滤、人脸/隐私处理、CSAM 筛除等均无任何说明。团队的合规策略主要前置在数据获取端——通过 Shutterstock research license 与 Getty Images 授权采购来保证版权清洁，而非通过下游内容分类器。技术报告「Social Impact」一节仅在部署层面做定性表态：承认合成媒体存在被用于欺骗性内容的风险，要求使用者明确披露合成来源并遵守内容安全准则，承认模型会继承训练数据中的视觉与听觉偏见，并将偏见缓解、真实性验证与可溯源性列为未来工作。模型卡的 Limitations 明确提示模型「可能生成不当内容、放大社会偏见」。未见任何自动化安全过滤模块、分类器或红队流程的描述。[不确定]

### [LongCat-Video](../models/LongCat-Video.md) ⚠️

[不确定]。技术报告未提及任何 NSFW 过滤、暴力/敏感内容过滤、版权过滤或人脸隐私保护措施。考虑到模型面向公开开源且由国内大厂发布，实际生产中几乎必然存在内容安全审核环节，但报告中无任何文字依据，无法确认其方法与强度。间接相关的仅有 Avatar 1.5 中把「synthetic content（合成内容）」列为情绪数据的硬性排除项，但那是出于标注可靠性而非安全考虑。

### [MOVA](../models/MOVA.md) ⚠️

论文完全未涉及安全与合规过滤：没有 NSFW 检测、没有版权过滤、没有人脸/隐私保护措施的任何描述，也没有模型卡级的安全声明或使用限制。这与其 Apache-2.0 完全开放商用的许可形成对比，是 MOVA 披露体系中的明显空白（与 Sora 2「安全披露详尽、数据披露空白」恰好相反：MOVA 是「数据方法披露详尽、安全披露空白」）。[不确定]

### [合并条目：① Mochi 1](../models/Mochi_MAGI_Motif.md) ⚠️

① Mochi 1：模型卡表明「NSFW content filtering has been applied（已施加 NSFW 内容过滤）」，但未说明方法、工具或阈值；同时明确提示「Genmo 视频模型会反映其训练数据中的偏见与成见」，建议机构在商用部署前实施额外安全协议。版权、人脸/隐私相关的数据侧过滤未提及。[部分不确定：NSFW 过滤的具体方法]
② MAGI-1：Section 3 DATA 全文未设安全与合规过滤环节，未提及 NSFW、暴力、版权或人脸隐私过滤。唯一与人脸相关的过滤是 Corner Face Detection，但其动机是「剔除解说类视频的固定角落主播画中画」这一视觉模式问题，而非隐私保护。[不确定]
③ Motif-Video 2B：安全过滤被显式前置到 sanitation 首关，且是本组中唯一给出双信号机制的：先由继承自旧爬取管线的 OCR 初筛以帧上文字检测标出高置信水印/台标/烧录字幕，存活片段再由 VLM 输出 nsfw、watermark、padded 等结构化标签，命中任一即硬删除；论文称第二遍 VLM 是「架在 OCR 之上的语义感知安全网（a semantically aware safety net on top of OCR）」。但仍未涉及版权审查、人脸/隐私脱敏、未成年人内容等专项，也未给 NSFW 分类器的具体型号与判定阈值。[部分不确定：NSFW 判定细则与版权/隐私处理]

### [Movie Gen](../models/Movie_Gen.md) ⚠️

[不确定] 技术报告未描述训练数据层面的 NSFW、版权、人脸/隐私过滤流程与工具。论文关于安全的表述集中在结论的「Safety considerations」段：模型为研究目的开发、部署前需多项改进；模型可能学到模态间的非预期关联；生成模型会学到各模态中的偏见（如视频训练数据中的视觉偏见、prompt语言中的偏见）；研究仅限英语文本输入；真正部署时会接入安全模型来拒绝违反政策的输入prompt或生成结果，以防滥用。数据侧的合规过滤、生成内容水印/来源标记均未披露。

### [NVIDIA NeMo Curator](../models/NeMo_Curator.md) ⚠️

NeMo Curator 的安全过滤能力在模态间严重不均衡：
【图像模态】提供 NSFW 检测 stage（官方 README 明确列为图像侧核心能力之一）。
【视频模态】官方文档的视频过滤章节只包含运动过滤与美学过滤两类，未提供 NSFW 分类器、人脸检测/打码、隐私信息移除、版权指纹比对等 stage。Cosmos WFM 论文的 curation 章节亦未描述数据侧的安全过滤——其安全能力集中在模型推理侧的 Cosmos Guardrails（输入 prompt 过滤 + 输出内容分类 + 人脸模糊化），属于部署护栏而非训练数据清洗。
【文本模态】有质量过滤与分类器体系，可间接承担部分内容安全职责。
【结论】用该框架构建视频训练数据时，NSFW、人脸隐私、版权三条合规线均需用户自行补充实现。[不确定]

### [OmniHuman 数据集 + OHBench](../models/OmniHuman.md) ⚠️

【完全缺失】论文未涉及任何安全与合规过滤：无 NSFW/色情/暴力内容检测、无未成年人内容识别、无版权过滤、无人脸隐私脱敏、无声纹隐私处理、无仇恨言论或有害语音内容过滤、无模型卡级的使用限制声明。全文亦无伦理声明章节。
【本数据集的特殊风险敞口】相较其他条目，此处缺失的后果更重：
1) 数据源为无差别的 YouTube 爬取，虽然 YouTube 平台自身有内容审核，但其审核标准（可公开播放）与训练数据标准（适合用于生成式模型训练）完全不同，例如大量成人向、擦边、暴力游戏实况内容在平台上合规却不适合作为训练数据；
2) 发布物包含 8 万真人的 ArcFace 人脸嵌入、134 点骨架、SMPL/MANO 三维参数、语音转写与情绪标签——这是一套可直接用于身份伪造的完整素材包，且无准入审核；
3) 内容类型明确包含「影视剧（film/TV）」，这部分内容的版权状况最为敏感，即便采用 URL 分发也难以完全规避；
4) 8 类内容中的「教育」「游戏」类别在 YouTube 上有大量儿童相关内容，而无未成年人识别机制。
【唯一的间接缓解】只发布标注与 URL、不发布视频本体，在版权维度提供缓冲（详见 provenance_licensing），但对内容安全与肖像隐私无缓解作用。
【与上游对照】OpenHumanVid 设有下载审批机制，OmniHuman 无任何准入控制。[不确定]

### [Open-Sora 系列](../models/Open-Sora.md) ⚠️

两个项目的技术报告与文档中**均未描述任何 NSFW 检测、暴力内容过滤、人脸/隐私保护或版权内容检测环节**，安全过滤实际上是完全缺位的，隐含地依赖上游公开数据集（Panda-70M、LAION、Webvid 等）自身已做过的安全清洗。间接相关的仅有：Open-Sora Plan 从 LAION-5B 中筛选人像图时做了「高质量」筛选但无隐私/肖像权考量；两者的 OCR 过滤剔除密集文字场景，可顺带滤掉部分带版权声明/台标的素材，但这是副作用而非设计目的。模型权重以 Apache 2.0 等宽松许可发布，也未附带输出侧安全分类器或水印。这是开源复现项目相对闭源商业模型（Sora 2 的 CSAM 源头筛除 + 多级安全分类器）最大的合规差距。[不确定]

### [Ovi](../models/Ovi.md) ⚠️

[不确定]。论文与仓库均未描述任何 NSFW 过滤、暴力/血腥内容过滤、版权内容识别、人脸隐私保护、名人肖像剔除、儿童内容处理等安全合规环节，也没有 Model Card 层面的 safety 章节或使用限制说明（仅有 Apache 2.0 许可）。考虑到 Character AI 作为面向消费者的对话产品公司通常有内部内容审核体系，其内部音视频语料可能已经过产品侧审核，但论文无任何相关表述。这是该工作在数据披露上的明显空白项。

### [Script-a-Video](../models/Script-a-Video.md) ⚠️

论文完全未涉及安全与合规过滤：无 NSFW 检测、无暴力/敏感内容过滤、无版权过滤、无人脸隐私保护措施、无模型卡级安全声明或使用限制。[不确定]
风险落差值得注意：本工作的 Identity Customization 模块支持从参考图像注入真实人物身份并在多镜头中保持一致，Event 流记录逐字台词并绑定说话人，Reference 流对人物做细粒度外观锚定（服装、配饰、发型）——这套能力组合恰好构成高质量深度伪造的技术前提，而论文对滥用风险零讨论。缓解因素是所有模型权重与数据均未公开发布。

### [Seedance 2.0 与 Seedance 1.5 pro](../models/Seedance_20_Seedance_15_pro.md) ⚠️

Seedance 1.0：部署先进分类器检测并剔除色情（pornography）、露骨暴力（explicit violence）、儿童剥削（child exploitation）、露骨裸露（explicit nudity）等有害或不当内容，以确保伦理合规与数据集安全。Seedance 2.0 报告声明在模型迭代全生命周期实施结构化安全评估框架并持续评估与缓解潜在风险。人脸/隐私、版权指纹等具体机制未披露。[不确定：人脸与隐私过滤、版权检测的具体方法]

### [SkyReels 系列](../models/SkyReels.md) ⚠️

两代论文均未设置安全与合规过滤章节：无 NSFW/暴力内容分类器说明，无人脸/隐私处理（去标识、肖像权）说明，无版权侵权检测，无 CSAM 筛除流程，亦无红队测试或社会影响（Social Impact）章节。间接相关的只有内容类型层面的剔除（监控录像被整类过滤，客观上减少了隐私敏感素材）与「授权内容」的采购口径。作为面向中国市场的商业产品，SkyReels 产品端应存在符合监管要求的内容安全审核与深度合成标识机制，但技术报告完全未涉及，无法确认其数据侧实现。[不确定]

### [Sora 2](../models/Sora_2.md)

这是训练数据侧唯一有实质披露的清洗环节。明确内容：(1) 使用「a combination of safety classifiers」防止有害或敏感内容被使用或生成，明确点名涉未成年人的性内容（CSAM）；(2) 儿童安全上采取「responsibly sourcing datasets to exclude CSAM」——即在数据源头筛除CSAM，并与NCMEC合作，对所有输入输出（含一方产品与API/企业版三方使用）执行强扫描，除非客户满足严格豁免标准；(3) 拥有专门的CSAM安全栈，复用其他产品的系统级缓解措施并叠加Sora专属保护。NSFW/版权/人脸隐私在训练数据侧未单独说明过滤方式，主要通过部署侧策略处理：不支持video-to-video、不支持公众人物的文生视频、除通过cameo显式opt-in授权的用户外阻断包含真人的生成、对疑似未成年人的上传素材施加更严阈值。

### [SpeakerVid-5M](../models/SpeakerVid-5M.md) ⚠️

论文与数据卡均未描述任何安全与合规过滤环节：无 NSFW/暴力内容检测、无版权内容识别、无人脸模糊化或隐私脱敏、无未成年人内容处理、无有害言论（基于 ASR 转写文本的内容审核）过滤。
【替代性的风险处置】唯一的合规措施是事后的、被动的：限定非商业科研教育用途、声明版权归原作者、提供 takedown 下架政策、不托管原始视频而只发布 YouTube ID。这是「转移责任」而非「主动过滤」。
【隐私敏感度评估】该数据集的隐私风险实际高于一般视频数据集——它包含 83K 个唯一说话人身份、逐 clip 的 ArcFace 人脸特征关联、DWpose 全身骨架、以及完整的 Whisper 语音转写文本（即「谁在什么时候说了什么」的可检索结构）。这一组合足以构成可用于身份追踪的生物特征数据库，但论文未做任何隐私影响评估，也未说明是否获得出镜人物同意（YouTube 公开视频的默认答案是没有）。
【内容政治敏感性】数据源明确包含「新闻与政治（news and politics）」与「辩论（debates）」类目，ASR 转写会完整保留其中的政治言论，但未见任何内容审核说明。
【结论】安全与合规过滤是本数据集披露体系中最大的空白。[不确定]

### [Step-Video-T2V](../models/Step-Video-T2V.md) ⚠️

披露很弱，只有一项：pipeline 第2阶段的 NSFW 打分，使用 LAION 提供的基于 CLIP ViT-L/14 的二分类 NSFW 检测器对抽样帧判分，据此过滤色情/不适宜内容。除此之外，报告未提及版权过滤、人脸/隐私保护（人脸打码、肖像权处理）、暴力血腥内容分类、政治敏感内容过滤、名人肖像剔除等任何环节，也未描述生成侧的安全对齐或输出审核。水印检测虽可间接剔除部分明显带版权标记的素材，但其设计动机是画面清洁而非合规。
作为中国境内商用产品（跃问 yuewen.cn 上线），实际生产系统必然存在完整的内容安全审核链路，但技术报告在数据侧的安全过滤上几乎零披露。[不确定]

### [UniTalking](../models/UniTalking.md) ⚠️

论文完全未涉及安全与合规过滤：无 NSFW/暴力内容检测、无版权过滤、无人脸隐私保护措施、无未成年人内容识别、无声纹隐私保护、无模型卡级安全声明或使用限制。
【责任落差尤其突出】相较同类工作，UniTalking 在这一维度的缺失更值得关注，原因有三：
1) 模型能力本身即为高风险深度伪造能力——给定任意人物图像（TI2AV）+ 任意人声参考（TR2AV）+ 任意文本，即可生成该人物说出任意内容的带声视频，这正是伪造名人发言视频的完整技术路径；
2) 数据侧主动构造了音色克隆训练对——用 IndexTTS2 对真实视频中的真人音色做零样本克隆，批量生成 690 万条克隆语音用于训练，这一步骤在声纹权层面本身即需要合规审视，论文未有任何讨论；
3) 论文以「开源可及」为核心动机，若真的发布权重，上述能力将无门槛扩散，而全文没有一句关于滥用防范、水印、内容溯源或使用条款的表述。
【上游的部分兜底】OpenHumanVid 为防止数据集被滥用，要求下载者提交用户信息并经审核批准——上游数据方有明确的滥用防范意识，而下游模型方（UniTalking）反而没有承接这一意识。[不确定]

### [UniVerse-1](../models/UniVerse-1.md) ⚠️

论文完全未涉及安全与合规过滤：没有 NSFW 检测、没有版权过滤、没有人脸隐私保护措施（虽然使用 RetinaFace 做人脸检测，但目的是筛选说话人镜头而非隐私脱敏），没有模型卡级的安全声明或使用限制说明。这与其 Apache-2.0 完全开放商用许可、且模型具备任意人脸唇同步生成能力（可用于制作伪造发言视频）之间存在明显的责任落差。考虑到该模型主打「参考图 + 文本 → 带语音的说话人视频」，其被滥用于深度伪造的风险显著高于纯音效类模型，而论文对此零讨论。[不确定]

### [Unison](../models/Unison.md) ⚠️

论文完全未涉及安全与合规过滤：无 NSFW 检测、无版权过滤、无人脸隐私保护措施、无有害内容审查、无模型卡级安全声明或使用限制说明、无输出侧水印。
【风险评估——本条目中风险敞口较大的一项】Unison 的能力画像是「给定一张人物图像与文本，生成该人物开口说出指定内容的带声视频」，且论文明确追求词级唇同步（word-level lip-sync），LSE-C 达 3.30、LSE-D 7.88，接近 LTX-2 水平。这正是深度伪造（deepfake）的核心能力。同时其训练数据 HDTF、VFHQ、CelebV-Text 大量包含真实名人与公众人物面孔。在这一背景下，论文对滥用风险零讨论、对肖像权零讨论、对输出溯源零措施，是明显的责任缺口。
【值得注意的是人脸检测的用途反向】pipeline 中确实使用了人脸检测器，但目的是圈定 SyncNet 的运行区域以筛选合格的唇同步样本，与隐私脱敏或人脸模糊化恰好相反——检测人脸是为了更好地学习人脸。
【间接的风险缓释】训练语料全部来自学术开源数据集（多限定非商业研究用途），且模型尚未开源，短期内滥用面有限。但论文承诺「接收后公开代码与模型」，届时若无配套的使用条款与安全措施，风险将实质化。论文对此无任何前瞻性说明。[不确定]

### [Veo 3 / Veo 3.1](../models/Veo_3_Veo_31.md) ⚠️

官方披露相对充分的一项。预训练阶段（pre-training mitigations）措施包括：按风险领域（risk areas）对预训练数据做安全过滤；对 caption 过滤不安全内容与个人可识别信息（PII）；训练视频按合规与安全指标过滤；对训练数据做有害内容分析与人群表征的公平性审查；移除重复及概念相似视频。风险领域覆盖儿童性虐待与剥削材料（CSAM）、仇恨言论、骚扰、错误信息、深度伪造、色情内容、暴力血腥等。后训练阶段（post-training mitigations）包括 SynthID 水印与生产环境输出过滤。安全策略与 Google 跨产品生成式 AI 框架及 Gemini / Imagen 3 技术报告一致，发布前由 Google DeepMind 责任与安全委员会（RSC）审批。[不确定] 未披露具体分类器、判定阈值与被过滤数据比例；人脸/隐私仅覆盖到 caption 侧 PII，视频侧人脸隐私处理策略未说明。

### [Vidu S1](../models/Vidu_S1.md) ⚠️

在第4级过滤中设有独立的内容安全过滤（Content Safety Filter），剔除 NSFW 及其他不当内容，动机是防止模型学到有害信息。未提及版权过滤、人脸隐私/肖像权处理、名人过滤等机制 [不确定]。

### [Wan 2.5 / 2.6 / 2.7](../models/Wan.md) ⚠️

披露有限，且集中在训练数据侧的单点动作与推理侧的合规标识。
【训练数据侧】
- NSFW：内部安全评估模型对「所有训练数据」计算 NSFW 分并系统性过滤不当内容（Wan 2.1 基础维度阶段的固定项）。
- 版权/权属：仅通过「内部版权来源」的获取端约束，以及水印与 logo 的检测与训练时裁剪，无下游版权分类器描述。
- 人脸/隐私：无通用隐私脱敏说明。人脸处理集中在个性化子集的构造上（1FPS 人脸检测、任一帧检出多张人脸即丢弃、超10%帧无人脸即丢弃、ArcFace 帧间相似度筛身份一致性、人脸分割去背景、人脸关键点检测用于画布对齐；且刻意不过滤小面积人脸，因为这类视频通常含全身人物）——这些是能力构建而非隐私保护动作。
- 未提及 CSAM 筛除、红队流程、名人肖像限制（相反，caption 模型专门训练了含数千身份的名人/地标/影视角色识别数据集）。
【推理侧】API 的 watermark 参数（右下角固定文案「AI 生成」）；负向提示词机制；prompt_extend 改写；商用服务受中国生成式 AI 服务管理办法与内容标识办法约束。
2.5/2.6/2.7 的安全过滤策略完全未公开。[不确定]

### [音视频生成评测基准合集](../models/av_benchmarks.md) ⚠️

披露有限：
【VABench】I2AV 路的高质量图像在人工精选与分类阶段做了隐私筛查（privacy-screened），是五者中唯一明确提及隐私过滤的。
【AV-SyncBench】人工阶段剔除语义模糊样本，未提及 NSFW/版权/人脸隐私专项过滤[不确定]。
【PhyAVBench】自录数据涉及 184 名参与者出镜，肖像权与知情同意流程未披露[不确定]；因内容为受控物理演示，天然规避 NSFW 风险。
【AVBench】基于公开数据集 OpenHumanVid，继承其安全过滤；自身未追加安全过滤描述[不确定]。
【Omni-Judge】使用商业模型 API 生成，安全过滤由 Sora 2 / Veo 3 自带的内容安全机制承担。
五者均未提及 NSFW 分类器、版权检测或人脸匿名化的具体工具与阈值[不确定]。

### [视频 Caption 模型生态](../models/caption_models.md) ⚠️

[不确定] 该字段在 captioner 生态中几乎完全空白，仅有间接证据：
· NSFW / 版权 / 人脸隐私过滤普遍在 captioner 上游由生成侧团队完成，captioner 论文一律不讨论。
· 唯一相关的设计是 LTX-2 captioner 的「comprehensive yet factual」原则 —— 只描述看到与听到的，显式禁止情绪解读（emotional interpretation）。这既是防幻觉措施，客观上也降低了 caption 对人物主观状态做出可能有偏见的判断的风险，是本生态中最接近「安全设计」的公开做法。
· AVoCaDO-SFT 含 TikTok-10M 与 ShortVideo 子集，涉及真实人物 UGC 内容，未讨论人脸/隐私处理。
· 涉及说话人身份标注的工作（AVoCaDO 的 (speaker, content) 二元组、LTX-2 的 speaker/language/accent、CineDance 的角色 anchor token 绑定）都在生成与人物身份高度相关的标注，但均未讨论隐私合规。
· 生成侧团队的安全过滤（Movie Gen、Veo 3）在其各自条目中记录，与 captioner 选型无直接耦合。

### [几何/结构化标注数据集合集](../models/geometric_datasets.md) ⚠️

SpatialVID 有显式的安全人工介入：初筛阶段人工审阅并剔除标题含不当词汇（inappropriate terms）的视频，同时剔除“全景相机”类不兼容内容；许可采用 CC-BY-NC-SA 4.0 限制商用。Action100M 的隐私保护来自上游——使用的是已做人脸模糊（face-blurred）处理的 HowTo100M 1,199,096 条版本，且只发布标注不发布视频，进一步降低版权与隐私暴露。WildWorld 天然无隐私风险，画面全部为游戏引擎渲染的虚拟角色与怪物，无真实人脸。SceneScribe-1M 未描述独立的 NSFW/人脸/版权过滤模块，安全性依赖上游数据集（HD-VILA、Panda-70M、Koala-36M）已有的清洗[不确定]

### [视频生成后训练数据策略](../models/post_training_data.md) ⚠️

锚论文 3.1 节把「unsafe outputs（不安全输出）」列为 SFT 阶段要消除的失败模式之一，但无任何具体过滤方法、分类器或数据处理描述 [不确定]（且如前所述该表述疑似来自 LLM 后训练文本的迁移）。第 6 节 Broader Impact 讨论的是商业应用价值而非风险缓解。
【横向】Sora 2 是唯一在后训练维度详述安全的对象，但其内容属于安全对齐评测而非能力后训练：通过定向红队收集「数千条对抗性 prompt」按用例与政策领域分类，用 helpful-only 版本视频模型生成输出后打分，转化为自动化评测集，测量 not_unsafe 与 not_overrefuse 两项指标（成人裸露/性内容不涉肖像 96.04%/96.20%、涉肖像 98.40%/97.60%、自残 99.70%/94.60%、暴力血腥 95.10%/97.00%、违规政治说服 95.52%/98.67%、极端主义仇恨 96.82%/99.11%）。Veo 3/3.1 官方所称的「post-training mitigations」实指 SynthID 水印与生产环境输出过滤，属部署侧而非数据侧。InstructAV2AV 的 Qwen3-Omni 五维自动验证把「安全性」作为训练样本准入条件之一，是学术侧少见的把安全并入 SFT 准入的做法。
【整体判断】后训练阶段的安全工作在公开材料中几乎全部集中在「输出侧拦截 + 红队评测」，而非「偏好数据中构造安全偏好对」。用 RLHF 做安全对齐（LLM 领域的标准做法）在视频生成领域尚未见公开实践。[不确定]

### [主流视频预训练数据集合并调研：Panda-70M、InternVid、Koala…](../models/pretraining_datasets.md) ⚠️

**七者的安全合规过滤整体薄弱，仅两家有专门的安全分类器**：
- **InternVid（有专门分类器）**：使用**二分类器识别并排除非伦理/NSFW视频**，同时在爬取源头就把检索词与频道限定在 SFW 范围。这是七者中最早也是最明确的安全环节。
- **MiraData（有专门分类器）**：使用 **Stable Diffusion Safety Checker**，对每条视频**均匀采样8帧**逐帧检查，NSFW片段从**包括788K未过滤池在内的所有版本中移除**——即安全过滤是前置且不可绕过的，这一点做得比其他家规范。
- **Panda-70M（有内容与隐私处理但无NSFW分类器）**：发布前用内部自动化流程过滤含有害/暴力语言以及提及毒品、仇恨言论的样本，并用 **NLTK 把caption中所有人名替换为「person」**。这是七者中唯一的隐私脱敏措施。但**没有视觉NSFW检测**。
- **Koala-36M（仅靠人工标注间接传递，是明确的弱点）**：唯一涉及安全的地方在人工标注准则的「视频自然度」维度——要求标注员剔除「政治、恐怖、暴力、血腥或其他令人不适的内容」。后果是：**安全性被折叠进一个阈值为2.5的标量 VTSS 中，与质量完全纠缠**，一条不安全但美学与动态优秀的视频完全可能通过。**没有独立的NSFW分类器，也没有明确的色情内容类目**，无法审计、无法单独调整。这在七者中是最应被点名的合规缺陷。
- **UltraVideo**：**无 NSFW 过滤、无人脸/隐私过滤**。其16项低质属性判断针对的是画质缺陷而非内容安全。安全性隐含依赖于「YouTube 4K/8K 精品池 + 人工复检」这一较干净的源头。
- **OpenVid-1M**：论文、仓库与HF卡片中**均未提及任何NSFW或安全过滤**。[不确定]
- **LVD-2M**：全文 grep 确认 **「NSFW」「watermark」出现0次**，**无任何安全过滤**，隐含依赖上游四个数据集各自的清洗。
- **版权内容检测**：七者**全部没有**。
- **输出侧**：七者均无生成内容水印或检测工具（它们是数据集，但配套发布的 UltraWan、MVDiT、ViCLIP 等模型也都未附带安全分类器）。
**总体判断**：安全过滤的完备度排序为 MiraData ≈ InternVid > Panda-70M（偏隐私与文本） > UltraVideo（靠源头） > Koala-36M（纠缠在VTSS里） > OpenVid-1M ≈ LVD-2M（无）。这与「质量过滤越做越精细」形成鲜明反差——**七个数据集在两年间把画质过滤从无做到72B VLM，安全过滤却基本原地踏步甚至倒退**。
