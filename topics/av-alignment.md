# 横向对比：音视频对齐

> 主题: 视频生成模型（含音视频同时生成）的数据处理：数据清洗流程、数据分布、打标方式

[← 返回首页](../index.md)

本页按字段横向对比所有条目。⚠️ 表示该条目此字段部分信息不确定。

**字段**: [音视频同步检测方法（口型同步、事件对齐）](#音视频同步检测方法口型同步事件对齐) · [同步检测具体指标与阈值（SyncNet/Synchformer/LSE/自研，阈值数值，如MOVA LSE-D≤9.5且LSE-C≥4.5、SkyReels-V4 SyncNet |offset|≤3∧conf>1.5）](#同步检测具体指标与阈值syncnetsynchformerlse自研阈值数值如mova-lse-d95且lse-c45skyreels-v4-syncnet-offset3conf15) · [时序同步vs语义同步的分离处理（时间对齐与内容语义匹配作为两个独立过滤条件）](#时序同步vs语义同步的分离处理时间对齐与内容语义匹配作为两个独立过滤条件) · [音频质量过滤（SNR、静音检测与静音占比阈值、无音轨剔除、画外音源剔除、背景音乐分离）](#音频质量过滤snr静音检测与静音占比阈值无音轨剔除画外音源剔除背景音乐分离) · [语音/音效/音乐的分类与分别处理策略](#语音音效音乐的分类与分别处理策略)

## 音视频同步检测方法（口型同步、事件对齐）

`av_sync_detection` · 详细程度: detailed

### [Allegro](../models/Allegro.md)

不适用。无音频模态，不存在音视频同步检测（口型同步或事件对齐）环节。

### [Apollo](../models/Apollo.md) ⚠️

音视频一致性检测是 Apollo 数据过滤中方法披露最明确的一环，采用**双工具、双维度**设计（置于音频过滤之后、数据分流之前）：
【时序对齐】使用 Synchformer 检测音视频的时间同步性。Synchformer 是基于 Transformer 的稀疏音视频同步检测模型，可估计音画时间偏移，也是 MMAudio、MOVA 等工作采用的同一工具（并对应评测指标 DeSync / AV-A）。
【语义对齐】使用 ImageBind 检测音视频的语义匹配度。ImageBind 将图像、音频等六种模态嵌入统一空间，通过跨模态余弦相似度衡量「画面里的东西和听到的声音是不是一回事」。
【论文原文】「We then assess audio–visual consistency, using Synchformer for temporal alignment and ImageBind for semantic alignment, ensuring high synchronization in both temporal and semantic dimensions.」
【唇同步专项】数据过滤环节未提及 SyncNet/LSE-D/LSE-C 等唇同步专用检测器——这是一个值得注意的缺口：Apollo 的核心卖点之一是解决「poor lip–speech alignment（唇语对齐差）」，但其数据侧的对齐过滤只用了通用的 Synchformer + ImageBind，未见针对人脸口型的专项过滤（对比 MOVA 明确用 LSE-D ≤ 9.5、LSE-C ≥ 4.5 筛出 2.5M 唇音高质量子集）。SyncNet Confidence 在 Apollo 中只作为**评测指标**（Table 3 中 Sync-conf 从 5.024 提升到 6.787）出现，而非数据过滤器。若属实，说明 Apollo 的唇同步能力主要靠架构（Omni-Full Attention）与多任务训练而非数据侧的唇同步筛选获得。[不确定]（唇同步是否用于数据过滤未明示）

### [CineDance / CineDance-1M](../models/CineDance.md)

音视频同步在本工作中分两条线：数据侧作为元数据指标全量计算，评测侧作为 CineBench 的独立维度：
【数据策展侧（阶段一）】
  · SyncNet —— 用于唇同步（lip-sync）检测，衡量说话镜头中口型与语音的时间对齐；
  · ImageBind —— 用于全局跨模态语义对齐（global alignment），衡量画面内容与音频内容在语义空间的匹配度，覆盖非语音场景（如音效与事件的对应）。
  两项均计算后存为元数据，不作硬性过滤。
【评测侧（CineBench 的 AV Sync 维度）】
  · Sync-C / Sync-D（SyncNet 的置信度与偏移距离两个分量）；
  · IB-Score（ImageBind 跨模态相似度）。
【设计取向】论文有意区分「唇同步」（SyncNet，针对语音-口型的帧级时序对齐）与「全局语义对齐」（ImageBind，针对整体音画内容匹配），构成两个互补的检测口径。
【局限】未使用 Synchformer、AV-align 等事件级同步检测方法；未见针对音效-事件（foley event）时序对齐的专门检测手段。

### [CogVideoX](../models/CogVideoX.md) ⚠️

CogVideoX 视频侧无音视频对齐环节（不使用音轨）。
CogSound 侧的对齐是建模层面的机制而非数据过滤层面的检测：
· 分块时序对齐交叉注意力（Block-wise Temporal Alignment Cross-attention）：通过学习帧级视频特征与音频特征之间的关系，把视频与音频的特征精确连接，确保音频与视频在时序与语义两个层面的一致性。
· 旋转位置编码（RoPE）：为音频序列每个位置提供唯一标识并捕捉相对关系，提升长时序任务下的时序一致性与稳定性。
· 基于 GLM-4V 的视频理解：先准确识别视频背后的语义与情感，再据此生成匹配的音频，属于语义层对齐。
数据侧是否做过同步性检测与过滤（口型同步、事件对齐）完全未披露 [不确定]。CogSound 明确定位为音效（sound effects）生成而非对白配音，未见任何口型同步（lip-sync）相关能力或指标。

### [Cosmos-Predict2.5](../models/Cosmos-Predict25.md)

不适用。模型不处理音频，pipeline 中不存在唇同步检测、事件级音画对齐或任何音视频同步环节。该工作中与「对齐」相关的检测全部在视觉/文本域：文本-视频对齐由 VideoAlign 奖励模型的 Text Alignment 维度评估（RL 阶段）；多视图之间的一致性由 TSE 与 CSE 指标评估（Cosmos-Transfer2.5 多视图驾驶模型，见效果对比）；控制信号与生成内容的对齐由车道检测 F1、立方体检测 LET-AP 等下游感知指标间接评估。

### [Data-Juicer 2.0](../models/Data-Juicer.md) ⚠️

[不确定] Data-Juicer 未提供专门的音视频同步检测算子，这是其在 AV 生成数据处理链上最关键的能力缺口。
【最接近同步检测的算子】video_active_speaker_detect_mapper —— 「通过分析视觉人脸轨迹与音频信号检测视频中的主动说话人」。该算子的底层原理（如 TalkNet/Light-ASD 类主动说话人检测模型）确实依赖唇部运动与音频的时序相关性，因此隐含了唇同步判断；但它的输出是「谁在说话」的身份/时段判定，而不是「音画偏移量是多少帧」「同步置信度多高」的可用于阈值过滤的连续分数。使用者无法直接用它做 LSE-D/LSE-C 式的同步质量筛选。
【缺失的能力】
  · 无 SyncNet 算子——无法输出音画偏移量（offset）与置信度（confidence），因而无法复现 SkyReels-V4 的「|offset|≤3 且 conf>1.5」式过滤。
  · 无 Synchformer 算子——无法对通用音视频事件（非语音）做时序对齐打分，因而无法复现 Foley-Omni 的 sync score ≥0.2 式过滤。
  · 无 AV-HuBERT、Perceiver 类音视频对齐表征算子。
  · 无 ImageBind 算子——无法做音视频跨模态语义一致性打分。
【视觉-文本对齐是有的】video_frames_text_similarity_filter 基于 CLIP 做「视频帧-文本」的跨模态一致性过滤，这是 DJ 唯一成熟且经实战验证的跨模态对齐算子（T2V 案例的核心判据，阈值0.306337）。但它处理的是视觉-文本轴，不是视觉-听觉轴。
【结论】若要用 Data-Juicer 构建音视频联合生成的训练数据，同步检测环节必须自行扩展算子。好在 DJ 的算子注册机制（继承 Filter 基类 + YAML 声明 + stats 字段写入）使得封装 SyncNet/Synchformer 为自定义算子的工程成本较低，且可直接复用其 Ray 分布式执行、GPU 自适应分配、批处理优化等基础设施——这正是 DJ 作为「算子库」而非「固定 pipeline」的价值所在。

### [Foley-Omni](../models/Foley-Omni.md) ⚠️

同步检测是过滤阶段「对齐维度」的一半，且同一技术栈贯穿清洗、建模、评测三个环节，一致性很强。
【清洗阶段】使用 Synchformer 计算 sync score，阈值 ≥0.2，剔除「unreliable synchronization」（同步不可靠）的片段。Synchformer 是基于 Transformer 的音视频同步检测模型，相比经典 SyncNet 的优势是不局限于口型同步，能处理通用音视频事件的时序对齐（如撞击声与撞击画面），这对以音效为重要生成目标的本任务是必要的——若用 SyncNet，非人脸场景的音效数据会被大量误杀。
【建模阶段】Synchformer 特征被直接用作模型的时序条件输入，通过一条 additive path（记为 Z_sync）注入 DiT。这意味着「用于过滤的同步表征」与「用于生成的同步条件」是同一个，过滤标准与模型能力天然对齐——通过过滤的样本正是模型能有效利用其同步信号的样本。这个设计在同类工作中值得注意。
【评测阶段】使用 DeSync 指标（越低越好）衡量生成音频与视频的时序对齐质量。Foley-Omni 在 V2ST-Bench 上 DeSync 0.16，非常接近 Ground Truth 的 0.14，显著优于级联基线 MMAudio+CosyVoice 3+AudioX 的 0.85 与 MMAudio+LipVoicer+AudioX 的 0.26。
【消融证据】Table 6 显示移除 Z_sync 同步附加通路后，IB 从 0.26 降至 0.22、V2ST WER 从 7.59 升至 12.40、FD_VGG 从 1.57 升至 2.21，三项指标同时劣化，证实同步特征通路对音视频一致性与语音质量均有实质贡献。
【口型同步】[不确定] 论文未在清洗阶段部署专门的唇同步检测器（如 SyncNet confidence），VisualTTS 支路的唇同步质量依赖数据集本身（GRID/LRS2 天然是唇读数据集，同步性有保证）与 Synchformer 的通用同步判据。这在纯语音场景下可能不如专用唇同步指标精确。

### [Goku](../models/Goku.md)

不适用。Goku 无音频模态，数据流水线中不存在音视频同步检测环节（无口型同步检测、无音画事件对齐、无 SyncNet/AV-align 类方法）。其数据流水线中与「对齐」相关的唯一概念是**视觉时序一致性**：用 DINOv2 相邻帧余弦相似度（480p ≥0.85、720p ≥0.90）保证片段内视觉连贯不跳变——这是视觉内部的时序一致性约束，与音视频跨模态同步在方法论上不同源。

### [HunyuanVideo-Foley](../models/HunyuanVideo-Foley.md)

音视频对齐检测是本工作数据流水线中方法论价值最高的一环，且在数据侧与模型侧形成了完整闭环：
【数据侧：双工具双维度检测（第 6 级）】
- ImageBind —— 负责语义对齐（semantic alignment）。把音频与视频分别编码到 ImageBind 的共享多模态嵌入空间，计算二者的相似度，判定「这段声音在语义上是不是这段画面该有的声音」。它能识别的典型错配是：画面是厨房切菜但音轨是流行音乐、画面是户外风景但音轨是室内旁白配音。这类后期配乐/旁白样本在网络视频中极为普遍，是 V2A 训练数据最主要的污染源，ImageBind 正是针对它的解药。
- AV-align —— 负责时序对齐（temporal alignment）。检测音频事件的起音点（onset）与视觉事件的变化点在时间轴上是否对应。它能识别的典型错配是：语义正确但存在系统性时延（音轨整体偏移几百毫秒）、或声音事件密度与画面事件密度不匹配。
【模型侧的对应机制】数据侧筛出的对齐样本，在模型侧由三重机制学习：(1) Synchformer 帧级同步特征通过门控调制注入，提供显式的时间轴锚点；(2) interleaved RoPE 把音、视 token 按时间交错编码，使同时刻的跨模态 token 位置相邻；(3) joint self-attention 让音视频 token 直接互相关注。数据筛选保证「学的是对的」，架构设计保证「学得进去」，两侧职责清晰。
【评测侧】用 DeSync（基于 Synchformer 计算的去同步程度，越低越好）衡量时序对齐，用 ImageBind 余弦相似度（IB，越高越好）衡量视觉语义对齐。三个基准上的成绩：Kling-Audio-Eval 上 DeSync 0.54 / IB 0.38（MMAudio 为 0.56 / 0.30）；VGGSound-Test 上 DeSync 0.53 / IB 0.36；MovieGen-Audio-Bench 上 DeSync 0.74 / IB 0.35（MMAudio 为 0.80 / 0.27）。IB 指标相对基线提升 26.7%，是提升幅度最大的维度之一。
【注意一处循环风险】ImageBind 既用于筛数据又用于评测 IB 指标，Synchformer 既用于模型条件注入又用于评测 DeSync 指标——两处「训练用什么、评测就用什么」的重合会系统性抬高相应指标的表现。论文未讨论这一评测偏置，读者在横向对比时应有所折扣。

### [HunyuanVideo](../models/HunyuanVideo.md) ⚠️

不适用。无音频模态，训练数据不含音轨，因而不存在音视频同步检测环节。[不确定]

### [InstructAV2AV](../models/InstructAV2AV.md) ⚠️

音视频同步在本工作中被处理为三个不同环节的问题——数据准入、合成时强制、结果评测，三处采用不同机制，是本pipeline中同步处理最有层次的部分。

【环节一：数据准入阶段的同步保障（源头控制，非分数过滤）】
  不使用 SyncNet/Synchformer 之类的同步打分器做阈值过滤，而是用「声画归属」这一更强的准入条件：TalkNet 定位主动说话人 + Scribe 提取语音时间戳，仅保留「语音与画内可见说话人在时间上对齐」的片段。非语音侧则要求单一清晰声源（剔除 ambiguous sound sources）。这实质是从源头只准入声画同源的素材，而非在混杂素材中筛出同步的——比分数阈值过滤更彻底，因为它排除的是「声音根本不来自画面」这类结构性问题，而非仅仅时间偏移。

【环节二：数据合成阶段的同步强制（生成时保证）】
  这是本工作最独特的一点：同步不是过滤出来的，而是生成时被强制的。mask-guided 视频编辑模型通过 frame-wise cross-attention 把已合成的目标音频特征逐帧注入视频生成过程，论文表述为「ensuring strict temporal synchronization」。合成顺序为先音后视，让口型/动作服从既定的音频时间轴。相比「先各自生成再筛掉不同步的」，这是效率高得多的做法——不合格样本压根不会被生成出来。

【环节三：自动验证阶段的同步复检】
  Qwen3-Omni 五维度打分中的第四项即「audio-video synchronization for cross-modal alignment」，作为必须通过的硬条件之一，对合成结果做语义层面的同步复核。这是用 MLLM 而非专用同步模型做判断，优点是能捕捉语义级的不匹配（如猫的画面配狗叫），缺点是时间精度不如 SyncNet 等专用模型。

【环节四：模型评测阶段的同步度量】
  使用四项音视频指标：AV-A（基于 ImageBind 的音视频语义对齐）、PEAVS（Perceptual Evaluation of Audio-Visual Synchrony，感知级同步评估）、Sync-C（SyncNet 置信度，越高越好）、Sync-D（SyncNet 距离，越低越好，配置为 vshift=15 帧）。InstructAV2AV 在 InsAVE-80K 评测集上 AV-A 27.72，优于 AvED 26.44、AVI-Edit 26.37、CoherentAVEdit 22.67；AvED-Bench 上 AV-A 23.71 vs AVI-Edit 23.21。

【方法论评价】「源头准入 + 生成时强制 + MLLM 复检 + 评测度量」的四层设计比单纯依赖同步分数阈值更完整，且各层职责清晰（结构性问题在准入层解决、时间精度在生成层保证、语义匹配在验证层把关、最终能力在评测层度量）。
[不确定] 数据构造全流程中未使用任何专用同步检测模型（SyncNet/Synchformer/AV-HuBERT）做定量过滤，因此无法给出数据集本身的同步质量分布统计；也未做「有无 frame-wise cross-attention」的数据引擎消融来量化该机制对合成数据同步质量的贡献。

### [2026 其他音视频联合生成](../models/JAVG_2026_misc.md) ⚠️

本批工作在同步检测上分为三档，且出现了工具多样化的趋势：
【NAVA —— 三检测器并用（本批最全面）】
数据过滤阶段的音视对齐算子明确为三个并行：「audio-visual alignment, measured by SyncNet, SyncFormer, and ImageBind」——SyncNet（唇音时序同步，经典口型检测）、SyncFormer/Synchformer（Transformer 架构的通用音视同步检测，可处理非语音事件对齐）、ImageBind（跨模态语义相似度，度量音画内容是否讲同一件事）。这三者恰好覆盖三个不同层次：SyncNet 管唇音时序、Synchformer 管通用事件时序、ImageBind 管语义匹配——是一个层次分明的对齐检测体系，比单用 SyncNet 的做法（如 Ovi、OmniCustom）完整得多。全部阈值未公开[不确定]，这是 NAVA 数据披露最大的遗憾。
【OmniCustom —— 单一 SyncNet 双阈值（阈值完全公开）】
沿用 Ovi 的标准：「|offset|≤3」且「confidence >1.5」。这组数值与 Ovi 论文完全一致（|offset|≤3 帧、confidence>1.5），说明 Ovi 的阈值已成为该领域的事实参考标准并被后续工作直接继承。按 24fps 计，|offset|≤3 帧即时间容差 ≤125 毫秒。仅针对含语音片段的唇音同步，无非语音事件对齐检测[不确定]。
【ALIVE —— TS-TalkNet 做主动说话人检测（用途不同于纯同步过滤）】
使用 TS-TalkNet（Target-Speaker TalkNet）「to evaluate the active speaker」（评估主动说话人），产出人脸-声音对应矩阵。注意其主要用途不是「过滤不同步样本」而是「判断多人画面中谁在说话」——这是一个更难的任务（active speaker detection 而非 lip-sync verification）。sync score 的另一用途是选取身份锚点（同步分最高的 1.5 秒子片段）。阈值未公开，仅说明「Filters videos failing high-confidence thresholds」（过滤未达高置信阈值的视频）[阈值数值不确定]。
此外 ALIVE 的音画相关性判断改由 MLLM 承担（见 model_as_data_judge），与 TS-TalkNet 的说话人检测形成分工：MLLM 管语义相关性、TS-TalkNet 管说话人归属。
【StreamChar / CCL / Baton —— 数据侧无同步过滤】
三者均未描述数据侧的同步检测[不确定]，依赖上游数据集（SpeakerVid-5M / TalkVid / OpenHumanVid 本身带同步筛选）或模型架构保证。CCL 在评测中报告 Sync-C / Sync-D / DeSync 三个同步指标，说明重视同步性但把保障放在架构（TARP 时序对齐 RoPE）而非数据侧。
【ITS-JAVG —— 把同步检测搬到推理时】
用 JavisScore 作为细粒度音视频同步验证器（「focuses on fine-grained audio-video synchronization」），另有 AVHScore 度量音频事件与视觉事件的语义一致性、ImageBind 度量音视频语义相似度。关键发现是只用 JavisScore 引导搜索会导致其他指标退化（非对称权衡），且搜索会钻同步验证器的空子。这对数据侧的启示很直接：仅用 SyncNet 单一阈值过滤，保留下来的数据会系统性偏向 SyncNet 的偏好（例如偏向正脸、清晰口型的样本），造成分布扭曲——NAVA 的三检测器并用恰好可以缓解这一问题。作者 Joon Son Chung 正是 SyncNet 原作者之一，其对同步度量局限性的判断尤具权威性。

### [音视频联合生成基线合集](../models/JavisDiT_baselines.md) ⚠️

音视频同步检测是本合集技术差异最大的维度，五项工作分别代表五种不同思路：
【MM-Diffusion（2022）—— 无同步检测，靠架构与数据选型】数据侧完全不做同步过滤。同步性靠两点保证：(1) 数据集选型——Landscape 的 9 类场景都是声画强因果绑定的自然现象，AIST++ 是舞蹈与音乐的节拍绑定；(2) 架构——random-shift based attention 块在时间邻域内做跨模态注意力。评测侧也没有专门的同步指标，只用 FVD（视频质量）+ FAD（音频质量）+ 人工图灵测试。这是「同步性无法量化，只能靠人眼」的早期阶段。
【AV-DiT（2024）—— 同样无同步检测】沿用 MM-Diffusion 的数据与评测设定。
【JavisDiT（2025）—— 自研同步指标 JavisScore，是本合集最重要的方法学贡献】作者指出已有同步指标在真实复杂内容上不可靠，因此提出 JavisScore：
- 计算方法：把每个音视频对切成 2 秒窗口、1.5 秒重叠的多个片段（即滑窗步长 0.5 秒）；对每个片段用 ImageBind 计算音视频同步性；具体做法是计算片段内所有帧与该片段音频的相似度，然后取「同步性最差的 40% 帧」参与打分（而非取平均）——这个「取最差 40%」的设计是关键，因为取平均会被大量对齐良好的帧稀释掉局部失步，而真实的失步感知恰恰由最差的片段主导。
- 有效性验证：构建 3,000 条人工标注样本的评估数据集，验证 JavisScore 比已有指标更贴近人类判断。
- 训练侧的同步机制：HiST-Sypo 时空同步先验 + 用合成异步负样本做对比学习训练先验估计器。即 JavisDiT 不做数据过滤式的同步筛选，而是把同步性建模成一个可学习的先验。
【JavisDiT++（2026）—— 显式帧级同步 + Synchformer 作为奖励】TA-RoPE（Temporal-Aligned RoPE）在位置编码层面强制音频 token 与视频 token 的帧级对齐（与 Ovi 的 scaled-RoPE 同源思想）；AV-DPO 阶段用 Synchformer 作为时序同步的奖励模型之一，把同步性纳入偏好优化目标。评测报 DeSync 指标。
【Harmony（2025）—— 三管齐下】(1) 训练机制层面：Cross-Task Synergy 用双向生成任务（音频驱动视频、视频驱动音频）抑制联合去噪的对齐漂移；(2) 架构层面：GLDI 模块把全局风格对齐与局部时序精度解耦到两个分支；(3) 推理层面：Synchronization-Enhanced CFG 放大对齐信号——消融显示这一项贡献了最大的同步增益（Sync-C 从 5.09 提升到 6.51）。数据侧则用「音视频一致性打分模型」筛选语音数据。
【UniAVGen（2025）—— 架构层面的非对称交互】靠非对称跨模态交互 + Face-Aware Modulation + 模态感知 CFG 保证同步，数据侧未描述同步过滤[不确定]。评测用 SyncNet 指标。

### [可灵 3.0 / 可灵视频 3.0 Omni](../models/Kling_30_Omni.md) ⚠️

[不确定] 可灵3.0 Omni 训练数据的音画同步检测方法未公开。Kling-Omni 报告在基础级仅提到“音视频损坏检测（audio-visual corruption detection）”，第三级多模态对齐主要针对 caption-视觉、参考图-视频、角色身份一致性，未点名口型同步检测。KlingAvatar 2.0 称有“广泛的过滤 pipeline 以确保高视觉保真与一致的音-唇同步（consistent audio-lip synchronization）”，确认存在唇同步过滤环节但未给方法。Kling-Foley 侧则在模型架构中内置了帧级的“视听同步模块”与视觉语义表征模块做条件对齐，并在评测中区分语义对齐与时序对齐两类指标——说明团队对“事件级音画对齐”有成体系的度量能力。综合判断：可灵3.0 Omni 的数据侧同步过滤应至少包含唇同步打分（语音类）与事件级时序对齐打分（音效类）两条，但均无公开细节。

### [LTX-2](../models/LTX-2.md) ⚠️

这是 LTX-2 数据披露中最大的空白，也是与调研主题最相关却最缺失的一环。技术报告未描述任何训练数据侧的音视频同步检测或异步样本剔除机制——没有唇同步检测器、没有事件对齐检测、没有异步剔除阈值。
其同步能力的来源被归结为三点，全部在数据筛选与架构侧而非同步检测侧：
(1) 数据侧：只保留「音频成分显著且信息量丰富」的片段——即保留原生自带同步音轨的真实视频，天然规避了配音/后期贴音等异步样本（但论文未说明是否显式剔除画外音、后期配乐等非同期声）；
(2) 标注侧：caption 精确转写对白并标注说话人，使文本-语音-唇动三者在语义层可对齐；
(3) 架构侧：贯穿全深度的双向音视频 cross-attention，且跨模态注意力只使用 1D 时间 RoPE，强制注意力集中在时间同步维度，论文称可以「亚帧精度（sub-frame precision）」把视觉事件（如物体撞击）映射到听觉事件（对应的 foley 声）；cross-modality AdaLN 进一步调控跨模态信息注入强度；推理时增大跨模态引导 s_m 可提升时序同步。
论文 Fig.3 通过可视化 AV cross-attention 图作为同步能力的定性证据：模型能空间跟踪行驶车辆、在两位说话人之间动态切换注意力、并在近景说话时聚焦唇部区域。[不确定]

### [LongCat-Video](../models/LongCat-Video.md)

基础版不适用（无音频）。
Avatar 1.5 将唇同步校验作为离线标注阶段的独立一环与在线过滤的首道关卡：离线做 lip-sync verification，移除「samples with large audio-visual offsets」（音视频偏移过大的样本）；在线 clip 级校验中「audio synchronization」是渐进过滤链的第一级。配套的主动说话人检测使用 TalkNet 与 UniTalk 两个模型联合判定，确保音轨归属于画面中的目标人物而非画外音；多人场景以 YOLOv6 + ByteTrack 跟踪关联身份，并剔除多人并发说话区间。另有专门的静默数据分支（Qwen3-Omni + Qwen3-VL 双模型一致判定主体未说话）以覆盖「有音无口型」的合法情形。事件级（非唇动）的音视频对齐检测未涉及。

### [MOVA](../models/MOVA.md)

MOVA 把音视频对齐做成了数据过滤中的一等公民，且在不同阶段使用不同工具、针对不同粒度：
【Stage 2 通用对齐过滤（双工具）】
- 时序同步：SynchFormer 计算每条视频的音视频时序同步性（DeSync 偏移量）。
- 语义对齐：ImageBind 计算音视频的跨模态语义对齐分（IB-Score）。
【Phase 2 唇同步专项过滤】使用 SyncNet 系的 LSE-D（Lip Sync Error - Distance）与 LSE-C（Lip Sync Error - Confidence），筛选高质量唇音对应片段。
【音频类型分流】EAT 音频分类模型区分 speech / non-speech，为“唇同步”与“通用 foley/环境音建模”两类能力分别构建子集——即不同能力对应不同的同步判据。
【架构层面的同步保障（非数据侧但强相关）】Aligned RoPE 将视频与音频 latent 映射到同一时间栅格；Bridge 双向 cross-attention 提供逐层跨模态交互；Dual Sigma Shift 让两模态独立采样噪声水平。论文在 Discussion 7.1 中提出一个有洞察力的观点：预定义的 sigma 调度实际上充当了**隐式的同步方向先验**——对唇部特写镜头（目标区域占画面比例大），视觉 latent 信息量相对充分，过程趋向 Video→Audio；当说话人只占画面一小块时，视觉证据相对不确定，过程自然偏向 Audio→Video，由语音提供更可靠的时间锚点。
【论文的核心经验结论】“architectural mechanisms alone (e.g., Bridge modules for cross-modal attention) are insufficient to achieve high-quality lip synchronization—the model must also learn phoneme-to-viseme mappings from data, which requires larger capacity and more training examples.” 即架构机制不足以获得高质量唇同步，音素到口型的映射必须从数据中学习，需要更大容量与更多训练样本。这是 MOVA 对“数据决定音视频同步上限”这一命题最直接的表述。

### [合并条目：① Mochi 1](../models/Mochi_MAGI_Motif.md)

不适用。三个模型均为无声视频生成，训练数据不含音轨，不存在口型同步或音画事件对齐的检测环节。

### [Movie Gen](../models/Movie_Gen.md)

不做口型同步检测（模型不生成语音，故 SyncNet/LSE-C 一类唇同步指标完全不适用）。音视频关联性的检测核心是 CAVTP（contrastive audio-video-text pre-training）模型：计算音频embedding与视频embedding的余弦相似度，作为「该音频有多大可能是画内音（diegetic）」的代理分数——因为CAVTP主要在以画内音为主的数据上训练，画内且与画面内容匹配的音频其音视频embedding距离更近。该分数与AED类型标签组合，构成预训练数据的分桶依据（表24）。
论文还从建模角度把音视频对应关系分为三个难度层级：①画内在屏音——音画对应极强、什么声音在什么时刻响是确定性的，要求强视频理解与稠密动作识别（如高尔夫球杆击球）；②画内画外音——需理解什么环境会出现什么声音、事件间的逻辑顺序（如林中鸟叫、难度动作完成后人群欢呼），要求更强推理；③画外音/非画内——仅在语义层面相关（背景音乐要匹配情绪、riser用于营造紧张感），要求对世界物理之外的人类情感建模。
评测端用人工评「diegetic sound synchronization」并配合 ImageBind score 客观指标。局限性中承认：当动作密集（踢踏舞）、目标小或被遮挡（脚步声）、或需要细粒度视觉理解（识别吉他和弦）时，生成音频会出现不同步。

### [NVIDIA NeMo Curator](../models/NeMo_Curator.md)

无。框架完全不具备音视频同步检测能力：视频 pipeline 不解析音轨，因此不存在唇同步检测、事件对齐检测或异步样本剔除的任何 stage；音频 pipeline 处理的是独立音频文件，无对应视频帧可供对齐。官方文档、Cosmos WFM 论文、NeMo 视频基础模型训练论文中均未出现 SyncNet、Synchformer、AV-align 等任何同步检测组件。
这构成 NeMo Curator 服务于音视频联合生成模型时的首要缺口——尽管它在「大规模视觉数据处理」上是事实标准，但 AV 数据构建的核心环节（同步性筛选）需完全自建。

### [OmniHuman 数据集 + OHBench](../models/OmniHuman.md)

音视同步在 OmniHuman 中被赋予了一个超出常规的角色——它不只是质量过滤条件，更是「说话人归属」这一结构化标注的生成手段，这是本工作最有价值的设计之一：
【常规用途：同步质量过滤】用 SyncNet 判定唇音对齐，不达标的样本被剔除。
【独特用途：音视觉归属（audio-visual attribution）】在多人场景中，仅知道「音频与画面同步」是不够的，还必须知道「这段话是谁说的」。OmniHuman 的解法是一条完整的归属链路：
1) 3D-Speaker 做说话人分离，得到 M 个活跃语音区间及说话人索引；
2) YOLOv11 + MOTRv2 得到画面中每个人的视觉 ID 轨迹；
3) 对每段音频与每条视觉轨迹计算 SyncNet 响应；
4) 贪心匹配（greedy matching）：把每段音频指派给响应最高的那条视觉 ID 轨迹；
5) ArcFace 人脸嵌入确认身份一致性（相似度 > 0.55）。
即 SyncNet 从「打分器」升格为「匹配器」——用同步响应的相对高低来解决归属问题，这比单纯设阈值信息量大得多。这也是 OmniHuman 能提供 speaker annotation 且支持双人对话数据的技术基础。
【保留条件】一个样本只有在「所有被检测到的主体」都满足 S_sync 高于预设阈值、且时间偏移在 ±3 帧以内时才被保留。注意「所有主体」（all detected subjects）这一措辞——这是全员通过制而非多数通过制，对双人及多人场景是相当严格的约束，也解释了为何数据集只覆盖到单人、双人、人-物三类而未涉及更多人的场景（人数越多，全员通过的概率越低）。
【±3 帧的含义】在统一的 30 FPS 下，±3 帧 = ±100 毫秒。这一容差与人类对音画不同步的感知阈值（一般认为音频滞后 100ms 以内、超前 50ms 以内不易察觉）大致相当，是一个合理的工程取值。
【副产物：唇同步质量作为标注保留】帧级标注中含 lip-sync quality assessment，即同步质量不只用于过滤，还作为可查询字段随数据发布，下游可据此筛选更严格的子集。
【事件级对齐：未涉及】数据 domain 为人物中心的语音内容，无 foley 音效的事件对齐检测（无 AV-align、无 Synchformer）。但评测侧用 ImageBind 衡量视听事件的时序偏移（V-A 指标），在一定程度上覆盖了非语音的视听对应。
【评测侧的同步指标】OHBench 个体层用 SyncNet 测唇音对齐准确度；全局层用 ImageBind 测视听事件的时序偏移。二者分工是「口型级」与「事件级」，粒度互补。

### [Open-Sora 系列](../models/Open-Sora.md)

不适用。无音频模态，不存在音视频同步检测环节。

### [Ovi](../models/Ovi.md) ⚠️

Ovi 把音视频同步过滤当作整个数据 pipeline 中优先级最高的一环，并且是唯一被作者用「strict」形容的过滤。
【方法】采用经典的 SyncNet（Chung & Zisserman, 2016）——基于 ConvNet 学习声音与嘴部图像之间的联合嵌入，输出两个标量：confidence（置信度，衡量音画相关性强度）与 offset（时间偏移，衡量音轨相对画面的超前/滞后帧数）。作者明确表示对该模型做了工程改造以处理「百万量级（on the scale of millions）」的视频数据，即把 SyncNet 从研究级脚本改造成大规模批处理组件。
【判定逻辑】仅保留同时满足 |offset| ≤ 3（帧）且 confidence > 1.5 的片段，并叠加音频平均音量 ≥ −60 dB 的条件。
【作用范围】主要针对「speech videos」（含语音的片段），用于剔除唇音不同步的数据；非语音片段的事件级音画对齐（如撞击声与撞击画面）未使用专门的检测器，也未使用 AV-align 类事件对齐指标[不确定]。
【设计理由（重要经验）】论文明确写道：「We have experimentally determined that even a small quantity of out-of-sync data can impede lip-sync abilities and chose these strict criteria to minimize the risk of misaligned data.」——即实验证明极少量不同步数据就足以损害模型的唇同步能力，因此宁可牺牲数据量也要用严格阈值把错配风险降到最低。这是全文关于「数据质量优先于数量」最直接的表述。
【无训练期同步损失】数据侧严格过滤之后，训练阶段不再使用任何显式同步损失、辅助同步模块或人脸 mask，同步完全依靠配对采样 + 共享 timestep + 双向跨模态注意力 + scaled RoPE 自发涌现。可以说 Ovi 把同步性的保证几乎完全押在了数据过滤上。

### [Script-a-Video](../models/Script-a-Video.md) ⚠️

需区分数据侧与评测侧，二者差异显著：
【数据侧（标注阶段）：以 schema 规则替代检测模型】这是本工作最值得关注的思路。MTSS 没有使用 SyncNet 等同步检测模型来过滤数据，而是把音视频对应关系直接编码进标注规则：
1) 严格音视频耦合准入原则——Event 流只收录「有直接视觉对应物或主题相关性」的音频事件，音效必须由画面中可见的主体产生。不满足条件的声音不被剔除而是降级到 global_audio。这在标注层面就完成了「有视觉依据的声音」与「无视觉依据的声音」的分离。
2) 结构性说话人绑定——Event 的 speaker 字段指向 Reference ID，与 Shot 的 references_in_shot 指向同一实体库，使「说话人是否在画面中」成为可通过引用关系直接判定的问题，无需人脸检测 + 唇动分析。
3) 三层时间戳体系——shot 级 time_range、event 级 time_range、描述内 micro-level timestamps 共享全局时间轴，事件对齐由时间戳直接表达，论文称目标是达成 sub-frame（亚帧级）的视听协调与「surgical synchronization」。
即：对齐不是被「检测出来的过滤条件」，而是被「写进结构里的标注内容」。这与 UniVerse-1（SyncNet conf>2.0 硬阈值过滤）、MOVA 等以判别模型做闸门的路线是根本不同的方法论。
【评测侧：使用 SyncNet】生成结果的唇同步准确性用 SyncNet 评估，模型输出同步置信度分数与以帧为单位的时间偏移量。此外用 Shot Boundary Deviation 衡量生成镜头边界与脚本指定边界的帧级绝对偏差。
【论文对同步指标可靠性的重要发现】详见 human_in_loop 与 sync_metric_and_threshold 字段——SyncNet 分数在音频信息稀疏（平坦环境噪声）时会给出虚高的同步表现，是一个具有普遍参考价值的陷阱提示。
【数据侧是否另有同步过滤】论文未披露 500K 数据集或生成侧四套数据是否经过任何音画同步检测过滤，是清洗流程整体缺失的一部分。[不确定]

### [Seedance 2.0 与 Seedance 1.5 pro](../models/Seedance_20_Seedance_15_pro.md) ⚠️

[不确定] 数据侧的音画同步检测方法未在报告中披露。Seedance 1.5 pro 仅在原则层面声明数据管线「优先保障视频-音频一致性（video-audio coherence）」，第三方摘要称第二阶段为「针对音画同步与片段质量的自动化过滤」，但该表述不见于论文原文，属未证实推断。评测侧则有完整定义：SeedVideoBench 1.5 的「音视频同步」指标衡量听觉流与视觉流的时间对齐，评估语音-唇动同步（缓解「腹语术」感知效应 ventriloquism effect）、音效与视觉事件的对齐、以及显著画面动作是否有对应听觉线索；SeedVideoBench 2.0 沿用并按 17 个细粒度类目打分（Seedance 2.0 的 T2V 音画同步总分 3.75，17 类中 16 类第一，可用率 93.30%、满意率 68.30%，远超竞品最高 25.45%）。

### [SkyReels 系列](../models/SkyReels.md)

仅 SkyReels-V4 涉及，且是其数据 pipeline 中唯一的跨模态过滤环节，方法明确：
【方法】使用 SyncNet 作为唇音同步判别器——论文描述其「采用 ConvNet 架构学习声音与嘴部图像之间的联合嵌入（joint embeddings between sound and mouth images）」，通过嵌入距离在时间轴上滑动匹配，输出音视频时间偏移量 offset 与置信度 confidence 两个量，用于剔除后期配音、音画错位、口型不匹配的样本。论文提到该 SyncNet 流程被适配到「数百万条视频样本（millions of video samples）」的规模上做分布式处理。
【定位】同步过滤发生在视频支路过滤与均衡之后、进入音视频联合训练之前，作为跨模态准入门槛。
【局限】(1) 只覆盖唇音同步（说话人场景），未见针对非语音事件（撞击-音效、脚步-画面）的事件级对齐检测（如 AV-align 类指标），音效类样本的时序对齐如何保证未说明；(2) 未说明无人脸/无说话人片段如何处理（这类样本无法计算 SyncNet 分数）；(3) 未说明是否剔除画外音、旁白、后期配乐等非同期声。架构侧的同步保障由逐层双向音视频交叉注意力与 RoPE 时间维缩放对齐承担。

### [Sora 2](../models/Sora_2.md) ⚠️

训练数据侧的音视频同步检测方法完全未披露——这是本次调研最核心的关切点，而 OpenAI 恰恰零披露。System Card 中没有任何关于唇形同步检测、事件对齐检测、异步样本剔除的表述。能力侧仅宣称音频「properly synchronized with on-screen action, including accurate lip-sync for speaking characters」，以及音效与画面事件绑定、音乐节奏匹配场景节奏。可以确定训练pipeline中必然存在某种AV同步质量控制（否则无法学得唇同步），但方法、模块、判据全部未知。[不确定]

### [Step-Video-T2V](../models/Step-Video-T2V.md) ⚠️

不适用。模型无音频模态，训练数据不含音轨，pipeline 中不存在音视频同步检测环节，报告全文无 SyncNet、AV-align、唇同步、事件对齐等任何相关内容。[不确定]

### [UniTalking](../models/UniTalking.md)

音视频同步在 UniTalking 中被拆解为「声源同源性」与「唇音时序对齐」两个独立子问题，分别由两个专用模型在跨模态过滤级串联把关——这一拆分是本工作在对齐检测上最有价值的设计：
【第一道：声源同源性检测（LightASD）】用于「过滤掉含纯 diegetic 音频的视频」。LightASD 是轻量级主动说话人检测（Active Speaker Detection）模型，其任务恰是判定「当前画面中的人是否正在说话 / 音轨中的语音是否来自画面中人」。这一步剔除的是画外音、旁白、后期配音、背景对话等「声音存在但声源不在画面内」的样本。这类样本的音轨与画面在时序上可能毫无关系，是说话人数据集最主要的污染源。UniVerse-1 完全没有对应设施（其明确未做画外音源剔除），这是 UniTalking 相对领先的一点。
【第二道：唇音时序对齐（LipSync）】在确认声源同源后，再评估唇部运动与语音的帧级时序对齐，剔除对齐差的样本。剔除对象是「声源确在画面内但存在音画偏移」的样本——常见成因包括转码/封装导致的音视频时间戳漂移、后期剪辑错位、以及配音替换。
【两道关卡的逻辑关系】前者是语义/来源层面的判定（声音是不是他发出的），后者是时序层面的判定（发出的时刻对不对）。串联顺序合理：先排除根本不同源的，再对同源者校验时序精度。这实质上构成了 temporal 与 semantic 两个维度的分离处理（见 temporal_vs_semantic_sync）。
【事件级对齐：不适用】UniTalking 的数据 domain 是纯语音说话人内容，不涉及 foley 音效的事件对齐，因此无 AV-align、无 Synchformer 类的通用事件对齐检测。论文在 Related Work 中明确批评现有统一生成工作「主要聚焦于 foley 声音与视频的同步，达不到语音所需的精度」，即刻意选择了不同的对齐目标。
【架构侧的对齐保障】数据过滤之外，对齐主要由三项架构设计承担：(1) Joint Attention 对拼接后的音视频 token 做单一注意力，显式建模 viseme-phoneme 对应；(2) 各向异性 RoPE 退化音频空间维度、强化时间维度；(3) 训练中的 TV2A 任务——通过施加「音频 token 到视频 token」方向的注意力掩码，阻断音频对视频分支的影响，迫使音频分支仅依据真实视频的 latent 特征预测音频，从而学习逐帧视觉运动到声学结果的精确映射。这是一个用注意力掩码构造单向监督信号的设计，是训练侧最直接的对齐监督。
【效果验证】Figure 5 的注意力图可视化显示：audio-to-video 注意力中面部与身体区域获得较高权重；video-to-audio 注意力中音频 token「专门（exclusively）」关注唇部区域——这为 Joint Attention 确实学到了唇音对应提供了直观证据。论文同时坦承注意力图中存在随训练迭代随机变化的错配高亮区域，推测源于「训练策略缺陷或数据噪声」——这是全文唯一一处承认数据噪声影响的表述。

### [UniVerse-1](../models/UniVerse-1.md)

音视频同步检测在 UniVerse-1 中是清洗漏斗的最后一道也是最硬的一道关卡，但只针对语音子集，且只做口型同步、不做通用事件对齐：
【口型同步检测（离线过滤用）】两步串联：
1) RetinaFace 人脸检测——先确认画面中存在可供唇同步核验的人脸，无人脸的语音片段无法进入语音子集；
2) SyncNet 唇音同步置信度——对含人脸的语音片段计算 confidence score，要求 > 2.0 方可保留，通过者被显式打上「含语音」标签。
这两步共同把 1,187 小时的高置信语音数据从更大的候选池中筛出。
【事件对齐（非语音）】论文未对非语音的通用音视频数据做任何时序对齐检测——没有 AV-align、没有 Synchformer 类的时序对齐打分作为过滤条件，也没有语义匹配过滤。即 3,074 小时通用数据 + 3,422 小时 VGGSound/AudioSet 完全不经过音视频对齐核验，仅依赖「原生同步音轨」这一天然假设来保证对齐。这是相当大的一处简化，也解释了为何模型在环境音的粗粒度协调上表现尚可、但缺少精确的事件级对齐能力。
【架构层的对齐保证】数据侧的对齐薄弱由架构与工程侧补偿：(1) 音频采样率从 44.1 kHz 下调至 25.6 kHz，使音频 latent 时间栅格与 25 fps 视频严格对齐；(2) 两塔在对应 block 层级深度融合 + 跨模态注意力，使对齐在特征层持续发生；(3) 在线标注保证文本条件与音视频窗口同源。
【评测侧】用 Synchformer 计算 AV-A（Audio-Video Alignment）时序同步分（越低越好，UniVerse-1 为 0.23），用 LSE-C 衡量唇同步（越高越好，1.34）。注意评测用的 Synchformer 并未被用作训练数据的过滤器。

### [Unison](../models/Unison.md)

音视频同步检测在 Unison 中是数据侧唯一被明确描述的过滤机制，但只覆盖唇同步、不覆盖通用事件对齐：
【唇同步检测——lip-filtering 算子，两步串联】
1) 人脸检测：检测画面中人脸的数量与位置，得到 bounding boxes。检测器型号未披露。同时获取「数量」这一信息，说明具备多人脸场景的识别能力；
2) 框内 SyncNet 核验：SyncNet「仅在这些检测框内」运行（applied exclusively within these bounding boxes）以验证对齐。这一「限定区域」设计是本工作在同步检测上最具工程价值的细节——在全帧上运行 SyncNet 时，若人脸在画面中占比小、或画面含多人，同步置信度会被大量无关像素稀释而失真；限定在人脸框内可显著提升判定信噪比。UniVerse-1 采用「RetinaFace 检测 → 全图 SyncNet」的松散串联，Unison 则把两者真正耦合起来。
【双重剔除目标】论文明确该算子「自然地过滤掉」两类样本：(i) 语音与唇动不同步的片段；(ii) 画外音/旁白（off-screen voice-overs）。第二类的显式声明值得强调——画外配音是纪录片、教程、vlog 类内容的常态，其音轨有语音而画面无对应说话人，若不剔除会教会模型「有语音时不必动嘴」，是唇同步学习中最具破坏性的一类噪声。Unison 是本次调研中少数把「剔除画外音」写为明确设计目标的工作。
【非语音事件对齐：无数据侧检测】论文未对 VGGSound 等非人脸数据做任何时序事件对齐核验——无 AV-align、无 Synchformer 类打分作为过滤条件、无起音（onset）对齐检测。即环境音与画面事件的对齐完全依赖「原生同步音轨」这一天然假设。
【对齐保障的重心在训练层而非数据层——这是 Unison 的核心方法论选择】数据侧只做唇过滤，事件级对齐的保障全部由三项机制承担：
1) 架构层：帧级双向交叉注意力，三帧窗口对齐、stride=1、仅保留中间帧表征——一个刻意设计的极短时窗局部对齐；
2) 训练层：双向跨模态 forcing，通过解耦去噪时间步让干净模态引导含噪模态，强制模型依赖跨模态信息而非各自的模态内先验；
3) 音频内部：语音流与音效流复用相同的 RoPE 时间索引，保证两条音频流之间严格时间对齐。
论文的立场很明确——对齐是「训练出来的」而非「过滤出来的」，这与 MOVA（Synchformer + ImageBind 双维度阈值过滤）的数据驱动路线形成路线之争。Table 2 的消融为这一立场提供了支撑：移除 CMFS 后 DS 从 0.08 恶化到 0.19（劣化 2.4 倍），是全部消融中最严重的单项退化，证明训练策略对对齐的贡献确实主导。
【评测侧同步指标】SyncNet 的 LSE-C（3.30）与 LSE-D（7.88）衡量唇同步；Synchformer 的 DeSync（DS）分数衡量模态起音的绝对时间偏移，Unison 的 0.08 为 TI2AV 全场最优（优于 LTX-2 的 0.10、MOVA 的 0.13），T2AV 设定下 0.06 同样最优。注意 Synchformer 仅用于评测，未用作数据过滤器——与 MOVA 把 Synchformer 用作过滤阈值的做法不同。

### [Veo 3 / Veo 3.1](../models/Veo_3_Veo_31.md) ⚠️

[不确定] 数据侧的音视频同步检测方法完全未披露。官方仅在架构层面解释同步来源：视频与音频 latent 在同一扩散过程中被联合去噪，同步性由架构保证而非后处理对齐。这意味着训练数据必须本身就是严格同步的原生配对音视频，因此必然存在某种同步质量筛选（唇形同步、事件对齐），但 Google 未披露任何检测手段。官方仅在效果层面提到 Veo 3.1 在音视频同步（audio-video synchronization）维度上表现最佳。

### [Vidu S1](../models/Vidu_S1.md)

音视频同步在数据流程中被放在两个位置：
(1) 预过滤级的粗筛：入 pipeline 前即按「音视频同步性（audio-visual synchronization）」这一指标剔除不同步的原始视频，与帧率、分辨率、音视频完整性并列为四项技术门槛；
(2) Diarization 级的精细对齐：论文强调「只有当模型在训练中观察到与对应语音一致的视觉表现，才能学到高保真的唇形同步」，因此用 VAD + ASD 定位每段语音的时间戳与说话人，并通过说话人与画面主体的匹配关系判定 onscreen/offscreen/overlap；offscreen（画外音，声画主体不一致）与 overlap（多人声重叠）均被视为破坏声画对应关系而处理——含 overlap 的 clip 直接整条剔除。此外镜头切分时切点避开语音中段，也是为保护语音-唇形段的完整性。
评测侧使用 Sync-D 指标（基于 SyncNet/Wav2Lip 的 lip-sync expert，引用文献[59] Prajwal 等 「A lip sync expert is all you need」）衡量音视频同步，Vidu S1 在 HDTF 上取得 7.8470（越低越好），优于 OmniAvatar 9.242、StableAvatar-1.3B 11.18、Hallo3 8.660、Wan2.2-S2V-14B 8.255、LiveAvatar 8.447、LemonSlice 7.921、HeyGen 8.037、Kling Avatar 2.0 8.158。

### [主流视频预训练数据集合并调研：Panda-70M、InternVid、Koala…](../models/pretraining_datasets.md)

不适用。七个数据集均无音频模态，不存在音视频同步检测环节，未使用任何口型同步或音画事件对齐方法。

## 同步检测具体指标与阈值（SyncNet/Synchformer/LSE/自研，阈值数值，如MOVA LSE-D≤9.5且LSE-C≥4.5、SkyReels-V4 SyncNet |offset|≤3∧conf>1.5）

`sync_metric_and_threshold` · 详细程度: detailed

### [Allegro](../models/Allegro.md)

不适用。未使用 SyncNet、AV-align 或任何自研同步指标，无相应阈值。

### [Apollo](../models/Apollo.md) ⚠️

【指标】明确：时序同步用 Synchformer 输出的对齐分/偏移量，语义同步用 ImageBind 跨模态相似度。
【阈值】完全未披露——论文只说「ensuring high synchronization in both temporal and semantic dimensions」（确保时序与语义两个维度均高度同步），没有给出 Synchformer 的偏移量上限（如 |offset| < 0.2s）或分数下限，也没有给出 ImageBind 相似度的阈值（如 IB-score > 0.2），更没有说明两个条件是与关系还是加权组合。
【对比参照】同期工作中 MOVA 公开了完整阈值表（Audiobox PQ>5.0/CU>4.5/CE>2.5、DOVER-Aesthetic>0.85、LSE-D≤9.5、LSE-C≥4.5 等），UniTalking 公开 SyncNet conf>0.9，Apollo 在这一维度的披露显著落后。
【评测侧数值（非过滤阈值，不可混用）】Table 3 给出模型输出的同步指标：DeSync 0.650（越低越好）、Sync-conf 6.787（越高越好）、IB 0.316（越高越好）。
[不确定]

### [CineDance / CineDance-1M](../models/CineDance.md) ⚠️

【指标选型】数据侧 SyncNet（唇同步）+ ImageBind（全局跨模态对齐）；评测侧 Sync-C、Sync-D、IB-Score。
【阈值】论文未公布任何同步指标的过滤阈值数值——这是该工作与 MOVA（LSE-D ≤ 9.5 且 LSE-C ≥ 4.5）、SkyReels-V4（SyncNet |offset| ≤ 3 且 conf > 1.5）等采用硬阈值的工作的根本方法论差异。CineDance 明确采取「全量打分 → 存为元数据 → 不硬剪枝」策略，原文表述为「we store all quality scores as metadata, enabling users to flexibly construct task-specific subsets」，将阈值决策权下放给下游使用者。
【取舍评价】优势是保留数据完整性与下游灵活性，避免过度过滤丢失长尾；劣势是数据集本身不保证同步质量下限，使用者必须自行设定阈值筛选，且论文未提供推荐阈值参考值。
【SyncNet 具体版本、ImageBind 版本、分数分布区间】均未披露。[不确定]

### [CogVideoX](../models/CogVideoX.md) ⚠️

[不确定] 无任何公开的同步检测指标或阈值。CogVideoX 视频侧不适用；CogSound 无技术报告，未公开 SyncNet / AV-align / ImageBind score 等指标的使用情况，也无置信度阈值数值。可确认的是全流程未使用 SyncNet 类唇同步指标（模型不生成语音）。

### [Cosmos-Predict2.5](../models/Cosmos-Predict25.md)

不适用。无 SyncNet / Synchformer / LSE-C / LSE-D 等任何音视频同步指标与阈值。作为对照，该工作中有明确数值阈值的数据准入规则集中在 Human Dynamics 域：人物需出现在 >40% 的帧中、任一帧可见人数 ≤8 人、至少一人占画面面积 ≥3%；以及时长阈值（丢弃 <5 秒、保留 5–60 秒）与端到端 4% 的保留率。

### [Data-Juicer 2.0](../models/Data-Juicer.md) ⚠️

[不确定] 由于不提供同步检测算子，Data-Juicer 也不存在任何同步指标与阈值的推荐值。
【DJ 公开的唯一跨模态阈值——供参照】video_frames_text_similarity_filter 在官方 T2V 最优数据池中的最小阈值为 0.306337（CLIP 视觉-文本余弦相似度）。这个数值的产生方式值得注意：它不是人工设定的整数（如常见的0.3），而是 Probe-Analyze-Refine 流程中按数据池分位切分自然落到的边界值——即「保留相似度最高的那一部分样本」时该分位点对应的具体数值。这种「阈值由目标保留比例反推」的做法与模型团队常见的「人工经验定阈值」形成对比。
【其他算子的阈值状况】DJ 全部算子的阈值均为 YAML 外置参数，官方 config_all.yaml 中给出的是占位默认值而非推荐值。文档明确不提供跨场景通用的推荐阈值，其立场是阈值应通过 Sandbox 在具体数据集与具体下游模型上搜索确定。
【方法论价值】对本调研而言，DJ 提供的不是「阈值数值」而是「确定阈值的方法」：把数据按某统计量三等分 → 各训一个参考模型 → 用统一基准评测 → 选出增益最大的分段。这套流程理论上可直接套用于同步指标（如把 SyncNet confidence 三等分做探测），只是 DJ 官方尚未在音视频同步维度做过此类实验。

### [Foley-Omni](../models/Foley-Omni.md) ⚠️

【清洗端具体数值】
  · Synchformer sync score ≥ 0.2 —— 时序同步可靠性门槛
  · ImageBind (IB) score ≥ 0.3 —— 音视频语义一致性门槛
  两项并列于 Table 7 的 Alignment 维度，均为独立硬阈值，通过条件为「同时满足」。
【评测端指标】
  · DeSync（越低越好）—— 时序对齐误差。Foley-Omni 0.16 vs Ground Truth 0.14 vs 级联基线 0.85/0.26。
  · IB score（越高越好）—— 音视频语义一致性。Foley-Omni 0.26 vs Ground Truth 0.36 vs 基线 0.25/0.16。注意生成结果的 IB 0.26 与清洗阈值 0.3 处于同一量级，说明 0.3 这个门槛在真实数据上已属中等偏严。
  · CLAP（越高越好）—— 音频与文本描述的语义相关性。Foley-Omni 0.27 vs GT 0.30。
  · WER（越低越好）—— 语音可懂度。V2ST-Bench 上 Foley-Omni 7.59，甚至低于 Ground Truth 的 8.03（因生成语音比真实录音更清晰、无环境干扰）；GRID 子集 15.3。
  · 主观 MOS 三项：A-MOS 3.92（音质）、S-MOS 4.13（语义一致）、T-MOS 4.14（时序同步），对照 GT 的 4.33/4.37/4.42，差距最小的是 S-MOS 与 T-MOS。
【阈值宽严评估】Synchformer 0.2 与 ImageBind 0.3 相比业界同类（如 UniVerse-1 的 SyncNet 严格阈值、MMAudio 系工作的对齐筛选）属于中等偏宽松的设定，倾向于保留数据量。论文未提供阈值敏感性分析。
[不确定] 阈值的确定依据未说明（是否经过网格搜索、是否有保留率-性能权衡曲线均无报告）。Synchformer/ImageBind 分数的具体计算配置（窗口长度、特征聚合方式）也未交代，影响复现精度。

### [Goku](../models/Goku.md)

不适用。无音视频同步指标与阈值。可对照记录的是其视觉侧一致性阈值体系（作为「阈值设定风格」的参考）：DINOv2 帧间余弦相似度 ≥0.85（480×864）/ ≥0.90（720×1280）；美学分 ≥4.3（480p）/ ≥4.5（720p+）；OCR 文字面积 ≤0.02（480p）/ ≤0.01（720p+）；RAFT 运动分数 [0.3, 20.0]（480p）/ [0.5, 15.0]（720p）/ [0.5, 8.0]（1080p）。这些阈值全部按分辨率分档、随清晰度提升而收紧，是 Goku 阈值设计的统一哲学。

### [HunyuanVideo-Foley](../models/HunyuanVideo-Foley.md) ⚠️

【指标明确、阈值全缺】这是本字段的核心结论。论文明确点名了两个检测工具（ImageBind 做语义、AV-align 做时序），但没有给出任何一个的过滤阈值数值——既没有 ImageBind 相似度的下限，也没有 AV-align 分数的门槛，也未说明两者是「与」关系（须同时通过）还是加权组合或分级处理。[不确定]
【与同类工作的披露对比】UniVerse-1 给出 SyncNet conf > 2.0；MOVA 给出 LSE-D ≤ 9.5 且 LSE-C ≥ 4.5，并对 SynchFormer 与 ImageBind 分别设阈。HunyuanVideo-Foley 在工具选型上与 MOVA 高度相似（同样是时序 + 语义双工具），但阈值披露程度显著更低——工具名有、数值无。
【任务差异导致的指标体系差异】本工作不涉及唇同步，因此完全不使用 SyncNet / LSE-C / LSE-D 这套人脸-语音同步指标体系，转而使用面向通用声学事件的 AV-align 与 Synchformer/DeSync。这两套指标体系的适用对象不同、数值不可比：LSE 系列衡量的是口型与音素的对应，AV-align/DeSync 衡量的是任意视觉事件与声学事件的时间对应，后者的判定难度更高（因为「什么算一个视觉事件」本身是模糊的）。
【评测阈值 vs 过滤阈值】评测侧的 DeSync 数值（0.53~0.74，单位为秒的错位估计）是模型输出的实测结果，不是数据过滤门槛，两者不可混用。
【阈值标定方法未说明】论文未说明任何阈值是经人工抽检标定、消融确定还是经验设定。[不确定]

### [HunyuanVideo](../models/HunyuanVideo.md) ⚠️

不适用。未使用 SyncNet、AV-align 或任何同步指标，无阈值。[不确定]

### [InstructAV2AV](../models/InstructAV2AV.md) ⚠️

【数据构造侧：无数值阈值】
  [不确定] 这是本工作与 MOVA（LSE-D≤9.5 且 LSE-C≥4.5）、SkyReels-V4（SyncNet |offset|≤3 且 conf>1.5）、Foley-Omni（Synchformer≥0.2、ImageBind≥0.3）等工作最明显的差异——InstructAV2AV 在数据构造阶段完全没有设置同步分数的数值阈值。其同步质量由两个非阈值机制保障：(1) TalkNet+Scribe 的「语音须与画内可见说话人时间对齐」这一布尔型准入条件；(2) 合成阶段 frame-wise cross-attention 的架构级强制。这是「用机制替代阈值」的思路，好处是不需要标定阈值、不会因阈值不当误杀或漏放，坏处是数据集的同步质量无法被量化描述，也无法与其他数据集横向比较。
  唯一有确切数值的数据侧阈值是音频静音的 -45 dBFS（PyDub），但那不是同步指标。其余阈值（CoTracker3 运动幅度、LAION 美学、Audiobox-Aesthetics、MLLM 五维度打分门槛）全部未给出数值。

【评测侧：指标与配置明确】
  · Sync-C（S-C，越高越好）—— SyncNet 唇同步置信度。
  · Sync-D（S-D，越低越好）—— SyncNet 同步距离/偏移。二者配置明确给出 vshift=15 帧（即在 ±15 帧窗口内搜索最佳对齐位置，这是 SyncNet 的标准设定）。
  · PEAVS（越高越好）—— 感知级音视频同步评估指标，相比 SyncNet 不局限于唇部，可覆盖通用音视频事件同步，对本工作的非语音编辑任务（实例插入/移除）是必要补充。
  · AV-A（越高越好）—— 基于 ImageBind 的音视频语义对齐分数。InsAVE-80K 评测集上 InstructAV2AV 27.72 vs AvED 26.44 / AVI-Edit 26.37 / CoherentAVEdit 22.67；AvED-Bench 上 23.71 vs AVI-Edit 23.21。
  四项指标覆盖了「唇同步精度（Sync-C/D）」「通用事件同步感知（PEAVS）」「跨模态语义匹配（AV-A）」三个层次，划分较完整。
  [不确定] Sync-C / Sync-D / PEAVS 的具体数值未在可获取的表格内容中完整呈现，仅 AV-A 有确切数字。

【方法论差距】数据侧无阈值、评测侧有指标，意味着「数据的同步质量」与「模型的同步能力」之间缺乏可对照的同一把尺子。若在数据构造阶段也用 Sync-C/Sync-D 度量并报告数据集的同步分数分布，就能回答「模型的同步能力是否已经逼近数据上限」这一关键问题，可惜未做。

### [2026 其他音视频联合生成](../models/JAVG_2026_misc.md) ⚠️

【完全公开阈值的仅一项】
OmniCustom：SyncNet「|offset|≤3」且「confidence>1.5」（与 Ovi 完全一致，直接继承）；配合美学分 <0.3 剔除、时长 <10 秒剔除。三条规则为「与」关系。

【ALIVE 的相关阈值（非同步阈值但同属数据筛选阈值，本批唯一公开的一组数值）】
Character-driven cross-pair 的有效配对判定：
- face similarity（ArcFace 人脸相似度）> 0.35
- CLIP similarity（CLIP 视觉相似度）> 0.7
- absolute proximity to 0.9
身份锚点选取：sync score 最大的 1.5 秒子片段。
SubjectID correction：过滤未达「high-confidence thresholds」的视频，具体数值未公开[不确定]。
其他：训练时 cross-attention signal dropout 概率 0.3；推理时 FAISS 检索相似度阈值 τ>0.85。
TS-TalkNet 的同步分阈值未公开[不确定]。

【NAVA】检测器名单公开（SyncNet、SyncFormer、ImageBind）但「does not provide exact numerical thresholds」——全部阈值未公开[不确定]。同样地，视觉质量算子（aesthetics、sharpness、brightness、motion score）与 AudioBox Aesthetic 的阈值也全部未公开[不确定]。这是一篇数据流程描述完整但数值全部隐去的典型工业论文。

【StreamChar / CCL / Baton】数据侧无同步阈值[不确定]。

【评测侧指标（非过滤阈值，但反映各工作对同步的度量口径）】
- CCL：Table 2 报告 WER、Sync-C、Sync-D、DeSync、IB 五项指标做消融——Sync-C/Sync-D 为 SyncNet 系置信度与距离，DeSync 为 Synchformer 系去同步度，IB 为 ImageBind 分数。五指标并用说明其对同步的度量比多数工作更细。
- NAVA：在 Verse-Bench 上刷新 Sync-C / Sync-D / 视频质量 / 音频 WER 的 SOTA，且参数量比开源基线少 2–5 倍。
- ALIVE：Alive-Bench 1.0 含 22 项细粒度指标（六大类），同步应为其中之一[细节不确定]。
- ITS-JAVG：JavisScore（细粒度同步）、AVHScore（音视事件语义一致）、ImageBind（音视语义相似）三个与同步相关的验证器；ARW 归一化公式 R(i)=Σ_k w_k·r_k(i)/(σ_k+ε)。

【横向观察】2026 年该领域的阈值披露呈两极：学术/中小团队（OmniCustom）完整公开且直接沿用 Ovi 标准；工业团队（NAVA、ALIVE）流程公开、数值隐去。SyncNet 的 |offset|≤3 / conf>1.5 已固化为社区默认配置。

### [音视频联合生成基线合集](../models/JavisDiT_baselines.md) ⚠️

本合集普遍报告同步指标但极少给出数据过滤阈值——这是与 Ovi（|offset|≤3 且 conf>1.5）、MOVA（LSE-D≤9.5 且 LSE-C≥4.5）、SkyReels-V4 等工业模型最大的差异：
【MM-Diffusion / AV-DiT】无同步指标、无阈值。评测只有 FVD（用 i3d 模型计算）与 FAD（用 AudioCLIP 计算），加人工图灵测试（1 万票，>80% 骗过人类）。
【JavisDiT —— 提出 JavisScore（本合集唯一的新指标）】
- 具体参数完整公开：窗口 2 秒、重叠 1.5 秒（滑窗步长 0.5 秒）、底层用 ImageBind 计算音视频相似度、在每个窗口内取「同步性最低的 40% 帧」聚合。
- 验证集：3,000 条人工标注样本。
- 这些是指标计算参数而非数据过滤阈值——JavisDiT 不用同步分数过滤训练数据[不确定]。
【JavisDiT++ —— 11 维评测体系】完整指标清单：质量维度 FVD、FAD；文本一致性维度 TV-IB（text-video ImageBind）、TA-IB（text-audio ImageBind）、CLIP-Score、CLAP-Score；音视频一致性维度 AV-IB、AVHScore、JavisScore；同步性维度 DeSync（基于 Synchformer）。AV-DPO 中用 Synchformer 作为同步奖励模型，但打分阈值/排序细节未公开[不确定]。全部评测在 240P、4 秒配置下进行。
【Harmony —— 四项同步指标】Sync-C 与 Sync-D（SyncNet 系的唇同步置信度与距离）、DeSync Score（基于 Synchformer）、ImageBind（IB）Score。消融给出的关键数值链：Sync-C 基线 4.29 →（加 RoPE 位置对齐）4.80 →（加 Cross-Task Synergy 训练）5.09 →（加 Synchronization-Enhanced CFG）6.51。数据过滤所用的「音视频一致性打分模型」的名称与阈值未披露[不确定]，这是 Harmony 数据侧最关键的缺失信息。
【UniAVGen】评测用 SyncNet（同步）、VBench（视频质量）、AudioBox-Aesthetics（音频质量）、Whisper WER（语音可懂度）；数据侧同步过滤阈值未提及[不确定]。
【横向意义】本合集提供了同步性「评测指标」的丰富选择（JavisScore / DeSync / Sync-C/D / AV-IB / AVHScore），但几乎不提供同步性「数据过滤阈值」，说明学术基线更关注如何度量同步，工业模型更关注如何用阈值筛出干净数据——两者形成互补。

### [可灵 3.0 / 可灵视频 3.0 Omni](../models/Kling_30_Omni.md) ⚠️

[不确定] 未公布任何同步指标名称与阈值数值（如 SyncNet confidence / LSE-C / LSE-D / AV-align 的具体门限）。公开材料中仅有的量化音频门限来自 Kling-Foley 的数据规范：VAD 静音占比 < 0.2；评测有效样本要求视频≥5秒、音效≥2秒。团队在 Kling-Foley 论文中报告了分布匹配、语义对齐、时序对齐、音频质量四大类SOTA指标，但那是模型评测指标而非数据过滤阈值。

### [LTX-2](../models/LTX-2.md) ⚠️

完全未披露。论文未使用 SyncNet、AV-align 或任何自研同步指标，未给出任何置信度阈值数值（无法与 UniTalking SyncNet conf>0.9 之类对比），也未报告 LTX-2 在任何客观音视频同步基准上的分数。全部音视频质量结论均来自内部人类偏好研究（对比 Ovi、Veo 3、Sora 2），且论文未给出人评的具体分数表，仅定性描述「显著优于 Ovi」「与领先闭源模型相当」。唯一的量化表格是推理速度对比（H100 上 121帧 720p、单步 Euler、CFG=1：LTX-2 19B 音视频 1.22秒/步 vs Wan 2.2-14B 纯视频 22.30秒/步，约18倍加速）。[不确定]

### [LongCat-Video](../models/LongCat-Video.md) ⚠️

[不确定]。Avatar 1.5 报告仅定性表述为移除「音视频偏移过大」的样本，未公布所用的同步度量指标（未提及 SyncNet conf / sync-c / sync-d / AV-align 等具体指标名），也未给出任何阈值数值。使用的 TalkNet、UniTalk 为主动说话人检测模型（输出说话/不说话及置信度），其判定阈值同样未披露。全流程中唯一公布了确切阈值的是情绪标注环节的 EmotiEffLib 置信度 s>0.7，与音视频同步无关。

### [MOVA](../models/MOVA.md)

全部阈值公开（附录 Table 9 与 4.3 节），是本次调研中同步过滤阈值披露最完整的样本之一：
【Stage 2 音视频对齐（宽松 OR 门）】
- ImageBind Score（语义对齐）≥ 0.2
- **OR** SynchFormer Offset（DeSync，时序偏移）≤ 0.5
注意这是逻辑“或”而非“与”，论文明确解释设计意图：“This ensures that both semantically relevant ambient sounds and temporally synchronized speech/actions are preserved.”（既保住语义相关的环境音，也保住时序同步的语音/动作）——因为环境音天然缺乏尖锐的时序 onset、难以通过时序同步检测，而快速动作音效可能语义分不高，用 AND 会两头误杀。
【Phase 2 唇同步专项阈值】
- LSE-D ≤ 9.5（距离越小越好）
- LSE-C ≥ 4.5（置信度越大越好）
两条同时满足，得到约 2.5M clips 的高质量唇音对应子集。相比 UniTalking 的 SyncNet conf > 0.9，MOVA 的 LSE-C ≥ 4.5 是在 SyncNet 原始置信度尺度上的取值，属中等偏严的门槛。
【Stage 2 音频质量相关阈值】静音占比 < 0.8、带宽 > 1,000 Hz、Audiobox PQ > 5.0 / CU > 4.5 / CE > 2.5。
【Stage 2 视频质量阈值】DOVER-Aesthetic > 0.85、DOVER-Technical > 0.05；Phase 2 收紧至 DOVER-Technical > 0.15；Phase 3（720p）为 DOVER-Technical > 0.14。
【评测侧同一套指标复用】DeSync（SynchFormer）、IB-Score（ImageBind）、LSE-D/LSE-C（SyncNet）在评测中同样使用，训练过滤与评测指标高度同源。MOVA-360p + dual CFG (s_B=3.5) 达 DeSync 0.351 / IB-Score 0.315 / LSE-D 7.004 / LSE-C 7.800。

### [合并条目：① Mochi 1](../models/Mochi_MAGI_Motif.md)

不适用。三者均未使用 SyncNet、AV-align 或任何自研音画同步指标，无相应阈值。

### [Movie Gen](../models/Movie_Gen.md)

使用自研 CAVTP 的音-视 embedding 余弦相似度作为唯一的量化对齐指标，阈值按音频类型分档给出（表24，均由人工检视 manual inspection 确定）：
· AED = sound / voice / voice+sound → 余弦相似度 > 0.2 判为 diegetic
· AED = music / music+sound / voice+music+sound → 余弦相似度 > 0.3 判为 diegetic（音乐类要求更高，因音乐与画面的天然相关性更弱、易误判）
· AED = music 且 相似度 < 0.1 → 判为 non-diegetic（背景配乐）
· AED = sound+music 且 0.1 < 相似度 < 0.25 → 判为 non-diegetic
· AED = sound+voice+music 且 0.1 < 相似度 < 0.25 → 判为 mixed（画内画外混合）
预训练只保留 diegetic 与 mixed 桶，外加少量 non-diegetic 背景音乐。
未使用 SyncNet、AV-align 等公开同步指标；评测阶段客观指标用 ImageBind score（音视频对齐）与 CLAP score（音文对齐），主观指标才细分为同步性（Sync.）与正确性（Corr.）。
其他相关阈值：PT2V 用 ArcFace 余弦相似度 > 0.5（相邻帧同一人）与 > 0.7（合成参考图身份保持）。

### [NVIDIA NeMo Curator](../models/NeMo_Curator.md)

无。不存在任何同步性指标或阈值（无 SyncNet 置信度/offset、无 LSE-D/LSE-C、无自研同步分），也无相关配置项。可作为对照参考的是该框架已有的、参数完全透明的其他阈值体系：美学 score_threshold = 3.5、运动 motion_score_global_mean = 0.00098、motion_score_per_patch_min_256 = 0.000001、DOVER 剔除底部 15%、clip 时长 2–60 秒、语义去重 k-means k = 10000。这套「默认阈值全部写进文档」的做法本身值得借鉴，但其中不含任何音视频维度。

### [OmniHuman 数据集 + OHBench](../models/OmniHuman.md) ⚠️

【数据侧使用的指标】SyncNet 响应分数 S_sync + 时间偏移量，两个量联合判定。
【已披露的数值】
- 时间偏移容差：±3 帧（在统一的 30 FPS 下即 ±100 毫秒）——这是同步维度唯一的明确数值；
- 人脸嵌入相似度阈值：0.55（ArcFace 余弦相似度，用于身份指派，与同步判定配合完成归属）；
- 跟踪丢失容忍跨度：5 帧。
【未披露的关键数值】S_sync 的阈值本身，论文仅表述为「above a preset threshold」（高于一个预设阈值）——即最核心的同步分数门槛是空白的。这与 MOVA（LSE-D ≤ 9.5 且 LSE-C ≥ 4.5）、SkyReels-V4（|offset| ≤ 3 且 conf > 1.5）等明确给出双阈值的工作相比，披露粒度明显不足。值得注意的是，OmniHuman 给出了偏移阈值（3 帧）却隐去了置信度阈值，恰好是把两个必要条件披露了一半。[不确定]
【判定逻辑的严格性】「所有被检测到的主体均需满足」（all detected subjects satisfy）——全员制而非任一制，这是比阈值数值本身更重要的一条信息，意味着双人场景中只要有一人的语音归属不确定，整条样本即被丢弃。
【SyncNet 变体未说明】未指明使用的是原始 SyncNet、SyncNet-python 的哪个实现、还是重训版本；也未说明 S_sync 是 confidence 还是 distance 口径。[不确定]
【评测侧的同步指标与阈值】OHBench 中：
- Lip-Sync（个体层）：SyncNet，衡量生成视频的唇音对齐准确度，未给出合格阈值（作为连续分数报告）；
- V-A / Audio-Visual Sync（全局层）：ImageBind，衡量视觉与听觉事件之间的时序偏移，作为连续分数报告。
【与训练数据阈值的一致性问题】数据过滤用 SyncNet + ±3 帧，评测也用 SyncNet——同一模型既做数据筛选又做效果评判，存在轻微的循环论证风险（用 SyncNet 筛出的数据训练的模型，在 SyncNet 指标上自然占优）。论文未讨论这一点。[不确定]

### [Open-Sora 系列](../models/Open-Sora.md)

不适用。未使用 SyncNet、AV-align 或任何同步指标。

### [Ovi](../models/Ovi.md) ⚠️

指标与阈值全部明确公开，是本工作数据侧披露最具体的部分：
- 检测器：SyncNet（Chung & Zisserman, 2016），经内部改造以支持百万级视频规模批处理。
- 指标 1：offset（音视频时间偏移，单位为帧）。阈值：|offset| ≤ 3。按 24fps 计算即容许误差 ≤ 125 毫秒。
- 指标 2：confidence（SyncNet 置信度）。阈值：confidence > 1.5。
- 指标 3（音频侧联合门槛）：mean volume ≥ −60 dB（平均音量不低于 −60 分贝），用于排除近似静音/音轨极弱的片段。
- 组合逻辑：三者为「与」关系，须同时满足才保留。
【横向对照】该 confidence>1.5 门槛属于中等偏严（SyncNet conf 常见取值范围约 0~10+，部分工作如 UniTalking 用 >0.9，talking-head 数据集常用 >3），而 |offset|≤3 帧属于较严格的时间容差。作者自评为 strict criteria。
【未公开】通过率、各阈值的消融曲线、阈值选取的实验依据数值均未给出[不确定]。

### [Script-a-Video](../models/Script-a-Video.md) ⚠️

【数据过滤阈值：无】论文未给出任何用于数据过滤的同步指标阈值——没有 SyncNet 置信度门槛、没有 AV-align 分数门槛、没有唇动检测门槛。这与 UniVerse-1（SyncNet conf > 2.0）、UniTalking（SyncNet conf > 0.9）等明确给出数值门槛的工作形成对比。本工作的对齐保证来自 schema 规则而非阈值过滤。[不确定]
【评测指标：SyncNet（chung2016out，Out of Time 原始版本）】输出同步置信度分数与帧级时间偏移。实测数值（多镜头场景）：
- LTX-2-AV（单体式提示基线）：6.86
- LTX-2-AV-MTSS（仅换提示词）：13.86
- Ours w/o ID：9.16
- Ours(Full)：9.72
【数值解读的关键警示】论文明确指出该自动指标具有欺骗性（"the automated A-V Sync metric can be deceptive"）：基线 6.86 看似最优，但其人评 A-V 分仅 1.18（1-3 分制最低档），原因是基线生成的是平坦的环境噪声——让近乎静止的音频包络与画面「同步」是平凡的（信息稀疏性伪影）。换用 MTSS 后模型开始尝试生成复杂对白，SyncNet 分数先升至 13.86（偏差扩大），完整方案再通过架构改进把时序缺口收窄到 9.72，同时人评 A-V 达 2.26（全表最优）。
这一发现对数据侧有直接的实践意义：若用 SyncNet 分数作为数据过滤闸门，会系统性地偏好无语音/弱音频内容的样本，反而滤掉信息丰富的对白样本。任何采用 SyncNet 阈值过滤的 pipeline 都应先做音频活跃度分层，再分层设阈值。
【其他相关指标与数值】
- Shot Boundary Deviation（帧）：LTX-2-AV 3.79 → LTX-2-AV-MTSS 3.27 → Ours w/o AV 1.28 → Ours w/o ID 0.38 → Ours(Full) 1.36；
- WER：LTX-2-AV 多镜头 0.84 → LTX-2-AV-MTSS 0.13 → Ours(Full) 0.19（单镜头基线 1.64 → 0.78 → 0.23）；
- Audio Quality（UTMOS）：4.12-4.18 → 4.60-4.79 → 4.68；
- Reference ID Similarity（ArcFace 余弦相似度）：单镜头 Ours w/o MS 0.62，多镜头 Ours w/o ID 无值、Ours(Full) 0.22、Ours w/o AV 0.20；
- Intra-Shot Subject Consistency（DINOv2 [CLS] 帧间平均余弦相似度）：LTX-2-AV 0.87（虚高）→ LTX-2-AV-MTSS 0.66 → Ours(Full) 0.59。

### [Seedance 2.0 与 Seedance 1.5 pro](../models/Seedance_20_Seedance_15_pro.md) ⚠️

[不确定] 完全未披露。报告中未出现 SyncNet、AV-align、LSE-C/LSE-D 等任何自动同步指标名称，也未给出任何过滤阈值数值。Seedance 系列的音画同步评估全部采用 1–5 分的人工/专家主观评分（SeedVideoBench），而非自动化客观指标——报告在 Arena 章节甚至专门指出其评估路线不依赖 FVD、CLIPScore 这类自动指标。因此无法与 UniTalking「SyncNet conf > 0.9」这类披露做对标。

### [SkyReels 系列](../models/SkyReels.md)

SkyReels-V4 给出了明确的阈值数值，在同类模型中属于披露较充分的一档：
(1) 同步准入条件：|offset| ≤ 3 ∧ confidence > 1.5 —— 即 SyncNet 估计的音视频时间偏移绝对值不超过3帧，且同步置信度大于1.5，两个条件需同时满足（合取）才保留该片段；
(2) 音量门限：要求平均音量不低于 -60 分贝（minimum mean volume of -60 dB），用于排除近乎无声、SyncNet 结果不可信的片段；
(3) 静音门限：VAD（语音活动检测）要求静音占比低于 0.2（silence ratio below 0.2）。
【对比参照】该阈值组合（offset≤3帧、conf>1.5）比 UniTalking 的 SyncNet conf>0.9 更严格于 confidence 维度而增加了 offset 维度约束，属于「偏移+置信度」双条件写法。
【未披露】SyncNet 的具体权重版本（原版 SyncNet 还是自训练版）、在 32FPS 下3帧偏移对应的绝对时间容差（约94ms）是否为设计意图、以及该过滤剔除了多大比例样本。SkyReels-V2 无此环节。

### [Sora 2](../models/Sora_2.md) ⚠️

完全未披露。无 SyncNet、AV-align 或任何自研同步指标的名称，无任何置信度阈值数值（无法与 UniTalking 的 SyncNet conf>0.9 之类做对比）。OpenAI 亦未公布 Sora 2 在任何第三方AV同步评测基准上的分数。[不确定]

### [Step-Video-T2V](../models/Step-Video-T2V.md) ⚠️

不适用。未使用任何音视频同步指标，无相应阈值。可作为对照说明的是，本条目在「跨模态对齐」上的唯一手段是视觉-文本对齐：从片段中均匀抽取 8 帧，分别计算帧 embedding 与 caption 文本 embedding 的余弦相似度并取平均，得到 CLIP Score 作为图文对齐度标签，用于过滤错配样本——但该阈值的具体数值同样未公布。[不确定]

### [UniTalking](../models/UniTalking.md) ⚠️

【关于「SyncNet 置信度 > 0.9」的核实结论（重要）】本次调研对 arXiv:2603.01418v1 的 HTML 全文与 PDF 全文（10 页）执行了全文关键词检索，结论是：论文中不存在任何同步检测的数值阈值，不存在「SyncNet」一词，也不存在「0.9」这一阈值数字（全文出现的 0.9 仅为 AdamW 优化器的 β₁=0.9）。唇同步过滤的完整原文表述仅为「samples exhibiting poor lip-synced alignment (using LipSync)」，既未指明 LipSync 的具体模型实现，也未给出置信度或距离阈值。正文第 4.4 节虽提及「Further details are provided in the Appendix」，但 arXiv v1 版 PDF 实际未附任何附录内容。因此「SyncNet conf > 0.9」这一说法无法在一手文献中得到确认，可能源于二手转述、其他版本，或与本论文无关的信息串扰。[不确定]
【论文实际给出的同步相关信息】
- 过滤工具：LightASD（声源同源性）+ LipSync（唇音时序对齐），均无阈值；
- 评测指标：Sync-C（越高越好）与 Sync-D（越低越好），即 SyncNet 系的置信度与距离指标。
【评测结果（Table 1，T2AV 任务唇同步）】
- UniVerse-1：Sync-C 1.85 / Sync-D 11.97
- OVI：Sync-C 6.56 / Sync-D 8.6
- Sora2：Sync-C 5.35 / Sync-D 7.78
- UniTalking：Sync-C 4.87 / Sync-D 8.05
即 UniTalking 相对 UniVerse-1 在 Sync-C 上领先 3.02 分、Sync-D 上领先 3.92；相对 OVI 在 Sync-D 上领先 0.55 但 Sync-C 落后。
【论文对 Sync-C 指标本身的质疑】作者明确指出 OVI 异常高的 Sync-C（6.56，甚至高于 Sora2 的 5.35）可能反映的是指标偏差而非真实同步质量——推测 Sync-C 偏好「夸张的口型运动」而非「自然的发音articulation」。这是一个有价值的指标批判：它意味着单纯以 SyncNet 置信度作为过滤阈值也会引入同样的偏好，可能系统性淘汰口型自然含蓄的样本、保留口型夸张的样本。UniTalking 未给出阈值，故无从判断其是否受此影响。
【横向对照】UniVerse-1 使用 SyncNet conf > 2.0（相对宽松）；MOVA 使用 LSE-D ≤ 9.5 且 LSE-C ≥ 4.5 的双向约束（较严）。UniTalking 完全不披露阈值，在这一维度上的透明度是三者中最低的。

### [UniVerse-1](../models/UniVerse-1.md) ⚠️

【唯一的硬阈值】SyncNet confidence score > 2.0，作用于经 RetinaFace 确认含人脸的语音片段，是训练数据中唯一的量化同步准入门槛。
【阈值宽严评估】2.0 属于相当宽松的档位。作为横向参照：MOVA 采用 LSE-D ≤ 9.5 且 LSE-C ≥ 4.5 的双向约束（LSE-C 与 SyncNet conf 同源，其阈值 4.5 明显严于 2.0）；调研中提到的 UniTalking 使用 SyncNet conf > 0.9（口径可能不同）。UniVerse-1 只用单边 conf 阈值、不配 LSE-D 距离约束，且门槛设在 2.0，说明其在「保住数据量」与「保证同步质量」之间偏向了前者——考虑到语音子集最终仅得 1,187 小时，若阈值再提高数据量将难以支撑训练。
【阈值标定方法】论文未说明 2.0 是如何确定的，既无人工抽检说明，也无阈值消融实验。[不确定]
【非语音数据无同步阈值】3,074 小时通用数据与 3,422 小时 VGGSound/AudioSet 均无任何同步阈值约束。
【评测指标与训练过滤指标的错位】评测用 LSE-C（1.34）与 Synchformer AV-A（0.23），而训练过滤只用 SyncNet conf——LSE-C 1.34 的绝对值偏低（MOVA-720p 达到 6.593），部分可归因于训练侧同步门槛宽松，部分归因于基座仅 Wan2.1-1.3B。

### [Unison](../models/Unison.md) ⚠️

【使用的指标明确，阈值全部缺失】数据过滤环节使用 SyncNet 在人脸框内计算唇音对齐，但论文未给出任何阈值数值——既无 confidence 下界（对比 UniVerse-1 的 conf > 2.0、UniTalking 的 conf > 0.9），也无 LSE-D 距离上界（对比 MOVA 的 LSE-D ≤ 9.5 且 LSE-C ≥ 4.5 双向约束），甚至未说明使用的是 confidence 还是 distance 口径，也未说明 SyncNet 的具体实现版本（原始 Chung & Zisserman 版本 vs Wav2Lip 团队的 LSE 变体）。论文在方法节引用 SyncNet 时用的是 [Chung16a]（原始 SyncNet 论文），而在评测节引用 LSE-C/LSE-D 时用的是 [Prajwal_2020]（Wav2Lip），暗示训练过滤与评测可能使用了不同的实现，但论文未澄清。[不确定]
【阈值标定方法】未说明。既无人工抽检说明，也无阈值消融实验。[不确定]
【非语音数据无同步阈值】VGGSound 等非人脸数据不经任何时序对齐阈值筛选。
【评测指标的完整清单与数值（TI2AV 设定）】
- LSE-C（越高越好）：3.30，次于 LTX-2 的 3.45，优于 MOVA 3.24、UniAVGen 2.89、Ovi 2.81、UniVerse-1 2.32；
- LSE-D（越低越好）：7.88，次于 LTX-2 的 7.62，优于 MOVA 7.92、UniAVGen 9.49、Ovi 9.12；
- DeSync / DS（越低越好，Synchformer 测模态起音的绝对时间偏移）：0.08，全场最优，优于 LTX-2 0.10、Ovi 0.12、MOVA 0.13、UniAVGen 0.15、UniVerse-1 0.50。
【一个值得注意的指标分化】Unison 在 DS（通用事件级时序对齐）上夺冠，但在 LSE-C/LSE-D（唇同步专项）上略逊 LTX-2；用户研究也呈现同一模式（唇同步 1.86 次于 LTX-2 的 1.74，但运动-音频对齐 1.92 与语音-音效谐和 1.55 均夺冠）。这一分化相当自洽地反映了两者的路线差异：LTX-2 以 19B 视频骨干与超大规模数据取胜于唇部细节，Unison 以 5B 骨干（小近 4 倍）+ 跨模态 forcing 取胜于全局事件对齐与声学层次。论文本身也强调「以近 1/4 的视频骨干参数量取得更优的跨模态同步」。
【阈值缺失的实际影响】由于数据过滤阈值完全未知，Unison 的唇同步数据质量无法与其他工作横向比较，也无法判断其 LSE-C 略逊于 LTX-2 是源于过滤宽松还是骨干规模——这是本工作可复现性上的一处实质缺口。

### [Veo 3 / Veo 3.1](../models/Veo_3_Veo_31.md) ⚠️

[不确定] 完全未披露。未提及 SyncNet、AV-align 或任何自研同步指标，更无置信度阈值数值（对比 UniTalking 公开的 SyncNet conf>0.9 之类）。

### [Vidu S1](../models/Vidu_S1.md) ⚠️

数据过滤阶段：论文只说明按「音视频同步性」预过滤，未给出所用同步检测模型（是否 SyncNet）与任何阈值数值 [不确定]。启发式规则中的「语音能量占比过低」同样未给出具体阈值 [不确定]。
评测阶段：明确使用 Sync-D（SyncNet 系 lip-sync expert 距离指标），Vidu S1 = 7.8470（HDTF，全场最优）；同期报告 CSIM = 0.9192（身份保持，最优）、DOVER = 0.5660（感知质量，最优）。

### [主流视频预训练数据集合并调研：Panda-70M、InternVid、Koala…](../models/pretraining_datasets.md)

不适用。七者均未使用SyncNet、Synchformer、LSE-C/LSE-D、AV-align或任何音视频同步指标与阈值。

## 时序同步vs语义同步的分离处理（时间对齐与内容语义匹配作为两个独立过滤条件）

`temporal_vs_semantic_sync` · 详细程度: brief

### [Allegro](../models/Allegro.md)

不适用（无音频）。若类比到纯视觉侧，Allegro 中与之最接近的设计是把「时序一致性」（LPIPS ≥0.05、UniMatch [1.0,100]）与「语义匹配」（CLIP 相似度 ≥0.17/0.20）拆为两个独立的过滤条件，分别作用于不同流水线级次——这一「时序质量」与「图文语义一致性」分离过滤的思路，与 AV 模型中时序同步/语义同步分离处理在方法论上同构。

### [Apollo](../models/Apollo.md)

Apollo 明确地把时序同步与语义同步分离为两个独立的过滤条件，并各配一个专用模型——这是其数据 pipeline 中设计意图最清晰的一处：
- 时序维度：Synchformer，回答「声音和画面在时间轴上对不对得上」（口型是否跟得上语音、脚步声是否踩在落脚帧）。
- 语义维度：ImageBind，回答「声音和画面讲的是不是同一件事」（画面是海浪、声音却是键盘敲击，则时序上可能无冲突但语义完全错配）。
二者并列施加，共同构成「高度同步」的判定。这种「时序 ≠ 语义、需分别把关」的认识，正是为解决论文开篇指出的两类失败模式而设——asynchrony（不同步，时序问题）与 misalignment/lip–speech mismatch（错配，语义与口型问题）。
值得注意的是这一分离在评测侧被对称复用：AV-A（Synchformer 系，时序）与 IB score（ImageBind，语义）作为两个独立指标并列报告，数据过滤器与评测指标同源，形成「用什么筛就用什么评」的闭环——这既保证了训练目标与评测的一致性，也带来指标自证的潜在风险（用 Synchformer 筛出的数据训出的模型，在 Synchformer 系指标上天然占优），论文未讨论这一点。

### [CineDance / CineDance-1M](../models/CineDance.md)

该工作在指标设计层面对时序同步与语义同步做了清晰的分离，属于其方法论上较为清醒的一点：
【时序同步（temporal）】由 SyncNet 承担，输出 Sync-C（置信度）与 Sync-D（偏移距离），刻画的是口型与语音在帧级时间轴上的对齐精度，只在有人物说话的镜头上有意义。
【语义同步（semantic）】由 ImageBind 承担（IB-Score / IB-A Score），刻画的是画面内容与音频内容在跨模态嵌入空间的语义匹配度，适用于音乐、环境音、音效等非语音场景，不要求帧级对齐。
【二者并列而非合并】在数据元数据中两项分别存储，在 CineBench 的 AV Sync 维度中两项分别报告，构成两个独立的判据。此外 Prompt Alignment 维度中的 IB-A Score 进一步把「音频与文本提示的语义一致性」也拆为独立项。
【未做的事】两项均不设过滤阈值，因此并未在数据集构建时用作「两个独立的过滤条件」——分离体现在度量与评测层面，而非过滤层面。

### [CogVideoX](../models/CogVideoX.md) ⚠️

视频侧不适用。CogSound 的公开描述在概念上区分了两个层面——分块时序对齐交叉注意力负责「时序层面的一致性」，基于 GLM-4V 的语义/情感理解负责「语义层面的匹配」，官方表述为「确保音频与视频在时序和语义层面的一致性」，可视为对时序对齐与内容语义匹配的分离处理，但这体现在模型结构设计上，而非作为两个独立的数据过滤条件 [数据侧是否分离处理不确定]。

### [Cosmos-Predict2.5](../models/Cosmos-Predict25.md)

不适用（无音频维度）。若把该概念迁移到纯视觉语境，该 pipeline 确实把「时序结构正确性」与「语义内容正确性」拆成了两个独立过滤级：时序/结构侧由 semantic artifacts filter（类 VTSS）负责，专门剔除劣质转场、画中画等结构异常；语义/内容侧由 content type classifier 与 VLM filter 负责，判断内容类别与是否含不良语义问题。此外镜头切分阶段「排除原始转场」也是纯时序层面的约束。这种分离与 AV 模型中「时序同步 vs 语义匹配」的二分在方法论上同构，但论文未作此类论述。

### [Data-Juicer 2.0](../models/Data-Juicer.md) ⚠️

[不确定] Data-Juicer 未涉及时序同步与语义同步的分离处理，因其不具备音视频同步检测能力（详见 av_sync_detection）。
【类比性的设计思想是存在的】DJ 的整体架构立场恰恰是「每个维度一个独立算子、一条独立阈值，不做加权合并」——所有 Filter 算子都是单指标独立判定后串行叠加，从不提供多指标综合评分。这在方法论上与 Foley-Omni 把 ImageBind（语义）与 Synchformer（时序）拆为两条独立阈值的做法同源：都主张不同性质的错误应由不同的判据独立把关，避免高分维度掩盖低分维度。因此若在 DJ 上扩展音视频同步能力，天然会落到「语义算子 + 时序算子分列」的形态，架构上无障碍。
【现有的一个分离实例】在视觉-文本轴上，DJ 确实把「语义匹配」（video_frames_text_similarity_filter，CLIP 相似度）与「内容属性」（video_tagging_from_frames_filter，标签匹配）分作两个独立算子，而非合并为一个综合对齐分——可视为同一设计原则在另一模态轴上的体现。

### [Foley-Omni](../models/Foley-Omni.md)

明确做了分离处理，这是本pipeline在对齐维度上的一个精细设计点。
 Table 7 的 Alignment 维度并列两项独立指标、两条独立阈值：
  · ImageBind score ≥0.3 负责【语义一致性】—— 判断音频内容与视觉内容是否讲的是同一件事（画面是狗、音轨是狗叫 vs 画面是狗、音轨是无关背景音乐）。ImageBind 通过统一的跨模态 embedding 空间计算余弦相似度，捕捉的是「内容匹配」。
  · Synchformer sync score ≥0.2 负责【时序同步】—— 判断音频事件与视觉事件在时间轴上是否对齐（撞击画面与撞击声是否同一帧）。Synchformer 专为时序对齐检测训练，捕捉的是「时间对齐」。
【为何必须分开】两类错误在真实数据中独立发生且性质不同：后期配乐的视频语义可能一致（悲伤画面配悲伤音乐）但时序完全不对齐；画外音/旁白的时序可能对齐（配合剪辑节奏）但语义与画面不匹配；音画不同步的错帧素材则是语义对但时序错。用单一综合分会让这三类问题相互掩盖——语义分高可以补偿时序分低，导致坏样本蒙混过关。设置两条独立硬门槛则要求样本在两个维度上都合格。
【论文原文佐证】过滤条件的表述把二者并列为两个独立剔除理由：「weak audiovisual semantic consistency, or unreliable synchronization」——用 or 连接，确认是两个独立的淘汰条件。
【延伸到建模与评测】这种分离贯穿全流程：模型侧 CLIP 特征承载语义、Synchformer 特征（Z_sync 通路）承载时序，两路分开注入；评测侧 IB 分数测语义一致、DeSync 测时序对齐，两个指标分开报告。清洗、建模、评测三处保持同一套语义/时序二分，方法论上相当自洽。

### [Goku](../models/Goku.md)

不适用（无音视频模态）。若强行类比 Goku 在视觉侧的「时序 vs 语义」分离处理，可以观察到一个近似结构：镜头切分阶段用 **PySceneDetect（像素/直方图层面的时序突变检测）** 与 **DINOv2 特征余弦相似度（语义层面的内容一致性判定）** 两个相互独立的条件串联把关——前者管「有没有硬切」，后者管「内容是不是同一件事」。这与音视频模型把「时间对齐」与「内容语义匹配」拆成两个独立过滤条件的思路同构，可作为方法论层面的间接参照。

### [HunyuanVideo-Foley](../models/HunyuanVideo-Foley.md)

本工作明确地把时序同步与语义同步作为两个正交维度分别处理，这是其数据流水线设计中最清晰、也最值得肯定的一点：
【显式的双维度分离】论文原文表述为「利用 ImageBind 与 AV-align 来处理语义对齐与时序对齐」（leverage ImageBind and AV-align to address the semantic and temporal alignments），两个工具、两个维度、一一对应，分工无歧义：
- 语义维度（ImageBind）：回答「声音内容与画面内容是否匹配」——这是一个与时间无关的整体性判断，即便音轨整体偏移几秒，语义相似度也几乎不变；
- 时序维度（AV-align）：回答「声音事件与画面事件在时间轴上是否对齐」——这是一个与内容语义无关的判断，即便声音语义完全正确，只要存在系统性时延就应被判为不合格。
【为何两者必须分开】两类错配的成因与后果完全不同：语义错配主要来自后期配乐、旁白、替换音轨（数据源问题），时序错配主要来自音视频编码延迟、剪辑对轨误差、跨设备录制不同步（技术问题）。用单一指标无法同时覆盖——ImageBind 对时延不敏感，AV-align 对内容语义不敏感，两者的检测盲区恰好互补。若只做其一，另一类污染会完整地流入训练集。
【与同类工作的横向定位】MOVA 同样做了这一分离（SynchFormer 管时序、ImageBind 管语义）；UniVerse-1 则两者都缺失（只有语音片段的 SyncNet，非语音数据零对齐检测，语义维度完全空白）。本工作与 MOVA 同属做得较完整的一档，且对 V2A 任务而言这一分离比对联合生成任务更为关键——因为 V2A 的全部学习信号就是「画面→声音」的映射，映射关系本身若有污染，模型将直接学到错误的对应。
【在模型侧的延续】双维度分离在架构上也有对应：语义信息由 SigLIP-2 视觉特征经 joint attention 承载，时序信息由 Synchformer 帧级特征经门控调制通路承载——两路信息走不同通道进入模型，与数据侧的双维度检测形成结构上的呼应。这种「数据侧怎么筛、模型侧就怎么建模」的一致性，是本工作设计上较为完整的体现。

### [HunyuanVideo](../models/HunyuanVideo.md) ⚠️

不适用。无音频，不存在时序同步与语义同步的分离处理。视觉侧的类比是：1.5 将「基础规则过滤（结构性缺陷）」与「视觉质量/美学打分（感知质量）」分为独立条件，属于同一思路在单模态上的体现。[不确定]

### [InstructAV2AV](../models/InstructAV2AV.md) ⚠️

本工作对时序同步与语义匹配做了实质性的分离处理，但分离的位置与 Foley-Omni 不同——Foley-Omni 是在同一个过滤阶段并列两条阈值，本工作则是把二者分派到pipeline的不同环节，由不同机制各自负责。

【时序同步由「结构机制」负责】
  · 数据准入层：TalkNet + Scribe 的「语音须与画内可见说话人时间对齐」是纯时序判据。
  · 数据合成层：frame-wise cross-attention 把音频特征逐帧注入视频生成，是架构级的时序强制。
  · 评测层：Sync-C / Sync-D（vshift=15）、PEAVS 度量时序对齐精度。

【语义匹配由「大模型判断」负责】
  · 数据准入层：非语音片段要求「唯一清晰的主导声源」并赋予语义声音事件标签（如 dog barking），实质是在做声源的语义归属确认。
  · 指令生成层：Qwen3-Omni 基于实例 mask 与音视频上下文生成指令，天然要求「这个视觉实体」与「这段声音」在语义上绑定（狗↔狗叫），否则生成的联合编辑指令不成立。
  · 自动验证层：五维度中的「audio-video synchronization for cross-modal alignment」由 MLLM 判断，实际更偏语义匹配（画面里是猫、音轨是不是猫叫）而非帧级时间精度。
  · 评测层：AV-A（ImageBind 跨模态语义相似度）度量语义匹配。

【为何这种分派是合理的】两类问题的性质决定了适用工具不同：时序对齐是信号级、可精确度量、且可在生成时被架构强制；语义匹配是内容级、需要理解、只能靠模型判断。本工作把信号问题交给信号机制、把语义问题交给大模型，分工清晰。相比之下 Foley-Omni 用两个打分模型（Synchformer 管时序、ImageBind 管语义）并列过滤，是同一思路在纯过滤范式下的实现。

【一个隐含的优势】由于 target 音频是先合成、视频再以其为条件合成，时序对齐在数据中天然近乎完美（不存在真实素材中常见的音画错帧问题），使得模型训练时的时序监督信号异常干净。这也解释了为何本工作敢于在数据侧完全不设同步分数阈值。
[不确定] 未做二者分离必要性的消融，也未报告数据集中语义匹配与时序对齐的独立质量统计。

### [2026 其他音视频联合生成](../models/JAVG_2026_misc.md) ⚠️

本批工作中，「时序对齐」与「语义匹配」的分离处理意识明显增强，NAVA 与 ALIVE 各有代表性做法：
【NAVA —— 数据侧三层分离 + 架构侧显式分离（本批最清晰）】
(1) 数据侧：SyncNet（唇音时序）、SyncFormer（通用事件时序）、ImageBind（跨模态语义）三个算子并行——前两者管时序、后者管语义，是显式的双轨过滤。
(2) 架构侧：NAVA 的核心论点就建立在这一分离上——批评「unified tri-modal designs」（统一三模态设计）「conflate semantic and low-level alignment」（把语义对齐与低层对齐混为一谈）。其 Align-then-Fuse 设计正是为解耦二者：先在专用对齐空间建立音视频的低层/细粒度对应（native alignment），再把上下文（文本、说话人嵌入）作为外部条件通过 cross-attention 注入语义（context as external conditioning）。这是本批中把「时序 vs 语义分离」提升到架构第一性原理高度的唯一工作。
【ALIVE —— 音画相关性作为独立于同步的过滤维度】
音频质量过滤明确采用双准则：第一条是音频质量分（噪声），第二条是「audio-visual coherence」（音画一致性）——后者独立于 TS-TalkNet 的时序同步检测，专门判断音频内容与画面内容是否语义相关，并据此把强相关样本单独归集、把 BGM 等弱相关样本的占比加以管控。这构成一个非常明确的三维过滤空间：时序同步（TS-TalkNet）× 语义相关（MLLM 判断）× 音频质量（MLLM 打分）。特别值得注意的是它对弱相关样本不是一刀切剔除而是「控制比例」——承认非画内音（BGM、旁白）在真实内容中普遍存在且模型需要具备生成能力，只是不能占比过高以免破坏音画因果关系。这比 Ovi 等工作「只做时序过滤、语义靠 caption 隐式绑定」的做法成熟。
【OmniCustom】只做时序（SyncNet 双阈值），无语义匹配过滤——但其任务性质（单人说话头，音频必为画中人所说）使语义错配风险天然很低，属合理省略。
【ITS-JAVG —— 在推理侧把二者拆成不同验证器并证明其冲突】
验证器组合中，JavisScore 度量细粒度时序同步，AVHScore 与 ImageBind 度量语义一致性——论文的核心实证正是这两类目标之间存在非对称 trade-off：优化时序同步会牺牲语义一致性，反之亦然，单一验证器无法兼顾。这为「时序与语义必须分离处理并显式平衡」提供了直接的量化证据，也是本批工作中对该问题最有理论价值的贡献。其 ARW 算法本质上就是在时序类与语义类奖励之间做自适应权衡。
【CCL】在评测中同时报告 Sync-C/Sync-D（时序）、DeSync（时序）与 IB（ImageBind，语义）——指标层面区分了二者，但数据侧无对应的分离过滤[不确定]。
【StreamChar / Baton】未做此区分[不确定]。Baton 的 planned tokens 试图统一编码语义蓝图，RS-RoPE 负责时序注入——架构上有隐含分工（语义靠 planner、时序靠 RoPE），与 Ovi 的「RoPE 管时序、cross-attention 管语义」思路一脉相承。

### [音视频联合生成基线合集](../models/JavisDiT_baselines.md) ⚠️

JavisDiT 系列在这一维度上有本合集最明确、最有理论深度的处理，其余各家分离度较弱：
【JavisDiT —— 分层设计即为时序与语义的显式分离】HiST-Sypo（Hierarchical Spatio-Temporal Synchronized Prior，分层时空同步先验）的「分层」本质就是把对齐拆成两层：
- 粗粒度全局先验（coarse-grained global prior）—— 对应语义层面的匹配（整体内容与风格是否一致）；
- 细粒度时空先验（fine-grained spatio-temporal prior）—— 对应时序与空间层面的精确对齐（何时、何处发出何声）。
两类先验从文本 prompt 中分别估计，再同时引导两路去噪。这是本合集中唯一把「语义同步」与「时序同步」在模型架构层面显式解耦的设计。
- JavisBench 的评测维度同样体现这一分离：五大维度中「时间构成（Temporal Composition）」与「空间构成（Spatial Composition）」是时序/空间维度，「事件场景（Event Scenario）」「声音类型（Sound Type）」「视频风格（Video Style）」是语义维度。
- 评测指标层面也是分离的：DeSync（Synchformer，纯时序）vs AV-IB / AVHScore（ImageBind 系，纯语义相似度）vs JavisScore（滑窗 + 最差 40% 帧，介于两者之间但偏时序局部）。
【JavisDiT++】延续这一分离，并在架构上进一步固化：TA-RoPE 专管帧级时序对齐（时序），MS-MoE 与跨模态注意力管语义交互（语义）；AV-DPO 的奖励设计同样分离——Synchformer 打时序同步分，ImageBind 打跨模态语义相似度分。
【Harmony —— GLDI 模块是架构层面的时序/语义解耦】Global-Local Decoupled Interaction 明确把交互拆成两支：全局分支负责「风格对齐（style alignment）」（语义层面），局部分支负责「时序精度（temporal precision）」（时序层面）。这与 JavisDiT 的分层先验异曲同工，是本合集第二个明确做此分离的工作。数据过滤层面则未分离——只有一个笼统的「音视频一致性打分模型」[不确定：该模型度量的是时序还是语义一致性]。
【MM-Diffusion / AV-DiT / UniAVGen】未做时序与语义同步的显式分离[不确定]。MM-Diffusion 的 random-shift attention 限定在时间邻域内做注意力，隐含了对时序局部性的偏好，但并非有意的分离设计。
【方法学价值】本合集验证了一个趋势：2025 年后的音视频联合生成工作普遍认识到「时序对齐」与「语义匹配」需要不同机制处理——JavisDiT 用分层先验、Harmony 用解耦交互模块、Ovi 用 RoPE 管时序 + cross-attention 管语义，三者殊途同归。

### [可灵 3.0 / 可灵视频 3.0 Omni](../models/Kling_30_Omni.md) ⚠️

[不确定，有强间接证据] 团队方法论上明确把“时序对齐”与“语义对齐”视为两个独立维度：Kling-Foley 在架构上分设视觉语义表征模块（管语义匹配）与视听同步模块（管帧级时间对齐），评测亦分列 semantic alignment 与 temporal alignment；数据侧则用 CLAP 计算音频与文本标签一致性（语义条件）、用 VAD/事件时长规则约束时间维度（时序条件），并要求“音效必须来自画面中可见的物体或动作”（语义因果条件）。可灵3.0 Omni 是否延续该二分尚无直接披露。

### [LTX-2](../models/LTX-2.md) ⚠️

论文未在数据过滤层面区分「时序同步」与「语义匹配」两个独立条件——数据侧只有「音频信息量」这一条筛选准则。但在方法论叙述与架构设计上，两者被明确区分并分别对应不同机制：
(1) 时序同步：由跨模态注意力中只保留 RoPE 时间分量来承担（明确表述为「强制跨模态注意力聚焦于时间上的同步而非空间对齐」），目标是唇同步与撞击-音效的亚帧对齐；
(2) 语义/环境匹配：由双向依赖建模承担——论文的核心论点是「唇同步主要由音频驱动，而声学环境（混响、foley）由视觉上下文决定」，因此必须联合建模双向依赖，这是反对级联 V2A/A2V 方案的主要理由；推理时增大跨模态引导 s_m 同时改善「时序同步」与「语义连贯」，论文将二者并列表述。
数据侧是否有对应的两类过滤条件，未披露。[不确定]

### [LongCat-Video](../models/LongCat-Video.md) ⚠️

在 Avatar 1.5 的过滤链中，时序对齐与语义匹配确实被拆为不同环节，但并非作为「同一目标的两个正交条件」显式论述：时序侧为 lip-sync verification（音视频偏移检测）与主动说话人检测（音轨归属的时间区间判定）；语义/内容侧为 Qwen3-Omni + Qwen3-VL 对「主体是否在说话」的语义一致性双判，以及 caption 层面对场景/关系/情节的客观描述约束。在线过滤链把「audio synchronization」列为首级、把「文本与视觉质量」列为后续独立一级，结构上体现了二者分离。基础版无此维度。[不确定：团队是否有意作此区分]

### [MOVA](../models/MOVA.md)

明确分离，且是 MOVA 数据过滤中一个被显式论证的设计决策：
- 时序同步由 SynchFormer 负责（DeSync 偏移量 ≤ 0.5），刻画“声音与画面事件是否在同一时刻发生”。
- 语义同步由 ImageBind 负责（IB-Score ≥ 0.2），刻画“声音内容与画面内容是否语义相关”。
- 二者作为两个独立条件，但组合方式是**宽松的逻辑 OR 门**而非 AND：“we apply a relaxed logical ‘OR’ gate between semantic and temporal alignment. A video is retained if it satisfies either IB-Score ≥ 0.2 OR DeSync ≤ 0.5.” 论文给出的理由是两类音频的对齐性质根本不同——语义相关的环境音/氛围声没有清晰的时序 onset，硬要求时序同步会把它们全部误杀；而快速动作音效可能在语义嵌入空间中得分不高，硬要求语义分会把它们误杀。用 OR 门可同时保留两类有价值的样本。
- 该分离在评测侧同样保持：Table 4 将 AV-Align 拆为 DeSync（时序）与 IB-Score（语义）两列分别报告，并单列 Lip Sync（LSE-D/LSE-C）作为最细粒度的连续时序对应。
- 论文在 6.3 节进一步区分了两类同步任务的难度层级：离散、onset 驱动的事件（如“切水果”“击鼓”）只需对齐少数显著时间点；而语音需要口型与音素在长时间跨度上的连续、细粒度对应，是最苛刻的一类。

### [合并条目：① Mochi 1](../models/Mochi_MAGI_Motif.md)

不适用（无音频）。若把该方法论映射到纯视觉侧，本组中确有明确的「时序质量」与「语义匹配」分离处理：
· MAGI-1 把时序侧（RAFT 光流的整体/前景/背景三档运动强度上下阈值、相机稳定性的光流一致性、幻灯片运动的光流散度）与语义侧（Transition Detection 用 CLIP 关键帧语义相似度判多镜头、MLLM 高级过滤做语义级复筛）拆成流水线上不同层级的独立条件。
· Motif-Video 2B 的分离更彻底且互为补充：底层时序信号是 UniMatch 光流的双侧截尾，语义信号是 VLM 的 action=Dynamic 标签，两者在 720p SFT 阶段被同时施加——前者管「像素层面动没动」，后者管「语义层面算不算动态动作」。同理，SigLIP 相似度用于时序连续性判定（stitch 合并），VLM 的 multi_scene 标签用于语义层面的多镜头判定，二者亦是「时序 vs 语义」的双轨兜底。
这一「低层统计 + 大模型语义」双轨过滤的结构，与 AV 模型中时序同步与语义同步分离处理在方法论上同构。

### [Movie Gen](../models/Movie_Gen.md)

概念上做了清晰的三层分离，但数据过滤时用的是同一个 CAVTP 分数 + AED 标签的组合来近似区分，并未训练两个独立的时序/语义打分器：
· 概念层：论文把音视频关系拆为「画内在屏（时序确定性对应，最强时间约束）」「画内画外（环境合理性与事件因果顺序，语义+弱时序）」「非画内（情绪与叙事语义，无时序约束）」三层，并指出三层对模型能力的要求依次上升。
· 数据层：diegetic/non-diegetic 的划分本质上就是「有无时间因果约束」的划分，用 CAVTP 阈值实现；音频类型（voice/music/sound）的划分则承载语义内容维度。二者交叉成表23的六格分类，实际是把「时序可对齐性」与「语义类别」作为两个正交轴来管理数据配比。
· 评测层分离最彻底：diegetic sound correctness（该不该响这个声音——语义匹配）与 diegetic sound synchronization（响得准不准——时序对齐）被拆为两个独立的人工评测维度；non-diegetic 侧则拆为 music mood alignment（情绪语义）与 music motion/scene alignment（与动作/场景/剪辑点的对齐）。

### [NVIDIA NeMo Curator](../models/NeMo_Curator.md)

不适用。框架不处理音视频关系，因而不存在时序同步与语义匹配的分离处理。
若强行类比，其视频侧确实存在「结构性指标」与「语义性指标」的分工设计：运动向量分数、DOVER 画质分属结构/信号层判据，由轻量统计与小模型承担；视频类型 taxonomy、文字叠加检测属语义层判据，由嵌入 + MLP 承担；caption 生成属语义描述层，由 VLM 承担。这种「按判据性质分配不同量级模型」的成本分层思路可迁移到 AV 同步过滤的设计中（如先用轻量能量包络相关性粗筛时序对齐，再用大模型做语义匹配复核），但框架本身未提供此类实现。

### [OmniHuman 数据集 + OHBench](../models/OmniHuman.md) ⚠️

OmniHuman 对时序与语义两个维度做了相当清晰的分离，且分离体现在数据侧与评测侧两处：
【数据侧：三个层次的分离处理】
- 时序层（帧级对齐）：SyncNet 的时间偏移量，硬约束 ±3 帧。管的是「说话的时刻对不对」；
- 归属层（声源同一性）：SyncNet 响应的贪心匹配 + ArcFace 相似度 0.55。管的是「这声音是不是画面里这个人发出的」——这一层在多数工作中不存在（UniTalking 用 LightASD 做了类似的事但只作二值过滤，OmniHuman 则做到了逐段指派）；
- 语义层（内容合理性）：Gemini-3 评估背景合理性与交互合理性。管的是「这个场景与这段交互在语义/物理上讲得通吗」。
三层的判据完全独立：偏移量是信号级的、归属是匹配级的、语义是理解级的，分别由不同性质的模型承担（专用同步模型 / 匹配算法 / 大模型），不会互相掩盖。这一分层比 UniTalking 的两层（LightASD 归属 + LipSync 时序）又多了一层语义判断。
【评测侧：同样分离】
- 时序同步：V-A（ImageBind，视听事件时序偏移）与 Lip-Sync（SyncNet，口型级）；
- 语义匹配：T-A（CLAP，音频与文本的语义相似度）——衡量生成的声音在语义上是否符合文本描述，与时序无关；
- 交互语义：IN（交互自然度）、LR（倾听者真实度）、ES（情绪相似度）、CN（接触自然度）等 Gemini-3 主观维度，衡量的是多主体之间的语义/行为协调性。
【一个值得注意的语义维度：情绪协调】ES（Emotion Similarity）评估双人之间「协调的情绪表达」——这是一种跨主体的语义一致性，既非时序对齐也非单体属性，在同类基准中未见先例。
【局限】数据侧的语义层判定（Gemini-3 背景合理性/交互评估）未说明其是作为硬过滤条件还是仅作标注字段，也未给出评分阈值。[不确定]

### [Open-Sora 系列](../models/Open-Sora.md)

不适用。无音频模态。（若强行类比，视觉-文本的语义匹配由 Open-Sora 1.x 的 CLIP matching score 承担，但与音视频同步无关。）

### [Ovi](../models/Ovi.md) ⚠️

Ovi 在数据过滤层面基本只处理「时序同步」，语义匹配主要交给 caption 与训练机制，两者未被设计成两个并列的独立过滤条件。
【时序同步（有显式过滤）】SyncNet 的 offset 与 confidence 双阈值，专门约束语音与嘴部动作的时间对齐；音频平均音量门槛排除无效音轨。
【语义同步（无显式过滤）】没有使用 CLAP/AV-CLIP 类跨模态语义相似度打分来剔除「音画内容不匹配」的样本（例如画面是海滩、音轨是无关旁白/后期配乐的情形）。语义一致性依赖两条间接路径：(1) caption 阶段 MLLM 同时看画面与听音轨，把双方内容写进同一条描述，从而在文本条件层面强制语义绑定；(2) 训练时双向跨模态注意力让两塔互为语义上下文。
【潜在缺口】画外音（voice-over）、后期配乐、配音译制等「时序可能对齐但语义与画面无因果关系」的样本，仅靠 SyncNet 难以完全剔除（SyncNet 对无嘴部画面的片段本身置信度低，可部分间接过滤），论文未讨论此风险[不确定]。
【架构层面的时序/语义分工】值得注意的是，Ovi 在模型侧对二者做了明确分工：时序由 scaled-RoPE（31/157≈0.197 频率缩放）负责，语义由双向 cross-attention 负责——论文摘要即表述为「blockwise exchange of timing (via scaled-RoPE embeddings) and semantics (through bidirectional cross-attention)」。这种「时序与语义分离处理」的思想体现在架构而非数据过滤上。

### [Script-a-Video](../models/Script-a-Video.md)

MTSS 在结构上把时序同步与语义同步彻底分开处理，这是 Relational Grounding 双链接设计的直接体现：
【时序对齐通道：时间链接（temporal links）】由三层时间戳承担——shot 级 time_range、event 级 time_range、描述内嵌的 intra-description / micro-level timestamps，加上 shared anchors（共享锚点）。这条通道只回答「什么时候」，全部信息为可做区间运算的数值区间。
【语义对齐通道：身份链接（identity links）】由中心化 Reference 实体库承担——Shot 的 references_in_shot、Event 的 speaker 均指向同一 ID 空间。这条通道只回答「是谁/是什么」，全部信息为符号引用。
【第三条隐性通道：因果对齐】Event 流的准入原则「音效必须由画面中可见的主体产生」实际上编码了声音与视觉的因果/来源关系，既非纯时序也非纯身份，而是「这个声音是画面里哪个东西发出的」。Shot 的 active_events 字段把镜头与并发事件关联，同样承载这层含义。
【三通道分离的价值】论文的核心论证正建立在此：单体式 caption 把 WHO、WHERE、WHEN 揉在同一段文本里，下游模型必须先做身份解析与时序去歧义才能推理；MTSS 把三者预先消歧，模型可直接专注逻辑推断。这解释了为什么 MTSS 在推理类基准（Daily-Omni、WorldSense）上的增益（Qwen3-Omni +127%）远大于在描述类基准上的增益。
【过滤条件层面】论文未把二者作为两个独立的数据过滤条件使用（因为整体上就没有披露过滤条件），二者的分离体现在表示层而非过滤层。这是与「时序同步与语义匹配作为两个独立过滤闸门」的传统做法不同的路径——MTSS 把这两个维度从「过滤时的判据」转化为「标注中的字段」。

### [Seedance 2.0 与 Seedance 1.5 pro](../models/Seedance_20_Seedance_15_pro.md) ⚠️

报告未在数据过滤层面明确拆分这两个条件，但评测体系已完成该拆分，可视为其数据质量观的映射：SeedVideoBench 1.5/2.0 将「音视频同步（Audio-Visual Sync）」定义为纯时间对齐维度（唇动-语音、音效-视觉事件的时序对齐），而将内容语义匹配另设为「音频指令跟随（Audio Prompt Following）」维度（评估人声、对白、音效是否忠实于用户指令与预期语义、有无语义漂移，典型失败模式含指定音效缺失、语言或方言不准确、以及「有语音但无对应唇动」的音画错配）；此外还独立设「音频质量（Audio Quality）」与「音频表现力（Audio Expressiveness）」两个维度。这种四维拆分强烈暗示数据侧亦按时序对齐与语义匹配两条独立条件把关。[不确定：数据过滤是否真按此拆分]

### [SkyReels 系列](../models/SkyReels.md) ⚠️

SkyReels-V4 在数据侧对二者做了事实上的分工，但论文未把它们表述为两个并列的过滤条件：
(1) 时序同步：由 SyncNet 的 |offset|≤3 条件承担，纯粹判断时间轴对齐程度；
(2) 语义/内容匹配：由 SyncNet 的 confidence>1.5 条件部分承担（置信度低意味着音画内容本身不匹配，如画外音、背景音乐场景），另由结构化 caption 的四类音频 token 承担——caption 把「画面里发生什么」与「听到什么」写在同一条文本中，语义匹配通过文本条件而非独立过滤器保证；
(3) 架构侧：逐 block 的双向交叉注意力负责持续对齐，RoPE 时间维缩放负责两模态 token 率的时间刻度统一。
论文未设置独立的「音画语义一致性」判别模型（例如用全模态 LLM 判断音轨与画面是否讲同一件事），也未说明音效/音乐类样本（无唇部信息、SyncNet 不适用）如何做语义匹配校验，这是该 pipeline 可见的缺口。[不确定]

### [Sora 2](../models/Sora_2.md) ⚠️

完全未披露。是否将「时间轴对齐」与「内容语义匹配」拆分为两个独立过滤条件，无任何信息。从能力描述可看出模型同时具备时序对齐（唇形与语音逐帧对齐、音效与碰撞时刻对齐）与语义匹配（环境音符合场景语境、音乐情绪匹配画面基调）两类能力，暗示数据侧可能有分离处理，但纯属推断。[不确定]

### [Step-Video-T2V](../models/Step-Video-T2V.md) ⚠️

不适用（无音频）。在纯视觉侧存在一个结构上的类比：pipeline 把「时序/运动维度」（第3阶段 Farneback 光流的三项运动分）与「语义匹配维度」（第6阶段 CLIP Score 图文对齐）设为两个相互独立的过滤条件，分别处理「动得对不对」和「拍的是不是描述的东西」，这与 AV 模型将时序同步与语义匹配拆成两个独立门限的思路同构，可作为方法论层面的弱参考。[不确定]

### [UniTalking](../models/UniTalking.md) ⚠️

UniTalking 是本次调研中少数在数据过滤层面真正把「语义/来源对齐」与「时序对齐」分离为两个独立条件的工作，尽管论文本身未用这套术语来表述：
【语义/来源维度：LightASD】判定「音轨中的语音是否由画面中的人物发出」。这本质上是内容层面的同源性判断，回答的是「声画是不是一回事」。剔除的是画外音、旁白、后期配音、背景人声等——这类样本的音画在时序上可能完全无关，属于语义级错配。
【时序维度：LipSync】在确认同源之后，判定唇部运动与语音在帧级上是否对齐。回答的是「对上的时刻准不准」。剔除的是音画偏移（时间戳漂移、剪辑错位、配音替换）。
【两者的串联顺序具有方法论意义】先做语义同源判定、再做时序精度判定，逻辑上是必要的——对一个声源根本不在画面内的样本计算唇同步分数是无意义的（分数低但原因不是时序偏移）。若跳过 ASD 直接用 SyncNet 阈值卡，画外音样本虽也会因低分被淘汰，但淘汰理由与真正的时序错位样本混同，且阈值需要为两类问题共同妥协。分开处理使每一级的判据更纯粹。
【与对照工作的比较】
- UniVerse-1：只有时序维度（SyncNet conf > 2.0），语义维度完全空白，明确未做画外音源剔除——其数据源含大量 vlog 与教程内容（后期配乐、旁白极常见），这是实质性的数据风险；
- MOVA：把 Stage 2 对齐评估拆为时序（SynchFormer）与语义（ImageBind）两个正交维度分别设阈，是显式的双维度设计；
- UniTalking：双维度分离在事实上成立（ASD + LipSync），但因 domain 极窄（纯说话人语音），其「语义对齐」被具体化为「声源同源性」这一特定形式，而非 MOVA 式的通用视听语义匹配。对于说话人 domain 而言，这个具体化是恰当且更有针对性的。
【缺失】未使用 ImageBind 类的通用跨模态 embedding 相似度做语义匹配；未做 caption 与音视频内容的一致性校验；两级均无阈值披露。[不确定]

### [UniVerse-1](../models/UniVerse-1.md) ⚠️

UniVerse-1 未把时序同步与语义同步作为两个独立过滤条件处理，实际上只覆盖了时序同步的一个特例（口型），语义同步维度完全缺失：
【时序同步】仅通过 SyncNet conf > 2.0 覆盖语音片段的唇音时序对应；非语音数据的事件级时序对齐（如「门关上的瞬间对应撞击声」）无任何检测与过滤。
【语义同步】完全没有——没有 ImageBind 类的跨模态 embedding 相似度过滤，没有「画面内容与声音内容是否语义匹配」的判定，也没有用 VLM/LLM 做视听语义自洽性裁决。这意味着诸如「画面是厨房但音轨是背景音乐/画外配音」这类语义错配样本无法被识别和剔除——考虑到数据源包含大量 vlog 与教程类内容（这类内容后期配乐、旁白极常见），这是一个实质性的数据风险。
【对比参照】MOVA 明确把 Stage 2 的对齐评估拆为时序（SynchFormer）与语义（ImageBind）两个正交维度分别设阈；UniVerse-1 两者都没有独立设施，语义维度更是完全空白。
【隐含的替代机制】论文的主张是：只要保证「原生同步音轨 + 在线标注窗口同源」，时序与语义的一致性就在数据源头得到了保障，无需事后过滤。这在原生音轨确实与画面对应时成立，但对配乐/旁白/后期音效类内容不成立。论文未讨论这一假设的边界。[不确定]

### [Unison](../models/Unison.md) ⚠️

Unison 未把时序同步与语义同步作为两个独立的数据过滤条件处理，但在训练与评测两个层面对二者做了较清晰的区分：
【数据过滤层：只有时序，无语义】唯一的过滤机制 lip-filtering 是纯时序性质的（唇动与语音在时间上是否对应）。无任何语义匹配过滤——没有 ImageBind 类跨模态 embedding 相似度门槛，没有「画面内容与声音内容是否语义一致」的判定，没有 VLM 做视听语义自洽性裁决。因此诸如「画面是厨房但音轨是后期配乐」的语义错配样本无法被识别剔除。不过需注意，画外音剔除（off-screen voice-over）在功能上部分覆盖了语音类的语义错配——它排除了「声画不属同源」的最常见情形。
【训练层：语义被交给了 SCG 门控】Unison 的处理方式是把语义一致性从「过滤问题」转化为「条件建模问题」——SCG 依据 caption 与 transcription 的全局语义向量动态决定语音流与音效流的相互影响强度。即模型不预先剔除语义不匹配样本，而是学会根据文本语义判断当前场景该以何种声学配比生成。这是一条与 MOVA（Synchformer 管时序、ImageBind 管语义，两个正交阈值分别过滤）截然不同的思路。
【评测层：时序与语义指标分列，划分清晰】这是 Unison 评测体系中设计较好的一处：
- 时序对齐类：LSE-C / LSE-D（SyncNet，唇部时序）、DS（Synchformer，模态起音的绝对时间偏移）；
- 语义一致类：TA（CLAP，音频-文本语义一致）、TV（VideoCLIP-XL-V2，视频-文本语义对齐）、AV（ImageBind，音频-视频语义相似度）。
Unison 的 AV 达 0.91，为 TI2AV 全场最优（LTX-2 0.89、MOVA 0.88、Ovi 0.87、UniAVGen 0.81、UniVerse-1 0.62），说明其视听语义匹配度确实领先——但这是训练策略的结果而非数据过滤的结果。
【核心张力】Unison 在评测端严格区分时序与语义，在数据端却对二者都近乎不设防（时序仅覆盖唇部，语义完全不覆盖），全部指望训练机制补偿。DS 0.08 与 AV 0.91 的成绩表明这条路走得通，但也意味着其效果依赖于上游开源数据集本身的语义纯净度——若换用嘈杂的自采网络数据，同一套方法未必成立。论文未讨论这一边界条件。[不确定]

### [Veo 3 / Veo 3.1](../models/Veo_3_Veo_31.md) ⚠️

[不确定] 未披露是否将「时序对齐」与「内容语义匹配」拆分为两个独立过滤条件。可作间接线索的是安全侧的多模态分类器设计思路——官方明确指出需要判断 caption 与 video 组合后的语义结果，体现了 Google 在数据侧确实关注跨模态语义匹配问题，但该机制服务于安全而非同步质量。

### [Vidu S1](../models/Vidu_S1.md)

存在事实上的分离处理，但论文未用此术语表述：时序同步由预过滤的音视频同步指标 + VAD 语音段时间戳 + 切点避开语音中段共同保证；语义/身份层面的声画匹配则由 ASD 主动说话人检测与 onscreen/offscreen 分类保证（判断「这个声音是不是画面里这个人发出的」），二者是流程中两个独立的过滤条件。此外 caption 的 dual-path 策略（视觉属性只看画面、听觉属性只听音轨）也是在语义标注层面刻意解耦两模态以避免跨模态幻觉。

### [主流视频预训练数据集合并调研：Panda-70M、InternVid、Koala…](../models/pretraining_datasets.md)

不适用（无音频模态）。若在视觉侧做类比，可对应的是「时序一致性」与「图文语义匹配」被分开处理：**时序侧**由切镜/转场检测（Panda-70M的ImageBind首尾距离≤1.0、Koala-36M的CSS+3σ、LVD-2M的0.5fps ContentDetector阈值50）与相邻帧一致性（OpenVid-1M的CLIP相邻帧余弦相似度双向剔除）承担；**语义匹配侧**由图文相似度承担（Panda-70M的UMT matching_score>0.43但仅用于选caption不用于剔除；**UltraVideo的VideoCLIP-XL-v2相似度<0.2剔除是七者中唯一把图文语义匹配作为独立剔除条件的**）。但这与音视频同步无关。

## 音频质量过滤（SNR、静音检测与静音占比阈值、无音轨剔除、画外音源剔除、背景音乐分离）

`audio_quality_filtering` · 详细程度: brief

### [Allegro](../models/Allegro.md)

不适用。原始视频的音轨在切分阶段即被丢弃，无 SNR、静音检测、无音轨剔除、画外音源剔除或背景音乐分离等处理。

### [Apollo](../models/Apollo.md) ⚠️

音频质量过滤是 Apollo 独立成段（Section 4.1 后半）的环节，维度枚举较全但阈值只公开一个：
【剔除条件】低 SNR（信噪比）、低 MOS（平均意见分，感知音质）、异常削波（abnormal clipping）、失真（distortion）、噪声（noise）。
【静音控制】要求静音占比低于 20%（「ensuring less than 20% silence」）——这是音频侧唯一公开的数值阈值，且在视频过滤条款中重复出现（「discard those videos with ... or over 20% silence」），说明静音占比是横跨音视频两道闸门的强约束，目的是避免模型学到大段无声输出。
【其他要求】高保真（high fidelity）、格式统一（consistent formatting，指采样率/声道/编码的规整化）。VAE 侧规格为 44.1 kHz 输入。
【未涉及的常见环节】论文未提及无音轨样本的剔除（虽然逻辑上被 pipeline 隐含要求）、未提及画外音/旁白源的剔除（即声源不在画面内的情形，这对唇同步训练是重要干扰源，MOVA 等工作会专门处理）、未提及背景音乐分离（BGM separation，如用 Demucs/UVR 剥离配乐）——对以歌唱为一级子集的 Apollo 而言，是否分离伴奏与人声是一个关键但未回答的问题。
【SNR/MOS 具体工具与阈值】未披露（如是否用 DNSMOS、UTMOS、Brouhaha 等）。[不确定]

### [CineDance / CineDance-1M](../models/CineDance.md) ⚠️

【使用的指标】
  · DNSMOS —— 语音信号保真度的无参考 MOS 估计，衡量噪声、失真等退化程度；
  · CLAP embedding 的时序方差（temporal variance）—— 衡量音频内容随时间的变化丰富度，可间接识别单调、恒定或近似静音的音轨（方差极低往往意味着无信息量的音频）。
【策略】同样遵循全局的「不硬剪枝」原则，两项指标均存为元数据供下游筛选，论文未给出 DNSMOS 分数下限或方差阈值。
【入口约束】素材准入要求「自带原生音轨」，因此无音轨样本在采集阶段即被排除。
【非语音区间处理】在 ASR-to-Character 绑定的窗口化方案中，会先过滤掉非语音区间（filters non-speech intervals）再做绑定——这是对静音/非语音段的一种功能性处理，但目的是提升绑定准确率而非数据过滤。
【未涉及】未提及 SNR 计算、静音占比阈值、画外音（off-screen voice）源剔除、背景音乐分离（source separation，如 Demucs/BS-RoFormer）等常见音频清洗手段。作为影视素材，背景音乐与对白混叠是普遍现象，论文未说明如何处理。[不确定]

### [CogVideoX](../models/CogVideoX.md) ⚠️

[不确定] CogVideoX 视频侧不涉及音频，训练时直接丢弃音轨，无 SNR、静音检测、无音轨剔除、画外音剔除或 BGM 分离等处理。CogSound 侧的音频质量过滤策略（信噪比阈值、静音占比、人声分离等）未公开。

### [Cosmos-Predict2.5](../models/Cosmos-Predict25.md)

不适用。pipeline 无任何音轨处理：不做 SNR 估计、不做静音检测与静音占比阈值、不剔除无音轨样本、不做画外音源判定、不做背景音乐分离。GPU 转码阶段的描述也只针对视觉流。

### [Data-Juicer 2.0](../models/Data-Juicer.md) ⚠️

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

### [Foley-Omni](../models/Foley-Omni.md) ⚠️

音频质量过滤集中于单一指标 + 一道能量门控，设计简洁。
【主过滤器】Meta AudioBox Aesthetics 分数 ≥0.6。这是 Meta 开源的无参考（no-reference）音频美学/质量评估模型，输出感知层面的综合质量分。选用学习型感知评估而非传统信号指标（SNR、THD、频谱平坦度）是2025–2026年音频数据清洗的方法演进——传统 SNR 无法识别过度压缩、削波、编码劣化、混音失衡等问题，而 AudioBox Aesthetics 与人耳判断相关性更高。
【静音处理】过滤阶段第一条剔除条件即为 silence（静音）。实现上很可能由 AudioBox Aesthetics 低分自然覆盖（静音片段美学分极低），论文未描述独立的静音检测器或静音占比阈值。无音轨样本自然被排除在外（本任务音轨是必需的监督目标）。[不确定] 未给出静音判定的能量阈值、静音时长占比上限等具体参数。
【能量门控（字段级）】声学后验证阶段对分离后的每个 stem 施加 E(a_c) > −35 dB 的 RMS 能量门槛。这不是片段级过滤而是字段级置空——不达标只删除该字段标注，片段本身仍保留用于其他字段的训练。这个粒度设计很关键：一个只有语音没有音乐的片段仍是合格的 VisualTTS/V2ST 训练样本，不应整条丢弃。
【背景音乐分离】使用 Bandit（cinematic audio source separation 模型）做 speech/effects/music 三路分离，但用途是验证标注而非清洗音轨——分离结果不作为训练目标，模型仍学习生成完整混合音轨。这与一些 V2A 工作「分离掉背景音乐再训练」的做法不同，Foley-Omni 反而要求模型能生成含音乐的完整配乐。
【画外音源剔除】[不确定] 论文未描述专门的画外音（off-screen sound）、旁白、解说的识别与剔除机制。ImageBind ≥0.3 的语义一致性门槛可以部分过滤掉与画面无关的旁白，但不是针对性设计。

### [Goku](../models/Goku.md)

不适用。数据流水线中无任何音频质量过滤环节：无 SNR 信噪比判定、无静音检测与静音占比阈值、无无音轨剔除、无画外音/旁白源剔除、无背景音乐分离（如 Demucs/BS-RoFormer）。训练数据形态为纯「视频-文本对」，音轨在数据构建中被完全忽略。

### [HunyuanVideo-Foley](../models/HunyuanVideo-Foley.md) ⚠️

音频质量过滤是本工作清洗流水线的绝对重心，共四道关卡，层次分明：
【1. 无音轨剔除】流水线第一步即剔除不含音频流的视频，一票否决。全部训练数据均含原生同步音轨。
【2. 静音占比阈值（比例型判据）】计算每个 8 秒片段的静音占比，超过 80% 者丢弃。设计特点是采用连续比例指标而非二元「是否含静音」判定，且阈值宽松（容忍高达 80% 的静音）——这一宽松度是刻意为 Foley 任务留出的空间：典型的 Foley 训练样本恰恰是「大段安静 + 一次短促事件音」的形态（如寂静房间里的一声关门），若按 UniVerse-1 式的「检出静音即剔除」处理，会把最有价值的样本误杀。80% 门槛实际只用于剔除近乎全静音的废片。未披露静音判定的能量门限与时间分辨率。[不确定]
【3. 带宽 / 有效采样率检测】仅保留有效采样率超过 32 kHz 的样本。「有效」二字是关键——通过频谱分析确认高频段确有实际能量，而非仅读取文件头声明值，因此能识别并剔除低码率音源上采样伪装的伪高清音频。32 kHz 有效采样率对应 16 kHz 的可听频率上限，这是保证镲片、玻璃、金属摩擦、布料摩擦等高频丰富的 Foley 音有足够频谱信息的必要条件，也与模型 48 kHz 的输出规格直接匹配——训练数据的带宽上限决定了模型能生成的频率上限。这条判据在同类工作中较为少见，是本工作音频质量把控上的一个亮点。
【4. 音频美学评分 + SNR】AudioBox-Aesthetics 工具包作为主指标（可输出 PQ 制作质量 / PC 制作复杂度 / CE 内容愉悦度 / CU 内容有用性四维），SNR 信噪比作为补充指标。均未给出阈值数值，也未说明四个维度中实际采用了哪几个。[不确定]
【未做的音频处理】
- 无背景音乐分离（source separation）——检测到音乐只打标签不剥离，因此训练样本中的 Foley 音效可能与背景音乐混在一起，这会给模型带来「事件音总是伴随音乐」的错误关联风险，论文未讨论；
- 无画外音/旁白源剔除——依赖 ImageBind 语义对齐间接过滤，但旁白往往语义上与画面相关（解说词描述画面内容），可能骗过语义检测；
- 无削波、失真、爆音等录音缺陷的专项检测（部分可由 AudioBox PQ 覆盖）；
- 无响度归一化说明。
【总体评价】音频侧的过滤设计明显比视觉侧用心，且带宽检测与比例型静音阈值两处体现了对 Foley 任务特性的理解。主要缺陷是阈值不公开、以及不做音源分离。

### [HunyuanVideo](../models/HunyuanVideo.md) ⚠️

不适用。无音轨处理，无 SNR/静音检测/背景音乐分离等环节。[不确定]

### [InstructAV2AV](../models/InstructAV2AV.md) ⚠️

音频质量控制分布在准入过滤与合成保护两个层面。

【准入过滤（作用于真实素材）】
  · 静音检测：PyDub 剔除低于 -45 dBFS 的静音片段。这是论文中唯一给出确切数值的阈值。-45 dBFS 是较宽松的门槛（约等于「几乎完全无声」），意在剔除无音轨或纯静默的片段，而非做精细的响度控制。[不确定] 未说明是整段平均电平还是逐段判定、未给出静音占比上限（如「静音时长超过 30% 即剔除」这类约束）。
  · 音频质量评估：Audiobox-Aesthetics（Tjandra et al., 2025，Meta 的无参考音频美学评估模型）剔除低质音频。这与 Foley-Omni 使用同一工具，属于2025–2026年音频数据清洗的新范式（用学习型感知质量评估替代传统 SNR/THD 等信号指标，能识别过度压缩、削波、编码劣化等传统指标测不出的问题）。[不确定] 阈值数值未给出（Foley-Omni 给出的对应阈值为 ≥0.6，可作参考量级）。
  · 声源清晰度：非语音片段剔除声源模糊（ambiguous sound sources）者，要求单一主导声源可被清晰识别。这是服务于下游可分离性的过滤条件，在通用音频清洗中不常见。
  · 画外音源剔除：语音片段要求「语音与画内可见说话人时间对齐」，实质剔除了旁白、解说、画外音、后期配音——这在本工作中是硬性准入条件而非软过滤，比多数工作彻底。

【背景音乐/环境声的处理——保护而非剔除】
  这是本工作与多数 V2A 工作的重要差异：背景音不是要被清除的干扰，而是要被精确保留的「内容保持」监督信号。SAM-Audio（Shi et al., 2025）把目标实体的声音从原混合音轨中语义分离出来，编辑后的新声音再与保留的背景音无缝混合。因此 source 与 target 的背景音成分严格一致，构成听觉侧的差分对照基准。
  [不确定] 未说明 SAM-Audio 分离的残差/伪影如何处理——语义源分离在实践中会留下频谱空洞或分离残留，若 target 的背景音因分离-重混过程产生了细微失真，模型会把这种失真当作「正常」学进去。论文未讨论，也未见任何分离质量的验证或客观指标。

【SNR】[不确定] 未提及信噪比（SNR）阈值或任何传统信号级音频指标，全部依赖 Audiobox-Aesthetics 这一学习型综合评估。

### [2026 其他音视频联合生成](../models/JAVG_2026_misc.md) ⚠️

【ALIVE —— 用 MLLM 替代传统信号处理指标（本批最具方向性意义的做法）】
原文：「We utilize an MLLM model ... to perform audio filtering based on two criteria. Firstly, for audio quality, the model assigns a score to each audio, allowing us to discard samples with significant background noise. Secondly, for audio-visual coherence, we assess the correlation ...」
(1) 音频质量：由 MLLM 直接对每段音频打分，据此剔除背景噪声显著的样本——注意这里没有出现 SNR、PESQ、DNSMOS 等任何传统音频质量指标，完全交给多模态大模型「听」并判断。这是本批中最能代表 2026 年趋势的单点：音频质量评估从信号处理指标转向大模型语义判断。
(2) 音画一致性：评估相关性，把强相关样本单独归集，管控 BGM 等弱相关数据的占比「to optimize the dataset's composition」。
(3) 未提及：SNR 阈值、静音检测与静音占比、无音轨剔除（但管线第一步已筛选「videos with audio」，等价于剔除无音轨）、音源分离（source separation 完全未提及）、削波/失真检测[全部不确定]。
【NAVA —— AudioBox Aesthetic 作为音频质量算子】
音频质量过滤使用「AudioBox Aesthetic scores」（Meta 提出的音频美学评估模型，输出多维音频质量分）。这是本批中唯一点名使用专业音频质量模型的工作，比传统 SNR 更贴近人类感知。另有 YAMNet 音频分类做类别标注。阈值未公开[不确定]。未提及 SNR、静音、音源分离[不确定]。
【OmniCustom —— 仅格式统一，无质量过滤】
「We extract audio files from videos and unify them into 16kHZ」——音频统一重采样至 16kHz，无音质过滤描述[不确定]。其过滤全靠 SyncNet 双阈值（间接排除音轨异常样本：音轨损坏或静音会导致 SyncNet confidence 极低而被剔除）。这是一种「用同步检测隐式兼做音频有效性检测」的省事做法。
【StreamChar / CCL / Baton】未描述音频质量过滤[不确定]。CCL 使用 WavCaps 与 VGGSound 做音频预训练，二者自带质量筛选；Baton 使用 AudioCaps + WavCaps 同理。三者均把音频质量控制外包给上游。
【ITS-JAVG】推理侧用 ImageBind-TA 评估文本与生成音频的语义连贯性，非音质指标；无音质验证器——这本身是个缺口：其六个验证器中没有一个专门度量音频保真度/音质，可能导致搜索出语义对但音质差的样本[论文未讨论此点，属本调研观察]。
【总体判断】音频质量过滤在本批中最显著的变化是「去信号处理化」：ALIVE 用 MLLM 打分、NAVA 用 AudioBox Aesthetic（学习型感知模型），二者都绕开了 SNR 这类传统指标。同时七项中无一使用音源分离（Demucs 等）——反映主流共识是保留原生混合音轨，因为目标就是生成含语音+音效+音乐的完整音景。

### [音视频联合生成基线合集](../models/JavisDiT_baselines.md) ⚠️

音频侧过滤普遍缺失，只有格式化预处理：
【JavisDiT / JavisDiT++】明确声明音频预训练阶段「不采用任何数据过滤策略（no data filtering strategy）」，以确保最大化的文本到音频生成能力覆盖通用音效、音乐与语音三类——这是一个刻意的「不过滤」决策，与多数模型的严格音频筛选相反。音频侧只有三步格式化处理：截断到 30 秒以内、统一重采样到 16kHz、提取音频统计信息。无 SNR 阈值、无静音检测、无无音轨剔除、无背景音乐分离、无削波/失真检测[不确定]。反倒是音视频阶段有一项特殊的「音频内容过滤」——用 FunASR 剔除含人类语音的视频，但这是类别过滤而非质量过滤。
【Harmony】语音数据经「音视频一致性打分模型」筛选，这是跨模态一致性过滤而非纯音频质量过滤；音频质量层面（SNR、静音、噪声）未描述任何过滤[不确定]。音频编码走 MMAudio 的 VAE 编码器 + F5-TTS 的语音编码器双路。参考音频取 1–3 秒随机片段，未说明是否做有效性检查[不确定]。
【UniAVGen】只有格式化处理：音频先按 24,000 Hz 采样、转换为 mel 频谱，生成后用 Vocos 声码器还原波形。注意其 24kHz 采样率高于 JavisDiT/AV-DiT/MM-Diffusion 的 16kHz，带宽更宽、语音保真度更好——这与其聚焦真人语音的定位一致。无音频质量过滤描述[不确定]。
【MM-Diffusion / AV-DiT】无音频质量过滤。AV-DiT 的处理为「截断或补齐到 1.6 秒波形、16kHz 采样、转 mel 频谱 40×16×8」，属纯格式化。
【共性判断】本合集的音频质量把控主要依赖上游数据集（AudioCaps、Clotho、Emilia 等本身是精选语料），自身不建音频质检环节——这在小规模精选数据上可行，但在自采集的百万级数据（如 Harmony 的 200 万条自采环境音片段）上是明显的风险敞口。

### [可灵 3.0 / 可灵视频 3.0 Omni](../models/Kling_30_Omni.md) ⚠️

[不确定，同团队有明确公开规范] 可灵3.0 Omni 未公开音频质量过滤细节，仅有 Kling-Omni 的“音视频损坏检测”。Kling-Foley 的公开规范颇为完整，可作范式：音频统一标准化为 WAV / 44kHz / 16bit / 立体声；质量指标包含 SNR（信噪比）、MOS 分、削波比（clipping ratio）、音频带宽；用 VAD 做静音过滤，静音占比≥0.2 的样本被剔除；评测样本要求前景音中不含人声、且排除含人声的受版权保护背景音乐（即含BGM人声分离/剔除的处理）。[不确定：可灵3.0 Omni 是否剔除画外音/旁白源、是否做背景音乐分离]

### [LTX-2](../models/LTX-2.md) ⚠️

唯一且核心的音频侧过滤准则即「significant and informative audio components（显著且信息量丰富的音频成分）」——这实质上同时承担了无音轨剔除、静音/近静音剔除、低信息量音轨（如纯噪声、极低响度）剔除的职能，并被论文称为使子集获得「视觉与听觉内容平衡分布」的关键。
【未披露】该准则的量化实现：无 SNR 阈值、无响度/RMS 阈值、无静音占比阈值、无音频事件密度指标；未说明是否剔除画外音/旁白等非同期声源，未说明是否做背景音乐分离（source separation）或人声-伴奏分离。这是「思路明确但实现细节完全不可复现」的典型。技术侧仅知音频以 16kHz 立体声 mel 频谱进入 VAE、24kHz 输出，未说明对原始音轨采样率/声道数的准入要求。[不确定]

### [LongCat-Video](../models/LongCat-Video.md) ⚠️

基础版不适用。
Avatar 1.5 的音频侧过滤描述较粗：离线阶段有专门的音频标注环节，「verifies whether a sample contains usable speech conditions」（校验样本是否含可用的语音条件），即以「是否存在可用语音」为准入门槛；数据来源选择上偏好访谈类等「主体稳定、语音清晰」的素材，属源头质量控制。报告提及使用了 vocal separation（人声分离），可用于把人声从背景音乐/环境音中剥离，但未说明所用模型与参数。画外音源（音轨主人不在画面内）通过主动说话人检测间接剔除。
未披露的有：SNR 阈值、静音检测与静音占比阈值、无音轨样本的处理规则、音频采样率/码率要求。[不确定：SNR、静音占比等具体阈值]

### [MOVA](../models/MOVA.md)

覆盖较全，阈值公开：
【无音轨剔除】预处理第一步即剔除“缺少有效音频通道”（missing valid audio channels）的样本，以及解码失败样本。因此训练集中不存在无声视频。
【静音占比】Silence Ratio < 0.8（即静音超过 80% 的片段被剔除）。这个阈值相当宽松，允许大量静音存在，与“只保留语音片段”的策略配合后实际静音比例应远低于此。
【带宽】Bandwidth > 1,000 Hz，用于剔除严重带限、编码劣化或电话线路质量的音轨。
【感知质量】Audiobox-aesthetics 三维打分：PQ（Production Quality，制作质量，反映录音干净度与制作水准）> 5.0、CU（Content Usefulness）> 4.5、CE（Content Enjoyment）> 2.5。PQ 阈值定得较高，实质上偏好录音棚级/专业制作的音频。
【SNR】未使用显式信噪比阈值，噪声控制由 Audiobox-PQ 间接承担。
【背景音乐分离】未做源分离（source separation）。音乐作为混音的一部分保留在训练音轨中，并由 Qwen3-Omni-Captioner 在 caption 中描述（论文示例 caption 中就包含对背景电子乐的详细描述：合成器 pad、稳定节拍、小调旋律线、混音层级低不干扰旁白）。
【画外音/旁白剔除】未做剔除，且论文给出的完整示例本身就是画外音旁白 + 画面无说话人的广告片段，说明画外音样本被保留在训练集中。这可能是多说话人场景下 active-speaker 归属模糊问题的来源之一（论文 Limitations 承认该问题）。
【响度归一化】Phase 2 起引入 LUFS 响度归一化，目的是缓解 CFG 引起的响度爆炸（loudness explosion）——这是训练数据侧的处理，但动机来自推理侧的 CFG 行为。

### [合并条目：① Mochi 1](../models/Mochi_MAGI_Motif.md)

不适用。三者的数据管线在切分阶段即丢弃音轨（MAGI-1 与 Motif 的管线图中均只有视觉分支），无 SNR、静音检测与静音占比阈值、无音轨剔除、画外音源剔除或背景音乐分离等处理。

### [Movie Gen](../models/Movie_Gen.md)

· 静音处理：用 AED 打标后，直接丢弃 silence 为主导类（dominant class）的全部视频；未报告静音占比的具体阈值数值。
· 音频质量：训练了音频质量预测模型输出1~10的连续分（标注方式仿照 LAION aesthetic 的收集方式，10为最高）。关键设计是这个分数不作硬过滤阈值，而是写进caption成为可控条件——推理时对纯音效生成条件在质量分7.0、对音效+音乐联合生成条件在8.0。消融显示条件质量分越高预测质量分越高，但主观偏好在约6.5之后趋于饱和（含非画内音乐的样本则可继续受益到更高分）。
· 画外/无关声剔除：通过 CAVTP 低相似度识别并把非画内音分流出预训练主集；微调的 cinematic split 明确追求「抑制环境噪与无关的画外声音」的专业混音特性。
· 影视级音质定义：论文把影视原声与低端设备录音（手机、监控）的差距归纳为音质（how it sounds，需专业话筒录制 + 混音母带处理，去除pop噪、风噪）与声音设计（what sounds to include，突出与叙事相关的爆炸/对话事件，环境音乐做fade-in/fade-out混合）两个方面，用影视感分类器+AED自动筛+人工标注来弥合。
· 人声剔除：cinematic split 排除含vocals的片段。
· 视觉侧连带过滤：剔除OCR检出文字的、静态的、分辨率<480px的视频以降低视觉模态噪声。
· 未报告 SNR 阈值、也未做背景音乐分离（BGM separation）——相反，非人声音乐是刻意保留并建模的目标之一。

### [NVIDIA NeMo Curator](../models/NeMo_Curator.md) ⚠️

视频 pipeline 中无（不处理音轨，故不存在无音轨剔除、静音占比阈值、画外音源剔除、背景音乐分离等）。
独立音频 pipeline 中有面向语音数据的质量过滤：(1) WER/CER 过滤——以 ASR 转写错误率为代理指标剔除低质量音频，是主力手段；(2) 音频分析——时长计算（duration-calculation）与格式校验（format-validation）；(3) 26.02 起的 VAD（语音活动检测）可用于定位有效语音段、剔除静音；(4) 26.07 新增 SQUIM 客观质量指标（可无参考地估计 SNR、STOI、PESQ 等语音质量维度）与带宽估计（识别上采样的伪高采样率音频），并支持可选的二次 ASR 打分作为交叉验证。
【缺失】源分离（人声/伴奏分离）、背景音乐检测、音效事件检测、响度归一化等均无内置 stage；各指标的默认阈值文档中未给出。[不确定]

### [OmniHuman 数据集 + OHBench](../models/OmniHuman.md) ⚠️

音频治理位于流水线第二级，是四级过滤中唯一专门针对音频的一级：
【剔除条件（三类）】
- 音轨缺失（missing tracks）：无声视频直接淘汰，保证了数据集 100% 含原生音轨；
- 时长异常（abnormal duration）：音轨时长与视频时长不匹配、或超出预设范围的样本，通常对应封装错误、音轨截断等技术故障；
- 低质量（low quality）：论文指出判定依据包括静音占比（silence ratio）与音量阈值（volume thresholds），但具体数值均未给出。[不确定]
【源分离而非过滤：核心策略】用 Demucs 做四源分离，取人声（vocals）作为目标轨、其余三轨混合为背景轨。这一步不淘汰任何样本，而是把「背景音干扰」这个问题从「过滤问题」转化为「表示问题」——带 BGM 的样本不再是噪声，而成为可分别使用的双轨数据。这是本数据集在音频侧最关键的取舍，也是其能在 YouTube 这种 BGM 泛滥的数据源上维持百万级规模的原因（若采用 UniTalking 式的「有背景音就剔除」，保留率会崩塌）。
【画外音源的处理】不做显式的「画外音剔除」步骤，而是通过 SyncNet 归属匹配隐式实现——若一段语音无法匹配到任何画面中的视觉轨迹（旁白、画外音、后期配音），该样本因「所有主体须满足同步条件」而被淘汰。效果上等价于画外音剔除，但机制是归属失败而非专门检测。
【SNR】未提及信噪比指标或阈值。音量阈值可视为部分替代，但不等价。[不确定]
【标注侧的音频质量记录】个体层标注中含「语音质量标注」（speech quality annotations），即音频质量不只用于过滤，也作为可查询字段保留，下游可据此筛选高音质子集。
【采样率标准化】统一到 44.1 kHz。
【评测侧的音频质量指标】OHBench 全局层：Audiobox Aesthetics 四维度均值（AbS，含 content enjoyment、content usefulness、production complexity、production quality）、KL 散度、Fréchet Distance（FD）；个体层：DNSMOS 的 OVRL 分数（Speech Realism）、SenseVoice 计算的 WER（发音准确度）。用 Audiobox-Aesthetics 做音频美学评估是较新的选型，比传统的 FAD 单一指标信息量更丰富。

### [Open-Sora 系列](../models/Open-Sora.md)

不适用。pipeline 不读取音轨，转码统一为 H.264 视频流，无 SNR、静音检测、音轨缺失剔除或人声/背景乐分离等任何处理。

### [Ovi](../models/Ovi.md) ⚠️

音频侧过滤相对简洁，仅一条硬性门槛加若干隐式约束：
(1) 音量门槛：平均音量必须 ≥ −60 dB，与 SyncNet 双阈值同时施加。作用是剔除静音、近似静音、音轨损坏或音量极低的片段——即隐式完成了「无有效音轨剔除」。
(2) 无显式 SNR/噪声过滤：论文未描述信噪比阈值、噪声抑制、混响过滤、削波/失真检测[不确定]。
(3) 无背景音乐分离：未使用音源分离（如 Demucs）把 BGM 与人声拆开——事实上 Ovi 有意保留原生混合音轨，因为其目标就是同时生成语音+音效+BGM 的完整音景。
(4) 无画外音源剔除的显式模块：仅靠 SyncNet 低置信度间接过滤[不确定]。
(5) 静音占比阈值：只有「平均音量」这一全局指标，未见逐段静音占比统计[不确定]。
(6) 采样率与带宽约束（工程层面）：音频统一走 MMAudio 的 16kHz 编码器变体（STFT → mel 频谱 → 1D VAE latent），论文 Limitations 承认这条 16kHz 固定 1D-VAE 路径限制了带宽与空间感，高保真音乐、空间线索与细微音色会被压平，未来可换更高带宽 latent 或做后处理带宽扩展。推理时由 BigVGAN 声码器还原波形。

### [Script-a-Video](../models/Script-a-Video.md) ⚠️

论文未披露任何数据侧的音频质量过滤：无 SNR 门槛、无静音检测与静音占比阈值、无「无音轨则剔除」规则、无背景音乐分离（如 Demucs 类源分离）、无响度/削波/失真检测。[不确定]
【schema 层面的等效机制】MTSS 用两条标注规则部分替代了传统的音频过滤：
1) 画外音源的处理——严格音视频耦合原则使无视觉对应的声音无法成为 Event 事件，实质上完成了「画外音源剔除」这一目标，但采用降级而非剔除：这些声音被归入 Global 流的 global_audio 字段继续保留。这是一个「不丢信息、只降层级」的设计，比直接剔除更节约数据。
2) 背景音乐的处理——不做信号级源分离，而是做语义级分层：构成独立叙事事件的音乐进 Event 流（type=music），仅作为氛围铺底的背景音乐进 Global 流的 global_audio。这是用标注层级替代音频信号处理的思路。
【评测侧的音频质量指标】UTMOS（轻量语音质量 MOS 预测器）用于评估生成音频的语音质量，非用于数据过滤。数值见 sync_metric_and_threshold 字段。
【隐含的音轨要求】整套 schema 强依赖真实音轨（Event 流与 global_audio 均需从原始音频中抽取），因此训练数据必然自带原生音轨，但论文未把这一点写成显式的过滤规则。

### [Seedance 2.0 与 Seedance 1.5 pro](../models/Seedance_20_Seedance_15_pro.md) ⚠️

[不确定] 未披露 SNR 阈值、静音占比阈值、无音轨样本剔除策略、画外音源剔除、背景音乐分离（source separation）等任何具体做法。仅有评测侧定义可参考：SeedVideoBench 1.5 的「音频质量」指标衡量输出的内在声学质量（含人声与非人声成分），关键判据包括是否存在削波（clipping）、截断（truncation）等伪影、空间声场渲染、音色真实感与整体信号清晰度——这些判据很可能同时被用作训练数据的音频质量准入标准。Seedance 2.0 报告在局限性中承认仍存在「偶发音频失真」。

### [SkyReels 系列](../models/SkyReels.md) ⚠️

SkyReels-V4 的音频质量过滤是本条目中指标最齐全的部分，采用四项客观指标 + 一项活动检测：
(1) SNR（信噪比）——剔除底噪过大样本；
(2) MOS 分数（Mean Opinion Score，由自动 MOS 预测模型给出的主观质量估计）——剔除听感质量差的样本；
(3) 削波率（clipping ratio）——剔除过载失真样本；
(4) 音频带宽（audio bandwidth）——识别并剔除经低码率压缩、高频缺失的伪高采样率音频；
(5) VAD（语音活动检测）——要求静音占比低于 0.2，即静音不得超过两成；
(6) 平均音量门限 -60dB（与同步过滤联用）。
此外做时长归一：长音频切成15秒块，短音频按类别拼接至15秒。
【未披露】各指标的具体阈值数值（除静音比0.2与-60dB外，SNR/MOS/削波率/带宽的门限均未给出）；未提及背景音乐分离（source separation）或人声-伴奏分离；未说明画外音/旁白等非同期声是否剔除；未说明采样率与声道数的准入要求（已知音频以44.1kHz、5秒→218 latent token 的比例编码）。SkyReels-V2 无音频处理。[不确定]

### [Sora 2](../models/Sora_2.md) ⚠️

完全未披露。未提及信噪比阈值、静音检测与静音占比阈值、无音轨样本剔除策略、画外音/旁白源剔除、背景音乐分离（source separation）等任何手段。[不确定]

### [Step-Video-T2V](../models/Step-Video-T2V.md) ⚠️

不适用。数据 pipeline 不提取、不保留、不处理音轨，无 SNR 计算、静音检测、无音轨剔除、画外音源剔除或背景音乐分离等任何环节。[不确定]

### [UniTalking](../models/UniTalking.md) ⚠️

音频质量过滤集中在漏斗第二级，共三条规则，工具为 PANNs + SentenceASD，全部无阈值数值：
【静音剔除】剔除 muted 样本，即无有效音频的视频。未说明判定方式（能量阈值？全零检测？）与静音占比阈值——只做「是否静音」的二元剔除，未见「静音占比过高则剔除」的比例型判定。[不确定]
【无语音剔除】剔除 lack speech 的样本。这是本级最核心的规则，由 PANNs + SentenceASD 执行语音事件检测（speech event detection）。PANNs 是在 AudioSet 上预训练的 527 类音频事件分类 CNN，可给出「Speech」类的存在概率；SentenceASD 与之配合。与 UniVerse-1 用 Whisper ASR 做语音有无的二分类闸门相比，用专用音频事件分类器更轻量且更鲁棒（不依赖 ASR 能否转写成功）。注意此处的淘汰是彻底的——UniVerse-1 把无语音样本降级保留为「通用音视频数据」，UniTalking 则直接丢弃，这直接决定了两者截然不同的 domain 定位。
【低信噪比剔除】剔除 low SNR 样本。未给出 SNR 阈值、未说明 SNR 的估计方法（是用专门的无参考 SNR 估计模型还是信号统计）。[不确定]
【未做的音频过滤】
- 无音频美学评分（对比 MOVA 使用 Audiobox-aesthetics 的 PQ/CU/CE 三分数）；
- 无带宽（bandwidth）下界约束，即未剔除经过重压缩、高频缺失的音轨；
- 无削波、失真、爆音等录音缺陷检测；
- 无背景音乐分离（source separation）——背景音乐不做剥离，直接连同人声一起进入训练（如前所述，这被转化为可由 prompt 控制的属性）；
- 无混响/房间声学过滤；
- 无采样率下界约束。
【采样率与音频表示】原始波形经 STFT 转为 Mel 频谱后由 MMAudio 的 1D VAE 编码；推理时 latent 解码回 Mel 频谱，再由 BigVGAN 声码器合成 44.1 kHz 波形。44.1 kHz 的输出采样率高于 UniVerse-1 为迁就 25 fps 视频而下调的 25.6 kHz——UniTalking 因采用架构层面的 RoPE 对齐而非采样率对齐，得以保留完整音频带宽，这是其架构选择带来的一个附带优势。
【整体评价】音频质量把关的覆盖面（静音/无语音/低 SNR/画外音源）对说话人 domain 而言是够用的，工具选型（PANNs + ASD）也贴合任务，但完全不披露阈值使其不可复现；与 MOVA 的音频美学评分体系相比在精细度上仍有明显差距。

### [UniVerse-1](../models/UniVerse-1.md) ⚠️

音频侧过滤在漏斗中仅一级，且只用信号级手工特征，无任何学习型音频质量评估：
【音频活动检测（唯一的音频过滤环节）】分析三项信号级统计量——音量（volume）、能量（energy）、过零率（zero-crossing rate）——用于识别并剔除静音片段（silent segments）。论文未给出任何具体阈值数值。[不确定]
【无音轨剔除】漏斗第一步即丢弃缺少音轨的视频，是最前置的一票否决。
【未做的音频过滤】
- 无 SNR/信噪比阈值；
- 无音频美学评分（MOVA 用 Audiobox-aesthetics 的 PQ>5.0 / CU>4.5 / CE>2.5 三分数，UniVerse-1 完全没有对应设施）；
- 无带宽（bandwidth）下界约束；
- 无静音占比（silence ratio）的比例型阈值——只做「是否静音」的二元剔除，不做「静音占比过高则剔除」的连续型判定；
- 无画外音/旁白源剔除；
- 无背景音乐分离（source separation），语音片段中的背景音乐不做剥离，直接连同人声一起训练；
- 无削波、失真、爆音等录音缺陷检测。
【采样率处理】统一重采样至 25.6 kHz（自 Ace-step 原生的 44.1 kHz 下调），目的是与 25 fps 视频对齐而非质量考量——但这实质上降低了音频带宽上限（奈奎斯特频率 12.8 kHz），对高频细节（如镲片、齿音）有损失，属于为跨模态对齐付出的音质代价。论文未讨论这一取舍的影响。
【总体判断】音频质量把关是 UniVerse-1 数据流水线中最薄弱的一环，与其「音频专家基座来自音乐生成模型 Ace-step、音频能力主要靠基座继承」的整体思路一致——数据侧不做深度音质筛选，依赖基座已有的音频先验。

### [Unison](../models/Unison.md) ⚠️

论文未描述任何音频质量过滤环节：无 SNR/信噪比阈值、无静音检测与静音占比阈值、无无音轨剔除说明、无音频美学评分门槛、无带宽下界约束、无削波/失真/爆音检测、无采样率统一说明。
【唯一被明确执行的音频处理是分离而非过滤】Mel-RoFormer 把混合音频解耦为「高保真的语音与音效成分」（high-fidelity speech and sound-effect components）。这是预处理，不是筛选——所有样本都被处理，没有样本因音频质量被剔除。
【与背景音乐分离的关系——一处重要区别】需特别澄清：MOVA、UniVerse-1 等工作提到「背景音乐分离」时通常指「把背景音乐剥离丢弃，只保留人声」；而 Unison 的 Mel-RoFormer 分离目的完全相反——分离出的两路都要保留，都作为 ground-truth 参与训练。语音流学语音，音效流学音效，任何一路都不丢弃。这是「解耦以便分别建模」而非「分离以便净化」，是本工作数据处理哲学上与同类工作的根本差异。
【「high-quality audio segments」的措辞】论文用这一表述形容 5,000 万段音频语料，暗示存在质量筛选，但未给出任何筛选依据、指标或阈值。最可能是继承自上游数据集（WavCaps、AudioSet、YuE 等各自的质量标准）而非 Unison 自建。[不确定]
【评测侧使用但未用于过滤的音频质量模型】Audiobox-Aesthetics（按 Audiobox 协议）计算感知质量 PQ 与内容有用性 CU。Unison 在两项上均为 TI2AV 全场最优（PQ 6.34、CU 5.61，均优于 LTX-2 的 6.30/5.58）。与视觉侧类似，团队掌握音频质量评分工具却未用作数据过滤器——对比 MOVA 明确用 Audiobox-aesthetics 的 PQ>5.0 / CU>4.5 / CE>2.5 三阈值做训练数据过滤，Unison 在这一点上是明确的空白。
【分离残留的隐患】Mel-RoFormer 的分离并非完美，语音轨中残留音效、音效轨中残留人声是常态。这些残留会作为「错误的 ground-truth」进入双流监督，理论上会削弱解耦效果。论文既未评估分离质量，也未讨论残留的影响，更未设置分离质量门槛来剔除难分离样本。这是双流架构在数据侧最值得追问的一处。[不确定]

### [Veo 3 / Veo 3.1](../models/Veo_3_Veo_31.md) ⚠️

[不确定] 完全未披露。未提及 SNR 阈值、静音检测与静音占比阈值、无音轨样本剔除规则、画外音/旁白音源剔除、背景音乐分离（BGM separation）等任何具体手段。仅可从「训练视频按 quality 过滤」这一笼统表述推断音频质量应包含在内。

### [Vidu S1](../models/Vidu_S1.md) ⚠️

(1) 无音轨/音视频不完整样本在预过滤阶段即被「音视频完整性」检查剔除；
(2) 画外音源剔除：ASD 判定为 offscreen 的语音段（说话人不在画面中）被单独标出并区别处理；overlap（多人声重叠）clip 直接整条剔除；
(3) 背景音乐/歌唱剔除：观察到 diarization 模型在唱歌或强背景音乐场景下不稳定（人声被误分到 music stem、产生合成音色与失真伪影），故引入启发式规则丢弃「说话人在发声但语音能量占比过低」的片段；这实际上也起到了人声与背景音乐分离质量的兜底作用；
(4) 语音成分提取：训练前从原始音频中显式抽取 speech component。
未提及 SNR 阈值、静音占比阈值等具体数值 [不确定]。

### [主流视频预训练数据集合并调研：Panda-70M、InternVid、Koala…](../models/pretraining_datasets.md)

不适用。七者的pipeline均不读取音轨，无SNR计算、无静音检测与静音占比阈值、无无音轨剔除、无画外音源剔除、无背景音乐分离。Koala-36M的重切分代码（cv2.VideoWriter/mp4v）在重新切片时会**直接丢弃音轨**；Panda-70M（download_audio:True）与UltraVideo（声明保留native audio）虽保留原始音轨但不做任何质量检查。

## 语音/音效/音乐的分类与分别处理策略

`audio_type_handling` · 详细程度: brief

### [Allegro](../models/Allegro.md)

不适用。无语音/音效/音乐的分类与分别处理策略。

### [Apollo](../models/Apollo.md) ⚠️

Apollo 把「按音频类型分类并分别处理」提升为 pipeline 的一级阶段（Section 4.2 Audio-Guided Data Splitting），这是其数据方法论中最具辨识度的设计：
【两级树状分类】
- 第一级：vocal（含人声）vs non-vocal（不含人声）→ 后者构成 sound split（自然声/音效子集）。
- 第二级：从 vocal 子集再分为 singing（歌唱）/ single-speaker speech（单说话人语音）/ multi-speaker speech（多说话人语音）三支。
【分类器】论文未说明用什么模型做人声/非人声判别与说话人数判别（对比 MOVA 明确使用 EAT 自监督音频 Transformer + Silero VAD）。推测可用 VAD + 说话人分离（diarization）+ 歌唱检测组合实现，但未证实。[不确定]
【差异化处理策略】
- 语音与歌唱子集：做逐字转写 + 说话人属性抽取（性别、年龄）+ 音频 caption + 视频 caption。
- sound split：只做音频 caption，不做转写、不做说话人属性（「the sound split receives only audio captions」）。
即标注成本按子集的信息结构定制投放，避免对无人声片段做无意义的 ASR。
【音乐的位置】论文的四类枚举中没有单列「音乐（music/instrumental）」——纯器乐内容归入 sound split 还是被过滤掉，未说明；含唱的音乐归入 singing。这与 MOVA 在音频塔预训练阶段专门引入 JamendoMaxCaps 音乐语料的做法不同。
【下游用途】该分流直接服务于多任务训练：sound split 主要喂养通用 T2A/foley 能力，speech/singing split 主要喂养 TTS 与唇同步能力，评测也相应分为 audio、TTS、audio-video consistency 等组（Figure 5）。

### [CineDance / CineDance-1M](../models/CineDance.md)

【分类方式】在镜头级音频 prompt 中把非语音音频显式拆为 music（音乐）、ambient sound（环境音）、effects（音效）三类分别描述；语音/对白则走独立的 ASR 通道并进一步绑定到角色 anchor token；角色嗓音另有 character voice description 字段。整体形成「语音 / 音乐 / 环境音 / 音效」四类的并行标注结构。
【分别处理策略】
  · 语音：句级 ASR 转写 → 窗口化说话人-角色绑定（95.4% 准确率）→ 角色音色描述；训练时以 (spkᵢ,ℓ, speechᵢ,ℓ) 二元组形式注入 prompt；评测时用 Whisper-large-v3 计算 WER/CER。
  · 音乐/环境音/音效：统一走自然语言音景描述（audio prompt），不做进一步的类别标签化或分轨；评测时用 AudioBox-Aesthetics 的 PQ（Production Quality）、CE（Content Enjoyment）、CU（Content Usefulness）三分量衡量。
【设计取向】语音走「精确转写 + 身份绑定」的结构化路线，非语音走「自然语言描述」的柔性路线，二者处理范式明确区分。
【未做】未做音频源分离（把对白、音乐、音效分轨），也未对三类音频设置差异化的质量阈值或采样配比。

### [CogVideoX](../models/CogVideoX.md) ⚠️

[不确定] CogVideoX 视频侧不适用。CogSound 从公开描述看是「单一模型生成多类音频」的形态——可生成爆炸、水流、乐器、动物叫声、交通工具声等复杂音效，并可生成节奏等音乐元素，未见按语音/音效/音乐分模型或分数据流处理的说明，也未见各类别的分类器、配比与差异化处理策略。

### [Cosmos-Predict2.5](../models/Cosmos-Predict25.md)

不适用。不存在语音/音效/音乐的分类与分流处理。该 pipeline 中与之地位相当的「类型分流」机制是纯视觉的内容类型分类：内部训练的 content type classifier 按 26 类自建 taxonomy 给每条 clip 打标，并作为 sharding 的首要轴用于域均衡采样；以及五大 Physical AI 域的独立 pipeline 分流。

### [Data-Juicer 2.0](../models/Data-Juicer.md) ⚠️

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

### [Foley-Omni](../models/Foley-Omni.md)

语音/音效/音乐的三分类是贯穿整个系统的组织主线，不只是清洗环节的分类，而是任务定义、标注schema、验证工具、训练课程、评测基准五个层面共用的同一套划分。
【分类工具双路】
  · 语义判别路径：Gemini 2.5 Pro 多模态判断三类组件各自是否 physically present（detect-then-describe）。
  · 声学验证路径：Bandit 源分离得到三个 stem，按 −35 dB RMS 能量门控做二次确认与否决。
  Bandit 的选型与三分类严格同构（cinematic audio source separation 天然分 speech/effects/music），避免了通用源分离模型类别不对应的问题。
【分别处理策略】
  · 标注层：三类各占一个独立字段 [WORDS]/[AUDIO]/[MUSIC]，可空。
  · 数据层：三类对应三个单任务组 TTS(1,253h) / TTA(912h) / TTM(139h)，并有对应的视频条件版本 VisualTTS(1,980h) / V2A(403h)（音乐无视频条件单任务组）。
  · 训练层：Stage 1 先在三类单任务上分别打基础（约0.7M文本-音频对，5 epochs），Stage 2 引入视觉条件（V2A + VisualTTS，3 epochs），Stage 3 才做三类联合生成（V2ST，2 epochs）。即先分后合的课程设计。
  · 生成层：三类不是分别生成后混音，而是在共享隐空间联合生成——分类用于条件控制与数据组织，不用于推理时的模块拆分。
  · 评测层：V2ST-Bench 按共存组合分三档（语音+音效、语音+音乐、三者齐全），并用不同指标侧重不同类别（WER 测语音、CLAP/IB 测音效语义、A-MOS 综合）。
【配比失衡】三类投入极不均衡：语音相关约66%、音效相关约27%、音乐仅2.8%（139小时TTM，且无独立的视频条件音乐任务组）。这是明显的取舍——论文的核心卖点是「speech intelligibility」的提升，音乐能力相对是配角。

### [Goku](../models/Goku.md)

不适用。无语音/音效/音乐的分类与差异化处理策略。Goku 在音频维度是完全空白的，其对本调研的价值集中在纯视觉侧的数据分布均衡与分辨率分档阈值体系上。

### [HunyuanVideo-Foley](../models/HunyuanVideo-Foley.md) ⚠️

【分类而不淘汰：本工作的核心处理原则】语音、音乐、音效三类音频在本工作中的处理方式是「全部检测、全部打标、全部保留、按标签配比」，与同类工作的「按类型淘汰或分流」形成明确差异。
【分类工具（流水线第 7 级）】两级串联：(1) 语音-音乐检测模型（speech-music detection），做 speech / music / 其他的粗粒度三分；(2) 音频分类模型（audio classification），做更细粒度的声学事件分类。两者均未点名具体模型，类目体系与类目数量均未披露。[不确定]
【三类音频在本任务中的角色】
- 音效（Foley/事件音）——模型的核心目标输出，是全部数据处理工作真正服务的对象；
- 语音——非目标输出但必须被识别。理想情况下模型应学会在有人说话的画面上保持人声空缺、只生成环境与动作音。保留适量语音样本（而非全部剔除）有助于模型见过含语音的真实声学场景、学会声学分层；但保留过量则会导致模型倾向于生成含糊的类语音声（babble），严重损害可用性。这一平衡点的把握正是「类别分布管理」要解决的关键问题，可惜论文完全未披露实际配比与效果。[不确定]
- 音乐——同样非目标输出。背景音乐在网络视频中极为普遍，若不控制其占比，模型会倾向于给任何画面配上音乐而非音效。检测出音乐后只打标签不做音源分离，说明团队选择了「用配比控制」而非「用信号处理剥离」的路线，成本更低但纯度更差。
【「分类 + 分布管理」这一路线的方法论价值】与 MOVA（只保留语音、其余丢弃）、UniVerse-1（按语音有无二分流、非语音降级保留）相比，本工作是三者中唯一把音频类型作为「可调节的配比变量」而非「保留/淘汰的判据」来使用的。理论上这是更精细的做法——不损失数据多样性，通过采样权重调节各类占比，且配比可以在不重新清洗数据的情况下调整。但这一优势完全依赖于配比策略的合理性，而论文对配比策略只字未提，使得该设计的实际价值无法评估。
【模型侧的类型区分能力】文本条件走 CLAP 编码器注入，用户理论上可通过 prompt 描述期望的声音类型来影响输出，但论文未评测模型对「只要音效不要音乐」这类负向指令的响应能力。

### [HunyuanVideo](../models/HunyuanVideo.md) ⚠️

不适用。无语音/音效/音乐的分类与分别处理。[不确定]

### [InstructAV2AV](../models/InstructAV2AV.md) ⚠️

音频按「语音 vs 非语音声源事件」二分，两类走完全独立的处理链路，这个分叉贯穿准入过滤、指令生成、音频合成三个阶段，是pipeline中最清晰的分类驱动设计。

【分类的判定与依据】论文未明说使用了哪个分类器做语音/非语音判别，从工具链推断由 TalkNet（主动说话人检测）的输出隐式决定——检测到画内说话人即走语音路，否则走事件路。[不确定] 分类的具体实现与两类的样本比例均未披露。

【语音路的处理链】
  准入：TalkNet 定位主动说话人 → Scribe 提取精确语音时间戳 → 硬性条件「语音须与画内可见说话人时间对齐」，否则丢弃。
  指令：Qwen3-Omni 生成语音编辑指令，目标台词以 <S>...<E> 标记包裹嵌入指令。
  合成：ElevenLabs 语音合成，按子集类型分为保持原音色改内容（identity-preserving speech modification）、克隆新音色（clone_voice）等模式。
  评测：Sync-C / Sync-D（SyncNet 唇同步）针对性度量。

【非语音事件路的处理链】
  准入：剔除声源模糊片段 → 为每个保留片段赋予唯一的语义声音事件标签（如 "dog barking"），基于其主导声源。
  指令：Qwen3-Omni 基于实例 mask 与声音事件标签生成实体编辑/插入/移除指令。
  合成：SAM-Audio 分离目标实体声 → ElevenLabs 文本转音频合成新声音 → 与背景音混合。
  评测：PEAVS（通用事件同步）、AV-A（跨模态语义）针对性度量。

【音乐的缺席】[不确定] 论文未把音乐单列为一类处理对象。这与 Foley-Omni 的 speech/effects/music 三分法形成对比。推测音乐（来源中电影素材普遍带配乐）被归入 SAM-Audio 分离后的「背景音」原样保留，即音乐是被保护的背景而非被编辑的对象——这与任务定义一致（没有「把配乐从悲伤改成欢快」这类编辑任务）。但论文未明确说明，也未讨论配乐的存在是否干扰 SAM-Audio 对目标实体声的分离。

【二分 vs 三分的取舍评价】本工作的二分法是任务导向的：编辑对象只可能是「说话的人」或「发声的物体」，音乐没有对应的视觉实体可供 mask 定位，因此天然不在编辑范围内。这个划分比 Foley-Omni 的三分法更简，但对本任务是充分的。

### [2026 其他音视频联合生成](../models/JAVG_2026_misc.md) ⚠️

【ALIVE —— 顶层 speaking/non-speaking 二分驱动全局（本批最系统）】
(1) 数据组织的最顶层就是音频类型二分：「First, we make a top-level distinction between core scenarios: 'speaking' and 'non-speaking' scenario.」——先按「有无人说话」把语料劈成两半，再在其上叠加九大领域的三级标签。这个顺序很关键：说明音频类型是比视觉领域更高优先级的组织维度，符合 AV 模型的本质需求（说话场景需要唇同步与音色一致，非说话场景需要事件级音画对齐，二者的建模难点完全不同）。
(2) caption 层面对应分流：<W> 标记逐字语音、<I> 标记非语音声学事件、Subjects 中的 acoustic profiles 描述音色属性。
(3) BGM 单独处理：作为「弱相关」音频的代表，其占比被主动管控而非剔除。
(4) 音频塔训练的两阶段类型切换：Stage I 纯语音（700k hours 转写语音，1 epoch）→ Stage II 语音+真实音景混合（5k hours 高质语音 + 111k hours 视频音轨，10 epochs）。
(5) Level-1 九大领域中 Home Sounds 与 Sound Effects 两类专属非语音音频，且每个 Level-3 视觉标签绑定音频关键词——视觉概念与声音概念成对索引。
【NAVA —— 五类音频标签（本批分类最细）】
YAMNet + omni-modal tagger 划分五类：single-speaker speech、multi-speaker speech、ambient sound、music、singing。相比常见的「语音/音效/音乐」三分，这套五分法的两处细化很有针对性：
- 单人语音 vs 多人语音分开：直接服务于其多说话人 timbre 控制能力（Timbre-in-Context Conditioning 需要知道有几个说话人、各自的 speech span 在哪）
- music 与 singing 分开：歌唱同时具备语音（有歌词、有音色）与音乐（有旋律、有节奏）双重属性，单列一类避免污染纯语音与纯音乐的建模
caption 音频板块也按 speech / SFX / music / ambient sound 四类描述。训练阶段用「audio-only : audio-visual」比例调度（Stage 1 为 3:1、Stage 2 为 1:2）间接控制音频类型的曝光。
【OmniCustom —— 单一类型（纯语音）】全部为单人说话语音，无音效/音乐处理。音频 caption 按说话人属性组织（age/gender/accent/pitch/prosody/emotion/speaking rate）。是本批中音频类型最单一的一项，但也因此其音色控制做得最深。
【StreamChar —— 语音为主】Emilia（纯语音多语种数据集）做编排器预训练，联合训练数据为角色说话视频。无音效/音乐类型设计[不确定]。
【CCL —— 预训练通用音频、联合训练偏语音】音频扩散预训练用 WavCaps（音频描述）+ VGGSound（视频音效），覆盖通用音效；联合训练数据（OpenHumanVid + 访谈/短剧/电影）以对白为主。形成「通用音频打底 + 语音场景微调」的结构。其多任务概率分布中 text-to-audio 占 0.1、video-to-audio 占 0.15，说明纯音频生成能力被显式保留。
【Baton】AudioCaps + WavCaps 提供通用音效/音景，OpenHuman-Vid 提供人声，无显式类型分流策略[不确定]。
【ITS-JAVG】所测两个基座音频类型侧重不同：MMDisCo 基于 VGGSound（音效为主）、JavisDiT 覆盖更广；其验证器中 ImageBind-TA 与 AVHScore 对语音与音效一视同仁，未做类型分流[不确定]。

### [音视频联合生成基线合集](../models/JavisDiT_baselines.md) ⚠️

语音/音效/音乐三类音频的处理策略是区分本合集各家定位的关键：
【MM-Diffusion / AV-DiT —— 二分且互不混合】Landscape 对应「环境音效」、AIST++ 对应「音乐」，两个数据集分别独立训练两个模型（仓库中 Landscape.pt 与 AIST++.pt 是两套独立权重），不做混合训练。完全不涉及语音。这种「一域一模型」的做法是小数据时代的必然选择。
【JavisDiT / JavisDiT++ —— 音频阶段全类混合，视频阶段刻意剔除语音】
- 音频预训练（78 万条）：明确追求全类覆盖，「不做任何过滤以确保最大化的 T2A 能力，覆盖通用音效（general sound）、音乐（music）与语音（speech）三类」。数据源的类型分工清晰：AudioSet/AudioCaps/VGGSound/WavCaps/Clotho 提供通用音效，GTZAN/MusicInstrument 提供音乐，UrbanSound8K/ESC50 提供环境音与城市噪声，MACS 提供声学场景。
- 音视频 SFT：用 FunASR 剔除大部分含语音的视频——即在联合生成阶段主动放弃语音类别。这是本合集中最明确的「音频类别取舍」决策，其代价是模型不具备唇同步与对白生成能力，收益是把有限的算力与数据全部用于环境音/音效的事件级对齐（也正是 JavisBench 的重点评测方向）。
【Harmony —— 语音与环境音严格 1:1、且分别配备不同编码器】
- 数据侧：语音 200 万条 vs 环境音 200 万条，1:1 混合，阶段一与阶段三均维持该比例。
- 架构侧分型：音频 VAE 编码器用 MMAudio（擅长通用音频/音效），语音编码器用 F5-TTS（擅长语音）——两类音频走不同编码路径，是本合集中唯一在编码器层面区分音频类型的工作。
- 训练侧分型：阶段二音色解耦对两类数据用不同配对策略（语音用同说话人跨话语配对，环境音用同片段非重叠段配对）。
- 评测侧分型：Harmony-Bench 三档子集分别对应环境音、语音、二者共现。
可以说 Harmony 是本合集中唯一系统性地把「语音」与「非语音」作为两条平行数据流全程分别处理，同时又坚持二者必须联合训练的工作。
【UniAVGen —— 语音单一路线】阶段一在 Emilia（纯 TTS 语料）上预训练，全流程聚焦真人语音，音效与音乐的处理未见描述[不确定]。这使其在配音、唇同步、音色/情绪一致性上有优势，但通用音效生成能力存疑。

### [可灵 3.0 / 可灵视频 3.0 Omni](../models/Kling_30_Omni.md) ⚠️

[不确定，同团队有公开范式] Kling-Foley 明确先用音频分类模型将素材分为音效（sound effects）、音乐（music）、语音（speech）、演唱（singing）四类，再针对每一类分别调用音频理解大模型生成描述、分别适用不同的标注与过滤规则，最后统一融合。KlingAvatar 2.0 也按中文语音/英文语音/演唱分别构建数据与评测。可灵3.0 Omni 的能力覆盖对白语音、环境音、音效三类且各自可控（可分别指定），强烈暗示训练侧存在同样的分类-分治处理，但官方未公开具体分类器与配比。

### [LTX-2](../models/LTX-2.md) ⚠️

在 caption 标注层面对四类音频做了显式区分与分别描述——语音（精确转写 + 说话人/语言/口音）、音乐（music）、环境音/氛围（ambient、background）、拟音音效（foley），这是模型能分别响应四类音频指令的基础。
但在数据过滤与训练配比层面，论文未说明是否对四类分别设计过滤规则、配比目标或损失权重——「音频信息量筛选」是一个统一的门槛，未见按类型分流处理。架构上音频是单一 5B 流统一建模，无按类型分支或专家路由。
间接证据显示存在类别不均衡：模型卡 Limitations 明确指出「生成非语音内容时音频质量较低（produce lower-quality audio when generating non-speech content）」，说明语音类数据在子集中占优，音效/音乐类相对不足。论文亦承认深层文本理解主要是为了服务语音的音素准确性，重心明显偏向语音。[不确定]

### [LongCat-Video](../models/LongCat-Video.md)

基础版不适用。
Avatar 1.5 按功能对音频做了粗粒度分流处理而非严格的语音/音效/音乐三分类：(1) 说话语音为主线，走完整的唇同步 + 主动说话人检测链路；(2) 歌唱/音乐单列一支（音乐视频来源），用于覆盖歌唱口型与节奏性肢体动作，与常规说话的口型动态特性不同；(3) 静默（无说话）单独构建子集，用双模型一致性判定保证「确实没在说话」，让模型学会音频存在但主体不发声的情形；(4) 多人并发说话区间被剔除，不作为一类保留。音效/foley 与环境音未作为独立类别处理——这与其定位一致：音频只是驱动条件，模型不生成音频，故无需覆盖完整音景类别。

### [MOVA](../models/MOVA.md)

分类工具与分流策略明确，且贯穿数据、训练、能力三个层次：
【分类工具】EAT（Efficient Audio Transformer，自监督预训练音频模型）作为音频分类器，对片段打 speech / non-speech 相关标签。语音子集的构造条件为：EAT-contained-Speech 与 EAT-contained-Singing 两个标签均判为 True（或满足模型正类置信度）。另有 Silero VAD 在预处理阶段做语音/非语音区间划分。
【分流目的】论文明确说明分流是“depending on the target capability (e.g., lip synchronization vs. general foley/ambience modeling)”——即为“唇同步”与“通用 foley/环境音建模”两种目标能力分别构建子集。
【实际策略：两段式分工】
- 音乐与音效能力在 1.3B 音频塔的预训练阶段注入：通用音效来自 WavCaps + VGGSound，音乐来自 JamendoMaxCaps，语音来自 in-house TTS。
- 语音与唇同步能力在联合训练阶段强化：联合训练最终只使用语音片段（占预处理片段 69.47%）。
【标注侧的分流】音频标注同样按类型分工——Qwen3-Omni-Instruct 只做语音转写（严禁包含非语音与音乐），Qwen3-Omni-Captioner 只做非语音音效与音乐描述；融合时非语音内容被限定在四个类目（环境音、音乐、音频主题/声源、结构性音频变化）且严禁出现人声与词语。
【代价】这一“重语音、轻音乐”的配比在 Limitations 中被明确承认为局限：模型在歌声、复杂声音纹理、音乐/器乐内容上表现退化，因为音频塔仅 1.3B、容量不足以承载精细的音高/谐波结构与长程时域依赖。此外论文还指出模型缺乏物理声学推理（如闪电与雷声之间的传播延迟未被显式建模或数据强制）。

### [合并条目：① Mochi 1](../models/Mochi_MAGI_Motif.md)

不适用。无语音/音效/音乐的分类与分别处理策略。

### [Movie Gen](../models/Movie_Gen.md)

分类：AED（AudioSet 527类本体）把每个样本映射到 voice（speech + singing 子类）、non-vocal music（music 子类）、general sound（其余子类）三类，允许多标签共存；与 diegetic/non-diegetic 交叉后得到表23的六格（如画内voice=现场说话、画外voice=旁白；画内music=现场演奏、画外music=背景配乐；画内sound=海浪拍打、画外sound=罐头笑声与riser）。
处理策略：
· 建模上不分模型——论文明确选择「用单一模型联合生成所有音频类别」，理由是不仅视频与音频之间存在相关性，不同音频类别之间也彼此相关（若分开生成，音效与音乐无法和谐混音）。
· 数据配比上分类别调阈值与配额：general sound 被列为最高优先级（低层物理规律最难学、错误最刺耳），量级上比其他类别高约10倍；music 类的 diegetic 判定阈值抬高到0.3；voice 类被排除出生成目标（仅作为存在性标签）。
· caption 上分字段控制：speech/singing 用二值标签、music 用连续概率、一般音效用自由描述、音乐风格单独一段且无条件追加。
· 微调上分数据源：影视级音视频（同时含画内音与画外的环境音乐、主题音乐）+ 不带视频的高质量纯音乐 O(10)K 小时与纯音效 O(10)K 小时，后者数量更易获得、用于拉高音质。消融证明加入大规模文本-音效与文本-音乐配对数据能让模型有效解耦不同音频类型。

### [NVIDIA NeMo Curator](../models/NeMo_Curator.md) ⚠️

无分类分流机制。音频 pipeline 的隐含假设是「输入即语音数据」，全部 stage（ASR 转写、WER/CER 过滤、VAD、说话人分离、SQUIM 质量评估、标点准备）均为语音服务，不区分也不分别处理音效（foley）、音乐与环境音。框架未提供音频事件分类器（如 AudioSet 标签体系）、音乐检测器或源分离模块。
26.07 的音频 tagging stage 是最接近的能力，但文档中其定位是语音属性标注而非四类音频的分类分流。[不确定]

### [OmniHuman 数据集 + OHBench](../models/OmniHuman.md) ⚠️

【核心机制：Demucs 四源分离后的二元重组】原始混合音轨经 Demucs 分离为四源（vocals / drums / bass / other），随后重组为两条轨道：人声轨（目标）与背景轨（其余三源混合）。这是本数据集处理音频类型的基础设施。
【语音（speech）】作为目标轨单独提取，并施加完整的处理链：3D-Speaker 分离 → FunASR-Nano 转写 → SyncNet 归属匹配 → 情绪标注 → 时间戳 → 语音质量评分。语音是标注最厚的音频类型。
【音乐（music）】不剔除，改为标注三个属性：类型（type）、情绪/氛围（mood）、相对音量（relative volume）。「相对音量」使音乐与人声的能量比成为可筛选维度。音乐信号本身完整保留在背景轨中。
【背景声/环境音（background sound）】视频级标注中的 background sound labels 字段，具体标签体系（是自由文本还是封闭类目集）未说明。[不确定]
【音效（sound effects）】在第一阶段属性抽取的清单中明确列出（appearance / motion / expression / speech / music / sound effects），说明 Qwen3-Omni 会抽取音效信息；但未见对应的独立标注字段说明，其最终是并入 caption 叙事还是单列字段不明。相比 Foley-Omni 的三字段专门设计，音效在此处是次要维度——这与数据集的人物中心定位一致。[不确定]
【静音】作为低质量条件之一被过滤（静音占比阈值），不作为独立类别保留。
【各类型的配比】未给出任何时长占比统计。[不确定]
【与两条既有路线的对比】UniTalking 是「只留语音、其余全部剔除」（纯化路线）；UniVerse-1 是「按语音/非语音分桶并控制比例 15.4% : 84.6%」（配比路线）；OmniHuman 是「不剔除不分桶，用源分离拆解后分别标注」（解耦路线）。第三条路线在数据利用率上最高、对下游最灵活，代价是把配比的责任完全转移给使用者——数据集本身不对音景构成做任何承诺。

### [Open-Sora 系列](../models/Open-Sora.md)

不适用。无语音/音效/音乐的分类与差异化处理。

### [Ovi](../models/Ovi.md) ⚠️

Ovi 的核心主张之一是「必须用统一音频模型同时覆盖语音与音效」，而非分而治之，但在数据组织上仍按类型分别处理：
【数据来源分型】语音主要来自内部语料（预训练主体）；音效来自 VGGSound / AudioSet / WavCaps（微调期引入）；音乐未见独立来源，主要以视频原生音轨中的 BGM 形式出现[不确定]。
【训练阶段分型】音频预训练以语音为主（学说话人身份、音高、情绪、韵律）；音频微调阶段引入多样音效，同时掺回内部音视频的音轨以保活 TTS 能力，使 Ovi-Aud 成为可同时做 T2A 与 TTS 的统一音频基座。
【打标分型】按「含语音 / 不含语音」二分：含语音者音频描述聚焦说话人声学属性；不含语音者聚焦 sound effects / background audio / musical elements 三类。纯音频数据中若无语音则转写字段留空。
【条件编码上不分型（关键决策）】早期版本曾用 CLAP 编码非语音描述、T5 编码语音转写以「解耦 T2A 与 TTS」，消融证明该分离方案会导致模型无法把音效与语音融合成连贯音轨，最终改为全部合并进单一 T5 嵌入。
【评测分型】T2A 侧按 MMAudio 协议报 FD_PANNs 18.03 / FD_VGG 5.02 / IS 11.20 / CLAP 0.224；TTS 侧在 Seed-TTS test-en 上报 WER 0.035。二者均接近各自领域专用模型（如 MMAudio-L 的 FD_PANNs 15.04、F5-TTS 的 WER 0.018），论文认为统一模型不必超越专用模型，能同时胜任才是音视频融合的前提。

### [Script-a-Video](../models/Script-a-Video.md) ⚠️

音频类型的分类与差异化处理是 MTSS 中定义最清晰的部分之一，采用「三类事件 + 一类全局」的四层划分：
【dialogue（对白/语音）】
- 归属：Event 流；
- 专属字段：speaker（关系绑定到 Reference ID，明确「谁在说」）、line（逐字台词文本）；
- description 字段额外承载情绪起伏（emotional shifts）与发声技巧（vocal techniques）等副语言信息；
- 是三类中字段最丰富的一类，反映其在音视频生成中的核心地位（唇同步与身份绑定均依赖它）。
【sfx（音效）】
- 归属：Event 流；
- 专属准入约束：必须由画面中可见的主体产生（"sound effects must be generated by a visible subject"）——这是三类中唯一带硬性视觉依据要求的类别，直接服务于 foley 类生成对「声音-动作因果关系」的需求；
- 无 speaker 与 line 字段，仅有 description。
【music（音乐）】
- 归属：Event 流（当音乐构成独立的叙事事件时）；
- 与 global_audio 中的背景音乐形成分层：前者是有明确起止、参与叙事的音乐（如画面中有人演奏、或作为情节点的配乐进入），后者是全片铺底、不构成独立事件的氛围配乐。
【global_audio（全局音频，第四层）】
- 归属：Global 流；
- 承载：不够格成为独立事件的环境音（ambient sounds）与背景音乐（background music），以及被过滤下来的无关背景噪声（irrelevant background noise）；
- 定位：兜底容器，保证音景信息不丢失的同时不污染 Event 流的信噪比。
【并发音源】拆分为并行的 event 条目，每条独立带 type、time_range 与内容块，不做混合描述。这使得「一个人说话的同时旁边有脚步声」这类场景可被完整且可分别控制地表达。
【生成侧的差异化验证】论文分别用 WER（对白内容准确性）、UTMOS（语音质量）、SyncNet（唇同步）三个指标评估语音相关能力，用定性的 RMS 包络分析评估音效——Figure 7/8 显示基线产生平坦的环境噪声型包络，MTSS 方案则呈现自然语音的节奏性起伏与动作驱动事件的周期性冲击（如脚步声的周期性 impact）。论文明确指出 Event 流把语音与音效生成「从近乎随机转变为语义准确」。
【数据侧配比】三类音频在训练数据中的比例未披露，也未提及按类别调整采样权重。[不确定]

### [Seedance 2.0 与 Seedance 1.5 pro](../models/Seedance_20_Seedance_15_pro.md) ⚠️

训练侧的分类与分别处理策略未披露，但产品与评测两端均体现出清晰的三类划分：Seedance 2.0 支持背景音乐（BGM）、环境音效（ambient sound effects）、角色人声旁白（character narration/voiceover）三类音轨的多轨并行输出并各自与画面节奏精确对齐，这在架构上要求训练数据具备按音轨类型分离或分别标注的能力。评测标签体系亦按此划分：SeedVideoBench 1.5 将音频分为人声类型、人声属性、非语音音频（音效+音乐，再按声源、声学属性、音乐流派、技术参数细分）；SeedVideoBench 2.0 的 17 类中语音类（方言/多人对话/综艺/戏曲/英语/少数民族语言/歌唱说唱/画外音/非言语人声）、音效类（人声+动作交互音/物体交互音/动物叫声/环境背景音/特效 ASMR）、音乐类（乐器与音频）、空间类（空间场景音/双声道音频）界限分明。[不确定：训练数据侧的实际分类器与处理分支]

### [SkyReels 系列](../models/SkyReels.md) ⚠️

SkyReels-V4 采取「先分类、再按类分流处理」的清晰策略，是其音频侧方法论的骨架：
(1) 分类：由 Qwen3-Omni 将音频判为音效（sound effects）、音乐（music）、语音（speech）、歌唱（singing）四类；
(2) 分流处理差异：语音与歌唱 → 走 Whisper 转写，转写文本进入 <dialogue> / <singing> token，并需通过 SyncNet 唇音同步过滤；音效与音乐 → 不转写，由 Qwen3-Omni 生成描述性 caption，进入 <sfx> / <bgm> token；
(3) 时长拼接按类别进行——只有同类音频才允许拼接凑满15秒，避免跨类别混杂造成语义污染；
(4) caption schema 的四个听觉 token 与四分类严格对应，使模型在生成时可按类别独立控制。
【未披露】四类各自的样本量与训练配比、是否对不同类别设置不同的质量阈值、损失权重是否分类别加权。已知音频主干预训练数据「以语音为主」，四类天然不均衡。SkyReels-V2 不涉及音频。[不确定]

### [Sora 2](../models/Sora_2.md) ⚠️

完全未披露。模型输出侧明确区分对白、音效/foley、音乐、环境音四类，但训练数据侧是否对这四类做显式分类并分别设计过滤/配比/标注策略，OpenAI 未做任何说明。[不确定]

### [Step-Video-T2V](../models/Step-Video-T2V.md) ⚠️

不适用。无语音/音效/音乐的分类与分别处理策略。阶跃星辰的语音侧能力封装在独立的 Step-Audio 模型中，与本条目的视频数据 pipeline 无交集。[不确定]

### [UniTalking](../models/UniTalking.md)

UniTalking 对三类音频采取的是「只要语音、其余淘汰」的极端策略，不做分类保留、不做分别处理：
【语音（speech）】唯一保留的类别，且经过四重条件筛选：非静音 → 含语音（PANNs + SentenceASD）→ SNR 达标 → 声源在画面内（LightASD）→ 唇音同步（LipSync）。最终 230 万条样本全部为人声说话内容。
【音效（foley）与环境音】不作为独立类别保留。仅含环境音而无语音的样本在第二级即被「lack speech」规则淘汰。这与 UniVerse-1「无语音样本降级为通用数据保留 3,074 小时 + 额外引入 3,422 小时 VGGSound/AudioSet 补强音效」的做法完全相反。副语言音效（如笑声）作为语音样本的伴随成分被保留（Figure 4 展示了「a short laugh」的可控生成）。
【音乐】不作为独立类别处理，无音乐数据集引入，无音乐能力的专门建设。背景音乐若伴随语音出现则被动保留，且未做源分离，转而由 caption 如实描述并成为可由 prompt 控制的属性（「without any background music」可有效生效）。
【分类机制】只有一道二分类闸门（有无语音），无多类音频分类器（对比 MOVA 使用 EAT 自监督音频 Transformer 做 speech/non-speech 分类）。非语音内部不做任何细分。
【差异化处理】不存在按音频类别的差异化损失或采样权重（对比 UniVerse-1 对 VGGSound/AudioSet 施加 LQLS 的按来源差异化损失）。全部数据同等对待。
【能力边界的直接后果】模型的音频能力完全集中于语音合成与音色克隆，不具备音效与音乐生成能力。论文结论中主张「框架可推广到通用音视频合成，包括音效与音乐」，但这一主张既无数据支撑（数据中不含此类内容）也无实验验证，属纯粹的展望。
【与阶段一数据的关系】音频分支的基础发音能力来自阶段一在「内部 TTS 数据」上的预训练，这同样是纯语音数据——即模型从预训练到联合训练全程只见过语音，音频能力的单一性是贯穿两阶段的一致设计。

### [UniVerse-1](../models/UniVerse-1.md)

语音、音效、音乐三类音频的分类与处理策略如下：
【分类机制】仅一道二分类闸门——Whisper ASR 检测片段中是否含有可识别语音：
- 检出语音 → 进入语音子集，继续接受 RetinaFace 人脸检测与 SyncNet conf>2.0 核验，通过后打上「含语音」显式标签，最终 1,187 小时；
- 未检出语音 → 不淘汰，降级归入「通用音视频数据」，最终 3,074 小时。
未使用任何专用音频分类模型（对比 MOVA 使用 EAT 自监督音频 Transformer 做 speech/non-speech 分类），也未在非语音内部进一步区分音效、音乐、环境音——这三类在 UniVerse-1 的数据体系中被合并为单一的「通用/环境音」桶。
【三类能力的分工来源】
- 语音与唇同步能力：来自 1,187 小时经三重核验的语音子集，以及为泛化而移除 speaker encoder 的架构决策；
- 音效/环境音能力：来自 3,074 小时通用数据 + 3,422 小时 VGGSound/AudioSet（后者是事件音标注数据集，专为补强音效而引入）；
- 音乐/乐器声能力：主要继承自基座 Ace-step（本身即音乐生成模型），数据侧由音乐综艺与古典音乐演奏素材强化，但未单列小时数。
这是一个「音乐靠基座、音效靠公开数据集、语音靠自采精筛」的三段式能力分工，与 MOVA「音效/音乐在音频塔预训练注入、语音在联合训练强化」的两段式分工思路相近，但 UniVerse-1 把音乐能力的获取整个外包给了基座，自身不做音乐数据的专门策划。
【标注侧的类型分离】caption schema 中语音内容与环境音描述是两个独立字段，因此模型在文本条件层面能区分「说的话」与「其他声音」，但环境音字段内部不再细分音效/音乐/环境音。
【差异化损失】VGGSound/AudioSet 因视觉质量低而被施加 LQLS（仅在 t>800 的高噪声段计算 Flow Matching 损失），这是唯一一处按数据来源做的差异化训练处理，本质上是「让低质数据只教语义、不教细节」。

### [Unison](../models/Unison.md) ⚠️

语音、音效、音乐、歌唱四类音频的处理策略是 Unison 全部数据设计中最系统的部分：
【分类机制：不做分类判定，而做强制分解】所有音频一律经 Mel-RoFormer 分解为 speech 与 sfx 两路，不存在「判定这段音频属于哪类」的分类环节。这与 UniVerse-1（Whisper 做语音/非语音二分类分流）、MOVA（EAT 自监督音频 Transformer 做 speech/non-speech 分类）的做法有本质区别：前两者是「分类后走不同路径」，Unison 是「所有样本都走双路径」。好处是无分类错误风险、无路径不平衡问题；代价是对无语音样本也要维护一条空的语音流，且完全依赖分离模型的质量。
【纯音频语料按四类分源采集，各司其职】
- 语音：内部语音数据 → 训练 Zipformer 增强后的音频分支的语音生成能力；
- 音效：YouTube-8M + AudioSet + WavCaps → 三个数据集叠加，是音效侧覆盖最厚的部分；
- 音乐：VidMuse → 器乐/配乐能力；
- 歌唱：YuE collection → 专门补强歌唱能力。
【歌唱被单列为独立类别的意义】这是 Unison 相对同类工作的一处差异化投入。论文在定性对比中明确指出 UniVerse-1 与 UniAVGen「难以合成音乐伴奏，且难以区分歌唱与普通说话」（fail to synthesize musical accompaniments and struggle to distinguish singing from standard speech），MOVA「在音乐场景能生成可信人声但复杂环境下有明显语音伪影」。引入 YuE 歌唱语料正是为攻克这一被点名的失败模式。同时，论文反复用「边弹奏乐器边唱歌」（singing while playing an instrument）作为最困难场景的代表例——歌唱（结构化的人声）与器乐（结构化的非人声）同时出现，是语音-音效二分法最难处理的边界情形。
【生成侧的类型分工】speech 流承载语音与歌唱（均为人声），sfx 流承载音效、音乐与环境音。注意歌唱与音乐分别落在两条不同的流上——这正是 Mel-RoFormer（人声分离模型）分离逻辑的自然结果：它按「人声 vs 非人声」而非按「语义类别」切分。这一划分对「边唱边弹」场景恰好合适（人声进 speech 流、伴奏进 sfx 流），是架构与工具选择上的一个巧合性契合。
【运行时的类型自适应】SCG 门控依据 caption/transcription 语义在推理时动态调节两流配比：叙述主导场景抑制音效流对语音流的注入以保护「语音纯度」（phonetic purity），复杂声景场景放大跨流影响以丰富非语音声学。Fig. 8c 的实例级分析给出了体育赛事解说的具体案例——门控约束解说流以防其掩盖看台氛围的高频瞬态（撞击声与人群欢呼）。
【未披露】四类音频各自的小时数与占比、训练时的采样权重、Stage 1 中四类数据的课程安排或混合比例。[不确定]

### [Veo 3 / Veo 3.1](../models/Veo_3_Veo_31.md) ⚠️

[不确定] 未披露语音/音效/音乐的分类与分别处理策略。模型输出层面明确区分对白、音效、环境音、背景音乐四类且可分别通过 prompt 控制，说明训练数据的 caption 中这些类别是被区分描述的，但是否存在独立的音频分类器、是否对各类音频施加不同的过滤阈值或训练权重，均无公开信息。

### [Vidu S1](../models/Vidu_S1.md) ⚠️

语音（speech）是唯一被作为训练与控制核心的音频类型：显式提取语音成分、做 VAD/ASD/说话人归属分类，并以语音作为流式生成的实时控制信号。音乐与歌唱被视为噪声源，通过语音能量占比启发式规则系统性剔除。音效与背景音乐不参与专门建模，但在结构化 caption 中保留为独立描述字段（sound effects、background music）。产品端支持用户选择不同音色（voice tones），推测音频侧接入了 TTS/音色控制，但论文未展开 [不确定]。

### [主流视频预训练数据集合并调研：Panda-70M、InternVid、Koala…](../models/pretraining_datasets.md)

不适用。七者均无语音/音效/音乐/环境音的分类与分别处理策略。
