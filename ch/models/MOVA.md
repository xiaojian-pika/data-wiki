# MOVA（MOSS Video and Audio）

> 主题: 视频生成模型（含音视频同时生成）的数据处理：数据清洗流程、数据分布、打标方式

[← 返回首页](../index.md)

**目录**: [基本信息](#基本信息) · [数据规模与分布](#数据规模与分布) · [清洗流程](#清洗流程) · [打标方式](#打标方式) · [音视频对齐](#音视频对齐) · [训练配合](#训练配合) · [效果对比](#效果对比)

## 基本信息

### 名称

MOVA（MOSS Video and Audio）

### 发布机构/公司

SII-OpenMOSS 团队。论文署名机构包括：上海创智学院（Shanghai Innovation Institute）、MOSI Intelligence（无问芯穹/MOSI 智能）、复旦大学、上海交通大学、华东师范大学、同济大学、东南大学、厦门大学、电子科技大学。项目负责人为 Qinyuan Cheng（程琴媛）与 Tianyi Liang；通讯作者为陈谐（上海交大，chenxie95@sjtu.edu.cn）与邱锡鹏（复旦大学，xpqiu@fudan.edu.cn）。属于复旦 OpenMOSS（MOSS 系列）开源体系的音视频生成分支。

### 发布时间（技术报告/论文/开源时间）

2026年1月29日首次开源发布（模型权重 + 代码，GitHub/HuggingFace 同步上线）；2026年2月发布 38 页技术报告 arXiv:2602.08794（v2 版本日期为 2026年2月10日，cs.CV）；2026年3月9日上线 API；2026年5月6日补充开源评测代码。

### 类型（模型/数据集/工具链/评测基准）

模型（音视频联合生成基础模型）。同时附带三项衍生产出：(1) 完整开源代码库（训练 pipeline、推理、LoRA 微调、prompt 增强工作流、评测代码）；(2) 自建的六类场景音视频联合生成评测基准（与 Verse-Bench 配合使用）；(3) Arena 式人类偏好评测协议。不是数据集发布（训练数据本身未开源）。

### 开源程度（权重/代码/数据/pipeline各自是否开源）

开源程度在同类音视频联合生成模型中属于最高一档，采用 Apache-2.0 许可证，允许无限制商用。
【权重】开源。发布 MOVA-360p 与 MOVA-720p 两个变体（HuggingFace: OpenMOSS-Team/MOVA-360p、OpenMOSS-Team/MOVA-720p，以及 mova collection）。
【代码】开源且覆盖全链路：训练 pipeline、高效推理、LoRA 微调脚本、prompt 增强（rewriter）工作流、评测代码。论文明确声明“we release all model weights along with training, inference, and fine-tuning code”。
【数据处理 pipeline】方法论层面在论文第 3 节与附录 A.3/A.4/A.5 中做了工业级细节披露——包含完整三阶段漏斗结构、逐指标过滤阈值表（Table 9）、逐级保留率表（Table 1）、语音窗口切分伪代码（Algorithm 1/2）、以及全部打标 prompt 原文与完整 caption 示例。这是本次调研中数据处理方法可复现性最高的样本之一。但代码库中并未单独发布数据清洗脚本，仅提供 mova/datasets/video_audio_dataset.py 数据集接口，用户需自备视频音频数据并按配置接入。
【数据】训练数据本身不开源。公开数据集部分（VGGSound、AutoReCap、ChronoMagic-Pro、ACAV-100M、OpenHumanVid、SpeakerVid-5M、OpenVid-1M）可自行获取，但其“过滤后 HQ 子集”的具体清单未发布；in-house 数据（中文剧集、动画、电影、YouTube 抓取等）未公开。

### 是否支持音视频同时生成，以及实现方式（原生联合/级联/MoE融合）

支持，且为原生联合生成（单次推理同时产出视频与音频，非级联）。实现方式为“非对称双塔 + 双向 cross-attention Bridge 融合”，同时视频塔本身是 MoE：
- 总参数 32B，推理时激活 18B。
- 视频塔：Wan2.2 I2V A14B（MoE 架构，含 high-noise 与 low-noise 两组 DiT 专家，按 timestep t 与阈值 δ 切换；训练时采用奇数步优化 high-noise DiT、偶数步优化 low-noise DiT 的交替优化策略以适配 FSDP 计算图一致性要求）。
- 音频塔：自训 1.3B 文本到音频 DiT，沿用 Wan2.1-1.3B 骨干，将 3D 位置编码 (f,h,w) 替换为沿时间轴的 1D 位置编码。
- 融合：轻量级 Bridge 模块，在两个 DiT 骨干的 hidden-state 层级插入两个 cross-attention 块（视频→音频、音频→视频），30 层交互。
- Aligned RoPE：因视频 latent 时间栅格粗、音频 latent 密集，将视频索引按 s = f_a/f_v 缩放映射到音频时间单位（p_v(i)=s·i，p_a(j)=j），使两模态处于同一时间尺度，避免跨模态注意力的时间错位漂移。
- VAE：视频用 Wan2.1 video VAE，音频用 HunyuanVideo-Foley 的 DAC 风格 audio VAE（48kHz 单声道），二者训练全程冻结。
- 训练目标：flow matching；Dual Sigma Shift 让视频、音频独立采样 timestep 与噪声调度。
- 推理：Dual Classifier-Free Guidance（文本 CFG + 跨模态 Bridge CFG 双分支）。
任务形态为 IT2VA（图像+文本→视频+音频），并涌现出 T2VA 能力（用纯白占位图替代参考图即可纯文本驱动）。

### 调研信息来源列表（论文/技术报告/官方文档/新闻的URL，标注每条来源的性质：官方一手/同团队旁证/第三方报道）

- 【官方一手】arXiv:2602.08794v2《MOVA: Towards Scalable and Synchronized Video–Audio Generation》技术报告（38页，2026年2月10日）：https://arxiv.org/abs/2602.08794 、PDF https://arxiv.org/pdf/2602.08794 —— 本条目绝大部分字段的唯一且直接的信息来源，特别是第3节 Data Engineering、第4.3节 Progressive Joint Training、附录 A.1/A.3/A.4/A.5/A.6。注意 arXiv 未提供 HTML 版（/html/2602.08794v2 返回 No HTML），需直接解析 PDF。
- 【官方一手】GitHub 代码库 OpenMOSS/MOVA：https://github.com/OpenMOSS/MOVA —— 开源内容清单、Apache-2.0 许可、发布时间线（2026-01-29 首发 / 2026-02-10 技术报告与推理工作流 / 2026-03-09 API / 2026-05-06 评测代码）、数据集接口 mova/datasets/video_audio_dataset.py。
- 【官方一手】HuggingFace 模型页：https://huggingface.co/OpenMOSS-Team/MOVA-720p 、https://huggingface.co/OpenMOSS-Team/MOVA-360p 、collection https://huggingface.co/collections/OpenMOSS-Team/mova ；论文页 https://huggingface.co/papers/2602.08794
- 【官方一手】项目博客：https://mosi.cn/models/mova
- 【第三方报道】ComfyUI-Wiki 新闻《OpenMOSS Releases MOVA - Open-Source Synchronized Video-Audio Generation》（2026-01-29）：https://comfyui-wiki.com/en/news/2026-01-29-openmoss-mova-video-audio-generation —— 用于交叉核对首发时间。
- 【第三方报道】AI Films Studio 博客《MOVA: Open Source Video-Audio Generation》：https://studio.aifilms.ai/blog/mova-open-source-video-generation —— 用于交叉核对 Apache-2.0 商用许可表述。

## 数据规模与分布

### 训练数据量级（视频条数/小时数/token数，预训练与SFT分开）

按训练阶段分列（论文以“小时数”与“clip 条数”双口径给出，未给出 token 数）：
【音频塔预训练】使用 WavCaps + VGGSound（通用音效）、JamendoMaxCaps（音乐）、in-house TTS 数据三大域，训练定长片段；具体小时数/条数未披露。
【联合训练 Phase 1（360p，多样数据）】约 61,500 小时视频-音频数据，1 个 epoch，耗时 15 天。
【联合训练 Phase 2（360p，质量过滤）】约 37,600 小时 / 16.8M clips（16.8M × 8.05s ≈ 37,560 小时，自洽），1 个 epoch，耗时 7 天。
【联合训练 Phase 3（720p，最高质量子集）】约 11,000 小时，1 个 epoch，耗时 20 天。
【总计】三阶段合计 42 天，1024 张 GPU（128 节点 × 8 卡），约 43,000 GPU-days。
注意：MOVA 无独立的 SFT / RLHF 后训练阶段，因此不存在“预训练 vs SFT”的规模拆分，取而代之的是三阶段渐进课程内部的数据规模递减（61.5k → 37.6k → 11k 小时），呈典型的“规模递减、质量递增”金字塔。

### 数据来源构成（自有/公开数据集/网络爬取/授权采购/合成数据）

两大来源：公开数据集的高质量子集 + 大量自有（in-house）数据。
【公开数据集（均使用其 filtered HQ subsets）】
- VGGSound（音视频事件对齐数据集）
- AutoReCap（大规模音频生成数据）
- ChronoMagic-Pro（延时/变形类视频）
- ACAV-100M（自动策划的大规模音视频表征学习数据集）
- OpenHumanVid（人物中心视频生成数据集）
- SpeakerVid-5M（音视频双人交互人体生成数据集，是 lip-sync 能力的核心来源）
- OpenVid-1M（文生视频高质量数据集）
【自有数据】论文称“a large amount of in-house data”。Phase 1 明确列出的数据来源为：SpeakerVid5M、中文剧集（Chinese drama）、卡通动画（cartoon）、电影（movies）、YouTube、OpenHumanVid。
【内容形态与主题】视频形态覆盖电影、vlog、动画；主题覆盖教育、体育、美妆、新闻、访谈、动画等。
【合成数据】未使用模型合成的视频-音频训练数据；唯一的“合成”成分是音频塔预训练中的 in-house TTS 数据，以及所有 caption 均由 MLLM/LLM 自动生成（属合成标注而非合成内容）。
【授权采购】论文未提及任何付费授权采购数据。

### 数据合规与溯源（授权数据占比、rights-cleared数据集、C2PA等） ⚠️

论文完全未讨论数据合规与溯源议题：未给出授权数据占比，未声明 rights-cleared 数据集，未提及 C2PA 或任何输出侧水印/溯源标记，未讨论版权风险、肖像权或数据来源的法律状态。可间接判断的是：训练语料中包含 YouTube 抓取内容、中文剧集与电影片段，这些属于版权敏感来源，而所引用的公开数据集（ACAV-100M、VGGSound 等）本身多为“YouTube 链接集合”形式，其版权状态由原始数据集的条款约束。模型权重侧采用 Apache-2.0 开放商用许可，但训练数据的合规性未作任何说明——这是 MOVA 在“方法论透明度极高”的同时“合规透明度为零”的明显不对称。[不确定]

### 片段时长分布与切分策略

采用严格的固定时长切分策略，不做变长分桶：
- 所有训练片段统一为 8.05 秒，精确对应 24fps 下的 193 帧（首帧 + 8 秒视频，即 1 + 8×24 = 193）。
- 三个训练阶段的帧数恒定为 193，不随分辨率变化。
- 切分不是等间隔滑窗，而是由 VAD（语音活动）+ PySceneDetect（场景切换）联合驱动的窗口生成算法（详见 shot_segmentation 与附录 A.3 的两段伪代码）。
- 语音片段的窗口起点会自适应前移/调整，以避免截断正在进行的语句、保证口语内容的连续性。
- 推理侧输出同样为 8 秒左右（720p、8s、24fps 的 clip 约产生 1.6×10^5 个 token，论文在 Limitations 中将序列长度列为主要瓶颈）。

### 分辨率/宽高比分布与分桶策略

【标准化流程】原始视频先用 FFmpeg cropdetect 滤镜检测黑边并保留核心画面，再将主体内容居中、缩放至 720p，并按需对称补黑边（pillarbox 或 letterbox），统一规整为 9:16 或 16:9 两种宽高比。帧率统一重采样为 24fps。因此数据侧只有两种宽高比，不存在多桶宽高比训练。
【分辨率课程】Phase 1 与 Phase 2 训练在 360×640（360p），Phase 3 上采样到 720×1280（720p）。对应发布 MOVA-360p 与 MOVA-720p 两个模型。
【工程影响】720p 阶段序列长度大幅增加，context parallelism 从 CP=8 提高到 CP=16，有效 batch size 从 128 降到 64，checkpoint 间隔从 5000 步缩短到 2000 步。
【效果】消融显示分辨率从 360p 提升到 720p 后，DeSync 从 0.475 微降到 0.485、IB-Score 从 0.286 微降到 0.277（几乎无退化），而 LSE-C 反而从 6.278 提升到 6.593、cpCER 从 0.177 降到 0.149，验证了先在低清建立跨模态对齐、再升清的课程有效性。

### 类别/domain分布与配比策略（人物、动作、场景、风格等比例控制与概念均衡） ⚠️

论文对训练数据的 domain 覆盖只给出定性列举，未给出任何比例数字，也未描述概念均衡（concept balancing）机制：
- 视频形态：电影（movies）、vlog、动画（animations）、中文剧集（Chinese drama）、卡通（cartoon）、YouTube 内容。
- 主题域：教育（education）、体育（sports）、美妆（beauty）、新闻（news）、访谈（interviews）等，论文称其提供了“泛化到复杂真实场景所需的分布多样性”。
- 唯一可确认的强配比倾向是“以人说话为中心”：Phase 1 数据源中 SpeakerVid-5M 与 OpenHumanVid 均为人物/说话人中心数据集，且最终训练只保留语音片段（见 audio_category_distribution），说明人物-对白类数据在配比中占绝对主导，这与模型主打多语种唇同步的定位一致。
- 论文中唯一带百分比的类目饼图（Figure 6a，含 “others 2.3%”）描述的是自建评测基准的样本类别分布，而非训练数据分布，不可混用。
训练数据的 domain 配比数字属信息空白。[不确定]

### 音频类别分布与配比（语音/音效foley/音乐/环境音/静音的比例及控制策略）——AV模型独有维度

这是 MOVA 数据设计中最具决断性的一环——最终训练集**只保留语音片段**，是一次极端的音频类别配比选择：
【预处理阶段分类】用 Silero VAD 将音轨切为 speech / non-speech 区间，结合 PySceneDetect 的场景切点，生成四类 8.05 秒定长片段：单场景语音、单场景非语音、多场景语音、多场景非语音。
【关键配比决策】“Ultimately, only speech segments are selected for training, accounting for 69.47% of all preprocessed segments.” 即语音片段占全部预处理片段的 69.47%，且最终只用这部分训练联合模型。对应到总时长保留率（Table 1）：原始 100% → Stage 1（语音+非语音）84.57% → Stage 1（仅语音）58.75%。也就是说，仅“只要语音”这一条就砍掉了约 26 个百分点的数据。
【音频类型分类器】使用 EAT 自监督音频 Transformer 分类模型对音频打标，构建 speech / non-speech 子集，按目标能力分流（唇同步 vs 通用 foley/环境音建模）。语音子集的构造条件为：EAT-contained-Speech 与 EAT-contained-Singing 两个标签均判为 True（或满足模型正类置信度）。
【音频塔预训练的类别配比】与联合训练不同，1.3B 音频塔预训练刻意覆盖三大类：通用音效（WavCaps + VGGSound）、音乐（JamendoMaxCaps）、语音（in-house TTS）。即“音效/音乐能力在音频塔预训练阶段注入，语音-唇同步能力在联合训练阶段强化”的两段式分工。
【代价】论文 Limitations 明确承认：由于音频塔仅 1.3B 且联合训练偏语音，模型在歌声、复杂音色纹理、音乐/器乐内容上表现退化。
【未披露】非语音片段内部（音效 / 音乐 / 环境音 / 静音）的细分比例未给出。

### 叙事结构分布（单镜头vs多镜头、平均clip时长、镜头数分布、是否含原生音轨）

MOVA 显式地把“单镜头 vs 多镜头”做成了一个正交的数据维度，这在同类工作中较为少见：
- 通过 VAD 时间信息与 PySceneDetect 场景切点的交叉组合，生成四类片段：单场景语音、单场景非语音、多场景语音、多场景非语音（即 {单/多镜头} × {语音/非语音} 的 2×2 划分）。
- 多镜头片段的判定条件是硬性的：窗口的时间跨度内必须至少包含一个场景切点，否则丢弃（附录 A.3 Algorithm 1 第 11 行）。
- 单镜头片段的判定条件同样是硬性的：窗口必须完整落在两个相邻场景切点之间（Algorithm 2 第 5、19 行）。
- 平均 clip 时长固定为 8.05 秒（193 帧 @24fps），无长度分布可言。
- 是否含原生音轨：训练数据必须自带原生同步音轨——预处理第一步即剔除“解码失败”或“缺少有效音频通道”的样本，因此不存在无音轨样本。
- 打标环节也针对多镜头做了专门要求：视频 caption 的 prompt 明确指示 MiMo-VL“focusing on video scene transitions”，即显式标注镜头转场。
- 未披露单镜头与多镜头片段的具体比例。

### 语言/口音分布（多语种唇同步能力的数据基础） ⚠️

论文将 MOVA 定位为“multilingual speech with high-quality lip synchronization”，但对语种/口音分布只给出定性证据，无任何比例数字：
【可确认的语种覆盖】中文与英文是明确演示并评测的两种语言。Figure 1 展示了英文多说话人与中文多说话人两组精确唇同步案例（中文示例含成人-儿童双说话人对话），并展示了中文屏幕文字生成。主观 Arena 评测集刻意构造为双语混合——将原本纯英文的 Verse-Bench 语音数据人工翻译了一半，形成中英双语混合的 732 条评测集（600 条来自 Verse-Bench + 132 条来自自建基准）。
【数据侧的语种基础】中文能力主要来自 in-house 的中文剧集（Chinese drama）语料；英文能力主要来自 SpeakerVid-5M、OpenHumanVid 及 YouTube 内容。
【口音标注】ASR 转写 prompt 明确要求“LAW OF LANGUAGE FIDELITY：Preserve the original language. No translation.”（保留原始语言、禁止翻译），因此 caption 中天然保留了多语种原文；且音频 caption 中会描述说话人的口音（论文给出的完整 caption 示例中就包含 “with a General American accent”，即通用美式口音），说明口音是被 Qwen3-Omni-Captioner 自然语言化描述的属性之一，但并非结构化枚举字段。
【空白】支持语种清单、各语种小时数占比、口音类别分布均未披露；论文亦在 Discussion 中提到不同语言的 phoneme-viseme 映射差异是难点，但未量化。[不确定]

## 清洗流程

### 清洗漏斗整体结构（几级过滤、各级顺序）

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

### 漏斗定量保留率（每级过滤的输入/输出量与最终保留率，如Apollo 27%）

论文以 Table 1《Retention Ratio of Total Dataset Duration》给出了按总时长计的逐级保留率（相对原始视频），这是本次调研中少见的公开定量漏斗：
- Raw（原始视频）：100%
- Stage 1（预处理后，语音 + 非语音片段）：84.57%
- Stage 1（仅保留语音片段）：58.75%
- Stage 2（质量与对齐过滤后）：26.39%
即最终整体保留率 26.39%，与 Apollo 的 27% 处于同一量级。拆解看，两次主要损耗是：(1) “只要语音”这一条从 84.57% → 58.75%，砍掉 25.8 个百分点，是最大的单一淘汰源（另有一处片段数口径的表述：语音片段占全部预处理片段的 69.47%）；(2) 三维质量与对齐过滤从 58.75% → 26.39%，再砍掉 32.4 个百分点，相对留存率约 44.9%。
此外训练阶段还有更严的二次收窄（相对 Phase 1 的 61,500 小时）：Phase 2 降至约 37,600 小时 / 16.8M clips（约 61%），Phase 3 再降至约 11,000 小时（约为 Phase 1 的 18%）。若把训练课程也算作漏斗的延伸，从原始素材到最终 720p 高质量子集的等效保留率远低于 26.39%。
Phase 2 三道子过滤各自的产出量也有公开数字：OCR 无字幕子集 ~9.5M clips、LSE 唇音高质量子集 ~2.5M clips、DOVER 技术分 >0.15 子集 ~2.4M clips，合并后的 Phase 2 数据集为 16.8M clips。

### 镜头切分方法（PySceneDetect/自研模型/shot-aware splitting）

使用 PySceneDetect 做场景切点检测，但 MOVA 的关键创新在于把场景切点与语音边界联合起来做窗口采样，而不是简单按场景切分：
【检测】PySceneDetect 检测并记录全片所有场景变化点时间戳；Silero VAD 并行标注语音区间。
【多镜头窗口生成（附录 A.3 Algorithm 1）】遍历 VAD 语音段：窗口起点上界设为当前语音段起点；起点下界取三者最大值——上一个语音段的结束时刻、当前语音段之前最近的场景切点、当前语音段起点减去半个窗长（8.05/2）；在该区间内均匀随机采样窗口起点；窗口长度固定 8.05 秒；只有当窗口时间跨度内至少包含一个场景切点时才保留为多镜头样本；随后跳到第一个起点晚于窗口结束时刻的语音段，避免窗口重叠。
【单镜头窗口生成（附录 A.3 Algorithm 2）】遍历相邻场景切点构成的场景区间，在每个场景内寻找起点满足“起点 + 8.05 秒仍完整落在该场景内”的语音段，起点下界取（上一语音段结束、场景起点、当前语音段起点减半窗长）三者最大值，随机采样起点；若窗口末端越过场景结束点则中止；同样跳过重叠窗口。
【设计意图】(1) 通过下界约束“不早于上一段语音结束”避免窗口切进上一句话；(2) 通过下界约束“不早于最近场景切点”鼓励自然转场；(3) 通过“减半窗长”约束避免窗口起点离目标语音过远导致前半段空白；(4) 随机采样起点带来数据增广与位置多样性。整体是一套 shot-aware + speech-aware 的双重感知切分，专为唇同步任务定制。

### 质量过滤（美学评分、清晰度、OCR文字过滤、黑边/水印/logo检测）

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

### 运动过滤（光流/运动分数阈值、静态与抖动剔除） ⚠️

论文未描述任何独立的运动过滤环节：没有光流计算、没有运动分数阈值、没有静态镜头剔除或手持抖动剔除。运动相关的质量控制隐含在两处：(1) DOVER 技术分与美学分本身包含对时域失真、卡顿、模糊等的感知评估；(2) 视觉打标 prompt 中的“LAW OF VISUAL DYNAMICS”要求标注器检测所有转场、精确记录运动轨迹、速度变化与视觉节奏，并规定“当无视觉变化时 visual_description 输出 null”——理论上可据此识别静态片段，但论文未说明是否用该 null 信号做过滤。由于 MOVA 的训练数据以说话人对白为主体，画面运动幅度天然偏小，运动过滤的必要性也相对较低。[不确定]

### 去重方法（精确去重与基于embedding的语义去重分别记录） ⚠️

论文完全未提及去重环节：既无哈希/指纹级精确去重，也无基于 embedding 的语义去重。考虑到数据来源同时包含 ACAV-100M、VGGSound、OpenVid-1M 等多个源自 YouTube 的公开数据集，以及 in-house 的 YouTube 抓取内容，跨数据集重复的风险客观存在，但论文对此未作任何说明。[不确定]

### VLM/LLM作为质检员（多模态大模型质量打分与错配剔除，2026年从浅层打分器转向大模型语义判断的趋势）

MOVA 体现了 2026 年“大模型深度介入数据环节”的典型形态，但其分工是清晰切分的——**大模型负责语义标注与跨模态一致性裁决，专用小模型负责质量与对齐打分**：
【大模型作为语义裁决者（核心用法）】GPT-OSS-120B 在 caption 合并环节承担的不只是文本拼接，而是显式的跨模态一致性检查：“the model verifies the alignment between visual scenes and audio events to resolve potential conflicts”（校验视觉场景与音频事件的一致性，消解潜在冲突），再综合为统一自然语言描述。这实质上是用 120B 级 LLM 做“视听内容语义是否自洽”的判官，是本 pipeline 中最接近“LLM-as-judge”的组件。
【反幻觉的自审计设计】视觉与语音标注 prompt 都内置了 final_verification_audit 自检字段（hallucination_check_passed、visual_changes_verified / speech_dynamics_verified 及 comment），要求 MLLM 输出结构化的自我审计结论；并用强约束“LAW”体系抑制跨模态串扰——视觉标注器被明令“Ignore audio and inferred context entirely”“Do not infer or hallucinate based on audio or context”，语音转写器被明令“Ignore non-speech sounds and music entirely”。这是针对多模态标注器最典型失效模式（用一个模态臆测另一个模态）的防御性设计。
【质量过滤仍由专用打分器承担】音频用 Audiobox-aesthetics（PQ/CU/CE 三分数），视频用 DOVER（技术/美学双分数），对齐用 SynchFormer（时序）与 ImageBind（语义），音频分类用 EAT，唇音用 SyncNet 系的 LSE-D/LSE-C。这些均为专用判别模型而非通用 VLM。
【结论】MOVA 并未走向“用 VLM 打一个综合质量分替代所有浅层打分器”的路线，而是采取“专用打分器管质量与对齐 + 大模型管语义标注与一致性”的混合分工，可视为该趋势的一种务实折中。
【局限】论文在 Limitations 中承认，标注可靠性是多说话人场景的瓶颈：说话人分离（diarization）错误与不完善的 active-speaker 标签会传播到训练，导致模型混淆说话人或学到不一致的监督信号。

### 安全与合规过滤（NSFW、版权、人脸/隐私） ⚠️

论文完全未涉及安全与合规过滤：没有 NSFW 检测、没有版权过滤、没有人脸/隐私保护措施的任何描述，也没有模型卡级的安全声明或使用限制。这与其 Apache-2.0 完全开放商用的许可形成对比，是 MOVA 披露体系中的明显空白（与 Sora 2「安全披露详尽、数据披露空白」恰好相反：MOVA 是「数据方法披露详尽、安全披露空白」）。[不确定]

## 打标方式

### 使用的caption模型（自研VLM/开源模型，模型规模）

全部使用开源模型，形成“双模态分工标注 + 大模型融合”的三模型流水线，未自研 captioner：
【视觉标注】MiMo-VL-7B-RL（小米 LLM-Core 团队，7B 视觉语言模型，RL 版本），指令中明确要求聚焦视频场景转场。
【音频标注 — 双模型策略】
- Qwen3-Omni-Instruct：负责语音转写（ASR），处理语音成分。
- Qwen3-Omni-Captioner：负责非语音声音与音乐的描述，处理非语音成分。
  论文强调该双模型分工可“全面覆盖语言内容与声学特征，减少信息损失，捕获多方面音频语义”。
【融合与一致性校验】GPT-OSS-120B（OpenAI 开源 120B 模型），负责把视频 caption 与聚合后的音频标注（含语音与非语音）合并，同时做跨模态一致性检查。
【推理侧的 prompt 增强模型（非训练数据打标）】Qwen3-VL 提取结构化视觉描述，Gemini 2.5 Pro 通过 in-context learning 改写为符合训练数据分布的视频生成 prompt。
【算力】打标使用 NVIDIA GPU 与昇腾 Ascend NPU 混合执行。
【规模对比】视觉标注器仅 7B、音频标注器为 Qwen3-Omni 系列、融合器 120B——把最大的模型放在融合与裁决环节而非感知环节，是成本与效果权衡下的选择。

### caption密度与结构化程度（短/长/密集描述、结构化字段如镜头运动、风格标签）

caption 结构在中间态是严格结构化的 JSON，在最终态是长篇融合自然语言段落，属“结构化中间表示 → 自然语言最终 caption”的两段式设计，全部 prompt 原文与完整示例见附录 A.5：
【中间态：结构化 JSON】
- 视觉侧输出 video_visual_report，含两个字段：visual_description（视觉元素如何演变的详细描述；无视觉变化时输出 null）与 on_screen_text（所有可见文字的精确转写；无文字时输出 null）；外加 final_verification_audit 自审计块。
- 语音侧输出 speech_transcription_report，含 speech_description（逐字转写；无语音输出 null；不清晰处用 [inaudible] 标记）；外加自审计块。
- 非语音侧为自由文本音频描述（prompt 极简：“Please describe the audio you hear in detail.”）。
【三条标注法则（视觉）】LAW OF VISUAL TRUTH（只描述可视觉验证的元素，不得依据音频或上下文推断）、LAW OF VISUAL SILENCE（无变化输出 null）、LAW OF VISUAL DYNAMICS（检测所有转场，精确记录运动轨迹、速度变化与视觉节奏）。
【最终态：融合长 caption】GPT-OSS-120B 按五条规则合成单一流畅段落，结构上仍隐含顺序化的信息层次：先视觉叙事并锚定说话人上下文（按时间顺序描述动作、物体、场景、光照、运动、转场，不做概括）→ 再把对白作为视觉锚定的引语嵌入（用反映语气的言说动词如 snaps/murmurs/laughs，以及反映语速的表达如 rushes/drawls/pauses）→ 最后以“The audio includes…”或“In the background…”引出非语音音频（仅限环境音、音乐、音频主题/声源、结构性音频变化四类）。
【显式风格标签】未采用离散标签体系，风格/运镜信息以自然语言描述形式内嵌（推理侧的 prompt rewriter 才有显式四类结构：视觉风格含色调与光照、摄影含景别构图、视觉元素含主体与空间关系、OCR 文字）。
【长度】论文给出的完整示例约 200 词的密集长 caption；Figure 6b 给出了 prompt 长度分布统计图（针对评测基准），训练 caption 的长度分布未量化。
【字幕条件化】Phase 2 中无烧录字幕的视频会在 prompt 末尾追加 “This video has no subtitles.”，即 caption 中存在显式的可控属性标记。

### 音视频联合caption结构（是否同时覆盖视觉+听觉轨道、是否分流为独立字段，如LTX-2全音景描述、Script-a-Video factorized streams、Foley-Omni三字段）

MOVA 采用的是“先分流为三条独立轨道、再由 LLM 融合为单一统一 caption”的 factorized-then-merged 方案，属于 Script-a-Video 式 factorized streams 与 LTX-2 式全音景描述之间的折中：
【三条独立轨道（分流阶段）】
1) 视觉轨（MiMo-VL-7B-RL）：视觉描述 + 屏幕文字，严禁参考音频。
2) 语音轨（Qwen3-Omni-Instruct）：逐字对白转写，严禁包含非语音与音乐。
3) 非语音音频轨（Qwen3-Omni-Captioner）：音效、音乐、环境音、录音质量、说话人音色与口音等声学特征描述。
三条轨道通过互斥的 prompt 约束严格隔离，防止模态间幻觉串扰。
【融合阶段的角色再分配（关键设计）】GPT-OSS-120B 的合并规则明确规定了三条轨道各自的“权威范围”：
- 对白的**实际内容**严格以 speech_description 为准；
- 说话人**动态信息**（说话人总数、说话人切换、音色特征如高音/沙哑）以 audio_description 为准；
- 说话人的**视觉锚定**（谁在说、站在哪、什么装扮）以 video_description 为准。
  三者交叉引用，把“谁说了什么、听起来怎样、看起来是谁”三层信息绑定在一起。
- 非语音音频只允许出现在四个类目中：环境/背景音、音乐、音频主题/声源、结构性音频变化（如静默转向渐强），且严禁提及任何人声与词语。
- 规则 4 明确要求避免跨段重复（如不得在音频段重述说话人的视觉位置）；规则 5 要求跳过空字段、最终读起来像人写的连贯叙事而非刚性结构。
【最终产物】单一融合自然语言段落，不再保留独立字段。因此训练时模型看到的是统一 caption，而非分离的视觉/音频条件。
【与同类对比】相比 Foley-Omni 的三字段并列保留、LTX-2 的全音景单描述，MOVA 的特点是“标注时分流、训练时融合”，兼顾了标注准确性（分流抑幻觉）与训练时的条件简洁性（单一文本条件）。

### 对白转写与说话人属性标注（ASR转写+说话人身份/语言/口音/情绪）

对白转写与说话人属性标注是 MOVA 数据体系的核心：
【转写】Qwen3-Omni-Instruct 执行 ASR，输出逐字转写（verbatim）。三条法则约束：LAW OF LANGUAGE FIDELITY（保留原始语言、严禁翻译）；LAW OF SPEECH DYNAMICS（当新的说话人 / 语言 / 语调开始时，创建新的事件条目）；LAW OF SILENCE（无语音时 speech_description 输出 null）。不清晰片段标记为 [inaudible]。
【说话人属性】通过 Qwen3-Omni-Captioner 的音频描述以自然语言形式覆盖：说话人数量（如 “two distinct voices” / “a group of overlapping speakers”）、说话人切换点、音色特征（如 high-pitched / gruff / mature male）、口音（示例中为 “a General American accent”）、语气与语速（calm/authoritative/professional；measured/declarative；rhythmic evenly paced cadence）、录音环境（studio-clean、slight reverb）。这些属性不是枚举字段，而是描述性文本，在融合阶段被绑定到画面中的具体人物。
【说话人身份绑定】融合 prompt 要求把语音以引语形式嵌入并与视觉主体锚定（如“the teenager in the corner”“the gray-haired woman”），实现“视觉身份—音色—对白内容”的三元绑定。
【评测侧的对应设施】用 MOSS Transcribe Diarize（同团队 2026 年发布的转写+说话人分离模型，arXiv:2601.01554）对生成结果做说话人分离（显式说话人标签 [S01]、[S02]）与 ASR，进而计算 cpCER（concatenated minimum permutation CER），评估说话人身份与对白内容是否被正确反映。MOVA-720p 达到最佳 cpCER 0.149。
【已知缺陷】论文 Limitations 明确指出：diarization 错误与不完善的 active-speaker 标签会传播进训练数据，导致多说话人场景下的口型-音频错配与时序漂移；改进方向包括更强的主动说话人检测、跨模态说话人跟踪与更好的噪声片段过滤。

### 几何与结构化标注（相机参数、深度、3D point tracks、动作标注、显式状态） ⚠️

论文未使用任何几何或结构化标注：无相机参数、无深度图、无 3D point tracks、无骨架/动作标注、无显式物理状态标注。唯一接近结构化的标注是 caption 中的 on_screen_text 字段（屏幕文字精确转写）与推理侧 prompt rewriter 提取的四类结构化视觉描述（视觉风格/摄影/视觉元素/OCR 文字），但后者服务于推理时的 prompt 增强，不属于训练数据标注 schema。运镜信息仅以自然语言形式出现在 caption 中（如“The camera alternates between…”“camera panning, zooming, and rotation”），无参数化表示。[不确定]

### 合成数据构造（受控扰动/编辑构造训练对，如InstructAV2AV） ⚠️

论文未构造任何合成训练对：没有受控扰动、没有编辑构造的配对数据（如 InstructAV2AV 式的编辑对），也没有用生成模型自产视频回灌训练。全部训练数据为真实视频。仅有的“合成”成分是：(1) 音频塔预训练中使用的 in-house TTS 合成语音数据；(2) 全部 caption 由 MLLM/LLM 自动生成（合成标注而非合成内容）；(3) 评测基准的 prompt 由 GPT-5 统一改写整合，以及推理工作流中由 Gemini 2.5 Pro 生成的改写 prompt。[不确定]

### 人工介入程度（人工标注、人工质检、模型初筛+人工复核）

人工介入程度很低，标注环节完全自动化，人工只出现在“阈值标定”与“最终评测”两处：
【阈值标定（唯一的数据侧人工介入）】“we manually inspect the videos retained under different metric cutoffs and set reasonable thresholds for each dimension accordingly”——研究者人工抽检不同 metric cutoff 下留存的视频，据此为音频质量、视频质量、音视频对齐三个维度分别设定阈值。附录亦重申这些阈值“determined by empirical observation”。这是典型的“人工定标 + 机器批量执行”模式。
【标注环节】零人工。视觉、语音、非语音三路标注与最终融合全部由 MiMo-VL / Qwen3-Omni ×2 / GPT-OSS-120B 完成，无人工复核或抽检的描述。
【评测环节的人工投入较重】Arena 式人类偏好评测收集了 5,000+ 有效投票，评测集 732 条（600 条 Verse-Bench + 132 条自建基准），其中一半原为纯英文的 Verse-Bench 语音数据由人工翻译为中文以构造双语混合集；评测者需在 prompt 遵循、视听同步、唇同步准确性、视频质量、音频语音保真度五个维度上做两两偏好判断；采用 ELO 评分（初始 1000，K=4，logistic scale 400，base 10，1000 次 bootstrap）。

## 音视频对齐

### 音视频同步检测方法（口型同步、事件对齐）

MOVA 把音视频对齐做成了数据过滤中的一等公民，且在不同阶段使用不同工具、针对不同粒度：
【Stage 2 通用对齐过滤（双工具）】
- 时序同步：SynchFormer 计算每条视频的音视频时序同步性（DeSync 偏移量）。
- 语义对齐：ImageBind 计算音视频的跨模态语义对齐分（IB-Score）。
【Phase 2 唇同步专项过滤】使用 SyncNet 系的 LSE-D（Lip Sync Error - Distance）与 LSE-C（Lip Sync Error - Confidence），筛选高质量唇音对应片段。
【音频类型分流】EAT 音频分类模型区分 speech / non-speech，为“唇同步”与“通用 foley/环境音建模”两类能力分别构建子集——即不同能力对应不同的同步判据。
【架构层面的同步保障（非数据侧但强相关）】Aligned RoPE 将视频与音频 latent 映射到同一时间栅格；Bridge 双向 cross-attention 提供逐层跨模态交互；Dual Sigma Shift 让两模态独立采样噪声水平。论文在 Discussion 7.1 中提出一个有洞察力的观点：预定义的 sigma 调度实际上充当了**隐式的同步方向先验**——对唇部特写镜头（目标区域占画面比例大），视觉 latent 信息量相对充分，过程趋向 Video→Audio；当说话人只占画面一小块时，视觉证据相对不确定，过程自然偏向 Audio→Video，由语音提供更可靠的时间锚点。
【论文的核心经验结论】“architectural mechanisms alone (e.g., Bridge modules for cross-modal attention) are insufficient to achieve high-quality lip synchronization—the model must also learn phoneme-to-viseme mappings from data, which requires larger capacity and more training examples.” 即架构机制不足以获得高质量唇同步，音素到口型的映射必须从数据中学习，需要更大容量与更多训练样本。这是 MOVA 对“数据决定音视频同步上限”这一命题最直接的表述。

### 同步检测具体指标与阈值（SyncNet/Synchformer/LSE/自研，阈值数值，如MOVA LSE-D≤9.5且LSE-C≥4.5、SkyReels-V4 SyncNet |offset|≤3∧conf>1.5）

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

### 时序同步vs语义同步的分离处理（时间对齐与内容语义匹配作为两个独立过滤条件）

明确分离，且是 MOVA 数据过滤中一个被显式论证的设计决策：
- 时序同步由 SynchFormer 负责（DeSync 偏移量 ≤ 0.5），刻画“声音与画面事件是否在同一时刻发生”。
- 语义同步由 ImageBind 负责（IB-Score ≥ 0.2），刻画“声音内容与画面内容是否语义相关”。
- 二者作为两个独立条件，但组合方式是**宽松的逻辑 OR 门**而非 AND：“we apply a relaxed logical ‘OR’ gate between semantic and temporal alignment. A video is retained if it satisfies either IB-Score ≥ 0.2 OR DeSync ≤ 0.5.” 论文给出的理由是两类音频的对齐性质根本不同——语义相关的环境音/氛围声没有清晰的时序 onset，硬要求时序同步会把它们全部误杀；而快速动作音效可能在语义嵌入空间中得分不高，硬要求语义分会把它们误杀。用 OR 门可同时保留两类有价值的样本。
- 该分离在评测侧同样保持：Table 4 将 AV-Align 拆为 DeSync（时序）与 IB-Score（语义）两列分别报告，并单列 Lip Sync（LSE-D/LSE-C）作为最细粒度的连续时序对应。
- 论文在 6.3 节进一步区分了两类同步任务的难度层级：离散、onset 驱动的事件（如“切水果”“击鼓”）只需对齐少数显著时间点；而语音需要口型与音素在长时间跨度上的连续、细粒度对应，是最苛刻的一类。

### 音频质量过滤（SNR、静音检测与静音占比阈值、无音轨剔除、画外音源剔除、背景音乐分离）

覆盖较全，阈值公开：
【无音轨剔除】预处理第一步即剔除“缺少有效音频通道”（missing valid audio channels）的样本，以及解码失败样本。因此训练集中不存在无声视频。
【静音占比】Silence Ratio < 0.8（即静音超过 80% 的片段被剔除）。这个阈值相当宽松，允许大量静音存在，与“只保留语音片段”的策略配合后实际静音比例应远低于此。
【带宽】Bandwidth > 1,000 Hz，用于剔除严重带限、编码劣化或电话线路质量的音轨。
【感知质量】Audiobox-aesthetics 三维打分：PQ（Production Quality，制作质量，反映录音干净度与制作水准）> 5.0、CU（Content Usefulness）> 4.5、CE（Content Enjoyment）> 2.5。PQ 阈值定得较高，实质上偏好录音棚级/专业制作的音频。
【SNR】未使用显式信噪比阈值，噪声控制由 Audiobox-PQ 间接承担。
【背景音乐分离】未做源分离（source separation）。音乐作为混音的一部分保留在训练音轨中，并由 Qwen3-Omni-Captioner 在 caption 中描述（论文示例 caption 中就包含对背景电子乐的详细描述：合成器 pad、稳定节拍、小调旋律线、混音层级低不干扰旁白）。
【画外音/旁白剔除】未做剔除，且论文给出的完整示例本身就是画外音旁白 + 画面无说话人的广告片段，说明画外音样本被保留在训练集中。这可能是多说话人场景下 active-speaker 归属模糊问题的来源之一（论文 Limitations 承认该问题）。
【响度归一化】Phase 2 起引入 LUFS 响度归一化，目的是缓解 CFG 引起的响度爆炸（loudness explosion）——这是训练数据侧的处理，但动机来自推理侧的 CFG 行为。

### 语音/音效/音乐的分类与分别处理策略

分类工具与分流策略明确，且贯穿数据、训练、能力三个层次：
【分类工具】EAT（Efficient Audio Transformer，自监督预训练音频模型）作为音频分类器，对片段打 speech / non-speech 相关标签。语音子集的构造条件为：EAT-contained-Speech 与 EAT-contained-Singing 两个标签均判为 True（或满足模型正类置信度）。另有 Silero VAD 在预处理阶段做语音/非语音区间划分。
【分流目的】论文明确说明分流是“depending on the target capability (e.g., lip synchronization vs. general foley/ambience modeling)”——即为“唇同步”与“通用 foley/环境音建模”两种目标能力分别构建子集。
【实际策略：两段式分工】
- 音乐与音效能力在 1.3B 音频塔的预训练阶段注入：通用音效来自 WavCaps + VGGSound，音乐来自 JamendoMaxCaps，语音来自 in-house TTS。
- 语音与唇同步能力在联合训练阶段强化：联合训练最终只使用语音片段（占预处理片段 69.47%）。
【标注侧的分流】音频标注同样按类型分工——Qwen3-Omni-Instruct 只做语音转写（严禁包含非语音与音乐），Qwen3-Omni-Captioner 只做非语音音效与音乐描述；融合时非语音内容被限定在四个类目（环境音、音乐、音频主题/声源、结构性音频变化）且严禁出现人声与词语。
【代价】这一“重语音、轻音乐”的配比在 Limitations 中被明确承认为局限：模型在歌声、复杂声音纹理、音乐/器乐内容上表现退化，因为音频塔仅 1.3B、容量不足以承载精细的音高/谐波结构与长程时域依赖。此外论文还指出模型缺乏物理声学推理（如闪电与雷声之间的传播延迟未被显式建模或数据强制）。

## 训练配合

### 多阶段训练课程与数据课程调度（阶段划分依据：分辨率/时长/质量分/模态；低清→高清、图像→视频、短→长）

两大阶段、三个联合训练 phase，课程划分依据同时包含**模态、数据质量、分辨率**三个轴：
【Stage A：音频塔预训练】单独训练 1.3B 文本到音频 DiT（Wan2.1-1.3B 骨干，3D 位置编码换为时间轴 1D），数据覆盖音乐、通用音效、TTS 三域，定长片段，每个 clip 配一条含显式时长 token 的文本 prompt 以控制目标长度。音频 VAE 冻结。
【Stage B：渐进式联合训练（三 phase）】视频塔由 Wan2.2 A14B 初始化、音频塔由 Stage A 初始化、Bridge 随机初始化，三者从第一步起端到端联合优化（论文说明：早期实验中“先冻结双塔训 Bridge、再全量微调”的两段式方案很早就到达性能平台期，因此改为端到端）。
- Phase 1（360×640，多样数据，~61,500 小时，1 epoch，15 天）：非对称 sigma shift（视频 5.0 激进去噪 / 音频 1.0 平缓），激进文本 dropout p=0.5 强迫模型依赖 Bridge 学习跨模态对齐而非走文本捷径。目标：快速建立基础同步能力。
- Phase 2（360×640，质量过滤数据，~37,600 小时 / 16.8M clips，1 epoch，7 天）：将音频 sigma shift 对齐到 5.0 以强化音频去噪、提升音色保真；文本 dropout 降到 0.2 以允许文本引导的语义精修；引入 LUFS 响度归一化。目标：在稳定的跨模态注意力下精修一致性与音色。
- Phase 3（720×1280，最高质量子集，~11,000 小时，1 epoch，20 天）：跨模态对齐已稳定，模型可安全地把容量分配给更高分辨率与更精细的空间细节而不破坏已学到的同步结构。CP 从 8 增至 16，batch size 从 128 降到 64，checkpoint 间隔从 5000 步缩短到 2000 步。
【课程有效性的量化验证】Figure 9 给出 LSE-C / LSE-D 随训练步数（0→400K）的完整曲线并按阶段着色：Stage 1 中 LSE-D 快速下降、LSE-C 上升（快速学到基础同步模式）；Stage 2 中 LSE-D 继续下降、LSE-C 出现显著跃升（一致性与置信度提升）；Stage 3 中 LSE-D 进一步下降并趋于平台、LSE-C 稳定在高位（收敛到高质量唇同步）。这是“数据课程逐级收紧 → 同步指标单调改善”的直接证据。
【课程设计的三条轴】分辨率（360p→360p→720p）、数据质量（多样→质量过滤→最高质量）、噪声调度与文本 dropout（探索性→精修性）——三者协同变化，而非只调分辨率。

### 数据配比随训练阶段的变化（预训练/退火annealing/SFT高质量子集）

数据配比随阶段变化的机制是“规模递减 + 质量门槛递增”，具体数字完整公开：
【Phase 1（探索期）】~61,500 小时，来源多样化：SpeakerVid5M、中文剧集、卡通、电影、YouTube、OpenHumanVid。此阶段追求分布覆盖广度，不做额外质量收窄。
【Phase 2（质量收敛期）】在 Phase 1 语料基础上叠加三道正交过滤：
- OCR 无烧录字幕：保留 ~9.5M clips，并给其 prompt 追加 “This video has no subtitles.”（过滤 + 条件化混合策略）
- 唇音对应：LSE-D ≤ 9.5 且 LSE-C ≥ 4.5，保留 ~2.5M clips
- 视觉保真：DOVER 技术分 > 0.15，保留 ~2.4M clips
三者合并后 Phase 2 数据集 = 16.8M clips ≈ 37,600 小时，论文称其“balancing scale and quality”（在规模与质量间取得平衡）。相对 Phase 1 保留约 61%。
【Phase 3（精修期）】只用 720p 最高质量子集，~11,000 小时，DOVER 技术分 > 0.14（720p 尺度重标定）。相对 Phase 1 仅约 18%。
【无退火/SFT 阶段】MOVA 没有独立的 annealing 或 SFT 阶段，Phase 3 的高质量小数据集事实上承担了退火/精修的角色。
【伴随的超参数配比变化（Table 7）】视觉 sigma shift 三阶段恒为 5.0；音频 sigma shift 从 1.0 变为 5.0（Phase 2 起）；文本 dropout 从 0.5 降到 0.2（Phase 2 起）；音频 loss 权重三阶段恒为 0.2（即视频与音频的 velocity 回归损失按 1:0.2 加权，视频占主导）；weight decay 恒为 0.001；backbone LR 恒为 1e-5、Bridge LR 恒为 2e-5；LUFS 归一化从 Phase 2 起开启。

### 后训练数据（SFT精选集规模与筛选标准、偏好对数量与标注方式、reward model训练数据） ⚠️

MOVA 没有传统意义上的后训练阶段：论文未描述任何 SFT 精选集、偏好对（preference pairs）、DPO/RLHF 或 reward model 训练数据。质量提升完全通过预训练课程内部的数据收紧（Phase 2/Phase 3 的高质量子集）实现，可视为“把 SFT 融进了预训练课程末端”。
最接近后训练的两项设施都在推理侧或评测侧而非训练侧：
(1) 推理侧的 prompt rewriter（Qwen3-VL 抽结构化视觉描述 + Gemini 2.5 Pro 通过 in-context learning 改写为符合训练分布的 prompt），用于弥合用户输入与训练数据分布的 gap——这是用推理时的 prompt 工程替代了后训练的指令对齐。人类 Arena 显示 rewriter 带来的 ELO 提升显著（MOVA-720p 从 982.9 提升到 1025.3）。
(2) 开源代码库提供 LoRA 微调脚本，把后训练能力开放给社区自行完成。
[不确定]

### 数据处理基础设施与吞吐（NeMo Curator/Data-Juicer/自研，GPU加速比，处理规模，成本）

【数据处理基础设施】基于 Ray 分布式框架自研视频预处理 pipeline，论文称其“balancing data quality and processing efficiency”。未使用 NeMo Curator 或 Data-Juicer 等现成框架。打标环节混合使用 NVIDIA GPU 与华为昇腾 Ascend NPU。论文未给出数据处理阶段的吞吐量、GPU 加速比或处理成本数字。
【训练基础设施】
- 规模：1024 张 GPU（128 节点 × 8 卡），DP replicate size 64。
- 并行策略：FSDP 分片模型参数 + USP 序列并行；360p 用 CP=8（有效 batch 128），720p 用 CP=16（有效 batch 64）。
- MFU：约 35%。
- VAE 冗余消除：借鉴 Wan 的做法，每个 CP group 内输入预处理（主要是 VAE 编码）只执行一次，再由指定 rank 广播预处理特征给同组其他 rank，避免序列并行带来的重复 VAE 计算。
- 内存管理：采用手动内存管理以规避 OpenSora2 报告过的 Python GC 开销问题。
- MoE 适配：因 FSDP 要求计算图一致，A14B MoE 视频塔采用交替优化——奇数步对全部样本采样 high-noise timestep 并优化 high-noise DiT，偶数步采样 low-noise timestep 并优化 low-noise DiT；共享 Bridge 与音频塔每步都优化。
- 国产化适配：训练栈已移植到昇腾 NPU，并对 attention kernel、张量布局变换、旋转位置编码计算做了算子融合以降低框架开销。8 卡 Ascend 910A2 微基准（CP=4，DP-shard=2）：单步 34.1 秒，FP16 376 TFLOPs，单卡显存占用约 40GB，主机内存 ≥128GB。论文提醒该数字依赖软件栈版本，不应外推为大规模训练成本。
- 总成本：42 天 × 1024 GPU ≈ 43,000 GPU-days。
【序列长度瓶颈】720p、8 秒、24fps 的 clip 约产生 1.6×10^5 个 token，论文将其列为训练吞吐与推理延迟的主要瓶颈（尤其在最通用的 guidance 设置 NFE=3 下），并提出未来方向：更激进的时空压缩、分层或分块生成、面向长上下文视频 token 的系统级优化。

## 效果对比

### 数据策略消融的量化影响（区分：过滤严格度ablation / caption密度风格ablation / 数据配比ablation，及对应评测指标） ⚠️

论文的消融实验主要针对**训练策略与推理配置**，缺少严格意义上的“数据策略消融”（未做过滤严格度对照组、未做 caption 密度/风格对照组、未做数据配比对照组）。已有的量化证据如下：
【分辨率/阶段消融（Table 4）——间接反映数据课程效果】MOVA-360p vs MOVA-720p：DeSync 0.475→0.485、IB-Score 0.286→0.277（几乎无退化）；LSE-C 6.278→6.593（提升）；cpCER 0.177→0.149（提升）；IS 4.269→3.936、DNSMOS 3.797→3.671（音频侧略降，论文解释为模型容量向视觉复杂度倾斜的正常权衡）。验证了“先低清建立对齐、再升清精修”的数据课程有效。
【训练进程消融（Figure 9）——最直接的数据规模效应证据】LSE-C / LSE-D 随训练步数（0→400K）呈现分阶段的单调改善，且改善节点与三个数据阶段边界吻合。论文据此得出核心结论：架构机制（Bridge cross-attention）本身不足以获得高质量唇同步，音素到口型的映射必须从数据中学到，这需要更大模型容量与更多训练样本。这是“数据规模驱动同步能力”的直接证据。
【Dual CFG 尺度消融（Table 5）——推理侧】s_B 从 1.0 增至 4.0：LSE-C 峰值 7.891、DeSync 最低 0.365、IB-Score 与 LSE-D 同步改善；但代价是 DNSMOS 下降、cpCER 从 0.177 升到 0.264。论文称之为“conditional interference（条件干扰）”与“over-regularization”——过高的 s_B 让模型优先满足同步的几何约束，牺牲了语音本身的生成保真度与指令遵循（说什么、说得多自然）。
【T2VA 涌现能力消融（Table 6）】把参考图替换为空占位符：IS 4.269→4.370（提升）、DeSync 0.475→0.441（提升）、IB-Score 0.286→0.281（基本持平）、LSE-C 6.278→5.830 与 LSE-D 8.098→8.362（下降，因空占位符不提供唇部几何）。说明模型在缺少视觉结构约束时能更自由地探索联合视听流形。
【主观 Arena 消融（Figure 7/8）】MOVA-720p ELO 1113.8，显著高于 LTX-2（1074.1）、Ovi（925.4）、WAN2.1+MMAudio（886.9）；对 Ovi 与级联系统的胜率均超 70%，对 LTX-2 胜率 51.5%。内部 Arena 显示 prompt rewriter 是影响人类偏好的最关键因素（720p 从 982.9 → 1025.3），而 dual CFG（s_B=3.5）虽显著改善客观对齐指标，却使人类偏好从 1025.3 略降到 1014.5——论文归因为放大跨模态引导时相对削弱了文本指令的引导比重，偶尔导致指令遵循下降。这构成一个有价值的观察：**客观同步指标与人类整体偏好并非同向**。
[不确定]

### 质量vs数量的证据（小而精数据超越大而杂的案例） ⚠️

论文没有做“小而精 vs 大而杂”的对照实验，但其整个训练课程设计本身就是该理念的工程化实践，且带有量化痕迹：
- 数据规模逐阶段大幅缩减而性能持续提升：61,500 小时（Phase 1）→ 37,600 小时（Phase 2，约 61%）→ 11,000 小时（Phase 3，约 18%），而 LSE-C/LSE-D 在三个阶段中持续单调改善（Figure 9），Phase 3 的 LSE-D 进一步下降并趋于平台。
- 整体清洗保留率仅 26.39%（Table 1），其中“只保留语音片段”一条就砍掉约 26 个百分点，是一次为了目标能力（唇同步）主动放弃 40% 以上数据多样性的激进取舍。
- Phase 2 的三道过滤中，唇音对应子集仅 ~2.5M clips、DOVER 技术分子集仅 ~2.4M clips，相对 OCR 无字幕子集的 ~9.5M 大幅收窄，论文明确称最终 16.8M clips 的数据集是“balancing scale and quality”。
- 时间分配上也体现了对高质量数据的重视：Phase 3 仅用 11,000 小时数据却训练了 20 天（三阶段中最长），远超 Phase 2 的 7 天 / 37,600 小时。
需注意这些都是设计意图与训练曲线的间接证据，没有“同等算力下大而杂数据表现更差”的对照组实验支撑。[不确定]

### 训练数据domain分布与评测基准类目体系的对齐关系（如VABench七大类） ⚠️

训练数据的 domain 列举与自建评测基准的类目体系存在明显呼应，但论文未显式论证二者的对齐关系，也未给出配比映射：
【自建评测基准的六类场景】
1) 多说话人场景（评估多角色间同步语音、面部表情与交互）
2) 电影视频（要求电影级叙事生成，情节需参照原片背景）
3) 体育竞赛（聚焦运动员表现，部分 prompt 含解说员旁白）
4) 游戏直播视频（涵盖射击类、3D 类、竞技类游戏）
5) 运镜序列（评估平移、变焦、旋转等相机运动下的视觉真实性）
6) 动漫风格视频（含 2D 动漫与 3D 动画）
基准构建方式：从真实视频中提取首帧图像与对应 prompt；prompt 简要描述场景设定、人物、环境条件，并按场景需要融入音频相关信息，形成用于视听联合生成的统一 prompt。
【与训练数据 domain 的对应】训练数据列举的域（电影、动画/卡通、体育、教育、新闻、访谈、中文剧集、vlog）与基准六类中的电影、动漫、体育、多说话人（对应访谈/剧集）高度重合；但“游戏直播”与“运镜序列”两类在训练数据描述中没有明确对应来源，属于基准覆盖超出训练数据显式列举范围的部分。
【另一评测基准】Verse-Bench（600 组图文 prompt 对），由 GPT-5 把视觉与音频描述统一为单条 prompt 后使用；论文批评其“并非专为多场景视听生成评测设计”，这正是自建基准的动机。
【训练侧无对应的 domain 配比控制】论文未描述按基准类目对训练数据做定向配比或补齐，因此二者的“对齐”更像是自然重合而非刻意设计。[不确定]

## 其他信息

### summary_note

核心结论：MOVA 是本次调研中**数据处理方法可复现性最高**的样本，也是开源音视频联合生成方向上目前披露最彻底的技术报告。其独特价值集中在四点：
(1) **完整的定量漏斗**：Table 1 给出逐级时长保留率（100% → 84.57% → 58.75% → 26.39%），Table 9 给出全部 8 项过滤指标的具体阈值，Phase 2 三道子过滤各自的 clip 产出量（9.5M / 2.5M / 2.4M → 16.8M）也全部公开。这在闭源模型（Sora 2、Veo 3、Kling、Seedance）全面零披露的背景下极为稀缺。
(2) **speech-aware + shot-aware 的双重感知切分**：把 Silero VAD 的语音边界与 PySceneDetect 的场景切点联合驱动窗口采样，并给出两段完整伪代码（附录 A.3），显式区分单镜头/多镜头样本，同时保证不截断语句。这是针对唇同步任务定制的切分范式，可直接复用。
(3) **时序同步与语义同步的 OR 门分离**：SynchFormer DeSync ≤ 0.5 与 ImageBind IB-Score ≥ 0.2 以逻辑“或”组合，论文明确论证了用 AND 会同时误杀“无 onset 的环境音”和“语义分低的快速动作音效”。这是本主题下对该维度论证最清晰的一次表述。
(4) **分流标注 + LLM 融合的 joint AV caption schema**：三条互斥轨道（MiMo-VL 视觉 / Qwen3-Omni-Instruct 语音转写 / Qwen3-Omni-Captioner 非语音音频）配合强约束 prompt 抑制跨模态幻觉，再由 GPT-OSS-120B 做一致性裁决并融合；融合规则明确规定了“对白内容以转写为准、说话人动态以音频描述为准、视觉锚定以视频描述为准”的权威边界。全部 prompt 原文与完整 caption 示例公开（附录 A.5），可直接复刻。
最激进的数据决策：**最终联合训练只保留语音片段**（占预处理片段 69.47%），为唇同步能力主动牺牲了音效/音乐/环境音的数据覆盖，代价是论文自承的歌声、音乐、复杂声纹表现退化——这是一个关于“能力聚焦 vs 覆盖广度”的清晰权衡案例。
主要信息空白：数据合规与溯源（零披露）、安全过滤（零披露）、去重（零披露）、运动过滤（零披露）、几何结构化标注（未使用）、后训练数据（无该阶段）、以及严格意义上的数据策略消融（未做过滤严格度/caption 风格/数据配比的对照实验）。此外训练数据的 domain 配比、单/多镜头比例、语种比例均无数字。
方法论上最值得引用的一句结论：架构机制本身不足以获得高质量唇同步，音素到口型（phoneme-to-viseme）的映射必须从数据中学习，需要更大容量与更多训练样本——Figure 9 中 LSE-C/LSE-D 随三阶段数据课程单调改善的曲线是该论断的直接支撑。

## 不确定字段

以下字段的调研信息部分不确定（⚠️ 标注来源）：

- provenance_licensing
- domain_distribution
- language_accent_distribution
- motion_filtering
- deduplication
- safety_filtering
- geometric_structured_annotation
- synthetic_data_synthesis
- post_training_data
- data_ablation
- quality_over_quantity_evidence
- benchmark_taxonomy_alignment
