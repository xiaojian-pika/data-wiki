# Open-Sora 系列（Open-Sora 1.0/1.1/1.2/1.3/2.0，HPC-AI Tech）与 Open-Sora Plan 系列（v1.0–v1.5，北京大学 PKU-YuanGroup）。二者是两个独立项目，常被并称为「开源 Sora 复现双雄」：Open-Sora 由潞晨科技/HPC-AI Tech 主导，主打极致低成本训练（2.0 版 20 万美元）；Open-Sora Plan 由北大袁粒团队主导，主打社区协作复现与多维数据清洗漏斗。本条目合并调研，凡字段内容不同处均分项说明。

> 主题: 视频生成模型（含音视频同时生成）的数据处理：数据清洗流程、数据分布、打标方式

[← 返回首页](../index.md)

**目录**: [基本信息](#基本信息) · [数据规模与分布](#数据规模与分布) · [清洗流程](#清洗流程) · [打标方式](#打标方式) · [音视频对齐](#音视频对齐) · [训练配合](#训练配合) · [效果对比](#效果对比)

## 基本信息

### 名称

Open-Sora 系列（Open-Sora 1.0/1.1/1.2/1.3/2.0，HPC-AI Tech）与 Open-Sora Plan 系列（v1.0–v1.5，北京大学 PKU-YuanGroup）。二者是两个独立项目，常被并称为「开源 Sora 复现双雄」：Open-Sora 由潞晨科技/HPC-AI Tech 主导，主打极致低成本训练（2.0 版 20 万美元）；Open-Sora Plan 由北大袁粒团队主导，主打社区协作复现与多维数据清洗漏斗。本条目合并调研，凡字段内容不同处均分项说明。

### 发布机构/公司

（1）Open-Sora：HPC-AI Tech / 潞晨科技（Colossal-AI 团队），GitHub 组织 hpcaitech。（2）Open-Sora Plan：北京大学袁粒课题组（PKU-YuanGroup），联合鹏城实验室、兔展智能（Rabbitpre AI）等，HuggingFace 组织为 LanguageBind。二者无隶属关系，仅名称相近。

### 发布时间（技术报告/论文/开源时间）

【Open-Sora（HPC-AI Tech）】v1.0：2024年3月；v1.1：2024年4月25日（首次发布完整视频数据处理pipeline）；v1.2：2024年6月17日（对应论文 arXiv:2412.20404《Open-Sora: Democratizing Efficient Video Production for All》，2024年12月29日挂arXiv）；v1.3（1B）：2025年2月20日；v2.0（11B）：2025年3月12日发布，技术报告 arXiv:2503.09642《Open-Sora 2.0: Training a Commercial-Level Video Generation Model in $200k》。
【Open-Sora Plan（北大）】v1.0.0：2024年4月；v1.1.0：2024年5月；v1.2.0：2024年7月；v1.3.0：2024年10月；技术报告 arXiv:2412.00131《Open-Sora Plan: Open-Source Large Video Generation Model》于2024年11月28日提交（内容对应 v1.3）；v1.5.0：2025年6月5日发布（8B，SUV 稀疏 MMDiT + 8×8×8 WFVAE）。

### 类型（模型/数据集/工具链/评测基准）

均为「模型 + 完整工具链」双重属性的开源项目：既发布模型权重（T2V/I2V 扩散模型 + VAE），也发布端到端训练代码与数据处理代码，同时附带部分标注数据集。非评测基准。Open-Sora 额外具备「训练成本工程化范本」属性（逐阶段成本核算），Open-Sora Plan 额外具备「社区协作复现范本」属性。

### 开源程度（权重/代码/数据/pipeline各自是否开源） ⚠️

两者都属于视频生成领域开源度最高的一档，但仍非「数据集全开源」。
【Open-Sora（HPC-AI Tech）】权重：开源（HuggingFace hpcai-tech，Apache 2.0）；代码：开源（推理+训练+分布式优化）；数据处理 pipeline：完整开源，这是该项目最具参考价值之处——v1.1/v1.2 分支下 tools/ 目录包含 scene_cut（PySceneDetect 切镜+切分）、scoring（aesthetic / optical_flow / ocr / matching 四个打分器）、caption（PLLaVA、LLaVA、LLaMA3 打标）、datasets（datautil 过滤清洗），并有 docs/data_processing.md 串联全流程，每一步都给出可直接运行的 torchrun 命令与阈值示例；训练数据本身：不开源（仅公布来源数据集名称与筛选阈值，未发布筛选后的 meta 文件）。需注意：2.0 版 main 分支已移除 tools/ 数据处理目录（tools/datasets、tools/scoring、docs/data_processing.md 在 main 分支返回 404），完整数据处理代码需回到 v1.2.0 等历史 tag 获取；2.0 技术报告描述的新 pipeline（PaddleOCR、VMAF、Laplacian 等）在 main 分支中并未见对应开源实现，此处开源度实际上是**退步**的。
【Open-Sora Plan（北大）】权重：开源；代码：开源（训练+推理+WFVAE）；数据：部分开源——各版本在 HuggingFace（LanguageBind/Open-Sora-Plan-v1.1.0 / v1.2.0 / v1.3.0）发布了「Data and Annotations」标注数据与 prompt_refiner 数据集；数据处理 pipeline：论文与 Report 文档中给出了完整的过滤步骤、工具、阈值与逐级保留率（这是全行业罕见的定量披露），但未见独立打包的 curation 代码库，复现需按文档自行拼装。[不确定]

### 是否支持音视频同时生成，以及实现方式（原生联合/级联/MoE融合）

均不支持。Open-Sora 与 Open-Sora Plan 全系列（截至 Open-Sora 2.0 / Open-Sora Plan v1.5.0）均为纯视觉（无声）视频生成模型，输出无音轨；技术报告、GitHub 文档与数据 pipeline 中均无任何音频编码器、音频 latent、音视频联合去噪或音视频对齐的设计与描述。数据处理侧也完全不涉及音轨——切分、打分、打标全部基于视觉帧。因此本条目下所有音频与音视频对齐相关字段（audio_category_distribution、joint_av_caption_schema、dialogue_transcription_attributes、av_sync_detection、sync_metric_and_threshold、temporal_vs_semantic_sync、audio_quality_filtering、audio_type_handling）均为「不适用」。

### 调研信息来源列表（论文/技术报告/官方文档/新闻的URL，标注每条来源的性质：官方一手/同团队旁证/第三方报道）

【官方一手】1) Open-Sora 2.0 技术报告 arXiv:2503.09642 https://arxiv.org/abs/2503.09642 与 HTML 全文 https://arxiv.org/html/2503.09642v1 （数据 pipeline、数据统计图、三阶段成本表）；2) Open-Sora 1.2 论文 arXiv:2412.20404 https://ar5iv.labs.arxiv.org/html/2412.20404 （数据来源、80k 小时统计、bucket 策略、35k H100 GPU 小时）；3) Open-Sora GitHub 主仓 https://github.com/hpcaitech/Open-Sora （版本时间线、开源范围）；4) Open-Sora v1.2.0 数据处理代码与文档（一手代码级证据）：docs/data_processing.md、tools/scoring/README.md、tools/scene_cut/README.md、tools/caption/README.md，raw 路径形如 https://raw.githubusercontent.com/hpcaitech/Open-Sora/v1.2.0/tools/scoring/README.md ；5) Open-Sora Plan 论文 arXiv:2412.00131 https://arxiv.org/html/2412.00131v1 （数据来源表、七级过滤漏斗与逐级保留率表、训练阶段表）；6) Open-Sora Plan GitHub https://github.com/PKU-YuanGroup/Open-Sora-Plan 及 docs/Report-v1.3.0.md、docs/Report-v1.5.0.md（阈值、27% 保留率、prompt refiner 细节、v1.5 数据规模）。
【第三方报道】7) MarkTechPost 关于 Open-Sora 2.0 发布的报道 https://www.marktechpost.com/2025/03/14/hpc-ai-tech-releases-open-sora-2-0-an-open-source-sota-level-video-generation-model-trained-for-just-200k/ ；8) HuggingFace Papers 页面 https://huggingface.co/papers/2503.09642 与 https://huggingface.co/papers/2412.00131 ；9) comfyui-wiki 发布快讯 https://comfyui-wiki.com/en/news/2025-03-13-open-sora-2-release 。

## 数据规模与分布

### 训练数据量级（视频条数/小时数/token数，预训练与SFT分开） ⚠️

【Open-Sora 2.0】按训练阶段给出数据量（论文表格口径为「参与该阶段训练的视频条数」）：阶段一 70M 视频（256px T2V）、阶段二 10M 视频（256px T2V/I2V）、阶段三 5M 视频（768px T2V/I2V）。报告未给出清洗前原始视频总量、总小时数或 token 数，也未单列 SFT 规模（阶段三 5M 高清子集即事实上的高质量精调集）。
【Open-Sora 1.2】原始池约 30M 视频片段（时长 2–16 秒），合计约 80k 小时；其中 Panda-70M 高质量子集 20M 条约 41k 小时；最终高质量阶段约 2M 片段、5k 小时。图像约 3M 张。
【Open-Sora Plan v1.3】图像 18.0M 张，视频约 28M 条（清洗前）；清洗后 Panda70M 部分保留约 19M 条（27% 保留率口径）。
【Open-Sora Plan v1.5】图像 1.1B 张（仅做分辨率检查，不做质量过滤），视频 40M 条高质量样本。
注：以上均为条数口径，两个项目都未公布训练 token 数，Open-Sora 2.0 未公布总小时数。[不确定]

### 数据来源构成（自有/公开数据集/网络爬取/授权采购/合成数据） ⚠️

两个项目均**完全依赖公开数据集与免费授权素材站，无自有版权库、无采购授权数据、无合成数据**，这是与 Sora 2 / Veo 3 / Seedance 等闭源模型最本质的差别，也是其数据配方可完全复现的原因。
【Open-Sora 1.2 明确列出】Webvid-10M（1000万视频-文本对，stock footage）、Panda-70M（7000万对，取 20M 高质量子集）、HD-VG-130M（1.3亿对，BLIP-2 caption）、MiraData（7.7万长视频，游戏/城市漫游）、Vript（40万密集标注视频）、Inter4K（1000条 4K 片段）、以及 Pexels / Pixabay / Mixkit 等免费许可素材站；图像用 LAION 子集（美学分>6.5）与 Unsplash-lite。
【Open-Sora 2.0】技术报告**未披露**数据来源与具体数据集名称，只描述了过滤方法，属于该报告的显著信息缺口（推测延续 1.x 的公开数据集组合，但无官方确认）。[不确定]
【Open-Sora Plan v1.3】图像：SAM 11.1M（配 LLaVA caption）、Anytext-3M 英文子集 1.8M（约占该集 50%）、LAION-5B 中筛出的 16 万张高质量人像、内部 QWen2-VL 标注数据 5.0M；视频：Panda70M 21M（横屏）、VIDAL 3M（竖屏，来自 YouTube Shorts）、ShareGPT4Video 0.8M（源自 Mixkit / Pexels / Pixabay 的 CC0 素材）。
【Open-Sora Plan v1.5】图像来自 Recap-DataComp-1B、COYO-700M、LAION-Aesthetics；视频来自 Panda-70M 与「内部来源（internal sources）」——v1.5 首次引入未公开的内部视频数据，是该系列数据透明度的一处倒退。

### 数据合规与溯源（授权数据占比、rights-cleared数据集、C2PA等） ⚠️

两个项目对训练数据合规与溯源的处理都很薄弱，属于典型的学术/开源项目做法：
- 依赖第三方公开数据集本身的许可（Panda-70M 源自 HD-VILA-100M 的 YouTube 视频，Webvid-10M 源自 Shutterstock 水印素材，VIDAL 源自 YouTube Shorts），这些数据集的版权状态本身存在争议，项目方未做二次授权核查，也未披露授权数据占比。
- Open-Sora Plan 使用的 ShareGPT4Video 部分明确标注来自 Mixkit / Pexels / Pixabay 的 CC0 免版权素材，Open-Sora 1.2 亦明确使用 Pexels / Pixabay / Mixkit 免费许可素材，这是两者中唯一可称为 rights-cleared 的部分，但均未给出占比数字。
- 未使用任何采购授权数据，未建立 rights-cleared 数据集清单。
- 输出侧：均未实现 C2PA 元数据、不可见水印或生成内容检测工具，模型权重以 Apache 2.0 等宽松许可放出，下游使用无溯源约束。
- 值得注意的是 Webvid-10M 素材带有 Shutterstock 水印，Open-Sora 1.2 将其用于第一阶段低分辨率预训练（240p/360p），意味着水印信号进入了模型早期训练——这是社区已知的开源复现通病，项目文档未讨论其影响。[不确定]

### 片段时长分布与切分策略

【Open-Sora 2.0】预处理阶段先剔除时长 <2 秒的原始视频；切镜后，长于 8 秒的片段被强制切成多个 8 秒段，短于 2 秒的片段丢弃，因此训练片段时长严格落在 [2s, 8s] 区间。论文 Figure 3 统计显示时长分布明显右偏，**接近一半的片段集中在 6–8 秒**。同时限制输出片段 fps<30。
【Open-Sora 1.x】片段时长 2–16 秒，配合 bucket 机制按帧数分桶，支持 2s~15s 变长训练。
【Open-Sora Plan v1.3】用 ffmpeg 统一切成 16 秒片段，随后用 LPIPS 做跳切（jump cut）检测，只保留帧数落在 [32, 512] 区间的片段（24fps 下约 1.3–21 秒）。
【Open-Sora Plan v1.5】视频训练从 57 帧 @24fps（约 2.4 秒）逐步升到 121 帧 @24fps（约 5 秒）。

### 分辨率/宽高比分布与分桶策略

【Open-Sora 2.0】预处理即剔除宽高比落在 [1/3, 3] 之外的视频，并把片段长边限制在 ≤1080px、统一转 H.264。Figure 3 显示宽高比（高/宽）**多数集中在 0.5–0.75 区间，即以 16:9 横屏为主体**。推理支持 256px 与 768px 两档，覆盖 16:9、9:16、1:1、2.39:1。
【Open-Sora 1.x】采用显式**分桶（bucket）策略**：把分辨率、宽高比、帧数三维组合成预定义 bucket，每个 bucket 单独设置 batch size 以均衡 GPU 负载，从而在同一次训练中混合 144p~720p、任意宽高比、2~15 秒的样本。这是 Open-Sora 最被广泛复用的工程设计之一。
【Open-Sora Plan v1.3】视频训练固定为 93×352×640（约 16:9）；v1.5 图像侧做多分辨率训练，覆盖 (1,1)、(3,4)、(4,3)、(9,16)、(16,9) 五种宽高比并配合 Min-Max Token 策略，视频侧则固定 9:16 比例，最高 121×576×1024。
【数据来源侧的比例控制】Open-Sora Plan 显式做了横竖屏配比：Panda70M 21M 提供横屏，VIDAL 3M 提供竖屏（YouTube Shorts），是少见的显式竖屏数据补充策略。

### 类别/domain分布与配比策略（人物、动作、场景、风格等比例控制与概念均衡） ⚠️

两个项目**均未做显式的类别/domain 配比控制与概念均衡**，这是其数据策略相对工业界模型（Seedance、Movie Gen 等有精细 domain taxonomy 与配比表）最明显的短板。
可识别的隐含 domain 结构：
- Open-Sora 1.2 通过混合不同来源数据间接实现 domain 多样性：Webvid/Panda-70M 提供通用网络视频（YouTube 长尾），MiraData 专门补充游戏画面与城市漫游长镜头，Vript 补充密集标注的电影化内容，Inter4K 补充 4K 高清素材，LAION/Unsplash 补充静态高美学图像。这是「按数据集功能分工」而非「按语义类目配比」的思路。
- Open-Sora Plan 从 LAION-5B 中专门筛出 16 万张高质量人像图，用于强化人物生成能力，是唯一一处显式的类目定向补充；VIDAL 3M 竖屏 Shorts 则隐含大量人物口播/生活类内容。
- 两者都未公布人物/动作/场景/风格的比例数字，未做长尾概念均衡（concept balancing），未做语义聚类后的重采样。Open-Sora 2.0 报告仅按「美学分/时长/宽高比/caption 长度」四个低层维度给出统计图（Figure 3），完全没有语义类目维度的统计。
- 风格维度仅在 caption 中作为一个描述字段存在（Open-Sora 2.0 的六要素之一「video style」），但未据此做训练集配比。
结论：domain 分布基本是源数据集分布的被动继承，而非主动设计。[不确定]

### 音频类别分布与配比（语音/音效foley/音乐/环境音/静音的比例及控制策略）——AV模型独有维度

不适用。Open-Sora 与 Open-Sora Plan 全系列均不生成音频，训练数据 pipeline 不处理音轨，无语音/音效/音乐/环境音的分类、配比或统计。

### 叙事结构分布（单镜头vs多镜头、平均clip时长、镜头数分布、是否含原生音轨）

两个项目的数据均为**严格的单镜头（single-shot）片段**，这是其 pipeline 的核心设计取向：
- Open-Sora 2.0 用 FFmpeg libavfilter 的 scene score 做镜头边界检测并切分，再叠加基于 PySceneDetect 的相机抖动/跳变检测二次剔除，确保片段内无镜头切换；长片段被机械切成 8 秒段。
- Open-Sora 1.x 用 PySceneDetect 检测场景并切分，输出命名为 `{video_id}_scene-{scene_id}.mp4`，同样是单场景片段。
- Open-Sora Plan v1.3 在 16 秒切分之后专门加了一级 **LPIPS 跳切检测（jump cut detection）**，用感知距离突变识别切换点，保留率 97%，明确目标就是滤除含镜头切换的片段。
因此：多镜头叙事样本被主动剔除而非保留，训练数据中平均镜头数=1，不含跨镜头一致性监督信号，模型也不具备多镜头叙事生成能力。平均 clip 时长：Open-Sora 2.0 偏向 6–8 秒，Open-Sora Plan 约 1.3–21 秒（16 秒切分后按帧数区间保留）。是否含原生音轨：数据本身有音轨但被 pipeline 完全忽略并在转码中丢弃。

### 语言/口音分布（多语种唇同步能力的数据基础）

不适用于语音层面（无音频生成）。文本侧：Open-Sora Plan 明确只用 Anytext-3M 的**英文子集**（约占该数据集 50%，1.8M），caption 与 prompt 全部为英文；Open-Sora 系列 caption 亦全部为英文（PLLaVA、LLaVA-Video、Qwen2.5-Max 均输出英文描述）。两个项目均未做多语种 prompt 支持的数据建设，中文 prompt 需依赖外部翻译。口音/唇同步相关数据基础：无。

## 清洗流程

### 清洗漏斗整体结构（几级过滤、各级顺序）

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

### 漏斗定量保留率（每级过滤的输入/输出量与最终保留率，如Apollo 27%） ⚠️

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

### 镜头切分方法（PySceneDetect/自研模型/shot-aware splitting）

【Open-Sora 1.x】使用 **PySceneDetect**，代码完全开源：`python tools/scene_cut/scene_detect.py meta.csv` 输出每个视频的场景时间戳列表（形如 `[('00:00:01.234','00:00:02.345'), ...]`），再由 `tools/scene_cut/cut.py` 按时间戳切分，产物命名 `{video_id}_scene-{scene_id}.mp4`；前置还有 `convert_id_to_path.py` 做损坏文件过滤（输出 intact 标记列）。
【Open-Sora 2.0】改用 **FFmpeg libavfilter 的 scene score** 做镜头边界检测（比 PySceneDetect 更快、可与转码同流水线完成），随后仍用 **PySceneDetect 的 Shot Boundary Detection** 作为相机抖动/异常跳变的二次检测器。切分后做固定 8 秒截断。
【Open-Sora Plan】不做基于内容的切镜，而是先用 **ffmpeg 机械切成 16 秒定长片段**，再用 **LPIPS 感知距离**检测片段内是否存在跳切（jump cut），把含切换的片段整条丢弃（保留率 97%）。这是与前者不同的技术路线：以「检测并丢弃」代替「检测并在切点处切分」，实现更简单但会损失部分可用素材。

### 质量过滤（美学评分、清晰度、OCR文字过滤、黑边/水印/logo检测）

【美学】两者都用 LAION 系的 CLIP+MLP 美学打分器（improved-aesthetic-predictor，在 176K SAC + 15K LAION-Logos + 250K AVA 图文对上训练，输出 1–10 分）。Open-Sora 文档给出的经验刻度：5.5 为「尚可」阈值，6.5 为「高美学」阈值，好的文生图模型可达 7.0+；视频取首/中/尾三帧评估；代码示例过滤命令为 `python -m tools.datasets.datautil meta.csv --aesmin 5.0`。Open-Sora 1.2 对 Panda-70M 的筛选阈值为**美学分 > 4.5**（得到 20M 子集 / 41k 小时）；图像侧 LAION 子集用**美学分 > 6.5**。Open-Sora Plan 用 **LAION Aesthetic v2 ≥ 4.75**（论文明确说明选 4.75 是因为它同时能顺带滤掉大量含过多文字的画面），且对**美学分 > 6.25 的样本在 caption 前自动加上 「A high-aesthetic scene」 前缀**——把美学分转化为可控条件而非仅作过滤。Open-Sora 2.0 的美学分分布统计显示训练数据主体落在 **4.5–5.5**（即刻意保留了大量「中等美学」样本以保证多样性，而非只留高分样本）。
【清晰度/模糊】Open-Sora 2.0 用 **OpenCV Laplacian 算子的方差**做模糊检测，视频取五个均匀间隔帧分别判定后多数表决；Open-Sora Plan 用 **DOVER 的 technical 分支预测分（阈值 ≥ 0）**，专门针对压缩伪影与低码率片源，比单纯的模糊算子更贴近真实画质退化。Open-Sora 2.0 另在预处理阶段用 **bpp（bits per pixel）< 0.02** 直接剔除低码率片源。
【OCR / 文字过滤】三条不同路线：Open-Sora 1.x 用 **DBNet++（MMOCR 实现）**，输出列 `ocr` 记录**检测置信度 > 0.3 的文字区域数量**，用于剔除新闻播报、广告等密集文字场景；Open-Sora 2.0 用 **PaddleOCR**，只统计置信度 > 0.7 的文字，文字过多则丢弃；Open-Sora Plan 用 **EasyOCR 只检测画面底部 20% 区域**（字幕的典型位置）并采取**裁剪而非丢弃**的策略（edge ratio 0.20），论文同时坦承该方法的局限——无法处理画面中央的广告字、演讲字幕等。
【黑边/水印/logo】三者均**未实现**专门的黑边检测、水印检测或 logo 检测模块，这是共同缺口；Webvid-10M 的 Shutterstock 水印问题因此未被处理。

### 运动过滤（光流/运动分数阈值、静态与抖动剔除）

【Open-Sora 1.x】使用 **UniMatch（GMFlow）光流模型**计算稠密光流分数（`tools/scoring/optical_flow/inference.py`，输出列 `flow`），分数越高运动越大，用于剔除静止/近静止片段；同时把光流结果用于**相机运动检测**（识别 pan left / zoom in 等），高置信度的相机运动标签会被拼接进 caption。相机运动检测模块单独开源在 tools 下。
【Open-Sora 2.0】改用 **FFmpeg libavfilter 的 VMAF motion score**（比光流模型快得多，可在转码流水线内顺带算出），**双向剔除**：运动分极低（静止画面）与极高（剧烈晃动/混乱切换）者都丢弃。运动分数还会被**追加到 caption 文本中**，使推理时可通过 prompt 控制生成视频的运动幅度。
【Open-Sora Plan】用 **LPIPS 跳帧感知距离**代替光流作为运动度量（计算成本远低于光流），保留区间 **0.001 ≤ score ≤ 0.3**：低于 0.001 视为几乎静止，高于 0.3 则「存在明显抖动与闪烁」。作者说明该阈值经**人工抽检 2000 条视频**验证精度可接受。且如前所述，在字幕裁剪之后会**再跑一遍运动复检**（保留率从 44% 降到 42%）。
三者共同点：运动过滤都是双向的（既剔静止也剔抖动），且运动分数最终都被利用为可控条件而非仅作过滤器。

### 去重方法（精确去重与基于embedding的语义去重分别记录） ⚠️

两个项目**均未实现实质性的去重环节**，这是其数据 pipeline 相对工业界的明显缺口。
- Open-Sora 的 tools/ 目录下没有独立的去重模块，README 特性列表虽提及过 deduplication，但 v1.2.0 的 docs/data_processing.md 四阶段流程中并无去重步骤，tools/datasets/datautil 主要做基于分数列的过滤与 meta 合并，未见 embedding 级语义去重实现。
- Open-Sora Plan 的七级漏斗表中同样不含去重级，论文与各版本 Report 均未讨论精确去重（哈希）或语义去重（embedding 聚类）。
- 潜在风险：Panda-70M 源自 HD-VILA-100M 的 380 万条长视频切成 7080 万片段，同一长视频的相邻片段高度相似；Open-Sora Plan 又叠加使用 ShareGPT4Video（同样源自 Pexels/Pixabay），来源重叠与片段内冗余都未被处理。[不确定]

### VLM/LLM作为质检员（多模态大模型质量打分与错配剔除，2026年从浅层打分器转向大模型语义判断的趋势）

两者都**停留在「浅层打分器」阶段，尚未转向「大模型语义判断」范式**——所有过滤判据（美学分、光流/VMAF/LPIPS 运动分、Laplacian 模糊、DOVER 技术分、OCR 文字数）都是轻量专用模型或传统 CV 算子输出的标量分数 + 阈值，没有任何一级过滤是由 VLM/LLM 做语义级判断。
唯一接近「模型作判官」的机制是 **Open-Sora 1.x 的 matching score（图文匹配分）**：用 **CLIP** 计算视频中间帧与 caption 的余弦相似度（`tools/scoring/matching/inference.py`，输出列 `match`），用于剔除图文错配样本。但 CLIP 的语义判别力远弱于当代 VLM 判官，且只取单帧、不看时序。
VLM 在两个项目中的角色是**打标器而非判官**（PLLaVA-13B / LLaVA-Video / Qwen2-VL-7B / Qwen2.5-Max 只负责生成 caption，其输出不被用来给样本打质量分或做剔除决策）。Open-Sora Plan 的 Qwen2-VL 打标后处理也只是用 28 条常见开场短语的规则表去掉 「The video shows」 之类前缀，属规则清洗而非模型判断。
对比意义：这正是 2024–2025 年开源复现项目与 2026 年前沿实践之间的主要代差所在——Open-Sora 系列的 pipeline 可视为「VLM-as-judge 之前」的完整技术快照。

### 安全与合规过滤（NSFW、版权、人脸/隐私） ⚠️

两个项目的技术报告与文档中**均未描述任何 NSFW 检测、暴力内容过滤、人脸/隐私保护或版权内容检测环节**，安全过滤实际上是完全缺位的，隐含地依赖上游公开数据集（Panda-70M、LAION、Webvid 等）自身已做过的安全清洗。间接相关的仅有：Open-Sora Plan 从 LAION-5B 中筛选人像图时做了「高质量」筛选但无隐私/肖像权考量；两者的 OCR 过滤剔除密集文字场景，可顺带滤掉部分带版权声明/台标的素材，但这是副作用而非设计目的。模型权重以 Apache 2.0 等宽松许可发布，也未附带输出侧安全分类器或水印。这是开源复现项目相对闭源商业模型（Sora 2 的 CSAM 源头筛除 + 多级安全分类器）最大的合规差距。[不确定]

## 打标方式

### 使用的caption模型（自研VLM/开源模型，模型规模）

【Open-Sora 1.x】主力为 **PLLaVA 13B**（选 13B 版本以平衡速度与质量，配置 2×2 空间池化 pooling_shape 4-12-12，输入从视频均匀抽取 4 帧）。文档明确解释了为何不用 GPT-4V：「GPT-4V 效果更好，但 20 秒/样本的速度对我们太慢」——这是开源项目在打标成本上的典型权衡记录。另提供 LLaVA v1.6-Mistral-7B 打标脚本与 **LLaMA 3** 用于从 caption 中抽取物体/标签（tag），代码见 tools/caption/。早期版本亦提及 GPT-4V 作为可选高质量通路。
【Open-Sora 2.0】按训练阶段分级用不同模型，形成「打标质量金字塔」与数据金字塔对应：低分辨率（256px）阶段海量数据用开源 **LLaVA-Video** 打标；高分辨率（768px）阶段的精选 5M 数据改用更强的 **Qwen 2.5 Max**（闭源 API 模型）重新打标，理由是「获得更准确、语义对齐更好的 caption」。这种「粗标底层 + 精标顶层」的分级打标是其低成本策略的重要一环。
【Open-Sora Plan v1.3】主力为 **Qwen2-VL-7B-Instruct**；Panda70M 部分直接复用其官方公开 caption；stock footage 部分使用 **ShareGPT4Video** 的标注；VIDAL 部分沿用其原有的 OFA / mPLUG-Owl / ChatGPT 多模型精修 caption。另外单独训练了一个 **prompt refiner**：以 **LLaMA-3.1-8B-Instruct** 为底座做 LoRA 微调（rank 64，batch 32，1 epoch，单张 H100 上 30 分钟训完）。

### caption密度与结构化程度（短/长/密集描述、结构化字段如镜头运动、风格标签）

两个项目都走**长密集结构化 caption** 路线，且都把结构化字段设计明文写入 prompt 模板，可直接复用：
【Open-Sora 2.0】要求 LLaVA-Video 覆盖**六个方面**：1) 主体（main subjects）；2) 主体动作（subjects' actions）；3) 背景与环境（background and environment）；4) 光照条件与氛围（lighting condition and atmosphere）；5) 相机运动（camera movement）；6) 视频风格（video style）。长度统计（Figure 3）：**超过 70% 的视频 caption 长度超过 75 个词**。此外把**运动分数（motion score）直接追加到 caption 文本末尾**，使推理时可用文本控制运动幅度。
【Open-Sora 1.x】采用「描述 + 显式条件标签」的拼接式结构：在 PLLaVA 生成的自然语言描述之后，追加**美学分、运动分、相机运动**三类标量/枚举条件，形如 `aesthetic score: 5.5, motion score: 10, camera motion: pan left`。这套「score condition」机制是 Open-Sora 1.2 的标志性设计，使得同一模型可在推理时通过文本控制画质与运镜，无需额外条件通道。
【Open-Sora Plan v1.3】prompt 要求模型**按时间顺序**描述物体、场景、人物与相机运动；后处理用 28 条常见开场短语表去掉 「The video shows」 等冗余前缀。对美学分 > 6.25 的样本自动前置 「A high-aesthetic scene」。
【训练-推理 caption 分布错配的显式处理（Open-Sora Plan 独有）】作者指出训练 caption 密集冗长而用户实际 prompt 常不足 10 词，为此专门构建 **19,500 条 caption 的 refiner 训练集**，覆盖四种长度/风格：11,000 条 COCO 短用户 caption、5,000 条 DiffusionDB 标签式 caption、3,000 条 JourneyDB 中等长度 LLM caption、500 条来自 Sora/Vidu/Pika/Veo 官方演示与 GPT 生成的超长超现实 caption；用 ChatGPT 将其统一改写为「主体描述 + 动作 + 场景描述（+ 可选镜头语言与氛围）」的目标格式，再 LoRA 微调 LLaMA-3.1-8B 作为推理时的 prompt 扩写器。这是本次调研中对「caption 分布错配」处理得最系统、且训练数据构成完全公开的案例。

### 音视频联合caption结构（是否同时覆盖视觉+听觉轨道、是否分流为独立字段，如LTX-2全音景描述、Script-a-Video factorized streams、Foley-Omni三字段）

不适用。两个项目均不生成音频，caption 只覆盖视觉轨道（主体/动作/背景/光照/运镜/风格），不存在听觉轨道描述字段，也无 LTX-2 式全音景描述或 Foley-Omni 式三字段分流设计。

### 对白转写与说话人属性标注（ASR转写+说话人身份/语言/口音/情绪）

不适用。数据 pipeline 完全不处理音轨，无 ASR 转写、无说话人身份/语言/口音/情绪标注。

### 几何与结构化标注（相机参数、深度、3D point tracks、动作标注、显式状态）

标注深度较浅，仅有二维运动层面的结构化信息，无三维几何标注：
- 有：**相机运动标签**——Open-Sora 1.x 基于 UniMatch 光流做相机运动分类（pan left、zoom in 等），只对高置信度片段打标并写入 caption，模块单独开源在 tools 下；Open-Sora 2.0 与 Open-Sora Plan 则把相机运动作为 caption 的一个描述字段由 VLM 自然语言生成，非独立结构化字段。
- 有：**标量质量条件**——美学分、运动分（VMAF/光流/LPIPS）、OCR 文字区域数、DOVER 技术分，均以结构化列存于 meta CSV 中，部分被拼进 caption 作为可控条件。
- 无：相机内外参、深度图、3D point tracks、光流场本身作为条件输入、显式物体状态/动作标注（action label）、分割掩码等。
- Open-Sora Plan v1.3 训练了独立的 **Structure Controller**，从数据中提取 **canny 边缘、depth 深度图、sketch 草图**三类结构信号作为控制条件（20k 步，8 卡 NPU），这是唯一一处涉及深度信息的标注，但用途是 ControlNet 式条件控制而非主训练数据的标注 schema。

### 合成数据构造（受控扰动/编辑构造训练对，如InstructAV2AV）

两个项目均**不使用模型合成的视频数据**，训练视频全部来自真实采集的公开数据集。存在的「构造式数据」仅限于两类，且都不是视频合成：
1. **掩码构造的多任务训练对（Open-Sora Plan v1.3 Image Controller）**：在同一批真实视频上通过**不同的帧掩码模式**构造出 T2V、I2V（首帧条件）、Transition（首尾帧条件）、Continuation（视频续写）四类任务样本，第一阶段共 50M 样本分 7 个渐进子步，帧保留率从 50% → 25% → 12.5% 逐步降低，任务配比逐步向 I2V（40%）与 Transition（40%）倾斜；第二阶段 15M 高质量数据延续该配比。这是用掩码从单一真实数据派生多任务监督信号的典型做法，不涉及生成模型。
2. **caption 层面的合成（Open-Sora Plan prompt refiner）**：用 ChatGPT 把 19,500 条真实来源 caption 改写为统一格式的「短 prompt → 长 prompt」配对，属文本侧合成。
此外 Open-Sora 2.0 在推理侧构建了 text-to-image-to-video 流水线（先用 FLUX 文生图再图生视频），但这是推理策略，不是训练数据合成。

### 人工介入程度（人工标注、人工质检、模型初筛+人工复核）

人工介入程度很低，主要用于**阈值校准与抽样验证**，而非规模化标注或复核：
- 【Open-Sora Plan】唯一明确记录的人工环节是：为验证 LPIPS 运动过滤阈值（0.001–0.3）的可靠性，**人工检查了 2000 条视频**，确认该方法精度足够后才全量应用。这是把人工用于「验证自动化规则」的低成本范式。
- 【Open-Sora 系列】文档中未记录任何人工标注或人工质检环节；caption 全部由 VLM 生成，无人工复核；tools/caption 的说明反而明确指出「人工标注视频昂贵且耗时，因此采用强大的图像/视频描述模型来生成 caption」，是主动放弃人工标注的显式表态。
- 人工介入更多体现在**评测侧**：Open-Sora 2.0 用 100 条 prompt 做人工偏好评测（对比 HunyuanVideo、Runway Gen-3 Alpha、Step-Video-T2V、Luma Ray2），从视觉质量、prompt 遵循、运动质量三个维度打胜率。
- 两个项目本质上都是「全自动 pipeline + 极少量人工抽检校准」，这也是其能以极低成本处理数千万级视频的前提。

## 音视频对齐

### 音视频同步检测方法（口型同步、事件对齐）

不适用。无音频模态，不存在音视频同步检测环节。

### 同步检测具体指标与阈值（SyncNet/Synchformer/LSE/自研，阈值数值，如MOVA LSE-D≤9.5且LSE-C≥4.5、SkyReels-V4 SyncNet |offset|≤3∧conf>1.5）

不适用。未使用 SyncNet、AV-align 或任何同步指标。

### 时序同步vs语义同步的分离处理（时间对齐与内容语义匹配作为两个独立过滤条件）

不适用。无音频模态。（若强行类比，视觉-文本的语义匹配由 Open-Sora 1.x 的 CLIP matching score 承担，但与音视频同步无关。）

### 音频质量过滤（SNR、静音检测与静音占比阈值、无音轨剔除、画外音源剔除、背景音乐分离）

不适用。pipeline 不读取音轨，转码统一为 H.264 视频流，无 SNR、静音检测、音轨缺失剔除或人声/背景乐分离等任何处理。

### 语音/音效/音乐的分类与分别处理策略

不适用。无语音/音效/音乐的分类与差异化处理。

## 训练配合

### 多阶段训练课程与数据课程调度（阶段划分依据：分辨率/时长/质量分/模态；低清→高清、图像→视频、短→长）

两个项目都采用清晰的多阶段渐进式课程，且**数据分层与训练阶段严格绑定**，这是其低成本策略的核心：
【Open-Sora 2.0（三阶段，与数据金字塔一一对应）】阶段一：70M 视频，256px，纯 T2V，85k 迭代，224 卡；阶段二：10M 视频，256px，T2V+I2V 混合，13k 迭代，192 卡；阶段三：5M 视频，768px，T2V+I2V，13k 迭代，192 卡。课程维度为「分辨率（256→768）× 数据质量（松→严）× 任务（T2V→加入 I2V）」三者同步收紧。报告强调低分辨率阶段用于低成本学习运动模式，高分辨率阶段用于提升感知质量，并特别指出**图生视频（I2V）微调比纯文生视频更省算力**——用 I2V 承接高分辨率阶段是其压缩成本的关键决策。
【Open-Sora 1.2】阶段一 30k 步：Webvid-10M，240p/360p，2–16 秒；阶段二 23k 步：Panda-70M 美学分>4.5 子集（41k 小时），360p/480p；阶段三 15k 步：约 2M 高质量片段（5k 小时），720p/1080p，配合 25% 掩码训练。全程 68k 步。
【Open-Sora Plan v1.3（T2V）】阶段一 100k 步：纯图像 1×320×320（SAM），用于把 3D 稠密注意力平滑迁移到稀疏注意力；阶段二 300k 步：图像-视频联合，最高 93×320×320，使用**未过滤**的 Panda70M（约 19M）；阶段三 30k 步：仅用**过滤后**的 Panda70M，固定 93×352×640，学习率从 2e-5 降到 1e-5。课程维度为「图像→视频、低清→高清、粗数据→精数据」。
【Open-Sora Plan v1.5】图像 4 阶段（256²→384²→288×512）+ 视频 5 阶段（57 帧→121×576×1024），最后阶段用高质量子集，全程在 Ascend 910B NPU 上训练。
共同范式：**低质量大数据打底 + 高质量小数据收尾**，且「过滤严格度」本身就是课程调度的一个显式维度（尤其 Open-Sora Plan 阶段二用未过滤数据、阶段三换过滤数据的对比最为直白）。

### 数据配比随训练阶段的变化（预训练/退火annealing/SFT高质量子集）

【Open-Sora 2.0】阶段间数据量比例 70M : 10M : 5M（14 : 2 : 1），后两阶段同时引入 I2V 任务与 T2V 混合训练（报告未给出 T2V/I2V 的具体配比数字）。无独立的退火（annealing）阶段命名，但阶段三（5M 高质量 + 768px）实质承担了退火/SFT 的角色。
【Open-Sora 1.2】阶段三对高质量数据采用 **25% 掩码训练**（mask ratio），使模型在高清精调阶段同时获得图生视频与视频续写能力。
【Open-Sora Plan v1.3】T2V 侧为「未过滤数据（阶段二）→ 过滤数据（阶段三）」的两段式；I2V 侧的任务配比调度非常细：第一阶段 50M 样本拆成 7 个渐进子步，从 continuation/random mask（帧保留率 50% → 25% → 12.5%）逐步过渡到以 I2V（40%）与 Transition（40%）为主，第二阶段 15M 高质量数据维持该配比并把分辨率提到 93×640×640。这是本调研中**任务配比随阶段变化描述得最细的开源案例**。
【Open-Sora Plan v1.5】图像侧 1.1B 全量无过滤，视频侧 40M 过滤后数据，最后阶段切换到高质量子集。

### 后训练数据（SFT精选集规模与筛选标准、偏好对数量与标注方式、reward model训练数据）

两个项目都**没有现代意义上的后训练（RLHF/DPO/reward model）**——不存在偏好对数据、不存在 reward model 训练集、未做人类偏好对齐。所谓「后训练」实际只是高质量数据上的继续预训练/精调：
- Open-Sora 2.0：阶段三的 5M 高分辨率精选数据 + Qwen2.5-Max 重新打标，即为其 SFT 等价物；筛选标准即数据金字塔顶层的最严阈值组合（未给出具体阈值数值）。
- Open-Sora 1.2：阶段三的 2M 片段 / 5k 小时高质量集，来源为 MiraData、Vript（GPT 打标）及其他免费素材站（PLLaVA 打标）。
- Open-Sora Plan：T2V 阶段三的过滤后 Panda70M（约 19M / 27% 保留）+ I2V 阶段二的 15M 高质量集。
- 唯一的「独立后训练模块」是 Open-Sora Plan 的 **prompt refiner**：19,500 条 caption、LLaMA-3.1-8B LoRA（rank 64，batch 32，1 epoch，单 H100 30 分钟），但它微调的是文本模型而非视频模型。
- 人类偏好数据仅用于**评测**（Open-Sora 2.0 的 100 prompt 人工胜率评测），未回流训练。

### 数据处理基础设施与吞吐（NeMo Curator/Data-Juicer/自研，GPU加速比，处理规模，成本） ⚠️

【框架】两者均**未使用 NeMo Curator 或 Data-Juicer 等成熟数据处理框架**，而是自研轻量 pipeline：Open-Sora 基于 CSV meta 文件 + torchrun 多卡并行的脚本组合（`torchrun --nproc_per_node 8 -m tools.scoring.aesthetic.inference meta.csv --bs 1024`），各步骤产出新列并由 `tools.datasets.datautil` 做合并与过滤；数据流以「一张不断加宽的 meta CSV」为中心，设计极简、易于复现。Open-Sora Plan 未公布数据处理的工程实现细节。
【吞吐（Open-Sora 官方给出的唯一实测数字）】美学打分在**单张 H800 上约 1000 视频/秒**，支持多卡线性加速；OCR、光流等重模块未给出吞吐数字。这是本调研中少数给出具体单卡数据处理吞吐的公开材料。
【成本 —— 本条目最具标志性的数据】Open-Sora 2.0 逐阶段训练成本全部公开：阶段一 224 卡 × 85k 迭代 = **$107.5k**；阶段二 192 卡 × 13k 迭代 = **$18.4k**；阶段三 192 卡 × 13k 迭代 = **$73.7k**；**总计 $199.6k（约 20 万美元）**，训练出 11B 模型。Open-Sora 1.2 的训练消耗为全程 68k 步、约 **35,000 H100 GPU 小时**。注意：这些成本口径只含**训练**，两个项目均**未披露数据处理环节本身的算力成本**——考虑到需对数千万条视频跑美学、光流/VMAF、OCR、模糊检测与 VLM 打标（尤其 VLM 打标成本极高），数据侧成本很可能是被隐藏的大头，「$200k 训出商业级模型」的口径应据此理解。[不确定]

## 效果对比

### 数据策略消融的量化影响（区分：过滤严格度ablation / caption密度风格ablation / 数据配比ablation，及对应评测指标） ⚠️

两个项目的报告中**均无严格意义上的数据消融实验**（没有「同架构同算力下，A 数据策略 vs B 数据策略」的对照表），这是其技术报告相对学术规范的主要弱点。可用的定量/定性证据如下：
【过滤严格度维度】Open-Sora Plan v1.3 提供了最接近消融的设计证据：训练阶段二用**未过滤** Panda70M、阶段三换成**过滤后**数据，作者报告阶段二在 100k 步左右即接近收敛，随后依靠阶段三的过滤数据（仅 30k 步、学习率降至 1e-5）实现质量跃升——间接说明「大量粗数据负责学分布，少量精数据负责提质量」。但未给出两种设置的指标对照。另外论文对阈值选择给出了理据性说明（美学 4.75 是因为兼具滤文字效果；LPIPS 上界 0.3 是因为超过即出现抖动闪烁；这些结论经 2000 条人工抽检验证），属于「阈值合理性论证」而非消融。
【caption 密度/风格维度】Open-Sora Plan 明确论证了训练 caption（密集长文）与用户 prompt（<10 词）的分布错配会损害生成效果，并以此为动机构建 prompt refiner——这是对 caption 风格影响的定性论证，但同样未给出 refiner 开/关的量化指标对比。Open-Sora 2.0 在高清阶段把 caption 模型从 LLaVA-Video 换成 Qwen2.5-Max，理由是「更准确、语义对齐更好」，同样无对照实验。
【数据配比维度】无任何配比消融。
【整体效果指标】Open-Sora 2.0 的 VBench 结果显示其与 Sora 的差距从 Open-Sora 1.2 的 **4.52% 缩小到 0.69%**，并超过 CogVideoX1.5-5B 与 HunyuanVideo；100 条 prompt 的人工评测中，在视觉质量/prompt 遵循/运动质量三项中至少两项优于 HunyuanVideo、Runway Gen-3 Alpha、Step-Video-T2V、Luma Ray2。但这是端到端结果，无法归因到数据策略单项。[不确定]

### 质量vs数量的证据（小而精数据超越大而杂的案例）

这是 Open-Sora 系列最有说服力的整体性证据，尽管缺少严格对照实验：
1. **Open-Sora 2.0 以 $199.6k、11B 参数、且高清阶段只用 5M 视频，达到与 11B HunyuanVideo、30B Step-Video-T2V 相当的 VBench 与人工偏好水平**，训练成本比同级模型低 5–10 倍。其核心机制正是「分层数据金字塔 + 分级打标 + 渐进课程」：把绝大部分算力花在低分辨率的粗筛数据上，只用最严过滤的 5M 数据（占底层 70M 的 7%）做高清收尾。
2. **Open-Sora Plan v1.3 把 Panda70M 砍到 27% 保留率**（或按论文漏斗表口径 42%）后仅用 30k 步精调即显著提升质量，且 v1.5 用 40M 视频、8B 模型达到与 HunyuanVideo 相当的水平，同样是「筛得狠、训得少」的路线。
3. **Open-Sora 1.2** 从 30M 片段 / 80k 小时中筛出 2M 片段 / 5k 小时（约 6.7%）作为高清阶段数据。
三个版本一致地把最终高质量子集控制在原始池的 **7% 量级**，且这一比例是跨项目、跨版本收敛的经验值，本身即是「质优于量」的强经验证据。反面注解：缺少「若用 5M 随机未筛数据做阶段三会怎样」的对照，因此严格来说是**工程实践证据而非科学证据**。

### 训练数据domain分布与评测基准类目体系的对齐关系（如VABench七大类） ⚠️

两个项目的训练数据都**没有建立 domain 类目体系**，因此谈不上与评测基准类目的对齐。评测侧主要使用 **VBench**（Open-Sora 2.0 报告 Table 13 给出完整 VBench 各维度分数，并以「与 Sora 的差距从 4.52% 缩小到 0.69%」为核心指标）与自建的 100 条 prompt 人工偏好评测（三维度：视觉质量 / prompt 遵循 / 运动质量）。VBench 的十六个细分维度（主体一致性、背景一致性、时序闪烁、运动平滑度、动态程度、美学质量、成像质量、物体类别、多物体、人物动作、色彩、空间关系、场景、外观风格、时序风格、整体一致性）与训练数据的筛选维度存在**部分间接对应**：美学质量↔LAION 美学分过滤、成像质量↔DOVER/Laplacian/bpp 过滤、动态程度与运动平滑度↔光流/VMAF/LPIPS 双向运动过滤、时序闪烁↔LPIPS 上界剔除闪烁片段。也就是说筛选器设计事实上是围绕 VBench 的**低层画质与运动维度**展开的，但对 VBench 的**语义类目维度**（物体类别、多物体、人物动作、空间关系、场景）没有任何数据侧的配比或覆盖度保障——这与前述 domain_distribution 缺位是同一问题的两面。与 VABench 这类音视频基准无任何关系（无音频能力）。[不确定]

## 其他信息

### summary_note

核心价值定位：Open-Sora 与 Open-Sora Plan 是本调研中**数据处理方法可复现性最高**的两个样本，与 Sora 2 / Veo 3 等「能力最强但披露为零」的闭源模型形成完美互补。具体可复用要点：(1) Open-Sora v1.2.0 分支的 tools/ 目录提供了**代码级、带阈值、带吞吐数字的完整视频数据 pipeline**（PySceneDetect 切镜 → CLIP+MLP 美学 → UniMatch 光流 → DBNet++ OCR → PLLaVA-13B 打标 → CLIP matching → 相机运动检测 → datautil 过滤），是搭建自有 pipeline 最直接的起点；(2) Open-Sora Plan 论文提供了**全行业罕见的逐级保留率表**（100%→97%→89%→89%→49%→44%→42%，其中美学过滤单级淘汰 40 个百分点），以及 27% 最终保留率这一与 Apollo 高度一致的经验值；(3) Open-Sora 2.0 的**分层数据金字塔 + 分级打标（低清用开源 LLaVA-Video、高清用 Qwen2.5-Max）+ 逐阶段成本表（$107.5k/$18.4k/$73.7k = $199.6k）**是低成本训练配方的完整披露；(4) 两者共同的「把美学分/运动分/相机运动追加进 caption 作为可控条件」是被广泛复用的工程设计。
主要局限（引用时须注意）：① 全系列**不支持音频**，音视频对齐相关的 8 个字段完全不适用，不能作为 AV 模型的数据处理参考；② **无去重、无安全过滤、无 VLM-as-judge**，过滤全靠浅层标量打分器 + 阈值，代表的是 2024–2025 年的技术水位，与 2026 年「大模型语义判断」的趋势有明显代差；③ **无 domain 配比与概念均衡**，数据分布被动继承自公开数据集；④ **无严格数据消融实验**，「质优于量」是工程实践证据而非对照实验证据；⑤ Open-Sora 2.0 的 main 分支**已移除数据处理代码**，报告中描述的新 pipeline（PaddleOCR / VMAF / Laplacian）无对应开源实现，「完整开源数据处理代码」这一评价严格来说适用于 1.x 版本；⑥ 数据来源全部为公开数据集（Panda-70M / Webvid / LAION 等），版权状态存在争议，且 $200k 成本口径不含数据处理与打标算力。

## 不确定字段

以下字段的调研信息部分不确定（⚠️ 标注来源）：

- openness
- data_scale
- data_sources
- provenance_licensing
- domain_distribution
- funnel_retention_rate
- deduplication
- safety_filtering
- data_infra_throughput
- data_ablation
- benchmark_taxonomy_alignment
