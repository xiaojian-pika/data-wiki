# 主流视频预训练数据集合并调研：Panda-70M、InternVid、Koala-36M、MiraData、OpenVid-1M、UltraVideo、LVD-2M（共7个公开视频-文本数据集，覆盖2023–2025年，重点比较清洗方法与caption策略）

> 主题: 视频生成模型（含音视频同时生成）的数据处理：数据清洗流程、数据分布、打标方式

[← 返回首页](../index.md)

**目录**: [基本信息](#基本信息) · [数据规模与分布](#数据规模与分布) · [清洗流程](#清洗流程) · [打标方式](#打标方式) · [音视频对齐](#音视频对齐) · [训练配合](#训练配合) · [效果对比](#效果对比)

## 基本信息

### 名称

主流视频预训练数据集合并调研：Panda-70M、InternVid、Koala-36M、MiraData、OpenVid-1M、UltraVideo、LVD-2M（共7个公开视频-文本数据集，覆盖2023–2025年，重点比较清洗方法与caption策略）

### 发布机构/公司

Panda-70M：Snap Inc.（Snap Research）+ 加州大学默塞德分校（UC Merced）。InternVid：上海人工智能实验室 / OpenGVLab（联合南京大学、香港大学、南洋理工、深圳先进院）。Koala-36M：快手科技（Kuaishou，仓库现属 KlingAIResearch，原 KwaiVGI）+ 深圳大学 + 清华大学。MiraData：腾讯 PCG ARC Lab + 香港中文大学。OpenVid-1M：南京大学 PCA Lab + 字节跳动 + 南开大学。UltraVideo：浙江大学 APRIL Lab + 上海交通大学 + 华中科技大学 + 南洋理工大学。LVD-2M：香港大学 + 字节跳动。

### 发布时间（技术报告/论文/开源时间）

Panda-70M：2024年2月29日 arXiv 首发（arXiv:2402.19479），CVPR 2024；2024年10月追加 desirability 过滤与镜头边界标注。InternVid：2023年7月13日 v1、2024年1月4日 v2（arXiv:2307.06942），ICLR 2024 spotlight。Koala-36M：2024年10月10日 v1、2025年4月26日 v2（arXiv:2410.08260），CVPR 2025。MiraData：2024年7月8日（arXiv:2407.06358，仅 v1），NeurIPS 2024 D&B；v1 数据于2024年7月10日发布，v0 beta 更早。OpenVid-1M：2024年7月1日发布（arXiv:2407.02371），ICLR 2025 接收（2025年1月）；OpenVidHD-0.4M 独立下载包2025年5月30日补充。UltraVideo：2025年6月16日（arXiv:2506.13691，仅 v1），NeurIPS 2025 D&B poster。LVD-2M：2024年10月14日 arXiv、10月15日数据发布（arXiv:2410.10816），NeurIPS 2024 D&B。

### 类型（模型/数据集/工具链/评测基准）

全部为「数据集」，但均附带不同程度的工具链与配套模型：Panda-70M 附切分代码 + 蒸馏 caption 学生模型；InternVid 附 ViCLIP 视频-文本模型；Koala-36M 附转场检测代码 + VTSS 打分模型（简化版）；MiraData 附 GPT-4V 打标脚本 + MiraBench 评测基准（兼具「评测基准」属性）；OpenVid-1M 附 MVDiT 生成模型与训练代码；UltraVideo 附 UltraWan-1K/4K 生成模型（LoRA）；LVD-2M 仅附 YouTube 下载脚本。七者均无商用级生成模型发布。

### 开源程度（权重/代码/数据/pipeline各自是否开源） ⚠️

关键差异在于「是否托管视频本体」——这是复现成本的分水岭。
【仅发元数据（URL+时间戳+caption），视频需自行爬取】Panda-70M（CSV，Google Drive，含 matching_score/desirability/shot_boundary 列）；InternVid（jsonlines，HF 上 40.9GB，gated 需填姓名/单位/邮箱）；Koala-36M（10个CSV分片共48.9GB，HF 上无 dataset card、无 license 标签）；MiraData（4个CSV：330K/93K/42K/9K）；LVD-2M（3个CSV共约5.08GB，托管在 S3）。
【真正托管视频本体】OpenVid-1M（HF nkp37/OpenVid-1M，约12.4TB，74+个zip分片，超50GB的需cat重组；OpenVidHD-0.4M 单独约4.5TB）；UltraVideo（HF APRIL-AIGC/UltraVideo，clips_short_{1..36}.zip 原生分辨率 + 1920/960 降采样版本）。这两者是七者中唯一可直接拿到像素的。
【清洗代码开源程度】Koala-36M 开源了转场检测完整实现（含拟合好的SVM系数）与 VTSS 推理代码，但明确说明发布的是「base version」——实测配置为 fragments-only 的 FAST-VQA 架构（PLCC 0.8684），并非论文中含 ConvNeXt 静态分支+WCGB 的完整模型（PLCC 0.8974），即**发布的打分器不是造数据用的那个**；MiraData 开源了 GPT-4V 打标 prompt 与 MiraBench 评测代码，但**过滤阈值论文称在补充材料中而补充材料实际不存在**；LVD-2M 完全未开源过滤代码，仅在论文附图中给出 PLLaVA prompt；UltraVideo 仅开源推理代码，训练与清洗 pipeline 均未发布（GitHub issue #8 追问未果）；Panda-70M 开源切分代码（cutscene_detect/event_stitching/video_splitting）与学生 caption 模型权重，但未开源教师模型推理脚本与 UMT 选择器权重；InternVid 开源 ViCLIP，清洗 pipeline 仅文字描述；OpenVid-1M 开源 MVDiT 训练/推理代码与256~1024分辨率检查点。
【许可证】Panda-70M：Snap 非商用研究许可 + 继承 HD-VILA-100M 许可；InternVid：CC-BY-NC-SA-4.0；Koala-36M：快手非商用研究许可 + 继承 HD-VILA-100M 许可；MiraData：代码 GPL-3.0，数据条款自相矛盾（README 前段禁止商用、末段称「支持商用」）；OpenVid-1M：CC-BY-4.0 但声明仅供研究与非商业用途，且须遵守上游 Panda/ChronoMagic/CelebvHQ/Open-Sora-Plan 各自许可；UltraVideo：自定 license-april-lab.txt，非商用、禁止二次分发原始视频、须遵守 YouTube ToS 与 GDPR/CCPA；LVD-2M：无独立 LICENSE 文件，声明与 HD-VILA 许可一致。七者全部为非商用研究许可（OpenVid 的 CC-BY-4.0 名义宽松但被自身声明收窄），且均无一家真正拥有 YouTube 素材版权。

### 是否支持音视频同时生成，以及实现方式（原生联合/级联/MoE融合）

七个数据集**全部不支持音视频同时生成**，均为纯视觉+文本数据集。音频处理情况分三档：(1) 完全不涉及——InternVid、Koala-36M、MiraData、OpenVid-1M、LVD-2M 的论文中「audio」一词仅出现在参考文献标题里（Koala-36M 与 LVD-2M 经全文 grep 确认各仅1次，均在文献题名中），元数据无任何音频字段；(2) 音轨被动保留但未处理——Panda-70M 的下载配置 panda70m.yaml 设 download_audio: True，用户拿到的 mp4 混流了原始 YouTube 音轨，但数据集本身不含任何音频标注、音频质检或语音标签，且其教师模型 Video-LLaMA 被明确关闭了音频分支（「We only use the vision branch」）；UltraVideo 保留原生音轨且在 Limitations 中提及「thanks to the preservation of native resolution, frame rate, and audio…可用于音乐生成等任务」，但同样无任何音频侧的筛选或标注；(3) Koala-36M 的重切分代码用 cv2.VideoWriter/mp4v 输出，会**直接丢弃音轨**。因此本条目下所有音频与音视频对齐字段（audio_category_distribution、joint_av_caption_schema、dialogue_transcription_attributes、av_sync_detection、sync_metric_and_threshold、temporal_vs_semantic_sync、audio_quality_filtering、audio_type_handling）均为「不适用」。

### 调研信息来源列表（论文/技术报告/官方文档/新闻的URL，标注每条来源的性质：官方一手/同团队旁证/第三方报道）

【官方一手·论文】1) Panda-70M arXiv:2402.19479 https://arxiv.org/html/2402.19479v1 与 CVPR2024 开放版 https://openaccess.thecvf.com/content/CVPR2024/papers/Chen_Panda-70M_Captioning_70M_Videos_with_Multiple_Cross-Modality_Teachers_CVPR_2024_paper.pdf ；2) InternVid arXiv:2307.06942 https://arxiv.org/html/2307.06942v2 ；3) Koala-36M arXiv:2410.08260 https://arxiv.org/html/2410.08260v2 （v2 含附录A–G，全部阈值出处）；4) MiraData arXiv:2407.06358 https://arxiv.org/pdf/2407.06358 与 NeurIPS 2024 camera-ready https://proceedings.neurips.cc/paper_files/paper/2024/file/57f6683e550eb067936c9e9f0bcb8e31-Paper-Datasets_and_Benchmarks_Track.pdf ；5) OpenVid-1M arXiv:2407.02371 https://arxiv.org/html/2407.02371v2 与 OpenReview https://openreview.net/forum?id=j7kdXSrISM ；6) UltraVideo arXiv:2506.13691 https://arxiv.org/html/2506.13691v1 与 NeurIPS 2025 poster #121373 / OpenReview zYqM6gkqBi ；7) LVD-2M arXiv:2410.10816 https://arxiv.org/html/2410.10816v1 。
【官方一手·代码与数据（含仅能从代码得到的关键参数）】8) https://github.com/snap-research/Panda-70M （splitting/captioning README、dataset_dataloading README 中的 10M/2M 筛选口径、panda70m.yaml 的 download_size:360 与 download_audio:True）；9) https://github.com/OpenGVLab/InternVideo/tree/main/Data/InternVid 与 https://huggingface.co/datasets/OpenGVLab/InternVid ；10) https://github.com/KlingAIResearch/Koala-36M （原 KwaiVGI/Koala-36M）中的 trainsition_detect/VideoTransitionAnalyzer.py——**SVM 系数、Canny 阈值、3σ 门限、±4帧腐蚀等仅存在于代码而不在论文中**；training_suitability_assessment/test.yml 暴露了发布版打分器实为 fragments-only；11) https://github.com/mira-space/MiraData/tree/v1 的 caption_gpt4v.py（GPT-4V 完整 prompt 原文）与 calculate_score.py（MiraBench 实际输出19个指标）；12) https://github.com/NJU-PCALab/OpenVid-1M 与 https://huggingface.co/datasets/nkp37/OpenVid-1M ；13) https://github.com/xzc-zju/UltraVideo 与 https://huggingface.co/datasets/APRIL-AIGC/UltraVideo （license-april-lab.txt）；14) https://github.com/SilentView/LVD-2M 与 S3 元数据 https://ic-cv-long-videos.s3.ap-northeast-2.amazonaws.com/LVD-2M/data/ 。
【同团队旁证/上游】15) HD-VILA-100M https://github.com/microsoft/XPretrain/tree/main/hd-vila-100m 与 arXiv:2111.10337——Panda-70M、Koala-36M、LVD-2M（经 HD-VG）共同的上游语料与许可来源。
【第三方/交叉互评（重要：各数据集论文互相批评，是本调研的核心证据链）】16) Koala-36M 论文 Table 1 与相关工作对 Panda-70M「caption 简短不完整、转场导致语义不一致」的批评；17) OpenVid-1M 论文对 WebVid-10M/Panda-70M「过度追求规模导致低质视频与短而不准 caption」的批评；18) MiraData 论文对 Panda-70M「未系统考虑正确切分、画质过滤与准确文本标注」的批评；19) UltraVideo 论文 Table 1 提供了七者中六者的统一口径对比（本调研多处交叉校验依赖该表）；20) LVD-2M 论文对 InternVid caption「缺乏时序动态」的批评；21) VidGen-1M arXiv:2408.02629（「coarse-to-fine 精修 Panda-70M」，佐证原始 Panda-70M 噪声过大）；22) 长视频生成综述 arXiv:2412.18688（**注意：该综述称 MiraData 为「77k long videos」系误引 v0 beta 数据，勿采用**）。
【社区反馈】23) Panda-70M GitHub issues #16/#41/#60/#62/#64/#65（YouTube 下载失败、私有化、需要种子镜像）；24) InternVideo issue #81（InternVid 下载困难）；25) OpenVid-1M issues #11/#21（zip 分片与视频对应关系缺失，社区自建 phil329/OpenVid-1M-mapping 补救；HD 子集下载方式混乱）；26) UltraVideo issues #3（caption prompt 未公开，关闭且无回复）/#5（108个主题与视频的映射关系缺失，未回复）/#6（VBench 子集构成，未回复）/#8（训练与清洗代码，未发布）。

## 数据规模与分布

### 训练数据量级（视频条数/小时数/token数，预训练与SFT分开） ⚠️

七者规模跨越四个数量级，且**规模与质量呈明显反向排列**（下列按clip数降序，口径为各自论文Table 1；UltraVideo Table 1 提供了统一交叉校验）：
1) **InternVid**：7.1M 源视频 → **234M clips**，760.3K 小时，caption 总计41亿词，平均clip 11.7秒、caption 17.6词；源视频平均时长351.9秒。子集：InternVid-10M-DIV（多样性采样1000万）、InternVid-10M-FLT（在DIV基础上取UMT-SIM分**前30%**）、InternVid-18M-AES（美学分≥4，1800万）。
2) **Panda-70M**：3.79M 源长视频 → **70,817,169 clips**（论文口径）/ **70,723,513**（实际发布口径，差约9.4万条系发布前的有害内容过滤，作者未明说，属推断），166.8K 小时，约36TB，平均clip **8.477秒**、caption **13.2词**。发布子集：10M（matching_score>0.43 且每源视频最多3条，10,473,922条/37.0K小时）、2M（从10M中每源视频恰取3条，800K×3=2,400,000条/7.56K小时）、验证/测试各2000源视频×3条。**注意论文里的「Panda-2M/Panda-5M」是随机子集，与仓库发布的质量过滤版2M/10M不是一回事，引用时极易混淆。**
3) **Koala-36M**：**36M clips**，172K小时（Table 1），平均clip 13.75秒、caption **202.1词**，720p。**该表存在自洽性问题：36M×13.75s=137.5K小时，与172K小时相差约25%**（UltraVideo Table 1 转引时写作137K小时/13.6秒，即采信了乘积口径）。另外论文摘要反复说「over 10M」，那是与 MiraData/VidGen/OpenVid（均≤1M）对比时的门槛表述，不是第二个计数。
4) **LVD-2M**：**2M clips**（发布约2.1M行），每条≥10秒，平均20.2秒、caption 88.7词，平均光流分47.8。**总时长论文未给**，按2M×20.2s推算约**11,200小时**（UltraVideo 表转引为14.6K小时/20.2秒/2.1M）。时长分布：10–15s约43.5%、15–20s约23%、20–30s约20.5%、30–50s约11%、>50s约2.5%。
5) **OpenVid-1M**：**1M clips**，其中 **OpenVidHD-0.4M = 433K 条 1080p**；按 UltraVideo Table 1：OpenVid-1M 平均7.2秒、2.1K小时、caption 126.5词；OpenVidHD 平均9.6秒、1.2K小时、caption 104.5词。HF 数据集卡显示「1.45M rows」（含HD子集重复计数）。**总时长/平均时长/FPS 原论文未披露**。
6) **MiraData**：未过滤池 **788K**，发布四档嵌套子集 **330K / 93K / 42K / 9K**。平均clip **72.1秒**（七者中最长）、caption **318词**，720p。**Table 1 的「16K小时」对应的是788K未过滤池（788K×72.1s≈15,785h），不是330K发布版——330K 按同口径约6,600小时**，这是极易误引的一处。（外部综述所称「77k long videos」系误引 v0 beta 的57,803条/1,754小时，勿用。）
7) **UltraVideo**：short 分割 **42,184 clips / 62小时 / 平均5.3秒 / caption 824.2词**；long 分割 **16,597 clips / 143小时 / 平均30.9秒 / caption 850.3词**。源视频仅 **5,000 条**。规模比 Koala-36M 小约2700倍（按小时计），但caption长约4倍、像素量高4–16倍。**注意其split命名是 short/long，不存在「UltraVideo-1K/42K」；1K/4K 指的是 UltraWan 模型的输出分辨率。**

### 数据来源构成（自有/公开数据集/网络爬取/授权采购/合成数据） ⚠️

**七者全部为公开来源二次加工，无一家使用自有版权库或采购授权数据**，且存在严重的同源套娃（HD-VILA-100M 是至少三者的共同祖先）：
- **Panda-70M**：100% YouTube，全部来自 **HD-VILA-100M**（微软，330万YouTube视频，720p，跨15个热门品类均衡采样），取其中380万条长视频重新切分。
- **Koala-36M**：论文原文「we start from the same raw data with Panda-70M」，即**同样是 HD-VILA-100M**，LICENSE 中指向 HD-VILA-100M 许可可佐证。**不是快手自有短视频**，这一点常被误解。
- **InternVid**：独立爬取 YouTube。两路：核心集约200万条来自16个品类的优质频道；另约510万条通过**约6,100条动作/活动检索词**召回，词表 = Kinetics/Something-Something/UCF101 的 **1,103** 个动作标签 + LLM 从视觉定位语料中抽取并人工校验的 **5,001** 个动作，并参照美国时间使用调查（ATUS 2017–2022）。爬取时排除截至2023年4月已存在于公开数据集中的视频以避免重叠。
- **MiraData**：四个平台 + 一个回收数据集。YouTube **156个人工挑选的频道**约6.8万条720p视频（切后约3.4万视频→约17.3万clips）；**HD-VILA-100M 回收**（约1亿clips输入，仅**19.5万条**幸存，作者以此极低留存率论证自家筛选之严）；Videvo约6.3万、Pixabay约4.3万、Pexels约31.8万。（论文脚注2把 Videvo 与 Pixabay 的网址写反了。）
- **OpenVid-1M**：从四个已有数据集**二次筛选**——**Panda-50M/70M（主力）、ChronoMagic、CelebvHQ、Open-Sora-Plan（即 Mixkit/Pexels/Pixabay 系）**。CelebvHQ 原本无caption，由本项目补标。各源精确条数论文未以表格形式给出。[不确定]
- **UltraVideo**：**仅 YouTube 4K/8K 视频池**（原文「the sole source」）。两条召回路径：(a) 从 **Koala-36M** 中按分辨率>4K、帧率>25FPS、时长>30s 复筛，再用**播放量/点赞/评论等用户行为元信号**剔除「用户不感兴趣」的视频，再按标题描述与预设主题的相似度做各类目均匀采样并去重；(b) 用 LLM 从108个主题生成检索词后**人工搜索**最新4K/8K视频。合计5,000条原始视频（时长1分钟至2小时），再经**二次人工复检**剔除低质/模糊/水印/抖动。
- **LVD-2M**：从四个已有语料共 **220M clips** 中筛选——**HD-VG-130M（1.3亿）、Panda-70M（7000万）、InternVid-38M（3800万）、WebVid-10M（1000万）**。选源逻辑写得很清楚：YouTube 系动态足但需切镜过滤（「InternVid 中仅15%的clip超过10秒，而这些长视频中约52.5%含镜头切换」），素材站系（WebVid）几乎无切镜但「近一半不够动态」。**注意：不含 HD-VILA-100M（仅在相关工作对比表中出现）、不含 Vidal-10M、不含 Ego4D。** 按发布文件名推断成分：YouTube系约60万（Panda-70M+InternVid混在一个文件）、HDVG约30万、**WebVid约120万（占比近六成）**——论文正文未给出该拆分。

### 数据合规与溯源（授权数据占比、rights-cleared数据集、C2PA等） ⚠️

**七者的合规基础都很脆弱，且质量参差**：
- **共同的根本问题**：绝大部分素材是 YouTube 视频，数据集方均不拥有版权，仅以「非商用研究」+「侵权可申请下架」两条软性条款自保。Panda-70M 与 MiraData 都写有明确的下架承诺（「We will remove the video samples from our dataset as long as you need it」）。UltraVideo 的许可证更进一步要求使用者自行遵守 YouTube ToS 与 GDPR/CCPA，并明确免责声明不保证素材权属——等于承认自己不拥有 YouTube 素材。
- **授权数据占比**：唯一可称为 rights-cleared 的成分是素材站 CC0/免费许可内容——MiraData 的 Videvo/Pixabay/Pexels 部分、OpenVid-1M 中来自 Open-Sora-Plan（Mixkit/Pexels/Pixabay）的部分、LVD-2M 中 WebVid 的 stock footage 部分（但 WebVid 本身带 Shutterstock 水印且已因版权问题下架，属高风险成分）。**没有任何一家披露 rights-cleared 数据的占比数字。**
- **不发视频本体作为规避手段**：Panda-70M / InternVid / Koala-36M / MiraData / LVD-2M 五者只发 URL+时间戳，把爬取的法律风险转嫁给使用者。InternVid 明确说明「遵循既有数据集协议只共享视频ID以符合YouTube政策」。代价是**链接腐烂（link rot）不可逆**——Panda-70M 仓库直接指导用户跳过 status=failed_to_download 的样本；LVD-2M 的下载脚本甚至内置多账号轮换（ACCOUNT_NUM）并警告 Google 账号可能被封。**无一家发布过 ID 存活率的实测数据。**
- **反向操作**：OpenVid-1M 与 UltraVideo 直接托管视频本体（12.4TB / 全量4K-8K clips），可复现性最好但法律暴露最大；UltraVideo 用「禁止二次分发原始视频、衍生品须继承同等条款」来收口。
- **隐私处理**：仅 **Panda-70M** 做了一处具体工作——用 NLTK 把 caption 中所有人名替换为「person」，并过滤含有害/暴力语言及毒品、仇恨言论的样本。其余六者均无人名脱敏、无人脸隐私处理。
- **C2PA / 水印 / 生成内容溯源**：七者**全部没有**，输出侧无任何溯源机制。
- **许可证自相矛盾**：MiraData 最严重——仓库标 GPL-3.0，README「License Agreement」段禁止一切商业用途，但同一 README 末句写「is supported for commercial usage」。可辩护的解读是：代码 GPL-3.0，数据仅供研究，视频版权仍属第三方。Koala-36M 的 HF 页面则**完全没有 license 标签也没有 dataset card**，仅 GitHub 仓库有非商用条款。

### 片段时长分布与切分策略 ⚠️

**片段时长是七者最本质的分野，直接对应「短片段易学、长片段难得」的取舍**，从8.5秒到72秒跨越近一个数量级：
- **Panda-70M（8.5秒，最短）**：切分后丢弃<2秒的片段，源视频超过60秒的截断至前60秒；另有一条关键操作——**每个片段首尾各裁掉10%**以去除不稳定镜头运动与转场残留。最终平均8.477秒。
- **InternVid（11.7秒）**：爬取时限定源视频10秒–30分钟；clip 时长2秒至30秒以上，**85%的clip落在0–10秒**。
- **Koala-36M（13.75秒，或按172K小时口径17.2秒）**：片段由转场检测自然切出，无固定长度截断；组装规则是片段需长于**8帧**才保留，且**首尾各腐蚀4帧**（cuts.append((start+4, i-4))）以确保内容内无转场——与 Panda-70M 的10%裁边是同一思路的不同实现。
- **OpenVid-1M（7.2秒，OpenVidHD 9.6秒）**：最短时长阈值未披露。[不确定]
- **UltraVideo（short 5.3秒 / long 30.9秒）**：**显式的双分割设计**——切镜后按帧数分流，3–10秒进 short 集，>10秒进 long 集。为扩充 short 集还做了定向截取：源视频<60秒的取**中间10秒**，>60秒的额外从**两侧各取10秒**。保留原生帧率不做抽帧。
- **LVD-2M（20.2秒，最低门槛10秒）**：**以「≥10秒」为一等设计目标**（四条准则之首），且不做上限截断，因此长尾丰富（>30秒的占约13.5%）。打标时才把>30秒的视频切成30秒段分别描述。
- **MiraData（72.1秒，最长）**：策略与所有其他数据集相反——**不是切得更准，而是切完再缝回去**。YouTube 来源的片段要求**>40秒**才保留，Videvo/Pixabay/Pexels 来源的要求**>10秒**。（v0 版曾把超过2分钟的硬切成2分钟段，v1 改为拼接策略。）实测官方100条样本集平均105.1秒，比数据集均值更长。
**汇总规律**：以 clip 长度换质量的三条路线清晰可辨——Panda/InternVid/OpenVid 走「短而多」，Koala 走「中等且转场干净」，LVD-2M/MiraData/UltraVideo-long 走「长镜头稀缺资源」。

### 分辨率/宽高比分布与分桶策略 ⚠️

**分辨率维度上七者高度趋同于720p，唯有 UltraVideo 与 OpenVidHD 例外**：
- **720p 一档（五家）**：Panda-70M（继承 HD-VILA-100M 的全720p；但注意**默认下载配置 download_size:360 实际只给360p**，不改配置拿不到720p，这是复现时的常见坑）；Koala-36M（720p）；MiraData（720p，实测样本1280×720@30fps）；LVD-2M（**论文从未把分辨率作为数据集属性说明**，因不托管视频；下载脚本默认 --resolution=720p，WebVid 分片文件名标 336，唯一出现的分辨率是处理参数——RAFT 光流在时间2fps、空间520×960 下计算）；InternVid（**85% 为720P**，其余15%为360P–512P，爬取时限定360P–720P）。
- **1080p**：OpenVid-1M 设**最低分辨率门槛 512×512**，OpenVidHD-0.4M 子集为 1080p（433K条）。宽高比与FPS分布论文均未披露。[不确定]
- **4K/8K（UltraVideo，唯一）**：short 集 4K **32,727** 条 + 8K **9,457** 条（8K占22.4%）；long 集 4K 12,277 + 8K 4,320。帧率只分「≤30 FPS」与「≥50 FPS」两桶（short：31,027 / 11,157；long：8,146 / 8,451），**31–49 之间是空的**。HF 数据集元信息显示帧尺寸 3,840–7,680 × 1,600–4,320，12–60 FPS，每clip 36–600帧。**保留原生分辨率与帧率不做转码**，作者以此论证可服务插帧/编解码研究。码率与编码格式全文未提及。
**分桶策略**：七者**均无训练时的分辨率/宽高比分桶（bucket）设计**——它们是数据集而非训练框架，分桶由下游使用者（如 Open-Sora）自行实现。UltraVideo 附带的 UltraWan 训练是固定尺寸的（1088×1920×81帧 / 2160×3840×29帧）。
**一个值得注意的缺口**：七者**没有一家给出宽高比分布统计**，也没有一家做竖屏内容的定向补充（对比 Open-Sora Plan 用 VIDAL 补竖屏），全部隐含以16:9横屏为主体。

### 类别/domain分布与配比策略（人物、动作、场景、风格等比例控制与概念均衡） ⚠️

**七者的类目治理水平差距极大，从「完全没有」到「7主题108子类的显式taxonomy」**：
- **Panda-70M（无类目体系）**：论文**没有品类饼图与百分比**，附录E仅列出8个用于可视化举例的类别（动物、风景、美食、体育活动、交通工具、教程与叙事、新闻与电视节目、游戏与3D渲染），无任何数字；图13是100K条caption的词云。作者在局限性中主动承认：由于 HD-VILA-100M 偏「口播密集型」，**主要类目实为新闻、电视节目、纪录片、第一视角视频、教学与叙事视频**。若需类目分布只能引用上游 HD-VILA-100M 的15类均衡采样。
- **Koala-36M（无类目体系）**：同源于 HD-VILA-100M，论文亦未给类目分布。
- **InternVid（16个主题，但仅有饼图）**：类目分布仅以图3饼图呈现，**正文与附录均无数值表**，无法引用精确百分比。可引用的分布是源视频时长（≤5分钟49%、5–10分钟26%、>20分钟仅8%）与国别（英美澳日韩中俄法等）。作者主动说明因版权原因**刻意稀缺或排除了监控录像、体育赛事、电影、纪录片**四类。
- **MiraData（7类YouTube taxonomy，仅柱状图）**：(1) 3D引擎渲染场景、(2) 城市/风景漫游、(3) 电影、(4) 第一人称视角、(5) 物体创造/物理规律演示、(6) 延时摄影、(7) 人体动作展示。**刻意超采(1)和(3)**，理由是「多样性更好、画质更高」，并主张3D引擎素材有助于学习物理规律一致性。**每类计数仅存在于图2的无标注柱状图**（纵轴0–12K视频 / 0–78K clips），无数值表；但元数据CSV的 source 列实际编码了平台+类目（如 `youtube, 3D engine-rendered scenes`、`videvo, nature` 等），**可自行 value_counts 还原精确成分**——这是本次调研中唯一可用的类目还原路径。注意：**没有 GTA-V 专项来源**，游戏内容归入「3D引擎渲染场景」，论文刻意不点名具体游戏（应为版权考虑）。
- **LVD-2M（8类，有精确百分比，最实用）**：用 **BART** 对caption做分类——风景24%、人物23%、交通13%、体育11%、动物11%、美食9%、游戏5%、其他4%。这是七者中**唯一给出完整数值化类目分布**的数据集。
- **UltraVideo（7大主题×108子类，体系最完整但数值缺失）**：taxonomy 由「对 Koala-36M 的caption做名词统计 → LLM 归纳 → 人工修订确认」得到。七大主题为：**视频场景、主体、动作、时间事件、镜头运动、视频类型、情绪**。这是七者中唯一把「镜头运动」「情绪」纳入数据配比维度的体系，并据此做**各类目均匀采样**（明确写明按主题相似度做 uniform per-category sampling）。**但108个子类从未在正文中列出，仅在图4(a)以占比图呈现**；GitHub issue #5 专门追问主题↔视频的映射关系，至今无人回复。
- **OpenVid-1M**：无类目体系披露。[不确定]
**总体判断**：只有 UltraVideo 做了「先定taxonomy、再按类目均匀采样」的主动配比设计，MiraData 做了粗粒度的类目定向超采，LVD-2M 只做了事后统计，其余四家的类目分布完全是上游语料分布的被动继承。**七者全部没有做长尾概念均衡（concept balancing）或语义聚类后重采样。**

### 音频类别分布与配比（语音/音效foley/音乐/环境音/静音的比例及控制策略）——AV模型独有维度

不适用。七个数据集均为纯视觉+文本数据集，不生成也不处理音频，无语音/音效/音乐/环境音的分类、配比或统计。仅 Panda-70M 与 UltraVideo 的视频文件被动保留了原始音轨（前者通过 download_audio:True，后者声明保留 native audio），但均无任何音频类别标注或配比控制；Koala-36M 的重切分代码则会直接丢弃音轨。

### 叙事结构分布（单镜头vs多镜头、平均clip时长、镜头数分布、是否含原生音轨）

**七者对「镜头数」的处理呈现两条截然相反的路线，这是本次调研最有价值的对比维度之一**：
【路线A：严格单镜头，把多镜头当噪声剔除——六家】
- Panda-70M：PySceneDetect 切镜 + ImageBind 特征约束，目标是单镜头且语义一致；但其2024年10月追加发布 **TransNetV2 镜头边界标注**（每clip一个区间列表，长度为1即单镜头）——**这本身就是承认原pipeline仍漏了多镜头片段**。
- InternVid：PySceneDetect（阈值27）切镜。
- Koala-36M：自研 Color-Struct SVM + 3σ 概率门限，**召回率0.9395**（刻意偏召回，「rarely misses transitions」），是七者中转场剔除最彻底的。
- OpenVid-1M：级联切镜检测器把多场景源视频切成单场景clip。
- UltraVideo：PySceneDetect AdaptiveDetector 两轮 + DINOv2 首尾5帧相似度补捉溶解转场。
- LVD-2M：**「无镜头切换的长镜头」是其四条准则之一**，用低采样率PySceneDetect把渐变转场也当硬切滤掉。人工评测显示其长镜头纯净率 **77.5%**（不含WebVid口径）/ **86.8%**（含WebVid全量口径），显著高于 HD-VG 55.0%、Panda-70M 50.0%、InternVid 47.5%——**这组数字是「主流数据集近半数片段含镜头切换」的最直接第三方证据**。作者坦承残余失败模式是「轻微跳切」，PySceneDetect 与 MLLM 都抓不到。
【路线B：先过切再缝合，主动保留长连续镜头——一家】
- MiraData：故意用**低阈值26**的 PySceneDetect 过度切分（「ensuring that all distinct clips are extracted」），再用**四模型投票**把相邻片段缝回去（详见 shot_segmentation 字段）。这是七者中唯一把「长镜头稀缺性」当作核心资产来经营的。作者在 v0 README 中主动承认局限：「本次发布中描述同一场景多机位的片段不多」。
【平均镜头数与原生音轨】七者训练片段的平均镜头数均为1（设计目标），因此**均不含跨镜头一致性监督信号**，也无法直接支撑多镜头叙事生成。原生音轨：仅 Panda-70M（可选）与 UltraVideo 保留，但均不做任何利用。
【平均clip时长】见 duration_distribution：8.5s（Panda）/ 11.7s（InternVid）/ 13.75s（Koala）/ 7.2s（OpenVid）/ 20.2s（LVD-2M）/ 5.3s与30.9s（UltraVideo双分割）/ 72.1s（MiraData）。

### 语言/口音分布（多语种唇同步能力的数据基础）

语音层面全部不适用（无音频建模）。文本层面：
- 七者的 caption **全部为英文单语**，无一家提供中文或多语caption，也无一家做多语prompt的数据建设。
- **InternVid 是唯一涉及多语数据的**：其内部采集包含 **11种语言的 ASR 字幕**（论文图16示例了英语、中文、韩语、德语），源视频国别覆盖英美澳日韩中俄法等。但**这些 ASR 转写既未用于生成caption，也未随数据集发布**——发布的 jsonlines 中无音频、无ASR、无字幕字段。这是一处被采集却被丢弃的多语资源。
- **Panda-70M 使用了字幕但仅作输入**：英文字幕（含 YouTube 自动字幕）被送入教师模型作为文本侧上下文（video2dataset 配置 subtitleslangs:['en'], writeautomaticsub:True），字幕本身不作为 Panda-70M 的字段发布，仅落在每clip的JSON元数据中。
- 口音、说话人身份、唇同步相关的数据基础：**七者全部为零**。
- 一个连带影响：由于 caption 均为英文且普遍冗长（Koala 202词、MiraData 318词、UltraVideo 824词），使用这些数据训练时**必须搭配长上下文文本编码器**——MiraData 因此明确弃用 CLIP 的77 token 编码器改用 **Flan-T5-XXL（512 token）**；LVD-2M 则在实验中吃了亏，其88.7词caption被冻结CLIP文本编码器的77token截断，作者将 I2V 文本匹配提升不明显直接归因于此。

## 清洗流程

### 清洗漏斗整体结构（几级过滤、各级顺序）

**七条漏斗的结构差异，本质是「用什么当质检员」的差异**，可归纳为三代技术水位：
【第一代·纯启发式打分器 + 阈值】
- **InternVid（最简）**：爬取限定（时长10s–30min、分辨率360P–720P、排除已有公开数据集）→ NSFW二分类器 → PySceneDetect(阈值27)切镜 → 剔除静止与极端动态片段 → 多尺度打标 → 事后用 UMT-SIM 与美学分派生子集。**过滤与打标基本无阈值披露，可复现性最低。**
- **Panda-70M（切分为核心，过滤极轻）**：其漏斗重心不在过滤而在**切分**——PySceneDetect(阈值25, min_scene_len=15帧) → 长于5秒的递归切出前5秒（发布代码改为7秒） → ImageBind首尾特征距离≤1.0保留 → 相邻片段距离≤0.6则缝合 → 丢弃<2秒及运动过小(距离≤0.15)者 → 与前序片段距离>0.3去重 → 首尾各裁10% → 八教师打标 → UMT检索模型选caption。**没有美学过滤、没有清晰度过滤、没有OCR过滤**——这是它后来被 OpenVid/Koala/MiraData 集体批评的根源。
- **OpenVid-1M（比例式过滤，四道）**：源数据集汇集 → 级联切镜 → LAION美学分（Panda-50M取**前20%**，其余三源取**前90%**）→ CLIP相邻帧余弦相似度双向剔除（过高=近静止、过低=闪烁）→ UniMatch光流双向剔除 → DOVER-Technical清晰度取**前30%** → LLaVA-v1.6-34B重打标。特点是**全部用百分位而非绝对阈值**，可移植性好但绝对质量不可控。
【第二代·MLLM 作为语义质检员】
- **LVD-2M（三级，全靠MLLM兜底语义）**：>10秒长度门 → PySceneDetect **ContentDetector(阈值50, min_scene_len=0, 0.5fps采样)** 剔切镜 → RAFT光流(2fps, 520×960)均值**<20则丢弃** → **PLLaVA-7B 取8帧做 GOOD/BAD 二分类**，两个prompt分别管「内容变化度」与「视觉多样性+文字占屏」。**没有美学打分器、没有OCR模型**——把这两件事全部外包给了MLLM的语义判断，是七者中最早、最彻底的「VLM-as-judge」实践。
- **UltraVideo（四阶段，统计+模型双层）**：源头控制（5,000条4K/8K，人工复检）→ PySceneDetect AdaptiveDetector两轮 + DINOv2首尾5帧补捉溶解 + 按帧数分流short/long → **统计过滤**（PaddleOCR文字面积、黑边、过曝、灰度四项，统一采用「坏帧率>5%则丢弃整条」的模式）→ **模型净化**（VTSS<0.01剔除、RAFT光流保留0.1–100、VideoCLIP-XL-v2图文相似度<0.2剔除、**Qwen2.5-VL-72B 对16种低质属性做二元判断，命中任一即删**）→ Qwen2.5-VL-72B 十维打标 + Qwen3-4B 汇总。
【第三代·把多个子指标喂给一个学习出来的打分网络】
- **Koala-36M（唯一一家反对多阈值范式）**：与 Panda-70M 同源数据 → Color-Struct SVM 转场检测重切分 → LLaVA系结构化打标 → **单一 VTSS 打分门限2.5**。其核心论证是**子指标彼此不正交**（清晰度-美学 Pearson 0.3774、清晰度-运动 −0.4028、运动-美学 −0.2515），多阈值串联会累积误差（附录D表8：仅清晰度阈值偏移10%误滤25万条/4800万，三个阈值同时偏移10%误滤34万条），因此改用一个**吃「视频像素 + 子指标」联合输入的网络直接回归出一个标量**。
【MiraData（结构最特殊：过切→缝合→筛选→打标）】
收集 → PySceneDetect(阈值**26**，刻意低)过度切分 → **四模型投票缝合** → 四项筛选（画面色彩、LAION美学、RAFT运动强度、Stable Diffusion Safety Checker）在**统一2fps**下计算，产出330K/93K/42K/9K四档递增严格度 → Panda-70M短caption做hint + GPT-4V一轮对话产出5个结构化字段。
**横向结论**：过滤严格度排序大致为 UltraVideo > MiraData(9K档) > Koala-36M ≈ LVD-2M > OpenVid-1M > InternVid > Panda-70M；而「质检员智能程度」排序为 UltraVideo(72B VLM) > LVD-2M(7B VLM) > Koala-36M(学习式打分网络) > OpenVid/MiraData(浅层打分器) > Panda-70M(几乎无)。

### 漏斗定量保留率（每级过滤的输入/输出量与最终保留率，如Apollo 27%） ⚠️

**定量披露程度差异巨大，只有三家给出了可用的漏斗数字**：
- **UltraVideo（披露最完整）**：5,000条原始视频 → 切镜与帧数过滤后 **62K short + 25K long** → 统计过滤后 **46K short + 19K long**（保留率约**74% / 76%**）→ 模型净化后 **42K short + 17K long**（保留率约**91% / 89%**）。**从切分后池子算起，short 整体保留率约68%**。但每一项检查各自剔除多少条未报告。
- **Koala-36M（给出了关键的两点）**：重切分+打标后未过滤池 **48M clips**（该数字出自附录D表8的分母）→ 人工多阈值过滤基线 **37M**（77.1%）→ **VTSS>2.5 最终 36M（75.0%）**。即质量过滤这一级保留率为**75%**，且 VTSS 比人工多阈值**略更严格**（36M vs 37M）却保住了更多高质样本。**从103M源视频到48M clips 这一段的切分产出率未报告。**
- **LVD-2M（只有首尾）**：**220M clips 输入 → 2M 输出，整体保留率约 0.91%**——七者中最低。**逐级保留率完全未报告**（论文图2只是示意图，无计数）。唯一的中间统计是对 InternVid 的诊断：仅15%的clip超过10秒，而这些长视频中约52.5%含镜头切换。
- **MiraData（有分档但无阈值）**：未过滤池 **788K** → 330K（41.9%）→ 93K（11.8%）→ 42K（5.3%）→ 9K（1.1%）。另有一个极具说服力的单点数字：**HD-VILA-100M 约1亿clips输入，最终仅19.5万条幸存（约0.2%）**，作者以此论证自家人工挑选的YouTube频道质量远高于通用爬取。**但四档之间的具体阈值论文称在补充材料中，而补充材料实际不存在**（我核查了 arXiv v1 附录、NeurIPS camera-ready 与补充ZIP链接，均无阈值），这是一处真实的可复现性缺口。
- **OpenVid-1M**：只有定性描述（Panda-50M候选池 → 过滤得 Ours-0.4M → 切分扩展为 Ours-0.6M → 合并其余三源达到约1M），**各级绝对数量与整体保留率未披露**。[不确定]
- **Panda-70M**：380万源视频 → 7,081.7万clips，这是**扩张比（约18.7倍）而非保留率**——它几乎不做质量过滤。唯一的过滤性数字是发布口径的 70,723,513（较论文少约9.4万条，系有害内容过滤，作者未明说）与 **89.6% 的样本 matching_score>0.43**。另有2024年10月追加的 desirability 分布（全量占比）：desirable **80.5%**、低期望分5.28%、静止前景6.82%、微小相机运动1.20%、画中画5.03%、屏幕录制1.13%——**即按其自评仍有约19.5%的样本是不理想的**。
- **InternVid**：**完全未披露任何漏斗数字**。[不确定]
**可引用的经验值**：跨数据集看，「从通用网络视频池筛到可用训练集」的保留率量级为——不做质量过滤约100%（Panda）、做常规质量过滤约70–75%（Koala、UltraVideo统计层）、做严格质量+长镜头筛选约1%（LVD-2M 0.91%、MiraData对HD-VILA的0.2%）。**「长镜头+高动态+无切换」这一组约束的代价是三个数量级的数据损耗**，这是本次调研最强的定量结论之一。

### 镜头切分方法（PySceneDetect/自研模型/shot-aware splitting） ⚠️

**这是七者技术分化最明显的一环，可分为四种路线**：
【路线1·PySceneDetect 直接用（参数各异，且参数选择本身即方法）】
- **InternVid**：ContentDetector 阈值 **27**，无其他修饰。
- **MiraData**：content-aware 阈值 **26**——刻意调低以**故意过度切分**（「ensuring that all distinct clips are extracted」），为后续缝合留料。
- **LVD-2M**：**最巧妙的参数化**。明确弃用 AdaptiveDetector + 滚动均值（「难以检测带淡入淡出的缓慢镜头切换」），改用 **ContentDetector，阈值50，min_scene_len=0帧，帧采样率 0.5fps**。机理是：0.5fps下相邻采样帧相隔2秒，**任何2秒内的显著变化都会被判为切换，从而把通常在2秒内完成的淡入淡出也一并滤掉**。用采样率而非检测器复杂度来解决渐变转场，是低成本高收益的设计。
- **UltraVideo**：AdaptiveDetector **两轮 + 滚动均值**以降低相机运动造成的误切，再用 **DINOv2 对每个clip的首尾各5帧**做特征相似度，补捉 AdaptiveDetector 漏掉的溶解转场。
【路线2·PySceneDetect + 语义特征二次约束与缝合（Panda-70M）】
PySceneDetect ContentDetector(阈值25, min_scene_len=15帧) → 对超过5秒的clip**递归切出前5秒**（用于处理无剪辑的连续拍摄素材；发布代码把5秒改成了7秒「以获得更好的切分效果」，**要精确复现论文需改 event_stitching.py 第199/200行**）→ 取每个n帧clip在 **0.1×n 与 0.9×n** 处的 **ImageBind** 特征，**首尾距离≤1.0** 才保留 → 相邻clip的 **‖f(C¹尾)−f(C²首)‖≤0.6** 则缝合。效果由其自创的 **Max Running LPIPS** 指标衡量（1fps关键帧间LPIPS的最大值）：字幕对齐法0.408/11.8秒、纯PySceneDetect **0.247**/4.1秒、本方法 **0.256/7.9秒**——**即以0.009的一致性代价换来近2倍的片段长度**，这是该论文最漂亮的一条量化论证。
【路线3·自研学习式转场检测（Koala-36M，唯一）】
**Color-Struct SVM（CSS）+ 时序3σ门限**，两阶段：
(a) 训练数据自监督构造——**同一视频内的帧对为负例（无转场）、不同视频的帧对为正例（有转场）**；两个特征：`d_color` = BGR直方图相关性（帧resize到256×256，每通道254 bins，范围[1,255]），`d_struct` = 在Canny增强的亮度图上算SSIM（E=max(Gray(I), Canny(Gray(I)))，灰度图resize到128×128，Canny(100,200)，data_range=255）；线性SVM，**发布代码中给出了拟合好的精确系数**：`svm_score = 4.61480465×bgr_similarity + 3.75211168×canny_similarity − 5.485968377115124`。
(b) 时序概率门限——对SVM分做**3抽头滑动平均**得 conv_svm；`svm_score<0` 直接判硬切；否则对帧号≥8且 conv_svm<0.75 者，用**前8帧**估计 μ、σ（先排序**去掉首尾各20%再算**，鲁棒统计），当 `conv_svm[i] < μ − 3×max(0.2, σ)` 判为转场（**σ下限0.2**防止退化）。片段需长于8帧才保留，且**首尾各腐蚀4帧**。
性能（1万条人工标注clip，约半数含转场）：**准确率0.7741 / 召回0.9395 / 精确率0.7547**，对比 PySceneDetect(HSL) 0.4421/0.3096/0.5920、PySceneDetect(HSL+edge) 0.4574/0.4146/0.5854。**换算F1：0.838 vs 0.407 / 0.485。** 速度上因特征提取固定降采样到256/128，在高分辨率上反超：1080p 12.26ms vs PySceneDetect-HSL 26.16ms、4K 41.98ms vs 102.55ms（对HSL+edge在4K快6.4倍），但**在256分辨率上反而更慢**（1.42 vs 0.68ms）。
【路线4·过切后用多模型投票缝合（MiraData，唯一）】
相邻短片段用**四模型「两两成对、组内需一致」的表决**重新合并：视觉语言模型组 **Qwen-VL-Chat + LLaVA**（判断「是否同一场景」），图像特征组 **ImageBind + DINOv2**（判断「特征是否相似」）。规则原文：「A connection is made only if **both** vision language models **or both** image feature extraction models agree」，即 (VLM₁∧VLM₂) ∨ (Feat₁∧Feat₂)。设计理由：VLM 擅长识别内容连贯的转场，特征相似度擅长修复被错误分离的片段。**这是七者中唯一把「缝合」而非「切分」当作核心动作的 pipeline**，也是 MiraData 能拿到72秒平均时长的直接原因。
【路线5·未披露细节】OpenVid-1M 仅称使用「级联的镜头/切点检测器」，具体模型与阈值未披露。[不确定]

### 质量过滤（美学评分、清晰度、OCR文字过滤、黑边/水印/logo检测） ⚠️

**按「美学 / 清晰度技术质量 / 文字OCR / 黑边水印」四类横向对比**：
【美学评分】
- **Panda-70M：无。** 这是其最大缺口。
- **InternVid**：LAION 美学预测器，**≥4** 得到 InternVid-18M-AES 子集（仅用于派生子集，非主漏斗）。
- **MiraData**：LAION-Aesthetic 预测器，在统一 **2fps** 下计算，**阈值未公开**（论文称在补充材料，补充材料不存在）。[不确定]
- **OpenVid-1M**：LAION 美学分，**Panda-50M 源取前20%，其余三源取前90%**——用百分位而非绝对分。
- **LVD-2M：不用美学打分器**，改由 PLLaVA-7B 的「视觉多样性」prompt 做语义判断（全文「aesthetic」仅作为VBench指标名出现一次）。
- **Koala-36M**：美学分是 VTSS 的输入子指标之一（CSV中 aesthetic_score 实测范围 **2.28–6.56**），但**不单独设阈值**。
- **UltraVideo：不用 LAION 美学分也不用 DOVER**，直接复用 **Koala-36M 的 VTSS**（原生分线性缩放到 −0.0575~0.0728），**VTSS<0.01 剔除**。
【清晰度 / 技术质量】
- **OpenVid-1M**：**DOVER technical 分取前30%**。
- **Koala-36M**：clarity_score 作为 VTSS 子指标（CSV实测范围 0–1）；论文用 DOVER 与 FastVQA 作为 TSA 网络的对比基线（TSA PLCC 0.8974 vs DOVER 0.8554 vs FastVQA 0.8684）。
- **MiraData**：**「画面色彩」检查**——计算平均色彩以及最亮/最暗80%帧的色彩，剔除过亮/过暗素材。这是七者中唯一显式的曝光/色彩层面过滤（与UltraVideo的过曝检查同类）。
- **Panda-70M / InternVid / LVD-2M：无专门的清晰度/技术质量打分器。**
- **UltraVideo：四项统计检查，全部采用「坏帧率>5%则丢弃整条视频」的统一模式**——(a) 黑边：从四边各向内延伸**3%**的矩形区域均值**<3** 判坏帧；(b) 过曝：像素值 **>250 或 <5** 的比例**>12%** 判坏帧；(c) 灰度化：逐像素RGB方差的全图均值 **<1.2** 判坏帧。这是七者中**唯一给出全部绝对阈值数值的统计过滤方案**，可直接复用。
【OCR / 文字过滤】
- **UltraVideo**：**PaddleOCR**，文字最小外接矩形并集面积 / 图像总面积 **>2%** 判坏帧，坏帧率>5%丢弃。
- **LVD-2M**：**不用OCR模型**，由 PLLaVA-7B 的第二个prompt判断「Text Presence: 文字叠加是否主导画面以致损害观感」。
- **Panda-70M / InternVid / Koala-36M / MiraData / OpenVid-1M：均无OCR文字过滤**（Koala 论文全文无 OCR，与常见猜测相反；OpenVid 未披露）。
【黑边 / 水印 / logo / 转场特效】
- **UltraVideo（唯一系统处理）**：黑边有专门的统计检查（见上）；水印、转场特效、分屏、屏幕录制、画中画等**16种低质属性**交由 **Qwen2.5-VL-72B 逐项二元判断，命中任意一项即删除整条**。另在源头做过一次人工复检剔除水印/模糊/抖动。
- **Koala-36M**：字幕、logo、特效、转场仅作为**人工标注员的评分指引**（「视频自然度」维度），通过200K标注间接、有损地传递给 VTSS，**没有独立的检测器**。
- **其余五家：均无黑边/水印/logo检测。** 其中 LVD-2M 使用了 WebVid 素材（带 Shutterstock 水印且该数据集已因版权下架），是明确的水印污染风险来源。
**一条重要的反直觉发现**：被广泛认为「必备」的美学打分+OCR组合，在最新的两个数据集（LVD-2M、UltraVideo）中正在被替代——LVD-2M 用7B VLM 全面替代，UltraVideo 用「精确统计阈值 + 72B VLM 属性判断」的组合替代，**都不再依赖 LAION 美学分**。

### 运动过滤（光流/运动分数阈值、静态与抖动剔除） ⚠️

**七者全部做运动过滤，且全部是双向的（既剔静止也剔抖动/剧烈），但度量工具分三派**：
【光流派】
- **LVD-2M**：**RAFT**，在时间 **2fps**、空间 **520×960** 下对相邻采样帧计算，取时空平均的光流幅值，**<20 则丢弃**（实测官方100条样本最小值20.19，完美印证该阈值；样本均值51.91、最大201.33；数据集整体均值47.8）。**这是七者中唯一给出可验证绝对光流阈值的数据集。** 目的是剔除静态场景与「静止背景前的口播」。作者同时指出光流的局限：**手持拍摄的抖动视频光流分很高但没有有意义的运动**——这正是他们额外加一层 PLLaVA 语义判断的动因。
- **MiraData**：**RAFT** 帧间光流作为「运动强度」筛选项，在统一2fps下计算，**阈值未公开**。[不确定] 另在评测端提出 **Tracking Strength（CoTracker 追踪点相对首帧的平均位移）** 作为光流动态度的修正——论文图5给出反例：某对视频光流动态度为1.2 vs 0.7（排序错误），而 Tracking Strength 为4.1 vs 11.8（排序正确），说明**光流会把相机抖动和往复局部运动误判为高动态**。
- **OpenVid-1M**：**UniMatch** 光流，剔除运动差异分最高与最低的两端，保留「适中」运动。阈值未披露。[不确定]
- **UltraVideo**：**RAFT** 按间隔采样后取全局平均，保留 **0.1 ≤ 分数 ≤ 100**。
【感知/特征距离派】
- **Panda-70M**：不用光流，用 **ImageBind 首尾特征距离**——距离 **≤0.15** 视为运动过小而丢弃（同时距离 >1.0 视为语义变化过大也丢弃），即用同一个特征距离同时承担「运动下限」与「一致性上限」两个职能。此外还用「与前序片段距离 >0.3」做多样性去重。
【学习式打分派】
- **Koala-36M**：motion_score 作为 VTSS 的三个输入子指标之一（CSV实测范围 **0.01–267**），**不单独设阈值**，由 VTSS 网络综合判断。其人工标注准则中对运动的定义值得引用：**「运动区域需覆盖画面30%以上，否则因动态不足而降分」**，并区分「业余相机抖动」与「专业运镜」分别惩罚/奖励。
- **InternVid**：仅定性描述「过滤掉静止或极端动态（如相册浏览）的片段」，**无模型无阈值**。[不确定]
- **LVD-2M 的人工验证结果**可作为运动过滤有效性的横向标尺（40条/数据集，3档评分，「非常动态/中等/不动态」）：**LVD-2M 30.0%/62.5%/7.5%**、HD-VG 20.0%/37.5%/42.5%、InternVid 15.0%/60.0%/25.0%、Panda-70M 7.5%/67.5%/25.0%、WebVid 7.5%/42.5%/50.0%。**Panda-70M 与 WebVid 各有约四分之一到一半的片段被人类判为「不动态」**，这是「大规模数据集普遍缺乏运动」的直接证据。
【共同缺口】七者**均未把运动分作为可控条件写入caption或训练条件**（对比 Open-Sora 把 motion score 拼进caption、Koala-36M 把三个分数经AdaLN注入——注意后者是其**生成模型**的设计而非数据集本身的标注策略，但 Koala-36M 的 CSV 确实发布了三个分数供下游做条件控制，这是七者中唯一提供了可用于条件注入的结构化分数列的数据集）。

### 去重方法（精确去重与基于embedding的语义去重分别记录） ⚠️

**七者的去重普遍薄弱，只有两家做了实质性工作，且都不是embedding级语义去重**：
- **Panda-70M（唯一有片段级去重机制）**：以每个clip第一阶段 ImageBind 特征的均值作为表示，**只保留与前序片段欧氏距离 >0.3 的片段**——这是切分流程内嵌的片段内冗余去重。另外在发布层面用「**每个源视频最多3条clip**」的规则做源级去重（10M子集），2M子集则是每源视频恰好3条。但**全量70M平均每源视频约18.7条clip，且数据未打散**（同一源视频的所有clip落在同一分片，仓库明确提示用户需自行shuffle），冗余风险很高。
- **UltraVideo**：在从 Koala-36M 复筛的路径中明确包含「去重」步骤（uniform per-category sampling 后 plus dedup），但**具体方法未披露**。[不确定]
- **InternVid**：**没有去重流程**，仅在爬取时排除了截至2023年4月已存在于公开数据集中的视频（这是跨数据集防重叠，不是内部去重）。
- **Koala-36M：完全没有去重。** 全文「duplicat」一词仅出现1次且在参考文献标题中（D4论文）。考虑到其48M clips 切自约1亿条 HD-VILA YouTube 视频，**近重复内容是未经处理的实质风险**。
- **MiraData / OpenVid-1M / LVD-2M：论文均未提及任何去重方法。**[不确定] 其中 LVD-2M 的风险最值得警惕——它同时使用了 **HD-VG-130M、Panda-70M、InternVid** 三个都源自 YouTube 的语料，**三者之间存在未经量化的源级重叠**（Panda-70M 来自 HD-VILA-100M，HD-VG-130M 亦为 YouTube 爬取），论文未做任何跨源去重。
- **精确去重（哈希）与语义去重（embedding聚类）**：**七者无一实现**。这是本次调研中七个数据集共同的、最一致的技术缺口，也是与工业界数据 pipeline 的显著代差。

### VLM/LLM作为质检员（多模态大模型质量打分与错配剔除，2026年从浅层打分器转向大模型语义判断的趋势）

**这是七者最能体现「2024→2025 技术代际迁移」的一条轴线，可清晰排出三代**：
【第0代·完全无模型判官（Panda-70M, 2024.02）】
过滤全部由 PySceneDetect 与 ImageBind 特征距离完成，没有任何语义级质量判断。唯一接近的是 **UMT 检索模型输出的 matching_score**（>0.43 视为图文关联强，全量89.6%达标），但它的职能是**从8条候选caption中选最优**，而非给样本打质量分做剔除——**VLM 在此仅是打标器（8个教师）而非判官**。
【第1代·浅层专用打分器 + 阈值（InternVid 2023, OpenVid-1M 2024, MiraData 2024）】
LAION美学预测器、DOVER、RAFT/UniMatch光流、CLIP相邻帧相似度——全部是输出标量的轻量专用模型，靠阈值/百分位裁切。MiraData 唯一用到大模型的地方是**缝合环节的 Qwen-VL-Chat + LLaVA 投票**（判断相邻片段是否同一场景），这是把 VLM 用于**结构判断**而非质量判断，属于过渡形态。
【第2代·MLLM 直接做二元语义裁决（LVD-2M 2024.10，最早）】
**LVD-2M 是本次调研中最早、最彻底地用 MLLM 替代打分器的案例**：**PLLaVA-7B 取8帧均匀采样，直接输出大写的 GOOD/BAD**，「只有在所有定义的指标上都被判为good的视频才保留」。两条prompt原文值得引用——(1) 内容变化度：「If the background, setting, and characters are in static states, the video lacks content variation… You must provide a capitalized either 'BAD' or 'GOOD' answer.」；(2) 视觉多样性+文字：「A visually diverse video should have rich content that is visually appealing. If the video is only some person talking to the camera with a static background, it is not diverse. And a video with only texts instead of objects is not diverse… Determine if text overlays dominate the video in a way that detracts from the visual experience.」**它以此完全取代了美学打分器与OCR模型**。动因写得很明确：光流抓不到「手持抖动虽然光流高但无有意义运动」这类问题。作者同时诚实地列为局限：「当前MLLM不保证具备视频质量评估能力，部分模型难以遵循相关指令」，并呼吁建立MLLM视频质量评估的基准。
【第2.5代·大模型属性清单式裁决（UltraVideo 2025.06，规模最大）】
**Qwen2.5-VL-72B 对16种低质属性做二元判断，命中任意一项即删除**（转场特效、水印、分屏、屏幕录制、画中画等）。同时用 **VideoCLIP-XL-v2** 计算视频与摘要caption的相似度，**<0.2 剔除**——这是七者中唯一显式的图文一致性剔除环节（Panda-70M 的 matching_score 只用于选caption不用于剔除）。72B 规模的判官 + 属性清单式的结构化裁决，是2025年的典型形态。
【第3代·把子指标喂进学习式网络输出统一分（Koala-36M，另一条路线）】
Koala-36M 走的不是「更大的VLM」而是「更好的融合」。其 **Training Suitability Assessment (TSA) 网络**有三个分支：动态分支（**3D Swin Transformer**）、静态分支（**ConvNeXt**）、标签分支（把清晰度/美学/运动等**传统子指标作为额外输入**而非阈值），再用 **Weight Cross-Gating Block (WCGB)** 把标签分支特征以可学习的融合权重注入动态与静态分支。**VTSS 是这个网络吃「像素+子指标」后直接输出的标量，不是对子分数做回归或加权求和**——否定加权阈值范式正是该论文的全部论点。训练标签来自 **20万条视频 × 8名专家 × 1–5分**，并做了两重偏差校正：个人偏好偏差用**逐专家 z-score 标准化**后按全局均值方差还原，标注波动偏差取**8人均值**。消融（PLCC/SRCC/KRCC/RMSE）：仅动态分支 0.8684/0.8580/0.7027/0.4644 → +静态 0.8730 → +标签分支 0.8953 → +WCGB **0.8974/0.8868/0.7406/0.4099**；对比 FastVQA 0.8684、DOVER 0.8554。**VTSS 阈值2.5 的选法也很朴素**：全量分布呈双峰（近似两个高斯），直接取两峰之间的谷值2.5（发布CSV中VTSS最小值恰为2.50，独立印证）。
**⚠️ 复现警告**：Koala-36M **发布的 VTSS 检查点并非造数据用的那个**——test.yml 显示其配置为 `DiViDeAddEvaluator + swin_tiny_grpb + fragments-only + divide_head:false`，即 FAST-VQA/DOVER 的纯 fragments 分支，性能恰好等于消融表中「仅动态分支」那一行（PLCC 0.8684），**ConvNeXt静态分支、标签分支与WCGB 在仓库中完全缺失**。任何人按其开源代码复现过滤，得不到论文的 VTSS。
**结论**：2024年上半年的数据集（Panda-70M、InternVid、OpenVid-1M、MiraData）尚停留在「浅层打分器+阈值」，2024年10月的 LVD-2M 与 Koala-36M 分别从「换判官」和「换融合方式」两个方向突破，2025年的 UltraVideo 则把判官规模推到 72B 并结构化为属性清单——**「VLM-as-judge」在这七个数据集上的演进时间线非常清晰**。

### 安全与合规过滤（NSFW、版权、人脸/隐私） ⚠️

**七者的安全合规过滤整体薄弱，仅两家有专门的安全分类器**：
- **InternVid（有专门分类器）**：使用**二分类器识别并排除非伦理/NSFW视频**，同时在爬取源头就把检索词与频道限定在 SFW 范围。这是七者中最早也是最明确的安全环节。
- **MiraData（有专门分类器）**：使用 **Stable Diffusion Safety Checker**，对每条视频**均匀采样8帧**逐帧检查，NSFW片段从**包括788K未过滤池在内的所有版本中移除**——即安全过滤是前置且不可绕过的，这一点做得比其他家规范。
- **Panda-70M（有内容与隐私处理但无NSFW分类器）**：发布前用内部自动化流程过滤含有害/暴力语言以及提及毒品、仇恨言论的样本，并用 **NLTK 把caption中所有人名替换为「person」**。这是七者中唯一的隐私脱敏措施。但**没有视觉NSFW检测**。
- **Koala-36M（仅靠人工标注间接传递，是明确的弱点）**：唯一涉及安全的地方在人工标注准则的「视频自然度」维度——要求标注员剔除「政治、恐怖、暴力、血腥或其他令人不适的内容」。后果是：**安全性被折叠进一个阈值为2.5的标量 VTSS 中，与质量完全纠缠**，一条不安全但美学与动态优秀的视频完全可能通过。**没有独立的NSFW分类器，也没有明确的色情内容类目**，无法审计、无法单独调整。这在七者中是最应被点名的合规缺陷。
- **UltraVideo**：**无 NSFW 过滤、无人脸/隐私过滤**。其16项低质属性判断针对的是画质缺陷而非内容安全。安全性隐含依赖于「YouTube 4K/8K 精品池 + 人工复检」这一较干净的源头。
- **OpenVid-1M**：论文、仓库与HF卡片中**均未提及任何NSFW或安全过滤**。[不确定]
- **LVD-2M**：全文 grep 确认 **「NSFW」「watermark」出现0次**，**无任何安全过滤**，隐含依赖上游四个数据集各自的清洗。
- **版权内容检测**：七者**全部没有**。
- **输出侧**：七者均无生成内容水印或检测工具（它们是数据集，但配套发布的 UltraWan、MVDiT、ViCLIP 等模型也都未附带安全分类器）。
**总体判断**：安全过滤的完备度排序为 MiraData ≈ InternVid > Panda-70M（偏隐私与文本） > UltraVideo（靠源头） > Koala-36M（纠缠在VTSS里） > OpenVid-1M ≈ LVD-2M（无）。这与「质量过滤越做越精细」形成鲜明反差——**七个数据集在两年间把画质过滤从无做到72B VLM，安全过滤却基本原地踏步甚至倒退**。

## 打标方式

### 使用的caption模型（自研VLM/开源模型，模型规模） ⚠️

**打标模型选择直接决定caption长度，从13.2词到824.2词跨越60倍**。
【Panda-70M·多教师+检索式择优，架构最复杂产出却最短】候选池**31个打标器 = 6个基座 × 权重变体 × 输入模态组合**（V=视觉/S=字幕/M=元数据即标题+描述）：Video-LLaMA（pretrain与finetune共8变体，**仅用视觉分支、音频分支明确关闭**，Vicuna-7B）、VideoChat（7B，4个）、VideoChat Text（4个，把视频文本化为标签+密集描述+总体描述再对话，因成本把ChatGPT-4换成LLaMA）、Video-ChatGPT（4个）、BLIP-2（opt2.7b/opt6.7b/flant5xl共3个，图像模型从0.3N–0.7N帧间随机取一帧）、MiniGPT-4（7B/13B共8个）。用1000条clip的用户研究做**贪心集合覆盖**：单个最好模型仅覆盖30.8%，31个全上84.7%，贪心选出的**8个覆盖76.8%**（Video-ChatGPT入池未入选）。视觉-only的prompt为「Please faithfully summarize the video (or image) in one sentence.」。**择优环节**用细粒度视频-文本检索：**UMT large = ViT-L/16 + BERT-large**，VTC+VTM双损失 + **难负例挖掘（7条未选中caption权重1.0，batch内其他负例权重0.01）**，12帧224×224，AdamW lr 2e-5，batch 32，10 epoch，8×A100-80G。效果：微调后UMT **R@1 35.90%** vs 预训练UMT 21.82%，**而人类之间的一致率仅44.9%**（说明「最佳caption」高度主观）。**蒸馏学生模型**：视觉分支沿用Video-LLaMA设计（8帧→冻结ViT-G/14(EVA-CLIP)+Q-Former→时序融合→32×4096），文本分支用**text Q-Former**以视频表示为query产出固定32×4096，拼成64×4096送入**Vicuna-7B-v0**；文本→视觉的梯度被阻断，元数据与字幕各以0.5概率独立丢弃；在全量Panda-70M上训**30万步、48×A100-80G**。动机很实际：完整pipeline每条clip要跑**8+1=9次**模型推理。
【InternVid·多尺度，两尺度用不同模型——最常被误传的一点】**粗尺度用BLIP-2描述中间帧；细尺度用轻量的Tag2Text以低fps逐帧描述，再用一个预训练语言模型合成为整段描述**。**细尺度不是BLIP-2**。该LLM身份论文未明确（引用中同时列T5与Vicuna却未说明实际用哪个）；HF上的`InternVid-18M-aes-vc2vicuna.jsonl`暗示存在VideoChat2+Vicuna的重打标通路，但那是后续重标而非原始摘要器的证据。**摘要prompt原文、Tag2Text的fps、UMT的具体变体均未公开。**[不确定]
【Koala-36M·GPT-4V教师→LLaVA学生蒸馏】先用**GPT-4V**在种子集上按结构化schema产出caption，再微调一个**基于LLaVA**的打标器跑全量（**不是Tarsier**）。微调经验值得复用：(a)训练视觉编码器（不冻结）提升caption准确性；(b)高分辨率视觉编码器有助于捕捉细节；(c)为控制token开销对**视觉token的空间维度做2×2平均池化**；(d)**图像+视频混合训练**，既教会静态/动态理解又缓解视频数据稀缺。
【MiraData·GPT-4V一轮对话产出5字段 + Panda-70M出短caption】6个字段中**5个由GPT-4V在同一轮对话中生成**，1个（short_caption）直接用**Panda-70M的打标模型**。输入形式特别：均匀取**8帧拼成2×4网格的单张图**（按时序从左到右、从上到下，白边分隔），并把Panda-70M短caption作为**hint**注入prompt。仓库里另有`default_prompt_wo_hit`变体完全去掉hint，作者称「在少量样本上测试结果几乎相同」——**这实际上削弱了Panda-70M那一步的必要性**。
【OpenVid-1M】**LLaVA-v1.6-34B**对全部视频重打标，包括为原本无caption的CelebvHQ补标。**prompt原文未公开。**[不确定]
【LVD-2M·分层三段式】(1)超过30秒的视频先切成**30秒段**；(2)每段均匀取**6帧拼成2×3网格单图**（方法源自IG-VLM），送**LLaVA-v1.6-34B**，产出背景、主要人物、主要动作、相机视角，**采样间隔是自适应的（段长/6，≤5秒）**；(3)用**Claude3-Haiku**精修与合并（两条prompt分别负责「精修单条粗caption」与「按时序合成整段描述」），动机是「LLaVA-v1.6-34B倾向于产生额外的推测与假设，导致冗余」；(4)仓库中还有论文未提及的第三条轨道`rewritten_caption`，由**LLaMA-v3.1-70B**改写为「简洁的用户输入风格」。实测100条样本平均词数：raw 215.4 → refined 81.8 → rewritten 41.3，**Claude3-Haiku把LLaVA输出压缩约2.6倍**。
【UltraVideo·全开源模型，规模最大】**Qwen2.5-VL-72B**产出9个维度，再用**Qwen3-4B**整合成第10个Summarized Description。作者明确与MiraData（GPT-4V+2×4网格）、Koala-36M（GPT-4V蒸馏进LLaVA）对比，强调自己用开源模型。**prompt与帧组装方式均未公开**——GitHub issue #3专门追问，被关闭且无维护者回复。[不确定]
【横向规律】2024上半年（Panda-70M、InternVid）走「多个小模型+融合/择优」的工程化路线；2024下半年起（MiraData、Koala-36M、OpenVid、LVD-2M）统一转向「一个强VLM（GPT-4V/LLaVA-34B）直接打标 + LLM精修」；2025年（UltraVideo）进一步转向「大参数开源VLM（72B）+ 小LLM汇总」。**教师模型的融合复杂度在下降，单模型能力在上升。**

### caption密度与结构化程度（短/长/密集描述、结构化字段如镜头运动、风格标签） ⚠️

**caption长度与结构化程度是七者最直观的差异**（平均词数按UltraVideo Table 1统一口径）：Panda-70M **13.2词/0字段**；InternVid **17.6词/0字段**；LVD-2M **88.8词/0字段（但时序密集）**；OpenVid-1M **126.5词/0字段**（HD子集104.5词）；Koala-36M **202.1词/6字段（融合成一段）**；MiraData **318.0词/6字段（独立可寻址）**；UltraVideo **824.2词/10维度**（long 850.3词）。
【无结构化的四家】Panda-70M：一句话13.2词，caption的教师来源分布见论文图11。InternVid：17.6词，约50%在10–20词、约1/3不足10词、约11%超过20词。OpenVid-1M：LLaVA单条自由文本，论文仅以图示定性说明「显著长于Panda-50M」，**未给出词数**（126.5词来自UltraVideo表转引）。**LVD-2M是独特的中间形态**——其结构化不体现为字段而体现为**时序密集性**：先分30秒段各自描述、再按时序合成，因此caption天然描述「事件如何随时间推进」而非静态画面罗列；词数分布30–50词约2%、50–70词约20%、**70–90词约38%**、90–110词约24.5%、>110词约13.5%。
【六字段结构化的两家，字段设计惊人地一致，可直接复用】
- **Koala-36M六字段**：(1)主体 subject、(2)主体动作 actions、(3)所处环境 environment、(4)**视觉语言 visual language**（风格/构图/光照）、(5)**镜头语言 camera language**（运镜/角度/焦距/景别）、(6)**世界知识 world knowledge**。六字段**分别生成后融合成一段连贯行文**，最终以单个caption字段发布（CSV中caption字符数实测123–3,320）。
- **MiraData六字段（七者中唯一独立可寻址）**：CSV列名即`short_caption`/`dense_caption`/`background_caption`/`main_object_caption`/`style_caption`/`camera_caption`。**正因如此它才能在MiraBench中做「相机对齐/主体对齐/背景对齐/风格对齐」四个分维度的图文一致性评测，别家做不到。** 词数构成：**dense caption约90词 + 四个结构化字段约214词 = 总计318词**（**常被误引为「dense caption 318词」，实为全字段总和**）。实测官方100条样本各字段均值：short 19.0、dense 112.3、main object 84.5、camera 83.8、background 77.2、style 37.3。GPT-4V prompt的五个编号项精确对应五个字段，**few-shot示例直接用了Sora官方演示prompt原文**（东京霓虹街道女子、红色针织摩托头盔太空人），并明示「参照DALL-E 3，让GPT-4V产出有利于学习文生视频模型的描述」，还特别约束「不要逐帧描述、不要出现'first frame'之类的词」。
【十维度的UltraVideo（结构化程度最高）】前9个由Qwen2.5-VL-72B生成、第10个由Qwen3-4B汇总：(1)Brief Description简述 (2)Detailed Description详述 (3)Background背景 (4)Theme Description主题 (5)Style风格 (6)**Shot Type景别** (7)**Camera Movement运镜** (8)**Lighting光照** (9)**Video Atmosphere氛围** (10)Summarized Description汇总。**「景别」「光照」「氛围」三个维度为七者独有**，明显面向影视化生成需求。各维度平均词数**只在图4(d)与图A1中以分布图呈现，从未列表**；HF浏览器给出字符区间可作代理：Brief 34–526、Detailed 479–2,610、Summarized 586–6,060字符。
【长caption带来的两条被实证的教训】
1. **文本编码器容量必须匹配**：MiraData明确因318词caption放不进CLIP的77 token而改用**Flan-T5-XXL（512 token）**；LVD-2M则吃了亏，其88.7词caption被冻结CLIP文本编码器截断，作者把I2V文本匹配提升不明显（16%/70%/14%）**直接归因于77 token上限**。
2. **训练caption与用户prompt的分布错配**：LVD-2M指出**VBench的prompt平均仅7.6词**，远接近WebVid的14.1词而非自家的88.7词，因此评测本身对长caption数据集**不利**。UltraVideo的应对是**随机caption采样策略**——以1/3概率从{Brief, Detailed, Summarized}中选一个，若选中Brief或Detailed则再从剩余7个结构化类目中**随机追加一个**作为最终prompt，让模型同时适应长短prompt。**这是七者中唯一针对分布错配设计训练时数据增强的方案，很值得复用。**

### 音视频联合caption结构（是否同时覆盖视觉+听觉轨道、是否分流为独立字段，如LTX-2全音景描述、Script-a-Video factorized streams、Foley-Omni三字段）

不适用。七个数据集的caption**全部只覆盖视觉轨道**，不含任何听觉轨道描述，也不存在视听分流的字段设计。即便是字段最多的UltraVideo（10维度）与MiraData（6个独立字段），其维度也全部是视觉的（主体/背景/风格/景别/运镜/光照/氛围等），无一涉及音景、音效、语音或音乐。因此不存在LTX-2式全音景描述、Script-a-Video式factorized streams或Foley-Omni式三字段结构的对应物。**这是本次调研七个数据集与音视频联合生成模型数据体系之间最根本的能力断层：现有主流视频预训练数据集无法为AV模型提供任何听觉侧监督信号，AV模型的音视频联合数据必须完全另起炉灶自建。**

### 对白转写与说话人属性标注（ASR转写+说话人身份/语言/口音/情绪）

不适用。七者均无ASR转写发布，无说话人身份/语言/口音/情绪标注。两处相关但不构成对白标注的情况需澄清：(1) **Panda-70M使用了英文字幕（含YouTube自动字幕）但仅作为教师模型的文本侧输入**（video2dataset配置writesubtitles:True, subtitleslangs:['en'], writeautomaticsub:True），字幕落在每clip的JSON元数据中，不作为数据集字段发布，且不含说话人属性；(2) **InternVid内部采集到11种语言的ASR字幕**（论文图16示例英/中/韩/德），但既未用于生成caption也未随数据集发布——这是一处被采集却被丢弃的多语资源。此外Panda-70M出于隐私考虑**用NLTK把caption中所有人名替换为「person」**，等于主动抹除了说话人身份信息。

### 几何与结构化标注（相机参数、深度、3D point tracks、动作标注、显式状态）

**七者的结构化标注都停留在二维语义层，无一提供三维几何信息**：
【有的部分】
- **相机运动**：六家把运镜作为caption中的描述字段（Koala-36M的「镜头语言」含运镜/角度/焦距/景别；MiraData有独立的`camera_caption`列；UltraVideo有独立的Camera Movement与Shot Type两个维度；LVD-2M的caption包含相机视角；OpenVid/InternVid的自由文本中零散涉及）。**但全部是自然语言描述，没有任何一家提供结构化的运镜标签枚举或相机轨迹参数。**
- **标量质量分（唯一真正结构化的部分）**：**Koala-36M最完整**——CSV直接发布三列可用于条件控制的分数：`clarity_score`(0–1)、`aesthetic_score`(2.28–6.56)、`motion_score`(0.01–267)，外加融合分`video_training_suitability_score`(2.50–4.95)。**Panda-70M**发布`matching_score`（UMT图文匹配分）、`desirable_filtering`（6类枚举）、`shot_boundary_detection`（TransNetV2输出的镜头区间列表，长度为1即单镜头）。**LVD-2M**发布`total score`（平均光流分）、`scene_cut`、`span`、`video_time`、`dataset_src`。**InternVid**发布UMT-SIM分与美学分。MiraData/OpenVid/UltraVideo的CSV则主要是caption字段，质量分未随数据发布。
- **视频类目标签**：仅**LVD-2M**用BART分类出8类标签（有精确百分比），**UltraVideo**有7主题108子类的体系（但映射关系未发布，issue #5追问无果）。
【完全没有的部分】相机内外参、深度图、3D point tracks、光流场本身作为条件、显式物体状态/动作标签（action label）、分割掩码、人体姿态/关键点——**七者全部为零**。唯一沾边的是MiraData在评测端（MiraBench）用**GVGC三维重建**算MAE/RMSE作为「3D一致性」指标、用**CoTracker**算Tracking Strength，但那是**评测指标而非数据标注**，不随数据集发布。
**结论**：这七个数据集是纯粹的「视频-文本对」资源，无法直接支撑相机可控生成、深度可控生成或3D一致性训练——下游若需这些能力必须自行补标。

### 合成数据构造（受控扰动/编辑构造训练对，如InstructAV2AV）

**七者全部不使用模型合成的视频数据**，训练视频100%为真实采集的网络视频或素材站素材。存在的「构造式数据」仅限于文本侧与标注侧：
- **Koala-36M（唯一的自监督数据构造，最值得复用）**：为训练Color-Struct SVM转场检测器，**用规则构造正负样本对——同一视频内的帧对为负例（无转场）、不同视频源的帧对为正例（有转场）**。这是把「拼接不同视频」当作转场的合成代理，构造成本极低且完全不需要人工标注。
- **LVD-2M（caption层面的多轨道改写）**：同一视频保留三条caption轨道——LLaVA-v1.6-34B的`raw_caption`(215.4词)、Claude3-Haiku精修的`refined_caption`(81.8词)、LLaMA-v3.1-70B改写的`rewritten_caption`(41.3词)。**这实质上是为同一视频合成了长/中/短三种prompt分布**，可直接用于缓解训练-推理prompt错配，是七者中唯一提供多粒度prompt配对的数据集（且第三条轨道论文未提及，只存在于发布数据中）。
- **MiraData**：GPT-4V打标时把Panda-70M的短caption作为hint注入，属文本侧级联生成而非合成数据。
- **UltraVideo**：Qwen3-4B把9个子caption整合为第10个汇总描述，属文本侧合成；另用LLM从108个主题**生成检索关键词**再人工搜索，属数据召回策略而非数据合成。
- **Panda-70M**：教师caption经UMT择优后用于蒸馏学生模型，属知识蒸馏而非数据合成。
- **InternVid / OpenVid-1M**：无任何数据构造环节。
**共同缺口**：七者**均无受控扰动/编辑构造的训练对**（如InstructAV2AV式的编辑前后配对），也无任何合成引擎（Unity/UE/Blender）渲染数据——MiraData的「3D引擎渲染场景」类目是**从YouTube收集的他人渲染成品**，不是自建渲染管线。

### 人工介入程度（人工标注、人工质检、模型初筛+人工复核） ⚠️

**人工介入从「零人工」到「20万条×8人专家打分」差异极大，用途高度分化为三类**：
【类型A·人工标注作为模型训练数据（两家）】
- **Koala-36M（20万条 × 8名专家，本次调研中人工投入最大）**：随机采样20万条视频，每条由**8名训练过的专家按1–5分**评「作为视频生成训练数据的适宜性」。三个维度：**动态质量**（运动区域需覆盖画面**30%以上**；区分业余抖动与专业运镜）、**静态质量**（主体细节丰富、构图合理、有美感、主体清晰、色彩饱满）、**视频自然度**（偏好自然未加工内容，惩罚特效/转场/字幕/logo；剔除政治、恐怖、暴力、血腥等内容）。两重偏差校正：**个人偏好偏差**用逐专家z-score标准化后按全局均值方差还原；**标注波动偏差**取8人均值。该标注直接训练出TSA网络。
- **Panda-70M（10万条，用于训练caption选择器）**：随机采样**10万条clip**，标注员从8条候选caption中选最好的一条。指令原文很有意思：「Imagine you are talking on the phone with your friend and you need to describe the video to him.」提供「All Bad」选项，**12,064条因全部caption不合格被丢弃**，剩余划分为86,131训练/1,805验证。另有2名标注员重标验证集以测算人类一致率（**仅44.9% R@1**）。
【类型B·人工用于规则/阈值校准与源头把关】Panda-70M：1000条clip×31条caption的教师筛选研究（要求「选出所有好的caption」，标准是「不含任何错误信息，且覆盖主要动作或所有主要物体」），据此做贪心集合覆盖。Koala-36M：**1万条视频的人工转场标注**（约半数含转场）作为转场检测器的评测基准。**UltraVideo（源头人工把关最重）**：(a)从LLM生成的关键词出发**人工搜索**最新4K/8K视频；(b)对5,000条原始视频做**二次人工复检**剔除低质/模糊/水印/抖动；(c)对LLM归纳的7主题108子类做**人工修订与确认**。MiraData：论文局限性中提及「人工核验无法保证完全准确」，暗示存在人工抽检但**未说明规模**。[不确定]
【类型C·人工仅用于最终评测，不回流数据】
- **UltraVideo**：**5名标注员各评1,000条**随机抽样视频（UltraVideo与Koala-36M各一份），「坏视频」定义为命中15种缺陷之一（字幕、异常色块、绿幕、蓝幕、转场特效、水印、贴纸、边框、分屏、屏幕录制、画中画、静止视频、模糊视频、花屏、纯色背景），并明确要求**评判时忽略分辨率**。结果：**UltraVideo坏视频率2.3% vs Koala-36M 41.5%**（未报告标注员间一致性）。另有10人参与UltraWan与官方Wan的偏好对比。
- **LVD-2M**：三组人工评测——长镜头一致性（每数据集40条盲测）、动态程度（40条3档评分）、caption质量（50条A/B）；生成实验的用户研究有**200份有效响应**。
- **Panda-70M**：200条视频、5名参与者的教师vs学生偏好研究。
【类型D·零人工】**InternVid**仅在动作词表构建时做过「LLM抽取5,001个动作后人工校验」，其余环节无人工。**OpenVid-1M**未见任何人工标注、质检或评测的记载。[不确定]
**两条关键结论**：(1) **七者无一对caption内容做人工核验或修正**——即便投入20万条标注的Koala-36M，人工也只用于打质量分，caption全部由VLM生成后直接使用；UltraVideo更是明确「无任何caption内容的人工核验或修正」。(2) 人工投入的主流用法正从「标注数据」转向「校准自动化规则+验证结果」，Koala-36M的20万条×8人是个例外，而其代价换来的却是一个**不可复现的打分器**（发布版并非论文版）。

## 音视频对齐

### 音视频同步检测方法（口型同步、事件对齐）

不适用。七个数据集均无音频模态，不存在音视频同步检测环节，未使用任何口型同步或音画事件对齐方法。

### 同步检测具体指标与阈值（SyncNet/Synchformer/LSE/自研，阈值数值，如MOVA LSE-D≤9.5且LSE-C≥4.5、SkyReels-V4 SyncNet |offset|≤3∧conf>1.5）

不适用。七者均未使用SyncNet、Synchformer、LSE-C/LSE-D、AV-align或任何音视频同步指标与阈值。

### 时序同步vs语义同步的分离处理（时间对齐与内容语义匹配作为两个独立过滤条件）

不适用（无音频模态）。若在视觉侧做类比，可对应的是「时序一致性」与「图文语义匹配」被分开处理：**时序侧**由切镜/转场检测（Panda-70M的ImageBind首尾距离≤1.0、Koala-36M的CSS+3σ、LVD-2M的0.5fps ContentDetector阈值50）与相邻帧一致性（OpenVid-1M的CLIP相邻帧余弦相似度双向剔除）承担；**语义匹配侧**由图文相似度承担（Panda-70M的UMT matching_score>0.43但仅用于选caption不用于剔除；**UltraVideo的VideoCLIP-XL-v2相似度<0.2剔除是七者中唯一把图文语义匹配作为独立剔除条件的**）。但这与音视频同步无关。

### 音频质量过滤（SNR、静音检测与静音占比阈值、无音轨剔除、画外音源剔除、背景音乐分离）

不适用。七者的pipeline均不读取音轨，无SNR计算、无静音检测与静音占比阈值、无无音轨剔除、无画外音源剔除、无背景音乐分离。Koala-36M的重切分代码（cv2.VideoWriter/mp4v）在重新切片时会**直接丢弃音轨**；Panda-70M（download_audio:True）与UltraVideo（声明保留native audio）虽保留原始音轨但不做任何质量检查。

### 语音/音效/音乐的分类与分别处理策略

不适用。七者均无语音/音效/音乐/环境音的分类与分别处理策略。

## 训练配合

### 多阶段训练课程与数据课程调度（阶段划分依据：分辨率/时长/质量分/模态；低清→高清、图像→视频、短→长） ⚠️

**七者是数据集而非训练框架，因此本身不定义训练课程**，多阶段调度由下游使用者实现。但有三处与课程相关的数据侧设计值得记录：
1. **MiraData的四档嵌套子集（330K/93K/42K/9K）本质上就是一条现成的数据课程**——按色彩/美学/运动阈值递增严格度分层，使用者可直接实现「大而松的底层打底 → 小而严的顶层收尾」的渐进课程，无需自建分层。这是七者中对课程调度最友好的设计。
2. **UltraVideo的short/long双分割**（3–10秒 42K条 / >10秒 17K条）天然支持「短→长」的时长课程。
3. **Panda-70M的10M/2M子集与InternVid的FLT/DIV/AES子集**同样提供了质量分层，但档位较粗（各2–3档）。
【配套模型的训练课程（作为旁证）】UltraWan是七者配套模型中课程信息最完整的：LoRA微调Wan-T2V-1.3B，UltraWan-1K为1088×1920×81帧（rank 64）、UltraWan-4K为2160×3840×29帧（rank 16），LoRA模块=自注意力QKV+输出线性层+FFN第1与第3个线性层，AdamW betas(0.9,0.999)、weight_decay 1e-2、**lr 1e-4、仅1个epoch**、每卡batch 1、无张量并行，在**H20**上训练，1K耗**3.4K GPU小时**、4K耗**7.6K GPU小时**。作者坦承UltraWan-4K因**只训1个epoch且受显存限制只能29帧**而明显欠训、主体伪影多于1K版。LVD-2M的实验则是「先在短片段上预训练、再在LVD-2M上做帧数扩展微调」的两段式（32帧→65帧）。
**共同缺口**：七者**均未定义「低清→高清」的分辨率课程**（因各自分辨率单一），也**均未提供按训练阶段划分的官方数据配比方案**。[不确定]

### 数据配比随训练阶段的变化（预训练/退火annealing/SFT高质量子集） ⚠️

**七者均未给出「随训练阶段变化的数据配比」方案**——这是数据集与模型技术报告的本质区别。可用的替代信息有三类：
1. **数据集内部的天然分层比例**：MiraData 788K未过滤→330K(41.9%)→93K(11.8%)→42K(5.3%)→9K(1.1%)；Koala-36M 48M未过滤→36M(75%)；UltraVideo 62K→46K→42K；Panda-70M 70.7M→10M(高质量子集)→2M；InternVid 234M→18M(AES)→10M(FLT/DIV)。使用者可直接把这些档位当作「预训练/退火/SFT」三段的数据源。
2. **源数据集之间的混合比例（LVD-2M最明确）**：其2M输出中**WebVid系约120万（近六成）、YouTube系（Panda-70M+InternVid）约60万、HD-VG约30万**——这是七者中唯一可从发布文件推算出的跨源配比。注意论文正文并未给出该拆分。
3. **OpenVid-1M的分源过滤强度差异**：对**Panda-50M源取美学分前20%、对其余三源取前90%**——这是一种隐式的配比控制，即对噪声最大的源施加最严过滤，从而压低其在最终集中的占比。这一「按源设定不同过滤强度」的做法很实用，值得复用。
**退火（annealing）与SFT高质量子集**：七者均无显式定义。若必须选一个作为「SFT精选集」，最合理的是 **MiraData-9K**（1.1%保留率）、**UltraVideo-42K**（人工验证坏视频率仅2.3%）或 **InternVid-18M-AES**（美学≥4，被开源社区广泛用作T2V精调集）。[不确定]

### 后训练数据（SFT精选集规模与筛选标准、偏好对数量与标注方式、reward model训练数据） ⚠️

**七者全部没有现代意义上的后训练数据**——无偏好对（preference pairs）、无reward model训练集、无RLHF/DPO数据。这是它们作为「预训练数据集」的定位使然，但也是相对工业界数据体系的显著缺口。具体情况：
- **偏好数据仅用于评测，不回流训练**：Panda-70M的200条视频×5人教师vs学生偏好研究；UltraVideo的10人UltraWan vs 官方Wan偏好对比（视频质量美感 **81.10% vs 18.90%**、时序稳定性55.70% vs 44.30%、文本一致性54.50% vs 45.50%）；LVD-2M的200份有效响应用户研究。这些人类偏好数据**均未被构造成训练用的偏好对**。
- **唯一被用于训练的人工标注是「质量分」而非「偏好对」**：Koala-36M的20万条×8专家×1–5分，训练的是数据过滤器（TSA网络）而非reward model；Panda-70M的10万条caption选择标注，训练的是caption选择器（UMT）而非奖励模型。**两者都是「用人工标注训练数据管线组件」，而非「用人工标注对齐生成模型」**——这个区分对理解2024–2025年开源数据集的定位很关键。
- **可作为SFT精选集使用的子集**：见stage_data_mixture字段（MiraData-9K、UltraVideo、InternVid-18M-AES）。其筛选标准即各自漏斗最严档的阈值组合。
- 配套生成模型（ViCLIP、MVDiT、UltraWan、MiraDiT）**全部只做预训练或LoRA微调，无一做过偏好对齐**。

### 数据处理基础设施与吞吐（NeMo Curator/Data-Juicer/自研，GPU加速比，处理规模，成本） ⚠️

**七者对数据处理基础设施与吞吐的披露极少，这是本次调研信息最匮乏的字段**：
- **框架**：七者**均未使用NeMo Curator、Data-Juicer等成熟数据处理框架**，全部为自研脚本。唯一有工程化封装的是**Panda-70M——它fork并修改了`video2dataset`**（上游iejMac版本无法解析其CSV格式），提供`panda70m.yaml`配置，是七者中下载与切分流程最工程化的。其配置项中两个容易踩坑的默认值：`download_size: 360`（**默认只下360p，不改配置拿不到720p**）、`download_audio: True`（默认带音轨，关掉可提速）。
- **明确给出的算力数字（仅三处，且全是模型训练而非数据处理）**：Panda-70M的UMT选择器微调 **8×A100-80G**、学生caption模型 **48×A100-80G训30万步**；InternVid的ViCLIP训练 **64×A100约3天**（5000万视频-文本对，用DeepSpeed + FlashAttention，含MAE式随机patch掩码加速，最后再以lr 4e-6做0.5个epoch的无掩码训练以弥合预训练/部署差异）；UltraVideo的UltraWan **1K版3.4K GPU小时 / 4K版7.6K GPU小时（H20）**。
- **唯一的数据处理侧性能数字（Koala-36M，很有参考价值）**：转场检测在CPU上每帧对的耗时（ms）——256分辨率 1.42、512 2.45、720p 6.15、1080p 12.26、4K 41.98；对比PySceneDetect(HSL) 0.68/2.63/10.73/26.16/102.55 与 PySceneDetect(HSL+edge) 2.50/8.82/30.57/70.11/267.18。**结论是其方法在256上反而更慢，但在1080p快2.4倍、4K快2.4倍（对HSL+edge在4K快6.4倍）**，原因是特征提取固定降采样到256/128，与源分辨率解耦。这是七者中**唯一给出具体数据处理吞吐的公开材料**。
- **成本**：七者**全部未披露数据处理环节的算力成本**。考虑到需对数千万条视频跑光流、美学、OCR以及VLM打标（Panda-70M每条clip要跑9次模型推理、UltraVideo用72B VLM逐条判断16个属性），这部分成本很可能是被隐藏的大头。
- **存储规模（可作为容量规划参考）**：Panda-70M全量约**36TB**（10M子集约8.0TB、2M子集约1.6TB）；OpenVid-1M约**12.4TB**（HD子集约4.5TB）；InternVid元数据40.9GB；Koala-36M元数据48.9GB；LVD-2M元数据约5.08GB。[不确定]

## 效果对比

### 数据策略消融的量化影响（区分：过滤严格度ablation / caption密度风格ablation / 数据配比ablation，及对应评测指标）

**七者中有四家做了同架构同算力下的数据对照实验，这是本次调研最有价值的量化证据**：
【Koala-36M——消融设计最完整】同一Sora式3D全注意力DiT（T5文本编码器+3D因果VAE）从零训练，2秒256×256，A100-80G，batch 32，lr 1e-4，全部训过140M样本，VBench评测（对VBench caption做prompt扩写以弥合域差）。六组对照的VBench总分：Panda-70M **0.6493** → Koala-w/o TSA(48M未过滤) **0.7140** → Koala-37M-manual(人工多阈值) **0.7073** → Koala-36M(VTSS) **0.7156** → Koala-w/o TSA+条件注入 **0.7433** → **Koala-36M+条件注入 0.7460**。三条可分离的结论：(a) **切分+打标改进独占大头**（Panda→Koala-w/o TSA 总分+6.5点，且主要体现在时序：主体一致性0.8584→0.9335、背景一致性0.9435→0.9668、时序闪烁0.9576→0.9857）；(b) **VTSS优于人工多阈值**（0.7156 > 0.7073，且VTSS集更小36M<37M）；(c) **把质量分作为条件注入是单项最大杠杆**（+0.0304），尤其挽救了动态度（0.5750→0.9194）。最终Koala-36M+条件 vs Panda-70M：**总分+9.67点、语义分+28.2点（0.5915 vs 0.3093）**，而**数据量只有Panda的一半**。附加分辨率/时长实验的FVD：256-2s **549.79 vs 570.87**、256-4s **354.79 vs 451.09**、512-2s **392.26 vs 579.57**。
【MiraData——caption密度/风格消融，七者中最干净】同模型下仅改prompt粒度：短caption→dense→结构化，动态度9.45→17.39→**19.53**、Tracking Strength 27.03→52.53→**68.85**、时序一致性DTC 24.39→46.13→**60.83**、总体对齐7.73→14.88→**15.36**。**结论极其明确且反直觉：更长更结构化的caption不提升画质**（美学分4.84→5.14→4.99，成像质量63.64→63.43→64.07基本持平），**但大幅提升动态性、时序一致性与文本对齐（总体对齐从短到结构化近乎翻倍）**。这是全行业最干净的「caption结构化收益」证据。另有数据源消融：MiraDiT在MiraData vs WebVid-10M上训练，动态度7.12→**15.46**、Tracking 22.36→**49.47**（均约翻倍），但作者诚实标注弱项——OpenSora在3D一致性（MAE 75.45 vs 85.27）与主体对齐（18.12 vs 14.67）上仍更优。
【LVD-2M——过滤严格度消融】同架构1.75B MagicVideo式T2V，32帧预训练后扩到65帧，两组数据完全同配置（batch 64、4步梯度累积、30k iter，约过一遍2M）。VBench **16项中LVD-2M赢10项**，关键项：动态度20.00%→**28.06%**、物体类别76.95%→**86.93%**、人物动作61.40%→**76.20%**；输的是背景一致性、时序闪烁、美学、成像质量、多物体——即**「更动态」与「更稳更美」的经典权衡**。作者主动指出该对比对自己不利：**VBench prompt平均仅7.6词，接近WebVid的14.1词而远离自家88.7词**。生成侧用户研究（200份）：扩散I2V动态度 **60%/34%/6%**（微调后/无偏好/微调前），LM式T2V动态度52%/24%/24%、文本匹配58%/10%/32%。
【OpenVid-1M——数据+架构双消融】STDiT 256×256：WebVid-10M(VQAA 13.40/VQAT 13.34/Blip_bleu 23.45/Clip_temp 99.62)、Panda-50M(17.08/9.60/24.06/99.60)、**OpenVid-1M(17.78/12.98/24.93/99.75)**；1024×1024下 WebVid+4×超分 69.26/65.74、Panda+4×超分 63.25/53.21、**OpenVidHD-0.4M 73.46/68.58**——**说明原生高清数据显著优于低清+超分**。架构+数据消融：STDiT+Ours-0.4M(11.11/12.46) → MVDiT+Ours-0.4M(22.39/14.15) → **MVDiT+OpenVid-1M全量(24.87/14.57)**。
【UltraVideo——分辨率消融】UltraWan-1K(LoRA)在VBench多项超过官方Wan-T2V-1.3B-480p（主体一致性97.27 vs 96.11、物体类别82.29 vs 66.66）；关键对照是**Wan-T2V-1.3B直接推1K会严重崩坏**（美学54.82、物体类别33.33、场景0.00），而用UltraVideo微调后恢复正常——**证明高分辨率能力必须由原生高分辨率数据提供，不能靠推理时放大**。注意其VBench只用了**约96条prompt（VBench的十分之一）**，样本量偏小；作者亦指出VBench不适合评高分辨率（动态度与背景一致性会与人类感知矛盾，真4K下运动平滑度与动态度直接OOM）。
【无数据消融的两家】**Panda-70M**只有「用它训 vs 用别的训」的端到端对比（AnimateDiff在Panda-2M上UCF101 FVD **421.9 vs 官方2.5M的499.3**，且**生成结果无水印**，对比WebVid派生模型；UMT在Panda-5M上零样本检索MSR-VTT R@1 **37.2 vs 30.2**），无内部策略消融。**InternVid**同样只有端到端：WebVid-10M单独 vs +InternVid-18M-AES，UCF101 FVD **705.25→616.51**、IS 13.97→**21.04**、FID 98.25→**60.25**、MSR-VTT CLIPSIM 0.2657→**0.2951**；且**无水印**是其被开源社区广泛采用的首要原因。但它**没有做粗尺度vs细尺度的caption消融**——这对一篇以「多尺度」为核心卖点的论文是明显缺失。

### 质量vs数量的证据（小而精数据超越大而杂的案例）

**本次调研在「质优于量」上积累了七个数据集的横向证据链，且强度远超单个模型报告**：
【最强的单点证据·Koala-36M】**与Panda-70M同源同架构同算力，数据量只有一半（36M vs 70M），VBench总分高9.67点、语义分高28.2点**。这是控制变量最严格的一组——同一批HD-VILA-100M原始素材，差别**只在切分方法、caption质量与过滤策略**，排除了数据来源差异的干扰。**这可能是全行业最干净的「同源数据、不同处理」对照实验。**
【最极端的比例证据·UltraVideo】**42K clips / 62小时**，比Koala-36M小约2,700倍（按小时计），却在人工盲评中坏视频率**2.3% vs Koala-36M 41.5%**，且其UltraWan-1K在视频美感上以**81.10% vs 18.90%**压倒官方Wan。说明在特定能力维度（高分辨率、高画质）上，**数据量可以让位于数据纯度三个数量级**。
【最极端的漏斗证据·LVD-2M与MiraData】LVD-2M **220M → 2M（保留率0.91%）**，MiraData 对HD-VILA-100M **1亿clips → 19.5万条（约0.2%）**。两者独立地表明：**「长镜头+高动态+无切换」这组约束的代价是2–3个数量级的数据损耗**，而换来的是人工评测中长镜头纯净率77.5%（vs Panda-70M 50.0%、InternVid 47.5%）与「非常动态」占比30.0%（vs Panda-70M 7.5%）。
【最直接的反面证据·Panda-70M自评】其2024年10月追加的desirability标注显示，即便按自家标准，**全量中仍有约19.5%的样本是不理想的**（低期望分5.28%、静止前景6.82%、微小相机运动1.20%、画中画5.03%、屏幕录制1.13%）。加上LVD-2M人工评测显示其**50%的片段含镜头切换、25%被判为「不动态」**，Panda-70M实际可用比例可能不足一半——**这解释了为何VidGen-1M、OpenVid-1M、Koala-36M三个后续工作都以「精修/重做Panda-70M」为立项动机**。
【caption维度的质优于量·MiraData】同样数据量下，仅把caption从短换成结构化，**文本对齐近乎翻倍（7.73→15.36）、动态度+107%、时序一致性+149%，而画质持平**——说明「caption质量」是与「视频质量」正交的另一条提质路径，且成本远低于重新筛选视频。
【一条重要的反面注解】**分辨率是唯一不能靠筛选获得的维度**。UltraVideo的对照显示，Wan-T2V-1.3B直接推1K会严重崩坏（场景分0.00），OpenVid的对照显示原生1080p（VQAA 73.46）显著优于低清+4×超分（69.26/63.25）。**即「质」中的画质、动态、语义可以靠筛选与重标获得，但像素级分辨率必须靠源头采集**——这是七者中唯一无法通过清洗弥补的短板，也是UltraVideo存在的全部理由。
【方法论提醒】上述除Koala-36M外，多数属于**跨数据集对比**而非严格的「同数据不同处理」对照，存在数据来源差异的混淆因素；且各家评测口径不一（VBench子集、prompt长度、模型规模均不同），**跨论文数字不可直接横向比较**。

### 训练数据domain分布与评测基准类目体系的对齐关系（如VABench七大类） ⚠️

**七者的训练数据类目体系与评测基准类目的对齐程度普遍很差，只有两家有实质对应**：
【评测基准使用情况】七者压倒性地依赖**VBench**（Koala-36M、LVD-2M、UltraVideo均以VBench为主），其次是FVD/FID/IS/CLIPSIM（Panda-70M、InternVid、LVD-2M），OpenVid-1M用DOVER派生的VQAA/VQAT加Blip_bleu与Clip_temp_score，**MiraData则自建了MiraBench**。
【MiraBench——唯一为自家caption结构定制的基准，也是唯一真正的类目对齐案例】17个指标 / 6大方面：**时序运动强度**（Dynamic Degree光流、**Tracking Strength（CoTracker，新提出）**）、**时序一致性**（DINOv2结构一致性、CLIP语义一致性、AMT运动平滑度）、**3D一致性（新提出）**（GVGC重建的MAE与RMSE）、**画质**（LAION美学、MUSIQ成像质量）、**文本-视频对齐**（ViCLIP算的**相机对齐/主体对齐/背景对齐/风格对齐/总体对齐**五项）、**分布相似度**（FVD/FID/KID）。**关键在于：文本对齐的五个子项与MiraData六字段caption逐一对应（camera_caption→相机对齐、main_object_caption→主体对齐、background_caption→背景对齐、style_caption→风格对齐）——这是七者中唯一做到「训练数据字段体系 ⇄ 评测指标体系」一一映射的设计**，其他六家因caption不可寻址而根本无法做分维度对齐评测。两处设计细节：Tracking Strength是为纠正光流动态度的误判而提出（论文图5给出反例：光流1.2 vs 0.7排序错误，Tracking 4.1 vs 11.8排序正确）；时序一致性三项在报告前会**乘以Tracking Strength**（因为「运动越大一致性天然越低」），**因此MiraBench的一致性数字与VBench不可比**。另注：实际代码输出19+个数值（3D一致性输出5个子指标、还算了论文未报告的KVD），与论文的17个不完全一致。
【UltraVideo——有最完整的数据taxonomy，却未与评测对齐】其7大主题（视频场景/主体/动作/时间事件/镜头运动/视频类型/情绪）×108子类是七者中最完整的数据类目体系，且据此做了各类目均匀采样，**但评测仍用通用VBench，且只用了约96条prompt（VBench的1/10）**，两个体系完全脱节。作者反而对VBench提出了尖锐批评：背景一致性与动态度会**与人类感知相矛盾**、人物动作与色彩两项**没有区分度**（多个模型同为66.66与100.0）、运动平滑度与动态度在真4K下**直接OOM**——即**现有基准无法评估高分辨率生成**，这是一个真实的基准缺口。
【其余五家均无对齐】Panda-70M、InternVid、Koala-36M、OpenVid-1M、LVD-2M**都没有建立数据侧的domain类目体系**（LVD-2M有BART分出的8类但仅作事后统计，未用于配比也未与评测对应），因此谈不上对齐。可观察到的只是**间接的、低层维度的对应**：VBench的美学质量↔LAION美学过滤、成像质量↔DOVER/清晰度过滤、动态程度与运动平滑度↔光流/LPIPS双向运动过滤、时序闪烁↔转场与一致性过滤。**即筛选器的设计事实上是围绕VBench的低层画质与运动维度展开的，而对VBench的语义类目维度（物体类别、多物体、人物动作、空间关系、场景）没有任何数据侧的配比或覆盖度保障**——这与domain_distribution字段中「类目治理缺位」是同一问题的两面。
【与VABench类音视频基准的关系】**七者全部无关**（均无音频能力），无法用于音视频联合生成基准的训练数据侧对齐。

## 其他信息

### summary_note

**核心定位**：这七个数据集构成了2023–2025年开源视频生成数据的完整谱系，且恰好可按「规模 ↔ 质量」排成一条清晰的演进轴：InternVid(234M) → Panda-70M(70.8M) → Koala-36M(36M) → LVD-2M(2M) → OpenVid-1M(1M) → MiraData(330K) → UltraVideo(42K)，**数据量每降一个量级，caption长度与过滤严格度就上一个台阶**（13.2词 → 202词 → 318词 → 824词）。
**最值得复用的六项具体技术**：(1) **Koala-36M的Color-Struct SVM转场检测**——代码开源且给出了拟合好的精确系数（`4.61480465×bgr_sim + 3.75211168×canny_sim − 5.485968377`）与3σ时序门限，准确率0.7741/召回0.9395，远超PySceneDetect的0.4574/0.4146，且在1080p以上比PySceneDetect更快；(2) **LVD-2M的低采样率切镜技巧**——ContentDetector阈值50 + **0.5fps采样**，用「2秒内的任何显著变化都判为切换」这一巧思，以零额外成本解决了淡入淡出漏检；(3) **MiraData的过切-缝合范式**——阈值26故意过度切分，再用Qwen-VL-Chat+LLaVA与ImageBind+DINOv2的**两两成对组内一致投票**缝回去，这是拿到72秒长镜头的唯一可行路径；(4) **UltraVideo的统计过滤四件套**——PaddleOCR文字面积>2%、黑边区域均值<3、过曝像素>12%、RGB方差<1.2，统一采用「坏帧率>5%丢弃整条」，**是七者中唯一给全绝对阈值的方案，可直接照搬**；(5) **MiraData与Koala-36M高度一致的六字段caption schema**（主体/动作/环境/风格构图光照/镜头语言/世界知识）；(6) **UltraVideo的随机caption采样**——1/3概率选长短caption再随机追加一个结构化类目，用以对抗训练-推理prompt分布错配。
**三条最强的量化结论**：(a) **Koala-36M vs Panda-70M 是全行业最干净的「同源数据不同处理」对照——同样的HD-VILA-100M素材，数据量减半而VBench总分+9.67点、语义分+28.2点**；(b) **MiraData证明caption结构化只提升动态性/时序一致性/文本对齐（对齐分近乎翻倍）而完全不提升画质**，这两条提质路径正交；(c) **「长镜头+高动态+无切换」的代价是2–3个数量级的数据损耗**（LVD-2M 220M→2M即0.91%，MiraData对HD-VILA仅0.2%）。
**七者共同的、最应警惕的缺口**：① **全部不支持音频**，8个音视频对齐字段完全不适用，无法为AV模型提供任何听觉侧监督；② **全部无实质去重**（精确哈希与embedding语义去重均为零），而LVD-2M同时用了三个YouTube同源语料却未做跨源去重；③ **全部无三维几何标注**（相机参数、深度、point tracks均无）；④ **安全过滤两年间几乎零进步**——画质过滤从无做到72B VLM，而NSFW检测只有InternVid与MiraData两家有，Koala-36M更是把安全性纠缠进一个阈值2.5的标量中无法审计；⑤ **无一对caption内容做人工核验**；⑥ **五家不托管视频本体**，链接腐烂不可逆且无一家发布过ID存活率实测。
**引用时必须注意的六处数据陷阱**：① Panda-70M论文中的「Panda-2M/Panda-5M」是**随机子集**，与仓库发布的质量过滤版2M/10M不是一回事；② Koala-36M的「36M×13.75s=137.5K小时」与其Table 1声称的「172K小时」**自相矛盾约25%**；③ MiraData Table 1的「16K小时」对应的是**788K未过滤池**，330K发布版实为约6,600小时；外部综述所称「77k long videos」系误引v0 beta，勿用；④ UltraVideo的split是**short/long**，不存在「UltraVideo-1K/42K」，1K/4K指UltraWan的输出分辨率；⑤ **Koala-36M发布的VTSS检查点不是造数据用的那个**（发布版为fragments-only的FAST-VQA基线PLCC 0.8684，论文版含ConvNeXt静态分支+WCGB为0.8974），按其开源代码无法复现论文过滤；⑥ **MiraData的过滤阈值论文称在补充材料而补充材料实际不存在**（已核查arXiv附录、NeurIPS camera-ready与补充ZIP链接），其四档子集的分档标准不可复现。

## 不确定字段

以下字段的调研信息部分不确定（⚠️ 标注来源）：

- openness
- data_scale
- data_sources
- provenance_licensing
- duration_distribution
- resolution_aspect_distribution
- domain_distribution
- funnel_retention_rate
- shot_segmentation
- quality_filtering
- motion_filtering
- deduplication
- safety_filtering
- caption_model
- caption_structure
- human_in_loop
- multi_stage_curriculum
- stage_data_mixture
- post_training_data
- data_infra_throughput
- benchmark_taxonomy_alignment
