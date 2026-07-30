# Foley-Omni (with accompanying evaluation benchmark V2ST-Bench). The paper's full title is *Foley-Omni: A Unified Multimodal Generation Model from Task-Level Audio Synthesis to Complete Video Soundtrack Generation* — that is, a unified multimodal generation model that advances from task-level audio synthesis toward complete video soundtrack (Video-to-Soundtrack, V2ST) generation. Its core positioning is to jointly model three audio-track types — speech, sound effects/foley, and music — within a single latent-space generation process.

> Topic: Data processing for video generation models (including joint audio-video generation): data cleaning pipeline, data distribution, annotation methods

[← Back to home](../index.md)

**Table of Contents**: [Basic Information](#basic-information) · [Data Scale and Distribution](#data-scale-and-distribution) · [Cleaning Pipeline](#cleaning-pipeline) · [Annotation Methods](#annotation-methods) · [Audio-Video Alignment](#audio-video-alignment) · [Training Coordination](#training-coordination) · [Results Comparison](#results-comparison)

## Basic Information

### Name

Foley-Omni (with accompanying evaluation benchmark V2ST-Bench). The paper's full title is *Foley-Omni: A Unified Multimodal Generation Model from Task-Level Audio Synthesis to Complete Video Soundtrack Generation* — that is, a unified multimodal generation model that advances from task-level audio synthesis toward complete video soundtrack (Video-to-Soundtrack, V2ST) generation. Its core positioning is to jointly model three audio-track types — speech, sound effects/foley, and music — within a single latent-space generation process.

### Publishing Organization/Company

Led by the School of Intelligence Science and Technology, Nanjing University, in collaboration with Video Rebirth (an industry partner), Shanghai Jiao Tong University, Beijing Jiaotong University, and Shanghai AI Laboratory. Author list: Ye Tao (陶烨, first author, contact email taoye0402@gmail.com, maintainer of the project homepage ty0402.github.io), Lupeng Liu, Xuenan Xu (徐雪男, a researcher in the audio captioning/audio description direction), Jiasun Feng, Jiarui Wang, Ying Qin, Shuiyang Mao, Wei Liu, and Shuai Wang (王帅, corresponding-author direction, Nanjing University's speech group). The GitHub organization is named NJU-Speech, and the HuggingFace account is CocoBro.

### Release Date (technical report/paper/open-source date)

Submitted to arXiv on June 2, 2026 (arXiv:2606.03672, in the cs.SD/cs.CV categories, v1). The project homepage https://ty0402.github.io/Foley-omni-Web/ and the GitHub repository NJU-Speech/Foley-Omni went live around the same time, with CocoBro/Foley-Omni inference weights released on HuggingFace. ResearchGate indexed it around the same time (publication 405852241). As of the time of this research (July 2026), the V2ST-Bench evaluation set is marked as "Coming soon" in the repository and has not yet been formally released.

### Type (model/dataset/toolchain/benchmark)

A combined output of model + evaluation benchmark + data-processing pipeline methodology. The main deliverable is a unified multimodal audio generation model for video-to-complete-soundtrack (V2ST) generation (roughly 5.5B parameters, based on a Diffusion Transformer); it also proposes the V2ST-Bench evaluation benchmark (300 video-audio-text triplets); and Section 3.1 of the paper fully describes an audio-video data cleaning and structured annotation pipeline. Note: this project generates audio/soundtrack, with the video side serving as conditioning input rather than the generation target, so it belongs to the V2A/V2ST branch within the "audio-video generation" lineage, rather than being a text-to-video model.

### Degree of Openness (whether weights/code/data/pipeline are each open-sourced)

Openness is above medium.
[Weights] Open-sourced. HuggingFace's CocoBro/Foley-Omni releases inference-only weights (v2st.pth), distributed together with dependent components such as the audio VAE, the BigVGAN vocoder, and the visual feature extraction model, declared under the MIT license.
[Code] Partially open-sourced. The GitHub repository NJU-Speech/Foley-Omni provides inference code and visual feature preprocessing scripts (CLIP feature and Synchformer sync feature extraction); training code has not been released. The repository's own license is not clearly labeled on the page, and it states that it redistributes components from upstream projects such as Wan2.2-TI2V-5B, MMAudio, and Ovi, each of which must comply with its own upstream license.
[Data] Training data is not open-sourced. Within the roughly 4.9k hours / 2.7M pairs of training corpus, the self-collected/internal portion is not public; the public-dataset portion (LJSpeech, LibriTTS, AudioCaps, Freesound, MusicCaps, MusicBench, AudioSet, VGGSound, GRID, LRS2, Chem, SpeakerVid, TalkVid, Kling-Foley) can be obtained independently.
[Evaluation benchmark] Promised to be open-sourced but not yet released. The paper states it will release V2ST-Bench's annotations, metadata, and processing scripts; due to copyright restrictions it will not directly distribute the raw video files, instead providing URLs + metadata (similar to the approach used by VGGSound/HowTo100M).
[Pipeline] Disclosure is fairly thorough at the methodological level: Table 7 gives the specific threshold values for the six filtering metrics, Table 12 gives the Gemini 2.5 Pro annotation prompt template, and the acoustic post-verification step gives an explicit -35 dB threshold formula. However, the complete cleaning scripts are not released, nor is a stage-by-stage retention-rate table given.

### Whether Joint Audio-Video Generation Is Supported, and the Implementation Approach (native joint / cascaded / MoE fusion)

This is not an "audio-video simultaneous generation" model, but rather "joint multi-category audio generation conditioned on video." The implementation is native joint generation rather than cascading: the three audio-track types — speech, sound effects, and music — are jointly produced in a single pass within the diffusion generation process of the same shared latent space, rather than being generated separately by TTS/Foley/music models and then mixed together. This is the core point of difference from the baselines (the MMAudio + CosyVoice 3 + AudioX cascaded pipeline, and the MMAudio + LipVoicer + AudioX cascaded pipeline).
Architecturally it is a Diffusion Transformer (DiT); the text side uses a shared UM-T5 encoder to encode the three-field structured text; the audio side uses a frozen Mel VAE (carried over from MMAudio) + a BigVGAN vocoder; the video side provides dual-path conditioning: CLIP features supply scene semantics, and Synchformer features supply temporal synchronization cues (the latter injected via an additive path Z_sync).
Empirical evidence supports joint generation outperforming cascading: on V2ST-Bench, Foley-Omni's WER of 7.59 is better than MMAudio+CosyVoice 3+AudioX's 10.57 and MMAudio+LipVoicer+AudioX's 37.84; its DeSync of 0.16 is likewise better than 0.85/0.26; and its three subjective scores — A-MOS 3.92 / S-MOS 4.13 / T-MOS 4.14 — lead the cascaded baselines across the board (compared with Ground Truth's 4.33/4.37/4.42).

### List of Research Information Sources (URLs of papers/technical reports/official documentation/news, with the nature of each source noted: first-party official / same-team corroboration / third-party report)

1. [First-party official] arXiv abstract page https://arxiv.org/abs/2606.03672 — title, authors, abstract, submission date of June 2, 2026.
2. [First-party official] arXiv full HTML text https://arxiv.org/html/2606.03672v1 — Section 3.1's data cleaning pipeline, Table 7's filtering thresholds, Table 9's training-data composition, Table 6's ablations, the V2ST-Bench main results table, and Appendices B/C.1. The core source of information.
3. [First-party official] arXiv PDF https://arxiv.org/pdf/2606.03672
4. [First-party official] HuggingFace model page https://huggingface.co/CocoBro/Foley-Omni — 5.5B parameter count, MIT license, inference-only weight composition, upstream dependency declarations.
5. [First-party official] GitHub repository https://github.com/NJU-Speech/Foley-Omni — inference code, feature extraction scripts, V2ST-Bench's "Coming soon" status.
6. [First-party official] Project homepage https://ty0402.github.io/Foley-omni-Web/ — demo samples and project introduction.
7. [Third-party index] ResearchGate listing page https://www.researchgate.net/publication/405852241_Foley-Omni_A_Unified_Multimodal_Generation_Model_from_Task-Level_Audio_Synthesis_to_Complete_Video_Soundtrack_Generation
8. [Third-party report] AI Films Studio blog https://studio.aifilms.ai/blog/foley-omni-video-soundtrack-generation — an accessible explainer aimed at creators, with no additional first-party data.

## Data Scale and Distribution

### Training Data Volume (number of video clips/hours/tokens, pretraining vs. SFT separated) ⚠️

Total training scale is approximately 2.7M paired audio/video-text samples, cumulatively about 4.9k hours of audio processed through the cleaning pipeline. Of these, the unified video-audio-text triplets (v_i, a_i, Ŝ_i) produced by the data cleaning pipeline number approximately 2.0M (this portion is the direct product of this paper's pipeline; the remainder is public TTS/TTA/TTM corpora taken directly as-is).
Broken down by the six task groups (Table 9):
- TTS (text-to-speech): LJSpeech + LibriTTS + internal, 1,253 hours
- TTA (text-to-audio/sound-effects): AudioCaps + Freesound, 912 hours
- TTM (text-to-music): MusicCaps + MusicBench + AudioSet, 139 hours
- VisualTTS (visually conditioned speech synthesis): Chem + GRID + LRS2 + SpeakerVid + TalkVid, 1,980 hours (the largest share)
- V2A (video-to-sound-effects): VGGSound + Kling-Foley + internal, 403 hours
- V2ST (video-to-complete-soundtrack, this paper's core task): internal + SpeakerVid, 216 hours (the smallest scale, used as Stage 3 fine-tuning data)
Stage perspective: Stage 1 uses about 0.7M text-audio pairs (TTA/TTS/TTM); Stage 2 expands to video conditioning (V2A/VisualTTS); Stage 3 performs joint fine-tuning using 216 hours of V2ST data + 100 hours of replay data from each preceding single-task domain.
On the evaluation side, V2ST-Bench consists of 300 clips of 5-10 seconds.
[Uncertain] The paper does not disclose the total size of the raw video pool before cleaning (in hours or clip count), so the overall retention rate cannot be back-calculated.

### Composition of Data Sources (in-house/public datasets/web scraping/licensed acquisition/synthetic data)

A mixture of sources, falling into two main categories — public datasets and internal in-house data — with no mention of large-scale web scraping or licensed acquisition.
[Public datasets] Speech category: LJSpeech, LibriTTS (the TTS foundation); GRID, LRS2, Chem (classic lip-reading/visual-speech datasets); SpeakerVid, TalkVid (newer, large-scale speaker-video datasets, used for both VisualTTS and V2ST). Sound-effects category: AudioCaps, Freesound, VGGSound, Kling-Foley (a foley dataset released by Kuaishou's Kling team). Music category: MusicCaps, MusicBench, AudioSet.
[Internal data] Appears in three places — TTS (internal speech library), V2A (internal audio-video corpus), and V2ST (internal data, forming the bulk of the 216 hours) — and is the primary target that this paper's data cleaning pipeline operates on. The paper describes this portion as "weakly labeled audiovisual data," i.e., raw material consisting only of the original video + original audio track, lacking component-level text annotation.
[Synthetic/constructed data] None. The three-field annotations generated by Gemini 2.5 Pro constitute "model annotation" rather than "synthetic audio-video data"; the pipeline does not involve manually perturbed construction of training pairs.
Notably, the task orientation of the data sources is very strong: VisualTTS accounts for 1,980 hours (about 40%), showing that the model invests the most in the "speaker video → speech" pathway, directly corresponding to its flagship advantage in speech intelligibility (WER 7.59, even approaching or bettering Ground Truth's 8.03 in some comparisons).

### Data Compliance and Provenance (proportion of licensed data, rights-cleared datasets, C2PA, etc.) ⚠️

Disclosure here is very thin. The paper only makes a general statement in its Ethics/Limitations section that the data was "collected under appropriate usage agreements," and acknowledges the model's potential misuse risk for producing deepfakes.
An indirect sign of compliance awareness: the V2ST-Bench release strategy explicitly avoids directly distributing raw video files due to redistributable content constraints, instead providing URLs and metadata for users to download themselves — a common workaround for academic audio-video datasets (as with VGGSound, AudioSet).
On the model-weights side, an MIT license is declared, and the redistribution of components from Wan2.2-TI2V-5B and MMAudio is explicitly labeled with guidance for users to consult the original upstream licenses — a relatively well-formed upstream compliance statement.
[Uncertain] The proportion of licensed data is not disclosed, no mention is made of acquiring rights-cleared datasets, no C2PA or any content-provenance/watermarking mechanism is mentioned, the specific acquisition channels and licensing form of the internal data are not explained, and the license compatibility of the individual public datasets (e.g., Freesound, AudioSet, VGGSound) is not discussed either.

### Clip Duration Distribution and Segmentation Strategy ⚠️

Disclosure is limited.
[Evaluation set] V2ST-Bench clip durations are explicitly 5-10 seconds.
[Training set] The paper does not give a duration-distribution histogram or average duration for training clips. A rough order of magnitude can be back-derived from the data scale: about 4.9k hours corresponds to about 2.7M samples, averaging roughly 6.5 seconds per sample, consistent with V2ST-Bench's 5-10 second range — indicating the overall approach follows a short-clip (roughly under 10 seconds) route, matching mainstream V2A settings such as MMAudio and VGGSound (10-second clips), against which it benchmarks.
[Segmentation strategy] [Uncertain] The paper does not describe how long videos are segmented into training clips, and does not mention shot-boundary detection, fixed sliding windows, or other segmentation methods. The filtering-stage metrics (motion score, Synchformer sync score, ImageBind score) are all computed at the clip level, hinting that segmentation occurs before filtering, but the specific segmentation algorithm is not explained.

### Resolution/Aspect-Ratio Distribution and Bucketing Strategy ⚠️

Only a filtering lower bound exists, with no distribution statistics. The filtering stage sets a hard visual-resolution gate of ≥480p, paired with a bitrate gate of ≥1 Mbps (a double safeguard against "fake-HD" samples that are low-bitrate but nominally high-resolution — this bitrate constraint is relatively uncommon among comparable work; UniVerse-1 uses a bitrate ratio, a similar idea).
[Uncertain] The resolution-distribution histogram of the final dataset is not disclosed, nor is the aspect-ratio distribution, nor is any resolution or aspect-ratio bucketing strategy mentioned. This has relatively limited impact on this project: since video serves only as conditioning input and is encoded by CLIP and Synchformer into fixed-dimensional features, resolution mainly affects feature-extraction quality rather than generation resolution, so strict bucketed training as used in text-to-video models is not needed.

### Category/Domain Distribution and Mixture Strategy (proportional control and concept balancing across people, actions, scenes, styles, etc.) ⚠️

The paper does not perform mixture control along a "visual domain/category" dimension; instead, the "task group" is the primary unit of mixture control — this is a fundamental difference between Foley-Omni's data organization and text-to-video models.
Hours-based mixture across the six task groups (total 4.9k hours): VisualTTS 1,980h (40.4%) > TTS 1,253h (25.6%) > TTA 912h (18.6%) > V2A 403h (8.2%) > V2ST 216h (4.4%) > TTM 139h (2.8%). Speech-related categories (VisualTTS + TTS) together account for 66%, sound-effects-related categories (TTA + V2A) account for 26.8%, and music-related (TTM) is only 2.8% — a heavy skew toward speech, which explains the model's standout advantage on the WER metric (7.59, lower than the two cascaded baselines' 10.57 and 37.84), while also being a possible hidden concern behind relatively weaker music-generation capability.
The visual-content domain can be indirectly inferred from the dataset sources: GRID, LRS2, and Chem are controlled/semi-controlled frontal speaker videos (lab readings, BBC news, chemistry lectures); SpeakerVid and TalkVid are large-scale in-the-wild speaker videos; VGGSound consists of 10-second in-the-wild event videos (310 sound-source classes); Kling-Foley is dedicated foley video. Overall, the domain is heavily concentrated in two categories — "scenes with a person speaking" and "scenes with clear sound-source events" — lacking visual domains such as pure scenery, abstract art, or animation.
[Motion intensity as an implicit domain control] The motion score is restricted to the [0.1, 3.2] range — the lower bound removes near-static footage (no visual event to align to, leaving V2A's supervisory signal empty), and the upper bound removes violent motion/fast cutting (unreliable optical-flow estimation, chaotic audio-visual correspondence); in effect, motion intensity is used to narrow the data domain down to "moderate dynamics, a single continuous scene."
[Uncertain] No explicit distribution statistics for visual category labels are given, no concept-balancing or long-tail category resampling strategy is mentioned, and no proportional control over people/actions/scenes/styles is given.

### Audio Category Distribution and Mixture (proportions and control strategy for speech/sound-effect foley/music/ambient sound/silence) — a dimension unique to AV models ⚠️

This is Foley-Omni's most distinctive dimension, and also its core contribution setting it apart from contemporaneous AV work — it elevates "audio category" from an implicit attribute to an explicit, first-class structured field.
[Three-way classification system] Speech [WORDS] / sound effects [AUDIO] / music [MUSIC], the three constituting parallel fields of the structured annotation Ŝ. The key design is that "each field can be left empty when the corresponding audio component is absent" — an empty field itself carries information, equivalent to an explicit negative annotation of "this clip does not contain this type of audio." This lets the same conditioning interface express both a single task (only one field filled = TTS/TTA/TTM) and a complete soundtrack (multiple fields coexisting = V2ST), achieving a unified formalization of the tasks.
[Mixture-control strategy — two-stage determination]
Stage one (joint visual + auditory detection): Gemini 2.5 Pro receives both video frames and the audio track simultaneously, and is explicitly instructed to "first determine whether a given audio component (Speech/Sound Effects/Music) is physically present, and only if present, then generate the corresponding description" — that is, existence binary classification is done first, followed by description generation, rather than generating all three descriptions indiscriminately.
Stage two (acoustic energy-gating correction): the Bandit source separation model (Watcharasupat et al., 2024, a speech/effects/music three-way separation model designed specifically for cinematic audio source separation) is used to separate the raw audio track into three stems; a field's annotation is retained only when the corresponding stem's RMS energy E(a_c) > −35 dB. This step is specifically designed to eliminate "visual hallucination" — i.e., Gemini seeing a piano/a person's mouth moving/a car in the frame and presuming the presence of music/speech/sound-effects, when in fact none of it is audible on the track. This is the methodological innovation the paper explicitly names: two-path validation combining visual-path annotation with acoustic-path verification, which the paper states is missing from typical audio-video dataset construction. The −35 dB threshold was determined through "manual inspection of a small validation subset."
[Silence handling] The very first filtering criterion removes clips containing silence; samples with no audio track/silence do not enter the subsequent pipeline.
[Actual distribution] The final proportions of the three audio categories in the training set are not given directly, but can be approximated from task-group hours: speech category 66%, sound-effects category 27%, music category 3%.
[Explicit mixture on the evaluation set] V2ST-Bench's 300 clips strictly require "≥2 audio components coexisting," with a mixture of speech+sound-effects 150 clips (50%), speech+music 120 clips (40%), and speech+sound-effects+music 30 clips (10%). Speech appears in 100% of the three combination types, showing the benchmark's design is also speech-centric; the hardest scenario with all three components is only 10%.
[Uncertain] The actual non-empty proportion for each of the three fields in the training set, and the sample-count distribution across the different combination patterns (single-field/two-field/three-field), are not disclosed.

### Narrative Structure Distribution (single-shot vs. multi-shot, average clip duration, shot-count distribution, whether a native audio track is included) ⚠️

Overall this follows a single-shot, short-clip paradigm. Judging comprehensively from the 5-10 second clip duration, the motion-score upper bound of 3.2 suppressing violent motion/fast cuts, and the data sources (GRID/LRS2/Chem/SpeakerVid/TalkVid being continuous speaker shots, VGGSound being single-event 10-second clips), the training and evaluation data are essentially all single-shot, continuous clips with no cut points. This matches the nature of the task: modeling cross-shot soundtrack coherence is not a goal of this paper.
[Whether a native audio track is included] All clips include a native audio track, and the native audio track is the sole supervisory target (the generation target is to restore/reconstruct the three components of that track). This contrasts with text-to-video models, where an audio track is optional — for Foley-Omni, samples without an audio track are filtered out directly, and native audio-track quality (AudioBox Aesthetics ≥0.6) is a hard gate.
[Uncertain] No shot-count distribution statistics are given, no precise average clip-duration value is given, and it is not stated whether cross-shot detection and removal was performed.

### Language/Accent Distribution (data basis for multilingual lip-sync capability) ⚠️

Disclosure is missing here, but it can be strongly inferred from the dataset sources that English is the overwhelming majority language. On the TTS side, LJSpeech (a single American-English speaker) and LibriTTS (English audiobooks with multiple speakers); on the VisualTTS side, GRID (a controlled English corpus with 34 British speakers), LRS2 (BBC British English), and Chem (English lectures) are all pure-English datasets; SpeakerVid and TalkVid are newer, large-scale speaker-video sets that may contain a small amount of multilingual content, though the proportion is unclear.
On the evaluation side, WER is used as the speech-intelligibility metric (V2ST-Bench WER 7.59, GRID subset WER 15.3); WER computation typically relies on an English ASR model, further corroborating that the evaluation scenario is English.
[Uncertain] The paper does not give any language-distribution table, does not mention accent annotation, does not mention multilingual lip-sync capability, and does not explain the language composition of the internal data. This is a clear shortcoming of this work relative to industrial-grade AV models (such as Veo 3, Sora 2, Kling Omni) along the multilingual lip-sync dimension.

## Cleaning Pipeline

### Overall Structure of the Cleaning Funnel (number of filtering stages, ordering)

The cleaning funnel is a three-stage serial structure, with the core design philosophy of "filter first, then annotate, then use acoustic signals to reverse-validate the annotation." Its input is weakly labeled audiovisual data, and its output is approximately 2.0M structured triplets (v_i, a_i, Ŝ_i).
[Stage 1: Multi-Dimensional Quality Filtering] Placed first, cutting away low-quality samples before the expensive dense-annotation step — a typical cost-oriented ordering. The paper's original text describes it as: "removes clips with silence, low visual resolution, poor audio quality, weak audiovisual semantic consistency, or unreliable synchronization" — five removal criteria mapping to three dimensions and six metrics (Table 7):
  · Visual dimension: resolution ≥480p, bitrate ≥1 Mbps, motion score ∈ [0.1, 3.2]
  · Audio dimension: Meta AudioBox Aesthetics audio-quality score ≥0.6 (simultaneously covering both silence and poor audio quality)
  · Alignment dimension: ImageBind (IB) score ≥0.3 (semantic consistency), Synchformer sync score ≥0.2 (temporal-synchronization reliability)
  Notably, "semantic consistency" and "temporal synchronization" are split into two independent metrics with two independent thresholds, rather than being merged into a single alignment score — a refined design choice in this pipeline.
[Stage 2: Structured Annotation] Gemini 2.5 Pro is invoked on the retained clips, receiving both video frames and the audio track simultaneously, and outputs according to the [WORDS]/[AUDIO]/[MUSIC] three-field template. The model is required to first perform component-existence detection, then generate descriptions (detect-then-describe); the prompt template is given in paper Table 12.
[Stage 3: Acoustic Post-Verification] The Bandit source separation model splits the original audio track into three stems — speech/effects/music — and applies an energy gate E(a_c) > −35 dB RMS field by field; fields that fail are set empty. This stage specifically corrects the visual-bias error from Stage 2, i.e., cases where the VLM fabricated audio components that are in fact inaudible, based on visual cues it saw.
[Overall assessment] This pipeline's methodological highlight is its "two-path validation": the visual/multimodal path is responsible for generating the annotation, while the purely acoustic path is responsible for vetoing the annotation; since the two paths draw on different sources of information, their error modes are uncorrelated, making the cross-validation effective. This idea has reference value for any work relying on a VLM for joint audio-video annotation. A relative shortcoming is the lack of disclosure around routine steps such as deduplication, safety filtering, and manual review, and the absence of a quantitative stage-by-stage retention-rate report.

### Quantitative Funnel Retention Rate (input/output volume and final retention rate at each filtering stage, e.g., Apollo's 27%) ⚠️

Quantitative disclosure is insufficient to construct a complete funnel table.
[Known endpoints] The pipeline outputs approximately 2.0M video-audio-text triplets; the total training corpus is approximately 2.7M pairs, about 4.9k hours.
[Missing] [Uncertain] The paper does not give the size of the raw pool before cleaning (in hours or clip count), does not give any input/output comparison for a single filtering stage, does not give the individual rejection proportion for each of the six metrics, and does not give the proportion of fields set empty during Stage 3's acoustic post-verification (this figure could directly quantify the severity of Gemini's visual hallucination and would be the most persuasive piece of evidence for this method — unfortunately it is not reported). The end-to-end retention rate, analogous to Apollo's 27%, therefore cannot be computed.
[A rough inference that can be made] Among the six filtering metrics, Synchformer sync score ≥0.2 and ImageBind ≥0.3 are typically the main culprits for in-the-wild audio-video data (a large amount of online video has post-hoc added soundtracks, off-screen narration, or audio unrelated to the picture); referencing comparable work (e.g., UniVerse-1 filtering from 40k+ hours down to 7,685 hours, about 19%; MMAudio-family work's alignment-filtering rejection rate on VGGSound often exceeding 50%), it can be speculated that this pipeline's raw pool is on the order of tens of thousands of hours, with an end-to-end retention rate roughly in the 20-50% range — but this is pure extrapolation, with no data support from the paper.

### Shot Segmentation Method (PySceneDetect / in-house model / shot-aware splitting) ⚠️

[Uncertain] The paper does not mention any shot-segmentation method at all. There is no description of PySceneDetect, TransNetV2, an in-house shot-boundary-detection model, or any shot-aware splitting.
Inferring from the data composition, the segmentation need itself appears weak: the training data is predominantly public datasets, and GRID, LRS2, VGGSound, AudioCaps, etc. are already pre-cut fixed-length short clips (VGGSound fixed at 10 seconds, GRID about 3 seconds), requiring no further segmentation; only the internal audio-video corpus and SpeakerVid/TalkVid may involve slicing from longer videos, but the paper does not explain how this is handled. The motion-score upper bound of 3.2 may indirectly serve to remove clips containing hard shot cuts (a shot change would cause an abnormal spike in optical-flow/motion score), but this is only a side effect rather than an explicit shot-segmentation design.

### Quality Filtering (aesthetic score, sharpness, OCR text filtering, letterbox/watermark/logo detection) ⚠️

Quality filtering is the first stage of the cleaning funnel, applying parallel gatekeeping across visual and audio channels; all criteria are threshold-based hard filters with no weighted composite score.
[Visual quality]
  · Resolution ≥480p — removes low-definition clips, ensuring the reliability of CLIP and Synchformer visual feature extraction. The 480p bar is relatively lenient (compared with the 720p/1080p bars common in text-to-video models), which is reasonable given that video here serves only as a condition rather than the generation target.
  · Bitrate ≥1 Mbps — forms a double safeguard together with resolution, specifically targeting samples that are "nominally high-resolution but heavily compressed with actual detail loss." This is a relatively uncommon but very practical criterion, since an upsampled low-quality video can pass the resolution check but fail the bitrate check.
  · Motion score ∈ [0.1, 3.2] — a two-sided threshold. The lower bound removes near-static footage (lacking visual events, unable to provide temporal cues for sound-effect generation), and the upper bound removes excessively violent motion (unreliable motion estimation, chaotic audio-visual correspondence). The paper does not specify the exact method used to compute motion score (optical-flow mean/frame difference).
[Audio quality]
  · Meta AudioBox Aesthetics score ≥0.6 — uses Meta's open-source AudioBox Aesthetics no-reference audio aesthetics/quality assessment model. This single metric shoulders multiple duties at once: low scores naturally filter out silent clips (an aesthetic score is extremely low with no content), and remove audio tracks with heavy background noise/clipping distortion/low sample rate/encoding degradation. Compared with traditional SNR thresholds, AudioBox Aesthetics is a learned perceptual-quality assessment, closer to human-ear judgment, and represents the new 2025-2026 paradigm of audio data cleaning.
[Common items not addressed] [Uncertain] The paper does not mention aesthetic scoring (e.g., LAION-Aesthetics), OCR text filtering, letterbox/black-bar detection, watermark/logo detection, burned-in subtitle detection, or other standard steps found in text-to-video cleaning. For this task these are indeed lower priority (vision only needs to support semantic and synchronization judgment; whether the image looks aesthetically pleasing does not affect audio generation), but a watermark/subtitle occluding the mouth region could affect VisualTTS's lip-sync learning — a risk the paper does not discuss.

### Motion Filtering (optical-flow/motion-score thresholds, removing static and jittery content) ⚠️

Motion filtering is implemented through the two-sided motion-score threshold range [0.1, 3.2], one of three visual-dimension metrics in the filtering stage.
[Lower bound 0.1] Removes static or near-static footage. This step is especially critical for the V2A/V2ST tasks: a static image has no visual event, so the model cannot learn the temporal mapping of "visual action → sound event" from it, and such samples would become pure-noise supervision.
[Upper bound 3.2] Removes clips with excessively violent motion. Jitter, fast pans, and high-frequency cutting distort Synchformer's temporal features, and the audio-visual correspondence in such clips is often chaotic (multiple events overlapping, sound sources leaving the frame).
[Difference from text-to-video models] Motion filtering in text-to-video models mainly serves the goal of "generating videos with a sense of motion" (removing static footage to avoid the model generating still frames); Foley-Omni's motion filtering instead serves "ensuring audio-visual events can be aligned" — a different starting point, though the threshold form is similar (both are two-sided ranges).
[Uncertain] The specific computation method for motion score (RAFT/UniMatch optical flow, frame-to-frame differencing, or something else) and the normalization scale are not explained, so the values [0.1, 3.2] are difficult to directly transfer and reuse.

### Deduplication Method (exact deduplication and embedding-based semantic deduplication recorded separately) ⚠️

[Uncertain] The paper does not mention deduplication at all, with no description of either exact deduplication (file hashing, perceptual hashing/pHash) or embedding-based semantic deduplication.
A potential risk point: the training data has dataset overlap across task groups — SpeakerVid appears both within VisualTTS (the 1,980h group) and within V2ST (the 216h group); AudioSet is used for TTM while VGGSound is itself a subset derivative of AudioSet (sharing the same source YouTube videos) — raising the possibility of samples being double-counted, or even train-evaluation contamination. Furthermore, the paper states that V2ST-Bench's 300 clips are "drawn from curated audiovisual pool" (taken from the same cleaned data pool); if strict removal from the training set was not performed, there is a risk of evaluation-set leakage — the paper only states that it underwent manual review, without explicitly declaring no overlap with the training set. This is a gap worth noting in this work's methodological rigor.

### VLM/LLM as Quality Inspector (multimodal large-model quality scoring and mismatch removal; the 2026 trend of shifting from shallow scorers to large-model semantic judgment) ⚠️

Gemini 2.5 Pro shoulders a dual role in this pipeline — "annotation generation + existence discrimination" — a typical case of the 2026 trend of "large models as data quality inspectors/annotators." But this paper goes a step further: it does not trust the large model's judgment on its own, and instead designs an independent acoustic verification channel to veto the large model's output.
[Gemini 2.5 Pro's discriminative duties] It receives video frames and the corresponding audio track (a dual multimodal input, not vision-only), and is explicitly instructed by the prompt to carry out a two-step detect-then-describe process: first determining whether each of the three components — Speech / Sound Effects / Music — is "physically present in the clip," and generating a descriptive caption only for the components judged present. This step is equivalent to using the large model for three-way multi-label discrimination + conditional description generation. The system prompt template is given in paper Table 12.
[Distrust of and correction for the large model's judgment] The paper explicitly points out that this step suffers from visual bias — the multimodal model tends to be dominated by visual cues, annotating music upon seeing an instrument, annotating speech upon seeing a mouth move, annotating engine sound upon seeing a vehicle — when in reality these components may not exist or may not be audible in the actual audio track at all. To address this, a third stage of Bandit source separation + energy gating (E(a_c) > −35 dB) is introduced as an independent, purely acoustic criterion; any field that fails to reach the energy threshold is uniformly set empty.
[Methodological value] This constitutes a "generate-then-verify" dual-path architecture: the verification path does not rely on any neural-network semantic understanding, but instead on physical energy measurements after source separation, fully decoupled from the error modes of the generation path. Compared with relying purely on VLM scoring (susceptible to same-source bias) or relying purely on signal processing (lacking semantic understanding), this combination is more robust. The paper calls this a two-path validation typically missing from audio-video dataset construction.
[Limitations] [Uncertain] The paper does not report what proportion of Gemini's annotation fields were actually vetoed by the Bandit verification stage — this figure is key evidence for quantifying the severity of VLM hallucination, and its absence makes it impossible to assess the design's actual benefit; nor is a data ablation of "with vs. without acoustic post-verification" performed to demonstrate its contribution to final model performance. Additionally, there is no mention of using any model to perform secondary scoring or removal on caption quality itself (descriptive accuracy, fine-grained detail).

### Safety and Compliance Filtering (NSFW, copyright, face/privacy) ⚠️

[Uncertain] No safety or compliance filtering step is described in the pipeline: no NSFW detection, no copyrighted-content identification, no face-privacy handling or de-identification measures.
The paper only acknowledges risk at the level of an ethics statement: it explicitly mentions the possibility of the model being misused to produce deepfakes (given that VisualTTS accounts for 40% of the training data and the model can generate matching speech from a speaker video, this risk is indeed substantively present), and states that data was "collected under appropriate usage agreements." But these are after-the-fact statements rather than technical measures within the data-processing pipeline.
There is no watermarking of generated content, no C2PA marking, and no authorization-confirmation mechanism for speaker identity. Given that datasets such as GRID, LRS2, SpeakerVid, and TalkVid all contain large amounts of identifiable faces and voiceprints, and that the model directly learns a "face → voice" mapping, the absence of privacy-dimension handling is a fairly clear compliance shortcoming of this work.

## Annotation Methods

### Caption Model Used (in-house VLM / open-source model, model scale) ⚠️

[Primary annotation model] Gemini 2.5 Pro (Google's closed-source commercial multimodal large model). Its selling point is native support for dual video + audio modality input — a hard requirement for this task, since annotation must be based simultaneously on "what is seen" and "what is heard"; a vision-only VLM (such as Qwen2.5-VL or InternVL) cannot handle audio-existence discrimination. Using a closed-source API means annotation cost is relatively high and not fully reproducible, but the paper does not disclose call volume or cost. Parameter scale is not public.
[Auxiliary model — acoustic verification] Bandit (Watcharasupat et al., 2024), a model designed for cinematic audio source separation, naturally separates into speech / effects / music, exactly isomorphic to this paper's three-field schema — a very well-matched choice; a general-purpose music source-separation model (such as Demucs's vocals/drums/bass/other) could not directly correspond to the three fields.
[Text encoder] The UM-T5 encoder (a multilingual T5 variant), used to encode the three-field structured text into a shared semantic space for conditioning injection into the DiT. All tasks share the same encoder, forming the implementation basis of the "unified interface" design.
[Visual feature model] CLIP (scene semantic features) + Synchformer (temporal synchronization features); the latter doubles as the synchronization scorer in the filtering stage — one model serving two purposes, with training and cleaning using the same synchronization representation, ensuring consistency between the filtering criterion and the model's learning objective.
[Uncertain] No independent ASR model is used for speech transcription (see the next item); there is no mention of any in-house captioner or of distilling Gemini's output to reduce cost.

### Caption Density and Degree of Structuring (short/long/dense description, structured fields such as camera motion, style tags) ⚠️

The degree of structuring is high, but the density of description within a single field is not high — this reflects a design orientation of "structure over density," markedly different from the route of pursuing long, dense captions favored by text-to-video models.
[Format] A unified template: `[WORDS] <spoken content>; [AUDIO] <sound event descriptions>; [MUSIC] <music specification>`, with the three fields in a fixed order, separated by explicit tags; field content may be an empty string.
[Three layers of value from structuring]
  1. A composable conditioning interface: filling a single field = a single task (TTS/TTA/TTM), filling multiple fields = a complete soundtrack (V2ST); one template covers all six task types, avoiding the need to design an independent conditioning pathway for each task.
  2. Explicit negative annotation: an empty field explicitly tells the model "this clip has no audio of this type," rather than leaving the model to guess from an absence. This directly helps suppress the model from conjuring music out of thin air when it should not generate any (a common failure mode of V2A models).
  3. Verifiability: fielding allows each annotation unit to be verified by independent acoustic evidence (the energy of the corresponding stem); a free-form whole-clip description would not allow this kind of fine-grained correction.
[Lineage relationship to contemporaneous work] In the paper's comparative context, this belongs to one branch of the 2026 trend of "splitting joint audio-video captions into separate streams": LTX-2 follows a "unified full-soundscape description" route (a single piece of text describing the entire soundscape), Script-a-Video follows factorized streams (decomposed into independent streams), and Foley-Omni is the simplest fixed three-field scheme. The advantage of the three-field approach is an extremely simple schema, a natural correspondence to source-separation models, and ease of verification; the cost is limited expressiveness — it cannot describe the relative position, loudness layering, or temporal order of multiple sound sources within a single field.
[Uncertain] No average-length/word-count statistics for the captions are given; it is not stated whether the [WORDS] field is a verbatim transcript or a content summary (inferring from WER being used as an evaluation metric, it should be a verbatim transcription text); it is not stated whether visual-side structured tags such as camera motion or visual style are included (judging from the template, they are not — visual information is carried entirely by CLIP/Synchformer features and does not enter the text condition).

### Joint Audio-Video Caption Structure (whether both visual + auditory tracks are covered simultaneously, whether they are split into separate fields, e.g. LTX-2's full soundscape description, Script-a-Video's factorized streams, Foley-Omni's three fields)

This is this work's most emblematic contribution, and the "three fields" specifically flagged in the note.
[Schema definition] The joint audio-video annotation is Ŝ = ([WORDS] spoken content, [AUDIO] sound-effect event description, [MUSIC] music description). The three fields sit in parallel, each independently allowed to be empty, jointly describing the complete auditory content of the same clip.
[Whether the visual track is covered] It is not. This is an important design choice: the schema structures only the "auditory" side; the visual side receives no text description, and is instead injected into the model directly via dual-path CLIP (semantic) + Synchformer (temporal) features. The rationale is that video in this task is a condition rather than the generation target, so there is no need to use text as an intermediary representation; however, the annotation-generation process itself is "audio-visual joint" — Gemini simultaneously watches the video frames and listens to the audio track to produce these three auditory fields, with visual information implicitly participating in the annotation decision (e.g., using the picture to judge whether a given sound belongs to an on-screen sound source).
[Whether split into separate fields] Yes, and this is precisely the core point. Compared with a "single full-soundscape description," splitting into streams brings three actionable benefits: (a) different conditioning strengths/dropout can be applied to different fields during training; (b) at inference time, a user can fill in only certain fields for targeted control (e.g., wanting sound effects but no music); (c) during cleaning, each field can be independently acoustically verified and individually set empty, whereas a whole-clip description can only be kept or discarded in its entirety — a much coarser granularity.
[Isomorphism with source separation] The three fields ↔ Bandit's three stems ↔ the three generation tasks (TTS/TTA/TTM) form a strict one-to-one correspondence; this cross-stage structural consistency is the elegance of this design — the annotation schema, the verification tool, the task definitions, and the evaluation dimensions are all aligned on the same three-way classification.
[Manifestation on the evaluation side] V2ST-Bench is organized by field-coexistence pattern: speech+sound-effects 150, speech+music 120, all three present 30 — directly testing the model's joint-generation capability when multiple fields are simultaneously non-empty.

### Dialogue Transcription and Speaker Attribute Annotation (ASR transcription + speaker identity/language/accent/emotion) ⚠️

Dialogue transcription is carried by the [WORDS] field, but its implementation path is unconventional — no independent ASR is used; instead, Gemini 2.5 Pro directly produces the spoken-content text as part of its multimodal annotation. The benefit of this is that transcription, existence discrimination, and the descriptions of the other two fields are all completed within a single call, simplifying both cost and process; the cost is that transcription accuracy is not as good as a dedicated ASR system (e.g., Whisper), and the paper performs no transcription-quality evaluation.
[Quality assurance] The [WORDS] field is likewise constrained by the −35 dB energy gate on the Bandit speech stem — if the energy of the separated speech track fails to meet the bar, it indicates that a person's mouth is moving on screen but there is in fact no audible speech (or the speech is drowned out), and the field is set empty, effectively preventing ASR/VLM from producing hallucinated transcriptions where there is no speech.
[Speaker attribute annotation] [Uncertain] The paper describes no structured annotation of speaker identity ID, gender, age, language, accent, or emotion. Within the three-field schema, [WORDS] carries only the spoken-content text, with no description of speaking style/timbre. This implies that the model's timbre-control capability relies mainly on visual conditioning (inferring timbre from the speaker's face) rather than on textual attribute annotation — consistent with the task setup of VisualTTS, but also limiting explicit controllability over speech style.
[Indirect evidence] The evaluation uses WER as a core metric (V2ST-Bench 7.59, GRID 15.3, compared with Ground Truth's 8.03), indicating the [WORDS] field is a verbatim transcript rather than a content summary — otherwise WER could not be computed.

### Geometric and Structured Annotation (camera parameters, depth, 3D point tracks, action labeling, explicit state) ⚠️

[Uncertain] The paper does not touch on any geometric or 3D structured annotation: no camera parameters (intrinsics/extrinsics, trajectory), no depth maps, no 3D point tracks, no human pose or action labeling, no explicit state annotation.
The closest thing to "structured visual information" is the temporal synchronization features extracted by Synchformer and the scene semantic features from CLIP, but these are implicit neural-network representations rather than interpretable geometric annotations. Motion score is the only explicit motion quantity, and it is used only for filtering-threshold judgment, not retained as a training annotation.
This absence has limited impact on this task: audio generation does not require precise 3D geometric understanding, since audio-visual alignment relies mainly on 2D temporal event cues. But for finer-grained spatial audio generation (such as binaural/surround sound, or sound-source position changing with camera motion), the lack of camera and 3D annotation would become a bottleneck — this paper generates a mono/stereo mixed audio track and does not address spatial audio.

### Synthetic Data Construction (controlled perturbation/edited construction of training pairs, e.g. InstructAV2AV) ⚠️

[Uncertain] The paper describes no synthetic-data-construction step. The pipeline contains no controlled perturbation, audio-track substitution, time-shift construction of negative samples, or edit-instruction pair construction similar to InstructAV2AV.
The only thing touching on "audio decomposition and recomposition" is Bandit source separation, but its purpose is to verify annotation (energy gating) rather than to construct training samples — the paper does not mention using the separated single-category stems in reverse as synthetic training data for TTA/TTM (this would be a natural data-augmentation idea: separating a pure sound-effects track out of V2ST data could expand the V2A training set, but this is neither adopted nor mentioned in the text).
All data consists of genuinely collected audio-video pairs + model-generated text annotation — a "real data + model annotation" paradigm, containing no generative data synthesis.

### Degree of Human Involvement (manual annotation, manual quality inspection, model pre-screening + manual review) ⚠️

The degree of human involvement is low, and concentrated in two places — threshold calibration and evaluation-set gatekeeping — while training-data annotation is fully automated.
[Threshold calibration] The −35 dB RMS energy gate used in acoustic post-verification was determined through "manual inspection of a small validation subset." This is a typical "human calibrates, machine executes" pattern — humans participate only in hyperparameter selection, not in per-sample judgment. The paper does not explain the basis for determining the other six thresholds in Table 7 (480p, 1 Mbps, [0.1,3.2], 0.6, 0.3, 0.2); presumably these likewise involved an element of empirical manual tuning.
[Manual review of the evaluation set] V2ST-Bench's 300 clips underwent manual review, with the review dimensions explicitly stated as three: audiovisual consistency, annotation accuracy, and suitability for mixed soundtrack evaluation. This is a "model pre-screening + manual review" pattern, but applied only to the 300 evaluation samples — an extremely small scale.
[Subjective evaluation] Three subjective opinion scores — A-MOS / S-MOS / T-MOS — are used, requiring human listening-test scoring. [Uncertain] The number of evaluators, recruitment method, scoring rubric, and consistency checks are not stated.
[Training data] All approximately 2.0M training annotations are produced automatically by Gemini + Bandit, with no manual review step, and no accuracy report from sample-based quality inspection is mentioned either.

## Audio-Video Alignment

### Audio-Video Synchronization Detection Method (lip sync, event alignment) ⚠️

Synchronization detection forms half of the "alignment dimension" in the filtering stage, and the same technology stack runs consistently through cleaning, modeling, and evaluation — a strong through-line.
[Cleaning stage] Synchformer is used to compute the sync score, with a threshold of ≥0.2, removing clips with "unreliable synchronization." Synchformer is a Transformer-based audio-video synchronization detection model; compared with the classic SyncNet, its advantage is that it is not limited to lip sync — it can handle temporal alignment of general audio-visual events (e.g., an impact sound and the impact visual), which is necessary for this task, where sound effects are an important generation target. Using SyncNet instead would cause large-scale mistaken rejection of sound-effect data in non-face scenes.
[Modeling stage] Synchformer features are used directly as the model's temporal conditioning input, injected into the DiT through an additive path (denoted Z_sync). This means the "synchronization representation used for filtering" and the "synchronization condition used for generation" are one and the same, naturally aligning the filtering criterion with the model's capability — the samples that pass filtering are precisely those from which the model can effectively exploit the synchronization signal. This design is noteworthy among comparable works.
[Evaluation stage] The DeSync metric (lower is better) is used to measure the temporal-alignment quality between generated audio and video. Foley-Omni achieves a DeSync of 0.16 on V2ST-Bench, very close to Ground Truth's 0.14, and markedly better than the cascaded baselines MMAudio+CosyVoice 3+AudioX's 0.85 and MMAudio+LipVoicer+AudioX's 0.26.
[Ablation evidence] Table 6 shows that after removing the Z_sync synchronization additive pathway, IB drops from 0.26 to 0.22, V2ST WER rises from 7.59 to 12.40, and FD_VGG rises from 1.57 to 2.21 — all three metrics degrade simultaneously, confirming that the synchronization-feature pathway makes a substantive contribution to both audio-visual consistency and speech quality.
[Lip sync] [Uncertain] The paper does not deploy a dedicated lip-sync detector (such as SyncNet confidence) at the cleaning stage; the lip-sync quality of the VisualTTS branch relies on the datasets themselves (GRID/LRS2 are naturally lip-reading datasets with guaranteed synchronization) and on Synchformer's general-purpose synchronization criterion. This may be less precise than a dedicated lip-sync metric in pure-speech scenarios.

### Specific Synchronization Detection Metrics and Thresholds (SyncNet/Synchformer/LSE/in-house, threshold values, e.g. MOVA LSE-D≤9.5 and LSE-C≥4.5, SkyReels-V4 SyncNet |offset|≤3∧conf>1.5) ⚠️

[Specific values on the cleaning side]
  · Synchformer sync score ≥ 0.2 — the temporal-synchronization reliability bar
  · ImageBind (IB) score ≥ 0.3 — the audio-video semantic-consistency bar
  The two sit side by side within the Alignment dimension of Table 7, each an independent hard threshold; the pass condition is that both are satisfied simultaneously.
[Metrics on the evaluation side]
  · DeSync (lower is better) — temporal-alignment error. Foley-Omni 0.16 vs. Ground Truth 0.14 vs. cascaded baselines 0.85/0.26.
  · IB score (higher is better) — audio-video semantic consistency. Foley-Omni 0.26 vs. Ground Truth 0.36 vs. baselines 0.25/0.16. Note that the generated result's IB of 0.26 is of the same order of magnitude as the cleaning threshold of 0.3, indicating that the 0.3 bar is already moderately-to-strictly set relative to real data.
  · CLAP (higher is better) — semantic relevance between the audio and the text description. Foley-Omni 0.27 vs. GT 0.30.
  · WER (lower is better) — speech intelligibility. On V2ST-Bench, Foley-Omni scores 7.59, even lower than Ground Truth's 8.03 (because the generated speech is clearer than the real recording, with no environmental interference); GRID subset 15.3.
  · Three subjective MOS scores: A-MOS 3.92 (audio quality), S-MOS 4.13 (semantic consistency), T-MOS 4.14 (temporal synchronization), compared with GT's 4.33/4.37/4.42; the smallest gaps are on S-MOS and T-MOS.
[Assessment of threshold strictness] Compared with comparable industry work (such as UniVerse-1's strict SyncNet threshold, or the alignment screening in MMAudio-family work), Synchformer 0.2 and ImageBind 0.3 are moderately-to-loosely set, leaning toward retaining more data volume. The paper provides no threshold-sensitivity analysis.
[Uncertain] The basis for determining the thresholds is not explained (whether a grid search was conducted, and whether a retention-rate vs. performance trade-off curve exists, are both unreported). The specific computation configuration of the Synchformer/ImageBind scores (window length, feature aggregation method) is also not explained, affecting reproduction precision.

### Separate Handling of Temporal Synchronization vs. Semantic Synchronization (time alignment and content-semantic matching treated as two independent filtering conditions)

Separate handling is explicitly performed, a refined design point of this pipeline on the alignment dimension.
The Alignment dimension of Table 7 places two independent metrics with two independent thresholds side by side:
  · ImageBind score ≥0.3 handles [semantic consistency] — judging whether the audio content and the visual content are describing the same thing (the picture is a dog and the audio is a dog barking, vs. the picture is a dog and the audio is unrelated background music). ImageBind computes cosine similarity through a unified cross-modal embedding space, capturing "content matching."
  · Synchformer sync score ≥0.2 handles [temporal synchronization] — judging whether the audio event and the visual event align on the timeline (whether an impact visual and an impact sound occur on the same frame). Synchformer is trained specifically for temporal-alignment detection, capturing "time alignment."
[Why they must be separated] These two error types occur independently in real data and are different in nature: a video with a post-added soundtrack might be semantically consistent (a sad scene paired with sad music) but completely misaligned in time; off-screen narration might be temporally aligned (matching the editing rhythm) but semantically mismatched with the picture; misaligned material with wrong frames could have correct semantics but incorrect timing. Using a single composite score would let these three problem types mask each other — a high semantic score could compensate for a low temporal score, letting bad samples slip through. Setting two independent hard gates instead requires a sample to qualify on both dimensions.
[Corroboration from the paper's original text] The wording of the filtering condition places the two side by side as two independent removal reasons: "weak audiovisual semantic consistency, or unreliable synchronization" — joined by "or," confirming they are two independent rejection criteria.
[Extension into modeling and evaluation] This separation runs through the entire pipeline: on the model side, CLIP features carry semantics and Synchformer features (the Z_sync pathway) carry timing, injected through two separate paths; on the evaluation side, the IB score measures semantic consistency and DeSync measures temporal alignment, reported as two separate metrics. Cleaning, modeling, and evaluation all maintain the same semantic/temporal binary split — quite self-consistent methodologically.

### Audio Quality Filtering (SNR, silence detection and silence-ratio thresholds, no-audio-track removal, off-screen-voice-source removal, background music separation) ⚠️

Audio quality filtering is concentrated in a single metric plus one energy gate — a simple design.
[Primary filter] Meta AudioBox Aesthetics score ≥0.6. This is Meta's open-source no-reference audio aesthetics/quality assessment model, outputting a composite perceptual-quality score. Choosing a learned perceptual assessment over traditional signal metrics (SNR, THD, spectral flatness) reflects the 2025-2026 methodological evolution of audio data cleaning — traditional SNR cannot identify problems such as over-compression, clipping, encoding degradation, or mix imbalance, whereas AudioBox Aesthetics correlates more closely with human-ear judgment.
[Silence handling] The very first removal criterion in the filtering stage is silence. In implementation this is likely naturally covered by AudioBox Aesthetics's low score (a silent clip's aesthetic score would be extremely low); the paper does not describe an independent silence detector or a silence-ratio threshold. Samples with no audio track are naturally excluded (an audio track is a required supervisory target for this task). [Uncertain] No specific parameters are given for the energy threshold used to determine silence, or the upper bound on silence-duration ratio.
[Energy gating (field-level)] The acoustic post-verification stage applies an E(a_c) > −35 dB RMS energy gate to each separated stem. This is not clip-level filtering but field-level nullification — failing the bar only removes that field's annotation; the clip itself is still retained for training on the other fields. This granularity of design is critical: a clip with speech but no music is still a qualified VisualTTS/V2ST training sample and should not be discarded in its entirety.
[Background music separation] Bandit (a cinematic audio source separation model) is used for three-way speech/effects/music separation, but its purpose is to verify annotation rather than to clean the audio track — the separation result is not used as a training target; the model still learns to generate a complete mixed audio track. This differs from the approach in some V2A work of "separating out background music before training"; Foley-Omni instead requires the model to be able to generate a complete soundtrack including music.
[Off-screen voice source removal] [Uncertain] The paper does not describe a dedicated mechanism for identifying and removing off-screen sound, narration, or voiceover. The ImageBind ≥0.3 semantic-consistency bar can partially filter out narration unrelated to the picture, but this is not a purpose-built design.

### Classification and Separate Handling Strategy for Speech/Sound Effects/Music

The three-way classification of speech/sound-effects/music is the organizing thread running through the entire system — not merely a classification used at the cleaning step, but the same partition shared across five levels: task definition, annotation schema, verification tooling, training curriculum, and evaluation benchmark.
[Dual-path classification tooling]
  · Semantic discrimination path: Gemini 2.5 Pro's multimodal judgment of whether each of the three components is physically present (detect-then-describe).
  · Acoustic verification path: Bandit source separation produces three stems, secondarily confirmed or vetoed via −35 dB RMS energy gating.
  Bandit's selection is strictly isomorphic to the three-way classification (cinematic audio source separation naturally splits into speech/effects/music), avoiding the category-mismatch problem that would arise with a general-purpose source-separation model.
[Separate handling strategy]
  · Annotation layer: the three categories each occupy an independent field, [WORDS]/[AUDIO]/[MUSIC], each nullable.
  · Data layer: the three categories correspond to three single-task groups — TTS (1,253h) / TTA (912h) / TTM (139h) — with corresponding video-conditioned versions VisualTTS (1,980h) / V2A (403h) (music has no video-conditioned single-task group).
  · Training layer: Stage 1 first builds a foundation separately on the three single tasks (about 0.7M text-audio pairs, 5 epochs); Stage 2 introduces video conditioning (V2A + VisualTTS, 3 epochs); only at Stage 3 does joint generation of the three categories occur (V2ST, 2 epochs) — i.e., a "separate first, then combine" curriculum design.
  · Generation layer: the three categories are not generated separately and then mixed, but jointly generated within a shared latent space — classification is used for conditioning control and data organization, not for module decomposition at inference time.
  · Evaluation layer: V2ST-Bench is divided into three tiers by coexistence combination (speech+sound-effects, speech+music, all three present), with different metrics emphasizing different categories (WER measures speech, CLAP/IB measure sound-effect semantics, A-MOS provides an overall composite).
[Imbalanced mixture] Investment across the three categories is extremely uneven: speech-related is about 66%, sound-effects-related is about 27%, and music is only 2.8% (139 hours of TTM, with no independent video-conditioned music task group). This is a clear trade-off — the paper's core selling point is the improvement in "speech intelligibility," with music capability relatively playing a supporting role.

## Training Coordination

### Multi-Stage Training Curriculum and Data Curriculum Scheduling (basis for stage division: resolution/duration/quality score/modality; low-res→high-res, image→video, short→long) ⚠️

A three-stage progressive curriculum is adopted, with the division basis being [modal complexity + task compositeness], rather than the resolution/duration dimensions common in text-to-video work. The overall route is "text conditioning → adding visual conditioning → multi-component joint."
[Stage 1: Text-driven foundation] Tasks TTA + TTS + TTM, 5 epochs, about 0.7M text-audio pairs. The goal is to first let the model master each of the three audio-generation capabilities under purely textual conditioning, establishing an audio prior. This stage has no video input. It has the most epochs (5 rounds), indicating that establishing the basic audio-generation capability is given the largest training budget.
[Stage 2: Introducing visual conditioning] Tasks V2A + VisualTTS, 3 epochs. The video-conditioning pathway (CLIP semantic features + Synchformer synchronization features Z_sync) is added, learning the semantic mapping and temporal alignment of "vision → audio." This stage is still single-category audio tasks, only with a more complex conditioning modality.
[Stage 3: Joint soundtrack fine-tuning] Task V2ST, 2 epochs, using the 216 hours of V2ST data. Learns to jointly generate multiple audio-track categories within the same clip while maintaining mutual coordination. It has the fewest epochs, constituting a lightweight fine-tune, consistent with the conventional approach of "using small-scale, high-quality data for final alignment."
[Quantitative evidence for the curriculum's effectiveness] The Table 6 ablation shows that if the curriculum is skipped in favor of direct single-stage joint training, performance collapses across the board: V2ST WER worsens from 7.59 to 29.29 (nearly a 4x degradation), GRID WER from 15.3 to 27.4, IB drops from 0.26 to 0.24, and FD_VGG rises from 1.57 to 1.73. Speech intelligibility suffers the most severe damage, indicating that a modality as structurally demanding and error-intolerant as speech is especially dependent on prior single-task pretraining — under direct mixed training, speech capability is drowned out by the gradients from the sound-effects/music tasks. This is this paper's most compelling piece of evidence regarding the data/training curriculum.
[Uncertain] The learning rate, batch size, total training steps, hardware configuration, and training duration for each stage are not disclosed in the accessible content.

### Changes in Data Mixture Across Training Stages (pretraining/annealing/SFT high-quality subset) ⚠️

The core change in mixture across stages is "expansion of the task set + introduction of a replay mechanism."
[Stage 1 → Stage 2] Data switches from pure text-audio pairs (TTA 912h + TTS 1,253h + TTM 139h, about 0.7M pairs) to video-audio pairs (VisualTTS 1,980h + V2A 403h). The task set is completely replaced rather than layered on top.
[Stage 2 → Stage 3] The switch is to V2ST data (216 hours, internal + SpeakerVid), but the key point is the introduction of an explicit experience-replay mechanism to mitigate catastrophic forgetting: alongside the V2ST fine-tuning data, 100 hours of data from each of the preceding single-task domains are mixed in. By this reckoning, Stage 3's mixture is roughly V2ST's 216 hours + 100 hours each from five preceding task domains (about 500 hours total), meaning the target task accounts for about 30% and replay data about 70% — a fairly high replay proportion, reflecting how seriously the authors treat the forgetting problem.
[Argument for the necessity of replay] This corroborates the single-stage training ablation in Table 6: in multi-task audio generation, different tasks interfere with each other severely (especially speech vs. non-speech) — one cannot mix-train from the start (as the stage ablation proves), nor can one completely abandon prior data later (hence the need for replay); balance can only be achieved through "separate first, then combine + a high replay ratio."
[Annealing] The paper does not use the term "annealing," but Stage 3's use of small-scale (216h) high-quality V2ST data + a decreasing epoch count (5→3→2) for the final fine-tune is functionally equivalent to an annealing stage.
[Uncertain] The sampling weights among different datasets within each stage (whether by natural hour-count proportion or weighted sampling) are not disclosed; nor is it stated whether the replay data is randomly drawn 100 hours or selected by quality score.

### Post-Training Data (SFT curated-set size and selection criteria, number of preference pairs and annotation method, reward-model training data) ⚠️

This paper's training pipeline ends with the three-stage supervised curriculum (Stage 3's V2ST fine-tuning is the final stage), and does not include a separate RLHF/DPO post-training step.
[The stage closest to SFT] Stage 3 can be regarded as an SFT-like stage: it uses the smallest-scale (216 hours) data, most closely matched to the final goal (V2ST complete soundtrack), for a lightweight 2-epoch fine-tune, paired with 100 hours of per-domain replay to prevent forgetting. The selection criteria are simply that a sample passed the complete cleaning pipeline (the six filtering thresholds + Gemini annotation + Bandit energy verification) and contains multiple audio components.
[Preference data] [Uncertain] No construction of preference pairs is seen, no reward model is trained, and no DPO/PPO-type preference optimization is used. The A-MOS/S-MOS/T-MOS subjective scores are used only for final evaluation and are not fed back as a training signal.
[Manually curated set] The 300 samples of V2ST-Bench, manually reviewed along three dimensions, are the only manually curated set in the entire pipeline, but they are explicitly used for evaluation rather than training.
Compared with the preference-alignment stage commonly found in industrial-grade models (Veo 3, Sora 2, etc.), Foley-Omni, as academic work, is blank on the post-training dimension — implying there remains room for improvement on subjective dimensions such as audio aesthetics and fit to creative intent (A-MOS 3.92 vs. GT's 4.33 is the largest gap among the three subjective scores).

### Data Processing Infrastructure and Throughput (NeMo Curator/Data-Juicer/in-house, GPU speedup ratio, processing scale, cost) ⚠️

[Uncertain] The paper discloses no data-processing infrastructure information at all: no mention of NeMo Curator, Data-Juicer, or an in-house distributed cleaning framework; no GPU speedup ratio, processing throughput (hours/GPU-day), cluster scale, or cleaning cost is given.
An inferrable cost structure: the most expensive step in the pipeline should be Gemini 2.5 Pro's API calls — performing multimodal (video frame + audio track) annotation on approximately 2.0M video clips represents significant expense at commercial API pricing, yet the paper does not discuss call volume or cost-control strategy at all (e.g., whether the prompt was first validated at small scale, or whether a downgraded model is used for some routing). This is a topic commonly avoided by work that uses closed-source APIs for large-scale annotation.
The remaining steps (resolution/bitrate parsing, AudioBox Aesthetics scoring, ImageBind/Synchformer inference, Bandit source separation) are all lightweight models amenable to batched GPU inference; their total compute demand should be an order of magnitude lower than Gemini annotation, but likewise no data is disclosed.
On the open-source side, the GitHub repository provides preprocessing extraction scripts for visual features (CLIP + Synchformer) — a minimally usable tooling support, but not a complete cleaning infrastructure.

## Results Comparison

### Quantitative Impact of Data-Strategy Ablations (distinguishing: filtering-strictness ablation / caption-density-style ablation / data-mixture ablation, and corresponding evaluation metrics) ⚠️

The paper's ablation experiments (Table 6) focus on two categories — [training curriculum] and [model architecture] — and do not perform any pure data-strategy ablation.
[Ablations performed — quantitative results]
  Metric definitions: FD_VGG↓ (Fréchet Distance on VGGSound, audio fidelity), WER_GRID↓ (speech intelligibility on GRID), IB_V2ST↑ (audio-video semantic consistency on V2ST-Bench), WER_V2ST↓ (speech intelligibility on V2ST-Bench).
  · Full model: FD_VGG 1.57 / WER_GRID 15.3 / IB_V2ST 0.26 / WER_V2ST 7.59
  · Single-stage training (removing the three-stage curriculum): 1.73 / 27.4 / 0.24 / 29.29 — WER_V2ST worsens by 3.9x, WER_GRID worsens by 79%, all four metrics degrade across the board. This belongs to a training-curriculum/data-scheduling ablation, and can be classified as a broadly defined data-mixture ablation: it proves that "feeding data in stages" has a decisive advantage over "one-shot mixing," and that the speech task suffers the most severe damage.
  · Removing the Z_sync synchronization additive pathway: 2.21 / 18.9 / 0.22 / 12.40 — IB drops from 0.26 to 0.22 (the most direct degradation in audio-video consistency), FD_VGG rises from 1.57 to 2.21 (a 41% degradation in audio fidelity), and WER_V2ST rises from 7.59 to 12.40. This proves that the Synchformer temporal-feature pathway makes a substantive contribution across all three dimensions.
[Ablations not performed — clear gaps] [Uncertain]
  · Filtering-strictness ablation: none of the six thresholds (480p / 1Mbps / motion[0.1,3.2] / AudioBox 0.6 / IB 0.3 / Sync 0.2) has ever undergone a threshold-sensitivity analysis, and there is no "loose filtering vs. strict filtering" performance comparison curve.
  · Caption density/style ablation: no comparison is made between the three-field structured annotation and a single free-form description (such as LTX-2-style full-soundscape description) — this would have been the most direct experiment for verifying the value of its core contribution, the "three-field schema," and its absence is rather regrettable.
  · Acoustic post-verification ablation: no comparison of "with/without Bandit energy gating" is performed, so the actual benefit brought by correcting visual hallucination cannot be quantified, nor can the reasonableness of the −35 dB threshold be verified.
  · Data-mixture ablation: no comparison is made of the impact of different hour-count mixtures across task groups (e.g., increasing the proportion of music).
  Taken together, this paper's ablations of model architecture and training curriculum are thorough, but it has performed almost no ablation verification of what it claims as one of its core contributions — the "data cleaning pipeline" — whose effectiveness is supported mainly by methodological narrative rather than experimental evidence.

### Evidence for Quality vs. Quantity (cases where small, refined data outperforms large, messy data) ⚠️

There is no direct controlled experiment for "small, refined data outperforming large, messy data," but several designs indirectly embody this idea.
[Indirect evidence one: Stage 3's small-data, high-efficiency fine-tuning] The final V2ST capability is acquired from only 216 hours of data (4.4% of the total) within 2 epochs, yet reaches WER 7.59 (better than Ground Truth's 8.03) and DeSync 0.16 (close to GT's 0.14). This shows that after a solid foundation is built in the preceding large-scale stages, the target task only needs an extremely small amount of highly matched data to align — a manifestation of "small and refined," though there is no control group of "training with a larger-scale but lower-quality V2ST dataset."
[Indirect evidence two: the pipeline's strict-filtering orientation] The design of six filtering thresholds + field-level energy gating itself embodies a quality-first orientation, especially in acoustic post-verification's willingness to leave a field empty (reducing the supervisory signal) rather than retain a possibly incorrect annotation — a clear case of "better to have less than have it wrong."
[Indirect evidence three: strict manual review of the evaluation set] The 300 evaluation samples underwent three-dimensional manual gatekeeping — a scale far smaller than common benchmarks, but with controlled quality.
[Counter-evidence] The data composition also reflects a pursuit of "quantity": VisualTTS invests 1,980 hours (40%) to support speech capability, showing that for this key capability the authors chose a large-scale-data route. It could be said that this paper follows a hybrid strategy of "quantity for foundational capability, quality for task alignment."
[Uncertain] The paper performs no data-scale scaling experiment at all (e.g., a performance comparison when trained with 50% of the data), so the quality-quantity trade-off cannot be quantitatively demonstrated.

### Alignment Between Training-Data Domain Distribution and Evaluation-Benchmark Taxonomy (e.g. VBench's seven major categories) ⚠️

The organizing dimension of the training data is highly aligned with the evaluation benchmark's taxonomy, and the alignment axis is "audio component type" rather than visual domain — this is a fundamental difference between Foley-Omni and text-to-video models (which align to visual taxonomies such as VBench/VABench's seven major categories).
[Alignment relationship]
  · Training-side task groups: TTS / TTA / TTM (three single tasks) → VisualTTS / V2A (video-conditioned single tasks) → V2ST (multi-component joint)
  · Annotation-side fields: [WORDS] / [AUDIO] / [MUSIC]
  · Verification-side stems: Bandit's speech / effects / music three-way separation
  · Evaluation-side V2ST-Bench tiers: speech+sound-effects 150 clips / speech+music 120 clips / all three present 30 clips
  All four are strictly isomorphic to the same speech-SFX-music three-way classification, forming a closed loop. This full-pipeline alignment means "however the training data was labeled is exactly how the evaluation tests it" — a very strong consistency.
[Correspondence between evaluation metrics and categories] WER corresponds to the [WORDS] speech field, CLAP/IB correspond to the semantic matching of the [AUDIO] sound effects, DeSync corresponds to cross-category temporal alignment, and A/S/T-MOS provide overall subjective supplementation.
[Imbalance within the alignment] Speech appears in 100% of V2ST-Bench's three tiers (150+120+30 all include speech), while a combination of pure sound-effects + music with no speech is entirely absent — the evaluation benchmark's speech-centric tendency is consistent with, and mutually reinforces, the training data's 66% speech proportion. This means the benchmark lacks coverage of "complete soundtracks for non-speech scenarios" (such as sound effects + score for nature documentaries or action sequences), and the model's capability in such scenarios goes untested.
[External benchmark alignment] Single-task performance is evaluated on general-purpose benchmarks: VGGSound (FD_VGG, the standard V2A benchmark) and GRID (WER, the standard lip-sync speech benchmark), ensuring comparability with expert systems. The paper states that on single tasks it "achieves competitive performance with expert systems."
[Uncertain] No cross-alignment analysis is performed against other benchmarks such as VABench, the official AudioCaps leaderboard, or the Kling-Foley evaluation set; whether V2ST-Bench is strictly non-overlapping with the training set is also not explicitly stated.

## Uncertain Fields

The research information for the following fields is partly uncertain (marked with the ⚠️ source annotation):

- data_scale
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
- audio_quality_filtering
- multi_stage_curriculum
- stage_data_mixture
- post_training_data
- data_infra_throughput
- data_ablation
- quality_over_quantity_evidence
- benchmark_taxonomy_alignment
