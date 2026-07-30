# MOVA (MOSS Video and Audio)

> Topic: Data processing for video generation models (including simultaneous audio-video generation): data cleaning pipeline, data distribution, annotation methods

[← Back to home](../index.md)

**Contents**: [Basic Information](#basic-information) · [Data Scale and Distribution](#data-scale-and-distribution) · [Cleaning Pipeline](#cleaning-pipeline) · [Annotation Methods](#annotation-methods) · [Audio-Video Alignment](#audio-video-alignment) · [Training Coordination](#training-coordination) · [Performance Comparison](#performance-comparison)

## Basic Information

### Name

MOVA (MOSS Video and Audio)

### Publishing organization/company

SII-OpenMOSS team. The paper's author affiliations include: Shanghai Innovation Institute, MOSI Intelligence (无问芯穹/MOSI Intelligence), Fudan University, Shanghai Jiao Tong University, East China Normal University, Tongji University, Southeast University, Xiamen University, and University of Electronic Science and Technology of China. The project leads are Qinyuan Cheng (程琴媛) and Tianyi Liang; the corresponding authors are Xie Chen (Shanghai Jiao Tong University, chenxie95@sjtu.edu.cn) and Xipeng Qiu (Fudan University, xpqiu@fudan.edu.cn). It belongs to the audio-video generation branch of Fudan's OpenMOSS (MOSS series) open-source ecosystem.

### Release date (technical report/paper/open-source date)

First open-source release on January 29, 2026 (model weights + code, simultaneously launched on GitHub/HuggingFace); the 38-page technical report arXiv:2602.08794 was released in February 2026 (v2 dated February 10, 2026, cs.CV); the API went live on March 9, 2026; and open-source evaluation code was added on May 6, 2026.

### Type (model/dataset/toolchain/evaluation benchmark)

Model (an audio-video joint generation foundation model). It also comes with three derivative outputs: (1) a complete open-source codebase (training pipeline, inference, LoRA fine-tuning, prompt-enhancement workflow, evaluation code); (2) a self-built six-category scenario audio-video joint generation evaluation benchmark (used alongside Verse-Bench); (3) an Arena-style human preference evaluation protocol. This is not a dataset release (the training data itself is not open-sourced).

### Degree of openness (whether weights/code/data/pipeline are each open-sourced)

The degree of openness is among the highest in comparable audio-video joint generation models, released under the Apache-2.0 license, permitting unrestricted commercial use.
【Weights】Open-sourced. Two variants released: MOVA-360p and MOVA-720p (HuggingFace: OpenMOSS-Team/MOVA-360p, OpenMOSS-Team/MOVA-720p, and the mova collection).
【Code】Open-sourced and covers the full pipeline: training pipeline, efficient inference, LoRA fine-tuning scripts, prompt-enhancement (rewriter) workflow, and evaluation code. The paper explicitly states "we release all model weights along with training, inference, and fine-tuning code."
【Data processing pipeline】At the methodological level, industrial-grade detail is disclosed in Section 3 and Appendix A.3/A.4/A.5 of the paper — including the complete three-stage funnel structure, per-metric filtering threshold tables (Table 9), stage-by-stage retention rate tables (Table 1), speech-window segmentation pseudocode (Algorithm 1/2), and the full text of all annotation prompts along with complete caption examples. This is one of the most reproducible samples of data processing methodology found in this survey. However, the codebase does not separately release the data cleaning scripts — it only provides the dataset interface mova/datasets/video_audio_dataset.py, requiring users to supply their own video-audio data and integrate it via configuration.
【Data】The training data itself is not open-sourced. The public dataset portions (VGGSound, AutoReCap, ChronoMagic-Pro, ACAV-100M, OpenHumanVid, SpeakerVid-5M, OpenVid-1M) can be obtained independently, but the specific manifest of their "filtered HQ subsets" has not been released; the in-house data (Chinese dramas, animation, movies, YouTube scraping, etc.) is not public.

### Whether simultaneous audio-video generation is supported, and the implementation approach (native joint / cascaded / MoE fusion)

Supported, and it is native joint generation (a single inference pass produces video and audio simultaneously, not cascaded). The implementation approach is an "asymmetric dual-tower architecture + bidirectional cross-attention Bridge fusion," with the video tower itself being an MoE:
- Total parameters: 32B, with 18B active at inference time.
- Video tower: Wan2.2 I2V A14B (MoE architecture, containing high-noise and low-noise DiT expert groups that switch based on timestep t and threshold δ; during training, an alternating optimization strategy is used — odd steps optimize the high-noise DiT and even steps optimize the low-noise DiT — to satisfy the computational graph consistency requirements of FSDP).
- Audio tower: a self-trained 1.3B text-to-audio DiT, built on the Wan2.1-1.3B backbone, replacing the 3D positional encoding (f, h, w) with 1D positional encoding along the time axis.
- Fusion: a lightweight Bridge module inserts two cross-attention blocks (video→audio, audio→video) at the hidden-state level between the two DiT backbones, with 30 layers of interaction.
- Aligned RoPE: because the video latent's temporal grid is coarse while the audio latent is dense, the video index is scaled and mapped to audio time units via s = f_a/f_v (p_v(i)=s·i, p_a(j)=j), placing both modalities on the same time scale to avoid temporal misalignment drift in cross-modal attention.
- VAE: the video side uses the Wan2.1 video VAE; the audio side uses a DAC-style audio VAE from HunyuanVideo-Foley (48kHz mono); both are frozen throughout training.
- Training objective: flow matching; Dual Sigma Shift allows video and audio to independently sample timesteps and noise schedules.
- Inference: Dual Classifier-Free Guidance (text CFG + cross-modal Bridge CFG dual branches).
The task form is IT2VA (image + text → video + audio), and it exhibits emergent T2VA capability (a pure text-driven mode, achieved by substituting a plain white placeholder image for the reference image).

### List of research information sources (URLs of papers/technical reports/official documentation/news, with each source's nature labeled: official primary source/same-team corroboration/third-party report)

- 【Official primary source】arXiv:2602.08794v2, "MOVA: Towards Scalable and Synchronized Video–Audio Generation" technical report (38 pages, February 10, 2026): https://arxiv.org/abs/2602.08794, PDF https://arxiv.org/pdf/2602.08794 — the sole and direct source of information for the vast majority of the fields in this entry, particularly Section 3 (Data Engineering), Section 4.3 (Progressive Joint Training), and Appendices A.1/A.3/A.4/A.5/A.6. Note: arXiv does not provide an HTML version (/html/2602.08794v2 returns "No HTML"), so the PDF had to be parsed directly.
- 【Official primary source】GitHub repository OpenMOSS/MOVA: https://github.com/OpenMOSS/MOVA — inventory of open-source content, Apache-2.0 license, release timeline (2026-01-29 initial release / 2026-02-10 technical report and inference workflow / 2026-03-09 API / 2026-05-06 evaluation code), and dataset interface mova/datasets/video_audio_dataset.py.
- 【Official primary source】HuggingFace model pages: https://huggingface.co/OpenMOSS-Team/MOVA-720p, https://huggingface.co/OpenMOSS-Team/MOVA-360p, collection https://huggingface.co/collections/OpenMOSS-Team/mova; paper page https://huggingface.co/papers/2602.08794
- 【Official primary source】Project blog: https://mosi.cn/models/mova
- 【Third-party report】ComfyUI-Wiki news article "OpenMOSS Releases MOVA - Open-Source Synchronized Video-Audio Generation" (2026-01-29): https://comfyui-wiki.com/en/news/2026-01-29-openmoss-mova-video-audio-generation — used to cross-check the initial release date.
- 【Third-party report】AI Films Studio blog "MOVA: Open Source Video-Audio Generation": https://studio.aifilms.ai/blog/mova-open-source-video-generation — used to cross-check the Apache-2.0 commercial-license description.

## Data Scale and Distribution

### Training data volume (number of video clips/hours/tokens, pretraining vs. SFT separated)

Broken down by training stage (the paper gives both "hours" and "number of clips" as dual metrics, without token counts):
【Audio tower pretraining】Uses three major domains: WavCaps + VGGSound (general sound effects), JamendoMaxCaps (music), and in-house TTS data, training on fixed-length segments; specific hour counts/clip counts are not disclosed.
【Joint training Phase 1 (360p, diverse data)】Approximately 61,500 hours of video-audio data, 1 epoch, taking 15 days.
【Joint training Phase 2 (360p, quality-filtered)】Approximately 37,600 hours / 16.8M clips (16.8M × 8.05s ≈ 37,560 hours, self-consistent), 1 epoch, taking 7 days.
【Joint training Phase 3 (720p, highest-quality subset)】Approximately 11,000 hours, 1 epoch, taking 20 days.
【Total】The three stages together take 42 days, using 1024 GPUs (128 nodes × 8 cards each), approximately 43,000 GPU-days.
Note: MOVA has no separate SFT / RLHF post-training stage, so there is no "pretraining vs. SFT" scale breakdown; instead, there is a decreasing data scale within the three-stage progressive curriculum (61.5k → 37.6k → 11k hours), forming a classic "decreasing scale, increasing quality" pyramid.

### Composition of data sources (proprietary/public datasets/web scraping/licensed procurement/synthetic data)

Two major sources: high-quality subsets of public datasets + a large amount of proprietary (in-house) data.
【Public datasets (all using their filtered HQ subsets)】
- VGGSound (an audio-video event alignment dataset)
- AutoReCap (large-scale audio generation data)
- ChronoMagic-Pro (time-lapse/morphing-type video)
- ACAV-100M (an automatically curated large-scale audio-video representation learning dataset)
- OpenHumanVid (a person-centric video generation dataset)
- SpeakerVid-5M (an audio-video dual-person interaction human-generation dataset, the core source of lip-sync capability)
- OpenVid-1M (a high-quality text-to-video dataset)
【In-house data】The paper states "a large amount of in-house data." The data sources explicitly listed for Phase 1 are: SpeakerVid5M, Chinese drama, cartoon animation, movies, YouTube, and OpenHumanVid.
【Content forms and themes】Video forms cover movies, vlogs, and animation; themes cover education, sports, beauty, news, interviews, animation, and more.
【Synthetic data】No model-synthesized video-audio training data is used; the only "synthetic" component is the in-house TTS data used in audio tower pretraining, along with the fact that all captions are automatically generated by an MLLM/LLM (synthetic annotation rather than synthetic content).
【Licensed procurement】The paper does not mention any paid, licensed, procured data.

### Data compliance and provenance (proportion of licensed data, rights-cleared datasets, C2PA, etc.) ⚠️

The paper does not discuss data compliance and provenance issues at all: no proportion of licensed data is given, no rights-cleared datasets are declared, no C2PA or any output-side watermarking/provenance marking is mentioned, and there is no discussion of copyright risk, portrait rights, or the legal status of data sources. What can be inferred indirectly is that the training corpus includes YouTube-scraped content, Chinese drama, and movie clips, all of which are copyright-sensitive sources; and the cited public datasets themselves (ACAV-100M, VGGSound, etc.) are mostly in the form of "collections of YouTube links," whose copyright status is governed by the terms of the original datasets. The model weights are released under the Apache-2.0 open commercial license, but no explanation whatsoever is given of the training data's compliance — this is a clear asymmetry in MOVA, where "methodological transparency is extremely high" while "compliance transparency is zero." [Uncertain]

### Clip duration distribution and segmentation strategy

A strict fixed-duration segmentation strategy is adopted, with no variable-length bucketing:
- All training clips are uniformly 8.05 seconds, precisely corresponding to 193 frames at 24fps (1 initial frame + 8 seconds of video, i.e., 1 + 8×24 = 193).
- The frame count across all three training stages is constant at 193, regardless of resolution.
- Segmentation is not equal-interval sliding-window, but rather a window-generation algorithm jointly driven by VAD (voice activity detection) + PySceneDetect (scene-cut detection) (see shot_segmentation and the two pseudocode blocks in Appendix A.3).
- The window start point for speech segments is adaptively shifted/adjusted forward to avoid truncating an ongoing utterance, ensuring the continuity of spoken content.
- Inference-side output is likewise about 8 seconds (a 720p, 8s, 24fps clip produces approximately 1.6×10^5 tokens; the paper lists sequence length as a major bottleneck in the Limitations section).

### Resolution/aspect ratio distribution and bucketing strategy

【Standardization pipeline】Raw videos are first processed with the FFmpeg cropdetect filter to detect and remove black bars while preserving the core frame content, then the subject content is centered, scaled to 720p, and symmetric padding is added as needed (pillarbox or letterbox), unified into either a 9:16 or 16:9 aspect ratio. Frame rate is uniformly resampled to 24fps. Consequently, the data side has only two aspect ratios, with no multi-bucket aspect-ratio training.
【Resolution curriculum】Phase 1 and Phase 2 train at 360×640 (360p), and Phase 3 upsamples to 720×1280 (720p). Correspondingly, two models are released: MOVA-360p and MOVA-720p.
【Engineering impact】In the 720p stage, sequence length increases substantially — context parallelism increases from CP=8 to CP=16, effective batch size drops from 128 to 64, and the checkpoint interval shortens from 5000 steps to 2000 steps.
【Effect】Ablation shows that as resolution goes from 360p to 720p, DeSync slightly decreases from 0.475 to 0.485 and IB-Score slightly decreases from 0.286 to 0.277 (essentially no degradation), while LSE-C actually improves from 6.278 to 6.593 and cpCER drops from 0.177 to 0.149, validating the effectiveness of the curriculum that first establishes cross-modal alignment at low resolution and then upscales.

### Category/domain distribution and mixture strategy (proportion control and concept balancing for people, actions, scenes, styles, etc.) ⚠️

The paper only gives a qualitative enumeration of domain coverage for the training data, without providing any proportion figures, and does not describe a concept-balancing mechanism:
- Video forms: movies, vlogs, animations, Chinese drama, cartoons, YouTube content.
- Thematic domains: education, sports, beauty, news, interviews, etc.; the paper states this provides "the distributional diversity needed to generalize to complex real-world scenarios."
- The only confirmable strong mixture bias is "centered on people speaking": among the Phase 1 data sources, both SpeakerVid-5M and OpenHumanVid are person/speaker-centric datasets, and the final training set only retains speech segments (see audio_category_distribution), indicating that people/dialogue-type data occupies an overwhelmingly dominant share of the mixture, which is consistent with the model's positioning as focused on multilingual lip sync.
- The only category pie chart with percentages in the paper (Figure 6a, including "others 2.3%") describes the sample category distribution of the self-built evaluation benchmark, not the training data distribution, and should not be conflated with it.
The domain mixture figures for the training data are an information gap. [Uncertain]

### Audio category distribution and mixture (proportions and control strategy for speech/sound-effect foley/music/ambient sound/silence) — an AV-model-specific dimension

This is the most decisive aspect of MOVA's data design — the final training set **retains only speech clips**, an extreme audio-category-mixture choice:
【Preprocessing-stage classification】Silero VAD is used to cut the audio track into speech / non-speech intervals, which are combined with PySceneDetect's scene-cut points to generate four types of fixed-length 8.05-second clips: single-scene speech, single-scene non-speech, multi-scene speech, and multi-scene non-speech.
【Key mixture decision】"Ultimately, only speech segments are selected for training, accounting for 69.47% of all preprocessed segments." That is, speech segments make up 69.47% of all preprocessed clips, and ultimately only this portion is used to train the joint model. Corresponding to the total-duration retention rate (Table 1): raw 100% → Stage 1 (speech + non-speech) 84.57% → Stage 1 (speech only) 58.75%. In other words, the "speech only" condition alone eliminates about 26 percentage points of data.
【Audio-type classifier】An EAT self-supervised audio Transformer classification model is used to tag audio, constructing speech / non-speech subsets, routed according to the target capability (lip sync vs. general foley/ambient-sound modeling). The construction condition for the speech subset is that both the EAT-contained-Speech and EAT-contained-Singing labels are judged True (or satisfy the model's positive-class confidence).
【Category mixture in audio tower pretraining】Unlike joint training, the 1.3B audio tower pretraining deliberately covers three major categories: general sound effects (WavCaps + VGGSound), music (JamendoMaxCaps), and speech (in-house TTS). That is, a two-stage division of labor: "sound-effect/music capability is injected during audio tower pretraining, while speech-lip-sync capability is reinforced during joint training."
【Cost】The paper's Limitations section explicitly acknowledges that because the audio tower is only 1.3B and joint training is speech-biased, the model degrades in performance on singing, complex timbral textures, and music/instrumental content.
【Undisclosed】The detailed proportions within non-speech clips (sound effects / music / ambient sound / silence) are not given.

### Narrative structure distribution (single-shot vs. multi-shot, average clip duration, shot-count distribution, whether native audio track is included)

MOVA explicitly makes "single-shot vs. multi-shot" an orthogonal data dimension, which is relatively uncommon in comparable work:
- Through the cross-combination of VAD temporal information and PySceneDetect scene-cut points, four types of clips are generated: single-scene speech, single-scene non-speech, multi-scene speech, multi-scene non-speech (i.e., a 2×2 division of {single/multi-shot} × {speech/non-speech}).
- The condition for a multi-shot clip is strict: the window's time span must contain at least one scene-cut point, or it is discarded (Appendix A.3, Algorithm 1, line 11).
- The condition for a single-shot clip is likewise strict: the window must fall entirely between two adjacent scene-cut points (Algorithm 2, lines 5 and 19).
- The average clip duration is fixed at 8.05 seconds (193 frames @24fps), so there is no length distribution to speak of.
- Whether a native audio track is included: the training data must come with its own native, synchronized audio track — the very first preprocessing step discards samples with "decoding failure" or "missing valid audio channels," so no audio-track-free samples exist.
- The annotation stage also imposes specific requirements for multi-shot content: the video-captioning prompt explicitly instructs MiMo-VL to "focus on video scene transitions," i.e., to explicitly annotate shot transitions.
- The specific ratio of single-shot to multi-shot clips is not disclosed.

### Language/accent distribution (the data basis for multilingual lip-sync capability) ⚠️

The paper positions MOVA as having "multilingual speech with high-quality lip synchronization," but only provides qualitative evidence for language/accent distribution, with no proportion figures whatsoever:
【Confirmable language coverage】Chinese and English are the two languages explicitly demonstrated and evaluated. Figure 1 shows two groups of precise lip-sync cases — English multi-speaker and Chinese multi-speaker (the Chinese example includes an adult-child dual-speaker dialogue) — and also shows Chinese on-screen text generation. The subjective Arena evaluation set is deliberately constructed as bilingual: half of the originally pure-English Verse-Bench speech data was manually translated, forming a Chinese-English bilingual mixed evaluation set of 732 items (600 from Verse-Bench + 132 from the self-built benchmark).
【The language basis on the data side】Chinese capability mainly comes from in-house Chinese drama corpora; English capability mainly comes from SpeakerVid-5M, OpenHumanVid, and YouTube content.
【Accent annotation】The ASR transcription prompt explicitly requires "LAW OF LANGUAGE FIDELITY: Preserve the original language. No translation," so the captions naturally retain the multilingual original text; and the audio captions describe the speaker's accent (the complete caption example given in the paper includes "with a General American accent"), indicating that accent is one of the attributes naturally described in language by Qwen3-Omni-Captioner, but it is not a structured enumerated field.
【Gaps】The list of supported languages, the hour-count proportion of each language, and the accent-category distribution are all undisclosed; the paper also mentions in the Discussion that phoneme-viseme mapping differences across languages are a difficulty, but does not quantify this. [Uncertain]

## Cleaning Pipeline

### Overall structure of the cleaning funnel (number of filtering stages, order of stages)

A three-stage funnel (Figure 3), implemented on the Ray distributed framework, using a mix of NVIDIA GPU and Huawei Ascend NPU compute:
【Stage 0 — Entry-level exclusion】Samples with decoding failure or missing valid audio channels are discarded; samples with abnormal container formats are remuxed, and those with abnormal codecs are transcoded.
【Stage 1 — Video preprocessing and standardization (Standardization → Detection → Segmentation)】
  1) Core content normalization: FFmpeg cropdetect detects and removes black bars while preserving core content → centering → scaling to 720p → symmetric padding into 9:16 or 16:9; frame rate resampled to 24fps.
  2) Voice activity detection: Silero VAD labels speech / non-speech intervals.
  3) Scene-transition analysis: PySceneDetect records the timestamps of all scene-cut points across the video.
  4) Segmentation: VAD and scene-cut timing information are fused to generate 8.05-second fixed-length clips of four types (single/multi-scene × speech/non-speech), with the start point of speech clips adaptively adjusted to avoid truncating utterances. Only speech clips are retained (69.47% of preprocessed clips).
【Stage 2 — Audio-video quality assessment (three-dimensional parallel filtering)】Audio quality (signal-level + aesthetic), video quality (technical + aesthetic), and audio-video alignment (temporal + semantic); thresholds are empirically set by manually inspecting retained videos under different cutoffs. An EAT classification model is also used to build speech/non-speech-specific subsets.
【Stage 3 — Joint audio-video annotation】MiMo-VL-7B-RL performs visual description, Qwen3-Omni-Instruct performs speech transcription, Qwen3-Omni-Captioner performs non-speech sound-effect/music description, and finally GPT-OSS-120B performs cross-modal consistency verification and merges everything into a unified natural-language caption.
【Secondary filtering within the training stage】Before Phase 2, three additional orthogonal filters are layered on: OCR (no burned-in subtitles), LSE lip-audio correspondence, and DOVER technical score, see quality_filtering and stage_data_mixture.
Overall characteristic: "standardize + segment" first, then "score and filter," and finally "annotate" — placing the most expensive MLLM annotation at the very end of the funnel, so annotation is only performed on clips that have already passed the quality and alignment filters, which is a reasonable ordering for cost control.

### Quantitative funnel retention rates (input/output volume at each filtering stage and final retention rate, e.g., Apollo's 27%)

The paper gives, via Table 1 ("Retention Ratio of Total Dataset Duration"), the stage-by-stage retention rate by total duration (relative to the original video) — a rare, publicly disclosed quantitative funnel in this survey:
- Raw (original video): 100%
- Stage 1 (after preprocessing, speech + non-speech clips): 84.57%
- Stage 1 (speech clips only): 58.75%
- Stage 2 (after quality and alignment filtering): 26.39%
That is, the final overall retention rate is 26.39%, on the same order of magnitude as Apollo's 27%. Breaking it down, the two main sources of loss are: (1) the "speech only" condition, which drops from 84.57% → 58.75%, eliminating 25.8 percentage points and constituting the single largest source of elimination (there is also a clip-count-based figure elsewhere: speech clips make up 69.47% of all preprocessed clips); (2) the three-dimensional quality and alignment filtering, which drops from 58.75% → 26.39%, eliminating a further 32.4 percentage points, with a relative retention rate of about 44.9%.
In addition, the training stage applies an even stricter secondary narrowing (relative to Phase 1's 61,500 hours): Phase 2 drops to approximately 37,600 hours / 16.8M clips (about 61%), and Phase 3 drops further to approximately 11,000 hours (about 18% of Phase 1). If the training curriculum is also counted as an extension of the funnel, the effective retention rate from raw material to the final 720p high-quality subset is far below 26.39%.
The output volumes of Phase 2's three sub-filters are also publicly given: OCR no-subtitle subset ~9.5M clips, LSE lip-audio high-quality subset ~2.5M clips, DOVER technical-score >0.15 subset ~2.4M clips; the merged Phase 2 dataset totals 16.8M clips.

### Shot segmentation method (PySceneDetect / self-developed model / shot-aware splitting)

PySceneDetect is used for scene-cut detection, but MOVA's key innovation lies in jointly combining scene-cut points with speech boundaries for window sampling, rather than simply splitting by scene:
【Detection】PySceneDetect detects and records the timestamps of all scene-transition points across the video; Silero VAD labels speech intervals in parallel.
【Multi-shot window generation (Appendix A.3, Algorithm 1)】Iterate over VAD speech segments: the window's start-point upper bound is set to the start of the current speech segment; the lower bound takes the maximum of three values — the end time of the previous speech segment, the nearest scene-cut point prior to the current speech segment, and the current speech segment's start time minus half the window length (8.05/2); a window start point is uniformly randomly sampled within this interval; the window length is fixed at 8.05 seconds; the window is retained as a multi-shot sample only if its time span contains at least one scene-cut point; the process then jumps to the first speech segment whose start time is later than the window's end time, avoiding window overlap.
【Single-shot window generation (Appendix A.3, Algorithm 2)】Iterate over the scene intervals formed by adjacent scene-cut points, and within each scene, look for speech segments whose start point satisfies "start + 8.05 seconds still falls entirely within this scene"; the start-point lower bound takes the maximum of (end of the previous speech segment, scene start, current speech segment's start minus half the window length), with the start point randomly sampled; if the window's end crosses the scene's end point, the process is aborted; overlapping windows are likewise skipped.
【Design intent】(1) The lower-bound constraint of "no earlier than the end of the previous speech segment" avoids the window cutting into the previous utterance; (2) the lower-bound constraint of "no earlier than the nearest scene-cut point" encourages natural transitions; (3) the "minus half window length" constraint avoids the window's start point being too far from the target speech, which would leave the first half empty; (4) randomly sampling the start point brings data augmentation and positional diversity. Overall, this is a set of shot-aware + speech-aware dual-sensing segmentation, custom-built for the lip-sync task.

### Quality filtering (aesthetic scoring, sharpness, OCR text filtering, black-bar/watermark/logo detection)

Divided into two layers — "Stage 2 general three-dimensional quality assessment" and "Phase 2 pre-training secondary specialized filtering" — with all thresholds publicly disclosed (Appendix Table 9):
【Stage 2 audio quality】
- Silence Ratio < 0.8
- Bandwidth > 1,000 Hz
- Audiobox-aesthetics PQ (Production Quality) > 5.0
- Audiobox-aesthetics CU (Content Usefulness) > 4.5
- Audiobox-aesthetics CE (Content Enjoyment) > 2.5
【Stage 2 video quality】The DOVER video-quality-assessment tool is used, scoring from both technical and aesthetic perspectives:
- DOVER-Aesthetic (aesthetic score) > 0.85
- DOVER-Technical (technical score) > 0.05
【Threshold-determination method】Not arbitrary or borrowed from the literature, but rather "we manually inspect the videos retained under different metric cutoffs and set reasonable thresholds for each dimension accordingly," ensuring corpus quality while preserving sufficient diversity.
【Phase 2 secondary specialized filtering (three orthogonal filters)】
1) OCR subtitle filtering: OCR is used to detect burned-in subtitles, retaining subtitle-free videos, approximately 9.5M clips, and appending the sentence "This video has no subtitles." to the end of their prompts — note this is not simple exclusion, but rather treats "presence or absence of subtitles" as a controllable attribute explicitly taught to the model, a hybrid "filter + conditionalize" approach.
2) Lip-audio correspondence filtering: videos with LSE-D ≤ 9.5 and LSE-C ≥ 4.5 are retained, yielding a high-quality lip-audio correspondence subset of approximately 2.5M clips.
3) Visual fidelity filtering: DOVER technical score > 0.15, yielding approximately 2.4M clips.
Merged, the Phase 2 dataset totals 16.8M clips / ~37,600 hours.
【Phase 3】Uses only the highest-quality 720p subset, with DOVER technical score > 0.14 (this threshold is recalibrated for 720p).
【Black bars/watermarks/logos】Black bars are detected and cropped during preprocessing via FFmpeg cropdetect, then re-padded. Watermarks and logos are not specifically detected, but the visual-annotation prompt explicitly requires "Ignore all text, subtitle and watermark in the video," i.e., watermarks/subtitles are excluded from the description content at the annotation level, preventing the model from learning to treat watermarks as a describable element.

### Motion filtering (optical-flow/motion-score thresholds, static and jitter exclusion) ⚠️

The paper describes no independent motion-filtering stage at all: no optical-flow computation, no motion-score threshold, no static-shot exclusion or handheld-jitter exclusion. Motion-related quality control is implicit in two places: (1) the DOVER technical and aesthetic scores themselves include a perceptual assessment of temporal distortion, stutter, blur, and the like; (2) the "LAW OF VISUAL DYNAMICS" in the visual-annotation prompt requires the annotator to detect all transitions and precisely record motion trajectories, speed changes, and visual rhythm, and stipulates that "when there is no visual change, visual_description outputs null" — in theory this could be used to identify static clips, but the paper does not state whether this null signal is used for filtering. Since MOVA's training data is primarily dialogue between speakers, the magnitude of on-screen motion is naturally small, and the necessity of motion filtering is correspondingly lower. [Uncertain]

### Deduplication methods (exact deduplication and embedding-based semantic deduplication recorded separately) ⚠️

The paper does not mention deduplication at all: neither hash/fingerprint-level exact deduplication nor embedding-based semantic deduplication. Given that the data sources include multiple YouTube-derived public datasets such as ACAV-100M, VGGSound, and OpenVid-1M, as well as in-house YouTube-scraped content, the risk of cross-dataset duplication objectively exists, but the paper provides no explanation of this. [Uncertain]

### VLM/LLM as quality inspector (multimodal large-model quality scoring and mismatch removal — the 2026 trend of shifting from shallow scorers to large-model semantic judgment)

MOVA embodies the typical 2026 form of "deep large-model involvement in the data pipeline," but its division of labor is clearly delineated — **the large model is responsible for semantic annotation and cross-modal consistency adjudication, while dedicated small models are responsible for quality and alignment scoring**:
【The large model as semantic adjudicator (core usage)】In the caption-merging step, GPT-OSS-120B is responsible for more than just text concatenation — it performs an explicit cross-modal consistency check: "the model verifies the alignment between visual scenes and audio events to resolve potential conflicts," and then synthesizes everything into a unified natural-language description. This is essentially using a 120B-scale LLM as the judge of "whether audiovisual content is semantically self-consistent," making it the component in this pipeline that most closely resembles "LLM-as-judge."
【Anti-hallucination self-auditing design】Both the visual and speech annotation prompts embed a final_verification_audit self-check field (hallucination_check_passed, visual_changes_verified / speech_dynamics_verified, and comment), requiring the MLLM to output a structured self-audit conclusion; and a strong "LAW" system is used to suppress cross-modal cross-contamination — the visual annotator is explicitly instructed to "Ignore audio and inferred context entirely" and "Do not infer or hallucinate based on audio or context," while the speech transcriber is explicitly instructed to "Ignore non-speech sounds and music entirely." This is a defensive design against the most typical failure mode of multimodal annotators (inferring one modality based on another).
【Quality filtering is still handled by dedicated scorers】For audio, Audiobox-aesthetics (PQ/CU/CE three scores) is used; for video, DOVER (technical/aesthetic dual scores); for alignment, SynchFormer (temporal) and ImageBind (semantic); for audio classification, EAT; for lip-audio, the SyncNet-family LSE-D/LSE-C. All of these are dedicated discriminative models, not general-purpose VLMs.
【Conclusion】MOVA did not go the route of "using a VLM to produce a single comprehensive quality score replacing all shallow scorers," but instead adopts a hybrid division of labor — "dedicated scorers handle quality and alignment + a large model handles semantic annotation and consistency" — which can be seen as a pragmatic compromise within this trend.
【Limitation】The paper's Limitations section acknowledges that annotation reliability is a bottleneck in multi-speaker scenarios: speaker diarization errors and imperfect active-speaker labels propagate into training, causing the model to confuse speakers or learn inconsistent supervisory signals.

### Safety and compliance filtering (NSFW, copyright, face/privacy) ⚠️

The paper does not touch on safety and compliance filtering at all: there is no description of NSFW detection, no copyright filtering, no face/privacy protection measures, and no model-card-level safety statement or usage restrictions. This stands in contrast to its Apache-2.0 fully open commercial license, and is a clear gap in MOVA's disclosure system (the exact opposite of Sora 2's "thorough safety disclosure, blank data disclosure": MOVA is "thorough data-methodology disclosure, blank safety disclosure"). [Uncertain]

## Annotation Methods

### Captioning models used (self-developed VLM/open-source model, model scale)

Entirely open-source models are used, forming a three-model pipeline of "dual-modality division-of-labor annotation + large-model fusion," with no self-developed captioner:
【Visual annotation】MiMo-VL-7B-RL (Xiaomi LLM-Core team, a 7B vision-language model, RL version), whose instructions explicitly require a focus on video scene transitions.
【Audio annotation — dual-model strategy】
- Qwen3-Omni-Instruct: responsible for speech transcription (ASR), handling the speech component.
- Qwen3-Omni-Captioner: responsible for describing non-speech sounds and music, handling the non-speech component.
  The paper emphasizes that this dual-model division of labor "comprehensively covers linguistic content and acoustic characteristics, reducing information loss and capturing multifaceted audio semantics."
【Fusion and consistency verification】GPT-OSS-120B (OpenAI's open-source 120B model), responsible for merging the video caption with the aggregated audio annotation (including both speech and non-speech), while performing cross-modal consistency checking.
【Inference-side prompt-enhancement model (not for training-data annotation)】Qwen3-VL extracts structured visual descriptions, and Gemini 2.5 Pro rewrites them via in-context learning into video-generation prompts that match the training data distribution.
【Compute】Annotation uses a mix of NVIDIA GPU and Huawei Ascend NPU.
【Scale comparison】The visual annotator is only 7B, the audio annotator is from the Qwen3-Omni family, and the fusion model is 120B — placing the largest model at the fusion and adjudication stage rather than the perception stage is a choice made under cost-effectiveness trade-offs.

### Caption density and degree of structuring (short/long/dense descriptions, structured fields such as camera motion, style tags)

The caption structure is strictly structured JSON in its intermediate form, and a long, fused natural-language paragraph in its final form — a two-stage design of "structured intermediate representation → natural-language final caption." The full text of all prompts and complete examples are in Appendix A.5:
【Intermediate form: structured JSON】
- The visual side outputs video_visual_report, containing two fields: visual_description (a detailed description of how visual elements evolve; outputs null when there is no visual change) and on_screen_text (a precise transcription of all visible text; outputs null when there is no text); plus a final_verification_audit self-audit block.
- The speech side outputs speech_transcription_report, containing speech_description (verbatim transcription; outputs null when there is no speech; unclear portions marked with [inaudible]); plus a self-audit block.
- The non-speech side is free-text audio description (a minimal prompt: "Please describe the audio you hear in detail.").
【Three annotation laws (visual)】LAW OF VISUAL TRUTH (describe only visually verifiable elements; must not infer from audio or context), LAW OF VISUAL SILENCE (output null when there is no change), LAW OF VISUAL DYNAMICS (detect all transitions, precisely recording motion trajectories, speed changes, and visual rhythm).
【Final form: fused long caption】GPT-OSS-120B synthesizes a single fluent paragraph following five rules, whose structure still implicitly follows a sequential information hierarchy: first, visual narrative anchoring the speaker context (describing actions, objects, scenes, lighting, motion, and transitions in chronological order, without summarizing) → then embedding dialogue as quotations anchored to the visuals (using speech verbs that reflect tone, such as snaps/murmurs/laughs, and expressions reflecting pace, such as rushes/drawls/pauses) → finally introducing non-speech audio with phrases like "The audio includes…" or "In the background…" (limited to four categories: ambient sound, music, audio theme/sound source, and structural audio changes).
【Explicit style tags】No discrete tag system is used; style/camera-work information is embedded in natural-language description form (only the inference-side prompt rewriter has an explicit four-category structure: visual style including color tone and lighting, cinematography including shot composition, visual elements including subject and spatial relationships, and OCR text).
【Length】The complete example given in the paper is a dense, long caption of about 200 words; Figure 6b gives a statistical chart of prompt-length distribution (for the evaluation benchmark), but the length distribution of training captions is not quantified.
【Subtitle conditionalization】In Phase 2, videos without burned-in subtitles have "This video has no subtitles." appended to the end of their prompt, i.e., an explicit controllable-attribute marker exists in the caption.

### Joint audio-video caption structure (whether visual + auditory tracks are covered simultaneously, whether split into separate fields, e.g., LTX-2's full soundscape description, Script-a-Video's factorized streams, Foley-Omni's three-field structure)

MOVA adopts a "factorize into three independent tracks first, then merge into a single unified caption via LLM" scheme, which is a factorized-then-merged approach that falls between the Script-a-Video-style factorized streams and the LTX-2-style full soundscape description:
【Three independent tracks (factorization stage)】
1) Visual track (MiMo-VL-7B-RL): visual description + on-screen text, strictly forbidden from referencing audio.
2) Speech track (Qwen3-Omni-Instruct): verbatim dialogue transcription, strictly forbidden from including non-speech content and music.
3) Non-speech audio track (Qwen3-Omni-Captioner): descriptions of sound effects, music, ambient sound, recording quality, and speaker timbre and accent-related acoustic characteristics.
The three tracks are strictly isolated through mutually exclusive prompt constraints, preventing cross-modal hallucination contamination.
【Role reassignment in the fusion stage (a key design)】GPT-OSS-120B's merging rules explicitly define the "authoritative scope" of each of the three tracks:
- The **actual content** of dialogue strictly follows speech_description;
- **Dynamic speaker information** (total number of speakers, speaker switches, timbre characteristics such as high-pitched/hoarse) follows audio_description;
- The speaker's **visual anchoring** (who is speaking, where they are standing, what they are wearing) follows video_description.
  The three cross-reference each other, binding together three layers of information: who said what, what it sounded like, and who it looked like.
- Non-speech audio is only permitted to appear under four categories: ambient/background sound, music, audio theme/sound source, and structural audio changes (e.g., a shift from silence to a crescendo), and must never mention any human voice or words.
- Rule 4 explicitly requires avoiding cross-segment repetition (e.g., must not restate the speaker's visual position within the audio segment); Rule 5 requires skipping empty fields, so that the final result reads like a coherent narrative written by a human rather than a rigid structure.
【Final product】A single fused natural-language paragraph, with no separate fields retained. Thus, during training, the model sees a unified caption rather than separated visual/audio conditions.
【Comparison with comparable work】Compared to Foley-Omni's three-field parallel retention and LTX-2's single full-soundscape description, MOVA's distinguishing feature is "factorize at annotation time, fuse at training time," balancing annotation accuracy (factorization suppresses hallucination) with conditioning simplicity at training time (a single text condition).

### Dialogue transcription and speaker attribute annotation (ASR transcription + speaker identity/language/accent/emotion)

Dialogue transcription and speaker attribute annotation are the core of MOVA's data system:
【Transcription】Qwen3-Omni-Instruct performs ASR, outputting verbatim transcription. Three laws constrain it: LAW OF LANGUAGE FIDELITY (preserve the original language, translation strictly forbidden); LAW OF SPEECH DYNAMICS (create a new event entry whenever a new speaker / language / intonation begins); LAW OF SILENCE (output null for speech_description when there is no speech). Unclear segments are marked [inaudible].
【Speaker attributes】Covered in natural-language form via Qwen3-Omni-Captioner's audio description: number of speakers (e.g., "two distinct voices" / "a group of overlapping speakers"), speaker-switch points, timbre characteristics (e.g., high-pitched / gruff / mature male), accent (e.g., "a General American accent" in the example), tone and pace (calm/authoritative/professional; measured/declarative; rhythmic evenly paced cadence), and recording environment (studio-clean, slight reverb). These attributes are not enumerated fields but descriptive text, bound to specific figures in the frame during the fusion stage.
【Speaker identity binding】The fusion prompt requires speech to be embedded as quoted dialogue anchored to visual subjects (e.g., "the teenager in the corner," "the gray-haired woman"), achieving a three-way binding of "visual identity — timbre — dialogue content."
【Corresponding facility on the evaluation side】MOSS Transcribe Diarize (a transcription + speaker-diarization model released by the same team in 2026, arXiv:2601.01554) is used to perform speaker diarization (with explicit speaker labels [S01], [S02]) and ASR on generated results, from which cpCER (concatenated minimum permutation CER) is computed to evaluate whether speaker identity and dialogue content are correctly reflected. MOVA-720p achieves the best cpCER of 0.149.
【Known deficiency】The paper's Limitations section explicitly points out that diarization errors and imperfect active-speaker labels propagate into the training data, causing lip-audio mismatch and temporal drift in multi-speaker scenarios; improvement directions include stronger active speaker detection, cross-modal speaker tracking, and better noisy-clip filtering.

### Geometric and structured annotation (camera parameters, depth, 3D point tracks, action annotation, explicit state) ⚠️

The paper does not use any geometric or structured annotation: no camera parameters, no depth maps, no 3D point tracks, no skeleton/action annotation, no explicit physical-state annotation. The only annotation that comes close to structured is the on_screen_text field in the caption (precise transcription of on-screen text) and the four-category structured visual description extracted by the inference-side prompt rewriter (visual style/cinematography/visual elements/OCR text), but the latter serves prompt enhancement at inference time and is not part of the training-data annotation schema. Camera-work information appears only in natural-language form within the caption (e.g., "The camera alternates between…," "camera panning, zooming, and rotation"), with no parametric representation. [Uncertain]

### Synthetic data construction (controlled perturbation/editing to construct training pairs, e.g., InstructAV2AV) ⚠️

The paper constructs no synthetic training pairs whatsoever: no controlled perturbation, no edit-constructed paired data (such as InstructAV2AV-style edit pairs), and no self-generated videos from a generative model fed back into training. All training data is real video. The only "synthetic" components are: (1) the in-house TTS-synthesized speech data used in audio tower pretraining; (2) all captions generated automatically by an MLLM/LLM (synthetic annotation, not synthetic content); (3) the evaluation benchmark's prompts, uniformly rewritten and integrated by GPT-5, and the rewritten prompts generated by Gemini 2.5 Pro in the inference workflow. [Uncertain]

### Degree of human involvement (human annotation, human quality inspection, model pre-screening + human review)

The degree of human involvement is very low; the annotation process is fully automated, with humans appearing only in "threshold calibration" and "final evaluation":
【Threshold calibration (the only data-side human involvement)】"we manually inspect the videos retained under different metric cutoffs and set reasonable thresholds for each dimension accordingly" — researchers manually inspect videos retained under different metric cutoffs and, based on this, set thresholds for the three dimensions of audio quality, video quality, and audio-video alignment. The appendix reiterates that these thresholds are "determined by empirical observation." This is a typical "human calibration + machine batch execution" pattern.
【Annotation process】Zero human involvement. The three-track annotation for visual, speech, and non-speech, and the final fusion, are all performed by MiMo-VL / Qwen3-Omni ×2 / GPT-OSS-120B, with no description of human review or spot-checking.
【Heavier human investment in the evaluation process】The Arena-style human preference evaluation collected over 5,000 valid votes; the evaluation set totals 732 items (600 from Verse-Bench + 132 from the self-built benchmark), half of which — originally pure-English Verse-Bench speech data — was manually translated into Chinese to construct the bilingual mixed set; evaluators make pairwise preference judgments across five dimensions: prompt adherence, audio-visual synchronization, lip-sync accuracy, video quality, and audio-speech fidelity; ELO scoring is used (initial 1000, K=4, logistic scale 400, base 10, 1000 bootstrap iterations).

## Audio-Video Alignment

### Audio-video synchronization detection methods (lip sync, event alignment)

MOVA treats audio-video alignment as a first-class citizen of data filtering, using different tools at different stages and targeting different granularities:
【Stage 2 general alignment filtering (dual tools)】
- Temporal synchronization: SynchFormer computes the audio-video temporal synchrony (DeSync offset) for every video.
- Semantic alignment: ImageBind computes the cross-modal semantic alignment score (IB-Score) of the audio and video.
【Phase 2 lip-sync-specific filtering】SyncNet-family LSE-D (Lip Sync Error - Distance) and LSE-C (Lip Sync Error - Confidence) are used to select high-quality lip-audio correspondence clips.
【Audio-type routing】The EAT audio classification model distinguishes speech / non-speech, constructing separate subsets for the two respective target capabilities of "lip sync" and "general foley/ambient-sound modeling" — i.e., different capabilities correspond to different synchronization criteria.
【Architecture-level synchronization safeguards (not data-side, but strongly related)】Aligned RoPE maps video and audio latents onto the same temporal grid; the bidirectional cross-attention Bridge provides layer-by-layer cross-modal interaction; Dual Sigma Shift allows the two modalities to independently sample noise levels. In Discussion 7.1, the paper raises an insightful point: the predefined sigma schedule actually functions as an **implicit synchronization-direction prior** — for close-up shots of lips (where the target region occupies a large proportion of the frame), the visual latent carries relatively sufficient information, and the process tends toward Video→Audio; when the speaker occupies only a small portion of the frame, the visual evidence is relatively uncertain, and the process naturally leans toward Audio→Video, with speech providing a more reliable temporal anchor.
【The paper's core empirical conclusion】"architectural mechanisms alone (e.g., Bridge modules for cross-modal attention) are insufficient to achieve high-quality lip synchronization—the model must also learn phoneme-to-viseme mappings from data, which requires larger capacity and more training examples." That is, architectural mechanisms alone are insufficient for high-quality lip synchronization; the phoneme-to-viseme mapping must be learned from data, requiring greater capacity and more training samples. This is MOVA's most direct statement of the thesis that "data determines the ceiling of audio-video synchronization."

### Specific synchronization-detection metrics and thresholds (SyncNet/Synchformer/LSE/self-developed, threshold values, e.g., MOVA's LSE-D≤9.5 and LSE-C≥4.5, SkyReels-V4's SyncNet |offset|≤3∧conf>1.5)

All thresholds are publicly disclosed (Appendix Table 9 and Section 4.3), making this one of the most complete disclosures of synchronization-filtering thresholds found in this survey:
【Stage 2 audio-video alignment (relaxed OR gate)】
- ImageBind Score (semantic alignment) ≥ 0.2
- **OR** SynchFormer Offset (DeSync, temporal offset) ≤ 0.5
Note that this is a logical "OR," not "AND"; the paper explicitly explains the design intent: "This ensures that both semantically relevant ambient sounds and temporally synchronized speech/actions are preserved" — because ambient sounds naturally lack a sharp temporal onset and are hard to pass via temporal-synchronization detection, while fast-action sound effects may score low on semantic relevance; using AND would wrongly eliminate both.
【Phase 2 lip-sync-specific thresholds】
- LSE-D ≤ 9.5 (smaller distance is better)
- LSE-C ≥ 4.5 (higher confidence is better)
Both conditions must be satisfied simultaneously, yielding a high-quality lip-audio correspondence subset of approximately 2.5M clips. Compared with UniTalking's SyncNet conf > 0.9, MOVA's LSE-C ≥ 4.5 is a value on the original SyncNet confidence scale, representing a moderately strict threshold.
【Stage 2 audio-quality-related thresholds】Silence ratio < 0.8, bandwidth > 1,000 Hz, Audiobox PQ > 5.0 / CU > 4.5 / CE > 2.5.
【Stage 2 video-quality thresholds】DOVER-Aesthetic > 0.85, DOVER-Technical > 0.05; tightened to DOVER-Technical > 0.15 for Phase 2; DOVER-Technical > 0.14 for Phase 3 (720p).
【Same metric set reused on the evaluation side】DeSync (SynchFormer), IB-Score (ImageBind), and LSE-D/LSE-C (SyncNet) are likewise used in evaluation, with training filters and evaluation metrics highly homologous. MOVA-360p + dual CFG (s_B=3.5) achieves DeSync 0.351 / IB-Score 0.315 / LSE-D 7.004 / LSE-C 7.800.

### Separation of temporal synchronization vs. semantic synchronization (temporal alignment and content-semantic matching treated as two independent filtering conditions)

Clearly separated, and an explicitly argued design decision in MOVA's data filtering:
- Temporal synchronization is handled by SynchFormer (DeSync offset ≤ 0.5), characterizing "whether the sound and the on-screen event occur at the same moment."
- Semantic synchronization is handled by ImageBind (IB-Score ≥ 0.2), characterizing "whether the audio content and the visual content are semantically related."
- The two serve as two independent conditions, but are combined via a **relaxed logical OR gate**, not AND: "we apply a relaxed logical 'OR' gate between semantic and temporal alignment. A video is retained if it satisfies either IB-Score ≥ 0.2 OR DeSync ≤ 0.5." The paper's stated rationale is that the alignment nature of the two types of audio is fundamentally different — semantically relevant ambient/atmosphere sounds have no clear temporal onset, and requiring temporal synchronization would wrongly eliminate all of them; whereas fast-action sound effects may score low in the semantic embedding space, and requiring a semantic score would wrongly eliminate them. Using an OR gate preserves both types of valuable samples simultaneously.
- This separation is likewise maintained on the evaluation side: Table 4 splits AV-Align into two separately reported columns, DeSync (temporal) and IB-Score (semantic), and lists Lip Sync (LSE-D/LSE-C) separately as the finest-grained continuous temporal correspondence metric.
- In Section 6.3, the paper further distinguishes difficulty tiers between the two types of synchronization tasks: discrete, onset-driven events (such as "cutting fruit" or "hitting a drum") only need alignment at a few salient time points; whereas speech requires continuous, fine-grained correspondence between lip shape and phonemes over long time spans, making it the most demanding category.

### Audio quality filtering (SNR, silence detection and silence-ratio thresholds, no-audio-track exclusion, off-screen-sound-source exclusion, background-music separation)

Coverage is fairly comprehensive, with thresholds publicly disclosed:
【No-audio-track exclusion】The very first preprocessing step discards samples with "missing valid audio channels," as well as decoding-failure samples. Consequently, no silent videos exist in the training set.
【Silence ratio】Silence Ratio < 0.8 (i.e., clips with more than 80% silence are discarded). This threshold is fairly lenient, allowing a substantial amount of silence, but combined with the "speech-only" strategy, the actual silence ratio should end up far below this.
【Bandwidth】Bandwidth > 1,000 Hz, used to exclude audio tracks that are severely band-limited, encoding-degraded, or of telephone-line quality.
【Perceptual quality】Audiobox-aesthetics three-dimensional scoring: PQ (Production Quality, reflecting recording cleanliness and production standard) > 5.0, CU (Content Usefulness) > 4.5, CE (Content Enjoyment) > 2.5. The PQ threshold is set fairly high, effectively favoring studio-grade/professionally produced audio.
【SNR】No explicit signal-to-noise-ratio threshold is used; noise control is handled indirectly via Audiobox-PQ.
【Background music separation】No source separation is performed. Music, as part of the mix, is retained in the training audio track and described by Qwen3-Omni-Captioner in the caption (the paper's example caption includes a detailed description of background electronic music: synthesizer pads, a steady beat, a minor-key melody line, and a low mix level that does not interfere with narration).
【Off-screen sound/narration exclusion】No exclusion is performed, and the complete example given in the paper is itself an ad clip consisting of off-screen narration + no on-screen speaker, indicating that off-screen-sound samples are retained in the training set. This may be one source of the ambiguous active-speaker attribution problem in multi-speaker scenarios (a problem the paper's Limitations section acknowledges).
【Loudness normalization】Starting from Phase 2, LUFS loudness normalization is introduced, aimed at mitigating the loudness explosion caused by CFG — this is a training-data-side treatment, but its motivation comes from CFG behavior on the inference side.

### Classification and separate handling strategy for speech/sound effects/music

The classification tools and routing strategy are clearly defined, spanning the three levels of data, training, and capability:
【Classification tool】EAT (Efficient Audio Transformer, a self-supervised pretrained audio model) serves as the audio classifier, tagging clips with speech / non-speech-related labels. The construction condition for the speech subset is that both the EAT-contained-Speech and EAT-contained-Singing labels are judged True (or satisfy the model's positive-class confidence). Silero VAD additionally performs speech/non-speech interval division during preprocessing.
【Purpose of routing】The paper explicitly states that the routing is "depending on the target capability (e.g., lip synchronization vs. general foley/ambience modeling)" — i.e., separate subsets are constructed for the two target capabilities of "lip sync" and "general foley/ambient-sound modeling."
【Actual strategy: a two-stage division of labor】
- Music and sound-effect capability is injected during the 1.3B audio tower's pretraining stage: general sound effects come from WavCaps + VGGSound, music comes from JamendoMaxCaps, and speech comes from in-house TTS.
- Speech and lip-sync capability is reinforced during the joint training stage: joint training ultimately uses only speech clips (69.47% of preprocessed clips).
【Routing on the annotation side】Audio annotation is likewise divided by type — Qwen3-Omni-Instruct only performs speech transcription (strictly forbidden from including non-speech content and music), while Qwen3-Omni-Captioner only describes non-speech sound effects and music; during fusion, non-speech content is restricted to four categories (ambient sound, music, audio theme/sound source, structural audio changes) and strictly forbidden from including human voice or words.
【Cost】This "speech-heavy, music-light" mixture is explicitly acknowledged as a limitation: the model degrades in performance on singing, complex sound textures, and music/instrumental content, because the audio tower is only 1.3B and lacks the capacity to carry fine-grained pitch/harmonic structure and long-range temporal dependencies. The paper also points out that the model lacks physical-acoustic reasoning (e.g., the propagation delay between lightning and thunder is not explicitly modeled or enforced by the data).

## Training Coordination

### Multi-stage training curriculum and data curriculum scheduling (basis for stage division: resolution/duration/quality score/modality; low-res→high-res, image→video, short→long)

Two major stages, three joint-training phases, with the curriculum division simultaneously based on three axes: **modality, data quality, and resolution**:
【Stage A: Audio tower pretraining】The 1.3B text-to-audio DiT is trained separately (Wan2.1-1.3B backbone, 3D positional encoding replaced with 1D time-axis encoding); the data covers three domains — music, general sound effects, and TTS — using fixed-length clips, with each clip paired with a text prompt containing an explicit duration token to control the target length. The audio VAE is frozen.【Stage B: Progressive joint training (three phases)】The video tower is initialized from Wan2.2 A14B, the audio tower is initialized from Stage A, and the Bridge is randomly initialized; all three are jointly optimized end-to-end from the very first step (the paper notes that in early experiments, a two-stage approach of "first freeze both towers and train only the Bridge, then fine-tune everything" plateaued very early in performance, so the approach was changed to end-to-end).
- Phase 1 (360×640, diverse data, ~61,500 hours, 1 epoch, 15 days): asymmetric sigma shift (video 5.0, aggressive denoising / audio 1.0, gentle); aggressive text dropout p=0.5, forcing the model to rely on the Bridge to learn cross-modal alignment rather than taking a text shortcut. Goal: rapidly establish basic synchronization capability.
- Phase 2 (360×640, quality-filtered data, ~37,600 hours / 16.8M clips, 1 epoch, 7 days): audio sigma shift is aligned to 5.0 to strengthen audio denoising and improve timbre fidelity; text dropout is reduced to 0.2 to allow text-guided semantic refinement; LUFS loudness normalization is introduced. Goal: refine consistency and timbre under stabilized cross-modal attention.
- Phase 3 (720×1280, highest-quality subset, ~11,000 hours, 1 epoch, 20 days): with cross-modal alignment already stabilized, the model can safely allocate capacity toward higher resolution and finer spatial detail without disrupting the already-learned synchronization structure. CP increases from 8 to 16, batch size drops from 128 to 64, and the checkpoint interval shortens from 5000 steps to 2000 steps.
【Quantitative validation of curriculum effectiveness】Figure 9 gives the complete LSE-C / LSE-D curves over training steps (0→400K), color-coded by stage: in Stage 1, LSE-D drops rapidly and LSE-C rises (basic synchronization patterns are quickly learned); in Stage 2, LSE-D continues to drop and LSE-C shows a marked jump (consistency and confidence improve); in Stage 3, LSE-D drops further and plateaus while LSE-C stabilizes at a high level (convergence to high-quality lip synchronization). This is direct evidence that "progressively tightening the data curriculum stage by stage → monotonic improvement in synchronization metrics."
【Three axes of curriculum design】resolution (360p→360p→720p), data quality (diverse→quality-filtered→highest-quality), and noise schedule and text dropout (exploratory→refining) — the three co-evolve, rather than only resolution being adjusted.

### Change in data mixture across training stages (pretraining/annealing/SFT high-quality subset)

The mechanism by which the data mixture changes across stages is "decreasing scale + increasing quality threshold," with the specific figures fully disclosed:
【Phase 1 (exploration period)】~61,500 hours, diversified sources: SpeakerVid5M, Chinese drama, cartoons, movies, YouTube, OpenHumanVid. This stage pursues broad distributional coverage, with no additional quality narrowing.
【Phase 2 (quality-convergence period)】Building on the Phase 1 corpus, three orthogonal filters are layered on:
- OCR no-burned-in-subtitles: retains ~9.5M clips, with "This video has no subtitles." appended to their prompts (a filter + conditionalize hybrid strategy)
- Lip-audio correspondence: LSE-D ≤ 9.5 and LSE-C ≥ 4.5, retaining ~2.5M clips
- Visual fidelity: DOVER technical score > 0.15, retaining ~2.4M clips
Merged, the Phase 2 dataset = 16.8M clips ≈ 37,600 hours; the paper calls this "balancing scale and quality." This retains about 61% relative to Phase 1.
【Phase 3 (refinement period)】Uses only the highest-quality 720p subset, ~11,000 hours, DOVER technical score > 0.14 (recalibrated at the 720p scale). Only about 18% relative to Phase 1.
【No annealing/SFT stage】MOVA has no separate annealing or SFT stage; Phase 3's small, high-quality dataset effectively takes on the role of annealing/refinement.
【Accompanying hyperparameter mixture changes (Table 7)】Visual sigma shift remains constant at 5.0 across all three stages; audio sigma shift changes from 1.0 to 5.0 (starting in Phase 2); text dropout drops from 0.5 to 0.2 (starting in Phase 2); the audio loss weight remains constant at 0.2 across all three stages (i.e., video and audio velocity regression losses are weighted 1:0.2, with video dominant); weight decay remains constant at 0.001; backbone LR remains constant at 1e-5, Bridge LR remains constant at 2e-5; LUFS normalization is turned on starting from Phase 2.

### Post-training data (SFT curated-set scale and selection criteria, number of preference pairs and annotation method, reward-model training data) ⚠️

MOVA has no traditional post-training stage in the conventional sense: the paper describes no SFT curated set, no preference pairs, no DPO/RLHF, and no reward-model training data. Quality improvement is achieved entirely through data-tightening within the pretraining curriculum (the Phase 2/Phase 3 high-quality subsets), which can be viewed as "folding SFT into the tail end of the pretraining curriculum."
The two facilities closest to post-training both sit on the inference side or the evaluation side, rather than the training side:
(1) The inference-side prompt rewriter (Qwen3-VL extracts structured visual descriptions + Gemini 2.5 Pro rewrites via in-context learning into prompts matching the training distribution), used to bridge the gap between user input and the training-data distribution — this substitutes prompt engineering at inference time for post-training instruction alignment. The human Arena shows a significant ELO boost from the rewriter (MOVA-720p rises from 982.9 to 1025.3).
(2) The open-source codebase provides LoRA fine-tuning scripts, opening up post-training capability for the community to complete on their own.
[Uncertain]

### Data-processing infrastructure and throughput (NeMo Curator/Data-Juicer/self-developed, GPU acceleration ratio, processing scale, cost)

【Data-processing infrastructure】A self-developed video-preprocessing pipeline built on the Ray distributed framework, which the paper describes as "balancing data quality and processing efficiency." Neither NeMo Curator nor Data-Juicer nor any other off-the-shelf framework is used. The annotation stage uses a mix of NVIDIA GPU and Huawei Ascend NPU. The paper gives no figures for the throughput, GPU acceleration ratio, or cost of the data-processing stage.
【Training infrastructure】
- Scale: 1024 GPUs (128 nodes × 8 cards each), DP replicate size 64.
- Parallelism strategy: FSDP shards model parameters + USP sequence parallelism; 360p uses CP=8 (effective batch 128), 720p uses CP=16 (effective batch 64).
- MFU: approximately 35%.
- VAE redundancy elimination: borrowing from Wan's approach, within each CP group, input preprocessing (mainly VAE encoding) is executed only once, and the preprocessed features are then broadcast by a designated rank to the other ranks in the same group, avoiding redundant VAE computation caused by sequence parallelism.
- Memory management: manual memory management is used to sidestep the Python GC overhead problem reported by OpenSora2.
- MoE adaptation: because FSDP requires computational-graph consistency, the A14B MoE video tower uses alternating optimization — odd steps sample high-noise timesteps for all samples and optimize the high-noise DiT, even steps sample low-noise timesteps and optimize the low-noise DiT; the shared Bridge and the audio tower are optimized every step.
- Domestic hardware adaptation: the training stack has been ported to Ascend NPU, with operator fusion applied to the attention kernel, tensor-layout transformations, and rotary positional-encoding computation to reduce framework overhead. An 8-card Ascend 910A2 micro-benchmark (CP=4, DP-shard=2): 34.1 seconds per step, 376 TFLOPs at FP16, approximately 40GB of per-card memory usage, and ≥128GB of host memory. The paper cautions that this figure depends on the software-stack version and should not be extrapolated to large-scale training cost.
- Total cost: 42 days × 1024 GPUs ≈ 43,000 GPU-days.
【Sequence-length bottleneck】A 720p, 8-second, 24fps clip produces approximately 1.6×10^5 tokens; the paper lists this as the primary bottleneck for training throughput and inference latency (especially under the most common guidance setting NFE=3), and proposes future directions: more aggressive spatiotemporal compression, hierarchical or chunked generation, and system-level optimization for long-context video tokens.

## Performance Comparison

### Quantitative impact of data-strategy ablations (distinguishing: filtering-strictness ablation / caption-density-style ablation / data-mixture ablation, and corresponding evaluation metrics) ⚠️

The paper's ablation experiments mainly target **training strategy and inference configuration**, and lack strict "data-strategy ablations" (no filtering-strictness control group, no caption-density/style control group, no data-mixture control group). The existing quantitative evidence is as follows:
【Resolution/stage ablation (Table 4) — indirectly reflects the effect of the data curriculum】MOVA-360p vs. MOVA-720p: DeSync 0.475→0.485, IB-Score 0.286→0.277 (essentially no degradation); LSE-C 6.278→6.593 (improvement); cpCER 0.177→0.149 (improvement); IS 4.269→3.936, DNSMOS 3.797→3.671 (slight decline on the audio side, which the paper explains as a normal trade-off resulting from model capacity tilting toward visual complexity). This validates the effectiveness of the data curriculum of "first establish alignment at low resolution, then refine at high resolution."
【Training-progression ablation (Figure 9) — the most direct evidence of the data-scale effect】LSE-C / LSE-D show a staged, monotonic improvement over training steps (0→400K), with the improvement inflection points coinciding with the boundaries of the three data stages. The paper draws from this its core conclusion: architectural mechanisms (Bridge cross-attention) alone are insufficient for high-quality lip synchronization; the phoneme-to-viseme mapping must be learned from data, which requires greater model capacity and more training samples. This is direct evidence that "data scale drives synchronization capability."
【Dual CFG scale ablation (Table 5) — inference side】As s_B increases from 1.0 to 4.0: LSE-C peaks at 7.891, DeSync reaches a minimum of 0.365, and IB-Score and LSE-D improve in step; but the cost is a decline in DNSMOS and cpCER rising from 0.177 to 0.264. The paper calls this "conditional interference" and "over-regularization" — an excessively high s_B causes the model to prioritize satisfying the geometric constraint of synchronization, sacrificing the fidelity of the speech generation itself and instruction adherence (what is said, and how naturally it is said).
【T2VA emergent-capability ablation (Table 6)】Replacing the reference image with an empty placeholder: IS 4.269→4.370 (improvement), DeSync 0.475→0.441 (improvement), IB-Score 0.286→0.281 (essentially unchanged), LSE-C 6.278→5.830 and LSE-D 8.098→8.362 (decline, because the empty placeholder provides no lip geometry). This indicates that the model can explore the joint audio-visual manifold more freely when lacking visual structural constraints.
【Subjective Arena ablation (Figure 7/8)】MOVA-720p's ELO is 1113.8, significantly higher than LTX-2 (1074.1), Ovi (925.4), and WAN2.1+MMAudio (886.9); its win rate against Ovi and cascaded systems both exceed 70%, and its win rate against LTX-2 is 51.5%. The internal Arena shows that the prompt rewriter is the single most influential factor affecting human preference (720p rises from 982.9 → 1025.3), while dual CFG (s_B=3.5), although significantly improving objective alignment metrics, causes human preference to drop slightly from 1025.3 to 1014.5 — the paper attributes this to the fact that amplifying cross-modal guidance relatively weakens the guiding weight of the text instruction, occasionally leading to reduced instruction adherence. This constitutes a valuable observation: **objective synchronization metrics and overall human preference are not always aligned in the same direction**.
[Uncertain]

### Evidence for quality vs. quantity (cases where small, refined data surpasses large, messy data) ⚠️

The paper does not conduct a "small and refined vs. large and messy" controlled experiment, but its entire training-curriculum design is itself an engineering embodiment of this idea, and it carries quantitative traces:
- Data scale shrinks substantially at each successive stage while performance continues to improve: 61,500 hours (Phase 1) → 37,600 hours (Phase 2, about 61%) → 11,000 hours (Phase 3, about 18%), while LSE-C/LSE-D improve monotonically across the three stages (Figure 9), with Phase 3's LSE-D dropping further and plateauing.
- The overall cleaning retention rate is only 26.39% (Table 1), of which the "speech clips only" condition alone eliminates about 26 percentage points — an aggressive trade-off that actively sacrifices over 40% of data diversity in service of a target capability (lip sync).
- Among Phase 2's three filters, the lip-audio correspondence subset is only ~2.5M clips and the DOVER technical-score subset only ~2.4M clips, substantially narrower than the OCR no-subtitle subset's ~9.5M; the paper explicitly describes the final 16.8M-clip dataset as "balancing scale and quality."
- Time allocation likewise reflects the emphasis placed on high-quality data: Phase 3 uses only 11,000 hours of data yet trains for 20 days (the longest of the three stages), far exceeding Phase 2's 7 days for 37,600 hours.
Note that all of this is indirect evidence from design intent and training curves, without a controlled experiment showing that "large, messy data performs worse under equal compute." [Uncertain]

### The alignment relationship between the training-data domain distribution and the evaluation-benchmark taxonomy (e.g., VABench's seven categories) ⚠️

There is a clear correspondence between the enumeration of training-data domains and the taxonomy of the self-built evaluation benchmark, but the paper does not explicitly argue for this alignment, nor does it provide a mixture mapping:
【The self-built benchmark's six scenario categories】
1) Multi-speaker scenarios (evaluating synchronized speech, facial expressions, and interaction across multiple characters)
2) Movie video (requiring cinematic-level narrative generation, with plot referencing the original film's background)
3) Sports competition (focused on athlete performance, with some prompts including commentator narration)
4) Game livestream video (covering shooter, 3D, and competitive game genres)
5) Camera-motion sequences (evaluating visual realism under camera movements such as pan, zoom, and rotation)
6) Anime-style video (including 2D anime and 3D animation)
Benchmark construction method: first frames and corresponding prompts are extracted from real videos; prompts briefly describe the scene setting, characters, and environmental conditions, incorporating audio-related information as needed for the scene, forming unified prompts for joint audio-visual generation.
【Correspondence with the training-data domains】The domains enumerated for training data (movies, animation/cartoons, sports, education, news, interviews, Chinese drama, vlogs) overlap heavily with movies, anime, sports, and multi-speaker (corresponding to interviews/drama) in the benchmark's six categories; but the two categories "game livestream" and "camera-motion sequences" have no clear corresponding source in the training-data description, representing a portion of the benchmark's coverage that extends beyond what is explicitly listed for the training data.
【Another evaluation benchmark】Verse-Bench (600 sets of image-text prompt pairs), used after GPT-5 unifies the visual and audio descriptions into a single prompt; the paper criticizes it as "not specifically designed for multi-scenario audio-visual generation evaluation," which is precisely the motivation for the self-built benchmark.
【No corresponding domain-mixture control on the training side】The paper does not describe any targeted mixture adjustment or supplementation of training data according to the benchmark's categories, so the "alignment" between the two is more a natural coincidence than a deliberate design. [Uncertain]

## Other Information

### summary_note

Core conclusion: MOVA is the sample with the **highest reproducibility of data-processing methodology** found in this survey, and it is also the most thoroughly disclosed technical report currently available in the open-source audio-video joint generation direction. Its unique value is concentrated in four points:
(1) **A complete quantitative funnel**: Table 1 gives the stage-by-stage duration retention rate (100% → 84.57% → 58.75% → 26.39%), Table 9 gives the specific thresholds for all 8 filtering metrics, and the clip output volumes of Phase 2's three sub-filters (9.5M / 2.5M / 2.4M → 16.8M) are also fully disclosed. This is extremely rare against the backdrop of closed-source models (Sora 2, Veo 3, Kling, Seedance) disclosing essentially nothing.
(2) **Dual speech-aware + shot-aware segmentation**: jointly driving window sampling from Silero VAD's speech boundaries and PySceneDetect's scene-cut points, with two complete pseudocode blocks given (Appendix A.3), explicitly distinguishing single-shot/multi-shot samples while ensuring that utterances are not truncated. This is a segmentation paradigm custom-built for the lip-sync task and can be directly reused.
(3) **OR-gate separation of temporal synchronization and semantic synchronization**: SynchFormer DeSync ≤ 0.5 and ImageBind IB-Score ≥ 0.2 are combined via logical "OR," with the paper explicitly arguing that using AND would wrongly eliminate both "ambient sounds with no onset" and "fast-action sound effects with low semantic scores." This is the clearest articulation of this dimension found under this topic.
(4) **A joint AV caption schema with factorized annotation + LLM fusion**: three mutually exclusive tracks (MiMo-VL visual / Qwen3-Omni-Instruct speech transcription / Qwen3-Omni-Captioner non-speech audio) paired with strongly constrained prompts to suppress cross-modal hallucination, then adjudicated for consistency and merged by GPT-OSS-120B; the merging rules explicitly define the authority boundaries — "dialogue content follows the transcription, speaker dynamics follow the audio description, visual anchoring follows the video description." The full text of all prompts and complete caption examples are disclosed (Appendix A.5), and can be directly replicated.
The most aggressive data decision: **the final joint training retains only speech clips** (69.47% of preprocessed clips), actively sacrificing data coverage of sound effects/music/ambient sound in service of lip-sync capability, at the self-acknowledged cost of degraded performance on singing, music, and complex sound textures — a clear case study of the trade-off between "capability focus vs. coverage breadth."
Major information gaps: data compliance and provenance (zero disclosure), safety filtering (zero disclosure), deduplication (zero disclosure), motion filtering (zero disclosure), geometric structured annotation (not used), post-training data (no such stage), and strict data-strategy ablations (no control experiments for filtering strictness/caption style/data mixture were conducted). In addition, no figures are given for the training data's domain mixture, single/multi-shot ratio, or language ratio.
The methodologically most quotable conclusion: architectural mechanisms alone are insufficient for high-quality lip synchronization; the phoneme-to-viseme mapping must be learned from data, requiring greater capacity and more training samples — the curve in Figure 9 showing LSE-C/LSE-D improving monotonically across the three-stage data curriculum is the direct support for this thesis.

## Uncertain Fields

The research information for the following fields is partially uncertain (sources marked with ⚠️):

- provenance_licensing
- domain_distribution
- language_accent_distribution
- motion_filtering
- deduplication
- safety_filtering
- geometric_structured_annotation
- synthetic_data_synthesis
- post_training_data
- data_ablation
- quality_over_quantity_evidence
- benchmark_taxonomy_alignment
