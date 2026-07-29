# 音视频联合生成模型：数据与训练方案（综合最佳实践）

> 基于本 wiki 34 个条目的横向对比提炼，优先采信两类来源：**2025H2–2026 最新工作**（Seedance 1.5pro/2.0、Kling 3.0 Omni、Sora 2、Veo 3.1、LTX-2、MOVA、Apollo、Vidu S1、CineDance 等）与**披露最详尽/有定量消融证据的工作**（Movie Gen、Allegro、Goku、HunyuanVideo、Open-Sora 2.0、Cosmos-Predict2.5、Koala-36M、Data-Juicer 等）。每条实践括号内标注来源。
>
> 生成日期：2026-07-29 · [← 返回首页](index.md)

---

## 0. 方案总览（TL;DR）

| 环节 | 推荐做法 | 关键数字锚点 |
|------|----------|--------------|
| 原始池 | 千万小时级采集，无音轨视频前置剔除 | HunyuanVideo 1.5 >1000万h；Cosmos 3500万h |
| 清洗漏斗 | 14 级级联：便宜规则在前、VLM 终审在后 | 端到端保留率：AV 联合 ~25%，纯视频严格档 1–5% |
| AV 对齐 | 时序（Synchformer/SyncNet）与语义（ImageBind）**分拆双指标** | SyncNet \|offset\|≤3 ∧ conf>1.5 已成事实标准 |
| 打标 | 音频可感知模型 + 密集结构化 caption + 多粒度混训 | 100–250 词、短/中/长三档随机采样 |
| 视觉课程 | T2I 打底 → 256p→480p→720p 金字塔，量退质进 | 数据量逐级缩：2亿→1亿→100万（HunyuanVideo 1.5） |
| 模态课程 | 海量纯音频先训音频塔 → 少量配对数据联合适配 | 跳过则音频崩塌：WER 恶化 4 倍（Foley-Omni） |
| SFT | 人工精选，约预训练量的 0.4–1.5% | HunyuanVideo 100 万条；NAVA 保留率 1.07% |
| 偏好对齐 | GRPO 在线 RL + 多维奖励（含音视频专用 RM） | SkyReels-V2 3 万偏好对训 RM + 三轮 DPO |
| 基建 | NeMo Curator（GPU）或 Data-Juicer（CPU）+ 分桶落盘 | 2000×H100 一天处理 ~100 万小时 720p |

核心信念（有强证据支撑，见 §9）：**数据处理质量本身是最大杠杆**——同源素材、同架构下，仅改进处理即可 VBench +9.67 点（Koala-36M）；**优质数据重复 6 遍胜过次优数据配 8 倍算力**（Data-Juicer）。

---

## 1. 数据规模与采集目标

**预训练（视觉主干）**：原始池千万小时级，经漏斗保留 1–5% 得数亿 clip（HunyuanVideo 1.5：>1000 万小时 → 8 亿 clip；Cosmos-Predict2.5：3500 万小时 → 60 亿候选 → 2 亿 clip，保留率约 4%）。

**音视频配对语料**：可低一档，万小时/千万对级即可起步（MOVA 三阶段 61.5k→37.6k→11k 小时；ALIVE 联合训练 11M 对；Movie Gen 音频预训练百万小时级）。

**音频塔单独预训练**：数十万小时纯音频，语音为主（ALIVE 70 万小时转写语音；Ovi、SkyReels-V4 数十万小时；Unison 13 万小时）。

**SFT**：百万级人工精选（HunyuanVideo ~1M；SkyReels-V4 5M→1M 两段）。

**分阶段金字塔**：数据量随分辨率/质量逐级收缩（HunyuanVideo 1.5：480p 2 亿→720p 1 亿→CT 100 万；Goku：480p 36M→720p 24M→1080p 7M）。

---

## 2. 时长切分与分辨率策略

**切分**：TransNetV2 场景切分（NeMo 实测 F1=0.967，优于 PySceneDetect/AutoShot）或双工具互补（LongCat、SkyReels），切后用 embedding 相似度 stitching 缝合防过切（NeMo、Motif、Panda-70M），首尾丢 3–10 帧去转场残留（Step-Video、Allegro）。

**AV 特有硬约束**：切点必须语音感知（speech-aware）——用 VAD + 场景切点联合驱动切窗，不切断句子（MOVA 8.05s 定长窗口 = 193 帧@24fps；Vidu S1 同）。

**训练主体时长** 5–15 秒（HunyuanVideo 1.5 统一 2–10s；Movie Gen 4–16s 且 >50% 落在 15–16s）。2026 年训练窗口延至 15 秒级（Seedance 2.0 直出 4–15s、Kling 3.0 目标 15s）。定长（MOVA/Ovi/HunyuanVideo-Foley）与时长分桶（Movie Gen 五桶、SkyReels-V2 双轴分桶）两条路线并存，定长实现简单、分桶利用率高。

**分辨率**：渐进课程 256p→480p→720p 为共识（Cosmos、HunyuanVideo 1.5、MOVA——后者消融证明先低清对齐再升清有效），末端小规模高清精修（Cosmos 4K 冷却 388K 条）。原生高清显著优于低清+超分（UltraVideo：VQAA 73.46 vs 69.26）——**分辨率是唯一无法靠后期筛选弥补的维度**，采集时就要保真。1080p+ 可外包给独立超分模块压成本（HunyuanVideo 1.5）。

**宽高比**：唯一定量配比来自 Movie Gen——预训练 60% 横屏 + 40% 竖屏，高分辨率阶段 80/20，五个 AR 桶；简化路线为 16:9/9:16 两种补边归一（MOVA）或等面积归一（Ovi 固定像素积，天然支持任意 AR）。

---

## 3. 清洗漏斗（视觉侧）

**总原则**：算力递增的级联审查——便宜规则粗筛在前 → 专用小模型打分居中 → 昂贵 VLM 终审与打标在末端（Cosmos、Allegro、MAGI-1、MOVA、Apollo 一致）。趋势是让 VLM 的"判分"与"打标"共用同一次前向（MAGI-1/Motif），构造性消除过滤与条件化的漂移。

**推荐级联顺序**（综合各家）：

1. **音轨存在性检查**（AV 模型一票否决，置于最前：MOVA、HunyuanVideo-Foley、UniVerse-1）
2. **硬指标预筛**：时长≥2s、分辨率≥720p（Ovi >720×720）、码率≥500kbps–1Mbps（Goku、Foley-Omni，防"伪高清"）、fps≥23
3. **MD5/源 ID 精确去重**
4. **TransNetV2 切分 + stitching + 首尾去帧**（语音数据 VAD 联合切窗）
5. **黑边/字幕/水印检测并裁剪**——优先裁剪而非丢弃（HunyuanVideo 1.5：裁后保留<60% 才弃；ffmpeg cropdetect 检黑边）
6. **美学 + 技术质量**：LAION 美学分 ≥4.0–4.75 起步、逐级收紧至 5.3（Movie Gen 4.0 → Allegro 4.8→5.3）；DOVER 技术分兜底；OCR 文字面积 ≤1–2%（Goku）
7. **光流双边运动过滤**：剔静止也剔抖动（RAFT/UniMatch；Goku 480p 档 0.3–20；Movie Gen 用"每秒镜头数≥0.85"作抖动代理）
8. **音频质量过滤**（见 §4）
9. **AV 对齐过滤**（见 §4)
10. **NSFW/安全分类器**（Seedance 专用分类器；轻量方案 LAION CLIP NSFW）
11. **SSCD 语义去重**：embedding 聚类→簇内比对→保留最高分辨率版本（Cosmos/NeMo/Seedance 1.0；NeMo 实测去掉约 30% 数据）
12. **VLM 终审**（Qwen2.5-VL 链末复筛：Cosmos；与打标共用前向：MAGI-1）
13. **MLLM 密集打标 + 独立通道反向校验**（OmniHuman 用跟踪/ASR 确定性输出反查 MLLM 幻觉）
14. **概念均衡重采样 + 分桶 sharding**（见 §6）

**保留率锚点**：AV 联合漏斗端到端约 1/4（Apollo 27%、MOVA 26.39%）；纯视频严格档 1–5%（Cosmos 4%、Movie Gen 高清档 <1%，其中分辨率、宽高比、OCR 三刀各砍 72–75%）；SFT 精品率再低两个数量级（NAVA 15M→160K 约 1%）。

**工程哲学分歧**（按需选择）：多阈值串联（主流）vs 单一学习式综合分（Koala-36M VTSS，论据是子指标不正交）；硬过滤 vs 全量打分存元数据、下游按需收紧（CineDance——推荐大池采用，灵活支持课程分层）；低质数据滤掉 vs 损失隔离（UniVerse-1 LQLS：低质数据仅在扩散 t>800 施加损失，去掉后 ID 一致性 0.89→0.78——"限制作用范围"优于直接丢弃）。

---

## 4. 音频过滤与音视频对齐

**推荐音频侧过滤链**：

1. 无音轨/解码失败剔除 → 2. 格式归一（44.1kHz）→ 3. 音量≥−60dB（Ovi/SkyReels-V4）+ 静音占比分档（**语音数据 <0.2 / 音效数据 <0.8**——HunyuanVideo-Foley 刻意宽松以保住"寂静+一声关门"型 Foley 样本）→ 4. 带宽验真：频谱检测有效采样率 >32kHz，剔除上采样伪高清（HunyuanVideo-Foley）→ 5. 感知质量：**Audiobox-Aesthetics 已取代 SNR 成为事实标准**（MOVA：PQ>5.0 / CU>4.5 / CE>2.5）→ 6. 音频四分类（语音/歌唱/音效/音乐，SkyReels-V4 用 Qwen3-Omni）→ 7. 分流对齐（下述）→ 8. 全量分数存元数据（CineDance）。

**时序 vs 语义分拆是强共识**（2025H2 起）：Synchformer/AV-align 管时序、ImageBind 管语义，盲区互补。组合逻辑：

- **通用音景数据用 OR 门**（MOVA：ImageBind≥0.2 OR DeSync≤0.5）——环境音无尖锐 onset 会被时序检测误杀，快动作音效会被语义检测误杀，OR 两头保住多样性
- **精标子集用 AND 门**（Foley-Omni：Synchformer≥0.2 ∧ ImageBind≥0.3）——纯度优先

**唇同步支路**（语音子集）：先 ASD 剔画外音（Unison lip-filtering、Vidu S1）→ 人脸框内跑 SyncNet（避免全帧稀释信噪比，Unison）→ **|offset|≤3 帧 ∧ conf>1.5**（Ovi 提出，SkyReels-V4、OmniCustom 原样继承，已成社区事实标准）；高质量子集加严至 LSE-D≤9.5 ∧ LSE-C≥4.5（MOVA 以此筛出 2.5M 唇音子集）。Ovi 实验结论：极少量不同步数据即损害唇同步能力，**宁严勿宽**。多人场景采用全员通过制 + SyncNet 做"谁在说话"归属匹配（OmniHuman）。

**警示**：SyncNet 对平坦环境噪声给虚高分，必须先按音频类型分层再分层设阈（Script-a-Video）；过滤器与评测指标同源（SyncNet 过滤 + Sync-C 评测）存在循环论证风险（ITS-JAVG）。

**BGM 处理**：通用音景模型不做分离、保留原生混音（Ovi/MOVA/HunyuanVideo-Foley）；人物中心数据例外——Demucs/Mel-RoFormer 拆人声/背景双轨分别监督（OmniHuman、Unison）。

---

## 5. 打标方案

**模型选型**：AV 打标必须用音频可感知模型，纯视觉 VLM 无法胜任（Ovi/Foley-Omni/OmniHuman 共同强调）。三条路线按预算选：

- **开源分工 + 大 LLM 融合**（MOVA：MiMo-VL-7B 视觉 + Qwen3-Omni 转写/音频 + GPT-OSS-120B 融合裁决——最大模型放融合而非感知环节）
- **闭源打精品、蒸馏到开源打全量**（Script-a-Video：Gemini-2.5-Pro 标 50 万条→微调 Qwen3-Omni，学生错误率已略优于教师；NAVA 双档位）
- **自研微调**（数据量越大越值得：Movie Gen 微调 LLaMa3-Video 8B/70B；SkyReels 72B 教师蒸馏进 Qwen2.5-VL-7B，影视字段准确率 76.3% 反超教师）

captioning 是全 pipeline 速率瓶颈（NeMo Curator），需 FP8 量化 / vLLM 服务化。

**Caption 结构**：长密集（100–250 词）+ 结构化槽位，固定覆盖：主体外观→动作→场景→运镜/景别→光照→风格（LTX-2、Movie Gen ~100 词、HunyuanVideo 七字段 JSON）。风格准则：只述事实、禁套话开头、按时间顺序（LTX-2、CogVideoX）。**多粒度是标配**：短/中/长三档并存 + 训练时随机采样（Motif 0.5/0.3/0.2）或字段 dropout（HunyuanVideo），解决训练长 caption 与用户短 prompt 的分布错配；推理侧配 prompt rewriter（CogVideoX/MAGI-1）。可控标记内嵌：运镜类别（Movie Gen 16 类+6 机位）、质量标签、"This video has no subtitles."（MOVA）。

**AV 联合 caption schema**（2026 核心分歧点，三条路线）：

1. **融合式全音景单段**（LTX-2：对白精确转写+说话人/语言/口音+音乐/环境音全覆盖；MOVA 标注时三轨互斥分流→融合）
2. **内联标签式**（Ovi：`<S>台词<E>` 按时序交织+末尾音景块，消融证明统一编码优于分离编码；ALIVE 台词/音效均内联时间轴）
3. **分字段 factorized**（Foley-Omni 三字段 `[WORDS]/[AUDIO]/[MUSIC]` 可独立验证与置空；Script-a-Video 四流 MTSS——同架构下人评音视对齐 +56%、WER 0.84→0.13）

**通用防幻觉设计**（各路线通用）：视觉轨严禁参考音频、音频轨严禁反推画面（MOVA、Vidu S1）；声学能量门控置空字段（Foley-Omni −35dB）；ASR 真值填占位符防台词幻觉（OmniHuman）。

**ASR 与说话人**：Whisper-V3 为主流（UniTalking/SkyReels/UniVerse-1）；说话人属性标准集 7 项：age/gender/accent/pitch/prosody/emotion/speaking rate（Ovi 体系）；"谁在说话"绑定用 ASD（TalkNet）+人脸跟踪+ArcFace（ALIVE、OmniHuman）；Omni 大模型+窗口化在说话人-角色绑定上（95.4%）远超 Pyannote 类专用工具（~63%）（CineDance）。转写保留原语言禁止翻译（MOVA）。

**几何标注**：通用 AV 生成不需要数值相机参数——运镜用语义标签即可（Movie Gen/HunyuanVideo/SkyReels 全走分类标签）。只在数字人/多人对话（人脸轨迹、DWPose、脸-声矩阵）或 Physical AI（深度/3D box）场景加。

**合成数据**：主干必须真实原生音轨；合成只用于填补真实数据不存在的配对——编辑对（InstructAV2AV）、音色解耦对（UniTalking 用 TTS 合成 690 万条防内容泄漏）、TTS 补长尾语种（SkyReels-V4）。

**人工介入**："人不进标注流、人进决策环"：①阈值标定抽检（MOVA、Open-Sora Plan 2000 条验证）②种子标注训打分器（CogVideoX 2 万条、SkyReels 9.3 万条运镜）③抽检（SkyReels：预训练 0.01%、后训练 0.1%）④SFT 末端逐条精选+caption 人工修订（Movie Gen、HunyuanVideo 100 万条人评）⑤偏好标注。

---

## 6. 分布配比与概念均衡

**概念均衡 = embedding 聚类 + 逆频率重采样**（强共识）：VideoCLIP 聚 1 万概念中心（HunyuanVideo）、12 万+簇兼做离群质检（Step-Video）、按簇大小 1/√size 采样（Movie Gen）。均衡代价可达砍半数据（SkyReels-V2）。

**人物刻意加权**：高清集 ≥60% 含人 + 600 个人类动词 taxonomy 定向检索（Movie Gen）；9 大类 86 子类显式均衡并倾斜人物（Goku）。负面清单：游戏/动画/录屏/口播（Cosmos、CogVideoX、SkyReels）。

**音频类别配比**（最大分歧点）：语音主导（MOVA 69.47%、Foley-Omni 66%）vs 音效主导（Movie Gen 音效:其他≈10:1、UniVerse-1 非语音 84.6%）。从零构建通用模型推荐**全类保留 + 标签化配比**（HunyuanVideo-Foley），BGM 按比例管控而非一刀切剔除（ALIVE），2026 年趋势歌唱单列为第四类（SkyReels-V4、Apollo）。阶段调度："大规模语音打底→混入音效保活语音"（Ovi；NAVA audio:AV 从 3:1 反转为 1:2）。

**语种**：中英双语为工业标配（MOVA 中文剧集+英文 YouTube；Seedance 2.0 覆盖方言/戏曲；Kling 3.0 五语种+口音控制）；accent 作显式标注字段（LTX-2、Ovi）。

**配比对齐评测基准**：训练 taxonomy 与评测类目一一对应（Movie Gen 五类概念；Cosmos 五数据域↔PAI-Bench 七域）；评测弱项→动态补对应类目数据的闭环（MAGI-1、Apollo、Seedance 1.0）。注意 Data-Juicer 式"VBench 指标直接嵌入配方搜索"最彻底但有过拟合基准风险。

---

## 7. 训练课程与数据调度

**视觉主干金字塔课程**（共识）：

1. **T2I 打底**：纯图像建立视觉先验（Movie Gen 证明从零联合训练显著更慢；LongCat-Video T2I 占总迭代 42%），图像在后续所有阶段持续混训防画质退化（HunyuanVideo、Goku、MAGI-1 图像:视频恒定 4:1）
2. **分辨率递进 + 数据量递减**：CogVideoX 256→480→768px、batch 2000→100；Open-Sora 2.0 三阶段 70M@256px→10M→5M@768px（高清阶段改 I2V 承接省算力）
3. **末段只升帧率不升分辨率**（Seedance 1.0、HunyuanVideo 1.5：16→24fps）
4. **退火**：top 10–20% 高质子集小步数精调（CogVideoX stage4 用 top 20%/10k 步，代价是语义能力轻微退化）；4K 冷却（Cosmos 388K 条）

切换判据按"收敛且视觉质量饱和"而非固定步数（Cosmos）；换更高质量数据时 loss 骤降可作切换信号（Step-Video）。**质量分层即课程**：同一语料按阈值切嵌套子集逐阶段收紧（Allegro 美学 4.8→5.3，且昂贵指标只在数据缩两个量级后才计算——关键的成本设计）。

**音视频模态课程**（事实标准，Ovi/MOVA/Harmony/ALIVE/SkyReels-V4/Unison 全部采用）：

- **先海量纯音频训透音频塔 → 少量配对数据联合适配**（音频阶段 epoch 远多于联合阶段，JavisDiT++ 为 50:2）
- **消融证据**：跳过课程直接联合训练则音频崩塌——Foley-Omni WER 恶化 4 倍、InstructAV2AV FAD 恶化 88%（音频梯度被视觉淹没）
- **联合阶段保护机制**：损失加权 λ_v=0.85/λ_a=0.15（Ovi）；非对称学习率 video 1e-5/audio 1e-6（ALIVE）；混入 50% 纯视频防视觉退化（SkyReels-V4）；防遗忘回放占比可达 70%（Foley-Omni）
- **端到端优于冻塔训 Bridge**：MOVA 实测"先冻双塔训桥"早早到平台期，改三塔端到端 + 文本 dropout 0.5→0.2 强迫跨模态对齐，LSE 逐阶段单调改善
- 强基座可免课程：UniVerse-1（Wan+Ace-step 双专家）单阶段 + LQLS 替代数据分级

**动态配比**：评测指标反馈自适应调采样比（MAGI-1、Apollo、Seedance）正在取代静态课程表；替代方案"各域独立 SFT + model soup 参数合并"规避配比调参（Cosmos）。

---

## 8. 后训练：SFT 与偏好对齐

**SFT**：规模约预训练量的 0.4–1.5%（HunyuanVideo 100 万人工精选、美学四项+运动三项标准；NAVA 保留率 1.07% 且用强模型重写 caption——数据与标注双重提纯）；概念平衡 k-NN + 人工挑影视感（Movie Gen，仅 512 卡）；高美学:写实 3:1、只训 0.5 epoch 防过拟合（ALIVE）。定量收益：Movie Gen 视频 SFT 整体质量净胜率 +34.65；音频微调用 <1.1% 预训练量换各维 +24.9~+43.0。

**偏好对齐**：工业标配四段式"预训练→CT→SFT→RLHF"（Seedance、Kling 3.0、HunyuanVideo 1.5）。做法演进：

- 离线 DPO：3 万偏好对训 Bradley-Terry RM + 三轮各 2 万对（SkyReels-V2）；GSB 人工标注（HunyuanVideo 1.5）
- **2026 趋势：GRPO 在线 RL + 多奖励模型**（LongCat-Video、Cosmos：HPSv3 + VideoAlign，灰度输入隔离运动评估抑制 reward hacking）
- 音视频专用：AV-DPO 2.5 万偏好对、六模型多维奖励、"模态感知排序"避免优质音频配劣质视频污染信号（JavisDiT++）；音视频定制 RLHF + 多维 RM（Seedance 1.5 pro）
- 口型/音色维度的 RM 是当前空白，开源侧几乎全部缺失偏好对齐——这是与闭源的最大差距

**蒸馏**：低成本最后一公里已成标配（步数 <1k；Vidu S1 三阶段：双向教师→因果适配→DMD 蒸馏）。

---

## 9. 为什么这么做：最强定量证据

| 结论 | 证据 | 来源 |
|------|------|------|
| 数据处理质量是最大杠杆 | 同源素材同架构，数据量减半，VBench +9.67 点（语义 +28.2） | Koala-36M vs Panda-70M |
| 质重于量 | 12.09% 保留率登顶 VBench（82.53%），算力低 22 倍；优质数据×6 遍 > 次优数据×8 倍算力 | Data-Juicer Sandbox |
| 结构化 caption 提升语义/动态 | 只换 caption：文本对齐 7.73→15.36、动态度 +107% | MiraData |
| 结构化 AV 脚本零成本增益 | 同架构只换标注格式：音视对齐 +56%、WER 降 85% | Script-a-Video |
| 高质量 SFT 值得人工投入 | <1% 精品率，整体质量净胜率 +34.65 | Movie Gen |
| 原生高清 > 低清+超分 | VQAA 73.46 vs 69.26，唯一无法靠筛选弥补的维度 | UltraVideo/OpenVid |
| 低质数据隔离优于丢弃 | LQLS（仅 t>800 施损失）去掉后 ID 一致性 0.89→0.78 | UniVerse-1 |
| 音频塔课程不可跳过 | 直接联合训练：WER×4（Foley-Omni）、FAD +88%（InstructAV2AV） | Foley-Omni / InstructAV2AV |
| 小模型+精数据可越级 | 2B 参数 <10M 片段，VBench 83.76% 超 Wan2.1-14B | Motif-Video 2B |
| 低成本可逼近 SOTA | $199.6k 全程训练，与 Sora 差距 0.69% | Open-Sora 2.0 |

（较弱证据需谨慎引用：HunyuanVideo 1.5"0.125% 保留率+8.3B 对标更大模型"无受控对比；Ovi"少量不同步数据即损害唇同步"实验未披露。）

---

## 10. 数据基建与成本参考

- **GPU 路线**：NeMo Curator 为工业事实标准——2000×H100 一天处理约 100 万小时 720p，等功耗较 CPU 89 倍加速，按"分辨率×宽高比×时长"分桶落盘直接对接课程训练
- **CPU 路线**：Data-Juicer——算子重排序省 70% 时间，8 节点 5TB 去重 2.8h
- **自研**：Seedance（BMF+Ray 异构三层）、MOVA（Ray + GPU/NPU 混合）
- **吞吐即训练收益**：离线 bucket 均衡采样器使 GPU 利用率 20%→90%、训练吞吐 5.4 倍（Motif-Video）
- **成本锚点**：Open-Sora 2.0 全程 $199.6k（11B，不含数据处理）；MOVA ~43,000 GPU-days；Movie Gen 峰值 6144×H100
- 共性经验：captioning 是公认瓶颈，放最后并服务化；I/O 而非算力常是实际瓶颈

---

## 11. 2026 年七大趋势

1. **多镜头叙事样本进入训练**：单镜头共识正被打破（Seedance 2.0 切片允许多连贯镜头、Kling 3.0 六镜头 Director Memory、CineDance-1M 平均 24.2 镜头/92.8s）
2. **VLM/LLM 语义质检取代浅层打分器**：终审员（Cosmos/Vidu S1）、判分打标合一（MAGI-1）、大模型互查（MOVA 用 120B 裁决 + 幻觉自审计）；但单一打分器会被钻空子，须多验证器组合（ITS-JAVG）
3. **结构化/时序落地 caption 成竞争焦点**：factorized streams、实体级 anchor token 跨镜头绑定（Script-a-Video、CineDance）；结构化 schema 本身即零成本提升
4. **评测反馈闭环的动态数据调度**取代静态课程表（MAGI-1/Apollo/Seedance）
5. **GRPO 在线 RL + 多维奖励**取代离线 DPO；音视频专用 RM（口型/音色）是待补空白
6. **"训练短、推理长"**：≤20s 训练 + chunk 拼接生成 5 分钟（StreamChar、LongCat 续写）
7. **小数据高效路线抬头**：强基座 + 窄域精数据对抗大数据（CCL 4M vs Ovi 30.7M、Motif-Video 2B、UniAVGen 1.3M）

---

*本文档由 wiki 的 7 个横向对比页综合提炼。各数字与出处详见对应对比页：[数据规模与分布](topics/data-scale-distribution.md) · [清洗流程](topics/cleaning-pipeline.md) · [打标方式](topics/captioning-annotation.md) · [音视频对齐](topics/av-alignment.md) · [训练配合](topics/training-strategy.md) · [效果对比](topics/data-ablation-evidence.md)*
