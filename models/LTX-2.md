# LTX-2（含后续 LTX-2.3；技术报告《LTX-2: Efficient Joint Audio-Visual Foundation Model》）

> 主题: 视频生成模型（含音视频同时生成）的数据处理：数据清洗流程、数据分布、打标方式

[← 返回首页](../index.md)

**目录**: [基本信息](#基本信息) · [数据规模与分布](#数据规模与分布) · [清洗流程](#清洗流程) · [打标方式](#打标方式) · [音视频对齐](#音视频对齐) · [训练配合](#训练配合) · [效果对比](#效果对比)

## 基本信息

### 名称

LTX-2（含后续 LTX-2.3；技术报告《LTX-2: Efficient Joint Audio-Visual Foundation Model》）

### 发布机构/公司

Lightricks（以色列，LTX Studio / LTXV 系列模型团队）

### 发布时间（技术报告/论文/开源时间）

2025年10月首次对外公布 LTX-2（预览/产品化接入 LTX Studio）；2026年1月6日正式开源全部模型权重与训练代码，同日在 arXiv 发布技术报告 arXiv:2601.03233v1（cs.CV，14页，含2页补充材料）；2026年3月5日发布升级版 LTX-2.3（22B，桌面端 LTX Desktop）。前代基础：LTX-Video 2B（2024年11月）、LTXV-13B（2025年5月）。

### 类型（模型/数据集/工具链/评测基准）

模型（开源的文本到「音频+视频」联合生成基础模型，T2AV），并附带开源工具链（ltx-core / ltx-pipelines / ltx-trainer 推理与训练微调代码、ComfyUI 与 Diffusers 集成、相机/姿态/唇形配音等控制 LoRA）。非数据集、非评测基准。

### 开源程度（权重/代码/数据/pipeline各自是否开源）

开源程度在同类音视频联合生成模型中最高，但数据侧仍封闭。
【已开源】(1) 全部模型权重：ltx-2-19b-dev（bf16 可训练）、fp8/fp4 量化版、8步蒸馏版 ltx-2-19b-distilled，以及空间/时间上采样器，均在 Hugging Face 发布（Lightricks/LTX-2，月下载量约42.7万）；(2) 推理代码与多后端集成；(3) 训练/微调代码 ltx-trainer（支持 LoRA、全参微调、IC-LoRA）；(4) 技术报告公开架构细节。
【未开源】训练数据本身、数据处理 pipeline 代码、内部 captioner 模型、美学评分模型、数据统计数字。
【许可】LTX-2 Open Weights License：学术研究免费，年经常性收入（ARR）低于1000万美元的公司商用免费；超过该阈值需向 Lightricks 取得商业许可。官方自称「首个真正开放权重的生产级音视频生成模型」。

### 是否支持音视频同时生成，以及实现方式（原生联合/级联/MoE融合）

支持，且为原生联合生成（native joint generation），是本模型的核心定位。实现方式为「解耦但集成的非对称双流」（decoupled yet integrated asymmetric dual-stream），既非级联也非 MoE 融合：
(1) 模态各自独立 VAE：视频用时空因果 VAE；音频先转 16kHz 立体声 mel 频谱（双通道沿 channel 维拼接）再过独立因果音频 VAE，每个 latent token 约对应 1/25 秒音频、128维；解码后经改造版 HiFi-GAN 声码器（通道数翻倍以支持立体声）上采样重建 24kHz 双声道波形。
(2) 非对称双流 DiT：14B 参数视频流 + 5B 参数音频流（共19B），共享深度。每个 dual-stream block 顺序执行：同模态 Self-Attention → 文本 Cross-Attention → 音视频 Cross-Attention → FFN，层间 RMSNorm。视频流用 3D RoPE，音频流用 1D 时间 RoPE。
(3) 跨模态交互：双向 audio-video cross-attention 贯穿全深度；跨模态注意力中只使用 RoPE 的时间分量（强制注意力聚焦于时间同步而非空间对齐）；引入 cross-modality AdaLN——一路模态的 scale/shift 由另一路的隐状态与 diffusion timestep 决定，用于调节各阶段吸收多少跨模态信息。
(4) 推理侧 modality-aware CFG（Bimodal CFG）：\hat{M}=M(x,t,m)+s_t(M(x,t,m)-M(x,∅,m))+s_m(M(x,t,m)-M(x,t,∅))，文本引导 s_t 与跨模态引导 s_m 独立调节；论文取视频流 s_t=3, s_m=3，音频流 s_t=7, s_m=3。增大 s_m 可提升时序同步与语义一致性。
(5) 解耦 latent 天然支持 V2A（为已有视频配同步音频）与 A2V（由音轨驱动视频）编辑工作流。
(6) 输出能力：最长20秒连续音视频（超过 Veo 3 的12秒、Sora 2 的16秒、Ovi 的10秒）；产品口径宣称原生 4K、最高 50fps；论文中描述的推理策略为多尺度多分块：先在约 0.5MP 低分辨率生成 base latent 建立全局构图/运动/音画同步 → latent 上采样器提升空间分辨率 → 重叠时空 tile 分块精修到 1080p 后在 latent 空间融合。（4K/50fps 为产品与新闻稿口径，论文正文只写到 Full-HD 1080p。）

### 调研信息来源列表（论文/技术报告/官方文档/新闻的URL，标注每条来源的性质：官方一手/同团队旁证/第三方报道）

- 【官方一手】LTX-2: Efficient Joint Audio-Visual Foundation Model, Yoav HaCohen 等, Lightricks, arXiv:2601.03233v1, 2026-01-06（第5节 Training Data 与 5.1 Captioning 为数据侧全部一手信息）: https://arxiv.org/abs/2601.03233
- 【官方一手】LTX-2 论文 PDF: https://arxiv.org/pdf/2601.03233
- 【官方一手】LTX-2 论文 HTML 版: https://arxiv.org/html/2601.03233v1
- 【官方一手·同团队旁证】LTX-Video: Realtime Video Latent Diffusion, arXiv:2501.00103（第3节 Data Preparation 含9级清洗 pipeline 图、美学 Siamese 模型、CLIP 去重、运动过滤、re-captioning，以及 Fig.13 词云、Fig.14 caption 词数与 clip 时长分布——LTX-2 数据集为其子集）: https://arxiv.org/abs/2501.00103
- 【官方一手】LTX-2 官方 GitHub 仓库（ltx-core / ltx-pipelines / ltx-trainer、LoRA、许可证）: https://github.com/Lightricks/LTX-2
- 【官方一手】LTX-2 许可证原文: https://github.com/Lightricks/LTX-2/blob/main/LICENSE
- 【官方一手】Hugging Face 模型卡 Lightricks/LTX-2（变体、量化版、蒸馏版、上采样器、Limitations）: https://huggingface.co/Lightricks/LTX-2
- 【官方一手】Hugging Face Lightricks/LTX-2.3（22B 升级版）: https://huggingface.co/Lightricks/LTX-2.3
- 【官方一手】Lightricks 开源 LTX-2 新闻稿, GlobeNewswire, 2026-01-06（4K、开放权重与训练代码、ARR 1000万美元许可门槛）: https://www.globenewswire.com/news-release/2026/01/06/3213304/0/en/Lightricks-Open-Sources-LTX-2-the-First-Production-Ready-Audio-and-Video-Generation-Model-With-Truly-Open-Weights.html
- 【官方一手】LTX 官方新闻室：LTX-2 全量权重开源公告: https://ltx.io/newsroom/ltx-2-is-now-open-source-full-model-weights-released
- 【官方一手】LTX-2 官方 Prompting 指南（音频/对白/口音/foley/环境音的提示结构，与训练 caption 同构）: https://ltx.io/model/model-blog/prompting-guide-for-ltx-2
- 【官方一手】LTX-2.3 官方 Prompt 指南: https://ltx.io/model/model-blog/ltx-2-3-prompt-guide
- 【官方一手】LTX 官方文档 Prompting Guide: https://docs.ltx.io/api-documentation/implementation-guides/prompting-guide
- 【官方一手·合规旁证】Lightricks 与 Shutterstock 视频训练数据合作新闻稿, PR Newswire, 2024-12（首个采用 research license 的全球伙伴）: https://www.prnewswire.com/news-releases/lightricks-partners-with-shutterstock-for-video-training-data-to-advance-open-source-ltxv-video-ai-generative-video-model-302331526.html
- 【官方一手·合规旁证】Shutterstock 投资者关系页同一新闻稿: https://investor.shutterstock.com/news-releases/news-release-details/lightricks-partners-shutterstock-video-training-data-advance
- 【官方一手】Lightricks 发布 13B LTX Video 模型新闻稿, PR Newswire, 2025-05（提及 Getty Images 战略合作）: https://www.prnewswire.com/news-releases/lightricks-launches-13b-parameters-ltx-video-model-breakthrough-rendering-approach-generates-high-quality-efficient-ai-video-30x-faster-than-comparable-models-302447660.html
- 【第三方报道】Wikipedia: LTX-2（时间线、规格、榜单表现、局限）: https://en.wikipedia.org/wiki/LTX-2
- 【第三方报道】Lightricks 13B LTX Video 与 Getty 合作报道, Dataconomy, 2025-05: https://dataconomy.com/2025/05/14/lightricks-unveils-13b-ltx-video-model-for-hq-ai-video-generation/
- 【第三方报道】Lightricks-Shutterstock 合作与 AI 训练数据许可标准, Metaverse Post: https://mpost.io/lightricks-shutterstock-partnership-sets-new-standards-for-ai-training-data-licensing/
- 【第三方报道】LTX-2 原生 4K 与全开放权重, Open Source For You, 2026-01: https://www.opensourceforu.com/2026/01/ltx-2-from-lightricks-delivers-native-4k-audio-video-with-fully-open-weights/
- 【第三方报道】LTX-2 技术解读, Introl Blog, 2026: https://introl.com/blog/ltx-2-audiovisual-diffusion-synchronized-video-audio-2026
- 【第三方整理】awesome-ltx2 提示词指南（对白引号、语言口音、foley/ambience/music 描述规范）: https://github.com/wildminder/awesome-ltx2/blob/main/LTX2-prompt-guide.md
- 【第三方报道】LTX-2 许可证与商用范围解读, WaveSpeed Blog: https://wavespeed.ai/blog/posts/blog-ltx-2-license-commercial-use/
- 【第三方报道】LTX-2.3 更新要点（2026-03-05 发布、22B、4K/50fps/20秒）, WaveSpeed Blog: https://wavespeed.ai/blog/posts/ltx-2-3-whats-new-2026/
- 【相关工作对照】Ovi: Twin Backbone Cross-Modal Fusion for Audio-Video Generation, arXiv:2510.01284（LTX-2 的主要开源对比对象）: https://arxiv.org/abs/2510.01284
- 【相关工作对照】Veo 3 技术报告（LTX-2 的闭源对比对象）: https://storage.googleapis.com/deepmind-media/veo/Veo-3-Tech-Report.pdf

## 数据规模与分布

### 训练数据量级（视频条数/小时数/token数，预训练与SFT分开） ⚠️

完全未披露。技术报告第5节「Training Data」全长仅两段（约150词），未给出任何视频条数、总时长、token 数，也未区分预训练与微调规模。仅定性说明使用「LTX-Video 所用同一数据集的一个子集（a subset of the same dataset employed in LTX-Video）」，该子集聚焦于「含有显著且信息量丰富的音频成分的视频片段」。前代 LTX-Video 论文同样未给出数据规模数字。[不确定]

### 数据来源构成（自有/公开数据集/网络爬取/授权采购/合成数据）

构成为「公开可得数据 + 授权采购素材」的混合。LTX-Video 论文原文：「Our training dataset comprises a robust collection of publicly available data, supplemented with licensed material」。LTX-2 直接复用该数据池的音频信息量子集。
授权来源有明确公开旁证：(1) Shutterstock——2024年12月官宣，Lightricks 是全球首个采用 Shutterstock「research license（研究许可）」训练开源模型的伙伴，可使用其 HD 与 4K 视频素材库；(2) Getty Images——2025年在研发 13B 模型期间建立战略合作，获取高质量视频素材库。此外训练中混入图像数据集（LTX-Video 明确将图像作为一种「分辨率-时长组合」参与训练，用以引入视频数据中不常见的概念）。未使用合成数据的说明。各来源占比未披露。

### 数据合规与溯源（授权数据占比、rights-cleared数据集、C2PA等） ⚠️

Lightricks 在同类模型中属于合规叙事最强的一方，但缺少定量披露。官方与媒体口径为「训练数据全部来自授权来源（Getty Images 与 Shutterstock），面向商用无版权顾虑」，Shutterstock 的「research license」被称为降低开源模型训练数据获取门槛的行业首例。但需注意：(1) LTX-Video 论文原文是「公开可得数据 + 授权素材补充」，与「全部授权」的媒体口径存在差异，授权数据占比从未公布；(2) 技术报告未给出 rights-cleared 数据集清单；(3) 未提及 C2PA、水印或任何输出侧溯源机制；(4) 报告「Social Impact」一节仅定性承认模型会反映训练数据中的偏见，并把「真实性验证与可溯源性改进」列为未来工作。[不确定]

### 片段时长分布与切分策略 ⚠️

LTX-2 未单独披露。可参考同源的 LTX-Video：论文 Fig.14(b) 给出过滤后 clip 时长分布直方图，范围约 0–30 秒，质量集中在较短片段（分布随时长单调下降）。切分策略：数据 pipeline 的输入单元本身即为「Input Shots（镜头）」，全流程以 shot 为处理粒度，最终产出「Final Shots」。训练时同时在多种「分辨率×时长」组合上训练，并通过 resize 使各样本 token 数近似相同。生成侧上限20秒，超过约20秒会出现时序漂移与同步退化。具体分桶数值未公布。[不确定]

### 分辨率/宽高比分布与分桶策略 ⚠️

无分布统计数字，但策略明确。(1) 多分辨率多时长联合训练：同时在多种宽/高/时长组合上训练，模型对未见配置泛化良好；(2) token 数对齐而非 padding/packing：将原视频 resize 到可比 token 数，并施加 0%–20% 的随机 token dropping（stochastic token dropping）使各序列 token 数固定，作者称此法比复杂的 token-packing/padding 更简单高效且保留数据多样性；(3) 宽高比标准化：pipeline 中显式「Crop Black Bars」裁除黑边以统一宽高比并提升有效视觉面积；(4) pipeline 含「Resize Shots」步骤。推理侧要求分辨率能被32整除、帧数满足 8n+1。各分辨率/宽高比的具体配比未披露。[不确定]

### 类别/domain分布与配比策略（人物、动作、场景、风格等比例控制与概念均衡） ⚠️

未披露任何类别/domain 配比数字，也未说明概念均衡策略。仅有一句定性说明：所选子集「提供了视觉与听觉内容的平衡分布（a balanced distribution of visual and auditory content）」，使得 caption 能同时充分覆盖图像域与听觉域信息。间接线索：(1) LTX-Video 论文 Fig.13 给出 caption 词云，可粗略反映概念分布，但无数值；(2) 训练中混入图像数据集专门用于补充「视频数据中不常见的概念」，说明团队关注概念覆盖；(3) 授权源为 Shutterstock/Getty 的商业素材库，其分布天然偏向专业拍摄的通用素材而非 UGC。局限性一节承认「训练数据中代表性不足的语言与方言」会导致效果下降，间接说明存在长尾分布问题且未做专门均衡。[不确定]

### 音频类别分布与配比（语音/音效foley/音乐/环境音/静音的比例及控制策略）——AV模型独有维度 ⚠️

这是 LTX-2 数据侧最有价值、也是唯一明确的筛选准则，但同样无比例数字。
【筛选策略】不是照单全收 LTX-Video 的全量视频，而是「聚焦于含有显著且信息量丰富的音频成分的视频片段（focusing on video clips that contained significant and informative audio components）」——即以「音频信息量」作为构建 AV 训练子集的核心门槛，剔除无声、静音占比过高或音轨与画面无信息关联的片段。这一「音频信息量筛选」是 LTX-2 相对纯视频模型在数据侧的关键增量。
【覆盖类别】caption 系统与模型能力覆盖四类：对白/语音（speech，含精确转写）、音乐（music）、环境音/氛围（ambient sounds、background）、拟音/音效（foley）。论文强调 LTX-2「不止于生成语音」，而是产出跟随角色、环境、风格与情绪的完整音景（full soundscape）。
【未披露】四类音频各自占比、静音片段保留比例、类别配比控制方法、「显著且信息量丰富」的具体量化判据（无 SNR/响度/静音占比等阈值）。模型卡亦承认「生成非语音内容时音频质量较低」，暗示语音类数据占比可能显著高于音效/音乐类。[不确定]

### 叙事结构分布（单镜头vs多镜头、平均clip时长、镜头数分布、是否含原生音轨） ⚠️

未披露单镜头 vs 多镜头配比、平均镜头数分布。可确定的是数据组织粒度为单镜头：pipeline 从「Input Shots」到「Final Shots」全程以 shot 为单位，因此训练样本以单镜头 clip 为主，模型也不以多镜头叙事为卖点（论文将「更深层的叙事连贯性」明确列为需依赖外部 LLM 生成条件文本来弥补的局限）。是否含原生音轨：LTX-2 子集按定义全部为自带且音频信息量显著的原生同步音轨样本，这是其区别于母集的关键。平均 clip 时长参见 LTX-Video 时长直方图（0–30秒）。[不确定]

### 语言/口音分布（多语种唇同步能力的数据基础） ⚠️

未给出语种列表与占比，但语言/口音在数据标注与架构两端都是一等公民。
【标注侧】caption 系统对对白做「精确转写 + 说话人（speaker）、语言（language）、口音（accent）识别标注」——这是本条目最具参考价值的打标设计之一，直接构成多语种唇同步与口音可控的数据基础。
【架构侧】采用 Gemma3-12B 多语言 LLM 作为文本编码器，并做多层特征抽取（早期层含原始音素信息、后期层含复杂语义），团队明确指出「深层文本理解不仅服务于全球语言支持，更直接决定生成语音的音素准确性」。推理时用户可在 prompt 中用引号包裹台词并指明期望的语言与口音。
【局限】论文承认：训练数据中代表性不足的语言或方言，其语音合成准确度与音画对齐会明显变差。具体覆盖多少语种、各语种/口音样本占比，完全未披露。[不确定]

## 清洗流程

### 清洗漏斗整体结构（几级过滤、各级顺序）

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

### 漏斗定量保留率（每级过滤的输入/输出量与最终保留率，如Apollo 27%） ⚠️

完全未披露。LTX-2 与 LTX-Video 两篇论文均未给出任何一级过滤的输入/输出样本量、淘汰率或最终保留率，无法与 Apollo 27% 之类的定量漏斗对比。仅知 LTX-2 训练集是 LTX-Video 数据集的「一个子集」，但子集占母集的比例同样未公布。[不确定]

### 镜头切分方法（PySceneDetect/自研模型/shot-aware splitting） ⚠️

pipeline 的输入单元直接标注为「Input Shots」，说明上游已完成镜头切分，且全流程（估运动、去重、resize、美学评分、过滤）均以 shot 为原子单位，最终产物为「Final Shots」。但两篇论文均未说明镜头切分所用的具体方法——既未提及 PySceneDetect/TransNetV2 等开源工具，也未说明是否为自研 shot-aware splitting 模型，无阈值参数。[不确定]

### 质量过滤（美学评分、清晰度、OCR文字过滤、黑边/水印/logo检测） ⚠️

视频侧质量过滤是 LTX 系列披露最细的环节，核心是自研的成对偏好美学评分模型：
(1) 标注对采样：先用一个多标签网络（multi-labeling network）为数百万样本打标签，只在「至少共享 top-3 标签之一」的样本之间采样配对——目的是让比较发生在同类内容之间，从而最小化按美学过滤时引入的分布漂移（distribution shift）。这一设计是该 pipeline 中最值得借鉴的细节。
(2) 人工标注：数万（tens of thousands）图像对由人工标注哪一张美学更优。
(3) 模型训练：用这些偏好对训练一个 Siamese Network，学习保持标注序关系的美学分数（排序式而非绝对回归式打分），可同时评估视频与图像。
(4) 应用：为每个样本计算美学分，低于阈值者剔除（阈值数值未公布）；微调阶段进一步只取美学分最高的子集。
(5) 几何清洗：显式 Crop Black Bars 裁除黑边。
【未提及】OCR/字幕文字过滤、水印与 logo 检测、清晰度/模糊度判据、压缩伪影检测均无任何说明。[不确定]

### 运动过滤（光流/运动分数阈值、静态与抖动剔除） ⚠️

pipeline 第3级为「Estimate Motion Level」，并在 Filter Shots 环节据此过滤。论文明确：「我们主动移除运动量不显著（insignificant motion）的视频，以确保数据集聚焦于动态内容」，理由是动态内容更贴合模型的目标能力。未说明运动量估计的具体算法（是否用光流、帧差或其他）、阈值数值，也未提及是否剔除抖动/手持晃动等劣质运动。[不确定]

### 去重方法（精确去重与基于embedding的语义去重分别记录） ⚠️

采用基于表征的语义去重：对每个镜头取中间帧（Mid-Frame）计算 CLIP 图像嵌入，然后在嵌入空间做「Cluster and De-Duplicate（聚类并去重）」。属于 embedding 级语义去重，而非哈希级精确去重；论文未单独说明是否另有精确去重步骤，也未公布聚类算法、相似度阈值、簇内保留策略与去重比例。[不确定]

### VLM/LLM作为质检员（多模态大模型质量打分与错配剔除，2026年从浅层打分器转向大模型语义判断的趋势） ⚠️

LTX 系列未使用 VLM/LLM 作为数据质检员，这是其与2026年主流趋势的显著差异点。数据 pipeline 中承担判别职责的全部是浅层/专用模型：CLIP 图像编码器（用于去重表征）、多标签分类网络（用于采样配对）、Siamese 美学排序网络（用于质量打分）、运动量估计器。大模型在 LTX-2 中的角色集中在两处，但都不是质检：(1) 内部自研 captioner（视频+音频双轨打标）；(2) Gemma3-12B 作为冻结文本编码器参与条件建模。技术报告未提及用多模态大模型做质量语义打分或图文/音画错配剔除。是否在未公开的内部流程中使用，无从判断。[不确定]

### 安全与合规过滤（NSFW、版权、人脸/隐私） ⚠️

训练数据侧的安全与合规过滤完全未披露：NSFW 过滤、人脸/隐私处理、CSAM 筛除等均无任何说明。团队的合规策略主要前置在数据获取端——通过 Shutterstock research license 与 Getty Images 授权采购来保证版权清洁，而非通过下游内容分类器。技术报告「Social Impact」一节仅在部署层面做定性表态：承认合成媒体存在被用于欺骗性内容的风险，要求使用者明确披露合成来源并遵守内容安全准则，承认模型会继承训练数据中的视觉与听觉偏见，并将偏见缓解、真实性验证与可溯源性列为未来工作。模型卡的 Limitations 明确提示模型「可能生成不当内容、放大社会偏见」。未见任何自动化安全过滤模块、分类器或红队流程的描述。[不确定]

## 打标方式

### 使用的caption模型（自研VLM/开源模型，模型规模） ⚠️

使用 Lightricks 自研的内部自动 captioner，未开源、未公布模型规模与基座。
【前代 LTX-Video】使用「internal automatic image and video captioner」对整个训练集做 re-caption（重打标），以保证文本描述准确、相关，改善视觉内容与文本标注的对齐。
【LTX-2 增量】论文第5.1节明确：为 LTX-2 训练「专门开发了一套新的视频 captioning 系统（a new video captioning system）」，能够以详尽细节同时描述片段的视觉轨道与听觉轨道，「捕捉每一个有意义的动作、外观和声音」。该系统被定位为「连接视频、音频与语言三域的综合文本接口，构成 LTX-2 多模态训练语料的描述基础」。
【未披露】captioner 的参数量、基座模型、是否为多模态 LLM、音频理解如何实现（是否内置 ASR + 音频事件检测 + 说话人分离的组合流水线）、打标吞吐与成本。注意 Gemma3-12B 是训练/推理时的文本编码器，不是 captioner。[不确定]

### caption密度与结构化程度（短/长/密集描述、结构化字段如镜头运动、风格标签）

走「超长、密集、单段连贯、纯事实」路线，是典型的 dense caption 范式。
【密度】LTX-Video 论文 Fig.14(a) 给出每条 caption 词数分布，跨度约 0–175 词，主体集中在数十词量级；LTX-2 的 caption 因需同时覆盖听觉轨道而更长更密。官方 prompting 指南建议用户写 4–8 个描述性句子、保持为单个连贯段落——推理 prompt 格式即训练 caption 格式的镜像。
【风格约束】论文明确写出打标准则：「comprehensive yet factual, describing only what is seen and heard without emotional interpretation」——全面但只陈述事实，只描述所见所闻，不做情绪化解读/主观评价。这是一条非常明确的 caption 风格规范。
【结构化程度】不采用显式 JSON/键值字段，而是把结构化信息熔入自然语言长描述中，按时间顺序（chronological）组织。LTX-Video 的 caption 示例显示固定覆盖的语义槽位包括：主体外观与细节 → 主体动作/行为 → 场景与背景元素 → 相机机位与运镜（如「camera follows … from behind, pans slightly」/「camera remains stationary」）→ 光照与色彩（「natural and slightly overcast lighting, casting soft shadows」）→ 风格/来源标签（固定后缀如「The scene is captured in real-life footage.」或「The scene appears to be from a movie or TV show.」）。LTX-2 在此基础上并入听觉槽位。

### 音视频联合caption结构（是否同时覆盖视觉+听觉轨道、是否分流为独立字段，如LTX-2全音景描述、Script-a-Video factorized streams、Foley-Omni三字段）

本条目的核心贡献，也是当前公开资料中音视频联合 caption 设计的最佳样本之一。
【设计原则】单一 caption 同时穷尽覆盖视觉与听觉两条轨道（describing both the visual and auditory tracks of a clip in exhaustive detail），而非拆分为多个独立字段——与 Script-a-Video 的 factorized streams、Foley-Omni 三字段属于不同路线：LTX-2 走的是「融合式全信息长描述」。
【听觉侧覆盖（full soundscape，全音景）】
- 音乐（music）：类型、配器、节奏、情绪
- 环境音/氛围（ambient sounds、background）：风、雨、城市底噪、室内房间声、人群嘈杂声
- 拟音/音效（foley）：脚步、衣物摩擦、键盘敲击、门吱声、玻璃碰撞
- 对白（dialogue）：精确转写（precise transcriptions），并附说话人（speaker）、语言（language）、口音（accent）识别
【视觉侧覆盖】相机运动（camera motion）、光照（lighting）、主体行为（subject behavior），以及外观、场景、风格来源。
【一致性设计】官方 prompting 指南与训练 caption 结构同构——用户提示词同样要求单段落、时间顺序、显式描述 ambience/foley/music/speech，台词放在引号内并可注明语言与口音，用生理动作线索（physical cues）而非情绪标签指导表演。这印证了「训练 caption 即推理接口」的设计闭环。
【意义】音频不再是视觉 caption 的附属，而是与视觉平权的描述维度；「音频信息量筛选（数据侧）+ 全音景双轨 caption（标注侧）」构成 LTX-2 数据方法论的完整逻辑：先保证样本音轨有信息量，再保证标注把这些信息完整暴露给模型。
【未披露】caption 中视觉与听觉内容的长度配比、听觉描述的模板或槽位定义、是否有质量校验环节。

### 对白转写与说话人属性标注（ASR转写+说话人身份/语言/口音/情绪）

明确执行且是 caption schema 的显式组成部分：caption 包含对白的「精确转写（precise transcriptions of dialogue）」，并同时标注说话人身份（speaker）、语言（language）与口音（accent）三类属性。这三项属性标注直接支撑了模型的多语种唇同步、说话人区分与口音可控能力（论文称模型可合成「不仅与唇动同步，且在节奏、口音与情绪基调上都自然」的语音）。
值得注意的对比：情绪（emotion）不作为标注字段——打标准则明确要求「不做情绪化解读」，只客观描述所见所闻；情绪表达被期望通过对白内容、生理动作线索与音色描述间接习得。官方 prompting 指南建议用户把台词拆成短句、在句间插入表演指示（acting directions），并用生理线索而非情绪标签。
未披露：ASR 使用的具体模型、说话人分离/聚类（diarization）方法、语言与口音识别模型、转写词错误率或质量把控手段。多说话人场景下模型仍会出现台词错配给角色的问题（论文局限性），暗示说话人-台词绑定的标注或建模仍不够强。

### 几何与结构化标注（相机参数、深度、3D point tracks、动作标注、显式状态） ⚠️

基本没有。两篇论文均未提及相机内外参、深度图、3D point tracks、光流场、显式物理状态或动作类别标签等结构化几何标注。唯一相关的是自然语言层面的相机描述——caption 中以文本形式记录运镜（跟拍、平移、固定机位等），属于弱结构化。附加线索：官方发布了相机运动控制 LoRA 与姿态控制 LoRA，说明存在针对性的控制数据集，但其构造方式（是否使用显式相机轨迹或姿态估计标注）未公开。[不确定]

### 合成数据构造（受控扰动/编辑构造训练对，如InstructAV2AV） ⚠️

技术报告未提及使用任何合成数据或受控扰动/编辑构造训练对（无类似 InstructAV2AV 的编辑配对构造）。训练数据均来自真实视频（公开可得 + 授权素材）加图像数据集。唯一涉及「模型生成内容」的地方是 LTX-Video 论文中用 Sora 生成的视频作为 captioner 效果的展示样例，属于演示而非训练数据。是否在未公开流程中使用蒸馏/自生成数据（如蒸馏版模型的训练）无从判断。[不确定]

### 人工介入程度（人工标注、人工质检、模型初筛+人工复核）

人工介入集中在质量评判器的构建环节，而非逐样本标注：
(1) 美学模型的标注：数万（tens of thousands）图像对由人工标注「哪一张美学更优」，用于训练 Siamese 排序网络；上游先用多标签网络对数百万样本自动打标以约束配对采样范围。这是典型的「人工标少量偏好对 → 训练自动打分器 → 全量自动过滤」的杠杆式设计。
(2) 逐样本 caption：完全自动，由内部 captioner 对全量训练集 re-caption，未提及人工复核或抽检流程。
(3) 评测环节：使用人类偏好研究（human preference study）评估视觉真实感、音频保真度与时序同步（唇同步、foley 准确度），LTX-Video 阶段的人评使用1000条文生视频 prompt 与1000对图生视频样本。
数据清洗环节是否有人工兜底质检，未披露。

## 音视频对齐

### 音视频同步检测方法（口型同步、事件对齐） ⚠️

这是 LTX-2 数据披露中最大的空白，也是与调研主题最相关却最缺失的一环。技术报告未描述任何训练数据侧的音视频同步检测或异步样本剔除机制——没有唇同步检测器、没有事件对齐检测、没有异步剔除阈值。
其同步能力的来源被归结为三点，全部在数据筛选与架构侧而非同步检测侧：
(1) 数据侧：只保留「音频成分显著且信息量丰富」的片段——即保留原生自带同步音轨的真实视频，天然规避了配音/后期贴音等异步样本（但论文未说明是否显式剔除画外音、后期配乐等非同期声）；
(2) 标注侧：caption 精确转写对白并标注说话人，使文本-语音-唇动三者在语义层可对齐；
(3) 架构侧：贯穿全深度的双向音视频 cross-attention，且跨模态注意力只使用 1D 时间 RoPE，强制注意力集中在时间同步维度，论文称可以「亚帧精度（sub-frame precision）」把视觉事件（如物体撞击）映射到听觉事件（对应的 foley 声）；cross-modality AdaLN 进一步调控跨模态信息注入强度；推理时增大跨模态引导 s_m 可提升时序同步。
论文 Fig.3 通过可视化 AV cross-attention 图作为同步能力的定性证据：模型能空间跟踪行驶车辆、在两位说话人之间动态切换注意力、并在近景说话时聚焦唇部区域。[不确定]

### 同步检测具体指标与阈值（SyncNet/Synchformer/LSE/自研，阈值数值，如MOVA LSE-D≤9.5且LSE-C≥4.5、SkyReels-V4 SyncNet |offset|≤3∧conf>1.5） ⚠️

完全未披露。论文未使用 SyncNet、AV-align 或任何自研同步指标，未给出任何置信度阈值数值（无法与 UniTalking SyncNet conf>0.9 之类对比），也未报告 LTX-2 在任何客观音视频同步基准上的分数。全部音视频质量结论均来自内部人类偏好研究（对比 Ovi、Veo 3、Sora 2），且论文未给出人评的具体分数表，仅定性描述「显著优于 Ovi」「与领先闭源模型相当」。唯一的量化表格是推理速度对比（H100 上 121帧 720p、单步 Euler、CFG=1：LTX-2 19B 音视频 1.22秒/步 vs Wan 2.2-14B 纯视频 22.30秒/步，约18倍加速）。[不确定]

### 时序同步vs语义同步的分离处理（时间对齐与内容语义匹配作为两个独立过滤条件） ⚠️

论文未在数据过滤层面区分「时序同步」与「语义匹配」两个独立条件——数据侧只有「音频信息量」这一条筛选准则。但在方法论叙述与架构设计上，两者被明确区分并分别对应不同机制：
(1) 时序同步：由跨模态注意力中只保留 RoPE 时间分量来承担（明确表述为「强制跨模态注意力聚焦于时间上的同步而非空间对齐」），目标是唇同步与撞击-音效的亚帧对齐；
(2) 语义/环境匹配：由双向依赖建模承担——论文的核心论点是「唇同步主要由音频驱动，而声学环境（混响、foley）由视觉上下文决定」，因此必须联合建模双向依赖，这是反对级联 V2A/A2V 方案的主要理由；推理时增大跨模态引导 s_m 同时改善「时序同步」与「语义连贯」，论文将二者并列表述。
数据侧是否有对应的两类过滤条件，未披露。[不确定]

### 音频质量过滤（SNR、静音检测与静音占比阈值、无音轨剔除、画外音源剔除、背景音乐分离） ⚠️

唯一且核心的音频侧过滤准则即「significant and informative audio components（显著且信息量丰富的音频成分）」——这实质上同时承担了无音轨剔除、静音/近静音剔除、低信息量音轨（如纯噪声、极低响度）剔除的职能，并被论文称为使子集获得「视觉与听觉内容平衡分布」的关键。
【未披露】该准则的量化实现：无 SNR 阈值、无响度/RMS 阈值、无静音占比阈值、无音频事件密度指标；未说明是否剔除画外音/旁白等非同期声源，未说明是否做背景音乐分离（source separation）或人声-伴奏分离。这是「思路明确但实现细节完全不可复现」的典型。技术侧仅知音频以 16kHz 立体声 mel 频谱进入 VAE、24kHz 输出，未说明对原始音轨采样率/声道数的准入要求。[不确定]

### 语音/音效/音乐的分类与分别处理策略 ⚠️

在 caption 标注层面对四类音频做了显式区分与分别描述——语音（精确转写 + 说话人/语言/口音）、音乐（music）、环境音/氛围（ambient、background）、拟音音效（foley），这是模型能分别响应四类音频指令的基础。
但在数据过滤与训练配比层面，论文未说明是否对四类分别设计过滤规则、配比目标或损失权重——「音频信息量筛选」是一个统一的门槛，未见按类型分流处理。架构上音频是单一 5B 流统一建模，无按类型分支或专家路由。
间接证据显示存在类别不均衡：模型卡 Limitations 明确指出「生成非语音内容时音频质量较低（produce lower-quality audio when generating non-speech content）」，说明语音类数据在子集中占优，音效/音乐类相对不足。论文亦承认深层文本理解主要是为了服务语音的音素准确性，重心明显偏向语音。[不确定]

## 训练配合

### 多阶段训练课程与数据课程调度（阶段划分依据：分辨率/时长/质量分/模态；低清→高清、图像→视频、短→长） ⚠️

训练课程有部分披露，但缺乏数据维度的阶段划分细节。
(1) 模态渐进：核心思路是「在已预训练好的视频 DiT 基础上扩展音频流」——结论部分表述为「通过用一个轻量的 3B 音频流扩展一个预训练的 13B 视频扩散 transformer」（正文与摘要给出的规格为 14B 视频 + 5B 音频，存在口径差异），经双向 cross-attention、1D 时间 RoPE 与 cross-modality AdaLN 连接，从而无需复制视觉主干；随后进行「渐进式联合训练（progressive joint training）」。这是明确的「先视频后音视频」的模态课程。
(2) 文本投影的两段式：多层特征抽取的投影矩阵 W 在一个简短的初始训练阶段与 LTX-2 联合优化（LLM 权重全程冻结，用标准扩散 MSE 损失），该阶段带来整体质量提升；此后 W 被冻结并沿用于所有后续训练。文本 connector（含 thinking tokens）则与音视频 DiT 块一起训练。
(3) 分辨率/时长：不做严格的低清→高清分段，而是多分辨率多时长同时训练 + token 数对齐 + 随机 token dropping；高分辨率通过推理时的多尺度多分块策略（0.5MP base → latent 上采样 → tile 精修）而非训练课程实现。
(4) 图像参与：图像作为一种「分辨率-时长组合」与视频混合训练。
(5) 前代 LTX-Video：预训练后在高美学子集上微调。
未披露各阶段的步数、数据量、切换判据。[不确定]

### 数据配比随训练阶段的变化（预训练/退火annealing/SFT高质量子集） ⚠️

LTX-2 未披露各训练阶段的数据配比变化，无 annealing/退火阶段说明，也未说明音视频联合训练阶段是否仍混入纯视频（无音轨）数据以维持视觉质量——考虑到论文强调「联合训练不损害视觉单模态质量」（Artificial Analysis 榜单上 I2V 第3、T2V 第4，超过 Sora 2 Pro 与 Wan 2.2-14B），推测存在纯视频数据的混合或视频流参数的部分保护，但无任何官方依据。可参考的同源做法仅有 LTX-Video 的「预训练全量 → 微调阶段只用最高美学子集」。[不确定]

### 后训练数据（SFT精选集规模与筛选标准、偏好对数量与标注方式、reward model训练数据） ⚠️

完全未披露。技术报告没有 SFT/RLHF/DPO 章节，无精选集规模与筛选标准，无偏好对数量，无 reward model 训练数据说明。已知存在但未说明数据来源的后训练产物：8步蒸馏版 ltx-2-19b-distilled（蒸馏数据构造方式未公开）、fp8/fp4 量化版（校准数据未说明）、以及一系列控制 LoRA（相机运动、姿态控制、唇形配音 lip dubbing），后者显然需要专门构造的配对数据集，但构造方法未公开。人类偏好研究仅用于最终评测，未说明其结果是否回流为训练信号。[不确定]

### 数据处理基础设施与吞吐（NeMo Curator/Data-Juicer/自研，GPU加速比，处理规模，成本） ⚠️

完全未披露。技术报告未提及 NeMo Curator、Data-Juicer 或任何自研数据处理框架，无 GPU 加速比、处理吞吐、处理规模或成本数据，也未说明数据 pipeline 的分布式实现。论文中所有效率数字均属推理侧：H100 上 121帧 720p 单步耗时 1.22 秒（对比 Wan 2.2-14B 的 22.30 秒，约18倍），并强调该差距在更高分辨率与更长时长下进一步扩大；模型设计目标之一是可在消费级 GPU 本地运行。[不确定]

## 效果对比

### 数据策略消融的量化影响（区分：过滤严格度ablation / caption密度风格ablation / 数据配比ablation，及对应评测指标） ⚠️

无任何数据策略消融实验。技术报告没有过滤严格度 ablation、没有 caption 密度/风格 ablation、没有数据配比 ablation，也没有「加入音频信息量筛选 vs 不加」的对照。
已有的消融/对比全部集中在架构与推理侧：
(1) modality-CFG：给出独立调节 s_t（文本引导）与 s_m（跨模态引导）的效果分析（Fig.5），结论为增大 s_m 提升时序同步与跨模态语义一致性，最终取视频流 (3,3)、音频流 (7,3)；
(2) 文本条件设计：定性说明多层特征抽取优于仅用末层 causal 嵌入、thinking tokens 显著改善语音音素准确性与复杂 prompt 遵循，但未给出量化对照表；投影矩阵 W 的初始联合训练「带来整体质量提升」亦无数字；
(3) 效率对比：与 Wan 2.2-14B 的单步耗时表（唯一的量化表格）；
(4) 质量对比：内部人类偏好研究（vs Ovi / Veo 3 / Sora 2）与 Artificial Analysis 公开榜单（截至2025年11月6日，I2V 第3、T2V 第4），均无分数明细。[不确定]

### 质量vs数量的证据（小而精数据超越大而杂的案例） ⚠️

存在隐含证据但无对照实验支撑。LTX-2 的整个数据策略就是「从更大的 LTX-Video 数据集中筛出音频信息量显著的子集」——用母集的一个真子集训练出开源 SOTA 的音视频联合模型，且视觉质量未退化（榜单上仍超过 Sora 2 Pro 与 Wan 2.2-14B），这本身是「有针对性筛选优于全量堆量」的一个案例。类似地，LTX-Video 的微调阶段只使用美学分最高的子集，也体现同一取向。
但两篇论文都没有做「全量 vs 子集」的对照实验，没有给出子集占比，也没有量化「质量优于数量」的收益。因此只能算方法论倾向而非实验证据。[不确定]

### 训练数据domain分布与评测基准类目体系的对齐关系（如VABench七大类） ⚠️

未披露训练数据的 domain 类目体系，因此也谈不上与评测基准类目的对齐关系（未涉及 VABench 七大类等类目化基准）。LTX-2 的评测策略是「人类偏好 + 公开榜单」而非类目化基准：(1) 内部人类偏好研究，评价维度为视觉真实感、音频保真度、时序同步（唇同步与 foley 准确度）三项，对比对象为 Ovi、Veo 3、Sora 2；(2) Artificial Analysis 公开排行榜的 I2V 与 T2V 两个赛道。未使用 VBench、未使用任何音视频同步客观指标，也未构建或公布自有的类目化评测集。评测维度（视觉/音频/同步）与数据构建逻辑（视觉质量过滤/音频信息量过滤/联合 caption）在概念上一一呼应，但这是结构上的巧合而非论文声明的对齐设计。[不确定]

## 其他信息

### summary_note

【定位】LTX-2 是当前「开源程度」与「数据方法论可读性」之间落差最典型的样本：权重、推理代码、训练代码全部开放（同类中最彻底），但14页技术报告中「Training Data」一节仅两段约150词，无一个数据统计数字。其数据侧的可复现性远低于其模型侧。
【最具参考价值的三点】
1. 音频信息量筛选（audio-informativeness filtering）：不把纯视频数据集直接拿来做音视频训练，而是以「音频成分是否显著且信息量丰富」为准则抽取子集，一举完成无声/静音/低信息音轨的剔除并获得视听内容的平衡分布。这是 AV 模型区别于纯视频模型的第一性数据操作，思路清晰、可迁移，但无量化判据。
2. 双轨全音景 caption（本条目的核心亮点）：单条 caption 穷尽覆盖视觉轨道（相机运动、光照、主体行为、外观、场景、风格来源）与听觉轨道（音乐、环境音、foley、对白精确转写 + 说话人/语言/口音标注），并明确「comprehensive yet factual，只述所见所闻、不做情绪化解读」的风格准则。听觉被提升为与视觉平权的描述维度，且训练 caption 结构与推理 prompt 结构同构（官方 prompting 指南要求单段落、时间顺序、显式描述 ambience/foley/music/speech、台词加引号并注明语言口音），形成完整的数据-接口闭环。说话人/语言/口音三属性标注是多语种唇同步能力的直接数据基础。
3. 可复用的视频清洗漏斗（继承自 LTX-Video）：9级 shot 级 pipeline（黑边裁剪 → 运动量估计 → 缩略图 → 中间帧 CLIP 嵌入 → 聚类去重 → resize → 美学评分 → 阈值过滤），其中美学评分器的构建方式最值得借鉴——先用多标签网络约束配对采样范围（只在共享 top-3 标签之一的样本间配对）以最小化过滤引入的分布漂移，再用数万人工偏好对训练 Siamese 排序网络，最后全量自动打分过滤。
【最大的信息缺口】音视频同步检测：作为主打「时序同步」的模型，其数据 pipeline 中没有任何唇同步/事件对齐检测环节的披露，没有 SyncNet 类指标、没有阈值、没有异步样本剔除说明。同步能力被完全归因于「原生同步音轨数据 + 双向 cross-attention 架构 + 时间维 RoPE + modality-CFG」，数据侧的同步质控是否存在无从判断。其次是全部定量信息的缺失：数据规模、漏斗保留率、音频类别配比、语种分布、caption 长度分布（仅前代有图无数）、数据消融，全部为零。
【合规特色】以「授权数据前置」替代「下游安全过滤」的路线：Shutterstock research license（2024年12月，全球首个伙伴）+ Getty Images 战略合作，使模型主打商业安全；但论文原文表述为「公开可得数据 + 授权素材补充」，与媒体「全部授权」口径有出入，授权占比未公布，且无 C2PA 等输出侧溯源机制。
【与同类对比】相比 Sora 2（数据侧近乎零披露）与 Veo 3，LTX-2 至少给出了筛选准则与 caption schema 的定性设计，且有可运行的开源代码作为旁证；相比 Ovi（双 5B 对称双主干），LTX-2 的非对称双流在效率上更优。若需可复现的 AV 数据处理方法，LTX-2 提供的是「思路模板」而非「参数配方」。

## 不确定字段

以下字段的调研信息部分不确定（⚠️ 标注来源）：

- data_scale
- provenance_licensing
- duration_distribution
- resolution_aspect_distribution
- domain_distribution
- audio_category_distribution
- narrative_structure_distribution
- language_accent_distribution
- funnel_retention_rate
- shot_segmentation
- quality_filtering
- motion_filtering
- deduplication
- model_as_data_judge
- safety_filtering
- caption_model
- geometric_structured_annotation
- synthetic_data_synthesis
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
