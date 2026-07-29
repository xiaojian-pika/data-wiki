# Movie Gen（含 Movie Gen Audio）

> 主题: 视频生成模型（含音视频同时生成）的数据处理：数据清洗流程、数据分布、打标方式

[← 返回首页](../index.md)

**目录**: [基本信息](#基本信息) · [数据规模与分布](#数据规模与分布) · [清洗流程](#清洗流程) · [打标方式](#打标方式) · [音视频对齐](#音视频对齐) · [训练配合](#训练配合) · [效果对比](#效果对比)

## 基本信息

### 名称

Movie Gen（含 Movie Gen Audio）

### 发布机构/公司

Meta（Meta AI / GenAI 团队，超过百人贡献者名单）

### 发布时间（技术报告/论文/开源时间）

2024年10月4日随官方博客与技术报告发布；arXiv 预印本 2410.13720（v1，2024年10月17日挂出），标题《Movie Gen: A Cast of Media Foundation Models》

### 类型（模型/数据集/工具链/评测基准）

模型（媒体生成基础模型家族：Movie Gen Video 30B 文生视频、Personalized Movie Gen Video、Movie Gen Edit 视频编辑、Movie Gen Audio 13B 音频生成）+ 评测基准（Movie Gen Video Bench、Movie Gen Audio Bench）

### 开源程度（权重/代码/数据/pipeline各自是否开源）

半开放：模型权重未开源、训练/推理代码未开源、训练数据未公开。但技术报告（92页）极其详细地公开了数据清洗 pipeline、各级过滤阈值与漏斗保留率、打标方案与训练配方，属于行业最详尽的公开数据工程文档之一。开源部分为评测资产：Movie Gen Video Bench（1003条prompt）与 Movie Gen Audio Bench，以及在这两个benchmark上非精选（non cherry-picked）的生成结果视频/音频，托管于 https://github.com/facebookresearch/MovieGenBench 。论文明确说明模型仅用于研究目的，部署前仍需多项改进。

### 是否支持音视频同时生成，以及实现方式（原生联合/级联/MoE融合）

不支持原生音视频联合生成，采用级联（cascade）方案：Movie Gen Video（30B，Flow Matching Transformer）先生成无声视频，再由 Movie Gen Audio（13B，Flow Matching DiT + DAC-VAE 潜空间）以视频和文本为条件生成同步音轨（V2A / TV2A），支持音频扩展（audio extension）实现最长数分钟的整片配乐。音频侧内部是「单一模型联合生成所有音频类别」（diegetic音效、non-diegetic音效、器乐音乐一起生成），而非按类别分多个模型，理由是不同音频类别之间也存在相关性。论文结论明确指出：视频与音频当前是分开训练的，「让模型联合生成这两个模态是重要的未来研究方向」。此外模型有意不生成人声/对白（认为可由TTS补齐、且无脚本时难以生成）。

### 调研信息来源列表（论文/技术报告/官方文档/新闻的URL，标注每条来源的性质：官方一手/同团队旁证/第三方报道）

- 官方一手: Movie Gen: A Cast of Media Foundation Models (arXiv:2410.13720, 92页全文含附录J.1阈值表) https://arxiv.org/html/2410.13720v1
- 官方一手: facebookresearch/MovieGenBench (开源评测基准) https://github.com/facebookresearch/MovieGenBench

## 数据规模与分布

### 训练数据量级（视频条数/小时数/token数，预训练与SFT分开） ⚠️

预训练（视频）：O(100)M 视频-文本对 + O(1)B 图文对；原始视频池时长4秒~2分钟（平均28秒），清洗后每条clip为4~16秒单镜头片段。
后训练（视频SFT）：人工精选的小规模高质量视频+人工caption集合，论文未给出具体条数[不确定]；训练仅用512张H100（64节点），相对预训练最多6144张H100的规模小两个数量级。
个性化（PT2V）：从预训练集中筛出 O(1)M 条单人视频；采样得 O(10)M paired 样本、O(10)K 真实 cross-paired、O(1)M 合成 cross-paired；SFT 集为 O(1000) 条高质量单人视频。
音频预训练：总计 O(100)M 样本 / O(1,000)K 小时（即百万小时量级），其中 Sound 类独占 O(100)M / O(1,000)K，Music、Sound+Music、Sound+Voice、Sound+Music+Voice 各为 O(10)M / O(100)K。
音频微调：影视级音视频 O(100)K 样本 / O(1)K 小时，高质量纯音频（音乐 O(10)K 小时 + 音效 O(10)K 小时）O(1,000)K 样本 / O(10)K 小时。

### 数据来源构成（自有/公开数据集/网络爬取/授权采购/合成数据） ⚠️

论文未披露数据的具体来源渠道与授权方式，仅以「a large pool of videos」「sourcing data from a large volume」描述，可判断为 Meta 自有的大规模内部视频/音频数据池 [来源构成不确定]。图文数据的清洗策略沿用 Meta 自家 Emu（Dai et al., 2023）的做法。可确认的构成特征：原始视频覆盖人类、自然、动物、物体等多个domain；音频预训练数据来自视频的原生音轨；音频微调额外使用了不带视频的高质量纯音乐与纯音效素材库（专业制作素材）。合成数据被显式使用于个性化（personalization图像生成模型造参考图）与视频编辑（仿射动画化的图像编辑对、生成式指令分割、backtranslation 反向配对）。未使用公开学术数据集作为训练主体（VGGSound 等仅用于评测）。

### 数据合规与溯源（授权数据占比、rights-cleared数据集、C2PA等） ⚠️

[不确定] 论文完全未讨论数据授权占比、版权清理（rights-cleared）、内容溯源标准（C2PA/水印）等问题。仅在结论的「Safety considerations」中提到：模型为研究目的开发；模型可能学到模态内的偏见（如视频训练数据中的视觉偏见）；真实部署时会接入安全模型拒绝违反政策的输入prompt与生成结果。数据侧的合规流程、人脸/隐私授权、生成内容水印均未披露。

### 片段时长分布与切分策略

原始视频4秒~2分钟、平均28秒；因训练需要4~16秒clip，先用 FFmpeg 做场景变化检测，每条视频采样1~2个时长超过16秒的场景，再从每个场景中随机抽取1个4~16秒的clip，明确避免不看镜头边界的随机采样（否则生成视频会频繁突兀转场）。超过50%的训练clip时长落在15~16秒。
分桶策略（表2，五个时长桶，桶内latent帧数一致便于batch）：10.67s@24fps→256帧/32 latent帧；16s@16fps→256帧/32 latent帧；12~16s@21~16fps→256帧/32 latent帧；8~12s@24~16fps→192帧/24 latent帧；4~8s@32~16fps→128帧/16 latent帧。前两桶取自10.67~12秒与≥16秒长视频的中段clip。帧率通过caption中的FPS token控制（16~32 FPS）。
SFT视频时长限定10.6~16秒，其中50%为16秒、50%为10.6~16秒；16秒的用16FPS训练，10.6~16秒的用24FPS训练。
音频侧：视频长度限制在4~120秒，预训练时序列上限30秒（750帧）超出则随机切块；微调时随机采样10秒和30秒片段；caption同时按10秒与30秒两种chunk制作，训练中按 5 batch : 1 batch 采样。

### 分辨率/宽高比分布与分桶策略

分辨率门槛随训练阶段收紧：低分辨率训练阶段要求最小宽高 ≥720px；高分辨率训练阶段要求最小宽高 ≥768px（表44中该步骤把数据从100%砍到25%）。训练分辨率从256px渐进到768px，另有独立的空间超分模型输出1080p HD。
宽高比：预训练集控制为 60% 横屏 + 40% 竖屏（偏好横屏，因其时长更长、美学更好、运动更稳定）；高分辨率集调整为 80% 横屏 + 20% 竖屏；清洗漏斗中「宽≥高」这一步把数据从25%砍到7%。
分桶：图像与视频各用五个宽高比桶，同桶内latent形状完全一致以便batching，因此模型可输出多种比例，如横屏1024×576、竖屏576×1024。
黑边问题单独处理：自研边框检测器（一阶导数找大梯度像素+扫描线算法定位边框）剔除带黑边视频，竖屏视频尤为常见。
音频侧对视频质量的要求较低：剔除分辨率<480px的视频即可。

### 类别/domain分布与配比策略（人物、动作、场景、风格等比例控制与概念均衡）

预训练原始池覆盖人类、自然、动物、物体等domain。概念均衡采用两步：①用视频-文本联合embedding模型提取语义embedding，聚类得到细粒度概念簇；②合并重复簇后，按簇大小的倒平方根（1/sqrt(cluster size)，沿用 Mahajan et al. 2018 的做法）从每个簇采样，从而压制头部主导概念、抬升长尾——该步在漏斗中把数据从1.15%降到0.94%。
人物是重点倾斜domain：最终高分辨率训练集中至少60%的视频包含人。为此专门建立了一套包含600个人类动词与表情（human verbs and expressions）的taxonomy，用该taxonomy做zero-shot 文本→视频检索来定向挑选含人视频，并在概念重采样阶段刻意保留（preserve）这些人物视频的频次，防止被均衡策略稀释。
SFT阶段的概念平衡更精细：先用同一套600动词taxonomy做 text k-NN 从候选池检索每个概念的视频，再人工为每个概念挑几条视觉上出彩的种子视频，用这些种子做 video k-NN，最终得到一个概念平衡且规模足够小、可全量人工审核的子集；k-NN 使用视频-文本联合embedding模型的视频与文本embedding。
评测端对应的概念分布为人类活动、动物、自然与风景、物理现象、非常规主体与非常规动作五类。

### 音频类别分布与配比（语音/音效foley/音乐/环境音/静音的比例及控制策略）——AV模型独有维度

Movie Gen Audio 建立了「音频类型 × diegetic属性」的二维分类体系（表23），这是其数据工程的核心创新。
第一轴音频类型：voice（speech + singing）、non-vocal music（纯器乐音乐）、general sound（一般音效），由 AED（audio event detection）模型基于 AudioSet 527类本体自动打标，一个样本可同时命中多类（AudioSet中speech/singing子类→voice；music子类→music；其余→sound）。
第二轴 diegetic（画内、与画面有因果关系，如现场说话、现场乐队、海浪声、画外鸟叫）/ non-diegetic（画外，如纪录片旁白、背景配乐、罐头笑声、riser）。判定用 CAVTP（对比式 audio-video-text 预训练模型）的音频与视频embedding余弦相似度——因该模型主要在diegetic数据上训练，画内音的音视频embedding更接近。
分桶阈值（表24，由人工检视确定）：AED为 sound / voice / voice+sound 时，CAVTP>0.2 判 diegetic；AED为 music / music+sound / voice+music+sound 时，CAVTP>0.3 判 diegetic；AED为 music 且 CAVTP<0.1 判 non-diegetic；AED为 sound+music 且 0.1<CAVTP<0.25 判 non-diegetic；AED为 sound+voice+music 且 0.1<CAVTP<0.25 判 mixed。
配比与取舍：首先丢弃 silence 为主导类的所有视频；预训练只使用 diegetic 或 diegetic/non-diegetic 混合的数据，另加入一小部分 non-diegetic 背景音乐；明确优先 general sound（因低层物理规律最难学、错误最易被察觉），实际分布上 Sound 类占 O(100)M样本/O(1,000)K小时的绝对主导，其余四类各仅 O(10)M/O(100)K，即音效对音乐/含人声类约为10:1的量级差。
有意排除：不生成 diegetic speech（无脚本时难、且生成视频有伪影）和 non-diegetic speech（可用TTS替代）；微调的 cinematic split 中带人声的片段被整体排除。
微调阶段配比：影视级音视频（Cin-AV）与高质量纯音频（HQ-A）按 10 batch : 1 batch 混合。

### 叙事结构分布（单镜头vs多镜头、平均clip时长、镜头数分布、是否含原生音轨）

视频训练数据全部为单镜头（single-shot）片段：通过 FFmpeg 场景边界检测 + PySceneDetect 抖动检测保证clip内无转场、无跳剪，且要求存在非平凡运动（non-trivial motion）；每条原视频只取1~2个场景、每场景1个clip，平均clip长度落在15~16秒区间（>50%）。因此模型是单镜头生成器，不建模多镜头叙事。
是否含原生音轨：Movie Gen Video 的训练完全不使用音轨（纯视觉+文本）；Movie Gen Audio 的预训练则依赖视频的原生音轨作为监督信号。两者数据流是分离的。
多镜头仅出现在评测侧：Movie Gen Audio Bench 把评测视频分为 single-shot（数量多、音效谱系广，测鲁棒性与泛化）与 multi-shot（取自短片，含场景转换、情绪与叙事更强，用于评测视频-音乐对齐，如音乐何时进入、如何随剧情演进、是否与剪辑点对齐、音效与音乐的混音是否和谐）。

### 语言/口音分布（多语种唇同步能力的数据基础）

不适用且未做建模：Movie Gen Audio 有意不生成任何人声/对白（diegetic speech 与 non-diegetic narration 均被排除在生成目标之外），因此没有语言、口音、唇同步相关的数据维度，也不存在多语种唇同步能力。数据侧对语音的处理仅止于用 AED 判定样本中是否存在 speech/singing，作为caption中的二值控制标签与分桶依据。文本条件方面，论文明确说明本研究「仅限英语文本输入」。

## 清洗流程

### 清洗漏斗整体结构（几级过滤、各级顺序）

【视频侧】三级过滤 + 一级打标的漏斗（论文图9）：
1) 视觉过滤（Visual filtering，6个filter）：最小宽高门槛 → 宽高比配比 → 视频OCR去多字幕/文字 → FFmpeg场景边界检测切4~16秒clip → 训练轻量视觉模型预测帧级美学/画质/大边框/视觉特效并过滤 → 参照 Panda-70M 移除与视频开头重合的clip的前几秒（开头常有不稳定运镜和转场特效）。
2) 运动过滤（Motion filtering）：内部静态视频检测模型去无运动 → FFmpeg VMAF motion score 与 motion vector 判定「合理运动」→ PySceneDetect 每秒镜头数识别并剔除抖动 → 剔除幻灯片等特效运动。
3) 内容过滤（Content filtering）：copy-detection embedding 感知去重 → 视频-文本联合embedding聚类 + 合并重复簇 + 1/sqrt(簇大小) 重采样做概念均衡。
4) 打标（Captioning）：LLaMa3-Video 8B/70B 生成平均100词的稠密caption + 16类相机运动分类器结果前缀拼接 + FPS token。
另有「多阶段curation」：用逐级更严的视觉/运动/内容阈值切出3个子集——720px低分辨率训练集、768px高分辨率训练集、以及新增数据扩充后的高分辨率集。
【视频SFT侧】四阶段串行：①自动严阈值筛选（美学、运动、场景切换严阈值 + 用 Detic 物体检测剔除主体过小的视频）得到几百万条但概念不均衡的候选；②概念均衡（600动词taxonomy做 text k-NN + 人工挑种子后做 video k-NN）缩到可人工审核的规模；③人工挑选影视感视频（需有角度光/自然光或棚拍光、色彩鲜艳但不过饱和、画面不杂乱、有非平凡运动、无相机抖动、无编辑特效与叠加文字），并由标注员亲手裁剪出最精彩的片段；④人工在 LLaMa3-Video caption 基础上精修与补全。
【音频侧】：AED 按 AudioSet 527类打标 → 丢弃 silence 主导样本 → 映射为 voice/music/sound 三类 → 用 CAVTP 余弦相似度分入 diegetic/non-diegetic/mixed 桶 → 视觉侧质量过滤（OCR去带文字视频、去静态视频、去<480px低清视频）→ 时长限制4~120秒 → copy-detection embedding 视觉去重 → 多模型合成结构化caption。微调数据额外用「影视感音视频分类器 + AED」自动初筛，再由人工标注做最终选择。

### 漏斗定量保留率（每级过滤的输入/输出量与最终保留率，如Apollo 27%） ⚠️

论文表44给出了业界罕见的逐级定量漏斗（针对最严格的高分辨率集curation阈值，数字为相对原始池的剩余体量百分比）：
时长 4s≤duration≤120s → 100%
分辨率 width≥768 且 height≥768 → 25%（单步砍掉75%）
宽高比 width≥height → 7%（单步砍掉72%，最陡的一刀）
无文字（所有采样帧的 词检测分×词识别分 均<0.6）→ 1.94%（单步砍掉72%）
无黑边 → 1.87%
无场景切换（从1个场景中采1个12~16秒clip）→ 1.78%
美学（clip中间帧 LAION 美学分 ≥4.0）→ 1.57%
非慢动作（motion score>2.0，motion vector均值>0.5 且 <7）→ 1.32%
非抖动（每秒镜头数<0.85）→ 1.22%
无内容重复（embedding余弦相似度<0.99）→ 1.15%
概念重采样（每簇保留量 1/sqrt(簇大小)）→ 0.94%
论文原文明确指出：在这套阈值下「数据接受率低于1%」（less than 1%），即约百分之一的原始视频进入高分辨率训练集。音频侧未给出对应的定量漏斗[不确定]。

### 镜头切分方法（PySceneDetect/自研模型/shot-aware splitting）

双工具组合：① FFmpeg 的场景变化检测（scene change detection）负责主切分——先检出场景边界，从每条原视频中采样1~2个时长超过16秒的场景，再从每个场景内随机抽取1个4~16秒片段作为训练clip；论文明确说明不做「不考虑场景边界的随机采样」，否则生成视频会出现频繁而突兀的场景切换。② PySceneDetect 的 Shot Boundary Detection 用作抖动检测器而非切分器——因为FFmpeg的motion score和motion vector难以识别频繁抖动的运镜，而抖动视频会被SBD拆成大量假阳性镜头，故用「每秒检出镜头数」作为抖动代理指标，超过0.85 shots/秒即剔除。③ 参照 Panda-70M 的做法，若clip起点与整段视频开头重合，则丢弃开头数秒（开头常含不稳定运镜或转场特效）。

### 质量过滤（美学评分、清晰度、OCR文字过滤、黑边/水印/logo检测）

视觉过滤共6个filter，逐项细节（附录J.1给出模型与阈值）：
· 分辨率：最小宽或高 <720px 剔除（高分辨率阶段提到768px）。
· 宽高比：过滤以达成 60%横屏/40%竖屏（高清集 80%/20%）的目标配比。
· OCR文字过滤：内部OCR模型自适应采样帧→检测词→识别词文本；只保留「所有采样帧上 词检测分 × 词识别分 均 <0.6」的视频，用于剔除字幕/花字/贴片文字过多的素材。这一步在漏斗中是最狠的几刀之一（7%→1.94%）。
· 黑边/边框检测：自研简易检测器，先用一阶导数找出垂直/水平方向像素差值大的位置，再用扫描线算法定位边框；动机是训练数据带边框会导致生成视频也带黑边，竖屏素材尤其常见。
· 美学评分：对每个clip的中间帧应用公开的 LAION aesthetics 图像模型，分数 <4.0 剔除，用于去掉模糊、压缩伪影严重的片段；论文特别做过对比——计算多帧平均美学分并未显著提升对低质片段的召回，故只用中间帧（省算力）。
· 视觉特效与画质：另外训练了若干轻量视觉模型，输出帧级的「视觉美学、视觉质量、大边框、视觉特效」预测信号用于过滤。
· 起始片段：移除与视频开头重合的clip的前几秒。
SFT阶段追加：用 Detic 物体检测模型剔除主体过小的视频；人工复核画面是否杂乱、色彩是否过饱和、有无叠加文字与编辑特效。
音频侧对视频的质量过滤更轻量：OCR去带文字视频、去静态视频、去分辨率<480px的视频。

### 运动过滤（光流/运动分数阈值、静态与抖动剔除）

四步串联：① 用内部的静态视频检测模型（internal static video detection model）直接剔除完全无运动的视频；② 用 FFmpeg 的 VMAF motion score 与 motion vector 判定「合理运动」，附录给出的定量阈值为 motion score > 2.0、motion vector 均值 > 0.5 且 < 7（下界剔除慢动作/近静止，上界剔除运动过剧烈的素材）；③ 用 PySceneDetect 的 Shot Boundary Detection 识别频繁抖动运镜——抖动视频会被误拆成大量镜头，故以「每秒检出镜头数 ≥ 0.85」为阈值剔除，动机是训练数据抖动会导致生成视频画面发抖；④ 剔除带特殊运动特效的视频，例如幻灯片式（slideshow）视频。整体做法沿用 Emu Video（Girdhar et al., 2024）的低运动自动过滤思路。SFT阶段对运动使用更严格的阈值，并由人工确认「有非平凡运动且无相机抖动」。

### 去重方法（精确去重与基于embedding的语义去重分别记录）

分两个层次，与字段要求的「精确/语义」二分基本对应：
· 感知级/近重复去重：使用 copy-detection embedding（SSCD，Pizzi et al., 2022）空间中的相似度剔除感知上重复的clip，附录表44给出的阈值为「embedding 余弦相似度 < 0.99 保留」，该步在漏斗中把1.22%降到1.15%（约剔除6%）。这一方法同样被用于 Movie Gen Audio 预训练数据的视觉去重。
· 语义级去重与均衡：用视频-文本联合embedding模型抽取语义embedding做聚类得到细粒度概念簇，先合并重复的簇（merge duplicate clusters），再按 1/sqrt(簇大小) 从每簇采样。该步既是语义去冗余也是概念均衡，把1.15%进一步降到0.94%。
· 评测侧亦关注重复：论文提到发现某些公开数据的train/eval split之间存在大量重复或近重复（如加了静态水印或文字的版本）。

### VLM/LLM作为质检员（多模态大模型质量打分与错配剔除，2026年从浅层打分器转向大模型语义判断的趋势）

Movie Gen 属于「2024年范式」的代表：数据质检几乎完全依赖一组专用的轻量判别模型/分类器与打分器，而非用通用VLM/LLM做整体语义质量判断与图文错配剔除；大模型（LLaMa3-Video）在这里承担的是打标（caption生成）角色而非judge角色。具体的模型清单：
视觉侧——内部OCR模型（词检测+词识别双分数）；自研黑边检测器（非学习式）；LAION aesthetics 图像美学预测模型；若干自训的帧级视觉模型（美学/画质/大边框/视觉特效）；内部静态视频检测模型；PySceneDetect 与 FFmpeg（场景/运动）；copy-detection embedding 模型（SSCD）；视频-文本联合embedding模型（聚类与k-NN检索）；16类相机运动分类器；Detic 物体检测模型（剔小主体）；ArcFace 人脸识别模型（PT2V身份一致性，帧间阈值0.5、合成参考图阈值0.7）；人脸检测与人脸区域分割模型。
音频侧——AED 音频事件检测模型（AudioSet 527类本体）；CAVTP 对比式音-视-文预训练模型（输出音视频余弦相似度，用于diegetic判定与分桶）；音频质量预测模型（输出1~10连续分，标注方式仿照 LAION aesthetic）；通用音频caption模型；音乐caption模型；影视感（cinematic）音视频分类器。
值得注意的是，音频质量分并非用作硬过滤阈值，而是被写进caption成为可控条件（推理时指定质量7.0/8.0），这是一种「把judge分数转成条件」的做法。而后训练阶段的语义级判断则交给人类标注员，而非大模型。

### 安全与合规过滤（NSFW、版权、人脸/隐私） ⚠️

[不确定] 技术报告未描述训练数据层面的 NSFW、版权、人脸/隐私过滤流程与工具。论文关于安全的表述集中在结论的「Safety considerations」段：模型为研究目的开发、部署前需多项改进；模型可能学到模态间的非预期关联；生成模型会学到各模态中的偏见（如视频训练数据中的视觉偏见、prompt语言中的偏见）；研究仅限英语文本输入；真正部署时会接入安全模型来拒绝违反政策的输入prompt或生成结果，以防滥用。数据侧的合规过滤、生成内容水印/来源标记均未披露。

## 打标方式

### 使用的caption模型（自研VLM/开源模型，模型规模）

视频侧：使用 LLaMa3-Video（Dubey et al., 2024）作为caption模型，对 8B 与 70B 两个规模的变体分别做视频captioning任务的微调，然后用它们标注全量训练clip。规模配比明确：训练集caption中 70% 来自 8B 模型、30% 来自 70B 模型（在成本与质量间取平衡）。这是原生的视频caption模型（直接吃视频），而非逐帧图像caption再拼接。
辅助模型：16类相机运动分类器（自训），高置信度预测结果作为前缀拼到caption上。
音频侧：使用多个模型协同生成合成caption——音频质量预测模型（输出1~10分）、AED模型（判定voice/singing存在性与music后验概率）、通用音频caption模型（自由形式描述声音）、音乐caption模型（补充mood与genre细节）。论文特别说明音乐caption模型主要在音乐样本上训练、无音乐时容易幻觉，因此同时保留AED的音乐概率与音乐caption两路信号，实测这种组合的可控性最好。
后训练阶段caption由人工在模型输出基础上精修。

### caption密度与结构化程度（短/长/密集描述、结构化字段如镜头运动、风格标签）

视频caption特征：稠密详细（detailed generated captions），平均长度约100词，具有「细节密集 + 段落结构一致」的统一风格。结构化成分包括：
· 相机运动前缀：16类相机运动分类结果（zoom in / zoom out / push in / pull out / pan right / pan left / truck right / truck left / tilt up / tilt down / pedestal up / pedestal down / arc shot / tracking shot / static shot / handheld shot）在高置信时前置拼接到caption，使推理时用户可显式指定运镜。
· FPS token：把帧率作为token加入caption，实现16~32 FPS的采样帧率控制。
· SFT阶段人工补全的字段：相机控制、人物表情、主体与背景信息、详细运动描述、光照信息；并新增标注6类镜别/机位（wide angle、close-up、aerial、low angle、over the shoulder、first person view），合计22类相机相关标签。
· 推理侧对齐：因用户实际prompt通常不足10词，与训练caption的长度和风格差异大，专门设计了 inference prompt rewrite 把短prompt改写成训练caption风格。
音频caption为高度模板化的结构化四段式（见 joint_av_caption_schema）。

### 音视频联合caption结构（是否同时覆盖视觉+听觉轨道、是否分流为独立字段，如LTX-2全音景描述、Script-a-Video factorized streams、Foley-Omni三字段）

Movie Gen 没有统一的音视频联合caption schema——视觉与听觉分属两条独立数据流：Movie Gen Video 的caption纯视觉、完全不描述声音；Movie Gen Audio 的caption纯听觉、不描述画面（画面信息由视频embedding直接条件输入，文本只作补充）。
但音频caption本身是结构化分流的四字段模板（表27），可视为「听觉轨道内部的factorized schema」：
① 音频质量：This audio has quality: 8.0.（1~10连续分，由音频质量预测模型给出，标注方式仿 LAION aesthetic）
② 人声与音乐存在性：This audio does not contain speech. This audio does not contain vocal singing. + This audio contains music with a 0.90 likelihood.（speech/singing用AED后验过阈值的二值输出；music因存在歧义——如影视中的riser既像音效又像音乐——保留为连续后验概率）
③ 声音描述：This audio has a description: 「gentle waves lapping against the shore, and music plays in the background.」（通用音频caption模型的自由形式描述）
④ 音乐风格描述：This audio has a music description (if applicable): 「A beautiful, romantic, and sentimental jazz piano solo.」（音乐caption模型输出mood与genre；无论样本是否含音乐都无条件追加此字段，配合②的音乐概率共同实现最佳可控性）
论文强调「优先视频对齐而非文本对齐」，因为文本只是补充、无法覆盖视频中的全部细节，且文本对最终观众不可见。

### 对白转写与说话人属性标注（ASR转写+说话人身份/语言/口音/情绪）

无。Movie Gen Audio 不做 ASR 转写，不标注说话人身份、语言、口音或情绪——因为模型有意不生成语音：diegetic speech 在没有台词脚本、且生成视频存在伪影时难以生成；non-diegetic speech（旁白）被认为可由TTS在给定脚本时补齐。对语音的全部处理仅限于：用 AED 模型（AudioSet 本体的 speech / singing 子类）判定样本中是否存在人声与歌唱，输出经预设后验阈值二值化后写入caption作为控制标签；以及在数据分桶时把 voice 作为三大音频类型之一参与 CAVTP 阈值判定。微调用的 cinematic split 更是直接把含人声（vocals）的片段整体排除。论文在局限性中明确承认：模型「由于设计选择目前不支持人声生成」。

### 几何与结构化标注（相机参数、深度、3D point tracks、动作标注、显式状态）

以相机语义标注为主，无显式几何量标注：
· 相机运动：自训分类器预测16类相机运动（推拉摇移升降、弧形、跟拍、固定、手持），高置信预测前缀入caption；SFT阶段人工额外标注6类镜别与机位（广角、特写、航拍、低角度、过肩、第一人称视角）。这是全paper最核心的结构化标注体系。
· 动作/行为：建立含600个人类动词与表情的taxonomy，用于zero-shot文本→视频检索与 text k-NN 检索，间接构成动作类目标注。
· 人脸/身份：PT2V数据用人脸检测器（每秒抽帧）、ArcFace 身份相似度（相邻帧>0.5判定为同一人贯穿全片）、人脸区域裁剪与分割（防止模型关注背景等非关键区域）；合成参考图用 ArcFace <0.7 剔除。
· 物体：SFT用 Detic 物体检测判断主体是否过小。
· 运动强度：FFmpeg motion score 与 motion vector 作为运动量标量，评测prompt还人工打了 high/medium/low 运动等级标签。
未使用相机内外参、深度图、3D point tracks、光流场等显式几何标注[确认为未采用]。

### 合成数据构造（受控扰动/编辑构造训练对，如InstructAV2AV）

大量且体系化地使用合成数据，但集中在个性化与编辑两个能力上，T2V主干仍用真实数据：
· 个性化（PT2V）cross-paired 合成：直接用paired数据训练会让模型学到copy-paste捷径（生成人物总是复刻参考图的表情与头部姿态、直视镜头）。除收集 O(10)K 条真实cross-pair（来自预训练数据中同一场景不同机位的片段）外，用一个预训练的个性化图像生成模型（He et al., 2024b）对paired数据每条视频的首帧生成参考图，通过多样prompt变化表情、头部姿态、光照等；用 ArcFace 相似度<0.7 剔除身份漂移的生成图，最终得到 O(1)M 条合成 cross-paired 样本，在预训练第三阶段专门用于提升自然度。
· 视频编辑（Movie Gen Edit）三阶段合成：论文的核心主张是「不依赖任何有监督视频编辑数据」也能获得SOTA视频编辑能力。阶段一多任务训练，把图像编辑当作单帧视频编辑与文生视频交替训练；阶段二引入两个合成的多帧编辑任务——Animated Frame Editing（对图像编辑样本对施加随机仿射变换生成「动画化」的多帧编辑对）与 Generative Instruction-Guided Video Segmentation（生成式指令引导视频分割），并与文生视频多任务微调约一千步；阶段三把NLP的 backtranslation 思想迁移到视频编辑：用阶段二模型生成编辑后视频，过滤后构造 (带噪的生成编辑视频 x̂, 反向编辑指令 c_instruct-bwd, 干净的真实视频 x) 三元组，训练模型从带噪编辑结果还原干净的真实视频，从而在多帧、高质量的真实输出视频上训练。消融显示 backtranslation 明显优于标准微调。

### 人工介入程度（人工标注、人工质检、模型初筛+人工复核）

预训练阶段基本全自动（人工仅用于阈值确定，如音频CAVTP阈值「由人工检视确定」、美学阈值经人工验证），后训练阶段则是重度人工介入的「模型初筛 + 人工复核 + 人工重标」范式：
· 视频SFT四阶段中后两阶段全靠人工：第三阶段人工判断影视感（角度光/自然光或棚拍光、色彩鲜艳但不过饱和、画面不杂乱、有非平凡运动、无相机抖动、无编辑特效与叠加文字）——论文明确指出「高质量微调数据的许多方面无法被自动过滤器以高精度和高召回可靠捕捉」；同阶段标注员还要亲手把视频裁剪到目标时长，主动挑出整段中最精彩、最有感染力的片段。第四阶段人工在 LLaMa3-Video 生成caption的基础上修正错误细节、补齐关键信息（相机控制、人物表情、主体与背景、详细运动、光照），并新增6类镜别标注。
· 个性化SFT：从T2V微调集出发挑单人视频后，人工挑选动作多样的视频以覆盖多种动作行为，最终得 O(1000) 条。
· 音频SFT：影视感音视频分类器 + AED 自动过滤后，接人工标注做最终选择。
· 评测端几乎完全依赖人工A/B测试（论文论证了自动指标不可靠），标注员经过专门培训并持续审计；还把同一批381条prompt的标注任务重复4次以估计各评测维度的标准差（整体质量5.07%、帧一致性4.08%等），用于判断胜负是否显著。

## 音视频对齐

### 音视频同步检测方法（口型同步、事件对齐）

不做口型同步检测（模型不生成语音，故 SyncNet/LSE-C 一类唇同步指标完全不适用）。音视频关联性的检测核心是 CAVTP（contrastive audio-video-text pre-training）模型：计算音频embedding与视频embedding的余弦相似度，作为「该音频有多大可能是画内音（diegetic）」的代理分数——因为CAVTP主要在以画内音为主的数据上训练，画内且与画面内容匹配的音频其音视频embedding距离更近。该分数与AED类型标签组合，构成预训练数据的分桶依据（表24）。
论文还从建模角度把音视频对应关系分为三个难度层级：①画内在屏音——音画对应极强、什么声音在什么时刻响是确定性的，要求强视频理解与稠密动作识别（如高尔夫球杆击球）；②画内画外音——需理解什么环境会出现什么声音、事件间的逻辑顺序（如林中鸟叫、难度动作完成后人群欢呼），要求更强推理；③画外音/非画内——仅在语义层面相关（背景音乐要匹配情绪、riser用于营造紧张感），要求对世界物理之外的人类情感建模。
评测端用人工评「diegetic sound synchronization」并配合 ImageBind score 客观指标。局限性中承认：当动作密集（踢踏舞）、目标小或被遮挡（脚步声）、或需要细粒度视觉理解（识别吉他和弦）时，生成音频会出现不同步。

### 同步检测具体指标与阈值（SyncNet/Synchformer/LSE/自研，阈值数值，如MOVA LSE-D≤9.5且LSE-C≥4.5、SkyReels-V4 SyncNet |offset|≤3∧conf>1.5）

使用自研 CAVTP 的音-视 embedding 余弦相似度作为唯一的量化对齐指标，阈值按音频类型分档给出（表24，均由人工检视 manual inspection 确定）：
· AED = sound / voice / voice+sound → 余弦相似度 > 0.2 判为 diegetic
· AED = music / music+sound / voice+music+sound → 余弦相似度 > 0.3 判为 diegetic（音乐类要求更高，因音乐与画面的天然相关性更弱、易误判）
· AED = music 且 相似度 < 0.1 → 判为 non-diegetic（背景配乐）
· AED = sound+music 且 0.1 < 相似度 < 0.25 → 判为 non-diegetic
· AED = sound+voice+music 且 0.1 < 相似度 < 0.25 → 判为 mixed（画内画外混合）
预训练只保留 diegetic 与 mixed 桶，外加少量 non-diegetic 背景音乐。
未使用 SyncNet、AV-align 等公开同步指标；评测阶段客观指标用 ImageBind score（音视频对齐）与 CLAP score（音文对齐），主观指标才细分为同步性（Sync.）与正确性（Corr.）。
其他相关阈值：PT2V 用 ArcFace 余弦相似度 > 0.5（相邻帧同一人）与 > 0.7（合成参考图身份保持）。

### 时序同步vs语义同步的分离处理（时间对齐与内容语义匹配作为两个独立过滤条件）

概念上做了清晰的三层分离，但数据过滤时用的是同一个 CAVTP 分数 + AED 标签的组合来近似区分，并未训练两个独立的时序/语义打分器：
· 概念层：论文把音视频关系拆为「画内在屏（时序确定性对应，最强时间约束）」「画内画外（环境合理性与事件因果顺序，语义+弱时序）」「非画内（情绪与叙事语义，无时序约束）」三层，并指出三层对模型能力的要求依次上升。
· 数据层：diegetic/non-diegetic 的划分本质上就是「有无时间因果约束」的划分，用 CAVTP 阈值实现；音频类型（voice/music/sound）的划分则承载语义内容维度。二者交叉成表23的六格分类，实际是把「时序可对齐性」与「语义类别」作为两个正交轴来管理数据配比。
· 评测层分离最彻底：diegetic sound correctness（该不该响这个声音——语义匹配）与 diegetic sound synchronization（响得准不准——时序对齐）被拆为两个独立的人工评测维度；non-diegetic 侧则拆为 music mood alignment（情绪语义）与 music motion/scene alignment（与动作/场景/剪辑点的对齐）。

### 音频质量过滤（SNR、静音检测与静音占比阈值、无音轨剔除、画外音源剔除、背景音乐分离）

· 静音处理：用 AED 打标后，直接丢弃 silence 为主导类（dominant class）的全部视频；未报告静音占比的具体阈值数值。
· 音频质量：训练了音频质量预测模型输出1~10的连续分（标注方式仿照 LAION aesthetic 的收集方式，10为最高）。关键设计是这个分数不作硬过滤阈值，而是写进caption成为可控条件——推理时对纯音效生成条件在质量分7.0、对音效+音乐联合生成条件在8.0。消融显示条件质量分越高预测质量分越高，但主观偏好在约6.5之后趋于饱和（含非画内音乐的样本则可继续受益到更高分）。
· 画外/无关声剔除：通过 CAVTP 低相似度识别并把非画内音分流出预训练主集；微调的 cinematic split 明确追求「抑制环境噪与无关的画外声音」的专业混音特性。
· 影视级音质定义：论文把影视原声与低端设备录音（手机、监控）的差距归纳为音质（how it sounds，需专业话筒录制 + 混音母带处理，去除pop噪、风噪）与声音设计（what sounds to include，突出与叙事相关的爆炸/对话事件，环境音乐做fade-in/fade-out混合）两个方面，用影视感分类器+AED自动筛+人工标注来弥合。
· 人声剔除：cinematic split 排除含vocals的片段。
· 视觉侧连带过滤：剔除OCR检出文字的、静态的、分辨率<480px的视频以降低视觉模态噪声。
· 未报告 SNR 阈值、也未做背景音乐分离（BGM separation）——相反，非人声音乐是刻意保留并建模的目标之一。

### 语音/音效/音乐的分类与分别处理策略

分类：AED（AudioSet 527类本体）把每个样本映射到 voice（speech + singing 子类）、non-vocal music（music 子类）、general sound（其余子类）三类，允许多标签共存；与 diegetic/non-diegetic 交叉后得到表23的六格（如画内voice=现场说话、画外voice=旁白；画内music=现场演奏、画外music=背景配乐；画内sound=海浪拍打、画外sound=罐头笑声与riser）。
处理策略：
· 建模上不分模型——论文明确选择「用单一模型联合生成所有音频类别」，理由是不仅视频与音频之间存在相关性，不同音频类别之间也彼此相关（若分开生成，音效与音乐无法和谐混音）。
· 数据配比上分类别调阈值与配额：general sound 被列为最高优先级（低层物理规律最难学、错误最刺耳），量级上比其他类别高约10倍；music 类的 diegetic 判定阈值抬高到0.3；voice 类被排除出生成目标（仅作为存在性标签）。
· caption 上分字段控制：speech/singing 用二值标签、music 用连续概率、一般音效用自由描述、音乐风格单独一段且无条件追加。
· 微调上分数据源：影视级音视频（同时含画内音与画外的环境音乐、主题音乐）+ 不带视频的高质量纯音乐 O(10)K 小时与纯音效 O(10)K 小时，后者数量更易获得、用于拉高音质。消融证明加入大规模文本-音效与文本-音乐配对数据能让模型有效解耦不同音频类型。

## 训练配合

### 多阶段训练课程与数据课程调度（阶段划分依据：分辨率/时长/质量分/模态；低清→高清、图像→视频、短→长）

· Movie Gen Video 三步课程（数据侧配套3个逐级更严的子集）：①先训纯文生图（T2I）热身，再联合训练 T2I + T2V（直接从零联合训练收敛显著更慢、同等GPU时下视觉与时序质量都更差）；②分辨率渐进，从256px低清训练到768px高清训练，数据从「最小宽高720px的低清集」切换到「最小宽高768px的高清集」；③用改进的数据集与优化配方持续训练（compute与时间受限下的continuous training）。训练中维护未见视频的验证集，验证loss与人工评的视觉质量高度相关，loss平台期即降学习率。分桶（5个时长桶 × 5个宽高比桶，桶内latent形状一致）保证变长变形训练可批处理，FPS token 提供帧率课程。最多使用6144张H100（700W TDP、80GB HBM3、Grand Teton平台）。
· PT2V 三阶段：Stage-I 身份注入（截断TAE embedding到8个latent帧即64个RGB帧，只训backbone、冻结视觉编码器，用paired样本快速注入身份）→ Stage-II 扩到长视频 → Stage-III 用 cross-paired（真实+合成）样本消除copy-paste效应、提升表情与运动自然度。
· Movie Gen Edit 三阶段：单帧编辑多任务 → 合成多帧编辑 → backtranslation 视频编辑。
· Movie Gen Audio 两阶段：预训练（序列上限30秒/750 token，batch 1536序列，500K步，384 GPU × 14天，常数lr 1e-4 + 5K步warmup）→ 微调（batch 256，50K步，64 GPU × 1天，lr线性升到1e-4再线性衰减到1e-8，EMA decay 0.999）。

### 数据配比随训练阶段的变化（预训练/退火annealing/SFT高质量子集）

· 视频预训练：低清阶段用720px门槛集（宽高比 60%横屏/40%竖屏）；高清阶段切到768px更严集，并新增数据扩充，配比调整为 80%横屏/20%竖屏 且 ≥60% 含人物——即随着阶段推进，分辨率门槛、美学/运动阈值、人物占比三者同时收紧，是典型的「质量随阶段单调提升」的数据课程。
· caption混合：全训练集中 70% 的caption来自 LLaMa3-Video 8B、30% 来自 70B。
· 视频SFT：数据完全换成人工精选集，时长限定10.6~16秒（50%恰为16秒），并按时长切换训练帧率（16秒用16FPS，10.6~16秒用24FPS），使模型同时支持10秒与16秒两种输出；训练多个SFT模型（不同微调数据版本、不同超参、不同预训练checkpoint）后做 model averaging（仿LLaMa3）以融合各模型在运动、一致性、相机控制上的不同长处。
· PT2V SFT：paired 与真实 cross-paired 参考图按 1:1 比例混合。
· 音频微调：影视级音视频（Cin-AV）与高质量纯音频（HQ-A）按 10 batch : 1 batch 混合；10秒chunk与30秒chunk按 5 batch : 1 batch 混合。
· 音频训练的条件dropout也构成一种「模态配比」：整体条件（视频+文本+音频上下文）以0.2概率全丢，文本与视频各自独立以0.1概率丢弃以降低对单一模态的依赖；masked audio 以0.5概率完全mask（纯生成），否则mask 75%~100%（音频扩展）。

### 后训练数据（SFT精选集规模与筛选标准、偏好对数量与标注方式、reward model训练数据） ⚠️

· 视频SFT集：目标是「运动好、真实感强、美学佳、概念覆盖广、caption质量高」的高质量视频。从大规模池经四阶段（自动严阈值→概念平衡k-NN→人工挑影视感→人工caption）产出，第一阶段后剩「几百万条但概念不均衡」，最终规模论文未披露[不确定]，但训练只用512张H100，属于极小规模。时长10.6~16秒，50%为16秒。
· PT2V SFT集：O(1000) 条高质量单人视频，人工挑选保证人类动作多样性，paired与真实cross-paired参考图1:1。
· 音频SFT集：cinematic split（专业制作、含画内音与画外环境/主题音乐、排除含人声片段，由影视感分类器+AED自动筛后人工选定）O(100)K 样本 / O(1)K 小时；high quality audio split（无视频的高质量音乐 O(10)K 小时 + 音效 O(10)K 小时）O(1,000)K 样本 / O(10)K 小时。
· 未使用偏好数据：全流程没有偏好对标注、没有 reward model、没有 RLHF/DPO；后训练完全是SFT + model averaging。人工评测数据仅用于评估而非训练。

### 数据处理基础设施与吞吐（NeMo Curator/Data-Juicer/自研，GPU加速比，处理规模，成本） ⚠️

[不确定] 论文未披露数据处理侧的基础设施、框架（未提及 NeMo Curator / Data-Juicer 等）、GPU加速比、单位吞吐与成本。可从文中推断的只有工具栈构成：开源工具用 FFmpeg（场景检测、VMAF motion score、motion vector）与 PySceneDetect（shot boundary detection），其余为一系列 Meta 内部模型（OCR、静态检测、美学、copy-detection embedding、视频-文本联合embedding、相机运动分类器、AED、CAVTP、音频质量打分、音频/音乐caption等）。训练侧基础设施披露充分：最多6144张H100（700W TDP、80GB HBM3、Grand Teton AI服务器平台），自研的PyTorch模型并行实现并编译为CUDAGraphs，配有解析框架建模计算与通信时间以消除重复激活的跨GPU通信；音频预训练384 GPU×14天、微调64 GPU×1天。考虑到最严阈值下接受率<1%，其数据清洗的算力开销应相当可观，但论文未给出数字。

## 效果对比

### 数据策略消融的量化影响（区分：过滤严格度ablation / caption密度风格ablation / 数据配比ablation，及对应评测指标）

论文给出多组与数据直接相关的量化消融（均为人工A/B的净胜率 win%−lose%）：
【caption密度与风格 ablation】LLaMa3-Video 8B 原生视频caption vs LLaMa3-FramesRewrite（对首/中/尾三帧做图像caption再用LLaMa改写合并）。①caption本身质量：人工A/B中视频caption被偏好67%，逐帧改写方案仅15%；②对生成模型的影响：整体prompt对齐提升 +10.8%，其中绝大部分增益来自运动对齐 +10.7%，在要求高运动的prompt上更是 +16.1%。结论是原生视频caption能准确描述更细粒度的运动细节，为训练提供更强的监督信号。
【过滤严格度 / 高质量SFT ablation（T2V）】微调模型 vs 预训练模型（24FPS、10.6秒）：整体质量 +34.65、帧一致性 +8.14、运动完整性 +18.38、运动自然度 +10.5、文本对齐 +9.97。
【高质量SFT ablation（音频）】FT vs PT：整体 +41.7±15.3、自然度 +37.8±16.3、专业度 +43.0±14.7、画内音正确性 +31.0±16.0、同步性 +24.9±17.7；论文评语为「微调后生成结果明显更具影视质感，凸显了微调阶段高质量数据curation的重要性」。
【数据配比 ablation（音频）】Cin-AV + HQ-A vs 仅 Cin-AV，在SFX任务上：整体 +21.5±18.7、自然度 +24.3±17.3、专业度 +22.8±18.7、画内音正确性 +11.7±17.4、同步性 +9.6±18.1；在SFX+音乐任务上对视频-音乐对齐提升显著。解释是大规模文本-音效与文本-音乐配对数据使模型有效解耦不同音频类型，连带提升音视频对齐。
【合成数据 ablation（PT2V）】cross-paired 训练会略微降低 ArcFace 身份相似度，但换来显著更多样的头部姿态与更自然的表情；消融表明 cross-paired 数据与SFT对视频自然度、质量与文本对齐都很关键。
【合成数据 ablation（编辑）】backtranslation 训练优于在同阶段做标准微调；多任务学习（文生视频+图像编辑）优于在冻结T2V上训ControlNet。
【条件质量分 ablation（音频）】提高条件音频质量分可持续提升客观质量分，主观偏好在约6.5饱和（含非画内音乐的样本可继续受益）。

### 质量vs数量的证据（小而精数据超越大而杂的案例）

证据非常强，且是全文的核心方法论主张。①预训练侧：最严格阈值下的数据接受率低于1%（漏斗末端0.94%），即从原始池中只挑出约百分之一的视频用于高分辨率训练——用极低的保留率换取质量。②后训练侧：视频SFT只用一个人工精选的小规模集合（训练仅512张H100，相比预训练最多6144张H100），却带来整体质量 +34.65 净胜率的巨大提升，是全文最大的单项质量增益之一。③音频侧：预训练用 O(1,000)K 小时（百万小时量级）数据，而微调仅用 O(1)K 小时影视级音视频 + O(10)K 小时高质量纯音频（合计不到预训练的1.1%），却在所有主观维度上取得 +24.9 至 +43.0 的净胜率，「小而精压倒大而杂」的对比极为直观。④论文结论原文总结这一配方：「我们专注于为预训练curate高质量的大规模数据，并为微调curate规模相对更小但质量更高的数据。这一通用配方对提升图像、视频与音频生成质量都很有效。」⑤同时论文也强调了规模的必要性——「同时扩展数据、训练算力与模型规模才带来显著提升」，因此其立场是「先用大规模严过滤数据打底 + 再用极小规模人工精选数据定调」，而非单纯的小数据路线。

### 训练数据domain分布与评测基准类目体系的对齐关系（如VABench七大类）

部分对齐，且对齐关系可追溯：
· Movie Gen Video Bench（1003条prompt，规模是此前工作prompt集的3倍以上）设置5类概念：①人类活动（肢体与口部运动、情绪等）②动物③自然与风景④物理现象（流体力学、重力、加速度、碰撞、爆炸等）⑤非常规主体与非常规动作；并对每条prompt额外打上 high/medium/low 三档运动等级标签。其中①②③直接对应训练数据声明覆盖的「人类、自然、动物、物体」domain；运动等级标签则与训练侧的运动过滤维度（motion score / motion vector / 抖动率）一一呼应；④物理现象与⑤非常规主体属于刻意设置的分布外（out-of-distribution）测试，用于检验泛化，训练数据并未针对性配比。
· 人物权重的一致性最明显：训练高清集 ≥60% 含人 + 600个人类动词/表情taxonomy定向检索，与benchmark中人类活动作为第一大类、并单列「肢体与口部运动、情绪」的细分完全一致。
· Movie Gen Audio Bench 的类目体系与音频训练数据的两条主线对应：单镜头视频（数量大、音效谱系广，测画内音效的鲁棒性与泛化，对应预训练中占绝对主导的 general sound / diegetic 数据）与多镜头视频（取自短片，含转场与更强叙事，测视频-音乐对齐、音乐进入时机、与剪辑点的呼应、音效与音乐的混音和谐度，对应微调中的 cinematic split 与 non-diegetic music 数据）。评测指标体系（音质 / 视频对齐 / 文本对齐三大轴，视频对齐再拆画内音正确性、画内音同步性、画外音乐情绪对齐、画外音乐动作场景对齐）与数据端 AED × CAVTP 的二维分类体系高度同构。
· 视频侧未采用类似 VABench 七大类的外部统一类目体系，两个benchmark均为 Meta 自建并随论文开源。

## 不确定字段

以下字段的调研信息部分不确定（⚠️ 标注来源）：

- data_scale
- data_sources
- provenance_licensing
- funnel_retention_rate
- safety_filtering
- post_training_data
- data_infra_throughput
