# 横向对比：数据规模与分布

> 主题: 视频生成模型（含音视频同时生成）的数据处理：数据清洗流程、数据分布、打标方式

[← 返回首页](../index.md)

本页按字段横向对比所有条目。⚠️ 表示该条目此字段部分信息不确定。

**字段**: [训练数据量级（视频条数/小时数/token数，预训练与SFT分开）](#训练数据量级视频条数小时数token数预训练与sft分开) · [数据来源构成（自有/公开数据集/网络爬取/授权采购/合成数据）](#数据来源构成自有公开数据集网络爬取授权采购合成数据) · [数据合规与溯源（授权数据占比、rights-cleared数据集、C2PA等）](#数据合规与溯源授权数据占比rights-cleared数据集c2pa等) · [片段时长分布与切分策略](#片段时长分布与切分策略) · [分辨率/宽高比分布与分桶策略](#分辨率宽高比分布与分桶策略) · [类别/domain分布与配比策略（人物、动作、场景、风格等比例控制与概念均衡）](#类别domain分布与配比策略人物动作场景风格等比例控制与概念均衡) · [音频类别分布与配比（语音/音效foley/音乐/环境音/静音的比例及控制策略）——AV模型独有维度](#音频类别分布与配比语音音效foley音乐环境音静音的比例及控制策略av模型独有维度) · [叙事结构分布（单镜头vs多镜头、平均clip时长、镜头数分布、是否含原生音轨）](#叙事结构分布单镜头vs多镜头平均clip时长镜头数分布是否含原生音轨) · [语言/口音分布（多语种唇同步能力的数据基础）](#语言口音分布多语种唇同步能力的数据基础)

## 训练数据量级（视频条数/小时数/token数，预训练与SFT分开）

`data_scale` · 详细程度: brief

### [Allegro](../models/Allegro.md)

最终精选训练集：1.06 亿（106M）图像 + 4800 万（48M）视频片段，均带有强关联文本 caption。
原始输入规模：4.12 亿（412M）原始图像 + 5 亿（500M）原始视频（此处 500M 指切分并初筛后的视频片段基数）。
分阶段可用数据量（同一份数据按阈值严格度分层，逐层收窄，非独立采集）：
· T2I 预训练：107M 图像（自 412M 原始图像过滤）
· T2V 预训练 360p：48M 视频片段（自 500M）
· T2V 预训练 720p：18M 视频片段（自 500M）
· T2V 微调（SFT）：约 2M 高质量视频片段（自 500M）
VideoVAE 单独训练集：54.7K 视频 + 3.73M 图像（要求短边 ≥720 像素）。
实际训练消耗样本数（含重复采样，与数据集条数不同）：T2I 预训练 700M samples；T2V Pre-train-1（368×640，40 帧）87M；Pre-train-2（720×1280，40 帧）21M；Pre-train-3（720×1280，88 帧）8M；Fine-tune 2.6M。

### [Apollo](../models/Apollo.md) ⚠️

【总量】8100 万（81 million）条带精确密集 caption 的音视频样本，这是论文给出的唯一规模数字，对应过滤后的最终训练集（论文原文：「81 million samples with accurate dense captions」）。
【口径说明】论文以「样本条数」为唯一口径，未给出总小时数、未给出 token 数、未给出单条样本的时长，因此无法换算为小时量级。若按同类工作常见的 5–10 秒 clip 粗估，81M 条约对应 11 万–22 万小时，但此为外推而非论文数据。
【阶段拆分】论文的三阶段训练（预训练 / 专项后训练 / 高质量后训练）均未给出各阶段的数据规模，也未给出预训练与 SFT 的分开量级。Stage III 使用的「manually-curated, high-quality dataset」规模完全未披露。
【类别拆分】81M 中单说话人语音、多说话人语音、歌唱、自然声四类各自的条数或占比未给出。
【结论】Apollo 在数据规模维度只公开了一个总数，粒度显著低于同期工作。[不确定]

### [CineDance / CineDance-1M](../models/CineDance.md) ⚠️

作为数据集发布，无预训练/SFT 的传统划分，规模数据以策展漏斗终点计：
【最终规模】1,021,657 条叙事序列（narrative sequences），总时长约 26.3K 小时（2.63 万小时）。
【原始采集】45,181 条长视频，总时长 32.8K 小时。
【中间产物】TransNetV2 切出 25,899,474 个原子镜头（atomic shots）；经叙事分组得到 1,201,912 条叙事序列。
【单序列特征】平均时长 92.8 秒，平均含 24.2 个连续镜头，最低空间分辨率 1080p。
【标注体量】平均每条视频 6,496.3 个词的结构化双模态标注（Tab.6），标注密度在同类数据集中处于数量级领先。
【首批开放量】HuggingFace CineDance_01 约 240,488 条片段 / 5.83 TB，为四分之一分片。
【模型训练用量】CineDance 模型两阶段训练各自使用的样本条数、batch size、epoch、学习率、总 token 数均未披露。[不确定]

### [CogVideoX](../models/CogVideoX.md) ⚠️

· 视频预训练：经过滤后剩余约 3500 万（35M）条单镜头 clip，平均每条约 6 秒（论文原文「approximately 35M single-shot clips remain, with each clip averaging about 6 seconds」），据此估算总时长约 5.8 万小时量级。
· 图像辅助数据：额外使用 20 亿（2B）张图像，取自 LAION-5B 与 COYO-700M 并以美学分过滤（超参表给出 Lowest aesthetic-value = 4.5）。训练时图像被当作单帧视频与视频混合训练。
· 高质量微调（第4阶段 FT）：从上述数据中选出质量更高的子集，占总数据量的 20%，训练 10k 步。
· 打标模型训练数据：为微调 GPT-4 摘要的替代模型（LLaMA2），收集了 5 万（50,000）条摘要数据点。
· 过滤器训练数据：人工标注 20,000 条视频的正/负样本标签，其中 10% 随机划为测试集。
· CogSound 的数据规模未披露 [不确定]。CogVideoX1.5 相对 1.0 的数据增量亦未披露 [不确定]。

### [Cosmos-Predict2.5](../models/Cosmos-Predict25.md) ⚠️

预训练与后训练的规模披露相当完整，是该报告最有价值的部分之一：
【原始输入】处理超过 2 亿（over 200 million）条原始视频，对应 3500 万小时（35 million hours）原始视频素材（作为对比，Cosmos-Predict1 为 2000 万小时）。
【切分产出】剔除时长不足 5 秒的片段后，得到超过 60 亿（over 6 billion）条候选 clip，时长范围 5–60 秒。
【最终预训练集】约 4% 通过全部过滤，得到约 2 亿（approximately 200 million）条可训练 clip，这 2 亿条即预训练数据集。
【后训练 SFT 各域规模】用 InternVideo2 embedding 上训练的多头分类器划分为五域并统计：object permanence（物体恒存）10.4M 条、high motion（高运动）1.0M 条、complex scenes（复杂场景）1.6M 条、driving（驾驶）3.1M 条、robotic manipulation（机器人操作）730K 条；另有 4K 高清冷却（cooldown）数据 388K 条。
【领域专项】自动驾驶专有数据 约 310 万（3.1M）段 20 秒 7 相机环视 clip；多视图模型 Cosmos-Predict2.5-2B/auto/multiview 训练用 150 万（1.5M）段带 caption 的多视图 clip；Smart Spaces 约 40K clip；机器人各数据集见 data_sources。
【训练迭代】每个领域 SFT 模型训练 30k iterations、batch size 256；RL 阶段 256 steps、batch size 32。预训练总 token 数与总迭代数未披露。[不确定：预训练总迭代数/token 数、35M 小时中各来源占比]

### [Data-Juicer 2.0](../models/Data-Juicer.md)

Data-Juicer 本身不训练模型，因此无「训练数据量级」；相关量级分两个口径。
【系统处理吞吐口径】Data-Juicer 2.0 论文报告可处理 TB 级数据、调度 10k+ CPU 核心；实验覆盖从 56万（560K）样本到 700亿（70B）样本的数据集规模区间；一个生产部署已稳定运行5个月以上、累计处理规模超过 TB 级。分布式测评中在 3200 Ray-DLC 核心上处理 500 倍数据集耗时 1780 秒、2500 倍数据集耗时 7083 秒。
【文生视频案例口径】Sandbox 的 T2V 案例是本调研最直接相关的数据。原始候选池由三个公开数据集构成，合计约 121.7 万条视频-文本对：InternVid 606k + Panda-70M 605k + MSR-VTT 6k。经筛选后开源的最优数据池为 147,176 条（约 227.5GB），另有一个 228k 条的更大数据池用于登顶 VBench 的最终模型（论文记该配置对应 640k 训练样本量，即含数据重复）。小规模探测实验统一使用约 40k 样本的数据池以控制变量。
【无预训练/SFT之分】作为工具链，不存在预训练与 SFT 数据量的划分。

### [Foley-Omni](../models/Foley-Omni.md) ⚠️

总训练规模约 2.7M 音频/视频-文本配对样本，累计约 4.9k 小时音频经过清洗pipeline处理。其中经数据清洗pipeline产出的统一视频-音频-文本三元组 (v_i, a_i, Ŝ_i) 约 2.0M 条（这部分是本文pipeline的直接产物，其余为直接取用的公开TTS/TTA/TTM语料）。
按六个任务组拆分（Table 9）：
- TTS（文本转语音）：LJSpeech + LibriTTS + internal，1,253 小时
- TTA（文本转音频/音效）：AudioCaps + Freesound，912 小时
- TTM（文本转音乐）：MusicCaps + MusicBench + AudioSet，139 小时
- VisualTTS（视觉条件语音合成）：Chem + GRID + LRS2 + SpeakerVid + TalkVid，1,980 小时（占比最大）
- V2A（视频转音效）：VGGSound + Kling-Foley + internal，403 小时
- V2ST（视频转完整配乐，本文核心任务）：internal + SpeakerVid，216 小时（规模最小，作为第三阶段微调数据）
阶段视角：Stage 1 使用约 0.7M 文本-音频对（TTA/TTS/TTM）；Stage 2 扩展至视频条件（V2A/VisualTTS）；Stage 3 用 216 小时 V2ST 数据 + 每个前序单任务域各 100 小时回放数据做联合微调。
评测侧 V2ST-Bench 为 300 条 5–10 秒片段。
[不确定] 论文未披露清洗前的原始视频池总量（小时数或条数），因此无法反推整体保留率。

### [Goku](../models/Goku.md)

【总量】最终训练集约 1.6 亿图文对（160M image-text pairs）+ 3600 万视频文本对（36M video-text pairs）。
【文生图 T2I】1 亿公开样本（来自 LAION）+ 6000 万内部高质量样本，合计 160M。明确分工：公开数据用于预训练（pre-training），内部数据用于微调（fine-tuning）。
【文生视频 T2V】1100 万公开片段 + 2500 万自有（in-house）片段，合计 36M clips。
【按分辨率分层】36M 片段可用于 480p 训练；其中 24M 满足 720p 阈值；7M 满足 1080p 阈值。即随分辨率提升数据量逐级收缩（36M → 24M → 7M），构成天然的分辨率课程数据金字塔。
【图生视频 I2V 后训练】约 450 万（4.5M）「文本-图像-视频」三元组。
【未披露】总视频小时数、token 数、原始采集量（因此无法反推整体保留率）。

### [HunyuanVideo-Foley](../models/HunyuanVideo-Foley.md) ⚠️

【最终产出】约 10 万小时（~100k hours）的文本-视频-音频三元组（TV2A）数据，这是论文摘要中作为第一大贡献点提出的核心数字（"a scalable data pipeline curating 100k-hour multimodal datasets through automated annotation"）。
【口径说明】仅给出小时数，未披露 clip 条数。若按 8 秒定长切片粗算，10 万小时约对应 4,500 万个片段——这是一个相当大的量级，也解释了为何论文强调 pipeline 的「可扩展性」（scalable）而非精细度。
【无预训练/SFT 拆分】整个 TV2A 主模型只有单一训练阶段，10 万小时数据一次性全量参与，不存在预训练与 SFT 的规模拆分，也没有高质量子集的退火阶段。
【音频自编码器的独立数据】DAC-VAE 单独用「约 10 万小时音频数据」训练 700k 步。论文未说明这 10 万小时音频与 TV2A 的 10 万小时是否为同一批数据的音轨——从数字巧合与流程顺序推测很可能同源或高度重叠，但无明确表述。[不确定]
【训练算力对应关系】主模型 128 张 H20，有效 batch size 2048，训练 200k 步；VAE 32 张 H20，batch size 256，700k 步。均未披露 GPU-days 或总训练时长。
【规模定位】10 万小时在 V2A 赛道属于头部量级：作为参照，公开数据集 VGGSound 仅约 550 小时（论文自述用量），AudioSet 约 5,800 小时；即本工作的自建数据规模约为 VGGSound 的 180 倍。相比同期的 Kling-Foley、MMAudio，其数据规模优势是论文归因音频质量领先的主要因素之一。

### [HunyuanVideo](../models/HunyuanVideo.md)

【HunyuanVideo 原版（2024）】总量口径披露不完整：论文明确给出的只有 SFT 阶段约 100 万（~1M）人工精选样本，以及图像侧「数十亿（billions）级」用于第一阶段T2I预训练、「数亿（hundreds of millions）级」用于第二阶段。视频侧的原始池规模与各分辨率档（256p/360p/540p/720p）的绝对条数/小时数未公布，只给出相对保留比例（每级保留上一级的1/2到1/5）。
【HunyuanVideo 1.5（2025）】口径显著更完整，是本条目最有价值的定量披露：
- 原始视频池：超过 1000 万小时（>10M hours）原始视频；
- 经切分与过滤后：约 8 亿（800M）高质量视频片段进入预训练；
- 后续各阶段逐级收缩：480p 阶段 2 亿（200M）、720p/16fps 阶段 1 亿（100M）、720p/24fps 阶段 1 亿（100M）；
- CT（继续训练）阶段：T2V 与 I2V 各 100 万（1M）高质量片段；
- 图像侧：从超过 100 亿（>10B）图像池中筛出 50 亿（5B）用于第一阶段 256p T2I 预训练，10 亿（1B）用于第二阶段 512p；
- 超分模块训练数据：100 万高质量视频片段（1K–4K 分辨率）+ 高分辨率图像。
SFT 阶段与 RLHF 阶段的精确样本数 1.5 报告未给出具体数字（仅描述筛选标准）。

### [InstructAV2AV](../models/InstructAV2AV.md) ⚠️

【最终数据集规模】论文口径：InsAVE-80K 共含 79K 自动验证的训练配对 + 1K 人工精选的评测配对，合计约 80K（数据集名称由此而来）。
【与实际发布的口径差异】HuggingFace 数据集卡显示实际发布 88,074 对（176,148 个文件，因每对含 source 与 target 两份），其中训练 87,074 对、评测 1,000 对，总体积约 139 GB，分 11 个 tar 分片。即实际释出量比论文报告的 79K 多出约 8K 对（约多10%），推测为论文投稿后pipeline继续运行扩充所致，也可能是论文取整表述。使用时应以 HF 实际发布数为准。
【单样本规格】统一为 5 秒片段、720p 分辨率、24 FPS、16 kHz 音频。按 79K×5s 换算，训练集总时长约 110 小时（source 侧）；若计入 target 侧则约 220 小时。这个量级相对文生视频预训练动辄数万至数十万小时而言极小，但对「编辑」这一后训练性质的任务是合理的——模型的生成能力来自 Ovi 预训练先验，InsAVE-80K 只负责教会模型「如何按指令改」。
【预训练与SFT的划分】论文未把 InsAVE-80K 拆为预训练/SFT 两部分。真正的预训练数据是上游 Ovi 与 Wan2.2 的训练语料（本文未涉及也未披露），InsAVE-80K 整体扮演的是编辑任务微调集的角色。
【评测规模】InsAVE-80K 评测集 1,000 对（人工精选）；另在外部 AvED-Bench 上做零样本评测。用户研究从每个数据集各随机抽 20 个样本。
[不确定] 论文未披露数据合成前的原始视频池规模（小时数或条数），也未披露 data engine 实际生成了多少候选样本后才筛出 79K，因此整条pipeline的绝对吞吐量与筛选强度均无法量化。

### [2026 其他音视频联合生成](../models/JAVG_2026_misc.md) ⚠️

各工作规模跨度超过两个数量级，且披露口径不一：
【ALIVE（字节，规模最大且分阶段披露最细）】按训练阶段给出样本数而非小时数：
- 音频 T2A Stage I：384M audio samples（对应 700k hours of transcribed speech，用于 TTS 预训练）
- 音频 T2A Stage II：19M samples（约 5k hours 高质量语音 + 111k hours 带声音标注的视频数据集）
- T2VA 联合训练：11M samples @480p/24fps（1.2 epoch）
- T2VA+I2VA：同一 11M samples（0.3 epoch）
- Continue-training：4.3M balanced samples（3 epoch）
- SFT：5M samples（0.5 epoch）
- 1080p Refiner：0.7M high-clarity samples（1 epoch）
- Character-driven：0.8M reference-paired samples
论文自述为「continue pretraining and finetuning on million-level high-quality data」。原始候选池总量未披露[不确定]。
【NAVA（百度）】唯一给出完整漏斗两端的工作：原始采集「approximately 20M audio clips and 100M video clips」，过滤后「around 15M clips for large-scale training」用于大规模训练；其中 Koala-36M 子集约占最终语料 20%。SFT 阶段进一步收敛到 160K high-quality samples。平均视频时长约 7 秒（按 15M×7s 估算约 2.9 万小时[此换算为推断，不确定]）。
【CCL（商汤系）】主打小数据：Table 1 明确列出自身训练数据量 4M，并与 Ovi 的 30.7M、MOVA 的 50M 并列对比——即用 Ovi 约 1/8、MOVA 约 1/12 的数据量。正文描述为「million-level audio-video pairs」。
【Baton】1.5 million video-audio clips，原文「Our training dataset (1.5 million video-audio clips) is aggregated from OpenHuman-Vid, AudioCaps, WavCaps, and videos collected from the internet.」——七项中训练规模最小的联合生成工作之一。
【OmniCustom】自建 OmniCustom-1M：约 1 million 单人视频片段，合计 2,500 hours，从 SpeakerVid-5M（原始「more than 5.2 million video clips」/「8,000 hours」）中筛得。即片段级保留率约 19%、时长级保留率约 31%（2500/8000）。
【StreamChar】未给总量数字，只给来源（SpeakerVid-5M + TalkVid + OpenHumanVid 组合）与训练步数（编排器 80k steps @batch 640；联合训练 100k steps @batch 128）。语音预训练用 Emilia 数据集。明确约束「training data contains no videos/transcripts longer than 20 seconds」[总规模不确定]。
【ITS-JAVG】无训练数据（training-free）；「数据量」体现在推理搜索预算：JavisDiT 每 prompt 采 5 samples（Best-of-N），MMDisCo 每 prompt 采 10 samples。

### [音视频联合生成基线合集](../models/JavisDiT_baselines.md) ⚠️

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

### [可灵 3.0 / 可灵视频 3.0 Omni](../models/Kling_30_Omni.md) ⚠️

[不确定] 官方从未公布可灵3.0 Omni的训练数据量级（视频条数/小时数/token数），预训练与SFT规模均未披露。Kling-Omni 技术报告仅定性称“大规模真实世界数据采集 + 面向任务的合成数据构造”，无任何数字。可参考的同团队量级旁证：Koala-36M 开源数据集为3600万clip、平均13.75秒、720p（约13.7万小时）；Kling-Foley 的 TV2A 训练数据为12.2万小时/5500万个8秒clip、合计超1亿条视音文三元组样本。可灵3.0 Omni 的实际规模应显著大于上述公开数据集，但无公开依据。

### [LTX-2](../models/LTX-2.md) ⚠️

完全未披露。技术报告第5节「Training Data」全长仅两段（约150词），未给出任何视频条数、总时长、token 数，也未区分预训练与微调规模。仅定性说明使用「LTX-Video 所用同一数据集的一个子集（a subset of the same dataset employed in LTX-Video）」，该子集聚焦于「含有显著且信息量丰富的音频成分的视频片段」。前代 LTX-Video 论文同样未给出数据规模数字。[不确定]

### [LongCat-Video](../models/LongCat-Video.md) ⚠️

[不确定]。技术报告全文未披露任何训练数据量级——既没有视频条数、总时长（小时），也没有 token 数或图像张数，预训练与 SFT 的规模均未给出。可间接推算的只有训练迭代量与批次配置：预训练五阶段合计约 678k iterations（Stage1 T2I 256p 285k + Stage2 140k + Stage3 164k + Stage4 36k + Stage5 53k），SFT 阶段 7.5k iterations（480p+720p×93帧，lr 1e-5），RLHF（GRPO）阶段约 0.5k iterations（group size 4、每步 64 prompts）。Avatar 1.5 同样未披露数据总量（Stage1 256p×93帧 batch 64 共 130k iterations，Stage2 480p×93帧 batch 32 共 45k iterations）。评测集规模是唯一给出确切数字的数据：内部 T2V 1228 条（500 条人工评测 + 728 条自动评测，覆盖 48 个类别）、I2V 400 条（100 张首帧参考图 × 4 类 prompt）。

### [MOVA](../models/MOVA.md)

按训练阶段分列（论文以“小时数”与“clip 条数”双口径给出，未给出 token 数）：
【音频塔预训练】使用 WavCaps + VGGSound（通用音效）、JamendoMaxCaps（音乐）、in-house TTS 数据三大域，训练定长片段；具体小时数/条数未披露。
【联合训练 Phase 1（360p，多样数据）】约 61,500 小时视频-音频数据，1 个 epoch，耗时 15 天。
【联合训练 Phase 2（360p，质量过滤）】约 37,600 小时 / 16.8M clips（16.8M × 8.05s ≈ 37,560 小时，自洽），1 个 epoch，耗时 7 天。
【联合训练 Phase 3（720p，最高质量子集）】约 11,000 小时，1 个 epoch，耗时 20 天。
【总计】三阶段合计 42 天，1024 张 GPU（128 节点 × 8 卡），约 43,000 GPU-days。
注意：MOVA 无独立的 SFT / RLHF 后训练阶段，因此不存在“预训练 vs SFT”的规模拆分，取而代之的是三阶段渐进课程内部的数据规模递减（61.5k → 37.6k → 11k 小时），呈典型的“规模递减、质量递增”金字塔。

### [合并条目：① Mochi 1](../models/Mochi_MAGI_Motif.md) ⚠️

① Mochi 1：完全未披露。官方从未给出视频条数、小时数或 token 数，仅称模型「trained entirely from scratch」且为「当时公开发布的最大视频生成模型」。[不确定]
② MAGI-1：只给出原始素材量级——「from tens of petabytes of raw videos and images collected from a wide range of sources」（数十 PB 原始视频与图像）。清洗后的训练集条数、小时数、各阶段样本量均未披露；Table 5 只写明「数据量随阶段递减」而不给数字。预训练与 SFT 也未分开计量（报告中无独立 SFT 阶段，见 post_training_data）。[不确定]
③ Motif-Video 2B：给出明确且刻意压低的上界——「fewer than 10M clips」（少于 1000 万视频片段）与「less than 100,000 H200 GPU hours」（少于 10 万 H200 GPU 小时）。HF 卡片表述为「约 1000 万视频片段」。论文将此对照同期开源模型的「hundreds of millions of curated clips」（Wan2.1、HunyuanVideo、Seedance 等为数亿片段、5B–14B 参数），自称数据量低一个数量级、参数量少 7 倍。SFT 语料规模未给绝对数值，仅说明为「curated high-quality subset」并按 subject 类目迭代补充（Fig.8 给的是分布比例而非绝对量）。[部分不确定：SFT 精选集绝对规模]

### [Movie Gen](../models/Movie_Gen.md) ⚠️

预训练（视频）：O(100)M 视频-文本对 + O(1)B 图文对；原始视频池时长4秒~2分钟（平均28秒），清洗后每条clip为4~16秒单镜头片段。
后训练（视频SFT）：人工精选的小规模高质量视频+人工caption集合，论文未给出具体条数[不确定]；训练仅用512张H100（64节点），相对预训练最多6144张H100的规模小两个数量级。
个性化（PT2V）：从预训练集中筛出 O(1)M 条单人视频；采样得 O(10)M paired 样本、O(10)K 真实 cross-paired、O(1)M 合成 cross-paired；SFT 集为 O(1000) 条高质量单人视频。
音频预训练：总计 O(100)M 样本 / O(1,000)K 小时（即百万小时量级），其中 Sound 类独占 O(100)M / O(1,000)K，Music、Sound+Music、Sound+Voice、Sound+Music+Voice 各为 O(10)M / O(100)K。
音频微调：影视级音视频 O(100)K 样本 / O(1)K 小时，高质量纯音频（音乐 O(10)K 小时 + 音效 O(10)K 小时）O(1,000)K 样本 / O(10)K 小时。

### [NVIDIA NeMo Curator](../models/NeMo_Curator.md)

框架本身不含数据，此处记录其「已验证处理规模」与「设计容量」：
【设计容量】官方反复给出的口径是「可高效地对 100 PB 或以上量级的视频进行切分、标注与过滤」（clip, annotate, and filter 100 PB or more of videos）。
【实测规模（Cosmos WFM 生产运行）】输入约 2000 万小时（20M hours）原始视频，分辨率 720p–4K；输出约 1 亿（100M）个 2–60 秒的 clip；其中约 10^8 量级用于预训练、约 10^7 量级用于微调（fine-tuning）。
【处理耗时】2000 万小时视频：Hopper（H100）GPU 集群约 40 天完成；Blackwell 约 14 天；同等条件下未优化的 CPU pipeline 需约 3.4 年。另一组官方口径为「用 1000 张 GPU 在 ISO 功耗下相较未优化 CPU pipeline 达 89 倍加速，把处理时间从数年压缩到数天」。
注意：这些数字是 NVIDIA 自家 Cosmos 项目的运行规模，不是 NeMo Curator 用户的通用数字。

### [OmniHuman 数据集 + OHBench](../models/OmniHuman.md) ⚠️

【核心规模】100 万条视频（1 million videos），总时长 1,800 小时，覆盖 8 万个不同身份（80,000 distinct identities）。这三个数字是论文给出的完整规模口径，比多数同类工作（往往只给条数）更完整。
【平均片段时长的反推】1,800 小时 ÷ 100 万条 ≈ 6.48 秒/条，即数据以 6 秒量级的短片段为主，符合当前 AV 联合生成模型的训练窗口（5–10 秒）。这一反推值可与 Figure 3(d) 的时长分布图相互印证，但论文正文未直接给出平均时长。
【身份密度】8 万身份 / 100 万条 ≈ 平均每个身份 12.5 条片段，身份多样性显著高于 HDTF、CelebV-HQ 等传统说话人数据集（后者通常仅数百至数千身份），这对模型的身份泛化能力是关键优势，也是论文在 Table 1 中主打的差异点之一。
【预训练 / SFT 划分】数据集本身不做预训练与 SFT 的划分——它是一份通用训练资源，划分由使用方决定。论文自身的验证实验只取了其中 20%（18 万单人 + 2 万双人 = 20 万样本）用于微调 LTX-2，属于 SFT 性质的用法（详见 data_ablation）。
【评测子集】OHBench 共 509 条视频：331 条单人、128 条双人、50 条人-物交互。该子集从 OmniHuman 中挑选，且论文明确说明其与训练集之间存在 domain gap（domain gaps relative to training set），即刻意选取分布外样本以避免评测被训练集污染——这是基准构建上一个值得肯定的细节。
【未披露】原始采集量（过滤前总量）完全未给出，因此 100 万这一数字只有分子没有分母（详见 funnel_retention_rate）；token 数不适用；单人/双人/人-物三类在 100 万条中的具体条数只在 Figure 3(b) 中以图示给出，正文未给数值。[不确定]

### [Open-Sora 系列](../models/Open-Sora.md) ⚠️

【Open-Sora 2.0】按训练阶段给出数据量（论文表格口径为「参与该阶段训练的视频条数」）：阶段一 70M 视频（256px T2V）、阶段二 10M 视频（256px T2V/I2V）、阶段三 5M 视频（768px T2V/I2V）。报告未给出清洗前原始视频总量、总小时数或 token 数，也未单列 SFT 规模（阶段三 5M 高清子集即事实上的高质量精调集）。
【Open-Sora 1.2】原始池约 30M 视频片段（时长 2–16 秒），合计约 80k 小时；其中 Panda-70M 高质量子集 20M 条约 41k 小时；最终高质量阶段约 2M 片段、5k 小时。图像约 3M 张。
【Open-Sora Plan v1.3】图像 18.0M 张，视频约 28M 条（清洗前）；清洗后 Panda70M 部分保留约 19M 条（27% 保留率口径）。
【Open-Sora Plan v1.5】图像 1.1B 张（仅做分辨率检查，不做质量过滤），视频 40M 条高质量样本。
注：以上均为条数口径，两个项目都未公布训练 token 数，Open-Sora 2.0 未公布总小时数。[不确定]

### [Ovi](../models/Ovi.md) ⚠️

论文只给出量级描述，未给精确数字。
【音视频配对语料】「millions of videos」（数百万条视频），全部为内部（in-house）音视频语料；按每条 121 帧 @24fps ≈ 5.04 秒估算，若为 300 万条则约 4200 小时量级，但论文未确认条数，此换算为推断[不确定]。
【纯音频语料】预训练阶段「hundreds of thousands of hours of raw audio」（数十万小时原始音频），以人声语音为主，波形长度最长 12 秒；微调阶段为 5.04 秒定长波形，混入 VGGSound / AudioSet / WavCaps 公开音效数据 + 从内部音视频语料中抽取的音轨。
【训练步数侧推算】音频预训练 50k 步 × batch 2880 ≈ 1.44 亿样本次；音视频融合训练 40k 步 × batch 768 ≈ 3072 万样本次（为 epoch 换算参考，非数据条数）。
【Ovi 1.1】README 明确「Dataset includes 100% more videos」，即音视频数据集规模较初版扩大一倍，且改为原生 960×960 分辨率数据训练；绝对数值未公布[不确定]。
预训练与 SFT 的严格拆分数字未公开[不确定]。

### [Script-a-Video](../models/Script-a-Video.md)

论文披露的数据规模按用途分为「caption 侧」与「生成侧」两套，二者独立：
【caption 侧（用于训练 MTSS 标注模型）】500K 高质量视频片段（500,000 clips）。仅给出条数，未给出总小时数、平均时长、token 数。若按典型 5-10 秒片段粗估约 700-1400 小时，但论文无任何时长口径数据，此为推断而非披露。
【生成侧（用于训练 LTX-2 改造模型）】四套数据分阶段使用：
- ID Customization 阶段：400K identity-centric dataset（身份中心数据集），训练 3 epochs；
- Multi-shot Control 阶段：250K multi-shot sequences（多镜头序列），训练 1.5 epochs；
- Audio-Visual Synergy 阶段：870K cinematic pairs（电影级音视频对），训练 3 epochs；
- 最终联合微调阶段：60K high-fidelity cinematic alignment pairs（高保真电影级对齐对）+ 250K multi-shot sequences 交错混合，训练 15K steps。
生成侧数据合计规模约 1.5M 量级（含跨阶段复用），同样仅给条数不给小时数。
【评测侧】内部评测集 125 条单镜头样本 + 100 条多镜头样本，共 225 条。
【预训练 vs SFT 的拆分】caption 模型侧不存在预训练——直接在开源 Qwen3-Omni-Instruct 上做单阶段 SFT，500K 即全部 SFT 数据；生成侧同样是在已预训练的 LTX-2 上做多阶段微调，无从零预训练。

### [Seedance 2.0 与 Seedance 1.5 pro](../models/Seedance_20_Seedance_15_pro.md) ⚠️

[不确定] 两份报告均未披露任何训练数据量级数字（视频条数、小时数、token 数，预训练与 SFT 均未给出）。经全文检索，Seedance 1.5 pro（v1/v3）与 Seedance 2.0 报告中均不含 million/hours/minutes 等规模表述。网络上流传的「约 1 亿分钟（~100 million minutes）野外音视频片段」说法来自 emergentmind 等 AI 自动摘要站点并声称引自该论文，但原文并无此数据，属未经证实的二手推断，不应采信。Seedance 1.0 报告同样未给出绝对数据量。

### [SkyReels 系列](../models/SkyReels.md)

【SkyReels-V2】原始数据集规模为 O(100M) 量级（约1亿条视频样本），并配有 O(100M) 量级的概念均衡图像数据参与联合训练。自采媒体估算总时长「6.2M+ 小时（620万小时以上）」，来自120多个国家的「280,000+ 部电影」与「800,000+ 集电视剧」。打标模型 SkyCaptioner-V1 的训练集为「从最初的1000万样本中精选出约200万条概念均衡视频」。后训练阶段因概念均衡「导致数据量减少了50%」。
【SkyReels-V4】按训练阶段给出数据量（Table 1）：Stage1 文生图 30亿张图像（3 epoch）；Stage2 图文+视频 10亿图像 + 4亿视频（3 epoch）；Stage3 inpainting 任务同为 10亿图 + 4亿视频（2 epoch）；Stage4 混合分辨率 各1亿（2 epoch）；Stage5 高分辨率 480/720/1080p 各5000万（2 epoch）；Stage6 多模态条件 2000万图像 + 5000万视频（2 epoch）；音频主干预训练「数十万小时（hundreds of thousands of hours）」、单条最长15秒（3 epoch）；音视频联合训练用「50%视频数据 + T2A 数据」（2 epoch）；SFT 第一阶段 500万条视频（其中20%带多模态条件），SFT 第二阶段 100万条人工精选高质量视频。未给出总视频小时数与 token 数。

### [Sora 2](../models/Sora_2.md) ⚠️

完全未披露。System Card 未给出任何视频条数、总小时数、token/patch 数量，未区分预训练与SFT规模，也未披露算力预算。OpenAI 对外亦无任何官方数据规模口径。[不确定]

### [Step-Video-T2V](../models/Step-Video-T2V.md)

【预训练总量】技术报告明确给出：构建了包含 20 亿（2B）视频-文本对 与 38 亿（3.8B）图文对 的大规模数据集。这是本条目最硬的定量披露之一。
【各训练阶段实际消耗样本】
· Step-1 文生图（T2I）预训练：256px，约 3.8B 图像样本，253k 迭代；
· Step-2 文生图/视频（T2VI）联合预训练低分辨率档：192×320，约 10 亿（1B）视频片段可用，实际 seen 样本 6.44 亿（644M），430k 迭代；
· Step-2 高分辨率档：544×992，实际 seen 样本 2730 万（27.3M），46k 迭代；
· Step-3 文生视频（T2V）微调 / SFT：544×992，约 3000 万（30M）高质量视频；
· Step-4 Video-DPO：544×992，基于模型自生成视频构造偏好对；蒸馏（Turbo）数据集约 9.5 万（95k）样本。
【说明】原始视频池的小时数、切分前的原片条数、以及各级过滤的绝对淘汰量报告未给出（漏斗以 Figure 11 的相对条形图形式呈现，未标注绝对数值）。视频总时长（小时数）与 token 数均未披露。

### [UniTalking](../models/UniTalking.md) ⚠️

【最终产出】经过全流程过滤后的人物中心数据集共 230 万（2.3 million）条对齐音视频样本。这是论文给出的唯一一个数据规模数字。
【口径特点】仅给出「样本条数」，未给出小时数、未给出总时长、未给出 token 数，也未给出原始采集量——因此既无法换算为小时数与其他工作横向对比，也无法反推保留率。若按 Wan2.2-5B 基座的典型 5 秒片段粗略估算，230万条约合 3,200 小时量级，但论文未给出片段时长故此推算无依据。[不确定]
【预训练与 SFT 的划分】训练确实分两阶段，但两阶段用的是不同数据集而非同一数据集的粗/精划分：
- 阶段一（音频分支预训练）：使用「内部 TTS 数据」（internal TTS data），规模、语种、时长全部未披露；batch size 256，学习率 1e-5，共 100,000 步。[不确定]
- 阶段二（音视频联合训练）：使用上述 230 万条音视频数据；batch size 64，学习率 1e-5，共 100,000 步。
【衍生数据】除 230 万条主数据外，还为每条视频用 IndexTTS2 合成 3 条参考音频，即额外产生约 690 万条合成参考音频片段（每条 3–5 秒）。
【规模定位】230 万条在说话人生成方向属较大规模，但论文在 Future Work 中明确承认「模型性能受限于可用训练资源与数据规模，尤其是与闭源模型相比」，即团队自认数据量仍是瓶颈。

### [UniVerse-1](../models/UniVerse-1.md)

总计 7,685 小时音视频数据（论文正文与模型卡多处约数表述为「约 7,600 小时」），全部用于单一微调阶段，不存在预训练/SFT 的规模拆分——因为 UniVerse-1 的「预训练」是直接继承 Wan2.1 与 Ace-step 两个已有专家的权重，本身只做一次联合微调。
【三部分构成及各自小时数】
- 经严格核验的语音中心数据（speech-centric）：1,187 小时；
- 通用音视频数据（general-purpose）：3,074 小时；
- 公开数据集 VGGSound + AudioSet：3,422 小时。
【口径特点】仅给出小时数，未披露 clip 条数与 token 数，也未给出原始采集量（因此无法反推整体保留率）。
【训练量】有效 batch size 128，梯度累积 4 步，共 50k 步，AdamW，学习率 5e-6，FSDP 多节点分布式训练。未披露 GPU 卡数与 GPU-days。
【规模定位】7.6k 小时在同类工作中属于中小规模——约为 MOVA Phase 1（61.5k 小时）的 1/8，接近 MOVA Phase 3 高质量子集（11k 小时）的量级。论文在 Limitations 中明确将算力受限列为主因，并把「更大规模、更精细的数据策划」列为未来方向。

### [Unison](../models/Unison.md)

论文对数据规模的全部披露集中于 4.1 节一句话，分音视频与纯音频两块，且明确标注为「经自动化处理 pipeline 精炼后」（After refinement through our automated processing pipeline）的最终产出量：
【音视频联合训练数据】约 200 万条同步音视频 clips，总计超过 3,000 小时。
【纯音频训练数据】5,000 万条高质量音频片段，总时长超过 130,000 小时。
【可推算的平均片段时长】
- 音视频侧：3,000 h × 3600 s ÷ 2,000,000 ≈ 5.4 秒/条，属于短片段密集型语料，与 UniVerse-1 的约 5 秒窗口、MOVA 的 8.05 秒定长处于同一量级；
- 纯音频侧：130,000 h × 3600 s ÷ 50,000,000 ≈ 9.4 秒/条。
（以上为本条目基于公开数字的推算，论文本身未给出平均时长。）
【规模定位】3,000+ 小时音视频在同类工作中属小规模——约为 UniVerse-1（7,685 小时）的 0.4 倍、MOVA Phase 1（61.5k 小时）的 1/20，与 Sora 2/Veo 3 一类百万小时级工业语料相差 2~3 个数量级。但音频侧 130,000 小时的量级相当可观，接近专业 TTS/音频生成基座的语料规模——这一「音频重、视频轻」的极端配比直接由其训练策略决定：联合训练阶段视频骨干完全冻结，Unison 实际上只训练音频分支与融合模块，视觉能力全部继承自 Wan2.2-5B，因此不需要大规模视频语料。
【预训练/SFT 拆分】不适用。论文的两阶段划分是「Stage 1 音频分支单模态训练」与「Stage 2 联合微调」，前者消费 130,000 小时纯音频，后者消费 3,000+ 小时音视频，属按模态而非按预训练/SFT 划分。无独立 SFT 阶段，无后训练数据。
【未披露】原始采集/聚合总量（分母）、各源数据集的贡献小时数与条数、token 数、Stage 2 的训练步数与 epoch 数、语料的时长直方图。

### [Veo 3 / Veo 3.1](../models/Veo_3_Veo_31.md) ⚠️

[不确定] 官方从未披露任何训练数据量级——无视频条数、无小时数、无 token 数，也未区分预训练与 SFT 规模。技术报告仅表述为「We train on a large dataset comprising images, videos, and associated annotations」。第三方博客流传的「数十亿音视频对」「数百万小时配对音视频」等说法均无官方出处，不可采信。可作弱旁证的是：训练使用 Google TPU Pod 集群、JAX 与 ML Pathways，暗示规模处于同期最大级别之一。

### [Vidu S1](../models/Vidu_S1.md) ⚠️

[不确定]。技术报告完全未披露训练数据的绝对规模（未给出视频条数、小时数、token 数），也未区分预训练与后训练的数据量。仅定性描述为「高质量、多样、强交互性的单人单镜头视频语料（a corpus of high-quality, diverse, and highly interactive single-person, single-shot video）」。三阶段训练（双向教师训练 / 因果自回归适配 / DMD 蒸馏）各阶段用量同样未公开。

### [主流视频预训练数据集合并调研：Panda-70M、InternVid、Koala…](../models/pretraining_datasets.md) ⚠️

七者规模跨越四个数量级，且**规模与质量呈明显反向排列**（下列按clip数降序，口径为各自论文Table 1；UltraVideo Table 1 提供了统一交叉校验）：
1) **InternVid**：7.1M 源视频 → **234M clips**，760.3K 小时，caption 总计41亿词，平均clip 11.7秒、caption 17.6词；源视频平均时长351.9秒。子集：InternVid-10M-DIV（多样性采样1000万）、InternVid-10M-FLT（在DIV基础上取UMT-SIM分**前30%**）、InternVid-18M-AES（美学分≥4，1800万）。
2) **Panda-70M**：3.79M 源长视频 → **70,817,169 clips**（论文口径）/ **70,723,513**（实际发布口径，差约9.4万条系发布前的有害内容过滤，作者未明说，属推断），166.8K 小时，约36TB，平均clip **8.477秒**、caption **13.2词**。发布子集：10M（matching_score>0.43 且每源视频最多3条，10,473,922条/37.0K小时）、2M（从10M中每源视频恰取3条，800K×3=2,400,000条/7.56K小时）、验证/测试各2000源视频×3条。**注意论文里的「Panda-2M/Panda-5M」是随机子集，与仓库发布的质量过滤版2M/10M不是一回事，引用时极易混淆。**
3) **Koala-36M**：**36M clips**，172K小时（Table 1），平均clip 13.75秒、caption **202.1词**，720p。**该表存在自洽性问题：36M×13.75s=137.5K小时，与172K小时相差约25%**（UltraVideo Table 1 转引时写作137K小时/13.6秒，即采信了乘积口径）。另外论文摘要反复说「over 10M」，那是与 MiraData/VidGen/OpenVid（均≤1M）对比时的门槛表述，不是第二个计数。
4) **LVD-2M**：**2M clips**（发布约2.1M行），每条≥10秒，平均20.2秒、caption 88.7词，平均光流分47.8。**总时长论文未给**，按2M×20.2s推算约**11,200小时**（UltraVideo 表转引为14.6K小时/20.2秒/2.1M）。时长分布：10–15s约43.5%、15–20s约23%、20–30s约20.5%、30–50s约11%、>50s约2.5%。
5) **OpenVid-1M**：**1M clips**，其中 **OpenVidHD-0.4M = 433K 条 1080p**；按 UltraVideo Table 1：OpenVid-1M 平均7.2秒、2.1K小时、caption 126.5词；OpenVidHD 平均9.6秒、1.2K小时、caption 104.5词。HF 数据集卡显示「1.45M rows」（含HD子集重复计数）。**总时长/平均时长/FPS 原论文未披露**。
6) **MiraData**：未过滤池 **788K**，发布四档嵌套子集 **330K / 93K / 42K / 9K**。平均clip **72.1秒**（七者中最长）、caption **318词**，720p。**Table 1 的「16K小时」对应的是788K未过滤池（788K×72.1s≈15,785h），不是330K发布版——330K 按同口径约6,600小时**，这是极易误引的一处。（外部综述所称「77k long videos」系误引 v0 beta 的57,803条/1,754小时，勿用。）
7) **UltraVideo**：short 分割 **42,184 clips / 62小时 / 平均5.3秒 / caption 824.2词**；long 分割 **16,597 clips / 143小时 / 平均30.9秒 / caption 850.3词**。源视频仅 **5,000 条**。规模比 Koala-36M 小约2700倍（按小时计），但caption长约4倍、像素量高4–16倍。**注意其split命名是 short/long，不存在「UltraVideo-1K/42K」；1K/4K 指的是 UltraWan 模型的输出分辨率。**

## 数据来源构成（自有/公开数据集/网络爬取/授权采购/合成数据）

`data_sources` · 详细程度: brief

### [Allegro](../models/Allegro.md) ⚠️

以公开数据集为主要来源，论文明确表述「现有公开数据集，如 WebVid、Panda-70M、HD-VILA、HD-VG 和 OpenVid-1M，为数据来源提供了坚实基础」。在此基础上团队自行执行完整的重新切分、过滤与重新打标，构建出自己的 106M 图像 + 48M 视频语料，而非直接使用原数据集的原始 caption 与切分。
未披露自有采购/授权数据、独家版权库或合成数据的使用；也未给出各来源数据集的具体配比。图像侧的 412M 原始图像来源未点名说明。[不确定]（各来源占比、是否掺杂自有网络爬取）

### [Apollo](../models/Apollo.md) ⚠️

论文对数据来源构成的披露基本为空白：未列举任何公开数据集名称，未说明自有数据、网络爬取、授权采购各自的占比，也未提及数据获取渠道。可作出的有限推断：
- 数据形态覆盖单说话人语音、多说话人语音、歌唱、自然声（natural sound）四类场景，且要求原生同步音轨（因为整个 pipeline 建立在对已有视频的音轨做质量与同步过滤之上，而非配音合成），说明数据来自真实带声视频而非无声视频后补音。
- 8100 万条的量级、以及快手作为短视频平台的属性，强烈暗示以平台自有/授权的短视频语料为主，但论文未作任何说明。
- 合成数据：论文未使用模型合成的音视频内容，唯一的「合成」成分是全部 caption 由 ASR/音频 captioner/视频专家模型自动生成（属合成标注而非合成内容）。
[不确定]

### [CineDance / CineDance-1M](../models/CineDance.md) ⚠️

以公开数据集二次策展 + 自主采集为主，未提及授权采购或合成数据：
【公开数据集来源】明确列出复用 MiraData、LVD-2M、Koala-36M 三个已有大规模视频数据集作为素材池之一。
【采集流程借鉴】声明遵循 SkyReels-V2 与 OpenHumanVid 的数据采集 pipeline 规范进行网络素材收集。
【内容类型】以电影 / 影视 / 剧情类长视频为主（cinematic），强调「故事片式」的多镜头叙事内容，故原始素材以长片而非 UGC 短视频为主体——原始仅 45,181 条视频却撑起 32.8K 小时，平均单条约 43 分钟，印证素材为长片级内容。
【硬性准入】最低 1080p 空间分辨率，且必须自带原生音轨。
【合成数据】无。全部为真实拍摄素材，未构造任何合成或编辑样本。
【各来源具体占比】未给出 MiraData / LVD-2M / Koala-36M / 自采各自的贡献比例。[不确定]

### [CogVideoX](../models/CogVideoX.md) ⚠️

论文仅笼统表述为来自互联网的视频（「Videos from the Internet usually include a significant amount of low-resolution ones」），未披露具体渠道、采购或授权方式，判断以网络爬取 + 内部数据池为主 [具体构成不确定]。可确认的部分：
· 图像侧明确使用两个公开数据集：LAION-5B 与 COYO-700M（经美学分过滤后取 2B）。
· 打标环节借用了公开视频 caption 数据集/模型：Panda-70M 的 caption 模型用于产生短 caption（论文同时批评 Panda70M、COCO Caption、WebVid 的 caption 过短、描述不全面，因此不直接使用其文本）。
· caption 训练数据部分为合成数据：由 CogVLM 逐帧图像 caption + GPT-4 摘要产生，属于「模型生成的合成文本」。
· 未提及授权采购数据、影视素材库或合成视频数据。

### [Cosmos-Predict2.5](../models/Cosmos-Predict25.md)

构成为「专有数据集 + 公开互联网平台 + 公开机器人数据集 + NVIDIA 内部采集」四类，但通用预训练部分只给定性描述、无比例：
(1) 通用预训练：「sourced from both proprietary datasets and open internet platforms」（专有数据集与公开互联网平台），覆盖 driving（驾驶）、object manipulation（物体操作）、spatial navigation（空间导航）、human interaction（人类交互）、nature scenes（自然场景）等域；两类来源的占比未披露。
(2) 机器人域为点名的公开数据集集合，且给出按相机视角的条数（中央视角/左视角/右视角）：AgiBot-Beta（双臂）194k/30k/30k、Bridge（单臂）36k、DROID（单臂）39k（腕部视角）/51k/51k、GR00T（双臂）3k、1X Technologies（双臂）17k、OpenX（单臂）500、RoboMIND（双臂/人形）16k/6k/7k。
(3) 自动驾驶为 NVIDIA 自有采集：「a proprietary dataset ... collected using NVIDIA's internal driving platform」，约 3.1M 段 20 秒环视 clip，7 路同步相机（front-wide、front-tele、left、right、rear、rear-left、rear-right）。
(4) Smart Spaces（仓库/工厂/工地等）：先用检索关键词召回候选视频，再用 VLM（Qwen2.5-VL 系）逐条判定相关性，切分过滤后约 40K clip 存活。
(5) 合成数据：预训练阶段主动排除（游戏画面、合成视觉图案、动画、卡通均被剔除）；但下游应用中 Cosmos-Transfer2.5 被用来生成合成增强数据训练机器人策略（见 synthetic_data_synthesis）。

### [Data-Juicer 2.0](../models/Data-Juicer.md)

作为工具，Data-Juicer 本身不持有数据，其数据来源问题体现在两处。
【官方案例的数据来源】文生视频案例完全使用公开数据集，无自有数据、无网络爬取、无授权采购、无合成数据：InternVid（上海AI Lab，YouTube来源大规模视频-文本数据集，贡献 606k）、Panda-70M（Snap/UCM，HD-VILA-100M 衍生的高质量视频-caption 数据集，贡献 605k）、MSR-VTT（微软经典小规模视频描述数据集，贡献 6k）。三者合计约 121.7 万条，可见配比上 InternVid 与 Panda-70M 近乎各半、MSR-VTT 仅作少量补充（0.5%）。
【系统支持的数据接入面】原生兼容 HuggingFace Datasets、ModelScope 数据集、本地文件系统、阿里云 OSS/NAS/CPFS、AWS S3（v1.4.4 起）等多种数据源；支持压缩数据集格式（v1.5.1）与视频字节流 I/O（v1.4.6，便于不落盘处理视频）。
【合成数据能力】229个算子中约50个专门用于数据合成与增广，DJ-Cookbook 内含「视频数据合成」（video-data-synthesis）YAML 配方，即 DJ 不仅能筛选真实数据，也支持构造合成数据——这与 NeMo Curator 侧重清洗的定位有所差异。

### [Foley-Omni](../models/Foley-Omni.md)

混合来源，公开数据集 + 内部自有数据两大类，未提及网络大规模爬取或授权采购。
【公开数据集】语音类：LJSpeech、LibriTTS（TTS基础）；GRID、LRS2、Chem（经典唇读/视觉语音数据集）；SpeakerVid、TalkVid（较新的大规模说话人视频数据集，同时用于 VisualTTS 与 V2ST）。音效类：AudioCaps、Freesound、VGGSound、Kling-Foley（快手可灵团队发布的foley数据集）。音乐类：MusicCaps、MusicBench、AudioSet。
【内部数据（internal）】在 TTS（内部语音库）、V2A（内部音视频语料）、V2ST（内部数据，构成216小时中的主体）三处出现，是本文数据清洗pipeline的主要作用对象——论文将这部分描述为「weakly labeled audiovisual data」（弱标注音视频对），即只有原始视频+原始音轨、缺乏组件级文本标注的原始素材。
【合成/构造数据】无。三字段标注由 Gemini 2.5 Pro 生成属于「模型标注」而非「合成音视频数据」，pipeline中不涉及人工扰动构造训练对。
值得注意的是数据来源的任务导向性很强：VisualTTS 占 1,980 小时（约40%）说明模型对「说话人视频→语音」这一支路投入最大，与其主打的 speech intelligibility（WER 7.59，甚至优于 Ground Truth 的 8.03 在部分对比中接近）优势直接对应。

### [Goku](../models/Goku.md)

四类混合：
(1) 公开数据集——图像侧为 LAION（Schuhmann et al., 2022，1亿样本）；视频侧明确列出 Panda-70M、InternVid、OpenVid-1M、Pexels，合计 1100 万片段。
(2) 自有/内部数据——图像侧 6000 万「高质量内部样本」，视频侧 2500 万自有片段（占视频数据约 69%，为主力来源）。内部数据的具体来源渠道（是否来自抖音/TikTok 生态、是否为授权采购）论文未说明。
(3) 网络爬取——未显式声明，但公开数据集本身（LAION、Panda-70M、InternVid）均为网络爬取来源。
(4) 合成数据——论文未使用合成视频数据；仅在评测环节用 GPT-4o 改写 GenEval 短提示词（属评测侧，非训练数据合成）。
整体呈现「公开数据打底 + 内部高质量数据精调」的经典字节系配方。

### [HunyuanVideo-Foley](../models/HunyuanVideo-Foley.md) ⚠️

论文对数据来源的披露极为克制，这是本工作数据侧最大的信息缺口：
【自建 TV2A 数据（10 万小时）】仅描述为「收集的音视频数据」（collected audio-visual data），未说明是网络爬取、腾讯自有内容库（腾讯视频/微视等）、商业采购还是三者混合，也未给出任何平台名称或内容来源清单。考虑到腾讯自身拥有大规模视频内容资产，且论文刻意回避来源描述，推测以自有/授权内容库为主、辅以网络采集，但无任何证据支撑。[不确定]
【明确点名的公开数据集（仅用于评测或 VAE 评测，非主训练）】VGGSound（论文提及约 550 小时，用作 VGGSound-Test 评测）、AudioCaps、WavCaps、AudioSet（用于 DAC-VAE 的重建质量评测）、Song Describer（音乐重建评测）、LibriTTS（语音重建评测）。这些数据集在论文中主要出现在评测环节，未明确声明被纳入 10 万小时主训练集。
【合成数据】未使用任何模型合成的音频或视频内容。唯一的「合成」成分是全部 caption 由 GenAU 模型自动生成（属合成标注而非合成内容）。
【授权采购】未提及。
【与同类工作的对比】UniVerse-1 明确列举 YouTube 题材清单与 Pexels，MOVA 也给出来源构成；HunyuanVideo-Foley 则完全不谈来源，只谈处理流程——这是一种「披露方法、隐藏原料」的策略，在商业公司主导的工作中较常见。

### [HunyuanVideo](../models/HunyuanVideo.md) ⚠️

两代均只作定性描述，未给出来源构成比例。原版称原始数据池覆盖人物、动物、植物、风景、载具、物体、建筑、动画等多个domain，来源未指明（隐含为网络爬取+自有/授权素材库混合）。1.5 报告表述为「视频来自多种渠道（a variety of channels），确保在内容、拍摄手法、镜头运动、风格、场景上的全面覆盖」，同样未区分自有/公开数据集/爬取/采购/合成。两代均未使用合成视频数据作为主训练语料（未提及）。图像侧 1.5 明确复用 HunyuanImage-3.0 的数据获取与处理pipeline。[不确定]

### [InstructAV2AV](../models/InstructAV2AV.md)

构成上属于「公开数据集 + 网络爬取 → 受控合成」的两段式结构，这是本工作与其他调研对象最本质的差异——最终训练数据中的 target 侧全部是模型合成产物，而非真实拍摄素材。
【第一段：真实素材来源（用作 source 与合成基底）】
  · 网络平台爬取：明确提到 YouTube（论文表述为 publicly accessible online platforms, e.g., YouTube）。
  · 公开数据集四种：MovieBench（Wu et al., 2025b，电影级长视频理解数据集）、Condensed Movies（Bain et al., 2020，电影关键片段集）、Short-Films-20K（Ghermi et al., 2024，短片数据集）、VGGSound（Chen et al., 2020，约31万条10秒野外音视频事件片段，310类声源）。
  来源构成的取向很明确：三个电影/短片类来源提供叙事性、有对白、有专业混音的高质量素材（支撑语音编辑与身份保持任务），VGGSound 提供明确声源事件的素材（支撑音效类实例编辑/插入/移除任务）。这种「影视 + 事件」二元配置与其四类编辑任务一一呼应。
【第二段：合成数据（target 侧）】
  target 视频由自建的 mask-guided 编辑模型（基于 Wan2.2-5B）合成；target 音频由 SAM-Audio 分离 + ElevenLabs 文本转音频/语音合成 + 与原背景音混合而成。因此每一对样本中，source 是真实的、target 是合成的，指令是 Qwen3-Omni 生成的。
【授权采购】无。未提及任何付费授权或 rights-cleared 数据采购。
【自有数据】无。未提及内部私有数据。
【方法论意义】这种「以真实素材为锚、用生成模型制造受控变化」的做法，是解决配对监督数据不存在问题的通用范式：现实世界中不存在「同一场景的编辑前后两份真实录像」，只能合成。代价是 target 侧继承了 Wan2.2-5B 与 ElevenLabs 的能力上界与失真模式，模型学到的编辑效果无法超越数据引擎本身——这是合成数据路线的固有天花板，论文未讨论此风险。

### [2026 其他音视频联合生成](../models/JAVG_2026_misc.md) ⚠️

呈现明显的「公开数据集复用 + 自有/爬取补充」两段式，且 OpenHumanVid 与 SpeakerVid-5M 成为 2026 年 JAVG 领域的两个事实标准上游：
【Baton】OpenHuman-Vid + AudioCaps + WavCaps + 互联网自采视频（internet-collected videos）四路混合，无授权采购与合成数据描述。
【OmniCustom】单一上游：SpeakerVid-5M（公开的音视频双人交互人体生成数据集），经自建规则筛出 OmniCustom-1M。评测集另含 70 段 YouTube 视频 + 30 位未出现在训练集的真人。
【StreamChar】三个公开人体/说话视频数据集拼合：SpeakerVid-5M（大规模高质量音视双人交互人体生成）、TalkVid（大规模多样化音频驱动说话头合成）、OpenHumanVid（大规模高质量以人为中心视频生成）；语音侧用 Emilia 大规模多语种语音数据集。全部为公开数据，是七项中数据来源最「学术可复现」的一项。
【ALIVE】未点名具体数据集，描述为「raw data pool」中先筛出「videos with audio」的自有大规模语料；音频侧有 700k hours 转写语音（疑为字节内部 Seed 语音语料[不确定]）、5k hours 高质量语音、111k hours 带声音标注的视频数据集。另有大量合成/增强构造（见 synthetic_data_synthesis）。整体为内部语料主导。
【CCL】OpenHumanVid（公开）+ in-house collections，后者明确覆盖「interviews, short dramas, and films」（访谈、短剧、电影）三类——这是一个很具体的来源画像，说明其数据偏「有人说话的叙事内容」；音频预训练额外引入学术数据集 WavCaps 与 VGGSound。
【NAVA】三路构成：Koala-36M（公开大规模视频数据集，约占最终语料 20%）+ TED-style speech videos（TED 风格演讲视频，为高质量单人语音源）+ raw movie/TV footage（影视原片）。是七项中唯一显式说明「影视原片」作为来源的工作。
【ITS-JAVG】无训练数据；评测用 VGGSound test set 与 JavisBench-mini。
【共性观察】七项中无一披露授权采购数据、无一使用 C2PA 类溯源，且四项（Baton/CCL/NAVA/ALIVE）都含互联网自采或影视原片，合规风险集中于此。

### [音视频联合生成基线合集](../models/JavisDiT_baselines.md) ⚠️

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

### [可灵 3.0 / 可灵视频 3.0 Omni](../models/Kling_30_Omni.md) ⚠️

定性披露为三类混合：(1) 自动化大规模互联网视频/图像挖掘——报告称使用自研 embedding 模型在海量互联网数据中检索语义相关的跨模态样本；(2) 自有/采购的影视级高质量素材，Kling-Omni 与 KlingAvatar 2.0 均强调“电影级（cinematic-level）视频数据”，KlingAvatar 2.0 明确提到播客、访谈、多角色影视剧作为多说话人对话数据源；(3) 专家模型驱动的合成数据 pipeline——用自研图像编辑模型与视频理解模型反向构造编辑/多图参考训练对。此外快手拥有短视频平台自有内容池，理论上是重要来源，但官方未确认其用于可灵训练。[不确定：自有平台数据、授权采购、爬取三者的具体占比]

### [LTX-2](../models/LTX-2.md)

构成为「公开可得数据 + 授权采购素材」的混合。LTX-Video 论文原文：「Our training dataset comprises a robust collection of publicly available data, supplemented with licensed material」。LTX-2 直接复用该数据池的音频信息量子集。
授权来源有明确公开旁证：(1) Shutterstock——2024年12月官宣，Lightricks 是全球首个采用 Shutterstock「research license（研究许可）」训练开源模型的伙伴，可使用其 HD 与 4K 视频素材库；(2) Getty Images——2025年在研发 13B 模型期间建立战略合作，获取高质量视频素材库。此外训练中混入图像数据集（LTX-Video 明确将图像作为一种「分辨率-时长组合」参与训练，用以引入视频数据中不常见的概念）。未使用合成数据的说明。各来源占比未披露。

### [LongCat-Video](../models/LongCat-Video.md) ⚠️

报告仅笼统表述为「We collect raw video data from a variety of sources」（从多种来源收集原始视频数据），未区分自有业务数据/公开数据集/网络爬取/授权采购的构成比例，也未点名任何具体数据集。可确认包含图像数据与视频数据两大类（Stage1 为纯 T2I 图像训练，Stage2 起图像与视频混合）。SFT 阶段额外「incorporated specialized datasets」以增强指令跟随能力，特别针对相机运动（camera motion）与视觉风格（visual style）两类专项数据集，来源未说明。
同系列 Avatar 1.5 披露了按功能划分的六类数据来源，可作为团队数据组织思路的旁证：(1) 近景人脸视频（用于面部建模与唇动）；(2) 访谈类视频（主体稳定、语音清晰）；(3) 表演类视频（提供镜头语言与姿态变化）；(4) 交互类视频（物体操持与手势）；(5) 音乐视频（歌唱与节奏性动作）；(6) 动画与风格化视频（用于非写实域泛化）。各来源占比未披露。[不确定：具体来源构成与配比]

### [MOVA](../models/MOVA.md)

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

### [合并条目：① Mochi 1](../models/Mochi_MAGI_Motif.md) ⚠️

① Mochi 1：官方仅有一句口径——Paras Jain 对 VentureBeat 表示「Generally, we use publicly available data and sometimes work with a variety of data partners」（一般使用公开可得数据，有时与各类数据合作方合作），并以竞争原因拒绝细说。既未点名任何公开数据集，也未说明爬取/授权比例。[不确定]
② MAGI-1：仅表述为「collected from a wide range of sources」（来源广泛），未区分自有爬取、公开数据集、授权采购或合成数据，未点名任何来源数据集。从「数十 PB 原始素材 + 自建可扩展数据处理系统 + PySceneDetect 从长视频切分」可推断以自有大规模网络爬取为主，但论文未明说。[不确定]
③ Motif-Video 2B：明确为两路来源——「an internal web-scale video collection（自有网络级视频爬取）」与「a set of publicly available image and video datasets（一批公开图像与视频数据集）」，两路共用同一套下游 sanitation / 过滤 / 去重 / 分阶段质量控制流程，以保证最终语料由单一标准治理。此外把原始池显式拆为 Image Real / Image Synthetic / Video Real / Video Synthetic 四个分支，即承认使用合成数据，且规定「Synthetic video is injected only at 720p（合成视频只在 720p 阶段注入）」，理由是其可控质量最契合该阶段的准入标准。但未点名具体公开数据集名称，也未给各来源配比。[部分不确定：具体公开数据集名单与配比]

### [Movie Gen](../models/Movie_Gen.md) ⚠️

论文未披露数据的具体来源渠道与授权方式，仅以「a large pool of videos」「sourcing data from a large volume」描述，可判断为 Meta 自有的大规模内部视频/音频数据池 [来源构成不确定]。图文数据的清洗策略沿用 Meta 自家 Emu（Dai et al., 2023）的做法。可确认的构成特征：原始视频覆盖人类、自然、动物、物体等多个domain；音频预训练数据来自视频的原生音轨；音频微调额外使用了不带视频的高质量纯音乐与纯音效素材库（专业制作素材）。合成数据被显式使用于个性化（personalization图像生成模型造参考图）与视频编辑（仿射动画化的图像编辑对、生成式指令分割、backtranslation 反向配对）。未使用公开学术数据集作为训练主体（VGGSound 等仅用于评测）。

### [NVIDIA NeMo Curator](../models/NeMo_Curator.md) ⚠️

作为工具链，它是「数据源无关（source-agnostic）」的，本身不规定数据来源。已支持的数据接入方式包括：本地文件系统、S3 兼容对象存储（26.04 新增 CommonCrawl S3 传输选项）、自定义 manifest、Hugging Face 数据集（音频侧示例为 FLEURS）、以及 WebDataset 格式的读写。
其上游生产用例（Cosmos WFM）的数据来源构成为：专有/自有视频数据集 + 公开互联网视频，并明确包含合成渲染视频（synthetically rendered，占最终配比约 4%）。NVIDIA 未披露各来源的具体占比、采购渠道或授权方式。[不确定]

### [OmniHuman 数据集 + OHBench](../models/OmniHuman.md) ⚠️

【唯一来源：YouTube 网络视频】HuggingFace 数据集页显示每条样本携带 YouTube 源 URL 与 clip 起止秒，这是本数据集来源的确凿证据——数据为网络爬取（web-crawled）的 YouTube 内容，不含自有拍摄、不含授权采购、不含合成数据、不复用已有公开数据集。
【论文正文的表述缺失】值得指出的是，论文正文本身并未说明视频来自何处，只笼统称使用了「全自动 pipeline」进行采集（fully automated pipeline for high-quality data collection）。来源信息完全依赖 HuggingFace 发布物的字段结构才得以确认，这是一个不小的披露疏漏——读者仅凭论文无法判断数据的合法性基础与潜在分布偏斜。
【采集导向：按场景与内容类型定向采集】从其覆盖的 8 类场景（居家空间、办公室、自然环境、城市/乡村景观、演出场馆、零售、工业区等）与 8 类内容（脱口秀、教育、烹饪、体育、音乐、游戏、测评、影视）看，采集应是按预设类目做定向检索而非无差别爬取——这正是其针对「现有数据集场景多样性不足」这一痛点的直接回应（传统说话人数据集集中于演播室半身正面镜头，本数据集刻意扩展到多样真实场景）。
【与 OpenHumanVid 等的关系】Table 1 中将 OpenHumanVid、SpeakerVid-5M、CelebV-HQ 等列为对照而非上游，说明未复用这些数据集的样本。
【合成数据】无。视频侧与音频侧均未使用任何合成或 TTS 生成内容（对比 UniTalking 用 IndexTTS2 合成 690 万条参考音频）。[不确定]

### [Open-Sora 系列](../models/Open-Sora.md) ⚠️

两个项目均**完全依赖公开数据集与免费授权素材站，无自有版权库、无采购授权数据、无合成数据**，这是与 Sora 2 / Veo 3 / Seedance 等闭源模型最本质的差别，也是其数据配方可完全复现的原因。
【Open-Sora 1.2 明确列出】Webvid-10M（1000万视频-文本对，stock footage）、Panda-70M（7000万对，取 20M 高质量子集）、HD-VG-130M（1.3亿对，BLIP-2 caption）、MiraData（7.7万长视频，游戏/城市漫游）、Vript（40万密集标注视频）、Inter4K（1000条 4K 片段）、以及 Pexels / Pixabay / Mixkit 等免费许可素材站；图像用 LAION 子集（美学分>6.5）与 Unsplash-lite。
【Open-Sora 2.0】技术报告**未披露**数据来源与具体数据集名称，只描述了过滤方法，属于该报告的显著信息缺口（推测延续 1.x 的公开数据集组合，但无官方确认）。[不确定]
【Open-Sora Plan v1.3】图像：SAM 11.1M（配 LLaVA caption）、Anytext-3M 英文子集 1.8M（约占该集 50%）、LAION-5B 中筛出的 16 万张高质量人像、内部 QWen2-VL 标注数据 5.0M；视频：Panda70M 21M（横屏）、VIDAL 3M（竖屏，来自 YouTube Shorts）、ShareGPT4Video 0.8M（源自 Mixkit / Pexels / Pixabay 的 CC0 素材）。
【Open-Sora Plan v1.5】图像来自 Recap-DataComp-1B、COYO-700M、LAION-Aesthetics；视频来自 Panda-70M 与「内部来源（internal sources）」——v1.5 首次引入未公开的内部视频数据，是该系列数据透明度的一处倒退。

### [Ovi](../models/Ovi.md) ⚠️

三类构成，界限清晰：
(1) 内部自有音视频配对语料（internal audio-video corpus）——占核心地位，论文称「composed of human and nonhuman data from diverse contexts」（涵盖有人与无人的多样场景），来源渠道未披露[不确定]。
(2) 内部纯音频语料（internal collections）——用于音频塔从零预训练，以人类语音为主，强调语言多样性、韵律与音色变化。README 称「high quality in-house audio datasets」。Character AI 作为对话陪伴产品公司，推测其内部语音资源与产品语音数据相关，但论文未说明[不确定]。
(3) 公开数据集——仅用于音频微调阶段补充音效能力：VGGSound（Chen et al., 2020）、AudioSet（Gemmeke et al., 2017）、WavCaps（Mei et al., 2024）。
(4) 无授权采购数据的说明，无合成数据构造环节的描述。
另外模型层面复用了大量开源资产（Wan2.2 5B 视频权重、Wan 的 T5 与视频 VAE、MMAudio 的 16kHz 音频 1D VAE、BigVGAN 声码器），可视为「间接继承了 Wan2.2 预训练语料的分布」。

### [Script-a-Video](../models/Script-a-Video.md)

披露极为简略，仅有领域级定性描述，无来源渠道构成比例：
【caption 侧 500K】明确表述为「internal dataset」（腾讯自有内部数据），领域覆盖三类：film（电影）、television（电视剧）、lifestyle（生活方式类内容）。未说明这些内容是自有版权库、采购授权、还是网络采集。
【生成侧】四套数据的来源同样未披露渠道，仅从命名可推断性质：identity-centric（以人物身份为中心，应为含明确人物主体的素材）、multi-shot sequences（多镜头序列，应来自影视剧等天然多镜头内容）、cinematic pairs（电影级音视频对）、cinematic alignment pairs（高保真电影级对齐对）。
【评测侧 225 条】覆盖 movie and TV drama clips（电影与电视剧片段）、short-form videos（短视频）、indoor scenes（室内场景）、outdoor scenes（室外场景）四类。
【公开数据集】未使用任何公开视频数据集训练（VGGSound、AudioSet、Panda-70M 等均未提及）；使用的公开资源仅限评测基准（Video-SALMONN-2 testset、UGC-VideoCap、Daily-Omni、WorldSense）。
【合成数据】未使用合成视频/音频；但 500K 数据集的「标注」全部由 Gemini-2.5-Pro 自动生成，属合成标注而非合成内容。
【采购授权】未提及。
整体来看，数据来源披露程度显著低于同类工作（如 UniVerse-1 逐项列举 YouTube 内容类型、MOVA 给出逐级构成），属于典型的大厂内部数据「只说领域不说渠道」披露风格。

### [Seedance 2.0 与 Seedance 1.5 pro](../models/Seedance_20_Seedance_15_pro.md) ⚠️

Seedance 1.5 pro 仅表述为「大规模混合模态数据集」，未拆解来源构成。Seedance 2.0 未披露。可参照同团队 Seedance 1.0 报告的表述：以「合乎伦理与法律的方式获取」（ethically and legally sourced），来自多样化的公开仓库与已授权仓库（diverse public and licensed repositories），并与图像数据联合训练（图像数据准备沿用 Seedream 的方法论）。自有/爬取/采购/合成各自占比均未披露。[不确定：1.5 与 2.0 的具体来源构成与配比]

### [SkyReels 系列](../models/SkyReels.md)

【SkyReels-V2】三类来源明确：(1) 通用开源数据集，点名 Koala-36M、HumanVid；(2) 自采媒体——120多个国家的28万+部电影与80万+集电视剧（估计620万+小时），即以正片影视为主的高质量素材；(3) 艺术素材库（artistic repositories），来自互联网的高质量视频资产。
【SkyReels-V4】分「真实数据 + 合成数据」两支。真实数据 = 公开数据集 + 授权自有内容：图像点名 LAION、Flickr；视频点名 WebVid-10M、Koala-36M、OpenHumanVid；音频点名 Emilia、AudioSet、VGGSound（部分资料另列 SoundNet）；自有侧为「授权的电影、电视剧、短视频与网络微短剧（licensed movies, TV series, short videos, web series）」——与昆仑万维短剧业务（短剧场景月活8000万）形成数据闭环。合成数据用于三处缺口：多语种画面文字生成、多语种语音合成（TTS）、以及 inpainting/editing 的配对数据。各来源占比未披露。

### [Sora 2](../models/Sora_2.md) ⚠️

仅有一句极高层级的定性描述，System Card 原文：「Sora 2 was trained on diverse datasets, including information that is publicly available on the internet, information that we partner with third parties to access, and information that our users or human trainers and researchers provide or generate.」即三类来源：(1) 互联网公开可得数据（网络爬取）；(2) 通过第三方合作/授权获取的数据；(3) 用户、人类训练师与研究员提供或生成的数据。未给出任何来源的占比、具体数据集名称、爬取范围、合作方名单。合成数据是否使用未明确说明（「generate」一词可能暗示包含人类训练师生成内容，但不等于模型合成数据）。[不确定]

### [Step-Video-T2V](../models/Step-Video-T2V.md) ⚠️

报告未说明来源构成。仅表述为「将原始视频（raw videos）经完整 pipeline 转化为适合预训练的高质量视频-文本对」，未区分自有素材库、公开数据集、网络爬取、授权采购或合成数据的比例；也未列举任何具体数据源名称。从 pipeline 设计可反推的间接线索：(1) 明确保留并使用了视频的「原始标题（Original Title）」作为一路 caption 来源，说明大量数据来自带有标题元数据的网络视频平台内容；(2) 使用 EfficientNet 水印分类器与 PaddleOCR 字幕检测，说明原始池中含大量带台标/字幕的二次传播内容，符合网络爬取特征；(3) 衍生模型 Step-Video-TI2V 的训练数据中超过 80% 为动漫风格视频，说明团队有大规模动漫内容来源。图文侧 3.8B 的来源同样未披露。[不确定]

### [UniTalking](../models/UniTalking.md) ⚠️

两个来源，二元构成，无第三方采购与网络自爬的显式说明：
【公开数据集：OpenHumanVid】复旦大学 CVPR 2025 发布的大规模人物中心视频数据集。其原始规模为 5,230 万片段 / 70.6K 小时，经其自身的画质与人物质量过滤后得到 1,320 万高质量片段；每条视频自带短/长/结构化三类文本 prompt、人体骨架序列与对应语音音轨——「自带语音音轨」正是 UniTalking 选择它作为底座的关键原因（多数视频数据集无音轨）。UniTalking 未说明从中取用了多少条、是否复用了其原生 caption 与骨架标注（从其重新用 Qwen3-VL/Qwen3-Omni 生成 caption 的做法看，原生 caption 应未被复用）。[不确定]
【内部数据：华为大规模内部采集】论文仅以「a large-scale, internal collection」一笔带过，未说明采集渠道（自爬/授权/自制）、地域、规模、时间跨度或内容类型。[不确定]
【两者配比】完全未披露。230 万条中 OpenHumanVid 与内部数据各占多少，是本工作数据描述中最关键的空白之一。[不确定]
【合成数据】使用，但仅限音频侧：用 IndexTTS2 零样本 TTS 为每条视频合成 3 条参考音色音频（详见 synthetic_data_synthesis 字段）。视频侧无任何合成内容。
【内部 TTS 数据】阶段一音频预训练使用的独立数据源，与上述 230 万音视频数据无关，来源与规模均未披露。[不确定]

### [UniVerse-1](../models/UniVerse-1.md)

三类来源，自采与公开数据集大致各占一半：
【自采网络数据（约 4,261 小时，即 1,187 + 3,074）】
- YouTube 为主要来源，内容类型明确列举为：音乐综艺节目（music variety shows）、古典音乐演奏（classical music performances）、烹饪教程（cooking tutorials）、公开演讲（public speeches）、访谈（interviews）、vlog、工具使用演示（tool usage demonstrations）；
- 电影片段（cinematic movie clips）；
- Pexels 高质量素材库视频（stock footage）。
【公开数据集（3,422 小时）】VGGSound 与 AudioSet，论文明确说明引入目的是「进一步强化音频模态」（bolster the audio modality），即补充音效/环境音的声学多样性，而非提升视觉质量——这也是后续对它们单独施加 LQLS 低质量损失策略的原因。
【授权采购】未提及任何付费授权采购数据。
【合成数据】未使用任何模型合成的视频或音频训练数据；唯一「合成」成分是全部标注由 QWen2.5-Omni 与 Whisper 自动生成（属合成标注而非合成内容）。
【来源选择的意图】所列内容类型高度贴合模型的三大目标能力：音乐综艺/古典演奏 → 乐器声生成；演讲/访谈/vlog → 语音与唇同步；烹饪/工具演示/Pexels → 环境音与 foley。

### [Unison](../models/Unison.md) ⚠️

全部为公开开源数据集聚合，仅音频侧含少量自有数据，无网络爬取、无授权采购、无合成数据。这是 Unison 与工业界工作最显著的差异——它是一个「站在已有公开数据集肩上」的学术项目。
【音视频联合训练语料（约 200 万 clips / 3,000+ 小时），五个开源数据集聚合】
1. OpenHumanVid —— 人物中心的大规模高质量视频数据集，是本条目「人物中心」定位的主要数据支撑，提供带 caption 的人物活动视频；
2. HDTF（High-Definition Talking Face）—— 高清说话人头视频，主要为演讲/新闻类正面人脸，贡献高质量唇同步样本；
3. VFHQ（Video Face High-Quality）—— 高保真人脸视频，源自访谈场景，贡献人脸清晰度与身份多样性；
4. CelebV-Text —— 带文本描述的野外人脸视频数据集，贡献人脸属性与动作的文本标注；
5. VGGSound —— 视听事件数据集（约 310 类声音事件），是五者中唯一的非人脸数据源，贡献环境音/音效与其视觉对应关系。
可以看出前四个数据源全部是人脸/人物中心数据集，VGGSound 是唯一的通用音效补强——这一构成决定了 Unison 的能力偏向：强于说话人与人物动作，环境音的视觉多样性依赖单一数据源。
【纯音频训练语料（5,000 万段 / 130,000+ 小时），按声音类型分工采集】
- 音效（sound effects）：YouTube-8M、AudioSet、WavCaps 三个数据集；
- 音乐（music）：VidMuse；
- 歌唱（singing）：主要取自 YuE collection（开源歌唱/音乐生成基座的语料）；
- 语音（speech）：包含「内部语音数据」（internal speech data），论文明确说明用于「进一步丰富训练语料的多样性与覆盖度」。这是全部语料中唯一的非公开来源，规模、语种、采集方式、授权状态均未披露。[不确定]
【授权采购】未提及任何付费授权采购。
【合成数据】未使用任何合成视频或合成音频作为训练数据。
【自采爬取】未提及自建爬虫或网络抓取——所有网络来源数据均以既有开源数据集为中介获取。
【设计意图】音频侧四类声音（语音/音效/音乐/歌唱）分别配备专用数据源，这一「按声音类型分源采集」的做法直接服务于其语音-音效双流解耦架构：模型需要在两条流上分别习得高质量先验，因此语料也按类型分置。

### [Veo 3 / Veo 3.1](../models/Veo_3_Veo_31.md) ⚠️

官方仅说明训练数据由「音频、视频、图像」三类构成（Model Card: 「Veo 3 was trained on audio, video, and image data」），未给出各来源占比。可确认的来源线索：(1) YouTube 视频——2025 年 6 月 Reuters/CNBC 报道并经 YouTube 证实，Google 使用 YouTube 语料库的一个子集（非全量）训练 Gemini 与 Veo 3，法律依据是 YouTube 服务条款中「全球、非独占、免版税」许可，多数创作者对此不知情；(2) 合成数据——官方确认生成合成 caption（synthetic captions）以提升概念多样性，但未说明是否使用合成视频；(3) 授权/采购与公开数据集的具体构成与比例[不确定]。整体来源构成比例[不确定]。

### [Vidu S1](../models/Vidu_S1.md)

明确为两大类原始视频来源（均未说明获取渠道是自有、采购还是爬取）：
(1) 直播与口播/说话人头部视频（livestream or talking-head videos）——主要用于学习面部表情、身体动作、唇形同步等细粒度特征；
(2) 影视剧高质量素材（high-quality footage from films and television dramas）——用于提升模型在不同镜头角度、场景与视觉风格上的泛化性与一致性。
未提及使用公开数据集、授权采购或合成数据。评测端使用了公开基准 HDTF 作为参考。

### [主流视频预训练数据集合并调研：Panda-70M、InternVid、Koala…](../models/pretraining_datasets.md) ⚠️

**七者全部为公开来源二次加工，无一家使用自有版权库或采购授权数据**，且存在严重的同源套娃（HD-VILA-100M 是至少三者的共同祖先）：
- **Panda-70M**：100% YouTube，全部来自 **HD-VILA-100M**（微软，330万YouTube视频，720p，跨15个热门品类均衡采样），取其中380万条长视频重新切分。
- **Koala-36M**：论文原文「we start from the same raw data with Panda-70M」，即**同样是 HD-VILA-100M**，LICENSE 中指向 HD-VILA-100M 许可可佐证。**不是快手自有短视频**，这一点常被误解。
- **InternVid**：独立爬取 YouTube。两路：核心集约200万条来自16个品类的优质频道；另约510万条通过**约6,100条动作/活动检索词**召回，词表 = Kinetics/Something-Something/UCF101 的 **1,103** 个动作标签 + LLM 从视觉定位语料中抽取并人工校验的 **5,001** 个动作，并参照美国时间使用调查（ATUS 2017–2022）。爬取时排除截至2023年4月已存在于公开数据集中的视频以避免重叠。
- **MiraData**：四个平台 + 一个回收数据集。YouTube **156个人工挑选的频道**约6.8万条720p视频（切后约3.4万视频→约17.3万clips）；**HD-VILA-100M 回收**（约1亿clips输入，仅**19.5万条**幸存，作者以此极低留存率论证自家筛选之严）；Videvo约6.3万、Pixabay约4.3万、Pexels约31.8万。（论文脚注2把 Videvo 与 Pixabay 的网址写反了。）
- **OpenVid-1M**：从四个已有数据集**二次筛选**——**Panda-50M/70M（主力）、ChronoMagic、CelebvHQ、Open-Sora-Plan（即 Mixkit/Pexels/Pixabay 系）**。CelebvHQ 原本无caption，由本项目补标。各源精确条数论文未以表格形式给出。[不确定]
- **UltraVideo**：**仅 YouTube 4K/8K 视频池**（原文「the sole source」）。两条召回路径：(a) 从 **Koala-36M** 中按分辨率>4K、帧率>25FPS、时长>30s 复筛，再用**播放量/点赞/评论等用户行为元信号**剔除「用户不感兴趣」的视频，再按标题描述与预设主题的相似度做各类目均匀采样并去重；(b) 用 LLM 从108个主题生成检索词后**人工搜索**最新4K/8K视频。合计5,000条原始视频（时长1分钟至2小时），再经**二次人工复检**剔除低质/模糊/水印/抖动。
- **LVD-2M**：从四个已有语料共 **220M clips** 中筛选——**HD-VG-130M（1.3亿）、Panda-70M（7000万）、InternVid-38M（3800万）、WebVid-10M（1000万）**。选源逻辑写得很清楚：YouTube 系动态足但需切镜过滤（「InternVid 中仅15%的clip超过10秒，而这些长视频中约52.5%含镜头切换」），素材站系（WebVid）几乎无切镜但「近一半不够动态」。**注意：不含 HD-VILA-100M（仅在相关工作对比表中出现）、不含 Vidal-10M、不含 Ego4D。** 按发布文件名推断成分：YouTube系约60万（Panda-70M+InternVid混在一个文件）、HDVG约30万、**WebVid约120万（占比近六成）**——论文正文未给出该拆分。

## 数据合规与溯源（授权数据占比、rights-cleared数据集、C2PA等）

`provenance_licensing` · 详细程度: brief

### [Allegro](../models/Allegro.md) ⚠️

论文未设置数据合规与溯源章节，未披露授权数据占比、rights-cleared 数据集比例、C2PA / 内容凭证、水印溯源或版权审查流程。所依赖的公开数据集（WebVid、Panda-70M、HD-VILA 等）本身多为学术研究许可、来源于网络视频平台，其商用合规性存在争议但论文未讨论。模型权重以 Apache 2.0 开源，但训练数据不开放、不提供来源清单。[不确定]

### [Apollo](../models/Apollo.md) ⚠️

论文完全未涉及数据合规与溯源议题：未给出授权数据占比，未声明 rights-cleared 数据集，未提及 C2PA 或任何输出侧水印/溯源标记，未讨论版权、肖像权或数据法律状态，也无模型卡级的使用条款。数据过滤清单中出现的「safety」一词是唯一与合规沾边的表述，但无任何展开。作为闭源工业模型，其合规工作可能存在于内部流程但未对外披露。[不确定]

### [CineDance / CineDance-1M](../models/CineDance.md) ⚠️

合规披露较为薄弱，是该工作相对明显的短板：
【发布许可】数据集在 HuggingFace 以 CC-BY-NC-SA-4.0 发布，限定非商业性研究与教育用途，并要求相同方式共享。
【访问控制】采用 gated access 门控机制，需 HuggingFace 账号 token 并经作者人工审核批准方可下载，属于对下游滥用的一层准入约束。
【上游版权】素材主体来自网络采集的影视内容与既有公开数据集（MiraData / LVD-2M / Koala-36M），论文未讨论上游影视素材的版权授权状态、rights-cleared 比例，也未提及 C2PA 等内容来源认证标准。
【风险免责】数据集卡明确声明「自动与人工策展无法保证移除每一条低质、敏感或其他不良样本」，将合规责任部分转移给使用者，要求使用者按自身场景补充质检。
【授权数据占比 / 采购数据】无相关披露，推断为零。[不确定]

### [CogVideoX](../models/CogVideoX.md) ⚠️

[不确定] 论文与开源仓库均未讨论训练数据的版权授权比例、rights-cleared 数据集、内容溯源标准（C2PA）或生成内容水印。仅有的相关信息是间接的：模型权重以 Apache 2.0（2B）与自有开源协议（5B 系列，允许商用）发布；清洗阶段刻意剔除「带明显字幕、水印」的视频（高质量微调阶段明确「effectively removed generated subtitles and watermarks」），但该动作的动机是画质而非版权合规。图像侧使用的 LAION-5B / COYO-700M 本身为公开的 URL-文本对数据集，其版权状态属公开争议范畴，论文未作说明。

### [Cosmos-Predict2.5](../models/Cosmos-Predict25.md) ⚠️

[不确定]。论文完全未涉及数据授权、版权与溯源议题：没有 rights-cleared 比例、没有采购/授权来源说明、没有 C2PA 或任何内容溯源与水印标识方案。仅笼统提到来源包含「proprietary datasets and open internet platforms」。发布侧的许可是清晰的（代码 Apache 2.0、模型 NVIDIA Open Model License），但这不构成对训练数据来源合法性的任何声明。NVIDIA Cosmos 平台层面另有 Cosmos-Guardrail 安全护栏（推理侧内容拦截），本篇论文亦未展开描述。

### [Data-Juicer 2.0](../models/Data-Juicer.md) ⚠️

【工具自身合规】Apache 2.0 许可，商用友好。公开的 T2V 数据池同样标注 Apache 2.0，但实际发布的是筛选后的样本元数据/索引，原始视频素材仍受 InternVid、Panda-70M、MSR-VTT 各自协议约束——这是学术视频数据集规避版权再分发的通行做法。
【面向合规的算子能力】DJ 提供若干可用于隐私与合规处理的算子：video_face_blur_mapper（检测并模糊视频中的人脸，直接服务于人脸隐私脱敏）、video_nsfw_filter（不良内容过滤）、video_watermark_filter 与 video_remove_watermark_mapper（水印检测与去除，间接关联版权标识处理）、文本侧另有敏感信息脱敏类算子。这使 DJ 成为少数在数据处理框架层面内置隐私脱敏算子的开源系统。
[不确定] 论文与文档未提及 C2PA 等内容溯源/来源认证标准的支持，未提供数据授权状态追踪、rights-cleared 数据集标记、许可证兼容性检查等治理型功能，也未披露阿里内部使用时的授权数据占比。整体上 DJ 提供的是「合规处理的工具」，而非「合规治理的框架」。

### [Foley-Omni](../models/Foley-Omni.md) ⚠️

披露非常薄弱。论文仅在 Ethics/Limitations 类表述中笼统声明数据「collected under appropriate usage agreements」（在适当的使用协议下收集），并承认模型存在被用于制作 deepfake 的潜在滥用风险。
间接可见的合规意识：V2ST-Bench 发布策略上明确因版权可再分发性限制（redistributable content constraints）而不直接分发原始视频文件，改为提供 URL 与 metadata，由使用者自行下载——这是学术音视频数据集的常见规避做法（同 VGGSound、AudioSet）。
模型权重侧声明 MIT 许可，并明确标注重分发了 Wan2.2-TI2V-5B 与 MMAudio 的组件、指引用户查阅上游原始许可，属于较规范的上游合规声明。
[不确定] 未披露授权数据占比、未提及 rights-cleared 数据集采购、未提及 C2PA 或任何内容溯源/水印机制、未说明 internal 数据的具体获取渠道与授权形式、公开数据集（如 Freesound、AudioSet、VGGSound）各自许可证的兼容性也未讨论。

### [Goku](../models/Goku.md) ⚠️

[不确定]。论文完全未讨论数据版权、授权比例、rights-cleared 数据集、C2PA/内容来源标注、水印溯源等合规议题，也未提及数据卡（datasheet）或使用条款。可确认的仅有：公开部分依赖 LAION、Panda-70M、InternVid、OpenVid-1M、Pexels 等既有学术/免版税数据集（其中 Pexels 为免版税素材站，LAION 为 CC 抓取的图文对索引，本身已知存在版权争议）；内部 2500 万视频片段与 6000 万图像的授权状态、来源合规性均未披露。这在 2025 年初的中国厂商论文中较为典型，与 Movie Gen / Veo 等强调授权与 C2PA 的做法形成对比。

### [HunyuanVideo-Foley](../models/HunyuanVideo-Foley.md) ⚠️

论文与模型卡均未讨论训练数据的合规与溯源问题：未给出授权数据占比，未声明 rights-cleared 数据集，未提及 C2PA 或任何输出侧的音频水印/溯源标记，未讨论音乐版权风险（尽管 pipeline 中明确包含音乐检测环节，说明数据中确实存在音乐内容），也未讨论人声/说话人隐私。
【可观察到的合规姿态在模型侧而非数据侧】采用 tencent-hunyuan-community 社区许可而非 Apache-2.0 等标准开源协议，并在 HuggingFace 上设置 extra_gated_eu_disallowed: true 明确排除欧盟地区用户——后者通常被解读为规避欧盟 AI Act 对通用目的 AI 模型的训练数据摘要披露义务与合规审查。这一设置本身是关于数据合规不确定性的一个间接信号：若训练数据来源完全清洁，通常无需做地域排除。
【风险点】音效生成模型的版权风险总体低于视频/音乐生成（Foley 音效多为通用声学事件，独创性表达成分低），但 pipeline 中的音乐检测环节表明原始数据含音乐，若未彻底剔除则存在音乐版权暴露面。论文对此零说明。[不确定]

### [HunyuanVideo](../models/HunyuanVideo.md) ⚠️

未披露。两份技术报告均未涉及授权数据占比、rights-cleared 数据集清单、版权处理策略、C2PA 等溯源标准。原版仅在过滤环节提到用 YOLOX-like 视觉模型剔除「水印、边框、logo 以及某些敏感信息（sensitive information）」，这更接近画面清洁而非版权合规。开源侧的合规约束体现在模型许可证（腾讯混元社区许可协议限制欧盟等地区使用、限制月活超1亿的商用），而非训练数据溯源。[不确定]

### [InstructAV2AV](../models/InstructAV2AV.md) ⚠️

[不确定] 合规与溯源披露薄弱，且存在明显的开放-合规张力。
【已知的正面做法】数据集以 MIT 许可发布，数据卡明确提示使用者需自行核验底层媒体内容适用的权利（users must verify compliance with rights applicable to underlying media content）——即作者把版权责任转移给了下游使用者。代码侧 Apache-2.0，并明确声明依赖 Wan-AI/Wan2.2-TI2V-5B 与 hkchengrex/MMAudio 等上游组件。
【存在的合规隐患】
  · 直接分发媒体本体：与 VGGSound、AudioSet、Foley-Omni V2ST-Bench 等采取「仅发 URL + metadata 规避再分发风险」的学术惯例不同，InsAVE-80K 直接打包分发 139 GB 视频/音频文件。其素材来源包含 YouTube 爬取内容与 Condensed Movies、MovieBench、Short-Films-20K 等电影/短片数据集，这些底层素材本身多受版权保护，直接再分发的法律风险显著高于 URL 分发。虽然 target 侧是合成产物，但 source 侧仍是原始素材。
  · 身份与声纹问题：数据集含 clone_id、clone_voice、clone_id_voice 三个子集，明确涉及人物身份克隆与音色克隆。使用 ElevenLabs 合成语音、TalkNet 定位说话人，实质是在真实人物的面孔与声纹上做替换与再合成。论文未提及任何肖像权/声音权授权、说话人知情同意、或去标识化措施。
【缺失项】未披露授权数据占比、未提及 rights-cleared 数据集、未提及 C2PA 或任何内容凭证/水印/溯源机制、未对生成内容做可检测标记。考虑到该模型的直接能力就是「换人、改口型、改台词」（即 deepfake 的技术定义），缺乏水印与溯源机制是该工作在合规维度上最值得关注的空白，论文的局限性讨论也仅聚焦于物理真实感、光照一致性、3D 空间一致性等技术缺陷，未涉及滥用风险。

### [2026 其他音视频联合生成](../models/JAVG_2026_misc.md) ⚠️

[不确定]——七项工作全部未讨论数据授权比例、rights-cleared 数据集、C2PA 水印溯源、版权合规审查流程。可推断的间接事实：
(1) 依赖公开学术数据集的工作（OmniCustom 全量依赖 SpeakerVid-5M；StreamChar 依赖 SpeakerVid-5M/TalkVid/OpenHumanVid/Emilia；Baton 与 CCL 部分依赖 OpenHumanVid/AudioCaps/WavCaps/VGGSound；NAVA 依赖 Koala-36M）实际上把合规责任外包给了上游数据集——而这些上游多基于 YouTube 抓取，本身带有来源争议。
(2) 明确含高风险来源的：NAVA 的「raw movie/TV footage」（影视原片，版权敏感度最高）、CCL 的「short dramas and films」（短剧与电影）、Baton 的「videos collected from the internet」。三者均无任何版权说明。
(3) 隐私侧：OmniCustom 与 StreamChar 属人脸+音色定制任务，直接涉及生物特征（人脸身份 + 声纹音色）。OmniCustom 评测集刻意使用「30 persons who were not included in training data」以验证零样本泛化，但未提及肖像权/声音权授权；ALIVE 的 character-driven pipeline 大量使用 ArcFace 人脸嵌入做身份匹配，同样无隐私声明。这是本批工作在数据合规上的共同空白。
(4) 许可层面唯一明确的是 NAVA：源码 Apache 2.0，但仓库明确声明「模型权重、预训练骨干、tokenizer、audio VAE、说话人编码器、prompt 改写模型可能受各自原始提供方的不同许可约束」——即许可覆盖代码而非数据。

### [音视频联合生成基线合集](../models/JavisDiT_baselines.md) ⚠️

整体披露稀薄，但 MM-Diffusion 与 JavisDiT 两家有明确动作，是本合集中少见的亮点：
【MM-Diffusion】(1) 选用 AIST++ 的一个显式理由是其 60 首舞曲为「copyright-cleared songs」（版权已清理歌曲），即在数据选型阶段就考虑了音乐版权；(2) 自建 Landscape 从 YouTube 爬取，未讨论视频版权，但数据集本身以研究用途公开发布；(3) 代码与权重以 MIT 许可发布。
【JavisDiT / JavisDiT++】版权意识最为明确：(1) 仓库明言「因版权问题无法发布 YouTube 原始视频」，只提供 33 万条 video ID 供用户自行下载——这是学术界规避视频版权的标准做法；(2) JavisBench 中来自 YouTube 的内容经过「严格的人工法律与伦理审核（strict manual legal and ethical verification）」；(3) 强调全部训练数据为公开数据集（public training entries），无采购、无内部私有数据，合规链条相对清晰。
【AV-DiT】未讨论数据授权与合规[不确定]；所用两个数据集均为已发布的学术数据集。
【Harmony】未讨论授权比例、rights-cleared 数据集、C2PA 等任何溯源机制；自采集的 200 万条片段的授权状况完全未披露[不确定]。论文仅笼统称环境音数据来自「public sources」。
【UniAVGen】完全未讨论数据合规、授权、隐私（注意其内部数据为「真人（real human）」音视频，涉及人脸与声纹，隐私敏感度高，但论文无任何相关表述）[不确定]。
共性缺口：五者均未提及 C2PA、内容水印、合成内容标识、数据主体删除请求等现代溯源机制。

### [可灵 3.0 / 可灵视频 3.0 Omni](../models/Kling_30_Omni.md) ⚠️

[不确定] 官方未公布授权数据占比、rights-cleared 数据集清单或数据溯源机制。产品侧的合规措施主要在输出端而非训练端：生成内容按中国《生成式AI服务管理暂行办法》与AI生成合成内容标识办法添加显式/隐式AI标识与水印（免费档强制水印、付费档可去水印）。[不确定：是否接入 C2PA Content Credentials——检索未见可灵官方声明支持C2PA，与 OpenAI/Adobe/Google 已公开支持形成对比]。数据侧仅在清洗环节提到内容安全/NSFW过滤，未提版权过滤细节。Kling-Foley 的开源评测集标注规范中明确要求“排除含人声的受版权保护背景音乐”，可视为团队版权意识的一个侧证。

### [LTX-2](../models/LTX-2.md) ⚠️

Lightricks 在同类模型中属于合规叙事最强的一方，但缺少定量披露。官方与媒体口径为「训练数据全部来自授权来源（Getty Images 与 Shutterstock），面向商用无版权顾虑」，Shutterstock 的「research license」被称为降低开源模型训练数据获取门槛的行业首例。但需注意：(1) LTX-Video 论文原文是「公开可得数据 + 授权素材补充」，与「全部授权」的媒体口径存在差异，授权数据占比从未公布；(2) 技术报告未给出 rights-cleared 数据集清单；(3) 未提及 C2PA、水印或任何输出侧溯源机制；(4) 报告「Social Impact」一节仅定性承认模型会反映训练数据中的偏见，并把「真实性验证与可溯源性改进」列为未来工作。[不确定]

### [LongCat-Video](../models/LongCat-Video.md) ⚠️

[不确定]。技术报告完全未涉及数据合规与溯源议题：没有授权数据占比、没有 rights-cleared 数据集声明、没有 C2PA 或任何内容溯源/水印标识方案的说明，也没有版权许可来源的讨论。这与国内厂商技术报告的普遍做法一致（合规细节不写入论文）。需注意模型权重本身采用 MIT 许可开放商用，但这不构成对训练数据来源合法性的任何声明。

### [MOVA](../models/MOVA.md) ⚠️

论文完全未讨论数据合规与溯源议题：未给出授权数据占比，未声明 rights-cleared 数据集，未提及 C2PA 或任何输出侧水印/溯源标记，未讨论版权风险、肖像权或数据来源的法律状态。可间接判断的是：训练语料中包含 YouTube 抓取内容、中文剧集与电影片段，这些属于版权敏感来源，而所引用的公开数据集（ACAV-100M、VGGSound 等）本身多为“YouTube 链接集合”形式，其版权状态由原始数据集的条款约束。模型权重侧采用 Apache-2.0 开放商用许可，但训练数据的合规性未作任何说明——这是 MOVA 在“方法论透明度极高”的同时“合规透明度为零”的明显不对称。[不确定]

### [合并条目：① Mochi 1](../models/Mochi_MAGI_Motif.md) ⚠️

三者均无实质性的数据合规与溯源披露，这是本组三个模型共同的最大缺口：
① Mochi 1：数据来源刻意模糊化，恰恰成为发布时的争议焦点——VentureBeat 报道直接指出「训练数据集是 AI 创作工具最具争议的方面之一，已有证据表明许多工具未经许可、未付报酬地使用了大量网络上的人类创作成果，其中部分为受版权保护作品」，而 Jain 对此「was coy（含糊其辞）」。未披露授权数据占比、rights-cleared 数据集、C2PA 内容凭证或水印溯源。模型卡仅在下游层面提示「Genmo 视频模型会反映其训练数据中的偏见与成见」，建议机构在商用部署前追加安全协议。[不确定]
② MAGI-1：技术报告无数据合规章节，未讨论版权、授权比例、C2PA 或来源清单，仅在权重层面给 Apache 2.0。[不确定]
③ Motif-Video 2B：同样无合规章节，未披露授权占比、rights-cleared 数据或内容凭证。相对的间接合规动作是把 NSFW 与水印过滤前置到 sanitation 首关（水印/台标本身也是版权信号的代理），并在 caption 阶段用 VLM 复核 watermark 标签做二次剔除——可以理解为「以工程手段规避带权利标记的素材」，但论文并未把它表述为合规措施。[不确定]

### [Movie Gen](../models/Movie_Gen.md) ⚠️

[不确定] 论文完全未讨论数据授权占比、版权清理（rights-cleared）、内容溯源标准（C2PA/水印）等问题。仅在结论的「Safety considerations」中提到：模型为研究目的开发；模型可能学到模态内的偏见（如视频训练数据中的视觉偏见）；真实部署时会接入安全模型拒绝违反政策的输入prompt与生成结果。数据侧的合规流程、人脸/隐私授权、生成内容水印均未披露。

### [NVIDIA NeMo Curator](../models/NeMo_Curator.md) ⚠️

NeMo Curator 框架层面未提供数据溯源与授权管理能力：官方文档中没有 C2PA 内容凭证写入、没有版权指纹检测、没有 rights-cleared 数据集管理模块，也没有 provenance metadata 的标准化字段规范。框架只保证在分片（sharding）时保留原始 metadata 透传。
合规责任被完全交给使用者：Apache 2.0 许可下 NVIDIA 不对用户所处理数据的合法性做任何保证；Cosmos 模型侧的合规叙事（Guardrails、可信 AI 声明）与数据 curation pipeline 是分离的两套东西。Cosmos WFM 论文亦未披露其 2000 万小时视频的授权比例与来源合法性论证。这是该工具链相对于 Adobe Firefly / Lightricks（Shutterstock、Getty 授权）等强合规叙事方案的明显空白。[不确定]

### [OmniHuman 数据集 + OHBench](../models/OmniHuman.md) ⚠️

合规与溯源是本工作披露最薄弱的一环，且风险敞口高于普通视频数据集：
【论文层面：零合规陈述】全文无伦理声明（ethics statement）、无版权讨论、无肖像权/隐私讨论、无知情同意说明、无 C2PA 或任何内容溯源标记的提及、无深度伪造滥用防范表述。论文亦无专门的 Limitations 章节讨论上述问题。
【发布层面的规避手段】采用「只发标注、不发视频」的分发模式（每条样本仅给 YouTube URL + 起止时间戳），这是学术界规避视频版权风险的标准做法，把下载与使用的法律责任转移给使用者。此举在版权维度提供了一定程度的缓冲，但对肖像权与隐私维度无效——8 万个真实自然人的面部、134 点全身骨架、SMPL/MANO 三维身体与手部参数、语音转写文本、情绪标签均以结构化形式被完整发布，且这些标注本身即可作为深度伪造的高质量素材，其中相当部分被拍摄者从未同意用于生成式模型训练。
【许可空白】HuggingFace 数据集页与 GitHub 仓库均未见许可声明，使用范围（研究用途 / 商用）、再分发条件、退出机制（数据主体请求删除的途径）全部缺失。作为一份面向公开发布的百万级真人数据集，这一空白比方法层面的疏漏更值得关注。
【与上游对照】OpenHumanVid 为防滥用要求下载者提交身份信息并经审核批准；OmniHuman 则无任何准入控制。
【授权数据占比】0%（全部为网络爬取的公开可见内容，无授权采购部分）。[不确定]

### [Open-Sora 系列](../models/Open-Sora.md) ⚠️

两个项目对训练数据合规与溯源的处理都很薄弱，属于典型的学术/开源项目做法：
- 依赖第三方公开数据集本身的许可（Panda-70M 源自 HD-VILA-100M 的 YouTube 视频，Webvid-10M 源自 Shutterstock 水印素材，VIDAL 源自 YouTube Shorts），这些数据集的版权状态本身存在争议，项目方未做二次授权核查，也未披露授权数据占比。
- Open-Sora Plan 使用的 ShareGPT4Video 部分明确标注来自 Mixkit / Pexels / Pixabay 的 CC0 免版权素材，Open-Sora 1.2 亦明确使用 Pexels / Pixabay / Mixkit 免费许可素材，这是两者中唯一可称为 rights-cleared 的部分，但均未给出占比数字。
- 未使用任何采购授权数据，未建立 rights-cleared 数据集清单。
- 输出侧：均未实现 C2PA 元数据、不可见水印或生成内容检测工具，模型权重以 Apache 2.0 等宽松许可放出，下游使用无溯源约束。
- 值得注意的是 Webvid-10M 素材带有 Shutterstock 水印，Open-Sora 1.2 将其用于第一阶段低分辨率预训练（240p/360p），意味着水印信号进入了模型早期训练——这是社区已知的开源复现通病，项目文档未讨论其影响。[不确定]

### [Ovi](../models/Ovi.md) ⚠️

[不确定]。论文与仓库均未讨论数据授权比例、rights-cleared 数据集、C2PA/水印溯源、版权合规审查等任何内容，也无 Model Card 层面的数据来源声明。可确认的仅有：微调期使用的三个公开音频数据集（VGGSound、AudioSet、WavCaps）均为学术研究许可数据集，其中 AudioSet/VGGSound 基于 YouTube 视频，本身带有来源合规争议；模型权重以 Apache 2.0 发布，但许可覆盖的是权重而非训练数据。内部语料的授权状况完全未披露。

### [Script-a-Video](../models/Script-a-Video.md) ⚠️

论文完全未涉及数据合规与溯源议题：未给出授权数据占比，未声明 rights-cleared 数据集，未提及 C2PA 或任何输出侧水印/溯源标记，未讨论版权、肖像权或数据使用许可。
可识别的合规风险点：
1) caption 侧 500K 与生成侧数据均大量取自 film / television / TV drama 等影视内容，属版权高度敏感来源，论文未说明获取途径与授权状态；
2) Reference 流对人物做细粒度外观锚定（clothing、accessories、hairstyles 等属性），Identity Customization 模块支持从参考图注入真实人物身份，Event 流记录 verbatim 台词并绑定说话人 ID——这套组合在肖像权与深度伪造风险上比纯场景描述型 caption 更敏感，论文对此零讨论；
3) 所有资产均未公开发布，客观上降低了外部滥用风险，但也意味着无模型卡级的使用限制声明。
此字段在论文中属完全空白。[不确定]

### [Seedance 2.0 与 Seedance 1.5 pro](../models/Seedance_20_Seedance_15_pro.md) ⚠️

仅有原则性表述，无量化披露。Seedance 1.0 报告称数据获取「优先选择伦理与法律上合规的内容，来自多样化的公开与授权仓库」，并在过滤环节部署分类器剔除色情、暴力、儿童剥削、露骨裸露等内容以保证「伦理合规与数据集安全」。Seedance 2.0 报告在引言末尾声明「安全是我们工作的核心考量」，在模型迭代全生命周期实施了结构化的安全评估框架并持续评估与缓解潜在风险，以支持负责任、合规、符合伦理的开发。[不确定：授权数据占比、rights-cleared 数据集清单、是否采用 C2PA/水印溯源等均未披露]

### [SkyReels 系列](../models/SkyReels.md) ⚠️

两代论文均无定量合规披露。仅有定性表述：V4 称自有数据为「licensed（已授权）」的影视、短视频与网络剧内容，V2 称自采影视媒体与「艺术素材库」，均未给出授权数据占比、rights-cleared 数据集清单、版权方名单或采购方式。两篇论文均未提及 C2PA、不可见水印、输出溯源或内容来源标识机制，也未见 NSFW/隐私合规章节。公开数据集侧使用了 LAION、WebVid-10M 等已知存在版权争议的语料，论文未对此作说明。作为在中国境内运营的生成式服务，SkyReels 产品端应受《生成式人工智能服务管理暂行办法》与深度合成标识要求约束，但论文未提及备案与标识实现。[不确定]

### [Sora 2](../models/Sora_2.md)

训练侧溯源披露极弱，输出侧溯源披露较强，二者需严格区分。
【训练侧】未披露授权数据占比，未公布任何 rights-cleared 数据集清单，未说明授权采购方。唯一明确的合规声明是儿童安全相关：「responsibly sourcing datasets to exclude CSAM」（负责任地筛选数据源以排除儿童性虐待材料），并与美国失踪与受虐儿童中心（NCMEC）合作。版权方面：Sora 2 上线时采取「opt-out」（版权方需主动要求排除）策略，引发SpongeBob、South Park、Scooby-Doo 等IP大规模被生成的争议；2025年10月3日（上线仅3天后）Sam Altman 发博客改为「opt-in」并承诺给版权方更细粒度控制与收益分成。美国电影协会（MPA）公开施压，日本政府亦正式要求 OpenAI 避免侵权。关键点：opt-in 政策仅约束「生成」环节，OpenAI 从未澄清该政策是否回溯适用于「训练数据」，即已被训练进模型的受版权内容并未被移除。后续 OpenAI 与迪士尼（2025年12月，三年授权、覆盖200+迪士尼/漫威/皮克斯/星战角色）、与 Getty Images（2026年6月，Getty 与 Shutterstock 37亿美元合并后的多年期合作）达成授权协议，但这些均为「生成侧IP授权/展示合作」，未明确为 Sora 2 的训练语料授权。
【输出侧】所有一方产品资产带 C2PA 元数据（行业标准可验证来源）；从 sora.com 与 Sora App 下载的视频带可见移动水印；OpenAI 保有内部检测工具判定某视频/音频是否由其产品生成。OpenAI 自承「provenance 不存在单一解法」。

### [Step-Video-T2V](../models/Step-Video-T2V.md) ⚠️

未披露。技术报告全文未涉及授权数据占比、rights-cleared 数据集、版权清算流程、C2PA 等内容溯源标准，也未提及生成内容水印/标识。数据合规维度上唯一可见的痕迹是 pipeline 中的 NSFW 打分（LAION CLIP-based NSFW 检测器）与水印检测（用于剔除带水印素材，动机更接近画面清洁与规避明显版权标记，而非系统性版权合规）。模型侧采用 MIT 协议开源，不含数据溯源承诺。作为中国境内发布的产品，实际生产必然存在内容安全与合规审核，但训练数据侧的合规方法零披露。[不确定]

### [UniTalking](../models/UniTalking.md) ⚠️

论文完全未讨论数据合规与溯源议题：未给出授权数据占比，未声明 rights-cleared 数据集，未提及 C2PA 或任何输出侧水印/溯源标记，未讨论版权、肖像权或深度伪造风险。可间接判断的合规状况：
- OpenHumanVid 部分：该数据集的视频样本采集自「公开可得的数据集」（publicly available datasets），使用者需遵循原始许可；且为防止滥用，其下载需提交用户信息经审核批准。UniTalking 作为使用方未说明是否履行了该审批流程，也未说明是否遵循了上游许可条款；
- 华为内部数据部分：来源不明，无任何合规陈述；
- IndexTTS2 合成的参考音频：以真实视频中的原始人声作为音色参考做零样本克隆，实质上生成了大量真人音色的克隆语音用于训练——这在声纹权与人格权层面是一个论文完全未触及的敏感点；
- 模型能力本身（任意身份图像 + 任意音色 → 说话视频）属深度伪造高风险能力，而论文无任何滥用防范、使用限制或伦理声明。
整体是典型的「方法披露有、合规披露零」，且相较同类工作，其合成音色克隆环节带来的额外合规问题更为突出。[不确定]

### [UniVerse-1](../models/UniVerse-1.md) ⚠️

论文完全未讨论数据合规与溯源议题：未给出授权数据占比，未声明 rights-cleared 数据集，未提及 C2PA 或任何输出侧水印/溯源标记，未讨论版权风险或肖像权。可间接判断的风险点是：训练语料明确包含 YouTube 抓取内容与电影片段（cinematic movie clips），属版权敏感来源；Verse-Bench 评测集同样从 YouTube 与 Bilibili 采集，并使用了 2025 年 9 月的 TED Talks 素材。Pexels 部分属于相对宽松的免费素材许可。所引用的 VGGSound、AudioSet 本身为 YouTube 链接集合形式，版权状态受原数据集条款约束。模型权重与代码采用 Apache-2.0 完全开放商用许可，但训练数据合规性零说明——与 MOVA 呈现相同的「方法披露有、合规披露无」不对称。[不确定]

### [Unison](../models/Unison.md) ⚠️

论文完全未讨论数据合规、授权与溯源议题：未给出授权数据占比，未声明 rights-cleared 数据集，未提及 C2PA 或任何输出侧水印/溯源标记，未讨论版权、肖像权或深度伪造风险，未设置模型使用限制说明。
【可间接判断的合规状况——总体优于纯爬取型工作】
- 音视频语料全部来自已发表的学术开源数据集（OpenHumanVid、HDTF、VFHQ、CelebV-Text、VGGSound），这些数据集各自有明确的学术使用许可条款，通常限定为非商业研究用途。Unison 复用它们意味着其训练数据的合规责任在相当程度上转移给了上游数据集的许可框架，比 UniVerse-1（自采 YouTube + 电影片段）风险更低；
- 但需注意 VGGSound、AudioSet、YouTube-8M 本质上都是 YouTube 视频链接集合，其内容版权归原上传者，学术数据集条款不等同于版权清理；
- HDTF、VFHQ、CelebV-Text 均为真实人脸数据集（含名人与公众人物），Unison 在此之上训练的是「任意人物说话视频生成」能力，肖像权与深度伪造滥用风险客观存在，而论文对此零讨论——考虑到有字节跳动与中国电信两家产业方参与，这一缺失更值得注意；
- 「内部语音数据」的来源与授权状态完全不明，是合规链条上最不透明的一环。
【与许可的矛盾点】论文声明代码与模型将在接收后公开，但其训练语料多为限定学术用途的数据集，模型权重的可商用性存疑，论文未作任何说明。[不确定]

### [Veo 3 / Veo 3.1](../models/Veo_3_Veo_31.md) ⚠️

官方可确认的合规措施：训练视频按「various compliance and safety metrics」过滤；caption 侧过滤掉不安全描述与个人可识别信息（PII）；训练数据被分析以识别潜在有害数据并审查人群表征的公平性；输出端统一嵌入 SynthID 隐形水印，并配合生产环境过滤降低信息完整性风险。Google 生态在 2025-2026 年同时对 Imagen / Veo / Lyria 输出采用 SynthID，并逐步对齐 C2PA Content Credentials 元数据标准（Veo 3 输出是否默认携带 C2PA 清单需以产品线为准）[不确定]。未披露的关键项：授权数据占比、rights-cleared 数据集清单、版权方分成或退出（opt-out）机制。YouTube 数据使用引发创作者知情权与 IP 争议，属公开争议点。

### [Vidu S1](../models/Vidu_S1.md) ⚠️

[不确定]。报告未提及任何数据授权、版权合规、rights-cleared 数据集比例、C2PA/水印溯源等信息。仅在清洗环节提到内容安全过滤（NSFW 及其他不当内容剔除）与画面干净度过滤（去除含水印、字幕、贴片广告的片段），后者更多是画质动机而非版权动机。考虑到数据来源包含影视剧素材，版权来源披露缺失是明显空白。

### [主流视频预训练数据集合并调研：Panda-70M、InternVid、Koala…](../models/pretraining_datasets.md) ⚠️

**七者的合规基础都很脆弱，且质量参差**：
- **共同的根本问题**：绝大部分素材是 YouTube 视频，数据集方均不拥有版权，仅以「非商用研究」+「侵权可申请下架」两条软性条款自保。Panda-70M 与 MiraData 都写有明确的下架承诺（「We will remove the video samples from our dataset as long as you need it」）。UltraVideo 的许可证更进一步要求使用者自行遵守 YouTube ToS 与 GDPR/CCPA，并明确免责声明不保证素材权属——等于承认自己不拥有 YouTube 素材。
- **授权数据占比**：唯一可称为 rights-cleared 的成分是素材站 CC0/免费许可内容——MiraData 的 Videvo/Pixabay/Pexels 部分、OpenVid-1M 中来自 Open-Sora-Plan（Mixkit/Pexels/Pixabay）的部分、LVD-2M 中 WebVid 的 stock footage 部分（但 WebVid 本身带 Shutterstock 水印且已因版权问题下架，属高风险成分）。**没有任何一家披露 rights-cleared 数据的占比数字。**
- **不发视频本体作为规避手段**：Panda-70M / InternVid / Koala-36M / MiraData / LVD-2M 五者只发 URL+时间戳，把爬取的法律风险转嫁给使用者。InternVid 明确说明「遵循既有数据集协议只共享视频ID以符合YouTube政策」。代价是**链接腐烂（link rot）不可逆**——Panda-70M 仓库直接指导用户跳过 status=failed_to_download 的样本；LVD-2M 的下载脚本甚至内置多账号轮换（ACCOUNT_NUM）并警告 Google 账号可能被封。**无一家发布过 ID 存活率的实测数据。**
- **反向操作**：OpenVid-1M 与 UltraVideo 直接托管视频本体（12.4TB / 全量4K-8K clips），可复现性最好但法律暴露最大；UltraVideo 用「禁止二次分发原始视频、衍生品须继承同等条款」来收口。
- **隐私处理**：仅 **Panda-70M** 做了一处具体工作——用 NLTK 把 caption 中所有人名替换为「person」，并过滤含有害/暴力语言及毒品、仇恨言论的样本。其余六者均无人名脱敏、无人脸隐私处理。
- **C2PA / 水印 / 生成内容溯源**：七者**全部没有**，输出侧无任何溯源机制。
- **许可证自相矛盾**：MiraData 最严重——仓库标 GPL-3.0，README「License Agreement」段禁止一切商业用途，但同一 README 末句写「is supported for commercial usage」。可辩护的解读是：代码 GPL-3.0，数据仅供研究，视频版权仍属第三方。Koala-36M 的 HF 页面则**完全没有 license 标签也没有 dataset card**，仅 GitHub 仓库有非商用条款。

## 片段时长分布与切分策略

`duration_distribution` · 详细程度: brief

### [Allegro](../models/Allegro.md)

切分与时长约束是漏斗的第一道关卡：
· 原始视频先剔除时长 <2 秒、帧率 <23 FPS 的样本；
· 经 PySceneDetect 场景切分后，仅保留 2–16 秒的单场景片段（clip 级时长上下界）；
· 720p 预训练后期与微调阶段进一步收紧为 6–16 秒（Table 1：T2V Pre-train 720p 为 [2s,16s] 与 [6s,16s] 两档，Fine-tune 为 [6s,16s]）。
分布（Appendix A Fig.11）：预训练阶段在 2–6 秒 / 6–10 秒 / 10–16 秒三个桶上分布较为均衡；微调阶段因 6 秒下限而整体右移，长片段占比显著提高，以匹配 88 帧（约 6 秒 @15FPS）的生成目标。
最终统一转码为 H.264、固定 30 FPS。

### [Apollo](../models/Apollo.md) ⚠️

论文未披露任何片段时长信息：未给出训练 clip 的固定时长或时长分布，未给出帧率，未给出最短/最长时长阈值，也未说明是定长切分还是变长分桶。仅能从 VAE 规格间接约束：Video-VAE 输出 3 Hz 的时序 embedding（即每秒 3 个 latent 帧），Audio-VAE 输出 43 Hz 的 embedding；论文明确 Video-VAE「handles input videos with varying resolutions and frame rates」，说明输入侧帧率不统一，由 VAE 归一化到 3 Hz 表征。切分策略层面唯一可确认的是场景切分（scene splitting）保证每条样本只含单一场景，但切分后的窗口长度规则未说明。[不确定]

### [CineDance / CineDance-1M](../models/CineDance.md) ⚠️

时长分布是该数据集最核心的差异化维度，并配有专门的「防碎片化」策略：
【平均时长】92.8 秒/序列，是同类多镜头数据集的数倍（对比 MiraData 72.1s、LVD-2M 20.2s、SpeakerVid-5M 8.3s）。
【切分策略】不采用「镜头即样本」的传统切法，而是两级：先用 TransNetV2 切出 2589 万个原子镜头，再按电影理论规则将连续镜头「自底向上」重组为叙事序列（narrative sequence），序列而非单镜头才是最终样本单位。
【最短时长约束】从人工参考集实测得到叙事完整性的经验最小时长为 18.4 秒，据此把软阈值设为 20 秒，以阻止解析出无意义的过短片段。消融显示最终数据中仅 3.1% 的序列短于 20 秒。
【时长剪裁】MLLM 引导的时序截断会去掉长视频首尾的片头/片尾内容，截断长度公式为 t = max(5 分钟, 0.1L)，L 为原视频总时长。
【分布图】论文 Fig.5 给出时长与镜头数的联合分布直方图，但具体分位数（P50/P90、最长最短）未以文字列出。[不确定]

### [CogVideoX](../models/CogVideoX.md) ⚠️

· 训练 clip 平均约 6 秒，且全部为单镜头（single-shot）片段；未给出时长直方图 [分布细节不确定]。
· 明确采用「混合时长训练（mixed-duration training）」：论文批评固定帧数训练必须「丢弃短视频、截断长视频」，导致数据利用不充分，因此把不同时长（同时也是不同分辨率）的视频放进同一 batch，通过 Multi-Resolution Frame Pack（受 Patch'n Pack 启发）保证 batch 内形状一致，再用 3D RoPE 建模不同形状的位置关系。这相当于用 packing 取代传统分桶（论文明确指出 SDXL 式 bucket 方案「让数据与训练 pipeline 更复杂」）。
· 训练阶段的时长上限逐级放宽：stage1/stage2 最大 6 秒，stage3/stage4(FT) 最大 10 秒（对应序列长度 25k → 75k → 700k）。
· 3D VAE 侧为节省显存，采用两阶段训练：先在 17 帧视频上训练，再用 context parallel 在 161 帧视频上微调。
· 切分策略见 shot_segmentation：论文未描述显式的场景切分工具，而是用「Lack of Motion Connectivity（运动不连贯）」分类器把含拼接/转场的片段整体剔除。

### [Cosmos-Predict2.5](../models/Cosmos-Predict25.md) ⚠️

切分与时长策略披露明确：
(1) 长视频先经 shot-aware splitting 切为单镜头段；
(2) 时长门槛：「Very short clips under 5 seconds are discarded」——短于 5 秒的片段直接丢弃；
(3) 保留区间：剩余片段时长为 5 至 60 秒（ranging from 5 to 60 seconds），共 60 亿+ 条；
(4) caption 粒度：每条 clip 再切成 5 秒窗口逐窗口打标（这是相对 Cosmos-Predict1 的「finer content granularity」改进）；
(5) 训练消费粒度固定：全部阶段统一生成 93 帧、16 fps，约 5.8 秒（对应 WAN2.1 VAE 4×8×8 压缩后的 24 个 latent 帧）；
(6) 领域数据的时长是定长的：自动驾驶 clip 统一 20 秒；Human Dynamics 要求至少 5 秒。
(7) 时长是最终 sharding 的四个轴之一（length），支撑按时长采样。
未给出 5–60 秒区间内的具体分布直方图或平均时长。[不确定：时长分布的具体统计量]

### [Data-Juicer 2.0](../models/Data-Juicer.md) ⚠️

【切分算子——DJ 的强项之一】提供三种互补的视频切分算子，可按需组合：
  · video_split_by_scene_mapper —— 基于场景变化检测把视频切成镜头级 clip（底层为 PySceneDetect 系方案），是最贴合视频生成训练数据构建的切分方式。
  · video_split_by_duration_mapper —— 按固定时长切分为等长片段，适合快速构建定长训练样本。
  · video_split_by_key_frame_mapper —— 按关键帧（I帧）边界切分，切点与编码结构对齐，解码开销低。
  另有 video_clip_reassembly_mapper 用于把重叠片段的处理结果重新组装回长视频（面向具身智能手部动作场景）。
【时长过滤】video_duration_filter 支持设定时长上下界区间，剔除过短（信息量不足）与过长（切分不干净）的片段。
[不确定] 官方 T2V 案例未披露最终数据池的片段时长分布直方图或平均时长；InternVid/Panda-70M/MSR-VTT 本身已是预切好的短片段数据集（Panda-70M 平均约8.5秒、MSR-VTT 约10-30秒、InternVid 平均约11秒），案例中未额外做二次切分，因此切分算子在该案例中未被实际启用。DJ 也未提供官方推荐的时长分桶策略。

### [Foley-Omni](../models/Foley-Omni.md) ⚠️

披露有限。
【评测集】V2ST-Bench 片段时长明确为 5–10 秒。
【训练集】论文未给出训练片段的时长分布直方图或平均时长。可从数据规模反推粗略量级：约 4.9k 小时对应约 2.7M 样本，平均每样本约 6.5 秒，与 V2ST-Bench 的 5–10 秒区间一致，说明整体走的是短片段（约10秒以内）路线——这与其对标的 MMAudio、VGGSound（10秒片段）等 V2A 主流设定吻合。
【切分策略】[不确定] 论文未描述如何将长视频切分为训练片段，未提及镜头边界检测、固定窗口滑窗或其他切分方法。过滤阶段的指标（motion score、Synchformer sync score、ImageBind score）均是片段级计算，暗示切分发生在过滤之前，但具体切分算法未交代。

### [Goku](../models/Goku.md)

【切分策略】原始长视频先切成语义连贯的短片段（clip），核心约束有二：
(1) 时长下限——预处理阶段直接丢弃时长 < 4 秒的片段（duration ≥ 4s）。
(2) 时长上限——单个片段最长截断为 10 秒（maximum clip length 10 seconds），超长的连续镜头被强制截断。
因此训练片段时长严格落在 [4s, 10s] 区间，这也决定了 Goku 的生成能力集中在 10 秒以内短视频。
【其他预处理约束】码率 ≥ 500 kbps；帧率 ≥ 24 FPS（电影标准）或 23.976 FPS（NTSC 标准），低帧率素材被剔除。
【未披露】区间内的具体时长直方图、平均片段时长、各时长桶的采样权重。

### [HunyuanVideo-Foley](../models/HunyuanVideo-Foley.md) ⚠️

【严格定长 8 秒】这是本工作时长策略最鲜明的特征——所有训练片段统一为 8 秒定长（"chunk them into 8-second intervals"），无变长设计、无长度分桶、无短片段与长片段的课程安排。
【两级切分逻辑】先用场景检测算法（scene detection）对原始长视频做镜头切分，得到语义完整的镜头段；再把镜头段切成 8 秒定长块。即「先按内容切、再按时长切」，与 UniVerse-1「场景切分后剔除 <5 秒、保留变长」的思路不同：HunyuanVideo-Foley 不剔除而是强制规整。
【8 秒的选择依据】论文未解释为何选 8 秒。可推断的原因有三：(1) 与主流 V2A 评测基准（VGGSound 片段 10 秒、MovieGen-Audio-Bench）的时长量级接近，便于对齐评测；(2) 8 秒 × 50 Hz latent 帧率 = 400 个音频 latent token，是一个训练友好的序列长度；(3) 8 秒足以覆盖绝大多数 Foley 事件的完整声学包络（脚步、开关门、碰撞等），同时避免长序列的同步漂移。[不确定]
【切分产生的边界问题未讨论】8 秒硬切会把跨越边界的声学事件截断（如一段持续 12 秒的雨声被切成 8 秒 + 4 秒残段），论文未说明残段如何处理——是丢弃、补零填充还是与下一段拼接，均无描述。[不确定]
【推理时长】开源实现支持任意时长视频的音效生成，但训练仅见过 8 秒样本，长视频的处理方式（分段生成后拼接？）未在论文中说明。

### [HunyuanVideo](../models/HunyuanVideo.md)

【原版】未给出时长直方图。切分策略明确：用 PySceneDetect 将原始视频切为单镜头（single-shot）片段，再用 OpenCV Laplacian 算子在片段内选取清晰帧作为起始帧。训练时长以帧数体现——从 65 帧（256×256×65）到 129 帧（720×1280×129），并支持 1–129 帧的多帧数分桶。
【1.5】明确得多：所有训练片段统一切分为 2–10 秒；预训练阶段 III–V 为 16fps、2–10s，阶段 VI 起升到 24fps、2–10s，CT/SFT 阶段维持 24fps、2–10s。即 1.5 走的是「短clip为主、以帧率而非时长做课程递进」的路线，未训练超过10秒的长片段。

### [InstructAV2AV](../models/InstructAV2AV.md) ⚠️

分布极度集中——全部片段统一为 5 秒定长，无分布可言。这是合成数据集相对真实采集数据集的一个典型特征：由于 target 由生成模型合成，而底座 Wan2.2-5B / Ovi 的生成窗口固定，输出时长必然被钉死在模型的原生窗口上（5秒 @ 24FPS = 120帧）。
【切分策略】两级：
  1. 镜头级切分：用 PySceneDetect（Castellano, 2024）把爬取/收集的原始长视频切为单镜头片段（single-shot clips）。这一步保证每个片段内无镜头切换，是后续所有处理（点跟踪、说话人定位、mask 传播、编辑合成）能够成立的前提——跨镜头片段会让 Grounded-SAM-2 的实例 mask 无法连续传播。
  2. 定长截取：从单镜头片段中取 5 秒窗口。[不确定] 论文未说明具体截取规则——是取首 5 秒、取中间 5 秒、还是滑窗切多个 5 秒段（后者会引入片段间高度重叠，是潜在的隐性重复来源）；也未说明短于 5 秒的单镜头片段如何处理（丢弃或补齐）。
【与调研中其他工作的对比】Foley-Omni 为 5–10 秒、VGGSound 为固定 10 秒，本工作 5 秒是同类中偏短的，直接限制了可编辑内容的复杂度——5秒内难以承载多轮对白或复杂事件序列，也解释了为何四类任务都是「单一目标的局部编辑」而非叙事级改写。

### [2026 其他音视频联合生成](../models/JAVG_2026_misc.md) ⚠️

各工作的时长策略高度分化，反映其任务定位：
【定长短片｜OmniCustom】严格定长设计，且与「参考-训练配对」耦合：先过滤掉「videos shorter than 10 seconds」，再从每段 ≥10 秒的片段中取「the first 4 seconds as the reference audio」，「the last 5 seconds ... designated as both the training audio and video clips」——即一条 10 秒素材同时产出 4 秒参考音频与 5 秒训练样本，二者音色相同但语音内容不同（「each reference-training pair shares the same timbre but contains distinct speech content」）。这是一个非常干净的「同音色异内容」配对构造法。
【短片段+长源｜ALIVE】Character-driven 数据从「long videos (10–30 minutes)」中抽取 N 个「3–10 seconds」的片段；生成输出为 5 至 10 秒。身份锚点则取「1.5-second sub-clip with maximum sync score」——即用 1.5 秒的同步分最高子片段作为身份代表帧源。训练数据整体的时长分布未披露[不确定]。
【平均 7 秒｜NAVA】明确「The average video duration is about 7 seconds」，是七项中唯一给出平均时长的工作。推理侧默认 37 帧 @24fps（约 1.5 秒）可配置。
【流式分块｜StreamChar】时长策略最特殊——训练侧受限于「training data contains no videos/transcripts longer than 20 seconds」（训练数据中无超过 20 秒的视频/台词），推理侧却要生成 5 分钟连续流，靠 chunk 拼接实现（每 chunk 33 帧 @24fps ≈ 1.375 秒），历史音频上下文窗口上限 15 秒。这构成一个显著的「训练短、推理长」的泛化跨度，也是其 progress-aware pointer 与 persistent visual anchor 两个抗漂移机制存在的根本原因。评测中「150 clips generating 10s audio-video pairs」与「50 clips paired with randomly sampled transcripts (>300 words) to produce 5-minute continuous streams」。
【固定短时｜ITS-JAVG（评测侧）】所测基座各自固定：JavisDiT 生成 4 秒视频、MMDisCo 生成 2 秒视频。
【未披露｜Baton、CCL】Baton 的 Sem100 评测集为「100 unseen videos (10 seconds long)」，训练片段时长未说明[不确定]；CCL 完全未提时长分布[不确定]。
【切分策略】仅 NAVA 提到「raw videos are first segmented at scale with a Hadoop-based pipeline」（大规模 Hadoop 管线切分），其余均未描述切分策略[不确定]。

### [音视频联合生成基线合集](../models/JavisDiT_baselines.md) ⚠️

各工作的时长策略高度体现其算力与目标差异，共同特征是「短、定长、不做多时长分桶」：
【MM-Diffusion】源视频切成 10 秒不重叠片段作为数据集单位，但实际训练样本更短：Landscape 与 AIST++ 均按固定帧数采样，音频对应片段长度约 1.6 秒量级[不确定：论文原文的具体训练片段秒数]。
【AV-DiT】最短：每个训练样本为 16 帧视频 + 截断或补齐到 1.6 秒的音频波形（16kHz）。视频与音频的时长严格对应，无分桶。
【JavisDiT / JavisDiT++】训练与评测统一为 4 秒片段（240P / 24fps），论文另做了延长至 10 秒的扩展测试。JavisDiT++ 的全部 JavisBench 评测同样固定在「240P、4 秒」。数据准备侧的硬约束：音频统一截断到 30 秒以内后再切分、视频至少 10 帧否则丢弃、fps 统一归一到 16Hz。
【Harmony】人类语音片段明确为 3–10 秒（这是有明确区间而非定长的少数案例）；阶段一音频预训练的最大片段长度为 10 秒，其中参考音频（reference audio）为随机截取的 1–3 秒片段。
【UniAVGen】视频统一按 16 fps 处理后经 VAE 编码；具体片段秒数未明确给出[不确定]。
共性局限：五者训练样本均在 10 秒以内，无一涉及分钟级长视频、多镜头拼接或时长分桶调度，这是学术基线相对工业模型（如 Veo/Sora 2/LTX-2）最本质的差距。

### [可灵 3.0 / 可灵视频 3.0 Omni](../models/Kling_30_Omni.md) ⚠️

[不确定] 未公布训练片段时长分布。可确认的仅为清洗规则层面：Kling-Omni 报告称基础过滤含“分辨率与时长阈值”，并剔除“突兀场景切换与不连贯镜头转场”的片段（即倾向保留单镜头连续片段）。同团队公开工作可作参考：Koala-36M 经时序切分后平均clip时长13.75秒；Kling-Foley 将长视频统一切为8—10秒片段。可灵3.0 Omni 的目标生成长度为15秒，训练片段时长分布很可能覆盖并略超该长度以支持15秒/多镜头生成。[不确定：具体分桶与长短样本配比]

### [LTX-2](../models/LTX-2.md) ⚠️

LTX-2 未单独披露。可参考同源的 LTX-Video：论文 Fig.14(b) 给出过滤后 clip 时长分布直方图，范围约 0–30 秒，质量集中在较短片段（分布随时长单调下降）。切分策略：数据 pipeline 的输入单元本身即为「Input Shots（镜头）」，全流程以 shot 为处理粒度，最终产出「Final Shots」。训练时同时在多种「分辨率×时长」组合上训练，并通过 resize 使各样本 token 数近似相同。生成侧上限20秒，超过约20秒会出现时序漂移与同步退化。具体分桶数值未公布。[不确定]

### [LongCat-Video](../models/LongCat-Video.md) ⚠️

报告未给出片段时长的统计分布（无平均时长、无时长直方图）。切分策略明确：以场景检测将长视频切成「training-friendly clips while maintaining content consistency」（训练友好且内容一致的片段），工具为 PySceneDetect + 内部自训练的 TransNetV2 双路联合。切分后在元数据中记录 duration 字段并用于后续过滤，但过滤阈值未公布。训练时的目标时长在架构层面是固定的：全部五个预训练阶段与 SFT、RLHF 阶段均采用 93 帧（30fps 下约 3.1 秒）的视频长度；分钟级长视频不靠单次生成长片段，而靠 Video-Continuation（视频续写）任务链式外推实现——即以多个前序帧作为条件帧递归续写。Avatar 1.5 的在线过滤中「duration」是显式的逐级过滤条件之一。[不确定：时长分布数值与过滤阈值]

### [MOVA](../models/MOVA.md)

采用严格的固定时长切分策略，不做变长分桶：
- 所有训练片段统一为 8.05 秒，精确对应 24fps 下的 193 帧（首帧 + 8 秒视频，即 1 + 8×24 = 193）。
- 三个训练阶段的帧数恒定为 193，不随分辨率变化。
- 切分不是等间隔滑窗，而是由 VAD（语音活动）+ PySceneDetect（场景切换）联合驱动的窗口生成算法（详见 shot_segmentation 与附录 A.3 的两段伪代码）。
- 语音片段的窗口起点会自适应前移/调整，以避免截断正在进行的语句、保证口语内容的连续性。
- 推理侧输出同样为 8 秒左右（720p、8s、24fps 的 clip 约产生 1.6×10^5 个 token，论文在 Limitations 中将序列长度列为主要瓶颈）。

### [合并条目：① Mochi 1](../models/Mochi_MAGI_Motif.md) ⚠️

① Mochi 1：训练侧时长分布与切分策略完全未披露；仅知生成侧上限为 84 帧 / 30 FPS / 5.4 秒。[不确定]
② MAGI-1：给出分阶段的时长上界（Table 5）——stage-1 与 stage-2 均为「≤ 8 秒」，stage-3 放宽到「≤ 16 秒」，论文说明最后阶段延长时长是为让模型捕捉更丰富的时序动态。切分策略为 PySceneDetect 切成单镜头 clip（详见 shot_segmentation）。未给下界、未给分布直方图。[部分不确定：时长下界与实际分布]
③ Motif-Video 2B：切分后设有明确下界——「Clips shorter than two seconds after merging are discarded（合并后短于 2 秒的片段丢弃）」，目的是保证每个训练片段覆盖有意义的时序跨度。训练侧的时长实际由帧数桶决定：帧数桶为单帧图像、33 帧、65 帧、121 帧四档，并在每个阶段转换时重新施加 clip-length 过滤（阈值随分辨率提升而收紧）。未给时长直方图。[部分不确定：具体分布]

### [Movie Gen](../models/Movie_Gen.md)

原始视频4秒~2分钟、平均28秒；因训练需要4~16秒clip，先用 FFmpeg 做场景变化检测，每条视频采样1~2个时长超过16秒的场景，再从每个场景中随机抽取1个4~16秒的clip，明确避免不看镜头边界的随机采样（否则生成视频会频繁突兀转场）。超过50%的训练clip时长落在15~16秒。
分桶策略（表2，五个时长桶，桶内latent帧数一致便于batch）：10.67s@24fps→256帧/32 latent帧；16s@16fps→256帧/32 latent帧；12~16s@21~16fps→256帧/32 latent帧；8~12s@24~16fps→192帧/24 latent帧；4~8s@32~16fps→128帧/16 latent帧。前两桶取自10.67~12秒与≥16秒长视频的中段clip。帧率通过caption中的FPS token控制（16~32 FPS）。
SFT视频时长限定10.6~16秒，其中50%为16秒、50%为10.6~16秒；16秒的用16FPS训练，10.6~16秒的用24FPS训练。
音频侧：视频长度限制在4~120秒，预训练时序列上限30秒（750帧）超出则随机切块；微调时随机采样10秒和30秒片段；caption同时按10秒与30秒两种chunk制作，训练中按 5 batch : 1 batch 采样。

### [NVIDIA NeMo Curator](../models/NeMo_Curator.md) ⚠️

【切分策略（框架能力）】提供两种 clip 抽取模式，均可配置：(1) fixed-stride 固定步长切分（按固定秒数硬切，配置 clip 长度与步长）；(2) scene-change detection 场景变化检测切分（基于 TransNetV2，shot-aware）。二者可叠加使用——先按镜头切，再对超长镜头按固定步长细分。
【生产实践中的时长阈值（Cosmos WFM）】切分后丢弃短于 2 秒的 clip；长于 60 秒的 clip 进一步细分；最终 clip 时长分布区间为 2–60 秒。captioning 时的处理粒度为「每 256 帧生成一条 caption」，即超长 clip 会被切成多个 caption 段。
【分布数值】各时长区间的具体样本占比未公布。框架本身不强制任何时长分布，由用户配置决定。[不确定]

### [OmniHuman 数据集 + OHBench](../models/OmniHuman.md) ⚠️

【分布图】片段时长分布由 Figure 3(d) 给出，正文未给出任何具体百分比、均值、中位数或分位数。[不确定]
【可反推的平均值】由 1,800 小时 / 100 万条可得平均约 6.5 秒/条，指向以 5–10 秒短片段为主体的分布形态。这一量级与当前 AV 联合生成模型的原生生成窗口（Ovi 5 秒、LTX-2 约 10 秒、Wan 系 5 秒）高度吻合，说明切分粒度是按下游训练需求设计的。
【切分策略】时长由 TransNetV2 的镜头切分结果自然决定，而非人为设定定长窗口——即一个 clip 对应一个完整镜头（详见 shot_segmentation）。这与「先定长切窗、再检查是否跨镜头」的路线相反，好处是每个片段内部天然无转场，代价是时长分布不可控、需要额外设置上下界裁剪（论文未说明是否设置了时长上下界）。[不确定]
【时长下界的隐含约束】音频治理阶段会剔除「时长异常」（abnormal duration）的样本，说明确实存在时长上下界的准入规则，但具体数值未披露。[不确定]
【逐样本时长可查】HuggingFace 的 metadata 中包含每条样本的 fps、duration、resolution 字段，以及 clip_start_sec / clip_end_sec，因此完整的时长分布对使用者是可实测的——这在一定程度上弥补了论文披露的不足。

### [Open-Sora 系列](../models/Open-Sora.md)

【Open-Sora 2.0】预处理阶段先剔除时长 <2 秒的原始视频；切镜后，长于 8 秒的片段被强制切成多个 8 秒段，短于 2 秒的片段丢弃，因此训练片段时长严格落在 [2s, 8s] 区间。论文 Figure 3 统计显示时长分布明显右偏，**接近一半的片段集中在 6–8 秒**。同时限制输出片段 fps<30。
【Open-Sora 1.x】片段时长 2–16 秒，配合 bucket 机制按帧数分桶，支持 2s~15s 变长训练。
【Open-Sora Plan v1.3】用 ffmpeg 统一切成 16 秒片段，随后用 LPIPS 做跳切（jump cut）检测，只保留帧数落在 [32, 512] 区间的片段（24fps 下约 1.3–21 秒）。
【Open-Sora Plan v1.5】视频训练从 57 帧 @24fps（约 2.4 秒）逐步升到 121 帧 @24fps（约 5 秒）。

### [Ovi](../models/Ovi.md) ⚠️

策略是「统一定长、不做多时长分桶」，这是 Ovi 数据设计上最鲜明的特点之一。
【音视频数据】经场景检测切分后统一取 121 帧 @24fps 的定长片段（= 5.04 秒），初版训练与推理均为该长度；Ovi 1.1 扩展到 10 秒（对应约 241 帧 @24fps）[10秒对应的帧数为推断，不确定]。
【纯音频数据】双时长设计：预训练用最长 12 秒的变长波形（论文明确「use variable-length audio to maximize coverage of diverse acoustics」，靠长波形学到说话人音高、情绪的长程一致性）；微调用 padding 到精确 5.04 秒的定长波形，以对齐 121 帧视频的时长，避免进入音视频融合阶段时再做时序重适配。
【设计动机】论文说明对所有注意力层统一施加 scaled RoPE，就是为了「避免转入音视频微调阶段时的重适配、免于维护多套音频 RoPE 尺度」。
【局限自述】论文 Limitations 明确承认模型被限定在短 5 秒片段，分钟级叙事、镜头间转场、全局故事一致性均不在范围内，并提出未来用 chunk-wise causal 音频模型 + 以上一段末帧为条件的因果视频骨干来拼接多个 5 秒 chunk。

### [Script-a-Video](../models/Script-a-Video.md) ⚠️

论文未披露任何片段时长信息：未给出 500K 片段的平均时长、时长上下界、时长直方图，也未说明切分后是否做定长截断或分桶。
可间接推断的线索：
1) MTSS 的 Shot 流以 "time_range" 为每个镜头的锚定字段，Event 流同样带 "time_range"，并在 Shot 的 visual_description 内部嵌入 intra-description timestamps（描述内时间戳）以锚定微动作到全局时间轴——说明每个片段都有一条明确的全局时间轴，且时长足以容纳多个镜头与多个音频事件；
2) 生成侧区分单镜头（125 条评测）与多镜头（100 条评测）两类，多镜头样本必然长于单镜头；
3) 评测指标包含 Shot Boundary Deviation（镜头边界偏差，以帧为单位，最优 0.38 帧），说明生成侧的时长与帧率有确定配置，但具体数值未披露。
切分策略同样未披露——论文未说明 500K 片段是如何从长视频中切出的（是否用 PySceneDetect、是否按场景切、是否人工挑选）。这是本工作数据侧披露最薄弱的环节之一。[不确定]

### [Seedance 2.0 与 Seedance 1.5 pro](../models/Seedance_20_Seedance_15_pro.md) ⚠️

[不确定] 1.5 与 2.0 均未披露训练片段时长分布。可参照 Seedance 1.0：原始长视频经镜头感知时序切分后，切片最大时长为 12 秒；预训练第一阶段使用 3–12 秒（12 fps）的视频片段与 256px 图像联合训练。生成侧可反推数据具备的时长覆盖：1.5 pro 支持约 10 秒级生成，Seedance 2.0 支持 4–15 秒的直出音视频生成，官方博客提及 15 秒高质量多镜头音视频输出，暗示训练数据已覆盖 15 秒量级的多镜头片段。

### [SkyReels 系列](../models/SkyReels.md) ⚠️

【SkyReels-V2】未给出时长直方图，但明确采用「双轴分桶（dual-axis bucketing）」：沿时间长度与空间宽高比两个维度构造 BT×BAR 的桶矩阵，样本按所属桶组织并做 FPS 标准化；切分粒度为单镜头 clip（所有原始视频先经镜头边界检测切成单镜头片段）。Diffusion Forcing 阶段进一步支持可变长度、乃至理论无限长的生成（论文以「无限时长」为卖点，实测可稳定生成30秒/60秒级片段）。具体桶边界与各桶占比未公布。
【SkyReels-V4】训练与生成的统一时长上限为15秒，Stage3 明确训练片段时长范围为「2–15秒」。音频侧做显式时长控制：长音频切成15秒块（chunk），过短音频按类别拼接凑到15秒——这是本条目中最具体的时长处理策略。视频侧各时长区间的样本占比未披露。[不确定]

### [Sora 2](../models/Sora_2.md) ⚠️

训练数据的片段时长分布与切分策略完全未披露。仅知推理侧输出规格：初始10秒；2025年10月更新后所有用户15秒，ChatGPT Pro 网页版最长25秒；Sora 2 Pro API 支持 10s/15s/25s 档位。训练片段时长如何分桶、如何从长视频切分为训练clip，无任何信息。（前代 Sora 1 曾说明按原生时长训练、不做统一裁剪到固定帧数，Sora 2 是否延续未确认。）[不确定]

### [Step-Video-T2V](../models/Step-Video-T2V.md)

未给出时长直方图，但切分与分桶策略明确：
【切分】用 PySceneDetect 的 AdaptiveDetector 函数检测场景变化，再由 FFmpeg 切分为单镜头（single-shot）片段；每个切出的片段还会额外丢弃首尾各 3 帧，以消除转场处不稳定的镜头运动与残留过渡帧——这是一个细颗粒但实用的工程细节，同期不少工作未做。
【时长表达】训练侧以帧数而非秒数表达，采用帧长分桶（frame-length bucketization）：1 帧、68 帧、136 帧、204 帧四档（1 帧档即图像，用于图像-视频联合训练）。模型最长单次生成 204 帧；按报告口径 204 帧对应约 8 秒级（衍生模型 Step-Video-TI2V 为 102 帧/5 秒，可反推约 20fps 左右口径）。
即课程设计上把「短→长」做成了显式的帧数分桶轴，而非固定 clip 时长。

### [UniTalking](../models/UniTalking.md) ⚠️

论文完全未披露片段时长信息：未给出最短/最长时长约束、未给出平均时长、未给出时长直方图，也未说明切分后是否统一截断为定长。可推断的间接线索：
【视频侧】视频分支继承 Wan2.2-5B，该基座的原生生成规格为 5 秒 / 24 fps（121 帧），因此训练片段极可能被规整到 5 秒左右的定长窗口，但论文无任何文字支撑。[不确定]
【音频侧的唯一硬数字】参考音频（reference audio）被显式约束在 3 至 5 秒之间——这是全文关于时长的唯一定量规定，但它约束的是合成的条件信号，不是训练样本主体的时长。
【latent 长度约束】条件侧的序列长度被固定：文本 latent 固定为 512，参考音频 latent 固定为 257，超长截断、不足补零。参考音频 latent 长度 257 结合 MMAudio VAE 的时间分辨率，对应约 8 秒的容量上限，即 3–5 秒的参考音频在其中留有较大补零余量。主生成流的音视频 latent 长度未披露。[不确定]
【切分策略】未提及任何切分策略（见 shot_segmentation）。

### [UniVerse-1](../models/UniVerse-1.md)

【切分产物时长】以 PySceneDetect 场景切分为基础，切分后长度短于 5 秒的片段一律剔除，因此保留片段的时长下界为 5 秒。论文未给出上界、均值或时长直方图，也未说明是否对长片段做二次截断，故实际保留的是变长片段而非定长片段。
【训练时的时长】不直接使用完整片段，而是由在线标注 pipeline 在每次采样时从片段中随机抽取一个定长窗口（论文举例为 5 秒）送入训练。即「入库变长（≥5s）、消费定长（约 5s）」的两级设计，随机窗口位置本身带来数据增广。
【VGGSound/AudioSet】走简化处理流程，同样施加 5 秒最短时长约束。
【与视频帧率的关系】视频统一为 25 fps，5 秒对应 125 帧；音频采样率被专门下调至 25.6 kHz 以与 25 fps 的时间栅格对齐。
【对比】与 MOVA 的严格定长 8.05 秒（193 帧 @24fps）+ VAD 感知窗口相比，UniVerse-1 的切分策略明显更粗放：无 VAD 驱动的窗口起点自适应，不保证窗口不截断语句。

### [Unison](../models/Unison.md) ⚠️

论文未直接给出任何时长分布数据、切分策略或时长阈值，本字段绝大部分内容为推算与间接推断。
【可推算的平均时长】由公开数字反推：音视频侧 3,000 小时 ÷ 200 万条 ≈ 5.4 秒/条；纯音频侧 130,000 小时 ÷ 5,000 万段 ≈ 9.4 秒/条。约 5.4 秒的平均 clip 时长表明这是一个短片段语料，与音视频联合生成领域的通行做法一致（UniVerse-1 约 5 秒窗口、MOVA 8.05 秒定长）。
【切分策略】完全未描述。论文既未提及任何镜头切分工具，也未说明是沿用上游数据集的既有切分还是重新切分。合理推断是前者——OpenHumanVid、HDTF、VFHQ、CelebV-Text、VGGSound 五个数据集本身就以「已切分好的 clip」形式发布（例如 VGGSound 为固定 10 秒片段、VFHQ 为已裁剪的人脸视频片段），Unison 的「自动化处理 pipeline」更可能是在既有 clip 基础上做筛选与再裁剪，而非从长视频重新切分。[不确定]
【训练时的时长配置】未披露 Stage 2 联合训练时每个样本的帧数/秒数配置，仅知输出为 25 FPS。若按约 5 秒计，对应约 125 帧。[不确定]
【下界/上界/直方图】均未给出。
【与 VGGSound 的口径冲突】VGGSound 原始发布为统一 10 秒片段，若原样使用则应拉高平均时长，而实测均值仅 5.4 秒，暗示 pipeline 对片段做了二次裁剪或 VGGSound 在 200 万条中占比很小。此为推断，无原文支撑。[不确定]

### [Veo 3 / Veo 3.1](../models/Veo_3_Veo_31.md) ⚠️

[不确定] 训练侧片段时长分布与切分策略完全未披露。仅可从产品侧反推：Veo 3 单次生成固定 8 秒；Veo 3.1 支持 4 / 6 / 8 秒基础片段，并通过 Extend 功能级联延长（Flow 中可延长至约 60 秒乃至 140 秒以上）。基础生成单元固定在 4-8 秒量级，间接暗示训练数据主体为经镜头切分后的秒级短片段而非长视频。

### [Vidu S1](../models/Vidu_S1.md)

切分后训练片段时长为 3~60 秒的单镜头 clip。切分策略：先沿镜头边界（shot boundaries）分割保证单镜头连续性；长镜头进一步细分，且切点被约束为不得落在一句话语音的中间（cut points constrained so as not to fall in the middle of speech），以保护语音-唇形的完整性。未给出时长的具体分布直方图或平均时长。

### [主流视频预训练数据集合并调研：Panda-70M、InternVid、Koala…](../models/pretraining_datasets.md) ⚠️

**片段时长是七者最本质的分野，直接对应「短片段易学、长片段难得」的取舍**，从8.5秒到72秒跨越近一个数量级：
- **Panda-70M（8.5秒，最短）**：切分后丢弃<2秒的片段，源视频超过60秒的截断至前60秒；另有一条关键操作——**每个片段首尾各裁掉10%**以去除不稳定镜头运动与转场残留。最终平均8.477秒。
- **InternVid（11.7秒）**：爬取时限定源视频10秒–30分钟；clip 时长2秒至30秒以上，**85%的clip落在0–10秒**。
- **Koala-36M（13.75秒，或按172K小时口径17.2秒）**：片段由转场检测自然切出，无固定长度截断；组装规则是片段需长于**8帧**才保留，且**首尾各腐蚀4帧**（cuts.append((start+4, i-4))）以确保内容内无转场——与 Panda-70M 的10%裁边是同一思路的不同实现。
- **OpenVid-1M（7.2秒，OpenVidHD 9.6秒）**：最短时长阈值未披露。[不确定]
- **UltraVideo（short 5.3秒 / long 30.9秒）**：**显式的双分割设计**——切镜后按帧数分流，3–10秒进 short 集，>10秒进 long 集。为扩充 short 集还做了定向截取：源视频<60秒的取**中间10秒**，>60秒的额外从**两侧各取10秒**。保留原生帧率不做抽帧。
- **LVD-2M（20.2秒，最低门槛10秒）**：**以「≥10秒」为一等设计目标**（四条准则之首），且不做上限截断，因此长尾丰富（>30秒的占约13.5%）。打标时才把>30秒的视频切成30秒段分别描述。
- **MiraData（72.1秒，最长）**：策略与所有其他数据集相反——**不是切得更准，而是切完再缝回去**。YouTube 来源的片段要求**>40秒**才保留，Videvo/Pixabay/Pexels 来源的要求**>10秒**。（v0 版曾把超过2分钟的硬切成2分钟段，v1 改为拼接策略。）实测官方100条样本集平均105.1秒，比数据集均值更长。
**汇总规律**：以 clip 长度换质量的三条路线清晰可辨——Panda/InternVid/OpenVid 走「短而多」，Koala 走「中等且转场干净」，LVD-2M/MiraData/UltraVideo-long 走「长镜头稀缺资源」。

## 分辨率/宽高比分布与分桶策略

`resolution_aspect_distribution` · 详细程度: brief

### [Allegro](../models/Allegro.md) ⚠️

按训练阶段分层的分辨率门槛（Table 1）：
· T2I 预训练与 T2V 360p 预训练：宽 W≥640 且高 H≥368（对应 368×640 训练分辨率）；
· T2V 720p 预训练与微调：宽 W≥1280 且高 H≥720（对应 720×1280 训练分辨率）；
· 原始入口另有 ≥360p 的最低门槛。
训练时分辨率并非多桶动态分辨率训练，而是固定两档（368×640 → 720×1280），走「低清→高清」的课程式提升。宽高比方面论文未披露分桶策略与多宽高比训练，训练分辨率均为 16:9 竖排表示的 720×1280 / 368×640。[不确定]（宽高比分布与分桶策略）

### [Apollo](../models/Apollo.md) ⚠️

论文未披露分辨率与宽高比的分布或分桶策略：未给出训练分辨率、未给出宽高比枚举、未提及多分辨率分桶（bucketing）或渐进式升清课程。相关的仅两处：(1) 过滤阶段会「discard those videos with low resolution」（剔除低分辨率视频），但未给出分辨率下限阈值；(2) Video-VAE 采用 CogVideoX 的 3D causal visual encoder，对高、宽各做 16 倍压缩，并声明可处理「varying resolutions and frame rates」，即架构上支持变分辨率输入，但训练实际使用的分辨率配置未公开。[不确定]

### [CineDance / CineDance-1M](../models/CineDance.md) ⚠️

【分辨率下限】硬性要求最低 1080p，是该数据集相对同类工作的关键卖点之一——论文对比表（Tab.6）中，同时满足「1080p + 原生音轨 + 镜头级双模态密集标注」三项的仅 CineDance-1M 一家。
【空间预处理】采用「粗裁 + 细验」两道：粗裁阶段用 EasyOCR 定位并裁除烧录字幕区域，用 FFmpeg 黑边检测（black-border detection）裁除信箱式黑边（letterboxing）；镜头切分完成后再对每个 clip 级片段复跑一次 OCR 与黑边检测做细粒度复验。
【宽高比分布 / 分桶策略】论文未给出宽高比的统计分布，也未描述训练时的分辨率分桶（bucketing）策略；由于素材以影院级影视内容为主，推测以宽银幕比例为主但无数据支撑。[不确定]

### [CogVideoX](../models/CogVideoX.md) ⚠️

· 分辨率课程（progressive training）：先在 256px 训练学习语义与低频知识，再逐级提升到 512px、768px 学习高频细节。训练超参表给出各阶段最大分辨率：stage1 256×384 → stage2 480×720 → stage3 768×1360 → stage4(FT) 768×1360。CogVideoX-5B 最终输出 720×480，CogVideoX1.5-5B 输出 1360×768、10 秒、16fps；产品版新清影可达 4K/60 帧。
· 宽高比：明确保持原始宽高比不变，只把短边 resize 到目标分辨率（「we keep the aspect ratio unchanged and resize the short side to above resolutions」），从而保留生成任意比例视频的能力；CogVideoX1.5-5B-I2V 支持任意分辨率输出。未给出横屏/竖屏配比数字 [不确定]。
· 分桶 vs packing：不使用 SDXL 式 bucketing，改用 Multi-Resolution Frame Pack 把不同分辨率与时长的样本打包进同一 batch。
· RoPE 适配高分辨率时对比了插值（interpolation）与外推（extrapolation）两种方案，最终选择外推以保留局部细节与相对位置关系。

### [Cosmos-Predict2.5](../models/Cosmos-Predict25.md) ⚠️

(1) 分桶：sharding 阶段显式沿 resolution（分辨率）与 aspect ratio（宽高比）两个轴分片，与 content type、length 共四轴，目的是支撑「efficient sampling, curriculum-based training, and fine-grained domain balancing」，因此存在明确的分辨率/宽高比分桶机制，但各桶占比未公布。
(2) 分辨率被用作去重的仲裁准则：语义去重时同簇内保留最高分辨率版本（the highest-resolution version is retained），理由是高分辨率保留更细的视觉细节、提供更丰富的训练信号；在线增量去重时以「更早 + 更高分辨率」作为 tie-breaking 优先级。
(3) 训练侧分辨率课程明确：256p（320×192）→ 480p（832×480）→ 720p（1280×704），逐级提升，每级待模型收敛、视觉质量饱和后再进入下一级。
(4) 冷却阶段专用一批精选 4K 高清视频（388K 条），学习率线性衰减至 0，用于提升细节精度与运动平滑度。
(5) 架构上移除了绝对位置编码、仅保留相对位置编码（3D RoPE），明确目的是提升对训练时未见过的分辨率与序列长度的泛化能力。
(6) 机器人域预过滤会剔除低分辨率视频。
[不确定：原始语料各分辨率/宽高比的占比数值]

### [Data-Juicer 2.0](../models/Data-Juicer.md) ⚠️

【相关算子】提供过滤与改写两类算子，覆盖较完整：
  · video_resolution_filter —— 按分辨率上下界过滤，剔除低清素材。
  · video_aspect_ratio_filter —— 按宽高比区间过滤，剔除极端长条/竖屏或异常比例样本。
  · video_resize_resolution_mapper —— 按宽高约束改写分辨率（而非丢弃），可用于把混合分辨率数据统一到目标规格。
  · video_resize_aspect_ratio_mapper —— 把视频重采样到指定宽高比区间内。
  「过滤 + 改写」双路设计是 DJ 相对纯过滤型框架的特点：不合规样本可以被修复而非直接丢弃，对数据量宝贵的场景更友好。
[不确定] DJ 未内置视频生成训练常用的「分辨率分桶」（resolution bucketing）功能——即按分辨率/宽高比把样本分组以便同批次训练，这属于训练框架侧（如 EasyAnimate、Diffusers）的职责，DJ 只负责把数据整理到可分桶的状态。官方 T2V 案例也未披露数据池的分辨率与宽高比分布统计。

### [Foley-Omni](../models/Foley-Omni.md) ⚠️

仅有过滤下界，无分布统计。过滤阶段设置视觉分辨率硬门槛 ≥480p，并配合码率门槛 ≥1 Mbps（双重把关，防止低码率高分辨率的「虚高清」样本混入——这一码率约束在同类工作中较少见，UniVerse-1 使用的是码率比 bitrate ratio，思路类似）。
[不确定] 未披露最终数据集的分辨率分布直方图、未披露宽高比（aspect ratio）分布、未提及任何分辨率或宽高比分桶（bucketing）策略。这一点在本项目中影响相对有限：由于视频仅作为条件输入且经 CLIP 与 Synchformer 编码为固定维度特征，分辨率主要影响特征抽取质量而非生成分辨率，因此不需要像文生视频模型那样做严格的分桶训练。

### [Goku](../models/Goku.md) ⚠️

【入库门槛】min{height, width} ≥ 480，即短边不低于 480 像素。
【分辨率分桶】按 480p / 720p / 1080p 三档分桶，且每档使用一套独立收紧的过滤阈值（论文 Table 4），高分辨率档位阈值更严格 → 数据量逐档收缩：480p 档 36M、720p 档 24M、1080p 档 7M。这套「分辨率越高、质量门槛越高、数据越少」的分桶设计直接服务于渐进式分辨率训练课程。
【训练分辨率序列】288×512 → 480×864 → 720×1280，均为 16:9 横屏比例；I2V/高分辨率阶段涉及 1080p 数据。
【宽高比】论文以固定 16:9 分辨率（288×512、480×864、720×1280）组织训练，未描述多宽高比（multi-aspect-ratio bucketing）分桶策略，也未给出竖屏/方屏数据占比。[不确定]（宽高比分布与是否支持任意比例训练）

### [HunyuanVideo-Foley](../models/HunyuanVideo-Foley.md) ⚠️

论文完全未讨论视频的分辨率、宽高比与分桶策略——这是 V2A 任务的结构性特点决定的：视觉信息只作为条件输入，且经 SigLIP-2 编码为固定维度的语义特征向量，原始分辨率在编码后即被抹平，因此不存在视频生成模型那种「训练分辨率决定输出分辨率」的约束，也就无需做分辨率分桶或宽高比对齐。
【实际约束】视觉侧的唯一处理是 SigLIP-2 视觉编码器的固定输入尺寸（通常为 224×224 或 384×384 的正方形裁剪/缩放），以及 Synchformer 的输入规格。论文未披露具体使用的 SigLIP-2 变体（so400m / base / large）与输入分辨率。[不确定]
【帧率】论文未说明视频帧的采样帧率——这对同步精度相当关键（Synchformer 的帧级同步特征密度直接受采样帧率影响），但正文与附录均无数值。[不确定]
【清洗漏斗中无视觉质量门槛】值得注意的是，整条清洗流水线中没有任何针对视频画质的过滤条件（无分辨率下限、无码率约束、无美学评分、无模糊/压缩伪影检测）——所有质量把关都集中在音频侧。这与 UniVerse-1「分辨率 ≥1080p + 码率比 ≥600 + DOVER ≥0.6」的严苛视觉门槛形成极端反差，直接原因是本模型不生成画面，画质高低不影响输出保真度，只需视觉语义可识别即可。这是一个由任务形态推导出的合理简化。

### [HunyuanVideo](../models/HunyuanVideo.md)

【原版】分辨率本身即是分层过滤漏斗的分层轴：构造 256p → 360p → 540p → 720p 四档递进数据集 + 一档 SFT 多尺度数据集，越高档过滤越严。训练侧采用 bucketing 策略同时支持多分辨率与多宽高比（并支持 1–129 可变帧数），从而可生成任意宽高比视频。各宽高比的具体占比未公布。
【1.5】阶段化更清晰：图像 256p → 512p；视频 256p → 480p → 720p，CT/SFT 阶段同时训练 480p 与 720p；另有独立超分模块用 1K–4K 数据训练，把 480p/720p 基座输出提升到 1080p 级别。宽高比分桶细节未披露占比。

### [InstructAV2AV](../models/InstructAV2AV.md) ⚠️

同样高度统一：全部为 720p / 24 FPS。分辨率由数据引擎的合成底座（Wan2.2-5B，即 Wan2.2-TI2V-5B）与模型训练分辨率共同决定，属于「合成即定标」——不需要像真实数据那样做分桶。
【质量下界控制】用 LAION Aesthetics Predictor（Romain and Christoph, 2022）过滤视觉低质片段，但 [不确定] 论文未给出美学分阈值数值。
[不确定] 未披露宽高比（aspect ratio）分布——从来源看，电影类素材（MovieBench、Condensed Movies、Short-Films-20K）多为宽银幕 21:9 或 16:9，VGGSound 与 YouTube 素材多为 16:9，推测最终统一为 16:9 单一比例（720p 通常指 1280×720），但论文未明说是裁剪、缩放还是加黑边处理，也未提及任何分桶（bucketing）策略。
【与文生视频模型的差异】文生视频模型需要多分辨率多宽高比分桶训练以支持任意输出尺寸；本工作作为编辑模型且数据全合成，单一规格即可，这是合成数据路线在工程上的一处简化红利，代价是模型对非 720p/16:9 输入的泛化能力未经验证。

### [2026 其他音视频联合生成](../models/JAVG_2026_misc.md) ⚠️

七项中仅少数披露，且普遍采用「低清训练 + 高清精修」的两级策略而非分桶：
【ALIVE】明确两级：基座在 480p / 24fps 上用 11M 样本训练，另设独立的 1080p Refiner 阶段用 0.7M「high-clarity samples」训练 1 epoch。这是一个典型的「大规模低清打底 + 小规模高清提纯」课程。宽高比分布与分桶策略未披露[不确定]。此外 ALIVE 有专门的 clarity filtering（清晰度过滤）阶段，用「reference images across six distinct clarity levels」（六档清晰度参考图，从模糊到锐利）作为评判基准，本质上是把分辨率/清晰度做成了六级序数标签而非连续分数——这一做法在同期工作中较少见。
【OmniCustom】完全统一归一化，不做分桶：「All videos are recorded in 480p at 24 FPS. We extract audio files from videos and unify them into 16kHZ.」——视频统一 480p/24fps，音频统一 16kHz。这与其单人说话头的窄任务定位一致。
【CCL】训练分辨率 352×640p（非标准分辨率，接近 9:16 竖屏），是七项中唯一给出精确像素尺寸的工作；单一分辨率训练，无分桶。
【NAVA】训练侧分辨率分布未披露[不确定]；推理侧默认 704×1280（9:16 竖屏），仓库称「flexible aspect ratios supported」，说明具备多宽高比能力，但训练侧的配比未说明。
【ITS-JAVG（评测基座）】JavisDiT 为 240p，MMDisCo 为 256×256——反映学术界开源 JAVG 基座的分辨率仍显著落后工业界。
【Baton、StreamChar】未披露分辨率/宽高比策略[不确定]。
【共性】七项中无一采用 Ovi 式的「等面积归一化」或 Seedance 式的多分辨率分桶调度，普遍是单一分辨率训练 + 可选高清精修，说明这批工作的算力预算与工程复杂度都偏节制。

### [音视频联合生成基线合集](../models/JavisDiT_baselines.md) ⚠️

五者的分辨率均远低于工业模型，且全部为固定分辨率、无宽高比分桶：
【MM-Diffusion】基础扩散在 64×64 像素空间进行，再经独立的超分（SR）模型上采样到 256×256；仓库同时提供基础模型（Landscape.pt / AIST++.pt）与对应 SR 模型（Landscape_SR.pt / AIST++_SR.pt）。这种「低清生成 + 独立 SR」的两段式在 2022 年是主流做法。宽高比固定为 1:1，无分桶。
【AV-DiT】视频帧裁剪（crop）到 256×256 分辨率，视频 latent 尺寸 32×32×4，音频 latent（mel 频谱）40×16×8。单一分辨率、单一宽高比。
【JavisDiT / JavisDiT++】训练与评测统一 240P、24fps（数据准备阶段 fps 归一到 16Hz）。数据 CSV 中虽保留 height、width、aspect_ratio、resolution 等字段（沿用 Open-Sora 的数据管理 schema，理论上支持多分桶），但论文报告的实验全部为固定 240P/4 秒配置，未见分桶策略的实际使用[不确定]。
【Harmony】论文未指明视频分辨率与帧率[不确定]；底座 Wan2.2-5B 原生支持 720P，推测继承其分辨率能力但未在文中确认。
【UniAVGen】仅明确视频按 16 fps 处理后经 VAE 编码，分辨率与宽高比未披露[不确定]。
共性：无一采用面积归一化（如 Ovi 的固定像素总数 518400）、无一采用多宽高比分桶调度，说明这批工作把算力集中在跨模态对齐机制而非视觉保真度上。

### [可灵 3.0 / 可灵视频 3.0 Omni](../models/Kling_30_Omni.md) ⚠️

[不确定] 未公布分辨率/宽高比分布与分桶策略。已知：模型支持原生4K（3840×2160）60fps 输出，官方强调是扩散过程中像素级原生生成而非后置超分，暗示训练集包含相当比例的真4K/高帧率素材；同时支持多种宽高比（16:9/9:16/1:1等），业界通行的多分辨率/多宽高比分桶（bucketing）与渐进式分辨率课程极可能被采用，但报告未明说。可参考旁证：Koala-36M 统一为720p；Kling-Foley 要求源视频≥720P。KlingAvatar 2.0 采用时空级联（空间超分+时间插帧）框架实现高分高帧，提示可灵体系中4K/60fps 可能部分由级联精修模块承担。[不确定：4K原生生成与级联精修的边界]

### [LTX-2](../models/LTX-2.md) ⚠️

无分布统计数字，但策略明确。(1) 多分辨率多时长联合训练：同时在多种宽/高/时长组合上训练，模型对未见配置泛化良好；(2) token 数对齐而非 padding/packing：将原视频 resize 到可比 token 数，并施加 0%–20% 的随机 token dropping（stochastic token dropping）使各序列 token 数固定，作者称此法比复杂的 token-packing/padding 更简单高效且保留数据多样性；(3) 宽高比标准化：pipeline 中显式「Crop Black Bars」裁除黑边以统一宽高比并提升有效视觉面积；(4) pipeline 含「Resize Shots」步骤。推理侧要求分辨率能被32整除、帧数满足 8n+1。各分辨率/宽高比的具体配比未披露。[不确定]

### [LongCat-Video](../models/LongCat-Video.md) ⚠️

报告未给出原始数据的分辨率/宽高比分布统计，也未描述宽高比分桶（aspect-ratio bucketing）机制。元数据层面记录 resolution、frame rate、bitrate 三项并据此筛选。真正明确的是训练侧的分辨率课程：256p（Stage1、Stage2、Stage3）→ 480p（Stage4）→ 480p+720p 混合（Stage5、SFT、RLHF），即低清到高清的渐进式提分辨率，Stage5 起同一阶段内混合两档分辨率训练，使模型同时适配 480p 与 720p 输出。推理侧最终交付 720p / 30fps。此外模型采用「时空双轴 coarse-to-fine」生成策略：先生成低分辨率低帧率的粗视频再精化，这一策略要求训练数据能同时支撑多档分辨率。VAE 为 WAN2.1 VAE，时空压缩比 4×8×8，叠加 patchify 后整体为 4×16×16。[不确定：原始数据分辨率分布与分桶细节]

### [MOVA](../models/MOVA.md)

【标准化流程】原始视频先用 FFmpeg cropdetect 滤镜检测黑边并保留核心画面，再将主体内容居中、缩放至 720p，并按需对称补黑边（pillarbox 或 letterbox），统一规整为 9:16 或 16:9 两种宽高比。帧率统一重采样为 24fps。因此数据侧只有两种宽高比，不存在多桶宽高比训练。
【分辨率课程】Phase 1 与 Phase 2 训练在 360×640（360p），Phase 3 上采样到 720×1280（720p）。对应发布 MOVA-360p 与 MOVA-720p 两个模型。
【工程影响】720p 阶段序列长度大幅增加，context parallelism 从 CP=8 提高到 CP=16，有效 batch size 从 128 降到 64，checkpoint 间隔从 5000 步缩短到 2000 步。
【效果】消融显示分辨率从 360p 提升到 720p 后，DeSync 从 0.475 微降到 0.485、IB-Score 从 0.286 微降到 0.277（几乎无退化），而 LSE-C 反而从 6.278 提升到 6.593、cpCER 从 0.177 降到 0.149，验证了先在低清建立跨模态对齐、再升清的课程有效性。

### [合并条目：① Mochi 1](../models/Mochi_MAGI_Motif.md) ⚠️

① Mochi 1：未披露训练数据的分辨率与宽高比分布及分桶策略；输出固定 848×480（约 16:9）。[不确定]
② MAGI-1：训练分辨率按三阶段推进——stage-1 为 256p/360p，stage-2 为 480p，stage-3 为 720p（Table 5）。VAE 推理侧采用滑窗支持任意分辨率（空间窗口 256×256、步长 192、25% 重叠；时间维不重叠），模型宣称支持任意分辨率，4.5B 版本默认 720×720。但训练数据的宽高比分布与分桶权重未披露。[部分不确定：宽高比分桶]
③ Motif-Video 2B：分辨率是整条管线最主要的分层轴——课程为 144p → 360p → 480p → 720p 四档十阶段，每次阶段跃迁都会重新施加 resolution / clip-length / motion / aesthetic 四类过滤且阈值更严。宽高比处理上有两条具体工程措施：一是用 ffmpeg cropdetect 基于亮度统计估计最大内容矩形，剔除 letterbox/pillarbox 黑边填充；二是把 OCR 检出区域与黑边裁剪框合成为单一最终矩形（排除内容区上方 20% 的台标与下方 20% 的字幕），在一次 ffmpeg 重编码中与分辨率缩放、帧率限制一并施加。训练时按「帧数桶 × 分辨率桶」联合分桶（帧数桶：1 / 33 / 65 / 121，每档再细分多个空间分辨率），并由离线 bucket 均衡采样器保证各 rank 上各桶样本数的变异系数最小。未给宽高比的定量分布。[部分不确定：宽高比定量分布]

### [Movie Gen](../models/Movie_Gen.md)

分辨率门槛随训练阶段收紧：低分辨率训练阶段要求最小宽高 ≥720px；高分辨率训练阶段要求最小宽高 ≥768px（表44中该步骤把数据从100%砍到25%）。训练分辨率从256px渐进到768px，另有独立的空间超分模型输出1080p HD。
宽高比：预训练集控制为 60% 横屏 + 40% 竖屏（偏好横屏，因其时长更长、美学更好、运动更稳定）；高分辨率集调整为 80% 横屏 + 20% 竖屏；清洗漏斗中「宽≥高」这一步把数据从25%砍到7%。
分桶：图像与视频各用五个宽高比桶，同桶内latent形状完全一致以便batching，因此模型可输出多种比例，如横屏1024×576、竖屏576×1024。
黑边问题单独处理：自研边框检测器（一阶导数找大梯度像素+扫描线算法定位边框）剔除带黑边视频，竖屏视频尤为常见。
音频侧对视频质量的要求较低：剔除分辨率<480px的视频即可。

### [NVIDIA NeMo Curator](../models/NeMo_Curator.md) ⚠️

【框架能力】分辨率与宽高比不作为过滤条件，而是作为分片（sharding）阶段的组织维度：最终 WebDataset 按「分辨率 × 宽高比 × 时长」三维分桶打包，目的是与下游训练课程（training curriculum）的分桶需求对齐，使训练时能按桶取样、避免同 batch 内 token 数差异过大。这是该 pipeline 在「数据基建与训练课程耦合」上最值得借鉴的设计。
【转码归一】所有 clip 统一转码为 H.264 mp4，编码器可选 h264_nvenc（GPU）/ libx264 / libopenh264（CPU）；解码器可选 nvdec（GPU）/ ffmpeg。抽帧阶段（ClipFrameExtractionStage）可配置目标 fps 与抽帧策略（如 sequence 策略供美学打分使用）。
【生产实践】Cosmos WFM 的输入素材为 720p 至 4K。各分辨率/宽高比桶的具体样本占比未公布。[不确定]

### [OmniHuman 数据集 + OHBench](../models/OmniHuman.md) ⚠️

【准入门槛】论文明确声明「所有视频均为高清或更高画质」（all videos are of high-definition quality or higher），即存在一条 720p 或以上的分辨率硬门槛，但未给出具体的像素数阈值，也未说明是按短边、长边还是总像素判定。[不确定]
【分布图】分辨率分布由 Figure 3(c) 给出，正文未给出各档位的具体占比。[不确定]
【宽高比】论文未讨论宽高比分布，未描述分桶（bucketing）策略，未说明是否做了统一裁剪或补边。考虑到数据源为 YouTube 且内容类型涵盖脱口秀、游戏、测评（多为 16:9 横屏）与烹饪、教育类（可能含竖屏短视频），实际分布应以横屏为主但混有竖屏，然而无数据支撑。[不确定]
【帧率标准化】明确统一到 30 FPS（与音频 44.1 kHz 一同作为标准化规格）。这是全文少数几个明确的数值规格之一，且帧率统一对逐帧标注（134 关键点、SMPL/MANO 跟踪）与音视同步判定（±3 帧的偏移容差需要固定帧率才有确定含义）是必要前提。
【空间裁剪】在时空清洁阶段，会用 OCR 与 logo 检测估计文字污染区域，并对帧做裁剪（frame cropping）以去除字幕与台标——这意味着部分样本的最终宽高比会因裁剪而偏离原始比例，论文未说明裁剪后如何处理比例变化。[不确定]
【逐样本可查】metadata 中含 resolution 字段，使用者可自行统计完整分布。

### [Open-Sora 系列](../models/Open-Sora.md)

【Open-Sora 2.0】预处理即剔除宽高比落在 [1/3, 3] 之外的视频，并把片段长边限制在 ≤1080px、统一转 H.264。Figure 3 显示宽高比（高/宽）**多数集中在 0.5–0.75 区间，即以 16:9 横屏为主体**。推理支持 256px 与 768px 两档，覆盖 16:9、9:16、1:1、2.39:1。
【Open-Sora 1.x】采用显式**分桶（bucket）策略**：把分辨率、宽高比、帧数三维组合成预定义 bucket，每个 bucket 单独设置 batch size 以均衡 GPU 负载，从而在同一次训练中混合 144p~720p、任意宽高比、2~15 秒的样本。这是 Open-Sora 最被广泛复用的工程设计之一。
【Open-Sora Plan v1.3】视频训练固定为 93×352×640（约 16:9）；v1.5 图像侧做多分辨率训练，覆盖 (1,1)、(3,4)、(4,3)、(9,16)、(16,9) 五种宽高比并配合 Min-Max Token 策略，视频侧则固定 9:16 比例，最高 121×576×1024。
【数据来源侧的比例控制】Open-Sora Plan 显式做了横竖屏配比：Panda70M 21M 提供横屏，VIDAL 3M 提供竖屏（YouTube Shorts），是少见的显式竖屏数据补充策略。

### [Ovi](../models/Ovi.md) ⚠️

【筛选门槛】切分阶段要求片段分辨率严格大于 720×720 像素（论文原文「clips are greater than 720x720 pixel resolution」），低于该门槛直接丢弃。
【归一化策略】不做分辨率分桶，而是统一归一：打包（packing）前先去除视频中已有的黑边/margin，再在保持宽高比（maintaining aspect ratio）的前提下将帧缩放到固定像素总数 518400（= 720×720）。即约束的是「面积」而非「边长」，因而不同宽高比的样本能落到同一 token 数，天然支持 9:16、16:9、1:1 等多宽高比而无需分桶调度。
【Ovi 1.1】改为原生 960×960 分辨率数据训练（像素总数升至 921600），推理支持 960×960 及其等面积的各种宽高比。
【各宽高比的实际配比数字未公开】[不确定]。
【打包】最终视频按 24fps 抽帧后转为字节数组，音频转为原始 wave 字节，供训练侧高吞吐读取。

### [Script-a-Video](../models/Script-a-Video.md) ⚠️

论文完全未讨论分辨率与宽高比：未给出训练数据的分辨率门槛、分辨率分布、宽高比分布，未描述任何分桶（bucketing）或多分辨率训练策略，未说明生成模型的输出分辨率与帧率配置。
仅有的间接信息是生成基座 LTX-2 本身的分辨率能力，但论文未说明在其改造版本中实际使用的训练/推理分辨率档位。
评测指标中 Shot Boundary Deviation 以「帧」为单位报告（3.79 → 0.38 帧），说明存在固定帧率设定，但帧率数值未给出。[不确定]

### [Seedance 2.0 与 Seedance 1.5 pro](../models/Seedance_20_Seedance_15_pro.md) ⚠️

[不确定] 未披露分辨率/宽高比的分布统计与分桶（bucketing）策略。可参照 Seedance 1.0 的渐进式分辨率课程：先以 256px 文生图充分训练 → 256px 图文视频联合训练 → 提升至 640px → 最后提升到 24 fps。Seedance 1.0 在数据分布再平衡（Distribution Rebalancing）时明确把「分辨率」列为需要统计频次并做上/下采样的属性维度之一。Seedance 2.0 的原生输出分辨率为 480p 与 720p（报告特别指出其在 720p 下的 Arena Elo 已超过对手的 1080p 模型）。

### [SkyReels 系列](../models/SkyReels.md) ⚠️

【SkyReels-V2】宽高比通过「双轴分桶」中的 BAR 轴显式管理（时长×宽高比桶矩阵），并对 FPS 做标准化；分辨率维度靠渐进式课程管理：预训练阶段1为256p、阶段2升至360p、阶段3升至540p，后训练最终 SFT 提升到720p。基础质量过滤会剔除低分辨率源与低帧率源，并裁除黑边（black bar）与分屏内容。各分辨率/宽高比桶的具体配比未公布。
【SkyReels-V4】按阶段分配分辨率与数据量：Stage1/2/3 为256px（16fps），Stage4 混合256/480px（各1亿），Stage5 混合480/720/1080px（各5000万），Stage6 与 SFT 用720/1080px。生成上限1080p/32FPS。宽高比分桶策略未单独说明。各分辨率样本比例可从上述「每档等量」的写法间接推断为等量配比。[不确定]

### [Sora 2](../models/Sora_2.md) ⚠️

训练侧分布与分桶策略未披露。推理侧规格：Sora 2 标准版 720p，Sora 2 Pro 支持 1024p 与原生 1080p（1920x1080），同时支持竖屏（1080x1920）与横屏输出。前代 Sora 1 技术博客明确采用「native size training」——不做resize/crop/trim到固定尺寸，因而可原生采样任意宽高比，推测 Sora 2 延续该策略（可变分辨率/宽高比patch打包），但 Sora 2 官方从未确认，亦无任何分桶比例数字。[不确定]

### [Step-Video-T2V](../models/Step-Video-T2V.md)

【分辨率】作为课程主轴：图像阶段 256px → 视频阶段 192×320（192P）→ 544×992（540P）。最终发布模型输出 540P，未训练到 720P/1080P，这是其与同期混元/万相的显著取舍差异（团队把算力投在了 204 帧长序列与深度压缩 VAE 上）。
【宽高比】采用宽高比分桶（aspect-ratio bucketization），分为横屏（landscape）、竖屏（portrait）、方形（square）三组，与帧长分桶（1/68/136/204）组合形成二维 bucket 体系，支持多分辨率多宽高比混训。
【黑边处理】pipeline 中用 FFmpeg 检测黑边（black border）尺寸并据此裁剪，保证入桶前画面无 padding 边框。
各宽高比桶的具体占比未公布。

### [UniTalking](../models/UniTalking.md) ⚠️

论文完全未披露分辨率与宽高比信息：无分辨率准入门槛、无分辨率分布统计、无宽高比分桶或补边策略、无帧率说明。仅在视频过滤环节笼统提到剔除「整体视觉质量低」（low overall visual quality）的视频，但未说明该判定是否包含分辨率维度、用何模型、阈值几何。
可推断的间接线索：视频分支继承 Wan2.2-5B（TI2V-5B），该基座原生支持 720P（704×1280 / 1280×704）@24fps，且其 3D causal VAE 采用 16×16×4 的时空压缩率（空间 16 倍、时间 4 倍，压缩率高于 Wan2.1 的 8×8×4），因此训练与推理分辨率大概率在 720p 量级，宽高比覆盖竖屏与横屏。但这些全部来自基座规格推断，论文本身零披露。[不确定]
对比参照：UniVerse-1 明确给出「低于 1080p 剔除 + 码率-分辨率比低于 600 剔除」的双重硬门槛；UniTalking 在这一维度的披露粒度显著更低。

### [UniVerse-1](../models/UniVerse-1.md) ⚠️

【分辨率准入门槛】原始视频分辨率低于 1080p 直接剔除，这是一条相当激进的高门槛过滤——大量 720p 素材被排除在外，体现「宁少勿滥」的取向。同时施加码率-分辨率比（bitrate-to-resolution ratio）低于 600 剔除的约束，用于排除高分辨率但低码率的「虚高清」重压缩素材，这一指标在同类工作中较少见。
【例外】VGGSound 与 AudioSet 走简化处理路径，不受 1080p 与码率比约束（这两个数据集本身视觉质量低），转而在训练侧用 LQLS 损失策略隔离其低质量视觉信号的影响。
【帧率】统一为 25 fps。
【宽高比】论文未描述任何宽高比标准化、分桶或补边策略，也未给出宽高比分布。
【训练/推理分辨率】论文未披露实际训练与推理时的视频分辨率与帧数配置，模型卡与代码库亦未给出分辨率规格。受基座 Wan2.1-1.3B 的能力约束，推测在 480p 量级，但无官方数据支撑。[不确定]

### [Unison](../models/Unison.md) ⚠️

论文完全未讨论分辨率与宽高比：未给出分辨率准入门槛、未给出宽高比分布、未描述分桶（bucketing）策略、未提及补边或裁剪处理、未披露训练分辨率。
【唯一的确定信息】输出视频帧率为 25 FPS（推理配置）。
【为何可能真的不需要】这一空白有其结构性原因：Stage 2 联合训练时视频骨干 Wan2.2-5B 完全冻结，只训练音频分支与融合模块（双向交叉注意力与 layer normalization）。这意味着 Unison 根本不训练视觉生成能力，分辨率相关的数据策划自然失去必要性——视觉侧的分辨率处理、分桶策略实际上继承自 Wan2.2 的预训练，超出本论文范围。这是与 MOVA（三阶段 360p→720p 分辨率课程）、UniVerse-1（≥1080p 硬门槛）等需要真正训练视觉分支的工作的本质区别。
【可由上游数据集间接推断的分辨率状况】HDTF 为 720p~1080p 高清说话人视频，VFHQ 明确以「高保真（high-fidelity）」为选材标准，OpenHumanVid 亦以「高质量」为名，三者视觉素质较高；而 VGGSound 源自 YouTube 普通视频，视觉质量明显偏低（UniVerse-1 曾专门为其设计 LQLS 损失隔离策略）。Unison 未对 VGGSound 的低视觉质量作任何特殊处理说明——但因视频骨干冻结，低质视觉数据不会污染视觉生成能力，风险被架构选择天然规避。此为本条目推断。[不确定]

### [Veo 3 / Veo 3.1](../models/Veo_3_Veo_31.md) ⚠️

[不确定] 训练数据的分辨率/宽高比分布与分桶（bucketing）策略未披露。产品侧输出规格：Veo 3 支持 720p / 1080p，宽高比 16:9 与 9:16；Veo 3.1 进一步支持至 4K，同样覆盖 16:9 横屏与 9:16 竖屏，帧率 24fps。同时支持横竖两种比例说明训练数据包含多宽高比样本并很可能采用了分桶或多分辨率混合训练，但具体比例与实现方式无官方依据。

### [Vidu S1](../models/Vidu_S1.md) ⚠️

[不确定]。训练侧仅在预过滤阶段以「帧率、分辨率」作为技术门槛剔除低帧率/低分辨率视频，未公布具体阈值、分辨率分布或分桶（bucketing）策略。推理/输出侧为 540p（960×540）、25 FPS 标准、最高 42 FPS（RTX 5090 实测），未提及多宽高比支持。

### [主流视频预训练数据集合并调研：Panda-70M、InternVid、Koala…](../models/pretraining_datasets.md) ⚠️

**分辨率维度上七者高度趋同于720p，唯有 UltraVideo 与 OpenVidHD 例外**：
- **720p 一档（五家）**：Panda-70M（继承 HD-VILA-100M 的全720p；但注意**默认下载配置 download_size:360 实际只给360p**，不改配置拿不到720p，这是复现时的常见坑）；Koala-36M（720p）；MiraData（720p，实测样本1280×720@30fps）；LVD-2M（**论文从未把分辨率作为数据集属性说明**，因不托管视频；下载脚本默认 --resolution=720p，WebVid 分片文件名标 336，唯一出现的分辨率是处理参数——RAFT 光流在时间2fps、空间520×960 下计算）；InternVid（**85% 为720P**，其余15%为360P–512P，爬取时限定360P–720P）。
- **1080p**：OpenVid-1M 设**最低分辨率门槛 512×512**，OpenVidHD-0.4M 子集为 1080p（433K条）。宽高比与FPS分布论文均未披露。[不确定]
- **4K/8K（UltraVideo，唯一）**：short 集 4K **32,727** 条 + 8K **9,457** 条（8K占22.4%）；long 集 4K 12,277 + 8K 4,320。帧率只分「≤30 FPS」与「≥50 FPS」两桶（short：31,027 / 11,157；long：8,146 / 8,451），**31–49 之间是空的**。HF 数据集元信息显示帧尺寸 3,840–7,680 × 1,600–4,320，12–60 FPS，每clip 36–600帧。**保留原生分辨率与帧率不做转码**，作者以此论证可服务插帧/编解码研究。码率与编码格式全文未提及。
**分桶策略**：七者**均无训练时的分辨率/宽高比分桶（bucket）设计**——它们是数据集而非训练框架，分桶由下游使用者（如 Open-Sora）自行实现。UltraVideo 附带的 UltraWan 训练是固定尺寸的（1088×1920×81帧 / 2160×3840×29帧）。
**一个值得注意的缺口**：七者**没有一家给出宽高比分布统计**，也没有一家做竖屏内容的定向补充（对比 Open-Sora Plan 用 VIDAL 补竖屏），全部隐含以16:9横屏为主体。

## 类别/domain分布与配比策略（人物、动作、场景、风格等比例控制与概念均衡）

`domain_distribution` · 详细程度: detailed

### [Allegro](../models/Allegro.md)

论文在 Appendix A 给出基于 Tag2Text 粗粒度标签的类目分布统计（Fig.11）：人物（people）、物体（objects）、风景/自然景观（landscapes）三类构成标签的绝大多数，论文将此归因于「数据集自然构成」（natural dataset composition），即分布由来源语料决定，而非人为配比设计。
值得注意的是，Allegro 的「stratification（分层）」是按质量阈值分层（同一语料按 aesthetic / CLIP / 清晰度阈值切成 4 个由宽到严的子集，供 4 个训练阶段使用），而不是按内容 domain 做配比均衡。论文没有描述概念均衡（concept balancing）、长尾类目上采样、按人物/动作/场景/风格的比例控制，也没有类目重采样权重表。这是其 pipeline 相对 Movie Gen、Seedance 等工作的明显缺口。
内容风格上有一条隐含约束：只保留单镜头（single-shot）素材，因此不含多镜头叙事类 domain。

### [Apollo](../models/Apollo.md) ⚠️

论文对视觉 domain（人物、动作、场景、风格等）的分布与配比策略完全没有描述，也未提及概念均衡（concept balancing）机制。论文中唯一的「分布」概念是按音频类型划分的四类场景（单说话人语音 / 多说话人语音 / 歌唱 / 自然声），即 Apollo 的数据组织轴心是**音频类型而非视觉 domain**——这是本工作在数据设计上最显著的取向：它把「听到什么」而非「看到什么」作为数据切分的第一性维度（见 audio_category_distribution）。
可间接确认的配比意图来自训练策略而非数据统计：Stage II「专项后训练」会「adaptively rebalance data distributions across scenarios and tasks to strengthen underperforming capabilities while preserving overall competency」（依据评测指标自适应地在场景与任务间重新平衡数据分布，以补强表现不佳的能力同时不损伤整体能力），说明存在指标驱动的动态配比机制，但具体的场景清单、初始配比与调整后配比均未给出数字。[不确定]

### [CineDance / CineDance-1M](../models/CineDance.md) ⚠️

采用八维分类体系（taxonomy）保障多样性，但未公布各类目的定量占比：
【八个维度】Genre（题材类型）、Format（形式/载体）、Region（地域）、Modality（模态/表现形式）、Story Logic（叙事逻辑）、Era（年代）、Tone（情绪基调）、Audience（受众定位）。
【设计意图】该体系用于在采集与筛选阶段主动铺开覆盖面，避免语料集中于单一题材或单一年代，服务于「电影级叙事」这一核心场景。
【与评测的关系】CineBench 的 1000 条测试 prompt 也按 Theme/Style（主题风格）× Duration/Shot Count（时长与镜头数）× Difficulty（难度）三轴做分层抽样，其中主题风格轴与训练侧 taxonomy 相呼应。
【配比控制策略】论文未描述基于该 taxonomy 的显式配比控制、概念均衡或重采样机制，八维体系更像是事后的多样性刻画而非事前的采样约束。
【各维度下的类目清单与百分比】未公布。人物/动作/场景/风格等细粒度比例亦无数据。[不确定]

### [CogVideoX](../models/CogVideoX.md) ⚠️

[不确定] 论文完全没有披露训练数据的类别/domain 分布、概念聚类或配比均衡策略——这是 CogVideoX 数据工程相对 Movie Gen、Seedance 等最明显的缺失环节：没有概念聚类、没有长尾重采样、没有人物占比目标、没有动词/动作 taxonomy。
可间接推断的 domain 倾向来自「负面标签」的反向约束：
· 「Lecture Type（讲座/直播口播类，画面基本静止只有人在说话）」被整类剔除 → 训练集中「静态人物说话」题材被系统性压低，这也解释了模型在唇动/口播场景上的弱势。
· 「Text Dominated（大量可见文字或以文本内容为主）」被整类剔除 → 图文类、字幕类、演示文稿类内容被排除。
· 「Noisy Screenshots（手机/电脑屏幕直录）」被整类剔除 → 录屏、游戏画面、UI 演示类被排除。
· 「Editing（含明显人工剪辑与特效）」被剔除 → 高度剪辑的 MV、预告片风格被排除。
因此数据集实际偏向「未经重剪辑的、有真实连续运动的实拍单镜头素材」。
· caption 侧的 GPT-4 摘要 prompt 显式要求覆盖「objects, scenery, animals, characters, and camera movements」五类要素，可视为团队关注的内容维度，但并未据此做配比控制。
· 评测端在 VBench 上强调 Human Action、Scene、Dynamic Degree、Multiple Objects、Appearance Style 等维度，属评测选择而非数据配比。

### [Cosmos-Predict2.5](../models/Cosmos-Predict25.md) ⚠️

domain 体系是该 pipeline 的核心设计之一，且分为「通用 26 类 taxonomy」与「Physical AI 五大专项域」两层：
【第一层 通用内容类型分类】过滤末端由内部训练的 content type classifier 为每条 clip 打一个语义标签，取自自建的 26 类视频类型 taxonomy（a custom-built taxonomy of 26 video types）。该标签是 sharding 的首要轴，用于「fine-grained domain balancing」（细粒度域均衡）与 curriculum-based training。26 类的具体类目名与各类占比均未公布。
【分布对齐约束】在此阶段显式剔除「physically unrealistic phenomena」——video games（游戏画面）、synthetic visual patterns（合成视觉图案）、animations（动画）、cartoons（卡通），理由是「maintain alignment with the physical world distributions」（保持与真实物理世界分布的对齐）。这是一条以物理真实性为准绳的分布裁剪原则，区别于一般视频生成模型追求风格多样性的做法。
【第二层 Physical AI 五大专项域】为增强 Physical AI 能力，另设五条领域专用 pipeline，产出并入通用预训练数据：Robotics（机器人）、Autonomous Driving（自动驾驶）、Smart Spaces（智能空间：仓库/工厂/工地）、Human Dynamics（人体动力学）、Physics（物理现象）。这些 pipeline 与通用流程结构相同，但有两处关键差异：过滤上省略昂贵的 VLM filter，改用「领域特定的过滤器子集 + 调整过的超参数」；打标上改用更大参数量的 VLM 并配领域定制 prompt。
【自动驾驶的目标分布采样】驾驶数据不是随机抽样，而是「sampled from a large-scale corpus to align with a target distribution of diverse driving attributes」——按预设目标分布在九个属性轴上均衡采样：地理区域（美国、欧洲）、交通密度（稀疏/拥堵）、自车速度（城市道路/高速）、自车加速度（匀速/急加速）、自车机动动作（缓弯/急转）、道路类型（城市/乡村）、罕见道路结构（隧道、收费站）、能见度（晴朗/雾天）、天气（干燥/雪天）、光照（白天/夜间）。这是全文最具体的配比控制策略。
【物理域的 taxonomy】先定义一套「可视觉观测的物理现象」分类体系，覆盖经典力学与流体力学等核心领域（如玻璃碎裂 shattering glass、滚球碰撞 colliding rolling balls、水流 flowing water），再据此定向采集能凸显这些动态属性的视频。
【Human Dynamics 的量化准入】人体域用硬性数值规则控制画面构成：人物需出现在超过 40% 的帧中；任一帧中可见人数不超过 8 人；至少一人占据画面面积 3% 以上。
【后训练五域】SFT 侧另有一套独立划分（object permanence / high motion / complex scenes / driving / robotic manipulation），由 InternVideo2 embedding 上的多头分类器产出，见 post_training_data。
[不确定：26 类 taxonomy 的具体类目与各类占比数值]

### [Data-Juicer 2.0](../models/Data-Juicer.md) ⚠️

Data-Juicer 在类别/domain 配比上的思路与生成模型团队根本不同：它不预设一套 domain 类目体系，而是提供「让 domain 分布可度量、可切分、可搜索」的机制，把配比决策交给 Sandbox 的反馈闭环去经验性确定。
【domain 标注能力】
  · video_tagging_from_frames_mapper —— 从抽帧生成视频内容标签（基于 RAM 等图像打标模型），可产出开放词表的语义标签，是构建 domain 分布画像的主要手段。
  · video_tagging_from_audio_mapper —— 基于 Audio Spectrogram Transformer 从音轨生成标签，提供听觉侧的 domain 信息（与视觉标签互补）。
  · video_tagging_from_frames_filter —— 按标签保留/剔除样本，即基于 domain 的定向筛选。
  · video_object_segmenting_mapper —— 基于 YOLOE + SAM2 的文本引导语义分割，可给出物体级的 domain 证据。
【人物中心 domain 的专项支持】v1.5.4 一次性新增9个人物中心视频理解算子，构成一条较完整的人物 domain 处理链：video_human_tracks_extraction_mapper（提取人脸与人体框轨迹）、video_face_ratio_filter（人脸占画面比例过滤，可用于筛选特写/中景/远景）、video_active_speaker_detect_mapper（主动说话人检测）、video_captioning_from_human_tracks_mapper（基于人物轨迹生成描述）、video_captioning_face_attribute_emotion_mapper（人脸属性与情绪描述）、video_human_tracks_face_demographic_mapper（人口统计属性）、video_whole_body_pose_estimation_mapper（身体/手/脚/面部2D全身关键点）。这条链直接对应视频生成中「人物、动作」这一最重要的 domain。
【配比策略——经验搜索而非先验设计】Sandbox 的 Probe-Analyze-Refine 工作流本质上是一种数据配比的自动搜索：对每个候选算子 i，按其统计量把数据池均分为低/中/高三个等大子池 P_i,low / P_i,middle / P_i,high，外加一个随机采样的对照池，分别训练参考模型并用 VBench 评测，据此对算子及其取值区间排序；再把排名靠前的若干算子组合成 2^n−1 个交叉数据池（金字塔结构，越上层质量越高但样本越少），继续搜索最优组合。这套流程把「哪个维度上该保留哪一段分布」变成了可被模型反馈直接回答的实验问题，而不是靠人工设定比例。
【DJ-Cookbook 中的配比相关配方】维护有「基于数据难度的课程学习」（curriculum learning）配方与对比学习配方，属于 domain/难度配比调度的现成模板。
[不确定] DJ 官方未发布任何视频生成的推荐 domain 类目体系（如人物/动作/场景/风格的目标比例），也未在 T2V 案例中报告最终数据池的类别分布统计；概念均衡（concept balancing）、长尾类别重采样等策略在文档中未见专门算子支持。

### [Foley-Omni](../models/Foley-Omni.md) ⚠️

论文没有以「视觉domain/类别」维度做配比控制，而是以「任务组」为一级配比单位——这是 Foley-Omni 数据组织方式与文生视频模型的根本差异。
六个任务组的小时数配比（总4.9k小时）：VisualTTS 1,980h（40.4%）> TTS 1,253h（25.6%）> TTA 912h（18.6%）> V2A 403h（8.2%）> V2ST 216h（4.4%）> TTM 139h（2.8%）。可见语音相关（VisualTTS + TTS）合计占66%，音效相关（TTA + V2A）占26.8%，音乐相关（TTM）仅2.8%——严重向语音倾斜，这解释了模型在 WER 指标上的突出优势（7.59，低于两个级联基线的10.57与37.84），同时也可能是音乐生成能力相对薄弱的隐忧。
视觉内容domain可从数据集来源间接推断：GRID、LRS2、Chem 为受控/半受控的说话人正面视频（实验室朗读、BBC新闻、化学讲座），SpeakerVid、TalkVid 为大规模野外说话人视频，VGGSound 为10秒野外事件视频（310类声源），Kling-Foley 为foley专用视频。整体domain高度集中在「有人说话的场景」+「有明确声源事件的场景」两类，缺乏纯风景、抽象艺术、动画等视觉domain。
【运动强度作为隐式domain控制】motion score 限定在 [0.1, 3.2] 区间——下界剔除近静止画面（无视觉事件可对齐，V2A监督信号空洞），上界剔除剧烈运动/快速剪辑（光流估计不可靠、声画对应关系混乱），实际上是通过运动强度把数据domain收窄到「中等动态、单一连续场景」。
[不确定] 未给出显式的视觉类别标签分布统计、未提及概念均衡（concept balancing）或长尾类别重采样策略、未给出人物/动作/场景/风格的比例控制。

### [Goku](../models/Goku.md)

★这是 Goku 最具特色、也是本次调研最高价值的部分：数据分布均衡（Data Distribution Balancing）被显式列为五阶段流水线的第五个独立阶段，而非附属于过滤环节。绝大多数同期工作（HunyuanVideo、Open-Sora、CogVideoX 等）仅把类别配比作为过滤的副产品或完全不提，Goku 将其提升为与「采集/切分/过滤/打标」并列的一级阶段，是对「数据分布」这一维度最显式的建模之一。
【类目体系】使用内部自研视频分类模型（internal video classification model），对每个片段均匀采样 4 帧关键帧打标签，输出两级类目：9 个一级大类 + 86 个二级子类。
  - 一级大类示例：human（人物）、scenery（风景）、animals（动物）、food（食物）、urban life（城市生活）等，其中人物、风景、食物、城市生活、动物为占比最高的几类。
  - 二级子类示例：half-selfie（半身自拍）、kid（儿童）、dinner（晚餐）、wedding（婚礼）等，粒度相当细。
【配比策略】论文明确表述为「emphasizes human-related content while ensuring equitable representation across subcategories」——即在总体上刻意加权人物相关内容（因为人物是视频生成最高频、最受关注的需求，也是最容易穿帮的难点），同时保证 86 个子类之间的表征相对均衡。具体手段有二：
  (1) 对过度表征（overrepresented）的类别做降采样（down-sampling）；
  (2) 对表征不足（underrepresented）的类别做数据增强与过采样（augmentation and oversampling）。
【可视化】论文 Figure 3 以 (a) 一级类目、(b) 二级子类两张分布图展示均衡后的语义分布。
【评价】该阶段体现了「概念均衡（concept balancing）」思路：过滤只保证单条样本质量，而分布均衡保证整个数据集在概念空间上的覆盖度与不偏斜，直接对应 VBench 中 human action、object class、scene 等多类目指标的表现（Goku 在 human action 得 79.48、scene 得 85.72，均显著高于同期开源模型）。
【未披露】各类目的具体百分比数值（仅有图，无表）、降采样/过采样的具体倍率、分类模型的架构与准确率、是否使用 VLM 而非专用分类器。

### [HunyuanVideo-Foley](../models/HunyuanVideo-Foley.md) ⚠️

【核心机制：用分类标签做分布管理，但不公布分布数字】这是本工作数据侧最值得记录、也最语焉不详的一环。论文明确表述：用「语音-音乐检测（speech-music detection）与音频分类模型」为每个保留片段打上类别标签（categorical tags），目的是「实现类别分布的有效管理」（enabling effective management of category distribution），并称由此「确保训练数据集中各类别的均衡表征」（ensuring balanced representation in the training dataset）。
【关键缺失】论文声明了做分布管理，却完全没有给出：类目体系是什么（几类？类目名称？基于 AudioSet 527 类本体还是自定义体系？）、各类目的实际占比、目标配比是什么、通过何种手段实现均衡（上采样低频类目？下采样高频类目？还是仅按阈值截断超量类目？）。因此这是一个「有机制、无数据」的字段——读者只能确认团队意识到了长尾与类别失衡问题并采取了措施，但无法知晓措施的具体形态与效果。[不确定]
【为何这一环重要】音效数据的天然分布极度长尾：说话声、背景音乐、交通噪声占据绝大部分时长，而真正的 Foley 事件音（脚步、翻书、倒水、刀切）占比很低。若不做主动干预，10 万小时数据训出的模型会强烈偏向生成环境底噪与音乐而非精确的事件音。论文把「类别分布管理」列为 pipeline 的最后一环（在所有质量过滤之后），说明它是最终决定数据构成的闸门，其重要性与信息披露的稀缺程度不成比例。
【间接证据：泛化能力覆盖面】官方与第三方报道均强调模型对「人物、动物、自然风光、卡通动画」等多种视频类型的泛化能力，这可视为类目覆盖广度的侧面印证，但属定性描述。
【与评测类目的关系】评测使用的 Kling-Audio-Eval 本身带有类目体系（快手可灵团队构建），但论文未说明训练数据的类目管理是否参照该体系设计。

### [HunyuanVideo](../models/HunyuanVideo.md) ⚠️

这是原版 HunyuanVideo 最有方法论价值的一点，也是同期少数显式做「概念均衡」的开源工作：
【原版】(1) 原始数据池按 domain 划分，明确列举人物（people）、动物（animals）、植物（plants）、风景（landscapes）、载具（vehicles）、物体（objects）、建筑（buildings）、动画（animation）等类别，覆盖广度作为数据构建目标；(2) 用自研内部 VideoCLIP 模型对片段抽 embedding，先按余弦距离做语义去重，再对 embedding 跑 k-means 得到约 1 万（~10K）个概念中心（concept centroids），基于这些中心做 concept resampling and balancing（概念重采样与均衡）——即用聚类中心作为概念代理，对过密概念下采样、对稀疏概念上采样，抑制长尾失衡。这套「VideoCLIP embedding + 10K 聚类中心 + 重采样」是原版数据处理的标志性设计。(3) 各 domain 的最终占比数字未公布。
【1.5】报告未重复描述概念均衡机制，只强调渠道多样性覆盖「内容、拍摄手法、镜头运动、风格、场景」，是否延续 10K 概念中心重采样未说明。1.5 在 RLHF 阶段有类目化痕迹：I2V RLHF prompt 覆盖「100+ 类别」，T2V DPO 使用在运动/场景/主体三个维度上做过平衡的 prompt 集。[不确定]

### [InstructAV2AV](../models/InstructAV2AV.md) ⚠️

本工作的 domain 组织维度不是「视觉内容类别」（人物/动作/场景/风格），而是「编辑操作类型」——这是编辑类数据集与生成类数据集的根本差异，配比控制的对象从「拍的是什么」变成了「改的是什么」。
【论文层面的四类任务taxonomy】
  (a) 身份保持的语音修改（identity-preserving speech modification）：改台词内容但保持说话人身份/音色/外貌；
  (b) 音视频实例编辑（AV instance editing）：改变画面中人或物的外观特征，同时改其发声；
  (c) 实例插入（AV instance insertion）：新增视觉元素及其对应声音；
  (d) 实例移除（AV instance removal）：删除视觉元素及其对应声音。
  四类的共同结构是「视觉目标 + 其绑定的声音」同步变更，非目标区域与环境声保持不变。
【HuggingFace 实际发布的五类子集命名（比论文更细）】add_and_remove（增删，对应 c+d 合并）、clone_id（身份克隆，仅换人不换声）、clone_voice（音色克隆，仅换声不换人）、clone_id_voice（身份+音色同时克隆）、general_editing（通用编辑，对应 b）。这个五分法暴露了论文未展开的一个重要设计：语音类编辑被进一步正交分解为「视觉身份」与「听觉音色」两个可独立操控的因子，并为三种组合（只改一个 / 只改另一个 / 两个都改）分别构造了数据。这种正交分解式的数据构造是合成数据相对真实采集的独特优势——现实中几乎不可能采集到「同一段视频里换了脸但保持原声」的真实素材，只能靠受控合成制造，且能精确控制哪些因子变、哪些不变，从而给模型提供干净的解耦监督信号。
【视觉内容 domain 的间接推断】由来源反推：MovieBench + Condensed Movies + Short-Films-20K 三者贡献叙事性影视场景（人物对白、室内外场景、多样光照与镜头语言），VGGSound 贡献野外声源事件（动物、乐器、机械、自然现象等310类），YouTube 贡献开放域素材。整体偏向「有人说话的影视场景」与「有明确发声物体的事件场景」两类，与 Foley-Omni 的 domain 集中方向类似。
【隐式的 domain 收窄机制】CoTracker3（Karaev et al.）做网格化点跟踪，剔除平均运动幅度低于阈值的片段——这一步把纯静态、近静止的场景排除在外，把 domain 收窄到「有可见运动」的片段。原因是编辑任务需要目标可被 mask 跟踪且有形变/位移，静止画面既缺乏编辑价值也让时序一致性无从检验。
[不确定] 未给出任何类别标签的定量分布表：五类子集各自的样本数、视觉内容类别（人物/动物/物体/场景）配比、语音类与非语音类样本的比例、概念均衡或长尾重采样策略，论文与数据卡均未披露。CoTracker3 的运动阈值与 LAION 美学阈值的具体数值也未给出。

### [2026 其他音视频联合生成](../models/JAVG_2026_misc.md) ⚠️

本维度上 ALIVE 与 NAVA 各建立了一套显式的类目体系，是七项中最有价值的部分；其余五项基本无配比控制。
【ALIVE —— 三级标签体系 + 主动配比调整（本批工作中最完整的 domain 控制设计）】
(1) 顶层二分：先做「speaking（说话）」与「non-speaking（非说话）」两大核心场景的划分——原文「First, we make a top-level distinction between core scenarios: 'speaking' and 'non-speaking' scenario.」这一二分直接决定了模型在「对白驱动」与「音效驱动」两类能力上的配比。
(2) 三级层次标签：在此之上建立 three-level hierarchy，Level 1 为九大领域：Animals（动物）、Home Sounds（家居声）、Entertainment（娱乐）、Environment（环境）、Food（食物）、Nature（自然）、Sound Effects（音效）、Vehicles（载具）、Sports（运动）。注意这九类是「视听联合类目」而非纯视觉类目——Home Sounds 与 Sound Effects 本身就是声学类目，说明该体系从设计之初就是为音视频联合建模服务的。
(3) 音视关键词绑定：每个 Level 3 视觉标签都通过「joint audio-visual keyword retrieval」（联合音视关键词检索）配上对应的音频关键词，形成视觉概念与声音概念的成对索引——这是把 domain 分布与 audio 分布做联合而非独立控制的关键机制。
(4) 主动配比调整：原文「Guided by prior knowledge, such as concept frequency and projected application scenarios, we then adjust the data proportions for each category.」——依据概念频次与预期应用场景两条先验主动调整各类目占比。Continue-training 阶段用的 4.3M「balanced samples」即为该配比调整的产物。具体百分比未公开[不确定]。
【NAVA —— 五类音频驱动的内容分类】用 YAMNet 音频分类 + omni-modal tagger 把片段划为五类：single-speaker speech（单说话人语音）、multi-speaker speech（多说话人语音）、ambient sound（环境音）、music（音乐）、singing（歌唱）。注意其分类轴是声学的而非视觉的，与 ALIVE 的视听双轴不同。各类占比未公开[不确定]。此外还有 VLM-based filtering and tagging 保留视觉质量清晰的片段，标签体系未详述。
【CCL —— 隐式 domain 画像】未做显式配比，但其数据来源「interviews, short dramas, and films」本身构成了一个窄而深的 domain：以人为中心、以对白为主的叙事内容。这解释了它为何能用 4M 数据达到 SOTA——牺牲广度换深度。其多任务训练概率分布（见 stage_data_mixture）中「联合生成 0.6」的高占比也说明其目标 domain 高度聚焦。
【OmniCustom —— 单一 domain】完全聚焦「单人说话视频（single-person video clips）」，无 domain 多样性诉求，其 100 万片段全部为同一类目。评测集刻意控制性别比为「1:1」——这是七项中唯一显式的人口统计学配比控制。
【StreamChar —— 单一 domain】以人为中心的角色说话视频，三个上游数据集均为 human-centric，无跨 domain 配比。
【Baton】未提及任何 domain 配比策略；其数据来源（OpenHuman-Vid 人体 + AudioCaps/WavCaps 通用音频 + 互联网视频）的混合比例未公开[不确定]。
【ITS-JAVG】无训练数据；但其评测基准 JavisBench-mini 本身带有多类目结构（继承自 JavisBench 的场景/事件分类体系），用于检验各验证器在不同类目上的表现差异[类目细节不确定]。

### [音视频联合生成基线合集](../models/JavisDiT_baselines.md) ⚠️

domain 覆盖面从「单域」到「双域配比」再到「通用」，是本合集演进最明显的维度之一：
【MM-Diffusion —— 单域且类目明确】Landscape 覆盖 9 类自然场景，且类目完整披露：爆炸（explosion）、火焰噼啪（fire cracking）、下雨（raining）、水花飞溅（splashing water）、挤压水声（squishing water）、雷声（thunder）、水下咕噜声（underwater burbling）、瀑布轰鸣（waterfall burbling）、风声（wind noise）——全部是「视觉事件与声音强因果绑定」的自然现象，是刻意为验证音画对齐而选的域。AIST++ 则是单一街舞域（人体舞蹈动作 ↔ 音乐节拍）。两个数据集分别代表「环境音对齐」与「节奏/动作对齐」两种同步类型，这一「双数据集互补」的设定被后续 AV-DiT 等工作完整沿用为标准评测配置。
【AV-DiT】完全沿用上述两域，无扩展。
【JavisDiT / JavisDiT++ —— 显式构建了类目分类体系】其最大贡献之一就是把 domain 分布问题显式化：JavisBench 建立了五个评测维度、共 19 个场景类目的分类体系（taxonomy），五个维度为：事件场景（Event Scenario）、视频风格（Video Style）、声音类型（Sound Type）、空间构成（Spatial Composition）、时间构成（Temporal Composition）。作者强调「超过 50% 的视频属于高度复杂与有挑战性的场景」「75% 的样本包含多个发声事件（multiple sounding events）」——这是对早期基线「单一发声源、单一事件」设定的直接批判与超越。训练侧则通过 TAVGBench 的通用 YouTube 分布获得 domain 广度，但训练数据本身的类目占比未公开[不确定]。
【Harmony —— 显式的 1:1 双域配比】把数据明确二分为「人类语音」与「环境音」两大 domain，并在阶段一与阶段三训练中都严格采用 1:1 混合比例。Harmony-Bench 同样按此切分为环境音-视频、语音-视频、复杂场景（环境音+语音共现）三档，构成「训练配比 ↔ 评测类目」的直接对应。这是本合集中唯一给出明确 domain 配比数字的工作。
【UniAVGen —— 单域聚焦真人】训练数据为「真人音视频」，domain 高度集中于人物说话/表演场景，配合 Face-Aware Modulation 人脸感知模块，属于刻意的窄域设计；评测测试集 100 条按「50% 真实图像 / 50% AIGC 与动漫图像」二分，说明其在图像风格维度上做了显式配比（真实 vs 二次元各半），但内容 domain 层面未做细分[不确定]。
【共性缺口】除 Harmony 的 1:1 与 UniAVGen 的 50/50 外，均未披露人物/动作/场景/风格的细粒度比例控制或概念均衡（concept balancing）策略[不确定]。

### [可灵 3.0 / 可灵视频 3.0 Omni](../models/Kling_30_Omni.md) ⚠️

[不确定] 未公布类别/domain配比数字。定性上，Kling-Omni 报告称数据收集追求“广泛场景覆盖”，横跨多样主体与风格变体；并在多模态对齐环节对“以人为中心”的任务做专门的角色身份一致性校验，说明人物类数据是被单独建模与配比的重点子域。产品能力侧重（AI导演分镜、多角色对白、影视级质感、物理规律与惯性/碰撞）反映出训练配比明显偏向：人物表演与对话、电影化运镜、物理交互场景。KlingAvatar 2.0 明确按中文语音/英文语音/演唱三类构造评测子集，提示语音类数据按用途细分配比。[不确定：具体比例控制与概念均衡（concept balancing）方法]

### [LTX-2](../models/LTX-2.md) ⚠️

未披露任何类别/domain 配比数字，也未说明概念均衡策略。仅有一句定性说明：所选子集「提供了视觉与听觉内容的平衡分布（a balanced distribution of visual and auditory content）」，使得 caption 能同时充分覆盖图像域与听觉域信息。间接线索：(1) LTX-Video 论文 Fig.13 给出 caption 词云，可粗略反映概念分布，但无数值；(2) 训练中混入图像数据集专门用于补充「视频数据中不常见的概念」，说明团队关注概念覆盖；(3) 授权源为 Shutterstock/Getty 的商业素材库，其分布天然偏向专业拍摄的通用素材而非 UGC。局限性一节承认「训练数据中代表性不足的语言与方言」会导致效果下降，间接说明存在长尾分布问题且未做专门均衡。[不确定]

### [LongCat-Video](../models/LongCat-Video.md) ⚠️

这是 LongCat-Video 数据工作中相对最有特色的一环，采用「caption 文本嵌入聚类 + LLM 语义命名」的无监督类目体系，而非人工预定义标签体系。具体做法：对全部视频 caption 做 text embedding，在嵌入空间做聚类，再用 LLM 对每个簇做类别归纳与命名，得到内容类型标签（报告举例：personal interactions 人际互动、artistic performances 艺术表演、natural landscapes 自然风光 等）。该类目体系的用途有三层：(1) 观测数据分布，识别过密/过稀类目；(2) 做「targeted data supplementation or rebalancing」——针对性补数据或重平衡配比；(3) 支撑「dynamic and precise allocation of data subsets tailored to specific requirements and objectives of different training phases」，即不同训练阶段动态、精确地调配数据子集。
更进一步，SFT 阶段把这套嵌入空间直接用作采样权重：样本被「selected inversely proportional to their density in the caption embedding space」（按其在 caption 嵌入空间中的密度的倒数采样），这是一种显式的概念均衡（concept balancing）机制——高密度的常见概念被降采样，长尾稀有概念被相对提升，从而让精选集的语义覆盖更均匀。
此外还有两类被单独强化的 domain：相机运动（camera motion）与视觉风格（visual style），SFT 阶段专门引入对应的专项数据集以增强这两个维度的指令跟随。
未披露的是：聚类簇数量、各类目的具体占比数字、重平衡后的目标配比。[不确定：各类目具体比例数值]

### [MOVA](../models/MOVA.md) ⚠️

论文对训练数据的 domain 覆盖只给出定性列举，未给出任何比例数字，也未描述概念均衡（concept balancing）机制：
- 视频形态：电影（movies）、vlog、动画（animations）、中文剧集（Chinese drama）、卡通（cartoon）、YouTube 内容。
- 主题域：教育（education）、体育（sports）、美妆（beauty）、新闻（news）、访谈（interviews）等，论文称其提供了“泛化到复杂真实场景所需的分布多样性”。
- 唯一可确认的强配比倾向是“以人说话为中心”：Phase 1 数据源中 SpeakerVid-5M 与 OpenHumanVid 均为人物/说话人中心数据集，且最终训练只保留语音片段（见 audio_category_distribution），说明人物-对白类数据在配比中占绝对主导，这与模型主打多语种唇同步的定位一致。
- 论文中唯一带百分比的类目饼图（Figure 6a，含 “others 2.3%”）描述的是自建评测基准的样本类别分布，而非训练数据分布，不可混用。
训练数据的 domain 配比数字属信息空白。[不确定]

### [合并条目：① Mochi 1](../models/Mochi_MAGI_Motif.md) ⚠️

这是三者差异最大的维度，恰好构成一条「无 → 动态自适应 → 显式类目均衡」的演进线：
① Mochi 1：无任何披露。既未给类目统计，也未提及概念均衡或配比控制。[不确定]
② MAGI-1：不做静态配比表，而是提出「动态分布调整（Dynamic Distribution Adjustment）」——论文明确指出「合适的数据分布对训练高性能模型至关重要，但事先确定最优分布极其困难」，并给出实际观察：「训练过程中我们发现风景类场景对模型而言相对容易学习，而人物表情则显著更难」，这类洞察事前无法预判。因此其做法是在训练全程持续监控模型在各语义概念上的表现，据评测结果自适应地调高欠拟合子集的采样比例，以针对性强化模型短板。这是一种「以评测反馈闭环驱动的在线数据配比」，比静态配比表更贴近课程学习的本意，但论文未公开任何类目清单、初始配比或调整幅度。同时 MAGI-1 还提到一项任务级配比：因产品场景以图生视频为主，「训练中分配了更大比例的 I2V 任务」。[部分不确定：类目清单与配比数值]
③ Motif-Video 2B：做显式的类目均衡，且把它接在 caption 元数据上形成闭环。VLM 在同一次前向中产出 subject 与 style 结构化标签，这两个标签「drive domain balancing for the 720p stage and SFT（驱动 720p 阶段与 SFT 的领域均衡）」。更关键的是 SFT 语料的组装方式是迭代式补短板：「我们在最新 checkpoint 上跑中间评测，识别出生成质量最弱的 subject 类目，然后针对性地补充这些类目的片段」。Fig.8 给出最终结果——图像侧 People（人物）占主导，反映以角色为中心的使用场景；视频侧则向 Transportation（交通工具）、Sports（运动）、Animals（动物）倾斜，因为这三类涉及剧烈动态、在中间评测中被识别为弱项。这与 MAGI-1 的动态调整思路同构，但 Motif 把它落到了可读的类目分布图上，并且额外用 action=Dynamic 标签作为 720p SFT 的动态运动准入条件。

### [Movie Gen](../models/Movie_Gen.md)

预训练原始池覆盖人类、自然、动物、物体等domain。概念均衡采用两步：①用视频-文本联合embedding模型提取语义embedding，聚类得到细粒度概念簇；②合并重复簇后，按簇大小的倒平方根（1/sqrt(cluster size)，沿用 Mahajan et al. 2018 的做法）从每个簇采样，从而压制头部主导概念、抬升长尾——该步在漏斗中把数据从1.15%降到0.94%。
人物是重点倾斜domain：最终高分辨率训练集中至少60%的视频包含人。为此专门建立了一套包含600个人类动词与表情（human verbs and expressions）的taxonomy，用该taxonomy做zero-shot 文本→视频检索来定向挑选含人视频，并在概念重采样阶段刻意保留（preserve）这些人物视频的频次，防止被均衡策略稀释。
SFT阶段的概念平衡更精细：先用同一套600动词taxonomy做 text k-NN 从候选池检索每个概念的视频，再人工为每个概念挑几条视觉上出彩的种子视频，用这些种子做 video k-NN，最终得到一个概念平衡且规模足够小、可全量人工审核的子集；k-NN 使用视频-文本联合embedding模型的视频与文本embedding。
评测端对应的概念分布为人类活动、动物、自然与风景、物理现象、非常规主体与非常规动作五类。

### [NVIDIA NeMo Curator](../models/NeMo_Curator.md)

这是本条目最有参考价值的字段之一——Cosmos WFM 通过 NeMo Curator/Cosmos-Curate 实现了显式的、基于分类器的 domain 配比控制，是公开资料中少见的「数据配比可量化」的工业案例。
【实现机制】在 InternVideo2 视频嵌入之上训练一个轻量 MLP 分类器，按预定义的视频类型 taxonomy 对每个 clip 打类别标签，然后据此做过滤与重采样。
【排除类别】抽象图案（abstract patterns）、电子游戏画面（video game footage）、动画（animation）——这三类被判定为不利于学习真实世界物理规律而整体剔除。
【重采样策略】上采样（upsample）人类动作与人-物交互类目；下采样（downsample）风景类目——因为风景视频虽然画质高但动态信息与物理交互信息稀薄，天然过量。
【最终九大类配比（Cosmos WFM 论文公布）】自然动态（nature dynamics）20%、手部动作/物体操控（hand motion & object manipulation）16%、空间感知与导航（spatial awareness & navigation）16%、驾驶（driving）11%、人体动作/活动（human motion & activity）10%、第一人称视角（first-person POV）8%、动态镜头（dynamic camera）8%、其他（other）7%、合成渲染（synthetically rendered）4%。
【说明】这套 taxonomy 服务于 Physical AI / 世界模型的目标（学习物理规律），与通用文生视频模型（重人物、影视感、风格多样性）的配比目标并不相同，迁移时需重新定义类目体系。NeMo Curator 开源版本中未内置该 taxonomy 分类器权重，需用户自行训练。

### [OmniHuman 数据集 + OHBench](../models/OmniHuman.md) ⚠️

domain 多样性是本数据集的首要立意——论文将「全局场景与相机多样性受限」列为现有数据集的第一条结构性缺陷，整个数据集即为此设计：
【场景类目（8 类）】居家空间（home spaces）、办公室（offices）、自然环境（natural environments）、城市/乡村景观（urban/rural landscapes）、演出场馆（performance venues）、零售场所（retail）、工业区（industrial areas）等八类场景类型。
【内容类目（8 类）】脱口秀（talk shows）、教育（educational）、烹饪/美食（cooking）、体育（sports）、音乐（music）、游戏（gaming）、测评（reviews）、影视剧（film/TV）八类内容类型。
【人物构型维度（3 类）】单人（single-person）、双人（two-person）、人-物交互（person-object）。这是本数据集区别于所有前作的核心维度——论文将「交互建模稀疏（人-人与人-物两方面）」列为第二条结构性缺陷，并专门为双人交互标注了交互标签、说话人 ID 与情绪，为人-物交互标注了对象信息。Table 1 显示，此前所有对照数据集（ActivityNet、TikTok-v4、OpenHumanVid、CelebV-HQ、VoxCeleb2、HDTF、SpeakerVid-5M）在「人-物交互」与「对象标注」两列上全部为空，OmniHuman 是唯一同时覆盖的。
【相机与镜头维度】标注涵盖 shot scale（景别）与 camera motion（相机运动），Table 1 中仅 OpenHumanVid 与 SpeakerVid-5M 同时具备这两项。评测中发现「模型在从特写切换到远景的镜头转换上性能下降」，说明景别标注确实支撑了细粒度诊断。
【配比策略：未披露】上述 8+8+3 个类目在 100 万条中的具体占比，除单人/双人比例见于 Figure 3(b) 外，均无数值。也未描述任何概念均衡（concept balancing）、重采样、长尾抑制或配额控制机制——即论文证明了「覆盖了这些类目」，但未证明「各类目分布是均衡的」。考虑到 YouTube 上脱口秀/测评类内容远多于工业区场景，实际分布极可能长尾严重。[不确定]
【与评测体系的呼应】OHBench 的 509 条评测视频按 331 单人 / 128 双人 / 50 人-物 的比例构成，与数据集的三类人物构型一一对应，形成训练分布与评测类目的显式对齐（详见 benchmark_taxonomy_alignment）。

### [Open-Sora 系列](../models/Open-Sora.md) ⚠️

两个项目**均未做显式的类别/domain 配比控制与概念均衡**，这是其数据策略相对工业界模型（Seedance、Movie Gen 等有精细 domain taxonomy 与配比表）最明显的短板。
可识别的隐含 domain 结构：
- Open-Sora 1.2 通过混合不同来源数据间接实现 domain 多样性：Webvid/Panda-70M 提供通用网络视频（YouTube 长尾），MiraData 专门补充游戏画面与城市漫游长镜头，Vript 补充密集标注的电影化内容，Inter4K 补充 4K 高清素材，LAION/Unsplash 补充静态高美学图像。这是「按数据集功能分工」而非「按语义类目配比」的思路。
- Open-Sora Plan 从 LAION-5B 中专门筛出 16 万张高质量人像图，用于强化人物生成能力，是唯一一处显式的类目定向补充；VIDAL 3M 竖屏 Shorts 则隐含大量人物口播/生活类内容。
- 两者都未公布人物/动作/场景/风格的比例数字，未做长尾概念均衡（concept balancing），未做语义聚类后的重采样。Open-Sora 2.0 报告仅按「美学分/时长/宽高比/caption 长度」四个低层维度给出统计图（Figure 3），完全没有语义类目维度的统计。
- 风格维度仅在 caption 中作为一个描述字段存在（Open-Sora 2.0 的六要素之一「video style」），但未据此做训练集配比。
结论：domain 分布基本是源数据集分布的被动继承，而非主动设计。[不确定]

### [Ovi](../models/Ovi.md) ⚠️

论文披露了一个明确且少见的「人物构成配比控制」机制，但未给出具体比例数字。
【显式配比控制】使用内部人脸检测模型（internal face detection model）将片段分为三类并「ensure an adequate mix」：单人视频（single-person）、多人视频（multi-person）、无人视频（person-free）。论文给出的动机是「让模型学会在多样上下文中生成视频，而不过拟合到某个特定子任务」——即避免像多数 A2V 模型那样退化为纯 talking-head 模型。这三类的具体百分比未公开[不确定]。
【语料定性构成】内部音视频语料被描述为「human and nonhuman data from diverse contexts」（人类与非人类内容、多样上下文）。
【间接的 domain 覆盖证据】论文 5.1 节的跨模态注意力可视化按内容类别组织，覆盖乐器演奏、鸟叫、火箭、动物、语音（多例）、直升机、体育等类别，可反推训练分布至少涵盖：人物对白、乐器/音乐、动物、载具/机械、体育运动、自然环境音等。
【音效侧 domain】通过引入 VGGSound / AudioSet / WavCaps，音效 domain 覆盖由这三个公开数据集的类目体系（AudioSet 632 类本体等）间接决定。
【未涉及】风格（写实/动画/CG）比例、场景类别比例、动作类别比例、概念均衡（concept balancing）等策略论文均未提及[不确定]。

### [Script-a-Video](../models/Script-a-Video.md) ⚠️

domain 层面的披露停留在类目列举，无任何配比数字，也无概念均衡机制描述：
【caption 侧 500K 的领域构成】三类定性列举：film（电影）、television（电视）、lifestyle（生活方式）。三者各占多少比例未披露，也未说明是否做过配比调控。这一选择明显偏向「有叙事、有对白、有多镜头、有人物」的内容——与 MTSS schema 的设计目标高度耦合：Reference 流需要有反复出现的人物，Shot 流需要有真实的镜头切换，Event 流需要有对白与音效。换言之，数据领域是被 schema 需求反推选定的，而非追求通用覆盖。
【生成侧的隐含 domain 划分】四套数据实际上是按「能力维度」而非「题材维度」划分的：identity-centric（人物身份能力）、multi-shot（镜头结构能力）、cinematic AV pairs（音视频协同能力）、cinematic alignment pairs（高保真对齐能力）。这是一种「按目标能力组织数据」的思路，每套数据服务一个训练阶段的特定学习目标，而非按内容题材做均衡采样。
【评测侧的 domain 覆盖】225 条评测样本覆盖电影/电视剧片段、短视频、室内场景、室外场景四类，论文称「covers a diverse range of categories and scenarios」，但各类占比未给出。
【概念均衡】论文未描述任何概念均衡、长尾补采、类目配额或重采样机制。
【MTSS 内部的类目体系】值得注意的是，MTSS schema 自身内建了一套实体分类体系：Reference 流将实体划分为 person（人物）、object（物体）、animal（动物）、scene（场景）四类，且只保留与主线情节相关（integral to the main plot）的实体，边缘元素一律降级到 global scene description。这是一种「叙事重要性驱动的实体筛选」，构成了标注层面的隐式 domain 结构，但并非训练数据的采样配比。
整体而言，训练数据的题材配比数字属信息空白。[不确定]

### [Seedance 2.0 与 Seedance 1.5 pro](../models/Seedance_20_Seedance_15_pro.md) ⚠️

1.5 与 2.0 的训练数据类目分布未披露，但可从两条侧证还原其配比意图：(1) Seedance 1.0 报告的「多样性导向数据采集」明确列出需最大化覆盖的维度：片段时长、分辨率、主体（人/动物/物体）、场景类型（自然风光/城市环境）、主体动作、体裁（纪录片/动画）、艺术风格、相机运动学、摄影技法；并在「分布再平衡」环节按主体类别、场景类型、主导动作、体裁、视觉风格、片段时长、分辨率、运动特性等维度统计频次，对过度表征的头部类目做下采样（downsampling），对长尾类目提高训练时采样概率（increase sampling probability）并发起定向数据补采（targeted data acquisition），目标是对视觉世界形成「更公平、更全面的表征」；SFT 阶段则定义了「数百个类别」（several hundred categories，按视觉风格、运动类型等关键属性划分）做定向采集。(2) Seedance 1.5 pro 报告称其数据管线优先保障「视频-音频一致性、运动表现力（motion expressiveness）与基于课程的数据调度」，说明配比上显式向高运动表现力样本倾斜——这与其在 SeedVideoBench 1.5 中新增「视频生动性（Video Vividness）」指标（拆分为动作、运镜、氛围、情绪四个维度）互为呼应，报告并批评业界普遍用慢动作换取稳定性的做法。Seedance 2.0 侧则强化了复杂交互与人体运动、多主体交互、多镜头叙事、中文方言/戏曲/唱段等类目。[不确定：所有具体比例数值]

### [SkyReels 系列](../models/SkyReels.md) ⚠️

这是 SkyReels 系列在数据侧最突出的方法论——「概念均衡（concept balance）」贯穿两代。
【SkyReels-V2】(1) 打标先行：由 SkyCaptioner-V1 输出的结构化字段中的「主体类别（subject category）」作为均衡的分类依据；(2) 均衡执行在后训练阶段——「根据字幕生成器的主体类别进行详细的概念均衡，这导致数据量减少了50%」，即为消除头部类别偏置不惜砍掉一半数据，是「质量优先于数量」的直接体现；(3) 预训练阶段亦使用「概念均衡的图像数据」与视频联合训练；(4) 视频类型层面做黑名单式剔除：监控录像、游戏录制、动画、无意义内容被整类过滤，说明目标域锁定在真实拍摄的影视/生活内容；(5) SkyCaptioner-V1 自身的训练集也是「从1000万条中精选200万条概念均衡视频」。论文未公布具体类目体系与各类目占比数值。
【SkyReels-V4】延续并扩展为两个正交维度：(1) 概念多样性（conceptual diversity）——基于类目体系（taxonomy）做匹配式均衡；(2) 运动多样性（motion diversity）——为每个主体类别或场景类别定义关键运动模式（key motion patterns），再按运动模式做均衡，避免同一类主体只出现单一动作。这一「类目×运动模式」的二维均衡是 V4 相对 V2 的主要增量。具体类目树与配比数字未公开。[不确定]

### [Sora 2](../models/Sora_2.md) ⚠️

完全未披露。System Card 未给出人物/动作/场景/风格等任何类别配比，未说明概念均衡（concept balancing）策略，未披露长尾概念处理方式。唯一可间接推断的线索来自能力描述：模型被强调在物理规律上表现更好（重力、动量、浮力、材料形变、碰撞动力学、物体恒存性），第三方解读称训练数据带有「物理标注」（physical annotations）覆盖这些概念，暗示可能存在面向物理交互的定向数据配比或标注体系——但该说法来自二手技术解读，非 OpenAI 官方表述，且无任何比例数字。此外从产品形态（cameo 真人出镜、社交feed）可推断人物/人脸类数据占比不低，但同样无官方依据。[不确定]

### [Step-Video-T2V](../models/Step-Video-T2V.md) ⚠️

报告未给出 domain 占比数字，但提供了一个在同期工作中相当有辨识度的概念均衡机制——「Video Concept Balancing（视频概念均衡）」：
· 用内部自研 VideoCLIP 模型对每个片段计算视频 embedding；
· 在高维 embedding 空间做 K-means 聚类，聚成 超过 12 万（120,000+）个簇 —— 这个簇数远高于同期同类工作（如 HunyuanVideo 的约 1 万个概念中心），意味着概念划分粒度极细；
· 为每个片段打两个派生标签：Cluster_Cnt（所属簇的样本数，用于识别过密/长尾概念）与 Center_Sim（到簇中心的余弦距离，用于识别簇内离群样本）；
· 基于这两个标签实现两类操作：其一，按簇大小做重采样，保证覆盖广泛类别、抑制头部概念过度集中；其二，在后训练阶段按 Center_Sim 剔除距簇中心过远的离群片段（把聚类同时用作「概念均衡」与「离群质检」两用，是本条目较有方法论价值的一点）。
各具体类目（人物/动作/场景/风格）的最终配比、以及重采样的具体倍率均未公布。Step-Video-TI2V 侧则有明确的风格失衡披露：训练数据中动漫风格视频占比超过 80%，且早期阶段仅用动漫数据，导致该模型动漫表现强而真实场景表现受限——这是一个罕见的、由官方自述的 domain 配比失衡及其后果案例。[不确定]

### [UniTalking](../models/UniTalking.md) ⚠️

UniTalking 是一个高度垂直的单 domain 数据集——目标 domain 极窄，因此不存在传统意义上的多类目配比问题：
【唯一 domain：人物中心的说话场景】整条流水线的设计目标被明确表述为「isolate high-quality, human-centric speech content」（分离出高质量的、以人为中心的语音内容）。三级过滤中的每一级都在向这个单一目标收敛：视频级剔除静态画面（保证人物有动作）、音频级剔除无语音样本（保证有说话）、跨模态级剔除唇音不同步样本（保证说话的人就在画面里）。最终产物被称为 human-centric dataset。
【与 UniVerse-1 的配比取向对比】UniVerse-1 是语音仅占 15.4%、非语音音景占 84.6% 的「通用音景优先」；UniTalking 则是接近 100% 的语音说话人内容，是完全相反的极端。这直接对应两者的能力定位差异：UniVerse-1 追求通用音视频生成，UniTalking 专攻 talking portrait。论文结论中承认「虽然仅在说话人生成上做了验证，但我们认为该框架可推广到通用音视频合成，包括音效与音乐」——即通用性只是主张，未经数据与实验支持。
【未披露的配比维度】230 万条内部的任何细分统计均无：无人物属性分布（性别/年龄/人种）、无场景分布（演播室/户外/会议/vlog）、无景别分布（特写/半身/全身）、无单人/多人比例、无正面/侧面比例、无 OpenHumanVid 与内部数据的来源配比。也无任何概念均衡（concept balancing）或重采样机制的描述。
【一个可推断的偏斜风险】OpenHumanVid 的原始来源为公开数据集集合，而华为内部数据来源不明，两者的人物与场景分布差异无从判断，论文亦无跨源分布对齐的说明。[不确定]

### [UniVerse-1](../models/UniVerse-1.md) ⚠️

训练数据的 domain 覆盖只有定性列举，无任何比例数字，也无概念均衡（concept balancing）机制描述：
【内容类目】音乐综艺、古典音乐演奏、烹饪教程、公开演讲、访谈、vlog、工具使用演示、电影片段、Pexels 素材片。
【唯一可量化的配比切分】按数据「性质」而非「题材」划分的三分法：语音中心 1,187 小时（15.4%）、通用音视频 3,074 小时（40.0%）、VGGSound+AudioSet 3,422 小时（44.5%）。可见语音数据仅占 15%，音效/环境音类数据占绝对多数——这与 MOVA「只保留语音片段」的极端取向恰好相反，UniVerse-1 是通用音景优先、语音为辅的配比。
【题材配比意图的间接证据】所选题材与三大目标能力一一对应（音乐类→乐器声、演讲访谈类→语音唇同步、烹饪工具类→foley 与环境音），说明配比是按「目标声音类型」倒推采集的，而非按视觉题材均衡。
【唯一的百分比类目表】论文附录给出的九类音频类目占比（自然环境 36.1% 等）描述的是 Verse-Bench 评测基准的样本分布，而非训练数据分布，不可混用。
训练数据的题材 domain 配比数字属信息空白。[不确定]

### [Unison](../models/Unison.md) ⚠️

论文未给出任何 domain 配比数字，也未描述概念均衡机制。以下为基于数据源构成的定性重建：
【定位：人物中心（human-centric）】这是论文标题即声明的定位。五个音视频数据源中四个是人脸/人物数据集（OpenHumanVid、HDTF、VFHQ、CelebV-Text），仅 VGGSound 为通用视听事件数据集，因此语料的绝对主体是含人物的画面。
【可从各源数据集反推的题材覆盖】
- OpenHumanVid → 人物动作、日常活动、影视片段中的人物场景，是人物运动（motion）多样性的主要来源；
- HDTF → 演讲、新闻播报类正面高清人头，贡献标准化的唇同步样本；
- VFHQ → 访谈场景人脸，贡献身份与表情多样性；
- CelebV-Text → 野外人脸视频，带属性/动作/情绪文本标注，贡献人脸属性多样性；
- VGGSound → 310 类视听事件（乐器演奏、动物、交通、机械、自然等），是唯一的非人物 domain 来源，也是模型环境音能力的视觉锚点。
【论文实际关注的「domain」维度不是题材而是声学场景类型】从定性论述可以看出，Unison 真正在意的 domain 划分是按「语音与音效的相对关系」切分的三类场景：
1) 叙述主导场景（narration-dominant，如密集旁白）——SCG 门控抑制音效流以保护语音纯度；
2) 复杂声景场景（complex soundscapes，如音乐演奏）——SCG 放大跨流影响以丰富非语音声学；
3) 二者并重的困难场景——论文反复举例「边弹奏乐器边唱歌」（singing while playing an instrument）、体育赛事解说（解说声不应掩盖看台欢呼与撞击声）、海滩场景（人声不应压过海浪）、钢琴演奏（音符起音须对应手指动作）、机车行驶（引擎声不应被人声压过）。
这一「按语音-音效比例而非按视觉题材」的场景分类是 Unison 的特色，也直接对应其 SCG 门控的实例级分析（Fig. 8c 给出了跨语义类别的平均门控值，但论文未列出类别清单与占比）。
【彻底缺失的信息】各数据源的小时数/条数占比、题材类目表及占比、人物属性（性别/年龄/族裔）分布、运动类型分布、场景类型分布、任何形式的概念均衡或重加权机制。论文既无 domain 配比表，也未说明训练时是否对某些子集做上/下采样。[不确定]

### [Veo 3 / Veo 3.1](../models/Veo_3_Veo_31.md) ⚠️

[不确定] 类别/domain 配比策略未披露。可作反推的官方线索：(1) Model Card 明确说明「生成合成 caption 以提升训练数据中与视频关联的概念的多样性与丰富度」（synthetic captions to improve the variety and diversity of concepts），这是一种以 caption 侧改写实现概念均衡的策略，而非直接的样本重采样；(2) 技术报告在「Dangerous Capabilities」一节指出 Veo 3「有生成电影化素材的偏好，频繁出现镜头切换与戏剧化机位」（bias for generating cinematic footage, with frequent camera cuts and dramatic camera angles），并因此难以生成低制作水准的写实胁迫类视频——这强烈暗示训练数据显著偏向电影/专业影视与高制作水准素材，UGC/监控/手机随手拍等低制作水准 domain 占比偏低；(3) 公平性评测显示模型在未指定人种时明显偏向浅肤色（skews towards lighter skin tones），并存在特定词汇与特定人群的语义偏差（semantic bias），反映训练数据的人物表征分布本身不均衡；(4) 官方承认在文字渲染上仍然较弱，暗示含文字/OCR 场景的数据覆盖或标注不足。以上均为从公开报告反推，非官方配比说明。

### [Vidu S1](../models/Vidu_S1.md) ⚠️

未给出定量配比，但存在很强的定性 domain 约束，本质上是一个高度垂直的数据分布：
(1) 主体约束：每个 clip 必须且仅含一个主体（exactly one subject），且主体在画面中占比合理——即严格的「单人」分布；
(2) 镜头约束：只保留静止镜头或慢速运动镜头（retain static shots or shots with slow motion），以降低长时生成中的镜头漂移风险——刻意压低了大运镜的比例；
(3) 交互性约束：要求画面中主体展现清晰的动作或行为（clear actions or behaviors），以保证模型学到有意义的运动信息；
(4) 风格覆盖：通过影视剧素材补充多角度、多场景、多视觉风格；产品端支持真人、动漫、宠物等多种自定义角色形象，暗示训练数据覆盖二次元与非人主体（论文提到人脸检测专家模型对夸张/高度风格化的2D动画主体泛化不佳，故引入 omni 模型补充）；
(5) 语义标签维度：由 omni 模型沿 editing（剪辑）、subject（主体）、action（动作）、emotion（情绪）、face（人脸）、speech（语音）、scene（场景）、shot（镜头）、tone（影调）九个质量维度打标，这九个维度构成了事实上的 domain 描述体系，但论文未公布各维度的配比数值。
两大来源（直播/口播 vs 影视剧）之间的比例同样未披露 [不确定]。

### [主流视频预训练数据集合并调研：Panda-70M、InternVid、Koala…](../models/pretraining_datasets.md) ⚠️

**七者的类目治理水平差距极大，从「完全没有」到「7主题108子类的显式taxonomy」**：
- **Panda-70M（无类目体系）**：论文**没有品类饼图与百分比**，附录E仅列出8个用于可视化举例的类别（动物、风景、美食、体育活动、交通工具、教程与叙事、新闻与电视节目、游戏与3D渲染），无任何数字；图13是100K条caption的词云。作者在局限性中主动承认：由于 HD-VILA-100M 偏「口播密集型」，**主要类目实为新闻、电视节目、纪录片、第一视角视频、教学与叙事视频**。若需类目分布只能引用上游 HD-VILA-100M 的15类均衡采样。
- **Koala-36M（无类目体系）**：同源于 HD-VILA-100M，论文亦未给类目分布。
- **InternVid（16个主题，但仅有饼图）**：类目分布仅以图3饼图呈现，**正文与附录均无数值表**，无法引用精确百分比。可引用的分布是源视频时长（≤5分钟49%、5–10分钟26%、>20分钟仅8%）与国别（英美澳日韩中俄法等）。作者主动说明因版权原因**刻意稀缺或排除了监控录像、体育赛事、电影、纪录片**四类。
- **MiraData（7类YouTube taxonomy，仅柱状图）**：(1) 3D引擎渲染场景、(2) 城市/风景漫游、(3) 电影、(4) 第一人称视角、(5) 物体创造/物理规律演示、(6) 延时摄影、(7) 人体动作展示。**刻意超采(1)和(3)**，理由是「多样性更好、画质更高」，并主张3D引擎素材有助于学习物理规律一致性。**每类计数仅存在于图2的无标注柱状图**（纵轴0–12K视频 / 0–78K clips），无数值表；但元数据CSV的 source 列实际编码了平台+类目（如 `youtube, 3D engine-rendered scenes`、`videvo, nature` 等），**可自行 value_counts 还原精确成分**——这是本次调研中唯一可用的类目还原路径。注意：**没有 GTA-V 专项来源**，游戏内容归入「3D引擎渲染场景」，论文刻意不点名具体游戏（应为版权考虑）。
- **LVD-2M（8类，有精确百分比，最实用）**：用 **BART** 对caption做分类——风景24%、人物23%、交通13%、体育11%、动物11%、美食9%、游戏5%、其他4%。这是七者中**唯一给出完整数值化类目分布**的数据集。
- **UltraVideo（7大主题×108子类，体系最完整但数值缺失）**：taxonomy 由「对 Koala-36M 的caption做名词统计 → LLM 归纳 → 人工修订确认」得到。七大主题为：**视频场景、主体、动作、时间事件、镜头运动、视频类型、情绪**。这是七者中唯一把「镜头运动」「情绪」纳入数据配比维度的体系，并据此做**各类目均匀采样**（明确写明按主题相似度做 uniform per-category sampling）。**但108个子类从未在正文中列出，仅在图4(a)以占比图呈现**；GitHub issue #5 专门追问主题↔视频的映射关系，至今无人回复。
- **OpenVid-1M**：无类目体系披露。[不确定]
**总体判断**：只有 UltraVideo 做了「先定taxonomy、再按类目均匀采样」的主动配比设计，MiraData 做了粗粒度的类目定向超采，LVD-2M 只做了事后统计，其余四家的类目分布完全是上游语料分布的被动继承。**七者全部没有做长尾概念均衡（concept balancing）或语义聚类后重采样。**

## 音频类别分布与配比（语音/音效foley/音乐/环境音/静音的比例及控制策略）——AV模型独有维度

`audio_category_distribution` · 详细程度: detailed

### [Allegro](../models/Allegro.md)

不适用。Allegro 为纯视觉视频生成模型，训练数据不保留、不处理音轨，论文全文未涉及语音/音效/音乐/环境音/静音的分类与配比。

### [Apollo](../models/Apollo.md) ⚠️

这是 Apollo 数据设计的核心维度，也是其区别于视觉优先同类工作的关键点——整个数据集按音频类型做树状分层切分（Section 4.2 Audio-Guided Data Splitting）：
【第一层：人声 vs 非人声】先将数据分为 vocal（含人声）与 non-vocal 两支，非人声支构成 sound split（自然声/音效子集）。
【第二层：人声内部三分】从 vocal 子集进一步划分为三类——singing（歌唱）、single-speaker speech（单说话人语音）、multi-speaker speech（多说话人语音）。
【最终四类】single-speaker speech / multi-speaker speech / singing / natural sound，论文原文：「The dataset contains single-speaker speech, multi-speaker speech, singing, and natural sound clips」。
【差异化标注】不同子集走不同标注路径——语音与歌唱子集额外抽取说话人属性（性别、年龄）并做逐字转写；sound split 只做音频 caption，不做转写。这是「按音频类别分流后再分别打标」的典型设计。
【与同类对比】值得注意的是 Apollo 显式保留了 singing（歌唱）这一类，而 MOVA 等工作因音频塔容量受限在歌声上表现退化并将其排除或弱化；Apollo 把歌唱作为一级子集，说明其在数据侧刻意覆盖了唱歌口型这一高难场景。同时它并未像 MOVA 那样「只保留语音」，而是让 natural sound（foley/环境音）与语音共存于同一训练集，配合多任务掩码训练同时习得 T2A 与唇同步能力。
【空白】四类各自的样本数或占比、音乐（instrumental music）是否单列、静音样本的处理（仅知静音占比 >20% 的样本被剔除）等均未披露。[不确定]（仅类别体系明确，比例数字缺失）

### [CineDance / CineDance-1M](../models/CineDance.md) ⚠️

音频被显式拆解为三类并分别标注，但未公布各类的数量占比：
【三类划分】在镜头级音频 prompt 中，音频内容被分解为 music（音乐）、ambient sound（环境音）、effects（音效）三类分别描述；对白/语音则通过独立的 ASR 通道处理，形成事实上的「语音 / 音乐 / 环境音 / 音效」四类并行标注结构。
【与说话人的关系】另有独立的 character voice description（角色音色描述）字段，刻画每个角色的嗓音特征。
【质量侧刻画】音频质量以 DNSMOS（信号保真度）与 CLAP embedding 的时序方差（衡量音频内容随时间的变化丰富度，可间接反映静音/单调音轨）两项指标量化，均作为元数据保留。
【占比与配比策略】论文未给出语音 / 音乐 / 音效 / 环境音 / 静音各自的时长或样本占比，也未描述任何主动的音频类别配比控制策略。这是该数据集在 AV 维度上披露不足之处。[不确定]

### [CogVideoX](../models/CogVideoX.md) ⚠️

不适用于 CogVideoX 本体：视频模型训练完全不使用音轨，数据 pipeline 中没有任何音频维度，也没有语音/音效/音乐/环境音/静音的分类与配比。
级联的 CogSound 侧存在音频数据，但其训练数据构成、音频类别配比、静音处理策略均未公开 [不确定]。公开信息仅说明其生成目标覆盖「爆炸、水流、乐器、动物叫声、交通工具声」等复杂音效以及节奏/音乐元素，暗示训练数据以 foley 音效为主并含一定音乐成分，但无量化配比。也未见其是否生成或建模语音对白的说明。

### [Cosmos-Predict2.5](../models/Cosmos-Predict25.md)

不适用。Cosmos-Predict2.5 无音频模态——不生成音频、不以音频为条件、数据 pipeline 中无任何音轨处理环节（不做音轨提取、不做语音/音效/音乐分类、不做静音检测）。全文未出现音频类别配比的任何设计。若需覆盖 Physical AI 场景的声音维度，需另行外接音频模型，论文未讨论。

### [Data-Juicer 2.0](../models/Data-Juicer.md) ⚠️

[不确定] Data-Juicer 未提供音频类别（语音/音效foley/音乐/环境音/静音）的分类与配比控制能力，这是其相对于 AV 生成数据 pipeline 的明显缺口。
【现有的音频侧能力仅限于】
  · 质量维度：audio_nmf_snr_filter（基于非负矩阵分解估计信噪比并按区间过滤）、audio_duration_filter（时长）、audio_size_filter（文件体积）。
  · 语义标注维度：video_tagging_from_audio_mapper 与 video_audio_ASR_mapper 均基于 Audio Spectrogram Transformer（AST，在 AudioSet 上训练）产出音频事件标签——AudioSet 本体包含语音/音乐/环境音等527类，理论上可据此近似区分三类音频，但 DJ 未把它封装为显式的三分类字段或配比控制算子。
  · 说话人属性维度：video_audio_detect_age_gender_mapper（wav2vec2 检测年龄性别）、video_audio_speech_emotion_mapper（语音情绪）。
【缺失的关键能力】无音轨源分离算子（如 Demucs/Bandit 式的 speech/effects/music 三分离），因此无法做 Foley-Omni 那样的字段级能量门控与音轨类别配比；无静音检测与静音占比阈值算子；无「无音轨样本剔除」的专用算子（需借 audio_duration/size filter 间接实现）；无音乐/人声分离用于背景音乐剥离。
【实践佐证】其唯一的官方视频案例（VBench T2V）为纯视觉任务，全程未启用任何音频算子，说明音频侧能力尚未经过大规模视频生成场景的实战检验。若要用 DJ 构建音视频联合生成的训练数据，音频类别体系需自行以自定义算子扩展——好在 DJ 的算子接口设计（Mapper/Filter 基类 + YAML 注册）使扩展成本较低，这也是团队宣称的可编程性（programmability）卖点之一。

### [Foley-Omni](../models/Foley-Omni.md) ⚠️

这是 Foley-Omni 最具特色的维度，也是其区别于同期 AV 工作的核心贡献——它把「音频类别」从隐式属性提升为显式的一等结构化字段。
【三分类体系】语音 [WORDS] / 音效 [AUDIO] / 音乐 [MUSIC]，三者构成结构化标注 Ŝ 的三个并列字段。关键设计是「字段可为空」（Each field can be left empty when the corresponding audio component is absent）——空字段本身携带信息，等价于「该片段不含此类音频」的显式负标注。这使得同一套条件接口既能表达单任务（仅填一个字段=TTS/TTA/TTM）也能表达完整配乐（多字段共存=V2ST），实现了任务的统一形式化。
【比例控制策略——双阶段判定】
第一阶段（视觉+听觉联合检测）：Gemini 2.5 Pro 同时接收视频帧与音轨，被显式指令「先判断某类音频组件（Speech/Sound Effects/Music）是否physically present，若存在再生成对应描述」——即先做存在性二分类，再做描述生成，而非一股脑生成三段描述。
第二阶段（声学能量门控纠偏）：用 Bandit 源分离模型（Watcharasupat et al., 2024，专为 cinematic audio source separation 设计的speech/effects/music三分离模型）把原始音轨分离为三个 stem，仅当对应 stem 的 RMS 能量 E(a_c) > −35 dB 时才保留该字段标注。此步骤专门用于消除「视觉幻觉」——即 Gemini 看到画面中有钢琴/有人张嘴/有汽车，就臆断存在音乐/语音/音效，但音轨里实际听不到。这是本文明确点名的方法论创新：视觉路径标注 + 声学路径验证的双路交叉校验（two-path validation），论文称在典型音视频数据集构建中缺失。−35 dB 阈值由「对小规模验证子集的人工检视」（manual inspection of a small validation subset）确定。
【静音处理】过滤阶段第一条即剔除含静音（silence）的片段，无音轨/静音样本不进入后续流程。
【实际分布】训练集三类音频的最终占比未直接给出，但可由任务组小时数近似：语音类66%、音效类27%、音乐类3%。
【评测集显式配比】V2ST-Bench 300条严格要求「≥2个音频组件共存」，配比为 语音+音效 150条（50%）、语音+音乐 120条（40%）、语音+音效+音乐 30条（10%）。三类组合中语音100%出现，可见基准设计也以语音为轴心；三组件齐全的最难场景仅占10%。
[不确定] 训练集中三字段各自的实际非空比例、各组合模式（单字段/双字段/三字段）的样本数分布未披露。

### [Goku](../models/Goku.md)

不适用。Goku 为纯视觉的图像+视频生成模型，训练数据为「图文对」与「视频文本对」，不包含音轨；论文全文未涉及语音/音效 foley/音乐/环境音/静音的任何分类、比例或控制策略，数据流水线中也无音频通道。若原始视频自带音轨，论文未说明是否保留或丢弃（从数据形态 video-text pair 判断应为直接丢弃音轨）。

### [HunyuanVideo-Foley](../models/HunyuanVideo-Foley.md) ⚠️

音频类别是本工作数据处理的核心组织维度（而非视频类别），但同样只有机制无数字：
【分类工具】两类模型串联使用：(1) 语音-音乐检测模型（speech-music detection），做 speech / music / 其他 的粗粒度判别；(2) 音频分类模型（audio classification model），做更细粒度的声学事件分类。两者均未点名具体模型（未说明是 PANNs、AST、BEATs、EAT 还是自研），也未给出类目数量。[不确定]
【静音的处理：比例型阈值而非二元判定】静音占比超过 80% 的片段被丢弃。这里的设计细节值得注意——采用的是「静音占比」（silence ratio）这一连续比例指标而非「是否含静音」的二元判据，且阈值定得相当宽松（允许最多 80% 静音）。宽松的原因可推断为：Foley 场景中大量样本本就是「长时间安静 + 短促事件音」的形态（如空旷房间里一次关门声），若阈值收紧会把最典型的 Foley 训练样本误杀。80% 这一数值实际上只用于剔除近乎完全无声的片段，是一条兜底规则而非质量筛选。这与 UniVerse-1「用音量/能量/过零率直接剔除静音片段」的二元做法相比，更贴合 Foley 任务特性。[不确定：阈值的具体计算方式，如静音判定的能量门限、时间分辨率均未说明]
【语音与音乐：检测但不必然剔除】论文只说做 speech-music 检测并打标签，没有说检测出语音或音乐后就丢弃。这意味着语音与音乐数据很可能被保留在训练集中，只是通过类别标签参与配比控制——这是一个与 MOVA（只保留语音）、UniVerse-1（按语音有无分流保留）都不同的第三条路线：全类保留 + 标签化配比。对 Foley 任务而言，保留少量语音与音乐有助于模型学会「什么时候不该生成音效」以及在混合场景中的声学分层，但过量则会稀释 Foley 事件音的学习信号——配比的把握正是「类别分布管理」要解决的问题，可惜未披露。[不确定]
【环境音/事件音】未单列比例。
【与 UniVerse-1 的取向对比】UniVerse-1 给出了明确的三段配比（语音 15.4% / 通用 40.0% / 公开数据集 44.5%），HunyuanVideo-Foley 数据规模大 13 倍却一个比例数字都没给。这是本工作在数据披露上明显逊于同类的地方。

### [HunyuanVideo](../models/HunyuanVideo.md) ⚠️

不适用。HunyuanVideo 与 HunyuanVideo 1.5 均不生成音频，训练数据不含音轨处理，两份报告均无音频类别配比内容。若需腾讯体系内的音频数据方法，应转向 HunyuanVideo-Foley（视频到音频，约10万小时TV2A数据集，含语音/音效/音乐的多标签平衡策略）等独立工作。[不确定]

### [InstructAV2AV](../models/InstructAV2AV.md) ⚠️

音频类别在本工作中被组织为「语音（speech）」与「非语音声源事件（sound events）」的二分，且两条支路走完全不同的处理链路——这是pipeline设计中最清晰的分叉点。
【二分体系与各自的处理链路】
  · 语音主导内容（speech-dominated）：TalkNet（Tao et al., 2021）做主动说话人定位（active speaker detection），ElevenLabs Scribe（2025）做 ASR 提取精确的语音时间戳。关键约束是：只有当语音与画面中可见的说话人在时间上对齐时才保留（clips retain speech only when temporally aligned with a visible on-screen speaker）——这一条同时完成了三件事：剔除画外音/旁白/解说、剔除后期配音、保证保留下来的样本天然具备唇同步监督信号。这是把「声音必须来自画内可见声源」作为硬性数据准入条件，比事后用同步分数过滤更彻底。
  · 非语音内容：过滤掉声源模糊（ambiguous sound sources）的片段以保证声源清晰度，然后为每个保留片段赋予一个唯一的语义声音事件标签（a distinct semantic sound-event label，例："dog barking"）。「distinct/唯一」是关键约束——要求片段内只有一个主导声源，多声源混杂的片段被排除。原因很直接：编辑任务要求「改这个物体的声音、不动其他声音」，若片段内声源混杂则无法构造干净的 source-target 对，SAM-Audio 也无法可靠分离。
【静音与音频质量】用 PyDub 剔除低于 -45 dBFS 的静音片段（这是论文中唯一给出确切数值的阈值）；用 Audiobox-Aesthetics（Tjandra et al., 2025）做音频质量评估并丢弃低质音频（阈值未给出）。
【背景音/环境声的特殊地位】环境声在本工作中不是被过滤的对象，而是被保护的对象——SAM-Audio（Shi et al., 2025）把目标实体的声音从原音轨中分离出来，编辑后的新声音再与保留的背景音重新混合。这意味着背景/环境声在 source 与 target 中逐样本保持严格一致，构成了「内容保持（content preservation）」这一评测维度在听觉侧的监督信号。这个设计与 Foley-Omni 用 Bandit 做三路分离的目的不同：Foley-Omni 分离是为了验证标注，本工作分离是为了构造「只改前景声、背景声不变」的受控配对。
【音乐】[不确定] 论文未把音乐单列为一类处理对象，未提及背景音乐的检测、保留或剔除策略。考虑到来源含大量电影片段（普遍带配乐），配乐大概率被归入「背景音」由 SAM-Audio 分离后原样保留，但未见明确说明。
[不确定] 未给出语音类与非语音类样本的实际数量或比例、未给出声音事件标签的类别分布（310类 VGGSound 标签体系是否沿用亦未说明）、未给出各类音频的时长占比统计。

### [2026 其他音视频联合生成](../models/JAVG_2026_misc.md) ⚠️

这是 AV 模型独有维度，本批工作中 ALIVE 与 NAVA 有实质设计，其余偏弱：
【ALIVE —— 分阶段的语音/音效两段式 + BGM 比例主动管控】
(1) 音频塔的两阶段配比切换极为鲜明：T2A Stage I 为纯语音预训练，用 700k hours of transcribed speech（384M samples，1 epoch，lr 5e-5），此阶段几乎全部是语音；T2A Stage II 切换为混合，「approx. 5k hours」高质量语音 + 「111k hours」带声音标注的视频数据（19M samples，10 epochs）——即从「70 万小时纯语音」骤降到「5 千小时精选语音」并大量掺入视频原生音轨。这是一个「大规模语音打底 → 小规模高质语音保活 + 大规模真实音景补充」的清晰调度。注意 Stage II 的 epoch 数（10）远高于 Stage I（1），说明高质量混合数据被反复利用。
(2) BGM 比例主动管控：原文「we assess the correlation and separate samples where the audio is highly correlated with the visual content, while also managing the proportion of weakly correlated data, such as background music (BGM), to optimize the dataset's composition.」——把音频按「与画面的相关性强弱」二分：强相关（diegetic，画内音）样本单独归集，弱相关（如 BGM，非画内音）样本则控制其占比而非全部剔除。这是一个比「一刀切剔除 BGM」更细腻的策略：保留一定比例弱相关音频可让模型学会生成配乐，但比例过高会破坏音画因果关系。具体比例未公开[不确定]。
(3) caption 层面的三类分流：<W> 标记语音（verbatim speech content），<I> 标记非语音声学事件（non-speech acoustic events），加上 Subjects 字段中的 acoustic profiles（声学画像），构成语音/音效/音色三条标注轨。
(4) 九大 Level-1 领域中 Home Sounds 与 Sound Effects 两类直接对应非语音音频类目，且每个 Level-3 视觉标签都绑定音频关键词——音频类别配比被嵌入到 domain 配比体系中统一调度。
【NAVA —— 五类音频标签驱动的语料组织】用 YAMNet 音频分类器配合 omni-modal tagger 将全部片段打上五类标签：single-speaker speech / multi-speaker speech / ambient sound / music / singing。这套五分法比常见的「语音/音效/音乐」三分更细，特别是把单人语音与多人语音分开（对应其多说话人 timbre 控制能力）、把 music 与 singing 分开（歌唱同时含语音与音乐属性，单独成类合理）。训练阶段的音频配比通过「audio-only : audio-visual」比例调度体现：Stage 1 为 3:1（音频侧主导，先把音频生成能力练强），Stage 2 反转为 1:2（音视频侧主导）。各音频类别的绝对占比未公开[不确定]。
【CCL】音频预训练引入 WavCaps 与 VGGSound 两个音效/通用音频数据集，联合训练数据（OpenHumanVid + 访谈/短剧/电影）则以对白语音为主——构成「通用音频预训练 + 语音为主联合训练」的结构。比例未公开[不确定]。
【Baton】数据源中 AudioCaps 与 WavCaps 提供通用音效/音景描述，OpenHuman-Vid 提供人声，混合比例未公开[不确定]。
【OmniCustom】音频几乎全部为单人语音（人声音色定制任务），无音效/音乐维度；音频统一重采样至 16kHz。属于音频类别最单一的一项。
【StreamChar】以角色语音为绝对主体，Emilia 为纯语音数据集；无音效/音乐类别设计[不确定]。
【ITS-JAVG】无训练数据；但其验证器组合覆盖了音频语义（ImageBind-TA、AVHScore）与同步（JavisScore）两类，评测基座之一 MMDisCo 基于 VGGSound（以音效为主），另一 JavisDiT 覆盖更广类目。

### [音视频联合生成基线合集](../models/JavisDiT_baselines.md) ⚠️

这一维度上五者分化极大，恰好构成「音效专注 → 语音专注 → 二者兼顾」的完整光谱：
【MM-Diffusion / AV-DiT —— 纯环境音与音乐，零语音】Landscape 全部是自然环境音（雨、雷、风、火、水），AIST++ 全部是音乐（60 首舞曲）。两个数据集都不含人类语音与对白，因此这两个早期基线完全不具备唇同步与 TTS 能力——这是它们与 2025 年后模型最根本的能力鸿沟。
【JavisDiT —— 音频预训练全类通吃，视频阶段刻意剔除语音】音频侧 78 万条来自 10 个数据集，JavisDiT++ 明确说明「不采用任何数据过滤策略，以确保最大化的文本到音频生成能力，覆盖通用音效（general sound）、音乐（music）与语音（speech）三类」——即音频预训练阶段刻意保持类别全覆盖、零过滤。但到了音视频 SFT 阶段策略反转：「使用 FunASR 检测工具剔除掉大部分包含人类语音的视频」。这是一个非常明确的类别配比决策——JavisDiT 系列有意放弃唇同步/对白生成能力，把模型能力集中在环境音与音效的事件级对齐上，从而避开高难度的唇形建模。JavisBench 的分类体系中「声音类型（Sound Type）」是五大评测维度之一，说明评测侧对音频类别是显式分层的。
【Harmony —— 严格 1:1 语音/环境音配比】最明确的配比策略：阶段一（音频预训练）与阶段三（跨任务联合训练）均采用「人类语音数据集与环境音数据集 1:1 混合」。语音侧 200 万条（Emilia + OpenHumanVid + SpeakerVid 经一致性筛选），环境音侧为 AudioCaps + Clotho + WavCaps + 自采 200 万条。评测端 Harmony-Bench 进一步把「语音+环境音共现的复杂场景」单独立为一档 50 条子集——直指真实视频中语音与环境音同时存在这一最难情形。可以说 Harmony 是本合集中对音频类别分布思考最系统的工作。
【UniAVGen —— 语音为绝对主体】阶段一在 Emilia（TTS 语料）英文子集上做纯音频预训练，阶段二三用真人音视频数据，全流程以人声/对白为核心；评测指标（SyncNet、Whisper WER、音色一致性、情绪一致性）也全部围绕语音，环境音与音乐能力未见报告[不确定]。
【静音处理】五者均未描述静音检测阈值或静音占比控制[不确定]。

### [可灵 3.0 / 可灵视频 3.0 Omni](../models/Kling_30_Omni.md) ⚠️

[不确定] 可灵3.0 Omni 未公布语音/音效/音乐/环境音/静音的配比。能力覆盖上三类齐全：对白语音（5语种+口音）、环境音（ambient）、音效（foley），官方未强调音乐生成。同团队 Kling-Foley 的做法可作为方法论旁证：先用音频分类模型将素材分为音效/音乐/语音/演唱四类，再对每类分别调用音频理解大模型生成描述；训练素材按 AudioSet 三级本体（约1000个三级类目，归并为1919个标签、九大声音场景：交通、人声、动物声等）做关键词检索与配比；并通过 VAD 过滤掉静音占比≥0.2 的样本，即对“静音比例”设有明确上限。可灵3.0 Omni 是否沿用该分类体系与静音阈值未公开。

### [LTX-2](../models/LTX-2.md) ⚠️

这是 LTX-2 数据侧最有价值、也是唯一明确的筛选准则，但同样无比例数字。
【筛选策略】不是照单全收 LTX-Video 的全量视频，而是「聚焦于含有显著且信息量丰富的音频成分的视频片段（focusing on video clips that contained significant and informative audio components）」——即以「音频信息量」作为构建 AV 训练子集的核心门槛，剔除无声、静音占比过高或音轨与画面无信息关联的片段。这一「音频信息量筛选」是 LTX-2 相对纯视频模型在数据侧的关键增量。
【覆盖类别】caption 系统与模型能力覆盖四类：对白/语音（speech，含精确转写）、音乐（music）、环境音/氛围（ambient sounds、background）、拟音/音效（foley）。论文强调 LTX-2「不止于生成语音」，而是产出跟随角色、环境、风格与情绪的完整音景（full soundscape）。
【未披露】四类音频各自占比、静音片段保留比例、类别配比控制方法、「显著且信息量丰富」的具体量化判据（无 SNR/响度/静音占比等阈值）。模型卡亦承认「生成非语音内容时音频质量较低」，暗示语音类数据占比可能显著高于音效/音乐类。[不确定]

### [LongCat-Video](../models/LongCat-Video.md) ⚠️

基础版 LongCat-Video 无音频模态，该维度不适用。
同系列 LongCat-Video-Avatar 1.5 涉及音频，但其音频处理是「作为驱动条件」而非生成目标，因此没有语音/音效/音乐/环境音/静音的生成侧配比设计。可归纳出的音频相关数据分层为：(1) 语音类为绝对主体（近景人脸、访谈两类来源均以清晰语音为核心）；(2) 歌唱/音乐类单列（音乐视频来源，用于歌唱口型与节奏性动作）；(3) 静默（非说话）数据被专门构建为独立子集——用 Qwen3-Omni 初判 + Qwen3-VL 复判的双模型一致性机制，仅当两个模型都判定主体未在说话时才保留该片段，用于让模型学会「有音频但主体不说话」的情形；(4) 多人并发说话区间被显式剔除（excluding intervals with concurrent speaker activity）。各类占比均未披露。[不确定：音频类别的具体配比数值]

### [MOVA](../models/MOVA.md)

这是 MOVA 数据设计中最具决断性的一环——最终训练集**只保留语音片段**，是一次极端的音频类别配比选择：
【预处理阶段分类】用 Silero VAD 将音轨切为 speech / non-speech 区间，结合 PySceneDetect 的场景切点，生成四类 8.05 秒定长片段：单场景语音、单场景非语音、多场景语音、多场景非语音。
【关键配比决策】“Ultimately, only speech segments are selected for training, accounting for 69.47% of all preprocessed segments.” 即语音片段占全部预处理片段的 69.47%，且最终只用这部分训练联合模型。对应到总时长保留率（Table 1）：原始 100% → Stage 1（语音+非语音）84.57% → Stage 1（仅语音）58.75%。也就是说，仅“只要语音”这一条就砍掉了约 26 个百分点的数据。
【音频类型分类器】使用 EAT 自监督音频 Transformer 分类模型对音频打标，构建 speech / non-speech 子集，按目标能力分流（唇同步 vs 通用 foley/环境音建模）。语音子集的构造条件为：EAT-contained-Speech 与 EAT-contained-Singing 两个标签均判为 True（或满足模型正类置信度）。
【音频塔预训练的类别配比】与联合训练不同，1.3B 音频塔预训练刻意覆盖三大类：通用音效（WavCaps + VGGSound）、音乐（JamendoMaxCaps）、语音（in-house TTS）。即“音效/音乐能力在音频塔预训练阶段注入，语音-唇同步能力在联合训练阶段强化”的两段式分工。
【代价】论文 Limitations 明确承认：由于音频塔仅 1.3B 且联合训练偏语音，模型在歌声、复杂音色纹理、音乐/器乐内容上表现退化。
【未披露】非语音片段内部（音效 / 音乐 / 环境音 / 静音）的细分比例未给出。

### [合并条目：① Mochi 1](../models/Mochi_MAGI_Motif.md)

不适用。三个模型均为纯视觉视频生成模型，训练数据不保留音轨，技术报告/博客全文均未涉及语音、音效 foley、音乐、环境音、静音的分类与配比控制。

### [Movie Gen](../models/Movie_Gen.md)

Movie Gen Audio 建立了「音频类型 × diegetic属性」的二维分类体系（表23），这是其数据工程的核心创新。
第一轴音频类型：voice（speech + singing）、non-vocal music（纯器乐音乐）、general sound（一般音效），由 AED（audio event detection）模型基于 AudioSet 527类本体自动打标，一个样本可同时命中多类（AudioSet中speech/singing子类→voice；music子类→music；其余→sound）。
第二轴 diegetic（画内、与画面有因果关系，如现场说话、现场乐队、海浪声、画外鸟叫）/ non-diegetic（画外，如纪录片旁白、背景配乐、罐头笑声、riser）。判定用 CAVTP（对比式 audio-video-text 预训练模型）的音频与视频embedding余弦相似度——因该模型主要在diegetic数据上训练，画内音的音视频embedding更接近。
分桶阈值（表24，由人工检视确定）：AED为 sound / voice / voice+sound 时，CAVTP>0.2 判 diegetic；AED为 music / music+sound / voice+music+sound 时，CAVTP>0.3 判 diegetic；AED为 music 且 CAVTP<0.1 判 non-diegetic；AED为 sound+music 且 0.1<CAVTP<0.25 判 non-diegetic；AED为 sound+voice+music 且 0.1<CAVTP<0.25 判 mixed。
配比与取舍：首先丢弃 silence 为主导类的所有视频；预训练只使用 diegetic 或 diegetic/non-diegetic 混合的数据，另加入一小部分 non-diegetic 背景音乐；明确优先 general sound（因低层物理规律最难学、错误最易被察觉），实际分布上 Sound 类占 O(100)M样本/O(1,000)K小时的绝对主导，其余四类各仅 O(10)M/O(100)K，即音效对音乐/含人声类约为10:1的量级差。
有意排除：不生成 diegetic speech（无脚本时难、且生成视频有伪影）和 non-diegetic speech（可用TTS替代）；微调的 cinematic split 中带人声的片段被整体排除。
微调阶段配比：影视级音视频（Cin-AV）与高质量纯音频（HQ-A）按 10 batch : 1 batch 混合。

### [NVIDIA NeMo Curator](../models/NeMo_Curator.md)

无。这是该工具链在音视频生成数据场景下最关键的缺失维度。
视频 pipeline 完全不处理音轨——没有语音/音效 foley/音乐/环境音/静音的分类 stage，没有各类音频的占比统计或配比控制策略，也没有从视频中抽取音轨的内置 stage。音频 pipeline 虽存在但其数据模型是「独立的音频文件 + 转写文本」（面向 ASR/TTS 语音数据），仅涵盖语音一类，且不与视频关联。26.07 版新增的音频增强 stage（tagging、SQUIM 质量指标、带宽估计、标点准备、可选二次 ASR 打分）也全部围绕语音质量，不涉及 foley/音乐/环境音分类。
因此若要用 NeMo Curator 构建音视频联合生成的训练数据，音频类别分布这一层需完全自建。

### [OmniHuman 数据集 + OHBench](../models/OmniHuman.md) ⚠️

音频侧采取「分离而非过滤」的策略，这是本数据集在音频维度最有价值的设计选择：
【核心做法：Demucs 四源分离】用 Demucs 对每条音轨做四源分离（four-source separation，标准 Demucs 输出为 vocals / drums / bass / other），随后「以人声（vocals）作为目标轨道，其余轨道混合作为背景轨」——即最终产出的是「人声轨 + 背景轨」的双轨结构。这与 UniTalking「剔除含背景音的样本」、UniVerse-1「按语音/非语音分桶」都不同：OmniHuman 既不剔除也不分桶，而是把混合音频拆解为可分别标注、可分别使用的两条轨道，把配比的决定权留给下游使用者。
【这一设计的实际价值】(1) 训练可控生成：模型可学到「有/无背景音乐」的条件控制；(2) 支持视频配音（video dubbing）任务：需要干净的人声轨作为监督；(3) 避免了「因为有背景音乐就丢掉整条高质量视频」的数据浪费——考虑到 YouTube 内容中带 BGM 的比例极高，若采用 UniTalking 式的剔除策略，保留率会灾难性下降。
【音乐属性标注】对背景中的音乐单独标注三个属性：类型（type）、情绪/氛围（mood）、相对音量（relative volume）。「相对音量」这一项尤其务实——它量化了音乐与人声的能量比，使下游可以按需筛选「音乐主导」与「人声主导」的样本。这是本次调研样本中少见的细粒度音乐标注。
【背景声标签】视频级标注中包含 background sound labels（背景声标签）与 music attributes（音乐属性）两类字段，分列于全局层。
【语音】通过 3D-Speaker 做说话人分离、FunASR-Nano 做转写、并标注情绪与时间戳（详见 dialogue_transcription_attributes）。
【未披露的配比】语音/音乐/音效/环境音/静音在 1,800 小时中的时长占比全部未给出；有无 BGM 的样本比例未给出；纯静音段的处理方式仅以「低质量剔除」笼统带过。[不确定]
【音频标准化】统一到 44.1 kHz 采样率。

### [Open-Sora 系列](../models/Open-Sora.md)

不适用。Open-Sora 与 Open-Sora Plan 全系列均不生成音频，训练数据 pipeline 不处理音轨，无语音/音效/音乐/环境音的分类、配比或统计。

### [Ovi](../models/Ovi.md) ⚠️

音频类别配比是 Ovi 数据设计的核心矛盾点，论文给出的是「分阶段偏置」而非静态比例。
【预训练阶段】「predominantly human speech」（以人类语音为压倒性主体），来自内部语料，强调语言多样性（linguistic diversity）、韵律（prosody）与音色变化（timbral variation）。此阶段几乎不含音效，目的是先把 TTS/说话人建模能力打牢。
【音频微调阶段】显式转向音效：「we emphasize modeling sound effects」，通过引入 VGGSound / AudioSet / WavCaps 补足 SFX；同时「为保持 TTS 能力并更好对齐下游目标」，额外掺入从内部音视频语料中抽取的音轨。这形成了一个「语音重→音效补充+语音保活」的两段式配比调度。三个公开音效集与内部语音音轨的混合比例未公开[不确定]。
【音视频融合阶段】音频侧完全来自配对视频的原生音轨，其语音/音效/音乐/环境音的自然比例由内部语料决定，未做人为再平衡的描述[不确定]。
【打标层面的类别分流】caption 阶段按「有语音 / 无语音」二分处理：含语音片段的音频描述强调说话人声学属性；非语音片段的音频描述则描述音效（sound effects）、背景音（background audio）、音乐元素（musical elements）——这实际是一个三类音频（语音/音效/音乐）的分类标注体系。
【静音处理】通过平均音量 ≥ −60 dB 的门槛剔除近似静音片段（见 audio_quality_filtering）。
【统一音频模型的必要性论证】论文在结果部分强调：真实世界视频常同时包含复杂音效与连贯语音，专用模型（纯 T2A 或纯 TTS）无法支持，因此必须训练一个统一的 T2A+TTS 音频塔——这是其音频类别混合训练策略的核心动机。
【音乐】BGM 生成能力被列为特性，但音乐数据是否单独引入未说明[不确定]。

### [Script-a-Video](../models/Script-a-Video.md) ⚠️

音频类别在 MTSS schema 层面有明确的三分类定义，但训练数据中各类别的比例完全未披露：
【schema 定义的三类音频事件】Event 流将所有音频事件严格分类为三种 type：
1) dialogue（对白/语音）——带 speaker（绑定 Reference ID）与 line（逐字台词文本）；
2) sfx（sound effect，音效）——要求必须由画面中可见的主体产生（"sound effects must be generated by a visible subject"）；
3) music（音乐）。
【第四类：全局音频】不构成独立事件的声音（环境底噪、背景音乐/氛围音）不进 Event 流，而是被归入 Global 流的 "global_audio" 字段。论文原文：「Irrelevant background noise is filtered into the global audio metadata」——即无关背景噪声被「过滤进」全局音频元数据，而非丢弃。这是一个四层分流设计：可见来源的音效 / 有说话人的对白 / 音乐 / 兜底的全局氛围音。
【筛选原则：严格音视频耦合】Event 流的准入条件是「strict audio-visual coupling principle」——只抽取具有直接视觉对应物或主题相关性的音频事件。这条原则实质上是在标注阶段就完成了「画外音源」的剔除与降级：无视觉对应的声音不会成为独立事件，从而保证 Event 流中的每一条都是可被生成模型学习的、有视觉线索的音频。这是本工作在音频侧最具方法论价值的设计。
【并发音源处理】同时发生的多个音源被「factorized into parallel event entries」（拆分为并行的事件条目），而非合并为一条混合描述。这保证了多声源场景下每条声音都有独立的 time_range、speaker 与 description，可被独立编辑与独立控制。
【比例数字】三类事件在 500K 数据集中各占多少、平均每个片段含多少条 event、dialogue 类事件占比多少，论文均未披露。[不确定]
【采样权重】未提及任何按音频类别调整训练采样权重的机制。

### [Seedance 2.0 与 Seedance 1.5 pro](../models/Seedance_20_Seedance_15_pro.md) ⚠️

[不确定] 训练侧的音频类别配比未披露。但 Seedance 1.5 pro/2.0 的评测标注体系高度细化，可视为其数据类目体系的镜像：SeedVideoBench 1.5 的音频标签主类为 —— 人声类型（Human Voice Types：语音 speech、歌唱 singing、非言语发声 non-verbal vocalizations 如笑声，并含细粒度子维度）；人声属性（Human Voice Attributes：音色 timbre、口音 accent、情绪基调 emotional tone）；非语音音频（Non-Speech Audio，含音效与音乐，按声源如动物/机械工具、声学属性、音乐流派、技术参数分类）。SeedVideoBench 2.0 的音频细粒度类目扩展为 17 类：中文方言/口音、中文多人对话、中文综艺人声、中文戏曲、英语、少数民族语言、歌唱/说唱、空间场景音、画外音（off-screen voice）、非言语人声、人声+动作交互音、物体交互音、动物叫声、环境/背景音、特殊音效（含 ASMR）、乐器与音频、双声道音频。Seedance 2.0 明确支持背景音乐 / 环境音效 / 角色配音三类音轨的多轨并行输出。静音样本比例、语音:音效:音乐的具体配比均未披露。

### [SkyReels 系列](../models/SkyReels.md) ⚠️

仅 SkyReels-V4 涉及，策略明确但无占比数字。
【分类体系】音频被显式分为四类：音效（sound effects / SFX）、音乐（music）、语音（speech）、歌唱（singing）——「歌唱」被单列为独立一类，是相对多数音视频模型（通常只分语音/音乐/音效三类）的细化点，直接服务于短剧与音乐场景的口型同步。
【分类工具】使用 Qwen3-Omni 全模态大模型对音频统一做类别判定，同时也由它统一生成音频 caption。
【类别驱动的处理差异】(1) 时长凑整时「按类别拼接（concatenate short clips by category）」，即同类音频才允许拼接，避免跨类混杂；(2) 语音与歌唱两类额外走 Whisper 转写，音效与音乐不转写；(3) caption 中以 <sfx>、<dialogue>、<singing>、<bgm> 四种特殊 token 分别承载，与四分类一一对应。
【未披露】四类各自的样本量与占比、静音样本保留比例、类别配比目标值。已知音频主干预训练「以语音为主（primarily speech data）」的数十万小时数据，说明语音类占绝对多数，音效/音乐/歌唱为少数类。[不确定]

### [Sora 2](../models/Sora_2.md) ⚠️

完全未披露。作为原生音视频模型，Sora 2 在能力上明确覆盖四类音频——对白（dialogue）、音效/foley（sound effects tied to on-screen actions）、背景音乐（background music matched to scene tone）、环境音/氛围声（context-aware ambient sounds / background soundscapes），且宣称声音的音量与空间定位随物体与镜头的距离变化。但训练数据中这四类音频各占多少比例、无音轨/静音片段如何处理与保留、是否对语音-音效-音乐做显式配比控制，OpenAI 未给出任何信息。这是本次调研中信息缺口最大的维度之一：模型明显具备该能力，但数据侧构造方式零披露。[不确定]

### [Step-Video-T2V](../models/Step-Video-T2V.md) ⚠️

不适用。Step-Video-T2V 不生成音频，训练数据 pipeline 完全不涉及音轨的提取、分类与配比，报告中无语音/音效/音乐/环境音/静音的任何比例控制内容。若需阶跃星辰体系内的音频数据方法，应转向独立的 Step-Audio 系列（语音交互模型，含情绪、方言、语种、歌声等维度的数据构建），但其与视频模型无数据共用关系。[不确定]

### [UniTalking](../models/UniTalking.md) ⚠️

音频类别配比在 UniTalking 中是被「过滤到单一类别」而非「按比例配置」的——这是其与通用 AV 模型最本质的数据差异：
【目标类别：仅语音】音频过滤环节明确剔除三类样本：静音（muted）、无语音（lack speech）、低信噪比（low SNR）。用 PANNs（大规模预训练音频神经网络，AudioSet 527 类事件分类器）与 SentenceASD 模型执行「语音事件检测」（speech event detection）。因此最终 230 万条样本理论上全部含有有效人声语音，音效与音乐不作为独立类别保留。
【非语音成分的处理：剔除而非分离】跨模态过滤阶段进一步「剔除仅含非画面内声源音频（purely diegetic audio）的视频」，使用 LightASD（轻量级主动说话人检测模型）判定。注意论文原文用词为 diegetic（叙事内声音）但语境实为剔除「声音不来自画面中人物」的样本，即画外音、旁白、配乐主导的片段被淘汰。这与 UniVerse-1「不做画外音源剔除」形成鲜明对比，是 UniTalking 在音频侧最有针对性的一道过滤。
【背景音乐】未做源分离（source separation），也未单独设阈。但存在两条间接证据表明背景音乐仍部分存留于训练数据中：其一，caption 中含有音频描述字段，能描述背景声学环境；其二，实验部分展示了用文本 prompt「without any background music」控制生成结果的能力——这种可控性只有在训练数据中同时存在「有背景音乐」与「无背景音乐」两类样本、且 caption 如实标注了这一差异时才可能习得。这是一个「不过滤、改为标注可控」的处理思路（与 MOVA 用「This video has no subtitles.」标记字幕的思路同源）。
【非语音音效】Figure 4 的展示样例中出现了 prompt「a short laugh」（一声短促的笑）并被正确生成，说明笑声等副语言（paralinguistic）音效在数据中有保留并被 caption 覆盖。
【未披露的配比】语音在总时长中的占比、背景音乐样本占比、静音段占比、说话人数量分布等全部空白。所有过滤均无阈值数值。[不确定]

### [UniVerse-1](../models/UniVerse-1.md)

音频类别配比是 UniVerse-1 与 MOVA 分道扬镳的关键维度——UniVerse-1 刻意保留了非语音数据的主导地位：
【三段式配比（按小时数）】
- 语音（speech-centric，经 Whisper 检出语音 + RetinaFace 人脸检测 + SyncNet conf>2.0 三重核验）：1,187 小时，占 15.4%；
- 通用音视频（general，即 Whisper 判定为无语音的片段，涵盖环境音、音效、器乐等）：3,074 小时，占 40.0%；
- VGGSound + AudioSet（音效/事件音专用补强）：3,422 小时，占 44.5%。
即非语音数据合计占 84.6%，语音仅占 15.4%。
【分类机制】用 Whisper ASR 做二分类闸门：能检出有效语音内容的片段进入语音子集并进一步做人脸与唇同步核验；检不出语音的片段不丢弃，而是降级为「通用音视频数据」保留。这是一个「不浪费」式设计——同一条过滤规则同时完成分流与保留，而非 MOVA 式的直接淘汰。
【静音处理】通过音量（volume）、能量（energy）、过零率（zero-crossing rate）三项信号级指标做音频活动检测，剔除静音片段。因此训练集中不含静音样本。
【音乐类别】音乐能力主要由基座 Ace-step（本身是音乐生成模型）承载，训练数据侧则由音乐综艺与古典演奏素材强化，但未单列音乐小时数。
【未披露】通用音视频子集内部（环境音/音效/器乐/背景音乐）的细分比例未给出；各类别的采样权重是否在训练时被调整亦未说明。
【与 Verse-Bench 的对照】评测侧给出了明确的九类音频占比：自然环境 36.1%、音乐与乐器 19.3%、日常生活 10.2%、人声 7.6%、交通 4.5%、工业与城市 3.9%、武器与爆炸 2.5%、特效音 2.3%、未分类 15.8%。评测集同样是自然环境音主导、人声仅 7.6%，与训练配比取向一致。

### [Unison](../models/Unison.md) ⚠️

音频类别是 Unison 全部数据设计中唯一被认真对待的分布维度——因为整个架构就是围绕「语音 vs 音效」的二分建立的。但即便如此，论文仍未给出任何比例数字。
【架构层的强制二分类：语音 / 音效】所有音频（无论来自音视频数据还是纯音频数据）一律经 Mel-RoFormer 分离为 speech 与 sound-effect（SFX）两路，分别编码为 ground-truth latent，再由两个独立的 flow-matching 损失分别监督。论文原文：「we leverage Mel-Roformer to disentangle mixed audio into high-fidelity speech and sound-effect (SFX) components」。这不是「分类后分别处理」，而是「每一条样本都被拆成两条流同时处理」——即使某条样本无语音，其 speech 流也存在（内容为空/近静音）。这是与 UniVerse-1（Whisper 二分类做分流闸门，语音/非语音走不同路径）、MOVA（EAT 分类器做 speech/non-speech 判定）完全不同的处理范式：Unison 不分流，而是并行双轨。
【纯音频语料的四类分工（按数据源可确定类别归属，但无比例数字）】
- 语音（speech）：内部语音数据（internal speech data）；
- 音效（sound effects）：YouTube-8M、AudioSet、WavCaps；
- 音乐（music）：VidMuse；
- 歌唱（singing）：YuE collection。
论文明确将 singing（歌唱）单列为独立类别，与 speech 和 music 并置——这在同类工作中较为少见，直接对应其定性结果中「Universe-1 与 UniAVGen 难以区分歌唱与普通说话」这一被点名的失败模式。Unison 专门引入 YuE 歌唱语料正是为解决该问题。
【静音处理】未提及静音检测、静音占比阈值或无音轨剔除。[不确定]
【比例控制策略】完全未披露。四类音频各自的小时数、条数、采样权重、是否在训练中重加权，均无说明。5,000 万段/130,000 小时的总量如何在语音/音效/音乐/歌唱四类间分配，是本字段最关键的信息空白。[不确定]
【运行时的动态平衡机制（非数据侧但功能等价）】值得记录的是，Unison 把「语音与音效的比例控制」从数据配比问题转化为了推理时的可学习门控问题——SCG 根据 caption 与 transcription 的全局语义向量预测两个门控系数（sigmoid 约束），在叙述主导场景抑制音效流入侵语音流，在复杂声景场景放大跨流影响。Fig. 8 的分析显示该门控具有层级（深层极化加剧）、时间步（去噪推进时门控发散加剧）、实例（跨语义类别自适应）三个维度的动态性。这是一条「配比不靠数据、靠模型自适应」的技术路线，与传统的数据配比控制形成有趣对照。

### [Veo 3 / Veo 3.1](../models/Veo_3_Veo_31.md) ⚠️

[不确定] 音频类别配比与控制策略完全未披露，这是 Veo 3 数据披露中最大的空白之一。可反推的线索：(1) 模型能力覆盖对白（dialogue）、音效/foley（sound effects）、环境音（ambient noise）与背景音乐（music）四大类，说明训练配对音视频数据在这四类上均有相当覆盖；(2) 官方强调 Veo 3 解决了此前视频模型的「无声电影」问题，说明训练时必须显式排除无音轨或音轨与画面无关的样本，但排除标准与静音样本占比未披露；(3) 技术报告在 deepfake 评估中提到 Veo 3 生成的深度伪造「在语音方面尤其难以控制」（much less controllable - particularly with respect to speech），暗示语音数据在说话人身份/音色维度上未做细粒度标注或条件化；(4) Model Card 未提及任何音源分离（如 BGM 与人声分离）、SNR 阈值或静音占比阈值。语音/音效/音乐/环境音的具体配比数值无任何公开依据。

### [Vidu S1](../models/Vidu_S1.md) ⚠️

未给出定量比例 [不确定]，但有明确的音频类别处理与筛选策略，方向是「纯化语音、剔除音乐」：
(1) 从原始音频中先提取语音成分（extract the speech component from the raw audio）；
(2) 通过 VAD（语音活动检测）+ ASD（主动说话人检测）标注每个语音段的时间戳与所属说话人；
(3) 将每段语音分为 onscreen（说话人即画面中人物）/ offscreen（画外音，说话人不在画面中）/ overlap（多人声重叠）三类；含 overlap 段的 clip 直接整条剔除；
(4) 针对唱歌与强背景音乐场景 diarization 模型不稳定（易把人声误判进音乐 stem、产生合成音色与失真伪影）的问题，引入启发式规则：若说话人在发声但语音能量占比过低（speech energy proportion too low），则丢弃该段——实际效果是系统性剔除歌唱与音乐主导片段。
(5) caption 层面仍标注 sound effects（音效）与 background music（背景音乐）字段，说明这两类音频信息被保留为可描述属性，但语音仍是训练分布的绝对主体。

### [主流视频预训练数据集合并调研：Panda-70M、InternVid、Koala…](../models/pretraining_datasets.md)

不适用。七个数据集均为纯视觉+文本数据集，不生成也不处理音频，无语音/音效/音乐/环境音的分类、配比或统计。仅 Panda-70M 与 UltraVideo 的视频文件被动保留了原始音轨（前者通过 download_audio:True，后者声明保留 native audio），但均无任何音频类别标注或配比控制；Koala-36M 的重切分代码则会直接丢弃音轨。

## 叙事结构分布（单镜头vs多镜头、平均clip时长、镜头数分布、是否含原生音轨）

`narrative_structure_distribution` · 详细程度: brief

### [Allegro](../models/Allegro.md)

全部为单镜头（single-shot）片段，无多镜头叙事结构：原始长视频经 PySceneDetect 切分为「单场景 clip」后才进入训练集，论文明确说明训练素材为 single-shot camera footage。
平均 clip 时长落在 2–16 秒区间（微调阶段 6–16 秒），训练时统一采样为 40 帧或 88 帧（88 帧 @15FPS ≈ 6 秒）。
不含原生音轨（音频被丢弃）。
镜头数分布不适用（恒为 1）；论文未做多镜头/故事板类数据的构造。

### [Apollo](../models/Apollo.md) ⚠️

【单镜头 vs 多镜头】明确采取纯单镜头策略：论文原文「We then apply scene splitting to ensure each sample contains only one scene」（应用场景切分以确保每条样本只包含一个场景）。即训练数据全部为单镜头片段，不包含跨镜头转场样本。这与 MOVA 刻意构造「单场景/多场景」2×2 数据维度的做法形成对比，也意味着 Apollo 未在数据侧建模镜头转场能力。
【原生音轨】训练数据必须自带原生同步音轨——整个 pipeline 的音频过滤与音视频一致性检测环节均以原始音轨为对象，无音轨或音轨异常的样本在过滤阶段被剔除。
【平均 clip 时长 / 镜头数分布】未披露。[不确定]（时长数字缺失，单镜头结论确定）

### [CineDance / CineDance-1M](../models/CineDance.md) ⚠️

叙事结构是本数据集的立身之本，也是论文着墨最多的部分：
【多镜头】平均 24.2 个连续镜头/序列，全部为多镜头样本，与 LVD-2M（1.86 镜头）、SpeakerVid-5M（1.27 镜头）等以单镜头为主的数据集形成代际差。
【平均序列时长】92.8 秒；对比 MiraData 72.1s / 7.15 镜头、LVD-2M 20.2s / 1.86 镜头。
【原生音轨】全量保留原始同步音轨，为对比表中唯一同时具备 1080p + 原生音轨 + 镜头级双模态密集标注的数据集。
【叙事序列的定义】「diegetic time and causality 的连续流」——即在戏剧时间与因果链上连续、且角色与环境状态保持延续的一段镜头组合，允许空间跳转但不允许叙事断裂。
【四条电影理论解析规则】① Multi-angle shots（多角度镜头）：仅机位角度变化而事件统一；② Cross-cutting（平行剪辑）：在不同空间间快速交替但被统一的因果张力绑定；③ Causal action / ellipsis（因果动作与省略）：存在时空跳跃但后续事件可直接、可解释地承接前一事件；④ Montage（蒙太奇）：镜头彼此割裂但被宏观主题或情绪弧线统一。
【镜头级属性】每个镜头标注五维属性：景别（scale）、机位角度（angle）、运镜（movement）、叙事功能（narrative function）、时长类别（duration category），外加转场类型（shot transition type）。
【分布细节】Fig.5 给出时长-镜头数联合分布，但镜头数的具体分位数未文字化。[不确定]

### [CogVideoX](../models/CogVideoX.md) ⚠️

· 全部训练片段为单镜头（single-shot）：论文明确「approximately 35M single-shot clips」，平均 6 秒。多镜头叙事不在建模范围内，模型输出为单镜头连续视频。
· 保证单镜头的机制不是场景切分器，而是负面标签分类器：「Lack of Motion Connectivity」专门识别「转场处运动不连贯、常见于人工拼接的视频或由静态图剪成的视频」，把含镜头切换/拼接的片段整体判负剔除；「Editing」标签进一步剔除带转场特效的素材。
· 镜头数分布、平均镜头数等统计未给出 [不确定]。
· 是否含原生音轨：不适用，视频侧训练完全丢弃音轨。
· caption 层面包含镜头运动（camera movements）描述，但不含多镜头结构化字段。

### [Cosmos-Predict2.5](../models/Cosmos-Predict25.md)

训练片段全部为单镜头（single-shot）：pipeline 第一阶段即用高精度镜头边界检测模型把长视频切为 shot，并明确「ensuring that raw shot transitions are excluded」（确保原始镜头转场被排除在外）；后续「semantic artifacts filter」（类 VTSS）进一步剔除 poor transitions（劣质转场）与 video-in-video（画中画）等结构异常样本。因此数据集内不含多镜头叙事样本，也不存在镜头数分布概念。
单 clip 时长 5–60 秒，caption 按 5 秒窗口切分标注；模型单次生成 93 帧 / 约 5.8 秒。长时程能力靠 Video2World 条件续写与 Cosmos-Transfer2.5 的 long-horizon 视频翻译实现，而非训练长片段。
多视图是该工作特有的「非叙事型结构维度」：自动驾驶数据为 7 路同步相机的环视 clip（30 FPS、20 秒），多视图模型把 7 个视角沿 latent 时间维拼接（latent 时间维压缩至 8 以容纳 7 视角），机器人 AgiBot 变体为 3 相机视角。
原生音轨：不涉及（无音频处理）。

### [Data-Juicer 2.0](../models/Data-Juicer.md) ⚠️

【单镜头 vs 多镜头的处理能力】DJ 通过 video_split_by_scene_mapper（场景变化检测切分）提供了把多镜头长视频拆解为单镜头 clip 的标准手段，这是控制叙事结构最直接的杠杆——启用该算子即意味着训练数据全部为单镜头；不启用则保留原始的多镜头结构。DJ 把这一选择开放给使用者，不做默认取向。
【官方案例的实际结构】T2V 案例的三个源数据集（InternVid、Panda-70M、MSR-VTT）本身均已是切好的单镜头或近单镜头短片段，案例中未再做场景切分，因此最终数据池以单镜头短 clip 为主。
【是否含原生音轨】案例未做要求也未做统计；DJ 支持处理带音轨与不带音轨的视频，video_split_by_scene_mapper 等切分算子会同步处理音轨。
[不确定] 未报告任何镜头数分布、平均 clip 时长、含音轨样本占比的统计；DJ 也未提供「镜头数统计」或「叙事结构画像」类的分析算子（其数据分析报告主要围绕单值统计量的直方图，而非结构化叙事属性）。

### [Foley-Omni](../models/Foley-Omni.md) ⚠️

整体属于单镜头短片段范式。从 5–10 秒的片段时长、motion score 上界 3.2 对剧烈运动/快切的抑制、以及数据来源（GRID/LRS2/Chem/SpeakerVid/TalkVid 为连续说话人镜头，VGGSound 为单事件10秒片段）综合判断，训练与评测数据基本都是单镜头、无剪辑点的连续片段。这与其任务性质匹配：跨镜头的配乐连贯性建模不是本文目标。
【是否含原生音轨】全部含原生音轨，且原生音轨是唯一的监督目标（生成目标即还原/重建该音轨的三类成分）。这一点与文生视频模型「音轨可有可无」形成对比——对 Foley-Omni 而言无音轨样本直接被过滤掉，原生音轨质量（AudioBox Aesthetics ≥0.6）是硬门槛。
[不确定] 未给出镜头数分布统计、未给出平均clip时长的精确数值、未说明是否做过跨镜头检测与剔除。

### [Goku](../models/Goku.md)

以单镜头（single-shot）片段为绝对主体。切分阶段的设计目标即为「消除镜头切换」：先用 PySceneDetect 做镜头边界检测，再用 DINOv2 特征余弦相似度做二次校验（480×864 档要求相邻帧 DINO 相似度 ≥ 0.85，720×1280 档要求 ≥ 0.90），双重机制确保每个训练片段内部视觉连续、不含转场。因此镜头数分布可视为「几乎全部为 1 个镜头」，不存在多镜头叙事结构建模。
平均 clip 时长落在 4~10 秒区间（具体均值未披露）。
是否含原生音轨：否（数据形态为纯视频-文本对）。
论文未涉及多镜头叙事、故事板、长视频连续性等结构化叙事数据设计。

### [HunyuanVideo-Foley](../models/HunyuanVideo-Foley.md) ⚠️

【单镜头为主，由切分策略决定】流水线先用场景检测切分镜头、再切 8 秒块，因此绝大多数训练片段落在单个镜头之内。但与 UniVerse-1「切分后剔除 <5 秒片段」不同，本工作是把镜头段规整为 8 秒块——若某个镜头长于 8 秒会产生多个同镜头块，若短于 8 秒则处理方式未说明（可能丢弃，也可能与相邻镜头拼接从而产生跨镜头样本）。论文未澄清这一点。[不确定]
【平均 clip 时长】严格 8 秒定长，无分布可言。
【镜头数分布】不适用（设计上为单镜头）。
【是否含原生音轨】强制含有。流水线第一步即剔除无音轨视频（"eliminate videos lacking audio streams"），这是一票否决条件，因此全部训练数据均自带原生同步音轨。这对 V2A 任务是根本性要求——模型学习的正是「画面 → 其真实伴随声音」的映射，后配音或替换音轨的样本会引入错误监督信号。
【叙事结构不构成训练维度】8 秒单镜头片段不承载叙事结构，本工作也不涉及镜头转场音效、场景切换的声学过渡等叙事层面的音频设计——这是 Foley 模型与完整电影声音设计之间的能力边界。论文未讨论长视频、多镜头场景下的音效连贯性问题。

### [HunyuanVideo](../models/HunyuanVideo.md)

两代均明确采用「单镜头（single-shot）」数据范式，不做多镜头叙事训练：
【原版】用 PySceneDetect 切分为单镜头片段；另用 Transnet v2 + PySceneDetect 双路提供场景边界信息以提高切分可靠性；dense description 字段中会记录场景转换（scene transitions）与镜头运动。
【1.5】更进一步：在 PySceneDetect 与自研算子切分之后，额外接一个专门的「转场分类器（transition classifier）」，把仍残留转场特效（渐变、叠化等）的片段整体剔除，确保每个训练clip是干净的单镜头。平均clip时长即 2–10 秒区间，镜头数分布无意义（均为1）。原生音轨：无（数据侧不保留音轨）。

### [InstructAV2AV](../models/InstructAV2AV.md)

全部为单镜头（single-shot）结构，无多镜头样本。PySceneDetect 的镜头切分是pipeline第一步，明确产出 single-shot clips，5 秒的定长进一步保证片段内不含剪辑点。
【平均 clip 时长】严格 5.0 秒，无方差。
【镜头数分布】恒为 1，无分布。
【是否含原生音轨】source 侧全部含原生音轨且为必需——无音轨或静音（<-45 dBFS）的片段在过滤阶段即被剔除。target 侧音轨为合成产物（分离出的背景音 + 新合成的目标声）。这构成本数据集的一个特殊属性：每对样本的两侧音轨共享同一背景音成分，仅前景声不同，是精确受控的差分对。
【为何必须单镜头】这不是审美选择而是技术必需：Grounded-SAM-2 的实例 mask 需要在片段内连续传播，CoTracker3 的点跟踪需要连续视野，mask-guided 视频编辑模型需要在稳定的时空上下文中做局部替换——任何镜头切换都会让这三者同时失效。
【局限】由此带来的能力边界很明确：模型无法处理跨镜头的编辑一致性（如「把这个人在整部片子里都换掉」），也无法编辑含剪辑的叙事段落。这与论文局限性中承认的「大幅相机运动下难以保持 3D 空间一致性与物体恒常性」是同一问题的两个侧面。

### [2026 其他音视频联合生成](../models/JAVG_2026_misc.md) ⚠️

【单镜头为主，长时叙事仅 StreamChar 一家涉及】
(1) 单镜头短片：OmniCustom（5 秒定长单人说话，单镜头）、CCL（352×640 短片段）、Baton（短片段，评测为 10 秒）、NAVA（平均 7 秒，经 Hadoop 管线切分，应为单镜头片段）、ALIVE（3–10 秒片段，输出 5–10 秒）均为单镜头结构，无镜头切换与转场样本。
(2) 长源→短片的抽取：ALIVE 的 character-driven pipeline 明确从「10–30 分钟长视频」中抽取 N 个 3–10 秒片段——即长视频被当作「同一身份的多镜头素材池」使用，用于构造跨片段的身份配对（cross-pair），这实际上间接引入了「同一角色跨镜头一致性」的监督信号，虽然单个训练样本仍是单镜头。
(3) 长时叙事的唯一探索者：StreamChar 是本批中唯一正面处理长时序结构的工作，但其解法不是用长视频数据训练——恰恰相反，训练数据被限制在 20 秒以内（「training data contains no videos/transcripts longer than 20 seconds」），长时能力完全通过推理时的 chunk 自回归拼接（每 chunk 33 帧 @24fps）+ 两个显式抗漂移机制获得：progress-aware pointer（用 ASR 时间戳的 ground-truth end indices 做 smooth L1 监督，让模型知道台词念到哪了）与 persistent visual anchor / sink chunk（持久视觉锚点抑制身份漂移）。消融显示去掉 sink chunk 后 drift 从 0.0067 恶化到 0.0304（劣化约 4.5 倍），量化证明了视觉锚点对长程一致性的关键作用。评测中生成 5 分钟连续流（用 >300 words 的随机台词）。
(4) 原生音轨占比：七项全部使用带原生同步音轨的配对数据——ALIVE 明确「begins by filtering videos with audio from the raw data pool」（第一步就是从原始池中筛出带音轨的视频），OmniCustom/StreamChar 的上游数据集本身即音视频配对，NAVA 的音视频子集同理。无一采用「先剥离音轨再后期配音」的路线。
(5) 镜头数分布、平均镜头数等统计七项均未披露[不确定]。

### [音视频联合生成基线合集](../models/JavisDiT_baselines.md) ⚠️

全部为「单镜头、短片段、含原生音轨」的最简设定，无一涉及多镜头叙事：
【镜头数】五者训练样本均为单镜头片段。MM-Diffusion 把 928 条源视频机械切成 1,000 条 10 秒不重叠片段（未提及镜头检测，切分点可能跨镜头，是早期数据处理的粗糙之处）；JavisDiT 依赖 TAVGBench 已有的片段划分；Harmony 的 3–10 秒片段与 UniAVGen 的真人片段同样为单镜头。
【平均 clip 时长】MM-Diffusion 数据集 10 秒、实际训练片段更短；AV-DiT 16 帧 + 1.6 秒音频；JavisDiT/JavisDiT++ 固定 4 秒；Harmony 3–10 秒（区间分布，本合集中唯一非定长）；UniAVGen 未披露[不确定]。
【原生音轨】五者训练数据 100% 含原生同步音轨——这是音视频联合生成的前提条件。MM-Diffusion 的 AIST++ 是个特例：其音乐是与舞蹈配套的伴奏而非现场录音，属于「制作时即对齐」而非「拍摄时同步采集」。
【叙事能力】五者均无镜头转场、多镜头一致性、故事结构等设计；Harmony-Bench 的「复杂场景」子集是最接近叙事复杂度的尝试，但复杂性体现在「语音与环境音共现」的声学层面而非镜头叙事层面。
【与工业模型的差距】相比 Veo 3 / Sora 2 / LTX-2 等已开始处理多镜头与分钟级叙事的模型，本合集全部停留在「单镜头 ≤10 秒」阶段，这既是学术算力限制的结果，也意味着这些基线的数据处理经验难以直接迁移到长视频场景。

### [可灵 3.0 / 可灵视频 3.0 Omni](../models/Kling_30_Omni.md) ⚠️

[不确定] 未公布单镜头vs多镜头样本比例与镜头数分布。可推断的结构性事实：(1) 基础清洗阶段会剔除含突兀转场/不连贯镜头切换的片段，说明预训练主体是单镜头连续clip；(2) 产品支持“智能分镜”与单条最多6个镜头、跨镜头角色与音色一致（Director Memory），意味着后训练阶段必然引入了多镜头/分镜级的长上下文样本（含镜头边界标注与跨镜头身份/音色对应关系）；(3) 原生音画同出要求训练样本自带原生音轨，因此高质量子集应以“有原生同期声”的片段为主，无音轨样本被剔除或仅用于纯视觉阶段。上述(2)(3)为基于产品能力的推断，报告未明确记述。

### [LTX-2](../models/LTX-2.md) ⚠️

未披露单镜头 vs 多镜头配比、平均镜头数分布。可确定的是数据组织粒度为单镜头：pipeline 从「Input Shots」到「Final Shots」全程以 shot 为单位，因此训练样本以单镜头 clip 为主，模型也不以多镜头叙事为卖点（论文将「更深层的叙事连贯性」明确列为需依赖外部 LLM 生成条件文本来弥补的局限）。是否含原生音轨：LTX-2 子集按定义全部为自带且音频信息量显著的原生同步音轨样本，这是其区别于母集的关键。平均 clip 时长参见 LTX-Video 时长直方图（0–30秒）。[不确定]

### [LongCat-Video](../models/LongCat-Video.md)

训练数据经场景检测切分后为单镜头片段（single-shot clips）——PySceneDetect + TransNetV2 的作用正是消除镜头切换，保证每个训练片段内容连续一致，因此训练集不含多镜头叙事样本，也没有镜头数分布的概念。平均 clip 时长未披露，但训练时统一采样为 93 帧（30fps 约 3.1 秒）。原生音轨：基础版训练完全丢弃音轨（纯视觉）；Avatar 系列则依赖原生音轨（音视频成对且需通过唇同步校验）。
多镜头/长叙事能力不在数据层解决，而在推理层通过 Video-Continuation 递归续写实现：以多个前序帧为条件帧连续生成，配合针对跨帧时序一致性与物理运动合理性的优化，抑制色彩漂移、画质衰减与运动断裂，从而稳定输出分钟级长视频。Avatar 1.5 在 GRPO 阶段支持最多 5 个 clip 的多段 rollout（仅最后一个 clip 参与优化），侧面反映其长视频是多段拼接范式。

### [MOVA](../models/MOVA.md)

MOVA 显式地把“单镜头 vs 多镜头”做成了一个正交的数据维度，这在同类工作中较为少见：
- 通过 VAD 时间信息与 PySceneDetect 场景切点的交叉组合，生成四类片段：单场景语音、单场景非语音、多场景语音、多场景非语音（即 {单/多镜头} × {语音/非语音} 的 2×2 划分）。
- 多镜头片段的判定条件是硬性的：窗口的时间跨度内必须至少包含一个场景切点，否则丢弃（附录 A.3 Algorithm 1 第 11 行）。
- 单镜头片段的判定条件同样是硬性的：窗口必须完整落在两个相邻场景切点之间（Algorithm 2 第 5、19 行）。
- 平均 clip 时长固定为 8.05 秒（193 帧 @24fps），无长度分布可言。
- 是否含原生音轨：训练数据必须自带原生同步音轨——预处理第一步即剔除“解码失败”或“缺少有效音频通道”的样本，因此不存在无音轨样本。
- 打标环节也针对多镜头做了专门要求：视频 caption 的 prompt 明确指示 MiMo-VL“focusing on video scene transitions”，即显式标注镜头转场。
- 未披露单镜头与多镜头片段的具体比例。

### [合并条目：① Mochi 1](../models/Mochi_MAGI_Motif.md) ⚠️

① Mochi 1：未披露。生成侧固定输出单段 5.4 秒视频，推断训练素材为单镜头短片段，但无官方说明。[不确定]
② MAGI-1：训练片段严格为单镜头——PySceneDetect 切分「ensuring that each clip contains only a single shot」，并额外用 Transition Detection actor 兜底（PySceneDetect 难以处理复杂转场，导致切出的 clip 仍可能含多镜头；因此稀疏采关键帧、用 CLIP 计算相邻关键帧语义相似度，低于阈值即判为多镜头并整段剔除）。平均 clip 时长按阶段为 ≤8s / ≤8s / ≤16s。不含原生音轨。
值得单列的是：MAGI-1 虽然训练素材是单镜头，却通过自回归 caption（见 caption_structure）和 chunk-wise 文本条件，在推理侧获得了多镜头叙事能力——技术报告 Fig.10 展示了一个近 30 秒、由 (a)–(g) 七段不同 prompt 逐段驱动的复杂动作与叙事结构生成示例，Fig.12 还展示了通过在不同去噪阶段调制 KV range 来实现「保持身份的镜头切换」与「保持场景布局但改变物体细节的转场」两类镜头转换。也就是说，多镜头叙事在 MAGI-1 中是被「架构 + caption schema」而非「多镜头训练数据」实现的，这一点在数据侧对照上很有意思。
③ Motif-Video 2B：同样严格单镜头，且切分策略更精细：先用「偏向过切分（宁可假阳性，不可漏检转场）」的保守阈值检测场景边界，再用 SigLIP embedding 相似度做 stitch detection 把因瞬时运动或曝光变化而被误切的连续镜头重新合并，最后丢弃合并后 <2 秒的片段。此外 VLM 产出的 multi_scene 标签被用作「对场景切分的二次检查」，命中即丢弃。不含原生音轨。镜头数恒为 1，无多镜头叙事数据构造。

### [Movie Gen](../models/Movie_Gen.md)

视频训练数据全部为单镜头（single-shot）片段：通过 FFmpeg 场景边界检测 + PySceneDetect 抖动检测保证clip内无转场、无跳剪，且要求存在非平凡运动（non-trivial motion）；每条原视频只取1~2个场景、每场景1个clip，平均clip长度落在15~16秒区间（>50%）。因此模型是单镜头生成器，不建模多镜头叙事。
是否含原生音轨：Movie Gen Video 的训练完全不使用音轨（纯视觉+文本）；Movie Gen Audio 的预训练则依赖视频的原生音轨作为监督信号。两者数据流是分离的。
多镜头仅出现在评测侧：Movie Gen Audio Bench 把评测视频分为 single-shot（数量多、音效谱系广，测鲁棒性与泛化）与 multi-shot（取自短片，含场景转换、情绪与叙事更强，用于评测视频-音乐对齐，如音乐何时进入、如何随剧情演进、是否与剪辑点对齐、音效与音乐的混音是否和谐）。

### [NVIDIA NeMo Curator](../models/NeMo_Curator.md) ⚠️

【单镜头 vs 多镜头】框架的默认产物是单镜头 clip：TransNetV2 做 shot-aware splitting，产出的每个 clip 原则上不跨镜头。但 pipeline 中有一个值得注意的反向设计——stitching（缝合）阶段：在切分之后，用图像嵌入相似度判断相邻 clip 是否内容连续，若相似度高则将其重新合并，以避免把同一连续场景过度碎片化（如摄影机短暂遮挡、闪光导致的误切）。这是「切分—再缝合」的两段式设计，比单纯依赖场景检测更稳健。
【镜头数分布】最终数据集以单镜头 clip 为主，多镜头叙事样本不是构建目标；镜头数分布无统计数字。
【平均 clip 时长】区间 2–60 秒，具体均值未公布。
【是否含原生音轨】pipeline 不保留、不处理音轨，转码输出为 H.264 视频，故最终 WebDataset 中不含音频轨道信息。[不确定]

### [OmniHuman 数据集 + OHBench](../models/OmniHuman.md) ⚠️

【单镜头为主，由切分保证】使用 TransNetV2 做镜头切分后，每个 clip 对应一个连续镜头，因此训练样本本质为单镜头片段。结合平均约 6.5 秒的片段时长，数据不承载跨镜头叙事。
【但保留了镜头级元数据】虽然样本本身是单镜头，标注却包含 shot scale（景别，如特写/半身/全身/远景）与 camera motion（相机运动）字段——这使得「镜头语言」成为可控条件而非被抹除的信息。评测中「模型在特写→远景的镜头切换上性能下降」这一发现，正是依托景别标注才能诊断出来的。
【平均 clip 时长】约 6.5 秒（由 1800h / 1M 反推），分布见 Figure 3(d)，正文无数值。[不确定]
【镜头数分布】不适用（每样本单镜头）。
【是否含原生音轨】强制含有。音频治理阶段剔除「音轨缺失」（missing tracks）的样本，且 SyncNet 归属校验要求音轨与画面中人物对应，因此所有样本均含与画面同源、同步的原生音轨。
【交互结构作为叙事替代维度】本数据集不追求跨镜头叙事，而是在单镜头内追求「关系结构」的丰富度——双人对话（含说话人轮替、倾听者反应）与人-物交互构成了其独特的结构性内容。这实际上是把叙事复杂度从「时间轴上的镜头组接」转移到了「空间内的多主体关系」，是与 MuSS 等多镜头数据集正交的另一条路线。

### [Open-Sora 系列](../models/Open-Sora.md)

两个项目的数据均为**严格的单镜头（single-shot）片段**，这是其 pipeline 的核心设计取向：
- Open-Sora 2.0 用 FFmpeg libavfilter 的 scene score 做镜头边界检测并切分，再叠加基于 PySceneDetect 的相机抖动/跳变检测二次剔除，确保片段内无镜头切换；长片段被机械切成 8 秒段。
- Open-Sora 1.x 用 PySceneDetect 检测场景并切分，输出命名为 `{video_id}_scene-{scene_id}.mp4`，同样是单场景片段。
- Open-Sora Plan v1.3 在 16 秒切分之后专门加了一级 **LPIPS 跳切检测（jump cut detection）**，用感知距离突变识别切换点，保留率 97%，明确目标就是滤除含镜头切换的片段。
因此：多镜头叙事样本被主动剔除而非保留，训练数据中平均镜头数=1，不含跨镜头一致性监督信号，模型也不具备多镜头叙事生成能力。平均 clip 时长：Open-Sora 2.0 偏向 6–8 秒，Open-Sora Plan 约 1.3–21 秒（16 秒切分后按帧数区间保留）。是否含原生音轨：数据本身有音轨但被 pipeline 完全忽略并在转码中丢弃。

### [Ovi](../models/Ovi.md) ⚠️

【单镜头为主】通过场景检测（scene detection）切分，切分粒度本身保证每个 121 帧片段位于单一镜头内，因此训练样本几乎全为单镜头（single-shot）片段，不含镜头切换。这与论文 Limitations 中「inter-shot transitions 与全局故事一致性不在范围内」互为印证。
【平均 clip 时长】固定 5.04 秒（Ovi 1.1 为 10 秒），无长度分布——所有样本等长。
【镜头数分布】每样本恒为 1 个镜头。
【原生音轨】100% 含原生音轨——音视频配对语料的全部价值就在于原生同步音轨，且经过 SyncNet 与音量门槛过滤，无音轨或近似静音的片段已被剔除。这与部分模型「先剥离音轨再后期配音」的做法相反。
【叙事能力的来源】论文称模型能做「cinematic storytelling」，但叙事性来自 caption 中按时序交织的视觉事件与台词标注（chronology 被显式要求），而非多镜头数据。

### [Script-a-Video](../models/Script-a-Video.md) ⚠️

叙事结构是 MTSS 的一等公民，schema 层面处理充分，但分布统计缺失：
【单镜头 vs 多镜头】明确同时支持且显式建模。Shot 流本身即为「把视觉呈现分解为一串电影化片段（a sequence of cinematic segments）」，每个 shot 带独立 time_range。评测集按 125 条单镜头 / 100 条多镜头（约 5.6:4.4）划分，说明多镜头是一等评测场景而非附属。
【镜头数分布】500K 数据集中每个片段平均含多少个 shot、shot 数的分布区间，论文未披露。[不确定]
【平均 clip 时长】未披露。
【是否含原生音轨】必然含有——Event 流的构建依赖真实音轨中的对白与音效，Global 流的 global_audio 依赖真实环境音，整套 schema 若无原生音轨则大半字段为空。但论文未显式声明「无音轨视频被剔除」这类前置过滤规则。
【镜头语言的显式建模】Shot 流内设 camera 字段，专门记录专业电影语言：镜头运动（movements）、视角（perspectives）、景别（scales）。这使得镜头语言成为可被结构化检索与可控生成的字段，而非散落在自由文本中。
【叙事连续性的保证机制】跨镜头的叙事连贯由 Relational Grounding 保证：references_in_shot 数组把每个镜头中出现的主体映射到持久的 Reference ID，active_events 把镜头链接到并发的听觉事件。因此同一人物在多个镜头中不需重复描述外观，只需引用同一 ID——这直接解决了单体式 caption 中「重复描述导致的身份漂移」问题。
【实际生成效果的量化】Shot Boundary Deviation 从基线 LTX-2-AV 的 3.79 帧降至 0.38 帧（Ours w/o ID 配置），人评 MSC（多镜头一致性）从 1.00 提升至 2.49-2.62，证明 Shot 流提供的时序分段先验确实使多镜头可控性成为可能。

### [Seedance 2.0 与 Seedance 1.5 pro](../models/Seedance_20_Seedance_15_pro.md) ⚠️

Seedance 1.0 的切分策略明确允许「每个切片可包含一个或多个时序连贯的镜头」（one or multiple temporally coherent shots），以保留局部叙事流，这是其多镜头生成能力的数据基础。Seedance 1.5 pro 强调「叙事连贯性（narrative coherence）」与多镜头视频生成工作流的应用潜力，并具备连续长镜头、希区柯克变焦（dolly zoom）等运镜与电影级转场能力。Seedance 2.0 主打「原生专业级多镜头叙事能力」，可自主规划镜头序列（shot sequencing）并设计视觉呈现模板，SeedVideoBench 2.0 为此新增叙事质量（narrative quality）指标，含摄影语言（镜头逻辑与表现力，检查冗余覆盖、越轴/180 度法则违规、景别错配、节奏不均）、情节设计、风格化美学三个子维度。训练数据原生音轨占比、平均 clip 时长、镜头数分布等具体统计均未披露。[不确定：定量分布]

### [SkyReels 系列](../models/SkyReels.md) ⚠️

【SkyReels-V2】数据处理粒度为单镜头：所有原始视频先经镜头边界检测切为单镜头 clip 后再进入后续流程，因此训练样本以单镜头为主；多镜头/长叙事能力由 Diffusion Forcing 框架在生成侧通过自回归续写实现（非递减噪声调度支持无限延长），而非靠多镜头训练样本。论文的核心卖点之一是「shot-aware（镜头语言感知）生成」，但这体现在 caption 的镜头类型/机位/运镜字段上，而非样本的多镜头结构上。
【SkyReels-V4】明确支持并评测「多镜头（multi-shot）电影级叙事」，SkyReels-VABench 的提示集覆盖「从单镜头到多镜头的复杂度梯度」，并提供视频延长（video extension）与镜头转场能力。原生音轨方面：V4 的音视频联合训练数据要求自带同步音轨并通过 SyncNet 校验，但论文未给出「含原生音轨样本占比」「单镜头 vs 多镜头样本比例」「平均镜头数/平均 clip 时长」等分布数字。[不确定]

### [Sora 2](../models/Sora_2.md) ⚠️

未披露训练数据的镜头数分布与单/多镜头配比。能力侧披露：Sora 2 可「follow intricate instructions spanning multiple shots while accurately persisting world state」，即支持跨多镜头的指令遵循并在镜头切换间保持角色、环境、光照一致，说明训练数据中必然包含多镜头叙事样本且带有跨镜头一致性监督信号。2025年10月推出的 Storyboard（故事板）工具进一步允许用户逐段规划多场景视频。是否含原生音轨：从原生音视频联合生成能力可反推训练数据以「自带同步原声的视频」为主体，但平均clip时长、镜头数直方图、有声/无声样本比例均无数字。[不确定]

### [Step-Video-T2V](../models/Step-Video-T2V.md)

明确采用「单镜头（single-shot）」数据范式，不做多镜头叙事训练：PySceneDetect AdaptiveDetector 检测场景变化后由 FFmpeg 切分，每段仅含一个镜头，且首尾各去 3 帧进一步剔除转场残留。因此镜头数分布恒为 1，平均 clip 长度对应 68/136/204 帧三档（外加 1 帧图像档）。
值得注意的是，dense caption 中会显式描述「镜头运动（camera movements）」，SFT 环节的人工评审标准中也包含「场景转换是否平滑（smooth scene transitions）」一项，说明团队关注镜头语言，但这是在单镜头内部的运镜层面，而非跨镜头的多镜头叙事结构。
原生音轨：无（数据侧不保留、不处理音轨）。

### [UniTalking](../models/UniTalking.md) ⚠️

【单镜头 vs 多镜头】论文未讨论镜头结构，也未描述任何镜头切分环节（见 shot_segmentation）。但可从两条约束间接推断训练数据实质为单镜头：其一，跨模态的唇同步过滤（LipSync）要求画面中持续存在可检测且与音轨同步的人脸，跨镜头切换会破坏这一连续性并大概率导致样本被淘汰；其二，LightASD 主动说话人检测同样依赖连续的人脸轨迹。因此多镜头样本极可能被过滤流程隐式排除，但这是过滤的副作用而非显式设计。[不确定]
【上游数据的镜头属性】OpenHumanVid 在其自身流水线中已完成「解码、裁剪、切分」（decoding, cropping, segmentation）预处理，因此 UniTalking 从中取用的片段本身已是切分后的短镜头单元——这可能正是 UniTalking 无需自建镜头切分模块的原因。华为内部数据是否经过同等处理未说明。[不确定]
【平均 clip 时长与镜头数分布】完全未披露。[不确定]
【是否含原生音轨】强制含有且必须是画面内声源。这是本工作对音轨的要求中最严格的一点：不仅要有音轨（排除无声视频）、不仅要有语音（排除纯环境音）、还必须是画面中人物发出的同步语音（LightASD 排除画外音源 + LipSync 排除唇音不同步）。三重条件串联，比 UniVerse-1 的「有音轨即可 + 语音子集额外查 SyncNet」要严苛得多。
【叙事复杂度】数据为短时单人说话片段，不承载叙事结构。模型也明确不支持多人参考生成（Future Work 中承认不具备 Sora2 的「Cameo」式多片段输入能力）。

### [UniVerse-1](../models/UniVerse-1.md)

【单镜头 vs 多镜头】UniVerse-1 采用「纯单镜头」策略但未显式讨论：PySceneDetect 检测场景切点后按切点切分，仅保留切分结果中长度≥5 秒的片段，因此每个训练片段天然落在单个镜头之内，不存在跨镜头样本。论文未把「多镜头」作为独立数据维度处理，也未生成多镜头训练样本——这与 MOVA 显式构造「单/多镜头 × 语音/非语音」2×2 四类片段的做法形成鲜明对比，也意味着 UniVerse-1 不具备镜头转场的生成能力。
【平均 clip 时长】仅知下界为 5 秒，训练消费窗口约 5 秒；均值与分布未披露。
【是否含原生音轨】强制含有。清洗流水线第一步即「缺少音轨的视频立即丢弃」（Videos lacking an audio track were immediately discarded），因此全部训练数据均自带原生同步音轨，不存在后配音或无音轨样本。
【镜头数分布】不适用（全为单镜头）。
【叙事复杂度】受 5 秒左右的窗口长度限制，训练数据为短时单镜头片段，不承载叙事结构信息。

### [Unison](../models/Unison.md) ⚠️

论文未把叙事结构作为独立维度讨论，无任何相关统计。以下为间接推断：
【单镜头 vs 多镜头】几乎可确定为纯单镜头。五个音视频数据源中，HDTF、VFHQ、CelebV-Text 均为已裁剪的单人脸单镜头片段，VGGSound 为固定短片段，OpenHumanVid 亦以切分好的 clip 形式发布。约 5.4 秒的平均时长也基本排除了多镜头样本的存在。论文未提及任何多镜头样本构造或镜头转场处理，模型不具备镜头切换生成能力。[不确定]
【平均 clip 时长】推算约 5.4 秒（3,000 小时 ÷ 200 万条），论文未直接给出。
【镜头数分布】不适用（推断全为单镜头）。
【是否含原生音轨】音视频侧全部为原生同步音轨——五个数据源均为带声视频，且 lip-filtering 算子显式剔除画外配音（off-screen voice-overs），说明保留下来的语音必须与画面内人物口型对应。这是一条比「有音轨即可」更严格的要求。
【纯音频侧不涉及叙事结构】130,000 小时音频数据无视觉伴随，仅用于 Stage 1 音频分支的单模态训练。
【叙事复杂度评价】约 5 秒的单镜头短片段不承载叙事结构信息，Unison 的目标能力是「短时窗内的多模态精确对齐」而非「长时叙事」。这与其架构选择一致——三帧窗口的交叉注意力（stride=1，仅保留中间帧）是一个极局部的对齐设计，天然面向短片段。

### [Veo 3 / Veo 3.1](../models/Veo_3_Veo_31.md) ⚠️

[不确定] 训练数据中单镜头 vs 多镜头的比例、平均 clip 时长、镜头数分布均未披露。反推线索：技术报告明确记载模型存在「频繁镜头切换与戏剧化机位」的电影化偏好，说明训练数据中包含大量多镜头（multi-shot）剪辑成片，且未被完全切分为单镜头片段，否则模型不会在 8 秒内自发产生剪辑点——这是 Veo 3 数据构成的一个重要间接证据。另一确定项：训练数据必然以「带原生音轨」为主，因为音视频 latent 需成对进入联合扩散过程。

### [Vidu S1](../models/Vidu_S1.md) ⚠️

全部为单镜头（single-shot）片段，明确排除多镜头叙事：原始视频沿镜头边界切分为单镜头 clip，长镜头再细分，最终均为 3~60 秒单镜头单人片段。全部训练数据均含原生音轨（预过滤阶段即以「音视频完整性 audio-visual integrity」剔除缺失音轨或音视频不完整的样本）。未给出平均 clip 时长与镜头数分布的数值 [不确定]。

### [主流视频预训练数据集合并调研：Panda-70M、InternVid、Koala…](../models/pretraining_datasets.md)

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

## 语言/口音分布（多语种唇同步能力的数据基础）

`language_accent_distribution` · 详细程度: brief

### [Allegro](../models/Allegro.md) ⚠️

不适用于唇同步（无音频）。文本侧使用 T5（非 mT5）作为文本编码器、最大 512 token，论文通过定性对比（Fig.7–8）说明 T5 优于 mT5，因此训练 caption 实质为英文单语；未披露多语种 caption 或语言分布统计。[不确定]（caption 语言构成的定量分布）

### [Apollo](../models/Apollo.md) ⚠️

论文完全未讨论语言与口音分布：未列举支持语种，未给出各语种占比，未提及口音标注或多语种唇同步的数据基础。可作出的间接推断：转写环节同时使用 Whisper-Large-v3（多语种 ASR）、SenseVoice（阿里开源的中英日韩粤多语种语音理解模型，对中文与中国方言支持突出）与 Qwen2.5-Omni（中英双强），三者并用的组合强烈暗示语料至少覆盖中文与英文双语，且 SenseVoice 的引入指向中文/方言语料占相当比重；caption 文本编码器为 Qwen2.5-7B（中英双语模型）也支持这一推断。但这些均为工具链推断而非论文陈述。评测指标含 WER（词错误率），说明存在转写准确性评估，但评测语种未说明。[不确定]

### [CineDance / CineDance-1M](../models/CineDance.md) ⚠️

【体系层面】八维 taxonomy 中包含 Region（地域）维度，间接覆盖语种与文化区域的多样性诉求。
【技术链路】ASR 采用 Qwen3-Omni-30B-A3B，该模型本身具备多语种能力；CineBench 评测侧的 WER/CER 使用 Whisper-large-v3 计算，同样为多语种模型，说明数据与评测均非单语设定。
【说话人属性】提供角色音色描述（character voice description）与 ASR-to-Character 绑定（语音句子归属到具体角色 anchor token），但标注 schema 中未见显式的语言标签、口音标签、情绪标签字段。
【定量分布】论文完全未给出语种占比、口音分布或中英文比例等任何统计数字。[不确定]

### [CogVideoX](../models/CogVideoX.md) ⚠️

不适用且未建模。CogVideoX 不生成语音，数据 pipeline 无 ASR、无说话人属性、无语言/口音标注，不具备唇同步能力（相反，「Lecture Type」标签把口播类视频整类剔除，使模型在人物说话场景上的数据基础被主动削弱）。
文本条件侧：caption 与 T5 文本编码器主要面向英文（论文与 caption 生成 prompt 均为英文，文本长度上限 226 token）；官方仓库建议使用英文 prompt，中文 prompt 需先经改写。CogSound 侧是否涉及多语种语音未披露 [不确定]。

### [Cosmos-Predict2.5](../models/Cosmos-Predict25.md) ⚠️

不适用于语音维度（无音频、无语音、无口音概念）。文本维度上，caption 由 Qwen2.5-VL-7B 生成，论文未说明 caption 语言，但从文中所有示例 prompt（如机器人焊接场景描述、驾驶场景 prompt 模板、Transfer2.5 的厨房场景 prompt 模板）均为英文、且文本编码器 Cosmos-Reason1 为英文为主的 Physical AI VLM 判断，训练 prompt 基本为纯英文单语，未见多语种 caption 或翻译增强环节。[不确定：是否存在未披露的非英文 caption]

### [Data-Juicer 2.0](../models/Data-Juicer.md) ⚠️

[不确定] Data-Juicer 未提供针对视频/音频的语言与口音分布分析或控制能力，官方 T2V 案例也未涉及语言维度。
【间接相关的能力】
  · 文本侧：DJ 1.0 起即内置 language_id_score_filter（基于 fastText 的语种识别与置信度过滤），可对 caption 文本做语种筛选，但这是文本语种而非语音语种。
  · 语音侧：video_audio_ASR_mapper 可对音轨做语音识别，其底层 ASR 模型的多语种能力决定了可转写的语言范围，但 DJ 未把「识别出的语言」封装为可过滤的结构化字段，也无口音标注算子。
  · 说话人属性：现有算子覆盖年龄、性别、情绪，唯独不覆盖语言与口音。
【评价】对多语种唇同步这一 AV 生成的关键能力，DJ 目前不提供数据侧支撑，需自行扩展。这与其算子演进路径有关——2026年的新增算子集中在具身智能（相机标定、位姿、手部重建）与人物中心视频理解两个方向，语音语言维度尚未进入优先级。

### [Foley-Omni](../models/Foley-Omni.md) ⚠️

披露缺失，但可从数据集来源强推断为以英语为绝对主体。TTS 侧 LJSpeech（美式英语单说话人）、LibriTTS（英语有声书多说话人）；VisualTTS 侧 GRID（英语受控语料，34名英式说话人）、LRS2（BBC 英式英语）、Chem（英语讲座）均为纯英语数据集；SpeakerVid、TalkVid 为较新的大规模说话人视频集，可能含少量多语种但比例不明。
评测侧使用 WER 作为语音可懂度指标（V2ST-Bench WER 7.59，GRID 子集 WER 15.3），WER 计算通常依赖英语 ASR 模型，亦佐证评测为英语场景。
[不确定] 论文未给出任何语言分布表、未提及口音标注、未提及多语种唇同步能力、未说明 internal 数据的语言构成。这是该工作相对于工业级 AV 模型（如 Veo 3、Sora 2、Kling Omni）在多语种唇同步维度的明显短板。

### [Goku](../models/Goku.md) ⚠️

不适用/未涉及。Goku 不生成语音与唇形，无多语种唇同步能力，数据流水线中无 ASR、无语言识别、无口音标注。唯一与语言相关的是 caption 文本语言：论文使用 InternVL2.0、Tarsier2、Qwen2 生成与合并英文 caption，评测基准（GenEval、DPG-Bench、VBench）亦为英文提示词体系，可推断训练 caption 以英文为主。[不确定]（caption 是否含中文或多语种、以及各语种占比论文未说明）

### [HunyuanVideo-Foley](../models/HunyuanVideo-Foley.md) ⚠️

论文完全未讨论语言与口音分布，且这对本工作而言基本不构成相关维度：
【任务定位决定】HunyuanVideo-Foley 的目标是生成 Foley 音效（拟音），不是生成对白语音。模型不做 TTS、不做唇同步、不接受台词文本输入，因此不存在多语种唇同步的数据需求。
【语音的角色是被识别而非被生成】流水线中的 speech detection 环节，其作用是给含语音的片段打标签以便做类别配比控制，而非为了训练语音生成能力。理想情况下模型应当学会在有人说话的画面中生成环境音与动作音、而不去合成对白。
【文本条件的语言】文本 prompt 经 CLAP 文本编码器处理，CLAP 的训练语料以英文为主，因此模型的文本条件理解能力应以英文为主。论文未说明是否支持中文 prompt，也未做多语种 prompt 的评测。开源社区实践中通常建议使用英文描述。[不确定]
【结论】语言/口音分布对本工作属不适用维度，信息缺失不构成实质性披露缺陷。

### [HunyuanVideo](../models/HunyuanVideo.md) ⚠️

不适用于唇同步（无音频生成）。文本侧：原版使用 MLLM 作为文本编码器并在 caption 生成与 prompt 改写中考虑多语言——用 Hunyuan-Large 大语言模型对用户 prompt 做改写（prompt rewrite），功能包括「prompt结构标准化、复杂术语简化、多语言适配（multilingual adaptation）」以对齐训练 caption 的分布；实际支持中英文 prompt。训练 caption 的语种构成比例未公布。1.5 未描述 prompt 改写与语种分布。[不确定]

### [InstructAV2AV](../models/InstructAV2AV.md) ⚠️

[不确定] 论文完全未披露语言与口音分布，这是该工作在数据描述上的明显空白，尤其考虑到语音编辑是其四类核心任务之一。
【可推断的强证据指向英语为主】
  · ASR 工具为 ElevenLabs Scribe，语音合成工具为 ElevenLabs——虽然二者均支持多语种，但论文未提及任何多语种配置。
  · 三个影视类来源 MovieBench、Condensed Movies、Short-Films-20K 均以英语影视内容为主体；VGGSound 为 YouTube 素材，语音成分本就稀少。
  · 评测使用 Sync-C / Sync-D（基于 SyncNet，vshift=15 帧）衡量唇同步，SyncNet 本身在英语数据上训练，对非英语音素的口型判别精度较低。
  · 数据集 CSV 中的指令用 <S> 与 <E> 标记包裹语音内容边界，指令本身为英文。
【缺失项】未见任何语言标签、语种分布表、口音标注、多语种唇同步能力的评测或讨论。相对 Veo 3、Sora 2、Kling Omni 等工业模型强调多语种唇同步，本工作在这一维度未做任何声明，实际能力应视为仅在英语上验证过。

### [2026 其他音视频联合生成](../models/JAVG_2026_misc.md) ⚠️

本批工作在语言维度上普遍薄弱，是共同短板：
【ALIVE】训练数据的语言分布未披露[不确定]；仅在评测环节提到评测 prompt 覆盖「multiple languages (Chinese and English)」（中英双语）——考虑到字节跳动的产品定位与 700k hours 转写语音语料，中英为主可推断但无正面证据[不确定]。caption 中 <W> 标签内为 verbatim speech content（逐字语音内容），说明转写保留原语言。
【NAVA】仓库明确「English speech generation supported」，并注明「limited other languages' support」（其他语言支持有限）——即以英语为主、多语能力弱。这与其数据来源含大量 TED-style speech videos（TED 演讲绝大多数为英语）一致。15M 语料的语种分布未公开[不确定]。架构上引入 ReDimNet 说话人嵌入器做音色控制，音色与语言解耦，但语言覆盖仍受数据限制。
【StreamChar】用 Emilia 做编排器语音预训练（80k steps）——Emilia 本身是大规模多语种语音数据集（含英/中/德/法/日/韩六语），这在原理上为多语种能力提供了基础，但论文未报告多语种评测，实际覆盖不确定[不确定]。
【OmniCustom】用 GLM-ASR 为每段 5 秒训练音频生成转写（「we used GLM-ASR to generate transcriptions for each 5s training audio clip」）——GLM-ASR 为中文优势模型，暗示语料含相当比例中文[推断，不确定]。caption 中显式标注 accent（口音）属性（沿用 Ovi 的属性集）。评测集控制性别 1:1，但未控制语言比例。
【CCL / Baton】未提及语言分布[不确定]。CCL 的 in-house 数据（访谈、短剧、电影）与 OpenHumanVid 混合，语种构成不明；CCL 报告 WER 指标说明其评估了语音可懂度，但语言未明。
【ITS-JAVG】无训练数据；评测基准 VGGSound 与 JavisBench-mini 以英语环境为主。
【总体判断】七项中无一给出定量的语种/口音分布表，无一报告多语种唇同步的分语种评测。OmniCustom 是唯一把 accent 作为显式标注字段的工作，NAVA 是唯一诚实声明「多语支持有限」的工作。

### [音视频联合生成基线合集](../models/JavisDiT_baselines.md) ⚠️

披露极少，且整体呈现「英语单一化」倾向：
【MM-Diffusion / AV-DiT】训练数据不含人类语音（自然环境音与器乐音乐），因此语言/口音维度不适用（N/A）。
【JavisDiT / JavisDiT++】音频预训练数据虽包含语音类数据集（AudioSet、WavCaps 中含语音成分），但语种分布未披露[不确定]；音视频 SFT 阶段用 FunASR 主动剔除含语音视频，因此模型基本不具备语言相关的唇同步能力，该维度对其实际不构成瓶颈。
【UniAVGen】明确只用 Emilia「多语种音频数据集的英文子集（English subset）」做音频预训练——即刻意放弃多语种，聚焦英语；评测用 GRID 基准（英语语料）与 Whisper WER，同样只验证英语。内部真人音视频数据的语种构成未披露[不确定]。
【Harmony】使用 Emilia（本身为多语种 TTS 语料）但未说明是否限定英文子集[不确定]；Harmony-Bench 的语音子集论文提到用于考察「唇同步保真度与多语种语音鲁棒性（multilingual speech robustness）」，是本合集中唯一明确宣称覆盖多语种的工作，但具体语种列表、各语种占比、各语种唇同步指标均未给出[不确定]。
【共性缺口】五者均未标注口音（accent）属性、未做语种平衡、未报告分语种的同步指标——与工业模型（如 Vidu、Kling 等强调多语种唇同步）形成鲜明对比。

### [可灵 3.0 / 可灵视频 3.0 Omni](../models/Kling_30_Omni.md) ⚠️

[不确定] 未公布语种/口音比例。能力口径明确：中文、英文、日文、韩文、西班牙文五语种，且支持多种地方口音/方言与口音控制（regional accent control），口型随对应语言同步。KlingAvatar 2.0 在多语言多角色对话场景上做了数据扩充，评测集按中文语音100例、英文语音100例、演唱100例构建，暗示中英为主力语种、其余三语种数据量较小。[不确定：各语种小时数、方言覆盖清单、口音标签的标注方式]

### [LTX-2](../models/LTX-2.md) ⚠️

未给出语种列表与占比，但语言/口音在数据标注与架构两端都是一等公民。
【标注侧】caption 系统对对白做「精确转写 + 说话人（speaker）、语言（language）、口音（accent）识别标注」——这是本条目最具参考价值的打标设计之一，直接构成多语种唇同步与口音可控的数据基础。
【架构侧】采用 Gemma3-12B 多语言 LLM 作为文本编码器，并做多层特征抽取（早期层含原始音素信息、后期层含复杂语义），团队明确指出「深层文本理解不仅服务于全球语言支持，更直接决定生成语音的音素准确性」。推理时用户可在 prompt 中用引号包裹台词并指明期望的语言与口音。
【局限】论文承认：训练数据中代表性不足的语言或方言，其语音合成准确度与音画对齐会明显变差。具体覆盖多少语种、各语种/口音样本占比，完全未披露。[不确定]

### [LongCat-Video](../models/LongCat-Video.md) ⚠️

[不确定]。基础版无语音模态，但文本侧明确支持中英双语：文本编码器采用 umT5（多语种编码器，报告明确说明同时支持英文与中文 caption），且 caption 增强环节包含中英互译（translating captions between Chinese and English），因此训练 prompt 分布覆盖中英两种语言，具体比例未披露。
Avatar 系列涉及真实语音驱动，但技术报告未披露任何语种、口音的分布统计，也未说明多语种唇同步的数据构成；未指定 ASR 转写模型。

### [MOVA](../models/MOVA.md) ⚠️

论文将 MOVA 定位为“multilingual speech with high-quality lip synchronization”，但对语种/口音分布只给出定性证据，无任何比例数字：
【可确认的语种覆盖】中文与英文是明确演示并评测的两种语言。Figure 1 展示了英文多说话人与中文多说话人两组精确唇同步案例（中文示例含成人-儿童双说话人对话），并展示了中文屏幕文字生成。主观 Arena 评测集刻意构造为双语混合——将原本纯英文的 Verse-Bench 语音数据人工翻译了一半，形成中英双语混合的 732 条评测集（600 条来自 Verse-Bench + 132 条来自自建基准）。
【数据侧的语种基础】中文能力主要来自 in-house 的中文剧集（Chinese drama）语料；英文能力主要来自 SpeakerVid-5M、OpenHumanVid 及 YouTube 内容。
【口音标注】ASR 转写 prompt 明确要求“LAW OF LANGUAGE FIDELITY：Preserve the original language. No translation.”（保留原始语言、禁止翻译），因此 caption 中天然保留了多语种原文；且音频 caption 中会描述说话人的口音（论文给出的完整 caption 示例中就包含 “with a General American accent”，即通用美式口音），说明口音是被 Qwen3-Omni-Captioner 自然语言化描述的属性之一，但并非结构化枚举字段。
【空白】支持语种清单、各语种小时数占比、口音类别分布均未披露；论文亦在 Discussion 中提到不同语言的 phoneme-viseme 映射差异是难点，但未量化。[不确定]

### [合并条目：① Mochi 1](../models/Mochi_MAGI_Motif.md) ⚠️

① Mochi 1：文本编码器为单个 T5-XXL（英文为主），caption 语言构成未披露。[不确定]
② MAGI-1：caption 由 MLLM 生成，示例（Table 4）均为英文；语言分布未披露。有一处间接线索：为缓解训练 caption 与真实用户输入的分布错配，MAGI-1 在推理侧设计了 Prompt Enhancement（PE）策略，并把大 MLLM 的改写能力蒸馏到约 7B 的小模型上，构建了约 200 万条训练语料——但未说明该语料的语言构成。[不确定]
③ Motif-Video 2B：caption 由 Qwen3-VL-30B-A3B 生成，schema 与示例均为英文，caption_long 定义为 150–250 words、caption_short 为 15–25 words，明确以词（word）计量，实为英文单语；文本编码器为 T5Gemma2（早期阶段为 sentence-level embedding 模型）。未披露多语种 caption 或语言分布统计。
唇同步/口音维度对三者均不适用（无音频）。[不确定]

### [Movie Gen](../models/Movie_Gen.md)

不适用且未做建模：Movie Gen Audio 有意不生成任何人声/对白（diegetic speech 与 non-diegetic narration 均被排除在生成目标之外），因此没有语言、口音、唇同步相关的数据维度，也不存在多语种唇同步能力。数据侧对语音的处理仅止于用 AED 判定样本中是否存在 speech/singing，作为caption中的二值控制标签与分桶依据。文本条件方面，论文明确说明本研究「仅限英语文本输入」。

### [NVIDIA NeMo Curator](../models/NeMo_Curator.md) ⚠️

视频侧完全不涉及（不处理音轨即无语言维度）。
音频侧有间接相关能力但无分布统计：音频 pipeline 使用 NVIDIA NeMo ASR 模型做转写，示例数据集为 FLEURS（多语种语音基准，覆盖 100+ 语种），并有 long-form audio cutting 教程涉及说话人分离（speaker diarization）。26.02 起支持 streaming Sortformer 做说话人分离、VAD 与说话人切分。但框架未提供语种识别（LID）stage、口音标注能力，也未给出任何语言/口音配比的控制机制或统计。多语种唇同步所需的数据基础无法由该框架直接产出。[不确定]

### [OmniHuman 数据集 + OHBench](../models/OmniHuman.md) ⚠️

【语种】主要为英语，占比高达 80%（primarily English, accounting for up to 80%）。这是论文给出的唯一语种数值。剩余 20% 的语种构成完全未说明——是集中于某几种主流语言还是长尾分散，无从判断。[不确定]
【口音】未做任何口音标注与统计。考虑到数据源为全球性的 YouTube 且英语内容占 80%，实际必然包含美式、英式、印度英语等多种口音，但既无标注也无分布统计。[不确定]
【语种标注的存在性】HuggingFace 数据集的 sample_json 中，speech 字段包含「情绪、语言、时序信息」（emotion / language / timing），即逐条样本是带语种标签的——因此完整的语种分布对使用者可实测，只是论文未汇总。这一点与时长/分辨率分布的情况相同：标注齐备但统计缺席。
【caption 语种】caption 提供英文与中文两个版本（English/Chinese variants），即标注文本本身是双语的，但这与音频内容的语种是两回事（80% 英语音频 + 中英双语 caption）。双语 caption 对中文社区使用者是一个实际便利。
【ASR 模型的语种能力】FunASR-Nano（达摩院 FunASR 系列）在中文上表现最强、同时支持多语种；评测侧的 WER 计算使用 SenseVoice（同为多语种 ASR），二者均不构成语种限制。
【对唇同步能力的含义】80% 英语的偏斜意味着基于本数据训练的模型，其 phoneme-viseme 映射会以英语音素体系为主，对声调语言（中文）与其他音素体系的唇形准确度可能偏弱，论文未讨论这一潜在偏斜。[不确定]

### [Open-Sora 系列](../models/Open-Sora.md)

不适用于语音层面（无音频生成）。文本侧：Open-Sora Plan 明确只用 Anytext-3M 的**英文子集**（约占该数据集 50%，1.8M），caption 与 prompt 全部为英文；Open-Sora 系列 caption 亦全部为英文（PLLaVA、LLaVA-Video、Qwen2.5-Max 均输出英文描述）。两个项目均未做多语种 prompt 支持的数据建设，中文 prompt 需依赖外部翻译。口音/唇同步相关数据基础：无。

### [Ovi](../models/Ovi.md) ⚠️

未给出定量分布。可确认的定性事实：
(1) 音频预训练数据「emphasize linguistic diversity, prosody, and timbral variation」，即语料在语言/口音层面有意做了多样性覆盖，但覆盖了哪些语种、各占多少未公开[不确定]。
(2) 打标阶段对含语音片段显式标注 accent（口音）属性，与 age、gender、pitch、prosody、emotion、speaking rate 并列写入音频描述，意味着口音是可控条件而非隐含变量。
(3) 评测层面仅报告英文 WER：在 Seed-TTS test-en 上 WER=0.035，未报告任何非英语指标，说明其可验证的语音能力以英语为主；社区实践与 demo 亦以英语对白为主[不确定]。
(4) 唇同步能力完全由数据驱动（无人脸 bbox、无 mask），因此多语种唇同步能力直接受限于内部语料的语种覆盖。

### [Script-a-Video](../models/Script-a-Video.md) ⚠️

论文未给出任何语言/口音分布数据：未列举支持语种、未给出各语种占比、未做口音标注、未讨论多语种唇同步。
可间接推断的线索：
1) 评测环节计算 WER 时明确使用「jieba-based tokenization for CJK text」（对中日韩文本采用 jieba 分词），说明生成与评测中确实存在中文（或更广义的 CJK）语音内容，训练数据非纯英文；
2) 数据来源以 film / television / TV drama 为主且为腾讯内部数据，中文影视内容占主导的可能性较高，但论文无任何佐证；
3) MTSS 的 Event 流为 dialogue 事件设有 "line" 字段记录逐字台词（verbatim text），schema 层面对语种无限制，但未设语种或口音字段；
4) Event 流的 "description" 字段用于捕捉细腻语义如情绪起伏（emotional shifts）与发声技巧（vocal techniques），这是最接近「说话人属性」的设计，但仍是自由文本而非结构化的口音/语种标签。
语种与口音属完全信息空白。[不确定]

### [Seedance 2.0 与 Seedance 1.5 pro](../models/Seedance_20_Seedance_15_pro.md) ⚠️

训练数据的语种/口音比例未披露，但能力与评测覆盖可反推：Seedance 1.5 pro 原生支持多语种与多地区方言的唇形同步，可捕捉各自独特的语音韵律与情绪张力；Seedance 2.0 在中文方言、传统戏曲、歌唱场景上的指令跟随准确率显著提升。SeedVideoBench 2.0 音频评测显式区分中文方言/口音、中文多人对话、中文综艺人声、中文戏曲、英语、少数民族语言等语种类目（2.0 在音频质量上英语 4.17、少数民族语言 3.82、中文方言 2.82；音画同步上英语 4.17、少数民族语言 3.88、中文方言 3.64），说明训练语料以中英为主并覆盖多种中文方言与少数民族语言。此外 Seedance 1.0 的 caption 模型即在中英双语数据上训练。[不确定：各语种具体占比]

### [SkyReels 系列](../models/SkyReels.md) ⚠️

【SkyReels-V2】纯视频模型，无语言维度；打标与提示词以中英为主。
【SkyReels-V4】语言是数据构造的显式对象，但无占比数字：(1) 合成数据专门用于「多语种画面文字生成」，覆盖中文、英文、日文、韩文、德文、法文等；(2) 合成数据亦用于「多语种语音合成（multilingual speech synthesis）」，即用 TTS 补齐语种覆盖缺口——这是解决长尾语种唇同步数据不足的直接手段；(3) 音频主干在数十万小时以语音为主的数据上预训练，语种覆盖通过 TTS 扩展；(4) 评测基准 SkyReels-VABench 的提示词覆盖多语种，「尤其强调中文与英文」。未给出各语种样本占比、口音（accent）标注字段，也未见方言/口音维度的分布控制——口音在 caption schema 中并非独立字段（与 LTX-2 显式标注 accent 不同）。[不确定]

### [Sora 2](../models/Sora_2.md) ⚠️

完全未披露。模型具备对白生成与唇形同步能力，实际使用中可生成多语种语音，但 OpenAI 未公布支持语种列表、各语种数据占比、口音分布，也未说明多语种唇同步的数据基础。安全侧仅提及会对音频转写文本（audio transcripts）过安全分类器，间接说明存在ASR能力，但未涉及语种覆盖。[不确定]

### [Step-Video-T2V](../models/Step-Video-T2V.md) ⚠️

不适用于唇同步（无音频生成）。文本侧是本条目的一个特色：模型采用两个双语（中英）文本编码器（Hunyuan-CLIP 与自研 Step-LLM 的双编码器组合），原生支持中英双语 prompt 输入，官方将「原生中英双语输入」列为核心能力之一；配套的评测基准 Step-Video-T2V-Eval 全部 128 条 prompt 为中文，也印证其数据与评测的中文侧重。但训练 caption 的中英文语种构成比例、以及是否对中文 caption 做专门增强，报告均未公布。[不确定]

### [UniTalking](../models/UniTalking.md) ⚠️

论文未给出任何训练数据的语种或口音分布统计：无语种清单、无各语种占比、无口音标注、无多语种 phoneme-viseme 映射的讨论。可确认的间接证据表明至少覆盖中英双语：
【评测侧的双语证据】TR2AV 任务的音色相似度评测使用 MiniMax Multilingual Test Set，结果分英文（0.703）与中文（0.662）两列分别报告——模型能在两个语种上产出可评测的克隆语音，说明训练数据中中英文均有相当占比。值得注意的是中文得分低于英文（0.662 vs 0.703），而三个对照方法（ElevenLabs 0.613/0.677、MiniMax 0.756/0.780、Qwen3-Omni 0.773/0.772）中有两个是中文优于英文——UniTalking 是唯一呈现明显「英强中弱」倾向的方法，间接暗示其训练数据可能以英语内容为主。这一推断未经论文确认。[不确定]
【TTS 评测的单语种性】阶段一音频模型的 WER 评测仅在 Seed-TTS test-en（英文）上进行，未报告 test-zh 结果，进一步指向英文为主要优化目标。
【文本编码器】使用 UMT5（多语种 T5），架构层面不构成语种限制。
【转写模型】Whisper-V3 为多语种 ASR，同样不构成限制。
【内部 TTS 数据】阶段一使用的内部 TTS 数据语种构成完全未披露，而音频分支的发音能力主要在此阶段建立，这是语种能力的真正决定性因素却完全是黑箱。[不确定]

### [UniVerse-1](../models/UniVerse-1.md) ⚠️

论文完全未讨论语言与口音分布：未给出支持语种清单、各语种小时数或占比，未做口音标注，未讨论多语种 phoneme-viseme 映射问题。可间接推断的线索：
- 训练侧使用 Whisper 做 ASR，Whisper 本身多语种，但论文未说明是否限定语种或是否统计语种分布；
- 数据源为 YouTube 的演讲/访谈/vlog，英文内容占比大概率最高；
- 评测侧 Verse-Bench 的 Set2-V 明确从 YouTube 与 Bilibili 双平台采集，Bilibili 的引入说明评测覆盖中文内容；Set3-Ted 取自 2025 年 9 月的 TED Talks，为英文演讲。因此中英双语至少在评测层面均有覆盖；
- 架构上移除了 Ace-step 的 speaker encoder，目的是不绑定特定说话人，但这不涉及语种维度。
语种与口音分布属彻底的信息空白。[不确定]

### [Unison](../models/Unison.md) ⚠️

论文完全未讨论语言与口音：未给出支持语种清单、各语种占比，未做口音/方言标注，未讨论多语种 phoneme-viseme 映射，未说明是否限定语种。
【可间接推断的线索】
- HDTF 源自 YouTube 英文演讲/新闻，为英语单语；VFHQ 源自访谈场景，以英语为主；CelebV-Text 为野外人脸视频，英语占主导；OpenHumanVid 含中英文影视素材，语种分布不明；因此音视频侧语料大概率以英语为绝对主体；
- 歌唱语料取自 YuE collection，YuE 系列以中英双语歌唱生成为特色，可能引入中文成分；
- 「内部语音数据」由中国机构（字节跳动、中国电信 TeleAI）提供，含中文语音的可能性较高，但论文零说明；
- 评测侧用 Whisper-large-v3 计算 WER，Whisper 本身多语种，但论文未说明评测集的语种构成，也未按语种分列 WER。
【与同类工作的对比】MOVA 构建了 732 条中英双语 Arena 评测集并给出 cpCER，UniVerse-1 的 Verse-Bench 覆盖 YouTube 与 Bilibili 双平台；Unison 在语种维度上没有任何对应设施，是其评测体系相对单薄的一处。
语种与口音分布属彻底的信息空白。[不确定]

### [Veo 3 / Veo 3.1](../models/Veo_3_Veo_31.md) ⚠️

[不确定] 语言与口音分布完全未披露。产品侧表现为可生成多语种对白并自动匹配口音与唇形（英语、西班牙语、普通话等），但官方文档未列出支持语种清单、未给出各语种数据占比、未说明口音标注体系。第三方评测普遍反映非英语对白的唇同步与发音质量明显弱于英语，间接说明训练数据以英语为主导。官方亦承认模型「不提供口音、音色等细粒度声音控制」，说明说话人属性未作为条件维度进入训练数据 schema。

### [Vidu S1](../models/Vidu_S1.md) ⚠️

[不确定]。论文未披露训练数据的语种与口音分布，也未说明多语种唇同步能力的数据基础。仅在 caption 结构中包含 dialogue（对白）字段，并在 Vidu-StreamBench 基准中提到覆盖多样的「说话人属性（speaker attributes）」与情绪，但未展开语种维度。产品面向中文市场，推测以中文为主但无一手依据。

### [主流视频预训练数据集合并调研：Panda-70M、InternVid、Koala…](../models/pretraining_datasets.md)

语音层面全部不适用（无音频建模）。文本层面：
- 七者的 caption **全部为英文单语**，无一家提供中文或多语caption，也无一家做多语prompt的数据建设。
- **InternVid 是唯一涉及多语数据的**：其内部采集包含 **11种语言的 ASR 字幕**（论文图16示例了英语、中文、韩语、德语），源视频国别覆盖英美澳日韩中俄法等。但**这些 ASR 转写既未用于生成caption，也未随数据集发布**——发布的 jsonlines 中无音频、无ASR、无字幕字段。这是一处被采集却被丢弃的多语资源。
- **Panda-70M 使用了字幕但仅作输入**：英文字幕（含 YouTube 自动字幕）被送入教师模型作为文本侧上下文（video2dataset 配置 subtitleslangs:['en'], writeautomaticsub:True），字幕本身不作为 Panda-70M 的字段发布，仅落在每clip的JSON元数据中。
- 口音、说话人身份、唇同步相关的数据基础：**七者全部为零**。
- 一个连带影响：由于 caption 均为英文且普遍冗长（Koala 202词、MiraData 318词、UltraVideo 824词），使用这些数据训练时**必须搭配长上下文文本编码器**——MiraData 因此明确弃用 CLIP 的77 token 编码器改用 **Flan-T5-XXL（512 token）**；LVD-2M 则在实验中吃了亏，其88.7词caption被冻结CLIP文本编码器的77token截断，作者将 I2V 文本匹配提升不明显直接归因于此。
