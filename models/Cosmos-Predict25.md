# Cosmos-Predict2.5（NVIDIA Cosmos 世界基础模型家族最新一代，论文《World Simulation with Video Foundation Models for Physical AI》arXiv:2511.00062；同篇一并发布 Cosmos-Transfer2.5 控制网家族）

> 主题: 视频生成模型（含音视频同时生成）的数据处理：数据清洗流程、数据分布、打标方式

[← 返回首页](../index.md)

**目录**: [基本信息](#基本信息) · [数据规模与分布](#数据规模与分布) · [清洗流程](#清洗流程) · [打标方式](#打标方式) · [音视频对齐](#音视频对齐) · [训练配合](#训练配合) · [效果对比](#效果对比)

## 基本信息

### 名称

Cosmos-Predict2.5（NVIDIA Cosmos 世界基础模型家族最新一代，论文《World Simulation with Video Foundation Models for Physical AI》arXiv:2511.00062；同篇一并发布 Cosmos-Transfer2.5 控制网家族）

### 发布机构/公司

NVIDIA（论文署名即 NVIDIA，含 88 位贡献者的大型协作团队，作者含 Arslan Ali、Junjie Bai、Sanja Fidler 等；隶属 NVIDIA Cosmos Lab / Physical AI 方向）

### 发布时间（技术报告/论文/开源时间）

arXiv v1 提交于 2025年10月28日（arXiv:2511.00062v1），v2 修订于 2026年2月24日（论文页眉标注 2026-2-26，版权标注 ©2026 NVIDIA）。代码与权重同期在 GitHub nvidia-cosmos/cosmos-predict2.5 与 Hugging Face nvidia/Cosmos-Predict2.5-2B / -14B 开放。

### 类型（模型/数据集/工具链/评测基准）

模型（视频世界基础模型 World Foundation Model）+ 配套工具链 + 自建评测基准。主体为基于 flow matching 的 DiT 视频生成模型，单一模型统一 Text2World / Image2World / Video2World 三种模式，发布 2B 与 14B 两档；同时发布 Cosmos-Transfer2.5 控制网系列（Sim2Real / Real2Real 世界翻译）与多个领域特化变体（自动驾驶 7 视角多视图、机器人动作条件、AgiBot 三视角、GR00T GR1）。评测侧使用自建的 PAI-Bench（Physical AI Bench）。数据处理侧关联 NVIDIA 独立开源的 Cosmos Curator / Cosmos-Xenna 与 NeMo Curator 视频 curation 框架。

### 开源程度（权重/代码/数据/pipeline各自是否开源）

开放程度在同级别闭源大厂模型中偏高，但训练数据本身不开放。
【已开源】(1) 源码：GitHub nvidia-cosmos/cosmos-predict2.5 与 nvidia-cosmos/cosmos-transfer2.5，采用 Apache 2.0 License；(2) 预训练与后训练检查点：2B / 14B 的 pre-trained、post-trained 版本，以及蒸馏版与领域特化版（auto/multiview、robot/action-cond、robot/multiview-agibot、robot/gr00tdream-gr1），发布于 Hugging Face，遵循 NVIDIA Open Model License；(3) curated benchmarks（PAI-Bench 相关评测集）与「curated post-training examples」（精选后训练示例）；(4) 数据 pipeline 的工程底座间接开源——Cosmos Curator（github.com/NVIDIA/cosmos-curator）与其 GPU 流式执行框架 Cosmos-Xenna 已独立开源，NeMo Curator 的视频 curation 模块亦公开，二者即论文所述 curation pipeline 的产品化形态。
【未开源】200M 预训练视频数据集本身；自研的内容类型分类器（26 类 taxonomy）、内部美学/运动/OCR/感知质量/语义 artifact 打分器的权重与阈值；NVIDIA 内部驾驶平台采集的 3.1M 段 7 相机专有数据；各过滤器的具体超参。
【定位】论文明确目标是「lower the barrier to adoption」，把预训练模型交给社区做领域特化，而非开放数据。

### 是否支持音视频同时生成，以及实现方式（原生联合/级联/MoE融合）

不支持。Cosmos-Predict2.5 是纯视觉的视频世界模型，输入为文本 / 图像 / 视频条件，输出为无音轨视频（93 帧、16fps、约 5.8 秒）。全文 44 页未出现 audio / speech / sound 相关的数据处理或生成描述，音频既不是条件也不是输出。其「多模态」体现在文本编码器换用 Cosmos-Reason1（Physical AI 专用 decoder-only VLM）以及机器人变体的 action 条件输入，而非音视频联合生成。因此本调研中所有音视频相关维度（音频类别配比、联合 caption、对白转写、唇同步/事件对齐、音频质量过滤、音频类型处理）对该工作均不适用。

### 调研信息来源列表（论文/技术报告/官方文档/新闻的URL，标注每条来源的性质：官方一手/同团队旁证/第三方报道）

1. arXiv:2511.00062《World Simulation with Video Foundation Models for Physical AI》 https://arxiv.org/abs/2511.00062 ，全文 PDF https://arxiv.org/pdf/2511.00062v2 —— 官方一手，核心依据。本调研的七阶段 pipeline、35M 小时/6B clip/4% 保留率/200M clip、各过滤器、Qwen2.5-VL-7B captioning、语义去重、26 类 taxonomy 分片、五大领域数据、SFT 五域规模表、RL 与蒸馏配置、基础设施与 MFU 数据均逐字出自 v2 版第 2 节 Data、第 4 节 Training 与第 6 节 Applications。
2. GitHub 官方仓库 https://github.com/nvidia-cosmos/cosmos-predict2.5 与 https://github.com/nvidia-cosmos/cosmos-transfer2.5 —— 官方一手（代码 Apache 2.0、模型 NVIDIA Open Model License、发布模型清单、guardrails 提及）。
3. Hugging Face 模型卡 https://huggingface.co/nvidia/Cosmos-Predict2.5-2B 与 https://huggingface.co/nvidia/Cosmos-Predict2.5-14B —— 官方一手（权重、许可、能力说明）。
4. NVIDIA Research Cosmos Lab 项目页 https://research.nvidia.com/labs/cosmos-lab/cosmos-predict2.5/ —— 官方一手（200M 高质量预训练 clip、模型合并与 RL 的对外口径）。
5. NVIDIA Cosmos Curator 开源仓库 https://github.com/NVIDIA/cosmos-curator 与文档 https://docs.nvidia.com/cosmos-curator-lha/current/introduction.html —— 官方一手，同团队旁证（论文所述 curation pipeline 的产品化实现，GPU 流式框架 Cosmos-Xenna、镜头切分/embedding/caption 各阶段的工程细节）。
6. NeMo Curator 视频 curation 文档 https://docs.nvidia.com/nemo/curator/curate-video 与 https://docs.nvidia.com/nemo/curator/latest/get-started/video.html —— 官方一手，旁证（GPU 加速视频 pipeline 能力、2000 块 H100 一天处理约 100 万小时 720p 视频的吞吐口径）。
7. 前代论文 NVIDIA (2025)《Cosmos World Foundation Model Platform for Physical AI》 —— 官方一手，旁证（20M 小时、30% 存活率的对照基线，以及 Cosmos-Guardrail 护栏体系）。
8. Emergent Mind 主题页 https://www.emergentmind.com/topics/cosmos-predict2-5 、DL 轮读会讲解 https://www.docswell.com/s/DeepLearning2023/KPGPD6-2025-11-17-152333 —— 第三方索引与解读（用于交叉核对数字，非主要依据）。

## 数据规模与分布

### 训练数据量级（视频条数/小时数/token数，预训练与SFT分开） ⚠️

预训练与后训练的规模披露相当完整，是该报告最有价值的部分之一：
【原始输入】处理超过 2 亿（over 200 million）条原始视频，对应 3500 万小时（35 million hours）原始视频素材（作为对比，Cosmos-Predict1 为 2000 万小时）。
【切分产出】剔除时长不足 5 秒的片段后，得到超过 60 亿（over 6 billion）条候选 clip，时长范围 5–60 秒。
【最终预训练集】约 4% 通过全部过滤，得到约 2 亿（approximately 200 million）条可训练 clip，这 2 亿条即预训练数据集。
【后训练 SFT 各域规模】用 InternVideo2 embedding 上训练的多头分类器划分为五域并统计：object permanence（物体恒存）10.4M 条、high motion（高运动）1.0M 条、complex scenes（复杂场景）1.6M 条、driving（驾驶）3.1M 条、robotic manipulation（机器人操作）730K 条；另有 4K 高清冷却（cooldown）数据 388K 条。
【领域专项】自动驾驶专有数据 约 310 万（3.1M）段 20 秒 7 相机环视 clip；多视图模型 Cosmos-Predict2.5-2B/auto/multiview 训练用 150 万（1.5M）段带 caption 的多视图 clip；Smart Spaces 约 40K clip；机器人各数据集见 data_sources。
【训练迭代】每个领域 SFT 模型训练 30k iterations、batch size 256；RL 阶段 256 steps、batch size 32。预训练总 token 数与总迭代数未披露。[不确定：预训练总迭代数/token 数、35M 小时中各来源占比]

### 数据来源构成（自有/公开数据集/网络爬取/授权采购/合成数据）

构成为「专有数据集 + 公开互联网平台 + 公开机器人数据集 + NVIDIA 内部采集」四类，但通用预训练部分只给定性描述、无比例：
(1) 通用预训练：「sourced from both proprietary datasets and open internet platforms」（专有数据集与公开互联网平台），覆盖 driving（驾驶）、object manipulation（物体操作）、spatial navigation（空间导航）、human interaction（人类交互）、nature scenes（自然场景）等域；两类来源的占比未披露。
(2) 机器人域为点名的公开数据集集合，且给出按相机视角的条数（中央视角/左视角/右视角）：AgiBot-Beta（双臂）194k/30k/30k、Bridge（单臂）36k、DROID（单臂）39k（腕部视角）/51k/51k、GR00T（双臂）3k、1X Technologies（双臂）17k、OpenX（单臂）500、RoboMIND（双臂/人形）16k/6k/7k。
(3) 自动驾驶为 NVIDIA 自有采集：「a proprietary dataset ... collected using NVIDIA's internal driving platform」，约 3.1M 段 20 秒环视 clip，7 路同步相机（front-wide、front-tele、left、right、rear、rear-left、rear-right）。
(4) Smart Spaces（仓库/工厂/工地等）：先用检索关键词召回候选视频，再用 VLM（Qwen2.5-VL 系）逐条判定相关性，切分过滤后约 40K clip 存活。
(5) 合成数据：预训练阶段主动排除（游戏画面、合成视觉图案、动画、卡通均被剔除）；但下游应用中 Cosmos-Transfer2.5 被用来生成合成增强数据训练机器人策略（见 synthetic_data_synthesis）。

### 数据合规与溯源（授权数据占比、rights-cleared数据集、C2PA等） ⚠️

[不确定]。论文完全未涉及数据授权、版权与溯源议题：没有 rights-cleared 比例、没有采购/授权来源说明、没有 C2PA 或任何内容溯源与水印标识方案。仅笼统提到来源包含「proprietary datasets and open internet platforms」。发布侧的许可是清晰的（代码 Apache 2.0、模型 NVIDIA Open Model License），但这不构成对训练数据来源合法性的任何声明。NVIDIA Cosmos 平台层面另有 Cosmos-Guardrail 安全护栏（推理侧内容拦截），本篇论文亦未展开描述。

### 片段时长分布与切分策略 ⚠️

切分与时长策略披露明确：
(1) 长视频先经 shot-aware splitting 切为单镜头段；
(2) 时长门槛：「Very short clips under 5 seconds are discarded」——短于 5 秒的片段直接丢弃；
(3) 保留区间：剩余片段时长为 5 至 60 秒（ranging from 5 to 60 seconds），共 60 亿+ 条；
(4) caption 粒度：每条 clip 再切成 5 秒窗口逐窗口打标（这是相对 Cosmos-Predict1 的「finer content granularity」改进）；
(5) 训练消费粒度固定：全部阶段统一生成 93 帧、16 fps，约 5.8 秒（对应 WAN2.1 VAE 4×8×8 压缩后的 24 个 latent 帧）；
(6) 领域数据的时长是定长的：自动驾驶 clip 统一 20 秒；Human Dynamics 要求至少 5 秒。
(7) 时长是最终 sharding 的四个轴之一（length），支撑按时长采样。
未给出 5–60 秒区间内的具体分布直方图或平均时长。[不确定：时长分布的具体统计量]

### 分辨率/宽高比分布与分桶策略 ⚠️

(1) 分桶：sharding 阶段显式沿 resolution（分辨率）与 aspect ratio（宽高比）两个轴分片，与 content type、length 共四轴，目的是支撑「efficient sampling, curriculum-based training, and fine-grained domain balancing」，因此存在明确的分辨率/宽高比分桶机制，但各桶占比未公布。
(2) 分辨率被用作去重的仲裁准则：语义去重时同簇内保留最高分辨率版本（the highest-resolution version is retained），理由是高分辨率保留更细的视觉细节、提供更丰富的训练信号；在线增量去重时以「更早 + 更高分辨率」作为 tie-breaking 优先级。
(3) 训练侧分辨率课程明确：256p（320×192）→ 480p（832×480）→ 720p（1280×704），逐级提升，每级待模型收敛、视觉质量饱和后再进入下一级。
(4) 冷却阶段专用一批精选 4K 高清视频（388K 条），学习率线性衰减至 0，用于提升细节精度与运动平滑度。
(5) 架构上移除了绝对位置编码、仅保留相对位置编码（3D RoPE），明确目的是提升对训练时未见过的分辨率与序列长度的泛化能力。
(6) 机器人域预过滤会剔除低分辨率视频。
[不确定：原始语料各分辨率/宽高比的占比数值]

### 类别/domain分布与配比策略（人物、动作、场景、风格等比例控制与概念均衡） ⚠️

domain 体系是该 pipeline 的核心设计之一，且分为「通用 26 类 taxonomy」与「Physical AI 五大专项域」两层：
【第一层 通用内容类型分类】过滤末端由内部训练的 content type classifier 为每条 clip 打一个语义标签，取自自建的 26 类视频类型 taxonomy（a custom-built taxonomy of 26 video types）。该标签是 sharding 的首要轴，用于「fine-grained domain balancing」（细粒度域均衡）与 curriculum-based training。26 类的具体类目名与各类占比均未公布。
【分布对齐约束】在此阶段显式剔除「physically unrealistic phenomena」——video games（游戏画面）、synthetic visual patterns（合成视觉图案）、animations（动画）、cartoons（卡通），理由是「maintain alignment with the physical world distributions」（保持与真实物理世界分布的对齐）。这是一条以物理真实性为准绳的分布裁剪原则，区别于一般视频生成模型追求风格多样性的做法。
【第二层 Physical AI 五大专项域】为增强 Physical AI 能力，另设五条领域专用 pipeline，产出并入通用预训练数据：Robotics（机器人）、Autonomous Driving（自动驾驶）、Smart Spaces（智能空间：仓库/工厂/工地）、Human Dynamics（人体动力学）、Physics（物理现象）。这些 pipeline 与通用流程结构相同，但有两处关键差异：过滤上省略昂贵的 VLM filter，改用「领域特定的过滤器子集 + 调整过的超参数」；打标上改用更大参数量的 VLM 并配领域定制 prompt。
【自动驾驶的目标分布采样】驾驶数据不是随机抽样，而是「sampled from a large-scale corpus to align with a target distribution of diverse driving attributes」——按预设目标分布在九个属性轴上均衡采样：地理区域（美国、欧洲）、交通密度（稀疏/拥堵）、自车速度（城市道路/高速）、自车加速度（匀速/急加速）、自车机动动作（缓弯/急转）、道路类型（城市/乡村）、罕见道路结构（隧道、收费站）、能见度（晴朗/雾天）、天气（干燥/雪天）、光照（白天/夜间）。这是全文最具体的配比控制策略。
【物理域的 taxonomy】先定义一套「可视觉观测的物理现象」分类体系，覆盖经典力学与流体力学等核心领域（如玻璃碎裂 shattering glass、滚球碰撞 colliding rolling balls、水流 flowing water），再据此定向采集能凸显这些动态属性的视频。
【Human Dynamics 的量化准入】人体域用硬性数值规则控制画面构成：人物需出现在超过 40% 的帧中；任一帧中可见人数不超过 8 人；至少一人占据画面面积 3% 以上。
【后训练五域】SFT 侧另有一套独立划分（object permanence / high motion / complex scenes / driving / robotic manipulation），由 InternVideo2 embedding 上的多头分类器产出，见 post_training_data。
[不确定：26 类 taxonomy 的具体类目与各类占比数值]

### 音频类别分布与配比（语音/音效foley/音乐/环境音/静音的比例及控制策略）——AV模型独有维度

不适用。Cosmos-Predict2.5 无音频模态——不生成音频、不以音频为条件、数据 pipeline 中无任何音轨处理环节（不做音轨提取、不做语音/音效/音乐分类、不做静音检测）。全文未出现音频类别配比的任何设计。若需覆盖 Physical AI 场景的声音维度，需另行外接音频模型，论文未讨论。

### 叙事结构分布（单镜头vs多镜头、平均clip时长、镜头数分布、是否含原生音轨）

训练片段全部为单镜头（single-shot）：pipeline 第一阶段即用高精度镜头边界检测模型把长视频切为 shot，并明确「ensuring that raw shot transitions are excluded」（确保原始镜头转场被排除在外）；后续「semantic artifacts filter」（类 VTSS）进一步剔除 poor transitions（劣质转场）与 video-in-video（画中画）等结构异常样本。因此数据集内不含多镜头叙事样本，也不存在镜头数分布概念。
单 clip 时长 5–60 秒，caption 按 5 秒窗口切分标注；模型单次生成 93 帧 / 约 5.8 秒。长时程能力靠 Video2World 条件续写与 Cosmos-Transfer2.5 的 long-horizon 视频翻译实现，而非训练长片段。
多视图是该工作特有的「非叙事型结构维度」：自动驾驶数据为 7 路同步相机的环视 clip（30 FPS、20 秒），多视图模型把 7 个视角沿 latent 时间维拼接（latent 时间维压缩至 8 以容纳 7 视角），机器人 AgiBot 变体为 3 相机视角。
原生音轨：不涉及（无音频处理）。

### 语言/口音分布（多语种唇同步能力的数据基础） ⚠️

不适用于语音维度（无音频、无语音、无口音概念）。文本维度上，caption 由 Qwen2.5-VL-7B 生成，论文未说明 caption 语言，但从文中所有示例 prompt（如机器人焊接场景描述、驾驶场景 prompt 模板、Transfer2.5 的厨房场景 prompt 模板）均为英文、且文本编码器 Cosmos-Reason1 为英文为主的 Physical AI VLM 判断，训练 prompt 基本为纯英文单语，未见多语种 caption 或翻译增强环节。[不确定：是否存在未披露的非英文 caption]

## 清洗流程

### 清洗漏斗整体结构（几级过滤、各级顺序）

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

### 漏斗定量保留率（每级过滤的输入/输出量与最终保留率，如Apollo 27%） ⚠️

论文给出了明确的端到端定量漏斗，且与前代做了直接对比，是本调研中定量披露最充分的案例之一：
【Cosmos-Predict2.5】原始素材 3500 万小时 / 2 亿+ 条原始视频 → 切分裁剪并剔除 <5 秒片段后得到 60 亿+ 条候选 clip（5–60 秒）→ 经七级过滤 + 语义去重后「Only about 4% of the initial clips pass all filters」→ 约 2 亿条可训练 clip。即 clip 级总保留率约 4%（6B → 200M，约 1/30）。
【Cosmos-Predict1 对比】原始 2000 万小时，clip 级存活率 30%。论文明确表述：「it achieves improved data quality control through a far stricter multi-stage filtering pipeline, which reduces survival from 30% of clips to only 4%」——即在原始素材量放大 1.75 倍（20M→35M 小时）的同时，把存活率压缩到原先的 1/7.5，是一次刻意的、大幅度的「以严格换质量」策略切换。
【领域侧】Smart Spaces 为「关键词召回 → VLM 相关性核验 → 切分 → 过滤 → 约 40K clip 存活」，但未给出召回端的输入量，无法计算该域保留率。
【未披露】七个过滤子级各自的输入/输出量与逐级通过率（只有端到端的 4% 总数），以及去重环节单独去掉了多少。[不确定：各过滤子级的分级保留率]

### 镜头切分方法（PySceneDetect/自研模型/shot-aware splitting） ⚠️

第一阶段即为「shot-aware video splitting」（镜头感知切分）：用高精度镜头边界检测模型（high-accuracy boundary detection models，复数，暗示多模型组合）将长视频切分为镜头级片段，核心质量要求是「ensuring that raw shot transitions are excluded」——切出的片段中不能残留原始转场帧，避免训练样本内部出现画面突变。论文未点名具体模型（未提 PySceneDetect / TransNetV2 等），但 NVIDIA 独立开源的 Cosmos Curator 中对应实现同时提供基于像素阈值的快速切分与基于 TransNetV2 的神经网络切分两种模式，可作为工程旁证。
切分之后紧接 GPU 转码与裁剪（去黑边与空间 padding），并以 5 秒为下限丢弃过短片段，上限 60 秒。
转场质量的兜底在过滤阶段：semantic artifacts filter（类 VTSS）专门剔除 poor transitions（劣质转场）与 video-in-video（画中画），即对切分漏检做二次补救。
[不确定：镜头边界检测模型的具体型号]

### 质量过滤（美学评分、清晰度、OCR文字过滤、黑边/水印/logo检测） ⚠️

质量过滤是七级漏斗中最重的一环，由六个独立过滤器串行构成，覆盖美学、技术、语义三个层次，且论文明确了排序原则（廉价在前、昂贵在后）：
(1) 美学打分 aesthetic scoring filter —— 按美学质量给输入打分并分级；打分器型号与阈值未披露。
(2) OCR filter —— 「attempts to remove clips with excessive text overlay」，剔除文字叠加过多的片段（字幕、贴片文字、台标类）；检测器与文字面积阈值未披露。
(3) perceptual quality filter 感知质量过滤 —— 论文明确「akin to DOVER (Wu et al., 2023)」，即采用 DOVER 式的分离视角视频质量评估（DOVER 将 VQA 拆为 technical 与 aesthetic 两支），目标是剔除「technical distortions and perceptual artifacts」（技术失真与感知伪影），如压缩块效应、噪声、模糊。
(4) semantic artifacts filter 语义伪影过滤 —— 明确「akin to VTSS (Wang et al., 2025)」，目标是剔除结构/语义层面的异常：video-in-video（画中画、录屏套娃）、poor transitions（劣质转场）等。这是相对 Predict1 新增的一级，专门对付「技术指标正常但结构不适合训练」的样本。
(5) VLM filter —— 用 Qwen2.5-VL 系视觉语言模型「with higher precision」剔除一组不良问题（a set of undesirable issues），作为终审；因计算昂贵而置于最末。
(6) content type classifier + 物理真实性裁剪 —— 内部训练的内容类型分类器给出 26 类 taxonomy 标签，并在此阶段剔除「physically unrealistic phenomena」：视频游戏画面、合成视觉图案、动画、卡通，以维持与真实物理世界分布的对齐。这是 Physical AI 定位下的特殊质量标准——「非物理真实」本身被定义为一种质量缺陷。
黑边与空间 padding 不靠过滤而靠第三阶段的 cropping 直接裁除（修复而非丢弃）。水印/logo 检测未被单独列出（可能并入 OCR 或 VLM filter）。
所有过滤器的阈值数值一律未公布；领域支线则明确使用「a domain-specific subset of filters with adjusted hyperparameter values」（领域特定的过滤器子集 + 调整过的超参），说明阈值是按域可调的。[不确定：各过滤器的具体模型型号与阈值数值]

### 运动过滤（光流/运动分数阈值、静态与抖动剔除） ⚠️

运动过滤是过滤链的第二级：「we apply a motion filter, which quantifies and removes clips based on their degree of motion」——量化每条 clip 的运动程度并据此剔除。论文未说明度量方式（未指明光流、帧差还是模型打分），也未给出阈值，更未说明是双侧过滤（同时剔除近静止与剧烈抖动）还是仅剔除低运动一侧。
可确认的相关证据有三处：(1) 机器人域预处理明确「filtered out low-resolution and near-static videos」——剔除低分辨率与近乎静止的视频，说明至少存在低运动侧的剔除；(2) 机器人域还做了运动节奏归一化——对机械臂动作过慢的视频提高播放速度（increased the playback speed），以保证跨数据集的动作节奏一致，这是一种少见的「主动改写运动速率」而非单纯过滤的做法；(3) 后训练 SFT 把 high motion（高运动）单独列为五大域之一并保留 1.0M 条，说明高运动样本是被刻意保留和强化的目标，而非被过滤对象。
[不确定：运动度量方法与阈值、是否过滤高频抖动]

### 去重方法（精确去重与基于embedding的语义去重分别记录） ⚠️

语义去重为独立的第六阶段，方案设计得比较完整，且专门考虑了增量扩容场景：
(1) 聚类：先用 embedding 相似度把 clip 分配到簇（assign video clips to clusters using embedding-based similarity），把全局两两比对降为簇内比对，这是可扩展性的关键；
(2) 簇内比对：簇内 clip 两两比较以检出语义相似内容（compared pairwise to detect semantically similar content）；
(3) 仲裁规则：重复组内保留分辨率最高的版本（the highest-resolution version is retained），论文给出的理由是高分辨率保留更细的视觉细节、提供更丰富的训练信号——这是一条明确的「同内容选优」而非「随机保一」策略；
(4) 在线增量去重（online deduplication strategy）：每条新入库 clip 与此前已保留的 clip 比对，tie-breaking 时优先保留更早的、分辨率更高的 clip。此设计使数据集可持续增长而不必全量重跑去重，同时保持全语料的语义一致性；
(5) 基础设施支撑：接入 Milvus 开源向量数据库做 embedding 检索，支撑语义相似度搜索与 caption 级文本 embedding 检索。
论文只描述了语义/embedding 级去重，未单独描述精确去重（MD5/字节级哈希）环节——推测由上游采集或 Cosmos Curator 工程实现承担，论文未提。[不确定：embedding 模型型号、相似度阈值、是否另有精确去重]

### VLM/LLM作为质检员（多模态大模型质量打分与错配剔除，2026年从浅层打分器转向大模型语义判断的趋势）

该 pipeline 是「浅层打分器 + 大模型终审」两段式的典型代表，且论文罕见地明确说明了为什么这么排：
【VLM 作为终审质检员】「Finally, we use a vision language model (VLM) (Bai et al., 2025) to further remove clips with a set of undesirable issues with higher precision. We apply the VLM at the very end of filtering because it is computationally more expensive.」——即 VLM（Qwen2.5-VL 系）不是替代浅层打分器，而是放在链末对幸存者做高精度复审，理由是算力昂贵。这正是「先用便宜过滤器砍掉大头、再让大模型精修边界样本」的经济学设计，也是 2026 年前后行业从纯浅层打分转向大模型语义判断的过渡形态。
【VLM 作为召回核验员】Smart Spaces 域：先用搜索关键词召回可能相关的候选视频，再「used a VLM (Bai et al., 2025) to verify its relevance」——用 VLM 逐条核验主题相关性，这是把 VLM 当作语义级的主题分类/召回净化器。
【VLM 作为标注员】captioning 全部由 Qwen2.5-VL-7B 承担，领域数据改用更大参数量的同族 VLM。
【专用分类器作为分域员】内容类型分类器（内部训练，26 类 taxonomy）在过滤末端为每条 clip 分类；后训练侧则用 InternVideo2 embedding 上的多头分类器把数据切成五个 SFT 域。这两处是判别式小模型而非生成式大模型，体现「能用小模型的地方不上大模型」。
【类模型打分器】perceptual quality filter 用 DOVER 式模型、semantic artifacts filter 用 VTSS 式模型，都是学习型质量评估模型而非规则阈值。
【RL 侧的模型裁判】后训练用 VideoAlign（VLM-based reward model）作为奖励模型，从文本对齐、运动质量、视觉质量三维打分，并配套自研的 Elastic Reward Service 弹性奖励服务（支持动态扩缩容、latent 传输压缩、decode 与 inference 流水线并行、CUDA IPC 零拷贝、Redis 存结果的异步 UUID 机制）——这是把「模型当裁判」工程化到服务级的少见案例。
【领域支线的例外】五大 Physical AI 域明确省略 VLM filter（omit the VLM filter），改用领域特定过滤器子集加调参。这说明团队认为在数据来源已经可信、域已收窄的情况下，昂贵的 VLM 终审性价比不足——是一个有价值的工程判断。

### 安全与合规过滤（NSFW、版权、人脸/隐私） ⚠️

[不确定]。论文层面几乎没有安全合规过滤的描述。唯一沾边的表述是过滤目标中包含「content that is unsuitable for training」（不适合训练的内容），未展开说明其定义、检测手段与规模，推测涵盖 NSFW / 暴力等，但无文字依据可确认。全文未提 NSFW 分类器、暴力检测、人脸隐私/去标识化、版权过滤等任何具体机制。
间接相关的是 NVIDIA Cosmos 平台层面的 Cosmos-Guardrail 护栏体系（前代 Cosmos-Predict1 中描述过、GitHub 仓库文档提及「robust guardrails」与「improved guardrails」），但那是推理侧的输入 prompt 与输出内容拦截，并非训练数据清洗环节，且本篇未展开。
值得注意的是本 pipeline 有一条与安全无关但方向相似的内容裁剪：剔除游戏/动画/卡通等非物理真实内容，出发点是分布对齐而非安全。

## 打标方式

### 使用的caption模型（自研VLM/开源模型，模型规模） ⚠️

采用「通用域用中等规模 VLM、专项域用更大 VLM」的分层策略：
(1) 通用预训练 caption：Qwen2.5-VL-7B（论文明确点名型号与参数量，引 Bai et al., 2025），对每个 5 秒窗口生成 caption，prompt 被工程化引导以产出「factual, context-aware captions」（事实性、上下文感知的描述）。
(2) 领域专项 caption：明确改用「a larger VLM model (Bai et al., 2025)」——同为 Qwen2.5-VL 系但更大档位（未指明是 32B 还是 72B），并为每个域配定制 prompt。团队的判断是：领域数据量小得多但对描述精度要求更高，值得上更大模型。
(3) 多视图驾驶模型的 caption：论文写为「Qwen2.5-7B-Instruct」，每 150 帧生成一次、三档长度（原文如此，与通用侧的 VL 版本表述不同，疑为笔误或指用 LLM 对视觉结果做二次改写）。
(4) 域分类模型：内容类型分类器为内部训练模型；后训练域划分用 InternVideo2 embedding 上训练的多头分类器。
(5) 文本编码器（非 caption 模型但相关）：用 Cosmos-Reason1 替换 Cosmos-Predict1 的 T5，是 Physical AI 专用的 decoder-only VLM；且不取单层输出，而是拼接多个 block 的激活再投影到 1024 维，以同时捕捉局部与全局语言上下文。
[不确定：领域 VLM 的确切参数档位；多视图 caption 模型表述的歧义]

### caption密度与结构化程度（短/长/密集描述、结构化字段如镜头运动、风格标签） ⚠️

结构化程度体现在「时间粒度切分 + 三档长度 + 领域定制字段」三个维度：
【时间粒度】每条 clip 切成 5 秒窗口逐窗口 caption，而非整条一句话。论文把这列为相较 Cosmos-Predict1 的核心改进之一：「introduces finer content granularity by segmenting clips into shorter temporal windows」，目的是获得「richer and more precise supervision signals」（更丰富精确的监督信号）。
【三档长度】caption 统一产出 short / medium / long 三种长度（Captions are produced at multiple lengths），既作训练监督信号又作条件 prompt，使模型同时适配简短用户输入与详尽描述。驾驶域同样三档。具体 token 长度区间未给出。
【通用侧内容约束】prompt engineering 引导模型描述 primary object（主体物体）、its motion（其运动）、key semantic details（场景关键语义细节）三要素，并强调 factual（事实性）以抑制幻觉。
【机器人域 caption schema】结构化最强，prompt 强制要求：先枚举 initial scene（初始场景），再按时间顺序描述机器人动作，且必须显式标注 motion type（运动类型，如 linear 线性 / rotational 旋转）、involved parts（涉及部件：arm 手臂 / wrist 腕 / gripper 夹爪）、camera motion（相机运动）、fine-grained object attributes（细粒度物体属性）。同时做 viewpoint 与 embodiment 的归一化表述（统一跨数据集的相机视角命名）。为降低幻觉，把数据集自带元数据注入 prompt：GR00T 注入带人工成功评级的任务描述、Bridge 注入 step-level 指令、AgiBot 注入初始场景描述。这套「元数据注入 prompt 以约束 VLM」的做法是抑制视频 caption 幻觉的有效范式。
【驾驶域 caption schema】六类必写要素：① 自车需注意的各类 agent（车辆/行人/骑行者）与交通元素（红绿灯/交通标志）；② 全局环境因素（天气、时段、路面状况）；③ 自车与周车的纵向与横向 meta action；④ 自车与周车速度；⑤ 其他物体的动态行为或状态转移；⑥ 关键物体之间的交互。
【Smart Spaces / Physics / Human Dynamics】分别定制 prompt：Smart Spaces 指明视频聚焦工厂/仓库/工业设施/汽车等制造环境并相应调整语言风格；Physics 引导 VLM 准确详细描述底层物理过程与物体交互；Human Dynamics 强调人体运动与动力学的细致描述。
未披露 caption 的平均 token 数、三档长度的混用采样比例。[不确定：caption 长度数值与三档配比]

### 音视频联合caption结构（是否同时覆盖视觉+听觉轨道、是否分流为独立字段，如LTX-2全音景描述、Script-a-Video factorized streams、Foley-Omni三字段）

不适用。模型无音频模态，caption 全部为纯视觉/语义描述，不包含任何听觉轨道字段，也不存在视觉流与听觉流分流的 schema。可类比的「多流分解」结构在该工作中体现为空间维度而非模态维度：驾驶域的七路相机各自独立编码、caption 每 150 帧生成一次；以及 caption 按 5 秒时间窗切分的时间维分流。与 LTX-2 全音景描述、Script-a-Video 的 factorized streams、Foley-Omni 三字段等真正的音视频联合 caption 范式无可比性。

### 对白转写与说话人属性标注（ASR转写+说话人身份/语言/口音/情绪）

不适用。无音频输入，因此不存在 ASR 转写、说话人身份/语言/口音/情绪标注等任何环节。与「人」相关的标注走的是纯视觉几何路线：Human Dynamics 域用 YOLOX 做人体检测、RTMPose 做全身关键点与面部 landmark 估计，用于筛选画面构成（人物帧占比、人数上限、单人面积占比），而非用于说话或表情语义标注。

### 几何与结构化标注（相机参数、深度、3D point tracks、动作标注、显式状态）

几何与结构化标注是该工作区别于通用视频生成模型的显著特征，因为面向 Physical AI 与机器人/驾驶仿真：
【人体关键点】Human Dynamics 域用 YOLOX（Ge et al., 2021）做人体检测、RTMPose（Jiang et al., 2023）做 full-body keypoints（全身关键点）与 facial landmark（面部关键点）估计；这些几何量被用作数据准入的硬性判据：人物出现帧占比 >40%、任一帧可见人数 ≤8、至少一人占画面面积 ≥3%。
【多相机标定与视角结构】驾驶数据为 7 路同步相机（front-wide、front-tele、front-left/left、front-right/right、rear、rear-left、rear-right，不同段落命名略有差异），30 FPS、20 秒；每视角在 latent 通道维拼接一个 7 维的 per-view 可学习 embedding，并对每个视角单独构造 3D-factorized RoPE，即视角身份是显式编码的结构化信息。
【驾驶结构化控制信号「world scenario map」】Cosmos-Transfer2.5-2B/auto/multiview 的控制输入由 HD 高精地图元素与动态物体投影到七个相机视角构成，包含车道线、道路标线、杆件、道路边界、交通信号灯（含灯态）、交通标志，可表达含立交桥在内的复杂道路拓扑；动态物体表示为 3D bounding box 立方体，按粗粒度类别本体（truck / vehicle / pedestrian 等）色彩编码，并用明暗区分正面与背面。这是全文最重的结构化标注体系。
【机器人域的动作与元数据】action-conditioned 变体以 action 作为条件输入；caption 侧强制标注运动类型（线性/旋转）、涉及部件（臂/腕/夹爪）、相机运动；并注入 GR00T 的人工成功评级、Bridge 的 step-level 指令等结构化元数据。策略学习实验中还有末端执行器位姿与夹爪指令的动作 chunk（8 步 horizon、10 FPS 采样）与关节状态。
【分割与检测（下游增强用）】机器人策略数据增强中用 Grounding DINO + SAMv2 做机器人像素的检测与分割，以实现「edge 控制作用于全图、blur 控制仅作用于机器人像素」的分区控制；Cosmos-Transfer2.5 的通用控制网支持 edge、blur、segmentation、depth 四类控制模态。
【未涉及】通用预训练语料未做深度图、3D point tracks、相机外参轨迹等标注（这些只出现在控制网的控制信号与驾驶专有数据中）。

### 合成数据构造（受控扰动/编辑构造训练对，如InstructAV2AV）

呈现出「训练数据侧排斥合成、下游应用侧生产合成」的双面态度，是该工作的一个鲜明特点：
【预训练侧：主动排除合成内容】过滤末端显式剔除 video games、synthetic visual patterns、animations、cartoons，理由是维持与真实物理世界分布的对齐。即在训练语料层面，合成/非物理内容被视为污染。
【机器人数据的受控改写】机器人域对动作过慢的视频提高播放速度以归一化动作节奏，属于对真实数据的受控时间扰动。
【下游：模型即合成数据工厂】这是该论文的核心应用主张之一——用 Cosmos-Transfer2.5-2B 为真实机器人演示做视觉增强以扩充策略训练集。具体做法：100 段人类遥操作演示（双臂抓苹果放入碗中的 pick-and-place 任务）→ 用 VLM 对样例视频生成详细 caption → 反复用 Cosmos-Transfer2.5 生成并核对以精修 caption → 把可变成分参数化为槽位模板（[TABLE]、[COLOR_APPLE]、[COLOR_BOWL]、[SENTENCE_LIGHT]、[SENTENCE_BACKGROUND]）→ 用 LLM 为各槽位生成候选变体 → 每段原始演示生成 5 个合成变体（共 500 段），只替换图像观测，动作与关节状态保持不变。控制参数：edge 阈值 medium、blur 阈值 very low、CFG scale 3。
【效果】真机 10 个测试场景各 3 次试验：仅用 100 段真实演示的 base policy 1/30 成功；用传统图像增强（亮度/对比度/饱和度/色相扰动、伽马校正、椒盐噪声、直方图均衡、随机模糊/锐化）的 baseline policy 5/30；用 Cosmos-Transfer2.5 增强的 proposed policy 24/30，在换物体、换桌布、加聚光灯、加干扰物、改背景柜、开抽屉与组合难例上均显著更强。论文据此论证：传统像素级增强无法做语义编辑（改物体颜色、环境外观、光照），而生成式增强可以。
【VLA 合成数据】另有 6.5 节专门讨论用 Cosmos-Predict2.5 为 Vision-Language-Action 模型训练生成合成数据。

### 人工介入程度（人工标注、人工质检、模型初筛+人工复核） ⚠️

数据清洗环节完全自动化，人工介入集中在评测与后训练选型两处，属于「人不进数据流、人进决策环」的模式：
【数据侧：几乎无人工】七阶段 pipeline 全部由模型与规则驱动，无人工标注或人工复核描述。唯一间接的人工痕迹是机器人数据中沿用了原数据集自带的人工标注——GR00T 的「human-labeled success ratings」（人工标注的成功评级）被注入 caption prompt；以及策略学习用的 100 段人类遥操作演示（human teleoperation demonstrations）是人工采集的。
【后训练数据侧的人工筛选】论文在引言中明确「manually curate specialized post-training data tailored for Physical AI」——为 Physical AI 手工精选专项后训练数据，但未说明人工筛选的规则、工作量与人员规模。
【评测与选型侧：大量人工】(1) 每个 SFT 域构建领域专属测试集并做 human preference study（人类偏好研究），以 SFT Win / Base Win / Tie 三分统计；(2) 模型合并环节：先在「a small, hand-picked set of challenging examples」（人工挑选的小规模困难样本集）上做质量评估从 20+ 候选合并模型中选优，再在更大评测集上用 human preference voting 验证；(3) RL 前后做人工投票对比；(4) 与 Wan2.1/2.2 的对比也用人工 win ratio，标注员按 realism（真实感）、visual quality（视觉质量）、temporal consistency（时序一致性）、alignment with conditioning inputs（与条件输入的对齐）四个准则做成对比较。
标注员人数、每样本评价人次、一致性指标均未披露。[不确定：人工评测的标注员规模与规范]

## 音视频对齐

### 音视频同步检测方法（口型同步、事件对齐）

不适用。模型不处理音频，pipeline 中不存在唇同步检测、事件级音画对齐或任何音视频同步环节。该工作中与「对齐」相关的检测全部在视觉/文本域：文本-视频对齐由 VideoAlign 奖励模型的 Text Alignment 维度评估（RL 阶段）；多视图之间的一致性由 TSE 与 CSE 指标评估（Cosmos-Transfer2.5 多视图驾驶模型，见效果对比）；控制信号与生成内容的对齐由车道检测 F1、立方体检测 LET-AP 等下游感知指标间接评估。

### 同步检测具体指标与阈值（SyncNet/Synchformer/LSE/自研，阈值数值，如MOVA LSE-D≤9.5且LSE-C≥4.5、SkyReels-V4 SyncNet |offset|≤3∧conf>1.5）

不适用。无 SyncNet / Synchformer / LSE-C / LSE-D 等任何音视频同步指标与阈值。作为对照，该工作中有明确数值阈值的数据准入规则集中在 Human Dynamics 域：人物需出现在 >40% 的帧中、任一帧可见人数 ≤8 人、至少一人占画面面积 ≥3%；以及时长阈值（丢弃 <5 秒、保留 5–60 秒）与端到端 4% 的保留率。

### 时序同步vs语义同步的分离处理（时间对齐与内容语义匹配作为两个独立过滤条件）

不适用（无音频维度）。若把该概念迁移到纯视觉语境，该 pipeline 确实把「时序结构正确性」与「语义内容正确性」拆成了两个独立过滤级：时序/结构侧由 semantic artifacts filter（类 VTSS）负责，专门剔除劣质转场、画中画等结构异常；语义/内容侧由 content type classifier 与 VLM filter 负责，判断内容类别与是否含不良语义问题。此外镜头切分阶段「排除原始转场」也是纯时序层面的约束。这种分离与 AV 模型中「时序同步 vs 语义匹配」的二分在方法论上同构，但论文未作此类论述。

### 音频质量过滤（SNR、静音检测与静音占比阈值、无音轨剔除、画外音源剔除、背景音乐分离）

不适用。pipeline 无任何音轨处理：不做 SNR 估计、不做静音检测与静音占比阈值、不剔除无音轨样本、不做画外音源判定、不做背景音乐分离。GPU 转码阶段的描述也只针对视觉流。

### 语音/音效/音乐的分类与分别处理策略

不适用。不存在语音/音效/音乐的分类与分流处理。该 pipeline 中与之地位相当的「类型分流」机制是纯视觉的内容类型分类：内部训练的 content type classifier 按 26 类自建 taxonomy 给每条 clip 打标，并作为 sharding 的首要轴用于域均衡采样；以及五大 Physical AI 域的独立 pipeline 分流。

## 训练配合

### 多阶段训练课程与数据课程调度（阶段划分依据：分辨率/时长/质量分/模态；低清→高清、图像→视频、短→长）

训练采用渐进式课程，论文明确其难度沿两条轴递增：pixel resolution（像素分辨率）与 task diversity（任务多样性），时长轴固定不变（始终 93 帧 / 16fps / 约 5.8 秒）。五个预训练阶段（Tab.4）：
· 阶段1：仅 Text2Image，256p（320×192），1 帧 —— 先学会生成高质量单帧，再处理运动与时序一致性；
· 阶段2：Text2Image | Video2World，256p，1 帧 | 93 帧 —— 引入视频，图文联合训练；
· 阶段3：Text2Image | Video2World，480p（832×480）；
· 阶段4：Text2Image | Video2World，720p（1280×704）；
· 阶段5：Text2Image | Video2World | Text2World，720p —— 最后加入无条件帧的 Text2World 任务。
分辨率推进的判据是「advancing to the next stage once the model converges and visual quality plateaus」（模型收敛且视觉质量饱和后才进下一级），而非固定 iteration 数。
【条件帧调度】阶段2–4 中随机采样 1 或 5 个条件帧，模型生成剩余 92 或 88 帧；阶段5 改为按概率采样 0 / 1 / 2 个条件帧，概率分别为 0.5 / 0.25 / 0.25 —— 这是把三种任务统一进同一批次的关键调度设计（条件帧数=0 即 T2W，=1 即 I2W，≥2 即 V2W）。条件方式为 frame-replacement（生成序列前若干帧被条件帧替换），并用 mask token 二元标记哪些是条件输入，损失只作用于待生成帧。
【噪声课程】timestep 从 logit-normal 分布采样，并施加随分辨率增长的 shift：256p 时 β=1，逐步增至 720p 时 β=5，使高分辨率训练偏向更高噪声区（论文解释：高分辨率像素高度相关，噪声不足则模型学不到有意义结构）。另有一处重要修正：观察到帧间突兀不自然的跳变伪影后，判断是高噪声区样本过少，遂改造调度器使 5% 的训练样本显式取自噪声分布最高的 2% 区间，显著减少了转场伪影、改善了时序一致性。
【后续阶段】预训练 → 领域 SFT（五域各训一个模型）→ 4K 冷却（学习率线性衰减至 0）→ 模型合并 → RL（GRPO）→ 时间步蒸馏（rCM，4 步）。
【优化器】AdamW，β1=0.9、β2=0.999，weight decay 0.001；2B 学习率 3e-5、14B 学习率 1.3e-5；线性衰减调度 + 2000 iteration warmup。

### 数据配比随训练阶段的变化（预训练/退火annealing/SFT高质量子集） ⚠️

配比策略最突出的一点是：论文明确选择「不做混合配比」，改用「各域独立 SFT + 事后模型合并」来规避配比调参：
【预训练侧】图像与视频联合训练（Text2Image 任务贯穿全部五个阶段），但图像与视频的采样比例未披露；任务配比通过条件帧数的概率分布隐式控制（阶段5：0/1/2 帧 = 0.5/0.25/0.25）。数据层面则由 sharding 的四轴结构（content type / resolution / aspect ratio / length）支撑「efficient sampling, curriculum-based training, and fine-grained domain balancing」，即配比调控能力被内建在数据组织形态里，但具体配比数值未公布。
【SFT 侧的显式设计取舍】「We fine-tune a separate model for each domain rather than training a single model jointly across all domains. This domain-specific strategy enables us to fully leverage the available data without the need to balance mixture ratios across a combined dataset.」——为每个域单独微调一个模型，而非联合训练，明确理由就是「无需在合并数据集上平衡混合比例」。实测结果是领域 SFT 显著提升本域表现、对通用域仅造成轻微退化，且退化可由后续模型合并进一步弥补。这实质上是把「数据配比问题」转换成了「参数空间合并问题」，是该工作在数据工程上最具启发性的判断。
【合并即配比】用 model merging 统一各域 SFT 模型与 4K 冷却模型，试了 model soup、TIES、DARE-Linear、DARE-TIES 四种方法，每种做超参扫描共生成 20+ 个合并模型；先在人工挑选的小规模困难样本集上选优，再用更大评测集的人工偏好投票验证。结论：除 DARE-Linear 外各法表现相当，最终因简单有效选用 model soup。另一个有价值的发现是「simple grid search over hyperparameters consistently outperforms heuristic selection based on individual fine-tuned models' win rates」——超参网格搜索稳定优于按各微调模型胜率启发式加权。
【退火/冷却】用精选 4K 高清视频做 cooldown，学习率线性衰减至零，作用是增强细粒度视觉细节、产生更平滑的运动——这是典型的高质量小数据退火。
[不确定：图像/视频采样比例、各域在合并权重中的占比数值]

### 后训练数据（SFT精选集规模与筛选标准、偏好对数量与标注方式、reward model训练数据）

后训练数据披露相当具体，分域划分、冷却、RL 三部分：
【SFT 域划分方式】在 InternVideo2 embedding（Wang et al., 2024）上训练一个多头分类器（multi-head classifier），把样本分入五个域。这是「用视频表征模型 + 轻量分类头做数据分域」的可复用范式，比人工规则或 VLM 逐条判断都更廉价。
【五域规模】object permanence（物体恒存）10.4M 条、high motion（高运动）1.0M 条、complex scenes（复杂场景）1.6M 条、driving（驾驶）3.1M 条、robotic manipulation（机器人操作）730K 条；另有 4K 冷却数据 388K 条。注意 object permanence 占绝对多数（10.4M），反映 Physical AI 对「物体不因遮挡而消失」这一基本物理常识的重视。
【SFT 训练配置】每个领域模型训练 30k iterations、batch size 256，超参沿用预训练最后阶段设置。评测显示每个 SFT 模型在其目标域上对预训练基线的人工胜率均显著更高（如机器人操作域 SFT Win 72.6% vs Base Win 8.3%，物体恒存域 50.9% vs 27.7%，高运动域 44.0% vs 34.7%，复杂场景域 42.6% vs 35.4%，驾驶域 47.9% vs 28.8%）。
【RL】用 GRPO 在合并模型（及预训练模型）上做在线强化，因此不存在预标注偏好对数据集——偏好由奖励模型在线给出。奖励模型为 VideoAlign（Liu et al., 2025），一个 VLM-based reward model，同时评估 text alignment、motion quality、visual quality 三维。配置：每个输入条件生成 8 个 rollout、每个用 20 个扩散步；组内归一化 reward 得 advantage；因显存限制把轨迹概率拆解为逐步条件概率之和，每两步计算一次梯度并沿整条轨迹（共 10 步）累积后做一次参数更新；训练 256 steps、batch size 32；用微调数据集上的 diffusion loss 做正则以缓解 reward hacking；最终发布合并模型 RL 后的 EMA 权重作为正式 post-train checkpoint。
【领域特化后训练数据】驾驶多视图模型另用 1.5M 段带 caption 的 7 相机 20 秒 30FPS clip；Cosmos-Transfer2.5 驾驶控制网用投影自 HD 地图与 3D 框的「world scenario map」作控制信号。
【蒸馏】rCM（Zheng et al., 2025）混合前向-反向联合蒸馏，融合连续时间一致性蒸馏与分布匹配蒸馏，蒸馏后 4 步即可生成，PAI-Bench 分数与教师几乎持平（T2W 0.764 vs 0.768，I2W 0.816 vs 0.810，蒸馏版 I2W 甚至略高）。

### 数据处理基础设施与吞吐（NeMo Curator/Data-Juicer/自研，GPU加速比，处理规模，成本） ⚠️

数据基础设施是该论文披露最详尽的维度之一，明确面向 PB 级：
【规模与弹性】基础设施已升级以支持 petabyte-scale（PB 级）数据处理；构建在高度并行化的工作流之上，具备 CPU 与 GPU worker 分配的 dynamic auto-scaling（动态自动扩缩容），确保异构资源上的负载均衡。
【吞吐优化】推理阶段采用 video chunking（视频分块）与 frame-rate control（帧率控制）以减少冗余计算，同时保持语义保真度；转码阶段为 GPU 加速（利用 NVIDIA 硬件编解码器）。
【数据湖与向量库】集成 Delta Lake 湖仓（Databricks, 2019）做大规模 SQL 分析；集成 Milvus（Zilliz, 2019）开源向量数据库做 embedding 检索，支撑语义级视频内容相似度搜索与 caption 级文本 embedding 检索。论文强调这些分析能力不只服务当下训练效率，还为大规模数据集探索、retrieval-augmented training（检索增强训练）与细粒度知识挖掘打基础。
【产品化对应物】该 pipeline 的工程形态即 NVIDIA 独立开源的 Cosmos Curator（github.com/NVIDIA/cosmos-curator，官方描述为「powers the training data generation for Cosmos at NVIDIA」），其 GPU 流式执行框架 Cosmos-Xenna 亦单独开源；同源能力也进入 NeMo Curator 的视频模块。NVIDIA 公开的吞吐口径为：2000 块 H100 一天可处理约 100 万小时 720p 视频（该数字出自 NeMo/Cosmos Curator 官方材料，非本篇论文）。
【训练侧基础设施（非数据处理）】FSDP2 混合分片（逐参数分片、异步分布式 checkpoint、meta-device 初始化，取自 TorchTitan）；Ulysses 式灵活上下文并行（相比 Cosmos-Predict1 的 ring attention 更简单、通信更省，且更好支持 NATTEN 稀疏注意力与带 JVP 的 fused flash attention；图像迭代时动态关闭上下文并行、视频批次再开启）；细粒度选择性激活重计算 SAC；Elastic Reward Service 弹性奖励服务（decode 与多奖励模型推理流水线并行、CUDA IPC 零拷贝、Redis + UUID 异步取结果、支持批处理）。效率数据：4096 块 NVIDIA H100、720p、93 帧下，2B 模型 MFU 36.49%（上下文并行度 2），14B 模型 MFU 33.08%（上下文并行度 8，因通信开销更大而下降）。
[不确定：本篇论文未直接给出 curation 的处理耗时/成本数值]

## 效果对比

### 数据策略消融的量化影响（区分：过滤严格度ablation / caption密度风格ablation / 数据配比ablation，及对应评测指标）

论文没有做严格意义上的「控制变量数据消融」（没有同配置下换数据集重训的对照实验），但提供了三类可量化的数据策略证据：
【① 过滤严格度：跨代对比而非同代消融】Cosmos-Predict2.5 相对 Cosmos-Predict1 把 clip 存活率从 30% 压到 4%，同时原始素材从 20M 小时扩到 35M 小时。整体收益体现在 PAI-Bench 上：Cosmos-Predict2.5-2B 预训练模型 T2W Overall 0.751、I2W Overall 0.799，post-train 后分别到 0.768 / 0.810；14B 预训练 0.757 / 0.806，post-train 0.768 / 0.810。但因架构（flow matching 替代 EDM）、文本编码器（Cosmos-Reason1 替代 T5）同时变更，无法把增益归因到数据过滤单项——这是该证据的主要局限。
【② 数据配比策略：领域 SFT 的人工胜率】五域 SFT 相对预训练基线的人工偏好胜率（SFT Win / Base Win / Tie）：机器人操作 72.6% / 8.3% / 19.0%；物体恒存 50.9% / 27.7% / 21.4%；驾驶 47.9% / 28.8% / 23.3%；高运动 44.0% / 34.7% / 21.3%；复杂场景 42.6% / 35.4% / 22.0%。可读出的规律是：域越窄、与通用预训练分布差异越大（机器人操作），领域数据的边际收益越高。
【③ 模型合并作为配比替代方案的验证】对比 Model Soup、TIES、DARE-TIES 与基础模型在六个维度（五个专项域 + General）上的胜率雷达图，结论是各合并法（除 DARE-Linear）表现相当，且合并模型在保持通用域性能的同时取得各域优势（「gets the best of all the worlds」）。这间接验证了「不调混合比例、改用参数合并」这一路线的可行性。
【④ 合成数据增强的严格对照（下游任务）】机器人策略学习实验是全文唯一的严格控制变量对照：同一任务、同样 10 个测试场景各 3 次试验，base policy（仅 100 段真实演示）1/30、传统图像增强 baseline 5/30、Cosmos-Transfer2.5 生成式增强 24/30。这是「数据增强策略」层面最有力的量化证据。
【⑤ RL 的数据/奖励侧收益】VideoAlign 奖励在 RL 前后：2B 预训练模型 T2W 总分 1.08→1.69、I2W 0.23→0.42；合并模型 T2W 1.23→1.74、I2W 0.24→0.45；人工投票 RL 胜率 40.0%（预训练+RL）与 46.7%（合并+RL），均优于 RL 前。
【未做】caption 密度/长度（short vs medium vs long）的消融、5 秒窗口切分粒度的消融、各过滤器单项开关的消融、26 类 taxonomy 配比的消融——这些是该报告在数据消融上的明显空缺。

### 质量vs数量的证据（小而精数据超越大而杂的案例） ⚠️

有较强的策略性证据但缺乏严格对照实验：
(1) 最直接的表述是跨代策略切换——把 clip 存活率从 30% 主动压到 4%（保留量约为原策略的 1/7.5），同时把原始素材池扩大 1.75 倍，本质是「用更大的漏斗口径换更小更干净的出口」。论文自评这带来「a dataset that is larger, cleaner, and semantically richer」以及「enhanced pre-training efficiency and improved downstream generalization」。
(2) 参数效率上的间接证据：Cosmos-Predict2.5-2B 在 PAI-Bench I2W 上（0.810）超过 Wan2.2-27B-A14B（0.806）与 Wan2.1-14B（0.797）；人工评测中 2B 模型在体积比 Wan2.2-5B 小 60.0%、比 Wan2.1-14B 小 85.7% 的情况下，仍略优于 Wan2.2-5B（30.0% vs 26.2%）、与 Wan2.1-14B 相当（33.0% vs 34.8%）；14B post-train 模型对 Wan2.1-14B 人工胜率 48.6% vs 31.8%，对参数量翻倍的 Wan2.2-27B-A14B 达 38.1% vs 35.9%。团队将此归因于更严格的数据与更好的文本编码器，但未做数据单因子拆解。
(3) 冷却阶段仅用 388K 条精选 4K 视频即可提升细节精度与运动平滑度，是典型的小规模高质量数据收尾。
(4) 机器人策略实验中 100 段真实演示 + 500 段生成式增强样本达成 24/30，而 100 段真实 + 无限次传统像素增强仅 5/30 —— 说明增强样本的语义多样性质量远比数量重要。
仍缺少「相同数据量下严格 vs 宽松过滤」的直接对照。[不确定：4% 保留率相对 30% 的单因子增益无法量化]

### 训练数据domain分布与评测基准类目体系的对齐关系（如VABench七大类） ⚠️

训练数据的域体系与评测基准的类目体系呈现明显但非严格声明的对齐关系，且这是该工作相对少见的优势：
【评测基准】主评测为 PAI-Bench（Zhou et al., 2025，Physical AI Bench，NVIDIA 同期工作），其 predict 任务报两个分数：Domain Score 通过 VQA 式评估覆盖七个域——av（自动驾驶）、common（通用）、human（人类）、industry（工业）、misc（杂项）、physics（物理）、robotics（机器人）；Quality Score 由改编自 VBench 的八个 T2V/I2V 指标构成；Overall Score 为二者平均。
【与训练数据域的对应】PAI-Bench 的七个 Domain 与论文第 2.2 节的五大专项数据域几乎一一对应：av↔Autonomous Driving、robotics↔Robotics、human↔Human Dynamics、industry↔Smart Spaces（仓库/工厂/工地）、physics↔Physics、common/misc↔通用预训练语料。这说明数据 curation 的域切分与评测类目体系是共同设计的（同为 NVIDIA 内部工作），是「训练数据 taxonomy 与 benchmark taxonomy 对齐」的一个清晰范例。不过论文未显式声明这种对应关系，也未给出两侧各类目的占比对照。
【与后训练五域的错位】需注意 SFT 的五域划分（object permanence / high motion / complex scenes / driving / robotic manipulation）与 PAI-Bench 七域并不一致——前者混合了「物理能力维度」（物体恒存、高运动、复杂场景）与「应用域维度」（驾驶、机器人），后者纯按应用域划分。SFT 侧另建了领域专属测试集做人工偏好评测。
【其他评测】多视图驾驶用 RDS-HQ-HL 数据集评 FVD StyleGAN / FVD I3D / FID 与多视图一致性 TSE / CSE，并用下游感知任务（车道检测 F1、x 坐标 rMSE、类别准确率；立方体 LET-AP/APL/APH）评估控制信号遵循度——这种「用感知模型验证生成内容的结构正确性」的评测方式与其结构化标注体系是配套的。
[不确定：训练侧 26 类 taxonomy 与 PAI-Bench 七域的精确映射关系]

## 不确定字段

以下字段的调研信息部分不确定（⚠️ 标注来源）：

- provenance_licensing
- duration_distribution
- resolution_aspect_distribution
- domain_distribution
- language_accent_distribution
- data_scale
- funnel_retention_rate
- shot_segmentation
- quality_filtering
- motion_filtering
- deduplication
- safety_filtering
- caption_model
- caption_structure
- human_in_loop
- stage_data_mixture
- data_infra_throughput
- quality_over_quantity_evidence
- benchmark_taxonomy_alignment
