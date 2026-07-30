# Side-by-Side Comparison: Basic Information

> Topic: Data processing for video generation models (including simultaneous audio-video generation): data filtering funnel, data distribution, annotation methods

[← Back to homepage](../index.md)

This page compares all entries side by side, field by field. ⚠️ indicates that some information for this entry's field is uncertain.

**Fields**: [Name](#name) · [Publishing organization/company](#publishing-organizationcompany) · [Release date (technical report/paper/open-source date)](#release-date-technical-reportpaperopen-source-date) · [Type (model/dataset/toolchain/evaluation benchmark)](#type-modeldatasettoolchainevaluation-benchmark) · [Degree of openness (whether weights/code/data/pipeline are each open-sourced)](#openness-whether-weightscodedatapipeline-are-each-open-sourced) · [Whether simultaneous audio-video generation is supported, and the implementation approach (native joint/cascaded/MoE fusion)](#whether-simultaneous-audio-video-generation-is-supported-and-the-implementation-approach-native-joint--cascade--moe-fusion) · [List of research information sources (URLs of papers/technical reports/official documentation/news, annotated with the nature of each source: official first-party/same-team corroboration/third-party reporting)](#research-information-source-list-urls-of-paperstechnical-reportsofficial-documentationnews-with-each-sources-nature-labeled-official-primary--same-team-corroboration--third-party-report)

## Name

`name` · Level of detail: minimal

### [Allegro](../models/Allegro.md)

Allegro (including the subsequent Allegro-TI2V image-to-video version, as well as the 40×720P / 40×360P variants; the accompanying annotation model is Aria, a multimodal MoE model from the same team)

### [Apollo](../models/Apollo.md)

Apollo (renamed to Klear starting from arXiv v2; the same paper arXiv:2601.04151, v1 titled "Apollo: Unified Multi-Task Audio-Video Joint Generation", v2 titled "Klear: Unified Multi-Task Audio-Video Joint Generation". Note this is unrelated to same-named works such as Meta's video understanding model Apollo or Meta's Apollo LMM — the arXiv number must be used to disambiguate)

### [CineDance / CineDance-1M](../models/CineDance.md)

CineDance / CineDance-1M (paper title: CineDance: Towards Next-Generation Multi-Shot Long-Form Cinematic Audio-Video Generation). This work comprises three outputs: the CineDance-1M dataset (1 million multi-shot, long-form audio-video sequences), the CineBench evaluation benchmark (1,000 test samples plus a six-dimensional metric system), and the CineDance generation model (an open-source baseline adapted from LTX-2.3).

### [CogVideoX](../models/CogVideoX.md)

CogVideoX (including CogVideoX-2B / 5B, CogVideoX1.5-5B / 5B-I2V, plus the accompanying CogVLM2-Caption annotation model; on the product side this corresponds to "Qingying" (清影 Ying), and on the sound side to CogSound)

### [Cosmos-Predict2.5](../models/Cosmos-Predict25.md)

Cosmos-Predict2.5 (the latest generation of NVIDIA's Cosmos world foundation model family; paper "World Simulation with Video Foundation Models for Physical AI", arXiv:2511.00062; the Cosmos-Transfer2.5 ControlNet family was released alongside the same paper)

### [Data-Juicer 2.0](../models/Data-Juicer.md)

Data-Juicer 2.0 (including the Data-Juicer Sandbox experimental lab). Positioned as a one-stop data processing system and cloud-scale adaptive operator library "for and with foundation models"; the full paper title is "Data-Juicer 2.0: Cloud-Scale Adaptive Data Processing for and with Foundation Models". The accompanying Sandbox component's full paper title is "Data-Juicer Sandbox: A Feedback-Driven Suite for Multimodal Data-Model Co-development", which is a data-model co-development middleware that closes the loop between "data recipes" and "model training/evaluation". In this survey of video-generation data processing, Data-Juicer's role is not that of a generative model but rather that of underlying data cleaning/annotation operator infrastructure reused by multiple teams.

### [Foley-Omni](../models/Foley-Omni.md)

Foley-Omni (accompanying evaluation benchmark: V2ST-Bench). The full paper title is "Foley-Omni: A Unified Multimodal Generation Model from Task-Level Audio Synthesis to Complete Video Soundtrack Generation", i.e., a unified multimodal generation model advancing from task-level audio synthesis toward complete Video-to-Soundtrack (V2ST) generation. Its core positioning is to jointly model speech, sound effects/foley, and music within a single latent-space generation process.

### [Goku](../models/Goku.md)

Goku ("Goku: Flow Based Video Generative Foundation Models", including the Goku-T2I / Goku-T2V / Goku-I2V series; arXiv:2502.04896, CVPR 2025 Highlight)

### [Hailuo / MiniMax Video](../models/Hailuo.md)

Hailuo / MiniMax Video (海螺AI视频). This is a product line rather than a single model; successive model IDs include: video-01 (September 2024, first generation, also known as Hailuo), video-01-live (Live2D/anime-specialized), video-01-director (camera motion control), S2V-01 (Subject-Reference), MiniMax-Hailuo-02 (June 2025), MiniMax-Hailuo-2.3 / 2.3-Fast (October 2025, still the latest online version in official documentation as of July 2026)

### [HunyuanVideo-Foley](../models/HunyuanVideo-Foley.md)

HunyuanVideo-Foley (Hunyuan Video Foley/sound-effects model; full title "HunyuanVideo-Foley: Multimodal Diffusion with Representation Alignment for High-Fidelity Foley Audio Generation")

### [HunyuanVideo](../models/HunyuanVideo.md)

HunyuanVideo (混元视频, the original 2024 13B version) + HunyuanVideo 1.5 (混元视频1.5, 8.3B)

### [InstructAV2AV](../models/InstructAV2AV.md)

InstructAV2AV (accompanying dataset: InsAVE-80K). Full paper title: "InstructAV2AV: Instruction-Guided Audio-Video Joint Editing", i.e., instruction-guided joint audio-video editing. Its core positioning is as the first end-to-end framework for "editing both video imagery and its accompanying audio track using text instructions alone" — while changing the specified visual target and the sound it produces, it strictly preserves the background footage and ambient sound in non-target regions. Its representative dimension in this survey's genealogy is "synthetic data": the paper constructs a scalable data synthesis pipeline (data engine) that manufactures paired source→target samples via controlled editing, addressing the fundamental problem that "paired supervision data simply does not exist" in the audio-video editing domain.

### [2026 Miscellaneous Joint Audio-Video Generation Works](../models/JAVG_2026_misc.md)

A 2026 collection of miscellaneous joint audio-video generation (JAVG) works — a merged survey of seven works: (1) Baton, "Baton: Explicit Semantic Blueprints for Joint Video-Audio Generation", arXiv:2605.25195; (2) OmniCustom, "OmniCustom: Sync Audio-Video Customization Via Joint Audio-Video Generation Model", arXiv:2602.12304 (including its self-built dataset OmniCustom-1M); (3) StreamChar, "StreamChar: Long-Horizon Streaming Character Audio-Video Generation with Decoupled Orchestration", arXiv:2605.25659; (4) ALIVE, "ALIVE: Animate Your World with Lifelike Audio-Video Generation", arXiv:2602.08682 (including Alive-Bench 1.0); (5) CCL, "Improving Joint Audio-Video Generation with Cross-Modal Context Learning", arXiv:2603.18600; (6) NAVA, "Native Audio-Visual Alignment for Generation", arXiv:2605.30073; (7) ITS-JAVG, "Inference-Time Scaling for Joint Audio-Video Generation", arXiv:2606.03183. This entry only extracts data-processing-related content; the seven works differ enormously in the depth of their data disclosure: ALIVE and NAVA are industrial-grade heavy disclosures, OmniCustom is a medium-disclosure dataset-construction type, Baton/CCL/StreamChar disclose only data sources and scale, and ITS-JAVG is fully training-free with no training data.

### [Joint Audio-Video Generation Baseline Collection](../models/JavisDiT_baselines.md)

Joint audio-video generation baseline collection (a merged survey of 4 works):
(1) JavisDiT / JavisDiT++ — "JavisDiT: Joint Audio-Video Diffusion Transformer with Hierarchical Spatio-Temporal Prior Synchronization" (arXiv:2503.23377) and its sequel "JavisDiT++: Unified Modeling and Optimization for Joint Audio-Video Generation" (arXiv:2602.19163, ICLR 2026), accompanied by the evaluation benchmark JavisBench / JavisBench-mini and the synchronization metric JavisScore;
(2) MM-Diffusion — "MM-Diffusion: Learning Multi-Modal Diffusion Models for Joint Audio and Video Generation" (arXiv:2212.09478, CVPR 2023), the pioneering work in joint generation, accompanied by the self-built Landscape dataset;
(3) AV-DiT — "AV-DiT: Efficient Audio-Visual Diffusion Transformer for Joint Audio and Video Generation" (arXiv:2406.07686);
(4) Harmony — "Harmony: Harmonizing Audio and Video Generation through Cross-Task Synergy" (arXiv:2511.21579), accompanied by the evaluation benchmark Harmony-Bench;
(5) UniAVGen — "UniAVGen: Unified Audio and Video Generation with Asymmetric Cross-Modal Interactions" (arXiv:2511.03334).
Among these, (2) and (3) are early small-scale academic baselines, (1) is a mid-period open-source academic baseline, and (4) and (5) are recent strong baselines from late 2025.

### [Kling 3.0 / Kling Video 3.0 Omni](../models/Kling_30_Omni.md)

可灵 3.0 / 可灵视频 3.0 Omni (Kling 3.0 / Kling Video 3.0 Omni, internally also called Kling O3)

### [LTX-2](../models/LTX-2.md)

LTX-2 (including the subsequent LTX-2.3; technical report "LTX-2: Efficient Joint Audio-Visual Foundation Model")

### [LongCat-Video](../models/LongCat-Video.md)

LongCat-Video (Meituan's LongCat team's video generation foundation model, technical report arXiv:2510.22200; same-series derivatives LongCat-Video-Avatar and LongCat-Video-Avatar 1.5, technical report arXiv:2605.26486)

### [MOVA](../models/MOVA.md)

MOVA (MOSS Video and Audio)

### [Merged entry: ① Mochi 1](../models/Mochi_MAGI_Motif.md)

Merged entry: ① Mochi 1 (including mochi-1-preview and AsymmVAE) ② MAGI-1 (including the 24B / 4.5B variants, the Distill distilled version, and the 2026 MAGI-1.1 24B) ③ Motif-Video 2B (including T2V and I2V extensions). All three are open-source, purely visual video generation foundation models; the core value of comparing them side by side is that the granularity of data disclosure shows a clear evolution curve from 2024 to 2026: "almost zero" (Mochi 1) → "detailed at the method level but without numbers" (MAGI-1) → "detailed at both the tool level and the parameter level, and explicitly winning through small data" (Motif-Video 2B).

### [Movie Gen](../models/Movie_Gen.md)

Movie Gen (including Movie Gen Audio)

### [NVIDIA NeMo Curator](../models/NeMo_Curator.md)

NVIDIA NeMo Curator (versions 26.02 / 26.04) + Cosmos-Xenna (the underlying distributed execution engine on the video side), together with the same-lineage productized implementation Cosmos-Curate (the training-data generation system for the Cosmos world foundation models)

### [OmniHuman Dataset + OHBench](../models/OmniHuman.md)

OmniHuman Dataset + OHBench (OmniHuman Benchmark) — a large-scale dataset and three-tier evaluation benchmark for human-centric audio-video generation

### [Open-Sora Series](../models/Open-Sora.md)

The Open-Sora series (Open-Sora 1.0/1.1/1.2/1.3/2.0, HPC-AI Tech) and the Open-Sora Plan series (v1.0–v1.5, Peking University's PKU-YuanGroup). These are two independent projects, often jointly referred to as the "twin heroes of open-source Sora reproduction": Open-Sora is led by HPC-AI Tech (潞晨科技), championing extreme low-cost training (the 2.0 version cost $200,000); Open-Sora Plan is led by Peking University's Yuan Li (袁粒) team, championing community-collaborative reproduction and a multi-dimensional data-cleaning funnel. This entry surveys both together, itemizing separately wherever field content differs.

### [Ovi](../models/Ovi.md)

Ovi (paper "Ovi: Twin Backbone Cross-Modal Fusion for Audio-Video Generation", arXiv:2510.01284; subsequent iteration Ovi 1.1; the audio tower is separately named Ovi-Aud)

### [Script-a-Video](../models/Script-a-Video.md)

Script-a-Video (the core output is the MTSS representation paradigm, full name Multi-Stream Scene Script)

### [Seedance 2.0 and Seedance 1.5 pro](../models/Seedance_20_Seedance_15_pro.md)

Seedance 2.0 and Seedance 1.5 pro (including Seedance 1.0 as a vertical baseline for the data pipeline)

### [SkyReels Series](../models/SkyReels.md)

The SkyReels series (this entry covers SkyReels-V2, "SkyReels-V2: Infinite-length Film Generative Model", arXiv:2504.13074, and SkyReels-V4, "SkyReels-V4: Multi-modal Video-Audio Generation, Inpainting and Editing model", arXiv:2602.21818; the intermediate version SkyReels-V3 was open-sourced on January 29, 2026, featuring reference-image-to-video R2V, video extension V2V, and audio-driven digital-human Talking Avatar as its three major capabilities, serving as a transition from V2 to V4)

### [Sora 2](../models/Sora_2.md)

Sora 2 (including Sora 2 Pro)

### [SpeakerVid-5M](../models/SpeakerVid-5M.md)

SpeakerVid-5M (full paper title: SpeakerVid-5M: A Large-Scale High-Quality Dataset for Audio-Visual Dyadic Interactive Human Generation)

### [Step-Video-T2V](../models/Step-Video-T2V.md)

Step-Video-T2V (阶跃视频, a 30B text-to-video foundation model) and its derivatives Step-Video-T2V-Turbo and Step-Video-TI2V

### [UniTalking](../models/UniTalking.md)

UniTalking (a unified audio-video talking-portrait generation framework)

### [UniVerse-1](../models/UniVerse-1.md)

UniVerse-1 (including the accompanying evaluation benchmark Verse-Bench)

### [Unison](../models/Unison.md)

Unison (full paper title "Unison: Harmonizing Motion, Speech, and Sound for Human-Centric Audio-Video Generation", literally "Unison: coordinating motion, speech, and sound for human-centric audio-video generation"). The name is taken from the musical term "unison" (齐奏/同度), echoing the paper's core claim — that motion, speech, and sound effects evolve "in Unison" (in step) with one another. Note the distinction from a contemporaneous same-named work: arXiv:2605.31530 "UNISON: A Unified Sound Generation and Editing Framework via Deep LLM Fusion" is a purely audio generation/editing work and is unrelated to this entry.

### [Veo 3 / Veo 3.1](../models/Veo_3_Veo_31.md)

Veo 3 / Veo 3.1 (including Veo 3.1 Lite)

### [Vidu S1](../models/Vidu_S1.md)

Vidu S1 (technical report "Vidu S1: A Real-Time Interactive Video Generation Model", arXiv:2607.03118v2; product name Vidu Stream, https://vidu.com/vidu-stream)

### [Wan 2.5 / 2.6 / 2.7](../models/Wan.md)

Wan 2.5 / 2.6 / 2.7 (通义万相, the closed-source commercial versions of the Wan series) — including its open-source predecessors Wan 2.1 (arXiv:2503.20314), Wan 2.2 (Apache 2.0), and same-team corroborating models Wan2.2-S2V (arXiv:2508.18621), Wan2.2-Animate, and Wan-Dancer (arXiv:2607.09581)

### [Audio-Video Generation Evaluation Benchmark Collection](../models/av_benchmarks.md)

Audio-video generation evaluation benchmark collection (VABench / AVBench / AV-SyncBench / PhyAVBench / Omni-Judge)

### [Video Captioning Model Ecosystem](../models/caption_models.md)

Video captioning model ecosystem (Video Captioner Ecosystem) — a merged survey entry covering ShareGPT4Video / ShareCaptioner-Video, the Tarsier & Tarsier2 series, CogVLM2-Caption, SkyCaptioner-V1, AVoCaDO, AVSCap, video-SALMONN 2, Qwen3-Omni-Captioner / Qwen3.5-Omni, AuroraCap, the Panda-70M multi-teacher captioner, LLaVA-Video / PLLaVA, Aria, Tag2Text, and others, summarizing their practical roles within the video-generation data pipeline. This entry is not a single model/dataset but a horizontal ecosystem map of the "captioner/annotator" pipeline component.

### [Geometric/Structured Annotation Dataset Collection](../models/geometric_datasets.md)

Geometric/structured annotation dataset collection (SceneScribe-1M / SpatialVID / WildWorld / Action100M)

### [Post-Training Data Strategies for Video Generation](../models/post_training_data.md)

Post-training data strategies for video generation (a cross-model horizontal topic) — anchored on "A Systematic Post-Train Framework for Video Generation" (arXiv:2604.25427), horizontally summarizing each model's SFT curated-set scale/selection criteria and preference-pair annotation methods

### [Merged Survey of Mainstream Video Pretraining Datasets: Panda-70M, InternVid, Koala…](../models/pretraining_datasets.md)

A merged survey of mainstream video pretraining datasets: Panda-70M, InternVid, Koala-36M, MiraData, OpenVid-1M, UltraVideo, LVD-2M (7 public video-text datasets in total, spanning 2023–2025, with an emphasis on comparing cleaning methods and captioning strategies)

## Publishing organization/company

`organization` · Level of detail: minimal

### [Allegro](../models/Allegro.md)

Rhymes AI (瑞莱斯智能 / Rhymes AI, an AI startup with a Singapore–Hong Kong background, the same team behind the Aria multimodal model). Paper authors: Yuan Zhou, Qiuyue Wang, Yuxuan Cai, Huan Yang (Huan Yang is the corresponding/lead author, formerly a researcher at Microsoft Research Asia)

### [Apollo](../models/Apollo.md) ⚠️

Kuaishou Technology (快手科技) Kling Team (可灵团队). Authors: Jun Wang, Chunyu Qiang, Yuxin Guo, Yiran Wang, Xijuan Zeng, Feng Deng, and others. This work is the research-oriented technical report behind Kuaishou Kling's (可灵) joint audio-video synchronization capability (Kling 2.6's "simultaneous audio-picture generation", Kling 3.0), representing an external paper disclosure from an industry closed-source model. [Uncertain] (the paper itself does not explicitly state its correspondence to the Kling product line; this association is an inference from the corroborating evidence of the HuggingFace paper page's byline "Kling Team, Kuaishou Technology" and the product timeline)

### [CineDance / CineDance-1M](../models/CineDance.md) ⚠️

A joint effort across multiple academic institutions, with no corporate lab byline. Participating units include: Shanghai Jiao Tong University, University of Electronic Science and Technology of China, Zhejiang University, The University of Tokyo, and Nanyang Technological University. Author list: Yuheng Chen, Teng Hu, Yuji Wang, Qingdong He, Zhucun Xue, Qianyu Zhou, Jason Li, Lizhuang Ma, Jiangning Zhang, Dacheng Tao. The project homepage is maintained by first author Yuheng Chen (github.com/AliothChen). The specific institutional affiliation of each author is not explicitly listed on the page. [Uncertain]

### [CogVideoX](../models/CogVideoX.md)

A joint team from Zhipu AI (智谱 AI) and Tsinghua University's THUDM; the paper's corresponding author is Jie Tang (唐杰); core contributors include Zhuoyi Yang, Jiayan Teng, Wendi Zheng, Ming Ding, Shiyu Huang, Xiaotao Gu, and others

### [Cosmos-Predict2.5](../models/Cosmos-Predict25.md)

NVIDIA (the paper is credited to NVIDIA itself, a large collaborative team with 88 contributors, including authors such as Arslan Ali, Junjie Bai, Sanja Fidler; affiliated with the NVIDIA Cosmos Lab / Physical AI direction)

### [Data-Juicer 2.0](../models/Data-Juicer.md)

Alibaba Group. The initiating team is the data direction team at Alibaba's Tongyi Lab (阿里巴巴通义实验室), jointly built with Alibaba Cloud PAI (Platform for AI), and collaborating with external units including Anyscale (the Ray team), NVIDIA, Sun Yat-sen University, and others. Core authors: Daoyuan Chen (陈道源, first/corresponding author, homepage yxdyc.github.io), Yilun Huang, Xuchen Pan, Nana Jiang, Haibin Wang, Yilei Zhang, Ce Ge, Yushuo Chen, Wenhao Zhang, Zhijian Ma, Jun Huang, Wei Lin, Yaliang Li (李雅亮), Bolin Ding (丁博麟), Jingren Zhou (周靖人). The Sandbox paper's authors are Daoyuan Chen, Haibin Wang, Yilun Huang, Ce Ge, Yaliang Li, Bolin Ding, Jingren Zhou. The GitHub organization has migrated from modelscope/data-juicer to datajuicer/data-juicer.

### [Foley-Omni](../models/Foley-Omni.md)

Led by the School of Intelligence Science and Technology, Nanjing University, jointly with Video Rebirth (an industry collaborator), Shanghai Jiao Tong University, Beijing Jiaotong University, and Shanghai AI Laboratory. Author list: Ye Tao (陶烨, first author, contact email taoye0402@gmail.com, maintainer of the project homepage ty0402.github.io), Lupeng Liu, Xuenan Xu (徐雪男, a researcher in the audio description/audio-captioning direction), Jiasun Feng, Jiarui Wang, Ying Qin, Shuiyang Mao, Wei Liu, Shuai Wang (王帅, in the corresponding-author direction, from Nanjing University's speech group). The GitHub organization is named NJU-Speech, and the HuggingFace account is CocoBro.

### [Goku](../models/Goku.md)

A joint effort between ByteDance and the University of Hong Kong (HKU). The paper is credited to 22 authors; first author Shoufa Chen (HKU); corresponding/senior authors include Ping Luo (HKU), Yi Jiang, Zehuan Yuan, Bingyue Peng, Xiaobing Liu (ByteDance). The team overlaps heavily with ByteDance Seed / the visual generation line; the project's code name derives from Goku of Dragon Ball (the organization is named Saiyan-World).

### [Hailuo / MiniMax Video](../models/Hailuo.md)

MiniMax (稀宇科技 / Shanghai Xiyu Extreme Intelligence Technology Co., Ltd.), Shanghai, China. Other product lines from the same company include the MiniMax-01/M1/M2/M2.5/M2.7/M3 series of large language models, MiniMax Speech, MiniMax Music, and MiniMax Code. The video product's public-facing brand is "Hailuo AI" (海螺AI, hailuoai.video / hailuoai.com)

### [HunyuanVideo-Foley](../models/HunyuanVideo-Foley.md)

Led by Tencent Hunyuan (腾讯混元), jointly with Zhejiang University and Nanjing University of Aeronautics and Astronautics. Author list: Sizhe Shan (单思哲, first author), Qiulin Li, Yutao Cui, Miles Yang, Yuehai Wang, Qun Yang, Jin Zhou, Zhao Zhong. The project homepage is maintained by szczesnys. It is positioned as the model that completes the "sound" component of Tencent Hunyuan's open-source matrix following HunyuanVideo (video generation); it shares the brand with the HunyuanVideo series but not the model architecture.

### [HunyuanVideo](../models/HunyuanVideo.md)

The Tencent Hunyuan Foundation Model Team

### [InstructAV2AV](../models/InstructAV2AV.md)

A joint effort between the Beijing Academy of Artificial Intelligence (BAAI, 北京智源人工智能研究院) and Peking University (北京大学). Author list: Haojie Zheng (郑浩杰, first author, dually affiliated with BAAI and PKU, personal homepage hjzheng.net), Yixin Yang (PKU), Siqi Yang (PKU), Shuchen Weng (翁书宸, dually affiliated with BAAI and PKU), Boxin Shi (施柏鑫, PKU School of Computer Science, corresponding author, in the vision and computational photography direction). The code repository is hosted under the personal GitHub account suimuc/InstructAV2AV, and the HuggingFace account is suimu.

### [2026 Miscellaneous Joint Audio-Video Generation Works](../models/JAVG_2026_misc.md) ⚠️

The seven works belong to different institutions:
(1) Baton: a joint effort between Fudan University (Shuyuan Tu, Zuxuan Wu, Yu-Gang Jiang, the Vision and Learning Lab) and Tencent Hunyuan (authors including Weijie Kong, Jiangfeng Xiong, Zhao Zhong and other members of the Hunyuan video generation team), with additional participation from Liefeng Bo (with a background at Alibaba Tongyi/DAMO Academy) and Qi Tian and Xintong Han [institutional affiliation is inferred from the author list, uncertain].
(2) OmniCustom: led by the University of Hong Kong (Guosheng Yin, Dong Xu), jointly with Shanda AI Research Tokyo (盛大 AI 研究院东京), XIntelligence Technology Co., Limited, and Shanghai AI Laboratory (Kaipeng Zhang); authors Maomao Li and Zhifeng Li have a Tencent AI Lab background.
(3) StreamChar: Alibaba's Tongyi Lab (阿里巴巴通义实验室), authors Linrui Tian, Qi Wang, Bang Zhang — i.e., the EMO / Wan-S2V digital-human team — the project page is hosted under the HumanAIGC organization.
(4) ALIVE: the ByteDance ALIVE Team, 16 authors, including Xiang Yin (in the Seed speech direction), Bingyue Peng, Zehuan Yuan, and others; the repository is under the FoundationVision organization.
(5) CCL: from SenseTime (商汤科技), authors Bingqi Ma, Guanglu Song, Yu Liu, Dailan He, and others [institution inferred from the authors, not explicitly listed on the paper page, uncertain].
(6) NAVA: Baidu ERNIE Research (文心研究团队), authors Longbin Ji, Guan Wang, Xuan Wei, Shuohuan Wang, Yu Sun, and others; weights are hosted on HuggingFace under both the baidu and ernie-research organizations.
(7) ITS-JAVG: KAIST (Korea Advanced Institute of Science and Technology), authors Jaemin Jung, Kyeongha Rho, Inkyu Shin, Joon Son Chung (Joon Son Chung is one of the authors of SyncNet, an authority in the audio-video synchronization field).

### [Joint Audio-Video Generation Baseline Collection](../models/JavisDiT_baselines.md)

(1) JavisDiT / JavisDiT++: led by the National University of Singapore (NUS, Hao Fei, Shengqiong Wu, Tat-Seng Chua, Wei Li, and others), jointly with Xiamen University (Jiayi Ji), Fudan University (Fan Zhou), the University of Rochester (Jiebo Luo), Nanyang Technological University (Ziwei Liu), and others; first author Kai Liu; the community organization is named JavisVerse.
(2) MM-Diffusion: a joint effort between Renmin University of China (Ludan Ruan, Qin Jin) and Microsoft Research Asia (MSRA) (Huan Yang, Bei Liu, Jianlong Fu, Nicholas Jing Yuan, Baining Guo), with additional participation from Peking University; the GitHub organization is researchmm (Microsoft Research Asia's multimedia group).
(3) AV-DiT: a joint effort between the University of Toronto (Kai Wang, Dimitrios Hatzinakos), UT Dallas (Shijian Deng, Yapeng Tian), and Adobe Research (Jing Shi).
(4) Harmony: a joint effort between Shanghai Jiao Tong University (Teng Hu, Ran Yi) and Tencent Hunyuan (Zhentao Yu, Guozhen Zhang, Zhengguang Zhou, Youliang Zhang, Yuan Zhou, Qinglin Lu).
(5) UniAVGen: led by Nanjing University (State Key Laboratory for Novel Software Technology, Guozhen Zhang, Limin Wang) and Tencent Hunyuan (Zixiang Zhou, Yi Chen, Yuan Zhou, Qinglin Lu), jointly with Shanghai Jiao Tong University (Teng Hu), Renmin University of China (Ziqiao Peng), Tsinghua University (Youliang Zhang), and Shanghai AI Laboratory.
Note: Harmony and UniAVGen have heavily overlapping authorship (Guozhen Zhang, Teng Hu, Youliang Zhang, Yuan Zhou, and Qinglin Lu all appear in both papers), and can be regarded as sister works on the same Tencent Hunyuan research line.

### [Kling 3.0 / Kling Video 3.0 Omni](../models/Kling_30_Omni.md)

Kuaishou Technology (快手科技) — the Kling large-model team / Kuaishou Visual Generation and Interaction Center (KwaiVGI, the GitHub organization is also KlingTeam / KlingAIResearch)

### [LTX-2](../models/LTX-2.md)

Lightricks (Israel, the team behind LTX Studio / the LTXV model series)

### [LongCat-Video](../models/LongCat-Video.md)

Meituan's LongCat team. The technical report is credited to the Meituan LongCat Team, with authors including Xunliang Cai, Qilong Huang, Zhuoliang Kang, Hongyu Li, Shijun Liang, Liya Ma, Siyu Ren, Xiaoming Wei, Rixu Xie, Tong Zhang, and others.

### [MOVA](../models/MOVA.md)

The SII-OpenMOSS team. Institutions credited on the paper include: the Shanghai Innovation Institute, MOSI Intelligence (无问芯穹/MOSI 智能), Fudan University, Shanghai Jiao Tong University, East China Normal University, Tongji University, Southeast University, and Xiamen University. The project leads are Qinyuan Cheng (程琴媛) and Tianyi Liang; the corresponding authors are Xie Chen (陈谐, Shanghai Jiao Tong University, chenxie95@sjtu.edu.cn) and Xipeng Qiu (邱锡鹏, Fudan University, xpqiu@fudan.edu.cn). This belongs to the audio-video generation branch of Fudan's OpenMOSS (MOSS series) open-source ecosystem.

### [Merged entry: ① Mochi 1](../models/Mochi_MAGI_Motif.md)

① Mochi 1: Genmo (Genmo AI, a San Francisco, US startup; CEO/co-founder Paras Jain, co-founder Ajay Jain, both with UC Berkeley PhD backgrounds; raised roughly $28.4 million in Series A funding in 2024, led by NEA).
② MAGI-1: Sand AI (三体智能 / sand.ai, Beijing, China; founder Yue Cao (曹越), a Tsinghua University PhD and former Microsoft Research Asia researcher, one of the authors of Swin Transformer; the technical report is credited to 39 authors).
③ Motif-Video 2B: Motif Technologies (a Korean AI company, sharing common origins with the AI chip/compiler company Moreh; previously released the Motif-2.6B / Motif-2-12.7B series of language models).

### [Movie Gen](../models/Movie_Gen.md)

Meta (the Meta AI / GenAI team, a contributor list of over one hundred people)

### [NVIDIA NeMo Curator](../models/NeMo_Curator.md)

NVIDIA. The three repositories belong to different GitHub organizations: NeMo Curator is under NVIDIA-NeMo/Curator, while Cosmos-Xenna and Cosmos-Curate are under the nvidia-cosmos organization (formerly NVIDIA/cosmos-curator). NeMo Curator is maintained by the NeMo Framework team, while Cosmos-Xenna was developed by the Cosmos (Physical AI / world foundation model) team before being independently open-sourced.

### [OmniHuman Dataset + OHBench](../models/OmniHuman.md)

A three-way collaboration between Peking University (Beijing, China), Tencent's WeChat Lab (China), and the Chinese Academy of Sciences (Beijing, China). Author list: Lei Zhu, Xing Cai, Yingjie Chen, Yiheng Li, Binxin Yang, Hao Liu, Jie Chen, Chen Li, Jing Liu (Jing LYu). Note: this work shares its name with, but is unrelated to, ByteDance's 2025 talking-head generation model OmniHuman-1 (arXiv:2502.01061); the latter is a model, while this work is a dataset + evaluation benchmark — the two must be distinguished when searching or citing.

### [Open-Sora Series](../models/Open-Sora.md)

(1) Open-Sora: HPC-AI Tech (潞晨科技, the Colossal-AI team), GitHub organization hpcaitech. (2) Open-Sora Plan: Peking University's Yuan Li research group (PKU-YuanGroup), jointly with Peng Cheng Laboratory, Rabbitpre AI (兔展智能), and others; the HuggingFace organization is LanguageBind. The two have no affiliation with each other and only share a similar name.

### [Ovi](../models/Ovi.md)

Released jointly by Character AI (the primary entity, authors Chetwin Low and Weimin Wang, with Weimin Wang as Project Lead) and Yale University (author Calder Katyal). The acknowledgments mention Yi Cui, Manav Shah, and Diego De La Torre as having participated in data preparation.

### [Script-a-Video](../models/Script-a-Video.md) ⚠️

The Tencent Hunyuan Team. The paper is credited to the team as a whole (the author field lists only "Tencent Hunyuan Team"); the body text does have a Project Contributors section but it is not expanded with specific names in the arXiv HTML version, so the first and corresponding authors cannot be confirmed. [Uncertain]

### [Seedance 2.0 and Seedance 1.5 pro](../models/Seedance_20_Seedance_15_pro.md)

The ByteDance Seed team (ByteDance Seed / Team Seedance)

### [SkyReels Series](../models/SkyReels.md)

The SkyReels team under Kunlun Tech (昆仑万维 / Skywork AI, 天工AI). The V4 paper has 50+ authors; the project lead is Guibin Chen (guibin.chen@kunlun-inc.com), and the project initiator is Yahui Zhou (周亚辉); the team is organized into infrastructure, data processing, video model training, audio model training, multimodal training, and evaluation groups.

### [Sora 2](../models/Sora_2.md)

OpenAI

### [SpeakerVid-5M](../models/SpeakerVid-5M.md)

Tsinghua University (Shenzhen International Graduate School) jointly with StepFun (阶跃星辰) and the Hong Kong University of Science and Technology (HKUST, including its Guangzhou campus). Author list: Youliang Zhang, Zhaoyang Li, Duomin Wang, Jiahe Zhang, Deyu Zhou, Zixin Yin, Xili Dai, Gang Yu, Xiu Li. The project lead is Duomin Wang (王多民, StepFun), and the corresponding author is Xiu Li (李秀, Tsinghua University). Gang Yu (于刚) is the head of StepFun's vision team.

### [Step-Video-T2V](../models/Step-Video-T2V.md)

StepFun (阶跃星辰, Step-Video Team / Shanghai StepFun Intelligent Technology Co., Ltd.)

### [UniTalking](../models/UniTalking.md)

Led by Huawei's Central Media Technology Institute, jointly with the School of Computer Science and Engineering, Beihang University. Author list: Hebeizi Li (李赫贝子, first author, Huawei + Beihang, completed during an internship), Zihao Liang (梁子豪, co-first author), Benyuan Sun (孙本源, Project Leader), Zihao Yin, Xiao Sha, Chenliang Wang, Yi Yang (杨毅, corresponding author). Positioned as an "open-source, reproducible unified talking-portrait audio-video generation framework", explicitly benchmarked against the closed-source inaccessibility of Veo3 and Sora2.

### [UniVerse-1](../models/UniVerse-1.md)

StepFun (阶跃星辰) as the primary/leading institution, jointly with the Hong Kong University of Science and Technology (Guangzhou), the Hong Kong University of Science and Technology, and Tsinghua University. Author list: Duomin Wang (王多民, first author, maintainer of the project homepage and code repository Dorniwang), Wei Zuo, Aojie Li, Ling-Hao Chen (陈凌灏, Tsinghua), Xinyao Liao, Deyu Zhou, Zixin Yin, Xili Dai (戴习理, HKUST-GZ), Daxin Jiang (姜大昕, founder/CEO of StepFun), Gang Yu (于刚, head of StepFun's vision division). Positioned as StepFun's first open-source exploration in the joint audio-video generation direction.

### [Unison](../models/Unison.md)

Jointly credited to five institutions, led by academia with participation from industry and state-owned telecom AI:
1. The State Key Laboratory of Information Engineering in Surveying, Mapping and Remote Sensing (LIESMARS), Wuhan University — the first-credited institution, home to corresponding author Zhigang Tu (涂志刚);
2. ByteDance, China;
3. Xi'an Jiaotong University, the National Key Laboratory of Human-Machine Hybrid Augmented Intelligence and the Institute of Artificial Intelligence and Robotics (IAIR);
4. China Telecom Artificial Intelligence Technology (Beijing) Co., Ltd.;
5. China Telecom Artificial Intelligence Research Institute (TeleAI).
[Author list] Shihao Cheng (程世豪) and Jiaxu Zhang (张家旭) are co-first authors (marked with † Equal contribution); Quanyue Song, Shansong Liu (刘杉松), Zhizhi Guo, Xiao-Lei Zhang (张晓雷, in the speech direction at NPU/XJTU), Chi Zhang (张驰), Xuelong Li (李学龙, CTO/Chief Scientist of TeleAI), Zhigang Tu (涂志刚, corresponding author 🖂). The paper also marks a "‡ project lead" role.
[Funding sources] The National Natural Science Foundation of China's Regional Innovation and Development Joint Fund U25A20537; the National Key Research and Development Program 2024YFC3015600. This is a clearly domestic, academia-led project, not an output of an industry large-model team, and its data scale and disclosure style are markedly different from industrial reports such as those of Sora 2 / Veo 3 / Seedance.

### [Veo 3 / Veo 3.1](../models/Veo_3_Veo_31.md)

Google DeepMind (Google)

### [Vidu S1](../models/Vidu_S1.md)

Shengshu Technology (生数科技) jointly with Tsinghua University. Authors include Jintao Zhang, Kai Jiang, Jintao Chen, and 27 others in total; advisors are Zhijie Deng, Fan Bao (鲍凡), Jianfei Chen, Jun Zhu (朱军)

### [Wan 2.5 / 2.6 / 2.7](../models/Wan.md)

Alibaba Group · Alibaba Cloud's Tongyi Lab (通义实验室, the Wan / Wanxiang team; human-centric models S2V, Animate, and Dancer are led by the HumanAIGC group). Service is delivered via Alibaba Cloud Bailian (Model Studio / DashScope) and the Wanxiang official site wan.video, tongyi.aliyun.com/wan.

### [Audio-Video Generation Evaluation Benchmark Collection](../models/av_benchmarks.md)

Multiple institutions; the five benchmarks belong to different teams:
[VABench] Peking University (Wentao Zhang's group, including Bohan Zeng, Hao Liang, Junbo Niu, and others) + Ant Group (Quanqing Xu) + the Institute of Automation, Chinese Academy of Sciences + Huazhong University of Science and Technology. First authors Daili Hua, Xizhi Wang.
[AVBench] Tsinghua University (Wenming Yang's group, first author Jialiang Yang, including Bin Xia, Ruihang Chu, and others) + the Chinese University of Hong Kong (Dingdong Wang, and others).
[AV-SyncBench] Alibaba Group (Jun Song, Bo Zheng, and others; correspondence jsong.sj@alibaba-inc.com) + Tsinghua University (first author Tianhong Zhou, zth24@mails.tsinghua.edu.cn) + Fudan University.
[PhyAVBench] HKUST(GZ) — the Hong Kong University of Science and Technology (Guangzhou) (corresponding author Li Liu, first author Tianxin Xie) + Tencent + Shanghai Jiao Tong University + TUM (Technical University of Munich); 29+ authors, 4 core contributors.
[Omni-Judge] University of Rochester (first author Susan Liang, corresponding author Chenliang Xu) + University of Michigan, Ann Arbor (Filippos Bellos, Jason J. Corso).

### [Video Captioning Model Ecosystem](../models/caption_models.md)

Many institutions, grouped by camp:
[Academic/open-source community] ShareGPT4Video & ShareCaptioner-Video (USTC + Shanghai AI Lab + the Chinese University of Hong Kong, Lin Chen, Xilin Wei, Jinsong Li, and 15 authors in total, NeurIPS 2024 Datasets & Benchmarks Track); AuroraCap (UC Santa Barbara / a multi-university collaboration, ICLR 2025); Panda-70M (Snap Research + UC Merced, CVPR 2024).
[Major Chinese tech companies] ByteDance's Tarsier / Tarsier2 / Tarsier2-Recap (bytedance/tarsier), together with Tsinghua's Department of Electronic Engineering on video-SALMONN 2 / video-SALMONN-o1; Zhipu AI (zai-org, formerly THUDM)'s CogVLM2-Caption; Alibaba Tongyi's Qwen series (Qwen2-VL / Qwen2.5-VL / Qwen3-VL / Qwen2.5-Omni / Qwen3-Omni-Captioner / Qwen3.5-Omni, accompanied by Omni-Captioner + Omni-Detective + Omni-Cloze, ICLR 2026); Kunlun Tech's SkyWork SkyCaptioner-V1; Kuaishou's Kling team AVoCaDO (jointly with the Institute of Automation CAS, UCAS, PKU, and Nanjing University) and the LLaVA-finetuned captioner for Koala-36M; Nanjing University's NJU-LINK + Kuaishou's Kling AVSCap; Xiaomi's MiMo-VL (used by MOVA as a vision annotator).
[Overseas major tech companies/closed source] OpenAI (Sora's highly descriptive captioner, Whisper ASR); Meta (Movie Gen's LLaMa3-Video 8B/70B); Google (the Gemini series as a general-purpose annotator; Veo 3 uses "multiple Gemini models"); Lightricks (the in-house self-developed audio-video captioner for LTX-Video / LTX-2); Rhymes AI (Aria, used by Allegro as a fine-grained annotator).

### [Geometric/Structured Annotation Dataset Collection](../models/geometric_datasets.md)

Multiple institutions: SceneScribe-1M — Shanghai Jiao Tong University + Ant Group + Oxford University's Visual Geometry Group (VGG) + the Eastern Institute of Technology, Ningbo's Digital Twin Institute (Zhejiang Provincial Key Laboratory of Industrial Intelligence and Digital Twin); SpatialVID — Nanjing University (NJU-3DV) + the Institute of Automation, Chinese Academy of Sciences; WildWorld — Shanda AI (盛大AI) + Shanghai AI Laboratory + Beijing Institute of Technology (authors including Zhen Li, Kaipeng Zhang, and others); Action100M — Meta AI (FAIR, code hosted under facebookresearch) + the Hong Kong University of Science and Technology (Pascale Fung's team) + the University of Amsterdam + Sorbonne University

### [Post-Training Data Strategies for Video Generation](../models/post_training_data.md)

Multiple institutions. The anchor paper is a joint work of the University of Hong Kong (HKU, Zeyue Xue, Mengzhao Chen, Ping Luo) + JD Explore Academy (京东探索研究院, Siming Fu, Jie Huang, Shuai Lu, Haoran Li, Haoyang Huang, Nan Duan, and others) + Tsinghua University + Peking University + Zhejiang University (Zeyue Xue is also the first author of DanceGRPO, and Nan Duan is a senior corresponding-level author). Objects covered horizontally include: ByteDance Seed (Seedance 1.0 / 1.5 pro), Tencent Hunyuan (HunyuanVideo / HunyuanVideo 1.5), Kuaishou Kling (Kling 3.0 Omni), Meituan (LongCat-Video), StepFun (Step-Video-T2V), Kunlun Tech (SkyReels-V2 / V4), NVIDIA (Cosmos-Predict 2.5), Meta (Movie Gen), Zhipu (CogVideoX), Rhymes AI (Allegro), ByteDance (Goku), Moonshot/Motif (Motif-Video 2B), Sand AI (MAGI-1), Genmo (Mochi 1), Lightricks (LTX-2), OpenAI (Sora 2), Google DeepMind (Veo 3/3.1), Shengshu (Vidu S1), HPC-AI Tech (Open-Sora 2.0), PKU-YuanGroup (Open-Sora Plan), as well as the academic-side works JavisDiT++, NAVA, ALIVE, and others.

### [Merged Survey of Mainstream Video Pretraining Datasets: Panda-70M, InternVid, Koala…](../models/pretraining_datasets.md)

Panda-70M: Snap Inc. (Snap Research) + UC Merced. InternVid: Shanghai AI Laboratory / OpenGVLab (jointly with Nanjing University, the University of Hong Kong, Nanyang Technological University, and the Shenzhen Institute of Advanced Technology). Koala-36M: Kuaishou Technology (the repository is now under KlingAIResearch, formerly KwaiVGI) + Shenzhen University + Tsinghua University. MiraData: Tencent PCG ARC Lab + the Chinese University of Hong Kong. OpenVid-1M: Nanjing University's PCA Lab + ByteDance + Nankai University. UltraVideo: Zhejiang University's APRIL Lab + Shanghai Jiao Tong University + Huazhong University of Science and Technology + Nanyang Technological University. LVD-2M: the University of Hong Kong + ByteDance.

## Release date (technical report/paper/open-source date)

`release_date` · Level of detail: minimal

### [Allegro](../models/Allegro.md)

arXiv preprint v1 submitted October 20, 2024 (arXiv:2410.15458, "Allegro: Open the Black Box of Commercial-Level Video Generation Model"); Allegro T2V weights and inference code open-sourced on GitHub / Hugging Face on October 22, 2024 (Apache 2.0); Allegro-TI2V (text+image-to-video) released November 25, 2024; Allegro T2V training code open-sourced December 10, 2024; the 40×720P / 40×360P variants released December 26, 2024; Allegro-TI2V training code open-sourced January 2, 2025

### [Apollo](../models/Apollo.md)

First arXiv submission (v1, titled Apollo) on January 7, 2026; v2 update (title changed to Klear) on January 13, 2026. The HuggingFace paper page lists an intake date of January 8, 2026. There is no independent open-source release date (not open-sourced).

### [CineDance / CineDance-1M](../models/CineDance.md)

First arXiv submission June 8, 2026 (arXiv:2606.09639 v1), updated to v2 on June 11, 2026. The GitHub repository and project homepage launched at the same time. The first shard of the dataset (CineDance_01) was released on HuggingFace in gated form around July 2026. As of July 2026, the code (curation pipeline, inference, training) and model weights are still marked "pending release" and have not yet been published.

### [CogVideoX](../models/CogVideoX.md)

CogVideoX-2B weights and code open-sourced August 6, 2024; arXiv preprint 2408.06072 v1 ("CogVideoX: Text-to-Video Diffusion Models with An Expert Transformer") August 12, 2024; CogVideoX-5B open-sourced August 27, 2024; CogVideoX-5B-I2V open-sourced September 2024; "New Qingying" launched and CogVideoX1.5-5B / 1.5-5B-I2V open-sourced November 8, 2024 (the CogSound sound-effects model was released in the same period); the paper was accepted at ICLR 2025, with v3 updated March 26, 2025

### [Cosmos-Predict2.5](../models/Cosmos-Predict25.md)

arXiv v1 submitted October 28, 2025 (arXiv:2511.00062v1), v2 revised February 24, 2026 (the paper header is dated 2026-2-26, copyright notice ©2026 NVIDIA). Code and weights were opened at the same time on GitHub at nvidia-cosmos/cosmos-predict2.5 and on Hugging Face at nvidia/Cosmos-Predict2.5-2B / -14B.

### [Data-Juicer 2.0](../models/Data-Juicer.md) ⚠️

Series timeline:
- 2023: Data-Juicer 1.0 first open-sourced, with a paper published at SIGMOD 2024 (roughly 50 text-only pretraining operators).
- July 16, 2024: the Data-Juicer Sandbox paper submitted to arXiv (arXiv:2407.11784 v1), latest v3 on June 4, 2025, accepted at ICML 2025 Spotlight. During the same period it topped the VBench text-to-video leaderboard and was promoted by the Alibaba Cloud developer community/ModelScope community.
- January 2025 (arXiv number 2501.14755; the submission history shows v1 on December 23, 2024, v2 on June 4, 2025, v3 on October 29, 2025): the Data-Juicer 2.0 paper released, accepted at NeurIPS 2025 Spotlight.
- Continuous iteration up to the time of this survey: v1.4.4 (2025-12-01, coinciding with the NeurIPS Spotlight, adding 6 video/multimodal operators and S3 I/O), v1.4.5 (2026-01-15, 20+ new operators, a Ray vLLM pipeline, embodied-AI operators), v1.4.6 (2026-02-02, a Q&A Copilot, video byte-stream I/O), v1.5.0 (2026-02-12, a partitioned Ray executor, embodied-AI video processing, operator-level environment management), v1.5.1 (2026-03-17), v1.5.2 (2026-05-29), v1.5.3 (2026-06-26, VLA embodied operators, a Ray repartition pipeline), v1.5.4 (2026-07-17, 9 new human-centric video-understanding operators, batch-local stage-fusion acceleration).
[Uncertain] The Data-Juicer 2.0 paper's first arXiv submission date appears in different sources as either December 23, 2024 or January 2025; inferring from the arXiv number 2501, the formal posting date should be January 2025.

### [Foley-Omni](../models/Foley-Omni.md)

Submitted to arXiv June 2, 2026 (arXiv:2606.03672, cs.SD/cs.CV direction, v1). The project homepage https://ty0402.github.io/Foley-omni-Web/ and the GitHub repository NJU-Speech/Foley-Omni launched during the same period; CocoBro/Foley-Omni inference weights were released on HuggingFace. ResearchGate indexed it in the same period (publication 405852241). As of the time of this survey (July 2026), the V2ST-Bench evaluation set is marked "Coming soon" in the repository and has not yet been formally released.

### [Goku](../models/Goku.md)

First submitted to arXiv February 7, 2025 (v1), updated to v2 on February 10, 2025; subsequently accepted at CVPR 2025 as a Highlight. The project homepage saiyan-world.github.io/goku and the GitHub repository Saiyan-World/goku launched during the same period.

### [Hailuo / MiniMax Video](../models/Hailuo.md)

No technical report or paper exists, only product release blog posts, with the following timeline:
- September 2024: video-01 (the first-generation Hailuo video, 6 seconds, 1280x720, 25fps)
- January 2025: S2V-01 (Subject-Reference / character consistency)
- June 18, 2025: MiniMax Hailuo 02 (introducing the NCR architecture, native 1080p)
- October 28, 2025: MiniMax Hailuo 2.3 / 2.3 Fast, simultaneously upgrading the Video Agent to a Media Agent
- December 15, 2025: open-sourced the VTP visual tokenizer (arXiv:2512.13687), a generative foundational component from the same team, not the video generation model itself
As of the time of this survey (July 29, 2026), the latest video model on MiniMax's official news page and platform documentation is still Hailuo 2.3, with no announcement of a new-generation video model released in 2026.

### [HunyuanVideo-Foley](../models/HunyuanVideo-Foley.md)

Submitted to arXiv August 23, 2025 (arXiv:2508.16930v1, UTC 07:30:18); formally open-sourced on GitHub / HuggingFace on August 28, 2025, releasing inference code and the XXL model weights; the XL version weights and offload low-VRAM inference support were released as an addendum on September 29, 2025. It has since entered a maintenance/iteration phase, with no v2 paper revision observed.

### [HunyuanVideo](../models/HunyuanVideo.md)

HunyuanVideo: technical report released December 3, 2024 (arXiv:2412.03603, with a later v2 revision), with weights and inference code open-sourced simultaneously; the HunyuanVideo-I2V image-to-video version released March 2025. HunyuanVideo 1.5: open-sourced November 21, 2025, technical report arXiv:2511.18870 (November 2025).

### [InstructAV2AV](../models/InstructAV2AV.md) ⚠️

Submitted to arXiv May 18, 2026 (arXiv:2605.18467, v1). The project homepage https://hjzheng.net/projects/InstructAV2AV/, the GitHub repository suimuc/InstructAV2AV, the HuggingFace model suimu/InstructAV2AV, and the dataset suimu/InsAVE-80K all launched during the same period. As of the time of this survey (July 2026), the inference code, weights, and dataset have all been released, while the training scripts are still marked "in progress" on the roadmap. [Uncertain] No formal conference/journal acceptance information has been found; it is currently at the arXiv preprint stage.

### [2026 Miscellaneous Joint Audio-Video Generation Works](../models/JAVG_2026_misc.md)

All are first-half-2026 arXiv debuts:
- ALIVE: v1 on February 9, 2026, v2 on February 10 (the earliest of the seven).
- OmniCustom: v1 on February 12, 2026, continuing revisions through v5 on July 23, 2026 (the longest-iterated).
- CCL: March 19, 2026 (arXiv number 2603.18600).
- Baton: v1 on May 24, 2026, v2 on June 1, 2026.
- StreamChar: May 25, 2026.
- NAVA: May 28, 2026, subsequently open-sourcing code and weights.
- ITS-JAVG: June 2, 2026 (number 2606.03183).
Chronologically this shows a clear quarterly rhythm: February saw major-lab foundation models (ALIVE) and customized tasks (OmniCustom) lead the way; March saw the emergence of a "low-cost, high-efficiency route" (CCL); May–June saw a concentrated appearance of four new branches — semantic planning (Baton), long-horizon streaming (StreamChar), native alignment (NAVA), and inference-time scaling (ITS-JAVG) — reflecting how the JAVG field shifted in the first half of 2026 from "can joint generation be done at all" to "how to make it cheaper, longer, more accurate, and more controllable."

### [Joint Audio-Video Generation Baseline Collection](../models/JavisDiT_baselines.md)

(2) MM-Diffusion: submitted to arXiv on December 19, 2022 (v1), revised in March 2023, accepted at CVPR 2023; the earliest work in this collection.
(3) AV-DiT: published on arXiv on June 11, 2024 (arXiv:2406.07686).
(1) JavisDiT: first published on arXiv March 30, 2025 (v1), revised February 22, 2026, accepted at ICLR 2026; its sequel JavisDiT++ was published in February 2026 (arXiv:2602.19163), also at ICLR 2026.
(5) UniAVGen: submitted to arXiv November 5, 2025 (arXiv:2511.03334), revised March 24, 2026.
(4) Harmony: submitted to arXiv November 26, 2025 (arXiv:2511.21579), revised November 28, 2025.

### [Kling 3.0 / Kling Video 3.0 Omni](../models/Kling_30_Omni.md)

The Kling 3.0 series (Kling Video 3.0, Kling Video 3.0 Omni, Kling Image 3.0, Kling Image 3.0 Omni) launched globally on February 4–5, 2026. Its direct technical predecessor and publicly citable official technical report is the "Kling-Omni Technical Report" (arXiv:2512.16776, submitted December 18, 2025, 67 authors, credited to the Kling Team); native simultaneous audio-picture generation was first introduced in Kling 2.6 in December 2025. Related same-lineage reports also include the "KlingAvatar 2.0 Technical Report" (arXiv:2512.13313, December 2025) and the "Kling-MotionControl Technical Report" (arXiv:2603.03160, March 2026).

### [LTX-2](../models/LTX-2.md)

LTX-2 first publicly announced in October 2025 (preview/product integration into LTX Studio); all model weights and training code formally open-sourced on January 6, 2026, with the technical report arXiv:2601.03233v1 (cs.CV, 14 pages plus 2 pages of supplementary material) released on arXiv the same day; the upgraded LTX-2.3 (22B, LTX Desktop) released March 5, 2026. Prior foundations: LTX-Video 2B (November 2024), LTXV-13B (May 2025).

### [LongCat-Video](../models/LongCat-Video.md)

LongCat-Video foundation model: arXiv v1 submitted October 25, 2025 (2510.22200), v2 on October 28; formal public release and open-sourcing occurred around October 27, 2025. Derivative versions: LongCat-Video-Avatar released December 23, 2025 (Meituan's technical team blog published the same day); LongCat-Video-Avatar 1.5 released May 2026 (arXiv:2605.26486).

### [MOVA](../models/MOVA.md)

First open-source release January 29, 2026 (model weights + code, launched simultaneously on GitHub/HuggingFace); the 38-page technical report arXiv:2602.08794 published in February 2026 (v2 dated February 10, 2026, cs.CV); API launched March 9, 2026; supplementary open-source evaluation code released May 6, 2026.

### [Merged entry: ① Mochi 1](../models/Mochi_MAGI_Motif.md)

① Mochi 1: mochi-1-preview released and open-sourced (weights + inference code, Apache 2.0) on October 22, 2024, with a preview of a 720p HD version announced the same day (subsequently never fully released on schedule); AsymmVAE and fine-tuning scripts were open-sourced progressively around November 2024. There is no formal paper or arXiv technical report, only an official blog post.
② MAGI-1: the 24B weights and inference code open-sourced April 21, 2025 (GitHub SandAI-org/MAGI-1); the arXiv technical report arXiv:2505.13211, "MAGI-1: Autoregressive Video Generation at Scale" (61 pages), released May 19, 2025; the 4.5B version released April 30, 2025; the 4.5B Distill released May 26, 2025; ComfyUI support added May 30, 2025; MAGI-1.1 24B open-sourced June 17, 2026.
③ Motif-Video 2B: weights released on Hugging Face April 14, 2026 (Motif-Technologies/Motif-Video-2B, Apache 2.0); arXiv:2604.16503, "Motif-Video 2B: Technical Report", with a v2 update on May 19, 2026; official Diffusers integration and community GGUF quantized versions offered during the same period.

### [Movie Gen](../models/Movie_Gen.md)

Released along with an official blog post and technical report on October 4, 2024; arXiv preprint 2410.13720 (v1, posted October 17, 2024), titled "Movie Gen: A Cast of Media Foundation Models"

### [NVIDIA NeMo Curator](../models/NeMo_Curator.md)

[NeMo Curator version timeline (dual CalVer/SemVer tracking, based on PyPI upload times)] 0.6.0 (2025-01-07, Dask architecture, text-only era) → 0.7.0 (2025-03-12) → 0.8.0 (2025-05-09) → 0.9.0 (2025-07-28) → 1.0.0 = version 25.09 (2025-10-01, milestone: backend fully switched from Dask to Ray, introducing video and audio modalities for the first time, forming a unified text/image/video/audio four-modality architecture) → 1.1.0 = version 26.02 (2026-02-23) → 1.2.0 = version 26.04 (2026-05-14) → 1.3.0 = version 26.07 (2026-07-27, released two days before the cutoff of this survey).
[Cosmos-Xenna] Split out and independently open-sourced from Cosmos-Curator (Apache 2.0) as the Cosmos platform evolved in 2025; NeMo Curator version 26.04 upgraded to Cosmos-Xenna 0.2.0. As of July 2026, the cosmos-xenna repository is marked "no longer actively developed," with official guidance to migrate to the Cosmos 3 / NeMo Curator ecosystem.
[Primary methodological literature] "Cosmos World Foundation Model Platform for Physical AI" (arXiv:2501.03575, 2025-01-07, Section 3 fully discloses the seven-tier video data curation pipeline); "Training Video Foundation Models with NVIDIA NeMo" (arXiv:2503.12964, 2025-03, specifically describing the dual clipping/sharding pipeline and GPU acceleration); NVIDIA's official blog first announced the "89x speedup" in 2025-01.

### [OmniHuman Dataset + OHBench](../models/OmniHuman.md) ⚠️

First submitted to arXiv April 20, 2026 (arXiv:2604.18326v1), with a revised v2 submitted May 30, 2026. Primary category cs.CV. The paper is 19 pages with 6 figures. As of July 2026, no clear conference-acceptance marking has been observed (the comments field contains only page and figure counts). Code and the evaluation toolchain have been released on GitHub (github.com/julia-cherry/OmniHuman), and dataset annotations have been released on HuggingFace (julia527/omnihuman). [Uncertain]

### [Open-Sora Series](../models/Open-Sora.md)

[Open-Sora (HPC-AI Tech)] v1.0: March 2024; v1.1: April 25, 2024 (first release of the complete video data processing pipeline); v1.2: June 17, 2024 (corresponding to paper arXiv:2412.20404, "Open-Sora: Democratizing Efficient Video Production for All", posted to arXiv December 29, 2024); v1.3 (1B): February 20, 2025; v2.0 (11B): released March 12, 2025, technical report arXiv:2503.09642, "Open-Sora 2.0: Training a Commercial-Level Video Generation Model in $200k".
[Open-Sora Plan (Peking University)] v1.0.0: April 2024; v1.1.0: May 2024; v1.2.0: July 2024; v1.3.0: October 2024; technical report arXiv:2412.00131, "Open-Sora Plan: Open-Source Large Video Generation Model", submitted November 28, 2024 (content corresponding to v1.3); v1.5.0: released June 5, 2025 (8B, SUV sparse MMDiT + 8×8×8 WFVAE).

### [Ovi](../models/Ovi.md)

Paper finalized September 29, 2025, published on arXiv October 1, 2025 (arXiv:2510.01284, v1); the HuggingFace model page lists a paper release date of September 30, 2025; the 11B model weights and inference code were open-sourced on GitHub (character-ai/Ovi) during the same period. The Ovi 1.1 update (native 960×960 training, 10-second generation, doubled dataset scale) released November 10, 2025. As of July 2026, no Ovi 2.0 or training-code release has been observed.

### [Script-a-Video](../models/Script-a-Video.md) ⚠️

Published on arXiv in April 2026 (arXiv:2604.11244). The latest visible version is v2, dated April 15, 2026 (cs.CV). v1 was submitted slightly earlier in the same month (arXiv:2604.11244v1). As of July 2026, no accompanying GitHub repository, project homepage, or conference-acceptance record has been found. [Uncertain]

### [Seedance 2.0 and Seedance 1.5 pro](../models/Seedance_20_Seedance_15_pro.md)

Seedance 1.5 pro: technical report arXiv:2512.13507, v1 submitted 2025-12-15, v3 updated 2025-12-23; the product launched in December 2025 on Doubao/Jimeng/Volcano Ark (model ID: Doubao-Seedance-1.5-pro). Seedance 2.0: formally released in China in early February 2026 (model ID: doubao-seedance-2-0-260128), technical report arXiv:2604.14148 submitted 2026-04-15, with the Volcano Engine API fully opened 2026-04-14. As a vertical comparison baseline, the Seedance 1.0 technical report is arXiv:2506.09113 (2025-06).

### [SkyReels Series](../models/SkyReels.md)

SkyReels-V1: open-sourced February 18, 2025 (a human-centric video foundation model). SkyReels-V2: paper submitted to arXiv on April 17, 2025 (2504.13074, v3 revised April 21), inference code and SkyCaptioner-V1 open-sourced April 21, 2025, the 720P version added April 24. SkyReels-V3: open-sourced January 29, 2026. SkyReels-V4: paper submitted February 24/25, 2026 (arXiv:2602.21818, v3 on March 18, 2026), formally launched as a product with an API at the Zhongguancun Forum on March 27, 2026; as of March 18, 2026, it ranked first on both the "text-to-video (with audio)" and "image-to-video (with audio)" leaderboards, and second on "text-to-video (without audio)," on the Artificial Analysis Video Arena.

### [Sora 2](../models/Sora_2.md)

Model and System Card released September 30, 2025 (sora.com and a standalone iOS Sora app launched the same day); API access (sora-2 / sora-2-pro) opened in early October 2025, extending the duration limit from 10 to 15 seconds (25 seconds on the Pro web version); a three-year licensing agreement with Disney reached in December 2025; in March 2026 OpenAI announced it would shut down the Sora consumer app (the app went offline April 26, 2026, and the API was discontinued September 24, 2026), terminating Disney's $1 billion investment and licensing agreement accordingly. Note: OpenAI never published a technical report or paper for Sora 2, only a 7-page System Card.

### [SpeakerVid-5M](../models/SpeakerVid-5M.md)

First submitted to arXiv July 14, 2025 (arXiv:2507.09862, cs.CV). Accompanying resources were released progressively thereafter: the HuggingFace dataset repository dorni/SpeakerVid-5M-Dataset created July 18, 2025, last updated August 4, 2025; the GitHub data-cleaning code repository Dorniwang/SpeakerVid-5M-Code and the project homepage https://dorniwang.github.io/SpeakerVid-5M/ launched during the same period.

### [Step-Video-T2V](../models/Step-Video-T2V.md)

Inference code and weights open-sourced simultaneously on February 17, 2025, with the technical report "Step-Video-T2V Technical Report: The Practice, Challenges, and Future of Video Foundation Model" (arXiv:2502.10248, with later v2/v3 revisions) released the same day; formally announced to the public February 18, 2025 (open-sourced in the same batch as the speech model Step-Audio). The derivative model Step-Video-TI2V (image-to-video) open-sourced March 17, 2025, technical report arXiv:2503.11251.

### [UniTalking](../models/UniTalking.md)

Submitted to arXiv March 2, 2026 (arXiv:2603.01418v1, primary category cs.CV, cross-listed under cs.MM, cs.SD). The paper is marked as accepted at CVPR 2026 (Findings Track). As of July 2026, no public release record of a code repository, model weights, or project homepage has been found.

### [UniVerse-1](../models/UniVerse-1.md)

Project homepage launched September 3, 2025; submitted to arXiv September 7, 2025 (arXiv:2509.06155v1, cs.CV, UTC 17:55:03); model weights, inference code, and the Verse-Bench dataset open-sourced September 8, 2025; technical report formally released September 9, 2025; the Verse-Bench evaluation metric toolkit released as an addendum September 28, 2025. There is also an OpenReview submission record (forum id 8aFYx2mDyE).

### [Unison](../models/Unison.md) ⚠️

arXiv:2605.08729. v1 submitted May 9, 2026 (cs.CV); v2 updated June 29, 2026 (this entry is based primarily on v2). v1 and v2 are identical in their description of training corpora and data processing; v2 mainly adds a Zipformer citation (adding yao2024zipformer) and some experimental wording. Licensed under CC BY 4.0.
The paper uses the Springer LNCS conference template (the multi-institution numbered \institute format), and the body text explicitly states "The code and models will be made publicly available upon acceptance," indicating that as of July 2026 it is still under conference review and has not been formally published. No project homepage, demo page, or code repository has been found online. [Uncertain]

### [Veo 3 / Veo 3.1](../models/Veo_3_Veo_31.md)

Veo 3 launched at Google I/O on May 20, 2025; the official Model Card was first published on 2025-05-23 and updated on 2026-01-13; the accompanying technical report "Veo: a text-to-video generation system" was released during the same period. Veo 3.1 launched in mid-October 2025 (Flow / Gemini API / Vertex AI). The Veo 3.1 Lite Model Card was published on 2026-04-08. The official statement notes "this Model Card covers Veo 3 and its subsequent versions," meaning the Veo 3.1 series inherits Veo 3's data and training disclosures.

### [Vidu S1](../models/Vidu_S1.md)

July 2026: the product was formally launched on July 6, 2026 at the Global Digital Economy Conference by Shengshu Technology founder Jun Zhu; arXiv preprint 2607.03118, with v2 dated July 21, 2026 (cs.CV, 13 pages)

### [Wan 2.5 / 2.6 / 2.7](../models/Wan.md) ⚠️

Series timeline (all officially, first-party verifiable):
- Wan 2.1: open-sourced February 25, 2025 (1.3B/14B), technical report "Wan: Open and Advanced Large-Scale Video Generative Models," arXiv:2503.20314, released March 26, 2025, 60 pages, 33 figures, with Chapter 3 as a complete data-processing chapter — the only first-party document in the entire Wan series to disclose the data methodology in detail.
- Wan 2.2: open-sourced July 28, 2025 (T2V-A14B / I2V-A14B MoE, TI2V-5B), Apache 2.0.
- Wan2.2-S2V-14B (audio-driven human video): paper arXiv:2508.18621 submitted August 26, 2025, weights uploaded to Hugging Face September 17, 2025.
- Wan2.2-Animate-14B: open-sourced in early November 2025.
- Wan 2.5-preview: launched as an API preview in late September 2025 (during the Yunqi Conference), model names wan2.5-t2v-preview / wan2.5-i2v-preview; official documentation example assets carry date stamps of 2025-09-23 / 2025-09-25, which serve as corroborating evidence for the timing. This series first achieved "synchronized audio-picture generation." [Uncertain: exact day]
- Wan 2.6: announced on the Wanxiang official site December 16, 2025, with a formal announcement from the Alibaba Cloud developer community on December 17; model names wan2.6-t2v / wan2.6-i2v / wan2.6-i2v-flash / wan2.6-t2v-us.
- Wan 2.7: June 2026, with the official API model name directly carrying a date stamp, wan2.7-t2v-2026-06-12, and wan2.7-i2v, currently the recommended flagship version.
- Wan-Dancer-14B (music-driven long-duration dance video): open-sourced July 2026, paper arXiv:2607.09581.
Note: the 2.5/2.6/2.7 versions have no technical report, no paper, and no weights, so their data methodology can only be inferred backward from the first-party disclosures of 2.1/2.2/S2V and API documentation capabilities.

### [Audio-Video Generation Evaluation Benchmark Collection](../models/av_benchmarks.md) ⚠️

[VABench] arXiv:2512.09299, v1 submitted December 10, 2025, v2 updated April 6, 2026 (24 pages/25 figures, cs.CV + cs.SD, CC BY 4.0). The research task description states it as a CVPR 2026 paper, but the arXiv comments field does not mark acceptance information [uncertain].
[AVBench] arXiv:2605.24652, published May 2026; the project page's figure naming points to an ECCV submission [uncertain].
[AV-SyncBench] arXiv:2607.00726, published July 2026; already accepted at Interspeech 2026.
[PhyAVBench] arXiv:2512.23994, v1 published December 2025 (at the time, a pure benchmark-design report, with model evaluation left for later), with subsequent versions completing full evaluation results for 17 models.
[Omni-Judge] arXiv:2602.01623, published February 2026.

### [Video Captioning Model Ecosystem](../models/caption_models.md)

Ecosystem evolution timeline (based on public release/arXiv submission dates):
· 2023: Tag2Text (a lightweight tag-based annotator, used by Allegro for coarse filtering).
· 2024-02: Panda-70M (arXiv 2402.19479, CVPR 2024), the first systematic "multi-teacher + retrieval-selection" captioning paradigm.
· 2024-06-06: ShareGPT4Video / ShareCaptioner-Video (arXiv 2406.04325), introducing DiffSW differential sliding-window captioning.
· 2024-08: CogVideoX (arXiv 2408.06072) disclosed a four-stage captioning chain, Panda-70M→CogVLM→GPT-4→LLaMA2; CogVLM2-Caption weights open-sourced 2024-09-19.
· 2024-10: AuroraCap + the VDC benchmark (arXiv 2410.03051); Koala-36M (arXiv 2410.08260).
· 2024-10: the Movie Gen technical report disclosed the LLaMa3-Video 8B/70B captioning approach.
· 2025-01-14: Tarsier2-7B (arXiv 2501.07888); the accompanying Tarsier2-Recap-585K dataset.
· 2025-02: video-SALMONN-o1 (arXiv 2502.11775).
· 2025-04: SkyCaptioner-V1 (open-sourced alongside SkyReels-V2).
· 2025-06-18: video-SALMONN 2 (arXiv 2506.15220, 3B/7B/72B).
· 2025-09: Qwen3-Omni-Captioner (audio captioner) open-sourced; 2025-10 Omni-Detective / Omni-Captioner open-sourced.
· 2025-10-12: AVoCaDO (arXiv 2510.10395), the first systematically built joint audio-video captioner.
· 2026-01: LTX-2 (arXiv 2601.03233) disclosed its in-house joint audio-video captioner.
· 2026-03: Qwen3.5-Omni (arXiv 2604.15804) elevated "script-level structured audio-video captioning" to a first-class capability, with the Omni-Cloze benchmark released.
· 2026-06: OmniCap-IF (arXiv 2606.08572), the first instruction-following benchmark for audio-video captioning.
· 2026-07-14: AVSCap + AVSCapBench (arXiv 2607.12820), the newest joint audio-video captioning model and dedicated benchmark.

### [Geometric/Structured Annotation Dataset Collection](../models/geometric_datasets.md)

SceneScribe-1M: arXiv 2604.07990, submitted April 9, 2026, revised April 26, accepted at CVPR 2026; SpatialVID: arXiv 2509.09676, v1 September 11, 2025, v2 December 18, 2025; WildWorld: arXiv 2603.23497, submitted March 24, 2026; Action100M: arXiv 2601.10592, submitted January 15, 2026

### [Post-Training Data Strategies for Video Generation](../models/post_training_data.md)

The anchor paper, arXiv:2604.25427v1, was submitted April 28, 2026 (cs.CV, CC BY 4.0). The horizontally covered objects span from August 2024 (CogVideoX) to the first half of 2026 (Seedance 1.5 pro, Kling 3.0 Omni, SkyReels-V4, Cosmos-Predict 2.5, HunyuanVideo 1.5, LongCat-Video, and others). Timeline of key papers underpinning the methodology: ImageReward (2023) → VideoReward/"Improving Video Generation with Human Feedback" (2025-01, arXiv:2501.13918) → DanceGRPO (2025-05, arXiv:2505.07818) → Flow-GRPO (2025-05) → MixGRPO (2025-07) → HPSv3 (ICCV 2025, arXiv:2508.03789) → Self-Forcing (2025-06) → OmniForcing (2026-03), Causal Forcing (2026-02), Astrolabe (2026-03).

### [Merged Survey of Mainstream Video Pretraining Datasets: Panda-70M, InternVid, Koala…](../models/pretraining_datasets.md)

Panda-70M: first published on arXiv February 29, 2024 (arXiv:2402.19479), CVPR 2024; additional desirability filtering and shot-boundary annotation added October 2024. InternVid: v1 July 13, 2023, v2 January 4, 2024 (arXiv:2307.06942), ICLR 2024 spotlight. Koala-36M: v1 October 10, 2024, v2 April 26, 2025 (arXiv:2410.08260), CVPR 2025. MiraData: July 8, 2024 (arXiv:2407.06358, v1 only), NeurIPS 2024 D&B; v1 data released July 10, 2024, with an earlier v0 beta. OpenVid-1M: released July 1, 2024 (arXiv:2407.02371), accepted at ICLR 2025 (January 2025); the OpenVidHD-0.4M standalone download package added May 30, 2025. UltraVideo: June 16, 2025 (arXiv:2506.13691, v1 only), NeurIPS 2025 D&B poster. LVD-2M: arXiv October 14, 2024, data released October 15, 2024 (arXiv:2410.10816), NeurIPS 2024 D&B.

## Type (model/dataset/toolchain/evaluation benchmark)

`type` · Level of detail: minimal

### [Allegro](../models/Allegro.md)

Model (a text-to-video / image-to-video Diffusion Transformer foundation model) + accompanying toolchain (an in-house VideoVAE 175M, VideoDiT 2.8B, a video captioning model fine-tuned from Aria, and complete training and inference code). The paper's core selling point is precisely "opening the black box of a commercial-level video generation model," so the disclosure granularity of its data-processing pipeline is among the finest in contemporaneous open-source work.

### [Apollo](../models/Apollo.md)

Model (a unified multi-task joint audio-video generation foundation model, 26B parameters). Accompanying output: the paper claims to have built the "first large-scale audio-video dataset with dense captions" (81M samples) and an accompanying automated data-construction pipeline, but neither the dataset nor the pipeline has been released, so it does not constitute a usable dataset output. Evaluation uses the third-party benchmark Verse-Bench, not a self-built one.

### [CineDance / CineDance-1M](../models/CineDance.md)

A composite work centered on a dataset, in three parts:
[Primary output] Dataset — CineDance-1M, the first large-scale, open, 1080p research dataset for multi-shot, long-form joint audio-video generation, comprising 1,021,657 narrative sequences / approximately 26.3K hours.
[Secondary output 1] Evaluation benchmark — CineBench, 1,000 stratified test prompts plus a six-dimensional human-alignment metric system.
[Secondary output 2] Model — CineDance, an open-source baseline adapted from LTX-2.3 (13B video + 3B audio + 3B cross-modal attention), used to validate the dataset's effectiveness.
[Secondary output 3] Toolchain — a three-stage data curation pipeline (cleaning / narrative parsing / dual-modality annotation), which the paper promises to open-source but has not yet released.

### [CogVideoX](../models/CogVideoX.md)

Model (a text-to-video/image-to-video diffusion Transformer foundation model series) + accompanying toolchain (a 3D causal VAE, the CogVLM2-Caption video annotation model, a caption-upsampler prompt rewriter, and the fine-tuning/inference code repository zai-org/CogVideo)

### [Cosmos-Predict2.5](../models/Cosmos-Predict25.md)

Model (a video World Foundation Model) + accompanying toolchain + a self-built evaluation benchmark. The core is a flow-matching-based DiT video generation model that unifies Text2World / Image2World / Video2World modes within a single model, released in 2B and 14B configurations; also released alongside are the Cosmos-Transfer2.5 ControlNet series (Sim2Real / Real2Real world translation) and multiple domain-specialized variants (7-view multi-camera autonomous driving, robot action-conditioned, AgiBot 3-view, GR00T GR1). Evaluation uses the self-built PAI-Bench (Physical AI Bench). On the data-processing side, it is associated with NVIDIA's independently open-sourced Cosmos Curator / Cosmos-Xenna and NeMo Curator video curation frameworks.

### [Data-Juicer 2.0](../models/Data-Juicer.md)

Toolchain (a data-processing system / operator library / data-model co-development platform), not a generative model and not a dataset. It can be broken down into three levels of output:
1. [Operator library and execution engine] Data-Juicer 2.0 itself — as of v1.5.4, 229 operators in total (138 Mappers, 58 Filters, 10 Deduplicators, 8 Formatters, 5 Selectors, 4 Aggregators, 3 Groupers, 3 Pipelines), covering five modalities: text/image/audio/video/multimodal, with multiple adaptive execution backends including Ray, MaxCompute, and single-machine.
2. [Data-model co-development suite] Data-Juicer Sandbox — provides a "Probe-Analyze-Refine" workflow that closes the loop between data-recipe search and model training/evaluation, with built-in integration for training and evaluation frameworks such as EasyAnimate, T2V-Turbo, ModelScope, and VBench.
3. [Derived datasets and recipes] Taking Sandbox's practice on text-to-video as an example, it has open-sourced a curated optimal T2V data pool (HuggingFace datasets such as datajuicer/data-juicer-t2v-optimal-data-pool) and corresponding YAML recipes, an output that is a trinity of "method + tool + data recipe."
Therefore, within the genealogy of this survey, Data-Juicer belongs to the "data infrastructure" category, standing in the most direct benchmarking relationship with NVIDIA NeMo Curator.

### [Foley-Omni](../models/Foley-Omni.md)

A combined output of model + evaluation benchmark + data-processing pipeline methodology. The core is a unified multimodal audio generation model for video-to-complete-soundtrack (V2ST) generation (roughly 5.5B parameters, based on a Diffusion Transformer); it also proposes the V2ST-Bench evaluation benchmark (300 video-audio-text triplets); Section 3.1 of the paper fully describes an audio-video data-cleaning and structured-annotation pipeline. Note: this project generates audio/soundtrack, with video serving as a conditioning input rather than the generation target, so it belongs to the V2A/V2ST branch within the "audio-video generation" genealogy, rather than being a text-to-video model.

### [Goku](../models/Goku.md)

Model (a joint image-and-video generation foundation model family, based on a rectified-flow Transformer/DiT). The paper also discloses in detail a five-stage data-processing pipeline (which can be regarded as a methodological description of a toolchain, but the pipeline code and the data itself have not been open-sourced). Not a dataset, not an evaluation benchmark.

### [Hailuo / MiniMax Video](../models/Hailuo.md)

Model (a closed-source commercial video generation model/product line, covering text-to-video T2V, image-to-video I2V, First-and-Last-Frame, Subject-Reference, and other modes). Not a dataset, not a toolchain, not an evaluation benchmark; MiniMax has also not released an accompanying video evaluation benchmark.

### [HunyuanVideo-Foley](../models/HunyuanVideo-Foley.md)

Model (with a complete accompanying inference toolchain). The core is a text-video-to-audio (TV2A) sound-effects generation model, i.e., given video + text description, it generates Foley sound effects synchronized with the imagery. It is not a video generation model itself, but rather a model that pairs with the "dubbing/sound-effects" stage alongside video generation models. It also comes with an in-house audio VAE (DAC-VAE, 48kHz). No dataset is released, and no evaluation benchmark is released (evaluation reuses others' Kling-Audio-Eval, MovieGen-Audio-Bench, and VGGSound-Test).

### [HunyuanVideo](../models/HunyuanVideo.md)

Model (an open-source video generation foundation model), with complete accompanying open-source code and inference framework. The original 13B version was the largest-parameter open-source video generation model at the time; the 1.5 version is a lightweight 8.3B model, primarily targeting consumer-grade GPUs (runnable with roughly 14GB of VRAM). Both are pure video generation models, not datasets, not evaluation benchmarks.

### [InstructAV2AV](../models/InstructAV2AV.md)

A trinity output of model + dataset + data-synthesis pipeline (data engine), rather than a plain video generation model.
[Model] InstructAV2AV, an instruction-guided joint audio-video editing model adapted from the Ovi twin-tower symmetric DiT architecture, taking "source audio-video + text instruction" as input and outputting "edited audio-video." Note its task is editing rather than generation from scratch.
[Dataset] InsAVE-80K, which the paper claims is the first large-scale audio-video editing dataset, containing source-to-target paired samples.
[Pipeline] The scalable data-synthesis engine described in Section 3 of the paper, this work's main contribution on the data dimension, and the focus of this survey.
[Evaluation] No independently named evaluation benchmark is proposed; instead, 1K manually curated samples are carved out from InsAVE-80K as an evaluation set, with additional zero-shot generalization evaluation on the existing AvED-Bench.

### [2026 Miscellaneous Joint Audio-Video Generation Works](../models/JAVG_2026_misc.md)

All seven are academic papers in form, but fall into four categories by nature:
[Foundation-model class] ALIVE (a complete T2VA/R2VA foundation model comprising a 12B VideoDiT + 2B AudioDiT, with the Alive-Bench 1.0 evaluation benchmark), NAVA (a 6.3B natively aligned joint generation model, with open-sourced weights and training code).
[Module/method class (enhancements on top of an existing foundation model)] Baton (adding a VA-Planner semantic planner + RS-RoPE ahead of a joint DiT, a method-level enhancement), CCL (four module-level improvements — TARP/LCT/DCR/UCG — for the dual-stream Transformer paradigm, primarily aimed at saving data and compute).
[Task + dataset class] OmniCustom (proposing the new task of "synchronized audio-video customization," with one of its core contributions being the self-built dataset OmniCustom-1M, accompanied by a 100-sample evaluation set).
[System/engineering class] StreamChar (a long-horizon streaming generation system, with two-stage distillation and a real-time inference pipeline).
[Inference-time algorithm class] ITS-JAVG (a fully training-free inference-time scaling algorithm + a multi-verifier combination study + the ARW optimization algorithm, with no model training at all — essentially "using evaluation models to do inference-time data selection").
From a data-survey perspective: ALIVE, NAVA, and OmniCustom disclose substantive data-processing pipelines; Baton, CCL, and StreamChar disclose only a list of data sources; ITS-JAVG has no training data, but its "verifier combination" idea is highly isomorphic to a data-quality-check scorer.

### [Joint Audio-Video Generation Baseline Collection](../models/JavisDiT_baselines.md)

All are "models," of which three additionally come with an evaluation benchmark or dataset artifact:
- JavisDiT/JavisDiT++: model + evaluation benchmark (JavisBench, 10,140 items; JavisBench-mini, 1,000 items) + a synchronization evaluation metric (JavisScore) + an open-source training/inference toolchain; the only one of the four to deliver "model + benchmark + metric + complete training code" simultaneously.
- MM-Diffusion: model + self-built dataset (Landscape, 1,000 10-second natural-scene audio-video clips) + open-sourced code and pretrained weights.
- AV-DiT: a pure model (a parameter-efficient joint generation architecture), with no new dataset and no new benchmark.
- Harmony: model + evaluation benchmark (Harmony-Bench, 150 items, split into three tiers of subsets).
- UniAVGen: model (a unified framework supporting joint generation, video-to-audio dubbing, audio-driven video animation, and other multi-tasks), with no new dataset.

### [Kling 3.0 / Kling Video 3.0 Omni](../models/Kling_30_Omni.md)

A closed-source commercial model (a video generation/editing foundation model + a unified multimodal generation model), served via the Kling AI web app, mobile app, and open-platform API (including third-party hosting such as Replicate/fal); not a dataset, not a toolchain, not an evaluation benchmark

### [LTX-2](../models/LTX-2.md)

Model (an open-source text-to-"audio+video" joint generation foundation model, T2AV), with an accompanying open-source toolchain (ltx-core / ltx-pipelines / ltx-trainer inference and training/fine-tuning code, ComfyUI and Diffusers integration, and control LoRAs for camera/pose/lip-sync dubbing, etc.). Not a dataset, not an evaluation benchmark.

### [LongCat-Video](../models/LongCat-Video.md)

Model (a video generation foundation model + accompanying open-source inference code/operators). A 13.6B-parameter Diffusion Transformer that unifies text-to-video (T2V), image-to-video (I2V), and video continuation within a single model; it also open-sources a forward/backward implementation of Block Sparse Attention (which can be regarded as a toolchain component). Not a dataset, not an evaluation benchmark (though the report does contain a self-built internal evaluation set).

### [MOVA](../models/MOVA.md)

Model (a joint audio-video generation foundation model). It also comes with three derivative outputs: (1) a complete open-source code base (training pipeline, inference, LoRA fine-tuning, a prompt-enhancement workflow, evaluation code); (2) a self-built, six-scenario-category joint audio-video generation evaluation benchmark (used together with Verse-Bench); (3) an Arena-style human-preference evaluation protocol. Not a dataset release (the training data itself has not been open-sourced).

### [Merged entry: ① Mochi 1](../models/Mochi_MAGI_Motif.md)

All three are "model + accompanying toolchain":
① Mochi 1: a 10B-parameter Asymmetric Diffusion Transformer (AsymmDiT, 48 layers, visual dimension 3072 / text dimension 1536) text-to-video model + an in-house AsymmVAE (362M parameters, 8×8 spatial + 6× temporal compression to a 12-channel latent space, total compression ratio 96×). Outputs 848×480, 30 FPS, up to 5.4 seconds (84 frames).
② MAGI-1: a chunk-wise autoregressive denoising video world model, up to 24B parameters, 24 frames per chunk, a context of up to 4 million tokens, unifying T2V / I2V / V2V video continuation modeling with streaming generation; comes with an in-house Transformer VAE (8× spatial / 4× temporal downsampling, 16 channels), the MagiAttention distributed attention library, and Shortcut Model distillation.
③ Motif-Video 2B: a 2B-parameter rectified-flow-matching text-to-video model + an image-to-video extension, with its core innovation being Shared Cross-Attention and a three-stage backbone functional decomposition (early fusion / joint representation / detail decoding); TREAD token routing and REPA representation alignment are used on the training side; it additionally contributes an "offline bucket-balanced sampler" data-infrastructure component. Explicitly positioned as a controlled experiment for the "micro-budget" route.

### [Movie Gen](../models/Movie_Gen.md)

Model (a media generation foundation model family: Movie Gen Video 30B text-to-video, Personalized Movie Gen Video, Movie Gen Edit video editing, Movie Gen Audio 13B audio generation) + evaluation benchmarks (Movie Gen Video Bench, Movie Gen Audio Bench)

### [NVIDIA NeMo Curator](../models/NeMo_Curator.md)

Toolchain / data infrastructure framework (a data-curation toolkit). Not a generative model, not a dataset, and not an evaluation benchmark. Positioned as a "reproducible, GPU-accelerated framework for building large-scale data-processing pipelines," covering the full load–filter–deduplicate–annotate–transform–write-out flow for four modalities: text, image, video, and audio. Within this survey's object set, it belongs to the "upstream infrastructure used by other video generation/world model teams," and NVIDIA's official positioning calls it an open-source reference implementation of industrial-grade video data processing.

### [OmniHuman Dataset + OHBench](../models/OmniHuman.md)

A trinity of dataset + evaluation benchmark + data-processing toolchain, containing no generative model. It has three components: (1) the OmniHuman dataset — a human-centric audio-video dataset of 1 million videos / 1,800 hours / 80,000 identities, with hierarchical annotations; (2) a fully automated data-collection and multimodal annotation pipeline — four-tier filtering + tracking + audio-visual attribution + two-stage caption generation; (3) OHBench — a three-tier (global/relational/individual), seven-dimension evaluation system, with 509 evaluation videos and an accompanying metrics toolkit. The paper itself does not train a new model; it only uses LTX-2 fine-tuning as a validation experiment for the dataset's effectiveness.

### [Open-Sora Series](../models/Open-Sora.md)

Both are open-source projects with the dual property of "model + complete toolchain": both release model weights (T2V/I2V diffusion models + VAE) and end-to-end training code and data-processing code, along with some annotated datasets. Not evaluation benchmarks. Open-Sora additionally has the property of an "engineered template for training cost accounting" (stage-by-stage cost breakdown), while Open-Sora Plan additionally has the property of a "template for community-collaborative reproduction."

### [Ovi](../models/Ovi.md)

Model (an open-source text/text+image-to-"audio+video" one-shot joint generation model, T2AV / I2AV), with an accompanying open-source inference toolchain (inference scripts, a Gradio app, multi-GPU sequence-parallel inference, fp8/qint8 quantization, ComfyUI integration). Not a dataset, and not an evaluation benchmark (evaluation borrows the Verse-Bench proposed by others).

### [Script-a-Video](../models/Script-a-Video.md)

A method/representation-paradigm paper, with a triple property of "annotation schema definition + internal dataset construction + downstream generative-model adaptation":
[Core] Proposes the MTSS structured audio-video captioning representation paradigm (a JSON-style four-stream schema definition), belonging to captioning methodology rather than being a model or dataset.
[Accompanying output 1] An internal MTSS-annotated dataset of 500K video clips (not open-sourced).
[Accompanying output 2] A dedicated captioning model fine-tuned from Qwen3-Omni-Instruct, Qwen3-Omni-MTSS-FT (not open-sourced).
[Accompanying output 3] A multi-shot joint audio-video generation model adapted from LTX-2 (introducing two architectural improvements, Shot-Aware Structured Attention and Identity Customization; not open-sourced).
[Accompanying output 4] An internal evaluation set (125 single-shot + 100 multi-shot samples, not open-sourced).
Not an evaluation-benchmark release — evaluation uses existing benchmarks such as Video-SALMONN-2, UGC-VideoCap, Daily-Omni, and WorldSense.

### [Seedance 2.0 and Seedance 1.5 pro](../models/Seedance_20_Seedance_15_pro.md)

Closed-source commercial models (video/joint audio-video generation foundation models) + accompanying self-built evaluation benchmarks (SeedVideoBench 1.0/1.5/2.0). Both technical reports are closer to model cards + evaluation reports than complete methodology papers.

### [SkyReels Series](../models/SkyReels.md)

Primarily models, but with three categories of artifacts including toolchain and evaluation benchmark: (1) Models — the SkyReels-V2 video generation foundation (1.3B/5B/14B, 540P/720P, including T2V, I2V, Diffusion Forcing long-video, and Camera Director variants) and the SkyReels-V4 unified multimodal audio-video foundation (generation + inpainting + editing); (2) Toolchain — the SkyCaptioner-V1 structured video annotation model (based on Qwen2.5-VL-7B-Instruct, already open-sourced) and its inference code, and the SkyReels-V2 training/inference framework; (3) Evaluation benchmarks — SkyReels-Bench (V2, a human-evaluation benchmark) and SkyReels-VABench (V4, a set of 2,000+ joint audio-video evaluation prompts).

### [Sora 2](../models/Sora_2.md)

A closed-source commercial model (a video generation foundation model with native joint audio-video generation + a consumer social app + an API service). Not a dataset, not a toolchain, not an evaluation benchmark.

### [SpeakerVid-5M](../models/SpeakerVid-5M.md)

Primarily a dataset (a large-scale dyadic audio-video human-interaction generation dataset), with two accompanying derivative outputs: (1) an autoregressive video-chat baseline model (0.8B trainable parameters); (2) an accompanying evaluation benchmark, VidChatBench (500 test pairs, with a six-dimension metric system). It also open-sources a complete data-curation code base, so it also has toolchain properties. It is not a generative model itself, but rather one of the upstream data suppliers for joint audio-video generation models such as MOVA.

### [Step-Video-T2V](../models/Step-Video-T2V.md)

Model (an open-source text-to-video foundation model, a 30B-parameter DiT + an in-house deep-compression Video-VAE), with an accompanying evaluation benchmark, Step-Video-T2V-Eval (128 Chinese prompts across 11 categories), and open-source inference code. Not a dataset, not a data toolchain — the training data and data-processing code have not been released.

### [UniTalking](../models/UniTalking.md)

Model (a joint audio-video generation framework). The core is UniTalking, a 10B-parameter dual-stream MM-DiT joint audio-video generation model, whose video branch inherits Wan2.2-5B weights while the audio branch is a symmetric twin structure with random initialization. No dataset is released (the self-built dataset of 2.3 million samples has not been open-sourced), and no evaluation benchmark is released (evaluation reuses Seed-TTS test-en, the MiniMax Multilingual Test Set, and a self-built subjective test set of 50 prompts, the latter also not made public).

### [UniVerse-1](../models/UniVerse-1.md)

A combined output of model + evaluation benchmark. The core is a joint audio-video generation model (UniVerse-1-Base, 7B parameters); it also releases the Verse-Bench evaluation benchmark (600 image-text prompt pairs, with an accompanying evaluation-metric toolkit). Not a dataset release — the 7,685 hours of training data itself has not been open-sourced. Complete inference code is included, but not training code or data-cleaning scripts.

### [Unison](../models/Unison.md)

Model (a joint audio-video generation framework). This is the point in this entry most in need of clarification: Unison is not a dataset-release work, and not an evaluation benchmark.
[It is a model] The core output is a dual-branch joint audio-video generation framework (a Wan2.2-5B video branch + an MMAudio/Zipformer audio branch), with three method-level innovations as its core contribution (Semantically-Guided Harmony Strategy SGHS, Bidirectional Audio Cross-Attention Bi-ACA, and a Bidirectional Cross-Modal Forcing Strategy CMFS).
[It is not a dataset] The roughly 2 million synchronized audio-video clips / 3,000+ hours of corpus is the product of "aggregating existing open-source datasets + an automated pipeline for refinement," but it was neither named nor released with statistics nor open-sourced — it is described in just two sentences in the experimental-setup section. The task description's claim of "an automated pipeline aggregating multiple large-scale datasets" is accurate, but note that this pipeline is barely elaborated on in the paper.
[Self-built evaluation set] A held-out internal test set of 1,000 samples was also built (with annotations generated by Gemini), but it was neither named nor open-sourced, and does not constitute a public evaluation benchmark.
[Task form] The primary task is TI2AV (text+image → audio+video), while also supporting T2AV (text-only → audio-video), A2V (audio → video), and V2A (video → audio) — four task types in total.

### [Veo 3 / Veo 3.1](../models/Veo_3_Veo_31.md)

Model (a closed-source commercial video+audio joint generation foundation model, served via the Gemini App, Flow, Google Vids, Google AI Studio, Gemini API, and Vertex AI)

### [Vidu S1](../models/Vidu_S1.md)

Model (a real-time, interactive, streaming joint audio-video generation model, voice-driven digital human/virtual character). It also self-builds an accompanying evaluation benchmark, Vidu-StreamBench (500 samples, an internal benchmark, not seen to be open-sourced)

### [Wan 2.5 / 2.6 / 2.7](../models/Wan.md)

Model (a video generation foundation model series). 2.5/2.6/2.7 are closed-source commercial API models; the same family also includes open-source models (Wan2.1/2.2/S2V/Animate/Dancer), an open-source inference and fine-tuning toolchain (the GitHub Wan-Video organization, ComfyUI/Diffusers integration, the Wan-skills Agent skill pack), and a self-built evaluation benchmark, Wan-Bench (3 major dimensions, 14 fine-grained metrics). Not a dataset entry.

### [Audio-Video Generation Evaluation Benchmark Collection](../models/av_benchmarks.md)

Evaluation benchmarks. All five are evaluation systems for the joint audio-video generation (Audio-Video Generation) direction, but with different emphases:
- VABench: a comprehensive, full-dimensional benchmark (three task types — T2AV / I2AV / stereo AV — spanning seven content categories + 15 evaluation dimensions);
- AVBench: a human-aligned automated evaluation benchmark + a trainable dedicated evaluator (10 dimensions, accompanied by 300K preference training samples; the evaluator itself can be reused as a data filter and an RLHF reward model);
- AV-SyncBench: a dedicated synchronization benchmark (decoupling temporal synchronization from semantic synchronization), which is also a dataset with perturbation annotations;
- PhyAVBench: a dedicated physical-commonsense benchmark, accompanied by a self-recorded real-world dataset, PhyAV-Sound-11K (11,605 videos / 25.5 hours);
- Omni-Judge: an evaluation-methodology study (investigating whether Omni-LLMs can serve as human-aligned judges), belonging to meta-evaluation.
Of these, AVBench, AV-SyncBench, and PhyAVBench simultaneously have "dataset" properties.

### [Video Captioning Model Ecosystem](../models/caption_models.md)

A composite ecosystem of model + toolchain + evaluation benchmark, in three layers:
(1) Captioning models (captioners) themselves — general-purpose VLMs (the Qwen-VL / InternVL / LLaVA families), specialized captioners (ShareCaptioner-Video, Tarsier2, CogVLM2-Caption, SkyCaptioner-V1, AuroraCap, AVoCaDO, AVSCap), and omni-modal captioners (Qwen3-Omni-Captioner, video-SALMONN 2, Qwen3.5-Omni);
(2) Datasets produced by these captioners (ShareGPT4Video-40K / 4.8M, Tarsier2-Recap-585K, Panda-70M, Koala-36M, AVoCaDO-SFT-107K, AVSCap-130K) — captioning models and datasets are mutually causal, a core feature of this ecosystem;
(3) Evaluation benchmarks (DREAM-1K, VDC/VDCScore, VidCapBench, AVSCapBench, UGC-VideoCap, Omni-Cloze, OmniCap-IF, the video-SALMONN 2 test set).

### [Geometric/Structured Annotation Dataset Collection](../models/geometric_datasets.md)

Datasets (all four are large-scale video datasets with accompanying self-built annotation pipelines; WildWorld additionally releases the WildBench evaluation benchmark, Action100M additionally releases the VL-JEPA pretrained model, and SpatialVID releases complete annotation-pipeline code)

### [Post-Training Data Strategies for Video Generation](../models/post_training_data.md)

A cross-cutting topic entry, not a single model/dataset/toolchain/evaluation benchmark. The anchor paper itself is of the type "methodological technical report" (a practical blueprint / systematic framework); it does not release model weights and does not release a dataset, belonging instead to an engineering blueprint for a four-stage post-training pipeline. The other horizontally covered objects fall variously into models (the majority), datasets (HPDv3, the VideoReward preference set), and reward models (HPSv3, VideoAlign/VideoReward, RewardDance, VisionReward).

### [Merged Survey of Mainstream Video Pretraining Datasets: Panda-70M, InternVid, Koala…](../models/pretraining_datasets.md)

All are "datasets," but each comes with a varying degree of accompanying toolchain and models: Panda-70M comes with splitting code + a distilled captioning student model; InternVid comes with the ViCLIP video-text model; Koala-36M comes with transition-detection code + a VTSS scoring model (simplified version); MiraData comes with a GPT-4V captioning script + the MiraBench evaluation benchmark (also possessing "evaluation benchmark" properties); OpenVid-1M comes with the MVDiT generative model and training code; UltraVideo comes with the UltraWan-1K/4K generative model (LoRA); LVD-2M comes only with a YouTube download script. None of the seven has released a commercial-grade generative model.
## Openness (whether weights/code/data/pipeline are each open-sourced)

`openness` · Detail level: brief

### [Allegro](../models/Allegro.md)

Overall openness is relatively high, but the data itself is not open:
· Weights: fully open-sourced, Apache 2.0 license (commercial use allowed), Hugging Face rhymes-ai/Allegro, rhymes-ai/Allegro-TI2V, including a 175M VideoVAE + 2.8B VideoDiT.
· Code: both inference code and training code are open-sourced (GitHub rhymes-ai/Allegro); training requires users to supply their own dataset in .parquet format.
· Pipeline: disclosed at high granularity in paper form — the tool name for each step of the 7-level filtering funnel (PySceneDetect / DOVER / LPIPS / UniMatch / LAION Aesthetics Predictor / CRAFT / Tag2Text / CLIP) and an item-by-item threshold table for each training stage (Table 1) are all disclosed, along with the retention amount at each stage. However, the cleaning code and threshold configuration scripts were not open-sourced along with the weights.
· Annotation model: Aria (rhymes-ai/Aria, a 25.3B total-parameter / 3.9B-activated multimodal native MoE, Apache 2.0, arXiv:2410.05993) has been open-sourced, but the fine-tuned version used for video captioning has not been separately released.
· Data: the training dataset itself is not open-sourced; it is only stated to be built based on public datasets such as WebVid / Panda-70M / HD-VILA / HD-VG / OpenVid-1M.

### [Apollo](../models/Apollo.md)

Fully closed-source — the lowest openness tier among the samples surveyed in this study.
[Weights] Not open-sourced; neither the paper nor the Hugging Face paper page includes a model link.
[Code] Not open-sourced; the paper does not provide a GitHub repository or project homepage URL.
[Data] The 81M-sample audio-video-caption triplet dataset has not been released; the paper's claim of being the "first large-scale densely-captioned audio-video dataset" is only stated as a contribution claim, with no public commitment of any kind.
[Pipeline] Framework-level disclosure was made at the methodology level (a four-stage funnel, a list of the annotation models used, alignment detection tools, a 27% retention rate), but key elements needed for reproducibility are missing: no per-metric threshold table, no per-stage input/output volumes, no original prompt text, no pseudocode, no data-processing scripts. Compared with contemporaneous open-source work (e.g., MOVA publishing the full Table 9 threshold table and the original annotation prompt text), Apollo's data disclosure stays at the level of "stating what was done without saying how it was done"; the only respect in which it exceeds its peers is providing the 27% end-to-end quantitative retention rate.

### [CineDance / CineDance-1M](../models/CineDance.md) ⚠️

Openness is medium-to-upper, but as of July 2026 it is still in a "phased release" state:
[Data] Partially open. Released on Hugging Face in gated (application-required) form; the first batch, CineDance_01, is the 1st of four shards, containing about 240,488 video clips, 150 TAR archives, totaling 5.83 TB. Currently it contains only the video itself (video only, with native audio tracks retained inside the video container); structured annotation files have not yet been released with the first batch. The license is CC-BY-NC-SA-4.0 (Attribution-NonCommercial-ShareAlike), explicitly restricted to non-commercial research and educational use. Downloading requires a Hugging Face token and manual review of the application.
[Code] Not open-sourced. The GitHub repository github.com/AliothChen/CineDance has been created, but the curation pipeline code, inference suite, inference code, and training code are all listed as pending release.
[Weights] Not open-sourced. The CineDance model checkpoint is listed as pending release.
[Pipeline] Methodology-level disclosure is thorough — the tool selection at each level of the three-stage process (EasyOCR, FFmpeg letterbox/black-bar detection, TransNetV2, Qwen3.5-27B, Qwen3.5-35B-A3B, Qwen3-Omni-30B-A3B), quantitative data at each funnel level, annotation schema fields, and ablation comparison tables are all given in the body text and can be reimplemented by a third party; however, the specific original prompt text and filtering threshold values have not been fully disclosed.
[Dependency statement] The README acknowledges LTX-2, the Qwen series, and vLLM. The repository itself does not label a code license. [Uncertain]

### [CogVideoX](../models/CogVideoX.md)

Its degree of openness is on the higher tier among contemporaneous closed-source large vendors, but the data itself is not open-sourced:
· Weights: open-sourced. CogVideoX-2B (Apache 2.0), CogVideoX-5B / 5B-I2V, and CogVideoX1.5-5B / 1.5-5B-I2V are all publicly released on Hugging Face (THUDM/zai-org organization) and in SAT format, allowing commercial use.
· Code: open-sourced. Inference, fine-tuning (LoRA/SFT), and both the SAT and diffusers implementations are all on GitHub at https://github.com/zai-org/CogVideo (formerly THUDM/CogVideo).
· Data-processing pipeline: publicly described in paper form (Section 3.4 Data + Appendix G "Dense Video Caption Data Generation" + Appendix K "Data Filtering Details"), disclosing the negative-label system, 6 Video-LLaMA classifiers and their test-set accuracy tables, the caption generation chain, and the full text of the GPT-4 summarization prompt; however the cleaning code and threshold configuration files are not open-sourced, and the dataset itself is not open.
· Annotation model: open-sourced. Both CogVLM2-Caption (huggingface.co/zai-org/cogvlm2-llama3-caption) and the 3D VAE weights have been released, which is the most substantive manifestation of "data pipeline openness" — externally, the annotation stage can be directly reproduced.
· Training data: not open-sourced; the specific sources and inventories of the 35M clips and 2B images have not been disclosed.
· CogSound (the sound-effect model): closed-source, available only in the Qingying (智谱清影) product and API, with no technical report.

### [Cosmos-Predict2.5](../models/Cosmos-Predict25.md)

Openness is relatively high among closed-source large-vendor models of the same tier, but the training data itself is not open.
[Open-sourced] (1) Source code: GitHub nvidia-cosmos/cosmos-predict2.5 and nvidia-cosmos/cosmos-transfer2.5, under the Apache 2.0 License; (2) pre-trained and post-trained checkpoints: 2B / 14B pre-trained and post-trained versions, along with distilled and domain-specialized versions (auto/multiview, robot/action-cond, robot/multiview-agibot, robot/gr00tdream-gr1), released on Hugging Face under the NVIDIA Open Model License; (3) curated benchmarks (PAI-Bench-related evaluation sets) and "curated post-training examples"; (4) the engineering foundation of the data pipeline is indirectly open-sourced — Cosmos Curator (github.com/NVIDIA/cosmos-curator) and its GPU streaming execution framework Cosmos-Xenna have been independently open-sourced, and NeMo Curator's video curation module is also public; together these are the productized form of the curation pipeline described in the paper.
[Not open-sourced] The 200M pre-training video dataset itself; the in-house content-type classifier (a 26-class taxonomy), and the weights and thresholds of the internal aesthetics/motion/OCR/perceptual-quality/semantic-artifact scorers; the 3.1M-segment, 7-camera proprietary data collected from NVIDIA's internal driving platform; the specific hyperparameters of each filter.
[Positioning] The paper explicitly states its goal is to "lower the barrier to adoption," handing pre-trained models to the community for domain specialization, rather than opening the data.

### [Data-Juicer 2.0](../models/Data-Juicer.md)

Openness is extremely high — the most open category of object in this survey.
[Code] Fully open-sourced. GitHub datajuicer/data-juicer (formerly modelscope/data-juicer), Apache 2.0 license, about 6.8k stars, with sustained high-frequency iteration (7 versions from v1.4.5 to v1.5.4 released within 2026 alone). The implementations of all 229 operators are fully readable and editable; v1.5.3 added 409 test cases in a single release, reflecting a relatively high engineering standard.
[Operators and recipes] Open-sourced. config_all.yaml exposes all operators and their hyperparameters; the DJ-Cookbook maintains 20+ ready-to-use data recipes (including vertical scenarios such as video data synthesis, contrastive learning, and curriculum learning).
[Data] Partially open-sourced. In the text-to-video use case, the optimal data pool obtained after filtering is publicly released as a Hugging Face dataset: datajuicer/data-juicer-t2v-optimal-data-pool, totaling 147,176 samples, about 227.5GB, under the Apache 2.0 license; however the underlying raw materials (InternVid, Panda-70M, MSR-VTT) must be obtained separately under their respective original licenses — what DJ makes public is the filtered sample index and metadata.
[Model weights] The Sandbox paper states that the T2V model it trained (fine-tuned from T2V-Turbo/EasyAnimate) is open-sourced together with the code and data.
[Pipeline] Fully public. Not only is the methodology disclosed, but the executable YAML recipes and threshold values are also disclosed (e.g., CLIP similarity threshold 0.306337), giving a lower reproduction barrier than the vast majority of model-side work.
[Documentation] A bilingual Chinese-English documentation site (datajuicer.github.io), including item-by-item Operator Schemas descriptions.

### [Foley-Omni](../models/Foley-Omni.md)

Openness is medium-to-upper.
[Weights] Open-sourced. Hugging Face CocoBro/Foley-Omni releases inference-only weights (v2st.pth), distributed together in the same package with dependent components such as the audio VAE, the BigVGAN vocoder, and a visual feature extraction model, stated to be under the MIT license.
[Code] Partially open-sourced. GitHub NJU-Speech/Foley-Omni provides inference code and visual feature preprocessing scripts (CLIP feature and Synchformer sync-feature extraction); training code has not been released. The repository's own license is not explicitly labeled on the page, and it states that it redistributes upstream components from Wan2.2-TI2V-5B, MMAudio, and Ovi, which must each follow their respective upstream licenses.
[Data] Training data is not open-sourced. Of the approximately 4.9k hours / 2.7M-pair training corpus, the self-collected/internal portion is not public; the public-dataset portion (LJSpeech, LibriTTS, AudioCaps, Freesound, MusicCaps, MusicBench, AudioSet, VGGSound, GRID, LRS2, Chem, SpeakerVid, TalkVid, Kling-Foley) can be obtained independently.
[Evaluation benchmark] Open-sourcing is promised but not yet released. The paper states it will release the annotations, metadata, and processing scripts of V2ST-Bench; due to copyright restrictions, the original video files are not directly distributed — instead URL + metadata is provided (similar to the approach used by VGGSound/HowTo100M).
[Pipeline] Methodology-level disclosure is fairly thorough: Table 7 gives specific threshold values for six filtering metrics, Table 12 gives the Gemini 2.5 Pro annotation prompt template, and acoustic post-verification gives an explicit -35 dB threshold formula. However the complete cleaning scripts were not released, nor was a per-stage retention-rate table provided.

### [Goku](../models/Goku.md)

Overall it belongs to the type "paper public, weights and data closed."
[Made public] (1) The technical paper fully discloses the thresholds at each level of the data pipeline, the model architecture (Goku-2B: 28 layers / dim 1792 / 28 heads; Goku-8B: 40 layers / dim 3072 / 48 heads), the rectified-flow training recipe, and the parallel training infrastructure; (2) the GitHub repository Saiyan-World/goku provides a code directory skeleton (configs/, goku/, tools/, etc.) and a requirements.txt; (3) the project homepage provides a large number of generated-sample visualizations.
[Not open-sourced] Model weights (no official Goku weights are released on Hugging Face), training data, data-processing pipeline code, the internal video classification model, the internal aesthetic scoring model. No LICENSE file is present in the repository either.
[Conclusion] Weights: no; code: partial/incomplete; data: no; pipeline: paper text description only, code not open-sourced.
Note: many third-party websites/products in the community named "Goku AI" are riding on the name and are not official releases.

### [Hailuo / MiniMax Video](../models/Hailuo.md)

Fully closed-source — one of the objects with the least data disclosure in this survey:
- Model weights: not open-sourced; inference service is only provided through the Hailuo (海螺AI) web/app, the MiniMax open platform API, and third-party hosting platforms such as Replicate and fal.ai;
- Training/inference code: not open-sourced;
- Training data: not open-sourced, not described;
- Data-processing pipeline: not open-sourced, not described;
- Technical report/paper: none at all. Across generations, there have only been product-launch blog posts (minimax.io/news), with content focused on capability demonstrations and pricing, containing almost no technical detail; the NCR architecture likewise has only a one-sentence naming-level description with no paper backing it.
By contrast, MiniMax's openness on the language-model side is very high (MiniMax-01, M1, M2/M2.1/M2.5/M2.7, M3 all have weights and technical reports released on Hugging Face), whereas the video side is deliberately fully closed. The only open-source artifact related to visual generation is the VTP (Visual Tokenizer Pre-training) series of visual tokenizers released on 2025-12-15 (Small 0.2B / Base 0.3B / Large 0.7B, Modified MIT license, arXiv:2512.13687), but it is positioned as an image-tokenizer base component; the model card does not explain any direct relationship to the Hailuo video model, and its training-data composition is likewise not clearly listed in the model card.

### [HunyuanVideo-Foley](../models/HunyuanVideo-Foley.md)

Openness is medium, a typical case of "weights and inference open, training and data closed."
[Weights] Open-sourced. Hugging Face tencent/HunyuanVideo-Foley provides two variants: XXL (default, hunyuanvideo_foley.pth, requiring about 20GB VRAM for regular inference, 12GB in offload mode) and XL (hunyuanvideo_foley_xl.pth, 16GB / 8GB offload). It also opens the weights of the in-house 48kHz DAC-VAE audio autoencoder and the Synchformer sync-feature extractor configuration.
[Code] Inference code is open-sourced (infer.py for batch inference, gradio_app.py web interactive interface, hunyuanvideo-foley-xxl.yaml config file), based on Hugging Face diffusers. Training code is not open-sourced.
[Data-processing pipeline] Only described procedurally in about one paragraph in Section 3.1 of the paper — the order of the seven stages and some thresholds (8-second segmentation, 80% silence-ratio, 32 kHz effective sample rate) have explicit values, but no cleaning scripts, model configurations, prompts, or per-stage statistics are disclosed. Reproducibility sits at the level of "knowing what was done but not how the parameters were tuned."
[Data] The 100,000-hour TV2A dataset is entirely closed, and the raw source has not been disclosed either.
[License] The tencent-hunyuan-community license (a non-standard, non-OSI open-source license); the Hugging Face model card sets extra_gated_eu_disallowed: true, explicitly restricting access for users in the EU region. This is the consistent practice of Tencent's Hunyuan model series; commercial use must comply with the community license terms.

### [HunyuanVideo](../models/HunyuanVideo.md)

A typical example of the "weights + code open, data and data pipeline not open" pattern.
[Weights] Open-sourced. HunyuanVideo 13B (DiT backbone + 3D VAE + text encoder) is released on Hugging Face (tencent/HunyuanVideo, tencent/HunyuanVideo-I2V); HunyuanVideo 1.5 (8.3B, including T2V/I2V/super-resolution modules) is released on GitHub at Tencent-Hunyuan/HunyuanVideo-1.5 and on Hugging Face.
[Code] Open-sourced. Inference code, parallel inference, quantization, LoRA, and ComfyUI/Diffusers integration are all provided; training code is not fully open.
[License] The Tencent Hunyuan Community License, a non-standard OSI open-source license, imposes usage restrictions related to regions such as the EU and monthly active user counts.
[Data] Not open-sourced. The training dataset itself, the filtered lists at each level, and the caption data are all undisclosed.
[Pipeline] Methodology-level disclosure is detailed (in particular, the original version's description of the hierarchical filtering funnel, the structured caption schema, and the shot-motion classifier is the most detailed among contemporaneous closed-source models), but the filter code and model weights (such as the in-house VideoCLIP, blur-detection model, YOLOX-like detector, and the caption VLM) are all not open-sourced, so it cannot be directly reproduced.

### [InstructAV2AV](../models/InstructAV2AV.md) ⚠️

Openness is relatively high — one of the best in terms of data openness among the audio-video works covered in this survey.
[Weights] Open-sourced. Hugging Face suimu/InstructAV2AV releases pretrained weights and provides six checkpoints fine-tuned for different subtasks (covering general editing, content addition/removal, identity/voice cloning scenarios, etc.). [Uncertain] This model repository page does not have a complete model card; the parameter count, license, and other information are not labeled on the model page.
[Code] Partially open-sourced. The GitHub repository releases inference code (scripts/edit.py, scripts/demo.py) and pipeline scripts under the Apache-2.0 license; training scripts are listed as "in progress" in the README roadmap and have not yet been released. It depends on upstream Wan-AI/Wan2.2-TI2V-5B and hkchengrex/MMAudio.
[Data] Fully open-sourced — this is the most notable openness point of this work. Hugging Face suimu/InsAVE-80K is released under the MIT license, actually packaging 88,074 video-audio pairs (176,148 files) in 11 tar shards, about 139 GB, distributing the video and audio files themselves rather than only URLs — this is relatively rare among audio-video datasets (compare Foley-Omni's V2ST-Bench, which only publishes URL+metadata due to copyright restrictions). The data card also reminds users to independently verify the rights compliance of the underlying media content.
[Pipeline] Methodology-level disclosure is thorough: it fully lists the specific models used at each stage (PySceneDetect, CoTracker3, LAION Aesthetics Predictor, PyDub, Audiobox-Aesthetics, TalkNet, ElevenLabs Scribe, Grounded-SAM-2, Qwen3-Omni, SAM-Audio, ElevenLabs TTS, Wan2.2-5B), but the specific values of most filtering thresholds are not given (only the -45 dBFS one has an explicit number), and there is no per-stage retention rate, so reproducibility is limited.

### [2026 Other Joint Audio-Video Generation](../models/JAVG_2026_misc.md) ⚠️

Openness varies enormously, from fully open-source to technical-report-only:
[Most open | NAVA] Apache 2.0 source-code license. Already released: complete inference pipeline, training code, a Gradio interactive demo, model weights NAVA.safetensors (24GB bf16) and a quantized version NAVA_fp8.safetensors (about 7GB). Dependent components reuse the Wan2.2-5B VAE, the T5 encoder, the LTX audio-VAE, and the ReDimNet speaker embedder (each under its own original license). Not open-sourced: the training data itself and the data-processing pipeline code (the repository explicitly mentions training data but no data pipeline has been released).
[Medium | OmniCustom] Provides a GitHub repository and project homepage, so the code is obtainable; its self-built OmniCustom-1M is derived from the public dataset SpeakerVid-5M, so the data construction process is reproducible (the filtering rules are all public), but whether the filtered list has been released is unclear [uncertain].
[Medium | StreamChar] Has a project homepage at humanaigc.github.io/StreamChar_page/; the training data all comes from three public datasets (SpeakerVid-5M / TalkVid / OpenHumanVid) plus Emilia, so the data side is highly reproducible; the release status of weights and code is unclear [uncertain].
[More closed | ALIVE] The GitHub repository FoundationVision/Alive is currently only a technical-report landing page (containing an assets directory, an arXiv link, a project page, and a Discord demo entry); no weight downloads, training code, inference code, or data release is seen; Alive-Bench 1.0 (264 general prompts + 90 reference-character prompts) has been defined in the paper, but its public-release status is unclear [uncertain]. Among the seven items, it has the most detailed data disclosure while having the lowest code openness.
[Closed | Baton / CCL] Paper only; the training data includes in-house / self-collected internet portions, not open-sourced; CCL's in-house data (interviews, short dramas, films) is explicitly unavailable.
[N/A | ITS-JAVG] Training-free, so no weights need to be open-sourced; the paper states "Project materials and code are available online," so the code and prompt set should be obtainable [uncertain: the specific repository address has not been verified]. The verifiers it depends on (VideoReward, JavisScore, ImageBind, VQAScore, AVHScore) and the base generators (JavisDiT, MMDisCo, LTX-2) are all already-open-source assets, giving it the highest reproducibility.

### [Joint Audio-Video Generation Baseline Collection](../models/JavisDiT_baselines.md) ⚠️

Openness varies enormously and can be divided into three tiers:
[Fully open-source tier]
- JavisDiT / JavisDiT++: GitHub JavisVerse/JavisDiT — weights (Hugging Face JavisVerse/JavisDiT-v1.0-jav), inference code, the complete three-stage training scripts, and evaluation tools (meta files and precomputed caches for 16 audio-video metrics) are all open; the JavisBench and JavisBench-mini benchmark data are public. The data side is semi-open: the first-stage audio pretraining data has been preprocessed and released on Hugging Face (JavisData-Audio); the second-stage video data comes from TAVGBench, and the repository provides a list of 330,000 video IDs, but explicitly states that "the original YouTube videos cannot be released due to copyright issues"; the third-stage DPO preference data is "being prepared for release." It has the highest data transparency in this collection.
- MM-Diffusion: MIT license; GitHub researchmm/MM-Diffusion opens all code, training scripts, evaluation pipeline, and 6 checkpoints (Landscape.pt / Landscape_SR.pt / AIST++.pt / AIST++_SR.pt / a guided-diffusion upsampling initialization model / i3d and AudioCLIP evaluation models); the Landscape and AIST++_crop preprocessed data are provided directly for download via Google Drive and Baidu Netdisk — it is the only work that also releases the training data itself.
[Promised-open-source tier]
- AV-DiT: the paper states "source code and pretrained models will be released"; as of the survey no confirmed public repository was found [uncertain]. The two datasets used (AIST++, Landscape) are themselves public.
[Closed-source tier]
- Harmony: the paper provides no code/weight repository link; whether Harmony-Bench is public is not stated [uncertain]; the self-collected 2 million environmental-sound audio-video clips in the training data are not open-sourced.
- UniAVGen: the paper provides no open-source information; the "internally collected real-person audio-video dataset" used for training is completely closed [uncertain].
Note: both Harmony and UniAVGen are built on open-source models as their base (Harmony uses Wan2.2-5B, MMAudio's audio VAE, F5-TTS's speech encoder, and the umT5 text encoder; JavisDiT++ uses Wan2.1-1.3B-T2V; JavisDiT v1 uses Open-Sora's video VAE and AudioLDM2's audio components), a pattern of "standing on an open-source base while itself being closed/semi-open."

### [Kling 3.0 / Kling Video 3.0 Omni](../models/Kling_30_Omni.md) ⚠️

Weights: not open-sourced; code: not open-sourced; training data: not public; data-processing pipeline: only qualitatively described in the technical report, not open-sourced. A typical closed-source commercial API model. [Uncertain: whether any component will be open-sourced later]. Open-source assets from the same team can serve as indirect circumstantial evidence: the Koala-36M video dataset and its cleaning/annotation pipeline (github.com/KwaiVGI/Koala-36M, arXiv:2410.08260), and the Kling-Foley video-to-audio model and the Kling-Audio-Eval evaluation set (arXiv:2506.19774), are both open-sourced by the Kuaishou Kling team, and can be viewed as a public slice of the Kling series' data-processing methodology, but are not equivalent to the actual training data of 3.0 Omni.

### [LTX-2](../models/LTX-2.md)

Openness is the highest among joint audio-video generation models of the same category, but the data side remains closed.
[Open-sourced] (1) All model weights: ltx-2-19b-dev (bf16, trainable), fp8/fp4 quantized versions, the 8-step distilled version ltx-2-19b-distilled, as well as spatial/temporal upsamplers, all released on Hugging Face (Lightricks/LTX-2, roughly 427,000 monthly downloads); (2) inference code and multi-backend integration; (3) training/fine-tuning code ltx-trainer (supporting LoRA, full-parameter fine-tuning, IC-LoRA); (4) the technical report discloses architecture details publicly.
[Not open-sourced] The training data itself, the data-processing pipeline code, the in-house captioner model, the aesthetic scoring model, and data statistics figures.
[License] The LTX-2 Open Weights License: free for academic research; free for commercial use by companies with annual recurring revenue (ARR) below $10 million; a commercial license from Lightricks is required above that threshold. Officially self-described as "the first truly open-weight production-grade audio-video generation model."

### [LongCat-Video](../models/LongCat-Video.md)

High openness for weights and code, data side fully closed.
[Open-sourced] (1) Model weights: Hugging Face meituan-longcat/LongCat-Video (13.6B), plus subsequent LongCat-Video-Avatar (wav2vec2 audio encoder) and LongCat-Video-Avatar-1.5 (Whisper-large-v3 audio encoder); (2) inference code: GitHub meituan-longcat/LongCat-Video, containing multiple inference modes and a Streamlit interactive interface; (3) the forward and backward implementations of the Block Sparse Attention operator are open-sourced together with the base model; (4) the technical report publicly discloses the architecture, a training-stage table, and RLHF details.
[License] MIT License (weights and code), commercially usable — one of the most permissive licenses among video-generation models of the same scale.
[Not open-sourced] The training data itself, the data-processing pipeline code, the in-house captioner (a fine-tuned LLaVA-Video), the in-house aesthetic/blur/watermark scorers, the in-house fine-tuned VideoAlign reward model, and all data-scale statistics. Training code is also not open-sourced (inference only).

### [MOVA](../models/MOVA.md)

Openness is among the highest tier for joint audio-video generation models of the same category, using the Apache-2.0 license, allowing unrestricted commercial use.
[Weights] Open-sourced. Releases two variants, MOVA-360p and MOVA-720p (Hugging Face: OpenMOSS-Team/MOVA-360p, OpenMOSS-Team/MOVA-720p, and the mova collection).
[Code] Open-sourced and covering the full chain: training pipeline, efficient inference, LoRA fine-tuning scripts, prompt-enhancement (rewriter) workflow, and evaluation code. The paper explicitly states: "we release all model weights along with training, inference, and fine-tuning code."
[Data-processing pipeline] Methodology-level disclosure at an industrial-grade level of detail in Section 3 and Appendices A.3/A.4/A.5 of the paper — including the complete three-stage funnel structure, a per-metric filtering threshold table (Table 9), a per-stage retention-rate table (Table 1), speech-window segmentation pseudocode (Algorithm 1/2), and the full original text of all annotation prompts together with complete caption examples. This is one of the samples in this survey with the highest reproducibility of its data-processing methodology. However the codebase does not separately release data-cleaning scripts, only providing the dataset interface mova/datasets/video_audio_dataset.py; users must supply their own video/audio data and connect it according to the configuration.
[Data] The training data itself is not open-sourced. The public-dataset portion (VGGSound, AutoReCap, ChronoMagic-Pro, ACAV-100M, OpenHumanVid, SpeakerVid-5M, OpenVid-1M) can be obtained independently, but the specific list of the "filtered HQ subset" has not been released; the in-house data (Chinese TV dramas, animation, films, YouTube scraping, etc.) is not public.

### [Merged entry: ① Mochi 1](../models/Mochi_MAGI_Motif.md)

① Mochi 1 (partially open, data side fully closed): weights and inference code are Apache 2.0 open-sourced (GitHub genmoai/mochi, HF genmo/mochi-1-preview), AsymmVAE is open-sourced together with it, allowing commercial use. However there is no technical report/paper, and no data-pipeline disclosure of any kind; the training data, cleaning process, and annotation method are all undisclosed, and officially the stance taken toward the data sources is explicitly "not disclosed for competitive reasons." Can be viewed as "open weights, closed method, closed data."
② MAGI-1 (highly open, data methodology open but numeric values closed): Apache 2.0 open-sources the 24B / 4.5B base and distilled weights, the FP8 quantized version, and inference code, and separately open-sources the independent MagiAttention library; the 61-page technical report fully discloses all methodology of Section 3 DATA (the segmentation tools, 11 categories of filtering actors, the dual-model deduplication, MLLM secondary filtering, two types of caption schemas, and the three-stage data configuration table). However all filtering thresholds are expressed only as "predefined threshold / lower and upper thresholds" without giving numeric values, the dataset scale is given only at the order-of-magnitude level of "tens of petabytes of raw material," and the cleaning code and the data itself are not open. Can be viewed as "open weights, open methodology, closed parameters, closed data."
③ Motif-Video 2B (the most thorough data disclosure among the three): weights Apache 2.0 open-sourced; the technical report writes the data pipeline as a reproducible engineering document — named tools (NeMo Curator, ffmpeg cropdetect, PaddleOCR-VL, SigLIP, Aesthetic Predictor V2.5, DOVER, UniMatch, SSCD sscd_disc_mixup, NVIDIA cuVS, Qwen3-VL-30B-A3B), named hyperparameters (SSCD 512-dimensional descriptor / 320×320 / 10th frame / cosine≥0.9 / k=64 / nprobe=16, OCR clustering requiring appearance in ≥50% of frames, exclusion of the outer 20% region on each side, caption sampling probabilities 0.5/0.3/0.2, SA optimization with 30,000 iterations, rolling-shuffle window of 4096), and a complete 10-stage curriculum. However the dataset itself and the cleaning code are likewise not open-sourced, and the Sankey diagram (Fig. 7) gives only flow directions, not absolute values. Can be viewed as "open weights, open methodology, open parameters, closed data."

### [Movie Gen](../models/Movie_Gen.md)

Semi-open: model weights are not open-sourced, training/inference code is not open-sourced, and training data is not public. However the (92-page) technical report discloses in extreme detail the data-cleaning pipeline, the filtering thresholds and funnel retention rates at each level, the annotation scheme, and the training recipe — one of the industry's most thorough public data-engineering documents. The open-sourced portion is the evaluation assets: Movie Gen Video Bench (1003 prompts) and Movie Gen Audio Bench, along with non-cherry-picked generated result videos/audio on these two benchmarks, hosted at https://github.com/facebookresearch/MovieGenBench. The paper explicitly states the model is for research purposes only and requires further improvements before deployment.

### [NVIDIA NeMo Curator](../models/NeMo_Curator.md)

Openness is the highest of its kind, belonging to the category of "pipeline fully open-sourced, data not included at all."
[Code] All NeMo Curator source code is Apache License 2.0, publicly hosted on GitHub (NVIDIA-NeMo/Curator), with the PyPI package name nemo-curator; official recommendation is installation via a preconfigured Docker container (the video/audio workflow requires preconfigured FFmpeg 8.0.1 + NVENC).
[Underlying engine] Cosmos-Xenna is likewise independently open-sourced under Apache 2.0, and can be used standalone outside of Curator.
[Productized implementation] The Cosmos-Curate source code is Apache 2.0; the model weights it invokes follow the NVIDIA Open Model License, with custom commercial licenses available on request.
[Pipeline configuration] Starting from 26.02, YAML declarative definition of an entire curation pipeline is supported, further lowering the reproduction barrier.
[Model weights] It does not train models itself; the discriminative/annotation models it invokes are mostly third-party open-source weights: Qwen2.5-VL / Qwen3-VL (captioning), Cosmos-Embed1 and InternVideo2 (video embedding; InternVideo2 was removed in 26.02), a CLIP-based aesthetic model (aesthetic scoring), TransNetV2 (shot segmentation), the NVIDIA NeMo ASR series (audio transcription), Nemotron Nano 12B V2 VLM and Nemotron 3 Nano Omni (captioning backends added in 26.04/26.07, with BF16/FP8/NVFP4 precision variants).
[Not open-sourced] The training data itself (the framework itself does not ship with any dataset); NVIDIA's internal list of actual data sources used for Cosmos and full-volume statistics are not public.

### [OmniHuman Dataset + OHBench](../models/OmniHuman.md) ⚠️

Openness is on the higher side among person-centric datasets of the same category, but with the inherent limitation of "video body not distributed":
[Code] Open-sourced. The GitHub repository github.com/julia-cherry/OmniHuman provides the OHBench evaluation toolkit, covering evaluation scripts, configuration files, and Python dependency specifications for metrics such as imaging quality, dynamic degree, identity consistency, audio-video alignment, audio quality, speech quality, and two-person interaction.
[Data (annotation)] Open-sourced. The Hugging Face dataset julia527/omnihuman, totaling about 62.7 GB of sharded tar archives, contains three types of assets: sample_json (per-sample audio-visual annotations: captions, subjects, speech, audio), metadata (JSONL index files for scanning and loading samples), and tracking_npz (frame-by-frame SMPL/MANO body and hand tracking data).
[Data (video body)] Not distributed. The original videos are not included in the release; each sample only provides the YouTube source URL plus precise clip start/end times (clip_start_sec / clip_end_sec), requiring users to download and locate the clips themselves — a common approach to avoid copyright risk (the same route as HD-VILA and Panda-70M), but this also means the dataset continually erodes as source videos are taken down (link rot), and the actual data obtained by different users will gradually diverge.
[Evaluation assets and weights] The Hugging Face repository julia527/omnihuman_benchmark provides OHBench's model checkpoints and evaluation assets, mostly in .pt / .onnx format (i.e., packaged inference weights of third-party discriminative models such as MUSIQ, RAFT, ArcFace, SyncNet, DNSMOS).
[Pipeline] Publicly disclosed at the methodology level (the order of the four-level filtering, the tool name used at each level, the algorithmic chain of the tracking and attribution modules, and the structure of the two-stage caption are all described), but threshold values are missing on a large scale (see pipeline_overview and funnel_retention_rate for details), and the cleaning scripts themselves have not been released. Reproducibility sits between "knowing what was done" and "being able to replicate it."
[License] The arXiv paper uses a perpetual non-exclusive license; neither the Hugging Face dataset page nor the GitHub repository has an explicit license statement or terms of use — the downstream usage scope of the data (whether commercial use is allowed, whether it is restricted to research use) is unclear. [Uncertain]

### [Open-Sora Series](../models/Open-Sora.md) ⚠️

Both belong to the highest-openness tier in the video-generation field, but neither is a case of "the full dataset is open."
[Open-Sora (HPC-AI Tech)] Weights: open-sourced (Hugging Face hpcai-tech, Apache 2.0); code: open-sourced (inference + training + distributed optimization); data-processing pipeline: fully open-sourced — this is the project's most valuable reference point — under the v1.1/v1.2 branches, the tools/ directory contains scene_cut (PySceneDetect shot detection + splitting), scoring (four scorers: aesthetic / optical_flow / ocr / matching), caption (PLLaVA, LLaVA, LLaMA3 annotation), and datasets (datautil filtering and cleaning), with docs/data_processing.md tying together the entire pipeline; every step is given a directly runnable torchrun command and threshold example; the training data itself: not open-sourced (only the names of the source datasets and the filtering thresholds are published, without releasing the filtered meta file). Note: the tools/ data-processing directory has been removed from the 2.0-version main branch (tools/datasets, tools/scoring, and docs/data_processing.md return 404 on the main branch); the complete data-processing code needs to be obtained from historical tags such as v1.2.0; the new pipeline described in the 2.0 technical report (PaddleOCR, VMAF, Laplacian, etc.) has no corresponding open-source implementation seen on the main branch — here, the degree of openness has actually **regressed**.
[Open-Sora Plan (Peking University)] Weights: open-sourced; code: open-sourced (training + inference + WFVAE); data: partially open-sourced — each version has released "Data and Annotations" annotation data and a prompt_refiner dataset on Hugging Face (LanguageBind/Open-Sora-Plan-v1.1.0 / v1.2.0 / v1.3.0); data-processing pipeline: the paper and Report documents give complete filtering steps, tools, thresholds, and per-stage retention rates (a rare quantitative disclosure across the whole industry), but no independently packaged curation code repository is seen, so reproduction requires assembling it oneself according to the documentation. [Uncertain]

### [Ovi](../models/Ovi.md)

A typical example of the "weights + inference code open, data and data-pipeline code closed" pattern.
[Open-sourced] (1) Model weights: an 11B (the Hugging Face page labels it as about 12B parameters, BF16) complete checkpoint, including three versions 720x720_5s, 960x960_5s, 960x960_10s, hosted on Hugging Face at chetwinlow1/Ovi; (2) inference code: text / text+image input, a Gradio App, multi-GPU (including sequence parallelism) inference, and weight-download scripts; (3) community-contributed fp8 (@rkfg) and qint8 (@gluttony-10) quantized weights, runnable with 24GB of VRAM; (4) the paper fully discloses the methodology and key thresholds of the data-processing pipeline (more open than most models of the same category on this point).
[Not open-sourced] The training scripts ("Training scripts" is still unchecked in the README's Todo List), the training data itself (the internal audio-video corpus and internal audio corpus), the data-processing pipeline code, the identity of the MLLM annotation model used, the quantitative retention rates at each filtering level, and the RL post-training details.
[License] Apache 2.0 (relatively permissive, commercially usable). Dependent components: the video-branch weights come from Wan2.2-TI2V-5B, the T5 text encoder and video VAE decoder come from Wan, and the audio VAE comes from MMAudio (all open-source models).

### [Script-a-Video](../models/Script-a-Video.md)

Openness is extremely low, belonging to the "paper disclosure only, zero code, zero weights, zero data" type:
[Weights] Not open-sourced. Neither Qwen3-Omni-MTSS-FT nor the generative model adapted from LTX-2 has been released.
[Code] Not open-sourced. The full paper text has no GitHub link, no project homepage URL, and no open-source commitment statement.
[Data] Not open-sourced. The 500K MTSS annotation dataset is explicitly described as an "internal dataset"; on the generation side, the four sets of data — 400K identity-centric / 250K multi-shot / 870K cinematic pairs / 60K cinematic alignment pairs — are likewise internal data; the 125+100 internal evaluation sets have also not been released.
[Pipeline] Methodology-level disclosure of the MTSS schema's field definitions is very thorough (the field names and semantics of the four streams — Reference/Shot/Event/Global — are all explained one by one in Section 3 of the body text, accompanied by a complete script example in Figure 3), with reproducibility mainly manifesting at the level of "the schema can be reimplemented by others"; but the data-cleaning process, the original text of the Gemini-2.5-Pro annotation prompt, and the filtering thresholds are all undisclosed.
[License] arXiv.org perpetual non-exclusive license (paper text only), no model/code license.
[Overall assessment] Its value lies in the paradigm and schema design itself being directly reusable, rather than in any directly usable asset.

### [Seedance 2.0 and Seedance 1.5 pro](../models/Seedance_20_Seedance_15_pro.md)

Weights: not open-sourced; code: not open-sourced; training data: not public; data-processing pipeline: only the Seedance 1.0 report gives a fairly complete textual description (no code, no thresholds, no open-source tools), the 1.5 pro report gives only a bullet-point-style paragraph, and the 2.0 report discloses no data section at all. Externally only API/product access is offered (Volcano Ark, Doubao, Jimeng, and third-party hosting such as Replicate).

### [SkyReels Series](../models/SkyReels.md) ⚠️

Displays a layered strategy of "early versions fully open-source, the latest flagship closed-source and productized."
[SkyReels-V1/V2/V3: weights + code open-sourced] V2 releases the full 1.3B/5B/14B series of weights on Hugging Face and ModelScope (DF long video, T2V, I2V, 540P/720P), together with open-source inference code and the SkyCaptioner-V1 annotation-model weights; V3 (R2V/V2V/Talking Avatar, 14B) was open-sourced in January 2026. The SkyReels series of open-source models have accumulated nearly 300,000 downloads on Hugging Face and over 10,000 GitHub stars.
[SkyReels-V4: no weight release found as of July 2026] The paper gives no code/weight release commitment and includes no GitHub link; it is offered externally as the skyreels.ai product (limited preview + free quota) and an open API, covering text-to-video, image-to-video, multimodal reference generation, video editing/restoration, and joint audio-video generation. Some Chinese-language media report that "V1–V4 are all open-sourced," but no SkyReels-V4 weight repository was found on Hugging Face/GitHub; it should be understood as "weights not yet open-sourced." [Uncertain]
[Data side] The training data itself, the data-cleaning pipeline code, and the internal quality-inspection models of all three versions are not open-sourced; only the model weights of the SkyCaptioner-V1 annotation step are open-sourced, which is the most valuable openness item on the data side for this series.

### [Sora 2](../models/Sora_2.md)

Fully closed. Weights not open-sourced, code not open-sourced, training data not open-sourced, data-processing pipeline not open-sourced. The only public material is the "Sora 2 System Card" (7 pages total) from September 30, 2025, in which the content about data amounts to only one paragraph (about 5 sentences) in Section 2 "Model Data & Data Filtering." There is no technical report, no paper, no architecture details, and no data-statistics figures of any kind. By comparison, the prior generation Sora 1 at least had a technical blog post, "Video generation models as world simulators," disclosing methodology such as spacetime patches, native-resolution training, and re-captioning. It was once commercially offered via API (sora-2, sora-2-pro); as of September 2026 the API has also been discontinued.

### [SpeakerVid-5M](../models/SpeakerVid-5M.md) ⚠️

A typical academic-dataset openness pattern of "metadata + annotations fully open, raw video not hosted":
[Data (annotations and index)] Open-sourced. Hugging Face dorni/SpeakerVid-5M-Dataset provides all_data_list.json (YouTube video IDs + names of clips after splitting, with the clip name serving as the unique key for locating annotations) and SFT_set.json (a high-quality subset list), plus five categories of annotation folders: merge_anno (clip-level metadata: timestamps, spatial bboxes, clear/DOVER quality scores), asr (Whisper transcriptions and confidence scores), l_score (face/hand clarity, i.e., blur scores), anno (Qwen-VL-generated MLLM structured captions), and dwpose (the actual skeleton sequences were not uploaded due to excessive size; only the computation code is provided).
[Data (raw video)] Not hosted. Users must download it themselves via yt-dlp based on the YouTube video ID, with the risk of reproducibility decay due to link rot.
[Data-processing pipeline code] Open-sourced — one of the most valuable parts of this entry. GitHub Dorniwang/SpeakerVid-5M-Code releases the complete six-stage cleaning-process code: base annotation (joint audio-video sync extraction + single-person detection), DWpose skeleton annotation, ASR annotation, blur-score computation, luminance computation, and scene detection + speaker diarization (partly precomputed, optional).
[Baseline model weights] Neither the paper nor the code repository gives a clear statement about weight release. [Uncertain]
[License] Explicitly restricted to "non-commercial, scientific research, and educational purposes only," explicitly prohibiting commercial use; content is sourced from the public internet, copyright belongs to the original creators, and a takedown policy is provided for rights holders to request removal. No standard SPDX license such as Apache/CC is used.

### [Step-Video-T2V](../models/Step-Video-T2V.md)

A pattern of "weights + code + evaluation benchmark open, data and data-pipeline code not open," but with a relatively high degree of openness among contemporaneous domestic Chinese models:
[Weights] Open-sourced. Both Step-Video-T2V (30B) and the distilled acceleration version Step-Video-T2V-Turbo are released on GitHub (stepfun-ai/Step-Video-T2V), Hugging Face (stepfun-ai/stepvideo-t2v, stepvideo-t2v-turbo), and ModelScope; the in-house Video-VAE (16×16 spatial, 8× temporal compression) and the bilingual text encoder are open-sourced together.
[Code] Open-sourced inference code (including multi-GPU parallel inference, xDiT/ComfyUI integration); training code not open.
[License] MIT License (more permissive than the Tencent Hunyuan Community License, Tongyi Wanxiang's custom license, etc. — a major highlight of its openness).
[Evaluation benchmark] Open-sourced Step-Video-T2V-Eval, containing 128 prompts and generated result videos from multiple open- and closed-source engines, directly reproducible for comparison.
[Data] Not open-sourced. Both the 2B video-text pairs and the 3.8B image-text pairs are internal data; no subset or list of any kind has been released.
[Pipeline] Methodology disclosure is fairly complete (a six-stage pipeline, with the specific open-source tools and model names used at almost every stage named, such as PySceneDetect AdaptiveDetector, the LAION aesthetic predictor, the LAION NSFW detector, the EfficientNet watermark classifier, PaddleOCR, Laplacian variance, FFmpeg cropdetect, Farneback optical flow, and the in-house VideoCLIP), but the specific filtering threshold values, the in-house caption VLM and VideoCLIP weights, and the pipeline code are all not public, so it cannot be directly reproduced.

### [UniTalking](../models/UniTalking.md) ⚠️

The paper positions itself as an "open and reproducible framework," with "closed-source models hindering academic progress" as its core motivation, but as of July 2026 its actual degree of openness is questionable:
[Weights] No weight release found on Hugging Face or any other platform. [Uncertain]
[Code] No GitHub repository found. Neither the paper body nor the abstract page gives a code link or a project homepage URL — a gap relative to its "open-source" narrative. [Uncertain]
[Data] Not open-sourced. Of the 2.3 million aligned samples, the OpenHumanVid portion can be independently obtained by third parties under its original license (requiring submission of user information for review and authorization to download), while the internally collected Huawei data and internal TTS data are entirely undisclosed.
[Pipeline] Methodology-level disclosure gives only the structure of the three-level filtering and the names of the tools used (PANNs, SentenceASD, LightASD, LipSync), without any threshold values, per-stage retention rates, original prompt text, or cleaning scripts — reproducibility is markedly lower than comparable work (compare UniVerse-1, which at least discloses the full set of thresholds: 1080p / bitrate ratio 600 / DOVER 0.6 / SyncNet 2.0).
[License] arXiv uses a perpetual non-exclusive license; the model and code license are not stated. [Uncertain]

### [UniVerse-1](../models/UniVerse-1.md)

Openness is medium-to-upper, using the Apache-2.0 license.
[Weights] Open-sourced. Hugging Face releases dorni/UniVerse-1-Base (7B parameters, F32/safetensors, bfloat16 inference), only this one variant, with no multi-resolution version.
[Code] Partially open-sourced. GitHub Dorniwang/UniVerse-1-code releases inference code (based on diffusers, Python≥3.10, PyTorch≥2.5.0-cu121); the training code and the server-side implementation of the online annotation pipeline are not open-sourced.
[Evaluation benchmark] Open-sourced. The Verse-Bench dataset and evaluation-metric tools are both released on Hugging Face.
[Data-processing pipeline] Methodology-level per-stage threshold disclosure is given in Section 3 of the paper (specific values are given for resolution, bitrate ratio, DOVER, clip duration, and SyncNet threshold), giving reasonable reproducibility; but no cleaning scripts or the original text of annotation prompts have been released, and the disclosure granularity is clearly lower than MOVA's (no per-stage retention-rate table, no full prompt text, no pseudocode).
[Data] Training data not open-sourced. The public-dataset portion (VGGSound, AudioSet) can be obtained independently; the self-collected YouTube/Pexels/film-clip portion is not public, and the filtered subset list has also not been released.

### [Unison](../models/Unison.md)

As of July 2026, it is in a fully unopened state, with only the paper itself open (CC BY 4.0).
[Weights] Not open-sourced. Original text of the paper: "The code and models will be made publicly available upon acceptance" — a conditional commitment premised on conference acceptance, not yet fulfilled.
[Code] Not open-sourced. No GitHub repository found, no project homepage or demo page found.
[Data] Not open-sourced, and in fact there is no "dataset that could be open-sourced" — the training corpus is a re-filtered and re-processed version of existing open-source datasets such as OpenHumanVid, HDTF, VFHQ, CelebV-Text, and VGGSound; the filtered sample list (about 2 million entries) has not been released. The audio side also includes undisclosed "internal speech data."
[Pipeline] Disclosure level is extremely low — the biggest information gap of this entry. The paper gives only one sentence about the "automated processing pipeline," with the only stage described in any detail being the lip-filtering operator (face detection + in-box SyncNet verification), and no threshold values, per-stage retention rates, or pseudocode are given at all. Compared with MOVA (per-stage thresholds + retention-rate table) and UniVerse-1 (full thresholds disclosed for a six-level funnel), Unison clearly lags a full tier behind in data-processing reproducibility — a typical characteristic of a "methodological innovation paper" rather than a "data engineering paper."
[Evaluation set] Neither the 1,000 test samples nor the Gemini-generated ground-truth annotations have been released; the raw data of the user study (40 samples × 25 people) has also not been released.
[Reused open-source assets] All base models and tool chains are open-source: Wan2.2-5B, MMAudio, Zipformer, Mel-RoFormer, SyncNet, Whisper-large-v3, Synchformer, ImageBind, CLAP, LAION-Aesthetic V2.5, DINOv3, Audiobox-Aesthetics.

### [Veo 3 / Veo 3.1](../models/Veo_3_Veo_31.md)

Fully closed-source. Weights not open-sourced, code not open-sourced, training data not public, data-processing pipeline not public, data mixture and scale not disclosed. Only a roughly 7-page technical report (Veo-3-Tech-Report.pdf) and a roughly 6-page Model Card are public, in which the data-related body text is fewer than 200 words, with the vast majority of the space devoted to responsibility and safety evaluation. Inference access is only offered through paid API/product forms (variants such as Veo 3, Veo 3 Fast, Veo 3.1, Veo 3.1 Fast, Veo 3.1 Lite).

### [Vidu S1](../models/Vidu_S1.md)

Closed-source. The technical report only publicly discusses the architecture and training-framework thinking; the model weights, inference/training code, data, and data-processing pipeline are not open-sourced; the in-house acceleration and serving components TurboDiffusion and TurboServe are also not open-sourced. The self-built benchmark Vidu-StreamBench has not been publicly released. Only an interactive online demo is provided via the official site https://vidu.com/vidu-stream. The paper is released under the CC-BY 4.0 license.

### [Wan 2.5 / 2.6 / 2.7](../models/Wan.md)

Displays a fault-line pattern of "previous generation fully open, current generation fully closed," with the data side closed from beginning to end.
[Wan 2.5 / 2.6 / 2.7] Fully closed-source: no weights, no code, no technical report, no paper. Service is provided only through Alibaba Cloud Bailian DashScope API (multiple regions: Beijing/Singapore/Virginia) and the Wanxiang official site/Tongyi App, billed by resolution tier and duration in seconds. As of July 2026, neither the Hugging Face Wan-AI organization nor the GitHub Wan-Video organization contains any Wan2.5/2.6/2.7 repository or weights — GitHub only has 5 repositories: Wan2.1, Wan2.2, Wan-Dancer, Wan-skills, and a diffusers branch.
[Wan 2.1] Weights + inference code open-sourced (Apache 2.0), and a 60-page technical report with a detailed data chapter is published; the training data, cleaning-pipeline code, in-house caption model, and various expert classifiers are not open-sourced.
[Wan 2.2] Weights + inference/fine-tuning code open-sourced (Apache 2.0); the README discloses the data-increment ratio relative to 2.1 and the "cinematic aesthetic label system," but there is no independent technical report, and the data details are far fewer than for 2.1.
[Wan2.2-S2V / Animate / Dancer] Weights + code open-sourced; the papers contain data-processing chapters (S2V's Chapter 2 is the only document in the Wan series that explicitly writes out an audio-visual sync data-filtering method).
[Conclusion] To study the data methodology of Wan 2.5+, one can only rely on the Wan 2.1 report as the backbone, with the Wan 2.2 README and the S2V/Dancer papers as incremental corroborating evidence, and infer the rest from API behavior.

### [Audio-Video Generation Evaluation Benchmark Collection](../models/av_benchmarks.md) ⚠️

[VABench] Paper CC BY 4.0; code repository https://github.com/tanABCC/VABench; no explicit dataset license statement. Prompts, VQA/AQA question-answer pairs, and evaluation scripts are the main open-source outputs; generated videos rely on each vendor's own API for reproduction.
[AVBench] Highest degree of openness: GitHub https://github.com/YaJialiang/AVBench; evaluator weights released on Hugging Face (iiiiii123/AVBench_model), also hosting a Hugging Face Leaderboard (spaces/iiiiii123/AVBenchLB). Data, code, and models are all released; the specific license is not labeled [uncertain].
[AV-SyncBench] The dataset is already live on ModelScope (coming245/AVSyncBench) and Hugging Face (coming245/AV-SyncBench); the code repository is https://github.com/fgt7t6g/AV-SyncBench (as of the survey, the README labels the evaluation code as "coming soon"); the paper uses the arXiv perpetual non-exclusive license.
[PhyAVBench] Project pages at https://imxtx.github.io/PhyAVBench/ and https://phyavbench.pages.dev/; publicly releases prompts, self-recorded ground-truth videos, and generated samples from various models, and commits to zero overlap with the training set; arXiv license for the paper.
[Omni-Judge] Only a project page at liangsusan-git.github.io/project/omni_judge/; the paper does not clearly state whether the code/data is open-sourced [uncertain].

### [Video Caption Model Ecosystem](../models/caption_models.md) ⚠️

Openness shows a clear three-tier split of "fully open in academia / semi-open at large vendors / closed on the generation side":
[Fully open (weights + code + data)] ShareGPT4Video: 40K GPT-4V dense captions and 4.8M ShareCaptioner-Video annotations are all public; the paper is CC BY 4.0; ShareCaptioner-Video weights are on HF (Lin-Chen/ShareCaptioner-Video, based on InternLM-XComposer2-4KHD). Tarsier series: bytedance/tarsier code + omni-research/Tarsier2-Recap-7b weights + Tarsier2-Recap-585K data all open, currently the most reused open-source captioner among downstream generative models. CogVLM2-Caption: weights open-sourced (zai-org/cogvlm2-llama3-caption), the only reproducible link in CogVideoX's data pipeline. SkyCaptioner-V1: weights + training details (Qwen2.5-VL-7B base, 32×A800, 2 million concept-balanced data points) fully disclosed. AVoCaDO: Apache-2.0 weights + code + project page, but AVoCaDO-SFT-107K has not been separately released. video-SALMONN 2: Apache-2.0, code/weights/testset all open. AuroraCap + the VDC benchmark are open-sourced.
[Semi-open] AVSCap: code + AVSCapBench are already open; the AVSCap-130K training-set README explicitly states "will release as soon as possible" but as of the survey has not been released; the availability of the AVSCap-7B weights is uncertain (the GitHub repository shows both an HF link and a "pending release" statement at the same time) [uncertain]. Qwen3-Omni-Captioner (the audio version) is open-sourced, but Qwen3.5-Omni's audio-video captioning capability is only available via API and has not been open-sourced as an independent annotation tool.
[Closed-source] Almost no in-house captioner from generation-side teams is open-sourced: OpenAI Sora's highly descriptive captioner, Meta's fine-tuned LLaMa3-Video captioning version, Google's Gemini-annotation variant used for Veo 3, Lightricks LTX-2's audio-video captioner, Tencent Hunyuan's three in-house caption models, StepFun Step-Video's in-house VLM, ByteDance Seedance 1.5/2.0's captioning system, and Kuaishou Kling 3.0 Omni's video-description enhancement module — all disclose only "what was used," not the parameter count, base model, or weights.
[A counterintuitive structural fact] The degree of openness of annotators is significantly higher than the degree of openness of generation models' data pipelines: most closed-source generation models' technical reports are willing to name-drop using Tarsier2 / Qwen3-Omni / LLaVA-Video, because the annotator is viewed as a "tool" rather than a "moat"; what is truly viewed as the moat is the original text of the annotation prompt, the field schema, and the threshold table — this part is almost entirely undisclosed across the industry.

### [Geometric/Structured Annotation Dataset Collection](../models/geometric_datasets.md) ⚠️

All are academic open-source datasets, with weights not being the primary product. SceneScribe-1M: the paper explicitly states it is open-source; project page https://wangyunnan.github.io/SceneScribe-1M; the data/annotations are publicly released, with the specific license subject to the official repository [uncertain]. SpatialVID: the highest degree of openness — Hugging Face releases two versions, SpatialVID (2.71 million clips) and SpatialVID-HQ (370,000 clips), totaling about 3.53TB, under the CC-BY-NC-SA 4.0 license (non-commercial use only), with the annotation-pipeline code open-sourced simultaneously. WildWorld: GitHub repository https://github.com/ShandaAI/WildWorld; the paper and benchmark are open, and the data is sourced from the commercial game "Monster Hunter Wilds," with redistribution terms not clearly stated [uncertain]. Action100M: GitHub repository https://github.com/facebookresearch/Action100M; the annotation volume is about 205GB (annotations only, video depends on the original HowTo100M source); the annotations are open-sourced.

### [Video Generation Post-Training Data Strategy](../models/post_training_data.md) ⚠️

Anchor paper (2604.25427): the paper is open (CC BY 4.0), but the code and weights are not open-sourced; the base model is "an internal video generation model"; neither the SFT dataset nor the RLHF prompt set is open-sourced, with only Figure 2 using the public Wan-2.1 for RLHF-effect visualization. Overall it belongs to "methodology open, data and model closed."
Cross-sectional openness gradient of post-training data (from high to low):
① Preference data fully open-sourced: HPDv3 (1.08 million text-image pairs, 1.17 million paired comparison annotations) and the VideoReward preference set (16,000 prompts / 108,000 videos / 182,000 annotation triplets, including VideoGen-RewardBench) — these two are currently the most important public preference assets for video/image generation post-training;
② Preference data "being prepared for release": JavisDiT++'s roughly 25,000 audio-video preference pairs (not yet public as of the survey) [uncertain];
③ Methodology and process public, data not public: SkyReels-V2 (30,000 manually curated sample pairs + three stages of 20,000 each, about 60,000 DPO data total), Step-Video-T2V (the Video-DPO process fully public, the quantity not public), HunyuanVideo 1.5 (the construction of the RLHF prompt set and the GSB annotation protocol public, the scale not public), LongCat-Video (the three-reward GRPO configuration public, the SFT set scale and RM annotation volume not public), Cosmos-Predict 2.5 (the five-domain SFT scale disclosed item by item, the GRPO configuration public; the data is not open-sourced but the post-RL EMA weights are released);
④ Only a single sentence or completely blank: Sora 2, Veo 3/3.1, LTX-2, Kling 3.0 Omni (only states DPO was used), Seedance 1.5 pro (only states RLHF + a multi-dimensional RM was used).
The reward-model side has significantly higher openness than the generation-model side: HPSv3, VideoAlign/VideoReward, VisionReward, and Unified Reward Model all have open-sourced weights, forming the de facto standard combination of "open-source RM + closed-source generator."

### [Combined Survey of Mainstream Video Pretraining Datasets: Panda-70M, InternVid, Koala…](../models/pretraining_datasets.md) ⚠️

The key difference lies in "whether the video body itself is hosted" — this is the watershed for reproduction cost.
[Metadata only (URL + timestamp + caption), video needs to be crawled independently] Panda-70M (CSV, Google Drive, including matching_score/desirability/shot_boundary columns); InternVid (jsonlines, 40.9GB on HF, gated, requiring name/institution/email to be filled in); Koala-36M (10 CSV shards totaling 48.9GB, no dataset card or license tag on HF); MiraData (4 CSVs: 330K/93K/42K/9K); LVD-2M (3 CSVs totaling about 5.08GB, hosted on S3).
[Video body genuinely hosted] OpenVid-1M (HF nkp37/OpenVid-1M, about 12.4TB, 74+ zip shards, those over 50GB needing to be reassembled with cat; OpenVidHD-0.4M is separately about 4.5TB); UltraVideo (HF APRIL-AIGC/UltraVideo, clips_short_{1..36}.zip in native resolution + 1920/960 downsampled versions). These two are the only ones among the seven where pixels can be obtained directly.
[Degree of openness of cleaning code] Koala-36M open-sources a complete implementation of scene-transition detection (including fitted SVM coefficients) and VTSS inference code, but explicitly states that what is released is the "base version" — the actual tested configuration is a fragments-only FAST-VQA architecture (PLCC 0.8684), not the complete model with a ConvNeXt static branch + WCGB described in the paper (PLCC 0.8974), i.e., **the released scorer is not the one that was used to build the data**; MiraData open-sources the GPT-4V annotation prompt and the MiraBench evaluation code, but **the paper states the filtering thresholds are in the supplementary material, while the supplementary material does not actually exist**; LVD-2M does not open-source any filtering code at all, only giving the PLLaVA prompt in a paper figure; UltraVideo only open-sources inference code, with training and cleaning pipeline both unreleased (GitHub issue #8 asking about this went unanswered); Panda-70M open-sources the splitting code (cutscene_detect/event_stitching/video_splitting) and the student caption-model weights, but does not open-source the teacher-model inference script or the UMT selector weights; InternVid open-sources ViCLIP, with the cleaning pipeline only textually described; OpenVid-1M open-sources MVDiT training/inference code and checkpoints at 256~1024 resolution.
[License] Panda-70M: Snap non-commercial research license + inherited HD-VILA-100M license; InternVid: CC-BY-NC-SA-4.0; Koala-36M: Kuaishou non-commercial research license + inherited HD-VILA-100M license; MiraData: code GPL-3.0, data terms self-contradictory (the README's early section prohibits commercial use, while the end states it "supports commercial use"); OpenVid-1M: CC-BY-4.0 but states it is for research and non-commercial use only, and must comply with the respective licenses of upstream Panda/ChronoMagic/CelebvHQ/Open-Sora-Plan; UltraVideo: a custom license-april-lab.txt, non-commercial, prohibiting redistribution of the original videos, and requiring compliance with YouTube ToS and GDPR/CCPA; LVD-2M: no independent LICENSE file, stated to be consistent with the HD-VILA license. All seven are non-commercial research licenses (OpenVid's nominally permissive CC-BY-4.0 is narrowed by its own statement), and none of them actually holds copyright to the YouTube material.
## Whether simultaneous audio-video generation is supported, and the implementation approach (native joint / cascade / MoE fusion)

`av_generation` · Detail level: brief

### [Allegro](../models/Allegro.md)

Not supported. Allegro is a purely visual text-to-video/image-to-video model that outputs silent video (88 frames, 720×1280, 15 FPS, approximately 6 seconds; can be frame-interpolated to 30 FPS with EMA-VFI). The entire data pipeline does not involve extraction, filtering, or annotation of any audio track, nor does it have a cascaded V2A module or an audio expert branch. All audio-related dimensions are not applicable to this entry.

### [Apollo](../models/Apollo.md)

Supported, and it is native joint generation (a single-tower, single forward pass produces both audio and video latents simultaneously, not cascaded). Implementation approach:
- Single-Tower MultiModal Diffusion Transformer (MMDiT) architecture, containing 32 joint diffusion layers, where audio and video share the same set of DiT block parameters, rather than a dual-tower + cross-attention design.
- The core mechanism is Omni-Full Attention: audio tokens and video tokens perform fully-connected self-attention within the same attention window, achieving tight audio-visual alignment and good scalability.
- Position encoding uses MixD-RoPE (Mixed Dimension Rotary Position Embedding), which uniformly handles the video's 3 dimensions (t, h, w) and the audio's 1-dimensional temporal axis index.
- The training objective is flow matching (conditional denoising).
- The model has 26B total parameters, with a flow-matching FFN dimension of 4096.
- Inputs consist of four streams: video, video-related text, audio-related text, and audio, each independently encoded before being fed into the MM-DiT.
【Comparative evidence against the dual-tower route】Table 2 of the paper directly compares "Dual Tower (standard cross-attention)" against "Single Tower (Omni-Full Attention)," with the conclusion supporting the single-tower full-attention scheme — this marks a clear architectural divide from dual-tower + bridge routes such as MOVA/HunyuanVideo-Foley.
【Unified multi-task support】Through random modality masking, the same model uniformly supports five task types — T2A, T2V, T2AV, I2V, and I2AV — within a single model, so the same weights can perform both joint generation and single-modality generation.

### [CineDance / CineDance-1M](../models/CineDance.md)

Supports simultaneous audio-video generation, belonging to the "native joint generation" route (native joint audio-video generation), rather than a cascaded scheme that generates video first and adds sound afterward.
【Data-side positioning】One of the core selling points of the dataset is "preserving the native audio track"; the paper lists "missing acoustic modality" as one of four major flaws of existing datasets. All sequences carry original synchronized audio, and annotations cover both the visual and auditory tracks.
【Model-side implementation】The CineDance model is based on LTX-2.3, with an architecture of 13B video DiT + 3B audio branch + 3B cross-modal cross-attention module. Video and audio are coupled through cross-modal attention within the same diffusion process, making it a native joint architecture of dual-tower + cross-attention fusion — not MoE fusion, and not cascaded post-hoc dubbing.
【Task definition】The paper frames the task as T2AV (Text-to-Audio-Video), i.e., generating a multi-shot long video with synchronized audio track in one pass from a text prompt.

### [CogVideoX](../models/CogVideoX.md)

The CogVideoX model itself does not support simultaneous audio-video generation; it is a purely visual text-to-video/image-to-video model, and its training data does not use audio tracks.
At the product layer ("New Qingying," November 2024), video "with built-in sound effects" is achieved via cascading: CogVideoX (version 1.5) first generates a 10-second, 4K/60fps silent video, and then a separate sound-effect model, CogSound, performs V2A (video-to-audio) dubbing conditioned on the video. CogSound's disclosed technical highlights are: extracting semantics/emotion via GLM-4V's video understanding capability → generating audio via a Latent Diffusion model → establishing correspondence between frame-level video features and audio features using "Block-wise Temporal Alignment Cross-attention" → adding Rotary Position Embedding (RoPE) to improve long-sequence temporal consistency. It can generate complex sound effects and rhythmic elements such as explosions, flowing water, musical instruments, animal calls, and vehicles.
The implementation is therefore "cascaded," rather than native joint or MoE fusion; and the data pipelines of the two models are entirely independent — the video-side paper does not touch on the audio dimension at all.

### [Cosmos-Predict2.5](../models/Cosmos-Predict25.md)

Not supported. Cosmos-Predict2.5 is a purely visual video world model whose input is text/image/video conditioning, and whose output is video without an audio track (93 frames, 16fps, about 5.8 seconds). Across the full 44-page text, no audio/speech/sound-related data processing or generation description appears — audio is neither a condition nor an output. Its "multimodality" is reflected in the text encoder being switched to Cosmos-Reason1 (a decoder-only VLM dedicated to Physical AI) and the action-conditioned input of the robotics variant, not in joint audio-video generation. Consequently, all audio-video-related dimensions in this survey (audio category mixture, joint captioning, dialogue transcription, lip-sync/event alignment, audio quality filtering, audio type handling) are not applicable to this work.

### [Data-Juicer 2.0](../models/Data-Juicer.md)

Does not support simultaneous audio-video generation — Data-Juicer itself is not a generative model and does not produce any audio or video content. However, on the data side it provides operator capabilities that support joint audio-video tasks, which has a substantive connection to this survey's AV mainline:
【Audio-side operators】audio_duration_filter (duration filtering), audio_nmf_snr_filter (NMF-based signal-to-noise-ratio filtering), audio_size_filter (file size filtering), audio_add_gaussian_noise_mapper (noise-augmentation), audio_ffmpeg_wrapped_mapper (FFmpeg audio filter wrapper).
【Audio-video cross-modal operators】video_audio_ASR_mapper (speech recognition/annotation from the audio track), video_tagging_from_audio_mapper (generating video tags from the audio track based on Audio Spectrogram Transformer), video_captioning_from_audio_mapper (generating video captions from the audio track based on Qwen-Audio), video_audio_detect_age_gender_mapper (detecting speaker age and gender from the audio track based on wav2vec2), video_audio_speech_emotion_mapper (speech emotion recognition), video_active_speaker_detect_mapper (active speaker detection combining visual face trajectories and audio signals — the operator in DJ closest to "audio-video synchronization determination").
【Positioning assessment】DJ's audio-video capabilities lean toward "understanding and annotation" rather than "synchronization quality gatekeeping": it has active speaker detection, but no SyncNet/Synchformer-style operators for scoring sync offset and confidence, nor operators for separating and controlling the mixture of the three audio-track types — speech/foley/music. Therefore, to reuse DJ for building an AV joint-generation training set, the synchronization-filtering and audio-track-classification steps still require custom-extended operators.
【Empirical case】Its official text-to-video showcase (topping VBench) is purely T2V, and the entire pipeline does not involve audio processing.

### [Foley-Omni](../models/Foley-Omni.md)

Not an "simultaneous audio-video generation" model, but rather "joint generation of multiple audio categories conditioned on video." The implementation is native joint generation rather than cascading: the three audio-track types — speech, sound effects, and music — are jointly produced in one pass within a single diffusion generation process sharing a latent space, rather than being generated separately by TTS/Foley/music models and then mixed. This is the core difference relative to the baselines (the cascaded pipelines MMAudio + CosyVoice 3 + AudioX, and MMAudio + LipVoicer + AudioX).
Architecturally it is a Diffusion Transformer (DiT). On the text side, a shared UM-T5 encoder encodes the three-field structured text; on the audio side, a frozen Mel VAE (carried over from MMAudio) + BigVGAN vocoder is used; on the video side there are two conditioning paths: CLIP features provide scene semantics, and Synchformer features provide temporal synchronization cues (the latter injected via an additive path Z_sync).
Empirical results support joint generation outperforming cascading: on V2ST-Bench, Foley-Omni's WER of 7.59 is better than MMAudio+CosyVoice 3+AudioX's 10.57 and MMAudio+LipVoicer+AudioX's 37.84; its DeSync of 0.16 is likewise better than 0.85/0.26; and its three subjective scores — A-MOS 3.92 / S-MOS 4.13 / T-MOS 4.14 — comprehensively lead the cascaded baselines (for reference, Ground Truth is 4.33/4.37/4.42).

### [Goku](../models/Goku.md)

Not supported. Goku is a purely visual image+video joint generation model (T2I / T2V / I2V); the paper does not touch on audio generation, audio-track modeling, speech, or sound effects anywhere in the text. Its "joint generation" refers to the joint modeling of two visual modalities — image and video — within the same rectified flow Transformer (images are treated as single-frame video, encoded via a 3D joint image-text VAE, and interact uniformly with video tokens under full attention), not audio-video joint generation. This entry is therefore not applicable on the audio-video dimension; its value lies primarily in its data-distribution-balancing methodology on the pure-video side.

### [Hailuo / MiniMax Video](../models/Hailuo.md)

Does not support simultaneous audio-video generation (as of Hailuo 2.3 / July 2026).
- The MiniMax open platform's video generation API documentation contains no audio/sound parameters at all; output is silent video.
- None of the 7 Hailuo/video-01 series models hosted on Replicate describe any capability to produce an audio track simultaneously with the video.
- MiniMax's audio capabilities are handled by entirely separate product lines: MiniMax Speech (speech synthesis, version 2.8) and MiniMax Music (music generation, version 3.0); on the Hailuo AI website, "Audio" is a standalone entry point parallel to video.
- From a product-experience perspective, Hailuo AI's Media Agent (launched alongside 2.3 in October 2025) can chain text → video → speech/music with one click, which is a cascade at the product-orchestration layer, not native joint generation or MoE fusion at the model layer.
This entry therefore serves as a counterexample/control group on the "joint audio-video generation" mainline of this survey: it represents a leading commercial model that remains in the pure-visual-generation paradigm even after peers such as Veo 3, Sora 2, Kling 3.0 Omni, and LTX-2 have already shifted to native AV joint generation. Accordingly, all AV-related fields in this survey have no substantive content for this object.

### [HunyuanVideo-Foley](../models/HunyuanVideo-Foley.md)

Does not support simultaneous audio-video generation — this is the key to understanding this work's positioning. HunyuanVideo-Foley is the "second half" of the cascade paradigm: the video already exists (it can be real footage, or a silent video generated by a model such as HunyuanVideo), and the model's task is to add sound to it. Strictly speaking, it therefore belongs to V2A / TV2A (Text-Video-to-Audio) unidirectional conditional generation, rather than native joint generation of the kind seen in UniVerse-1, MOVA, or Ovi.
【Division of labor versus joint-generation models】Joint generation models need to solve two problems simultaneously — "what the video looks like" and "what the sound is" — whereas this model solves only the latter, so it can devote its entire model capacity and data budget to fidelity and synchronization precision on the audio side. This also explains why its audio quality metrics (PQ 6.59, MOS-Q 4.14) are markedly higher than contemporaneous joint-generation models.
【Architectural implementation】A hybrid multimodal diffusion Transformer with approximately 3 billion parameters: the front section has 18 layers of MMDiT (multimodal dual-stream blocks, where audio and video tokens go through their own parameter branches but share a joint self-attention pass), and the back section has 36 layers of unimodal audio DiT (pure-audio single-stream blocks for refining audio detail). Hidden dimension is 1536, with 12 attention heads.
【Three conditioning paths, each injected differently】(1) Video semantics — a SigLIP-2 visual encoder extracts frame features, which are concatenated with the audio latent and undergo deep fusion via joint self-attention; (2) Text semantics — encoded by a CLAP text encoder and injected via cross-attention (deliberately routed differently from video; the paper states this is to resolve "modal competition," i.e., to prevent the dominant visual condition from swamping the text condition); (3) Frame-level synchronization signal — Synchformer extracts frame-level sync features, injected via an adaLN + gated modulation pathway that does not participate in attention computation.
【Interleaved RoPE】Audio tokens and visual tokens are interleaved along the time axis before Rotary Position Embedding is applied, so that audio and video tokens at the same moment are positionally adjacent in the encoding, reinforcing frame-level synchronization dependency. Ablation shows that removing it worsens DeSync from 0.78 to 0.79 and changes CLAP from 0.30, with the impact mainly showing up in temporal alignment.
【Audio representation】An in-house DAC-VAE at a 48 kHz sample rate, replacing DAC's original RVQ discrete quantization block with a continuous variational autoencoder using a Gaussian distribution + KL regularization, outputting a 128-dimensional continuous latent at a latent frame rate of 50 Hz.

### [HunyuanVideo](../models/HunyuanVideo.md)

Not supported. Both HunyuanVideo and HunyuanVideo 1.5 are purely visual video generation models, outputting video without an audio track. The 1.5 technical report contains no audio-generation-related content anywhere in the text, and the data side does not involve any audio-track processing either.
Tencent's audio capabilities are handled by separate models rather than joint generation: HunyuanVideo-Foley (August 2025, video-to-audio/Foley generation, based on a roughly 100,000-hour TV2A dataset), HunyuanVideo-Avatar (audio-driven digital avatars), and so on, which are "cascaded/bolt-on" forms — HunyuanVideo generates the picture first, and then a Foley model dubs it, rather than native joint denoising. This entry therefore does not constitute a reference sample on the joint audio-video generation dimension, and all audio-related fields in this survey (audio_category_distribution, av_sync_detection, sync_metric_and_threshold, temporal_vs_semantic_sync, audio_quality_filtering, audio_type_handling, joint_av_caption_schema, dialogue_transcription_attributes) are not applicable.

### [InstructAV2AV](../models/InstructAV2AV.md)

Supports joint audio-video generation, but the task form is "joint editing" rather than "joint generation from scratch," and the implementation is native joint rather than cascaded.
【Model-side architecture】Based on Ovi's (Low et al., 2025) symmetric twin-backbone diffusion transformer: the video tower and the audio tower run in parallel, achieving joint denoising through cross-tower interaction; the video side uses a spatial-temporal VAE for encoding/decoding, the audio side uses a 1D VAE for encoding/decoding, and the text side uses T5 to encode instructions. Audio and video are produced synchronously within the same diffusion process, naturally guaranteeing temporal alignment, rather than a cascade of "generate video first, then add sound."
【Three modifications tailored to the editing task】
  1. Source Concatenation (SC): concatenates the source audio-video latent with the noise latent along the channel dimension, anchoring the generation process to the source context; this is the core mechanism for keeping non-target regions unchanged. Ablation shows that removing it worsens FVD from 180.38 to 467.20 (a 1.6x degradation) with severe background degradation — the largest impact among the three designs.
  2. Source-Instruction Gated Attention (SIGA): soft gated attention between source information and instruction information, balancing the conflicting objectives of "following the instruction to make changes" versus "preserving the original content." Ablation shows that removing it raises FAD from 2.75 to 3.26, with audio hallucination and stuttering appearing.
  3. Two-Stage Training Strategy (TSTS): adapting each modality separately first, then jointly fine-tuning, used to smoothly transfer pretrained priors. Ablation shows that removing it raises FVD to 291.55 and FAD to 5.18, with visual distortion and inconsistency appearing.
【Loss weighting】Joint training uses modality-balancing weights λ_v = 0.85, λ_a = 0.15, with the visual-side weight far higher than the audio side (about 5.7:1), reflecting that the visual branch is harder to converge.
【Audio-video coupling on the data-engine side】Worth noting separately: the mask-guided video-editing model (based on Wan2.2-5B) used to construct the training data itself also incorporates an audio-video coupling design — injecting audio features into the video synthesis process via frame-wise cross-attention, to ensure that the synthesized target video is strictly temporally synchronized with the already-synthesized target audio. That is, audio-video alignment is already being enforced at the "synthetic data stage" itself, rather than via post-hoc filtering.

### [2026 Miscellaneous Joint Audio-Video Generation](../models/JAVG_2026_misc.md)

All seven items center on simultaneous audio-video generation, but their implementation paths fall into five categories, forming a cross-section of the 2026 JAVG technical landscape:
(1) 【Native joint + upfront semantic planning】Baton: before joint DiT denoising, a VA-Planner (a multimodal LLM built on Qwen3-8B, with dual semantic alignment towers) first generates a batch of "planned tokens" as a shared audio-video semantic blueprint, and then Relative Semantic RoPE (RS-RoPE) injects the blueprint into the denoising process. Its core argument is that "existing methods rely on coarse-grained embeddings from off-the-shelf text encoders, losing fine-grained semantics and lacking shared long-horizon planning." The planner does not predict discrete tokens but instead regresses continuous features (L2 regression to the penultimate layer of frozen SigLip2 video features and WavTokenizer audio features), on the grounds that "regressing continuous features preserves richer semantic structure."
(2) 【Native joint + customization LoRA】OmniCustom: on top of an existing joint audio-video generation backbone, two independent LoRA sets (a reference identity LoRA and an audio timbre LoRA) are each applied to the self-attention layers, enabling "given a reference image + a reference audio clip, generate synchronized audio-video that preserves that identity, mimics that timbre, and speaks the dialogue specified in the prompt." A contrastive learning objective is additionally introduced (the predicted flow with reference conditioning as positive, without reference conditioning as negative) running in parallel with flow matching.
(3) 【Streaming autoregressive + decoupled orchestration】StreamChar: decouples "long-horizon orchestration" from "short-window denoising" — an LLM orchestrator reads the full dialogue script and historical context to produce frame-aligned audio conditioning; a joint audio-video DiT performs bidirectional denoising within a local window, conditioned on a reference frame and motion frames. Each chunk outputs 33 frames @24fps, with a historical audio context cap of 15 seconds, paired with a progress-aware pointer (PAP, using ASR-timestamp ground-truth end indices for smooth L1 supervision) and a persistent visual anchor (sink chunk) to suppress long-range drift. Runs in real time on a single H100 and can continuously generate a 5-minute stream.
(4) 【Dual-tower native joint (major-lab backbone)】ALIVE: a unified audio-video synthesis model adapted from a pretrained T2V model, VideoDiT 12B + AudioDiT 2B, supporting Text-to-Video&Audio and Reference-to-Video&Audio (animation). Key architectural components are TA-CrossAttn (temporally-aligned cross-modal fusion) and UniTemp-RoPE (unified temporal RoPE for precise audio-visual alignment). Two stages: a 480p base and a 1080p refiner.
(5) 【Patching the dual-stream Transformer paradigm】CCL: does not change the paradigm, but specifically diagnoses and fixes three classes of flaws in dual-stream Transformers — model manifold drift caused by gating mechanisms, bias introduced by cross-modal attention in multimodal background regions, and inconsistency between multimodal CFG training and inference. Corresponding to four modules: TARP (temporally-aligned RoPE and partitioning), LCT + DCR (a stable unconditional anchor and dynamic routing of cross-modal information), and UCG (Unconditional Context Guidance, for inference consistency). Its biggest selling point is achieving SOTA with far less data and compute than comparable work.
(6) 【Align-then-Fuse native alignment】NAVA: explicitly argues against two existing designs — dual-tower designs that weaken fine-grained synchronization, and unified tri-modal designs that conflate semantic alignment with low-level alignment. Instead it uses Align-then-Fuse MMDiT: first establishing audio-video correspondence in a dedicated alignment space, then folding context (text, speaker embeddings) as external conditioning into shared denoising via cross-attention. It includes Timbre-in-Context Conditioning, which binds reference-timbre cues to corresponding speech spans, supporting multi-speaker reference-timbre control. 6.3B parameters, native stereo output, supporting T2AV / I2AV / T2A.
(7) 【Inference-time scaling (model unchanged)】ITS-JAVG: transplants Inference-Time Scaling from single-modality domains to joint audio-video generation, entirely training-free. Its core finding is that a single verifier inevitably leads to verifier hacking and asymmetric trade-offs among metrics, necessitating a combination of multiple verifiers; it proposes Adaptive Reward Weighting (ARW), treating reward aggregation as an online optimization problem. This can be seen as replicating, at inference time, the "joint multi-scorer filtering" idea from training-data pipelines.

### [Collection of Joint Audio-Video Generation Baselines](../models/JavisDiT_baselines.md)

All five support simultaneous (joint) audio-video generation, and all belong to "native joint" rather than cascaded, but their cross-modal interaction mechanisms differ, forming a clear line of technical evolution:
(1) MM-Diffusion (2022): a dual-branch sequential multimodal U-Net, using 2D+1D spatio-temporal convolution for video and dilated convolution for audio; the core is random-shift based attention (randomly shifted attention blocks) bridging the two sub-networks, reducing cross-modal attention complexity from O((F×H×W)×T) to O((S×H×W)×(S×T/F)). Diffusion happens in pixel space, not latent space. It can also zero-shot transfer to video-to-audio and audio-to-video conditional generation (via gradient guidance), without any additional training.
(2) AV-DiT (2024): centers on "parameter efficiency" — sharing a single DiT backbone pretrained only on image data (frozen), with only lightweight adapters inserted and trainable on the audio and video paths; the video branch adds trainable temporal attention inside the frozen DiT blocks to ensure temporal consistency, while the audio branch likewise relies on lightweight parameter adaptation plus a cross-modal feature interaction module. This belongs to the "frozen shared backbone + dual-modality adapter" joint-generation paradigm.
(3) JavisDiT (2025): a dual-tower DiT (the video tower derived from Open-Sora, the audio tower from AudioLDM2, with both VAEs frozen); its core innovation is the HiST-Sypo Estimator (Hierarchical Spatio-Temporal Synchrony Prior Estimator) — first estimating a set of "coarse-grained global priors + fine-grained spatio-temporal priors" from the text prompt, then using that prior to jointly guide denoising of both the audio and video paths, achieving fine-grained spatio-temporal alignment; cross-modal interaction relies on cross-attention and bidirectional attention modules.
(4) JavisDiT++ (2026): switches to Wan2.1-1.3B-T2V as the base, with three upgrades — Modality-Specific Mixture-of-Experts (MS-MoE), which improves single-modality quality while preserving cross-modal interaction; Temporal-Aligned RoPE (TA-RoPE), which achieves explicit frame-level synchronization between audio tokens and video tokens, an idea sharing its lineage with Ovi's scaled-RoPE; and Audio-Video Direct Preference Optimization (AV-DPO).
(5) UniAVGen (2025): a dual-branch joint synthesis architecture, with two parallel DiTs handling audio and video respectively; the core is "Asymmetric Cross-Modal Interactions" — information flow between the two modalities is unequal — paired with a Face-Aware Modulation module and Modality-Aware Classifier-Free Guidance; a single framework uniformly supports 5 task types including joint generation, video-to-audio dubbing, and audio-driven video animation.
(6) Harmony (2025): the video branch is initialized from Wan2.2-5B, and the audio side uses MMAudio's VAE encoder + F5-TTS's speech encoder. Three innovations: Cross-Task Synergy training (using the two bidirectional generation tasks "audio-driven video" and "video-driven audio" to suppress alignment drift during joint denoising), the Global-Local Decoupled Interaction module (GLDI, where the global branch handles style alignment and the local branch handles temporal precision), and Synchronization-Enhanced CFG (amplifying the alignment signal at inference time). The authors explicitly identify three pain points of the joint diffusion paradigm: alignment instability under concurrent noise evolution, the attention mechanism's inefficiency for temporal precision, and the standard CFG's lack of cross-modal synchronization guidance.

### [Kling 3.0 / Kling Video 3.0 Omni](../models/Kling_30_Omni.md) ⚠️

Supported. Officially positioned as "native audio-visual co-output/native audio-visual synchronization": video, dialogue speech, ambient sound, and sound effects are jointly produced by a unified model within a single generation pass, requiring no cascaded post-hoc dubbing or V2A process (distinguishing it from the earlier Kling-Foley-style video→audio cascade). Based on a unified Multimodal Visual-Language (MVL) representation and 3D spatio-temporal joint attention, audio and vision are jointly modeled within a shared embedding space. Supports lip-sync and emotion sync across five languages — Chinese, English, Japanese, Korean, and Spanish — and various regional accents; the Omni version additionally supports multi-image + timbre reference binding (locking a character's appearance and timbre together), co-reference and timbre differentiation for 3+ multi-speaker scenarios, and audio continuity across multi-shot storyboards (via a Director Memory context repository). Specs: up to 15 seconds per generation, up to native 4K (3840×2160), up to 60fps, and up to 6 shots freely combinable within a single clip. [Uncertain: whether the underlying mechanism is a single DiT doing joint denoising, or a video/audio dual-branch + cross-modal-attention MoE-style fusion — the technical report does not disclose details of the audio branch]

### [LTX-2](../models/LTX-2.md)

Supported, and it is native joint generation — this is the model's core positioning. The implementation is a "decoupled yet integrated asymmetric dual-stream," neither cascaded nor MoE fusion:
(1) Each modality has its own independent VAE: video uses a spatio-temporal causal VAE; audio is first converted to a 16kHz stereo mel spectrogram (two channels concatenated along the channel dimension) and then passed through an independent causal audio VAE, where each latent token corresponds to about 1/25 second of audio, 128 dimensions; after decoding, a modified HiFi-GAN vocoder (with doubled channel count to support stereo) upsamples to reconstruct a 24kHz stereo waveform.
(2) Asymmetric dual-stream DiT: 14B-parameter video stream + 5B-parameter audio stream (19B total), sharing the same depth. Each dual-stream block executes sequentially: same-modality self-attention → text cross-attention → audio-video cross-attention → FFN, with RMSNorm between layers. The video stream uses 3D RoPE, and the audio stream uses 1D temporal RoPE.
(3) Cross-modal interaction: bidirectional audio-video cross-attention runs through the entire depth; only the temporal component of RoPE is used in cross-modal attention (forcing attention to focus on temporal synchronization rather than spatial alignment); cross-modality AdaLN is introduced — the scale/shift of one modality is determined by the hidden state and diffusion timestep of the other modality, regulating how much cross-modal information each stage absorbs.
(4) On the inference side, modality-aware CFG (Bimodal CFG): \hat{M}=M(x,t,m)+s_t(M(x,t,m)-M(x,∅,m))+s_m(M(x,t,m)-M(x,t,∅)), where text guidance s_t and cross-modal guidance s_m are tuned independently; the paper sets s_t=3, s_m=3 for the video stream and s_t=7, s_m=3 for the audio stream. Increasing s_m improves temporal synchronization and semantic consistency.
(5) The decoupled latents naturally support V2A (adding synchronized audio to an existing video) and A2V (driving video from an audio track) editing workflows.
(6) Output capability: up to 20 seconds of continuous audio-video (exceeding Veo 3's 12 seconds, Sora 2's 16 seconds, and Ovi's 10 seconds); the product-level claim is native 4K at up to 50fps; the inference strategy described in the paper is multi-scale, multi-tile: first generate a base latent at roughly 0.5MP low resolution to establish global composition/motion/audio-visual sync → a latent upsampler raises spatial resolution → overlapping spatio-temporal tiles are refined to 1080p and then fused in latent space. (4K/50fps is the product and press-release claim; the paper body only describes up to Full-HD 1080p.)

### [LongCat-Video](../models/LongCat-Video.md)

The base LongCat-Video does not support simultaneous audio-video generation — it is a purely visual generation model (T2V/I2V/video continuation) that outputs video without an audio track.
The same-series LongCat-Video-Avatar / Avatar 1.5 are "audio-driven video generation" (Audio-Text-to-Video, AT2V, and Audio-Text-Image-to-Video, ATI2V), belonging to a unidirectional-driving form where audio is the conditioning input and video is the output — not joint audio-video generation (no audio is generated). The implementation injects audio features into the same 13.6B DiT backbone: Avatar uses a wav2vec2 audio encoder, while Avatar 1.5 upgrades to a Whisper-Large encoder to improve lip-sync precision, supporting both single-stream and multi-stream (multi-person) audio input. Therefore, under the "native joint / cascade / MoE fusion" classification, this series does not fall into any of these categories and should instead be classified as "audio-conditioned video generation."

### [MOVA](../models/MOVA.md)

Supported, and it is native joint generation (a single inference pass produces both video and audio simultaneously, not cascaded). The implementation is "asymmetric dual-tower + bidirectional cross-attention Bridge fusion," with the video tower itself also being MoE:
- 32B total parameters, with 18B activated during inference.
- Video tower: Wan2.2 I2V A14B (MoE architecture, containing two groups of DiT experts — high-noise and low-noise — switched based on timestep t and threshold δ; during training, an alternating optimization strategy is used, optimizing the high-noise DiT on odd steps and the low-noise DiT on even steps, to satisfy FSDP computation-graph consistency requirements).
- Audio tower: a self-trained 1.3B text-to-audio DiT, carrying over the Wan2.1-1.3B backbone, replacing the 3D position encoding (f,h,w) with 1D position encoding along the time axis.
- Fusion: a lightweight Bridge module inserts two cross-attention blocks (video→audio and audio→video) at the hidden-state level of the two DiT backbones, with 30 layers of interaction.
- Aligned RoPE: since the video latent's temporal grid is coarse while the audio latent is dense, video indices are scaled by s = f_a/f_v and mapped to audio time units (p_v(i)=s·i, p_a(j)=j), placing the two modalities on the same time scale and avoiding temporal-misalignment drift in cross-modal attention.
- VAE: video uses the Wan2.1 video VAE, and audio uses HunyuanVideo-Foley's DAC-style audio VAE (48kHz mono); both are kept frozen throughout training.
- Training objective: flow matching; Dual Sigma Shift lets video and audio independently sample the timestep and noise schedule.
- Inference: Dual Classifier-Free Guidance (a dual-branch of text CFG + cross-modal Bridge CFG).
The task form is IT2VA (image+text → video+audio), and T2VA capability emerges as well (purely text-driven generation is achieved simply by substituting a plain white placeholder image for the reference image).

### [Merged entry: ① Mochi 1](../models/Mochi_MAGI_Motif.md)

None of the three support simultaneous audio-video generation; all are purely visual (silent) video generation models.
① Mochi 1: outputs 848×480 / 30fps / 5.4s silent video, with a single T5-XXL text encoder; there is no audio branch in the architecture, and neither the official blog nor the model card mentions audio.
② MAGI-1: the technical report in its entirety (including Section 3 DATA and Section 4 Infrastructure) does not involve audio-track extraction, audio filtering, or audio-visual alignment; the data pipeline retains only the visual frame sequence after PySceneDetect segmentation. MAGI-1.1 likewise did not add audio capability.
③ Motif-Video 2B: the data pipeline explicitly processes only four visual branches — Image Real / Image Synthetic / Video Real / Video Synthetic — and the caption's JSON schema contains no auditory fields at all; there is no audio/speech/music-related processing anywhere in the text.
All audio-related dimensions in this entry (audio_category_distribution, joint_av_caption_schema, dialogue_transcription_attributes, av_sync_detection, sync_metric_and_threshold, audio_quality_filtering, audio_type_handling) are therefore "not applicable."

### [Movie Gen](../models/Movie_Gen.md)

Does not support native joint audio-video generation; instead it uses a cascaded scheme: Movie Gen Video (30B, a Flow Matching Transformer) first generates a silent video, and then Movie Gen Audio (13B, a Flow Matching DiT + DAC-VAE latent space) generates a synchronized audio track conditioned on the video and text (V2A / TV2A), supporting audio extension to produce a full-length score of up to several minutes. Internally, the audio side is "a single model jointly generating all audio categories" (diegetic sound effects, non-diegetic sound effects, and instrumental music are generated together) rather than splitting into multiple models by category, on the grounds that correlations also exist across different audio categories. The paper's conclusion explicitly states that video and audio are currently trained separately, and that "enabling the model to jointly generate these two modalities is an important direction for future research." In addition, the model deliberately does not generate vocals/dialogue (reasoning that this can be supplemented by TTS, and is difficult to generate without a script).

### [NVIDIA NeMo Curator](../models/NeMo_Curator.md)

Not applicable — this entry is a data-processing toolchain rather than a generative model, and it does not produce any audio-video content.
Regarding "joint audio-video processing capability," its current limitations must be pointed out explicitly: although NeMo Curator claims to support four modalities (text/image/video/audio), the video pipeline and the audio pipeline are architecturally independent and non-overlapping: the video side only processes the visual track (segmentation/transcoding/frame extraction/motion and aesthetic filtering/captioning/embedding/deduplication/sharding), while the audio side is an independent workflow geared toward ASR speech data (loading → NeMo ASR transcription → WER/CER quality assessment → hand-off to text curation → export). The official documentation's audio module makes no mention of any linkage with video, nor is there a stage for extracting audio tracks from video, performing audio-video alignment, or joint annotation. It therefore currently cannot directly support the data construction needs of "simultaneous audio-video generation" models, and represents this framework's biggest gap relative to the data requirements of AV joint-generation models such as LTX-2 / Ovi / Sora 2.

### [OmniHuman Dataset + OHBench](../models/OmniHuman.md)

This entry is a dataset and evaluation benchmark rather than a generative model, and therefore does not itself generate audio-video; but its entire design serves "joint audio-video generation," making it one of the few resources in this survey built natively, on the data side, around the AV joint paradigm:
【AV nativeness on the data side】Every sample is required to contain an original audio track synchronized with the picture, and is verified via SyncNet attribution (see av_sync_detection for details); audio undergoes Demucs four-stem separation and is annotated separately for vocals and background sound; captions cover both the visual track (appearance, actions, expressions) and the auditory track (speech content, emotion, background sound, music attributes).
【Tasks supported on the evaluation side】OHBench explicitly supports several human-centric generation tasks: joint audio-video generation (audio-video joint generation, i.e., I2AV), speech-to-video generation, video dubbing, controllable human video editing, and downstream speech-generation tasks. Of these, I2AV is the primary evaluation task, covering 10 models in total (5 closed-source: Veo3.1, Wan2.5, Sora2, Kling2.6, SeedDance-1.5-pro; 5 open-source: UniVerse-1, UniAVGen, Ovi, MOVA, LTX-2); in the speech-to-video task, InfiniteTalk performs best on some metrics.
【Neutrality toward generation paradigms】The data and evaluation do not presuppose any particular implementation among "native joint / cascade / MoE fusion," accommodating both joint-generation models (Ovi, MOVA, LTX-2) and cascaded/dubbing-style models (the InfiniteTalk class), consistent with its positioning as a general-purpose benchmark.
【Empirical support】Fine-tuning LTX-2 (a native AV joint-generation model) with OmniHuman data and achieving comprehensive gains on OHBench demonstrates that this data does provide genuine benefit to AV joint-generation models (see data_ablation for details).

### [Open-Sora series](../models/Open-Sora.md)

None are supported. The entire Open-Sora and Open-Sora Plan lineages (up through Open-Sora 2.0 / Open-Sora Plan v1.5.0) are purely visual (silent) video generation models, outputting video without an audio track; the technical reports, GitHub documentation, and data pipelines contain no design or description of any audio encoder, audio latent, joint audio-video denoising, or audio-video alignment whatsoever. The data processing side likewise does not touch on the audio track at all — segmentation, scoring, and annotation are all based purely on visual frames. All audio- and audio-video-alignment-related fields under this entry (audio_category_distribution, joint_av_caption_schema, dialogue_transcription_attributes, av_sync_detection, sync_metric_and_threshold, temporal_vs_semantic_sync, audio_quality_filtering, audio_type_handling) are therefore "not applicable."

### [Ovi](../models/Ovi.md)

Supported, and it is native joint generation (one-pass joint AV generation), neither cascaded nor MoE fusion; the authors call it "twin backbone blockwise cross-modal fusion."
(1) Symmetric twin DiT: the audio tower and the video tower are architecturally identical (Model Dim 3072, FFN Dim 14336, 24 heads, head dim 128, 30 blocks, with each block containing 30 layers of Self-Attn / Text Cross-Attn / AV Cross-Attn). The video branch is initialized from Wan2.2 5B, and the audio branch is a same-architecture 5B model trained from scratch, totaling about 11B.
(2) Block-wise bidirectional cross-modal attention: within each transformer block, the audio stream attends to the video stream and the video stream attends back to the audio stream, so synchronization cues run through the entire network. Because the two towers share the same hidden dimension, no projection layers are needed at all (contrast with UniVerse-1, which requires inserted blocks + projections + an auxiliary semantic-alignment loss).
(3) Temporal alignment relies on scaled-RoPE: the video latent has 31 frames and the audio latent has 157 tokens (16kHz×5s/512); the audio branch's RoPE frequencies are scaled by 31/157≈0.197, aligning the diagonal of the cross-modal RoPE affinity matrix.
(4) A single frozen T5 encoder encodes the "merged prompt" (visual description + <S>dialogue<E> + audio description); the same text embedding does cross-attention with both the audio tower and the video tower separately, providing unified cross-modal semantic control.
(5) The training objective is flow matching; the two modalities share the same timestep t but sample independent noise, with a total loss of L=0.85·L_video+0.15·L_audio; there is no explicit synchronization loss, no face mask, no post-hoc alignment, and no auxiliary synchronization module. At inference time both branches share the same ODE solver (UniPC).
(6) Capability: the original Ovi produces 5-second 720×720@24fps output; Ovi 1.1 produces 10-second 960×960@24fps output, supporting multiple aspect ratios such as 9:16, 16:9, and 1:1, while simultaneously outputting dialogue, sound effects, and background music.

### [Script-a-Video](../models/Script-a-Video.md)

Supports simultaneous audio-video generation, but two levels need to be distinguished:
【Positioning of this work】Script-a-Video itself is not a new audio-video generation backbone; MTSS is a "structured conditioning representation on the input side." Generation capability is validated by performing conditioning substitution and lightweight architectural modifications on an existing backbone.
【Generation backbone】LTX-2 is chosen as the generation framework, for two reasons: (1) its Gemma-family VLM encoder is naturally adept at parsing JSON-style structured syntax such as MTSS, extracting fine-grained semantic instructions from the split fields; (2) its asymmetric dual-stream Diffusion Transformer architecture is itself designed for joint audio-video synthesis, allowing MTSS's Shot stream and Event stream to be mapped respectively into the latent spaces of the video branch and the audio branch.
【Implementation classification】Native joint generation (a single generation process simultaneously produces synchronized video and audio), neither cascaded nor MoE fusion.
【Two architectural improvements】
1) Shot-Aware Structured Attention: splits the Gemma-3 text embedding along MTSS's shot boundaries, then restricts each shot's corresponding video tokens to cross-attend only with that shot's semantic segment, achieving context isolation between shots and preventing cross-shot semantic bleed-through.
2) Identity Customization: via reference VAE features + learnable reference-learnable-tokens, explicitly aligns ID symbols in the Reference stream (such as "PERSON_1") with their corresponding reference images, serving as the relational bridge between visual identity and linguistic reference.
【Multimodal input form】Multimodal information is fed into Gemma-3 in an interleaved image-text format, providing semantic representations for both the video and audio branches simultaneously.
【Task form】MTSS triplet (S_ref, S_shot, S_eve) → synchronized video-audio pair (V, A), with the goal of simultaneously satisfying identity persistence and temporal-auditory precision.

### [Seedance 2.0 and Seedance 1.5 pro](../models/Seedance_20_Seedance_15_pro.md) ⚠️

Supported. Seedance 1.5 pro is native joint audio-video generation: it employs a dual-branch Diffusion Transformer (described as a dual-branch DiT in the report's abstract, and as a unified MMDiT-based framework in the body text) + a cross-modal joint module, performing multi-task pretraining on large-scale mixed-modality data while simultaneously supporting T2VA, I2VA, and purely visual T2V/I2V — it therefore belongs to "native joint" rather than the cascaded video-first-then-dub approach. Seedance 2.0 further upgrades to a unified multi-modal audio-video joint generation architecture (the official blog mentions adopting a sparse architecture), supporting four input modalities: text/image/audio/video; on the audio side it adds binaural (dual-channel stereo) capability, and supports multi-track parallel output of background music, ambient sound effects, and character vocal narration, precisely time-aligned with the visual rhythm. [Uncertain: Seedance 2.0's specific fusion method (whether it is MoE fusion) is not stated in the report]

### [SkyReels series](../models/SkyReels.md)

SkyReels-V2 does not support audio generation (a purely visual model). SkyReels-V4 supports native joint audio-video generation, implemented as a "dual-stream MMDiT (Multimodal Diffusion Transformer)," which is native joint rather than cascaded or MoE:
(1) The video branch and the audio branch form a symmetric twin backbone; the video branch is initialized from an existing text-to-video model, while the audio branch is trained from scratch, with the two matched in specification;
(2) A shared frozen multimodal large language model (MLLM) serves as the text encoder, directly processing descriptive text that unifies "visual + auditory" content, so a single prompt drives both modalities simultaneously;
(3) Each transformer block internally features bidirectional audio-video cross-attention (audio→video and video→audio conditioning each other); the front section consists of dual-stream MM layers, transitioning to single-stream hybrid blocks in the back section;
(4) The two modalities' timing is aligned via RoPE temporal-dimension scaling (the audio/video token ratio is about 0.0963; on the audio side, 44.1kHz over 5 seconds corresponds to 218 latent tokens, aligning with the temporal resolution of 21 video frames);
(5) Generation capability: up to 1080p, 32FPS, 15 seconds, supporting multi-shot cinematic-grade content with synchronized audio track;
(6) Efficiency strategy: the base model first generates "a full low-resolution sequence + high-resolution keyframes," and a Refiner module (joint super-resolution + frame interpolation based on Video Sparse Attention) then produces the final output;
(7) Unification: through a channel-concatenation inpainting framework + different mask configurations, generation, inpainting, and editing are unified into the same model, with inputs that can be text, images, video clips, masks, or audio references.

### [Sora 2](../models/Sora_2.md) ⚠️

Supported, and it is native joint generation — this is Sora 2's core upgrade relative to Sora 1. The System Card explicitly positions it as a "video and audio generation model," with new capabilities including "synchronized audio." Audio is not a post-hoc cascaded video-to-audio module, but is produced via joint denoising with video within the same generation pipeline: video and audio are each compressed into latents by their own encoders, and the same transformer diffusion backbone then denoises both latent streams simultaneously. It can generate dialogue (including lip-sync), sound effects/foley, ambient sound, and background music, with volume and spatial positioning varying with object and camera distance. Note: the architectural description above (dual encoders + shared diffusion backbone, 3D RoPE, an audio backbone sharing lineage with the GPT-4o multimodal system) all comes from third-party technical interpretation and secondhand reporting; OpenAI has never officially confirmed it, and it should be treated as speculative information. [Uncertain]

### [SpeakerVid-5M](../models/SpeakerVid-5M.md)

At the dataset level: natively audio-video paired (every clip comes with a native audio track synchronized to the picture; samples with no audio track or with unsynchronized audio and video are removed during the cleaning stage), making it qualified training corpus for "joint audio-video generation," and also one of the few current audio-video datasets organized around dyadic interaction as its core form.
At the baseline-model level: supports simultaneous audio-video generation, implemented as a native joint autoregressive framework, not cascaded. Specifically, it consists of Qwen2.5-Omni as the multimodal understanding backbone → simultaneously predicting video tokens and audio tokens; the video side is encoded with a 3D VAE (temporal stride 4, spatial stride 8), and the audio side uses the CosyVoice2 audio tokenizer; this is followed by a spatial transformer for frame-by-frame refinement and a diffusion MLP for visual detail enhancement. Trainable parameters: 0.8B. A noise injection strategy is introduced during training to mitigate autoregressive error accumulation.
This dataset has been directly adopted as training corpus by multiple downstream joint audio-video generation models; a typical example is MOVA, which lists SpeakerVid-5M as one of its Phase 1 data sources, calling it the core source of its lip-sync capability.

### [Step-Video-T2V](../models/Step-Video-T2V.md)

Not supported. Both Step-Video-T2V and Step-Video-TI2V are purely visual video generation models, outputting video without an audio track; the technical report does not touch on the audio modality anywhere in the text, and the data pipeline contains no audio-track-processing steps whatsoever (only visual frames are retained after segmentation).
StepFun's audio capabilities are handled by an entirely separate model line: Step-Audio (a product-grade open-source speech interaction model supporting emotion, dialects, languages, singing voice, and personalized styles), open-sourced the same day as Step-Video-T2V, along with the subsequent Step-Audio series. There is no joint training, no shared latent, and no joint denoising between the two; this is a typical case of a "video model + speech model" running in parallel dual lines, rather than cascaded or native joint AV generation.
This entry therefore does not constitute a reference sample on the joint audio-video generation dimension, and all audio-related fields in this survey (audio_category_distribution, joint_av_caption_schema, dialogue_transcription_attributes, av_sync_detection, sync_metric_and_threshold, temporal_vs_semantic_sync, audio_quality_filtering, audio_type_handling) are not applicable.

### [UniTalking](../models/UniTalking.md)

Supported, as native end-to-end joint generation (a single diffusion inference pass simultaneously produces video and speech), implemented as "symmetric twin dual-stream + joint attention":
【Architecture】Based on MM-DiT (Multi-Modal Diffusion Transformer), with a backbone of N=30 MM-DiT blocks, model dimension dim=3072, 24 attention heads, and a total model parameter count of 10B. Trained as a continuous normalizing flow (CNF) via Flow Matching; inference uses the UniPC solver together with Classifier-Free Guidance.
【Asymmetric design of dual-stream initialization】The video stream directly inherits the architecture and weights of Wan2.2-5B, serving as a strong visual prior; the audio stream is designed as an "identical twin" of the video stream — architecturally identical but with randomly initialized parameters — the purpose being to place both modalities naturally in an isomorphic representation space so as to facilitate fusion at the latent level. This is fundamentally different from UniVerse-1's route of "stitching together two heterogeneous experts (Wan2.1 + Ace-step) and then aligning them via layer interpolation": UniTalking trades "structural symmetry + training an audio tower from scratch" for simplicity of fusion, at the cost of the audio tower lacking a pretrained prior (necessitating a separate audio-pretraining stage to compensate).
【Fusion mechanism: Joint Attention】The video and audio latent tokens are concatenated and fed into a single attention operation, letting the model simultaneously learn intra-modal and inter-modal dependencies within one attention pass, explicitly modeling the temporal correspondence between visemes and phonemes. This is "shared self-attention," rather than UniVerse-1/JavisDiT-style cross-modal cross-attention.
【Conditioning injection: Cross Attention】The text condition (UMT5-encoded) and the reference-timbre condition (MMAudio VAE-encoded) each undergo cross-attention through their own key-value projection layers, and the two outputs are fused via element-wise summation. Setting up a dedicated set of KV projection layers for the reference audio is this work's specific design choice for conditioning injection.
【Anisotropic RoPE】Following OVI's approach, the temporal axis t uses standard RoPE, but the audio token's spatial dimensions (h, w) use RoPE derived from a single fixed position. This "spatial degeneration" design forces the model to concentrate its attention budget on the temporal dimension, architecturally biasing it toward audio-video temporal alignment.
【Supported tasks】T2AV (text → audio-video), TI2AV (text + identity image → audio-video), TR2AV (text + reference timbre → audio-video), TV2A (text + video → audio, used as a unidirectional supervision signal during training).

### [UniVerse-1](../models/UniVerse-1.md)

Supported, as native joint generation (a single inference pass simultaneously produces video and audio), but the implementation path is this work's core innovation — rather than training a joint model from scratch, it "stitches" two already-pretrained single-modality experts together (Stitching of Experts, SoE):
【Expert sources】The video expert is Wan2.1 (1.3B DiT + 3D VAE + umT5 text encoder); the audio expert is Ace-step (a 3.5B music generation model comprising Music-DCAE + umT5 + a lyrics encoder + a speaker encoder + DiT). After merging, the total model size is about 7B.
【Fusion method】Deep fusion is performed at the level of corresponding blocks between the two DiTs, rather than merely concatenating at the input/output ends. The fusion component is a lightweight cross-modal MLP connector: a two-layer linear adapter aligns the feature spaces, and dedicated key (kproj) and value (vproj) projection layers are configured for cross-modal attention.
【Depth-alignment problem】The two experts have different numbers of transformer layers, which is resolved via layer interpolation — new blocks are inserted at uniform intervals, with the weights of each new block initialized by linearly interpolating the weights of adjacent layers, so that the two towers' depths can be stitched together one-to-one.
【Structural modification】Ace-step's original speaker encoder is removed, with the goal of freeing the model from the constraint of speaker-specific generation so it generalizes to arbitrary speakers (timbre is implicitly determined jointly by the reference image and the text).
【Temporal-grid alignment】The audio sample rate is adjusted from the original 44.1 kHz to 25.6 kHz, so that the audio latent's temporal grid is strictly aligned with the video's 25 fps — an engineering trade-off of adapting the sample rate to the video frame rate, different from MOVA's approach of index scaling via Aligned RoPE.
【VAE】The video 3D VAE compresses (3,T,H,W) to (16,T/4,H/8,W/8); the audio Music-DCAE compresses the mel spectrogram (8,T,F) to (8,T/8,F/8).
【Noise sampling】An Independent Noise Sampling Strategy (INSS) is proposed, avoiding spurious cross-modal noise correlation that would arise if the two modalities shared PRNG state.
【Task form】Reference image + text → video + audio (IT2VA); Verse-Bench supports evaluation of four task types simultaneously: joint generation, audio-to-video, video-to-audio, and TTS.

### [Unison](../models/Unison.md)

Supports simultaneous audio-video generation, implemented as "dual-branch native joint generation" — two expert models coupled via frame-level bidirectional cross-attention, a continuation and deepening of UniVerse-1's "Stitching of Experts" paradigm, rather than a single-tower unified model trained from scratch, and not cascaded either.
【Branch composition】
- Video branch: based on Wan2.2-5B, 29 layers (L_v = 29). The video backbone is completely frozen during the joint training stage.
- Audio branch: based on MMAudio, 23 layers (L_a = 23), with Zipformer (from the ZipVoice family) introduced to obtain speech-generation capability — MMAudio was originally a foley/sound-effect generation model without speech synthesis capability, and the introduction of Zipformer is precisely meant to fill that gap.
【Coupling method】Frame-level bidirectional cross-attention, where the video latent and the audio latent each serve as query for the other, achieving bidirectional information exchange. This contrasts with unidirectional-conditioning schemes such as UniAVGen and Harmony, and the paper lists "bidirectionality" as a core distinguishing point.
【Three-layer safeguard mechanism for lip-sync】The paper explicitly decomposes word-level lip-sync into three levels of constraint applied simultaneously, which is the key to understanding Unison's data-side design:
1) Architecture level: cross-attention aligns features within a three-frame window, stride=1, retaining only the representation of the middle frame upon recovery — an extremely short-window local-alignment design;
2) Data level: a lip-filtering operator (first detecting the number and position of faces, then running SyncNet verification for alignment only within the face bounding boxes);
3) Training level: a bidirectional cross-modal forcing strategy (CMFS).
【Dual-stream decoupling on the audio side — this work's most distinctive feature】The audio latent is not a single tensor, but a dual-stream tensor of [speech, sfx]. The source audio is separated into speech and sound-effect paths via Mel-RoFormer, each encoded as a temporally aligned independent sequence; the two paths share the same RoPE temporal index to guarantee strict temporal alignment, and "modality-specific learnable biases" are used to distinguish the two within the shared self-attention layer. Each Transformer block executes an "interact–merge–split" cycle; at the exit, the streams are re-separated into two independent generation trajectories, ultimately supervised by two independent flow-matching losses. This means Unison physically isolates speech and sound effects already at the generation stage, eliminating the "speech suppressing ambient sound" problem at the source — a fundamental difference from single-mixed-audio-track schemes such as Ovi/LTX-2/MOVA.
【Cross-modal forcing】The diffusion timesteps for video and audio are sampled independently (the audio branch is mapped to a restricted interval); the modality with lower noise (i.e., "cleaner") guides the "dirtier" modality, with a direction indicator dynamically designating the "student modality" and upweighting its loss. A three-stage curriculum (synchronized warmup → incremental decoupling → full independence, with stage proportions of 0.3/0.4/0.3) ensures training stability.
【Inference configuration】A 50-step flow-matching sampler, CFG scale = 6.0, outputting 25 FPS video.

### [Veo 3 / Veo 3.1](../models/Veo_3_Veo_31.md)

Supported, and it is native joint generation, not a cascaded pipeline. The original text of the official technical report states: "Veo 3 uses latent diffusion, in which the diffusion process is applied jointly to the temporal audio latents, and the spatio-temporal video latents." That is, video and audio are each encoded by their own autoencoder into compressed latent representations (spatio-temporal latents for video, temporal latents for audio), after which a Transformer-based denoising network jointly denoises the two types of latents within the same diffusion process — every denoising step processes audio-video tokens simultaneously, naturally guaranteeing synchronization at the generation stage. The generated content covers dialogue (with lip-sync), sound effects/foley, ambient sound, and background music. It does not employ MoE fusion or a two-stage cascade of video-first-then-dub.

### [Vidu S1](../models/Vidu_S1.md)

Supported, and it is native joint generation. The model concatenates the clean video representation v_0^i and the audio representation a_0^i of frame i along the modality dimension into a joint state x_0^i = [v_0^i; a_0^i], denoising the joint video-audio latent sequence uniformly within the same diffusion denoising model — neither cascaded nor MoE. The unified conditioning interface c simultaneously contains speech, text prompt, and a reference first-frame image — that is, speech serves both as a conditioning control signal (real-time user speech instructions controlling character behavior) and, within the joint state, is also part of the generated audio track. The overall paradigm is autoregressive + diffusion (AR+Diffusion) causal streaming generation, with sliding-window decoding supporting unbounded-length generation.

### [Wan 2.5 / 2.6 / 2.7](../models/Wan.md) ⚠️

Supported, and this is a watershed capability introduced in version 2.5. The official claim is "native audio-visual synchronization," but paper-level evidence is lacking.
【Capability evolution】
- Wan 2.1 (Feb 2025): audio is a separate cascaded V2A module (Section 5.7 of the technical report), generating video first and then dubbing it; and it explicitly produces only ambient sound and background music, explicitly excluding speech/vocal singing.
- Wan2.2-S2V (Aug 2025): a dedicated model for audio-driven character animation (audio→video), belonging to conditional generation rather than joint generation.
- Wan 2.5-preview (Sep 2025): the "audio-enabled video / audio-visual sync" label appears for the first time in the official capability listing; both T2V and I2V can automatically add sound when no audio is provided (generating matching background music or sound effects), and can also accept audio via input.audio_url to drive generation. Lip-sync capability becomes a selling point of the series from this point on.
- Wan 2.6 (Dec 2025): the capability label upgrades to "multi-shot narrative + audio-visual sync"; officially described as "native audio-visual sync, with the picture perfectly matching voice, sound effects, and BGM," and adds a "voice-driven" mode (audio directly drives character lip movement and performance) and "role play" (uploading a personal video to replicate appearance and voice). It is officially billed as "the first video generation model in China to support role play" and "the most feature-complete video generation model in the world."
- Wan 2.7 (Jun 2026): continues "multi-shot narrative + audio-visual sync," with audio as a first-class input modality (input.audio_url, wav/mp3, 2–30 seconds, ≤15MB); on the I2V side the input modalities expand to "text+image+audio+video" (supporting first-frame, first-and-last-frame, video continuation, and continuation+last-frame control). This is consistent with this survey's note that "2.7 adds native audio conditioning" — audio shifts from "output-side dubbing" to "input-side conditioning."
【Implementation approach】The official marketing copy describes it as "Alibaba's in-house native multimodal architecture," pointing toward native joint generation rather than cascading; but no paper, architecture diagram, or parameter disclosure confirms this, and it is impossible to determine whether MoE fusion is used (Wan 2.2's MoE is a high-noise/low-noise dual-expert split of denoising stages, unrelated to modality). [Uncertain: the specific fusion mechanism]
【Output specifications】wan2.5-*-preview: 480P/720P/1080P, 5s or 10s; wan2.6-*: 720P/1080P, integer 2–15s; wan2.7-*: 720P/1080P, integer 2–15s, with ratio support for 16:9/9:16/1:1/4:3/3:4 (e.g., 1080P 16:9 = 1920×1080, 4:3 = 1648×1248); the entire lineup is fixed at 30fps, MP4/H.264.

### [Collection of Audio-Video Generation Evaluation Benchmarks](../models/av_benchmarks.md)

None of the five generate audio-video themselves; instead, they evaluate joint audio-video generation capability. The forms of systems-under-test that they cover happen to constitute the three current technical routes of AV generation, and the evaluation design explicitly distinguishes between:
1) Native joint generation (end-to-end T2AV/I2AV): Sora 2, Veo 3 / Veo 3.1 / Veo3-Fast, Wan 2.5 Preview / Wan 2.6, Kling 2.5 Turbo / Kling v2.6, Seedance 1.5 Pro; on the open-source side, Ovi, LTX, MOVA, UniVerse-1, JavisDiT / JavisDiT++;
2) Cascaded combinations (V+A, generating video first and then dubbing): on the video side, Seedance-1.0-Lite / Wan2.2-TI2V / Kling2.5 Turbo; on the audio side, MMAudio, ThinkSound(-Light), HunyuanVideo-Foley, FoleyCrafter;
3) Representation/discriminative models (the systems under test in AV-SyncBench): Synchformer, SparseSync, ImageBind, CAV-MAE, CAV-MAE-Sync.
VABench additionally introduces the route of "stereo audio-video generation," using 116 prompts that explicitly specify left/right channel positioning to examine spatial audio generation capability — a currently rare stereo AV evaluation dimension.

### [Video Captioning Model Ecosystem](../models/caption_models.md)

Captioners themselves do not perform generation; in this entry, this field is reinterpreted as "whether audio-video dual-modality input is supported and a joint description is output" — this is the most critical capability watershed in this ecosystem during 2025–2026:
【Purely visual captioners (not listening to sound)】ShareCaptioner-Video, Tarsier / Tarsier2, CogVLM2-Caption, SkyCaptioner-V1, AuroraCap, LLaVA-Video, PLLaVA, Aria, Qwen2.5-VL / Qwen3-VL, and all 31 candidate annotators from Panda-70M (the Panda-70M paper explicitly states that its Video-LLaMA branch "uses only the vision branch, with the audio branch explicitly disabled"). This tier dominated the mainstream absolutely through 2024 and the first half of 2025.
【Cascaded audio-video annotation (multiple models dividing labor, then fused by an LLM)】The mainstream engineering approach, represented by: MOVA = MiMo-VL-7B-RL (vision) + Qwen3-Omni-Instruct (ASR) + Qwen3-Omni-Captioner (non-speech) + GPT-OSS-120B (fusion and consistency verification); the Movie Gen audio side = four models working together — audio quality prediction + AED + general audio captioning + music captioning; UniTalking = Qwen3-VL + Whisper-V3 + Qwen3-Omni-Captioner + Qwen3-Omni; Kling-Foley = a three-stage pipeline of audio classification + a large audio-understanding model + LLM fusion.
【Native joint audio-video captioner (a single model both seeing and listening)】A new paradigm that took shape starting Q4 2025: AVoCaDO (Qwen2.5-Omni-7B backbone, ~9B end-to-end), AVSCap-7B (same backbone), video-SALMONN 2 (LLaVA-OneVision + audio LoRA, 3B/7B/72B), Qwen3-Omni / Qwen3.5-Omni, and an unnamed system developed in-house by Lightricks for LTX-2. UniVerse-1 goes even further, using a single Qwen2.5-Omni to output speech content/video caption/ambient-sound caption in parallel in one pass.
【Key empirical finding】The zero-shot annotation capability of bare Qwen2.5-Omni is very poor (AVSCapBench overall only 21.53, Speech 13.92); it must go through caption-specific SFT+RL before it can be used as an annotator (AVoCaDO 49.31, AVSCap 60.44) — "having an omni backbone ≠ being usable as an omni annotator" is the most important engineering lesson in this ecosystem.

### [Collection of Geometric/Structured Annotation Datasets](../models/geometric_datasets.md)

None support simultaneous audio-video generation, nor do any involve the audio modality. All four are purely visual geometric/structured annotation paradigms: SceneScribe-1M and SpatialVID target camera-controllable video generation and 3D/4D perception, WildWorld targets action-conditioned world models, and Action100M targets action understanding and video-text representation learning. Action100M derives from HowTo100M (which includes narration ASR), but the ASR is used only to assist text supervision, not for audio generation.

### [Post-Training Data Strategies for Video Generation](../models/post_training_data.md)

This topic is not a model itself, but the objects it covers include numerous joint audio-video generation models, and their post-training exhibits a clear stratification:
【Those that have already incorporated AV into post-training rewards】Seedance 1.5 pro — explicitly adopts "an RLHF algorithm customized for audio-video scenarios" with a multi-dimensional reward model, jointly optimizing motion quality, visual aesthetics, and audio fidelity, and applies infrastructure optimizations to the RLHF pipeline bringing nearly 3x training acceleration; Kling 3.0 Omni — samples multiple video variants under the same MVL (multimodal visual-language) condition, with human evaluators comparing them to form preference pairs for DPO (though whether the audio dimension is scored as an independent item is not disclosed); JavisDiT++'s AV-DPO — division of labor among six reward models, with temporal synchronization handled by Synchformer, audio quality handled by AudioBox-Aesthetics, and text-audio and cross-modal similarity handled by ImageBind — currently the only work to fully disclose the details of constructing AV preference pairs.
【The anchor paper's AV handling】This is involved only at the autoregressive distillation stage: for models with audio-video generation capability, following OmniForcing (arXiv:2603.11647), the model is equipped with "asymmetric block-causal alignment" and an "audio sink token." That is, AV only manifests at the distillation-architecture level; the framework's four reward models for SFT and GRPO (video aesthetics/image aesthetics/motion quality/text-video alignment) are all purely visual dimensions, containing no audio or audio-video synchronization reward whatsoever — a notable gap in this framework in the AV era.
【Entirely blank】Academic AV works such as HunyuanVideo-Foley, Ovi (original version), UniVerse-1, UniTalking, Unison, Foley-Omni, and InstructAV2AV all have no preference-alignment post-training whatsoever.

### [Merged Survey of Mainstream Video Pretraining Datasets: Panda-70M, InternVid, Koala…](../models/pretraining_datasets.md)

**All seven datasets do not support simultaneous audio-video generation**; all are pure visual+text datasets. Audio handling falls into three tiers: (1) Not involved at all — in the papers for InternVid, Koala-36M, MiraData, OpenVid-1M, and LVD-2M, the word "audio" appears only in reference-list titles (a full-text grep confirms Koala-36M and LVD-2M each contain it only once, in a citation title), and the metadata contains no audio fields whatsoever; (2) Audio track passively retained but not processed — Panda-70M's download configuration panda70m.yaml sets download_audio: True, so the mp4 files users obtain are muxed with the original YouTube audio track, but the dataset itself contains no audio annotation, audio quality control, or speech labels, and its teacher model Video-LLaMA has its audio branch explicitly disabled ("We only use the vision branch"); UltraVideo retains the native audio track and mentions in its Limitations that "thanks to the preservation of native resolution, frame rate, and audio…[it] can be used for tasks such as music generation," but likewise has no audio-side filtering or annotation whatsoever; (3) Koala-36M's re-segmentation code outputs via cv2.VideoWriter/mp4v, which **directly discards the audio track**. All audio and audio-video-alignment fields under this entry (audio_category_distribution, joint_av_caption_schema, dialogue_transcription_attributes, av_sync_detection, sync_metric_and_threshold, temporal_vs_semantic_sync, audio_quality_filtering, audio_type_handling) are therefore "not applicable."
## Research information source list (URLs of papers/technical reports/official documentation/news, with each source's nature labeled: official primary / same-team corroboration / third-party report)

`sources` · Detail level: brief

### [Allegro](../models/Allegro.md)

1. https://arxiv.org/abs/2410.15458 —— Official primary. The Allegro paper "Allegro: Open the Black Box of Commercial-Level Video Generation Model," 2024-10-20 v1. The data chapter (Sec. 2 Data: Data Filtering / Data Annotation / Data Stratification, the Table 1 threshold table, Appendix A data distribution figure Fig.11) is the primary source of all quantitative information in this entry.
2. https://arxiv.org/html/2410.15458v1 —— Official primary. Full HTML text of the paper, including Table 1 (staged filtering thresholds), the training-stage configuration table (resolution / frame count / sample count / batch / steps / GPU count), and the Appendix A distribution statistics.
3. https://github.com/rhymes-ai/Allegro —— Official primary. Code repository and README, including the Apache 2.0 license, release timelines for each version, model specifications, and training code.
4. https://huggingface.co/rhymes-ai/Allegro —— Official primary. T2V weights card.
5. https://huggingface.co/rhymes-ai/Allegro-TI2V —— Official primary. TI2V weights card (2024-11-25).
6. https://huggingface.co/blog/RhymesAI/allegro —— Official primary. Rhymes AI's official blog post "Allegro: Advanced Video Generation Model."
7. https://arxiv.org/abs/2410.05993 —— Same-team corroboration. The Aria multimodal MoE model paper (the base model for the captioning model: 25.3B total parameters / 3.9B activated vision tokens, supports 64K multimodal input, captions a 256-frame video in under 10 seconds).
8. https://huggingface.co/papers/2410.15458 —— Third-party aggregation. Hugging Face paper page and community discussion.
9. https://www.marktechpost.com/2024/11/28/rhymes-ai-unveils-allegro-ti2v-a-breakthrough-in-visual-storytelling-with-open-source-ai-video-generation-technology/ —— Third-party report. Report on the Allegro-TI2V release.

### [Apollo](../models/Apollo.md)

- 【Official primary】arXiv:2601.04151 "Apollo: Unified Multi-Task Audio-Video Joint Generation" (v1, 2026-01-07) / "Klear: Unified Multi-Task Audio-Video Joint Generation" (v2, 2026-01-13): abstract page https://arxiv.org/abs/2601.04151 , full HTML https://arxiv.org/html/2601.04151v2 and https://arxiv.org/html/2601.04151v1 , PDF https://arxiv.org/pdf/2601.04151 —— The sole direct source for almost every field in this entry. The data-related content is concentrated in Section 4, Dataset Construction (4.1 Dataset Filtering, 4.2 Audio-Guided Data Splitting, 4.3 Dense Annotation and Integration) and Figure 3, the data annotation pipeline diagram. This section is extremely short (about one page), which is the root cause of why so many fields in this entry are marked as uncertain.
- 【Official primary】HuggingFace paper page https://huggingface.co/papers/2601.04151 —— Confirms the credited affiliation as "Kling Team, Kuaishou Technology," and confirms that no model, dataset, or code repository link is attached.
- 【Same-team corroboration】Kuaishou investor relations announcement "Kling AI Launches Video 2.6 Model with 'Simultaneous Audio-Visual Generation' Capability": https://ir.kuaishou.com/news-releases/news-release-details/kling-ai-launches-video-26-model-simultaneous-audio-visual —— Used to corroborate the correspondence between this research and the Kling product line's simultaneous audio-visual generation capability (corroborating evidence, not stated explicitly in the paper).
- 【Third-party index】NASA ADS entry https://ui.adsabs.harvard.edu/abs/2026arXiv260104151W/abstract , ResearchGate entry https://www.researchgate.net/publication/399559825_Klear_Unified_Multi-Task_Audio-Video_Joint_Generation —— Used to cross-check the fact of the Apollo→Klear renaming and the author list.

### [CineDance / CineDance-1M](../models/CineDance.md)

1. 【Official primary】Paper arXiv abstract page https://arxiv.org/abs/2606.09639 —— Title, authors, v1/v2 submission dates, abstract.
2. 【Official primary】Paper full text HTML https://arxiv.org/html/2606.09639v2 —— The three-stage pipeline, the filtering-funnel table (Tab.3), the annotation schema, the six CineBench dimensions, the ablation tables (Tab.4/5/7), and the dataset comparison table (Tab.6).
3. 【Official primary】Paper PDF https://arxiv.org/pdf/2606.09639
4. 【Official primary】Project homepage https://aliothchen.github.io/projects/CineDance/ —— The five participating institutions, an overview of dataset scale, and entry points for each resource.
5. 【Official primary】GitHub repository https://github.com/AliothChen/CineDance —— Open-source progress checklist (dataset released gated in batches, code and weights pending).
6. 【Official primary】HuggingFace dataset card https://huggingface.co/datasets/CineDance/CineDance_01 —— Scale of the first shard release: 5.83TB / 240,488 clips / 150 TAR files, CC-BY-NC-SA-4.0 license, gated access, video-only current status, and limitations statement.
7. 【Third-party index】awesome-video-generation list https://github.com/kongzhecn/awesome-video-generation —— Inclusion as corroboration.

### [CogVideoX](../models/CogVideoX.md)

- Official primary: CogVideoX: Text-to-Video Diffusion Models with An Expert Transformer (arXiv:2408.06072v3, ICLR 2025, 30 pages including Appendices G/J/K) https://arxiv.org/abs/2408.06072 and https://arxiv.org/pdf/2408.06072
- Official primary: zai-org/CogVideo open-source repository (code, weights, captioning toolchain) https://github.com/zai-org/CogVideo
- Official primary: CogVLM2-Caption captioning model weights (the video captioning model used in CogVideoX training) https://huggingface.co/zai-org/cogvlm2-llama3-caption
- Same-team corroboration: Zhipu's "New Qingying" CogVideoX+CogSound technical deep dive (reposted by BAAI Hub from Zhipu's official technical explainer, including a description of CogSound's chunked temporal-alignment cross-attention and data filtering framework) https://hub.baai.ac.cn/view/40956
- Third-party report: Zhipu's video generation model Qingying upgrade (new Qingying: 10s/4K/60fps/built-in sound effects, 2024-11-08) https://cn.technode.com/post/2024-11-08/zhipu-qingying-new/
- Third-party report: Introduction to CogSound's sound-effect model capabilities https://ai-bot.cn/cogsound/
- Third-party compilation: CogVideoX paper literature review (Moonlight, including data filtering and the 35M-clips figure) https://www.themoonlight.io/en/review/cogvideox-text-to-video-diffusion-models-with-an-expert-transformer

### [Cosmos-Predict2.5](../models/Cosmos-Predict25.md)

1. arXiv:2511.00062 "World Simulation with Video Foundation Models for Physical AI" https://arxiv.org/abs/2511.00062 , full PDF https://arxiv.org/pdf/2511.00062v2 —— Official primary, the core basis. This survey's seven-stage pipeline, the 35M hours / 6B clips / 4% retention rate / 200M clips figures, the various filters, Qwen2.5-VL-7B captioning, semantic deduplication, the 26-category taxonomy sharding, the five major domains of data, the SFT five-domain scale table, the RL and distillation configuration, and the infrastructure and MFU data are all taken verbatim from Section 2 (Data), Section 4 (Training), and Section 6 (Applications) of the v2 version.
2. Official GitHub repositories https://github.com/nvidia-cosmos/cosmos-predict2.5 and https://github.com/nvidia-cosmos/cosmos-transfer2.5 —— Official primary (Apache 2.0 code license, NVIDIA Open Model License for the model, list of released models, mentions of guardrails).
3. Hugging Face model cards https://huggingface.co/nvidia/Cosmos-Predict2.5-2B and https://huggingface.co/nvidia/Cosmos-Predict2.5-14B —— Official primary (weights, license, capability description).
4. NVIDIA Research Cosmos Lab project page https://research.nvidia.com/labs/cosmos-lab/cosmos-predict2.5/ —— Official primary (the official messaging on the 200M high-quality pretraining clips, model merging, and RL).
5. NVIDIA Cosmos Curator open-source repository https://github.com/NVIDIA/cosmos-curator and documentation https://docs.nvidia.com/cosmos-curator-lha/current/introduction.html —— Official primary, same-team corroboration (the productized implementation of the curation pipeline described in the paper, engineering details of the GPU streaming framework Cosmos-Xenna and the shot-splitting / embedding / captioning stages).
6. NeMo Curator video curation documentation https://docs.nvidia.com/nemo/curator/curate-video and https://docs.nvidia.com/nemo/curator/latest/get-started/video.html —— Official primary, corroboration (GPU-accelerated video pipeline capabilities, the throughput figure of roughly 1 million hours of 720p video processed per day on 2,000 H100s).
7. Prior-generation paper, NVIDIA (2025) "Cosmos World Foundation Model Platform for Physical AI" —— Official primary, corroboration (the reference baseline of 20M hours / 30% survival rate, and the Cosmos-Guardrail safety system).
8. Emergent Mind topic page https://www.emergentmind.com/topics/cosmos-predict2-5 , DL reading-group explainer https://www.docswell.com/s/DeepLearning2023/KPGPD6-2025-11-17-152333 —— Third-party index and interpretation (used to cross-check figures, not a primary basis).

### [Data-Juicer 2.0](../models/Data-Juicer.md)

1. 【Official primary】Data-Juicer 2.0 paper arXiv abstract page https://arxiv.org/abs/2501.14755 —— Authors, version history (v1 2024-12-23 / v2 2025-06-04 / v3 2025-10-29), NeurIPS 2025 Spotlight.
2. 【Official primary】Data-Juicer 2.0 full text HTML https://arxiv.org/html/2501.14755v3 —— The three-layer architecture, the taxonomy of 150+ multimodal operators, the TB-scale / 10k+ CPU core scale, MinHash Ray deduplication performance, speedup figures, and the Alibaba Cloud PAI deployment.
3. 【Official primary】Data-Juicer 2.0 PDF https://arxiv.org/pdf/2501.14755
4. 【Official primary】Data-Juicer Sandbox paper arXiv abstract page https://arxiv.org/abs/2407.11784 —— ICML 2025 Spotlight, Probe-Analyze-Refine, topping the VBench leaderboard.
5. 【Official primary】Sandbox full text HTML https://arxiv.org/html/2407.11784v3 —— The text-to-video case study, data-pool splitting, operator selection, the VBench score table, and the quality-vs-compute conclusion.
6. 【Official primary】GitHub main repository https://github.com/datajuicer/data-juicer —— Version timeline, operator statistics, list of adopters, list of related papers, Apache 2.0 license.
7. 【Official primary】Operator summary documentation https://datajuicer.github.io/data-juicer/en/main/docs/Operators.html and https://raw.githubusercontent.com/datajuicer/data-juicer/main/docs/Operators.md —— The item-by-item names and descriptions of the 229 operators; the authoritative source for the list of video/audio operators.
8. 【Official primary】HuggingFace dataset https://huggingface.co/datasets/datajuicer/data-juicer-t2v-optimal-data-pool —— 147,176 samples, 12.09% retention rate, 227.5GB, composition of source datasets (InternVid 606k / Panda-70M 605k / MSR-VTT 6k), the two filtering operators, and the CLIP threshold of 0.306337.
9. 【Official primary】OpenReview review pages https://openreview.net/forum?id=NiL5U1DrRN (DJ 2.0) and https://openreview.net/forum?id=zIGIvysR1H (Sandbox).
10. 【Same-team corroboration】Alibaba Cloud Developer Community article "New VBench leaderboard champion! Data-Juicer's Sandbox Lab powers collaborative development of multimodal data and models" https://developer.aliyun.com/article/1570605 and the identical article on ModelScope Community https://community.modelscope.cn/669f1a7b76e87a79e35ada49.html —— Chinese-language case-study description.
11. 【Same-team corroboration】HumanVBench paper https://arxiv.org/html/2412.17574v2 (CVPR 2026) —— Uses 20+ DJ operators to build a person-centric video annotation pipeline; a use case of DJ's video operators in real-world dataset construction.
12. 【Official primary】ICML 2025 Poster page https://icml.cc/virtual/2025/poster/43484 and the official PMLR version https://proceedings.mlr.press/v267/chen25bm.html
13. 【Third-party index】ResearchGate https://www.researchgate.net/publication/388421863_Data-Juicer_20_Cloud-Scale_Adaptive_Data_Processing_for_Foundation_Models
14. 【Third-party report】CSDN blog post "Data-Juicer: Alibaba's honorary open-source large-model data cleaning framework" https://blog.csdn.net/qq_41895747/article/details/140150556 —— Chinese community interpretation, no new primary data.

### [Foley-Omni](../models/Foley-Omni.md)

1. 【Official primary】arXiv paper abstract page https://arxiv.org/abs/2606.03672 —— Title, authors, abstract, submission date of June 2, 2026.
2. 【Official primary】arXiv full text HTML https://arxiv.org/html/2606.03672v1 —— Section 3.1's data cleaning pipeline, Table 7's filtering thresholds, Table 9's training data composition, Table 6's ablations, the V2ST-Bench main results table, and Appendices B/C.1. The core source of information.
3. 【Official primary】arXiv PDF https://arxiv.org/pdf/2606.03672
4. 【Official primary】HuggingFace model page https://huggingface.co/CocoBro/Foley-Omni —— 5.5B parameter count, MIT license, inference-only weight composition, upstream dependency statements.
5. 【Official primary】GitHub repository https://github.com/NJU-Speech/Foley-Omni —— Inference code, feature extraction scripts, V2ST-Bench listed as "coming soon."
6. 【Official primary】Project homepage https://ty0402.github.io/Foley-omni-Web/ —— Demo examples and project introduction.
7. 【Third-party index】ResearchGate listing page https://www.researchgate.net/publication/405852241_Foley-Omni_A_Unified_Multimodal_Generation_Model_from_Task-Level_Audio_Synthesis_to_Complete_Video_Soundtrack_Generation
8. 【Third-party report】AI Films Studio blog post https://studio.aifilms.ai/blog/foley-omni-video-soundtrack-generation —— A creator-facing plain-language explainer, no new primary data.

### [Goku](../models/Goku.md)

1. 【Official primary】https://arxiv.org/abs/2502.04896 — Goku paper arXiv abstract page (v1 2025-02-07, v2 2025-02-10).
2. 【Official primary】https://arxiv.org/html/2502.04896v2 — Full text HTML of the paper; Section 4, Data Curation Pipeline, is the primary basis for this survey, including Table 4's per-resolution filtering thresholds and Figure 3's semantic category distribution chart.
3. 【Official primary】https://github.com/Saiyan-World/goku — Official GitHub repository and README (the full VBench comparison table, BibTeX, author and institutional credit: HKU, ByteDance).
4. 【Official primary】https://saiyan-world.github.io/goku/ — Project homepage, generation-sample visualizations.
5. 【Third-party aggregation】https://huggingface.co/papers/2502.04896 — HuggingFace Papers page, community discussion and voting.
6. 【Third-party report】https://www.etcentric.org/bytedances-goku-video-model-is-latest-in-chinese-ai-streak/ , https://stable-learn.com/en/goku-video-model-introduction/ , https://www.analyticsvidhya.com/blog/2025/02/goku-ai/ — Media interpretations, used to cross-corroborate model scale and positioning; not relied upon for data details.

### [Hailuo / MiniMax Video](../models/Hailuo.md)

Research date: July 29, 2026. This subject has no paper and no technical report; all information comes from product blog posts, API documentation, and third-party hosting platforms, and disclosure on the data side is extremely limited.
1) Official primary: MiniMax Hailuo 02 release announcement https://www.minimax.io/news/minimax-hailuo-02 (June 18, 2025; the only source in this survey that mentions training data scale — 3x the parameter count, 4x the training data, improved quality and diversity; NCR architecture 2.5x efficiency; native 1080p; three tiers of 768p-6s / 768p-10s / 1080p-6s; #2 globally on the Artificial Analysis Video Arena)
2) Official primary: MiniMax Hailuo 2.3 / 2.3 Fast release announcement https://www.minimax.io/news/minimax-hailuo-23 (October 28, 2025; body movement, stylization including ink-wash and game-CG styles, micro-expressions, motion-instruction responsiveness; Video Agent upgraded to Media Agent; 2.3 Fast reduces batch cost by up to 50%; explicitly contains no details on resolution, duration, audio, training data, or architecture)
3) Official primary: MiniMax Open Platform video generation documentation https://platform.minimax.io/docs/guides/video-generation (model IDs: MiniMax-Hailuo-2.3 supports T2V/I2V, MiniMax-Hailuo-02 supports first/last-frame conditioning, S2V-01 supports subject reference; 1080P/6-second examples; the API contains no audio parameters whatsoever)
4) Official primary: Hailuo AI product site https://hailuoai.video/ (video/image/audio are presented as parallel, independent entry points, corroborating that audio and video are not the same model; template categories; AI-generated-content compliance notices)
5) Official primary / same-team corroboration: MiniMax HuggingFace organization page https://huggingface.co/MiniMaxAI (as of July 2026, the open-sourced models are language models — MiniMax-M3 (427B), M3-MXFP8, M2.7/M2.5/M2.1/M2 (229B), M1-40k-hf (456B), etc. — plus the VTP series visual tokenizer; no video generation model has been open-sourced, a key piece of corroboration that "the video line is entirely closed-source")
6) Same-team corroboration: VTP visual tokenizer model card https://huggingface.co/MiniMaxAI/VTP-Large-f16d64 and the paper "Towards Scalable Pre-training of Visual Tokenizers for Generation" https://arxiv.org/abs/2512.13687 (December 15, 2025, Modified MIT; a visual tokenizer jointly optimized with image-text contrastive, self-supervised, and reconstruction losses; the model card does not state any direct relationship to the Hailuo video model, and the training data is likewise not explicitly listed)
7) Third-party hosting platform: Replicate's list of MiniMax models https://replicate.com/minimax (lists seven video models — hailuo-2.3, hailuo-2.3-fast, hailuo-02, hailuo-02-fast, video-01, video-01-live, video-01-director — with run counts; key corroboration: none of the listed models describe an ability to produce an audio track alongside the video)
Note: this survey's WebSearch quota was exhausted during the session; all of the above information was obtained purely by direct fetching (WebFetch) of known official URLs, and did not cover unofficial leaks or reverse-engineering analysis from community sources such as Reddit, Zhihu, or CSDN, nor did it exhaustively search for any new version announcements that may exist from the first half of 2026. If supplementation is needed later, recommended search directions include: whether MiniMax released a video model with native audio in 2026, whether a paper on the NCR architecture has been made public, and Chinese-community (Zhihu / Jiqizhixin / Qbitai) reporting on the data sources behind Hailuo Video.

### [HunyuanVideo-Foley](../models/HunyuanVideo-Foley.md)

- 【Official primary】arXiv:2508.16930v1 "HunyuanVideo-Foley: Multimodal Diffusion with Representation Alignment for High-Fidelity Foley Audio Generation" (submitted 2025-08-23): https://arxiv.org/abs/2508.16930 , full HTML https://arxiv.org/html/2508.16930v1 , PDF https://arxiv.org/pdf/2508.16930 —— The sole direct source for the vast majority of fields in this entry, especially Section 3.1's data pipeline, Section 3.2's architecture, Section 4's experiments and ablations, and Appendix A.1 (the high-quality tagging strategy) and A.2 (evaluation metric definitions).
- 【Official primary】GitHub code repository Tencent-Hunyuan/HunyuanVideo-Foley: https://github.com/Tencent-Hunyuan/HunyuanVideo-Foley —— The open-source manifest, release timeline (first release 2025-08-28, XL version 2025-09-29), VRAM requirement table, and the list of pretrained models it depends on (SigLIP-2 / CLAP / Synchformer / DAC).
- 【Official primary】HuggingFace model page tencent/HunyuanVideo-Foley: https://huggingface.co/tencent/HunyuanVideo-Foley , original model-card text https://huggingface.co/tencent/HunyuanVideo-Foley/blob/main/README.md —— License type (tencent-hunyuan-community), EU regional restriction (extra_gated_eu_disallowed), the XXL/XL two-tier weight specifications.
- 【Official primary】Project homepage: https://szczesnys.github.io/hunyuanvideo-foley/ —— Demo examples and a method overview.
- 【Official primary】HuggingFace Papers page: https://huggingface.co/papers/2508.16930 —— Cross-checked against the abstract and the three major contributions (scalable data pipeline / representation alignment REPA / multimodal DiT).
- 【Third-party aggregation】alphaXiv paper analysis page: https://www.alphaxiv.org/abs/2508.16930 —— Used to cross-check architectural hyperparameters (18 MMDiT + 36 unimodal DiT, hidden size 1536, 12 heads), the REPA ablation table, the DAC-VAE reconstruction metrics table, and the values in the three benchmark main-results tables.
- 【Third-party deployment】Replicate hosting page tencent/hunyuanvideo-foley: https://replicate.com/tencent/hunyuanvideo-foley —— Used to confirm the shape of the inference interface (video + text → audio) and actual availability.
- 【Third-party Chinese-language report】OSCHINA news: https://www.oschina.net/news/368967 ; IT Home: https://www.ithome.com/0/878/633.htm ; AI Tool Collection entry: https://ai-bot.cn/hunyuanvideo-foley/ —— Used to cross-check the roughly 3-billion parameter count and the Chinese-language phrasing of the seven-stage pipeline, and the open-source release date. The parameter-count figure appears only in secondhand Chinese-language reports; neither the paper nor the model card states it directly.
- 【Third-party comparison】Kling-Foley paper arXiv:2506.19774 (Kuaishou's Kling, 2025-06): https://arxiv.org/pdf/2506.19774 —— Provides the Kling-Audio-Eval evaluation set, the most important contemporaneous comparison work and evaluation-benchmark source for this work.
- 【Third-party comparison】MMAudio, MovieGen-Audio-Bench (Meta Movie Gen) —— The primary baselines and evaluation benchmarks, used for lateral comparison of data strategy and performance.

### [HunyuanVideo](../models/HunyuanVideo.md)

- 【Official primary】HunyuanVideo: A Systematic Framework For Large Video Generative Models, Tencent Hunyuan, arXiv:2412.03603 (includes Section 3's data preprocessing / tiered filtering funnel, structured captioning, the 14-category shot-motion classifier, ~1M manually annotated SFT samples): https://arxiv.org/abs/2412.03603
- 【Official primary】HunyuanVideo paper HTML v1 full text (the data chapter is searchable): https://arxiv.org/html/2412.03603v1
- 【Official primary】HunyuanVideo paper HTML v2 full text: https://arxiv.org/html/2412.03603v2
- 【Official primary】HunyuanVideo 1.5 Technical Report, Tencent Hunyuan, arXiv:2511.18870, 2025-11 (includes >10M hours of raw video, 800M clips, Table 2's eight-stage training-data table, three captioning models, OPA-DPO, CT/SFT/RLHF): https://arxiv.org/abs/2511.18870
- 【Official primary】HunyuanVideo 1.5 technical report full text HTML: https://arxiv.org/html/2511.18870v1
- 【Official primary】HunyuanVideo 1.5 technical report PDF: https://arxiv.org/pdf/2511.18870
- 【Official primary】HunyuanVideo GitHub open-source repository (weights, inference code, license): https://github.com/Tencent-Hunyuan/HunyuanVideo
- 【Official primary】HunyuanVideo Hugging Face model card: https://huggingface.co/tencent/HunyuanVideo
- 【Official primary】HunyuanVideo-I2V Hugging Face model card (the image-to-video version, 2025-03): https://huggingface.co/tencent/HunyuanVideo-I2V
- 【Same-team corroboration】HunyuanVideo-Foley (Tencent's video-to-audio / Foley generation model, roughly 100,000 hours of TV2A data, indicating that Tencent's audio capability is a separate cascaded model rather than jointly generated with video): https://github.com/Tencent-Hunyuan/HunyuanVideo-Foley
- 【Third-party】alphaXiv interpretation page for the HunyuanVideo 1.5 paper: https://www.alphaxiv.org/overview/2511.18870v1
- 【Third-party】Emergent Mind: overview of the open-source video synthesis model HunyuanVideo 1.5: https://www.emergentmind.com/topics/hunyuanvideo-1-5
- 【Third-party】Hugging Face Papers: HunyuanVideo paper page and community discussion: https://huggingface.co/papers/2412.03603
- 【Third-party】ResearchGate: HunyuanVideo 1.5 Technical Report entry: https://www.researchgate.net/publication/397934115_HunyuanVideo_15_Technical_Report

### [InstructAV2AV](../models/InstructAV2AV.md)

1. 【Official primary】arXiv abstract page https://arxiv.org/abs/2605.18467 —— Title, authors, abstract, submission date of May 18, 2026.
2. 【Official primary】arXiv full text HTML https://arxiv.org/html/2605.18467v1 —— Section 3's data synthesis pipeline, method design, experiment tables, ablations, user study, and limitations. The core source of information.
3. 【Official primary】arXiv PDF https://arxiv.org/pdf/2605.18467
4. 【Official primary】Project homepage https://hjzheng.net/projects/InstructAV2AV/ —— Demos for the four editing task categories (identity-preserving voice editing / AV instance editing / instance insertion / instance removal), comparison examples against AvED, CoherentAVEdit, and AVI-Edit, and links to resources.
5. 【Official primary】GitHub repository https://github.com/suimuc/InstructAV2AV —— Apache-2.0 license, inference and pipeline scripts, checkpoints for six sub-tasks, roadmap status of training scripts, and upstream dependency statements.
6. 【Official primary】HuggingFace dataset https://huggingface.co/datasets/suimu/InsAVE-80K —— 88,074 pairs, 139 GB, 11 shards, the naming of five training subsets by category (add_and_remove / clone_id / clone_id_voice / clone_voice / general_editing), CSV field structure (original_video / target_video / instruction / instruction_reverse), the <S>/<E> speech markers, and MIT license. This is a key corroborating source that fills in the data-organization details not disclosed in the paper.
7. 【Official primary】HuggingFace model https://huggingface.co/suimu/InstructAV2AV —— Weights released, but no model card.
8. 【Third-party report】X/Twitter post by @wildmindai https://x.com/wildmindai/status/2058634841013285372 —— Popularizes it as an "Ovi + Wan2.2 + T5" tech stack with mask-free object/sound replacement capability; no new primary data.
9. 【Third-party index】The 500 Feed https://www.the500feed.com/story/e7f8eb37a6ffa09c , ai-search.io tool page https://ai-search.io/tool/instructav2av —— Aggregate introductions, no new information.
10. 【Third-party】YouTube demo video https://www.youtube.com/watch?v=0qp0B4jkWjE

### [2026 Miscellaneous Joint Audio-Video Generation](../models/JAVG_2026_misc.md)

【Official primary · Papers】
1) Baton https://arxiv.org/abs/2605.25195 ／ https://arxiv.org/html/2605.25195v2 (data information is concentrated in the Implementation Details section)
2) OmniCustom https://arxiv.org/abs/2602.12304 ／ https://arxiv.org/html/2602.12304 (the OmniCustom-1M dataset-construction chapter is one of the most valuable data disclosures for this entry)
3) StreamChar https://arxiv.org/abs/2605.25659 ／ https://arxiv.org/html/2605.25659v1
4) ALIVE https://arxiv.org/abs/2602.08682 ／ https://arxiv.org/html/2602.08682v2 (the Data chapter's six-stage pipeline is the most thoroughly disclosed data section among the seven items)
5) CCL https://arxiv.org/abs/2603.18600 ／ https://arxiv.org/html/2603.18600v1 (Table 1 provides a lateral comparison of training-data volume)
6) NAVA https://arxiv.org/abs/2605.30073 ／ https://arxiv.org/pdf/2605.30073 (the Data chapter includes a 20M/100M→15M funnel and an operator list)
7) ITS-JAVG https://arxiv.org/abs/2606.03183 ／ https://arxiv.org/html/2606.03183v1
【Official primary · Project pages/repositories】
8) NAVA project page https://ernie-research.github.io/NAVA/ ; code https://github.com/ernie-research/NAVA ; weights https://huggingface.co/baidu/NAVA and https://huggingface.co/ernie-research/NAVA (scope of open-sourcing, license, resolution/duration, language support)
9) ALIVE repository https://github.com/FoundationVision/Alive ; project page foundationvision.github.io (model scale: VideoDiT 12B / AudioDiT 2B, Alive-Bench 1.0)
10) StreamChar project page https://humanaigc.github.io/StreamChar_page/
11) OmniCustom project page and GitHub (given in the paper)
【Same-team corroboration / aggregation pages】
12) HuggingFace Papers: https://huggingface.co/papers/2605.30073 (NAVA), https://huggingface.co/papers/2605.25659 (StreamChar)
13) ResearchGate CCL entry https://www.researchgate.net/publication/402860562_Improving_Joint_Audio-Video_Generation_with_Cross-Modal_Context_Learning
【Third-party】
14) ChatPaper's StreamChar interpretation https://chatpaper.com/paper/286175 ; Takara TLDR of CCL https://tldr.takara.ai/p/2603.18600 ; X/Twitter CS Vision Papers' promotion of NAVA (all restatements of the paper abstracts, no new primary information)
【Primary sources for upstream datasets (reused by these works)】
15) SpeakerVid-5M (a shared upstream for OmniCustom and StreamChar), TalkVid, OpenHumanVid (shared by Baton/CCL/StreamChar), Koala-36M (roughly 20% of NAVA), AudioCaps, WavCaps, VGGSound, Emilia (StreamChar's speech pretraining)

### [Joint Audio-Video Generation Baseline Collection](../models/JavisDiT_baselines.md)

1) Official primary｜JavisDiT paper https://arxiv.org/abs/2503.23377 (ICLR 2026; includes the construction of JavisBench/JavisScore and the three-stage training data scale)
2) Official primary｜JavisDiT++ paper https://arxiv.org/abs/2602.19163 and HTML version https://arxiv.org/html/2602.19163v1 (data filtering strategy, the TAVGBench 1.1M→355K funnel, AV-DPO preference-data construction, the data-quality ablation in Appendix D.2)
3) Official primary｜JavisDiT GitHub https://github.com/JavisVerse/JavisDiT and https://github.com/JavisDiT/JavisDiT (scope of open-sourcing, data copyright statement)
4) Official primary｜JavisDiT data-preparation documentation https://raw.githubusercontent.com/JavisDiT/JavisDiT/main/assets/docs/data.md (the three-stage data CSV schema, 30-second audio truncation, 16kHz resampling, 16fps video normalization, a minimum 10-frame filter, DPO 1+3 candidate construction)
5) Official primary｜JavisDiT++ project homepage https://javisverse.github.io/JavisDiT2-page/
6) Official primary｜OpenReview review page https://openreview.net/forum?id=hRRWfFpKRp
7) Official primary｜MM-Diffusion paper https://arxiv.org/abs/2212.09478 and the CVPR 2023 version https://openaccess.thecvf.com/content/CVPR2023/html/Ruan_MM-Diffusion_Learning_Multi-Modal_Diffusion_Models_for_Joint_Audio_and_Video_CVPR_2023_paper.html
8) Official primary｜MM-Diffusion GitHub https://github.com/researchmm/MM-Diffusion (MIT license, data download, dual-scale 64×64 and 256×256 super-resolution)
9) Official primary｜AV-DiT paper https://arxiv.org/abs/2406.07686 and HTML version https://arxiv.org/html/2406.07686v1 (16 frames at 256×256, 1.6-second 16kHz audio, mel spectrogram at 40×16×8)
10) Official primary｜AV-DiT OpenReview https://openreview.net/forum?id=FE6zflN5G5
11) Official primary｜Harmony paper https://arxiv.org/abs/2511.21579 and HTML version https://arxiv.org/html/2511.21579v1 (a corpus of 4 million clips, Gemini automatic annotation, three-stage training, Harmony-Bench)
12) Official primary｜UniAVGen paper https://arxiv.org/abs/2511.03334 and HTML version https://arxiv.org/html/2511.03334v1 (the Emilia English subset, in-house real human audio-video data, a 1.3M vs. 30.1M comparison, three-stage training hyperparameters)
13) Same-team corroboration｜HuggingFace Papers pages https://huggingface.co/papers/2602.19163 and https://huggingface.co/papers/2406.07686
14) Third-party corroboration｜MM-LDM paper https://arxiv.org/pdf/2410.01594 , MMDisCo https://arxiv.org/pdf/2405.17842 , UniForm https://arxiv.org/pdf/2502.03897 (all restate the statistical figures for Landscape/AIST++, usable for cross-validation)

### [Kling 3.0 / Kling Video 3.0 Omni](../models/Kling_30_Omni.md)

- Official primary: Kling-Omni Technical Report (arXiv:2512.16776, the three-tier data system / training stages / DPO) https://arxiv.org/abs/2512.16776
- Same-team corroboration: Koala-36M (arXiv:2410.08260, the quantified funnel / CSS splitting / VTSS / structured captioning) https://arxiv.org/abs/2410.08260 and https://github.com/KwaiVGI/Koala-36M
- Same-team corroboration: Kling-Foley (arXiv:2506.19774, the audio data pipeline / VAD threshold of 0.2 / CLAP filtering) https://arxiv.org/abs/2506.19774
- Same-team corroboration: KlingAvatar 2.0 (arXiv:2512.13313, multi-speaker data sources and automated annotation pipeline) https://arxiv.org/abs/2512.13313
- Same-team corroboration: Kling-MotionControl Technical Report (arXiv:2603.03160) https://arxiv.org/abs/2603.03160
- Third-party report: People's Daily Online / Economic Information Daily's press release on the 2026-02-05 launch of the Kling 3.0 series http://finance.people.com.cn/n1/2026/0205/c1004-40660255.html
- Official primary: Kling Video 3.0 Omni user guide https://kling.ai/quickstart/klingai-video-3-omni-model-user-guide
- Official primary: Replicate's official hosted-API specification page https://replicate.com/kwaivgi/kling-v3-video
- Third-party report: 302.AI Benchmark Lab's hands-on evaluation of Kling O3 https://zhuanlan.zhihu.com/p/2004988563100566106

### [LTX-2](../models/LTX-2.md)

- 【Official primary】LTX-2: Efficient Joint Audio-Visual Foundation Model, Yoav HaCohen et al., Lightricks, arXiv:2601.03233v1, 2026-01-06 (Section 5, Training Data, and 5.1, Captioning, are the entirety of the primary data-side information): https://arxiv.org/abs/2601.03233
- 【Official primary】LTX-2 paper PDF: https://arxiv.org/pdf/2601.03233
- 【Official primary】LTX-2 paper HTML version: https://arxiv.org/html/2601.03233v1
- 【Official primary · same-team corroboration】LTX-Video: Realtime Video Latent Diffusion, arXiv:2501.00103 (Section 3, Data Preparation, includes a 9-stage cleaning pipeline diagram, the Siamese aesthetic model, CLIP deduplication, motion filtering, re-captioning, plus Fig.13's word cloud and Fig.14's caption word-count and clip-duration distributions — the LTX-2 dataset is a subset of this): https://arxiv.org/abs/2501.00103
- 【Official primary】LTX-2 official GitHub repository (ltx-core / ltx-pipelines / ltx-trainer, LoRA, license): https://github.com/Lightricks/LTX-2
- 【Official primary】LTX-2 license text: https://github.com/Lightricks/LTX-2/blob/main/LICENSE
- 【Official primary】Hugging Face model card Lightricks/LTX-2 (variants, quantized versions, distilled versions, upsampler, Limitations): https://huggingface.co/Lightricks/LTX-2
- 【Official primary】Hugging Face Lightricks/LTX-2.3 (the 22B upgraded version): https://huggingface.co/Lightricks/LTX-2.3
- 【Official primary】Lightricks open-sources LTX-2 press release, GlobeNewswire, 2026-01-06 (4K, open weights and training code, $10M ARR licensing threshold): https://www.globenewswire.com/news-release/2026/01/06/3213304/0/en/Lightricks-Open-Sources-LTX-2-the-First-Production-Ready-Audio-and-Video-Generation-Model-With-Truly-Open-Weights.html
- 【Official primary】LTX official newsroom: LTX-2 full-weight open-source announcement: https://ltx.io/newsroom/ltx-2-is-now-open-source-full-model-weights-released
- 【Official primary】LTX-2 official prompting guide (the prompt structure for audio/dialogue/accent/foley/ambience, isomorphic to the training captions): https://ltx.io/model/model-blog/prompting-guide-for-ltx-2
- 【Official primary】LTX-2.3 official prompt guide: https://ltx.io/model/model-blog/ltx-2-3-prompt-guide
- 【Official primary】LTX official documentation Prompting Guide: https://docs.ltx.io/api-documentation/implementation-guides/prompting-guide
- 【Official primary · compliance corroboration】Press release on the Lightricks-Shutterstock video training-data partnership, PR Newswire, 2024-12 (the first global partner adopting a research license): https://www.prnewswire.com/news-releases/lightricks-partners-with-shutterstock-for-video-training-data-to-advance-open-source-ltxv-video-ai-generative-video-model-302331526.html
- 【Official primary · compliance corroboration】Shutterstock investor relations page carrying the same press release: https://investor.shutterstock.com/news-releases/news-release-details/lightricks-partners-shutterstock-video-training-data-advance
- 【Official primary】Press release on Lightricks' launch of the 13B LTX Video model, PR Newswire, 2025-05 (mentions the Getty Images strategic partnership): https://www.prnewswire.com/news-releases/lightricks-launches-13b-parameters-ltx-video-model-breakthrough-rendering-approach-generates-high-quality-efficient-ai-video-30x-faster-than-comparable-models-302447660.html
- 【Third-party report】Wikipedia: LTX-2 (timeline, specifications, leaderboard performance, limitations): https://en.wikipedia.org/wiki/LTX-2
- 【Third-party report】Report on the Lightricks 13B LTX Video and Getty partnership, Dataconomy, 2025-05: https://dataconomy.com/2025/05/14/lightricks-unveils-13b-ltx-video-model-for-hq-ai-video-generation/
- 【Third-party report】Lightricks-Shutterstock partnership and AI training-data licensing standards, Metaverse Post: https://mpost.io/lightricks-shutterstock-partnership-sets-new-standards-for-ai-training-data-licensing/
- 【Third-party report】LTX-2's native 4K and fully open weights, Open Source For You, 2026-01: https://www.opensourceforu.com/2026/01/ltx-2-from-lightricks-delivers-native-4k-audio-video-with-fully-open-weights/
- 【Third-party report】LTX-2 technical interpretation, Introl Blog, 2026: https://introl.com/blog/ltx-2-audiovisual-diffusion-synchronized-video-audio-2026
- 【Third-party compilation】awesome-ltx2 prompt guide (dialogue quoting, language/accent, foley/ambience/music description conventions): https://github.com/wildminder/awesome-ltx2/blob/main/LTX2-prompt-guide.md
- 【Third-party report】Interpretation of the LTX-2 license and scope of commercial use, WaveSpeed Blog: https://wavespeed.ai/blog/posts/blog-ltx-2-license-commercial-use/
- 【Third-party report】Highlights of the LTX-2.3 update (released 2026-03-05, 22B, 4K/50fps/20 seconds), WaveSpeed Blog: https://wavespeed.ai/blog/posts/ltx-2-3-whats-new-2026/
- 【Related work for comparison】Ovi: Twin Backbone Cross-Modal Fusion for Audio-Video Generation, arXiv:2510.01284 (LTX-2's primary open-source comparison target): https://arxiv.org/abs/2510.01284
- 【Related work for comparison】Veo 3 technical report (LTX-2's closed-source comparison target): https://storage.googleapis.com/deepmind-media/veo/Veo-3-Tech-Report.pdf

### [LongCat-Video](../models/LongCat-Video.md)

1. arXiv:2510.22200 "LongCat-Video Technical Report" https://arxiv.org/abs/2510.22200 and full text https://arxiv.org/html/2510.22200v2 —— Official primary (the core basis; the data-processing chapter, the training-stage table, and the RLHF details are all taken from here).
2. Official GitHub repository https://github.com/meituan-longcat/LongCat-Video —— Official primary (scope of open-sourcing, MIT license, model variants and release timeline).
3. Hugging Face model card https://huggingface.co/meituan-longcat/LongCat-Video —— Official primary (license, capability description, MOS scores).
4. arXiv:2605.26486 "LongCat-Video-Avatar 1.5 Technical Report" https://arxiv.org/abs/2605.26486 —— Official primary (same-team corroboration; the audio-video alignment / audio filtering / emotion-and-silence data curation details are mostly drawn from this paper).
5. Meituan technical team blog https://tech.meituan.com/2025/12/23/longcat-video-avatar.html and https://tech.meituan.com/tags/longcat.html —— Official primary (the Chinese-language release messaging).
6. Meituan press release "LongCat-Video anchors on long video to build the technical foundation for world models" https://www.meituan.com/news/NN251205166001020 —— Official primary (positioning and the world-model narrative).
7. 36Kr report https://www.36kr.com/p/3527169453464452 , Zhihu interpretation https://zhuanlan.zhihu.com/p/1966806796392966062 —— Third-party report (corroboration of the release date and the inference-speedup multiplier, etc.).
8. HyperAI paper page https://hyper.ai/en/papers/2605.26486 —— Third-party index.

### [MOVA](../models/MOVA.md)

- 【Official primary】arXiv:2602.08794v2 "MOVA: Towards Scalable and Synchronized Video–Audio Generation" technical report (38 pages, February 10, 2026): https://arxiv.org/abs/2602.08794 , PDF https://arxiv.org/pdf/2602.08794 —— The sole and direct source for the vast majority of fields in this entry, especially Section 3 (Data Engineering), Section 4.3 (Progressive Joint Training), and Appendices A.1/A.3/A.4/A.5/A.6. Note that arXiv provides no HTML version (/html/2602.08794v2 returns "No HTML"), so the PDF must be parsed directly.
- 【Official primary】GitHub code repository OpenMOSS/MOVA: https://github.com/OpenMOSS/MOVA —— Manifest of open-sourced content, Apache-2.0 license, release timeline (first release 2026-01-29 / technical report and inference workflow 2026-02-10 / API 2026-03-09 / evaluation code 2026-05-06), dataset interface mova/datasets/video_audio_dataset.py.
- 【Official primary】HuggingFace model pages: https://huggingface.co/OpenMOSS-Team/MOVA-720p , https://huggingface.co/OpenMOSS-Team/MOVA-360p , collection https://huggingface.co/collections/OpenMOSS-Team/mova ; paper page https://huggingface.co/papers/2602.08794
- 【Official primary】Project blog: https://mosi.cn/models/mova
- 【Third-party report】ComfyUI-Wiki news item "OpenMOSS Releases MOVA - Open-Source Synchronized Video-Audio Generation" (2026-01-29): https://comfyui-wiki.com/en/news/2026-01-29-openmoss-mova-video-audio-generation —— Used to cross-check the first-release date.
- 【Third-party report】AI Films Studio blog post "MOVA: Open Source Video-Audio Generation": https://studio.aifilms.ai/blog/mova-open-source-video-generation —— Used to cross-check the description of the Apache-2.0 commercial-use license.

### [Merged entry: ① Mochi 1](../models/Mochi_MAGI_Motif.md)

1. https://arxiv.org/abs/2604.16503 —— Official primary. "Motif-Video 2B: Technical Report," Motif Technologies, v2 updated 2026-05-19. The source of all quantitative information in this entry's Motif section (Section 4's training curriculum Table 1, Section 5's complete data pipeline, Table 2's sampler utilization, Table 3's 16-dimension VBench results).
2. https://arxiv.org/html/2604.16503 —— Official primary. Full HTML text of the Motif technical report (this survey fetched and locally parsed the full text; Section 5.1's data processing pipeline, 5.2's video captioning, and 5.3's offline bucket-balanced sampler were checked section by section).
3. https://huggingface.co/Motif-Technologies/Motif-Video-2B —— Official primary. Motif-Video 2B weights card, Apache 2.0, released 2026-04-14, states roughly 10 million annotated video clips, <100,000 H200 GPU hours, and ≈90% data-utilization rate.
4. https://arxiv.org/abs/2505.13211 —— Official primary. "MAGI-1: Autoregressive Video Generation at Scale," Sand AI, 2025-05-19, 61 pages.
5. https://arxiv.org/html/2505.13211v1 —— Official primary. Full HTML text of the MAGI-1 technical report (this survey fetched and locally parsed the full text; Section 3, DATA, subsections 3.1 Filter Actors / 3.2 De-duplication / 3.3 MLLM as Advanced Filter / 3.4 Caption / 3.5 Data Adjustment, Table 3's caption attribute table, Table 4's caption examples, and Table 5's three-stage data configuration were checked section by section).
6. https://github.com/SandAI-org/MAGI-1 —— Official primary. MAGI-1 code repository and README, including the Apache 2.0 license, the 24B/4.5B/Distill/FP8 variant lineup, and release timeline (2025-04-21 through MAGI-1.1 on 2026-06-17).
7. https://www.genmo.ai/blog/mochi-1-a-new-sota-in-open-text-to-video —— Official primary. Genmo's official blog post "Mochi 1: A new SOTA in open text-to-video," covering the AsymmDiT architecture, the AsymmVAE's 96× compression, the 44,520-token context, and use of Gemini-1.5-Pro-002 as an automated prompt-adherence judge, etc.; the full text contains no training-data chapter.
8. https://huggingface.co/genmo/mochi-1-preview —— Official primary. Mochi 1 model card (48-layer AsymmDiT, 362M-parameter AsymmVAE, 480p/84 frames, Apache 2.0, roughly 60GB VRAM, NSFW filtering already applied, an acknowledgment that the model reflects biases in its training data).
9. https://github.com/genmoai/mochi —— Official primary. Mochi 1 code repository.
10. https://venturebeat.com/ai/video-ai-startup-genmo-launches-mochi-1-an-open-source-model-to-rival-runway-kling-and-others —— Third-party report (VentureBeat, 2024-10-22), including a direct quote from Paras Jain about the training data — "Generally, we use publicly available data and sometimes work with a variety of data partners" — and his refusal to elaborate; the sole primary-source statement on Mochi 1's data provenance.
11. https://siliconangle.com/2024/10/22/genmo-introduces-mochi-1-open-source-text-video-generation-model/ —— Third-party report, Mochi 1 release report and specification cross-check.
12. https://www.oschina.net/news/346129/sand-ai-magi1 —— Third-party report (OSCHINA, Chinese-language), background on Sand AI and the Cao Yue team, corroborating the MAGI-1 release information.
13. https://huggingface.co/papers/2505.13211 —— Third-party aggregation, MAGI-1 paper page and community discussion.

### [Movie Gen](../models/Movie_Gen.md)

- Official primary: Movie Gen: A Cast of Media Foundation Models (arXiv:2410.13720, 92-page full text including Appendix J.1's threshold table) https://arxiv.org/html/2410.13720v1
- Official primary: facebookresearch/MovieGenBench (open-source evaluation benchmark) https://github.com/facebookresearch/MovieGenBench

### [NVIDIA NeMo Curator](../models/NeMo_Curator.md)

- 【Official primary】NeMo Curator GitHub main repository (Apache 2.0, four-modality capability, Ray/Xenna architecture, performance claims, installation method): https://github.com/NVIDIA-NeMo/Curator
- 【Official primary】NeMo Curator Releases page (highlights of each version, including 26.02/26.04/26.07): https://github.com/NVIDIA-NeMo/Curator/releases
- 【Official primary】PyPI nemo-curator release history (precise release dates: 1.0.0=2025-10-01, 1.1.0/26.02=2026-02-23, 1.2.0/26.04=2026-05-14, 1.3.0/26.07=2026-07-27): https://pypi.org/project/nemo-curator/
- 【Official primary】NeMo Curator video curation overview documentation (each stage and the models used: TransNetV2, Qwen-VL, Cosmos-Embed1, CLIP aesthetic, NVENC/NVDEC, semantic deduplication, WebDataset): https://docs.nvidia.com/nemo/curator/curate-video
- 【Official primary】NeMo Curator video filtering documentation (the algorithms for motion filtering and aesthetic filtering, stage names, and all default thresholds and parameters): https://docs.nvidia.com/nemo/curator/curate-video/process-data/filtering
- 【Official primary】NeMo Curator video architecture documentation (Ray Actor management, the Ray Object Store, the Cosmos-Xenna executor, autoscaling, and all default configuration items): https://docs.nvidia.com/nemo/curator/about/concepts/video/architecture.md
- 【Official primary】NeMo Curator audio curation documentation (ASR transcription, WER/CER filtering, speaker separation; confirms no coupling with video): https://docs.nvidia.com/nemo/curator/curate-audio
- 【Official primary】NeMo Curator 26.07 release notes / migration checklist (audio-enhancement stage, Nemotron-CLIMB, the captioning-backend matrix, breaking changes): https://docs.nvidia.com/nemo/curator/about/release-notes
- 【Official primary】NeMo Curator 26.02 video quickstart documentation: https://docs.nvidia.com/nemo/curator/26.02/get-started/video.html
- 【Official primary】NeMo Curator 26.02 transcoding / clip-encoding documentation: https://docs.nvidia.com/nemo/curator/26.02/curate-video/process-data/transcoding.html
- 【Official primary】Cosmos-Xenna GitHub repository README (a Ray-based distributed data-pipeline library, three modes — streaming/batch/serving —, autoscaling and bin-packing, backpressure, SPMD, P2P distribution, Apache 2.0, a note that the repository is no longer under active development): https://github.com/nvidia-cosmos/cosmos-xenna
- 【Official primary】Cosmos-Curate GitHub repository README (the Cosmos training-data generation system, built atop Cosmos-Xenna, code under Apache 2.0 / model under the NVIDIA Open Model License): https://github.com/nvidia-cosmos/cosmos-curate
- 【Official primary · core methodology】Cosmos World Foundation Model Platform for Physical AI, NVIDIA, arXiv:2501.03575 (Section 3 fully discloses the seven-tier curation pipeline: TransNetV2 selection with an F1=0.967 comparative benchmark, NVDEC/NVENC 6.5x, the ViT optical-flow motion classifier, DOVER bottom-15% cut, an aesthetic threshold of 3.5, InternVideo2+MLP overlay-text and genre classification, VILA 13B FP8 TensorRT-LLM 10x, captions averaging 559 characters / 97 words, k-means with k=10000 deduplication removing 30%, the nine major category-mixture figures, 20M hours→100M clips): https://arxiv.org/abs/2501.03575
- 【Official primary】Training Video Foundation Models with NVIDIA NeMo, arXiv:2503.12964 (the dual clipping/sharding pipeline structure, the 100PB+ scale figure, NVDEC/NVENC 3x, captioning as the bottleneck stage, Ray auto-balancing of workers): https://arxiv.org/abs/2503.12964
- 【Official primary · reprint】Accelerate Custom Video Foundation Model Pipelines with New NVIDIA NeMo Framework Capabilities (NVIDIA official blog post; the original source of the 89x speedup figure — "1K GPUs, ISO power, versus an unoptimized CPU pipeline"; 20M hours; 100PB+; heterogeneous L40S/H100/GB200 clusters): https://www.edge-ai-vision.com/2025/01/accelerate-custom-video-foundation-model-pipelines-with-new-nvidia-nemo-framework-capabilities/
- 【Official primary】Advancing Physical AI with NVIDIA Cosmos World Foundation Model Platform (NVIDIA technical blog post; 20 million hours processed in 40 days on Hopper / 14 days on Blackwell / 3.4 years on CPU): https://developer.nvidia.com/blog/advancing-physical-ai-with-nvidia-cosmos-world-foundation-model-platform/
- 【Official primary】World Simulation with Video Foundation Models for Physical AI (the Cosmos follow-up paper arXiv:2511.00062): https://arxiv.org/pdf/2511.00062
- 【Official primary】NVIDIA Nemotron 3 Nano Omni technical blog post (background on Curator's introduction of an all-modality model in 26.07): https://developer.nvidia.com/blog/nvidia-nemotron-3-nano-omni-powers-multimodal-agent-reasoning-in-a-single-efficient-open-model/
- 【Official primary】NeMo Curator video-splitting example code tutorials/video/getting-started/video_split_clip_example.py: https://github.com/NVIDIA-NeMo/Curator/blob/main/tutorials/video/getting-started/video_split_clip_example.py
- 【Official primary】Cosmos Curator video pipeline reference documentation docs/curator/reference/video-pipelines.md: https://github.com/NVIDIA/cosmos-curator/blob/main/docs/curator/reference/video-pipelines.md
- 【Third-party report】Architecting Data Pipelines for Multimodal Datasets at Scale (Anyscale blog post, a Ray-centric perspective on multimodal data-pipeline architecture): https://www.anyscale.com/blog/architecting-multimodal-data-pipelines-that-scale-with-ray

### [OmniHuman Dataset + OHBench](../models/OmniHuman.md)

- 【Official primary】arXiv:2604.18326 "OmniHuman: A Large-scale Dataset and Benchmark for Human-Centric Video Generation" —— v1 submitted 2026-04-20, v2 submitted 2026-05-30, cs.CV, 19 pages with 6 figures. Abstract page https://arxiv.org/abs/2604.18326 , full HTML (v2) https://arxiv.org/html/2604.18326v2 , PDF https://arxiv.org/pdf/2604.18326 . The sole direct source for the vast majority of fields in this entry.
- 【Official primary】GitHub code repository https://github.com/julia-cherry/OmniHuman —— The OHBench evaluation toolkit, including per-dimension evaluation scripts, configuration, dependencies, and a model-checkpoint index. Note that the paper body once contained the (non-hyphenated) spelling github.com/juliacherry/OmniHuman, which returns a 404; the correct address is the hyphenated version.
- 【Official primary】HuggingFace dataset https://huggingface.co/datasets/julia527/omnihuman —— Roughly 62.7GB of sharded tar files, including sample_json (per-sample audio-visual annotations), metadata (a JSONL index), and tracking_npz (per-frame SMPL/MANO tracking); does not include raw video, only YouTube URLs plus clip start/end timestamps.
- 【Official primary】HuggingFace evaluation-asset repository julia527/omnihuman_benchmark —— The third-party discriminator model checkpoints (.pt/.onnx) required by OHBench.
- 【Comparison-work corroboration】OpenHumanVid (arXiv:2412.00115, Fudan University, CVPR 2025), SpeakerVid-5M, CelebV-HQ, HDTF, VoxCeleb2, TikTok-v4, ActivityNet —— The comparison datasets in the paper's Table 1, used for cross-understanding of OmniHuman's incremental additions along the annotation dimensions (see benchmark_taxonomy_alignment).
- 【Evaluation-subject corroboration】The original papers for open-source joint AV generation models evaluated on the benchmark, such as LTX-2, MOVA, Ovi, UniVerse-1, and UniAVGen —— Used to understand the context of the OHBench evaluation results; LTX-2 is simultaneously the fine-tuning base for this work's data-validity verification experiments.
- 【Toolchain corroboration】TransNetV2 (shot detection), Demucs (four-source separation), DOVER (video quality), UniMatch (optical flow), SigLIP (semantic similarity), YOLOv11 + MOTRv2 (detection and multi-object tracking), DWPose-L (134 keypoints), 3D-Speaker (speaker separation), SyncNet (lip-audio sync), ArcFace (face identity), FunASR-Nano (ASR), Qwen3-Omni (all-modality annotation), Gemini-3 / Gemini-3-pro (semantic adjudication), MUSIQ, RAFT, ImageBind, CLAP, Audiobox-Aesthetics, SenseVoice, DNSMOS —— The original papers and repositories for each, used to verify the actual function and output form of each tool.

### [Open-Sora Series](../models/Open-Sora.md)

【Official primary】1) Open-Sora 2.0 technical report arXiv:2503.09642 https://arxiv.org/abs/2503.09642 and full HTML https://arxiv.org/html/2503.09642v1 (the data pipeline, data statistics charts, three-stage cost table); 2) Open-Sora 1.2 paper arXiv:2412.20404 https://ar5iv.labs.arxiv.org/html/2412.20404 (data sources, the 80k-hour figure, the bucketing strategy, 35k H100-GPU-hours); 3) Open-Sora main GitHub repository https://github.com/hpcaitech/Open-Sora (version timeline, scope of open-sourcing); 4) Open-Sora v1.2.0 data-processing code and documentation (primary code-level evidence): docs/data_processing.md, tools/scoring/README.md, tools/scene_cut/README.md, tools/caption/README.md, raw paths of the form https://raw.githubusercontent.com/hpcaitech/Open-Sora/v1.2.0/tools/scoring/README.md ; 5) Open-Sora Plan paper arXiv:2412.00131 https://arxiv.org/html/2412.00131v1 (a data-sources table, a seven-tier filtering funnel with per-tier retention-rate table, a training-stage table); 6) Open-Sora Plan GitHub https://github.com/PKU-YuanGroup/Open-Sora-Plan plus docs/Report-v1.3.0.md and docs/Report-v1.5.0.md (thresholds, the 27% retention rate, prompt-refiner details, v1.5 data scale).
【Third-party report】7) MarkTechPost's report on the Open-Sora 2.0 release https://www.marktechpost.com/2025/03/14/hpc-ai-tech-releases-open-sora-2-0-an-open-source-sota-level-video-generation-model-trained-for-just-200k/ ; 8) HuggingFace Papers pages https://huggingface.co/papers/2503.09642 and https://huggingface.co/papers/2412.00131 ; 9) comfyui-wiki release brief https://comfyui-wiki.com/en/news/2025-03-13-open-sora-2-release .

### [Ovi](../models/Ovi.md)

1) Official primary｜Full paper https://arxiv.org/abs/2510.01284 and https://arxiv.org/pdf/2510.01284 ("Ovi: Twin Backbone Cross-Modal Fusion for Audio-Video Generation"; Section 3, Data Processing Pipeline, is the sole authoritative disclosure on the data side)
2) Official primary｜GitHub repository and README https://github.com/character-ai/Ovi (scope of open-sourcing, Todo List, the Ovi 1.1 update notes, prompt format)
3) Official primary｜HuggingFace model card https://huggingface.co/chetwinlow1/Ovi (parameter count, base model, license, resolution/duration versions)
4) Official primary｜Project homepage https://aaxwaz.github.io/Ovi/ (demo and code entry points)
5) Same-team corroboration｜HuggingFace Papers page https://huggingface.co/papers/2510.01284
6) Third-party report｜CSDN Chinese-language interpretation https://blog.csdn.net/SuaniCommunity/article/details/154737163 and Tencent News https://view.inews.qq.com/a/20251113A02WA900 (restate the paper's four-step data pipeline and the 720×720 threshold; no new primary information)
7) Third-party corroboration｜Tencent Cloud Developer Community technical explainer https://cloud.tencent.cn/developer/article/2584843
8) Third-party platform｜WaveSpeed AI hosting page https://wavespeed.ai/models/character-ai/ovi/text-to-video (commercial availability)

### [Script-a-Video](../models/Script-a-Video.md)

- 【Official primary】arXiv abstract page https://arxiv.org/abs/2604.11244 —— Paper title, institution (Tencent Hunyuan Team), core figures from the abstract (a 25% average reduction in total error rate on Video-SALMONN-2, a 67% average improvement on Daily-Omni, and human-evaluated gains of +45% identity consistency / +56% audio-video alignment / +71% temporal controllability for multi-shot generation).
- 【Official primary】arXiv full text HTML v2 https://arxiv.org/html/2604.11244v2 (dated 2026-04-15, cs.CV) —— The sole direct source for the vast majority of fields in this entry: Section 3's complete field definitions for the MTSS four-stream schema, Section 4.1's caption-dataset construction with Tables 1/2, Section 4.2's generation experiments with Tables 3/4, Section 4.2.2's training details, and the Limitations section.
- 【Official primary】arXiv full text HTML v1 https://arxiv.org/html/2604.11244v1 —— Used to cross-check against v2 (the v1 abstract states a 67% improvement on Daily-Omni, whereas v2's Figure 1 and body text give figures of 110% and 127% respectively under different statistical framings, indicating discrepancies in the numbers across versions / accounting methods).
- 【Official primary】arXiv PDF https://arxiv.org/pdf/2604.11244 —— The chart layout and Figure 3's MTSS script example.
- 【Third-party corroboration】Tencent Hunyuan GitHub organization https://github.com/Tencent-Hunyuan —— Used to verify whether an accompanying open-source repository exists; as of the research date this organization hosts repositories such as HunyuanVideo, HunyuanVideo-1.5, HunyuanVideo-Avatar, HunyuanVideo-Foley, and HunyuanVideo-I2V, but no Script-a-Video / MTSS-related repository, from which it is judged that this work has not been open-sourced.
- 【Note】No third-party media reports, technical-blog interpretations, or community reproduction discussions were found; this work was newly released at the time of research and has limited reach so far.

### [Seedance 2.0 and Seedance 1.5 pro](../models/Seedance_20_Seedance_15_pro.md)

- Official primary: Seedance 2.0 report (arXiv:2604.14148, 26 pages, only introduction + evaluation + contributors, no data chapter) https://arxiv.org/abs/2604.14148
- Official primary: Seedance 1.5 pro report (arXiv:2512.13507, all of v1-v3 checked, no data chapter) https://arxiv.org/abs/2512.13507
- Official primary: Seedance 1.0 technical report (arXiv:2506.09113, contains a complete Data chapter; the verifiable baseline for this entry's data pipeline) https://arxiv.org/abs/2506.09113
- Debunking note: the "100 million minutes of training data / four-stage pipeline" claim circulating on AI-summary sites such as Emergent Mind returned zero hits across a full-text search of all three PDF versions, and is judged to be a hallucination; not adopted.

### [SkyReels Series](../models/SkyReels.md)

- Official primary | https://arxiv.org/abs/2602.21818 — SkyReels-V4 technical report (submitted 2026-02-24, v3 2026-03-18), including data collection/processing/annotation, a six-stage training table, the SyncNet threshold, SkyReels-VABench
- Official primary | https://arxiv.org/html/2602.21818v3 — V4 paper full HTML (the data chapter, Table 1's training-stage table)
- Official primary | https://arxiv.org/abs/2504.13074 — SkyReels-V2 technical report (2025-04-17), including the data pipeline, SkyCaptioner-V1, multi-stage training, and RL
- Official primary | https://github.com/SkyworkAI/SkyReels-V2 — V2 official repository (weights list, SkyCaptioner-V1, release timeline)
- Official primary | https://huggingface.co/Skywork/SkyCaptioner-V1 — SkyCaptioner-V1 model card (Qwen2.5-VL-7B-Instruct base, 10 million→2 million concept-balanced data, 32×A800, accuracy)
- Official primary | https://github.com/SkyworkAI/SkyReels-V3 — V3 official repository (open-sourced 2026-01-29)
- Third-party interpretation | https://blog.csdn.net/Together_CZ/article/details/148583114 — CSDN's detailed Chinese-language restatement of V2's data chapter (O(100M) scale, 280,000 movies / 800,000 TV episodes / 6.2 million hours, the list of filters, the manual spot-check rate, the DPO data volume)
- Third-party interpretation | https://www.alphaxiv.org/overview/2504.13074 — alphaXiv's structured interpretation of V2 (100 million video samples, PyDetect+TransNet-V2, Qwen2.5-VL-7B, 30k sample pairs)
- Third-party interpretation | https://www.alphaxiv.org/overview/2602.21818 — alphaXiv's interpretation of V4 (VABench's five dimensions, bidirectional cross-attention, the Refiner)
- Third-party interpretation | https://www.emergentmind.com/papers/2504.13074 — A summary of V2's data pipeline and training curriculum (dual-axis bucketing, the four-stage post-training)
- Third-party report | https://finance.sina.com.cn/tech/roll/2026-01-30/doc-inhkakfi6974616.shtml — Sina Finance: report on the SkyReels-V3 open-source release
- Third-party report | https://tech.cnr.cn/techgd/20260328/t20260328_527565454.shtml — CNR: Kunlun Wanwei's release of SkyReels-V4 at the 2026 Zhongguancun Forum
- Third-party report | https://wavespeed.ai/blog/posts/what-is-skyreels-v4/ — V4 capabilities and a note on its status: "weights not yet released, limited preview"
- Third-party report | https://comfyui-wiki.com/en/news/2025-04-21-skyreels-v2-infinite-length-film-generative-model — V2's release and its overall VBench score of 83.9%

### [Sora 2](../models/Sora_2.md)

- Sora 2 System Card, OpenAI, 2025-09-30 (PDF): https://cdn.openai.com/pdf/50d5973c-c4ff-4c2d-986f-c72b5d0ff069/sora_2_system_card.pdf
- Sora 2 System Card index page: https://openai.com/index/sora-2-system-card/
- OpenAI Deployment Safety Hub - Sora 2: https://deploymentsafety.openai.com/sora-2
- Sora System Card (Sora 1, including CSAM safety-stack details): https://openai.com/index/sora-system-card/
- Video generation models as world simulators (Sora 1 technical blog post, including spacetime patches / native resolution / re-captioning): https://openai.com/index/video-generation-models-as-world-simulators/
- Sora 2 is here (release announcement): https://openai.com/index/sora-2/
- Disney and OpenAI licensing agreement announcement: https://openai.com/index/disney-sora-agreement/
- Sora, Not Sorry: OpenAI Backtracks on Opt-Out Copyright Policy, Copyright Lately: https://copyrightlately.com/openai-backtracks-sora-opt-out-copyright-policy/
- Sora 2 Does A Copyright Somersault Upon Launch, Forbes, 2025-10-17: https://www.forbes.com/sites/legalentertainment/2025/10/17/sora-2-does-a-copyright-somersault-upon-launch/
- MPA demands Sora 2 stop the infringement, CNBC, 2025-10-07: https://www.cnbc.com/2025/10/07/openais-sora-2-must-stop-allowing-copyright-infringement-mpa-says.html
- Japanese government makes a copyright-infringement demand to OpenAI over Sora 2, EC IP Helpdesk: https://intellectual-property-helpdesk.ec.europa.eu/news-events/news/japanese-government-requests-openai-avoid-copyright-infringement-sora-2-us-federal-judge-dismisses-2025-10-23_en
- Public Citizen's open letter calling for OpenAI to withdraw Sora 2: https://www.citizen.org/news/public-citizen-letter-calls-on-open-ai-to-withdraw-sora-2-video-generation/
- OpenAI Will Shut Down Sora Video App; Disney Drops $1B Investment, Variety, 2026-03: https://variety.com/2026/digital/news/openai-shutting-down-sora-video-disney-1236698277/
- Sora Shutting Down, Disney Investment Dead, Deadline, 2026-03: https://deadline.com/2026/03/sora-shut-down-disney-investment-1236764689/
- OpenAI Adds Longer Clips and Storyboarding to Sora 2, eWeek: https://www.eweek.com/openai/openai-adds-longer-clips-sora-2/
- How OpenAI Built Sora 2: Training, Data, and Model Design (third-party technical interpretation, not official): https://skywork.ai/blog/openai-sora-2-2025-ultimate-guide-training-model-design/
- How to Uncover Sora 2's Training Datasets (third-party, not official): https://skywork.ai/blog/how-to-uncover-sora-2s-training-datasets/
- Sora 2 API on Replicate (specifications and pricing): https://replicate.com/openai/sora-2
- Report on the Getty Images/OpenAI licensing partnership (2026-06): https://finance.yahoo.com/markets/stocks/articles/getty-images-openai-deal-gives-154500732.html

### [SpeakerVid-5M](../models/SpeakerVid-5M.md)

- 【Official primary】arXiv:2507.09862 "SpeakerVid-5M: A Large-Scale High-Quality Dataset for Audio-Visual Dyadic Interactive Human Generation" (submitted July 14, 2025, cs.CV): https://arxiv.org/abs/2507.09862 , full HTML https://arxiv.org/html/2507.09862v1 —— The direct source for the vast majority of fields in this entry, especially Section 3's data-construction pipeline, the quality-filtering thresholds, the four-branch organization, Table 1's dataset comparison, and Table 2's ablation results.
- 【Official primary】Project homepage https://dorniwang.github.io/SpeakerVid-5M/ —— The authors' institutional affiliations (Tsinghua University, StepFun, HKUST/Guangzhou), project lead Duomin Wang and corresponding author Xiu Li, resource-release status, and links to each entry point.
- 【Official primary】GitHub data-cleaning code repository https://github.com/Dorniwang/SpeakerVid-5M-Code —— The manifest of the six-stage curation pipeline code, the dependency toolchain (3D-Speaker/YOLO/DWpose/Whisper/SceneDetect/SyncNet/ArcFace/Deepface/UniSpeech/Deep3DFaceRecon/yt-dlp), and the license terms restricting use to non-commercial research and education, crediting the original copyright holders, and providing a takedown policy.
- 【Official primary】GitHub project-page repository https://github.com/Dorniwang/SpeakerVid-5M —— The source of the project homepage.
- 【Official primary】HuggingFace dataset https://huggingface.co/datasets/dorni/SpeakerVid-5M-Dataset (created 2025-07-18, last updated 2025-08-04, roughly 1,021 downloads, 18 likes) —— The manifest of what was actually released: all_data_list.json, SFT_set.json, and the five annotation folders merge_anno / dwpose / asr / l_score / anno; confirms that the raw videos are not hosted and must be downloaded individually by YouTube ID; confirms that the dwpose skeleton data was not uploaded due to its size.
- 【Same-team corroboration】arXiv:2602.08794 "MOVA: Towards Scalable and Synchronized Video–Audio Generation" —— Not the same team, but a downstream user; its Section 3 lists SpeakerVid-5M as a Phase 1 training-data source and explicitly positions it as the core source of lip-sync capability, used to cross-corroborate this dataset's language emphasis and audio-category attributes (this is corroborating evidence, not an official statement from SpeakerVid-5M itself).

### [Step-Video-T2V](../models/Step-Video-T2V.md)

- 【Official primary】Step-Video-T2V Technical Report: The Practice, Challenges, and Future of Video Foundation Model, Step-Video Team (StepFun), arXiv:2502.10248, 2025-02 (including Section 7, Data: the six stages of video splitting/quality assessment/motion assessment/captioning/concept balancing/video-text alignment, Figure 11's tiered-filtering diagram, the training-stage configuration table): https://arxiv.org/abs/2502.10248
- 【Official primary】Step-Video-T2V technical report ar5iv full text (the data chapter is searchable, including the 7 quality-label categories, the three Farneback optical-flow metrics, 120,000+ K-means clusters, 8-frame CLIP Score, 30M SFT samples, 95k distillation samples, and figures such as 644M/27.3M seen samples): https://ar5iv.labs.arxiv.org/html/2502.10248
- 【Official primary】Step-Video-T2V technical report arXiv HTML v1: https://arxiv.org/html/2502.10248v1
- 【Official primary】Step-Video-T2V technical report arXiv HTML v2 (including the tiered-filtering description with Figure 11, the Step-Video-T2V-Eval 128-prompt/11-category set, the Video-DPO human-preference annotation workflow): https://arxiv.org/html/2502.10248v2
- 【Official primary】Step-Video-T2V GitHub open-source repository (weights, inference code, Video-VAE, the bilingual text encoder, MIT license, released 2025-02-17): https://github.com/stepfun-ai/Step-Video-T2V
- 【Official primary】Step-Video-T2V Hugging Face model card: https://huggingface.co/stepfun-ai/stepvideo-t2v
- 【Official primary】Step-Video-T2V-Eval evaluation-benchmark dataset (128 Chinese-language prompts, 11 categories, including comparison generation results across multiple engines): https://huggingface.co/datasets/stepfun-ai/Step-Video-T2V-Eval
- 【Same-team corroboration】Step-Video-TI2V Technical Report, arXiv:2503.11251, 2025-03 (including 5M text-image-video triplets, a self-acknowledged data-mixture imbalance with >80% anime content, the optical-flow motion-score extraction method and threshold filtering, using motion score as a controllable condition, fine-tuning the captioning model to strengthen camera-movement descriptions, the Step-Video-TI2V-Eval set of 178+120 items): https://arxiv.org/html/2503.11251v1
- 【Same-team corroboration】Step-Video-TI2V GitHub open-source repository: https://github.com/stepfun-ai/Step-Video-TI2V
- 【Third-party】The Moonlight literature review: interpretation of the Step-Video-T2V technical report (the tiered filtering and the description of 6 pretraining subsets): https://www.themoonlight.io/en/review/step-video-t2v-technical-report-the-practice-challenges-and-future-of-video-foundation-model
- 【Third-party】Kingy AI: summary of the Step-Video-T2V technical report paper (the data pipeline and an overview of SFT/DPO): https://kingy.ai/blog/step-video-t2v-technical-report-paper-summary/
- 【Third-party】Hugging Face Papers: Step-Video-T2V paper page and community discussion: https://huggingface.co/papers/2502.10248
- 【Third-party】CSDN: StepFun releases the strongest open-source video generation model, Step-Video-T2V (a detailed paper walkthrough, breaking down the Chinese-language data chapter item by item): https://blog.csdn.net/sherlockMa/article/details/145706142
- 【Third-party】CSDN: StepFun's open-source exploration — an in-depth analysis of Step-Video-T2V and Step-Audio (explaining that the two are independent model lines, not joint AV generation): https://blog.csdn.net/liaoqingjian/article/details/145820964
- 【Third-party】Zhihu: a brief analysis of StepFun's 30B video generation model, Step-Video: https://zhuanlan.zhihu.com/p/24619034131
- 【Third-party】Zhihu: introduction to StepFun's open-sourced image-to-video model Step-Video-TI2V (102 frames/5 seconds/540P, controllable motion magnitude and controllable camera movement): https://zhuanlan.zhihu.com/p/31775732208
- 【Third-party】BAAI Hub: StepFun's first open-sourcing of the Step series multimodal large models (2025-02-18 official-release report): https://hub.baai.ac.cn/view/43466
- 【Third-party】NeuroHive: interpretation of Step-Video-T2V's open-source-model breakthrough of 16x video compression: https://neurohive.io/en/state-of-the-art/step-video-t2v-text-to-video-open-source-model-achieves-16x-video-compression-breakthrough/

### [UniTalking](../models/UniTalking.md)

- 【Official primary】arXiv:2603.01418v1 "UniTalking: A Unified Audio-Video Framework for Talking Portrait Generation" (submitted 2026-03-02, accepted at CVPR 2026 Findings): https://arxiv.org/abs/2603.01418 , full HTML https://arxiv.org/html/2603.01418v1 , PDF https://arxiv.org/pdf/2603.01418 —— The sole direct source for all fields in this entry. The full text is 10 pages (including references); Section 3, "Data Preparation," is the entirety of the data section, roughly one page long; the Appendix referenced in the body is not actually included in the v1 PDF.
- 【Third-party corroboration】ResearchGate entry: https://www.researchgate.net/publication/401470058_UniTalking_A_Unified_Audio-Video_Framework_for_Talking_Portrait_Generation —— Used to cross-check the title and authors.
- 【Third-party corroboration】X/Twitter post by @CSVisionPapers: https://x.com/CSVisionPapers/status/2028948978948051095 —— Used to confirm the author list and the arXiv categories (cs.CV, cs.MM, cs.SD).
- 【Third-party corroboration】talking-face-arxiv-daily paper-tracking repository: https://github.com/liutaocode/talking-face-arxiv-daily —— Used to confirm the paper's inclusion in the talking-face generation direction and to cross-verify that no official code repository exists.
- 【Primary source for upstream data】OpenHumanVid (CVPR 2025, Fudan University) arXiv:2412.00115: https://arxiv.org/abs/2412.00115 , project homepage https://fudan-generative-vision.github.io/OpenHumanVid , code repository https://github.com/fudan-generative-vision/OpenHumanVid —— Used to supplement upstream-dataset attributes not described by UniTalking itself (52.3M raw clips / 70.6K hours → 13.2M high-quality clips after filtering, three caption types — short/long/structured —, skeleton sequences and audio tracks, a download license requiring review and authorization). This is corroborating evidence; UniTalking's own paper does not restate these figures.
- 【Base-model corroboration】Wan technical report arXiv:2503.20314 —— Used to confirm Wan2.2's 3D causal VAE compression ratio, resolution, and frame-rate specifications (UniTalking's video-side specifications are implicitly determined by this base model; the paper itself does not disclose them).
- 【Comparison-work corroboration】OVI (arXiv:2510.01284) and UniVerse-1 (arXiv:2509.06155) —— UniTalking's primary comparison baselines; its RoPE strategy explicitly follows OVI, and its data pipeline can be directly compared with UniVerse-1's.

### [UniVerse-1](../models/UniVerse-1.md)

- 【Official primary】arXiv:2509.06155v1 "UniVerse-1: Unified Audio-Video Generation via Stitching of Experts" (submitted 2025-09-07): https://arxiv.org/abs/2509.06155 , full HTML https://arxiv.org/html/2509.06155v1 —— The sole direct source for the vast majority of fields in this entry, especially Section 3's data construction and online annotation pipeline, Section 4's experiments and ablations, and the Verse-Bench category table in the Appendix.
- 【Official primary】GitHub code repository Dorniwang/UniVerse-1-code: https://github.com/Dorniwang/UniVerse-1-code/ —— The manifest of open-sourced content, Apache-2.0 license, release timeline (project page 2025-09-03 / weights and Verse-Bench 09-08 / technical report 09-09 / evaluation tools 09-28).
- 【Official primary】HuggingFace model page dorni/UniVerse-1-Base: https://huggingface.co/dorni/UniVerse-1-Base —— 7B parameter scale, bfloat16 inference configuration, Apache-2.0 statement, a stated 7,600 hours of training data.
- 【Official primary】Project homepage: https://dorniwang.github.io/UniVerse-1/ —— Institutional affiliations (StepFun / HKUST(GZ) / HKUST / Tsinghua University), demo examples, a description of the layer-interpolation method, the composition of Verse-Bench's three subsets.
- 【Official primary】HuggingFace Papers page: https://huggingface.co/papers/2509.06155 —— Cross-checked against the abstract and core contributions.
- 【Same-team corroboration】OpenReview submission page https://openreview.net/forum?id=8aFYx2mDyE —— Used to confirm the paper's conference-submission status.
- 【Third-party corroboration】ResearchGate paper entry: https://www.researchgate.net/publication/395356081_UniVerse-1_Unified_Audio-Video_Generation_via_Stitching_of_Experts
- 【Third-party corroboration】Citations of and comparisons to UniVerse-1 by later work in the same direction, such as UniAVGen (arXiv:2511.03334), JavisDiT++ (arXiv:2602.19163), and MMControl (arXiv:2604.19679) —— Used to confirm that UniVerse-1 has become one of the standard comparison baselines for joint audio-video generation.

### [Unison](../models/Unison.md)

- 【Official primary】arXiv:2605.08729v2 "Unison: Harmonizing Motion, Speech, and Sound for Human-Centric Audio-Video Generation" (v1 submitted 2026-05-09, v2 updated 2026-06-29, cs.CV, CC BY 4.0): abstract page https://arxiv.org/abs/2605.08729 , full HTML https://arxiv.org/html/2605.08729v2 , PDF https://arxiv.org/pdf/2605.08729 —— The sole direct source for all fields in this entry. Data-related information is concentrated in Section 4.1, Training Corpora (one paragraph), and the lip-filtering description (three sentences) at the start of Section 3's Methods; no other data field has textual support.
- 【Official primary】arXiv:2605.08729v1 HTML: https://arxiv.org/html/2605.08729v1 —— Compared section by section; the training-corpus and data-processing descriptions are identical to v2, confirming that the data-side content has been unrevised since v1.
- 【Cited upstream dataset, third-party primary】OpenHumanVid (arXiv:2412.00115, a large-scale, high-quality, person-centric video dataset), HDTF (High-Definition Talking Face, CVPR 2021), VFHQ (arXiv:2205.03409, high-fidelity face video), CelebV-Text (CVPR 2023, text-annotated face video), VGGSound (ICASSP 2020, roughly 200K clips / 550 hours of audio-visual event data) —— The five sources for the joint audio-video training corpus; each one's original scale, collection method, and annotation scheme serve as indirect evidence for inferring Unison's data distribution.
- 【Cited upstream dataset, third-party primary】YouTube-8M, AudioSet, WavCaps (sources of sound effects); VidMuse (source of music); YuE / YuE-scaling (arXiv, source of singing) —— Sources of unimodal training data for the audio branch.
- 【Cited upstream models and tools, third-party primary】Wan2.2 (video base model), MMAudio (arXiv:2412.15322, audio base model), Zipformer/ZipVoice (speech-generation enhancement), Mel-RoFormer (arXiv:2409.04702, vocal separation, used both for training-data decoupling and evaluation preprocessing), SyncNet (lip sync), Whisper-large-v3 (WER evaluation), Synchformer (DeSync evaluation), ImageBind/CLAP/VideoCLIP-XL-V2/LAION-Aesthetic V2.5/DINOv3/Audiobox-Aesthetics (evaluation metrics).
- 【Same-direction corroboration, for lateral comparison】UniVerse-1 (arXiv:2509.06155), Ovi, UniAVGen (arXiv:2511.03334), Harmony (arXiv:2511.21579), MOVA (arXiv:2602.08794), LTX-2 (arXiv, the 19B joint audio-video generation model), JavisDiT (arXiv:2503.23377) —— All comparison baselines in Unison's Table 1; their publicly disclosed data pipelines can be used to highlight the gaps in Unison's own disclosure.
- 【Same-name work, to be excluded】arXiv:2605.31530 "UNISON: A Unified Sound Generation and Editing Framework via Deep LLM Fusion" —— A pure audio generation and editing framework, unrelated to this entry; care must be taken to distinguish it during searches.

### [Veo 3 / Veo 3.1](../models/Veo_3_Veo_31.md)

- Official primary: Veo 3 Tech Report (PDF, the Model & Data / Mitigations sections) https://storage.googleapis.com/deepmind-media/veo/Veo-3-Tech-Report.pdf
- Official primary: Veo 3 Model Card (PDF, Training Dataset/Processing/Evaluation) https://storage.googleapis.com/deepmind-media/Model-Cards/Veo-3-Model-Card.pdf
- Official primary: Veo 3.1 Lite Model Card (confirms the 3.1 series carries forward Veo 3's data disclosures) https://deepmind.google/models/model-cards/veo-3-1-lite/
- Third-party report: CNBC — Google used a subset of YouTube videos to train Gemini and Veo 3 https://www.cnbc.com/2025/06/19/google-youtube-ai-training-veo-3.html
- Official primary: Video models are zero-shot learners and reasoners (arXiv:2509.20328, inferring the absence of explicit geometric annotation) https://arxiv.org/abs/2509.20328
- Official primary: Veo product page (output specifications and MovieGenBench results) https://deepmind.google/models/veo/

### [Vidu S1](../models/Vidu_S1.md)

1) Official primary: arXiv:2607.03118v2 "Vidu S1: A Real-Time Interactive Video Generation Model" https://arxiv.org/abs/2607.03118 (including Section 2.1, Data Preparation, and Figure 2's data-filtering pipeline diagram; almost all data-side information in this survey comes from this section)
2) Official primary: product page and online demo https://vidu.com/vidu-stream
3) Third-party report: China Daily's technology channel, "Shengshu Technology releases Vidu S1, pushing video generation into a new era of 'real-time interaction'" https://tech.chinadaily.com.cn/a/202607/06/WS6a4b12eea310d709c2fbbecb.html
4) Third-party report: Leiphone https://www.leiphone.com/category/industrynews/6GlFzI5hMwcfRoGZ.html ; iFanr https://www.ifanr.com/digest/1670950 (release date, the AR+Diffusion architecture, product parameters such as 540P/25-42FPS)
5) Third-party compilation: an AI-technology deep-dive entry on Vidu S1 https://www.ai-all.info/ai-models/vidu-s1 (TurboDiffusion/TurboServe, runs on consumer GPUs starting from an RTX 3060, end-to-end latency <200ms, etc.; secondhand information)

### [Wan 2.5 / 2.6 / 2.7](../models/Wan.md)

- 【Official primary · core】Wan: Open and Advanced Large-Scale Video Generative Models (Wan Team, Alibaba's Tongyi Lab), arXiv:2503.20314, 2025-03-26, 60 pages with 33 figures. Chapter 3, "Data Processing Pipeline," is the sole detailed primary source for the entire series' data methodology: 3.1 Pretraining data (the four-step cleaning process, 9 basic filters, roughly a 50% elimination rate, quota sampling across 100 clusters, six-tier motion grading, synthesis of on-screen text), 3.2 Post-training data (top 20%, 12 major categories, millions of samples), 3.3 Dense video captioning (an LLaVA-style architecture, slow-fast encoding, a 10-dimension F1 score benchmarked against Gemini 1.5 Pro); 4.2.2/4.2.3 the training curriculum; 4.6 Wan-Bench; 4.7.2 architecture ablations; 5.7 audio generation (V2A's 1D-VAE, three-stage audio-video captioning, an O(1)-thousand-hour subset, excluding speech): https://arxiv.org/abs/2503.20314
- 【Official primary】Wan 2.1 technical report PDF (this survey's quotations from the data chapter are all drawn from this PDF): https://arxiv.org/pdf/2503.20314
- 【Official primary】Wan-S2V: Audio-Driven Cinematic Video Generation, arXiv:2508.18621, 2025-08-26. Chapter 2, "Data Processing Pipeline," is the only document in the Wan series that spells out the audio-video sync data-filtering method: Light-ASD's two active-speaker-detection exclusion rules, VitPose/DWPose pose tracking, the five quality metrics of Dover/UniMatch/Laplacian/aesthetic/OCR, and QwenVL2.5-72B captioning specifications; Table 1 provides quantitative comparisons including Sync-C: https://arxiv.org/abs/2508.18621
- 【Official primary】Wan2.2 GitHub repository README (the data increments "images +65.6%, video +83.2%," a cinematic-aesthetic tag system covering lighting/composition/contrast/color-tone, MoE dual-expert denoising, the Wan2.2-VAE's 16×16×4 compression, S2V and Animate, Apache 2.0): https://github.com/Wan-Video/Wan2.2
- 【Official primary】Wan2.1 GitHub repository: https://github.com/Wan-Video/Wan2.1
- 【Official primary】Wan-Video GitHub organization page (as of 2026-07, only five repositories exist — Wan2.1/Wan2.2/Wan-Dancer/Wan-skills/diffusers — with no Wan2.5/2.6/2.7, the direct evidence of closed-sourcing): https://github.com/Wan-Video
- 【Official primary】Hugging Face Wan-AI organization page (the latest weights stop at the Wan2.2 series and Wan-Dancer-14B, with no 2.5/2.6/2.7): https://huggingface.co/Wan-AI
- 【Official primary · capability matrix】Alibaba Cloud Bailian's "video generation" model overview documentation (the full capability tags, input modalities, resolution tiers, duration, and fps for all versions — wan2.7/2.6/2.5/2.2/2.1; explicitly states 2.5 = "audio video · synchronized audio-visual generation," 2.6/2.7 = "audio video · multi-shot narrative · synchronized audio-visual generation," and that 2.7-i2v's input modalities include text/image/audio/video): https://help.aliyun.com/zh/model-studio/video-generation
- 【Official primary · API】Wanxiang 2.7 text-to-video API reference (model name wan2.7-t2v-2026-06-12 with a release-date stamp, the audio_url audio-conditioning input, automatic dubbing, a timestamped storyboard-script example, duration range [2,15], the ratio-to-resolution correspondence table, a 5,000-character prompt limit, the watermark parameter): https://help.aliyun.com/zh/model-studio/text-to-video-api-reference
- 【Official primary · API】Wanxiang image-to-video (first-frame-based) API reference (2.1–2.6) (wan2.6's shot_type:"multi" multi-shot narrative, wan2.5/2.6's audio_url and automatic dubbing, the 3–30 second/wav/mp3/≤15MB audio constraints and truncation rules, per-version resolution and duration enumerations, differing prompt character limits): https://help.aliyun.com/zh/model-studio/image-to-video-api-reference
- 【Official primary · release announcement】"The all-new Wanxiang 2.6 series model, officially released!," Alibaba Cloud Developer Community, 2025-12-17 (China's first video generation model supporting role-play, synchronized audio-visual generation, multi-shot generation, sound-driven generation, up to 15 seconds per generation, storyboard control and cross-shot consistency modeling; the Wanxiang family now supports over 10 visual-creation capabilities): https://developer.aliyun.com/article/1693622
- 【Official primary · release announcement】"Wanxiang Wan2.6 all-new upgrade released! The era where everyone can be a director has arrived," Alibaba Cloud Developer Community, 2025-12-16: https://developer.aliyun.com/article/1693451
- 【Official primary · same-team corroboration】Wan-Dancer (music-driven minute-scale dance-video generation), arXiv:2607.09581, 2026-07. Data side: an in-house proprietary dataset of roughly 200 hours at ≥720p/30fps, five dance genres in near-equal proportion to mitigate class imbalance, a 5-second clipping strategy with 50% overlap, Librosa audio features, SEA-RAFT optical flow masked into the loss: https://arxiv.org/abs/2607.09581
- 【Official primary】Wan-Dancer GitHub repository README: https://github.com/Wan-Video/Wan-Dancer
- 【Official primary】Wanxiang's official site (the product page and online-demo entry points for Wan 2.5/2.6/2.7): https://wan.video/
- 【Official primary】Tongyi Wanxiang's official entry point: https://tongyi.aliyun.com/wan/
- 【Third-party compilation】Wanxiang 2.6 product description page ("Major release on 2025.12.16," native audio-visual synchronization, multi-shot narrative, the first video-generation-based role-play feature, an audio-driven mode, 15 seconds/1080P, an improvement over Wan 2.5's 10 seconds): https://wan2.video/zh/wan2.6
- 【Third-party compilation】Wanxiang 2.7 product description page (1080P/15-second ceiling, first/last-frame control, character cloning, instruction-based editing, single-image multi-shot narrative, native audio-visual synchronization): https://wan2.video/zh/wan2.7
- 【Third-party report】"Tongyi Wanxiang Wan2.6 released: from 'random generation' to 'precise direction,'" Tencent News, 2025-12-17: https://news.qq.com/rain/a/20251217A031VN00
- 【Third-party report】"Alibaba's Tongyi Wanxiang Wan 2.6 released, from 'generating a video clip' to 'helping you shoot the scene,'" Zhihu column: https://zhuanlan.zhihu.com/p/1984672026435294934
- 【Third-party report】"Tongyi Wanxiang Wan2.6 officially released! How to use it?," Zhihu column: https://zhuanlan.zhihu.com/p/1986838016199766019
- 【Related work for comparison】LTX-2: Efficient Joint Audio-Visual Foundation Model, arXiv:2601.03233 (information-based audio filtering and full-soundscape dual-track captioning, contrasted with Wan's three-stage V2A captioning): https://arxiv.org/abs/2601.03233
- 【Method citation · third-party】Light-ASD: A Light Weight Model for Active Speaker Detection, Liao et al., CVPR 2023 (the model used for audio-visual sync filtering in Wan2.2-S2V): https://openaccess.thecvf.com/content/CVPR2023/html/Liao_A_Light_Weight_Model_for_Active_Speaker_Detection_CVPR_2023_paper.html

### [Audio-Video Generation Evaluation Benchmark Collection](../models/av_benchmarks.md)

- https://arxiv.org/abs/2512.09299 —— VABench: A Comprehensive Benchmark for Audio-Video Generation, arXiv abstract page (official primary). Authors Daili Hua, Xizhi Wang, Bohan Zeng, Xinyi Huang, Hao Liang, Junbo Niu, Xinlong Chen, Quanqing Xu, Wentao Zhang; v1 2025-12-10, v2 2026-04-06; 24 pages with 25 figures; cs.CV + cs.SD; CC BY 4.0.
- https://arxiv.org/html/2512.09299v1 —— VABench paper full HTML (official primary). Extracted: the seven-category taxonomy, the 15 evaluation dimensions across two modules, the T2AV/I2AV dual-path data-curation workflow, the 778/521/116 sample counts, 9 stereo acoustic metrics, the list of evaluated models, and the author institutions (Peking University / Ant Group / CASIA / Huazhong University of Science and Technology).
- https://github.com/tanABCC/VABench —— VABench official code repository (official primary).
- https://arxiv.org/abs/2605.24652 —— AVBench: Human-Aligned and Automated Evaluation Benchmark for Audio-Video Generative Models, arXiv abstract page (official primary). Authors Jialiang Yang, Bin Xia, Ruihang Chu, Dingdong Wang, Wanke Xia, Zhun Mou, Tianyang Zhong, Yiting Zhao, Wenming Yang.
- https://arxiv.org/html/2605.24652v1 —— AVBench paper full HTML (official primary). Extracted: 10 evaluation dimensions, 470 evaluation prompts with a Normal/Hard split, a 30K→300K hard-negative synthesis recipe, the Qwen2.5-Omni 7B and Qwen2-Audio 7B evaluator architectures, a 4-expert 2AFC annotation protocol, the discussion of using it for data filtering and RLHF reward, and the author institutions (Tsinghua University / Chinese University of Hong Kong).
- https://yajialiang.github.io/AVBench-site/ —— AVBench official project page (official primary). Contains links to GitHub, HuggingFace models, and the leaderboard.
- https://github.com/YaJialiang/AVBench —— AVBench official code repository (official primary).
- https://huggingface.co/iiiiii123/AVBench_model —— AVBench evaluator weights (official primary).
- https://huggingface.co/spaces/iiiiii123/AVBenchLB —— AVBench online leaderboard (official primary).
- https://arxiv.org/abs/2607.00726 —— AV-SyncBench: Decoupled Benchmarking of Temporal and Semantic Audio-Visual Synchronization, arXiv abstract page (official primary). Authors Tianhong Zhou, Mingyang Han, Boyu Li, Yuxuan Jiang, Jiaxin Ye, Dongxiao Wang, Haoxiang Shi, Kunpeng Wang, Jun Song, Cheng Yu, Bo Zheng; accepted at Interspeech 2026; 3,269 videos / 38,390 samples.
- https://arxiv.org/html/2607.00726v1 —— AV-SyncBench paper full HTML (official primary). Extracted: a list of 10 scenarios and 5 challenge tasks, a 0.64-second chunk diagonal-similarity evaluation protocol, a Gemini 3 Flash automatic-filtering + 5-annotator ≥3-way cross-review workflow, the perturbation-parameter spectrum (offset: 5 tiers of 50–500ms / jitter: 3 tiers of 30–700ms / speed change: 10 tiers of 0.8×–1.25×), the OpenVoice V2 and DDSP semantic-perturbation tools, findings from 5 baseline-model tests, and the author institutions (Alibaba Group / Tsinghua University / Fudan University).
- https://fgt7t6g.github.io/AV-SyncBench —— AV-SyncBench official project page (official primary). Contains links to the ModelScope / HuggingFace / GitHub dataset and code.
- https://github.com/fgt7t6g/AV-SyncBench —— AV-SyncBench official code repository (official primary); at the time of research, the evaluation code was marked "coming soon."
- https://huggingface.co/datasets/coming245/AV-SyncBench —— AV-SyncBench dataset (official primary).
- https://modelscope.cn/datasets/coming245/AVSyncBench —— AV-SyncBench dataset ModelScope mirror (official primary).
- https://arxiv.org/abs/2512.23994 —— PhyAVBench: A Challenging Audio Physics-Sensitivity Benchmark for Physically Grounded Text-to-Audio-Video Generation, arXiv abstract page (official primary). First author Tianxin Xie, with 29+ collaborators; 25.5 hours / 11,605 videos / 337 paired prompt sets / 6 dimensions with 41 test points / 17 models evaluated.
- https://arxiv.org/html/2512.23994v1 —— PhyAVBench v1 paper full HTML (official primary). Extracted: the five-stage curation workflow, the breakdown of physical dimensions and test points, definitions of the CPRS and FGAS metrics, and the author institutions (HKUST(GZ) / Tencent / Shanghai Jiao Tong University / Technical University of Munich; corresponding author Li Liu). Note that v1 is a pure benchmark-design report (describing 1,000 prompt sets, 50 test points, with model evaluation left for later versions), and differs in version from the 337-set / 41-test-point figures in later versions; the main text follows the later version.
- https://arxiv.org/html/2512.23994v3 —— PhyAVBench latest-version paper full HTML (official primary). Extracted: the PhyAV-Sound-11K dataset specification (11,605 items / 25.5 hours / 184 participants / an average of 17 ground-truth items per set), CPRS switched to use CLAP embeddings, PVR-MOS with 74 raters, the complete list of the 17 evaluated models along with results such as Sora 2's CPRS of 0.4512, and a Pearson correlation of 0.92 between CPRS and human judgment.
- https://imxtx.github.io/PhyAVBench/ —— PhyAVBench official project page (official primary).
- https://phyavbench.pages.dev/ —— PhyAVBench official project page mirror (official primary).
- https://arxiv.org/abs/2602.01623 —— Omni-Judge: Can Omni-LLMs Serve as Human-Aligned Judges for Text-Conditioned Audio-Video Generation?, arXiv abstract page (official primary). First author Susan Liang, with 9 collaborators including Jason J. Corso and Chenliang Xu.
- https://arxiv.org/html/2602.01623v1 —— Omni-Judge paper full HTML (official primary). Extracted: the Qwen3-Omni (30B/3B activated) judge model, 9 evaluation dimensions, 300 VidProM prompts + 600 videos each generated by Sora 2 and Veo 3, a 6-PhD-student 1–5 scale annotation protocol, per-dimension Kendall τ_b and Spearman ρ correlation results, and the author institutions (University of Rochester / University of Michigan, Ann Arbor).
- https://liangsusan-git.github.io/project/omni_judge/ —— Omni-Judge official project page (official primary).

### [Video Captioning Model Ecosystem](../models/caption_models.md)

- https://arxiv.org/abs/2406.04325 — ShareGPT4Video paper (official primary, NeurIPS 2024 D&B Track, CC BY 4.0; 40K GPT4V dense captions + 4.8M ShareCaptioner-Video annotations + the DiffSW method)
- https://huggingface.co/Lin-Chen/ShareCaptioner-Video — ShareCaptioner-Video model card (official primary; confirms the base is InternLM-XComposer2-4KHD, supports streaming differential sliding-window captioning and clip summarization)
- https://arxiv.org/abs/2501.07888 — Tarsier2 paper (official primary; 7B, pretrained on 11M→40M video-text pairs, fine-grained temporal-alignment SFT, model-based sampling + DPO; a DREAM-1K F1 score exceeding GPT-4o by 2.8% and Gemini-1.5-Pro by 5.8%; human evaluation gains of +8.6% / +24.9%; SOTA on 15 public benchmarks)
- https://github.com/bytedance/tarsier — Tarsier official repository (official primary; the Tarsier2-Recap-7b weights and the Tarsier2-Recap-585K dataset)
- https://huggingface.co/omni-research/Tarsier2-Recap-7b — Tarsier2-Recap model card (official primary; 585K clips drawn from public datasets such as VATEX/TGIF/LSMDC, annotated by Tarsier2-7B; upstream is a recaptioning of 1M videos)
- https://github.com/zai-org/CogVideo/blob/main/tools/caption/README.md — CogVideoX captioning-tool documentation (official primary; the use of CogVLM2-Caption and the closed video→caption→video loop)
- https://arxiv.org/pdf/2408.06072 — CogVideoX paper (official primary; the four-stage captioning chain — Panda-70M short captions → CogVLM per-frame dense captioning → GPT-4 summarization → fine-tuning a LLaMA2 summarization model on 50,000 samples → distillation into CogVLM2-Caption — with the complete prompt given in Appendix G)
- https://arxiv.org/abs/2410.03051 — AuroraCap + VDC benchmark (official primary; token merging to reduce visual tokens, a three-stage training process over 20 million image/video-text pairs, VDC's 1,000+ structured captions with five fields — camera/short/background/main object/detailed —, and the divide-and-conquer LLM-based VDCScore evaluation; a Flickr30k CIDEr of 88.9, exceeding GPT-4V's 55.3 and Gemini-1.5-Pro's 82.2)
- https://arxiv.org/abs/2410.08260 — Koala-36M (official primary; 36M clips averaging 13.75s, 720p, captions averaging 202 words, GPT-4V-generated seed captions used to fine-tune LLaVA, six categories of structured information, the VTSS training-suitability score)
- https://openaccess.thecvf.com/content/CVPR2024/papers/Chen_Panda-70M_Captioning_70M_Videos_with_Multiple_Cross-Modality_Teachers_CVPR_2024_paper.pdf — Panda-70M (official primary; 31 candidate captioners = 6 base models × weight/modality variants, greedy set-cover selection of 8 covering 76.8%, UMT-large fine-grained retrieval for selecting the best, a human-agreement rate of only 44.9%)
- https://arxiv.org/abs/2510.10395 — AVoCaDO (official primary; a Qwen2.5-Omni-7B base, the six-source composition of AVoCaDO-SFT-107K, a five-dimension keypoint checklist, GRPO with three reward terms including a dialogue edit-distance DP-alignment threshold of 0.6)
