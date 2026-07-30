# HunyuanVideo（混元视频，2024原版13B）+ HunyuanVideo 1.5（混元视频1.5，8.3B）

> 主题: 视频生成模型（含音视频同时生成）的数据处理：数据清洗流程、数据分布、打标方式

[← 返回首页](../index.md)

**目录**: [基本信息](#基本信息) · [数据规模与分布](#数据规模与分布) · [清洗流程](#清洗流程) · [打标方式](#打标方式) · [音视频对齐](#音视频对齐) · [训练配合](#训练配合) · [效果对比](#效果对比)

## 基本信息

### 名称

HunyuanVideo（混元视频，2024原版13B）+ HunyuanVideo 1.5（混元视频1.5，8.3B）

### 发布机构/公司

腾讯混元基础模型团队（Tencent Hunyuan Foundation Model Team）

### 发布时间（技术报告/论文/开源时间）

HunyuanVideo：2024年12月3日发布技术报告（arXiv:2412.03603，后有v2修订）并同步开源权重与推理代码；2025年3月发布 HunyuanVideo-I2V 图生视频版本。HunyuanVideo 1.5：2025年11月21日开源，技术报告 arXiv:2511.18870（2025年11月）。

### 类型（模型/数据集/工具链/评测基准）

模型（开源视频生成基础模型），并附带完整的开源代码与推理框架。原版13B为当时参数量最大的开源视频生成模型；1.5为8.3B轻量化版本，主打消费级显卡（约14GB显存）可跑。二者均为纯视频生成模型，不是数据集、不是评测基准。

### 开源程度（权重/代码/数据/pipeline各自是否开源）

属于「权重+代码开源、数据与数据pipeline不开源」的典型模式。
【权重】开源。HunyuanVideo 13B（DiT主干+3D VAE+文本编码器）在 Hugging Face（tencent/HunyuanVideo、tencent/HunyuanVideo-I2V）发布；HunyuanVideo 1.5（8.3B，含T2V/I2V/超分模块）在 GitHub Tencent-Hunyuan/HunyuanVideo-1.5 与 Hugging Face 发布。
【代码】开源。推理代码、并行推理、量化、LoRA、ComfyUI/Diffusers 集成均提供；训练代码未完整开放。
【许可证】腾讯混元社区许可协议（Tencent Hunyuan Community License），非标准OSI开源协议，对欧盟等地区与月活用户数有使用限制。
【数据】不开源。训练数据集本身、各级过滤后的清单、caption数据均未公开。
【pipeline】方法论层面披露详尽（尤其原版对分层过滤漏斗、结构化caption schema、镜头运动分类器的描述在同期闭源模型中属最详细一档），但过滤器代码、模型权重（如自研VideoCLIP、blur检测模型、YOLOX-like检测器、caption VLM）均未开源，无法直接复现。

### 是否支持音视频同时生成，以及实现方式（原生联合/级联/MoE融合）

不支持。HunyuanVideo 与 HunyuanVideo 1.5 均为纯视觉视频生成模型，输出无音轨。1.5 技术报告全文未出现任何音频生成相关内容，数据侧也不涉及音轨处理。
腾讯的音频能力由独立模型承担而非联合生成：HunyuanVideo-Foley（2025年8月，视频到音频/Foley生成，基于约10万小时TV2A数据集）、HunyuanVideo-Avatar（音频驱动数字人）等，属于「级联/外挂」形态——先由 HunyuanVideo 生成画面，再由 Foley 模型配音，而非原生联合去噪。因此本条目在音视频联合生成维度上不构成参考样本，本调研中所有音频相关字段（audio_category_distribution、av_sync_detection、sync_metric_and_threshold、temporal_vs_semantic_sync、audio_quality_filtering、audio_type_handling、joint_av_caption_schema、dialogue_transcription_attributes）均不适用。

### 调研信息来源列表（论文/技术报告/官方文档/新闻的URL，标注每条来源的性质：官方一手/同团队旁证/第三方报道）

- 【官方一手】HunyuanVideo: A Systematic Framework For Large Video Generative Models, 腾讯混元, arXiv:2412.03603（含 Section 3 数据预处理/分层过滤漏斗、结构化caption、14类镜头运动分类器、~1M SFT人工标注）: https://arxiv.org/abs/2412.03603
- 【官方一手】HunyuanVideo 论文 HTML v1 全文（数据章节可检索）: https://arxiv.org/html/2412.03603v1
- 【官方一手】HunyuanVideo 论文 HTML v2 全文: https://arxiv.org/html/2412.03603v2
- 【官方一手】HunyuanVideo 1.5 Technical Report, 腾讯混元, arXiv:2511.18870, 2025-11（含 >10M小时原始视频、800M片段、Table 2 八阶段训练数据表、三套caption模型、OPA-DPO、CT/SFT/RLHF）: https://arxiv.org/abs/2511.18870
- 【官方一手】HunyuanVideo 1.5 技术报告 HTML 全文: https://arxiv.org/html/2511.18870v1
- 【官方一手】HunyuanVideo 1.5 技术报告 PDF: https://arxiv.org/pdf/2511.18870
- 【官方一手】HunyuanVideo GitHub 开源仓库（权重、推理代码、许可证）: https://github.com/Tencent-Hunyuan/HunyuanVideo
- 【官方一手】HunyuanVideo Hugging Face 模型卡: https://huggingface.co/tencent/HunyuanVideo
- 【官方一手】HunyuanVideo-I2V Hugging Face 模型卡（2025-03 图生视频版本）: https://huggingface.co/tencent/HunyuanVideo-I2V
- 【同团队旁证】HunyuanVideo-Foley（腾讯视频到音频/Foley 生成模型，约10万小时TV2A数据集，说明腾讯的音频能力为独立级联模型而非与视频联合生成）: https://github.com/Tencent-Hunyuan/HunyuanVideo-Foley
- 【第三方】alphaXiv HunyuanVideo 1.5 论文解读页: https://www.alphaxiv.org/overview/2511.18870v1
- 【第三方】Emergent Mind: HunyuanVideo 1.5 开源视频合成模型综述: https://www.emergentmind.com/topics/hunyuanvideo-1-5
- 【第三方】Hugging Face Papers: HunyuanVideo 论文页与社区讨论: https://huggingface.co/papers/2412.03603
- 【第三方】ResearchGate: HunyuanVideo 1.5 Technical Report 条目: https://www.researchgate.net/publication/397934115_HunyuanVideo_15_Technical_Report

## 数据规模与分布

### 训练数据量级（视频条数/小时数/token数，预训练与SFT分开）

【HunyuanVideo 原版（2024）】总量口径披露不完整：论文明确给出的只有 SFT 阶段约 100 万（~1M）人工精选样本，以及图像侧「数十亿（billions）级」用于第一阶段T2I预训练、「数亿（hundreds of millions）级」用于第二阶段。视频侧的原始池规模与各分辨率档（256p/360p/540p/720p）的绝对条数/小时数未公布，只给出相对保留比例（每级保留上一级的1/2到1/5）。
【HunyuanVideo 1.5（2025）】口径显著更完整，是本条目最有价值的定量披露：
- 原始视频池：超过 1000 万小时（>10M hours）原始视频；
- 经切分与过滤后：约 8 亿（800M）高质量视频片段进入预训练；
- 后续各阶段逐级收缩：480p 阶段 2 亿（200M）、720p/16fps 阶段 1 亿（100M）、720p/24fps 阶段 1 亿（100M）；
- CT（继续训练）阶段：T2V 与 I2V 各 100 万（1M）高质量片段；
- 图像侧：从超过 100 亿（>10B）图像池中筛出 50 亿（5B）用于第一阶段 256p T2I 预训练，10 亿（1B）用于第二阶段 512p；
- 超分模块训练数据：100 万高质量视频片段（1K–4K 分辨率）+ 高分辨率图像。
SFT 阶段与 RLHF 阶段的精确样本数 1.5 报告未给出具体数字（仅描述筛选标准）。

### 数据来源构成（自有/公开数据集/网络爬取/授权采购/合成数据） ⚠️

两代均只作定性描述，未给出来源构成比例。原版称原始数据池覆盖人物、动物、植物、风景、载具、物体、建筑、动画等多个domain，来源未指明（隐含为网络爬取+自有/授权素材库混合）。1.5 报告表述为「视频来自多种渠道（a variety of channels），确保在内容、拍摄手法、镜头运动、风格、场景上的全面覆盖」，同样未区分自有/公开数据集/爬取/采购/合成。两代均未使用合成视频数据作为主训练语料（未提及）。图像侧 1.5 明确复用 HunyuanImage-3.0 的数据获取与处理pipeline。[不确定]

### 数据合规与溯源（授权数据占比、rights-cleared数据集、C2PA等） ⚠️

未披露。两份技术报告均未涉及授权数据占比、rights-cleared 数据集清单、版权处理策略、C2PA 等溯源标准。原版仅在过滤环节提到用 YOLOX-like 视觉模型剔除「水印、边框、logo 以及某些敏感信息（sensitive information）」，这更接近画面清洁而非版权合规。开源侧的合规约束体现在模型许可证（腾讯混元社区许可协议限制欧盟等地区使用、限制月活超1亿的商用），而非训练数据溯源。[不确定]

### 片段时长分布与切分策略

【原版】未给出时长直方图。切分策略明确：用 PySceneDetect 将原始视频切为单镜头（single-shot）片段，再用 OpenCV Laplacian 算子在片段内选取清晰帧作为起始帧。训练时长以帧数体现——从 65 帧（256×256×65）到 129 帧（720×1280×129），并支持 1–129 帧的多帧数分桶。
【1.5】明确得多：所有训练片段统一切分为 2–10 秒；预训练阶段 III–V 为 16fps、2–10s，阶段 VI 起升到 24fps、2–10s，CT/SFT 阶段维持 24fps、2–10s。即 1.5 走的是「短clip为主、以帧率而非时长做课程递进」的路线，未训练超过10秒的长片段。

### 分辨率/宽高比分布与分桶策略

【原版】分辨率本身即是分层过滤漏斗的分层轴：构造 256p → 360p → 540p → 720p 四档递进数据集 + 一档 SFT 多尺度数据集，越高档过滤越严。训练侧采用 bucketing 策略同时支持多分辨率与多宽高比（并支持 1–129 可变帧数），从而可生成任意宽高比视频。各宽高比的具体占比未公布。
【1.5】阶段化更清晰：图像 256p → 512p；视频 256p → 480p → 720p，CT/SFT 阶段同时训练 480p 与 720p；另有独立超分模块用 1K–4K 数据训练，把 480p/720p 基座输出提升到 1080p 级别。宽高比分桶细节未披露占比。

### 类别/domain分布与配比策略（人物、动作、场景、风格等比例控制与概念均衡） ⚠️

这是原版 HunyuanVideo 最有方法论价值的一点，也是同期少数显式做「概念均衡」的开源工作：
【原版】(1) 原始数据池按 domain 划分，明确列举人物（people）、动物（animals）、植物（plants）、风景（landscapes）、载具（vehicles）、物体（objects）、建筑（buildings）、动画（animation）等类别，覆盖广度作为数据构建目标；(2) 用自研内部 VideoCLIP 模型对片段抽 embedding，先按余弦距离做语义去重，再对 embedding 跑 k-means 得到约 1 万（~10K）个概念中心（concept centroids），基于这些中心做 concept resampling and balancing（概念重采样与均衡）——即用聚类中心作为概念代理，对过密概念下采样、对稀疏概念上采样，抑制长尾失衡。这套「VideoCLIP embedding + 10K 聚类中心 + 重采样」是原版数据处理的标志性设计。(3) 各 domain 的最终占比数字未公布。
【1.5】报告未重复描述概念均衡机制，只强调渠道多样性覆盖「内容、拍摄手法、镜头运动、风格、场景」，是否延续 10K 概念中心重采样未说明。1.5 在 RLHF 阶段有类目化痕迹：I2V RLHF prompt 覆盖「100+ 类别」，T2V DPO 使用在运动/场景/主体三个维度上做过平衡的 prompt 集。[不确定]

### 音频类别分布与配比（语音/音效foley/音乐/环境音/静音的比例及控制策略）——AV模型独有维度 ⚠️

不适用。HunyuanVideo 与 HunyuanVideo 1.5 均不生成音频，训练数据不含音轨处理，两份报告均无音频类别配比内容。若需腾讯体系内的音频数据方法，应转向 HunyuanVideo-Foley（视频到音频，约10万小时TV2A数据集，含语音/音效/音乐的多标签平衡策略）等独立工作。[不确定]

### 叙事结构分布（单镜头vs多镜头、平均clip时长、镜头数分布、是否含原生音轨）

两代均明确采用「单镜头（single-shot）」数据范式，不做多镜头叙事训练：
【原版】用 PySceneDetect 切分为单镜头片段；另用 Transnet v2 + PySceneDetect 双路提供场景边界信息以提高切分可靠性；dense description 字段中会记录场景转换（scene transitions）与镜头运动。
【1.5】更进一步：在 PySceneDetect 与自研算子切分之后，额外接一个专门的「转场分类器（transition classifier）」，把仍残留转场特效（渐变、叠化等）的片段整体剔除，确保每个训练clip是干净的单镜头。平均clip时长即 2–10 秒区间，镜头数分布无意义（均为1）。原生音轨：无（数据侧不保留音轨）。

### 语言/口音分布（多语种唇同步能力的数据基础） ⚠️

不适用于唇同步（无音频生成）。文本侧：原版使用 MLLM 作为文本编码器并在 caption 生成与 prompt 改写中考虑多语言——用 Hunyuan-Large 大语言模型对用户 prompt 做改写（prompt rewrite），功能包括「prompt结构标准化、复杂术语简化、多语言适配（multilingual adaptation）」以对齐训练 caption 的分布；实际支持中英文 prompt。训练 caption 的语种构成比例未公布。1.5 未描述 prompt 改写与语种分布。[不确定]

## 清洗流程

### 清洗漏斗整体结构（几级过滤、各级顺序）

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

### 漏斗定量保留率（每级过滤的输入/输出量与最终保留率，如Apollo 27%）

【原版】只有相对口径，无绝对数字：论文原文称各级过滤「A large portion of data will be removed at each stage, ranging from half to one-fifth of the data from the previous stage」——即每一级只保留上一级的 1/2 到 1/5（换算为保留率 20%–50%，淘汰率 50%–80%）。若四级串联按此区间估算，256p→SFT 的整体保留率大致落在 0.16%–6.25% 的宽区间，论文未给出终值。
【1.5】有绝对数字可算，是同类工作中较罕见的定量漏斗：
· 原始池 >1000万小时视频 → 切分+过滤后 8 亿片段（按平均6秒计约133万小时，粗估整体保留率约 13%，此换算为本调研推算而非论文原文）；
· 8亿（256p）→ 2亿（480p）：保留 25%；
· 2亿（480p）→ 1亿（720p/16fps）：保留 50%；
· 1亿 → 1亿（720p 16fps→24fps）：保留 100%（同规模换帧率）；
· 1亿 → 100万（CT/高质量档）：保留 1%；
· 图像侧：>100亿 → 50亿（保留 <50%）→ 10亿（二阶段保留 20%）。
从 8 亿到 100 万的端到端保留率为 0.125%，是本调研中定量最完整的漏斗之一，可直接与 Apollo 的 27% 等口径对照（注意口径不同：Apollo 为单级过滤保留率，此处为多级串联终值）。

### 镜头切分方法（PySceneDetect/自研模型/shot-aware splitting）

【原版】PySceneDetect 做主切分（切为单镜头 clip）；Transnet v2 与 PySceneDetect 双路提供场景边界信息用于交叉验证；切分后用 OpenCV Laplacian 算子在片段内挑选清晰帧作为 clip 起始帧，避免以转场模糊帧开头。
【1.5】PySceneDetect + 自研算子（custom operators）联合检测场景边界，统一切为 2–10 秒 clip；关键增量是在切分之后再接一个专门训练的「转场分类器（transition classifier）」做二次清洗，剔除仍含渐变/叠化等转场特效的片段——说明团队认为纯阈值型 shot detection 会漏检软转场，需要模型级补刀。

### 质量过滤（美学评分、清晰度、OCR文字过滤、黑边/水印/logo检测）

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

### 运动过滤（光流/运动分数阈值、静态与抖动剔除） ⚠️

【原版】用光流估计（optical flow estimation）计算运动幅度，剔除静止（static）与慢动作（slow-motion）视频；SFT 人工标注环节还专门对「运动速度（motion speed）、动作完整性（action integrity）、运动模糊（motion blur）」三项做人工判定，即运动质量同时有自动与人工两道关。
【1.5】在基础过滤级中剔除「静止或低运动场景（static or low-motion scenes）」，未说明是否仍用光流；SFT 阶段的筛选标准之一为「运动流畅性（motion smoothness）」。两代均无阈值数值。[不确定]

### 去重方法（精确去重与基于embedding的语义去重分别记录） ⚠️

【原版】做了语义级去重且方法明确：用内部自研 VideoCLIP 模型抽取视频 embedding，按余弦距离（cosine distance）判定重复并去重；同一套 embedding 随后跑 k-means 得到约 1 万个概念中心用于概念重采样与均衡。未提及哈希级精确去重。
【1.5】仅表述为「basic deduplication and the removal of corrupted files（基础去重与损坏文件剔除）」作为最前置步骤，未说明是哈希去重还是 embedding 语义去重，也未说明是否延续 VideoCLIP 方案。1.5 在去重描述上明显弱于原版。[不确定]

### VLM/LLM作为质检员（多模态大模型质量打分与错配剔除，2026年从浅层打分器转向大模型语义判断的趋势） ⚠️

两代都大量使用模型作为质检员，但形态不同，且都不是「用通用大 VLM 做端到端语义评判」：
【原版】使用的是一组专用小模型/判别模型组成的评审团——Dover（美学+技术双视角）、自研 blur 检测模型、OCR 模型、YOLOX-like 检测模型、光流模型、Transnet v2 场景检测、自研 VideoCLIP（去重与概念均衡）、自研镜头运动分类器（14类）。属于典型的 2024 年「浅层多打分器」范式。真正的大模型只用在打标环节（自研 VLM 生成结构化 caption）与 prompt 改写（Hunyuan-Large）。
【1.5】向「模型化算子」演进但仍未公开使用通用大 VLM 判分：新增了专门训练的「转场分类器」、四维度「综合视觉质量评估算子」、美学打分算子、镜头运动识别模型（clip级+时序级）。最能体现大模型参与数据构建的是 caption 侧：对 caption 模型本身用 OPA-DPO 强化学习做后训练，专门优化「描述丰富度 vs 事实准确性」的权衡以压制幻觉——即用 RL 保证数据标注器的可信度，这是把「模型作为标注员」的可靠性问题当作一等公民来处理，属于该维度上较前沿的做法。是否用 VLM 对 video-caption 错配做剔除，两份报告均未说明。[不确定]

### 安全与合规过滤（NSFW、版权、人脸/隐私） ⚠️

披露很弱。原版仅一句提及用 YOLOX-like 视觉模型剔除「水印、边框、logo 及某些敏感信息（sensitive information）」，未展开何为敏感信息，未提 NSFW 分类器、人脸/隐私保护、版权过滤。1.5 技术报告的数据章节完全未涉及安全过滤。作为中国境内发布的产品，实际生产系统必然存在内容安全审核（模型卡与许可证中有合规使用条款、开源仓库要求遵守当地法律法规），但训练数据侧的安全过滤方法零披露。[不确定]

## 打标方式

### 使用的caption模型（自研VLM/开源模型，模型规模） ⚠️

【原版】自研 VLM（in-house Vision Language Model）作为 caption 模型，用于生成 JSON 结构化 caption；未公布参数规模、架构或训练细节。另有一个独立的自研镜头运动分类器（camera movement classifier）。prompt 侧使用 Hunyuan-Large 大语言模型做用户 prompt 改写。
【1.5】升级为「三模型分工」的打标体系，是相对原版最大的结构性改进：
1. 图像 caption 模型——沿用 HunyuanImage-3.0 的方法；
2. 视频 caption 模型——产出高度结构化的多组件描述；
3. 图生视频指令式 caption 模型（Image-to-Video Instructional Captioning）——这是新增模块，不描述整段画面内容，而是专门描述「相对首帧的时间演化/变化」，涵盖前景主体与背景环境的变化，使 I2V 训练的文本条件从「描述性」转为「指令性」，与 I2V 推理时用户实际输入（给一张图+说想让它怎么动）的分布对齐；
另加一个镜头运动识别模型（clip级 + 时序级双粒度）。
三个 caption 模型的名称、参数量、基座均未披露；只披露了训练手段：用 OPA-DPO（一种针对多模态幻觉的偏好优化方法）做 RL 后训练以抑制幻觉。[不确定]

### caption密度与结构化程度（短/长/密集描述、结构化字段如镜头运动、风格标签）

【原版】密集且强结构化，是 2024 年开源视频模型中 caption schema 最完整的方案之一。采用 JSON 格式，包含七个维度字段：
1. Short Description（短描述，概括主要内容）；
2. Dense Description（密集描述，详细场景元素，且明确包含场景转换与镜头运动的叙述）；
3. Background（背景/环境）；
4. Style（风格：纪录片/电影感/写实/科幻等）；
5. Shot Type（景别与机位：航拍/特写/中景/远景等）；
6. Lighting（光照条件）；
7. Atmosphere（氛围情绪：温馨/紧张/神秘等）。
此外还有元数据派生字段（source tags、quality tags）。镜头运动分类器的高置信度预测结果也会合并进 JSON caption，从而赋予模型运镜可控能力。
训练时采用「dropout 机制 + 排列组合（permutation and combination）策略」，从这套 JSON 字段中随机丢弃/重组字段，合成出长度与句式多样的 caption，避免模型过拟合到单一 caption 模板——这一点很关键，直接决定了推理时短 prompt 也能work。
【1.5】延续并细化「多层级文本叙述 + 电影美学属性集」的结构：cinematic/aesthetic 属性明确列举 shot type（景别）、shot angle（机位角度）、composition（构图）、lighting（光照）、style（风格）、color palette（色调/调色板）、atmosphere（氛围）——相对原版新增了 shot angle 与 color palette 两个字段；镜头运动以自然语言形式写入 caption，且区分 clip 级（整段的主导运镜）与 sequential 级（随时间变化的运镜序列）。1.5 未复述 dropout+排列组合策略。

### 音视频联合caption结构（是否同时覆盖视觉+听觉轨道、是否分流为独立字段，如LTX-2全音景描述、Script-a-Video factorized streams、Foley-Omni三字段） ⚠️

不适用。两代模型均无音频模态，caption 只覆盖视觉轨道，不存在视觉+听觉双轨 caption 结构。可作为「纯视觉结构化 caption」的对照基线，与 LTX-2 全音景描述、Script-a-Video factorized streams、Foley-Omni 三字段等 AV 方案形成对比：HunyuanVideo 的七字段/多属性 schema 在视觉侧的字段化程度已经很高，AV 模型的音轨字段可视为在此之上的正交扩展。[不确定]

### 对白转写与说话人属性标注（ASR转写+说话人身份/语言/口音/情绪） ⚠️

不适用。无音轨处理，无 ASR 转写，无说话人身份/语言/口音/情绪标注。腾讯体系内相关能力在 HunyuanVideo-Avatar（音频驱动数字人）等独立模型中，不在本条目范围。[不确定]

### 几何与结构化标注（相机参数、深度、3D point tracks、动作标注、显式状态）

有限但明确的一项：镜头运动（camera movement）标注。
【原版】训练了一个镜头运动分类器，可预测 14 种镜头运动类型：zoom in、zoom out、pan up、pan down、pan left、pan right、tilt up、tilt down、tilt left、tilt right、around left、around right、static shot、handheld shot。仅高置信度预测被写入 JSON caption。
【1.5】镜头运动识别模型不再限定为固定14类枚举（报告称识别「多种镜头运动类型」但未列举），改为 clip 级 + 时序级双粒度输出，并转写为自然语言融入 caption。
两代均未使用显式相机内外参、深度图、3D point tracks、显式物理状态或动作骨架标注——镜头运动是以离散标签/自然语言而非几何参数的形式表达的。这与 Seedance、Movie Gen 等引入更强几何监督的路线不同。

### 合成数据构造（受控扰动/编辑构造训练对，如InstructAV2AV） ⚠️

两代均未使用合成数据构造训练对，报告中无受控扰动/编辑式配对数据（如 InstructAV2AV 类）的描述。唯一带「构造」性质的是 1.5 的 T2V DPO 阶段：对同一 prompt 用模型自身采样 N 个候选视频，两两组成非重复的偏好对（non-repetitive pairs），再由人工 GSB 标注胜负——属于模型自采样构造偏好数据，而非训练语料合成。此外原版的 caption dropout+排列组合可视为文本侧的数据增广。[不确定]

### 人工介入程度（人工标注、人工质检、模型初筛+人工复核）

两代都把人工放在漏斗最末端（模型初筛 + 人工精选）而非全程标注：
【原版】SFT 数据集约 100 万样本全部经人工标注筛选（manually annotated），标注员按两大类共七项维度评判：美学侧——色彩和谐（color harmony）、光照质量（lighting）、主体突出（object emphasis）、空间布局（spatial layout）；运动侧——运动速度（motion speed）、动作完整性（action integrity）、运动模糊（motion blur）。目标是选出「视觉美观且运动细节丰富」的片段。
【1.5】人工介入分布在三处：(1) SFT 数据集最终由人工标注（manual annotation）构建；(2) I2V RLHF 的 prompt 与配图经人工核验图文一致性；(3) T2V DPO 的偏好对由人工做 GSB（Good/Same/Bad）标注；(4) 评测环节动用「100+ 名专业评估人员」对 300 条文本 prompt 与 300 张图像样本做 GSB 人评。未披露标注团队规模与人时成本。

## 音视频对齐

### 音视频同步检测方法（口型同步、事件对齐） ⚠️

不适用。无音频模态，训练数据不含音轨，因而不存在音视频同步检测环节。[不确定]

### 同步检测具体指标与阈值（SyncNet/Synchformer/LSE/自研，阈值数值，如MOVA LSE-D≤9.5且LSE-C≥4.5、SkyReels-V4 SyncNet |offset|≤3∧conf>1.5） ⚠️

不适用。未使用 SyncNet、AV-align 或任何同步指标，无阈值。[不确定]

### 时序同步vs语义同步的分离处理（时间对齐与内容语义匹配作为两个独立过滤条件） ⚠️

不适用。无音频，不存在时序同步与语义同步的分离处理。视觉侧的类比是：1.5 将「基础规则过滤（结构性缺陷）」与「视觉质量/美学打分（感知质量）」分为独立条件，属于同一思路在单模态上的体现。[不确定]

### 音频质量过滤（SNR、静音检测与静音占比阈值、无音轨剔除、画外音源剔除、背景音乐分离） ⚠️

不适用。无音轨处理，无 SNR/静音检测/背景音乐分离等环节。[不确定]

### 语音/音效/音乐的分类与分别处理策略 ⚠️

不适用。无语音/音效/音乐的分类与分别处理。[不确定]

## 训练配合

### 多阶段训练课程与数据课程调度（阶段划分依据：分辨率/时长/质量分/模态；低清→高清、图像→视频、短→长）

两代都是教科书式的「图像→视频、低清→高清、低帧率→高帧率」渐进课程，且数据集分层与训练阶段一一对应（数据档位就是课程台阶）：
【原版】
· 图像预训练两阶段：Stage 1 —— 256px 多宽高比训练；Stage 2 —— 256px 与 512px 混尺度训练（mix-scale）；
· 视频-图像联合训练三阶段：低分辨率短视频（256×256×65 帧）→ 低分辨率长视频 → 高分辨率长视频（720×1280×129 帧）；训练全程保持视频与图像联合训练以防止图像能力退化；VAE 训练中视频:图像混合比为 4:1；
· 最后叠加 SFT（约100万人工精选样本，多尺度）。
【1.5】共八阶段（表2），课程轴同时包含分辨率、帧率与任务类型：
· Stage I：256p 图像，50亿，T2I；
· Stage II：512p 图像，10亿，T2I；
· Stage III：256p / 16fps / 2–10s，8亿，T2V+I2V+T2I 混合训练，任务比例 1:6:3（I2V 占比最高，达60%）；
· Stage IV：480p / 16fps / 2–10s，2亿；
· Stage V：720p / 16fps / 2–10s，1亿；
· Stage VI：720p / 24fps / 2–10s，1亿（此阶段只升帧率、不升分辨率）；
· Stage VII：CT 继续训练，480p/720p / 24fps，100万，专攻 T2V；
· Stage VIII：CT 继续训练，480p/720p / 24fps，100万，专攻 I2V；
· 之后接 SFT → RLHF/DPO。另有独立超分模块用 1K–4K 数据单独训练。
值得注意的差异：原版把长时长作为一个课程轴（65帧→129帧），1.5 全程固定 2–10 秒不做时长递进，改用帧率（16fps→24fps）作为新轴，并把提分辨率的任务外包给独立超分模块——这是 1.5 为压低 8.3B 模型训练成本所做的关键取舍。

### 数据配比随训练阶段的变化（预训练/退火annealing/SFT高质量子集）

【原版】视频与图像全程联合训练（joint image-video training）以防图像先验退化；VAE 训练明确视频:图像 = 4:1；DiT 各阶段的具体视频/图像配比未公布。SFT 阶段切换到约100万人工精选高质量子集。
【1.5】配比披露更具体：预训练 Stage III–VI 采用 T2V : I2V : T2I = 1 : 6 : 3 的任务混合比例（I2V 占绝对主导，反映产品侧图生视频是主用例，也说明 I2V 指令式 caption 是配套设计）；数据量随阶段单调收缩 8亿→2亿→1亿→1亿→100万，即典型的「量退质进」退火式调度：早期用大而杂的低清数据建立世界先验，后期用小而精的高清数据抛光。CT 阶段按任务拆成两个独立分支（T2V 与 I2V 各 100 万），SFT 与 RLHF 也按任务分别执行。

### 后训练数据（SFT精选集规模与筛选标准、偏好对数量与标注方式、reward model训练数据）

【原版】只有 SFT：约 100 万人工精选样本，筛选标准为美学四项（色彩和谐、光照、主体突出、空间布局）+ 运动三项（运动速度、动作完整性、运动模糊）。原版未做 RLHF/DPO。
【1.5】后训练为三段式 CT → SFT → RLHF，且 T2V 与 I2V 全程分开：
· CT（继续训练）：每个任务 100 万高质量片段，480p/720p、24fps；
· SFT：在 CT 数据之上再严格筛选，标准为美学吸引力（aesthetic appeal）、清晰度（clarity）、运动流畅性（motion smoothness）三项，最终数据集经人工标注构建；具体样本数与阈值未公布；
· RLHF/DPO：I2V 侧构建覆盖 100+ 类别的 prompt 集，配图从高美学图像中精选并经人工核验图文一致性；T2V 侧构建规模 O(10K)（万级）的均衡 prompt 集，来源为 LLM 生成 prompt 与训练 caption 混合，在运动/场景/主体三个维度上做类目平衡；对每条 prompt 采样 N 个候选视频组成非重复偏好对，由人工做 GSB 标注得到偏好数据；
· 超分模块：独立用 100 万条 1K–4K 高质量视频片段 + 高分辨率图像训练。
未披露 reward model 的训练数据规模。

### 数据处理基础设施与吞吐（NeMo Curator/Data-Juicer/自研，GPU加速比，处理规模，成本） ⚠️

两代均未披露数据处理基础设施细节。未提及 NeMo Curator、Data-Juicer 或自研数据处理框架的名称、GPU 加速比、吞吐量或处理成本。可反推的规模压力：1.5 需要对超过 1000 万小时原始视频完成切分 + 转场分类 + 四维质量打分 + 美学打分 + 三套 caption 模型推理，这是十亿级 clip 量级的多模型批量推理任务，工程量极大，但论文未着墨。训练侧基础设施略有披露（1.5 使用 SSTA 稀疏注意力等提效手段、8.3B 模型可在约14GB显存消费级显卡上推理），但不属于数据处理吞吐。[不确定]

## 效果对比

### 数据策略消融的量化影响（区分：过滤严格度ablation / caption密度风格ablation / 数据配比ablation，及对应评测指标） ⚠️

两份技术报告均未提供针对数据策略的受控消融实验——既无过滤严格度 ablation（如放宽/收紧美学阈值对指标的影响），也无 caption 密度/风格 ablation（如短 caption vs 密集 caption 的对比），也无数据配比 ablation（如 T2V:I2V:T2I = 1:6:3 与其他比例的对比）。这是本条目最主要的方法论短板：pipeline 描述详尽，但每一步的必要性缺乏量化证据支撑。
两代提供的均为端到端人工对比评测而非消融：原版由 60 名专业评估人员在 1533 条 prompt 上对比 Gen-3、Luma 1.6 等五个闭源模型，在文本对齐、运动质量、视觉质量上综合胜出；1.5 由 100+ 名专业评估人员在 300 条文本 prompt 与 300 张图像上，用 Rating 与 GSB 方法对比 Wan2.2、Kling 2.1、Seedance Pro、Veo3。这些结果无法归因到具体数据策略。[不确定]

### 质量vs数量的证据（小而精数据超越大而杂的案例） ⚠️

无直接的对照实验证据，但两代的漏斗设计本身是「质优于量」立场的强表达，且 1.5 提供了可量化的间接证据：
· 1.5 的数据量从 8 亿单调收缩到 100 万（端到端保留率 0.125%），却用仅 8.3B 参数（约为原版 13B 的 64%）在人评上对标 Kling 2.1、Seedance Pro、Veo3 等更大更贵的模型，团队将此归因于「meticulous data curation（精细的数据策划）」是几个关键要素之一——这是「小模型+精数据可以抗衡大模型」的一个案例级证据，但缺少「同模型不同数据」的受控对比，因此严格意义上不构成实验证明。
· 原版明确表述每级过滤淘汰 50%–80% 数据，并把最贵的人工标注只用在最后 100 万样本上，体现「越靠后越精、投入越集中」的资源分配哲学。
· 值得注意的反向信号：1.5 相对原版把原始池扩大到 1000 万小时以上（显著扩量），说明团队实践是「先扩大候选池、再加严过滤」，即质量提升靠的是更高的淘汰率而非更小的初始池——量与质并非取舍关系。[不确定]

### 训练数据domain分布与评测基准类目体系的对齐关系（如VABench七大类） ⚠️

训练数据 domain 类目体系与评测基准类目未做显式对齐说明。原版列举的数据 domain（人物/动物/植物/风景/载具/物体/建筑/动画）与其人评维度（文本对齐、运动质量、视觉质量）之间无映射关系。1.5 在 RLHF 阶段的类目体系（I2V 的 100+ 类别、T2V 的运动/场景/主体三维度平衡）更接近评测导向的类目划分，但报告未说明其与 VBench、VABench 等公开基准类目体系的对应关系。两代均以人工 GSB 评测为主、未把公开基准的类目体系反向用于训练数据配比。[不确定]

## 其他信息

### summary_note

核心结论：HunyuanVideo 系列是本调研中「纯视觉侧数据处理方法论披露最完整」的开源样本之一，但完全不涉及音频，对音视频联合生成议题无直接参考价值。
两代的价值分工清晰：
(1) HunyuanVideo 原版（2024-12）——贡献的是「分层过滤漏斗 + 结构化 caption schema」这套范式模板：四档分辨率数据集逐级加严（每级保留 1/2 到 1/5）、VideoCLIP embedding 语义去重 + 1万概念中心 k-means 重采样做概念均衡、Dover/blur/光流/OCR/YOLOX-like 多打分器并联、七字段 JSON caption + 14 类镜头运动标签 + caption dropout 与排列组合增广、末端 100 万人工精选 SFT。这套设计被后续大量开源工作直接借鉴。
(2) HunyuanVideo 1.5（2025-11）——贡献的是「定量漏斗 + 打标器可信度治理」：给出了从 >1000万小时原始视频 → 8亿片段 → 2亿 → 1亿 → 100万的完整量化链条（端到端保留率 0.125%），把水印/字幕处理从「整片丢弃」改为「空间裁剪+60%面积下限」，用专门的转场分类器补 shot detection 的漏检，把视觉质量拆成清晰度/细节保留/噪声伪影/动态范围四维算子，并首创性地为 I2V 设计「指令式 caption」（描述相对首帧的时间演化而非静态内容）、用 OPA-DPO 对 caption 模型做 RL 后训练来压制标注幻觉。训练侧配比 T2V:I2V:T2I = 1:6:3 也是少见的显式披露。
主要缺陷：两代都没有任何数据策略的受控消融实验，pipeline 各环节的收益无法量化归因；安全/版权/隐私过滤披露近乎空白；数据处理基础设施与吞吐零披露；1.5 在去重与概念均衡上的描述反而弱于原版（不清楚是简化了还是省略未写）。
若研究目标涉及音视频联合生成的数据构建，本条目仅可作为视觉侧过滤与打标的基线参考，音频侧需另行调研腾讯的 HunyuanVideo-Foley（约10万小时 TV2A 数据集）与 HunyuanVideo-Avatar。

## 不确定字段

以下字段的调研信息部分不确定（⚠️ 标注来源）：

- data_sources
- provenance_licensing
- domain_distribution
- audio_category_distribution
- language_accent_distribution
- motion_filtering
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
- data_infra_throughput
- data_ablation
- quality_over_quantity_evidence
- benchmark_taxonomy_alignment
