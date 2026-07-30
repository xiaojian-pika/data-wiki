# 音视频联合生成模型：数据与训练方案（综合最佳实践）

> 基于本 wiki 41 个条目的横向对比提炼，优先采信两类来源：**2025H2–2026 最新工作**（Seedance 1.5pro/2.0、Kling 3.0 Omni、Sora 2、Veo 3.1、LTX-2、MOVA、Apollo、Vidu S1、CineDance、Wan 2.5+ 等）与**披露最详尽/有定量消融证据的工作**（Movie Gen、Allegro、Goku、HunyuanVideo、Open-Sora 2.0、Cosmos-Predict2.5、Koala-36M、MiraData、Data-Juicer、Wan 2.1 等）。另有四个横向专题条目提供跨工作归纳：**评测基准合集**（VABench/AVBench/AV-SyncBench/PhyAVBench/Omni-Judge）、**后训练数据专题**、**打标器生态专题**、**预训练数据集谱系**（Panda-70M→InternVid→Koala-36M→LVD-2M→OpenVid-1M→MiraData→UltraVideo）与**几何/结构化标注数据集**（SceneScribe-1M/SpatialVID/WildWorld/Action100M）。每条实践括号内标注来源。
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
| SFT | 人工精选 + caption 重打标，占预训练量 0.4–20% | HunyuanVideo 100 万条；NAVA 1.07%；CogVideoX 20% |
| 偏好对齐 | GRPO 在线 RL + 多维奖励（含音视频专用 RM） | 偏好对量级 10⁴–10⁵（SkyReels-V2 3 万、JavisDiT++ 2.5 万） |
| 基建 | NeMo Curator（GPU）或 Data-Juicer（CPU）+ 分桶落盘 | 2000×H100 一天处理 ~100 万小时 720p |

核心信念（有强证据支撑，见 §9）：**数据处理质量本身是最大杠杆**——同源素材、同架构下，仅改进处理即可 VBench +9.67 点（Koala-36M）；**优质数据重复 6 遍胜过次优数据配 8 倍算力**（Data-Juicer）。补充推论：**视频质量与 caption 质量是两条正交的提质路径**——只换 caption 大幅提升动态性/时序一致性/文本对齐而完全不提升画质（MiraData），因此二者应各自独立投入而非互相替代。

---

## 1. 数据规模与采集目标

**预训练（视觉主干）**：原始池千万小时级，经漏斗保留 1–5% 得数亿 clip（HunyuanVideo 1.5：>1000 万小时 → 8 亿 clip；Cosmos-Predict2.5：3500 万小时 → 60 亿候选 → 2 亿 clip，保留率约 4%）。

**音视频配对语料**：可低一档，万小时/千万对级即可起步（MOVA 三阶段 61.5k→37.6k→11k 小时；ALIVE 联合训练 11M 对；Movie Gen 音频预训练百万小时级）。

**音频塔单独预训练**：数十万小时纯音频，语音为主（ALIVE 70 万小时转写语音；Ovi、SkyReels-V4 数十万小时；Unison 13 万小时）。

**SFT**：百万级人工精选（HunyuanVideo ~1M；SkyReels-V4 5M→1M 两段）。跨 15+ 工作统计的精选比例区间为**预训练量的 0.4%–20%**（Allegro 0.4%、NAVA 1.07%、Cosmos ~10%、Goku 12.5%、CogVideoX/SkyReels-V4 二段 20%），绝对量级 10⁶–10⁷ 条；偏好对齐数据再小 2–3 个数量级，为 10⁴–10⁵ 对（后训练数据专题）。

**人物/对话专项语料**：双人交互数据集可做到万小时级——SpeakerVid-5M 从 153K 源视频/64,386 小时得到 5.2M clips/8,743 小时（端到端时长保留率约 13.6%），再切出 571K clips/1,368 小时作 SFT（相对原始素材约 2.1%）。注意其损耗主体不是质量过滤而是**结构性损耗**（只留两个主说话人、只留明确说话/倾听时段、单人裁剪损失画面），因此保留率不可与 MOVA 26.39%/Apollo 27% 直接横比（SpeakerVid-5M）。

**分阶段金字塔**：数据量随分辨率/质量逐级收缩（HunyuanVideo 1.5：480p 2 亿→720p 1 亿→CT 100 万；Goku：480p 36M→720p 24M→1080p 7M）。

---

## 2. 时长切分与分辨率策略

**切分**：TransNetV2 场景切分（NeMo 实测 F1=0.967，优于 PySceneDetect/AutoShot）或双工具互补（LongCat、SkyReels），切后用 embedding 相似度 stitching 缝合防过切（NeMo、Motif、Panda-70M），首尾丢 3–10 帧去转场残留（Step-Video、Allegro）。轻量替代方案两条可直接照搬：**Koala-36M 的 Color-Struct SVM**（开源且给出拟合系数 `4.61480465×bgr_sim + 3.75211168×canny_sim − 5.485968377` + 3σ 时序门限，准确率/召回 0.7741/0.9395 远超 PySceneDetect 的 0.4574/0.4146，且 1080p 以上更快）；**LVD-2M 的低采样率技巧**（ContentDetector 阈值 50 + 0.5fps 采样，用"2 秒内任何显著变化即判切换"零成本解决淡入淡出漏检）。追求长镜头时用"过切—缝合"范式：阈值故意调低过度切分，再用 VLM 与 ImageBind+DINOv2 的两两成对组内一致投票缝回（MiraData，是拿到 72 秒长镜头的唯一可行路径）。

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
7. **光流双边运动过滤**：剔静止也剔抖动（RAFT/UniMatch；Goku 480p 档 0.3–20；Movie Gen 用"每秒镜头数≥0.85"作抖动代理）。更成熟的形态是**六档分级 + 差异化采样率而非二元阈值**：最优运动/中等运动保留，静态视频（访谈类）与相机主导运动（航拍）**降采样而非删除**以保住概念，只有低质运动（主体过多/严重遮挡）与抖动镜头被真正排除（Wan 2.1）
   - 绝对阈值参考（唯一给全数值可直接照搬的一套）：PaddleOCR 文字面积 >2%、黑边区域均值 <3、过曝像素 >12%、RGB 方差 <1.2，统一"坏帧率 >5% 丢弃整条"（UltraVideo）
8. **音频质量过滤**（见 §4）
9. **AV 对齐过滤**（见 §4)
10. **NSFW/安全分类器**（Seedance 专用分类器；轻量方案 LAION CLIP NSFW）
11. **SSCD 语义去重**：embedding 聚类→簇内比对→保留最高分辨率版本（Cosmos/NeMo/Seedance 1.0；NeMo 实测去掉约 30% 数据）
12. **VLM 终审**（Qwen2.5-VL 链末复筛：Cosmos；与打标共用前向：MAGI-1）
13. **MLLM 密集打标 + 独立通道反向校验**（OmniHuman 用跟踪/ASR 确定性输出反查 MLLM 幻觉）
14. **概念均衡重采样 + 分桶 sharding**（见 §6）

**参考实现·Wan 2.1 四步漏斗**（开源侧披露最完整的纯视频清洗蓝本之一）：第一步"基础维度"由 9 项轻量检测并联（OCR 覆盖率、LAION-5B 美学、NSFW、水印/logo、黑边、过曝、合成图、模糊、时长与分辨率门槛），**单这一级即淘汰约 50%**；第二步"视觉质量"拆为"聚类 + 打分"——先分 **100 簇按簇配额采样**再用人工 1–5 分标注训练的专家模型全量打分，配额的明确目的是防止长尾中"小而重要"的数据被全局阈值整体抹除；第三步"运动质量"六档分级差异化处置；第四步后训练精选（图像取专家模型 top 20%，视频按类别均衡取 12 大类）。最后全量重打标（Wan 2.1）。其**杠杆式标注范式**在质量打分、相机运动、伪影检测三处反复复用："人工标少量种子 → 训专家小模型 → 全量自动扩标"。

**保留率锚点**：AV 联合漏斗端到端约 1/4（Apollo 27%、MOVA 26.39%）；纯视频严格档 1–5%（Cosmos 4%、Movie Gen 高清档 <1%，其中分辨率、宽高比、OCR 三刀各砍 72–75%）；SFT 精品率再低两个数量级（NAVA 15M→160K 约 1%）。开源数据集侧给出的分档经验值更清晰：不做质量过滤约 100%（Panda-70M，实为 380 万源视频→7080 万 clip 的 18.7 倍扩张）→ 常规质量过滤约 70–75%（Koala-36M 48M→36M、UltraVideo 统计层 74–76%）→ **叠加"长镜头 + 高动态 + 无切换"约束则骤降至约 1%**（LVD-2M 220M→2M 即 0.91%、MiraData 对 HD-VILA-100M 仅 0.2%）——这组约束的代价是 2–3 个数量级的数据损耗，换来的是长镜头纯净率 77.5%（vs Panda-70M 50.0%）与"非常动态"占比 30.0%（vs 7.5%）（预训练数据集谱系）。

**警示·上游数据集自评**：Panda-70M 自家追加的 desirability 标注显示全量中仍有约 19.5% 不理想（低期望分 5.28%、静止前景 6.82%、画中画 5.03%、屏幕录制 1.13%），LVD-2M 人评则显示其 50% 片段含镜头切换、25% 被判"不动态"——**直接使用公开数据集不等于免去清洗**，这正是 VidGen-1M/OpenVid-1M/Koala-36M 三个后续工作均以"重做 Panda-70M"立项的原因（预训练数据集谱系）。

**工程哲学分歧**（按需选择）：多阈值串联（主流）vs 单一学习式综合分（Koala-36M VTSS，论据是子指标不正交）；硬过滤 vs 全量打分存元数据、下游按需收紧（CineDance——推荐大池采用，灵活支持课程分层）；低质数据滤掉 vs 损失隔离（UniVerse-1 LQLS：低质数据仅在扩散 t>800 施加损失，去掉后 ID 一致性 0.89→0.78——"限制作用范围"优于直接丢弃）。

---

## 4. 音频过滤与音视频对齐

**推荐音频侧过滤链**：

1. 无音轨/解码失败剔除 → 2. 格式归一（44.1kHz）→ 3. 音量≥−60dB（Ovi/SkyReels-V4）+ 静音占比分档（**语音数据 <0.2 / 音效数据 <0.8**——HunyuanVideo-Foley 刻意宽松以保住"寂静+一声关门"型 Foley 样本）→ 4. 带宽验真：频谱检测有效采样率 >32kHz，剔除上采样伪高清（HunyuanVideo-Foley）→ 5. 感知质量：**Audiobox-Aesthetics 已取代 SNR 成为事实标准**（MOVA：PQ>5.0 / CU>4.5 / CE>2.5）→ 6. 音频四分类（语音/歌唱/音效/音乐，SkyReels-V4 用 Qwen3-Omni）→ 7. 分流对齐（下述）→ 8. 全量分数存元数据（CineDance）。

**语音子集专用的 ASR 三项诊断阈值**（可直接照搬）：Whisper 转写置信度 <−1.5 剔除（HQ 子集要求 >−1）、no-speech probability >0.8 剔除、compression ratio >2.5 剔除（用于识别重复退化/幻觉输出），并额外检测 language mismatch（SpeakerVid-5M）。这套阈值把 ASR 从"只产文本"扩展为"兼作音频质量判别器"，成本几乎为零。

**数据清洗的质控流程模板**（可复用于任何需要人工复核的环节）：**"商业闭源多模态大模型高召回粗筛 + N=5 标注池、每样本 ≥3 人独立交叉复核"**——AV-SyncBench 用 Gemini 3 Flash 作自动过滤第一关剔除"画外声源"与"明显视听错配"两类样本，随后 5 名标注员每条至少 3 人独立审核主声源画面可见性。用 Flash 级轻量模型做召回、用人力做精度的成本-效果权衡，是目前公开文献中少见的把闭源大模型直接嵌入 pipeline 首级的做法（AV-SyncBench）。

**时序 vs 语义分拆是强共识**（2025H2 起）：Synchformer/AV-align 管时序、ImageBind 管语义，盲区互补。组合逻辑：

- **通用音景数据用 OR 门**（MOVA：ImageBind≥0.2 OR DeSync≤0.5）——环境音无尖锐 onset 会被时序检测误杀，快动作音效会被语义检测误杀，OR 两头保住多样性
- **精标子集用 AND 门**（Foley-Omni：Synchformer≥0.2 ∧ ImageBind≥0.3）——纯度优先

**同步过滤器的物理下限（新证据，与上述阈值建议并存）**：AV-SyncBench 用受控扰动谱（全局偏移 50–500ms 五档、局部抖动 30–700ms 三档、变速 0.8×–1.25× 十档）实测发现，**在 50ms 偏移档位上各 SOTA 同步模型的判别准确率仅约 0.51（接近随机猜测）**。这意味着 Synchformer/SparseSync 类模型作过滤器时**有效分辨率下限约在 50ms 以上**（24fps 下约 1.2 帧），低于此量级的失配无法可靠检出。与前述 Ovi "|offset|≤3 帧 ∧ conf>1.5"的建议并不冲突，二者适用条件不同：**3 帧 ≈ 125ms，位于检测器可靠区间内，故仍是有效的工程阈值**；但若试图把阈值收紧到 1 帧以内（≈42ms）以追求极致纯度，则过滤器输出已是噪声，收紧不再带来实际纯度提升，反而白白损失数据量——**同步阈值的收紧存在由检测器分辨率决定的硬底**（AV-SyncBench）。同一实测还表明单一模型无法兼顾两类过滤：ImageBind 在音色/语义任务上最强（0.859）但时序弱，Synchformer/SparseSync 擅长时序偏移，CAV-MAE 擅长局部抖动与变速——这为 §4 开头的"时序/语义分拆"提供了模型能力层面的直接证据，实践上应**组合使用而非择一**。

**唇同步支路**（语音子集）：先 ASD 剔画外音（Unison lip-filtering、Vidu S1）→ 人脸框内跑 SyncNet（避免全帧稀释信噪比，Unison）→ **|offset|≤3 帧 ∧ conf>1.5**（Ovi 提出，SkyReels-V4、OmniCustom 原样继承，已成社区事实标准）；高质量子集加严至 LSE-D≤9.5 ∧ LSE-C≥4.5（MOVA 以此筛出 2.5M 唇音子集）。Ovi 实验结论：极少量不同步数据即损害唇同步能力，**宁严勿宽**。多人场景采用全员通过制 + SyncNet 做"谁在说话"归属匹配（OmniHuman）。

**ASD 作为二合一过滤器（最简洁的工业写法）**：用 Light-ASD 执行两条排除规则——①音频与画面中的活跃说话人不同步；②画面中根本不存在活跃说话人——一次同时完成"时序不同步剔除"与"声源不在画内剔除"两件事；配套前置条件是"只保留全序列中人脸持续一致可见的视频"，理由是唇同步学习的前提是人脸全程可见（Wan2.2-S2V）。

**SyncNet 的进阶用法：从过滤器升格为结构解析器**（SpeakerVid-5M）。在双人交互数据上，同步检测承担三项超出质量过滤的职能：①**音画身份绑定**——3D-Speaker 声纹 diarization 挑出两个主说话人（合计占总说话时长 80%+），再用 SyncNet confidence 把音频侧说话人 ID 绑定到 YOLO 检出的人体 bbox，回答"这段声音属于画面里哪个人"；②**说话/倾听状态判定**——同框场景下若仅一人在说且两人 SyncNet 分差超阈值，则低分方判为"倾听中"；非同框场景下若 ASR 有效且转写置信度高但 SyncNet 分低，同样判为倾听者（由此派生出 listening 分支，是把"谁在听"显式建模为标签的少见设计）；③**跨片段身份校正**——ArcFace 人脸余弦相似度验证一致性，离群样本若与其他 ID 更相似则重新分配。三步串联回答"谁在说、谁在听、他在画面哪里、跨片段是不是同一个人"，最终得到 83K（单人分支）/16K（对话分支）个全局可追溯的说话人 ID。同步指标三项输出（offset/confidence/embedding distance）逐 clip 持久化存元数据供下游按需筛选，与 CineDance 的"全量打分存元数据"思路一致。

**警示**：SyncNet 对平坦环境噪声给虚高分，必须先按音频类型分层再分层设阈（Script-a-Video）；过滤器与评测指标同源（SyncNet 过滤 + Sync-C 评测）存在循环论证风险（ITS-JAVG）。

**BGM 处理**：通用音景模型不做分离、保留原生混音（Ovi/MOVA/HunyuanVideo-Foley）；人物中心数据例外——Demucs/Mel-RoFormer 拆人声/背景双轨分别监督（OmniHuman、Unison）。

---

## 5. 打标方案

**模型选型**：AV 打标必须用音频可感知模型，纯视觉 VLM 无法胜任（Ovi/Foley-Omni/OmniHuman 共同强调）。三条路线按预算选：

- **开源分工 + 大 LLM 融合**（MOVA：MiMo-VL-7B 视觉 + Qwen3-Omni 转写/音频 + GPT-OSS-120B 融合裁决——最大模型放融合而非感知环节）
- **闭源打精品、蒸馏到开源打全量**（Script-a-Video：Gemini-2.5-Pro 标 50 万条→微调 Qwen3-Omni，学生错误率已略优于教师；NAVA 双档位）
- **自研微调**（数据量越大越值得：Movie Gen 微调 LLaMa3-Video 8B/70B；SkyReels 72B 教师蒸馏进 Qwen2.5-VL-7B，影视字段准确率 76.3% 反超教师）

captioning 是全 pipeline 速率瓶颈（NeMo Curator），需 FP8 量化 / vLLM 服务化。

**打标器生态与选型规律**（打标器生态专题，跨 30+ 工作归纳）：

- **梯队一 · 专训视频 captioner**（下游复用率最高）：Tarsier2-7B（被 Seedance 1.0 作基座、Goku、LongCat 复用，是生成侧采用最广的单一开源 captioner；天然描述相机运动是其独特优势）、ShareCaptioner-Video（其 **DiffSW 差分滑窗**把"全帧→caption"转成"首帧详述 + 逐步差分描述"，天然携带时序增量且支持任意时长线性扩展，是唯一为可扩展性设计的结构）、CogVLM2-Caption（CogVideoX pipeline 中唯一可复现的一环）、SkyCaptioner-V1（Qwen2.5-VL-7B 基座 + 三子专家融合蒸馏，影视字段 76.3%）、AuroraCap
- **梯队二 · 通用 VLM 直接打标**（成本最低最普遍）：Qwen 系列已是 2025–2026 事实标准（Qwen2-VL/2.5-VL/3-VL）；**选 MoE 稀疏架构（约 3B 激活）在吞吐与质量间取平衡是 2026 新趋势**（Motif 用 Qwen3-VL-30B-A3B、Allegro 用 Aria 25.3B MoE/3.9B 激活可 10 秒标 256 帧）
- **梯队三 · 全模态 captioner**（2025Q4 起爆发，AV 生成刚需）：AVoCaDO（Qwen2.5-Omni-7B + GRPO，reward = 五维 checklist 覆盖 + 对白 F1 + 长度正则，Apache-2.0）、AVSCap-7B（AVSCapBench overall 60.44 追平 Gemini-3-Pro 的 60.97）、video-SALMONN 2/2+（MrDPO 多轮 DPO，caption 错误率相对基线降 28%）、Qwen3-Omni-Captioner / Qwen3.5-Omni（后者显式把"剧本级描述 + 自动切片 + 时间戳打标 + 人物-音频关系描述"定位为视频合成训练数据生成能力）
- **梯队四 · 闭源 API 作教师**：GPT-4V（ShareGPT4Video-40K、Koala-36M 种子、MiraData）、Gemini 2.5/3 系列（Script-a-Video 500K、Harmony 400 万条、Foley-Omni 主标注——选它的理由明确是"原生支持视频+音频双输入，纯视觉 VLM 无法胜任音频存在性判别"）

五条选型规律：①**参数量普遍收敛到 7B**（亿级样本打标必须控单样本成本），把大模型放融合/裁决/教师环节而非感知环节；②**多模型分工 > 单模型通吃**——Panda-70M 实证最有力：单个最好打标器仅覆盖 30.8% 样本，贪心选出的 8 个覆盖 76.8%，31 个全上 84.7%；③**运镜识别一律用专用分类器而非 VLM**（Movie Gen 16 类、SkyCaptioner 6DoF 子专家 89% 准确率、混元/LongCat 独立分类器）；④**蒸馏降本是标准动作**（闭源教师 → 7B 学生，几乎每个工作都做）；⑤**2026 年的分水岭是"能不能听"**，omni captioner 已成 AV 生成模型的刚需组件。唯一公开"大小模型混比"数字的是 Movie Gen：训练集 caption 中 70% 来自 8B、30% 来自 70B。

**caption 长度谱系与文本编码器的硬约束**：从 Panda-70M 的 13.2 词到 UltraVideo 的 824.2 词，跨度 60 倍；粗略规律是**数据量每降一个量级，caption 长度与过滤严格度就上一个台阶**（13.2→202→318→824 词）。但长 caption 有两条被实证的教训：①**文本编码器容量必须先行匹配**——MiraData 因 318 词放不进 CLIP 的 77 token 而改用 Flan-T5-XXL（512 token）；LVD-2M 则吃了亏，其 88.7 词 caption 被冻结 CLIP 截断，作者把 I2V 文本匹配提升不明显直接归因于 77 token 上限；②**训练 caption 与用户 prompt 的分布错配会反噬评测**——VBench prompt 平均仅 7.6 词，对长 caption 数据集天然不利。**编码器 token 上限直接截断了 caption 密度的可用上界**，选定 caption 结构实际上同时锁定了编码器选型与推理侧 prompt 改写器（打标器生态专题）。UltraVideo 给出了对抗错配的最简增强：以 1/3 概率从 {Brief, Detailed, Summarized} 中选一个，若选中前两者则再从剩余 7 个结构化类目中随机追加一个作最终 prompt。

**caption 风格约束（被严重低估的工程细节）**：CogVideoX 附录 G 明令禁止 "The video presents/depicts/showcases"、"throughout the video" 等套话与换行符；LTX-2 的原则是 "comprehensive yet factual"，只描述看到和听到的，**显式禁止情绪解读**（防止 caption 向 T2V 训练引入不可控的主观信号）；MiraData 额外约束"不要逐帧描述、不要出现 'first frame' 之类的词"。这三条是全生态最有参考价值的风格设计经验。

**Caption 结构**：长密集（100–250 词）+ 结构化槽位，固定覆盖：主体外观→动作→场景→运镜/景别→光照→风格（LTX-2、Movie Gen ~100 词、HunyuanVideo 七字段 JSON）。风格准则：只述事实、禁套话开头、按时间顺序（LTX-2、CogVideoX）。**多粒度是标配**：短/中/长三档并存 + 训练时随机采样（Motif 0.5/0.3/0.2）或字段 dropout（HunyuanVideo），解决训练长 caption 与用户短 prompt 的分布错配；推理侧配 prompt rewriter（CogVideoX/MAGI-1）。可控标记内嵌：运镜类别（Movie Gen 16 类+6 机位）、质量标签、"This video has no subtitles."（MOVA）。

**AV 联合 caption schema**（2026 核心分歧点，三条路线）：

1. **融合式全音景单段**（LTX-2：对白精确转写+说话人/语言/口音+音乐/环境音全覆盖；MOVA 标注时三轨互斥分流→融合）
2. **内联标签式**（Ovi：`<S>台词<E>` 按时序交织+末尾音景块，消融证明统一编码优于分离编码；ALIVE 台词/音效均内联时间轴）
3. **分字段 factorized**（Foley-Omni 三字段 `[WORDS]/[AUDIO]/[MUSIC]` 可独立验证与置空；Script-a-Video 四流 MTSS——同架构下人评音视对齐 +56%、WER 0.84→0.13）
4. **模板拼接式三段结构 + 分量随机掩码**（Wan 2.1 V2A）：caption = 密集视频描述 + 环境音刻画 + 背景音乐分析（后者按风格/节奏/旋律/配器四属性），由视觉 captioner 与 Qwen2-Audio 分别产出后按固定句式拼接，且**显式支持负向声明**（模板实例："The video description: a horse is running. This audio contains ambient sound: the sound of clip-clop. **This audio does not contain music.**"）——让"音乐是否存在"本身成为可控条件。其最值得借鉴的训练侧配套是 **caption 分量随机掩码**：训练时以预设概率随机省略环境音与音乐两段 caption，强制模型仅凭视觉线索也能建立稳健跨模态关联，同时保留有文本时的可控性
5. **评测基准反向给出的 factorized 佐证**：VABench 把统一视听描述由 LLM 解耦为 visual sub-prompt 与 auditory sub-prompt 两条独立流做分维度评测——**分字段 schema 是"能做分维度验证"的前提**，融合式单段 caption 结构上无法支持这类拆解（av_benchmarks；同理 MiraData 唯有六字段独立可寻址才能做相机/主体/背景/风格四分维图文一致性评测）

**schema 验收标准的形式化**（目前最可操作的一套）：高质量 omni-modal caption 需同时满足三条——Acoustic Completeness（覆盖 Speech/SFX/Music 三类）、Visual Completeness（环境、人物、动作、物体交互、camera motion、OCR）、Audio-Visual Synergy（视听事件的显式绑定与时序对齐）（AVSCap）。AVoCaDO 的 reward checklist 另贡献一个独有维度：**Cross-modal Narrative Logic**（视听互相解释/补充/引导）。

**AV 联合打标的不对称性**：Ovi 的打标输入形态为"7 帧均匀采样关键帧 + 完整音轨"，揭示了一条隐含约束——AV 联合 caption 天然是**"视觉粗、音频细"**的不对称形态，视觉描述偏事件级/语义级而非逐帧密集（Ovi）。

**通用防幻觉设计**（各路线通用）：视觉轨严禁参考音频、音频轨严禁反推画面（MOVA、Vidu S1）；声学能量门控置空字段（Foley-Omni −35dB）；ASR 真值填占位符防台词幻觉（OmniHuman）。

**ASR 与说话人**：Whisper-V3 为主流（UniTalking/SkyReels/UniVerse-1）；说话人属性标准集 7 项：age/gender/accent/pitch/prosody/emotion/speaking rate（Ovi 体系）；"谁在说话"绑定用 ASD（TalkNet）+人脸跟踪+ArcFace（ALIVE、OmniHuman）；Omni 大模型+窗口化在说话人-角色绑定上（95.4%）远超 Pyannote 类专用工具（~63%）（CineDance）。转写保留原语言禁止翻译（MOVA）。

**几何标注**：通用 AV 生成不需要数值相机参数——运镜用语义标签即可（Movie Gen/HunyuanVideo/SkyReels 全走分类标签）。只在数字人/多人对话（人脸轨迹、DWPose、脸-声矩阵）或 Physical AI（深度/3D box）场景加。详见 §5.1。

### 5.1 几何与结构化标注：caption 之外的第二类标注范式

2025H2 起出现一批以**几何真值而非自然语言**为核心标注的数据集（SceneScribe-1M 100 万 clip/4000+ 小时、SpatialVID 271 万 clip/7,089 小时、WildWorld 1.08 亿帧引擎直采、Action100M 1.47 亿时序定位片段），构成与 caption 并列的第二条标注路线。

**标注内容**：相机内外参 + 时序一致稠密深度 + 3D 点轨迹三件套（SceneScribe-1M 用 MegaSaM 做相机与深度、TAPIP3D 做 3D point tracks；论文比较过 DPVO/Fast3r/MonST3R/VGGT 后选定 MegaSaM，理由是特征点稀缺与低相机视差时更稳健）。SpatialVID 额外贡献一条对生成侧最有迁移价值的设计：**把连续相机位姿离散化为受控的电影摄影术语词表**（dolly in / pan left / truck right 等）形成"序列化运动指令"，使几何真值可被文本模型直接消费——这是几何标注与 caption 路线的天然接口。WildWorld 走引擎直采路线拿到逐帧 119 列无噪真值（骨骼关键点、世界状态、相机内外参、无损深度、离散动作标签）。Action100M 则是"结构化动作标注"而非 3D 几何：时序定位边界 + 开放词表动作标签 + actor 识别 + 分层 caption 树。

**最重要的反直觉发现（对通用 pipeline 有直接警示）**：**常规视频生成 pipeline 剔除运动过大的样本，几何标注数据集恰恰剔除"运动不足"的样本**——SpatialVID 实测 **Panda-70M 中超过 80% 的片段因视差/运动不足而无法被 MegaSaM 成功重建**，因此必须主动检索 walk/tour/drone 类高运动素材，最终库中 80% 的 clip 具有弯曲或转向轨迹；SceneScribe-1M 同样以"运动多样性"为初筛主轴，Qwen2.5-VL-72B 判定运动强度未知者直接剔除，并因此大幅缩减源池。**结论：通用数据集的"防抖动/求稳定"过滤取向与几何标注的"要运动"需求方向相反，两类数据不可共用同一条运动过滤链，需在漏斗中分叉**（几何/结构化标注数据集合集）。

**运动过滤的几何化度量**：SpatialVID 用三项相机轨迹度量替代光流幅值——MoveDist（总位移）、RotAngle（累计旋转角）、TrajTurns（方向变化次数），并用基于加速度的检测器识别突兀非物理的抖动。相比标量光流分，这组度量能区分"平移推轨""环绕""手持抖动"三类语义不同的运动，值得在通用 pipeline 中借鉴为运动分级的特征。

**漏斗量级参考**：SpatialVID 21,789 小时原始 → 7,089 小时动态内容（时长保留率约 32.5%）→ HQ 子集再留 13.7%（端到端约 4.5%）。

**适用判断**：对纯 AV 联合生成，这套标注仍属可选项；但若目标含相机可控生成、世界模型/Physical AI、或需要跨镜头 3D 一致性，则几何标注是 caption 无法替代的监督信号——而 §9 所列七个主流预训练数据集（Panda-70M/InternVid/Koala-36M/LVD-2M/OpenVid-1M/MiraData/UltraVideo）**全部无任何三维几何标注**，这是公开数据生态的系统性空白。

**合成数据**：主干必须真实原生音轨；合成只用于填补真实数据不存在的配对——编辑对（InstructAV2AV）、音色解耦对（UniTalking 用 TTS 合成 690 万条防内容泄漏）、TTS 补长尾语种（SkyReels-V4）。

**人工介入**："人不进标注流、人进决策环"：①阈值标定抽检（MOVA、Open-Sora Plan 2000 条验证）②种子标注训打分器（CogVideoX 2 万条、SkyReels 9.3 万条运镜）③抽检（SkyReels：预训练 0.01%、后训练 0.1%）④SFT 末端逐条精选+caption 人工修订（Movie Gen、HunyuanVideo 100 万条人评）⑤偏好标注。

---

## 6. 分布配比与概念均衡

**推荐先落一张三轴 domain 坐标系再谈配比**（评测基准合集的核心产出——五个 AV 评测基准的类目体系可直接反向作为训练数据分布的设计依据，好处是训练 taxonomy 与评测口径天然对齐）：

- **轴一 · 内容 domain（用 VABench 七大类目）**：动物 / 人声（**必须区分语言性与非语言性**——前者需唇同步与 ASR 标注，后者不需要）/ 音乐 / 环境音（自然·城市·室内）/ 同步物理声 / 复杂场景（复杂声景·主观感受·世界知识·符号联想·画外声源）/ 虚拟世界。一并移植的关键设计：**虚拟世界类目应豁免物理合理性质量门槛**，与写实类目分开设定过滤标准（VABench 本身即不对该类目打 Audio/Visual Realism 分）
- **轴二 · 音频类型与同步难度（用 AV-SyncBench 三分法 + 十场景）**：Voice / Music / Sound 下设动作、动物声、物体声、环境声、群体发声、单人说话、对话、演唱、单乐器、合奏。该体系按**同步难度而非内容主题**切分，其"单声源 vs 多声源"的区分（单人说话 vs 对话/群体发声、单乐器 vs 合奏）直接决定多声源混叠样本的配比需求，也决定同步过滤器的分域选型（多声源场景下 Synchformer 类模型可靠性下降）
- **轴三 · 物理声学覆盖 checklist（用 PhyAVBench 六维 41 测试点）**：声源力学 / 流体与空气动力学 / 声传播环境 / 观察者物理 / 时间与因果 / 复杂耦合与极端物理。这是一份可勾选的审计表而非配比表，用于发现训练数据的物理盲区；评测实证已指明**全行业共同盲区是流体动力学与声传播环境两项**，可作为优先补数据的方向
- **正交约束层（用 AVBench 的配额规则）**：Hard Quota-Based Greedy Sampling **强制任一单属性占比 ≤50%**，并按 Normal(1–2 说话人) : Hard(3–4 说话人 + 语音重叠 + 噪声背景) ≈ 3:1 做难度分层。这两条可直接移植为采样时的概念均衡策略与难例配比锚点
- **真实分布校准（用 Omni-Judge 的 VidProM 提示词分布）**：前三轴均为专家设计的理想分类，容易与真实需求脱节；Omni-Judge 的 300 条提示词直接采样自 VidProM 真实用户分布，**用它给前述类目定权重**可避免长尾类目过度投入
- **落地形态**：一张三维配比矩阵（内容 domain × 音频类型 × 物理测试点覆盖），以 VidProM 定权重、以 AVBench 配额规则约束单属性占比、以 PhyAVBench checklist 审计盲区、以 AV-SyncBench 场景标签驱动过滤器分域选型。**已知空白**：多镜头长叙事、多语种口音、跨镜头音轨连续性目前均无基准可对标，这些 domain 的数据投入暂时无法用公开基准验证效果

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
