# 音视频联合生成基线合集（4 项工作合并调研）：
(1) JavisDiT / JavisDiT++ —— 《JavisDiT: Joint Audio-Video Diffusion Transformer with Hierarchical Spatio-Temporal Prior Synchronization》(arXiv:2503.23377) 与续作《JavisDiT++: Unified Modeling and Optimization for Joint Audio-Video Generation》(arXiv:2602.19163, ICLR 2026)，并附带评测基准 JavisBench / JavisBench-mini 与同步指标 JavisScore；
(2) MM-Diffusion —— 《MM-Diffusion: Learning Multi-Modal Diffusion Models for Joint Audio and Video Generation》(arXiv:2212.09478, CVPR 2023)，联合生成开山之作，附带自建 Landscape 数据集；
(3) AV-DiT —— 《AV-DiT: Efficient Audio-Visual Diffusion Transformer for Joint Audio and Video Generation》(arXiv:2406.07686)；
(4) Harmony —— 《Harmony: Harmonizing Audio and Video Generation through Cross-Task Synergy》(arXiv:2511.21579)，附带评测基准 Harmony-Bench；
(5) UniAVGen —— 《UniAVGen: Unified Audio and Video Generation with Asymmetric Cross-Modal Interactions》(arXiv:2511.03334)。
其中 (2)(3) 为早期小规模学术基线，(1) 为中期开源学术基线，(4)(5) 为 2025 年底的近期强基线。

> 主题: 视频生成模型（含音视频同时生成）的数据处理：数据清洗流程、数据分布、打标方式

[← 返回首页](../index.md)

**目录**: [基本信息](#基本信息) · [数据规模与分布](#数据规模与分布) · [清洗流程](#清洗流程) · [打标方式](#打标方式) · [音视频对齐](#音视频对齐) · [训练配合](#训练配合) · [效果对比](#效果对比)

## 基本信息

### 名称

音视频联合生成基线合集（4 项工作合并调研）：
(1) JavisDiT / JavisDiT++ —— 《JavisDiT: Joint Audio-Video Diffusion Transformer with Hierarchical Spatio-Temporal Prior Synchronization》(arXiv:2503.23377) 与续作《JavisDiT++: Unified Modeling and Optimization for Joint Audio-Video Generation》(arXiv:2602.19163, ICLR 2026)，并附带评测基准 JavisBench / JavisBench-mini 与同步指标 JavisScore；
(2) MM-Diffusion —— 《MM-Diffusion: Learning Multi-Modal Diffusion Models for Joint Audio and Video Generation》(arXiv:2212.09478, CVPR 2023)，联合生成开山之作，附带自建 Landscape 数据集；
(3) AV-DiT —— 《AV-DiT: Efficient Audio-Visual Diffusion Transformer for Joint Audio and Video Generation》(arXiv:2406.07686)；
(4) Harmony —— 《Harmony: Harmonizing Audio and Video Generation through Cross-Task Synergy》(arXiv:2511.21579)，附带评测基准 Harmony-Bench；
(5) UniAVGen —— 《UniAVGen: Unified Audio and Video Generation with Asymmetric Cross-Modal Interactions》(arXiv:2511.03334)。
其中 (2)(3) 为早期小规模学术基线，(1) 为中期开源学术基线，(4)(5) 为 2025 年底的近期强基线。

### 发布机构/公司

(1) JavisDiT / JavisDiT++：新加坡国立大学（NUS，Hao Fei、Shengqiong Wu、Tat-Seng Chua、Wei Li 等）主导，联合厦门大学（Jiayi Ji）、复旦大学（Fan Zhou）、罗切斯特大学（Jiebo Luo）、南洋理工大学（Ziwei Liu）等，第一作者 Kai Liu；社区组织名为 JavisVerse。
(2) MM-Diffusion：中国人民大学（Ludan Ruan、Qin Jin）与微软亚洲研究院 MSRA（Huan Yang、Bei Liu、Jianlong Fu、Nicholas Jing Yuan、Baining Guo）联合，另有北京大学参与；GitHub 组织为 researchmm（微软研究院多媒体组）。
(3) AV-DiT：多伦多大学（University of Toronto，Kai Wang、Dimitrios Hatzinakos）、德克萨斯大学达拉斯分校（UT Dallas，Shijian Deng、Yapeng Tian）与 Adobe Research（Jing Shi）联合。
(4) Harmony：上海交通大学（Teng Hu、Ran Yi）与腾讯混元 Tencent Hunyuan（Zhentao Yu、Guozhen Zhang、Zhengguang Zhou、Youliang Zhang、Yuan Zhou、Qinglin Lu）联合。
(5) UniAVGen：南京大学（新型软件技术国家重点实验室，Guozhen Zhang、Limin Wang）与腾讯混元 Tencent Hunyuan（Zixiang Zhou、Yi Chen、Yuan Zhou、Qinglin Lu）主导，联合上海交通大学（Teng Hu）、中国人民大学（Ziqiao Peng）、清华大学（Youliang Zhang）、上海人工智能实验室。
注：Harmony 与 UniAVGen 作者高度重叠（Guozhen Zhang、Teng Hu、Youliang Zhang、Yuan Zhou、Qinglin Lu 均在两篇中出现），可视为腾讯混元同一研究线上的姊妹工作。

### 发布时间（技术报告/论文/开源时间）

(2) MM-Diffusion：2022 年 12 月 19 日提交 arXiv（v1），2023 年 3 月修订，CVPR 2023 录用；是本合集中最早的工作。
(3) AV-DiT：2024 年 6 月 11 日 arXiv 公开（arXiv:2406.07686）。
(1) JavisDiT：2025 年 3 月 30 日 arXiv 首发（v1），2026 年 2 月 22 日修订，被 ICLR 2026 录用；续作 JavisDiT++ 为 2026 年 2 月公开（arXiv:2602.19163），同为 ICLR 2026。
(5) UniAVGen：2025 年 11 月 5 日 arXiv 提交（arXiv:2511.03334），2026 年 3 月 24 日修订。
(4) Harmony：2025 年 11 月 26 日 arXiv 提交（arXiv:2511.21579），2025 年 11 月 28 日修订。

### 类型（模型/数据集/工具链/评测基准）

全部为「模型」，其中三项额外附带评测基准或数据集产物：
- JavisDiT/JavisDiT++：模型 + 评测基准（JavisBench，10,140 条；JavisBench-mini，1,000 条）+ 同步性评测指标（JavisScore）+ 开源训练/推理工具链；是四者中唯一同时交付「模型 + benchmark + metric + 完整训练代码」的工作。
- MM-Diffusion：模型 + 自建数据集（Landscape，1,000 条 10 秒自然场景音视频片段）+ 开源代码与预训练权重。
- AV-DiT：纯模型（参数高效的联合生成架构），无新数据集、无新基准。
- Harmony：模型 + 评测基准（Harmony-Bench，150 条，分三档子集）。
- UniAVGen：模型（统一框架，支持联合生成、视频到音频配音、音频驱动视频动画等多任务），无新数据集。

### 开源程度（权重/代码/数据/pipeline各自是否开源） ⚠️

开源程度差异极大，可分三档：
【全开源档】
- JavisDiT / JavisDiT++：GitHub JavisVerse/JavisDiT，权重（HuggingFace JavisVerse/JavisDiT-v1.0-jav）、推理代码、完整三阶段训练脚本、评测工具（16 项音视频指标的 meta 文件与预计算 cache）全部开放；JavisBench 与 JavisBench-mini 基准数据公开。数据侧半开放：第一阶段音频预训练数据已预处理后发布于 HuggingFace（JavisData-Audio）；第二阶段视频数据来自 TAVGBench，仓库提供 33 万条 video ID 列表，但明确声明「因版权问题无法发布 YouTube 原始视频」；第三阶段 DPO 偏好数据「正在准备发布」。是本合集中数据透明度最高的一家。
- MM-Diffusion：MIT 许可，GitHub researchmm/MM-Diffusion 开放全部代码、训练脚本、评测流程，以及 6 个 checkpoint（Landscape.pt / Landscape_SR.pt / AIST++.pt / AIST++_SR.pt / guided-diffusion 上采样初始化模型 / i3d 与 AudioCLIP 评测模型）；Landscape 与 AIST++_crop 预处理数据通过 Google Drive 与百度网盘直接提供下载——是唯一把训练数据本体一并放出的工作。
【承诺开源档】
- AV-DiT：论文称「源代码与预训练模型将会发布」，截至调研时未见确认的公开仓库[不确定]。所用两个数据集（AIST++、Landscape）本身是公开的。
【闭源档】
- Harmony：论文未提供代码/权重仓库链接，Harmony-Bench 是否公开未见说明[不确定]；训练数据中自采集的 200 万条环境音音视频片段未开源。
- UniAVGen：论文未提供开源信息，训练用「内部采集的真人音视频数据集」完全闭源[不确定]。
注：Harmony 与 UniAVGen 均以开源模型为底座（Harmony 用 Wan2.2-5B、MMAudio 的音频 VAE、F5-TTS 的语音编码器、umT5 文本编码器；JavisDiT++ 用 Wan2.1-1.3B-T2V；JavisDiT v1 用 Open-Sora 的视频 VAE 与 AudioLDM2 的音频组件），属于「站在开源底座上但自身闭源/半开源」的模式。

### 是否支持音视频同时生成，以及实现方式（原生联合/级联/MoE融合）

五者全部支持音视频同时（联合）生成，且全部属于「原生联合」而非级联，但跨模态交互机制各不相同，构成一条清晰的技术演进线：
(1) MM-Diffusion（2022）：双分支序列式多模态 U-Net，视频用 2D+1D 时空卷积、音频用空洞卷积（dilated conv）；核心是 random-shift based attention（随机偏移注意力块）跨接两个子网，把跨模态注意力复杂度从 O((F×H×W)×T) 降到 O((S×H×W)×(S×T/F))。像素空间扩散，非 latent。另可零样本迁移到视频到音频、音频到视频的条件生成（靠梯度引导），无需额外训练。
(2) AV-DiT（2024）：主打「参数高效」——共享一个仅在图像数据上预训练的 DiT 主干（冻结），音频与视频两路仅插入轻量 adapter 可训练；视频分支在冻结 DiT block 中加入可训练时间注意力保证时序一致性，音频分支同样靠轻量参数适配，再加跨模态特征交互模块。属于「冻结共享骨干 + 双模态 adapter」的联合生成范式。
(3) JavisDiT（2025）：双塔 DiT（视频塔源自 Open-Sora，音频塔源自 AudioLDM2，二者冻结 VAE），核心创新是 HiST-Sypo Estimator（分层时空同步先验估计器）——从文本 prompt 中先估计出一组「粗粒度全局先验 + 细粒度时空先验」，再以该先验同时引导音频与视频两路去噪，实现细粒度时空对齐；跨模态交互靠 cross-attention 与 bidirectional attention 模块。
(4) JavisDiT++（2026）：换用 Wan2.1-1.3B-T2V 为底座，三项升级——模态特定混合专家 MS-MoE（Modality-Specific Mixture-of-Experts，在保证跨模态交互的同时提升单模态质量）、时间对齐旋转位置编码 TA-RoPE（Temporal-Aligned RoPE，实现音频 token 与视频 token 的显式帧级同步，思路与 Ovi 的 scaled-RoPE 同源）、以及音视频直接偏好优化 AV-DPO。
(5) UniAVGen（2025）：双分支联合合成架构，两个并行 DiT 分别处理音频与视频，核心是「非对称跨模态交互（Asymmetric Cross-Modal Interactions）」——两个模态间的信息流不对等，配合 Face-Aware Modulation（人脸感知调制）模块与 Modality-Aware Classifier-Free Guidance（模态感知无分类器引导）；一套框架统一支持联合生成、视频到音频配音（dubbing）、音频驱动视频动画等 5 类任务。
(6) Harmony（2025）：视频分支由 Wan2.2-5B 初始化，音频侧用 MMAudio 的 VAE 编码器 + F5-TTS 的语音编码器。三项创新：跨任务协同训练（Cross-Task Synergy，用「音频驱动视频」「视频驱动音频」两个双向生成任务来抑制联合去噪时的对齐漂移）、全局-局部解耦交互模块（Global-Local Decoupled Interaction, GLDI，全局分支管风格对齐、局部分支管时序精度）、同步增强的无分类器引导（Synchronization-Enhanced CFG，推理期放大对齐信号）。作者明确指出联合扩散范式的三大痛点：并发噪声演化下的对齐不稳定、注意力机制对时序精度的低效、标准 CFG 缺乏跨模态同步引导。

### 调研信息来源列表（论文/技术报告/官方文档/新闻的URL，标注每条来源的性质：官方一手/同团队旁证/第三方报道）

1) 官方一手｜JavisDiT 论文 https://arxiv.org/abs/2503.23377 （ICLR 2026，含 JavisBench/JavisScore 构建与三阶段训练数据规模）
2) 官方一手｜JavisDiT++ 论文 https://arxiv.org/abs/2602.19163 与 HTML 版 https://arxiv.org/html/2602.19163v1 （数据过滤策略、TAVGBench 1.1M→355K 漏斗、AV-DPO 偏好数据构建、附录 D.2 数据质量消融）
3) 官方一手｜JavisDiT GitHub https://github.com/JavisVerse/JavisDiT 与 https://github.com/JavisDiT/JavisDiT （开源范围、数据版权声明）
4) 官方一手｜JavisDiT 数据准备文档 https://raw.githubusercontent.com/JavisDiT/JavisDiT/main/assets/docs/data.md （三阶段数据 CSV schema、音频 30 秒截断、16kHz 重采样、视频 16fps 归一、最少 10 帧过滤、DPO 1+3 候选构造）
5) 官方一手｜JavisDiT++ 项目主页 https://javisverse.github.io/JavisDiT2-page/
6) 官方一手｜OpenReview 评审页 https://openreview.net/forum?id=hRRWfFpKRp
7) 官方一手｜MM-Diffusion 论文 https://arxiv.org/abs/2212.09478 与 CVPR 2023 版 https://openaccess.thecvf.com/content/CVPR2023/html/Ruan_MM-Diffusion_Learning_Multi-Modal_Diffusion_Models_for_Joint_Audio_and_Video_CVPR_2023_paper.html
8) 官方一手｜MM-Diffusion GitHub https://github.com/researchmm/MM-Diffusion （MIT 许可、数据下载、64×64 与 256×256 SR 双尺度）
9) 官方一手｜AV-DiT 论文 https://arxiv.org/abs/2406.07686 与 HTML 版 https://arxiv.org/html/2406.07686v1 （16 帧 256×256、1.6 秒 16kHz 音频、mel 40×16×8）
10) 官方一手｜AV-DiT OpenReview https://openreview.net/forum?id=FE6zflN5G5
11) 官方一手｜Harmony 论文 https://arxiv.org/abs/2511.21579 与 HTML 版 https://arxiv.org/html/2511.21579v1 （400 万片段构成、Gemini 自动标注、三阶段训练、Harmony-Bench）
12) 官方一手｜UniAVGen 论文 https://arxiv.org/abs/2511.03334 与 HTML 版 https://arxiv.org/html/2511.03334v1 （Emilia 英文子集、内部真人音视频数据、1.3M vs 30.1M 对比、三阶段训练超参）
13) 同团队旁证｜HuggingFace Papers 页 https://huggingface.co/papers/2602.19163 与 https://huggingface.co/papers/2406.07686
14) 第三方旁证｜MM-LDM 论文 https://arxiv.org/pdf/2410.01594 、MMDisCo https://arxiv.org/pdf/2405.17842 、UniForm https://arxiv.org/pdf/2502.03897 （均复述 Landscape/AIST++ 的统计口径，可交叉验证）

## 数据规模与分布

### 训练数据量级（视频条数/小时数/token数，预训练与SFT分开） ⚠️

各工作数据规模跨度达四个数量级，是本合集最能体现「学术基线 → 工业基线」演进的维度：
【MM-Diffusion（2022）——万级帧、小时级】
- Landscape：928 条 YouTube 源视频 → 切成 1,000 条不重叠的 10 秒片段，总时长约 2.7 小时、约 30 万帧。
- AIST++：1,020 条街舞视频片段，总时长 5.2 小时、约 56 万帧，配 60 首版权已清理（copyright-cleared）的舞曲。
- 两个数据集合计不到 8 小时，是典型的「小规模、单域、高保真」学术设定。
【AV-DiT（2024）——沿用同两个数据集】规模与 MM-Diffusion 完全一致（AIST++ 1,020 条/5.2 小时、Landscape 1,000 条/2.7 小时），未引入新数据；训练 10 万迭代、batch 16。
【JavisDiT（2025）——百万级条目】
- 第一阶段音频预训练：78 万条（0.78M）音频-文本对，训练 55 个 epoch（JavisDiT++ 中记为 50 个 epoch）。
- 第二阶段 ST-Prior 训练：61 万条（0.6M）同步的「文本-视频-音频」三元组，外加合成的异步负样本。
- 第三阶段 JAVG 训练：同样 61 万条三元组，微调 cross-attention 与双向注意力模块。
- 消融实验用 6 万条（60K）子集在 JavisBench-mini 上快速评估。
【JavisDiT++（2026）——约 100 万条公开数据】
- 音频：78 万条音频-文本对（沿用 JavisDiT 的集合），50 epoch，训练音频 FFN 共 794M 参数。
- 视频：TAVGBench 原始 110 万条三元组 → 过滤后 35.5 万条，其中 33 万条用于音视频 SFT（2 epoch，LoRA 121M 参数）、2.5 万条用于 AV-DPO（1 epoch，LoRA 121M 参数）。
- 作者强调全部为「public training entries」（公开数据），约 100 万条量级即达到 SOTA，是「小数据打大模型」的代表。
【Harmony（2025）——400 万+片段】
- 总计「超过 400 万条音视频片段」，覆盖人类语音与环境音两大类。
- 人类语音侧：从 Emilia、OpenHumanVid、SpeakerVid 三个来源汇总后，经音视频一致性打分模型筛出 200 万条高质量片段，每条 3–10 秒。
- 环境音侧：AudioCaps（约 128 小时，人工标注）+ Clotho（约 31 小时，人工标注）+ WavCaps（约 7,600 小时，自动标注）+ 自采集的 200 万条富含环境音的音视频片段。
- 阶段一音频预训练 10 万迭代、全局 batch 1536；阶段二 2 万迭代；阶段三跨任务联合训练 1 万迭代、batch 128。
【UniAVGen（2025）——130 万样本，主打「少数据高效」】
- 论文核心卖点即「以 1.3M 训练样本 vs 对比方法 30.1M（该对照数字取自 Ovi 的联合训练样本量）」实现在音视频同步、音色一致性、情绪一致性上的整体优势。
- 阶段一：Emilia 多语种音频数据集的英文子集，16 万步（160k steps），batch 256，lr 2e-5。
- 阶段二：内部采集的真人音视频数据，3 万步，batch 32，lr 5e-6。
- 阶段三：多任务学习 1 万步，五类任务配比 4:1:1:2:2。
- 1.3M 究竟指内部数据集条数还是跨阶段累计样本量，论文未澄清[不确定]。

### 数据来源构成（自有/公开数据集/网络爬取/授权采购/合成数据） ⚠️

呈现「自建小数据集 → 公开数据集拼装 → 公开+内部混合」的三段式演进：
【MM-Diffusion】(1) 自建 Landscape：作者自行从 YouTube 爬取 928 条自然场景视频后切分，是本合集中唯一从零爬取并公开发布的数据集；(2) 公开数据集 AIST++：AIST 数据集的子集，街舞视频配 60 首版权已清理的舞曲，天然规避版权风险。零样本迁移实验另涉及 AudioSet 类数据[不确定]。
【AV-DiT】纯公开数据集复用：AIST++ 与 Landscape，无自有数据、无爬取、无采购。
【JavisDiT / JavisDiT++】几乎全部为公开学术数据集拼装，这是其可复现性的关键：
- 音频侧（78 万条）来自 10 个公开集合：AudioSet、AudioCaps、VGGSound、WavCaps、Clotho、ESC50、GTZAN、MACS、UrbanSound8K、MusicInstrument——覆盖通用音效、音乐、语音三大类。
- 视频侧来自 TAVGBench（110 万条文本-音频-视频三元组的公开基准，底层为 YouTube 视频）。
- 偏好数据（DPO）为模型自生成 + 真值混合，属于合成/自举数据。
- JavisBench 评测集来自「已有数据集测试集（Landscape / AIST++ / FAVDBench）+ 2024 年 6 至 12 月上传的 YouTube 视频」混合采集。
【Harmony】公开 + 自采集混合：
- 公开：Emilia（TTS 专用语音语料）、OpenHumanVid（人物视频）、SpeakerVid（双人交互式人物生成数据）、AudioCaps、Clotho、WavCaps。
- 自采集：额外 200 万条富含环境音的音视频片段（论文称「newly collected」，来源渠道未披露）[不确定]。
【UniAVGen】公开 + 内部混合，内部占核心：
- 公开：Emilia 多语种音频数据集的英文子集（仅用于阶段一音频预训练）。
- 内部：「internally collected real human audio-video dataset」（内部采集的真人音视频数据集），承担阶段二、三的全部训练，来源、规模、采集方式全部未披露[不确定]。腾讯混元背景下推测与其内部视频/直播语料相关，但无任何文本依据。

### 数据合规与溯源（授权数据占比、rights-cleared数据集、C2PA等） ⚠️

整体披露稀薄，但 MM-Diffusion 与 JavisDiT 两家有明确动作，是本合集中少见的亮点：
【MM-Diffusion】(1) 选用 AIST++ 的一个显式理由是其 60 首舞曲为「copyright-cleared songs」（版权已清理歌曲），即在数据选型阶段就考虑了音乐版权；(2) 自建 Landscape 从 YouTube 爬取，未讨论视频版权，但数据集本身以研究用途公开发布；(3) 代码与权重以 MIT 许可发布。
【JavisDiT / JavisDiT++】版权意识最为明确：(1) 仓库明言「因版权问题无法发布 YouTube 原始视频」，只提供 33 万条 video ID 供用户自行下载——这是学术界规避视频版权的标准做法；(2) JavisBench 中来自 YouTube 的内容经过「严格的人工法律与伦理审核（strict manual legal and ethical verification）」；(3) 强调全部训练数据为公开数据集（public training entries），无采购、无内部私有数据，合规链条相对清晰。
【AV-DiT】未讨论数据授权与合规[不确定]；所用两个数据集均为已发布的学术数据集。
【Harmony】未讨论授权比例、rights-cleared 数据集、C2PA 等任何溯源机制；自采集的 200 万条片段的授权状况完全未披露[不确定]。论文仅笼统称环境音数据来自「public sources」。
【UniAVGen】完全未讨论数据合规、授权、隐私（注意其内部数据为「真人（real human）」音视频，涉及人脸与声纹，隐私敏感度高，但论文无任何相关表述）[不确定]。
共性缺口：五者均未提及 C2PA、内容水印、合成内容标识、数据主体删除请求等现代溯源机制。

### 片段时长分布与切分策略 ⚠️

各工作的时长策略高度体现其算力与目标差异，共同特征是「短、定长、不做多时长分桶」：
【MM-Diffusion】源视频切成 10 秒不重叠片段作为数据集单位，但实际训练样本更短：Landscape 与 AIST++ 均按固定帧数采样，音频对应片段长度约 1.6 秒量级[不确定：论文原文的具体训练片段秒数]。
【AV-DiT】最短：每个训练样本为 16 帧视频 + 截断或补齐到 1.6 秒的音频波形（16kHz）。视频与音频的时长严格对应，无分桶。
【JavisDiT / JavisDiT++】训练与评测统一为 4 秒片段（240P / 24fps），论文另做了延长至 10 秒的扩展测试。JavisDiT++ 的全部 JavisBench 评测同样固定在「240P、4 秒」。数据准备侧的硬约束：音频统一截断到 30 秒以内后再切分、视频至少 10 帧否则丢弃、fps 统一归一到 16Hz。
【Harmony】人类语音片段明确为 3–10 秒（这是有明确区间而非定长的少数案例）；阶段一音频预训练的最大片段长度为 10 秒，其中参考音频（reference audio）为随机截取的 1–3 秒片段。
【UniAVGen】视频统一按 16 fps 处理后经 VAE 编码；具体片段秒数未明确给出[不确定]。
共性局限：五者训练样本均在 10 秒以内，无一涉及分钟级长视频、多镜头拼接或时长分桶调度，这是学术基线相对工业模型（如 Veo/Sora 2/LTX-2）最本质的差距。

### 分辨率/宽高比分布与分桶策略 ⚠️

五者的分辨率均远低于工业模型，且全部为固定分辨率、无宽高比分桶：
【MM-Diffusion】基础扩散在 64×64 像素空间进行，再经独立的超分（SR）模型上采样到 256×256；仓库同时提供基础模型（Landscape.pt / AIST++.pt）与对应 SR 模型（Landscape_SR.pt / AIST++_SR.pt）。这种「低清生成 + 独立 SR」的两段式在 2022 年是主流做法。宽高比固定为 1:1，无分桶。
【AV-DiT】视频帧裁剪（crop）到 256×256 分辨率，视频 latent 尺寸 32×32×4，音频 latent（mel 频谱）40×16×8。单一分辨率、单一宽高比。
【JavisDiT / JavisDiT++】训练与评测统一 240P、24fps（数据准备阶段 fps 归一到 16Hz）。数据 CSV 中虽保留 height、width、aspect_ratio、resolution 等字段（沿用 Open-Sora 的数据管理 schema，理论上支持多分桶），但论文报告的实验全部为固定 240P/4 秒配置，未见分桶策略的实际使用[不确定]。
【Harmony】论文未指明视频分辨率与帧率[不确定]；底座 Wan2.2-5B 原生支持 720P，推测继承其分辨率能力但未在文中确认。
【UniAVGen】仅明确视频按 16 fps 处理后经 VAE 编码，分辨率与宽高比未披露[不确定]。
共性：无一采用面积归一化（如 Ovi 的固定像素总数 518400）、无一采用多宽高比分桶调度，说明这批工作把算力集中在跨模态对齐机制而非视觉保真度上。

### 类别/domain分布与配比策略（人物、动作、场景、风格等比例控制与概念均衡） ⚠️

domain 覆盖面从「单域」到「双域配比」再到「通用」，是本合集演进最明显的维度之一：
【MM-Diffusion —— 单域且类目明确】Landscape 覆盖 9 类自然场景，且类目完整披露：爆炸（explosion）、火焰噼啪（fire cracking）、下雨（raining）、水花飞溅（splashing water）、挤压水声（squishing water）、雷声（thunder）、水下咕噜声（underwater burbling）、瀑布轰鸣（waterfall burbling）、风声（wind noise）——全部是「视觉事件与声音强因果绑定」的自然现象，是刻意为验证音画对齐而选的域。AIST++ 则是单一街舞域（人体舞蹈动作 ↔ 音乐节拍）。两个数据集分别代表「环境音对齐」与「节奏/动作对齐」两种同步类型，这一「双数据集互补」的设定被后续 AV-DiT 等工作完整沿用为标准评测配置。
【AV-DiT】完全沿用上述两域，无扩展。
【JavisDiT / JavisDiT++ —— 显式构建了类目分类体系】其最大贡献之一就是把 domain 分布问题显式化：JavisBench 建立了五个评测维度、共 19 个场景类目的分类体系（taxonomy），五个维度为：事件场景（Event Scenario）、视频风格（Video Style）、声音类型（Sound Type）、空间构成（Spatial Composition）、时间构成（Temporal Composition）。作者强调「超过 50% 的视频属于高度复杂与有挑战性的场景」「75% 的样本包含多个发声事件（multiple sounding events）」——这是对早期基线「单一发声源、单一事件」设定的直接批判与超越。训练侧则通过 TAVGBench 的通用 YouTube 分布获得 domain 广度，但训练数据本身的类目占比未公开[不确定]。
【Harmony —— 显式的 1:1 双域配比】把数据明确二分为「人类语音」与「环境音」两大 domain，并在阶段一与阶段三训练中都严格采用 1:1 混合比例。Harmony-Bench 同样按此切分为环境音-视频、语音-视频、复杂场景（环境音+语音共现）三档，构成「训练配比 ↔ 评测类目」的直接对应。这是本合集中唯一给出明确 domain 配比数字的工作。
【UniAVGen —— 单域聚焦真人】训练数据为「真人音视频」，domain 高度集中于人物说话/表演场景，配合 Face-Aware Modulation 人脸感知模块，属于刻意的窄域设计；评测测试集 100 条按「50% 真实图像 / 50% AIGC 与动漫图像」二分，说明其在图像风格维度上做了显式配比（真实 vs 二次元各半），但内容 domain 层面未做细分[不确定]。
【共性缺口】除 Harmony 的 1:1 与 UniAVGen 的 50/50 外，均未披露人物/动作/场景/风格的细粒度比例控制或概念均衡（concept balancing）策略[不确定]。

### 音频类别分布与配比（语音/音效foley/音乐/环境音/静音的比例及控制策略）——AV模型独有维度 ⚠️

这一维度上五者分化极大，恰好构成「音效专注 → 语音专注 → 二者兼顾」的完整光谱：
【MM-Diffusion / AV-DiT —— 纯环境音与音乐，零语音】Landscape 全部是自然环境音（雨、雷、风、火、水），AIST++ 全部是音乐（60 首舞曲）。两个数据集都不含人类语音与对白，因此这两个早期基线完全不具备唇同步与 TTS 能力——这是它们与 2025 年后模型最根本的能力鸿沟。
【JavisDiT —— 音频预训练全类通吃，视频阶段刻意剔除语音】音频侧 78 万条来自 10 个数据集，JavisDiT++ 明确说明「不采用任何数据过滤策略，以确保最大化的文本到音频生成能力，覆盖通用音效（general sound）、音乐（music）与语音（speech）三类」——即音频预训练阶段刻意保持类别全覆盖、零过滤。但到了音视频 SFT 阶段策略反转：「使用 FunASR 检测工具剔除掉大部分包含人类语音的视频」。这是一个非常明确的类别配比决策——JavisDiT 系列有意放弃唇同步/对白生成能力，把模型能力集中在环境音与音效的事件级对齐上，从而避开高难度的唇形建模。JavisBench 的分类体系中「声音类型（Sound Type）」是五大评测维度之一，说明评测侧对音频类别是显式分层的。
【Harmony —— 严格 1:1 语音/环境音配比】最明确的配比策略：阶段一（音频预训练）与阶段三（跨任务联合训练）均采用「人类语音数据集与环境音数据集 1:1 混合」。语音侧 200 万条（Emilia + OpenHumanVid + SpeakerVid 经一致性筛选），环境音侧为 AudioCaps + Clotho + WavCaps + 自采 200 万条。评测端 Harmony-Bench 进一步把「语音+环境音共现的复杂场景」单独立为一档 50 条子集——直指真实视频中语音与环境音同时存在这一最难情形。可以说 Harmony 是本合集中对音频类别分布思考最系统的工作。
【UniAVGen —— 语音为绝对主体】阶段一在 Emilia（TTS 语料）英文子集上做纯音频预训练，阶段二三用真人音视频数据，全流程以人声/对白为核心；评测指标（SyncNet、Whisper WER、音色一致性、情绪一致性）也全部围绕语音，环境音与音乐能力未见报告[不确定]。
【静音处理】五者均未描述静音检测阈值或静音占比控制[不确定]。

### 叙事结构分布（单镜头vs多镜头、平均clip时长、镜头数分布、是否含原生音轨） ⚠️

全部为「单镜头、短片段、含原生音轨」的最简设定，无一涉及多镜头叙事：
【镜头数】五者训练样本均为单镜头片段。MM-Diffusion 把 928 条源视频机械切成 1,000 条 10 秒不重叠片段（未提及镜头检测，切分点可能跨镜头，是早期数据处理的粗糙之处）；JavisDiT 依赖 TAVGBench 已有的片段划分；Harmony 的 3–10 秒片段与 UniAVGen 的真人片段同样为单镜头。
【平均 clip 时长】MM-Diffusion 数据集 10 秒、实际训练片段更短；AV-DiT 16 帧 + 1.6 秒音频；JavisDiT/JavisDiT++ 固定 4 秒；Harmony 3–10 秒（区间分布，本合集中唯一非定长）；UniAVGen 未披露[不确定]。
【原生音轨】五者训练数据 100% 含原生同步音轨——这是音视频联合生成的前提条件。MM-Diffusion 的 AIST++ 是个特例：其音乐是与舞蹈配套的伴奏而非现场录音，属于「制作时即对齐」而非「拍摄时同步采集」。
【叙事能力】五者均无镜头转场、多镜头一致性、故事结构等设计；Harmony-Bench 的「复杂场景」子集是最接近叙事复杂度的尝试，但复杂性体现在「语音与环境音共现」的声学层面而非镜头叙事层面。
【与工业模型的差距】相比 Veo 3 / Sora 2 / LTX-2 等已开始处理多镜头与分钟级叙事的模型，本合集全部停留在「单镜头 ≤10 秒」阶段，这既是学术算力限制的结果，也意味着这些基线的数据处理经验难以直接迁移到长视频场景。

### 语言/口音分布（多语种唇同步能力的数据基础） ⚠️

披露极少，且整体呈现「英语单一化」倾向：
【MM-Diffusion / AV-DiT】训练数据不含人类语音（自然环境音与器乐音乐），因此语言/口音维度不适用（N/A）。
【JavisDiT / JavisDiT++】音频预训练数据虽包含语音类数据集（AudioSet、WavCaps 中含语音成分），但语种分布未披露[不确定]；音视频 SFT 阶段用 FunASR 主动剔除含语音视频，因此模型基本不具备语言相关的唇同步能力，该维度对其实际不构成瓶颈。
【UniAVGen】明确只用 Emilia「多语种音频数据集的英文子集（English subset）」做音频预训练——即刻意放弃多语种，聚焦英语；评测用 GRID 基准（英语语料）与 Whisper WER，同样只验证英语。内部真人音视频数据的语种构成未披露[不确定]。
【Harmony】使用 Emilia（本身为多语种 TTS 语料）但未说明是否限定英文子集[不确定]；Harmony-Bench 的语音子集论文提到用于考察「唇同步保真度与多语种语音鲁棒性（multilingual speech robustness）」，是本合集中唯一明确宣称覆盖多语种的工作，但具体语种列表、各语种占比、各语种唇同步指标均未给出[不确定]。
【共性缺口】五者均未标注口音（accent）属性、未做语种平衡、未报告分语种的同步指标——与工业模型（如 Vidu、Kling 等强调多语种唇同步）形成鲜明对比。

## 清洗流程

### 清洗漏斗整体结构（几级过滤、各级顺序） ⚠️

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

### 漏斗定量保留率（每级过滤的输入/输出量与最终保留率，如Apollo 27%） ⚠️

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

### 镜头切分方法（PySceneDetect/自研模型/shot-aware splitting） ⚠️

本合集在镜头切分上普遍薄弱，是与工业级 pipeline 差距最明显的环节之一：
【MM-Diffusion】明确采用「机械等长切分」而非镜头感知切分：把 928 条源视频「分成 1,000 条不重叠的 10 秒片段」，未使用 PySceneDetect、TransNetV2 或任何场景检测工具。这意味着切出的片段可能跨越镜头边界，含转场画面——是早期数据处理的已知粗糙点。作者对 AIST++ 另做了 crop 处理（仓库中的 AIST++_crop），属于空间裁剪而非时序切分。
【AV-DiT】不做切分，直接从已切好的数据集片段中采样 16 帧。
【JavisDiT / JavisDiT++】依赖上游 TAVGBench 已完成的片段划分，自身 pipeline 中无镜头检测环节；data.md 中的视频处理步骤只有「剔除损坏视频、过滤少于 10 帧的样本、fps 归一到 16Hz」，无 scene detection[不确定：TAVGBench 上游是否做过镜头检测]。
【Harmony】3–10 秒的片段划分方式未说明是否基于镜头检测[不确定]；自采集的 200 万条环境音片段的切分方法同样未披露。
【UniAVGen】未描述任何切分方法[不确定]。
【影响】缺乏镜头感知切分意味着训练数据中可能混入含转场的片段，理论上会损害时序一致性；但由于所有工作的片段都极短（1.6–10 秒），跨镜头风险相对可控。这也解释了为何这批模型全部不具备多镜头生成能力。

### 质量过滤（美学评分、清晰度、OCR文字过滤、黑边/水印/logo检测） ⚠️

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

### 运动过滤（光流/运动分数阈值、静态与抖动剔除） ⚠️

【JavisDiT++（唯一明确使用）】采用「光流评分（flow scoring）」作为 TAVGBench 数据过滤的三项质量指标之一，遵循 Open-Sora 的实现（Open-Sora 通常用 UniMatch 或 RAFT 计算光流幅值作为 motion score）。用途是剔除运动过少的静态片段。具体阈值、是否也剔除运动过强/抖动的一端、motion score 是否作为训练条件或采样权重，论文均未说明[不确定]。
【MM-Diffusion】无运动过滤。但值得注意的是其数据集选型本身隐含了运动约束：Landscape 的 9 类场景（爆炸、火焰、下雨、水花、雷、瀑布、风等）全部是有持续动态的自然现象，AIST++ 全是舞蹈动作——即通过 domain 选择而非过滤器保证了运动充分性，这是小数据集时代的「以选型代替过滤」思路。
【AV-DiT】无运动过滤。
【Harmony / UniAVGen】均未提及任何运动过滤或光流打分[不确定]。
【共性】除 JavisDiT++ 外无一使用光流工具，这与本合集数据规模小、可人工把控质量有关；一旦数据规模上到百万级（JavisDiT++ 的 110 万条 TAVGBench），运动过滤就成为必需项。

### 去重方法（精确去重与基于embedding的语义去重分别记录） ⚠️

本合集五项工作全部未描述任何去重环节[不确定]，无论是精确去重（哈希/pHash/帧指纹）还是基于 embedding 的语义去重（CLIP/视频 embedding 聚类），这是一个统一的空白维度。
各自的间接情况：
- MM-Diffusion：Landscape 明确切成「不重叠（non-overlapped）」的 10 秒片段，这是片段级的重叠避免而非跨视频去重；928 条源视频规模极小，重复风险天然较低。
- AV-DiT：使用现成公开数据集，去重责任在上游数据集作者。
- JavisDiT / JavisDiT++：唯一相关的设计是「DPO 用的 3 万条 prompt 池与 SFT 训练数据严格不重叠（apart from the SFT training data）」，以及 33 万 SFT 与 2.5 万 DPO 样本互不重叠——这是训练/评测集划分层面的隔离，不是数据去重。TAVGBench 上游是否做过去重未知[不确定]。JavisBench 构建中的「后置过滤以保证多样性（ensure diversity）」在功能上接近语义去重，但作者未使用去重的表述、也未说明用什么方法度量多样性[不确定]。
- Harmony：400 万条来自多个公开数据集拼装（AudioCaps、Clotho、WavCaps 之间存在已知的音频重叠，例如 WavCaps 与 AudioSet 有交集），但论文未提及跨数据集去重[不确定]，这是一个潜在的重复风险点。
- UniAVGen：无任何描述[不确定]。

### VLM/LLM作为质检员（多模态大模型质量打分与错配剔除，2026年从浅层打分器转向大模型语义判断的趋势） ⚠️

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

### 安全与合规过滤（NSFW、版权、人脸/隐私） ⚠️

整体披露极少，仅 JavisDiT 一家有明确动作：
【JavisDiT / JavisDiT++（唯一）】JavisBench 构建过程中，对全部来自 YouTube 的内容进行了「严格的人工法律与伦理审核（strict manual legal and ethical verification）」——这是本合集中唯一明确的安全合规环节，且是人工而非自动化的。但该审核针对的是评测集（10,140 条），训练侧的 33 万条 TAVGBench 数据是否做过安全过滤未说明[不确定]；也未描述 NSFW 检测器、暴力内容过滤、人脸隐私保护等自动化手段。
【MM-Diffusion】未描述 NSFW/版权/隐私过滤[不确定]。间接的合规意识体现在数据选型上：选用 AIST++ 的理由之一是其配乐为版权已清理歌曲。
【AV-DiT】未描述任何安全过滤[不确定]。
【Harmony】未描述任何安全过滤[不确定]。其数据含大量真人视频（OpenHumanVid、SpeakerVid、自采集片段），人脸与声纹隐私风险实际存在但论文无相关表述。
【UniAVGen】未描述任何安全过滤[不确定]。风险最高——训练数据为「内部采集的真人（real human）音视频数据集」，且模型具备音频驱动人脸动画（audio-driven animation）能力，属于深度伪造敏感能力，但论文既无数据侧隐私说明，也无模型侧滥用防护或使用限制声明。
【共性缺口】五者均未提供 Model Card 级别的 safety 章节、未讨论深度伪造滥用防范、未提及名人肖像剔除或儿童内容处理。

## 打标方式

### 使用的caption模型（自研VLM/开源模型，模型规模） ⚠️

从「无 caption」到「MLLM 全自动标注」的完整演进：
【无 caption 阶段（MM-Diffusion / AV-DiT）】两者均为无条件生成（unconditional generation）——直接从高斯噪声生成音视频对，不接受文本条件，因此训练数据完全不需要 caption。AV-DiT 论文在局限性中明确承认「主要聚焦于无条件的音频与视频生成」。这是它们与后续所有 T2AV 模型最根本的范式差异。MM-Diffusion 的零样本条件生成（V2A/A2V）靠梯度引导实现，同样不涉及文本 caption。
【复用上游 caption（JavisDiT / JavisDiT++）】自身不做视频打标，直接使用 TAVGBench 数据集自带的文本 caption（TAVGBench 的 caption 由其原作者用自动化方法生成）；音频侧 78 万条同样使用 AudioCaps、Clotho、WavCaps 等数据集自带的音频文本描述（其中 AudioCaps 与 Clotho 为人工标注，WavCaps 为自动标注）。data.md 的 CSV schema 中 text 与 audio_text 两列即对应视频与音频的文本描述。JavisDiT 团队唯一自建 caption 的地方是评测集 JavisBench：使用「先进的 Qwen 系列模型」生成 caption 并做 19 类场景归类，模型的具体版本与规模未指明[不确定]。
【MLLM 全自动标注（Harmony，本合集中最先进）】使用 Google Gemini 对全部 400 万条音视频片段做自动标注，一次性产出三类内容：ASR 转写文本、描述性视频 caption、背景音 caption。Gemini 的具体版本（1.5 Pro / 2.0 / 2.5 等）、prompt 模板、输出质量校验协议均未披露[不确定]。选用 Gemini 而非开源 VLM，说明其看重原生的长视频 + 音频联合理解能力——这与 Ovi 使用「音频可感知的 MLLM」是同一思路。
【UniAVGen】未提及任何 caption 模型或文本标注流程[不确定]；其任务形态（音频驱动动画、视频到音频配音、真人视频生成）对文本 caption 的依赖度本身较低。

### caption密度与结构化程度（短/长/密集描述、结构化字段如镜头运动、风格标签） ⚠️

【MM-Diffusion / AV-DiT】无 caption（无条件生成），该维度不适用。
【JavisDiT / JavisDiT++】caption 结构继承自上游数据集，非自建：
- 视频侧：TAVGBench 提供的单条自然语言描述（同时描述视听内容），密度属于中等长度的整体描述，无结构化字段。
- 音频侧：各音频数据集自带的音频 caption，风格差异大——AudioCaps/Clotho 为人工写的一句话音频描述，WavCaps 为 ChatGPT 辅助生成的描述，ESC50/UrbanSound8K/GTZAN 等本质是类别标签而非描述句。这种「多来源 caption 风格混杂」是其音频预训练数据的固有特点，论文未做风格统一化处理[不确定]。
- CSV schema 层面确有结构化管理：path、id、relpath、num_frames、height、width、aspect_ratio、fps、resolution、audio_path、audio_fps、text、audio_text ——但这些是元数据字段而非 caption 内部的结构化标签。
- 无镜头运动、景别、光照、风格等显式结构化标注[不确定]。
【Harmony（本合集中结构化程度最高）】Gemini 一次标注产出三类分离的文本字段：(1) ASR 转写（transcript）、(2) 描述性视频 caption（video caption）、(3) 背景音 caption（audio caption / background sound caption）。这是明确的多字段分流（factorized）结构，与 Ovi 的「单条 caption 内嵌标签」路线相反、与 Script-a-Video 的因子化流更接近。这套三字段结构直接决定了 Harmony-Bench 的条件设置：环境音子集用音频/视频 caption 作条件，语音子集以转写文本为主要条件，复杂场景子集则使用「全套多模态 prompt」——即三个字段可按任务灵活组合，是 factorized 设计带来的直接好处。prompt 模板细节与各字段的平均长度未披露[不确定]。
【UniAVGen】未描述[不确定]。

### 音视频联合caption结构（是否同时覆盖视觉+听觉轨道、是否分流为独立字段，如LTX-2全音景描述、Script-a-Video factorized streams、Foley-Omni三字段） ⚠️

本合集在音视频联合 caption 结构上的做法分化明显，恰好覆盖了三种典型路线：
【路线零：无联合 caption（MM-Diffusion / AV-DiT）】无条件生成，不存在联合 caption schema。音画对齐完全靠架构机制（random-shift attention / 跨模态 adapter）与数据本身的原生同步性，不借助任何文本中介。
【路线一：单条混合 caption（JavisDiT / JavisDiT++）】使用 TAVGBench 的单条文本，该文本同时描述视觉与听觉内容，但不分流为独立字段，也不含时间戳或语音标签。JavisDiT 的关键洞察恰恰在于「单条文本条件不足以保证细粒度时空同步」——因此它不在 caption 结构上做文章，而是在文本之外额外引入 HiST-Sypo（分层时空同步先验估计器），从 prompt 中先估计出粗粒度全局先验与细粒度时空先验，再用该先验同时引导两路去噪。可以理解为：JavisDiT 把「联合 caption 应该编码的时空对齐信息」从文本层移到了隐式先验层。这是与 Ovi（靠 <S>台词<E> 内联标签编码时序）、Foley-Omni（三字段拆分）截然不同的第三条路。
【路线二：三字段完全分流（Harmony，本合集最完整的联合 schema）】Gemini 标注产出三条独立的文本轨道：
(1) 视觉轨道 —— 描述性视频 caption；
(2) 语音轨道 —— ASR 转写文本（transcript）；
(3) 非语音听觉轨道 —— 背景音/环境音 caption。
三者作为独立条件字段而非拼接成一句，因此可按任务组合使用。Harmony-Bench 的三档子集直接验证了这一设计的价值：环境音子集只用视频+音频 caption、语音子集主要用转写、复杂场景子集用全套多模态 prompt——说明该 schema 支持「按需激活条件通道」。这与 Ovi 消融得出的「合并单一 T5 嵌入优于 CLAP/T5 分离编码」结论看似矛盾，实则针对的问题不同：Ovi 的分离是编码器层面的分离（不同编码器导致表征空间割裂），Harmony 的分流是字段层面的分流（可能仍共用同一文本编码器 umT5）[不确定：Harmony 三字段是否共用 umT5 编码后再拼接]。
【UniAVGen】未描述联合 caption 结构[不确定]；其条件形态以参考图像 + 参考音频 + 转写为主，文本 caption 不是核心条件通道。

### 对白转写与说话人属性标注（ASR转写+说话人身份/语言/口音/情绪） ⚠️

【MM-Diffusion / AV-DiT】训练数据完全不含人类语音（自然环境音与器乐音乐），无转写、无说话人属性标注，该维度不适用。
【JavisDiT / JavisDiT++】反向操作——不但不做转写，反而主动用 FunASR 检测并剔除含人类语音的视频，把语音从音视频训练数据中清除出去。这是一个刻意的能力取舍：放弃唇同步与对白生成，换取在环境音/音效事件对齐上的专注。FunASR 在此扮演的是「语音检测器」而非「转写器」的角色。音频预训练阶段虽包含语音类数据，但未做转写标注。
【Harmony（有完整转写）】Gemini 自动标注明确产出 ASR 转写文本（transcripts），并作为语音条件通道送入模型；语音数据来自 Emilia（TTS 语料，本身自带转写）、OpenHumanVid、SpeakerVid（双人交互数据，天然含多说话人场景）。说话人属性方面，Harmony 有一个独立于转写的机制——阶段二「音色解耦微调（Timbre Disentanglement Finetuning）」，用同一说话人的跨话语错配参考-目标配对（cross-utterance mismatched reference/target；环境音则用同一片段的非重叠段），训练模型从 1–3 秒参考音频中提取音色特征而不泄漏内容。这实际上是用「数据配对方式」而非「属性标注」来解耦音色，比显式标注年龄/性别/口音的做法更轻量。但年龄、性别、口音、情绪等显式属性标签未见标注[不确定]，说话人日志（diarization）与说话人 ID 也未提及[不确定]。
【UniAVGen】评测端使用 Whisper 计算 WER，说明转写在评测环节存在；训练数据侧是否做 ASR 转写标注未说明[不确定]。模型显式优化「音色一致性（timbre consistency）」与「情绪一致性（emotion consistency）」两项指标，暗示训练中存在音色与情绪的条件通道（很可能通过参考音频而非文本属性标签实现），但具体标注方式未披露[不确定]。

### 几何与结构化标注（相机参数、深度、3D point tracks、动作标注、显式状态） ⚠️

本合集几乎全线空白，仅 UniAVGen 有一项人脸相关的结构化信息：
【UniAVGen（唯一）】包含 Face-Aware Modulation（人脸感知调制）模块，该模块需要人脸区域信息才能对特定区域施加调制，因此数据侧应存在人脸检测/人脸区域标注环节，但论文未描述使用何种人脸检测器、是 bounding box 还是 mask、是否离线预计算[不确定]。评测使用 EMTD 基准（音频驱动人脸动画基准）也佐证其人脸相关处理。
【MM-Diffusion】无任何几何标注。值得一提的是 AIST++ 数据集本身自带 3D 人体姿态与相机参数标注（AIST++ 的原始设计目标就是 3D 舞蹈动作生成），但 MM-Diffusion 完全未使用这些标注，只用了 RGB 视频与音频——即「数据集有几何标注但工作没用」。
【AV-DiT】无任何几何标注。
【JavisDiT / JavisDiT++】无相机参数、无深度、无 3D point tracks、无姿态关键点、无动作标签、无分割 mask。唯一接近结构化的是光流评分（flow score，标量运动强度）与 CSV 中的分辨率/帧数等元数据。JavisDiT 的 HiST-Sypo 先验虽名为「时空先验」，但它是从文本估计出的隐式向量而非显式几何标注。
【Harmony】无几何标注[不确定]。
【共性判断】本合集的技术路线普遍是「靠跨模态注意力机制自发学习对齐」，而非「靠显式几何先验约束对齐」——这与 Ovi 强调「不需要人脸 bbox、不需要 face mask」的主张一致，说明音视频联合生成社区整体倾向于数据驱动而非几何先验驱动。UniAVGen 的 Face-Aware Modulation 是这一趋势中的少数反例，代价是把模型能力限定在真人说话场景。

### 合成数据构造（受控扰动/编辑构造训练对，如InstructAV2AV） ⚠️

本合集中有两项工作使用了合成/自举数据构造，且都是本合集数据侧最具借鉴价值的部分：
【JavisDiT —— 异步负样本的受控构造（ST-Prior 训练）】为训练 HiST-Sypo 时空同步先验估计器，需要区分「同步」与「不同步」的音视频对。作者在 61 万条同步三元组之外，额外「合成异步负样本（synthesized asynchronous negative samples）」，配合对比学习（contrastive learning）训练先验估计器。这是典型的「受控扰动构造训练对」——通过对音轨做时移、替换或跨样本错配来人工制造负例。具体的负样本构造方式（时移多少帧、是否跨样本随机配对、正负样本比例）在论文附录 C.2.4「Negative Sample Construction」中，但公开 HTML 版本此节内容被截断，未能获取细节[不确定]。这一设计与 Ovi「靠严格阈值筛掉不同步数据」形成鲜明对照：Ovi 是把不同步数据丢弃，JavisDiT 是把不同步数据主动造出来当负例用。
【JavisDiT++ —— 模型自生成的偏好数据（AV-DPO）】用参考模型对 3 万条 prompt 池中的每条 prompt 生成 N=3 个音视频候选，与 1 条真值组成「1 真值 + 3 生成」的候选组，再用六个奖励模型打分排序，构造出约 2.5 万条偏好对。这是完全的模型自举合成数据。一个值得记录的量化发现：最终偏好数据中约 30% 的 winning 样本来自模型生成而非真值——即模型在近三成的情形下能生成优于真实数据的样本，作者据此判断基线模型已具备相当强的生成能力。这个比例对判断「何时该转向 DPO/RLHF」是个有参考价值的信号。
【Harmony —— 数据配对层面的构造】阶段二音色解耦微调采用「跨话语错配的参考-目标配对」：语音数据取同一说话人的不同话语作为参考与目标，环境音数据取同一片段的非重叠段作为参考与目标。这不是生成新数据，而是通过重新配对已有数据来构造训练信号，属于轻量级的合成配对策略。
【MM-Diffusion / AV-DiT / UniAVGen】未使用任何合成数据构造[不确定]。

### 人工介入程度（人工标注、人工质检、模型初筛+人工复核） ⚠️

人工介入普遍集中在评测端而非数据端，是本合集的共性特征：
【MM-Diffusion（评测端人工投入最大）】开展了大规模图灵测试（Turing test），累计收集 1 万次人工投票，结果显示超过 80% 的生成音视频片段能骗过人类评估者。数据端未描述人工标注或质检环节[不确定]，但 Landscape 仅 1,000 条的规模与「high-fidelity」的自我描述暗示可能存在人工筛选。
【AV-DiT】未描述人工介入[不确定]。
【JavisDiT / JavisDiT++（数据端与评测端均有明确人工投入，本合集最充分）】
- 数据端：JavisBench 构建时对全部 YouTube 来源内容做「严格的人工法律与伦理审核」——这是本合集中唯一明确的数据端人工审核环节。
- 指标验证端：为验证 JavisScore 指标的有效性，构建了一个含 3,000 条样本的人工标注评估数据集，用人类对同步性的判断来校准指标。这种「先造指标、再用人工标注验证指标本身」的做法在评测方法学上较为规范，但标注者数量、招募方式、标注一致性（inter-annotator agreement）均未披露[不确定]。
- 模型评测端：JavisDiT++ 用 JavisBench 中的 100 条 prompt，由 3 名志愿者做盲测的 win-tie-lose 偏好判断。
【Harmony】做了系统性的人工消融研究（human ablation studies）验证各组件贡献；数据端的人工介入方面，其数据源中 AudioCaps（约 128 小时）与 Clotho（约 31 小时）是人工标注的音频 caption 数据集，而 WavCaps（约 7,600 小时）为自动标注——论文对三者的人工/自动标注属性做了明确区分，说明团队对标注质量分层是有意识的。自有 400 万条片段的标注则完全依赖 Gemini，无人工复核描述[不确定]。
【UniAVGen】未描述任何人工介入环节[不确定]。
【共性观察】五者的数据 pipeline 基本都是全自动的，人工主要用于评测与指标校准，而非逐条数据审核——这与工业级模型（往往有专职数据标注团队）形成对比，也是学术工作的现实约束。

## 音视频对齐

### 音视频同步检测方法（口型同步、事件对齐） ⚠️

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

### 同步检测具体指标与阈值（SyncNet/Synchformer/LSE/自研，阈值数值，如MOVA LSE-D≤9.5且LSE-C≥4.5、SkyReels-V4 SyncNet |offset|≤3∧conf>1.5） ⚠️

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

### 时序同步vs语义同步的分离处理（时间对齐与内容语义匹配作为两个独立过滤条件） ⚠️

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

### 音频质量过滤（SNR、静音检测与静音占比阈值、无音轨剔除、画外音源剔除、背景音乐分离） ⚠️

音频侧过滤普遍缺失，只有格式化预处理：
【JavisDiT / JavisDiT++】明确声明音频预训练阶段「不采用任何数据过滤策略（no data filtering strategy）」，以确保最大化的文本到音频生成能力覆盖通用音效、音乐与语音三类——这是一个刻意的「不过滤」决策，与多数模型的严格音频筛选相反。音频侧只有三步格式化处理：截断到 30 秒以内、统一重采样到 16kHz、提取音频统计信息。无 SNR 阈值、无静音检测、无无音轨剔除、无背景音乐分离、无削波/失真检测[不确定]。反倒是音视频阶段有一项特殊的「音频内容过滤」——用 FunASR 剔除含人类语音的视频，但这是类别过滤而非质量过滤。
【Harmony】语音数据经「音视频一致性打分模型」筛选，这是跨模态一致性过滤而非纯音频质量过滤；音频质量层面（SNR、静音、噪声）未描述任何过滤[不确定]。音频编码走 MMAudio 的 VAE 编码器 + F5-TTS 的语音编码器双路。参考音频取 1–3 秒随机片段，未说明是否做有效性检查[不确定]。
【UniAVGen】只有格式化处理：音频先按 24,000 Hz 采样、转换为 mel 频谱，生成后用 Vocos 声码器还原波形。注意其 24kHz 采样率高于 JavisDiT/AV-DiT/MM-Diffusion 的 16kHz，带宽更宽、语音保真度更好——这与其聚焦真人语音的定位一致。无音频质量过滤描述[不确定]。
【MM-Diffusion / AV-DiT】无音频质量过滤。AV-DiT 的处理为「截断或补齐到 1.6 秒波形、16kHz 采样、转 mel 频谱 40×16×8」，属纯格式化。
【共性判断】本合集的音频质量把控主要依赖上游数据集（AudioCaps、Clotho、Emilia 等本身是精选语料），自身不建音频质检环节——这在小规模精选数据上可行，但在自采集的百万级数据（如 Harmony 的 200 万条自采环境音片段）上是明显的风险敞口。

### 语音/音效/音乐的分类与分别处理策略 ⚠️

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

## 训练配合

### 多阶段训练课程与数据课程调度（阶段划分依据：分辨率/时长/质量分/模态；低清→高清、图像→视频、短→长） ⚠️

本合集在训练课程设计上呈现明显的收敛趋势——2025 年后的三项工作不约而同采用「音频单模态预训练 → 音视频联合训练 → 强化/多任务」的三阶段范式：
【MM-Diffusion（无课程，但有分辨率两段式）】单阶段联合训练，无多阶段课程。唯一的分段是空间尺度上的：基础模型在 64×64 训练，另训一个独立的超分模型上采样到 256×256（该 SR 模型从 guided-diffusion 的上采样模型初始化）。这是「低清 → 高清」课程的原始形态，但两个模型独立而非渐进式课程。
【AV-DiT（无课程）】单阶段训练 10 万迭代，batch 16，lr 5e-4，AdamW 优化器，在 NVIDIA RTX A6000 上完成；仅新插入的 adapter 层可训练，DiT 主干冻结。参数高效路线本身替代了课程的作用。
【JavisDiT（三阶段，划分依据是「模态 → 对齐能力 → 联合生成」）】
- 阶段一：音频预训练——78 万条音频-文本对，55 个 epoch，把音频分支的 T2A 能力打牢（视频塔用 Open-Sora、音频塔用 AudioLDM2，VAE 全程冻结）。
- 阶段二：ST-Prior 训练——61 万条同步三元组 + 合成异步负样本，用对比学习训练 HiST-Sypo 时空同步先验估计器。这是一个专门用于「学会判断什么叫同步」的独立阶段，在本合集中独一无二。
- 阶段三：JAVG 训练——61 万条三元组，微调 cross-attention 与双向注意力模块。
【JavisDiT++（三阶段，参数量分配值得记录）】
- 阶段一 音频预训练：78 万条音频-文本，50 epoch，训练音频 FFN（794M 参数）。
- 阶段二 音视频 SFT：33 万条三元组，2 epoch，LoRA 训练（121M 参数）。
- 阶段三 AV-DPO：2.5 万条偏好对，1 epoch，LoRA 训练（121M 参数）。
注意 epoch 数的悬殊（50 : 2 : 1）与可训练参数量的下降（794M → 121M → 121M）：音频能力靠大量 epoch 从头学，联合能力靠少量 epoch 的 LoRA 适配，偏好对齐只需 1 个 epoch——这套配置对「如何在小数据上做联合生成」很有参考价值。
【Harmony（三阶段，划分依据是「音频基础 → 音色解耦 → 跨任务联合」）】
- 阶段一 基础音频预训练：10 万迭代，全局 batch 1536，最大片段 10 秒，语音与环境音 1:1 混合，参考音频为随机 1–3 秒片段。
- 阶段二 音色解耦微调：2 万迭代，跨话语错配的参考-目标配对。
- 阶段三 跨任务音视频联合训练：1 万迭代，batch 128，仍维持 1:1 数据配比，λv=0.1（视频驱动损失）、λa=0.3（音频驱动损失）。
- 全程 lr 恒定 1e-5。视频分支由 Wan2.2-5B 初始化。
【UniAVGen（三阶段，划分依据是「音频 → 联合 → 多任务」）】
- 阶段一 音频预训练：Emilia 英文子集，16 万步，batch 256，lr 2e-5。
- 阶段二 联合训练：内部真人音视频数据，3 万步，batch 32，lr 5e-6。
- 阶段三 多任务学习：1 万步，五类任务配比 4:1:1:2:2。
【共性规律（重要结论）】JavisDiT++、Harmony、UniAVGen、以及同期的 Ovi 全部采用「先用大量纯音频数据把音频分支训好，再用少量音视频配对数据做联合适配」的策略，且音频阶段的步数/epoch 远多于联合阶段。原因是高质量音视频配对数据稀缺而纯音频数据充裕，这已成为音视频联合生成领域的事实标准课程。
【共性缺失】五者均未采用「低清→高清」「短→长」「图像→视频」的渐进式课程，训练全程分辨率与时长固定。

### 数据配比随训练阶段的变化（预训练/退火annealing/SFT高质量子集） ⚠️

各阶段数据配比的披露程度差异很大，Harmony 与 UniAVGen 给出了明确数字：
【Harmony（最明确）】
- 阶段一与阶段三均严格维持「人类语音 : 环境音 = 1:1」的混合比例——这是本合集唯一贯穿多阶段的显式模态内类别配比，且在联合训练阶段仍然保持，说明团队认为语音能力与环境音能力必须同步维持而非分阶段偏置（与 Ovi「预训练语音为主 → 微调补音效」的分阶段偏置策略形成对照）。
- 损失权重层面的配比：λv=0.1（视频驱动生成任务的损失权重）、λa=0.3（音频驱动生成任务的损失权重）——注意这是 Cross-Task Synergy 中两个辅助任务的权重，音频驱动任务权重是视频驱动的 3 倍。
【UniAVGen（多任务配比明确）】阶段三多任务学习的五类任务配比为 4:1:1:2:2，是本合集中唯一给出多任务采样比例的工作。主任务（联合生成）占 4/10，两项占 1/10 的任务与两项占 2/10 的任务分别对应视频到音频配音、音频驱动动画等衍生任务[不确定：五个数字与五类任务的具体对应关系]。跨阶段数据配比方面：阶段一纯音频（Emilia 英文子集）→ 阶段二三纯内部音视频数据，是完全替换而非混合，未见防遗忘的音频数据回放[不确定]。
【JavisDiT / JavisDiT++】阶段间数据完全替换而非混合：
- 阶段一 78 万条纯音频 → 阶段二 33 万条音视频三元组 → 阶段三 2.5 万条偏好对，三者数据类型不同、无混合、无回放。
- 音频阶段内部：10 个数据集混合但配比未公开[不确定]，只知道「不做过滤以保证音效/音乐/语音三类的最大覆盖」。
- 一个隐含的配比决策：阶段二用 FunASR 把语音类视频剔除，相当于把音视频阶段的语音占比压到接近零——与阶段一的全类覆盖形成强烈反差。这可能导致音频分支在阶段一学到的语音能力在阶段二被削弱（灾难性遗忘风险），论文未讨论此问题[不确定]。
【MM-Diffusion / AV-DiT】单阶段训练，无阶段配比；两个数据集分别独立训练，无跨域混合。
【退火/高质量 SFT 子集】五者均未描述退火（annealing）阶段或末期高质量子集精调[不确定]，JavisDiT++ 的 DPO 阶段在功能上部分承担了这一角色。

### 后训练数据（SFT精选集规模与筛选标准、偏好对数量与标注方式、reward model训练数据） ⚠️

只有 JavisDiT++ 有完整的后训练数据体系，是本合集在此维度上的唯一贡献者，也是最值得借鉴的部分：
【JavisDiT++ 的 AV-DPO（音视频直接偏好优化）—— 完整披露】
- 偏好数据规模：约 2.5 万条音视频偏好对（约 25k audio-video preference pairs），训练 1 个 epoch，LoRA 121M 参数可训。
- prompt 池：3 万条文本 caption，明确与 SFT 训练数据不重叠（apart from the SFT training data），避免数据泄漏。
- 候选构造：对每条 prompt 用参考模型生成 N=3 个音视频对，加上 1 条真值样本，共 4 个候选组成候选组。
- 奖励模型组合（六个模型分工，是本合集最完整的多维奖励设计）：
  · 音频质量 → AudioBox（AudioBox-Aesthetics）
  · 文本-音频对齐 → ImageBind
  · 视频质量 → VideoAlign
  · 文本-视频对齐 → ImageBind
  · 音视频跨模态相似度 → ImageBind
  · 时序同步性 → Synchformer
- 配对策略：采用「归一化的模态感知排序（normalized modality-aware ranking）」选取 winning/losing 对，作者明确说明目的是「保证每个模态内部的一致性，而不是把优质音频与劣质视频混搭配对」——这是多维奖励下构造偏好对的核心工程经验，直接对应「优质音频 + 劣质视频」这类矛盾样本会污染偏好信号的问题。
- 一个有参考价值的量化观测：最终偏好数据中约 30% 的 winning 样本来自模型生成而非真值，作者据此判断「基线模型本身已具备相当强的生成能力」。这个比例可作为判断模型是否已到达「可以用自生成数据做偏好优化」阶段的经验信号。
- 开源状态：DPO 偏好数据「正在准备发布」，尚未公开[不确定]。
【JavisDiT（v1）】无 DPO/RLHF，三阶段全为监督训练。
【Harmony】无偏好优化数据。其「跨任务协同训练（Cross-Task Synergy）」在功能上部分类似——用双向生成任务提供额外监督信号来抑制对齐漂移，但属于多任务监督学习而非偏好学习。
【UniAVGen】阶段三的多任务学习（五任务配比 4:1:1:2:2）同样是多任务监督而非偏好优化；无 SFT 精选集、无偏好对、无 reward model[不确定]。
【MM-Diffusion / AV-DiT】无后训练环节。
【横向意义】JavisDiT++ 的 AV-DPO 是音视频联合生成领域较早的系统性偏好优化实践，其「多奖励模型 + 模态感知排序」的设计可直接迁移到其他 AV 模型的后训练。

### 数据处理基础设施与吞吐（NeMo Curator/Data-Juicer/自研，GPU加速比，处理规模，成本） ⚠️

本合集在数据基础设施上披露极少，且规模远小于工业模型，均未使用专业数据处理框架：
【共性】五者均未提及 NeMo Curator、Data-Juicer、Ray Data、Spark 等成熟数据处理框架，也未给出 GPU 加速比、处理吞吐（clip/小时）、处理成本等定量指标[不确定]。
【JavisDiT / JavisDiT++（披露相对最多，且工程可复现）】
- 数据管理采用 CSV 为中心的方案（沿用 Open-Sora 的数据管理体系），列包括 path、id、relpath、num_frames、height、width、aspect_ratio、fps、resolution、audio_path、audio_fps、text、audio_text；支持多来源 CSV 合并（merging）。这种轻量方案适合百万级以内规模，在千万级以上会成为瓶颈。
- 数据准备为一系列脚本化步骤（convert → get info → trim → resample → add dummy video → output CSV），完整文档化于 assets/docs/data.md，可复现性在本合集中最高。
- 训练侧：JavisDiT++ 的可训练参数量做了精细控制（阶段一 794M 音频 FFN、阶段二三各 121M LoRA），是明显的显存/算力优化设计；具体 GPU 卡型、卡数、训练时长未公开[不确定]。
【AV-DiT（唯一公开硬件规格的工作）】在 NVIDIA RTX A6000 GPU 上训练 10 万迭代、batch 16——这是消费级/工作站级硬件，与工业模型的千卡集群形成极端对比，也印证了其「参数高效」的定位（仅 adapter 可训练）。
【MM-Diffusion】未披露训练硬件与时长[不确定]。数据以预处理好的形式打包发布于 Google Drive 与百度网盘，方便复现。
【Harmony】可从批次规模反推算力：阶段一全局 batch 1536、10 万迭代，阶段三 batch 128、1 万迭代，配合 Wan2.2-5B 底座，应使用较大规模 GPU 集群，但卡数、卡型、训练时长均未公开[不确定]。400 万条片段的 Gemini 自动标注涉及大规模 API 调用，其成本与吞吐未披露[不确定]——这实际是该工作数据侧最大的隐性成本。
【UniAVGen】阶段一 batch 256、16 万步；阶段二 batch 32、3 万步（batch 骤降 8 倍，说明音视频联合训练的显存压力远高于纯音频）；硬件规格未公开[不确定]。

## 效果对比

### 数据策略消融的量化影响（区分：过滤严格度ablation / caption密度风格ablation / 数据配比ablation，及对应评测指标） ⚠️

本合集的数据消融整体偏少，绝大多数消融针对架构组件而非数据策略，仅 JavisDiT++ 有专门的数据消融：
【JavisDiT++ —— 唯一的数据质量/数量消融（附录 D.2）】论文正文 4.3 节引述其结论：「Sec. D.2 揭示，确保良好的数据质量是增加样本数量以提升训练效果的前提基础，为未来 JAVG 模型的规模化提供了新的洞察（ensuring good data quality is the foundation to increase the sample quantity to improve training efficacy, providing a new insight to scale up JAVG models in the future）」。这个结论的表述很关键——它不是简单的「质量优于数量」，而是「质量是数量能起作用的前提」，即在数据质量不达标时单纯扩量无效甚至有害。遗憾的是附录 D.2 的具体实验设置（对比了哪些规模/质量的子集、各自的指标数值）在公开 HTML 版本中未能获取[不确定]。
- 相关但独立的数据决策证据：音频阶段明确「不做任何过滤以最大化 T2A 能力覆盖三类音频」vs 视频阶段做三重质量过滤 + FunASR 语音剔除——这一「音频不过滤、视频严过滤」的反差本身就是团队对两种模态数据质量-数量权衡的实践判断。
- JavisDiT（v1）的消融使用 6 万条子集在 JavisBench-mini（1,000 条）上快速评估，属于为降低消融成本的采样策略，而非数据消融本身。
【Harmony —— 组件消融（非数据消融）但给出了完整的同步性增益链】按顺序累加四个组件，Sync-C 指标逐级提升：
- 加 Global-Local Decoupled Interaction（GLDI）模块（基线）
- 加 RoPE 位置对齐：Sync-C 4.29 → 4.80（+0.51）
- 加 Cross-Task Synergy 训练：Sync-C 4.80 → 5.09（+0.29）
- 加 Synchronization-Enhanced CFG（推理期）：Sync-C 5.09 → 6.51（+1.42，最大增益）
最有价值的发现是：推理期的同步增强 CFG 贡献的增益（+1.42）超过所有训练期改动之和（+0.80）——说明在同步性这个问题上，推理期引导的性价比极高。但这些都不是数据策略消融。数据侧的 1:1 语音/环境音配比未做消融验证[不确定]，「音视频一致性打分模型」筛选是否有效也未做对照实验[不确定]。
【UniAVGen —— 数据量对比而非消融】核心宣称「1.3M vs 30.1M 训练样本」下仍在音视频同步、音色一致性、情绪一致性上取得整体优势，这是与 Ovi 的跨模型对比而非受控消融（架构、底座、数据分布均不同），因此只能作为「架构效率高」的佐证，不能作为「小数据优于大数据」的严格证据。
【MM-Diffusion / AV-DiT】无数据消融。MM-Diffusion 的消融集中于 random-shift attention 的设计（shift 大小等）。
【共性缺口】五者均未做 caption 密度/风格的消融、未做过滤阈值扫描、未做 domain 配比消融[不确定]。

### 质量vs数量的证据（小而精数据超越大而杂的案例） ⚠️

本合集提供了两类不同性质的「质量/效率优先」证据，需区别看待：
【类型一：明确的质量-数量关系论断（JavisDiT++，最有价值）】
- 附录 D.2 的结论表述精准：「确保良好的数据质量是增加样本数量以提升训练效果的基础」——这不是常见的「小而精胜过大而杂」，而是更精细的层次关系：质量是门槛条件，数量是门槛之上的杠杆。这与 Ovi「即使少量不同步数据也会损害唇同步能力」的表述在精神上一致，但 JavisDiT++ 把它上升为可规模化的原则。
- 行为层面的印证：主动把 TAVGBench 110 万条砍到 35.5 万条（保留率 32.3%），宁可只用三成数据也要过滤掉含语音、低美学、低运动、多文字的样本。
- 但需注意：该团队在音频侧做了完全相反的决策（「不做任何过滤」以保证类别覆盖最大化），说明「质量优先」并非无条件原则，而是取决于该模态的瓶颈是质量还是覆盖度——音频数据本身较干净且类别覆盖是瓶颈，视频数据则质量参差且是主要瓶颈。这一「按模态区分过滤策略」的思路是本合集最细致的数据洞察之一。
【类型二：训练效率宣称（UniAVGen）】以 130 万训练样本对比 Ovi 的 3,010 万样本，在音视频同步、音色一致性、情绪一致性三项上取得整体优势。但这是跨模型对比而非受控实验：UniAVGen 用内部精选真人数据 + 非对称跨模态交互架构 + 三阶段课程，与 Ovi 的数据分布、架构、目标域（Ovi 覆盖通用场景，UniAVGen 聚焦真人）均不同，因此这个 23 倍的数据效率差距更多归因于「窄域聚焦 + 架构效率」而非纯粹的数据质量。作为「垂直域小数据可以打赢通用域大数据」的案例有参考价值，但不能直接推广。
【类型三：数据源质量分层意识（Harmony）】论文明确标注了三个音频数据源的标注质量属性——AudioCaps（约 128 小时，人工标注）、Clotho（约 31 小时，人工标注）、WavCaps（约 7,600 小时，自动标注），并对语音数据额外施加一致性打分筛选。这种「小而精的人工标注集 + 大而粗的自动标注集」并用的配置，本身就是质量与数量并重的实践，但未给出二者贡献的对照实验[不确定]。
【MM-Diffusion 的另一种质量优先】用 2.7 小时 + 5.2 小时共不到 8 小时的极小数据集，在 2022 年就实现了 80% 骗过人类的图灵测试通过率——靠的是数据集选型上的极致聚焦（9 类声画强绑定的自然场景 + 单一舞蹈域）。这是「窄域高保真小数据」路线的早期成功案例。
【反例警示】本合集全部工作的能力边界（≤10 秒、单镜头、固定分辨率、窄域）也说明：小而精的数据能验证方法有效性，但无法支撑通用能力——质量优先有其适用边界。

### 训练数据domain分布与评测基准类目体系的对齐关系（如VABench七大类） ⚠️

本合集中 JavisDiT 与 Harmony 都构建了自己的评测基准，但两者与训练数据分布的对齐程度截然不同：
【JavisDiT 的 JavisBench —— 有完整分类体系，但与训练数据分布刻意不对齐（是有意为之的「压力测试」设计）】
- 分类体系：五个评测维度、19 个场景类目——事件场景（Event Scenario）、视频风格（Video Style）、声音类型（Sound Type）、空间构成（Spatial Composition）、时间构成（Temporal Composition）。这是本合集中最系统的音视频评测类目体系，与 VABench 七大类属同一类努力。
- 难度设计：「超过 50% 的视频属于高度复杂与有挑战性的场景」「75% 的样本包含多个发声事件（multiple sounding events）」——刻意超出早期基线（Landscape 单一自然音、AIST++ 单一舞蹈）的简单设定。
- 与训练数据的关系：训练数据来自 TAVGBench（YouTube 通用分布，且剔除了语音），而 JavisBench 混合了 Landscape/AIST++/FAVDBench 测试集与 2024 年 6–12 月的新 YouTube 视频，并按 19 类均衡组织——两者类目体系无对应关系，训练侧甚至没有按 19 类做配比控制[不确定]。这意味着 JavisBench 更像一个「域外泛化压力测试」而非「训练分布的镜像评测」，作者也确实用它来暴露现有模型在复杂场景下的不足。
- 时间隔离设计值得注意：评测视频取 2024 年 6–12 月上传的内容，晚于多数模型的训练数据截止时间，降低了训练数据泄漏风险——这是一个规范的基准构建实践。
- 规模分层：完整版 10,140 条用于主实验，JavisBench-mini（随机抽 1,000 条）用于消融，兼顾严谨性与迭代成本。
【Harmony 的 Harmony-Bench —— 训练配比与评测类目严格对齐（是本合集唯一的对齐案例）】
- 三档子集各 50 条，共 150 条：环境音-视频（非语音声学事件同步）、语音-视频（唇同步保真度与多语种鲁棒性）、复杂场景（语音 + 环境音共现）。
- 与训练数据的对应关系非常直接：训练数据严格按语音 : 环境音 = 1:1 混合 → 评测集前两档正好对应这两类；训练时两类混合训练所要培养的「同时处理语音与环境音」的能力 → 第三档复杂场景子集正是这一能力的定向考察。这是「训练配比 ↔ 评测类目」的一一映射，在本合集中独此一家。
- 条件设置也随类目变化：环境音子集用音频/视频 caption、语音子集以转写为主、复杂场景用全套多模态 prompt——评测协议与三字段 caption schema 同样对齐。
【JavisDiT++ 的多维指标体系】11 个指标覆盖 4 个维度（质量 FVD/FAD、文本一致性 TV-IB/TA-IB/CLIP-Score/CLAP-Score、音视频一致性 AV-IB/AVHScore/JavisScore、同步性 DeSync），维度划分与其架构设计（MS-MoE 管单模态质量、TA-RoPE 管同步、AV-DPO 管三者综合）形成对应，属于「架构-指标对齐」而非「数据-指标对齐」。
【UniAVGen】评测用自建 100 条测试集（50% 真实图像 + 50% AIGC/动漫图像的二分，与训练数据的真人聚焦定位部分对齐）+ GRID（配音）+ EMTD（音频驱动动画）两个公开基准，属于按任务而非按内容类目组织；未建立类目分类体系[不确定]。
【MM-Diffusion / AV-DiT】无类目体系，评测按数据集分（Landscape / AIST++），指标只有 FVD 与 FAD，训练与评测使用同一数据集的不同划分，属于严格的域内评测——这也是早期工作泛化能力未被检验的原因。

## 不确定字段

以下字段的调研信息部分不确定（⚠️ 标注来源）：

- openness
- data_scale
- data_sources
- provenance_licensing
- duration_distribution
- resolution_aspect_distribution
- domain_distribution
- audio_category_distribution
- narrative_structure_distribution
- language_accent_distribution
- pipeline_overview
- funnel_retention_rate
- shot_segmentation
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
- data_infra_throughput
- data_ablation
- quality_over_quantity_evidence
- benchmark_taxonomy_alignment
