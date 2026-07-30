# 音视频生成评测基准合集（VABench / AVBench / AV-SyncBench / PhyAVBench / Omni-Judge）

> 主题: 视频生成模型（含音视频同时生成）的数据处理：数据清洗流程、数据分布、打标方式

[← 返回首页](../index.md)

**目录**: [基本信息](#基本信息) · [数据规模与分布](#数据规模与分布) · [清洗流程](#清洗流程) · [打标方式](#打标方式) · [音视频对齐](#音视频对齐) · [训练配合](#训练配合) · [效果对比](#效果对比)

## 基本信息

### 名称

音视频生成评测基准合集（VABench / AVBench / AV-SyncBench / PhyAVBench / Omni-Judge）

### 发布机构/公司

多家机构，五个基准分属不同团队：
【VABench】北京大学（Wentao Zhang 组，含 Bohan Zeng、Hao Liang、Junbo Niu 等）+ 蚂蚁集团 Ant Group（Quanqing Xu）+ 中国科学院自动化研究所 + 华中科技大学。第一作者 Daili Hua、Xizhi Wang。
【AVBench】清华大学（Wenming Yang 组，第一作者 Jialiang Yang，含 Bin Xia、Ruihang Chu 等）+ 香港中文大学（Dingdong Wang 等）。
【AV-SyncBench】阿里巴巴集团（Jun Song、Bo Zheng 等，通讯 jsong.sj@alibaba-inc.com）+ 清华大学（第一作者 Tianhong Zhou，zth24@mails.tsinghua.edu.cn）+ 复旦大学。
【PhyAVBench】香港科技大学（广州）HKUST(GZ)（通讯作者 Li Liu，第一作者 Tianxin Xie）+ 腾讯 Tencent + 上海交通大学 + 慕尼黑工业大学 TUM；作者 29+ 人，4 位 core contributor。
【Omni-Judge】罗切斯特大学 University of Rochester（第一作者 Susan Liang，通讯 Chenliang Xu）+ 密歇根大学安娜堡分校（Filippos Bellos、Jason J. Corso）。

### 发布时间（技术报告/论文/开源时间） ⚠️

【VABench】arXiv:2512.09299，v1 于 2025年12月10日提交，v2 于 2026年4月6日更新（24页/25图，cs.CV + cs.SD，CC BY 4.0）。调研任务描述其为 CVPR 2026 论文，但 arXiv comments 字段未标注录用信息[不确定]。
【AVBench】arXiv:2605.24652，2026年5月发布；项目页图表命名指向 ECCV 投稿[不确定]。
【AV-SyncBench】arXiv:2607.00726，2026年7月发布；已被 Interspeech 2026 接收。
【PhyAVBench】arXiv:2512.23994，v1 于 2025年12月发布（当时为纯 benchmark 设计报告，模型评测留待后续），后续版本补齐 17 个模型的完整评测结果。
【Omni-Judge】arXiv:2602.01623，2026年2月发布。

### 类型（模型/数据集/工具链/评测基准）

评测基准（benchmark）。五者均为音视频联合生成（Audio-Video Generation）方向的评测体系，但侧重不同：
- VABench：综合型全维度基准（T2AV / I2AV / 立体声 AV 三类任务，七大内容类目 + 15 项评测维度）；
- AVBench：人类对齐的自动化评测基准 + 可训练的专用评测器（10 维度，附带 300K 偏好训练数据，评测器本身可复用为数据过滤器与 RLHF reward）；
- AV-SyncBench：专项同步性基准（时序同步与语义同步解耦），同时也是一个带扰动标注的数据集；
- PhyAVBench：物理常识专项基准，附带自录真实数据集 PhyAV-Sound-11K（11,605 条视频 / 25.5 小时）；
- Omni-Judge：评测方法学研究（探究 Omni-LLM 能否充当人类对齐的裁判），属于 meta-evaluation。
其中 AVBench、AV-SyncBench、PhyAVBench 同时具备「数据集」属性。

### 开源程度（权重/代码/数据/pipeline各自是否开源） ⚠️

【VABench】论文 CC BY 4.0；代码仓库 https://github.com/tanABCC/VABench；未明确声明数据集许可证。提示词、VQA/AQA 问答对与评测脚本为主要开源产出，生成视频依赖各家 API 自行复现。
【AVBench】开源程度最高：GitHub https://github.com/YaJialiang/AVBench，评测器权重发布于 HuggingFace（iiiiii123/AVBench_model），并托管 HuggingFace Leaderboard（spaces/iiiiii123/AVBenchLB）。数据、代码、模型三者均释出；具体许可证未标注[不确定]。
【AV-SyncBench】数据集已上线 ModelScope（coming245/AVSyncBench）与 HuggingFace（coming245/AV-SyncBench），代码仓库 https://github.com/fgt7t6g/AV-SyncBench（截至调研时 README 标注评测代码 coming soon）；论文采用 arXiv perpetual non-exclusive license。
【PhyAVBench】项目页 https://imxtx.github.io/PhyAVBench/ 与 https://phyavbench.pages.dev/；公开释出提示词、自录 ground-truth 视频与各模型生成样本，并承诺与训练集零重叠；论文 arXiv 许可证。
【Omni-Judge】仅有项目页 liangsusan-git.github.io/project/omni_judge/，论文未明确声明代码/数据开源[不确定]。

### 是否支持音视频同时生成，以及实现方式（原生联合/级联/MoE融合）

五者本身均不生成音视频，而是评测音视频联合生成能力。它们覆盖的被测系统形态恰好构成了当前 AV 生成的三条技术路线，评测设计上明确区分：
1) 原生联合生成（端到端 T2AV/I2AV）：Sora 2、Veo 3 / Veo 3.1 / Veo3-Fast、Wan 2.5 Preview / Wan 2.6、Kling 2.5 Turbo / Kling v2.6、Seedance 1.5 Pro，开源侧 Ovi、LTX、MOVA、UniVerse-1、JavisDiT / JavisDiT++；
2) 级联组合（V+A，先生成视频再配音）：视频端 Seedance-1.0-Lite / Wan2.2-TI2V / Kling2.5 Turbo，音频端 MMAudio、ThinkSound(-Light)、HunyuanVideo-Foley、FoleyCrafter；
3) 表征/判别模型（AV-SyncBench 的被测对象）：Synchformer、SparseSync、ImageBind、CAV-MAE、CAV-MAE-Sync。
VABench 额外引入「立体声音视频生成」这一路线，用 116 条显式指定左右声道方位的提示词考察空间音频生成能力，是目前少见的立体声 AV 评测维度。

### 调研信息来源列表（论文/技术报告/官方文档/新闻的URL，标注每条来源的性质：官方一手/同团队旁证/第三方报道）

- https://arxiv.org/abs/2512.09299 —— VABench: A Comprehensive Benchmark for Audio-Video Generation，arXiv 摘要页（官方一手）。作者 Daili Hua、Xizhi Wang、Bohan Zeng、Xinyi Huang、Hao Liang、Junbo Niu、Xinlong Chen、Quanqing Xu、Wentao Zhang；v1 2025-12-10，v2 2026-04-06；24页25图；cs.CV + cs.SD；CC BY 4.0。
- https://arxiv.org/html/2512.09299v1 —— VABench 论文 HTML 全文（官方一手）。提取七大类目体系、15 项评测维度与两大模块、T2AV/I2AV 双路数据策展流程、778/521/116 样本量、立体声 9 项声学指标、被测模型清单与作者机构（北京大学 / 蚂蚁集团 / 中科院自动化所 / 华中科技大学）。
- https://github.com/tanABCC/VABench —— VABench 官方代码仓库（官方一手）。
- https://arxiv.org/abs/2605.24652 —— AVBench: Human-Aligned and Automated Evaluation Benchmark for Audio-Video Generative Models，arXiv 摘要页（官方一手）。作者 Jialiang Yang、Bin Xia、Ruihang Chu、Dingdong Wang、Wanke Xia、Zhun Mou、Tianyang Zhong、Yiting Zhao、Wenming Yang。
- https://arxiv.org/html/2605.24652v1 —— AVBench 论文 HTML 全文（官方一手）。提取 10 项评测维度、470 条评测提示词与 Normal/Hard 分层、30K→300K 硬负例合成配方、Qwen2.5-Omni 7B 与 Qwen2-Audio 7B 评测器架构、4 名专家 2AFC 标注协议、可用作数据过滤与 RLHF reward 的论述、作者机构（清华大学 / 香港中文大学）。
- https://yajialiang.github.io/AVBench-site/ —— AVBench 官方项目页（官方一手）。含 GitHub、HuggingFace 模型与 Leaderboard 链接。
- https://github.com/YaJialiang/AVBench —— AVBench 官方代码仓库（官方一手）。
- https://huggingface.co/iiiiii123/AVBench_model —— AVBench 评测器权重（官方一手）。
- https://huggingface.co/spaces/iiiiii123/AVBenchLB —— AVBench 在线排行榜（官方一手）。
- https://arxiv.org/abs/2607.00726 —— AV-SyncBench: Decoupled Benchmarking of Temporal and Semantic Audio-Visual Synchronization，arXiv 摘要页（官方一手）。作者 Tianhong Zhou、Mingyang Han、Boyu Li、Yuxuan Jiang、Jiaxin Ye、Dongxiao Wang、Haoxiang Shi、Kunpeng Wang、Jun Song、Cheng Yu、Bo Zheng；已被 Interspeech 2026 接收；3,269 视频 / 38,390 样本。
- https://arxiv.org/html/2607.00726v1 —— AV-SyncBench 论文 HTML 全文（官方一手）。提取 10 场景与 5 挑战任务清单、0.64 秒 chunk 对角线相似度评测协议、Gemini 3 Flash 自动过滤 + 5 名标注员 ≥3 人交叉复核流程、扰动参数谱（偏移 50–500ms 五档 / 抖动 30–700ms 三档 / 变速 0.8×–1.25× 十档）、OpenVoice V2 与 DDSP 语义扰动工具、5 个基线模型实测结论、作者机构（阿里巴巴集团 / 清华大学 / 复旦大学）。
- https://fgt7t6g.github.io/AV-SyncBench —— AV-SyncBench 官方项目页（官方一手）。含 ModelScope / HuggingFace / GitHub 数据集与代码链接。
- https://github.com/fgt7t6g/AV-SyncBench —— AV-SyncBench 官方代码仓库（官方一手），调研时评测代码标注 coming soon。
- https://huggingface.co/datasets/coming245/AV-SyncBench —— AV-SyncBench 数据集（官方一手）。
- https://modelscope.cn/datasets/coming245/AVSyncBench —— AV-SyncBench 数据集 ModelScope 镜像（官方一手）。
- https://arxiv.org/abs/2512.23994 —— PhyAVBench: A Challenging Audio Physics-Sensitivity Benchmark for Physically Grounded Text-to-Audio-Video Generation，arXiv 摘要页（官方一手）。第一作者 Tianxin Xie，29+ 位合作者；25.5 小时 / 11,605 条视频 / 337 组配对提示词 / 6 维度 41 测试点 / 17 个模型评测。
- https://arxiv.org/html/2512.23994v1 —— PhyAVBench v1 论文 HTML（官方一手）。提取五阶段策展流程、物理维度与测试点明细、CPRS 与 FGAS 指标定义、作者机构（HKUST(GZ) / 腾讯 / 上海交通大学 / 慕尼黑工业大学，通讯作者 Li Liu）。注意 v1 为纯 benchmark 设计报告（描述 1,000 组提示词、50 测试点、模型评测留待后续），与后续版本的 337 组 / 41 测试点数据存在版本差异，正文以后续版本为准。
- https://arxiv.org/html/2512.23994v3 —— PhyAVBench 最新版论文 HTML（官方一手）。提取 PhyAV-Sound-11K 数据集规格（11,605 条 / 25.5 小时 / 184 名参与者 / 平均 17 条 GT 每组）、CPRS 改用 CLAP 嵌入、PVR-MOS 74 名评分员、17 个被测模型完整清单与 Sora 2 CPRS 0.4512 等结果、CPRS 与人类判断 Pearson 0.92。
- https://imxtx.github.io/PhyAVBench/ —— PhyAVBench 官方项目页（官方一手）。
- https://phyavbench.pages.dev/ —— PhyAVBench 官方项目页镜像（官方一手）。
- https://arxiv.org/abs/2602.01623 —— Omni-Judge: Can Omni-LLMs Serve as Human-Aligned Judges for Text-Conditioned Audio-Video Generation?，arXiv 摘要页（官方一手）。第一作者 Susan Liang，含 Jason J. Corso、Chenliang Xu 等 9 位合作者。
- https://arxiv.org/html/2602.01623v1 —— Omni-Judge 论文 HTML 全文（官方一手）。提取 Qwen3-Omni（30B/3B 激活）裁判模型、9 项评测维度、300 条 VidProM 提示词 + Sora 2/Veo 3 各生成 600 条视频、6 名博士生 1–5 分标注协议、逐维度 Kendall τ_b 与 Spearman ρ 相关性结果、作者机构（罗切斯特大学 / 密歇根大学安娜堡分校）。
- https://liangsusan-git.github.io/project/omni_judge/ —— Omni-Judge 官方项目页（官方一手）。

## 数据规模与分布

### 训练数据量级（视频条数/小时数/token数，预训练与SFT分开）

五者均为评测/元评测规模，量级远小于训练集，但 AVBench 与 PhyAVBench 附带了可观的数据资产：
【VABench】共 1,299+ 条测试用例：T2AV 778 条提示词、I2AV 521 条（含配对图像）、其中立体声专项 116 条；每条样本附 3–7 个音频问答对（AQA）与 3–7 个视觉问答对（VQA）。
【AVBench】评测集 470 条 ≥720p 高清提示词（Normal 子集 350 条，1–2 说话人；Hard 子集 120 条，3–4 说话人 + 语音重叠 + 噪声背景）。评测器训练集规模显著更大：自 OpenHumanVid 抽取 30K 真实片段，按维度扩展为每维度 100K 对，三类一致性维度合计 300K 条监督样本（全部为带硬负例的偏好对）。
【AV-SyncBench】3,269 条 in-the-wild 视频，扩展为 38,390 条评测样本（时序挑战 37,569 样本 / 2,717 视频；语义挑战 821 样本 / 552 视频；其中人声音色替换 592、乐器音色迁移 229）。
【PhyAVBench】数据集 PhyAV-Sound-11K：11,605 条全新录制视频，累计 25.5 小时；337 组受控配对提示词；184 名参与者出镜/操作；每组配对提示词平均配备约 17 条 ground-truth 真实录制视频（论文要求 N≥20 用于均值降噪，实际平均 17）。
【Omni-Judge】300 条提示词（取自 VidProM 真实用户提示词库），由 Sora 2 与 Veo 3 各生成 1 条，合计 600 条生成视频，配套 600×9 维度的人工评分。

### 数据来源构成（自有/公开数据集/网络爬取/授权采购/合成数据）

【VABench】T2AV 侧为 LLM + 专家模板批量合成的提示词（无真实视频来源）；I2AV 侧为人工精选并做隐私筛查的高质量图像，再由多模态大模型生成统一音视描述。属于「合成提示词 + 人工策展图像」构成。
【AVBench】评测提示词来自独立提示词池（≥720p 高清真实人物场景）；评测器训练数据来自公开数据集 OpenHumanVid（真实人物视频），负例由 LLM 驱动的扰动与算法性错配合成 —— 即「公开数据集真实正例 + 程序化合成硬负例」。
【AV-SyncBench】全部为公开网络平台抓取的 in-the-wild 视频（未指明具体平台）；扰动样本由算法与语音/乐器转换模型合成。
【PhyAVBench】最具特色 —— 全部视频为「全新录制或采集」，明确目的是避免数据泄漏（zero training-set overlap），在可控环境下拍摄，并跨不同个体、演示者与录制设备采集多样本。不使用任何现成公开数据集。
【Omni-Judge】提示词来自 VidProM（真实用户提示词画廊），视频为 Sora 2 与 Veo 3 现场生成。

### 数据合规与溯源（授权数据占比、rights-cleared数据集、C2PA等） ⚠️

整体披露较弱，仅 PhyAVBench 做了系统性设计：
【PhyAVBench】通过「全部自录 + 零训练集重叠保证」从根本上规避版权与数据泄漏问题，是五者中溯源最干净的；涉及 184 名出镜参与者，但论文未披露肖像授权与知情同意的具体流程[不确定]。
【VABench】I2AV 图像明确经过隐私筛查（privacy-screened）后才纳入；未披露图像版权来源。
【AV-SyncBench】数据来自公开网络平台，论文未讨论版权授权或 C2PA 等溯源机制[不确定]。
【AVBench】训练数据基于公开数据集 OpenHumanVid，继承其许可证；评测提示词池来源未详述。
【Omni-Judge】使用 VidProM 提示词（该数据集自身为 CC BY-NC 类许可），生成视频版权归属各商业模型服务条款约束。
五者均未提及 C2PA / 内容水印溯源标准。

### 片段时长分布与切分策略 ⚠️

【VABench】被测视频时长受各模型默认设置约束而非基准强制：Sora2 为 10 秒 @30FPS；Veo3-fast、Wan2.5、Seedance、Wan2.2、Kling 为 5–8 秒 @24FPS。Synchformer 的 Desync 指标仅取首尾各 4.8 秒窗口计算，这一 4.8s 窗口设定是其时序评测的关键切分策略。
【AV-SyncBench】切分策略最细致：clip 时长 3–13 秒；评测时统一将视频与音频切成互不重叠的 0.64 秒 chunk，逐 chunk 提取视听嵌入后取对角线相似度均值 —— 0.64s 是其时序分辨率的基本单元。
【PhyAVBench】11,605 条视频 / 25.5 小时，平均约 7.9 秒/条。
【AVBench】评测提示词对应的生成时长未单独规定[不确定]；训练用的 OpenHumanVid 片段切分策略未披露[不确定]。
【Omni-Judge】Sora 2 / Veo 3 默认输出时长（约 8–10 秒）[不确定]。

### 分辨率/宽高比分布与分桶策略 ⚠️

【VABench】统一要求 720P（或最接近的宽高比档位），音频统一 48kHz 立体声抽取保留；帧率随模型默认 24–30 FPS，未做强制归一。
【AVBench】评测提示词明确要求 ≥720p 高清（HD prompts），这是其「真实人物场景」定位的前置门槛。
【AV-SyncBench】评测前统一归一化：视频解码为 25 FPS，音频重采样为 16 kHz；原始分辨率与宽高比分布未披露[不确定]。
【PhyAVBench】录制分辨率与设备型号未在论文中给出具体分布[不确定]，仅说明跨多种录制设备采集以增加多样性。
【Omni-Judge】沿用 Sora 2 / Veo 3 默认输出规格[不确定]。
可反向借鉴的分桶启示：AV 评测普遍收敛到 720P + 5–10s + 48kHz（生成端）/ 16kHz + 25FPS（判别端）两套规格，训练数据分桶可对齐这两组锚点。

### 类别/domain分布与配比策略（人物、动作、场景、风格等比例控制与概念均衡） ⚠️

本项是本次调研对训练数据侧最有反向指导价值的字段 —— 五个基准的类目体系可直接作为训练数据 domain 配比的标准坐标系。

【VABench 七大类目体系（最完整的内容侧分类学）】
1. 动物（Animals）：物种特异性发声与视听行为一致性；
2. 人声（Human Sounds）：细分为语言性（含语义内容，涉及唇同步）与非语言性（生理/动作类，如咳嗽、鼓掌、脚步）；
3. 音乐（Music）：跨曲风的结构化音频，考察旋律与节奏连贯性；
4. 环境音（Environmental Sounds）：再分自然、城市、室内三大声景；
5. 同步物理声（Synchronous Physical Sounds）：即时/节律性物理交互声，要求严格遵循材质属性；
6. 复杂场景（Complex Scenes）：高阶场景，含五个子维度 —— 复杂声景、主观感受、世界知识、符号联想、画外（不可见）声源；
7. 虚拟世界（Virtual Worlds）：超越物理定律的非现实场景，仅用于 T2AV 任务。
数据量分配：778 条 T2AV + 521 条 I2AV 按上述类目层级分布（论文以 sunburst 图呈现，未给出逐类目精确条数[不确定]）。注意第 7 类「虚拟世界」不参与 Audio Realism / Visual Realism 两项物理合理性打分 —— 这一「写实类目与风格化类目分离评分」的设计，同样适用于训练数据的分域质量门槛设定。

【AV-SyncBench 十场景体系（同步性视角的正交切分）】
动作（Action）、动物声（Animal Sound）、物体声（Object Sound）、环境声（Ambient Sound）、群体发声（Group Vocalization）、单人说话（Single Speaker）、对话（Dialogue）、演唱（Singing）、单乐器（Single Instrument）、合奏（Ensemble）。上层归并为 Voice / Music / Sound 三大音频类。该体系的价值在于按「同步难度」而非「内容主题」切分：单人说话与对话、单乐器与合奏的区分，直接对应训练数据中多声源混叠场景的配比需求。

【PhyAVBench 六大物理维度 + 41 个细粒度测试点（物理正确性视角）】
① 声源力学（材质硬度/阻尼、密度、表面纹理、物体几何尺寸/形状/厚度、接触动力学如撞击速度/接触面积/激励连续性）；② 流体与空气动力学（流速、液体撞击/飞溅、亥姆霍兹共振、容器材质、粘度、气泡、气动啸声）；③ 声传播环境（混响的空间体积与表面吸声、回声、空间切换、衍射、散射、水下声学、固体传声、真空、遮挡与隔声）；④ 观察者物理（距离平方反比律、空气吸收、多普勒效应的接近/远离与旋转声源、双耳水平/垂直定位）；⑤ 时间与因果（光声速差导致的远场延迟与近场同步、瞬态同步的起振与止声、周期/非周期运动的节奏一致性）；⑥ 复杂耦合与极端物理（相变如沸腾/结冰碎裂、爆炸与冲击波的超音速与非线性失真）。这套体系可直接作为训练数据「物理声学覆盖度」的 checklist。

【AVBench 场景分层】以真实人物场景为核心，按说话人数量与声学难度分层：Normal（1–2 说话人）350 条 / Hard（3–4 说话人 + 语音重叠 + 噪声背景）120 条，并施加 Hard Quota-Based Greedy Sampling 采样约束，强制任一单属性占比 ≤50% —— 这是明确的概念均衡（concept balancing）机制，可直接移植为训练数据采样时的属性配额控制策略。

【Omni-Judge】不设内容类目，其 300 条提示词直接取自 VidProM 真实用户分布，代表「真实用户意图分布」这一另类的 domain 基准。

### 音频类别分布与配比（语音/音效foley/音乐/环境音/静音的比例及控制策略）——AV模型独有维度 ⚠️

AV 独有维度上，五者提供了三套互补的音频类目划分：

【AV-SyncBench 三分法】Voice / Music / Sound，是最简洁可操作的顶层音频分类。其十场景映射关系为：Voice ← 单人说话、对话、群体发声、演唱；Music ← 单乐器、合奏、演唱（跨类）；Sound ← 动作、动物声、物体声、环境声。语义挑战任务也按此分流：人声走音色替换（OpenVoice V2），乐器走音色迁移（预训练 DDSP）—— 说明两类音频的语义扰动机制本质不同，训练数据的语义对齐过滤也应分流处理。样本量上时序挑战（37,569）远多于语义挑战（821），反映真实数据中语义错配样本更难构造。

【VABench 内容驱动划分】动物声、人声（语言性/非语言性）、音乐、环境音（自然/城市/室内）、同步物理声、复杂声景（含画外声源）、虚拟音效。相比三分法更细，且显式引入「画外/不可见声源」这一类目 —— 训练数据中此类样本通常被 on-screen 过滤器误杀，但评测体系承认其存在，提示训练侧应保留一定比例的画外声样本以支持复杂声景生成。

【PhyAVBench Foley 物理向划分】以发声机制而非内容主题分类（固体撞击/机械结构/流体空气动力/传播环境/观察者位置/时序因果），本质是对 Foley 音效类目的深度展开。

【AVBench】以语音为绝对重心（10 维度中 Speech Content Accuracy、Speech Realism、Lip Sync 三项直接针对语音），对应 OpenHumanVid 人物视频数据源，音乐与音效类目基本不覆盖。

五者未给出各自的音频类别精确百分比配比[不确定]，但类目并集（语音 / 音乐 / 物理音效 Foley / 环境声景 / 动物声 / 画外声 / 静音与虚拟音）可直接作为训练数据音频域配比表的行标签。

### 叙事结构分布（单镜头vs多镜头、平均clip时长、镜头数分布、是否含原生音轨）

五者均聚焦单镜头短片段评测，未构建多镜头叙事评测：
【VABench】5–10 秒单镜头生成，但「复杂场景」类目通过复杂声景、世界知识、符号联想等子维度间接考察叙事表达力；Module 2 中的 Expressiveness（叙事有效性与情绪对齐）与 Artistry（视听融合的美学表现力）两项是仅有的叙事层面指标。
【AV-SyncBench】3–13 秒 in-the-wild 片段，全部含原生音轨（这是纳入的前置条件），单镜头为主；对话（Dialogue）与合奏（Ensemble）场景隐含多声源但非多镜头。
【PhyAVBench】受控录制的单镜头短片段，全部含原生同期声。
【AVBench】真实人物场景单镜头，Hard 子集的 3–4 说话人重叠语音是其最接近多主体叙事的设定。
【Omni-Judge】Sora 2 / Veo 3 默认单提示词生成，未涉及多镜头。
整体而言，长时序、多镜头、跨镜头音轨连续性仍是当前 AV 评测体系的空白区，训练数据侧若已在做多镜头叙事数据，暂无对应基准可验证。

### 语言/口音分布（多语种唇同步能力的数据基础） ⚠️

披露普遍薄弱，是五个基准共同的短板：
【AVBench】唇同步与语音内容准确率（Speech Content Accuracy）为核心维度，但论文未披露评测提示词与 OpenHumanVid 训练片段的语种与口音构成[不确定]。
【VABench】唇同步指标仅施加于「人声-语言性」子集中检测到说话人头部的样本；未披露多语种覆盖情况[不确定]。
【AV-SyncBench】人声类场景（单人说话/对话/群体发声/演唱）语义扰动使用 OpenVoice V2 做音色替换（该工具本身支持跨语种音色克隆），但基准未按语种/口音分层统计[不确定]。
【PhyAVBench】以非语音的物理音效为主，语音仅在 WER 指标（Whisper-Large V3）中间接涉及；184 名参与者的语言背景未披露[不确定]。
【Omni-Judge】未涉及语种维度[不确定]。
结论：多语种/多口音唇同步目前无公开基准可对标，训练数据的语种配比无法用现有基准反向验证。

## 清洗流程

### 清洗漏斗整体结构（几级过滤、各级顺序）

五者的「清洗流程」实为评测数据策展流程，但其漏斗结构对训练数据 pipeline 有直接借鉴价值：

【PhyAVBench 五阶段策展流程（最完整）】① 音频物理知识调研 —— LLM 头脑风暴候选物理原理，人类专家剔除不可行/冗余/无关条目；② 分类体系构建 —— LLM 生成层级结构，专家审核消歧去冗；③ 物理约束提示词对设计 —— LLM 生成候选模板，专家人工核验并改写，保证每对提示词仅在单一物理变量上有差异、其余非目标条件全部不变，并避免主观描述与对预期声学结果的显式提示；④ 真实音视频采集 —— 全新录制，可控环境，跨个体/演示者/设备多样采样；⑤ 迭代式质控与过滤 —— LLM 先做语义歧义与非预期物理混杂因素的初筛，人类复核物理一致性、文本-音频-视频三方对齐与现实合理性，问题样本删除、修订或重新采集。这是一条典型的「LLM 生成 → 专家修订 → 采集 → LLM 初筛 + 人工复核 → 迭代回流」闭环。

【AV-SyncBench 三级漏斗】① 网络采集 in-the-wild 视频；② Gemini 3 Flash 自动过滤 —— 剔除画外声源（off-screen sound sources）样本与明显视听错配样本；③ 人工复核 —— 5 名标注员，每条 clip 至少由 3 人独立审核，确认主声源在画面内可见，并剔除音质差、噪声过大、语义模糊的片段。之后进入第四步：程序化扰动生成时序与语义负样本。这是本次调研中「大模型初筛 + 多人交叉人工复核」的标准范式样板。

【VABench 双路策展】T2AV 路：LLM + 专家模板批量生成原始提示词 → LLM 结构化解耦为视觉子提示与听觉子提示 → 生成 VQA/AQA 问答对 → 人工核验类目归属正确性、要素可观测性、物理与常识约束满足性。I2AV 路：人工精选并分类高质量图像（含隐私筛查）→ 多模态 LLM 生成统一视听描述（视觉客观 + 音频常识推断）→ 描述同时用于构造 VQA/AQA 并由 LLM 解耦为子提示 → 人工复核听觉推断的合理性与问题的区分度。论文明确表述「employed human workers and large language models to filter testing samples and adjust the distribution of test data」——即人机协同同时承担过滤与分布调整两项职责。

【AVBench 两条流水线】评测集路：从提示词池按 Hard Quota-Based Greedy Sampling 采样 470 条 ≥720p 提示词，配额约束单属性占比 ≤50%，再分层为 Normal/Hard 子集。训练集路：OpenHumanVid 抽取 30K 真实片段作正例 → LLM 驱动扰动 + 算法性错配生成硬负例 → 每维度扩展至 100K 对 → 三维度合计 300K。

【Omni-Judge】无数据清洗流程，属元评测：300 条 VidProM 提示词 → Sora 2 / Veo 3 各生成 1 条 → 6 名博士生按 9 维度打分 → 计算 Omni-LLM 判分与人类判分的相关性。

### 漏斗定量保留率（每级过滤的输入/输出量与最终保留率，如Apollo 27%） ⚠️

五个基准均未公布逐级过滤的输入/输出量与保留率数字[不确定]。可间接推算的仅有：
【AV-SyncBench】最终保留 3,269 条视频，但网络采集的原始池规模未披露，故 Gemini 3 Flash 初筛与 5 人复核两级的保留率无法计算[不确定]。
【PhyAVBench】11,605 条视频对应 337 组配对提示词，平均每组约 17 条 ground-truth 视频；论文设计目标为每组 N≥20 条，实际均值 17 说明质控阶段存在明显淘汰（粗估保留率约 85%，属推算而非论文披露[不确定]）。
【AVBench】30K 真实片段 → 300K 训练对属扩增而非过滤，无保留率概念；470 条评测提示词的原始候选池规模未披露[不确定]。
【VABench】1,299 条最终样本对应的候选生成量未披露[不确定]。
相较于训练侧数据集（如 Apollo 公布 27% 端到端保留率、MOVA 公布逐级保留率表），评测基准论文普遍不披露漏斗定量信息，这是本类文献的通病。

### 镜头切分方法（PySceneDetect/自研模型/shot-aware splitting） ⚠️

五者均不涉及传统意义上的镜头切分（PySceneDetect 类），因为素材本身即为短单镜头片段。相关的时间轴切分策略为：
【AV-SyncBench】统一切成 0.64 秒互不重叠 chunk 用于视听嵌入的逐段对齐计算，这是其时序评测的基本粒度（对应 CAV-MAE / Synchformer 类模型的原生窗口设定）。
【VABench】Desync 指标仅取生成视频的首 4.8 秒与末 4.8 秒两个窗口送入 Synchformer 预测偏移，规避中段长时序不稳定问题。
【PhyAVBench】录制阶段按「单一物理事件」为单位切分，每条视频对应一次完整的发声事件（如一次撞击、一段流水），属语义事件级切分而非镜头级。
【AVBench / Omni-Judge】未涉及切分[不确定]。

### 质量过滤（美学评分、清晰度、OCR文字过滤、黑边/水印/logo检测） ⚠️

各基准的质量把关手段：
【AVBench】质量维度本身即是可复用的过滤器，其 6 项单模态质量维度分别绑定成熟工具：音频质量用 NISQA MOS、音频美学用 Audiobox、视频技术质量用 DOVER++、视频美学单列一维，另加 Speech Content Accuracy 与 Speech Realism 两项语音专项。论文明确指出其连续可微分打分可直接用作数据过滤机制与 RLHF 奖励信号 —— 这是本次调研中最直接可移植为训练数据质量过滤器的基准。
【VABench】Module 1 的单模态音频质量三件套：SpeechClarity 用 DNSMOS 检测背景噪声、SpeechQual&Nat 用 NISQAv2 评整体质量与自然度、AudioAesthetic 用 Audiobox 评愉悦度/有用性/制作复杂度与质量。视觉侧质量并入 MLLM 打分的 Visual Realism 与 Artistry。此外立体声专项提供 9 项声学质量指标：空间成像质量（立体声宽度、成像稳定性、电平稳定性、声道间一致性）与信号完整性/兼容性（相位相干性分低/中/高三频段评估、单声道兼容性 Mono Compat = 1 − 归一化单声道损失、方向一致性、Mid/Side 能量比测声场宽度）。
【AV-SyncBench】人工阶段明确剔除「音质差、噪声过大、语义模糊」三类样本，即音频 SNR、清晰度与语义可判别性三重门槛（未给出量化阈值[不确定]）。
【PhyAVBench】质控聚焦物理正确性而非画面美学：LLM 初筛语义歧义与非预期物理混杂因素，人工复核物理一致性与现实合理性；因是可控环境自录，画面清晰度、水印、黑边等问题天然不存在。
【Omni-Judge】不做过滤；但其结论对质量过滤有警示意义 —— Omni-LLM 在 video quality 维度上与人类的相关性极低（Kendall τ_b ≈ 0.020），说明用 Omni-LLM 直接替代传统美学/技术质量打分器做数据过滤是不可靠的，画质类过滤仍需专用模型。
未提及 OCR 文字过滤、水印/logo 检测等训练侧常见手段[不确定]。

### 运动过滤（光流/运动分数阈值、静态与抖动剔除） ⚠️

五者均未采用光流或运动分数阈值做过滤[不确定]，但存在若干功能等价的替代机制：
【AV-SyncBench】Local Jitter 挑战任务（30–700 ms 三档严重度的局部抖动扰动）与 Global Speed Change（0.8×–1.25× 共 10 档变速）本质是在人为构造「运动-音频时间基不一致」的负样本，可反向用作训练数据中变速/抽帧异常样本的检测思路。
【PhyAVBench】Time and Causality 维度下的「周期运动 / 非周期运动节奏一致性」测试点，以及 Transient Synchronization（起振与止声）测试点，考察的正是运动与声音的因果一致性，比单纯运动幅度阈值更语义化。
【VABench】未做运动过滤；生成侧被测模型的静态画面问题由 Visual Realism 与 Artistry 打分间接惩罚。

### 去重方法（精确去重与基于embedding的语义去重分别记录） ⚠️

五者均未披露显式的精确去重或 embedding 语义去重流程[不确定]。相关的近似机制为：
【PhyAVBench】采用「反向去重」思路 —— 主动保证与现有训练集零重叠（全部新录制），并在采集时刻意跨不同个体、演示者与录制设备取样以提升样本内多样性，这是从源头避免同质化而非事后去重。
【AVBench】Hard Quota-Based Greedy Sampling 强制任一单属性占比 ≤50%，功能上等价于属性层面的去冗余/均衡采样。
【AV-SyncBench / VABench / Omni-Judge】未提及去重[不确定]。

### VLM/LLM作为质检员（多模态大模型质量打分与错配剔除，2026年从浅层打分器转向大模型语义判断的趋势）

这是本次调研中五个基准最具训练侧参考价值的字段，五者恰好构成「大模型作质检员」这条 2026 年趋势的完整证据链：

【AV-SyncBench —— Gemini 作前置质检员的实证】明确使用 Gemini 3 Flash 作为自动过滤第一关，任务是剔除「画外声源」与「明显视听错配」两类样本，随后才交人工复核。这是目前公开文献中少见的、把商业闭源多模态大模型直接嵌入数据 pipeline 首级过滤的做法，其成本-效果权衡（用 Flash 级轻量模型做高召回粗筛，再用人力做高精度精筛）对大规模训练数据清洗有直接可移植性。

【AVBench —— 训练专用评测器替代通用大模型】走了另一条路：不用通用 MLLM 直接打分，而是通过偏好学习训练专用评测器。VT 与 AV 维度基于 Qwen2.5-Omni (7B) 仅微调 LLM 部分；AT 维度基于 Qwen2-Audio (7B) 微调 LLM 与 connector 层。训练约束设计巧妙：模型仅输出单 token（Yes/No），通过 token 概率比归一化得到连续分数，从而把离散判断转成连续可微信号，可直接用于数据过滤与 RLHF reward。这解决了 LLM-as-judge 打分离散、方差大、不可微的核心痛点，是训练侧最值得复用的技术细节。

【VABench —— 通用 MLLM 做语义层评测】Module 2 使用 Qwen2.5-Omni 7B 承担 5 项宏观打分（Alignment / Artistry / Expressiveness / Audio Realism / Visual Realism，1–5 分）与 2 项微观问答（每样本 3–7 条音频 QA 与 3–7 条视觉 QA）。「宏观打分 + 微观 QA 双层」结构值得借鉴：细粒度 QA 把模糊的整体评分拆解为可验证的事实判断，显著降低 MLLM 打分的主观漂移。

【Omni-Judge —— 能力边界的系统性测绘】最重要的负面结论提供者。以 Qwen3-Omni（30B 总参 / 3B 激活）为对象，对比 instruct 版与 reasoning 增强版，在 9 个维度上与 6 名博士生标注做相关性分析。结论清晰分层：语义类维度可用 —— audio-text alignment 在 Sora 2 子集上达 τ_b=0.292 / ρ=0.345，audio-video-text 三模态一致性 0.139/0.151；感知类维度不可用 —— video quality τ_b≈0.020，audio-video synchronization 仅 0.142，作者归因于 Omni-LLM 的时间分辨率不足。对训练数据 pipeline 的直接指导：Omni-LLM 适合承担「语义匹配/错配剔除」，但时序同步与画质判定必须交给 Synchformer、DOVER++ 这类专用模型。此外论文展示了利用 Omni-Judge 可解释反馈做生成模型「反馈式修正」的应用（基于识别出的错误生成修正帧），但未主张用于训练数据过滤。

【PhyAVBench —— LLM 在知识构建与初筛两端介入】LLM 用于物理知识头脑风暴、分类体系生成、提示词模板生成，以及质控阶段的语义歧义与混杂因素初筛；但所有 LLM 产出均经人类专家审核修订，形成严格的「LLM 提效 + 专家把关」分工。

综合判断：2026 年的共识是「大模型判语义、专用模型判感知、人类判物理与最终边界」三层分工，而非用大模型一把梭。

### 安全与合规过滤（NSFW、版权、人脸/隐私） ⚠️

披露有限：
【VABench】I2AV 路的高质量图像在人工精选与分类阶段做了隐私筛查（privacy-screened），是五者中唯一明确提及隐私过滤的。
【AV-SyncBench】人工阶段剔除语义模糊样本，未提及 NSFW/版权/人脸隐私专项过滤[不确定]。
【PhyAVBench】自录数据涉及 184 名参与者出镜，肖像权与知情同意流程未披露[不确定]；因内容为受控物理演示，天然规避 NSFW 风险。
【AVBench】基于公开数据集 OpenHumanVid，继承其安全过滤；自身未追加安全过滤描述[不确定]。
【Omni-Judge】使用商业模型 API 生成，安全过滤由 Sora 2 / Veo 3 自带的内容安全机制承担。
五者均未提及 NSFW 分类器、版权检测或人脸匿名化的具体工具与阈值[不确定]。

## 打标方式

### 使用的caption模型（自研VLM/开源模型，模型规模） ⚠️

评测基准侧的「打标」主要指提示词构造与问答对生成，所用模型如下：
【VABench】提示词生成使用通用 LLM（具体型号未指名[不确定]）配合专家模板；I2AV 路使用多模态 LLM 从图像生成统一视听描述（型号未指名[不确定]）；评测阶段的 MLLM 打分器明确为 Qwen2.5-Omni 7B。
【AVBench】评测器为自训练模型：Qwen2.5-Omni 7B（VT、AV 维度，仅微调 LLM 部分）与 Qwen2-Audio 7B（AT 维度，微调 LLM + connector）；硬负例构造使用 LLM 驱动扰动（型号未指名[不确定]）。
【AV-SyncBench】数据质控使用 Gemini 3 Flash；语义扰动使用 OpenVoice V2（人声音色替换）与预训练 DDSP（乐器音色迁移）；不涉及 caption 生成。
【PhyAVBench】LLM 用于知识调研、分类体系构建、提示词模板生成与质控初筛（型号未指名[不确定]）；语音任务的转写使用 Whisper-Large V3 计算 WER。
【Omni-Judge】被测裁判模型为 Qwen3-Omni（30B 总参 / 3B 激活），对比 instruct 与 reasoning 增强两个变体。

### caption密度与结构化程度（短/长/密集描述、结构化字段如镜头运动、风格标签） ⚠️

各基准的提示词/标注结构化程度差异显著，对训练侧 caption 设计有直接映射：
【VABench 结构化解耦最彻底】原始提示词由 LLM 结构化解耦为「视觉子提示」与「听觉子提示」两条独立流，再据此分别生成 VQA（3–7 条）与 AQA（3–7 条）细粒度问答对。即一条样本携带四类结构化字段：完整提示词 + 视觉子提示 + 听觉子提示 + 分模态问答对。这种「整体描述 → 分模态解耦 → 可验证问答」的三层结构，可直接迁移为训练数据的分层 caption schema。
【PhyAVBench 单变量受控提示词】337 组配对提示词，每组两条仅在单一物理变量上不同，且刻意避免主观描述、避免显式提示预期声学结果 —— 这是极端结构化的「受控变量式 caption」，本质是把物理属性作为可枚举的标签维度写进文本。
【AVBench】470 条 ≥720p 真实人物场景提示词，按说话人数量与声学复杂度分层标注；采样时按属性配额控制，暗示提示词已被打上可枚举的属性标签（说话人数、是否重叠语音、背景噪声等级）[具体标签体系未完整披露，不确定]。
【AV-SyncBench】不生成 caption，但每条样本携带结构化的扰动元数据：扰动类型（全局偏移/局部抖动/全局变速/人声音色替换/乐器音色迁移）+ 扰动强度档位 + 场景类目 + 音频大类。这套「扰动标签」结构对构造训练侧同步性负样本极具参考价值。
【Omni-Judge】直接使用 VidProM 真实用户提示词，不做结构化改写 —— 代表真实用户 caption 的自然分布（通常短、模糊、缺听觉描述）。

### 音视频联合caption结构（是否同时覆盖视觉+听觉轨道、是否分流为独立字段，如LTX-2全音景描述、Script-a-Video factorized streams、Foley-Omni三字段）

本字段是评测体系对训练侧 caption schema 最直接的反向指导：
【VABench 的 factorized 双流方案】明确将统一的视听描述由 LLM 解耦为 visual sub-prompt 与 auditory sub-prompt 两条独立字段流，并分别生成 VQA / AQA 问答对做交叉验证。这与训练侧 Script-a-Video 的 factorized streams、Foley-Omni 三字段方案属同一设计哲学。特别值得注意的是 I2AV 路的做法：多模态 LLM 从静态图像生成的描述中，视觉部分要求客观可观测，音频部分要求基于常识推断（commonsense-inferred audio），然后由人工复核听觉推断的合理性 —— 这为「无音轨视频/图像如何补齐音频 caption」提供了可操作范式。
【VABench 的可观测性约束】人工核验环节明确检查「要素可观测性」与「物理/常识约束满足性」，即 caption 中的听觉描述必须是从画面可合理推断的，不得凭空捏造。这一约束可直接作为训练数据音频 caption 的质检标准。
【AVBench 的三维一致性拆分】把视听文本关系拆为 AT（音频-文本）、VT（视频-文本）、AV（音频-视频）三条独立一致性维度，各自训练专用评测器 —— 说明 joint caption 的质量应按这三条边分别度量而非合并成单一分数。
【AV-SyncBench 的正交拆分】进一步将 AV 边拆为「时序对齐」与「语义匹配」两个正交子维度，这意味着完整的 joint AV caption 质检需要至少 4 条边：AT、VT、AV-时序、AV-语义。
【PhyAVBench】提示词中音频属性以物理因果形式隐式表达（描述物理条件而非直接描述声音），是另一种 schema 思路：让模型从物理条件推导声学结果，而非直接给出声音标签。
【Omni-Judge】评测的 9 维度中包含 audio-video-text 三模态联合一致性（tri-modal coherence），是唯一显式定义「三模态联合」这一 caption 质量指标的基准。

### 对白转写与说话人属性标注（ASR转写+说话人身份/语言/口音/情绪） ⚠️

【AVBench 覆盖最全】10 维度中 Speech Content Accuracy（语音内容准确率，隐含 ASR 转写比对）、Speech Realism（语音真实度）、Lip Sync Consistency（唇同步一致性）三项直接针对对白；硬负例构造涵盖说话人身份错配（speaker identity mismatches）与情绪极性反转（emotional polarity reversals）—— 说明其数据标注中包含说话人身份与情绪标签。Hard 子集标注了说话人数量（3–4 人）与语音重叠属性。
【PhyAVBench】语音任务使用 Whisper-Large V3 计算 WER 作为内容准确性指标。
【VABench】唇同步指标借鉴 LatentSync 方法计算对齐置信度，仅施加于检测到 talking head 的「人声-语言性」子集。
【AV-SyncBench】人声场景使用 OpenVoice V2 做音色替换构造语义负样本，隐含说话人音色/身份属性的可控编辑能力；场景标签区分单人说话、对话、群体发声、演唱四类。
【Omni-Judge】未单列对白维度[不确定]。
口音、语言标签的标注情况五者均未披露[不确定]。

### 几何与结构化标注（相机参数、深度、3D point tracks、动作标注、显式状态） ⚠️

【PhyAVBench 最丰富】其 41 个细粒度测试点本质就是一套结构化物理属性标注体系：材质硬度/阻尼、密度、表面纹理、物体尺寸/形状/厚度、撞击速度、接触面积/锐度、转速、松紧度、张力、流速、粘度、空间体积、表面吸声、传播介质、声源距离、方位（水平/垂直定位）等，均为可枚举的显式状态标注。此外 Observer Physics 维度隐含相机/听者位置与距离参数，Binaural Effect 测试点涉及双耳空间方位标注。
【VABench 立体声子集】116 条提示词显式指定左右声道空间方位，等价于声源方位的结构化标注；评测侧计算 Mid/Side 能量比、相位相干性、声道间一致性等空间几何量。
【AV-SyncBench】标注扰动的精确时间参数（偏移 50–500ms 五档、抖动 30–700ms 三档、变速 0.8×–1.25× 十档），是时间轴上的结构化标注。
【AVBench】硬负例含时间偏移 0.2–3.0s 的精确标注。
五者均未涉及相机参数、深度图、3D point tracks 等视觉几何标注[不确定]。

### 合成数据构造（受控扰动/编辑构造训练对，如InstructAV2AV） ⚠️

本字段是 AVBench 与 AV-SyncBench 的核心方法论，对训练侧构造受控数据对极具价值：
【AVBench 的硬负例合成】从 OpenHumanVid 抽取 30K 真实片段作正例，通过 LLM 驱动扰动 + 算法性错配，按维度扩展至每维度 100K 对、合计 300K 条监督样本。硬负例类型明确列举五类：① 时间偏移（0.2–3.0 秒）；② 声学损坏（音高变换、变速）；③ 说话人身份错配；④ 情绪极性反转；⑤ 状态转换（state transitions）。这套「真实正例 + 五类可控降级负例」的配方可直接复用于训练评测器或数据过滤器。
【AV-SyncBench 的双轴扰动合成】时序轴三种扰动：全局偏移（50–500 ms，五个档位）、局部抖动（30–700 ms，三个严重度）、全局变速（0.8×–1.25×，十个档位）；语义轴两种扰动：人声音色替换（OpenVoice V2）与乐器音色迁移（预训练 DDSP），关键设计是语义扰动强制保持时间不变性（temporal invariance）—— 编辑后的音频与原音频时序完全一致、仅语义属性改变，从而实现两个轴的严格解耦。这是「正交扰动构造正交评测维度」的教科书式做法。
【PhyAVBench 的单变量配对】337 组配对提示词，每组仅单一物理变量不同、其余条件全部不变，属于文本侧的受控变量合成；对应的真实视频则为实拍而非合成。
【VABench】提示词层面为 LLM 合成，但视听内容全部由被测模型生成，不做人工降级构造[不确定是否含负样本设计]。
【Omni-Judge】不构造合成数据。

### 人工介入程度（人工标注、人工质检、模型初筛+人工复核） ⚠️

五者人工介入程度差异明显，构成从「重人工」到「轻人工」的完整光谱：
【PhyAVBench 人工介入最重】人类专家贯穿全部五个阶段：剔除 LLM 生成的不可行/冗余物理原理、审核分类体系消歧去冗、逐条核验并改写配对提示词以保证单变量差异、亲自组织 184 名参与者在可控环境录制 11,605 条视频、以及质控阶段复核物理一致性与三方对齐并对问题样本做删除/修订/重采集。主观评测环节 PVR-MOS 动用 74 名评分员按 1–5 分评估音频差异与物理变化的对应程度。
【AV-SyncBench 交叉复核机制最规范】5 名人工标注员，每条视频片段至少由 3 人独立审核，核验主声源画面可见性并剔除低质样本 —— 「N=5 标注池、每样本 ≥3 人交叉」是可直接复用的质检配置。前置由 Gemini 3 Flash 做粗筛，构成标准的「模型初筛 + 人工复核」两级结构。
【Omni-Judge】6 名具备音视频生成研究经验的博士生对 600 条视频按 4 个方面（质量、语义对齐、时序对齐、美学）打 1–5 分整数分，作为衡量 Omni-LLM 裁判能力的人类基准。
【AVBench】4 名领域专家执行成对比较（2AFC），按维度选出更优模型并允许打平，胜率计算式为 (W + 0.5T)/(W + T + L)，再用 Pearson 相关系数验证自动打分与人类偏好的一致性。
【VABench】明确使用 human workers 配合 LLM 过滤测试样本并调整测试数据分布，人工核验类目归属、要素可观测性、物理常识约束、听觉推断合理性与问题区分度；但未披露标注人数与一致性指标[不确定]。

## 音视频对齐

### 音视频同步检测方法（口型同步、事件对齐）

五个基准共同构成了当前最完整的 AV 同步检测方法论集合：

【AV-SyncBench —— 解耦式同步检测（本次调研核心方法）】统一协议：将视频与音频切成互不重叠的 0.64 秒 chunk，独立提取视觉与音频嵌入，计算时序相似度矩阵的对角线元素余弦相似度并取均值 S = (1/N)Σ sim(v_i, a_i)；判定采用二值准确率 —— 原始配对的得分是否高于扰动配对。时序侧考察三类扰动的可分性（全局偏移、局部抖动、全局变速），语义侧在严格保持时间不变的前提下只改音色/声源，考察语义可分性。被测表征模型五个：Synchformer、SparseSync、ImageBind、CAV-MAE、CAV-MAE-Sync。

【VABench】两条同步线：① Desync —— 用 Synchformer 预测音视频偏移量，仅在首/末各 4.8 秒窗口计算；② Lip-Sync —— 借鉴 LatentSync 方法计算对齐置信度，仅施加于检测到 talking head 的人声-语言性子集。跨模态语义对齐另用三件套：ViCLIP（文本-视频，具时序理解）、CLAP（文本-音频余弦相似度）、ImageBind（音视频联合嵌入空间）。

【AVBench】将 AV 一致性做成可训练的专用评测器（Qwen2.5-Omni 7B 微调），而非依赖固定的 SyncNet 类模型；另设独立的 Lip Sync Consistency 维度。

【PhyAVBench】提出 FGAS（Fine-Grained Alignment Score）—— 帧级视觉与音频 token 的余弦相似度，取时序相似度矩阵对角线元素均值；核心指标 CPRS 使用 CAV-MAE Sync 编码器（后续版本改用 CLAP 嵌入）度量配对样本间声学变化方向与真实参考向量的一致性。

【Omni-Judge】把 audio-video synchronization 作为 9 维度之一交由 Qwen3-Omni 判断，结论为负面：τ_b 仅 0.142，明确指出 Omni-LLM 因时间分辨率不足而不适合承担同步检测。

### 同步检测具体指标与阈值（SyncNet/Synchformer/LSE/自研，阈值数值，如MOVA LSE-D≤9.5且LSE-C≥4.5、SkyReels-V4 SyncNet |offset|≤3∧conf>1.5） ⚠️

具体指标与量化参数（可直接作为训练数据同步过滤的参数参考）：

【AV-SyncBench 的扰动强度谱 —— 最有价值的阈值标定数据】
- 全局偏移：50–500 ms，五个档位；
- 局部抖动：30–700 ms，三档严重度；
- 全局变速：0.8×–1.25×，十个档位；
- 时间粒度：0.64 秒 chunk；视频统一 25 FPS，音频统一 16 kHz。
关键实测结论：在 50 ms 偏移档位上，各 SOTA 同步模型的判别准确率仅约 0.51（接近随机猜测）。这意味着以 Synchformer/SparseSync 类模型做训练数据同步过滤时，其有效分辨率下限约在 50 ms 以上，低于此量级的失配无法可靠检出 —— 对设定同步阈值有直接指导意义。
模型能力分化实测：ImageBind 在音色编辑（语义）任务上总体准确率最高 0.859，但时序对齐任务较弱；SparseSync 呈相反趋势；Synchformer 与 SparseSync 擅长时序偏移检测；CAV-MAE 在局部抖动与变速上较强。结论：单一模型无法同时胜任时序与语义两类过滤，训练 pipeline 应组合使用。

【AVBench 硬负例参数】时间偏移 0.2–3.0 秒（覆盖粗粒度失配区间）；声学损坏含音高变换与变速；评测器输出经 Yes/No 单 token 概率比归一化为连续分数（0–1），天然可设阈值。

【VABench】Desync 基于 Synchformer 预测偏移，计算窗口为首/末各 4.8 秒；MLLM 宏观打分为 1–5 分制；立体声 Mono Compat = 1 − 归一化单声道损失。未公布用于筛选的绝对阈值[不确定]。

【PhyAVBench】CPRS = ½(cosine_similarity + 1)，归一化到 [0,1]，1.0 为完美对齐、0.5 为正交、0.0 为反向；要求每组配对至少 N≥20 条 ground-truth 样本取均值以抑制噪声（实测均值 17）。

五者均未使用 SyncNet/LSE-D/LSE-C 这套经典指标[不确定为何未采用]，而是普遍转向 Synchformer / CAV-MAE-Sync / ImageBind 等更现代的表征模型 —— 这本身是 2026 年同步检测技术栈迁移的信号。

### 时序同步vs语义同步的分离处理（时间对齐与内容语义匹配作为两个独立过滤条件）

AV-SyncBench 是这一理念的首个系统化实现，论文明确宣称为「第一个完全分离音视频同步的时序与语义评测的基准」，其解耦设计对训练数据过滤有直接可移植性：

【解耦的实现机制】关键在于扰动构造的正交性 —— 时序扰动只改时间轴不改音频内容（全局偏移、局部抖动、全局变速），语义扰动则强制施加时间不变性约束（temporal invariance），只替换音色/声源而保持时间结构完全一致（OpenVoice V2 做人声音色替换、预训练 DDSP 做乐器音色迁移）。由于两类扰动在构造上正交，模型在两个轴上的表现可被独立观测。

【解耦的实证价值】实测显示两个轴确实测量不同能力：ImageBind 语义判别强（0.859）但时序对齐弱，SparseSync 恰好相反。这直接证明「一个同步分数走天下」的做法会掩盖模型的能力缺陷，也意味着训练数据过滤时若只用单一同步分数，会同时漏掉「时间对但内容错」和「内容对但时间错」两类坏样本中的一类。

【对训练数据 pipeline 的启示】同步过滤应设为两个独立的串联条件：① 时序对齐条件 —— 用 Synchformer / SparseSync 类模型检出偏移，注意 50ms 以下不可靠；② 语义匹配条件 —— 用 ImageBind / CLAP 类跨模态嵌入检出声画内容错配（如配错的音效、后期贴的背景音乐、画外解说）。

【其他基准的对应设计】VABench 同样做了拆分：Desync（Synchformer 时序偏移）与 Audio-Visual Align（ImageBind 语义对齐）分列两项独立指标，Module 2 的 Alignment 维度则显式定义为「时序同步 + 语义对应」两者的合并判断。AVBench 的 AV Consistency 维度合并了两者未做拆分，其硬负例中时间偏移（时序）与说话人身份错配/情绪反转（语义）虽分属两类但用同一评测器判定。PhyAVBench 的 Time and Causality 维度专攻时序因果（光声速差远场延迟、瞬态起振止声、周期/非周期节奏一致性），而其余五个维度属声学语义正确性，也构成隐式的时序/语义分离。

### 音频质量过滤（SNR、静音检测与静音占比阈值、无音轨剔除、画外音源剔除、背景音乐分离） ⚠️

【AV-SyncBench 的双关卡】第一关 Gemini 3 Flash 自动剔除「画外声源（off-screen sound sources）」样本 —— 这是 AV 训练数据最关键也最难做的过滤项（后期配音、画外解说、贴片背景音乐均属此类）；第二关 5 人交叉人工复核，确认主声源在画面内可见，并剔除音质差、噪声过大、语义模糊的片段。二者共同保证「所见即所闻」，是本次调研中 on-screen 声源过滤的最佳实践样板（未公布 SNR 等量化阈值[不确定]）。
【VABench】音频质量三件套可直接复用为过滤器：DNSMOS 测背景噪声/语音清晰度、NISQAv2 测整体质量与自然度、Audiobox 测音频美学（愉悦度、有用性、制作复杂度与质量）；生成侧统一抽取 48kHz 立体声轨。
【AVBench】NISQA MOS 测音频质量、Audiobox 测音频美学，二者输出连续分数，论文明确其可用作数据过滤机制。Hard 子集刻意保留噪声背景与重叠语音样本，说明其定位是「难而非脏」—— 训练数据过滤时也应区分「噪声大但真实」与「音质坏需剔除」。
【PhyAVBench】受控环境录制从源头保证音质；质控阶段人工复核剔除含非预期物理混杂因素（相当于剔除混入的无关声源）的样本。
【Omni-Judge】设有 audio quality 与 audio aesthetic 两个维度交由 Omni-LLM 判断，但整体结论提示感知类维度的 LLM 判分不可靠。
五者均未提及静音占比阈值、无音轨剔除、背景音乐分离（source separation）的具体做法[不确定]。

### 语音/音效/音乐的分类与分别处理策略 ⚠️

【AV-SyncBench 分类分治最明确】按 Voice / Music / Sound 三大类分流处理：语义扰动上，人声类走 OpenVoice V2 音色替换，乐器类走 DDSP 音色迁移，通用音效类不做语义扰动（故语义挑战样本仅 821 条，远少于时序挑战的 37,569 条）。这说明不同音频类型的可控编辑难度差异极大 —— 语音与音乐有成熟的音色解耦工具，通用 Foley 音效则缺乏，这一现实也约束了训练数据侧构造 Foley 负样本的可行性。十场景标签进一步细分为单声源与多声源（单人说话 vs 对话/群体发声、单乐器 vs 合奏）。
【VABench 按类目施加差异化指标】唇同步指标仅施加于人声-语言性子集且需检测到 talking head；语音质量指标（DNSMOS、NISQAv2）仅对含语音样本有意义；Audio Realism 物理合理性打分明确排除虚拟世界类目。这种「指标按音频类型条件化施加」的机制，对应到训练数据侧即为分域设置不同质量门槛。
【PhyAVBench 专攻 Foley 物理音效】其六大维度全部围绕非语音的物理发声机制展开，语音仅通过 WER 间接涉及；这填补了 Foley 类音效在物理正确性上的评测空白。
【AVBench 专攻语音】10 维度中三项直接针对语音（内容准确率、真实度、唇同步），音乐与通用音效基本不覆盖，是纯人声场景基准。
【Omni-Judge】未按音频类型分层分析[不确定]。
综合：语音 / 音乐 / Foley 音效三类在评测指标、可控编辑工具、数据可得性上均高度异质，训练 pipeline 必须分类分治，不宜用统一阈值。

## 训练配合

### 多阶段训练课程与数据课程调度（阶段划分依据：分辨率/时长/质量分/模态；低清→高清、图像→视频、短→长） ⚠️

五者作为评测基准本身不涉及生成模型的训练课程[不适用]。唯一涉及训练的是 AVBench 的评测器训练：VT 与 AV 评测器基于 Qwen2.5-Omni 7B 采用「仅微调 LLM 部分、冻结其余」的部分参数策略；AT 评测器基于 Qwen2-Audio 7B 采用「微调 LLM + connector 层」策略。二者均为单阶段偏好学习微调，未采用多阶段课程；具体学习率、epoch 数、batch size 等超参未在可获取内容中披露[不确定]。
可反向借鉴的课程设计信号：AVBench 的 Normal（1–2 说话人）/ Hard（3–4 说话人 + 重叠语音 + 噪声）双子集分层、AV-SyncBench 的扰动强度多档位设计（偏移 5 档、抖动 3 档、变速 10 档），本质都是难度分级体系，可直接映射为训练数据的由易到难课程编排依据。

### 数据配比随训练阶段的变化（预训练/退火annealing/SFT高质量子集） ⚠️

五者均不涉及生成模型训练的阶段性配比[不适用/不确定]。可提取的配比相关设计为：
【AVBench】评测器训练数据按维度均匀配比 —— 每个一致性维度各 100K 对，AT/VT/AV 三维度合计 300K，即严格 1:1:1 均衡配比；正负样本比例未披露[不确定]。评测集则按 350:120（约 74.5% : 25.5%）划分 Normal 与 Hard 子集，这一「约 3:1 的常规:困难」比例可作为 SFT 高质量子集中难例占比的参考锚点。
【AV-SyncBench】时序挑战 37,569 样本 vs 语义挑战 821 样本，比例约 46:1，反映的是构造成本而非理想配比。
【VABench】T2AV 778 : I2AV 521 : 立体声 116，约 6.7 : 4.5 : 1。
【PhyAVBench】337 组配对提示词跨 6 大维度 41 个测试点，平均每测试点约 8 组；各维度间的组数分配未逐项披露[不确定]。

### 后训练数据（SFT精选集规模与筛选标准、偏好对数量与标注方式、reward model训练数据） ⚠️

严格意义上五者均不产出生成模型的后训练数据[不适用]，但 AVBench 与 AV-SyncBench 的产出可直接充当后训练资产：
【AVBench 最具后训练价值】其 300K 条带硬负例的偏好对本身就是标准的偏好数据格式；论文明确指出评测器输出的连续可微分数可用作 RLHF 的 reward signal 与数据过滤机制，即该评测器可直接充当 AV 生成模型后训练的 reward model。这是本次调研中唯一显式定位为「可复用 reward model」的基准。其人工偏好标注采用 4 名专家 2AFC 成对比较、允许打平、胜率式 (W + 0.5T)/(W + T + L)，是标准的偏好标注协议。
【AV-SyncBench】38,390 条带精确扰动标签的样本可直接作为同步性判别模型或 sync reward model 的训练/校准数据。
【Omni-Judge】600 条视频 × 9 维度 × 6 名博士生的人工评分构成小规模但高质量的人类偏好校准集，可用于校准自动 reward model 与人类的一致性。
【VABench / PhyAVBench】定位为纯评测，未提供偏好对或 reward 训练数据[不适用]。

### 数据处理基础设施与吞吐（NeMo Curator/Data-Juicer/自研，GPU加速比，处理规模，成本） ⚠️

五者均未披露数据处理基础设施、GPU 加速比、处理吞吐或成本[不确定]，这是评测基准类论文的普遍缺失。可间接观察到的工程量级为：
【PhyAVBench】11,605 条视频 / 25.5 小时全部为新录制，涉及 184 名参与者、多种录制设备与可控环境布置，另有 74 名 PVR-MOS 评分员与 29+ 位作者协作，是五者中人力工程投入最大的（成本数据未披露[不确定]）。
【AV-SyncBench】3,269 条视频经 Gemini 3 Flash 全量调用做初筛，再由 5 名标注员做 ≥3 人交叉复核（约 9,800+ 人次审核），随后程序化生成 38,390 条扰动样本；Gemini API 调用成本未披露[不确定]。选用 Flash 级轻量模型而非 Pro 级，本身即是成本-吞吐权衡的体现。
【AVBench】300K 训练样本的合成与 7B 模型微调所需算力未披露[不确定]；提供 HuggingFace Leaderboard 说明有持续的在线评测服务基础设施。
【VABench】1,299 条样本 × 多个商业模型 API 生成（Veo3、Wan2.5、Sora2、Seedance、Kling 走官方 API，ThinkSound 与 MMAudio 本地部署），API 调用成本未披露[不确定]。
未提及 NeMo Curator、Data-Juicer 等专用数据处理框架[不确定]。

## 效果对比

### 数据策略消融的量化影响（区分：过滤严格度ablation / caption密度风格ablation / 数据配比ablation，及对应评测指标） ⚠️

五者作为评测基准不做训练数据策略消融[不适用]，但提供了大量可用于指导数据策略的诊断性结论：
【PhyAVBench 的模型能力剖面】17 个 SOTA 模型全面评测，最强的 Sora 2 的 CPRS 仅 0.4512（满分 1.0，0.5 即为随机正交水平），说明当前所有模型在音频物理正确性上基本处于「几乎无物理敏感性」状态；V2A 路线中 MMAudio 最优但也仅 0.4003。跨模型一致的弱项是流体动力学与声传播环境两个维度 —— 这直接指出训练数据中缺失的 domain：流水/气泡/粘度类流体声、混响/回声/遮挡/水下/固体传声类传播环境样本。这是最可执行的数据补充清单。
【PhyAVBench 的指标可信度验证】CPRS 与人类判断的 Pearson 相关系数达 0.92，说明该自动指标可靠，可用作数据筛选的代理指标。
【AV-SyncBench 的检测器能力剖面】ImageBind 语义强（0.859）时序弱、SparseSync 时序强语义弱、CAV-MAE 在抖动与变速上强、Synchformer 与 SparseSync 擅长偏移检测；50 ms 细粒度偏移下全体模型准确率跌至约 0.51。这等价于一次「过滤器选型消融」，结论是必须组合多个模型分工过滤。
【AVBench 的对齐度验证】用 Pearson 相关系数验证自动打分与 4 名专家 2AFC 偏好的一致性；其硬负例设计（时间偏移 0.2–3.0s、音高/变速损坏、说话人错配、情绪反转、状态转换五类）经偏好学习后可检出细微跨模态不一致，间接证明「带受控降级的合成负例」这一数据构造策略有效。
【Omni-Judge 的裁判能力消融】按维度逐项测量 Omni-LLM 与人类的相关性，得到清晰的能力分界：语义类维度可用（audio-text τ_b=0.292/ρ=0.345、audio-video-text 0.139/0.151），感知类维度不可用（video quality τ_b≈0.020、AV sync 0.142）；同时对比了 instruct 版与 reasoning 增强版的差异。这是对「用大模型做数据质检」这一策略最直接的量化消融。
【VABench】15 维度 × 8 个模型/组合的横向对比，可读出端到端原生联合生成（Sora2/Veo3/Wan2.5）与级联 V+A 组合在各维度上的系统性差异[具体数值未在可获取内容中完整披露，不确定]。

### 质量vs数量的证据（小而精数据超越大而杂的案例）

【PhyAVBench 是最强证据】仅用 337 组配对提示词、11,605 条精心录制的视频（25.5 小时，量级极小），就把 17 个 SOTA 模型的 CPRS 全部压在 0.45 以下，暴露出用海量网络数据训练的商业大模型在基础音频物理上的系统性失败。其核心方法论主张是「新录制以保证零训练集重叠」—— 强调数据的纯净性与受控性优先于规模，且明确要求每组配对至少 N≥20 条真实样本取均值以抑制噪声，体现「少而精 + 多次重复采样」的思路。
【AVBench 的 30K→300K 扩增思路】不是靠采集更多数据，而是对 30K 条高质量真实片段施加受控降级扩增到 300K 训练对，用合成负例的多样性替代真实数据的规模，是「小种子 + 可控合成」的质量优先路线。
【AV-SyncBench】3,269 条经 Gemini 初筛 + 5 人交叉复核的干净视频，扩展出 38,390 条评测样本；其严格的 on-screen 声源要求意味着大量原始数据被剔除，换取标签的绝对可靠。
【VABench】1,299 条样本覆盖 15 维度评测，靠类目体系的结构化设计而非样本量取胜。
【Omni-Judge】仅 300 条提示词 / 600 条视频，即得出足以改变数据 pipeline 设计的结论（Omni-LLM 不可用于感知类判分）。
共同启示：评测侧的「小而精」范式已被验证；映射到训练侧，对 domain 覆盖度与标签可靠性的投入回报可能高于单纯扩大数据量。

### 训练数据domain分布与评测基准类目体系的对齐关系（如VABench七大类）

本字段是本次调研的核心产出 —— 五个基准的类目体系可直接反向作为训练数据 domain 分布的标准坐标系，建议按三个正交轴组合使用：

【轴一：内容 domain 轴 —— 采用 VABench 七大类目】动物 / 人声（语言性·非语言性）/ 音乐 / 环境音（自然·城市·室内）/ 同步物理声 / 复杂场景（复杂声景·主观感受·世界知识·符号联想·画外声源）/ 虚拟世界。这是最完整的内容侧分类学，可直接作为训练数据配比表的一级行标签。两个关键设计需一并移植：① 人声必须区分语言性与非语言性（前者需唇同步与 ASR 标注，后者不需要）；② 虚拟世界类目应豁免物理合理性质量门槛，与写实类目分开设定过滤标准。

【轴二：音频类型与同步难度轴 —— 采用 AV-SyncBench 三分法 + 十场景】Voice / Music / Sound 三大类下设动作、动物声、物体声、环境声、群体发声、单人说话、对话、演唱、单乐器、合奏十场景。该体系按同步难度而非主题切分，其单声源 vs 多声源的区分（单人说话 vs 对话/群体发声、单乐器 vs 合奏）直接对应训练数据中多声源混叠样本的配比需求，也决定了同步过滤器的选型（多声源场景下 Synchformer 类模型可靠性下降）。

【轴三：物理正确性覆盖轴 —— 采用 PhyAVBench 六维 41 测试点】声源力学 / 机械结构 / 流体与空气动力学 / 声传播环境 / 观察者物理 / 时间与因果（及复杂耦合与极端物理）。这是一份可勾选的 checklist，用于审计训练数据在物理声学上的覆盖盲区。评测实证已指明当前全行业的共同盲区是流体动力学与声传播环境两项，可作为优先补数据的方向。

【正交约束层 —— 采用 AVBench 的配额均衡机制】Hard Quota-Based Greedy Sampling 强制任一单属性占比 ≤50%，以及 Normal : Hard ≈ 3:1 的难度分层。这两条可直接移植为训练数据采样时的概念均衡策略与难例配比锚点。

【真实分布校准 —— 采用 Omni-Judge 的 VidProM 提示词分布】前四轴均为专家设计的理想分类，容易与真实用户需求脱节；Omni-Judge 直接采样 VidProM 真实用户提示词，提供了「用户实际想生成什么」的经验分布，可用于校准前述类目体系的权重，避免长尾类目过度投入。

【落地建议】构建一张三维配比矩阵（内容 domain × 音频类型 × 物理测试点覆盖），以 VidProM 真实分布定权重、以 AVBench 配额规则约束单属性占比、以 PhyAVBench checklist 审计盲区、以 AV-SyncBench 场景标签驱动同步过滤器分域选型。同时注意五个基准的共同空白 —— 多镜头长叙事、多语种口音、跨镜头音轨连续性目前均无基准可对标，这些 domain 的训练数据投入暂时无法用公开基准验证效果。

## 不确定字段

以下字段的调研信息部分不确定（⚠️ 标注来源）：

- release_date
- openness
- provenance_licensing
- duration_distribution
- resolution_aspect_distribution
- domain_distribution
- audio_category_distribution
- language_accent_distribution
- funnel_retention_rate
- shot_segmentation
- quality_filtering
- motion_filtering
- deduplication
- safety_filtering
- caption_model
- caption_structure
- dialogue_transcription_attributes
- geometric_structured_annotation
- synthetic_data_synthesis
- human_in_loop
- sync_metric_and_threshold
- audio_quality_filtering
- audio_type_handling
- multi_stage_curriculum
- stage_data_mixture
- post_training_data
- data_infra_throughput
- data_ablation
