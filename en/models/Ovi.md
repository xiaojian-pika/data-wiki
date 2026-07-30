# Ovi (paper "Ovi: Twin Backbone Cross-Modal Fusion for Audio-Video Generation," arXiv:2510.01284; subsequent iteration Ovi 1.1; the audio tower is separately named Ovi-Aud)

> Topic: Data processing for video generation models (including joint audio-video generation): data cleaning pipeline, data distribution, annotation approach

[← Back to home](../index.md)

**Contents**: [Basic Information](#basic-information) · [Data Scale and Distribution](#data-scale-and-distribution) · [Cleaning Pipeline](#cleaning-pipeline) · [Captioning Approach](#captioning-approach) · [Audio-Video Alignment](#audio-video-alignment) · [Training Coordination](#training-coordination) · [Effectiveness Comparison](#effectiveness-comparison)

## Basic Information

### Name

Ovi (paper "Ovi: Twin Backbone Cross-Modal Fusion for Audio-Video Generation," arXiv:2510.01284; subsequent iteration Ovi 1.1; the audio tower is separately named Ovi-Aud)

### Releasing institution/company

Released jointly by Character AI (the primary body, authors Chetwin Low and Weimin Wang, with Weimin Wang as Project Lead) and Yale University (author Calder Katyal). The acknowledgments note that Yi Cui, Manav Shah, and Diego De La Torre contributed to data preparation.

### Release date (technical report/paper/open-source date)

The paper was finalized on September 29, 2025, and made public on arXiv on October 1, 2025 (arXiv:2510.01284, v1); the HuggingFace model page lists the paper's publication date as September 30, 2025. The 11B model weights and inference code were open-sourced on GitHub (character-ai/Ovi) at the same time. An Ovi 1.1 update was released on November 10, 2025 (native 960×960 training, 10-second generation, dataset scale doubled). As of July 2026, no Ovi 2.0 or training code has been released.

### Type (model/dataset/toolchain/evaluation benchmark)

A model (an open-source text/text+image-to-"audio+video" joint one-pass generation model, T2AV / I2AV), accompanied by an open-source inference toolchain (inference scripts, a Gradio app, multi-card sequence-parallel inference, fp8/qint8 quantization, ComfyUI integration). It is not a dataset, nor an evaluation benchmark (evaluation borrows the Verse-Bench proposed by others).

### Degree of openness (whether weights/code/data/pipeline are each open source)

A typical example of the "weights + inference code open source, data and data pipeline code closed source" pattern.
[Open-sourced] (1) Model weights: 11B (the HuggingFace page lists about 12B parameters, BF16) complete checkpoints, including three versions — 720x720_5s, 960x960_5s, 960x960_10s — hosted on HuggingFace at chetwinlow1/Ovi; (2) inference code: text/text+image input, a Gradio App, multi-GPU (including sequence-parallel) inference, weight-download scripts; (3) community-contributed fp8 (@rkfg) and qint8 (@gluttony-10) quantized weights, runnable on 24GB of VRAM; (4) the paper fully discloses the method and key thresholds of the data-processing pipeline (more open than most peer models on this point).
[Not open-sourced] Training scripts (the "Training scripts" item in the README's Todo List remains unchecked), the training data itself (the internal audio-video corpus and internal audio corpus), the data-processing pipeline code, the identity of the MLLM used for captioning, the quantitative retention rate at each filtering level, and RL post-training details.
[License] Apache 2.0 (relatively permissive, commercial use allowed). Dependent components: the video-branch weights come from Wan2.2-TI2V-5B, the text encoder T5 and the video VAE decoder come from Wan, and the audio VAE comes from MMAudio (all open-source models).

### Whether joint audio-video generation is supported, and the implementation approach (native joint/cascade/MoE fusion)

Supported, and it is native joint generation (one-pass joint AV generation), neither cascaded nor MoE-fused; the authors call it "twin backbone blockwise cross-modal fusion."
(1) Symmetric twin DiT: the audio tower and video tower have identical architecture (Model Dim 3072, FFN Dim 14336, 24 heads, head dim 128, 30 blocks, each block containing 30 layers of Self-Attn / Text Cross-Attn / AV Cross-Attn). The video branch is initialized from Wan2.2 5B, and the audio branch is a same-architecture 5B trained from scratch, totaling about 11B.
(2) Block-by-block bidirectional cross-modal attention: within each transformer block, the audio stream attends to the video stream and the video stream attends back to the audio stream, propagating synchronization cues throughout the entire network. Because the two towers share the same hidden dimension, no projection layer is needed (in contrast to UniVerse-1, which requires inserted blocks, projections, and an auxiliary semantic-alignment loss).
(3) Temporal alignment relies on scaled RoPE: the video latent has 31 frames and the audio latent has 157 tokens (16kHz × 5s / 512); the audio branch's RoPE frequency is scaled by 31/157 ≈ 0.197, so the cross-modal RoPE affinity matrix's diagonal aligns.
(4) A single frozen T5 encoder encodes the "merged prompt" (visual description + <S>dialogue<E> + audio description); the same text embedding does cross-attention separately with the audio tower and video tower, providing unified cross-modal semantic control.
(5) The training objective is flow matching, with both modalities sharing the same timestep t but each having independent noise; the total loss is L=0.85·L_video+0.15·L_audio; there is no explicit synchronization loss, no face mask, no post-hoc alignment, and no auxiliary synchronization module. At inference, both branches share the same ODE solver (UniPC).
(6) Capability: the initial version of Ovi generates 5 seconds at 720×720@24fps; Ovi 1.1 generates 10 seconds at 960×960@24fps, supporting multiple aspect ratios such as 9:16, 16:9, and 1:1, while simultaneously outputting dialogue, sound effects, and background music.

### List of research information sources (URLs of papers/technical reports/official documentation/news, with each source's nature annotated: official first-hand/same-team corroboration/third-party report)

1) Official first-hand | Full paper text https://arxiv.org/abs/2510.01284 and https://arxiv.org/pdf/2510.01284 ("Ovi: Twin Backbone Cross-Modal Fusion for Audio-Video Generation," Section 3 "Data Processing Pipeline" is the sole authoritative disclosure on the data side)
2) Official first-hand | GitHub repository and README https://github.com/character-ai/Ovi (scope of open-sourcing, Todo List, Ovi 1.1 update notes, prompt format)
3) Official first-hand | HuggingFace model card https://huggingface.co/chetwinlow1/Ovi (parameter count, base models, license, resolution/duration versions)
4) Official first-hand | Project homepage https://aaxwaz.github.io/Ovi/ (demo and code entry point)
5) Same-team corroboration | HuggingFace Papers page https://huggingface.co/papers/2510.01284
6) Third-party reporting | CSDN Chinese-language interpretation https://blog.csdn.net/SuaniCommunity/article/details/154737163 and Tencent News https://view.inews.qq.com/a/20251113A02WA900 (restate the paper's four-step data pipeline and the 720×720 threshold, no new first-hand information)
7) Third-party corroboration | Tencent Cloud Developer Community technical write-up https://cloud.tencent.cn/developer/article/2584843
8) Third-party platform | WaveSpeed AI hosting page https://wavespeed.ai/models/character-ai/ovi/text-to-video (commercial availability)

## Data Scale and Distribution

### Training data magnitude (video count/hour count/token count, pretraining and SFT listed separately) ⚠️

The paper gives only order-of-magnitude descriptions, not precise figures.
[Audio-video paired corpus] "millions of videos" (several million videos), entirely in-house audio-video corpus; estimating at 121 frames @24fps ≈ 5.04 seconds per clip, if there were 3 million clips this would be on the order of about 4,200 hours, but the paper does not confirm the clip count, so this conversion is an inference [Uncertain].
[Pure audio corpus] Pretraining stage: "hundreds of thousands of hours of raw audio" (hundreds of thousands of hours), predominantly human speech, with waveform length up to 12 seconds; fine-tuning stage: fixed-length 5.04-second waveforms, mixed with public sound-effect data from VGGSound / AudioSet / WavCaps plus audio tracks extracted from the internal audio-video corpus.
[Inference from training-step counts] Audio pretraining, 50k steps × batch 2880 ≈ 144 million sample instances; audio-video fusion training, 40k steps × batch 768 ≈ 30.72 million sample instances (for epoch-conversion reference, not a data-item count).
[Ovi 1.1] The README explicitly states "Dataset includes 100% more videos," i.e. the audio-video dataset scale is doubled compared to the initial version, and training switches to native 960×960 resolution data; absolute figures are not published [Uncertain].
The strict split between pretraining and SFT figures is not published [Uncertain].

### Data source composition (proprietary/public datasets/web crawling/licensed acquisition/synthetic data) ⚠️

Three clearly delineated categories:
(1) Internal proprietary audio-video paired corpus — occupying the core position; the paper describes it as "composed of human and nonhuman data from diverse contexts" (covering both scenes with and without people); its sourcing channel is not disclosed [Uncertain].
(2) Internal pure-audio corpus ("internal collections") — used to pretrain the audio tower from scratch, predominantly human speech, emphasizing linguistic diversity, prosody, and timbral variation. The README calls it "high quality in-house audio datasets." Character AI, as a conversational-companion product company, is presumed to have internal voice resources related to its product's speech data, but the paper does not state this [Uncertain].
(3) Public datasets — used only to supplement sound-effect capability during audio fine-tuning: VGGSound (Chen et al., 2020), AudioSet (Gemmeke et al., 2017), WavCaps (Mei et al., 2024).
(4) No description of purchased/licensed data, and no description of a synthetic-data construction step.
In addition, at the model level Ovi reuses a large amount of open-source assets (Wan2.2 5B video weights, Wan's T5 and video VAE, MMAudio's 16kHz audio 1D VAE, the BigVGAN vocoder), which can be viewed as indirectly inheriting the distribution of Wan2.2's pretraining corpus.

### Data compliance and provenance (proportion of licensed data, rights-cleared datasets, C2PA, etc.) ⚠️

[Uncertain]. Neither the paper nor the repository discusses the proportion of licensed data, rights-cleared datasets, C2PA/watermark provenance, copyright compliance review, or anything similar, and there is no data-source statement at the Model Card level. The only confirmable facts are: the three public audio datasets used during fine-tuning (VGGSound, AudioSet, WavCaps) are all academic-research-license datasets, with AudioSet/VGGSound based on YouTube videos and thus carrying their own sourcing-compliance disputes; model weights are released under Apache 2.0, but this license covers the weights, not the training data. The licensing status of the internal corpus is entirely undisclosed.

### Clip duration distribution and splitting strategy ⚠️

The strategy is "uniform fixed length, no multi-duration bucketing" — one of the most distinctive features of Ovi's data design.
[Audio-video data] After scene-detection splitting, clips are uniformly taken at a fixed length of 121 frames @24fps (= 5.04 seconds); both the initial-version training and inference use this length; Ovi 1.1 extends this to 10 seconds (corresponding to roughly 241 frames @24fps) [the frame count corresponding to 10 seconds is an inference, uncertain].
[Pure audio data] A dual-duration design: pretraining uses variable-length waveforms up to 12 seconds long (the paper explicitly states it will "use variable-length audio to maximize coverage of diverse acoustics," relying on the long waveforms to learn long-range consistency of a speaker's pitch and emotion); fine-tuning pads to an exact 5.04-second fixed length, to align with the duration of the 121-frame video, avoiding a temporal re-adaptation once entering the audio-video fusion stage.
[Design rationale] The paper explains that applying scaled RoPE uniformly across all attention layers exists precisely "to avoid re-adaptation when moving into the audio-video fine-tuning stage, and to avoid maintaining multiple sets of audio RoPE scales."
[Self-acknowledged limitation] The paper's Limitations section explicitly acknowledges that the model is confined to short 5-second clips, and that minute-scale narratives, inter-shot transitions, and global story consistency are all out of scope, proposing future work using a chunk-wise causal audio model plus a causal video backbone conditioned on the final frame of the previous segment to stitch together multiple 5-second chunks.

### Resolution/aspect-ratio distribution and bucketing strategy ⚠️

[Filtering threshold] The splitting stage requires clip resolution to be strictly greater than 720×720 pixels (the paper's original text: "clips are greater than 720x720 pixel resolution"); anything below this threshold is directly discarded.
[Normalization strategy] No resolution bucketing is done; instead, uniform normalization is applied — before packing, existing black bars/margins in the video are first removed, and then frames are scaled to a fixed total pixel count of 518,400 (= 720×720) while maintaining aspect ratio. That is, the constraint is on "area" rather than "edge length," so samples of different aspect ratios can land on the same token count, naturally supporting multiple aspect ratios such as 9:16, 16:9, and 1:1 without needing bucket scheduling.
[Ovi 1.1] Switches to training on native 960×960 resolution data (total pixel count raised to 921,600); inference supports 960×960 and various aspect ratios of equal area.
[Actual mixture figures for each aspect ratio are not disclosed] [Uncertain].
[Packing] The final video is frame-sampled at 24fps and converted into a byte array, and audio is converted to raw wave bytes, for high-throughput reading on the training side.

### Category/domain distribution and mixture strategy (proportion control and concept balancing for people, actions, scenes, styles, etc.) ⚠️

The paper discloses one clear and uncommon "person-composition mixture control" mechanism, but does not give specific proportion figures.
[Explicit mixture control] An internal face-detection model is used to classify clips into three categories and "ensure an adequate mix": single-person videos, multi-person videos, and person-free videos. The stated motivation is "to let the model learn to generate videos in diverse contexts, without overfitting to a specific subtask" — i.e., avoiding degenerating into a pure talking-head model, as many A2V models do. The specific percentages of these three categories are not disclosed [Uncertain].
[Qualitative composition of the corpus] The internal audio-video corpus is described as "human and nonhuman data from diverse contexts."
[Indirect evidence of domain coverage] Section 5.1's cross-modal attention visualizations are organized by content category, covering instrument playing, bird calls, rockets, animals, speech (multiple examples), helicopters, sports, etc., allowing the training distribution to be inferred as covering at least: human dialogue, instruments/music, animals, vehicles/machinery, sports, and natural ambient sounds.
[Sound-effect-side domain] Through the introduction of VGGSound / AudioSet / WavCaps, sound-effect domain coverage is indirectly determined by these three public datasets' taxonomy (e.g. AudioSet's 632-class ontology).
[Not addressed] The paper does not mention proportions for style (realistic/animated/CG), scene-category proportions, action-category proportions, or concept-balancing strategies [Uncertain].

### Audio category distribution and mixture (proportion and control strategy for speech/sound effects/music/ambient sound/silence) — a dimension unique to AV models ⚠️

Audio-category mixture is the core tension in Ovi's data design; what the paper gives is a "phase-dependent bias" rather than a static proportion.
[Pretraining stage] "Predominantly human speech," from the internal corpus, emphasizing linguistic diversity, prosody, and timbral variation. This stage contains almost no sound effects, aimed at first solidifying TTS/speaker-modeling ability.
[Audio fine-tuning stage] Explicitly shifts toward sound effects: "we emphasize modeling sound effects," supplementing SFX coverage by introducing VGGSound / AudioSet / WavCaps; simultaneously, to "maintain TTS abilities and better align with the downstream goal," audio tracks extracted from the internal audio-video corpus are additionally mixed in. This forms a two-stage mixture schedule of "speech-heavy → sound-effect supplement + speech kept alive." The mixture ratio between the three public sound-effect sets and the internal speech tracks is not disclosed [Uncertain].
[Audio-video fusion stage] Audio on this side comes entirely from the native audio track of the paired video, so the natural proportion of speech/sound effects/music/ambient sound is determined by the internal corpus, with no described artificial rebalancing [Uncertain].
[Category routing at the captioning level] Captioning is handled bifurcated by "contains speech / no speech": for clips containing speech, the audio description emphasizes the speaker's acoustic attributes; for non-speech clips, the audio description instead describes sound effects, background audio, and musical elements — effectively a three-category (speech/sound-effects/music) annotation scheme.
[Silence handling] Near-silent clips are removed via a mean-volume ≥ −60 dB threshold (see audio_quality_filtering).
[Justification for a unified audio model] In the results section the paper emphasizes: real-world video often contains both complex sound effects and coherent speech simultaneously; specialized models (pure T2A or pure TTS) cannot support this, hence the necessity of training a unified T2A+TTS audio tower — this is the core motivation behind its mixed-audio-category training strategy.
[Music] BGM-generation capability is listed as a feature, but whether music data was separately introduced is not stated [Uncertain].

### Narrative structure distribution (single-shot vs. multi-shot, average clip duration, shot-count distribution, presence of native audio track) ⚠️

[Predominantly single-shot] Splitting via scene detection means the granularity of splitting itself guarantees each 121-frame clip lies within a single shot, so training samples are almost entirely single-shot clips with no shot changes. This corroborates the paper's Limitations statement that "inter-shot transitions and global story consistency are out of scope."
[Average clip duration] Fixed at 5.04 seconds (10 seconds for Ovi 1.1), with no length distribution — all samples are equal length.
[Shot-count distribution] Always exactly 1 shot per sample.
[Native audio track] 100% contain a native audio track — the entire value of the audio-video paired corpus lies in its native, synchronized audio track, which has been filtered by SyncNet and a volume threshold; clips with no audio track or near-silence have been removed. This is the opposite of some models' approach of "stripping the audio track first and dubbing it in post."
[Source of narrative capability] The paper claims the model can do "cinematic storytelling," but this narrative quality comes from captions that chronologically interweave visual events and dialogue annotations (chronology is explicitly required), not from multi-shot data.

### Language/accent distribution (data foundation for multilingual lip-sync capability) ⚠️

No quantitative distribution is given. The confirmable qualitative facts are:
(1) The audio pretraining data "emphasize linguistic diversity, prosody, and timbral variation," meaning the corpus has deliberate diversity coverage at the language/accent level, but which languages are covered and in what proportion is not disclosed [Uncertain].
(2) At the captioning stage, clips containing speech are explicitly annotated with an accent attribute, listed alongside age, gender, pitch, prosody, emotion, and speaking rate — meaning accent is a controllable condition rather than a hidden variable.
(3) At the evaluation level, only English WER is reported: WER=0.035 on Seed-TTS test-en, with no non-English metric reported, indicating that its verifiable speech capability is primarily English; community practice and demos are likewise mostly in English dialogue [Uncertain].
(4) Lip-sync capability is entirely data-driven (no face bounding box, no mask), so multilingual lip-sync capability is directly limited by the language coverage of the internal corpus.

## Cleaning Pipeline

### Overall structure of the cleaning funnel (number of levels, ordering)

Ovi adopts a "two corpora, each with its own funnel" dual-track design: the audio-video paired corpus goes through a four-step serial funnel, while the pure-audio corpus goes through a simplified two-step process.

[A. Audio-video paired corpus — a four-step pipeline (explicitly numbered in Section 3.2 of the paper)]
Step 1, splitting and filtering: scene detection splits out 121-frame @24fps clips → resolution must exceed 720×720 → RAFT optical flow removes static videos and produces a motion score → the LAION aesthetic predictor removes low-quality data → an internal face-detection model performs single-person/multi-person/person-free composition-mixture control.
Step 2, sync detection: SyncNet produces confidence and offset scalars → only clips with |offset| ≤ 3 and confidence > 1.5 are kept → layered with an audio threshold of mean volume ≥ −60 dB.
Step 3, captioning: an MLLM takes 7 uniformly sampled frames plus the full audio track as input → outputs a merged caption of "narration of visual events + interleaved <S>dialogue<E> + a trailing <AUDCAP>audio description<ENDAUDCAP>."
Step 4, packing: remove black bars → scale to a fixed 518,400 pixels (equal-area to 720×720) while maintaining aspect ratio → frame-sample at 24fps and convert to a byte array → convert audio to raw wave bytes.

[B. Pure audio corpus — a simplified process]
Split by duration into two tiers (pretraining ≤12s, fine-tuning exactly 5.04s) → use the same MLLM as on the audio-video side to produce an audio transcript (left blank if no speech) and an audio description → proceed to training. No sync/aesthetic/motion-style filtering is applied on this side.

[Design orientation] Overall, the funnel places "synchronization" as the highest priority (the paper states explicitly, "even a small quantity of out-of-sync data would harm lip-sync abilities, hence choosing strict criteria to minimize the risk of misalignment"), followed by baseline quality gates on resolution and motion/aesthetics; deduplication, safety, and watermark/OCR dimensions receive no attention.

### Quantitative funnel retention rate (input/output volume at each level and final retention rate, e.g. Apollo's 27%) ⚠️

[Uncertain]. The paper gives no input/output volume, pass rate, or overall retention rate for any filtering level, and there is no Apollo-style 27%-type quantitative funnel table. Only a qualitative judgment can be made that filtering strictness is high: the authors themselves describe the SyncNet conditions (|offset|≤3 and confidence>1.5) as "strict criteria," layered on top of the >720×720 resolution threshold, RAFT static removal, and aesthetic removal; the actual retention rate should be significantly lower than in typical loose pipelines, but there is no data to support this. Ovi 1.1 discloses only that the final dataset size doubled, without disclosing the original candidate-pool size.

### Shot-splitting method (PySceneDetect/in-house model/shot-aware splitting) ⚠️

Uses "scene detection" to split long videos into intra-shot clips, then extracts a fixed-length 121-frame @24fps clip from each. The paper only writes the generic term "scene detection," without naming the specific tool (it is not stated whether this is PySceneDetect, TransNetV2, or an in-house model) [Uncertain]. Splitting and filtering are coupled in the same step: a split clip must simultaneously satisfy resolution, motion, aesthetics, and face-composition conditions to be retained, so this is not a two-stage "split then filter" process but rather "splitting is filtering." The splitting granularity determines that training samples are entirely single-shot, with no shot-transition samples.

### Quality filtering (aesthetic scoring, sharpness, OCR text filtering, black-bar/watermark/logo detection) ⚠️

Visual-quality filtering consists of three explicit modules, all within step 1 of the pipeline:
(1) Resolution threshold: clip resolution must exceed 720×720 pixels, forcibly removing low-definition footage. Ovi 1.1 raises this to native 960×960 data.
(2) Aesthetic scoring: uses the LAION aesthetic predictor (Schuhmann, 2022, i.e. the LAION-Aesthetics aesthetic scoring model) to remove low-quality data. The specific threshold is not disclosed [Uncertain].
(3) Motion quality: RAFT optical flow removes static clips (see motion_filtering for details).
(4) Structural cleanup (in the packing stage rather than the filtering stage): existing margins/letterboxing in the video are explicitly removed, then scaled to equal area. This treats "black bars" as a fixable flaw rather than a reason for discarding.
(5) Content-composition control: the internal face-detection model ensures a reasonable mix of the three categories single-person/multi-person/person-free (a distribution-control measure rather than a quality filter, but performed in the same step).
[Common techniques not mentioned] OCR/subtitle text filtering, watermark detection, logo detection, compression-artifact/blur detection, brightness/contrast filtering, JPEG block-artifact detection, etc., are all not described in the paper [Uncertain]. This may be a potential risk point for the model on subtitled footage.
[Audio-side quality filtering] See audio_quality_filtering (mean volume ≥ −60 dB).

### Motion filtering (optical-flow/motion-score thresholds, removal of static and jittery content) ⚠️

Uses the optical-flow model RAFT (Teed & Deng, 2020) for two purposes: (1) filtering out static videos, i.e. clips lacking effective motion are directly removed; (2) computing and storing a motion score for retained clips as an annotation/filtering signal.
[Threshold] The optical-flow-magnitude threshold for static determination, the range of the motion score, and any high/low-end truncation strategy are all not disclosed [Uncertain].
[Jitter removal] The paper mentions only removing the "static" end, and does not mention whether the excessive-motion end (severe camera shake/rapid panning) is also removed [Uncertain].
[Downstream use of the motion score] The paper does not state whether it is used as a training condition, a sampling weight, or a basis for curriculum scheduling [Uncertain].

### Deduplication method (exact deduplication and embedding-based semantic deduplication recorded separately) ⚠️

[Uncertain]. The four-step data pipeline in the paper contains no deduplication step whatsoever — neither exact deduplication (hashing/pHash/frame fingerprinting) nor embedding-based semantic deduplication (CLIP/video-embedding clustering to remove redundancy) is described. The pure-audio side likewise has no deduplication description. Since the data comes from internal corpora rather than large-scale web crawling, redundancy may be naturally lower, but this is only an inference with no textual basis.

### VLM/LLM as quality inspector (multimodal-large-model quality scoring and mismatch removal; the 2026 trend of shifting from shallow scorers to large-model semantic judgment) ⚠️

Ovi is at the stage of "MLLM used for captioning rather than for quality-judgment scoring," which only partially overlaps with the mainstream "large-model semantic judge" route of 2026.
[Model-based judgment already in use]
(1) MLLM as annotator: taking 7 uniformly sampled frames plus the full audio track as input, producing a long caption that interweaves visual events and dialogue along with a structured audio description. The paper emphasizes that it "conducted extensive experiments to ensure the captioning included all relevant visual and audio events while respecting chronology" — i.e., extensive prompt experimentation to ensure event completeness and correct chronological order, an indirect form of quality control over MLLM output, but without describing the use of a second model to verify whether the caption is semantically mismatched with the content.
(2) Discriminative models as filters: SyncNet (the synchronization judge), RAFT (the motion judge), the LAION aesthetic predictor (the aesthetic judge), and the internal face-detection model (the composition judge) — all shallow/task-specific scorers, consistent with typical mid-2025 practice.
[Techniques not used] No use of a VLM/LLM to give an overall quality score to a clip, no use of a large model for secondary removal of "caption-video semantic mismatch," no use of a large model for content-compliance judgment, and no model-as-judge scoring threshold. This dimension overall is a weak point of Ovi relative to strong contemporaneous pipelines (e.g. HunyuanVideo, which uses an MLLM in place of T5 alongside a robust data filter) [Uncertain: whether an undisclosed internal MLLM quality-inspection step exists].
[MLLM identity] Throughout, the paper writes only "an MLLM," without specifying whether it is GPT-4o/Gemini/an in-house model, or its scale [Uncertain].

### Safety and compliance filtering (NSFW, copyright, face/privacy) ⚠️

[Uncertain]. Neither the paper nor the repository describes any NSFW filtering, violent/gory content filtering, copyrighted-content identification, face-privacy protection, celebrity-likeness removal, or child-content handling, and there is no safety section or usage-restriction statement at the Model Card level (only an Apache 2.0 license). Given that Character AI, as a consumer-facing conversational product company, likely has an internal content-moderation system, its internal audio-video corpus may already have been reviewed on the product side, but the paper makes no such statement. This is a clear gap in this work's data disclosure regarding safety.

## Captioning Approach

### Caption models used (in-house VLM/open-source model, model scale) ⚠️

Uses "an MLLM" (multimodal large language model) to uniformly handle captioning for both the audio-video corpus and the pure-audio corpus, with the audio-video side and pure-audio side explicitly using the same model ("the same MLLM as used in our audio-video data").
[Model identity and scale] Not named in the paper; it is not stated whether it is a commercial API (e.g. GPT-4o, Gemini) or an internal in-house model, nor is a parameter scale given [Uncertain].
[Input form (a key design)] Not pure visual input, but "7 uniformly sampled keyframes + the full audio track" fed in simultaneously — i.e. the captioning model itself must have audio-understanding capability (an audio-capable MLLM), a core requirement that distinguishes AV joint-generation-model captioning from pure video-model captioning. The sparse sampling of 7 frames means the visual-side description skews toward the event/semantic level rather than dense frame-by-frame description.
[Prompt-engineering investment] The paper explicitly states it "conducted extensive experiments" to ensure captions cover all relevant visual and auditory events while respecting chronology, indicating the caption template went through many rounds of iteration.
[No independent ASR] Dialogue transcription is produced directly by this MLLM from the audio track, with no mention of an independent ASR model such as Whisper being used [Uncertain].
[Inference-side counterpart] User prompts must follow the same format as the training captions; the repository provides GPT-generated example prompt CSVs (gpt_examples_t2v.csv / gpt_examples_i2v.csv), i.e. it is recommended to use an LLM to expand user prompts according to the template.

### Caption density and degree of structuring (short/long/dense description, structured fields such as camera motion, style tags) ⚠️

A design of "a single long caption with embedded structured tags," rather than multi-field JSON.
[Overall form] One verbose (long/dense) natural-language caption, with visual events narrated in chronological order, dialogue embedded inline as tags, and the audio description placed uniformly at the end.
[Tag system]
- Speech: <S>dialogue content<E> (start-of-speech / end-of-speech tags), which may appear multiple times in a caption, interleaved chronologically with visual events, thereby implicitly encoding the temporal information of "who said what, and when."
- Audio description: <AUDCAP>audio description<ENDAUDCAP>, placed at the end of the caption. Starting with Ovi 1.1, this format is simplified to a plain-text prefix, "Audio: ...", placed at the end of the prompt.
[Density] The visual side is a verbose long description; because input is limited to 7 frames, the density skews toward the event level rather than dense frame-by-frame description.
[Structured fields] No explicit structured fields for camera movement, shot scale, style tags, or lighting are observed [Uncertain]. The known structured dimensions are concentrated on the audio side (speaker attributes) and the implicit chronological ordering.
[Single-prompt conditioning] Captions are not split into separate video-prompt and audio-prompt streams; instead they are merged into a single string fed to a single frozen T5 encoder, with the same embedding doing cross-attention separately with the video tower and audio tower. The paper's intuitive explanation is: visual-context detail improves the specificity and diversity of the audio, while acoustic-context detail guides facial and body movement. The ablation study (Section 5.5) confirms this merged design outperforms a separated-encoding scheme.

### Joint audio-video caption structure (whether both visual and auditory tracks are covered, whether split into independent fields, e.g. LTX-2's full soundscape description, Script-a-Video's factorized streams, Foley-Omni's three fields)

Ovi's joint caption schema is a hybrid structure of "interleaved inline tags + a trailing soundscape block," covering three tracks simultaneously — visual, speech, and non-speech auditory:
(1) Visual track: natural-language narration of visual events, unfolding in chronological order, forming the main body of the caption.
(2) Speech track: embedded inline within the main body via <S>...<E> tags at the corresponding temporal position, achieving "temporal interleaving of visual events and dialogue" — a key design for aligning semantics and timing (roughly equivalent to weaving a script together with shot descriptions).
(3) Non-speech auditory track: encapsulated at the end via <AUDCAP>...<ENDAUDCAP> (in v1.1, "Audio: ..."), forming a complete soundscape description block. Its content adaptively switches based on whether the clip contains speech:
   - Clips with speech: emphasize the speaker's acoustic attributes — age, gender, accent, pitch, prosody, emotion, speaking rate;
   - Clips without speech: describe sound effects, background audio, and musical elements.
[Positioning relative to other schemes] Unlike LTX-2's unified full-soundscape description, Script-a-Video's fully factorized independent field streams, or Foley-Omni's three-field split, Ovi chooses "not to split into independent conditions" — all three tracks are packed into the same string and the same T5 embedding, distinguished by tags rather than by fields.
[Ablation verification] Section 5.5's sole ablation targets exactly this point: the initial design used a CLAP encoder to encode the non-speech audio description and T5 to encode the speech transcript, attempting to decouple the T2A and TTS tasks; the result was that this separated setup limited the model's ability to generate coherent joint output — it could do sound effects alone or speech alone, but struggled to fuse the two into a unified, coherent audio track. After switching to a merged single T5 embedding, WER stayed roughly flat (0.033→0.035) while FD_PANNs dropped from 20.78 to 18.03, FD_VGG dropped from 7.13 to 5.02, IS rose from 8.34 to 11.20, and CLAP rose from 0.190 to 0.224. This provides direct quantitative evidence that "a joint AV caption should be uniformly encoded rather than factorized."
[Alignment for pure-audio data] The pure-audio corpus likewise uses the same MLLM to produce a "transcript + audio description" pair (left blank if no speech), keeping the schema consistent with the audio-video side, ensuring the conditioning distribution does not drift across the two training stages.

### Dialogue transcription and speaker attribute annotation (ASR transcription + speaker identity/language/accent/emotion) ⚠️

[Transcription] The audio-understanding MLLM produces dialogue text directly from the full audio track, writing it into <S>...<E> tags interleaved chronologically with the visual description; the pure-audio data likewise produces a transcript, left blank when there is no speech. There is no mention of using an independent ASR (e.g. Whisper) or of any transcription quality verification/WER filtering [Uncertain].
[Speaker attributes] For clips containing speech, the audio description is explicitly required to emphasize the speaker's related acoustic attributes: age, gender, accent, pitch, prosody, emotion, speaking rate. This attribute set allows the model to control timbre and emotion directly via the prompt at inference time; Ovi 1.1 further strengthens this into "prompt-based emotion instruction tags."
[Speaker identity/separation] No mention of speaker diarization, speaker ID annotation, multi-speaker separation annotation, speaker embeddings, or reference-timbre conditioning ("Reference voice condition" remains unimplemented in the README's Todo), so in multi-speaker dialogue scenes there is a lack of explicit supervision for "which line belongs to which person" [Uncertain].
[Language annotation] Accent is annotated, but whether language is separately annotated is not stated [Uncertain].

### Geometric and structured annotation (camera parameters, depth, 3D point tracks, action annotation, explicit state) ⚠️

[Uncertain]. Ovi has no geometric/structured annotation step of any kind: no camera intrinsics/extrinsics, no camera-motion labels, no depth maps, no 3D point tracks, no pose/keypoints, no action-classification labels, no segmentation masks, and no explicit state annotation.
The two items that come closest are: (1) the motion score produced by RAFT (a scalar motion-intensity value, not structured geometric information); (2) the face-presence/count output by the internal face-detection model (used only for data-composition mixture control, not injected into the model as a condition).
The paper specifically emphasizes that its method "does not require face bounding boxes or face masks" or other geometric heuristic priors (in contrast to HunyuanVideo-Avatar, which uses a mask to constrain audio features to the facial region), treating the lack of dependence on geometric annotation as a design advantage — lip sync is entirely learned spontaneously from data and cross-modal attention; Section 5.1's attention visualizations show speech tokens automatically focusing on the mouth, drum sounds focusing on the drum, and animal sounds focusing on the corresponding body part.

### Synthetic data construction (controlled perturbation/editing to construct training pairs, e.g. InstructAV2AV) ⚠️

[Uncertain]. The paper describes no synthetic-data-construction step of any kind: no controlled-perturbation-constructed training pairs, no editing-style paired data (e.g. of the InstructAV2AV type), no TTS-synthesized speech fed back in, no audio-visual-mismatch negative-sample construction, and no audio-track-replacement/shift data augmentation. All audio-video training samples are genuinely collected native paired data.
The only "artificial construction" is at the data-format level: the pure-audio fine-tuning data is padded to an exact 5.04 seconds to match the 121-frame video duration; and on the inference side, it is recommended to use GPT to expand the user prompt according to the training caption template (an inference-time, not training-time, synthesis).

### Degree of human involvement (human annotation, human quality inspection, model-prescreen + human review) ⚠️

[Uncertain: almost no description on the data side]. Confirmable human involvement is concentrated at two ends:
(1) Data side: the acknowledgments mention that Yi Cui, Manav Shah, and Diego De La Torre "contributed to data preparation," indicating dedicated personnel were responsible for data-preparation work, but whether their role was manual annotation, manual quality inspection, or engineering-pipeline building is not stated. The main text's four-step data pipeline is entirely automated by models/rules, with no human-review step appearing, so the overall process can be judged to be "a fully automated pipeline, with no per-item human review." The "extensive experiments" on the captioning prompt represent human involvement at the level of template design.
(2) Evaluation side: human involvement is substantial and is the primary evaluation method — organizing a blind pairwise preference study with 50 participants, comparing Ovi against JavisDiT and UniVerse-1 across three dimensions — audio quality, video quality, and audio-video synchronization — reporting a Pairwise Win Rate (PWR).
No mention of RLHF preference annotation or a data-cleaning spot-check rate is found.

## Audio-Video Alignment

### Audio-video synchronization detection methods (lip sync, event alignment) ⚠️

Ovi treats audio-video synchronization filtering as the single highest-priority step in its entire data pipeline, and it is the only filter the authors describe as "strict."
[Method] Uses the classic SyncNet (Chung & Zisserman, 2016) — a ConvNet-based model that learns a joint embedding between sound and mouth-region imagery, outputting two scalars: confidence (measuring the strength of audio-visual correlation) and offset (measuring how far the audio track leads/lags the video, in frames). The authors explicitly state they engineered the model to handle data "on the scale of millions" of videos — i.e., they converted SyncNet from a research-scale script into a large-scale batch-processing component.
[Decision logic] Only clips satisfying both |offset| ≤ 3 (frames) and confidence > 1.5 are kept, layered with a condition of mean audio volume ≥ −60 dB.
[Scope of application] Primarily targets "speech videos" (clips containing speech), used to remove audio-video-desynchronized data; for non-speech clips, event-level audio-visual alignment (e.g. an impact sound with the corresponding impact visual) uses no dedicated detector, nor is an AV-align-style event-alignment metric used [Uncertain].
[Design rationale (an important lesson)] The paper states explicitly: "We have experimentally determined that even a small quantity of out-of-sync data can impede lip-sync abilities and chose these strict criteria to minimize the risk of misaligned data." — i.e., experiments demonstrated that even a tiny amount of desynchronized data is enough to harm the model's lip-sync ability, so the team chose to sacrifice data volume in favor of strict thresholds to minimize misalignment risk. This is the most direct statement in the entire paper of "data quality over quantity."
[No training-time sync loss] After strict data-side filtering, the training stage uses no explicit synchronization loss, no auxiliary synchronization module, and no face mask at all; synchronization relies entirely on paired sampling + shared timestep + bidirectional cross-modal attention + scaled RoPE emerging spontaneously. It could be said that Ovi bets almost the entirety of its synchronization guarantee on data filtering.

### Specific sync-detection metrics and thresholds (SyncNet/Synchformer/LSE/in-house, threshold values, e.g. MOVA's LSE-D≤9.5 ∧ LSE-C≥4.5, SkyReels-V4's SyncNet |offset|≤3 ∧ conf>1.5) ⚠️

The metrics and thresholds are all explicitly disclosed, making this the most concretely disclosed part of this work's data side:
- Detector: SyncNet (Chung & Zisserman, 2016), internally engineered to support batch processing at the scale of millions of videos.
- Metric 1: offset (audio-video time offset, in frames). Threshold: |offset| ≤ 3. At 24fps this is a tolerance of ≤ 125 milliseconds.
- Metric 2: confidence (SyncNet confidence). Threshold: confidence > 1.5.
- Metric 3 (a joint audio-side threshold): mean volume ≥ −60 dB (mean volume no lower than −60 decibels), used to exclude near-silent clips or clips with an extremely weak audio track.
- Combination logic: all three are combined with "AND" — all must be satisfied simultaneously for a clip to be kept.
[Cross-comparison] This confidence > 1.5 threshold is moderately strict (SyncNet confidence commonly ranges roughly 0-10+; some works such as UniTalking use >0.9, while talking-head datasets commonly use >3), while |offset| ≤ 3 frames is a relatively strict time tolerance. The authors self-assess these as strict criteria.
[Not disclosed] The pass rate, the ablation curves for each threshold, and the experimental values behind the threshold choices are all not given [Uncertain].

### Separate handling of temporal sync vs. semantic sync (temporal alignment and content-semantic matching treated as two independent filtering conditions) ⚠️

At the data-filtering level, Ovi essentially only addresses "temporal synchronization"; semantic matching is mainly left to the caption and the training mechanism, and the two are not designed as two parallel, independent filtering conditions.
[Temporal sync (has explicit filtering)] SyncNet's dual thresholds on offset and confidence, specifically constraining the temporal alignment of speech and mouth movement; the mean-audio-volume threshold excludes ineffective audio tracks.
[Semantic sync (no explicit filtering)] No CLAP/AV-CLIP-style cross-modal semantic-similarity scoring is used to remove samples where "audio-visual content is mismatched" (for example, a beach scene paired with an unrelated voice-over or post-production music track). Semantic consistency instead relies on two indirect paths: (1) at the captioning stage, the MLLM watches both the frames and listens to the audio track simultaneously and writes both parties' content into the same description, thereby forcing semantic binding at the text-conditioning level; (2) during training, bidirectional cross-modal attention lets the two towers serve as each other's semantic context.
[Potential gap] Samples such as voice-overs, post-production scoring, or dubbed/localized audio — where the timing may be aligned but the semantics bear no causal relationship to the visuals — are difficult for SyncNet alone to fully remove (SyncNet's confidence is naturally low for clips lacking mouth-region visuals, which can partially and indirectly filter such cases); the paper does not discuss this risk [Uncertain].
[Architectural division of temporal/semantic labor] Notably, Ovi makes an explicit division of labor between the two at the model level: timing is handled by scaled-RoPE (frequency scaling of 31/157 ≈ 0.197), and semantics is handled by bidirectional cross-attention — the abstract itself phrases this as "blockwise exchange of timing (via scaled-RoPE embeddings) and semantics (through bidirectional cross-attention)." This idea of "separate handling of temporal vs. semantic synchronization" is embodied in the architecture rather than in data filtering.

### Audio quality filtering (SNR, silence detection and silence-ratio thresholds, removal of no-audio-track clips, removal of off-screen sound sources, background-music separation) ⚠️

Audio-side filtering is relatively simple, consisting of one hard threshold plus a few implicit constraints:
(1) Volume threshold: mean volume must be ≥ −60 dB, applied simultaneously with the dual SyncNet thresholds. Its effect is to remove silent, near-silent, corrupted, or extremely-low-volume audio tracks — i.e., it implicitly accomplishes "removal of clips lacking a valid audio track."
(2) No explicit SNR/noise filtering: the paper does not describe an SNR threshold, noise suppression, reverberation filtering, or clipping/distortion detection [Uncertain].
(3) No background-music separation: no source separation (e.g. Demucs) is used to split BGM from vocals — in fact Ovi deliberately preserves the native mixed audio track, since its goal is precisely to generate a complete soundscape of speech + sound effects + BGM simultaneously.
(4) No explicit off-screen-sound-source removal module: only SyncNet's low-confidence output provides indirect filtering [Uncertain].
(5) Silence-ratio threshold: only the "mean volume" global metric exists; no per-segment silence-ratio statistic is seen [Uncertain].
(6) Sample-rate and bandwidth constraints (an engineering-level detail): audio uniformly goes through MMAudio's 16kHz encoder variant (STFT → mel spectrogram → 1D VAE latent); the paper's Limitations section acknowledges that this fixed 16kHz 1D-VAE path limits bandwidth and spatial perception, flattening high-fidelity music, spatial cues, and subtle timbral detail, with future work possibly switching to a higher-bandwidth latent or applying post-hoc bandwidth extension. At inference, the BigVGAN vocoder restores the waveform.

### Classification and separate handling strategies for speech/sound effects/music ⚠️

One of Ovi's core claims is that a unified audio model must cover both speech and sound effects simultaneously rather than handling them separately, yet the data is still organized by type when processed:
[Data-source typing] Speech mainly comes from the internal corpus (the main body of pretraining); sound effects come from VGGSound / AudioSet / WavCaps (introduced during fine-tuning); music has no independently identified source, appearing mainly as BGM within the video's native audio track [Uncertain].
[Training-stage typing] Audio pretraining is predominantly speech (learning speaker identity, pitch, emotion, prosody); the audio fine-tuning stage introduces diverse sound effects, while also mixing back in audio-video audio tracks to keep TTS ability alive, making Ovi-Aud a unified audio backbone that can do both T2A and TTS simultaneously.
[Captioning typing] Bifurcated as "contains speech / no speech": for clips with speech, the audio description focuses on the speaker's acoustic attributes; for clips without speech, it focuses on the three categories sound effects / background audio / musical elements. In pure-audio data, the transcript field is left blank if there is no speech.
[No typing at the conditioning-encoding level (a key decision)] An earlier version used CLAP to encode the non-speech description and T5 to encode the speech transcript in order to "decouple T2A and TTS," but the ablation showed this separated scheme causes the model to fail at fusing sound effects and speech into a coherent audio track; the final version merges everything into a single T5 embedding.
[Evaluation typing] T2A is reported per the MMAudio protocol at FD_PANNs 18.03 / FD_VGG 5.02 / IS 11.20 / CLAP 0.224; TTS is reported on Seed-TTS test-en at WER 0.035. Both are close to their respective specialized models (e.g. MMAudio-L's FD_PANNs of 15.04, F5-TTS's WER of 0.018); the paper argues the unified model need not surpass specialized models — being able to competently handle both simultaneously is the precondition for audio-video fusion.

## Training Coordination

### Multi-stage training curriculum and data-curriculum scheduling (basis for stage division: resolution/duration/quality score/modality; low-res→high-res, image→video, short→long) ⚠️

A three-stage curriculum, with the division basis being "modality → duration → modality fusion" rather than the more common resolution or quality score:
[Stage 1 | Audio pretraining (Ovi-Aud pretraining)] Trains the audio tower from scratch (architecture copied from Wan2.2 5B), on hundreds of thousands of hours of variable-length raw audio up to 12 seconds long, predominantly human speech. The goal is to solidly build long-range modeling of speaker identity, pitch, emotion, and prosody. 50k steps, batch 2880, lr 1e-4, AdamW (β1=0.9, β2=0.999, ε=1e-8). A key engineering detail: scaled RoPE is applied to all attention layers starting from pretraining, thereby avoiding re-adaptation when entering the audio-video stage and avoiding the need to maintain multiple sets of audio RoPE scales.
[Stage 2 | Audio fine-tuning] Switches to waveforms padded to an exact 5.04 seconds (aligned to 121 frames @24fps), mixing in VGGSound/AudioSet/WavCaps sound-effect data plus audio tracks extracted from the internal audio-video corpus, so the audio distribution matches downstream fusion training, while also supplementing sound-effect capability and keeping TTS ability alive.
[Stage 3 | Audio-video fusion fine-tuning] Splices the already-pretrained audio tower onto the Wan2.2 5B video tower; the cross-modal attention layers are initialized from scratch, and all FFNs are frozen to save memory — of the 11B total, 5.7B parameters are trainable (only the single-modality self-attention and the two types of cross-attention, text-to-modality and modality-to-modality, are trained). 40k steps, batch 768, lr 5e-5, AdamW (β1=0.9, β2=0.95, ε=1e-8). The loss is a weighted sum of the two modalities' flow-matching losses, λ_v=0.85, λ_a=0.15, sharing the same timestep with independent noise per modality.
[Subsequent (Ovi 1.1)] The README Todo has already checked off "Finetune model with higher resolution data, and RL for performance improvement" and "Longer video generation (10s)," i.e., a higher-resolution (native 960×960) fine-tuning stage, RL optimization, and duration extension were added, forming a de facto "low-res short clips → high-res long clips" curriculum; the data and training details of this stage are not disclosed [Uncertain].
[Not adopted] An image-to-video progressive curriculum, a low-res-to-high-res multi-resolution curriculum (the initial version is entirely 720×720 equal-area fixed length throughout), and a quality-score-stratified curriculum schedule do not appear in the paper.

### How data mixture changes across training stages (pretraining/annealing/SFT high-quality subset) ⚠️

The change in mixture across stages is explicit and clearly motivated, but lacks numerical values:
(1) Audio pretraining → audio fine-tuning: switches from "predominantly human speech, up to 12 seconds variable length" to "5.04 seconds fixed length + large amounts of sound effects (VGGSound/AudioSet/WavCaps) + internal video audio tracks." Two adjustments happen at once: the duration distribution narrows to align with the downstream stage, and the audio-category mix shifts from speech-only to a speech-plus-sound-effects mix. The purpose of mixing back in the internal video audio tracks is explicitly stated as to "maintain TTS abilities and better align with the downstream goal" — a typical anti-forgetting replay mixture. The mixing-ratio value is not disclosed [Uncertain].
(2) Audio-video fusion stage: data is entirely switched to the internal audio-video paired corpus that has been cleaned by the four-step pipeline; pure audio data no longer participates [Uncertain: whether a small amount of audio data is still mixed in].
(3) At the loss-weighting level, a "modality mixture": λ_v=0.85 / λ_a=0.15, with the video loss weighted far higher than the audio loss — this can be viewed as a training-objective tilt toward video fidelity, to mitigate the video-quality degradation that fusion training can cause.
(4) Ovi 1.1: the dataset is doubled in size and switched to native 960×960 data; changes to the mixture structure are not disclosed [Uncertain].
(5) There is no annealing stage and no described high-quality SFT subset [Uncertain].

### Post-training data (scale and selection criteria of the SFT curated set, number of preference pairs and annotation method, reward-model training data) ⚠️

The initial paper contains no post-training in the traditional sense: no SFT curated set, no preference pairs, no reward model, and no DPO/RLHF data description. What the paper calls "audio post-training" is actually the audio tower's 5-second fixed-length fine-tuning stage (see stage 2 of multi_stage_curriculum), which is distribution adaptation rather than preference optimization.
[Ovi 1.1's RL] The README Todo item "Finetune model with higher resolution data, and RL for performance improvement" has been checked off complete, indicating that version 1.1 introduced reinforcement-learning optimization, but the RL algorithm, the reward signal (whether based on synchronization/aesthetics/human preference), the scale of preference data, and the annotation method are all undisclosed, with no corresponding technical report [Uncertain].
[The only scenario in which human preference data appears] The 50-person blind pairwise preference study is used only for evaluation (the PWR metric) and is not fed back into reward-model training [Uncertain].
[Distillation] The README Todo item "Distilled model for faster inference" remains incomplete; the paper's Limitations section proposes that a step-distillation framework such as DMD2 could be used, listed as future work.

### Data-processing infrastructure and throughput (NeMo Curator/Data-Juicer/in-house, GPU acceleration ratio, processing scale, cost) ⚠️

Disclosure is extremely limited. Confirmable engineering details include:
(1) SyncNet's scale-up engineering: the paper explicitly states, "We adapt the model to handle video data on the scale of millions" — converting a research-scale SyncNet into a batch-processing component capable of handling millions of videos, the only named infrastructure engineering effort in the data pipeline.
(2) Data packing into bytes: the packing stage frame-samples the video at 24fps and converts it into a byte array, and converts audio into raw wave bytes, feeding the training side in byte form — a storage-format design aimed at high-throughput reading.
(3) Training-side infrastructure: entirely bf16 precision, using DeepSpeed (Rasley et al., 2020) for sharded distributed data parallel (sharded DDP); audio pretraining uses batch 2880 and fusion training uses batch 768, implying a fairly large-scale GPU cluster was used, but card count, card type, training duration, and total compute cost are all undisclosed [Uncertain].
(4) No use of, or mention of, established data-processing frameworks such as NeMo Curator or Data-Juicer, and no GPU acceleration ratio, processing throughput (clips/hour), or processing cost is given of any kind [Uncertain].
(5) Inference-side (not data-side) disclosure is more complete: single-card peak VRAM is about 80GB (with FlashAttention-3); standard 32GB, fp8-quantized 24GB is runnable; multi-card sequence parallelism is supported; the Todo items for FSDP sharded inference and sequence-parallel efficiency optimization remain incomplete.

## Effectiveness Comparison

### Quantified impact of data-strategy ablations (distinguishing: filtering-strictness ablation / caption-density-style ablation / data-mixture ablation, and corresponding evaluation metrics) ⚠️

The paper contains only one formal ablation experiment, and it belongs to the "caption/conditioning-encoding structure" category rather than the "filtering-strictness" or "data-mixture" categories — but it happens to be the key piece of evidence for the joint AV caption design.
[Ablation 1 (the sole formal ablation, Section 5.5 + Table 3): separated vs. merged encoding of audio conditioning]
- Variant A "Ovi with CLAP": uses a CLAP text encoder to encode the non-speech audio description and T5 to encode the speech transcript, attempting to decouple the T2A and TTS tasks and avoid mutual interference.
- Variant B "Ovi" (the final scheme): merges the speech transcript and the audio description into a single string, encoded with a single T5 embedding.
- Results: FD_PANNs 20.78 → 18.03 (↓2.75), FD_VGG 7.13 → 5.02 (↓2.11), IS 8.34 → 11.20 (↑2.86), CLAP 0.190 → 0.224 (↑0.034), WER 0.033 → 0.035 (essentially flat, no loss of linguistic correctness).
- Conclusion: separated encoding can each handle sound effects or speech individually, but cannot integrate the two into a unified, coherent audio track; merged encoding significantly improves audio fidelity and text-audio alignment while preserving linguistic correctness, and lets the two towers share the same T5 representation, simplifying cross-modal modeling and enhancing multimodal coherence.
[Ablation 2 (architecture side, Figure 2, qualitative): scaled RoPE] Without scaling, the cross-modal RoPE affinity matrix's diagonal is misaligned, hindering synchronization; after scaling by 31/157 ≈ 0.197, the diagonal aligns sharply. Only a qualitative visualization is given, with no quantitative metric.
["Experimental conclusion" on filtering strictness (no table)] The paper claims to have "experimentally determined that even a small quantity of out-of-sync data can impede lip-sync abilities," i.e., experiments on varying proportions of out-of-sync data were conducted and the strict threshold was chosen based on them, but the experimental setup and quantitative results are not given [Uncertain]. This is the most worth-pursuing-yet-undisclosed data ablation in the entire paper.
[The cost of video quality (not a formal ablation, but an important observation)] Compared to the base model Wan2.2, Ovi's video quality shows slight degradation, which the paper attributes to "the audio-video dataset used for joint training being much narrower than Wan2.2's pretraining corpus" — a quantitatively perceptible cost from narrowed data coverage, which also indirectly corroborates that strict synchronization filtering reduces data diversity.
[Missing] Ablations of caption density/style, ablations of data mixture, and threshold-sweep experiments (the effect curve of varying SyncNet confidence or offset) are all not provided [Uncertain].

### Evidence on quality vs. quantity (cases where small, precise data beats large, messy data) ⚠️

There is a clear qualitative claim, but it lacks a controlled-experiment comparison demonstrating that "a small, precise dataset surpasses a large, messy one."
[Positive evidence (author's own account)] The only direct statement appears at the sync-filtering step: "even a small quantity of out-of-sync data can impede lip-sync abilities, hence choosing strict standards to minimize the risk of misalignment." This statement expresses the classic quality-first principle — better to substantially reduce usable data volume than to tolerate even a small amount of dirty data contaminating synchronization ability. The design orientation of the entire pipeline (the hard >720×720 threshold, RAFT static removal, aesthetic removal, SyncNet's strict dual thresholds) is consistent with this.
[Counter-evidence (the cost)] Quality-first is not without cost: the paper acknowledges that after joint training, video quality shows slight degradation relative to the Wan2.2 base model, precisely because "the audio-video paired dataset is narrower than Wan2.2's large-scale pretraining corpus" — i.e., the high purity gained from strict filtering comes at the cost of insufficient coverage breadth. The authors consider this trade-off marginal and acceptable.
[Evidence that scale is still pursued] One of Ovi 1.1's main improvements is precisely "Dataset includes 100% more videos for greater diversity" — indicating the team, while maintaining the quality bar, still primarily uses scale expansion as its main lever for improvement: quality is the gate, quantity is the lever.
[No quantitative comparison] No side-by-side training results of "a strictly filtered subset vs. a loose large set" are provided [Uncertain].

### Alignment between training-data domain distribution and evaluation-benchmark taxonomy (e.g. VABench's seven major categories) ⚠️

The alignment relationship is weak; there is no deliberately designed mapping between the training-data domain distribution and the evaluation taxonomy system.
[Evaluations used] (1) Joint AV generation: a 50-person blind pairwise preference study on Verse-Bench (proposed by the authors of UniVerse-1, Wang et al., 2025, not self-built by Ovi), evaluated along only three dimensions — audio quality, video quality, and audio-video synchronization — with Ovi winning by a significant margin over both JavisDiT and UniVerse-1 on all three. This is a "quality-dimension" rather than "content-taxonomy" evaluation system, with no category-level correspondence to the training data's domain distribution. (2) Audio tower: T2A follows MMAudio's evaluation protocol (FD_PANNs / FD_VGG / IS / CLAP), whose data distribution is naturally homologous with the sound-effect taxonomy of VGGSound/AudioSet introduced during training — the only place on the audio side where any taxonomy alignment exists. (3) TTS: WER is reported on Seed-TTS test-en, covering English only. (4) Video side: relative comparison against the base model Wan2.2 to confirm no degradation from joint training.
[No VABench-style taxonomy alignment] The paper neither builds nor aligns to any seven-category or multi-category evaluation taxonomy, and does not report metrics grouped by content category; the only explicit mixture control on the training side (single-person/multi-person/person-free) has no corresponding grouped results in evaluation [Uncertain].
[Gap] No standard video benchmark scores such as VBench or VideoScore are reported, and no objective synchronization metrics such as AV-Align or SyncNet confidence are reported (synchronization is judged by human evaluation only), leaving the connection between the data strategy and evaluation without a traceable, quantitative closed loop.

## Uncertain Fields

The research information for the following fields is partially uncertain (⚠️-marked sources):

- data_scale
- data_sources
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
- caption_structure
- dialogue_transcription_attributes
- geometric_structured_annotation
- synthetic_data_synthesis
- human_in_loop
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
