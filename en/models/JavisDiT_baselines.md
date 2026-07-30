# Audio-Video Joint Generation Baselines Collection (Combined Survey of 4 Works):
(1) JavisDiT / JavisDiT++ — "JavisDiT: Joint Audio-Video Diffusion Transformer with Hierarchical Spatio-Temporal Prior Synchronization" (arXiv:2503.23377) and its follow-up "JavisDiT++: Unified Modeling and Optimization for Joint Audio-Video Generation" (arXiv:2602.19163, ICLR 2026), together with the evaluation benchmarks JavisBench / JavisBench-mini and the synchronization metric JavisScore;
(2) MM-Diffusion — "MM-Diffusion: Learning Multi-Modal Diffusion Models for Joint Audio and Video Generation" (arXiv:2212.09478, CVPR 2023), the pioneering work in joint generation, with its self-built Landscape dataset;
(3) AV-DiT — "AV-DiT: Efficient Audio-Visual Diffusion Transformer for Joint Audio and Video Generation" (arXiv:2406.07686);
(4) Harmony — "Harmony: Harmonizing Audio and Video Generation through Cross-Task Synergy" (arXiv:2511.21579), with the evaluation benchmark Harmony-Bench;
(5) UniAVGen — "UniAVGen: Unified Audio and Video Generation with Asymmetric Cross-Modal Interactions" (arXiv:2511.03334).
Among these, (2) and (3) are early small-scale academic baselines, (1) is a mid-period open-source academic baseline, and (4) and (5) are recent strong baselines from late 2025.

> Topic: Data processing for video generation models (including simultaneous audio-video generation): data cleaning pipeline, data distribution, annotation methods

[← Back to Home](../index.md)

**Table of Contents**: [Basic Information](#basic-information) · [Data Scale and Distribution](#data-scale-and-distribution) · [Cleaning Pipeline](#cleaning-pipeline) · [Annotation Approach](#annotation-approach) · [Audio-Video Alignment](#audio-video-alignment) · [Training Coordination](#training-coordination) · [Results Comparison](#results-comparison)

## Basic Information

### Name

Audio-Video Joint Generation Baselines Collection (Combined Survey of 4 Works):
(1) JavisDiT / JavisDiT++ — "JavisDiT: Joint Audio-Video Diffusion Transformer with Hierarchical Spatio-Temporal Prior Synchronization" (arXiv:2503.23377) and its follow-up "JavisDiT++: Unified Modeling and Optimization for Joint Audio-Video Generation" (arXiv:2602.19163, ICLR 2026), together with the evaluation benchmarks JavisBench / JavisBench-mini and the synchronization metric JavisScore;
(2) MM-Diffusion — "MM-Diffusion: Learning Multi-Modal Diffusion Models for Joint Audio and Video Generation" (arXiv:2212.09478, CVPR 2023), the pioneering work in joint generation, with its self-built Landscape dataset;
(3) AV-DiT — "AV-DiT: Efficient Audio-Visual Diffusion Transformer for Joint Audio and Video Generation" (arXiv:2406.07686);
(4) Harmony — "Harmony: Harmonizing Audio and Video Generation through Cross-Task Synergy" (arXiv:2511.21579), with the evaluation benchmark Harmony-Bench;
(5) UniAVGen — "UniAVGen: Unified Audio and Video Generation with Asymmetric Cross-Modal Interactions" (arXiv:2511.03334).
Among these, (2) and (3) are early small-scale academic baselines, (1) is a mid-period open-source academic baseline, and (4) and (5) are recent strong baselines from late 2025.

### Publishing Institutions/Companies

(1) JavisDiT / JavisDiT++: Led by the National University of Singapore (NUS) — Hao Fei, Shengqiong Wu, Tat-Seng Chua, Wei Li, and others — jointly with Xiamen University (Jiayi Ji), Fudan University (Fan Zhou), University of Rochester (Jiebo Luo), Nanyang Technological University (Ziwei Liu), and others; first author Kai Liu; the community organization is named JavisVerse.
(2) MM-Diffusion: A joint effort by Renmin University of China (Ludan Ruan, Qin Jin) and Microsoft Research Asia (MSRA) (Huan Yang, Bei Liu, Jianlong Fu, Nicholas Jing Yuan, Baining Guo), with additional participation from Peking University; the GitHub organization is researchmm (the Microsoft Research multimedia group).
(3) AV-DiT: A joint effort by the University of Toronto (Kai Wang, Dimitrios Hatzinakos), UT Dallas (Shijian Deng, Yapeng Tian), and Adobe Research (Jing Shi).
(4) Harmony: A joint effort by Shanghai Jiao Tong University (Teng Hu, Ran Yi) and Tencent Hunyuan (Zhentao Yu, Guozhen Zhang, Zhengguang Zhou, Youliang Zhang, Yuan Zhou, Qinglin Lu).
(5) UniAVGen: Led by Nanjing University (State Key Laboratory for Novel Software Technology, Guozhen Zhang, Limin Wang) and Tencent Hunyuan (Zixiang Zhou, Yi Chen, Yuan Zhou, Qinglin Lu), jointly with Shanghai Jiao Tong University (Teng Hu), Renmin University of China (Ziqiao Peng), Tsinghua University (Youliang Zhang), and Shanghai AI Laboratory.
Note: The Harmony and UniAVGen author lists overlap heavily (Guozhen Zhang, Teng Hu, Youliang Zhang, Yuan Zhou, and Qinglin Lu all appear in both papers), and the two can be regarded as sister works from the same Tencent Hunyuan research line.

### Publication Date (technical report/paper/open-source release)

(2) MM-Diffusion: Submitted to arXiv (v1) on December 19, 2022, revised in March 2023, accepted at CVPR 2023; it is the earliest work in this collection.
(3) AV-DiT: Made public on arXiv on June 11, 2024 (arXiv:2406.07686).
(1) JavisDiT: First released on arXiv (v1) on March 30, 2025, revised on February 22, 2026, and accepted at ICLR 2026; the follow-up JavisDiT++ was released in February 2026 (arXiv:2602.19163), also at ICLR 2026.
(5) UniAVGen: Submitted to arXiv on November 5, 2025 (arXiv:2511.03334), revised on March 24, 2026.
(4) Harmony: Submitted to arXiv on November 26, 2025 (arXiv:2511.21579), revised on November 28, 2025.

### Type (model/dataset/toolchain/evaluation benchmark)

All five are "models," with three additionally shipping evaluation benchmarks or dataset artifacts:
- JavisDiT/JavisDiT++: Model + evaluation benchmark (JavisBench, 10,140 entries; JavisBench-mini, 1,000 entries) + synchronization evaluation metric (JavisScore) + open-source training/inference toolchain; the only work in this collection that delivers "model + benchmark + metric + complete training code" simultaneously.
- MM-Diffusion: Model + self-built dataset (Landscape, 1,000 ten-second natural-scene audio-video clips) + open-source code and pretrained weights.
- AV-DiT: A pure model (a parameter-efficient joint generation architecture), with no new dataset and no new benchmark.
- Harmony: Model + evaluation benchmark (Harmony-Bench, 150 entries, split into three tiers of subsets).
- UniAVGen: Model (a unified framework supporting joint generation, video-to-audio dubbing, audio-driven video animation, and other multi-task capabilities), with no new dataset.

### Degree of Openness (whether weights/code/data/pipeline are each open-sourced) ⚠️

The degree of openness varies enormously and can be divided into three tiers:
[Fully Open Tier]
- JavisDiT / JavisDiT++: GitHub JavisVerse/JavisDiT, with weights (HuggingFace JavisVerse/JavisDiT-v1.0-jav), inference code, complete three-stage training scripts, and evaluation tools (meta files and precomputed caches for 16 audio-video metrics) all publicly available; the JavisBench and JavisBench-mini benchmark data are public. The data side is half-open: the Stage 1 audio pretraining data has been preprocessed and released on HuggingFace (JavisData-Audio); the Stage 2 video data comes from TAVGBench, and the repository provides a list of 330,000 video IDs, but explicitly states that "the original YouTube videos cannot be released due to copyright issues"; the Stage 3 DPO preference data is "being prepared for release." This is the work with the highest data transparency in the collection.
- MM-Diffusion: MIT licensed; GitHub researchmm/MM-Diffusion openly provides all code, training scripts, evaluation pipeline, and 6 checkpoints (Landscape.pt / Landscape_SR.pt / AIST++.pt / AIST++_SR.pt / the guided-diffusion upsampling initialization model / the i3d and AudioCLIP evaluation models); the Landscape and AIST++_crop preprocessed data are provided directly for download via Google Drive and Baidu Netdisk — the only work here that also releases the training data itself.
[Promised-Openness Tier]
- AV-DiT: The paper states that "source code and pretrained models will be released," but no confirmed public repository was found as of the survey [uncertain]. The two datasets used (AIST++, Landscape) are themselves public.
[Closed Tier]
- Harmony: The paper provides no code/weights repository link; whether Harmony-Bench is public is not stated [uncertain]; the 2 million self-collected environmental-sound audio-video clips used in training are not open-sourced.
- UniAVGen: The paper provides no openness information [uncertain], and the "internally collected real human audio-video dataset" used for training is entirely closed-source.
Note: Both Harmony and UniAVGen build on open-source model foundations (Harmony uses Wan2.2-5B, MMAudio's audio VAE, F5-TTS's speech encoder, and the umT5 text encoder; JavisDiT++ uses Wan2.1-1.3B-T2V; JavisDiT v1 uses Open-Sora's video VAE and AudioLDM2's audio components), following a pattern of "building atop open-source foundations while remaining closed/half-open themselves."

### Whether Simultaneous Audio-Video Generation Is Supported, and How (native joint / cascaded / MoE fusion)

All five support simultaneous (joint) audio-video generation, and all fall under "native joint" generation rather than cascading, but the cross-modal interaction mechanisms differ, forming a clear technical evolution line:
(1) MM-Diffusion (2022): A dual-branch sequential multi-modal U-Net, with 2D+1D spatio-temporal convolutions for video and dilated convolutions for audio; the core is random-shift based attention (randomly shifted attention blocks) that bridges the two subnetworks, reducing the cross-modal attention complexity from O((F×H×W)×T) to O((S×H×W)×(S×T/F)). Pixel-space diffusion, not latent. It can also zero-shot transfer to conditional video-to-audio and audio-to-video generation (via gradient guidance) without additional training.
(2) AV-DiT (2024): Focused on "parameter efficiency" — a single DiT backbone pretrained only on image data (frozen) is shared, with only lightweight adapters trainable in the audio and video branches; the video branch inserts trainable temporal attention into the frozen DiT blocks to ensure temporal consistency, and the audio branch is similarly adapted with lightweight parameters, plus a cross-modal feature interaction module. This is a "frozen shared backbone + dual-modality adapters" paradigm for joint generation.
(3) JavisDiT (2025): A dual-tower DiT (the video tower derives from Open-Sora, the audio tower from AudioLDM2, with both VAEs frozen); the core innovation is the HiST-Sypo Estimator (Hierarchical Spatio-Temporal Synchronized Prior Estimator) — which first estimates a set of "coarse-grained global priors + fine-grained spatio-temporal priors" from the text prompt, and then uses that prior to jointly guide the denoising of both audio and video, achieving fine-grained spatio-temporal alignment; cross-modal interaction relies on cross-attention and bidirectional attention modules.
(4) JavisDiT++ (2026): Switches to Wan2.1-1.3B-T2V as the base, with three upgrades — Modality-Specific Mixture-of-Experts (MS-MoE, which improves single-modality quality while preserving cross-modal interaction), Temporal-Aligned RoPE (TA-RoPE, which achieves explicit frame-level synchronization between audio tokens and video tokens, an idea sharing the same lineage as Ovi's scaled-RoPE), and Audio-Video Direct Preference Optimization (AV-DPO).
(5) UniAVGen (2025): A dual-branch joint synthesis architecture, with two parallel DiTs handling audio and video respectively; the core is "Asymmetric Cross-Modal Interactions" — the information flow between the two modalities is unequal — paired with a Face-Aware Modulation module and Modality-Aware Classifier-Free Guidance; a single framework uniformly supports 5 task categories including joint generation, video-to-audio dubbing, and audio-driven video animation.
(6) Harmony (2025): The video branch is initialized from Wan2.2-5B, and the audio side uses MMAudio's VAE encoder + F5-TTS's speech encoder. Three innovations: cross-task collaborative training (Cross-Task Synergy, using two bidirectional generation tasks — "audio-driven video" and "video-driven audio" — to suppress alignment drift during joint denoising), a Global-Local Decoupled Interaction module (GLDI, where the global branch handles style alignment and the local branch handles temporal precision), and Synchronization-Enhanced Classifier-Free Guidance (which amplifies the alignment signal at inference time). The authors explicitly identify three pain points of the joint diffusion paradigm: alignment instability under concurrent noise evolution, the low efficiency of attention mechanisms for temporal precision, and the lack of cross-modal synchronization guidance in standard CFG.

### List of Research Sources (URLs of papers/technical reports/official documentation/news, with each source's nature noted: primary official / same-team corroboration / third-party report)

1) Primary official | JavisDiT paper https://arxiv.org/abs/2503.23377 (ICLR 2026, includes JavisBench/JavisScore construction and three-stage training data scale)
2) Primary official | JavisDiT++ paper https://arxiv.org/abs/2602.19163 and HTML version https://arxiv.org/html/2602.19163v1 (data filtering strategy, TAVGBench 1.1M→355K funnel, AV-DPO preference data construction, Appendix D.2 data quality ablation)
3) Primary official | JavisDiT GitHub https://github.com/JavisVerse/JavisDiT and https://github.com/JavisDiT/JavisDiT (scope of openness, data copyright statement)
4) Primary official | JavisDiT data preparation documentation https://raw.githubusercontent.com/JavisDiT/JavisDiT/main/assets/docs/data.md (three-stage data CSV schema, 30-second audio truncation, 16kHz resampling, video 16fps normalization, minimum-10-frame filtering, DPO 1+3 candidate construction)
5) Primary official | JavisDiT++ project homepage https://javisverse.github.io/JavisDiT2-page/
6) Primary official | OpenReview review page https://openreview.net/forum?id=hRRWfFpKRp
7) Primary official | MM-Diffusion paper https://arxiv.org/abs/2212.09478 and CVPR 2023 version https://openaccess.thecvf.com/content/CVPR2023/html/Ruan_MM-Diffusion_Learning_Multi-Modal_Diffusion_Models_for_Joint_Audio_and_Video_CVPR_2023_paper.html
8) Primary official | MM-Diffusion GitHub https://github.com/researchmm/MM-Diffusion (MIT license, data download, dual-scale 64×64 and 256×256 SR)
9) Primary official | AV-DiT paper https://arxiv.org/abs/2406.07686 and HTML version https://arxiv.org/html/2406.07686v1 (16 frames at 256×256, 1.6-second 16kHz audio, mel spectrogram 40×16×8)
10) Primary official | AV-DiT OpenReview https://openreview.net/forum?id=FE6zflN5G5
11) Primary official | Harmony paper https://arxiv.org/abs/2511.21579 and HTML version https://arxiv.org/html/2511.21579v1 (composition of 4 million clips, Gemini automatic annotation, three-stage training, Harmony-Bench)
12) Primary official | UniAVGen paper https://arxiv.org/abs/2511.03334 and HTML version https://arxiv.org/html/2511.03334v1 (Emilia English subset, internal real human audio-video data, 1.3M vs 30.1M comparison, three-stage training hyperparameters)
13) Same-team corroboration | HuggingFace Papers pages https://huggingface.co/papers/2602.19163 and https://huggingface.co/papers/2406.07686
14) Third-party corroboration | MM-LDM paper https://arxiv.org/pdf/2410.01594, MMDisCo https://arxiv.org/pdf/2405.17842, UniForm https://arxiv.org/pdf/2502.03897 (all restate the Landscape/AIST++ statistics, allowing cross-validation)

## Data Scale and Distribution

### Training Data Volume (video count/hours/token count, pretraining and SFT separated) ⚠️

Data scale spans four orders of magnitude across these works, making this the dimension that best illustrates the "academic baseline → industrial baseline" evolution in this collection:
[MM-Diffusion (2022) — tens of thousands of frames, hour-scale]
- Landscape: 928 source YouTube videos → cut into 1,000 non-overlapping 10-second clips, totaling about 2.7 hours and roughly 300,000 frames.
- AIST++: 1,020 street-dance video clips, totaling 5.2 hours and roughly 560,000 frames, paired with 60 copyright-cleared dance songs.
- The two datasets together total under 8 hours, a typical "small-scale, single-domain, high-fidelity" academic setting.
[AV-DiT (2024) — reuses the same two datasets] Scale is identical to MM-Diffusion (AIST++ 1,020 clips/5.2 hours, Landscape 1,000 clips/2.7 hours), with no new data introduced; trained for 100,000 iterations, batch size 16.
[JavisDiT (2025) — million-scale entries]
- Stage 1 audio pretraining: 780,000 (0.78M) audio-text pairs, trained for 55 epochs (recorded as 50 epochs in JavisDiT++).
- Stage 2 ST-Prior training: 610,000 (0.6M) synchronized "text-video-audio" triplets, plus synthesized asynchronous negative samples.
- Stage 3 JAVG training: the same 610,000 triplets, fine-tuning the cross-attention and bidirectional attention modules.
- Ablation experiments use a 60,000 (60K) subset for rapid evaluation on JavisBench-mini.
[JavisDiT++ (2026) — roughly 1 million entries of public data]
- Audio: 780,000 audio-text pairs (reusing JavisDiT's set), 50 epochs, training an audio FFN with 794M parameters in total.
- Video: the original 1.1 million triplets of TAVGBench → filtered down to 355,000, of which 330,000 are used for audio-video SFT (2 epochs, LoRA with 121M parameters) and 25,000 for AV-DPO (1 epoch, LoRA with 121M parameters).
- The authors emphasize that all data are "public training entries," reaching SOTA at a scale of roughly 1 million entries — representative of "small data driving a large model."
[Harmony (2025) — 4 million+ clips]
- A total of "over 4 million audio-video clips," covering both human speech and environmental sound.
- Human speech side: aggregated from Emilia, OpenHumanVid, and SpeakerVid, then filtered by an audio-video consistency scoring model down to 2 million high-quality clips, each 3–10 seconds.
- Environmental sound side: AudioCaps (about 128 hours, manually annotated) + Clotho (about 31 hours, manually annotated) + WavCaps (about 7,600 hours, automatically annotated) + 2 million self-collected audio-video clips rich in environmental sound.
- Stage 1 audio pretraining: 100,000 iterations, global batch size 1536; Stage 2: 20,000 iterations; Stage 3 cross-task joint training: 10,000 iterations, batch size 128.
[UniAVGen (2025) — 1.3 million samples, focused on "high efficiency with less data"]
- The core selling point of the paper is achieving overall superiority in audio-video synchronization, timbre consistency, and emotion consistency using "1.3M training samples vs. the comparison method's 30.1M" (this comparison figure is drawn from Ovi's joint training sample count).
- Stage 1: the English subset of the Emilia multilingual audio dataset, 160,000 steps (160k steps), batch size 256, lr 2e-5.
- Stage 2: internally collected real human audio-video data, 30,000 steps, batch size 32, lr 5e-6.
- Stage 3: multi-task learning, 10,000 steps, with a 4:1:1:2:2 ratio across five task categories.
- Whether the 1.3M figure refers to the internal dataset's entry count or the cumulative sample count across stages is not clarified by the paper [uncertain].

### Composition of Data Sources (proprietary/public datasets/web scraping/licensed procurement/synthetic data) ⚠️

Shows a three-stage evolution from "self-built small dataset → assembled public datasets → public + internal mix":
[MM-Diffusion] (1) Self-built Landscape: the authors themselves scraped 928 natural-scene videos from YouTube and then split them, the only dataset in this collection scraped from scratch and publicly released; (2) Public dataset AIST++: a subset of the AIST dataset, street-dance videos paired with 60 copyright-cleared dance songs, naturally avoiding copyright risk. The zero-shot transfer experiments additionally involve AudioSet-type data [uncertain].
[AV-DiT] Pure reuse of public datasets: AIST++ and Landscape, with no proprietary data, no scraping, no procurement.
[JavisDiT / JavisDiT++] Almost entirely an assembly of public academic datasets, which is key to its reproducibility:
- The audio side (780,000 entries) comes from 10 public collections: AudioSet, AudioCaps, VGGSound, WavCaps, Clotho, ESC50, GTZAN, MACS, UrbanSound8K, MusicInstrument — covering general sound effects, music, and speech.
- The video side comes from TAVGBench (a public benchmark of 1.1 million text-audio-video triplets, underpinned by YouTube videos).
- The preference (DPO) data is a mix of model self-generated and ground-truth data, i.e., synthetic/self-bootstrapped data.
- The JavisBench evaluation set is a mixed collection from "test sets of existing datasets (Landscape / AIST++ / FAVDBench) + YouTube videos uploaded between June and December 2024."
[Harmony] A public + self-collected mix:
- Public: Emilia (a TTS-specific speech corpus), OpenHumanVid (human videos), SpeakerVid (two-person interactive human generation data), AudioCaps, Clotho, WavCaps.
- Self-collected: an additional 2 million audio-video clips rich in environmental sound (the paper calls these "newly collected," with the source channel undisclosed) [uncertain].
[UniAVGen] Public + internal mix, with internal data at the core:
- Public: the English subset of the Emilia multilingual audio dataset (used only for Stage 1 audio pretraining).
- Internal: an "internally collected real human audio-video dataset," which carries the entirety of Stage 2 and Stage 3 training; its source, scale, and collection method are entirely undisclosed [uncertain]. Given the Tencent Hunyuan background, it is speculated to be related to their internal video/livestream corpora, but there is no textual basis for this.

### Data Compliance and Provenance (proportion of licensed data, rights-cleared datasets, C2PA, etc.) ⚠️

Overall disclosure is sparse, but MM-Diffusion and JavisDiT show clear actions here, making them the rare bright spots in this collection:
[MM-Diffusion] (1) One explicit reason for choosing AIST++ is that its 60 dance songs are "copyright-cleared songs" — i.e., music copyright was considered at the data-selection stage; (2) the self-built Landscape was scraped from YouTube without discussion of video copyright, but the dataset itself was publicly released for research purposes; (3) code and weights are released under the MIT license.
[JavisDiT / JavisDiT++] The clearest copyright awareness: (1) the repository explicitly states that "the original YouTube videos cannot be released due to copyright issues," providing only 330,000 video IDs for users to download themselves — a standard academic practice for avoiding video copyright; (2) content from YouTube in JavisBench underwent "strict manual legal and ethical verification"; (3) the paper emphasizes that all training data are public dataset entries (public training entries), with no procurement and no internal proprietary data, giving a relatively clear compliance chain.
[AV-DiT] No discussion of data licensing or compliance [uncertain]; the two datasets used are both already-published academic datasets.
[Harmony] No discussion of the proportion of licensed data, rights-cleared datasets, C2PA, or any other provenance mechanism; the licensing status of the 2 million self-collected clips is entirely undisclosed [uncertain]. The paper only vaguely states that the environmental-sound data comes from "public sources."
[UniAVGen] No discussion whatsoever of data compliance, licensing, or privacy (note that its internal data is "real human" audio-video, involving faces and voiceprints with high privacy sensitivity, yet the paper contains no related statement) [uncertain].
Common gap: none of the five mentions C2PA, content watermarking, synthetic-content labeling, data-subject deletion requests, or other modern provenance mechanisms.

### Clip Duration Distribution and Segmentation Strategy ⚠️

The duration strategies of each work strongly reflect differences in compute budget and goals, with a shared characteristic of being "short, fixed-length, with no multi-duration bucketing":
[MM-Diffusion] Source videos are cut into 10-second non-overlapping clips as the dataset unit, but the actual training samples are shorter: both Landscape and AIST++ are sampled at a fixed number of frames, with the corresponding audio clip length on the order of about 1.6 seconds [uncertain: the exact training-clip duration in seconds stated in the original paper].
[AV-DiT] The shortest: each training sample is 16 frames of video plus audio waveform truncated or padded to 1.6 seconds (16kHz). Video and audio durations correspond strictly, with no bucketing.
[JavisDiT / JavisDiT++] Training and evaluation are uniformly 4-second clips (240P / 24fps); the paper also runs an extended test at 10 seconds. All JavisDiT++ JavisBench evaluation is likewise fixed at "240P, 4 seconds." Hard constraints on the data preparation side: audio is uniformly truncated to under 30 seconds before splitting, video must have at least 10 frames or it is discarded, and fps is uniformly normalized to 16Hz.
[Harmony] Human speech clips are explicitly 3–10 seconds (this is one of the few cases with a clear range rather than a fixed length); the maximum clip length for Stage 1 audio pretraining is 10 seconds, with reference audio being a randomly extracted 1–3 second clip.
[UniAVGen] Video is uniformly processed at 16 fps before VAE encoding; the specific clip duration in seconds is not clearly given [uncertain].
Common limitation: all five works' training samples are within 10 seconds, none involve minute-scale long videos, multi-shot stitching, or duration bucketing/scheduling — this is the most fundamental gap between these academic baselines and industrial models (such as Veo/Sora 2/LTX-2).

### Resolution/Aspect-Ratio Distribution and Bucketing Strategy ⚠️

All five works' resolutions are far below industrial models, and all use fixed resolution with no aspect-ratio bucketing:
[MM-Diffusion] The base diffusion process runs at 64×64 pixel space, then a separate super-resolution (SR) model upsamples to 256×256; the repository provides both base models (Landscape.pt / AIST++.pt) and their corresponding SR models (Landscape_SR.pt / AIST++_SR.pt). This "low-resolution generation + separate SR" two-stage approach was mainstream in 2022. Aspect ratio is fixed at 1:1, with no bucketing.
[AV-DiT] Video frames are cropped to 256×256 resolution, with a video latent size of 32×32×4 and an audio latent (mel spectrogram) size of 40×16×8. Single resolution, single aspect ratio.
[JavisDiT / JavisDiT++] Training and evaluation are uniformly 240P, 24fps (fps normalized to 16Hz during data preparation). Although the data CSV retains fields such as height, width, aspect_ratio, and resolution (inheriting Open-Sora's data management schema, which in theory supports multi-bucketing), all experiments reported in the paper use a fixed 240P/4-second configuration, and no actual use of bucketing strategy is observed [uncertain].
[Harmony] The paper does not specify video resolution or frame rate [uncertain]; the Wan2.2-5B base natively supports 720P, so the model likely inherits this resolution capability, but this is not confirmed in the text.
[UniAVGen] Only states that video is processed at 16 fps before VAE encoding; resolution and aspect ratio are undisclosed [uncertain].
Common trait: none adopts area normalization (such as Ovi's fixed total pixel count of 518,400), and none adopts multi-aspect-ratio bucketing/scheduling, indicating that these works concentrate their compute on cross-modal alignment mechanisms rather than visual fidelity.

### Category/Domain Distribution and Ratio Control Strategy (proportion control and concept balancing across people, actions, scenes, styles, etc.) ⚠️

Domain coverage ranges from "single-domain" to "dual-domain ratio" to "general-purpose," making this one of the most clearly evolving dimensions in this collection:
[MM-Diffusion — single-domain with clearly stated categories] Landscape covers 9 natural-scene categories, fully disclosed: explosion, fire cracking, raining, splashing water, squishing water, thunder, underwater burbling, waterfall burbling, and wind noise — all natural phenomena with strong causal binding between visual events and sound, deliberately chosen to validate audio-visual alignment. AIST++ is a single street-dance domain (human dance movement ↔ musical beat). The two datasets respectively represent "environmental sound alignment" and "rhythm/motion alignment" as two types of synchronization, and this "complementary dual-dataset" setup was fully carried forward by later works such as AV-DiT as a standard evaluation configuration.
[AV-DiT] Fully reuses the two domains above, with no expansion.
[JavisDiT / JavisDiT++ — explicitly constructed a category taxonomy] One of its greatest contributions is making the domain-distribution problem explicit: JavisBench establishes a taxonomy of 5 evaluation dimensions and 19 scene categories in total, the five dimensions being: Event Scenario, Video Style, Sound Type, Spatial Composition, and Temporal Composition. The authors emphasize that "over 50% of the videos belong to highly complex and challenging scenarios" and "75% of the samples contain multiple sounding events" — a direct critique and extension beyond the "single sound source, single event" setting of earlier baselines. On the training side, domain breadth comes from TAVGBench's general YouTube distribution, but the category proportions within the training data itself are not disclosed [uncertain].
[Harmony — an explicit 1:1 dual-domain ratio] Explicitly divides data into two domains, "human speech" and "environmental sound," and strictly maintains a 1:1 mixing ratio in both Stage 1 and Stage 3 training. Harmony-Bench is likewise split into three tiers along this line: environmental sound-video, speech-video, and complex scenes (environmental sound + speech co-occurrence), forming a direct correspondence between "training ratio ↔ evaluation category." This is the only work in this collection to give an explicit domain-ratio figure.
[UniAVGen — single-domain focus on real humans] Training data is "real human audio-video," with the domain highly concentrated on scenes of people speaking/performing, paired with the Face-Aware Modulation face-perception module, a deliberately narrow-domain design; the evaluation test set of 100 entries is split "50% real images / 50% AIGC and anime-style images," indicating explicit ratio control along the image-style dimension (real vs. anime-style, evenly split), though no fine-grained breakdown is given at the content-domain level [uncertain].
[Common gap] Aside from Harmony's 1:1 and UniAVGen's 50/50, none discloses fine-grained proportion control or concept-balancing strategies for people/action/scene/style [uncertain].

### Audio Category Distribution and Ratio (proportions and control strategy for speech/foley/music/ambient sound/silence) — a dimension unique to AV models ⚠️

This dimension shows extreme divergence across the five works, forming a full spectrum from "sound-effect focused → speech focused → both balanced":
[MM-Diffusion / AV-DiT — pure ambient sound and music, zero speech] Landscape is entirely natural ambient sound (rain, thunder, wind, fire, water), and AIST++ is entirely music (60 dance songs). Neither dataset contains human speech or dialogue, so these two early baselines have no lip-sync or TTS capability whatsoever — this is the most fundamental capability gap between them and post-2025 models.
[JavisDiT — audio pretraining covers all categories, while speech is deliberately excluded at the video stage] The audio side's 780,000 entries come from 10 datasets, and JavisDiT++ explicitly states that it "does not adopt any data filtering strategy, in order to maximize text-to-audio generation capability, covering three categories: general sound, music, and speech." That is, the audio pretraining stage deliberately maintains full category coverage with zero filtering. But at the audio-video SFT stage, the strategy reverses: "the FunASR detection tool is used to remove most videos containing human speech." This is a very clear category-ratio decision — the JavisDiT series intentionally forgoes lip-sync/dialogue generation capability, concentrating the model's capacity on event-level alignment of ambient sound and sound effects, thereby avoiding the highly difficult problem of lip-shape modeling. "Sound Type" being one of the five evaluation dimensions in JavisBench's taxonomy shows that the evaluation side explicitly stratifies audio categories.
[Harmony — a strict 1:1 speech/ambient-sound ratio] The clearest ratio strategy: both Stage 1 (audio pretraining) and Stage 3 (cross-task joint training) adopt a "human speech dataset : ambient sound dataset = 1:1 mix." The speech side has 2 million entries (Emilia + OpenHumanVid + SpeakerVid, filtered by consistency scoring), and the ambient sound side is AudioCaps + Clotho + WavCaps + 2 million self-collected entries. On the evaluation end, Harmony-Bench further isolates a "complex scene with speech + ambient sound co-occurrence" tier into a separate 50-entry subset — directly targeting the hardest real-world case where speech and ambient sound coexist. Harmony can be said to be the work in this collection with the most systematic thinking about audio category distribution.
[UniAVGen — speech as the absolute focus] Stage 1 performs pure audio pretraining on the English subset of Emilia (TTS corpus), and Stages 2 and 3 use real human audio-video data, with the entire pipeline centered on human voice/dialogue; the evaluation metrics (SyncNet, Whisper WER, timbre consistency, emotion consistency) are also all centered on speech, with no report of ambient sound or music capability [uncertain].
[Silence handling] None of the five describes a silence-detection threshold or silence-proportion control [uncertain].

### Narrative Structure Distribution (single-shot vs. multi-shot, average clip duration, shot-count distribution, whether native audio track is included) ⚠️

All use the simplest setting of "single-shot, short clip, with native audio track," with none involving multi-shot narrative:
[Shot count] Training samples across all five works are single-shot clips. MM-Diffusion mechanically cuts 928 source videos into 1,000 non-overlapping 10-second clips (no mention of shot detection, so cut points could cross shot boundaries — a known rough spot in early data processing); JavisDiT relies on TAVGBench's already-existing clip division; Harmony's 3–10 second clips and UniAVGen's real-human clips are likewise single-shot.
[Average clip duration] MM-Diffusion's dataset unit is 10 seconds, with the actual training clip shorter; AV-DiT is 16 frames + 1.6-second audio; JavisDiT/JavisDiT++ is fixed at 4 seconds; Harmony is 3–10 seconds (a range distribution, the only non-fixed-length case in this collection); UniAVGen is undisclosed [uncertain].
[Native audio track] 100% of the five works' training data contains a native, synchronized audio track — a precondition for joint audio-video generation. MM-Diffusion's AIST++ is a special case: its music is the accompaniment paired with the dance rather than an on-location recording, i.e., "aligned at production time" rather than "synchronously captured at the time of filming."
[Narrative capability] None of the five works involves shot transitions, multi-shot consistency, or story structure design; Harmony-Bench's "complex scene" subset is the closest attempt to narrative complexity, but the complexity lies in the acoustic layer of "speech and ambient sound co-occurring" rather than the shot-narrative layer.
[Gap vs. industrial models] Compared to models like Veo 3 / Sora 2 / LTX-2 that have begun handling multi-shot and minute-scale narrative, this entire collection remains at the "single-shot, ≤10 seconds" stage — both a result of limited academic compute and a sign that these baselines' data-processing experience is hard to transfer directly to long-video scenarios.

### Language/Accent Distribution (data foundation for multilingual lip-sync capability) ⚠️

Disclosure is extremely sparse, with an overall tendency toward "English-only":
[MM-Diffusion / AV-DiT] Training data contains no human speech (natural ambient sound and instrumental music), so the language/accent dimension is not applicable (N/A).
[JavisDiT / JavisDiT++] Although the audio pretraining data includes speech-related datasets (AudioSet and WavCaps contain speech components), the language distribution is not disclosed [uncertain]; the audio-video SFT stage actively removes videos containing speech using FunASR, so the model essentially lacks language-related lip-sync capability, and this dimension does not really constitute a bottleneck for it.
[UniAVGen] Explicitly uses only Emilia's "English subset" of its multilingual audio dataset for audio pretraining — i.e., deliberately forgoing multilingual support to focus on English; evaluation uses the GRID benchmark (English corpus) and Whisper WER, again validating English only. The language composition of the internal real-human audio-video data is undisclosed [uncertain].
[Harmony] Uses Emilia (itself a multilingual TTS corpus) but does not state whether it is restricted to the English subset [uncertain]; the paper mentions that Harmony-Bench's speech subset is used to examine "lip-sync fidelity and multilingual speech robustness," making it the only work in this collection to explicitly claim multilingual coverage — but the specific language list, per-language proportions, and per-language sync metrics are not given [uncertain].
[Common gap] None of the five annotates accent attributes, performs language balancing, or reports per-language synchronization metrics — a stark contrast to industrial models (such as Vidu, Kling, etc.) that emphasize multilingual lip-sync.

## Cleaning Pipeline

### Overall Filtering Funnel Structure (number of filtering stages, order) ⚠️

The complexity of the filtering funnels across the five works ranges from "almost none" to "three-stage filtering + large-model annotation":

[MM-Diffusion (simplest)] Scrape → split → no explicit filtering. 928 natural-scene videos are scraped from YouTube, mechanically split into 1,000 non-overlapping 10-second clips, and organized into 9 scene categories. The paper describes no quality filtering, deduplication, or synchronization-detection step. AIST++ is taken directly from the existing dataset (the authors additionally performed cropping, named AIST++_crop in the repository). This reflects the 2022-era stage of joint-generation research where "make the task work first, refine data quality rather than quantity."

[AV-DiT (no self-built pipeline)] Directly reuses two public datasets, with only formatting preprocessing: video sampled to 16 frames and cropped to 256×256, audio truncated or padded to 1.6 seconds @16kHz and converted to mel spectrogram (40×16×8). No filtering funnel.

[JavisDiT / JavisDiT++ (the most complete and transparent funnel in this collection)] Two independent pipelines, one for training and one for evaluation:
《Training side — data preparation for each of the three stages》(fully public in the GitHub data.md)
- Stage 1 (audio pretraining, 780K entries): audio folder → generate metadata CSV → extract audio statistics → truncate to under 30 seconds → uniformly resample to 16kHz → add placeholder dummy video reference → output train_audio.csv. Explicitly states "no data filtering strategy is adopted" to maximize T2A capability.
- Stage 2 (audio-video SFT): TAVGBench's 1.1 million triplets → use FunASR detection to remove most videos containing human speech → apply the Open-Sora method for three-part filtering: aesthetic scoring, optical flow/motion scoring, and OCR text scoring → remove corrupted videos, filter out samples with fewer than 10 frames → normalize fps uniformly to 16Hz → align with TAVGBench's captions → merge multi-source CSVs → output train_av_sft.csv (330K entries).
- Stage 3 (AV-DPO): from a pool of 30,000 prompts that does not overlap with SFT data, use a reference model to generate N=3 audio-video candidates per prompt → extract metadata and audio for the generated samples → pair with 1 ground-truth entry to form a "1 ground truth + 3 generated" group of 4 candidates → use multiple reward models to do modality-aware scoring → normalized modality-aware ranking to select winning/losing pairs → output train_av_dpo.csv (about 25K preference pairs).
《Evaluation side — JavisBench construction》: about 30,000 candidate sounding videos are collected from the test sets of existing datasets (Landscape / AIST++ / FAVDBench) and YouTube videos uploaded between June and December 2024 → pre-filtering (removing low-quality/noisy candidates) → Qwen-family models automatically generate captions and classify by the 19-category taxonomy → post-filtering (to ensure diversity based on content) → strict manual legal and ethical verification → final 10,140 entries; an additional random subset of 1,000 entries forms JavisBench-mini.

[Harmony] The funnel structure is relatively simple, but introduces model-based quality control: on the speech side, data is aggregated from Emilia + OpenHumanVid + SpeakerVid → filtered using an "audio-visual consistency scoring model" → yielding 2 million high-quality clips (3–10 seconds); on the environmental sound side, AudioCaps + Clotho + WavCaps + 2 million self-collected entries are aggregated → all data is automatically annotated by Google Gemini (ASR transcription + video description caption + background sound description) → entered into training at a 1:1 mix.

[UniAVGen] Describes no data cleaning pipeline whatsoever [uncertain], only stating that Stage 1 uses the Emilia English subset and Stages 2–3 use internal real human audio-video data, plus two formatting steps: audio sampled at 24kHz and converted to mel spectrogram, and video processed at 16fps before VAE encoding. This is the work with the least data disclosure in this collection.

### Quantified Funnel Retention Rates (input/output volume and final retention rate at each filtering stage, e.g., Apollo's 27%) ⚠️

Only JavisDiT++ provides computable funnel retention rates in this collection; all others are missing:
[JavisDiT++ (the only work with a quantified funnel)]
- Overall video-side retention rate: TAVGBench's original 1.1 million triplets → filtered to 355,000, an overall retention rate of about 32.3% (355K/1.1M). This figure is on the same order of magnitude as publicly reported funnels of comparable scope in the industry (such as Apollo's 27%), giving it cross-comparison value.
- Downstream allocation of the retained data: 330,000 entries used for audio-video SFT (93% of the retained volume), 25,000 entries for AV-DPO (7%), with the two strictly non-overlapping.
- Stage-by-stage retention rates are missing: the paper gives only the total, without breaking down how many entries were eliminated by "FunASR speech removal," "aesthetic scoring," "optical flow scoring," or "OCR scoring" individually, nor the specific thresholds for each [uncertain]. Given that TAVGBench is underpinned by YouTube video with a high proportion of speech content, it can be inferred that the FunASR step is likely the largest elimination stage, but there is no data to support this.
- Audio-side retention rate is 100% (explicitly "no data filtering strategy is adopted").
- Internal proportion in the DPO stage: of the final roughly 25,000 preference pairs, about 30% of the winning samples come from model generation (rather than ground truth), from which the authors infer that "the baseline model itself already possesses fairly strong generation capability."
[JavisBench evaluation set funnel] About 30,000 candidates → pre-filtering for quality + Qwen automatic labeling/classification + post-filtering for diversity + manual legal-ethical review → 10,140 entries, a retention rate of about 33.8% (10140/30000), coincidentally close to the training-side funnel's retention rate. The elimination amounts at each stage are not broken down [uncertain].
[Harmony] The speech side filters a pooled aggregate of Emilia + OpenHumanVid + SpeakerVid down to 2 million entries via consistency scoring, but the original size of the pool is not given, so the retention rate cannot be calculated [uncertain].
[MM-Diffusion / AV-DiT / UniAVGen] No quantified funnel data whatsoever [uncertain]. MM-Diffusion's 928 source videos → 1,000 clips is a split, not a filter, and does not constitute a retention rate.

### Shot Segmentation Method (PySceneDetect / in-house model / shot-aware splitting) ⚠️

Shot segmentation is broadly weak across this collection, one of the areas showing the clearest gap from industrial-grade pipelines:
[MM-Diffusion] Explicitly uses "mechanical equal-length splitting" rather than shot-aware splitting: 928 source videos are "split into 1,000 non-overlapping 10-second clips," without using PySceneDetect, TransNetV2, or any scene-detection tool. This means the resulting clips may cross shot boundaries and contain transition frames — a known rough spot in early data processing. The authors additionally performed cropping on AIST++ (named AIST++_crop in the repository), which is spatial cropping rather than temporal splitting.
[AV-DiT] No splitting performed; frames are sampled directly (16 frames) from already-split dataset clips.
[JavisDiT / JavisDiT++] Relies on the upstream clip division already completed by TAVGBench, with no shot-detection step in its own pipeline; the video processing steps in data.md only include "removing corrupted videos, filtering samples with fewer than 10 frames, normalizing fps to 16Hz" — no scene detection [uncertain: whether TAVGBench itself performed shot detection upstream].
[Harmony] Whether the 3–10 second clip division is based on shot detection is not stated [uncertain]; the segmentation method for the 2 million self-collected environmental-sound clips is likewise undisclosed.
[UniAVGen] Describes no segmentation method whatsoever [uncertain].
[Impact] The lack of shot-aware splitting means training data may contain clips with transitions mixed in, which could theoretically harm temporal consistency; but since all works' clips are extremely short (1.6–10 seconds), the cross-shot risk is relatively contained. This also explains why none of these models has multi-shot generation capability.

### Quality Filtering (aesthetic scoring, sharpness, OCR text filtering, letterbox/watermark/logo detection) ⚠️

Only JavisDiT++ has systematic visual quality filtering; the rest are largely blank:
[JavisDiT++ (the only one in this collection)] Explicitly "follows the Open-Sora method" for three-part quality filtering:
(1) Aesthetic scoring — removing clips of poor visual quality;
(2) Optical flow/motion scoring — see motion_filtering;
(3) OCR scoring — detecting and removing clips with excessive on-screen text (subtitles, danmaku, title cards, etc.), avoiding the model learning to generate garbled text.
The specific thresholds for all three are undisclosed [uncertain], and the elimination amount for each is not broken down [uncertain].
Additionally, data.md includes two engineering-level cleanup steps: removing corrupted videos, and filtering out samples with fewer than 10 frames.
[Unmentioned common techniques] Even JavisDiT++ does not describe watermark detection, logo detection, letterbox detection and removal, compression-artifact detection, blur detection, or brightness/contrast filtering [uncertain].
[MM-Diffusion] Mentions that the self-built Landscape dataset is "high-fidelity," but describes no automated quality-filtering pipeline; the small scale of 1,000 entries suggests possible manual curation, though the paper does not state this [uncertain].
[AV-DiT] No quality filtering; public datasets are used directly.
[Harmony] Visual quality filtering is not mentioned [uncertain]. The only filtering is the "audio-visual consistency scoring model," which is a cross-modal alignment filter rather than a visual quality filter.
[UniAVGen] Not mentioned at all [uncertain].
[JavisBench evaluation set] Mentions a two-stage approach — "pre-filtering to ensure quality, removing noisy candidates" and "post-filtering to ensure diversity" — but the specific tools and thresholds used are undisclosed [uncertain].

### Motion Filtering (optical flow/motion score threshold, removing static or shaky footage) ⚠️

[JavisDiT++ (the only work to explicitly use this)] Adopts "flow scoring" as one of three quality metrics for TAVGBench data filtering, following the Open-Sora implementation (Open-Sora typically uses UniMatch or RAFT to compute optical-flow magnitude as a motion score). The purpose is to remove overly static clips. The specific threshold, whether it also removes the excessive-motion/shaky end of the spectrum, and whether the motion score is used as a training condition or sampling weight are all unstated by the paper [uncertain].
[MM-Diffusion] No motion filtering. Notably, its dataset selection itself implicitly constrains motion: Landscape's 9 scene categories (explosion, fire, rain, splashing water, thunder, waterfall, wind, etc.) are all natural phenomena with sustained dynamics, and AIST++ is entirely dance movement — i.e., motion sufficiency is guaranteed through domain selection rather than a filter, a "selection in place of filtering" mindset typical of the small-dataset era.
[AV-DiT] No motion filtering.
[Harmony / UniAVGen] Neither mentions any motion filtering or optical-flow scoring [uncertain].
[Common trait] Aside from JavisDiT++, none of the works uses an optical-flow tool, which relates to the small data scale in this collection where quality can be manually controlled; once data scale reaches the million level (as with JavisDiT++'s 1.1 million TAVGBench entries), motion filtering becomes a necessity.

### Deduplication Method (exact deduplication and embedding-based semantic deduplication recorded separately) ⚠️

None of the five works in this collection describes any deduplication step [uncertain], whether exact deduplication (hashing/pHash/frame fingerprinting) or embedding-based semantic deduplication (CLIP/video-embedding clustering) — this is a uniformly blank dimension.
Indirect circumstances for each:
- MM-Diffusion: Landscape is explicitly split into "non-overlapped" 10-second clips, which is clip-level overlap avoidance rather than cross-video deduplication; with only 928 source videos, the duplication risk is inherently low.
- AV-DiT: Uses ready-made public datasets, with deduplication responsibility resting with the upstream dataset authors.
- JavisDiT / JavisDiT++: The only relevant design is that "the 30,000-entry prompt pool used for DPO is strictly non-overlapping with the SFT training data (apart from the SFT training data)," and the 330K SFT and 25K DPO samples do not overlap with each other — this is an isolation at the train/eval split level, not data deduplication. Whether TAVGBench performed deduplication upstream is unknown [uncertain]. The "post-filtering to ensure diversity" in JavisBench construction is functionally close to semantic deduplication, but the authors do not use the term deduplication nor state what method was used to measure diversity [uncertain].
- Harmony: The 4 million entries are assembled from multiple public datasets (AudioCaps, Clotho, WavCaps have known audio overlaps with each other, e.g., WavCaps intersects with AudioSet), but the paper does not mention cross-dataset deduplication [uncertain] — a potential duplication risk point.
- UniAVGen: No description whatsoever [uncertain].

### VLM/LLM as Quality Judge (multimodal large-model quality scoring and mismatch removal — the 2026 trend of shifting from shallow scorers to large-model semantic judgment) ⚠️

This dimension happens to fully illustrate a three-generation evolution across this collection, from "no model judge → dedicated discriminative model → large-model semantic judge":
[Generation 1: No model judge (MM-Diffusion 2022 / AV-DiT 2024)] The data side does not use any model for quality control or scoring, relying instead on dataset selection and small-scale manual quality control. Models are only used at evaluation time (i3d for FVD, AudioCLIP for FAD), not for data filtering.
[Generation 2: Dedicated discriminative models as filters (JavisDiT++ 2026)] Uses a set of shallow/dedicated models as filters and scorers, consistent with mainstream 2024–2025 practice:
- FunASR (Alibaba's open-source speech recognition toolkit) as a "speech-presence detector," used to remove videos containing human speech — a clever use of an ASR model as a binary-classification filter.
- An aesthetic predictor model as the quality judge.
- An optical-flow model as the motion judge.
- An OCR model as the text-contamination judge.
[Generation 2.5: Multi-reward-model ensemble as preference judge (JavisDiT++'s AV-DPO, the most valuable practice in this collection)] Uses six models each responsible for a scoring dimension to construct preference pairs, a typical case of "model-as-judge" used for data construction rather than filtering:
- Audio quality → AudioBox (AudioBox-Aesthetics)
- Text-audio alignment → ImageBind
- Video quality → VideoAlign
- Text-video alignment → ImageBind
- Audio-video cross-modal similarity → ImageBind
- Temporal synchronization → Synchformer
and adopts "normalized modality-aware ranking" to select winning/losing pairs, with the authors explicitly stating this is to "ensure internal consistency within each modality, avoiding pairing high-quality audio with low-quality video" — a key engineering lesson for constructing preference data under multi-dimensional rewards.
[Generation 3: Multimodal large models as annotator/judge (Harmony 2025 and JavisDiT's JavisBench)]
- Harmony: uses Google Gemini to automatically annotate all 4 million clips (ASR transcription + video description + background sound description), and uses an "audio-visual consistency scoring model" to filter the speech data — the latter being a direct practice of handing cross-modal consistency judgment to a model, though the model's identity, scoring dimensions, and thresholds are all undisclosed [uncertain].
- JavisDiT's JavisBench: uses "advanced Qwen-family models" to simultaneously perform caption generation and 19-category scene classification, i.e., the MLLM acts as both annotator and classification judge.
[UniAVGen] No mention of any model-based data judge whatsoever [uncertain].

### Safety and Compliance Filtering (NSFW, copyright, faces/privacy) ⚠️

Overall disclosure is extremely sparse, with only JavisDiT showing a clear action:
[JavisDiT / JavisDiT++ (the only one)] During JavisBench construction, all content sourced from YouTube underwent "strict manual legal and ethical verification" — the only clearly defined safety-compliance step in this collection, and one performed manually rather than automatically. However, this review targets the evaluation set (10,140 entries); whether the 330,000-entry TAVGBench training data underwent safety filtering is unstated [uncertain]; nor is there any description of NSFW detectors, violent-content filtering, or facial privacy protection mechanisms.
[MM-Diffusion] No description of NSFW/copyright/privacy filtering [uncertain]. Indirect compliance awareness appears in dataset selection: one reason for choosing AIST++ was that its soundtrack is composed of copyright-cleared songs.
[AV-DiT] No description of any safety filtering [uncertain].
[Harmony] No description of any safety filtering [uncertain]. Its data contains a large amount of real human video (OpenHumanVid, SpeakerVid, self-collected clips), so facial and voiceprint privacy risk actually exists, but the paper contains no related statement.
[UniAVGen] No description of any safety filtering [uncertain]. This is the highest-risk case — the training data is an "internally collected real human audio-video dataset," and the model has audio-driven animation capability, a deepfake-sensitive capability, yet the paper provides no data-side privacy statement, nor any model-side abuse prevention or usage-restriction declaration.
[Common gap] None of the five provides a Model Card-level safety section, discusses deepfake abuse prevention, or mentions removal of celebrity likenesses or handling of content involving minors.

## Annotation Approach

### Caption Models Used (in-house VLM / open-source model, model scale) ⚠️

A full evolution from "no caption" to "fully automatic MLLM annotation":
[No-caption stage (MM-Diffusion / AV-DiT)] Both are unconditional generation — generating audio-video pairs directly from Gaussian noise, with no text conditioning, so their training data requires no caption at all. AV-DiT's paper explicitly acknowledges in its limitations that it "primarily focuses on unconditional audio and video generation." This is the most fundamental paradigm difference between them and all subsequent T2AV models. MM-Diffusion's zero-shot conditional generation (V2A/A2V) is achieved via gradient guidance, likewise not involving text captions.
[Reusing upstream captions (JavisDiT / JavisDiT++)] Does not perform its own video captioning; directly uses the text captions that come with the TAVGBench dataset (TAVGBench's captions were generated by its original authors using automated methods); the audio side's 780,000 entries likewise use the audio text descriptions that come with datasets such as AudioCaps, Clotho, and WavCaps (of which AudioCaps and Clotho are manually annotated, while WavCaps is automatically annotated). The data.md CSV schema's text and audio_text columns correspond to video and audio text descriptions respectively. The only place where the JavisDiT team builds its own captions is the evaluation set JavisBench: using "advanced Qwen-family models" to generate captions and classify into the 19 scene categories, with the specific model version and scale unspecified [uncertain].
[Fully automatic MLLM annotation (Harmony, the most advanced in this collection)] Uses Google Gemini to automatically annotate all 4 million audio-video clips, producing three types of content in one pass: ASR transcript text, descriptive video caption, and background sound caption. Gemini's specific version (1.5 Pro / 2.0 / 2.5, etc.), prompt template, and output-quality validation protocol are all undisclosed [uncertain]. Choosing Gemini rather than an open-source VLM suggests they value native long-video + audio joint understanding capability — the same idea behind Ovi's use of an "audio-aware MLLM."
[UniAVGen] No mention of any caption model or text-annotation pipeline whatsoever [uncertain]; its task forms (audio-driven animation, video-to-audio dubbing, real human video generation) are themselves less dependent on text captions.

### Caption Density and Structuring Level (short/long/dense description, structured fields such as camera movement, style tags) ⚠️

[MM-Diffusion / AV-DiT] No captions (unconditional generation); this dimension does not apply.
[JavisDiT / JavisDiT++] Caption structure is inherited from upstream datasets, not self-built:
- Video side: TAVGBench provides a single natural-language description (describing both audiovisual content at once), of medium-length overall description density, with no structured fields.
- Audio side: audio captions that come with each audio dataset, with widely varying style — AudioCaps/Clotho are one-sentence human-written audio descriptions, WavCaps are ChatGPT-assisted generated descriptions, and ESC50/UrbanSound8K/GTZAN are essentially category labels rather than descriptive sentences. This "mixed multi-source caption style" is an inherent characteristic of its audio pretraining data, which the paper does not standardize [uncertain].
- There is genuine structured management at the CSV schema level: path, id, relpath, num_frames, height, width, aspect_ratio, fps, resolution, audio_path, audio_fps, text, audio_text — but these are metadata fields rather than structured tags within the caption itself.
- No explicit structured annotation of camera movement, shot scale, lighting, style, etc. [uncertain].
[Harmony (the highest degree of structuring in this collection)] A single Gemini annotation pass produces three separate text fields: (1) ASR transcript (transcript), (2) descriptive video caption (video caption), and (3) background sound caption (audio caption / background sound caption). This is a clearly factorized multi-field structure, opposite to Ovi's "single caption with embedded tags" approach and closer to Script-a-Video's factorized-stream approach. This three-field structure directly determines Harmony-Bench's conditioning setup: the environmental-sound subset uses audio/video caption as the condition, the speech subset uses transcript text as the primary condition, and the complex-scene subset uses "the full set of multimodal prompts" — i.e., the three fields can be flexibly combined per task, a direct benefit of the factorized design. Prompt-template details and the average length of each field are undisclosed [uncertain].
[UniAVGen] Not described [uncertain].

### Joint Audio-Video Caption Structure (whether visual + auditory tracks are covered simultaneously, whether split into separate fields, e.g., LTX-2's full soundscape description, Script-a-Video's factorized streams, Foley-Omni's three fields) ⚠️

The five works' approaches to joint audio-video caption structure diverge markedly, conveniently covering three typical paths:
[Path Zero: no joint caption (MM-Diffusion / AV-DiT)] Unconditional generation, no joint-caption schema exists. Audiovisual alignment relies entirely on architectural mechanisms (random-shift attention / cross-modal adapters) and the native synchrony of the data itself, without any text intermediary.
[Path One: single mixed caption (JavisDiT / JavisDiT++)] Uses TAVGBench's single piece of text, which describes both visual and auditory content at once, without splitting into separate fields and without timestamps or speech tags. JavisDiT's key insight is precisely that "a single text condition is insufficient to guarantee fine-grained spatio-temporal synchronization" — so instead of engineering the caption structure, it additionally introduces HiST-Sypo (the Hierarchical Spatio-Temporal Synchronized Prior Estimator) outside of the text, which first estimates a coarse-grained global prior and a fine-grained spatio-temporal prior from the prompt, then uses that prior to jointly guide denoising of the two streams. This can be understood as: JavisDiT moves the "spatio-temporal alignment information the joint caption should encode" from the text layer to an implicit prior layer. This is a third path, distinct from both Ovi (encoding timing via inline <S>dialogue<E> tags) and Foley-Omni (three-field split).
[Path Two: fully split into three fields (Harmony, the most complete joint schema in this collection)] Gemini annotation produces three independent text tracks:
(1) Visual track — descriptive video caption;
(2) Speech track — ASR transcript text;
(3) Non-speech auditory track — background sound/ambient sound caption.
The three exist as independent condition fields rather than being concatenated into one sentence, so they can be combined per task. Harmony-Bench's three-tier subsets directly validate the value of this design: the environmental-sound subset uses only video+audio caption, the speech subset mainly uses transcript, and the complex-scene subset uses the full set of multimodal prompts — showing that this schema supports "on-demand activation of condition channels." This appears to contradict Ovi's ablation finding that "a merged single T5 embedding outperforms separate CLAP/T5 encoding," but the two actually target different problems: Ovi's separation happens at the encoder level (different encoders leading to a fractured representation space), while Harmony's split happens at the field level (which may still share the same text encoder, umT5) [uncertain: whether Harmony's three fields share a umT5 encoding before being concatenated].
[UniAVGen] Does not describe a joint caption structure [uncertain]; its conditioning form is primarily reference image + reference audio + transcript, with text caption not being a core condition channel.

### Dialogue Transcription and Speaker Attribute Annotation (ASR transcription + speaker identity/language/accent/emotion) ⚠️

[MM-Diffusion / AV-DiT] Training data contains no human speech at all (natural ambient sound and instrumental music), so there is no transcription and no speaker-attribute annotation; this dimension does not apply.
[JavisDiT / JavisDiT++] The reverse operation — not only does it not transcribe, it actively uses FunASR to detect and remove videos containing human speech, purging speech from the audio-video training data. This is a deliberate capability trade-off: giving up lip-sync and dialogue generation in exchange for focus on environmental sound/sound-effect event alignment. FunASR here plays the role of a "speech detector" rather than a "transcriber." Although the audio pretraining stage includes speech-type data, no transcription annotation is performed on it.
[Harmony (has complete transcription)] Gemini automatic annotation explicitly produces ASR transcript text, which is fed into the model as a speech condition channel; the speech data comes from Emilia (TTS corpus, which itself carries transcripts), OpenHumanVid, and SpeakerVid (two-person interactive data, naturally containing multi-speaker scenarios). On the speaker-attribute front, Harmony has a mechanism independent of transcription — Stage 2 "Timbre Disentanglement Finetuning," which uses cross-utterance mismatched reference/target pairs from the same speaker (and for ambient sound, non-overlapping segments of the same clip), training the model to extract timbre features from a 1–3 second reference audio without leaking content. This effectively decouples timbre through "data pairing method" rather than "attribute annotation," a lighter-weight approach than explicitly labeling age/gender/accent. However, explicit attribute labels such as age, gender, accent, and emotion are not seen to be annotated [uncertain], and speaker diarization or speaker ID are not mentioned either [uncertain].
[UniAVGen] The evaluation side uses Whisper to compute WER, indicating transcription exists at the evaluation stage; whether ASR transcription annotation is performed on the training data side is unstated [uncertain]. The model explicitly optimizes for "timbre consistency" and "emotion consistency" metrics, implying the existence of timbre and emotion condition channels during training (very likely realized via reference audio rather than text attribute labels), but the specific annotation method is undisclosed [uncertain].

### Geometric and Structured Annotation (camera parameters, depth, 3D point tracks, action labels, explicit state) ⚠️

This collection is almost entirely blank here, with only UniAVGen having one face-related structured element:
[UniAVGen (the only one)] Includes a Face-Aware Modulation module, which requires facial region information to apply modulation to a specific region, so the data side presumably has a face-detection/facial-region annotation step, but the paper does not describe which face detector is used, whether it is a bounding box or a mask, or whether it is precomputed offline [uncertain]. Evaluation using the EMTD benchmark (audio-driven facial-animation benchmark) also corroborates its face-related processing.
[MM-Diffusion] No geometric annotation whatsoever. Notably, the AIST++ dataset itself comes with 3D human pose and camera parameter annotations (AIST++'s original design goal was 3D dance-motion generation), but MM-Diffusion makes no use of these annotations at all, using only RGB video and audio — i.e., "the dataset has geometric annotation, but the work doesn't use it."
[AV-DiT] No geometric annotation whatsoever.
[JavisDiT / JavisDiT++] No camera parameters, no depth, no 3D point tracks, no pose keypoints, no action labels, no segmentation masks. The only thing approaching structured data is the flow score (a scalar motion-intensity value) and the resolution/frame-count metadata in the CSV. JavisDiT's HiST-Sypo prior, while named a "spatio-temporal prior," is an implicit vector estimated from text rather than an explicit geometric annotation.
[Harmony] No geometric annotation [uncertain].
[Common judgment] The technical route across this collection is generally "letting cross-modal attention mechanisms learn alignment autonomously" rather than "constraining alignment via explicit geometric priors" — consistent with Ovi's stance that "no face bbox or face mask is needed," suggesting the audio-video joint-generation community broadly favors data-driven approaches over geometric-prior-driven ones. UniAVGen's Face-Aware Modulation is a rare exception to this trend, at the cost of limiting the model's capability to real-human speaking scenarios.

### Synthetic Data Construction (controlled perturbation/editing to construct training pairs, e.g., InstructAV2AV) ⚠️

Two works in this collection use synthetic/self-bootstrapped data construction, and both are the most instructive parts of the data side of this collection:
[JavisDiT — controlled construction of asynchronous negative samples (ST-Prior training)] To train the HiST-Sypo spatio-temporal synchronized-prior estimator, it is necessary to distinguish "synchronized" from "unsynchronized" audio-video pairs. Beyond the 610,000 synchronized triplets, the authors additionally "synthesize asynchronous negative samples," paired with contrastive learning to train the prior estimator. This is a typical case of "constructing training pairs via controlled perturbation" — manually creating negative examples by time-shifting the audio track, substituting it, or cross-sample mismatching. The specific negative-sample construction method (how many frames of time-shift, whether cross-sample random pairing is used, the positive-to-negative sample ratio) is described in Appendix C.2.4 "Negative Sample Construction" of the paper, but this section is truncated in the public HTML version and details could not be obtained [uncertain]. This design stands in sharp contrast to Ovi's approach of "using strict thresholds to discard unsynchronized data": Ovi discards unsynchronized data, while JavisDiT actively manufactures unsynchronized data to use as negative examples.
[JavisDiT++ — model-self-generated preference data (AV-DPO)] Uses a reference model to generate N=3 audio-video candidates for each prompt in a pool of 30,000 prompts, combined with 1 ground-truth sample to form a "1 ground truth + 3 generated" candidate group, then uses six reward models to score and rank, constructing about 25,000 preference pairs. This is entirely model-self-bootstrapped synthetic data. A quantitative finding worth noting: about 30% of the winning samples in the final preference data come from model generation rather than ground truth — i.e., in nearly three out of ten cases the model can generate a sample better than the real data, from which the authors infer the baseline model already possesses fairly strong generation capability. This proportion is a useful reference signal for judging "when to switch to DPO/RLHF."
[Harmony — construction at the data-pairing level] Stage 2 timbre disentanglement finetuning uses "cross-utterance mismatched reference-target pairing": speech data takes different utterances from the same speaker as reference and target, and ambient-sound data takes non-overlapping segments of the same clip as reference and target. This is not generating new data, but constructing a training signal by re-pairing existing data — a lightweight synthetic-pairing strategy.
[MM-Diffusion / AV-DiT / UniAVGen] Use no synthetic data construction whatsoever [uncertain].

### Degree of Human Involvement (manual annotation, manual quality control, model pre-screening + manual review) ⚠️

Human involvement generally concentrates on the evaluation side rather than the data side — a shared trait of this collection:
[MM-Diffusion (the largest human effort on the evaluation side)] Conducted a large-scale Turing test, collecting 10,000 human votes in total, showing that over 80% of the generated audio-video clips fooled human evaluators. No manual annotation or quality-control step is described on the data side [uncertain], but Landscape's small scale of just 1,000 entries and its self-description of "high-fidelity" hints at possible manual curation.
[AV-DiT] No human involvement described [uncertain].
[JavisDiT / JavisDiT++ (clear human involvement on both the data side and evaluation side, the most thorough in this collection)]
- Data side: during JavisBench construction, all YouTube-sourced content underwent "strict manual legal and ethical verification" — the only clearly defined manual data-side review step in this collection.
- Metric-validation side: to validate the effectiveness of the JavisScore metric, a human-annotated evaluation dataset of 3,000 samples was built, using human judgments of synchronization to calibrate the metric. This "build the metric first, then use manual annotation to validate the metric itself" approach is relatively rigorous evaluation methodology, though the number of annotators, recruitment method, and inter-annotator agreement are undisclosed [uncertain].
- Model-evaluation side: JavisDiT++ uses 100 prompts from JavisBench, with 3 volunteers performing blind win-tie-lose preference judgments.
[Harmony] Conducted systematic human ablation studies to validate the contribution of each component; on the data-side human involvement, its data sources include AudioCaps (about 128 hours) and Clotho (about 31 hours), both manually annotated audio-caption datasets, while WavCaps (about 7,600 hours) is automatically annotated — the paper clearly distinguishes the manual/automatic annotation nature of the three, showing the team is deliberate about stratifying annotation quality. Annotation of their own 4 million clips relies entirely on Gemini, with no manual-review description [uncertain].
[UniAVGen] Describes no human-involvement step whatsoever [uncertain].
[Common observation] The data pipelines of the five works are essentially fully automated, with human effort mainly used for evaluation and metric calibration rather than entry-by-entry data review — a contrast with industrial-grade models (which often have dedicated data-annotation teams), and a realistic constraint of academic work.

## Audio-Video Alignment

### Audio-Video Synchronization Detection Method (lip sync, event alignment) ⚠️

Audio-video synchronization detection is the dimension of greatest technical divergence in this collection, with the five works representing five distinct approaches:
[MM-Diffusion (2022) — no sync detection, relying on architecture and dataset selection] The data side does no synchronization filtering at all. Synchronization is ensured by two things: (1) dataset selection — Landscape's 9 scene categories are all natural phenomena with strong causal binding between visuals and sound, and AIST++ is dance movement bound to musical beat; (2) architecture — the random-shift based attention block performs cross-modal attention within a temporal neighborhood. The evaluation side also has no dedicated synchronization metric, using only FVD (video quality) + FAD (audio quality) + manual Turing test. This is the early stage where "synchronization cannot be quantified, only judged by eye."
[AV-DiT (2024) — likewise no sync detection] Reuses MM-Diffusion's data and evaluation setup.
[JavisDiT (2025) — proposes the in-house synchronization metric JavisScore, the most important methodological contribution in this collection] The authors point out that existing synchronization metrics are unreliable on real-world complex content, and thus propose JavisScore:
- Computation method: each audio-video pair is cut into multiple 2-second windows with 1.5-second overlap (i.e., a sliding-window stride of 0.5 seconds); ImageBind is used to compute audio-video synchronization for each window; specifically, the similarity is computed between all frames within the window and that window's audio, and then the "worst-synchronized 40% of frames" are taken to participate in the score (rather than averaging) — this "take the worst 40%" design is key, because averaging would let a large number of well-aligned frames dilute local desynchronization, whereas the real perception of desync is precisely dominated by the worst-aligned segments.
- Validity verification: a 3,000-sample human-annotated evaluation dataset is built to verify that JavisScore aligns more closely with human judgment than existing metrics.
- Training-side synchronization mechanism: the HiST-Sypo spatio-temporal synchronized prior + contrastive learning using synthesized asynchronous negative samples to train the prior estimator. That is, rather than doing synchronization filtering at the data level, JavisDiT models synchronization as a learnable prior.
[JavisDiT++ (2026) — explicit frame-level synchronization + Synchformer as reward] TA-RoPE (Temporal-Aligned RoPE) enforces frame-level alignment between audio tokens and video tokens at the positional-encoding level (sharing the same lineage of idea as Ovi's scaled-RoPE); the AV-DPO stage uses Synchformer as one of the reward models for temporal synchronization, incorporating synchronization into the preference-optimization objective. Evaluation reports the DeSync metric.
[Harmony (2025) — a three-pronged approach] (1) At the training-mechanism level: Cross-Task Synergy uses bidirectional generation tasks (audio-driven video, video-driven audio) to suppress alignment drift during joint denoising; (2) at the architecture level: the GLDI module decouples global style alignment and local temporal precision into two branches; (3) at the inference level: Synchronization-Enhanced CFG amplifies the alignment signal — ablation shows this component contributes the largest synchronization gain (Sync-C improving from 5.09 to 6.51). On the data side, an "audio-visual consistency scoring model" is used to filter speech data.
[UniAVGen (2025) — architecture-level asymmetric interaction] Relies on asymmetric cross-modal interaction + Face-Aware Modulation + modality-aware CFG to ensure synchronization; no synchronization filtering is described on the data side [uncertain]. Evaluation uses the SyncNet metric.

### Specific Synchronization-Detection Metrics and Thresholds (SyncNet/Synchformer/LSE/in-house, threshold values, e.g., MOVA's LSE-D≤9.5 and LSE-C≥4.5, SkyReels-V4's SyncNet |offset|≤3∧conf>1.5) ⚠️

This collection broadly reports synchronization metrics but rarely gives data-filtering thresholds — the biggest gap versus industrial models such as Ovi (|offset|≤3 and conf>1.5), MOVA (LSE-D≤9.5 and LSE-C≥4.5), and SkyReels-V4:
[MM-Diffusion / AV-DiT] No synchronization metric, no threshold. Evaluation consists only of FVD (computed with the i3d model) and FAD (computed with AudioCLIP), plus a manual Turing test (10,000 votes, >80% fooling humans).
[JavisDiT — proposes JavisScore (the only new metric in this collection)]
- Specific parameters fully disclosed: a 2-second window, 1.5-second overlap (0.5-second sliding-window stride), the underlying similarity computed with ImageBind, aggregating the "worst-synchronized 40% of frames" within each window.
- Validation set: 3,000 human-annotated samples.
- These are metric-computation parameters, not data-filtering thresholds — JavisDiT does not filter training data by synchronization score [uncertain].
[JavisDiT++ — an 11-dimension evaluation system] Full metric list: quality dimension — FVD, FAD; text-consistency dimension — TV-IB (text-video ImageBind), TA-IB (text-audio ImageBind), CLIP-Score, CLAP-Score; audio-video consistency dimension — AV-IB, AVHScore, JavisScore; synchronization dimension — DeSync (based on Synchformer). AV-DPO uses Synchformer as a synchronization reward model, but the scoring threshold/ranking details are undisclosed [uncertain]. All evaluation is conducted at the 240P, 4-second configuration.
[Harmony — four synchronization metrics] Sync-C and Sync-D (SyncNet-family lip-sync confidence and distance), DeSync Score (based on Synchformer), and ImageBind (IB) Score. The ablation gives a key chain of values: Sync-C baseline 4.29 → (add RoPE positional alignment) 4.80 → (add Cross-Task Synergy training) 5.09 → (add Synchronization-Enhanced CFG) 6.51. The name and threshold of the "audio-visual consistency scoring model" used for data filtering are undisclosed [uncertain], the most critical missing information on Harmony's data side.
[UniAVGen] Evaluation uses SyncNet (synchronization), VBench (video quality), AudioBox-Aesthetics (audio quality), and Whisper WER (speech intelligibility); the data-side synchronization filtering threshold is not mentioned [uncertain].
[Cross-collection significance] This collection offers a rich set of synchronization "evaluation metrics" (JavisScore / DeSync / Sync-C/D / AV-IB / AVHScore), but almost never offers synchronization "data-filtering thresholds" — showing that academic baselines focus more on how to measure synchronization, while industrial models focus more on how to use thresholds to filter clean data — the two are complementary.

### Separate Handling of Temporal vs. Semantic Synchronization (temporal alignment and content-semantic matching as two independent filtering conditions) ⚠️

The JavisDiT series has the clearest and most theoretically deep treatment of this dimension in the collection; the other works show weaker separation:
[JavisDiT — the hierarchical design itself is an explicit separation of temporal and semantic] The "hierarchical" nature of HiST-Sypo (Hierarchical Spatio-Temporal Synchronized Prior) is essentially splitting alignment into two layers:
- Coarse-grained global prior — corresponding to semantic-level matching (whether overall content and style are consistent);
- Fine-grained spatio-temporal prior — corresponding to precise temporal and spatial alignment (when, where, what sound is emitted).
The two priors are separately estimated from the text prompt, then jointly guide the denoising of both streams. This is the only design in this collection to explicitly decouple "semantic synchronization" from "temporal synchronization" at the model-architecture level.
- JavisBench's evaluation dimensions reflect the same separation: among the five dimensions, "Temporal Composition" and "Spatial Composition" are temporal/spatial dimensions, while "Event Scenario," "Sound Type," and "Video Style" are semantic dimensions.
- The evaluation metrics are also separated: DeSync (Synchformer, purely temporal) vs. AV-IB / AVHScore (ImageBind family, purely semantic similarity) vs. JavisScore (sliding window + worst 40% of frames, in between the two but leaning toward local temporal).
[JavisDiT++] Continues this separation and further solidifies it architecturally: TA-RoPE is specifically for frame-level temporal alignment (temporal), while MS-MoE and cross-modal attention handle semantic interaction (semantic); the AV-DPO reward design is likewise separated — Synchformer scores temporal synchronization, ImageBind scores cross-modal semantic similarity.
[Harmony — the GLDI module is an architecture-level temporal/semantic decoupling] Global-Local Decoupled Interaction explicitly splits interaction into two branches: the global branch handles "style alignment" (semantic level), and the local branch handles "temporal precision" (temporal level). This echoes JavisDiT's layered prior in spirit, making it the second work in this collection to explicitly perform this separation. However, data filtering is not separated — there is only one generic "audio-visual consistency scoring model" [uncertain: whether this model measures temporal or semantic consistency].
[MM-Diffusion / AV-DiT / UniAVGen] Perform no explicit separation of temporal and semantic synchronization [uncertain]. MM-Diffusion's random-shift attention is confined to a temporal neighborhood for attention, implying a preference for temporal locality, but this is not a deliberate separation design.
[Methodological value] This collection validates a trend: post-2025 audio-video joint-generation work broadly recognizes that "temporal alignment" and "semantic matching" require different mechanisms — JavisDiT uses layered priors, Harmony uses a decoupled interaction module, and Ovi uses RoPE for temporal + cross-attention for semantic, three different roads converging to the same destination.

### Audio Quality Filtering (SNR, silence detection and silence-proportion threshold, no-audio-track removal, off-screen source removal, background music separation) ⚠️

Audio-side filtering is broadly missing, with only formatting preprocessing:
[JavisDiT / JavisDiT++] Explicitly states that the audio pretraining stage "adopts no data filtering strategy," to ensure maximum text-to-audio generation capability covering general sound, music, and speech — a deliberate "no filtering" decision, opposite to most models' strict audio screening. The audio side has only three formatting steps: truncation to under 30 seconds, uniform resampling to 16kHz, and extraction of audio statistics. No SNR threshold, no silence detection, no no-audio-track removal, no background-music separation, no clipping/distortion detection [uncertain]. There is, however, one special "audio-content filter" at the audio-video stage — using FunASR to remove videos containing human speech, but this is a category filter rather than a quality filter.
[Harmony] Speech data is filtered via the "audio-visual consistency scoring model," which is cross-modal consistency filtering rather than pure audio quality filtering; no filtering is described at the audio-quality level (SNR, silence, noise) [uncertain]. Audio encoding goes through two paths: MMAudio's VAE encoder + F5-TTS's speech encoder. Reference audio is a random 1–3 second clip, with no statement on whether validity checks are performed [uncertain].
[UniAVGen] Only formatting processing: audio is sampled at 24,000 Hz and converted to mel spectrogram; after generation, the waveform is restored using the Vocos vocoder. Note that its 24kHz sampling rate is higher than JavisDiT/AV-DiT/MM-Diffusion's 16kHz, giving wider bandwidth and better speech fidelity — consistent with its focus on real human speech. No audio quality filtering is described [uncertain].
[MM-Diffusion / AV-DiT] No audio quality filtering. AV-DiT's processing is "truncate or pad to a 1.6-second waveform, 16kHz sampling, convert to mel spectrogram 40×16×8" — purely formatting.
[Common judgment] Audio quality control in this collection largely relies on upstream datasets (AudioCaps, Clotho, Emilia, etc. are themselves curated corpora), without building their own audio quality-control step — feasible on small-scale curated data, but a clear risk exposure on self-collected million-scale data (such as Harmony's 2 million self-collected environmental-sound clips).

### Classification and Separate Handling Strategy for Speech/Sound Effects/Music ⚠️

The handling strategy for the three audio categories — speech, sound effects, and music — is key to distinguishing each work's positioning in this collection:
[MM-Diffusion / AV-DiT — binary and never mixed] Landscape corresponds to "environmental sound effects," AIST++ corresponds to "music," and the two datasets each independently train two separate models (Landscape.pt and AIST++.pt are two independent sets of weights in the repository), with no mixed training. Speech is not involved at all. This "one domain, one model" approach was an inevitable choice of the small-data era.
[JavisDiT / JavisDiT++ — full-category mixing at the audio stage, deliberate speech removal at the video stage]
- Audio pretraining (780K entries): explicitly pursues full-category coverage — "no filtering is applied to ensure maximum T2A capability, covering three categories: general sound, music, and speech." The data sources have a clear division of labor: AudioSet/AudioCaps/VGGSound/WavCaps/Clotho provide general sound, GTZAN/MusicInstrument provide music, UrbanSound8K/ESC50 provide environmental sound and urban noise, and MACS provides acoustic scenes.
- Audio-video SFT: uses FunASR to remove most videos containing speech — i.e., actively abandoning the speech category at the joint-generation stage. This is the clearest "audio-category trade-off" decision in the collection, whose cost is that the model lacks lip-sync and dialogue-generation capability, and whose benefit is that all limited compute and data are devoted to event-level alignment of environmental sound/sound effects (also the focus of JavisBench evaluation).
[Harmony — strict 1:1 speech/environmental-sound split, each with its own encoder]
- Data side: 2 million speech entries vs. 2 million environmental-sound entries, 1:1 mixed, with the ratio maintained in both Stage 1 and Stage 3.
- Architecture-side differentiation: the audio VAE encoder uses MMAudio (skilled at general audio/sound effects), and the speech encoder uses F5-TTS (skilled at speech) — the two audio categories take different encoding paths, the only work in this collection to differentiate audio type at the encoder level.
- Training-side differentiation: Stage 2 timbre disentanglement uses different pairing strategies for the two data types (speech uses same-speaker cross-utterance pairing, environmental sound uses non-overlapping segments of the same clip).
- Evaluation-side differentiation: Harmony-Bench's three tiers correspond respectively to environmental sound, speech, and their co-occurrence.
It can be said that Harmony is the only work in this collection that systematically treats "speech" and "non-speech" as two parallel data streams processed separately throughout, while still insisting they must be jointly trained.
[UniAVGen — a single speech-only route] Stage 1 pretrains on Emilia (a pure TTS corpus), with the whole pipeline focused on real human speech; the handling of sound effects and music is not described [uncertain]. This gives it an advantage in dubbing, lip-sync, and timbre/emotion consistency, but its general sound-effect generation capability is questionable.

## Training Coordination

### Multi-Stage Training Curriculum and Data-Curriculum Scheduling (basis for stage division: resolution/duration/quality score/modality; low-res→high-res, image→video, short→long) ⚠️

This collection shows a clear convergent trend in training-curriculum design — three post-2025 works independently adopt a three-stage paradigm of "audio-only pretraining → audio-video joint training → reinforcement/multi-task":
[MM-Diffusion (no curriculum, but a two-stage resolution scheme)] Single-stage joint training, no multi-stage curriculum. The only staging is spatial-scale: the base model trains at 64×64, and a separate super-resolution model is trained to upsample to 256×256 (this SR model is initialized from guided-diffusion's upsampling model). This is a primitive form of "low-res → high-res" curriculum, but the two models are independent rather than a progressive curriculum.
[AV-DiT (no curriculum)] Single-stage training for 100,000 iterations, batch size 16, lr 5e-4, AdamW optimizer, completed on NVIDIA RTX A6000 GPUs; only the newly inserted adapter layers are trainable, with the DiT backbone frozen. The parameter-efficient route itself substitutes for the role of a curriculum.
[JavisDiT (three stages, dividing basis is "modality → alignment capability → joint generation")]
- Stage 1: audio pretraining — 780,000 audio-text pairs, 55 epochs, solidifying the audio branch's T2A capability (the video tower uses Open-Sora, the audio tower uses AudioLDM2, with the VAE frozen throughout).
- Stage 2: ST-Prior training — 610,000 synchronized triplets + synthesized asynchronous negative samples, using contrastive learning to train the HiST-Sypo spatio-temporal synchronized prior estimator. This is a dedicated stage for "learning to judge what counts as synchronized," unique in this collection.
- Stage 3: JAVG training — 610,000 triplets, fine-tuning the cross-attention and bidirectional attention modules.
[JavisDiT++ (three stages, with a parameter-allocation scheme worth recording)]
- Stage 1 audio pretraining: 780,000 audio-text entries, 50 epochs, training the audio FFN (794M parameters).
- Stage 2 audio-video SFT: 330,000 triplets, 2 epochs, LoRA training (121M parameters).
- Stage 3 AV-DPO: 25,000 preference pairs, 1 epoch, LoRA training (121M parameters).
Note the stark difference in epoch counts (50 : 2 : 1) and the decrease in trainable parameters (794M → 121M → 121M): audio capability is learned from scratch through many epochs, joint capability is adapted through few epochs of LoRA, and preference alignment needs only 1 epoch — this configuration provides useful reference for "how to perform joint generation with small data."
[Harmony (three stages, dividing basis is "audio foundation → timbre disentanglement → cross-task joint")]
- Stage 1 base audio pretraining: 100,000 iterations, global batch size 1536, maximum clip length 10 seconds, speech and environmental sound mixed 1:1, reference audio being a random 1–3 second clip.
- Stage 2 timbre disentanglement finetuning: 20,000 iterations, cross-utterance mismatched reference-target pairing.
- Stage 3 cross-task audio-video joint training: 10,000 iterations, batch size 128, still maintaining the 1:1 data ratio, with λv=0.1 (video-driven loss) and λa=0.3 (audio-driven loss).
- lr held constant at 1e-5 throughout. The video branch is initialized from Wan2.2-5B.
[UniAVGen (three stages, dividing basis is "audio → joint → multi-task")]
- Stage 1 audio pretraining: Emilia English subset, 160,000 steps, batch size 256, lr 2e-5.
- Stage 2 joint training: internal real human audio-video data, 30,000 steps, batch size 32, lr 5e-6.
- Stage 3 multi-task learning: 10,000 steps, five task categories at a 4:1:1:2:2 ratio.
[Common regularity (an important conclusion)] JavisDiT++, Harmony, UniAVGen, and the contemporaneous Ovi all adopt the strategy of "first training the audio branch well with a large amount of pure-audio data, then doing joint adaptation with a small amount of audio-video paired data," with the audio-stage step/epoch count far exceeding the joint stage. The reason is that high-quality audio-video paired data is scarce while pure-audio data is abundant, which has become a de facto standard curriculum in the field of audio-video joint generation.
[Common gap] None of the five adopts a progressive curriculum of "low-res→high-res," "short→long," or "image→video"; resolution and duration remain fixed throughout training.

### Changes in Data Ratio Across Training Stages (pretraining/annealing/SFT high-quality subset) ⚠️

The degree of disclosure of per-stage data ratios varies widely, with Harmony and UniAVGen giving explicit figures:
[Harmony (the most explicit)]
- Both Stage 1 and Stage 3 strictly maintain a "human speech : environmental sound = 1:1" mixing ratio — the only explicit intra-modality category ratio in this collection that persists across multiple stages, and it is maintained even at the joint-training stage, showing the team believes speech capability and environmental-sound capability must be maintained in tandem rather than staged-biased (in contrast to Ovi's staged-bias strategy of "pretrain speech-dominant → finetune to add sound effects").
- Loss-weight-level ratio: λv=0.1 (loss weight for the video-driven generation task), λa=0.3 (loss weight for the audio-driven generation task) — note this is the weighting between the two auxiliary tasks in Cross-Task Synergy, with the audio-driven task weighted 3× the video-driven task.
[UniAVGen (multi-task ratio explicit)] Stage 3 multi-task learning uses a 4:1:1:2:2 ratio across five task categories, the only work in this collection to give explicit multi-task sampling ratios. The main task (joint generation) takes 4/10, with two tasks at 1/10 and two tasks at 2/10 corresponding respectively to derivative tasks such as video-to-audio dubbing and audio-driven animation [uncertain: the exact correspondence between the five numbers and the five task categories]. On cross-stage data ratio: Stage 1 pure audio (Emilia English subset) → Stages 2–3 pure internal audio-video data is a complete replacement rather than a mix, with no anti-forgetting audio-data replay observed [uncertain].
[JavisDiT / JavisDiT++] Data is completely replaced rather than mixed between stages:
- Stage 1's 780,000 pure-audio entries → Stage 2's 330,000 audio-video triplets → Stage 3's 25,000 preference pairs — the three differ in data type, with no mixing and no replay.
- Within the audio stage: the 10 datasets are mixed but the ratio is undisclosed [uncertain]; only that "no filtering is applied to ensure maximum coverage of the three categories, sound effects/music/speech."
- An implicit ratio decision: Stage 2 uses FunASR to remove speech-containing videos, effectively pushing the speech proportion in the audio-video stage down to near zero — a stark contrast to Stage 1's full-category coverage. This could lead to catastrophic forgetting of the speech capability the audio branch learned in Stage 1, an issue the paper does not discuss [uncertain].
[MM-Diffusion / AV-DiT] Single-stage training, no stage ratios; the two datasets are trained independently, with no cross-domain mixing.
[Annealing/high-quality SFT subset] None of the five describes an annealing stage or a late-stage high-quality subset fine-tune [uncertain]; JavisDiT++'s DPO stage functionally partially fills this role.

### Post-Training Data (SFT curated-subset scale and screening criteria, preference-pair count and annotation method, reward-model training data) ⚠️

Only JavisDiT++ has a complete post-training data system, making it the sole contributor to this collection on this dimension, and the part most worth learning from:
[JavisDiT++'s AV-DPO (Audio-Video Direct Preference Optimization) — fully disclosed]
- Preference-data scale: about 25,000 audio-video preference pairs (about 25k audio-video preference pairs), trained for 1 epoch, with LoRA's 121M parameters trainable.
- Prompt pool: 30,000 text captions, explicitly non-overlapping with the SFT training data (apart from the SFT training data), avoiding data leakage.
- Candidate construction: for each prompt, a reference model generates N=3 audio-video candidates, plus 1 ground-truth sample, forming a candidate group of 4.
- Reward-model ensemble (six models each with its own responsibility, the most complete multi-dimensional reward design in this collection):
  · Audio quality → AudioBox (AudioBox-Aesthetics)
  · Text-audio alignment → ImageBind
  · Video quality → VideoAlign
  · Text-video alignment → ImageBind
  · Audio-video cross-modal similarity → ImageBind
  · Temporal synchronization → Synchformer
- Pairing strategy: adopts "normalized modality-aware ranking" to select winning/losing pairs, with the authors explicitly stating the goal is to "ensure internal consistency within each modality, rather than pairing high-quality audio with low-quality video" — a core engineering lesson for constructing preference pairs under multi-dimensional rewards, directly addressing the problem where contradictory samples like "high-quality audio + low-quality video" would pollute the preference signal.
- A quantitatively valuable observation: about 30% of the winning samples in the final preference data come from model generation rather than ground truth, from which the authors infer that "the baseline model itself already possesses fairly strong generation capability." This proportion can serve as an empirical signal for judging whether a model has reached the stage where "self-generated data can be used for preference optimization."
- Open-source status: the DPO preference data is "being prepared for release," not yet public [uncertain].
[JavisDiT (v1)] No DPO/RLHF; all three stages are purely supervised training.
[Harmony] No preference-optimization data. Its "Cross-Task Synergy" functionally resembles this in part — using bidirectional generation tasks to provide additional supervisory signal to suppress alignment drift, but this is multi-task supervised learning, not preference learning.
[UniAVGen] Stage 3's multi-task learning (five tasks at a 4:1:1:2:2 ratio) is likewise multi-task supervision rather than preference optimization; no SFT curated subset, no preference pairs, no reward model [uncertain].
[MM-Diffusion / AV-DiT] No post-training stage.
[Cross-collection significance] JavisDiT++'s AV-DPO is a relatively early systematic preference-optimization practice in the audio-video joint-generation field, and its "multi-reward-model + modality-aware ranking" design can be directly transferred to the post-training of other AV models.

### Data-Processing Infrastructure and Throughput (NeMo Curator/Data-Juicer/in-house, GPU acceleration ratio, processing scale, cost) ⚠️

Disclosure of data infrastructure in this collection is extremely sparse, and scale is far smaller than industrial models; none uses a professional data-processing framework:
[Common trait] None of the five mentions mature data-processing frameworks such as NeMo Curator, Data-Juicer, Ray Data, or Spark, nor gives GPU acceleration ratios, processing throughput (clips/hour), or processing cost figures [uncertain].
[JavisDiT / JavisDiT++ (relatively the most disclosed, and engineering-reproducible)]
- Data management uses a CSV-centric approach (inheriting Open-Sora's data-management system), with columns including path, id, relpath, num_frames, height, width, aspect_ratio, fps, resolution, audio_path, audio_fps, text, audio_text; it supports merging multi-source CSVs. This lightweight approach suits sub-million scale and would become a bottleneck above ten-million scale.
- Data preparation is a series of scripted steps (convert → get info → trim → resample → add dummy video → output CSV), fully documented in assets/docs/data.md, giving it the highest reproducibility in this collection.
- Training side: JavisDiT++'s trainable-parameter count is finely controlled (794M audio FFN in Stage 1, 121M LoRA each in Stages 2 and 3), a clear memory/compute optimization design; the specific GPU model, GPU count, and training duration are undisclosed [uncertain].
[AV-DiT (the only work with publicly disclosed hardware specs)] Trained for 100,000 iterations, batch size 16, on NVIDIA RTX A6000 GPUs — consumer/workstation-grade hardware, an extreme contrast to industrial models' thousand-GPU clusters, and also confirms its "parameter-efficient" positioning (only adapters trainable).
[MM-Diffusion] Training hardware and duration undisclosed [uncertain]. Data is released pre-packaged in preprocessed form via Google Drive and Baidu Netdisk, for ease of reproduction.
[Harmony] Compute scale can be inferred from batch sizes: Stage 1 global batch 1536 with 100,000 iterations, Stage 3 batch 128 with 10,000 iterations, paired with a Wan2.2-5B base — should require a fairly large GPU cluster, but GPU count, model, and training duration are all undisclosed [uncertain]. Gemini automatic annotation of 4 million clips involves large-scale API calls, with the cost and throughput undisclosed [uncertain] — this is actually the largest hidden cost on this work's data side.
[UniAVGen] Stage 1 batch 256, 160,000 steps; Stage 2 batch 32, 30,000 steps (an 8× drop in batch size, indicating that audio-video joint training's memory pressure is far higher than pure-audio training); hardware specs undisclosed [uncertain].

## Results Comparison

### Quantified Impact of Data-Strategy Ablations (distinguishing: filter-strictness ablation / caption density-style ablation / data-ratio ablation, and corresponding evaluation metrics) ⚠️

Data ablations are broadly sparse across this collection; the vast majority of ablations target architectural components rather than data strategy, with only JavisDiT++ having a dedicated data ablation:
[JavisDiT++ — the only data quality/quantity ablation (Appendix D.2)] The paper's main text Section 4.3 cites its conclusion: "Sec. D.2 reveals that ensuring good data quality is the foundation to increase the sample quantity to improve training efficacy, providing a new insight to scale up JAVG models in the future." This conclusion's phrasing is key — it is not the simplistic "quality over quantity," but a more refined hierarchical relationship: quality is a threshold condition, and quantity is a lever above that threshold. Unfortunately, the specific experimental setup of Appendix D.2 (which scales/quality subsets were compared, and their respective metric values) could not be obtained in the public HTML version [uncertain].
- Related but independent data-decision evidence: the audio stage explicitly "applies no filtering to maximize T2A capability covering three categories" vs. the video stage applying triple quality filtering + FunASR speech removal — this "no filtering for audio, strict filtering for video" contrast is itself the team's practical judgment on the quality-quantity trade-off for the two modalities' data.
- JavisDiT (v1)'s ablation uses a 60,000-entry subset for rapid evaluation on JavisBench-mini (1,000 entries), a sampling strategy to reduce ablation cost, not a data ablation itself.
[Harmony — component ablation (not data ablation) but gives a complete synchronization-gain chain] Adding four components in sequence, with the Sync-C metric improving stepwise:
- Adding the Global-Local Decoupled Interaction (GLDI) module (baseline)
- Adding RoPE positional alignment: Sync-C 4.29 → 4.80 (+0.51)
- Adding Cross-Task Synergy training: Sync-C 4.80 → 5.09 (+0.29)
- Adding Synchronization-Enhanced CFG (inference-time): Sync-C 5.09 → 6.51 (+1.42, the largest gain)
The most valuable finding is that the inference-time synchronization-enhanced CFG contributes more gain (+1.42) than all training-time changes combined (+0.80) — showing that on the synchronization problem, inference-time guidance offers an extremely high cost-benefit ratio. But none of these are data-strategy ablations. The 1:1 speech/environmental-sound ratio on the data side is not ablated [uncertain], nor is there a controlled experiment on whether the "audio-visual consistency scoring model" filtering is effective [uncertain].
[UniAVGen — a data-scale comparison, not an ablation] The core claim of "1.3M vs. 30.1M training samples" achieving overall superiority in audio-video synchronization, timbre consistency, and emotion consistency is a cross-model comparison against Ovi, not a controlled ablation (architecture, base model, and data distribution all differ), so it can only serve as evidence of "high architectural efficiency," not as strict evidence that "small data beats large data."
[MM-Diffusion / AV-DiT] No data ablation. MM-Diffusion's ablation focuses on the design of random-shift attention (shift size, etc.).
[Common gap] None of the five performs an ablation on caption density/style, a filter-threshold sweep, or a domain-ratio ablation [uncertain].

### Evidence on Quality vs. Quantity (cases where small, curated data outperforms large, messy data) ⚠️

This collection offers two different types of "quality/efficiency-first" evidence, which should be viewed distinctly:
[Type One: a clear quality-quantity relationship claim (JavisDiT++, the most valuable)]
- Appendix D.2's conclusion is precisely worded: "ensuring good data quality is the foundation to increase the sample quantity to improve training efficacy" — this is not the common "small and curated beats large and messy," but a more refined hierarchical relationship: quality is a threshold condition, and quantity is the lever above that threshold. This is spiritually consistent with Ovi's statement that "even a small amount of unsynchronized data can harm lip-sync ability," but JavisDiT++ elevates it to a scalable principle.
- Behavioral corroboration: it actively cuts TAVGBench's 1.1 million entries down to 355,000 (a 32.3% retention rate), willing to use only three-tenths of the data in order to filter out samples with speech, low aesthetics, low motion, and excess text.
- But note: the same team makes the opposite decision on the audio side ("no filtering at all" to maximize category coverage), indicating that "quality first" is not an unconditional principle but depends on whether the bottleneck for that modality is quality or coverage — audio data is inherently cleaner and category coverage is the bottleneck, while video data is of uneven quality and quality is the primary bottleneck. This "filtering strategy differentiated by modality" mindset is one of the most nuanced data insights in this collection.
[Type Two: a training-efficiency claim (UniAVGen)] Comparing 1.3 million training samples against Ovi's 30.1 million, achieving overall superiority across audio-video synchronization, timbre consistency, and emotion consistency. But this is a cross-model comparison rather than a controlled experiment: UniAVGen uses internally curated real-human data + an asymmetric cross-modal interaction architecture + a three-stage curriculum, differing from Ovi in data distribution, architecture, and target domain (Ovi covers general scenarios, UniAVGen focuses on real humans), so this 23x data-efficiency gap is more attributable to "narrow-domain focus + architectural efficiency" than to pure data quality. It has reference value as a case of "vertical-domain small data can beat general-domain big data," but cannot be directly generalized.
[Type Three: awareness of data-source quality stratification (Harmony)] The paper explicitly labels the annotation-quality attributes of three audio data sources — AudioCaps (about 128 hours, manually annotated), Clotho (about 31 hours, manually annotated), WavCaps (about 7,600 hours, automatically annotated) — and additionally applies consistency-score filtering to the speech data. This "small curated manually-annotated set + large coarse automatically-annotated set used together" configuration is itself a practice that balances quality and quantity, but no controlled experiment on each one's contribution is given [uncertain].
[MM-Diffusion's alternative quality-first case] Using an extremely small dataset of just under 8 hours total (2.7 hours + 5.2 hours), it achieved an 80% Turing-test fooling rate as early as 2022 — relying on extreme focus in dataset selection (9 categories of strongly audiovisually-bound natural scenes + a single dance domain). This is an early success case of the "narrow-domain, high-fidelity, small data" route.
[Cautionary counterexample] The capability boundaries of all works in this collection (≤10 seconds, single-shot, fixed resolution, narrow domain) also show that: small, curated data can validate a method's effectiveness, but cannot support general-purpose capability — quality-first has its applicable limits.

### Alignment Between Training-Data Domain Distribution and Evaluation-Benchmark Category Taxonomy (e.g., VABench's seven major categories) ⚠️

Both JavisDiT and Harmony build their own evaluation benchmarks in this collection, but the degree of alignment between the two and their training-data distribution is starkly different:
[JavisDiT's JavisBench — a complete taxonomy, but deliberately misaligned with training-data distribution (an intentional "stress test" design)]
- Taxonomy: 5 evaluation dimensions, 19 scene categories — Event Scenario, Video Style, Sound Type, Spatial Composition, and Temporal Composition. This is the most systematic audio-video evaluation taxonomy in this collection, belonging to the same class of effort as VABench's seven major categories.
- Difficulty design: "over 50% of the videos belong to highly complex and challenging scenarios" and "75% of samples contain multiple sounding events" — deliberately exceeding the simplistic settings of earlier baselines (Landscape's single natural sound, AIST++'s single dance domain).
- Relationship to training data: training data comes from TAVGBench (a general YouTube distribution, with speech removed), while JavisBench mixes test sets from Landscape/AIST++/FAVDBench with new YouTube videos from June–December 2024, organized in a balanced way across the 19 categories — the two taxonomies have no correspondence, and the training side is not even ratio-controlled by the 19 categories [uncertain]. This means JavisBench is more like an "out-of-domain generalization stress test" than a "mirror evaluation of the training distribution," and the authors indeed use it to expose existing models' shortcomings in complex scenarios.
- The time-isolation design is notable: evaluation videos are taken from content uploaded June–December 2024, later than most models' training-data cutoffs, reducing training-data leakage risk — a rigorous benchmark-construction practice.
- Scale stratification: the full version of 10,140 entries is used for the main experiments, and JavisBench-mini (a random subset of 1,000 entries) is used for ablations, balancing rigor and iteration cost.
[Harmony's Harmony-Bench — training ratio and evaluation category strictly aligned (the only alignment case in this collection)]
- Three tiers of 50 entries each, 150 total: environmental sound-video (non-speech acoustic event synchronization), speech-video (lip-sync fidelity and multilingual robustness), and complex scenes (speech + environmental sound co-occurrence).
- The correspondence with training data is very direct: training data is strictly mixed at speech : environmental sound = 1:1 → the first two evaluation tiers correspond exactly to these two categories; the capability of "handling speech and environmental sound simultaneously" that the mixed training aims to cultivate → the third tier's complex-scene subset is a targeted examination of exactly this capability. This is a one-to-one mapping of "training ratio ↔ evaluation category," unique to this collection.
- The conditioning setup also varies by category: the environmental-sound subset uses audio/video captions, the speech subset mainly uses transcripts, and the complex-scene subset uses the full set of multimodal prompts — the evaluation protocol is likewise aligned with the three-field caption schema.
[JavisDiT++'s multi-dimensional metric system] 11 metrics covering 4 dimensions (quality: FVD/FAD; text consistency: TV-IB/TA-IB/CLIP-Score/CLAP-Score; audio-video consistency: AV-IB/AVHScore/JavisScore; synchronization: DeSync), with the dimension division corresponding to its architectural design (MS-MoE handles single-modality quality, TA-RoPE handles synchronization, AV-DPO handles all three in combination) — this is an "architecture-metric alignment" rather than a "data-metric alignment."
[UniAVGen] Evaluation uses a self-built 100-entry test set (50% real images + 50% AIGC/anime-style images, partially aligned with the training data's real-human focus) plus two public benchmarks, GRID (dubbing) and EMTD (audio-driven animation), organized by task rather than by content category; no category taxonomy is established [uncertain].
[MM-Diffusion / AV-DiT] No category taxonomy; evaluation is organized by dataset (Landscape / AIST++), with only FVD and FAD as metrics, using different splits of the same dataset for training and evaluation — a strictly in-domain evaluation, which is also why the generalization ability of these early works was never examined.

## Uncertain Fields

Research information for the following fields is partially uncertain (⚠️-marked sources):

- openness
- data_scale
- data_sources
- provenance_licensing
- duration_distribution
- resolution_aspect_distribution
- domain_distribution
- audio_category_distribution
- narrative_structure_distribution
- language_accent_distribution
- pipeline_overview
- funnel_retention_rate
- shot_segmentation
- quality_filtering
- motion_filtering
- deduplication
- model_as_data_judge
- safety_filtering
- caption_model
- caption_structure
- joint_av_caption_schema
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
