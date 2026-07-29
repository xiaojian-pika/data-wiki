# 横向对比：基本信息

> 主题: 视频生成模型（含音视频同时生成）的数据处理：数据清洗流程、数据分布、打标方式

[← 返回首页](../index.md)

本页按字段横向对比所有条目。⚠️ 表示该条目此字段部分信息不确定。

**字段**: [名称](#名称) · [发布机构/公司](#发布机构公司) · [发布时间（技术报告/论文/开源时间）](#发布时间技术报告论文开源时间) · [类型（模型/数据集/工具链/评测基准）](#类型模型数据集工具链评测基准) · [开源程度（权重/代码/数据/pipeline各自是否开源）](#开源程度权重代码数据pipeline各自是否开源) · [是否支持音视频同时生成，以及实现方式（原生联合/级联/MoE融合）](#是否支持音视频同时生成以及实现方式原生联合级联moe融合) · [调研信息来源列表（论文/技术报告/官方文档/新闻的URL，标注每条来源的性质：官方一手/同团队旁证/第三方报道）](#调研信息来源列表论文技术报告官方文档新闻的url标注每条来源的性质官方一手同团队旁证第三方报道)

## 名称

`name` · 详细程度: minimal

### [Allegro](../models/Allegro.md)

Allegro（含后续 Allegro-TI2V 图生视频版本，以及 40×720P / 40×360P 变体；配套打标模型为同团队的多模态 MoE 模型 Aria）

### [Apollo](../models/Apollo.md)

Apollo（arXiv v2 起更名为 Klear；同一篇论文 arXiv:2601.04151，v1 标题《Apollo: Unified Multi-Task Audio-Video Joint Generation》，v2 标题《Klear: Unified Multi-Task Audio-Video Joint Generation》。注意与 Meta 的视频理解模型 Apollo、Meta 的 Apollo LMM 等同名工作无关，需以 arXiv 编号区分）

### [CineDance / CineDance-1M](../models/CineDance.md)

CineDance / CineDance-1M（论文标题：CineDance: Towards Next-Generation Multi-Shot Long-Form Cinematic Audio-Video Generation）。该工作包含三个产出：CineDance-1M 数据集（100 万条多镜头长篇音视频序列）、CineBench 评测基准（1000 条测试样例 + 六维度指标体系）、CineDance 生成模型（基于 LTX-2.3 改造的开源基线）。

### [CogVideoX](../models/CogVideoX.md)

CogVideoX（含 CogVideoX-2B / 5B、CogVideoX1.5-5B / 5B-I2V，以及配套的 CogVLM2-Caption 打标模型；产品侧对应「清影 Ying」，音效侧对应 CogSound）

### [Cosmos-Predict2.5](../models/Cosmos-Predict25.md)

Cosmos-Predict2.5（NVIDIA Cosmos 世界基础模型家族最新一代，论文《World Simulation with Video Foundation Models for Physical AI》arXiv:2511.00062；同篇一并发布 Cosmos-Transfer2.5 控制网家族）

### [Data-Juicer 2.0](../models/Data-Juicer.md)

Data-Juicer 2.0（含 Data-Juicer Sandbox 沙盒实验室）。定位为「面向基础模型、并借助基础模型」的一站式数据处理系统与云规模自适应算子库，论文全称《Data-Juicer 2.0: Cloud-Scale Adaptive Data Processing for and with Foundation Models》。配套的 Sandbox 组件论文全称《Data-Juicer Sandbox: A Feedback-Driven Suite for Multimodal Data-Model Co-development》，是把「数据配方」与「模型训练/评测」闭环起来的数据-模型协同开发中间件。在本次视频生成数据处理调研中，Data-Juicer 的角色不是生成模型，而是被多个团队复用的底层数据清洗/标注算子基础设施。

### [Foley-Omni](../models/Foley-Omni.md)

Foley-Omni（配套评测基准 V2ST-Bench）。论文全称《Foley-Omni: A Unified Multimodal Generation Model from Task-Level Audio Synthesis to Complete Video Soundtrack Generation》，即从任务级音频合成迈向完整视频配乐（Video-to-Soundtrack, V2ST）生成的统一多模态生成模型。核心定位是在同一个隐空间生成过程中联合建模语音（speech）、音效（sound effects/foley）与音乐（music）三类音轨。

### [Goku](../models/Goku.md)

Goku（《Goku: Flow Based Video Generative Foundation Models》，含 Goku-T2I / Goku-T2V / Goku-I2V 系列；arXiv:2502.04896，CVPR 2025 Highlight）

### [Hailuo / MiniMax Video](../models/Hailuo.md)

Hailuo / MiniMax Video（海螺AI视频）。这是一个产品线而非单一模型，历代模型ID包括：video-01（2024年9月，初代，别名 Hailuo）、video-01-live（Live2D/动漫特化）、video-01-director（镜头运动控制）、S2V-01（Subject-Reference 主体参考）、MiniMax-Hailuo-02（2025年6月）、MiniMax-Hailuo-2.3 / 2.3-Fast（2025年10月，截至2026年7月仍为官方文档中的最新在线版本）

### [HunyuanVideo-Foley](../models/HunyuanVideo-Foley.md)

HunyuanVideo-Foley（混元视频音效模型，全称《HunyuanVideo-Foley: Multimodal Diffusion with Representation Alignment for High-Fidelity Foley Audio Generation》）

### [HunyuanVideo](../models/HunyuanVideo.md)

HunyuanVideo（混元视频，2024原版13B）+ HunyuanVideo 1.5（混元视频1.5，8.3B）

### [InstructAV2AV](../models/InstructAV2AV.md)

InstructAV2AV（配套数据集 InsAVE-80K）。论文全称《InstructAV2AV: Instruction-Guided Audio-Video Joint Editing》，即指令引导的音视频联合编辑。核心定位是首个端到端的「仅凭文本指令同时编辑视频画面与其配套音轨」的框架——在改变指定视觉目标及其发声内容的同时，严格保留非目标区域的背景画面与环境声。其在本次调研谱系中的代表性维度是「合成数据」：论文构建了一条可扩展的数据合成pipeline（data engine），用受控编辑的方式人工制造 source→target 配对样本，解决音视频编辑领域「成对监督数据本不存在」的根本困境。

### [2026 其他音视频联合生成](../models/JAVG_2026_misc.md)

2026 其他音视频联合生成（JAVG）工作集锦 —— 七项工作合并调研：(1) Baton《Baton: Explicit Semantic Blueprints for Joint Video-Audio Generation》arXiv:2605.25195；(2) OmniCustom《OmniCustom: Sync Audio-Video Customization Via Joint Audio-Video Generation Model》arXiv:2602.12304（含其自建数据集 OmniCustom-1M）；(3) StreamChar《StreamChar: Long-Horizon Streaming Character Audio-Video Generation with Decoupled Orchestration》arXiv:2605.25659；(4) ALIVE《ALIVE: Animate Your World with Lifelike Audio-Video Generation》arXiv:2602.08682（含 Alive-Bench 1.0）；(5) CCL《Improving Joint Audio-Video Generation with Cross-Modal Context Learning》arXiv:2603.18600；(6) NAVA《Native Audio-Visual Alignment for Generation》arXiv:2605.30073；(7) ITS-JAVG《Inference-Time Scaling for Joint Audio-Video Generation》arXiv:2606.03183。本条目只提取数据处理相关内容，七项工作在数据披露深度上差异极大：ALIVE 与 NAVA 属工业级重披露，OmniCustom 属数据集构建型中等披露，Baton/CCL/StreamChar 仅披露数据来源与规模，ITS-JAVG 为完全 training-free 无训练数据。

### [音视频联合生成基线合集](../models/JavisDiT_baselines.md)

音视频联合生成基线合集（4 项工作合并调研）：
(1) JavisDiT / JavisDiT++ —— 《JavisDiT: Joint Audio-Video Diffusion Transformer with Hierarchical Spatio-Temporal Prior Synchronization》(arXiv:2503.23377) 与续作《JavisDiT++: Unified Modeling and Optimization for Joint Audio-Video Generation》(arXiv:2602.19163, ICLR 2026)，并附带评测基准 JavisBench / JavisBench-mini 与同步指标 JavisScore；
(2) MM-Diffusion —— 《MM-Diffusion: Learning Multi-Modal Diffusion Models for Joint Audio and Video Generation》(arXiv:2212.09478, CVPR 2023)，联合生成开山之作，附带自建 Landscape 数据集；
(3) AV-DiT —— 《AV-DiT: Efficient Audio-Visual Diffusion Transformer for Joint Audio and Video Generation》(arXiv:2406.07686)；
(4) Harmony —— 《Harmony: Harmonizing Audio and Video Generation through Cross-Task Synergy》(arXiv:2511.21579)，附带评测基准 Harmony-Bench；
(5) UniAVGen —— 《UniAVGen: Unified Audio and Video Generation with Asymmetric Cross-Modal Interactions》(arXiv:2511.03334)。
其中 (2)(3) 为早期小规模学术基线，(1) 为中期开源学术基线，(4)(5) 为 2025 年底的近期强基线。

### [可灵 3.0 / 可灵视频 3.0 Omni](../models/Kling_30_Omni.md)

可灵 3.0 / 可灵视频 3.0 Omni（Kling 3.0 / Kling Video 3.0 Omni，内部亦称 Kling O3）

### [LTX-2](../models/LTX-2.md)

LTX-2（含后续 LTX-2.3；技术报告《LTX-2: Efficient Joint Audio-Visual Foundation Model》）

### [LongCat-Video](../models/LongCat-Video.md)

LongCat-Video（美团 LongCat 团队视频生成基础模型，技术报告 arXiv:2510.22200；同系列衍生 LongCat-Video-Avatar 与 LongCat-Video-Avatar 1.5，技术报告 arXiv:2605.26486）

### [MOVA](../models/MOVA.md)

MOVA（MOSS Video and Audio）

### [合并条目：① Mochi 1](../models/Mochi_MAGI_Motif.md)

合并条目：① Mochi 1（含 mochi-1-preview 与 AsymmVAE）② MAGI-1（含 24B / 4.5B 变体、Distill 蒸馏版，以及 2026 年的 MAGI-1.1 24B）③ Motif-Video 2B（含 T2V 与 I2V 扩展）。三者均为开源纯视觉视频生成基础模型，放在一起对照的核心价值是：数据披露粒度从「几乎为零」（Mochi 1）→「方法级详尽但无数值」（MAGI-1）→「工具级+参数级详尽且明确以小数据取胜」（Motif-Video 2B）呈现出 2024→2026 的清晰演进曲线。

### [Movie Gen](../models/Movie_Gen.md)

Movie Gen（含 Movie Gen Audio）

### [NVIDIA NeMo Curator](../models/NeMo_Curator.md)

NVIDIA NeMo Curator（26.02 / 26.04 版）+ Cosmos-Xenna（视频侧底层分布式执行引擎），并附带同源的产品化实现 Cosmos-Curate（Cosmos 世界基础模型的训练数据生成系统）

### [OmniHuman 数据集 + OHBench](../models/OmniHuman.md)

OmniHuman 数据集 + OHBench（OmniHuman Benchmark）——人物中心音视频生成的大规模数据集与三级评测基准

### [Open-Sora 系列](../models/Open-Sora.md)

Open-Sora 系列（Open-Sora 1.0/1.1/1.2/1.3/2.0，HPC-AI Tech）与 Open-Sora Plan 系列（v1.0–v1.5，北京大学 PKU-YuanGroup）。二者是两个独立项目，常被并称为「开源 Sora 复现双雄」：Open-Sora 由潞晨科技/HPC-AI Tech 主导，主打极致低成本训练（2.0 版 20 万美元）；Open-Sora Plan 由北大袁粒团队主导，主打社区协作复现与多维数据清洗漏斗。本条目合并调研，凡字段内容不同处均分项说明。

### [Ovi](../models/Ovi.md)

Ovi（论文《Ovi: Twin Backbone Cross-Modal Fusion for Audio-Video Generation》，arXiv:2510.01284；后续迭代版本 Ovi 1.1；音频塔单独命名为 Ovi-Aud）

### [Script-a-Video](../models/Script-a-Video.md)

Script-a-Video（核心产出为 MTSS 表示范式，全称 Multi-Stream Scene Script，多流场景脚本）

### [Seedance 2.0 与 Seedance 1.5 pro](../models/Seedance_20_Seedance_15_pro.md)

Seedance 2.0 与 Seedance 1.5 pro（含 Seedance 1.0 作为数据管线纵向基线）

### [SkyReels 系列](../models/SkyReels.md)

SkyReels 系列（本条目覆盖 SkyReels-V2《SkyReels-V2: Infinite-length Film Generative Model》arXiv:2504.13074，与 SkyReels-V4《SkyReels-V4: Multi-modal Video-Audio Generation, Inpainting and Editing model》arXiv:2602.21818；中间版本 SkyReels-V3 于2026年1月29日开源，含参考图生视频R2V、视频延长V2V、音频驱动数字人Talking Avatar三大能力，作为V2到V4的过渡）

### [Sora 2](../models/Sora_2.md)

Sora 2（含 Sora 2 Pro）

### [SpeakerVid-5M](../models/SpeakerVid-5M.md)

SpeakerVid-5M（论文全称：SpeakerVid-5M: A Large-Scale High-Quality Dataset for Audio-Visual Dyadic Interactive Human Generation）

### [Step-Video-T2V](../models/Step-Video-T2V.md)

Step-Video-T2V（阶跃视频，30B 文生视频基础模型）及其衍生 Step-Video-T2V-Turbo、Step-Video-TI2V

### [UniTalking](../models/UniTalking.md)

UniTalking（统一音视频说话人肖像生成框架）

### [UniVerse-1](../models/UniVerse-1.md)

UniVerse-1（含配套评测基准 Verse-Bench）

### [Unison](../models/Unison.md)

Unison（论文全称《Unison: Harmonizing Motion, Speech, and Sound for Human-Centric Audio-Video Generation》，直译「Unison：为人物中心音视频生成协调运动、语音与声音」）。名称取自音乐术语「齐奏/同度」，呼应论文核心主张——让运动、语音、音效三者「in Unison」（步调一致）地演进。注意与同期同名工作区分：arXiv:2605.31530《UNISON: A Unified Sound Generation and Editing Framework via Deep LLM Fusion》是纯音频生成/编辑工作，与本条目无关。

### [Veo 3 / Veo 3.1](../models/Veo_3_Veo_31.md)

Veo 3 / Veo 3.1（含 Veo 3.1 Lite）

### [Vidu S1](../models/Vidu_S1.md)

Vidu S1（技术报告《Vidu S1: A Real-Time Interactive Video Generation Model》，arXiv:2607.03118v2；产品名 Vidu Stream，https://vidu.com/vidu-stream）

### [Wan 2.5 / 2.6 / 2.7](../models/Wan.md)

Wan 2.5 / 2.6 / 2.7（通义万相，Wan 系列闭源商用版本）——含其开源前代 Wan 2.1（arXiv:2503.20314）、Wan 2.2（Apache 2.0）及同团队旁证模型 Wan2.2-S2V（arXiv:2508.18621）、Wan2.2-Animate、Wan-Dancer（arXiv:2607.09581）

### [音视频生成评测基准合集](../models/av_benchmarks.md)

音视频生成评测基准合集（VABench / AVBench / AV-SyncBench / PhyAVBench / Omni-Judge）

### [视频 Caption 模型生态](../models/caption_models.md)

视频 Caption 模型生态（Video Captioner Ecosystem）——合并调研条目，覆盖 ShareGPT4Video / ShareCaptioner-Video、Tarsier & Tarsier2 系列、CogVLM2-Caption、SkyCaptioner-V1、AVoCaDO、AVSCap、video-SALMONN 2、Qwen3-Omni-Captioner / Qwen3.5-Omni、AuroraCap、Panda-70M 多教师 captioner、LLaVA-Video / PLLaVA、Aria、Tag2Text 等，并归纳它们在视频生成数据 pipeline 中的实际应用方式。本条目不是单一模型/数据集，而是「打标器」这一 pipeline 组件的横向生态图谱。

### [几何/结构化标注数据集合集](../models/geometric_datasets.md)

几何/结构化标注数据集合集（SceneScribe-1M / SpatialVID / WildWorld / Action100M）

### [视频生成后训练数据策略](../models/post_training_data.md)

视频生成后训练数据策略（跨模型横向专题）——以《A Systematic Post-Train Framework for Video Generation》(arXiv:2604.25427) 为锚，横向汇总各模型的 SFT 精选集规模/筛选标准与偏好对标注方式

### [主流视频预训练数据集合并调研：Panda-70M、InternVid、Koala…](../models/pretraining_datasets.md)

主流视频预训练数据集合并调研：Panda-70M、InternVid、Koala-36M、MiraData、OpenVid-1M、UltraVideo、LVD-2M（共7个公开视频-文本数据集，覆盖2023–2025年，重点比较清洗方法与caption策略）

## 发布机构/公司

`organization` · 详细程度: minimal

### [Allegro](../models/Allegro.md)

Rhymes AI（瑞莱斯智能 / Rhymes AI，新加坡—香港背景的 AI 初创公司，与 Aria 多模态模型为同一团队）。论文作者：Yuan Zhou、Qiuyue Wang、Yuxuan Cai、Huan Yang（Huan Yang 为通讯/负责人，曾任微软亚洲研究院研究员）

### [Apollo](../models/Apollo.md) ⚠️

快手科技（Kuaishou Technology）可灵团队（Kling Team）。作者：Jun Wang、Chunyu Qiang、Yuxin Guo、Yiran Wang、Xijuan Zeng、Feng Deng 等。该工作是快手可灵（Kling）系列音视频同步生成能力（Kling 2.6「音画同出」、Kling 3.0）背后的研究性技术报告，属工业界闭源模型的对外论文披露。[不确定]（论文本身未显式声明与 Kling 产品线的对应关系，此关联为 HuggingFace 论文页署名「Kling Team, Kuaishou Technology」与产品时间线的旁证推断）

### [CineDance / CineDance-1M](../models/CineDance.md) ⚠️

学术界多机构联合，无企业实验室署名。参与单位包括：上海交通大学（Shanghai Jiao Tong University）、电子科技大学（University of Electronic Science and Technology of China）、浙江大学（Zhejiang University）、东京大学（The University of Tokyo）、南洋理工大学（Nanyang Technological University）。作者列表：Yuheng Chen、Teng Hu、Yuji Wang、Qingdong He、Zhucun Xue、Qianyu Zhou、Jason Li、Lizhuang Ma、Jiangning Zhang、Dacheng Tao。项目主页维护者为一作 Yuheng Chen（github.com/AliothChen）。具体每位作者对应哪个机构未在页面明确列出。[不确定]

### [CogVideoX](../models/CogVideoX.md)

智谱 AI（Zhipu AI）与清华大学 THUDM 联合团队，论文通讯作者唐杰；核心贡献者 Zhuoyi Yang、Jiayan Teng、Wendi Zheng、Ming Ding、Shiyu Huang、Xiaotao Gu 等

### [Cosmos-Predict2.5](../models/Cosmos-Predict25.md)

NVIDIA（论文署名即 NVIDIA，含 88 位贡献者的大型协作团队，作者含 Arslan Ali、Junjie Bai、Sanja Fidler 等；隶属 NVIDIA Cosmos Lab / Physical AI 方向）

### [Data-Juicer 2.0](../models/Data-Juicer.md)

阿里巴巴集团。发起团队为阿里巴巴通义实验室（Alibaba Tongyi Lab）数据方向团队，与阿里云 PAI（Platform for AI）联合共建，并与 Anyscale（Ray 团队）、NVIDIA、中山大学等外部单位协作。核心作者：Daoyuan Chen（陈道源，一作/通讯，主页 yxdyc.github.io）、Yilun Huang、Xuchen Pan、Nana Jiang、Haibin Wang、Yilei Zhang、Ce Ge、Yushuo Chen、Wenhao Zhang、Zhijian Ma、Jun Huang、Wei Lin、Yaliang Li（李雅亮）、Bolin Ding（丁博麟）、Jingren Zhou（周靖人）。Sandbox 论文作者为 Daoyuan Chen、Haibin Wang、Yilun Huang、Ce Ge、Yaliang Li、Bolin Ding、Jingren Zhou。GitHub 组织已从 modelscope/data-juicer 迁移至 datajuicer/data-juicer。

### [Foley-Omni](../models/Foley-Omni.md)

以南京大学智能科学与技术学院（School of Intelligence Science and Technology, Nanjing University）为主导，联合 Video Rebirth（工业界合作方）、上海交通大学、北京交通大学、上海人工智能实验室（Shanghai AI Laboratory）。作者列表：Ye Tao（陶烨，一作，联系邮箱 taoye0402@gmail.com，项目主页 ty0402.github.io 维护者）、Lupeng Liu、Xuenan Xu（徐雪男，音频描述/音频caption方向研究者）、Jiasun Feng、Jiarui Wang、Ying Qin、Shuiyang Mao、Wei Liu、Shuai Wang（王帅，通讯作者方向，南大语音组）。GitHub 组织名为 NJU-Speech，HuggingFace 账号为 CocoBro。

### [Goku](../models/Goku.md)

字节跳动（ByteDance）与香港大学（HKU）联合。论文署名 22 位作者，一作 Shoufa Chen（HKU），通讯/资深作者含 Ping Luo（HKU）、Yi Jiang、Zehuan Yuan、Bingyue Peng、Xiaobing Liu（字节跳动）。团队与字节 Seed / 视觉生成线高度重叠，项目代号来源于《龙珠》悟空（Saiyan-World 组织名）。

### [Hailuo / MiniMax Video](../models/Hailuo.md)

MiniMax（稀宇科技 / 上海稀宇极智科技有限公司），中国上海。同一公司的其他产品线包括 MiniMax-01/M1/M2/M2.5/M2.7/M3 系列大语言模型、MiniMax Speech 语音、MiniMax Music 音乐、MiniMax Code。视频产品对外品牌为「海螺AI」（Hailuo AI，hailuoai.video / hailuoai.com）

### [HunyuanVideo-Foley](../models/HunyuanVideo-Foley.md)

腾讯混元（Tencent Hunyuan）主导，联合浙江大学（Zhejiang University）与南京航空航天大学（Nanjing University of Aeronautics and Astronautics）。作者列表：Sizhe Shan（单思哲，一作）、Qiulin Li、Yutao Cui、Miles Yang、Yuehai Wang、Qun Yang、Jin Zhou、Zhao Zhong。项目主页由 szczesnys 维护。定位为腾讯混元开源矩阵中继 HunyuanVideo（视频生成）之后补齐「声音」环节的配套模型，与 HunyuanVideo 系列共享品牌但不共享模型架构。

### [HunyuanVideo](../models/HunyuanVideo.md)

腾讯混元基础模型团队（Tencent Hunyuan Foundation Model Team）

### [InstructAV2AV](../models/InstructAV2AV.md)

北京智源人工智能研究院（Beijing Academy of Artificial Intelligence, BAAI）+ 北京大学（Peking University）联合。作者列表：Haojie Zheng（郑浩杰，一作，BAAI 与 PKU 双属，个人主页 hjzheng.net）、Yixin Yang（PKU）、Siqi Yang（PKU）、Shuchen Weng（翁书宸，BAAI 与 PKU 双属）、Boxin Shi（施柏鑫，PKU 计算机学院，通讯作者，视觉与计算摄影方向）。代码仓库托管于个人 GitHub 账号 suimuc/InstructAV2AV，HuggingFace 账号为 suimu。

### [2026 其他音视频联合生成](../models/JAVG_2026_misc.md) ⚠️

七项工作分属不同机构：
(1) Baton：复旦大学（Shuyuan Tu、Zuxuan Wu、Yu-Gang Jiang，视觉与学习实验室）× 腾讯混元（Tencent Hunyuan，作者含 Weijie Kong、Jiangfeng Xiong、Zhao Zhong 等混元视频生成团队成员）联合，另有 Liefeng Bo（阿里通义/达摩院背景）与 Qi Tian、Xintong Han 参与[机构归属为按作者名单推断，不确定]。
(2) OmniCustom：香港大学（The University of Hong Kong，Guosheng Yin、Dong Xu）主导，联合 Shanda AI Research Tokyo（盛大 AI 研究院东京）、XIntelligence Technology Co., Limited，以及上海人工智能实验室（Kaipeng Zhang）；作者 Maomao Li、Zhifeng Li 有腾讯 AI Lab 背景。
(3) StreamChar：阿里巴巴通义实验室（Tongyi Lab, Alibaba Group），作者 Linrui Tian、Qi Wang、Bang Zhang，即 EMO / Wan-S2V 系列数字人团队，项目页托管于 HumanAIGC 组织。
(4) ALIVE：字节跳动 ALIVE 团队（Bytedance ALIVE Team），16 位作者，含 Xiang Yin（Seed 语音方向）、Bingyue Peng、Zehuan Yuan 等；仓库位于 FoundationVision 组织下。
(5) CCL：商汤科技（SenseTime）系，作者 Bingqi Ma、Guanglu Song、Yu Liu、Dailan He 等[机构为按作者推断，论文页未显式列出，不确定]。
(6) NAVA：百度 ERNIE Research（文心研究团队），作者 Longbin Ji、Guan Wang、Xuan Wei、Shuohuan Wang、Yu Sun 等，权重同时托管于 HuggingFace 的 baidu 与 ernie-research 组织。
(7) ITS-JAVG：KAIST（韩国科学技术院），作者 Jaemin Jung、Kyeongha Rho、Inkyu Shin、Joon Son Chung（Joon Son Chung 即 SyncNet 作者之一，音视频同步领域权威）。

### [音视频联合生成基线合集](../models/JavisDiT_baselines.md)

(1) JavisDiT / JavisDiT++：新加坡国立大学（NUS，Hao Fei、Shengqiong Wu、Tat-Seng Chua、Wei Li 等）主导，联合厦门大学（Jiayi Ji）、复旦大学（Fan Zhou）、罗切斯特大学（Jiebo Luo）、南洋理工大学（Ziwei Liu）等，第一作者 Kai Liu；社区组织名为 JavisVerse。
(2) MM-Diffusion：中国人民大学（Ludan Ruan、Qin Jin）与微软亚洲研究院 MSRA（Huan Yang、Bei Liu、Jianlong Fu、Nicholas Jing Yuan、Baining Guo）联合，另有北京大学参与；GitHub 组织为 researchmm（微软研究院多媒体组）。
(3) AV-DiT：多伦多大学（University of Toronto，Kai Wang、Dimitrios Hatzinakos）、德克萨斯大学达拉斯分校（UT Dallas，Shijian Deng、Yapeng Tian）与 Adobe Research（Jing Shi）联合。
(4) Harmony：上海交通大学（Teng Hu、Ran Yi）与腾讯混元 Tencent Hunyuan（Zhentao Yu、Guozhen Zhang、Zhengguang Zhou、Youliang Zhang、Yuan Zhou、Qinglin Lu）联合。
(5) UniAVGen：南京大学（新型软件技术国家重点实验室，Guozhen Zhang、Limin Wang）与腾讯混元 Tencent Hunyuan（Zixiang Zhou、Yi Chen、Yuan Zhou、Qinglin Lu）主导，联合上海交通大学（Teng Hu）、中国人民大学（Ziqiao Peng）、清华大学（Youliang Zhang）、上海人工智能实验室。
注：Harmony 与 UniAVGen 作者高度重叠（Guozhen Zhang、Teng Hu、Youliang Zhang、Yuan Zhou、Qinglin Lu 均在两篇中出现），可视为腾讯混元同一研究线上的姊妹工作。

### [可灵 3.0 / 可灵视频 3.0 Omni](../models/Kling_30_Omni.md)

快手科技（Kuaishou Technology）——可灵大模型团队 / 快手视觉生成与互动中心（KwaiVGI，GitHub 组织亦作 KlingTeam / KlingAIResearch）

### [LTX-2](../models/LTX-2.md)

Lightricks（以色列，LTX Studio / LTXV 系列模型团队）

### [LongCat-Video](../models/LongCat-Video.md)

美团（Meituan）LongCat 团队。技术报告署名 Meituan LongCat Team，作者含 Xunliang Cai、Qilong Huang、Zhuoliang Kang、Hongyu Li、Shijun Liang、Liya Ma、Siyu Ren、Xiaoming Wei、Rixu Xie、Tong Zhang 等。

### [MOVA](../models/MOVA.md)

SII-OpenMOSS 团队。论文署名机构包括：上海创智学院（Shanghai Innovation Institute）、MOSI Intelligence（无问芯穹/MOSI 智能）、复旦大学、上海交通大学、华东师范大学、同济大学、东南大学、厦门大学、电子科技大学。项目负责人为 Qinyuan Cheng（程琴媛）与 Tianyi Liang；通讯作者为陈谐（上海交大，chenxie95@sjtu.edu.cn）与邱锡鹏（复旦大学，xpqiu@fudan.edu.cn）。属于复旦 OpenMOSS（MOSS 系列）开源体系的音视频生成分支。

### [合并条目：① Mochi 1](../models/Mochi_MAGI_Motif.md)

① Mochi 1：Genmo（Genmo AI，美国旧金山创业公司，CEO/联合创始人 Paras Jain，联合创始人 Ajay Jain，均为 UC Berkeley 博士背景；2024 年完成约 2840 万美元 A 轮融资，NEA 领投）。
② MAGI-1：Sand AI（三体智能 / sand.ai，中国北京，创始人曹越 Yue Cao，清华大学博士、原微软亚洲研究院研究员，Swin Transformer 作者之一；技术报告署名作者 39 人）。
③ Motif-Video 2B：Motif Technologies（韩国 AI 公司，与 AI 芯片/编译器公司 Moreh 同源，此前发布过 Motif-2.6B / Motif-2-12.7B 语言模型系列）。

### [Movie Gen](../models/Movie_Gen.md)

Meta（Meta AI / GenAI 团队，超过百人贡献者名单）

### [NVIDIA NeMo Curator](../models/NeMo_Curator.md)

NVIDIA。三个仓库分属不同 GitHub 组织：NeMo Curator 在 NVIDIA-NeMo/Curator，Cosmos-Xenna 与 Cosmos-Curate 在 nvidia-cosmos 组织（早期为 NVIDIA/cosmos-curator）。NeMo Curator 由 NeMo Framework 团队维护，Cosmos-Xenna 由 Cosmos（Physical AI / 世界基础模型）团队开发后独立开源。

### [OmniHuman 数据集 + OHBench](../models/OmniHuman.md)

北京大学（Peking University, Beijing, China）+ 腾讯微信实验室（WeChat Lab, Tencent, China）+ 中国科学院（Chinese Academy of Sciences, Beijing, China）三方合作。作者列表：Lei Zhu、Xing Cai、Yingjie Chen、Yiheng Li、Binxin Yang、Hao Liu、Jie Chen、Chen Li、Jing Liu（Jing LYu）。注意：本工作与字节跳动 2025 年的 OmniHuman-1 说话人生成模型（arXiv:2502.01061）同名但无关，后者是模型、本工作是数据集+评测基准，检索与引用时需区分。

### [Open-Sora 系列](../models/Open-Sora.md)

（1）Open-Sora：HPC-AI Tech / 潞晨科技（Colossal-AI 团队），GitHub 组织 hpcaitech。（2）Open-Sora Plan：北京大学袁粒课题组（PKU-YuanGroup），联合鹏城实验室、兔展智能（Rabbitpre AI）等，HuggingFace 组织为 LanguageBind。二者无隶属关系，仅名称相近。

### [Ovi](../models/Ovi.md)

Character AI（主体，作者 Chetwin Low、Weimin Wang，Weimin Wang 为 Project Lead）与 Yale University（耶鲁大学，作者 Calder Katyal）联合发布。致谢中提到 Yi Cui、Manav Shah、Diego De La Torre 参与数据准备工作。

### [Script-a-Video](../models/Script-a-Video.md) ⚠️

腾讯混元团队（Tencent Hunyuan Team）。论文署名为团队整体署名（作者栏仅写 Tencent Hunyuan Team），正文末尾设有 Project Contributors 章节但未在 arXiv HTML 版本中展开具体人名，故无法确认一作与通讯作者。[不确定]

### [Seedance 2.0 与 Seedance 1.5 pro](../models/Seedance_20_Seedance_15_pro.md)

字节跳动 Seed 团队（ByteDance Seed / Team Seedance）

### [SkyReels 系列](../models/SkyReels.md)

昆仑万维（Kunlun Tech / Skywork AI，天工AI）旗下 SkyReels 团队。V4 论文作者50余人，项目负责人 Guibin Chen（guibin.chen@kunlun-inc.com），项目发起人周亚辉（Yahui Zhou）；团队按基础设施、数据处理、视频模型训练、音频模型训练、多模态训练、评测分组。

### [Sora 2](../models/Sora_2.md)

OpenAI

### [SpeakerVid-5M](../models/SpeakerVid-5M.md)

清华大学（深圳国际研究生院，Tsinghua University）联合 StepFun（阶跃星辰）与香港科技大学（HKUST，含广州校区）。作者列表：Youliang Zhang、Zhaoyang Li、Duomin Wang、Jiahe Zhang、Deyu Zhou、Zixin Yin、Xili Dai、Gang Yu、Xiu Li。项目负责人（Project Lead）为 Duomin Wang（王多民，StepFun），通讯作者为李秀（Xiu Li，清华大学）。Gang Yu（于刚）为 StepFun 视觉团队负责人。

### [Step-Video-T2V](../models/Step-Video-T2V.md)

阶跃星辰（StepFun，Step-Video Team / 上海阶跃星辰智能科技有限公司）

### [UniTalking](../models/UniTalking.md)

华为中央媒体技术研究院（Central Media Technology Institute, Huawei）为主导机构，联合北京航空航天大学计算机学院。作者列表：Hebeizi Li（李赫贝子，一作，华为+北航，实习期间完成本工作）、Zihao Liang（梁子豪，共同一作）、Benyuan Sun（孙本源，Project Leader）、Zihao Yin、Xiao Sha、Chenliang Wang、Yi Yang（杨毅，通讯作者）。定位为「开源可复现的说话人肖像音视频统一生成框架」，明确对标 Veo3 与 Sora2 的闭源不可及性。

### [UniVerse-1](../models/UniVerse-1.md)

阶跃星辰（StepFun）为第一/主导机构，联合香港科技大学（广州）、香港科技大学、清华大学。作者列表：Duomin Wang（王多民，一作，项目主页与代码库维护者 Dorniwang）、Wei Zuo、Aojie Li、Ling-Hao Chen（陈凌灏，清华）、Xinyao Liao、Deyu Zhou、Zixin Yin、Xili Dai（戴习理，港科广）、Daxin Jiang（姜大昕，StepFun 创始人/CEO）、Gang Yu（于刚，StepFun 视觉负责人）。定位为 StepFun 在音视频联合生成方向的首个开源探索。

### [Unison](../models/Unison.md)

五家机构联合署名，学术机构主导、产业界与国资电信 AI 参与：
1. 武汉大学 测绘遥感信息工程国家重点实验室（State Key Laboratory of Information Engineering in Surveying, Mapping and Remote Sensing, LIESMARS）——第一署名单位，通讯作者涂志刚（Zhigang Tu）所在单位；
2. 字节跳动（ByteDance），中国；
3. 西安交通大学 人机混合增强智能全国重点实验室、人工智能与机器人研究所（IAIR）；
4. 中国电信人工智能科技（北京）有限公司；
5. 中国电信 人工智能研究院（TeleAI）。
【作者列表】Shihao Cheng（程世豪）、Jiaxu Zhang（张家旭）为共同第一作者（标注 † Equal contribution）；Quanyue Song、Shansong Liu（刘杉松）、Zhizhi Guo、Xiao-Lei Zhang（张晓雷，西工大/西交大语音方向）、Chi Zhang（张驰）、Xuelong Li（李学龙，TeleAI CTO/首席科学家）、Zhigang Tu（涂志刚，通讯作者 🖂）。论文另标注有「‡ 项目负责人」角色。
【经费来源】国家自然科学基金区域创新发展联合基金 U25A20537；国家重点研发计划 2024YFC3015600。这是一个明确的国内学术界主导项目，非工业界大模型团队产出，其数据规模与披露风格也与 Sora 2/Veo 3/Seedance 一类工业报告截然不同。

### [Veo 3 / Veo 3.1](../models/Veo_3_Veo_31.md)

Google DeepMind（Google）

### [Vidu S1](../models/Vidu_S1.md)

生数科技（Shengshu Technology）联合清华大学（Tsinghua University）。作者含 Jintao Zhang、Kai Jiang、Jintao Chen 等27人，顾问为 Zhijie Deng、Fan Bao（鲍凡）、Jianfei Chen、Jun Zhu（朱军）

### [Wan 2.5 / 2.6 / 2.7](../models/Wan.md)

阿里巴巴集团 · 阿里云通义实验室（Tongyi Lab，Wan / 万相团队；人物类模型 S2V、Animate、Dancer 由 HumanAIGC 组主导）。服务载体为阿里云百炼（Model Studio / DashScope）与万相官网 wan.video、tongyi.aliyun.com/wan。

### [音视频生成评测基准合集](../models/av_benchmarks.md)

多家机构，五个基准分属不同团队：
【VABench】北京大学（Wentao Zhang 组，含 Bohan Zeng、Hao Liang、Junbo Niu 等）+ 蚂蚁集团 Ant Group（Quanqing Xu）+ 中国科学院自动化研究所 + 华中科技大学。第一作者 Daili Hua、Xizhi Wang。
【AVBench】清华大学（Wenming Yang 组，第一作者 Jialiang Yang，含 Bin Xia、Ruihang Chu 等）+ 香港中文大学（Dingdong Wang 等）。
【AV-SyncBench】阿里巴巴集团（Jun Song、Bo Zheng 等，通讯 jsong.sj@alibaba-inc.com）+ 清华大学（第一作者 Tianhong Zhou，zth24@mails.tsinghua.edu.cn）+ 复旦大学。
【PhyAVBench】香港科技大学（广州）HKUST(GZ)（通讯作者 Li Liu，第一作者 Tianxin Xie）+ 腾讯 Tencent + 上海交通大学 + 慕尼黑工业大学 TUM；作者 29+ 人，4 位 core contributor。
【Omni-Judge】罗切斯特大学 University of Rochester（第一作者 Susan Liang，通讯 Chenliang Xu）+ 密歇根大学安娜堡分校（Filippos Bellos、Jason J. Corso）。

### [视频 Caption 模型生态](../models/caption_models.md)

多家，按阵营归类：
【学术/开源社区】ShareGPT4Video & ShareCaptioner-Video（中科大 + 上海 AI Lab + 香港中文大学，Lin Chen、Xilin Wei、Jinsong Li 等 15 位作者，NeurIPS 2024 Datasets & Benchmarks Track）；AuroraCap（UC Santa Barbara / 多校合作，ICLR 2025）；Panda-70M（Snap Research + UC Merced，CVPR 2024）。
【中国大厂】字节跳动 Tarsier / Tarsier2 / Tarsier2-Recap（bytedance/tarsier），并与清华电子系合作 video-SALMONN 2 / video-SALMONN-o1；智谱 AI（zai-org / 原 THUDM）CogVLM2-Caption；阿里通义 Qwen 系列（Qwen2-VL / Qwen2.5-VL / Qwen3-VL / Qwen2.5-Omni / Qwen3-Omni-Captioner / Qwen3.5-Omni，配套 Omni-Captioner + Omni-Detective + Omni-Cloze，ICLR 2026）；昆仑万维 SkyWork SkyCaptioner-V1；快手可灵团队 AVoCaDO（联合中科院自动化所、国科大、北大、南大）与 Koala-36M 的 LLaVA 微调 captioner；南京大学 NJU-LINK + 快手 Kling AVSCap；小米 MiMo-VL（被 MOVA 用作视觉标注器）。
【海外大厂/闭源】OpenAI（Sora 的 highly descriptive captioner、Whisper ASR）；Meta（Movie Gen 的 LLaMa3-Video 8B/70B）；Google（Gemini 系列作为通用打标器，Veo 3 使用「多个 Gemini 模型」）；Lightricks（LTX-Video / LTX-2 的内部自研音视频 captioner）；Rhymes AI（Aria，被 Allegro 用作细粒度打标器）。

### [几何/结构化标注数据集合集](../models/geometric_datasets.md)

多家机构：SceneScribe-1M——上海交通大学 + 蚂蚁集团 + 牛津大学视觉几何组(VGG) + 东方理工宁波数字孪生研究院（浙江省工业智能与数字孪生重点实验室）；SpatialVID——南京大学（NJU-3DV）+ 中国科学院自动化研究所；WildWorld——盛大AI（ShandaAI）+ 上海人工智能实验室 + 北京理工大学（作者含 Zhen Li、Kaipeng Zhang 等）；Action100M——Meta AI（FAIR，代码托管于 facebookresearch）+ 香港科技大学（Pascale Fung 团队）+ 阿姆斯特丹大学 + 索邦大学

### [视频生成后训练数据策略](../models/post_training_data.md)

多家。锚论文为香港大学（HKU，Zeyue Xue、Mengzhao Chen、Ping Luo）+ 京东探索研究院（JD Explore Academy，Siming Fu、Jie Huang、Shuai Lu、Haoran Li、Haoyang Huang、Nan Duan 等）+ 清华大学 + 北京大学 + 浙江大学的联合工作（Zeyue Xue 亦是 DanceGRPO 一作，Nan Duan 为通讯层作者）。横向覆盖对象包括：字节跳动 Seed（Seedance 1.0 / 1.5 pro）、腾讯混元（HunyuanVideo / HunyuanVideo 1.5）、快手可灵（Kling 3.0 Omni）、美团（LongCat-Video）、阶跃星辰（Step-Video-T2V）、昆仑万维（SkyReels-V2 / V4）、NVIDIA（Cosmos-Predict 2.5）、Meta（Movie Gen）、智谱（CogVideoX）、Rhymes AI（Allegro）、字节（Goku）、Moonshot/Motif（Motif-Video 2B）、Sand AI（MAGI-1）、Genmo（Mochi 1）、Lightricks（LTX-2）、OpenAI（Sora 2）、Google DeepMind（Veo 3/3.1）、生数（Vidu S1）、HPC-AI Tech（Open-Sora 2.0）、PKU-YuanGroup（Open-Sora Plan）、以及学术侧 JavisDiT++、NAVA、ALIVE 等。

### [主流视频预训练数据集合并调研：Panda-70M、InternVid、Koala…](../models/pretraining_datasets.md)

Panda-70M：Snap Inc.（Snap Research）+ 加州大学默塞德分校（UC Merced）。InternVid：上海人工智能实验室 / OpenGVLab（联合南京大学、香港大学、南洋理工、深圳先进院）。Koala-36M：快手科技（Kuaishou，仓库现属 KlingAIResearch，原 KwaiVGI）+ 深圳大学 + 清华大学。MiraData：腾讯 PCG ARC Lab + 香港中文大学。OpenVid-1M：南京大学 PCA Lab + 字节跳动 + 南开大学。UltraVideo：浙江大学 APRIL Lab + 上海交通大学 + 华中科技大学 + 南洋理工大学。LVD-2M：香港大学 + 字节跳动。

## 发布时间（技术报告/论文/开源时间）

`release_date` · 详细程度: minimal

### [Allegro](../models/Allegro.md)

2024年10月20日 arXiv 预印本 v1 提交（arXiv:2410.15458，《Allegro: Open the Black Box of Commercial-Level Video Generation Model》）；2024年10月22日在 GitHub / Hugging Face 开源 Allegro T2V 权重与推理代码（Apache 2.0）；2024年11月25日发布 Allegro-TI2V（文本+图像转视频）；2024年12月10日开源 Allegro T2V 训练代码；2024年12月26日发布 40×720P / 40×360P 变体；2025年1月2日开源 Allegro-TI2V 训练代码

### [Apollo](../models/Apollo.md)

2026年1月7日首次提交 arXiv（v1，标题为 Apollo）；2026年1月13日更新 v2（标题改为 Klear）。HuggingFace 论文页收录日期为 2026年1月8日。无独立的开源发布时间（未开源）。

### [CineDance / CineDance-1M](../models/CineDance.md)

2026年6月8日首次提交 arXiv（arXiv:2606.09639 v1），2026年6月11日更新 v2。GitHub 仓库与项目主页同期上线。数据集第一批分片（CineDance_01）于 2026年7月前后在 HuggingFace 以 gated 门控方式放出。代码（curation pipeline、推理、训练）与模型权重截至 2026年7月仍标注为「pending release」，尚未发布。

### [CogVideoX](../models/CogVideoX.md)

2024年8月6日开源 CogVideoX-2B 权重与代码；2024年8月12日 arXiv 预印本 2408.06072 v1（《CogVideoX: Text-to-Video Diffusion Models with An Expert Transformer》）；2024年8月27日开源 CogVideoX-5B；2024年9月开源 CogVideoX-5B-I2V；2024年11月8日发布「新清影」并开源 CogVideoX1.5-5B / 1.5-5B-I2V（同期发布音效模型 CogSound）；论文被 ICLR 2025 接收，v3 版 2025年3月26日更新

### [Cosmos-Predict2.5](../models/Cosmos-Predict25.md)

arXiv v1 提交于 2025年10月28日（arXiv:2511.00062v1），v2 修订于 2026年2月24日（论文页眉标注 2026-2-26，版权标注 ©2026 NVIDIA）。代码与权重同期在 GitHub nvidia-cosmos/cosmos-predict2.5 与 Hugging Face nvidia/Cosmos-Predict2.5-2B / -14B 开放。

### [Data-Juicer 2.0](../models/Data-Juicer.md) ⚠️

系列时间线：
- 2023年：Data-Juicer 1.0 首次开源，论文发表于 SIGMOD 2024（约50个纯文本预训练算子）。
- 2024年7月16日：Data-Juicer Sandbox 论文提交 arXiv（arXiv:2407.11784 v1），最新 v3 为2025年6月4日，录用于 ICML 2025 Spotlight。同期在 VBench 文生视频榜单登顶并由阿里云开发者社区/魔搭社区发文宣传。
- 2025年1月（arXiv 编号 2501.14755，提交历史显示 v1 为2024年12月23日、v2 为2025年6月4日、v3 为2025年10月29日）：Data-Juicer 2.0 论文发布，录用于 NeurIPS 2025 Spotlight。
- 持续迭代至调研时点：v1.4.4（2025-12-01，NeurIPS Spotlight 同期，新增6个视频/多模态算子、S3 I/O）、v1.4.5（2026-01-15，20+新算子、Ray vLLM 流水线、具身智能算子）、v1.4.6（2026-02-02，Q&A Copilot、视频字节流 I/O）、v1.5.0（2026-02-12，分区 Ray 执行器、具身智能视频处理、算子级环境管理）、v1.5.1（2026-03-17）、v1.5.2（2026-05-29）、v1.5.3（2026-06-26，VLA 具身算子、Ray repartition 流水线）、v1.5.4（2026-07-17，新增9个人物中心视频理解算子、batch-local stage fusion 加速）。
[不确定] Data-Juicer 2.0 论文 arXiv 首次提交日期在不同来源上呈现为2024年12月23日与2025年1月两种说法，以 arXiv 编号 2501 推断正式挂出应为2025年1月。

### [Foley-Omni](../models/Foley-Omni.md)

2026年6月2日提交至 arXiv（arXiv:2606.03672，cs.SD/cs.CV 方向，v1版本）。同期上线项目主页 https://ty0402.github.io/Foley-omni-Web/ 与 GitHub 仓库 NJU-Speech/Foley-Omni，HuggingFace 上发布 CocoBro/Foley-Omni 推理权重。ResearchGate 于同期收录（publication 405852241）。V2ST-Bench 评测集截至调研时（2026年7月）仓库标注为 Coming soon，尚未正式放出。

### [Goku](../models/Goku.md)

2025年2月7日首次提交至 arXiv（v1），2025年2月10日更新 v2；后被 CVPR 2025 接收为 Highlight。项目主页 saiyan-world.github.io/goku 与 GitHub 仓库 Saiyan-World/goku 同期上线。

### [Hailuo / MiniMax Video](../models/Hailuo.md)

无技术报告或论文，只有产品发布博客，时间线如下：
- 2024年9月：video-01（初代海螺视频，6秒，1280x720，25fps）
- 2025年1月：S2V-01（主体参考/人物一致性）
- 2025年6月18日：MiniMax Hailuo 02（提出 NCR 架构，原生1080p）
- 2025年10月28日：MiniMax Hailuo 2.3 / 2.3 Fast，同步将 Video Agent 升级为 Media Agent
- 2025年12月15日：开源 VTP 视觉 tokenizer（arXiv:2512.13687），属同团队生成式基础组件，非视频生成模型本体
截至调研时点（2026年7月29日），MiniMax 官网 news 页与 platform 文档中最新的视频模型仍为 Hailuo 2.3，未见 2026 年发布的新一代视频模型公告。

### [HunyuanVideo-Foley](../models/HunyuanVideo-Foley.md)

2025年8月23日 arXiv 提交（arXiv:2508.16930v1，UTC 07:30:18）；2025年8月28日在 GitHub / HuggingFace 正式开源，发布推理代码与 XXL 模型权重；2025年9月29日追加发布 XL 版本权重及 offload 低显存推理支持。此后进入维护迭代阶段，未见 v2 论文修订版。

### [HunyuanVideo](../models/HunyuanVideo.md)

HunyuanVideo：2024年12月3日发布技术报告（arXiv:2412.03603，后有v2修订）并同步开源权重与推理代码；2025年3月发布 HunyuanVideo-I2V 图生视频版本。HunyuanVideo 1.5：2025年11月21日开源，技术报告 arXiv:2511.18870（2025年11月）。

### [InstructAV2AV](../models/InstructAV2AV.md) ⚠️

2026年5月18日提交至 arXiv（arXiv:2605.18467，v1版本）。同期上线项目主页 https://hjzheng.net/projects/InstructAV2AV/、GitHub 仓库 suimuc/InstructAV2AV、HuggingFace 模型 suimu/InstructAV2AV 与数据集 suimu/InsAVE-80K。截至调研时（2026年7月），推理代码、权重、数据集均已释出，训练脚本仍在 roadmap 中标注为进行中。[不确定] 未见正式会议/期刊接收信息，目前为 arXiv 预印本状态。

### [2026 其他音视频联合生成](../models/JAVG_2026_misc.md)

均为 2026 年上半年 arXiv 首发：
- ALIVE：2026年2月9日 v1，2月10日 v2（七项中最早）。
- OmniCustom：2026年2月12日 v1，持续修订至 2026年7月23日 v5（迭代最久）。
- CCL：2026年3月19日（arXiv 编号 2603.18600）。
- Baton：2026年5月24日 v1，2026年6月1日 v2。
- StreamChar：2026年5月25日。
- NAVA：2026年5月28日，随后开源代码与权重。
- ITS-JAVG：2026年6月2日（编号 2606.03183）。
时间上呈现清晰的季度节奏：2 月为大厂基座模型（ALIVE）与定制化任务（OmniCustom）先行，3 月出现「低成本高效路线」（CCL），5—6 月集中出现语义规划（Baton）、流式长时（StreamChar）、原生对齐（NAVA）与推理时缩放（ITS-JAVG）四条新分支，反映 JAVG 领域在 2026 年上半年从「能不能联合生成」转向「如何更省、更长、更准、更可控」。

### [音视频联合生成基线合集](../models/JavisDiT_baselines.md)

(2) MM-Diffusion：2022 年 12 月 19 日提交 arXiv（v1），2023 年 3 月修订，CVPR 2023 录用；是本合集中最早的工作。
(3) AV-DiT：2024 年 6 月 11 日 arXiv 公开（arXiv:2406.07686）。
(1) JavisDiT：2025 年 3 月 30 日 arXiv 首发（v1），2026 年 2 月 22 日修订，被 ICLR 2026 录用；续作 JavisDiT++ 为 2026 年 2 月公开（arXiv:2602.19163），同为 ICLR 2026。
(5) UniAVGen：2025 年 11 月 5 日 arXiv 提交（arXiv:2511.03334），2026 年 3 月 24 日修订。
(4) Harmony：2025 年 11 月 26 日 arXiv 提交（arXiv:2511.21579），2025 年 11 月 28 日修订。

### [可灵 3.0 / 可灵视频 3.0 Omni](../models/Kling_30_Omni.md)

2026年2月4—5日全球正式上线可灵3.0系列（可灵视频3.0、可灵视频3.0 Omni、可灵图片3.0、可灵图片3.0 Omni）。其直接技术前身与可公开引用的官方技术报告为《Kling-Omni Technical Report》（arXiv:2512.16776，2025年12月18日提交，67位作者，署名 Kling Team）；原生音画同出能力最早在2025年12月的可灵2.6上引入。相关同源报告还有《KlingAvatar 2.0 Technical Report》（arXiv:2512.13313，2025年12月）与《Kling-MotionControl Technical Report》（arXiv:2603.03160，2026年3月）。

### [LTX-2](../models/LTX-2.md)

2025年10月首次对外公布 LTX-2（预览/产品化接入 LTX Studio）；2026年1月6日正式开源全部模型权重与训练代码，同日在 arXiv 发布技术报告 arXiv:2601.03233v1（cs.CV，14页，含2页补充材料）；2026年3月5日发布升级版 LTX-2.3（22B，桌面端 LTX Desktop）。前代基础：LTX-Video 2B（2024年11月）、LTXV-13B（2025年5月）。

### [LongCat-Video](../models/LongCat-Video.md)

LongCat-Video 基础模型：2025年10月25日 arXiv v1 提交（2510.22200），10月28日 v2；对外正式发布与开源约在 2025年10月27日。衍生版本：LongCat-Video-Avatar 于 2025年12月23日发布（美团技术团队博客同日发文）；LongCat-Video-Avatar 1.5 于 2026年5月发布（arXiv:2605.26486）。

### [MOVA](../models/MOVA.md)

2026年1月29日首次开源发布（模型权重 + 代码，GitHub/HuggingFace 同步上线）；2026年2月发布 38 页技术报告 arXiv:2602.08794（v2 版本日期为 2026年2月10日，cs.CV）；2026年3月9日上线 API；2026年5月6日补充开源评测代码。

### [合并条目：① Mochi 1](../models/Mochi_MAGI_Motif.md)

① Mochi 1：2024 年 10 月 22 日发布并开源 mochi-1-preview（权重 + 推理代码，Apache 2.0），同日开放 720p HD 版本的预告（后续未按期完整开源）；2024 年 11 月前后陆续开源 AsymmVAE 与微调脚本。无正式论文或 arXiv 技术报告，仅有官方博客。
② MAGI-1：2025 年 4 月 21 日开源 24B 权重与推理代码（GitHub SandAI-org/MAGI-1）；2025 年 5 月 19 日发布 arXiv 技术报告 arXiv:2505.13211《MAGI-1: Autoregressive Video Generation at Scale》（61 页）；2025 年 4 月 30 日发布 4.5B 版本；2025 年 5 月 26 日发布 4.5B Distill；2025 年 5 月 30 日支持 ComfyUI；2026 年 6 月 17 日开源 MAGI-1.1 24B。
③ Motif-Video 2B：2026 年 4 月 14 日在 Hugging Face 发布权重（Motif-Technologies/Motif-Video-2B，Apache 2.0）；arXiv:2604.16503《Motif-Video 2B: Technical Report》，v2 版本更新于 2026 年 5 月 19 日；同期提供 Diffusers 官方集成与社区 GGUF 量化版本。

### [Movie Gen](../models/Movie_Gen.md)

2024年10月4日随官方博客与技术报告发布；arXiv 预印本 2410.13720（v1，2024年10月17日挂出），标题《Movie Gen: A Cast of Media Foundation Models》

### [NVIDIA NeMo Curator](../models/NeMo_Curator.md)

【NeMo Curator 版本线（CalVer 与 SemVer 双轨，PyPI 上传时间为准）】0.6.0（2025-01-07，Dask 架构，纯文本时代）→ 0.7.0（2025-03-12）→ 0.8.0（2025-05-09）→ 0.9.0（2025-07-28）→ 1.0.0 = 25.09 版（2025-10-01，里程碑：后端由 Dask 全面切换为 Ray，首次引入视频与音频模态，形成文本/图像/视频/音频四模态统一架构）→ 1.1.0 = 26.02 版（2026-02-23）→ 1.2.0 = 26.04 版（2026-05-14）→ 1.3.0 = 26.07 版（2026-07-27，本调研截止日前两天发布）。
【Cosmos-Xenna】随 Cosmos 平台于 2025 年从 Cosmos-Curator 中拆分独立开源（Apache 2.0）；26.04 版 NeMo Curator 升级至 Cosmos-Xenna 0.2.0。截至 2026 年 7 月，cosmos-xenna 仓库标注「不再积极开发」，官方引导迁移至 Cosmos 3 / NeMo Curator 体系。
【方法论一手文献】Cosmos World Foundation Model Platform for Physical AI（arXiv:2501.03575，2025-01-07，第3节完整披露七级视频数据 curation pipeline）；Training Video Foundation Models with NVIDIA NeMo（arXiv:2503.12964，2025-03，专门描述 clipping/sharding 双 pipeline 与 GPU 加速）；NVIDIA 官方博客首次公布「89x 加速」为 2025-01。

### [OmniHuman 数据集 + OHBench](../models/OmniHuman.md) ⚠️

2026年4月20日首次提交至 arXiv（arXiv:2604.18326v1），2026年5月30日提交修订版 v2。主类 cs.CV。论文篇幅 19 页、6 张图。截至2026年7月未见明确的会议接收标注（comments 字段仅有页数与图数信息）。代码与评测工具链已在 GitHub（github.com/julia-cherry/OmniHuman）发布，数据集标注已在 HuggingFace（julia527/omnihuman）发布。[不确定]

### [Open-Sora 系列](../models/Open-Sora.md)

【Open-Sora（HPC-AI Tech）】v1.0：2024年3月；v1.1：2024年4月25日（首次发布完整视频数据处理pipeline）；v1.2：2024年6月17日（对应论文 arXiv:2412.20404《Open-Sora: Democratizing Efficient Video Production for All》，2024年12月29日挂arXiv）；v1.3（1B）：2025年2月20日；v2.0（11B）：2025年3月12日发布，技术报告 arXiv:2503.09642《Open-Sora 2.0: Training a Commercial-Level Video Generation Model in $200k》。
【Open-Sora Plan（北大）】v1.0.0：2024年4月；v1.1.0：2024年5月；v1.2.0：2024年7月；v1.3.0：2024年10月；技术报告 arXiv:2412.00131《Open-Sora Plan: Open-Source Large Video Generation Model》于2024年11月28日提交（内容对应 v1.3）；v1.5.0：2025年6月5日发布（8B，SUV 稀疏 MMDiT + 8×8×8 WFVAE）。

### [Ovi](../models/Ovi.md)

论文成稿日期 2025年9月29日，2025年10月1日在 arXiv 公开（arXiv:2510.01284，v1），HuggingFace 模型页标注论文发布日 2025年9月30日；同期在 GitHub（character-ai/Ovi）开源 11B 模型权重与推理代码。2025年11月10日发布 Ovi 1.1 更新（原生 960×960 训练、10 秒生成、数据集扩容一倍）。截至 2026年7月未见 Ovi 2.0 或训练代码发布。

### [Script-a-Video](../models/Script-a-Video.md) ⚠️

2026年4月发布于 arXiv（arXiv:2604.11244）。当前可见最新版本为 v2，标注日期 2026年4月15日（cs.CV）。v1 提交时间在同月稍早（arXiv:2604.11244v1）。截至 2026年7月未见配套的 GitHub 仓库、项目主页或会议接收记录。[不确定]

### [Seedance 2.0 与 Seedance 1.5 pro](../models/Seedance_20_Seedance_15_pro.md)

Seedance 1.5 pro：技术报告 arXiv:2512.13507，v1 于 2025-12-15 提交、v3 于 2025-12-23 更新，产品于 2025 年 12 月上线豆包/即梦/火山方舟（模型 ID：Doubao-Seedance-1.5-pro）。Seedance 2.0：2026 年 2 月初在中国正式发布（模型 ID：doubao-seedance-2-0-260128），技术报告 arXiv:2604.14148 于 2026-04-15 提交，火山引擎 API 于 2026-04-14 全面开放。作为纵向对比基线的 Seedance 1.0 技术报告为 arXiv:2506.09113（2025-06）。

### [SkyReels 系列](../models/SkyReels.md)

SkyReels-V1：2025年2月18日开源（人物中心视频基座模型）。SkyReels-V2：论文2025年4月17日投稿 arXiv:2504.13074（v3 于4月21日修订），2025年4月21日开源推理代码与 SkyCaptioner-V1，4月24日补发720P版本。SkyReels-V3：2026年1月29日开源。SkyReels-V4：论文2026年2月24/25日投稿 arXiv:2602.21818（v3 版本2026年3月18日），2026年3月27日在中关村论坛正式对外发布产品与API；截至2026年3月18日在 Artificial Analysis Video Arena 的「文生视频（含音频）」与「图生视频（含音频）」双榜第一、「文生视频（无音频）」第二。

### [Sora 2](../models/Sora_2.md)

2025年9月30日发布模型与 System Card（同日上线 sora.com 与独立 iOS Sora 应用）；2025年10月上旬开放 API（sora-2 / sora-2-pro）并将时长上限从10秒扩展到15秒（Pro 网页版25秒）；2025年12月与迪士尼达成三年授权协议；2026年3月OpenAI宣布关停Sora消费级应用（应用于2026年4月26日下线，API于2026年9月24日停用），迪士尼10亿美元投资与授权协议随之终止。注意：OpenAI 从未发布 Sora 2 的技术报告或论文，仅有一份7页的 System Card。

### [SpeakerVid-5M](../models/SpeakerVid-5M.md)

2025年7月14日首次提交 arXiv（arXiv:2507.09862，cs.CV）。配套资源随后陆续开放：HuggingFace 数据集仓库 dorni/SpeakerVid-5M-Dataset 创建于 2025年7月18日，最后更新于 2025年8月4日；GitHub 数据清洗代码库 Dorniwang/SpeakerVid-5M-Code 与项目主页 https://dorniwang.github.io/SpeakerVid-5M/ 同期上线。

### [Step-Video-T2V](../models/Step-Video-T2V.md)

2025年2月17日同步开源推理代码与权重、2025年2月17日发布技术报告《Step-Video-T2V Technical Report: The Practice, Challenges, and Future of Video Foundation Model》（arXiv:2502.10248，后有 v2/v3 修订）；2025年2月18日官方对外正式宣布（与语音模型 Step-Audio 同批开源）。衍生模型 Step-Video-TI2V（图生视频）于 2025年3月17日开源，技术报告 arXiv:2503.11251。

### [UniTalking](../models/UniTalking.md)

2026年3月2日提交至 arXiv（arXiv:2603.01418v1，主类 cs.CV，兼投 cs.MM、cs.SD）。论文标注已被 CVPR 2026 接收（Findings Track）。截至2026年7月未检索到代码仓库、模型权重或项目主页的公开发布记录。

### [UniVerse-1](../models/UniVerse-1.md)

2025年9月3日项目主页上线；2025年9月7日 arXiv 提交（arXiv:2509.06155v1，cs.CV，UTC 17:55:03）；2025年9月8日开源模型权重、推理代码与 Verse-Bench 数据集；2025年9月9日技术报告正式发布；2025年9月28日补充发布 Verse-Bench 评测指标工具。另有 OpenReview 投稿记录（forum id 8aFYx2mDyE）。

### [Unison](../models/Unison.md) ⚠️

arXiv:2605.08729。v1 于 2026 年 5 月 9 日提交（cs.CV）；v2 于 2026 年 6 月 29 日更新（本条目主要依据 v2）。v1 与 v2 在训练语料与数据处理描述上完全一致，v2 主要补充了 Zipformer 引用（增补 yao2024zipformer）与部分实验表述。许可为 CC BY 4.0。
论文采用 Springer LNCS 会议模板（\institute 多机构编号格式），且正文明确写有「代码与模型将在论文被接收后公开」（The code and models will be made publicly available upon acceptance），表明截至 2026 年 7 月仍处于会议投稿在审状态，尚未正式发表。未见项目主页、演示页或代码仓库上线。[不确定]

### [Veo 3 / Veo 3.1](../models/Veo_3_Veo_31.md)

Veo 3 于 2025 年 5 月 20 日在 Google I/O 发布，官方 Model Card 首次发布于 2025-05-23，并于 2026-01-13 更新；配套的《Veo: a text-to-video generation system》技术报告同期发布。Veo 3.1 于 2025 年 10 月中旬发布（Flow / Gemini API / Vertex AI 上线）。Veo 3.1 Lite 的 Model Card 发布于 2026-04-08。官方声明「本 Model Card 覆盖 Veo 3 及其后续版本」，即 Veo 3.1 系列沿用 Veo 3 的数据与训练披露。

### [Vidu S1](../models/Vidu_S1.md)

2026年7月：2026年7月6日于全球数字经济大会由生数科技创始人朱军正式发布产品；arXiv 预印本 2607.03118，v2 版本时间为 2026年7月21日（cs.CV，13页）

### [Wan 2.5 / 2.6 / 2.7](../models/Wan.md) ⚠️

系列时间线（均为官方一手可查）：
- Wan 2.1：2025年2月25日开源（1.3B/14B），技术报告《Wan: Open and Advanced Large-Scale Video Generative Models》arXiv:2503.20314 于2025年3月26日发布，60页33图，第3章为完整数据处理章节——这是整个 Wan 系列唯一详细披露数据方法的一手文档。
- Wan 2.2：2025年7月28日开源（T2V-A14B / I2V-A14B MoE、TI2V-5B），Apache 2.0。
- Wan2.2-S2V-14B（音频驱动人物视频）：论文 arXiv:2508.18621 于2025年8月26日提交，权重2025年9月17日上 Hugging Face。
- Wan2.2-Animate-14B：2025年11月上旬开源。
- Wan 2.5-preview：2025年9月下旬（云栖大会期间）以 API 预览版上线，模型名 wan2.5-t2v-preview / wan2.5-i2v-preview；官方文档示例素材日期戳为 2025-09-23 / 2025-09-25，可作时间旁证。系列首次实现「声画同步」。[不确定：具体到日]
- Wan 2.6：2025年12月16日万相官网发布、12月17日阿里云开发者社区正式发布公告；模型名 wan2.6-t2v / wan2.6-i2v / wan2.6-i2v-flash / wan2.6-t2v-us。
- Wan 2.7：2026年6月，官方 API 模型名直接带日期戳 wan2.7-t2v-2026-06-12 与 wan2.7-i2v，为目前推荐主力版本。
- Wan-Dancer-14B（音乐驱动长时舞蹈视频）：2026年7月开源，论文 arXiv:2607.09581。
注：2.5/2.6/2.7 三个版本均无技术报告、无论文、无权重，因此其数据方法只能由 2.1/2.2/S2V 的一手披露与 API 文档能力反推。

### [音视频生成评测基准合集](../models/av_benchmarks.md) ⚠️

【VABench】arXiv:2512.09299，v1 于 2025年12月10日提交，v2 于 2026年4月6日更新（24页/25图，cs.CV + cs.SD，CC BY 4.0）。调研任务描述其为 CVPR 2026 论文，但 arXiv comments 字段未标注录用信息[不确定]。
【AVBench】arXiv:2605.24652，2026年5月发布；项目页图表命名指向 ECCV 投稿[不确定]。
【AV-SyncBench】arXiv:2607.00726，2026年7月发布；已被 Interspeech 2026 接收。
【PhyAVBench】arXiv:2512.23994，v1 于 2025年12月发布（当时为纯 benchmark 设计报告，模型评测留待后续），后续版本补齐 17 个模型的完整评测结果。
【Omni-Judge】arXiv:2602.01623，2026年2月发布。

### [视频 Caption 模型生态](../models/caption_models.md)

生态演进时间线（以公开发布/arXiv 提交为准）：
· 2023：Tag2Text（标签式轻量打标器，被 Allegro 用作粗筛）。
· 2024-02：Panda-70M（arXiv 2402.19479，CVPR 2024），首个系统化的「多教师 + 检索择优」打标范式。
· 2024-06-06：ShareGPT4Video / ShareCaptioner-Video（arXiv 2406.04325），提出 DiffSW 差分滑窗打标。
· 2024-08：CogVideoX（arXiv 2408.06072）披露 Panda-70M→CogVLM→GPT-4→LLaMA2 的四段式打标链；2024-09-19 开源 CogVLM2-Caption 权重。
· 2024-10：AuroraCap + VDC benchmark（arXiv 2410.03051）；Koala-36M（arXiv 2410.08260）。
· 2024-10：Movie Gen 技术报告披露 LLaMa3-Video 8B/70B 打标方案。
· 2025-01-14：Tarsier2-7B（arXiv 2501.07888）；配套 Tarsier2-Recap-585K 数据集。
· 2025-02：video-SALMONN-o1（arXiv 2502.11775）。
· 2025-04：SkyCaptioner-V1（随 SkyReels-V2 开源）。
· 2025-06-18：video-SALMONN 2（arXiv 2506.15220，3B/7B/72B）。
· 2025-09：Qwen3-Omni-Captioner（音频 captioner）开源；2025-10 Omni-Detective / Omni-Captioner 开源。
· 2025-10-12：AVoCaDO（arXiv 2510.10395），首个成体系的音视频联合 captioner。
· 2026-01：LTX-2（arXiv 2601.03233）披露自研音视频联合 captioner。
· 2026-03：Qwen3.5-Omni（arXiv 2604.15804）把「剧本级结构化音视频 caption」列为一等能力，Omni-Cloze benchmark 发布。
· 2026-06：OmniCap-IF（arXiv 2606.08572），首个音视频 caption 指令跟随基准。
· 2026-07-14：AVSCap + AVSCapBench（arXiv 2607.12820），最新的音视频联合 caption 模型与专用基准。

### [几何/结构化标注数据集合集](../models/geometric_datasets.md)

SceneScribe-1M：arXiv 2604.07990，2026年4月9日提交、4月26日修订，已被 CVPR 2026 接收；SpatialVID：arXiv 2509.09676，2025年9月11日 v1、2025年12月18日 v2；WildWorld：arXiv 2603.23497，2026年3月24日提交；Action100M：arXiv 2601.10592，2026年1月15日提交

### [视频生成后训练数据策略](../models/post_training_data.md)

锚论文 arXiv:2604.25427v1 提交于 2026年4月28日（cs.CV，CC BY 4.0）。横向对象覆盖 2024年8月（CogVideoX）至 2026年上半年（Seedance 1.5 pro、Kling 3.0 Omni、SkyReels-V4、Cosmos-Predict 2.5、HunyuanVideo 1.5、LongCat-Video 等）。作为方法论支撑的关键论文时间线：ImageReward(2023)→VideoReward/Improving Video Generation with Human Feedback(2025-01, arXiv:2501.13918)→DanceGRPO(2025-05, arXiv:2505.07818)→Flow-GRPO(2025-05)→MixGRPO(2025-07)→HPSv3(ICCV 2025, arXiv:2508.03789)→Self-Forcing(2025-06)→OmniForcing(2026-03)、Causal Forcing(2026-02)、Astrolabe(2026-03)。

### [主流视频预训练数据集合并调研：Panda-70M、InternVid、Koala…](../models/pretraining_datasets.md)

Panda-70M：2024年2月29日 arXiv 首发（arXiv:2402.19479），CVPR 2024；2024年10月追加 desirability 过滤与镜头边界标注。InternVid：2023年7月13日 v1、2024年1月4日 v2（arXiv:2307.06942），ICLR 2024 spotlight。Koala-36M：2024年10月10日 v1、2025年4月26日 v2（arXiv:2410.08260），CVPR 2025。MiraData：2024年7月8日（arXiv:2407.06358，仅 v1），NeurIPS 2024 D&B；v1 数据于2024年7月10日发布，v0 beta 更早。OpenVid-1M：2024年7月1日发布（arXiv:2407.02371），ICLR 2025 接收（2025年1月）；OpenVidHD-0.4M 独立下载包2025年5月30日补充。UltraVideo：2025年6月16日（arXiv:2506.13691，仅 v1），NeurIPS 2025 D&B poster。LVD-2M：2024年10月14日 arXiv、10月15日数据发布（arXiv:2410.10816），NeurIPS 2024 D&B。

## 类型（模型/数据集/工具链/评测基准）

`type` · 详细程度: minimal

### [Allegro](../models/Allegro.md)

模型（文生视频 / 图生视频的 Diffusion Transformer 基础模型）+ 配套工具链（自研 VideoVAE 175M、VideoDiT 2.8B、基于 Aria 微调的视频 caption 模型、完整训练与推理代码）。论文本身的核心卖点即「公开商用级视频生成模型的黑箱」，因此其数据处理 pipeline 的披露粒度是同期开源工作中最细的之一

### [Apollo](../models/Apollo.md)

模型（统一多任务音视频联合生成基础模型，26B 参数）。附带产出：论文自称构建了「首个大规模带密集 caption 的音视频数据集」（81M 样本）及配套自动化数据构造 pipeline，但该数据集与 pipeline 均未发布，因此不构成可用的数据集产出。评测沿用第三方基准 Verse-Bench，非自建基准。

### [CineDance / CineDance-1M](../models/CineDance.md)

以数据集为主体的复合型工作，三位一体：
【主产出】数据集 —— CineDance-1M，首个面向多镜头长篇（multi-shot long-form）音视频联合生成的 1080p 大规模开放研究数据集，规模 1,021,657 条叙事序列 / 约 26.3K 小时。
【副产出 1】评测基准 —— CineBench，1000 条分层测试 prompt + 六维度人类对齐指标体系。
【副产出 2】模型 —— CineDance，由 LTX-2.3 适配而来的开源基线（13B 视频 + 3B 音频 + 3B 跨模态注意力），用于验证数据集有效性。
【副产出 3】工具链 —— 三阶段数据策展 pipeline（清洗 / 叙事解析 / 双模态标注），论文承诺开源但尚未释出。

### [CogVideoX](../models/CogVideoX.md)

模型（文生视频/图生视频扩散 Transformer 基础模型系列）+ 配套工具链（3D 因果 VAE、CogVLM2-Caption 视频打标模型、caption upsampler 提示词改写、微调与推理代码库 zai-org/CogVideo）

### [Cosmos-Predict2.5](../models/Cosmos-Predict25.md)

模型（视频世界基础模型 World Foundation Model）+ 配套工具链 + 自建评测基准。主体为基于 flow matching 的 DiT 视频生成模型，单一模型统一 Text2World / Image2World / Video2World 三种模式，发布 2B 与 14B 两档；同时发布 Cosmos-Transfer2.5 控制网系列（Sim2Real / Real2Real 世界翻译）与多个领域特化变体（自动驾驶 7 视角多视图、机器人动作条件、AgiBot 三视角、GR00T GR1）。评测侧使用自建的 PAI-Bench（Physical AI Bench）。数据处理侧关联 NVIDIA 独立开源的 Cosmos Curator / Cosmos-Xenna 与 NeMo Curator 视频 curation 框架。

### [Data-Juicer 2.0](../models/Data-Juicer.md)

工具链（数据处理系统 / 算子库 / 数据-模型协同开发平台），不是生成模型也不是数据集。可细分为三个产出层次：
1. 【算子库与执行引擎】Data-Juicer 2.0 本体——截至 v1.5.4 共 229 个算子（Mapper 138、Filter 58、Deduplicator 10、Formatter 8、Selector 5、Aggregator 4、Grouper 3、Pipeline 3），覆盖文本/图像/音频/视频/多模态五类模态，配套 Ray、MaxCompute、单机等多种自适应执行后端。
2. 【数据-模型协同开发套件】Data-Juicer Sandbox——提供「探测-分析-精炼」（Probe-Analyze-Refine）工作流，把数据配方搜索与模型训练/评测串成闭环，内置对 EasyAnimate、T2V-Turbo、ModelScope、VBench 等训练与评测框架的接入。
3. 【衍生数据集与配方】以 Sandbox 在文生视频上的实践为例，开源了经筛选的 T2V 最优数据池（datajuicer/data-juicer-t2v-optimal-data-pool 等 HuggingFace 数据集）与对应 YAML 配方，属于「方法+工具+数据配方」三位一体的产出。
因此在本调研的谱系中，Data-Juicer 属于「数据基础设施」类目，与 NVIDIA NeMo Curator 是最直接的对标关系。

### [Foley-Omni](../models/Foley-Omni.md)

模型 + 评测基准 + 数据处理pipeline方法论 的组合产出。主体是视频到完整配乐（V2ST）的统一多模态音频生成模型（约5.5B参数量级，基于 Diffusion Transformer）；同时提出 V2ST-Bench 评测基准（300条视频-音频-文本三元组）；论文第3.1节完整描述了一条音视频数据清洗与结构化标注pipeline。注意：本项目生成的是音频/配乐，视频侧为条件输入而非生成目标，因此属于「音视频生成」谱系中的V2A/V2ST分支，而非文生视频模型。

### [Goku](../models/Goku.md)

模型（联合图像与视频生成基础模型族，基于 rectified flow 的 Transformer/DiT）。论文同时详细披露了一条五阶段数据处理流水线（可视为工具链方法论描述，但流水线代码与数据本身未开源）。非数据集、非评测基准。

### [Hailuo / MiniMax Video](../models/Hailuo.md)

模型（闭源商用视频生成模型/产品线，覆盖文生视频 T2V、图生视频 I2V、首尾帧 First-and-Last-Frame、主体参考 Subject-Reference 等模式）。不是数据集，不是工具链，不是评测基准；MiniMax 也未发布配套的视频评测基准。

### [HunyuanVideo-Foley](../models/HunyuanVideo-Foley.md)

模型（并附带完整的推理工具链）。主体是一个 text-video-to-audio（TV2A）音效生成模型，即给定视频 + 文本描述生成与画面同步的 Foley 音效。不是视频生成模型本身，而是与视频生成模型配套的「配音/配效」环节模型。同时附带一个自研的音频 VAE（DAC-VAE，48kHz）。不发布数据集，不发布评测基准（评测复用他人的 Kling-Audio-Eval、MovieGen-Audio-Bench、VGGSound-Test）。

### [HunyuanVideo](../models/HunyuanVideo.md)

模型（开源视频生成基础模型），并附带完整的开源代码与推理框架。原版13B为当时参数量最大的开源视频生成模型；1.5为8.3B轻量化版本，主打消费级显卡（约14GB显存）可跑。二者均为纯视频生成模型，不是数据集、不是评测基准。

### [InstructAV2AV](../models/InstructAV2AV.md)

模型 + 数据集 + 数据合成pipeline（data engine）的三位一体产出，而非单纯的视频生成模型。
【模型】InstructAV2AV，基于 Ovi 双塔对称 DiT 架构改造的指令引导音视频联合编辑模型，输入为「源音视频 + 文本指令」，输出为「编辑后的音视频」。注意其任务是编辑（editing）而非从零生成（generation）。
【数据集】InsAVE-80K，论文自称是首个大规模音视频编辑数据集，含 source-to-target 配对样本。
【pipeline】论文第3节描述的可扩展数据合成引擎，是本工作在数据维度上的主要贡献，也是本次调研关注的重点。
【评测】未提出独立命名的评测基准，而是从 InsAVE-80K 中划出 1K 人工精选样本作为评测集，并额外在已有的 AvED-Bench 上做零样本泛化评测。

### [2026 其他音视频联合生成](../models/JAVG_2026_misc.md)

七项均为学术论文形态，但性质分四类：
【基座模型类】ALIVE（VideoDiT 12B + AudioDiT 2B 的完整 T2VA/R2VA 基座模型，附 Alive-Bench 1.0 评测基准）、NAVA（6.3B 原生对齐联合生成模型，附开源权重与训练代码）。
【模块/方法类（在已有基座上做增强）】Baton（在联合 DiT 前加 VA-Planner 语义规划器 + RS-RoPE，属方法增强）、CCL（针对双流 Transformer 范式的四个模块级改进 TARP/LCT/DCR/UCG，主打省数据省算力）。
【任务+数据集类】OmniCustom（提出「同步音视频定制」新任务，核心贡献之一是自建数据集 OmniCustom-1M，并配 100 例评测集）。
【系统/工程类】StreamChar（长时流式生成系统，含两阶段蒸馏与实时推理管线）。
【推理时算法类】ITS-JAVG（完全 training-free 的推理时缩放算法 + 多验证器组合研究 + ARW 优化算法，无任何模型训练，本质上是「用评测模型做推理时数据筛选」）。
从数据调研角度：ALIVE、NAVA、OmniCustom 三者含实质数据处理 pipeline 披露；Baton、CCL、StreamChar 仅有数据来源清单；ITS-JAVG 无训练数据但其「验证器组合」思想与数据质检打分器高度同构。

### [音视频联合生成基线合集](../models/JavisDiT_baselines.md)

全部为「模型」，其中三项额外附带评测基准或数据集产物：
- JavisDiT/JavisDiT++：模型 + 评测基准（JavisBench，10,140 条；JavisBench-mini，1,000 条）+ 同步性评测指标（JavisScore）+ 开源训练/推理工具链；是四者中唯一同时交付「模型 + benchmark + metric + 完整训练代码」的工作。
- MM-Diffusion：模型 + 自建数据集（Landscape，1,000 条 10 秒自然场景音视频片段）+ 开源代码与预训练权重。
- AV-DiT：纯模型（参数高效的联合生成架构），无新数据集、无新基准。
- Harmony：模型 + 评测基准（Harmony-Bench，150 条，分三档子集）。
- UniAVGen：模型（统一框架，支持联合生成、视频到音频配音、音频驱动视频动画等多任务），无新数据集。

### [可灵 3.0 / 可灵视频 3.0 Omni](../models/Kling_30_Omni.md)

闭源商业模型（视频生成/编辑基础模型 + 多模态统一生成模型），通过可灵AI网页版、App 与开放平台 API（含 Replicate/fal 等第三方托管）提供服务；非数据集、非工具链、非评测基准

### [LTX-2](../models/LTX-2.md)

模型（开源的文本到「音频+视频」联合生成基础模型，T2AV），并附带开源工具链（ltx-core / ltx-pipelines / ltx-trainer 推理与训练微调代码、ComfyUI 与 Diffusers 集成、相机/姿态/唇形配音等控制 LoRA）。非数据集、非评测基准。

### [LongCat-Video](../models/LongCat-Video.md)

模型（视频生成基础模型 + 配套开源推理代码/算子）。13.6B 参数的 Diffusion Transformer，单一模型统一支持 文生视频(T2V)、图生视频(I2V)、视频续写(Video-Continuation) 三类任务；并开源了 Block Sparse Attention 的前向/反向实现（可视为工具链组件）。非数据集、非评测基准（但报告内含自建内部评测集）。

### [MOVA](../models/MOVA.md)

模型（音视频联合生成基础模型）。同时附带三项衍生产出：(1) 完整开源代码库（训练 pipeline、推理、LoRA 微调、prompt 增强工作流、评测代码）；(2) 自建的六类场景音视频联合生成评测基准（与 Verse-Bench 配合使用）；(3) Arena 式人类偏好评测协议。不是数据集发布（训练数据本身未开源）。

### [合并条目：① Mochi 1](../models/Mochi_MAGI_Motif.md)

三者均为「模型 + 配套工具链」：
① Mochi 1：10B 参数的 Asymmetric Diffusion Transformer（AsymmDiT，48 层，视觉维度 3072 / 文本维度 1536）文生视频模型 + 自研 AsymmVAE（362M 参数，8×8 空间 + 6× 时间压缩至 12 通道潜空间，总压缩率 96×）。输出 848×480、30 FPS、最长 5.4 秒（84 帧）。
② MAGI-1：块级自回归扩散（chunk-wise autoregressive denoising）视频世界模型，最大 24B 参数，每 chunk 24 帧，上下文最长 400 万 token，支持 T2V / I2V / V2V 视频续写统一建模与流式生成；配套自研 Transformer VAE（8× 空间 / 4× 时间下采样，16 通道）、MagiAttention 分布式注意力库、Shortcut Model 蒸馏。
③ Motif-Video 2B：2B 参数的 rectified flow matching 文生视频模型 + 图生视频扩展，核心创新为 Shared Cross-Attention 与三段式骨干功能分解（早期融合 / 联合表征 / 细节解码），训练侧使用 TREAD token 路由与 REPA 表征对齐；额外贡献一套「离线 bucket 均衡采样器」数据基础设施。定位明确为「micro-budget（微预算）」路线的对照实验。

### [Movie Gen](../models/Movie_Gen.md)

模型（媒体生成基础模型家族：Movie Gen Video 30B 文生视频、Personalized Movie Gen Video、Movie Gen Edit 视频编辑、Movie Gen Audio 13B 音频生成）+ 评测基准（Movie Gen Video Bench、Movie Gen Audio Bench）

### [NVIDIA NeMo Curator](../models/NeMo_Curator.md)

工具链 / 数据基础设施框架（data curation toolkit）。不是生成模型，不是数据集，也不是评测基准。定位为「可复现的、GPU 加速的大规模数据处理流水线构建框架」，覆盖文本、图像、视频、音频四种模态的加载—过滤—去重—标注—变换—写出全流程。在本调研的对象体系中，它属于「被其他视频生成/世界模型团队使用的上游基建」，NVIDIA 官方口径称其为工业级视频数据处理的开源参考实现。

### [OmniHuman 数据集 + OHBench](../models/OmniHuman.md)

数据集 + 评测基准 + 数据处理工具链（三位一体），不含生成模型。三部分构成：(1) OmniHuman 数据集——100万条视频 / 1800小时 / 8万个身份的人物中心音视频数据集，带层次化标注；(2) 全自动数据采集与多模态标注 pipeline——四级过滤 + 跟踪 + 音视觉归属 + 两阶段 caption 生成；(3) OHBench——三级（全局/关系/个体）七维度评测体系，含 509 条评测视频与配套指标工具包。论文本身不训练新模型，仅用 LTX-2 微调作为数据有效性的验证实验。

### [Open-Sora 系列](../models/Open-Sora.md)

均为「模型 + 完整工具链」双重属性的开源项目：既发布模型权重（T2V/I2V 扩散模型 + VAE），也发布端到端训练代码与数据处理代码，同时附带部分标注数据集。非评测基准。Open-Sora 额外具备「训练成本工程化范本」属性（逐阶段成本核算），Open-Sora Plan 额外具备「社区协作复现范本」属性。

### [Ovi](../models/Ovi.md)

模型（开源的文本/文本+图像到「音频+视频」一次性联合生成模型，T2AV / I2AV），并附带开源推理工具链（推理脚本、Gradio 应用、多卡序列并行推理、fp8/qint8 量化、ComfyUI 集成）。不是数据集，也不是评测基准（评测借用他人提出的 Verse-Bench）。

### [Script-a-Video](../models/Script-a-Video.md)

方法/表示范式论文，兼具「打标 schema 定义 + 内部数据集构建 + 下游生成模型改造」三重属性：
【主体】提出 MTSS 结构化音视频 caption 表示范式（一套 JSON 式的四流 schema 定义），属于打标方法论而非模型或数据集。
【附带产出 1】一个 500K 视频片段的内部 MTSS 标注数据集（未开源）。
【附带产出 2】一个基于 Qwen3-Omni-Instruct 微调的专用 caption 模型 Qwen3-Omni-MTSS-FT（未开源）。
【附带产出 3】基于 LTX-2 改造的多镜头音视频联合生成模型（引入 Shot-Aware Structured Attention 与 Identity Customization 两处架构改进，未开源）。
【附带产出 4】一个内部评测集（125 条单镜头 + 100 条多镜头样本，未开源）。
不属于评测基准发布——评测使用的是 Video-SALMONN-2、UGC-VideoCap、Daily-Omni、WorldSense 等已有基准。

### [Seedance 2.0 与 Seedance 1.5 pro](../models/Seedance_20_Seedance_15_pro.md)

闭源商用模型（视频/音视频联合生成基础模型）+ 配套自研评测基准（SeedVideoBench 1.0/1.5/2.0）。两份技术报告本身更接近 model card + 评测报告，而非完整方法论论文。

### [SkyReels 系列](../models/SkyReels.md)

模型为主，兼有工具链与评测基准三类产物：(1) 模型——SkyReels-V2 视频生成基座（1.3B/5B/14B，540P/720P，含 T2V、I2V、Diffusion Forcing 长视频、Camera Director 变体）与 SkyReels-V4 统一多模态音视频基座（生成+inpainting+editing）；(2) 工具链——SkyCaptioner-V1 结构化视频打标模型（基于 Qwen2.5-VL-7B-Instruct，已开源）及其推理代码、SkyReels-V2 训练/推理框架；(3) 评测基准——SkyReels-Bench（V2，人评基准）与 SkyReels-VABench（V4，2000+条音视频联合评测提示集）。

### [Sora 2](../models/Sora_2.md)

闭源商业模型（原生音视频联合生成的视频生成基础模型 + 消费级社交应用 + API 服务）。非数据集、非工具链、非评测基准。

### [SpeakerVid-5M](../models/SpeakerVid-5M.md)

数据集（大规模音视频双人交互人体生成数据集）为主体，同时附带两项衍生产出：(1) 一个自回归式 video chat 基线模型（0.8B 可训练参数）；(2) 一个配套评测基准 VidChatBench（500 组测试对，含六维指标体系）。此外开源了完整的数据清洗（data curation）代码库，因此也兼具工具链属性。它不是生成模型本身，而是 MOVA 等音视频联合生成模型的上游数据供给方之一。

### [Step-Video-T2V](../models/Step-Video-T2V.md)

模型（开源文生视频基础模型，30B 参数 DiT + 自研深度压缩 Video-VAE），同时附带评测基准 Step-Video-T2V-Eval（128 条中文 prompt、11 个类目）与开源推理代码。不是数据集、不是数据工具链——训练数据与数据处理代码均未发布。

### [UniTalking](../models/UniTalking.md)

模型（音视频联合生成框架）。主体是 10B 参数的双流 MM-DiT 音视频联合生成模型 UniTalking，视频分支继承 Wan2.2-5B 权重、音频分支为其对称孪生结构并随机初始化。不发布数据集（230万样本的自建数据集未开源），不发布评测基准（评测复用 Seed-TTS test-en、MiniMax Multilingual Test Set 以及自建的50条 prompt 主观测试集，后者亦未公开）。

### [UniVerse-1](../models/UniVerse-1.md)

模型 + 评测基准的组合产出。主体是音视频联合生成模型（UniVerse-1-Base，7B 参数）；同时发布 Verse-Bench 评测基准（600 条图文 prompt 对，含配套评测指标工具）。不是数据集发布——7,685 小时训练数据本身未开源。附带完整推理代码，但不含训练代码与数据清洗脚本。

### [Unison](../models/Unison.md)

模型（音视频联合生成框架）。这是本条目最需要澄清的一点：Unison 不是数据集发布工作，也不是评测基准。
【是模型】主体产出是一个双分支音视频联合生成框架（Wan2.2-5B 视频分支 + MMAudio/Zipformer 音频分支），核心贡献为三项方法创新（语义引导的谐和策略 SGHS、双向音频交叉注意力 Bi-ACA、双向跨模态 forcing 策略 CMFS）。
【不是数据集】约 200 万同步音视频 clips / 3,000+ 小时的语料是「聚合已有开源数据集 + 自动化 pipeline 精炼」的产物，既未命名、未统计发布、也未开源，仅在实验设置一节以两句话交代。任务描述中的「自动化 pipeline 聚合多个大规模数据集」是准确的，但需注意该 pipeline 在论文中几乎没有展开描述。
【自建评测集】另构建了 1,000 条 held-out 样本的内部测试集（标注由 Gemini 生成），但未命名、未开源，不构成公开评测基准。
【任务形态】主任务为 TI2AV（文本+图像 → 音频+视频），同时支持 T2AV（纯文本 → 音视频）、A2V（音频 → 视频）、V2A（视频 → 音频）四类。

### [Veo 3 / Veo 3.1](../models/Veo_3_Veo_31.md)

模型（闭源商业视频+音频联合生成基础模型，通过 Gemini App、Flow、Google Vids、Google AI Studio、Gemini API、Vertex AI 提供服务）

### [Vidu S1](../models/Vidu_S1.md)

模型（实时交互式流式音视频联合生成模型，语音驱动数字人/虚拟角色）。同时自建了配套评测基准 Vidu-StreamBench（500样本，内部基准，未见开源）

### [Wan 2.5 / 2.6 / 2.7](../models/Wan.md)

模型（视频生成基础模型系列）。2.5/2.6/2.7 为闭源商用 API 模型；同族另有开源模型（Wan2.1/2.2/S2V/Animate/Dancer）、开源推理与微调工具链（GitHub Wan-Video 组织、ComfyUI/Diffusers 集成、Wan-skills Agent 技能包），以及自研评测基准 Wan-Bench（3大维度14项细粒度指标）。非数据集条目。

### [音视频生成评测基准合集](../models/av_benchmarks.md)

评测基准（benchmark）。五者均为音视频联合生成（Audio-Video Generation）方向的评测体系，但侧重不同：
- VABench：综合型全维度基准（T2AV / I2AV / 立体声 AV 三类任务，七大内容类目 + 15 项评测维度）；
- AVBench：人类对齐的自动化评测基准 + 可训练的专用评测器（10 维度，附带 300K 偏好训练数据，评测器本身可复用为数据过滤器与 RLHF reward）；
- AV-SyncBench：专项同步性基准（时序同步与语义同步解耦），同时也是一个带扰动标注的数据集；
- PhyAVBench：物理常识专项基准，附带自录真实数据集 PhyAV-Sound-11K（11,605 条视频 / 25.5 小时）；
- Omni-Judge：评测方法学研究（探究 Omni-LLM 能否充当人类对齐的裁判），属于 meta-evaluation。
其中 AVBench、AV-SyncBench、PhyAVBench 同时具备「数据集」属性。

### [视频 Caption 模型生态](../models/caption_models.md)

模型 + 工具链 + 评测基准的复合生态。三层结构：
(1) 打标模型（captioner）本体——通用 VLM（Qwen-VL / InternVL / LLaVA 系）、专用 captioner（ShareCaptioner-Video、Tarsier2、CogVLM2-Caption、SkyCaptioner-V1、AuroraCap、AVoCaDO、AVSCap）、全模态 captioner（Qwen3-Omni-Captioner、video-SALMONN 2、Qwen3.5-Omni）；
(2) 由打标器产出的数据集（ShareGPT4Video-40K / 4.8M、Tarsier2-Recap-585K、Panda-70M、Koala-36M、AVoCaDO-SFT-107K、AVSCap-130K）——打标模型与数据集互为因果，是本生态的核心特征；
(3) 评测基准（DREAM-1K、VDC/VDCScore、VidCapBench、AVSCapBench、UGC-VideoCap、Omni-Cloze、OmniCap-IF、video-SALMONN 2 testset）。

### [几何/结构化标注数据集合集](../models/geometric_datasets.md)

数据集（四者均为大规模视频数据集，附带自建标注流水线；WildWorld 同时发布 WildBench 评测基准，Action100M 同时发布 VL-JEPA 预训练模型，SpatialVID 发布完整标注 pipeline 代码）

### [视频生成后训练数据策略](../models/post_training_data.md)

专题综述条目（cross-cutting topic），非单一模型/数据集/工具链/评测基准。锚论文本身类型为「方法论技术报告」（a practical blueprint / systematic framework），不发布模型权重、不发布数据集，属于四阶段后训练流水线的工程蓝图；其余横向对象则分属模型（多数）、数据集（HPDv3、VideoReward 偏好集）与奖励模型（HPSv3、VideoAlign/VideoReward、RewardDance、VisionReward）。

### [主流视频预训练数据集合并调研：Panda-70M、InternVid、Koala…](../models/pretraining_datasets.md)

全部为「数据集」，但均附带不同程度的工具链与配套模型：Panda-70M 附切分代码 + 蒸馏 caption 学生模型；InternVid 附 ViCLIP 视频-文本模型；Koala-36M 附转场检测代码 + VTSS 打分模型（简化版）；MiraData 附 GPT-4V 打标脚本 + MiraBench 评测基准（兼具「评测基准」属性）；OpenVid-1M 附 MVDiT 生成模型与训练代码；UltraVideo 附 UltraWan-1K/4K 生成模型（LoRA）；LVD-2M 仅附 YouTube 下载脚本。七者均无商用级生成模型发布。

## 开源程度（权重/代码/数据/pipeline各自是否开源）

`openness` · 详细程度: brief

### [Allegro](../models/Allegro.md)

整体开放度较高，但数据本体未开放：
· 权重：完全开源，Apache 2.0 许可（允许商用），Hugging Face rhymes-ai/Allegro、rhymes-ai/Allegro-TI2V，含 175M VideoVAE + 2.8B VideoDiT。
· 代码：推理代码与训练代码均开源（GitHub rhymes-ai/Allegro），训练需用户自备 .parquet 格式数据集。
· Pipeline：以论文形式高粒度公开——7 级过滤漏斗的每一步工具名（PySceneDetect / DOVER / LPIPS / UniMatch / LAION Aesthetics Predictor / CRAFT / Tag2Text / CLIP）与每个训练阶段的逐项阈值表（Table 1）全部披露，并给出各阶段的保留量。但清洗代码、阈值配置脚本未随权重开源。
· 打标模型：Aria（rhymes-ai/Aria，25.3B 总参 / 3.9B 激活的多模态原生 MoE，Apache 2.0，arXiv:2410.05993）已开源，但用于视频 caption 的微调版本未单独发布。
· 数据：训练数据集本身未开源，仅说明基于 WebVid / Panda-70M / HD-VILA / HD-VG / OpenVid-1M 等公开数据集构建。

### [Apollo](../models/Apollo.md)

完全闭源，是本次调研样本中开放度最低的一档。
【权重】未开源，论文与 HuggingFace 论文页均无模型链接。
【代码】未开源，论文中未给出 GitHub 仓库或项目主页 URL。
【数据】81M 样本的音视频-caption 三元组数据集未发布，论文对其「首个大规模带密集 caption 音视频数据集」的定位仅作贡献声明，无任何公开承诺。
【pipeline】方法论层面做了框架级披露（四阶段漏斗、所用打标模型清单、对齐检测工具、27% 保留率），但缺少可复现的关键要素：无逐指标阈值表、无逐级输入/输出量、无 prompt 原文、无伪代码、无数据处理脚本。相较同期开源工作（如 MOVA 公开 Table 9 全阈值表与打标 prompt 原文），Apollo 的数据披露停留在「说了做什么、没说怎么做」的层次，唯一超出同行的是给出了 27% 的端到端定量保留率。

### [CineDance / CineDance-1M](../models/CineDance.md) ⚠️

开放程度中等偏上，但截至 2026年7月仍处于「分批释出中」状态：
【数据】部分开放。HuggingFace 上以 gated（门控申请）方式发布，首批 CineDance_01 为四分片中的第 1 份，含约 240,488 条视频片段、150 个 TAR 归档、总计 5.83 TB，目前仅含视频本体（video only，原生音轨保留在视频容器内），结构化标注文件尚未随首批放出。许可为 CC-BY-NC-SA-4.0（署名-非商业性使用-相同方式共享），明确限定非商业研究与教育用途。下载需 HuggingFace token 且需人工审核申请。
【代码】未开源。GitHub 仓库 github.com/AliothChen/CineDance 已建立，但 curation pipeline 代码、推理套件、推理代码、训练代码均列为待发布（pending）。
【权重】未开源。CineDance 模型 checkpoint 列为待发布。
【pipeline】方法层面披露充分——三阶段流程的每一级工具选型（EasyOCR、FFmpeg 黑边检测、TransNetV2、Qwen3.5-27B、Qwen3.5-35B-A3B、Qwen3-Omni-30B-A3B）、漏斗各级定量数据、标注 schema 字段、消融对比表均在正文给出，可被第三方重新实现；但具体 prompt 原文与过滤阈值数值未完整公开。
【依赖声明】README 致谢 LTX-2、Qwen 系列、vLLM。仓库本身未标注 code license。[不确定]

### [CogVideoX](../models/CogVideoX.md)

开源程度在同期闭源大厂中属于较高的一档，但数据本身未开源：
· 权重：开源。CogVideoX-2B（Apache 2.0）、CogVideoX-5B / 5B-I2V、CogVideoX1.5-5B / 1.5-5B-I2V 均在 Hugging Face（THUDM/zai-org 组织）与 SAT 格式下公开发布，允许商用。
· 代码：开源。推理、微调（LoRA/SFT）、SAT 与 diffusers 两套实现均在 GitHub https://github.com/zai-org/CogVideo （原 THUDM/CogVideo）。
· 数据处理 pipeline：以论文形式公开描述（第3.4节 Data + 附录 G「Dense Video Caption Data Generation」+ 附录 K「Data Filtering Details」），公开了负面标签体系、6 个 Video-LLaMA 分类器及其测试集准确率表、caption 生成链路与 GPT-4 摘要 prompt 全文；但清洗代码与阈值配置文件未开源，数据集本身未开放。
· 打标模型：开源。CogVLM2-Caption（huggingface.co/zai-org/cogvlm2-llama3-caption）与 3D VAE 权重均已发布，是「数据 pipeline 公开」的最实质体现——外部可直接复现其打标环节。
· 训练数据：未开源，35M clips 与 2B 图像的具体来源、清单均未公布。
· CogSound（音效模型）：闭源，仅在清影产品与 API 中提供，无技术报告。

### [Cosmos-Predict2.5](../models/Cosmos-Predict25.md)

开放程度在同级别闭源大厂模型中偏高，但训练数据本身不开放。
【已开源】(1) 源码：GitHub nvidia-cosmos/cosmos-predict2.5 与 nvidia-cosmos/cosmos-transfer2.5，采用 Apache 2.0 License；(2) 预训练与后训练检查点：2B / 14B 的 pre-trained、post-trained 版本，以及蒸馏版与领域特化版（auto/multiview、robot/action-cond、robot/multiview-agibot、robot/gr00tdream-gr1），发布于 Hugging Face，遵循 NVIDIA Open Model License；(3) curated benchmarks（PAI-Bench 相关评测集）与「curated post-training examples」（精选后训练示例）；(4) 数据 pipeline 的工程底座间接开源——Cosmos Curator（github.com/NVIDIA/cosmos-curator）与其 GPU 流式执行框架 Cosmos-Xenna 已独立开源，NeMo Curator 的视频 curation 模块亦公开，二者即论文所述 curation pipeline 的产品化形态。
【未开源】200M 预训练视频数据集本身；自研的内容类型分类器（26 类 taxonomy）、内部美学/运动/OCR/感知质量/语义 artifact 打分器的权重与阈值；NVIDIA 内部驾驶平台采集的 3.1M 段 7 相机专有数据；各过滤器的具体超参。
【定位】论文明确目标是「lower the barrier to adoption」，把预训练模型交给社区做领域特化，而非开放数据。

### [Data-Juicer 2.0](../models/Data-Juicer.md)

开源程度极高，是本调研中开放度最高的一类对象。
【代码】完全开源。GitHub datajuicer/data-juicer（原 modelscope/data-juicer），Apache 2.0 许可证，约 6.8k stars，持续高频迭代（2026年内已发布 v1.4.5 至 v1.5.4 共7个版本）。229个算子的实现全部可读可改，v1.5.3 一次性新增409个测试用例，工程规范度较高。
【算子与配方】开源。config_all.yaml 暴露全部算子及其超参；DJ-Cookbook 维护20+条即用型数据配方（含视频数据合成、对比学习、课程学习等垂直场景配方）。
【数据】部分开源。文生视频案例中筛选得到的最优数据池以 HuggingFace 数据集形式公开：datajuicer/data-juicer-t2v-optimal-data-pool 共 147,176 条样本、约 227.5GB，Apache 2.0 许可；但底层原始素材（InternVid、Panda-70M、MSR-VTT）需按各自原始协议自行获取，DJ 公开的是筛选后的样本索引与元数据。
【模型权重】Sandbox 论文声明所训练的 T2V 模型（基于 T2V-Turbo/EasyAnimate 微调）连同代码、数据一并开源。
【pipeline】完全公开。不仅公开方法，还公开可执行的 YAML 配方与阈值数值（如 CLIP 相似度阈值 0.306337），复现门槛低于绝大多数模型侧工作。
【文档】中英双语文档站（datajuicer.github.io），含算子提要（Operator Schemas）逐条说明。

### [Foley-Omni](../models/Foley-Omni.md)

开源程度中等偏上。
【权重】开源。HuggingFace CocoBro/Foley-Omni 发布 inference-only 权重（v2st.pth），同包分发音频VAE、BigVGAN vocoder、视觉特征抽取模型等依赖组件，声明为 MIT 许可证。
【代码】部分开源。GitHub NJU-Speech/Foley-Omni 提供推理代码与视觉特征预处理脚本（CLIP特征、Synchformer sync特征抽取），训练代码未见发布。仓库自身许可证未在页面明确标注，并声明重分发了 Wan2.2-TI2V-5B、MMAudio、Ovi 等上游项目组件，需分别遵循上游许可。
【数据】训练数据不开源。约4.9k小时、2.7M对训练语料中自采/内部部分（internal）不公开；公开数据集部分（LJSpeech、LibriTTS、AudioCaps、Freesound、MusicCaps、MusicBench、AudioSet、VGGSound、GRID、LRS2、Chem、SpeakerVid、TalkVid、Kling-Foley）可自行获取。
【评测基准】承诺开源但尚未释出。论文称将发布 V2ST-Bench 的 annotations、metadata 与 processing scripts；因版权限制不直接分发原始视频文件，改为提供 URL + metadata（类似 VGGSound/HowTo100M 的做法）。
【pipeline】方法论层面披露较充分：Table 7 给出六项过滤指标的具体阈值数值，Table 12 给出 Gemini 2.5 Pro 标注 prompt 模板，声学后验证给出 -35 dB 显式阈值公式。但未发布完整清洗脚本，也未给出逐级保留率表。

### [Goku](../models/Goku.md)

整体属于「论文公开、权重与数据封闭」类型。
【已公开】(1) 技术论文完整披露数据流水线各级阈值、模型架构（Goku-2B：28层/dim 1792/28头；Goku-8B：40层/dim 3072/48头）、rectified flow 训练配方与并行训练基础设施；(2) GitHub 仓库 Saiyan-World/goku 提供了 configs/、goku/、tools/ 等代码目录骨架与 requirements.txt；(3) 项目主页提供大量生成样例可视化。
【未开源】模型权重（HuggingFace 上无官方 Goku 权重发布）、训练数据、数据处理流水线代码、内部视频分类模型、内部美学评分模型。仓库亦未见 LICENSE 文件。
【结论】权重：否；代码：部分/不完整；数据：否；pipeline：仅论文文字描述，代码未开源。
注：社区中大量以「Goku AI」命名的第三方网站/产品为蹭名，非官方发布。

### [Hailuo / MiniMax Video](../models/Hailuo.md)

完全闭源，是本次调研中数据披露最少的对象之一：
- 模型权重：未开源，仅通过海螺AI网页/App、MiniMax 开放平台 API，以及 Replicate、fal.ai 等第三方托管平台提供推理服务；
- 训练/推理代码：未开源；
- 训练数据：未开源、未描述；
- 数据处理 pipeline：未开源、未描述；
- 技术报告/论文：完全没有。历代模型均只有产品发布博客（minimax.io/news），内容以能力演示与定价为主，几乎不含技术细节；NCR 架构也只有一句名词性说明，无论文支撑。
对比之下 MiniMax 在语言模型侧开源程度很高（MiniMax-01、M1、M2/M2.1/M2.5/M2.7、M3 均在 HuggingFace 发布权重与技术报告），视频侧则是刻意的完全封闭。唯一与视觉生成相关的开源物是 2025年12月15日发布的 VTP（Visual Tokenizer Pre-training）系列视觉 tokenizer（Small 0.2B / Base 0.3B / Large 0.7B，Modified MIT 许可，arXiv:2512.13687），但其定位是图像 tokenizer 基础组件，模型卡未说明其与 Hailuo 视频模型的直接关系，其训练数据构成同样未在模型卡中明确列出。

### [HunyuanVideo-Foley](../models/HunyuanVideo-Foley.md)

开源程度中等，属于「权重与推理开放、训练与数据封闭」的典型形态。
【权重】开源。HuggingFace tencent/HunyuanVideo-Foley 提供两个变体：XXL（默认，hunyuanvideo_foley.pth，常规推理约需 20GB 显存，offload 模式 12GB）与 XL（hunyuanvideo_foley_xl.pth，16GB / offload 8GB）。同时开放自研的 48kHz DAC-VAE 音频自编码器权重与 Synchformer 同步特征提取器配置。
【代码】推理代码开源（infer.py 批量推理、gradio_app.py 网页交互界面、hunyuanvideo-foley-xxl.yaml 配置文件），基于 HuggingFace diffusers。训练代码未开源。
【数据处理 pipeline】仅在论文第 3.1 节以约一段文字的篇幅做流程性描述——七个环节的顺序与部分阈值（8 秒切分、静音占比 80%、有效采样率 32 kHz）有明确数值，但未公开任何清洗脚本、模型配置、prompt 或逐级统计。可复现性属「知道做了什么、不知道怎么调参」的层次。
【数据】10 万小时 TV2A 数据集完全不开源，原始来源亦未披露。
【许可证】tencent-hunyuan-community 社区许可（非标准 OSI 开源协议），HuggingFace 模型卡设置 extra_gated_eu_disallowed: true，即明确限制欧盟地区用户获取。这是腾讯混元系列模型的一贯做法，商用需遵守社区许可条款。

### [HunyuanVideo](../models/HunyuanVideo.md)

属于「权重+代码开源、数据与数据pipeline不开源」的典型模式。
【权重】开源。HunyuanVideo 13B（DiT主干+3D VAE+文本编码器）在 Hugging Face（tencent/HunyuanVideo、tencent/HunyuanVideo-I2V）发布；HunyuanVideo 1.5（8.3B，含T2V/I2V/超分模块）在 GitHub Tencent-Hunyuan/HunyuanVideo-1.5 与 Hugging Face 发布。
【代码】开源。推理代码、并行推理、量化、LoRA、ComfyUI/Diffusers 集成均提供；训练代码未完整开放。
【许可证】腾讯混元社区许可协议（Tencent Hunyuan Community License），非标准OSI开源协议，对欧盟等地区与月活用户数有使用限制。
【数据】不开源。训练数据集本身、各级过滤后的清单、caption数据均未公开。
【pipeline】方法论层面披露详尽（尤其原版对分层过滤漏斗、结构化caption schema、镜头运动分类器的描述在同期闭源模型中属最详细一档），但过滤器代码、模型权重（如自研VideoCLIP、blur检测模型、YOLOX-like检测器、caption VLM）均未开源，无法直接复现。

### [InstructAV2AV](../models/InstructAV2AV.md) ⚠️

开源程度较高，是本调研涉及的音视频工作中数据开放度最好的之一。
【权重】开源。HuggingFace suimu/InstructAV2AV 发布预训练权重，并提供六个面向不同子任务微调的 checkpoint（覆盖通用编辑、内容增删、身份/音色克隆等场景）。[不确定] 该模型仓库页面未见完整 model card，参数量、许可证等信息未在模型页标注。
【代码】部分开源。GitHub 仓库以 Apache-2.0 许可发布推理代码（scripts/edit.py、scripts/demo.py）与pipeline脚本；训练脚本在 README roadmap 中列为「进行中」，尚未释出。依赖上游 Wan-AI/Wan2.2-TI2V-5B 与 hkchengrex/MMAudio。
【数据】完整开源，这是本工作最突出的开放点。HuggingFace suimu/InsAVE-80K 以 MIT 许可发布，实际打包 88,074 对视频（176,148 个文件），分 11 个 tar 分片，约 139 GB，直接分发视频与音频文件本体而非仅 URL——这在音视频数据集中较为少见（对比 Foley-Omni 的 V2ST-Bench 因版权限制仅发 URL+metadata）。数据卡同时提示使用者需自行核验底层媒体内容的权利合规性。
【pipeline】方法论层面披露充分：完整列出了每一环节所用的具体模型（PySceneDetect、CoTracker3、LAION Aesthetics Predictor、PyDub、Audiobox-Aesthetics、TalkNet、ElevenLabs Scribe、Grounded-SAM-2、Qwen3-Omni、SAM-Audio、ElevenLabs TTS、Wan2.2-5B），但绝大多数过滤阈值的具体数值未给出（仅 -45 dBFS 一项有明确数字），也无逐级保留率，因此可复现性受限。

### [2026 其他音视频联合生成](../models/JAVG_2026_misc.md) ⚠️

开放程度差异极大，从完全开源到仅有技术报告：
【最开放｜NAVA】Apache 2.0 源码许可。已发布：完整推理管线、训练代码（training code）、Gradio 交互 demo、模型权重 NAVA.safetensors（24GB bf16）与量化版 NAVA_fp8.safetensors（约 7GB）。依赖组件复用 Wan2.2-5B VAE、T5 编码器、LTX audio-VAE、ReDimNet 说话人嵌入器（各自遵循原许可）。未开源：训练数据本身与数据处理 pipeline 代码（仓库明确提到训练数据但无数据管线释出）。
【中等｜OmniCustom】提供 GitHub 仓库与项目主页，代码可得；其自建 OmniCustom-1M 派生自公开数据集 SpeakerVid-5M，因此数据构建过程可复现（筛选规则全部公开），但过滤后清单是否释出未明[不确定]。
【中等｜StreamChar】有项目主页 humanaigc.github.io/StreamChar_page/，训练数据全部来自三个公开数据集（SpeakerVid-5M / TalkVid / OpenHumanVid）+ Emilia，因此数据侧可复现度高；权重与代码释出情况未明[不确定]。
【偏封闭｜ALIVE】GitHub 仓库 FoundationVision/Alive 目前仅为技术报告发布页（含 assets 目录、arXiv 链接、项目页、Discord demo 入口），未见权重下载、训练代码、推理代码或数据释出；Alive-Bench 1.0（264 general prompts + 90 reference-character prompts）已在论文中定义但公开释出状态未明[不确定]。是七项中数据披露最详细、代码开放度最低的一项。
【封闭｜Baton / CCL】仅论文；训练数据含 in-house / 互联网自采部分，未开源；CCL 的 in-house 数据（访谈、短剧、电影）明确不可得。
【N/A｜ITS-JAVG】training-free，无权重需开源；论文称「Project materials and code are available online」，代码与 prompt 集应可得[不确定：具体仓库地址未验证]。所依赖的验证器（VideoReward、JavisScore、ImageBind、VQAScore、AVHScore）与基座生成器（JavisDiT、MMDisCo、LTX-2）均为已开源资产，可复现性最高。

### [音视频联合生成基线合集](../models/JavisDiT_baselines.md) ⚠️

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

### [可灵 3.0 / 可灵视频 3.0 Omni](../models/Kling_30_Omni.md) ⚠️

权重：不开源；代码：不开源；训练数据：不公开；数据处理 pipeline：仅在技术报告中做定性描述，未开源。属于典型的闭源商业API模型。[不确定：是否有任何组件后续开源]。可参考的同团队开源资产为间接旁证：Koala-36M 视频数据集及其清洗/打标 pipeline（github.com/KwaiVGI/Koala-36M，arXiv:2410.08260）、Kling-Foley 视频到音频模型与 Kling-Audio-Eval 评测集（arXiv:2506.19774）均由快手可灵团队开源，可视为可灵系列数据处理方法论的公开切片，但不等同于3.0 Omni的实际训练数据。

### [LTX-2](../models/LTX-2.md)

开源程度在同类音视频联合生成模型中最高，但数据侧仍封闭。
【已开源】(1) 全部模型权重：ltx-2-19b-dev（bf16 可训练）、fp8/fp4 量化版、8步蒸馏版 ltx-2-19b-distilled，以及空间/时间上采样器，均在 Hugging Face 发布（Lightricks/LTX-2，月下载量约42.7万）；(2) 推理代码与多后端集成；(3) 训练/微调代码 ltx-trainer（支持 LoRA、全参微调、IC-LoRA）；(4) 技术报告公开架构细节。
【未开源】训练数据本身、数据处理 pipeline 代码、内部 captioner 模型、美学评分模型、数据统计数字。
【许可】LTX-2 Open Weights License：学术研究免费，年经常性收入（ARR）低于1000万美元的公司商用免费；超过该阈值需向 Lightricks 取得商业许可。官方自称「首个真正开放权重的生产级音视频生成模型」。

### [LongCat-Video](../models/LongCat-Video.md)

权重与代码开源程度高，数据侧完全封闭。
【已开源】(1) 模型权重：Hugging Face meituan-longcat/LongCat-Video（13.6B），以及后续 LongCat-Video-Avatar（wav2vec2 音频编码器）、LongCat-Video-Avatar-1.5（Whisper-large-v3 音频编码器）；(2) 推理代码：GitHub meituan-longcat/LongCat-Video，含多种推理模式与 Streamlit 交互界面；(3) Block Sparse Attention 算子的 forward 与 backward 实现随基座模型一并开源；(4) 技术报告公开架构、训练阶段表与 RLHF 细节。
【许可】MIT License（权重与代码），可商用，是同规模视频生成模型中最宽松的许可之一。
【未开源】训练数据本身、数据处理 pipeline 代码、内部 captioner（微调版 LLaVA-Video）、内部美学/模糊/水印打分器、内部微调的 VideoAlign 奖励模型、以及全部数据规模统计数字。训练代码亦未开源（仅推理）。

### [MOVA](../models/MOVA.md)

开源程度在同类音视频联合生成模型中属于最高一档，采用 Apache-2.0 许可证，允许无限制商用。
【权重】开源。发布 MOVA-360p 与 MOVA-720p 两个变体（HuggingFace: OpenMOSS-Team/MOVA-360p、OpenMOSS-Team/MOVA-720p，以及 mova collection）。
【代码】开源且覆盖全链路：训练 pipeline、高效推理、LoRA 微调脚本、prompt 增强（rewriter）工作流、评测代码。论文明确声明“we release all model weights along with training, inference, and fine-tuning code”。
【数据处理 pipeline】方法论层面在论文第 3 节与附录 A.3/A.4/A.5 中做了工业级细节披露——包含完整三阶段漏斗结构、逐指标过滤阈值表（Table 9）、逐级保留率表（Table 1）、语音窗口切分伪代码（Algorithm 1/2）、以及全部打标 prompt 原文与完整 caption 示例。这是本次调研中数据处理方法可复现性最高的样本之一。但代码库中并未单独发布数据清洗脚本，仅提供 mova/datasets/video_audio_dataset.py 数据集接口，用户需自备视频音频数据并按配置接入。
【数据】训练数据本身不开源。公开数据集部分（VGGSound、AutoReCap、ChronoMagic-Pro、ACAV-100M、OpenHumanVid、SpeakerVid-5M、OpenVid-1M）可自行获取，但其“过滤后 HQ 子集”的具体清单未发布；in-house 数据（中文剧集、动画、电影、YouTube 抓取等）未公开。

### [合并条目：① Mochi 1](../models/Mochi_MAGI_Motif.md)

① Mochi 1（部分开放，数据侧完全封闭）：权重与推理代码 Apache 2.0 开源（GitHub genmoai/mochi、HF genmo/mochi-1-preview），AsymmVAE 一并开源，允许商用。但无技术报告/论文，无任何数据 pipeline 披露，训练数据、清洗流程、打标方法均未公开，官方对数据来源明确采取「因竞争原因不透露」的态度。可视为「开权重、闭方法、闭数据」。
② MAGI-1（高度开放，数据方法开放但数值封闭）：Apache 2.0 开源 24B / 4.5B 基座与蒸馏权重、FP8 量化版、推理代码，另开源 MagiAttention 独立库；61 页技术报告完整披露了 Section 3 DATA 的全部方法论（切分工具、11 类过滤 actor、去重双模型、MLLM 二次过滤、两类 caption schema、三阶段数据配置表）。但所有过滤阈值均以「predefined threshold / lower and upper thresholds」表述而不给数值，数据集规模仅给「tens of petabytes 原始素材」这一量级词，清洗代码与数据本体不开放。可视为「开权重、开方法、闭参数、闭数据」。
③ Motif-Video 2B（三者中数据披露最彻底）：权重 Apache 2.0 开源，技术报告把数据管线写成可复现的工程文档——具名工具（NeMo Curator、ffmpeg cropdetect、PaddleOCR-VL、SigLIP、Aesthetic Predictor V2.5、DOVER、UniMatch、SSCD sscd_disc_mixup、NVIDIA cuVS、Qwen3-VL-30B-A3B）、具名超参（SSCD 512 维描述子 / 320×320 / 第 10 帧 / cosine≥0.9 / k=64 / nprobe=16、OCR 聚类需出现于 ≥50% 帧、上下各 20% 区域排除、caption 采样概率 0.5/0.3/0.2、SA 优化 30000 次迭代、rolling shuffle 窗口 4096）、以及完整 10 阶段课程表。但数据集本体与清洗代码同样不开源，Sankey 图（Fig.7）只给流向不给绝对数值。可视为「开权重、开方法、开参数、闭数据」。

### [Movie Gen](../models/Movie_Gen.md)

半开放：模型权重未开源、训练/推理代码未开源、训练数据未公开。但技术报告（92页）极其详细地公开了数据清洗 pipeline、各级过滤阈值与漏斗保留率、打标方案与训练配方，属于行业最详尽的公开数据工程文档之一。开源部分为评测资产：Movie Gen Video Bench（1003条prompt）与 Movie Gen Audio Bench，以及在这两个benchmark上非精选（non cherry-picked）的生成结果视频/音频，托管于 https://github.com/facebookresearch/MovieGenBench 。论文明确说明模型仅用于研究目的，部署前仍需多项改进。

### [NVIDIA NeMo Curator](../models/NeMo_Curator.md)

开源程度在同类中最高，属于「pipeline 完全开源、数据完全不含」的类型。
【代码】NeMo Curator 全部源码 Apache License 2.0，GitHub 公开（NVIDIA-NeMo/Curator），PyPI 包名 nemo-curator，官方推荐 Docker 容器安装（视频/音频 workflow 需要预配置的 FFmpeg 8.0.1 + NVENC）。
【底层引擎】Cosmos-Xenna 同为 Apache 2.0 独立开源，可脱离 Curator 单独使用。
【产品化实现】Cosmos-Curate 源码 Apache 2.0；其调用的模型权重走 NVIDIA Open Model License，可申请自定义商业许可。
【pipeline 配置】26.02 起支持 YAML 声明式定义整条 curation pipeline，进一步降低复现门槛。
【模型权重】自身不训练模型，调用的判别/标注模型多为第三方开源权重：Qwen2.5-VL / Qwen3-VL（captioning）、Cosmos-Embed1 与 InternVideo2（视频嵌入，InternVideo2 在 26.02 被移除）、CLIP-based aesthetic model（美学打分）、TransNetV2（镜头切分）、NVIDIA NeMo ASR 系列（音频转写）、Nemotron Nano 12B V2 VLM 与 Nemotron 3 Nano Omni（26.04/26.07 新增的 captioning 后端，含 BF16/FP8/NVFP4 三种精度变体）。
【不开源】训练数据本身（框架本身不附带任何数据集）；NVIDIA 内部用于 Cosmos 的实际数据源清单与全量统计不公开。

### [OmniHuman 数据集 + OHBench](../models/OmniHuman.md) ⚠️

开放程度在同类人物中心数据集中属于较高档位，但存在「视频本体不分发」的固有限制：
【代码】开源。GitHub 仓库 github.com/julia-cherry/OmniHuman 提供 OHBench 评测工具包，覆盖成像质量、动态程度、身份一致性、音视对齐、音频质量、语音质量、双人交互等各类指标的评估脚本、配置文件与 Python 依赖说明。
【数据（标注）】开源。HuggingFace 数据集 julia527/omnihuman，总计约 62.7 GB 的分片 tar 归档，包含三类资产：sample_json（逐样本的音视觉标注：captions、subjects、speech、audio）、metadata（JSONL 索引文件，用于扫描与加载样本）、tracking_npz（逐帧 SMPL/MANO 身体与手部跟踪数据）。
【数据（视频本体）】不分发。原始视频不包含在发布内容中，每条样本只给出 YouTube 源 URL 加上精确的片段起止时间（clip_start_sec / clip_end_sec），需使用者自行下载定位——这是规避版权风险的常规做法（与 HD-VILA、Panda-70M 同路线），但也意味着随着源视频下架，数据集会持续损耗（link rot），且不同使用者拿到的实际数据会逐渐不一致。
【评测资产与权重】HuggingFace 仓库 julia527/omnihuman_benchmark 提供 OHBench 的模型检查点与评测资产，多为 .pt / .onnx 格式（即 MUSIQ、RAFT、ArcFace、SyncNet、DNSMOS 等第三方判别模型的推理权重打包）。
【pipeline】方法层面公开（四级过滤的顺序、每一级所用工具名、跟踪与归属模块的算法链路、两阶段 caption 的结构均有描述），但阈值数值大面积缺失（详见 pipeline_overview 与 funnel_retention_rate），清洗脚本本身未见发布。可复现性介于「知道做了什么」与「能照做」之间。
【许可】arXiv 论文采用 perpetual non-exclusive license；HuggingFace 数据集页与 GitHub 仓库均未见明确的许可声明或使用条款，数据的下游可用范围（是否允许商用、是否限研究用途）不明。[不确定]

### [Open-Sora 系列](../models/Open-Sora.md) ⚠️

两者都属于视频生成领域开源度最高的一档，但仍非「数据集全开源」。
【Open-Sora（HPC-AI Tech）】权重：开源（HuggingFace hpcai-tech，Apache 2.0）；代码：开源（推理+训练+分布式优化）；数据处理 pipeline：完整开源，这是该项目最具参考价值之处——v1.1/v1.2 分支下 tools/ 目录包含 scene_cut（PySceneDetect 切镜+切分）、scoring（aesthetic / optical_flow / ocr / matching 四个打分器）、caption（PLLaVA、LLaVA、LLaMA3 打标）、datasets（datautil 过滤清洗），并有 docs/data_processing.md 串联全流程，每一步都给出可直接运行的 torchrun 命令与阈值示例；训练数据本身：不开源（仅公布来源数据集名称与筛选阈值，未发布筛选后的 meta 文件）。需注意：2.0 版 main 分支已移除 tools/ 数据处理目录（tools/datasets、tools/scoring、docs/data_processing.md 在 main 分支返回 404），完整数据处理代码需回到 v1.2.0 等历史 tag 获取；2.0 技术报告描述的新 pipeline（PaddleOCR、VMAF、Laplacian 等）在 main 分支中并未见对应开源实现，此处开源度实际上是**退步**的。
【Open-Sora Plan（北大）】权重：开源；代码：开源（训练+推理+WFVAE）；数据：部分开源——各版本在 HuggingFace（LanguageBind/Open-Sora-Plan-v1.1.0 / v1.2.0 / v1.3.0）发布了「Data and Annotations」标注数据与 prompt_refiner 数据集；数据处理 pipeline：论文与 Report 文档中给出了完整的过滤步骤、工具、阈值与逐级保留率（这是全行业罕见的定量披露），但未见独立打包的 curation 代码库，复现需按文档自行拼装。[不确定]

### [Ovi](../models/Ovi.md)

属于「权重+推理代码开源、数据与数据 pipeline 代码闭源」的典型模式。
【已开源】(1) 模型权重：11B（HuggingFace 页面标注约 12B 参数、BF16）完整 checkpoint，含三个版本 720x720_5s、960x960_5s、960x960_10s，托管于 HuggingFace chetwinlow1/Ovi；(2) 推理代码：文本/文本+图像输入、Gradio App、多 GPU（含序列并行）推理、权重下载脚本；(3) 社区贡献的 fp8（@rkfg）与 qint8（@gluttony-10）量化权重，24GB 显存可跑；(4) 论文完整披露了数据处理 pipeline 的方法与关键阈值（这一点比多数同类模型开放）。
【未开源】训练脚本（README 的 Todo List 中「Training scripts」仍未勾选）、训练数据本身（内部音视频语料与内部音频语料）、数据处理 pipeline 代码、所用 MLLM 打标模型的身份、各级过滤的定量保留率、RL 后训练细节。
【许可】Apache 2.0（相对宽松，可商用）。依赖组件：视频分支权重来自 Wan2.2-TI2V-5B，文本编码器 T5 与视频 VAE 解码器来自 Wan，音频 VAE 来自 MMAudio（均为开源模型）。

### [Script-a-Video](../models/Script-a-Video.md)

开源程度极低，属于「纯论文披露、零代码零权重零数据」类型：
【权重】未开源。Qwen3-Omni-MTSS-FT 与基于 LTX-2 改造的生成模型均未发布。
【代码】未开源。论文全文无 GitHub 链接、无项目主页 URL、无开源承诺声明。
【数据】未开源。500K MTSS 标注数据集明确表述为「internal dataset」（内部数据集）；生成侧的 400K identity-centric / 250K multi-shot / 870K cinematic pairs / 60K cinematic alignment pairs 四套数据同样为内部数据；125+100 条内部评测集也未发布。
【pipeline】方法论层面对 MTSS schema 的字段定义披露非常充分（Reference/Shot/Event/Global 四流的全部字段名与语义在正文第 3 节逐一说明，并配有 Figure 3 完整脚本示例），可复现性主要体现在「schema 可被他人重新实现」这一层；但数据清洗流程、Gemini-2.5-Pro 标注 prompt 原文、过滤阈值均未披露。
【许可】arXiv.org perpetual non-exclusive license（仅论文文本），无模型/代码许可。
【整体判断】其价值在于范式与 schema 设计本身可被直接借鉴复刻，而非在于可直接取用的资产。

### [Seedance 2.0 与 Seedance 1.5 pro](../models/Seedance_20_Seedance_15_pro.md)

权重：不开源；代码：不开源；训练数据：不公开；数据处理 pipeline：仅 Seedance 1.0 报告给出较完整的文字描述（无代码、无阈值、无开源工具），1.5 pro 仅一段要点式描述，2.0 报告完全未披露数据章节。对外仅提供 API/产品访问（火山方舟 Ark、豆包 Doubao、即梦 Jimeng，以及 Replicate 等第三方托管）。

### [SkyReels 系列](../models/SkyReels.md) ⚠️

呈现「早期版本全开源、最新旗舰闭源产品化」的分层策略。
【SkyReels-V1/V2/V3：权重+代码开源】V2 在 Hugging Face 与 ModelScope 发布 1.3B/5B/14B 全系列权重（DF 长视频、T2V、I2V、540P/720P），配套开源推理代码与 SkyCaptioner-V1 打标模型权重；V3（R2V/V2V/Talking Avatar，14B）2026年1月开源。SkyReels 系列开源模型在 HuggingFace 累计下载近30万、GitHub star 超1万。
【SkyReels-V4：截至2026年7月未见权重开源】论文未给出任何代码/权重发布承诺，也未附 GitHub 链接；对外以 skyreels.ai 产品（限量预览+免费额度）与开放 API 形式提供，覆盖文生视频、图生视频、多模态参考生成、视频编辑修复、音视频联合生成。部分中文媒体口径称「V1–V4 均已开源」，但 HuggingFace/GitHub 上未检索到 SkyReels-V4 权重仓库，应以「暂未开源权重」理解。[不确定]
【数据侧】三个版本的训练数据本体、数据清洗 pipeline 代码、内部质检模型均未开源；仅 SkyCaptioner-V1 这一打标环节的模型权重开源，这是该系列在数据侧最有价值的开放物。

### [Sora 2](../models/Sora_2.md)

全封闭。权重不开源、代码不开源、训练数据不开源、数据处理pipeline不开源。唯一公开材料为2025年9月30日的《Sora 2 System Card》（共7页），其中关于数据的内容仅第2节「Model Data & Data Filtering」一个自然段（约5句话）。没有技术报告、没有论文、没有架构细节、没有任何数据统计数字。相比之下前代 Sora 1 至少有一篇技术博客《Video generation models as world simulators》披露了时空patch、原生分辨率训练、重打标等方法论。曾以API形式（sora-2、sora-2-pro）商业开放，2026年9月API也已停用。

### [SpeakerVid-5M](../models/SpeakerVid-5M.md) ⚠️

属于「元数据+标注全开、原始视频不托管」的典型学术数据集开放模式：
【数据（标注与索引）】开源。HuggingFace dorni/SpeakerVid-5M-Dataset 提供 all_data_list.json（YouTube 视频 ID + 切分后 clip 名称，clip 名即定位标注的唯一键）与 SFT_set.json（高质量子集清单），以及五类标注文件夹：merge_anno（clip 级元数据：时间戳、空间 bbox、clear/DOVER 质量分）、asr（Whisper 转写与置信度）、l_score（人脸/手部清晰度即模糊分）、anno（Qwen-VL 生成的 MLLM 结构化 caption）、dwpose（因体积过大未上传实际骨架序列，仅提供计算代码）。
【数据（原始视频）】不托管。需用户依据 YouTube video ID 自行用 yt-dlp 下载，存在链接失效（link rot）导致的可复现性衰减风险。
【数据处理 pipeline 代码】开源，这是本条目最有价值的部分之一。GitHub Dorniwang/SpeakerVid-5M-Code 发布了六段式完整清洗流程代码：base annotation（音视频同步抽取 + 单人检测）、DWpose 骨架标注、ASR 标注、blur score 计算、luminance 计算、scene detection + speaker diarization（部分预计算，可选）。
【基线模型权重】论文与代码库均未见权重开源的明确说明。[不确定]
【许可】明确限定为「non-commercial, scientific research, and educational purposes only」（仅限非商业的科研与教育用途），显式禁止商业使用；内容源自公开互联网，版权归原创作者所有，并提供 takedown 政策供版权方申请下架。未采用 Apache/CC 等标准 SPDX 许可证。

### [Step-Video-T2V](../models/Step-Video-T2V.md)

属于「权重+代码+评测基准开源，数据与数据 pipeline 代码不开源」模式，但开源程度在同期国产模型中较高：
【权重】开源。Step-Video-T2V（30B）与蒸馏加速版 Step-Video-T2V-Turbo 权重均在 GitHub（stepfun-ai/Step-Video-T2V）、Hugging Face（stepfun-ai/stepvideo-t2v、stepvideo-t2v-turbo）与 ModelScope 发布；自研 Video-VAE（16×16 空间、8× 时间压缩）与双语文本编码器一并开放。
【代码】开源推理代码（含多 GPU 并行推理、xDiT/ComfyUI 集成），训练代码未开放。
【许可证】MIT License（相较腾讯混元社区协议、通义万相等自定义协议更宽松，是其开源度的一大亮点）。
【评测基准】开源 Step-Video-T2V-Eval，含 128 条 prompt 及多个开闭源引擎的生成结果视频，可直接复现对比。
【数据】不开源。2B 视频-文本对与 3.8B 图文对均为内部数据，未发布任何子集或清单。
【pipeline】方法论披露较完整（六阶段流水线、各阶段所用的具体开源工具与模型名称几乎全部点名，如 PySceneDetect AdaptiveDetector、LAION 美学预测器、LAION NSFW 检测器、EfficientNet 水印分类器、PaddleOCR、Laplacian 方差、FFmpeg cropdetect、Farneback 光流、内部 VideoCLIP），但过滤阈值数值、自研 caption VLM 与 VideoCLIP 权重、pipeline 代码均未公开，不可直接复现。

### [UniTalking](../models/UniTalking.md) ⚠️

论文自我定位为「开源框架」（open and reproducible framework），并以「闭源模型阻碍学术进步」作为核心动机，但截至2026年7月实际开放程度存疑：
【权重】未检索到 HuggingFace 或其他平台的权重发布。[不确定]
【代码】未检索到 GitHub 仓库。论文正文与摘要页均未给出代码链接或项目主页 URL，这与其「开源」叙事之间存在落差。[不确定]
【数据】不开源。230万对齐样本中，OpenHumanVid 部分可由第三方按其原始许可（需提交用户信息审核后授权下载）自行获取，华为内部采集部分与内部 TTS 数据完全不公开。
【pipeline】方法论层面仅给出三级过滤的结构与所用工具名（PANNs、SentenceASD、LightASD、LipSync），未给出任何阈值数值、逐级保留率、prompt 原文或清洗脚本，可复现性显著低于同类工作（对比 UniVerse-1 至少公开了 1080p / 码率比 600 / DOVER 0.6 / SyncNet 2.0 等全部阈值）。
【许可】arXiv 采用 perpetual non-exclusive license，模型与代码许可未声明。[不确定]

### [UniVerse-1](../models/UniVerse-1.md)

开源程度中等偏上，采用 Apache-2.0 许可证。
【权重】开源。HuggingFace 上发布 dorni/UniVerse-1-Base（7B 参数，F32/safetensors，bfloat16 推理），仅此一个变体，无多分辨率版本。
【代码】部分开源。GitHub Dorniwang/UniVerse-1-code 发布推理代码（基于 diffusers，Python≥3.10、PyTorch≥2.5.0-cu121），训练代码与在线标注 pipeline 的服务端实现未开源。
【评测基准】开源。Verse-Bench 数据集与评测指标工具均在 HuggingFace 发布。
【数据处理 pipeline】方法论层面在论文第 3 节做了逐级阈值披露（分辨率、码率比、DOVER、片段时长、SyncNet 阈值均给出具体数值），可复现性尚可；但未发布任何清洗脚本或标注 prompt 原文，披露粒度明显低于 MOVA（无逐级保留率表、无 prompt 全文、无伪代码）。
【数据】训练数据不开源。公开数据集部分（VGGSound、AudioSet）可自取，自采的 YouTube/Pexels/电影片段部分未公开，过滤后子集清单亦未发布。

### [Unison](../models/Unison.md)

截至 2026 年 7 月为完全未开源状态，仅论文本身开放（CC BY 4.0）。
【权重】未开源。论文原文：「The code and models will be made publicly available upon acceptance」——附条件承诺，以会议接收为前提，尚未兑现。
【代码】未开源。未见 GitHub 仓库，未见项目主页或 demo 页面。
【数据】不开源，且实际上无「可开源的数据集」——训练语料是对 OpenHumanVid、HDTF、VFHQ、CelebV-Text、VGGSound 等既有开源数据集的重新筛选与再加工，筛选后的样本清单（约 200 万条）未发布。音频侧另含未公开的「内部语音数据」（internal speech data）。
【pipeline】披露度极低，是本条目最大的信息短板。论文对「自动化处理 pipeline」（automated processing pipeline）仅有一句话交代，唯一展开描述的环节是 lip-filtering 算子（人脸检测 + 框内 SyncNet 核验），且未给出任何阈值数值、逐级保留率或伪代码。与 MOVA（逐级阈值 + 保留率表）、UniVerse-1（六级漏斗全阈值公开）相比，Unison 在数据处理可复现性上明显落后一个层级——这是一篇「方法创新论文」而非「数据工程论文」的典型特征。
【评测集】1,000 条测试样本及 Gemini 生成的 ground-truth 标注均未发布，用户研究（40 样本 × 25 人）的原始数据亦未发布。
【复用的开源资产】所有基座与工具链均为开源：Wan2.2-5B、MMAudio、Zipformer、Mel-RoFormer、SyncNet、Whisper-large-v3、Synchformer、ImageBind、CLAP、LAION-Aesthetic V2.5、DINOv3、Audiobox-Aesthetics。

### [Veo 3 / Veo 3.1](../models/Veo_3_Veo_31.md)

完全闭源。权重不开源、代码不开源、训练数据不公开、数据处理 pipeline 不公开、数据配比与规模不披露。仅公开一份约 7 页的技术报告（Veo-3-Tech-Report.pdf）和一份约 6 页的 Model Card，其中与数据相关的正文不足 200 词，绝大部分篇幅是责任与安全评测。仅通过付费 API / 产品形态开放推理调用（Veo 3、Veo 3 Fast、Veo 3.1、Veo 3.1 Fast、Veo 3.1 Lite 等变体）。

### [Vidu S1](../models/Vidu_S1.md)

闭源。技术报告仅公开架构与训练框架思路，未开源模型权重、推理/训练代码、数据、数据处理 pipeline；自研加速与服务组件 TurboDiffusion、TurboServe 亦未开源。自建基准 Vidu-StreamBench 未公开发布。仅通过官网 https://vidu.com/vidu-stream 提供可交互在线 demo。论文以 CC-BY 4.0 许可发布。

### [Wan 2.5 / 2.6 / 2.7](../models/Wan.md)

呈现「前代全开、当代全闭」的断层式格局，且数据侧自始至终封闭。
【Wan 2.5 / 2.6 / 2.7】完全闭源：无权重、无代码、无技术报告、无论文。仅通过阿里云百炼 DashScope API（北京/新加坡/弗吉尼亚多地域）与万相官网/通义 App 提供服务，按分辨率档位与秒数计费。截至2026年7月，Hugging Face 的 Wan-AI 组织与 GitHub 的 Wan-Video 组织中均不存在 Wan2.5/2.6/2.7 任何仓库或权重——GitHub 仅有 Wan2.1、Wan2.2、Wan-Dancer、Wan-skills、diffusers 分支5个仓库。
【Wan 2.1】权重+推理代码开源（Apache 2.0），并发布60页技术报告含详细数据章节；训练数据、清洗 pipeline 代码、内部 caption 模型、各类专家分类器均未开源。
【Wan 2.2】权重+推理/微调代码开源（Apache 2.0），README 披露相对 2.1 的数据增量比例与「电影美学标签体系」，但无独立技术报告，数据细节远少于 2.1。
【Wan2.2-S2V / Animate / Dancer】权重+代码开源，论文含数据处理章节（S2V 的第2章是 Wan 系唯一明确写出音画同步数据过滤方法的文档）。
【结论】要研究 Wan 2.5+ 的数据方法，只能以 Wan 2.1 报告为主干、Wan 2.2 README 与 S2V/Dancer 论文为增量旁证，其余靠 API 行为反推。

### [音视频生成评测基准合集](../models/av_benchmarks.md) ⚠️

【VABench】论文 CC BY 4.0；代码仓库 https://github.com/tanABCC/VABench；未明确声明数据集许可证。提示词、VQA/AQA 问答对与评测脚本为主要开源产出，生成视频依赖各家 API 自行复现。
【AVBench】开源程度最高：GitHub https://github.com/YaJialiang/AVBench，评测器权重发布于 HuggingFace（iiiiii123/AVBench_model），并托管 HuggingFace Leaderboard（spaces/iiiiii123/AVBenchLB）。数据、代码、模型三者均释出；具体许可证未标注[不确定]。
【AV-SyncBench】数据集已上线 ModelScope（coming245/AVSyncBench）与 HuggingFace（coming245/AV-SyncBench），代码仓库 https://github.com/fgt7t6g/AV-SyncBench（截至调研时 README 标注评测代码 coming soon）；论文采用 arXiv perpetual non-exclusive license。
【PhyAVBench】项目页 https://imxtx.github.io/PhyAVBench/ 与 https://phyavbench.pages.dev/；公开释出提示词、自录 ground-truth 视频与各模型生成样本，并承诺与训练集零重叠；论文 arXiv 许可证。
【Omni-Judge】仅有项目页 liangsusan-git.github.io/project/omni_judge/，论文未明确声明代码/数据开源[不确定]。

### [视频 Caption 模型生态](../models/caption_models.md) ⚠️

开源度呈明显的「学术全开 / 大厂半开 / 生成侧闭源」三档分化：
【全开（权重+代码+数据）】ShareGPT4Video：40K GPT-4V 密集 caption 与 4.8M ShareCaptioner-Video 标注全部公开，论文 CC BY 4.0，ShareCaptioner-Video 权重在 HF（Lin-Chen/ShareCaptioner-Video，基座 InternLM-XComposer2-4KHD）。Tarsier 系列：bytedance/tarsier 代码 + omni-research/Tarsier2-Recap-7b 权重 + Tarsier2-Recap-585K 数据全开，是目前被下游生成模型复用最多的开源 captioner。CogVLM2-Caption：权重开源（zai-org/cogvlm2-llama3-caption），是 CogVideoX 数据 pipeline 中唯一可复现的一环。SkyCaptioner-V1：权重 + 训练细节（Qwen2.5-VL-7B 基座、32×A800、200 万条概念均衡数据）全披露。AVoCaDO：Apache-2.0 权重 + 代码 + 项目页，但 AVoCaDO-SFT-107K 未单独发布。video-SALMONN 2：Apache-2.0，代码/权重/testset 全开。AuroraCap + VDC 基准开源。
【半开】AVSCap：代码 + AVSCapBench 已开放，AVSCap-130K 训练集 README 明示「will release as soon as possible」，截至调研时未发布；AVSCap-7B 权重可用性存疑（GitHub 同时出现 HF 链接与「待发布」表述）[不确定]。Qwen3-Omni-Captioner（音频版）开源，但 Qwen3.5-Omni 的音视频 caption 能力仅 API 可用，未作为独立打标工具开源。
【闭源】生成侧团队自研 captioner 几乎全部不开源：OpenAI Sora 的 highly descriptive captioner、Meta 的 LLaMa3-Video captioning 微调版、Google 用于 Veo 3 的 Gemini 打标变体、Lightricks LTX-2 的音视频 captioner、腾讯混元的三个自研 caption 模型、阶跃星辰 Step-Video 的 in-house VLM、字节 Seedance 1.5/2.0 的字幕系统、快手 Kling 3.0 Omni 的视频描述增强模块——均只披露「用了什么」，不披露参数量、基座与权重。
【一个反常识的结构性事实】打标器的开源度显著高于生成模型的数据 pipeline 开源度：多数闭源生成模型的技术报告愿意点名自己用了 Tarsier2 / Qwen3-Omni / LLaVA-Video，因为打标器被视为「工具」而非「壁垒」；真正被视为壁垒的是打标 prompt 原文、字段 schema 与阈值表，这部分几乎全行业不公开。

### [几何/结构化标注数据集合集](../models/geometric_datasets.md) ⚠️

均为学术开源数据集，权重非主要产物。SceneScribe-1M：论文明确表述开源，项目页 https://wangyunnan.github.io/SceneScribe-1M ，数据/标注对外发布，具体许可以官方仓库为准[不确定]；SpatialVID：开源程度最高，HuggingFace 发布 SpatialVID（271万clip）与 SpatialVID-HQ（37万clip）两个版本，约3.53TB，采用 CC-BY-NC-SA 4.0 许可（仅限非商用），标注流水线代码同步开源；WildWorld：GitHub 仓库 https://github.com/ShandaAI/WildWorld ，论文与基准开放，数据源自商业游戏《怪物猎人：荒野》，再分发条款未明确说明[不确定]；Action100M：GitHub 仓库 https://github.com/facebookresearch/Action100M ，标注体量约205GB（仅标注，视频依赖 HowTo100M 原始源），标注开源

### [视频生成后训练数据策略](../models/post_training_data.md) ⚠️

锚论文（2604.25427）：论文开源（CC BY 4.0）、代码与权重均未开源、基座为「an internal video generation model」（内部模型）、SFT 数据集与 RLHF prompt 集均未开源，仅在图 2 用公开的 Wan-2.1 做 RLHF 效果可视化。整体属「方法开源、数据与模型封闭」。
横向的后训练数据开放度梯度（从高到低）：
① 偏好数据完全开源：HPDv3（108万文本-图像对、117万成对比较标注）、VideoReward 偏好集（1.6万 prompt / 10.8万视频 / 18.2万标注三元组，含 VideoGen-RewardBench）——这两者是目前视频/图像生成后训练最重要的公开偏好资产；
② 偏好数据「准备发布」：JavisDiT++ 的约 2.5 万条音视频偏好对（截至调研时尚未公开）[不确定]；
③ 方法与流程公开、数据不公开：SkyReels-V2（3万人工样本对 + 三阶段各 2万共约 6万 DPO 数据）、Step-Video-T2V（Video-DPO 流程全公开、数量未公开）、HunyuanVideo 1.5（RLHF prompt 集构造与 GSB 标注协议公开、规模未公开）、LongCat-Video（GRPO 三奖励配置公开、SFT 集规模与 RM 标注量未公开）、Cosmos-Predict 2.5（五域 SFT 规模逐条公开、GRPO 配置公开，数据不开源但发布 RL 后 EMA 权重）；
④ 仅有一句话或完全空白：Sora 2、Veo 3/3.1、LTX-2、Kling 3.0 Omni（仅说用了 DPO）、Seedance 1.5 pro（仅说用了 RLHF + 多维 RM）。
奖励模型侧开源程度显著优于生成模型侧：HPSv3、VideoAlign/VideoReward、VisionReward、Unified Reward Model 均开源权重，构成了「开源 RM + 闭源生成器」的事实标准组合。

### [主流视频预训练数据集合并调研：Panda-70M、InternVid、Koala…](../models/pretraining_datasets.md) ⚠️

关键差异在于「是否托管视频本体」——这是复现成本的分水岭。
【仅发元数据（URL+时间戳+caption），视频需自行爬取】Panda-70M（CSV，Google Drive，含 matching_score/desirability/shot_boundary 列）；InternVid（jsonlines，HF 上 40.9GB，gated 需填姓名/单位/邮箱）；Koala-36M（10个CSV分片共48.9GB，HF 上无 dataset card、无 license 标签）；MiraData（4个CSV：330K/93K/42K/9K）；LVD-2M（3个CSV共约5.08GB，托管在 S3）。
【真正托管视频本体】OpenVid-1M（HF nkp37/OpenVid-1M，约12.4TB，74+个zip分片，超50GB的需cat重组；OpenVidHD-0.4M 单独约4.5TB）；UltraVideo（HF APRIL-AIGC/UltraVideo，clips_short_{1..36}.zip 原生分辨率 + 1920/960 降采样版本）。这两者是七者中唯一可直接拿到像素的。
【清洗代码开源程度】Koala-36M 开源了转场检测完整实现（含拟合好的SVM系数）与 VTSS 推理代码，但明确说明发布的是「base version」——实测配置为 fragments-only 的 FAST-VQA 架构（PLCC 0.8684），并非论文中含 ConvNeXt 静态分支+WCGB 的完整模型（PLCC 0.8974），即**发布的打分器不是造数据用的那个**；MiraData 开源了 GPT-4V 打标 prompt 与 MiraBench 评测代码，但**过滤阈值论文称在补充材料中而补充材料实际不存在**；LVD-2M 完全未开源过滤代码，仅在论文附图中给出 PLLaVA prompt；UltraVideo 仅开源推理代码，训练与清洗 pipeline 均未发布（GitHub issue #8 追问未果）；Panda-70M 开源切分代码（cutscene_detect/event_stitching/video_splitting）与学生 caption 模型权重，但未开源教师模型推理脚本与 UMT 选择器权重；InternVid 开源 ViCLIP，清洗 pipeline 仅文字描述；OpenVid-1M 开源 MVDiT 训练/推理代码与256~1024分辨率检查点。
【许可证】Panda-70M：Snap 非商用研究许可 + 继承 HD-VILA-100M 许可；InternVid：CC-BY-NC-SA-4.0；Koala-36M：快手非商用研究许可 + 继承 HD-VILA-100M 许可；MiraData：代码 GPL-3.0，数据条款自相矛盾（README 前段禁止商用、末段称「支持商用」）；OpenVid-1M：CC-BY-4.0 但声明仅供研究与非商业用途，且须遵守上游 Panda/ChronoMagic/CelebvHQ/Open-Sora-Plan 各自许可；UltraVideo：自定 license-april-lab.txt，非商用、禁止二次分发原始视频、须遵守 YouTube ToS 与 GDPR/CCPA；LVD-2M：无独立 LICENSE 文件，声明与 HD-VILA 许可一致。七者全部为非商用研究许可（OpenVid 的 CC-BY-4.0 名义宽松但被自身声明收窄），且均无一家真正拥有 YouTube 素材版权。

## 是否支持音视频同时生成，以及实现方式（原生联合/级联/MoE融合）

`av_generation` · 详细程度: brief

### [Allegro](../models/Allegro.md)

不支持。Allegro 为纯视觉的文生视频/图生视频模型，输出无声视频（88 帧、720×1280、15 FPS、约 6 秒，可用 EMA-VFI 插帧至 30 FPS）。全文数据 pipeline 不涉及任何音频轨道的抽取、过滤或标注，也没有级联的 V2A 模块或音频专家分支。所有音频相关维度在本条目中均不适用。

### [Apollo](../models/Apollo.md)

支持，且为原生联合生成（单塔单次前向同时产出音频与视频 latent，非级联）。实现方式：
- 单塔（Single-Tower）MultiModal Diffusion Transformer（MMDiT）架构，含 32 层联合扩散层（joint diffusion layers），音频与视频共享同一套 DiT block 参数，而非双塔 + cross-attention。
- 核心机制为 Omni-Full Attention：音频 token 与视频 token 在同一注意力窗口内做全连接自注意力，实现紧密的音视觉对齐与良好可扩展性。
- 位置编码为 MixD-RoPE（Mixed Dimension Rotary Position Embedding），统一处理视频的 3 维（t,h,w）与音频的 1 维时间轴索引。
- 训练目标为 flow matching（条件去噪）。
- 模型总参数 26B，flow-matching FFN 维度 4096。
- 输入为四路：视频、视频相关文本、音频相关文本、音频，各自独立编码后送入 MM-DiT。
【与双塔路线的对比证据】论文 Table 2 直接对比「Dual Tower（标准 cross-attention）」与「Single Tower（Omni-Full Attention）」，结论支持单塔全注意力方案，这与 MOVA/HunyuanVideo-Foley 等双塔+bridge 路线形成明确的架构分野。
【多任务统一】通过随机模态掩码（random modality masking）在同一模型内统一支持 T2A、T2V、T2AV、I2V、I2AV 五类任务，因此同一权重既能联合生成也能单模态生成。

### [CineDance / CineDance-1M](../models/CineDance.md)

支持音视频同时生成，属于「原生联合生成」路线（native joint audio-video generation），而非先出视频再配音的级联方案。
【数据侧定位】数据集的核心卖点之一即「保留原生音轨」（native audio track），论文将「缺失声学模态」列为现有数据集四大缺陷之一，所有序列均带原始同步音频，标注同时覆盖视觉与听觉两条轨道。
【模型侧实现】CineDance 模型基于 LTX-2.3，架构为 13B 视频 DiT + 3B 音频分支 + 3B 跨模态 cross-attention 模块，视频与音频在同一扩散过程中通过跨模态注意力耦合，属于双塔 + 跨注意力融合的原生联合架构，非 MoE 融合、非级联后配音。
【任务定义】论文将任务表述为 T2AV（Text-to-Audio-Video），即由文本提示一次性生成带同步音轨的多镜头长视频。

### [CogVideoX](../models/CogVideoX.md)

CogVideoX 模型本体不支持音视频同时生成，为纯视觉的文生视频/图生视频模型，训练数据不使用音轨。
产品层（「新清影」，2024年11月）通过级联（cascade）方式实现「自带音效」的视频：先由 CogVideoX（1.5 版）生成 10 秒 4K/60 帧无声视频，再由独立的音效模型 CogSound 以视频为条件做 V2A（video-to-audio）配音。CogSound 的公开技术要点为：基于 GLM-4V 的视频理解能力提取语义/情感 → 潜空间扩散模型（Latent Diffusion）生成音频 → 用「分块时序对齐交叉注意力（Block-wise Temporal Alignment Cross-attention）」建立帧级视频特征与音频特征的对应关系 → 叠加旋转位置编码（RoPE）提升长序列时序一致性。可生成爆炸、水流、乐器、动物叫声、交通工具等复杂音效与节奏元素。
因此实现方式为「级联」，而非原生联合或 MoE 融合；且两个模型的数据管线彼此独立，视频侧论文完全不涉及音频维度。

### [Cosmos-Predict2.5](../models/Cosmos-Predict25.md)

不支持。Cosmos-Predict2.5 是纯视觉的视频世界模型，输入为文本 / 图像 / 视频条件，输出为无音轨视频（93 帧、16fps、约 5.8 秒）。全文 44 页未出现 audio / speech / sound 相关的数据处理或生成描述，音频既不是条件也不是输出。其「多模态」体现在文本编码器换用 Cosmos-Reason1（Physical AI 专用 decoder-only VLM）以及机器人变体的 action 条件输入，而非音视频联合生成。因此本调研中所有音视频相关维度（音频类别配比、联合 caption、对白转写、唇同步/事件对齐、音频质量过滤、音频类型处理）对该工作均不适用。

### [Data-Juicer 2.0](../models/Data-Juicer.md)

不支持音视频同时生成——Data-Juicer 本身不是生成模型，不产出任何音频或视频内容。但它在数据侧提供了支撑音视频联合任务的算子能力，与本调研的 AV 主线有实质关联：
【音频侧算子】audio_duration_filter（时长过滤）、audio_nmf_snr_filter（基于 NMF 的信噪比过滤）、audio_size_filter（文件大小过滤）、audio_add_gaussian_noise_mapper（加噪增广）、audio_ffmpeg_wrapped_mapper（FFmpeg 音频滤镜封装）。
【音视频跨模态算子】video_audio_ASR_mapper（从音轨做语音识别/打标）、video_tagging_from_audio_mapper（基于 Audio Spectrogram Transformer 从音轨生成视频标签）、video_captioning_from_audio_mapper（基于 Qwen-Audio 根据音轨为视频生成 caption）、video_audio_detect_age_gender_mapper（基于 wav2vec2 从音轨检测说话人年龄与性别）、video_audio_speech_emotion_mapper（语音情绪识别）、video_active_speaker_detect_mapper（联合视觉人脸轨迹与音频信号做主动说话人检测——这是 DJ 中最接近「音视频同步判定」的算子）。
【定位判断】DJ 的音视频能力偏向「理解与标注」而非「同步质量把关」：它有主动说话人检测，但没有 SyncNet/Synchformer 式的同步偏移量与置信度打分算子，也没有 speech/foley/music 三类音轨分离与配比控制算子。因此若要复用 DJ 构建 AV 联合生成训练集，同步过滤与音轨分类环节仍需自行扩展算子。
【实证案例】其官方文生视频案例（VBench 登顶）为纯文生视频（T2V），全流程未涉及音频处理。

### [Foley-Omni](../models/Foley-Omni.md)

不是「音视频同时生成」模型，而是「视频条件下的多类音频联合生成」。实现方式为原生联合生成（native joint generation）而非级联：语音、音效、音乐三类音轨在同一个共享隐空间的扩散生成过程中一次性联合产出，而非分别用TTS/Foley/音乐模型生成后混音。这是相对基线（MMAudio + CosyVoice 3 + AudioX 级联管线、MMAudio + LipVoicer + AudioX 级联管线）的核心差异点。
架构上为 Diffusion Transformer（DiT），文本侧用共享的 UM-T5 encoder 编码三字段结构化文本，音频侧用 frozen Mel VAE（沿用 MMAudio）+ BigVGAN vocoder，视频侧双路条件：CLIP 特征提供场景语义、Synchformer 特征提供时序同步线索（后者以 additive path Z_sync 注入）。
实证支持联合生成优于级联：V2ST-Bench 上 Foley-Omni 的 WER 7.59 优于 MMAudio+CosyVoice 3+AudioX 的 10.57 与 MMAudio+LipVoicer+AudioX 的 37.84，DeSync 0.16 亦优于 0.85/0.26，且三项主观分 A-MOS 3.92 / S-MOS 4.13 / T-MOS 4.14 全面领先级联基线（对照 Ground Truth 为 4.33/4.37/4.42）。

### [Goku](../models/Goku.md)

不支持。Goku 是纯视觉的图像+视频联合生成模型（T2I / T2V / I2V），论文全文未涉及音频生成、音轨建模、语音或音效。其「联合生成」指的是图像与视频两种视觉模态在同一 rectified flow Transformer 中的联合建模（图像视为单帧视频，经 3D 联合图文 VAE 编码后与视频 token 在全注意力中统一交互），而非音视频联合。因此本条目在音视频维度上不适用，其价值主要体现在纯视频侧的数据分布均衡方法论。

### [Hailuo / MiniMax Video](../models/Hailuo.md)

不支持音视频同时生成（截至 Hailuo 2.3 / 2026年7月）。
- MiniMax 开放平台视频生成 API 文档中不含任何音频/声音参数，输出为无声视频；
- Replicate 上托管的全部 7 个 Hailuo/video-01 系列模型，均未描述任何随视频同时产出音轨的能力；
- MiniMax 的音频能力由完全独立的产品线承担：MiniMax Speech（语音合成，2.8 版）与 MiniMax Music（音乐生成，3.0 版），海螺AI 网站上「Audio」是与视频并列的独立入口；
- 若从产品体验层面看，海螺AI 的 Media Agent（2025年10月随 2.3 上线）可一键串联文本→视频→语音/音乐，属于产品编排层的级联（cascade），而非模型层的原生联合生成（native joint）或 MoE 融合。
因此本条目在「音视频联合生成」这一调研主线上属于反例/对照组：它代表了在 Veo 3、Sora 2、Kling 3.0 Omni、LTX-2 等已转向原生 AV 联合生成之后，仍停留在纯视觉生成范式的头部商用模型。相应地，本调研中所有 AV 相关字段对该对象均无实质内容。

### [HunyuanVideo-Foley](../models/HunyuanVideo-Foley.md)

不支持音视频同时生成——这是理解本工作定位的关键。HunyuanVideo-Foley 是级联范式中的「后半段」：视频已经存在（可以是真实拍摄素材，也可以是 HunyuanVideo 等模型生成的无声视频），模型的任务是为其补配音效。因此严格说属于 V2A / TV2A（Text-Video-to-Audio）单向条件生成，而非 UniVerse-1、MOVA、Ovi 那类原生联合生成。
【与联合生成模型的分工差异】联合生成模型需要同时解决「视频长什么样」与「声音是什么」两个问题，本模型只解决后者，因此可以把全部模型容量与数据预算投入到音频侧的保真度与同步精度上——这也解释了为何其音频质量指标（PQ 6.59、MOS-Q 4.14）显著高于同期的联合生成模型。
【架构实现】约 30 亿参数的混合式多模态扩散 Transformer：前段 18 层 MMDiT（多模态双流块，音频与视频 token 走各自的参数分支但共享一次 joint self-attention），后段 36 层 unimodal audio DiT（纯音频单流块，做音频细节精修）。隐层维度 1536，注意力头数 12。
【三路条件注入方式各不相同】(1) 视频语义——SigLIP-2 视觉编码器提取帧特征，与音频 latent 拼接后走 joint self-attention 深度融合；(2) 文本语义——CLAP 文本编码器编码，通过 cross-attention 注入（刻意与视频走不同通路，论文称此举是为化解「模态竞争」modal competition，即防止强势的视觉条件淹没文本条件）；(3) 帧级同步信号——Synchformer 提取帧级同步特征，通过 adaLN + 门控调制（gated modulation）通路注入，不参与注意力计算。
【交错 RoPE（interleaved RoPE）】把音频 token 与视觉 token 按时间轴交错排列后再施加旋转位置编码，使同一时刻的音、视 token 在位置编码上彼此邻近，强化帧级同步依赖。消融显示去掉后 DeSync 从 0.78 恶化到 0.79、CLAP 从 0.30 变化，影响主要体现在时序对齐上。
【音频表示】自研 DAC-VAE，48 kHz 采样率，将 DAC 原有的 RVQ 离散量化块替换为高斯分布 + KL 正则的连续变分自编码器，输出 128 维连续 latent，latent 帧率 50 Hz。

### [HunyuanVideo](../models/HunyuanVideo.md)

不支持。HunyuanVideo 与 HunyuanVideo 1.5 均为纯视觉视频生成模型，输出无音轨。1.5 技术报告全文未出现任何音频生成相关内容，数据侧也不涉及音轨处理。
腾讯的音频能力由独立模型承担而非联合生成：HunyuanVideo-Foley（2025年8月，视频到音频/Foley生成，基于约10万小时TV2A数据集）、HunyuanVideo-Avatar（音频驱动数字人）等，属于「级联/外挂」形态——先由 HunyuanVideo 生成画面，再由 Foley 模型配音，而非原生联合去噪。因此本条目在音视频联合生成维度上不构成参考样本，本调研中所有音频相关字段（audio_category_distribution、av_sync_detection、sync_metric_and_threshold、temporal_vs_semantic_sync、audio_quality_filtering、audio_type_handling、joint_av_caption_schema、dialogue_transcription_attributes）均不适用。

### [InstructAV2AV](../models/InstructAV2AV.md)

支持音视频联合生成，但任务形态是「联合编辑」而非「从零联合生成」，且实现方式为原生联合（native joint）而非级联。
【模型侧架构】基于 Ovi（Low et al., 2025）的对称双塔扩散 Transformer（symmetric twin-backbone diffusion transformer）：视频塔与音频塔并行，通过跨塔交互实现联合去噪，视频侧用时空 VAE（spatial-temporal VAE）编解码、音频侧用 1D VAE 编解码、文本侧用 T5 编码指令。音视频在同一次扩散过程中同步产出，天然保证时序对齐，而非「先生成视频再配音」的级联。
【三项针对编辑任务的改造】
  1. Source Concatenation（SC）：把源音视频的 latent 与噪声 latent 沿通道拼接，为生成过程锚定源上下文，是保持非目标区域不变的核心机制。消融显示去掉后 FVD 从 180.38 恶化至 467.20（劣化1.6倍），背景严重退化——是三项设计中影响最大的。
  2. Source-Instruction Gated Attention（SIGA）：源信息与指令信息之间的软门控注意力，平衡「跟随指令改」与「保留原内容」这对矛盾目标。消融去掉后 FAD 从 2.75 升至 3.26，出现音频幻觉与结巴（stuttering）。
  3. 两阶段训练策略（TSTS）：先分模态各自适配、再联合微调，用于平滑迁移预训练先验。消融去掉后 FVD 升至 291.55、FAD 升至 5.18，出现视觉扭曲与不一致。
【损失加权】联合训练采用模态平衡权重 λ_v = 0.85、λ_a = 0.15，视觉侧权重远高于音频侧（约 5.7:1），反映视觉分支收敛难度更大。
【数据引擎侧的音视频耦合】值得单独指出的是，构造训练数据时用的 mask-guided 视频编辑模型（基于 Wan2.2-5B）本身也做了音视频耦合设计——通过 frame-wise cross-attention 把音频特征注入视频合成过程，以保证合成出的目标视频与已合成的目标音频严格时序同步。即「合成数据阶段」就已经在强制音视频对齐，而非事后过滤。

### [2026 其他音视频联合生成](../models/JAVG_2026_misc.md)

七项全部围绕音视频同时生成，但实现路径分五种，恰好构成 2026 年 JAVG 技术路线的一个横切面：
(1) 【原生联合 + 语义前置规划】Baton：在联合 DiT 去噪之前，先由 VA-Planner（以 Qwen3-8B 为底的多模态 LLM，带 dual semantic alignment towers）生成一批「planned tokens」作为音视频共享的语义蓝图，再用 Relative Semantic RoPE（RS-RoPE）把蓝图注入去噪过程。其核心论点是「现有方法依赖 off-the-shelf 文本编码器的粗粒度 embedding，丢失细粒度语义且缺乏共享长时规划」。规划器不预测离散 token，而是回归连续特征（L2 回归到冻结 SigLip2 视频特征与 WavTokenizer 音频特征的倒数第二层），理由是「regressing continuous features preserves richer semantic structure」。
(2) 【原生联合 + 定制化 LoRA】OmniCustom：在已有联合音视频生成基座上，用两组独立 LoRA（reference identity LoRA 与 audio timbre LoRA）分别作用于 self-attention 层，实现「给一张参考图 + 一段参考音频，生成保持该身份、模仿该音色、说出 prompt 指定台词的同步音视频」。额外引入对比学习目标（有参考条件的预测流为正例、无参考条件的为负例）与 flow matching 并行。
(3) 【流式自回归 + 解耦编排】StreamChar：把「长时编排」与「短窗去噪」解耦——LLM 编排器（Orchestrator）读取台词全文与历史上下文，产出逐帧对齐的音频条件；联合音视频 DiT 在局部窗口内做双向去噪，配 reference frame 与 motion frame 条件。每 chunk 输出 33 帧 @24fps，历史音频上下文上限 15 秒，配 progress-aware pointer（PAP，用 ASR 时间戳的 ground-truth end indices 做 smooth L1 监督）与 persistent visual anchor（sink chunk）抑制长程漂移。单张 H100 实时，可连续生成 5 分钟流。
(4) 【双塔原生联合（大厂基座）】ALIVE：由预训练 T2V 模型改造而来的统一音视频合成模型，VideoDiT 12B + AudioDiT 2B，支持 Text-to-Video&Audio 与 Reference-to-Video&Audio（动画化）。关键架构件为 TA-CrossAttn（时序对齐的跨模态融合）与 UniTemp-RoPE（统一时序 RoPE 做精确音画对齐）。480p 基座 + 1080p refiner 两级。
(5) 【双流 Transformer 范式的修补】CCL：不改范式，专门诊断并修复双流 Transformer 的三类缺陷——门控机制引起的模型流形漂移、跨模态注意力在多模态背景区域引入的偏置、多模态 CFG 训练与推理不一致。对应四个模块：TARP（时序对齐 RoPE 与分区）、LCT + DCR（稳定的无条件锚点与跨模态信息的动态路由）、UCG（Unconditional Context Guidance，推理一致性）。最大卖点是用远少于同类的数据与算力达到 SOTA。
(6) 【Align-then-Fuse 原生对齐】NAVA：明确反对两种既有设计——双塔（dual-tower）削弱细粒度同步、统一三模态（unified tri-modal）把语义对齐与低层对齐混为一谈。改为 Align-then-Fuse MMDiT：先在专用对齐空间建立音视频对应关系，再把上下文（文本、说话人嵌入）作为外部条件通过 cross-attention 融入共享去噪。附 Timbre-in-Context Conditioning，把参考音色线索绑定到对应的语音片段（speech spans），支持多说话人参考音色控制。6.3B 参数，原生立体声输出，支持 T2AV / I2AV / T2A。
(7) 【推理时缩放（不改模型）】ITS-JAVG：把单模态领域的 Inference-Time Scaling 迁到联合音视频生成，完全 training-free。核心发现是单一验证器必然导致 verifier hacking 与指标间的非对称 trade-off，必须用多验证器组合；并提出 Adaptive Reward Weighting（ARW），把奖励聚合当作在线优化问题。可视为在推理阶段复刻了训练数据 pipeline 中的「多打分器联合过滤」思想。

### [音视频联合生成基线合集](../models/JavisDiT_baselines.md)

五者全部支持音视频同时（联合）生成，且全部属于「原生联合」而非级联，但跨模态交互机制各不相同，构成一条清晰的技术演进线：
(1) MM-Diffusion（2022）：双分支序列式多模态 U-Net，视频用 2D+1D 时空卷积、音频用空洞卷积（dilated conv）；核心是 random-shift based attention（随机偏移注意力块）跨接两个子网，把跨模态注意力复杂度从 O((F×H×W)×T) 降到 O((S×H×W)×(S×T/F))。像素空间扩散，非 latent。另可零样本迁移到视频到音频、音频到视频的条件生成（靠梯度引导），无需额外训练。
(2) AV-DiT（2024）：主打「参数高效」——共享一个仅在图像数据上预训练的 DiT 主干（冻结），音频与视频两路仅插入轻量 adapter 可训练；视频分支在冻结 DiT block 中加入可训练时间注意力保证时序一致性，音频分支同样靠轻量参数适配，再加跨模态特征交互模块。属于「冻结共享骨干 + 双模态 adapter」的联合生成范式。
(3) JavisDiT（2025）：双塔 DiT（视频塔源自 Open-Sora，音频塔源自 AudioLDM2，二者冻结 VAE），核心创新是 HiST-Sypo Estimator（分层时空同步先验估计器）——从文本 prompt 中先估计出一组「粗粒度全局先验 + 细粒度时空先验」，再以该先验同时引导音频与视频两路去噪，实现细粒度时空对齐；跨模态交互靠 cross-attention 与 bidirectional attention 模块。
(4) JavisDiT++（2026）：换用 Wan2.1-1.3B-T2V 为底座，三项升级——模态特定混合专家 MS-MoE（Modality-Specific Mixture-of-Experts，在保证跨模态交互的同时提升单模态质量）、时间对齐旋转位置编码 TA-RoPE（Temporal-Aligned RoPE，实现音频 token 与视频 token 的显式帧级同步，思路与 Ovi 的 scaled-RoPE 同源）、以及音视频直接偏好优化 AV-DPO。
(5) UniAVGen（2025）：双分支联合合成架构，两个并行 DiT 分别处理音频与视频，核心是「非对称跨模态交互（Asymmetric Cross-Modal Interactions）」——两个模态间的信息流不对等，配合 Face-Aware Modulation（人脸感知调制）模块与 Modality-Aware Classifier-Free Guidance（模态感知无分类器引导）；一套框架统一支持联合生成、视频到音频配音（dubbing）、音频驱动视频动画等 5 类任务。
(6) Harmony（2025）：视频分支由 Wan2.2-5B 初始化，音频侧用 MMAudio 的 VAE 编码器 + F5-TTS 的语音编码器。三项创新：跨任务协同训练（Cross-Task Synergy，用「音频驱动视频」「视频驱动音频」两个双向生成任务来抑制联合去噪时的对齐漂移）、全局-局部解耦交互模块（Global-Local Decoupled Interaction, GLDI，全局分支管风格对齐、局部分支管时序精度）、同步增强的无分类器引导（Synchronization-Enhanced CFG，推理期放大对齐信号）。作者明确指出联合扩散范式的三大痛点：并发噪声演化下的对齐不稳定、注意力机制对时序精度的低效、标准 CFG 缺乏跨模态同步引导。

### [可灵 3.0 / 可灵视频 3.0 Omni](../models/Kling_30_Omni.md) ⚠️

支持。官方定位为“原生音画同出/原生音画同步”：视频、对白语音、环境音与音效在同一次生成pass内由统一模型联合产出，无需级联的后置配音或V2A流程（区别于早期 Kling-Foley 式的视频→音频级联方案）。基于统一的多模态视觉语言（MVL）表征与3D时空联合注意力，音频与视觉在共享embedding空间内联合建模。支持中、英、日、韩、西五种语言及多种地方口音的口型与情绪同步；Omni 版本额外支持多图+音色参考绑定（角色形象与音色一同锁定）、3个以上角色的多说话人共指与区分音色、多镜头分镜间的音频连续性（Director Memory 上下文库）。规格：单次最长15秒、最高原生4K（3840×2160）、最高60fps、单条内可自由组合最多6个镜头。[不确定：底层是单一DiT联合去噪，还是视频/音频双分支+跨模态注意力的MoE式融合——技术报告未披露音频分支细节]

### [LTX-2](../models/LTX-2.md)

支持，且为原生联合生成（native joint generation），是本模型的核心定位。实现方式为「解耦但集成的非对称双流」（decoupled yet integrated asymmetric dual-stream），既非级联也非 MoE 融合：
(1) 模态各自独立 VAE：视频用时空因果 VAE；音频先转 16kHz 立体声 mel 频谱（双通道沿 channel 维拼接）再过独立因果音频 VAE，每个 latent token 约对应 1/25 秒音频、128维；解码后经改造版 HiFi-GAN 声码器（通道数翻倍以支持立体声）上采样重建 24kHz 双声道波形。
(2) 非对称双流 DiT：14B 参数视频流 + 5B 参数音频流（共19B），共享深度。每个 dual-stream block 顺序执行：同模态 Self-Attention → 文本 Cross-Attention → 音视频 Cross-Attention → FFN，层间 RMSNorm。视频流用 3D RoPE，音频流用 1D 时间 RoPE。
(3) 跨模态交互：双向 audio-video cross-attention 贯穿全深度；跨模态注意力中只使用 RoPE 的时间分量（强制注意力聚焦于时间同步而非空间对齐）；引入 cross-modality AdaLN——一路模态的 scale/shift 由另一路的隐状态与 diffusion timestep 决定，用于调节各阶段吸收多少跨模态信息。
(4) 推理侧 modality-aware CFG（Bimodal CFG）：\hat{M}=M(x,t,m)+s_t(M(x,t,m)-M(x,∅,m))+s_m(M(x,t,m)-M(x,t,∅))，文本引导 s_t 与跨模态引导 s_m 独立调节；论文取视频流 s_t=3, s_m=3，音频流 s_t=7, s_m=3。增大 s_m 可提升时序同步与语义一致性。
(5) 解耦 latent 天然支持 V2A（为已有视频配同步音频）与 A2V（由音轨驱动视频）编辑工作流。
(6) 输出能力：最长20秒连续音视频（超过 Veo 3 的12秒、Sora 2 的16秒、Ovi 的10秒）；产品口径宣称原生 4K、最高 50fps；论文中描述的推理策略为多尺度多分块：先在约 0.5MP 低分辨率生成 base latent 建立全局构图/运动/音画同步 → latent 上采样器提升空间分辨率 → 重叠时空 tile 分块精修到 1080p 后在 latent 空间融合。（4K/50fps 为产品与新闻稿口径，论文正文只写到 Full-HD 1080p。）

### [LongCat-Video](../models/LongCat-Video.md)

基础版 LongCat-Video 不支持音视频同时生成——它是纯视觉生成模型（T2V/I2V/视频续写），输出无音轨。
同系列的 LongCat-Video-Avatar / Avatar 1.5 是「音频驱动视频生成」（Audio-Text-to-Video, AT2V 与 Audio-Text-Image-to-Video, ATI2V），属于音频作为条件输入、视频作为输出的单向驱动，而非音视频联合生成（不生成音频）。实现方式为在同一 13.6B DiT 基座上注入音频特征：Avatar 用 wav2vec2 音频编码器，Avatar 1.5 升级为 Whisper-Large 编码器以提升唇同步精度，支持单流与多流（多人）音频输入。因此按「原生联合/级联/MoE融合」分类，该系列均不属于任何一类，应归为「音频条件驱动的视频生成」。

### [MOVA](../models/MOVA.md)

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

### [合并条目：① Mochi 1](../models/Mochi_MAGI_Motif.md)

三者均不支持音视频同时生成，全部为纯视觉（无声）视频生成模型。
① Mochi 1：输出 848×480 / 30fps / 5.4s 无声视频，文本编码器为单个 T5-XXL，架构中无音频分支，官方博客与模型卡均未提及音频。
② MAGI-1：技术报告全文（含 Section 3 DATA 与 Section 4 Infrastructure）不涉及音轨抽取、音频过滤或音画对齐；数据管线在 PySceneDetect 切分后仅保留视觉帧序列。MAGI-1.1 亦未加入音频能力。
③ Motif-Video 2B：数据管线明确只处理 Image Real / Image Synthetic / Video Real / Video Synthetic 四个视觉分支，caption 的 JSON schema 中不含任何听觉字段，全文无 audio / speech / music 相关处理。
因此本条目中所有音频相关维度（audio_category_distribution、joint_av_caption_schema、dialogue_transcription_attributes、av_sync_detection、sync_metric_and_threshold、audio_quality_filtering、audio_type_handling）均为「不适用」。

### [Movie Gen](../models/Movie_Gen.md)

不支持原生音视频联合生成，采用级联（cascade）方案：Movie Gen Video（30B，Flow Matching Transformer）先生成无声视频，再由 Movie Gen Audio（13B，Flow Matching DiT + DAC-VAE 潜空间）以视频和文本为条件生成同步音轨（V2A / TV2A），支持音频扩展（audio extension）实现最长数分钟的整片配乐。音频侧内部是「单一模型联合生成所有音频类别」（diegetic音效、non-diegetic音效、器乐音乐一起生成），而非按类别分多个模型，理由是不同音频类别之间也存在相关性。论文结论明确指出：视频与音频当前是分开训练的，「让模型联合生成这两个模态是重要的未来研究方向」。此外模型有意不生成人声/对白（认为可由TTS补齐、且无脚本时难以生成）。

### [NVIDIA NeMo Curator](../models/NeMo_Curator.md)

不适用——本条目是数据处理工具链而非生成模型，不产出任何音视频内容。
就「音视频联合处理能力」而言需明确指出其当前局限：NeMo Curator 虽然号称四模态（文本/图像/视频/音频），但视频 pipeline 与音频 pipeline 在架构上相互独立、不交叉：视频侧只处理视觉轨道（切分/转码/抽帧/运动与美学过滤/captioning/嵌入/去重/分片），音频侧是面向 ASR 语音数据的独立工作流（加载→NeMo ASR 转写→WER/CER 质量评估→与文本 curation 衔接→导出）。官方文档中音频模块未提及任何与视频的联动，也没有从视频中抽取音轨、做音视频对齐或联合打标的 stage。因此它目前不能直接支撑「音视频同时生成」模型的数据构建，是该框架相对于 LTX-2 / Ovi / Sora 2 等 AV 联合生成模型数据需求的最大缺口。

### [OmniHuman 数据集 + OHBench](../models/OmniHuman.md)

本条目是数据集与评测基准而非生成模型，因此不自身生成音视频；但其全部设计都以「音视频联合生成」为服务对象，是本次调研中少数在数据侧原生按 AV 联合范式构建的资源：
【数据侧的 AV 原生性】所有样本强制含有与画面同步的原始音轨，并经 SyncNet 归属校验（详见 av_sync_detection）；音频被 Demucs 做四源分离后分别标注人声与背景音；caption 中同时覆盖视觉（外观、动作、表情）与听觉（语音内容、情绪、背景声、音乐属性）两条轨道。
【评测侧支持的任务】OHBench 明确支持多种人物中心生成任务：音视频联合生成（audio-video joint generation，即 I2AV）、语音驱动视频生成（speech to video）、视频配音（video dubbing）、可控人物视频编辑（controllable human video editing），以及下游语音生成任务。其中 I2AV 是主评测任务，共评测 10 个模型（闭源 5 个：Veo3.1、Wan2.5、Sora2、Kling2.6、SeedDance-1.5-pro；开源 5 个：UniVerse-1、UniAVGen、Ovi、MOVA、LTX-2）；speech-to-video 任务中 InfiniteTalk 在部分指标上表现最佳。
【对生成范式的中立性】数据与评测不预设「原生联合 / 级联 / MoE 融合」中的任何一种实现方式，同时容纳联合生成模型（Ovi、MOVA、LTX-2）与级联/配音式模型（InfiniteTalk 类），这与其作为通用基准的定位一致。
【实证支撑】通过用 OmniHuman 数据微调 LTX-2（原生 AV 联合生成模型）并在 OHBench 上取得全面提升，证明该数据对 AV 联合生成模型确有增益（详见 data_ablation）。

### [Open-Sora 系列](../models/Open-Sora.md)

均不支持。Open-Sora 与 Open-Sora Plan 全系列（截至 Open-Sora 2.0 / Open-Sora Plan v1.5.0）均为纯视觉（无声）视频生成模型，输出无音轨；技术报告、GitHub 文档与数据 pipeline 中均无任何音频编码器、音频 latent、音视频联合去噪或音视频对齐的设计与描述。数据处理侧也完全不涉及音轨——切分、打分、打标全部基于视觉帧。因此本条目下所有音频与音视频对齐相关字段（audio_category_distribution、joint_av_caption_schema、dialogue_transcription_attributes、av_sync_detection、sync_metric_and_threshold、temporal_vs_semantic_sync、audio_quality_filtering、audio_type_handling）均为「不适用」。

### [Ovi](../models/Ovi.md)

支持，且是原生联合生成（one-pass joint AV generation），不是级联也不是 MoE 融合，作者称之为「twin backbone（孪生/双骨干）blockwise cross-modal fusion」。
(1) 对称孪生 DiT：音频塔与视频塔架构完全相同（Model Dim 3072、FFN Dim 14336、24 heads、head dim 128、30 个 block，每个 block 各有 30 层 Self-Attn / Text Cross-Attn / AV Cross-Attn）。视频分支由 Wan2.2 5B 初始化，音频分支为同架构从零训练的 5B，合计约 11B。
(2) 逐 block 双向跨模态注意力：每个 transformer block 内音频流 attend 视频流、视频流反向 attend 音频流，同步线索贯穿全网络。因两塔隐维一致，无需任何投影层（对比 UniVerse-1 需要插块+投影+辅助语义对齐损失）。
(3) 时序对齐靠 scaled-RoPE：视频 latent 31 帧、音频 latent 157 token（16kHz×5s/512），将音频分支 RoPE 频率按 31/157≈0.197 缩放，使跨模态 RoPE 亲和矩阵对角线对齐。
(4) 单一冻结 T5 编码器编码「合并 prompt」（视觉描述 + <S>台词<E> + 音频描述），同一份文本嵌入分别与音频塔、视频塔做 cross-attention，统一跨模态语义控制。
(5) 训练目标为 flow matching，两模态共享同一 timestep t、各自独立噪声，总损失 L=0.85·L_video+0.15·L_audio；无任何显式同步损失、无人脸 mask、无 post-hoc 对齐、无辅助同步模块。推理时两分支共用同一 ODE 求解器（UniPC）。
(6) 能力：Ovi 初版 5 秒 720×720@24fps；Ovi 1.1 为 10 秒 960×960@24fps，支持 9:16、16:9、1:1 等多种宽高比，同时输出对白、音效与背景音乐。

### [Script-a-Video](../models/Script-a-Video.md)

支持音视频同时生成，但需区分两个层次：
【本工作的定位】Script-a-Video 本身不是一个新的音视频生成基座，MTSS 是「输入侧的结构化条件表示」。生成能力通过在已有基座上做条件替换与轻量架构改造来验证。
【生成基座】选用 LTX-2 作为生成框架，原因有二：(1) 其 Gemma 系 VLM 编码器天然擅长解析 MTSS 这类 JSON 式结构化语法，可从分流字段中抽取细粒度语义指令；(2) 其非对称双流 Diffusion Transformer（asymmetric dual-stream DiT）架构本身即为音视频联合合成设计，可把 MTSS 的 Shot 流与 Event 流分别映射到视频分支与音频分支的隐空间。
【实现方式归类】原生联合生成（单次生成过程同时产出同步的视频与音频），非级联、非 MoE 融合。
【两处架构改进】
1) Shot-Aware Structured Attention（镜头感知结构化注意力）：按 MTSS 的 shot 边界切分 Gemma-3 文本 embedding，再让每个镜头对应的视频 token 只与本镜头的语义片段做交叉注意力，实现镜头间上下文隔离，防止跨镜头语义串扰。
2) Identity Customization（身份定制）：通过 reference VAE 特征 + 可学习的 reference-learnable-tokens，把 Reference 流中的 ID 符号（如 "PERSON_1"）与对应参考图像显式对齐，作为视觉身份与语言指称之间的关系桥梁。
【多模态输入形态】多模态信息以图文交错（interleaved image-text）格式送入 Gemma-3，同时为视频与音频两个分支提供语义表示。
【任务形态】MTSS 三元组（S_ref, S_shot, S_eve）→ 同步的视频-音频对 (V, A)，目标是同时满足身份持久性与时序-听觉精确性。

### [Seedance 2.0 与 Seedance 1.5 pro](../models/Seedance_20_Seedance_15_pro.md) ⚠️

支持。Seedance 1.5 pro 为原生音视频联合生成（native joint audio-video generation）：采用双分支 Diffusion Transformer（报告摘要表述为 dual-branch DiT，正文表述为基于 MMDiT 的统一框架）+ 跨模态联合模块（cross-modal joint module），在大规模混合模态数据上做多任务预训练，同时支持 T2VA、I2VA 以及纯视觉的 T2V/I2V，因此属于「原生联合」而非级联式先视频后配音。Seedance 2.0 进一步升级为统一的多模态音视频联合生成架构（unified multi-modal audio-video joint generation architecture，官方博客提及采用稀疏架构 sparse architecture），支持文本/图像/音频/视频四种输入模态；音频侧新增双声道（binaural/双通道立体声）能力，并支持背景音乐、环境音效、角色人声旁白的多轨并行输出，与画面节奏精确时间对齐。[不确定：2.0 的具体融合方式（是否为 MoE 融合）未在报告中说明]

### [SkyReels 系列](../models/SkyReels.md)

SkyReels-V2 不支持音频生成（纯视频模型）。SkyReels-V4 支持音视频原生联合生成，实现方式为「双流 MMDiT（dual-stream Multimodal Diffusion Transformer）」，属原生联合而非级联，也非 MoE：
(1) 视频分支与音频分支为对称双主干（symmetric twin backbone），视频分支由已有文生视频模型初始化，音频分支从零训练，两者规格对齐；
(2) 共享一个冻结的多模态大模型（MLLM）作为文本编码器，直接处理「视觉+听觉合一」的描述文本，使同一条 prompt 同时驱动两个模态；
(3) 每个 transformer block 内部设双向音视频交叉注意力（audio→video 与 video→audio 互为条件），前段为 dual-stream MM 层、后段转 single-stream 混合块；
(4) 通过 RoPE 时间维缩放对齐两模态时序（音频/视频 token 比例约 0.0963；音频侧 44.1kHz、5秒对应218个 latent token，对齐视频21帧的时间分辨率）；
(5) 生成能力：最高 1080p、32FPS、15秒，支持多镜头电影级内容与同步音轨；
(6) 效率策略：基座模型先生成「低分辨率全序列 + 高分辨率关键帧」，再由 Refiner 模块（基于 Video Sparse Attention 的联合超分+插帧）出片；
(7) 统一性：通过通道拼接（channel-concatenation）的 inpainting 框架 + 不同 mask 配置，把生成、修补、编辑统一到同一模型，输入可为文本、图像、视频片段、mask、音频参考。

### [Sora 2](../models/Sora_2.md) ⚠️

支持，且为原生联合生成（native joint generation），这是 Sora 2 相对 Sora 1 的核心升级点。System Card 明确将其定位为「video and audio generation model」，新增能力包括「synchronized audio」。音频不是后处理级联的video-to-audio模块，而是与视频在同一生成管线内联合去噪产出：视频与音频分别经各自编码器压缩到latent，再由同一个transformer扩散主干对两路latent同时去噪。可生成对白（含唇形同步）、音效/foley、环境音与背景音乐，音量与空间定位随物体与镜头距离变化。注意：上述架构描述（双编码器+共享扩散主干、3D RoPE、音频backbone与GPT-4o多模态系统同源）均来自第三方技术解读与二手报道，OpenAI 官方从未确认，属于推测性信息。[不确定]

### [SpeakerVid-5M](../models/SpeakerVid-5M.md)

数据集层面：原生音视频成对（每个 clip 都自带与画面同步的原生音轨，无音轨或音视频不同步的样本在清洗阶段即被剔除），因此它是「音视频联合生成」训练的合格语料，也是当前少数以双人对话交互（dyadic interaction）为核心组织形态的音视频数据集。
基线模型层面：支持音视频同时生成，实现方式为原生联合的自回归框架，而非级联。具体构成为 Qwen2.5-Omni 作多模态理解主干 → 同时预测视频 token 与音频 token；视频侧用 3D VAE（时间 stride 4、空间 stride 8）编码，音频侧用 CosyVoice2 audio tokenizer；再接 spatial transformer 做逐帧精修、diffusion MLP 做视觉细节增强。可训练参数 0.8B。训练中引入 noise injection 策略缓解自回归的误差累积。
该数据集被下游多个音视频联合生成模型直接采用为训练语料，典型如 MOVA 将 SpeakerVid-5M 列为其 Phase 1 数据来源之一，并称其为唇同步（lip-sync）能力的核心来源。

### [Step-Video-T2V](../models/Step-Video-T2V.md)

不支持。Step-Video-T2V 与 Step-Video-TI2V 均为纯视觉视频生成模型，输出无音轨；技术报告全文不涉及音频模态，数据 pipeline 中也不含任何音轨处理环节（切分后仅保留视觉帧）。
阶跃星辰的音频能力由完全独立的模型线承担：与 Step-Video-T2V 同日开源的 Step-Audio（产品级开源语音交互模型，支持情绪、方言、语种、歌声与个性化风格），以及后续的 Step-Audio 系列。二者之间没有联合训练、没有共享 latent、没有联合去噪，属于典型的「视频模型 + 语音模型」双线并行而非级联或原生联合的 AV 生成。
因此本条目在音视频联合生成维度上不构成参考样本，本调研中所有音频相关字段（audio_category_distribution、joint_av_caption_schema、dialogue_transcription_attributes、av_sync_detection、sync_metric_and_threshold、temporal_vs_semantic_sync、audio_quality_filtering、audio_type_handling）均不适用。

### [UniTalking](../models/UniTalking.md)

支持，为原生端到端联合生成（单次扩散推理同时产出视频与语音），实现方式是「对称孪生双流 + 联合注意力」：
【架构】基于 MM-DiT（Multi-Modal Diffusion Transformer），骨干为 N=30 个 MM-DiT block，模型维度 dim=3072，注意力头数 24，全模型参数量 10B。以 Flow Matching 训练为连续归一化流（CNF），推理用 UniPC 求解器，并采用 Classifier-Free Guidance。
【双流初始化的不对称设计】视频流直接继承 Wan2.2-5B 的架构与权重，作为强视觉先验；音频流被设计为视频流的「完全相同的孪生体」（identical twin）——架构一模一样但参数随机初始化，目的是让两个模态天然处于同构表示空间以促进 latent 层融合。这是与 UniVerse-1「缝合两个异构专家（Wan2.1 + Ace-step）再做层插值对齐」路线的根本差异：UniTalking 用「结构对称 + 从零训练音频塔」换取融合的简洁性，代价是音频塔缺乏预训练先验（因此必须补一个单独的音频预训练阶段）。
【融合机制：Joint Attention】把视频与音频的 latent token 拼接（concatenate）后送入单一注意力操作，让模型在一次 attention 内同时学习模态内与模态间依赖，显式建模 viseme（视素）与 phoneme（音素）的时间对应。这是「共享自注意力」而非 UniVerse-1/JavisDiT 式的跨模态 cross-attention。
【条件注入：Cross Attention】文本条件（UMT5 编码）与参考音色条件（MMAudio VAE 编码）分别经各自的 key-value 投影层做 cross-attention，两路输出以逐元素求和融合。为参考音频专设一套 KV 投影层是本工作在条件注入上的具体设计。
【各向异性 RoPE】沿用 OVI 的做法，时间轴 t 使用标准 RoPE；但音频 token 的空间维度 (h, w) 使用来自单一固定位置的 RoPE。这一「空间退化」设计强制模型把注意力预算集中在时间维度上，从架构层面偏置向音视频时序对齐。
【支持任务】T2AV（文本→音视频）、TI2AV（文本+身份图像→音视频）、TR2AV（文本+参考音色→音视频）、TV2A（文本+视频→音频，训练时用作单向监督信号）。

### [UniVerse-1](../models/UniVerse-1.md)

支持，为原生联合生成（单次推理同时产出视频与音频），但实现路径是本工作的核心创新——不从零训练联合模型，而是把两个已预训练的单模态专家「缝合」（Stitching of Experts, SoE）：
【专家来源】视频专家 Wan2.1（1.3B DiT + 3D VAE + umT5 文本编码器）；音频专家 Ace-step（3.5B 音乐生成模型，含 Music-DCAE + umT5 + 歌词编码器 + 说话人编码器 + DiT）。合并后模型总规模约 7B。
【融合方式】在两个 DiT 的对应 block 层级做深度融合，而非仅在输入/输出端拼接。融合件是轻量级跨模态 MLP connector：两层线性 adapter 做特征空间对齐，并为跨模态注意力配置专用的 key（kproj）与 value（vproj）投影层。
【深度对齐问题】两个专家的 transformer 层数不同，采用层插值（layer interpolation）解决——按均匀间隔插入新 block，新 block 权重由相邻层权重线性插值初始化，使两塔深度可一一对应缝合。
【结构改造】移除了 Ace-step 原有的 speaker encoder，目的是让模型摆脱说话人特定生成的约束、泛化到任意说话人（音色由参考图像与文本共同隐式决定）。
【时间栅格对齐】将音频采样率从原始 44.1 kHz 调整为 25.6 kHz，使音频 latent 的时间栅格与视频 25 fps 严格对齐——这是一个用采样率迁就视频帧率的工程取舍，与 MOVA 用 Aligned RoPE 做索引缩放的思路不同。
【VAE】视频 3D VAE 将 (3,T,H,W) 压缩为 (16,T/4,H/8,W/8)；音频 Music-DCAE 将 mel 频谱 (8,T,F) 压缩为 (8,T/8,F/8)。
【噪声采样】提出独立噪声采样策略（Independent Noise Sampling Strategy, INSS），避免两个模态共享 PRNG 状态时产生虚假的跨模态噪声相关性。
【任务形态】参考图 + 文本 → 视频 + 音频（IT2VA）；Verse-Bench 同时支持联合生成、audio-to-video、video-to-audio、TTS 四类任务的评测。

### [Unison](../models/Unison.md)

支持音视频同时生成，实现方式为「双分支原生联合生成」——两个专家模型经帧级双向交叉注意力耦合，属于 UniVerse-1「专家缝合（Stitching of Experts）」范式的延续与深化，而非从零训练的单塔统一模型，也非级联。
【分支构成】
- 视频分支：基于 Wan2.2-5B，29 层（L_v = 29）。联合训练阶段视频骨干完全冻结。
- 音频分支：基于 MMAudio，23 层（L_a = 23），并引入 Zipformer（来自 ZipVoice 系）以获得语音生成能力——MMAudio 原本是 foley/音效生成模型，不具备语音合成能力，Zipformer 的引入正是为补齐这一短板。
【耦合方式】帧级双向交叉注意力（frame-level bidirectional cross-attention），视频 latent 与音频 latent 互为 query，实现双向信息交换。这与 UniAVGen、Harmony 等单向条件化（unidirectional conditioning）方案形成对比，论文将「双向」列为核心区分点。
【唇同步的三层保障机制】论文明确把词级唇同步（word-level lip-sync）拆解到三个层面同时施加约束，这是理解 Unison 数据侧设计的关键：
1) 架构层：交叉注意力在三帧窗口内对齐特征，stride=1，恢复时仅保留中间帧的表征——一个极短时窗的局部对齐设计；
2) 数据层：lip-filtering 算子（先检测人脸数量与位置，再仅在人脸框内运行 SyncNet 核验对齐）；
3) 训练层：双向跨模态 forcing 策略（CMFS）。
【音频侧的双流解耦——本工作最独特处】音频 latent 不是单一张量，而是 [speech, sfx] 双流张量。源音频经 Mel-RoFormer 分离为语音与音效两路，各自编码为时序对齐的独立序列，两路共用相同的 RoPE 时间索引以保证严格时间对齐，并用「模态特定可学习偏置」在共享自注意力层内区分二者。每个 Transformer block 内执行「交互—合并—拆分」（interact-merge-split）循环，出口处重新分离为两条独立生成轨迹，最终由两个独立的 flow-matching 损失分别监督。这意味着 Unison 在生成阶段就物理隔离了语音与音效，从源头消除「语音压制环境音」的问题——这是与 Ovi/LTX-2/MOVA 等单一混合音轨方案的根本差异。
【跨模态 forcing】视频与音频的扩散 timestep 独立采样（音频分支映射到受限区间），噪声更低（更「干净」）的模态引导更「脏」的模态，通过方向指示器动态指定「学生模态」并上调其损失权重。三阶段课程（同步 warmup → 增量解耦 → 完全独立，阶段比例 0.3/0.4/0.3）保证训练稳定。
【推理配置】50 步 flow-matching 采样器，CFG scale = 6.0，输出 25 FPS 视频。

### [Veo 3 / Veo 3.1](../models/Veo_3_Veo_31.md)

支持，且为原生联合生成（native joint generation），不是级联 pipeline。官方技术报告原文：「Veo 3 uses latent diffusion, in which the diffusion process is applied jointly to the temporal audio latents, and the spatio-temporal video latents.」即视频与音频分别由各自的 autoencoder 编码为压缩潜表示（视频为时空潜变量 spatio-temporal latents，音频为时序潜变量 temporal latents），随后一个基于 Transformer 的去噪网络在同一个扩散过程中对两类 latent 联合去噪，每一步 denoising 同时处理音视频 token，从而在生成阶段天然保证同步。生成内容涵盖对白（带唇形同步）、音效/foley、环境音与背景音乐。未采用 MoE 融合或先视频后配音的两阶段级联方案。

### [Vidu S1](../models/Vidu_S1.md)

支持，且为原生联合生成（native joint）。模型将第 i 帧的干净视频表示 v_0^i 与音频表示 a_0^i 沿模态维度拼接为联合状态 x_0^i = [v_0^i; a_0^i]，在同一扩散去噪模型中对视频-音频联合潜在序列统一去噪，非级联、非 MoE。统一条件接口 c 同时包含语音（speech）、文本提示、参考首帧图像——即语音既是条件控制信号（用户实时语音指令控制角色行为），联合状态中又包含音频轨道的生成。整体为自回归 + 扩散（AR+Diffusion）的因果流式生成范式，滑动窗口解码支持无限长生成。

### [Wan 2.5 / 2.6 / 2.7](../models/Wan.md) ⚠️

支持，且是 2.5 版本的分水岭能力。官方口径为「原生音画同步」，但缺少论文级证据。
【能力演进】
- Wan 2.1（2025.02）：音频为独立的级联 V2A 模块（技术报告 5.7 节），先生成视频再配音；且明确只产环境音与背景音乐，显式排除语音/人声演唱。
- Wan2.2-S2V（2025.08）：音频驱动人物动画的专用模型（audio→video），属条件生成而非联合生成。
- Wan 2.5-preview（2025.09）：官方能力标签首次出现「有声视频 / 声画同步」，T2V 与 I2V 均可在不给音频时自动配音（生成匹配的背景音乐或音效），也可通过 input.audio_url 传入音频驱动生成。唇同步能力自此成为系列卖点。
- Wan 2.6（2025.12）：能力标签升级为「多镜头叙事 + 声画同步」，官方称「原生音画同步，画面与人声、音效、BGM 完美匹配」，并新增「声音驱动」模式（音频直接驱动角色口型与表演）与「角色扮演」（上传个人视频复刻形象与声音）。官方自称「国内首个支持角色扮演的视频生成模型」「全球功能最全的视频生成模型」。
- Wan 2.7（2026.06）：延续「多镜头叙事 + 声画同步」，音频作为一等输入模态（input.audio_url，wav/mp3，2～30秒，≤15MB），I2V 侧输入模态扩展为「文本+图像+音频+视频」（支持首帧、首尾帧、视频续写、续写+尾帧控制）。这与调研 note 中「2.7 加入 native audio conditioning」的说法一致——音频从「输出侧配音」变为「输入侧条件」。
【实现方式】官方宣传语为「阿里自研的原生多模态架构」，指向原生联合生成而非级联；但无任何论文、架构图或参数披露可证实，也无法判断是否采用 MoE 融合（Wan 2.2 的 MoE 是高噪/低噪双专家的去噪阶段划分，与模态无关）。[不确定：具体融合机制]
【输出规格】wan2.5-*-preview：480P/720P/1080P，5s 或 10s；wan2.6-*：720P/1080P，2–15s 整数；wan2.7-*：720P/1080P，2–15s 整数，ratio 支持 16:9/9:16/1:1/4:3/3:4（如 1080P 16:9 = 1920×1080，4:3 = 1648×1248）；全系固定 30fps、MP4/H.264。

### [音视频生成评测基准合集](../models/av_benchmarks.md)

五者本身均不生成音视频，而是评测音视频联合生成能力。它们覆盖的被测系统形态恰好构成了当前 AV 生成的三条技术路线，评测设计上明确区分：
1) 原生联合生成（端到端 T2AV/I2AV）：Sora 2、Veo 3 / Veo 3.1 / Veo3-Fast、Wan 2.5 Preview / Wan 2.6、Kling 2.5 Turbo / Kling v2.6、Seedance 1.5 Pro，开源侧 Ovi、LTX、MOVA、UniVerse-1、JavisDiT / JavisDiT++；
2) 级联组合（V+A，先生成视频再配音）：视频端 Seedance-1.0-Lite / Wan2.2-TI2V / Kling2.5 Turbo，音频端 MMAudio、ThinkSound(-Light)、HunyuanVideo-Foley、FoleyCrafter；
3) 表征/判别模型（AV-SyncBench 的被测对象）：Synchformer、SparseSync、ImageBind、CAV-MAE、CAV-MAE-Sync。
VABench 额外引入「立体声音视频生成」这一路线，用 116 条显式指定左右声道方位的提示词考察空间音频生成能力，是目前少见的立体声 AV 评测维度。

### [视频 Caption 模型生态](../models/caption_models.md)

captioner 本身不做生成，此字段在本条目按「是否支持音视频双模态输入并输出联合描述」重新诠释，这是 2025–2026 年该生态最关键的能力分水岭：
【纯视觉 captioner（不听声音）】ShareCaptioner-Video、Tarsier / Tarsier2、CogVLM2-Caption、SkyCaptioner-V1、AuroraCap、LLaVA-Video、PLLaVA、Aria、Qwen2.5-VL / Qwen3-VL、Panda-70M 的全部 31 个候选打标器（Panda-70M 论文明确说明 Video-LLaMA 分支「仅用视觉分支、音频分支明确关闭」）。此档占据 2024–2025 上半年的绝对主流。
【级联式音视频标注（多模型分工后由 LLM 融合）】主流工程做法，代表：MOVA = MiMo-VL-7B-RL（视觉）+ Qwen3-Omni-Instruct（ASR）+ Qwen3-Omni-Captioner（非语音）+ GPT-OSS-120B（融合与一致性校验）；Movie Gen 音频侧 = 音频质量预测 + AED + 通用音频 caption + 音乐 caption 四模型协同；UniTalking = Qwen3-VL + Whisper-V3 + Qwen3-Omni-Captioner + Qwen3-Omni；Kling-Foley = 音频分类 + 音频理解大模型 + LLM 融合三段式。
【原生联合音视频 captioner（单模型同时看+听）】2025Q4 起成型的新范式：AVoCaDO（Qwen2.5-Omni-7B 基座，~9B 全栈）、AVSCap-7B（同基座）、video-SALMONN 2（LLaVA-OneVision + 音频 LoRA，3B/7B/72B）、Qwen3-Omni / Qwen3.5-Omni、以及 Lightricks 为 LTX-2 自研的未具名系统。UniVerse-1 更极端，用单个 Qwen2.5-Omni 一次性并列输出语音内容/视频 caption/环境音 caption 三路。
【关键实证】裸 Qwen2.5-Omni 的零样本打标能力很差（AVSCapBench overall 仅 21.53、Speech 13.92），必须经 caption 专项 SFT+RL 才能当打标器用（AVoCaDO 49.31、AVSCap 60.44）——「有 omni 基座 ≠ 能当 omni 打标器」是本生态最重要的工程教训。

### [几何/结构化标注数据集合集](../models/geometric_datasets.md)

均不支持音视频同时生成，也不涉及音频模态。四者是纯视觉几何/结构化标注范式：SceneScribe-1M 与 SpatialVID 面向相机可控视频生成与3D/4D感知，WildWorld 面向动作条件世界模型，Action100M 面向动作理解与视频-文本表征学习。Action100M 源自 HowTo100M（含旁白ASR），但ASR仅用于文本监督辅助，不做音频生成

### [视频生成后训练数据策略](../models/post_training_data.md)

本专题不是模型本身，但覆盖的对象含大量音视频联合生成模型，其后训练呈现明显分层：
【已把 AV 纳入后训练奖励的】Seedance 1.5 pro——明确采用「专为音视频场景定制的 RLHF 算法」与多维奖励模型，同时优化运动质量、视觉美学与音频保真度，并对 RLHF 流水线做基础设施优化带来近 3 倍训练加速；Kling 3.0 Omni——对同一 MVL（多模态视觉语言）条件采样多个视频变体，由人类评估者比较形成偏好对做 DPO（但音频维度是否作为独立打分项未披露）；JavisDiT++ 的 AV-DPO——六个奖励模型分工，其中时序同步性由 Synchformer 承担、音频质量由 AudioBox-Aesthetics 承担、文本-音频与跨模态相似度由 ImageBind 承担，是目前唯一完整披露 AV 偏好对构造细节的工作。
【锚论文的 AV 处理】仅在自回归蒸馏阶段涉及：对具备音视频生成能力的模型，遵循 OmniForcing（arXiv:2603.11647）为模型配备「非对称块因果对齐（asymmetric block-causal alignment）」与「音频 sink token」。即 AV 只体现在蒸馏架构层面，其 SFT 与 GRPO 的四个奖励模型（视频美学/图像美学/运动质量/文本-视频对齐）全部是纯视觉维度，不含任何音频或音视频同步奖励——这是该框架在 AV 时代的显著缺口。
【完全空白的】HunyuanVideo-Foley、Ovi（初版）、UniVerse-1、UniTalking、Unison、Foley-Omni、InstructAV2AV 等学术 AV 工作全部无偏好对齐后训练。

### [主流视频预训练数据集合并调研：Panda-70M、InternVid、Koala…](../models/pretraining_datasets.md)

七个数据集**全部不支持音视频同时生成**，均为纯视觉+文本数据集。音频处理情况分三档：(1) 完全不涉及——InternVid、Koala-36M、MiraData、OpenVid-1M、LVD-2M 的论文中「audio」一词仅出现在参考文献标题里（Koala-36M 与 LVD-2M 经全文 grep 确认各仅1次，均在文献题名中），元数据无任何音频字段；(2) 音轨被动保留但未处理——Panda-70M 的下载配置 panda70m.yaml 设 download_audio: True，用户拿到的 mp4 混流了原始 YouTube 音轨，但数据集本身不含任何音频标注、音频质检或语音标签，且其教师模型 Video-LLaMA 被明确关闭了音频分支（「We only use the vision branch」）；UltraVideo 保留原生音轨且在 Limitations 中提及「thanks to the preservation of native resolution, frame rate, and audio…可用于音乐生成等任务」，但同样无任何音频侧的筛选或标注；(3) Koala-36M 的重切分代码用 cv2.VideoWriter/mp4v 输出，会**直接丢弃音轨**。因此本条目下所有音频与音视频对齐字段（audio_category_distribution、joint_av_caption_schema、dialogue_transcription_attributes、av_sync_detection、sync_metric_and_threshold、temporal_vs_semantic_sync、audio_quality_filtering、audio_type_handling）均为「不适用」。

## 调研信息来源列表（论文/技术报告/官方文档/新闻的URL，标注每条来源的性质：官方一手/同团队旁证/第三方报道）

`sources` · 详细程度: brief

### [Allegro](../models/Allegro.md)

1. https://arxiv.org/abs/2410.15458 —— 官方一手，Allegro 论文《Allegro: Open the Black Box of Commercial-Level Video Generation Model》，2024-10-20 v1，数据章节（Sec. 2 Data：Data Filtering / Data Annotation / Data Stratification，Table 1 阈值表，Appendix A 数据分布图 Fig.11）为本条目全部定量信息的主要来源。
2. https://arxiv.org/html/2410.15458v1 —— 官方一手，论文 HTML 全文，含 Table 1（分阶段过滤阈值）、Table 训练阶段配置（分辨率/帧数/样本数/batch/步数/GPU 数）、Appendix A 分布统计。
3. https://github.com/rhymes-ai/Allegro —— 官方一手，代码仓库与 README，含 Apache 2.0 许可、各版本发布时间线、模型规格与训练代码。
4. https://huggingface.co/rhymes-ai/Allegro —— 官方一手，T2V 权重卡片。
5. https://huggingface.co/rhymes-ai/Allegro-TI2V —— 官方一手，TI2V 权重卡片（2024-11-25）。
6. https://huggingface.co/blog/RhymesAI/allegro —— 官方一手，Rhymes AI 官方博客《Allegro: Advanced Video Generation Model》。
7. https://arxiv.org/abs/2410.05993 —— 同团队旁证，Aria 多模态 MoE 模型论文（打标模型的基座，25.3B 总参 / 3.9B 视觉 token 激活，支持 64K 多模态输入，10 秒内 caption 256 帧视频）。
8. https://huggingface.co/papers/2410.15458 —— 第三方聚合，Hugging Face 论文页与社区讨论。
9. https://www.marktechpost.com/2024/11/28/rhymes-ai-unveils-allegro-ti2v-a-breakthrough-in-visual-storytelling-with-open-source-ai-video-generation-technology/ —— 第三方报道，Allegro-TI2V 发布报道。

### [Apollo](../models/Apollo.md)

- 【官方一手】arXiv:2601.04151《Apollo: Unified Multi-Task Audio-Video Joint Generation》（v1，2026-01-07）/《Klear: Unified Multi-Task Audio-Video Joint Generation》（v2，2026-01-13）：摘要页 https://arxiv.org/abs/2601.04151 、HTML 全文 https://arxiv.org/html/2601.04151v2 与 https://arxiv.org/html/2601.04151v1 、PDF https://arxiv.org/pdf/2601.04151 —— 本条目几乎全部字段的唯一直接来源，数据相关内容集中在第 4 节 Dataset Construction（4.1 Dataset Filtering、4.2 Audio-Guided Data Splitting、4.3 Dense Annotation and Integration）与 Figure 3 数据标注 pipeline 图。该节篇幅极短（约一页），是本条目大量字段标为不确定的根本原因。
- 【官方一手】HuggingFace 论文页 https://huggingface.co/papers/2601.04151 —— 确认署名单位为「Kling Team, Kuaishou Technology」，确认未挂载任何模型、数据集或代码仓库链接。
- 【同团队旁证】快手投资者关系公告《Kling AI Launches Video 2.6 Model with 「Simultaneous Audio-Visual Generation」 Capability》：https://ir.kuaishou.com/news-releases/news-release-details/kling-ai-launches-video-26-model-simultaneous-audio-visual —— 用于印证该研究与可灵产品线音画同出能力的对应关系（旁证，非论文明示）。
- 【第三方索引】NASA ADS 条目 https://ui.adsabs.harvard.edu/abs/2026arXiv260104151W/abstract 、ResearchGate 条目 https://www.researchgate.net/publication/399559825_Klear_Unified_Multi-Task_Audio-Video_Joint_Generation —— 用于交叉核对 Apollo→Klear 的改名事实与作者列表。

### [CineDance / CineDance-1M](../models/CineDance.md)

1. 【官方一手】论文 arXiv 摘要页 https://arxiv.org/abs/2606.09639 —— 标题、作者、v1/v2 提交日期、摘要。
2. 【官方一手】论文全文 HTML https://arxiv.org/html/2606.09639v2 —— 三阶段 pipeline、漏斗表（Tab.3）、标注 schema、CineBench 六维度、消融表（Tab.4/5/7）、数据集横向对比表（Tab.6）。
3. 【官方一手】论文 PDF https://arxiv.org/pdf/2606.09639
4. 【官方一手】项目主页 https://aliothchen.github.io/projects/CineDance/ —— 五家参与机构、数据集规模概述、各资源入口。
5. 【官方一手】GitHub 仓库 https://github.com/AliothChen/CineDance —— 开源进度 checklist（数据集 gated 分批释出、代码与权重 pending）。
6. 【官方一手】HuggingFace 数据集卡 https://huggingface.co/datasets/CineDance/CineDance_01 —— 首批分片规模 5.83TB / 240,488 clips / 150 TAR、CC-BY-NC-SA-4.0 许可、gated 访问、video-only 现状、局限性声明。
7. 【第三方索引】awesome-video-generation 列表 https://github.com/kongzhecn/awesome-video-generation —— 收录旁证。

### [CogVideoX](../models/CogVideoX.md)

- 官方一手: CogVideoX: Text-to-Video Diffusion Models with An Expert Transformer (arXiv:2408.06072v3, ICLR 2025, 30页含附录G/J/K) https://arxiv.org/abs/2408.06072 与 https://arxiv.org/pdf/2408.06072
- 官方一手: zai-org/CogVideo 开源仓库（代码、权重、caption 工具链） https://github.com/zai-org/CogVideo
- 官方一手: CogVLM2-Caption 打标模型权重（CogVideoX 训练中使用的视频 caption 模型） https://huggingface.co/zai-org/cogvlm2-llama3-caption
- 同团队旁证: 智谱「新清影」CogVideoX+CogSound 技术详解（智源社区转载智谱官方技术解读，含 CogSound 分块时序对齐交叉注意力与数据筛选框架描述） https://hub.baai.ac.cn/view/40956
- 第三方报道: 智谱视频生成大模型清影升级（新清影 10s/4K/60帧/自带音效，2024-11-08） https://cn.technode.com/post/2024-11-08/zhipu-qingying-new/
- 第三方报道: CogSound 音效模型能力介绍 https://ai-bot.cn/cogsound/
- 第三方整理: CogVideoX 论文文献综述（Moonlight，含数据过滤与 35M clips 说明） https://www.themoonlight.io/en/review/cogvideox-text-to-video-diffusion-models-with-an-expert-transformer

### [Cosmos-Predict2.5](../models/Cosmos-Predict25.md)

1. arXiv:2511.00062《World Simulation with Video Foundation Models for Physical AI》 https://arxiv.org/abs/2511.00062 ，全文 PDF https://arxiv.org/pdf/2511.00062v2 —— 官方一手，核心依据。本调研的七阶段 pipeline、35M 小时/6B clip/4% 保留率/200M clip、各过滤器、Qwen2.5-VL-7B captioning、语义去重、26 类 taxonomy 分片、五大领域数据、SFT 五域规模表、RL 与蒸馏配置、基础设施与 MFU 数据均逐字出自 v2 版第 2 节 Data、第 4 节 Training 与第 6 节 Applications。
2. GitHub 官方仓库 https://github.com/nvidia-cosmos/cosmos-predict2.5 与 https://github.com/nvidia-cosmos/cosmos-transfer2.5 —— 官方一手（代码 Apache 2.0、模型 NVIDIA Open Model License、发布模型清单、guardrails 提及）。
3. Hugging Face 模型卡 https://huggingface.co/nvidia/Cosmos-Predict2.5-2B 与 https://huggingface.co/nvidia/Cosmos-Predict2.5-14B —— 官方一手（权重、许可、能力说明）。
4. NVIDIA Research Cosmos Lab 项目页 https://research.nvidia.com/labs/cosmos-lab/cosmos-predict2.5/ —— 官方一手（200M 高质量预训练 clip、模型合并与 RL 的对外口径）。
5. NVIDIA Cosmos Curator 开源仓库 https://github.com/NVIDIA/cosmos-curator 与文档 https://docs.nvidia.com/cosmos-curator-lha/current/introduction.html —— 官方一手，同团队旁证（论文所述 curation pipeline 的产品化实现，GPU 流式框架 Cosmos-Xenna、镜头切分/embedding/caption 各阶段的工程细节）。
6. NeMo Curator 视频 curation 文档 https://docs.nvidia.com/nemo/curator/curate-video 与 https://docs.nvidia.com/nemo/curator/latest/get-started/video.html —— 官方一手，旁证（GPU 加速视频 pipeline 能力、2000 块 H100 一天处理约 100 万小时 720p 视频的吞吐口径）。
7. 前代论文 NVIDIA (2025)《Cosmos World Foundation Model Platform for Physical AI》 —— 官方一手，旁证（20M 小时、30% 存活率的对照基线，以及 Cosmos-Guardrail 护栏体系）。
8. Emergent Mind 主题页 https://www.emergentmind.com/topics/cosmos-predict2-5 、DL 轮读会讲解 https://www.docswell.com/s/DeepLearning2023/KPGPD6-2025-11-17-152333 —— 第三方索引与解读（用于交叉核对数字，非主要依据）。

### [Data-Juicer 2.0](../models/Data-Juicer.md)

1. 【官方一手】Data-Juicer 2.0 论文 arXiv 摘要页 https://arxiv.org/abs/2501.14755 —— 作者、版本历史（v1 2024-12-23 / v2 2025-06-04 / v3 2025-10-29）、NeurIPS 2025 Spotlight。
2. 【官方一手】Data-Juicer 2.0 全文 HTML https://arxiv.org/html/2501.14755v3 —— 三层架构、150+ 多模态算子分类、TB级/10k+ CPU核规模、MinHash Ray 去重性能、加速比数据、阿里云 PAI 落地。
3. 【官方一手】Data-Juicer 2.0 PDF https://arxiv.org/pdf/2501.14755
4. 【官方一手】Data-Juicer Sandbox 论文 arXiv 摘要页 https://arxiv.org/abs/2407.11784 —— ICML 2025 Spotlight、Probe-Analyze-Refine、VBench 登顶。
5. 【官方一手】Sandbox 全文 HTML https://arxiv.org/html/2407.11784v3 —— 文生视频案例、数据池切分、算子选择、VBench 分数表、质量vs算力结论。
6. 【官方一手】GitHub 主仓库 https://github.com/datajuicer/data-juicer —— 版本时间线、算子统计、采用方名单、关联论文列表、Apache 2.0 许可。
7. 【官方一手】算子提要文档 https://datajuicer.github.io/data-juicer/en/main/docs/Operators.html 与 https://raw.githubusercontent.com/datajuicer/data-juicer/main/docs/Operators.md —— 229个算子的逐条名称与说明，视频/音频算子清单的权威来源。
8. 【官方一手】HuggingFace 数据集 https://huggingface.co/datasets/datajuicer/data-juicer-t2v-optimal-data-pool —— 147,176条样本、12.09%保留率、227.5GB、来源数据集构成（InternVid 606k / Panda-70M 605k / MSR-VTT 6k）、两个过滤算子及 CLIP 阈值 0.306337。
9. 【官方一手】OpenReview 评审页 https://openreview.net/forum?id=NiL5U1DrRN （DJ 2.0）与 https://openreview.net/forum?id=zIGIvysR1H （Sandbox）。
10. 【同团队旁证】阿里云开发者社区文章《VBench 视频生成新榜首！Data-Juicer 沙盒实验室助力多模态数据与模型协同开发》 https://developer.aliyun.com/article/1570605 及魔搭社区同文 https://community.modelscope.cn/669f1a7b76e87a79e35ada49.html —— 中文版案例说明。
11. 【同团队旁证】HumanVBench 论文 https://arxiv.org/html/2412.17574v2 （CVPR 2026）—— 使用 DJ 20+算子构建人物中心视频标注pipeline，是 DJ 视频算子在真实数据集构建中的用例。
12. 【官方一手】ICML 2025 Poster 页 https://icml.cc/virtual/2025/poster/43484 与 PMLR 正式版 https://proceedings.mlr.press/v267/chen25bm.html
13. 【第三方索引】ResearchGate https://www.researchgate.net/publication/388421863_Data-Juicer_20_Cloud-Scale_Adaptive_Data_Processing_for_Foundation_Models
14. 【第三方报道】CSDN 博客《Data-Juicer：阿里巴巴荣誉出品的大模型数据清洗框架》 https://blog.csdn.net/qq_41895747/article/details/140150556 —— 中文社区解读，无新增一手数据。

### [Foley-Omni](../models/Foley-Omni.md)

1. 【官方一手】arXiv 论文摘要页 https://arxiv.org/abs/2606.03672 —— 标题、作者、摘要、2026年6月2日提交日期。
2. 【官方一手】arXiv HTML 全文 https://arxiv.org/html/2606.03672v1 —— 第3.1节数据清洗pipeline、Table 7 过滤阈值、Table 9 训练数据构成、Table 6 消融、V2ST-Bench 主结果表、附录 B/C.1。核心信息来源。
3. 【官方一手】arXiv PDF https://arxiv.org/pdf/2606.03672
4. 【官方一手】HuggingFace 模型页 https://huggingface.co/CocoBro/Foley-Omni —— 5.5B 参数量、MIT 许可证、inference-only 权重构成、上游依赖声明。
5. 【官方一手】GitHub 仓库 https://github.com/NJU-Speech/Foley-Omni —— 推理代码、特征抽取脚本、V2ST-Bench「Coming soon」状态。
6. 【官方一手】项目主页 https://ty0402.github.io/Foley-omni-Web/ —— demo 样例与项目介绍。
7. 【第三方索引】ResearchGate 收录页 https://www.researchgate.net/publication/405852241_Foley-Omni_A_Unified_Multimodal_Generation_Model_from_Task-Level_Audio_Synthesis_to_Complete_Video_Soundtrack_Generation
8. 【第三方报道】AI Films Studio 博客 https://studio.aifilms.ai/blog/foley-omni-video-soundtrack-generation —— 面向创作者的通俗解读，无新增一手数据。

### [Goku](../models/Goku.md)

1. 【官方一手】https://arxiv.org/abs/2502.04896 — Goku 论文 arXiv 摘要页（v1 2025-02-07，v2 2025-02-10）。
2. 【官方一手】https://arxiv.org/html/2502.04896v2 — 论文 HTML 全文，第4节 Data Curation Pipeline 为本次调研的主要依据，含 Table 4 各分辨率过滤阈值、Figure 3 语义类别分布图。
3. 【官方一手】https://github.com/Saiyan-World/goku — 官方 GitHub 仓库与 README（VBench 完整对比表、BibTeX、作者与机构署名 HKU, ByteDance）。
4. 【官方一手】https://saiyan-world.github.io/goku/ — 项目主页，生成样例可视化。
5. 【第三方聚合】https://huggingface.co/papers/2502.04896 — HuggingFace Papers 页面，社区讨论与投票。
6. 【第三方报道】https://www.etcentric.org/bytedances-goku-video-model-is-latest-in-chinese-ai-streak/ 、https://stable-learn.com/en/goku-video-model-introduction/ 、https://www.analyticsvidhya.com/blog/2025/02/goku-ai/ — 媒体解读，用于交叉印证模型规模与定位，数据细节不以其为准。

### [Hailuo / MiniMax Video](../models/Hailuo.md)

调研时点：2026年7月29日。本对象无论文、无技术报告，全部信息来自产品博客、API 文档与第三方托管平台，数据侧披露极其有限。
1) 官方一手：MiniMax Hailuo 02 发布公告 https://www.minimax.io/news/minimax-hailuo-02 （2025年6月18日；本调研中唯一提及训练数据规模的来源——参数量3倍、训练数据4倍、质量与多样性提升；NCR 架构效率2.5倍；原生1080p；768p-6s/768p-10s/1080p-6s 三档；Artificial Analysis Video Arena 全球第二）
2) 官方一手：MiniMax Hailuo 2.3 / 2.3 Fast 发布公告 https://www.minimax.io/news/minimax-hailuo-23 （2025年10月28日；肢体运动、风格化含水墨与游戏CG、微表情、运动指令响应；Video Agent 升级为 Media Agent；2.3 Fast 批量成本降低最多50%；明确不含分辨率、时长、音频、训练数据与架构细节）
3) 官方一手：MiniMax 开放平台视频生成文档 https://platform.minimax.io/docs/guides/video-generation （模型ID：MiniMax-Hailuo-2.3 支持 T2V/I2V、MiniMax-Hailuo-02 支持首尾帧、S2V-01 支持主体参考；1080P/6秒示例；API 无任何音频参数）
4) 官方一手：海螺AI 产品站 https://hailuoai.video/ （视频/图像/音频为并列独立入口，佐证音频与视频非同一模型；模板类目；AI生成内容合规提示）
5) 官方一手/同团队旁证：MiniMax HuggingFace 组织页 https://huggingface.co/MiniMaxAI （截至2026年7月开源模型为 MiniMax-M3(427B)、M3-MXFP8、M2.7/M2.5/M2.1/M2(229B)、M1-40k-hf(456B) 等语言模型及 VTP 系列视觉tokenizer；无任何视频生成模型开源，是「视频线完全闭源」的关键佐证）
6) 同团队旁证：VTP 视觉 tokenizer 模型卡 https://huggingface.co/MiniMaxAI/VTP-Large-f16d64 与论文《Towards Scalable Pre-training of Visual Tokenizers for Generation》 https://arxiv.org/abs/2512.13687 （2025年12月15日，Modified MIT；图文对比+自监督+重建三损失联合优化的视觉tokenizer；模型卡未说明与 Hailuo 视频模型的直接关系，训练数据亦未明确列出）
7) 第三方托管平台：Replicate MiniMax 模型列表 https://replicate.com/minimax （列出 hailuo-2.3、hailuo-2.3-fast、hailuo-02、hailuo-02-fast、video-01、video-01-live、video-01-director 共7个视频模型及运行量；关键佐证：全部模型均未描述随视频产出音轨的能力）
注：本次调研的 WebSearch 配额在会话中已耗尽，上述信息全部通过对已知官方 URL 的直接抓取（WebFetch）获得，未能覆盖 Reddit、知乎、CSDN 等社区侧的非官方爆料与逆向分析，也未能穷尽检索 2026 年上半年可能存在的新版本公告。若后续需补充，建议优先检索方向：MiniMax 是否在 2026 年发布带原生音频的视频模型、NCR 架构是否有论文公开、以及中文社区（知乎/机器之心/量子位）对海螺视频数据来源的报道。

### [HunyuanVideo-Foley](../models/HunyuanVideo-Foley.md)

- 【官方一手】arXiv:2508.16930v1《HunyuanVideo-Foley: Multimodal Diffusion with Representation Alignment for High-Fidelity Foley Audio Generation》（2025-08-23 提交）：https://arxiv.org/abs/2508.16930 ，HTML 全文 https://arxiv.org/html/2508.16930v1 ，PDF https://arxiv.org/pdf/2508.16930 —— 本条目绝大多数字段的唯一直接来源，特别是第 3.1 节数据 pipeline、第 3.2 节架构、第 4 节实验与消融、附录 A.1（high-quality 标签策略）与 A.2（评测指标定义）。
- 【官方一手】GitHub 代码库 Tencent-Hunyuan/HunyuanVideo-Foley：https://github.com/Tencent-Hunyuan/HunyuanVideo-Foley —— 开源清单、发布时间线（2025-08-28 首发、2025-09-29 XL 版本）、显存需求表、依赖的预训练模型清单（SigLIP-2 / CLAP / Synchformer / DAC）。
- 【官方一手】HuggingFace 模型页 tencent/HunyuanVideo-Foley：https://huggingface.co/tencent/HunyuanVideo-Foley ，模型卡原文 https://huggingface.co/tencent/HunyuanVideo-Foley/blob/main/README.md —— 许可证类型（tencent-hunyuan-community）、欧盟地区限制（extra_gated_eu_disallowed）、XXL/XL 两档权重规格。
- 【官方一手】项目主页：https://szczesnys.github.io/hunyuanvideo-foley/ —— 演示样例与方法概览。
- 【官方一手】HuggingFace Papers 页：https://huggingface.co/papers/2508.16930 —— 摘要与三大贡献点（可扩展数据 pipeline / 表征对齐 REPA / 多模态 DiT）交叉核对。
- 【第三方聚合】alphaXiv 论文解析页：https://www.alphaxiv.org/abs/2508.16930 —— 用于交叉核对架构超参（18 MMDiT + 36 unimodal DiT、hidden 1536、12 heads）、REPA 消融表、DAC-VAE 重建指标表、三个 benchmark 主表数值。
- 【第三方部署】Replicate 托管页 tencent/hunyuanvideo-foley：https://replicate.com/tencent/hunyuanvideo-foley —— 用于确认推理接口形态（视频 + 文本 → 音频）与实际可用性。
- 【第三方中文报道】OSCHINA 新闻：https://www.oschina.net/news/368967 ；IT之家：https://www.ithome.com/0/878/633.htm ；AI工具集条目：https://ai-bot.cn/hunyuanvideo-foley/ —— 用于交叉核对约 30 亿参数量、七阶段 pipeline 的中文表述与开源时间点。参数量数字仅见于中文二手报道，论文与模型卡均未直接给出。
- 【第三方对照】Kling-Foley 论文 arXiv:2506.19774（快手可灵，2025-06）：https://arxiv.org/pdf/2506.19774 —— 提供 Kling-Audio-Eval 评测集，是本工作最主要的同期对照工作与评测基准来源。
- 【第三方对照】MMAudio、MovieGen-Audio-Bench（Meta Movie Gen）—— 主要基线与评测基准，用于横向对比数据策略与性能。

### [HunyuanVideo](../models/HunyuanVideo.md)

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

### [InstructAV2AV](../models/InstructAV2AV.md)

1. 【官方一手】arXiv 摘要页 https://arxiv.org/abs/2605.18467 —— 标题、作者、摘要、2026年5月18日提交日期。
2. 【官方一手】arXiv HTML 全文 https://arxiv.org/html/2605.18467v1 —— 第3节数据合成pipeline、方法设计、实验表格、消融、用户研究、局限性。核心信息来源。
3. 【官方一手】arXiv PDF https://arxiv.org/pdf/2605.18467
4. 【官方一手】项目主页 https://hjzheng.net/projects/InstructAV2AV/ —— 四类编辑任务 demo（身份保持语音修改 / AV实例编辑 / 实例插入 / 实例移除）、与 AvED、CoherentAVEdit、AVI-Edit 的对比样例、资源链接。
5. 【官方一手】GitHub 仓库 https://github.com/suimuc/InstructAV2AV —— Apache-2.0 许可、推理与pipeline脚本、六个子任务 checkpoint、训练脚本 roadmap 状态、上游依赖声明。
6. 【官方一手】HuggingFace 数据集 https://huggingface.co/datasets/suimu/InsAVE-80K —— 88,074 对、139 GB、11 分片、五类训练子集命名（add_and_remove / clone_id / clone_id_voice / clone_voice / general_editing）、CSV 字段结构（original_video / target_video / instruction / instruction_reverse）、<S>/<E> 语音标记、MIT 许可。这是补足论文未披露的数据组织细节的关键旁证来源。
7. 【官方一手】HuggingFace 模型 https://huggingface.co/suimu/InstructAV2AV —— 权重发布，但无 model card。
8. 【第三方报道】X/Twitter @wildmindai 帖文 https://x.com/wildmindai/status/2058634841013285372 —— 通俗概括为「Ovi + Wan2.2 + T5」技术栈与 mask-free 物体/声音替换能力，无新增一手数据。
9. 【第三方索引】The 500 Feed https://www.the500feed.com/story/e7f8eb37a6ffa09c、ai-search.io 工具页 https://ai-search.io/tool/instructav2av —— 聚合性介绍，无新增信息。
10. 【第三方】YouTube 演示视频 https://www.youtube.com/watch?v=0qp0B4jkWjE

### [2026 其他音视频联合生成](../models/JAVG_2026_misc.md)

【官方一手 · 论文】
1) Baton https://arxiv.org/abs/2605.25195 ／ https://arxiv.org/html/2605.25195v2 （数据信息集中在 Implementation Details）
2) OmniCustom https://arxiv.org/abs/2602.12304 ／ https://arxiv.org/html/2602.12304 （OmniCustom-1M 数据集构建章节为本条目最有价值的数据披露之一）
3) StreamChar https://arxiv.org/abs/2605.25659 ／ https://arxiv.org/html/2605.25659v1
4) ALIVE https://arxiv.org/abs/2602.08682 ／ https://arxiv.org/html/2602.08682v2 （Data 章节六阶段 pipeline，七项中数据披露最详尽）
5) CCL https://arxiv.org/abs/2603.18600 ／ https://arxiv.org/html/2603.18600v1 （Table 1 含训练数据量横向对比）
6) NAVA https://arxiv.org/abs/2605.30073 ／ https://arxiv.org/pdf/2605.30073 （Data 章节含 20M/100M→15M 漏斗与算子清单）
7) ITS-JAVG https://arxiv.org/abs/2606.03183 ／ https://arxiv.org/html/2606.03183v1
【官方一手 · 项目页/仓库】
8) NAVA 项目页 https://ernie-research.github.io/NAVA/ ；代码 https://github.com/ernie-research/NAVA ；权重 https://huggingface.co/baidu/NAVA 与 https://huggingface.co/ernie-research/NAVA （开源范围、许可、分辨率/时长、语言支持）
9) ALIVE 仓库 https://github.com/FoundationVision/Alive ；项目页 foundationvision.github.io（模型规模 VideoDiT 12B / AudioDiT 2B、Alive-Bench 1.0）
10) StreamChar 项目页 https://humanaigc.github.io/StreamChar_page/
11) OmniCustom 项目页与 GitHub（论文中给出）
【同团队旁证 / 聚合页】
12) HuggingFace Papers：https://huggingface.co/papers/2605.30073 （NAVA）、https://huggingface.co/papers/2605.25659 （StreamChar）
13) ResearchGate CCL 条目 https://www.researchgate.net/publication/402860562_Improving_Joint_Audio-Video_Generation_with_Cross-Modal_Context_Learning
【第三方】
14) ChatPaper StreamChar 解读 https://chatpaper.com/paper/286175 ；Takara TLDR CCL https://tldr.takara.ai/p/2603.18600 ；X/Twitter CS Vision Papers 对 NAVA 的推介（均为论文摘要复述，无新增一手信息）
【上游数据集一手来源（被这些工作复用）】
15) SpeakerVid-5M（OmniCustom 与 StreamChar 的共同上游）、TalkVid、OpenHumanVid（Baton/CCL/StreamChar 三方共用）、Koala-36M（NAVA 约占 20%）、AudioCaps、WavCaps、VGGSound、Emilia（StreamChar 语音预训练）

### [音视频联合生成基线合集](../models/JavisDiT_baselines.md)

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

### [可灵 3.0 / 可灵视频 3.0 Omni](../models/Kling_30_Omni.md)

- 官方一手: Kling-Omni Technical Report (arXiv:2512.16776, 三级数据体系/训练阶段/DPO) https://arxiv.org/abs/2512.16776
- 同团队旁证: Koala-36M (arXiv:2410.08260, 量化漏斗/CSS切分/VTSS/结构化caption) https://arxiv.org/abs/2410.08260 与 https://github.com/KwaiVGI/Koala-36M
- 同团队旁证: Kling-Foley (arXiv:2506.19774, 音频数据pipeline/VAD 0.2阈值/CLAP过滤) https://arxiv.org/abs/2506.19774
- 同团队旁证: KlingAvatar 2.0 (arXiv:2512.13313, 多说话人数据源与自动化标注pipeline) https://arxiv.org/abs/2512.13313
- 同团队旁证: Kling-MotionControl Technical Report (arXiv:2603.03160) https://arxiv.org/abs/2603.03160
- 第三方报道: 人民网/经济参考网 可灵3.0系列2026-02-05上线通稿 http://finance.people.com.cn/n1/2026/0205/c1004-40660255.html
- 官方一手: Kling Video 3.0 Omni 使用指南 https://kling.ai/quickstart/klingai-video-3-omni-model-user-guide
- 官方一手: Replicate官方托管API规格页 https://replicate.com/kwaivgi/kling-v3-video
- 第三方报道: 302.AI基准实验室Kling O3实测 https://zhuanlan.zhihu.com/p/2004988563100566106

### [LTX-2](../models/LTX-2.md)

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

### [LongCat-Video](../models/LongCat-Video.md)

1. arXiv:2510.22200 《LongCat-Video Technical Report》 https://arxiv.org/abs/2510.22200 与全文 https://arxiv.org/html/2510.22200v2 —— 官方一手（核心依据，数据处理章节、训练阶段表、RLHF 细节均出自此）。
2. GitHub 官方仓库 https://github.com/meituan-longcat/LongCat-Video —— 官方一手（开源范围、MIT 许可、模型变体与发布时间线）。
3. Hugging Face 模型卡 https://huggingface.co/meituan-longcat/LongCat-Video —— 官方一手（许可、能力描述、MOS 分数）。
4. arXiv:2605.26486 《LongCat-Video-Avatar 1.5 Technical Report》 https://arxiv.org/abs/2605.26486 —— 官方一手（同团队旁证，音视频对齐/音频过滤/情绪与静默数据 curation 细节主要来自此篇）。
5. 美团技术团队博客 https://tech.meituan.com/2025/12/23/longcat-video-avatar.html 与 https://tech.meituan.com/tags/longcat.html —— 官方一手（中文发布口径）。
6. 美团新闻稿《LongCat-Video 以长视频为锚点，构建世界模型技术底座》 https://www.meituan.com/news/NN251205166001020 —— 官方一手（定位与世界模型叙事）。
7. 36氪报道 https://www.36kr.com/p/3527169453464452 、知乎解读 https://zhuanlan.zhihu.com/p/1966806796392966062 —— 第三方报道（发布时间与推理加速倍数等佐证）。
8. HyperAI 论文页 https://hyper.ai/en/papers/2605.26486 —— 第三方索引。

### [MOVA](../models/MOVA.md)

- 【官方一手】arXiv:2602.08794v2《MOVA: Towards Scalable and Synchronized Video–Audio Generation》技术报告（38页，2026年2月10日）：https://arxiv.org/abs/2602.08794 、PDF https://arxiv.org/pdf/2602.08794 —— 本条目绝大部分字段的唯一且直接的信息来源，特别是第3节 Data Engineering、第4.3节 Progressive Joint Training、附录 A.1/A.3/A.4/A.5/A.6。注意 arXiv 未提供 HTML 版（/html/2602.08794v2 返回 No HTML），需直接解析 PDF。
- 【官方一手】GitHub 代码库 OpenMOSS/MOVA：https://github.com/OpenMOSS/MOVA —— 开源内容清单、Apache-2.0 许可、发布时间线（2026-01-29 首发 / 2026-02-10 技术报告与推理工作流 / 2026-03-09 API / 2026-05-06 评测代码）、数据集接口 mova/datasets/video_audio_dataset.py。
- 【官方一手】HuggingFace 模型页：https://huggingface.co/OpenMOSS-Team/MOVA-720p 、https://huggingface.co/OpenMOSS-Team/MOVA-360p 、collection https://huggingface.co/collections/OpenMOSS-Team/mova ；论文页 https://huggingface.co/papers/2602.08794
- 【官方一手】项目博客：https://mosi.cn/models/mova
- 【第三方报道】ComfyUI-Wiki 新闻《OpenMOSS Releases MOVA - Open-Source Synchronized Video-Audio Generation》（2026-01-29）：https://comfyui-wiki.com/en/news/2026-01-29-openmoss-mova-video-audio-generation —— 用于交叉核对首发时间。
- 【第三方报道】AI Films Studio 博客《MOVA: Open Source Video-Audio Generation》：https://studio.aifilms.ai/blog/mova-open-source-video-generation —— 用于交叉核对 Apache-2.0 商用许可表述。

### [合并条目：① Mochi 1](../models/Mochi_MAGI_Motif.md)

1. https://arxiv.org/abs/2604.16503 —— 官方一手，《Motif-Video 2B: Technical Report》，Motif Technologies，v2 于 2026-05-19 更新。本条目 Motif 部分的全部定量信息来源（Section 4 训练课程 Table 1、Section 5 Data 完整数据管线、Table 2 采样器利用率、Table 3 VBench 16 维结果）。
2. https://arxiv.org/html/2604.16503 —— 官方一手，Motif 技术报告 HTML 全文（本次调研抓取全文本地解析，Section 5.1 数据处理管线、5.2 视频打标、5.3 离线 bucket 均衡采样器逐段核对）。
3. https://huggingface.co/Motif-Technologies/Motif-Video-2B —— 官方一手，Motif-Video 2B 权重卡片，Apache 2.0，2026-04-14 发布，标注约 1000 万视频片段、<100,000 H200 GPU 小时、≈90% 数据利用率。
4. https://arxiv.org/abs/2505.13211 —— 官方一手，《MAGI-1: Autoregressive Video Generation at Scale》，Sand AI，2025-05-19，61 页。
5. https://arxiv.org/html/2505.13211v1 —— 官方一手，MAGI-1 技术报告 HTML 全文（本次调研抓取全文本地解析，Section 3 DATA 的 3.1 Filter Actors / 3.2 De-duplication / 3.3 MLLM as Advanced Filter / 3.4 Caption / 3.5 Data Adjustment、Table 3 caption 属性表、Table 4 caption 示例、Table 5 三阶段数据配置逐段核对）。
6. https://github.com/SandAI-org/MAGI-1 —— 官方一手，MAGI-1 代码仓库与 README，含 Apache 2.0 许可、24B/4.5B/Distill/FP8 变体清单与发布时间线（2025-04-21 至 2026-06-17 MAGI-1.1）。
7. https://www.genmo.ai/blog/mochi-1-a-new-sota-in-open-text-to-video —— 官方一手，Genmo 官方博客《Mochi 1: A new SOTA in open text-to-video》，AsymmDiT 架构、AsymmVAE 96× 压缩、44,520 token 上下文、Gemini-1.5-Pro-002 作为 prompt adherence 自动评测员等；全文无训练数据章节。
8. https://huggingface.co/genmo/mochi-1-preview —— 官方一手，Mochi 1 模型卡（48 层 AsymmDiT、362M AsymmVAE、480p/84 帧、Apache 2.0、约 60GB 显存、已做 NSFW 过滤、承认模型反映训练数据偏见）。
9. https://github.com/genmoai/mochi —— 官方一手，Mochi 1 代码仓库。
10. https://venturebeat.com/ai/video-ai-startup-genmo-launches-mochi-1-an-open-source-model-to-rival-runway-kling-and-others —— 第三方报道（VentureBeat，2024-10-22），含 Paras Jain 关于训练数据的直接引语「Generally, we use publicly available data and sometimes work with a variety of data partners」以及其拒绝细说的表态，是 Mochi 1 数据来源唯一的一手表述。
11. https://siliconangle.com/2024/10/22/genmo-introduces-mochi-1-open-source-text-video-generation-model/ —— 第三方报道，Mochi 1 发布报道与规格核对。
12. https://www.oschina.net/news/346129/sand-ai-magi1 —— 第三方报道（OSCHINA 中文），Sand AI 与曹越团队背景、MAGI-1 发布信息旁证。
13. https://huggingface.co/papers/2505.13211 —— 第三方聚合，MAGI-1 论文页与社区讨论。

### [Movie Gen](../models/Movie_Gen.md)

- 官方一手: Movie Gen: A Cast of Media Foundation Models (arXiv:2410.13720, 92页全文含附录J.1阈值表) https://arxiv.org/html/2410.13720v1
- 官方一手: facebookresearch/MovieGenBench (开源评测基准) https://github.com/facebookresearch/MovieGenBench

### [NVIDIA NeMo Curator](../models/NeMo_Curator.md)

- 【官方一手】NeMo Curator GitHub 主仓库（Apache 2.0，四模态能力、Ray/Xenna 架构、性能声明、安装方式）: https://github.com/NVIDIA-NeMo/Curator
- 【官方一手】NeMo Curator Releases 页面（各版本 highlights，含 26.02/26.04/26.07）: https://github.com/NVIDIA-NeMo/Curator/releases
- 【官方一手】PyPI nemo-curator 发布历史（精确发布时间：1.0.0=2025-10-01、1.1.0/26.02=2026-02-23、1.2.0/26.04=2026-05-14、1.3.0/26.07=2026-07-27）: https://pypi.org/project/nemo-curator/
- 【官方一手】NeMo Curator 视频 curation 总览文档（各 stage 与所用模型：TransNetV2、Qwen-VL、Cosmos-Embed1、CLIP aesthetic、NVENC/NVDEC、语义去重、WebDataset）: https://docs.nvidia.com/nemo/curator/curate-video
- 【官方一手】NeMo Curator 视频过滤文档（运动过滤与美学过滤的算法、stage 名称、全部默认阈值与参数）: https://docs.nvidia.com/nemo/curator/curate-video/process-data/filtering
- 【官方一手】NeMo Curator 视频架构文档（Ray Actor 管理、Ray Object Store、Cosmos-Xenna executor、自动扩缩容与全部默认配置项）: https://docs.nvidia.com/nemo/curator/about/concepts/video/architecture.md
- 【官方一手】NeMo Curator 音频 curation 文档（ASR 转写、WER/CER 过滤、说话人分离；确认与视频无联动）: https://docs.nvidia.com/nemo/curator/curate-audio
- 【官方一手】NeMo Curator 26.07 Release Notes / 迁移清单（音频增强 stage、Nemotron-CLIMB、captioning 后端矩阵、breaking changes）: https://docs.nvidia.com/nemo/curator/about/release-notes
- 【官方一手】NeMo Curator 26.02 视频快速上手文档: https://docs.nvidia.com/nemo/curator/26.02/get-started/video.html
- 【官方一手】NeMo Curator 26.02 转码/Clip Encoding 文档: https://docs.nvidia.com/nemo/curator/26.02/curate-video/process-data/transcoding.html
- 【官方一手】Cosmos-Xenna GitHub 仓库 README（Ray 分布式数据流水线库、streaming/batch/serving 三模式、自动扩缩容与 bin-packing、backpressure、SPMD、P2P 分发、Apache 2.0、仓库已停止积极开发的说明）: https://github.com/nvidia-cosmos/cosmos-xenna
- 【官方一手】Cosmos-Curate GitHub 仓库 README（Cosmos 训练数据生成系统，构建于 Cosmos-Xenna 之上，代码 Apache 2.0 / 模型 NVIDIA Open Model License）: https://github.com/nvidia-cosmos/cosmos-curate
- 【官方一手·方法论核心】Cosmos World Foundation Model Platform for Physical AI, NVIDIA, arXiv:2501.03575（第3节完整披露七级 curation pipeline：TransNetV2 选型 F1=0.967 横评、NVDEC/NVENC 6.5x、ViT 光流运动分类器、DOVER 底部15%、美学阈值3.5、InternVideo2+MLP 文字叠加与类型分类、VILA 13B FP8 TensorRT-LLM 10x、caption 559字符/97词、k-means k=10000 去重移除30%、九大类配比数字、20M小时→100M clip）: https://arxiv.org/abs/2501.03575
- 【官方一手】Training Video Foundation Models with NVIDIA NeMo, arXiv:2503.12964（clipping/sharding 双 pipeline 结构、100PB+ 口径、NVDEC/NVENC 3x、captioning 为瓶颈 stage、Ray 自动均衡 worker）: https://arxiv.org/abs/2503.12964
- 【官方一手·转载】Accelerate Custom Video Foundation Model Pipelines with New NVIDIA NeMo Framework Capabilities（NVIDIA 官方博客，89x 加速原始出处：「1K GPUs、ISO 功耗、相较未优化 CPU pipeline」；20M 小时；100PB+；L40S/H100/GB200 异构集群）: https://www.edge-ai-vision.com/2025/01/accelerate-custom-video-foundation-model-pipelines-with-new-nvidia-nemo-framework-capabilities/
- 【官方一手】Advancing Physical AI with NVIDIA Cosmos World Foundation Model Platform（NVIDIA 技术博客，2000万小时在 Hopper 40天 / Blackwell 14天 / CPU 3.4年）: https://developer.nvidia.com/blog/advancing-physical-ai-with-nvidia-cosmos-world-foundation-model-platform/
- 【官方一手】World Simulation with Video Foundation Models for Physical AI（Cosmos 后续论文 arXiv:2511.00062）: https://arxiv.org/pdf/2511.00062
- 【官方一手】NVIDIA Nemotron 3 Nano Omni 技术博客（26.07 引入 Curator 的全模态模型背景）: https://developer.nvidia.com/blog/nvidia-nemotron-3-nano-omni-powers-multimodal-agent-reasoning-in-a-single-efficient-open-model/
- 【官方一手】NeMo Curator 视频切分示例代码 tutorials/video/getting-started/video_split_clip_example.py: https://github.com/NVIDIA-NeMo/Curator/blob/main/tutorials/video/getting-started/video_split_clip_example.py
- 【官方一手】Cosmos Curator 视频 pipeline 参考文档 docs/curator/reference/video-pipelines.md: https://github.com/NVIDIA/cosmos-curator/blob/main/docs/curator/reference/video-pipelines.md
- 【第三方报道】Architecting Data Pipelines for Multimodal Datasets at Scale（Anyscale 博客，Ray 侧视角的多模态数据流水线架构）: https://www.anyscale.com/blog/architecting-multimodal-data-pipelines-that-scale-with-ray

### [OmniHuman 数据集 + OHBench](../models/OmniHuman.md)

- 【官方一手】arXiv:2604.18326《OmniHuman: A Large-scale Dataset and Benchmark for Human-Centric Video Generation》——v1 提交于 2026-04-20，v2 提交于 2026-05-30，cs.CV，19页6图。摘要页 https://arxiv.org/abs/2604.18326 ，HTML 全文（v2）https://arxiv.org/html/2604.18326v2 ，PDF https://arxiv.org/pdf/2604.18326 。本条目绝大多数字段的唯一直接来源。
- 【官方一手】GitHub 代码仓库 https://github.com/julia-cherry/OmniHuman ——OHBench 评测工具包，含各维度评测脚本、配置、依赖与模型检查点索引。注意论文正文中曾出现 github.com/juliacherry/OmniHuman（无连字符）的写法，该地址返回 404，正确地址为带连字符版本。
- 【官方一手】HuggingFace 数据集 https://huggingface.co/datasets/julia527/omnihuman ——约 62.7GB 分片 tar，含 sample_json（逐样本音视觉标注）、metadata（JSONL 索引）、tracking_npz（逐帧 SMPL/MANO 跟踪），不含原始视频，仅提供 YouTube URL + clip 起止秒。
- 【官方一手】HuggingFace 评测资产仓库 julia527/omnihuman_benchmark ——OHBench 所需的第三方判别模型检查点（.pt/.onnx）。
- 【对照工作旁证】OpenHumanVid（arXiv:2412.00115，复旦，CVPR 2025）、SpeakerVid-5M、CelebV-HQ、HDTF、VoxCeleb2、TikTok-v4、ActivityNet——论文 Table 1 的对照数据集，用于交叉理解 OmniHuman 在标注维度上的增量（见 benchmark_taxonomy_alignment）。
- 【评测对象旁证】LTX-2、MOVA、Ovi、UniVerse-1、UniAVGen 等开源 AV 联合生成模型的原始论文——用于理解 OHBench 评测结果的语境；其中 LTX-2 同时是本工作数据有效性验证实验的微调底座。
- 【工具链旁证】TransNetV2（镜头切分）、Demucs（四源分离）、DOVER（视频质量）、UniMatch（光流）、SigLIP（语义相似度）、YOLOv11 + MOTRv2（检测与多目标跟踪）、DWPose-L（134关键点）、3D-Speaker（说话人分离）、SyncNet（唇音同步）、ArcFace（人脸身份）、FunASR-Nano（ASR）、Qwen3-Omni（全模态标注）、Gemini-3 / Gemini-3-pro（语义裁决）、MUSIQ、RAFT、ImageBind、CLAP、Audiobox-Aesthetics、SenseVoice、DNSMOS——各自的原始论文与仓库，用于核对工具的实际功能与输出形式。

### [Open-Sora 系列](../models/Open-Sora.md)

【官方一手】1) Open-Sora 2.0 技术报告 arXiv:2503.09642 https://arxiv.org/abs/2503.09642 与 HTML 全文 https://arxiv.org/html/2503.09642v1 （数据 pipeline、数据统计图、三阶段成本表）；2) Open-Sora 1.2 论文 arXiv:2412.20404 https://ar5iv.labs.arxiv.org/html/2412.20404 （数据来源、80k 小时统计、bucket 策略、35k H100 GPU 小时）；3) Open-Sora GitHub 主仓 https://github.com/hpcaitech/Open-Sora （版本时间线、开源范围）；4) Open-Sora v1.2.0 数据处理代码与文档（一手代码级证据）：docs/data_processing.md、tools/scoring/README.md、tools/scene_cut/README.md、tools/caption/README.md，raw 路径形如 https://raw.githubusercontent.com/hpcaitech/Open-Sora/v1.2.0/tools/scoring/README.md ；5) Open-Sora Plan 论文 arXiv:2412.00131 https://arxiv.org/html/2412.00131v1 （数据来源表、七级过滤漏斗与逐级保留率表、训练阶段表）；6) Open-Sora Plan GitHub https://github.com/PKU-YuanGroup/Open-Sora-Plan 及 docs/Report-v1.3.0.md、docs/Report-v1.5.0.md（阈值、27% 保留率、prompt refiner 细节、v1.5 数据规模）。
【第三方报道】7) MarkTechPost 关于 Open-Sora 2.0 发布的报道 https://www.marktechpost.com/2025/03/14/hpc-ai-tech-releases-open-sora-2-0-an-open-source-sota-level-video-generation-model-trained-for-just-200k/ ；8) HuggingFace Papers 页面 https://huggingface.co/papers/2503.09642 与 https://huggingface.co/papers/2412.00131 ；9) comfyui-wiki 发布快讯 https://comfyui-wiki.com/en/news/2025-03-13-open-sora-2-release 。

### [Ovi](../models/Ovi.md)

1) 官方一手｜论文全文 https://arxiv.org/abs/2510.01284 与 https://arxiv.org/pdf/2510.01284 （《Ovi: Twin Backbone Cross-Modal Fusion for Audio-Video Generation》，第3节 Data Processing Pipeline 为数据侧唯一权威披露）
2) 官方一手｜GitHub 仓库与 README https://github.com/character-ai/Ovi （开源范围、Todo List、Ovi 1.1 更新说明、prompt 格式）
3) 官方一手｜HuggingFace 模型卡 https://huggingface.co/chetwinlow1/Ovi （参数量、基座模型、许可、分辨率/时长版本）
4) 官方一手｜项目主页 https://aaxwaz.github.io/Ovi/ （demo 与代码入口）
5) 同团队旁证｜HuggingFace Papers 页 https://huggingface.co/papers/2510.01284
6) 第三方报道｜CSDN 中文解读 https://blog.csdn.net/SuaniCommunity/article/details/154737163 与腾讯新闻 https://view.inews.qq.com/a/20251113A02WA900 （复述论文数据 pipeline 四步与 720×720 门槛，无新增一手信息）
7) 第三方旁证｜腾讯云开发者社区技术详解 https://cloud.tencent.cn/developer/article/2584843
8) 第三方平台｜WaveSpeed AI 托管页 https://wavespeed.ai/models/character-ai/ovi/text-to-video （商用可得性）

### [Script-a-Video](../models/Script-a-Video.md)

- 【官方一手】arXiv 摘要页 https://arxiv.org/abs/2604.11244 —— 论文题目、机构（Tencent Hunyuan Team）、摘要核心数字（Video-SALMONN-2 总错误率平均降低 25%、Daily-Omni 平均提升 67%、多镜头生成人评身份一致性 +45% / 音视频对齐 +56% / 时序可控性 +71%）。
- 【官方一手】arXiv HTML 全文 v2 https://arxiv.org/html/2604.11244v2 （标注日期 2026-04-15，cs.CV）—— 本条目绝大多数字段的唯一直接来源：第 3 节 MTSS 四流 schema 完整字段定义、第 4.1 节 caption 数据集构建与 Table 1/Table 2、第 4.2 节生成实验与 Table 3/Table 4、第 4.2.2 节训练细节、Limitations 章节。
- 【官方一手】arXiv HTML 全文 v1 https://arxiv.org/html/2604.11244v1 —— 用于与 v2 交叉核对（v1 摘要称 Daily-Omni 提升 67%，v2 Figure 1 与正文分别给出 110% 与 127% 的口径，说明不同版本/不同统计口径下数字有差异）。
- 【官方一手】arXiv PDF https://arxiv.org/pdf/2604.11244 —— 图表版式与 Figure 3 的 MTSS 脚本示例。
- 【第三方旁证】腾讯混元 GitHub 组织 https://github.com/Tencent-Hunyuan —— 用于核实是否存在配套开源仓库；截至调研时该组织下有 HunyuanVideo、HunyuanVideo-1.5、HunyuanVideo-Avatar、HunyuanVideo-Foley、HunyuanVideo-I2V 等仓库，但无 Script-a-Video / MTSS 相关仓库，据此判定未开源。
- 【说明】未检索到任何第三方媒体报道、技术博客解读或社区复现讨论，该工作在调研时点尚属新发布且传播度有限。

### [Seedance 2.0 与 Seedance 1.5 pro](../models/Seedance_20_Seedance_15_pro.md)

- 官方一手: Seedance 2.0 报告 (arXiv:2604.14148, 26页, 仅引言+评测+贡献者, 无数据章节) https://arxiv.org/abs/2604.14148
- 官方一手: Seedance 1.5 pro 报告 (arXiv:2512.13507, v1-v3全部核对, 无数据章节) https://arxiv.org/abs/2512.13507
- 官方一手: Seedance 1.0 技术报告 (arXiv:2506.09113, 含完整Data章节, 本条目数据管线的可考证基线) https://arxiv.org/abs/2506.09113
- 辨伪说明: emergentmind等AI摘要站流传的'1亿分钟训练数据/四阶段pipeline'经三版PDF全文检索零命中, 判定为幻觉, 不采信

### [SkyReels 系列](../models/SkyReels.md)

- 官方一手 | https://arxiv.org/abs/2602.21818 — SkyReels-V4 技术报告（2026-02-24 投稿，v3 2026-03-18），含数据采集/处理/打标、六阶段训练表、SyncNet 阈值、SkyReels-VABench
- 官方一手 | https://arxiv.org/html/2602.21818v3 — V4 论文 HTML 全文（数据章节、Table 1 训练阶段表）
- 官方一手 | https://arxiv.org/abs/2504.13074 — SkyReels-V2 技术报告（2025-04-17），含数据 pipeline、SkyCaptioner-V1、多阶段训练与 RL
- 官方一手 | https://github.com/SkyworkAI/SkyReels-V2 — V2 官方仓库（权重清单、SkyCaptioner-V1、发布时间线）
- 官方一手 | https://huggingface.co/Skywork/SkyCaptioner-V1 — SkyCaptioner-V1 模型卡（Qwen2.5-VL-7B-Instruct 基座、1000万→200万概念均衡数据、32×A800、准确率）
- 官方一手 | https://github.com/SkyworkAI/SkyReels-V3 — V3 官方仓库（2026-01-29 开源）
- 第三方解读 | https://blog.csdn.net/Together_CZ/article/details/148583114 — CSDN 对 V2 论文数据章节的详细中文转述（O(100M) 规模、28万部电影/80万集剧集/620万小时、过滤器清单、人工抽检率、DPO 数据量）
- 第三方解读 | https://www.alphaxiv.org/overview/2504.13074 — alphaXiv 对 V2 的结构化解读（1亿视频样本、PyDetect+TransNet-V2、Qwen2.5-VL-7B、30k 样本对）
- 第三方解读 | https://www.alphaxiv.org/overview/2602.21818 — alphaXiv 对 V4 的解读（VABench 五维度、双向交叉注意力、Refiner）
- 第三方解读 | https://www.emergentmind.com/papers/2504.13074 — V2 数据流水线与训练课程摘要（双轴分桶、四阶段后训练）
- 第三方报道 | https://finance.sina.com.cn/tech/roll/2026-01-30/doc-inhkakfi6974616.shtml — 新浪科技：SkyReels-V3 开源报道
- 第三方报道 | https://tech.cnr.cn/techgd/20260328/t20260328_527565454.shtml — 央广网：昆仑万维2026中关村论坛发布 SkyReels-V4
- 第三方报道 | https://wavespeed.ai/blog/posts/what-is-skyreels-v4/ — V4 能力与「权重尚未发布、限量预览」的状态说明
- 第三方报道 | https://comfyui-wiki.com/en/news/2025-04-21-skyreels-v2-infinite-length-film-generative-model — V2 发布与 VBench 83.9% 总分

### [Sora 2](../models/Sora_2.md)

- Sora 2 System Card, OpenAI, 2025-09-30 (PDF): https://cdn.openai.com/pdf/50d5973c-c4ff-4c2d-986f-c72b5d0ff069/sora_2_system_card.pdf
- Sora 2 System Card 索引页: https://openai.com/index/sora-2-system-card/
- OpenAI Deployment Safety Hub - Sora 2: https://deploymentsafety.openai.com/sora-2
- Sora System Card (Sora 1, 含CSAM安全栈细节): https://openai.com/index/sora-system-card/
- Video generation models as world simulators (Sora 1 技术博客，含时空patch/原生分辨率/重打标): https://openai.com/index/video-generation-models-as-world-simulators/
- Sora 2 is here (发布公告): https://openai.com/index/sora-2/
- Disney 与 OpenAI 授权协议公告: https://openai.com/index/disney-sora-agreement/
- Sora, Not Sorry: OpenAI Backtracks on Opt-Out Copyright Policy, Copyright Lately: https://copyrightlately.com/openai-backtracks-sora-opt-out-copyright-policy/
- Sora 2 Does A Copyright Somersault Upon Launch, Forbes, 2025-10-17: https://www.forbes.com/sites/legalentertainment/2025/10/17/sora-2-does-a-copyright-somersault-upon-launch/
- MPA 要求 Sora 2 停止侵权, CNBC, 2025-10-07: https://www.cnbc.com/2025/10/07/openais-sora-2-must-stop-allowing-copyright-infringement-mpa-says.html
- 日本政府就 Sora 2 侵权向 OpenAI 提出要求, EC IP Helpdesk: https://intellectual-property-helpdesk.ec.europa.eu/news-events/news/japanese-government-requests-openai-avoid-copyright-infringement-sora-2-us-federal-judge-dismisses-2025-10-23_en
- Public Citizen 要求 OpenAI 撤回 Sora 2 的公开信: https://www.citizen.org/news/public-citizen-letter-calls-on-open-ai-to-withdraw-sora-2-video-generation/
- OpenAI Will Shut Down Sora Video App; Disney Drops $1B Investment, Variety, 2026-03: https://variety.com/2026/digital/news/openai-shutting-down-sora-video-disney-1236698277/
- Sora Shutting Down, Disney Investment Dead, Deadline, 2026-03: https://deadline.com/2026/03/sora-shut-down-disney-investment-1236764689/
- OpenAI Adds Longer Clips and Storyboarding to Sora 2, eWeek: https://www.eweek.com/openai/openai-adds-longer-clips-sora-2/
- How OpenAI Built Sora 2: Training, Data, and Model Design (第三方技术解读，非官方): https://skywork.ai/blog/openai-sora-2-2025-ultimate-guide-training-model-design/
- How to Uncover Sora 2's Training Datasets (第三方，非官方): https://skywork.ai/blog/how-to-uncover-sora-2s-training-datasets/
- Sora 2 API on Replicate (规格与定价): https://replicate.com/openai/sora-2
- Getty Images/OpenAI 授权合作报道 (2026-06): https://finance.yahoo.com/markets/stocks/articles/getty-images-openai-deal-gives-154500732.html

### [SpeakerVid-5M](../models/SpeakerVid-5M.md)

- 【官方一手】arXiv:2507.09862《SpeakerVid-5M: A Large-Scale High-Quality Dataset for Audio-Visual Dyadic Interactive Human Generation》（2025年7月14日提交，cs.CV）：https://arxiv.org/abs/2507.09862 ，全文 HTML https://arxiv.org/html/2507.09862v1 —— 本条目绝大多数字段的直接来源，特别是第3节数据构建 pipeline、质量过滤阈值、四分支组织、Table 1 数据集对比与 Table 2 消融结果。
- 【官方一手】项目主页 https://dorniwang.github.io/SpeakerVid-5M/ —— 作者所属机构（清华大学、StepFun、香港科技大学/广州）、项目负责人 Duomin Wang 与通讯作者 Xiu Li、资源发布状态与各链接入口。
- 【官方一手】GitHub 数据清洗代码库 https://github.com/Dorniwang/SpeakerVid-5M-Code —— 六段式 curation pipeline 代码清单、依赖工具栈（3D-Speaker/YOLO/DWpose/Whisper/SceneDetect/SyncNet/ArcFace/Deepface/UniSpeech/Deep3DFaceRecon/yt-dlp）、以及「仅限非商业科研教育用途、版权归原作者、提供 takedown 政策」的许可条款。
- 【官方一手】GitHub 项目页仓库 https://github.com/Dorniwang/SpeakerVid-5M —— 项目主页源。
- 【官方一手】HuggingFace 数据集 https://huggingface.co/datasets/dorni/SpeakerVid-5M-Dataset （创建于 2025-07-18，最后更新 2025-08-04，约 1021 次下载，18 likes）—— 实际发布内容清单：all_data_list.json、SFT_set.json 与 merge_anno / dwpose / asr / l_score / anno 五类标注文件夹；确认原始视频不托管、需按 YouTube ID 自行下载；确认 dwpose 骨架数据因体积未上传。
- 【同团队旁证】arXiv:2602.08794《MOVA: Towards Scalable and Synchronized Video–Audio Generation》—— 非同团队，但为下游使用方，其第3节将 SpeakerVid-5M 列为 Phase 1 训练数据来源并明确定位为唇同步能力的核心来源，用于交叉印证本数据集的语种重心与音频类别属性（属旁证，非 SpeakerVid-5M 官方表述）。

### [Step-Video-T2V](../models/Step-Video-T2V.md)

- 【官方一手】Step-Video-T2V Technical Report: The Practice, Challenges, and Future of Video Foundation Model, Step-Video Team（阶跃星辰）, arXiv:2502.10248, 2025-02（含 Section 7 Data：视频切分/质量评估/运动评估/打标/概念均衡/视频文本对齐六阶段，Figure 11 分层过滤示意，训练阶段配置表）: https://arxiv.org/abs/2502.10248
- 【官方一手】Step-Video-T2V 技术报告 ar5iv 全文（数据章节可检索，含 7 类质量标签、Farneback 光流三指标、12 万+ K-means 簇、8 帧 CLIP Score、30M SFT、95k 蒸馏样本、644M/27.3M seen samples 等数值）: https://ar5iv.labs.arxiv.org/html/2502.10248
- 【官方一手】Step-Video-T2V 技术报告 arXiv HTML v1: https://arxiv.org/html/2502.10248v1
- 【官方一手】Step-Video-T2V 技术报告 arXiv HTML v2（含分层过滤与 Figure 11 说明、Step-Video-T2V-Eval 128 prompt/11 类目、Video-DPO 人工偏好标注流程）: https://arxiv.org/html/2502.10248v2
- 【官方一手】Step-Video-T2V GitHub 开源仓库（权重、推理代码、Video-VAE、双语文本编码器、MIT 许可证、2025-02-17 发布）: https://github.com/stepfun-ai/Step-Video-T2V
- 【官方一手】Step-Video-T2V Hugging Face 模型卡: https://huggingface.co/stepfun-ai/stepvideo-t2v
- 【官方一手】Step-Video-T2V-Eval 评测基准数据集（128 条中文 prompt、11 类目、含多引擎对比生成结果）: https://huggingface.co/datasets/stepfun-ai/Step-Video-T2V-Eval
- 【同团队旁证】Step-Video-TI2V Technical Report, arXiv:2503.11251, 2025-03（含 5M 文本-图像-视频三元组、动漫数据占比 >80% 的配比失衡自述、光流运动分提取方法与阈值过滤、运动分作为可控条件、caption 模型微调强化运镜描述、Step-Video-TI2V-Eval 178+120 条）: https://arxiv.org/html/2503.11251v1
- 【同团队旁证】Step-Video-TI2V GitHub 开源仓库: https://github.com/stepfun-ai/Step-Video-TI2V
- 【第三方】The Moonlight 文献综述：Step-Video-T2V 技术报告解读（分层过滤与 6 个预训练子集的说明）: https://www.themoonlight.io/en/review/step-video-t2v-technical-report-the-practice-challenges-and-future-of-video-foundation-model
- 【第三方】Kingy AI：Step-Video-T2V 技术报告论文摘要（数据 pipeline 与 SFT/DPO 概述）: https://kingy.ai/blog/step-video-t2v-technical-report-paper-summary/
- 【第三方】Hugging Face Papers：Step-Video-T2V 论文页与社区讨论: https://huggingface.co/papers/2502.10248
- 【第三方】CSDN：Step-Video-T2V 阶跃星辰发布最强开源视频生成模型（论文详解，中文数据章节逐条拆解）: https://blog.csdn.net/sherlockMa/article/details/145706142
- 【第三方】CSDN：阶跃星辰的开源探索——Step-Video-T2V 与 Step-Audio 深度解析（说明二者为独立模型线，非联合 AV 生成）: https://blog.csdn.net/liaoqingjian/article/details/145820964
- 【第三方】知乎：阶跃星辰 30B 视频生成模型 Step-Video 简析: https://zhuanlan.zhihu.com/p/24619034131
- 【第三方】知乎：阶跃星辰开源 Step-Video-TI2V 图生视频模型介绍（102 帧/5 秒/540P、运动幅度可控与镜头运动可控）: https://zhuanlan.zhihu.com/p/31775732208
- 【第三方】智源社区：阶跃星辰首次开源 Step 系列多模态大模型（2025-02-18 官方发布报道）: https://hub.baai.ac.cn/view/43466
- 【第三方】NeuroHive：Step-Video-T2V 开源模型 16x 视频压缩突破解读: https://neurohive.io/en/state-of-the-art/step-video-t2v-text-to-video-open-source-model-achieves-16x-video-compression-breakthrough/

### [UniTalking](../models/UniTalking.md)

- 【官方一手】arXiv:2603.01418v1《UniTalking: A Unified Audio-Video Framework for Talking Portrait Generation》（2026-03-02 提交，CVPR 2026 Findings 接收）：https://arxiv.org/abs/2603.01418 ，HTML 全文 https://arxiv.org/html/2603.01418v1 ，PDF https://arxiv.org/pdf/2603.01418 —— 本条目全部字段的唯一直接来源。全文共10页（含参考文献），正文第3节「Data Preparation」为数据部分的完整内容，篇幅约一页，正文中提及的 Appendix 在 v1 版 PDF 中并未实际包含。
- 【第三方旁证】ResearchGate 条目：https://www.researchgate.net/publication/401470058_UniTalking_A_Unified_Audio-Video_Framework_for_Talking_Portrait_Generation —— 用于交叉核对标题与作者。
- 【第三方旁证】X/Twitter @CSVisionPapers 推文：https://x.com/CSVisionPapers/status/2028948978948051095 —— 用于确认作者列表与 arXiv 分类（cs.CV, cs.MM, cs.SD）。
- 【第三方旁证】talking-face-arxiv-daily 论文追踪仓库：https://github.com/liutaocode/talking-face-arxiv-daily —— 用于确认论文在说话人生成方向的收录情况，并交叉验证无官方代码仓库。
- 【上游数据来源一手】OpenHumanVid（CVPR 2025，复旦大学）arXiv:2412.00115：https://arxiv.org/abs/2412.00115 ，项目主页 https://fudan-generative-vision.github.io/OpenHumanVid ，代码库 https://github.com/fudan-generative-vision/OpenHumanVid —— 用于补充 UniTalking 未描述的上游数据集属性（52.3M 原始片段 / 70.6K 小时 → 过滤后 13.2M 高质量片段、短/长/结构化三类 caption、骨架序列与语音音轨、需审核授权的下载许可）。此为旁证性质，UniTalking 论文本身未复述这些数字。
- 【基座模型旁证】Wan 技术报告 arXiv:2503.20314 —— 用于确认 Wan2.2 的 3D causal VAE 压缩率、分辨率与帧率规格（UniTalking 视频侧规格由该基座隐含决定，论文本身未披露）。
- 【对照工作旁证】OVI（arXiv:2510.01284）与 UniVerse-1（arXiv:2509.06155）—— UniTalking 的主要对照基线，其 RoPE 策略明确 following OVI，数据流水线可与 UniVerse-1 直接对比。

### [UniVerse-1](../models/UniVerse-1.md)

- 【官方一手】arXiv:2509.06155v1《UniVerse-1: Unified Audio-Video Generation via Stitching of Experts》（2025-09-07 提交）：https://arxiv.org/abs/2509.06155 ，HTML 全文 https://arxiv.org/html/2509.06155v1 —— 本条目绝大多数字段的唯一直接来源，特别是第 3 节数据构建与在线标注 pipeline、第 4 节实验与消融、附录 Verse-Bench 类目表。
- 【官方一手】GitHub 代码库 Dorniwang/UniVerse-1-code：https://github.com/Dorniwang/UniVerse-1-code/ —— 开源清单、Apache-2.0 许可、发布时间线（2025-09-03 项目页 / 09-08 权重与 Verse-Bench / 09-09 技术报告 / 09-28 评测工具）。
- 【官方一手】HuggingFace 模型页 dorni/UniVerse-1-Base：https://huggingface.co/dorni/UniVerse-1-Base —— 7B 参数规模、bfloat16 推理配置、Apache-2.0 声明、7,600 小时训练数据表述。
- 【官方一手】项目主页：https://dorniwang.github.io/UniVerse-1/ —— 机构归属（StepFun / 港科广 / 港科大 / 清华）、演示样例、层插值方法描述、Verse-Bench 三子集构成。
- 【官方一手】HuggingFace Papers 页：https://huggingface.co/papers/2509.06155 —— 摘要与核心贡献点交叉核对。
- 【同团队旁证】OpenReview 投稿页 https://openreview.net/forum?id=8aFYx2mDyE —— 用于确认论文的会议投稿状态。
- 【第三方旁证】ResearchGate 论文条目：https://www.researchgate.net/publication/395356081_UniVerse-1_Unified_Audio-Video_Generation_via_Stitching_of_Experts
- 【第三方旁证】后续同方向工作对 UniVerse-1 的引用与对比，如 UniAVGen（arXiv:2511.03334）、JavisDiT++（arXiv:2602.19163）、MMControl（arXiv:2604.19679）—— 用于确认 UniVerse-1 已成为音视频联合生成的标准对照基线之一。

### [Unison](../models/Unison.md)

- 【官方一手】arXiv:2605.08729v2《Unison: Harmonizing Motion, Speech, and Sound for Human-Centric Audio-Video Generation》（v1 2026-05-09 提交，v2 2026-06-29 更新，cs.CV，CC BY 4.0）：摘要页 https://arxiv.org/abs/2605.08729 ，HTML 全文 https://arxiv.org/html/2605.08729v2 ，PDF https://arxiv.org/pdf/2605.08729 —— 本条目全部字段的唯一直接来源。数据相关信息集中于 4.1 节 Training Corpora（一段）与 3 节方法开头的 lip-filtering 描述（三句），其余数据字段均无原文支撑。
- 【官方一手】arXiv:2605.08729v1 HTML：https://arxiv.org/html/2605.08729v1 —— 已逐段比对，训练语料与数据处理描述与 v2 完全一致，可确认数据侧内容自 v1 起未作修订。
- 【引用的上游数据集，第三方一手】OpenHumanVid（arXiv:2412.00115，人物中心大规模高质量视频数据集）、HDTF（High-Definition Talking Face，CVPR 2021）、VFHQ（arXiv:2205.03409，高保真人脸视频）、CelebV-Text（CVPR 2023，文本标注人脸视频）、VGGSound（ICASSP 2020，约 200K clips / 550 小时视听事件数据集）—— 音视频联合训练语料的五个来源，其各自的原始规模、采集方式与标注体系是推断 Unison 数据分布的间接依据。
- 【引用的上游数据集，第三方一手】YouTube-8M、AudioSet、WavCaps（音效来源）；VidMuse（音乐来源）；YuE / YuE-scaling（arXiv，歌唱来源）—— 音频分支单模态训练语料来源。
- 【引用的上游模型与工具，第三方一手】Wan2.2（视频基座）、MMAudio（arXiv:2412.15322，音频基座）、Zipformer/ZipVoice（语音生成增强）、Mel-RoFormer（arXiv:2409.04702，人声分离，同时用于训练数据解耦与评测预处理）、SyncNet（唇同步）、Whisper-large-v3（WER 评测）、Synchformer（DeSync 评测）、ImageBind/CLAP/VideoCLIP-XL-V2/LAION-Aesthetic V2.5/DINOv3/Audiobox-Aesthetics（评测指标）。
- 【同方向旁证，用于横向对照】UniVerse-1（arXiv:2509.06155）、Ovi、UniAVGen（arXiv:2511.03334）、Harmony（arXiv:2511.21579）、MOVA（arXiv:2602.08794）、LTX-2（arXiv，19B 音视频联合生成）、JavisDiT（arXiv:2503.23377）—— 均为 Unison 在 Table 1 中的对比基线，其公开的数据 pipeline 可用于反衬 Unison 的披露缺口。
- 【重名工作，需排除】arXiv:2605.31530《UNISON: A Unified Sound Generation and Editing Framework via Deep LLM Fusion》—— 纯音频生成与编辑框架，与本条目无关，检索时须注意区分。

### [Veo 3 / Veo 3.1](../models/Veo_3_Veo_31.md)

- 官方一手: Veo 3 Tech Report (PDF, Model & Data / Mitigations章节) https://storage.googleapis.com/deepmind-media/veo/Veo-3-Tech-Report.pdf
- 官方一手: Veo 3 Model Card (PDF, Training Dataset/Processing/Evaluation) https://storage.googleapis.com/deepmind-media/Model-Cards/Veo-3-Model-Card.pdf
- 官方一手: Veo 3.1 Lite Model Card (确认3.1系列沿用Veo 3数据披露) https://deepmind.google/models/model-cards/veo-3-1-lite/
- 第三方报道: CNBC - Google使用YouTube视频子集训练Gemini与Veo 3 https://www.cnbc.com/2025/06/19/google-youtube-ai-training-veo-3.html
- 官方一手: Video models are zero-shot learners and reasoners (arXiv:2509.20328, 反推无显式几何标注) https://arxiv.org/abs/2509.20328
- 官方一手: Veo产品页 (输出规格与MovieGenBench结果) https://deepmind.google/models/veo/

### [Vidu S1](../models/Vidu_S1.md)

1) 官方一手：arXiv:2607.03118v2《Vidu S1: A Real-Time Interactive Video Generation Model》https://arxiv.org/abs/2607.03118 （含第2.1节 Data Preparation 与 Figure 2 数据过滤流水线图，本调研数据侧信息几乎全部来自此节）
2) 官方一手：产品页与在线 demo https://vidu.com/vidu-stream
3) 第三方报道：中国日报网《生数科技发布 Vidu S1，推动视频生成迈向「实时交互」新时代》https://tech.chinadaily.com.cn/a/202607/06/WS6a4b12eea310d709c2fbbecb.html
4) 第三方报道：雷峰网 https://www.leiphone.com/category/industrynews/6GlFzI5hMwcfRoGZ.html ；爱范儿 https://www.ifanr.com/digest/1670950 （发布时间、AR+Diffusion 架构、540P/25-42FPS 等产品参数）
5) 第三方整理：AI 科技深度解读 Vidu S1 词条 https://www.ai-all.info/ai-models/vidu-s1 （TurboDiffusion/TurboServe、消费级 GPU RTX 3060 起、端到端延迟<200ms 等，属二手信息）

### [Wan 2.5 / 2.6 / 2.7](../models/Wan.md)

- 【官方一手·核心】Wan: Open and Advanced Large-Scale Video Generative Models（Wan Team, 阿里通义实验室）, arXiv:2503.20314, 2025-03-26, 60页33图。第3章「Data Processing Pipeline」为全系列数据方法的唯一详细一手来源：3.1 预训练数据（四步清洗、9项基础过滤器、约50%淘汰率、100簇配额采样、六档运动分级、视觉文字合成）、3.2 后训练数据（top20%、12大类、百万级）、3.3 密集视频caption（LLaVA式架构、slow-fast编码、10维F1对标Gemini 1.5 Pro）；4.2.2/4.2.3 训练课程；4.6 Wan-Bench；4.7.2 架构消融；5.7 音频生成（V2A的1D-VAE、三段式音视频caption、O(1)千小时子集、排除语音）: https://arxiv.org/abs/2503.20314
- 【官方一手】Wan 2.1 技术报告 PDF（本调研的数据章节原文均引自此PDF）: https://arxiv.org/pdf/2503.20314
- 【官方一手】Wan-S2V: Audio-Driven Cinematic Video Generation, arXiv:2508.18621, 2025-08-26。第2章「Data Processing Pipeline」是Wan系唯一写出音画同步数据过滤方法的文档：Light-ASD主动说话人检测两条排除规则、VitPose/DWPose姿态跟踪、Dover/UniMatch/Laplacian/美学/OCR五项质量指标、QwenVL2.5-72B打标规范；表1含Sync-C等定量对比: https://arxiv.org/abs/2508.18621
- 【官方一手】Wan2.2 GitHub 仓库 README（数据增量「图像+65.6%、视频+83.2%」、电影美学标签体系含光影/构图/对比度/色调、MoE双专家去噪、Wan2.2-VAE 16×16×4压缩、S2V与Animate、Apache 2.0）: https://github.com/Wan-Video/Wan2.2
- 【官方一手】Wan2.1 GitHub 仓库: https://github.com/Wan-Video/Wan2.1
- 【官方一手】Wan-Video GitHub 组织页（截至2026-07仅有 Wan2.1/Wan2.2/Wan-Dancer/Wan-skills/diffusers 五个仓库，无 Wan2.5/2.6/2.7 —— 闭源的直接证据）: https://github.com/Wan-Video
- 【官方一手】Hugging Face Wan-AI 组织页（最新权重止于 Wan2.2 系列与 Wan-Dancer-14B，无 2.5/2.6/2.7）: https://huggingface.co/Wan-AI
- 【官方一手·能力矩阵】阿里云百炼「视频生成」模型总览文档（wan2.7/2.6/2.5/2.2/2.1 全版本能力标签、输入模态、分辨率档位、时长、fps、地域部署；明确 2.5=「有声视频·声画同步」、2.6/2.7=「有声视频·多镜头叙事·声画同步」，2.7-i2v 输入模态含文本/图像/音频/视频）: https://help.aliyun.com/zh/model-studio/video-generation
- 【官方一手·API】万相2.7-文生视频 API 参考（模型名 wan2.7-t2v-2026-06-12 含发布日期戳、audio_url 音频条件输入、自动配音、时间戳分镜脚本示例、duration [2,15]、ratio 与分辨率对照表、prompt 上限5000字符、watermark 参数）: https://help.aliyun.com/zh/model-studio/text-to-video-api-reference
- 【官方一手·API】万相-图生视频-基于首帧 API 参考（2.1–2.6）（wan2.6 的 shot_type:"multi" 多镜头叙事、wan2.5/2.6 的 audio_url 与自动配音、音频3–30秒/wav/mp3/≤15MB 约束与截断规则、各版本分辨率与时长枚举、prompt 字符上限差异）: https://help.aliyun.com/zh/model-studio/image-to-video-api-reference
- 【官方一手·发布公告】《全新万相2.6系列模型，正式发布！》, 阿里云开发者社区, 2025-12-17（国内首个支持角色扮演的视频生成模型、音画同步、多镜头生成、声音驱动、单次最长15秒、分镜控制与跨镜头一致性建模、万相家族已支持10余种视觉创作能力）: https://developer.aliyun.com/article/1693622
- 【官方一手·发布公告】《万相 Wan2.6 全新升级发布！人人都能当导演的时代来了》, 阿里云开发者社区, 2025-12-16: https://developer.aliyun.com/article/1693451
- 【官方一手·同团队旁证】Wan-Dancer（音乐驱动分钟级舞蹈视频生成）, arXiv:2607.09581, 2026-07。数据侧：自建约200小时≥720p/30fps专有数据集、五个舞种近似均匀分布以缓解类别不均衡、5秒clip且50%重叠的切分策略、Librosa音频特征、SEA-RAFT光流mask入损失: https://arxiv.org/abs/2607.09581
- 【官方一手】Wan-Dancer GitHub 仓库 README: https://github.com/Wan-Video/Wan-Dancer
- 【官方一手】万相官网（Wan 2.5/2.6/2.7 产品页与在线体验入口）: https://wan.video/
- 【官方一手】通义万相官网入口: https://tongyi.aliyun.com/wan/
- 【第三方整理】万相2.6 产品说明页（「2025.12.16 重磅发布」、原生音画同步、多镜头叙事、首创视频角色扮演、音频驱动模式、15秒/1080P、相较 Wan 2.5 的10秒提升）: https://wan2.video/zh/wan2.6
- 【第三方整理】万相2.7 产品说明页（1080P/15秒上限、首尾帧控制、角色克隆、指令编辑、单图多镜头叙事、原生音画同步）: https://wan2.video/zh/wan2.7
- 【第三方报道】《通义万相Wan2.6发布：从「随机生成」迈向「精确执导」》, 腾讯新闻, 2025-12-17: https://news.qq.com/rain/a/20251217A031VN00
- 【第三方报道】《阿里通义万相 Wan 2.6 发布，从「生成一段视频」到「帮你把戏拍出来」》, 知乎专栏: https://zhuanlan.zhihu.com/p/1984672026435294934
- 【第三方报道】《阿里通义万相Wan2.6正式发布！如何使用？》, 知乎专栏: https://zhuanlan.zhihu.com/p/1986838016199766019
- 【相关工作对照】LTX-2: Efficient Joint Audio-Visual Foundation Model, arXiv:2601.03233（音频信息量筛选与全音景双轨caption，与Wan的三段式V2A caption构成对照）: https://arxiv.org/abs/2601.03233
- 【方法引用·第三方】Light-ASD: A Light Weight Model for Active Speaker Detection, Liao et al., CVPR 2023（Wan2.2-S2V 音画同步过滤所用模型）: https://openaccess.thecvf.com/content/CVPR2023/html/Liao_A_Light_Weight_Model_for_Active_Speaker_Detection_CVPR_2023_paper.html

### [音视频生成评测基准合集](../models/av_benchmarks.md)

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

### [视频 Caption 模型生态](../models/caption_models.md)

- https://arxiv.org/abs/2406.04325 — ShareGPT4Video 论文（官方一手，NeurIPS 2024 D&B Track，CC BY 4.0；40K GPT4V 密集 caption + 4.8M ShareCaptioner-Video 标注 + DiffSW 方法）
- https://huggingface.co/Lin-Chen/ShareCaptioner-Video — ShareCaptioner-Video 模型卡（官方一手，确认基座为 InternLM-XComposer2-4KHD、支持流式差分滑窗与 clip summarization）
- https://arxiv.org/abs/2501.07888 — Tarsier2 论文（官方一手；7B、预训练 11M→40M video-text pairs、细粒度时序对齐 SFT、model-based sampling + DPO；DREAM-1K F1 超 GPT-4o 2.8%、超 Gemini-1.5-Pro 5.8%；人评 +8.6% / +24.9%；15 个公开基准 SOTA）
- https://github.com/bytedance/tarsier — Tarsier 官方仓库（官方一手；Tarsier2-Recap-7b 权重与 Tarsier2-Recap-585K 数据集）
- https://huggingface.co/omni-research/Tarsier2-Recap-7b — Tarsier2-Recap 模型卡（官方一手；585K clip 来自 VATEX/TGIF/LSMDC 等公开数据集，由 Tarsier2-7B 标注；上游为 1M 视频的 recaption）
- https://github.com/zai-org/CogVideo/blob/main/tools/caption/README.md — CogVideoX 打标工具文档（官方一手；CogVLM2-Caption 用途与 video→caption→video 闭环）
- https://arxiv.org/pdf/2408.06072 — CogVideoX 论文（官方一手；Panda-70M 短 caption → CogVLM 逐帧稠密 caption → GPT-4 摘要 → 5 万条数据微调 LLaMA2 摘要模型 → 蒸馏为 CogVLM2-Caption 的四段式打标链，附录 G 给出完整 prompt）
- https://arxiv.org/abs/2410.03051 — AuroraCap + VDC benchmark（官方一手；token merging 降视觉 token、2000 万图/视频-文本对三阶段训练、VDC 千余条结构化 caption 含 camera/short/background/main object/detailed 五字段、VDCScore 分治式 LLM 评测；Flickr30k CIDEr 88.9 > GPT-4V 55.3 > Gemini-1.5-Pro 82.2）
- https://arxiv.org/abs/2410.08260 — Koala-36M（官方一手；36M clip、均长 13.75s、720p、caption 均长 202 词、GPT-4V 生成种子 caption 微调 LLaVA、六类结构化信息、VTSS 训练适用性评分）
- https://openaccess.thecvf.com/content/CVPR2024/papers/Chen_Panda-70M_Captioning_70M_Videos_with_Multiple_Cross-Modality_Teachers_CVPR_2024_paper.pdf — Panda-70M（官方一手；31 个候选打标器 = 6 基座 × 权重/模态变体、贪心集合覆盖选出 8 个覆盖 76.8%、UMT-large 细粒度检索择优、人类一致率仅 44.9%）
- https://arxiv.org/abs/2510.10395 — AVoCaDO（官方一手；Qwen2.5-Omni-7B 基座、AVoCaDO-SFT-107K 六源构成、五维 keypoint checklist、GRPO 三项 reward 含对白编辑距离 DP 对齐阈值 0.6）
- https://github.com/AVoCaDO-Captioner/AVoCaDO — AVoCaDO 官方仓库与 Apache-2.0 权重（官方一手）
- https://arxiv.org/abs/2607.12820 — AVSCap + AVSCapBench（官方一手，2026-07-14；三准则 Acoustic/Visual Completeness + AV Synergy、AVSCap-130K = 4 万视频 × 3 份标注、SFT→GRPO、AVSCapBench 1226 视频细粒度 event recall 横向对比表）
- https://github.com/NJU-LINK/AVSCap — AVSCap 官方仓库（官方一手；训练集尚未释放）
- https://arxiv.org/pdf/2506.15220 — video-SALMONN 2（官方一手，清华电子系 + 字节；LLaVA-OneVision + 音频 LoRA、3B/7B/72B、MrDPO 多轮 DPO、caption 错误率相对基线降 28%）
- https://github.com/ddlBoJack/Omni-Captioner — Omni-Detective / Omni-Captioner / Omni-Cloze（官方一手，ICLR 2026；agentic Query-Observation 循环标注框架，音频版即 Qwen3-Omni-Captioner，音视频版并入 Qwen3.5-Omni）
- https://arxiv.org/html/2604.15804v2 — Qwen3.5-Omni 技术报告（官方一手；明确把「剧本级细粒度描述、自动切片、时间戳打标、人物与音频关系描述」定位为视频合成模型训练数据生成能力；Omni-Cloze Plus 64.8 > Gemini-3.1 Pro 57.2）
- https://arxiv.org/html/2601.03233v1 — LTX-2（官方一手；第 5.1 节自研音视频 captioner，字段含 camera motion / lighting / subject behavior + music / ambient / 对白转写含 speaker-language-accent，原则为 comprehensive yet factual、显式禁止情绪解读）
- https://arxiv.org/html/2502.12782v1 — VidCapBench（第三方基准；面向可控 T2V 的视频 caption 评测）
- https://arxiv.org/abs/2606.08572 — OmniCap-IF（第三方/学术；首个音视频 caption 指令跟随基准，50 类约束 + format/content 双维评分）
- https://arxiv.org/html/2507.11336 — UGC-VideoCaptioner + UGC-VideoCap（第三方/学术；1000 条 TikTok 视频 + 4000 QA，三阶段 human-in-the-loop 分别标 audio-only / visual-only / joint AV）
- https://arxiv.org/abs/2502.11775 — video-SALMONN-o1（官方一手；pDPO 过程级偏好优化 + RivaBench，定位 QA/reasoning 而非 captioner）
- https://huggingface.co/AVoCaDO-Captioner/AVoCaDO — AVoCaDO 权重（官方一手）
- https://arxiv.org/html/2412.09283v1 — InstanceCap（第三方/学术；实例感知结构化 caption 提升 T2V）
- https://www.marktechpost.com/2025/01/15/bytedance-researchers-introduce-tarsier2-a-large-vision-language-model-lvlm-with-7b-parameters-designed-to-address-the-core-challenges-of-video-understanding/ — Tarsier2 报道（第三方报道）
- 本仓库同批调研的 30 份生成模型条目（Movie_Gen.json / HunyuanVideo.json / Seedance_20_Seedance_15_pro.json / Open-Sora.json / SkyReels.json / MOVA.json / LTX-2.json / CogVideoX.json / Allegro.json / Goku.json / LongCat-Video.json / Step-Video-T2V.json / Sora_2.json / Veo_3_Veo_31.json / pretraining_datasets.json 等）的 caption_model 与 data_ablation 字段 — 生成侧对 captioner 选型的一手技术报告转述（同项目旁证）

### [几何/结构化标注数据集合集](../models/geometric_datasets.md)

- url: https://arxiv.org/abs/2604.07990 | title: SceneScribe-1M: A Large-Scale Video Dataset with Comprehensive Geometric and Semantic Annotations（CVPR 2026接收） | nature: 官方一手（论文摘要页）
- url: https://arxiv.org/html/2604.07990v2 | title: SceneScribe-1M 论文全文HTML版（含数据源、过滤规格、MegaSaM/TAPIP3D 标注方法、六项下游消融表） | nature: 官方一手（论文正文）
- url: https://wangyunnan.github.io/SceneScribe-1M | title: SceneScribe-1M 官方项目主页 | nature: 官方一手（项目页）
- url: https://arxiv.org/abs/2509.09676 | title: SpatialVID: A Large-Scale Video Dataset with Spatial Annotations（arXiv v1 2025-09-11, v2 2025-12-18） | nature: 官方一手（论文摘要页）
- url: https://arxiv.org/html/2509.09676v2 | title: SpatialVID 论文全文HTML版（含四维过滤阈值、MegaSaM+UniDepth v2+Depth Anything v2、SAM2动态掩膜、双阶段caption、HQ子集统计） | nature: 官方一手（论文正文）
- url: https://huggingface.co/datasets/SpatialVID/SpatialVID-HQ | title: SpatialVID-HQ HuggingFace 数据集页（CC-BY-NC-SA 4.0，3.53TB，74分组，标注字段清单） | nature: 官方一手（数据集发布页）
- url: https://arxiv.org/abs/2603.23497 | title: WildWorld: A Large-Scale Dataset for Dynamic World Modeling with Actions and Explicit State toward Generative ARPG | nature: 官方一手（论文摘要页）
- url: https://arxiv.org/html/2603.23497v1 | title: WildWorld 论文全文HTML版（含采集平台、119列逐帧标注、动作三元组统计、WildBench四维评测、五种条件方法消融） | nature: 官方一手（论文正文）
- url: https://github.com/ShandaAI/WildWorld | title: WildWorld 官方 GitHub 仓库（盛大AI） | nature: 官方一手（代码仓库）
- url: https://arxiv.org/abs/2601.10592 | title: Action100M: A Large-scale Video Action Dataset | nature: 官方一手（论文摘要页）
- url: https://arxiv.org/html/2601.10592v1 | title: Action100M 论文全文HTML版（含V-JEPA 2分割、Tree-of-Captions五字段、三模型caption链、GPT-OSS-120B Self-Refine、GPU小时成本、16个下游基准） | nature: 官方一手（论文正文）
- url: https://github.com/facebookresearch/Action100M | title: Action100M 官方 GitHub 仓库（Meta FAIR） | nature: 官方一手（代码仓库）

### [视频生成后训练数据策略](../models/post_training_data.md)

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

### [主流视频预训练数据集合并调研：Panda-70M、InternVid、Koala…](../models/pretraining_datasets.md)

【官方一手·论文】1) Panda-70M arXiv:2402.19479 https://arxiv.org/html/2402.19479v1 与 CVPR2024 开放版 https://openaccess.thecvf.com/content/CVPR2024/papers/Chen_Panda-70M_Captioning_70M_Videos_with_Multiple_Cross-Modality_Teachers_CVPR_2024_paper.pdf ；2) InternVid arXiv:2307.06942 https://arxiv.org/html/2307.06942v2 ；3) Koala-36M arXiv:2410.08260 https://arxiv.org/html/2410.08260v2 （v2 含附录A–G，全部阈值出处）；4) MiraData arXiv:2407.06358 https://arxiv.org/pdf/2407.06358 与 NeurIPS 2024 camera-ready https://proceedings.neurips.cc/paper_files/paper/2024/file/57f6683e550eb067936c9e9f0bcb8e31-Paper-Datasets_and_Benchmarks_Track.pdf ；5) OpenVid-1M arXiv:2407.02371 https://arxiv.org/html/2407.02371v2 与 OpenReview https://openreview.net/forum?id=j7kdXSrISM ；6) UltraVideo arXiv:2506.13691 https://arxiv.org/html/2506.13691v1 与 NeurIPS 2025 poster #121373 / OpenReview zYqM6gkqBi ；7) LVD-2M arXiv:2410.10816 https://arxiv.org/html/2410.10816v1 。
【官方一手·代码与数据（含仅能从代码得到的关键参数）】8) https://github.com/snap-research/Panda-70M （splitting/captioning README、dataset_dataloading README 中的 10M/2M 筛选口径、panda70m.yaml 的 download_size:360 与 download_audio:True）；9) https://github.com/OpenGVLab/InternVideo/tree/main/Data/InternVid 与 https://huggingface.co/datasets/OpenGVLab/InternVid ；10) https://github.com/KlingAIResearch/Koala-36M （原 KwaiVGI/Koala-36M）中的 trainsition_detect/VideoTransitionAnalyzer.py——**SVM 系数、Canny 阈值、3σ 门限、±4帧腐蚀等仅存在于代码而不在论文中**；training_suitability_assessment/test.yml 暴露了发布版打分器实为 fragments-only；11) https://github.com/mira-space/MiraData/tree/v1 的 caption_gpt4v.py（GPT-4V 完整 prompt 原文）与 calculate_score.py（MiraBench 实际输出19个指标）；12) https://github.com/NJU-PCALab/OpenVid-1M 与 https://huggingface.co/datasets/nkp37/OpenVid-1M ；13) https://github.com/xzc-zju/UltraVideo 与 https://huggingface.co/datasets/APRIL-AIGC/UltraVideo （license-april-lab.txt）；14) https://github.com/SilentView/LVD-2M 与 S3 元数据 https://ic-cv-long-videos.s3.ap-northeast-2.amazonaws.com/LVD-2M/data/ 。
【同团队旁证/上游】15) HD-VILA-100M https://github.com/microsoft/XPretrain/tree/main/hd-vila-100m 与 arXiv:2111.10337——Panda-70M、Koala-36M、LVD-2M（经 HD-VG）共同的上游语料与许可来源。
【第三方/交叉互评（重要：各数据集论文互相批评，是本调研的核心证据链）】16) Koala-36M 论文 Table 1 与相关工作对 Panda-70M「caption 简短不完整、转场导致语义不一致」的批评；17) OpenVid-1M 论文对 WebVid-10M/Panda-70M「过度追求规模导致低质视频与短而不准 caption」的批评；18) MiraData 论文对 Panda-70M「未系统考虑正确切分、画质过滤与准确文本标注」的批评；19) UltraVideo 论文 Table 1 提供了七者中六者的统一口径对比（本调研多处交叉校验依赖该表）；20) LVD-2M 论文对 InternVid caption「缺乏时序动态」的批评；21) VidGen-1M arXiv:2408.02629（「coarse-to-fine 精修 Panda-70M」，佐证原始 Panda-70M 噪声过大）；22) 长视频生成综述 arXiv:2412.18688（**注意：该综述称 MiraData 为「77k long videos」系误引 v0 beta 数据，勿采用**）。
【社区反馈】23) Panda-70M GitHub issues #16/#41/#60/#62/#64/#65（YouTube 下载失败、私有化、需要种子镜像）；24) InternVideo issue #81（InternVid 下载困难）；25) OpenVid-1M issues #11/#21（zip 分片与视频对应关系缺失，社区自建 phil329/OpenVid-1M-mapping 补救；HD 子集下载方式混乱）；26) UltraVideo issues #3（caption prompt 未公开，关闭且无回复）/#5（108个主题与视频的映射关系缺失，未回复）/#6（VBench 子集构成，未回复）/#8（训练与清洗代码，未发布）。
