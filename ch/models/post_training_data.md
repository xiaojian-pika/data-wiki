# 视频生成后训练数据策略（跨模型横向专题）——以《A Systematic Post-Train Framework for Video Generation》(arXiv:2604.25427) 为锚，横向汇总各模型的 SFT 精选集规模/筛选标准与偏好对标注方式

> 主题: 视频生成模型（含音视频同时生成）的数据处理：数据清洗流程、数据分布、打标方式

[← 返回首页](../index.md)

**目录**: [基本信息](#基本信息) · [数据规模与分布](#数据规模与分布) · [清洗流程](#清洗流程) · [打标方式](#打标方式) · [音视频对齐](#音视频对齐) · [训练配合](#训练配合) · [效果对比](#效果对比)

## 基本信息

### 名称

视频生成后训练数据策略（跨模型横向专题）——以《A Systematic Post-Train Framework for Video Generation》(arXiv:2604.25427) 为锚，横向汇总各模型的 SFT 精选集规模/筛选标准与偏好对标注方式

### 发布机构/公司

多家。锚论文为香港大学（HKU，Zeyue Xue、Mengzhao Chen、Ping Luo）+ 京东探索研究院（JD Explore Academy，Siming Fu、Jie Huang、Shuai Lu、Haoran Li、Haoyang Huang、Nan Duan 等）+ 清华大学 + 北京大学 + 浙江大学的联合工作（Zeyue Xue 亦是 DanceGRPO 一作，Nan Duan 为通讯层作者）。横向覆盖对象包括：字节跳动 Seed（Seedance 1.0 / 1.5 pro）、腾讯混元（HunyuanVideo / HunyuanVideo 1.5）、快手可灵（Kling 3.0 Omni）、美团（LongCat-Video）、阶跃星辰（Step-Video-T2V）、昆仑万维（SkyReels-V2 / V4）、NVIDIA（Cosmos-Predict 2.5）、Meta（Movie Gen）、智谱（CogVideoX）、Rhymes AI（Allegro）、字节（Goku）、Moonshot/Motif（Motif-Video 2B）、Sand AI（MAGI-1）、Genmo（Mochi 1）、Lightricks（LTX-2）、OpenAI（Sora 2）、Google DeepMind（Veo 3/3.1）、生数（Vidu S1）、HPC-AI Tech（Open-Sora 2.0）、PKU-YuanGroup（Open-Sora Plan）、以及学术侧 JavisDiT++、NAVA、ALIVE 等。

### 发布时间（技术报告/论文/开源时间）

锚论文 arXiv:2604.25427v1 提交于 2026年4月28日（cs.CV，CC BY 4.0）。横向对象覆盖 2024年8月（CogVideoX）至 2026年上半年（Seedance 1.5 pro、Kling 3.0 Omni、SkyReels-V4、Cosmos-Predict 2.5、HunyuanVideo 1.5、LongCat-Video 等）。作为方法论支撑的关键论文时间线：ImageReward(2023)→VideoReward/Improving Video Generation with Human Feedback(2025-01, arXiv:2501.13918)→DanceGRPO(2025-05, arXiv:2505.07818)→Flow-GRPO(2025-05)→MixGRPO(2025-07)→HPSv3(ICCV 2025, arXiv:2508.03789)→Self-Forcing(2025-06)→OmniForcing(2026-03)、Causal Forcing(2026-02)、Astrolabe(2026-03)。

### 类型（模型/数据集/工具链/评测基准）

专题综述条目（cross-cutting topic），非单一模型/数据集/工具链/评测基准。锚论文本身类型为「方法论技术报告」（a practical blueprint / systematic framework），不发布模型权重、不发布数据集，属于四阶段后训练流水线的工程蓝图；其余横向对象则分属模型（多数）、数据集（HPDv3、VideoReward 偏好集）与奖励模型（HPSv3、VideoAlign/VideoReward、RewardDance、VisionReward）。

### 开源程度（权重/代码/数据/pipeline各自是否开源） ⚠️

锚论文（2604.25427）：论文开源（CC BY 4.0）、代码与权重均未开源、基座为「an internal video generation model」（内部模型）、SFT 数据集与 RLHF prompt 集均未开源，仅在图 2 用公开的 Wan-2.1 做 RLHF 效果可视化。整体属「方法开源、数据与模型封闭」。
横向的后训练数据开放度梯度（从高到低）：
① 偏好数据完全开源：HPDv3（108万文本-图像对、117万成对比较标注）、VideoReward 偏好集（1.6万 prompt / 10.8万视频 / 18.2万标注三元组，含 VideoGen-RewardBench）——这两者是目前视频/图像生成后训练最重要的公开偏好资产；
② 偏好数据「准备发布」：JavisDiT++ 的约 2.5 万条音视频偏好对（截至调研时尚未公开）[不确定]；
③ 方法与流程公开、数据不公开：SkyReels-V2（3万人工样本对 + 三阶段各 2万共约 6万 DPO 数据）、Step-Video-T2V（Video-DPO 流程全公开、数量未公开）、HunyuanVideo 1.5（RLHF prompt 集构造与 GSB 标注协议公开、规模未公开）、LongCat-Video（GRPO 三奖励配置公开、SFT 集规模与 RM 标注量未公开）、Cosmos-Predict 2.5（五域 SFT 规模逐条公开、GRPO 配置公开，数据不开源但发布 RL 后 EMA 权重）；
④ 仅有一句话或完全空白：Sora 2、Veo 3/3.1、LTX-2、Kling 3.0 Omni（仅说用了 DPO）、Seedance 1.5 pro（仅说用了 RLHF + 多维 RM）。
奖励模型侧开源程度显著优于生成模型侧：HPSv3、VideoAlign/VideoReward、VisionReward、Unified Reward Model 均开源权重，构成了「开源 RM + 闭源生成器」的事实标准组合。

### 是否支持音视频同时生成，以及实现方式（原生联合/级联/MoE融合）

本专题不是模型本身，但覆盖的对象含大量音视频联合生成模型，其后训练呈现明显分层：
【已把 AV 纳入后训练奖励的】Seedance 1.5 pro——明确采用「专为音视频场景定制的 RLHF 算法」与多维奖励模型，同时优化运动质量、视觉美学与音频保真度，并对 RLHF 流水线做基础设施优化带来近 3 倍训练加速；Kling 3.0 Omni——对同一 MVL（多模态视觉语言）条件采样多个视频变体，由人类评估者比较形成偏好对做 DPO（但音频维度是否作为独立打分项未披露）；JavisDiT++ 的 AV-DPO——六个奖励模型分工，其中时序同步性由 Synchformer 承担、音频质量由 AudioBox-Aesthetics 承担、文本-音频与跨模态相似度由 ImageBind 承担，是目前唯一完整披露 AV 偏好对构造细节的工作。
【锚论文的 AV 处理】仅在自回归蒸馏阶段涉及：对具备音视频生成能力的模型，遵循 OmniForcing（arXiv:2603.11647）为模型配备「非对称块因果对齐（asymmetric block-causal alignment）」与「音频 sink token」。即 AV 只体现在蒸馏架构层面，其 SFT 与 GRPO 的四个奖励模型（视频美学/图像美学/运动质量/文本-视频对齐）全部是纯视觉维度，不含任何音频或音视频同步奖励——这是该框架在 AV 时代的显著缺口。
【完全空白的】HunyuanVideo-Foley、Ovi（初版）、UniVerse-1、UniTalking、Unison、Foley-Omni、InstructAV2AV 等学术 AV 工作全部无偏好对齐后训练。

### 调研信息来源列表（论文/技术报告/官方文档/新闻的URL，标注每条来源的性质：官方一手/同团队旁证/第三方报道）

- [官方一手] A Systematic Post-Train Framework for Video Generation, arXiv:2604.25427v1, 2026-04-28（本条目锚论文，HKU + JD Explore Academy + 清华 + 北大 + 浙大）: https://arxiv.org/abs/2604.25427
- [官方一手] 同上 HTML 全文（含 GRPO 公式、等时分组、Temporal Gradient Rectification、四奖励模型定义、GSB 结果 31%/20%）: https://arxiv.org/html/2604.25427v1
- [官方一手] DanceGRPO: Unleashing GRPO on Visual Generation, arXiv:2505.07818（锚论文 GRPO 基础，同一一作 Zeyue Xue）: https://arxiv.org/abs/2505.07818
- [官方一手] Improving Video Generation with Human Feedback (VideoReward/VideoAlign), arXiv:2501.13918（1.6万prompt/10.8万视频/18.2万三元组/12个T2V模型/BTT带平局建模/Flow-DPO/Flow-RWR/Flow-NRG）: https://arxiv.org/abs/2501.13918
- [官方一手] HPSv3: Towards Wide-Spectrum Human Preference Score, ICCV 2025, arXiv:2508.03789（HPDv3：108万文本-图像对、117万成对比较标注）: https://arxiv.org/abs/2508.03789
- [官方一手] Flow-GRPO: Training Flow Matching Models via Online RL, arXiv:2505.05470: https://arxiv.org/abs/2505.05470
- [官方一手] MixGRPO: Unlocking Flow-based GRPO Efficiency with Mixed ODE-SDE, arXiv:2507.21802: https://arxiv.org/abs/2507.21802
- [官方一手] RePrompt: Reasoning-Augmented Reprompting for T2I via RL, arXiv:2505.17540（锚论文 Prompt Enhancer 所遵循的范式）: https://arxiv.org/abs/2505.17540
- [官方一手] Self Forcing: Bridging the Train-Test Gap in Autoregressive Video Diffusion, arXiv:2506.08009（锚论文 AD 阶段基础）: https://arxiv.org/abs/2506.08009
- [同团队旁证] OmniForcing: Unleashing Real-Time Joint Audio-Visual Generation, arXiv:2603.11647（锚论文 AV 蒸馏所遵循，作者与锚论文高度重合）: https://arxiv.org/abs/2603.11647
- [同团队旁证] Astrolabe: Steering Forward-Process RL for Distilled Autoregressive Video Models, arXiv:2603.17051（同团队，Zeyue Xue/Siming Fu/Nan Duan 等）: https://arxiv.org/abs/2603.17051
- [官方一手] Causal Forcing: Autoregressive Diffusion Distillation Done Right, arXiv:2602.02214: https://arxiv.org/abs/2602.02214
- [官方一手] BranchGRPO: Stable and Efficient GRPO with Structured Branching in Diffusion Models, arXiv:2509.06040: https://arxiv.org/abs/2509.06040
- [官方一手] E-GRPO: High Entropy Steps Drive Effective RL for Flow Models, arXiv:2601.00423: https://arxiv.org/abs/2601.00423
- [官方一手] TempFlow-GRPO: When Timing Matters for GRPO in Flow Models, arXiv:2508.04324: https://arxiv.org/abs/2508.04324
- [官方一手] Coefficients-Preserving Sampling for RL with Flow Matching (Flow-CPS), arXiv:2509.05952: https://arxiv.org/abs/2509.05952
- [官方一手] RewardDance: Reward Scaling in Visual Generation, arXiv:2509.08826: https://arxiv.org/abs/2509.08826
- [官方一手] VisionReward: Fine-grained Multi-dimensional Human Preference Learning, AAAI 2026: https://arxiv.org/abs/2412.21059
- [官方一手] Seedance 1.0: Exploring the Boundaries of Video Generation Models, arXiv:2506.09113（SFT 数百类目定向采集 + model merging；RLHF 多维标注协议；三个专用 RM）: https://arxiv.org/abs/2506.09113
- [官方一手] HunyuanVideo: A Systematic Framework for Large Video Generative Models, arXiv:2412.03603（100万人工精选 SFT，美学四项 + 运动三项 rubric）: https://arxiv.org/abs/2412.03603
- [官方一手] Kling-Omni Technical Report, arXiv:2512.16776（同 MVL 条件多变体 + 人类比较形成偏好对做 DPO）: https://arxiv.org/abs/2512.16776
- [官方一手] Wan: Open and Advanced Large-Scale Video Generative Models, arXiv:2503.20314: https://arxiv.org/abs/2503.20314
- [官方一手] VBench: Comprehensive Benchmark Suite for Video Generative Models, CVPR 2024: https://arxiv.org/abs/2311.17982
- [官方一手] Qwen3-VL Technical Report, arXiv:2511.21631（锚论文奖励模型主干「Qwen3.5」的引文指向）: https://arxiv.org/abs/2511.21631
- [本项目内部调研结果，二次汇总] 本条目的横向数字（Allegro/Apollo/CogVideoX/Cosmos-Predict25/Goku/HunyuanVideo/JavisDiT_baselines/Kling_30_Omni/LongCat-Video/Movie_Gen/Motif/Open-Sora/Seedance/SkyReels/Step-Video-T2V/Sora_2/Veo_3/caption_models/JAVG_2026_misc 等）均取自 /Users/jan/wangxj/data_survey/video-gen-data-processing/results/ 目录下各条目的 post_training_data 字段，其原始出处见各条目自身的 sources 列表

## 数据规模与分布

### 训练数据量级（视频条数/小时数/token数，预训练与SFT分开） ⚠️

【锚论文】未披露任何数字：正文 4.1 节 Dataset 全文仅两句——「First, we constructed a high-quality text-video dataset for SFT. Subsequently, we curated the prompt set as described in the experimental settings.」SFT 集条数、RLHF prompt 集条数、group size N、训练步数、GPU 规模全部缺失。这是本条目最需要指出的一点：一篇以「后训练系统框架」为标题的报告，在数据规模维度上是零披露的。
【横向 SFT 精选集规模谱系（可对标的公开数字）】
· Step-Video-T2V：约 3000 万（30M）条高质量视频；
· Cosmos-Predict 2.5：按五域拆分——物体恒存 10.4M、驾驶 3.1M、复杂场景 1.6M、高运动 1.0M、机器人操作 730K，另 4K 冷却数据 388K，另驾驶多视图 1.5M 段 7 相机 20 秒 30FPS clip；
· SkyReels-V4：SFT 两阶段 500 万条带多模态条件视频（3 epoch）→ 100 万条人工精选（3 epoch）；
· ALIVE：Continue-Training 4.3M → SFT 5M（0.5 epoch）→ 1080p Refiner 0.7M → 角色专项 0.8M；
· Open-Sora 2.0：阶段三 5M 高分辨率精选；Open-Sora 1.2：2M 片段 / 5k 小时；Open-Sora Plan：I2V 阶段二 15M；
· Goku：I2V 微调 4.5M 文本-图像-视频三元组（占 36M 视频池 12.5%）；
· Allegro：约 2M（占 500M 原始片段 0.4%）；
· HunyuanVideo 原版：约 100 万人工精选；HunyuanVideo 1.5：CT 每任务 100 万，SFT 规模未公开；
· CogVideoX：预训练数据中质量最高的 20% 子集，10k 步；
· NAVA：从 15M 中精选 160K（保留率约 1.07%），且用更贵的 Gemini-3-Pro 对该子集重打 caption；
· Movie Gen：视频 SFT 集规模未披露但「训练只用 512 张 H100」，PT2V SFT 集 O(1000) 条，音频 SFT 集 cinematic split O(100)K 样本 / O(1)K 小时 + high-quality audio split O(1,000)K 样本 / O(10)K 小时；
· Motif-Video 2B：两次 SFT（480p Stage 7、720p Stage 10），规模未给。
【偏好数据规模谱系】HPDv3 108 万文本-图像对 / 117 万成对比较；VideoReward 1.6 万 prompt / 10.8 万视频 / 18.2 万标注三元组（其中 1.3 万三元组作 held-out 验证集，其 prompt 不出现在训练集）；SkyReels-V2 3 万人工样本对训练 Bradley-Terry 奖励模型 + 每阶段 2 万 × 3 阶段共约 6 万条 DPO 数据；JavisDiT++ 约 2.5 万条音视频偏好对（prompt 池 3 万条、1 epoch、LoRA 121M 可训参数）；HunyuanVideo 1.5 T2V 侧 RLHF prompt 集为 O(10K) 万级。
【量级规律】SFT 精选集的典型量级为 10^6–10^7 条、占预训练池 0.4%–20%；偏好数据的典型量级为 10^4–10^5 对，比 SFT 集小 2–3 个数量级——这与 LLM 后训练的比例结构一致。

### 数据来源构成（自有/公开数据集/网络爬取/授权采购/合成数据） ⚠️

【锚论文】仅称 SFT 用「high-quality text-video dataset」、RLHF 用「curated prompt set」，来源构成完全未说明；基座为内部模型，故 SFT 数据大概率来自其预训练语料的高分子集，但无文字依据 [不确定]。
【横向的后训练数据来源四类范式】
① 预训练池内的高分子集（最普遍）：CogVideoX（top 20%）、Allegro（500M→2M）、Motif、Open-Sora、MAGI-1（最后阶段用更严过滤数据）、NAVA（15M→160K）——本质是「同源提纯」；
② 定向采集/独立采购的精品集：Seedance 1.0 按视觉风格/运动类型等属性定义「数百个类别」做定向采集；Movie Gen 人工挑「影视感（cinematic）」素材并重写 caption；LongCat-Video 额外并入相机运动与视觉风格专项数据集；
③ 模型自生成数据（RLHF/DPO 的候选来源）：Step-Video-T2V 对每条 prompt 用不同随机种子生成多个视频；Kling 3.0 Omni 对同一 MVL 条件采样多个变体；JavisDiT++ 每 prompt 生成 N=3 个候选加 1 条真值共 4 个；Seedance 1.0 的偏好数据「涵盖模型不同阶段生成的合成视频等多来源素材」；StreamChar 蒸馏 Stage II 直接用学生模型的在线 rollout；
④ 线上真实用户 prompt 回流：Seedance 1.0 明确「从训练集与线上用户收集 prompt，做数据均衡与信息过滤以剔除重复与含糊 prompt」——这是闭源商业模型相对学术工作的结构性优势。
【prompt 集与 SFT 集去重的必要性】JavisDiT++ 明确其 3 万条 DPO prompt 池「与 SFT 训练数据不重叠（apart from the SFT training data）」；VideoReward 保留 1.3 万 prompt 从不出现在训练集的三元组作验证集。二者共同说明后训练阶段 prompt 泄漏是一个被明确防范的风险点。

### 数据合规与溯源（授权数据占比、rights-cleared数据集、C2PA等） ⚠️

锚论文与绝大多数横向对象在后训练数据的版权与溯源维度均无任何披露 [不确定]。可记录的相关事实：
· 后训练数据因规模小（10^4–10^7），理论上比预训练更容易做到 rights-cleared，Movie Gen 的「人工挑影视感素材 + 人工重写 caption」流程与 SkyReels-V4 的「100 万条人工精选」在成本结构上具备逐条授权审查的可行性，但均未声明；
· 偏好数据的额外合规维度是「标注员劳动」——Step-Video-T2V 提到偏好标注「全程由质控人员监督一致性」，HunyuanVideo 1.5 用 GSB 标注，Seedance 1.0 用「在指定维度选最优/最差，同时保证最优者在其他维度不劣于最差者」的多维度标注协议，但均未披露标注团队规模、来源、报酬与培训方式；
· 模型自生成的偏好候选（Seedance 1.0、Step-Video、Kling、JavisDiT++）规避了外部素材版权问题，但引入了「合成数据回灌」的分布风险；
· 公开偏好数据集 HPDv3（1.08M 文本-图像对）与 VideoReward（10.8 万视频来自 12 个 T2V 模型：Gen2、SVD、Pika 1.0、Vega、PixVerse v1/v2、HiDream、Dreamina、Luma、Gen3、Kling 1.0/1.5）——后者的视频全部由商业模型生成，其再分发的许可状态是一个未被讨论的灰色地带。[不确定]

### 片段时长分布与切分策略 ⚠️

【锚论文】未披露 SFT 数据时长分布与切分策略 [不确定]。仅从自回归蒸馏（Self-Forcing + KV cache 逐帧 rollout）可推断其部署目标是流式长视频。
【横向可对标数字】Movie Gen 视频 SFT 集时长 10.6–16 秒且 50% 为 16 秒（明显偏长、偏向完整叙事）；Allegro SFT 严格限定 6–16 秒；MAGI-1 最后阶段放宽到 ≤16s；LongCat-Video 的 SFT 与 GRPO 均为 93 帧；Cosmos-Predict 2.5 驾驶多视图 SFT 为 20 秒 30FPS。
【规律】后训练阶段普遍把时长上限推到该模型能力的极限（16 秒左右）并偏好长片段，与预训练阶段偏好短片段（2–8 秒）相反——因为 SFT 要教的是「完整镜头的叙事与运动完整性」，而 RLHF 阶段则因 rollout 成本极高（锚论文明确「rollout generation is expensive」）反而倾向用较短片段。

### 分辨率/宽高比分布与分桶策略 ⚠️

【锚论文】未披露 [不确定]。
【横向】后训练阶段几乎一致地采用「分辨率阶梯的最后一级」：CogVideoX SFT 为 768×1360；Allegro SFT 要求 ≥1280×720；LongCat-Video SFT 与 GRPO 均为 480p+720p 混训；Motif-Video 2B 设 480p 与 720p 两次 SFT，且 720p 预训练从 480p SFT checkpoint 起步（而非从预训练 checkpoint 起步）——这是把 SFT 插进分辨率阶梯中间的非常规做法；SkyReels-V2 先做 540p 概念均衡 SFT 再做 720p 高质量 SFT；HunyuanVideo 1.5 的 CT 阶段为 480p/720p、24fps，另用 100 万条 1K–4K 片段单独训超分模块；ALIVE 用 0.7M high-clarity 样本训独立的 1080p Refiner。
【规律】SFT 是分辨率课程的收尾环节，而 RLHF/GRPO 因 rollout 成本通常在比 SFT 更低或持平的分辨率上进行（LongCat 保持 480p+720p、Cosmos 用 8 rollout × 20 扩散步的极简配置）。宽高比分桶在后训练阶段普遍不再单独讨论。[不确定]

### 类别/domain分布与配比策略（人物、动作、场景、风格等比例控制与概念均衡） ⚠️

这是后训练数据策略中区分度最高、也最能体现团队工程成熟度的维度。锚论文对此完全空白（仅说「curated the prompt set」）[不确定]，但横向存在四种清晰可辨的配比范式：
【范式一：密度倒数采样做长尾提升（最优雅）】LongCat-Video 的 SFT 精选集在多指标过滤（美学分/视频质量/运动质量）之上，第二层按「样本在 caption 嵌入空间中的密度的倒数」采样（inversely proportional to their density in the caption embedding space），直接实现长尾概念的相对提升。这是把「概念均衡」形式化为可计算准则的代表做法。
【范式二：k-NN 概念平衡 + 人工影视感筛选】Movie Gen 的视频 SFT 集经四阶段产出：自动严阈值过滤 → k-NN 概念平衡 → 人工挑「影视感」 → 人工重写 caption；论文明确指出第一阶段后剩「几百万条但概念不均衡」，即概念均衡是独立于质量过滤的第二道工序。
【范式三：分类器分域 + 逐域独立 SFT（Physical AI 路线）】Cosmos-Predict 2.5 在 InternVideo2 embedding 上训练多头分类器把样本分入五域，逐域训练独立 SFT 模型（每域 30k iterations、batch 256），再做模型合并。域内规模差异极大（物体恒存 10.4M vs 机器人操作 730K），反映其对「物体不因遮挡而消失」这一基本物理常识的优先级。逐域人工胜率均显著优于预训练基线：机器人操作 72.6% vs 8.3%、物体恒存 50.9% vs 27.7%、高运动 44.0% vs 34.7%、复杂场景 42.6% vs 35.4%、驾驶 47.9% vs 28.8%。
【范式四：数百类目定向采集 + 子模型融合】Seedance 1.0 按视觉风格、运动类型等属性定义「数百个类别」做定向采集，训练多个覆盖不同风格/运动/场景的子模型再做 model merging，并用比预训练更小的学习率、有限 GPU 数配合早停以防过拟合、保持文本可控性。这是「用多个专精 SFT 模型的权重平均替代单一混合 SFT 集」的思路，与 Movie Gen 的 model averaging 同源。
【偏好数据侧的配比】HunyuanVideo 1.5 的 T2V RLHF prompt 集在「运动/场景/主体」三个维度上做类目平衡，来源为 LLM 生成 prompt 与训练 caption 混合；I2V 侧构建覆盖 100+ 类别的 prompt 集，配图从高美学图像中精选并经人工核验图文一致性。Seedance 1.0 的 RLHF prompt 做「数据均衡与信息过滤」以剔除重复与含糊 prompt。
【Motif-Video 2B 的迭代补短板】SFT 语料组装采用迭代式补短板：在常规过滤基础上叠加更严的美学截断、由 style/subject 标签驱动的 domain-balancing、以及视频侧 action=Dynamic 的动态运动准入。

### 音频类别分布与配比（语音/音效foley/音乐/环境音/静音的比例及控制策略）——AV模型独有维度 ⚠️

锚论文的四个奖励模型（视频美学、图像美学、运动质量、文本-视频对齐）无一涉及音频，SFT 数据的音频类别配比亦无任何说明——这是该框架相对 2026 年 AV 主流的显著缺口 [不确定]。
【横向仅三例给出音频侧后训练配比】
① Movie Gen 音频 SFT 集是本主题中披露最细的：分两个 split——cinematic split（专业制作、含画内音与画外环境/主题音乐、明确排除含人声片段，由影视感分类器 + AED 声音事件检测自动筛后人工选定）O(100)K 样本 / O(1)K 小时；high quality audio split（无视频的高质量音乐 O(10)K 小时 + 音效 O(10)K 小时）O(1,000)K 样本 / O(10)K 小时。「排除含人声片段」是一个关键决策：Movie Gen Audio 定位为音效+音乐生成而非对白生成，因此在 SFT 阶段主动剔除语音类样本。
② Foley-Omni 的 Stage 3（V2ST 完整配乐微调）用规模最小的 216 小时数据做 2 epochs 轻量微调，并配每域 100 小时回放防遗忘，准入条件是「含多个音频组件」；
③ JavisDiT++ 的 AV-DPO 用 AudioBox-Aesthetics 单独评估音频质量、用 Synchformer 单独评估时序同步性，即在偏好排序层面把音频作为独立模态处理，但未披露语音/音效/音乐的比例。
【Seedance 1.5 pro】称多维奖励模型覆盖「音频保真度（audio fidelity）」，但语音/音效/音乐是否分维度打分未披露 [不确定]。Kling 3.0 Omni 的音频维度偏好标注（口型同步度、音色一致性是否作为独立打分项）同样未披露 [不确定]。

### 叙事结构分布（单镜头vs多镜头、平均clip时长、镜头数分布、是否含原生音轨） ⚠️

锚论文未涉及 [不确定]。横向可记录：
· Movie Gen 视频 SFT 集 50% 为 16 秒的长片段，且人工筛选标准是「影视感（cinematic）」，暗示以单镜头完整叙事为主；
· 后训练阶段普遍偏好单镜头无转场素材——Step-Video-T2V 的人工评审标准之一即「场景转换是否平滑」，Motif 的 SFT 准入含 action=Dynamic；
· 多镜头能力主要由预训练与推理侧的 prompt 结构承担，未见任何工作在 SFT/偏好数据层面显式控制镜头数分布 [不确定]；
· 自回归蒸馏（锚论文 Phase 4、StreamChar、LongLive、OmniForcing、Causal Forcing）把叙事结构问题转成「长时 rollout 误差累积」问题，其训练数据是学生模型自生成的在线 rollout 而非采集素材——这是叙事结构维度上一条与数据采集完全不同的技术路线。

### 语言/口音分布（多语种唇同步能力的数据基础） ⚠️

锚论文与绝大多数横向对象在后训练阶段均无语言/口音配比披露 [不确定]。可记录的间接事实：
· 锚论文的 prompt enhancer（Phase 3）是用 GRPO 训练的 LLM，其训练 prompt 的语种构成未说明 [不确定]；
· MOVA 构建 732 条中英双语 Arena 评测集（用于评测，非训练）；UniTalking 的 20 人 × 50 条盲测、Unison 的 40 样本 × 25 人排序投票同为评测用；
· HPDv3 与 VideoReward 两个公开偏好集均以英文 prompt 为主 [不确定]，中文场景的偏好数据是公开生态的明确空白；
· HunyuanVideo 1.5 的 RLHF prompt 集来源为「LLM 生成 prompt 与训练 caption 混合」，语种未说明 [不确定]。

## 清洗流程

### 清洗漏斗整体结构（几级过滤、各级顺序） ⚠️

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

### 漏斗定量保留率（每级过滤的输入/输出量与最终保留率，如Apollo 27%） ⚠️

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

### 镜头切分方法（PySceneDetect/自研模型/shot-aware splitting） ⚠️

锚论文未涉及 [不确定]。后训练阶段一般不再重做镜头切分——SFT 精选集是从已完成切分的预训练池中筛选的子集，因此镜头切分属于上游预训练数据 pipeline 的职责。横向唯一相关的是 Step-Video-T2V 的 SFT 人工评审把「场景转换是否平滑」列为四项标准之一，即在后训练阶段用人工把关切分残留问题（一个 clip 内混入转场）——这是「后训练阶段作为切分质量最后一道防线」的少数明证。Motif-Video 2B 的 SFT 准入含 action=Dynamic，间接排除了静止的转场残片。

### 质量过滤（美学评分、清晰度、OCR文字过滤、黑边/水印/logo检测） ⚠️

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

### 运动过滤（光流/运动分数阈值、静态与抖动剔除） ⚠️

锚论文在数据侧无运动过滤描述，但在奖励侧设有专用的 Motion Quality 奖励模型（评估运动动态的真实性、平滑性与连贯性，抑制抖动、不连续运动、时序不一致的物体转换），并在结果中报告运动质量是 RLHF 增益最显著的两个维度之一 [数据侧不确定]。
【横向】
· Allegro SFT：UniMatch 光流运动分数区间 [1.0,100]；
· HunyuanVideo 原版 SFT 人工标准含运动速度、动作完整性、运动模糊三项；
· Motif-Video 2B SFT 显式要求 action=Dynamic；
· SkyReels-V2 的 RL 目标明确聚焦运动质量（动态一致性与流畅度）而非通用美学偏好，其偏好数据的自动侧样本正是「对真实视频施加受控失真」生成的损坏样本；
· LongCat-Video 的 Motion Quality 奖励模型有一个值得借鉴的设计：以 VideoAlign 为基座在内部标注数据上微调，输入灰度视频（去色以迫使模型只评估运动而不被色彩/美学干扰）——这是解耦运动奖励与美学奖励的直接工程手段；
· Cosmos-Predict 2.5 单列「high motion」域（1.0M 条）做专项 SFT。

### 去重方法（精确去重与基于embedding的语义去重分别记录） ⚠️

锚论文未涉及 [不确定]。后训练阶段的去重关注点与预训练不同：不是去除重复素材，而是去除重复/含糊的 prompt。Seedance 1.0 明确在 RLHF prompt 收集后做「数据均衡与信息过滤以剔除重复与含糊 prompt」；Step-Video-T2V 邀请标注员依指引合成补充 prompt 以保证 prompt 多样性；JavisDiT++ 保证 3 万条 DPO prompt 池与 SFT 数据不重叠；VideoReward 保留 1.3 万 prompt 从未出现在训练集的三元组作验证集。
素材级去重在 SFT 阶段最典型的形态是 Step-Video-T2V 的 VideoCLIP K-means 簇内离群剔除（形式上是聚类，作用上兼顾了语义去重与离群剔除）与 LongCat-Video 的 caption 嵌入密度倒数采样（对高密度即近重复区域降采样，是一种连续化的软去重）。这两者代表了「用 embedding 空间密度替代硬去重阈值」的方向。

### VLM/LLM作为质检员（多模态大模型质量打分与错配剔除，2026年从浅层打分器转向大模型语义判断的趋势）

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

### 安全与合规过滤（NSFW、版权、人脸/隐私） ⚠️

锚论文 3.1 节把「unsafe outputs（不安全输出）」列为 SFT 阶段要消除的失败模式之一，但无任何具体过滤方法、分类器或数据处理描述 [不确定]（且如前所述该表述疑似来自 LLM 后训练文本的迁移）。第 6 节 Broader Impact 讨论的是商业应用价值而非风险缓解。
【横向】Sora 2 是唯一在后训练维度详述安全的对象，但其内容属于安全对齐评测而非能力后训练：通过定向红队收集「数千条对抗性 prompt」按用例与政策领域分类，用 helpful-only 版本视频模型生成输出后打分，转化为自动化评测集，测量 not_unsafe 与 not_overrefuse 两项指标（成人裸露/性内容不涉肖像 96.04%/96.20%、涉肖像 98.40%/97.60%、自残 99.70%/94.60%、暴力血腥 95.10%/97.00%、违规政治说服 95.52%/98.67%、极端主义仇恨 96.82%/99.11%）。Veo 3/3.1 官方所称的「post-training mitigations」实指 SynthID 水印与生产环境输出过滤，属部署侧而非数据侧。InstructAV2AV 的 Qwen3-Omni 五维自动验证把「安全性」作为训练样本准入条件之一，是学术侧少见的把安全并入 SFT 准入的做法。
【整体判断】后训练阶段的安全工作在公开材料中几乎全部集中在「输出侧拦截 + 红队评测」，而非「偏好数据中构造安全偏好对」。用 RLHF 做安全对齐（LLM 领域的标准做法）在视频生成领域尚未见公开实践。[不确定]

## 打标方式

### 使用的caption模型（自研VLM/开源模型，模型规模） ⚠️

锚论文未披露 SFT 数据的 caption 模型 [不确定]，但其 Phase 3 提示增强（PE）实质上是「用 GRPO 训练一个 LLM 做 prompt 改写」，采用 RePrompt（arXiv:2505.17540）范式，冻结生成主干、只优化策略 πθ，因此可无缝套用到任意现成 T2I/T2V 生成器。PE 的模型规模、基座名称与训练 prompt 规模均未披露 [不确定]。
【横向后训练阶段的 caption/prompt 模型实践】
· NAVA 的做法最值得注意：SFT 精选子集（160K）的 caption 由更贵的 Gemini-3-Pro 重新生成（原预训练 caption 由 Flash 生成），即「精选子集 + 升级标注质量」的双重提纯——SFT 阶段不仅换数据，还换标注器；
· Open-Sora 2.0：阶段三 5M 精选数据用 Qwen2.5-Max 重新打标；
· Movie Gen：SFT 集的 caption 由人工重写（四阶段的最后一阶段）；
· Step-Video-T2V：SFT 的人工评审环节同时对 caption 做优化；
· MAGI-1 的推理侧 Prompt Enhancement 小模型（约 7B）用约 200 万条 MLLM 生成的增强 prompt 语料蒸馏而成，并过滤掉目标文本过长的样本以控制输出长度——是 PE 训练数据规模的唯一公开数字；
· Open-Sora Plan 的 prompt refiner：19,500 条 caption、LLaMA-3.1-8B LoRA（rank 64，batch 32，1 epoch，单 H100 30 分钟）——PE 的成本下界样本；
· MOVA 用推理侧 prompt rewriter（Qwen3-VL 抽结构化视觉描述 + Gemini 2.5 Pro 通过 in-context learning 改写为符合训练分布的 prompt）替代后训练指令对齐，人类 Arena ELO 从 982.9 提升到 1025.3——量化证明了 PE 的独立价值。
【规律】重打标（re-captioning）是 SFT 阶段的标配动作，而非可选项：预训练用便宜模型批量打标，SFT 子集用最贵的模型或人工重打，是当前公认的成本最优配置。

### caption密度与结构化程度（短/长/密集描述、结构化字段如镜头运动、风格标签） ⚠️

锚论文未披露 SFT caption 的结构与密度 [不确定]。PE 的奖励设计中含一项「Structure Reward（结构奖励）」，强制格式合规与长度约束以保证 prompt 有效可执行——这是唯一与 caption 结构相关的信息，说明其增强后的 prompt 有明确的结构化模板，但模板内容未公开 [不确定]。
【横向】后训练阶段的 caption 结构趋势是「更长、更密、更结构化」：NAVA 的 SFT caption 由 Gemini-3-Pro 生成「更准确、更结构化、时序落地（temporally grounded）」的音视觉描述；HunyuanVideo 1.5 的 RLHF prompt 集来源为 LLM 生成 prompt 与训练 caption 混合，说明训练 caption 与用户 prompt 的分布差异被显式建模；MOVA 的 rewriter 明确目标是「弥合用户输入与训练数据分布的 gap」——这实际上揭示了 SFT caption 结构设计的核心张力：caption 越密越有利于训练，但离用户实际 prompt 分布越远，因此必须配套 PE 才能兑现收益。锚论文把 PE 单列为一个阶段，正是对这一张力的系统性回应。

### 音视频联合caption结构（是否同时覆盖视觉+听觉轨道、是否分流为独立字段，如LTX-2全音景描述、Script-a-Video factorized streams、Foley-Omni三字段） ⚠️

锚论文完全未涉及音视频联合 caption [不确定]——其四个奖励模型全为视觉维度，PE 的三项奖励（文本-视频对齐、视频美学、结构奖励）亦无音频项。对于具备 AV 生成能力的模型，论文只在蒸馏阶段引用 OmniForcing 的非对称块因果对齐与音频 sink token，属架构而非标注 schema。
【横向后训练侧的 AV caption 实践】
· NAVA 的 SFT 子集用 Gemini-3-Pro 重打的正是「更准确、结构化、时序落地的音视觉 caption（audio-visual captions）」，是后训练阶段升级 AV 联合标注的明确案例；
· Movie Gen 音频 SFT 集的 cinematic split 由影视感分类器 + AED（声音事件检测）自动筛后人工选定，标注体系区分画内音与画外环境/主题音乐；
· JavisDiT++ 的偏好排序在文本-音频与文本-视频两条对齐链路上分别用 ImageBind 打分，等价于把 caption 分流为视听两个评估轨道；
· 生态侧 captioner 的后训练已高度成熟：AVoCaDO 的 GRPO 奖励含五维 checklist 覆盖度（GPT-4.1 判定）、对白 F1（speaker 准确率 + content 编辑距离 DP 对齐相似度，阈值 0.6）、长度正则（>4096 token 惩罚）；AVSCap 的 GRPO 混合奖励 = 长度控制 + 语音保真 + 音视一致性，并给出关键论断「RL 增益 > 扩 SFT 数据量」；腾讯混元 1.5 的 caption 模型用 OPA-DPO 抑制多模态幻觉。这些是 caption 侧后训练的成熟范式，但尚未反向传导到视频生成模型自身的 AV 偏好奖励设计中。

### 对白转写与说话人属性标注（ASR转写+说话人身份/语言/口音/情绪） ⚠️

锚论文完全未涉及 [不确定]。横向：后训练阶段的对白与说话人属性标注几乎空白——Movie Gen 音频 SFT 的 cinematic split 反而明确排除含人声片段；Seedance 1.5 pro 称奖励覆盖音频保真度但未说明是否含对白准确率或说话人一致性 [不确定]；Kling 3.0 Omni 的偏好标注是否含口型同步度与音色一致性未披露 [不确定]。
生态侧的可迁移经验来自 captioner 后训练：AVoCaDO 的对白 F1 奖励（speaker 准确率 + 内容编辑距离 DP 对齐相似度，阈值 0.6）是目前唯一把「对白转写准确性」形式化为可优化奖励的设计，理论上可直接迁移为 AV 生成模型的对白维度奖励，但截至调研时尚无生成侧工作这样做。

### 几何与结构化标注（相机参数、深度、3D point tracks、动作标注、显式状态） ⚠️

锚论文未涉及 [不确定]。横向仅两例：Cosmos-Predict 2.5 的领域特化后训练用「world scenario map」（由 HD 地图与 3D 框投影而来）作为 Cosmos-Transfer2.5 驾驶控制网的控制信号，是后训练阶段使用几何结构化标注的最明确案例；LTX-2 发布了相机运动、姿态控制、唇形配音（lip dubbing）等一系列控制 LoRA，显然需要专门构造的几何/姿态配对数据集，但构造方法完全未公开 [不确定]。LongCat-Video 在 SFT 阶段并入「相机运动与视觉风格」的专项数据集以强化指令跟随，但未说明是否含显式相机参数标注 [不确定]。LongCat Avatar 1.5 在 GRPO 中引入首帧手部可见性检查以优先采样含手样本，是把结构化检测器用于 rollout 采样调度的独特做法。

### 合成数据构造（受控扰动/编辑构造训练对，如InstructAV2AV）

后训练阶段的「合成数据」有两条完全不同的路径，本条目需严格区分：
【路径一：受控失真构造负样本（自动化偏好对）】SkyReels-V2 的「半自动数据生产线」是最完整的公开案例——自动侧通过对真实视频施加受控失真生成损坏样本，与真实视频天然构成偏好对，覆盖 V2V、I2V、T2V 三种变体，与人工标注的 3 万样本对互补。这条路径的优势是零标注成本、负样本可控；劣势是失真类型与模型真实失效模式可能不匹配。
【路径二：模型自采样构造候选组（在线/离线偏好对）】锚论文的 GRPO 属此路径的在线形态：给定 prompt c，从参考策略采样 N 条轨迹组成一组，用组内归一化优势 A_i =（R_i − mean）/ std 做相对比较，无需预标注偏好对。Cosmos-Predict 2.5 同为在线形态（每条件 8 rollout × 20 扩散步、组内归一化、训练 256 steps、batch 32）。离线形态则有 Step-Video-T2V（每 prompt 不同随机种子生成多个视频后人工标注）、Kling 3.0 Omni（同一 MVL 条件多变体 + 人类比较）、JavisDiT++（每 prompt 3 个生成 + 1 条真值共 4 候选）。
【锚论文奖励模型训练中的合成数据】Stage 2 明确「利用由不同能力等级模型与不同 RL 迭代产生的无标注数据」训练奖励模型——即用模型演化过程中的中间产物作为 RM 的额外训练素材，这是一个成本极低且分布贴合的合成数据来源，也是 Seedance 1.0「偏好数据涵盖模型不同阶段生成的合成视频」的同类做法。
【蒸馏侧的自生成数据】锚论文 Phase 4 的 Self-Forcing 蒸馏中，每帧基于此前自生成输出做自回归 rollout（带 KV cache），DMD loss 在视频级施加——训练数据即学生模型自己的 rollout，是纯粹的自生成数据。StreamChar 的两阶段蒸馏 Stage II（400 steps，student lr 2e-6、fake score network lr 4e-7）同样用在线 chunk rollout。

### 人工介入程度（人工标注、人工质检、模型初筛+人工复核） ⚠️

【锚论文】人工只出现在两处：① 奖励模型训练所用的「human preference annotation (y1,y2)」，标注量、标注协议、标注员规模全未披露 [不确定]；② 最终评测采用 GSB（Good–Same–Bad）协议，并明确「we ask the human artist to give an overall comparison for the results」——由人类艺术家给出整体比较。论文选择 GSB 而非强制二选一的理由说得很清楚：GSB 允许标注者在差异细微时表达「无差别」，从而减少边缘case上被迫做出的噪声判断，这对视频评测尤其重要。
【横向的人工介入形态谱系（后训练是人工密度最高的阶段）】
· 纯人工偏好标注：Step-Video-T2V（标注员对多种子生成结果做偏好评分，全程由质控人员监督一致性）、Kling 3.0 Omni（人类评估者比较多变体）、HunyuanVideo 1.5（GSB 标注，I2V 侧配图还需人工核验图文一致性）；
· 多维度结构化标注协议（最精细）：Seedance 1.0——「在指定维度选最优/最差，同时保证最优者在其他维度不劣于最差者」。这个约束条件直接解决了多维奖励下的矛盾配对问题，与 JavisDiT++ 的「归一化模态感知排序（normalized modality-aware ranking）」是同一问题的两种解法（后者明确目的是「保证每个模态内部的一致性，而不是把优质音频与劣质视频混搭配对」）；
· 半自动生产线：SkyReels-V2（人工 3 万样本对 + 自动受控失真样本）；
· SFT 集的人工终审：HunyuanVideo 100 万人工精选（七维 rubric）、SkyReels-V4 100 万人工精选、Movie Gen 人工挑影视感 + 人工重写 caption、Step-Video-T2V 逐条人工评审、Apollo Stage III 的 manually-curated 高质量集；
· 公开偏好集的专业标注：VideoReward 雇佣「professional annotators」对每个三元组分别在 VQ/MQ/TA 三维记录成对偏好（A 胜/平局/B 胜），并用带平局的 Bradley-Terry 模型（BTT）建模；
· 人工只用于评测不回流训练（本主题最普遍的浪费）：Movie Gen、CogVideoX、Open-Sora 2.0（100 prompt 人工胜率）、Ovi（50 人盲测）、Unison（40 样本 × 25 人 = 1000 次排序投票）、UniTalking（20 人 × 50 prompt）、InstructAV2AV（25 人 × 20 样本 × 3 维）、Foley-Omni（A/S/T-MOS）、HunyuanVideo-Foley（MOS-Q/S/T）、CineDance（每视频 10 名评测员 5 点量表）、UltraVideo（10 人对比）、LVD-2M（200 份响应）、Script-a-Video（20 名专业评分员）——这些工作都已组织了人力做主观评测，把同一批标注扩展为偏好对的边际成本极低，却无一这样做。这是开源/学术侧后训练的系统性缺口。

## 音视频对齐

### 音视频同步检测方法（口型同步、事件对齐） ⚠️

锚论文在后训练数据侧完全不涉及音视频同步检测 [不确定]——四个奖励模型（视频美学、图像美学、运动质量、文本-视频对齐）无一涉及同步性，PE 的三项奖励亦然。对 AV 模型的处理仅在 Phase 4 蒸馏阶段，遵循 OmniForcing 配备非对称块因果对齐与音频 sink token，属架构对齐而非数据侧同步检测。这意味着：即使用该框架对 AV 模型做完整四阶段后训练，唇同步与事件对齐质量也不会被任何奖励信号直接优化。
【横向后训练侧唯一的完整同步奖励实践】JavisDiT++ 的 AV-DPO：六个奖励模型中 Synchformer 专司时序同步性，ImageBind 专司跨模态语义相似度，二者在偏好排序中并列参与——这是把 AV 同步作为独立偏好维度的唯一公开案例。约 2.5 万条偏好对、3 万条 prompt 池（与 SFT 数据不重叠）、每 prompt 3 个生成 + 1 条真值共 4 候选、1 epoch、LoRA 121M 可训参数。
【其余】Seedance 1.5 pro 的 RLHF 称覆盖「音频保真度」，但是否含同步维度未披露 [不确定]；Kling 3.0 Omni 的 DPO 是否把口型同步作为独立打分项未披露 [不确定]。数据侧的 AV 同步过滤（SyncNet/Synchformer 阈值筛选）在各模型中一律发生在预训练数据 pipeline 而非后训练阶段。

### 同步检测具体指标与阈值（SyncNet/Synchformer/LSE/自研，阈值数值，如MOVA LSE-D≤9.5且LSE-C≥4.5、SkyReels-V4 SyncNet |offset|≤3∧conf>1.5） ⚠️

锚论文无任何同步指标与阈值 [不确定]。横向在后训练阶段唯一可点名的指标是 JavisDiT++ 用 Synchformer 作为时序同步奖励模型，但作为奖励模型使用时是连续分数参与归一化排序，不设阈值——这与数据过滤阶段的阈值范式（如 MOVA 的 LSE-D≤9.5 且 LSE-C≥4.5、SkyReels-V4 的 SyncNet |offset|≤3 且 conf>1.5、UniTalking 的 SyncNet conf>0.9）有本质区别：
· 数据过滤阶段：硬阈值二分（通过/剔除），目的是保证训练素材本身同步；
· 后训练奖励阶段：连续分数参与组内相对比较，目的是把模型输出推向更同步的方向。
这个「阈值→连续奖励」的转换是同步性从数据侧走向训练信号侧的关键形态变化，但目前只有 JavisDiT++ 一例完整实现。Unison 的评测体系含 LSE-C/LSE-D，UniVerse-1 的 Verse-Bench 含 LSE-C/AV-A，均仅用于评测未回流训练。[不确定]

### 时序同步vs语义同步的分离处理（时间对齐与内容语义匹配作为两个独立过滤条件） ⚠️

锚论文不涉及 [不确定]。JavisDiT++ 的六奖励设计事实上实现了这一分离：时序同步由 Synchformer 单独承担，语义匹配由 ImageBind 在文本-音频、文本-视频、音视频跨模态三条链路上分别承担——即时间对齐与内容语义匹配是两个独立的奖励项，而非合并为单一「AV 质量分」。其「归一化的模态感知排序」进一步保证配对时不会把优质音频与劣质视频混搭，等价于在偏好构造层面维持模态内一致性。这一设计与 Seedance 1.0 人工标注协议的约束条件（最优者在其他维度不劣于最差者）在目标上完全一致：防止多维奖励下产生自相矛盾的偏好信号。这是本主题在后训练数据构造上最具可迁移性的两条工程经验。

### 音频质量过滤（SNR、静音检测与静音占比阈值、无音轨剔除、画外音源剔除、背景音乐分离） ⚠️

锚论文不涉及 [不确定]。后训练阶段的音频质量控制主要以奖励模型形态出现而非阈值过滤：JavisDiT++ 用 AudioBox-Aesthetics 评估音频质量；Seedance 1.5 pro 的多维奖励含音频保真度（audio fidelity）。数据侧的例外是 Movie Gen 的音频 SFT 集——cinematic split 由影视感分类器 + AED 声音事件检测自动筛后人工选定，且明确排除含人声片段；Foley-Omni Stage 3 的准入条件是通过完整清洗 pipeline（六项过滤阈值 + Gemini 标注 + Bandit 能量验证）且含多个音频组件。SNR、静音占比、无音轨剔除、背景音乐分离等具体阈值在所有后训练材料中均未出现 [不确定]——这些属于预训练数据 pipeline 的职责。

### 语音/音效/音乐的分类与分别处理策略 ⚠️

锚论文不涉及 [不确定]。横向：Movie Gen 是唯一在后训练阶段对音频类型做显式分治的工作——cinematic split（含画内音与画外环境/主题音乐、排除人声）与 high quality audio split（纯音乐 O(10)K 小时 + 纯音效 O(10)K 小时）分开构建，规模差两个数量级，说明其把「配乐/音效学习」与「视频-音频对应关系学习」当作两个可分离的目标。Foley-Omni 的三阶段课程（音效→音乐→完整配乐）亦是类型分治的课程形态，但止于监督训练。偏好学习层面无任何工作对语音/音效/音乐分别设计奖励 [不确定]——这是 AV 后训练最明确的待补空白：现有的音频奖励模型（AudioBox-Aesthetics）是通用美学分，无法区分「对白清晰度不够」与「音效与画面事件错位」两类完全不同的失效。

## 训练配合

### 多阶段训练课程与数据课程调度（阶段划分依据：分辨率/时长/质量分/模态；低清→高清、图像→视频、短→长）

【锚论文的四阶段课程即其核心贡献】Phase 1 SFT → Phase 2 RLHF(GRPO) → Phase 3 PE → Phase 4 AD。四阶段之间的依赖关系被论证得比较清晰：SFT 为 RLHF 提供稳定参考策略并扩大探索空间；PE 与 RLHF 共用同一套奖励回路但作用于输入侧，二者互补；AD 把前三阶段的能力迁移到因果架构。AD 内部又细分三个子阶段：① DMD 蒸馏——先把原始预训练模型蒸成只需少量去噪步的双向学生模型，保留全局注意力感受野，为后续迁移到因果架构提供高质量、易回归的教师轨迹；② Causal ODE 回归——直接用 DMD loss 训因果学生会因架构差异而不稳定，故引入高效初始化策略并装配块因果掩码，训练模型仅凭因果历史做有效去噪预测；③ Self-Forcing 蒸馏——每帧基于此前自生成输出通过带 KV cache 的自回归 rollout 生成，从而在视频级施加 DMD loss，直接评估整段序列的质量。
【GRPO 的具体课程设计（锚论文的技术要点）】沿用 DanceGRPO 把流匹配采样表述为 MDP，稀疏奖励仅在终止步给出。针对视频生成，作者指出 MixGRPO 在随机子集较小时会出现 reward collapse，故受 Flash-GRPO 启发采用「等时分组（isotemporal grouping）」——每条 prompt 被指派一个不同的时间步 t_i，去噪过程中每个 prompt 组只在其指派时间步做一次 ODE→SDE 转换，该步用 SDE 采样以支持探索与梯度计算，其余所有时间步用确定性 ODE 更新以产出更高质量的生成与更可靠的奖励信号。并引入「时序梯度校正（Temporal Gradient Rectification）」显式归一化时间相关缩放因子 λ(t)=√Δt/σ_t + σ_t√Δt(1−t)/(2t)。遵循 DanceGRPO 省略 KL 正则项。
【横向课程对比】HunyuanVideo 1.5 的 CT→SFT→RLHF 三段且 T2V/I2V 全程分开；SkyReels-V2 的 540p 概念均衡 SFT→720p 高质量 SFT→三阶段 DPO；Cosmos-Predict 2.5 的五域并行 SFT→模型合并→GRPO→rCM 蒸馏；Motif-Video 2B 把 SFT 插入分辨率阶梯中间（720p 预训练从 480p SFT checkpoint 起步）；Seedance 1.0 的多子模型 SFT→model merging→RLHF；LongCat Avatar 1.5 把 GRPO 从视频级奖励扩展到逐帧奖励，多段 rollout 支持最多 5 个 clip 且仅末段参与优化，并用 DMD2 蒸馏至 8 步。

### 数据配比随训练阶段的变化（预训练/退火annealing/SFT高质量子集） ⚠️

【锚论文】各阶段数据配比全部未披露 [不确定]。可确认的结构性事实是：SFT 用「文本-视频配对数据」，RLHF/DMD/ODE 回归/self-forcing 四个环节共用一个「curated prompt set」（只需 prompt 不需配对视频）——这是后训练数据结构的关键分野：SFT 需要成对数据，而 RL 与蒸馏只需 prompt。这大幅降低了后三个阶段的数据获取成本，也解释了为何后训练阶段的 prompt 集构建（多样性、类目均衡、去重、与 SFT 集不重叠）比素材采集更受重视。
【横向的阶段配比数字】
· ALIVE 最完整：Continue-Training 4.3M balanced samples（3 epochs）→ SFT 5M（高美学与写实数据 3:1 混合，仅 0.5 epoch，非对称学习率 video 1e-5 / audio 1e-6）→ 1080p Refiner 0.7M（1 epoch）→ Character-driven 0.8M。SFT 刻意只训 0.5 epoch 以避免过拟合到高美学分布而损失多样性，是配比之外的重要 trick；
· SkyReels-V4：500 万（3 epoch）→ 100 万人工精选（3 epoch）；
· Cosmos-Predict 2.5：五域各自 30k iterations、batch 256，另 388K 条 4K 冷却数据；
· Movie Gen：SFT 后做 model averaging；Seedance 1.0：多子模型 SFT 后 model merging——权重平均是「数据配比」的一种替代实现；
· Foley-Omni：Stage 3 用 216 小时目标域数据 + 每域 100 小时回放防遗忘，是后训练阶段显式做经验回放的少数案例；
· HunyuanVideo 1.5：CT 每任务 100 万 → SFT 更严子集 → RLHF O(10K) prompt；超分模块独立用 100 万条 1K–4K 片段训练。
【规律】SFT 阶段普遍训得「浅」（低 epoch、小学习率、早停）——Seedance 1.0 用比预训练更小的学习率、有限 GPU 数并配合早停以防过拟合与保持文本可控性，ALIVE 只训 0.5 epoch，CogVideoX 只训 10k 步。这与预训练的「训到收敛」形成鲜明对比，本质是在「贴合高质量分布」与「保留预训练的多样性与可控性」之间求平衡。

### 后训练数据（SFT精选集规模与筛选标准、偏好对数量与标注方式、reward model训练数据）

本条目即以此字段为主题，此处给出可直接复用的结论性汇总。
【一、SFT 精选集的规模与筛选标准】规模量级 10^6–10^7 条、占预训练池 0.4%–20%（NAVA 1.07%、Allegro 0.4%、CogVideoX 20%、Goku 12.5%、Cosmos ~10%、SkyReels-V4 第二段 20%）。筛选标准的通用结构为四层叠加：① 预训练阈值的严格化版本（美学分、清晰度、运动分、文字面积、CLIP 相似度——Allegro 给出了唯一一套完整可复现的阈值组合）；② 概念/领域均衡（LongCat 的 caption 嵌入密度倒数采样、Movie Gen 的 k-NN 概念平衡、Cosmos 的分类器五域拆分、Seedance 的数百类目定向采集、Motif 的 style/subject 标签驱动 domain-balancing）；③ caption 重打标升级（NAVA 用 Gemini-3-Pro 重打、Open-Sora 2.0 用 Qwen2.5-Max 重打、Movie Gen 人工重写、Step-Video 人工优化）；④ 人工终审（HunyuanVideo 的美学四项 + 运动三项七维 rubric、Step-Video 的清晰度/美学/运动恰当性/转场平滑四项）。训练配置上普遍「浅训」以防过拟合。
【二、偏好对的数量与标注方式】规模量级 10^4–10^5 对，比 SFT 集小 2–3 个数量级。四种构造方式：
① 人工标注真实偏好（最贵、最可靠）：Step-Video-T2V（多种子生成 + 标注员评分 + 质控监督一致性）、Kling 3.0 Omni（同 MVL 多变体人类比较）、HunyuanVideo 1.5（GSB 标注 + 非重复配对）、SkyReels-V2（3 万人工样本对）；
② 自动构造负样本：SkyReels-V2 的受控失真损坏样本（覆盖 V2V/I2V/T2V）；
③ 多奖励模型自动排序：JavisDiT++（六 RM + 归一化模态感知排序，2.5 万对）；
④ 在线奖励取代离线偏好对（当前主流）：GRPO 路线的锚论文、LongCat-Video（group size 4、每步 64 prompts、约 0.5k iterations）、Cosmos-Predict 2.5（8 rollout × 20 步、组内归一化、256 steps、batch 32）——不存在预标注偏好对数据集，偏好由 RM 在 rollout 时在线给出。
标注协议的两条关键工程经验：Seedance 1.0 的「指定维度选最优/最差且最优者在其他维度不劣于最差者」；JavisDiT++ 的「归一化模态感知排序」以避免优质音频与劣质视频混搭配对。评测协议上 GSB（Good–Same–Bad）已成为工业报告的事实标准（锚论文、Kling、Seedance、HunyuanVideo 1.5 均用），其优点是允许标注者表达无差别以减少边缘case噪声。
【三、Reward Model 训练数据】公开资产两个：HPDv3（108 万文本-图像对、117 万成对比较）与 VideoReward 偏好集（1.6 万 prompt、12 个 T2V 模型生成的 10.8 万视频、18.2 万三元组，专业标注员在 VQ/MQ/TA 三维分别记录 A 胜/平局/B 胜，采用带平局的 Bradley-Terry 模型 BTT，另留 1.3 万 prompt 不重叠的三元组作验证集）。锚论文的 RM 训练范式：Qwen3.5 VLM 主干 + MLP 头、不确定性感知排序损失、两阶段训练（Stage 1 数据感知正交梯度投影融合 HPDv3++ 的多样美学偏好同时保留 HPSv3 原有知识；Stage 2 利用不同能力等级模型与不同 RL 迭代产生的无标注数据）。多奖励融合被论文明确指认为系统的关键难点，需精心设计聚合策略与权重系数。
【四、行业分层结论】闭源工业模型（Seedance、Kling、HunyuanVideo 1.5、Step-Video、SkyReels、LongCat）已普遍完成 SFT + 偏好对齐两段式后训练；开源与学术侧（Movie Gen、CogVideoX、Allegro、Goku、Motif、MAGI-1、Open-Sora、以及几乎全部 AV 学术工作）绝大多数止步于 SFT，偏好对齐是最大缺口，其中 JAVG 音视频联合生成领域除 JavisDiT++ 外为完全空白。

### 数据处理基础设施与吞吐（NeMo Curator/Data-Juicer/自研，GPU加速比，处理规模，成本） ⚠️

锚论文未披露任何基础设施与吞吐数据（无 GPU 数、无训练时长、无成本）[不确定]，但正文两次强调后训练的算力约束是核心矛盾：「post-training must operate under strict constraints on sampling cost, evaluation quality, and system efficiency」「rollout generation is expensive」，并引 DanceGRPO 为据。其对策有三：① 等时分组——每个 prompt 只在一个指派时间步做 SDE 采样与梯度计算，其余步用 ODE，把 GRPO 的梯度计算量压到接近单步；② 冻结生成主干只训 PE 的 LLM 策略，使 PE 可套用任意现成生成器而无需重训视频模型；③ 自回归蒸馏把双向模型压成因果少步模型以降低部署推理成本。
【横向的后训练算力数字】Movie Gen 的视频 SFT 只用 512 张 H100（相对其预训练规模属极小）；Allegro SFT 消耗 2.6M 样本、batch 256、10K 步、256× H100；Cosmos-Predict 2.5 的 GRPO 因显存限制把轨迹概率拆解为逐步条件概率之和，每两步计算一次梯度并沿整条 10 步轨迹累积后做一次参数更新，训练 256 steps、batch 32；Seedance 1.5 pro 称针对 RLHF 流水线的基础设施优化带来近 3 倍训练加速；BranchGRPO 通过把 rollout 组织成分支树、共享前缀降低开销并剪除低奖励路径；MixGRPO 用混合 ODE-SDE 提升训练效率。数据处理工具侧：NeMo Curator 与 Data-Juicer 均不提供视频生成的偏好对构造能力（Data-Juicer 2.0 声明支持基础模型后训练但视频侧仅覆盖监督式蒸馏路线），后训练数据构建目前完全依赖各团队自研 [不确定]。

## 效果对比

### 数据策略消融的量化影响（区分：过滤严格度ablation / caption密度风格ablation / 数据配比ablation，及对应评测指标） ⚠️

【锚论文的量化结果（仅两个数字，无逐组件消融）】在其内部视频生成模型上，RLHF 阶段在整体 GSB 指标上取得 31% 的提升，增益「最显著体现在视觉质量与运动质量两个维度，二者均有巨大增强」，而文本对齐的提升相对温和——作者归因于当前文本对齐奖励模型准确率有限，限制了语义正确性的优化空间。在此之上叠加 prompt enhancer 再带来 20% 的整体 GSB 提升，同样由视觉与运动质量驱动，且保持文本对齐不退化。此外提供了 Wan-2.1 上的 RLHF 效果可视化（图 2）。
【方法学缺陷（必须标注）】论文未给出：基线模型名称、GSB 的分母定义（31% 是 Good 率还是 (G−B)/N）、标注样本量、标注员人数、置信区间、逐维度分数、与其他后训练方案的对比、SFT 单独的增益、AD 蒸馏前后的质量-速度权衡曲线。无任何数据侧消融（未做过滤严格度 ablation、caption 密度 ablation、数据配比 ablation），也未在 VBench 等公开基准上报告成绩。因此其「31%/20%」不具备跨工作可比性 [不确定]。
【横向的可比消融证据】
· Cosmos-Predict 2.5 提供了本主题最扎实的逐域 SFT 消融（人工胜率 SFT vs 预训练基线）：机器人操作 72.6% vs 8.3%、物体恒存 50.9% vs 27.7%、高运动 44.0% vs 34.7%、复杂场景 42.6% vs 35.4%、驾驶 47.9% vs 28.8%——直接量化了「按域精选 SFT 数据」的收益，且收益与域内数据量不成正比（730K 的机器人操作域收益最大，10.4M 的物体恒存域收益中等），说明域与基座能力的差距比数据量更决定 SFT 增益；
· Cosmos 的 rCM 蒸馏消融：4 步生成的 PAI-Bench 分数与教师几乎持平（T2W 0.764 vs 0.768，I2W 0.816 vs 0.810，蒸馏版 I2W 甚至略高）；
· AVSCap 的关键论断：RL 增益 > 扩 SFT 数据量——是「后训练投入优先于标注量扩张」的直接依据；
· MOVA 的 prompt rewriter 消融：人类 Arena ELO 从 982.9 提升到 1025.3，独立量化了 PE 的价值（与锚论文 PE 带来 20% GSB 的定性结论方向一致）；
· StreamChar 消融：单阶段蒸馏效果劣于两阶段（与锚论文 AD 分三子阶段的设计相互印证）；
· video-SALMONN 2 的 MrDPO：7B 模型 caption 错误率相对基线降 28%；
· LongCat-Video 的 HPSv3-percentile 设计（取分数最高前 30% 帧）与 Motion RM 输入灰度视频的设计，均是针对奖励信号污染的定向消融产物，但论文未给对照数字 [不确定]。

### 质量vs数量的证据（小而精数据超越大而杂的案例） ⚠️

本专题是「质量优于数量」命题在视频生成领域最集中的证据带：
【最强的量化证据】
· NAVA：从 15M 训练语料中经多算子协同过滤取 160K（1.07%）做 SFT，且该子集的 caption 由更贵的 Gemini-3-Pro 重新生成——「千分之一精选 + 标注升级」是本次调研中最激进的质量优先实践；
· Allegro：500M → 2M（0.4%）；
· Movie Gen：视频 SFT 训练只用 512 张 H100，相对其预训练规模是极小投入，却承担了美学与影视感的最终定型；
· Cosmos-Predict 2.5：730K 条机器人操作数据的 SFT 带来 72.6% vs 8.3% 的人工胜率碾压，是「小数据集在目标域上超越大而杂预训练」的直接实证；
· AVSCap 的「RL 增益 > 扩 SFT 数据量」论断，把命题从「小而精数据」推进到「优化范式 > 数据规模」；
· ALIVE 的 SFT 只训 0.5 epoch，是「精选数据也不宜训透」的反向注脚——质量优先并不等于让模型完全拟合精选分布。
【锚论文的立场】其核心论断「SFT as the foundation for RLHF」实际上是质量-数量命题的一个更进阶版本：SFT 的价值不在于直接提升主观质量（论文明确说「SFT is not intended to fully solve alignment or optimize subjective quality」），而在于提供一个稳定的参考策略并「扩大 RLHF 的探索空间」。也就是说，SFT 精选数据的最终价值要通过 RLHF 才能兑现——这把「小而精数据」从终点重新定位为中间件。这是本专题最值得记录的观点转变，尽管论文没有为它提供任何消融证据 [不确定]。
【反向证据/警示】Apollo 明确采用人工精选高质量数据做 Stage III 质量精修但完全不做偏好对齐，其对齐能力全部来自数据质量与架构；Step-Video-T2V 则指出 DPO 的收益会在模型能轻易区分正负样本时饱和——两者共同说明「数据质量」与「偏好优化」的收益曲线形状不同，前者边际递减更早，后者需要持续更新的奖励信号才能维持梯度。

### 训练数据domain分布与评测基准类目体系的对齐关系（如VABench七大类） ⚠️

【锚论文】其评测类目体系为三维：视觉质量（整体外观、锐利度、无伪影）、运动质量（时序连贯性、平滑性、运动模式合理性）、文本对齐（生成视频与输入 prompt 语义的一致性）——与其四个奖励模型中的三个（视频美学/图像美学合并为视觉质量、运动质量、文本-视频对齐）严格一一对应。这种「奖励维度 = 评测维度」的设计是后训练工作的通行做法，其优点是训练目标与评测口径完全一致，缺点是天然存在自证嫌疑：用 RM 优化的模型再用同维度的人评来验证，无法排除 reward hacking 只是没被这套维度捕捉到。论文未在 VBench、EvalCrafter 等外部基准上报告成绩（尽管引用了它们并批评其「evaluation signals are often noisy」）[不确定]。
【三维体系的行业收敛】VQ/MQ/TA 三维已成为视频生成后训练的事实标准类目：VideoReward（VQ/MQ/TA 三维分别标注）、LongCat-Video（VQ/MQ/TA 三奖励模型）、锚论文（三维 GSB）、HunyuanVideo 1.5（美学吸引力/清晰度/运动流畅性）、Seedance 1.0（Foundational/Motion/Aesthetic 三 RM）——这套三维体系正是训练数据 SFT 筛选标准（美学 + 运动 + 图文一致）的镜像，即筛选维度、奖励维度、评测维度三者已高度对齐。
【AV 维度的缺位】VABench 七大类等音视频评测体系与后训练奖励维度尚未打通：三维体系全为视觉维度，音频保真、唇同步、事件对齐、音色一致性均不在其中。JavisDiT++ 的六奖励（音频质量/文本-音频/视频质量/文本-视频/跨模态相似/时序同步）是唯一把评测类目扩展到 AV 全维的后训练体系，可视为 AV 时代三维体系的候选继任者。Movie Gen 音频侧的 MOS-Q/S/T、Unison 的 11 项客观指标（VA/ID/PQ/CU/WER/TA/TV/AV/LSE-C/LSE-D/DS）、UniVerse-1 的 Verse-Bench（MS/AS/ID/FD/KL/CLAP/LSE-C/AV-A/WER）等 AV 评测类目均未被任何后训练工作用作奖励维度 [不确定]。

## 其他信息

### summary_note

【核心结论】① 锚论文 arXiv:2604.25427 是 HKU + 京东探索研究院等的四阶段后训练蓝图（SFT→GRPO RLHF→Prompt Enhancement→Autoregressive Distillation），其技术贡献集中在 GRPO 侧（等时分组 isotemporal grouping 缓解 MixGRPO 的 reward collapse、Temporal Gradient Rectification、省略 KL 项、四奖励融合）与蒸馏侧（DMD→Causal ODE 回归→Self-Forcing 三子阶段），但在数据维度上几近零披露——SFT 数据描述仅一句「we constructed a high-quality text-video dataset for SFT」，无规模、无筛选标准、无 prompt 集条数；且 3.1 节 SFT 的失败模式列举（refusal cascades、incoherent reasoning）明显是从 LLM 后训练文本迁移，不含视频特有内容。因此它作为「后训练数据策略」的信息源价值有限，其真正价值在于提供了四阶段框架的组织逻辑与「SFT 是 RLHF 的地基而非终点」这一定位论断。
② 真正可复用的数据方法论来自横向对象：SFT 精选集的规模区间（占预训练池 0.4%–20%，绝对量 10^6–10^7）、四层筛选结构（严格化阈值 + 概念均衡 + caption 重打标 + 人工终审）、Allegro 的完整阈值组合、HunyuanVideo 的七维人工 rubric、LongCat 的密度倒数采样、Cosmos 的分类器分域与逐域胜率消融、Movie Gen 的四阶段与音频双 split、Seedance 的数百类目定向采集 + 子模型融合。
③ 偏好数据侧的关键判断：行业正从「离线偏好对 + DPO」快速转向「在线奖励 + GRPO」，后者不需要预标注偏好对数据集（锚论文、LongCat、Cosmos 均已如此），因此「偏好对数量」这一指标本身正在失去意义，取而代之的竞争维度是奖励模型的质量与多奖励融合策略。仍需人工标注的部分从「标注偏好对」转移到「标注奖励模型的训练数据」。两条最具可迁移性的标注工程经验是 Seedance 1.0 的多维一致性约束与 JavisDiT++ 的归一化模态感知排序，二者解决的是同一个问题：多维奖励下如何避免自相矛盾的偏好信号。
④ 最大缺口：音视频维度。锚论文的四个奖励模型全为视觉维度，AV 只体现在蒸馏架构（OmniForcing 的音频 sink token）；全行业除 JavisDiT++ 的 AV-DPO（2.5 万对、Synchformer 管同步、AudioBox 管音质、ImageBind 管语义）外，没有任何工作把唇同步、音效事件对齐、音色一致性纳入偏好奖励。同时，大量学术工作已组织了人工主观评测（Unison 1000 次排序投票、Ovi 50 人盲测、InstructAV2AV 25 人×20 样本×3 维等）却无一将其回流为训练信号，是边际成本极低却普遍未做的动作。
⑤ 建议的复用路径：SFT 阶段参照 Allegro 阈值 + HunyuanVideo 七维 rubric + LongCat 密度倒数采样；奖励模型直接复用开源的 VideoAlign/VideoReward 与 HPSv3（并按 LongCat 的做法在内部数据上微调、按其灰度输入技巧解耦运动与美学）；RL 阶段用 GRPO 在线奖励避免偏好对标注成本，并按 Cosmos 的做法用 diffusion loss 正则抑制 reward hacking；AV 模型需自行补齐 Synchformer/AudioBox 类同步与音质奖励。

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
- safety_filtering
- caption_model
- caption_structure
- joint_av_caption_schema
- dialogue_transcription_attributes
- geometric_structured_annotation
- human_in_loop
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
