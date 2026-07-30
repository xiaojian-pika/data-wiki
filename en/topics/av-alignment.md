# Cross-model comparison: Audio-video alignment

> Topic: Data processing for video generation models (including joint audio-video generation): data cleaning pipeline, data distribution, annotation methods

[← Back to home](../index.md)

This page compares all entries side-by-side by field. ⚠️ indicates that some information in this field for the entry is uncertain.

**Fields**: [Audio-video sync detection methods (lip sync, event alignment)](#audio-video-sync-detection-methods-lip-sync-event-alignment) · [Specific sync detection metrics and thresholds (SyncNet/Synchformer/LSE/in-house, threshold values, e.g. MOVA LSE-D≤9.5 and LSE-C≥4.5, SkyReels-V4 SyncNet |offset|≤3∧conf>1.5)](#specific-sync-detection-metrics-and-thresholds-syncnetsynchformerlsein-house-threshold-values-eg-mova-lse-d≤95-and-lse-c≥45-skyreels-v4-syncnet-offset≤3∧conf15) · [Separate handling of temporal sync vs. semantic sync (temporal alignment and content-semantic matching as two independent filtering conditions)](#separate-handling-of-temporal-sync-vs-semantic-sync-temporal-alignment-and-content-semantic-matching-as-two-independent-filtering-conditions) · [Audio quality filtering (SNR, silence detection and silence-ratio threshold, removal of tracks without audio, removal of off-screen audio sources, background music separation)](#audio-quality-filtering-snr-silence-detection-and-silence-ratio-thresholds-no-audio-track-removal-off-screen-source-removal-background-music-separation) · [Classification of speech/sound effects/music and separate handling strategies](#classification-and-separate-handling-strategies-for-speechsound-effectsmusic)

## Audio-video sync detection methods (lip sync, event alignment)

`av_sync_detection` · Detail level: detailed

### [Allegro](../models/Allegro.md)

Not applicable. There is no audio modality, so there is no audio-video sync detection step (lip sync or event alignment) at all.

### [Apollo](../models/Apollo.md) ⚠️

Audio-video consistency detection is the most clearly disclosed method in Apollo's data filtering pipeline, using a **dual-tool, dual-dimension** design (placed after audio filtering and before data routing):
【Temporal alignment】Synchformer is used to detect the temporal synchrony of audio and video. Synchformer is a Transformer-based sparse audio-video sync detection model that estimates the audio-visual time offset; it is the same tool adopted by works such as MMAudio and MOVA (and corresponds to the evaluation metrics DeSync / AV-A).
【Semantic alignment】ImageBind is used to detect the semantic match between audio and video. ImageBind embeds six modalities including image and audio into a unified space, and cross-modal cosine similarity is used to measure "whether what's in the frame is the same thing as what's heard."
【Quote from the paper】"We then assess audio–visual consistency, using Synchformer for temporal alignment and ImageBind for semantic alignment, ensuring high synchronization in both temporal and semantic dimensions."
【Lip-sync-specific note】The data filtering stage makes no mention of lip-sync-specific detectors such as SyncNet/LSE-D/LSE-C — a notable gap: one of Apollo's core selling points is solving "poor lip–speech alignment," yet its data-side alignment filtering uses only the general-purpose Synchformer + ImageBind, with no dedicated filtering for facial lip movement observed (compare with MOVA, which explicitly uses LSE-D ≤ 9.5 and LSE-C ≥ 4.5 to select a 2.5M lip-audio high-quality subset). SyncNet Confidence appears in Apollo only as an **evaluation metric** (in Table 3, Sync-conf improves from 5.024 to 6.787), not as a data filter. If accurate, this suggests Apollo's lip-sync capability is achieved mainly through architecture (Omni-Full Attention) and multi-task training rather than data-side lip-sync selection. [Uncertain] (whether lip sync is used for data filtering is not made explicit)

### [CineDance / CineDance-1M](../models/CineDance.md)

Audio-video sync in this work follows two tracks: on the data side it is computed in full as a metadata metric, and on the evaluation side it is an independent dimension of CineBench:
【Data curation side (Stage 1)】
  · SyncNet — used for lip-sync detection, measuring the temporal alignment between mouth movement and speech in talking shots;
  · ImageBind — used for global cross-modal semantic alignment, measuring the semantic-space match between visual content and audio content, covering non-speech scenarios (e.g., the correspondence between sound effects and events).
  Both are computed and stored as metadata, without being used as a hard filter.
【Evaluation side (the AV Sync dimension of CineBench)】
  · Sync-C / Sync-D (the confidence and offset-distance components of SyncNet);
  · IB-Score (ImageBind cross-modal similarity).
【Design orientation】The paper deliberately distinguishes "lip sync" (SyncNet, for frame-level temporal alignment between speech and mouth movement) from "global semantic alignment" (ImageBind, for overall audio-visual content matching), forming two complementary detection criteria.
【Limitation】Synchformer, AV-align, and other event-level sync detection methods are not used; no dedicated detection means for foley-event temporal alignment are seen.

### [CogVideoX](../models/CogVideoX.md) ⚠️

CogVideoX's video side has no audio-video alignment step (it does not use an audio track).
Alignment on the CogSound side is a modeling-level mechanism rather than a data-filtering-level detection:
· Block-wise Temporal Alignment Cross-attention: by learning the relationship between frame-level video features and audio features, it precisely connects video and audio features, ensuring consistency between audio and video at both the temporal and semantic levels.
· Rotary Position Embedding (RoPE): provides a unique identity for each position in the audio sequence and captures relative relationships, improving temporal consistency and stability in long-horizon tasks.
· Video understanding based on GLM-4V: first accurately identifies the semantics and emotion behind the video, then generates matching audio accordingly, which is a form of semantic-level alignment.
Whether the data side ever performed sync detection and filtering (lip sync, event alignment) is entirely undisclosed [Uncertain]. CogSound is explicitly positioned as sound-effect generation rather than dialogue dubbing, and no lip-sync-related capability or metric is seen.

### [Cosmos-Predict2.5](../models/Cosmos-Predict25.md)

Not applicable. The model does not process audio, so there is no lip-sync detection, event-level audio-visual alignment, or any audio-video sync step in the pipeline. In this work, everything related to "alignment" is in the visual/text domain: text-video alignment is evaluated by the Text Alignment dimension of the VideoAlign reward model (RL stage); multi-view consistency is evaluated by the TSE and CSE metrics (in the Cosmos-Transfer2.5 multi-view driving model, see the effect comparisons); and alignment between control signals and generated content is evaluated indirectly via downstream perception metrics such as lane-detection F1 and cuboid-detection LET-AP.

### [Data-Juicer 2.0](../models/Data-Juicer.md) ⚠️

[Uncertain] Data-Juicer does not provide a dedicated audio-video sync detection operator, which is its most critical capability gap in the AV generation data-processing chain.
【The operator closest to sync detection】video_active_speaker_detect_mapper — "detects the active speaker in a video by analyzing visual face trajectories together with the audio signal." The underlying principle of this operator (models of the TalkNet/Light-ASD active-speaker-detection type) does rely on the temporal correlation between lip movement and audio, so it implicitly involves a lip-sync judgment; but its output is an identity/time-span determination of "who is speaking," not a continuous score of "how many frames of offset" or "what sync confidence," usable for threshold-based filtering. Users cannot directly use it to do LSE-D/LSE-C-style sync-quality selection.
【Missing capabilities】
  · No SyncNet operator — cannot output the audio-visual offset and confidence, and therefore cannot reproduce SkyReels-V4's "|offset|≤3 and conf>1.5" filtering.
  · No Synchformer operator — cannot score general audio-video events (non-speech) for temporal alignment, and therefore cannot reproduce Foley-Omni's sync score ≥0.2 filtering.
  · No AV-HuBERT, Perceiver-type audio-video alignment representation operators.
  · No ImageBind operator — cannot perform audio-video cross-modal semantic-consistency scoring.
【Visual-text alignment does exist】video_frames_text_similarity_filter performs "video frame–text" cross-modal consistency filtering based on CLIP; this is DJ's only mature, battle-tested cross-modal alignment operator (the core criterion in the T2V case, with a threshold of 0.306337). But it handles the visual-text axis, not the visual-auditory axis.
【Conclusion】If Data-Juicer is used to build training data for joint audio-video generation, the sync-detection step must be extended manually. Fortunately, DJ's operator-registration mechanism (inheriting the Filter base class + YAML declaration + writing to stats fields) makes it low-engineering-cost to wrap SyncNet/Synchformer as custom operators, and one can directly reuse its Ray distributed execution, GPU adaptive allocation, batch-processing optimization, and other infrastructure — this is exactly the value of DJ as an "operator library" rather than a "fixed pipeline."

### [Foley-Omni](../models/Foley-Omni.md) ⚠️

Sync detection is half of the "alignment dimension" of the filtering stage, and the same technology stack runs consistently through cleaning, modeling, and evaluation:
【Cleaning stage】Synchformer is used to compute a sync score, with a threshold of ≥0.2, removing segments with "unreliable synchronization." Synchformer is a Transformer-based audio-video sync detection model; compared to classic SyncNet, its advantage is that it is not limited to lip sync and can handle temporal alignment for general audio-video events (e.g., an impact sound and an impact scene), which is necessary for this task where sound effects are an important generation target — if SyncNet were used, sound-effect data in non-face scenes would be mistakenly removed in large numbers.
【Modeling stage】Synchformer features are used directly as the model's temporal conditioning input, injected into the DiT via an additive path (denoted Z_sync). This means the "sync representation used for filtering" and the "sync condition used for generation" are the same thing, so the filtering criterion is naturally aligned with the model's capability — samples that pass the filter are exactly the samples from which the model can effectively exploit its sync signal. This design is worth noting among comparable works.
【Evaluation stage】The DeSync metric (lower is better) is used to measure the temporal alignment quality between generated audio and video. Foley-Omni achieves a DeSync of 0.16 on V2ST-Bench, very close to the ground-truth value of 0.14, and significantly better than the cascaded baselines MMAudio+CosyVoice 3+AudioX at 0.85 and MMAudio+LipVoicer+AudioX at 0.26.
【Ablation evidence】Table 6 shows that after removing the Z_sync sync additive pathway, IB drops from 0.26 to 0.22, V2ST WER rises from 7.59 to 12.40, and FD_VGG rises from 1.57 to 2.21 — all three metrics degrade simultaneously, confirming that the sync-feature pathway makes a substantive contribution to both audio-visual consistency and speech quality.
【Lip sync】[Uncertain] The paper does not deploy a dedicated lip-sync detector (such as SyncNet confidence) in the cleaning stage; the lip-sync quality of the VisualTTS branch relies on the dataset itself (GRID/LRS2 are naturally lip-reading datasets with guaranteed synchrony) and on Synchformer's general sync criterion. This may be less precise than a dedicated lip-sync metric in pure speech scenarios.

### [Goku](../models/Goku.md)

Not applicable. Goku has no audio modality, so there is no audio-video sync detection step in the data pipeline (no lip-sync detection, no audio-visual event alignment, no SyncNet/AV-align-type methods). The only concept related to "alignment" in its data pipeline is **visual temporal consistency**: adjacent-frame cosine similarity from DINOv2 (480p ≥0.85, 720p ≥0.90) is used to ensure visual coherence within a clip without jump cuts — this is a within-visual temporal-consistency constraint, methodologically distinct from cross-modal audio-video sync.

### [Hailuo / MiniMax Video](../models/Hailuo.md)

Not applicable. The model does not generate audio, so there is no audio-video sync detection step (lip sync, event alignment) in the training pipeline. Even if the training data retains the original audio track, there is no evidence that it was used for sync-based filtering.

### [HunyuanVideo-Foley](../models/HunyuanVideo-Foley.md)

Audio-video alignment detection is the step of the highest methodological value in this work's data pipeline, and it forms a complete closed loop between the data side and the model side:
【Data side: dual-tool, dual-dimension detection (Level 6)】
- ImageBind — handles semantic alignment. Audio and video are each encoded into ImageBind's shared multimodal embedding space, and their similarity is computed to determine "whether this sound is semantically the sound this scene should have." Typical mismatches it can identify include: the scene is chopping vegetables in a kitchen but the audio track is pop music, or the scene is an outdoor landscape but the audio track is an indoor narration. Such post-production score/narration samples are extremely common in web video and are the main source of contamination for V2A training data — ImageBind is precisely the remedy for this.
- AV-align — handles temporal alignment. It detects whether the onset of audio events and the change points of visual events correspond on the time axis. Typical mismatches it can identify include: semantically correct content with a systematic time delay (the audio track is offset by hundreds of milliseconds overall), or a mismatch between the density of sound events and the density of visual events.
【Corresponding mechanism on the model side】Aligned samples selected on the data side are learned via three mechanisms on the model side: (1) Synchformer frame-level sync features are injected via gated modulation, providing an explicit time-axis anchor; (2) interleaved RoPE encodes audio and visual tokens interleaved by time, so that cross-modal tokens at the same moment are positionally adjacent; (3) joint self-attention lets audio and video tokens attend directly to each other. Data filtering ensures "what is learned is correct," while the architectural design ensures "it can actually be learned" — a clear division of responsibilities between the two sides.
【Evaluation side】DeSync (the degree of desynchronization computed from Synchformer, lower is better) is used to measure temporal alignment, and ImageBind cosine similarity (IB, higher is better) is used to measure visual-semantic alignment. Results across three benchmarks: on Kling-Audio-Eval, DeSync 0.54 / IB 0.38 (MMAudio: 0.56 / 0.30); on VGGSound-Test, DeSync 0.53 / IB 0.36; on MovieGen-Audio-Bench, DeSync 0.74 / IB 0.35 (MMAudio: 0.80 / 0.27). The IB metric improves 26.7% relative to the baseline, one of the largest gains of any dimension.
【A note on a circularity risk】ImageBind is used both to filter data and to evaluate the IB metric, and Synchformer is used both for model conditioning injection and to evaluate the DeSync metric — in both cases, the overlap between "what is used in training" and "what is used in evaluation" will systematically inflate the corresponding metric's performance. The paper does not discuss this evaluation bias, and readers should discount accordingly when making cross-model comparisons.

### [HunyuanVideo](../models/HunyuanVideo.md) ⚠️

Not applicable. There is no audio modality, and the training data contains no audio track, so there is no audio-video sync detection step. [Uncertain]

### [InstructAV2AV](../models/InstructAV2AV.md) ⚠️

Audio-video sync in this work is handled as a problem spanning three distinct stages — data admission, enforcement during synthesis, and result evaluation — with a different mechanism used at each stage. This is the most layered treatment of sync in this pipeline.

【Stage 1: sync assurance at the data-admission stage (source control, not score-based filtering)】
  Rather than using a sync scorer such as SyncNet/Synchformer for threshold filtering, this work uses the stronger admission condition of "audio-visual attribution": TalkNet locates the active speaker and Scribe extracts speech timestamps, and only clips where "the speech is temporally aligned with a visible on-screen speaker" are retained. For the non-speech side, a single clear sound source is required (ambiguous sound sources are removed). This effectively admits only material that is inherently audio-visually co-sourced from the start, rather than filtering synced material out of a mixed pool — it is more thorough than score-threshold filtering because it excludes the structural problem of "the sound doesn't come from the scene at all," not merely a time offset.

【Stage 2: sync enforcement at the data-synthesis stage (guaranteed at generation time)】
  This is the most distinctive aspect of this work: sync is not filtered for, but enforced during generation. A mask-guided video-editing model injects the already-synthesized target audio features frame-by-frame into the video generation process via frame-wise cross-attention, described in the paper as "ensuring strict temporal synchronization." The synthesis order is audio first, then video, so that lip movement/motion follows the established audio timeline. Compared to "generate each independently, then filter out the unsynced ones," this is far more efficient — disqualified samples are simply never generated in the first place.

【Stage 3: automatic-verification-stage sync re-check】
  The fourth of Qwen3-Omni's five scoring dimensions is exactly "audio-video synchronization for cross-modal alignment," used as one of the mandatory pass conditions, performing a semantic-level sync re-check on the synthesized results. This is a judgment made by an MLLM rather than a dedicated sync model — the advantage is catching semantic-level mismatches (e.g., a cat's picture paired with a dog's bark), while the disadvantage is lower temporal precision than dedicated models such as SyncNet.

【Stage 4: sync measurement at the model-evaluation stage】
  Four audio-video metrics are used: AV-A (ImageBind-based audio-video semantic alignment), PEAVS (Perceptual Evaluation of Audio-Visual Synchrony, perceptual-level sync evaluation), Sync-C (SyncNet confidence, higher is better), and Sync-D (SyncNet distance, lower is better, configured with vshift=15 frames). InstructAV2AV achieves an AV-A of 27.72 on the InsAVE-80K evaluation set, better than AvED's 26.44, AVI-Edit's 26.37, and CoherentAVEdit's 22.67; on AvED-Bench, AV-A is 23.71 vs. AVI-Edit's 23.21.

【Methodological assessment】The four-layer design of "source admission + generation-time enforcement + MLLM re-check + evaluation measurement" is more complete than relying solely on a sync-score threshold, with clear responsibilities at each layer (structural problems are resolved at the admission layer, temporal precision is guaranteed at the generation layer, semantic matching is checked at the verification layer, and final capability is measured at the evaluation layer).
[Uncertain] No dedicated sync detection model (SyncNet/Synchformer/AV-HuBERT) is used for quantitative filtering anywhere in the data-construction pipeline, so no statistics on the sync-quality distribution of the dataset itself can be given; nor is there a data-engine ablation of "with vs. without frame-wise cross-attention" to quantify this mechanism's contribution to the sync quality of the synthesized data.

### [2026 Miscellaneous joint audio-video generation works](../models/JAVG_2026_misc.md) ⚠️

This batch of works falls into three tiers on sync detection, and a trend toward tool diversification is emerging:
【NAVA — three detectors used together (the most comprehensive in this batch)】
The data-filtering-stage audio-visual alignment operator is explicitly described as three parallel ones: "audio-visual alignment, measured by SyncNet, SyncFormer, and ImageBind" — SyncNet (lip-audio temporal sync, the classic lip-sync detector), SyncFormer/Synchformer (Transformer-based general audio-visual sync detection, capable of handling non-speech event alignment), and ImageBind (cross-modal semantic similarity, measuring whether audio and video content are "about the same thing"). These three exactly cover three different levels: SyncNet handles lip-audio timing, Synchformer handles general event timing, and ImageBind handles semantic matching — a well-layered alignment-detection system, far more complete than approaches using only SyncNet (e.g., Ovi, OmniCustom). None of the thresholds are disclosed [Uncertain], which is the biggest regret in NAVA's data disclosure.
【OmniCustom — single SyncNet with dual thresholds (thresholds fully disclosed)】
Follows Ovi's standard: "|offset|≤3" and "confidence >1.5." These values are identical to those in the Ovi paper (|offset|≤3 frames, confidence>1.5), showing that Ovi's thresholds have become the de facto reference standard in this field, directly inherited by later work. At 24fps, |offset|≤3 frames corresponds to a time tolerance of ≤125 milliseconds. This applies only to lip-audio sync for clips containing speech; there is no detection of non-speech event alignment [Uncertain].
【ALIVE — TS-TalkNet for active speaker detection (a different use case than pure sync filtering)】
Uses TS-TalkNet (Target-Speaker TalkNet) "to evaluate the active speaker," producing a face-voice correspondence matrix. Note its primary purpose is not "filtering out unsynced samples" but "determining who is speaking in a multi-person scene" — a harder task (active speaker detection rather than lip-sync verification). A secondary use of the sync score is selecting identity anchors (the 1.5-second sub-clip with the highest sync score). The threshold is undisclosed; it is only stated that the method "filters videos failing high-confidence thresholds" [threshold value uncertain].
In addition, ALIVE's audio-visual relevance judgment is handled instead by an MLLM (see model_as_data_judge), forming a division of labor with TS-TalkNet's speaker detection: the MLLM handles semantic relevance, TS-TalkNet handles speaker attribution.
【StreamChar / CCL / Baton — no data-side sync filtering】
None of the three describe data-side sync detection [Uncertain]; they rely on upstream datasets (SpeakerVid-5M / TalkVid / OpenHumanVid, which already have sync filtering built in) or on architectural guarantees. CCL reports Sync-C / Sync-D / DeSync as three sync metrics in evaluation, showing that it values sync but places the guarantee in the architecture (TARP, Temporally-Aligned RoPE) rather than the data side.
【ITS-JAVG — moves sync detection to inference time】
Uses JavisScore as a fine-grained audio-video sync verifier ("focuses on fine-grained audio-video synchronization"), along with AVHScore to measure the semantic consistency between audio events and visual events, and ImageBind to measure audio-video semantic similarity. A key finding is that guiding search with JavisScore alone causes other metrics to degrade (an asymmetric trade-off), and that search can exploit loopholes in the sync verifier. The implication for the data side is direct: filtering with a single SyncNet threshold alone will cause the retained data to systematically skew toward SyncNet's preferences (e.g., toward frontal faces, clear mouth movement), causing distributional distortion — NAVA's combined use of three detectors happens to mitigate exactly this problem. Author Joon Son Chung is one of the original authors of SyncNet, which lends particular authority to his judgment on the limitations of sync metrics.

### [Baseline collection for joint audio-video generation](../models/JavisDiT_baselines.md) ⚠️

Audio-video sync detection is the dimension with the greatest technical divergence in this collection; the five works represent five different approaches:
【MM-Diffusion (2022) — no sync detection, relying on architecture and data selection】The data side performs no sync filtering at all. Synchrony is ensured by two things: (1) dataset selection — Landscape's 9 scene categories all have strongly causally bound audio-visual pairing in nature, and AIST++ has dance bound to musical beat; (2) architecture — a random-shift-based attention block does cross-modal attention within a local temporal neighborhood. The evaluation side also has no dedicated sync metric, only FVD (video quality) + FAD (audio quality) + a manual Turing test. This is an early-stage example of "sync quality cannot be quantified, only judged by eye."
【AV-DiT (2024) — likewise no sync detection】Follows MM-Diffusion's data and evaluation setup.
【JavisDiT (2025) — in-house sync metric JavisScore, the most important methodological contribution in this collection】The authors point out that existing sync metrics are unreliable on real, complex content, and therefore propose JavisScore:
- Computation method: each audio-video pair is cut into multiple 2-second windows with 1.5-second overlap (i.e., a sliding-window stride of 0.5 seconds); ImageBind is used to compute audio-video sync for each segment; specifically, the similarity between all frames within a segment and that segment's audio is computed, and then the "worst-synced 40% of frames" are used for scoring (rather than the average) — this "take the worst 40%" design is key, because averaging would dilute local desync among a large number of well-aligned frames, whereas the perceived desync in reality is dominated precisely by the worst segments.
- Validity verification: a manually annotated evaluation dataset of 3,000 samples was built to verify that JavisScore tracks human judgment more closely than existing metrics.
- Sync mechanism on the training side: a HiST-Sypo spatiotemporal sync prior + contrastive learning with synthesized asynchronous negative samples to train the prior estimator. That is, JavisDiT does not perform data-filtering-style sync selection, but instead models synchrony as a learnable prior.
【JavisDiT++ (2026) — explicit frame-level sync + Synchformer as a reward】TA-RoPE (Temporal-Aligned RoPE) enforces frame-level alignment between audio tokens and video tokens at the positional-encoding level (sharing the same underlying idea as Ovi's scaled-RoPE); in the AV-DPO stage, Synchformer is used as one of the reward models for temporal sync, incorporating synchrony into the preference-optimization objective. DeSync is reported for evaluation.
【Harmony (2025) — a three-pronged approach】(1) At the training-mechanism level: Cross-Task Synergy uses bidirectional generation tasks (audio-driven video, video-driven audio) to suppress alignment drift during joint denoising; (2) at the architecture level: the GLDI module decouples global style alignment and local temporal precision into two branches; (3) at the inference level: Synchronization-Enhanced CFG amplifies the alignment signal — ablation shows this contributes the largest sync gain (Sync-C improves from 5.09 to 6.51). On the data side, an "audio-visual consistency scoring model" is used to filter speech data.
【UniAVGen (2025) — architecture-level asymmetric interaction】Relies on asymmetric cross-modal interaction + Face-Aware Modulation + modality-aware CFG to ensure sync; no sync filtering is described on the data side [Uncertain]. Evaluation uses the SyncNet metric.

### [Kling 3.0 / Kling Video 3.0 Omni](../models/Kling_30_Omni.md) ⚠️

[Uncertain] The audio-visual sync detection method used for Kling 3.0 Omni's training data is undisclosed. Kling-Omni reports that at the base level it mentions only "audio-visual corruption detection," and the third-level multimodal alignment is mainly aimed at caption-vision, reference-image-to-video, and character-identity consistency, without naming lip-sync detection. KlingAvatar 2.0 states it has an "extensive filtering pipeline to ensure high visual fidelity and consistent audio-lip synchronization," confirming the existence of a lip-sync-filtering step but not disclosing the method. On the Kling-Foley side, the model architecture builds in a frame-level "audio-visual sync module" and a visual-semantic representation module for conditional alignment, and the evaluation distinguishes semantic alignment from temporal alignment as two metric categories — indicating the team has a systematic capability to measure "event-level audio-visual alignment." Overall assessment: Kling 3.0 Omni's data-side sync filtering likely includes at least a lip-sync score (for speech content) and an event-level temporal-alignment score (for sound-effect content), but neither has public details.

### [LTX-2](../models/LTX-2.md) ⚠️

This is the biggest gap in LTX-2's data disclosure, and also the step most relevant to — yet most lacking for — the topic of this survey. The technical report describes no training-data-side audio-video sync detection or asynchronous-sample-removal mechanism whatsoever — no lip-sync detector, no event-alignment detection, no asynchrony-removal threshold.
The source of its sync capability is attributed to three points, all on the data-selection and architecture side rather than the sync-detection side:
(1) Data side: only clips with "a significant and information-rich audio component" are retained — i.e., real videos with their natively synchronized audio track are kept, which naturally avoids asynchronous samples such as dubbing or post-hoc added audio (but the paper does not state whether off-screen narration, post-production scoring, and other non-synchronous audio are explicitly removed);
(2) Annotation side: captions accurately transcribe dialogue and label speakers, so text, speech, and lip movement can be aligned semantically;
(3) Architecture side: bidirectional audio-video cross-attention running through the full depth, where the cross-modal attention uses only 1D temporal RoPE, forcing attention to focus on the temporal-sync dimension; the paper claims this can map visual events (e.g., an object impact) to auditory events (the corresponding foley sound) with "sub-frame precision"; cross-modality AdaLN further modulates the strength of cross-modal information injection; at inference time, increasing the cross-modal guidance scale s_m improves temporal sync.
Paper Fig. 3 uses visualizations of AV cross-attention maps as qualitative evidence of sync capability: the model can spatially track a moving vehicle, switch attention dynamically between two speakers, and focus on the lip region during close-up speech. [Uncertain]

### [LongCat-Video](../models/LongCat-Video.md)

Not applicable to the base version (no audio).
For Avatar 1.5, lip-sync verification is treated as both an independent step in the offline annotation stage and the first checkpoint in online filtering: offline, lip-sync verification is performed, removing "samples with large audio-visual offsets"; in the online clip-level verification, "audio synchronization" is the first tier of the progressive filtering chain. The accompanying active-speaker detection jointly uses two models, TalkNet and UniTalk, to ensure the audio track belongs to the target person on screen rather than being off-screen audio; in multi-person scenes, YOLOv6 + ByteTrack are used to track and associate identities, and concurrent multi-person speech segments are removed. There is also a dedicated silence-data branch (Qwen3-Omni + Qwen3-VL, with both models agreeing the subject is not speaking) to cover the legitimate case of "audio present but no lip movement." Event-level (non-lip) audio-video alignment detection is not addressed.

### [MOVA](../models/MOVA.md)

MOVA treats audio-video alignment as a first-class citizen of data filtering, using different tools at different stages targeting different granularities:
【Stage 2 general alignment filtering (dual tools)】
- Temporal sync: SynchFormer computes the audio-video temporal-sync value (DeSync offset) for every video.
- Semantic alignment: ImageBind computes the audio-video cross-modal semantic-alignment score (IB-Score).
【Phase 2 lip-sync-specific filtering】SyncNet-family LSE-D (Lip Sync Error - Distance) and LSE-C (Lip Sync Error - Confidence) are used to select high-quality lip-audio-corresponding clips.
【Audio-type routing】The EAT audio-classification model distinguishes speech from non-speech, building separate subsets for the two capabilities of "lip sync" and "general foley/ambient-sound modeling" — i.e., different capabilities correspond to different sync criteria.
【Architecture-level sync guarantee (not data-side but strongly related)】Aligned RoPE maps video and audio latents onto the same temporal grid; the Bridge bidirectional cross-attention provides layer-by-layer cross-modal interaction; Dual Sigma Shift lets the two modalities sample noise levels independently. In Discussion 7.1 the paper offers an insightful point: the predefined sigma schedule actually acts as an **implicit sync-direction prior** — for close-up shots of the mouth (where the target region occupies a large fraction of the frame), the visual latent carries relatively rich information, and the process tends toward Video→Audio; when the speaker occupies only a small part of the frame, visual evidence is relatively uncertain, and the process naturally shifts toward Audio→Video, with speech providing a more reliable temporal anchor.
【The paper's core takeaway】"architectural mechanisms alone (e.g., Bridge modules for cross-modal attention) are insufficient to achieve high-quality lip synchronization—the model must also learn phoneme-to-viseme mappings from data, which requires larger capacity and more training examples." That is, architectural mechanisms alone are insufficient for high-quality lip sync; the phoneme-to-viseme mapping must be learned from data, requiring larger capacity and more training examples. This is MOVA's most direct statement of the thesis that "data determines the ceiling of audio-video sync."

### [Merged entry: ① Mochi 1](../models/Mochi_MAGI_Motif.md)

Not applicable. All three models are silent video generators, and their training data contains no audio track, so there is no lip-sync or audio-visual event-alignment detection step at all.

### [Movie Gen](../models/Movie_Gen.md)

No lip-sync detection is performed (the model does not generate speech, so lip-sync metrics of the SyncNet/LSE-C type do not apply at all). The core of audio-video correlation detection is the CAVTP (contrastive audio-video-text pre-training) model: it computes the cosine similarity between the audio embedding and the video embedding as a proxy score for "how likely this audio is diegetic (in-scene)" — because CAVTP is trained primarily on data dominated by diegetic sound, in-scene audio that matches the on-screen content has a closer audio-video embedding distance. This score, combined with the AED type label, forms the basis for bucketing pretraining data (Table 24).
The paper also divides audio-video correspondence into three difficulty tiers from a modeling perspective: ① on-screen diegetic sound — an extremely strong audio-visual correspondence where exactly what sound occurs at what moment is deterministic, requiring strong video understanding and dense action recognition (e.g., a golf-club swing); ② diegetic off-screen sound — requires understanding what sounds occur in what environment, and the logical order between events (e.g., birdsong in a forest, a crowd cheering after a difficult move is completed), requiring stronger reasoning; ③ non-diegetic/off-screen sound — relevant only at the semantic level (background music must match the mood, a riser is used to build tension), requiring modeling of human emotion beyond world physics.
On the evaluation side, "diegetic sound synchronization" is rated by humans, paired with the objective ImageBind score metric. The limitations section admits that when action is dense (tap dancing), the target is small or occluded (footsteps), or fine-grained visual understanding is required (identifying guitar chords), the generated audio can be out of sync.

### [NVIDIA NeMo Curator](../models/NeMo_Curator.md)

None. The framework has no audio-video sync detection capability whatsoever: the video pipeline does not parse the audio track, so there is no stage for lip-sync detection, event-alignment detection, or asynchronous-sample removal; the audio pipeline processes standalone audio files with no corresponding video frames available for alignment. None of SyncNet, Synchformer, AV-align, or any other sync-detection component appears in the official documentation, the Cosmos WFM paper, or the NeMo video foundation-model training paper.
This constitutes NeMo Curator's primary gap when serving joint audio-video generation models — although it is the de facto standard for "large-scale visual data processing," the core step of AV data construction (sync-based filtering) has to be built entirely from scratch.

### [OmniHuman dataset + OHBench](../models/OmniHuman.md)

In OmniHuman, audio-visual sync is given a role beyond the ordinary — it is not merely a quality-filtering condition but also the generating mechanism for the structured annotation of "speaker attribution," one of the most valuable designs in this work:
【Conventional use: sync-quality filtering】SyncNet is used to judge lip-audio alignment; samples that fail are removed.
【Distinctive use: audio-visual attribution】In multi-person scenes, knowing merely that "the audio and video are in sync" is not enough — one must also know "who said this." OmniHuman's solution is a complete attribution chain:
1) 3D-Speaker performs speaker diarization, yielding M active speech segments with speaker indices;
2) YOLOv11 + MOTRv2 yields visual-ID tracks for every person in the scene;
3) A SyncNet response is computed for every combination of audio segment and visual track;
4) Greedy matching: each audio segment is assigned to the visual-ID track with the highest response;
5) ArcFace face embeddings confirm identity consistency (similarity > 0.55).
That is, SyncNet is upgraded from a "scorer" to a "matcher" — using the relative magnitude of the sync response to resolve attribution, which carries far more information than a simple threshold. This is also the technical basis on which OmniHuman is able to provide speaker annotation and support two-person dialogue data.
【Retention condition】A sample is retained only if "all detected subjects" have an S_sync above the preset threshold and a time offset within ±3 frames. Note the wording "all detected subjects" — this is a pass-for-all rule rather than a pass-for-majority rule, which is a fairly strict constraint for two-person and multi-person scenes, and also explains why the dataset covers only single-person, two-person, and person-object categories and not scenes with more people (the more people involved, the lower the probability that all of them pass).
【The meaning of ±3 frames】At a unified 30 FPS, ±3 frames = ±100 milliseconds. This tolerance roughly matches the human perceptual threshold for audio-visual desync (generally believed to be imperceptible within audio lag of up to 100ms or audio lead of up to 50ms), a reasonable engineering choice.
【Byproduct: lip-sync quality retained as annotation】Frame-level annotations include a lip-sync quality assessment — that is, sync quality is not only used for filtering but is also released as a queryable field alongside the data, so downstream users can filter for a stricter subset if desired.
【Event-level alignment: not addressed】The data domain is human-centered speech content, with no event-alignment detection for foley sound effects (no AV-align, no Synchformer). But on the evaluation side, ImageBind is used to measure the temporal offset of audio-visual events (the V-A metric), which to some extent covers non-speech audio-visual correspondence.
【Sync metrics on the evaluation side】At the individual level, OHBench uses SyncNet to measure lip-audio alignment accuracy; at the global level, it uses ImageBind to measure the temporal offset of audio-visual events. The two divide labor between "lip-level" and "event-level," complementary in granularity.

### [Open-Sora series](../models/Open-Sora.md)

Not applicable. There is no audio modality, so there is no audio-video sync detection step.

### [Ovi](../models/Ovi.md) ⚠️

Ovi treats audio-video sync filtering as the highest-priority step in the entire data pipeline, and it is the only filter the authors describe with the word "strict."
【Method】The classic SyncNet (Chung & Zisserman, 2016) is used — a ConvNet-based model that learns a joint embedding between sound and mouth images, outputting two scalars: confidence (measuring the strength of audio-visual correlation) and offset (measuring how many frames the audio track leads/lags the picture). The authors explicitly state that they engineered the model to handle video data "on the scale of millions" — i.e., they transformed SyncNet from a research-grade script into a large-scale batch-processing component.
【Decision logic】Only clips satisfying both |offset| ≤ 3 (frames) and confidence > 1.5 are retained, together with the additional condition of average audio volume ≥ −60 dB.
【Scope of application】Mainly targets "speech videos" (clips containing speech), used to remove lip-audio-unsynced data; for non-speech clips, no dedicated detector is used for event-level audio-visual alignment (e.g., an impact sound and an impact scene), and no AV-align-type event-alignment metric is used either [Uncertain].
【Design rationale (an important lesson)】The paper states explicitly: "We have experimentally determined that even a small quantity of out-of-sync data can impede lip-sync abilities and chose these strict criteria to minimize the risk of misaligned data." That is, experiments showed that even a small amount of unsynced data is enough to harm the model's lip-sync ability, so the authors chose to sacrifice data volume in favor of a strict threshold to minimize the risk of misaligned data. This is the paper's most direct statement anywhere on "data quality over quantity."
【No sync loss during training】After strict data-side filtering, the training stage uses no explicit sync loss, no auxiliary sync module, and no face mask; sync emerges entirely from paired sampling + shared timestep + bidirectional cross-modal attention + scaled RoPE. In a sense, Ovi bets almost the entire guarantee of synchrony on data filtering.

### [Script-a-Video](../models/Script-a-Video.md) ⚠️

The data side and the evaluation side must be distinguished, as they differ significantly:
【Data side (annotation stage): schema rules in place of a detection model】This is the most notable idea in this work. MTSS does not use a sync-detection model such as SyncNet to filter data; instead it encodes audio-video correspondence directly into the annotation rules:
1) Strict audio-visual coupling admission principle — the Event stream records only audio events that have "a direct visual counterpart or thematic relevance"; a sound effect must be produced by a subject visible in the frame. Sounds that don't satisfy this are not discarded but downgraded to global_audio. This achieves, at the annotation level, the separation of "sounds with visual grounding" from "sounds without visual grounding."
2) Structural speaker binding — an Event's speaker field points to a Reference ID, pointing to the same entity library as a Shot's references_in_shot, turning "is the speaker in the frame" into a question directly answerable via reference relationships, with no need for face detection + lip-movement analysis.
3) Three-tier timestamp system — shot-level time_range, event-level time_range, and micro-level timestamps within descriptions all share a global time axis, so event alignment is directly expressed via timestamps; the paper states the goal is to achieve sub-frame audio-visual coordination and "surgical synchronization."
That is: alignment is not a "detected filtering condition" but "annotation content written directly into the structure." This is fundamentally different methodologically from routes such as UniVerse-1 (a hard SyncNet conf>2.0 threshold filter) and MOVA that use a discriminative model as the gate.
【Evaluation side: SyncNet is used】The lip-sync accuracy of generation results is evaluated with SyncNet, which outputs a sync-confidence score and a time offset in frames. In addition, Shot Boundary Deviation measures the frame-level absolute deviation between generated shot boundaries and the boundaries specified in the script.
【An important finding on sync-metric reliability in the paper】See the human_in_loop and sync_metric_and_threshold fields for details — SyncNet scores are inflated when audio information is sparse (flat ambient noise), giving falsely high apparent synchrony — a pitfall warning of general reference value.
【Whether the data side has additional sync filtering】The paper does not disclose whether the 500K dataset or the four generation-side datasets underwent any audio-visual sync-detection filtering; this is part of the overall absence of the cleaning pipeline description. [Uncertain]

### [Seedance 2.0 and Seedance 1.5 pro](../models/Seedance_20_Seedance_15_pro.md) ⚠️

[Uncertain] The data-side audio-visual sync detection method is not disclosed in the report. Seedance 1.5 pro states only, at a principled level, that its data pipeline "prioritizes video-audio coherence"; a third-party summary states that the second stage is "automated filtering for audio-visual sync and clip quality," but this wording does not appear in the original paper and is an unverified inference. The evaluation side, however, has a complete definition: SeedVideoBench 1.5's "audio-video synchronization" metric measures the temporal alignment between the auditory and visual streams, evaluating speech-lip sync (mitigating the perceived "ventriloquism effect"), the alignment between sound effects and visual events, and whether salient on-screen actions have corresponding auditory cues; SeedVideoBench 2.0 carries this forward and scores it across 17 fine-grained categories (Seedance 2.0's T2V audio-video-sync overall score is 3.75, ranking first in 16 of the 17 categories, with a 93.30% usability rate and a 68.30% satisfaction rate, far ahead of the best competitor's 25.45%).

### [SkyReels series](../models/SkyReels.md)

Only SkyReels-V4 is relevant, and it is the only cross-modal filtering step in its data pipeline, with a clearly stated method:
【Method】SyncNet is used as a lip-audio sync discriminator — the paper describes it as "using a ConvNet architecture to learn joint embeddings between sound and mouth images," matching via a sliding window over embedding distance along the time axis, outputting two quantities: the audio-video time offset and a confidence value, used to remove samples with post-hoc dubbing, audio-visual misalignment, or mismatched lip movement. The paper mentions this SyncNet pipeline was adapted to run at the scale of "millions of video samples" via distributed processing.
【Position in the pipeline】Sync filtering occurs after video-branch filtering and rebalancing, before entering joint audio-video training, serving as a cross-modal admission gate.
【Limitations】(1) Only lip-audio sync (speaker scenes) is covered; no event-level alignment detection for non-speech events (impact–sound effect, footstep–visual) is seen (e.g., AV-align-type metrics), and how temporal alignment is ensured for sound-effect samples is not explained; (2) it is not explained how clips with no face/no speaker are handled (such samples cannot have a SyncNet score computed); (3) it is not stated whether off-screen narration, voice-over, or post-production scoring — non-synchronous audio — is removed. Architecture-side sync assurance is handled by layer-by-layer bidirectional audio-video cross-attention and RoPE temporal-dimension scaled alignment.

### [Sora 2](../models/Sora_2.md) ⚠️

The training-data-side audio-video sync detection method is completely undisclosed — this is the most central concern of this survey, and it is exactly where OpenAI discloses nothing. The System Card contains no statement whatsoever about lip-sync detection, event-alignment detection, or asynchronous-sample removal. On the capability side, it is only claimed that audio is "properly synchronized with on-screen action, including accurate lip-sync for speaking characters," along with sound effects bound to on-screen events and music tempo matching scene rhythm. It can be assumed that some form of AV sync quality control must exist in the training pipeline (otherwise lip sync could not be learned), but the method, module, and criteria are all unknown. [Uncertain]

### [SpeakerVid-5M](../models/SpeakerVid-5M.md)

Audio-video sync detection is the hub of the SpeakerVid-5M pipeline, and its role goes beyond the usual scope of "quality filtering" — it takes on three structural functions:
【Function 1: audio-visual identity binding (the most central)】After human detection, SyncNet is used to compute the lip-audio overlap between every candidate human bounding box and the audio track, and based on the confidence score, the speaker ID on the audio side is bound to the bounding box on the visual side. This step answers "which person on screen produced this sound," equivalent to active speaker detection, and is the prerequisite for constructing the entire dataset — without it, the voiceprint IDs from diarization cannot be mapped to people in the picture.
【Function 2: speaking/listening state determination】
  - Co-present scenes: if only one person is speaking, and the difference in SyncNet score between the two people exceeds a preset threshold, the person with the lower score is judged to be in a "listening state."
  - Non-co-present scenes: if the ASR result is valid, the transcription confidence exceeds a threshold, but that person's SyncNet score is below a preset threshold, they are judged to be a listener.
  This gives rise to the listening branch — an example of upgrading SyncNet from a "filter" to a "semantic state classifier."
【Function 3: sync-quality filtering and annotation】SyncNet's three outputs (offset, confidence, embedding distance) are persisted as annotations for every clip in the dataset, available for downstream users to filter as needed; Figure 3 gives a distribution histogram of SyncNet confidence.
【Event-level alignment】Not addressed. The dataset consists of pure-speech scenes, with no need for sound-effect-to-visual-event alignment detection (e.g., footsteps, collisions), and Synchformer, ImageBind, and other semantic-alignment tools are not used.
【Evaluation side】One of VidChatBench's six dimensions is audio-video synchronization, likewise measured with SyncNet confidence.

### [Step-Video-T2V](../models/Step-Video-T2V.md) ⚠️

Not applicable. The model has no audio modality, and the training data contains no audio track, so there is no audio-video sync detection step in the pipeline; the report contains no mention whatsoever of SyncNet, AV-align, lip sync, event alignment, or any related content. [Uncertain]

### [UniTalking](../models/UniTalking.md)

Audio-video sync in UniTalking is decomposed into two independent sub-problems — "sound-source homology" and "lip-audio temporal alignment" — each gated by a dedicated model in series at the cross-modal filtering stage. This decomposition is the most valuable design in this work regarding alignment detection:
【First gate: sound-source homology detection (LightASD)】Used "to filter out videos containing purely diegetic audio." LightASD is a lightweight Active Speaker Detection model, whose task is precisely to determine "whether the person currently on screen is speaking / whether the speech in the audio track comes from a person on screen." This step removes samples where "sound is present but its source is not on screen," such as off-screen voice-overs, narration, post-production dubbing, and background dialogue. In such samples, the audio track and picture may have no temporal relationship whatsoever, and this is the primary source of contamination for speaker datasets. UniVerse-1 has no corresponding facility at all (it explicitly does not remove off-screen sound sources), which is a point where UniTalking is relatively ahead.
【Second gate: lip-audio temporal alignment (LipSync)】After confirming the sound source is homologous, the frame-level temporal alignment between lip movement and speech is evaluated, and poorly aligned samples are removed. What is removed here is samples where "the sound source is indeed on screen but there is an audio-visual offset" — common causes include audio-video timestamp drift from transcoding/muxing, post-production editing misalignment, and dub replacement.
【Logical relationship between the two gates】The former is a semantic/source-level judgment (is the sound produced by them), the latter is a temporal-level judgment (is the timing of what's produced correct). The series order is sound: first exclude those that are not co-sourced at all, then verify temporal precision for those that are co-sourced. This essentially constitutes a separate treatment of the temporal and semantic dimensions (see temporal_vs_semantic_sync).
【Event-level alignment: not applicable】UniTalking's data domain is pure speech-speaker content, not involving foley-event alignment, so there is no AV-align or Synchformer-type general event-alignment detection. In the Related Work section, the paper explicitly criticizes existing unified generation work for "mainly focusing on the sync between foley sound and video, failing to reach the precision required for speech" — i.e., it deliberately chose a different alignment target.
【Architecture-side alignment guarantee】Beyond data filtering, alignment is mainly carried by three architectural designs: (1) Joint Attention performs a single attention pass over concatenated audio-video tokens, explicitly modeling the viseme-phoneme correspondence; (2) anisotropic RoPE degrades the audio's spatial dimension while strengthening the temporal dimension; (3) the TV2A task during training — by imposing an attention mask in the "audio token to video token" direction, it blocks the audio branch's influence on the video branch, forcing the audio branch to predict audio based solely on the latent features of the ground-truth video, thereby learning the precise mapping from frame-by-frame visual motion to acoustic outcome. This is a design that constructs a one-directional supervision signal using an attention mask, the most direct alignment supervision on the training side.
【Effect verification】The attention-map visualization in Figure 5 shows: in audio-to-video attention, the face and body regions receive higher weight; in video-to-audio attention, the audio tokens attend "exclusively" to the lip region — providing intuitive evidence that Joint Attention has indeed learned lip-audio correspondence. The paper also admits that there are mismatched highlighted regions in the attention maps that vary randomly across training iterations, speculated to stem from "flaws in the training strategy or data noise" — the only place in the paper that acknowledges the impact of data noise.

### [UniVerse-1](../models/UniVerse-1.md)

Audio-video sync detection is the final and hardest gate in UniVerse-1's cleaning funnel, but it applies only to the speech subset, and only handles lip sync, not general event alignment:
【Lip-sync detection (for offline filtering)】Two steps in series:
1) RetinaFace face detection — first confirms a face exists in the frame that can be used for lip-sync verification; speech clips with no face cannot enter the speech subset;
2) SyncNet lip-audio sync confidence — for speech clips containing a face, a confidence score is computed, requiring > 2.0 to be retained; clips that pass are explicitly tagged "contains speech."
Together, these two steps filter 1,187 hours of high-confidence speech data out of a larger candidate pool.
【Event alignment (non-speech)】The paper performs no temporal-alignment detection whatsoever on non-speech general audio-video data — no AV-align, no Synchformer-type temporal-alignment scoring as a filtering condition, and no semantic-matching filter either. That is, the 3,074 hours of general data plus 3,422 hours of VGGSound/AudioSet undergo no audio-video alignment verification at all, relying solely on the natural assumption of "a native, synchronized audio track" to ensure alignment. This is a fairly significant simplification, and also explains why the model performs reasonably at coarse-grained coordination of ambient sound but lacks precise event-level alignment capability.
【Architecture-level alignment guarantee】The weak data-side alignment is compensated for by architecture and engineering: (1) the audio sample rate is downsampled from 44.1 kHz to 25.6 kHz, so the audio latent's temporal grid strictly aligns with 25 fps video; (2) the two towers are deeply fused at corresponding block-level depths + cross-modal attention, so alignment continues to occur at the feature level; (3) online annotation ensures the text condition and the audio-video window share the same source.
【Evaluation side】Synchformer is used to compute AV-A (Audio-Video Alignment), a temporal-sync score (lower is better; UniVerse-1 scores 0.23), and LSE-C is used to measure lip sync (higher is better; 1.34). Note that the Synchformer used for evaluation is not used as a filter for training data.

### [Unison](../models/Unison.md)

Audio-video sync detection is the only explicitly described data-side filtering mechanism in Unison, but it covers only lip sync, not general event alignment:
【Lip-sync detection — the lip-filtering operator, two steps in series】
1) Face detection: detects the number and locations of faces in the frame, yielding bounding boxes. The detector model is undisclosed. It also obtains the "count" information, indicating an ability to recognize multi-face scenes;
2) In-box SyncNet verification: SyncNet is run "exclusively within these bounding boxes" to verify alignment. This "restricted-region" design is the most engineering-valuable detail in this work's sync detection — when SyncNet is run over the full frame, if the face occupies a small fraction of the frame, or the frame contains multiple people, the sync confidence gets diluted and distorted by a large amount of irrelevant pixels; restricting to the face box substantially improves the signal-to-noise ratio of the judgment. UniVerse-1 adopts a loose serial combination of "RetinaFace detection → full-frame SyncNet," whereas Unison genuinely couples the two together.
【Dual removal targets】The paper explicitly states this operator "naturally filters out" two categories of samples: (i) clips where speech and lip movement are out of sync; (ii) off-screen voice-overs/narration. The explicit statement of the second category deserves emphasis — off-screen narration is the norm for documentary, tutorial, and vlog-type content, where the audio track has speech but the picture has no corresponding speaker; failing to remove such samples would teach the model that "speech doesn't require mouth movement," one of the most destructive kinds of noise for lip-sync learning. Unison is one of the few works in this survey to explicitly state "removing off-screen audio" as a design goal.
【Non-speech event alignment: no data-side detection】The paper performs no temporal-event-alignment verification on non-face data such as VGGSound — no AV-align, no Synchformer-type scoring as a filtering condition, no onset-alignment detection. That is, the alignment between ambient sound and on-screen events relies entirely on the natural assumption of "a native, synchronized audio track."
【The center of gravity for alignment assurance is at the training layer, not the data layer — Unison's core methodological choice】The data side only performs lip filtering; event-level alignment assurance is carried entirely by three mechanisms:
1) Architecture layer: frame-level bidirectional cross-attention, with a three-frame window for alignment, stride=1, retaining only the middle-frame representation — a deliberately designed, extremely short local-alignment window;
2) Training layer: bidirectional cross-modal forcing, where decoupled denoising timesteps let a clean modality guide a noisy modality, forcing the model to rely on cross-modal information rather than its own within-modality prior;
3) Within-audio: the speech stream and the sound-effect stream reuse the same RoPE temporal index, ensuring the two audio streams are strictly time-aligned with each other.
The paper's position is clear — alignment is "trained into the model" rather than "filtered into the data," which stands in contrast to MOVA's data-driven route (Synchformer + ImageBind dual-dimension threshold filtering). The ablation in Table 2 supports this position: removing CMFS worsens DS from 0.08 to 0.19 (a 2.4x degradation), the most severe single-item degradation among all ablations, demonstrating that the training strategy does dominate the contribution to alignment.
【Evaluation-side sync metrics】SyncNet's LSE-C (3.30) and LSE-D (7.88) measure lip sync; Synchformer's DeSync (DS) score measures the absolute time offset of modal onsets — Unison's 0.08 is the best across the entire TI2AV field (better than LTX-2's 0.10 and MOVA's 0.13), and 0.06 under the T2AV setting is likewise the best. Note Synchformer is used only for evaluation, not as a data filter — unlike MOVA, which uses Synchformer as a filtering threshold.

### [Veo 3 / Veo 3.1](../models/Veo_3_Veo_31.md) ⚠️

[Uncertain] The data-side audio-video sync detection method is entirely undisclosed. Officially, only the architectural source of synchrony is explained: video and audio latents are jointly denoised within the same diffusion process, so synchrony is guaranteed by architecture rather than by post-hoc alignment. This implies the training data itself must already be strictly synchronized native audio-video pairs, so some form of sync-quality screening (lip sync, event alignment) must exist, but Google discloses no detection method. Officially it is only stated, at the results level, that Veo 3.1 performs best on the audio-video synchronization dimension.

### [Vidu S1](../models/Vidu_S1.md)

Audio-video sync is placed at two points in the data pipeline:
(1) Coarse screening at the pre-filtering stage: before entering the pipeline, raw videos that are unsynced are removed based on the "audio-visual synchronization" metric, listed alongside frame rate, resolution, and audio-video completeness as one of four technical gating criteria;
(2) Fine-grained alignment at the diarization stage: the paper emphasizes that "only when the model observes, during training, visual expressions consistent with the corresponding speech can it learn high-fidelity lip sync." Therefore VAD + ASD are used to locate the timestamp and speaker of every speech segment, and the match between the speaker and the on-screen subject determines onscreen/offscreen/overlap. Offscreen (voice-over, where the audio subject and picture subject don't match) and overlap (multiple overlapping voices) are both treated as breaking the audio-visual correspondence — clips containing overlap are removed in their entirety. Additionally, shot boundaries during segmentation avoid cutting through the middle of speech, also to protect the integrity of speech-lip segments.
The evaluation side uses the Sync-D metric (based on the SyncNet/Wav2Lip lip-sync expert, citing reference [59] Prajwal et al., "A lip sync expert is all you need") to measure audio-video sync; Vidu S1 achieves 7.8470 on HDTF (lower is better), better than OmniAvatar's 9.242, StableAvatar-1.3B's 11.18, Hallo3's 8.660, Wan2.2-S2V-14B's 8.255, LiveAvatar's 8.447, LemonSlice's 7.921, HeyGen's 8.037, and Kling Avatar 2.0's 8.158.

### [Wan 2.5 / 2.6 / 2.7](../models/Wan.md) ⚠️

The only clearly written audio-visual sync data-filtering method in the Wan family appears in the Wan2.2-S2V paper — this is the single most valuable piece of first-hand information in this entry for the survey topic; 2.5/2.6/2.7 themselves disclose nothing.
【Wan2.2-S2V's approach (end of Chapter 2, "Pose Tracking and Fine-grained Filtering," in the paper)】Quote: "to address audio-visual alignment challenges, we utilized Light-ASD to detect and exclude videos where (1) the audio is not synchronized with the active speaker, or (2) no active speaker exists in the scene."
That is, Light-ASD (a lightweight active-speaker-detection model from CVPR 2023, Liao et al. 2023) is used as the audio-visual alignment discriminator, executing two exclusion rules:
(1) the audio does not sync with the active speaker on screen → removed (directly corresponding to dubbing, post-hoc added audio, off-screen sound, and other asynchronous samples);
(2) there is no active speaker in the frame at all → removed (corresponding to narration, off-screen sound sources, and samples where audio and picture have no causal relationship).
The combination of these two rules effectively achieves both "removal of temporally unsynced samples" and "removal of samples whose sound source is not on screen" at once — a fairly clean design.
【Accompanying precondition】The same paragraph also requires "retaining videos where the face remains consistently visible throughout the entire sequence," with the reason explicitly stated as "to ensure the model can learn audio-driven facial expressions from the given audio signal" — i.e., the prerequisite for lip-sync learning is that the face is visible throughout, a data-admission condition complementary to sync detection.
【Wan 2.1 V2A's handling of alignment (architecture side, not filtering side)】There is no sync-detection filter at all; instead, temporal alignment is ensured by three things: (1) abandoning the "mel-spectrogram + image-style VAE" route, because the DiT needs to patch the 2D latent and reshape it into (Ha·Wa)×Ca, a process that destroys temporal alignment with the video content; instead, a 1D-VAE is trained directly on the raw waveform, producing a Ta×Ca latent that explicitly preserves the time axis, which the report states is "necessary for precise synchronization." (2) CLIP is used to extract frame-by-frame visual embeddings, which are then rate-adapted to match the audio feature sample rate via feature replication, then linearly projected and added element-wise for fusion. (3) In implementation, input video is downsampled to a fixed ratio of "12 seconds corresponding to 48 frames" to ensure precise frame-level synchronization.
【2.5/2.6/2.7】Officially there are only promotional statements of "native audio-visual sync" and "sound directly drives the character's lip movement and performance," with no detection method, model, or process disclosed. [Uncertain]

### [Collection of audio-video generation evaluation benchmarks](../models/av_benchmarks.md)

Five benchmarks together constitute the most complete collection of AV sync detection methodology currently available:

【AV-SyncBench — decoupled sync detection (the core method for this survey)】Unified protocol: video and audio are cut into non-overlapping 0.64-second chunks, visual and audio embeddings are extracted independently, and the diagonal elements of the temporal similarity matrix are computed as cosine similarity and averaged: S = (1/N)Σ sim(v_i, a_i); judgment uses binary accuracy — whether the score for the original pairing is higher than for a perturbed pairing. The temporal side examines separability across three types of perturbation (global shift, local jitter, global speed change); the semantic side, while strictly preserving temporal invariance, changes only timbre/sound source, examining semantic separability. Five representation models are tested: Synchformer, SparseSync, ImageBind, CAV-MAE, and CAV-MAE-Sync.

【VABench】Two sync tracks: ① Desync — Synchformer is used to predict the audio-video offset, computed only within the first/last 4.8-second windows; ② Lip-Sync — borrowing the LatentSync method to compute alignment confidence, applied only to the speech-language subset where a talking head is detected. Cross-modal semantic alignment additionally uses a three-piece set: ViCLIP (text-video, with temporal understanding), CLAP (text-audio cosine similarity), and ImageBind (joint audio-video embedding space).

【AVBench】Turns AV consistency into a trainable, dedicated evaluator (a fine-tuned Qwen2.5-Omni 7B) rather than relying on a fixed SyncNet-type model; it also has a separate Lip Sync Consistency dimension.

【PhyAVBench】Proposes FGAS (Fine-Grained Alignment Score) — the cosine similarity between frame-level visual and audio tokens, taking the mean of the diagonal elements of the temporal similarity matrix; its core metric CPRS uses the CAV-MAE Sync encoder (later versions switch to CLAP embeddings) to measure the consistency between the direction of acoustic change across paired samples and the true reference vector.

【Omni-Judge】Treats audio-video synchronization as one of 9 dimensions handed to Qwen3-Omni for judgment, with a negative conclusion: τ_b is only 0.142, explicitly indicating that Omni-LLMs are unsuited to sync detection due to insufficient temporal resolution.

### [Video captioning model ecosystem](../models/caption_models.md) ⚠️

The relationship between the captioner ecosystem and audio-video sync detection is "indirect but critical" — captioners do not directly perform sync detection, but they serve as the semantic acceptance-check for sync quality:
【Sync-related functions taken on by captioners】
· AVSCap's third criterion, Audio-Visual Synergy, explicitly requires captions to explicitly bind audiovisual events and perform temporal alignment — an attempt to elevate "synchrony" from a signal-level metric to a semantic-level description. AVSCapBench's Synergy dimension quantifies exactly this capability: AVSCap-7B 57.70, Gemini-3-Pro 48.88, AVoCaDO 29.13, video-SALMONN-2 12.43, bare Qwen2.5-Omni base model 7.00.
· AVoCaDO's fifth dimension, Cross-modal Narrative Logic (audio and video mutually explaining/supplementing/guiding one another), is an earlier form of the same idea, but its Synergy score is only 29.13, showing that having this dimension in a checklist does not mean the model has actually learned it.
· Ovi explicitly requires captions to cover all relevant visual and auditory events while "respecting chronology," and conducted multiple rounds of prompt-iteration experiments for this purpose.
【Signal-level sync detection is handled by dedicated tools (upstream/downstream of the captioner)】SyncNet and Synchformer are the mainstream choices: Foley-Omni uses Synchformer both as a training feature provider and as the sync scorer in the filtering stage (one model serving two purposes, keeping the filtering criterion aligned with the model's learning objective); MOVA uses LSE-D/LSE-C; SkyReels-V4 uses SyncNet. These are decoupled from caption generation.
【The speaker-attribution problem for lip sync】TalkNet (InstructAV2AV) performs active speaker detection to determine "who is speaking"; CineDance uses Qwen3-Omni + a sliding-window prompt to raise ASR-to-character binding accuracy from 67.2% to 95.4% — speaker attribution is the task on the caption side that comes closest to sync detection.
[Uncertain] No captioner work outputs sync-detection results directly as a caption field (e.g., "audio-visual desync of 0.3 seconds"); sync information is only implicitly present in captions in the semantic form of "event binding."

### [Collection of geometric/structured annotation datasets](../models/geometric_datasets.md)

Not applicable. None of the four datasets have an audio track, so there is no lip-sync or audio-visual event-alignment detection involved. The analogous concept of "alignment" in this batch of datasets is expressed as cross-modal geometric alignment — WildWorld uses multi-source timestamp synchronization to ensure RGB frames, depth frames, skeleton data, and world state are strictly aligned at the same instant (this is the core engineering challenge of the capture platform); SpatialVID uses camera-pose priors to verify and correct VLM-generated motion-direction descriptions, which is "text-geometry alignment" rather than audio-video alignment.

### [Video generation post-training data strategies](../models/post_training_data.md) ⚠️

The anchor paper's post-training data side does not involve audio-video sync detection at all [Uncertain] — none of the four reward models (video aesthetics, image aesthetics, motion quality, text-video alignment) involve synchrony, nor do PE's three rewards. AV models are handled only in the Phase 4 distillation stage, following OmniForcing with asymmetric block-causal alignment and an audio sink token — an architectural alignment, not data-side sync detection. This means that even when this framework is used to run a complete four-stage post-training process on an AV model, lip-sync and event-alignment quality are not directly optimized by any reward signal.
【The only complete sync-reward practice on the post-training side across the survey】JavisDiT++'s AV-DPO: among six reward models, Synchformer specifically handles temporal synchrony and ImageBind specifically handles cross-modal semantic similarity, and both participate jointly in preference ranking — the only public case of treating AV sync as an independent preference dimension. About 25,000 preference pairs, a prompt pool of 30,000 (non-overlapping with the SFT data), 3 generations + 1 ground truth per prompt for 4 candidates total, 1 epoch, LoRA with 121M trainable parameters.
【Others】Seedance 1.5 pro's RLHF is stated to cover "audio fidelity," but whether it includes a sync dimension is undisclosed [Uncertain]; whether Kling 3.0 Omni's DPO treats lip sync as an independent scoring item is undisclosed [Uncertain]. Across all models, data-side AV sync filtering (SyncNet/Synchformer threshold selection) uniformly occurs in the pretraining data pipeline rather than the post-training stage.

### [Combined survey of mainstream video pretraining datasets: Panda-70M, InternVid, Koala…](../models/pretraining_datasets.md)

Not applicable. None of the seven datasets have an audio modality, so there is no audio-video sync detection step, and no lip-sync or audio-visual event-alignment method is used.
## Specific sync-detection metrics and thresholds (SyncNet/Synchformer/LSE/in-house, threshold values, e.g. MOVA LSE-D≤9.5 and LSE-C≥4.5, SkyReels-V4 SyncNet |offset|≤3∧conf>1.5)

`sync_metric_and_threshold` · Detail level: detailed

### [Allegro](../models/Allegro.md)

Not applicable. No use of SyncNet, AV-align, or any in-house sync metric; no corresponding threshold.

### [Apollo](../models/Apollo.md) ⚠️

[Metrics] Clear: temporal synchronization uses the alignment score/offset output by Synchformer; semantic synchronization uses ImageBind cross-modal similarity.
[Thresholds] Completely undisclosed — the paper only states "ensuring high synchronization in both temporal and semantic dimensions," without giving an upper bound on the Synchformer offset (e.g., |offset| < 0.2s) or a lower bound on the score, nor an ImageBind similarity threshold (e.g., IB-score > 0.2), and it does not clarify whether the two conditions are combined via AND or a weighted combination.
[Comparative reference] Contemporaneous work MOVA published a complete threshold table (Audiobox PQ>5.0/CU>4.5/CE>2.5, DOVER-Aesthetic>0.85, LSE-D≤9.5, LSE-C≥4.5, etc.), and UniTalking disclosed SyncNet conf>0.9; Apollo's disclosure on this dimension lags significantly behind.
[Evaluation-side figures (not filtering thresholds, do not conflate)] Table 3 gives the model's output sync metrics: DeSync 0.650 (lower is better), Sync-conf 6.787 (higher is better), IB 0.316 (higher is better).
[Uncertain]

### [CineDance / CineDance-1M](../models/CineDance.md) ⚠️

[Metric selection] Data side: SyncNet (lip sync) + ImageBind (global cross-modal alignment); evaluation side: Sync-C, Sync-D, IB-Score.
[Thresholds] The paper does not disclose any filtering threshold value for any sync metric — this is the fundamental methodological difference between this work and works that adopt hard thresholds such as MOVA (LSE-D ≤ 9.5 and LSE-C ≥ 4.5) and SkyReels-V4 (SyncNet |offset| ≤ 3 and conf > 1.5). CineDance explicitly adopts a "score everything → store as metadata → no hard pruning" strategy; the original text states "we store all quality scores as metadata, enabling users to flexibly construct task-specific subsets," delegating threshold decisions to downstream users.
[Trade-off assessment] The advantage is preserving data completeness and downstream flexibility, avoiding loss of the long tail from over-filtering; the disadvantage is that the dataset itself provides no guaranteed lower bound on sync quality — users must set their own filtering thresholds, and the paper offers no recommended threshold reference values.
[Specific SyncNet version, ImageBind version, score distribution range] All undisclosed. [Uncertain]

### [CogVideoX](../models/CogVideoX.md) ⚠️

[Uncertain] No publicly disclosed sync-detection metric or threshold whatsoever. Not applicable on the video side of CogVideoX; CogSound has no technical report, so it is undisclosed whether it uses metrics such as SyncNet / AV-align / ImageBind score, nor is any confidence threshold value disclosed. What can be confirmed is that no SyncNet-style lip-sync metric is used anywhere in the pipeline (the model does not generate speech).

### [Cosmos-Predict2.5](../models/Cosmos-Predict25.md)

Not applicable. No audio-visual sync metrics or thresholds of any kind (SyncNet / Synchformer / LSE-C / LSE-D, etc.). For reference, this work's data-admission rules with explicit numeric thresholds are concentrated in the Human Dynamics domain: a person must appear in >40% of frames, at most ≤8 people visible in any given frame, at least one person occupying ≥3% of the frame area; plus duration thresholds (discard <5 seconds, keep 5–60 seconds) and an end-to-end retention rate of 4%.

### [Data-Juicer 2.0](../models/Data-Juicer.md) ⚠️

[Uncertain] Since it provides no sync-detection operator, Data-Juicer has no recommended value for any sync metric or threshold either.
[The one cross-modal threshold DJ does disclose — for reference] video_frames_text_similarity_filter's minimum threshold in the official T2V optimal data pool is 0.306337 (CLIP visual-text cosine similarity). How this number arises is worth noting: it is not a manually set round number (like the common 0.3) but rather the boundary value that naturally falls out of splitting the data pool by percentile within the Probe-Analyze-Refine workflow — i.e., the specific value corresponding to the percentile point when "the subset with the highest similarity is retained." This "threshold derived by working backward from a target retention ratio" contrasts with the common "threshold set by human experience" approach used by model teams.
[Threshold status of other operators] The thresholds for all DJ operators are externalized YAML parameters; the values given in the official config_all.yaml are placeholder defaults rather than recommended values. The documentation explicitly states it does not provide cross-scenario general-purpose recommended thresholds; its position is that thresholds should be determined via Sandbox search on the specific dataset and specific downstream model.
[Methodological value] For this survey, what DJ provides is not "threshold values" but "a method for determining thresholds": split the data into three tiers by some statistic → train a reference model on each → evaluate with a unified benchmark → select the segment with the largest gain. This pipeline could in principle be applied directly to sync metrics (e.g., splitting SyncNet confidence into three tiers for probing), though DJ has not officially run such an experiment on the audio-visual sync dimension.

### [Foley-Omni](../models/Foley-Omni.md) ⚠️

[Specific figures on the cleaning side]
  · Synchformer sync score ≥ 0.2 — the temporal sync reliability gate
  · ImageBind (IB) score ≥ 0.3 — the audio-visual semantic consistency gate
  Both appear together under the Alignment dimension of Table 7, each an independent hard threshold; the pass condition is "both must be satisfied."
[Evaluation-side metrics]
  · DeSync (lower is better) — temporal alignment error. Foley-Omni 0.16 vs. Ground Truth 0.14 vs. cascaded baselines 0.85/0.26.
  · IB score (higher is better) — audio-visual semantic consistency. Foley-Omni 0.26 vs. Ground Truth 0.36 vs. baselines 0.25/0.16. Note that the generated results' IB of 0.26 sits at the same order of magnitude as the filtering threshold of 0.3, indicating that this 0.3 gate is already moderately-to-strictly set relative to real data.
  · CLAP (higher is better) — semantic relevance between audio and the text description. Foley-Omni 0.27 vs. GT 0.30.
  · WER (lower is better) — speech intelligibility. On the V2ST-Bench, Foley-Omni scores 7.59, even lower than Ground Truth's 8.03 (because the generated speech is cleaner and free of ambient interference compared to real recordings); on the GRID subset, 15.3.
  · Three subjective MOS scores: A-MOS 3.92 (audio quality), S-MOS 4.13 (semantic consistency), T-MOS 4.14 (temporal sync), against GT's 4.33/4.37/4.42 — the smallest gaps are on S-MOS and T-MOS.
[Threshold strictness assessment] Compared with similar work in the field (e.g., UniVerse-1's strict SyncNet threshold, the alignment filters used in MMAudio-family work), Synchformer 0.2 and ImageBind 0.3 are moderately-to-loosely set, favoring retention of data volume. The paper provides no threshold sensitivity analysis.
[Uncertain] The basis on which the thresholds were determined is unstated (whether via grid search, whether a retention-rate-vs-performance trade-off curve was reported — neither is documented). The specific computation configuration for the Synchformer/ImageBind scores (window length, feature aggregation method) is also unstated, affecting reproduction accuracy.

### [Goku](../models/Goku.md)

Not applicable. No audio-visual sync metric or threshold. Worth recording for reference is its visual-side consistency threshold system (as a reference for "threshold-setting style"): DINOv2 inter-frame cosine similarity ≥0.85 (480×864) / ≥0.90 (720×1280); aesthetic score ≥4.3 (480p) / ≥4.5 (720p+); OCR text area ≤0.02 (480p) / ≤0.01 (720p+); RAFT motion score [0.3, 20.0] (480p) / [0.5, 15.0] (720p) / [0.5, 8.0] (1080p). These thresholds are all tiered by resolution and tighten as resolution increases — a unifying philosophy in Goku's threshold design.

### [Hailuo / MiniMax Video](../models/Hailuo.md)

Not applicable. No sync metric or threshold of any kind (SyncNet / Synchformer / LSE-D / LSE-C, etc.). Not comparable with entities such as MOVA (LSE-D≤9.5 and LSE-C≥4.5), SkyReels-V4 (SyncNet |offset|≤3 and conf>1.5), or Vidu S1 (Sync-D evaluation).

### [HunyuanVideo-Foley](../models/HunyuanVideo-Foley.md) ⚠️

[Metrics clear, thresholds entirely absent] This is the core conclusion for this field. The paper explicitly names two detection tools (ImageBind for semantics, AV-align for timing) but gives no filtering threshold value for either — neither a lower bound on ImageBind similarity nor a gate on the AV-align score, nor whether the two are combined via AND (both must pass), weighted combination, or tiered handling. [Uncertain]
[Disclosure comparison with similar work] UniVerse-1 gives SyncNet conf > 2.0; MOVA gives LSE-D ≤ 9.5 and LSE-C ≥ 4.5, with separate thresholds set for SynchFormer and ImageBind respectively. HunyuanVideo-Foley's tool selection closely resembles MOVA's (likewise a dual-tool setup of temporal + semantic), but its threshold disclosure is markedly lower — tool names given, numbers absent.
[Metric-system divergence caused by task differences] This work does not involve lip sync, so it makes no use whatsoever of the SyncNet / LSE-C / LSE-D face-speech sync metric family, instead using the general-purpose acoustic-event-oriented AV-align and Synchformer/DeSync. These two metric families target different objects and are not numerically comparable: the LSE family measures the correspondence between lip shape and phonemes, while AV-align/DeSync measures the temporal correspondence between arbitrary visual events and acoustic events — the latter is inherently harder to judge, since "what counts as a visual event" is itself ambiguous.
[Evaluation threshold vs. filtering threshold] The evaluation-side DeSync figures (0.53–0.74, a misalignment estimate in seconds) are measured results from model output, not a data-filtering gate; the two must not be conflated.
[Threshold calibration method unstated] The paper does not state whether any threshold was calibrated by manual spot-checking, determined via ablation, or set by experience. [Uncertain]

### [HunyuanVideo](../models/HunyuanVideo.md) ⚠️

Not applicable. No use of SyncNet, AV-align, or any sync metric; no threshold. [Uncertain]

### [InstructAV2AV](../models/InstructAV2AV.md) ⚠️

[Data-construction side: no numeric threshold]
  [Uncertain] This is the most obvious difference between this work and works such as MOVA (LSE-D≤9.5 and LSE-C≥4.5), SkyReels-V4 (SyncNet |offset|≤3 and conf>1.5), and Foley-Omni (Synchformer≥0.2, ImageBind≥0.3) — InstructAV2AV sets no numeric threshold for a sync score whatsoever during the data-construction stage. Its sync quality is instead guaranteed by two non-threshold mechanisms: (1) TalkNet+Scribe's Boolean admission condition that "the speech must be temporally aligned with a visible on-screen speaker"; (2) the architecture-level enforcement of frame-wise cross-attention during the synthesis stage. This is a "mechanism replaces threshold" approach — the benefit is that no threshold needs to be calibrated and there is no risk of false rejection/acceptance from a poorly set threshold; the drawback is that the dataset's sync quality cannot be quantitatively described nor compared horizontally against other datasets.
  The only data-side threshold with an exact numeric value is the audio-silence threshold of −45 dBFS (PyDub), but that is not a sync metric. The remaining thresholds (CoTracker3 motion magnitude, LAION aesthetics, Audiobox-Aesthetics, the MLLM five-dimension scoring gate) are all given with no numeric value.

[Evaluation side: metrics and configuration are clear]
  · Sync-C (S-C, higher is better) — SyncNet lip-sync confidence.
  · Sync-D (S-D, lower is better) — SyncNet sync distance/offset. Both have an explicitly given configuration of vshift=15 frames (i.e., searching for the best alignment position within a ±15-frame window — SyncNet's standard setting).
  · PEAVS (higher is better) — a perceptual-level audio-visual sync evaluation metric that, unlike SyncNet, is not limited to the lips and can cover general audio-visual event synchrony — a necessary complement for this work's non-speech editing tasks (instance insertion/removal).
  · AV-A (higher is better) — an ImageBind-based audio-visual semantic alignment score. On the InsAVE-80K evaluation set, InstructAV2AV 27.72 vs. AvED 26.44 / AVI-Edit 26.37 / CoherentAVEdit 22.67; on AvED-Bench, 23.71 vs. AVI-Edit 23.21.
  These four metrics span three levels — "lip-sync precision (Sync-C/D)," "general event-synchrony perception (PEAVS)," and "cross-modal semantic matching (AV-A)" — a fairly complete coverage.
  [Uncertain] The specific values for Sync-C / Sync-D / PEAVS are not fully presented in the accessible table content; only AV-A has an exact figure.

[Methodological gap] No threshold on the data side but metrics on the evaluation side means there is no common yardstick for cross-referencing "the sync quality of the data" against "the model's sync capability." If Sync-C/Sync-D had also been used to measure and report the dataset's sync-score distribution during data construction, it would answer the key question of whether the model's sync capability has already approached the data ceiling — unfortunately this was not done.

### [Other Audio-Visual Joint Generation, 2026](../models/JAVG_2026_misc.md) ⚠️

[Only one item with a fully disclosed threshold]
OmniCustom: SyncNet "|offset|≤3" and "confidence>1.5" (identical to and directly inherited from Ovi); paired with removal for aesthetic score <0.3 and removal for duration <10 seconds. The three rules are combined via AND.

[ALIVE's related thresholds (not sync thresholds, but likewise data-filtering thresholds — the only set of numeric values disclosed in this batch)]
Valid-pairing determination for Character-driven cross-pair:
- face similarity (ArcFace face similarity) > 0.35
- CLIP similarity (CLIP visual similarity) > 0.7
- absolute proximity to 0.9
Identity anchor selection: the 1.5-second sub-clip with the highest sync score.
SubjectID correction: filters out videos that fail to meet "high-confidence thresholds," with the specific value undisclosed [Uncertain].
Other: cross-attention signal dropout probability of 0.3 during training; FAISS retrieval similarity threshold τ>0.85 during inference.
TS-TalkNet's sync-score threshold is undisclosed [Uncertain].

[NAVA] The list of detectors is disclosed (SyncNet, SyncFormer, ImageBind), but it "does not provide exact numerical thresholds" — all thresholds are undisclosed [Uncertain]. Likewise, thresholds for the visual-quality operators (aesthetics, sharpness, brightness, motion score) and AudioBox Aesthetic are all undisclosed [Uncertain]. This is a typical industrial paper with a fully described data pipeline but all numeric values withheld.

[StreamChar / CCL / Baton] No sync threshold on the data side [Uncertain].

[Evaluation-side metrics (not filtering thresholds, but reflecting each work's measurement approach to sync)]
- CCL: Table 2 reports five metrics for ablation — WER, Sync-C, Sync-D, DeSync, IB. Sync-C/Sync-D are SyncNet-family confidence and distance, DeSync is Synchformer-family desynchronization, and IB is the ImageBind score. Using all five together indicates a finer-grained measurement of sync than most other works.
- NAVA: sets new SOTA on Verse-Bench for Sync-C / Sync-D / video quality / audio WER, with 2–5x fewer parameters than open-source baselines.
- ALIVE: Alive-Bench 1.0 contains 22 fine-grained metrics (across six major categories), and sync should be among them [details uncertain].
- ITS-JAVG: three sync-related verifiers — JavisScore (fine-grained sync), AVHScore (audio-visual event semantic consistency), ImageBind (audio-visual semantic similarity); the ARW normalization formula is R(i)=Σ_k w_k·r_k(i)/(σ_k+ε).

[Cross-work observation] In 2026, threshold disclosure in this field polarizes: academic/small-to-mid-size teams (OmniCustom) disclose fully and directly adopt the Ovi standard; industrial teams (NAVA, ALIVE) disclose the process but withhold the numbers. SyncNet's |offset|≤3 / conf>1.5 has become entrenched as a de facto community default configuration.

### [Audio-Visual Joint Generation Baseline Collection](../models/JavisDiT_baselines.md) ⚠️

This collection generally reports sync metrics but rarely gives data-filtering thresholds — this is the biggest difference from industrial models such as Ovi (|offset|≤3 and conf>1.5), MOVA (LSE-D≤9.5 and LSE-C≥4.5), and SkyReels-V4:
[MM-Diffusion / AV-DiT] No sync metric, no threshold. Evaluation uses only FVD (computed with an i3d model) and FAD (computed with AudioCLIP), plus a human Turing test (10,000 votes, >80% fooled the humans).
[JavisDiT — proposes JavisScore (the only new metric in this collection)]
- Specific parameters fully disclosed: 2-second window, 1.5-second overlap (0.5-second sliding step); ImageBind used underneath to compute audio-visual similarity; within each window, aggregation is taken over the "worst-synchronized 40% of frames."
- Validation set: 3,000 manually annotated samples.
- These are metric-computation parameters, not data-filtering thresholds — JavisDiT does not filter training data by sync score [Uncertain].
[JavisDiT++ — an 11-dimensional evaluation system] Complete metric list: quality dimension FVD, FAD; text-consistency dimension TV-IB (text-video ImageBind), TA-IB (text-audio ImageBind), CLIP-Score, CLAP-Score; audio-visual consistency dimension AV-IB, AVHScore, JavisScore; sync dimension DeSync (based on Synchformer). AV-DPO uses Synchformer as a sync reward model, but the scoring threshold/ranking details are undisclosed [Uncertain]. All evaluation is done at a 240P, 4-second configuration.
[Harmony — four sync metrics] Sync-C and Sync-D (SyncNet-family lip-sync confidence and distance), DeSync Score (based on Synchformer), ImageBind (IB) Score. The ablation gives a key chain of figures: Sync-C baseline 4.29 → (adding RoPE positional alignment) 4.80 → (adding Cross-Task Synergy training) 5.09 → (adding Synchronization-Enhanced CFG) 6.51. The name and threshold of the "audio-visual consistency scoring model" used for data filtering are undisclosed [Uncertain] — this is the single most critical piece of missing information on Harmony's data side.
[UniAVGen] Evaluation uses SyncNet (sync), VBench (video quality), AudioBox-Aesthetics (audio quality), Whisper WER (speech intelligibility); the data-side sync filtering threshold is not mentioned [Uncertain].
[Cross-work significance] This collection offers a rich selection of sync "evaluation metrics" (JavisScore / DeSync / Sync-C/D / AV-IB / AVHScore) but almost no sync "data-filtering thresholds," indicating that academic baselines focus more on how to measure sync, while industrial models focus more on how to use thresholds to filter for clean data — the two are complementary.

### [Kling 3.0 / Kling Video 3.0 Omni](../models/Kling_30_Omni.md) ⚠️

[Uncertain] No sync metric name or threshold value is disclosed (e.g., a specific gate for SyncNet confidence / LSE-C / LSE-D / AV-align). The only quantified audio gate in public materials comes from Kling-Foley's data specification: VAD silence ratio < 0.2; valid-sample requirements for evaluation are video ≥5 seconds, sound effect ≥2 seconds. The team reported SOTA figures across four major categories — distribution matching, semantic alignment, temporal alignment, and audio quality — in the Kling-Foley paper, but those are model evaluation metrics, not data-filtering thresholds.

### [LTX-2](../models/LTX-2.md) ⚠️

Completely undisclosed. The paper uses no SyncNet, AV-align, or any in-house sync metric, gives no confidence threshold value (cannot be compared with e.g. UniTalking's SyncNet conf>0.9), and reports no LTX-2 score on any objective audio-visual sync benchmark. All audio-visual quality conclusions come from an internal human preference study (comparing against Ovi, Veo 3, Sora 2), and the paper gives no specific score table for the human evaluation, only qualitative descriptions such as "significantly better than Ovi" and "comparable to leading closed-source models." The only quantitative table is an inference-speed comparison (on H100, 121 frames at 720p, single-step Euler, CFG=1: LTX-2 19B audio-video 1.22 s/step vs. Wan 2.2-14B video-only 22.30 s/step, roughly 18x speedup). [Uncertain]

### [LongCat-Video](../models/LongCat-Video.md) ⚠️

[Uncertain]. The Avatar 1.5 report only states qualitatively that samples with "excessive audio-visual offset" are removed, without disclosing the sync metric used (no mention of specific metric names such as SyncNet conf / sync-c / sync-d / AV-align) or any threshold value. TalkNet and UniTalk, used as active-speaker detection models (outputting speaking/not-speaking plus confidence), likewise have their decision thresholds undisclosed. The only stage in the entire pipeline with a disclosed exact threshold is the emotion-annotation step's EmotiEffLib confidence s>0.7, which is unrelated to audio-visual sync.

### [MOVA](../models/MOVA.md)

All thresholds disclosed (Appendix Table 9 and Section 4.3), one of the most complete samples in this survey in terms of sync-filtering threshold disclosure:
[Stage 2 audio-visual alignment (loose OR gate)]
- ImageBind Score (semantic alignment) ≥ 0.2
- **OR** SynchFormer Offset (DeSync, temporal offset) ≤ 0.5
Note this is a logical OR rather than AND; the paper explicitly explains the design intent: "This ensures that both semantically relevant ambient sounds and temporally synchronized speech/actions are preserved" — because ambient sound inherently lacks sharp temporal onsets and is hard to verify via temporal-sync detection, while fast action sound effects may score low in the semantic embedding space; using AND would falsely reject both.
[Phase 2 lip-sync-specific thresholds]
- LSE-D ≤ 9.5 (smaller distance is better)
- LSE-C ≥ 4.5 (larger confidence is better)
Both must be satisfied simultaneously, yielding a high-quality lip-audio-correspondence subset of about 2.5M clips. Compared with UniTalking's SyncNet conf > 0.9, MOVA's LSE-C ≥ 4.5 is a value on the raw SyncNet confidence scale and represents a moderately-to-strictly set gate.
[Stage 2 audio-quality-related thresholds] Silence ratio < 0.8, bandwidth > 1,000 Hz, Audiobox PQ > 5.0 / CU > 4.5 / CE > 2.5.
[Stage 2 video-quality thresholds] DOVER-Aesthetic > 0.85, DOVER-Technical > 0.05; tightened to DOVER-Technical > 0.15 in Phase 2; DOVER-Technical > 0.14 in Phase 3 (720p).
[The same metric set reused on the evaluation side] DeSync (SynchFormer), IB-Score (ImageBind), LSE-D/LSE-C (SyncNet) are also used in evaluation, so the training-filtering and evaluation metrics are highly homologous. MOVA-360p + dual CFG (s_B=3.5) achieves DeSync 0.351 / IB-Score 0.315 / LSE-D 7.004 / LSE-C 7.800.

### [Merged entry: ① Mochi 1](../models/Mochi_MAGI_Motif.md)

Not applicable. None of the three use SyncNet, AV-align, or any in-house audio-visual sync metric; no corresponding threshold.

### [Movie Gen](../models/Movie_Gen.md)

Uses the cosine similarity of the in-house CAVTP audio-video embedding as the sole quantitative alignment metric, with thresholds tiered by audio type (Table 24, all determined by manual inspection):
· AED = sound / voice / voice+sound → cosine similarity > 0.2 judged as diegetic
· AED = music / music+sound / voice+music+sound → cosine similarity > 0.3 judged as diegetic (music types require a higher bar, since music's natural correlation with the picture is weaker and more prone to misjudgment)
· AED = music and similarity < 0.1 → judged as non-diegetic (background score)
· AED = sound+music and 0.1 < similarity < 0.25 → judged as non-diegetic
· AED = sound+voice+music and 0.1 < similarity < 0.25 → judged as mixed (on-screen/off-screen mix)
Pretraining keeps only the diegetic and mixed buckets, plus a small amount of non-diegetic background music.
No use of public sync metrics such as SyncNet or AV-align; at the evaluation stage, objective metrics use ImageBind score (audio-visual alignment) and CLAP score (audio-text alignment), while subjective metrics are further broken down into synchrony (Sync.) and correctness (Corr.).
Other related thresholds: PT2V uses ArcFace cosine similarity > 0.5 (same person across adjacent frames) and > 0.7 (identity preservation against a synthesized reference image).

### [NVIDIA NeMo Curator](../models/NeMo_Curator.md)

None. No sync metric or threshold of any kind exists (no SyncNet confidence/offset, no LSE-D/LSE-C, no in-house sync score), nor any related configuration item. For reference, the framework's other threshold systems that are fully transparent in their parameters can serve as a comparison: aesthetic score_threshold = 3.5, motion motion_score_global_mean = 0.00098, motion_score_per_patch_min_256 = 0.000001, DOVER removes the bottom 15%, clip duration 2–60 seconds, semantic dedup k-means k = 10000. This practice of "writing all default thresholds into the documentation" is itself worth emulating, but none of it covers the audio-visual dimension.

### [OmniHuman Dataset + OHBench](../models/OmniHuman.md) ⚠️

[Metric used on the data side] SyncNet response score S_sync + temporal offset, the two jointly determining the outcome.
[Disclosed values]
- Temporal offset tolerance: ±3 frames (±100 ms at a unified 30 FPS) — the only explicit numeric value on the sync dimension;
- Face-embedding similarity threshold: 0.55 (ArcFace cosine similarity, used for identity assignment, working together with the sync determination to complete attribution);
- Tracking-loss tolerance span: 5 frames.
[Key undisclosed value] The threshold for S_sync itself; the paper only states "above a preset threshold" — i.e., the most core sync-score gate is left blank. Compared with works that give explicit dual thresholds, such as MOVA (LSE-D ≤ 9.5 and LSE-C ≥ 4.5) and SkyReels-V4 (|offset| ≤ 3 and conf > 1.5), the granularity of disclosure here is clearly insufficient. Notably, OmniHuman discloses the offset threshold (3 frames) but withholds the confidence threshold — disclosing exactly half of the two necessary conditions. [Uncertain]
[Strictness of the decision logic] "All detected subjects satisfy [the condition]" — an all-must-pass rule rather than any-one-passes, a piece of information more important than the threshold value itself: in a two-person scene, if even one person's speech attribution is uncertain, the entire sample is discarded.
[SyncNet variant unstated] Not specified which implementation of the original SyncNet, which implementation of SyncNet-python, or a retrained version is used; also unstated whether S_sync is on a confidence or distance scale. [Uncertain]
[Sync metrics and thresholds on the evaluation side] In OHBench:
- Lip-Sync (individual level): SyncNet, measuring lip-audio alignment accuracy of the generated video; no passing threshold given (reported as a continuous score);
- V-A / Audio-Visual Sync (global level): ImageBind, measuring the temporal offset between visual and auditory events, reported as a continuous score.
[Consistency issue with the training-data threshold] Data filtering uses SyncNet + ±3 frames, and evaluation also uses SyncNet — the same model both filters the data and judges the outcome, presenting a mild risk of circular reasoning (a model trained on data filtered by SyncNet will naturally score well on SyncNet-based metrics). The paper does not discuss this. [Uncertain]

### [Open-Sora series](../models/Open-Sora.md)

Not applicable. No use of SyncNet, AV-align, or any sync metric.

### [Ovi](../models/Ovi.md) ⚠️

Both the metrics and thresholds are fully and explicitly disclosed — the most concrete part of this work's data-side disclosure:
- Detector: SyncNet (Chung & Zisserman, 2016), internally modified to support batch processing at million-video scale.
- Metric 1: offset (audio-visual temporal offset, in frames). Threshold: |offset| ≤ 3. At 24fps this corresponds to a tolerated error of ≤ 125 ms.
- Metric 2: confidence (SyncNet confidence). Threshold: confidence > 1.5.
- Metric 3 (audio-side joint gate): mean volume ≥ −60 dB (average volume no lower than −60 decibels), used to exclude near-silent/very weak audio-track clips.
- Combination logic: the three are combined via AND — all must be satisfied simultaneously to be retained.
[Cross-comparison] This confidence>1.5 gate is moderately-to-strictly set (the common range of SyncNet conf is roughly 0–10+; some works such as UniTalking use >0.9, while talking-head datasets commonly use >3), while |offset|≤3 frames is a relatively strict temporal tolerance. The authors self-describe this as strict criteria.
[Undisclosed] Pass rate, ablation curves for each threshold, and the experimental basis for threshold selection are all unstated [Uncertain].

### [Script-a-Video](../models/Script-a-Video.md) ⚠️

[Data-filtering threshold: none] The paper gives no threshold value of any sync metric for data filtering — no SyncNet confidence gate, no AV-align score gate, no lip-motion-detection gate. This contrasts with works that give explicit numeric gates, such as UniVerse-1 (SyncNet conf > 2.0) and UniTalking (SyncNet conf > 0.9). This work's alignment guarantee comes from schema rules rather than threshold filtering. [Uncertain]
[Evaluation metric: SyncNet (chung2016out, the original "Out of Time" version)] Outputs a sync-confidence score and frame-level temporal offset. Measured values (multi-shot scenario):
- LTX-2-AV (monolithic-prompt baseline): 6.86
- LTX-2-AV-MTSS (prompt swap only): 13.86
- Ours w/o ID: 9.16
- Ours (Full): 9.72
[Key caveat on interpreting these figures] The paper explicitly points out that this automated metric can be deceptive ("the automated A-V Sync metric can be deceptive"): the baseline's 6.86 looks best, but its human-rated A-V score is only 1.18 (the lowest tier on a 1–3 scale), because the baseline generates flat ambient noise — synchronizing a nearly static audio envelope with the picture is trivial (an artifact of information sparsity). After switching to MTSS, the model begins attempting to generate complex dialogue, and the SyncNet score first rises to 13.86 (the deviation widens); the full solution then narrows the temporal gap to 9.72 through architectural improvements, while the human A-V rating reaches 2.26 (the best in the entire table).
This finding has direct practical implications for the data side: using SyncNet score as a data-filtering gate would systematically favor samples with no speech/weak audio content, instead filtering out information-rich dialogue samples. Any pipeline that adopts a SyncNet threshold filter should first stratify by audio activity level before setting thresholds per stratum.
[Other related metrics and figures]
- Shot Boundary Deviation (frames): LTX-2-AV 3.79 → LTX-2-AV-MTSS 3.27 → Ours w/o AV 1.28 → Ours w/o ID 0.38 → Ours (Full) 1.36;
- WER: LTX-2-AV multi-shot 0.84 → LTX-2-AV-MTSS 0.13 → Ours (Full) 0.19 (single-shot baseline 1.64 → 0.78 → 0.23);
- Audio Quality (UTMOS): 4.12–4.18 → 4.60–4.79 → 4.68;
- Reference ID Similarity (ArcFace cosine similarity): single-shot Ours w/o MS 0.62; multi-shot Ours w/o ID has no value, Ours (Full) 0.22, Ours w/o AV 0.20;
- Intra-Shot Subject Consistency (DINOv2 [CLS] inter-frame mean cosine similarity): LTX-2-AV 0.87 (inflated) → LTX-2-AV-MTSS 0.66 → Ours (Full) 0.59.

### [Seedance 2.0 and Seedance 1.5 pro](../models/Seedance_20_Seedance_15_pro.md) ⚠️

[Uncertain] Completely undisclosed. No mention of any automated sync-metric name such as SyncNet, AV-align, or LSE-C/LSE-D in the report, and no filtering threshold value of any kind. Seedance's audio-visual sync evaluation relies entirely on 1–5-point human/expert subjective scoring (SeedVideoBench) rather than automated objective metrics — the report, in its Arena section, even specifically notes that its evaluation approach does not rely on automated metrics such as FVD and CLIPScore. It therefore cannot be benchmarked against disclosures such as UniTalking's "SyncNet conf > 0.9."

### [SkyReels series](../models/SkyReels.md)

SkyReels-V4 gives explicit threshold values, placing it among the more fully disclosed models of its kind:
(1) Sync admission condition: |offset| ≤ 3 ∧ confidence > 1.5 — i.e., the absolute value of the SyncNet-estimated audio-visual temporal offset must not exceed 3 frames, and the sync confidence must exceed 1.5; both conditions must be satisfied simultaneously (conjunction) for the clip to be retained;
(2) Volume gate: requires a minimum mean volume of −60 dB, used to exclude nearly silent clips where SyncNet's results would be unreliable;
(3) Silence gate: VAD (voice activity detection) requires a silence ratio below 0.2.
[Comparative reference] This threshold combination (offset≤3 frames, conf>1.5) is stricter than UniTalking's SyncNet conf>0.9 on the confidence dimension and additionally adds an offset-dimension constraint — a dual-condition "offset + confidence" formulation.
[Undisclosed] The specific weight version of SyncNet (original SyncNet vs. a self-trained version), whether the 3-frame offset at 32FPS (corresponding to roughly 94ms of absolute time tolerance) was an intentional design choice, and what proportion of samples this filter removed. SkyReels-V2 has no such step.

### [Sora 2](../models/Sora_2.md) ⚠️

Completely undisclosed. No name of SyncNet, AV-align, or any in-house sync metric, no confidence threshold value of any kind (cannot be compared with e.g. UniTalking's SyncNet conf>0.9). OpenAI also has not published Sora 2's score on any third-party AV-sync evaluation benchmark. [Uncertain]

### [SpeakerVid-5M](../models/SpeakerVid-5M.md) ⚠️

[Metrics used] SyncNet-family metrics, recorded per-clip across three items: offset (number of audio-visual temporal-offset frames), confidence (sync confidence), and embedding distance (audio-visual embedding distance, the equivalent of LSE-D). The dataset publishes these three as annotation fields rather than using them only for internal filtering.
[State of threshold disclosure] The paper uses the phrase "pre-defined threshold" in multiple places without giving a specific value — the most regrettable disclosure gap in this entry:
  - In the speaking/listening determination, "the SyncNet score difference between the two people exceeds a pre-defined threshold" — the difference threshold is undisclosed.
  - In the non-co-located listening determination, "the SyncNet score falls below a pre-defined threshold" — this lower bound is undisclosed.
  - The five filtering conditions for the HQ / SFT subset (hand blur > 0.5, face blur > 0.7, DOVER > 0.6, motion score > 2, ASR confidence > −1) do not include a SyncNet item, indicating that SyncNet was not used as a hard gate for the HQ subset.
[Comparison with similar works] MOVA explicitly gives a dual threshold of LSE-D ≤ 9.5 and LSE-C ≥ 4.5 and discloses that this filter yields about 2.5M clips; SkyReels-V4 gives SyncNet |offset| ≤ 3 and conf > 1.5. Although SpeakerVid-5M has the deepest dependence on SyncNet among the three (using it for identity binding and state classification, not just filtering), it is the only one of the three that does not disclose specific threshold values — a clear shortcoming in method reproducibility, though its open-sourced curation codebase may contain the actual threshold constants, which could be pursued as a supplementary verification path.
[In-house metric] None; no new sync metric is proposed. [Uncertain]

### [Step-Video-T2V](../models/Step-Video-T2V.md) ⚠️

Not applicable. No use of any audio-visual sync metric; no corresponding threshold. For reference, this entry's sole means of "cross-modal alignment" is visual-text alignment: 8 frames are uniformly sampled from a clip, the cosine similarity between each frame's embedding and the caption text embedding is computed and averaged to obtain a CLIP Score as a text-image alignment label, used to filter mismatched samples — but the specific value of this threshold is likewise undisclosed. [Uncertain]

### [UniTalking](../models/UniTalking.md) ⚠️

[Verification finding on the claim "SyncNet confidence > 0.9" (important)] This survey performed a full-text keyword search of both the HTML and PDF (10 pages) full text of arXiv:2603.01418v1, and the conclusion is: the paper contains no numeric threshold for sync detection whatsoever, the word "SyncNet" does not appear anywhere, and the threshold figure "0.9" does not appear either (the only occurrence of 0.9 in the full text is the AdamW optimizer's β₁=0.9). The complete original-text statement on lip-sync filtering is only "samples exhibiting poor lip-synced alignment (using LipSync)," which neither specifies the concrete model implementation of LipSync nor gives a confidence or distance threshold. Section 4.4 does mention "Further details are provided in the Appendix," but the arXiv v1 PDF in fact contains no appendix content at all. Therefore the claim "SyncNet conf > 0.9" cannot be confirmed in the primary source, and may originate from secondhand paraphrase, a different version, or cross-contamination with unrelated information. [Uncertain]
[Sync-related information actually given in the paper]
- Filtering tools: LightASD (source homogeneity of the voice) + LipSync (lip-audio temporal alignment), neither with a threshold;
- Evaluation metrics: Sync-C (higher is better) and Sync-D (lower is better), i.e., SyncNet-family confidence and distance metrics.
[Evaluation results (Table 1, T2AV task lip sync)]
- UniVerse-1: Sync-C 1.85 / Sync-D 11.97
- OVI: Sync-C 6.56 / Sync-D 8.6
- Sora2: Sync-C 5.35 / Sync-D 7.78
- UniTalking: Sync-C 4.87 / Sync-D 8.05
I.e., UniTalking leads UniVerse-1 by 3.02 points on Sync-C and by 3.92 on Sync-D; it leads OVI by 0.55 on Sync-D but trails on Sync-C.
[The paper's own critique of the Sync-C metric] The authors explicitly point out that OVI's unusually high Sync-C (6.56, even higher than Sora2's 5.35) may reflect a metric bias rather than genuine sync quality — speculating that Sync-C favors "exaggerated mouth movement" over "natural articulation." This is a valuable metric critique: it implies that using SyncNet confidence alone as a filtering threshold would introduce the same bias, potentially systematically eliminating samples with natural, understated mouth movement while retaining samples with exaggerated mouth movement. UniTalking gives no threshold, so it cannot be determined whether it is affected by this.
[Cross-comparison] UniVerse-1 uses SyncNet conf > 2.0 (relatively loose); MOVA uses the two-sided constraint of LSE-D ≤ 9.5 and LSE-C ≥ 4.5 (stricter). UniTalking discloses no threshold at all, making it the least transparent of the three on this dimension.

### [UniVerse-1](../models/UniVerse-1.md) ⚠️

[The single hard threshold] SyncNet confidence score > 2.0, applied to speech clips confirmed by RetinaFace to contain a face — the sole quantitative sync-admission gate in the training data.
[Threshold strictness assessment] 2.0 is a fairly loose tier. For cross-comparison: MOVA adopts the two-sided constraint LSE-D ≤ 9.5 and LSE-C ≥ 4.5 (LSE-C shares the same origin as SyncNet conf, and its threshold of 4.5 is markedly stricter than 2.0); UniTalking, as mentioned in this survey, uses SyncNet conf > 0.9 (though the basis may differ). UniVerse-1 uses only a one-sided conf threshold with no accompanying LSE-D distance constraint, and sets the gate at 2.0, suggesting it leans toward "preserving data volume" over "guaranteeing sync quality" — given that the final speech subset amounts to only 1,187 hours, raising the threshold further would make it difficult to sustain training.
[Threshold calibration method] The paper does not explain how 2.0 was determined — no manual spot-check description, no threshold ablation experiment. [Uncertain]
[No sync threshold for non-speech data] The 3,074 hours of general data and 3,422 hours of VGGSound/AudioSet data are under no sync-threshold constraint whatsoever.
[Mismatch between evaluation metrics and training-filter metrics] Evaluation uses LSE-C (1.34) and Synchformer AV-A (0.23), while training filtering uses only SyncNet conf — LSE-C's absolute value of 1.34 is on the low side (MOVA-720p reaches 6.593), attributable partly to the loose training-side sync gate and partly to the backbone being only Wan2.1-1.3B.

### [Unison](../models/Unison.md) ⚠️

[Metric used is clear; all thresholds absent] The data-filtering stage uses SyncNet to compute lip-audio alignment within the face bounding box, but the paper gives no threshold value whatsoever — no confidence lower bound (compare UniVerse-1's conf > 2.0, UniTalking's conf > 0.9), no LSE-D distance upper bound (compare MOVA's two-sided constraint LSE-D ≤ 9.5 and LSE-C ≥ 4.5), and it does not even state whether it uses a confidence or distance scale, nor the specific implementation version of SyncNet used (the original Chung & Zisserman version vs. the Wav2Lip team's LSE variant). In the method section the paper cites SyncNet as [Chung16a] (the original SyncNet paper), while in the evaluation section it cites LSE-C/LSE-D as [Prajwal_2020] (Wav2Lip), hinting that the training filter and the evaluation may use different implementations — but the paper does not clarify this. [Uncertain]
[Threshold calibration method] Unstated. No manual spot-check description, no threshold ablation experiment. [Uncertain]
[No sync threshold for non-speech data] Non-face data such as VGGSound is not filtered by any temporal-alignment threshold.
[Complete list and values of evaluation metrics (TI2AV setting)]
- LSE-C (higher is better): 3.30, second to LTX-2's 3.45, better than MOVA 3.24, UniAVGen 2.89, Ovi 2.81, UniVerse-1 2.32;
- LSE-D (lower is better): 7.88, second to LTX-2's 7.62, better than MOVA 7.92, UniAVGen 9.49, Ovi 9.12;
- DeSync / DS (lower is better, Synchformer measuring the absolute temporal offset of onset across modalities): 0.08, the best across the board — better than LTX-2's 0.10, Ovi's 0.12, MOVA's 0.13, UniAVGen's 0.15, UniVerse-1's 0.50.
[A notable metric divergence] Unison tops the field on DS (general event-level temporal alignment) but trails LTX-2 slightly on LSE-C/LSE-D (lip-sync-specific). The user study shows the same pattern (lip sync 1.86, behind LTX-2's 1.74, but leading on both motion-audio alignment 1.92 and speech-sound-effect harmony 1.55). This divergence rather self-consistently reflects the two works' differing routes: LTX-2 wins on lip detail via a 19B video backbone and vastly larger-scale data, while Unison wins on global event alignment and acoustic-level performance via a 5B backbone (nearly 4x smaller) plus cross-modal forcing. The paper itself emphasizes "achieving superior cross-modal synchrony with roughly one-quarter the video-backbone parameters."
[Practical impact of the missing threshold] Since the data-filtering threshold is entirely unknown, Unison's lip-sync data quality cannot be compared horizontally with other works, nor can it be determined whether its slight LSE-C shortfall relative to LTX-2 stems from a looser filter or from the smaller backbone — a substantive reproducibility gap in this work.

### [Veo 3 / Veo 3.1](../models/Veo_3_Veo_31.md) ⚠️

[Uncertain] Completely undisclosed. No mention of SyncNet, AV-align, or any in-house sync metric, and no confidence threshold value whatsoever (compare with UniTalking's disclosed SyncNet conf>0.9).

### [Vidu S1](../models/Vidu_S1.md) ⚠️

Data-filtering stage: the paper only states that pre-filtering is done by "audio-visual synchrony," without giving the sync-detection model used (whether SyncNet) or any threshold value [Uncertain]. In the heuristic rules, "speech-energy ratio too low" likewise gives no specific threshold [Uncertain].
Evaluation stage: explicitly uses Sync-D (a SyncNet-family lip-sync-expert distance metric); Vidu S1 = 7.8470 (HDTF, best across the board); at the same time reports CSIM = 0.9192 (identity preservation, best) and DOVER = 0.5660 (perceptual quality, best).

### [Wan 2.5 / 2.6 / 2.7](../models/Wan.md) ⚠️

The training-data-filtering threshold is entirely undisclosed across the board; there is one comparable figure on the evaluation side.
[Filtering threshold] Wan2.2-S2V only states that Light-ASD is used to "detect and exclude" out-of-sync samples, without giving a confidence threshold, a temporal-offset tolerance window (e.g., |offset|≤3 frames), or any numeric value; Wan 2.1 V2A has only categorical rules ("remove clips with no audio track, remove clips containing speech/human voice"), no numeric threshold; 2.5/2.6/2.7 give none. It therefore cannot be aligned with works that give explicit thresholds, such as MOVA (LSE-D≤9.5 and LSE-C≥4.5), SkyReels-V4 (SyncNet |offset|≤3 ∧ conf>1.5), and UniTalking (SyncNet conf>0.9).
[Evaluation metric] On the EMTD dataset, Wan2.2-S2V uses Sync-C (derived from Chung & Zisserman 2017's SyncNet confidence) as its lip-audio sync quality metric; measured results: Wan-S2V 4.51, compared with HY-Avatar 4.71 (higher), EMO2 4.58 (higher), EchoMimicV2 4.44, FantasyTalking 3.00, MimicMotion 2.68. I.e., on the lip-sync metric alone, Wan-S2V is not the best (trailing slightly behind HunyuanVideo-Avatar and EMO2); its advantage shows up instead in visual quality and identity consistency: FID 15.66 (best), FVD 129.57, SSIM 0.734 (best), PSNR 20.49 (best), CSIM 0.677 (best), HKC 0.435, HKV 0.142, EFID 0.283.
[No sync dimension in Wan-Bench] None of the 14 metrics across the three major dimensions of Wan 2.1's self-built Wan-Bench include any audio-visual sync category — audio-visual sync capability has never entered its main benchmark. 2.5/2.6/2.7 have not published any sync-benchmark scores. [Uncertain]

### [Audio-Visual Generation Evaluation Benchmark Collection](../models/av_benchmarks.md) ⚠️

Specific metrics and quantitative parameters (directly usable as reference parameters for training-data sync filtering):

[AV-SyncBench's perturbation-intensity spectrum — the most valuable threshold-calibration data]
- Global offset: 50–500 ms, five tiers;
- Local jitter: 30–700 ms, three severity tiers;
- Global speed change: 0.8×–1.25×, ten tiers;
- Temporal granularity: 0.64-second chunks; video uniformly at 25 FPS, audio uniformly at 16 kHz.
Key empirical finding: at the 50 ms offset tier, the discrimination accuracy of every SOTA sync model is only about 0.51 (near random guessing). This implies that when using Synchformer/SparseSync-family models to filter training data for sync, their effective resolution floor is above roughly 50 ms — misalignment below this magnitude cannot be reliably detected — a directly instructive finding for setting sync thresholds.
Empirical divergence in model capability: ImageBind has the highest overall accuracy (0.859) on timbre-editing (semantic) tasks but is weaker on temporal-alignment tasks; SparseSync shows the opposite trend; Synchformer and SparseSync excel at temporal-offset detection; CAV-MAE is stronger on local jitter and speed change. Conclusion: no single model can handle both temporal and semantic filtering equally well, so a training pipeline should combine multiple models.

[AVBench hard-negative parameters] Temporal offset 0.2–3.0 seconds (covering the coarse-grained mismatch range); acoustic corruption includes pitch shifting and speed changes; the evaluator's output is normalized via a Yes/No single-token probability ratio into a continuous score (0–1), which naturally supports threshold setting.

[VABench] Desync is based on Synchformer-predicted offset, with a computation window of the first/last 4.8 seconds each; the MLLM macro-score is on a 1–5 scale; stereo Mono Compat = 1 − normalized mono-downmix loss. The absolute threshold used for filtering is undisclosed [Uncertain].

[PhyAVBench] CPRS = ½(cosine_similarity + 1), normalized to [0,1], with 1.0 being perfect alignment, 0.5 orthogonal, and 0.0 inverse alignment; each pairing group requires at least N≥20 ground-truth samples averaged to suppress noise (empirical mean of 17).

None of the five use the classic SyncNet/LSE-D/LSE-C metric family [uncertain why not adopted], instead generally shifting toward more modern representation models such as Synchformer / CAV-MAE-Sync / ImageBind — itself a signal of the 2026 shift in the sync-detection technology stack.

### [Video Caption Model Ecosystem](../models/caption_models.md) ⚠️

[Uncertain] Captioners themselves set no sync-metric threshold; this field records comparable thresholds in the ecosystem related to captioning:
[Caption-quality thresholds (native to this ecosystem)]
· AVoCaDO: GPT-4.1 scores synthesis completeness on a 1–5 scale, with a retention threshold of ≥4; dialogue-content similarity uses edit distance + dynamic-programming alignment, threshold 0.6; caption length cap of 4096 tokens (penalized beyond that by ℛ_L), with an optimal range of 2048–4096 tokens.
· Foley-Omni: Bandit's acoustic post-hoc verification uses an energy-gating threshold of −35 dB (used to correct visual hallucinations in Gemini annotations, i.e., objects seen in the picture but actually silent).
· Panda-70M: UMT retrieval-based hard-negative weighting (unselected captions get a weight of 1.0, other in-batch negatives get a weight of 0.01).
[Downstream generative models' sync thresholds (for reference, not captioner thresholds)] MOVA requires LSE-D ≤ 9.5 and LSE-C ≥ 4.5; SkyReels-V4 requires SyncNet |offset| ≤ 3 and conf > 1.5; Foley-Omni uses Sync score ≥ 0.2, IB ≥ 0.3, AudioBox ≥ 0.6; OmniCustom uses a SyncNet dual threshold. All of these are executed outside the captioner.
[Ecosystem gap] No work has published a "joint threshold table for caption quality and audio-visual sync" — i.e., there is no public protocol for "how the caption of a sample should be handled (discard/downgrade/flag) when its sync quality falls below some threshold." This is a clear methodological gap in the AV data pipeline.

### [Geometric / Structured Annotation Dataset Collection](../models/geometric_datasets.md)

Not applicable — none of the four use audio-visual sync metrics or thresholds such as SyncNet / Synchformer / LSE-D / LSE-C. The quantitative thresholds in their corresponding positions instead appear on the geometry and image-quality dimensions: SpatialVID removes samples with aesthetic score <4.0, requires brightness within [20,140], removes samples with OCR text area >30%, and requires clip duration 3–15 seconds; SceneScribe-1M requires resolution >1080p, frame rate ≥10fps, duration 5–60 seconds; Action100M requires clips >0.5 seconds, tree nodes >4 seconds; WildWorld's State Alignment uses skeletal-keypoint pixel-distance thresholds of 4/8/16/32 px.

### [Post-Training Data Strategy for Video Generation](../models/post_training_data.md) ⚠️

The anchor paper has no sync metric or threshold [Uncertain]. Cross-referencing the post-training stage, the only metric worth naming is JavisDiT++'s use of Synchformer as a temporal-sync reward model, but as a continuous score participating in normalized ranking when used as a reward model, no threshold is set — this is fundamentally different from the threshold paradigm used at the data-filtering stage (e.g., MOVA's LSE-D≤9.5 and LSE-C≥4.5, SkyReels-V4's SyncNet |offset|≤3 and conf>1.5, UniTalking's SyncNet conf>0.9):
· Data-filtering stage: a hard binary threshold (pass/reject), aimed at ensuring the training material itself is synchronized;
· Post-training reward stage: a continuous score participating in within-group relative comparison, aimed at pushing the model's output toward greater synchrony.
This "threshold → continuous reward" transformation is the key morphological shift of sync quality moving from the data side to the training-signal side, but currently only JavisDiT++ implements it fully. Unison's evaluation system includes LSE-C/LSE-D, and UniVerse-1's Verse-Bench includes LSE-C/AV-A, but both are used only for evaluation and are not fed back into training. [Uncertain]

### [Survey of Combined Mainstream Video Pretraining Datasets: Panda-70M, InternVid, Koala…](../models/pretraining_datasets.md)

Not applicable. None of the seven use SyncNet, Synchformer, LSE-C/LSE-D, AV-align, or any audio-visual sync metric or threshold.

## Separate handling of temporal sync vs. semantic sync (temporal alignment and content-semantic matching as two independent filtering conditions)

`temporal_vs_semantic_sync` · Detail level: brief

### [Allegro](../models/Allegro.md)

Not applicable (no audio). If analogized to the pure-visual side, the design in Allegro closest to this is splitting "temporal consistency" (LPIPS ≥0.05, UniMatch [1.0,100]) and "semantic matching" (CLIP similarity ≥0.17/0.20) into two independent filtering conditions applied at different pipeline stages — this separation of "temporal quality" filtering from "image-text semantic consistency" filtering is methodologically isomorphic to the separate handling of temporal sync/semantic sync in AV models.

### [Apollo](../models/Apollo.md)

Apollo explicitly separates temporal sync and semantic sync into two independent filtering conditions, each paired with a dedicated model — the clearest point of design intent in its data pipeline:
- Temporal dimension: Synchformer, answering "do the sound and picture line up on the time axis" (does the lip motion keep pace with the speech, does a footstep sound land on the foot-strike frame).
- Semantic dimension: ImageBind, answering "are the sound and picture describing the same thing" (if the picture shows ocean waves but the sound is keyboard typing, there may be no temporal conflict but a complete semantic mismatch).
The two are applied side by side, jointly constituting the judgment of "high synchronization." This recognition that "temporal ≠ semantic, each requires its own gate" is designed precisely to address the two failure modes the paper identifies at the outset — asynchrony (a timing problem) and misalignment/lip-speech mismatch (a semantic and lip-shape problem).
Notably, this separation is symmetrically reused on the evaluation side: AV-A (Synchformer-family, temporal) and IB score (ImageBind, semantic) are reported as two independent metrics side by side — the data filter and the evaluation metric share the same origin, forming a closed loop of "evaluate with what you filter with." This both ensures consistency between the training objective and evaluation, and carries the potential risk of metric self-validation (a model trained on data filtered by Synchformer will naturally do well on Synchformer-family metrics) — the paper does not discuss this.

### [CineDance / CineDance-1M](../models/CineDance.md)

This work makes a clear separation between temporal sync and semantic sync at the metric-design level, one of the more lucid points in its methodology:
[Temporal sync] Handled by SyncNet, outputting Sync-C (confidence) and Sync-D (offset distance), characterizing the frame-level temporal alignment precision between lip motion and speech — meaningful only in shots where a character is speaking.
[Semantic sync] Handled by ImageBind (IB-Score / IB-A Score), characterizing the semantic-matching degree between picture content and audio content in the cross-modal embedding space — applicable to non-speech scenarios such as music, ambient sound, and sound effects, and not requiring frame-level alignment.
[The two are kept side by side rather than merged] They are stored as two separate items in the data metadata, and reported as two separate items under CineBench's AV Sync dimension, forming two independent criteria. In addition, the IB-A Score under the Prompt Alignment dimension further splits out "audio-to-text-prompt semantic consistency" as its own independent item.
[What is not done] Neither item carries a filtering threshold, so they are not actually used as "two independent filtering conditions" in constructing the dataset — the separation is expressed at the measurement and evaluation level, not at the filtering level.

### [CogVideoX](../models/CogVideoX.md) ⚠️

Not applicable on the video side. The public description of CogSound conceptually distinguishes two levels — chunk-wise temporal-alignment cross-attention handles "consistency at the temporal level," while GLM-4V-based semantic/emotion understanding handles "matching at the semantic level"; the official statement is "ensures consistency between audio and video at both temporal and semantic levels," which can be seen as a separation of temporal alignment from content-semantic matching. But this is reflected in the model's architectural design, not as two independent data-filtering conditions [whether the data side is handled separately is uncertain].

### [Cosmos-Predict2.5](../models/Cosmos-Predict25.md)

Not applicable (no audio dimension). If this concept is transferred to the pure-visual context, the pipeline does indeed split "temporal/structural correctness" and "semantic/content correctness" into two independent filtering stages: the temporal/structural side is handled by the semantic artifacts filter (VTSS-like), specifically removing poor transitions, picture-in-picture, and other structural anomalies; the semantic/content side is handled by the content-type classifier and the VLM filter, judging content category and whether it contains objectionable semantic issues. In addition, "excluding native transitions" at the shot-segmentation stage is likewise a purely temporal-level constraint. This separation is methodologically isomorphic to the "temporal sync vs. semantic matching" dichotomy in AV models, though the paper makes no such argument.

### [Data-Juicer 2.0](../models/Data-Juicer.md) ⚠️

[Uncertain] Data-Juicer does not address separate handling of temporal sync and semantic sync, since it has no audio-visual sync-detection capability (see av_sync_detection).
[An analogous design philosophy does exist] DJ's overall architectural stance is precisely "one independent operator and one independent threshold per dimension, no weighted merging" — all Filter operators make single-metric independent judgments that are then chained in series; no combined multi-metric score is ever provided. This shares its origin methodologically with Foley-Omni's approach of splitting ImageBind (semantic) and Synchformer (temporal) into two independent thresholds: both hold that different kinds of errors should be gated independently by different criteria, to avoid a high score on one dimension masking a low score on another. So if audio-visual sync capability were extended onto DJ, it would naturally fall into the form of "a semantic operator plus a separate temporal operator" — no architectural obstacle exists.
[One existing instance of separation] On the visual-text axis, DJ does split "semantic matching" (video_frames_text_similarity_filter, CLIP similarity) and "content attributes" (video_tagging_from_frames_filter, tag matching) into two independent operators rather than merging them into a single composite alignment score — this can be seen as the same design principle manifesting on another modality axis.

### [Foley-Omni](../models/Foley-Omni.md)

Explicitly performs separate handling — a refined design point in this pipeline's alignment dimension.
Table 7's Alignment dimension lists two independent metrics side by side, with two independent thresholds:
  · ImageBind score ≥0.3 handles [semantic consistency] — judging whether the audio content and visual content describe the same thing (picture is a dog, audio track is barking vs. picture is a dog, audio track is unrelated background music). ImageBind computes cosine similarity through a unified cross-modal embedding space, capturing "content matching."
  · Synchformer sync score ≥0.2 handles [temporal sync] — judging whether audio events and visual events line up on the time axis (does the impact on screen occur in the same frame as the impact sound). Synchformer is trained specifically for temporal-alignment detection, capturing "time alignment."
[Why they must be separated] These two categories of error occur independently in real data and are of a different nature: a video with post-added scoring may be semantically consistent (a sad scene paired with sad music) but completely misaligned in time; voice-over/narration may be temporally aligned (matched to editing rhythm) but semantically mismatched with the picture; audio-visual misaligned mis-cut footage is a case where semantics are correct but timing is wrong. A single composite score would let these three problem types mask one another — a high semantic score could compensate for a low temporal score, letting bad samples slip through. Setting two independent hard gates requires a sample to qualify on both dimensions.
[Corroborated by the original text] The filtering condition's wording lists the two as two independent rejection reasons: "weak audiovisual semantic consistency, or unreliable synchronization" — connected by "or," confirming two independent elimination conditions.
[Extended to modeling and evaluation] This separation runs through the entire pipeline: on the model side, CLIP features carry semantics and Synchformer features (the Z_sync pathway) carry timing, injected via two separate pathways; on the evaluation side, the IB score measures semantic consistency and DeSync measures temporal alignment, reported as two separate metrics. Cleaning, modeling, and evaluation all maintain the same semantic/temporal dichotomy — a fairly self-consistent methodology.

### [Goku](../models/Goku.md)

Not applicable (no audio-visual modality). If Goku's "temporal vs. semantic" separation is forcibly analogized on the visual side, an approximate structure can be observed: the shot-segmentation stage chains two mutually independent conditions — **PySceneDetect** (pixel/histogram-level temporal-jump detection) and **DINOv2 feature cosine similarity** (semantic-level content-consistency determination) — the former governs "is there a hard cut," the latter governs "is the content the same thing." This is isomorphic to the idea of splitting "temporal alignment" and "content semantic matching" into two independent filtering conditions in audio-visual models, and can serve as an indirect methodological reference.

### [Hailuo / MiniMax Video](../models/Hailuo.md)

Not applicable. There is no issue of separate handling of temporal sync and semantic sync.

### [HunyuanVideo-Foley](../models/HunyuanVideo-Foley.md)

This work explicitly treats temporal sync and semantic sync as two orthogonal dimensions handled separately — the clearest and most commendable point in its data-pipeline design:
[Explicit two-dimensional separation] The paper's original wording is "leverage ImageBind and AV-align to address the semantic and temporal alignments" — two tools, two dimensions, a one-to-one correspondence with an unambiguous division of labor:
- Semantic dimension (ImageBind): answers "does the audio content match the picture content" — a judgment independent of timing; even if the whole audio track is shifted by several seconds, semantic similarity barely changes;
- Temporal dimension (AV-align): answers "are the sound events and picture events aligned on the time axis" — a judgment independent of content semantics; even if the sound's semantics are entirely correct, the sample should still be judged unqualified if there is a systematic delay.
[Why the two must be separated] The two kinds of mismatch have entirely different causes and consequences: semantic mismatch mainly comes from post-added scoring, narration, and replaced audio tracks (a data-source problem); temporal mismatch mainly comes from audio-visual codec delay, editing/track-alignment errors, and cross-device recording desynchronization (a technical problem). A single metric cannot cover both — ImageBind is insensitive to delay, AV-align is insensitive to content semantics; the two detectors' blind spots happen to be complementary. Doing only one would let the other category of contamination flow entirely into the training set.
[Cross-comparison with similar works] MOVA likewise performs this separation (SynchFormer for timing, ImageBind for semantics); UniVerse-1 lacks both (only SyncNet on speech clips, zero alignment detection on non-speech data, semantic dimension entirely absent). This work is, along with MOVA, among the more complete in this respect, and for the V2A task this separation is even more critical than for joint-generation tasks — because the entire learning signal for V2A is the "picture→sound" mapping; if the mapping relationship itself is contaminated, the model will directly learn an incorrect correspondence.
[Continuation on the model side] The two-dimensional separation also has an architectural counterpart: semantic information is carried by SigLIP-2 visual features via joint attention, while temporal information is carried by Synchformer frame-level features via a gated-modulation pathway — the two streams of information enter the model through different channels, echoing the two-dimensional detection on the data side structurally. This consistency of "the model is built the way the data is filtered" is a relatively complete aspect of this work's design.

### [HunyuanVideo](../models/HunyuanVideo.md) ⚠️

Not applicable. No audio, so there is no separate handling of temporal sync and semantic sync. A visual-side analogy: 1.5 splits "basic rule-based filtering (structural defects)" and "visual quality/aesthetic scoring (perceptual quality)" into independent conditions, an instance of the same idea on a single modality. [Uncertain]

### [InstructAV2AV](../models/InstructAV2AV.md) ⚠️

This work performs a substantive separation of temporal sync and semantic matching, but the location of the separation differs from Foley-Omni's — Foley-Omni lists two thresholds side by side within the same filtering stage, whereas this work dispatches the two to different stages of the pipeline, each handled by a different mechanism.

[Temporal sync is handled by a "structural mechanism"]
  · Data-admission layer: TalkNet + Scribe's condition that "the speech must be temporally aligned with a visible on-screen speaker" is a purely temporal criterion.
  · Data-synthesis layer: frame-wise cross-attention injects audio features frame-by-frame into video generation — an architecture-level temporal enforcement.
  · Evaluation layer: Sync-C / Sync-D (vshift=15) and PEAVS measure temporal-alignment precision.

[Semantic matching is handled by "large-model judgment"]
  · Data-admission layer: non-speech clips are required to have "a single clearly dominant sound source" and are given a semantic sound-event label (e.g., dog barking) — in essence a semantic-attribution confirmation of the sound source.
  · Instruction-generation layer: Qwen3-Omni generates instructions based on instance masks and audio-visual context, which inherently requires "this visual entity" and "this audio segment" to be semantically bound (dog ↔ barking) — otherwise the generated joint editing instruction would not hold.
  · Automated-verification layer: among the five dimensions, "audio-video synchronization for cross-modal alignment" is judged by the MLLM, and in practice leans more toward semantic matching (is the picture a cat and the audio track a meow) rather than frame-level timing precision.
  · Evaluation layer: AV-A (ImageBind cross-modal semantic similarity) measures semantic matching.

[Why this dispatch is reasonable] The nature of the two problem types dictates which tool applies: temporal alignment is signal-level, precisely measurable, and can be architecturally enforced at generation time; semantic matching is content-level, requires understanding, and can only be judged by a model. This work hands the signal problem to a signal mechanism and the semantic problem to a large model — a clear division of labor. By comparison, Foley-Omni uses two scoring models (Synchformer for timing, ImageBind for semantics) applied side by side as filters — an implementation of the same idea under a purely filtering paradigm.

[An implicit advantage] Because the target audio is synthesized first and the video is then synthesized conditioned on it, temporal alignment in the data is near-perfect by construction (the audio-visual frame-misalignment problem common in real footage does not occur), giving the model an unusually clean temporal supervision signal during training. This also explains why this work dares to set no sync-score threshold whatsoever on the data side.
[Uncertain] No ablation was done on the necessity of this separation, and no independent quality statistics on semantic matching vs. temporal alignment within the dataset are reported.

### [Other Audio-Visual Joint Generation, 2026](../models/JAVG_2026_misc.md) ⚠️

In this batch of work, awareness of separating "temporal alignment" from "semantic matching" is markedly stronger, with NAVA and ALIVE each offering representative approaches:
[NAVA — three-layer separation on the data side + explicit separation on the architecture side (the clearest in this batch)]
(1) Data side: SyncNet (lip-audio timing), SyncFormer (general-event timing), and ImageBind (cross-modal semantics) operate in parallel — the first two govern timing and the latter governs semantics, an explicit dual-track filter.
(2) Architecture side: NAVA's core argument is built on precisely this separation — it criticizes "unified tri-modal designs" for "conflat[ing] semantic and low-level alignment." Its Align-then-Fuse design is specifically meant to decouple the two: first establish low-level/fine-grained audio-visual correspondence in a dedicated alignment space (native alignment), then inject context (text, speaker embeddings) as semantics via cross-attention (context as external conditioning). This is the only work in this batch that elevates "temporal vs. semantic separation" to the level of a first architectural principle.
[ALIVE — audio-visual relevance as a filtering dimension independent of sync]
Audio-quality filtering explicitly adopts a dual criterion: the first is an audio-quality score (noise), the second is "audio-visual coherence" — independent of TS-TalkNet's temporal-sync detection, dedicated to judging whether the audio content is semantically related to the picture content, and used to collect strongly related samples separately while controlling the proportion of weakly related samples (e.g., BGM). This forms a very clear three-dimensional filtering space: temporal sync (TS-TalkNet) × semantic relevance (MLLM judgment) × audio quality (MLLM scoring). Particularly notable is that weakly related samples are not simply removed but have their proportion "controlled" — acknowledging that non-diegetic audio (BGM, narration) is common in real content and the model needs the ability to generate it, while ensuring it does not dominate the mix to the point of breaking the audio-visual causal relationship. This is more mature than the approach in works such as Ovi, which "only filters by timing and binds semantics implicitly via caption."
[OmniCustom] Handles only timing (SyncNet dual threshold), with no semantic-matching filter — but given the nature of its task (a single talking head, where the audio must be spoken by the person on screen), the risk of semantic mismatch is inherently low, so this is a reasonable omission.
[ITS-JAVG — splits the two into different verifiers on the inference side and demonstrates their conflict]
Among its combination of verifiers, JavisScore measures fine-grained temporal sync, while AVHScore and ImageBind measure semantic consistency — the paper's core empirical finding is precisely that an asymmetric trade-off exists between these two objectives: optimizing temporal sync sacrifices semantic consistency, and vice versa; a single verifier cannot satisfy both. This provides direct quantitative evidence that "temporal and semantic must be handled separately and explicitly balanced," and is the most theoretically valuable contribution in this batch on this question. Its ARW algorithm is essentially an adaptive trade-off between temporal-class and semantic-class rewards.
[CCL] Reports Sync-C/Sync-D (temporal), DeSync (temporal), and IB (ImageBind, semantic) simultaneously in evaluation — the two are distinguished at the metric level, but there is no corresponding separated filter on the data side [Uncertain].
[StreamChar / Baton] Make no such distinction [Uncertain]. Baton's planned tokens attempt to encode a unified semantic blueprint, while RS-RoPE handles temporal injection — an implicit division of labor in the architecture (semantics via the planner, timing via RoPE), in the same vein as Ovi's approach of "RoPE handles timing, cross-attention handles semantics."

### [Audio-Visual Joint Generation Baseline Collection](../models/JavisDiT_baselines.md) ⚠️

The JavisDiT series has the clearest and most theoretically deep treatment of this dimension in this collection; the rest handle the separation more weakly:
[JavisDiT — the layered design itself is an explicit separation of temporal and semantic] The essence of HiST-Sypo's (Hierarchical Spatio-Temporal Synchronized Prior) "layering" is to split alignment into two layers:
- Coarse-grained global prior — corresponding to semantic-level matching (is the overall content and style consistent);
- Fine-grained spatio-temporal prior — corresponding to precise temporal and spatial alignment (when, where, what sound is emitted).
The two priors are estimated separately from the text prompt and then jointly guide the two denoising paths. This is the only design in this collection that explicitly decouples "semantic sync" from "temporal sync" at the model-architecture level.
- JavisBench's evaluation dimensions embody the same separation: among the five major dimensions, "Temporal Composition" and "Spatial Composition" are temporal/spatial dimensions, while "Event Scenario," "Sound Type," and "Video Style" are semantic dimensions.
- The evaluation metrics are also separated: DeSync (Synchformer, purely temporal) vs. AV-IB / AVHScore (ImageBind-family, purely semantic similarity) vs. JavisScore (sliding window + worst 40% of frames, between the two but leaning toward local temporal).
[JavisDiT++] Continues this separation and further solidifies it architecturally: TA-RoPE handles frame-level temporal alignment (temporal) exclusively, while MS-MoE and cross-modal attention handle semantic interaction (semantic); AV-DPO's reward design is likewise separated — Synchformer scores temporal sync, ImageBind scores cross-modal semantic similarity.
[Harmony — the GLDI module is an architecture-level temporal/semantic decoupling] Global-Local Decoupled Interaction explicitly splits interaction into two branches: the global branch handles "style alignment" (semantic level), the local branch handles "temporal precision" (temporal level). This parallels JavisDiT's layered prior and is the second work in this collection to explicitly perform this separation. Data filtering, however, is not separated — there is only one generic "audio-visual consistency scoring model" [Uncertain: whether this model measures temporal or semantic consistency].
[MM-Diffusion / AV-DiT / UniAVGen] Make no explicit separation of temporal and semantic sync [Uncertain]. MM-Diffusion's random-shift attention restricts attention to a temporal neighborhood, implying a preference for temporal locality, but this is not an intentional separation design.
[Methodological value] This collection confirms a trend: post-2025 audio-visual joint generation work generally recognizes that "temporal alignment" and "semantic matching" need to be handled by different mechanisms — JavisDiT uses a layered prior, Harmony uses a decoupled interaction module, Ovi uses RoPE for timing plus cross-attention for semantics — three different roads converging on the same idea.

### [Kling 3.0 / Kling Video 3.0 Omni](../models/Kling_30_Omni.md) ⚠️

[Uncertain, with strong indirect evidence] The team's methodology clearly treats "temporal alignment" and "semantic alignment" as two independent dimensions: Kling-Foley's architecture separately sets up a visual-semantic representation module (handling semantic matching) and an audio-visual-sync module (handling frame-level temporal alignment), and its evaluation likewise lists semantic alignment and temporal alignment separately; on the data side, CLAP is used to compute audio-to-text-label consistency (a semantic condition), while VAD/event-duration rules constrain the temporal dimension (a temporal condition), and it is required that "sound effects must come from an object or action visible in the picture" (a semantic-causality condition). Whether Kling 3.0 Omni continues this dichotomy is not directly disclosed.

### [LTX-2](../models/LTX-2.md) ⚠️

The paper does not distinguish "temporal sync" from "semantic matching" as two independent conditions at the data-filtering level — the only filtering criterion on the data side is "audio information content." But in the methodological narrative and architectural design, the two are explicitly distinguished and mapped to different mechanisms:
(1) Temporal sync: handled by keeping only the RoPE temporal component in cross-modal attention (explicitly stated as "forcing cross-modal attention to focus on temporal synchrony rather than spatial alignment"), targeting lip sync and sub-frame alignment of impact-to-sound-effect events;
(2) Semantic/ambient matching: handled by bidirectional dependency modeling — the paper's core argument is that "lip sync is mainly driven by audio, while the acoustic environment (reverb, foley) is determined by visual context," hence the need to jointly model bidirectional dependency — this is the main argument against cascaded V2A/A2V approaches; at inference time, increasing the cross-modal guidance s_m simultaneously improves both "temporal sync" and "semantic coherence," which the paper states side by side.
Whether there is a corresponding pair of filtering conditions on the data side is undisclosed. [Uncertain]

### [LongCat-Video](../models/LongCat-Video.md) ⚠️

In Avatar 1.5's filtering chain, temporal alignment and semantic matching are indeed split into different steps, though not explicitly argued as "two orthogonal conditions for the same goal": the temporal side consists of lip-sync verification (audio-visual offset detection) and active-speaker detection (determining the time interval attributed to an audio track); the semantic/content side consists of Qwen3-Omni + Qwen3-VL's dual judgment on "is the subject speaking," plus caption-level constraints on objective description of scene/relationship/plot. In the online filtering chain, "audio synchronization" is listed as a first-tier criterion and "text and visual quality" as a subsequent independent tier — structurally reflecting the separation. The base version has no such dimension. [Uncertain whether the team intentionally made this distinction]

### [MOVA](../models/MOVA.md)

Explicitly separated, and an explicitly argued design decision in MOVA's data-filtering process:
- Temporal sync is handled by SynchFormer (DeSync offset ≤ 0.5), characterizing "do the sound and picture event occur at the same moment."
- Semantic sync is handled by ImageBind (IB-Score ≥ 0.2), characterizing "is the sound content semantically related to the picture content."
- The two are independent conditions, but combined via a **loose logical OR gate** rather than AND: "we apply a relaxed logical 'OR' gate between semantic and temporal alignment. A video is retained if it satisfies either IB-Score ≥ 0.2 OR DeSync ≤ 0.5." The paper's stated reason is that the two audio types have fundamentally different alignment properties — semantically relevant ambient sound/atmosphere naturally lacks a sharp temporal onset, so requiring temporal sync would falsely reject all of it; whereas fast action sound effects may score low in the semantic embedding space, so requiring a semantic score would falsely reject them. An OR gate preserves both valuable categories of samples.
- This separation is likewise maintained on the evaluation side: Table 4 splits AV-Align into two separately reported columns, DeSync (temporal) and IB-Score (semantic), and lists Lip Sync (LSE-D/LSE-C) separately as the most fine-grained continuous temporal correspondence.
- Section 6.3 further distinguishes two difficulty tiers of sync tasks: discrete, onset-driven events (e.g., "cutting fruit," "drumming") require aligning only a few salient time points, whereas speech requires continuous, fine-grained correspondence between lip shape and phonemes over long spans — the most demanding category.

### [Merged entry: ① Mochi 1](../models/Mochi_MAGI_Motif.md)

Not applicable (no audio). Mapped onto the pure-visual side, this group does show a clear separation of "temporal quality" and "semantic matching":
· MAGI-1 splits the temporal side (RAFT optical-flow overall/foreground/background three-tier motion-intensity upper/lower bounds, camera-stability optical-flow consistency, slideshow-motion optical-flow divergence) and the semantic side (Transition Detection using CLIP keyframe semantic similarity to judge multi-shot, MLLM high-level filtering for semantic-level re-screening) into independent conditions at different pipeline tiers.
· Motif-Video 2B's separation is more thorough and complementary: the low-level temporal signal is UniMatch optical-flow two-sided trimming, the semantic signal is the VLM's action=Dynamic tag, and the two are applied simultaneously in the 720p SFT stage — the former governs "did it move at the pixel level," the latter governs "does it count as a dynamic action at the semantic level." Similarly, SigLIP similarity is used for temporal-continuity judgment (stitch merging), while the VLM's multi_scene tag is used for semantic-level multi-shot judgment — again a "temporal vs. semantic" dual-track backstop.
This "low-level statistics + large-model semantics" dual-track filtering structure is methodologically isomorphic to the separate handling of temporal sync and semantic sync in AV models.

### [Movie Gen](../models/Movie_Gen.md)

Conceptually makes a clean three-layer separation, but in data filtering uses the same combination of CAVTP score + AED label to approximate the distinction, without training two separate temporal/semantic scoring models:
· Conceptual layer: the paper splits audio-visual relationships into three layers — "on-screen (diegetic) (temporal-deterministic correspondence, the strongest temporal constraint)," "on-screen/off-screen mix (environmental plausibility and event causal order, semantic + weak temporal)," and "non-diegetic (emotional and narrative semantics, no temporal constraint)" — with the demands on model capability increasing across the three.
· Data layer: the diegetic/non-diegetic split is essentially a split of "presence or absence of temporal-causal constraint," implemented via the CAVTP threshold; the audio-type split (voice/music/sound) carries the semantic-content dimension. The two cross to form Table 23's six-cell classification, effectively managing "temporal alignability" and "semantic category" as two orthogonal axes for data mixture.
· Evaluation layer separation is the most thorough: diegetic sound correctness (should this sound be occurring at all — semantic matching) and diegetic sound synchronization (is its timing accurate — temporal alignment) are split into two independent human-evaluation dimensions; the non-diegetic side is likewise split into music mood alignment (emotional semantics) and music motion/scene alignment (alignment with action/scene/cut points).

### [NVIDIA NeMo Curator](../models/NeMo_Curator.md)

Not applicable. The framework does not handle audio-visual relationships, so there is no separate handling of temporal sync and semantic matching.
If forcibly analogized, its video side does have a division of labor between "structural metrics" and "semantic metrics": motion-vector score and DOVER image-quality score are structural/signal-level criteria, handled by lightweight statistics and small models; video-type taxonomy and text-overlay detection are semantic-level criteria, handled by embeddings + an MLP; caption generation is a semantic-description layer handled by a VLM. This cost-tiering approach of "assigning models of different scale to criteria of different nature" could be transferred to the design of AV sync filtering (e.g., first coarsely screening temporal alignment with lightweight energy-envelope correlation, then using a large model for semantic-matching re-verification), but the framework itself provides no such implementation.

### [OmniHuman Dataset + OHBench](../models/OmniHuman.md) ⚠️

OmniHuman makes a fairly clear separation of the temporal and semantic dimensions, reflected both on the data side and the evaluation side:
[Data side: separation across three layers]
- Temporal layer (frame-level alignment): SyncNet's temporal offset, a hard constraint of ±3 frames. Governs "is the moment of speaking correct";
- Attribution layer (source identity): greedy matching of SyncNet responses + ArcFace similarity of 0.55. Governs "is this voice produced by this person in the picture" — a layer absent in most works (UniTalking's LightASD does something similar but only as a binary filter, whereas OmniHuman achieves per-segment assignment);
- Semantic layer (content plausibility): Gemini-3 assesses background plausibility and interaction plausibility. Governs "does this scene and this interaction make sense semantically/physically."
The criteria for the three layers are entirely independent: offset is signal-level, attribution is matching-level, semantics is understanding-level, each handled by a different kind of model (a dedicated sync model / a matching algorithm / a large model), so they do not mask one another. This layering has one more layer than UniTalking's two layers (LightASD attribution + LipSync timing) — an added semantic judgment.
[Evaluation side: likewise separated]
- Temporal sync: V-A (ImageBind, audio-visual event temporal offset) and Lip-Sync (SyncNet, lip level);
- Semantic matching: T-A (CLAP, semantic similarity between audio and text) — measures whether the generated sound semantically matches the text description, unrelated to timing;
- Interactive semantics: IN (interaction naturalness), LR (listener realism), ES (emotion similarity), CN (contact naturalness), and other Gemini-3 subjective dimensions, measuring semantic/behavioral coordination among multiple subjects.
[A notable semantic dimension: emotional coordination] ES (Emotion Similarity) assesses "coordinated emotional expression" between two people — a form of cross-subject semantic consistency that is neither temporal alignment nor a single-subject attribute, unprecedented in similar benchmarks.
[Limitation] Whether the semantic-layer judgment on the data side (Gemini-3 background plausibility/interaction assessment) is used as a hard filtering condition or merely as an annotation field is unstated, and no scoring threshold is given. [Uncertain]

### [Open-Sora series](../models/Open-Sora.md)

Not applicable. No audio modality. (If forcibly analogized, visual-text semantic matching is handled by Open-Sora 1.x's CLIP matching score, but this is unrelated to audio-visual sync.)

### [Ovi](../models/Ovi.md) ⚠️

At the data-filtering level, Ovi essentially handles only "temporal sync"; semantic matching is left mainly to captioning and the training mechanism, and the two are not designed as two parallel independent filtering conditions.
[Temporal sync (has explicit filtering)] SyncNet's offset and confidence dual thresholds, specifically constraining the temporal alignment of speech and mouth movement; an average-audio-volume gate excludes invalid audio tracks.
[Semantic sync (no explicit filtering)] No cross-modal semantic-similarity scoring such as CLAP/AV-CLIP is used to remove "content-mismatched audio-visual" samples (e.g., a beach scene with an unrelated voice-over/post-added score). Semantic consistency instead relies on two indirect paths: (1) at the captioning stage, the MLLM watches the picture and listens to the audio track simultaneously, writing both contents into the same description, thereby forcing semantic binding at the text-condition level; (2) during training, bidirectional cross-modal attention lets the two towers serve as each other's semantic context.
[Potential gap] Samples such as voice-over, post-added scoring, or dubbing/re-recording — where timing may be aligned but semantics has no causal relationship to the picture — are difficult to fully remove using SyncNet alone (SyncNet's confidence is naturally low on clips with no visible mouth, providing partial indirect filtering); the paper does not discuss this risk. [Uncertain]
[Architecture-level temporal/semantic division of labor] Notably, Ovi does explicitly divide labor between the two on the model side: timing is handled by scaled-RoPE (a frequency scaling of 31/157≈0.197), semantics by bidirectional cross-attention — the paper's abstract states this as "blockwise exchange of timing (via scaled-RoPE embeddings) and semantics (through bidirectional cross-attention)." This "temporal and semantic separation" idea is reflected in the architecture rather than in data filtering.

### [Script-a-Video](../models/Script-a-Video.md)

MTSS structurally separates temporal sync and semantic sync completely — a direct manifestation of the Relational Grounding dual-link design:
[Temporal-alignment channel: temporal links] Handled by three layers of timestamps — shot-level time_range, event-level time_range, and intra-description/micro-level timestamps embedded in the description, plus shared anchors. This channel only answers "when," and all information consists of numeric intervals amenable to interval arithmetic.
[Semantic-alignment channel: identity links] Handled by a centralized Reference entity repository — a shot's references_in_shot and an event's speaker both point into the same ID space. This channel only answers "who/what," and all information consists of symbolic references.
[A third implicit channel: causal alignment] The Event stream's admission principle that "a sound effect must be produced by a subject visible in the picture" in effect encodes the causal/source relationship between sound and picture — neither purely temporal nor purely identity, but "which thing in the picture produced this sound." A shot's active_events field associates a shot with concurrent events, also carrying this meaning.
[The value of separating the three channels] The paper's core argument rests on exactly this: monolithic captions blend WHO, WHERE, and WHEN into the same passage of text, forcing a downstream model to first perform identity resolution and temporal disambiguation before it can reason; MTSS disambiguates all three in advance, letting the model focus directly on logical inference. This explains why MTSS's gains on reasoning-type benchmarks (Daily-Omni, WorldSense) (Qwen3-Omni +127%) far exceed its gains on description-type benchmarks.
[At the filtering-condition level] The paper does not use the two as two independent data-filtering conditions (since no filtering conditions are disclosed at all) — the separation is expressed at the representation layer rather than the filtering layer. This is a different path from the traditional practice of "temporal sync and semantic matching as two independent filtering gates" — MTSS converts these two dimensions from "criteria used at filtering time" into "fields recorded in annotation."

### [Seedance 2.0 and Seedance 1.5 pro](../models/Seedance_20_Seedance_15_pro.md) ⚠️

The report does not explicitly split these two conditions at the data-filtering level, but the evaluation system does complete this split, which can be seen as a projection of its data-quality philosophy: SeedVideoBench 1.5/2.0 defines "Audio-Visual Sync" as a purely temporal-alignment dimension (temporal alignment of lip motion-speech and sound effect-visual event), while separately setting up "Audio Prompt Following" as a content-semantic-matching dimension (assessing whether the voice, dialogue, and sound effects faithfully follow the user's instruction and expected semantics, and whether there is semantic drift — typical failure modes including missing a specified sound effect, inaccurate language or dialect, and "speech present but with no corresponding lip movement" as an audio-visual mismatch); in addition, "Audio Quality" and "Audio Expressiveness" are set up as two further independent dimensions. This four-way split strongly suggests that the data side likewise gates by two independent conditions of temporal alignment and semantic matching. [Uncertain: whether data filtering actually follows this split]

### [SkyReels series](../models/SkyReels.md) ⚠️

SkyReels-V4 achieves a de facto division of labor between the two on the data side, but the paper does not describe them as two parallel filtering conditions:
(1) Temporal sync: handled by SyncNet's |offset|≤3 condition, purely judging the degree of temporal-axis alignment;
(2) Semantic/content matching: partially handled by SyncNet's confidence>1.5 condition (low confidence implies the audio-visual content itself is mismatched, as in voice-over or background-music scenes), and otherwise handled by the four categories of audio tokens in structured captions — the caption writes "what is happening in the picture" and "what is heard" into the same text, so semantic matching is guaranteed via the text condition rather than an independent filter;
(3) Architecture side: per-block bidirectional cross-attention provides continuous alignment, and RoPE temporal-dimension scaling unifies the temporal scale of token rates across the two modalities.
The paper sets up no independent "audio-visual semantic consistency" discriminator model (e.g., an omni-modal LLM judging whether the audio track and picture describe the same thing), nor does it explain how sound-effect/music-type samples (with no lip information, where SyncNet is inapplicable) are semantically matching-verified — a visible gap in this pipeline. [Uncertain]

### [Sora 2](../models/Sora_2.md) ⚠️

Completely undisclosed. There is no information on whether "temporal alignment" and "content semantic matching" are split into two independent filtering conditions. The capability description shows the model possesses both temporal-alignment capability (frame-by-frame lip-speech alignment, sound effects aligned to collision moments) and semantic-matching capability (ambient sound matching the scene context, music mood matching the picture's tone) simultaneously, hinting that the data side may involve separate handling — but this is pure inference. [Uncertain]

### [SpeakerVid-5M](../models/SpeakerVid-5M.md)

SpeakerVid-5M handles only temporal sync, not semantic sync; the two are not separated into independent filtering conditions:
[Temporal sync] Fully covered by SyncNet — offset directly measures the number of audio-visual time-misalignment frames, while confidence and embedding distance measure the frame-level correspondence strength between lip shape and phonemes. This is the dataset's sole alignment dimension, running through identity binding, state determination, and quality annotation.
[Semantic sync (content-level audio-visual semantic matching)] Entirely absent. No use of ImageBind, a Synchformer semantic branch, CLAP, or any cross-modal semantic-similarity model to judge whether "the audio content semantically matches the picture content."
[Why this absence is reasonable] The dataset consists of pure speech, close-up character scenes; the audio content is exactly the speech of the person in the picture, so the semantic-matching relationship is naturally guaranteed by SyncNet's lip-audio correspondence — as long as the lip shape matches, the semantics necessarily match too. The main value of semantic-sync filtering lies in removing mismatches such as "picture is ocean waves, audio is a piano piece" in general audio-visual data, and such mismatches do not occur in dyadic talking scenarios.
[Comparison with MOVA] MOVA explicitly separates temporal alignment (SynchFormer) and semantic alignment (ImageBind, IB-Score) into two independent filtering conditions, because its data covers foley, music, ambient sound, and other general audio-visual content; SpeakerVid-5M has no such need because its scenario is vertical. This is a methodological difference driven by dataset positioning, not an oversight.

### [Step-Video-T2V](../models/Step-Video-T2V.md) ⚠️

Not applicable (no audio). A structural analogy exists on the pure-visual side: the pipeline sets "temporal/motion dimension" (stage 3's three Farneback optical-flow motion scores) and "semantic matching dimension" (stage 6's CLIP Score image-text alignment) as two mutually independent filtering conditions, respectively handling "is the motion right" and "does it depict what is described" — isomorphic to the idea of splitting AV models' temporal sync and semantic matching into two independent gates, serving as a weak methodological reference. [Uncertain]

### [UniTalking](../models/UniTalking.md) ⚠️

UniTalking is among the few works in this survey that genuinely separates "semantic/source alignment" from "temporal alignment" into two independent conditions at the data-filtering level, even though the paper itself does not use this terminology:
[Semantic/source dimension: LightASD] Determines "whether the speech in the audio track is produced by the person in the picture." This is essentially a content-level homology judgment, answering "is the audio and picture the same thing." It removes voice-over, narration, post-added dubbing, and background voices — samples whose audio and picture may be entirely unrelated in timing, a semantic-level mismatch.
[Temporal dimension: LipSync] After confirming homology, determines whether lip motion and speech are aligned at the frame level. Answers "is the timing accurate." It removes audio-visual offset (timestamp drift, editing misalignment, dubbing substitution).
[The sequencing of the two carries methodological significance] Doing the semantic-homology determination first and the temporal-precision determination second is logically necessary — computing a lip-sync score for a sample whose sound source is not even in the picture is meaningless (a low score would not stem from temporal offset). If ASD were skipped and a SyncNet threshold applied directly, voice-over samples would also be eliminated by a low score, but the reason for elimination would be conflated with genuinely temporally-misaligned samples, and the threshold would need to compromise for both problem types simultaneously. Handling them separately keeps the criterion at each stage purer.
[Comparison with reference works]
- UniVerse-1: has only the temporal dimension (SyncNet conf > 2.0); the semantic dimension is entirely absent, with no removal of voice-over sources — its data sources include large amounts of vlog and tutorial content (where post-added scoring and narration are extremely common), a substantive data risk;
- MOVA: splits Stage 2 alignment evaluation into temporal (SynchFormer) and semantic (ImageBind) as two orthogonal dimensions each with its own threshold — an explicit dual-dimension design;
- UniTalking: the dual-dimension separation holds de facto (ASD + LipSync), but because its domain is extremely narrow (pure speaker audio), its "semantic alignment" is concretized into the specific form of "sound-source homology," rather than MOVA's general audio-visual semantic matching. For the speaker domain, this concretization is appropriate and more targeted.
[Missing] No use of an ImageBind-style general cross-modal embedding similarity for semantic matching; no consistency verification between caption and audio-visual content; neither tier discloses a threshold. [Uncertain]

### [UniVerse-1](../models/UniVerse-1.md) ⚠️

UniVerse-1 does not handle temporal sync and semantic sync as two independent filtering conditions — in fact it covers only a special case of temporal sync (lip shape), while the semantic-sync dimension is entirely absent:
[Temporal sync] Covered only via SyncNet conf > 2.0 for the lip-audio temporal correspondence of speech clips; there is no detection or filtering whatsoever for event-level temporal alignment of non-speech data (e.g., "the moment a door closes corresponding to the impact sound").
[Semantic sync] Entirely absent — no ImageBind-style cross-modal embedding-similarity filter, no determination of "whether the picture content and sound content are semantically matched," and no use of a VLM/LLM to adjudicate audio-visual semantic self-consistency. This means samples with semantic mismatches, such as "picture is a kitchen but the audio track is background music/voice-over dubbing," cannot be identified and removed — given that the data sources include large amounts of vlog and tutorial content (where post-added scoring and narration are extremely common), this is a substantive data risk.
[Comparative reference] MOVA explicitly splits Stage 2 alignment evaluation into temporal (SynchFormer) and semantic (ImageBind) as two orthogonal dimensions each with its own threshold; UniVerse-1 has neither set up independently, and the semantic dimension is entirely absent.
[Implicit substitute mechanism] The paper's argument is that as long as "native synced audio track + co-sourced online annotation window" is guaranteed, temporal and semantic consistency are guaranteed at the data source, requiring no post-hoc filtering. This holds when the native audio track genuinely corresponds to the picture, but does not hold for scored/narrated/post-added-sound-effect content. The paper does not discuss the boundary of this assumption. [Uncertain]

### [Unison](../models/Unison.md) ⚠️

Unison does not treat temporal sync and semantic sync as two independent data-filtering conditions, but does draw a fairly clear distinction between them at the training and evaluation levels:
[Data-filtering level: only temporal, no semantic] The sole filtering mechanism, lip-filtering, is purely temporal in nature (whether lip motion and speech correspond in time). There is no semantic-matching filter at all — no ImageBind-style cross-modal embedding-similarity gate, no determination of "whether the picture content and sound content are semantically consistent," no VLM adjudicating audio-visual semantic self-consistency. Semantically mismatched samples such as "picture is a kitchen but the audio track is post-added scoring" therefore cannot be identified and removed. It should be noted, however, that off-screen voice-over removal functionally covers part of the semantic mismatch for speech-type content — it excludes the most common case of "audio and picture not being co-sourced."
[Training level: semantics handed off to SCG gating] Unison's approach converts semantic consistency from a "filtering problem" into a "conditional-modeling problem" — SCG dynamically determines the mutual influence strength of the speech stream and sound-effect stream based on the global semantic vector of the caption and transcription. That is, the model does not pre-remove semantically mismatched samples but instead learns to determine the appropriate acoustic mixture ratio for the current scene based on text semantics. This is a completely different approach from MOVA's (Synchformer for timing, ImageBind for semantics, two orthogonal thresholds filtering separately).
[Evaluation level: temporal and semantic metrics are listed separately, a clear division] This is a well-designed aspect of Unison's evaluation system:
- Temporal-alignment class: LSE-C / LSE-D (SyncNet, lip timing), DS (Synchformer, absolute temporal offset of onset across modalities);
- Semantic-consistency class: TA (CLAP, audio-text semantic consistency), TV (VideoCLIP-XL-V2, video-text semantic alignment), AV (ImageBind, audio-video semantic similarity).
Unison's AV reaches 0.91, the best across the board for TI2AV (LTX-2 0.89, MOVA 0.88, Ovi 0.87, UniAVGen 0.81, UniVerse-1 0.62), indicating its audio-visual semantic-matching quality does indeed lead — but this is a result of training strategy rather than data filtering.
[Central tension] Unison strictly distinguishes temporal from semantic on the evaluation side, yet on the data side leaves both nearly unguarded (temporal covers only the lips; semantic is not covered at all), relying entirely on the training mechanism to compensate. The results of DS 0.08 and AV 0.91 show this approach works, but it also means its effectiveness depends on the semantic cleanliness of the upstream open-source dataset itself — if switched to noisy, web-scraped data, the same method might not hold. The paper does not discuss this boundary condition. [Uncertain]

### [Veo 3 / Veo 3.1](../models/Veo_3_Veo_31.md) ⚠️

[Uncertain] Undisclosed whether "temporal alignment" and "content semantic matching" are split into two independent filtering conditions. An indirect clue is the design approach of the safety-side multimodal classifier — the official description explicitly notes the need to judge the semantic outcome of a caption-video combination, showing that Google's data side does attend to cross-modal semantic-matching issues, but this mechanism serves safety rather than sync quality.

### [Vidu S1](../models/Vidu_S1.md)

A de facto separation exists, though the paper does not use this terminology: temporal sync is jointly guaranteed by the pre-filtering audio-visual sync metric + VAD speech-segment timestamps + cut points avoiding mid-speech segments; semantic/identity-level audio-visual matching is guaranteed by ASD active-speaker detection and on-screen/off-screen classification (judging "is this voice produced by this person in the picture") — the two are two independent filtering conditions in the pipeline. In addition, the caption's dual-path strategy (visual attributes drawn only from the picture, auditory attributes drawn only from the audio track) is likewise a deliberate decoupling of the two modalities at the semantic-annotation level, to avoid cross-modal hallucination.

### [Wan 2.5 / 2.6 / 2.7](../models/Wan.md) ⚠️

The Wan series distinguishes "temporal alignment" from "semantic/emotional matching" in its design intent, but does not implement the two as independent data-filtering conditions.
[Wan 2.1 V2A's explicit dichotomy] The report opens by splitting the V2A objective into two parallel requirements: (1) ambient sound "must be temporally aligned with the video's visual content"; (2) accompanying music "should accurately reflect the video's emotional tone and contextual setting." I.e., ambient sound/sound effects correspond to temporal sync, and music corresponds to semantic and emotional matching — a fairly clear dichotomy, directly reflected in its three-part caption structure (the ambient-sound field carries temporal events, the music field carries semantic attributes such as style/tempo/melody/instrumentation).
[Division of labor at the mechanism level] The temporal side is handled by the 1D-VAE preserving the time axis + CLIP frame-level feature replication for alignment + a fixed 12-second/48-frame ratio; the semantic side is handled by umT5 text conditioning and music captions; the framework also lets users specify "on-screen/off-screen" and the music's style and presence via text, essentially exposing the semantic condition of "whether the sound source is on screen" explicitly to the user.
[Filtering side] Wan2.2-S2V's Light-ASD's two rules actually imply the same dichotomy — rule (1), "audio not synchronized with the active speaker," is a temporal condition, and rule (2), "no active speaker in the scene," is a semantic/source-attribution condition; but the paper does not describe them as two independent filtering dimensions.
2.5/2.6/2.7 undisclosed. [Uncertain]

### [Audio-Visual Generation Evaluation Benchmark Collection](../models/av_benchmarks.md)

AV-SyncBench is the first systematic implementation of this idea, explicitly claiming to be "the first benchmark to fully separate temporal and semantic evaluation of audio-visual sync" — its decoupled design has direct portability to training-data filtering:

[Mechanism of the decoupling] The key lies in the orthogonality of the perturbation construction — temporal perturbations change only the time axis without changing audio content (global offset, local jitter, global speed change), while semantic perturbations enforce a temporal-invariance constraint, replacing only timbre/sound source while keeping the temporal structure exactly the same (OpenVoice V2 for voice-timbre replacement, a pretrained DDSP for instrument-timbre transfer). Because the two perturbation types are constructed orthogonally, the model's performance on the two axes can be observed independently.

[Empirical value of the decoupling] Measurements show the two axes indeed measure different capabilities: ImageBind is semantically discriminative (0.859) but weak at temporal alignment; SparseSync shows exactly the opposite. This directly proves that "one sync score fits all" would mask a model's capability deficiencies, and also implies that if training-data filtering uses only a single sync score, it will simultaneously miss one of the two categories of bad samples — "timing right but content wrong" and "content right but timing wrong."

[Implications for the training-data pipeline] Sync filtering should be set up as two independent chained conditions: ① a temporal-alignment condition — using Synchformer/SparseSync-family models to detect offset, noting that below 50ms is unreliable; ② a semantic-matching condition — using ImageBind/CLAP-family cross-modal embeddings to detect audio-visual content mismatch (such as mismatched sound effects, post-added background music, or voice-over narration).

[Corresponding designs in other benchmarks] VABench likewise performs a split: Desync (Synchformer temporal offset) and Audio-Visual Align (ImageBind semantic alignment) are listed as two separate independent metrics, and Module 2's Alignment dimension is explicitly defined as a combined judgment of "temporal sync + semantic correspondence." AVBench's AV Consistency dimension merges the two without splitting them, and although its hard negatives fall into two categories — temporal offset (temporal) and speaker-identity mismatch/emotion reversal (semantic) — they are judged by the same evaluator. PhyAVBench's Time and Causality dimension focuses specifically on temporal causality (light-sound-speed-difference far-field delay, transient onset/offset, periodic/non-periodic rhythm consistency), while its remaining five dimensions pertain to acoustic-semantic correctness — also constituting an implicit temporal/semantic separation.

### [Video Caption Model Ecosystem](../models/caption_models.md)

This ecosystem shows a clear methodological evolution in the separate handling of "temporal sync" and "semantic matching":
[Three layers of evidence for the separation]
(1) AVSCap's evaluation-dimension design is itself a separation: Visual/Speech/Music/SFX, four items measuring **semantic completeness** (was the content that should be mentioned actually mentioned), while Synergy alone measures **cross-modal binding and temporal alignment**. The data show the two can diverge severely — AVoCaDO's Speech reaches as high as 70.42 (semantically complete) but Synergy is only 29.13 (binding failure), proving that "clearly heard" does not equal "correctly matched."
(2) Foley-Omni's dual-feature pathway: CLIP provides scene-semantic features, Synchformer provides temporal-sync features, injected via two independent pathways. Ablation shows that removing the Z_sync sync pathway drops IB_V2ST from 0.26 to 0.22, raises FD_VGG from 1.57 to 2.21 (a 41% degradation), and raises WER_V2ST from 7.59 to 12.40 — quantitatively proving that the temporal-feature pathway also makes a substantive contribution to semantic consistency; the two are not fully orthogonal.
(3) MOVA's filtering design chains "cross-modal semantic-consistency checking" (GPT-OSS-120B) and "lip-sync signal detection" (LSE-D/LSE-C thresholds) as two independent conditions.
[Division of labor on the caption side] Semantic matching relies on the captioner and an LLM judge (whether the correct sound source is described, whether there is voice-over/off-screen-audio hallucination); temporal alignment relies on signal-level detection by Synchformer/SyncNet and ASR timestamps (ElevenLabs Scribe, CineDance's sentence-level ASR).
[A cautionary counterexample] Foley-Omni's motivation for using Bandit for acoustic post-hoc verification is precisely that: audio descriptions Gemini generates from the picture may describe sounds that do not actually exist (visual hallucination) — an error that purely semantic judgment cannot self-correct, requiring the introduction of signal-level verification — showing that the semantic chain and the temporal/signal chain serve as mutual backstops, and neither should be neglected.

### [Geometric / Structured Annotation Dataset Collection](../models/geometric_datasets.md)

Not applicable to the audio-visual context. In the context of geometric annotation, the four datasets do show a similar "temporal vs. semantic" separation idea: SpatialVID maintains temporal-level camera trajectory (per-frame pose, MoveDist/RotAngle/TrajTurns) and semantic-level scene description (weather/time-of-day/crowd/lighting/scene-type tags) as two independent sets of annotations in parallel, cross-verifying them against each other at the refinement stage; Action100M explicitly distinguishes temporal localization (when it happens, determined by V-JEPA 2 embedding clustering) from semantic content (what happens, determined by VLM captions), decoupling the two pipelines; WildWorld's WildBench likewise splits Action Following (whether the correct action semantically was executed) and State Alignment (whether spatio-temporal state is precisely aligned) into two independent metrics.

### [Post-Training Data Strategy for Video Generation](../models/post_training_data.md) ⚠️

The anchor paper does not address this [Uncertain]. JavisDiT++'s six-reward design in effect achieves this separation: temporal sync is handled solely by Synchformer, and semantic matching is handled by ImageBind across three separate paths — text-audio, text-video, and audio-video cross-modal — i.e., temporal alignment and content semantic matching are two independent reward terms rather than merged into a single "AV quality score." Its "modality-aware normalized ranking" further ensures that pairing does not mix high-quality audio with low-quality video, equivalent to maintaining within-modality consistency at the preference-construction level. This design is fully aligned in objective with the constraint in Seedance 1.0's manual-annotation protocol (the best sample must not be worse than the worst on any other dimension): preventing self-contradictory preference signals under multi-dimensional rewards. These are the two most transferable engineering lessons from this topic on post-training data construction.

### [Survey of Combined Mainstream Video Pretraining Datasets: Panda-70M, InternVid, Koala…](../models/pretraining_datasets.md)

Not applicable (no audio modality). If analogized on the visual side, the corresponding split would be between "temporal consistency" and "image-text semantic matching": the **temporal side** is handled by shot-cut/transition detection (Panda-70M's ImageBind first/last-frame distance ≤1.0, Koala-36M's CSS+3σ, LVD-2M's 0.5fps ContentDetector threshold of 50) and adjacent-frame consistency (OpenVid-1M's bidirectional removal via CLIP adjacent-frame cosine similarity); the **semantic-matching side** is handled by image-text similarity (Panda-70M's UMT matching_score>0.43, though used only to select the caption rather than for removal; **UltraVideo's VideoCLIP-XL-v2 similarity<0.2 removal is the only one among the seven to use image-text semantic matching as an independent removal condition**). But none of this relates to audio-visual sync.
## Audio quality filtering (SNR, silence detection and silence-ratio thresholds, no-audio-track removal, off-screen source removal, background music separation)

`audio_quality_filtering` · Level of detail: brief

### [Allegro](../models/Allegro.md)

Not applicable. The audio track of the raw video is discarded at the splitting stage, so there is no SNR, silence detection, no-audio-track removal, off-screen source removal, or background music separation.

### [Apollo](../models/Apollo.md) ⚠️

Audio quality filtering is a standalone section in Apollo (the second half of Section 4.1). The dimensions enumerated are fairly comprehensive, but only one threshold is disclosed:
【Removal criteria】Low SNR (signal-to-noise ratio), low MOS (mean opinion score, perceptual audio quality), abnormal clipping, distortion, noise.
【Silence control】Requires "ensuring less than 20% silence" — this is the only publicly disclosed numeric threshold on the audio side, and it also reappears in the video filtering clauses ("discard those videos with ... or over 20% silence"), which indicates the silence-ratio criterion is a strong constraint spanning both the audio and video gates, aimed at preventing the model from learning to produce long stretches of silent output.
【Other requirements】High fidelity, consistent formatting (referring to standardization of sample rate/channels/encoding). The VAE-side spec is 44.1 kHz input.
【Common steps not covered】The paper does not mention removal of no-audio-track samples (although this is logically implied by the pipeline), does not mention removal of off-screen voice/narration sources (i.e., cases where the sound source is not visible on screen — an important interference factor for lip-sync training that works like MOVA specifically handle), and does not mention background music separation (BGM separation, e.g., using Demucs/UVR to strip the accompaniment) — for Apollo, where singing is a first-class subset, whether the accompaniment is separated from the vocals is a key but unanswered question.
【Specific SNR/MOS tools and thresholds】Not disclosed (e.g., whether DNSMOS, UTMOS, Brouhaha, etc. are used). [Uncertain]

### [CineDance / CineDance-1M](../models/CineDance.md) ⚠️

【Metrics used】
  · DNSMOS — a reference-free MOS estimate of speech-signal fidelity, measuring degradation such as noise and distortion;
  · Temporal variance of CLAP embeddings — measures the richness of change in audio content over time, and can indirectly identify monotonous, constant, or near-silent audio tracks (extremely low variance often implies uninformative audio).
【Strategy】Also follows the global "no hard pruning" principle: both metrics are stored as metadata for downstream filtering, and the paper does not give a lower bound on the DNSMOS score or a variance threshold.
【Admission constraint】Source material admission requires "coming with a native audio track," so samples without an audio track are excluded at the collection stage.
【Handling of non-speech intervals】In the ASR-to-Character windowed binding scheme, non-speech intervals are filtered out before binding — this is a functional treatment of silent/non-speech segments, but its purpose is to improve binding accuracy rather than to filter data.
【Not covered】No mention of SNR computation, silence-ratio thresholds, removal of off-screen voice sources, or background music separation (source separation, e.g., Demucs/BS-RoFormer) or other common audio-cleaning measures. As film/TV material, background music overlapping with dialogue is a common phenomenon, and the paper does not state how this is handled. [Uncertain]

### [CogVideoX](../models/CogVideoX.md) ⚠️

[Uncertain] On the video side, CogVideoX does not involve audio; the audio track is discarded directly during training, with no SNR, silence detection, no-audio-track removal, off-screen source removal, or BGM separation. The audio quality filtering strategy on the CogSound side (SNR thresholds, silence ratio, vocal separation, etc.) is not disclosed.

### [Cosmos-Predict2.5](../models/Cosmos-Predict25.md)

Not applicable. The pipeline performs no audio-track processing whatsoever: no SNR estimation, no silence detection or silence-ratio thresholds, no removal of no-audio-track samples, no off-screen source determination, no background music separation. The description of the GPU transcoding stage also applies only to the visual stream.

### [Data-Juicer 2.0](../models/Data-Juicer.md) ⚠️

Audio quality filtering is the most substantive part of DJ's audio-side capability, but its coverage is markedly narrower than on the video side.
【Existing operators】
  · audio_nmf_snr_filter — "keeps samples whose audio signal-to-noise ratio (SNR) falls within a specified range." The underlying method uses NMF (non-negative matrix factorization) to estimate SNR, decomposing the audio into a "signal component" and a "noise component" and computing their ratio. Compared to simple energy statistics, NMF estimation is more robust for scenarios with structured noise (wind noise, background hum, buzz). This is DJ's only core audio-quality operator.
  · audio_duration_filter — filters by a duration range. Can be used indirectly to remove no-audio-track samples (duration of 0) and overly short audio clips.
  · audio_size_filter — filters by audio file size, which can roughly identify samples with abnormally low bitrate (too small in size) or an abnormal format.
  · audio_ffmpeg_wrapped_mapper — wraps FFmpeg audio filters, and can apply arbitrary repair operations such as denoising, loudness normalization, resampling, and channel processing. This is a "repair" path rather than a "filter" path, consistent with the dual-path design on the video side.
【Missing key capabilities】[Uncertain]
  · No silence-detection operator — there is no way to set a threshold of the form "silence ratio must not exceed X%," nor to locate and remove long silent segments. Only audio_duration_filter can indirectly exclude fully silent audio.
  · No audio-track-presence-detection operator — "whether the video has an audio track" must be judged manually during data preparation.
  · No learned audio-quality-assessment operator — Meta AudioBox Aesthetics, NISQA, DNSMOS, and other reference-free perceptual quality models are not integrated, which lags behind the 2025–2026 trend of audio cleaning shifting from signal metrics to perceptual models (compare Foley-Omni's use of AudioBox Aesthetics ≥0.6).
  · No source-separation operator — there is no way to perform background music separation or vocal extraction, and therefore no way to achieve "strip BGM and keep vocals" or "quality-check by audio-track category."
  · No off-screen voice/narration recognition — video_active_speaker_detect_mapper can determine whether someone in the frame is speaking, which could in theory help identify the case of "speech present but no speaker visible on screen" (off-screen voice), but DJ has not packaged this usage into a ready-made operator or recipe.
【Overall assessment】DJ's audio quality filtering remains at the level of traditional signal processing (SNR, duration, size), sufficient to support cleaning of ASR/TTS-style corpora, but its coverage is insufficient for the fine-grained audio quality control needed for joint audio-video generation (silence ratio, perceptual quality, audio-track classification, determining whether the sound source is on- or off-screen).

### [Foley-Omni](../models/Foley-Omni.md) ⚠️

Audio quality filtering is concentrated in a single metric plus one energy gate, a simple design.
【Main filter】Meta AudioBox Aesthetics score ≥0.6. This is Meta's open-source reference-free audio aesthetics/quality assessment model, which outputs a comprehensive perceptual-level quality score. Choosing a learned perceptual assessment over traditional signal metrics (SNR, THD, spectral flatness) reflects the 2025–2026 evolution in audio-data-cleaning methodology — traditional SNR cannot identify issues such as over-compression, clipping, encoding degradation, or mix imbalance, whereas AudioBox Aesthetics correlates more closely with human perceptual judgment.
【Silence handling】The first removal criterion in the filtering stage is silence. In practice this is likely naturally covered by a low AudioBox Aesthetics score (silent clips score extremely low on aesthetics); the paper does not describe an independent silence detector or a silence-ratio threshold. No-audio-track samples are naturally excluded (in this task, the audio track is a required supervision target). [Uncertain] No energy threshold for silence determination or upper bound on silence-duration ratio is given.
【Energy gating (field-level)】At the acoustic post-verification stage, an RMS energy gate of E(a_c) > −35 dB is applied to each separated stem. This is not clip-level filtering but field-level nulling — failing to meet the threshold only voids that field's annotation, while the clip itself is still kept for training on other fields. This granularity is important: a clip with only speech and no music is still a valid VisualTTS/V2ST training sample and should not be discarded wholesale.
【Background music separation】Bandit (a cinematic audio source-separation model) is used to perform three-way speech/effects/music separation, but its purpose is to verify annotations rather than to clean the audio track — the separation result is not used as a training target, and the model still learns to generate the full mixed audio track. This differs from some V2A work that "separates out the background music before training" — Foley-Omni instead requires the model to be able to generate a complete soundtrack that includes music.
【Off-screen source removal】[Uncertain] The paper does not describe a dedicated mechanism for identifying and removing off-screen sound, narration, or voice-over. The ImageBind ≥0.3 semantic-consistency gate can partly filter out narration unrelated to the visuals, but it is not a purpose-built design for this.

### [Goku](../models/Goku.md)

Not applicable. There is no audio quality filtering step anywhere in the data pipeline: no SNR determination, no silence detection or silence-ratio thresholds, no removal of no-audio-track samples, no removal of off-screen voice/narration sources, no background music separation (e.g., Demucs/BS-RoFormer). The training data takes the form of pure "video-text pairs," with the audio track entirely ignored during data construction.

### [Hailuo / MiniMax Video](../models/Hailuo.md) ⚠️

Not applicable + [Uncertain]. There is no audio modeling objective, and it is not officially disclosed whether the training data retains audio tracks; if the audio track is discarded at the splitting stage, then there is no SNR, silence-ratio, off-screen-source-removal, background-music-separation, or any other audio quality filtering step at all. No first-hand information is available.

### [HunyuanVideo-Foley](../models/HunyuanVideo-Foley.md) ⚠️

Audio quality filtering is by far the central focus of this work's cleaning pipeline, comprising four clearly layered gates:
【1. No-audio-track removal】The very first step of the pipeline removes videos that do not contain an audio stream — a one-vote veto. All training data therefore contain a native, synchronized audio track.
【2. Silence-ratio threshold (a proportional criterion)】The silence ratio of every 8-second clip is computed, and clips exceeding 80% are discarded. The distinctive design point is the use of a continuous ratio metric rather than a binary "contains silence or not" judgment, and the threshold is lenient (tolerating as much as 80% silence) — this leniency is deliberately left as room for the Foley task: a typical Foley training sample is precisely the "long quiet stretch + one brief event sound" shape (e.g., a single door-closing sound in a silent room), and if this were treated the way UniVerse-1 does ("discard as soon as any silence is detected"), the most valuable samples would be mistakenly killed. The 80% threshold is in practice only used to discard clips that are almost entirely silent. The energy threshold and time resolution used for silence determination are not disclosed. [Uncertain]
【3. Bandwidth / effective sample-rate detection】Only samples with an effective sample rate above 32 kHz are retained. The word "effective" is key — spectral analysis confirms that high-frequency energy is actually present, rather than merely reading the declared value from the file header, so it can identify and remove fake high-resolution audio that has been up-sampled from a low-bitrate source. An effective sample rate of 32 kHz corresponds to an audible frequency ceiling of 16 kHz, which is the necessary condition for ensuring sufficient spectral information for high-frequency-rich Foley sounds such as cymbals, glass, metal friction, and cloth friction, and it also directly matches the model's 48 kHz output spec — the bandwidth ceiling of the training data determines the frequency ceiling the model can generate. This criterion is relatively rare among comparable works and is a highlight of this work's audio quality control.
【4. Audio aesthetic score + SNR】The AudioBox-Aesthetics toolkit serves as the main metric (able to output four dimensions: PQ production quality / PC production complexity / CE content enjoyment / CU content usefulness), with SNR as a supplementary metric. No threshold values are given for either, and it is not stated which of the four dimensions are actually used. [Uncertain]
【Audio processing not performed】
- No background music separation (source separation) — detected music is only tagged, not stripped, so the Foley sound effects in the training samples may be mixed together with background music, which risks the model learning the incorrect association that "event sounds are always accompanied by music"; the paper does not discuss this;
- No removal of off-screen voice/narration sources — this relies indirectly on ImageBind semantic alignment for filtering, but narration is often semantically related to the visuals (a voice-over describing what's on screen), which can fool semantic detection;
- No dedicated detection of recording defects such as clipping, distortion, or pops (partially covered by AudioBox PQ);
- No description of loudness normalization.
【Overall assessment】The filtering design on the audio side is clearly more carefully considered than on the visual side, and the bandwidth-detection and proportional silence-threshold steps in particular reflect an understanding of the Foley task's characteristics. The main shortcomings are undisclosed thresholds and the absence of source separation.

### [HunyuanVideo](../models/HunyuanVideo.md) ⚠️

Not applicable. No audio-track processing, no SNR/silence-detection/background-music-separation, etc. [Uncertain]

### [InstructAV2AV](../models/InstructAV2AV.md) ⚠️

Audio quality control is distributed across two layers: admission filtering and synthesis protection.

【Admission filtering (applied to real-world material)】
  · Silence detection: PyDub removes silent segments below −45 dBFS. This is the only precise numeric threshold given anywhere in the paper. −45 dBFS is a relatively lenient bar (roughly "almost completely silent"), intended to remove tracks that are audio-free or purely silent, rather than to perform fine-grained loudness control. [Uncertain] It is not stated whether this is an overall average level or a segment-by-segment judgment, and no upper bound on the silence ratio is given (e.g., a constraint like "discard if silent duration exceeds 30%").
  · Audio quality assessment: Audiobox-Aesthetics (Tjandra et al., 2025, Meta's reference-free audio aesthetics assessment model) removes low-quality audio. This uses the same tool as Foley-Omni, and belongs to the new 2025–2026 paradigm of audio-data cleaning (replacing traditional signal metrics like SNR/THD with learned perceptual-quality assessment, which can identify problems traditional metrics cannot detect, such as over-compression, clipping, and encoding degradation). [Uncertain] No threshold value is given (Foley-Omni's corresponding threshold is ≥0.6, which can serve as a reference order of magnitude).
  · Sound-source clarity: For non-speech clips, those with ambiguous sound sources are removed, requiring that a single dominant sound source can be clearly identified. This is a filtering condition serving downstream separability, and is uncommon in general-purpose audio cleaning.
  · Removal of off-screen sources: speech clips are required to have "the speech temporally aligned with a visible on-screen speaker," which in effect removes narration, voice-over, off-screen voice, and post-production dubbing — in this work this is a hard admission condition rather than a soft filter, more thorough than in most works.

【Handling of background music/ambient sound — preservation rather than removal】
  This is an important difference between this work and most V2A works: background sound is not a disturbance to be removed, but a "content-preservation" supervisory signal to be precisely retained. SAM-Audio (Shi et al., 2025) semantically separates the target entity's sound from the original mixed audio track, and the edited new sound is then seamlessly mixed back with the retained background sound. As a result, the background-sound components of the source and target are strictly identical, forming an auditory-side differential control benchmark.
  [Uncertain] It is not stated how the residuals/artifacts of the SAM-Audio separation are handled — in practice, semantic source separation leaves spectral holes or separation residue, and if the target's background sound has subtle distortion introduced by the separation-and-remix process, the model would learn this distortion as "normal." The paper does not discuss this, nor does it present any verification or objective metric of separation quality.

【SNR】[Uncertain] No mention of an SNR threshold or any traditional signal-level audio metric; everything relies on the single learned comprehensive assessment, Audiobox-Aesthetics.

### [2026 miscellaneous joint audio-video generation works](../models/JAVG_2026_misc.md) ⚠️

【ALIVE — using an MLLM in place of traditional signal-processing metrics (the most directionally significant approach in this batch)】
Original text: "We utilize an MLLM model ... to perform audio filtering based on two criteria. Firstly, for audio quality, the model assigns a score to each audio, allowing us to discard samples with significant background noise. Secondly, for audio-visual coherence, we assess the correlation ..."
(1) Audio quality: an MLLM directly scores each audio clip, and samples with significant background noise are removed on this basis — notably, no traditional audio quality metric such as SNR, PESQ, or DNSMOS appears here at all; everything is handed over to the multimodal large model to "listen" and judge. This is the single point in this batch most representative of the 2026 trend: audio quality assessment shifting from signal-processing metrics to large-model semantic judgment.
(2) Audio-visual consistency: correlation is assessed, and strongly correlated samples are grouped separately, controlling the proportion of weakly correlated data such as BGM "to optimize the dataset's composition."
(3) Not mentioned: an SNR threshold, silence detection and silence-ratio, no-audio-track removal (though the first pipeline step already filters for "videos with audio," which is equivalent to removing no-audio-track samples), source separation (not mentioned at all), clipping/distortion detection [all uncertain].
【NAVA — AudioBox Aesthetic as the audio-quality operator】
Audio quality filtering uses "AudioBox Aesthetic scores" (Meta's audio-aesthetics assessment model, which outputs a multi-dimensional audio quality score). This is the only work in this batch that explicitly names the use of a dedicated audio-quality model, closer to human perception than traditional SNR. It also uses YAMNet audio classification for category labeling. Thresholds are not disclosed [Uncertain]. No mention of SNR, silence, or source separation [Uncertain].
【OmniCustom — format unification only, no quality filtering】
"We extract audio files from videos and unify them into 16kHZ" — audio is uniformly resampled to 16 kHz, with no description of audio-quality filtering [Uncertain]. Its filtering relies entirely on the SyncNet dual thresholds (which indirectly exclude audio-track-anomaly samples: a corrupted or silent audio track will produce extremely low SyncNet confidence and be removed). This is an economical approach that "implicitly performs audio-validity detection using sync detection."
【StreamChar / CCL / Baton】No description of audio quality filtering [Uncertain]. CCL uses WavCaps and VGGSound for audio pretraining, both of which come with their own built-in quality screening; Baton similarly uses AudioCaps + WavCaps. All three outsource audio quality control to upstream sources.
【ITS-JAVG】On the inference side, ImageBind-TA is used to assess semantic coherence between text and generated audio — not an audio-quality metric; there is no audio-quality verifier — this is itself a gap: none of its six verifiers specifically measures audio fidelity/quality, which could lead to retrieving semantically matched but poor-audio-quality samples [not discussed in the paper — this is an observation from this survey].
【Overall assessment】The most notable shift in audio quality filtering in this batch is "de-signal-processing-ization": ALIVE uses MLLM scoring, NAVA uses AudioBox Aesthetic (a learned perceptual model), and both bypass traditional metrics such as SNR. At the same time, none of the seven works uses source separation (Demucs, etc.) — reflecting a mainstream consensus of preserving the native mixed audio track, because the goal is precisely to generate a complete soundscape containing speech + sound effects + music.

### [Compilation of joint audio-video generation baselines](../models/JavisDiT_baselines.md) ⚠️

Filtering on the audio side is generally absent, with only formatting preprocessing:
【JavisDiT / JavisDiT++】Explicitly states that during the audio pretraining stage "no data filtering strategy" is applied, to ensure maximal coverage of text-to-audio generation ability across the three categories of general sound effects, music, and speech — this is a deliberate "do not filter" decision, the opposite of the strict audio screening used by most models. On the audio side there are only three formatting steps: truncation to under 30 seconds, uniform resampling to 16 kHz, and extraction of audio statistics. No SNR threshold, no silence detection, no no-audio-track removal, no background music separation, no clipping/distortion detection [Uncertain]. Conversely, there is one special "audio content filter" at the joint audio-video stage — FunASR is used to remove videos that contain human speech, but this is a category filter rather than a quality filter.
【Harmony】Speech data is screened by an "audio-video consistency scoring model," which is cross-modal consistency filtering rather than pure audio quality filtering; at the audio-quality level (SNR, silence, noise), no filtering is described [Uncertain]. Audio encoding uses a dual path of MMAudio's VAE encoder + F5-TTS's speech encoder. The reference audio is a random 1–3 second clip, and it is not stated whether a validity check is performed [Uncertain].
【UniAVGen】Only formatting processing: audio is first sampled at 24,000 Hz and converted to a mel spectrogram, and after generation the waveform is restored with the Vocos vocoder. Note its 24 kHz sample rate is higher than the 16 kHz used by JavisDiT/AV-DiT/MM-Diffusion, giving wider bandwidth and better speech fidelity — consistent with its focus on real human speech. No description of audio quality filtering [Uncertain].
【MM-Diffusion / AV-DiT】No audio quality filtering. AV-DiT's processing is "truncate or pad to a 1.6-second waveform, sample at 16 kHz, convert to a 40×16×8 mel spectrogram" — purely formatting.
【Common pattern】Audio quality control across this compilation relies mainly on upstream datasets (AudioCaps, Clotho, Emilia, etc., which are themselves curated corpora) rather than building an in-house audio quality-check step — this is workable on small-scale curated data, but is a clear risk exposure on self-collected data at the million-scale (such as Harmony's 2 million self-collected ambient-sound clips).

### [Kling 3.0 / Kling Video 3.0 Omni](../models/Kling_30_Omni.md) ⚠️

[Uncertain, though the same team has published explicit public specifications elsewhere] Kling 3.0 Omni does not disclose audio-quality-filtering details, and only has Kling-Omni's "audio-video corruption detection." Kling-Foley's published specification is fairly complete and can serve as a paradigm: audio is uniformly standardized to WAV / 44 kHz / 16 bit / stereo; quality metrics include SNR (signal-to-noise ratio), MOS score, clipping ratio, and audio bandwidth; VAD is used for silence filtering, and samples with a silence ratio ≥0.2 are removed; evaluation samples are required to have foreground audio free of human voice, and copyrighted background music containing vocals is excluded (i.e., there is BGM-vocal separation/removal processing). [Uncertain: whether Kling 3.0 Omni removes off-screen voice/narration sources, and whether it performs background music separation]

### [LTX-2](../models/LTX-2.md) ⚠️

The sole and central audio-side filtering criterion is "significant and informative audio components" — this in effect simultaneously serves the functions of removing no-audio-track samples, removing silent/near-silent samples, and removing low-information audio tracks (e.g., pure noise, extremely low loudness), and the paper describes this as the key to giving the subset "a balanced distribution of visual and auditory content."
【Not disclosed】The quantitative implementation of this criterion: no SNR threshold, no loudness/RMS threshold, no silence-ratio threshold, no audio-event-density metric; it is not stated whether off-screen voice/narration and other non-diegetic sound sources are removed, nor whether background music separation (source separation) or vocal-accompaniment separation is performed. This is a typical case of "the approach is clear but the implementation details are entirely non-reproducible." On the technical side, it is only known that audio enters the VAE as a 16 kHz stereo mel spectrogram and is output at 24 kHz; the admission requirements for the original audio track's sample rate/channel count are not stated. [Uncertain]

### [LongCat-Video](../models/LongCat-Video.md) ⚠️

Not applicable for the base version.
The description of audio-side filtering for Avatar 1.5 is fairly coarse: the offline stage includes a dedicated audio-annotation step that "verifies whether a sample contains usable speech conditions," i.e., using "presence of usable speech" as the admission gate; in terms of source selection, material such as interviews — with a stable subject and clear speech — is preferred, which is a form of source-level quality control. The report mentions the use of vocal separation, which can be used to strip vocals from background music/ambient sound, but does not state which model or parameters were used. Off-screen sources (where the owner of the audio track is not visible on screen) are removed indirectly through active-speaker detection.
Not disclosed: SNR threshold, silence detection and silence-ratio threshold, the rule for handling no-audio-track samples, audio sample-rate/bitrate requirements. [Uncertain: specific thresholds for SNR, silence ratio, etc.]

### [MOVA](../models/MOVA.md)

Coverage is fairly comprehensive, with disclosed thresholds:
【No-audio-track removal】The first step of preprocessing removes samples "missing valid audio channels," as well as samples that fail to decode. Consequently there are no silent videos in the training set.
【Silence ratio】Silence Ratio < 0.8 (i.e., clips with more than 80% silence are removed). This threshold is quite lenient, allowing a substantial amount of silence to remain; combined with the "keep only speech clips" strategy, the actual silence proportion in practice should be well below this.
【Bandwidth】Bandwidth > 1,000 Hz, used to remove audio tracks that are severely band-limited, encoding-degraded, or of telephone-line quality.
【Perceptual quality】Three-dimensional Audiobox-aesthetics scores: PQ (Production Quality, reflecting recording cleanliness and production standard) > 5.0, CU (Content Usefulness) > 4.5, CE (Content Enjoyment) > 2.5. The PQ threshold is set fairly high, effectively favoring studio-grade/professionally produced audio.
【SNR】No explicit signal-to-noise-ratio threshold is used; noise control is handled indirectly via Audiobox-PQ.
【Background music separation】No source separation is performed. Music is retained as part of the mix in the training audio track, and is described in the caption by Qwen3-Omni-Captioner (the paper's example caption includes a detailed description of background electronic music: synth pads, a steady beat, a minor-key melodic line, and a mix level low enough not to interfere with the narration).
【Off-screen voice/narration removal】Not performed, and the paper's full worked example is itself a clip of off-screen narration + no visible speaker on screen, showing that off-screen samples are retained in the training set. This may be one source of the ambiguous active-speaker-attribution problem in multi-speaker scenes (which the paper's Limitations section acknowledges).
【Loudness normalization】Starting at Phase 2, LUFS loudness normalization is introduced, aimed at mitigating the loudness explosion caused by CFG — this is a training-data-side treatment, but its motivation comes from CFG behavior on the inference side.

### [Merged entry: ① Mochi 1](../models/Mochi_MAGI_Motif.md)

Not applicable. The data pipelines of all three discard the audio track at the splitting stage (the pipeline diagrams for both MAGI-1 and Motif contain only a visual branch), with no SNR, silence detection or silence-ratio threshold, no-audio-track removal, off-screen-source removal, or background music separation.

### [Movie Gen](../models/Movie_Gen.md)

· Silence handling: after labeling with AED, entire videos whose dominant class is silence are discarded outright; no specific numeric threshold for silence ratio is reported.
· Audio quality: an audio-quality prediction model was trained to output a continuous score from 1–10 (the annotation approach mirrors the collection method of LAION aesthetics, with 10 as the highest). The key design point is that this score is not used as a hard filtering threshold, but is instead written into the caption as a controllable condition — at inference time, the quality-score condition is set to 7.0 for pure sound-effect generation and to 8.0 for joint sound-effect + music generation. Ablations show that higher conditioning quality scores yield higher predicted quality, but subjective preference saturates after around 6.5 (samples containing off-screen music can continue to benefit up to higher scores).
· Removal of off-screen/irrelevant sound: low-similarity clips are identified via CAVTP and off-screen audio is routed out of the main pretraining set; the fine-tuned cinematic split explicitly pursues the professional-mixing characteristic of "suppressing ambient noise and irrelevant off-screen sound."
· Definition of film-grade audio quality: the paper attributes the gap between cinematic sound and low-end device recordings (phone, surveillance camera) to two aspects: sound quality (how it sounds, requiring professional microphone recording plus mix/mastering processing to remove pop noise and wind noise) and sound design (what sounds to include, foregrounding narratively relevant events such as explosions/dialogue, with ambient music mixed in with fade-in/fade-out) — a cinematic-feel classifier plus AED automated screening plus manual annotation are used to bridge this gap.
· Vocal removal: the cinematic split excludes clips containing vocals.
· Associated visual-side filtering: videos with OCR-detected text, static videos, and videos with resolution <480px are removed to reduce visual-modality noise.
· No SNR threshold is reported, and no background music separation (BGM separation) is performed — on the contrary, non-vocal music is deliberately retained and is one of the deliberate modeling targets.

### [NVIDIA NeMo Curator](../models/NeMo_Curator.md) ⚠️

None in the video pipeline (audio tracks are not processed, so there is no no-audio-track removal, silence-ratio threshold, off-screen-source removal, or background music separation).
The standalone audio pipeline has quality filtering oriented toward speech data: (1) WER/CER filtering — using ASR transcription error rate as a proxy metric to remove low-quality audio, the primary means; (2) audio analysis — duration-calculation and format-validation; (3) VAD (voice activity detection), added in 26.02, can be used to locate valid speech segments and remove silence; (4) SQUIM objective quality metrics (which can estimate SNR, STOI, PESQ, and other speech-quality dimensions without a reference), added in 26.07, along with bandwidth estimation (identifying up-sampled fake-high-sample-rate audio), plus support for optional secondary ASR scoring as cross-validation.
【Missing】Source separation (vocal/accompaniment separation), background-music detection, sound-event detection, loudness normalization — none of these have a built-in stage; default thresholds for the various metrics are not given in the documentation. [Uncertain]

### [OmniHuman dataset + OHBench](../models/OmniHuman.md) ⚠️

Audio governance sits at the second level of the pipeline, the only one of the four filtering levels dedicated specifically to audio:
【Removal criteria (three categories)】
- Missing audio tracks: silent videos are eliminated outright, ensuring the dataset is 100% native-audio-track;
- Abnormal duration: samples where the audio-track duration does not match the video duration, or falls outside the preset range, typically corresponding to technical faults such as container errors or audio-track truncation;
- Low quality: the paper states that the criteria include silence ratio and volume thresholds, but no specific numeric values are given. [Uncertain]
【Source separation rather than filtering: the core strategy】Demucs is used for four-source separation, taking vocals as the target track and mixing the other three tracks into a background track. This step does not eliminate any samples; instead it converts the problem of "background-audio interference" from a "filtering problem" into a "representation problem" — samples containing BGM are no longer treated as noise, but become dual-track data that can be used separately. This is the most critical trade-off this dataset makes on the audio side, and it is why it is able to maintain a million-scale dataset from a BGM-saturated source like YouTube (had a UniTalking-style "discard anything with background audio" approach been used, the retention rate would have collapsed).
【Handling of off-screen sources】There is no explicit "off-screen removal" step; instead this is achieved implicitly through SyncNet attribution matching — if a speech segment cannot be matched to any visual trajectory on screen (narration, off-screen voice, post-production dubbing), the sample is eliminated because "all subjects must satisfy the synchronization condition." The effect is equivalent to off-screen removal, but the mechanism is attribution failure rather than dedicated detection.
【SNR】No mention of an SNR metric or threshold. The volume threshold can be viewed as a partial substitute, but it is not equivalent. [Uncertain]
【Audio-quality records at the annotation level】Individual-level annotations include "speech quality annotations," meaning audio quality is not used only for filtering but is also retained as a queryable field, from which downstream users can screen for a high-audio-quality subset.
【Sample-rate standardization】Unified to 44.1 kHz.
【Audio-quality metrics on the evaluation side】OHBench global level: the mean of the four Audiobox Aesthetics dimensions (AbS, comprising content enjoyment, content usefulness, production complexity, production quality), KL divergence, Fréchet Distance (FD); individual level: DNSMOS's OVRL score (Speech Realism), WER computed by SenseVoice (pronunciation accuracy). Using Audiobox-Aesthetics for audio-aesthetics assessment is a relatively new choice, richer in information than the traditional single FAD metric.

### [Open-Sora series](../models/Open-Sora.md)

Not applicable. The pipeline does not read the audio track; transcoding is unified to an H.264 video stream, with no SNR, silence detection, no-audio-track removal, or vocal/background-music separation.

### [Ovi](../models/Ovi.md) ⚠️

Audio-side filtering is relatively simple, with just one hard threshold plus a few implicit constraints:
(1) Volume threshold: average volume must be ≥ −60 dB, applied together with the SyncNet dual thresholds. This serves to remove silence, near-silence, corrupted audio tracks, or clips with extremely low volume — i.e., implicitly accomplishing "removal of samples without a valid audio track."
(2) No explicit SNR/noise filtering: the paper does not describe an SNR threshold, noise suppression, reverberation filtering, or clipping/distortion detection [Uncertain].
(3) No background music separation: source separation (e.g., Demucs) is not used to split BGM from vocals — in fact Ovi deliberately preserves the native mixed audio track, because its goal is to simultaneously generate a complete soundscape of speech + sound effects + BGM.
(4) No explicit module for off-screen source removal: filtering relies only indirectly on low SyncNet confidence [Uncertain].
(5) Silence-ratio threshold: only the global "average volume" metric exists; no segment-level silence-ratio statistic is seen [Uncertain].
(6) Sample-rate and bandwidth constraints (engineering level): audio is uniformly processed through MMAudio's 16 kHz encoder variant (STFT → mel spectrogram → 1D VAE latent). The paper's Limitations section acknowledges that this fixed 16 kHz 1D-VAE path limits bandwidth and spatial perception, flattening out high-fidelity music, spatial cues, and subtle timbre, and suggests that a higher-bandwidth latent or post-hoc bandwidth expansion could be used in the future. At inference time, the waveform is restored by a BigVGAN vocoder.

### [Script-a-Video](../models/Script-a-Video.md) ⚠️

The paper discloses no data-side audio quality filtering at all: no SNR threshold, no silence detection or silence-ratio threshold, no "discard if no audio track" rule, no background music separation (source separation such as Demucs), no loudness/clipping/distortion detection. [Uncertain]
【Schema-level equivalent mechanism】MTSS partially substitutes for traditional audio filtering with two annotation rules:
1) Handling of off-screen sources — the strict audio-visual coupling principle means sound with no visual counterpart cannot become an Event, which in effect achieves the goal of "removal of off-screen sources," but through demotion rather than removal: such sounds are folded into the global_audio field of the Global stream and retained. This is a "don't lose information, only demote the level" design, more data-efficient than outright removal.
2) Handling of background music — rather than signal-level source separation, semantic-level layering is used: music that constitutes an independent narrative event goes into the Event stream (type=music), while music that only serves as ambient underscore goes into the global_audio field of the Global stream. This is an approach that substitutes annotation-level layering for audio signal processing.
【Audio-quality metric on the evaluation side】UTMOS (a lightweight speech-quality MOS predictor) is used to assess the speech quality of generated audio, not for data filtering. Values are found in the sync_metric_and_threshold field.
【Implicit audio-track requirement】The entire schema is heavily dependent on real audio tracks (both the Event stream and global_audio must be extracted from the original audio), so the training data necessarily comes with native audio tracks, but the paper does not write this as an explicit filtering rule.

### [Seedance 2.0 and Seedance 1.5 pro](../models/Seedance_20_Seedance_15_pro.md) ⚠️

[Uncertain] No disclosure of an SNR threshold, silence-ratio threshold, no-audio-track-sample removal strategy, off-screen-source removal, or background music separation (source separation), or any other specific practice. Only an evaluation-side definition is available for reference: SeedVideoBench 1.5's "audio quality" metric measures the intrinsic acoustic quality of the output (including both vocal and non-vocal components), with key criteria including the presence of artifacts such as clipping or truncation, spatial-sound-field rendering, timbral realism, and overall signal clarity — these criteria are quite likely also used as admission standards for training-data audio quality. Seedance 2.0's report acknowledges in its limitations that "occasional audio distortion" still occurs.

### [SkyReels series](../models/SkyReels.md) ⚠️

SkyReels-V4's audio quality filtering has the most complete set of metrics in this entry, using four objective metrics plus one activity-detection step:
(1) SNR (signal-to-noise ratio) — removes samples with excessive background noise;
(2) MOS score (Mean Opinion Score, a subjective-quality estimate given by an automated MOS-prediction model) — removes samples with poor perceived quality;
(3) Clipping ratio — removes samples with overload distortion;
(4) Audio bandwidth — identifies and removes fake high-sample-rate audio that lacks high frequencies due to low-bitrate compression;
(5) VAD (voice activity detection) — requires the silence ratio to be below 0.2, i.e., silence may not exceed 20%;
(6) Average volume threshold of −60 dB (used together with sync filtering).
In addition, duration is normalized: long audio is cut into 15-second chunks, and short audio is concatenated by category up to 15 seconds.
【Not disclosed】Specific threshold values for each metric (aside from the 0.2 silence ratio and −60 dB, no thresholds are given for SNR/MOS/clipping ratio/bandwidth); background music separation (source separation) or vocal-accompaniment separation is not mentioned; it is not stated whether off-screen/narration and other non-diegetic sound is removed; the admission requirements for sample rate and channel count are not stated (audio is known to be encoded at 44.1 kHz, with a ratio of 5 seconds → 218 latent tokens). SkyReels-V2 has no audio processing. [Uncertain]

### [Sora 2](../models/Sora_2.md) ⚠️

Entirely undisclosed. No mention of an SNR threshold, silence detection and silence-ratio threshold, no-audio-track-sample removal strategy, off-screen-voice/narration-source removal, background music separation (source separation), or any other specific measure. [Uncertain]

### [SpeakerVid-5M](../models/SpeakerVid-5M.md) ⚠️

Audio-side filtering relies entirely on Whisper's diagnostic outputs, using three parallel veto rules — meeting any one leads to removal:
【1. Transcription confidence】Samples with a Whisper confidence score < −1.5 are removed; the HQ / SFT subsets tighten this further to > −1. This metric indirectly reflects the intelligibility and signal-to-noise level of the audio — noisy, heavily reverberant, or far-field recordings significantly lower the confidence score.
【2. No-speech probability】Samples with no-speech probability > 0.8 are removed. This one criterion serves two roles simultaneously: removing silent clips and pure-background-sound clips, and constraining the dataset to be a pure speech corpus (clips dominated by sound effects, music, or ambient sound are eliminated by this rule).
【3. Compression ratio】Samples with compression ratio > 2.5 are removed. This is a metric Whisper uses internally to detect transcription degeneration — when the model falls into a repetitive looping output (e.g., "thanks for watching, thanks for watching…"), the compression ratio rises abnormally. This criterion in effect removes ASR-hallucination samples, and is a safeguard on annotation reliability rather than a direct measure of audio-signal quality.
【4. Language mismatch】The pipeline also mentions filtering based on "detected language mismatches," i.e., removing samples where the language detected by Whisper does not match the expected language.
【Processing not performed】The paper describes no explicit SNR estimation and threshold, no silence-ratio statistic and threshold, no background music separation (BGM separation / source separation), no speech enhancement or denoising, no removal of off-screen voice/narration sources, and no loudness normalization. Compared with MOVA's use of Audiobox-aesthetics' three-dimensional PQ/CU/CE aesthetic scores plus silence-ratio and bandwidth thresholds, SpeakerVid-5M's audio quality control is noticeably thinner — it accomplishes all audio screening by relying on the by-products of a single model, Whisper, which is a low-cost but single-dimensional approach.
【No-audio-track samples】The original collection criterion is itself audio-visual paired video, and the dual filtering by no-speech probability and confidence score ensures the retained samples all contain valid speech, so no-audio-track or no-speech samples do not exist. [Uncertain]

### [Step-Video-T2V](../models/Step-Video-T2V.md) ⚠️

Not applicable. The data pipeline does not extract, retain, or process the audio track; there is no SNR computation, silence detection, no-audio-track removal, off-screen-source removal, or background music separation, or any other such step. [Uncertain]

### [UniTalking](../models/UniTalking.md) ⚠️

Audio quality filtering is concentrated at the second level of the funnel, comprising three rules, with the tools being PANNs + SentenceASD — none of them have disclosed threshold values:
【Silence removal】Muted samples, i.e., videos with no valid audio, are removed. It is not stated how this determination is made (an energy threshold? all-zero detection?) or what silence-ratio threshold is used — only a binary "is it silent or not" removal is performed; no proportional judgment of the form "remove if silence ratio is too high" is seen. [Uncertain]
【No-speech removal】Samples that lack speech are removed. This is the most central rule at this level, carried out by PANNs + SentenceASD performing speech event detection. PANNs is a 527-class audio-event-classification CNN pretrained on AudioSet, which can give the existence probability of the "Speech" class; SentenceASD works alongside it. Compared with UniVerse-1's use of Whisper ASR to build a binary speech-presence gate, using a dedicated audio-event classifier is lighter-weight and more robust (it does not depend on whether the ASR can successfully transcribe). Note that the elimination here is thorough — UniVerse-1 demotes and retains no-speech samples as "general-purpose audio-video data," whereas UniTalking discards them outright, which directly determines the two works' very different domain positioning.
【Low-SNR removal】Low-SNR samples are removed. No SNR threshold is given, and it is not stated how SNR is estimated (a dedicated reference-free SNR-estimation model, or signal statistics). [Uncertain]
【Audio filtering not performed】
- No audio-aesthetics scoring (compare MOVA's use of Audiobox-aesthetics' three PQ/CU/CE scores);
- No lower-bound bandwidth constraint, i.e., audio tracks that have been heavily compressed with missing high frequencies are not removed;
- No detection of recording defects such as clipping, distortion, or pops;
- No background music separation (source separation) — background music is not stripped, and goes into training together with the vocals unchanged (as noted earlier, this is instead turned into an attribute controllable via prompt);
- No reverberation/room-acoustics filtering;
- No lower-bound sample-rate constraint.
【Sample rate and audio representation】The raw waveform is converted to a mel spectrogram via STFT and then encoded by MMAudio's 1D VAE; at inference time, the latent is decoded back to a mel spectrogram and then synthesized into a 44.1 kHz waveform by a BigVGAN vocoder. The 44.1 kHz output sample rate is higher than the 25.6 kHz that UniVerse-1 down-adjusts to in order to accommodate 25 fps video — because UniTalking uses architecture-level RoPE alignment rather than sample-rate alignment, it is able to preserve full audio bandwidth, an incidental advantage of its architectural choice.
【Overall assessment】The coverage of audio quality control (silence / no-speech / low SNR / off-screen sources) is adequate for the speaker domain, and the tool selection (PANNs + ASD) fits the task well, but the complete non-disclosure of thresholds makes it non-reproducible; there remains a clear gap in granularity compared with MOVA's audio-aesthetics scoring system.

### [UniVerse-1](../models/UniVerse-1.md) ⚠️

Audio-side filtering occupies only a single level of the funnel, and uses only signal-level handcrafted features, with no learned audio-quality assessment at all:
【Audio activity detection (the only audio filtering step)】Three signal-level statistics are analyzed — volume, energy, and zero-crossing rate — used to identify and remove silent segments. The paper gives no specific numeric threshold. [Uncertain]
【No-audio-track removal】The very first step of the funnel discards videos lacking an audio track — the most front-loaded one-vote veto.
【Audio filtering not performed】
- No SNR/signal-to-noise-ratio threshold;
- No audio-aesthetics scoring (MOVA uses Audiobox-aesthetics' three scores PQ>5.0 / CU>4.5 / CE>2.5; UniVerse-1 has no corresponding facility at all);
- No lower-bound bandwidth constraint;
- No proportional silence-ratio threshold — only a binary "is it silent or not" removal is performed, with no continuous judgment of the form "remove if silence ratio is too high";
- No removal of off-screen voice/narration sources;
- No background music separation (source separation) — background music in speech clips is not stripped, and is trained together with the vocals unchanged;
- No detection of recording defects such as clipping, distortion, or pops.
【Sample-rate handling】Uniformly resampled to 25.6 kHz (down-adjusted from Ace-step's native 44.1 kHz), for the purpose of aligning with 25 fps video rather than for quality considerations — but this in effect lowers the audio bandwidth ceiling (a Nyquist frequency of 12.8 kHz), incurring a loss of high-frequency detail (such as cymbals and sibilance), a quality cost paid for the sake of cross-modal alignment. The paper does not discuss the impact of this trade-off.
【Overall assessment】Audio quality control is the weakest link in UniVerse-1's data pipeline, consistent with its overall approach of "the audio expert foundation comes from the music-generation model Ace-step, with audio capability mainly inherited from the base model" — the data side does not perform deep audio-quality screening, relying instead on the audio priors already present in the base model.

### [Unison](../models/Unison.md) ⚠️

The paper describes no audio quality filtering step at all: no SNR/signal-to-noise-ratio threshold, no silence detection or silence-ratio threshold, no description of no-audio-track removal, no audio-aesthetics scoring threshold, no lower-bound bandwidth constraint, no clipping/distortion/pop detection, no description of sample-rate unification.
【The only explicitly performed audio processing is separation, not filtering】Mel-RoFormer decouples the mixed audio into "high-fidelity speech and sound-effect components." This is preprocessing, not screening — all samples are processed, and no sample is removed for audio quality reasons.
【An important distinction from background music separation】One important clarification is needed: when works such as MOVA and UniVerse-1 mention "background music separation," they usually mean "strip out and discard the background music, keeping only the vocals"; whereas Unison's Mel-RoFormer separation has exactly the opposite purpose — both separated streams are retained, and both serve as ground truth in training. The speech stream trains speech, the sound-effect stream trains sound effects, and neither stream is discarded. This is "decoupling in order to model separately" rather than "separating in order to purify" — a fundamental difference in data-processing philosophy from comparable works.
【The phrase "high-quality audio segments"】The paper uses this phrase to describe its 50-million-segment audio corpus, implying that some quality screening exists, but gives no screening basis, metric, or threshold. This is most likely inherited from the quality standards of upstream datasets (WavCaps, AudioSet, YuE, etc.) rather than being self-built by Unison. [Uncertain]
【Audio-quality model used in evaluation but not in filtering】Audiobox-Aesthetics (following the Audiobox protocol) is used to compute perceptual quality PQ and content usefulness CU. Unison is the best performer across the entire TI2AV field on both metrics (PQ 6.34, CU 5.61, both better than LTX-2's 6.30/5.58). As on the visual side, the team has an audio-quality-scoring tool at hand but does not use it as a data filter — compared with MOVA, which explicitly uses Audiobox-aesthetics' three thresholds PQ>5.0 / CU>4.5 / CE>2.5 to filter training data, Unison has a clear gap here.
【The hidden risk of separation residue】Mel-RoFormer's separation is not perfect — residual sound effects in the speech track and residual vocals in the sound-effect track are the norm. Such residue enters the dual-stream supervision as "incorrect ground truth," which in theory weakens the decoupling effect. The paper neither evaluates separation quality nor discusses the impact of the residue, nor does it set a separation-quality threshold to remove hard-to-separate samples. This is the aspect of the dual-stream architecture most worth questioning on the data side. [Uncertain]

### [Veo 3 / Veo 3.1](../models/Veo_3_Veo_31.md) ⚠️

[Uncertain] Entirely undisclosed. No mention of an SNR threshold, silence detection and silence-ratio threshold, a rule for removing no-audio-track samples, removal of off-screen voice/narration sources, background music separation (BGM separation), or any other specific measure. It can only be inferred from the general statement that "training videos are filtered by quality" that audio quality should be included in this.

### [Vidu S1](../models/Vidu_S1.md) ⚠️

(1) No-audio-track / incomplete-audio-video samples are removed at the pre-filtering stage by an "audio-video completeness" check;
(2) Removal of off-screen sources: speech segments judged by ASD as offscreen (the speaker is not visible on screen) are specially flagged and handled differently; overlap clips (multiple overlapping voices) are removed outright;
(3) Removal of background music/singing: it was observed that the diarization model is unstable in scenes with singing or strong background music (vocals get misassigned to the music stem, producing synthetic timbre and distortion artifacts), so a heuristic rule was introduced to discard clips where "the speaker is vocalizing but the proportion of speech energy is too low"; this in effect also serves as a backstop for the separation quality between vocals and background music;
(4) Speech-component extraction: before training, the speech component is explicitly extracted from the original audio.
No specific numeric values are given for an SNR threshold or silence-ratio threshold, etc. [Uncertain]

### [Wan 2.5 / 2.6 / 2.7](../models/Wan.md) ⚠️

On the training side there are only category-level rules, with no quantitative threshold at the signal level at all.
【Wan 2.1 V2A】The only audio-side filtering consists of two exclusion rules: remove videos lacking soundtracks, and remove videos containing speech or vocal music. Of these, "remove no-audio-track" serves the function of eliminating silent samples. No SNR threshold, loudness/RMS threshold, silence-ratio threshold, or audio-event-density metric is seen; it is not stated whether source separation of background music and vocals is performed; the admission requirements for the original audio track's sample rate and channel count are not stated.
【Wan2.2-S2V】Off-screen/no-on-screen-source samples are indirectly removed via Light-ASD, but again with no audio-signal-quality metric (no SNR, no silence detection).
【Technical spec as circumstantial evidence】Wan 2.1 V2A's output is 44.1 kHz stereo, at most 12 seconds; audio is compressed directly in the waveform domain via a 1D-VAE.
【Input audio constraints on the inference side (which may reflect a preference in the training-side spec)】For wan2.5/2.6, the audio_url is required to be wav/mp3, 3–30 seconds long, ≤15 MB; wan2.7 relaxes this to 2–30 seconds. Any portion exceeding the duration limit is automatically truncated and discarded; when the audio is shorter than the video duration, the portion of video beyond the audio length is output as silent video (e.g., a 3-second audio paired with a 5-second video would have sound for the first 3 seconds and silence for the last 2) — this behavior indicates the model has conditional tolerance for "partial silence," which may correspond to the presence of samples with silent segments in the training data.
The audio quality filtering strategy for 2.5/2.6/2.7 is not disclosed. [Uncertain]

### [Compilation of audio-video generation evaluation benchmarks](../models/av_benchmarks.md) ⚠️

【AV-SyncBench's two gates】The first gate: Gemini 3 Flash automatically removes samples with "off-screen sound sources" — this is one of the most critical and hardest-to-implement filtering items for AV training data (post-production dubbing, off-screen narration, and pasted-in background music all fall into this category); the second gate: cross-checking by 5 human reviewers, confirming that the main sound source is visible on screen, and removing clips with poor audio quality, excessive noise, or ambiguous semantics. Together these two gates ensure "what is seen is what is heard," and represent the best-practice example in this survey for on-screen source filtering (no quantitative thresholds such as SNR are published [Uncertain]).
【VABench】A three-piece set of audio-quality metrics can be directly reused as filters: DNSMOS measures background noise/speech clarity, NISQAv2 measures overall quality and naturalness, and Audiobox measures audio aesthetics (enjoyment, usefulness, production complexity, and quality); on the generation side, a 48 kHz stereo track is uniformly extracted.
【AVBench】NISQA MOS measures audio quality, and Audiobox measures audio aesthetics; both output continuous scores, and the paper explicitly states they can be used as a data-filtering mechanism. The Hard subset deliberately retains samples with noisy backgrounds and overlapping speech, indicating its positioning is "hard rather than dirty" — when filtering training data, one should likewise distinguish between "noisy but genuine" and "poor audio quality that needs removal."
【PhyAVBench】Recording in a controlled environment guarantees audio quality from the source; at the quality-control stage, human review removes samples containing unintended physical interference factors (equivalent to removing mixed-in irrelevant sound sources).
【Omni-Judge】Has two dimensions, audio quality and audio aesthetic, judged by an Omni-LLM, but the overall conclusion suggests that LLM scoring of perceptual dimensions is unreliable.
None of the five mention a silence-ratio threshold, no-audio-track removal, or the specific approach to background music separation (source separation) [Uncertain].

### [Video caption model ecosystem](../models/caption_models.md) ⚠️

In the captioner ecosystem, audio-quality handling is mainly manifested as a "routing gate before labeling" and "acoustic verification after labeling":
【Routing before labeling】UniVerse-1, in its offline funnel, uses Whisper to determine whether a clip contains speech, as a routing gate deciding which annotation path to take; the video stage of the JavisDiT series uses FunASR to remove clips containing speech (while the audio stage explicitly states it applies "no filtering at all, to maximize T2A capability coverage across the three categories of audio" — the contrast of "no filtering for audio, strict filtering for video" is a practical judgment the team made about the quality-quantity trade-off for the two modalities); LTX-2's data-subset screening criterion is "contains significant, information-rich audio components," to remove silence and uninformative audio tracks.
【SNR / audio-quality scoring】Movie Gen uses a dedicated audio-quality prediction model (outputting a score of 1–10); Foley-Omni uses an AudioBox threshold of 0.6; InstructAV2AV uses an Audiobox threshold. All of these occur upstream of the captioner.
【Background music separation and source separation】Bandit (Foley-Omni, three-way speech/effects/music separation, isomorphic with the three-field schema); Mel-RoFormer (Unison, on the training side decoupling the mixed audio into two ground-truth latent streams of speech and sound effects, and on the evaluation side performing vocal separation prior to WER); SAM-Audio (InstructAV2AV, semantically separating the target entity's sound from the mixed audio track).
【Removal of off-screen sources】Foley-Omni's −35 dB energy gate is in essence removing "hallucinated" annotations of the form "present in the frame but absent from the audio track"; run in reverse, the same mechanism can identify off-screen sound (present in the audio track but absent from the frame). Movie Gen's evaluation dimension "on-screen sound correctness" is the only metric that explicitly quantifies this issue, with its SFT showing a +31.0±16.0 improvement relative to PT.
[Uncertain] No captioner work discloses the silence-ratio threshold or SNR threshold of its training data.

### [Compilation of geometric/structured annotation datasets](../models/geometric_datasets.md) ⚠️

Not applicable. None of the four datasets process the audio track, so there is no filtering step for SNR, silence ratio, off-screen sources, or background music separation. The paper does not state whether SpatialVID retains the original audio track when transcoding to H.265 MP4 [Uncertain]; Action100M only releases text annotations, with audio to be obtained by users themselves from the HowTo100M source.

### [Post-training data strategies for video generation](../models/post_training_data.md) ⚠️

The anchor paper does not address this [Uncertain]. In the post-training stage, audio quality control mainly appears in the form of reward models rather than threshold filtering: JavisDiT++ uses AudioBox-Aesthetics to assess audio quality; Seedance 1.5 pro's multi-dimensional reward includes audio fidelity. The exception on the data side is Movie Gen's audio SFT set — the cinematic split is automatically screened by a cinematic-feel classifier + AED sound-event detection and then manually selected, explicitly excluding clips containing vocals; the admission condition for Foley-Omni Stage 3 is passing the full cleaning pipeline (six filtering thresholds + Gemini annotation + Bandit energy verification) and containing multiple audio components. Specific thresholds for SNR, silence ratio, no-audio-track removal, background music separation, etc. do not appear in any of the post-training materials [Uncertain] — these are the responsibility of the pretraining data pipeline.

### [Compiled survey of mainstream video pretraining datasets: Panda-70M, InternVid, Koala…](../models/pretraining_datasets.md)

Not applicable. None of the seven pipelines read the audio track: no SNR computation, no silence detection or silence-ratio threshold, no no-audio-track removal, no off-screen-source removal, no background music separation. Koala-36M's re-splitting code (cv2.VideoWriter/mp4v) **directly discards the audio track** when re-clipping; Panda-70M (download_audio: True) and UltraVideo (which states it preserves native audio) do retain the original audio track, but perform no quality checks of any kind.
## Classification and separate handling strategies for speech/sound effects/music

`audio_type_handling` · Detail level: brief

### [Allegro](../models/Allegro.md)

Not applicable. No classification or separate handling strategy for speech/sound effects/music.

### [Apollo](../models/Apollo.md) ⚠️

Apollo elevates "classifying by audio type and handling each type separately" to a first-class stage of the pipeline (Section 4.2, Audio-Guided Data Splitting) — this is the most distinctive design in its data methodology:
[Two-level tree classification]
- Level 1: vocal (contains human voice) vs non-vocal (no human voice) → the latter forms the sound split (natural sound/sound-effects subset).
- Level 2: the vocal subset is further split into three branches: singing / single-speaker speech / multi-speaker speech.
[Classifier] The paper does not specify which model performs vocal/non-vocal discrimination or speaker-count determination (compare MOVA, which explicitly uses the EAT self-supervised audio Transformer + Silero VAD). It is speculated that a combination of VAD + speaker diarization + singing detection could implement this, but this is unconfirmed. [Uncertain]
[Differentiated handling strategy]
- Speech and singing subsets: verbatim transcription + speaker attribute extraction (gender, age) + audio caption + video caption.
- Sound split: only audio captioning, no transcription, no speaker attributes ("the sound split receives only audio captions").
That is, annotation cost is allocated according to each subset's information structure, avoiding meaningless ASR on segments with no human voice.
[Music's place] The paper's four-category enumeration does not list "music/instrumental" as a separate category — whether purely instrumental content falls into the sound split or is filtered out is not stated; music with vocals is classified under singing. This differs from MOVA's approach of specifically introducing the JamendoMaxCaps music corpus during the audio-tower pretraining stage.
[Downstream use] This splitting directly serves multi-task training: the sound split primarily feeds general T2A/foley capability, while the speech/singing split primarily feeds TTS and lip-sync capability; evaluation is correspondingly divided into groups such as audio, TTS, and audio-video consistency (Figure 5).

### [CineDance / CineDance-1M](../models/CineDance.md)

[Classification method] Within shot-level audio prompts, non-speech audio is explicitly split into three categories — music, ambient sound, and effects — each described separately; speech/dialogue goes through an independent ASR channel and is further bound to character anchor tokens; character vocal timbre has a separate character voice description field. Overall this forms a parallel annotation structure across four categories: speech / music / ambient sound / effects.
[Separate handling strategy]
  · Speech: sentence-level ASR transcription → windowed speaker-character binding (95.4% accuracy) → character voice-timbre description; injected into the prompt during training as (spkᵢ,ℓ, speechᵢ,ℓ) tuples; evaluated with Whisper-large-v3 to compute WER/CER.
  · Music/ambient sound/effects: uniformly handled via natural-language soundscape description (audio prompt), with no further category labeling or stem separation; evaluated using AudioBox-Aesthetics' three components — PQ (Production Quality), CE (Content Enjoyment), CU (Content Usefulness).
[Design orientation] Speech follows a structured route of "precise transcription + identity binding," while non-speech follows a flexible route of "natural-language description" — the two handling paradigms are clearly distinguished.
[Not done] No audio source separation (splitting dialogue, music, and effects into separate stems) was performed, nor were differentiated quality thresholds or sampling ratios set for the three audio categories.

### [CogVideoX](../models/CogVideoX.md) ⚠️

[Uncertain] Not applicable to the video side of CogVideoX. Based on the public description, CogSound takes the form of "a single model generating multiple audio categories" — it can generate complex sound effects such as explosions, flowing water, instruments, animal calls, and vehicle sounds, and can also generate musical elements such as rhythm. There is no mention of separating models or data flows by speech/sound-effects/music, nor of classifiers, mixture ratios, or differentiated handling strategies for each category.

### [Cosmos-Predict2.5](../models/Cosmos-Predict25.md)

Not applicable. There is no classification or splitting/handling of speech/sound-effects/music. The mechanism in this pipeline that plays an equivalent role to "type splitting" is a purely visual content-type classification: an internally trained content type classifier labels each clip according to a self-built 26-category taxonomy, and serves as the primary axis for sharding used in domain-balanced sampling; there is also splitting across independent pipelines for the five major Physical AI domains.

### [Data-Juicer 2.0](../models/Data-Juicer.md) ⚠️

[Uncertain] Data-Juicer does not provide an explicit classification system or separate handling strategy for speech/sound-effects/music — this is the core shortfall of its audio capability.
[Indirect means usable for approximate classification]
  · video_tagging_from_audio_mapper — produces audio event tags based on the Audio Spectrogram Transformer (AST, trained on AudioSet). The AudioSet ontology is a hierarchical set of 527 classes, whose top-level categories happen to include Human sounds (including Speech), Music, Sounds of things, Natural sounds, Environment and background, etc. — in theory the top-level categories could be used to coarsely classify audio into speech/music/sound-effects/ambient-sound. However, DJ does not package this mapping into an explicit category field, nor does it provide an operator for category-based ratio sampling; users must parse the tags themselves and write the mapping logic.
  · video_audio_ASR_mapper — if it succeeds in transcribing meaningful text, this can serve as strong evidence of "containing speech."
  · video_audio_speech_emotion_mapper / video_audio_detect_age_gender_mapper — these two operators are only meaningful when a human voice is present, so their usability itself constitutes an indirect indicator of speech presence.
[Key gaps]
  · No audio-track source-separation operator (Demucs, Bandit, HTDemucs, etc. are none of them integrated), so a mixed audio track cannot be split into the three stems of speech/effects/music for separate handling and verification — this directly means the Foley-Omni-style "field-level energy gating" correction mechanism cannot be reproduced.
  · No ratio-control or stratified-sampling operator by audio category.
  · No dedicated music-detection/BGM-recognition operator.
[Significance for this survey] Data-Juicer's current operator system has evolved along the main line of "text → image → video → embodied-AI geometric annotation," with audio consistently playing a supporting role (out of 229 operators, only 3 Filters + 2 Mappers are purely audio). If it were used as the foundation for an audio-video joint-generation data pipeline, roughly on the order of 10 operators would need to be added on the audio side (source separation, silence detection, perceptual quality assessment, audio-track classification, sync detection, audio-track presence detection, loudness statistics, etc.) — a non-trivial amount of work, but entirely feasible architecturally.

### [Foley-Omni](../models/Foley-Omni.md)

The three-way classification of speech/sound-effects/music is the organizing thread running through the entire system — not merely a classification used in the cleaning stage, but the same partition shared across five levels: task definition, annotation schema, verification tooling, training curriculum, and evaluation benchmark.
[Dual classification tooling]
  · Semantic-discrimination path: Gemini 2.5 Pro makes a multimodal judgment of whether each of the three components is physically present (detect-then-describe).
  · Acoustic-verification path: Bandit source separation produces three stems, which undergo secondary confirmation/veto via −35 dB RMS energy gating.
  Bandit's selection is strictly isomorphic with the three-way classification (cinematic audio source separation naturally splits into speech/effects/music), avoiding the category-mismatch problem of general-purpose source-separation models.
[Separate handling strategy]
  · Annotation level: each of the three categories occupies its own independent field, [WORDS]/[AUDIO]/[MUSIC], any of which may be empty.
  · Data level: the three categories correspond to three single-task groups, TTS (1,253h) / TTA (912h) / TTM (139h), with corresponding video-conditioned versions VisualTTS (1,980h) / V2A (403h) (music has no video-conditioned single-task group).
  · Training level: Stage 1 first lays a foundation separately on the three single tasks (about 0.7M text-audio pairs, 5 epochs), Stage 2 introduces visual conditioning (V2A + VisualTTS, 3 epochs), and only Stage 3 performs joint generation of all three categories (V2ST, 2 epochs) — a curriculum design of separating first, then combining.
  · Generation level: the three categories are not generated separately and then mixed, but rather jointly generated in a shared latent space — classification is used for conditioning control and data organization, not for module splitting at inference time.
  · Evaluation level: V2ST-Bench splits into three tiers by co-occurrence combination (speech+effects, speech+music, all three present), and uses different metrics to emphasize different categories (WER for speech, CLAP/IB for sound-effects semantics, A-MOS as an overall metric).
[Ratio imbalance] Investment across the three categories is highly unbalanced: speech-related ≈66%, sound-effects-related ≈27%, music only 2.8% (139 hours of TTM, with no independent video-conditioned music task group). This is a clear trade-off — the paper's core selling point is the improvement of "speech intelligibility," with music capability relatively a secondary concern.

### [Goku](../models/Goku.md)

Not applicable. No classification or differentiated handling strategy for speech/sound-effects/music. Goku is a complete blank on the audio dimension; its value for this survey is concentrated on the purely visual side's data-distribution balancing and resolution-tiering threshold system.

### [Hailuo / MiniMax Video](../models/Hailuo.md)

Not applicable. No distinction is made among speech/sound-effects/music; there is no audio-type classification or separate handling strategy. MiniMax's speech (MiniMax Speech 2.8) and music (MiniMax Music 3.0) capabilities are handled by independent models and independent data pipelines, and no evidence of any data-level integration with the video line is seen.

### [HunyuanVideo-Foley](../models/HunyuanVideo-Foley.md) ⚠️

[Classify rather than discard: the core handling principle of this work] The three audio categories — speech, music, and sound effects — are handled in this work by "detect all, label all, keep all, ratio-control by label," which forms a clear contrast with peer works' approach of "discard or split by type."
[Classification tooling (pipeline stage 7)] Two stages in series: (1) a speech-music detection model, performing a coarse-grained three-way split into speech / music / other; (2) an audio classification model, performing finer-grained acoustic-event classification. Neither model is named specifically, and neither the taxonomy nor the number of categories is disclosed. [Uncertain]
[Roles of the three audio categories in this task]
- Sound effects (Foley/event sound) — the model's core target output, the true object that all data-processing work serves;
- Speech — not a target output but must be recognized. Ideally the model should learn to keep the vocal channel empty over on-screen speaking scenes and generate only ambient and action sounds. Keeping a moderate number of speech samples (rather than removing them all) helps the model see real acoustic scenes containing speech and learn acoustic layering; but keeping too many leads the model toward generating vague speech-like babble, severely harming usability. Striking this balance is precisely the key problem that "category distribution management" must solve — unfortunately the paper discloses nothing about the actual ratio or its effect. [Uncertain]
- Music — likewise not a target output. Background music is extremely common in web videos; without controlling its proportion, the model would tend to add music rather than sound effects to any scene. Detecting music and only labeling it, without doing audio source separation, indicates the team chose a "ratio-control" route rather than a "signal-processing stripping" route — lower cost but lower purity.
[Methodological value of the "classification + distribution management" route] Compared with MOVA (keeping only speech, discarding the rest) and UniVerse-1 (splitting into two by presence/absence of speech, keeping non-speech at a downgraded status), this work is the only one of the three that uses audio type as an "adjustable ratio variable" rather than a "keep/discard criterion." In theory this is a more refined approach — it does not sacrifice data diversity, adjusts each category's proportion via sampling weights, and the ratio can be adjusted without re-cleaning the data. But this advantage depends entirely on the soundness of the ratio strategy, and the paper says nothing whatsoever about the ratio strategy, making the actual value of this design impossible to assess.
[Type-discrimination capability on the model side] Text conditioning is injected via the CLAP encoder; users can in theory influence the output by describing the desired sound type in the prompt, but the paper does not evaluate the model's ability to respond to negative instructions such as "sound effects only, no music."

### [HunyuanVideo](../models/HunyuanVideo.md) ⚠️

Not applicable. No classification or separate handling of speech/sound-effects/music. [Uncertain]

### [InstructAV2AV](../models/InstructAV2AV.md) ⚠️

Audio is split in two — "speech vs. non-speech sound-source events" — and the two categories go through entirely independent processing chains; this fork runs through the three stages of admission filtering, instruction generation, and audio synthesis, and is the clearest classification-driven design in the pipeline.

[Classification determination and basis] The paper does not explicitly state which classifier performs speech/non-speech discrimination; inferring from the tool chain, it is implicitly determined by the output of TalkNet (active speaker detection) — detecting an on-screen active speaker routes to the speech path, otherwise to the event path. [Uncertain] Neither the specific implementation of the classification nor the sample ratio between the two categories is disclosed.

[Processing chain of the speech path]
  Admission: TalkNet locates the active speaker → Scribe extracts precise speech timestamps → hard condition: "speech must be time-aligned with the visible on-screen speaker," otherwise discarded.
  Instruction: Qwen3-Omni generates speech-editing instructions, with the target line wrapped in <S>...<E> markers embedded in the instruction.
  Synthesis: ElevenLabs speech synthesis, split by subset type into modes such as identity-preserving speech modification (keeping the original timbre while changing content) and clone_voice (cloning a new timbre).
  Evaluation: targeted measurement via Sync-C / Sync-D (SyncNet lip sync).

[Processing chain of the non-speech event path]
  Admission: remove segments with ambiguous sound sources → assign each retained segment a unique semantic sound-event label (e.g., "dog barking") based on its dominant sound source.
  Instruction: Qwen3-Omni generates entity edit/insert/remove instructions based on the instance mask and the sound-event label.
  Synthesis: SAM-Audio separates the target entity's sound → ElevenLabs text-to-audio synthesizes the new sound → mixed with the background sound.
  Evaluation: targeted measurement via PEAVS (general event synchronization) and AV-A (cross-modal semantics).

[The absence of music] [Uncertain] The paper does not list music as a separate handling category. This contrasts with Foley-Omni's three-way speech/effects/music split. It is speculated that music (given that the source movie footage commonly carries a score) is classified under the "background sound" left intact after SAM-Audio separation — i.e., music is protected background rather than an editing target, which is consistent with the task definition (there is no editing task such as "change the score from sad to cheerful"). But the paper does not state this explicitly, nor does it discuss whether the presence of a score interferes with SAM-Audio's separation of the target entity's sound.

[Assessment of the two-way vs. three-way trade-off] This work's two-way split is task-oriented: the editing target can only be "a person speaking" or "an object producing sound"; music has no corresponding visual entity that a mask can locate, so it naturally falls outside the editing scope. This partition is simpler than Foley-Omni's three-way split, but is sufficient for this task.

### [2026 Other Audio-Video Joint Generation Works](../models/JAVG_2026_misc.md) ⚠️

[ALIVE — top-level speaking/non-speaking binary drives everything (the most systematic in this batch)]
(1) The very top level of data organization is a binary audio-type split: "First, we make a top-level distinction between core scenarios: 'speaking' and 'non-speaking' scenario." — the corpus is first split in two by "whether or not someone is speaking," and a three-level label set across nine major domains is layered on top of that. This ordering matters: it shows that audio type is a higher-priority organizing dimension than visual domain, consistent with the essential needs of AV models (speaking scenes need lip sync and timbre consistency, non-speaking scenes need event-level audio-visual alignment — the two have entirely different modeling difficulties).
(2) Corresponding splitting at the caption level: the <W> marker for verbatim speech, the <I> marker for non-speech acoustic events, and acoustic profiles within Subjects describing timbre attributes.
(3) BGM is handled separately: as a representative of "weakly correlated" audio, its proportion is actively managed rather than removed.
(4) Two-stage type-switching in audio-tower training: Stage I is pure speech (700k hours of transcribed speech, 1 epoch) → Stage II is a mix of speech + real soundscapes (5k hours of high-quality speech + 111k hours of video audio tracks, 10 epochs).
(5) Among the nine Level-1 domains, Home Sounds and Sound Effects are two categories dedicated to non-speech audio, and every Level-3 visual label is bound to audio keywords — visual concepts and sound concepts are indexed in pairs.

[NAVA — five audio-type labels (the finest-grained classification in this batch)]
YAMNet + an omni-modal tagger divide audio into five categories: single-speaker speech, multi-speaker speech, ambient sound, music, and singing. Compared with the common three-way "speech/sound-effects/music" split, this five-way scheme has two well-targeted refinements:
- Separating single-speaker speech from multi-speaker speech: directly serves its multi-speaker timbre control capability (Timbre-in-Context Conditioning needs to know how many speakers there are and where each one's speech span lies)
- Separating music from singing: singing simultaneously has speech-like properties (lyrics, timbre) and music-like properties (melody, rhythm); listing it as its own category avoids contaminating the modeling of pure speech and pure music.
The audio section of the caption is also described across four categories: speech / SFX / music / ambient sound. During training, an "audio-only : audio-visual" ratio schedule (3:1 in Stage 1, 1:2 in Stage 2) indirectly controls exposure to each audio type.
[OmniCustom — a single type (pure speech)] All content is single-speaker speech, with no sound-effects/music handling. Audio captions are organized by speaker attributes (age/gender/accent/pitch/prosody/emotion/speaking rate). This is the item with the single most narrow-focused audio type in this batch, but precisely because of that its timbre control is the deepest.
[StreamChar — predominantly speech] Emilia (a pure-speech, multilingual dataset) is used to pretrain the orchestrator, with joint-training data consisting of character-speaking videos. No sound-effects/music type design [Uncertain].
[CCL — general audio pretraining, speech-leaning joint training] Audio-diffusion pretraining uses WavCaps (audio descriptions) + VGGSound (video sound effects), covering general sound effects; the joint-training data (OpenHumanVid + interviews/short dramas/movies) is predominantly dialogue. This forms a structure of "general-audio grounding + speech-scene fine-tuning." In its multi-task probability distribution, text-to-audio accounts for 0.1 and video-to-audio for 0.15, indicating that pure-audio generation capability is explicitly retained.
[Baton] AudioCaps + WavCaps provide general sound effects/soundscapes, OpenHuman-Vid provides human voice; no explicit type-splitting strategy [Uncertain].
[ITS-JAVG] The two base models tested emphasize different audio types: MMDisCo is based on VGGSound (predominantly sound effects), while JavisDiT has broader coverage; in its verifiers, ImageBind-TA and AVHScore treat speech and sound effects identically, with no type splitting done [Uncertain].

### [Audio-Video Joint-Generation Baseline Collection](../models/JavisDiT_baselines.md) ⚠️

The handling strategy for the three audio categories — speech, sound effects, and music — is the key that differentiates the positioning of each work in this collection:
[MM-Diffusion / AV-DiT — binary and mutually unmixed] Landscape corresponds to "ambient sound effects," AIST++ corresponds to "music"; the two datasets each independently train a separate model (in the repository, Landscape.pt and AIST++.pt are two independent sets of weights), with no mixed training. Speech is not involved at all. This "one domain, one model" approach was an inevitable choice in the small-data era.
[JavisDiT / JavisDiT++ — full-category mixing at the audio stage, deliberate exclusion of speech at the video stage]
- Audio pretraining (780,000 items): explicitly pursues full-category coverage — "no filtering is applied in order to maximize T2A capability, covering the three categories of general sound, music, and speech." The type division of labor among data sources is clear: AudioSet/AudioCaps/VGGSound/WavCaps/Clotho provide general sound effects, GTZAN/MusicInstrument provide music, UrbanSound8K/ESC50 provide ambient sound and urban noise, and MACS provides acoustic scenes.
- Audio-video SFT: FunASR is used to remove most videos containing speech — i.e., the speech category is actively abandoned at the joint-generation stage. This is the clearest "audio-category trade-off" decision in this collection; its cost is that the model lacks lip-sync and dialogue-generation capability, and its benefit is that all limited compute and data are devoted to event-level alignment of ambient sound/sound effects (which is also the focus of JavisBench's evaluation).
[Harmony — speech and ambient sound strictly 1:1, each equipped with a different encoder]
- Data side: 2 million speech items vs. 2 million ambient-sound items, mixed 1:1, with this ratio maintained in both Stage 1 and Stage 3.
- Architecture-side differentiation: the audio VAE encoder uses MMAudio (skilled at general audio/sound effects), while the speech encoder uses F5-TTS (skilled at speech) — the two audio types go through different encoding paths, making this the only work in the collection that distinguishes audio types at the encoder level.
- Training-side differentiation: in Stage 2's timbre decoupling, the two categories of data use different pairing strategies (speech uses cross-utterance pairing from the same speaker, ambient sound uses non-overlapping-segment pairing from the same clip).
- Evaluation-side differentiation: Harmony-Bench's three-tier subsets correspond respectively to ambient sound, speech, and the co-occurrence of both.
It can be said that Harmony is the only work in this collection that systematically treats "speech" and "non-speech" as two parallel data streams handled separately throughout, while still insisting that the two must be trained jointly.
[UniAVGen — a speech-only route] Stage 1 pretrains on Emilia (a pure-TTS corpus), with the entire pipeline focused on real human speech; no description of handling sound effects or music is found [Uncertain]. This gives it an advantage in dubbing, lip sync, and timbre/emotion consistency, but its general sound-effects generation capability is questionable.

### [Kling 3.0 / Kling Video 3.0 Omni](../models/Kling_30_Omni.md) ⚠️

[Uncertain, though the same team has a published paradigm] Kling-Foley explicitly first uses an audio classification model to split material into four categories — sound effects, music, speech, and singing — then calls a large audio-understanding model separately for each category to generate descriptions, applies different annotation and filtering rules to each, and finally merges them uniformly. KlingAvatar 2.0 likewise builds data and evaluation separately by Chinese speech / English speech / singing. Kling 3.0 Omni's capability covers three controllable categories — dialogue speech, ambient sound, and sound effects (each individually specifiable) — strongly suggesting the same classify-and-conquer processing exists on the training side, but the specific classifier and mixture ratio have not been officially disclosed.

### [LTX-2](../models/LTX-2.md) ⚠️

At the caption-annotation level, four audio categories are explicitly distinguished and described separately — speech (precise transcription + speaker/language/accent), music, ambient/atmosphere (ambient, background), and Foley effects — and this is the basis on which the model can respond separately to instructions for the four categories.
But at the level of data filtering and training ratios, the paper does not state whether filtering rules, ratio targets, or loss weights are designed separately for the four categories — "audio informativeness filtering" is a single unified threshold, with no evidence of type-based splitting. Architecturally, audio is uniformly modeled as a single 5B stream, with no type-based branching or expert routing.
Indirect evidence shows category imbalance exists: the model card's Limitations section explicitly states that it "produce[s] lower-quality audio when generating non-speech content," indicating that speech-category data dominates the subset, with sound-effects/music categories relatively insufficient. The paper also acknowledges that deep text understanding mainly serves the phonemic accuracy of speech, with the emphasis clearly skewed toward speech. [Uncertain]

### [LongCat-Video](../models/LongCat-Video.md)

Not applicable to the base version.
Avatar 1.5 applies a coarse-grained, function-based split of audio rather than a strict three-way speech/sound-effects/music classification: (1) speaking speech is the main line, going through the full lip-sync + active-speaker-detection chain; (2) singing/music is listed as its own branch (sourced from music videos), used to cover singing mouth shapes and rhythmic body movement, whose lip-motion dynamics differ from ordinary speaking; (3) silence (no speaking) is built as a separate subset, using dual-model consistency judgment to guarantee "genuinely not speaking," so the model learns cases where audio is present but the subject makes no sound; (4) intervals of concurrent multi-person speaking are removed and not retained as a category. Sound effects/Foley and ambient sound are not handled as independent categories — consistent with its positioning: audio is merely a driving condition, and the model does not generate audio, so there is no need to cover the full range of soundscape categories.

### [MOVA](../models/MOVA.md)

The classification tooling and splitting strategy are explicit, and run through three levels: data, training, and capability.
[Classification tooling] EAT (Efficient Audio Transformer, a self-supervised pretrained audio model) serves as the audio classifier, tagging segments with speech/non-speech-related labels. The condition for constructing the speech subset is: both the EAT-contained-Speech and EAT-contained-Singing labels are judged True (or meet the model's positive-class confidence). Silero VAD additionally performs speech/non-speech interval segmentation during preprocessing.
[Purpose of the split] The paper explicitly states that the split is "depending on the target capability (e.g., lip synchronization vs. general foley/ambience modeling)" — that is, subsets are built separately for the two target capabilities of "lip sync" and "general foley/ambience modeling."
[Actual strategy: two-stage division of labor]
- Music and sound-effects capability are injected during the pretraining stage of the 1.3B audio tower: general sound effects come from WavCaps + VGGSound, music comes from JamendoMaxCaps, and speech comes from in-house TTS.
- Speech and lip-sync capability are reinforced during the joint-training stage: joint training ultimately uses only speech segments (69.47% of preprocessed segments).
[Splitting at the annotation side] Audio annotation is likewise divided by type — Qwen3-Omni-Instruct performs only speech transcription (strictly forbidden from including non-speech or music), while Qwen3-Omni-Captioner performs only descriptions of non-speech sound effects and music; when merged, non-speech content is restricted to four categories (ambient sound, music, audio theme/sound source, structural audio changes) and human voice or words are strictly forbidden from appearing.
[Cost] This ratio of "heavy on speech, light on music" is explicitly acknowledged as a limitation in the Limitations section: the model's performance degrades on singing, complex sound textures, and music/instrumental content, because the audio tower is only 1.3B and lacks the capacity to carry fine-grained pitch/harmonic structure and long-range temporal dependencies. The paper further notes that the model lacks physical acoustic reasoning (e.g., the propagation delay between lightning and thunder is not explicitly modeled or enforced by the data).

### [Merged entry: ① Mochi 1](../models/Mochi_MAGI_Motif.md)

Not applicable. No classification or separate handling strategy for speech/sound-effects/music.

### [Movie Gen](../models/Movie_Gen.md)

Classification: AED (the 527-category AudioSet ontology) maps each sample into three categories — voice (speech + singing subclasses), non-vocal music (music subclass), and general sound (remaining subclasses) — allowing multiple labels to co-occur; crossing this with diegetic/non-diegetic yields the six cells of Table 23 (e.g., diegetic voice = live speaking, non-diegetic voice = narration; diegetic music = live performance, non-diegetic music = background score; diegetic sound = waves crashing, non-diegetic sound = canned laughter and risers).
Handling strategy:
· No model separation for modeling — the paper explicitly chooses to "jointly generate all audio categories with a single model," reasoning that correlation exists not only between video and audio but also among different audio categories (if generated separately, sound effects and music could not be mixed harmoniously).
· Category-specific threshold and quota tuning for data ratios: general sound is designated the highest priority (low-level physical regularities are hardest to learn and errors are most jarring), and is roughly 10x higher in scale than other categories; the diegetic-determination threshold for the music category is raised to 0.3; the voice category is excluded from the generation target (used only as a presence label).
· Field-specific control at the caption level: speech/singing use binary labels, music uses a continuous probability, general sound effects use free-form description, and musical style occupies its own segment appended unconditionally.
· Source-specific data for fine-tuning: film/TV-grade audio-video (containing both diegetic sound and non-diegetic ambient/theme music) + high-quality pure-music O(10)K hours and pure-sound-effects O(10)K hours without accompanying video, the latter being easier to obtain in quantity and used to raise audio quality. Ablations show that adding large-scale text-sound-effects and text-music paired data lets the model effectively decouple different audio types.

### [NVIDIA NeMo Curator](../models/NeMo_Curator.md) ⚠️

No classification/splitting mechanism. The implicit assumption of the audio pipeline is "input equals speech data" — all stages (ASR transcription, WER/CER filtering, VAD, speaker separation, SQUIM quality assessment, punctuation preparation) serve speech, with no distinction or separate handling of sound effects (Foley), music, or ambient sound. The framework provides no audio event classifier (such as the AudioSet label system), music detector, or source-separation module.
The audio tagging stage in 26.07 is the closest capability, but in the documentation it is positioned as speech-attribute annotation rather than classification/splitting across the four audio categories. [Uncertain]

### [OmniHuman Dataset + OHBench](../models/OmniHuman.md) ⚠️

[Core mechanism: binary recombination after Demucs four-source separation] The original mixed audio track is separated by Demucs into four sources (vocals / drums / bass / other), then recombined into two tracks: the vocal track (target) and the background track (a mix of the other three sources). This is the infrastructure underlying how this dataset handles audio types.
[Speech] Extracted separately as the target track, subjected to a full processing chain: 3D-Speaker separation → FunASR-Nano transcription → SyncNet attribution matching → emotion annotation → timestamps → speech quality scoring. Speech is the audio type with the heaviest annotation.
[Music] Not removed; instead, three attributes are annotated: type, mood/atmosphere, and relative volume. "Relative volume" turns the energy ratio between music and voice into a filterable dimension. The music signal itself remains fully intact within the background track.
[Background sound/ambient sound] The background sound labels field in video-level annotation; the specific label system (whether free text or a closed category set) is not stated. [Uncertain]
[Sound effects] Explicitly listed in the Stage 1 attribute-extraction checklist (appearance / motion / expression / speech / music / sound effects), indicating that Qwen3-Omni does extract sound-effects information; but no corresponding independent annotation field is described, and it is unclear whether it ultimately gets merged into the caption narrative or is kept as a separate field. Compared with Foley-Omni's dedicated three-field design, sound effects is a secondary dimension here — consistent with the dataset's person-centric positioning. [Uncertain]
[Silence] Filtered as one of the low-quality conditions (a silence-proportion threshold), not retained as an independent category.
[Ratio across types] No duration/proportion statistics are given for any type. [Uncertain]
[Comparison with two existing routes] UniTalking follows "keep only speech, discard everything else" (a purification route); UniVerse-1 follows "bucket by speech/non-speech and control the ratio at 15.4% : 84.6%" (a ratio route); OmniHuman follows "no discarding, no bucketing — decompose via source separation and annotate each part separately" (a decoupling route). The third route has the highest data utilization and is the most flexible for downstream use, at the cost of shifting responsibility for the ratio entirely onto the user — the dataset itself makes no commitment about soundscape composition.

### [Open-Sora series](../models/Open-Sora.md)

Not applicable. No classification or differentiated handling of speech/sound-effects/music.

### [Ovi](../models/Ovi.md) ⚠️

One of Ovi's core claims is that "a unified audio model must cover both speech and sound effects simultaneously" rather than divide-and-conquer, yet its data organization still handles the types separately:
[Type-specific data sources] Speech mainly comes from an internal corpus (the bulk of pretraining); sound effects come from VGGSound / AudioSet / WavCaps (introduced during fine-tuning); no independent source for music is found — it mainly appears in the form of BGM within videos' native audio tracks [Uncertain].
[Type-specific training stages] Audio pretraining is predominantly speech-based (learning speaker identity, pitch, emotion, prosody); the audio fine-tuning stage introduces diverse sound effects while also mixing back in internal audio-video tracks to keep TTS capability alive, making Ovi-Aud a unified audio backbone capable of both T2A and TTS.
[Type-specific annotation] A binary split of "contains speech / does not contain speech": for items containing speech, the audio description focuses on speaker acoustic attributes; for items without speech, it focuses on three categories — sound effects / background audio / musical elements. In pure-audio data, if there is no speech, the transcription field is left empty.
[No type separation at conditioning encoding (a key decision)] An early version once used CLAP to encode non-speech descriptions and T5 to encode speech transcriptions in order to "decouple T2A and TTS"; ablations showed this separation scheme caused the model to be unable to blend sound effects and speech into a coherent audio track, so it was ultimately changed to merge everything into a single T5 embedding.
[Type-specific evaluation] On the T2A side, results are reported under the MMAudio protocol: FD_PANNs 18.03 / FD_VGG 5.02 / IS 11.20 / CLAP 0.224; on the TTS side, WER 0.035 is reported on Seed-TTS test-en. Both are close to their respective domain-specialized models (e.g., MMAudio-L's FD_PANNs 15.04, F5-TTS's WER 0.018); the paper argues that a unified model need not surpass specialized models — being simultaneously competent at both is the precondition for audio-video fusion.

### [Script-a-Video](../models/Script-a-Video.md) ⚠️

Classification and differentiated handling of audio types is one of the most clearly defined parts of MTSS, adopting a four-tier partition of "three event categories + one global category":
[dialogue (dialogue/speech)]
- Belongs to: the Event stream;
- Dedicated fields: speaker (bound via relation to a Reference ID, making explicit "who is speaking") and line (verbatim dialogue text);
- The description field additionally carries paralinguistic information such as emotional shifts and vocal techniques;
- This is the richest of the three categories in terms of fields, reflecting its core position in audio-video generation (both lip sync and identity binding depend on it).
[sfx (sound effects)]
- Belongs to: the Event stream;
- Dedicated admission constraint: must be produced by a subject visible in the frame ("sound effects must be generated by a visible subject") — this is the only category among the three with a hard visual-evidence requirement, directly serving Foley-style generation's need for "sound-action causal relationship";
- No speaker or line field, only description.
[music]
- Belongs to: the Event stream (when music constitutes an independent narrative event);
- Forms a layered relationship with the background music within global_audio: the former is music with a clear start/end that participates in the narrative (e.g., someone performing on screen, or a score entering as a plot point), while the latter is atmospheric scoring that underlies the entire piece and does not constitute an independent event.
[global_audio (global audio, the fourth tier)]
- Belongs to: the Global stream;
- Carries: ambient sounds and background music that do not qualify as independent events, as well as irrelevant background noise that has been filtered out;
- Positioning: a catch-all container that preserves soundscape information without polluting the signal-to-noise ratio of the Event stream.
[Concurrent sound sources] Split into parallel event entries, each independently carrying type, time_range, and a content block, with no merged description. This allows scenarios such as "a person talking while footsteps sound nearby" to be expressed fully and with separate controllability.
[Differentiated validation on the generation side] The paper evaluates speech-related capability using three separate metrics — WER (dialogue-content accuracy), UTMOS (speech quality), SyncNet (lip sync) — and evaluates sound effects using qualitative RMS envelope analysis. Figures 7/8 show that the baseline produces a flat, ambient-noise-like envelope, whereas the MTSS scheme exhibits the rhythmic fluctuation of natural speech and the periodic impacts of action-driven events (such as the periodic impact of footsteps). The paper explicitly states that the Event stream transforms speech and sound-effects generation "from nearly random to semantically accurate."
[Ratio on the data side] The proportion of the three audio categories in the training data is not disclosed, nor is any mention made of adjusting sampling weights by category. [Uncertain]

### [Seedance 2.0 and Seedance 1.5 pro](../models/Seedance_20_Seedance_15_pro.md) ⚠️

The classification and separate-handling strategy on the training side is not disclosed, but a clear three-way division is reflected on both the product and evaluation sides: Seedance 2.0 supports parallel multi-track output of three audio-track categories — background music (BGM), ambient sound effects, and character narration/voiceover — each precisely aligned with the visual rhythm, which architecturally requires the training data to have the ability to separate or annotate by audio-track type. The evaluation label system is likewise divided this way: SeedVideoBench 1.5 splits audio into voice type, voice attributes, and non-speech audio (sound effects + music, further subdivided by sound source, acoustic attributes, music genre, and technical parameters); among SeedVideoBench 2.0's 17 categories, the boundaries are clear between the speech category (dialect / multi-person conversation / variety show / traditional opera / English / minority languages / singing-rap / voiceover / non-verbal vocalizations), the sound-effects category (voice + action-interaction sound / object-interaction sound / animal calls / ambient background sound / special-effects ASMR), the music category (instruments and audio), and the spatial category (spatial-scene sound / binaural audio). [Uncertain: the actual classifier and processing branches on the training-data side]

### [SkyReels series](../models/SkyReels.md) ⚠️

SkyReels-V4 adopts a clear strategy of "classify first, then split and process by category," which is the backbone of its audio-side methodology:
(1) Classification: Qwen3-Omni judges audio into four categories — sound effects, music, speech, and singing;
(2) Split-processing differences: speech and singing → go through Whisper transcription, with the transcribed text entering <dialogue> / <singing> tokens and required to pass SyncNet lip-audio sync filtering; sound effects and music → not transcribed, with Qwen3-Omni generating a descriptive caption entering <sfx> / <bgm> tokens;
(3) Duration concatenation is performed by category — only audio of the same category is allowed to be concatenated to reach 15 seconds, avoiding semantic contamination from cross-category mixing;
(4) The four auditory tokens of the caption schema correspond strictly to the four categories, enabling the model to control by category independently at generation time.
[Not disclosed] The sample size and training ratio of each of the four categories, whether different quality thresholds are set for different categories, and whether loss weights are weighted by category. It is known that the audio-backbone pretraining data is "predominantly speech," so the four categories are inherently imbalanced. SkyReels-V2 does not involve audio. [Uncertain]

### [Sora 2](../models/Sora_2.md) ⚠️

Completely undisclosed. On the model-output side, four categories — dialogue, sound effects/Foley, music, and ambient sound — are clearly distinguished, but OpenAI provides no statement whatsoever as to whether these four categories are explicitly classified on the training-data side, or whether filtering/ratio/annotation strategies are separately designed for each. [Uncertain]

### [SpeakerVid-5M](../models/SpeakerVid-5M.md)

SpeakerVid-5M does not perform classification or separate handling of audio types — it excludes non-speech types wholesale via filtering, rather than classifying and then treating them differently:
[Actual strategy: convergence to a single type] The rule of discarding anything with no-speech probability > 0.8, combined with a lower bound on ASR confidence, causes the final corpus to converge, in terms of audio type, to a single "human speech" category. Sound effects (Foley), music, ambient sound, and silent segments are not classified and retained but are directly discarded. Consequently, there is no audio-type branch within the dataset that would require "separate handling."
[Tools not used] No audio-type classifier (compare MOVA, which uses the EAT self-supervised audio Transformer to build speech/non-speech subsets and split by target capability), no audio source separation, no dedicated annotation or filtering path for music or sound effects.
[The only type-related design is a role dimension, not an acoustic dimension] The dataset's branch division (talking / listening / dialogue / multi-turn) distinguishes "speaker role and interaction form" — the audio tracks of all four categories are speech, and this does not constitute an acoustic-type split. In the listening branch, the on-screen subject does not speak but the audio track is still the other party's speech — this is a mismatch of audio-visual role, not a difference of audio type.
[Downstream consequence] A model trained using this dataset can only acquire speech-lip-sync capability, and cannot learn Foley, music, or ambient-sound generation. This is consistent with MOVA's practice: MOVA positions SpeakerVid-5M as the source of lip-sync capability, while general sound-effects and music capability are separately injected from datasets such as VGGSound, WavCaps, and JamendoMaxCaps during the audio-tower pretraining stage. This pattern of "acquiring different audio capabilities via a division of labor across datasets" is precisely SpeakerVid-5M's positioning in the ecosystem as a vertical corpus.

### [Step-Video-T2V](../models/Step-Video-T2V.md) ⚠️

Not applicable. No classification or separate handling strategy for speech/sound-effects/music. StepFun's speech-side capability is encapsulated in an independent Step-Audio model, which has no overlap with this entry's video data pipeline. [Uncertain]

### [UniTalking](../models/UniTalking.md)

UniTalking adopts an extreme strategy toward the three audio categories of "speech only, discard everything else" — no classify-and-retain, no separate handling:
[Speech] The only category retained, and it undergoes four rounds of conditional filtering: non-silent → contains speech (PANNs + SentenceASD) → SNR meets the standard → sound source is on screen (LightASD) → lip-audio sync (LipSync). The final 2.3 million samples are all human-speaking content.
[Sound effects (Foley) and ambient sound] Not retained as an independent category. Samples containing only ambient sound with no speech are discarded by the "lack speech" rule at the second stage. This is the exact opposite of UniVerse-1's approach of "downgrading no-speech samples and retaining them as general data (3,074 hours) + additionally introducing 3,422 hours of VGGSound/AudioSet to reinforce sound effects." Paralinguistic sound effects (such as laughter) are retained as an accompanying component of speech samples (Figure 4 shows the controllable generation of "a short laugh").
[Music] Not handled as an independent category; no music dataset is introduced; there is no dedicated development of music capability. If background music accompanies speech, it is passively retained, with no source separation performed; instead, it is honestly described by the caption and becomes an attribute controllable via prompt (e.g., "without any background music" is effective).
[Classification mechanism] There is only one binary gate (presence/absence of speech), with no multi-class audio classifier (compare MOVA, which uses the EAT self-supervised audio Transformer to perform speech/non-speech classification). No further subdivision is made within non-speech.
[Differentiated handling] There is no differentiated loss or sampling weight by audio category (compare UniVerse-1, which applies source-differentiated LQLS loss to VGGSound/AudioSet). All data is treated equally.
[Direct consequence for capability boundaries] The model's audio capability is entirely concentrated on speech synthesis and voice cloning, with no sound-effects or music generation capability. The paper's conclusion asserts that "the framework can be generalized to general audio-video synthesis, including sound effects and music," but this claim is supported neither by data (no such content is in the data) nor by experimental validation — it is purely aspirational.
[Relationship to Stage-1 data] The audio branch's foundational articulation capability comes from Stage-1 pretraining on "internal TTS data," which is likewise pure speech data — that is, the model has seen only speech throughout, from pretraining to joint training; the singularity of its audio capability is a consistent design running through both stages.

### [UniVerse-1](../models/UniVerse-1.md)

The classification and handling strategy for the three audio categories — speech, sound effects, and music — is as follows:
[Classification mechanism] Only one binary gate — Whisper ASR detects whether a segment contains recognizable speech:
- Speech detected → enters the speech subset, further undergoing RetinaFace face detection and SyncNet conf>2.0 verification; upon passing, it is explicitly labeled "contains speech," yielding a final 1,187 hours;
- No speech detected → not discarded, but downgraded and classified into "general audio-video data," yielding a final 3,074 hours.
No dedicated audio classification model is used (compare MOVA, which uses the EAT self-supervised audio Transformer for speech/non-speech classification), and no further distinction is made within non-speech among sound effects, music, and ambient sound — these three categories are merged into a single "general/ambient sound" bucket within UniVerse-1's data system.
[Sources of division of labor for the three capabilities]
- Speech and lip-sync capability: comes from the 1,187-hour speech subset that underwent triple verification, and from the architectural decision to remove the speaker encoder for the sake of generalization;
- Sound-effects/ambient-sound capability: comes from 3,074 hours of general data + 3,422 hours of VGGSound/AudioSet (the latter being an event-sound annotation dataset introduced specifically to reinforce sound effects);
- Music/instrumental capability: mainly inherited from the Ace-step base model (itself a music-generation model), reinforced on the data side by music variety-show and classical-music performance footage, though no separate hour count is given.
This is a three-part division of labor — "music relies on the base model, sound effects rely on public datasets, speech relies on self-collected fine curation" — similar in approach to MOVA's two-stage division ("sound effects/music injected during audio-tower pretraining, speech reinforced during joint training"), but UniVerse-1 entirely outsources the acquisition of music capability to the base model, doing no dedicated curation of music data itself.
[Type separation on the annotation side] In the caption schema, speech content and ambient-sound description are two independent fields, so the model can distinguish "what is said" from "other sounds" at the text-conditioning level, but the ambient-sound field is not further subdivided internally into sound effects/music/ambient sound.
[Differentiated loss] VGGSound/AudioSet, due to low visual quality, has LQLS applied to it (Flow Matching loss is computed only over the high-noise segment where t>800) — this is the only place where differentiated training treatment is applied by data source, essentially "letting low-quality data teach only semantics, not detail."

### [Unison](../models/Unison.md) ⚠️

The handling strategy for the four audio categories — speech, sound effects, music, and singing — is the most systematic part of Unison's entire data design:
[Classification mechanism: no classification judgment, but forced decomposition] All audio, without exception, is decomposed via Mel-RoFormer into two paths — speech and sfx — with no classification step that "determines which category this segment of audio belongs to." This is fundamentally different from UniVerse-1 (Whisper performing a binary speech/non-speech split) and MOVA (the EAT self-supervised audio Transformer performing speech/non-speech classification): the latter two are "classify, then take different paths," whereas Unison is "every sample takes both paths." The benefit is no risk of classification error and no path-imbalance problem; the cost is that even a sample with no speech must still maintain an empty speech stream, and everything depends entirely on the quality of the separation model.
[The pure-audio corpus is collected from sources divided by the four categories, each with its own role]
- Speech: internal speech data → trains the speech-generation capability of the audio branch enhanced by Zipformer;
- Sound effects: YouTube-8M + AudioSet + WavCaps → three datasets stacked together, the most heavily covered part on the sound-effects side;
- Music: VidMuse → instrumental/scoring capability;
- Singing: the YuE collection → specifically reinforces singing capability.
[The significance of singling out singing as an independent category] This is a piece of differentiated investment by Unison relative to peer works. In its qualitative comparison, the paper explicitly notes that UniVerse-1 and UniAVGen "fail to synthesize musical accompaniments and struggle to distinguish singing from standard speech," and that MOVA "can generate believable vocals in music scenes but shows obvious speech artifacts under complex conditions." Introducing the YuE singing corpus is precisely aimed at overcoming this named failure mode. At the same time, the paper repeatedly uses "singing while playing an instrument" as the representative example of the most difficult scenario — the simultaneous occurrence of singing (structured vocal sound) and instrumental music (structured non-vocal sound) is the hardest boundary case for the speech-sound-effects binary to handle.
[Type division of labor on the generation side] The speech stream carries speech and singing (both vocal sound), while the sfx stream carries sound effects, music, and ambient sound. Note that singing and music fall onto two different streams respectively — this is precisely the natural result of Mel-RoFormer's (a vocal-separation model) separation logic: it splits by "vocal vs. non-vocal" rather than by "semantic category." This partition happens to fit the "singing while playing" scenario well (vocals go into the speech stream, accompaniment into the sfx stream) — a coincidental fit between architecture and tool choice.
[Runtime type-adaptivity] SCG gating dynamically adjusts the ratio between the two streams at inference time based on caption/transcription semantics: narration-dominated scenes suppress the injection of the sfx stream into the speech stream to protect "phonetic purity," while complex-soundscape scenes amplify cross-stream influence to enrich non-speech acoustics. The instance-level analysis in Fig. 8c gives a concrete case of sports-event commentary — the gating constrains the commentary stream to prevent it from masking the high-frequency transients of the stadium atmosphere (impact sounds and crowd cheering).
[Not disclosed] The hour count and proportion of each of the four audio categories, the sampling weights during training, and the curriculum arrangement or mixing ratio of the four categories of data within Stage 1. [Uncertain]

### [Veo 3 / Veo 3.1](../models/Veo_3_Veo_31.md) ⚠️

[Uncertain] No classification or separate-handling strategy for speech/sound-effects/music is disclosed. At the model-output level, four categories — dialogue, sound effects, ambient sound, and background music — are clearly distinguished and can each be controlled via prompt, indicating that these categories are described separately in the training data's captions; but whether an independent audio classifier exists, and whether different filtering thresholds or training weights are applied to each audio category, is entirely unpublished.

### [Vidu S1](../models/Vidu_S1.md) ⚠️

Speech is the only audio type treated as the core of training and control: speech components are explicitly extracted, VAD/ASD/speaker-attribution classification is performed, and speech is used as the real-time control signal for streaming generation. Music and singing are treated as noise sources and are systematically removed via a heuristic rule based on speech-energy proportion. Sound effects and background music do not participate in dedicated modeling, but are retained as independent description fields (sound effects, background music) within the structured caption. On the product side, users can select different voice tones, suggesting that TTS/timbre control is connected on the audio side, but the paper does not elaborate on this [Uncertain].

### [Wan 2.5 / 2.6 / 2.7](../models/Wan.md) ⚠️

The Wan series has a clear divide-and-conquer strategy for audio types, and has undergone one notable strategic reversal.
[Stage 1 (Wan 2.1, Feb 2025) — three-way divide-and-conquer excluding speech]
- Ambient sound: retained, described in its own dedicated segment within the caption;
- Background music: retained, analyzed in its own dedicated caption segment across four attributes — style/rhythm/melody/instrumentation;
- Speech and vocal music: the entire video is removed directly from the V2A training set.
These three categories are generated and classified by Qwen2-Audio for audio captions; during training, random masking is applied to the captions of the ambient-sound and music categories to strengthen visual→audio associative learning. The report candidly states that this results in the model being unable to generate laughter, crying, or speaking sounds, and lists speech generation as future work.
[Stage 2 (Wan2.2-S2V, Aug 2025) — pivoting to speech-dedicated] The entire model is built around speaking/singing scenes, with the data pipeline centered on active speaker detection — a dedicated accumulation of speech-category data capability.
[Stage 3 (from Wan 2.5 onward, Sep 2025+) — unified joint generation of all three categories] The official description states that 2.6 achieves "perfect matching between the visuals and voice, sound effects, and BGM" — that is, the three categories of speech (voice/dialogue), sound effects (Foley), and background music are generated uniformly within the same model and synchronized with the visuals; when no audio is provided, "matching background music or sound effects" is generated automatically, and when audio is provided, it drives lip motion and performance.
[Gaps] The ratio of the three audio categories within the 2.5+ training set, whether type-based splitting (such as separate filtering rules, loss weights, or expert routing) is still applied, and the handling strategy for silent samples are all undisclosed. [Uncertain]

### [Audio-Video Generation Evaluation Benchmark Collection](../models/av_benchmarks.md) ⚠️

[AV-SyncBench has the clearest classify-and-conquer approach] It splits and processes across three major categories — Voice / Music / Sound: for semantic perturbation, the voice category goes through OpenVoice V2 timbre substitution, the instrument category goes through DDSP timbre transfer, and the general sound-effects category undergoes no semantic perturbation at all (hence only 821 semantic-challenge samples, far fewer than the 37,569 temporal-challenge samples). This shows that the difficulty of controllable editing varies greatly across audio types — speech and music have mature timbre-decoupling tools, while general Foley sound effects lack them; this reality also constrains the feasibility of constructing Foley negative samples on the training-data side. The ten scene labels are further subdivided into single-source and multi-source (single person speaking vs. dialogue/group vocalization, single instrument vs. ensemble).
[VABench applies differentiated metrics by category] The lip-sync metric is applied only to the voice-linguistic subset and requires a talking head to be detected; speech-quality metrics (DNSMOS, NISQAv2) are meaningful only for samples containing speech; the Audio Realism physical-plausibility score explicitly excludes the virtual-world category. This mechanism of "applying metrics conditionally by audio type" corresponds, on the training-data side, to setting different quality thresholds by domain.
[PhyAVBench focuses on Foley physical sound effects] All six of its dimensions revolve around non-speech physical sound-production mechanisms, with speech only indirectly involved via WER; this fills the evaluation gap regarding physical correctness for Foley-type sound effects.
[AVBench focuses on speech] Of its 10 dimensions, three target speech directly (content accuracy, realism, lip sync), with music and general sound effects essentially uncovered — this is a purely vocal-scene benchmark.
[Omni-Judge] No stratified analysis by audio type [Uncertain].
Overall: the three categories of speech / music / Foley sound effects are highly heterogeneous in evaluation metrics, controllable-editing tools, and data availability; the training pipeline must classify and conquer, and a uniform threshold is not appropriate.

### [Video Caption Model Ecosystem](../models/caption_models.md)

Classification and separate handling of the three audio categories — speech / sound effects / music — is the most mature design pattern in the AV captioner ecosystem:
[Two-model division of labor (the most common)] MOVA: Qwen3-Omni-Instruct handles speech transcription, Qwen3-Omni-Captioner handles description of non-speech sound and music; the paper emphasizes that this division "comprehensively covers linguistic content and acoustic features, reducing information loss." UniTalking: Whisper-V3 transcription + Qwen3-Omni-Captioner describing the acoustic environment + Qwen3-Omni performing fusion. SkyReels-V4: Qwen3-Omni uniformly generates the audio caption, Whisper transcribes speech and singing.
[Fixed three-field schema] Foley-Omni's three fields — speech / effects / music — are isomorphic with Bandit's separation output; AVSCap's Acoustic Completeness criterion likewise splits three ways into Speech / SFX / Music.
[Four-model collaboration (Movie Gen, the most fine-grained)] An audio-quality prediction model (1-10 score) + an AED model (determining voice/singing presence and the posterior probability of music) + a general audio caption model (free-form description of sound) + a music caption model (supplementing mood and genre). The paper specifically notes that the music caption model is mainly trained on music samples and is prone to hallucination when there is no music, so **both the AED music probability signal and the music caption signal are retained simultaneously**; in practice this redundant combination gives the best controllability — this is the ecosystem's most valuable engineering lesson regarding "audio-type hallucination."
[A single model handling everything (the aggressive route)] UniVerse-1 uses a single Qwen2.5-Omni to output all three streams in parallel at once; Ovi uses the same MLLM to process both audio-video corpora and pure-audio corpora; CineDance uses Qwen3-Omni-30B-A3B to simultaneously perform three subtasks — sentence-level ASR, shot-level audio prompts (music/ambient sound/sound effects), and character voice-timbre description.
[Current state of capability (quantified by AVSCapBench)] The description-capability gap across the three audio categories is enormous and highly consistent: Speech (40-80 points) ≫ Music (4-40 points) > SFX (7-32 points). SFX is the weakest link industry-wide — the best open-source model, AVSCap-7B, scores only 30.82, and Gemini-3-Flash scores 32.34 — this directly caps the upper bound of training data available for Foley and ambient-sound generation models, and is the direction most urgently in need of a breakthrough in the AV captioner ecosystem.

### [Geometric/Structured Annotation Dataset Collection](../models/geometric_datasets.md)

Not applicable. None of the four datasets has a classification or differentiated handling strategy for speech/sound-effects/music.

### [Post-Training Data Strategy for Video Generation](../models/post_training_data.md) ⚠️

The anchor paper does not address this [Uncertain]. Cross-comparison: Movie Gen is the only work that performs explicit divide-and-conquer by audio type at the post-training stage — the cinematic split (containing both diegetic sound and non-diegetic ambient/theme music, excluding human voice) and the high-quality audio split (pure music O(10)K hours + pure sound effects O(10)K hours) are built separately, differing in scale by two orders of magnitude, indicating that it treats "learning to score/add sound effects" and "learning video-audio correspondence" as two separable objectives. Foley-Omni's three-stage curriculum (sound effects → music → full score) is likewise a curriculum form of type divide-and-conquer, but it stops at supervised training. At the preference-learning level, no work designs separate rewards for speech/sound-effects/music [Uncertain] — this is the clearest gap remaining to be filled in AV post-training: existing audio reward models (AudioBox-Aesthetics) provide a general aesthetic score and cannot distinguish between two entirely different kinds of failure — "insufficient dialogue clarity" versus "sound effects misaligned with on-screen events."

### [Merged Survey of Mainstream Video Pretraining Datasets: Panda-70M, InternVid, Koala...](../models/pretraining_datasets.md)

Not applicable. None of the seven has a classification or separate handling strategy for speech/sound-effects/music/ambient sound.
