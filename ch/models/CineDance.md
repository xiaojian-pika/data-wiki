# CineDance / CineDance-1M（论文标题：CineDance: Towards Next-Generation Multi-Shot Long-Form Cinematic Audio-Video Generation）。该工作包含三个产出：CineDance-1M 数据集（100 万条多镜头长篇音视频序列）、CineBench 评测基准（1000 条测试样例 + 六维度指标体系）、CineDance 生成模型（基于 LTX-2.3 改造的开源基线）。

> 主题: 视频生成模型（含音视频同时生成）的数据处理：数据清洗流程、数据分布、打标方式

[← 返回首页](../index.md)

**目录**: [基本信息](#基本信息) · [数据规模与分布](#数据规模与分布) · [清洗流程](#清洗流程) · [打标方式](#打标方式) · [音视频对齐](#音视频对齐) · [训练配合](#训练配合) · [效果对比](#效果对比)

## 基本信息

### 名称

CineDance / CineDance-1M（论文标题：CineDance: Towards Next-Generation Multi-Shot Long-Form Cinematic Audio-Video Generation）。该工作包含三个产出：CineDance-1M 数据集（100 万条多镜头长篇音视频序列）、CineBench 评测基准（1000 条测试样例 + 六维度指标体系）、CineDance 生成模型（基于 LTX-2.3 改造的开源基线）。

### 发布机构/公司 ⚠️

学术界多机构联合，无企业实验室署名。参与单位包括：上海交通大学（Shanghai Jiao Tong University）、电子科技大学（University of Electronic Science and Technology of China）、浙江大学（Zhejiang University）、东京大学（The University of Tokyo）、南洋理工大学（Nanyang Technological University）。作者列表：Yuheng Chen、Teng Hu、Yuji Wang、Qingdong He、Zhucun Xue、Qianyu Zhou、Jason Li、Lizhuang Ma、Jiangning Zhang、Dacheng Tao。项目主页维护者为一作 Yuheng Chen（github.com/AliothChen）。具体每位作者对应哪个机构未在页面明确列出。[不确定]

### 发布时间（技术报告/论文/开源时间）

2026年6月8日首次提交 arXiv（arXiv:2606.09639 v1），2026年6月11日更新 v2。GitHub 仓库与项目主页同期上线。数据集第一批分片（CineDance_01）于 2026年7月前后在 HuggingFace 以 gated 门控方式放出。代码（curation pipeline、推理、训练）与模型权重截至 2026年7月仍标注为「pending release」，尚未发布。

### 类型（模型/数据集/工具链/评测基准）

以数据集为主体的复合型工作，三位一体：
【主产出】数据集 —— CineDance-1M，首个面向多镜头长篇（multi-shot long-form）音视频联合生成的 1080p 大规模开放研究数据集，规模 1,021,657 条叙事序列 / 约 26.3K 小时。
【副产出 1】评测基准 —— CineBench，1000 条分层测试 prompt + 六维度人类对齐指标体系。
【副产出 2】模型 —— CineDance，由 LTX-2.3 适配而来的开源基线（13B 视频 + 3B 音频 + 3B 跨模态注意力），用于验证数据集有效性。
【副产出 3】工具链 —— 三阶段数据策展 pipeline（清洗 / 叙事解析 / 双模态标注），论文承诺开源但尚未释出。

### 开源程度（权重/代码/数据/pipeline各自是否开源） ⚠️

开放程度中等偏上，但截至 2026年7月仍处于「分批释出中」状态：
【数据】部分开放。HuggingFace 上以 gated（门控申请）方式发布，首批 CineDance_01 为四分片中的第 1 份，含约 240,488 条视频片段、150 个 TAR 归档、总计 5.83 TB，目前仅含视频本体（video only，原生音轨保留在视频容器内），结构化标注文件尚未随首批放出。许可为 CC-BY-NC-SA-4.0（署名-非商业性使用-相同方式共享），明确限定非商业研究与教育用途。下载需 HuggingFace token 且需人工审核申请。
【代码】未开源。GitHub 仓库 github.com/AliothChen/CineDance 已建立，但 curation pipeline 代码、推理套件、推理代码、训练代码均列为待发布（pending）。
【权重】未开源。CineDance 模型 checkpoint 列为待发布。
【pipeline】方法层面披露充分——三阶段流程的每一级工具选型（EasyOCR、FFmpeg 黑边检测、TransNetV2、Qwen3.5-27B、Qwen3.5-35B-A3B、Qwen3-Omni-30B-A3B）、漏斗各级定量数据、标注 schema 字段、消融对比表均在正文给出，可被第三方重新实现；但具体 prompt 原文与过滤阈值数值未完整公开。
【依赖声明】README 致谢 LTX-2、Qwen 系列、vLLM。仓库本身未标注 code license。[不确定]

### 是否支持音视频同时生成，以及实现方式（原生联合/级联/MoE融合）

支持音视频同时生成，属于「原生联合生成」路线（native joint audio-video generation），而非先出视频再配音的级联方案。
【数据侧定位】数据集的核心卖点之一即「保留原生音轨」（native audio track），论文将「缺失声学模态」列为现有数据集四大缺陷之一，所有序列均带原始同步音频，标注同时覆盖视觉与听觉两条轨道。
【模型侧实现】CineDance 模型基于 LTX-2.3，架构为 13B 视频 DiT + 3B 音频分支 + 3B 跨模态 cross-attention 模块，视频与音频在同一扩散过程中通过跨模态注意力耦合，属于双塔 + 跨注意力融合的原生联合架构，非 MoE 融合、非级联后配音。
【任务定义】论文将任务表述为 T2AV（Text-to-Audio-Video），即由文本提示一次性生成带同步音轨的多镜头长视频。

### 调研信息来源列表（论文/技术报告/官方文档/新闻的URL，标注每条来源的性质：官方一手/同团队旁证/第三方报道）

1. 【官方一手】论文 arXiv 摘要页 https://arxiv.org/abs/2606.09639 —— 标题、作者、v1/v2 提交日期、摘要。
2. 【官方一手】论文全文 HTML https://arxiv.org/html/2606.09639v2 —— 三阶段 pipeline、漏斗表（Tab.3）、标注 schema、CineBench 六维度、消融表（Tab.4/5/7）、数据集横向对比表（Tab.6）。
3. 【官方一手】论文 PDF https://arxiv.org/pdf/2606.09639
4. 【官方一手】项目主页 https://aliothchen.github.io/projects/CineDance/ —— 五家参与机构、数据集规模概述、各资源入口。
5. 【官方一手】GitHub 仓库 https://github.com/AliothChen/CineDance —— 开源进度 checklist（数据集 gated 分批释出、代码与权重 pending）。
6. 【官方一手】HuggingFace 数据集卡 https://huggingface.co/datasets/CineDance/CineDance_01 —— 首批分片规模 5.83TB / 240,488 clips / 150 TAR、CC-BY-NC-SA-4.0 许可、gated 访问、video-only 现状、局限性声明。
7. 【第三方索引】awesome-video-generation 列表 https://github.com/kongzhecn/awesome-video-generation —— 收录旁证。

## 数据规模与分布

### 训练数据量级（视频条数/小时数/token数，预训练与SFT分开） ⚠️

作为数据集发布，无预训练/SFT 的传统划分，规模数据以策展漏斗终点计：
【最终规模】1,021,657 条叙事序列（narrative sequences），总时长约 26.3K 小时（2.63 万小时）。
【原始采集】45,181 条长视频，总时长 32.8K 小时。
【中间产物】TransNetV2 切出 25,899,474 个原子镜头（atomic shots）；经叙事分组得到 1,201,912 条叙事序列。
【单序列特征】平均时长 92.8 秒，平均含 24.2 个连续镜头，最低空间分辨率 1080p。
【标注体量】平均每条视频 6,496.3 个词的结构化双模态标注（Tab.6），标注密度在同类数据集中处于数量级领先。
【首批开放量】HuggingFace CineDance_01 约 240,488 条片段 / 5.83 TB，为四分之一分片。
【模型训练用量】CineDance 模型两阶段训练各自使用的样本条数、batch size、epoch、学习率、总 token 数均未披露。[不确定]

### 数据来源构成（自有/公开数据集/网络爬取/授权采购/合成数据） ⚠️

以公开数据集二次策展 + 自主采集为主，未提及授权采购或合成数据：
【公开数据集来源】明确列出复用 MiraData、LVD-2M、Koala-36M 三个已有大规模视频数据集作为素材池之一。
【采集流程借鉴】声明遵循 SkyReels-V2 与 OpenHumanVid 的数据采集 pipeline 规范进行网络素材收集。
【内容类型】以电影 / 影视 / 剧情类长视频为主（cinematic），强调「故事片式」的多镜头叙事内容，故原始素材以长片而非 UGC 短视频为主体——原始仅 45,181 条视频却撑起 32.8K 小时，平均单条约 43 分钟，印证素材为长片级内容。
【硬性准入】最低 1080p 空间分辨率，且必须自带原生音轨。
【合成数据】无。全部为真实拍摄素材，未构造任何合成或编辑样本。
【各来源具体占比】未给出 MiraData / LVD-2M / Koala-36M / 自采各自的贡献比例。[不确定]

### 数据合规与溯源（授权数据占比、rights-cleared数据集、C2PA等） ⚠️

合规披露较为薄弱，是该工作相对明显的短板：
【发布许可】数据集在 HuggingFace 以 CC-BY-NC-SA-4.0 发布，限定非商业性研究与教育用途，并要求相同方式共享。
【访问控制】采用 gated access 门控机制，需 HuggingFace 账号 token 并经作者人工审核批准方可下载，属于对下游滥用的一层准入约束。
【上游版权】素材主体来自网络采集的影视内容与既有公开数据集（MiraData / LVD-2M / Koala-36M），论文未讨论上游影视素材的版权授权状态、rights-cleared 比例，也未提及 C2PA 等内容来源认证标准。
【风险免责】数据集卡明确声明「自动与人工策展无法保证移除每一条低质、敏感或其他不良样本」，将合规责任部分转移给使用者，要求使用者按自身场景补充质检。
【授权数据占比 / 采购数据】无相关披露，推断为零。[不确定]

### 片段时长分布与切分策略 ⚠️

时长分布是该数据集最核心的差异化维度，并配有专门的「防碎片化」策略：
【平均时长】92.8 秒/序列，是同类多镜头数据集的数倍（对比 MiraData 72.1s、LVD-2M 20.2s、SpeakerVid-5M 8.3s）。
【切分策略】不采用「镜头即样本」的传统切法，而是两级：先用 TransNetV2 切出 2589 万个原子镜头，再按电影理论规则将连续镜头「自底向上」重组为叙事序列（narrative sequence），序列而非单镜头才是最终样本单位。
【最短时长约束】从人工参考集实测得到叙事完整性的经验最小时长为 18.4 秒，据此把软阈值设为 20 秒，以阻止解析出无意义的过短片段。消融显示最终数据中仅 3.1% 的序列短于 20 秒。
【时长剪裁】MLLM 引导的时序截断会去掉长视频首尾的片头/片尾内容，截断长度公式为 t = max(5 分钟, 0.1L)，L 为原视频总时长。
【分布图】论文 Fig.5 给出时长与镜头数的联合分布直方图，但具体分位数（P50/P90、最长最短）未以文字列出。[不确定]

### 分辨率/宽高比分布与分桶策略 ⚠️

【分辨率下限】硬性要求最低 1080p，是该数据集相对同类工作的关键卖点之一——论文对比表（Tab.6）中，同时满足「1080p + 原生音轨 + 镜头级双模态密集标注」三项的仅 CineDance-1M 一家。
【空间预处理】采用「粗裁 + 细验」两道：粗裁阶段用 EasyOCR 定位并裁除烧录字幕区域，用 FFmpeg 黑边检测（black-border detection）裁除信箱式黑边（letterboxing）；镜头切分完成后再对每个 clip 级片段复跑一次 OCR 与黑边检测做细粒度复验。
【宽高比分布 / 分桶策略】论文未给出宽高比的统计分布，也未描述训练时的分辨率分桶（bucketing）策略；由于素材以影院级影视内容为主，推测以宽银幕比例为主但无数据支撑。[不确定]

### 类别/domain分布与配比策略（人物、动作、场景、风格等比例控制与概念均衡） ⚠️

采用八维分类体系（taxonomy）保障多样性，但未公布各类目的定量占比：
【八个维度】Genre（题材类型）、Format（形式/载体）、Region（地域）、Modality（模态/表现形式）、Story Logic（叙事逻辑）、Era（年代）、Tone（情绪基调）、Audience（受众定位）。
【设计意图】该体系用于在采集与筛选阶段主动铺开覆盖面，避免语料集中于单一题材或单一年代，服务于「电影级叙事」这一核心场景。
【与评测的关系】CineBench 的 1000 条测试 prompt 也按 Theme/Style（主题风格）× Duration/Shot Count（时长与镜头数）× Difficulty（难度）三轴做分层抽样，其中主题风格轴与训练侧 taxonomy 相呼应。
【配比控制策略】论文未描述基于该 taxonomy 的显式配比控制、概念均衡或重采样机制，八维体系更像是事后的多样性刻画而非事前的采样约束。
【各维度下的类目清单与百分比】未公布。人物/动作/场景/风格等细粒度比例亦无数据。[不确定]

### 音频类别分布与配比（语音/音效foley/音乐/环境音/静音的比例及控制策略）——AV模型独有维度 ⚠️

音频被显式拆解为三类并分别标注，但未公布各类的数量占比：
【三类划分】在镜头级音频 prompt 中，音频内容被分解为 music（音乐）、ambient sound（环境音）、effects（音效）三类分别描述；对白/语音则通过独立的 ASR 通道处理，形成事实上的「语音 / 音乐 / 环境音 / 音效」四类并行标注结构。
【与说话人的关系】另有独立的 character voice description（角色音色描述）字段，刻画每个角色的嗓音特征。
【质量侧刻画】音频质量以 DNSMOS（信号保真度）与 CLAP embedding 的时序方差（衡量音频内容随时间的变化丰富度，可间接反映静音/单调音轨）两项指标量化，均作为元数据保留。
【占比与配比策略】论文未给出语音 / 音乐 / 音效 / 环境音 / 静音各自的时长或样本占比，也未描述任何主动的音频类别配比控制策略。这是该数据集在 AV 维度上披露不足之处。[不确定]

### 叙事结构分布（单镜头vs多镜头、平均clip时长、镜头数分布、是否含原生音轨） ⚠️

叙事结构是本数据集的立身之本，也是论文着墨最多的部分：
【多镜头】平均 24.2 个连续镜头/序列，全部为多镜头样本，与 LVD-2M（1.86 镜头）、SpeakerVid-5M（1.27 镜头）等以单镜头为主的数据集形成代际差。
【平均序列时长】92.8 秒；对比 MiraData 72.1s / 7.15 镜头、LVD-2M 20.2s / 1.86 镜头。
【原生音轨】全量保留原始同步音轨，为对比表中唯一同时具备 1080p + 原生音轨 + 镜头级双模态密集标注的数据集。
【叙事序列的定义】「diegetic time and causality 的连续流」——即在戏剧时间与因果链上连续、且角色与环境状态保持延续的一段镜头组合，允许空间跳转但不允许叙事断裂。
【四条电影理论解析规则】① Multi-angle shots（多角度镜头）：仅机位角度变化而事件统一；② Cross-cutting（平行剪辑）：在不同空间间快速交替但被统一的因果张力绑定；③ Causal action / ellipsis（因果动作与省略）：存在时空跳跃但后续事件可直接、可解释地承接前一事件；④ Montage（蒙太奇）：镜头彼此割裂但被宏观主题或情绪弧线统一。
【镜头级属性】每个镜头标注五维属性：景别（scale）、机位角度（angle）、运镜（movement）、叙事功能（narrative function）、时长类别（duration category），外加转场类型（shot transition type）。
【分布细节】Fig.5 给出时长-镜头数联合分布，但镜头数的具体分位数未文字化。[不确定]

### 语言/口音分布（多语种唇同步能力的数据基础） ⚠️

【体系层面】八维 taxonomy 中包含 Region（地域）维度，间接覆盖语种与文化区域的多样性诉求。
【技术链路】ASR 采用 Qwen3-Omni-30B-A3B，该模型本身具备多语种能力；CineBench 评测侧的 WER/CER 使用 Whisper-large-v3 计算，同样为多语种模型，说明数据与评测均非单语设定。
【说话人属性】提供角色音色描述（character voice description）与 ASR-to-Character 绑定（语音句子归属到具体角色 anchor token），但标注 schema 中未见显式的语言标签、口音标签、情绪标签字段。
【定量分布】论文完全未给出语种占比、口音分布或中英文比例等任何统计数字。[不确定]

## 清洗流程

### 清洗漏斗整体结构（几级过滤、各级顺序）

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

### 漏斗定量保留率（每级过滤的输入/输出量与最终保留率，如Apollo 27%）

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

### 镜头切分方法（PySceneDetect/自研模型/shot-aware splitting）

【工具】TransNetV2，业界成熟的深度学习镜头边界检测模型（相较 PySceneDetect 这类基于像素直方图的传统方法，对渐变转场、溶解、划像更鲁棒），从 44,579 条清洗后视频中切出 25,899,474 个原子镜头。
【关键差异化】切分只是中间步骤而非终点。传统数据集把「一个镜头 = 一个样本」，本工作在镜头之上再做一层「自底向上的叙事分组」，用 Qwen3.5-27B 依据四条电影理论规则（多角度、平行剪辑、因果动作/省略、蒙太奇）把语义连贯的相邻镜头重组为 1,201,912 条叙事序列，序列才是最终样本。
【抗幻觉设计】不让 LLM 直接输出时间戳，而是先把镜头编号索引化，让 LLM 输出镜头编号的分组结果（bottom-up shot indexing），大幅降低时间戳幻觉。
【长视频推理】上下文感知滑动窗口，窗口约 3 分钟，且窗口切分点强制对齐到镜头边界，避免把一个镜头劈成两半。
【抗碎片化】人工参考集实测叙事完整性最短需 18.4 秒，故设 20 秒软阈值；最终仅 3.1% 序列短于 20 秒。
【解析质量】Qwen3.5-27B + 自底向上策略在解析任务上取得 F1 = 88.4%（Tab.4）。

### 质量过滤（美学评分、清晰度、OCR文字过滤、黑边/水印/logo检测） ⚠️

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

### 运动过滤（光流/运动分数阈值、静态与抖动剔除） ⚠️

【指标】使用 AMT 计算 Motion Smoothness（运动平滑度）作为运动维度的量化刻画，与 DOVER 的美学/技术分并列存入元数据。
【策略】同样遵循「不硬剪枝」原则，运动分数不设过滤阈值，仅作为元数据供下游按需筛选。
【间接的静态剔除】人工伪影审计的目标清单中包含「静帧停留（still-frame holds）」与「转场特效（transition effects）」，属于对静止/异常运动内容的定性把关，但非基于光流阈值的自动过滤。
【未采用】论文未提及光流（optical flow）计算、运动幅度阈值、抖动（camera shake）检测等常见运动过滤手段，也未给出任何运动分数的分布或阈值数值。这是相对其他大规模数据集较弱的一环。[不确定]

### 去重方法（精确去重与基于embedding的语义去重分别记录） ⚠️

论文未描述任何去重环节——既无基于哈希/指纹的精确去重，也无基于 embedding（如 CLIP/ImageBind 特征）的语义去重，正文的三阶段 pipeline 中不含去重步骤。
【推测性缓解因素】① 原始素材仅 45,181 条长视频，条目数量级不大，长片之间的完全重复概率相对较低；② 上游 MiraData / LVD-2M / Koala-36M 各自已做过去重；③ 叙事序列由同一部长片内的连续镜头组成，同片内不同序列天然不重叠。
【潜在风险】跨来源数据集之间（MiraData 与 Koala-36M 可能都收录同一部影片）的重叠未见处理说明。整体属于披露缺失。[不确定]

### VLM/LLM作为质检员（多模态大模型质量打分与错配剔除，2026年从浅层打分器转向大模型语义判断的趋势）

大模型深度介入是该 pipeline 的核心特征，且不止用于打分，更承担了「结构化语义解析」这一重任务，充分体现 2026 年从浅层打分器转向大模型语义判断的趋势：
【MLLM 用于时序截断】阶段一用多模态大模型判断长视频的片头/片尾边界，决定截断位置。
【LLM 用于叙事解析（最重的一环）】Qwen3.5-27B 依据四条电影理论规则判断哪些相邻镜头构成一条完整叙事序列——这已超出「质检」范畴，是把大模型当作电影语法解析器使用。关键工程手段是自底向上镜头索引（避免时间戳幻觉）与镜头边界对齐的 3 分钟滑窗。Tab.4 消融显示该组合达 F1 = 88.4%，且仅 3.1% 序列短于 20 秒软阈值。
【MLLM 用于视觉标注】Qwen3.5-35B-A3B（MoE 架构，35B 总参 / 3B 激活）产出镜头五维属性与描述。
【Omni 模型用于音频标注】Qwen3-Omni-30B-A3B（30B 总参 / 3B 激活）承担 ASR、音频 prompt、角色音色描述三项，并做 ASR-to-Character 绑定。
【抗幻觉的三项工程设计】① 音频标注拆成三个子任务分别调用，而非一次性输出（降低幻觉）；② ASR 阶段刻意不做说话人-角色绑定，绑定作为独立后续步骤；③ 绑定采用窗口化方案（过滤非语音区间、保持镜头完整与句子完整），使 Qwen3-Omni-30B-A3B 的绑定准确率从整片输入的 67.2% 提升到 95.4%。
【与传统 diarization 的对比（Tab.5/7）】窗口化 Qwen3-Omni 95.4% > Gemini 系列 82.8%~87.4% > DiariZen 63.1% > Pyannote-3.1 62.7%，验证了「用 Omni 大模型替代专用 diarization 工具」的路线优势。
【质量打分侧】DOVER/DNSMOS/ImageBind/SyncNet 等模型打分器全量运行但不做硬裁剪，仅存元数据。

### 安全与合规过滤（NSFW、版权、人脸/隐私） ⚠️

安全与合规过滤是该工作披露最薄弱的维度：
【论文层面】三阶段 pipeline 中不含 NSFW 检测、暴力内容过滤、人脸隐私处理或版权检测环节的任何描述。人工审计的伪影清单也全部是画质类问题（字幕、水印、logo、黑边、录屏等），不含安全类目。
【发布层面的替代性约束】① 采用 gated access 门控发布，需人工审核申请者；② 许可限定为 CC-BY-NC-SA-4.0 非商业用途；③ 数据集卡明确免责——「自动与人工策展无法保证移除每一条低质、敏感或其他不良样本」，要求使用者按自身应用场景自行做质检。
【判断】安全责任事实上通过门控 + 免责声明转移给了下游使用者，而非在 pipeline 内解决。考虑到素材为影视内容（可能含暴力、成人情节），这是实际使用时需重点补足的环节。[不确定]

## 打标方式

### 使用的caption模型（自研VLM/开源模型，模型规模） ⚠️

全链路使用 Qwen 系列开源大模型，无自研 caption 模型，推理框架致谢 vLLM：
【叙事解析骨干】Qwen3.5-27B —— 承担镜头分组与叙事边界判定，Tab.4 消融中对比了不同 Qwen 规模的骨干，27B 版本配合自底向上策略取得最优 F1 = 88.4%。
【视觉标注模型】Qwen3.5-35B-A3B —— MoE 架构（约 35B 总参数、3B 激活参数），产出镜头级五维属性、转场类型、局部角色表、活跃场景、镜头描述与转场描述。
【音频标注模型】Qwen3-Omni-30B-A3B —— 全模态 MoE 模型（约 30B 总参 / 3B 激活），同时承担句级 ASR 抽取、镜头级音频 prompt（音乐/环境音/音效）生成、角色音色描述三项子任务，并完成 ASR 句子到角色 anchor token 的绑定。
【选型逻辑】视觉与音频用不同专精模型分工，且均选 MoE 稀疏架构（3B 激活），在百万级序列的标注吞吐与质量之间取平衡；ASR 与音频描述不外挂 Whisper 等专用模型，而统一交给 Omni 模型，理由是 Tab.5/7 显示其在说话人归属任务上远超 Pyannote-3.1（62.7%）与 DiariZen（63.1%）等专用 diarization 工具。
【评测侧另用】CineBench 的 WER/CER 计算使用 Whisper-large-v3。
【未披露】标注 prompt 原文、单模型推理成本、是否对 Qwen 做过任务微调。[不确定]

### caption密度与结构化程度（短/长/密集描述、结构化字段如镜头运动、风格标签）

标注密度与结构化程度是该数据集相对现有工作的最大代差，论文称之为「可配置的分层双模态标注（Configurable / Hierarchical Dual-Modal Annotation）」：
【密度】平均每条视频 6,496.3 个词的结构化标注（Tab.6），远超 MiraData 等仅有视频级 caption 的数据集，且是同类中唯一提供镜头级密集标注的。
【两层结构】
  · 全局层（Global）：全局角色表 [⟨char₁⟩,…,⟨charₙ⟩] 与全局场景表 [⟨scene₁⟩,…,⟨sceneₘ⟩]，以 anchor token 形式定义；
  · 镜头层（Shot-level）：每个镜头的描述显式引用上述 anchor token，实现跨镜头的身份与场景绑定。
【Anchor Token 机制的价值】这是解决多镜头一致性的关键设计——生成模型可据此知道「第 3 镜头的 ⟨char₂⟩ 与第 17 镜头的 ⟨char₂⟩ 是同一人」，评测侧 CineBench 也直接用该 token 做身份连续性（ArcFace 聚类）与场景连续性（DINOv2 余弦相似度）的判定锚点。
【镜头级结构化字段】五维镜头属性：scale（景别）、angle（机位角度）、movement（运镜）、narrative function（叙事功能）、duration category（时长类别）；外加 shot transition type（转场类型）、localized character list（该镜头出现的角色）、active scene（该镜头所处场景）、shot description（镜头描述）、transition description（转场描述）。
【「可配置」的含义】所有字段与质量分均可组合筛选，用户能按需拼装出 task-specific 子集（例如只要含运镜标注的、只要高美学分的）。
【模型侧使用格式】训练时组织为：全局头部（角色/场景定义）+ 逐镜头块 [SHOT i | scene sᵢ | camera κᵢ] ⊕ 转场描述 div ⊕ 对白 dia ⊕ {(说话人 spkᵢ,ℓ, 台词 speechᵢ,ℓ)}。

### 音视频联合caption结构（是否同时覆盖视觉+听觉轨道、是否分流为独立字段，如LTX-2全音景描述、Script-a-Video factorized streams、Foley-Omni三字段）

音视频联合 caption 采用「双轨并行、字段分流」设计，视觉与听觉各成体系但通过共享的 anchor token 与镜头索引对齐：
【视觉轨（Qwen3.5-35B-A3B 产出）】镜头五维属性（景别/角度/运镜/叙事功能/时长类别）、转场类型、局部角色表、活跃场景、镜头描述、转场描述。
【听觉轨（Qwen3-Omni-30B-A3B 产出，三字段分流）】
  ① 句级 ASR 转写（sentence-level ASR segments）——此阶段刻意不绑定说话人身份；
  ② 镜头级音频 prompt——覆盖 music（音乐）、ambient sound（环境音）、effects（音效）的自然语言描述，即非语音音景（soundscape）描述；
  ③ 角色音色描述（character voice description）——刻画每个角色的嗓音特征。
【两轨的耦合点】① 共用全局角色 anchor token ⟨charₖ⟩：ASR 句子在后续绑定步骤中挂到具体角色 token 上，从而与视觉轨中该角色出现的镜头对齐；② 共用镜头索引：音频 prompt 是镜头级的，与视觉描述逐镜头一一对应。
【与同类方案对比】相比 LTX-2 的「全音景单段描述」更结构化；与 Script-a-Video 的 factorized streams（分流式多流脚本）、Foley-Omni 的三字段设计属同一思路谱系，但 CineDance 的特色是把 anchor token 一致性机制引入到跨镜头长序列场景，并把「说话人-角色绑定」作为独立可评测的子任务（95.4% 准确率）。
【设计动机】三个音频子任务分开调用而非一次性输出，论文明确说明是为了降低幻觉（reduce hallucination）。

### 对白转写与说话人属性标注（ASR转写+说话人身份/语言/口音/情绪） ⚠️

对白处理是该数据集在 AV 维度上做得最细的部分：
【ASR 转写】由 Qwen3-Omni-30B-A3B 完成句级（sentence-level）转写，而非整片流式输出，切分粒度对齐句子边界。
【说话人身份绑定（核心贡献）】设计为独立于 ASR 的后续步骤，把每条 ASR 句子绑定到全局角色 anchor token ⟨charₖ⟩。采用窗口化（windowed）方案：先过滤掉非语音区间，窗口切分同时保证镜头不被切断、句子不被截断。
【定量效果（Tab.5 / Tab.7）】在 100 条人工标注片段的基准上：
  · Qwen3-Omni-30B-A3B + 窗口化：95.4%（整片输入仅 67.2%，滑窗 prompt 83.1%，整片 56.4%——不同表中口径略有差异）；
  · Gemini 系列模型：82.8% ~ 87.4%；
  · DiariZen：63.1%；
  · Pyannote-3.1：62.7%。
  结论是 Omni 大模型 + 窗口化工程显著优于传统专用说话人分离工具。
【音色属性】提供 character voice description 字段描述角色嗓音特征。
【缺失维度】未见显式的语言标签、口音标签、情绪标签字段，也未标注说话人的性别/年龄等结构化属性。[不确定]

### 几何与结构化标注（相机参数、深度、3D point tracks、动作标注、显式状态）

【已有的结构化标注】镜头级五维属性中包含较强的摄影语法信息：scale（景别，如特写/中景/远景）、angle（机位角度）、movement（运镜方式，如推拉摇移）、narrative function（叙事功能）、duration category（时长类别）；以及 shot transition type（转场类型，如硬切/溶解）。训练时以 [SHOT i | scene sᵢ | camera κᵢ] 的形式把 camera 参数作为独立条件项注入。
【场景与身份的结构化】全局场景表 ⟨sceneₘ⟩ 与全局角色表 ⟨charₙ⟩ 构成一套显式的状态标注体系，使跨镜头的身份/场景延续可被显式建模与评测。
【缺失维度】未提供数值型相机外参/内参（camera pose、焦距等）、深度图、3D point tracks、光流场、人体姿态或动作骨架等几何标注。相机信息是语言化的定性描述（κᵢ）而非数值参数。
【评测侧的几何替代】CineBench 用 ArcFace（人脸聚类）与 DINOv2（视觉特征余弦相似度）做一致性判定，属于表征级而非几何级。

### 合成数据构造（受控扰动/编辑构造训练对，如InstructAV2AV）

无。CineDance-1M 全部为真实拍摄的影视素材，pipeline 中不含任何受控扰动、编辑构造、成对样本合成（如 InstructAV2AV 式的 before/after 编辑对）或数据增强式合成环节。
【最接近的操作】仅有的「构造」是把原子镜头重组为叙事序列，属于对真实素材的重新分组而非合成；以及 CineDance 模型训练中的 DARC 课程（对参考帧做连续加噪 ρ(rₖ; ηᵥ(u)) = ηᵥ(u)·rₖ + (1−ηᵥ(u))·εₖ、随机索引切换、参考帧按概率 p_drop(u) 丢弃），属于训练时的输入扰动策略而非离线合成数据构造。

### 人工介入程度（人工标注、人工质检、模型初筛+人工复核） ⚠️

人工介入定位为「阈值标定 + 质量抽检 + 基准构建」，不参与大规模标注生产（标注全部由大模型完成）：
【① 阈值标定】通过人工参考集测得叙事完整性的经验最小时长为 18.4 秒，据此把抗碎片化软阈值设为 20 秒——这是人工先验直接决定 pipeline 参数的典型案例。
【② 伪影人工审计】随机抽取 500 条片段，由三名独立标注员分别审查残留伪影（烧录字幕、logo、水印、电视网叠加层、片名卡、片尾字幕、录屏、转场特效、静帧停留等），分歧由联合复议（joint review）裁定，得出 2.8% 不合规率并与 Koala-36M 的 37.4% 对照。
【③ 人工标注基准】构建 100 条人工标注片段的小型基准，用于评估 ASR-to-Character 绑定准确率（95.4%）与各类 diarization 基线的对比。
【④ CineBench 人类评测】每条视频由 10 名独立评测员打分，采用 5 点 Likert 量表（1=不可用，5=优秀），双盲、随机呈现顺序，再用 Spearman 秩相关验证自动指标与人类判断的一致性。
【模式总结】属于「大模型全量标注 + 人工小样本校验与对齐」的现代范式，人工成本集中在验证侧而非生产侧。人工标注员的招募方式、人数总量、报酬等未披露。[不确定]

## 音视频对齐

### 音视频同步检测方法（口型同步、事件对齐）

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

### 同步检测具体指标与阈值（SyncNet/Synchformer/LSE/自研，阈值数值，如MOVA LSE-D≤9.5且LSE-C≥4.5、SkyReels-V4 SyncNet |offset|≤3∧conf>1.5） ⚠️

【指标选型】数据侧 SyncNet（唇同步）+ ImageBind（全局跨模态对齐）；评测侧 Sync-C、Sync-D、IB-Score。
【阈值】论文未公布任何同步指标的过滤阈值数值——这是该工作与 MOVA（LSE-D ≤ 9.5 且 LSE-C ≥ 4.5）、SkyReels-V4（SyncNet |offset| ≤ 3 且 conf > 1.5）等采用硬阈值的工作的根本方法论差异。CineDance 明确采取「全量打分 → 存为元数据 → 不硬剪枝」策略，原文表述为「we store all quality scores as metadata, enabling users to flexibly construct task-specific subsets」，将阈值决策权下放给下游使用者。
【取舍评价】优势是保留数据完整性与下游灵活性，避免过度过滤丢失长尾；劣势是数据集本身不保证同步质量下限，使用者必须自行设定阈值筛选，且论文未提供推荐阈值参考值。
【SyncNet 具体版本、ImageBind 版本、分数分布区间】均未披露。[不确定]

### 时序同步vs语义同步的分离处理（时间对齐与内容语义匹配作为两个独立过滤条件）

该工作在指标设计层面对时序同步与语义同步做了清晰的分离，属于其方法论上较为清醒的一点：
【时序同步（temporal）】由 SyncNet 承担，输出 Sync-C（置信度）与 Sync-D（偏移距离），刻画的是口型与语音在帧级时间轴上的对齐精度，只在有人物说话的镜头上有意义。
【语义同步（semantic）】由 ImageBind 承担（IB-Score / IB-A Score），刻画的是画面内容与音频内容在跨模态嵌入空间的语义匹配度，适用于音乐、环境音、音效等非语音场景，不要求帧级对齐。
【二者并列而非合并】在数据元数据中两项分别存储，在 CineBench 的 AV Sync 维度中两项分别报告，构成两个独立的判据。此外 Prompt Alignment 维度中的 IB-A Score 进一步把「音频与文本提示的语义一致性」也拆为独立项。
【未做的事】两项均不设过滤阈值，因此并未在数据集构建时用作「两个独立的过滤条件」——分离体现在度量与评测层面，而非过滤层面。

### 音频质量过滤（SNR、静音检测与静音占比阈值、无音轨剔除、画外音源剔除、背景音乐分离） ⚠️

【使用的指标】
  · DNSMOS —— 语音信号保真度的无参考 MOS 估计，衡量噪声、失真等退化程度；
  · CLAP embedding 的时序方差（temporal variance）—— 衡量音频内容随时间的变化丰富度，可间接识别单调、恒定或近似静音的音轨（方差极低往往意味着无信息量的音频）。
【策略】同样遵循全局的「不硬剪枝」原则，两项指标均存为元数据供下游筛选，论文未给出 DNSMOS 分数下限或方差阈值。
【入口约束】素材准入要求「自带原生音轨」，因此无音轨样本在采集阶段即被排除。
【非语音区间处理】在 ASR-to-Character 绑定的窗口化方案中，会先过滤掉非语音区间（filters non-speech intervals）再做绑定——这是对静音/非语音段的一种功能性处理，但目的是提升绑定准确率而非数据过滤。
【未涉及】未提及 SNR 计算、静音占比阈值、画外音（off-screen voice）源剔除、背景音乐分离（source separation，如 Demucs/BS-RoFormer）等常见音频清洗手段。作为影视素材，背景音乐与对白混叠是普遍现象，论文未说明如何处理。[不确定]

### 语音/音效/音乐的分类与分别处理策略

【分类方式】在镜头级音频 prompt 中把非语音音频显式拆为 music（音乐）、ambient sound（环境音）、effects（音效）三类分别描述；语音/对白则走独立的 ASR 通道并进一步绑定到角色 anchor token；角色嗓音另有 character voice description 字段。整体形成「语音 / 音乐 / 环境音 / 音效」四类的并行标注结构。
【分别处理策略】
  · 语音：句级 ASR 转写 → 窗口化说话人-角色绑定（95.4% 准确率）→ 角色音色描述；训练时以 (spkᵢ,ℓ, speechᵢ,ℓ) 二元组形式注入 prompt；评测时用 Whisper-large-v3 计算 WER/CER。
  · 音乐/环境音/音效：统一走自然语言音景描述（audio prompt），不做进一步的类别标签化或分轨；评测时用 AudioBox-Aesthetics 的 PQ（Production Quality）、CE（Content Enjoyment）、CU（Content Usefulness）三分量衡量。
【设计取向】语音走「精确转写 + 身份绑定」的结构化路线，非语音走「自然语言描述」的柔性路线，二者处理范式明确区分。
【未做】未做音频源分离（把对白、音乐、音效分轨），也未对三类音频设置差异化的质量阈值或采样配比。

## 训练配合

### 多阶段训练课程与数据课程调度（阶段划分依据：分辨率/时长/质量分/模态；低清→高清、图像→视频、短→长） ⚠️

CineDance 模型（LTX-2.3 适配）采用双重课程设计——数据课程与参考帧课程并行推进：
【数据驱动课程（Data-Driven Curriculum，两阶段）】
  · 阶段 1：2–3 个相邻镜头、10–12 秒，训练目标为局部镜头切换（local shot switching）能力；
  · 阶段 2：最多 8 个镜头、约 30 秒，训练目标为跨镜头一致性（cross-shot consistency）。
  课程划分依据是「镜头数 + 时长」这一叙事复杂度轴，而非传统的分辨率或质量分轴——这是与多数视频生成模型（低清→高清、图像→视频）不同的课程设计取向，反映本工作的核心矛盾在叙事长度而非画质。
【双轴参考课程 DARC（Dual-Axis Reference Curriculum）】随训练进度 u 渐进移除参考帧脚手架：
  · 视觉脚手架：对参考帧连续加噪 ρ(rₖ; ηᵥ(u)) = ηᵥ(u)·rₖ + (1−ηᵥ(u))·εₖ，噪声比例随训练递增；
  · 时序脚手架：以概率 q(ηₜ) 做随机索引切换（stochastic index switching）；
  · 参考帧丢弃：以概率 p_drop(u) 整体移除参考 token。
  三者共同实现从「强参考引导」到「自主生成」的平滑过渡。
【分辨率课程】素材统一为 1080p 起，论文未描述低清到高清的分辨率渐进策略。
【未披露】各阶段的训练步数、样本数、batch size、学习率、总计算量。[不确定]

### 数据配比随训练阶段的变化（预训练/退火annealing/SFT高质量子集） ⚠️

【已知的配比变化】数据配比的变化主要体现为「叙事复杂度」维度的递进而非模态或质量配比：阶段 1 使用 2–3 镜头 / 10–12 秒的短序列子集，阶段 2 切换到最多 8 镜头 / 约 30 秒的长序列子集。由于 CineDance-1M 平均含 24.2 镜头 / 92.8 秒，两个训练阶段实际上都是从完整序列中截取子片段使用，未用满全长。
【子集构造的支撑机制】数据集把所有质量分（DOVER 美学/技术、AMT 运动、DNSMOS、CLAP 方差、ImageBind、SyncNet）与结构化字段全量存为元数据，论文强调这使得使用者能「flexibly construct task-specific subsets」，即配比调整的基础设施是齐备的。
【未披露】各阶段的具体样本数与占比、是否存在退火（annealing）阶段、是否有高质量子集的 SFT 阶段、图文数据与视频数据的混合比例、音频-only 数据的补充。论文对模型训练的数据配比几乎无量化描述。[不确定]

### 后训练数据（SFT精选集规模与筛选标准、偏好对数量与标注方式、reward model训练数据） ⚠️

论文未描述任何后训练环节——无 SFT 精选集、无偏好对（preference pairs）标注、无 reward model 训练数据、无 RLHF/DPO 相关内容。CineDance 模型的训练描述止于两阶段数据课程 + DARC 参考课程，定位为「robust open baseline」（稳健的开源基线）而非追求 SOTA 的完整产品化模型，因此后训练不在本文范围内。
【唯一相关的人工偏好数据】CineBench 评测中收集的人类 Likert 打分（每视频 10 名评测员、5 点量表、双盲随机），但其用途是验证自动指标与人类判断的 Spearman 秩相关，属于评测校准数据，未用于训练 reward model。[不确定]

### 数据处理基础设施与吞吐（NeMo Curator/Data-Juicer/自研，GPU加速比，处理规模，成本） ⚠️

论文完全未披露数据处理基础设施与吞吐相关信息——无 GPU 型号与数量、无 GPU 小时数、无 pipeline 端到端处理耗时、无标注成本、无并行框架（如 Ray/Spark）说明，也未提及 NeMo Curator、Data-Juicer 等现成数据处理框架。
【唯一线索】GitHub README 的致谢中提到 vLLM，可推断大规模 LLM/MLLM 标注推理由 vLLM 承载以提升吞吐。
【规模推算参考（非论文数据）】处理量级为：32.8K 小时视频过 TransNetV2 切出 2589 万镜头，再用 Qwen3.5-27B 做 120 万条序列的解析、Qwen3.5-35B-A3B 做全量视觉标注、Qwen3-Omni-30B-A3B 做全量音频标注，平均每条视频产出 6,496.3 词标注 —— 百万级序列 × 6.5K 词的输出量意味着极大的推理开销，但论文对此毫无量化交代，是复现该 pipeline 的主要障碍之一。[不确定]

## 效果对比

### 数据策略消融的量化影响（区分：过滤严格度ablation / caption密度风格ablation / 数据配比ablation，及对应评测指标） ⚠️

论文的消融集中在「标注 pipeline 的方法选型」，而非传统意义上的「数据策略对生成效果的影响」，这是需要明确区分的：
【已做的 pipeline 消融】
  · Tab.4（叙事解析策略消融）：对比「直接让 LLM 输出时间戳」vs「自底向上镜头索引分组」，并横向对比不同规模的 Qwen 骨干。最优组合 Qwen3.5-27B + 自底向上策略取得 F1 = 88.4%，且仅 3.1% 序列短于 20 秒软阈值（碎片化率显著低于对照组）。
  · Tab.5（说话人分离消融）：100 条片段基准上，Qwen3-Omni-30B + 滑窗 prompt 达 83.1%，整片输入仅 56.4%。
  · Tab.7（ASR-角色绑定消融）：窗口化方案把 Qwen3-Omni 从 67.2% 提升到 95.4%；对照 Gemini 系列 82.8%–87.4%、DiariZen 63.1%、Pyannote-3.1 62.7%。
  · 伪影审计对照：CineDance-1M 不合规率 2.8% vs Koala-36M 37.4%（13.4× 改善），可视为「清洗严格度」的定性证据。
【未做的数据消融】论文未报告任何「去掉某级过滤 / 改变镜头分组阈值 / 调整 caption 密度或风格 / 改变数据配比」对下游生成质量（CineBench 各维度分数）的量化影响。也即：过滤严格度 ablation、caption 密度风格 ablation、数据配比 ablation 三类经典数据消融均缺失。
【判断】该工作的实证重心在「标注质量本身可被度量并优化」，而非「数据策略如何改变生成模型表现」，后者是明显的证据缺口。[不确定]

### 质量vs数量的证据（小而精数据超越大而杂的案例） ⚠️

论文未提供「小而精超越大而杂」的直接对照实验——没有用 CineDance-1M 子集 vs 更大杂质数据集训练同一模型的对比。
【间接证据】
  · 伪影审计对照：CineDance-1M 不合规率 2.8% vs Koala-36M 37.4%，13.4 倍改善。Koala-36M 规模远大于 CineDance-1M，论文以此论证「本数据集虽小但干净得多」，但未把这种清洁度差异转化为下游生成指标的对比。
  · 数据集横向对比表（Tab.6）：以「1080p + 原生音轨 + 镜头级双模态密集标注 + 92.8s/24.2 镜头」的组合优势论证质量维度的全面领先，属于属性对比而非效果对比。
【论文的整体立场】其实更偏向「质量与规模兼得」而非「以质换量」——刻意保留 100 万条序列的规模，同时通过「不硬剪枝、全量存元数据」的设计把质量-数量的权衡决策交给下游用户，本身就是对「一刀切追求高质量子集」路线的一种回避。
【结论】该维度缺乏直接实验支撑。[不确定]

### 训练数据domain分布与评测基准类目体系的对齐关系（如VABench七大类）

训练数据 taxonomy 与评测基准 CineBench 的类目体系存在部分呼应但非严格一一对齐：
【训练侧 taxonomy（8 维）】Genre、Format、Region、Modality、Story Logic、Era、Tone、Audience。
【CineBench 的分层轴（3 轴）】
  · Theme/Style（主题与风格）—— 与训练侧的 Genre/Tone/Era 等维度呼应；
  · Duration/Shot Count（时长与镜头数）—— 分三档：10s / 2–3 镜头、30s / 4–9 镜头、60s / 10–20 镜头，直接对应训练数据的核心差异化维度，也与模型两阶段课程（10–12s/2–3 镜头 → 30s/8 镜头）严格对齐；
  · Difficulty（难度）—— 每档内按三分位切为 Easy/Medium/Hard，难度公式 D = n_char + 1.5·n_scene + 0.4·log(1 + n_spk·L_ASR)，其中 n_char 角色数、n_scene 场景数、n_spk 说话人数、L_ASR 台词长度 —— 该公式的每一项都直接取自训练数据标注 schema 中的字段（全局角色表、全局场景表、ASR 绑定结果），是数据 schema 与评测体系耦合最紧密之处。
【六维评测指标与数据维度的对应】
  1) Video Quality（Aesthetic Quality、MUSIQ 成像质量、AMT 运动平滑度）—— 对应数据侧 DOVER/AMT；
  2) Audio Quality（AudioBox-Aesthetics 的 PQ/CE/CU、Whisper-large-v3 的 WER/CER）—— 对应数据侧 DNSMOS 与 ASR 标注；
  3) AV Sync（Sync-C/Sync-D、ImageBind IB-Score）—— 对应数据侧 SyncNet/ImageBind；
  4) Prompt Alignment（ViCLIP 镜头级、VideoScore-v1.1 视频级、IB-A Score 音频级）；
  5) Narrative Continuity（身份连续性用 ArcFace 聚类对照 ⟨charₖ⟩ token；场景连续性用 DINOv2 余弦相似度对照共享 ⟨sceneₖ⟩ 的镜头）—— 直接消费数据标注中的 anchor token；
  6) Shot Structure Response，SSR = S_cnt^0.35 · S_seg^0.65，其中 S_cnt = min(N,M)/max(N,M)（镜头数匹配度），S_seg 为双向时序 IoU（镜头切分位置匹配度）—— 检验模型是否遵循 prompt 指定的镜头结构。
【人类对齐验证】每视频 10 名独立评测员、5 点 Likert 量表、双盲随机呈现，用 Spearman 秩相关验证自动指标与人类判断一致性。
【总体判断】CineBench 是围绕 CineDance-1M 的标注 schema 反向设计的，二者耦合度极高——anchor token、镜头属性、ASR 绑定这些数据侧设计直接成为评测侧的判定依据，这是该工作在「数据-评测协同设计」上的突出特点，但也意味着 CineBench 对其他数据集训练的模型可能不完全公平。

## 不确定字段

以下字段的调研信息部分不确定（⚠️ 标注来源）：

- organization
- openness
- data_scale
- data_sources
- provenance_licensing
- duration_distribution
- resolution_aspect_distribution
- domain_distribution
- audio_category_distribution
- language_accent_distribution
- quality_filtering
- motion_filtering
- deduplication
- safety_filtering
- caption_model
- dialogue_transcription_attributes
- sync_metric_and_threshold
- audio_quality_filtering
- multi_stage_curriculum
- stage_data_mixture
- post_training_data
- data_infra_throughput
- data_ablation
- quality_over_quantity_evidence
- human_in_loop
