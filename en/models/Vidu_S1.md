# Vidu S1 (technical report "Vidu S1: A Real-Time Interactive Video Generation Model," arXiv:2607.03118v2; product name Vidu Stream, https://vidu.com/vidu-stream)

> Topic: Data processing for video generation models (including simultaneous audio-video generation): data cleaning pipeline, data distribution, annotation methods

[← Back to Home](../index.md)

**Table of Contents**: [Basic Information](#basic-information) · [Data Scale and Distribution](#data-scale-and-distribution) · [Filtering Pipeline](#filtering-pipeline) · [Annotation/Captioning Method](#annotationcaptioning-method) · [Audio-Video Alignment](#audio-video-alignment) · [Training Integration](#training-integration) · [Performance Comparison](#performance-comparison)

## Basic Information

### Name

Vidu S1 (technical report "Vidu S1: A Real-Time Interactive Video Generation Model," arXiv:2607.03118v2; product name Vidu Stream, https://vidu.com/vidu-stream)

### Publishing Organization/Company

Shengshu Technology (生数科技) in collaboration with Tsinghua University (清华大学). The authors include Jintao Zhang, Kai Jiang, Jintao Chen, and others (27 authors in total), with advisors Zhijie Deng, Fan Bao (鲍凡), Jianfei Chen, and Jun Zhu (朱军).

### Release Date (Technical Report/Paper/Open-Source Release)

July 2026: on July 6, 2026, Shengshu Technology founder Jun Zhu officially launched the product at the Global Digital Economy Conference (全球数字经济大会); the arXiv preprint 2607.03118, v2 dated July 21, 2026 (cs.CV, 13 pages).

### Type (Model/Dataset/Toolchain/Benchmark)

Model (a real-time interactive streaming audio-video joint generation model, for speech-driven digital humans/virtual characters). The team also built a companion evaluation benchmark, Vidu-StreamBench (500 samples, an internal benchmark, not seen to be open-sourced).

### Openness (Whether Weights/Code/Data/Pipeline Are Open-Sourced)

Closed-source. The technical report only discloses the architecture and training-framework ideas; it does not open-source the model weights, inference/training code, data, or data processing pipeline. The in-house acceleration and serving components TurboDiffusion and TurboServe are likewise not open-sourced. The self-built benchmark Vidu-StreamBench has not been publicly released. An interactive online demo is available only via the official website https://vidu.com/vidu-stream. The paper is published under a CC-BY 4.0 license.

### Whether Joint Audio-Video Generation Is Supported, and How (Native Joint/Cascaded/MoE Fusion)

Supported, and implemented as native joint generation. The model concatenates the clean video representation v_0^i of frame i with the audio representation a_0^i along the modality dimension into a joint state x_0^i = [v_0^i; a_0^i], and performs unified denoising on the joint video-audio latent sequence within the same diffusion denoising model — neither cascaded nor MoE. The unified conditioning interface c simultaneously includes speech, text prompts, and a reference first-frame image — that is, speech serves both as a conditioning control signal (the user's real-time voice commands control character behavior) and, within the joint state, as the generation target for the audio track. Overall, this is a causal streaming generation paradigm combining autoregression and diffusion (AR+Diffusion), with sliding-window decoding supporting infinite-length generation.

### List of Research Information Sources (URLs of Papers/Technical Reports/Official Documentation/News; Nature of Each Source Noted: Official Primary/Same-Team Corroboration/Third-Party Reporting)

1) Official primary: arXiv:2607.03118v2 "Vidu S1: A Real-Time Interactive Video Generation Model" https://arxiv.org/abs/2607.03118 (including Section 2.1 Data Preparation and Figure 2, the data-filtering-pipeline diagram; nearly all data-side information in this research comes from this section)
2) Official primary: product page and online demo https://vidu.com/vidu-stream
3) Third-party reporting: China Daily Tech, "Shengshu Technology Releases Vidu S1, Pushing Video Generation into a New Era of 'Real-Time Interaction'" https://tech.chinadaily.com.cn/a/202607/06/WS6a4b12eea310d709c2fbbecb.html
4) Third-party reporting: Leiphone https://www.leiphone.com/category/industrynews/6GlFzI5hMwcfRoGZ.html ; iFanr https://www.ifanr.com/digest/1670950 (release timing, AR+Diffusion architecture, 540P/25-42FPS and other product parameters)
5) Third-party compilation: AI Tech In-Depth Analysis, Vidu S1 entry https://www.ai-all.info/ai-models/vidu-s1 (TurboDiffusion/TurboServe, consumer-grade GPU starting from RTX 3060, end-to-end latency <200ms, etc. — secondhand information)

## Data Scale and Distribution

### Training Data Scale (Number of Video Clips/Hours/Tokens, Pretraining vs. SFT Separately) ⚠️

[Uncertain]. The technical report discloses no absolute scale of the training data at all (no number of video clips, hours, or tokens given), nor does it distinguish between pretraining and post-training data volumes. It is only qualitatively described as "a corpus of high-quality, diverse, and highly interactive single-person, single-shot video." The data volumes used at each of the three training stages (bidirectional teacher training / causal autoregressive adaptation / DMD distillation) are likewise undisclosed.

### Data Source Composition (In-House/Public Datasets/Web Scraping/Licensed Procurement/Synthetic Data)

Explicitly two major categories of raw video sources (neither states whether the acquisition channel is in-house, purchased, or scraped):
(1) Livestream or talking-head videos — mainly used to learn fine-grained features such as facial expressions, body movements, and lip sync;
(2) High-quality footage from films and television dramas — used to improve the model's generalization and consistency across different camera angles, scenes, and visual styles.
No mention is made of using public datasets, licensed procurement, or synthetic data. On the evaluation side, the public benchmark HDTF is used as a reference.

### Data Compliance and Provenance (Proportion of Licensed Data, Rights-Cleared Datasets, C2PA, etc.) ⚠️

[Uncertain]. The report makes no mention of data licensing, copyright compliance, the proportion of rights-cleared datasets, C2PA/watermark provenance, or similar information. It only mentions, within the cleaning stage, content safety filtering (removal of NSFW and other inappropriate content) and frame-cleanliness filtering (removing clips containing watermarks, subtitles, or overlaid advertisements) — the latter is motivated more by image quality than by copyright concerns. Given that the data sources include film and television footage, the absence of copyright-provenance disclosure is a clear gap.

### Clip Duration Distribution and Segmentation Strategy

After segmentation, training clips are single-shot clips ranging from 3 to 60 seconds. Segmentation strategy: first split along shot boundaries to ensure single-shot continuity; long shots are further subdivided, and cut points are constrained so as not to fall in the middle of speech, in order to protect the integrity of speech-lip sync. No specific duration-distribution histogram or average duration is given.

### Resolution/Aspect Ratio Distribution and Bucketing Strategy ⚠️

[Uncertain]. On the training side, "frame rate and resolution" are used only as technical thresholds during the pre-filtering stage to remove low-frame-rate/low-resolution videos; no specific thresholds, resolution distribution, or bucketing strategy are disclosed. On the inference/output side, the standard is 540p (960×540) at 25 FPS, with a maximum of 42 FPS (measured on an RTX 5090); no mention of multi-aspect-ratio support.

### Category/Domain Distribution and Mixture Strategy (Proportional Control and Concept Balancing across People, Actions, Scenes, Styles, etc.) ⚠️

No quantitative mixture ratios are given, but there are strong qualitative domain constraints, which in essence constitute a highly vertical data distribution:
(1) Subject constraint: each clip must contain exactly one subject, with the subject occupying a reasonable proportion of the frame — i.e., a strict "single-person" distribution;
(2) Shot constraint: only static shots or shots with slow motion are retained, to reduce the risk of shot drift during long-duration generation — deliberately suppressing the proportion of large camera movements;
(3) Interactivity constraint: the subject in frame must display clear actions or behaviors, to ensure the model learns meaningful motion information;
(4) Style coverage: film and television footage supplements multiple camera angles, scenes, and visual styles; the product side supports multiple customizable character appearances including real people, anime, and pets, implying that the training data covers anime-style and non-human subjects (the paper notes that the face-detection expert model generalizes poorly to exaggerated or highly stylized 2D animated subjects, hence the introduction of the omni model as a supplement);
(5) Semantic label dimensions: the omni model labels along nine quality dimensions — editing, subject, action, emotion, face, speech, scene, shot, and tone — which together form a de facto domain description system, though the paper does not disclose the proportional figures for each dimension.
The ratio between the two major sources (livestream/talking-head vs. film and television footage) is likewise undisclosed [uncertain].

### Audio Category Distribution and Mixture (Proportions and Control Strategy for Speech/Sound Effects (Foley)/Music/Ambient Sound/Silence) — A Dimension Unique to AV Models ⚠️

No quantitative proportions are given [uncertain], but there is an explicit audio-category processing and filtering strategy, oriented toward "purifying speech, removing music":
(1) The speech component is first extracted from the raw audio;
(2) VAD (voice activity detection) + ASD (active speaker detection) are used to label the timestamp and associated speaker of each speech segment;
(3) Each speech segment is classified into one of three types — onscreen (the speaker is the person in frame), offscreen (voice-over, the speaker is not in frame), or overlap (multiple overlapping voices); clips containing an overlap segment are discarded in their entirety;
(4) To address the instability of the diarization model in singing and heavy-background-music scenarios (which tends to misclassify vocals into the music stem and produce synthetic timbre and distortion artifacts), a heuristic rule is introduced: if the speaker is vocalizing but the speech energy proportion is too low, the segment is discarded — the practical effect is to systematically remove singing- and music-dominated clips.
(5) At the caption level, sound effects and background music fields are still labeled, indicating that these two types of audio information are retained as describable attributes, though speech remains the overwhelming majority of the training distribution.

### Narrative Structure Distribution (Single-Shot vs. Multi-Shot, Average Clip Duration, Shot-Count Distribution, Whether Native Audio Tracks Are Included) ⚠️

All clips are single-shot, explicitly excluding multi-shot narratives: raw videos are split along shot boundaries into single-shot clips, with long shots further subdivided, ultimately yielding single-shot, single-person clips of 3–60 seconds. All training data includes a native audio track (the pre-filtering stage already removes samples with missing audio tracks or incomplete audio-video using the "audio-visual integrity" criterion). No values are given for average clip duration or shot-count distribution [uncertain].

### Language/Accent Distribution (Data Basis for Multilingual Lip-Sync Capability) ⚠️

[Uncertain]. The paper does not disclose the language or accent distribution of the training data, nor does it explain the data basis for multilingual lip-sync capability. It only includes a dialogue field in the caption structure, and mentions coverage of diverse "speaker attributes" and emotions in the Vidu-StreamBench benchmark, without elaborating on the language dimension. The product targets the Chinese market, suggesting a Chinese-language predominance, but there is no primary evidence for this.

## Filtering Pipeline

### Overall Structure of the Filtering Funnel (Number of Filtering Stages, Order)

A six-stage progressive filtering funnel (explicitly diagrammed in Figure 2 of the paper, from Raw Videos to the final training data):
Stage 1, Prefiltering: deduplication first, then removal of unreliable videos based on four technical metrics — frame rate, resolution, audio-visual integrity, and audio-visual synchronization → output: Pre-filtered;
Stage 2, Clipping: split along shot boundaries into single-shot clips, with long shots subdivided and cut points avoiding the middle of speech, plus a Duration Filter retaining 3–60 second clips → output: Single-Shot Clips;
Stage 3, Subject Filtering: the Subject Filter ensures exactly one subject in frame with a reasonable proportion → output: Single-Person Clips;
Stage 4, Other Filtering: four parallel filters — Frame Content / Visual Quality Filter, Content Safety Filter, Shot Stability Filter, and Interactivity Filter;
Stage 5, Diarization: VAD + ASD label speech segments and speakers, classified into onscreen/offscreen/overlap; the Overlap Filter removes clips with overlapping voices, and the Speech Energy Filter removes clips with a speech energy proportion that is too low (singing/heavy background music);
Stage 6, Caption + Embedding: generates structured captions at two granularities — clip-level and speech-aware chunk-level — and the finally selected clips are embedded into training data together with their captions.
The design goal is summarized as simultaneously improving visual clarity, temporal stability, audio-visual consistency, and cross-modal interpretability.

### Quantitative Retention Rate of the Funnel (Input/Output Volume and Final Retention Rate at Each Stage, e.g., Apollo's 27%) ⚠️

[Uncertain]. Although the paper provides a complete six-stage funnel diagram (Figure 2), it gives no input/output data volumes or retention rates for any stage, nor a final retention rate. This is one of the main gaps in this report's data disclosure (compared to work such as Apollo, which gives a 27% retention rate).

### Shot Segmentation Method (PySceneDetect/In-House Model/Shot-Aware Splitting) ⚠️

Raw videos are split into single-shot clips along shot boundaries; long shots are further subdivided. The key constraint is that "cut points must not fall in the middle of speech" (speech-aware cutting), a segmentation strategy customized for the lip-sync task. It is not stated whether PySceneDetect, TransNetV2, or an in-house shot-boundary detection model is used [uncertain].

### Quality Filtering (Aesthetic Score, Sharpness, OCR Text Filtering, Black-Bar/Watermark/Logo Detection) ⚠️

A multi-dimensional quality filter combining expert models and large models, covering six areas of concern:
(1) Subject detection: each clip contains exactly one subject occupying a reasonable proportion of the frame;
(2) Frame cleanliness: clips containing information irrelevant to the visual content are removed, explicitly listing watermarks, subtitles, and overlaid advertisements;
(3) Visual quality: dual scoring via aesthetic scoring and technical scoring is used to select clear, complete, visually pleasing clips, avoiding artifacts such as blur, jitter, and flicker;
(4) Content safety: removal of NSFW and other inappropriate content;
(5) Shot stability: only static shots or shots with slow motion are retained, to reduce the risk of shot drift during long-duration streaming generation;
(6) Interactivity: the subject must display clear actions/behaviors.
No specific names or threshold values for any of the scorers are disclosed [partially uncertain]. There is no mention of the specific implementation of OCR text detection or black-bar detection.

### Motion Filtering (Optical-Flow/Motion-Score Thresholds, Removal of Static and Jittery Clips) ⚠️

There are two motion constraints pulling in opposite directions: on one hand, "shot stability" filtering retains only static or slow-motion shots, removing large camera movements/jitter (jitter is also removed under the visual-quality criterion); on the other hand, "interactivity" filtering requires the subject itself to display clear, discernible actions or behaviors, removing samples that are completely static with no action. In other words: "the shot should be stable, but the subject should move." There is no mention of using optical flow or specific motion-score thresholds [specific method and values uncertain].

### Deduplication Method (Exact Deduplication and Embedding-Based Semantic Deduplication Recorded Separately) ⚠️

At the very front of the pipeline (before formally entering the pipeline), all data are first deduplicated, and then pre-filtered. The paper does not distinguish between exact deduplication and embedding-based semantic deduplication, nor does it specify the features or similarity thresholds used [uncertain].

### VLM/LLM as Quality Judge (Multimodal Large-Model Quality Scoring and Mismatch Removal — the 2026 Trend of Shifting from Shallow Scorers to Large-Model Semantic Judgment) ⚠️

This is the most prominent methodological claim on the data side of this report, reflecting the 2026 trend of "shifting from shallow scorers to large-model semantic judgment": the paper explicitly points out that evaluating video data using expert models only has clear limitations — for example, face-detection models struggle to generalize to exaggerated or highly stylized 2D animated subjects; some expert models are image-based and can only judge extracted frames, producing misjudgments due to insufficient global information. As a supplement, an omni model (an all-modality large model, citing references [24, 25]) is therefore introduced to perform global semantic understanding of the full video, generating semantic labels along nine quality dimensions: editing, subject, action, emotion, face, speech, scene, shot, and tone. Together, the expert models and the omni model form a joint filtering system that combines global contextual awareness with local detail sensitivity. The specific identity and scale of the omni model are undisclosed [uncertain]. In addition, caption annotation is likewise performed by an annotation model.

### Safety and Compliance Filtering (NSFW, Copyright, Face/Privacy) ⚠️

Stage 4 of filtering includes a dedicated Content Safety Filter, which removes NSFW and other inappropriate content, motivated by preventing the model from learning harmful information. There is no mention of copyright filtering, face-privacy/portrait-rights handling, celebrity filtering, or similar mechanisms [uncertain].

## Annotation/Captioning Method

### Caption Model Used (In-House VLM/Open-Source Model, Model Scale) ⚠️

An "annotation model" is used to generate structured natural-language descriptions, and an omni model (all-modality large model) is used during the quality-filtering stage. The paper does not disclose whether these models are in-house or open-source, nor their parameter scale [uncertain].

### Caption Density and Degree of Structuring (Short/Long/Dense Descriptions, Structured Fields such as Camera Motion, Style Tags)

Dual-granularity and structured, customized for streaming interactive generation:
(1) Full-clip caption (whole-segment level): provides a coherent, global semantic anchor for the entire video;
(2) Speech-aware chunk-level caption: aligns descriptions with their corresponding time intervals, providing fine-grained, temporally localized conditioning signals for controllable interactive streaming generation — this is the key design that adapts the system to the autoregressive streaming paradigm, allowing users to alter subsequent content at any point during generation via voice commands.
Structured fields cover three categories of attributes — visual, auditory, and dialogue — explicitly listing: subject appearance, actions, motion, emotion, scene context, camera language, cinematic properties, lighting, on-screen text, dialogue, sound effects, and background music, 12 categories in total.

### Joint Audio-Video Caption Structure (Whether Both Visual and Auditory Tracks Are Covered Simultaneously, Whether Split into Separate Fields — e.g., LTX-2's Full Soundscape Description, Script-a-Video's Factorized Streams, Foley-Omni's Three Fields)

This is a typical "joint audio-video caption with dual-path decoupling" scheme: the caption covers both the visual track and the auditory track simultaneously (the auditory side includes dialogue, sound effects, and background music fields). To improve annotation fidelity and reduce cross-modal hallucination, a dual-path strategy is adopted to decouple the two modalities — visual attributes are inferred exclusively from video frames, and auditory attributes are inferred exclusively from the audio track; the two are combined into a structured description only after being inferred without cross-referencing each other. The paper states that this structured annotation scheme improves the quality and consistency of the multimodal representation, and makes annotation more reliable and generation more controllable. This shares the same underlying idea as LTX-2's full soundscape description and Foley-Omni's three fields, but Vidu S1's distinguishing feature is the addition of speech-aware temporal chunk alignment.

### Dialogue Transcription and Speaker Attribute Annotation (ASR Transcription + Speaker Identity/Language/Accent/Emotion) ⚠️

There is a fairly complete speaker-attribute annotation pipeline: the speech component is extracted from the raw audio → VAD (voice activity detection) + ASD (active speaker detection) label the timestamp of each speech segment and its associated speaker → based on the correspondence between the speaker and the on-screen subject, each speech segment is labeled as onscreen / offscreen / overlap. The caption includes a dialogue field. However, the paper does not clearly state whether full ASR text transcription is performed, nor does it annotate the speaker's language, accent, age, or other attributes [partially uncertain]. The Vidu-StreamBench evaluation benchmark mentions coverage of diverse speaker attributes and emotions.

### Geometric and Structured Annotation (Camera Parameters, Depth, 3D Point Tracks, Action Annotation, Explicit State) ⚠️

There is no explicit geometric annotation. The caption includes semi-structured text tags such as "camera language," "cinematic properties," and "lighting," and the omni model additionally outputs semantic labels such as shot and scene, but the paper makes no mention of camera parameter estimation, depth maps, 3D point tracks, pose/skeleton keypoints, or other geometric or explicit-state annotation [uncertain].

### Synthetic Data Construction (Constructing Training Pairs via Controlled Perturbation/Editing, e.g., InstructAV2AV) ⚠️

[Uncertain]. The paper makes no mention of using synthetic data or constructing training pairs via controlled perturbation/editing. The "artificial supervision" on the training side manifests in the training strategy rather than in data construction: Stage 2 uses a mixed Teacher Forcing and Diffusion Forcing strategy for causal adaptation, and Stage 3 uses Distribution Matching Distillation (DMD) with Phased Consistency Models (PCM) regularization for few-step distillation, with the bidirectional teacher model providing distribution-matching supervision to the student.

### Degree of Human Involvement (Human Annotation, Human Quality Inspection, Model Pre-Screening + Human Review) ⚠️

The data side is almost entirely automated: filtering and annotation are all performed by expert models + the omni large model + heuristic rules (such as the speech-energy-proportion rule); the paper makes no mention of any human annotation or human review step [degree of human involvement on the data side uncertain]. Human involvement clearly appears on the evaluation side: human preference evaluation is conducted on the self-built Vidu-StreamBench (500 samples), with paired A/B tests against three commercial systems — HeyGen, LemonSlice, and Kling Avatar 2.0 — across evaluation dimensions including Overall, Motion Dynamics, Identity Consistency, Audio-Video Sync, and Subject Controllability.

## Audio-Video Alignment

### Audio-Video Synchronization Detection Method (Lip Sync, Event Alignment)

Audio-video synchronization is addressed at two points in the data pipeline:
(1) Coarse screening at the pre-filtering stage: before entering the pipeline, raw videos deemed unsynchronized are removed based on the "audio-visual synchronization" metric, which stands alongside frame rate, resolution, and audio-visual integrity as one of four technical thresholds;
(2) Fine-grained alignment at the diarization stage: the paper emphasizes that "only when the model observes, during training, visual performance consistent with the corresponding speech can it learn high-fidelity lip sync"; VAD + ASD are therefore used to locate the timestamp and speaker of each speech segment, and the correspondence between the speaker and the on-screen subject determines onscreen/offscreen/overlap classification — offscreen (voice-over, where the audio subject does not match the visual subject) and overlap (multiple overlapping voices) are both treated as breaking the audio-visual correspondence, with clips containing overlap discarded entirely. In addition, during shot segmentation, cut points avoid the middle of speech, which likewise protects the integrity of speech-lip-sync segments.
On the evaluation side, the Sync-D metric (based on a SyncNet/Wav2Lip lip-sync expert, citing reference [59], Prajwal et al., "A Lip Sync Expert Is All You Need") is used to measure audio-video synchronization; Vidu S1 achieves 7.8470 on HDTF (lower is better), outperforming OmniAvatar (9.242), StableAvatar-1.3B (11.18), Hallo3 (8.660), Wan2.2-S2V-14B (8.255), LiveAvatar (8.447), LemonSlice (7.921), HeyGen (8.037), and Kling Avatar 2.0 (8.158).

### Specific Synchronization Detection Metrics and Thresholds (SyncNet/Synchformer/LSE/In-House, Threshold Values — e.g., MOVA's LSE-D≤9.5 and LSE-C≥4.5, SkyReels-V4's SyncNet |offset|≤3 ∧ conf>1.5) ⚠️

Data filtering stage: the paper only states that pre-filtering is performed based on "audio-visual synchronization," without disclosing the synchronization detection model used (whether SyncNet or not) or any threshold values [uncertain]. The "speech energy proportion too low" heuristic rule likewise gives no specific threshold [uncertain].
Evaluation stage: Sync-D (a SyncNet-family lip-sync expert distance metric) is explicitly used; Vidu S1 = 7.8470 (HDTF, best in class); the same report also reports CSIM = 0.9192 (identity preservation, best) and DOVER = 0.5660 (perceptual quality, best).

### Separate Handling of Temporal Synchronization vs. Semantic Synchronization (Time Alignment and Content-Semantic Matching as Two Independent Filtering Conditions)

There is a de facto separation, though the paper does not use this terminology: temporal synchronization is jointly ensured by the pre-filtering audio-visual synchronization metric, VAD speech-segment timestamps, and cut points avoiding the middle of speech; semantic/identity-level audio-visual matching is ensured by ASD active-speaker detection and onscreen/offscreen classification (determining "whether this voice was produced by this person in frame"). These constitute two independent filtering conditions in the pipeline. In addition, the caption's dual-path strategy (visual attributes inferred only from frames, auditory attributes inferred only from the audio track) is likewise a deliberate decoupling of the two modalities at the semantic annotation level, to avoid cross-modal hallucination.

### Audio Quality Filtering (SNR, Silence Detection and Silence-Proportion Thresholds, Removal of No-Audio-Track Samples, Removal of Off-Screen Audio Sources, Background Music Separation) ⚠️

(1) Samples with no audio track or incomplete audio-video are removed at the pre-filtering stage via the "audio-visual integrity" check;
(2) Removal of off-screen audio sources: speech segments judged by ASD as offscreen (speaker not in frame) are flagged separately and handled differently; overlap (multiple overlapping voices) clips are discarded in their entirety;
(3) Removal of background music/singing: it was observed that the diarization model is unstable in singing or heavy-background-music scenarios (vocals get misclassified into the music stem, producing synthetic timbre and distortion artifacts), so a heuristic rule was introduced to discard clips where "the speaker is vocalizing but the speech energy proportion is too low"; this in effect also serves as a backstop for the quality of vocal/background-music separation;
(4) Speech component extraction: the speech component is explicitly extracted from the raw audio prior to training.
No specific values are given for SNR thresholds, silence-proportion thresholds, or similar [uncertain].

### Classification and Separate Handling Strategy for Speech/Sound Effects/Music ⚠️

Speech is the only audio type treated as the core of both training and control: the speech component is explicitly extracted, VAD/ASD/speaker-attribution classification is performed, and speech serves as the real-time control signal for streaming generation. Music and singing are treated as noise sources, systematically removed via the speech-energy-proportion heuristic rule. Sound effects and background music are not specifically modeled, but are retained as independent description fields (sound effects, background music) in the structured caption. On the product side, users can select different voice tones, suggesting that TTS/voice-tone control is integrated on the audio side, but the paper does not elaborate on this [uncertain].

## Training Integration

### Multi-Stage Training Curriculum and Data-Curriculum Scheduling (Basis for Stage Division: Resolution/Duration/Quality Score/Modality; Low-Res→High-Res, Image→Video, Short→Long) ⚠️

Training is divided into three stages, but the basis for stage division is the generation paradigm and inference efficiency, rather than a data curriculum (no low-res→high-res or short→long curriculum organized by resolution/duration/quality score is observed):
Stage 1, Bidirectional Teacher Training: a bidirectional teacher model is trained on complete video-audio sequences, conditioned on the full sequence c^{1:N}, denoising the joint latent state x_0^{1:N}, serving as the foundation of the entire framework;
Stage 2, Causal Adaptation: an autoregressive model is initialized from the pretrained bidirectional model and adapted to the causal generation setting using a mixed training strategy combining unified Teacher Forcing and Diffusion Forcing, giving the model stable multi-step autoregressive generation capability;
Stage 3, DMD Distillation: Distribution Matching Distillation is applied together with Phased Consistency Models (PCM) regularization, distilling the autoregressive model into a few-step generator, substantially improving inference efficiency while maintaining generation quality.
Whether the data used differs across stages is not specified [uncertain].

### Changes in Data Mixture across Training Stages (Pretraining/Annealing/High-Quality SFT Subset) ⚠️

[Uncertain]. The paper does not disclose any changes in data mixture across the three training stages, nor does it mention an annealing stage or a high-quality subset. On the data side, only a single unified cleaning pipeline is described, without explaining how its output is divided across stages.

### Post-Training Data (Scale and Selection Criteria of the SFT Curated Set, Number and Annotation Method of Preference Pairs, Reward-Model Training Data) ⚠️

[Uncertain/essentially none]. The paper makes no mention of any SFT curated set, preference pairs (DPO), RLHF, or reward-model training data. The Stage-3 DMD+PCM distillation is a model compression/acceleration technique rather than preference-based post-training; its supervision comes from the distribution of the Stage-1 bidirectional teacher model rather than from human preference data. Human preference is used only for evaluation (the Vidu-StreamBench 500-sample A/B test) and is not fed back into training.

### Data Processing Infrastructure and Throughput (NeMo Curator/Data-Juicer/In-House, GPU Acceleration Ratio, Processing Scale, Cost) ⚠️

[Uncertain]. The paper discloses nothing about the infrastructure or framework used on the data-processing side (no mention of NeMo Curator / Data-Juicer / an in-house system), GPU acceleration ratio, processing scale, or cost. The report's engineering discussion is devoted entirely to the inference and serving side: the in-house TurboDiffusion (model acceleration) and TurboServe (efficient streaming scheduling), which use Ulysses-style context parallelism for multi-GPU sharding, operator fusion to reduce host-launch and synchronization overhead, RoPE Repositioning to avoid redundant computation of historical features, and TwinCache (a stage-aware dual cache combining a noise cache and a clean cache) to improve streaming inference efficiency and temporal stability; measured performance reaches 540p at 42 FPS on an RTX 5090, exceeding the 30 FPS real-time playback threshold, and the model can run on consumer-grade GPUs.

## Performance Comparison

### Quantitative Impact of Data-Strategy Ablations (Distinguishing: Filtering-Strictness Ablation / Caption-Density-and-Style Ablation / Data-Mixture Ablation, and Corresponding Evaluation Metrics) ⚠️

[Uncertain]. The paper conducts no data-side ablation experiments whatsoever — no filtering-strictness ablation, no caption density/style ablation, no data-mixture ablation. All experiments are end-to-end method comparisons (Vidu-StreamBench human-preference A/B tests + CSIM/Sync-D/DOVER comparisons on HDTF) and inference-efficiency validation. Every data-processing design element (omni-model-assisted filtering, dual-granularity captioning, dual-path decoupled annotation, overlap removal, speech-energy rule) is given only qualitative motivation, with no quantitative evidence of benefit.

### Evidence for Quality vs. Quantity (Cases Where Small, Curated Data Outperforms Large, Noisy Data)

There is no direct quantitative evidence, but the overall design philosophy clearly prioritizes quality and distributional purity over quantity: data scale is never mentioned at all, while the filtering conditions are extremely strict (single person, single shot, static or slow-motion shots, must display clear action, no watermarks/subtitles/ads, no overlapping voices, no clips with low speech-energy proportion, 3–60 seconds), resulting in a highly vertical final data distribution. The paper's qualitative claim is that "training data quality substantially affects training performance and model generalization capability," and it emphasizes that strict filtering rules are the means of ensuring training data quality. Indirect corroborating evidence: as a 540p real-time model, Vidu S1 comprehensively outperforms offline open-source models such as Wan2.2-S2V-14B, OmniAvatar, Hallo3, and StableAvatar-1.3B, as well as commercial systems such as HeyGen, LemonSlice, and Kling Avatar 2.0, on all three metrics — CSIM (0.9192), Sync-D (7.8470), and DOVER (0.5660) — on HDTF.

### Alignment between Training-Data Domain Distribution and the Evaluation Benchmark's Taxonomy (e.g., VABench's Seven Major Categories) ⚠️

There is a relatively clear alignment: the self-built benchmark Vidu-StreamBench contains 500 samples, each consisting of a triplet of "action instruction + reference first frame + audio clip," covering diverse action instructions, reference-image styles, speaker attributes, emotions, and application scenarios — this corresponds point-for-point with the training data's filtering dimensions (interactivity/clear action, subject, emotion, multiple visual styles, speaker voice). The five dimensions of human evaluation (Overall, Motion Dynamics, Identity Consistency, Audio-Video Sync, Subject Controllability) also respectively correspond to the training data's interactivity filtering, single-subject filtering + shot stability, diarization-based audio-visual correspondence, and the controllability afforded by speech-aware chunk captions. The nine quality dimensions labeled by the omni model (editing/subject/action/emotion/face/speech/scene/shot/tone) can be regarded as the taxonomy on the data side, but the paper provides no explicit mapping table or per-category proportions linking it to the benchmark's taxonomy [partially uncertain]. On the public-benchmark side, only HDTF (a standard audio-driven avatar benchmark) is used; the paper notes that public benchmarks such as HDTF cannot adequately evaluate instruction-following and natural motion in real-time interaction, which is precisely the motivation for building Vidu-StreamBench in-house.

## Uncertain Fields

The following fields have research information that is partially uncertain (sources marked with ⚠️):

- data_scale
- provenance_licensing
- resolution_aspect_distribution
- domain_distribution
- audio_category_distribution
- narrative_structure_distribution
- language_accent_distribution
- funnel_retention_rate
- shot_segmentation
- quality_filtering
- deduplication
- motion_filtering
- model_as_data_judge
- safety_filtering
- caption_model
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
- benchmark_taxonomy_alignment
