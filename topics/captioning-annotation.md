# 横向对比：打标方式

> 主题: 视频生成模型（含音视频同时生成）的数据处理：数据清洗流程、数据分布、打标方式

[← 返回首页](../index.md)

本页按字段横向对比所有条目。⚠️ 表示该条目此字段部分信息不确定。

**字段**: [使用的caption模型（自研VLM/开源模型，模型规模）](#使用的caption模型自研vlm开源模型模型规模) · [caption密度与结构化程度（短/长/密集描述、结构化字段如镜头运动、风格标签）](#caption密度与结构化程度短长密集描述结构化字段如镜头运动风格标签) · [音视频联合caption结构（是否同时覆盖视觉+听觉轨道、是否分流为独立字段，如LTX-2全音景描述、Script-a-Video factorized streams、Foley-Omni三字段）](#音视频联合caption结构是否同时覆盖视觉听觉轨道是否分流为独立字段如ltx-2全音景描述script-a-video-factorized-streamsfoley-omni三字段) · [对白转写与说话人属性标注（ASR转写+说话人身份/语言/口音/情绪）](#对白转写与说话人属性标注asr转写说话人身份语言口音情绪) · [几何与结构化标注（相机参数、深度、3D point tracks、动作标注、显式状态）](#几何与结构化标注相机参数深度3d-point-tracks动作标注显式状态) · [合成数据构造（受控扰动/编辑构造训练对，如InstructAV2AV）](#合成数据构造受控扰动编辑构造训练对如instructav2av) · [人工介入程度（人工标注、人工质检、模型初筛+人工复核）](#人工介入程度人工标注人工质检模型初筛人工复核)

## 使用的caption模型（自研VLM/开源模型，模型规模）

`caption_model` · 详细程度: detailed

### [Allegro](../models/Allegro.md)

两级打标，分工明确：
【粗粒度】Tag2Text（Huang et al., 2023）：作为过滤漏斗的第 6 级，对视频中间帧生成标签式描述，提供初步语义信息，其输出直接用作第 7 级 CLIP 相似度过滤的文本侧输入，同时也是数据类目分布统计（people / objects / landscapes）的依据。选它的原因是轻量、可在 500M 量级片段上全量跑。
【细粒度】Aria（Li et al., 2024b，同为 Rhymes AI 自研）在视频 captioning 任务上微调后的版本：Aria 是开源的多模态原生 Mixture-of-Experts 模型，总参 25.3B、每视觉 token 激活 3.9B（每文本 token 激活 3.5B），支持最长 64K 多模态输入，官方称可在 10 秒内为 256 帧视频生成 caption——这一吞吐特性正是其被选为大规模视频打标器的关键。基座模型已按 Apache 2.0 开源（rhymes-ai/Aria），但视频 caption 微调版权重未单独发布。
【文本编码器】训练时使用 T5（非 mT5），最大 512 token；论文通过定性对比（Fig.7–8）论证 T5 在该任务上优于 mT5。

### [Apollo](../models/Apollo.md)

采用「按子集分流 + 多模型并用 + 统一融合」的标注模型矩阵，全部为开源或商用 API 模型，视频侧为未具名自研模型：
【语音/歌唱转写（三模型并用）】Whisper-Large-v3（OpenAI，多语种 ASR，~1.55B）、SenseVoice（阿里 FunAudioLLM，多语种语音理解，支持中英粤日韩及情感/事件识别，~234M）、Qwen2.5-Omni（阿里全模态大模型，7B/3B 版本）。三者并用的组合在同类工作中少见，推测用于交叉验证或按语种/场景分派。
【音频 caption（双模型并用）】Qwen2.5-Omni（开源全模态）+ Gemini 2.5-Pro（Google 闭源旗舰多模态）。在 8100 万条量级上调用 Gemini 2.5-Pro 是显著的成本投入，也说明部分高价值子集走了更强模型。
【视频 caption】「a video expert model for detailed video labels」（视频专家模型，产出详细视频标签）——未具名、未给规模，推测为快手内部自研视频理解模型（与可灵体系配套）。
【说话人属性抽取】性别、年龄等属性由上述音频模型体系抽取，未单列专用模型。
【融合】「All annotations are merged into unified dense captions」，融合执行者未说明是规则拼接还是 LLM 改写——这是与 MOVA（明确用 GPT-OSS-120B 做融合与一致性校验）的显著差异点。
【下游文本编码】caption 送入 Qwen2.5-7B 作为文本编码器；另有 1024 维的 TTS 专用文本编码器处理待合成的语音文本；v1 中另提及 Qwen3-8B Embedding。

### [CineDance / CineDance-1M](../models/CineDance.md) ⚠️

全链路使用 Qwen 系列开源大模型，无自研 caption 模型，推理框架致谢 vLLM：
【叙事解析骨干】Qwen3.5-27B —— 承担镜头分组与叙事边界判定，Tab.4 消融中对比了不同 Qwen 规模的骨干，27B 版本配合自底向上策略取得最优 F1 = 88.4%。
【视觉标注模型】Qwen3.5-35B-A3B —— MoE 架构（约 35B 总参数、3B 激活参数），产出镜头级五维属性、转场类型、局部角色表、活跃场景、镜头描述与转场描述。
【音频标注模型】Qwen3-Omni-30B-A3B —— 全模态 MoE 模型（约 30B 总参 / 3B 激活），同时承担句级 ASR 抽取、镜头级音频 prompt（音乐/环境音/音效）生成、角色音色描述三项子任务，并完成 ASR 句子到角色 anchor token 的绑定。
【选型逻辑】视觉与音频用不同专精模型分工，且均选 MoE 稀疏架构（3B 激活），在百万级序列的标注吞吐与质量之间取平衡；ASR 与音频描述不外挂 Whisper 等专用模型，而统一交给 Omni 模型，理由是 Tab.5/7 显示其在说话人归属任务上远超 Pyannote-3.1（62.7%）与 DiariZen（63.1%）等专用 diarization 工具。
【评测侧另用】CineBench 的 WER/CER 计算使用 Whisper-large-v3。
【未披露】标注 prompt 原文、单模型推理成本、是否对 Qwen 做过任务微调。[不确定]

### [CogVideoX](../models/CogVideoX.md)

分两条路线，构成一个「teacher 生成 → student 蒸馏」的打标体系：
【离线 teacher pipeline（Dense Video Caption Data Generation，图 7）】
1) Panda-70M 的视频 caption 模型 → 先生成短 caption；
2) CogVLM（即 CogView3 中使用的图像 recaption 模型）→ 每隔 2 秒抽 1 帧，为每帧生成稠密图像 caption；
3) GPT-4 → 把带时间戳的 image_captions 字典摘要成最终的视频 caption（附录 G 给出完整 prompt，要求按时间顺序描述内容与变化，覆盖物体、景物、动物、人物与镜头运动，并明令禁止使用「The video presents / depicts / showcases」「throughout the video」等套话与换行符）；
4) 为加速，收集 5 万条 GPT-4 摘要数据微调一个 LLaMA2 作为摘要模型替代 GPT-4，实现大规模生产。
【在线 student 模型】CogVLM2-Caption：以 CogVLM2-Video + Llama3 为底座，用上述 pipeline 产出的稠密 caption 数据微调而成的端到端视频理解/打标模型，规模约 12B 级（Llama3-8B 系底座 + 视觉侧），用于进一步加速全量 recaption。该模型已开源（huggingface.co/zai-org/cogvlm2-llama3-caption），是本工作数据 pipeline 中唯一可被外部直接复用的组件。论文还发现把 CogVLM2-Caption 与 CogVideoX 串联即可实现 video-to-video 生成（附录 I），侧面证明其 caption 几乎捕捉了原视频的全部细节。
【推理侧】另有 caption upsampler：用微调 LLM（文生视频）或 GPT-4V / CogVLM（图生视频）把用户短 prompt 改写成训练 caption 风格的长描述，论文明确说明微调 LLM 优于零样本/少样本。

### [Cosmos-Predict2.5](../models/Cosmos-Predict25.md) ⚠️

采用「通用域用中等规模 VLM、专项域用更大 VLM」的分层策略：
(1) 通用预训练 caption：Qwen2.5-VL-7B（论文明确点名型号与参数量，引 Bai et al., 2025），对每个 5 秒窗口生成 caption，prompt 被工程化引导以产出「factual, context-aware captions」（事实性、上下文感知的描述）。
(2) 领域专项 caption：明确改用「a larger VLM model (Bai et al., 2025)」——同为 Qwen2.5-VL 系但更大档位（未指明是 32B 还是 72B），并为每个域配定制 prompt。团队的判断是：领域数据量小得多但对描述精度要求更高，值得上更大模型。
(3) 多视图驾驶模型的 caption：论文写为「Qwen2.5-7B-Instruct」，每 150 帧生成一次、三档长度（原文如此，与通用侧的 VL 版本表述不同，疑为笔误或指用 LLM 对视觉结果做二次改写）。
(4) 域分类模型：内容类型分类器为内部训练模型；后训练域划分用 InternVideo2 embedding 上训练的多头分类器。
(5) 文本编码器（非 caption 模型但相关）：用 Cosmos-Reason1 替换 Cosmos-Predict1 的 T5，是 Physical AI 专用的 decoder-only VLM；且不取单层输出，而是拼接多个 block 的激活再投影到 1024 维，以同时捕捉局部与全局语言上下文。
[不确定：领域 VLM 的确切参数档位；多视图 caption 模型表述的歧义]

### [Data-Juicer 2.0](../models/Data-Juicer.md) ⚠️

Data-Juicer 不自研 caption 模型，而是把多种 caption 模型封装为可插拔算子，构成本调研中视频 captioning 算子最丰富的开源框架之一。共6个视频 captioning 相关 Mapper：
  · video_captioning_from_vlm_mapper —— 「使用可接收视频输入的 VLM 生成视频描述」。最通用的接口，可接入任意视频 VLM（Qwen2.5-VL、InternVL、Video-LLaVA 等），是2026年主流做法的对应算子。模型规模由使用者选择。
  · video_captioning_from_video_mapper —— 使用 HuggingFace video-to-text 模型生成描述（如 VideoBLIP 系）。
  · video_captioning_from_frames_mapper —— 抽帧后用 image-to-text 模型（BLIP/BLIP-2 系）逐帧生成描述再聚合。成本最低但缺乏时序理解。
  · video_captioning_from_audio_mapper —— 「基于 Qwen-Audio 根据音轨为视频生成描述」。听觉侧 caption，是构建音视频联合标注的关键零件。
  · video_captioning_from_summarizer_mapper —— 「通过对多种已生成文本做摘要来生成视频描述」。这是 DJ 最有设计感的 captioning 算子：把前述各路（帧描述、音频描述、OCR 文字、内容标签、ASR 转写）的产出汇总后交给 LLM 做摘要融合，产出一条综合性的密集描述。这条「多源分述 → LLM 融合」的路径与 Movie Gen、Panda-70M 等团队的多模型融合 captioning 思路一致。
  · video_captioning_from_human_tracks_mapper 与 video_captioning_face_attribute_emotion_mapper —— 人物中心的定向描述（v1.5.4 新增），前者基于人物轨迹生成描述，后者为每个被追踪的人生成面部属性与情绪描述。
【配套的标注型算子】video_tagging_from_frames_mapper（视觉标签）、video_tagging_from_audio_mapper（音频标签，AST）、video_audio_ASR_mapper（语音识别）——这些不产出自然语言描述但产出结构化标签，是 summarizer 算子的输入源。
【实际使用情况】[不确定] 官方 T2V 案例未启用任何 captioning 算子——三个源数据集自带 caption（InternVid 用 BLIP-2 + Tag2Text 生成、Panda-70M 用多模型 caption 融合 + 检索式挑选、MSR-VTT 为人工标注），案例只做了 CLIP 对齐过滤而未重新打标。因此 DJ 的 captioning 算子链虽然完备，但缺乏在大规模视频生成语料上的公开效果验证与推荐配置。DJ 官方也未发布视频 captioning 的推荐模型选型与 prompt 模板。

### [Foley-Omni](../models/Foley-Omni.md) ⚠️

【主标注模型】Gemini 2.5 Pro（Google 闭源商用多模态大模型）。选型特点是原生支持视频+音频双模态输入——这对本任务是硬需求，因为标注必须同时依据「看到什么」和「听到什么」，纯视觉 VLM（如 Qwen2.5-VL、InternVL）无法胜任音频存在性判别。使用闭源 API 意味着标注成本较高且不可完全复现，但论文未披露调用量与成本。参数规模不公开。
【辅助模型 —— 声学验证】Bandit（Watcharasupat et al., 2024），一个面向 cinematic audio source separation（电影音频源分离）的模型，天然按 speech / effects / music 三路分离，与本文三字段schema完全同构——这是一个非常契合的选型，若用通用音乐源分离模型（如 Demucs 的 vocals/drums/bass/other）则无法直接对应三字段。
【文本编码器】UM-T5 encoder（多语言版 T5），用于把三字段结构化文本编码到共享语义空间，供 DiT 条件注入。所有任务共享同一编码器，是「统一接口」设计的实现基础。
【视觉特征模型】CLIP（场景语义特征）+ Synchformer（时序同步特征），后者同时兼任过滤阶段的同步性打分器——一模型两用，训练与清洗使用同一同步表征，保证了过滤标准与模型学习目标的一致性。
[不确定] 未使用独立 ASR 模型做语音转写（见下条）；未提及任何自研 captioner 或对 Gemini 输出做蒸馏以降低成本。

### [Goku](../models/Goku.md) ⚠️

★采用「双 VLM 互补 + LLM 融合」的三模型协同打标方案，是当时较为先进的组合：
(1) **InternVL2.0** —— 负责关键帧（keyframe）级别的图像 caption，强项在静态画面的细粒度内容、物体、属性、场景描述。
(2) **Tarsier2** —— 负责整段视频（video-wide）级别的 caption。论文特别指出 Tarsier2 的一个内在优势：它天然会描述**相机运动类型**（camera motion types，如 zoom in 推近、pan right 右摇），因此无需额外的相机运动标注模块即可获得镜头语言标签。
(3) **Qwen2** —— 作为 LLM 融合器（merger），将关键帧 caption 与视频 caption 合并为一条统一、无冗余、无矛盾的最终描述。论文原文：「we utilize Qwen2 to merge the keyframe and video captions」。
【设计动机】单一图像 VLM 缺时序与运动信息，单一视频 VLM 在静态细节上颗粒度不足，两路互补后由 LLM 消歧融合，兼得空间细节与时序/镜头语义。
【未披露】各模型的具体参数规模与版本号（InternVL2.0 有 1B~76B 多档、Tarsier2 有 7B 档）、是否做过领域微调、打标算力成本与吞吐、图像数据（T2I 的 160M）是否使用同一套 caption 方案。[部分不确定]

### [HunyuanVideo-Foley](../models/HunyuanVideo-Foley.md) ⚠️

标注环节的模型配置极简，与其他工作动辄三四个大模型协同的做法形成鲜明对比：
【唯一的 captioning 模型：GenAU】用于为每个 8 秒片段生成音频 caption，论文描述其提供「对音频内容的简洁描述」（concise descriptions of the audio content）。GenAU（Generative Audio Understanding / 出自「Taming Data and Transformers for Audio Generation」一系工作）是一个专用的音频 captioning 模型，规模远小于通用多模态大模型。论文未披露使用的具体版本与参数规模。[不确定]
【关键特征：只标音频，不标视频】整条流水线中没有任何视频 captioning 环节——不生成画面描述、不描述镜头运动、不做视觉标签。这是由任务形态直接决定的：训练时视频是通过 SigLIP-2 编码的视觉特征直接输入的，不需要经过语言中介；文本条件的作用是补充描述期望的声音特性，因此只需要音频侧的语言描述。这使得本工作的标注成本远低于视频生成类工作（省掉了最贵的视频 captioning 环节）。
【条件编码器（非标注模型，但决定 caption 如何被消费）】
- 文本：CLAP 文本编码器（Elizalde et al. 2023）。选用 CLAP 而非 T5/umT5 是一个有讲究的决定——CLAP 是音频-文本对比学习模型，其文本嵌入空间天然与音频语义对齐，对「描述声音的文本」的表征能力强于通用文本编码器。代价是 CLAP 文本编码器的上下文长度较短（通常 77 token），无法承载长篇密集描述，这直接约束了 caption 的长度上限。
- 视觉：SigLIP-2 视觉编码器，提取帧级语义特征。
- 同步：Synchformer，提取帧级同步特征，走独立的门控调制通路。
【无自研标注模型】全部使用现成开源模型，无任何自研 captioner 或 tagger。
【标注模型总规模远小于同类】对比 MOVA（MiMo-VL-7B + Qwen3-Omni×2 + GPT-OSS-120B）、UniVerse-1（Qwen2.5-Omni + Whisper），本工作仅一个 GenAU，这是 10 万小时规模下对标注成本的必然妥协——规模越大，单样本标注预算越低。

### [HunyuanVideo](../models/HunyuanVideo.md) ⚠️

【原版】自研 VLM（in-house Vision Language Model）作为 caption 模型，用于生成 JSON 结构化 caption；未公布参数规模、架构或训练细节。另有一个独立的自研镜头运动分类器（camera movement classifier）。prompt 侧使用 Hunyuan-Large 大语言模型做用户 prompt 改写。
【1.5】升级为「三模型分工」的打标体系，是相对原版最大的结构性改进：
1. 图像 caption 模型——沿用 HunyuanImage-3.0 的方法；
2. 视频 caption 模型——产出高度结构化的多组件描述；
3. 图生视频指令式 caption 模型（Image-to-Video Instructional Captioning）——这是新增模块，不描述整段画面内容，而是专门描述「相对首帧的时间演化/变化」，涵盖前景主体与背景环境的变化，使 I2V 训练的文本条件从「描述性」转为「指令性」，与 I2V 推理时用户实际输入（给一张图+说想让它怎么动）的分布对齐；
另加一个镜头运动识别模型（clip级 + 时序级双粒度）。
三个 caption 模型的名称、参数量、基座均未披露；只披露了训练手段：用 OPA-DPO（一种针对多模态幻觉的偏好优化方法）做 RL 后训练以抑制幻觉。[不确定]

### [InstructAV2AV](../models/InstructAV2AV.md) ⚠️

本工作不做传统意义的视频 caption（描述视频内容），而是做「编辑指令生成」（描述如何改变视频），因此「caption 模型」在此处对应的是指令生成模型与一系列结构化标注工具。

【指令生成模型】Qwen3-Omni（Xu et al., 2025b，阿里通义开源的全模态大模型）。选型理由是原生支持视频+音频+文本联合输入，能同时理解画面中的可编辑实体与音轨中该实体的发声，从而生成音视频联合的编辑指令。[不确定] 未披露使用的具体规模档位（Qwen3-Omni 有多个尺寸）、未给出 prompt 模板、未说明每个 source 生成多少条指令、未说明指令的多样性如何保证（是否用了模板库、温度采样或类别均衡采样）。

【同一模型兼任裁判】Qwen3-Omni 亦承担阶段三的五维度验证打分（见 model_as_data_judge）。生成与验收同源是本pipeline的一处方法论隐患。

【配套的结构化标注工具链】
  · Grounded-SAM-2（Ren et al., 2024）—— 开放词表检测 + 分割 + 视频跟踪，产出实例级 mask，为指令提供「可编辑对象」的锚点，也为 mask-guided 合成提供编辑区域。
  · TalkNet（Tao et al., 2021）—— 主动说话人检测，标注哪些人在说话及其时空位置。
  · ElevenLabs Scribe（2025）—— ASR，提取精确语音时间戳。
  · SAM-Audio（Shi et al., 2025）—— 音频侧的「分割万物」，按语义把目标实体的声音从混合音轨中分离。
  · 语义声音事件标签 —— 为每个非语音片段赋予唯一标签（如 "dog barking"），标注工具未指明。

【选型评价】整条链路全部使用开源或商用现成模型，无自研 captioner，也未对 Qwen3-Omni 做蒸馏或微调。优点是可复现性相对高（除 ElevenLabs 为商用 API 外）、工程成本低；缺点是各环节的能力上界完全受制于所选现成模型，且论文未做任何工具选型的对比实验。

### [2026 其他音视频联合生成](../models/JAVG_2026_misc.md) ⚠️

本批工作在 caption 模型上呈现「专用小模型 → 通用大模型 → 多模型分工」的明显演进：
【ALIVE —— 自训 caption 模型 + 两轮 SFT + 人工修订（自建路线）】
(1) 视觉 caption 模型经过两轮 SFT（two-round supervised fine-tuning），训练数据为 MLLM 生成后再「manually revised caption data」（人工修订的 caption 数据）——即「大模型生成 → 人工修订 → SFT 蒸馏成专用 caption 模型」的经典自建路径，用一次性的人工成本换取大规模推理的低成本。
(2) 第二类人工标注：「decomposing sub-motion units to annotate the start and end timestamps of each complete small action」——把动作分解为 sub-motion units（子动作单元）并人工标注每个完整小动作的起止时间戳。这是极高成本的细粒度时序标注，目的是让 caption 具备精确的时间轴信息（服务于 Narration 字段的时序叙述与 UniTemp-RoPE 的时序对齐）。
(3) 音频信息融入：一个 MLLM 处理视觉描述以「incorporate the relevant audio information」（融入相关音频信息），实现视听合流。caption 主模型的基座名称论文未一致标明[不确定]；管线其他环节出现 Gemini 2.5 与 Qwen3-omni。
【NAVA —— 三模型分工 + 双档位策略（外部大模型路线）】
(1) 全量数据：Qwen3-VL 生成视频 caption、Qwen3-Omni 生成音频 caption，两者「are then fused by either direct concatenation or rewriting by Gemini-3-Flash」——由 Gemini-3-Flash 做直接拼接或改写融合。注意「direct concatenation or rewriting」二选一，说明存在成本分层：部分数据只做拼接（零成本），部分做改写（有成本）。
(2) 高质量子集：升级为 Gemini-3-Pro，「to produce more accurate, structured, and temporally grounded audio-visual captions」（更准确、更结构化、时序落地更强的音视频 caption）。「temporally grounded」（时序落地）是关键词，与 ALIVE 的 sub-motion 时间戳标注异曲同工，说明 2026 年 AV caption 的竞争焦点已在时序精度上。
(3) 这种「Flash 打全量 + Pro 打精品」的双档位策略是本批中最清晰的 caption 成本工程实践。
【OmniCustom】用 GLM-ASR 为每段 5 秒训练音频生成转写；音频 caption 的属性体系明确「Following the OVI model approach」（沿用 Ovi 的做法），强调说话人的 age、gender、accent 及 vocal characteristics（pitch、prosody、emotion、speaking rate）。生成这些属性标注所用的模型未说明[不确定]。视觉 caption 是否自建未说明[不确定]。
【StreamChar】使用 ASR 时间戳（「ground-truth end indices derived from ASR timestamps」）作为 progress-aware pointer 的监督信号，但未点名 ASR 工具[不确定]；视觉 caption 依赖上游数据集自带标注[不确定]。
【CCL / Baton】完全未描述 caption 模型或打标流程[不确定]。Baton 虽以 Qwen3-8B 为 VA-Planner 底座并回归 SigLip2 / WavTokenizer 特征，但这是模型架构而非数据标注环节；其训练数据的 caption 来自 OpenHuman-Vid / AudioCaps / WavCaps 自带标注，互联网自采部分如何打标未说明[不确定]。
【总体规律】数据量越大越倾向自建 caption 模型（ALIVE 11M 样本 → 自训+人工修订），中等规模倾向调用外部大模型（NAVA 15M 但用 Qwen3/Gemini 组合，且分档控成本），小规模则直接复用上游标注（Baton 1.5M、CCL 4M）。

### [音视频联合生成基线合集](../models/JavisDiT_baselines.md) ⚠️

从「无 caption」到「MLLM 全自动标注」的完整演进：
【无 caption 阶段（MM-Diffusion / AV-DiT）】两者均为无条件生成（unconditional generation）——直接从高斯噪声生成音视频对，不接受文本条件，因此训练数据完全不需要 caption。AV-DiT 论文在局限性中明确承认「主要聚焦于无条件的音频与视频生成」。这是它们与后续所有 T2AV 模型最根本的范式差异。MM-Diffusion 的零样本条件生成（V2A/A2V）靠梯度引导实现，同样不涉及文本 caption。
【复用上游 caption（JavisDiT / JavisDiT++）】自身不做视频打标，直接使用 TAVGBench 数据集自带的文本 caption（TAVGBench 的 caption 由其原作者用自动化方法生成）；音频侧 78 万条同样使用 AudioCaps、Clotho、WavCaps 等数据集自带的音频文本描述（其中 AudioCaps 与 Clotho 为人工标注，WavCaps 为自动标注）。data.md 的 CSV schema 中 text 与 audio_text 两列即对应视频与音频的文本描述。JavisDiT 团队唯一自建 caption 的地方是评测集 JavisBench：使用「先进的 Qwen 系列模型」生成 caption 并做 19 类场景归类，模型的具体版本与规模未指明[不确定]。
【MLLM 全自动标注（Harmony，本合集中最先进）】使用 Google Gemini 对全部 400 万条音视频片段做自动标注，一次性产出三类内容：ASR 转写文本、描述性视频 caption、背景音 caption。Gemini 的具体版本（1.5 Pro / 2.0 / 2.5 等）、prompt 模板、输出质量校验协议均未披露[不确定]。选用 Gemini 而非开源 VLM，说明其看重原生的长视频 + 音频联合理解能力——这与 Ovi 使用「音频可感知的 MLLM」是同一思路。
【UniAVGen】未提及任何 caption 模型或文本标注流程[不确定]；其任务形态（音频驱动动画、视频到音频配音、真人视频生成）对文本 caption 的依赖度本身较低。

### [可灵 3.0 / 可灵视频 3.0 Omni](../models/Kling_30_Omni.md) ⚠️

[不确定] 可灵3.0 Omni 使用的caption模型未公开，官方仅称有“视频描述增强”环节并由自研视频理解/多模态模型驱动。同团队公开做法可作参考：Koala-36M 使用 GPT-4V 生成种子caption后微调 LLaVA 作为最终打标模型（高分辨率视觉编码器 + 2×2 空间池化，图像与视频混合训练）；Kling-Foley 使用音频分类模型 + 音频理解大模型 + LLM 融合三段式生成音视频描述。快手另有自研多模态大模型 Kwai Keye-VL（arXiv:2507.01949）具备承担大规模视频打标的能力。[不确定：模型规模、是否已换代为自研VLM]

### [LTX-2](../models/LTX-2.md) ⚠️

使用 Lightricks 自研的内部自动 captioner，未开源、未公布模型规模与基座。
【前代 LTX-Video】使用「internal automatic image and video captioner」对整个训练集做 re-caption（重打标），以保证文本描述准确、相关，改善视觉内容与文本标注的对齐。
【LTX-2 增量】论文第5.1节明确：为 LTX-2 训练「专门开发了一套新的视频 captioning 系统（a new video captioning system）」，能够以详尽细节同时描述片段的视觉轨道与听觉轨道，「捕捉每一个有意义的动作、外观和声音」。该系统被定位为「连接视频、音频与语言三域的综合文本接口，构成 LTX-2 多模态训练语料的描述基础」。
【未披露】captioner 的参数量、基座模型、是否为多模态 LLM、音频理解如何实现（是否内置 ASR + 音频事件检测 + 说话人分离的组合流水线）、打标吞吐与成本。注意 Gemma3-12B 是训练/推理时的文本编码器，不是 captioner。[不确定]

### [LongCat-Video](../models/LongCat-Video.md) ⚠️

采用「主 captioner + 蒸馏增强 + 多个专项判别模型」的组合，而非单一模型：
(1) 主 captioner：在内部合成 pair 数据上微调的 LLaVA-Video（fine-tuned LLaVA-Video model using in-house synthetic pairs），负责生成基础 caption，同时覆盖视觉与时序两个方面；
(2) 时序增强：引入 Tarsier2 的标注参与训练/补强，Tarsier2 是以细粒度时序事件描述见长的视频描述模型，用于弥补静态描述对动作演进刻画不足的问题；
(3) 结构化属性判别：Qwen2.5VL 负责景别、镜头类型、写实度、动画风格、色调；相机运动（pan/tilt/zoom）由单独训练的轻量分类器负责（而非 VLM，推测出于成本与精度考虑）；
(4) 类目归纳：LLM（型号未披露）对 caption 嵌入聚类结果做类别命名。
各模型的参数规模均未披露（LLaVA-Video 常见为 7B/72B 档，Qwen2.5VL 常见 7B/72B 档，报告未指明选用哪档）。
Avatar 1.5 的 captioner 换为 Qwen3-Omni（全模态，可同时看音频与视频）。[不确定：各 caption 模型的具体参数规模]

### [MOVA](../models/MOVA.md)

全部使用开源模型，形成“双模态分工标注 + 大模型融合”的三模型流水线，未自研 captioner：
【视觉标注】MiMo-VL-7B-RL（小米 LLM-Core 团队，7B 视觉语言模型，RL 版本），指令中明确要求聚焦视频场景转场。
【音频标注 — 双模型策略】
- Qwen3-Omni-Instruct：负责语音转写（ASR），处理语音成分。
- Qwen3-Omni-Captioner：负责非语音声音与音乐的描述，处理非语音成分。
  论文强调该双模型分工可“全面覆盖语言内容与声学特征，减少信息损失，捕获多方面音频语义”。
【融合与一致性校验】GPT-OSS-120B（OpenAI 开源 120B 模型），负责把视频 caption 与聚合后的音频标注（含语音与非语音）合并，同时做跨模态一致性检查。
【推理侧的 prompt 增强模型（非训练数据打标）】Qwen3-VL 提取结构化视觉描述，Gemini 2.5 Pro 通过 in-context learning 改写为符合训练数据分布的视频生成 prompt。
【算力】打标使用 NVIDIA GPU 与昇腾 Ascend NPU 混合执行。
【规模对比】视觉标注器仅 7B、音频标注器为 Qwen3-Omni 系列、融合器 120B——把最大的模型放在融合与裁决环节而非感知环节，是成本与效果权衡下的选择。

### [合并条目：① Mochi 1](../models/Mochi_MAGI_Motif.md) ⚠️

① Mochi 1：打标模型完全未披露。已知的只有训练时的文本编码器——单个 T5-XXL（官方强调「a single T5-XXL language model rather than multiple pretrained models」，即刻意不做多编码器融合），文本维度 1536、视觉维度 3072，视觉流参数量约为文本流的 4 倍。是否使用 VLM 重打标、caption 长度与结构一概不明。[不确定]
② MAGI-1：使用「一个 MLLM」为数据打标，但技术报告始终未点名具体模型与规模（既未说是自研还是开源微调）。有一个明确的工程参数：因主流 MLLM 多为图像设计，团队把每个视频抽取一组关键帧构成图像序列送入；经实证分析，「每个视频片段用 4 到 12 帧（依时长而定）在描述准确度与计算效率之间达到最佳权衡」。另有一个规模数字属于推理侧而非打标侧：为轻量化部署，团队把大 MLLM 生成的增强 prompt 蒸馏到约 7B 的小模型，训练语料约 200 万条（并过滤掉目标文本过长的样本以控制输出长度），人评显示蒸馏模型的生成质量与大模型相当而延迟与算力开销大幅下降。[部分不确定：打标 MLLM 型号与规模]
③ Motif-Video 2B：明确点名——所有 caption 与标签均来自 Qwen3-VL-30B-A3B（30B 总参 / 3A B 激活的 MoE 视觉语言模型）。视频输入为从片段中均匀采样的 N 帧，图像则直接输入。另有一个独立的 OCR 专用模型 PaddleOCR-VL（经 vLLM 服务化）用于帧上文字检测。文本编码器方面：第 1–3 阶段用 sentence-level embedding 模型，第 4 阶段起换为 T5Gemma2——论文借鉴 PixArt-α 的「类条件到文本条件」课程思路，假设低维条件空间能在早期加速收敛，待需要细粒度组合控制时再换用更强的编码器。

### [Movie Gen](../models/Movie_Gen.md)

视频侧：使用 LLaMa3-Video（Dubey et al., 2024）作为caption模型，对 8B 与 70B 两个规模的变体分别做视频captioning任务的微调，然后用它们标注全量训练clip。规模配比明确：训练集caption中 70% 来自 8B 模型、30% 来自 70B 模型（在成本与质量间取平衡）。这是原生的视频caption模型（直接吃视频），而非逐帧图像caption再拼接。
辅助模型：16类相机运动分类器（自训），高置信度预测结果作为前缀拼到caption上。
音频侧：使用多个模型协同生成合成caption——音频质量预测模型（输出1~10分）、AED模型（判定voice/singing存在性与music后验概率）、通用音频caption模型（自由形式描述声音）、音乐caption模型（补充mood与genre细节）。论文特别说明音乐caption模型主要在音乐样本上训练、无音乐时容易幻觉，因此同时保留AED的音乐概率与音乐caption两路信号，实测这种组合的可控性最好。
后训练阶段caption由人工在模型输出基础上精修。

### [NVIDIA NeMo Curator](../models/NeMo_Curator.md)

该字段在版本演进上信息最丰富，是选型参考的重点：
【Cosmos WFM 生产版（2025-01）】使用 NVIDIA 内部微调的 VILA 13B 作为 captioner；部署为 FP8 量化的 TensorRT-LLM engine，相较未优化推理获得约 10 倍加速。输入为从 clip 中均匀采样的 8 帧。
【NeMo Curator 25.09 / 26.02】开源版默认 captioning 后端为 Qwen-VL 系列（Qwen2.5-VL），并支持可选的 Qwen-LM 对生成的 caption 做二次改写增强（caption enhancement / rewriting）。
【26.04 新增】Nemotron Nano 12B V2 VLM 作为与 Qwen-VL 并列的 captioning 后端，提供三种精度变体：nemotron / nemotron-bf16（默认 BF16，自动下载）、nemotron-fp8（FP8 量化，显存占用更低）、nemotron-nvfp4（NVFP4 量化感知蒸馏 checkpoint）。
【26.07 扩展】captioning 后端矩阵扩展为 Qwen2.5-VL、Qwen3-VL 与多个 Nemotron 变体（含 Nemotron 3 Nano Omni 全模态模型），各自可选 BF16/FP8/NVFP4；同时移除了视频预处理选项，统一改用 vLLM 或 HuggingFace 的预处理路径。
【工程要点】captioning 被官方明确指认为整条 pipeline 的速率瓶颈（rate-limiting stage）——其吞吐显著低于嵌入生成等其他 stage，因此 Xenna 的自动扩缩容会把最多的 worker 分配给该 stage。这一观察对任何自建视频数据 pipeline 都有直接参考价值。

### [OmniHuman 数据集 + OHBench](../models/OmniHuman.md) ⚠️

全部使用现成模型，无自研 captioner，形成「开源全模态模型作主力 + 闭源顶级模型作语义裁判」的双层结构：
【Qwen3-Omni——标注推理核心】阿里开源的全模态大模型（同时接收视频、音频、文本），是 caption 生成的 inference core，负责第一阶段的层次化特征提取：全局视频上下文 + 细粒度属性集（外观 appearance、动作 motion、表情 expression、语音 speech、音乐 music、音效 sound effects）。选择全模态而非纯视觉 VLM 是本工作的关键决策——因为要标注的内容本身跨视听两个模态（背景声标签、音乐属性、语音情绪都必须听到音频才能标），纯 VLM 无法胜任。使用的参数档位未披露。[不确定]
【Gemini-3 / Gemini-3-pro——语义判定】负责背景合理性与交互评估，是标注中最难的语义/物理层判断，也是 OHBench 主观维度的评分模型。使用闭源 API 模型说明团队认为开源模型在该维度尚不可靠。
【FunASR-Nano——ASR】达摩院 FunASR 系列的轻量档，负责语音转写，产出带说话人 ID 与时间戳的文本。选 Nano 档是百万级规模下的成本考量。
【3D-Speaker——说话人分离】识别音频中的 M 个活跃语音区间及说话人索引。
【SyncNet——音视觉归属】解析哪段语音属于画面中的哪个人。
【ArcFace——身份嵌入】人脸特征提取，用于身份指派与参考人脸选取。
【YOLOv11 + MOTRv2——检测与跟踪】人体实例检测（NMS 精修）与跨帧关联（query propagation）。
【DWPose-L——姿态】134 个全骨架关键点（身体+脸部+足部）+ 专门的手部检测与优化。
【与同类工作的对比】标注模型阵容的规模与专业化程度是本次调研样本中最高的之一：MOVA 用 MiMo-VL-7B + Qwen3-Omni×2 + GPT-OSS-120B（四个大模型但无几何管线）；UniTalking 用 Qwen3-VL + Whisper-V3 + Qwen3-Omni（三个大模型，几何标注为零）；OmniHuman 则是 2 个大模型 + 7 个专用判别/几何模型的组合，其独特之处在于把几何感知（跟踪、姿态、身份）与语义标注（caption）打通并互相校验。
【标注 prompt】全部未公开。[不确定]

### [Open-Sora 系列](../models/Open-Sora.md)

【Open-Sora 1.x】主力为 **PLLaVA 13B**（选 13B 版本以平衡速度与质量，配置 2×2 空间池化 pooling_shape 4-12-12，输入从视频均匀抽取 4 帧）。文档明确解释了为何不用 GPT-4V：「GPT-4V 效果更好，但 20 秒/样本的速度对我们太慢」——这是开源项目在打标成本上的典型权衡记录。另提供 LLaVA v1.6-Mistral-7B 打标脚本与 **LLaMA 3** 用于从 caption 中抽取物体/标签（tag），代码见 tools/caption/。早期版本亦提及 GPT-4V 作为可选高质量通路。
【Open-Sora 2.0】按训练阶段分级用不同模型，形成「打标质量金字塔」与数据金字塔对应：低分辨率（256px）阶段海量数据用开源 **LLaVA-Video** 打标；高分辨率（768px）阶段的精选 5M 数据改用更强的 **Qwen 2.5 Max**（闭源 API 模型）重新打标，理由是「获得更准确、语义对齐更好的 caption」。这种「粗标底层 + 精标顶层」的分级打标是其低成本策略的重要一环。
【Open-Sora Plan v1.3】主力为 **Qwen2-VL-7B-Instruct**；Panda70M 部分直接复用其官方公开 caption；stock footage 部分使用 **ShareGPT4Video** 的标注；VIDAL 部分沿用其原有的 OFA / mPLUG-Owl / ChatGPT 多模型精修 caption。另外单独训练了一个 **prompt refiner**：以 **LLaMA-3.1-8B-Instruct** 为底座做 LoRA 微调（rank 64，batch 32，1 epoch，单张 H100 上 30 分钟训完）。

### [Ovi](../models/Ovi.md) ⚠️

使用「一个 MLLM（多模态大语言模型）」统一承担音视频语料与纯音频语料的打标工作，音视频侧与纯音频侧明确使用同一个模型（「the same MLLM as used in our audio-video data」）。
【模型身份与规模】论文未点名，未说明是商用 API（如 GPT-4o、Gemini）还是内部自研模型，也未给参数规模[不确定]。
【输入形式（关键设计）】不是纯视觉输入，而是「7 帧均匀采样的关键帧 + 完整音轨」同时喂入——即 caption 模型本身必须具备音频理解能力（audio-capable MLLM），这是 AV 联合生成模型打标区别于纯视频模型的核心要求。7 帧的稀疏采样意味着视觉侧描述偏事件级/语义级，而非逐帧密集。
【prompt 工程投入】论文明确「conducted extensive experiments」以确保 caption 覆盖所有相关视觉与听觉事件并遵守时间顺序（respecting chronology），说明 caption 模板经过多轮迭代。
【无独立 ASR】台词转写直接由该 MLLM 从音轨产出，未提及使用 Whisper 等独立 ASR 模型[不确定]。
【推理侧对偶】用户 prompt 需遵循与训练 caption 相同的格式，仓库提供 GPT 生成的示例 prompt CSV（gpt_examples_t2v.csv / gpt_examples_i2v.csv），即推荐用 LLM 按模板扩写 prompt。

### [Script-a-Video](../models/Script-a-Video.md) ⚠️

标注侧涉及两类模型，角色分工明确：
【教师模型 / 数据标注模型】Gemini-2.5-Pro（Google 闭源）。承担 500K 片段的全部 MTSS 标注生成，是整个数据集的唯一标注来源。参数规模未公开（闭源模型）。
【学生模型 / 专用标注模型】Qwen3-Omni-Instruct（阿里通义千问全模态开源模型）。在 500K MTSS 数据上做监督微调（论文原文为 "supervised sine-tuning"，应为 supervised fine-tuning 笔误），得到 Qwen3-Omni-MTSS-FT。论文未披露使用的具体参数档位（Qwen3-Omni 有多个规模变体）。[不确定]
【蒸馏效果的量化】学生模型经 MTSS 数据微调后大幅逼近教师：
- Video-SALMONN-2 总错误率：Qwen3-Omni 原生 0.5853 → 零样本 MTSS 提示 0.5156 → MTSS 微调后 0.3913（教师 Gemini-2.5-Pro 原生为 0.3959，即微调后的 8B/30B 级开源模型在该指标上已略优于 Gemini-2.5-Pro 的原生 caption）；
- UGC-VideoCap 综合分：62.80 → 71.54 → 85.11（教师 93.97）；
- Daily-Omni：0.1806 → 0.4117 → 0.5945（教师 0.6825）；
- WorldSense：0.1569 → 0.3106 → 0.3875（教师 0.4332）。
【对比基线（均为 caption 模型）】AVoCaDO、ASID-Captioner-7B（7B 参数）、Qwen3.5-Omni-Flash（闭源）、Gemini-2.5-Pro（闭源）。
【关键发现：MTSS 是零成本增益】论文强调 MTSS 对所有测试模型都有效，包括未经任何微调的零样本提示场景——即仅把输出格式要求从「写一段话」改为「填 MTSS 四流结构」，Gemini-2.5-Pro 的总错误率就从 0.3959 降到 0.2511、Qwen3.5-Omni-Flash 从 0.5217 降到 0.3655。这说明结构化 schema 本身即为一种有效的提示工程手段，可被任何团队零成本复用。
【生成侧的文本编码器】LTX-2 内置的 Gemma-3 VLM 编码器，负责解析 MTSS 的 JSON 式语法并输出送入视频/音频双分支的语义表示；多模态输入以图文交错格式喂入。

### [Seedance 2.0 与 Seedance 1.5 pro](../models/Seedance_20_Seedance_15_pro.md) ⚠️

Seedance 1.0：caption 模型以 Tarsier2（强视频理解能力的开源模型）为基座，在人工高质量标注数据上训练，训练时冻结视觉编码器（visual encoder frozen）、对语言模型做全参微调（fully fine-tuned），并在中英双语数据上训练以获得双语能力；推理时用同源的 Prompt Engineering（PE）模型把用户 prompt 改写成与训练 caption 在内容和结构上对齐的详细视频描述。Seedance 1.5 pro 称其「先进的字幕系统」为视频与音频两种模态提供丰富的专业级描述，但未指明基座模型与参数规模。Seedance 2.0 未披露。[不确定：1.5/2.0 的 caption 模型身份与规模；推测已切换至自研 Seed-VL 系列]

### [SkyReels 系列](../models/SkyReels.md) ⚠️

【SkyReels-V2 → SkyCaptioner-V1（已开源，本系列最有参考价值的数据侧产出）】
- 基座：Qwen2.5-VL-7B-Instruct（7B，小模型部署以支撑亿级数据的打标吞吐）；
- 构造方式：知识蒸馏式融合——先由通用大 MLLM（Qwen2.5-VL-72B-Instruct）产出通用描述，再由三个「子专家 captioner」补充影视专业维度，二者的结果融合后蒸馏进 7B 统一模型；
- 三个子专家模型：① 镜头 captioner（判别镜头类型/机位角度/机位位置）；② 表情 captioner（面部表情与情绪强度）；③ 相机运动 captioner（6自由度运镜识别，基于分类，训练数据为9.3万条高置信度人工标注 + 1.6万条运动轴均衡合成数据，1.5万条测试集上单类型运动准确率89%）；
- 训练配置：约200万条概念均衡视频（从1000万条精选），全局 batch size 512，32 张 A800；
- 效果：影视专业字段平均准确率 76.3%，其中镜头类型 93.7%、机位角度 89.8%、机位位置 83.1%、相机运动 85.3%，显著超过 Qwen2.5-VL-72B 等更大模型与专用竞品。
【SkyReels-V4】未沿用 SkyCaptioner-V1 的公开说明，改为多模型组合：视觉侧 caption 由（未点名的）VLM 生成短/长/结构化三档描述；音频侧 caption 由 Qwen3-Omni 统一生成；语音与歌唱内容由 Whisper 转写。此外训练时使用一个冻结的 MLLM 作为文本编码器（非 captioner）。V4 未公布视觉 captioner 的型号与规模。[不确定]

### [Sora 2](../models/Sora_2.md) ⚠️

完全未披露。Sora 2 未公布任何caption模型信息（自研VLM名称、规模、是否使用GPT-4o/GPT-5系列打标）。可参考前代：Sora 1 技术博客称训练了一个「highly descriptive captioner model」为训练视频生成高描述性文本caption（沿用 DALL·E 3 的 re-captioning 思路），并在推理时用 GPT 将用户短prompt扩写为长详细caption。Sora 2 极可能延续并升级该方案，但无官方确认，也无模型规模数字。[不确定]

### [Step-Video-T2V](../models/Step-Video-T2V.md) ⚠️

使用自研内部视觉语言模型（in-house Vision Language Model）作为 caption 模型，为每个视频片段生成描述。报告未披露该 VLM 的名称、参数规模、基座架构、训练数据或训练方式，也未说明是否基于阶跃星辰自家的 Step-1V 多模态模型改造。未对 caption 模型做幻觉抑制方面的专门后训练（无 DPO/RLHF 类描述），caption 质量的兜底依赖 SFT 阶段的人工复核（人工会「优化 caption」）与第6阶段的 CLIP Score 对齐过滤。
衍生模型 Step-Video-TI2V 披露了一步增量：对内部视频打标模型做了微调（fine-tuned an in-house video captioning model），专门强化对「物体运动动态」与「镜头运动」的描述能力，给出的示例 caption 形如「a flock of birds flying over a tree at sunset, camera pans left」——即让 caption 显式携带运镜指令，与 I2V 的可控运镜设计配套。[不确定]

### [UniTalking](../models/UniTalking.md) ⚠️

全部使用开源模型（以阿里 Qwen 系列为主），无自研 captioner，四个模型分工明确：
【Qwen3-VL】负责纯视觉的视频 caption 生成，产出详细版与简短版两种粒度。未披露使用的参数档位（Qwen3-VL 有多个规模变体）。[不确定]
【Whisper-V3】OpenAI 多语种 ASR 模型，负责语音转写。用于提供音频的文本内容（台词）。
【Qwen3-Omni-Captioner】Qwen3-Omni 系列中专门用于音频描述的 captioner 变体，负责生成音频 caption（描述声学内容与环境）。
【Qwen3-Omni】全模态大模型，同时接收视频与音频两个流，产出单一的融合式音视听统一描述（第三种 caption 格式）。这是四个模型中唯一做跨模态融合的。规模档位未披露。[不确定]
【IndexTTS2】不属于 captioner 但属于标注/衍生数据环节，为零样本 TTS 模型，负责合成参考音色音频。
【文本编码器】训练与推理时用冻结的 UMT5 编码文本条件，文本 latent 序列长度固定为 512（超长截断、不足补零）。
【设计特点：同一内容用两条路径分别标注】既有「分头标注后拼接」（Qwen3-VL + Qwen3-Omni-Captioner/Whisper-V3）也有「统一模型融合标注」（Qwen3-Omni），两条路径的产物在训练时被随机采样使用——这是一种把「标注策略的选择」交给随机采样、让模型同时适应两种文本分布的做法，见 caption_structure。
【与同类工作的规模对比】标注模型总规模明显小于 MOVA（MiMo-VL-7B + Qwen3-Omni×2 + GPT-OSS-120B），但多于 UniVerse-1（QWen2.5-Omni + Whisper）；且 UniTalking 的标注为离线一次性完成，不像 UniVerse-1 那样在线随训练实时生成。
【标注 prompt】全部未公开，无任何 caption 示例（Figure 4 中展示了 prompt 但仅为推理时的测试 prompt，非训练 caption 原文）。[不确定]

### [UniVerse-1](../models/UniVerse-1.md) ⚠️

全部使用开源模型，无自研 captioner，模型数量与规模均远少于同类工作：
【多模态标注】QWen2.5-Omni（阿里通义千问全模态模型），承担全部三路标注——同时输出核验后的语音内容、视频描述性 caption、环境音 caption。这是单模型多路输出，而非 MOVA 式的三模型分工。论文未披露使用的具体参数规模档位（3B / 7B）。[不确定]
【语音转写】Whisper（OpenAI），执行 ASR。在离线漏斗中用于判定片段是否含语音（分流闸门），在在线标注中用于对采样窗口的音频做转写。论文未披露 Whisper 的具体版本（base/large-v2/large-v3）。[不确定]
【无融合模型】没有 MOVA 式的大参数量 LLM 融合与一致性裁决环节（MOVA 用 GPT-OSS-120B）。三路标注由 QWen2.5-Omni 一次性并列产出，保持独立字段形态。
【文本编码器】训练与推理侧的文本条件编码使用 umT5（两个基座专家共用同一文本编码器族，这也是 SoE 缝合能够成立的前提之一）。
【评测侧的标注模型】Verse-Bench 的 Set2-V 子集使用 LLM 生成 caption + Whisper 做 ASR，再经人工核验。
【规模对比】UniVerse-1 的标注模型总规模（QWen2.5-Omni + Whisper）远小于 MOVA（MiMo-VL-7B + Qwen3-Omni×2 + GPT-OSS-120B），这与其在线标注的架构选择直接相关——在线标注要求标注模型足够轻量才能跟上训练吞吐，无法承受 120B 级模型的逐样本推理开销。这是一个「标注时机」与「标注模型容量」之间的权衡。

### [Unison](../models/Unison.md) ⚠️

论文未披露训练数据的 caption 由何种模型生成——这是打标方式类字段中最核心的空白。
【已确认使用的模型及其用途，须严格区分训练侧与评测侧】
- Gemini（具体版本未披露）：仅用于评测侧。为 1,000 条 held-out 测试样本生成 ground-truth 标注（支撑 T2AV/TI2AV 评测），并在用户研究中作为 LLM 裁判为 Motion-Speech-SFX 连贯性打分。未见用于训练数据标注。[不确定]
- Whisper-large-v3：仅用于评测侧。计算 WER 时，先用 Mel-RoFormer 从生成音频中分离人声，再用 Whisper-large-v3 转写并与目标文本比对。论文未说明训练数据的 transcription 是否也由 Whisper 产生——但模型的文本输入明确包含 transcription text（转写文本）这一独立通道，训练时必须有转写来源，最可能的情况是沿用上游数据集自带的转写或用 Whisper 离线转写，两种可能论文均未确认。[不确定]
- Mel-RoFormer：训练与评测双侧使用。训练侧用于把混合音频解耦为语音与音效两路 ground-truth latent；评测侧用于 WER 计算前的人声分离。这是唯一横跨训练与评测的工具模型。
【推测的训练 caption 来源】Unison 的输入包含 caption text 与 transcription text 两路文本条件，且 SCG 门控依赖二者的平均池化全局语义向量——因此每条训练样本必须同时具备 caption 与 transcription。最可能的来源是上游数据集自带标注：CelebV-Text 本身就是「视频 + 文本描述」配对数据集，OpenHumanVid 亦发布了配套 caption，VGGSound 自带声音事件类别标签，WavCaps 是带 caption 的音频数据集。若如此，则 Unison 未自建 captioner，caption 的密度与风格在五个数据源之间存在显著异质性（CelebV-Text 的人脸属性描述 vs VGGSound 的类别标签 vs WavCaps 的音频 caption），而论文对这一异质性如何统一处理毫无说明。此为本条目推断，无原文支撑。[不确定]
【无自研 captioner】论文未提及任何自研或微调的标注模型。

### [Veo 3 / Veo 3.1](../models/Veo_3_Veo_31.md) ⚠️

使用「多个 Gemini 模型」（multiple Gemini models）对音频与视频片段进行文本 caption 标注，这是官方在技术报告与 Model Card 中重复强调的核心数据处理手段。采用多个模型而非单一模型，对应「不同详细程度」的多粒度标注需求（可推测为大模型产出长密集描述、小模型产出短描述以控制成本）。[不确定] 未披露具体 Gemini 版本（1.5 / 2.0 / 2.5 系列）、参数规模、是否为针对视频标注微调的内部专用变体、标注吞吐与成本。

### [Vidu S1](../models/Vidu_S1.md) ⚠️

使用了一个「标注模型（annotation model）」生成结构化自然语言描述，并在质量过滤阶段使用了 omni model（全模态大模型）。论文均未披露这些模型是自研还是开源模型、参数规模多大 [不确定]。

### [主流视频预训练数据集合并调研：Panda-70M、InternVid、Koala…](../models/pretraining_datasets.md) ⚠️

**打标模型选择直接决定caption长度，从13.2词到824.2词跨越60倍**。
【Panda-70M·多教师+检索式择优，架构最复杂产出却最短】候选池**31个打标器 = 6个基座 × 权重变体 × 输入模态组合**（V=视觉/S=字幕/M=元数据即标题+描述）：Video-LLaMA（pretrain与finetune共8变体，**仅用视觉分支、音频分支明确关闭**，Vicuna-7B）、VideoChat（7B，4个）、VideoChat Text（4个，把视频文本化为标签+密集描述+总体描述再对话，因成本把ChatGPT-4换成LLaMA）、Video-ChatGPT（4个）、BLIP-2（opt2.7b/opt6.7b/flant5xl共3个，图像模型从0.3N–0.7N帧间随机取一帧）、MiniGPT-4（7B/13B共8个）。用1000条clip的用户研究做**贪心集合覆盖**：单个最好模型仅覆盖30.8%，31个全上84.7%，贪心选出的**8个覆盖76.8%**（Video-ChatGPT入池未入选）。视觉-only的prompt为「Please faithfully summarize the video (or image) in one sentence.」。**择优环节**用细粒度视频-文本检索：**UMT large = ViT-L/16 + BERT-large**，VTC+VTM双损失 + **难负例挖掘（7条未选中caption权重1.0，batch内其他负例权重0.01）**，12帧224×224，AdamW lr 2e-5，batch 32，10 epoch，8×A100-80G。效果：微调后UMT **R@1 35.90%** vs 预训练UMT 21.82%，**而人类之间的一致率仅44.9%**（说明「最佳caption」高度主观）。**蒸馏学生模型**：视觉分支沿用Video-LLaMA设计（8帧→冻结ViT-G/14(EVA-CLIP)+Q-Former→时序融合→32×4096），文本分支用**text Q-Former**以视频表示为query产出固定32×4096，拼成64×4096送入**Vicuna-7B-v0**；文本→视觉的梯度被阻断，元数据与字幕各以0.5概率独立丢弃；在全量Panda-70M上训**30万步、48×A100-80G**。动机很实际：完整pipeline每条clip要跑**8+1=9次**模型推理。
【InternVid·多尺度，两尺度用不同模型——最常被误传的一点】**粗尺度用BLIP-2描述中间帧；细尺度用轻量的Tag2Text以低fps逐帧描述，再用一个预训练语言模型合成为整段描述**。**细尺度不是BLIP-2**。该LLM身份论文未明确（引用中同时列T5与Vicuna却未说明实际用哪个）；HF上的`InternVid-18M-aes-vc2vicuna.jsonl`暗示存在VideoChat2+Vicuna的重打标通路，但那是后续重标而非原始摘要器的证据。**摘要prompt原文、Tag2Text的fps、UMT的具体变体均未公开。**[不确定]
【Koala-36M·GPT-4V教师→LLaVA学生蒸馏】先用**GPT-4V**在种子集上按结构化schema产出caption，再微调一个**基于LLaVA**的打标器跑全量（**不是Tarsier**）。微调经验值得复用：(a)训练视觉编码器（不冻结）提升caption准确性；(b)高分辨率视觉编码器有助于捕捉细节；(c)为控制token开销对**视觉token的空间维度做2×2平均池化**；(d)**图像+视频混合训练**，既教会静态/动态理解又缓解视频数据稀缺。
【MiraData·GPT-4V一轮对话产出5字段 + Panda-70M出短caption】6个字段中**5个由GPT-4V在同一轮对话中生成**，1个（short_caption）直接用**Panda-70M的打标模型**。输入形式特别：均匀取**8帧拼成2×4网格的单张图**（按时序从左到右、从上到下，白边分隔），并把Panda-70M短caption作为**hint**注入prompt。仓库里另有`default_prompt_wo_hit`变体完全去掉hint，作者称「在少量样本上测试结果几乎相同」——**这实际上削弱了Panda-70M那一步的必要性**。
【OpenVid-1M】**LLaVA-v1.6-34B**对全部视频重打标，包括为原本无caption的CelebvHQ补标。**prompt原文未公开。**[不确定]
【LVD-2M·分层三段式】(1)超过30秒的视频先切成**30秒段**；(2)每段均匀取**6帧拼成2×3网格单图**（方法源自IG-VLM），送**LLaVA-v1.6-34B**，产出背景、主要人物、主要动作、相机视角，**采样间隔是自适应的（段长/6，≤5秒）**；(3)用**Claude3-Haiku**精修与合并（两条prompt分别负责「精修单条粗caption」与「按时序合成整段描述」），动机是「LLaVA-v1.6-34B倾向于产生额外的推测与假设，导致冗余」；(4)仓库中还有论文未提及的第三条轨道`rewritten_caption`，由**LLaMA-v3.1-70B**改写为「简洁的用户输入风格」。实测100条样本平均词数：raw 215.4 → refined 81.8 → rewritten 41.3，**Claude3-Haiku把LLaVA输出压缩约2.6倍**。
【UltraVideo·全开源模型，规模最大】**Qwen2.5-VL-72B**产出9个维度，再用**Qwen3-4B**整合成第10个Summarized Description。作者明确与MiraData（GPT-4V+2×4网格）、Koala-36M（GPT-4V蒸馏进LLaVA）对比，强调自己用开源模型。**prompt与帧组装方式均未公开**——GitHub issue #3专门追问，被关闭且无维护者回复。[不确定]
【横向规律】2024上半年（Panda-70M、InternVid）走「多个小模型+融合/择优」的工程化路线；2024下半年起（MiraData、Koala-36M、OpenVid、LVD-2M）统一转向「一个强VLM（GPT-4V/LLaVA-34B）直接打标 + LLM精修」；2025年（UltraVideo）进一步转向「大参数开源VLM（72B）+ 小LLM汇总」。**教师模型的融合复杂度在下降，单模型能力在上升。**

## caption密度与结构化程度（短/长/密集描述、结构化字段如镜头运动、风格标签）

`caption_structure` · 详细程度: detailed

### [Allegro](../models/Allegro.md) ⚠️

属于「长密集描述 + 半结构化字段嵌入」形态：
· 覆盖维度（论文原文列举）：主体属性（attributes of the subjects）、主体间交互（interactions among the subjects）、背景描述、环境、风格（style）、氛围（atmosphere）、镜头角度与镜头运动（camera angle and motion）、以及时序变化（temporal changes）——即同时覆盖空间信息与时间信息。
· 组合方式：视频 caption = 中间帧的静态帧 caption + 整段视频的动态 caption 拼接，兼顾单帧细节保真与全片动态叙述。
· 结构化字段：镜头运动被以固定句式显式凸显，形如「Camera [MOTION_PATTERN]」的句子被插入 caption，使模型能学到可控的运镜指令响应；其余维度以自然语言散文形式表达，未拆为独立 JSON 字段。
· 双粒度并存：所有样本同时保有粗粒度（Tag2Text 全局语义标签）与细粒度（Aria 时空细节）两套注释，前者主要用于过滤与统计，后者用于训练。
· caption 长度受 T5 编码器 512 token 上限约束。
· 未披露 caption 长度分布、caption 多样性重写（recaption / prompt upsampling）策略。[部分不确定：长度分布与短/长 caption 混训比例]

### [Apollo](../models/Apollo.md) ⚠️

【总体定位】论文将其数据集定位为「the first large-scale audio–video dataset with dense captions」（首个带密集 caption 的大规模音视频数据集），密集（dense）与精确（accurate）是其自我标榜的两个关键属性，但 caption 的实际形态未给出任何样例。
【结构化程度】论文明确 caption 覆盖两个层次：「including both meta information and detailed content」（同时包含元信息与详细内容）。元信息层包括说话人属性（性别、年龄）等结构化字段；详细内容层为自然语言描述。这是一种「结构化 meta + 自然语言正文」的混合结构。
【分流条件化】caption 内容随音频子集而变：语音/歌唱子集含转写文本 + 说话人属性 + 音频 caption + 视频 caption；sound split 只含音频 caption + 视频 caption，不含转写与说话人属性。即 caption schema 是随数据类别动态裁剪的。
【最终态】「merged into unified dense captions」——所有标注最终融合为统一的密集 caption，不保留分离字段（与 MOVA 的「标注时分流、训练时融合」路径一致）。
【模型侧的条件通道】值得注意的是，模型架构层面输入是四路而非一路：视频、视频相关文本（video-related text）、音频、音频相关文本（audio-related text）——即视频 caption 与音频 caption 在**输入通道层面仍然是分离的两个文本条件**，并非合并为单一 prompt。这与「unified dense captions」的表述存在张力，可能的解释是数据侧生成统一 caption 后在训练时再拆回两路，或两种表述描述的是不同环节。
【关键缺失】caption 长度分布、词数统计、prompt 原文、完整 caption 示例、是否含镜头运动/风格标签等结构化字段，全部未披露。[不确定]

### [CineDance / CineDance-1M](../models/CineDance.md)

标注密度与结构化程度是该数据集相对现有工作的最大代差，论文称之为「可配置的分层双模态标注（Configurable / Hierarchical Dual-Modal Annotation）」：
【密度】平均每条视频 6,496.3 个词的结构化标注（Tab.6），远超 MiraData 等仅有视频级 caption 的数据集，且是同类中唯一提供镜头级密集标注的。
【两层结构】
  · 全局层（Global）：全局角色表 [⟨char₁⟩,…,⟨charₙ⟩] 与全局场景表 [⟨scene₁⟩,…,⟨sceneₘ⟩]，以 anchor token 形式定义；
  · 镜头层（Shot-level）：每个镜头的描述显式引用上述 anchor token，实现跨镜头的身份与场景绑定。
【Anchor Token 机制的价值】这是解决多镜头一致性的关键设计——生成模型可据此知道「第 3 镜头的 ⟨char₂⟩ 与第 17 镜头的 ⟨char₂⟩ 是同一人」，评测侧 CineBench 也直接用该 token 做身份连续性（ArcFace 聚类）与场景连续性（DINOv2 余弦相似度）的判定锚点。
【镜头级结构化字段】五维镜头属性：scale（景别）、angle（机位角度）、movement（运镜）、narrative function（叙事功能）、duration category（时长类别）；外加 shot transition type（转场类型）、localized character list（该镜头出现的角色）、active scene（该镜头所处场景）、shot description（镜头描述）、transition description（转场描述）。
【「可配置」的含义】所有字段与质量分均可组合筛选，用户能按需拼装出 task-specific 子集（例如只要含运镜标注的、只要高美学分的）。
【模型侧使用格式】训练时组织为：全局头部（角色/场景定义）+ 逐镜头块 [SHOT i | scene sᵢ | camera κᵢ] ⊕ 转场描述 div ⊕ 对白 dia ⊕ {(说话人 spkᵢ,ℓ, 台词 speechᵢ,ℓ)}。

### [CogVideoX](../models/CogVideoX.md)

· 密度：明确追求稠密长 caption。论文批评 Panda70M、COCO Caption、WebVid 等现有数据集的 caption「通常非常短、无法全面描述视频」，因此自建 pipeline 产出段落级长描述。附录 H 的对照示例极具代表性：同一视频 Panda-70M 输出「A crab is walking on the beach with a light bulb on its back.」（一句话），CogVLM2-Caption 输出近 80 词的段落，包含主体外观（dark glossy shell、reddish-brown legs）、动作、光照变化（from a soft glow to a more pronounced illumination）、场景（sandy terrain、tranquil sea backdrop）、时间（at night）与氛围评价（serene yet whimsical atmosphere）。
· 结构化程度：属于「非结构化的稠密自然语言段落」，没有 Movie Gen 式的显式字段（无独立的相机运动标签前缀、无 FPS token、无风格标签枚举）。但通过 GPT-4 摘要 prompt 强制约束了内容覆盖面与写作风格，形成事实上的隐式 schema：必须覆盖 objects / scenery / animals / characters / camera movements 五类要素；必须按时间顺序（in chronological order）描述内容及其变化；禁止使用固定套话开头以避免风格坍缩。
· 长度约束：文本编码器（T5）侧的 Text Length 上限为 226 token，是 caption 长度的实际天花板。
· 训练-推理分布对齐：因用户实际输入远短于训练 caption，专门设计 caption upsampler 在推理时把短 prompt 改写为长描述，与 DALL·E 3 的做法一致。

### [Cosmos-Predict2.5](../models/Cosmos-Predict25.md) ⚠️

结构化程度体现在「时间粒度切分 + 三档长度 + 领域定制字段」三个维度：
【时间粒度】每条 clip 切成 5 秒窗口逐窗口 caption，而非整条一句话。论文把这列为相较 Cosmos-Predict1 的核心改进之一：「introduces finer content granularity by segmenting clips into shorter temporal windows」，目的是获得「richer and more precise supervision signals」（更丰富精确的监督信号）。
【三档长度】caption 统一产出 short / medium / long 三种长度（Captions are produced at multiple lengths），既作训练监督信号又作条件 prompt，使模型同时适配简短用户输入与详尽描述。驾驶域同样三档。具体 token 长度区间未给出。
【通用侧内容约束】prompt engineering 引导模型描述 primary object（主体物体）、its motion（其运动）、key semantic details（场景关键语义细节）三要素，并强调 factual（事实性）以抑制幻觉。
【机器人域 caption schema】结构化最强，prompt 强制要求：先枚举 initial scene（初始场景），再按时间顺序描述机器人动作，且必须显式标注 motion type（运动类型，如 linear 线性 / rotational 旋转）、involved parts（涉及部件：arm 手臂 / wrist 腕 / gripper 夹爪）、camera motion（相机运动）、fine-grained object attributes（细粒度物体属性）。同时做 viewpoint 与 embodiment 的归一化表述（统一跨数据集的相机视角命名）。为降低幻觉，把数据集自带元数据注入 prompt：GR00T 注入带人工成功评级的任务描述、Bridge 注入 step-level 指令、AgiBot 注入初始场景描述。这套「元数据注入 prompt 以约束 VLM」的做法是抑制视频 caption 幻觉的有效范式。
【驾驶域 caption schema】六类必写要素：① 自车需注意的各类 agent（车辆/行人/骑行者）与交通元素（红绿灯/交通标志）；② 全局环境因素（天气、时段、路面状况）；③ 自车与周车的纵向与横向 meta action；④ 自车与周车速度；⑤ 其他物体的动态行为或状态转移；⑥ 关键物体之间的交互。
【Smart Spaces / Physics / Human Dynamics】分别定制 prompt：Smart Spaces 指明视频聚焦工厂/仓库/工业设施/汽车等制造环境并相应调整语言风格；Physics 引导 VLM 准确详细描述底层物理过程与物体交互；Human Dynamics 强调人体运动与动力学的细致描述。
未披露 caption 的平均 token 数、三档长度的混用采样比例。[不确定：caption 长度数值与三档配比]

### [Data-Juicer 2.0](../models/Data-Juicer.md) ⚠️

【DJ 的立场：提供组装零件，不规定 caption 范式】DJ 不预设 caption 应该多长、包含哪些字段，而是提供从「短标签」到「长密集描述」的全谱系算子，由使用者按需组合。
【可产出的 caption 密度谱系】
  · 最稀疏：video_tagging_from_frames_mapper / video_tagging_from_audio_mapper 产出的是标签词表（如 AudioSet 527类标签、RAM 开放词表标签），不是句子。
  · 中等：video_captioning_from_frames_mapper / from_video_mapper 产出单句或数句的常规描述。
  · 最密集：video_captioning_from_summarizer_mapper 通过融合多源信息产出综合长描述；video_captioning_from_vlm_mapper 可通过 prompt 控制产出任意长度与结构的描述。
【结构化字段的支持情况】DJ 的数据模型是「样本 = 多模态字段 + 任意数量的 stats/tags 字段」，每个算子把自己的产出写入独立的字段而非合并进 caption 文本。这意味着结构化信息天然是分字段存储的：镜头级的美学分、运动分、OCR 占比、人脸比例、内容标签、说话人年龄性别情绪、相机内参与位姿、深度、全身关键点等，都是并列的结构化字段，可被下游训练框架按需拼接进条件文本或作为独立条件通道。这种「字段化优先」的设计在结构化程度上实际高于多数只产出一段长文本的模型团队 pipeline。
[不确定] DJ 未提供官方推荐的视频生成 caption 模板（如是否应包含镜头运动、光照、风格、主体动作等固定字段），也未在任何公开案例中报告 caption 平均长度、字段构成或不同 caption 结构的效果对比。这属于其「工具中立」定位的必然结果，但也意味着使用者无法从 DJ 获得 caption 设计的最佳实践指引。

### [Foley-Omni](../models/Foley-Omni.md) ⚠️

结构化程度高，但单字段描述密度不高——属于「结构优先于密度」的设计取向，与文生视频模型追求长密集caption的路线明显不同。
【格式】统一模板 `[WORDS] <spoken content>; [AUDIO] <sound event descriptions>; [MUSIC] <music specification>`，三个字段固定顺序、显式标签分隔，字段内容可为空字符串。
【结构化的三层价值】
  1. 可组合的条件接口：单字段填充=单任务（TTS/TTA/TTM），多字段填充=完整配乐（V2ST），一套模板覆盖六类任务，避免了为每个任务设计独立条件通路。
  2. 显式负标注：空字段明确告知模型「此片段无该类音频」，而非让模型从缺失中猜测。这对抑制模型在不该生成音乐时凭空加入配乐（V2A 模型的常见失败模式）有直接作用。
  3. 可验证性：字段化使得每个标注单元可被独立的声学证据（对应 stem 的能量）校验，若是自由形式的整段描述则无法做这种细粒度纠错。
【与同期工作的谱系关系】按论文对比语境，这属于「音视频联合caption分流化」这一2026年趋势的一支：LTX-2 走「全音景统一描述」路线（一段文本描述整个声景），Script-a-Video 走 factorized streams（分解为独立流），Foley-Omni 则是最简洁的固定三字段方案。三字段的优势是schema极简、与源分离模型天然对应、易于验证；代价是表达力受限——无法描述字段内多个声源的相对位置、响度层次、时间先后。
[不确定] 未给出 caption 的平均长度/词数统计，未说明 [WORDS] 字段是逐字转写还是内容概述（从 WER 作为评测指标推断应为逐字转写文本），未说明是否包含镜头运动、视觉风格等视觉侧结构化标签（从模板看不包含——视觉信息完全由 CLIP/Synchformer 特征承载，不进入文本条件）。

### [Goku](../models/Goku.md) ⚠️

属于**密集长描述（dense caption）+ 少量结构化标签追加**的混合形态：
【主体】自然语言密集描述，由 InternVL2.0（关键帧细节：物体、属性、场景、构图）与 Tarsier2（时序动态、动作演进、相机运动如 zoom in / pan right）两路生成后经 Qwen2 融合为单条统一长文本。因此单条 caption 同时覆盖「画面内容 + 时序动作 + 镜头语言」三个层面。
【结构化字段追加】最具特色的一点是：**将 RAFT 计算出的运动分数（motion score）数值直接追加到 caption 末尾**，使运动强度成为一个显式的、可在推理时人为指定的条件变量。这样用户可通过在 prompt 中给定运动分数来控制生成视频的动态幅度，把一个原本只用于过滤的统计量转化为可控生成接口——是「过滤信号复用为条件信号」的典型范例。
【未采用】未见分层多粒度 caption（短/中/长多版本）、未见风格标签/光照/画质等独立结构化字段体系、未见 caption dropout 或短长 caption 混合训练策略的描述。[不确定]（caption 平均长度、是否保留短 caption 版本）

### [HunyuanVideo-Foley](../models/HunyuanVideo-Foley.md) ⚠️

【密度：明确的低密度、短描述路线】论文用「简洁描述」（concise descriptions）定性其 caption 形态，这与视频生成领域普遍追求的长篇密集描述（dense caption）方向相反。
【为何选择短 caption——三重约束】
1) 编码器约束：CLAP 文本编码器上下文长度有限（约 77 token），物理上无法承载长描述；
2) 任务约束：音频内容的语言可描述性本就低于视觉——「一阵急促的脚步声，伴随远处的雨声」已接近人类对一段 8 秒音频所能给出的信息上限，继续拉长只会引入猜测性内容；
3) 成本约束：10 万小时约 4,500 万片段的标注量下，长 caption 的生成成本不可承受。
【唯一的结构化字段：high-quality 质量标签】这是本工作 caption 设计中最具巧思的一处，见附录 A.1：采样率高于 16 kHz 的音频样本，其 caption 中会被追加一个「high-quality」标签；推理时则对所有输入 caption 一律追加该标签。其机制是让模型在训练中学会「high-quality 标签 ↔ 高采样率、高频细节丰富」的关联，推理时通过强制打标把生成分布拉向高质量端，从而「增强高频细节的保留」（enhanced high-frequency detail preservation）。
这是一种典型的「质量可控标记」（quality conditioning）技术——同类思路见于 MOVA 的「This video has no subtitles.」显式属性标记、以及图像生成领域的美学分数标记。其价值在于：不必把中低质量数据全部丢弃（那样会损失数据量与多样性），而是让模型显式感知质量维度，推理时再定向选择。这实质上是把「过滤决策」延后到了推理阶段，与 UniVerse-1 的 LQLS 损失隔离策略殊途同归（都是「不丢弃低质数据、而是控制其影响方式」），但实现路径更简单——只需在 caption 里加一个词。
【注意阈值的不一致】质量标签的判据是采样率 >16 kHz，而清洗漏斗第 4 级的准入门槛是有效采样率 >32 kHz。两个数字不同，说明标签打在了一个比准入门槛更低的分界线上——这暗示训练数据中可能存在采样率介于 16-32 kHz 之间的样本（即漏斗门槛可能并非对全部数据严格执行，或存在其他数据来源旁路）。论文未澄清这一表面矛盾。[不确定]
【未披露】caption 的平均长度、词表统计、完整示例、生成 prompt 均未公开；无镜头运动/风格/景别等视觉侧结构化字段（因不做视频 caption）。[不确定]

### [HunyuanVideo](../models/HunyuanVideo.md)

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

### [InstructAV2AV](../models/InstructAV2AV.md) ⚠️

本工作的「文本条件」是编辑指令而非内容描述，其结构特征与生成类模型的密集 caption 有本质差异——它描述的是差分（改什么），而非状态（是什么）。

【论文层面的披露】仅表述为「diverse text instructions for comprehensive editing operations」，覆盖四类编辑操作。[不确定] 论文未给出指令的模板、平均长度、词数统计、语法结构或任何示例统计。

【HuggingFace 数据集卡揭示的实际结构（重要旁证，论文未提及）】每条 CSV 记录含四个字段：
  · original_video —— 源视频路径
  · target_video —— 目标视频路径
  · instruction —— 正向编辑指令（source → target）
  · instruction_reverse —— 逆向编辑指令（target → source）
【双向指令的意义】instruction_reverse 字段是论文正文完全未提及、但方法论价值不小的设计：每合成一对 (source, target)，天然可得两条训练样本——「A 加正向指令得 B」与「B 加逆向指令得 A」。这使得数据量直接翻倍，且逆向样本的「source」是合成产物、「target」是真实素材，与正向样本方向相反，可缓解模型只学会模仿数据引擎输出风格的偏置（正向样本的 target 全是合成的，若只训正向，模型的输出分布会被锚定到 Wan2.2-5B 的合成分布上；加入逆向样本后，有一半样本的目标是真实视频分布）。这是合成数据构造中一个巧妙且低成本的增益点。也部分解释了 HF 实际发布 87K 训练对而论文称 79K 的差异来源之一。
【语音标记】指令中可包含 <S> 与 <E> 标记以界定语音内容边界（denote spoken content boundaries）——即指令内嵌一段被特殊 token 包裹的目标台词文本，把「说什么话」这一精确内容与「怎么改」这一编辑意图在同一条指令中分层表达。这是一种轻量的结构化：指令主体为自然语言，语音内容为标记包裹的字面文本，避免模型把台词误当作编辑意图解析。
【与视觉侧结构化标签的对比】不含镜头运动、风格标签、相机参数等文生视频常见的结构化字段——因为编辑任务的视觉上下文由 source 视频本身直接提供（通过 Source Concatenation 进入模型），无需用文本作中介描述。这是编辑范式相对生成范式在文本条件上的天然简化。
[不确定] 指令的长度分布、四类编辑操作在指令中的语言表达模式、是否存在指令模板库、<S>/<E> 标记在多大比例样本中出现，均未披露。

### [2026 其他音视频联合生成](../models/JAVG_2026_misc.md) ⚠️

【ALIVE —— 三段式结构化 + 内联时序标签（本批最精细）】
 caption schema 明确分三个字段：
- Subjects（主体）：包含「visual appearance + acoustic profiles」——即每个主体同时携带视觉外观描述与声学画像（acoustic profile）。把「这个人长什么样」与「这个人声音是什么样」绑在同一个主体条目下，是 AV 模型 caption 设计的关键创新：它让模型在生成时能把身份的视觉与听觉两面统一起来（这也是其 SubjectID correction 阶段要费五步管线校正说话人归属的原因——归属错了，声学画像就会挂到错误的人脸上）。
- Visual（视觉上下文）：场景设定（scene setting）。
- Narration（叙述）：动作的时序序列（temporal sequence of actions），内嵌两类特殊 token：
  · <W>：包裹 verbatim speech content（逐字语音内容）
  · <I>：标记 non-speech acoustic events（非语音声学事件）
即视觉动作叙述、台词、音效三者按时间顺序交织在同一条 Narration 中。这与 Ovi 的 <S>...<E> + <AUDCAP> 方案思路相近但更彻底——Ovi 把音频描述统一放在末尾，ALIVE 则让非语音事件 <I> 也内联到时间轴上，时序表达力更强。
配合人工标注的 sub-motion units 起止时间戳，Narration 的时序精度有 ground-truth 支撑。
【NAVA —— 统一模板的结构化密集 caption（四大板块）】
明确采用「structured dense captions rather than free-form short descriptions」（结构化密集描述而非自由形式的短描述），统一模板覆盖四个板块：
- 全局视觉语义：style（风格）、mood（氛围）、subject（主体）、appearance（外观）、background（背景）、lighting（光照）
- 时序动态：initial state（初始状态）、action sequences（动作序列）、motion details（运动细节）、ending state（结束状态）——这是一个明确的「起始态→过程→终止态」三段式时序骨架
- 镜头行为：camera angle（机位角度）、shot scale（景别）、framing（构图取景）、composition（构图）
- 音频描述：speech（语音）、SFX（音效）、music（音乐）、ambient sound（环境音）四类
合计约 18 个显式字段，是本批中字段最完整的 caption schema。特别值得注意的是它把「镜头行为」单列一个板块（4 个字段），这在 AV 生成工作中不常见，说明 NAVA 对相机可控性有明确诉求。
【OmniCustom】音频 caption 沿用 Ovi 体系：说话人 age、gender、accent + vocal characteristics（pitch、prosody、emotion、speaking rate）共 7 项属性；另有 GLM-ASR 产出的逐段转写。视觉 caption 结构未描述[不确定]。属于「音频侧结构化、视觉侧不详」的偏科设计，符合其身份+音色定制的窄任务。
【Baton —— 不用文本 caption，用连续特征做「语义蓝图」（本批最异类）】
Baton 的核心主张恰恰是「文本 caption 不够用」：现有方法依赖 off-the-shelf 文本编码器的粗粒度 embedding，「discards fine-grained semantics」（丢弃细粒度语义）且缺乏共享的长时规划。其替代方案是让 VA-Planner（Qwen3-8B 底座）产出 planned tokens，并通过 dual semantic alignment towers（双语义对齐塔，各带 learnable queries 做跨模态注意力）把 planned tokens 投影到预训练感知编码器的连续特征域——视频侧对齐冻结 SigLip2 的倒数第二层特征，音频侧对齐冻结 WavTokenizer 的倒数第二层特征，用 L2 回归监督。论文明确说明为何不用离散 token：「regressing continuous features preserves richer semantic structure」（回归连续特征能保留更丰富的语义结构）。
这在 caption 结构谱系中代表了一个新方向：从「越来越结构化的文本 caption」转向「绕过文本、直接用连续感知特征作为跨模态语义中介」。若这条路线成立，未来 AV 数据管线的重点可能从「怎么写好 caption」转向「怎么选好感知编码器」。
【CCL / StreamChar / ITS-JAVG】未描述 caption 结构[不确定]。StreamChar 使用的是 transcript（台词全文）而非描述性 caption，其编排器输入为「transcript 与历史上下文」，属于「脚本驱动」而非「描述驱动」的条件形式。

### [音视频联合生成基线合集](../models/JavisDiT_baselines.md) ⚠️

【MM-Diffusion / AV-DiT】无 caption（无条件生成），该维度不适用。
【JavisDiT / JavisDiT++】caption 结构继承自上游数据集，非自建：
- 视频侧：TAVGBench 提供的单条自然语言描述（同时描述视听内容），密度属于中等长度的整体描述，无结构化字段。
- 音频侧：各音频数据集自带的音频 caption，风格差异大——AudioCaps/Clotho 为人工写的一句话音频描述，WavCaps 为 ChatGPT 辅助生成的描述，ESC50/UrbanSound8K/GTZAN 等本质是类别标签而非描述句。这种「多来源 caption 风格混杂」是其音频预训练数据的固有特点，论文未做风格统一化处理[不确定]。
- CSV schema 层面确有结构化管理：path、id、relpath、num_frames、height、width、aspect_ratio、fps、resolution、audio_path、audio_fps、text、audio_text ——但这些是元数据字段而非 caption 内部的结构化标签。
- 无镜头运动、景别、光照、风格等显式结构化标注[不确定]。
【Harmony（本合集中结构化程度最高）】Gemini 一次标注产出三类分离的文本字段：(1) ASR 转写（transcript）、(2) 描述性视频 caption（video caption）、(3) 背景音 caption（audio caption / background sound caption）。这是明确的多字段分流（factorized）结构，与 Ovi 的「单条 caption 内嵌标签」路线相反、与 Script-a-Video 的因子化流更接近。这套三字段结构直接决定了 Harmony-Bench 的条件设置：环境音子集用音频/视频 caption 作条件，语音子集以转写文本为主要条件，复杂场景子集则使用「全套多模态 prompt」——即三个字段可按任务灵活组合，是 factorized 设计带来的直接好处。prompt 模板细节与各字段的平均长度未披露[不确定]。
【UniAVGen】未描述[不确定]。

### [可灵 3.0 / 可灵视频 3.0 Omni](../models/Kling_30_Omni.md) ⚠️

[不确定，有同团队公开范式] 可灵3.0 Omni 的caption结构未公开，但 Kling-Omni 报告指出预训练文本条件“从简短提示到详尽叙述（concise prompts to elaborate narratives）”跨度很大，即刻意做了caption长度/密度的多样化混合，以保证模型对长短prompt均鲁棒（这也是其可承接vCoT长指令的数据基础）。同团队 Koala-36M 公开了明确的六字段结构化caption schema：主体（subject）、主体动作（subject actions）、环境（environment）、视觉语言（风格/光照/构图）、镜头语言（运镜/机位角度/景别）、世界知识（world knowledge），平均长度202词（对比 Panda-70M 的13.2词）。可灵3.0 强大的运镜控制与“智能分镜”能力高度依赖此类含镜头语言字段的结构化密集caption。

### [LTX-2](../models/LTX-2.md)

走「超长、密集、单段连贯、纯事实」路线，是典型的 dense caption 范式。
【密度】LTX-Video 论文 Fig.14(a) 给出每条 caption 词数分布，跨度约 0–175 词，主体集中在数十词量级；LTX-2 的 caption 因需同时覆盖听觉轨道而更长更密。官方 prompting 指南建议用户写 4–8 个描述性句子、保持为单个连贯段落——推理 prompt 格式即训练 caption 格式的镜像。
【风格约束】论文明确写出打标准则：「comprehensive yet factual, describing only what is seen and heard without emotional interpretation」——全面但只陈述事实，只描述所见所闻，不做情绪化解读/主观评价。这是一条非常明确的 caption 风格规范。
【结构化程度】不采用显式 JSON/键值字段，而是把结构化信息熔入自然语言长描述中，按时间顺序（chronological）组织。LTX-Video 的 caption 示例显示固定覆盖的语义槽位包括：主体外观与细节 → 主体动作/行为 → 场景与背景元素 → 相机机位与运镜（如「camera follows … from behind, pans slightly」/「camera remains stationary」）→ 光照与色彩（「natural and slightly overcast lighting, casting soft shadows」）→ 风格/来源标签（固定后缀如「The scene is captured in real-life footage.」或「The scene appears to be from a movie or TV show.」）。LTX-2 在此基础上并入听觉槽位。

### [LongCat-Video](../models/LongCat-Video.md) ⚠️

结构化程度较高，采用「基础描述 + 属性标签 + 随机组装」的合成式 caption 构造范式：
(1) 基础层：LLaVA-Video/Tarsier2 产出的自然语言描述，兼顾视觉内容与时序动作；
(2) 结构化标签层，分两组——
   · 电影语言（cinematography）：相机运动（pan/tilt/zoom）、景别（shot size）、镜头类型（lens type）；
   · 视觉风格（visual style）：写实度、动画风格、色调；
(3) 增强层：中英互译（生成双语 caption，配合 umT5 双语文本编码器）、生成精简摘要（concise summaries，形成长短两档 caption，使模型同时适配简短 prompt 与详尽 prompt）；
(4) 组装层：「randomly selecting elements from cinematography and visual style categories and integrating them with augmented captions」——从电影语言与视觉风格类目中随机抽取若干元素，拼接进增强 caption。这种随机组装是关键设计：它让模型在训练中见到「有时提及镜头运动、有时不提」的多样 prompt 形态，从而在推理时既能响应显式的镜头/风格指令，也不会在用户不提时强行施加某种镜头语言，是 SFT 阶段相机运动与视觉风格指令跟随能力的数据基础。
未披露 caption 的平均长度、token 数分布以及长短 caption 的混用比例。[不确定：caption 长度分布与长短配比]

### [MOVA](../models/MOVA.md)

caption 结构在中间态是严格结构化的 JSON，在最终态是长篇融合自然语言段落，属“结构化中间表示 → 自然语言最终 caption”的两段式设计，全部 prompt 原文与完整示例见附录 A.5：
【中间态：结构化 JSON】
- 视觉侧输出 video_visual_report，含两个字段：visual_description（视觉元素如何演变的详细描述；无视觉变化时输出 null）与 on_screen_text（所有可见文字的精确转写；无文字时输出 null）；外加 final_verification_audit 自审计块。
- 语音侧输出 speech_transcription_report，含 speech_description（逐字转写；无语音输出 null；不清晰处用 [inaudible] 标记）；外加自审计块。
- 非语音侧为自由文本音频描述（prompt 极简：“Please describe the audio you hear in detail.”）。
【三条标注法则（视觉）】LAW OF VISUAL TRUTH（只描述可视觉验证的元素，不得依据音频或上下文推断）、LAW OF VISUAL SILENCE（无变化输出 null）、LAW OF VISUAL DYNAMICS（检测所有转场，精确记录运动轨迹、速度变化与视觉节奏）。
【最终态：融合长 caption】GPT-OSS-120B 按五条规则合成单一流畅段落，结构上仍隐含顺序化的信息层次：先视觉叙事并锚定说话人上下文（按时间顺序描述动作、物体、场景、光照、运动、转场，不做概括）→ 再把对白作为视觉锚定的引语嵌入（用反映语气的言说动词如 snaps/murmurs/laughs，以及反映语速的表达如 rushes/drawls/pauses）→ 最后以“The audio includes…”或“In the background…”引出非语音音频（仅限环境音、音乐、音频主题/声源、结构性音频变化四类）。
【显式风格标签】未采用离散标签体系，风格/运镜信息以自然语言描述形式内嵌（推理侧的 prompt rewriter 才有显式四类结构：视觉风格含色调与光照、摄影含景别构图、视觉元素含主体与空间关系、OCR 文字）。
【长度】论文给出的完整示例约 200 词的密集长 caption；Figure 6b 给出了 prompt 长度分布统计图（针对评测基准），训练 caption 的长度分布未量化。
【字幕条件化】Phase 2 中无烧录字幕的视频会在 prompt 末尾追加 “This video has no subtitles.”，即 caption 中存在显式的可控属性标记。

### [合并条目：① Mochi 1](../models/Mochi_MAGI_Motif.md) ⚠️

① Mochi 1：未披露。[不确定]
② MAGI-1（两类 caption 并存，是其最有特色的数据设计）：
【Highly Descriptive Caption（高描述性 caption）】沿用 DALL-E 3（Betker et al., 2023）的结论，用 MLLM 生成高密度描述。针对视频比图像多出的时序信息（动作、相机运动、场景变化），采用两阶段 prompt 结构：第一阶段先用一组定向问题引导模型对预定义属性作结构化分析，属性清单（Table 3）共 8 项——Scene Count（识别视频中不同场景的数量）、Camera Transitions（记录镜头间可察觉的转场）、Camera Shot Type（指明所用景别）、Camera Movement（描述相机运动）、Main Subject Identification（确定视频的核心焦点是谁/什么）、Subject Attributes（描述主体外观）、Subject Position（指明主体在画面中的位置）、Subject Action（说明主体在做什么）；第二阶段才生成最终描述性 caption，可吸收第一阶段分析中识别出的显著观察。对图像数据则不做属性预分析、直接生成 caption。注意这些属性是「引导分析用的中间产物」而非独立保留的 JSON 字段，最终落到训练的仍是自然语言散文。
【Auto-Regressive Caption（自回归 caption）】这是 MAGI-1 独有、与其块级自回归架构强耦合的设计。因模型逐块生成、可为视频不同部分提供不同文本条件，团队为每个片段提供「逐秒（second-by-second）」的细粒度描述：第 1 秒的 caption 被要求生成详细描述，后续每秒的 caption 则聚焦于「相对上一秒的变化」。Table 4 的示例清楚展现了这一增量式结构（如「2nd second: 女子头部微倾，表情从中性转为浅笑，口红仍在手中，镜头保持静止」）。这套 caption 使 chunk-wise 文本控制在训练侧有据可依，也是 MAGI-1 能用单镜头素材实现推理侧多镜头叙事的数据基础。AR caption 在训练中按阶段配比使用：stage-1 为 0%，stage-2 与 stage-3 各为 10%（Table 5）。
【推理侧补丁】因模型以结构化高描述性 caption 训练、而真实用户输入从极简到冗长差异极大，团队在推理侧加了 Prompt Enhancement 策略并蒸馏到约 7B 小模型（语料约 200 万条），以弥合训练分布与用户输入分布的错配。
③ Motif-Video 2B（三变体 caption + 固定 JSON schema，直接针对训练/推理分布错配）：
· schema：要求每个响应遵循固定 JSON 格式，同时含自由文本 caption 字段与结构化字段（style、subject、action、camera_move、quality 等），格式非法的响应会被重采样。
· 双 prompt：视频 prompt 与图像 prompt 共享同一 JSON schema、仅在时序字段上不同。视频 prompt 把采样帧视作单一描述对象，并按固定顺序索取——相机属性（景别、机位角度、运动）、主体、动作、环境、光照与色彩、任何屏幕上的文字。图像 prompt 去掉时序字段，改为索取构图、取景与文字的逐字转录。
· 反幻觉约束：两个 prompt 都明令禁止三件事——做出画面中无依据的断言、逐帧式叙述（frame-by-frame narration）、以及对质量/流畅度/氛围的主观评论；目的是减少幻觉标签与描述漂移。
· 三变体与采样概率（针对训练/推理错配的核心设计）：每个片段保留同一次 VLM 响应派生的三个 caption 变体——caption_long（150–250 词的详细描述）、caption_short（15–25 词的单句）、caption_truncated（只保留 caption_long 的首句）；训练时按固定概率 (p_long, p_short, p_truncated) = (0.5, 0.3, 0.2) 采样。论文说明其意图是缩小「长合成 caption」与「用户实际给的短 prompt」之间的训练—测试错配，并指出短/截断变体同时起到温和的 caption dropout 作用、降低对 VLM 特定措辞的过拟合。论文诚实标注这是「务实的配方选择而非经隔离验证的最优性主张（a pragmatic recipe choice rather than an isolated claim of optimality）」。
对照要点：MAGI-1 与 Motif 面对的是同一个问题（训练用长密集 caption vs 用户给短 prompt），但解法方向相反——MAGI-1 在推理侧加 Prompt Enhancement 把短 prompt 补长，Motif 在训练侧混入短 caption 让模型直接适应短输入。这是本条目中一个非常干净的方法论对照。

### [Movie Gen](../models/Movie_Gen.md)

视频caption特征：稠密详细（detailed generated captions），平均长度约100词，具有「细节密集 + 段落结构一致」的统一风格。结构化成分包括：
· 相机运动前缀：16类相机运动分类结果（zoom in / zoom out / push in / pull out / pan right / pan left / truck right / truck left / tilt up / tilt down / pedestal up / pedestal down / arc shot / tracking shot / static shot / handheld shot）在高置信时前置拼接到caption，使推理时用户可显式指定运镜。
· FPS token：把帧率作为token加入caption，实现16~32 FPS的采样帧率控制。
· SFT阶段人工补全的字段：相机控制、人物表情、主体与背景信息、详细运动描述、光照信息；并新增标注6类镜别/机位（wide angle、close-up、aerial、low angle、over the shoulder、first person view），合计22类相机相关标签。
· 推理侧对齐：因用户实际prompt通常不足10词，与训练caption的长度和风格差异大，专门设计了 inference prompt rewrite 把短prompt改写成训练caption风格。
音频caption为高度模板化的结构化四段式（见 joint_av_caption_schema）。

### [NVIDIA NeMo Curator](../models/NeMo_Curator.md) ⚠️

【密度】走密集长描述路线。Cosmos WFM 生产版的实测统计：单条 caption 平均 559 个字符 / 97 个词。
【时间粒度】以「每 256 帧一条 caption」为标注粒度，即长 clip 会被切成多个时间段分别描述，而非整段一条——这使 caption 具备粗粒度的时序对应关系。
【采样方式】VLM 输入为均匀采样的 8 帧。
【增强流程】开源版支持两段式：VLM 先生成基础 caption，再由 LLM（Qwen-LM）做改写增强，可用于风格统一、补充细节或生成多种长度变体。
【下游用途】caption 会进一步经文本嵌入模型编码（26.04 起语义去重默认改用 vLLM 后端的 google/embeddings-gemma-300m 嵌入模型），文本嵌入与视频嵌入一并写入 WebDataset。
【结构化程度】官方未公布 caption 的模板、槽位定义或结构化字段规范（如是否强制覆盖主体/动作/场景/运镜/光照等），也未公布 prompt 全文。可确定的是运镜类型（pan/zoom/tilt）由独立的运动分类器以标签形式产出，而非依赖 caption 文本——即结构化标签与自然语言描述是两套并行的标注体系。[不确定]

### [OmniHuman 数据集 + OHBench](../models/OmniHuman.md) ⚠️

采用「层次化标注 + 两阶段生成 + 占位符锚定」的结构，是本工作方法层面最有辨识度的设计：
【三个层次的标注体系（论文的核心组织原则）】
- 视频级 / 全局层（video-level scenes）：场景类型、景别（shot scale）、相机运动（camera motion）、背景、光照、背景声标签、音乐属性（类型/情绪/相对音量）；
- 帧级 / 交互层（frame-level interactions）：人-人交互标签（含说话人 ID 与情绪）、人-物交互标注、DWPose-L 的 134 个全骨架关键点、人脸清晰度分数 Cs = Var(ΔR)、唇同步质量评估；
- 个体级 / 属性层（individual-level attributes）：人物 ID 指派（人脸嵌入相似度 > 0.55）、肢体动作、外观、表情状态、带情绪与时间戳的语音转写、语音质量标注。
三层分别对应论文所指出的现有数据集三大缺陷（场景多样性不足 / 交互建模稀疏 / 个体属性对齐不足），结构与问题诊断严格一一对应，是论文叙事上很干净的一点。
【结构化输出形式】每条视频产出一组逐主体的结构化记录：{(ID_i, B_i, K_i, X_i, A_i, C_i)}，其中 ID 为身份、B 为包围框、K 为关键点、X 为属性、A 为音频相关、C 为层次化 caption。这是一种「以人为单位组织」而非「以视频为单位组织」的 schema——对多人场景尤其重要，因为它天然支持「第 2 个人在说话、第 1 个人在倾听」这类关系表述。
【两阶段 caption 生成】
- 阶段一：Qwen3-Omni 抽取碎片化属性（外观、动作、表情、语音、音乐、音效等分散的属性集）；
- 阶段二：把碎片化属性合成为连贯的长篇叙事式 caption（coherent long-form narrative caption）。
即「先结构化、再自然语言化」，与直接让 MLLM 一次性写长 caption 相比，这种做法把「看到什么」与「怎么描述」解耦，能显著降低长文本生成中的属性遗漏与臆造。
【占位符锚定机制（placeholder mechanism）】caption 中预留固定位置的锚点，如 [speech_m] 之类的占位符，在生成后再被替换为实际内容（如 ASR 转写的真实台词）。这一设计解决了一个具体问题：MLLM 在长 caption 中复述语音内容时极易产生幻觉或改写，改为「MLLM 只负责在正确位置留一个坑、真实内容由 ASR 填入」，就从机制上杜绝了台词幻觉。这是本工作在标注可靠性上一个很实用的小设计。
【身份对齐机制】基于参考人脸图像的身份对齐，通过 ⟨REF_i⟩ 后缀标记把 caption 中提到的每个人与具体的参考人脸绑定——使得 caption 中的「左边的男人」「穿红衣的女人」这类指称有确定的视觉锚点，对多人场景的可控生成至关重要。参考人脸的选取方式为「取人脸关键点平均置信度最高的那一帧」。
【caption 语种】提供英文与中文两个版本。
【未披露】caption 的长度统计、词数分布、示例原文、标注 prompt 全部未给出。[不确定]

### [Open-Sora 系列](../models/Open-Sora.md)

两个项目都走**长密集结构化 caption** 路线，且都把结构化字段设计明文写入 prompt 模板，可直接复用：
【Open-Sora 2.0】要求 LLaVA-Video 覆盖**六个方面**：1) 主体（main subjects）；2) 主体动作（subjects' actions）；3) 背景与环境（background and environment）；4) 光照条件与氛围（lighting condition and atmosphere）；5) 相机运动（camera movement）；6) 视频风格（video style）。长度统计（Figure 3）：**超过 70% 的视频 caption 长度超过 75 个词**。此外把**运动分数（motion score）直接追加到 caption 文本末尾**，使推理时可用文本控制运动幅度。
【Open-Sora 1.x】采用「描述 + 显式条件标签」的拼接式结构：在 PLLaVA 生成的自然语言描述之后，追加**美学分、运动分、相机运动**三类标量/枚举条件，形如 `aesthetic score: 5.5, motion score: 10, camera motion: pan left`。这套「score condition」机制是 Open-Sora 1.2 的标志性设计，使得同一模型可在推理时通过文本控制画质与运镜，无需额外条件通道。
【Open-Sora Plan v1.3】prompt 要求模型**按时间顺序**描述物体、场景、人物与相机运动；后处理用 28 条常见开场短语表去掉 「The video shows」 等冗余前缀。对美学分 > 6.25 的样本自动前置 「A high-aesthetic scene」。
【训练-推理 caption 分布错配的显式处理（Open-Sora Plan 独有）】作者指出训练 caption 密集冗长而用户实际 prompt 常不足 10 词，为此专门构建 **19,500 条 caption 的 refiner 训练集**，覆盖四种长度/风格：11,000 条 COCO 短用户 caption、5,000 条 DiffusionDB 标签式 caption、3,000 条 JourneyDB 中等长度 LLM caption、500 条来自 Sora/Vidu/Pika/Veo 官方演示与 GPT 生成的超长超现实 caption；用 ChatGPT 将其统一改写为「主体描述 + 动作 + 场景描述（+ 可选镜头语言与氛围）」的目标格式，再 LoRA 微调 LLaMA-3.1-8B 作为推理时的 prompt 扩写器。这是本次调研中对「caption 分布错配」处理得最系统、且训练数据构成完全公开的案例。

### [Ovi](../models/Ovi.md) ⚠️

属于「单条长 caption、内嵌结构化标签」的设计，而非多字段 JSON。
【总体形态】一条 verbose（冗长/密集）的自然语言 caption，视觉事件按时间顺序叙述，台词以内联标签嵌入其中，音频描述统一放在末尾。
【标签体系】
- 语音：<S> 台词内容 <E>（start-of-speech / end-of-speech 标签），可在 caption 中多次出现，与视觉事件按时序交织（interleaved），从而隐式编码「谁在何时说什么」的时间信息。
- 音频描述：<AUDCAP> 音频描述 <ENDAUDCAP>，置于 caption 末尾。Ovi 1.1 起该格式简化为纯文本前缀「Audio: ...」，放在 prompt 结尾。
【密度】视觉侧为 verbose 长描述；因输入仅 7 帧，密度偏事件级而非逐帧密集描述。
【结构化字段】未见镜头运动（camera movement）、景别、风格标签、光照等显式结构化字段[不确定]。已知的结构化维度集中在音频侧（说话人属性）与隐式的时序顺序上。
【单一 prompt 条件化】caption 不拆成视频 prompt 与音频 prompt 两路，而是合并成一条送入单个冻结 T5 编码器，同一嵌入分别与视频塔、音频塔做 cross-attention。论文的直觉解释是：视觉上下文细节提升音频的具体性与多样性，声学上下文细节则引导面部动作与肢体动作。消融实验（5.5 节）证实了这一合并设计优于分离编码方案。

### [Script-a-Video](../models/Script-a-Video.md)

MTSS 是本次调研中结构化程度最高的 caption schema 之一，属于「深度结构化 + 显式关系图」类型，与主流的自由文本长描述形成范式级差异：
【总体形态】JSON 式的四流结构，而非自然语言段落。四流为 Reference（参考流）、Shot（镜头流）、Event（事件流）、Global（全局流）。
【流 1：Reference Stream（实体库，回答 WHO 与 WHERE）】
- 定位：持久实体银行（Entity Bank），为整个脚本提供身份锚点；
- 实体分类：person / object / animal / scene 四类；
- 筛选原则：只收录与主线情节相关（integral to the main plot）的实体，边缘元素降级到 Global 流的场景描述；
- 字段：semantic_description（实体整体状态描述）、timestamp（出现时间）、appearance_anchor（外观锚点）；
- appearance_anchor 内部：通用的 detail_description 适用于所有实体类型；针对 person 类别额外扩展细粒度属性——服装（clothing）、配饰（accessories）、发型（hairstyles）；
- 设计收益：后续所有流引用持久 Reference ID 而非重复描述，从根本上保证跨镜头身份绝对一致，同时消除文本冗余。
【流 2：Shot Stream（视觉分段，回答 WHAT-visual 与 HOW）】
- 每个 shot 由精确的 time_range 锚定；
- 视觉空间层：visual_description（对核心动作的客观、按时序的叙述）+ camera 字段（专业电影语言：镜头运动 movements、视角 perspectives、景别 scales）；
- 关系层：references_in_shot 数组（把画面中可见主体映射到 Reference ID）+ active_events（链接到本镜头内并发的听觉事件）；
- 时间层：在描述文本内部嵌入 intra-description timestamps（描述内时间戳），把微动作锚定到全局时间轴，论文称之为实现「surgical synchronization」（外科手术级同步）。
【流 3：Event Stream（音频事件，回答 WHAT-audio）】
- 准入原则：strict audio-visual coupling（严格音视频耦合）——只收录有直接视觉对应物或主题相关性的音频事件，音效必须由画面中可见主体产生；
- 事件类型：dialogue / sfx / music 三类；
- 字段：type、time_range、以及内容块含 speaker（关系绑定到 Reference ID）、line（逐字台词文本）、description（捕捉情绪起伏、发声技巧等细腻语义）；
- 并发处理：同时发生的多个音源拆分为并行事件条目而非合并；
- 降级通道：无关背景噪声过滤进 Global 流的 global_audio；
- 对齐：通过 micro-level timestamps（微观时间戳）与视觉轨精确对齐。
【流 4：Global Stream（宏观语境）】
- scene_description（视频事件的整体描述）、global_style（整体美学风格或类型/genre）、global_audio（不构成独立事件的环境音与背景音乐）。
【两大设计原则】
1) Stream Factorization（流分解）：把持久信息与时变信息分离，降低语义冗余，支持局部更新；
2) Relational Grounding（关系接地）：身份链接（中心化实体库 + ID 引用）+ 时间链接（共享锚点 + 描述内时间戳）把孤立的流重新编织为连贯脚本。
【核心主张：可编辑性】单体式 caption 的致命缺陷是局部修改触发全局重写（"local edits inevitably trigger global rewrites"）——想改一个运镜或一个音效，就得重写整段以维持叙事连贯。MTSS 使依赖关系可追溯，支持精确的局部更新，这是其相对于长密集 caption 的结构性优势，也是「scalability」论证的落点。
【核心主张：可学习性】MTSS 显著缩小了小模型与大模型之间的性能差距（Figure 1 明确以此为第二大卖点），论文解释为：单体式文本要求模型自行解开密集交织的关系，而 MTSS 已经把 WHO、WHERE、WHEN 预先消歧，下游推理模块可专注逻辑推断而非身份解析与时序去歧义。

### [Seedance 2.0 与 Seedance 1.5 pro](../models/Seedance_20_Seedance_15_pro.md) ⚠️

Seedance 1.0 采用「密集描述（dense caption）」风格，融合动态特征与静态特征两类：动态特征细致描述片段中的动作与相机运动，突出变化元素，覆盖运动、主体或场景的变化、镜头运动三个类目；静态特征描述核心人物或场景的特征，覆盖外观、美学、风格等类目。团队为这些类目采集多样数据并进行高质量人工标注用于训练 caption 模型。Continue Training 阶段针对 I2V 任务设计了两种 caption：(1) 同时含动静态细节的原始长 caption；(2) 去除首帧对应静态描述、只聚焦运动动态的短 caption，以强化与训练目标的语义对齐。Seedance 1.5 pro 表述为「显著提升视频描述的丰富度与专业性，并引入音频描述」。[不确定：1.5/2.0 是否新增结构化字段 schema 及字段清单]

### [SkyReels 系列](../models/SkyReels.md)

【SkyReels-V2】走「结构化字段 + 影视镜头语言」路线，是与多数厂商「自然语言长描述」路线的显著差异，也是该系列的方法论核心。结构化表示包含两大组：
- 主体维度：主体类型（subject type / category）、主体外观（appearance）、动作（action）、表情（expression）、位置（position）；
- 镜头维度（sub-expert 提供）：镜头类型（shot type，如特写/中景/远景）、机位角度（shot angle）、机位位置（shot position）、相机运动（camera motion，6DoF）、环境（environment）、光照（lighting）。
论文主张「由 MLLM 提供通用描述 + 由子专家模型提供细粒度镜头语言」的组合式结构表示，是实现 shot-aware（镜头语言可控）生成的数据基础；其中「主体类别」字段还被复用为概念均衡的分类依据——打标 schema 直接服务于数据配比，是很值得借鉴的闭环设计。
【SkyReels-V4】改为「三档 caption 并行」：
- 短 caption（short）：简要描述；
- 长 caption（long）：全面描述环境、主体、光照、氛围；
- 结构化 caption（structured）：遵循标准化的描述顺序，并用特殊 token 标注画面内文字、音效、对白、歌唱、背景音乐五类元素。
三档并存使模型既能响应简短提示也能响应密集提示；推理侧配套「prompt enhancement（提示词增强）」模块，把用户自由输入重写为与训练结构化表示一致的形式——训练 caption 结构即推理接口的典型闭环。各档 caption 的长度分布与混合比例未披露。

### [Sora 2](../models/Sora_2.md) ⚠️

完全未披露。无caption长度分布、密集程度、是否结构化字段（镜头运动、构图、风格标签等）的信息。前代 Sora 1 强调「highly descriptive captions」提升文本忠实度与视频质量，可推断 Sora 2 仍走长密caption路线，且从其「enhanced steerability（增强可控性）」「expanded stylistic range（扩展风格范围）」「follows intricate instructions spanning multiple shots」等能力宣称可反推caption中很可能包含风格标签、镜头/运镜描述与多镜头分段结构，但均属推测。[不确定]

### [Step-Video-T2V](../models/Step-Video-T2V.md)

采用「三路并行 caption」而非单一密集描述，是本条目在打标维度上最值得记录的设计：
1. Short Caption（短描述）：极简，仅聚焦主体与动作（focusing solely on the main subject and action）；
2. Dense Caption（密集描述）：详述关键元素，强调主体、事件、环境与视觉表现，并明确包含镜头运动（camera movements）的描述；
3. Original Title（原始标题）：直接沿用视频自带的原始标题文本，团队给出的理由是引入「风格多样性（stylistic diversity）」——即保留人类真实撰写的、带口语化/标题党/风格化色彩的文本分布，防止模型只见过 VLM 生成的规整句式，从而在推理时更能适配真实用户的随意 prompt。这一点与 HunyuanVideo 用 caption 字段 dropout + 排列组合做增广的目的相同（对齐推理期 prompt 分布），但实现路径不同——阶跃用的是「引入真实人类文本源」，混元用的是「结构化字段的随机重组」。
【结构化程度评价】相比 HunyuanVideo 的七字段 JSON schema（含 Style、Shot Type、Lighting、Atmosphere 等独立字段），Step-Video-T2V 的 caption 是自然语言长短文本而非强字段化 schema，风格/光照/氛围等属性隐含在 dense caption 的自由文本中，未拆成独立可控字段。这限制了训练时按字段做条件控制的能力（其可控性主要通过 TI2V 的运动分标量实现）。
三路 caption 在训练时如何混用/采样（比例、随机切换策略）报告未说明。

### [UniTalking](../models/UniTalking.md) ⚠️

采用「三种并存格式 + 训练时随机采样」的设计，是本工作在标注侧最有辨识度的一点：
【格式 a：详细视频 caption + 音频 caption（拼接）】由 Qwen3-VL 生成详细视频描述，与 Qwen3-Omni-Captioner/Whisper-V3 产出的音频描述拼接为一段文本。
【格式 b：简短视频 caption + 音频 caption（拼接）】同上，但视频描述换为「short and concise」版本。
【格式 c：融合描述（fused description）】视频与音频流同时输入 Qwen3-Omni，直接产出单一的统一音视听描述，不做拼接。
【使用方式：随机采样】训练时对每个样本从三种 prompt 中随机采样一种作为文本条件输入。这一做法的作用是多重的：
1) caption 长度/密度增广——模型同时见到长描述与短描述，推理时对不同详略程度的用户 prompt 都能响应，缓解「训练 caption 过密导致短 prompt 失效」的经典问题；
2) caption 结构增广——模型同时见到「拼接式（视觉段+听觉段界限分明）」与「融合式（视听交织）」两种文本分布，不对单一 caption 风格过拟合；
3) 相当于一种廉价的 prompt 鲁棒性正则化，无需额外数据成本。
这是与 MOVA「统一融合为单一 caption」、UniVerse-1「三字段始终分离」都不同的第三条路线：UniTalking 是「两种形态都要，训练时随机切换」。
【结构化程度】三种格式内部均为自然语言自由文本，无枚举字段（无镜头运动标签、无景别标签、无风格标签、无情绪枚举）。所谓「多层次」（multi-level）指的是详略粒度与融合形态的层次，不是字段结构的层次。
【密度与长度】未给出任何长度统计、未提供 caption 示例、未公开标注 prompt。「detailed」与「short and concise」的具体长度界限未定义。[不确定]
【可控属性的间接证据】实验部分展示 prompt「without any background music」能有效控制生成结果的背景声——说明 caption 中确实包含了对背景音乐有无的描述，且描述足够一致以支撑可控生成。但论文未把这类可控属性显式化为标注规范。
【时序性】未说明 caption 是否包含时间轴上的事件排序，也未说明 caption 对应的是整段片段还是子窗口。[不确定]

### [UniVerse-1](../models/UniVerse-1.md) ⚠️

采用三字段并列的结构化标注，不做融合，属于典型的 factorized schema：
【三路输出】QWen2.5-Omni 被指令输出三条彼此对齐的独立标注：
1) verified speech content —— 核验后的语音内容（Whisper ASR 转写结果经多模态模型核验/规整）；
2) descriptive video caption —— 视频画面的描述性 caption；
3) ambient audio caption —— 环境音/非语音音频的 caption。
【结构化程度】三字段的划分本身是结构化的，但每个字段内部为自然语言自由文本，无进一步的枚举字段（无镜头运动标签、无风格标签、无景别构图字段、无屏幕文字字段）。
【密度与长度】论文未给出 caption 长度统计、未提供完整 caption 示例、未公开标注 prompt 原文。密度属性无法评估。[不确定]
【时序性】caption 对应的是训练时实际采样的约 5 秒窗口，因此在时间粒度上是「窗口级」而非「整片级」——这正是在线标注要解决的核心问题。但在窗口内部是否做时间轴上的细粒度事件排序（如「先…后…」）未说明。
【与训练条件的关系】三个字段作为并列的文本条件送入模型（经 umT5 编码），保持分离而不合并为单一段落——这与 MOVA「标注时分流、训练时融合为单一 caption」的做法路径不同：UniVerse-1 是「标注时分流、训练时也分流」。
【可控属性】未设计任何显式的可控标记（如 MOVA 的「This video has no subtitles.」）。

### [Unison](../models/Unison.md) ⚠️

论文未给出 caption 长度统计、未提供 caption 示例、未公开标注 prompt、未描述结构化字段设计。caption 的密度与风格属完全空白。[不确定]
【可从模型输入接口反推的确定结构——双路文本条件】这是本字段唯一有原文支撑的部分。Unison 的生成过程形式化为以 caption text 与 transcription text 为并列输入的函数，二者是两个独立的文本通道：
1) caption text（画面/场景描述文本）；
2) transcription text（语音转写文本，即「人物说了什么」）。
二者各自经平均池化得到全局语义向量，共同送入 SCG 预测两个门控系数。这构成了一个明确的二字段结构化 schema：描述性文本与台词文本严格分离，不合并为单一段落。
【该结构的功能意义】caption 与 transcription 的分离不只是标注组织形式，而是直接承载功能——SCG 通过比较两路语义向量的相对强度来判断当前是「叙述主导场景」还是「复杂声景场景」，进而动态调节语音流与音效流的相互影响。即文本 schema 的设计与音频架构的门控机制是耦合的：若不区分 caption 与 transcription，SCG 就无从判断场景类型。这是本条目中「标注结构服务于架构设计」的一个清晰案例。
【缺失的结构化字段】未见镜头运动标签、景别构图字段、风格标签、屏幕文字字段、情绪标签、时间轴事件排序等任何进一步的结构化设计。
【异质性隐患】五个数据源的自带 caption 风格差异极大（详见 caption_model 字段），论文未说明是否做过统一改写或格式规整。[不确定]

### [Veo 3 / Veo 3.1](../models/Veo_3_Veo_31.md) ⚠️

可确认两点：(1) 多粒度——「annotated with text captions at different levels of detail」，即同一条数据配有从短到长/从粗到细的多个层级 caption，这是提升 prompt 长度鲁棒性（短 prompt 与长 prompt 均可用）的标准做法；(2) 合成 caption 用于概念扩展——「We generate synthetic captions to improve the variety and diversity of concepts associated with videos in the training data」，即不依赖原始 alt-text/标题，而由 Gemini 重写生成，从而覆盖摄影手法、动作、风格、场景语境等原始元数据不包含的概念维度。这与模型能精确响应「风格、机位、运镜及其组合」的能力描述相互印证。[不确定] 未披露是否存在结构化字段 schema（如显式的 camera motion / shot size / lighting / style 标签位）、caption 平均长度、每条数据的 caption 数量、多粒度 caption 在训练中的采样比例。

### [Vidu S1](../models/Vidu_S1.md)

双粒度 + 结构化，为流式交互式生成定制：
(1) full-clip caption（整段级）：为整条视频提供连贯的全局语义锚点；
(2) speech-aware chunk-level caption（语音感知的分块级）：将描述与其对应的时间区间对齐，为可控的交互式流式生成提供细粒度、时序局部化的条件信号——这是适配自回归流式范式的关键设计，使得用户可在生成过程中随时用语音指令改变后续内容。
结构化字段覆盖视觉、听觉、对白三类属性，明确列举：主体外观（subject appearance）、动作（actions）、运动（motion）、情绪（emotion）、场景上下文（scene context）、镜头语言（camera language）、影视化属性（cinematic properties）、光照（lighting）、画面文字（on-screen text）、对白（dialogue）、音效（sound effects）、背景音乐（background music）共12类。

### [主流视频预训练数据集合并调研：Panda-70M、InternVid、Koala…](../models/pretraining_datasets.md) ⚠️

**caption长度与结构化程度是七者最直观的差异**（平均词数按UltraVideo Table 1统一口径）：Panda-70M **13.2词/0字段**；InternVid **17.6词/0字段**；LVD-2M **88.8词/0字段（但时序密集）**；OpenVid-1M **126.5词/0字段**（HD子集104.5词）；Koala-36M **202.1词/6字段（融合成一段）**；MiraData **318.0词/6字段（独立可寻址）**；UltraVideo **824.2词/10维度**（long 850.3词）。
【无结构化的四家】Panda-70M：一句话13.2词，caption的教师来源分布见论文图11。InternVid：17.6词，约50%在10–20词、约1/3不足10词、约11%超过20词。OpenVid-1M：LLaVA单条自由文本，论文仅以图示定性说明「显著长于Panda-50M」，**未给出词数**（126.5词来自UltraVideo表转引）。**LVD-2M是独特的中间形态**——其结构化不体现为字段而体现为**时序密集性**：先分30秒段各自描述、再按时序合成，因此caption天然描述「事件如何随时间推进」而非静态画面罗列；词数分布30–50词约2%、50–70词约20%、**70–90词约38%**、90–110词约24.5%、>110词约13.5%。
【六字段结构化的两家，字段设计惊人地一致，可直接复用】
- **Koala-36M六字段**：(1)主体 subject、(2)主体动作 actions、(3)所处环境 environment、(4)**视觉语言 visual language**（风格/构图/光照）、(5)**镜头语言 camera language**（运镜/角度/焦距/景别）、(6)**世界知识 world knowledge**。六字段**分别生成后融合成一段连贯行文**，最终以单个caption字段发布（CSV中caption字符数实测123–3,320）。
- **MiraData六字段（七者中唯一独立可寻址）**：CSV列名即`short_caption`/`dense_caption`/`background_caption`/`main_object_caption`/`style_caption`/`camera_caption`。**正因如此它才能在MiraBench中做「相机对齐/主体对齐/背景对齐/风格对齐」四个分维度的图文一致性评测，别家做不到。** 词数构成：**dense caption约90词 + 四个结构化字段约214词 = 总计318词**（**常被误引为「dense caption 318词」，实为全字段总和**）。实测官方100条样本各字段均值：short 19.0、dense 112.3、main object 84.5、camera 83.8、background 77.2、style 37.3。GPT-4V prompt的五个编号项精确对应五个字段，**few-shot示例直接用了Sora官方演示prompt原文**（东京霓虹街道女子、红色针织摩托头盔太空人），并明示「参照DALL-E 3，让GPT-4V产出有利于学习文生视频模型的描述」，还特别约束「不要逐帧描述、不要出现'first frame'之类的词」。
【十维度的UltraVideo（结构化程度最高）】前9个由Qwen2.5-VL-72B生成、第10个由Qwen3-4B汇总：(1)Brief Description简述 (2)Detailed Description详述 (3)Background背景 (4)Theme Description主题 (5)Style风格 (6)**Shot Type景别** (7)**Camera Movement运镜** (8)**Lighting光照** (9)**Video Atmosphere氛围** (10)Summarized Description汇总。**「景别」「光照」「氛围」三个维度为七者独有**，明显面向影视化生成需求。各维度平均词数**只在图4(d)与图A1中以分布图呈现，从未列表**；HF浏览器给出字符区间可作代理：Brief 34–526、Detailed 479–2,610、Summarized 586–6,060字符。
【长caption带来的两条被实证的教训】
1. **文本编码器容量必须匹配**：MiraData明确因318词caption放不进CLIP的77 token而改用**Flan-T5-XXL（512 token）**；LVD-2M则吃了亏，其88.7词caption被冻结CLIP文本编码器截断，作者把I2V文本匹配提升不明显（16%/70%/14%）**直接归因于77 token上限**。
2. **训练caption与用户prompt的分布错配**：LVD-2M指出**VBench的prompt平均仅7.6词**，远接近WebVid的14.1词而非自家的88.7词，因此评测本身对长caption数据集**不利**。UltraVideo的应对是**随机caption采样策略**——以1/3概率从{Brief, Detailed, Summarized}中选一个，若选中Brief或Detailed则再从剩余7个结构化类目中**随机追加一个**作为最终prompt，让模型同时适应长短prompt。**这是七者中唯一针对分布错配设计训练时数据增强的方案，很值得复用。**

## 音视频联合caption结构（是否同时覆盖视觉+听觉轨道、是否分流为独立字段，如LTX-2全音景描述、Script-a-Video factorized streams、Foley-Omni三字段）

`joint_av_caption_schema` · 详细程度: detailed

### [Allegro](../models/Allegro.md)

不适用。Allegro 无音频模态，caption 仅覆盖视觉轨道（含镜头运动），不存在音视频联合 caption 结构、独立听觉字段或全音景描述。

### [Apollo](../models/Apollo.md) ⚠️

Apollo 采用的是「按音频类型分流标注 → 各轨独立产出 → 融合为统一密集 caption」的方案，但在模型输入侧又保留了视觉/听觉双文本通道，属混合形态：
【标注轨道划分】三类标注并行产出——(1) 语音转写（speech transcripts，仅 vocal 子集）；(2) 音频 caption（audio captions，所有子集）；(3) 视频 caption（video captions，所有子集）。语音与歌唱子集额外附加说话人属性（性别、年龄）作为 meta 字段。
【与同类方案的坐标】
- 相比 Foley-Omni 的三字段并列长期保留，Apollo 在数据侧做了融合（merged into unified dense captions）。
- 相比 LTX-2 的单一全音景描述，Apollo 保留了「转写 / 音频描述 / 视频描述」的三轨来源结构与按子集裁剪的条件化 schema。
- 相比 Script-a-Video 的 factorized streams，Apollo 的分流依据是**音频内容类型（人声/非人声、单人/多人/歌唱）**而非叙事要素，这是其 schema 设计最独特之处：schema 本身随音频类别变化。
- 相比 MOVA 的「三轨严格互斥 prompt + 120B LLM 融合并做跨模态一致性裁决」，Apollo 未披露任何轨道间的互斥约束（如禁止视觉标注器参考音频）与融合规则，融合环节是黑箱。
【模型侧双文本通道】架构上音频相关文本与视频相关文本作为两个独立输入分别编码后送入 MM-DiT，因此推理时可对音、视两侧分别下达指令（这正是论文强调的「instruction following in both joint and unimodal settings」的基础），也支撑 T2A / T2V / T2AV 多任务统一。
【缺失】融合规则、字段名、schema 定义、示例均未公开。[不确定]（分轨结构确定，schema 细节缺失）

### [CineDance / CineDance-1M](../models/CineDance.md)

音视频联合 caption 采用「双轨并行、字段分流」设计，视觉与听觉各成体系但通过共享的 anchor token 与镜头索引对齐：
【视觉轨（Qwen3.5-35B-A3B 产出）】镜头五维属性（景别/角度/运镜/叙事功能/时长类别）、转场类型、局部角色表、活跃场景、镜头描述、转场描述。
【听觉轨（Qwen3-Omni-30B-A3B 产出，三字段分流）】
  ① 句级 ASR 转写（sentence-level ASR segments）——此阶段刻意不绑定说话人身份；
  ② 镜头级音频 prompt——覆盖 music（音乐）、ambient sound（环境音）、effects（音效）的自然语言描述，即非语音音景（soundscape）描述；
  ③ 角色音色描述（character voice description）——刻画每个角色的嗓音特征。
【两轨的耦合点】① 共用全局角色 anchor token ⟨charₖ⟩：ASR 句子在后续绑定步骤中挂到具体角色 token 上，从而与视觉轨中该角色出现的镜头对齐；② 共用镜头索引：音频 prompt 是镜头级的，与视觉描述逐镜头一一对应。
【与同类方案对比】相比 LTX-2 的「全音景单段描述」更结构化；与 Script-a-Video 的 factorized streams（分流式多流脚本）、Foley-Omni 的三字段设计属同一思路谱系，但 CineDance 的特色是把 anchor token 一致性机制引入到跨镜头长序列场景，并把「说话人-角色绑定」作为独立可评测的子任务（95.4% 准确率）。
【设计动机】三个音频子任务分开调用而非一次性输出，论文明确说明是为了降低幻觉（reduce hallucination）。

### [CogVideoX](../models/CogVideoX.md) ⚠️

不适用。CogVideoX 的 caption 为纯视觉描述，完全不涉及听觉轨道，训练数据不含音轨，因此不存在音视频联合 caption schema，也没有视觉/听觉分流字段。
级联的 CogSound 侧是否使用文本 caption 作为条件、其音频 caption 结构如何，均未公开 [不确定]；从公开描述看 CogSound 主要以视频特征（GLM-4V 理解 + 帧级特征交叉注意力）为条件，文本条件的角色未见说明。

### [Cosmos-Predict2.5](../models/Cosmos-Predict25.md)

不适用。模型无音频模态，caption 全部为纯视觉/语义描述，不包含任何听觉轨道字段，也不存在视觉流与听觉流分流的 schema。可类比的「多流分解」结构在该工作中体现为空间维度而非模态维度：驾驶域的七路相机各自独立编码、caption 每 150 帧生成一次；以及 caption 按 5 秒时间窗切分的时间维分流。与 LTX-2 全音景描述、Script-a-Video 的 factorized streams、Foley-Omni 三字段等真正的音视频联合 caption 范式无可比性。

### [Data-Juicer 2.0](../models/Data-Juicer.md) ⚠️

[不确定] Data-Juicer 未定义任何音视频联合 caption 的 schema，这是其相对于 LTX-2（全音景描述）、Script-a-Video（factorized streams）、Foley-Omni（[WORDS]/[AUDIO]/[MUSIC] 三字段）等专门工作的显著空白。
【但具备拼装出联合 schema 的零件】DJ 的字段化数据模型使使用者可以自行构造类似 schema：
  · 听觉侧内容描述：video_captioning_from_audio_mapper（Qwen-Audio 根据音轨生成描述）→ 可作为「音景描述」字段。
  · 听觉侧标签：video_tagging_from_audio_mapper（AST，AudioSet 527类）→ 可近似区分语音/音乐/环境音，作为音频类别字段。
  · 语音内容：video_audio_ASR_mapper → 可作为对白转写字段。
  · 说话人属性：video_audio_detect_age_gender_mapper、video_audio_speech_emotion_mapper → 可作为说话人属性字段。
  · 视觉侧描述：video_captioning_from_vlm_mapper / from_frames_mapper → 视觉描述字段。
  · 融合：video_captioning_from_summarizer_mapper 可把上述各字段交给 LLM 融合为一条统一描述——这实际上就是「全音景 + 视觉」统一描述路线的可行实现。
【关键缺失】无音轨源分离算子，因此无法像 Foley-Omni 那样把音轨拆成 speech/effects/music 三个 stem 并分字段描述与验证；无声学能量门控机制来纠正 VLM 的视觉幻觉（看到乐器就写有音乐）。这两点是构建高质量 AV 联合标注的关键环节，DJ 目前不提供。
【结论】DJ 提供了构建 AV 联合 caption 的大部分原材料，但没有提供 schema 设计与跨模态交叉验证的方法论，需使用方自行补齐约30%的关键环节（源分离 + 能量验证 + schema 定义）。

### [Foley-Omni](../models/Foley-Omni.md)

这是本工作最具代表性的贡献，也是 note 中特别标注的「三字段」所指。
【schema 定义】音视频联合标注 Ŝ = ([WORDS] 语音内容, [AUDIO] 音效事件描述, [MUSIC] 音乐描述)。三字段并列、独立可空，共同描述同一片段的完整听觉内容。
【是否覆盖视觉轨道】不覆盖。这是一个重要的设计选择：schema 只结构化「听觉」侧，视觉侧不做文本描述，而是直接以 CLIP（语义）+ Synchformer（时序）双路特征注入模型。理由是视频在本任务中是条件而非生成目标，无需用文本作为中介表示；但标注生成过程是「视听联合」的——Gemini 同时看视频帧和听音轨来产出这三个听觉字段，视觉信息以隐式方式参与了标注决策（例如通过画面判断某个声音是否属于画内声源）。
【是否分流为独立字段】是，且这正是核心。与「全音景单段描述」相比，分流带来三个可操作的好处：(a) 训练时可对不同字段施加不同的条件强度/dropout；(b) 推理时用户可只填某几个字段做定向控制（如只要音效不要音乐）；(c) 清洗时可对每个字段做独立的声学验证并单独置空，而整段描述只能整体保留或整体丢弃，粒度粗得多。
【与源分离的同构性】三字段 ↔ Bandit 的三个 stem ↔ 三类生成任务（TTS/TTA/TTM）形成严格一一对应，这种跨环节的结构一致性是该设计优雅之处：标注schema、验证工具、任务定义、评测维度全部对齐在同一套三分类上。
【评测端的体现】V2ST-Bench 按字段共存模式组织：语音+音效150、语音+音乐120、三者齐全30，直接测试模型在多字段同时非空时的联合生成能力。

### [Goku](../models/Goku.md)

不适用。Goku 无音频模态，caption 仅覆盖视觉轨道（画面内容 + 动作 + 相机运动 + 运动分数），不存在听觉轨道描述、音视频分流字段或全音景（full soundscape）描述。可作为「纯视觉时代 caption schema」的基线，与 LTX-2 全音景描述、Script-a-Video 因子化流、Foley-Omni 三字段等音视频联合 schema 形成代际对照：Goku 的「关键帧+整段」双路融合思路，在结构上与后续音视频模型「视觉流+听觉流」因子化再融合的思路同源，只是维度换成了时间粒度而非模态。

### [HunyuanVideo-Foley](../models/HunyuanVideo-Foley.md)

HunyuanVideo-Foley 不存在传统意义上的「音视频联合 caption」——这是本字段最重要的结论，且其原因具有方法论意义：
【只有音频轨的语言描述，没有视觉轨的语言描述】caption 体系是单轨的：GenAU 生成的音频描述 + high-quality 质量标签。全流程不生成任何画面描述文本。
【视觉信息以特征而非语言的形式进入模型】这是关键设计——视觉语义不经过「画面→文字→文本编码器」的语言中介，而是由 SigLIP-2 直接编码为视觉特征、与音频 latent 一起进入 joint self-attention。即视觉走「特征通路」，文本走「语言通路」，两者在模型内部融合而非在标注阶段融合。
【这一设计与 factorized schema 的本质区别】UniVerse-1 的 factorized streams（视觉 caption / 语音内容 / 环境音 caption 三字段并列）和 MOVA 的融合式单一 caption，都是在「语言空间」内组织多模态信息；HunyuanVideo-Foley 则是在「表征空间」内组织——只有那些无法从视觉特征中推断、需要用户显式指定的信息（期望的声音特性、质量等级）才用语言表达。
【为何这是合理的】对 V2A 任务而言，画面信息是完整可得的（推理时视频就在手边），把它转成文字再转回特征是有损且冗余的；文本条件的真正职能是补充画面无法决定的部分（如「远处传来」的空间感、「金属质感」的音色偏好）。因此文本在本模型中的角色是「补充与控制」而非「描述」，caption 自然应当简短而非密集。
【模态竞争问题】论文明确指出多模态 DiT 需要「化解模态竞争」（resolving modal competition）——视觉条件信息量大、与音频相关性强，容易主导生成而使文本条件失效。其解法是让视觉走 joint self-attention（深度融合）、文本走 cross-attention（独立通路），两条通路结构性分离，避免文本被视觉淹没。这是「联合 schema」问题在架构层而非标注层的解决方案，是本工作在这一维度上真正的贡献所在。
【局限】由于没有视觉 caption，也就无从做「视觉描述与音频描述是否一致」的跨模态一致性校验——这一职能被完全转移给了清洗环节的 ImageBind 语义对齐检测（用 embedding 相似度而非语言比对来判定一致性）。

### [HunyuanVideo](../models/HunyuanVideo.md) ⚠️

不适用。两代模型均无音频模态，caption 只覆盖视觉轨道，不存在视觉+听觉双轨 caption 结构。可作为「纯视觉结构化 caption」的对照基线，与 LTX-2 全音景描述、Script-a-Video factorized streams、Foley-Omni 三字段等 AV 方案形成对比：HunyuanVideo 的七字段/多属性 schema 在视觉侧的字段化程度已经很高，AV 模型的音轨字段可视为在此之上的正交扩展。[不确定]

### [InstructAV2AV](../models/InstructAV2AV.md)

本工作的音视频联合文本条件形态与调研中其他工作差异显著：它不是「联合描述」（joint caption）而是「联合指令」（joint instruction），描述对象是跨模态的变更操作而非跨模态的内容。

【是否同时覆盖视听两轨】覆盖，但是隐式的一体化覆盖而非显式分流。一条指令同时蕴含视觉变更与听觉变更，且二者由语义因果绑定——「把狗换成猫」这条单一指令同时要求画面中的狗变猫、音轨中的狗叫变猫叫。指令本身不拆分为「视觉指令」与「音频指令」两个字段，而是让模型从单一自然语言指令中推断出两个模态各自应做的改变。这一点是本工作的核心设定：论文强调「given solely text instructions」（仅凭文本指令）即可完成联合编辑，用户无需分别指定音视频要怎么改。

【与调研中其他 schema 的谱系对比】
  · LTX-2：全音景统一描述（一段文本描述整个声景）—— 描述型、听觉单轨。
  · Script-a-Video：factorized streams（分解为独立的视觉流与听觉流）—— 描述型、双轨分流。
  · Foley-Omni：[WORDS]/[AUDIO]/[MUSIC] 三字段 —— 描述型、听觉三分流。
  · InstructAV2AV：单条自然语言编辑指令 + <S>/<E> 语音内容标记 —— 指令型、模态融合不分流。
  可见本工作在「结构化程度」这一轴上处于最低端（几乎无字段化），但这不是设计粗糙，而是任务性质使然：编辑指令若强行分流为「视觉改什么/听觉改什么」，反而会破坏其核心卖点（用户只需说一句人话）并引入两模态指令不一致的风险。结构化的收益（可分别控制、可独立验证）在编辑场景下让位于易用性与跨模态一致性。

【结构化的痕迹仍存在于两处】
  1. <S>...<E> 标记——把语音的字面内容（要说的新台词）与编辑意图分层，这是唯一显式的字段化设计。
  2. instruction / instruction_reverse 双向字段——把编辑方向显式结构化。

【视觉侧不做文本描述】源视频的内容不以文本形式进入条件，而是通过 Source Concatenation（源 latent 与噪声 latent 拼接）直接注入扩散过程。因此模型的「视觉理解」走的是 latent 通路而非文本通路，与 Foley-Omni 用 CLIP/Synchformer 特征承载视觉信息的思路一致——都是「视觉不经文本中介」。

【数据构造侧的联合性保证】值得注意的是，音视频的联合一致性主要不是靠 schema 保证，而是靠数据引擎的生成顺序保证：先按指令合成新音轨，再以音频特征做 frame-wise cross-attention 条件合成新画面。即「联合」是在数据生成的因果链中被强制的，而非在标注结构中被声明的。这是本工作相对其他工作在联合性实现路径上的独特之处。

### [2026 其他音视频联合生成](../models/JAVG_2026_misc.md) ⚠️

本批工作在音视频联合 caption 上分化为四种范式，是 2026 年该维度最具信息量的横切面：
【范式一｜交织内联式（ALIVE）】三字段 + 双标签：Subjects[视觉外观 + 声学画像] / Visual[场景] / Narration[动作时序 + <W>逐字台词 + <I>非语音声学事件]。三条轨道（视觉动作、语音、音效）全部内联到同一条时序叙述中，靠标签而非字段分流。相比 Ovi 的「<S>台词<E> 内联 + <AUDCAP>音景块置尾」，ALIVE 把 <I>（非语音事件）也拉到时间轴上内联，时序表达更彻底。另一独到之处是 Subjects 字段把「视觉外观」与「声学画像」绑定在同一主体下——这是把音视频对齐从「时序层面」提升到「实体层面」的设计，也是其必须用五步 SubjectID correction 管线保证归属正确的根本原因。
【范式二｜分字段并列式（NAVA）】统一模板下音频作为独立板块与视觉板块并列，音频板块内再细分 speech / SFX / music / ambient sound 四类。生产方式是「双模型分产 + 第三模型融合」：Qwen3-VL 产视频 caption、Qwen3-Omni 产音频 caption，Gemini-3-Flash（或 Pro）做拼接或改写融合。这种「先分后合」的生产方式与「分字段并列」的最终结构互相印证，也带来一个隐患：两个模型独立生成时可能出现音视描述互相矛盾，融合模型需要承担一致性校验职责（论文未说明是否有此校验[不确定]）。
【范式三｜属性化音频 caption（OmniCustom）】明确「Following the OVI model approach」构造音频 caption，强调「speaker's age, gender, accent, and vocal characteristics (e.g., pitch, prosody, emotion, and speaking rate)」——即音频 caption 不描述「听到了什么」而描述「说话人是什么样的」，是一种说话人属性化的 schema。配合 GLM-ASR 的逐段转写形成「属性描述 + 台词内容」两轨。这套 schema 直接服务于其音色定制任务：属性可控 = 音色可控。
【范式四｜绕过文本 caption（Baton）】不构造联合 caption，改用 VA-Planner 输出的 planned tokens 作为音视频共享的语义蓝图，视频侧与音频侧分别通过 dual semantic alignment towers 对齐到 SigLip2 与 WavTokenizer 的连续特征域。这是对「联合 caption」这一范式本身的否定——其论点是文本 embedding 天然丢失细粒度语义，用连续感知特征作跨模态中介信息量更足。RS-RoPE（Relative Semantic RoPE）负责把蓝图注入去噪过程。
【无 schema｜CCL / StreamChar / ITS-JAVG】CCL 未描述联合 caption 设计[不确定]；StreamChar 用 transcript（台词）而非描述性 caption 作为主条件，属「脚本驱动」范式；ITS-JAVG 无训练数据，但其验证器 AVHScore 恰恰是在推理时度量「音频事件与视觉事件的语义一致性」——相当于把联合 caption 想要编码的音视语义绑定关系，改用判别式模型在输出端检验。
【与既有工作的谱系定位】把本批放入更大图景：Ovi 为「内联标签 + 末尾音景块」、LTX-2 为「全音景统一描述」、Script-a-Video 为「完全 factorized 独立流」、Foley-Omni 为「三字段拆分」。ALIVE 属于 Ovi 路线的加强版（内联更彻底 + 实体级绑定），NAVA 属于 factorized 路线（分板块并列），OmniCustom 属于属性化专用路线，Baton 则开辟了「非文本中介」的第四条路。2026 年上半年的态势是：内联时序化与分字段结构化两条主流并行，同时出现了绕过文本的探索。

### [音视频联合生成基线合集](../models/JavisDiT_baselines.md) ⚠️

本合集在音视频联合 caption 结构上的做法分化明显，恰好覆盖了三种典型路线：
【路线零：无联合 caption（MM-Diffusion / AV-DiT）】无条件生成，不存在联合 caption schema。音画对齐完全靠架构机制（random-shift attention / 跨模态 adapter）与数据本身的原生同步性，不借助任何文本中介。
【路线一：单条混合 caption（JavisDiT / JavisDiT++）】使用 TAVGBench 的单条文本，该文本同时描述视觉与听觉内容，但不分流为独立字段，也不含时间戳或语音标签。JavisDiT 的关键洞察恰恰在于「单条文本条件不足以保证细粒度时空同步」——因此它不在 caption 结构上做文章，而是在文本之外额外引入 HiST-Sypo（分层时空同步先验估计器），从 prompt 中先估计出粗粒度全局先验与细粒度时空先验，再用该先验同时引导两路去噪。可以理解为：JavisDiT 把「联合 caption 应该编码的时空对齐信息」从文本层移到了隐式先验层。这是与 Ovi（靠 <S>台词<E> 内联标签编码时序）、Foley-Omni（三字段拆分）截然不同的第三条路。
【路线二：三字段完全分流（Harmony，本合集最完整的联合 schema）】Gemini 标注产出三条独立的文本轨道：
(1) 视觉轨道 —— 描述性视频 caption；
(2) 语音轨道 —— ASR 转写文本（transcript）；
(3) 非语音听觉轨道 —— 背景音/环境音 caption。
三者作为独立条件字段而非拼接成一句，因此可按任务组合使用。Harmony-Bench 的三档子集直接验证了这一设计的价值：环境音子集只用视频+音频 caption、语音子集主要用转写、复杂场景子集用全套多模态 prompt——说明该 schema 支持「按需激活条件通道」。这与 Ovi 消融得出的「合并单一 T5 嵌入优于 CLAP/T5 分离编码」结论看似矛盾，实则针对的问题不同：Ovi 的分离是编码器层面的分离（不同编码器导致表征空间割裂），Harmony 的分流是字段层面的分流（可能仍共用同一文本编码器 umT5）[不确定：Harmony 三字段是否共用 umT5 编码后再拼接]。
【UniAVGen】未描述联合 caption 结构[不确定]；其条件形态以参考图像 + 参考音频 + 转写为主，文本 caption 不是核心条件通道。

### [可灵 3.0 / 可灵视频 3.0 Omni](../models/Kling_30_Omni.md) ⚠️

[不确定] 可灵3.0 Omni 的音视频联合caption schema未公开。原生音画同出必然要求训练样本同时具备视觉与听觉两条描述轨道。同团队 Kling-Foley 给出的公开范式是“分流后融合”：先用音频分类模型将音轨分为音效/音乐/语音/演唱，各类分别用音频理解大模型生成音频描述，同时生成视频描述，最后由LLM将视频描述与音频描述融合为统一caption；其开源评测集 Kling-Audio-Eval 则明确要求视频caption与音频caption独立标注（避免跨模态偏置互相污染），并附带声音事件标签（1919个标签/九大声音场景）。这一“独立字段 + 事件标签 + 融合caption”的三层结构，很可能是可灵3.0 Omni 联合caption的雏形，但无官方确认。

### [LTX-2](../models/LTX-2.md)

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

### [LongCat-Video](../models/LongCat-Video.md)

基础版无音频模态，不适用。
Avatar 1.5 的 caption 方案虽用了全模态模型 Qwen3-Omni，但其产出是三个视觉/叙事维度的字段——「Spatial Environment（空间环境）、Interpersonal Relationships（人物关系）、Plot Progression（情节推进）」，并要求描述聚焦客观的物理表现（physical manifestation）而非主观推断。这是为音频驱动人物视频服务的叙事上下文 schema，并非覆盖听觉轨道的音景描述，与 LTX-2 的全音景描述、Foley-Omni 三字段等真正的音视频联合 caption 范式不同。此外 Avatar 1.5 离线阶段还生成「temporal-span caption」（分时段/时间跨度 caption），用于描述局部片段，属于时间维度的细粒度描述而非模态维度的联合描述。

### [MOVA](../models/MOVA.md)

MOVA 采用的是“先分流为三条独立轨道、再由 LLM 融合为单一统一 caption”的 factorized-then-merged 方案，属于 Script-a-Video 式 factorized streams 与 LTX-2 式全音景描述之间的折中：
【三条独立轨道（分流阶段）】
1) 视觉轨（MiMo-VL-7B-RL）：视觉描述 + 屏幕文字，严禁参考音频。
2) 语音轨（Qwen3-Omni-Instruct）：逐字对白转写，严禁包含非语音与音乐。
3) 非语音音频轨（Qwen3-Omni-Captioner）：音效、音乐、环境音、录音质量、说话人音色与口音等声学特征描述。
三条轨道通过互斥的 prompt 约束严格隔离，防止模态间幻觉串扰。
【融合阶段的角色再分配（关键设计）】GPT-OSS-120B 的合并规则明确规定了三条轨道各自的“权威范围”：
- 对白的**实际内容**严格以 speech_description 为准；
- 说话人**动态信息**（说话人总数、说话人切换、音色特征如高音/沙哑）以 audio_description 为准；
- 说话人的**视觉锚定**（谁在说、站在哪、什么装扮）以 video_description 为准。
  三者交叉引用，把“谁说了什么、听起来怎样、看起来是谁”三层信息绑定在一起。
- 非语音音频只允许出现在四个类目中：环境/背景音、音乐、音频主题/声源、结构性音频变化（如静默转向渐强），且严禁提及任何人声与词语。
- 规则 4 明确要求避免跨段重复（如不得在音频段重述说话人的视觉位置）；规则 5 要求跳过空字段、最终读起来像人写的连贯叙事而非刚性结构。
【最终产物】单一融合自然语言段落，不再保留独立字段。因此训练时模型看到的是统一 caption，而非分离的视觉/音频条件。
【与同类对比】相比 Foley-Omni 的三字段并列保留、LTX-2 的全音景单描述，MOVA 的特点是“标注时分流、训练时融合”，兼顾了标注准确性（分流抑幻觉）与训练时的条件简洁性（单一文本条件）。

### [合并条目：① Mochi 1](../models/Mochi_MAGI_Motif.md)

不适用。三者均无音频模态，caption 仅覆盖视觉轨道。Motif-Video 2B 虽有严格的 JSON schema（camera / subject / action / environment / lighting / on-screen text 等字段），但全部为视觉字段，不含任何听觉轨道描述；MAGI-1 的 8 项预定义属性同样纯视觉。不存在全音景描述、factorized audio/visual streams 或独立音频字段。

### [Movie Gen](../models/Movie_Gen.md)

Movie Gen 没有统一的音视频联合caption schema——视觉与听觉分属两条独立数据流：Movie Gen Video 的caption纯视觉、完全不描述声音；Movie Gen Audio 的caption纯听觉、不描述画面（画面信息由视频embedding直接条件输入，文本只作补充）。
但音频caption本身是结构化分流的四字段模板（表27），可视为「听觉轨道内部的factorized schema」：
① 音频质量：This audio has quality: 8.0.（1~10连续分，由音频质量预测模型给出，标注方式仿 LAION aesthetic）
② 人声与音乐存在性：This audio does not contain speech. This audio does not contain vocal singing. + This audio contains music with a 0.90 likelihood.（speech/singing用AED后验过阈值的二值输出；music因存在歧义——如影视中的riser既像音效又像音乐——保留为连续后验概率）
③ 声音描述：This audio has a description: 「gentle waves lapping against the shore, and music plays in the background.」（通用音频caption模型的自由形式描述）
④ 音乐风格描述：This audio has a music description (if applicable): 「A beautiful, romantic, and sentimental jazz piano solo.」（音乐caption模型输出mood与genre；无论样本是否含音乐都无条件追加此字段，配合②的音乐概率共同实现最佳可控性）
论文强调「优先视频对齐而非文本对齐」，因为文本只是补充、无法覆盖视频中的全部细节，且文本对最终观众不可见。

### [NVIDIA NeMo Curator](../models/NeMo_Curator.md)

无。这是该工具链面向音视频联合生成场景的核心空白。
视频 captioning 只描述视觉轨道，不涉及任何听觉内容（无音乐/环境音/foley/对白的描述字段）；音频 pipeline 只产出 ASR 转写文本，不与视频画面关联。框架中不存在能同时覆盖视觉与听觉两条轨道的 caption schema，既没有 LTX-2 式的「融合式全音景长描述」，也没有 Script-a-Video 的 factorized streams 或 Foley-Omni 的三字段设计。
26.07 引入的 Nemotron 3 Nano Omni 是一个具备音视频理解能力的全模态模型，理论上具备产出联合 caption 的潜力，但官方文档中它的用途是 OCR 合成数据的质量打分与视频 captioning 后端，未见音视频联合标注的 stage 或示例。若用该框架服务 AV 联合生成模型，joint caption 层需完全自建。

### [OmniHuman 数据集 + OHBench](../models/OmniHuman.md) ⚠️

OmniHuman 的联合音视 caption 采取「结构化多字段并存 + 最终合成为单一叙事」的混合方案，且在字段维度上是本次调研样本中覆盖最广的：
【听觉侧字段】背景声标签、音乐属性三元组（类型/情绪/相对音量）、逐说话人的语音转写文本、语音情绪、语音时间戳、语音质量标注、唇同步质量评估。
【视觉侧字段】场景类型、景别、相机运动、背景、光照、人物外观、肢体动作、表情状态、人-人交互、人-物交互、134 关键点、人脸清晰度。
【融合方式】阶段一由 Qwen3-Omni（全模态，同时消费视听两路）抽取上述所有属性——注意这里的抽取本身就是跨模态的，而非视觉模型标视觉、音频模型标音频再拼接；阶段二将全部碎片合成为单一的长篇叙事 caption，视听内容在叙事中交织而非分段并列。
【与三条既有路线的定位对比】
- UniVerse-1「标注分流、训练也分流」（三字段始终独立）；
- MOVA「分头标注、用 120B LLM 合成单一段落」；
- UniTalking「因子化与融合两种成品都保留、训练时随机二选一」；
- OmniHuman 则是「用全模态模型一次性跨模态抽取结构化字段 → 合成单一叙事 caption」，且结构化字段本身也随数据集一同发布（sample_json 中 captions / subjects / speech / audio 四类字段并列）。这意味着使用者既能拿到融合式的自然语言 caption，也能拿到原始的结构化字段自行重组——在实用性上优于只发布最终 caption 的方案。
【最具特色的一点：音乐的「相对音量」字段】显式量化背景音乐与人声的能量关系，使下游可按音景构成筛选数据（如只要人声主导的样本训练配音模型、只要音乐丰富的样本训练全音景生成）。这一粒度在同类工作中未见。
【幻觉防护】通过占位符机制（语音内容由 ASR 填入）与一致性校验（主体数对齐跟踪结果、转写内容对齐 ASR）实现，详见 model_as_data_judge——这是本 schema 相对同类工作的可靠性优势。
【局限】caption 与结构化字段之间是否完全一致未做全面验证（仅校验了主体数与语音内容两项）；音效（sound effects）虽在阶段一的抽取属性中列出，但未见对应的独立标注字段说明，其在最终 caption 中的呈现粒度不明。[不确定]

### [Open-Sora 系列](../models/Open-Sora.md)

不适用。两个项目均不生成音频，caption 只覆盖视觉轨道（主体/动作/背景/光照/运镜/风格），不存在听觉轨道描述字段，也无 LTX-2 式全音景描述或 Foley-Omni 式三字段分流设计。

### [Ovi](../models/Ovi.md)

Ovi 的联合 caption schema 是「交织式内联 + 末尾音景块」的混合结构，同时覆盖视觉、语音、非语音听觉三条轨道：
(1) 视觉轨道：自然语言叙述视觉事件，按时间顺序展开，构成 caption 主干。
(2) 语音轨道：以 <S>...<E> 标签内联嵌入 caption 主干的对应时间位置，实现「视觉事件与台词的时序交织」，是其对齐语义与时序的关键设计（相当于把 script 与 shot description 编织在一起）。
(3) 非语音听觉轨道：以 <AUDCAP>...<ENDAUDCAP>（1.1 版为「Audio: ...」）封装在末尾，是一个完整的音景（soundscape）描述块。其内容按片段是否含语音自适应切换：
   - 含语音片段：强调说话人声学属性——年龄（age）、性别（gender）、口音（accent）、音高（pitch）、韵律（prosody）、情绪（emotion）、语速（speaking rate）；
   - 无语音片段：描述音效（sound effects）、背景音（background audio）、音乐元素（musical elements）。
【与其他方案的对比定位】不同于 LTX-2 的全音景统一描述、Script-a-Video 的完全 factorized 独立字段流、Foley-Omni 的三字段拆分，Ovi 选择「不分流为独立条件」——三条轨道全部塞进同一条文本、同一个 T5 嵌入，靠标签而非字段做区分。
【消融验证】5.5 节的唯一消融就是针对这一点：初版设计用 CLAP 编码器编码非语音音频描述、T5 编码语音转写，试图解耦 T2A 与 TTS 任务；结果发现该分离设置限制了模型生成连贯输出的能力——能单独做音效或单独做语音，但难以把二者融合成统一连贯的音轨。改为合并单一 T5 嵌入后，WER 基本持平（0.033→0.035）而 FD_PANNs 从 20.78 降到 18.03、FD_VGG 从 7.13 降到 5.02、IS 从 8.34 升到 11.20、CLAP 从 0.190 升到 0.224。这为「联合 AV caption 应统一编码而非分流」提供了直接的量化证据。
【纯音频数据的对齐】纯音频语料同样用同一 MLLM 产出「转写 + 音频描述」两部分（无语音则转写留空），schema 与音视频侧保持一致，保证两阶段训练的条件分布不漂移。

### [Script-a-Video](../models/Script-a-Video.md)

本字段是 Script-a-Video 最具价值的贡献所在，其方案在同类工作中处于结构化光谱的最深端：
【定位对比】沿「视听信息如何组织」这条轴排列：LTX-2 的全音景单一描述（融合式）→ Foley-Omni 的三字段并列（浅分流）→ UniVerse-1 的三路独立字段（分流但无关系）→ MOVA 的分流后 LLM 融合为单段落（分流再融合）→ MTSS 的四流分解 + 显式关系图（分流再显式重连）。MTSS 是唯一在分流之后用「可机读的引用关系」而非「自然语言叙述」把各流重新连接起来的方案。
【是否同时覆盖视觉与听觉】是。Shot 流承载视觉轨，Event 流承载听觉轨（对白/音效/音乐三类），Global 流的 global_audio 承载环境底噪与背景音乐，Reference 流跨模态共用（既被 Shot 的 references_in_shot 引用，也被 Event 的 speaker 引用）。
【是否分流为独立字段】是，且是四流而非常见的二流或三流。关键差异在于多出了一条 Reference 流——这条流不属于任何单一模态，而是作为跨模态的共享身份锚点存在，是 MTSS 区别于其他 factorized 方案的结构性创新。
【跨模态连接的两种机制】
1) 身份链接：Event 的 speaker 字段与 Shot 的 references_in_shot 数组同时指向 Reference 流中的同一 ID。这意味着「谁在说话」这件事在数据结构上被显式表达为「音频事件的 speaker 指针」与「视觉镜头的可见主体指针」指向同一实体——说话人与画面人物的绑定是结构性的、可机读校验的，而非依赖自然语言描述的隐式对应。这是解决音视频身份错配的最直接手段。
2) 时间链接：Shot 的 active_events 字段把镜头链接到并发的音频事件；两侧共用同一条全局时间轴，Shot 与 Event 各自的 time_range 可直接做区间运算；更细的对应由双方的 intra-description / micro-level timestamps 承担。
【音频事件的准入过滤即为对齐保证】严格音视频耦合原则（音效必须由可见主体产生）实质上把「音画不匹配」的样本在标注阶段就化解掉了——不匹配的声音不会成为 Event，而是降级为 global_audio。这是一个用 schema 设计替代过滤器的巧思：不需要额外的音画一致性判别模型，标注规则本身就完成了分流。
【下游生成的直接收益（量化）】把单体式提示词换成 MTSS 脚本、架构完全不变（LTX-2-AV → LTX-2-AV-MTSS）时：人评身份一致性 1.22 → 1.77（+45%）、人评音视频对齐 1.18 → 1.85（+56%）、人评多镜头可控性 1.00 → 1.71（+71%）；WER 从 0.84 降至 0.13；Shot Boundary Deviation 3.79 → 3.27 帧。这组「同架构同训练范式、唯一变量是提示词结构」的对照，是本工作最有说服力的证据——论文明确指出两个基线共享完全相同的架构与训练范式，因此二者之间的任何性能差距可直接且完全归因于 MTSS 范式本身。
【局限】schema 未设置显式的音频质量/响度/声学环境字段（如录音环境、混响、音量），也无语种/口音结构化标签，音色描述仅隐含在 Event 的 description 自由文本中。

### [Seedance 2.0 与 Seedance 1.5 pro](../models/Seedance_20_Seedance_15_pro.md) ⚠️

Seedance 1.5 pro 是该系列首次引入音频侧描述的版本：其字幕系统「为视频与音频两种模态提供丰富的专业级描述」，即在传统「视频-caption」数据之外系统性引入结构化的音频描述（structured audio descriptions），使模型内化更丰富的音视频联合语义空间。但报告未公开具体 schema——未说明是单条全音景描述（如 LTX-2 风格）还是拆分为独立字段流（如 Script-a-Video 的 factorized streams 或 Foley-Omni 的三字段）。从 SeedVideoBench 1.5/2.0 的音频标签体系（人声类型 / 人声属性 / 非语音音频三大主类，2.0 细化到 17 个类目）以及 Seedance 2.0 支持背景音乐、环境音效、角色人声三轨并行输出来看，训练侧极可能采用按音轨分流的多字段标注结构，但这属于推断而非论文明示。[不确定：schema 的确切形态与字段名]

### [SkyReels 系列](../models/SkyReels.md)

仅 SkyReels-V4 具备，采用「单条结构化 caption + 特殊 token 分区」的方案，介于 LTX-2 的融合式长描述与 Foley-Omni 式多字段之间：
【Token 化 schema】结构化 caption 用五种成对特殊 token 显式分区标注：
- <text></text>：画面内出现的文字（in-video text，服务于多语种文字渲染能力）；
- <sfx></sfx>：音效 / 拟音；
- <dialogue></dialogue>：对白内容（Whisper 转写结果直接填入）；
- <singing></singing>：歌唱内容（含歌词转写）；
- <bgm></bgm>：背景音乐。
【设计要点】(1) 视觉与听觉信息在同一条文本序列中共存，由共享的冻结 MLLM 文本编码器统一编码后同时驱动视频流与音频流——这是「共享文本编码器 + token 分区」实现音视频语义对齐的关键；(2) 四个听觉 token 与音频四分类（音效/音乐/语音/歌唱）严格一一对应，分类结果可直接决定 token 的填充；(3) <text> 这一视觉 token 与听觉 token 并列，说明画面文字被视为与音轨同级的「需精确复现的符号内容」；(4) 特殊 token 使推理时可精准控制某一模态成分（例如只指定 <sfx> 而留空 <dialogue> 以生成无对白场景）。
【未披露】token 内部是否有更细的属性槽位（如说话人、情绪、音色）、视觉描述与听觉描述的长度配比、caption 质量校验方式。

### [Sora 2](../models/Sora_2.md) ⚠️

完全未披露。作为原生音视频模型，Sora 2 必然需要覆盖听觉轨道的标注（否则无法响应对白/音效/音乐类文本指令），实际使用中用户可通过prompt中的引号对白直接指定台词内容，说明caption schema中存在明确的对白字段或等价机制。但 OpenAI 未说明是采用单一融合caption、还是像 LTX-2（全音景描述）、Script-a-Video（factorized streams）、Foley-Omni（三字段）那样分流为视觉/对白/音效独立字段。这是与开源同类工作对比时最大的信息空白。[不确定]

### [Step-Video-T2V](../models/Step-Video-T2V.md) ⚠️

不适用。模型无音频模态，caption 只覆盖视觉轨道（主体、动作、环境、视觉表现、镜头运动），不存在视觉+听觉双轨 caption 结构，也不存在音景描述、说话人字段或音效字段。可作为「纯视觉 caption」的对照基线：其三路 caption 设计（短/密集/原始标题）中「保留原始标题以引入真实人类文本风格分布」的思路，理论上可正交迁移到 AV caption 体系（如保留视频原始标题与人工描述作为音景 caption 的风格多样性来源），但报告本身无任何 AV 相关内容。[不确定]

### [UniTalking](../models/UniTalking.md) ⚠️

UniTalking 的联合音视频 caption schema 是本次调研样本中唯一同时提供「因子化」与「融合式」两种方案并让模型随机接触二者的设计，可视为对该问题的一种折中回答：
【两条路线并存】
- 因子化路线（格式 a、b）：视觉描述与听觉描述由不同的专用模型分别生成，然后拼接为一段文本。虽然物理上拼接为单一字符串送入 UMT5，但两段内容在语义上界限分明，接近 Script-a-Video 的 factorized streams 与 Foley-Omni 的多字段并列思路；
- 融合路线（格式 c）：Qwen3-Omni 同时消费视频流与音频流，产出单一的、视听交织的统一描述。这更接近 LTX-2 的全音景单一描述与 MOVA 用大 LLM 融合的思路。
【与两个对照工作的定位】UniVerse-1 是「标注时分流、训练时也分流」（三字段始终独立并列送入模型）；MOVA 是「标注时分流、训练时融合」（分头标注后用 120B LLM 合成单一段落）；UniTalking 则是「两种成品都保留、训练时按样本随机二选一」。三者构成了该设计空间的三个不同点位。
【条件注入形态】无论哪种格式，最终都被压平为单一文本序列（长度固定 512）经冻结 UMT5 编码后送入 cross-attention，模型侧不区分文本来自哪种格式，也没有为视觉描述与听觉描述保留独立的条件通道。这意味着因子化只存在于标注阶段，模型架构层面并不因子化——真正因子化的条件通道是另一路：参考音色音频（c_audio）有其独立的 KV 投影层，与文本条件分离后再求和融合。因此 UniTalking 的条件分离维度是「语义（文本）vs 音色（参考音频）」，而不是「视觉 vs 听觉」。
【转写内容的归属】Whisper-V3 的 ASR 转写被归入音频 caption 一侧，即台词内容与声学环境描述共处同一段文本，未单列为独立字段（对比 UniVerse-1 把 verified speech content 单列为一个字段）。这在训练上要求模型自行从混合文本中分离「要说什么」与「听起来怎样」。[不确定]
【缺失的机制】三种 caption 之间无一致性校验；无跨模态幻觉防护；无 caption 质量打分或过滤；无 caption 与音视频内容的对齐验证。也未做「哪种 caption 格式更有效」的消融。

### [UniVerse-1](../models/UniVerse-1.md)

UniVerse-1 采用纯 factorized streams 方案，是本次调研中音视频联合 caption 结构最「彻底分离」的样本之一：
【三条轨道并列保留，不融合】视觉轨（video caption）、语音轨（verified speech content）、非语音音频轨（ambient audio caption）三者由 QWen2.5-Omni 一次性产出并作为三个独立字段保留到训练阶段，全程不做合并。
【设计取向】与 LTX-2 的「全音景单一描述」、MOVA 的「分流后由 120B LLM 融合为单一自然语言段落」相比，UniVerse-1 的方案更接近 Script-a-Video 的 factorized streams 与 Foley-Omni 的三字段并列。其优势是三个模态的条件可在推理时独立控制（用户可分别指定台词、画面、环境音），劣势是缺少跨模态一致性校验环节——三路标注由同一模型同时产出，理论上共享上下文，但论文未说明是否有机制防止或检测三路之间的语义冲突。
【语音与非语音的显式切分】把「说了什么」与「听起来还有什么声音」拆成两个字段，是对齐其数据配比（语音仅 15.4%、非语音 84.6%）的自然选择：大量样本的 speech 字段为空，ambient 字段为主；少量语音样本则两者兼有。
【与生成任务的对应】三字段结构直接支撑 Verse-Bench 的四类评测任务（联合生成、audio-to-video、video-to-audio、TTS）——因为条件是分离的，可以任意屏蔽其中若干路来构造不同任务形态。
【关键差异化：在线对齐】本方案真正的独特之处不在字段划分本身，而在于这三条轨道的标注是在训练时针对实际采样窗口即时生成的，因此三路标注彼此之间、以及与送入模型的音视频 latent 之间，天然共享同一时间窗口，从根本上消除了离线标注的时序错配。这是「联合 caption schema」在时间维度上的对齐保证，而非仅仅在字段维度上的对齐。

### [Unison](../models/Unison.md) ⚠️

Unison 的音视频联合标注 schema 是二字段 factorized 结构，但其真正的独特性不在标注层而在「标注 schema 与音频生成流的一一对应」上：
【标注层：caption + transcription 二字段并列】视觉/场景描述与语音转写作为两路独立文本条件输入，全程不合并。与 UniVerse-1 的三字段（video caption / verified speech content / ambient audio caption）相比少了一个字段——Unison 没有独立的「环境音描述」字段，环境音信息隐含在 caption text 中。与 MOVA「分流标注后由 120B LLM 融合为单一段落」的做法路径不同，Unison 是分流标注、分流使用。
【生成层：speech 流 + sfx 流双流并列】音频 latent 被物理拆分为语音与音效两条流，各自有独立的 ground-truth latent（Mel-RoFormer 分离产出）与独立的 flow-matching 损失。
【两层之间的对应关系——本工作的结构性特色】transcription text 天然对应 speech 流（说什么话），caption text 天然对应 sfx 流与视觉流（画面中有什么、因此该有什么声音）。SCG 正是利用这一对应关系工作：以 transcription 的全局语义向量调节 speech 流对 sfx 流的门控，以 caption 的全局语义向量调节 sfx 流对 speech 流的门控。因此「标注 schema 的字段划分」与「音频生成的流划分」在 Unison 中是同构的——这是一种比单纯的 factorized caption 更彻底的解耦：不只标注分流，连生成目标与监督信号都分流。
【与同类方案的定位】
- LTX-2「全音景单一描述」：标注不分流、生成不分流；
- MOVA：标注分流后融合为单一 caption，生成不分流；
- UniVerse-1：标注三路分流，生成不分流（单一音频 latent）；
- Foley-Omni/Script-a-Video：标注多字段分流，生成情况各异；
- Unison：标注二路分流 + 生成二流分流 + 两者一一对应 + 门控动态再平衡。在解耦彻底性上是本次调研中最高的一档。
【代价与局限】缺少独立的环境音描述字段意味着用户无法像 UniVerse-1 那样单独指定环境音内容，环境音只能通过场景 caption 间接控制；同时两路标注均未经跨模态一致性校验，caption 与 transcription 之间若存在语义冲突（如 caption 描述安静图书馆而 transcription 是喊叫）无检测机制。论文未讨论。[不确定]
【消融支撑】Table 2 显示移除 SGHS（语义引导谐和策略，即取消双流解耦）导致 PQ 从 6.34 跌至 6.12，是全部音频侧消融中降幅最大者，论文归因于「频谱干扰」（spectral interference）；移除 SCG（即保留双流但取消语义门控）PQ 降至 6.21、LSE-C 从 3.30 降至 3.22。这两项共同证明了「解耦」与「基于文本语义的再平衡」各自的独立贡献。

### [Veo 3 / Veo 3.1](../models/Veo_3_Veo_31.md) ⚠️

[不确定] 官方表述为「Audio and video clips were annotated with text captions at different levels of detail」，即音频与视频均被文本化标注，说明 caption 覆盖听觉与视觉两条轨道；模型能按 prompt 中的对白引号、音效描述、环境音描述分别响应，也说明训练 caption 中这三类听觉信息是被显式描述的。但官方从未说明其组织形式：是像 LTX-2 那样融合为单条「全音景」描述，还是像 Script-a-Video 那样分流为 factorized streams，抑或像 Foley-Omni 那样拆为独立三字段；音视频 caption 是否共享同一条文本、是否分别注入不同 cross-attention 分支，均无公开信息。

### [Vidu S1](../models/Vidu_S1.md)

是典型的「音视频联合 caption 但双路解耦」方案：caption 同时覆盖视觉轨道与听觉轨道（听觉侧含 dialogue、sound effects、background music 三类字段）。为提升标注保真度、减少跨模态幻觉（cross-modal hallucination），采用 dual-path strategy 解耦两个模态——视觉属性只从视频帧推断（inferred exclusively from video frames），听觉属性只从音轨推断（inferred exclusively from the audio track），二者互不交叉参考后再合并为结构化描述。论文称该结构化标注方案提升了多模态表征的质量与一致性，并使标注更可靠、生成更可控。与 LTX-2 的全音景描述、Foley-Omni 三字段属同一类思路，但 Vidu S1 的特色是叠加了 speech-aware 的时序分块对齐。

### [主流视频预训练数据集合并调研：Panda-70M、InternVid、Koala…](../models/pretraining_datasets.md)

不适用。七个数据集的caption**全部只覆盖视觉轨道**，不含任何听觉轨道描述，也不存在视听分流的字段设计。即便是字段最多的UltraVideo（10维度）与MiraData（6个独立字段），其维度也全部是视觉的（主体/背景/风格/景别/运镜/光照/氛围等），无一涉及音景、音效、语音或音乐。因此不存在LTX-2式全音景描述、Script-a-Video式factorized streams或Foley-Omni式三字段结构的对应物。**这是本次调研七个数据集与音视频联合生成模型数据体系之间最根本的能力断层：现有主流视频预训练数据集无法为AV模型提供任何听觉侧监督信号，AV模型的音视频联合数据必须完全另起炉灶自建。**

## 对白转写与说话人属性标注（ASR转写+说话人身份/语言/口音/情绪）

`dialogue_transcription_attributes` · 详细程度: brief

### [Allegro](../models/Allegro.md)

不适用。无音频处理链路，未做 ASR 转写、说话人身份/语言/口音/情绪标注。

### [Apollo](../models/Apollo.md) ⚠️

这是 Apollo 数据体系中披露相对最实的一环，也是其相对同类工作的差异化投入：
【转写】对 vocal 子集（含歌唱、单说话人语音、多说话人语音）做语音转写，三模型并用：Whisper-Large-v3、SenseVoice、Qwen2.5-Omni。三者能力互补（Whisper 多语种通用、SenseVoice 中文/方言与情感事件、Qwen2.5-Omni 全模态上下文理解），推测用于交叉验证或按场景分派，但仲裁规则未披露。
【说话人属性】明确抽取说话人属性并举例为性别（gender）、年龄（age）：「For speech and singing, we extract speaker attributes (e.g., gender, age)」。「e.g.」说明属性集不止这两项，但完整清单未给出。相比 MOVA 以自然语言描述音色/口音/语速的做法，Apollo 的性别、年龄更接近离散结构化字段。
【说话人数量维度】通过数据切分而非标注字段来表达——单说话人语音与多说话人语音是两个独立子集，模型可据此区分单人/多人场景。但多说话人子集内部是否做了说话人分离（diarization）与说话人标签（[S01]/[S02]）标注，论文未说明；这恰是 MOVA 明确点出的多说话人场景瓶颈。
【口音/情绪】未提及口音标注；未提及情绪标注（尽管 SenseVoice 原生支持情感识别）。
【评测对应】评测指标含 WER（词错误率）与 SyncNet Confidence（唇同步置信度），对应验证转写内容的可生成性与唇形对齐。Table 3 显示全任务训练把 WER 从 0.044 降到 0.028。
[不确定]（属性集完整清单、diarization 处理、多 ASR 仲裁规则缺失）

### [CineDance / CineDance-1M](../models/CineDance.md) ⚠️

对白处理是该数据集在 AV 维度上做得最细的部分：
【ASR 转写】由 Qwen3-Omni-30B-A3B 完成句级（sentence-level）转写，而非整片流式输出，切分粒度对齐句子边界。
【说话人身份绑定（核心贡献）】设计为独立于 ASR 的后续步骤，把每条 ASR 句子绑定到全局角色 anchor token ⟨charₖ⟩。采用窗口化（windowed）方案：先过滤掉非语音区间，窗口切分同时保证镜头不被切断、句子不被截断。
【定量效果（Tab.5 / Tab.7）】在 100 条人工标注片段的基准上：
  · Qwen3-Omni-30B-A3B + 窗口化：95.4%（整片输入仅 67.2%，滑窗 prompt 83.1%，整片 56.4%——不同表中口径略有差异）；
  · Gemini 系列模型：82.8% ~ 87.4%；
  · DiariZen：63.1%；
  · Pyannote-3.1：62.7%。
  结论是 Omni 大模型 + 窗口化工程显著优于传统专用说话人分离工具。
【音色属性】提供 character voice description 字段描述角色嗓音特征。
【缺失维度】未见显式的语言标签、口音标签、情绪标签字段，也未标注说话人的性别/年龄等结构化属性。[不确定]

### [CogVideoX](../models/CogVideoX.md) ⚠️

无。CogVideoX 数据 pipeline 中没有 ASR 转写、没有说话人身份/语言/口音/情绪标注，模型也不生成语音。相反，团队主动把「Lecture Type（以人物持续说话为主、有效运动极少）」列为负面标签并以 0.99 准确率的分类器整类剔除，等于在数据层面就排除了对白密集的素材。CogSound 侧是否处理语音未披露 [不确定]。

### [Cosmos-Predict2.5](../models/Cosmos-Predict25.md)

不适用。无音频输入，因此不存在 ASR 转写、说话人身份/语言/口音/情绪标注等任何环节。与「人」相关的标注走的是纯视觉几何路线：Human Dynamics 域用 YOLOX 做人体检测、RTMPose 做全身关键点与面部 landmark 估计，用于筛选画面构成（人物帧占比、人数上限、单人面积占比），而非用于说话或表情语义标注。

### [Data-Juicer 2.0](../models/Data-Juicer.md) ⚠️

【转写能力】video_audio_ASR_mapper —— 从视频的音轨做语音识别，文档描述为「基于 Audio Spectrogram Transformer 从音频流生成视频标签」（该算子在文档中与 tagging 类算子描述重合，实际承担 ASR 与音频事件标注职责）。可产出语音转写文本作为独立字段。
【说话人属性——DJ 覆盖较好的一组能力】
  · video_audio_detect_age_gender_mapper —— 「基于预训练 wav2vec2 模型从视频音频信号中检测年龄与性别」。产出说话人的年龄段与性别属性，这在开源数据处理框架中较少见。
  · video_audio_speech_emotion_mapper —— 语音情绪识别，产出情绪标签字段。
  · video_active_speaker_detect_mapper —— 「通过分析视觉人脸轨迹与音频信号检测视频中的主动说话人」。这是把「说话人」从音轨绑定到画面中具体人物的关键算子，多说话人场景下用于确定「此刻是谁在说话」。
  · video_captioning_face_attribute_emotion_mapper —— 为每个被追踪的人生成面部属性与情绪的自然语言描述，是视觉侧的情绪证据，可与语音情绪交叉验证。
  · video_human_tracks_face_demographic_mapper —— 人物轨迹的人口统计属性。
  · video_human_tracks_extraction_mapper —— 提取人脸与人体边界框轨迹，是上述人物算子的公共前置。
【属性覆盖对照】年龄✓、性别✓、情绪✓（音+视双路）、说话人身份（轨迹级 ID）✓、说话人-画面绑定✓；语言✗、口音✗。
【应用佐证】HumanVBench（CVPR 2026，同团队）明确基于 Data-Juicer 构建，使用20+个 SOTA 算子搭建了「人物中心视频标注 pipeline」（Human-Centric Video Annotation Pipeline），基准覆盖情绪感知、人物识别、行为分析、语音-视觉对齐四个维度——这是上述算子链在真实数据集构建中的完整落地案例，也间接说明这组算子已具备生产可用性。
[不确定] 未披露 ASR 底层模型的具体选型与多语种支持范围；未提供转写质量评估或置信度过滤机制。

### [Foley-Omni](../models/Foley-Omni.md) ⚠️

对白转写以 [WORDS] 字段承载，但实现路径不常规——不使用独立 ASR，而是由 Gemini 2.5 Pro 在多模态标注中直接产出语音内容文本。这样做的好处是转写与存在性判别、与其他两字段的描述在同一次调用中完成，成本与流程都更简洁；代价是转写精度不如专用 ASR（Whisper 等），且论文未做转写质量评估。
【质量保障】[WORDS] 字段同样受 Bandit speech stem 的 −35 dB 能量门控约束——若分离出的语音轨能量不达标，说明画面中人物张嘴但实际无可闻语音（或语音被淹没），该字段被置空，可有效防止 ASR/VLM 在无语音处产生幻觉转写。
【说话人属性标注】[不确定] 论文未描述任何说话人身份 ID、性别、年龄、语言、口音、情绪的结构化标注。三字段 schema 中 [WORDS] 只承载语音内容文本，不含说话风格/音色描述。这意味着模型的音色控制能力主要依赖视觉条件（从说话人面部推断音色），而非文本属性标注——这与 VisualTTS 的任务设定一致，但也限制了对语音风格的显式可控性。
【间接证据】评测使用 WER 作为核心指标（V2ST-Bench 7.59、GRID 15.3、对照 Ground Truth 8.03），说明 [WORDS] 字段是逐字转写文本而非内容摘要，否则 WER 无法计算。

### [Goku](../models/Goku.md)

不适用/未涉及。数据流水线中无 ASR 转写环节，无说话人身份、语言、口音、情绪等属性标注。相反，Goku 对画面中的文字（含字幕）是**主动剔除**的（OCR 文字面积超 1%~2% 即丢弃片段），说明其数据取向刻意规避对白与字幕场景。

### [HunyuanVideo-Foley](../models/HunyuanVideo-Foley.md) ⚠️

本工作不涉及对白转写与说话人属性标注，且这是明确的设计边界而非疏漏：
【无 ASR】整条流水线中没有语音识别环节——不使用 Whisper 或任何 ASR 模型，不转写台词内容，不产生语音文本。
【speech detection 的用途仅是分类而非转写】流水线第 7 级的语音-音乐检测模型只做「这段音频里有没有人说话」的判别，输出是类别标签而非文字内容，用途是类别分布管理。
【无说话人属性】不标注说话人身份、性别、年龄、音色、口音、情绪、语速；不做说话人分离（diarization）；不做人脸检测（整条流水线无 RetinaFace 或任何人脸相关模型）。
【无唇同步检测】不使用 SyncNet、不计算 LSE-C/LSE-D——这是与 UniVerse-1、MOVA 等说话人视频生成工作最显著的流程差异。
【任务边界】模型的目标输出是 Foley 音效与环境音，不生成对白。理想行为是：面对有人说话的画面，生成该场景的环境音与动作音，而把对白留给专门的 TTS/配音环节。这符合真实影视后期制作中「拟音师、音效师、配音演员」的分工——本模型对标的是拟音师（Foley artist）这一职能，而非配音。
【实际表现的开放问题】论文未评测模型在含人物说话画面上的行为——是否会生成含糊的类语音声音（babble）、是否会正确地保持人声空缺，均无数据。这在实际应用中是重要的产品问题（生成的假语音会严重破坏可用性），但论文与评测基准均未覆盖。[不确定]

### [HunyuanVideo](../models/HunyuanVideo.md) ⚠️

不适用。无音轨处理，无 ASR 转写，无说话人身份/语言/口音/情绪标注。腾讯体系内相关能力在 HunyuanVideo-Avatar（音频驱动数字人）等独立模型中，不在本条目范围。[不确定]

### [InstructAV2AV](../models/InstructAV2AV.md) ⚠️

对白处理是本工作四类任务中最复杂的一支（对应 HF 的 clone_id / clone_voice / clone_id_voice 三个子集），处理链路完整但属性标注披露不足。

【转写环节】使用 ElevenLabs Scribe（2025）做 ASR，明确用途是「extract precise speech timestamps」——注意重点在精确时间戳而非仅文本内容。时间戳精度对本任务是刚需：编辑后的新台词必须在原语音的时间窗内替换，且新口型需与新语音逐帧对齐，时间边界不准会直接导致唇同步失败。

【说话人定位与画内性判定】TalkNet（Tao et al., 2021）做主动说话人检测（active speaker detection），定位画面中「正在说话的那个人」。与转写结果结合后施加一条硬性准入条件：只有当语音与画面中可见的说话人时间对齐时，该片段才被保留（clips retain speech only when temporally aligned with a visible on-screen speaker）。这一条同时排除了画外音、旁白、解说、后期配音、以及说话人不在画面内的场景，等价于在数据层面强制「所有语音样本都自带唇同步监督」。这是比事后用 SyncNet 分数过滤更彻底的做法——后者是在混杂数据中筛出同步的，前者是从源头只准入声画同源的。

【语音合成与身份/音色的解耦标注】HF 的三个子集命名揭示了一套隐含的属性维度体系：
  · clone_id —— 换视觉身份，保持原音色；
  · clone_voice —— 换音色，保持原视觉身份；
  · clone_id_voice —— 视觉身份与音色同时更换。
  这意味着数据构造时「视觉身份」与「听觉音色」被当作两个可独立操控的正交因子，并为三种组合分别构造了受控样本。这是合成数据独有的能力：现实中不可能采集到「同一段视频换了脸但声音没变」的真实素材，只有受控合成能提供这种解耦监督，让模型学会把身份与音色分开控制。这一点是本工作在「合成数据维度代表性」上最具说服力的实例。
  语音合成由 ElevenLabs 完成（该平台以音色克隆能力著称），推测 clone_voice 类样本用其 voice cloning 功能生成新音色、identity-preserving speech modification 类样本用其保持原音色仅改内容的能力生成——但论文未描述具体配置。

[不确定] 大量属性标注信息缺失：未见说话人身份 ID、性别、年龄、语言、口音、情绪的结构化标注字段；未说明 ASR 转写文本是否作为数据字段保留（从指令中的 <S>/<E> 标记看，目标台词文本确实进入了指令，但源台词转写是否保留未知）；未说明 ElevenLabs 的调用配置（音色克隆参数、语速控制、情感设定）；未做转写质量或合成语音质量的定量评估。评测侧使用 Sync-C / Sync-D 衡量唇同步，但未使用 WER 衡量语音内容准确性——这意味着「模型是否真的说出了指令要求的台词」这一最直接的语音编辑保真度指标缺失评测。

### [2026 其他音视频联合生成](../models/JAVG_2026_misc.md) ⚠️

【OmniCustom —— 最完整（对白转写 + 七项说话人属性）】
(1) 转写：「we used GLM-ASR to generate transcriptions for each 5s training audio clip」——用 GLM-ASR 为每个 5 秒训练音频片段生成转写。GLM-ASR 为智谱系语音识别模型，中文能力较强。
(2) 说话人属性（沿用 Ovi 体系）：age（年龄）、gender（性别）、accent（口音）+ vocal characteristics 四项（pitch 音高、prosody 韵律、emotion 情绪、speaking rate 语速），合计 7 项。
(3) 音色的隐式标注：其「同音色异内容」配对构造（前 4 秒参考音频 + 后 5 秒训练音频来自同一段视频、同一说话人）本质上提供了一种无需显式声纹标注的音色监督——同一 pair 内音色恒定、内容不同，天然构成音色解耦的对比学习信号。这比显式标注声纹 ID 成本低得多，是很聪明的数据构造。
(4) 评测集控制性别比 1:1，且含 30 位训练集外真人以验证零样本音色迁移。
【ALIVE —— 说话人归属校正是其独有贡献】
(1) 转写标注：<W> 标签包裹 verbatim speech content（逐字语音内容），说明有精确 ASR 转写；<I> 标签标记非语音声学事件。
(2) 声学画像：Subjects 字段中每个主体带 acoustic profiles（声学画像），即说话人音色属性与视觉外观绑定。
(3) SubjectID correction 五步管线（针对多人场景「谁在说话」的错误归属，是本批唯一系统处理该问题的工作）：
  ① TS-TalkNet 做目标说话人主动说话人检测（Target-Speaker TalkNet），生成人脸-声音对应矩阵（face-voice correspondence matrix）
  ② Qwen3-omni 解析 narration 的时间戳
  ③ majority voting（多数投票）建立「句子 → TrackID（人脸轨迹 ID）」的映射
  ④ CLIP + ArcFace 嵌入做视觉-语义匹配
  ⑤ 用校正后的标签重写文本（text rewriting with corrected labels）
未通过高置信阈值的视频被直接过滤，得到「simplified」但高质量的初始数据集。这个管线解决的是纯视频模型完全不会遇到的问题：多人画面中把台词挂到错误的人脸上，会让模型学到错误的「脸-声」绑定，直接破坏多人对话场景的唇同步与音色一致性。阈值数值未公开[不确定]。
【StreamChar】使用 ASR 时间戳作为监督：「ground-truth end indices derived from ASR timestamps」用于 progress-aware pointer 的 smooth L1 损失——即把「台词念到第几个字」这一进度信息作为显式监督信号，是 transcript fidelity（台词保真）的数据基础。ASR 工具未点名[不确定]；说话人属性标注未描述[不确定]。
【NAVA】Timbre-in-Context Conditioning 用「reference utterance」（参考话语）经 timbre encoder（实现上为 ReDimNet 说话人嵌入器）编码，把参考音色线索关联到对应的 speech spans（语音片段）——即音色不是全局条件而是按语音段绑定，支持多说话人各自音色。但论文未说明标注协议（如何确定 speech span 边界、是否用 diarization）[不确定]；ASR 方法未详述[不确定]。其 YAMNet 五类标签中区分 single-speaker / multi-speaker speech，可视为一种粗粒度的说话人数标注。
【CCL】报告 WER 指标说明其评估了语音可懂度，隐含有转写-生成对照，但数据侧的转写流程未描述[不确定]。
【Baton】未描述[不确定]。

### [音视频联合生成基线合集](../models/JavisDiT_baselines.md) ⚠️

【MM-Diffusion / AV-DiT】训练数据完全不含人类语音（自然环境音与器乐音乐），无转写、无说话人属性标注，该维度不适用。
【JavisDiT / JavisDiT++】反向操作——不但不做转写，反而主动用 FunASR 检测并剔除含人类语音的视频，把语音从音视频训练数据中清除出去。这是一个刻意的能力取舍：放弃唇同步与对白生成，换取在环境音/音效事件对齐上的专注。FunASR 在此扮演的是「语音检测器」而非「转写器」的角色。音频预训练阶段虽包含语音类数据，但未做转写标注。
【Harmony（有完整转写）】Gemini 自动标注明确产出 ASR 转写文本（transcripts），并作为语音条件通道送入模型；语音数据来自 Emilia（TTS 语料，本身自带转写）、OpenHumanVid、SpeakerVid（双人交互数据，天然含多说话人场景）。说话人属性方面，Harmony 有一个独立于转写的机制——阶段二「音色解耦微调（Timbre Disentanglement Finetuning）」，用同一说话人的跨话语错配参考-目标配对（cross-utterance mismatched reference/target；环境音则用同一片段的非重叠段），训练模型从 1–3 秒参考音频中提取音色特征而不泄漏内容。这实际上是用「数据配对方式」而非「属性标注」来解耦音色，比显式标注年龄/性别/口音的做法更轻量。但年龄、性别、口音、情绪等显式属性标签未见标注[不确定]，说话人日志（diarization）与说话人 ID 也未提及[不确定]。
【UniAVGen】评测端使用 Whisper 计算 WER，说明转写在评测环节存在；训练数据侧是否做 ASR 转写标注未说明[不确定]。模型显式优化「音色一致性（timbre consistency）」与「情绪一致性（emotion consistency）」两项指标，暗示训练中存在音色与情绪的条件通道（很可能通过参考音频而非文本属性标签实现），但具体标注方式未披露[不确定]。

### [可灵 3.0 / 可灵视频 3.0 Omni](../models/Kling_30_Omni.md) ⚠️

[不确定] 未公开ASR转写与说话人属性标注的具体方案。能力反推：模型支持5语种+口音的口型同步、多角色（3+）不同音色共指、以及“上传参考素材即可绑定特定主体的视觉特征与音色”，这要求训练数据具备逐句对白转写、时间戳对齐、说话人分离（diarization）与说话人身份/音色ID、语言与口音标签、情绪标签等结构化标注。KlingAvatar 2.0 明确从播客、访谈、多角色电视剧构建多说话人对话数据，并对多角色视频建立自动化标注 pipeline（YOLO 人物检测 + DWPose 关键点 + SAM2 分割与时序跟踪，并与帧级检测结果交叉校验），用于把“哪个人在说话”与音轨绑定——这是说话人-画面对应关系标注的公开证据。[不确定：使用的ASR模型、情绪标签体系]

### [LTX-2](../models/LTX-2.md)

明确执行且是 caption schema 的显式组成部分：caption 包含对白的「精确转写（precise transcriptions of dialogue）」，并同时标注说话人身份（speaker）、语言（language）与口音（accent）三类属性。这三项属性标注直接支撑了模型的多语种唇同步、说话人区分与口音可控能力（论文称模型可合成「不仅与唇动同步，且在节奏、口音与情绪基调上都自然」的语音）。
值得注意的对比：情绪（emotion）不作为标注字段——打标准则明确要求「不做情绪化解读」，只客观描述所见所闻；情绪表达被期望通过对白内容、生理动作线索与音色描述间接习得。官方 prompting 指南建议用户把台词拆成短句、在句间插入表演指示（acting directions），并用生理线索而非情绪标签。
未披露：ASR 使用的具体模型、说话人分离/聚类（diarization）方法、语言与口音识别模型、转写词错误率或质量把控手段。多说话人场景下模型仍会出现台词错配给角色的问题（论文局限性），暗示说话人-台词绑定的标注或建模仍不够强。

### [LongCat-Video](../models/LongCat-Video.md) ⚠️

基础版不适用。
Avatar 1.5 涉及说话人处理，但未采用 ASR 转写路线——报告未指定任何转写模型，也无逐字文本标注。其说话人相关处理集中在「谁在说话」而非「说了什么」：使用 TalkNet 与 UniTalk 两个主动说话人检测（Active Speaker Detection）模型；多人场景用 YOLOv6 检测 + ByteTrack 做人物跟踪以关联身份，并显式剔除多人同时说话的时间区间（excluding intervals with concurrent speaker activity），避免音画归属歧义。说话人属性标注方面，做了情绪维度：定义 6 类情绪，用 EmotiEffLib 以置信度 s>0.7 筛选，并对含合成内容、主体多于两人、身份切换、主体占画面比例过小的样本一律标 null。语言/口音属性未做标注。[不确定：是否有未披露的 ASR 环节]

### [MOVA](../models/MOVA.md)

对白转写与说话人属性标注是 MOVA 数据体系的核心：
【转写】Qwen3-Omni-Instruct 执行 ASR，输出逐字转写（verbatim）。三条法则约束：LAW OF LANGUAGE FIDELITY（保留原始语言、严禁翻译）；LAW OF SPEECH DYNAMICS（当新的说话人 / 语言 / 语调开始时，创建新的事件条目）；LAW OF SILENCE（无语音时 speech_description 输出 null）。不清晰片段标记为 [inaudible]。
【说话人属性】通过 Qwen3-Omni-Captioner 的音频描述以自然语言形式覆盖：说话人数量（如 “two distinct voices” / “a group of overlapping speakers”）、说话人切换点、音色特征（如 high-pitched / gruff / mature male）、口音（示例中为 “a General American accent”）、语气与语速（calm/authoritative/professional；measured/declarative；rhythmic evenly paced cadence）、录音环境（studio-clean、slight reverb）。这些属性不是枚举字段，而是描述性文本，在融合阶段被绑定到画面中的具体人物。
【说话人身份绑定】融合 prompt 要求把语音以引语形式嵌入并与视觉主体锚定（如“the teenager in the corner”“the gray-haired woman”），实现“视觉身份—音色—对白内容”的三元绑定。
【评测侧的对应设施】用 MOSS Transcribe Diarize（同团队 2026 年发布的转写+说话人分离模型，arXiv:2601.01554）对生成结果做说话人分离（显式说话人标签 [S01]、[S02]）与 ASR，进而计算 cpCER（concatenated minimum permutation CER），评估说话人身份与对白内容是否被正确反映。MOVA-720p 达到最佳 cpCER 0.149。
【已知缺陷】论文 Limitations 明确指出：diarization 错误与不完善的 active-speaker 标签会传播进训练数据，导致多说话人场景下的口型-音频错配与时序漂移；改进方向包括更强的主动说话人检测、跨模态说话人跟踪与更好的噪声片段过滤。

### [合并条目：① Mochi 1](../models/Mochi_MAGI_Motif.md)

不适用。三者均无音频处理链路，未做 ASR 转写、说话人身份/语言/口音/情绪标注。需要注意的是，Motif-Video 2B 的 caption schema 中确有「verbatim text transcription（文字逐字转录）」与「on-screen text（屏幕文字）」字段，MAGI-1 也有专门的字幕检测逻辑，但这些指的都是画面上的视觉文字（OCR），与语音转写无关；而且在 Motif 的管线中，检出持久文字区域的后果是被裁剪或整段剔除，并非作为标注保留。

### [Movie Gen](../models/Movie_Gen.md)

无。Movie Gen Audio 不做 ASR 转写，不标注说话人身份、语言、口音或情绪——因为模型有意不生成语音：diegetic speech 在没有台词脚本、且生成视频存在伪影时难以生成；non-diegetic speech（旁白）被认为可由TTS在给定脚本时补齐。对语音的全部处理仅限于：用 AED 模型（AudioSet 本体的 speech / singing 子类）判定样本中是否存在人声与歌唱，输出经预设后验阈值二值化后写入caption作为控制标签；以及在数据分桶时把 voice 作为三大音频类型之一参与 CAVTP 阈值判定。微调用的 cinematic split 更是直接把含人声（vocals）的片段整体排除。论文在局限性中明确承认：模型「由于设计选择目前不支持人声生成」。

### [NVIDIA NeMo Curator](../models/NeMo_Curator.md) ⚠️

视频 pipeline 中无对白转写（不处理音轨）。
音频 pipeline 具备部分能力但与视频割裂：(1) ASR 转写——使用 NVIDIA NeMo ASR 模型系列生成转写文本；(2) 转写质量评估——以 WER（词错误率）与 CER（字错误率）作为质量指标并据此过滤（WER-filtering），这是该框架在语音数据侧最明确的质量把控手段；(3) 说话人分离——26.02 起支持 streaming Sortformer 做 speaker diarization 与 VAD（语音活动检测），并有 long-form audio cutting 教程；(4) 26.07 新增音频增强 stage：tagging、SQUIM 客观质量指标、带宽估计（bandwidth estimation）、标点准备（punctuation preparation）、可选的二次 ASR 打分。
【缺失】说话人身份属性标注、语种识别、口音标注、情绪标注均无内置 stage；WER/CER 的默认阈值未在文档中给出。这些能力对多语种唇同步数据构建是必需的，需自建。[不确定]

### [OmniHuman 数据集 + OHBench](../models/OmniHuman.md) ⚠️

对白与说话人属性是本数据集标注最厚的一块，也是其相对 OpenHumanVid 等前作的核心增量：
【转写】FunASR-Nano 执行 ASR，产出语音文本，并附带说话人 ID 与时间戳（speaker ID and timestamps）。转写结果有两重用途：一是作为标注内容本身，二是作为校验 MLLM caption 的真值（编辑距离校验）。
【说话人分离】3D-Speaker 做 diarization，识别出 M 个活跃语音区间及各自的说话人索引。这是双人场景标注的前提。
【说话人-画面归属（本工作的关键步骤）】用 SyncNet 解析音视觉归属，通过贪心匹配（greedy matching）把每段音频指派给响应最高的视觉 ID 轨迹；再用 ArcFace 人脸嵌入完成身份确认（嵌入相似度 > 0.55 判为同一人，跟踪丢失容忍跨度 5 帧）。这一步把「音频中的说话人 m」与「画面中的人物 i」绑定，是双人对话数据可用的必要条件——没有它，模型无法知道该让谁的嘴动。Table 1 显示，此前具备 speaker annotation 的对照数据集只有 SpeakerVid-5M 一家。
【说话人属性标注】
- 情绪（emotion）：逐说话人、逐语音段标注，同时出现在个体层（语音转写附带情绪）与交互层（人-人交互标签含说话人 ID 与情绪）；
- 语言（language）：sample_json 的 speech 字段含 language 标签；
- 时序（timing）：语音段的起止时间戳；
- 语音质量（speech quality）：个体层标注中单列的语音质量标注；
- 唇同步质量：帧级标注中的 lip-sync quality assessment。
【未标注】说话人的性别、年龄、人种未见标注；音色/声纹的文本化描述未见（与 MOVA 的详细音色描述不同）；口音未标注；语速与音量未见逐说话人标注（音乐有相对音量、语音无）。[不确定]
【多说话人支持】完整支持，且是数据集的设计重心之一：128 条双人评测视频、双人交互标签、双人 ID 一致性评测指标（IC*，用 ArcFace 余弦相似度）、倾听者真实度指标（LR，评估音视觉归属准确性与倾听者表情自然度）。「倾听者」这一维度尤其少见——大多数说话人数据集只关心说话的人，OmniHuman 显式评测了不说话的那个人是否表现自然，这直接对应双人对话生成中最常见的失败模式（两个人同时张嘴说话）。
【评测侧指标】发音准确度用 SenseVoice 计算 WER；语音真实度用 DNSMOS 的 OVRL 分数；唇同步用 SyncNet。

### [Open-Sora 系列](../models/Open-Sora.md)

不适用。数据 pipeline 完全不处理音轨，无 ASR 转写、无说话人身份/语言/口音/情绪标注。

### [Ovi](../models/Ovi.md) ⚠️

【转写】由音频理解型 MLLM 直接从完整音轨产出台词文本，写入 <S>...<E> 标签，并按时序穿插在视觉描述中；纯音频数据同样产出转写，无语音时该字段留空。未提及使用独立 ASR（Whisper 等）或做转写质量校验/WER 过滤[不确定]。
【说话人属性】对含语音片段，音频描述被明确要求强调说话人相关的声学属性：年龄、性别、口音、音高、韵律、情绪、语速（age, gender, accent, pitch, prosody, emotion, speaking rate）。这套属性集使模型在推理时可通过 prompt 直接调控音色与情绪，Ovi 1.1 进一步强化为「基于 prompt 的情绪指令标签」。
【说话人身份/分离】未提及说话人日志（diarization）、说话人 ID 标注、多说话人分离标注，也未提及说话人 embedding 或参考音色条件（README Todo 中「Reference voice condition」仍未实现），因此多人对话场景中「哪句话属于哪个人」缺乏显式监督[不确定]。
【语言标注】口音被标注，但语种是否单独标注未说明[不确定]。

### [Script-a-Video](../models/Script-a-Video.md) ⚠️

对白转写在 MTSS 中有明确的结构化承载，但说话人属性标注偏薄：
【转写】Event 流中 type 为 dialogue 的事件设有 "line" 字段，记录 verbatim text（逐字台词文本）。论文未说明这些台词是由 ASR 转写还是由 Gemini-2.5-Pro 直接从音频中听写生成——考虑到整套标注均由 Gemini-2.5-Pro 一次性产出，后者可能性更高，即无独立的 Whisper 类 ASR 环节。[不确定]
【说话人身份】通过 "speaker" 字段实现，值为 Reference 流中的实体 ID（如 PERSON_1）。这是一种关系绑定而非属性描述——说话人身份直接指向实体库中带完整外观锚点（服装/配饰/发型）的条目，因此「声音」与「长相」在数据结构层面被硬绑定。这一设计对训练音视频联合生成模型辨识「谁在说话」尤为关键。
【说话人属性】未设独立的结构化字段。性别、年龄、音色、语种、口音、语速均无对应字段。最接近的是 Event 的 "description" 字段，论文描述其用于捕捉「nuanced semantics like emotional shifts or vocal techniques」（细腻语义如情绪起伏或发声技巧）——即情绪与发声方式以自由文本形式记录，但非枚举标签，不可直接用于配比统计或条件控制。
【多说话人处理】通过并发事件拆分机制天然支持：同时发生的多个音源被拆为并行的 event 条目，各自带独立的 speaker、time_range 与 line，因此多人对话场景可被完整表达而不会混作一团。这是相对于「一段话描述整个音景」类方案的明确优势。
【生成侧的转写准确性验证】WER 用 Whisper-large-v3 转写生成音频后、用 jiwer 对照真值文本计算（CJK 文本用 jieba 分词）。结果：LTX-2-AV 基线单镜头 1.64 / 多镜头 0.84 → LTX-2-AV-MTSS 单镜头 0.78 / 多镜头 0.13 → Ours(Full) 多镜头 0.19。论文指出 Event 流提供的显式音频事件描述把语音与音效生成从「近乎随机」（near-random）转变为「语义准确」。
【音质指标】Audio Quality 用 UTMOS（轻量语音质量 MOS 预测器）评估：基线 4.12-4.18，MTSS 版本 4.60，完整方案 4.68。

### [Seedance 2.0 与 Seedance 1.5 pro](../models/Seedance_20_Seedance_15_pro.md) ⚠️

[不确定] 报告未明示 ASR 转写与说话人属性标注流程。间接证据强烈支持其存在：Seedance 1.5 pro 具备精准的多语种与方言唇形同步、能捕捉各语言独特的语音韵律与情绪张力；SeedVideoBench 1.5 的「人声属性」标签体系明确包含音色（timbre）、口音（accent）、情绪基调（emotional tone）三类；SeedVideoBench 2.0 覆盖中文方言/口音、多人对话、少数民族语言、画外音等 17 类。这些能力必须以对白转写 + 说话人身份/语言/口音/情绪的标注为数据基础，但论文未描述具体做法（是否用自研 ASR、说话人分离 diarization 等）。

### [SkyReels 系列](../models/SkyReels.md) ⚠️

【转写】SkyReels-V4 对四类音频中的「语音（speech）」与「歌唱（singing）」使用 Whisper 进行内容转写，转写文本填入结构化 caption 的 <dialogue> 与 <singing> token 内；音效与音乐类不转写，改由 Qwen3-Omni 生成描述性 caption。
【说话人属性】论文未提及说话人分离（diarization）、说话人身份标注、情绪标注、口音标注或音色描述等属性字段——这是与 LTX-2（显式标注 speaker/language/accent）相比的明显缺口。多说话人场景下的台词-角色绑定如何处理未说明。
【语言】多语种能力主要通过合成 TTS 数据补齐（中/英/日/韩/德/法等），而非通过对真实数据做语种标注与均衡。Whisper 具备语种识别能力，但论文未说明是否把语种作为显式字段落库。[不确定]

### [Sora 2](../models/Sora_2.md) ⚠️

训练数据侧未披露。间接证据：安全栈明确将「audio transcripts（音频转写）」作为独立通道送入安全分类器，说明 OpenAI 具备并常规使用视频音轨的ASR转写能力，该能力用于训练数据打标是自然延伸。说话人身份/语言/口音/情绪等属性标注，无任何披露。产品侧的 cameo 功能要求用户录制一次性的视频+音频样本以完成身份验证并绑定其形象与声音，说明存在「说话人身份-音色-形象」的绑定表征，但这属于个性化/条件注入机制而非训练数据标注schema。[不确定]

### [Step-Video-T2V](../models/Step-Video-T2V.md) ⚠️

不适用。数据 pipeline 不处理音轨，无 ASR 转写，无说话人身份/语言/口音/情绪标注。阶跃星辰体系内这类能力在独立的 Step-Audio 系列语音模型中（该模型支持情绪、方言、语种、歌声等属性的语音生成），但与 Step-Video-T2V 的训练数据无任何交集。[不确定]

### [UniTalking](../models/UniTalking.md) ⚠️

【转写】使用 Whisper-V3 执行 ASR，产出语音的文本内容。转写结果作为音频 caption 的组成部分参与文本条件构建。论文未说明：是否保留原始语种（还是统一翻译为英文）、是否做标点与数字规整、是否过滤转写置信度低的样本、是否处理转写失败的片段、是否保留时间戳（word-level timestamp）。[不确定]
【说话人属性标注：显式标注完全缺失】未标注说话人身份、性别、年龄、人种、情绪、语速、音量，未做说话人分离（diarization），未做说话人聚类或身份去重。数据集中的唯一说话人数量亦未报告。
【音色的处理方式：不标注、改为音频条件】这是 UniTalking 与 MOVA 路线的关键分野。MOVA 用自然语言详细描述音色、口音、语气、录音环境，把说话人属性显式化为文本；UniTalking 则完全不做文本化描述，而是把音色作为一路独立的音频条件（c_audio）直接注入——由冻结的 MMAudio VAE 编码 3–5 秒参考音频（latent 长度固定 257），经专设的 KV 投影层做 cross-attention。这是「用音频表示音频」而非「用文本描述音频」，避开了音色难以用语言精确描述的固有困难。
【为此付出的数据代价】由于源数据本身不含「参考音频」这一配对信号，团队必须人工构造（见 synthetic_data_synthesis），这是整条流水线中成本最高、也最有风险的一步。
【音色能力的实测水平】TR2AV 音色相似度：英文 0.703、中文 0.662。与专业 TTS 相比处于中下水平——低于 Qwen3-Omni（0.773/0.772）与 MiniMax（0.756/0.780），仅与 ElevenLabs（0.613/0.677）相当。论文坦承「难以超越 SOTA 大型音频模型的性能」，仅主张达到了「一定程度的语音克隆能力」。
【多说话人】不支持。Future Work 明确承认框架尚不支持多人参考生成（类似 Sora2 的 Cameo 功能）。训练数据也未见多说话人对话样本的构造。
【评测指标】WER（Seed-TTS test-en，0.038）衡量文本忠实度，Speaker Similarity 衡量音色克隆，Sync-C/Sync-D 衡量唇同步。无 cpCER 类的多说话人指标。

### [UniVerse-1](../models/UniVerse-1.md) ⚠️

【转写】使用 Whisper 做 ASR，产出逐字语音内容；再由 QWen2.5-Omni 输出「verified speech content」（核验后的语音内容），即多模态模型对 ASR 结果做了一层核验/规整。论文未说明核验的具体规则、是否处理不可辨识片段、是否保留原始语种不做翻译。[不确定]
【说话人属性标注】基本缺失。论文未标注说话人身份、性别、年龄、音色、口音、情绪、语速等任何说话人属性，也未做说话人分离（diarization）。
【说话人建模的架构层决策】更关键的是，UniVerse-1 主动移除了 Ace-step 原有的 speaker encoder，明确目的是「让模型泛化到超越特定说话人的生成」（generalize the model beyond speaker-specific generation）。这意味着音色不由显式的说话人 embedding 控制，而是由参考图像与文本条件隐式决定——模型卡描述其能力为「生成与角色形象相匹配音色的语音」（character-matched voice timbre）。这是一条与 MOVA（用自然语言详细描述音色、口音、语气、录音环境，并在融合阶段绑定到画面人物）完全不同的技术路线：UniVerse-1 靠架构简化 + 视觉条件隐式建模，MOVA 靠标注显式化。
【多说话人场景】论文未讨论多说话人、说话人切换或对话轮次的处理，训练数据的单镜头约 5 秒窗口也基本不承载多轮对话。
【评测】用 WER（词错误率，UniVerse-1 为 0.18）衡量生成语音的可懂度与文本忠实度，用 LSE-C（1.34）衡量唇同步——但没有 MOVA 式的 cpCER（区分说话人的拼接最小置换字错误率），因为不涉及说话人身份评估。

### [Unison](../models/Unison.md) ⚠️

【转写通道确实存在】模型输入明确包含 transcription text（语音转写文本）作为独立条件通道，与 caption text 并列，且其平均池化语义向量参与 SCG 门控计算。因此每条含语音的训练样本必须配有转写文本。
【转写来源未披露】论文未说明训练数据的转写由何而来——是沿用上游数据集自带转写，还是用 Whisper 等 ASR 离线生成。Whisper-large-v3 在论文中仅出现于评测环节（计算生成语音的 WER），未声明用于训练数据转写。[不确定]
【说话人属性标注：完全缺失】未标注说话人身份、性别、年龄、音色、口音、情绪、语速中的任何一项；未做说话人分离（diarization）；未做多说话人场景的处理说明。这是 Unison 与 MOVA（用自然语言详细描述音色/口音/语气/录音环境）之间的显著差距。
【人脸数量检测暗示的多说话人意识】lip-filtering 算子明确「检测人脸的数量与位置」（detects the number and location of faces），说明 pipeline 知晓画面中有几张脸。这为多说话人处理留下了接口，但论文未说明检测到多张人脸时的策略——是剔除、是选主说话人、还是逐一核验后保留。这是一处有价值但未展开的设计细节。[不确定]
【音色控制机制未说明】模型如何决定生成语音的音色（是否由参考图像隐式决定、是否有 speaker embedding、是否随机）完全无说明。作为对比，UniVerse-1 明确移除了 speaker encoder 以求泛化，MOVA 用文本描述显式指定音色，Unison 在这一维度上沉默。[不确定]
【评测】WER 是 Unison 的最强项之一：TI2AV 设定下 0.22（全场最优，优于 LTX-2 的 0.25、MOVA 的 0.29），T2AV 设定下 0.09（同样最优，优于 LTX-2 的 0.11）。这一优势可直接归因于其语音-音效双流解耦——语音流不受音效干扰，转写可懂度自然提升。但因无 cpCER 类指标，无法评估多说话人场景下的表现。

### [Veo 3 / Veo 3.1](../models/Veo_3_Veo_31.md) ⚠️

[不确定] 未披露任何 ASR 转写流程或说话人属性标注体系。反推线索：模型可按 prompt 中引号内文本精确生成对应对白并保持唇同步，说明训练数据必然包含逐字对白转写（很可能由 Gemini 的音频理解能力或 Google 内部 ASR 完成），且转写与视频时间轴对齐。但官方在评估 deepfake 风险时明确指出模型「在语音方面可控性差」、不提供口音与音色的细粒度控制，强烈暗示说话人身份、音色、口音、情绪等属性未被系统性标注为独立条件字段。

### [Vidu S1](../models/Vidu_S1.md) ⚠️

有较完整的说话人属性标注链路：从原始音频中提取语音成分 → VAD（voice activity detection）+ ASD（active speaker detection）标注每个语音段的时间戳及其关联说话人 → 依据说话人与画面主体的对应关系将每段标为 onscreen / offscreen / overlap 三类。caption 中含 dialogue 字段。但论文未明确说明是否做了完整 ASR 文本转写，也未标注说话人的语种、口音、年龄等属性 [部分不确定]。评测基准 Vidu-StreamBench 中提到覆盖多样的说话人属性与情绪。

### [主流视频预训练数据集合并调研：Panda-70M、InternVid、Koala…](../models/pretraining_datasets.md)

不适用。七者均无ASR转写发布，无说话人身份/语言/口音/情绪标注。两处相关但不构成对白标注的情况需澄清：(1) **Panda-70M使用了英文字幕（含YouTube自动字幕）但仅作为教师模型的文本侧输入**（video2dataset配置writesubtitles:True, subtitleslangs:['en'], writeautomaticsub:True），字幕落在每clip的JSON元数据中，不作为数据集字段发布，且不含说话人属性；(2) **InternVid内部采集到11种语言的ASR字幕**（论文图16示例英/中/韩/德），但既未用于生成caption也未随数据集发布——这是一处被采集却被丢弃的多语资源。此外Panda-70M出于隐私考虑**用NLTK把caption中所有人名替换为「person」**，等于主动抹除了说话人身份信息。

## 几何与结构化标注（相机参数、深度、3D point tracks、动作标注、显式状态）

`geometric_structured_annotation` · 详细程度: brief

### [Allegro](../models/Allegro.md)

仅有轻量的相机运动语义标注：caption 中以「Camera [MOTION_PATTERN]」的显式句式标注运镜模式（由微调 Aria 生成的语义描述，非数值参数）。
未做相机内外参估计、深度图、3D point tracks、动作/骨骼标注或显式物体状态标注。运动信息在过滤侧仅体现为 UniMatch 的标量光流幅度（[1.0, 100]），未保留为稠密光流条件或结构化字段。

### [Apollo](../models/Apollo.md)

论文未使用任何几何或结构化的视觉标注：无相机参数（内外参、轨迹）、无深度图、无 3D point tracks、无骨架/关键点/动作标注、无显式物理状态标注、无 bounding box 或分割掩码。唯一接近结构化的标注是音频侧的说话人属性 meta 字段（性别、年龄）。位置信息完全由 MixD-RoPE 在模型内部以位置编码形式隐式处理（视频 3 维 + 音频 1 维），而非作为数据标注引入。「camera stability（镜头稳定性）」虽在过滤维度中出现，但那是一个用于筛选的标量质量分，不构成相机运动的结构化标注。

### [CineDance / CineDance-1M](../models/CineDance.md)

【已有的结构化标注】镜头级五维属性中包含较强的摄影语法信息：scale（景别，如特写/中景/远景）、angle（机位角度）、movement（运镜方式，如推拉摇移）、narrative function（叙事功能）、duration category（时长类别）；以及 shot transition type（转场类型，如硬切/溶解）。训练时以 [SHOT i | scene sᵢ | camera κᵢ] 的形式把 camera 参数作为独立条件项注入。
【场景与身份的结构化】全局场景表 ⟨sceneₘ⟩ 与全局角色表 ⟨charₙ⟩ 构成一套显式的状态标注体系，使跨镜头的身份/场景延续可被显式建模与评测。
【缺失维度】未提供数值型相机外参/内参（camera pose、焦距等）、深度图、3D point tracks、光流场、人体姿态或动作骨架等几何标注。相机信息是语言化的定性描述（κᵢ）而非数值参数。
【评测侧的几何替代】CineBench 用 ArcFace（人脸聚类）与 DINOv2（视觉特征余弦相似度）做一致性判定，属于表征级而非几何级。

### [CogVideoX](../models/CogVideoX.md)

基本没有显式的几何或结构化标注：论文未使用相机内外参、深度图、3D point tracks、光流场（光流仅作为标量分数用于过滤，不作为标注保留）、分割掩码或动作类别标签。
唯一的结构化要素是隐式的语言标注：GPT-4 摘要 prompt 明确要求 caption 中包含「camera movements（镜头运动）」，因此镜头运动信息以自然语言形式嵌在 caption 里，而非作为独立的分类字段或标签体系（对比 Movie Gen 的 16 类相机运动分类器 + 前缀拼接）。
模型侧的「结构化」主要体现在位置编码而非数据标注：3D RoPE 显式建模文本-时间-空间三个维度的位置关系，Multi-Resolution Frame Pack 在 batch 内携带各样本的形状信息。

### [Cosmos-Predict2.5](../models/Cosmos-Predict25.md)

几何与结构化标注是该工作区别于通用视频生成模型的显著特征，因为面向 Physical AI 与机器人/驾驶仿真：
【人体关键点】Human Dynamics 域用 YOLOX（Ge et al., 2021）做人体检测、RTMPose（Jiang et al., 2023）做 full-body keypoints（全身关键点）与 facial landmark（面部关键点）估计；这些几何量被用作数据准入的硬性判据：人物出现帧占比 >40%、任一帧可见人数 ≤8、至少一人占画面面积 ≥3%。
【多相机标定与视角结构】驾驶数据为 7 路同步相机（front-wide、front-tele、front-left/left、front-right/right、rear、rear-left、rear-right，不同段落命名略有差异），30 FPS、20 秒；每视角在 latent 通道维拼接一个 7 维的 per-view 可学习 embedding，并对每个视角单独构造 3D-factorized RoPE，即视角身份是显式编码的结构化信息。
【驾驶结构化控制信号「world scenario map」】Cosmos-Transfer2.5-2B/auto/multiview 的控制输入由 HD 高精地图元素与动态物体投影到七个相机视角构成，包含车道线、道路标线、杆件、道路边界、交通信号灯（含灯态）、交通标志，可表达含立交桥在内的复杂道路拓扑；动态物体表示为 3D bounding box 立方体，按粗粒度类别本体（truck / vehicle / pedestrian 等）色彩编码，并用明暗区分正面与背面。这是全文最重的结构化标注体系。
【机器人域的动作与元数据】action-conditioned 变体以 action 作为条件输入；caption 侧强制标注运动类型（线性/旋转）、涉及部件（臂/腕/夹爪）、相机运动；并注入 GR00T 的人工成功评级、Bridge 的 step-level 指令等结构化元数据。策略学习实验中还有末端执行器位姿与夹爪指令的动作 chunk（8 步 horizon、10 FPS 采样）与关节状态。
【分割与检测（下游增强用）】机器人策略数据增强中用 Grounding DINO + SAMv2 做机器人像素的检测与分割，以实现「edge 控制作用于全图、blur 控制仅作用于机器人像素」的分区控制；Cosmos-Transfer2.5 的通用控制网支持 edge、blur、segmentation、depth 四类控制模态。
【未涉及】通用预训练语料未做深度图、3D point tracks、相机外参轨迹等标注（这些只出现在控制网的控制信号与驾驶专有数据中）。

### [Data-Juicer 2.0](../models/Data-Juicer.md) ⚠️

这是 Data-Juicer 在2026年投入最集中、也是其相对其他数据处理框架最独特的方向——由具身智能（Embodied AI / VLA）需求驱动，v1.4.5、v1.5.0、v1.5.3 连续三个版本大量新增几何与结构化标注算子。对视频生成而言，这批算子恰好对应 SceneScribe-1M、SpatialVID 等「文本 caption 之外的第二类标注范式」。
【相机参数标注——提供三种实现】
  · video_camera_calibration_deepcalib_mapper —— 基于 DeepCalib 计算静态相机的内参与视场角（FOV）。
  · video_camera_calibration_droidcalib_mapper —— 基于 DroidCalib 从视频提取相机内参。
  · video_camera_calibration_moge_mapper —— 基于 MoGe-2 计算内参与 FOV。
  · video_undistort_mapper —— 用已求得的内参与畸变系数对原始视频做去畸变校正。
【相机位姿/轨迹】
  · video_camera_pose_megasam_mapper —— 「结合 MegaSaM 与 MoGe-2 提取相机位姿」。MegaSaM 是2025年的动态场景 SLAM 方法，可在含运动物体的野外视频上估计相机轨迹——这正是视频生成中「镜头运动」条件标注所需的核心能力。
【深度】
  · video_depth_estimation_mapper —— 对视频做深度估计，产出稠密深度图序列。
【人体与手部——动作标注链】
  · video_whole_body_pose_estimation_mapper —— 提取身体、手部、脚部、面部关键点的2D全身姿态估计。
  · video_hand_reconstruction_hawor_mapper —— 基于 HaWoR + MoGe-2 的手部重建。
  · video_hand_reconstruction_mapper —— 基于 WiLoR 模型的手部定位与重建。
  · video_hand_motion_smooth_mapper —— 对世界坐标系下的手部运动做平滑并剔除离群点。
  · video_hand_action_compute_mapper —— 「从手部重建结果与相机位姿计算 7-DoF 动作与 8 维状态」。这是明确的「显式状态」标注——把视频转化为机器人可用的动作-状态序列，是 VLA 训练数据的标准格式。
  · video_atomic_action_segment_mapper —— 把统一的手部轨迹切分为原子动作片段，即动作级的时序分割标注。
  · video_trajectory_overlay_mapper —— 「采样帧并叠加手部轨迹，准备 VLM 可读的帧」。把几何轨迹可视化叠加到画面上再喂给 VLM，是几何标注与语义标注的桥接技巧。
  · video_clip_reassembly_mapper —— 把重叠 clip 上的手部动作结果重组回长视频时间轴。
【物体级】
  · video_object_segmenting_mapper —— 基于 YOLOE + SAM2 的文本引导语义分割，产出全视频的物体掩码序列。
【评价】这套算子链使 DJ 成为目前开源框架中几何标注能力最完整的一个：相机内参、畸变、位姿轨迹、深度、全身/手部姿态、7-DoF 动作、8维状态、原子动作分割、物体分割全部具备，且大量采用2025–2026年的最新模型（MegaSaM、MoGe-2、HaWoR、WiLoR、SAM2、YOLOE）。虽然设计初衷是具身智能，但对需要相机控制、深度条件、动作条件的可控视频生成而言可直接复用。
[不确定] 未见 3D point tracks（如 CoTracker/SpatialTracker 式的长时序点轨迹）专用算子；这批算子的处理吞吐、成本与在大规模视频生成语料上的实战效果均未公开。

### [Foley-Omni](../models/Foley-Omni.md) ⚠️

[不确定] 论文未涉及任何几何或3D结构化标注：无相机参数（内外参、轨迹）、无深度图、无3D point tracks、无人体姿态或动作标注、无显式状态标注。
最接近「结构化视觉信息」的是 Synchformer 抽取的时序同步特征与 CLIP 的场景语义特征，但这些是隐式的神经网络表征而非可解释的几何标注。motion score 是唯一的显式运动量化，且仅用于过滤阈值判断，不作为训练标注保留。
这一缺位对本任务影响不大：音频生成不需要精确的3D几何理解，声画对齐主要依赖2D时序事件线索。但对更精细的空间音频生成（如双耳/环绕声、声源方位随镜头运动变化）而言，缺乏相机与3D标注会成为瓶颈——本文生成的是单声道/立体声混合音轨，未涉及空间音频。

### [Goku](../models/Goku.md)

结构化标注非常轻量，仅两项：
(1) **相机运动**——不通过几何估计获得，而是由 Tarsier2 在生成视频 caption 时以自然语言形式隐式描述（zoom in、pan right 等），属于「语言化的镜头标签」而非参数化相机轨迹。
(2) **运动强度**——RAFT 光流得到的标量 motion score，既作过滤阈值也作 caption 中的数值条件。
【未涉及】相机内外参标定、深度图、3D point tracks、光流场作为条件输入、人体姿态/骨架、物体框与轨迹、分割掩码、显式状态标注等。Goku 的条件控制维度仅有「文本 + （I2V 场景下的）首帧图像 + 运动分数」，结构化程度显著低于 2026 年前后强调几何可控性的工作。

### [HunyuanVideo-Foley](../models/HunyuanVideo-Foley.md) ⚠️

论文未使用任何几何或结构化标注：无相机参数、无深度图、无 3D point tracks、无骨架/姿态估计、无物体检测框、无光流场、无动作标注、无物理状态标注。
【唯一涉及时间结构的「结构化」信息：Synchformer 帧级同步特征】这不是标注而是模型内部的条件特征——Synchformer 从视频帧序列中提取帧级的同步相关表征，通过 adaLN 与门控调制通路注入 DiT。它编码的是「画面在何时发生了什么变化」的时序结构信息，在功能上承担了「视觉事件时间轴」的角色，是模型实现精确音画同步的核心依据。但它以稠密特征而非离散标注的形式存在，不可解释、不可编辑、不落盘为数据集字段。
【为何 V2A 任务不需要几何标注】几何标注（相机参数、深度、点轨迹）在视频生成中的价值是控制空间结构与运动一致性。对音效生成而言，理论上有价值的几何信息是「声源距离」（决定音量与混响比）与「空间方位」（决定立体声声像），但本模型输出为单声道/立体声音频且不做空间音频建模，因此这类信息未被利用。
【一个未被开发的方向】若引入深度信息判断声源远近、或引入物体检测定位发声体，理论上可提升音效的空间真实感与多声源场景下的准确性。论文未探索这一方向，可视为后续工作的空间。[不确定：此为分析而非论文内容]

### [HunyuanVideo](../models/HunyuanVideo.md)

有限但明确的一项：镜头运动（camera movement）标注。
【原版】训练了一个镜头运动分类器，可预测 14 种镜头运动类型：zoom in、zoom out、pan up、pan down、pan left、pan right、tilt up、tilt down、tilt left、tilt right、around left、around right、static shot、handheld shot。仅高置信度预测被写入 JSON caption。
【1.5】镜头运动识别模型不再限定为固定14类枚举（报告称识别「多种镜头运动类型」但未列举），改为 clip 级 + 时序级双粒度输出，并转写为自然语言融入 caption。
两代均未使用显式相机内外参、深度图、3D point tracks、显式物理状态或动作骨架标注——镜头运动是以离散标签/自然语言而非几何参数的形式表达的。这与 Seedance、Movie Gen 等引入更强几何监督的路线不同。

### [InstructAV2AV](../models/InstructAV2AV.md) ⚠️

本工作使用了两类结构化视觉标注，但均为编辑任务的工具性中间产物，不作为训练标注保留。
【实例级分割 mask】Grounded-SAM-2（Ren et al., 2024）产出实例级 mask，是本pipeline中最重要的结构化标注。其双重用途：(1) 为 Qwen3-Omni 生成指令提供可编辑对象的锚点（「画面里有哪些实体可以被指令引用」）；(2) 为 mask-guided 视频编辑模型限定编辑区域，从构造上保证 mask 外的像素不被改动——这是「内容保持」在数据层面的硬性机制保障，比依赖模型自觉不改背景可靠得多。注意 mask 只在数据合成阶段使用，最终模型 InstructAV2AV 是 mask-free 的（用户只需给文本指令，无需提供 mask），即 mask 的信息在数据中被隐式蒸馏进了模型。这个「构造时用 mask、推理时不用 mask」的设计是本工作一个清晰的价值点。
【点轨迹】CoTracker3 做网格化点跟踪，产出点轨迹用于计算平均运动幅度。轨迹本身仅用于过滤判据，未作为标注保留。
【说话人时空定位】TalkNet 输出主动说话人的时空位置，用于声画归属判定，同样是过滤用途。
[不确定] 未涉及任何三维或相机层面的标注：无相机内外参、无相机轨迹、无深度图、无 3D point tracks、无人体/面部关键点或姿态标注、无显式的物体状态标注。
【这一缺失与论文局限性的直接关联】论文承认模型「在大幅相机运动下难以保持 3D 空间一致性与物体恒常性（preserving 3D spatial consistency or object permanence during extensive camera movements）」。这一失败模式与数据侧缺乏相机与深度标注、也未对相机运动做分层控制直接相关——数据中既没有告诉模型相机在怎么动，也没有对不同运镜强度的样本做配比控制或分级课程，模型只能从像素中隐式学习，在大运动下自然失效。若后续工作引入相机参数标注或按运镜强度分桶，是一条明确的改进路径。

### [2026 其他音视频联合生成](../models/JAVG_2026_misc.md) ⚠️

本批工作在几何标注上普遍薄弱，唯一的亮点在人脸/身份相关的结构化标注（这对 AV 模型比相机参数更关键）：
【ALIVE —— 人脸轨迹与身份的结构化标注（本批唯一有实质结构化标注的工作）】
(1) 人脸轨迹 ID（TrackID）：SubjectID correction 管线中建立「句子 → TrackID」映射，说明数据中存在人脸轨迹级的结构化标注（每个人脸在时间上被跟踪并分配 ID）。
(2) 人脸-声音对应矩阵：TS-TalkNet 产出的 face-voice correspondence matrix 是一个显式的结构化跨模态关联标注。
(3) 身份嵌入：ArcFace 人脸嵌入 + CLIP 视觉嵌入，用于跨片段身份匹配。Character-driven pipeline 中给出了明确的匹配阈值：face similarity >0.35、CLIP similarity >0.7、absolute proximity to 0.9——这是本批中唯一公开的身份匹配阈值组。
(4) 同步分：TS-TalkNet 的 sync score 用于选取身份锚点（「1.5-second sub-clip with maximum sync score」）。
(5) 人工的 sub-motion units 时间戳标注：把动作分解为子动作单元并标注起止时间——这是一种时序结构化标注（动作边界），虽非几何但属显式结构化。
(6) 未涉及：相机内外参、深度图、3D point tracks、姿态关键点、分割 mask[全部不确定]。
【NAVA】caption 模板中含「camera behavior」板块（camera angle 机位角度、shot scale 景别、framing 取景、composition 构图）——这是语言化的相机描述而非数值化的相机参数，属于「弱结构化」标注。无深度、3D track、姿态等[不确定]。说话人嵌入（ReDimNet）可视为声学侧的结构化标注。
【OmniCustom】参考图通过「randomly sampling and cropping a frame containing a face」（随机采样并裁剪含人脸的帧）获得——隐含人脸检测标注，但仅用于裁剪，未保留为结构化字段[不确定]。无几何标注。
【StreamChar】使用 reference frame 与 motion frame 作为条件——motion frame 是一种隐式的运动状态表示，非显式几何标注。ASR 时间戳属时序结构化标注。
【Baton】planned tokens 本质是「连续语义特征」，可视为一种非人类可读的隐式结构化中间表示——这是对传统结构化标注的另一种替代思路：不标注人类定义的结构（相机、深度、动作），而让模型自己学出一套语义结构。SigLip2 与 WavTokenizer 的倒数第二层特征即其「结构」的载体。
【CCL / ITS-JAVG】无[不确定]。
【总体判断】AV 生成模型的「结构化标注」重心与纯视频生成模型明显不同：后者关注相机参数、深度、3D 轨迹；前者关注人脸轨迹 ID、脸-声对应、说话人嵌入、语音时间戳。ALIVE 的 face-voice correspondence matrix 与 TrackID 体系是这一差异最典型的体现。

### [音视频联合生成基线合集](../models/JavisDiT_baselines.md) ⚠️

本合集几乎全线空白，仅 UniAVGen 有一项人脸相关的结构化信息：
【UniAVGen（唯一）】包含 Face-Aware Modulation（人脸感知调制）模块，该模块需要人脸区域信息才能对特定区域施加调制，因此数据侧应存在人脸检测/人脸区域标注环节，但论文未描述使用何种人脸检测器、是 bounding box 还是 mask、是否离线预计算[不确定]。评测使用 EMTD 基准（音频驱动人脸动画基准）也佐证其人脸相关处理。
【MM-Diffusion】无任何几何标注。值得一提的是 AIST++ 数据集本身自带 3D 人体姿态与相机参数标注（AIST++ 的原始设计目标就是 3D 舞蹈动作生成），但 MM-Diffusion 完全未使用这些标注，只用了 RGB 视频与音频——即「数据集有几何标注但工作没用」。
【AV-DiT】无任何几何标注。
【JavisDiT / JavisDiT++】无相机参数、无深度、无 3D point tracks、无姿态关键点、无动作标签、无分割 mask。唯一接近结构化的是光流评分（flow score，标量运动强度）与 CSV 中的分辨率/帧数等元数据。JavisDiT 的 HiST-Sypo 先验虽名为「时空先验」，但它是从文本估计出的隐式向量而非显式几何标注。
【Harmony】无几何标注[不确定]。
【共性判断】本合集的技术路线普遍是「靠跨模态注意力机制自发学习对齐」，而非「靠显式几何先验约束对齐」——这与 Ovi 强调「不需要人脸 bbox、不需要 face mask」的主张一致，说明音视频联合生成社区整体倾向于数据驱动而非几何先验驱动。UniAVGen 的 Face-Aware Modulation 是这一趋势中的少数反例，代价是把模型能力限定在真人说话场景。

### [可灵 3.0 / 可灵视频 3.0 Omni](../models/Kling_30_Omni.md) ⚠️

[不确定] Kling-Omni 报告未披露相机参数、深度、3D point tracks 等几何标注。间接证据：可灵3.0 强调“3D时空联合注意力”，能建模惯性、重量转移、碰撞检测等物理交互，且历史上可灵有强运镜控制（camera control）能力，暗示训练数据含相机运动/景别的结构化标签（Koala-36M 的“镜头语言”字段即为文本化的运镜标注）。人体结构化标注在 KlingAvatar 2.0 中有明确证据：DWPose 关键点、SAM2 实例分割与时序tracking。[不确定：是否有显式的相机外参估计、深度图或point tracking标注]

### [LTX-2](../models/LTX-2.md) ⚠️

基本没有。两篇论文均未提及相机内外参、深度图、3D point tracks、光流场、显式物理状态或动作类别标签等结构化几何标注。唯一相关的是自然语言层面的相机描述——caption 中以文本形式记录运镜（跟拍、平移、固定机位等），属于弱结构化。附加线索：官方发布了相机运动控制 LoRA 与姿态控制 LoRA，说明存在针对性的控制数据集，但其构造方式（是否使用显式相机轨迹或姿态估计标注）未公开。[不确定]

### [LongCat-Video](../models/LongCat-Video.md) ⚠️

结构化标注集中在「镜头语言」层面而非几何层面：标注了相机运动类别（pan/tilt/zoom，由专用分类器判定）、景别与镜头类型（Qwen2.5VL），但这些是离散语义标签，不是连续的相机外参/内参（无相机位姿轨迹、无 camera pose 数值）。运动方面标注了光流统计量，属稠密二维运动场的聚合量。
报告未涉及深度图、3D point tracks、点云、动作骨架（pose/keypoint，基础版）等几何标注。
Avatar 1.5 因人物驱动需要，做了人脸检测与人脸关键点（landmark）提取以验证面部可见性，以及基于 ByteTrack 的人物跟踪框，并在训练取数时施加 mask-area 约束（对人物 mask 面积占比的限制），这些属于二维结构化标注，仍非 3D 几何。[不确定：是否有未披露的深度/位姿标注]

### [MOVA](../models/MOVA.md) ⚠️

论文未使用任何几何或结构化标注：无相机参数、无深度图、无 3D point tracks、无骨架/动作标注、无显式物理状态标注。唯一接近结构化的标注是 caption 中的 on_screen_text 字段（屏幕文字精确转写）与推理侧 prompt rewriter 提取的四类结构化视觉描述（视觉风格/摄影/视觉元素/OCR 文字），但后者服务于推理时的 prompt 增强，不属于训练数据标注 schema。运镜信息仅以自然语言形式出现在 caption 中（如“The camera alternates between…”“camera panning, zooming, and rotation”），无参数化表示。[不确定]

### [合并条目：① Mochi 1](../models/Mochi_MAGI_Motif.md) ⚠️

① Mochi 1：无披露。架构侧使用扩展到三维的可学习 RoPE 位置编码，但这属于模型设计而非数据标注。[不确定]
② MAGI-1：仅有语义级的相机结构化标注，无数值化几何标注。caption 属性表中包含 Camera Shot Type（景别）、Camera Movement（相机运动）、Camera Transitions（转场）、Subject Position（主体在画面中的位置）、Subject Action（主体动作）五项与结构/几何相关的属性，但均以自然语言描述形式经由 caption 传递，未做相机内外参估计、深度图、3D point tracks 或骨骼动作标注。过滤侧的几何相关量是 RAFT 光流的标量统计（整体/前景/背景三个运动强度）与光流散度、光流一致性，这些只用于判别不作为条件保留。
③ Motif-Video 2B：同样是语义级而非数值级。VLM 结构化标签中的 camera_move 字段以及 video prompt 索取的「shot type、angle、motion」三项相机属性构成了可用于条件化的相机描述，但仍是文本形式；未做相机参数回归、深度、3D 追踪或显式状态标注。几何相关的数值处理只出现在预处理（cropdetect 的内容矩形、OCR 检测框的 IoU 聚类、SSCD 512 维描述子）与过滤（UniMatch 光流分）中。
三者均未采用 2025–2026 年部分工作（如引入相机轨迹参数、点追踪、深度条件）的稠密几何标注路线。

### [Movie Gen](../models/Movie_Gen.md)

以相机语义标注为主，无显式几何量标注：
· 相机运动：自训分类器预测16类相机运动（推拉摇移升降、弧形、跟拍、固定、手持），高置信预测前缀入caption；SFT阶段人工额外标注6类镜别与机位（广角、特写、航拍、低角度、过肩、第一人称视角）。这是全paper最核心的结构化标注体系。
· 动作/行为：建立含600个人类动词与表情的taxonomy，用于zero-shot文本→视频检索与 text k-NN 检索，间接构成动作类目标注。
· 人脸/身份：PT2V数据用人脸检测器（每秒抽帧）、ArcFace 身份相似度（相邻帧>0.5判定为同一人贯穿全片）、人脸区域裁剪与分割（防止模型关注背景等非关键区域）；合成参考图用 ArcFace <0.7 剔除。
· 物体：SFT用 Detic 物体检测判断主体是否过小。
· 运动强度：FFmpeg motion score 与 motion vector 作为运动量标量，评测prompt还人工打了 high/medium/low 运动等级标签。
未使用相机内外参、深度图、3D point tracks、光流场等显式几何标注[确认为未采用]。

### [NVIDIA NeMo Curator](../models/NeMo_Curator.md) ⚠️

结构化标注能力有限但存在两处：
【1. 镜头运动标签】运动分类器（ViT + 光流序列）在过滤的同时对镜头运动类型打标：pan（平移）、zoom（推拉）、tilt（俯仰）。这是可直接用于运镜控制训练的显式结构化标签，是该 pipeline 少见的非文本标注产出。
【2. 运动分数】motion_score_global_mean 与 motion_score_per_patch_min_256 两个数值指标随 clip 一并保留，可作为运动强度的连续型条件信号。
【3. 嵌入】Cosmos-Embed1 视频嵌入与 caption 文本嵌入均以 Parquet 存储并打包进 WebDataset，可用于检索、聚类与条件建模。
【缺失】相机内外参估计、深度图、3D point tracks、光流场落盘、物体框/分割掩码、动作类别标签、显式物理状态等均无内置 stage。相较 Cosmos 平台整体（另有 Cosmos Transfer 等涉及深度/分割条件的组件），curation 侧的几何标注是薄弱环节。[不确定]

### [OmniHuman 数据集 + OHBench](../models/OmniHuman.md) ⚠️

几何与结构化标注是 OmniHuman 相对本次调研中所有其他条目最突出的优势维度——多数 AV 生成数据集在此项为零，OmniHuman 则构建了完整的几何管线：
【人体检测与跟踪】YOLOv11 检测人体实例 → NMS 精修 → MOTRv2 通过 query propagation 建模跨帧关联，输出逐人的身份轨迹与包围框（B_i）。跟踪丢失容忍跨度为 5 帧（超过则视为轨迹中断）。
【2D 骨架：134 关键点】DWPose-L 提取身体、脸部、足部的 134 个全骨架关键点（134 full-skeletal keypoints），并配以专门的手部检测与优化（dedicated hand detection and optimization）——手部单独优化是一个务实的细节，因为通用姿态估计器在手部的精度普遍最差，而手部动作恰是人-物交互与手势表达的关键。134 点对应 COCO-WholeBody 体系（17 身体 + 6 足部 + 68 面部 + 42 手部）。
【3D 参数化人体：SMPL / MANO】HuggingFace 发布物中的 tracking_npz 明确包含「逐帧 SMPL/MANO 身体与手部跟踪」（Per-frame SMPL/MANO body & hand tracking）——即数据集还提供了三维参数化人体模型与手部模型的逐帧参数。这是论文正文未强调但发布物中确实存在的一项重要资产，为姿态可控生成、三维一致性约束、动作重定向等下游应用提供了直接支撑。注意论文正文只提到 DWPose 的 2D 关键点，SMPL/MANO 的获取方式（用了什么估计器、如何与 2D 关键点融合）完全未说明。[不确定]
【身份嵌入】ArcFace 人脸特征，用于身份指派（相似度 > 0.55）与跨帧/跨样本的身份一致性。参考人脸的选取规则为「人脸关键点平均置信度最高的那一帧」。
【人脸清晰度】Cs = Var(ΔR)，拉普拉斯响应方差，作为可查询的标注字段保留。
【镜头语言参数】景别（shot scale）与相机运动（camera motion）作为视频级标注字段，虽为标签而非连续参数，但已足以支撑镜头级的可控生成与诊断。
【人-物交互标注】帧级标注包含 person-object interaction annotations 与对象标注（Table 1 的 Object Anno. 列），是所有对照数据集中唯一具备的。评测侧对应 Object Consistency（对象一致性）与 Contact Naturalness（接触自然度，含空间准确性、时序同步、受力真实性）两个指标。
【未提供】相机内外参（未做相机标定或位姿估计）、深度图、3D point tracks、光流场（UniMatch 的光流仅用于过滤，未作为标注保留）、表情系数（3DMM/FLAME blendshape，虽有 SMPL 但未见面部表情参数化）。[不确定]

### [Open-Sora 系列](../models/Open-Sora.md)

标注深度较浅，仅有二维运动层面的结构化信息，无三维几何标注：
- 有：**相机运动标签**——Open-Sora 1.x 基于 UniMatch 光流做相机运动分类（pan left、zoom in 等），只对高置信度片段打标并写入 caption，模块单独开源在 tools 下；Open-Sora 2.0 与 Open-Sora Plan 则把相机运动作为 caption 的一个描述字段由 VLM 自然语言生成，非独立结构化字段。
- 有：**标量质量条件**——美学分、运动分（VMAF/光流/LPIPS）、OCR 文字区域数、DOVER 技术分，均以结构化列存于 meta CSV 中，部分被拼进 caption 作为可控条件。
- 无：相机内外参、深度图、3D point tracks、光流场本身作为条件输入、显式物体状态/动作标注（action label）、分割掩码等。
- Open-Sora Plan v1.3 训练了独立的 **Structure Controller**，从数据中提取 **canny 边缘、depth 深度图、sketch 草图**三类结构信号作为控制条件（20k 步，8 卡 NPU），这是唯一一处涉及深度信息的标注，但用途是 ControlNet 式条件控制而非主训练数据的标注 schema。

### [Ovi](../models/Ovi.md) ⚠️

[不确定]。Ovi 没有任何几何/结构化标注环节：无相机内外参、无相机运动标签、无深度图、无 3D point tracks、无姿态/关键点、无动作分类标签、无分割 mask、无显式状态标注。
唯一接近的两项是：(1) RAFT 产出的 motion score（标量运动强度，非结构化几何信息）；(2) 内部人脸检测模型输出的人脸存在性/人数（仅用于数据构成配比，未作为条件注入模型）。
论文特别强调其方法「不需要人脸 bounding box、不需要 face mask」等启发式几何先验（对比 HunyuanVideo-Avatar 用 mask 限制音频特征作用于面部区域），把不依赖几何标注作为一项设计优势——唇同步完全由数据与跨模态注意力自发学得，5.1 节的注意力可视化显示语音 token 自动聚焦于嘴部、鼓声聚焦于鼓、动物叫声聚焦于对应身体部位。

### [Script-a-Video](../models/Script-a-Video.md)

MTSS 不包含任何数值型几何标注：无相机内外参、无深度图、无 3D point tracks、无光流场、无骨架/姿态关键点、无边界框坐标、无分割掩码。
但本工作在「非数值型结构化标注」维度做得相当深，是本条目的实质内容：
【镜头语言的语义化标注】Shot 流的 camera 字段用专业电影语言记录镜头运动（movements）、视角（perspectives）、景别（scales）。这是把相机信息以「导演语言」而非「参数矩阵」的形式结构化——牺牲了精确性，换取了可被 LLM 直接理解与生成的可控性。对于以文本为条件接口的生成模型而言，这是比数值相机参数更实用的形式。
【实体级的结构化外观锚定】Reference 流的 appearance_anchor 对 person 类实体展开为服装、配饰、发型等细粒度属性槽位。这是一种「以属性槽位替代像素级标注」的身份表示。
【时间轴的结构化】这是 MTSS 最硬核的结构化维度——三层时间标注：shot 级 time_range、event 级 time_range、以及嵌入描述文本内部的 intra-description / micro-level timestamps。三层共用同一条全局时间轴，构成一个可做区间运算的显式时序结构。论文称其目标是达成 sub-frame（亚帧级）的视听协调。
【关系图结构】references_in_shot（镜头→实体的多对多边）、active_events（镜头→事件的边）、speaker（事件→实体的边）三类引用共同构成一张显式的实体-镜头-事件三部图。这是整个 schema 中最接近「知识图谱」的部分，也是「Relational Grounding」一词的实质所指。
【动作标注】以自由文本形式存在于 visual_description 中，要求「客观、按时序」叙述核心动作，并由 intra-description timestamps 锚定，但无动作类别标签体系。
【显式状态】Reference 流的 semantic_description 记录实体的整体状态，属于弱化版的显式状态标注。

### [Seedance 2.0 与 Seedance 1.5 pro](../models/Seedance_20_Seedance_15_pro.md) ⚠️

[不确定] 未披露相机参数、深度、3D point tracks 等几何标注。已知的结构化标注仅停留在文本 caption 层面：Seedance 1.0 的 caption 显式包含相机运动（camera movements）类目作为动态特征之一；Seedance 1.5 pro 具备自主运镜调度能力（连续长镜头、dolly zoom/希区柯克变焦、电影级转场与专业调色）；Seedance 2.0 具备基本的导演与摄影推理能力，可自主规划镜头序列。这些提示运镜以语言标签而非数值相机参数的形式进入训练。

### [SkyReels 系列](../models/SkyReels.md) ⚠️

【SkyReels-V2】存在明确的「摄影几何」层面的结构化标注，但为分类标签而非连续参数：镜头类型、机位角度、机位位置、相机运动（6自由度运镜的分类识别，由专门的运动识别 captioner 输出，配套9.3万人工标注+1.6万合成样本）。这些字段以离散标签形式进入 caption，支撑了 SkyReels-V2-Camera-Director（相机导演）变体的可控运镜能力。未使用相机内外参数值、深度图、3D point tracks、光流场等连续几何标注。
【SkyReels-V4】结构化标注的重心转向「掩码与条件」而非几何：为统一 inpainting/editing 框架，训练数据需带有区域 mask、参考图像、参考视频片段、参考音频等条件通道（Stage6 中图像参考与视频参考各占20%）；视觉分割模型被用于构造 mask。论文未提及相机参数、深度、点轨迹、显式物理状态等标注。[不确定]

### [Sora 2](../models/Sora_2.md) ⚠️

完全未披露。未提及相机参数、深度图、3D point tracks、动作标注或显式物理状态标注。第三方技术解读称训练数据带有覆盖重力、动量、浮力、材料形变、碰撞动力学的「物理标注（physical annotations）」，若属实则构成显式结构化状态标注，但该说法非 OpenAI 官方表述，无法证实。OpenAI 官方仅在能力层面宣称模型「understands real-world physics」并将 Sora 2 定位为迈向物理世界模拟器（world simulator）的一步。[不确定]

### [Step-Video-T2V](../models/Step-Video-T2V.md)

结构化标注较少，主要是两类标量与一类文本：
【运动标量】Farneback 光流导出的 Motion_Mean / Motion_Max / Motion_Min 三个数值标签（预训练用于过滤；在 Step-Video-TI2V 中升级为注入模型的显式可控条件，实现运动幅度可控）；
【聚类标量】VideoCLIP embedding 的 Cluster_Cnt 与 Center_Sim；
【镜头运动】以自然语言形式写入 dense caption（如「camera pans left」），而非离散标签枚举——这与 HunyuanVideo 训练了 14 类镜头运动分类器输出离散标签的做法不同；Step-Video-TI2V 通过微调 caption 模型强化运镜描述来实现「镜头运动可控」，仍走自然语言路径。
【完全缺失】未使用相机内外参估计、深度图、3D point tracks、光流场本身作为监督信号、动作骨架、显式物理状态等几何结构化标注。这与 Movie Gen、Seedance 等引入更强几何/结构监督的路线形成对比，也是团队在报告「Challenges and Future」章节中自陈的短板之一（复杂物理与因果建模能力不足）。

### [UniTalking](../models/UniTalking.md) ⚠️

论文未使用任何几何或结构化标注：无相机参数、无深度图、无 3D point tracks、无光流场、无面部关键点（landmark）、无表情系数（3DMM/FLAME/blendshape）、无头部姿态、无显式物理状态标注。运镜、景别、构图等信息也未做参数化或标签化，仅可能以自然语言散落在 Qwen3-VL 的视频 caption 中，但无 caption 示例可供确认。[不确定]
【流水线中涉及空间结构的模型仅两个，且输出均未保留】
- LightASD：主动说话人检测，内部需定位并跟踪人脸区域，但其输出在流水线中仅用作布尔判据（发声者是否在画面内），人脸框与轨迹未作为标注保留；
- LipSync：唇同步评估，同样需定位嘴部区域，但仅输出一个用于过滤的对齐分数，唇部区域信息未保留。
即人脸空间信息在过滤阶段被使用后即被丢弃，未转化为可供训练使用的条件。
【一处被放弃的现成结构化标注】其上游数据源 OpenHumanVid 原生自带人体骨架序列（skeleton sequences）作为运动条件，这是一份现成的高价值结构化标注。UniTalking 完全未使用（论文对骨架只字未提），其条件体系仅有文本、身份图像、参考音频三路。对于说话人生成任务而言，骨架对上半身与手部动作的可控性本可带来增益，放弃它是一个未加解释的取舍。[不确定]
【替代性的结构建模在架构侧而非数据侧】UniTalking 对时空结构的处理体现在各向异性 RoPE 上：音频 token 的空间维度 (h, w) 使用单一固定位置的 RoPE，退化掉空间维度以强制模型把建模能力集中在时间轴——这是用位置编码而非标注来注入结构先验。

### [UniVerse-1](../models/UniVerse-1.md) ⚠️

论文未使用任何几何或结构化标注：无相机参数、无深度图、无 3D point tracks、无骨架/姿态/动作标注、无显式物理状态标注、无光流场。
唯一涉及空间结构的模型是 RetinaFace 人脸检测，但其输出仅用作漏斗中的布尔判据（画面中是否存在人脸，以决定该片段能否进入语音子集并接受 SyncNet 核验），检测框坐标与人脸关键点并未作为标注保留或作为训练条件使用。
运镜、景别、构图等信息也未做参数化或标签化，仅可能以自然语言形式散落在 QWen2.5-Omni 生成的 video caption 中，但论文未公开 caption 示例故无从确认。[不确定]

### [Unison](../models/Unison.md) ⚠️

论文未使用任何几何或结构化标注：无相机参数、无深度图、无 3D point tracks、无骨架/姿态/关键点标注、无光流场、无物理状态标注、无动作类别标签。
【唯一涉及空间结构的元素是人脸检测框】lip-filtering 算子检测人脸的数量与位置（bounding boxes），但这些检测框的用途是限定 SyncNet 的运行区域（applied exclusively within these bounding boxes），属于流水线内部的中间产物，未作为标注保留、也未作为训练条件输入模型。检测器型号未披露。[不确定]
【值得注意的是「motion」的建模方式】尽管标题以 motion 为核心概念，Unison 并未对运动做任何显式的几何参数化（无骨架、无关键点轨迹、无手部姿态）——这与其数据源之一 OpenHumanVid 提供骨架标注形成对比，说明 Unison 并未利用该数据集的这一特性。运动与音频的对应关系完全依靠隐式学习：架构上靠三帧窗口的帧级交叉注意力，训练上靠双向跨模态 forcing。论文的定性结果（钢琴演奏中音符起音对应手指动作、撞击瞬间对应冲击声）表明这种隐式路径在短时窗内是有效的，但缺乏显式几何监督也意味着无法验证或控制对应关系的物理正确性。
【运镜/景别信息】未做参数化或标签化，可能以自然语言形式散落在 caption text 中，但因无 caption 示例无从确认。[不确定]

### [Veo 3 / Veo 3.1](../models/Veo_3_Veo_31.md) ⚠️

[不确定] 未披露任何几何或结构化标注。未提及相机参数（内外参/位姿）、深度图、3D point tracks、光流、动作标签或显式物理状态标注。可作反推的两点：(1) Veo 3.1 提供了明确的「camera controls（精确取景与运镜控制）」产品能力，暗示训练数据中存在某种形式的运镜标注（最可能仍是通过 caption 中的自然语言运镜描述实现，而非显式相机参数）；(2) DeepMind 自家论文《Video models are zero-shot learners and reasoners》(arXiv:2509.20328) 用 62 项视觉任务测试 Veo 3，发现其零样本具备分割、边缘检测、关键点定位、超分、去模糊、去噪等能力，并提出「chain-of-frames」概念——作者强调这些任务模型均未被显式训练，反而说明这些几何/结构能力是从大规模自然视频中涌现的，而非来自显式几何标注。

### [Vidu S1](../models/Vidu_S1.md) ⚠️

无显式几何标注。caption 中含「镜头语言（camera language）」与「影视化属性（cinematic properties）」「光照（lighting）」等半结构化文本标签，omni 模型另输出 shot、scene 等语义标签，但论文未提及相机参数估计、深度图、3D point tracks、姿态/骨骼关键点等几何或显式状态标注 [不确定]。

### [主流视频预训练数据集合并调研：Panda-70M、InternVid、Koala…](../models/pretraining_datasets.md)

**七者的结构化标注都停留在二维语义层，无一提供三维几何信息**：
【有的部分】
- **相机运动**：六家把运镜作为caption中的描述字段（Koala-36M的「镜头语言」含运镜/角度/焦距/景别；MiraData有独立的`camera_caption`列；UltraVideo有独立的Camera Movement与Shot Type两个维度；LVD-2M的caption包含相机视角；OpenVid/InternVid的自由文本中零散涉及）。**但全部是自然语言描述，没有任何一家提供结构化的运镜标签枚举或相机轨迹参数。**
- **标量质量分（唯一真正结构化的部分）**：**Koala-36M最完整**——CSV直接发布三列可用于条件控制的分数：`clarity_score`(0–1)、`aesthetic_score`(2.28–6.56)、`motion_score`(0.01–267)，外加融合分`video_training_suitability_score`(2.50–4.95)。**Panda-70M**发布`matching_score`（UMT图文匹配分）、`desirable_filtering`（6类枚举）、`shot_boundary_detection`（TransNetV2输出的镜头区间列表，长度为1即单镜头）。**LVD-2M**发布`total score`（平均光流分）、`scene_cut`、`span`、`video_time`、`dataset_src`。**InternVid**发布UMT-SIM分与美学分。MiraData/OpenVid/UltraVideo的CSV则主要是caption字段，质量分未随数据发布。
- **视频类目标签**：仅**LVD-2M**用BART分类出8类标签（有精确百分比），**UltraVideo**有7主题108子类的体系（但映射关系未发布，issue #5追问无果）。
【完全没有的部分】相机内外参、深度图、3D point tracks、光流场本身作为条件、显式物体状态/动作标签（action label）、分割掩码、人体姿态/关键点——**七者全部为零**。唯一沾边的是MiraData在评测端（MiraBench）用**GVGC三维重建**算MAE/RMSE作为「3D一致性」指标、用**CoTracker**算Tracking Strength，但那是**评测指标而非数据标注**，不随数据集发布。
**结论**：这七个数据集是纯粹的「视频-文本对」资源，无法直接支撑相机可控生成、深度可控生成或3D一致性训练——下游若需这些能力必须自行补标。

## 合成数据构造（受控扰动/编辑构造训练对，如InstructAV2AV）

`synthetic_data_synthesis` · 详细程度: brief

### [Allegro](../models/Allegro.md)

论文未使用合成数据构造训练对，未做受控扰动/编辑构造（如 InstructAV2AV 式的成对数据合成）。数据全部为真实视频经切分与筛选而来。唯一的「生成式」环节是 caption 由模型生成（Tag2Text / Aria），属于合成标注而非合成视频。

### [Apollo](../models/Apollo.md)

论文未提及任何合成数据构造：无受控扰动/编辑构造的训练对，无 InstructAV2AV 式的指令编辑数据对，无 TTS 配音合成的伪音视频对，无模型自生成数据回流（self-distillation / rejection sampling）。全部 8100 万训练样本来自真实带原生音轨的视频，pipeline 的角色是过滤与标注而非生成。唯一的「合成」成分是全部 caption 由自动模型产出（合成标注，非合成内容）。多任务能力（T2A / T2V / T2AV / I2V / I2AV）不是通过构造不同形态的合成数据对得到的，而是通过**随机模态掩码（random modality masking）**在同一批真实数据上动态构造训练目标——这是一种「训练时合成任务」而非「数据侧合成样本」的思路，也是 Apollo 相对省数据工程量的关键设计。

### [CineDance / CineDance-1M](../models/CineDance.md)

无。CineDance-1M 全部为真实拍摄的影视素材，pipeline 中不含任何受控扰动、编辑构造、成对样本合成（如 InstructAV2AV 式的 before/after 编辑对）或数据增强式合成环节。
【最接近的操作】仅有的「构造」是把原子镜头重组为叙事序列，属于对真实素材的重新分组而非合成；以及 CineDance 模型训练中的 DARC 课程（对参考帧做连续加噪 ρ(rₖ; ηᵥ(u)) = ηᵥ(u)·rₖ + (1−ηᵥ(u))·εₖ、随机索引切换、参考帧按概率 p_drop(u) 丢弃），属于训练时的输入扰动策略而非离线合成数据构造。

### [CogVideoX](../models/CogVideoX.md) ⚠️

· 视觉数据：全部为真实采集视频，未构造任何合成视频、未做受控扰动或编辑构造训练对。
· 文本数据：caption 全部为合成文本，且是多级模型合成链——Panda-70M 短 caption + CogVLM 逐帧图像 caption → GPT-4 摘要（5 万条）→ 蒸馏进 LLaMA2 → 再蒸馏进 CogVLM2-Caption。这是本工作中唯一体系化的合成数据构造，属于「合成标注」而非「合成样本」。
· 推理侧合成：caption upsampler 用 LLM 合成长 prompt，使推理输入分布向训练 caption 分布靠拢，本质也是一种合成数据对齐手段。
· 图生视频（I2V）模型的训练对构造细节见附录 D，论文正文未描述是否使用合成首帧 [不确定]。

### [Cosmos-Predict2.5](../models/Cosmos-Predict25.md)

呈现出「训练数据侧排斥合成、下游应用侧生产合成」的双面态度，是该工作的一个鲜明特点：
【预训练侧：主动排除合成内容】过滤末端显式剔除 video games、synthetic visual patterns、animations、cartoons，理由是维持与真实物理世界分布的对齐。即在训练语料层面，合成/非物理内容被视为污染。
【机器人数据的受控改写】机器人域对动作过慢的视频提高播放速度以归一化动作节奏，属于对真实数据的受控时间扰动。
【下游：模型即合成数据工厂】这是该论文的核心应用主张之一——用 Cosmos-Transfer2.5-2B 为真实机器人演示做视觉增强以扩充策略训练集。具体做法：100 段人类遥操作演示（双臂抓苹果放入碗中的 pick-and-place 任务）→ 用 VLM 对样例视频生成详细 caption → 反复用 Cosmos-Transfer2.5 生成并核对以精修 caption → 把可变成分参数化为槽位模板（[TABLE]、[COLOR_APPLE]、[COLOR_BOWL]、[SENTENCE_LIGHT]、[SENTENCE_BACKGROUND]）→ 用 LLM 为各槽位生成候选变体 → 每段原始演示生成 5 个合成变体（共 500 段），只替换图像观测，动作与关节状态保持不变。控制参数：edge 阈值 medium、blur 阈值 very low、CFG scale 3。
【效果】真机 10 个测试场景各 3 次试验：仅用 100 段真实演示的 base policy 1/30 成功；用传统图像增强（亮度/对比度/饱和度/色相扰动、伽马校正、椒盐噪声、直方图均衡、随机模糊/锐化）的 baseline policy 5/30；用 Cosmos-Transfer2.5 增强的 proposed policy 24/30，在换物体、换桌布、加聚光灯、加干扰物、改背景柜、开抽屉与组合难例上均显著更强。论文据此论证：传统像素级增强无法做语义编辑（改物体颜色、环境外观、光照），而生成式增强可以。
【VLA 合成数据】另有 6.5 节专门讨论用 Cosmos-Predict2.5 为 Vision-Language-Action 模型训练生成合成数据。

### [Data-Juicer 2.0](../models/Data-Juicer.md) ⚠️

【合成能力的规模】229个算子中约50个专门服务于数据合成与增广，是 DJ 的四大能力板块之一（分析、合成、标注、后训练）。这一比例明显高于以清洗为主的同类框架。
【视频侧的合成/构造算子】
  · video_ffmpeg_wrapped_mapper —— 封装 FFmpeg 视频滤镜，可施加任意视频变换（裁剪、调色、加噪、变速、转场等），是构造受控扰动样本的通用工具。
  · audio_ffmpeg_wrapped_mapper —— 音频侧对应物。
  · audio_add_gaussian_noise_mapper —— 音频加高斯噪声增广。
  · video_resize_resolution_mapper / video_resize_aspect_ratio_mapper —— 规格改写，可用于构造多分辨率训练对。
  · video_remove_watermark_mapper —— 水印去除，可与原始带水印样本构成「去水印」编辑训练对。
  · video_object_segmenting_mapper —— 产出物体掩码，是构造对象级编辑/替换训练对的前置条件。
【DJ-Cookbook 中的合成配方】维护有「视频数据合成」（video-data-synthesis）YAML 配方、对比学习数据合成配方（对应 CVPR 2025 的 ImgDiff 工作，arXiv:2408.04594，通过对比式数据合成提升 VLLM）、角色导向对话合成、以及基于数据难度的课程学习配方。
【关联研究】同团队的 ImgDiff（CVPR 2025）与 MindGYM（NeurIPS 2025，问题合成）都是「用大模型合成训练数据」的方法论工作，其能力已回流为 DJ 算子。
[不确定] DJ 未提供类似 InstructAV2AV 的音视频编辑指令对构造能力，也未提供针对视频生成的「受控扰动构造正负样本对」的现成配方（如构造运动强度对比对、音画错位负样本对）。视频合成配方的具体内容与实际产出规模未公开。

### [Foley-Omni](../models/Foley-Omni.md) ⚠️

[不确定] 论文未描述任何合成数据构造环节。pipeline 中不存在受控扰动、音轨替换、时间偏移构造负样本、或类似 InstructAV2AV 的编辑指令对构造。
唯一涉及「音频拆解重组」的是 Bandit 源分离，但其用途是验证标注（能量门控）而非构造训练样本——论文未提及把分离得到的单类 stem 反过来当作 TTA/TTM 的合成训练数据（这本是一个自然的数据增广思路：从 V2ST 数据中分离出纯音效轨可扩充 V2A 训练集，但文中未采用或未提及）。
数据全部为真实采集的音视频对 + 模型生成的文本标注，属于「真实数据 + 模型标注」范式，不含生成式数据合成。

### [Goku](../models/Goku.md) ⚠️

训练数据中未使用合成数据构造。论文没有受控扰动、编辑构造训练对、渲染引擎合成、数据增强生成新样本等任何合成数据管线。
唯一与「合成/改写」沾边的两处均非训练数据：
(1) 分布均衡阶段对表征不足类别做的「augmentation and oversampling」——这里的 augmentation 应指传统数据增强/重采样，论文未说明是否含生成式扩增；
(2) 评测阶段用 GPT-4o 扩写 GenEval 的短提示词（「we expand the original short prompts in GenEval with ChatGPT-4o, preserving their semantics while enhancing descriptive detail」），属于 prompt rewriting 的评测技巧，用于弥合训练期密集 caption 与评测期短提示词之间的分布差距——这一点本身是「训练 caption 风格影响推理提示词形态」的重要证据。[部分不确定]（均衡阶段 augmentation 的具体含义）

### [HunyuanVideo-Foley](../models/HunyuanVideo-Foley.md)

论文未构造任何合成数据：
- 没有受控扰动构造训练对（无音效替换、无时间偏移、无音量扰动等构造的正负样本对）；
- 没有编辑式数据构造（无 InstructAV2AV 式的音频编辑指令对）；
- 没有人工混音合成（未用干净音效库 + 视频素材人工配对合成训练样本——这在 Foley 领域是一条可行但未被采用的路线）；
- 没有 TTS 或音频生成模型自产数据做自蒸馏；
- 没有负样本构造用于对比学习。
全部训练数据均为真实采集的原生音视频片段，且强制要求自带原生同步音轨。
【唯一的「合成」成分】全部 caption 由 GenAU 模型自动生成，属合成标注而非合成内容。
【唯一接近「构造」的设计：high-quality 标签】通过在 caption 中人为追加质量标签、并在推理时强制使用，构造了一种「训练分布与推理分布刻意错开」的条件化机制——训练时标签如实反映样本质量，推理时一律使用高质量标签，从而把生成引导至分布的高质量端。这不是数据合成，但属于对数据条件的人为构造，思路上与 classifier-free guidance 的条件操纵同源。
【值得注意的未采用路线】Foley 领域有一条经典的数据构造路线：用专业音效库的干净单音效 + 视频素材，通过人工对齐合成「完美对齐、完美纯净」的训练对。本工作完全走真实数据路线，好处是声学场景真实、多声源混合关系自然，代价是无法获得完全纯净的监督信号（真实音轨中总混有无关背景声）。论文未讨论这一取舍。

### [HunyuanVideo](../models/HunyuanVideo.md) ⚠️

两代均未使用合成数据构造训练对，报告中无受控扰动/编辑式配对数据（如 InstructAV2AV 类）的描述。唯一带「构造」性质的是 1.5 的 T2V DPO 阶段：对同一 prompt 用模型自身采样 N 个候选视频，两两组成非重复的偏好对（non-repetitive pairs），再由人工 GSB 标注胜负——属于模型自采样构造偏好数据，而非训练语料合成。此外原版的 caption dropout+排列组合可视为文本侧的数据增广。[不确定]

### [InstructAV2AV](../models/InstructAV2AV.md) ⚠️

这是 InstructAV2AV 在本次调研中最具代表性的维度，也是 note 中把它列为「合成数据维度代表」的原因。本工作的训练数据中，每一条 target 都是合成产物——不是真实拍摄的，而是被受控制造出来的。

【为何必须合成：任务的根本困境】指令引导编辑需要 (source, instruction, target) 三元组监督，其中 source 与 target 必须是「同一场景下仅目标区域不同」的两份视频。这样的配对在现实中根本不存在——你无法拍摄同一个场景两遍且只有一只狗变成了猫、其余像素完全相同。论文明确以此为出发点：「Given the scarcity of such source-to-target paired resources in the audio-visual domain」。因此数据合成不是提效手段，而是任务成立的前提。

【合成机制的三层受控设计】
  1. 空间受控（视觉侧）：Grounded-SAM-2 产出实例 mask，mask-guided 编辑模型（基于 Wan2.2-5B，以 source-conditioned flow matching 目标训练）只在 mask 区域内合成新内容，mask 外像素来自源视频。这从构造上硬保证了「非目标区域严格未变」，使 source-target 构成像素级干净的差分对。
  2. 声源受控（音频侧）：SAM-Audio 把目标实体的声音从原混合音轨中分离出来，ElevenLabs 合成新的目标声，再与保留的原背景音无缝混合。同理保证「非目标声音严格未变」，环境声/背景音在两侧完全一致。
  3. 跨模态同步受控：视频合成模型通过 frame-wise cross-attention 接收已合成的目标音频特征作为条件，强制新画面与新音轨逐帧对齐。合成顺序为「先音后视」——语音的时长与节奏一旦确定，口型必须服从它。
  这三层受控共同产出了真实数据永远无法提供的东西：精确的因子级差分监督。

【正交因子的解耦构造】五个子集 add_and_remove / clone_id / clone_voice / clone_id_voice / general_editing 体现了对「视觉身份」「听觉音色」「实体存在与否」这几个因子的独立操控与组合枚举。尤其 clone_id（换脸不换声）与 clone_voice（换声不换脸）这一对，构成了严格的对照组——模型可以从中学到身份与音色是可分离的两个属性。这类解耦监督是合成数据相对真实数据的核心优势，也是本工作最值得借鉴的构造思想。

【双向配对的数据增益】HF 数据集含 instruction_reverse 字段，同一对 (source, target) 可同时用作正向（真实→合成）与逆向（合成→真实）两条训练样本。逆向样本的生成目标是真实视频，可缓解模型输出分布被合成分布锚定的问题。

【合成路线的固有风险（论文未讨论）】
  · 能力天花板继承：target 的质量上界由 Wan2.2-5B 与 ElevenLabs 决定，InstructAV2AV 学到的编辑效果原则上无法超越其数据引擎。论文承认的局限（物理真实感、光照一致性、3D 一致性）明确表述为「inherit from the foundational generation model」，正是这一风险的体现。
  · 分布偏置自我强化：训练集与评测集（1K）出自同一引擎，模型可能在「模仿该引擎风格」上得高分而非真的会编辑。论文用外部 AvED-Bench 零样本评测部分回应了这一质疑（FVD 227.82 vs AVI-Edit 372.37）。
  · 失败模式同源：数据引擎的系统性缺陷（如某类物体总是合成不好）会被完整传递给下游模型且不可被自动验证发现——因为验证用的 Qwen3-Omni 与生成用的 Qwen3-Omni 是同一模型。
[不确定] 未披露数据引擎的一次合成成功率、每个 source 平均产出多少条配对、合成的算力成本与总耗时。

### [2026 其他音视频联合生成](../models/JAVG_2026_misc.md) ⚠️

【ALIVE —— 本批唯一系统性构造合成/增强数据的工作，且方法值得细看】
Character-driven data pipeline 分两路：
(1) Cross-pair Pipeline（跨片段配对，用真实数据构造「同一身份不同场景」的训练对）：
  - 从 10–30 分钟长视频中抽取 N 个 3–10 秒片段
  - 选取「1.5-second sub-clip with maximum sync score」（同步分最高的 1.5 秒子片段）作为身份锚点（identity anchor）——用同步分而非画质分选锚点很有讲究：同步分高意味着此刻人物正在清晰说话、正脸朝向摄像机，是身份特征最充分暴露的时刻
  - 用 CLIP + ArcFace 嵌入做多主体特征匹配
  - 有效跨片段配对需同时满足：face similarity >0.35、CLIP similarity >0.7、absolute proximity to 0.9
  这一路是「用真实长视频挖掘同身份配对」，不是生成合成像素，但属于典型的配对数据构造（training pair construction）。
(2) In-pair Augmentation（片段内增强，真正的合成数据构造）：
  - pose/expression perturbation（姿态/表情扰动）
  - background augmentation（背景增强/替换）
  - semantic editing via Qwen-Image-Edit（用 Qwen-Image-Edit 做语义编辑）
  目的明确：「to decouple identity from appearance」（把身份与外观解耦）——即让模型学会「这个人换了姿势、换了背景、换了服装仍是同一个人」，而不是把背景和衣着记成身份的一部分。用图像编辑大模型做受控扰动来构造正样本对，是 2026 年很典型的做法（与 InstructAV2AV 类思路同源，但 ALIVE 用于身份解耦而非指令编辑）。
(3) Prompt 侧的合成：训练时以 0.3 概率 dropout 跨注意力信号（cross-attention signal），属条件 dropout 增强；推理侧用 FAISS 构建稠密向量索引，从用户 prompt 中抽取声音事件（sound events）并检索最近邻，相似度阈值 τ>0.85，做检索增强的 prompt 改写——这是推理期而非训练期的合成。
【OmniCustom —— 配对构造而非像素合成】其「前 4 秒作参考音频 / 后 5 秒作训练音视频」的切分策略，是一种零成本的配对数据构造：「each reference-training pair shares the same timbre but contains distinct speech content」（同音色、异内容）。这直接为音色解耦提供监督：模型必须学会从参考音频中只提取音色而不复制内容。参考图则从含人脸的帧中随机采样裁剪，同样是配对构造。无像素级合成或编辑。
【StreamChar —— 蒸馏数据即自生成数据】两阶段蒸馏中 Stage II 为「rollout consistency」（在线 chunk rollout 下微调 student），本质上是学生模型自己生成的 rollout 序列作为训练数据——属于 on-policy 的自生成数据，与传统合成数据构造不同但同属「模型产出数据回灌训练」。Stage I 600 steps 做步数压缩到 4 步生成器，Stage II 400 steps，student lr 2e-6、fake score network lr 4e-7。蒸馏数据源复用同一套音视频数据集，无额外构造。
【NAVA / CCL / Baton】未描述合成数据构造[不确定]。Baton 的三阶段训练中 Stage 2→Stage 3 的过渡涉及「从干净 ground-truth 特征切换到规划器预测特征」，这是一种课程式的输入扰动（用模型自身的不完美预测替代干净标签），思路上接近 scheduled sampling，可视为一种训练输入侧的合成，但不产生新数据样本。
【ITS-JAVG】其 Best-of-N 与 EvoSearch 在推理时大量生成候选再择优，若把择优结果回灌训练即构成合成数据管线——但论文明确 training-free，未做此扩展。这是一个显而易见的后续方向（用 ITS 的多验证器筛选产出高质量合成 SFT 数据），论文未涉及。

### [音视频联合生成基线合集](../models/JavisDiT_baselines.md) ⚠️

本合集中有两项工作使用了合成/自举数据构造，且都是本合集数据侧最具借鉴价值的部分：
【JavisDiT —— 异步负样本的受控构造（ST-Prior 训练）】为训练 HiST-Sypo 时空同步先验估计器，需要区分「同步」与「不同步」的音视频对。作者在 61 万条同步三元组之外，额外「合成异步负样本（synthesized asynchronous negative samples）」，配合对比学习（contrastive learning）训练先验估计器。这是典型的「受控扰动构造训练对」——通过对音轨做时移、替换或跨样本错配来人工制造负例。具体的负样本构造方式（时移多少帧、是否跨样本随机配对、正负样本比例）在论文附录 C.2.4「Negative Sample Construction」中，但公开 HTML 版本此节内容被截断，未能获取细节[不确定]。这一设计与 Ovi「靠严格阈值筛掉不同步数据」形成鲜明对照：Ovi 是把不同步数据丢弃，JavisDiT 是把不同步数据主动造出来当负例用。
【JavisDiT++ —— 模型自生成的偏好数据（AV-DPO）】用参考模型对 3 万条 prompt 池中的每条 prompt 生成 N=3 个音视频候选，与 1 条真值组成「1 真值 + 3 生成」的候选组，再用六个奖励模型打分排序，构造出约 2.5 万条偏好对。这是完全的模型自举合成数据。一个值得记录的量化发现：最终偏好数据中约 30% 的 winning 样本来自模型生成而非真值——即模型在近三成的情形下能生成优于真实数据的样本，作者据此判断基线模型已具备相当强的生成能力。这个比例对判断「何时该转向 DPO/RLHF」是个有参考价值的信号。
【Harmony —— 数据配对层面的构造】阶段二音色解耦微调采用「跨话语错配的参考-目标配对」：语音数据取同一说话人的不同话语作为参考与目标，环境音数据取同一片段的非重叠段作为参考与目标。这不是生成新数据，而是通过重新配对已有数据来构造训练信号，属于轻量级的合成配对策略。
【MM-Diffusion / AV-DiT / UniAVGen】未使用任何合成数据构造[不确定]。

### [可灵 3.0 / 可灵视频 3.0 Omni](../models/Kling_30_Omni.md) ⚠️

有，且是官方明确披露的重点。Kling-Omni 报告称数据构建为“大规模真实世界数据采集 + 面向任务的合成数据构造”双轨；合成侧由“专家模型驱动的合成 pipeline”实现，使用自研图像编辑模型与视频理解模型构造图像/视频编辑任务与多图参考任务的训练对；并提出“自动逆向合成策略（automatic reverse synthesis strategy）”构造 reference-to-video 训练样本，即从成片反向抽取参考图/条件、再配对为(条件, 目标视频)训练对。这直接支撑了 Omni Edit（局部定向编辑不重生成整段）与多图+音色参考锁定能力。[不确定：合成数据在总量中的占比；是否存在音频侧的受控扰动式合成，如 InstructAV2AV 类的音视频编辑对构造]

### [LTX-2](../models/LTX-2.md) ⚠️

技术报告未提及使用任何合成数据或受控扰动/编辑构造训练对（无类似 InstructAV2AV 的编辑配对构造）。训练数据均来自真实视频（公开可得 + 授权素材）加图像数据集。唯一涉及「模型生成内容」的地方是 LTX-Video 论文中用 Sora 生成的视频作为 captioner 效果的展示样例，属于演示而非训练数据。是否在未公开流程中使用蒸馏/自生成数据（如蒸馏版模型的训练）无从判断。[不确定]

### [LongCat-Video](../models/LongCat-Video.md) ⚠️

有限使用。明确的一处是主 captioner 的训练：LLaVA-Video 是「using in-house synthetic pairs」（用内部合成的图文/视频-文本 pair）微调的，即合成数据用于训练标注器本身，而非直接作为生成模型的训练样本。
视频生成侧未见受控扰动/编辑构造训练对的做法（如 InstructAV2AV 式的成对编辑数据构造）。Video-Continuation 任务的训练数据构造属于自监督式的条件切分——以片段的前若干帧作为条件帧、后续帧作为预测目标，用条件帧数量区分三类任务（T2V 为 0 帧条件、I2V 为 1 帧参考、VC 为多个前序帧），这是从真实视频自动派生监督信号，而非合成生成新内容。
此外 caption 的「随机组装电影语言与风格标签」可视为一种 prompt 侧的合成增强。[不确定：是否使用了生成模型自举的合成视频数据]

### [MOVA](../models/MOVA.md) ⚠️

论文未构造任何合成训练对：没有受控扰动、没有编辑构造的配对数据（如 InstructAV2AV 式的编辑对），也没有用生成模型自产视频回灌训练。全部训练数据为真实视频。仅有的“合成”成分是：(1) 音频塔预训练中使用的 in-house TTS 合成语音数据；(2) 全部 caption 由 MLLM/LLM 自动生成（合成标注而非合成内容）；(3) 评测基准的 prompt 由 GPT-5 统一改写整合，以及推理工作流中由 Gemini 2.5 Pro 生成的改写 prompt。[不确定]

### [合并条目：① Mochi 1](../models/Mochi_MAGI_Motif.md) ⚠️

① Mochi 1：无披露。[不确定]
② MAGI-1：数据全部为真实素材经切分筛选而来，未构造受控扰动/编辑训练对。唯一的「合成」环节是文本侧——caption 由 MLLM 生成（合成标注），以及约 200 万条用于蒸馏 Prompt Enhancement 小模型的合成 prompt 改写语料。
③ Motif-Video 2B：本组中唯一在视觉侧显式使用合成数据的。原始池被拆为 Image Real / Image Synthetic / Video Real / Video Synthetic 四个分支，即图像与视频两侧都设有合成支路。关键的调度约束是：「Synthetic video is injected only at 720p, where its controlled quality is most compatible with the admission criteria（合成视频只在 720p 阶段注入，因为其可控的质量最契合该阶段的准入标准）」——也就是把合成数据当作高质量阶段的补充而非预训练的填充料。但论文未说明合成视频由何模型生成、占比多少、如何避免自蒸馏式的分布塌缩，也未做合成 vs 真实的消融。同样未构造 InstructAV2AV 式的成对编辑数据。[部分不确定：合成数据的生成方式、占比与影响]

### [Movie Gen](../models/Movie_Gen.md)

大量且体系化地使用合成数据，但集中在个性化与编辑两个能力上，T2V主干仍用真实数据：
· 个性化（PT2V）cross-paired 合成：直接用paired数据训练会让模型学到copy-paste捷径（生成人物总是复刻参考图的表情与头部姿态、直视镜头）。除收集 O(10)K 条真实cross-pair（来自预训练数据中同一场景不同机位的片段）外，用一个预训练的个性化图像生成模型（He et al., 2024b）对paired数据每条视频的首帧生成参考图，通过多样prompt变化表情、头部姿态、光照等；用 ArcFace 相似度<0.7 剔除身份漂移的生成图，最终得到 O(1)M 条合成 cross-paired 样本，在预训练第三阶段专门用于提升自然度。
· 视频编辑（Movie Gen Edit）三阶段合成：论文的核心主张是「不依赖任何有监督视频编辑数据」也能获得SOTA视频编辑能力。阶段一多任务训练，把图像编辑当作单帧视频编辑与文生视频交替训练；阶段二引入两个合成的多帧编辑任务——Animated Frame Editing（对图像编辑样本对施加随机仿射变换生成「动画化」的多帧编辑对）与 Generative Instruction-Guided Video Segmentation（生成式指令引导视频分割），并与文生视频多任务微调约一千步；阶段三把NLP的 backtranslation 思想迁移到视频编辑：用阶段二模型生成编辑后视频，过滤后构造 (带噪的生成编辑视频 x̂, 反向编辑指令 c_instruct-bwd, 干净的真实视频 x) 三元组，训练模型从带噪编辑结果还原干净的真实视频，从而在多帧、高质量的真实输出视频上训练。消融显示 backtranslation 明显优于标准微调。

### [NVIDIA NeMo Curator](../models/NeMo_Curator.md)

curation 框架本身不做视频合成数据构造，但相关能力在演进：
【视频侧】无受控扰动/编辑构造训练对的 stage（无类似 InstructAV2AV 的编辑配对构造）。Cosmos WFM 的训练数据中包含约 4% 的「合成渲染（synthetically rendered）」视频，但那是外部输入的素材，不是 pipeline 生成的。
【文本/多模态侧】26.04 起集成 NeMo Data Designer（合成数据设计工具）；26.07 新增 Nemotron OCR 合成数据流水线，可从已有数据集生成 OCR 训练记录，并用 Nemotron Nano Omni 做可选质量打分；26.07 还引入 Nemotron-CLIMB 数据配比优化工作流（对文档做嵌入—聚类—配比搜索，从 64 个候选配比中筛选出最终方案）。
可见 NVIDIA 正把「合成数据生成」与「数据配比自动优化」纳入 Curator 版图，但目前仅覆盖文本与 OCR，视频模态尚未跟进。

### [OmniHuman 数据集 + OHBench](../models/OmniHuman.md)

【无】数据集完全由真实采集的 YouTube 视频构成，未使用任何合成或受控编辑构造的训练对：视频侧无生成/编辑样本、音频侧无 TTS 合成内容（对比 UniTalking 用 IndexTTS2 合成 690 万条参考音频）、无受控扰动构造的编辑指令对（非 InstructAV2AV 路线）。
【唯一的「构造」性操作：源分离产生的派生轨道】Demucs 四源分离把原始混合音轨拆为人声轨与背景轨，这产生了原始数据中并不独立存在的两条派生信号。严格说这是信号处理层面的分解而非数据合成，但它在功能上确实构造出了新的训练配对可能性——例如「人声轨 + 视频」可训练配音模型、「背景轨 + 视频」可训练环境音生成、「视频 → 人声+背景」可训练全音景生成。这是一种低成本、零幻觉风险的数据增益方式，与 TTS 合成路线相比不引入训练-推理域差。
【帧裁剪也带来轻微的分布改变】去除字幕/台标的帧裁剪操作改变了部分样本的构图与宽高比，属于预处理而非合成。
【设计取向的判断】本数据集的定位是「真实世界复杂物理场景」（complex, real-world physical scenes），刻意追求真实分布，因此不引入合成数据在立意上是自洽的——合成数据会引入分布偏移，与其针对「场景多样性不足」的问题诊断相悖。

### [Open-Sora 系列](../models/Open-Sora.md)

两个项目均**不使用模型合成的视频数据**，训练视频全部来自真实采集的公开数据集。存在的「构造式数据」仅限于两类，且都不是视频合成：
1. **掩码构造的多任务训练对（Open-Sora Plan v1.3 Image Controller）**：在同一批真实视频上通过**不同的帧掩码模式**构造出 T2V、I2V（首帧条件）、Transition（首尾帧条件）、Continuation（视频续写）四类任务样本，第一阶段共 50M 样本分 7 个渐进子步，帧保留率从 50% → 25% → 12.5% 逐步降低，任务配比逐步向 I2V（40%）与 Transition（40%）倾斜；第二阶段 15M 高质量数据延续该配比。这是用掩码从单一真实数据派生多任务监督信号的典型做法，不涉及生成模型。
2. **caption 层面的合成（Open-Sora Plan prompt refiner）**：用 ChatGPT 把 19,500 条真实来源 caption 改写为统一格式的「短 prompt → 长 prompt」配对，属文本侧合成。
此外 Open-Sora 2.0 在推理侧构建了 text-to-image-to-video 流水线（先用 FLUX 文生图再图生视频），但这是推理策略，不是训练数据合成。

### [Ovi](../models/Ovi.md) ⚠️

[不确定]。论文未描述任何合成数据构造环节：无受控扰动构造训练对、无编辑式配对数据（如 InstructAV2AV 类）、无 TTS 合成语音回灌、无音画错配负样本构造、无音轨替换/移位的数据增强。所有音视频训练样本均为真实采集的原生配对数据。
唯一的「人工构造」是数据格式层面的处理：纯音频微调数据 padding 到精确 5.04 秒以匹配 121 帧视频时长；以及推理侧推荐用 GPT 按训练 caption 模板扩写用户 prompt（属于推理期而非训练期的合成）。

### [Script-a-Video](../models/Script-a-Video.md)

论文未构造任何合成数据：无受控扰动构造训练对，无编辑式数据增广（不存在 InstructAV2AV 式的音视频编辑指令对），无 TTS 合成语音，无音效叠加混音，无生成模型自产数据的自蒸馏。全部训练数据均为真实采集的影视/生活类音视频片段。
【唯一的「合成」成分是标注而非内容】500K 数据集的全部 MTSS 标注由 Gemini-2.5-Pro 自动生成，属合成标注（synthetic annotation）。这构成一条标准的模型蒸馏链路：闭源强模型标注 → 开源模型 SFT → 得到成本更低的专用标注器。
【最接近数据构造的设计】MTSS 本身可视为一种「表示层的数据重构」——同一段视频在单体式 caption 与 MTSS 脚本两种表示下构成一组天然的对照数据对，论文正是用这组对照（LTX-2-AV vs LTX-2-AV-MTSS，同架构同训练、唯一变量是提示词结构）完成核心论证。但这是评测设计而非数据合成。
【可编辑性的潜在合成价值（论文未实践）】MTSS 的局部可编辑特性理论上非常适合构造受控扰动对——只改 Event 流中一条音效的 description、或只改某个 shot 的 camera 字段，即可得到一组语义差异受控的训练对，而无需重写全文。论文提出了这一结构优势但未将其用于合成数据构造，属于该 schema 尚未被挖掘的应用方向。

### [Seedance 2.0 与 Seedance 1.5 pro](../models/Seedance_20_Seedance_15_pro.md) ⚠️

[不确定] 未披露合成数据构造流程。唯一相关的是 RLHF 环节：Seedance 1.0 明确将「模型不同训练阶段生成的合成视频（synthetic videos generated by different stages of our model）」纳入人类偏好标注的数据对来源，实验表明多来源视觉素材能提升 RM 的领域容量、扩展偏好上界并增强泛化。这属于偏好数据合成而非受控扰动式的编辑训练对构造。Seedance 2.0 具备视频编辑与续写能力（针对指定片段/角色/动作/情节的定向修改、前后向时间轴续写），推理其训练需要成对的编辑前后数据，但构造方式未披露。

### [SkyReels 系列](../models/SkyReels.md)

SkyReels 系列在两代都重度使用合成数据，且用途各异，是本条目信息量较大的一项：
【SkyReels-V2】合成数据服务于运动质量：(1) 打标侧——生成1.6万条「运动轴均衡」的合成数据，补齐真实数据中稀缺的运动方向/类型，用于训练运动识别 captioner；(2) 强化学习侧——通过对真实视频施加「受控失真（controlled distortions）」自动构造负样本，与人工标注的偏好对共同组成 DPO 训练数据；失真样本覆盖 V2V、I2V、T2V 三种变体形态，形成「半自动偏好数据生产线（semi-automatic data production pipeline）」，这是「用可控扰动造训练对」的典型实践。
【SkyReels-V4】合成数据填补三类真实数据缺口：(1) 多语种画面文字——合成含中/英/日/韩/德/法等文字的图像视频，解决多语种字形渲染数据不足；(2) 多语种语音——TTS 合成语音补齐语种覆盖；(3) inpainting/editing 配对数据——论文明确指出「配对训练数据在真实数据集中天然不存在（inherently unavailable in real-world datasets）」，因此通过「视觉分割模型 + 图像/视频编辑模型 + 可控生成技术」组合的复杂流水线构造「编辑前/编辑后」配对样本，这是 V4 统一生成-修补-编辑能力的数据前提。具体使用的分割/编辑模型型号、构造样本量与质量校验方式未公布。

### [Sora 2](../models/Sora_2.md) ⚠️

完全未披露。System Card 中「information that our users or human trainers and researchers provide or generate」的「generate」一词可能涵盖人类训练师生成的内容，但是否包含模型合成数据、是否通过受控扰动/编辑构造训练对（如 InstructAV2AV 式的编辑配对），完全无信息。安全评测环节使用了「helpful-only 版本的视频模型」批量生成对抗样本输出用于构建自动化评测集，这是模型生成数据用于评测/安全对齐的明确案例，但不属于主训练数据合成。[不确定]

### [Step-Video-T2V](../models/Step-Video-T2V.md) ⚠️

训练语料侧未使用合成数据构造（无受控扰动/编辑构造训练对，如 InstructAV2AV 那类做法）。带「构造」性质的只有后训练两处，均为模型自采样而非语料合成：
· Video-DPO 偏好数据：对每条 prompt 用不同随机种子让 Step-Video-T2V 生成多个视频，组成候选集供人工偏好排序——属于模型自生成的偏好对；
· 蒸馏数据：为训练加速版 Step-Video-T2V-Turbo，采样约 9.5 万（95k）条数据构成蒸馏数据集。
此外 prompt 侧有轻度合成：DPO 的 prompt 集除从训练数据中随机抽取外，还邀请人工标注员按精心设计的指引「合成（synthesize）」新 prompt，以扩大 prompt 分布覆盖。[不确定]

### [UniTalking](../models/UniTalking.md) ⚠️

UniTalking 构造了一类明确的合成训练数据，且规模不小——这是其数据流水线中最具工程特色也最值得推敲的一环：
【构造动机】模型要支持 TR2AV（文本 + 参考音色 → 音视频）任务，需要「参考音频 + 目标音视频」的配对训练数据。但源数据集中每条视频只有其自身的音轨，不存在「同一说话人的另一段音频」作为参考——论文原文：「our source dataset lacks corresponding reference audio」。因此必须合成。
【构造方法】使用 IndexTTS2（零样本、情感可控、时长可控的自回归 TTS 模型）。对每条视频：
1) 输入一：视频的原始音轨，作为目标音色参考（timbre reference）；
2) 输入二：一段随机生成的文本 prompt——构造方式是「随机采样词与数字的序列」（randomly sampling sequences of words and numbers），即语义无意义的随机词串；
3) 输出：一段音色与原视频说话人一致、但内容完全无关的合成语音；
4) 时长约束：每条合成参考音频严格控制在 3 至 5 秒之间；
5) 数据增广：对每条视频重复该过程 3 次，生成 3 条不同的参考音频。按 230 万条视频计，共合成约 690 万条参考音频片段。
【设计意图剖析：为什么用随机词串而非有意义文本】这是本设计最关键也最巧妙的一点。若用有意义的文本合成参考音频，模型可能学会从参考音频中提取语义内容并泄漏到输出中，造成「参考音频的内容污染生成内容」的捷径学习。使用语义无意义的随机词串，使参考音频中唯一稳定可用的信息就是音色本身，从而在数据层面强制实现「音色与内容的解耦」。这是一种典型的「用数据构造消除捷径」的做法，比在损失函数或架构上做解耦更直接。
【设计意图剖析：为什么用 TTS 合成而非切分原音轨】更朴素的做法是从同一视频的音轨中切出另一段作为参考。但那样参考音频与目标音频来自同一次录制，共享录音环境、背景噪声、乃至部分内容，模型极易通过声学指纹匹配而非音色理解来「作弊」。经 TTS 重新合成则切断了这些旁路信息，只保留被 TTS 模型建模并重建出的音色特征。
【潜在风险】参考音色来自 IndexTTS2 的重建而非真实录音，其音色保真度上限即为 IndexTTS2 的克隆能力上限；同时合成语音特有的声学统计特征（TTS artifacts）会成为参考条件的固有分布，导致训练-推理不一致——推理时用户提供的是真实录音，与训练时见到的 TTS 合成音存在域差。这可能正是 TR2AV 音色相似度仅 0.66–0.70、落后于专业 TTS 方法的原因之一。论文未讨论这一域差问题，也未做「合成参考 vs 真实参考」的消融。[不确定]
【视频侧无合成】未使用任何合成或编辑构造的视频数据，无受控扰动训练对（非 InstructAV2AV 式）。
【另一处广义的合成】阶段一音频预训练使用的「内部 TTS 数据」性质不明——若为 TTS 训练语料则为真实录音，若为 TTS 生成语料则为合成数据，论文未说明。[不确定]

### [UniVerse-1](../models/UniVerse-1.md)

论文未构造任何合成数据：没有受控扰动构造训练对，没有编辑式数据增广（如 InstructAV2AV 式的音视频编辑指令对），没有 TTS 合成语音，没有音效叠加/混音合成，也没有用生成模型自产数据做自蒸馏。全部训练数据均为真实采集的原生音视频片段（且强制要求自带原生同步音轨）。
【最接近「构造」的两处设计，但均不属于数据合成】
1) 在线标注中的随机时间窗口采样——同一条源片段在不同 epoch/不同步会被采样到不同的 5 秒窗口并配以不同的即时标注，客观上起到了数据增广作用，但增广的是「窗口位置 + 标注」而非合成新内容；
2) 独立噪声采样策略（INSS）——为视频与音频两个模态独立采样扩散噪声，避免共享 PRNG 状态产生虚假跨模态相关性。这是训练侧的噪声构造技巧，不涉及数据内容合成。消融显示其影响显著（去掉 INSS 后 FD 从 1.25 恶化到 1.43、KL 从 2.70 恶化到 3.51、CLAP 从 0.16 掉到 0.11、LSE-C 从 1.34 掉到 0.99），说明跨模态噪声去相关对音频质量与唇同步都很关键。

### [Unison](../models/Unison.md) ⚠️

论文未构造任何合成数据：无受控扰动构造训练对，无编辑式数据增广（如 InstructAV2AV 式的音视频编辑指令对），无 TTS 合成语音，无音效叠加混音合成，无生成模型自产数据的自蒸馏，无视频重渲染或换脸类构造。全部训练数据均为真实采集的原生音视频与音频。
【最接近「构造」的两处操作，但均不属于数据合成】
1) Mel-RoFormer 音源分离 —— 把真实混合音轨拆解为语音与音效两条流作为双流监督的 ground-truth。这是对真实数据的分解（decomposition）而非合成（synthesis），产出的两路信号都是真实音频的成分，不含任何生成内容。但它确实创造了原始数据中不存在的监督信号形态——原始数据只有混合音轨，「纯净语音轨」与「纯净音效轨」是分离模型的产物，因此这两路 ground-truth 的质量上限受限于 Mel-RoFormer 的分离能力，分离残留（如语音轨中的音效泄漏）会成为训练噪声。论文未讨论分离误差对双流监督的影响，也未给出分离质量的量化评估，这是双流架构的一个潜在薄弱点。[不确定]
2) 跨模态 forcing 的异步噪声采样 —— 为视频与音频独立采样 timestep，构造出「一个模态干净、另一个模态含噪」的训练配置，并动态指定学生模态。这是训练侧的噪声调度构造，不涉及数据内容合成，但它确实人为制造了原始数据中不存在的「噪声不匹配」训练场景，功能上类似一种数据增广。消融证明其价值显著：移除 CMFS 后 DS 从 0.08 恶化至 0.19（全部消融中最差）、LSE-C 从 3.30 跌至 3.02、VA 从 4.02 降至 3.91。
【值得记录的现象】移除 CMFS 不仅损害对齐，还损害视频质量（VA 4.02→3.91），论文对此的解读是「视频分支受益于音频流的引导以强化视觉连贯性」——即在视频骨干冻结的前提下，音频信号仍能通过融合模块反向改善视觉输出的连贯性。这是跨模态互训价值的一个直接证据。

### [Veo 3 / Veo 3.1](../models/Veo_3_Veo_31.md) ⚠️

可确认的合成仅限于文本侧：合成 caption（synthetic captions）由 Gemini 生成，用于扩展与视频关联的概念多样性。[不确定] 未披露任何视觉/音频侧的合成数据构造，未提及通过受控扰动、编辑对、指令式音视频编辑对（如 InstructAV2AV 式构造）来生成训练对；Veo 3.1 具备 Insert/Remove 物体插入删除、首尾帧过渡、outpainting、风格参考等编辑类能力，这类能力通常需要成对的编辑前后训练样本，很可能存在合成构造流程，但官方无任何说明。

### [Vidu S1](../models/Vidu_S1.md) ⚠️

[不确定]。论文未提及使用合成数据或通过受控扰动/编辑构造训练对。训练侧的「人造监督」体现在训练策略而非数据构造上：第二阶段用 Teacher Forcing 与 Diffusion Forcing 混合策略做因果适配，第三阶段用 DMD（Distribution Matching Distillation）+ PCM（Phased Consistency Models）正则做少步蒸馏，由双向教师模型为学生提供分布匹配监督。

### [主流视频预训练数据集合并调研：Panda-70M、InternVid、Koala…](../models/pretraining_datasets.md)

**七者全部不使用模型合成的视频数据**，训练视频100%为真实采集的网络视频或素材站素材。存在的「构造式数据」仅限于文本侧与标注侧：
- **Koala-36M（唯一的自监督数据构造，最值得复用）**：为训练Color-Struct SVM转场检测器，**用规则构造正负样本对——同一视频内的帧对为负例（无转场）、不同视频源的帧对为正例（有转场）**。这是把「拼接不同视频」当作转场的合成代理，构造成本极低且完全不需要人工标注。
- **LVD-2M（caption层面的多轨道改写）**：同一视频保留三条caption轨道——LLaVA-v1.6-34B的`raw_caption`(215.4词)、Claude3-Haiku精修的`refined_caption`(81.8词)、LLaMA-v3.1-70B改写的`rewritten_caption`(41.3词)。**这实质上是为同一视频合成了长/中/短三种prompt分布**，可直接用于缓解训练-推理prompt错配，是七者中唯一提供多粒度prompt配对的数据集（且第三条轨道论文未提及，只存在于发布数据中）。
- **MiraData**：GPT-4V打标时把Panda-70M的短caption作为hint注入，属文本侧级联生成而非合成数据。
- **UltraVideo**：Qwen3-4B把9个子caption整合为第10个汇总描述，属文本侧合成；另用LLM从108个主题**生成检索关键词**再人工搜索，属数据召回策略而非数据合成。
- **Panda-70M**：教师caption经UMT择优后用于蒸馏学生模型，属知识蒸馏而非数据合成。
- **InternVid / OpenVid-1M**：无任何数据构造环节。
**共同缺口**：七者**均无受控扰动/编辑构造的训练对**（如InstructAV2AV式的编辑前后配对），也无任何合成引擎（Unity/UE/Blender）渲染数据——MiraData的「3D引擎渲染场景」类目是**从YouTube收集的他人渲染成品**，不是自建渲染管线。

## 人工介入程度（人工标注、人工质检、模型初筛+人工复核）

`human_in_loop` · 详细程度: brief

### [Allegro](../models/Allegro.md)

数据 pipeline 全自动，无人工标注与人工质检环节：从切分、7 级过滤到 caption 生成全部由工具与模型完成，论文未提及任何人工复核、抽检或标注团队。
人工介入仅出现在评测环节：用户研究采用 46 条覆盖多样场景的 prompt、六个评价维度（视频—文本相关性、外观畸变、外观美感、运动自然度、运动幅度、整体质量），每对视频由 2 名标注员评分，累计 5,448 条评分。此外 VBench 定量评测使用 946 条 prompt 生成 4,730 条视频。

### [Apollo](../models/Apollo.md) ⚠️

人工介入程度极低且披露模糊，全流程唯一一处人工痕迹在训练末端而非数据 pipeline 中：
【数据 pipeline】完全自动化。论文明确定位为「a novel automated data-construction pipeline」（新颖的自动化数据构造 pipeline），81M 样本全部为「automatically annotated samples」（自动标注样本）。过滤阈值如何确定（是否像 MOVA 那样由人工抽检不同 cutoff 下的留存样本来标定）未说明；标注结果是否做人工抽检验证未说明；未提及任何人工标注、人工质检或模型初筛+人工复核环节。
【训练末端】Stage III 使用「the manually-curated, high-quality dataset」（人工精选的高质量数据集）做最终微调——这是全文唯一的人工介入表述，但该数据集的规模、人工筛选标准、参与人数、标注规范全部未披露。
【结论】Apollo 的定位是「自动化 pipeline 做大规模、人工精选做末端提纯」的两段式，人力集中投放在最后一公里，但这一公里的所有细节都是黑箱。[不确定]

### [CineDance / CineDance-1M](../models/CineDance.md) ⚠️

人工介入定位为「阈值标定 + 质量抽检 + 基准构建」，不参与大规模标注生产（标注全部由大模型完成）：
【① 阈值标定】通过人工参考集测得叙事完整性的经验最小时长为 18.4 秒，据此把抗碎片化软阈值设为 20 秒——这是人工先验直接决定 pipeline 参数的典型案例。
【② 伪影人工审计】随机抽取 500 条片段，由三名独立标注员分别审查残留伪影（烧录字幕、logo、水印、电视网叠加层、片名卡、片尾字幕、录屏、转场特效、静帧停留等），分歧由联合复议（joint review）裁定，得出 2.8% 不合规率并与 Koala-36M 的 37.4% 对照。
【③ 人工标注基准】构建 100 条人工标注片段的小型基准，用于评估 ASR-to-Character 绑定准确率（95.4%）与各类 diarization 基线的对比。
【④ CineBench 人类评测】每条视频由 10 名独立评测员打分，采用 5 点 Likert 量表（1=不可用，5=优秀），双盲、随机呈现顺序，再用 Spearman 秩相关验证自动指标与人类判断的一致性。
【模式总结】属于「大模型全量标注 + 人工小样本校验与对齐」的现代范式，人工成本集中在验证侧而非生产侧。人工标注员的招募方式、人数总量、报酬等未披露。[不确定]

### [CogVideoX](../models/CogVideoX.md) ⚠️

人工介入集中在「训练过滤器的种子标注」这一个点上，整体属于「少量人工标注 → 训练模型 → 全自动大规模筛选」的经典范式：
· 过滤器种子标注：人工采样并标注 20,000 条视频的正/负质量标签（覆盖 6 类负面标签），其中随机 10% 作为测试集用于报告分类器准确率。这是全流程唯一明确的人工标注环节。
· 负面标签体系本身由人工设计（论文列出 6 类定义并在图 16 给出每类的样例视频）。
· 打标环节无人工：caption 全部由模型链自动生成，未提及人工精修或复核（对比 Movie Gen 在 SFT 阶段由标注员逐条重写 caption）。
· 高质量微调子集（20%）的挑选依据未说明是自动分数还是人工筛选 [不确定]，从行文推测为自动阈值筛选。
· 评测端有重度人工介入：100 条精心设计的 prompt，人工评审按 0–1 打细项分、0–5 打总分，并规定「若未遵循指令则总分不得超过 2」；评测维度为感官质量（Sensory Quality）、指令遵循（Instruction Following）、物理模拟（Physics Simulation）、封面图质量等，各维度均给出三档（1 / 0.5 / 0）的详细评分细则（附录 J）。致谢中专门感谢了 data annotators。

### [Cosmos-Predict2.5](../models/Cosmos-Predict25.md) ⚠️

数据清洗环节完全自动化，人工介入集中在评测与后训练选型两处，属于「人不进数据流、人进决策环」的模式：
【数据侧：几乎无人工】七阶段 pipeline 全部由模型与规则驱动，无人工标注或人工复核描述。唯一间接的人工痕迹是机器人数据中沿用了原数据集自带的人工标注——GR00T 的「human-labeled success ratings」（人工标注的成功评级）被注入 caption prompt；以及策略学习用的 100 段人类遥操作演示（human teleoperation demonstrations）是人工采集的。
【后训练数据侧的人工筛选】论文在引言中明确「manually curate specialized post-training data tailored for Physical AI」——为 Physical AI 手工精选专项后训练数据，但未说明人工筛选的规则、工作量与人员规模。
【评测与选型侧：大量人工】(1) 每个 SFT 域构建领域专属测试集并做 human preference study（人类偏好研究），以 SFT Win / Base Win / Tie 三分统计；(2) 模型合并环节：先在「a small, hand-picked set of challenging examples」（人工挑选的小规模困难样本集）上做质量评估从 20+ 候选合并模型中选优，再在更大评测集上用 human preference voting 验证；(3) RL 前后做人工投票对比；(4) 与 Wan2.1/2.2 的对比也用人工 win ratio，标注员按 realism（真实感）、visual quality（视觉质量）、temporal consistency（时序一致性）、alignment with conditioning inputs（与条件输入的对齐）四个准则做成对比较。
标注员人数、每样本评价人次、一致性指标均未披露。[不确定：人工评测的标注员规模与规范]

### [Data-Juicer 2.0](../models/Data-Juicer.md) ⚠️

Data-Juicer 的设计取向是「尽可能减少人工介入，用模型反馈替代人工判断」，人工的角色被限定在两个环节。
【人工介入点一：配方设计与阈值决策的最终裁定】DJ 不预设任何默认阈值，所有算子超参外置于 YAML。但 Sandbox 的核心贡献恰恰是把这个环节从「人工凭经验设阈值」改造为「用小规模参考模型训练 + 基准评测的反馈自动排序」——人只需要指定候选算子集合与评测基准，具体保留哪一段分布由实验数据决定。这是本调研中唯一把阈值确定过程系统化、自动化的工作。
【人工介入点二：交互式操作与可视化审查】接口层提供多级人机交互通道：低层 Python API（工程师编程）、RESTful 端点（服务化调用）、可视化编辑器（阿里云 PAI Designer 组件，拖拽式配置数据处理流程）、基于 AgentScope 智能体的自然语言对话式指令（用一句话描述数据处理需求）。v1.4.6 引入的「Q&A Copilot」进一步降低使用门槛。此外 DJ 提供数据分析报告与追踪器（Tracer，v1.4.6 支持 Ray 模式），可让人查看每个算子实际剔除了哪些样本——这是人工抽样质检的支撑设施，但质检本身由人自行进行。
【规模化的群体参与】DJ 2.0 论文提到其支撑了3000+团队参与的数据过滤/合成竞赛，属于一种众包式的配方探索，但不是逐样本的人工标注。
【无人工标注环节】DJ 全部标注能力均由模型算子提供，不含任何人工标注工作流、标注平台对接或标注质量抽检机制。
[不确定] 官方 T2V 案例中未披露是否有人工复核环节；DJ 也未提供标注一致性检验、人工-模型标注对比等质量保障工具。

### [Foley-Omni](../models/Foley-Omni.md) ⚠️

人工介入程度低，且集中在阈值标定与评测集把关两处，训练数据标注全自动。
【阈值标定环节】声学后验证的 −35 dB RMS 能量门限通过「manual inspection of a small validation subset」（对小规模验证子集的人工检视）确定。这是典型的「人工定标、机器执行」模式——人只参与超参数选择，不参与逐样本判断。Table 7 中其余六项阈值（480p、1 Mbps、[0.1,3.2]、0.6、0.3、0.2）的确定依据论文未说明，推测同样有经验性的人工调参成分。
【评测集人工复核】V2ST-Bench 的300条片段经人工审核，审核维度明确为三项：audiovisual consistency（音视频一致性）、annotation accuracy（标注准确性）、suitability for mixed soundtrack evaluation（是否适合作混合配乐评测）。这属于「模型初筛 + 人工复核」模式，但仅施加于300条评测样本，规模极小。
【主观评测】使用 A-MOS / S-MOS / T-MOS 三项主观意见分，需要人类听测评分。[不确定] 未说明评测者人数、招募方式、评分细则与一致性检验。
【训练数据】约2.0M条训练标注全部由 Gemini + Bandit 自动产出，无人工复核环节，也未提及抽样质检的准确率报告。

### [Goku](../models/Goku.md) ⚠️

论文中**未提及任何人工标注、人工质检或人工复核环节**，整条五阶段流水线呈现为全自动、纯模型与阈值驱动的设计（PySceneDetect + DINOv2 + 美学模型 + OCR + RAFT + 分类模型 + VLM/LLM 打标）。
可能存在但未披露的人工介入点包括：内部视频分类模型的 9 大类/86 子类类目体系设计与训练标注、6000 万内部高质量图像的「高质量」判定标准、各级阈值（4.3/4.5、0.85/0.90、0.02/0.01、运动分数上下限）的确定过程——这些阈值的选取本身几乎必然经过人工试错与抽样目检，但论文未展开。此外论文的主观质量对比（Figure 5 等）涉及人工评判，但属于评测而非数据流程。[不确定]（实际人工投入程度）

### [HunyuanVideo-Foley](../models/HunyuanVideo-Foley.md) ⚠️

人工介入程度接近于零——论文摘要明确将 pipeline 定性为「通过自动化标注」（through automated annotation）构建：
【训练数据侧全自动】七级清洗漏斗（音轨检测 → 场景切分 → 静音占比 → 带宽检测 → AudioBox+SNR → ImageBind+AV-align → 分类标注）与 caption 生成（GenAU）全部为模型与阈值的自动判定，论文未提及任何人工标注、人工抽检、人工复核环节，也未说明各项阈值是如何标定的（未提人工验证、未提消融实验）。[不确定：阈值标定方法]
【规模的必然性】10 万小时约 4,500 万个 8 秒片段，即便按每片段 5 秒的人工审听计算也需约 6 万人时，人工全量介入在经济上不可行。这与其「可扩展 pipeline」的定位一致——可扩展性的前提正是无人工瓶颈。
【唯一的人工介入在评测侧】主观评测环节使用人工听测，产出 MOS-Q（音质）、MOS-S（语义对齐）、MOS-T（时序对齐）三项主观意见分，在 MovieGen-Audio-Bench 上分别达到 4.14±0.68、4.12±0.77、4.15±0.75（5 分制）。论文未披露评测者人数、招募方式、听测环境（是否使用监听耳机）与打分指引——这些对 MOS 的可信度相当关键。[不确定]
【资源分配取向】与 UniVerse-1 相同，属「训练数据全自动、评测半人工」的分配模式，人力集中投入到效果验证而非数据质量保证上。

### [HunyuanVideo](../models/HunyuanVideo.md)

两代都把人工放在漏斗最末端（模型初筛 + 人工精选）而非全程标注：
【原版】SFT 数据集约 100 万样本全部经人工标注筛选（manually annotated），标注员按两大类共七项维度评判：美学侧——色彩和谐（color harmony）、光照质量（lighting）、主体突出（object emphasis）、空间布局（spatial layout）；运动侧——运动速度（motion speed）、动作完整性（action integrity）、运动模糊（motion blur）。目标是选出「视觉美观且运动细节丰富」的片段。
【1.5】人工介入分布在三处：(1) SFT 数据集最终由人工标注（manual annotation）构建；(2) I2V RLHF 的 prompt 与配图经人工核验图文一致性；(3) T2V DPO 的偏好对由人工做 GSB（Good/Same/Bad）标注；(4) 评测环节动用「100+ 名专业评估人员」对 300 条文本 prompt 与 300 张图像样本做 GSB 人评。未披露标注团队规模与人时成本。

### [InstructAV2AV](../models/InstructAV2AV.md) ⚠️

人工介入程度低且高度聚焦，集中于评测集把关与用户研究两处，79K 训练数据的构造与验收全自动。

【评测集人工验收（唯一介入训练/评测数据构建的环节）】组织方式披露相当具体：20 名志愿者被编为 10 个评审对（judge pairs），每个候选样本由 5 个独立的评审对评估，每个评审对只负责五项标准中的一项；样本必须通过全部五项才能进入 1K 评测集。
  这一设计有两处值得记录：(1) 「一对评委只看一个维度」的分维度独立评审，避免了单人对多维度打分时的光环效应与维度间相互污染，比常见的「一人给所有维度打分」更严谨；(2) 采用「对」而非单人作为评审单元，隐含了对内一致性的双人校验。
  [不确定] 未说明评审对内部意见分歧时如何裁决、未报告评审者间一致性（inter-rater agreement / Cohen's kappa）、未说明志愿者的背景与培训方式、未报告从多少候选中筛出 1K。

【用户研究（评测环节，非数据构建）】25 名志愿者，对每个数据集各随机抽取 20 个样本，沿三个维度做偏好选择：音视频同步（AVS）、文本对齐（TA）、整体偏好（OP）。结果：InsAVE-80K 上 AVS 49.00% / TA 45.40% / OP 46.60%；AvED-Bench 上 AVS 46.80% / TA 41.80% / OP 43.60%——均为四方比较中的最高偏好率（四个方法均分则每个约 25%，故 40%+ 属显著领先）。[不确定] 未说明志愿者招募方式、是否与评测集验收的 20 人重叠、评分细则与一致性检验。

【训练数据零人工】79K 训练对完全由 Qwen3-Omni 自动验收，无人工复核、无抽样质检、未报告自动验收结果与人工判断的吻合率。这是可扩展性的代价，也是「可扩展数据合成pipeline」这一定位的必然选择——但缺少一次「在 1K 人工集上对比自动与人工结论」的交叉验证实验，使自动验收的可靠性无法量化，是一处可惜的方法论空白。

【人工在阈值标定中的角色】[不确定] CoTracker3 运动阈值、LAION 美学阈值、Audiobox 阈值、MLLM 打分门槛的确定依据均未说明，推测有经验性人工调参成分，但论文未如 Foley-Omni 那样明说「通过人工检视小规模验证子集确定」。

### [2026 其他音视频联合生成](../models/JAVG_2026_misc.md) ⚠️

本批工作中人工介入程度差异显著，ALIVE 是唯一有实质人工标注投入的：
【ALIVE —— 两类人工标注 + 人工评测（本批投入最重）】
(1) 人工修订 caption：caption 模型的 SFT 训练数据为「manually revised caption data」（人工修订的 caption 数据），经两轮 SFT。这是「大模型初标 + 人工复核修订 + 蒸馏成专用模型」的标准范式，人工成本一次性投入、收益规模化。
(2) 人工标注 sub-motion units 时间戳：「decomposing sub-motion units to annotate the start and end timestamps of each complete small action」——把动作分解为子动作单元，人工标注每个完整小动作的起止时间戳。这是本批中成本最高的人工标注项（细粒度时序标注单位成本远高于打描述性标签），说明团队高度重视时序精度。
(3) 两类标注的规模（标注条数、标注员数量、成本）均未量化[不确定]。
(4) 人工评测：与 Veo 3.1、Kling 2.6、Wan 2.6、Sora 2、LTX-2 做盲测 side-by-side 对比；Alive-Bench 1.0 的 264 general prompts + 90 reference-character prompts 与 22 项细粒度指标（六大类）应含人工设计与评判成分[评判方式细节不确定]。
【StreamChar —— 人工评测明确量化】「24 participants, each presented with 50 randomly sampled cases」——24 名参与者、每人评 50 个随机抽样案例。这是本批中唯一给出人评参与者数与样本数的工作。数据侧无人工标注[不确定]。
【OmniCustom】数据管线全自动（SyncNet + 美学分 + 时长三条规则 + GLM-ASR），无人工标注环节。评测集构建有人工介入：精心挑选 100 例（30 位训练集外真人 + 70 段 YouTube 视频），并控制性别比 1:1——这个 1:1 的配比必然是人工筛选的结果。
【NAVA】数据管线高度自动化（全部依赖 OCR / VideoCLIP / 各类打分器 / VLM / LLM），无人工标注描述[不确定]。160K 高质量 SFT 子集由「multi-operator collaborative filtering」自动筛出而非人工挑选。评测含 user studies（用户研究），评估视频质量、音视频同步、音频保真度、音色可控性四个维度，参与者规模未说明[不确定]。
【CCL / Baton】无人工介入描述[不确定]。
【ITS-JAVG】无训练数据故无标注；其核心方法论恰恰是「用模型验证器替代人工判断」——多验证器组合 + ARW 自适应加权，本质上是在自动化地逼近人类偏好。但论文同时揭示了这条路的风险（verifier hacking），间接论证了人工评测不可完全替代。
【规律】人工投入量与数据披露详细度正相关：ALIVE 数据披露最详、人工投入最重；反之 CCL/Baton 披露最少、无人工描述。这可能反映了「重视数据的团队才会既投入人工又详细披露」的一致性。

### [音视频联合生成基线合集](../models/JavisDiT_baselines.md) ⚠️

人工介入普遍集中在评测端而非数据端，是本合集的共性特征：
【MM-Diffusion（评测端人工投入最大）】开展了大规模图灵测试（Turing test），累计收集 1 万次人工投票，结果显示超过 80% 的生成音视频片段能骗过人类评估者。数据端未描述人工标注或质检环节[不确定]，但 Landscape 仅 1,000 条的规模与「high-fidelity」的自我描述暗示可能存在人工筛选。
【AV-DiT】未描述人工介入[不确定]。
【JavisDiT / JavisDiT++（数据端与评测端均有明确人工投入，本合集最充分）】
- 数据端：JavisBench 构建时对全部 YouTube 来源内容做「严格的人工法律与伦理审核」——这是本合集中唯一明确的数据端人工审核环节。
- 指标验证端：为验证 JavisScore 指标的有效性，构建了一个含 3,000 条样本的人工标注评估数据集，用人类对同步性的判断来校准指标。这种「先造指标、再用人工标注验证指标本身」的做法在评测方法学上较为规范，但标注者数量、招募方式、标注一致性（inter-annotator agreement）均未披露[不确定]。
- 模型评测端：JavisDiT++ 用 JavisBench 中的 100 条 prompt，由 3 名志愿者做盲测的 win-tie-lose 偏好判断。
【Harmony】做了系统性的人工消融研究（human ablation studies）验证各组件贡献；数据端的人工介入方面，其数据源中 AudioCaps（约 128 小时）与 Clotho（约 31 小时）是人工标注的音频 caption 数据集，而 WavCaps（约 7,600 小时）为自动标注——论文对三者的人工/自动标注属性做了明确区分，说明团队对标注质量分层是有意识的。自有 400 万条片段的标注则完全依赖 Gemini，无人工复核描述[不确定]。
【UniAVGen】未描述任何人工介入环节[不确定]。
【共性观察】五者的数据 pipeline 基本都是全自动的，人工主要用于评测与指标校准，而非逐条数据审核——这与工业级模型（往往有专职数据标注团队）形成对比，也是学术工作的现实约束。

### [可灵 3.0 / 可灵视频 3.0 Omni](../models/Kling_30_Omni.md)

以“模型自动筛选为主 + 人工介入于关键环节”为基本形态。已确认的人工介入点：后训练阶段的 DPO 偏好数据由人类评估同一MVL条件下的多个视频变体并给出偏好排序（Kling-Omni 报告明确为 human-evaluated）；数据侧另有“人工精选资源（manually curated resources）”补充数据来源。同团队 Kling-Foley 的 Kling-Audio-Eval 完全由人工标注/校对：20,935个样本，标注员在模型预生成caption基础上做修正，并核验标签是否属于预定义体系、核验音画质量与对齐（有效样本规则：前景音无人声、音效来自画面可见物体/动作、视频≥5秒、音效≥2秒、无含人声的版权背景音乐）。[不确定：大规模训练数据是否有人工抽检比例与SFT精选集的人工标注规模]

### [LTX-2](../models/LTX-2.md)

人工介入集中在质量评判器的构建环节，而非逐样本标注：
(1) 美学模型的标注：数万（tens of thousands）图像对由人工标注「哪一张美学更优」，用于训练 Siamese 排序网络；上游先用多标签网络对数百万样本自动打标以约束配对采样范围。这是典型的「人工标少量偏好对 → 训练自动打分器 → 全量自动过滤」的杠杆式设计。
(2) 逐样本 caption：完全自动，由内部 captioner 对全量训练集 re-caption，未提及人工复核或抽检流程。
(3) 评测环节：使用人类偏好研究（human preference study）评估视觉真实感、音频保真度与时序同步（唇同步、foley 准确度），LTX-Video 阶段的人评使用1000条文生视频 prompt 与1000对图生视频样本。
数据清洗环节是否有人工兜底质检，未披露。

### [LongCat-Video](../models/LongCat-Video.md) ⚠️

训练数据侧的人工介入几乎未被提及——数据 pipeline 通篇为自动化的模型打标 + 阈值过滤，无人工标注或人工复核环节的描述。
人工介入明确存在于两处下游环节：
(1) 评测：内部 T2V 评测集中 500 条 prompt 专供人工评测，采用 MOS（Mean Opinion Score）绝对打分与 GSB（Good-Same-Bad）相对比较双轨制，每条视频由 3 名独立标注员评价（three independent annotators）；I2V 评测集 400 条同样含人工评测。
(2) 奖励模型训练：Motion Quality 与 Text-Alignment 两个奖励模型均为「VideoAlign-based model fine-tuned on internal annotated datasets/internally annotated data」——即基于内部人工标注数据集微调，说明存在人工偏好/质量标注的投入，但标注量、标注规范、标注员规模与一致性指标均未披露。
可以概括为：人工不参与数据清洗，而是通过「标注奖励模型」的方式间接把人类偏好注入训练。[不确定：人工标注数据的规模]

### [MOVA](../models/MOVA.md)

人工介入程度很低，标注环节完全自动化，人工只出现在“阈值标定”与“最终评测”两处：
【阈值标定（唯一的数据侧人工介入）】“we manually inspect the videos retained under different metric cutoffs and set reasonable thresholds for each dimension accordingly”——研究者人工抽检不同 metric cutoff 下留存的视频，据此为音频质量、视频质量、音视频对齐三个维度分别设定阈值。附录亦重申这些阈值“determined by empirical observation”。这是典型的“人工定标 + 机器批量执行”模式。
【标注环节】零人工。视觉、语音、非语音三路标注与最终融合全部由 MiMo-VL / Qwen3-Omni ×2 / GPT-OSS-120B 完成，无人工复核或抽检的描述。
【评测环节的人工投入较重】Arena 式人类偏好评测收集了 5,000+ 有效投票，评测集 732 条（600 条 Verse-Bench + 132 条自建基准），其中一半原为纯英文的 Verse-Bench 语音数据由人工翻译为中文以构造双语混合集；评测者需在 prompt 遵循、视听同步、唇同步准确性、视频质量、音频语音保真度五个维度上做两两偏好判断；采用 ELO 评分（初始 1000，K=4，logistic scale 400，base 10，1000 次 bootstrap）。

### [合并条目：① Mochi 1](../models/Mochi_MAGI_Motif.md) ⚠️

① Mochi 1：数据侧无任何人工介入的披露。评测侧使用了自动化指标——官方沿用 OpenAI DALL-E 3 的 prompt adherence 评测范式，以 Gemini-1.5-Pro-002 作为评判模型（即用大模型代替人评）。[部分不确定：数据侧是否有人工质检]
② MAGI-1：数据管线全自动（PySceneDetect + 11 类 Filter Actor + 双模型去重 + MLLM 过滤与打标），未提及人工标注或人工质检。人工介入集中在评测环节且做得相当重：团队自建了一套内部人评基准——设计分层的指标体系（强调完备性优先于简洁性，并要求指标之间正交以避免冗余），组织为 Overall / Motion Quality / Instruction Following / Visual Quality 四个主维度并各自细分子指标；基准数据集由 100 组精选的图像—prompt 对构成，来源刻意多元化（现有视频生成平台的用户真实输入、FLUX 生成的合成图像、公开图库的真实照片、专业影视素材四类），每个样本按指标框架标注具体评测目标；评测采用严格双盲的成对比较（Win/Tie/Lose），评委为「具备良好审美训练的专家」。结论是整体优于开源的 Wan-2.1、明显优于 Hailuo(i2v-01) 与 HunyuanVideo、略逊于商业模型 Kling1.6(HD)，其中指令跟随与运动质量为强项、视觉质量仍有差距。
③ Motif-Video 2B：数据管线全自动，无人工标注/质检。但有一处「人在环」体现在训练调度而非数据标注上——SFT 语料的组装是迭代式的：跑中间评测 → 由人判断哪些 subject 类目最弱 → 针对性补数据 → 再评测。论文还把整个训练描述为「a diagnostic loop rather than a single forward pass through a predefined schedule（一个诊断循环而非一次性走完预定义课程）」，Stage 9 的 Shared Cross-Attention 中途插入正是因为人观察到 720p 阶段出现语义对齐回退。评测侧主要依赖 VBench 自动指标（Table 3，16 维），未报告人评。

### [Movie Gen](../models/Movie_Gen.md)

预训练阶段基本全自动（人工仅用于阈值确定，如音频CAVTP阈值「由人工检视确定」、美学阈值经人工验证），后训练阶段则是重度人工介入的「模型初筛 + 人工复核 + 人工重标」范式：
· 视频SFT四阶段中后两阶段全靠人工：第三阶段人工判断影视感（角度光/自然光或棚拍光、色彩鲜艳但不过饱和、画面不杂乱、有非平凡运动、无相机抖动、无编辑特效与叠加文字）——论文明确指出「高质量微调数据的许多方面无法被自动过滤器以高精度和高召回可靠捕捉」；同阶段标注员还要亲手把视频裁剪到目标时长，主动挑出整段中最精彩、最有感染力的片段。第四阶段人工在 LLaMa3-Video 生成caption的基础上修正错误细节、补齐关键信息（相机控制、人物表情、主体与背景、详细运动、光照），并新增6类镜别标注。
· 个性化SFT：从T2V微调集出发挑单人视频后，人工挑选动作多样的视频以覆盖多种动作行为，最终得 O(1000) 条。
· 音频SFT：影视感音视频分类器 + AED 自动过滤后，接人工标注做最终选择。
· 评测端几乎完全依赖人工A/B测试（论文论证了自动指标不可靠），标注员经过专门培训并持续审计；还把同一批381条prompt的标注任务重复4次以估计各评测维度的标准差（整体质量5.07%、帧一致性4.08%等），用于判断胜负是否显著。

### [NVIDIA NeMo Curator](../models/NeMo_Curator.md) ⚠️

框架定位为全自动流水线，无内置的人工标注/复核界面或工作流，官方文档未描述任何 human-in-the-loop 环节（无标注平台集成、无抽检采样 stage、无人工仲裁队列）。
人工介入实际发生在框架之外的两处：(1) 判别器的构建——Cosmos WFM 中的文字叠加检测 MLP、视频类型 taxonomy MLP、运动分类 ViT 都需要人工标注的训练集，但标注规模与流程未披露；(2) 阈值的人工设定——美学 3.5、运动 0.00098、DOVER 底部 15%、k=10000 等参数由工程师经验设定，框架只提供默认值。
框架在可观测性上做了替代性设计以降低对人工的依赖：被过滤的 clip 移入 video.filtered_clips 并更新统计、26.02 起自动记录性能指标、提供各 stage 的资源与瓶颈监控面板、Pipeline.describe() 供开发期审查——这些让工程师可以事后审计漏斗而非逐样本人工看。[不确定]

### [OmniHuman 数据集 + OHBench](../models/OmniHuman.md) ⚠️

【数据侧：论文声称完全无人工】流水线被明确定位为「fully automated pipeline for high-quality data collection and multi-modal annotation」（全自动的高质量数据采集与多模态标注流水线），全自动性正是论文主张的贡献之一。四级过滤、跟踪与姿态估计、音视觉归属、层次化标注、一致性校验，全部由模型执行，论文未提及任何人工标注、人工抽检或人工复核环节。
【自动化的质量兜底靠交叉校验而非人工】用跟踪结果校验 MLLM 的主体计数、用 ASR 转写校验 caption 的语音内容（编辑距离），这是以「模态冗余」替代「人工复核」的设计（详见 model_as_data_judge）。原理上比无任何校验的全自动流水线可靠，但仍无法覆盖属性描述准确性、交互标签正确性、情绪标注一致性等无法被交叉校验的维度。
【阈值标定的人工介入未说明】各级过滤的阈值（CLIP 美学、DOVER、光流、SyncNet、0.55 相似度、编辑距离边界）如何确定，论文一字未提——是否经人工抽检标定（如 MOVA 明确描述的「人工检查不同 cutoff 下的留存样本」）不得而知。0.55 与 5 帧、3 帧这类数值看起来像经验值，但无标定过程记录。[不确定]
【评测侧：主观维度交给 Gemini-3 而非人类】这是本工作在人工介入上最激进也最值得商榷的选择。OHBench 的核心主张是「引入与人类感知高度一致的指标」（metrics that are highly consistent with human perception），但实现方式是用 Gemini-3 / Gemini-3-pro 做 1–10 分制打分，而非组织人工评测。
【关键缺口：人类一致性的证据不足】论文正文中，对「与人类感知高度一致」这一核心主张只给出了定性表述（称在补充材料中有「证据表明我们的指标更符合人类判断」），主文未给出任何用户研究细节——无评分者人数、无评分者背景、无 Spearman/Kendall 相关系数、无与 VBench 等既有指标的相关性对比数值、无评分者间一致性（IRR）。对一个以「人类感知一致性」作为核心卖点的评测基准而言，这是最应当量化却恰恰缺失的部分，也是本工作方法学上最大的软肋。[不确定]
【对比参照】MOVA 构建了 732 条中英双语 Arena 人工评测集；UniTalking 做了 20 人 × 50 prompt 的盲测偏好研究——二者虽规模有限但至少有真实人类参与；OHBench 在主文中未见等价证据。

### [Open-Sora 系列](../models/Open-Sora.md)

人工介入程度很低，主要用于**阈值校准与抽样验证**，而非规模化标注或复核：
- 【Open-Sora Plan】唯一明确记录的人工环节是：为验证 LPIPS 运动过滤阈值（0.001–0.3）的可靠性，**人工检查了 2000 条视频**，确认该方法精度足够后才全量应用。这是把人工用于「验证自动化规则」的低成本范式。
- 【Open-Sora 系列】文档中未记录任何人工标注或人工质检环节；caption 全部由 VLM 生成，无人工复核；tools/caption 的说明反而明确指出「人工标注视频昂贵且耗时，因此采用强大的图像/视频描述模型来生成 caption」，是主动放弃人工标注的显式表态。
- 人工介入更多体现在**评测侧**：Open-Sora 2.0 用 100 条 prompt 做人工偏好评测（对比 HunyuanVideo、Runway Gen-3 Alpha、Step-Video-T2V、Luma Ray2），从视觉质量、prompt 遵循、运动质量三个维度打胜率。
- 两个项目本质上都是「全自动 pipeline + 极少量人工抽检校准」，这也是其能以极低成本处理数千万级视频的前提。

### [Ovi](../models/Ovi.md) ⚠️

[不确定：数据侧几乎无描述]。可确认的人工介入分布在两端：
(1) 数据侧：致谢中提到 Yi Cui、Manav Shah、Diego De La Torre「for their contributions to data preparation」，说明有专人负责数据准备工作，但其职责是人工标注、人工质检还是工程管线搭建未说明。论文正文的数据 pipeline 四步全部由模型/规则自动化完成，未出现人工复核环节，可判断整体为「全自动 pipeline，无逐条人工审核」。caption prompt 的「extensive experiments」是人工在模板设计层面的介入。
(2) 评测侧：人工介入充分且是主要评测手段——组织 50 名参与者做盲测成对偏好研究（blind pairwise preference study），在音频质量、视频质量、音视频同步三个维度对比 Ovi 与 JavisDiT、UniVerse-1，报告 Pairwise Win Rate（PWR）。
未见 RLHF 偏好标注、人工数据清洗抽检率等描述。

### [Script-a-Video](../models/Script-a-Video.md) ⚠️

人工介入程度极低，且分布不均：
【标注环节：零人工】500K 片段的 MTSS 标注全部由 Gemini-2.5-Pro 自动生成，论文未提及任何人工标注、人工抽检、人工复核或标注规范培训环节，也未报告标注一致性指标。对于一个依赖交叉引用完整性的结构化 schema 而言，缺少人工或程序化的校验环节是明显缺口。[不确定]
【评测环节：有相当规模的人工投入】生成侧组织了 20 名专业评分员（20 professional raters）对 225 条生成视频（125 单镜头 + 100 多镜头）做人工评分，覆盖五个维度：Text Alignment（文本对齐）、Visual Quality（视觉质量）、Multi-Shot-Consistency（多镜头一致性）、Identity Consistency（身份一致性）、Audio-Video Synchronization（音视频同步），均为 1-3 分制。
【人工评测的方法论价值】论文明确指出人工评测揭示了自动指标的欺骗性，这是本工作一个有价值的观察：
1) Intra-Shot SC 指标上基线 0.87 高于完整方案 0.59，但这是基线生成近乎静止内容的假象；
2) A-V Sync（SyncNet）指标上基线 6.86「领先」于完整方案 9.72，但基线的人评 A-V 分仅 1.18——论文解释为「信息稀疏性伪影」：让平坦的环境噪声与画面同步是平凡的（trivially easy），而生成复杂对白后 SyncNet 分数会先升高（LTX-2-AV-MTSS 达 13.86，偏离越大越差），完整方案再把它拉回 9.72 同时人评达 2.26。这一分析对任何依赖 SyncNet 做数据过滤的团队都有直接警示价值：SyncNet 分数对无语音/平坦音频样本存在系统性偏置。
3) Reference ID Similarity 在多镜头场景降至约 0.22（因剧烈视角与光照变化），但人评 Cons. 稳定在 2.40 以上——自动 ID 相似度在跨镜头场景下同样不可靠。
【标注 prompt】未公开，也未提及是否有人工参与 prompt 迭代。

### [Seedance 2.0 与 Seedance 1.5 pro](../models/Seedance_20_Seedance_15_pro.md) ⚠️

人工介入贯穿多个环节且被明确提及：(1) Seedance 1.0 为训练 caption 模型采集多样类目数据并进行「高质量人工标注（high-quality manual annotations）」；(2) SFT 阶段使用「人工核验过 caption」的高质量视频-文本对（manually verified captions），并按数百个类别定向采集，构成人工策展数据集（Human-Curated Dataset）；(3) RLHF 阶段采用多维度人工标注协议——在特定维度下选出最好与最差视频，同时确保最好者在其他维度上不劣于最差者；(4) 数据处理基础设施的统一平台层显式支持「自动化的 human-in-the-loop 工作流（automating human-in-the-loop workflows）」、任务管理、数据可视化与流水线监控；(5) Seedance 2.0 在评测端引入广告与游戏制作领域的专家评审做主观打分与盲审，并做真实性辨别研究（评审区分生成视频与真实片段），结果反哺美学调优流程。整体模式为「模型初筛 + 人工复核/精标」。[不确定：人工标注规模与人力投入]

### [SkyReels 系列](../models/SkyReels.md)

SkyReels 系列的人工介入设计较为清晰，采取「人工建判据 + 低比例抽检」的杠杆式策略：
【SkyReels-V2】
(1) 分层抽检率（本条目最具参考价值的定量披露）：预训练阶段人工抽检率 0.01%（每1万个样本抽检1个），后训练阶段提升到 0.1%（每1000个抽检1个）——按数据阶段差异化配置质检强度；
(2) 人工在环验证覆盖 pipeline 的每个阶段（Human-In-The-Loop Validation at every stage）；
(3) 打标模型的种子标注：运动识别 captioner 依赖9.3万条高置信度人工标注样本；SkyCaptioner-V1 的能力评估依赖人工评测（平均准确率76.3%）；
(4) 强化学习偏好数据：人工标注形成「3万条样本对（30k sample pairs）」用于 Bradley-Terry 奖励建模，并与自动生成的失真样本混合；
(5) 概念均衡的类目体系由人工定义。
【SkyReels-V4】
(1) SFT 第二阶段使用「100万条人工精选（manually curated）高质量视频」——人工介入直接体现在最终精调集的构建上；
(2) 评测侧组织「50名具有影视制作、音频工程与内容创作背景的专业评估员」，在2000+条提示上做5分制绝对评分与 GSB 两两对比双指标评测；
(3) 未披露数据清洗环节的人工抽检率与标注工时。

### [Sora 2](../models/Sora_2.md) ⚠️

训练数据标注侧：仅知存在「human trainers and researchers provide or generate」的数据，说明有人工参与数据提供与生成，但规模、流程、是否人工质检复核均未披露。部署侧人工介入有明确说明：与外部 OpenAI Red Team Network 红队成员合作测试（覆盖性内容、裸露、极端主义、自残、违法行为、暴力血腥、政治说服，以及青少年安全与肖像使用等专项政策），红队反馈用于调整prompt过滤器、屏蔽词表与分类器阈值；内容审核采用「自动化 + 人工复核」组合以识别滥用模式，并提供应用内举报通道与申诉机制。[不确定]

### [Step-Video-T2V](../models/Step-Video-T2V.md)

人工被集中投放在漏斗最末端（预训练全自动、后训练重人工），分三处：
1. SFT 数据人工精选：30M 高质量视频集的构建为「自动化 + 人工」两段式——先用各项评估分数与启发式规则自动过滤，再按视频类目（簇）剔除距簇中心超阈值的离群样本，最后由人工标注员逐条评审，评审维度为：画面清晰度（clarity）、美学质量（aesthetics）、运动是否恰当（appropriate motion）、场景转换是否平滑（smooth scene transitions）；人工同时会对 caption 做优化/改写（caption refinement），即人不仅筛数据也修标注。
2. DPO 偏好标注：人工标注员按设计指引合成 prompt；对同一 prompt 不同种子生成的多个视频进行偏好打分排序；全流程由质控人员（quality control personnel）监督以保证标注的准确性与一致性。
3. 评测人工评审：在 Step-Video-T2V-Eval 上以人工评测为主要评估手段，对比多个开源与商用引擎。
未披露标注团队规模、人时成本或标注一致性指标（如 Kappa 值）。报告在展望部分明确指出 DPO 的收益会在「模型能轻易区分正负样本时饱和」，并提出未来改用 reward model 对新生成样本动态打分来替代纯人工标注——即团队已意识到人工偏好标注的可扩展性瓶颈。

### [UniTalking](../models/UniTalking.md) ⚠️

人工介入在训练数据侧为零，在评测侧则是唯一且核心的评估手段——这是一个「数据全自动、评测全人工」的极端分配：
【训练数据侧：完全无人工】三级过滤（视频/音频/音视频）全部为自动化模型判定，标注全部由 Qwen3-VL/Whisper-V3/Qwen3-Omni 自动生成，参考音频由 IndexTTS2 自动合成。论文未提及任何人工标注、人工抽检、人工复核，也未说明各项过滤阈值是否经人工验证标定（对比 MOVA 明确写出「人工抽检不同 cutoff 下留存的视频来设定阈值」）。考虑到全文未披露任何阈值数值，其标定过程更是完全的黑箱。
【评测侧：人工是主要评估方式】T2AV 联合生成任务的核心评估采用盲测两两偏好研究（blind pairwise preference study）：
- 参与人数：每组模型对比 20 名参与者；
- 测试集：50 条说话人生成测试 prompt；
- 评估维度三项：视频质量、音频质量、音视频同步；
- 计分方式：以 OVI 为基线（归一化为 100），胜负率计算公式为 rate = (win + tie) / (lose + tie)；
- 结果：UniTalking 相对 OVI 在音频质量得 116%、音视听同步得 107%，视频质量持平（论文解释为两者共用 Wan2.2 基座，视频侧本就同源）。
【人工评测的方法学局限】20 人 × 50 条 prompt 的规模偏小，未报告置信区间或统计显著性检验，未说明参与者背景与筛选标准，未说明是否做了样本随机化与顺序平衡。相较 MOVA 构建 732 条中英双语 Arena 评测集，规模与严谨度均有差距。[不确定]
【客观指标的辅助地位】Sync-C/Sync-D、Speaker Similarity、WER 三项客观指标用于补充，但论文对 Sync-C 的可信度提出了质疑（认为该指标偏好夸张的口型运动，故 OVI 的 6.56 高分未必代表真实同步质量更好），这本身也说明团队把人工主观评测视为更可靠的判据。

### [UniVerse-1](../models/UniVerse-1.md)

训练数据侧的人工介入几乎为零：整条清洗漏斗（音轨检测 → 分辨率/码率/DOVER → PySceneDetect → 音频活动检测 → Whisper 语音判定 → RetinaFace + SyncNet）全部为自动化阈值判定，标注环节则完全由 QWen2.5-Omni 与 Whisper 在线自动生成。论文未提及任何人工标注、人工抽检、人工复核，也未说明各项阈值是否经人工验证标定——这一点与 MOVA 明确写出「人工抽检不同 cutoff 下留存的视频来设定阈值」形成对比。
【唯一的人工介入在评测侧】Verse-Bench 的 Set2-V 子集（295 条，来自 YouTube 与 Bilibili）的构建流程为：LLM 生成 caption + Whisper 做 ASR，随后经人工核验（human verification）。即人力被集中投入到评测基准的质量保证上，而非训练数据。
【设计含义】这是一个「训练数据全自动、评测数据半人工」的资源分配取向，符合其算力/人力受限的整体定位（Limitations 明确承认受计算资源约束）。

### [Unison](../models/Unison.md) ⚠️

训练数据侧的人工介入为零：整条 pipeline（数据集聚合 → 自动化精炼 → 人脸检测 + 框内 SyncNet 唇过滤 → Mel-RoFormer 音源解耦）全部自动化，论文明确称其为「automated processing pipeline」。未提及任何人工标注、人工抽检、人工复核，未说明 SyncNet 等环节的阈值是否经人工校准。
【间接继承的人工成果】需要指出，Unison 的语料虽然自身无人工介入，但五个上游开源数据集在各自构建时均包含大量人工工作——CelebV-Text 的文本描述、VGGSound 的声音事件类别标注、VFHQ/HDTF 的质量筛选，都是人工或半人工产物。聚合型数据构建的一个隐性收益正是「白嫖」了上游的人力投入，这也是 Unison 能以纯自动化 pipeline 获得可用语料的前提。
【人工介入全部集中在评测侧】
1) 测试集「curated」（精选）—— 1,000 条 held-out 样本被形容为「curated test set」，暗示存在人工挑选环节，但标注本身由 Gemini 生成，人工的具体职责未说明；[不确定]
2) 用户研究 —— 25 名「来自不同背景」的参与者对 40 组样本做排序评价，共产生 1,000 次平均排序投票（40 × 25）。评价维度为三项：唇-语音同步（Lip-sync）、语音-音效谐和（Speech-Sound Harmony）、运动-音频对齐（Motion-Audio Align.，同时考虑语音与环境音）。参与者需对打乱顺序的多方法视频进行排序（越低越好）。结果：Unison 在语音-音效谐和（1.55）与运动-音频对齐（1.92）两项夺冠，唇同步（1.86）次于 LTX-2（1.74）——这一排名分布恰好印证了论文的贡献定位：其创新在「声学层次平衡」与「运动-音频对齐」，而非唇同步本身。
【资源分配取向】与 UniVerse-1 一致，均为「训练数据全自动、评测数据半人工」；但 Unison 的人工评测设计更完整（三维度排序 + LLM 裁判交叉验证），而 UniVerse-1 完全无主观人评。

### [Veo 3 / Veo 3.1](../models/Veo_3_Veo_31.md) ⚠️

可确认的人工介入集中在评测与安全侧而非数据标注侧：(1) 模型效果评测由人类评分员（human raters）做 head-to-head 偏好对比（MovieGenBench、VBench-I2V）；(2) 红队测试由内部专家团队与外部招募参与者混合执行，贯穿模型开发全过程；(3) 保障性评测（assurance evaluations）由独立于模型开发团队的专门团队开发与执行，评测数据集严格保留（held out）；(4) 偏见评测涉及人工分析——用 140 种职业各生成 16 条视频，再按感知肤度（Monk 肤色量表）、感知年龄、感知性别归类；(5) 发布前由 Google DeepMind 责任与安全委员会（RSC）人工审批。[不确定] 训练数据本身的人工标注规模、人工质检抽检比例、「模型初筛+人工复核」是否存在，均未披露——从表述看 caption 与过滤基本为全自动模型驱动。

### [Vidu S1](../models/Vidu_S1.md) ⚠️

数据侧几乎全自动：过滤与标注均由专家模型 + omni 大模型 + 启发式规则（如语音能量占比规则）完成，论文未提及任何人工标注或人工复核环节 [数据侧人工介入程度不确定]。人工介入明确出现在评测端：在自建的 Vidu-StreamBench（500 样本）上组织人工偏好评测，与 HeyGen、LemonSlice、Kling Avatar 2.0 三家商用系统做成对 A/B 测试，评测维度含 Overall、Motion Dynamics、Identity Consistency、Audio-Video Sync、Subject Controllability。

### [主流视频预训练数据集合并调研：Panda-70M、InternVid、Koala…](../models/pretraining_datasets.md) ⚠️

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
