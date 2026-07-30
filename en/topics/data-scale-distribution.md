# Cross-comparison: Data Scale and Distribution

> Topic: Data processing for video generation models (including joint audio-video generation): data cleaning pipelines, data distribution, and annotation methods

[← Back to Home](../index.md)

This page compares all entries side by side, field by field. ⚠️ indicates that some information for that entry/field is uncertain.

**Fields**: [Training data scale (video count/hours/token count, pretraining vs. SFT separated)](#training-data-scale-video-counthourstoken-count-pretraining-vs-sft-separated) · [Data source composition (in-house/public datasets/web crawling/licensed procurement/synthetic data)](#data-source-composition-proprietary-data--public-datasets--web-scraping--licensed-acquisition--synthetic-data) · [Data compliance and provenance (share of licensed data, rights-cleared datasets, C2PA, etc.)](#data-compliance-and-provenance-proportion-of-licensed-data-rights-cleared-datasets-c2pa-etc) · [Clip duration distribution and segmentation strategy](#clip-duration-distribution-and-splitting-strategy) · [Resolution/aspect ratio distribution and bucketing strategy](#resolutionaspect-ratio-distribution-and-bucketing-strategy) · [Category/domain distribution and mixture strategy (proportion control and concept balancing for people, actions, scenes, styles, etc.)](#categorydomain-distribution-and-mixture-strategy-proportion-control-and-concept-balancing-for-people-actions-scenes-styles-etc) · [Audio category distribution and mixture (proportions and control strategy for speech/sound effects (foley)/music/ambient sound/silence) — a dimension unique to AV models](#audio-category-distribution-and-mixture-ratios-and-control-strategies-for-speechsound-effects-foleymusicambient-soundsilence-—-a-dimension-unique-to-av-models) · [Narrative structure distribution (single-shot vs. multi-shot, average clip duration, shot-count distribution, whether native audio track is included)](#narrative-structure-distribution-single-shot-vs-multi-shot-average-clip-duration-shot-count-distribution-native-audio-track) · [Language/accent distribution (the data foundation for multilingual lip-sync capability)](#languageaccent-distribution-data-foundation-for-multilingual-lip-sync-capability)

## Training data scale (video count/hours/token count, pretraining vs. SFT separated)

`data_scale` · Level of detail: brief

### [Allegro](../models/Allegro.md)

Final curated training set: 106 million (106M) images + 48 million (48M) video clips, all with strongly associated text captions.
Raw input scale: 412 million (412M) raw images + 500 million (500M) raw videos (here 500M refers to the video-clip base count after segmentation and initial screening).
Data available at each stage (the same dataset layered by increasingly strict thresholds, progressively narrowing down — not independently collected at each stage):
· T2I pretraining: 107M images (filtered from 412M raw images)
· T2V pretraining 360p: 48M video clips (from 500M)
· T2V pretraining 720p: 18M video clips (from 500M)
· T2V fine-tuning (SFT): approx. 2M high-quality video clips (from 500M)
VideoVAE separate training set: 54.7K videos + 3.73M images (requiring short side ≥720 pixels).
Actual number of samples consumed during training (including repeated sampling, distinct from dataset entry counts): T2I pretraining 700M samples; T2V Pre-train-1 (368×640, 40 frames) 87M; Pre-train-2 (720×1280, 40 frames) 21M; Pre-train-3 (720×1280, 88 frames) 8M; Fine-tune 2.6M.

### [Apollo](../models/Apollo.md) ⚠️

【Total】81 million audio-video samples with accurate dense captions — this is the only scale figure given in the paper, corresponding to the final filtered training set (original paper text: "81 million samples with accurate dense captions").
【Basis of measurement】The paper uses "sample count" as its sole unit; it gives no total hour count, no token count, and no per-sample duration, so it cannot be converted into an hour-level figure. If roughly estimated using the typical 5–10 second clip length common in similar work, 81M samples would correspond to roughly 110,000–220,000 hours, but this is an extrapolation, not data from the paper.
【Stage breakdown】The paper's three-stage training (pretraining / task-specific post-training / high-quality post-training) does not give the data scale for each stage, nor the separate scale for pretraining vs. SFT. The scale of the "manually-curated, high-quality dataset" used in Stage III is not disclosed at all.
【Category breakdown】Within the 81M total, the individual counts or proportions of the four categories — single-speaker speech, multi-speaker speech, singing, and natural sound — are not given.
【Conclusion】Apollo discloses only a single total figure for data scale, at a granularity notably coarser than contemporaneous work. [Uncertain]

### [CineDance / CineDance-1M](../models/CineDance.md) ⚠️

Released as a dataset, there is no traditional pretraining/SFT split; scale figures are given at the endpoint of the curation funnel:
【Final scale】1,021,657 narrative sequences, with a total duration of approximately 26.3K hours (26,300 hours).
【Raw collection】45,181 long videos, totaling 32.8K hours.
【Intermediate products】TransNetV2 segmented these into 25,899,474 atomic shots; narrative grouping yielded 1,201,912 narrative sequences.
【Per-sequence characteristics】Average duration 92.8 seconds, averaging 24.2 consecutive shots per sequence, minimum spatial resolution 1080p.
【Annotation volume】An average of 6,496.3 words of structured bimodal annotation per video (Tab. 6), an annotation density that leads similar datasets by an order of magnitude.
【First open release volume】The HuggingFace CineDance_01 release contains approximately 240,488 clips / 5.83 TB, a one-quarter shard.
【Model training usage】The sample counts, batch size, epochs, learning rate, and total token count used in each of the CineDance model's two training stages are not disclosed. [Uncertain]

### [CogVideoX](../models/CogVideoX.md) ⚠️

· Video pretraining: after filtering, approximately 35 million (35M) single-shot clips remain, each averaging about 6 seconds (paper's original wording: "approximately 35M single-shot clips remain, with each clip averaging about 6 seconds"), from which a total duration on the order of 58,000 hours can be estimated.
· Auxiliary image data: an additional 2 billion (2B) images, drawn from LAION-5B and COYO-700M and filtered by aesthetic score (the hyperparameter table gives a lowest aesthetic value of 4.5). During training, images are treated as single-frame video and mixed in with video training.
· High-quality fine-tuning (Stage 4 FT): a higher-quality subset selected from the above data, comprising 20% of the total data volume, trained for 10k steps.
· Captioning model training data: to fine-tune a substitute model (LLaMA2) for GPT-4 summarization, 50,000 summary data points were collected.
· Filter training data: 20,000 videos were manually labeled with positive/negative sample labels, of which 10% were randomly held out as a test set.
· The data scale of CogSound is not disclosed [Uncertain]. The data increment of CogVideoX1.5 relative to 1.0 is also not disclosed [Uncertain].

### [Cosmos-Predict2.5](../models/Cosmos-Predict25.md) ⚠️

The disclosure of pretraining and post-training scale is fairly complete, one of the most valuable parts of this report:
【Raw input】Processed over 200 million raw videos, corresponding to 35 million hours of raw video material (as a comparison, Cosmos-Predict1 used 20 million hours).
【Segmentation output】After removing clips shorter than 5 seconds, over 6 billion candidate clips were obtained, ranging from 5–60 seconds in duration.
【Final pretraining set】About 4% passed all filters, yielding approximately 200 million trainable clips — this 200M is the pretraining dataset.
【Post-training SFT scale by domain】Using a multi-head classifier trained on InternVideo2 embeddings, data was divided into five domains and counted: object permanence 10.4M, high motion 1.0M, complex scenes 1.6M, driving 3.1M, robotic manipulation 730K; plus 388K of 4K high-definition cooldown data.
【Domain-specific data】Proprietary autonomous-driving data of approximately 3.1M 20-second, 7-camera surround-view clips; the multi-view model Cosmos-Predict2.5-2B/auto/multiview was trained on 1.5 million captioned multi-view clips; Smart Spaces has approximately 40K clips; robotics datasets are detailed under data_sources.
【Training iterations】Each domain's SFT model is trained for 30k iterations at batch size 256; the RL stage uses 256 steps at batch size 32. Total pretraining token count and total iteration count are not disclosed. [Uncertain: total pretraining iterations/token count, and the source composition within the 35M hours]

### [Data-Juicer 2.0](../models/Data-Juicer.md)

Data-Juicer itself does not train models, so there is no "training data scale" per se; the relevant figures fall into two categories.
【System processing throughput】The Data-Juicer 2.0 paper reports the ability to process TB-scale data and orchestrate 10k+ CPU cores; experiments cover dataset sizes ranging from 560,000 (560K) samples to 70 billion (70B) samples; a production deployment has run stably for over 5 months, cumulatively processing over TB-scale volumes. In distributed benchmarking on 3,200 Ray-DLC cores, processing a 500x dataset took 1,780 seconds, and a 2,500x dataset took 7,083 seconds.
【Text-to-video case study】The Sandbox T2V case study is the most directly relevant data point for this survey. The raw candidate pool is composed of three public datasets, totaling approximately 1.217 million video-text pairs: InternVid 606K + Panda-70M 605K + MSR-VTT 6K. After filtering, the optimal open-sourced data pool contains 147,176 entries (approximately 227.5GB); there is also a larger 228K-entry data pool used for the final model that topped VBench (the paper notes this configuration corresponds to a training sample volume of 640K, i.e., involving data repetition). Small-scale probing experiments uniformly use a data pool of about 40K samples to control variables.
【No pretraining/SFT distinction】As a toolchain, there is no division between pretraining and SFT data volumes.

### [Foley-Omni](../models/Foley-Omni.md) ⚠️

Total training scale is approximately 2.7M audio/video-text paired samples, cumulatively about 4.9K hours of audio processed through the cleaning pipeline. Of these, the unified video-audio-text triples (v_i, a_i, Ŝ_i) produced by the data cleaning pipeline number approximately 2.0M (this portion is the direct output of this paper's pipeline; the rest are public TTS/TTA/TTM corpora used directly).
Broken down by the six task groups (Table 9):
- TTS (text-to-speech): LJSpeech + LibriTTS + internal, 1,253 hours
- TTA (text-to-audio/sound effects): AudioCaps + Freesound, 912 hours
- TTM (text-to-music): MusicCaps + MusicBench + AudioSet, 139 hours
- VisualTTS (visually-conditioned speech synthesis): Chem + GRID + LRS2 + SpeakerVid + TalkVid, 1,980 hours (the largest share)
- V2A (video-to-sound-effect): VGGSound + Kling-Foley + internal, 403 hours
- V2ST (video-to-full-soundtrack, the paper's core task): internal + SpeakerVid, 216 hours (the smallest scale, used as third-stage fine-tuning data)
Stage-by-stage view: Stage 1 uses approximately 0.7M text-audio pairs (TTA/TTS/TTM); Stage 2 expands to video-conditioned tasks (V2A/VisualTTS); Stage 3 does joint fine-tuning using the 216 hours of V2ST data plus 100 hours of replay data from each preceding single-task domain.
On the evaluation side, V2ST-Bench consists of 300 clips of 5–10 seconds.
[Uncertain] The paper does not disclose the total size of the raw video pool prior to cleaning (in hours or clip count), so the overall retention rate cannot be inferred.

### [Goku](../models/Goku.md)

【Total】The final training set comprises approximately 160 million image-text pairs (160M image-text pairs) + 36 million video-text pairs (36M video-text pairs).
【Text-to-image T2I】100M public samples (from LAION) + 60M internal high-quality samples, totaling 160M. Clear division of labor: public data is used for pretraining, internal data for fine-tuning.
【Text-to-video T2V】11M public clips + 25M in-house clips, totaling 36M clips.
【Layered by resolution】Of the 36M clips, all can be used for 480p training; of these, 24M meet the 720p threshold; 7M meet the 1080p threshold. That is, as resolution increases, the data volume shrinks progressively (36M → 24M → 7M), forming a natural resolution-curriculum data pyramid.
【Image-to-video I2V post-training】Approximately 4.5 million "text-image-video" triples.
【Not disclosed】Total video hours, token count, and raw collection volume (hence the overall retention rate cannot be inferred).

### [Hailuo / MiniMax Video](../models/Hailuo.md) ⚠️

[Uncertain] (only a single relative figure is disclosed, with no absolute numbers whatsoever). The official MiniMax Hailuo 02 release blog post (June 18, 2025) is the only public material that mentions training data scale at all; the original wording states: compared with the previous generation model, parameter count increased approximately 3x, and training data volume expanded approximately 4-fold, with quality and diversity improving simultaneously. Beyond this:
- No video count, total duration (hours), or token count has ever been published;
- Pretraining and SFT/post-training data volumes have never been distinguished;
- The base figure for the previous-generation video-01 was likewise never disclosed, so the "4x" figure cannot be anchored to any absolute value;
- The Hailuo 2.3 release announcement makes no mention of data scale at all, describing only capability improvements (body motion, stylization, micro-expressions, instruction following).
This is a textbook case of "marketing-style disclosure": a growth multiplier is given to convey the scale of investment, but no verifiable base figure is provided.

### [HunyuanVideo-Foley](../models/HunyuanVideo-Foley.md) ⚠️

【Final output】Approximately 100,000 hours (~100k hours) of text-video-audio triple (TV2A) data — this is the core figure presented as the paper's first major contribution in the abstract ("a scalable data pipeline curating 100k-hour multimodal datasets through automated annotation").
【Basis of measurement】Only an hour figure is given; the clip count is not disclosed. Estimating with a fixed 8-second clip length, 100,000 hours corresponds to roughly 45 million clips — a considerable volume, which also explains why the paper emphasizes the "scalability" of the pipeline rather than its fine-grained precision.
【No pretraining/SFT split】The entire TV2A main model has only a single training stage; the 100,000 hours of data participate in full at once — there is no pretraining vs. SFT scale split, and no annealing stage on a high-quality subset.
【Separate data for the audio autoencoder】The DAC-VAE is separately trained on "approximately 100,000 hours of audio data" for 700k steps. The paper does not clarify whether this 100,000 hours of audio is the same batch of data (the audio tracks) as the 100,000-hour TV2A set — judging by the numerical coincidence and the pipeline order, it is very likely the same source or heavily overlapping, but there is no explicit statement. [Uncertain]
【Correspondence to training compute】The main model uses 128 H20 GPUs, effective batch size 2048, trained for 200k steps; the VAE uses 32 H20 GPUs, batch size 256, 700k steps. Neither GPU-days nor total training time is disclosed.
【Scale positioning】100,000 hours is a leading-tier scale in the V2A track: as a reference, the public dataset VGGSound is only about 550 hours (as used in the paper's own account), and AudioSet is about 5,800 hours — meaning this work's self-built data scale is roughly 180x that of VGGSound. Compared with contemporaneous work such as Kling-Foley and MMAudio, this data-scale advantage is one of the main factors the paper attributes its audio quality lead to.

### [HunyuanVideo](../models/HunyuanVideo.md)

【Original HunyuanVideo (2024)】The total-scale disclosure is incomplete: the paper explicitly gives only the SFT stage's approximately 1 million (~1M) manually curated samples, along with "billions"-level images used for first-stage T2I pretraining and "hundreds of millions"-level images for the second stage. The raw pool size for video and the absolute counts/hours for each resolution tier (256p/360p/540p/720p) are not published — only relative retention ratios are given (each tier retains 1/2 to 1/5 of the previous tier).
【HunyuanVideo 1.5 (2025)】The disclosure basis is significantly more complete, making this the most valuable quantitative disclosure in this entry:
- Raw video pool: over 10 million hours (>10M hours) of raw video;
- After segmentation and filtering: approximately 800 million (800M) high-quality video clips enter pretraining;
- Subsequent stages shrink progressively: 480p stage 200M, 720p/16fps stage 100M, 720p/24fps stage 100M;
- CT (continued training) stage: T2V and I2V each use 1 million (1M) high-quality clips;
- Image side: from a pool of over 10 billion (>10B) images, 5 billion (5B) are filtered out for first-stage 256p T2I pretraining, and 1 billion (1B) for the second stage at 512p;
- Super-resolution module training data: 1 million high-quality video clips (1K–4K resolution) + high-resolution images.
The 1.5 report does not give exact sample counts for the SFT stage or the RLHF stage (only describing the selection criteria).

### [InstructAV2AV](../models/InstructAV2AV.md) ⚠️

【Final dataset scale】Per the paper: InsAVE-80K contains 79K automatically verified training pairs + 1K manually curated evaluation pairs, totaling approximately 80K (the origin of the dataset's name).
【Discrepancy from the actually released version】The HuggingFace dataset card shows that the actual release contains 88,074 pairs (176,148 files, since each pair includes a source and a target copy), of which 87,074 pairs are for training and 1,000 for evaluation, totaling about 139 GB across 11 tar shards. That is, the actual released volume exceeds the paper-reported 79K by about 8K pairs (roughly 10% more), presumably because the pipeline continued running and expanding after the paper was submitted, or possibly because the paper's figure was rounded. The HF actual release numbers should be used as authoritative.
【Per-sample specifications】Uniformly 5-second clips, 720p resolution, 24 FPS, 16 kHz audio. Converting at 79K × 5s, the training set totals about 110 hours (source side); including the target side as well, about 220 hours. This scale is extremely small relative to the tens to hundreds of thousands of hours typical of text-to-video pretraining, but it is reasonable for the post-training-oriented task of "editing" — the model's generative capability comes from the Ovi pretraining prior, and InsAVE-80K's role is solely to teach the model "how to edit according to instructions."
【Pretraining vs. SFT split】The paper does not split InsAVE-80K into pretraining/SFT portions. The true pretraining data is the upstream training corpora of Ovi and Wan2.2 (not covered or disclosed in this paper); InsAVE-80K as a whole plays the role of a fine-tuning set for the editing task.
【Evaluation scale】The InsAVE-80K evaluation set has 1,000 pairs (manually curated); there is also zero-shot evaluation on the external AvED-Bench. The user study randomly draws 20 samples from each dataset.
[Uncertain] The paper does not disclose the size of the raw video pool prior to data synthesis (in hours or count), nor how many candidate samples the data engine actually generated before filtering down to 79K — so neither the absolute throughput nor the selection intensity of the entire pipeline can be quantified.

### [2026 Miscellaneous Joint Audio-Video Generation Works](../models/JAVG_2026_misc.md) ⚠️

The scale of the various works spans more than two orders of magnitude, with inconsistent disclosure conventions:
【ALIVE (ByteDance, the largest scale and the most finely broken down by stage)】Gives sample counts by training stage rather than hours:
- Audio T2A Stage I: 384M audio samples (corresponding to 700k hours of transcribed speech, used for TTS pretraining)
- Audio T2A Stage II: 19M samples (approximately 5k hours of high-quality speech + 111k hours of sound-annotated video dataset)
- T2VA joint training: 11M samples @480p/24fps (1.2 epochs)
- T2VA+I2VA: the same 11M samples (0.3 epochs)
- Continue-training: 4.3M balanced samples (3 epochs)
- SFT: 5M samples (0.5 epochs)
- 1080p Refiner: 0.7M high-clarity samples (1 epoch)
- Character-driven: 0.8M reference-paired samples
The paper describes this as "continue pretraining and finetuning on million-level high-quality data." The total size of the raw candidate pool is not disclosed [Uncertain].
【NAVA (Baidu)】The only work that gives both ends of the full funnel: raw collection of "approximately 20M audio clips and 100M video clips," filtered down to "around 15M clips for large-scale training" used for large-scale training; within this, the Koala-36M subset accounts for about 20% of the final corpus. The SFT stage further converges to 160K high-quality samples. Average video duration is about 7 seconds (estimated at approximately 29,000 hours based on 15M × 7s [this conversion is an inference, uncertain]).
【CCL (SenseTime lineage)】Focused on small data: Table 1 explicitly lists its own training data volume as 4M, comparing it side by side with Ovi's 30.7M and MOVA's 50M — i.e., using about 1/8 of Ovi's data and about 1/12 of MOVA's. The main text describes it as "million-level audio-video pairs."
【Baton】1.5 million video-audio clips, original text: "Our training dataset (1.5 million video-audio clips) is aggregated from OpenHuman-Vid, AudioCaps, WavCaps, and videos collected from the internet." — among the seven works, one of the smallest training scales for joint generation.
【OmniCustom】Self-built OmniCustom-1M: approximately 1 million single-person video clips, totaling 2,500 hours, filtered from SpeakerVid-5M (whose raw scale is "more than 5.2 million video clips" / "8,000 hours"). That is, clip-level retention is about 19%, and duration-level retention is about 31% (2500/8000).
【StreamChar】No total figure is given, only sources (SpeakerVid-5M + TalkVid + OpenHumanVid combined) and training step counts (orchestrator 80k steps @batch 640; joint training 100k steps @batch 128). Speech pretraining uses the Emilia dataset. It explicitly constrains that "training data contains no videos/transcripts longer than 20 seconds" [total scale uncertain].
【ITS-JAVG】No training data (training-free); "data volume" is instead reflected in the inference search budget: JavisDiT samples 5 samples per prompt (Best-of-N), and MMDisCo samples 10 samples per prompt.

### [Joint Audio-Video Generation Baseline Collection](../models/JavisDiT_baselines.md) ⚠️

The data scale of the various works spans four orders of magnitude, making this collection the best illustration of the "academic baseline → industrial baseline" progression:
【MM-Diffusion (2022) — tens-of-thousands-of-frames, hour-level】
- Landscape: 928 source YouTube videos → cut into 1,000 non-overlapping 10-second clips, totaling about 2.7 hours, approximately 300,000 frames.
- AIST++: 1,020 street-dance video clips, totaling 5.2 hours, approximately 560,000 frames, paired with 60 copyright-cleared dance tracks.
- The two datasets combined total less than 8 hours — a typical "small-scale, single-domain, high-fidelity" academic setting.
【AV-DiT (2024) — reuses the same two datasets】Scale is identical to MM-Diffusion (AIST++ 1,020 clips/5.2 hours, Landscape 1,000 clips/2.7 hours), with no new data introduced; trained for 100,000 iterations at batch size 16.
【JavisDiT (2025) — million-level entries】
- First-stage audio pretraining: 780,000 (0.78M) audio-text pairs, trained for 55 epochs (recorded as 50 epochs in JavisDiT++).
- Second-stage ST-Prior training: 610,000 (0.6M) synchronized "text-video-audio" triples, plus synthesized asynchronous negative samples.
- Third-stage JAVG training: the same 610,000 triples, fine-tuning the cross-attention and bidirectional attention modules.
- Ablation experiments use a 60,000 (60K) subset for rapid evaluation on JavisBench-mini.
【JavisDiT++ (2026) — approximately 1 million public entries】
- Audio: 780,000 audio-text pairs (reusing the JavisDiT collection), 50 epochs, training an audio FFN with 794M parameters in total.
- Video: TAVGBench's raw 1.1 million triples → 355,000 after filtering, of which 330,000 are used for audio-video SFT (2 epochs, LoRA 121M parameters) and 25,000 for AV-DPO (1 epoch, LoRA 121M parameters).
- The authors emphasize that all data are "public training entries," and that achieving SOTA at the roughly 1-million-entry scale represents "beating a large model with small data."
【Harmony (2025) — 4 million+ clips】
- A total of "over 4 million audio-video clips," covering two major categories: human speech and ambient sound.
- Human speech side: aggregated from Emilia, OpenHumanVid, and SpeakerVid, then filtered by an audio-video consistency scoring model down to 2 million high-quality clips, each 3–10 seconds long.
- Ambient sound side: AudioCaps (about 128 hours, manually annotated) + Clotho (about 31 hours, manually annotated) + WavCaps (about 7,600 hours, automatically annotated) + a self-collected 2 million audio-video clips rich in ambient sound.
- Stage-one audio pretraining: 100,000 iterations, global batch size 1536; stage two: 20,000 iterations; stage three (cross-task joint training): 10,000 iterations, batch size 128.
【UniAVGen (2025) — 1.3 million samples, emphasizing "high efficiency with less data"】
- The paper's core selling point is achieving overall superiority in audio-video synchronization, timbre consistency, and emotion consistency using "1.3M training samples vs. the comparison method's 30.1M" (this comparison figure is taken from Ovi's joint training sample count).
- Stage 1: the English subset of the multilingual Emilia audio dataset, 160,000 steps (160k steps), batch 256, lr 2e-5.
- Stage 2: internally collected real human audio-video data, 30,000 steps, batch 32, lr 5e-6.
- Stage 3: multi-task learning, 10,000 steps, with a task-type mixture ratio of 4:1:1:2:2 across five task categories.
- Whether 1.3M refers to the internal dataset's entry count or the cumulative sample count across all stages is not clarified by the paper [Uncertain].

### [Kling 3.0 / Kling Video 3.0 Omni](../models/Kling_30_Omni.md) ⚠️

[Uncertain] Official sources have never disclosed the training data scale of Kling 3.0 Omni (video count/hours/token count); neither pretraining nor SFT scale is disclosed. The Kling-Omni technical report only qualitatively states "large-scale real-world data collection + task-oriented synthetic data construction," with no numbers. Circumstantial reference points from the same team's other public data include: the open-sourced Koala-36M dataset, 36 million clips, averaging 13.75 seconds, at 720p (about 137,000 hours); and Kling-Foley's TV2A training data of 122,000 hours / 55 million 8-second clips, totaling over 100 million video-audio-text triple samples. Kling 3.0 Omni's actual scale should significantly exceed these publicly known datasets, but there is no public basis for this.

### [LTX-2](../models/LTX-2.md) ⚠️

Completely undisclosed. Section 5, "Training Data," of the technical report totals only two paragraphs (about 150 words), giving no video count, total duration, or token count, and not distinguishing between pretraining and fine-tuning scale. It only qualitatively states that it uses "a subset of the same dataset employed in LTX-Video," a subset focused on "video clips containing significant and information-rich audio components." The prior LTX-Video paper likewise gives no data scale figures. [Uncertain]

### [LongCat-Video](../models/LongCat-Video.md) ⚠️

[Uncertain]. The technical report gives no training data scale figures anywhere in the text — no video count, total duration (hours), token count, or image count, and no scale for either pretraining or SFT. The only indirectly derivable figures are training iteration counts and batch configurations: the five pretraining stages total approximately 678k iterations (Stage 1 T2I 256p 285k + Stage 2 140k + Stage 3 164k + Stage 4 36k + Stage 5 53k); the SFT stage is 7.5k iterations (480p+720p × 93 frames, lr 1e-5); the RLHF (GRPO) stage is approximately 0.5k iterations (group size 4, 64 prompts per step). Avatar 1.5 likewise does not disclose total data volume (Stage 1: 256p × 93 frames, batch 64, 130k iterations total; Stage 2: 480p × 93 frames, batch 32, 45k iterations total). The evaluation set size is the only figure given with certainty: an internal T2V set of 1,228 entries (500 manually evaluated + 728 automatically evaluated, covering 48 categories), and an I2V set of 400 entries (100 first-frame reference images × 4 prompt categories).

### [MOVA](../models/MOVA.md)

Broken down by training stage (the paper gives both "hours" and "clip count" as dual units, but no token count):
【Audio tower pretraining】Uses three major domains — WavCaps + VGGSound (general sound effects), JamendoMaxCaps (music), and in-house TTS data — training on fixed-length clips; specific hour counts/clip counts are not disclosed.
【Joint training Phase 1 (360p, diverse data)】Approximately 61,500 hours of video-audio data, 1 epoch, taking 15 days.
【Joint training Phase 2 (360p, quality-filtered)】Approximately 37,600 hours / 16.8M clips (16.8M × 8.05s ≈ 37,560 hours, self-consistent), 1 epoch, taking 7 days.
【Joint training Phase 3 (720p, highest-quality subset)】Approximately 11,000 hours, 1 epoch, taking 20 days.
【Total】The three stages combined take 42 days, using 1,024 GPUs (128 nodes × 8 cards), roughly 43,000 GPU-days.
Note: MOVA has no independent SFT/RLHF post-training stage, so there is no "pretraining vs. SFT" scale split; instead, there is a decreasing data scale across the three-stage progressive curriculum (61.5k → 37.6k → 11k hours), forming a typical pyramid of "decreasing scale, increasing quality."

### [Merged entry: ① Mochi 1](../models/Mochi_MAGI_Motif.md) ⚠️

① Mochi 1: Completely undisclosed. Official sources have never given video count, hour count, or token count, only stating the model was "trained entirely from scratch" and was, at the time, "the largest openly released video generation model." [Uncertain]
② MAGI-1: Only the raw material scale is given — "from tens of petabytes of raw videos and images collected from a wide range of sources" (tens of petabytes of raw video and images). The entry counts, hour counts, and per-stage sample volumes of the cleaned training set are not disclosed; Table 5 only states that "data volume decreases across stages" without giving numbers. Pretraining and SFT are also not measured separately (the report has no independent SFT stage; see post_training_data). [Uncertain]
③ Motif-Video 2B: Gives a clear, deliberately understated upper bound — "fewer than 10M clips" and "less than 100,000 H200 GPU hours." The HF card states "approximately 10 million video clips." The paper contrasts this with contemporaneous open-source models' "hundreds of millions of curated clips" (Wan2.1, HunyuanVideo, Seedance, etc., which use hundreds of millions of clips and 5B–14B parameters), claiming its own data volume is an order of magnitude lower and its parameter count 7x smaller. The SFT corpus scale is not given as an absolute number, only described as a "curated high-quality subset" iteratively supplemented by subject category (Fig. 8 gives distribution proportions rather than absolute quantities). [Partially uncertain: absolute scale of the curated SFT set]

### [Movie Gen](../models/Movie_Gen.md) ⚠️

Pretraining (video): O(100)M video-text pairs + O(1)B image-text pairs; the raw video pool ranges from 4 seconds to 2 minutes in duration (averaging 28 seconds), and after cleaning each clip is a 4–16 second single-shot segment.
Post-training (video SFT): a small, manually curated collection of high-quality videos with human-written captions; the paper does not give a specific entry count [Uncertain]; training uses only 512 H100 GPUs (64 nodes), two orders of magnitude smaller than pretraining's scale of up to 6,144 H100 GPUs.
Personalization (PT2V): O(1)M single-person videos are filtered out from the pretraining set; O(10)M paired samples are sampled, along with O(10)K real cross-paired samples and O(1)M synthetic cross-paired samples; the SFT set consists of O(1000) high-quality single-person videos.
Audio pretraining: a total of O(100)M samples / O(1,000)K hours (i.e., million-hour scale), of which the Sound category alone accounts for O(100)M / O(1,000)K, while Music, Sound+Music, Sound+Voice, and Sound+Music+Voice each account for O(10)M / O(100)K.
Audio fine-tuning: cinematic-grade audio-video O(100)K samples / O(1)K hours; high-quality pure-audio (music O(10)K hours + sound effects O(10)K hours) O(1,000)K samples / O(10)K hours.

### [NVIDIA NeMo Curator](../models/NeMo_Curator.md)

The framework itself contains no data; here we record its "validated processing scale" and "designed capacity."
【Designed capacity】The official recurring claim is the ability to "efficiently clip, annotate, and filter 100 PB or more of videos."
【Measured scale (Cosmos WFM production run)】Input of approximately 20 million hours (20M hours) of raw video, at resolutions of 720p–4K; output of approximately 100 million (100M) clips of 2–60 seconds each; of these, on the order of 10^8 are used for pretraining and on the order of 10^7 for fine-tuning.
【Processing time】Processing 20 million hours of video: about 40 days on a Hopper (H100) GPU cluster; about 14 days on Blackwell; under equivalent conditions, an unoptimized CPU pipeline would require approximately 3.4 years. Another official figure states "using 1,000 GPUs achieves an 89x speedup over an unoptimized CPU pipeline at ISO power, compressing processing time from years to days."
Note: these figures reflect the operating scale of NVIDIA's own Cosmos project, not general-purpose figures for NeMo Curator users at large.

### [OmniHuman Dataset + OHBench](../models/OmniHuman.md) ⚠️

【Core scale】1 million videos, totaling 1,800 hours, covering 80,000 distinct identities. These three figures constitute the complete scale disclosure given in the paper, more complete than most comparable work (which often gives only a clip count).
【Derived average clip duration】1,800 hours ÷ 1,000,000 clips ≈ 6.48 seconds/clip — that is, the data is predominantly made up of short clips on the order of 6 seconds, consistent with the training window (5–10 seconds) of current AV joint generation models. This derived value is corroborated by the duration-distribution chart in Figure 3(d), but the paper's main text does not directly state the average duration.
【Identity density】80,000 identities / 1,000,000 clips ≈ an average of 12.5 clips per identity — identity diversity is significantly higher than traditional talking-head datasets such as HDTF and CelebV-HQ (which typically cover only hundreds to a few thousand identities). This is a key advantage for the model's identity-generalization ability, and one of the differentiating points the paper emphasizes in Table 1.
【Pretraining/SFT split】The dataset itself does not distinguish between pretraining and SFT — it is a general-purpose training resource, and the split is left to the user. The paper's own validation experiments used only 20% of it (180,000 single-person + 20,000 two-person = 200,000 samples) to fine-tune LTX-2, an SFT-style usage (see data_ablation for details).
【Evaluation subset】OHBench comprises 509 videos total: 331 single-person, 128 two-person, 50 human-object interaction. This subset was selected from OmniHuman, and the paper explicitly states there is a domain gap relative to the training set — i.e., out-of-distribution samples were deliberately selected to avoid the benchmark being contaminated by the training set, a commendable detail in the benchmark's construction.
【Not disclosed】The raw collection volume (the total before filtering) is not given at all, so the 1-million figure has only a numerator and no denominator (see funnel_retention_rate for details); token count is not applicable; the exact single-person/two-person/three-category breakdown within the 1 million clips is given only graphically in Figure 3(b), with no values in the main text. [Uncertain]

### [Open-Sora Series](../models/Open-Sora.md) ⚠️

【Open-Sora 2.0】Gives data volume by training stage (the paper's table convention is "the number of videos participating in training at that stage"): Stage 1: 70M videos (256px T2V); Stage 2: 10M videos (256px T2V/I2V); Stage 3: 5M videos (768px T2V/I2V). The report does not give the total raw video volume before cleaning, total hours, or token count, nor does it list SFT scale separately (Stage 3's 5M high-definition subset is, in effect, the high-quality fine-tuning set).
【Open-Sora 1.2】The raw pool is approximately 30M video clips (2–16 seconds in duration), totaling approximately 80K hours; the Panda-70M high-quality subset within it is 20M clips, approximately 41K hours; the final high-quality stage is approximately 2M clips, 5K hours. Images total approximately 3M.
【Open-Sora Plan v1.3】Images: 18.0M; videos: approximately 28M (before cleaning); after cleaning, the Panda70M portion retains approximately 19M (a 27% retention rate).
【Open-Sora Plan v1.5】Images: 1.1B (only resolution-checked, not quality-filtered); videos: 40M high-quality samples.
Note: all of the above figures are entry-count-based; neither project publishes training token counts, and Open-Sora 2.0 does not publish total hour counts. [Uncertain]

### [Ovi](../models/Ovi.md) ⚠️

The paper gives only order-of-magnitude descriptions, not precise numbers.
【Audio-video paired corpus】"Millions of videos," entirely in-house audio-video corpora; estimating at 121 frames @24fps ≈ 5.04 seconds per clip, if there were 3 million clips this would correspond to roughly 4,200 hours, but the paper does not confirm the entry count, so this conversion is an inference [Uncertain].
【Pure-audio corpus】In the pretraining stage, "hundreds of thousands of hours of raw audio," predominantly human speech, with waveform lengths of up to 12 seconds; in the fine-tuning stage, fixed-length 5.04-second waveforms are used, mixed with public sound-effect data from VGGSound / AudioSet / WavCaps plus audio tracks extracted from the internal audio-video corpus.
【Estimation from training step counts】Audio pretraining: 50k steps × batch 2880 ≈ 144 million sample instances; audio-video fusion training: 40k steps × batch 768 ≈ 30.72 million sample instances (for reference in epoch conversion, not a data entry count).
【Ovi 1.1】The README explicitly states "Dataset includes 100% more videos," i.e., the scale of the audio-video dataset was doubled relative to the initial version, and training switched to native 960×960-resolution data; absolute values are not published [Uncertain].
The strict split of pretraining vs. SFT data is not publicly disclosed [Uncertain].

### [Script-a-Video](../models/Script-a-Video.md)

The data scale disclosed in the paper is divided by purpose into two independent sets — "captioning side" and "generation side":
【Captioning side (used to train the MTSS annotation model)】500K high-quality video clips (500,000 clips). Only the entry count is given; total hours, average duration, and token count are not given. A rough estimate at typical 5–10 second clip lengths would give 700–1,400 hours, but the paper provides no duration-basis data at all, so this is an inference, not a disclosure.
【Generation side (used to train the LTX-2-derived model)】Four datasets used across stages:
- ID Customization stage: a 400K identity-centric dataset, trained for 3 epochs;
- Multi-shot Control stage: 250K multi-shot sequences, trained for 1.5 epochs;
- Audio-Visual Synergy stage: 870K cinematic pairs, trained for 3 epochs;
- Final joint fine-tuning stage: 60K high-fidelity cinematic alignment pairs + 250K multi-shot sequences, interleaved and mixed, trained for 15K steps.
The generation-side data totals approximately 1.5M in scale (including cross-stage reuse), again giving only entry counts, no hour counts.
【Evaluation side】An internal evaluation set of 125 single-shot samples + 100 multi-shot samples, 225 total.
【Pretraining vs. SFT split】On the captioning-model side there is no pretraining — SFT is done directly, in a single stage, on the open-source Qwen3-Omni-Instruct, and the 500K entries constitute the entirety of the SFT data; on the generation side, likewise, multi-stage fine-tuning is performed on the already-pretrained LTX-2, with no from-scratch pretraining involved.

### [Seedance 2.0 and Seedance 1.5 pro](../models/Seedance_20_Seedance_15_pro.md) ⚠️

[Uncertain] Neither report discloses any training data scale figures (video count, hours, token count; neither pretraining nor SFT is given). After a full-text search, neither the Seedance 1.5 pro (v1/v3) nor the Seedance 2.0 reports contain any scale expressions such as million/hours/minutes. The claim circulating online of "approximately 100 million minutes (~100 million minutes) of in-the-wild audio-video clips" originates from AI auto-summary sites such as emergentmind, which claim to cite this paper — but the original text contains no such figure. This is an unverified secondhand inference and should not be trusted. The Seedance 1.0 report likewise gives no absolute data volume.

### [SkyReels Series](../models/SkyReels.md)

【SkyReels-V2】The raw dataset scale is on the order of O(100M) (approximately 100 million video samples), paired with an O(100M)-scale concept-balanced image dataset for joint training. The self-collected media is estimated to total "6.2M+ hours" (over 6.2 million hours), sourced from "280,000+ movies" and "800,000+ TV episodes" from over 120 countries. The captioning model SkyCaptioner-V1's training set is "approximately 2 million concept-balanced videos curated from an initial 10 million samples." In the post-training stage, concept balancing "resulted in a 50% reduction in data volume."
【SkyReels-V4】Gives data volume by training stage (Table 1): Stage 1 text-to-image: 3 billion images (3 epochs); Stage 2 text/image + video: 1 billion images + 400 million videos (3 epochs); Stage 3 inpainting task: also 1 billion images + 400 million videos (2 epochs); Stage 4 mixed resolution: 100 million each (2 epochs); Stage 5 high resolution 480/720/1080p: 50 million each (2 epochs); Stage 6 multimodal conditioning: 20 million images + 50 million videos (2 epochs); audio backbone pretraining: "hundreds of thousands of hours," with each clip up to 15 seconds (3 epochs); audio-video joint training: uses "50% video data + T2A data" (2 epochs); SFT first stage: 5 million videos (20% carrying multimodal conditions); SFT second stage: 1 million manually curated high-quality videos. Total video hours and token count are not given.

### [Sora 2](../models/Sora_2.md) ⚠️

Completely undisclosed. The System Card gives no video count, total hours, token/patch count, no distinction between pretraining and SFT scale, and no disclosure of the compute budget. OpenAI has no official data-scale figures externally either. [Uncertain]

### [SpeakerVid-5M](../models/SpeakerVid-5M.md)

The paper gives figures in three units — "clip count + hours + speaker ID count" — but no token count:
【Raw collection】153K audio-video videos, totaling 64,386 hours of raw material.
【Final dataset total】Over 5.2M (5.2 million) video clips, over 8,743 hours.
【Single-person branch (including talking and listening)】5.2M clips / 8.7K hours / 83K unique speaker IDs.
【Dialogue branch】770K clip pairs / 1.8K hours / 16K unique speaker IDs.
【High-quality SFT subset】571K clips / 1,368 hours.
【Pretraining vs. SFT split】The baseline model's pretraining uses 7,375 hours (i.e., 8,743 − 1,368, the remaining large-scale data after removing the HQ subset); SFT/fine-tuning uses the high-quality subset of 571K clips / 1,368 hours. This is a clearly defined "large-scale pretraining + small, refined post-training" two-stage scale design, with the HQ subset accounting for about 15.6% of total hours.

### [Step-Video-T2V](../models/Step-Video-T2V.md)

【Total pretraining volume】The technical report explicitly states: a large-scale dataset was constructed containing 2 billion (2B) video-text pairs and 3.8 billion (3.8B) image-text pairs. This is one of the most solid quantitative disclosures in this entry.
【Actual samples consumed per training stage】
· Step-1 text-to-image (T2I) pretraining: 256px, approximately 3.8B image samples, 253k iterations;
· Step-2 text-to-image/video (T2VI) joint pretraining, low-resolution tier: 192×320, approximately 1 billion (1B) video clips available, actual samples seen 644 million (644M), 430k iterations;
· Step-2 high-resolution tier: 544×992, actual samples seen 27.3 million (27.3M), 46k iterations;
· Step-3 text-to-video (T2V) fine-tuning / SFT: 544×992, approximately 30 million (30M) high-quality videos;
· Step-4 Video-DPO: 544×992, constructing preference pairs based on the model's own self-generated videos; the distillation (Turbo) dataset is approximately 95,000 (95k) samples.
【Notes】The raw video pool's hour count, the pre-segmentation clip count, and the absolute elimination counts at each filtering level are not given in the report (the funnel is presented as relative bar charts in Figure 11, with no absolute values labeled). Total video duration (hours) and token count are not disclosed.

### [UniTalking](../models/UniTalking.md) ⚠️

【Final output】After the full pipeline of filtering, the person-centric dataset totals 2.3 million (2.3 million) aligned audio-video samples. This is the only data scale figure given in the paper.
【Basis of measurement】Only "sample count" is given; no hours, no total duration, no token count, and no raw collection volume are given — so it can neither be converted to hours for horizontal comparison with other work, nor can the retention rate be inferred. If roughly estimated using the typical 5-second clip length of the Wan2.2-5B base model, 2.3 million entries would correspond to roughly 3,200 hours, but the paper gives no clip-duration basis, so this estimate is unsupported. [Uncertain]
【Pretraining vs. SFT split】Training is indeed done in two stages, but the two stages use different datasets rather than a coarse/fine split of the same dataset:
- Stage 1 (audio branch pretraining): uses "internal TTS data," with scale, language, and duration all undisclosed; batch size 256, learning rate 1e-5, 100,000 steps total. [Uncertain]
- Stage 2 (audio-video joint training): uses the above-mentioned 2.3 million audio-video entries; batch size 64, learning rate 1e-5, 100,000 steps total.
【Derived data】Beyond the 2.3 million main entries, 3 reference audio clips are synthesized for each video using IndexTTS2, producing an additional approximately 6.9 million synthetic reference audio clips (each 3–5 seconds).
【Scale positioning】2.3 million entries is a relatively large scale in the speaker-generation direction, but the paper's Future Work section explicitly acknowledges that "model performance is constrained by available training resources and data scale, especially compared with closed-source models" — i.e., the team itself acknowledges that data volume remains a bottleneck.

### [UniVerse-1](../models/UniVerse-1.md)

Total 7,685 hours of audio-video data (the paper's body text and model card in multiple places round this to "approximately 7,600 hours"), all used in a single fine-tuning stage — there is no pretraining/SFT scale split, because UniVerse-1's "pretraining" is simply inheriting the weights of two existing expert models, Wan2.1 and Ace-step, with only a single round of joint fine-tuning performed on top.
【Three components and their respective hour counts】
- Rigorously verified speech-centric data: 1,187 hours;
- General-purpose audio-video data: 3,074 hours;
- Public datasets VGGSound + AudioSet: 3,422 hours.
【Basis of measurement】Only hour counts are given; clip counts and token counts are not disclosed, nor is the raw collection volume (hence the overall retention rate cannot be inferred).
【Training volume】Effective batch size 128, gradient accumulation over 4 steps, 50k steps total, AdamW, learning rate 5e-6, FSDP multi-node distributed training. GPU count and GPU-days are not disclosed.
【Scale positioning】7.6K hours is a small-to-medium scale among comparable work — about 1/8 of MOVA Phase 1 (61.5K hours), and close to the scale of MOVA Phase 3's high-quality subset (11K hours). The paper's Limitations section explicitly cites compute constraints as the main reason, and lists "larger-scale, more finely curated data" as future work.

### [Unison](../models/Unison.md)

The paper's entire disclosure of data scale is concentrated in a single sentence in Section 4.1, split into audio-video and pure-audio portions, and explicitly labeled as the final output "after refinement through our automated processing pipeline":
【Audio-video joint training data】Approximately 2 million synchronized audio-video clips, totaling over 3,000 hours.
【Pure-audio training data】50 million high-quality audio clips, totaling over 130,000 hours.
【Derivable average clip duration】
- Audio-video side: 3,000 h × 3600 s ÷ 2,000,000 ≈ 5.4 seconds/clip, a short-clip-dense corpus, on the same order of magnitude as UniVerse-1's approximately 5-second window and MOVA's fixed 8.05-second length;
- Pure-audio side: 130,000 h × 3600 s ÷ 50,000,000 ≈ 9.4 seconds/clip.
(The above are estimates derived by this entry from the public figures; the paper itself does not give average durations.)
【Scale positioning】3,000+ hours of audio-video data is small-scale among comparable work — about 0.4x of UniVerse-1 (7,685 hours), 1/20 of MOVA Phase 1 (61.5K hours), and 2–3 orders of magnitude below million-hour-scale industrial corpora such as Sora 2/Veo 3. However, the audio side's 130,000-hour scale is quite considerable, approaching the corpus scale of dedicated TTS/audio-generation foundation models — this extreme "audio-heavy, video-light" mixture ratio is a direct consequence of the training strategy: during the joint training stage, the video backbone is completely frozen, so Unison effectively only trains the audio branch and the fusion module, with all visual capability inherited from Wan2.2-5B, hence there is no need for large-scale video corpora.
【Pretraining/SFT split】Not applicable. The paper's two-stage division is "Stage 1: single-modality training of the audio branch" and "Stage 2: joint fine-tuning" — the former consumes 130,000 hours of pure audio, the latter consumes 3,000+ hours of audio-video data — a split by modality rather than by pretraining/SFT. There is no independent SFT stage and no post-training data.
【Not disclosed】The raw collection/aggregation total (the denominator), the hour and entry contributions of each source dataset, token count, the number of training steps and epochs for Stage 2, and the corpus's duration histogram.

### [Veo 3 / Veo 3.1](../models/Veo_3_Veo_31.md) ⚠️

[Uncertain] Official sources have never disclosed any training data scale — no video count, no hour count, no token count, and no distinction between pretraining and SFT scale. The technical report states only that "We train on a large dataset comprising images, videos, and associated annotations." Third-party blog claims such as "billions of audio-video pairs" or "millions of hours of paired audio-video" have no official source and should not be trusted. A weak circumstantial indicator is that training uses Google TPU Pod clusters, JAX, and ML Pathways, hinting at a scale on the order of the largest in the same period.

### [Vidu S1](../models/Vidu_S1.md) ⚠️

[Uncertain]. The technical report gives no absolute training data scale at all (no video count, hours, or token count), nor does it distinguish between pretraining and post-training data volume. It is only qualitatively described as "a corpus of high-quality, diverse, and highly interactive single-person, single-shot video." The usage volumes for each of the three training stages (bidirectional teacher training / causal autoregressive adaptation / DMD distillation) are likewise not disclosed.

### [Wan 2.5 / 2.6 / 2.7](../models/Wan.md) ⚠️

2.5/2.6/2.7 are completely undisclosed. All available scale information comes from previous generations and the same family:
- Wan 2.1: The candidate pool is "billions of videos and images"; after basic-dimension filtering, about 50% is eliminated; no hour count or token count is given, and pretraining vs. SFT absolute volumes are not distinguished.
- Wan 2.2: The README explicitly gives a relative increment — compared with Wan 2.1, "images +65.6%, videos +83.2%," this is the only cross-version quantitative data-scale comparison in the Wan family; the official claim is that this increment significantly improved generalization in motion, semantics, and aesthetics.
- Wan 2.1 post-training set: curated images at the "millions" scale; on the video side, "millions of simple-motion clips" and "millions of complex-motion clips" were separately selected.
- Wan 2.1 V2A audio subset: after rigorous filtering from the video generation dataset, only O(1) thousand hours remain — this is the only audio-side scale figure in the Wan family, indicating that audio-visual usable data is an extremely small subset relative to video data.
- Wan 2.1 person-personalization subset: an internal portrait classifier first filters out approximately O(100)M videos, ultimately constructing approximately O(10)M personalized videos (each averaging 5 attached segmented face images), plus a synthesized O(1)M face-swap videos.
- Wan-Dancer (2026.07): a self-built collection of approximately 200 hours of high-quality dance videos (≥720p/30fps).
Starting with 2.5, newly added capabilities — voice, lip sync, multi-shot narrative, role-play — necessarily entail substantial expansion of audio-visual and multi-shot data volume, but no numbers are given. [Uncertain]

### [Audio-Video Generation Evaluation Benchmark Collection](../models/av_benchmarks.md)

All five are evaluation/meta-evaluation scale figures, far smaller than training-set scale, though AVBench and PhyAVBench come with substantial accompanying data assets:
【VABench】A total of 1,299+ test cases: T2AV 778 prompts, I2AV 521 (including paired images), of which 116 are dedicated to stereo audio; each sample is accompanied by 3–7 audio question-answer pairs (AQA) and 3–7 visual question-answer pairs (VQA).
【AVBench】Evaluation set of 470 ≥720p high-definition prompts (Normal subset 350, 1–2 speakers; Hard subset 120, 3–4 speakers + overlapping speech + background noise). The evaluator's training set is significantly larger: 30K real clips are drawn from OpenHumanVid, expanded into 100K pairs per dimension, totaling 300K supervised samples across three consistency dimensions (all preference pairs with hard negatives).
【AV-SyncBench】3,269 in-the-wild videos, expanded into 38,390 evaluation samples (temporal challenge: 37,569 samples / 2,717 videos; semantic challenge: 821 samples / 552 videos; of which 592 are human voice timbre substitution, 229 are instrument timbre transfer).
【PhyAVBench】The dataset PhyAV-Sound-11K: 11,605 newly recorded videos, totaling 25.5 hours; 337 sets of controlled paired prompts; 184 participants appearing on camera/operating; each set of paired prompts is accompanied by an average of approximately 17 ground-truth real recorded videos (the paper requires N≥20 for mean-noise reduction, with an actual average of 17).
【Omni-Judge】300 prompts (drawn from the VidProM real-user prompt library), each generating 1 video from Sora 2 and 1 from Veo 3, totaling 600 generated videos, paired with human ratings across 600×9 dimensions.

### [Video Captioning Model Ecosystem](../models/caption_models.md) ⚠️

Here we distinguish two figures — "the captioner's own training data scale" and "the annotation volume the captioner produces" — which must be strictly separated:
【Captioner training data (pretraining/SFT listed separately)】
· Tarsier2-7B: pretraining on 40M video-text pairs (a 3.6x expansion from Tarsier1's 11M); SFT stage performs fine-grained temporal alignment (the paper cites approximately 150K human-precision-annotated entries, including event timestamps) [scale figures partially uncertain]; followed by preference pairs automatically constructed via model-based sampling + DPO.
· ShareCaptioner-Video: SFT data = 40K dense captions annotated by GPT-4V (DiffSW format), base model InternLM-XComposer2-4KHD. Achieves large-scale inference capability with an extremely small SFT volume — a typical example of "distilling from a small amount of high-quality teacher data."
· AuroraCap: three-stage training, cumulatively over 20 million high-quality image/video-text pairs.
· SkyCaptioner-V1: approximately 2 million concept-balanced videos (curated from 10 million, a 20% retention rate), trained on 32 A800 GPUs, global batch size 512.
· AVoCaDO: AVoCaDO-SFT 107K (TikTok-10M 24K + ShortVideo 18K + Shot2Story 20K + FineVideo 29K + YouTube-Commons 11K + CinePile 5K), followed by GRPO.
· AVSCap: AVSCap-130K = 40K videos × 3 annotations each (visual / audio / synergistic omni-modal), followed by GRPO; the paper explicitly states "the gain from RL exceeds the gain from expanding SFT data volume."
· CogVLM2-Caption: dense caption data produced by a teacher chain (of which 50,000 GPT-4 summary samples are used to fine-tune the LLaMA2 summarizer); the total volume on the student side is not disclosed [Uncertain].
· video-SALMONN 2: specific entry counts not disclosed [Uncertain], model sizes 3B/7B/72B.
【Captioner-produced annotation scale (downstream usable volume)】
· ShareCaptioner-Video → 4.8 million high-quality aesthetic video annotations, one of the largest single-model outputs in the open-source community.
· Tarsier2 → recaptions 1 million public dataset videos, releasing 585K of them (Tarsier2-Recap-585K).
· Panda-70M → captions for 70 million clips, though the captions are extremely short (averaging 13.2 words).
· Koala-36M → 36 million clips, captions averaging 202.3 words (about 15x longer than Panda-70M).
· The implicit output volume on the generation side is even larger: Movie Gen's entire clip set is annotated by LLaMa3-Video (70% from the 8B model, 30% from the 70B model); Apollo/Klear performs multi-model annotation (including Gemini-2.5-Pro calls) on 81 million samples; Harmony uses Gemini to annotate 4 million audio-video clips.
【Ecosystem-level observation】Caption length spans a 60x range — from Panda-70M's 13.2 words to the longest at 824.2 words (see the pretraining_datasets entry for details); the choice of captioning model directly determines caption length, which in turn determines the downstream T2V model's sensitivity range to prompt length.

### [Geometric/Structured Annotation Dataset Collection](../models/geometric_datasets.md)

SceneScribe-1M: approximately 1 million video clips, 156.7M frames, totaling 4,000+ hours; SpatialVID: raw 33,443 YouTube long videos (21,789 hours), producing 2.71 million clips, 127.60M frames, 7,089 hours of dynamic content, with the high-quality subset SpatialVID-HQ at 0.37M clips, 20.63M frames; WildWorld: over 108 million frames, approximately 1,800 hours (converted at 30FPS), with 119 columns of annotation per frame; Action100M: sourced from 1.2 million (1,199,096) instructional videos totaling approximately 14.6 years in duration (of which 72%, i.e., 10.6 years, has usable ASR), producing 147 million temporally-localized segments and approximately 21.3 billion English words of annotation. All four are single datasets that do not distinguish between pretraining and SFT splits (Action100M provides a semantic resampling subset to mitigate long-tail effects).

### [Video Generation Post-Training Data Strategies](../models/post_training_data.md) ⚠️

【Anchor paper】No figures disclosed at all: the entirety of Section 4.1, "Dataset," consists of only two sentences: "First, we constructed a high-quality text-video dataset for SFT. Subsequently, we curated the prompt set as described in the experimental settings." The SFT set entry count, the RLHF prompt set entry count, group size N, training step count, and GPU scale are all missing. This is the most notable point to highlight in this entry: a report titled around a "post-training system framework" discloses zero information on the data-scale dimension.
【Cross-comparable landscape of curated SFT-set scale (benchmarkable public figures)】
· Step-Video-T2V: approximately 30 million high-quality videos;
· Cosmos-Predict 2.5: broken down across five domains — object permanence 10.4M, driving 3.1M, complex scenes 1.6M, high motion 1.0M, robotic manipulation 730K, plus 4K cooldown data 388K, plus driving multi-view 1.5M segments of 7-camera 20-second 30FPS clips;
· SkyReels-V4: SFT in two stages — 5 million videos with multimodal conditions (3 epochs) → 1 million manually curated (3 epochs);
· ALIVE: Continue-Training 4.3M → SFT 5M (0.5 epochs) → 1080p Refiner 0.7M → character-specific 0.8M;
· Open-Sora 2.0: Stage 3 5M high-resolution curated set; Open-Sora 1.2: 2M clips / 5K hours; Open-Sora Plan: I2V Stage 2 15M;
· Goku: I2V fine-tuning 4.5M text-image-video triples (12.5% of the 36M video pool);
· Allegro: approximately 2M (0.4% of the 500M raw clips);
· HunyuanVideo original: approximately 1 million manually curated; HunyuanVideo 1.5: 1 million per task for CT, SFT scale undisclosed;
· CogVideoX: the top-20% highest-quality subset of the pretraining data, 10k steps;
· NAVA: curated 160K from 15M (retention rate approximately 1.07%), and re-captioned this subset using the more expensive Gemini-3-Pro;
· Movie Gen: video SFT set scale not disclosed but "training uses only 512 H100 GPUs"; PT2V SFT set O(1000) entries; audio SFT set — cinematic split O(100)K samples / O(1)K hours + high-quality audio split O(1,000)K samples / O(10)K hours;
· Motif-Video 2B: two rounds of SFT (480p Stage 7, 720p Stage 10), scale not given.
【Landscape of preference-data scale】HPDv3: 1.08 million text-image pairs / 1.17 million pairwise comparisons; VideoReward: 16K prompts / 108K videos / 182K annotated triples (of which 13K triples are held out as a validation set, whose prompts do not appear in the training set); SkyReels-V2: 30K manually annotated pairs used to train a Bradley-Terry reward model + 20K × 3 stages ≈ 60K DPO data total per stage; JavisDiT++: approximately 25K audio-video preference pairs (prompt pool of 30K, 1 epoch, LoRA 121M trainable parameters); HunyuanVideo 1.5: the T2V-side RLHF prompt set is on the order of O(10K)+.
【Scale pattern】The typical scale of curated SFT sets is 10^6–10^7 entries, accounting for 0.4%–20% of the pretraining pool; the typical scale of preference data is 10^4–10^5 pairs, 2–3 orders of magnitude smaller than the SFT set — consistent with the proportional structure of LLM post-training.

### [Combined Survey of Mainstream Video Pretraining Datasets: Panda-70M, InternVid, Koala…](../models/pretraining_datasets.md) ⚠️

The seven datasets span four orders of magnitude in scale, and **scale and quality are notably inversely ranked** (listed below in descending order of clip count, based on each paper's own Table 1; UltraVideo's Table 1 provides a unified cross-validation):
1) **InternVid**: 7.1M source videos → **234M clips**, 760.3K hours, captions totaling 4.1 billion words, averaging 11.7 seconds/clip and 17.6 words/caption; source videos average 351.9 seconds in duration. Subsets: InternVid-10M-DIV (diversity-sampled 10 million), InternVid-10M-FLT (taking the **top 30%** by UMT-SIM score on top of DIV), InternVid-18M-AES (aesthetic score ≥4, 18 million).
2) **Panda-70M**: 3.79M source long videos → **70,817,169 clips** (per the paper) / **70,723,513** (actual release count — the roughly 94,000-clip difference is attributed to harmful-content filtering before release, not explicitly stated by the authors, an inference), 166.8K hours, approximately 36TB, averaging **8.477 seconds**/clip and **13.2 words**/caption. Released subsets: 10M (matching_score > 0.43 and at most 3 clips per source video, 10,473,922 clips / 37.0K hours), 2M (exactly 3 clips per source video drawn from the 10M set, 800K × 3 = 2,400,000 clips / 7.56K hours), and validation/test sets of 2,000 source videos × 3 clips each. **Note: the paper's "Panda-2M/Panda-5M" are random subsets, distinct from the quality-filtered 2M/10M versions released in the repository — these are easily confused when cited.**
3) **Koala-36M**: **36M clips**, 172K hours (Table 1), averaging 13.75 seconds/clip and **202.1 words**/caption, at 720p. **This table has an internal-consistency issue: 36M × 13.75s = 137.5K hours, about 25% off from the stated 172K hours** (when UltraVideo's Table 1 cites this, it uses 137K hours/13.6 seconds, i.e., adopting the product-based figure). Separately, the paper's abstract repeatedly says "over 10M" — that is a threshold statement used when comparing against MiraData/VidGen/OpenVid (all ≤1M), not a second count.
4) **LVD-2M**: **2M clips** (the release contains approximately 2.1M rows), each ≥10 seconds, averaging 20.2 seconds/clip and 88.7 words/caption, average optical-flow score 47.8. **Total duration is not given in the paper**; estimated at approximately **11,200 hours** using 2M × 20.2s (UltraVideo's table cites this as 14.6K hours/20.2 seconds/2.1M). Duration distribution: 10–15s about 43.5%, 15–20s about 23%, 20–30s about 20.5%, 30–50s about 11%, >50s about 2.5%.
5) **OpenVid-1M**: **1M clips**, of which **OpenVidHD-0.4M = 433K clips at 1080p**; per UltraVideo's Table 1: OpenVid-1M averages 7.2 seconds/clip, 2.1K hours, 126.5 words/caption; OpenVidHD averages 9.6 seconds/clip, 1.2K hours, 104.5 words/caption. The HF dataset card shows "1.45M rows" (including duplicate counting of the HD subset). **Total duration/average duration/FPS are not disclosed in the original paper**.
6) **MiraData**: unfiltered pool of **788K**, releasing four nested subsets of **330K / 93K / 42K / 9K**. Average clip duration **72.1 seconds** (the longest of the seven), caption length **318 words**, at 720p. **Table 1's "16K hours" corresponds to the unfiltered pool of 788K (788K × 72.1s ≈ 15,785 hours), not the released 330K version — at the same basis, 330K corresponds to approximately 6,600 hours**, an easily misattributed point. (The externally circulated claim of "77k long videos" is a misattribution of the v0 beta's 57,803 clips/1,754 hours — do not use.)
7) **UltraVideo**: short split — **42,184 clips / 62 hours / averaging 5.3 seconds/clip / 824.2 words/caption**; long split — **16,597 clips / 143 hours / averaging 30.9 seconds/clip / 850.3 words/caption**. Source videos number only **5,000**. About 2,700x smaller than Koala-36M by hour count, but captions are about 4x longer and pixel volume 4–16x higher. **Note: its split naming is short/long — there is no "UltraVideo-1K/42K"; 1K/4K instead refers to the output resolution of the UltraWan model.**
## Data source composition (proprietary data / public datasets / web scraping / licensed acquisition / synthetic data)

`data_sources` · Level of detail: brief

### [Allegro](../models/Allegro.md) ⚠️

Primarily sourced from public datasets; the paper explicitly states that "existing public datasets such as WebVid, Panda-70M, HD-VILA, HD-VG, and OpenVid-1M provide a solid foundation for data sourcing." On this basis, the team carried out its own complete re-splitting, filtering, and re-captioning to build its own corpus of 106M images + 48M videos, rather than directly using the original captions and splits of the source datasets.
It does not disclose the use of proprietary/purchased data, exclusive copyrighted libraries, or synthetic data, nor does it give the specific mixture ratio of the various source datasets. The source of the 412M raw images on the image side is not named. [Uncertain] (proportion contributed by each source, whether mixed with in-house web scraping)

### [Apollo](../models/Apollo.md) ⚠️

The paper's disclosure of data source composition is essentially blank: it lists no public dataset names, does not state the respective proportions of proprietary data, web scraping, and licensed acquisition, and does not mention the data acquisition channels. Limited inferences that can be drawn:
- Data modalities cover four scenario types — single-speaker speech, multi-speaker speech, singing, and natural sound — and require native synchronized audio tracks (since the entire pipeline is built on quality and sync filtering of the audio tracks of existing videos, rather than dubbing synthesis onto silent video), indicating the data comes from real videos with sound rather than silent video with audio added afterward.
- The scale of 81 million clips, together with Kuaishou's identity as a short-video platform, strongly suggests that platform-owned/licensed short-video corpora are the primary source, but the paper makes no statement on this.
- Synthetic data: the paper does not use any model-synthesized audio/video content. The only "synthetic" component is that all captions are automatically generated by ASR/audio captioners/video expert models (synthetic annotation, not synthetic content).
[Uncertain]

### [CineDance / CineDance-1M](../models/CineDance.md) ⚠️

Primarily a combination of secondary curation of public datasets plus self-collection; no licensed acquisition or synthetic data is mentioned:
[Public dataset sources] Explicitly lists reuse of three existing large-scale video datasets — MiraData, LVD-2M, and Koala-36M — as part of the material pool.
[Collection process reference] States that it followed the data collection pipeline conventions of SkyReels-V2 and OpenHumanVid for web material collection.
[Content type] Predominantly film/television/narrative long-form video (cinematic), emphasizing "feature-film-style" multi-shot narrative content, so the raw material is dominated by feature-length content rather than UGC short video — the raw set of only 45,181 videos accounts for 32.8K hours, averaging about 43 minutes per clip, confirming the material is feature-film-grade content.
[Hard admission criteria] Minimum 1080p spatial resolution, and must carry a native audio track.
[Synthetic data] None. All material is genuine captured footage; no synthetic or edited samples were constructed.
[Specific proportion of each source] The respective contribution proportions of MiraData / LVD-2M / Koala-36M / self-collected material are not given. [Uncertain]

### [CogVideoX](../models/CogVideoX.md) ⚠️

The paper only generally states that the data is video from the internet ("Videos from the Internet usually include a significant amount of low-resolution ones"), without disclosing the specific channels, procurement, or licensing methods; judged to be primarily web scraping + internal data pool [specific composition uncertain]. Confirmed portions:
· On the image side, two public datasets are explicitly used: LAION-5B and COYO-700M (2B taken after aesthetic-score filtering).
· In the captioning stage, public video captioning datasets/models are borrowed: Panda-70M's caption model is used to produce short captions (the paper also criticizes the captions of Panda70M, COCO Caption, and WebVid as too short and not comprehensive, so their text is not used directly).
· Part of the caption training data is synthetic: produced by frame-by-frame CogVLM image captioning + GPT-4 summarization, which constitutes "model-generated synthetic text."
· No mention of licensed/purchased data, film/TV asset libraries, or synthetic video data.

### [Cosmos-Predict2.5](../models/Cosmos-Predict25.md)

The composition is "proprietary datasets + public internet platforms + public robotics datasets + NVIDIA internal collection," but the general pretraining portion is only described qualitatively, with no proportions given:
(1) General pretraining: "sourced from both proprietary datasets and open internet platforms," covering domains such as driving, object manipulation, spatial navigation, human interaction, and nature scenes; the proportion of the two source types is not disclosed.
(2) The robotics domain is a named collection of public datasets, with counts given by camera viewpoint (center view/left view/right view): AgiBot-Beta (dual-arm) 194k/30k/30k, Bridge (single-arm) 36k, DROID (single-arm) 39k (wrist view)/51k/51k, GR00T (dual-arm) 3k, 1X Technologies (dual-arm) 17k, OpenX (single-arm) 500, RoboMIND (dual-arm/humanoid) 16k/6k/7k.
(3) Autonomous driving is NVIDIA's own collection: "a proprietary dataset ... collected using NVIDIA's internal driving platform," approximately 3.1M 20-second surround-view clips, with 7 synchronized cameras (front-wide, front-tele, left, right, rear, rear-left, rear-right).
(4) Smart Spaces (warehouses/factories/construction sites, etc.): candidate videos are first retrieved using search keywords, then a VLM (the Qwen2.5-VL series) judges relevance clip by clip; after splitting and filtering, about 40K clips survive.
(5) Synthetic data: actively excluded during the pretraining stage (game footage, synthetic visual patterns, animation, and cartoons are all removed); but in downstream applications, Cosmos-Transfer2.5 is used to generate synthetic augmented data to train robot policies (see synthetic_data_synthesis).

### [Data-Juicer 2.0](../models/Data-Juicer.md)

As a tool, Data-Juicer itself does not hold data; its data-source questions manifest in two places.
[Data sources in official case studies] The text-to-video case studies use exclusively public datasets, with no proprietary data, no web scraping, no licensed acquisition, and no synthetic data: InternVid (Shanghai AI Lab, a large-scale YouTube-sourced video-text dataset, contributing 606k), Panda-70M (Snap/UCM, a high-quality video-caption dataset derived from HD-VILA-100M, contributing 605k), MSR-VTT (Microsoft's classic small-scale video description dataset, contributing 6k). The three together total about 1.217 million entries; the mixture shows InternVid and Panda-70M contributing nearly equal shares, with MSR-VTT serving as only a small supplement (0.5%).
[Data access surface supported by the system] Natively compatible with HuggingFace Datasets, ModelScope datasets, the local filesystem, Alibaba Cloud OSS/NAS/CPFS, and AWS S3 (since v1.4.4), among other data sources; supports compressed dataset formats (v1.5.1) and video byte-stream I/O (v1.4.6, enabling video processing without writing to disk).
[Synthetic data capability] Of the 229 operators, about 50 are dedicated to data synthesis and augmentation; the DJ-Cookbook includes a "video data synthesis" (video-data-synthesis) YAML recipe — i.e., DJ can not only filter real data but also supports constructing synthetic data, which differs from NeMo Curator's focus on cleaning.

### [Foley-Omni](../models/Foley-Omni.md)

Mixed sourcing, with two major categories — public datasets + internal proprietary data — and no mention of large-scale web scraping or licensed acquisition.
[Public datasets] Speech: LJSpeech, LibriTTS (TTS foundation); GRID, LRS2, Chem (classic lip-reading/visual-speech datasets); SpeakerVid, TalkVid (newer large-scale talking-person video datasets, used for both VisualTTS and V2ST). Sound effects: AudioCaps, Freesound, VGGSound, Kling-Foley (a foley dataset released by Kuaishou's Kling team). Music: MusicCaps, MusicBench, AudioSet.
[Internal data] Appears in TTS (internal speech library), V2A (internal audio-video corpus), and V2ST (internal data, the main body of the 216 hours) — this is the main object acted on by this paper's data-cleaning pipeline. The paper describes this portion as "weakly labeled audiovisual data," i.e., raw material with only the original video + original audio track, lacking component-level text annotation.
[Synthetic/constructed data] None. The three-field annotation generated by Gemini 2.5 Pro constitutes "model annotation" rather than "synthetic audio-video data"; the pipeline does not involve manually perturbed constructed training pairs.
Notably, the data sourcing is strongly task-oriented: VisualTTS accounts for 1,980 hours (about 40%), showing the model invests most heavily in the "speaker video → speech" pathway, directly corresponding to its headline advantage in speech intelligibility (WER 7.59, in some comparisons even approaching the Ground Truth's 8.03).

### [Goku](../models/Goku.md)

A mixture of four categories:
(1) Public datasets — on the image side, LAION (Schuhmann et al., 2022, 100M samples); on the video side, Panda-70M, InternVid, OpenVid-1M, and Pexels are explicitly listed, totaling 11 million clips.
(2) Proprietary/internal data — 60 million "high-quality internal samples" on the image side, and 25 million proprietary clips on the video side (about 69% of the video data, the dominant source). The specific source channel of the internal data (whether from the Douyin/TikTok ecosystem, whether licensed acquisition) is not stated in the paper.
(3) Web scraping — not explicitly stated, but the public datasets themselves (LAION, Panda-70M, InternVid) are all web-scraped sources.
(4) Synthetic data — the paper does not use synthetic video data for training; GPT-4o is used only in the evaluation phase to rewrite GenEval short prompts (an evaluation-side, not training-data-synthesis, use).
Overall this presents the classic ByteDance-style recipe of "public data as the base + internal high-quality data for fine refinement."

### [Hailuo / MiniMax Video](../models/Hailuo.md) ⚠️

[Uncertain]. MiniMax has never publicly disclosed the data-source composition of the Hailuo video model, and has not stated the proportions or presence of proprietary data, public datasets, web scraping, licensed acquisition, or synthetic data. Indirect circumstantial evidence (none official, low credibility):
- The Hailuo 02 blog post claims the training data improved in "quality and diversity," implying an expansion and filtering of data sources occurred, but names no source;
- Hailuo 2.3 explicitly expanded support for styles such as ink wash painting, game CG, anime, and illustration, and there was an early Live2D/anime-specialized model, video-01-live — suggesting the training data contains a substantial proportion of anime/game-style material, which is common among Chinese vendors;
- The model's emphasis on real-person micro-expressions and live-action facial performance suggests a large amount of real film/TV and talking-head material.
The above are only inferences about the data distribution reverse-engineered from capabilities, with no first-hand evidence.

### [HunyuanVideo-Foley](../models/HunyuanVideo-Foley.md) ⚠️

The paper's disclosure of data sources is extremely restrained — this is the biggest information gap on the data side of this work:
[Self-built TV2A data (100,000 hours)] Described only as "collected audio-visual data," without stating whether it is web scraping, Tencent's own content library (Tencent Video/Weishi, etc.), commercial procurement, or a mix of the three; nor does it give any platform names or a content-source list. Given that Tencent itself possesses large-scale video content assets, and that the paper deliberately avoids describing the source, it is speculated that proprietary/licensed content libraries dominate, supplemented by web collection, but there is no evidence to support this. [Uncertain]
[Explicitly named public datasets (used only for evaluation or VAE evaluation, not primary training)] VGGSound (the paper mentions about 550 hours, used as the VGGSound-Test evaluation), AudioCaps, WavCaps, AudioSet (used to evaluate DAC-VAE reconstruction quality), Song Describer (music reconstruction evaluation), LibriTTS (speech reconstruction evaluation). These datasets mainly appear in the evaluation stage in the paper, and it is not explicitly stated that they were incorporated into the 100,000-hour primary training set.
[Synthetic data] No model-synthesized audio or video content is used. The only "synthetic" component is that all captions are automatically generated by the GenAU model (synthetic annotation, not synthetic content).
[Licensed acquisition] Not mentioned.
[Comparison with similar work] UniVerse-1 explicitly lists a YouTube content-type inventory and Pexels; MOVA also gives its source composition; HunyuanVideo-Foley discusses the source not at all, discussing only the processing pipeline — this is a strategy of "disclosing the method, hiding the raw material," which is fairly common in work led by commercial companies.

### [HunyuanVideo](../models/HunyuanVideo.md) ⚠️

Both generations give only qualitative descriptions, without disclosing the proportions of the source composition. The original version states that the raw data pool covers multiple domains including people, animals, plants, landscapes, vehicles, objects, and buildings/animation, with the source unspecified (implicitly a mix of web scraping + proprietary/licensed asset libraries). The 1.5 report states that "videos come from a variety of channels, ensuring comprehensive coverage in content, cinematography, camera movement, style, and scene," again without distinguishing proprietary/public datasets/scraping/purchased/synthetic. Neither generation uses synthetic video data as primary training corpus (not mentioned). On the image side, 1.5 explicitly reuses the data acquisition and processing pipeline of HunyuanImage-3.0. [Uncertain]

### [InstructAV2AV](../models/InstructAV2AV.md)

The composition follows a "public datasets + web scraping → controlled synthesis" two-stage structure — this is the most fundamental difference between this work and the other subjects surveyed: the target side of the final training data is entirely model-synthesized product, not genuine captured footage.
[Stage one: real material sources (used as source and synthesis base)]
  · Web platform scraping: YouTube is explicitly mentioned (the paper phrases it as publicly accessible online platforms, e.g., YouTube).
  · Four public datasets: MovieBench (Wu et al., 2025b, a film-level long-video understanding dataset), Condensed Movies (Bain et al., 2020, a collection of key movie clips), Short-Films-20K (Ghermi et al., 2024, a short-film dataset), VGGSound (Chen et al., 2020, about 310,000 10-second in-the-wild audio-visual event clips across 310 sound-source classes).
  The orientation of the source composition is clear: the three film/short-film sources provide narrative, dialogue-driven, professionally mixed high-quality material (supporting speech editing and identity-preservation tasks), while VGGSound provides material with clear sound-source events (supporting sound-effect instance editing/insertion/removal tasks). This "film + event" dual configuration maps one-to-one to its four editing task types.
[Stage two: synthetic data (target side)]
  Target videos are synthesized by a self-built mask-guided editing model (based on Wan2.2-5B); target audio is produced by SAM-Audio separation + ElevenLabs text-to-audio/speech synthesis, mixed with the original background sound. Thus in each pair, the source is real and the target is synthetic, with instructions generated by Qwen3-Omni.
[Licensed acquisition] None. No paid licensing or rights-cleared data acquisition is mentioned.
[Proprietary data] None. No internal private data is mentioned.
[Methodological significance] This approach of "anchoring on real material, using a generative model to produce controlled variation" is a general paradigm for solving the problem of nonexistent paired supervision data: there is no such thing in the real world as "two genuine recordings of the same scene before and after editing" — it can only be synthesized. The cost is that the target side inherits the capability ceiling and distortion patterns of Wan2.2-5B and ElevenLabs; the editing effect the model learns cannot exceed the data engine itself — this is an inherent ceiling of the synthetic-data route, which the paper does not discuss as a risk.

### [2026 Miscellaneous Joint Audio-Video Generation](../models/JAVG_2026_misc.md) ⚠️

Shows a clear two-stage pattern of "reuse of public datasets + proprietary/scraped supplementation," with OpenHumanVid and SpeakerVid-5M becoming the two de facto standard upstream sources in the 2026 JAVG field:
[Baton] A four-way mix of OpenHuman-Vid + AudioCaps + WavCaps + internet-collected videos, with no description of licensed acquisition or synthetic data.
[OmniCustom] Single upstream source: SpeakerVid-5M (a public dual-person interactive human-generation audio-video dataset), from which OmniCustom-1M is filtered using self-built rules. The evaluation set additionally includes 70 YouTube clips + 30 real people not present in the training set.
[StreamChar] A combination of three public human/talking-video datasets: SpeakerVid-5M (large-scale, high-quality dual-person interactive human generation with audio-video), TalkVid (large-scale, diverse audio-driven talking-head synthesis), and OpenHumanVid (large-scale, high-quality human-centric video generation); the speech side uses Emilia, a large-scale multilingual speech dataset. All are public data, making this the most "academically reproducible" of the seven works in terms of data sourcing.
[ALIVE] No specific dataset is named; described as a proprietary large-scale corpus first filtered for "videos with audio" from a "raw data pool." The audio side includes 700k hours of transcribed speech (suspected to be ByteDance's internal Seed speech corpus [uncertain]), 5k hours of high-quality speech, and a 111k-hour video dataset with sound annotations. There is also a large amount of synthetic/augmented construction (see synthetic_data_synthesis). Overall dominated by internal corpora.
[CCL] OpenHumanVid (public) + in-house collections, with the latter explicitly covering three categories: "interviews, short dramas, and films" — a fairly specific source profile, indicating a bias toward "narrative content with people talking"; audio pretraining additionally introduces the academic datasets WavCaps and VGGSound.
[NAVA] A three-way composition: Koala-36M (a public large-scale video dataset, accounting for about 20% of the final corpus) + TED-style speech videos (a high-quality single-speaker speech source) + raw movie/TV footage. This is the only one of the seven works to explicitly state "raw film/TV footage" as a source.
[ITS-JAVG] No training data; evaluation uses the VGGSound test set and JavisBench-mini.
[Common observations] None of the seven discloses licensed/purchased data, none uses C2PA-style provenance tracking, and four (Baton/CCL/NAVA/ALIVE) all contain internet self-collected material or raw film/TV footage — this is where compliance risk concentrates.

### [Joint Audio-Video Generation Baseline Collection](../models/JavisDiT_baselines.md) ⚠️

Shows a three-stage evolution of "self-built small datasets → assembling public datasets → mixing public + internal data":
[MM-Diffusion] (1) Self-built Landscape: the authors scraped 928 natural-scene videos from YouTube and split them themselves — the only dataset in this collection scraped from scratch and publicly released; (2) the public dataset AIST++: a subset of the AIST dataset, street-dance videos paired with 60 copyright-cleared dance tracks, naturally avoiding copyright risk. Zero-shot transfer experiments also involve AudioSet-type data [uncertain].
[AV-DiT] Purely a reuse of public datasets: AIST++ and Landscape, with no proprietary data, no scraping, and no purchased data.
[JavisDiT / JavisDiT++] Almost entirely an assembly of public academic datasets, which is key to its reproducibility:
- The audio side (780,000 entries) comes from 10 public collections: AudioSet, AudioCaps, VGGSound, WavCaps, Clotho, ESC50, GTZAN, MACS, UrbanSound8K, and MusicInstrument — covering the three major categories of general sound effects, music, and speech.
- The video side comes from TAVGBench (a public benchmark of 1.1 million text-audio-video triplets, underlying it YouTube videos).
- Preference data (DPO) is a mix of model self-generated and ground-truth data, constituting synthetic/bootstrapped data.
- The JavisBench evaluation set is collected from a mix of "test sets of existing datasets (Landscape / AIST++ / FAVDBench) + YouTube videos uploaded between June and December 2024."
[Harmony] A mix of public + self-collected data:
- Public: Emilia (a TTS-dedicated speech corpus), OpenHumanVid (person-centric video), SpeakerVid (dual-person interactive human generation data), AudioCaps, Clotho, WavCaps.
- Self-collected: an additional 2 million audio-video clips rich in ambient sound (the paper calls these "newly collected"; the acquisition channel is not disclosed) [uncertain].
[UniAVGen] A mix of public + internal data, with internal data as the core:
- Public: the English subset of the Emilia multilingual audio dataset (used only for stage-one audio pretraining).
- Internal: an "internally collected real human audio-video dataset," which carries the entirety of stage-two and stage-three training; source, scale, and collection method are all undisclosed [uncertain]. Given the Tencent Hunyuan background, it is speculated to be related to its internal video/livestreaming corpora, but there is no textual basis for this.

### [Kling 3.0 / Kling Video 3.0 Omni](../models/Kling_30_Omni.md) ⚠️

Qualitatively disclosed as a mix of three categories: (1) automated large-scale internet video/image mining — the report states that it uses a self-developed embedding model to retrieve semantically related cross-modal samples from massive internet data; (2) proprietary/purchased film-grade high-quality material — both Kling-Omni and KlingAvatar 2.0 emphasize "cinematic-level video data," and KlingAvatar 2.0 explicitly mentions podcasts, interviews, and multi-character film/TV drama as sources of multi-speaker dialogue data; (3) a synthetic-data pipeline driven by expert models — using self-developed image-editing models and video-understanding models to reverse-construct editing/multi-image-reference training pairs. Additionally, Kuaishou owns a proprietary short-video platform content pool, which in theory is an important source, but the official has not confirmed its use in training Kling. [Uncertain: the specific proportions of proprietary platform data, licensed acquisition, and scraping]

### [LTX-2](../models/LTX-2.md)

The composition is a mix of "publicly available data + licensed acquired material." The LTX-Video paper states verbatim: "Our training dataset comprises a robust collection of publicly available data, supplemented with licensed material." LTX-2 directly reuses an audio-informative subset of this data pool.
There is clear public corroborating evidence for the licensed sources: (1) Shutterstock — announced in December 2024, Lightricks is the first global partner to use Shutterstock's "research license" to train an open-source model, gaining access to its HD and 4K video asset library; (2) Getty Images — a strategic partnership was established in 2025 during development of the 13B model, gaining access to a high-quality video asset library. In addition, image datasets are mixed into training (LTX-Video explicitly treats images as one "resolution-duration combination" participating in training, to introduce concepts uncommon in video data). No use of synthetic data is stated. The proportion of each source is not disclosed.

### [LongCat-Video](../models/LongCat-Video.md) ⚠️

The report only states in general terms that "We collect raw video data from a variety of sources," without distinguishing the composition of proprietary business data/public datasets/web scraping/licensed acquisition, and without naming any specific dataset. It is confirmed to include two major categories, image data and video data (Stage 1 is pure T2I image training; from Stage 2 onward, image and video are mixed). The SFT stage additionally "incorporated specialized datasets" to strengthen instruction-following ability, specifically targeting camera motion and visual style categories, with the source unstated.
The same series' Avatar 1.5 discloses six categories of data source divided by function, which can serve as corroborating evidence of the team's approach to data organization: (1) close-up facial videos (for facial modeling and lip motion); (2) interview-type videos (stable subject, clear speech); (3) performance-type videos (providing cinematography and pose variation); (4) interaction-type videos (object handling and gestures); (5) music videos (singing and rhythmic movement); (6) animated and stylized videos (for non-photorealistic domain generalization). The proportion of each source is not disclosed. [Uncertain: specific source composition and mixture ratio]

### [MOVA](../models/MOVA.md)

Two major sources: high-quality subsets of public datasets + a large amount of in-house data.
[Public datasets (all using their filtered HQ subsets)]
- VGGSound (an audio-video event alignment dataset)
- AutoReCap (large-scale audio-generation data)
- ChronoMagic-Pro (time-lapse/morphing-type video)
- ACAV-100M (an automatically curated large-scale audio-visual representation learning dataset)
- OpenHumanVid (a person-centric video generation dataset)
- SpeakerVid-5M (a dual-person interactive human-generation audio-video dataset, the core source of lip-sync capability)
- OpenVid-1M (a high-quality text-to-video dataset)
[In-house data] The paper states "a large amount of in-house data." The data sources explicitly listed for Phase 1 are: SpeakerVid5M, Chinese drama, cartoon, movies, YouTube, and OpenHumanVid.
[Content modalities and themes] Video modalities cover movies, vlogs, and animation; themes cover education, sports, beauty, news, interviews, animation, etc.
[Synthetic data] No model-synthesized video-audio training data is used; the only "synthetic" component is the in-house TTS data used in audio-tower pretraining, plus the fact that all captions are automatically generated by an MLLM/LLM (synthetic annotation, not synthetic content).
[Licensed acquisition] The paper does not mention any paid licensed/purchased data.

### [Merged entry: ① Mochi 1](../models/Mochi_MAGI_Motif.md) ⚠️

① Mochi 1: The official statement is a single sentence — Paras Jain told VentureBeat, "Generally, we use publicly available data and sometimes work with a variety of data partners," and declined to elaborate for competitive reasons. It names no public dataset and states no scraping/licensing proportions. [Uncertain]
② MAGI-1: Described only as "collected from a wide range of sources," without distinguishing proprietary scraping, public datasets, licensed acquisition, or synthetic data, and naming no source dataset. From "tens of PB of raw material + a self-built scalable data-processing system + PySceneDetect splitting long videos," it can be inferred that proprietary large-scale web scraping is the primary source, but the paper does not state this explicitly. [Uncertain]
③ Motif-Video 2B: Explicitly two sources — "an internal web-scale video collection" and "a set of publicly available image and video datasets" — both sharing the same downstream sanitation/filtering/deduplication/staged quality-control pipeline, ensuring the final corpus is governed by a single standard. In addition, it explicitly splits the raw pool into four branches — Image Real / Image Synthetic / Video Real / Video Synthetic — i.e., it acknowledges using synthetic data, and stipulates that "Synthetic video is injected only at 720p," reasoning that its controllable quality best matches the admission standard for that stage. However, it names no specific public dataset and gives no mixture ratio across sources. [Partially uncertain: the specific list of public datasets and their mixture ratio]

### [Movie Gen](../models/Movie_Gen.md) ⚠️

The paper does not disclose the specific source channels or licensing method of the data, describing it only as "a large pool of videos" / "sourcing data from a large volume," which can be judged to be Meta's own large-scale internal video/audio data pool [source composition uncertain]. The cleaning strategy for image-text data follows Meta's own Emu (Dai et al., 2023) approach. Confirmed compositional features: the raw video covers multiple domains including humans, nature, animals, and objects; audio pretraining data comes from the native audio tracks of the videos; audio fine-tuning additionally uses high-quality video-free pure music and pure sound-effect asset libraries (professionally produced material). Synthetic data is explicitly used for personalization (a personalized image-generation model producing reference images) and video editing (affine-animated image-editing pairs, generative instruction segmentation, and backtranslation reverse pairing). No public academic dataset is used as the main training body (VGGSound and similar are used only for evaluation).

### [NVIDIA NeMo Curator](../models/NeMo_Curator.md) ⚠️

As a toolchain, it is "source-agnostic" and does not itself dictate data sources. Supported data-access methods include: local filesystem, S3-compatible object storage (CommonCrawl S3 transfer option added in 26.04), custom manifests, Hugging Face datasets (the audio-side example is FLEURS), and reading/writing in the WebDataset format.
The data-source composition of its upstream production use case (Cosmos WFM) is: proprietary/in-house video datasets + public internet video, and explicitly includes synthetically rendered video (accounting for about 4% of the final mixture). NVIDIA has not disclosed the specific proportion of each source, procurement channels, or licensing methods. [Uncertain]

### [OmniHuman Dataset + OHBench](../models/OmniHuman.md) ⚠️

[Sole source: YouTube web video] The HuggingFace dataset page shows each sample carrying a YouTube source URL and clip start/end seconds — this is definitive evidence of the source of this dataset: the data is web-crawled YouTube content, containing no proprietary footage, no licensed acquisition, no synthetic data, and no reuse of existing public datasets.
[Absence of disclosure in the paper body] Notably, the paper body itself does not state where the video comes from, only vaguely mentioning use of a "fully automated pipeline" for collection (fully automated pipeline for high-quality data collection). Source information is confirmed entirely through the field structure of the HuggingFace release — a considerable disclosure gap, since readers cannot judge the legal basis or potential distribution skew of the data from the paper alone.
[Collection orientation: targeted collection by scene and content type] Judging by the 8 scene categories it covers (home spaces, offices, natural environments, urban/rural landscapes, performance venues, retail, industrial areas, etc.) and 8 content categories (talk shows, education, cooking, sports, music, gaming, reviews, film/TV), the collection appears to be targeted retrieval by preset categories rather than indiscriminate scraping — a direct response to the pain point of "insufficient scene diversity in existing datasets" (traditional talking-person datasets concentrate on studio, front-facing, half-body shots; this dataset deliberately expands to diverse real-world scenes).
[Relationship with OpenHumanVid and similar] Table 1 lists OpenHumanVid, SpeakerVid-5M, and CelebV-HQ as comparisons rather than upstream sources, indicating samples from these datasets are not reused.
[Synthetic data] None. Neither the video side nor the audio side uses any synthetic or TTS-generated content (compare UniTalking, which uses IndexTTS2 to synthesize 6.9 million reference audio clips). [Uncertain]

### [Open-Sora Series](../models/Open-Sora.md) ⚠️

Both projects **rely entirely on public datasets and free-license asset sites, with no proprietary copyrighted library, no purchased/licensed data, and no synthetic data** — this is the most fundamental difference from closed-source models such as Sora 2 / Veo 3 / Seedance, and is also why its data recipe is fully reproducible.
[Explicitly listed in Open-Sora 1.2] Webvid-10M (10 million video-text pairs, stock footage), Panda-70M (70 million pairs, a 20M high-quality subset taken), HD-VG-130M (130 million pairs, BLIP-2 captions), MiraData (77,000 long videos, gaming/city walkthroughs), Vript (400,000 densely annotated videos), Inter4K (1,000 4K clips), and free-license asset sites such as Pexels / Pixabay / Mixkit; images use a LAION subset (aesthetic score > 6.5) and Unsplash-lite.
[Open-Sora 2.0] The technical report **does not disclose** the data source or specific dataset names, describing only the filtering method — a notable information gap in this report (presumably continuing the 1.x public-dataset combination, but with no official confirmation). [Uncertain]
[Open-Sora Plan v1.3] Images: SAM 11.1M (paired with LLaVA captions), the English subset of Anytext-3M at 1.8M (about 50% of that set), 160,000 high-quality portraits filtered from LAION-5B, and 5.0M internally annotated by QWen2-VL; video: Panda70M 21M (landscape), VIDAL 3M (portrait, from YouTube Shorts), ShareGPT4Video 0.8M (sourced from CC0 material on Mixkit / Pexels / Pixabay).
[Open-Sora Plan v1.5] Images come from Recap-DataComp-1B, COYO-700M, LAION-Aesthetics; video comes from Panda-70M and "internal sources" — v1.5 is the first to introduce undisclosed internal video data, a step backward in this series' data transparency.

### [Ovi](../models/Ovi.md) ⚠️

Three clearly delineated categories:
(1) Internal proprietary audio-video paired corpus — occupying the core position; the paper states it is "composed of human and nonhuman data from diverse contexts," with the acquisition channel undisclosed [uncertain].
(2) Internal pure-audio corpus — used for training the audio tower from scratch, predominantly human speech, emphasizing linguistic diversity, prosody, and timbre variation. The README calls it "high quality in-house audio datasets." As Character AI is a conversational-companion product company, it is speculated its internal speech resources relate to product speech data, but the paper does not state this. [Uncertain]
(3) Public datasets — used only in the audio fine-tuning stage to supplement sound-effect capability: VGGSound (Chen et al., 2020), AudioSet (Gemmeke et al., 2017), WavCaps (Mei et al., 2024).
(4) No description of licensed/purchased data, and no description of any synthetic-data-construction step.
In addition, at the model level a large amount of open-source assets are reused (Wan2.2 5B video weights, Wan's T5 and video VAE, MMAudio's 16kHz audio 1D VAE, the BigVGAN vocoder), which can be viewed as "indirectly inheriting the distribution of Wan2.2's pretraining corpus."

### [Script-a-Video](../models/Script-a-Video.md)

Disclosure is extremely sparse — only domain-level qualitative description, with no source-channel composition ratio:
[Caption-side 500K] Explicitly described as an "internal dataset" (Tencent's own internal data), covering three domains: film, television, and lifestyle-type content. It is not stated whether this content comes from a proprietary copyrighted library, licensed acquisition, or web collection.
[Generation side] The source channels of the four generation-side datasets are likewise undisclosed; only the nature of each can be inferred from the naming: identity-centric (material with a clearly identifiable person as subject), multi-shot sequences (should come from naturally multi-shot content such as film/TV drama), cinematic pairs (film-grade audio-video pairs), cinematic alignment pairs (high-fidelity film-grade alignment pairs).
[Evaluation-side 225 entries] Covers four categories: movie and TV drama clips, short-form videos, indoor scenes, and outdoor scenes.
[Public datasets] No public video dataset is used for training (VGGSound, AudioSet, Panda-70M, etc. are not mentioned); the public resources used are limited to evaluation benchmarks (the Video-SALMONN-2 testset, UGC-VideoCap, Daily-Omni, WorldSense).
[Synthetic data] No synthetic video/audio is used; but the "annotation" of the 500K dataset is entirely automatically generated by Gemini-2.5-Pro, which is synthetic annotation, not synthetic content.
[Purchased/licensed] Not mentioned.
Overall, the degree of data-source disclosure is markedly lower than comparable works (such as UniVerse-1, which itemizes YouTube content types, or MOVA, which gives a level-by-level composition) — typical of the "domain stated, channel unstated" disclosure style of large-vendor internal data.

### [Seedance 2.0 and Seedance 1.5 pro](../models/Seedance_20_Seedance_15_pro.md) ⚠️

Seedance 1.5 pro is described only as "a large-scale mixed-modality dataset," without breaking down the source composition. Seedance 2.0 discloses nothing. One can refer to the phrasing of the same team's Seedance 1.0 report: acquired "in an ethically and legally sourced" way, from diverse public and licensed repositories, and jointly trained with image data (image data preparation follows the methodology of Seedream). The respective proportions of proprietary/scraped/purchased/synthetic data are all undisclosed. [Uncertain: the specific source composition and mixture ratio for 1.5 and 2.0]

### [SkyReels Series](../models/SkyReels.md)

[SkyReels-V2] Three categories of source are explicitly given: (1) general open-source datasets, naming Koala-36M and HumanVid; (2) self-collected media — over 280,000 films and over 800,000 TV episodes from more than 120 countries (estimated 6.2M+ hours), i.e., high-quality material dominated by official theatrical/TV releases; (3) artistic repositories — high-quality video assets from the internet.
[SkyReels-V4] Split into "real data + synthetic data." Real data = public datasets + licensed proprietary content: images name LAION and Flickr; video names WebVid-10M, Koala-36M, and OpenHumanVid; audio names Emilia, AudioSet, and VGGSound (some materials additionally list SoundNet); the proprietary side is "licensed movies, TV series, short videos, and web series" — forming a data closed-loop with Kunlun Wanwei's short-drama business (short-drama scenario monthly active users of 80 million). Synthetic data is used to fill three gaps: multilingual on-screen text generation, multilingual speech synthesis (TTS), and paired data for inpainting/editing. The proportion of each source is not disclosed.

### [Sora 2](../models/Sora_2.md) ⚠️

Only a single very high-level qualitative description; the System Card states verbatim: "Sora 2 was trained on diverse datasets, including information that is publicly available on the internet, information that we partner with third parties to access, and information that our users or human trainers and researchers provide or generate." That is, three source categories: (1) publicly available internet data (web scraping); (2) data obtained through third-party partnerships/licensing; (3) data provided or generated by users, human trainers, and researchers. No proportion of any source, specific dataset names, scraping scope, or list of partners is given. Whether synthetic data is used is not clearly stated (the word "generate" may hint at inclusion of human-trainer-generated content, but this is not equivalent to model-synthesized training data). [Uncertain]

### [SpeakerVid-5M](../models/SpeakerVid-5M.md)

A single source: YouTube web scraping, with no proprietary captured data, no licensed acquisition, and no synthetic data.
[Collection method] Manually collected high-quality two-person dialogue YouTube videos, spanning nearly two decades from 2006 to June 2025 (the present at time of paper writing).
[Genre composition] Interviews, news reports, seminars/lectures, television programs, variety shows, debates, and educational videos.
[YouTube channel category composition] Entertainment, people and blogs, comedy, news and politics, education, science.
[Relationship to public datasets] Does not reuse existing datasets such as CelebV-HQ, HDTF, MultiTalk, or OpenHumanVid — it is an entirely independently collected new corpus. The paper's Table 1 compares it against these datasets: compared to OpenHumanVid's 13.4M clips / 16.7K hours, SpeakerVid-5M does not lead in total volume, but is unique along dimensions such as "two-person interaction pairing," "speaker ID scale," "1080P proportion," and "structured body-composition annotation."
[Synthetic data] None.

### [Step-Video-T2V](../models/Step-Video-T2V.md) ⚠️

The report does not state the source composition. It is described only as "raw videos processed through a complete pipeline into high-quality video-text pairs suitable for pretraining," without distinguishing the proportions of proprietary asset libraries, public datasets, web scraping, licensed acquisition, or synthetic data; nor does it list any specific data source names. Indirect clues that can be inferred from the pipeline design: (1) the "Original Title" of the video is explicitly retained and used as one caption source, indicating a large amount of data comes from web video platform content carrying title metadata; (2) an EfficientNet watermark classifier and PaddleOCR subtitle detection are used, indicating the raw pool contains a large amount of re-distributed content with station logos/subtitles, consistent with web-scraping characteristics; (3) the training data of the derivative model Step-Video-TI2V is more than 80% anime-style video, indicating the team has a large-scale anime content source. The source of the 3.8B image-text data is likewise undisclosed. [Uncertain]

### [UniTalking](../models/UniTalking.md) ⚠️

Two sources, a binary composition, with no explicit statement of third-party purchase or independent web scraping:
[Public dataset: OpenHumanVid] A large-scale person-centric video dataset released by Fudan University at CVPR 2025. Its raw scale is 52.3 million clips / 70.6K hours; after its own image-quality and person-quality filtering, 13.2 million high-quality clips remain; each video comes with short/long/structured text prompts, a human skeleton sequence, and a corresponding speech audio track — "carrying its own speech audio track" is precisely the key reason UniTalking chose it as a base (most video datasets lack an audio track). UniTalking does not state how many clips were drawn from it, nor whether its native captions and skeleton annotations were reused (judging by its re-generation of captions using Qwen3-VL/Qwen3-Omni, the native captions were presumably not reused). [Uncertain]
[Internal data: Huawei's large-scale internal collection] The paper covers this with only a single phrase, "a large-scale, internal collection," without stating the collection channel (self-scraped/licensed/self-produced), region, scale, time span, or content type. [Uncertain]
[Mixture ratio of the two] Entirely undisclosed. How much of the 2.3 million entries comes from OpenHumanVid versus internal data is one of the most critical gaps in this work's data description. [Uncertain]
[Synthetic data] Used, but only on the audio side: IndexTTS2 zero-shot TTS is used to synthesize 3 reference-timbre audio clips per video (see the synthetic_data_synthesis field for details). No synthetic content on the video side.
[Internal TTS data] An independent data source used in stage-one audio pretraining, unrelated to the 2.3 million audio-video entries above; both source and scale are undisclosed. [Uncertain]

### [UniVerse-1](../models/UniVerse-1.md)

Three categories of source, with self-collected and public datasets each roughly accounting for half:
[Self-collected web data (approximately 4,261 hours, i.e., 1,187 + 3,074)]
- YouTube as the primary source, with content types explicitly listed as: music variety shows, classical music performances, cooking tutorials, public speeches, interviews, vlogs, and tool-usage demonstrations;
- cinematic movie clips;
- Pexels stock footage.
[Public datasets (3,422 hours)] VGGSound and AudioSet; the paper explicitly states the purpose of introducing them is to "bolster the audio modality," i.e., to supplement acoustic diversity of sound effects/ambient sound rather than to improve visual quality — this is also why the LQLS low-quality loss strategy is applied separately to them afterward.
[Licensed acquisition] No paid licensed/purchased data is mentioned.
[Synthetic data] No model-synthesized video or audio training data is used; the only "synthetic" component is that all annotations are automatically generated by QWen2.5-Omni and Whisper (synthetic annotation, not synthetic content).
[Intent behind source selection] The listed content types closely match the model's three major target capabilities: music variety/classical performance → instrument sound generation; speeches/interviews/vlogs → speech and lip-sync; cooking/tool demonstrations/Pexels → ambient sound and foley.

### [Unison](../models/Unison.md) ⚠️

Entirely an aggregation of public open-source datasets, with only a small amount of proprietary data on the audio side; no web scraping, no licensed acquisition, no synthetic data. This is the most notable difference between Unison and industrial work — it is an academic project "standing on the shoulders of existing public datasets."
[Joint audio-video training corpus (about 2 million clips / 3,000+ hours), an aggregation of five open-source datasets]
1. OpenHumanVid — a large-scale, high-quality person-centric video dataset, the main data support for this entry's "person-centric" positioning, providing captioned videos of people's activities;
2. HDTF (High-Definition Talking Face) — high-definition talking-head video, mainly frontal faces from speeches/news, contributing high-quality lip-sync samples;
3. VFHQ (Video Face High-Quality) — high-fidelity facial video, sourced from interview scenes, contributing facial clarity and identity diversity;
4. CelebV-Text — an in-the-wild facial video dataset with text descriptions, contributing text annotation of facial attributes and actions;
5. VGGSound — an audio-visual event dataset (about 310 sound-event categories), the only non-face data source among the five, contributing ambient sound/sound-effect and its visual correspondence.
It can be seen that the first four data sources are all face/person-centric datasets, with VGGSound the sole general sound-effect supplement — this composition determines Unison's capability bias: strong on speaker and person motion, while the visual diversity of ambient sound depends on a single data source.
[Pure-audio training corpus (50 million segments / 130,000+ hours), collected with division of labor by sound type]
- Sound effects: the three datasets YouTube-8M, AudioSet, and WavCaps;
- Music: VidMuse;
- Singing: mainly drawn from the YuE collection (corpus of an open-source singing/music-generation foundation model);
- Speech: includes "internal speech data," which the paper explicitly states is used to "further enrich the diversity and coverage of the training corpus." This is the only non-public source among the entire corpus; scale, language, collection method, and licensing status are all undisclosed. [Uncertain]
[Licensed acquisition] No paid licensed/purchased data is mentioned.
[Synthetic data] No synthetic video or synthetic audio is used as training data.
[Self-scraping] No self-built crawler or web scraping is mentioned — all web-sourced data is obtained via mediation through existing open-source datasets.
[Design intent] The four sound categories on the audio side (speech/sound effects/music/singing) are each equipped with a dedicated data source — this "source acquisition divided by sound type" approach directly serves its speech-sound-effect dual-stream decoupled architecture: the model needs to learn a high-quality prior on each of the two streams separately, so the corpus is likewise organized by type.

### [Veo 3 / Veo 3.1](../models/Veo_3_Veo_31.md) ⚠️

The official statement only says the training data consists of three categories — "audio, video, and image" (Model Card: "Veo 3 was trained on audio, video, and image data") — without giving the proportion of each source. Confirmable source clues: (1) YouTube video — reported by Reuters/CNBC in June 2025 and confirmed by YouTube, Google uses a subset (not the full corpus) of the YouTube corpus to train Gemini and Veo 3, with the legal basis being the "worldwide, non-exclusive, royalty-free" license in YouTube's Terms of Service, of which most creators are unaware; (2) synthetic data — the official confirms generating synthetic captions to improve conceptual diversity, but does not state whether synthetic video is used; (3) the specific composition and proportion of licensed/purchased data versus public datasets [uncertain]. The overall source composition proportion [uncertain].

### [Vidu S1](../models/Vidu_S1.md)

Explicitly two major categories of raw video source (neither states whether the acquisition channel is proprietary, purchased, or scraped):
(1) Livestream and talking-head videos — mainly used to learn fine-grained features such as facial expression, body movement, and lip-sync;
(2) High-quality footage from films and television dramas — used to improve the model's generalization and consistency across different camera angles, scenes, and visual styles.
No mention of using public datasets, licensed acquisition, or synthetic data. The evaluation side uses the public benchmark HDTF as a reference.

### [Wan 2.5 / 2.6 / 2.7](../models/Wan.md) ⚠️

2.5/2.6/2.7 are undisclosed. The Wan 2.1 report states verbatim that candidate datasets are "sourced from both internal copyrighted sources and publicly accessible data" — i.e., two legs, "internal copyrighted data + publicly accessible data"; Alibaba's own ecosystem (Taobao/Youku, etc.) is not named but is a reasonable inference [uncertain].
Compositional breakdown (per the 2.1 report and related papers in the same family):
1) Internal copyrighted sources: proportion undisclosed;
2) Publicly accessible data: Wan2.2-S2V explicitly states it collected from open-source video datasets and did a coarse caption-keyword filter, with additional manual curation from publicly accessible sources of videos containing complex human activity (talking, singing, dancing);
3) Synthetic data: to improve Chinese glyph rendering, Wan 2.1 rendered "hundreds of millions" of text-containing images by synthesizing Chinese characters on a pure white background; for personalization data, InstantID was used to synthesize O(1)M face-swap videos;
4) Licensed acquisition: the report mentions no named asset-library licensing partnership (unlike Lightricks' route of purchasing Shutterstock/Getty).
No figures for the proportion of each source.

### [Audio-Video Generation Evaluation Benchmark Collection](../models/av_benchmarks.md)

[VABench] The T2AV side uses prompts batch-synthesized by an LLM + expert templates (no genuine video source); the I2AV side uses high-quality images manually curated and privacy-screened, subsequently used by a multimodal large model to generate unified audio-visual descriptions. Composed of "synthetic prompts + manually curated images."
[AVBench] Evaluation prompts come from an independent prompt pool (≥720p HD real-person scenes); the evaluator's training data comes from the public dataset OpenHumanVid (real human video), with negative examples synthesized via LLM-driven perturbation and algorithmic mismatch — i.e., "genuine positive examples from a public dataset + programmatically synthesized hard negatives."
[AV-SyncBench] Entirely in-the-wild video scraped from public web platforms (specific platform unspecified); perturbed samples are synthesized by algorithms and speech/instrument conversion models.
[PhyAVBench] The most distinctive — all video is "newly recorded or collected," with the explicit purpose of avoiding data leakage (zero training-set overlap); shot in a controlled environment and collecting multiple samples across different individuals, presenters, and recording devices. No existing public dataset is used.
[Omni-Judge] Prompts come from VidProM (a gallery of real user prompts); videos are generated live by Sora 2 and Veo 3.

### [The Video Captioning Model Ecosystem](../models/caption_models.md)

Source composition of captioner training/evaluation data:
[Teacher distillation as the absolute mainstream (an unchanged paradigm from 2024–2026)] GPT-4V → ShareGPT4Video-40K, Koala-36M seed captions, MiraData structured captions; GPT-4 → the summarization step of CogVideoX; Gemini-2.5-Pro → AVoCaDO-SFT, Script-a-Video's 500K MTSS annotation, Harmony's 4 million entries; the Gemini-3 series → the default teacher for new 2026 work. Student models are generally 7B-class open-source bases (InternLM-XComposer2, Qwen2.5-VL-7B, Qwen2.5-Omni-7B, LLaVA-Video, LLaMA2/LLaMA3).
[Reuse of public datasets] Tarsier2-Recap-585K comes entirely from public datasets (VATEX, TGIF, LSMDC, etc.); AVoCaDO-SFT comes from public sources such as Shot2Story, FineVideo, YouTube-Commons, CinePile + TikTok/short video; AVSCap-130K comes from AVoCaDO-107K, ASID-1M, FineVideo, TimeChatCap-40K, and Movie101 — showing a clear snowball pattern of "dataset stacked on dataset."
[Web scraping] ShareGPT4Video's 4.8M aesthetic videos, Koala-36M, and Panda-70M (derived from HD-VILA-100M) are mainly YouTube and other public web video.
[Manual precision annotation (small in quantity but critical)] SkyCaptioner-V1's camera-motion sub-expert uses 93,000 high-confidence manually annotated entries + 16,000 motion-axis-balanced synthetic entries; Movie Gen's post-training-stage captions are manually refined on top of model output; ALIVE's caption-model training data is "MLLM-generated then manually revised."
[Synthetic/bootstrapped] video-SALMONN 2 explicitly uses its own high-quality caption corpus output for subsequent SFT (bootstrapped data production); Tarsier2 uses model-based sampling to automatically construct preference pairs.

### [Geometric/Structured Annotation Dataset Collection](../models/geometric_datasets.md)

SceneScribe-1M: a hybrid reuse of existing public datasets + web material — HD-VILA-100M (large-scale multi-category video), Panda-70M (video-caption pairs), Koala-36M (precise temporal splitting), Pexels-Video (668,000 high-quality Pexels videos collected via the OpenVideo toolbox); SpatialVID: self-scraped from YouTube, retrieved using motion-related keywords (walk / tour / drone, etc.) to ensure camera-trajectory diversity; WildWorld: entirely synthetic/engine-captured — a self-built game-data collection platform recording in real time from the engine of the AAA photorealistic action-RPG "Monster Hunter: Wilds," neither scraped nor filmed with real people; Action100M: reuses HowTo100M (YouTube instructional video, the 1,199,096-entry version already face-blurred), a secondary annotation of a public dataset

### [Post-Training Data Strategy for Video Generation](../models/post_training_data.md) ⚠️

[Anchor paper] States only that SFT uses a "high-quality text-video dataset" and RLHF uses a "curated prompt set," with the source composition entirely unstated; the base model is internal, so the SFT data is likely drawn from a high-scoring subset of its pretraining corpus, but there is no textual basis for this [uncertain].
[Four cross-cutting paradigms for post-training data sources]
① High-scoring subset within the pretraining pool (most common): CogVideoX (top 20%), Allegro (500M→2M), Motif, Open-Sora, MAGI-1 (stricter filtering data used in the final stage), NAVA (15M→160K) — essentially "same-source purification";
② Targeted collection/independently procured premium sets: Seedance 1.0 defines "hundreds of categories" by attributes such as visual style/motion type for targeted collection; Movie Gen manually selects "cinematic" material and rewrites captions; LongCat-Video additionally incorporates specialized datasets for camera motion and visual style;
③ Model self-generated data (a candidate source for RLHF/DPO): Step-Video-T2V generates multiple videos per prompt with different random seeds; Kling 3.0 Omni samples multiple variants for the same MVL condition; JavisDiT++ generates N=3 candidates per prompt plus 1 ground-truth for 4 total; the preference data of Seedance 1.0 "covers synthetic video generated at different stages of the model, among other multi-source material"; StreamChar distillation Stage II directly uses the student model's online rollout;
④ Feedback loop of real online user prompts: Seedance 1.0 explicitly states it "collects prompts from the training set and online users, performing data balancing and information filtering to remove duplicate and ambiguous prompts" — this is a structural advantage of closed-source commercial models relative to academic work.
[Necessity of deduplication between the prompt set and the SFT set] JavisDiT++ explicitly states its 30,000-entry DPO prompt pool "does not overlap with the SFT training data (apart from the SFT training data)"; VideoReward reserves 13,000 prompt triplets that never appear in the training set as a validation set. Both together illustrate that prompt leakage during the post-training stage is a risk explicitly guarded against.

### [Combined Survey of Mainstream Video Pretraining Datasets: Panda-70M, InternVid, Koala…](../models/pretraining_datasets.md) ⚠️

**All seven are secondary processing of public sources, with none using a proprietary copyrighted library or purchased/licensed data**, and there is serious common-ancestor nesting (HD-VILA-100M is the shared ancestor of at least three of them):
- **Panda-70M**: 100% YouTube, entirely derived from **HD-VILA-100M** (Microsoft, 3.3 million YouTube videos, 720p, balanced sampling across 15 popular categories), re-split from 3.8 million long videos taken from it.
- **Koala-36M**: the paper states verbatim, "we start from the same raw data with Panda-70M," i.e., it is likewise **HD-VILA-100M**, corroborated by the LICENSE pointing to the HD-VILA-100M license. **It is not Kuaishou's own short-video data** — a point commonly misunderstood.
- **InternVid**: independently scraped from YouTube. Two tracks: a core set of about 2 million entries from high-quality channels across 16 categories; another approximately 5.1 million entries retrieved via **about 6,100 action/activity search terms**, the vocabulary being the **1,103** action labels from Kinetics/Something-Something/UCF101 plus **5,001** actions extracted by an LLM from visual grounding corpora and manually verified, with reference to the American Time Use Survey (ATUS 2017–2022). Videos already existing in public datasets as of April 2023 were excluded during scraping to avoid overlap.
- **MiraData**: four platforms + one recycled dataset. YouTube contributes about 68,000 720p videos from **156 manually curated channels** (after splitting, about 34,000 videos → about 173,000 clips); **HD-VILA-100M recycling** (about 100 million clips as input, only **195,000** surviving, with the authors citing this extremely low retention rate to argue for the strictness of their own filtering); Videvo contributes about 63,000, Pixabay about 43,000, Pexels about 318,000. (Paper footnote 2 has the URLs for Videvo and Pixabay swapped.)
- **OpenVid-1M**: **secondary filtering** from four existing datasets — **Panda-50M/70M (the main contributor), ChronoMagic, CelebvHQ, and Open-Sora-Plan (i.e., the Mixkit/Pexels/Pixabay family)**. CelebvHQ originally had no captions, which were added by this project. The exact count from each source is not given in a table in the paper. [Uncertain]
- **UltraVideo**: **YouTube 4K/8K video pool only** (the paper states "the sole source"). Two retrieval paths: (a) from **Koala-36M**, re-filtered by resolution > 4K, frame rate > 25FPS, and duration > 30s, then using **user-behavior meta-signals such as view count/likes/comments** to remove videos "of no user interest," followed by uniform sampling across categories based on similarity between the title/description and preset topics, and deduplication; (b) using an LLM to generate search terms from 108 topics, followed by **manual search** for the latest 4K/8K videos. A total of 5,000 raw videos (duration from 1 minute to 2 hours), then further **manual secondary review** to remove low-quality/blurry/watermarked/shaky footage.
- **LVD-2M**: filtered from a total of **220M clips** across four existing corpora — **HD-VG-130M (130 million), Panda-70M (70 million), InternVid-38M (38 million), WebVid-10M (10 million)**. The source-selection logic is clearly stated: YouTube-family sources have sufficient dynamics but need shot-cut filtering ("only 15% of clips in InternVid exceed 10 seconds, and about 52.5% of these long videos contain a shot change"), whereas stock-footage-family sources (WebVid) have almost no shot cuts but "nearly half are not dynamic enough." **Note: does not include HD-VILA-100M (which appears only in the related-work comparison table), does not include Vidal-10M, and does not include Ego4D.** Composition inferred from the released filenames: about 600,000 from the YouTube family (Panda-70M + InternVid mixed in one file), about 300,000 from HDVG, and **about 1.2 million from WebVid (nearly 60% of the total)** — this breakdown is not given in the paper body.

## Data Compliance and Provenance (proportion of licensed data, rights-cleared datasets, C2PA, etc.)

`provenance_licensing` · Level of detail: brief

### [Allegro](../models/Allegro.md) ⚠️

The paper sets up no data-compliance-and-provenance section, and does not disclose the proportion of licensed data, the proportion of rights-cleared datasets, C2PA/content credentials, watermark provenance, or a copyright review process. The public datasets it relies on (WebVid, Panda-70M, HD-VILA, etc.) are themselves mostly under academic research licenses sourced from web video platforms, whose commercial-use compliance is disputed but not discussed in the paper. The model weights are open-sourced under Apache 2.0, but the training data is not released and no source list is provided. [Uncertain]

### [Apollo](../models/Apollo.md) ⚠️

The paper does not touch on data compliance and provenance issues at all: it gives no proportion of licensed data, states no rights-cleared datasets, mentions no C2PA or any output-side watermark/provenance marker, does not discuss copyright, portrait rights, or the legal status of the data, and has no model-card-level terms of use. The word "safety," which appears in the data-filtering checklist, is the only term touching on compliance, but it is not elaborated on at all. As a closed-source industrial model, its compliance work may exist internally but is not disclosed externally. [Uncertain]

### [CineDance / CineDance-1M](../models/CineDance.md) ⚠️

Compliance disclosure is relatively weak — a fairly notable shortcoming of this work:
[Release license] The dataset is released on HuggingFace under CC-BY-NC-SA-4.0, restricted to non-commercial research and educational use, and requiring sharing under the same terms.
[Access control] A gated-access mechanism is used, requiring a HuggingFace account token and manual review approval by the authors before download — a layer of admission control against downstream misuse.
[Upstream copyright] The material is mainly drawn from web-collected film/TV content and existing public datasets (MiraData / LVD-2M / Koala-36M); the paper does not discuss the copyright licensing status of the upstream film/TV material, the proportion that is rights-cleared, nor does it mention content-provenance standards such as C2PA.
[Risk disclaimer] The dataset card explicitly states that "automated and manual curation cannot guarantee the removal of every low-quality, sensitive, or otherwise undesirable sample," partially shifting compliance responsibility to the user, who is required to supplement quality checks for their own use case.
[Proportion of licensed data / purchased data] No relevant disclosure; inferred to be zero. [Uncertain]

### [CogVideoX](../models/CogVideoX.md) ⚠️

[Uncertain] Neither the paper nor the open-source repository discusses the proportion of copyright-licensed training data, rights-cleared datasets, content-provenance standards (C2PA), or watermarking of generated content. The only related information is indirect: the model weights are released under Apache 2.0 (2B) and a proprietary open-source license (the 5B series, permitting commercial use); the cleaning stage deliberately removes videos with "obvious subtitles, watermarks" (the high-quality fine-tuning stage explicitly states it "effectively removed generated subtitles and watermarks"), but this action is motivated by image quality rather than copyright compliance. The image-side datasets used, LAION-5B / COYO-700M, are themselves public URL-text pair datasets whose copyright status is a matter of public dispute, on which the paper offers no statement.

### [Cosmos-Predict2.5](../models/Cosmos-Predict25.md) ⚠️

[Uncertain]. The paper does not touch on data licensing, copyright, or provenance issues at all: there is no rights-cleared proportion, no explanation of purchased/licensed sources, no C2PA or any content-provenance-and-watermark identification scheme. It only vaguely mentions the source as including "proprietary datasets and open internet platforms." The release-side licensing is clear (code under Apache 2.0, model under the NVIDIA Open Model License), but this does not constitute any statement about the legality of the training data's origin. The NVIDIA Cosmos platform additionally has a Cosmos-Guardrail safety guardrail (inference-side content interception) at the platform level, which this paper also does not elaborate on.

### [Data-Juicer 2.0](../models/Data-Juicer.md) ⚠️

[Compliance of the tool itself] Apache 2.0 licensed, commercial-use friendly. The publicly released T2V data pool is likewise labeled Apache 2.0, but what is actually released is filtered sample metadata/indices — the underlying video material remains subject to the respective licenses of InternVid, Panda-70M, and MSR-VTT — a common practice among academic video datasets to avoid re-distributing copyrighted content.
[Compliance-oriented operator capabilities] DJ provides several operators usable for privacy and compliance processing: video_face_blur_mapper (detects and blurs faces in video, directly serving face-privacy de-identification), video_nsfw_filter (undesirable content filtering), video_watermark_filter and video_remove_watermark_mapper (watermark detection and removal, indirectly related to processing of copyright markers), and additional sensitive-information de-identification operators on the text side. This makes DJ one of the few open-source systems with built-in privacy de-identification operators at the data-processing-framework level.
[Uncertain] The paper and documentation make no mention of support for content-provenance/authentication standards such as C2PA, provide no governance-type functionality such as data-authorization-status tracking, rights-cleared dataset marking, or license-compatibility checking, and do not disclose the proportion of licensed data in Alibaba's internal use. Overall, DJ provides "tools for compliant processing" rather than "a framework for compliance governance."

### [Foley-Omni](../models/Foley-Omni.md) ⚠️

Disclosure is very weak. The paper states, in an Ethics/Limitations-type statement, only in general terms that the data is "collected under appropriate usage agreements," while acknowledging that the model carries a potential misuse risk of being used to produce deepfakes.
An indirectly visible compliance awareness: the V2ST-Bench release strategy explicitly states that, due to redistributable content constraints related to copyright, it does not directly distribute the original video files, instead providing URLs and metadata for users to download themselves — a common avoidance approach for academic audio-video datasets (as with VGGSound and AudioSet).
On the model-weight side, an MIT license is stated, and the redistribution of the Wan2.2-TI2V-5B and MMAudio components is explicitly labeled, directing users to consult the upstream original licenses — a relatively well-formed upstream compliance statement.
[Uncertain] The proportion of licensed data is not disclosed; no rights-cleared dataset acquisition is mentioned; no C2PA or any content-provenance/watermarking mechanism is mentioned; the specific acquisition channel and licensing form of the internal data is not stated; the license compatibility of the individual public datasets used (such as Freesound, AudioSet, VGGSound) is also not discussed.

### [Goku](../models/Goku.md) ⚠️

[Uncertain]. The paper does not discuss data copyright, licensing proportion, rights-cleared datasets, C2PA/content-provenance labeling, watermark provenance, or any other compliance issue at all, and does not mention a datasheet or terms of use. The only confirmable facts are: the public portion relies on existing academic/royalty-free datasets such as LAION, Panda-70M, InternVid, OpenVid-1M, and Pexels (of which Pexels is a royalty-free stock site, and LAION is a CC-scraped image-text pair index that is itself known to have copyright disputes); the licensing status and source compliance of the internal 25 million video clips and 60 million images are both undisclosed. This is fairly typical of Chinese-vendor papers from early 2025, contrasting with the approach of Movie Gen / Veo, which emphasize licensing and C2PA.

### [Hailuo / MiniMax Video](../models/Hailuo.md) ⚠️

[Uncertain]. MiniMax has not published any data-compliance statement for the Hailuo video model: no proportion of licensed data is disclosed, no rights-cleared datasets are mentioned, and no C2PA or any content-provenance/credential standard is mentioned. The Hailuo AI web frontend carries only a general disclaimer, "Content is AI-generated; please use this feature legally and in a friendly manner," which is a user-facing usage guideline rather than a training-data-provenance commitment. Whether output video carries a visible or invisible watermark, or embeds C2PA metadata, is likewise unstated in official documentation. This stands in clear contrast to the compliance disclosure practices of Adobe Firefly, OpenAI Sora 2 (C2PA + visible watermark), and Google Veo (SynthID).

### [HunyuanVideo-Foley](../models/HunyuanVideo-Foley.md) ⚠️

Neither the paper nor the model card discusses the compliance and provenance of the training data: no proportion of licensed data is given, no rights-cleared datasets are declared, no C2PA or any output-side audio watermark/provenance marker is mentioned, no discussion of music copyright risk (despite the pipeline explicitly including a music-detection step, indicating music content indeed exists in the data), and no discussion of voice/speaker privacy.
[The observable compliance posture is on the model side, not the data side] It adopts the tencent-hunyuan-community license rather than a standard open-source license such as Apache-2.0, and sets extra_gated_eu_disallowed: true on HuggingFace, explicitly excluding users in the EU region — the latter is generally interpreted as avoiding the EU AI Act's requirements for training-data-summary disclosure and compliance review of general-purpose AI models. This setting itself is an indirect signal regarding uncertainty in data compliance: if the training data's source were entirely clean, such regional exclusion would typically not be necessary.
[Risk point] The copyright risk of a sound-effect generation model is generally lower than that of video/music generation (foley sound effects are mostly generic acoustic events with a low degree of originality of expression), but the pipeline's music-detection step indicates the raw data contains music; if it is not thoroughly removed, there is a music-copyright exposure surface. The paper offers zero discussion of this. [Uncertain]

### [HunyuanVideo](../models/HunyuanVideo.md) ⚠️

Undisclosed. Neither technical report touches on the proportion of licensed data, a list of rights-cleared datasets, copyright-handling strategy, or provenance standards such as C2PA. The original version only mentions, in the filtering stage, using a YOLOX-like vision model to remove "watermarks, borders, logos, and certain sensitive information," which is closer to visual cleanliness than to copyright compliance. The open-source-side compliance constraints manifest in the model license (the Tencent Hunyuan Community License restricting use in the EU and other regions, and restricting commercial use for entities with more than 100 million monthly active users), rather than in training-data provenance. [Uncertain]

### [InstructAV2AV](../models/InstructAV2AV.md) ⚠️

[Uncertain] Compliance and provenance disclosure is weak, and there is a notable open-vs-compliance tension.
[Known positive practices] The dataset is released under an MIT license, and the dataset card explicitly notes that users must verify compliance with rights applicable to underlying media content — i.e., the authors shift copyright responsibility onto downstream users. The code side is Apache-2.0, and it explicitly states dependence on upstream components such as Wan-AI/Wan2.2-TI2V-5B and hkchengrex/MMAudio.
[Existing compliance concerns]
  · Direct distribution of the media itself: unlike the academic convention of VGGSound, AudioSet, and Foley-Omni's V2ST-Bench, which "only release URL + metadata to avoid redistribution risk," InsAVE-80K directly packages and distributes 139 GB of video/audio files. Its source material includes YouTube-scraped content and film/short-film datasets such as Condensed Movies, MovieBench, and Short-Films-20K, whose underlying material is largely copyright-protected — the legal risk of direct redistribution is significantly higher than URL distribution. Although the target side is synthetic product, the source side is still original material.
  · Identity and voiceprint issues: the dataset includes three subsets — clone_id, clone_voice, and clone_id_voice — explicitly involving identity cloning and voice-timbre cloning. Using ElevenLabs to synthesize speech and TalkNet to localize speakers is, in essence, replacement and re-synthesis performed on real people's faces and voiceprints. The paper mentions no portrait-right/voice-right licensing, speaker informed consent, or de-identification measures.
[Missing items] No proportion of licensed data is disclosed; no rights-cleared datasets are mentioned; no C2PA or any content-credential/watermark/provenance mechanism is mentioned; no detectable marking of generated content. Given that this model's direct capability is "swapping a person, changing lip motion, changing dialogue" (i.e., the technical definition of a deepfake), the lack of a watermarking and provenance mechanism is the most notable compliance gap in this work; the paper's limitations discussion also focuses only on technical shortcomings such as physical realism, lighting consistency, and 3D spatial consistency, without touching on misuse risk.

### [2026 Miscellaneous Joint Audio-Video Generation](../models/JAVG_2026_misc.md) ⚠️

[Uncertain] — none of the seven works discusses the proportion of licensed data, rights-cleared datasets, C2PA watermark provenance, or a copyright-compliance review process. Indirect facts that can be inferred:
(1) Works relying on public academic datasets (OmniCustom depends entirely on SpeakerVid-5M; StreamChar depends on SpeakerVid-5M/TalkVid/OpenHumanVid/Emilia; Baton and CCL partly depend on OpenHumanVid/AudioCaps/WavCaps/VGGSound; NAVA depends on Koala-36M) effectively outsource compliance responsibility to upstream datasets — and these upstream datasets are mostly based on YouTube scraping, which itself carries provenance disputes.
(2) Explicitly higher-risk sources: NAVA's "raw movie/TV footage" (raw film/TV footage, the most copyright-sensitive), CCL's "short dramas and films," and Baton's "videos collected from the internet." None of the three has any copyright statement.
(3) Privacy side: OmniCustom and StreamChar are face+voice-timbre customization tasks, directly involving biometric features (facial identity + voiceprint/timbre). OmniCustom's evaluation set deliberately uses "30 persons who were not included in training data" to verify zero-shot generalization, but does not mention portrait-right/voice-right licensing; ALIVE's character-driven pipeline makes heavy use of ArcFace face embeddings for identity matching, likewise with no privacy statement. This is a common gap in data compliance across this batch of work.
(4) At the licensing level, the only clear case is NAVA: source code is Apache 2.0, but the repository explicitly states that "model weights, pretrained backbones, tokenizer, audio VAE, speaker encoder, and the prompt-rewriting model may be subject to different licenses from their respective original providers" — i.e., the license covers code, not data.

### [Joint Audio-Video Generation Baseline Collection](../models/JavisDiT_baselines.md) ⚠️

Overall disclosure is sparse, but MM-Diffusion and JavisDiT stand out with explicit actions — a rare bright spot in this collection:
[MM-Diffusion] (1) One explicit reason for choosing AIST++ is that its 60 dance tracks are "copyright-cleared songs" — i.e., music copyright was already considered at the data-selection stage; (2) the self-built Landscape dataset is scraped from YouTube, with no discussion of video copyright, but the dataset itself is publicly released for research purposes; (3) code and weights are released under the MIT license.
[JavisDiT / JavisDiT++] The clearest copyright awareness: (1) the repository explicitly states it "cannot release the raw YouTube videos due to copyright issues," providing only 330,000 video IDs for users to download themselves — the standard academic approach to avoiding video copyright issues; (2) content from YouTube in JavisBench undergoes "strict manual legal and ethical verification"; (3) it emphasizes that all training data are public dataset entries, with no purchased data and no internal private data, making the compliance chain relatively clear.
[AV-DiT] Data licensing and compliance are not discussed [uncertain]; both datasets used are already-published academic datasets.
[Harmony] No discussion of the proportion of licensed data, rights-cleared datasets, C2PA, or any other provenance mechanism; the licensing status of the 2 million self-collected clips is entirely undisclosed [uncertain]. The paper describes the ambient-sound data only in general terms as coming from "public sources."
[UniAVGen] Data compliance, licensing, and privacy are not discussed at all (note its internal data is "real human" audio-video, involving faces and voiceprints, which is highly privacy-sensitive, but the paper contains no relevant statement) [uncertain].
Common gap: none of the five mentions C2PA, content watermarking, synthetic-content identification, or a data-subject deletion-request mechanism, or any other modern provenance mechanism.

### [Kling 3.0 / Kling Video 3.0 Omni](../models/Kling_30_Omni.md) ⚠️

[Uncertain] The official has not published the proportion of licensed data, a list of rights-cleared datasets, or a data-provenance mechanism. Compliance measures on the product side are mainly at the output end rather than the training end: generated content adds explicit/implicit AI labels and watermarks in accordance with China's "Interim Measures for the Administration of Generative AI Services" and the AI-generated-content-labeling measures (mandatory watermarking on the free tier, removable on the paid tier). [Uncertain: whether C2PA Content Credentials are integrated — no official statement by Kling of C2PA support was found in the search, contrasting with the already-public support declared by OpenAI/Adobe/Google]. On the data side, only content safety/NSFW filtering is mentioned in the cleaning stage, with no copyright-filtering details. The annotation standard for Kling-Foley's open-source evaluation set explicitly requires "excluding copyrighted background music containing vocals," which can be viewed as circumstantial evidence of the team's copyright awareness.

### [LTX-2](../models/LTX-2.md) ⚠️

Among comparable models, Lightricks has the strongest compliance narrative, but lacks quantitative disclosure. The official and media line is that "training data comes entirely from licensed sources (Getty Images and Shutterstock), with no copyright concerns for commercial use," with Shutterstock's "research license" being called the first instance in the industry of lowering the barrier to training-data acquisition for open-source models. However, note that: (1) the original LTX-Video paper states "publicly available data + licensed material as a supplement," which differs from the media line of "entirely licensed" — the proportion of licensed data has never been published; (2) the technical report gives no list of rights-cleared datasets; (3) no mention of C2PA, watermarking, or any output-side provenance mechanism; (4) the "Social Impact" section of the report only qualitatively acknowledges that the model will reflect biases present in the training data, and lists "authenticity verification and improvements in traceability" as future work. [Uncertain]

### [LongCat-Video](../models/LongCat-Video.md) ⚠️

[Uncertain]. The technical report does not touch on data compliance and provenance at all: no proportion of licensed data, no rights-cleared dataset statement, no explanation of C2PA or any content-provenance/watermark identification scheme, and no discussion of copyright-licensing sources. This is consistent with the common practice of domestic (Chinese) vendor technical reports (compliance details are not written into the paper). Note that the model weights themselves are released under the MIT license, permitting commercial use, but this does not constitute any statement about the legality of the training data's origin.

### [MOVA](../models/MOVA.md) ⚠️

The paper does not discuss data compliance and provenance at all: no proportion of licensed data is given, no rights-cleared datasets are declared, no C2PA or any output-side watermark/provenance marker is mentioned, and there is no discussion of copyright risk, portrait rights, or the legal status of the data source. What can be indirectly judged is that the training corpus includes YouTube-scraped content, Chinese TV dramas, and film clips — copyright-sensitive sources — while the referenced public datasets (ACAV-100M, VGGSound, etc.) are themselves mostly in the form of "YouTube link collections," whose copyright status is bound by the terms of the original dataset. The model weights are released under the Apache-2.0 license, permitting commercial use, but the compliance of the training data is not addressed at all — a clear asymmetry in MOVA between "extremely high methodological transparency" and "zero compliance transparency." [Uncertain]

### [Merged entry: ① Mochi 1](../models/Mochi_MAGI_Motif.md) ⚠️

None of the three has substantive data-compliance-and-provenance disclosure — the biggest common gap among this group of three models:
① Mochi 1: the data source is deliberately obscured, which became precisely the point of controversy at release — a VentureBeat report directly points out that "training datasets are one of the most contentious aspects of AI creative tools, with evidence that many tools have used vast amounts of human-created content scraped from the web without permission or compensation, some of it copyrighted," and Jain "was coy" about this. No proportion of licensed data, rights-cleared datasets, C2PA content credentials, or watermark provenance is disclosed. The model card only notes at the downstream level that "Genmo's video models may reflect biases and stereotypes present in their training data," recommending institutions add additional safety protocols before commercial deployment. [Uncertain]
② MAGI-1: the technical report has no data-compliance section, does not discuss copyright, licensing proportion, C2PA, or a source list, and only gives Apache 2.0 at the weight level. [Uncertain]
③ Motif-Video 2B: likewise has no compliance section, and does not disclose the licensing proportion, rights-cleared data, or content credentials. The relative indirect compliance action is placing NSFW and watermark filtering at the front of the sanitation pipeline (a watermark/station-logo is itself a proxy signal for copyright), and using a VLM in the captioning stage to double-check watermark labels for secondary removal — this can be understood as "engineering means to avoid material bearing rights markers," but the paper does not frame it as a compliance measure. [Uncertain]

### [Movie Gen](../models/Movie_Gen.md) ⚠️

[Uncertain] The paper does not discuss the proportion of licensed data, rights clearance, or content-provenance standards (C2PA/watermarking) at all. Only in the conclusion's "Safety considerations" is it mentioned that: the model was developed for research purposes; the model may learn intra-modal biases (such as visual biases in the video training data); in real deployment, a safety model would be integrated to reject inputs and generated results that violate policy. The compliance process on the data side, face/privacy licensing, and generated-content watermarking are all undisclosed.

### [NVIDIA NeMo Curator](../models/NeMo_Curator.md) ⚠️

At the framework level, NeMo Curator provides no data-provenance-and-licensing-management capability: the official documentation includes no C2PA content-credential writing, no copyright-fingerprint detection, no rights-cleared-dataset management module, and no standardized provenance-metadata field specification. The framework only guarantees that original metadata passes through unchanged during sharding.
Compliance responsibility is entirely handed to the user: under the Apache 2.0 license, NVIDIA makes no guarantee whatsoever about the legality of the data the user processes; the compliance narrative on the Cosmos-model side (Guardrails, trustworthy-AI statements) is a separate matter from the data-curation pipeline. The Cosmos WFM paper also does not disclose the licensing proportion or the legal-basis argument for its 20-million-hour video corpus. This is a clear gap in this toolchain relative to strong-compliance-narrative solutions such as Adobe Firefly / Lightricks (Shutterstock, Getty licensing). [Uncertain]

### [OmniHuman Dataset + OHBench](../models/OmniHuman.md) ⚠️

Compliance and provenance is the weakest-disclosed aspect of this work, with a risk exposure higher than typical video datasets:
[Paper level: zero compliance statement] The full text has no ethics statement, no copyright discussion, no portrait-right/privacy discussion, no informed-consent statement, no mention of C2PA or any content-provenance marker, and no statement on preventing deepfake misuse. The paper also has no dedicated Limitations section discussing the above.
[Mitigation at the release level] It adopts an "annotation-only, no video release" distribution model (each sample gives only the YouTube URL + start/end timestamp) — the standard academic approach to avoiding video-copyright risk, shifting the legal responsibility for downloading and use onto the user. This provides some buffer on the copyright dimension, but is ineffective on the portrait-right and privacy dimensions — the faces of 80,000 real natural persons, 134-point full-body skeletons, SMPL/MANO 3D body and hand parameters, speech transcripts, and emotion labels are all fully released in structured form, and these annotations themselves can serve as high-quality material for deepfakes, with a considerable portion of the people filmed never having consented to their use for training generative models.
[Licensing gap] Neither the HuggingFace dataset page nor the GitHub repository shows a license statement; the scope of use (research / commercial), redistribution conditions, and opt-out mechanism (a channel for a data subject to request deletion) are all missing. As a publicly released dataset of millions of real people, this gap deserves more attention than any methodological oversight.
[Comparison with upstream] OpenHumanVid requires downloaders to submit identity information and undergo review to prevent misuse; OmniHuman has no admission control whatsoever.
[Proportion of licensed data] 0% (entirely web-scraped publicly visible content, with no licensed/purchased portion). [Uncertain]

### [Open-Sora Series](../models/Open-Sora.md) ⚠️

Both projects handle data compliance and provenance weakly, typical of academic/open-source project practice:
- They rely on the licenses of the third-party public datasets themselves (Panda-70M derived from HD-VILA-100M's YouTube videos, Webvid-10M derived from Shutterstock-watermarked material, VIDAL derived from YouTube Shorts) — these datasets' copyright status is itself disputed, and the project did not perform a secondary licensing check, nor did it disclose the proportion of licensed data.
- The ShareGPT4Video portion used by Open-Sora Plan is explicitly labeled as coming from CC0 royalty-free material on Mixkit / Pexels / Pixabay, and Open-Sora 1.2 likewise explicitly uses free-license material from Pexels / Pixabay / Mixkit — this is the only portion in either project that could be called rights-cleared, but neither gives a proportion figure.
- No purchased/licensed data is used, and no rights-cleared dataset list is established.
- Output side: neither implements C2PA metadata, invisible watermarking, or any generated-content-detection tool; model weights are released under permissive licenses such as Apache 2.0, with no provenance constraint on downstream use.
- Notably, the Webvid-10M material carries a Shutterstock watermark, and Open-Sora 1.2 uses it for stage-one low-resolution pretraining (240p/360p), meaning the watermark signal enters early model training — a common issue in open-source reproduction known to the community, whose impact is not discussed in the project documentation. [Uncertain]

### [Ovi](../models/Ovi.md) ⚠️

[Uncertain]. Neither the paper nor the repository discusses the proportion of licensed data, rights-cleared datasets, C2PA/watermark provenance, copyright compliance review, or any other topic, and there is no data-source statement at the Model Card level. The only confirmable facts: the three public audio datasets used during fine-tuning (VGGSound, AudioSet, WavCaps) are all academic-research-licensed datasets, of which AudioSet/VGGSound are based on YouTube videos, themselves carrying source-compliance disputes; the model weights are released under Apache 2.0, but the license covers the weights, not the training data. The licensing status of the internal corpus is entirely undisclosed.

### [Script-a-Video](../models/Script-a-Video.md) ⚠️

The paper does not touch on data compliance and provenance at all: no proportion of licensed data is given, no rights-cleared datasets are declared, no C2PA or any output-side watermark/provenance marker is mentioned, and there is no discussion of copyright, portrait rights, or data-use licensing.
Identifiable compliance risk points:
1) both the caption-side 500K and the generation-side data draw heavily on film/television/TV-drama content, a highly copyright-sensitive source category, and the paper does not state the acquisition channel or licensing status;
2) the Reference stream anchors fine-grained appearance attributes of a person (clothing, accessories, hairstyle, etc.), the Identity Customization module supports injecting a real person's identity from a reference image, and the Event stream records verbatim dialogue bound to a speaker ID — this combination is more sensitive with respect to portrait rights and deepfake risk than purely scene-descriptive captioning, and the paper offers zero discussion of this;
3) none of the assets are publicly released, which objectively lowers the risk of external misuse, but also means there is no model-card-level statement of usage restriction.
This field is entirely blank in the paper. [Uncertain]

### [Seedance 2.0 and Seedance 1.5 pro](../models/Seedance_20_Seedance_15_pro.md) ⚠️

Only principle-level statements, no quantitative disclosure. The Seedance 1.0 report states that data acquisition "prioritizes ethically and legally compliant content, from diverse public and licensed repositories," and deploys classifiers during the filtering stage to remove pornography, violence, child exploitation, and explicit nudity to ensure "ethical compliance and dataset safety." The Seedance 2.0 report states, at the end of the introduction, that "safety is a core consideration of our work," and that a structured safety-evaluation framework is implemented throughout the model's iteration lifecycle, continually assessing and mitigating potential risk to support responsible, compliant, and ethical development. [Uncertain: the proportion of licensed data, a list of rights-cleared datasets, and whether C2PA/watermark provenance is adopted are all undisclosed]

### [SkyReels Series](../models/SkyReels.md) ⚠️

Neither generation's paper gives quantitative compliance disclosure. Only qualitative statements: V4 describes its proprietary data as "licensed" film/TV, short-video, and web-series content; V2 describes self-collected film/TV media and an "artistic repository"; neither gives a proportion of licensed data, a list of rights-cleared datasets, a list of rights holders, or a purchasing method. Neither paper mentions C2PA, invisible watermarking, output provenance, or any content-source-identification mechanism, and neither has an NSFW/privacy-compliance section. On the public-dataset side, LAION, WebVid-10M, and similar corpora with known copyright disputes are used, on which neither paper offers comment. As a generative service operating within China, SkyReels' product side should be subject to the "Interim Measures for the Administration of Generative AI Services" and deep-synthesis-labeling requirements, but neither paper mentions filing or labeling implementation. [Uncertain]

### [Sora 2](../models/Sora_2.md)

Provenance disclosure on the training side is extremely weak, while output-side provenance disclosure is comparatively strong; the two need to be strictly distinguished.
[Training side] The proportion of licensed data is not disclosed, no list of rights-cleared datasets is published, and no licensed/purchasing party is stated. The only clear compliance statement concerns child safety: "responsibly sourcing datasets to exclude CSAM," working in cooperation with the National Center for Missing & Exploited Children (NCMEC) in the US. On copyright: at launch, Sora 2 adopted an "opt-out" (rights holders must actively request exclusion) policy, triggering a controversy over large-scale generation of IPs such as SpongeBob, South Park, and Scooby-Doo; on October 3, 2025 (just 3 days after launch), Sam Altman published a blog post switching to "opt-in" and promising rights holders finer-grained control and revenue sharing. The Motion Picture Association (MPA) publicly pressured OpenAI, and the Japanese government also formally requested OpenAI avoid infringement. A key point: the opt-in policy applies only to the "generation" step, and OpenAI has never clarified whether this policy retroactively applies to "training data" — i.e., copyrighted content already trained into the model has not been removed. OpenAI subsequently reached licensing agreements with Disney (December 2025, a three-year license covering 200+ Disney/Marvel/Pixar/Star Wars characters) and Getty Images (June 2026, a multi-year partnership after Getty and Shutterstock's $3.7 billion merger), but these are all "generation-side IP licensing/showcase partnerships," not explicitly stated as training-corpus licensing for Sora 2.
[Output side] All first-party product assets carry C2PA metadata (an industry-standard verifiable provenance format); videos downloaded from sora.com and the Sora App carry a visible moving watermark; OpenAI retains internal detection tools to determine whether a given video/audio was generated by its products. OpenAI itself acknowledges that "provenance has no single solution."

### [SpeakerVid-5M](../models/SpeakerVid-5M.md) ⚠️

Disclosure is incomplete, a typical academic-dataset compliance posture:
[Stated] (1) explicitly restricted to non-commercial research and educational use, with commercial use explicitly prohibited; (2) states the content comes from the public internet, with copyright belonging to the original creators; (3) provides a takedown policy, allowing rights holders to request removal; (4) does not host the original video, releasing only YouTube video IDs and annotations, shifting copyright risk to the user who downloads it — a common approach to avoid direct distribution of copyrighted content.
[Not addressed] The proportion of licensed data is not given (in fact 0%, entirely scraped); no rights-cleared commercial dataset is used; no mention of C2PA, content-provenance watermarking, or source-trust markers; no discussion of the portrait rights and informed consent of the people shown (the data is entirely real human faces and bodies, including 83K identifiable speaker identities with ArcFace facial features for ID association — genuinely high privacy sensitivity, but the paper offers no privacy-impact assessment).
[Downstream impact] Because the license restricts non-commercial use, any use of SpeakerVid-5M to train a commercial model conflicts with its terms — this poses a potential license-propagation conflict for downstream models that list it as a training-data source (such as MOVA, which adopts Apache-2.0 permitting commercial use), but neither the paper nor the downstream work discusses this. [Uncertain]

### [Step-Video-T2V](../models/Step-Video-T2V.md) ⚠️

Undisclosed. The technical report does not touch on the proportion of licensed data, rights-cleared datasets, a copyright-clearance process, or content-provenance standards such as C2PA, and does not mention generated-content watermarking/identification. The only visible trace of data compliance in the pipeline is the NSFW scoring (a LAION CLIP-based NSFW detector) and watermark detection (used to remove watermarked material, motivated more by visual cleanliness and avoiding obvious copyright markers than by systematic copyright compliance). The model side is open-sourced under the MIT license, with no data-provenance commitment. As a product released within China, actual production must involve content-safety and compliance review, but the compliance methodology on the training-data side has zero disclosure. [Uncertain]

### [UniTalking](../models/UniTalking.md) ⚠️

The paper does not discuss data compliance and provenance at all: no proportion of licensed data is given, no rights-cleared datasets are declared, no C2PA or any output-side watermark/provenance marker is mentioned, and there is no discussion of copyright, portrait rights, or deepfake risk. The indirectly judgeable compliance status:
- The OpenHumanVid portion: this dataset's video samples are collected from "publicly available datasets," and users are required to follow the original licenses; and to prevent misuse, its download requires submitting user information for review and approval. UniTalking, as a downstream user, does not state whether it followed this review process, nor whether it complied with the upstream license terms;
- The Huawei internal-data portion: source unstated, with no compliance statement whatsoever;
- Reference audio synthesized by IndexTTS2: uses the original human voice in real videos as a timbre reference for zero-shot cloning, effectively generating a large amount of cloned speech in real people's voice timbres for training — this is a sensitive point at the level of voiceprint rights and personality rights that the paper does not touch on at all;
- The model's capability itself (arbitrary identity image + arbitrary voice timbre → talking video) is a high-risk deepfake capability, and the paper has no misuse-prevention, usage-restriction, or ethics statement.
Overall this is a typical case of "method disclosure present, compliance disclosure zero," and compared to similar work, the additional compliance issue introduced by its synthetic voice-cloning step is more pronounced. [Uncertain]

### [UniVerse-1](../models/UniVerse-1.md) ⚠️

The paper does not discuss data compliance and provenance at all: no proportion of licensed data is given, no rights-cleared datasets are declared, no C2PA or any output-side watermark/provenance marker is mentioned, and there is no discussion of copyright risk or portrait rights. Indirectly judgeable risk points: the training corpus explicitly includes YouTube-scraped content and cinematic movie clips, copyright-sensitive sources; the Verse-Bench evaluation set is likewise collected from YouTube and Bilibili, and uses TED Talks material from September 2025. The Pexels portion falls under a relatively permissive free-material license. The referenced VGGSound and AudioSet are themselves in the form of YouTube link collections, whose copyright status is bound by the original dataset's terms. The model weights and code are released under the fully commercially permissive Apache-2.0 license, but training-data compliance receives zero explanation — the same "method disclosure present, compliance disclosure absent" asymmetry seen with MOVA. [Uncertain]

### [Unison](../models/Unison.md) ⚠️

The paper does not discuss data compliance, licensing, or provenance at all: no proportion of licensed data is given, no rights-cleared datasets are declared, no C2PA or any output-side watermark/provenance marker is mentioned, there is no discussion of copyright, portrait rights, or deepfake risk, and no model-use-restriction statement is set.
[Indirectly judgeable compliance status — overall better than purely-scraped-type work]
- The audio-video corpus comes entirely from published academic open-source datasets (OpenHumanVid, HDTF, VFHQ, CelebV-Text, VGGSound), each with a clear academic-use license, typically restricted to non-commercial research use. Unison's reuse of them means the compliance responsibility for its training data is, to a considerable degree, shifted onto the licensing framework of the upstream datasets, lower risk than UniVerse-1 (self-collected YouTube + movie clips);
- Note, however, that VGGSound, AudioSet, and YouTube-8M are all essentially YouTube video-link collections, with content copyright belonging to the original uploaders; academic dataset terms do not equal copyright clearance;
- HDTF, VFHQ, and CelebV-Text are all real human-face datasets (including celebrities and public figures); Unison trains an "arbitrary-person talking-video generation" capability on top of them, so portrait-rights and deepfake-misuse risk objectively exists, and the paper offers zero discussion of this — given the participation of two industrial partners, ByteDance and China Telecom, this gap is worth extra attention;
- The source and licensing status of the "internal speech data" is entirely unclear, the least transparent link in the compliance chain.
[Contradiction with licensing] The paper states that code and model will be made public upon acceptance, but its training corpus is largely made up of datasets restricted to academic use, so the commercial usability of the model weights is questionable — the paper offers no explanation of this. [Uncertain]

### [Veo 3 / Veo 3.1](../models/Veo_3_Veo_31.md) ⚠️

Officially confirmable compliance measures: training video is filtered by "various compliance and safety metrics"; on the caption side, unsafe descriptions and personally identifiable information (PII) are filtered out; training data is analyzed to identify potentially harmful data and to review fairness of demographic representation; the output side uniformly embeds the SynthID invisible watermark, paired with production-environment filtering to reduce information-integrity risk. In 2025–2026, the Google ecosystem simultaneously applies SynthID to Imagen / Veo / Lyria output, and is progressively aligning with the C2PA Content Credentials metadata standard (whether Veo 3 output carries a C2PA manifest by default depends on the specific product line) [uncertain]. Key undisclosed items: the proportion of licensed data, a list of rights-cleared datasets, and rights-holder revenue-sharing or opt-out mechanisms. YouTube's use of data has triggered a public dispute over creators' right to know and IP issues.

### [Vidu S1](../models/Vidu_S1.md) ⚠️

[Uncertain]. The report does not mention any information on data licensing, copyright compliance, the proportion of rights-cleared datasets, or C2PA/watermark provenance. The cleaning stage only mentions content-safety filtering (removing NSFW and other inappropriate content) and visual-cleanliness filtering (removing clips containing watermarks, subtitles, or bumper ads), the latter more a picture-quality motivation than a copyright motivation. Given that the data source includes film/TV-drama material, the absence of copyright-source disclosure is a clear gap.

### [Wan 2.5 / 2.6 / 2.7](../models/Wan.md) ⚠️

Disclosure is minimal, the weakest link in the Wan series.
- The report covers the copyright source with only the phrase "internal copyrighted sources," without giving the proportion of licensed data, without listing rights-cleared datasets, and without mentioning C2PA or any other output-side content-provenance standard.
- The only visible provenance mechanism is on the inference side: the API provides a boolean watermark parameter, which when enabled adds an explicit watermark with the fixed text "AI-generated" in the lower-right corner of the video (wan2.7-t2v defaults to false). This corresponds to the requirements for explicit/implicit labeling under China's "Measures for the Labeling of AI-Generated and Synthesized Content" (effective September 1, 2025); whether implicit labeling (metadata) is written is not stated in the documentation. [Uncertain]
- The only clear compliance action on the training side is an internal NSFW safety-evaluation model scoring and filtering the full training dataset, plus detection and training-time cropping of watermarks/logos (both a quality action and a rights-related action).
- No explanation is found of any face/portrait-rights, privacy-de-identification, or data-subject opt-out mechanism.

### [Audio-Video Generation Evaluation Benchmark Collection](../models/av_benchmarks.md) ⚠️

Overall disclosure is weak; only PhyAVBench provides a systematic design:
[PhyAVBench] Fundamentally avoids copyright and data-leakage issues through "entirely self-recorded footage + a guarantee of zero training-set overlap," the cleanest in terms of provenance among the five; it involves 184 on-camera participants, but the paper does not disclose the specific process for portrait licensing and informed consent [uncertain].
[VABench] I2AV images are explicitly privacy-screened before inclusion; the copyright source of the images is not disclosed.
[AV-SyncBench] Data comes from public web platforms; the paper does not discuss copyright licensing or any provenance mechanism such as C2PA [uncertain].
[AVBench] Training data is based on the public dataset OpenHumanVid, inheriting its license; the source of the evaluation prompt pool is not detailed.
[Omni-Judge] Uses VidProM prompts (this dataset itself carries a CC BY-NC-type license); the copyright of generated videos is bound by the terms of service of the respective commercial model.
None of the five mentions C2PA / content-watermark provenance standards.

### [The Video Captioning Model Ecosystem](../models/caption_models.md) ⚠️

This dimension is the weakest link in this ecosystem, with the vast majority of captioner papers not discussing it at all:
[Model licensing is clear] AVoCaDO (Apache-2.0), video-SALMONN 2 (Apache-2.0), Tarsier2-Recap-7b, CogVLM2-Caption, SkyCaptioner-V1, and Aria (Apache-2.0) all give explicit weight licenses; the ShareGPT4Video paper is CC BY 4.0.
[Almost no disclosure of data-source compliance] The copyright status and licensing source of ShareGPT4Video's 4.8M web videos, Koala-36M, and Panda-70M are not stated; AVoCaDO-SFT contains TikTok-10M and ShortVideo subsets, and the redistribution compliance of platform content is not discussed; no captioner work mentions C2PA, rights-cleared datasets, or the proportion of licensed/purchased data.
[A structural risk] The teacher-distillation paradigm generally violates the terms of service of commercial APIs (both OpenAI and Google prohibit using their output to train competing models); ShareGPT4Video, Koala-36M, AVoCaDO, Script-a-Video, and much other work rely on GPT-4V/Gemini distillation, which is unproblematic for academic release but carries legal uncertainty for commercial deployment — a point not raised in any of the papers.
[Generation-side is relatively more cautious] Technical reports from large vendors such as Movie Gen and Veo 3 spend more effort on data compliance, but when it comes to the captioner itself, they say only "an internal model was used," which in fact avoids the issue.
[Uncertain] No publicly quantitative data on licensing proportion exists.

### [Geometric/Structured Annotation Dataset Collection](../models/geometric_datasets.md) ⚠️

SpatialVID has the clearest compliance posture: the entire library is released under CC-BY-NC-SA 4.0 non-commercial license; the manual initial-screening stage removes videos with titles containing inappropriate terms, but the licensing status of the underlying YouTube material itself has not been cleared entry by entry [uncertain]; SceneScribe-1M inherits the respective licenses of upstream datasets (HD-VILA/Panda-70M/Koala-36M), with the Pexels portion being freely commercially usable material, and the overall licensing proportion undisclosed [uncertain]; WildWorld's data is generated by a game engine, involving no real-person portraits or web copyright, but the copyright of the game content itself belongs to the publisher, and the paper does not clarify the terms for academic redistribution [uncertain]; Action100M releases only annotations (about 205GB), requiring users to obtain the video themselves from HowTo100M, avoiding video-redistribution risk, with the source video already face-blurred to protect privacy. None of the four mentions C2PA or any other content-provenance watermarking mechanism.

### [Post-Training Data Strategy for Video Generation](../models/post_training_data.md) ⚠️

Neither the anchor paper nor the vast majority of the works surveyed disclose anything about the copyright and provenance dimension of post-training data [uncertain]. Relevant facts that can be recorded:
· Because post-training data is small in scale (10^4–10^7), it is in theory easier to achieve rights-clearance than pretraining data; Movie Gen's process of "manually selecting cinematic material + manually rewriting captions" and SkyReels-V4's "1 million manually curated entries" are, in terms of cost structure, feasible for entry-by-entry licensing review, but neither states this;
· An additional compliance dimension for preference data is "annotator labor" — Step-Video-T2V mentions that preference annotation is "supervised throughout by quality-control personnel for consistency," HunyuanVideo 1.5 uses GSB labeling, and Seedance 1.0 uses a multi-dimensional annotation protocol of "select the best/worst along a specified dimension, while ensuring the best is not inferior to the worst along other dimensions" — but none discloses the annotation team's size, source, compensation, or training method;
· Model-self-generated preference candidates (Seedance 1.0, Step-Video, Kling, JavisDiT++) avoid external-material copyright issues, but introduce the distributional risk of "synthetic-data feedback";
· The public preference dataset HPDv3 (1.08M text-image pairs) and VideoReward (108,000 videos from 12 T2V models: Gen2, SVD, Pika 1.0, Vega, PixVerse v1/v2, HiDream, Dreamina, Luma, Gen3, Kling 1.0/1.5) — the latter's videos are all generated by commercial models, and the licensing status of their redistribution is an undiscussed gray area. [Uncertain]

### [Combined Survey of Mainstream Video Pretraining Datasets: Panda-70M, InternVid, Koala…](../models/pretraining_datasets.md) ⚠️

**The compliance foundation of all seven is fragile, and quality is uneven**:
- **The common underlying problem**: the vast majority of material is YouTube video, and none of the dataset publishers owns the copyright; they protect themselves only with the two soft clauses of "non-commercial research" + "infringement can be requested to be taken down." Both Panda-70M and MiraData carry an explicit takedown commitment ("We will remove the video samples from our dataset as long as you need it"). UltraVideo's license goes further, requiring users to comply with YouTube's ToS and GDPR/CCPA themselves, and explicitly disclaims any guarantee of ownership of the material — effectively admitting it does not own the YouTube material.
- **Proportion of licensed data**: the only component that could be called rights-cleared is CC0/free-license content from stock sites — the Videvo/Pixabay/Pexels portion of MiraData, the portion of OpenVid-1M coming from Open-Sora-Plan (Mixkit/Pexels/Pixabay), and the WebVid stock-footage portion of LVD-2M (though WebVid itself carries a Shutterstock watermark and has already been taken down due to copyright issues, making it a high-risk component). **None discloses a figure for the proportion of rights-cleared data.**
- **Not releasing the video itself as an avoidance measure**: Panda-70M / InternVid / Koala-36M / MiraData / LVD-2M — all five release only URL + timestamps, shifting the legal risk of scraping onto the user. InternVid explicitly states it "follows the convention of existing datasets, sharing only video IDs to comply with YouTube's policy." The cost is **irreversible link rot** — the Panda-70M repository directly instructs users to skip samples with status=failed_to_download; LVD-2M's download script even has built-in multi-account rotation (ACCOUNT_NUM) and warns that Google accounts may be banned. **None has published measured data on ID survival rate.**
- **The opposite approach**: OpenVid-1M and UltraVideo directly host the video itself (12.4TB / the full set of 4K-8K clips), giving the best reproducibility but the greatest legal exposure; UltraVideo closes the loop by "prohibiting redistribution of the raw video, requiring derivatives to inherit the same terms."
- **Privacy handling**: only **Panda-70M** does one concrete piece of work — using NLTK to replace all person names in captions with "person," and filtering out samples containing harmful/violent language and drug- or hate-speech content. The other six have no name de-identification and no facial-privacy processing.
- **C2PA / watermarking / generated-content provenance**: **none of the seven** has any, and there is no provenance mechanism whatsoever on the output side.
- **Self-contradictory license terms**: MiraData is the worst offender — the repository is labeled GPL-3.0, and the "License Agreement" section of the README prohibits all commercial use, yet the same README's final sentence states it "is supported for commercial usage." A defensible reading is: the code is GPL-3.0, the data is for research only, and video copyright still belongs to third parties. Koala-36M's HF page, meanwhile, **has no license tag and no dataset card at all**, with the non-commercial terms present only in the GitHub repository.
## Clip Duration Distribution and Splitting Strategy

`duration_distribution` · Level of detail: brief

### [Allegro](../models/Allegro.md)

Splitting and duration constraints are the first gate in the filtering funnel:
· Raw videos are first stripped of samples with duration <2 seconds or frame rate <23 FPS;
· After PySceneDetect scene splitting, only single-scene clips of 2–16 seconds are retained (clip-level duration bounds);
· The later 720p pre-training and fine-tuning stages further tighten this to 6–16 seconds (Table 1: T2V Pre-train 720p uses two tiers, [2s,16s] and [6s,16s]; Fine-tune uses [6s,16s]).
Distribution (Appendix A Fig.11): during pre-training, the three buckets 2–6s / 6–10s / 10–16s are fairly evenly distributed; during fine-tuning, the overall distribution shifts right due to the 6-second floor, and the share of longer clips rises significantly to match the 88-frame (~6s @15FPS) generation target.
The final clips are uniformly transcoded to H.264 at a fixed 30 FPS.

### [Apollo](../models/Apollo.md) ⚠️

The paper discloses no clip-duration information whatsoever: no fixed training-clip duration or duration distribution is given, no frame rate is given, no minimum/maximum duration threshold is given, and it is not stated whether fixed-length splitting or variable-length bucketing is used. The only indirect constraint that can be inferred comes from the VAE spec: the Video-VAE outputs a 3 Hz temporal embedding (i.e., 3 latent frames per second), and the Audio-VAE outputs a 43 Hz embedding; the paper explicitly states the Video-VAE "handles input videos with varying resolutions and frame rates," indicating the input-side frame rate is not uniform and is normalized to a 3 Hz representation by the VAE. The only splitting-strategy element that can be confirmed is scene splitting, which guarantees each sample contains only a single scene, but the window-length rule after splitting is not stated. [Uncertain]

### [CineDance / CineDance-1M](../models/CineDance.md) ⚠️

Duration distribution is the most defining differentiating dimension of this dataset, paired with a dedicated "anti-fragmentation" strategy:
【Average duration】92.8 seconds/sequence, several times longer than comparable multi-shot datasets (compare MiraData 72.1s, LVD-2M 20.2s, SpeakerVid-5M 8.3s).
【Splitting strategy】Rather than the conventional "one shot = one sample" approach, a two-level scheme is used: first TransNetV2 extracts 25.89 million atomic shots, then consecutive shots are recombined "bottom-up" according to film-theory rules into narrative sequences — the sequence, not the single shot, is the final sample unit.
【Minimum-duration constraint】An empirically-derived minimum duration for narrative completeness of 18.4 seconds was measured from a human-annotated reference set, from which a soft threshold of 20 seconds was set to prevent parsing out meaningless overly short clips. Ablations show only 3.1% of sequences in the final data are shorter than 20 seconds.
【Duration trimming】MLLM-guided temporal truncation removes opening/closing-credit content from the head and tail of long videos, with the truncation length given by t = max(5 minutes, 0.1L), where L is the total duration of the original video.
【Distribution plot】Paper Fig.5 gives a joint histogram of duration and shot count, but specific quantiles (P50/P90, min/max) are not stated in text. [Uncertain]

### [CogVideoX](../models/CogVideoX.md) ⚠️

· Training clips average roughly 6 seconds and are all single-shot; no duration histogram is given [distribution details uncertain].
· Explicitly adopts "mixed-duration training": the paper criticizes fixed-frame-count training for having to "discard short videos and truncate long videos," which under-utilizes the data. Instead, videos of different durations (and, correspondingly, different resolutions) are packed into the same batch, with shape consistency within a batch ensured via a Multi-Resolution Frame Pack (inspired by Patch'n Pack), and 3D RoPE used to model positional relationships across different shapes. This effectively replaces traditional bucketing with packing (the paper explicitly notes that SDXL-style bucketing "complicates both the data and the training pipeline").
· The training-stage duration ceiling is progressively relaxed: stage1/stage2 cap at 6 seconds, stage3/stage4(FT) cap at 10 seconds (corresponding to sequence lengths 25k → 75k → 700k).
· On the 3D VAE side, to save memory, a two-stage training scheme is used: first training on 17-frame videos, then fine-tuning on 161-frame videos using context parallelism.
· The splitting strategy (see shot_segmentation): the paper does not describe an explicit scene-splitting tool, instead using a "Lack of Motion Connectivity" classifier to remove clips containing splices/transitions wholesale.

### [Cosmos-Predict2.5](../models/Cosmos-Predict25.md) ⚠️

Splitting and duration strategy are clearly disclosed:
(1) Long videos are first split into single-shot segments via shot-aware splitting;
(2) Duration threshold: "Very short clips under 5 seconds are discarded";
(3) Retained range: remaining clips have durations from 5 to 60 seconds (ranging from 5 to 60 seconds), totaling 6+ billion clips;
(4) Caption granularity: each clip is further split into 5-second windows and captioned window by window (this is the "finer content granularity" improvement relative to Cosmos-Predict1);
(5) Fixed training-consumption granularity: all stages uniformly generate 93 frames at 16 fps, roughly 5.8 seconds (corresponding to 24 latent frames after WAN2.1 VAE's 4×8×8 compression);
(6) Domain data durations are fixed-length: autonomous-driving clips are uniformly 20 seconds; Human Dynamics requires at least 5 seconds.
(7) Duration is one of the four axes (length) used for final sharding, supporting duration-based sampling.
No specific distribution histogram or average duration is given for the 5–60 second range. [Uncertain: specific statistics of the duration distribution]

### [Data-Juicer 2.0](../models/Data-Juicer.md) ⚠️

【Splitting operators — one of DJ's strengths】Provides three complementary video-splitting operators that can be combined as needed:
  · video_split_by_scene_mapper — splits video into shot-level clips based on scene-change detection (built on a PySceneDetect-family approach), the splitting method best suited to constructing training data for video-generation models.
  · video_split_by_duration_mapper — splits into equal-length clips of a fixed duration, suitable for quickly constructing fixed-length training samples.
  · video_split_by_key_frame_mapper — splits at key-frame (I-frame) boundaries, aligning cut points with the encoding structure for low decoding overhead.
  There is also video_clip_reassembly_mapper, used to reassemble the processing results of overlapping clips back into a long video (aimed at embodied-AI hand-action scenarios).
【Duration filtering】video_duration_filter supports setting a duration lower/upper bound interval, removing clips that are too short (insufficient information) or too long (splitting not clean).
[Uncertain] The official T2V case study does not disclose a clip-duration distribution histogram or average duration for the final data pool; InternVid/Panda-70M/MSR-VTT are themselves already pre-split short-clip datasets (Panda-70M averages ~8.5 seconds, MSR-VTT ~10–30 seconds, InternVid averages ~11 seconds); the case study does not perform additional re-splitting, so the splitting operators were not actually invoked in that case. DJ also does not provide an officially recommended duration-bucketing strategy.

### [Foley-Omni](../models/Foley-Omni.md) ⚠️

Limited disclosure.
【Eval set】V2ST-Bench clip duration is explicitly 5–10 seconds.
【Training set】The paper gives no distribution histogram or average duration for training clips. A rough order of magnitude can be back-derived from the data scale: roughly 4.9k hours corresponds to roughly 2.7M samples, giving an average of ~6.5 seconds per sample, consistent with V2ST-Bench's 5–10 second range — indicating the overall approach favors short clips (roughly within 10 seconds), aligning with the mainstream V2A settings it benchmarks against, such as MMAudio and VGGSound (10-second clips).
【Splitting strategy】[Uncertain] The paper does not describe how long videos are split into training clips, and does not mention shot-boundary detection, fixed-window sliding, or other splitting methods. The metrics used at the filtering stage (motion score, Synchformer sync score, ImageBind score) are all computed at the clip level, implying splitting happens before filtering, but the specific splitting algorithm is not explained.

### [Goku](../models/Goku.md)

【Splitting strategy】Raw long videos are first split into semantically coherent short clips, with two core constraints:
(1) Duration floor — clips shorter than 4 seconds are directly discarded during preprocessing (duration ≥ 4s).
(2) Duration ceiling — individual clips are truncated to a maximum of 10 seconds (maximum clip length 10 seconds); overly long continuous shots are forcibly truncated.
Training-clip durations therefore fall strictly within the [4s, 10s] range, which also determines that Goku's generation capability is concentrated on short videos under 10 seconds.
【Other preprocessing constraints】Bitrate ≥ 500 kbps; frame rate ≥ 24 FPS (film standard) or 23.976 FPS (NTSC standard); low-frame-rate material is removed.
【Not disclosed】The specific duration histogram within the range, average clip duration, and sampling weights for each duration bucket.

### [Hailuo / MiniMax Video](../models/Hailuo.md) ⚠️

[Uncertain] (only output-side specs are available; no training-data-side distribution).
Output side: video-01 is fixed at 6 seconds; Hailuo 02 offers three tiers — 768p-6s, 768p-10s, 1080p-6s; Hailuo 02 Fast offers 6s and 10s at 512p; the official documentation example for Hailuo 2.3 is 1080p / 6 seconds.
Training side: the duration distribution of training clips, average clip length, and the strategy for splitting long videos into clips have never been disclosed officially. Only from the 6s/10s output tiers can one infer that training clips likely fall roughly within 10 seconds, but this is an inference, not a disclosure.

### [HunyuanVideo-Foley](../models/HunyuanVideo-Foley.md) ⚠️

【Strictly fixed at 8 seconds】This is the most distinctive feature of this work's duration strategy — all training clips are uniformly fixed at 8 seconds ("chunk them into 8-second intervals"), with no variable-length design, no length bucketing, and no short/long-clip curriculum.
【Two-level splitting logic】First, scene-detection algorithms are used to perform shot splitting on raw long videos, yielding semantically complete shot segments; the shot segments are then cut into fixed 8-second blocks. That is, "split by content first, then by duration" — different from UniVerse-1's approach of "scene-split then discard <5s, keep variable length": HunyuanVideo-Foley does not discard but forcibly regularizes.
【Rationale for choosing 8 seconds】The paper does not explain why 8 seconds was chosen. Three plausible reasons can be inferred: (1) it is close in order of magnitude to mainstream V2A evaluation benchmarks (VGGSound clips of 10 seconds, MovieGen-Audio-Bench), facilitating evaluation alignment; (2) 8 seconds × 50 Hz latent frame rate = 400 audio latent tokens, a training-friendly sequence length; (3) 8 seconds is enough to cover the complete acoustic envelope of the vast majority of Foley events (footsteps, door open/close, collisions, etc.), while avoiding sync drift over long sequences. [Uncertain]
【Boundary issues from splitting not discussed】An 8-second hard cut will truncate acoustic events that span the boundary (e.g., a 12-second-long rain sound cut into an 8-second piece plus a 4-second remainder); the paper does not explain how the remainder is handled — whether it is discarded, zero-padded, or concatenated with the next segment is not described. [Uncertain]
【Inference-time duration】The open-source implementation supports sound-effect generation for videos of arbitrary duration, but training only ever saw 8-second samples; how long videos are handled (segment-then-generate then stitch?) is not explained in the paper.

### [HunyuanVideo](../models/HunyuanVideo.md)

【Original version】No duration histogram given. The splitting strategy is explicit: PySceneDetect is used to cut raw videos into single-shot clips, then an OpenCV Laplacian operator selects a sharp starting frame within each clip. Training duration is expressed via frame count — from 65 frames (256×256×65) up to 129 frames (720×1280×129), with support for multi-frame-count bucketing across 1–129 frames.
【1.5】Much more explicit: all training clips are uniformly split to 2–10 seconds; pre-training stages III–V use 16fps at 2–10s, stage VI onward raises this to 24fps at 2–10s, and the CT/SFT stages maintain 24fps at 2–10s. That is, 1.5 follows a route of "short clips as the mainstay, curriculum progression via frame rate rather than duration," and never trains on clips longer than 10 seconds.

### [InstructAV2AV](../models/InstructAV2AV.md) ⚠️

The distribution is extremely concentrated — all clips are uniformly fixed at 5 seconds, so there is no distribution to speak of. This is a typical feature of synthetic datasets relative to real, captured datasets: since the target is synthesized by a generative model, and the generation window of the underlying models Wan2.2-5B / Ovi is fixed, the output duration is necessarily pinned to the models' native window (5 seconds @ 24FPS = 120 frames).
【Splitting strategy】Two levels:
  1. Shot-level splitting: PySceneDetect (Castellano, 2024) is used to split crawled/collected raw long videos into single-shot clips. This step guarantees no shot changes within a clip, a precondition for all subsequent processing (point tracking, speaker localization, mask propagation, edit synthesis) to hold — a cross-shot clip would prevent Grounded-SAM-2's instance masks from propagating continuously.
  2. Fixed-length extraction: a 5-second window is taken from the single-shot clip. [Uncertain] The paper does not state the specific extraction rule — whether the first 5 seconds, the middle 5 seconds, or a sliding window producing multiple 5-second segments (the latter would introduce high overlap between clips, a potential hidden source of duplication); nor does it state how single-shot clips shorter than 5 seconds are handled (discarded or padded).
【Comparison with other works surveyed】Foley-Omni uses 5–10 seconds, VGGSound uses a fixed 10 seconds; this work's 5 seconds is on the short side among comparable work, directly limiting the complexity of editable content — 5 seconds can hardly carry multi-turn dialogue or complex event sequences, which also explains why all four task types are "localized edits of a single target" rather than narrative-level rewrites.

### [2026 Other Audio-Video Joint Generation Works](../models/JAVG_2026_misc.md) ⚠️

The duration strategies of these works are highly differentiated, reflecting their distinct task positioning:
【Fixed short clips | OmniCustom】Strictly fixed-length design, coupled with a "reference-training pairing" scheme: first filter out "videos shorter than 10 seconds," then from each ≥10-second clip take "the first 4 seconds as the reference audio" and "the last 5 seconds ... designated as both the training audio and video clips" — i.e., a single 10-second source item simultaneously produces a 4-second reference audio and a 5-second training sample, sharing the same timbre but with different speech content ("each reference-training pair shares the same timbre but contains distinct speech content"). This is a very clean "same-timbre, different-content" pairing construction method.
【Short clips + long source | ALIVE】Character-driven data extracts N clips of "3–10 seconds" from "long videos (10–30 minutes)"; generation output is 5 to 10 seconds. The identity anchor is instead taken as a "1.5-second sub-clip with maximum sync score" — i.e., the 1.5-second sub-clip with the highest sync score is used as the identity-representative frame source. The overall training-data duration distribution is not disclosed [uncertain].
【Average 7 seconds | NAVA】Explicitly states "The average video duration is about 7 seconds," the only work among the seven to give an average duration. Inference defaults to 37 frames @24fps (~1.5 seconds), configurable.
【Streaming chunking | StreamChar】The most distinctive duration strategy — training is constrained by "training data contains no videos/transcripts longer than 20 seconds," while inference must generate a continuous 5-minute stream, achieved by chunk concatenation (each chunk 33 frames @24fps ≈ 1.375 seconds), with a historical audio context window capped at 15 seconds. This constitutes a notable "train short, infer long" generalization gap, and is also the fundamental reason the progress-aware pointer and persistent visual anchor anti-drift mechanisms exist. In evaluation, "150 clips generating 10s audio-video pairs" and "50 clips paired with randomly sampled transcripts (>300 words) to produce 5-minute continuous streams" are used.
【Fixed short duration | ITS-JAVG (evaluation side)】The tested base models are each fixed: JavisDiT generates 4-second video, MMDisCo generates 2-second video.
【Not disclosed | Baton, CCL】Baton's Sem100 eval set consists of "100 unseen videos (10 seconds long)"; training-clip duration is not stated [uncertain]; CCL does not mention duration distribution at all [uncertain].
【Splitting strategy】Only NAVA mentions that "raw videos are first segmented at scale with a Hadoop-based pipeline" (large-scale Hadoop-pipeline splitting); none of the others describe a splitting strategy [uncertain].

### [Audio-Video Joint Generation Baseline Collection](../models/JavisDiT_baselines.md) ⚠️

The duration strategies of these works strongly reflect differences in compute budget and objective, with the common trait of being "short, fixed-length, no multi-duration bucketing":
【MM-Diffusion】Source videos are cut into 10-second non-overlapping clips as the dataset unit, but actual training samples are shorter: both Landscape and AIST++ are sampled by fixed frame count, with the corresponding audio segment length on the order of ~1.6 seconds [uncertain: the exact training-clip duration in seconds stated in the original paper].
【AV-DiT】The shortest: each training sample is 16 video frames plus an audio waveform (16kHz) truncated or padded to 1.6 seconds. Video and audio durations correspond strictly; no bucketing.
【JavisDiT / JavisDiT++】Training and evaluation are uniformly 4-second clips (240P / 24fps); the paper additionally runs an extended test at 10 seconds. All of JavisDiT++'s JavisBench evaluations are likewise fixed at "240P, 4 seconds." Hard constraints on the data-preparation side: audio is uniformly truncated to within 30 seconds before splitting; videos with fewer than 10 frames are discarded; fps is uniformly normalized to 16Hz.
【Harmony】Human speech clips are explicitly 3–10 seconds (one of the few cases with an explicit range rather than a fixed length); the maximum clip length in stage-one audio pre-training is 10 seconds, within which the reference audio is a randomly extracted 1–3 second clip.
【UniAVGen】Video is uniformly processed at 16 fps before VAE encoding; the specific clip duration in seconds is not explicitly given [uncertain].
Shared limitation: all five works' training samples are within 10 seconds, none involves minute-scale long videos, multi-shot stitching, or duration-bucket scheduling — this is the most fundamental gap between academic baselines and industrial models (e.g., Veo/Sora 2/LTX-2).

### [Kling 3.0 / Kling Video 3.0 Omni](../models/Kling_30_Omni.md) ⚠️

[Uncertain] The training-clip duration distribution is not published. Only cleaning-rule-level facts can be confirmed: Kling-Omni reports that its basic filtering includes "resolution and duration thresholds" and removes clips with "abrupt scene cuts and incoherent shot transitions" (i.e., it favors retaining continuous single-shot clips). Related work from the same team can serve as a reference point: Koala-36M's clips average 13.75 seconds after temporal splitting; Kling-Foley uniformly splits long videos into 8–10 second clips. Kling 3.0 Omni's target generation length is 15 seconds, so training-clip durations very likely cover and slightly exceed that length to support 15-second/multi-shot generation. [Uncertain: specific bucketing and the short/long sample mixture ratio]

### [LTX-2](../models/LTX-2.md) ⚠️

LTX-2 does not disclose this separately. The related sibling model LTX-Video can be used as a reference: paper Fig.14(b) gives a histogram of post-filtering clip-duration distribution, ranging roughly 0–30 seconds, with quality concentrated in shorter clips (the distribution decreases monotonically with duration). Splitting strategy: the input unit of the data pipeline is itself an "Input Shot," with the entire pipeline operating at shot granularity and ultimately producing "Final Shots." Training simultaneously spans multiple "resolution × duration" combinations, with resizing used to make the token count roughly equal across samples. On the generation side, the ceiling is 20 seconds; beyond roughly 20 seconds, temporal drift and desynchronization appear. Specific bucket values are not published. [Uncertain]

### [LongCat-Video](../models/LongCat-Video.md) ⚠️

The report gives no statistical distribution of clip duration (no average duration, no duration histogram). The splitting strategy is explicit: scene detection is used to cut long videos into "training-friendly clips while maintaining content consistency," using a combined PySceneDetect + in-house self-trained TransNetV2 dual-path approach. After splitting, a duration field is recorded in the metadata and used for subsequent filtering, but the filtering thresholds are not published. The target training duration is architecturally fixed: all five pre-training stages plus SFT and RLHF stages use a video length of 93 frames (≈3.1 seconds at 30fps); minute-scale long videos are not achieved by generating long clips in a single pass, but rather via chained extrapolation using a Video-Continuation task — i.e., recursively continuing generation conditioned on multiple preceding frames. In Avatar 1.5's online filtering, "duration" is one of the explicit progressive filtering conditions. [Uncertain: specific duration-distribution values and filtering thresholds]

### [MOVA](../models/MOVA.md)

Adopts a strict fixed-duration splitting strategy, with no variable-length bucketing:
- All training clips are uniformly 8.05 seconds, corresponding precisely to 193 frames at 24fps (a first frame plus 8 seconds of video, i.e., 1 + 8×24 = 193).
- Across the three training stages, the frame count is held constant at 193 regardless of resolution.
- Splitting is not equal-interval sliding-window, but rather a window-generation algorithm jointly driven by VAD (voice activity detection) + PySceneDetect (scene change) (see shot_segmentation and the two pseudocode blocks in Appendix A.3).
- The start point of speech-segment windows is adaptively shifted/adjusted forward to avoid cutting off an ongoing utterance, ensuring continuity of spoken content.
- Inference-side output is likewise around 8 seconds (a 720p, 8s, 24fps clip yields roughly 1.6×10^5 tokens; the paper lists sequence length as a major bottleneck in its Limitations section).

### [Merged entry: ① Mochi 1](../models/Mochi_MAGI_Motif.md) ⚠️

① Mochi 1: The training-side duration distribution and splitting strategy are entirely undisclosed; only known is that the generation-side ceiling is 84 frames / 30 FPS / 5.4 seconds. [Uncertain]
② MAGI-1: Gives a staged duration ceiling (Table 5) — both stage-1 and stage-2 are "≤ 8 seconds," stage-3 relaxes to "≤ 16 seconds," with the paper explaining that the final stage extends duration to let the model capture richer temporal dynamics. The splitting strategy uses PySceneDetect to cut single-shot clips (see shot_segmentation). No lower bound and no distribution histogram are given. [Partially uncertain: duration lower bound and actual distribution]
③ Motif-Video 2B: Has an explicit lower bound after splitting — "Clips shorter than two seconds after merging are discarded" — with the goal of ensuring every training clip covers a meaningful temporal span. Training-side duration is actually determined by frame-count buckets: four tiers of single frame (image), 33 frames, 65 frames, 121 frames, with clip-length filtering re-applied at every stage transition (thresholds tighten as resolution increases). No duration histogram is given. [Partially uncertain: specific distribution]

### [Movie Gen](../models/Movie_Gen.md)

Raw videos range from 4 seconds to 2 minutes, averaging 28 seconds; since training requires 4–16 second clips, FFmpeg is first used to detect scene changes, 1–2 scenes exceeding 16 seconds are sampled per video, and one 4–16 second clip is randomly drawn from each such scene — explicitly avoiding shot-boundary-agnostic random sampling (which would otherwise produce videos with frequent, jarring transitions when generated). Over 50% of training clips fall in the 15–16 second range.
Bucketing strategy (Table 2, five duration buckets, with consistent latent frame counts within a bucket to facilitate batching): 10.67s@24fps → 256 frames/32 latent frames; 16s@16fps → 256 frames/32 latent frames; 12–16s@21–16fps → 256 frames/32 latent frames; 8–12s@24–16fps → 192 frames/24 latent frames; 4–8s@32–16fps → 128 frames/16 latent frames. The first two buckets are taken from the middle segment of 10.67–12 second and ≥16 second long videos. Frame rate is controlled via an FPS token in the caption (16–32 FPS).
SFT video duration is limited to 10.6–16 seconds, of which 50% is 16 seconds and 50% is 10.6–16 seconds; the 16-second ones are trained at 16FPS, the 10.6–16 second ones at 24FPS.
Audio side: video length is limited to 4–120 seconds; the pre-training sequence ceiling is 30 seconds (750 frames), beyond which random chunking is applied; fine-tuning randomly samples 10-second and 30-second clips; captions are likewise produced at both 10-second and 30-second chunk granularities, sampled during training at a 5-batch : 1-batch ratio.

### [NVIDIA NeMo Curator](../models/NeMo_Curator.md) ⚠️

【Splitting strategy (framework capability)】Provides two configurable clip-extraction modes: (1) fixed-stride splitting (hard-cutting by a fixed number of seconds, with configurable clip length and stride); (2) scene-change-detection splitting (based on TransNetV2, shot-aware). The two can be stacked — first split by shot, then further subdivide overly long shots by fixed stride.
【Duration thresholds in production practice (Cosmos WFM)】Clips shorter than 2 seconds are discarded after splitting; clips longer than 60 seconds are further subdivided; the final clip-duration distribution range is 2–60 seconds. The processing granularity for captioning is "one caption per 256 frames" — i.e., an overly long clip is split into multiple caption segments.
【Distribution values】The specific share of samples in each duration range is not published. The framework itself does not enforce any particular duration distribution; that is determined by user configuration. [Uncertain]

### [OmniHuman Dataset + OHBench](../models/OmniHuman.md) ⚠️

【Distribution plot】Clip-duration distribution is given by Figure 3(d); the main text gives no specific percentages, mean, median, or quantiles. [Uncertain]
【Back-derivable average】From 1,800 hours / 1 million clips, one can derive an average of roughly 6.5 seconds/clip, pointing to a distribution dominated by short clips of 5–10 seconds. This order of magnitude closely matches the native generation windows of current AV joint-generation models (Ovi 5 seconds, LTX-2 ~10 seconds, Wan family 5 seconds), suggesting the splitting granularity was designed around downstream training needs.
【Splitting strategy】Duration is naturally determined by TransNetV2's shot-splitting results rather than a manually set fixed-length window — i.e., one clip corresponds to one complete shot (see shot_segmentation). This is the opposite of the "cut fixed windows first, then check for shot crossing" route; the benefit is that each clip is inherently free of transitions internally, at the cost of an uncontrolled duration distribution that requires additional upper/lower bound trimming (the paper does not state whether such bounds were set). [Uncertain]
【Implicit constraint on the duration floor】The audio-governance stage removes samples with "abnormal duration," indicating that duration admission rules do in fact exist, but the specific values are not disclosed. [Uncertain]
【Per-sample duration is queryable】The HuggingFace metadata includes fps, duration, and resolution fields for each sample, as well as clip_start_sec / clip_end_sec, so the full duration distribution is empirically measurable by users — which partly compensates for the paper's disclosure gaps.

### [Open-Sora Series](../models/Open-Sora.md)

【Open-Sora 2.0】The preprocessing stage first removes raw videos shorter than 2 seconds; after shot cutting, clips longer than 8 seconds are forcibly split into multiple 8-second segments, and clips shorter than 2 seconds are discarded, so training-clip durations fall strictly within the [2s, 8s] range. Paper Figure 3 statistics show a distribution markedly skewed right, with **nearly half of clips concentrated in the 6–8 second range**. Output clips are also constrained to fps<30.
【Open-Sora 1.x】Clip durations of 2–16 seconds, paired with a bucket mechanism that buckets by frame count, supporting variable-length training from 2s to 15s.
【Open-Sora Plan v1.3】Uses ffmpeg to uniformly cut into 16-second clips, then applies LPIPS-based jump-cut detection, retaining only clips whose frame count falls within [32, 512] (roughly 1.3–21 seconds at 24fps).
【Open-Sora Plan v1.5】Video training progressively increases from 57 frames @24fps (≈2.4 seconds) to 121 frames @24fps (≈5 seconds).

### [Ovi](../models/Ovi.md) ⚠️

The strategy is "unified fixed length, no multi-duration bucketing" — one of Ovi's most distinctive data-design features.
【Audio-video data】After scene-detection splitting, a uniform fixed-length clip of 121 frames @24fps (= 5.04 seconds) is taken; both the initial training and inference use this length; Ovi 1.1 extends this to 10 seconds (corresponding to roughly 241 frames @24fps) [the frame count for 10 seconds is inferred, uncertain].
【Audio-only data】A dual-duration design: pre-training uses variable-length waveforms up to 12 seconds long (the paper explicitly states it will "use variable-length audio to maximize coverage of diverse acoustics," relying on long waveforms to learn long-range consistency of speaker pitch and emotion); fine-tuning uses waveforms padded to exactly 5.04 seconds, to align with the duration of the 121-frame video and avoid needing to re-adapt temporally when entering the audio-video fusion stage.
【Design rationale】The paper explains that applying scaled RoPE uniformly across all attention layers is meant to "avoid re-adaptation when transitioning to the audio-video fine-tuning stage, and avoid maintaining multiple audio RoPE scales."
【Self-acknowledged limitation】The paper's Limitations section explicitly acknowledges the model is restricted to short 5-second clips, with minute-scale narratives, inter-shot transitions, and global story consistency all out of scope, and proposes future work using a chunk-wise causal audio model plus a causal video backbone conditioned on the final frame of the previous segment to stitch together multiple 5-second chunks.

### [Script-a-Video](../models/Script-a-Video.md) ⚠️

The paper discloses no clip-duration information whatsoever: it gives no average duration for the 500K clips, no duration bounds, no duration histogram, and does not state whether fixed-length truncation or bucketing is applied after splitting.
Indirectly inferable clues:
1) The MTSS Shot stream uses "time_range" as the anchoring field for every shot, and the Event stream likewise carries "time_range," with intra-description timestamps embedded inside the Shot's visual_description to anchor micro-actions to the global timeline — indicating each clip has an explicit global timeline long enough to accommodate multiple shots and multiple audio events;
2) On the generation side, single-shot (125 evaluation items) and multi-shot (100 evaluation items) categories are distinguished, with multi-shot samples necessarily longer than single-shot ones;
3) The evaluation metrics include Shot Boundary Deviation (measured in frames, best value 0.38 frames), indicating the generation side has a definite duration/frame-rate configuration, but the specific values are not disclosed.
The splitting strategy is likewise undisclosed — the paper does not explain how the 500K clips were cut from long videos (whether PySceneDetect was used, whether scene-based splitting was applied, whether manual selection was involved). This is one of the weakest data-disclosure aspects of this work. [Uncertain]

### [Seedance 2.0 and Seedance 1.5 pro](../models/Seedance_20_Seedance_15_pro.md) ⚠️

[Uncertain] Neither 1.5 nor 2.0 discloses the training-clip duration distribution. Seedance 1.0 can be referenced: raw long videos, after shot-aware temporal splitting, produce clips with a maximum duration of 12 seconds; the first pre-training stage jointly trains on 3–12 second video clips (12 fps) and 256px images. The duration coverage of the underlying data can be back-inferred from generation-side capability: 1.5 pro supports roughly 10-second-class generation, Seedance 2.0 supports 4–15 second direct audio-video generation, and the official blog mentions 15-second high-quality multi-shot audio-video output, implying the training data already covers multi-shot clips on the order of 15 seconds.

### [SkyReels Series](../models/SkyReels.md) ⚠️

【SkyReels-V2】No duration histogram is given, but it explicitly adopts "dual-axis bucketing": a BT×BAR bucket matrix is constructed along the two dimensions of temporal length and spatial aspect ratio, with samples organized by their assigned bucket and FPS normalized accordingly; the splitting granularity is single-shot clips (all raw videos are first split into single-shot segments via shot-boundary detection). The Diffusion Forcing stage further supports variable-length, even theoretically unlimited-length generation (the paper markets "unlimited duration," with 30-second/60-second-class clips generated stably in practice). Specific bucket boundaries and per-bucket shares are not published.
【SkyReels-V4】The unified duration ceiling for training and generation is 15 seconds; Stage3 explicitly states the training-clip duration range as "2–15 seconds." Explicit duration control is applied on the audio side: long audio is cut into 15-second chunks, and overly short audio is concatenated by category up to 15 seconds — the most concrete duration-handling strategy in this entry. The per-range sample shares on the video side are not disclosed. [Uncertain]

### [Sora 2](../models/Sora_2.md) ⚠️

The training-data clip-duration distribution and splitting strategy are entirely undisclosed. Only the inference-side output specs are known: initially 10 seconds; after the October 2025 update, all users get 15 seconds, with the ChatGPT Pro web version supporting up to 25 seconds; the Sora 2 Pro API supports 10s/15s/25s tiers. There is no information at all on how training-clip durations are bucketed or how long videos are split into training clips. (The predecessor Sora 1 stated it trained on native durations without uniform cropping to a fixed frame count; whether Sora 2 continues this is unconfirmed.) [Uncertain]

### [SpeakerVid-5M](../models/SpeakerVid-5M.md)

Uses variable-length splitting, with the length band strictly confined to a narrow interval:
【Splitting strategy】PySceneDetect is first used for scene splitting, then clips are trimmed to between 3 and 14 seconds; overly fragmented clips shorter than 3 seconds and overly long clips longer than 14 seconds are excluded or truncated.
【Estimated average duration】Single-person branch: 8.7K hours / 5.2M clips ≈ 6.0 seconds/clip; dialogue branch: 1.8K hours / 770K pairs ≈ 8.4 seconds/pair (if counted per single stream within a pair, roughly 4.2 seconds per stream). Overall: 8,743 hours / 5.2M clips ≈ 6.05 seconds, sitting toward the lower part of the 3–14 second range.
【Comparison with fixed-duration schemes】Unlike MOVA's fixed 8.05-second window, SpeakerVid-5M retains a variable-length character driven by original semantic boundaries, with split points jointly determined by scene changes and speaker turns, more closely matching the granularity of "one natural conversational turn."
【Distribution plot】Figure 3 gives a duration-distribution histogram, but the main text does not give the specific percentage for each duration bucket.
【Temporal span of the multi-turn branch】The multi-turn branch takes a historical window of length T before the current turn's timestamp x, [x−T, x], to aggregate prior turns; adjacent clips with a time gap below threshold δt are judged to be continuous dialogue and can be concatenated into longer, more natural dialogue sequences, so the effective context duration of the multi-turn branch can far exceed the single-clip 14-second cap.

### [Step-Video-T2V](../models/Step-Video-T2V.md)

No duration histogram is given, but the splitting and bucketing strategy is explicit:
【Splitting】PySceneDetect's AdaptiveDetector function detects scene changes, and FFmpeg then splits the video into single-shot clips; each extracted clip additionally has its first and last 3 frames dropped, to eliminate unstable camera motion and residual transition frames near cut points — a fine-grained but practical engineering detail that many contemporaneous works skip.
【Duration expression】The training side expresses duration in frame counts rather than seconds, using frame-length bucketization: four tiers of 1 frame, 68 frames, 136 frames, 204 frames (the 1-frame tier is an image, used for joint image-video training). The model's longest single generation is 204 frames; by the paper's own accounting, 204 frames corresponds to roughly the 8-second class (the derivative model Step-Video-TI2V is 102 frames/5 seconds, from which an implied frame rate of roughly 20fps can be back-derived).
That is, the "short → long" curriculum design is made an explicit frame-count bucketing axis, rather than a fixed clip duration.

### [UniTalking](../models/UniTalking.md) ⚠️

The paper discloses no clip-duration information at all: no minimum/maximum duration constraint is given, no average duration is given, no duration histogram is given, and it is not stated whether clips are uniformly truncated to a fixed length after splitting. Indirect clues that can be inferred:
【Video side】The video branch inherits Wan2.2-5B, whose native generation spec is 5 seconds / 24 fps (121 frames), so training clips are very likely regularized to a roughly 5-second fixed window, though the paper provides no textual support for this. [Uncertain]
【The one hard number on the audio side】The reference audio is explicitly constrained to between 3 and 5 seconds — the only quantitative duration specification in the entire paper, but it constrains the synthesized conditioning signal, not the duration of the main body of training samples.
【Latent-length constraint】The sequence length on the conditioning side is fixed: text latent fixed at 512, reference-audio latent fixed at 257, with truncation for overlength and zero-padding for underlength. The reference-audio latent length of 257, combined with the MMAudio VAE's temporal resolution, corresponds to a capacity ceiling of roughly 8 seconds, meaning the 3–5 second reference audio leaves considerable zero-padding headroom. The latent length of the main generation stream's audio-video is not disclosed. [Uncertain]
【Splitting strategy】No splitting strategy is mentioned at all (see shot_segmentation).

### [UniVerse-1](../models/UniVerse-1.md)

【Duration of the splitting output】Based on PySceneDetect scene splitting, clips shorter than 5 seconds after splitting are removed entirely, so the retained clips have a duration floor of 5 seconds. The paper gives no upper bound, mean, or duration histogram, and does not state whether a secondary truncation is applied to long clips, so what is actually retained is a variable-length clip set rather than fixed-length clips.
【Duration at training time】Rather than using the full clip directly, the online annotation pipeline randomly draws a fixed-length window from the clip at each sampling step (the paper gives 5 seconds as an example) for training. That is, a two-level design of "variable length at ingestion (≥5s), fixed length at consumption (~5s)," with the random window position itself providing data augmentation.
【VGGSound/AudioSet】Follow a simplified processing route, likewise applying the 5-second minimum-duration constraint.
【Relationship to video frame rate】Video is uniformly 25 fps, with 5 seconds corresponding to 125 frames; the audio sampling rate is specifically lowered to 25.6 kHz to align with the 25 fps time grid.
【Comparison】Compared with MOVA's strict fixed 8.05-second duration (193 frames @24fps) plus VAD-aware windowing, UniVerse-1's splitting strategy is noticeably coarser: there is no VAD-driven adaptive window start, and there is no guarantee that a window will not truncate an utterance.

### [Unison](../models/Unison.md) ⚠️

The paper gives no duration-distribution data, splitting strategy, or duration thresholds directly; the vast majority of the content in this field is estimation and indirect inference.
【Estimable average duration】Back-derived from published numbers: audio-video side, 3,000 hours ÷ 2 million clips ≈ 5.4 seconds/clip; audio-only side, 130,000 hours ÷ 50 million segments ≈ 9.4 seconds/segment. The ~5.4-second average clip duration indicates a short-clip corpus, consistent with common practice in the audio-video joint-generation field (UniVerse-1's ~5-second window, MOVA's fixed 8.05 seconds).
【Splitting strategy】Entirely undescribed. The paper neither mentions any shot-splitting tool nor states whether it reuses the existing splits of upstream datasets or re-splits from scratch. The reasonable inference is the former — the five source datasets OpenHumanVid, HDTF, VFHQ, CelebV-Text, and VGGSound are themselves released as "already-split clips" (e.g., VGGSound is fixed 10-second clips, VFHQ is already-cropped face-video clips), so Unison's "automated processing pipeline" more likely performs filtering and re-cropping on top of existing clips rather than re-splitting from long videos. [Uncertain]
【Duration configuration at training time】The frame-count/second configuration per sample during Stage 2 joint training is not disclosed; only that the output is 25 FPS is known. At roughly 5 seconds, this would correspond to roughly 125 frames. [Uncertain]
【Lower bound/upper bound/histogram】None are given.
【Discrepancy with VGGSound's stated spec】VGGSound's original release is uniformly 10-second clips; if used as-is, this should pull the average duration up, yet the measured mean is only 5.4 seconds, suggesting the pipeline applied secondary cropping to clips, or that VGGSound accounts for only a small share of the 2 million clips. This is an inference with no support from the original text. [Uncertain]

### [Veo 3 / Veo 3.1](../models/Veo_3_Veo_31.md) ⚠️

[Uncertain] The training-side clip-duration distribution and splitting strategy are entirely undisclosed. Only back-inferable from the product side: Veo 3 generates a fixed 8 seconds per run; Veo 3.1 supports 4/6/8-second base clips, extendable via the Extend feature (up to roughly 60 seconds or even beyond 140 seconds within Flow). The base generation unit being fixed on the order of 4–8 seconds indirectly implies the training-data body consists mainly of shot-split second-scale short clips rather than long videos.

### [Vidu S1](../models/Vidu_S1.md)

Post-splitting training-clip durations are 3–60 second single-shot clips. Splitting strategy: first split along shot boundaries to guarantee single-shot continuity; long shots are further subdivided, with cut points constrained so as not to fall in the middle of speech, to protect the integrity of speech-lip synchronization. No specific distribution histogram or average duration is given for durations.

### [Wan 2.5 / 2.6 / 2.7](../models/Wan.md) ⚠️

Training side (first-hand from Wan 2.1):
- Hard admission threshold: "video duration must exceed 4 seconds" — videos shorter than 4 seconds are removed already at the basic-dimension stage.
- Training-sample duration: all three stages of joint image-text-video training use 5-second video clips (stage one at 192px/16fps, stage two at 480px, stage three at 720px), and post-training likewise uses 5-second clips at 480px/720px. That is, in the Wan 2.1 era, training granularity was fixed at "5-second single-shot clips."
- Wan-Dancer's splitting strategy can serve as corroborating evidence from the same team: raw long videos are split into 5-second clips with 50% overlap between adjacent clips, to strengthen learning of short-timescale motion dynamics.
Generation side (2.5/2.6/2.7 official specs): wan2.5 offers a choice of 5s/10s; wan2.6 and wan2.7 open this up to any integer within [2,15] seconds (default 5 seconds), with the 15-second option officially described as "the longest single-generation duration domestically." The shift from fixed 5s to a continuous 2–15s range implies the training-data splitting has moved from fixed-length clips toward variable-length clips and multi-shot long clips, but the splitting method (PySceneDetect / an in-house shot-aware model) has never been disclosed across the whole series. [Uncertain]

### [Audio-Video Generation Evaluation Benchmark Collection](../models/av_benchmarks.md) ⚠️

【VABench】The duration of tested videos is constrained by each model's default settings rather than mandated by the benchmark: Sora2 is 10 seconds @30FPS; Veo3-fast, Wan2.5, Seedance, Wan2.2, and Kling are 5–8 seconds @24FPS. Synchformer's Desync metric only computes over the first and last 4.8-second windows — this 4.8s window setting is the key splitting strategy for its temporal evaluation.
【AV-SyncBench】The most fine-grained splitting strategy: clip durations are 3–13 seconds; during evaluation, video and audio are uniformly cut into non-overlapping 0.64-second chunks, with audio-visual embeddings extracted chunk by chunk and the mean diagonal similarity taken — 0.64s is the basic unit of its temporal resolution.
【PhyAVBench】11,605 videos / 25.5 hours, averaging roughly 7.9 seconds/video.
【AVBench】The generation duration corresponding to evaluation prompts is not separately specified [uncertain]; the splitting strategy for the OpenHumanVid clips used in training is not disclosed [uncertain].
【Omni-Judge】Sora 2 / Veo 3 default output duration (roughly 8–10 seconds) [uncertain].

### [Video Captioning Model Ecosystem](../models/caption_models.md) ⚠️

The input-duration handling and adaptation capability of captioners (this directly determines what kind of clip it can annotate):
【Short clips (5–20 seconds) are the mainstream adaptation range】The vast majority of generative-model training clips fall in this range, and captioners are optimized accordingly. Koala-36M clips average 13.75 seconds; Foley-Omni uses fixed 8-second clips.
【Frame-sampling strategy is a core engineering parameter, varying greatly across implementations】Open-Sora's PLLaVA samples 4 frames uniformly; Ovi samples 7 key frames plus the full audio track; MAGI-1 empirically settled on "4–12 frames per clip (depending on duration)" as the optimal tradeoff between description accuracy and compute; the CogVideoX teacher chain samples 1 frame every 2 seconds for dense image captioning; Panda-70M's BLIP-2 branch randomly samples a single frame from the 0.3N–0.7N frame range; Aria officially claims it can generate captions for a 256-frame video within 10 seconds (this throughput advantage is why Allegro chose it).
【Dedicated long-video solutions】ShareCaptioner-Video's DiffSW is the only scalable solution designed for videos of arbitrary length — it first generates a detailed caption for the first frame, then processes subsequent frames in temporal order with a sliding window of length 2, at each step taking as input "the previous frame + its differential caption + the current frame" and outputting the inter-frame change (covering camera motion, object motion, character actions, scene transitions); its complexity grows linearly rather than quadratically with frame count, and it supports clip summarization (quickly summarizing already-processed clips without reprocessing frames). AVoCaDO's optimal output-length range is 2048–4096 tokens, with lengths beyond 4096 penalized by an ℛ_L length regularizer. AVSCapBench video durations are 30–120 seconds, representing a shift in evaluation focus toward medium-to-long videos.
[Uncertain] None of the captioners publish a duration-distribution histogram for their training data.

### [Geometric/Structured Annotation Dataset Collection](../models/geometric_datasets.md) ⚠️

SceneScribe-1M: spec filtering requires durations between 5 seconds and 1 minute; videos judged "discontinuous" are re-filtered after TransNetV2 shot splitting; the final clips average roughly 14.4 seconds (156.7M frames/1 million clips, an approximation computed via fps) [uncertain]; SpatialVID: uses a modified PySceneDetect to split into 3–15 second clips, averaging roughly 9.4 seconds (127.6M frames/2.71M clips at roughly 30fps, back-derived); WildWorld: training samples are fixed at 81-frame clips (recorded at 30FPS, inference at 16FPS), organized by action sequence; Action100M: hierarchical temporal segmentation, discarding clips shorter than 0.5 seconds, with the LLM-aggregation stage further removing nodes shorter than 4 seconds, forming a multi-level temporal tree from second-scale to minute-scale.

### [Video-Generation Post-Training Data Strategy](../models/post_training_data.md) ⚠️

【Anchor paper】The SFT data duration distribution and splitting strategy are not disclosed [uncertain]. Only from self-distillation via autoregressive rollout (Self-Forcing + KV-cache frame-by-frame rollout) can one infer the deployment target is streaming long-video generation.
【Cross-model comparable numbers】Movie Gen's video SFT set duration is 10.6–16 seconds with 50% at 16 seconds (noticeably longer, favoring complete narratives); Allegro's SFT strictly limits to 6–16 seconds; MAGI-1's final stage relaxes to ≤16s; both LongCat-Video's SFT and GRPO use 93 frames; Cosmos-Predict 2.5's driving multi-view SFT is 20 seconds at 30FPS.
【Pattern】Post-training stages generally push the duration ceiling to the limit of that model's capability (around 16 seconds) and favor longer clips, the opposite of pre-training's preference for short clips (2–8 seconds) — because SFT needs to teach "the narrative and motion completeness of a full shot," whereas RLHF, due to extremely high rollout cost (the anchor paper explicitly states "rollout generation is expensive"), tends toward shorter clips instead.

### [Survey of Mainstream Video Pre-training Datasets: Panda-70M, InternVid, Koala…](../models/pretraining_datasets.md) ⚠️

**Clip duration is the most fundamental point of divergence among the seven, directly corresponding to the tradeoff between "short clips being easy to learn from" and "long clips being scarce"**, spanning nearly an order of magnitude from 8.5 seconds to 72 seconds:
- **Panda-70M (8.5 seconds, shortest)**: after splitting, clips shorter than 2 seconds are discarded, and source videos longer than 60 seconds are truncated to the first 60 seconds; another key operation — **the first and last 10% of each clip is trimmed** to remove unstable camera motion and transition residue at the edges. The final average is 8.477 seconds.
- **InternVid (11.7 seconds)**: crawling is restricted to source videos of 10 seconds–30 minutes; clip durations range from 2 seconds to over 30 seconds, with **85% of clips falling in the 0–10 second range**.
- **Koala-36M (13.75 seconds, or 17.2 seconds under the 172K-hour accounting)**: clips are naturally cut out by transition detection, with no fixed-length truncation; the assembly rule requires a clip to be longer than **8 frames** to be retained, with **4 frames eroded from each end** (cuts.append((start+4, i-4))) to ensure no transition remains within the content — the same idea as Panda-70M's 10% edge-trimming, implemented differently.
- **OpenVid-1M (7.2 seconds, OpenVidHD 9.6 seconds)**: the minimum duration threshold is not disclosed. [Uncertain]
- **UltraVideo (short 5.3 seconds / long 30.9 seconds)**: an **explicit dual-split design** — after shot cutting, clips are routed by frame count, with 3–10 seconds going into the short set and >10 seconds into the long set. To expand the short set, targeted extraction is also performed: source videos under 60 seconds have their **middle 10 seconds** taken, and those over 60 seconds additionally have **10 seconds taken from each side**. Native frame rate is preserved, with no frame-rate resampling.
- **LVD-2M (20.2 seconds, minimum threshold 10 seconds)**: takes "**≥10 seconds**" as its foremost design goal (the first of four criteria), with no upper-bound truncation, hence a rich long tail (clips >30 seconds account for roughly 13.5%). At captioning time, videos longer than 30 seconds are cut into 30-second segments and described separately.
- **MiraData (72.1 seconds, longest)**: the strategy is the opposite of all the other datasets — **rather than cutting more precisely, it cuts and then stitches back together**. Clips sourced from YouTube must be **>40 seconds** to be retained, and those from Videvo/Pixabay/Pexels must be **>10 seconds**. (The v0 release used to hard-cut anything over 2 minutes into 2-minute segments; v1 switched to a concatenation strategy.) The official 100-sample set measured in practice averages 105.1 seconds, longer than the dataset-wide average.
**Summary pattern**: three distinct routes for trading clip length against quality are clearly visible — Panda/InternVid/OpenVid follow "short and plentiful," Koala follows "medium-length with clean transitions," and LVD-2M/MiraData/UltraVideo-long follow "long shots as a scarce resource."

## Resolution/Aspect-Ratio Distribution and Bucketing Strategy

`resolution_aspect_distribution` · Level of detail: brief

### [Allegro](../models/Allegro.md) ⚠️

Resolution thresholds layered by training stage (Table 1):
· T2I pre-training and T2V 360p pre-training: width W≥640 and height H≥368 (corresponding to a 368×640 training resolution);
· T2V 720p pre-training and fine-tuning: width W≥1280 and height H≥720 (corresponding to a 720×1280 training resolution);
· The raw-ingestion entry point additionally applies a minimum threshold of ≥360p.
Training resolution is not multi-bucket dynamic-resolution training but rather two fixed tiers (368×640 → 720×1280), following a "low-res → high-res" progressive curriculum. On the aspect-ratio side, the paper discloses no bucketing strategy or multi-aspect-ratio training; all training resolutions are 720×1280 / 368×640 expressed as 16:9 portrait framing. [Uncertain] (aspect-ratio distribution and bucketing strategy)

### [Apollo](../models/Apollo.md) ⚠️

The paper discloses no resolution or aspect-ratio distribution or bucketing strategy: no training resolution is given, no aspect-ratio enumeration is given, and no multi-resolution bucketing or progressive resolution-upgrading curriculum is mentioned. Only two related points exist: (1) the filtering stage will "discard those videos with low resolution," but no lower resolution threshold is given; (2) the Video-VAE adopts CogVideoX's 3D causal visual encoder, applying 16× compression to both height and width, and states it can handle "varying resolutions and frame rates" — i.e., the architecture supports variable-resolution input, but the resolution configuration actually used in training is not disclosed. [Uncertain]

### [CineDance / CineDance-1M](../models/CineDance.md) ⚠️

【Resolution floor】A hard requirement of at least 1080p, one of this dataset's key selling points relative to comparable works — in the paper's comparison table (Tab.6), CineDance-1M is the only one simultaneously satisfying all three of "1080p + native audio track + shot-level dense bimodal annotation."
【Spatial preprocessing】Two passes of "coarse cropping + fine verification": the coarse-crop stage uses EasyOCR to locate and remove burned-in subtitle regions, and FFmpeg black-border detection to remove letterboxing; after shot splitting, each individual clip is re-run through OCR and black-border detection for a fine-grained re-check.
【Aspect-ratio distribution / bucketing strategy】The paper gives no statistical distribution of aspect ratio and does not describe a resolution-bucketing strategy for training; given the material is predominantly cinema-grade film content, it is presumed to be dominated by widescreen ratios, but this is unsupported by data. [Uncertain]

### [CogVideoX](../models/CogVideoX.md) ⚠️

· Resolution curriculum (progressive training): first trained at 256px to learn semantics and low-frequency knowledge, then progressively raised to 512px and 768px to learn high-frequency detail. The training hyperparameter table gives the maximum resolution per stage: stage1 256×384 → stage2 480×720 → stage3 768×1360 → stage4(FT) 768×1360. CogVideoX-5B ultimately outputs 720×480; CogVideoX1.5-5B outputs 1360×768, 10 seconds, 16fps; the product version, the new "Qingying," can reach 4K/60fps.
· Aspect ratio: explicitly preserves the original aspect ratio, resizing only the short side to the target resolution ("we keep the aspect ratio unchanged and resize the short side to above resolutions"), thereby preserving the ability to generate video at arbitrary aspect ratios; CogVideoX1.5-5B-I2V supports arbitrary output resolution. No landscape/portrait mixture ratio is given. [Uncertain]
· Bucketing vs. packing: does not use SDXL-style bucketing; instead uses a Multi-Resolution Frame Pack to pack samples of different resolutions and durations into the same batch.
· When adapting RoPE for high resolution, both interpolation and extrapolation were compared, with extrapolation ultimately chosen to preserve local detail and relative positional relationships.

### [Cosmos-Predict2.5](../models/Cosmos-Predict25.md) ⚠️

(1) Bucketing: the sharding stage explicitly shards along the two axes of resolution and aspect ratio, together with content type and length for four axes total, aimed at supporting "efficient sampling, curriculum-based training, and fine-grained domain balancing" — so an explicit resolution/aspect-ratio bucketing mechanism exists, but the share of each bucket is not published.
(2) Resolution is used as the arbitration criterion for deduplication: during semantic deduplication, the highest-resolution version within a cluster is retained, on the grounds that higher resolution preserves finer visual detail and provides richer training signal; in online incremental deduplication, "earlier + higher resolution" serves as the tie-breaking priority.
(3) The training-side resolution curriculum is explicit: 256p (320×192) → 480p (832×480) → 720p (1280×704), progressively increasing, moving to the next tier only once the model has converged and visual quality has saturated at the current tier.
(4) The cooldown stage uses a dedicated batch of curated 4K high-definition videos (388K clips), with learning rate linearly decayed to 0, used to improve detail fidelity and motion smoothness.
(5) Architecturally, absolute positional encoding is removed, retaining only relative positional encoding (3D RoPE), explicitly intended to improve generalization to resolutions and sequence lengths unseen during training.

(6) Robotics-domain pre-filtering removes low-resolution videos.
[Uncertain: the share of each resolution/aspect ratio in the original corpus]

### [Data-Juicer 2.0](../models/Data-Juicer.md) ⚠️

【Related operators】Provides fairly comprehensive filtering and rewriting operators:
  · video_resolution_filter — filters by resolution upper/lower bounds, removing low-definition material.
  · video_aspect_ratio_filter — filters by aspect-ratio range, removing extreme long-strip/portrait or anomalous-ratio samples.
  · video_resize_resolution_mapper — rewrites resolution according to width/height constraints (rather than discarding), usable to unify mixed-resolution data to a target spec.
  · video_resize_aspect_ratio_mapper — resamples video to within a specified aspect-ratio range.
  The "filter + rewrite" dual-path design is a feature of DJ relative to purely filter-based frameworks: non-compliant samples can be repaired rather than simply discarded, which is friendlier for scenarios where data is precious.
[Uncertain] DJ does not build in the "resolution bucketing" functionality commonly used in video-generation training — i.e., grouping samples by resolution/aspect ratio for same-batch training — that responsibility belongs to the training framework side (e.g., EasyAnimate, Diffusers); DJ is only responsible for organizing the data into a bucketable state. The official T2V case study also does not disclose resolution or aspect-ratio distribution statistics for the data pool.

### [Foley-Omni](../models/Foley-Omni.md) ⚠️

Only a filtering floor is given, no distribution statistics. The filtering stage sets a hard visual-resolution threshold of ≥480p, paired with a bitrate threshold of ≥1 Mbps (a dual check to prevent "falsely high-definition" samples with high resolution but low bitrate from slipping in — this bitrate constraint is relatively rare among comparable works; UniVerse-1 uses a bitrate ratio with a similar idea).
[Uncertain] No resolution-distribution histogram is disclosed for the final dataset, no aspect-ratio distribution is disclosed, and no resolution or aspect-ratio bucketing strategy is mentioned. This has relatively limited impact for this project: since video is only used as a conditioning input and is encoded into fixed-dimensional features via CLIP and Synchformer, resolution mainly affects feature-extraction quality rather than generation resolution, so strict bucketed training as required by text-to-video models is unnecessary.

### [Goku](../models/Goku.md) ⚠️

【Admission floor】min{height, width} ≥ 480, i.e., the shorter side must be no less than 480 pixels.
【Resolution bucketing】Bucketed into three tiers — 480p / 720p / 1080p — each with its own progressively tightened filtering thresholds (paper Table 4); higher-resolution tiers have stricter thresholds → data volume shrinks tier by tier: 36M at 480p, 24M at 720p, 7M at 1080p. This "higher resolution, higher quality bar, less data" bucketing design directly serves the progressive-resolution training curriculum.
【Training-resolution sequence】288×512 → 480×864 → 720×1280, all 16:9 landscape ratio; the I2V/high-resolution stages involve 1080p data.
【Aspect ratio】The paper organizes training around fixed 16:9 resolutions (288×512, 480×864, 720×1280) and does not describe a multi-aspect-ratio bucketing strategy, nor does it give the portrait/square data share. [Uncertain] (aspect-ratio distribution and whether arbitrary-ratio training is supported)

### [Hailuo / MiniMax Video](../models/Hailuo.md) ⚠️

[Uncertain] (again, only output-side specs are available).
Output side: the original video-01 is 1280×720 / 25fps; Hailuo 02 claims "native 1080p" and offers two tiers, 768p and 1080p; Hailuo 02 Fast is a low-cost 512p tier; the Hailuo 2.3 documentation lists 1080P.
Training side: the resolution distribution, aspect-ratio distribution, and whether a multi-resolution/multi-aspect-ratio bucketing training strategy is used are entirely undisclosed officially. The phrase "native 1080p" implies high-resolution data holds a substantial share of the training set (rather than relying purely on super-resolution post-processing), and a low-to-high resolution curriculum very likely exists, but there is no first-hand confirmation.

### [HunyuanVideo-Foley](../models/HunyuanVideo-Foley.md) ⚠️

The paper does not discuss video resolution, aspect ratio, or bucketing strategy at all — a structural feature determined by the nature of the V2A task: visual information serves only as conditioning input and is encoded via SigLIP-2 into a fixed-dimensional semantic feature vector, so the original resolution is flattened away after encoding. Consequently, none of the "training resolution determines output resolution" constraints that apply to video-generation models apply here, and no resolution bucketing or aspect-ratio alignment is needed.
【Actual constraint】The only visual-side processing is the fixed input size of the SigLIP-2 visual encoder (typically a 224×224 or 384×384 square crop/resize), plus Synchformer's input spec. The paper does not disclose which SigLIP-2 variant (so400m / base / large) or input resolution is actually used. [Uncertain]
【Frame rate】The paper does not state the sampling frame rate of video frames — quite important for sync accuracy (Synchformer's frame-level sync feature density is directly affected by the sampling frame rate) — but no value appears in either the main text or appendix. [Uncertain]
【No visual quality gate in the filtering funnel】Notably, the entire cleaning pipeline contains no filtering condition targeting visual image quality (no resolution floor, no bitrate constraint, no aesthetic score, no blur/compression-artifact detection) — all quality gating is concentrated on the audio side. This is a stark contrast to UniVerse-1's strict visual gate of "resolution ≥1080p + bitrate ratio ≥600 + DOVER ≥0.6," and the direct reason is that this model does not generate imagery — image quality does not affect output fidelity, and only visual semantic recognizability matters. This is a reasonable simplification derived from the task's form.

### [HunyuanVideo](../models/HunyuanVideo.md)

【Original version】Resolution itself is the axis of the layered filtering funnel: four progressively stricter datasets are constructed — 256p → 360p → 540p → 720p — plus one SFT multi-scale dataset, with filtering getting stricter at each higher tier. The training side uses a bucketing strategy that simultaneously supports multiple resolutions and multiple aspect ratios (and supports 1–129 variable frame counts), enabling generation of video at arbitrary aspect ratios. Specific per-aspect-ratio shares are not published.
【1.5】More clearly staged: image 256p → 512p; video 256p → 480p → 720p, with the CT/SFT stages training on 480p and 720p simultaneously; a separate super-resolution module is trained on 1K–4K data, lifting 480p/720p base-model output to the 1080p class. Aspect-ratio bucketing details and shares are not disclosed.

### [InstructAV2AV](../models/InstructAV2AV.md) ⚠️

Similarly highly unified: everything is 720p / 24 FPS. Resolution is jointly determined by the data engine's synthetic backbone (Wan2.2-5B, i.e., Wan2.2-TI2V-5B) and the model's training resolution — a case of "synthesis fixes the spec," which removes the need for bucketing the way real data requires.
【Quality-floor control】A LAION Aesthetics Predictor (Romain and Christoph, 2022) is used to filter visually low-quality clips, but [Uncertain] the paper gives no specific aesthetic-score threshold.
[Uncertain] The aspect-ratio distribution is not disclosed — judging from the sources, film-type material (MovieBench, Condensed Movies, Short-Films-20K) is mostly widescreen 21:9 or 16:9, while VGGSound and YouTube material is mostly 16:9, so it is presumed the final result is unified to a single 16:9 ratio (720p typically means 1280×720), but the paper does not state whether this was done via cropping, resizing, or padding with black bars, nor does it mention any bucketing strategy.
【Difference from text-to-video models】Text-to-video models need multi-resolution, multi-aspect-ratio bucketed training to support arbitrary output sizes; this work, as an editing model with entirely synthetic data, can use a single spec, an engineering simplification dividend of the synthetic-data route, at the cost of the model's generalization to non-720p/16:9 inputs remaining unverified.

### [2026 Other Audio-Video Joint Generation Works](../models/JAVG_2026_misc.md) ⚠️

Only a few of the seven disclose this, and most adopt a two-tier "low-res training + high-res refinement" strategy rather than bucketing:
【ALIVE】Explicitly two-tier: the base model trains on 11M samples at 480p/24fps, with a separate 1080p Refiner stage trained for 1 epoch on 0.7M "high-clarity samples." This is a typical "large-scale low-res groundwork + small-scale high-res refinement" curriculum. Aspect-ratio distribution and bucketing strategy are not disclosed [uncertain]. In addition, ALIVE has a dedicated clarity-filtering stage, using "reference images across six distinct clarity levels" (six clarity tiers of reference images, from blurry to sharp) as the evaluation benchmark — essentially turning resolution/clarity into six ordinal labels rather than a continuous score, an approach relatively rare among contemporaneous works.
【OmniCustom】Fully unified normalization, no bucketing: "All videos are recorded in 480p at 24 FPS. We extract audio files from videos and unify them into 16kHZ." — video uniformly 480p/24fps, audio uniformly 16kHz. This is consistent with its narrow task positioning of single-person talking heads.
【CCL】Training resolution 352×640p (a non-standard resolution close to 9:16 portrait); the only work among the seven to give a precise pixel size; single-resolution training, no bucketing.
【NAVA】Training-side resolution distribution is not disclosed [uncertain]; the inference side defaults to 704×1280 (9:16 portrait), and the repository states "flexible aspect ratios supported," indicating multi-aspect-ratio capability, but the training-side mixture ratio is not stated.
【ITS-JAVG (evaluation baselines)】JavisDiT is 240p, MMDisCo is 256×256 — reflecting that the resolution of open-source academic JAVG baselines still lags significantly behind industry.
【Baton, StreamChar】Resolution/aspect-ratio strategy not disclosed [uncertain].
【Common trait】None of the seven adopt Ovi-style "equal-area normalization" or Seedance-style multi-resolution bucketing scheduling; most are single-resolution training plus optional high-res refinement, indicating this batch of works has a relatively modest compute budget and engineering complexity.

### [Audio-Video Joint Generation Baseline Collection](../models/JavisDiT_baselines.md) ⚠️

The resolutions of all five works are far below industrial models, and all use a fixed resolution with no aspect-ratio bucketing:
【MM-Diffusion】The base diffusion process operates in a 64×64 pixel space, then upsampled to 256×256 via an independent super-resolution (SR) model; the repository provides both base models (Landscape.pt / AIST++.pt) and their corresponding SR models (Landscape_SR.pt / AIST++_SR.pt). This "low-res generation + independent SR" two-stage approach was mainstream in 2022. Aspect ratio is fixed at 1:1, no bucketing.
【AV-DiT】Video frames are cropped to 256×256 resolution; the video latent size is 32×32×4, and the audio latent (mel spectrogram) is 40×16×8. Single resolution, single aspect ratio.
【JavisDiT / JavisDiT++】Training and evaluation are unified at 240P, 24fps (the data-preparation stage normalizes fps to 16Hz). Although the data CSV retains fields such as height, width, aspect_ratio, and resolution (inherited from Open-Sora's data-management schema, theoretically supporting multiple buckets), all reported experiments use a fixed 240P/4-second configuration, and no actual use of a bucketing strategy is observed [uncertain].
【Harmony】The paper does not specify video resolution or frame rate [uncertain]; the underlying Wan2.2-5B natively supports 720P, so it is presumed to inherit that resolution capability, but this is not confirmed in the text.
【UniAVGen】Only states video is processed at 16 fps before VAE encoding; resolution and aspect ratio are not disclosed [uncertain].
Common trait: none adopts area normalization (like Ovi's fixed pixel count of 518400), and none adopts multi-aspect-ratio bucket scheduling, indicating this batch of works concentrates its compute on cross-modal alignment mechanisms rather than visual fidelity.

### [Kling 3.0 / Kling Video 3.0 Omni](../models/Kling_30_Omni.md) ⚠️

[Uncertain] The resolution/aspect-ratio distribution and bucketing strategy are not published. What is known: the model supports native 4K (3840×2160) 60fps output, with the official statement emphasizing pixel-level native generation within the diffusion process rather than post-hoc super-resolution, implying the training set includes a substantial proportion of genuine 4K/high-frame-rate material; it also supports multiple aspect ratios (16:9/9:16/1:1, etc.), so the industry-standard practice of multi-resolution/multi-aspect-ratio bucketing plus a progressive resolution curriculum is very likely used, though the report does not state this explicitly. Corroborating references: Koala-36M is unified at 720p; Kling-Foley requires source video ≥720P. Kling Avatar 2.0 uses a spatio-temporal cascade (spatial super-resolution + temporal frame interpolation) framework to achieve high resolution/high frame rate, suggesting that within the Kling family, 4K/60fps may be partly handled by a cascaded refinement module. [Uncertain: the boundary between native 4K generation and cascaded refinement]

### [LTX-2](../models/LTX-2.md) ⚠️

No distribution statistics are given, but the strategy is explicit. (1) Joint multi-resolution multi-duration training: training simultaneously spans multiple width/height/duration combinations, and the model generalizes well to unseen configurations; (2) alignment by token count rather than padding/packing: the raw video is resized to a comparable token count, with 0%–20% stochastic token dropping applied to fix the token count of each sequence, which the authors say is simpler and more efficient than complex token-packing/padding while preserving data diversity; (3) aspect-ratio standardization: the pipeline explicitly includes a "Crop Black Bars" step to remove black bars, standardizing aspect ratio and increasing effective visual area; (4) the pipeline includes a "Resize Shots" step. On the inference side, resolution must be divisible by 32 and frame count must satisfy 8n+1. Specific per-resolution/aspect-ratio shares are not disclosed. [Uncertain]

### [LongCat-Video](../models/LongCat-Video.md) ⚠️

The report gives no statistics on the resolution/aspect-ratio distribution of the raw data, nor does it describe an aspect-ratio bucketing mechanism. At the metadata level, resolution, frame rate, and bitrate are recorded and used for filtering. What is truly explicit is the training-side resolution curriculum: 256p (Stage1, Stage2, Stage3) → 480p (Stage4) → a 480p+720p mix (Stage5, SFT, RLHF), i.e., a progressive low-to-high resolution curriculum, with Stage5 onward mixing two resolution tiers within the same stage so the model adapts to both 480p and 720p output simultaneously. The inference side ultimately delivers 720p / 30fps. The model also uses a "spatio-temporal dual-axis coarse-to-fine" generation strategy: first generating a coarse low-resolution, low-frame-rate video, then refining it — a strategy that requires the training data to support multiple resolution tiers simultaneously. The VAE is the WAN2.1 VAE, with a spatio-temporal compression ratio of 4×8×8, giving an overall 4×16×16 after patchify is layered on top. [Uncertain: raw-data resolution distribution and bucketing details]

### [MOVA](../models/MOVA.md)

【Standardization pipeline】Raw video first has black bars detected and removed via the FFmpeg cropdetect filter, retaining the core frame; the main content is then centered, scaled to 720p, and symmetrically padded with black bars as needed (pillarbox or letterbox), uniformly regularizing to one of two aspect ratios, 9:16 or 16:9. Frame rate is uniformly resampled to 24fps. Consequently the data side has only two aspect ratios, and there is no multi-bucket aspect-ratio training.
【Resolution curriculum】Phase 1 and Phase 2 train at 360×640 (360p), Phase 3 upsamples to 720×1280 (720p). This corresponds to the released MOVA-360p and MOVA-720p models.
【Engineering impact】At the 720p stage, sequence length increases substantially; context parallelism increases from CP=8 to CP=16, effective batch size drops from 128 to 64, and the checkpoint interval shortens from 5000 steps to 2000 steps.
【Effect】Ablations show that raising resolution from 360p to 720p causes DeSync to slightly drop from 0.475 to 0.485 and IB-Score to slightly drop from 0.286 to 0.277 (near-zero degradation), while LSE-C actually improves from 6.278 to 6.593 and cpCER drops from 0.177 to 0.149, validating the effectiveness of a curriculum that first establishes cross-modal alignment at low resolution before upscaling.

### [Merged entry: ① Mochi 1](../models/Mochi_MAGI_Motif.md) ⚠️

① Mochi 1: The resolution and aspect-ratio distribution and bucketing strategy of the training data are undisclosed; output is fixed at 848×480 (roughly 16:9). [Uncertain]
② MAGI-1: Training resolution progresses across three stages — stage-1 is 256p/360p, stage-2 is 480p, stage-3 is 720p (Table 5). At inference, the VAE uses a sliding window to support arbitrary resolution (spatial window 256×256, stride 192, 25% overlap; no overlap along the time dimension); the model claims to support arbitrary resolution, with the 4.5B version defaulting to 720×720. But the aspect-ratio distribution and bucketing weights of the training data are not disclosed. [Partially uncertain: aspect-ratio bucketing]
③ Motif-Video 2B: Resolution is the primary layering axis of the entire pipeline — the curriculum is 144p → 360p → 480p → 720p across four tiers and ten stages, with resolution/clip-length/motion/aesthetic filtering re-applied and tightened at every stage transition. Two concrete engineering measures handle aspect ratio: first, ffmpeg cropdetect estimates the maximum content rectangle from luminance statistics, removing letterbox/pillarbox black-bar padding; second, the OCR-detected region and the black-bar crop box are merged into a single final rectangle (excluding the top 20% for logos/watermarks and bottom 20% for subtitles of the content area), applied together with resolution scaling and frame-rate limiting in a single ffmpeg re-encode. Training uses a joint "frame-count bucket × resolution bucket" scheme (frame-count buckets: 1/33/65/121, each further subdivided into multiple spatial resolutions), with an offline bucket-balancing sampler minimizing the coefficient of variation in per-bucket sample counts across ranks. No quantitative aspect-ratio distribution is given. [Partially uncertain: quantitative aspect-ratio distribution]

### [Movie Gen](../models/Movie_Gen.md)

Resolution thresholds tighten with training stage: the low-resolution training stage requires minimum width/height ≥720px; the high-resolution training stage requires minimum width/height ≥768px (Table 44 shows this step cuts the data from 100% to 25%). Training resolution progresses from 256px to 768px, with an independent spatial super-resolution model additionally used to output 1080p HD.
Aspect ratio: the pre-training set is controlled to 60% landscape + 40% portrait (favoring landscape, which tends to have longer duration, better aesthetics, and more stable motion); the high-resolution set is adjusted to 80% landscape + 20% portrait; in the filtering funnel, the "width ≥ height" step cuts the data from 25% to 7%.
Bucketing: both images and videos use five aspect-ratio buckets, with identical latent shapes within a bucket to facilitate batching, so the model can output multiple ratios, e.g., landscape 1024×576, portrait 576×1024.
Black bars are handled separately: an in-house border detector (first-derivative gradient detection plus a scan-line algorithm to locate borders) removes videos with black bars, which are especially common in portrait video.
The audio side has a lower requirement for video quality: it is sufficient to remove videos with resolution <480px.

### [NVIDIA NeMo Curator](../models/NeMo_Curator.md) ⚠️

【Framework capability】Resolution and aspect ratio are not used as filtering conditions but as organizational dimensions at the sharding stage: the final WebDataset is packed into buckets along the three dimensions of "resolution × aspect ratio × duration," with the goal of aligning with the downstream training curriculum's bucketing needs, enabling training to sample by bucket and avoiding excessive token-count variance within a batch. This is the most worth-emulating design in this pipeline regarding "coupling data infrastructure with the training curriculum."
【Transcoding normalization】All clips are uniformly transcoded to H.264 mp4, with a choice of encoder — h264_nvenc (GPU) / libx264 / libopenh264 (CPU); decoder choice of nvdec (GPU) / ffmpeg. The frame-extraction stage (ClipFrameExtractionStage) can be configured with a target fps and extraction strategy (e.g., a sequence strategy for aesthetic scoring).
【Production practice】The input material for Cosmos WFM ranges from 720p to 4K. The specific share of each resolution/aspect-ratio bucket is not published. [Uncertain]

### [OmniHuman Dataset + OHBench](../models/OmniHuman.md) ⚠️

【Admission threshold】The paper explicitly states "all videos are of high-definition quality or higher," i.e., there is a hard resolution floor at 720p or above, but no specific pixel-count threshold is given, nor is it stated whether the judgment is by short side, long side, or total pixel count. [Uncertain]
【Distribution plot】The resolution distribution is given by Figure 3(c); the main text gives no specific share for each tier. [Uncertain]
【Aspect ratio】The paper does not discuss aspect-ratio distribution, does not describe a bucketing strategy, and does not state whether uniform cropping or padding was applied. Given the data source is YouTube covering content types spanning talk shows, gaming, and reviews (mostly 16:9 landscape) as well as cooking and education (possibly including portrait short-form video), the actual distribution is likely landscape-dominant with some portrait mixed in, but there is no data to support this. [Uncertain]
【Frame-rate standardization】Explicitly unified to 30 FPS (together with 44.1 kHz audio as one of the standardized specs). This is one of the few explicit numeric specs in the paper, and frame-rate standardization is a necessary precondition for frame-by-frame annotation (134 keypoints, SMPL/MANO tracking) and audio-visual sync judgment (a ±3-frame offset tolerance only has a well-defined meaning at a fixed frame rate).
【Spatial cropping】During the spatio-temporal cleaning stage, OCR and logo detection are used to estimate text-contaminated regions, and frames are cropped to remove subtitles and watermarks — meaning the final aspect ratio of some samples will deviate from the original ratio due to cropping; the paper does not state how the ratio change after cropping is handled. [Uncertain]
【Per-sample queryable】The metadata contains a resolution field, so users can compute the full distribution themselves.

### [Open-Sora Series](../models/Open-Sora.md)

【Open-Sora 2.0】Preprocessing removes videos whose aspect ratio (height/width) falls outside [1/3, 3], and caps the long side of clips at ≤1080px, with everything uniformly transcoded to H.264. Figure 3 shows the aspect ratio (height/width) **mostly concentrated in the 0.5–0.75 range, i.e., dominated by 16:9 landscape**. Inference supports two tiers, 256px and 768px, covering 16:9, 9:16, 1:1, and 2.39:1.
【Open-Sora 1.x】Uses an explicit **bucketing strategy**: resolution, aspect ratio, and frame count are combined into three-dimensional predefined buckets, with each bucket independently assigned a batch size to balance GPU load, allowing a single training run to mix samples of 144p~720p, arbitrary aspect ratio, and 2~15 seconds. This is one of Open-Sora's most widely reused engineering designs.
【Open-Sora Plan v1.3】Video training is fixed at 93×352×640 (roughly 16:9); v1.5's image side performs multi-resolution training, covering five aspect ratios — (1,1), (3,4), (4,3), (9,16), (16,9) — combined with a Min-Max Token strategy, while the video side is fixed to a 9:16 ratio, up to 121×576×1024.
【Ratio control on the data-sourcing side】Open-Sora Plan explicitly performs landscape/portrait ratio control: Panda70M 21M provides landscape, and VIDAL 3M provides portrait (YouTube Shorts) — a rare example of explicit supplementary portrait-data sourcing.

### [Ovi](../models/Ovi.md) ⚠️

【Filtering threshold】The splitting stage requires clip resolution to be strictly greater than 720×720 pixels (paper's original wording, "clips are greater than 720x720 pixel resolution"); anything below this threshold is discarded directly.
【Normalization strategy】Rather than resolution bucketing, a unified normalization is applied: before packing, existing black bars/margins are first removed from the video, then frames are scaled, while maintaining aspect ratio, to a fixed total pixel count of 518400 (= 720×720). That is, it constrains "area" rather than "edge length," so samples of different aspect ratios can land on the same token count, naturally supporting multiple aspect ratios such as 9:16, 16:9, and 1:1 without bucketing scheduling.
【Ovi 1.1】Switches to training on native 960×960-resolution data (total pixel count raised to 921600); inference supports 960×960 and its equal-area equivalents at various aspect ratios.
【Actual per-aspect-ratio mixture numbers not published】[Uncertain].
【Packing】Final video is frame-extracted at 24fps and converted into byte arrays, with audio converted into raw wave bytes, for high-throughput reading on the training side.

### [Script-a-Video](../models/Script-a-Video.md) ⚠️

The paper does not discuss resolution and aspect ratio at all: no training-data resolution threshold is given, no resolution distribution, no aspect-ratio distribution, no bucketing or multi-resolution training strategy is described, and the output resolution/frame-rate configuration of the generative model is not stated.
The only indirect information is the resolution capability of the underlying generation backbone LTX-2 itself, but the paper does not state the actual training/inference resolution tier used in its modified version.
The evaluation metric Shot Boundary Deviation is reported in "frames" (3.79 → 0.38 frames), indicating a fixed frame-rate setting exists, but the frame-rate value is not given. [Uncertain]

### [Seedance 2.0 and Seedance 1.5 pro](../models/Seedance_20_Seedance_15_pro.md) ⚠️

[Uncertain] The resolution/aspect-ratio distribution statistics and bucketing strategy are not disclosed. Seedance 1.0's progressive resolution curriculum can be referenced: first fully training a 256px text-to-image stage → 256px joint image-text-video training → raised to 640px → finally raised to 24 fps. Seedance 1.0 explicitly lists "resolution" as one of the attribute dimensions whose frequency is tallied and up/down-sampled during Distribution Rebalancing. Seedance 2.0's native output resolution is 480p and 720p (the report specifically notes that its Arena Elo at 720p already surpasses competitors' 1080p models).

### [SkyReels Series](../models/SkyReels.md) ⚠️

【SkyReels-V2】Aspect ratio is explicitly managed via the BAR axis of "dual-axis bucketing" (a duration × aspect-ratio bucket matrix), with FPS standardized accordingly; the resolution dimension is managed via a progressive curriculum: pre-training stage 1 is 256p, stage 2 rises to 360p, stage 3 rises to 540p, and post-training's final SFT raises it to 720p. Basic quality filtering removes low-resolution and low-frame-rate sources, and crops out black bars and split-screen content. Specific per-resolution/aspect-ratio bucket shares are not published.
【SkyReels-V4】Resolution and data volume are allocated by stage: Stage1/2/3 is 256px (16fps), Stage4 mixes 256/480px (100 million each), Stage5 mixes 480/720/1080px (50 million each), Stage6 and SFT use 720/1080px. The generation ceiling is 1080p/32FPS. The aspect-ratio bucketing strategy is not separately explained. Per-resolution sample proportions can be indirectly inferred as roughly equal from the "each tier equal amount" wording above. [Uncertain]

### [Sora 2](../models/Sora_2.md) ⚠️

The training-side distribution and bucketing strategy are not disclosed. Inference-side specs: Sora 2 standard is 720p; Sora 2 Pro supports 1024p and native 1080p (1920×1080), and also supports both portrait (1080×1920) and landscape output. The predecessor Sora 1's technical blog explicitly used "native size training" — no resize/crop/trim to a fixed size, so arbitrary aspect ratios can be natively sampled; it is presumed Sora 2 continues this strategy (variable-resolution/aspect-ratio patch packing), but Sora 2's official materials have never confirmed this, nor is there any bucketing-proportion number. [Uncertain]

### [SpeakerVid-5M](../models/SpeakerVid-5M.md) ⚠️

【Native resolution】The dataset preserves the native resolution of source videos, and the resolution tiers are quite high: 93% of videos are 1080P or higher, and 98% exceed 720P. This is one of its core quality advantages relative to older datasets like CelebV-HQ and HDTF; Figure 3 gives the resolution-distribution plot.
【Cropping strategy】Rather than uniform resolution normalization or black-bar padding, temporal and spatial cropping is applied per speaker based on YOLO-based human detection and tracking results — i.e., a sub-region centered on a single speaker is cropped from the original frame. So the aspect ratio of the actual clip is determined by the human bounding box rather than being fixed at 16:9 or 9:16.
【Body-framing bucketing】The paper uses "body composition" rather than aspect ratio as its primary framing-bucketing dimension, with captions explicitly labeling half-body vs. full-body framing, as well as facing direction/body orientation, covering full-body, half-body, and side-profile framings — a differentiating annotation field in Table 1 relative to datasets like CelebV-HQ, which are mostly head-focused.
【Training side】The baseline model uniformly normalizes frames to 480×768 resolution for both training and inference (a roughly 5:8 portrait ratio); the 3D VAE uses a temporal stride of 4 and a spatial stride of 8. That is, the data side is high-resolution and diverse, while the training side is uniformly downsampled.
【Bucketing strategy】The paper does not describe an aspect-ratio-based multi-bucket training (aspect-ratio bucketing) strategy. [Uncertain]

### [Step-Video-T2V](../models/Step-Video-T2V.md)

【Resolution】Serves as the curriculum's main axis: image stage at 256px → video stage at 192×320 (192P) → 544×992 (540P). The final released model outputs 540P and was not trained up to 720P/1080P — a notable tradeoff relative to contemporaneous HunyuanVideo/Wan, as the team invested compute instead into 204-frame long sequences and a deeply compressed VAE.
【Aspect ratio】Uses aspect-ratio bucketization, divided into landscape, portrait, and square groups, combined with frame-length buckets (1/68/136/204) to form a two-dimensional bucket system supporting joint multi-resolution, multi-aspect-ratio training.
【Black-bar handling】The pipeline uses FFmpeg to detect black-border size and crop accordingly, ensuring frames entering a bucket are free of padding borders.
The specific share of each aspect-ratio bucket is not published.

### [UniTalking](../models/UniTalking.md) ⚠️

The paper discloses no resolution or aspect-ratio information at all: no resolution admission threshold, no resolution distribution statistics, no aspect-ratio bucketing or padding strategy, and no frame-rate statement. Only in the video filtering section is there a general mention of removing videos with "low overall visual quality," without stating whether this judgment includes a resolution dimension, what model is used, or what the threshold is.
Indirectly inferable clues: the video branch inherits Wan2.2-5B (TI2V-5B), whose native support is 720P (704×1280 / 1280×704) @24fps, with a 3D causal VAE using a 16×16×4 spatio-temporal compression ratio (16× spatial, 4× temporal, higher compression than Wan2.1's 8×8×4), so training and inference resolution is very likely in the 720p class, covering both portrait and landscape ratios. But all of this is inferred purely from the base model's specs; the paper itself discloses nothing.
Reference comparison: UniVerse-1 explicitly gives a dual hard gate of "discard below 1080p + discard if bitrate-to-resolution ratio below 600"; UniTalking's disclosure granularity on this dimension is notably lower.

### [UniVerse-1](../models/UniVerse-1.md) ⚠️

【Resolution admission threshold】Raw video with resolution below 1080p is directly removed — a fairly aggressive high bar, excluding a large amount of 720p material, reflecting a "rather too little than too messy" preference. A bitrate-to-resolution ratio constraint below 600 is also applied, used to exclude high-resolution but low-bitrate "falsely high-definition" over-compressed material — a metric relatively rare among comparable works.
【Exception】VGGSound and AudioSet follow a simplified processing path, not subject to the 1080p or bitrate-ratio constraints (these two datasets are inherently low visual quality), instead isolating the effect of their low-quality visual signal via an LQLS loss strategy on the training side.
【Frame rate】Uniformly 25 fps.
【Aspect ratio】The paper describes no aspect-ratio standardization, bucketing, or padding strategy, and gives no aspect-ratio distribution.
【Training/inference resolution】The paper does not disclose the actual video resolution and frame-count configuration used at training and inference time; neither the model card nor the code repository give a resolution spec. Constrained by the capability of the underlying Wan2.1-1.3B backbone, it is presumed to be in the 480p class, but there is no official data to support this. [Uncertain]

### [Unison](../models/Unison.md) ⚠️

The paper does not discuss resolution and aspect ratio at all: no resolution admission threshold is given, no aspect-ratio distribution is given, no bucketing strategy is described, no padding or cropping processing is mentioned, and no training resolution is disclosed.
【The only certain piece of information】The output video frame rate is 25 FPS (inference configuration).
【Why this may genuinely be unnecessary】This gap has a structural reason: during Stage 2 joint training, the video backbone Wan2.2-5B is entirely frozen, and only the audio branch and fusion module (bidirectional cross-attention and layer normalization) are trained. This means Unison does not train visual-generation capability at all, so resolution-related data curation naturally becomes unnecessary — resolution processing and bucketing on the visual side are effectively inherited from Wan2.2's pre-training and fall outside the scope of this paper. This is a fundamental difference from works like MOVA (a three-stage 360p→720p resolution curriculum) and UniVerse-1 (a hard ≥1080p gate), which genuinely need to train a visual branch.
【Resolution status indirectly inferable from upstream datasets】HDTF is 720p–1080p high-definition talking-head video, VFHQ explicitly uses "high-fidelity" as its selection criterion, and OpenHumanVid is likewise named for "high quality" — all three have relatively high visual quality; whereas VGGSound is sourced from ordinary YouTube video and has notably lower visual quality (UniVerse-1 specifically designed an LQLS loss to isolate it). Unison provides no special handling statement for VGGSound's low visual quality — but since the video backbone is frozen, low-quality visual data cannot contaminate visual-generation capability, so the risk is naturally averted by the architectural choice. This is an inference made in this entry. [Uncertain]

### [Veo 3 / Veo 3.1](../models/Veo_3_Veo_31.md) ⚠️

[Uncertain] The training data's resolution/aspect-ratio distribution and bucketing strategy are undisclosed. Product-side output specs: Veo 3 supports 720p / 1080p, aspect ratios 16:9 and 9:16; Veo 3.1 further supports up to 4K, likewise covering 16:9 landscape and 9:16 portrait, at 24fps. Supporting both landscape and portrait ratios implies the training data includes multi-aspect-ratio samples and very likely uses bucketing or mixed multi-resolution training, but the specific proportions and implementation have no official basis.

### [Vidu S1](../models/Vidu_S1.md) ⚠️

[Uncertain]. On the training side, only "frame rate, resolution" are used as technical thresholds at the pre-filtering stage to remove low-frame-rate/low-resolution video; no specific thresholds, resolution distribution, or bucketing strategy are published. The inference/output side is 540p (960×540), 25 FPS standard, up to 42 FPS (measured on an RTX 5090); multi-aspect-ratio support is not mentioned.

### [Wan 2.5 / 2.6 / 2.7](../models/Wan.md) ⚠️

No distribution statistics, but a clear staged-threshold strategy.
- In Wan 2.1's basic-dimension filtering, "resolution thresholds are applied at different training stages to filter out low-quality data" — resolution thresholds are not a single cutoff but dynamic thresholds that increase with training stage, the core of its bucketing-like thinking.
- Resolution curriculum: 256px text-to-image pre-training → joint training stage one (image 256px + video 192px) → stage two (image and video both raised to 480px) → stage three (720px) → post-training jointly trains at 480px and 720px.
- Black-bar handling: heuristic black-bar detection with automatic cropping, to focus on content-rich regions — equivalent to an aspect-ratio normalization action.
- Generation-side aspect-ratio system (from the official wan2.7-t2v table): 720P tier — 16:9=1280×720, 9:16=720×1280, 1:1=960×960, 4:3=1104×832, 3:4=832×1104; 1080P tier — 16:9=1920×1080, 9:16=1080×1920, 1:1=1440×1440, 4:3=1648×1248, 3:4=1248×1648. For I2V, "the video's aspect ratio tries to match the input image as closely as possible," implying training coverage spans a continuous aspect-ratio distribution rather than a few fixed tiers.
The training-sample proportions for each resolution/aspect ratio are not disclosed. [Uncertain]

### [Audio-Video Generation Evaluation Benchmark Collection](../models/av_benchmarks.md) ⚠️

【VABench】Uniformly requires 720P (or the closest available aspect-ratio tier), with audio uniformly extracted at 48kHz stereo; frame rate follows each model's default of 24–30 FPS with no forced normalization.
【AVBench】Evaluation prompts explicitly require ≥720p HD (HD prompts) — a prerequisite gate consistent with its "real human scenes" positioning.
【AV-SyncBench】Uniform normalization before evaluation: video decoded at 25 FPS, audio resampled to 16 kHz; the original resolution and aspect-ratio distribution are not disclosed [uncertain].
【PhyAVBench】Recording resolution and device model distributions are not given in the paper [uncertain]; it only states that multiple recording devices were used to increase diversity.
【Omni-Judge】Follows the default output specs of Sora 2 / Veo 3 [uncertain].
Bucketing insights worth borrowing in the other direction: AV evaluation broadly converges on two spec sets — 720P + 5–10s + 48kHz (generation side) / 16kHz + 25FPS (discrimination side) — training-data bucketing could align to these two anchor points.

### [Video Captioning Model Ecosystem](../models/caption_models.md) ⚠️

[Uncertain] This dimension is generally under-disclosed in the captioner ecosystem, with only scattered evidence:
· ShareCaptioner-Video's official model card explicitly claims "support for video of various durations, aspect ratios, and resolutions"; its backbone InternLM-XComposer2-4KHD has 4K high-resolution understanding capability — a differentiating capability relative to contemporaneous captioners.
· Koala-36M's clips are uniformly 720p.
· AuroraCap uses token merging to compress visual tokens to manage long-sequence overhead, indirectly indicating that high resolution/long video input is a major cost driver.
· Panda-70M's UMT retrieval-selection model is fixed at 12 frames of 224×224.
· The mainstream practice is for captioners to run on low-resolution extracted frames (annotation only needs to be semantically correct, not pixel-faithful), so resolution has far less impact on annotation quality than on generation-training quality — which explains why this dimension is generally not discussed.
· No captioner work has been observed reporting a "resolution/aspect-ratio bucketing strategy" — this kind of bucketing belongs to the generative-model training side, not the annotation side.

### [Geometric/Structured Annotation Dataset Collection](../models/geometric_datasets.md)

SceneScribe-1M: spec filtering requires resolution above 1080p and frame rate ≥10fps, to preserve fine-grained geometric detail; SpatialVID: all clips are uniformly standardized to 1280×720, H.265-encoded MP4; WildWorld: 2K full-screen recording, 720p sub-window, model training resolution 544×960 (a variant of roughly a 9:16 portrait crop ratio); Action100M: retains HowTo100M's original resolution with no uniform bucketing strategy, with visual encoding using V-JEPA 2's fixed sampling window. None of the four adopts the multi-resolution/aspect-ratio bucketed training strategy common to video-generation models — because their positioning is as annotation datasets rather than training recipes.

### [Video-Generation Post-Training Data Strategy](../models/post_training_data.md) ⚠️

【Anchor paper】Not disclosed [uncertain].
【Cross-model comparison】Post-training stages almost uniformly adopt "the final tier of the resolution ladder": CogVideoX SFT is 768×1360; Allegro SFT requires ≥1280×720; LongCat-Video's SFT and GRPO both mix 480p+720p; Motif-Video 2B has two SFT passes at 480p and 720p, with the 720p pre-training starting from the 480p SFT checkpoint (rather than from the pre-training checkpoint) — an unconventional approach of inserting SFT into the middle of the resolution ladder; SkyReels-V2 first does 540p concept-balancing SFT then 720p high-quality SFT; HunyuanVideo 1.5's CT stage is 480p/720p at 24fps, with a separate super-resolution module trained on 1 million 1K–4K clips; ALIVE trains a separate 1080p Refiner on 0.7M high-clarity samples.
【Pattern】SFT is the closing stage of the resolution curriculum, while RLHF/GRPO, due to rollout cost, is typically run at a resolution lower than or equal to SFT (LongCat keeps 480p+720p, Cosmos uses a minimalist configuration of 8 rollouts × 20 diffusion steps). Aspect-ratio bucketing is generally no longer separately discussed at the post-training stage. [Uncertain]

### [Survey of Mainstream Video Pre-training Datasets: Panda-70M, InternVid, Koala…](../models/pretraining_datasets.md) ⚠️

**On the resolution dimension, the seven datasets converge heavily on 720p, with only UltraVideo and OpenVidHD as exceptions**:
- **720p tier (five datasets)**: Panda-70M (inherits full 720p from HD-VILA-100M; but note the **default download configuration, download_size:360, actually only yields 360p** — you must change the config to get 720p, a common pitfall when reproducing); Koala-36M (720p); MiraData (720p, measured samples at 1280×720@30fps); LVD-2M (**the paper never states resolution as a dataset attribute**, since it does not host the video itself; the download script defaults to --resolution=720p, WebVid shard filenames are labeled 336, and the only resolution figure that appears is a processing parameter — RAFT optical flow computed at 2fps temporally and 520×960 spatially); InternVid (**85% is 720P**, the remaining 15% is 360P–512P, with crawling restricted to 360P–720P).
- **1080p**: OpenVid-1M sets a **minimum-resolution floor of 512×512**; the OpenVidHD-0.4M subset is 1080p (433K clips). Neither aspect-ratio nor FPS distributions are disclosed in the paper. [Uncertain]
- **4K/8K (UltraVideo, the only one)**: short set — 4K **32,727** clips + 8K **9,457** clips (8K accounts for 22.4%); long set — 4K 12,277 + 8K 4,320. Frame rate is only split into two buckets, "≤30 FPS" and "≥50 FPS" (short: 31,027 / 11,157; long: 8,146 / 8,451), with **the 31–49 range empty**. HF dataset metadata shows frame sizes of 3,840–7,680 × 1,600–4,320, 12–60 FPS, 36–600 frames per clip. **Native resolution and frame rate are preserved with no transcoding**, which the authors cite as enabling frame-interpolation/codec research. Bitrate and encoding format are not mentioned anywhere in the paper.
**Bucketing strategy**: none of the seven has **a resolution/aspect-ratio bucketing design at training time** — they are datasets, not training frameworks; bucketing is left to downstream users (e.g., Open-Sora) to implement themselves. The UltraWan training accompanying UltraVideo uses fixed sizes (1088×1920×81 frames / 2160×3840×29 frames).
**A gap worth noting**: none of the seven **gives aspect-ratio distribution statistics**, and none performs targeted supplementation of portrait content (contrast with Open-Sora Plan's use of VIDAL to supplement portrait data) — all implicitly assume 16:9 landscape as the dominant format.
## Category/Domain Distribution and Mixture Strategy (Proportion Control and Concept Balancing for People, Actions, Scenes, Styles, etc.)

`domain_distribution` · Detail level: detailed

### [Allegro](../models/Allegro.md)

The paper's Appendix A gives category distribution statistics based on coarse-grained Tag2Text labels (Fig. 11): people, objects, and landscapes/natural scenery make up the vast majority of the tags, which the paper attributes to "natural dataset composition" — i.e., the distribution is determined by the source corpus rather than by a deliberately designed mixture.
It's worth noting that Allegro's "stratification" is stratification by quality threshold (the same corpus is cut into 4 progressively stricter subsets by aesthetic / CLIP / sharpness thresholds, used across 4 training stages), not a mixture balance across content domains. The paper does not describe concept balancing, long-tail category upsampling, proportion control by people/actions/scenes/styles, or any category resampling weight table. This is a clear gap in its pipeline relative to works like Movie Gen and Seedance.
There is one implicit constraint on content style: only single-shot footage is retained, so multi-shot narrative-type domains are not included.

### [Apollo](../models/Apollo.md) ⚠️

The paper gives no description whatsoever of the distribution or mixture strategy for visual domains (people, actions, scenes, styles, etc.), nor does it mention any concept balancing mechanism. The only "distribution" concept in the paper is the four scene categories split by audio type (single-speaker speech / multi-speaker speech / singing / natural sound) — that is, Apollo's data organization axis is **audio type rather than visual domain**. This is the most salient orientation of this work's data design: it treats "what is heard" rather than "what is seen" as the first-order dimension for data partitioning (see audio_category_distribution).
The only indirectly confirmable mixture intent comes from the training strategy rather than from data statistics: Stage II "specialized post-training" will "adaptively rebalance data distributions across scenarios and tasks to strengthen underperforming capabilities while preserving overall competency," indicating there is a metric-driven dynamic mixture mechanism, but no numbers are given for the specific scenario list, the initial mixture, or the post-adjustment mixture. [Uncertain]

### [CineDance / CineDance-1M](../models/CineDance.md) ⚠️

An eight-dimensional taxonomy is used to ensure diversity, but the quantitative share of each category is not published:
[Eight dimensions] Genre, Format (form/medium), Region, Modality (modality/form of expression), Story Logic (narrative logic), Era, Tone (emotional register), Audience (target audience).
[Design intent] This system is used to actively spread coverage during the collection and filtering stages, avoiding concentration of the corpus in a single genre or era, in service of the core scenario of "cinematic-grade narrative."
[Relationship to evaluation] CineBench's 1,000 test prompts are also stratified-sampled along three axes — Theme/Style × Duration/Shot Count × Difficulty — where the theme/style axis echoes the training-side taxonomy.
[Mixture control strategy] The paper does not describe explicit mixture control, concept balancing, or resampling mechanisms based on this taxonomy; the eight-dimensional system reads more like an after-the-fact characterization of diversity than an a priori sampling constraint.
[Category lists and percentages under each dimension] Not published. There is likewise no data on fine-grained proportions for people/actions/scenes/styles. [Uncertain]

### [CogVideoX](../models/CogVideoX.md) ⚠️

[Uncertain] The paper discloses nothing about the training data's category/domain distribution, concept clustering, or mixture-balancing strategy — this is the most conspicuous missing piece in CogVideoX's data engineering relative to works like Movie Gen and Seedance: no concept clustering, no long-tail resampling, no target proportion for people, no verb/action taxonomy.
The domain leanings that can be indirectly inferred come from the reverse constraints imposed by "negative labels":
· "Lecture Type" content (lecture/live talking-head footage where the frame is essentially static and only the person is speaking) is removed as an entire category → the "static person talking" subject matter is systematically suppressed in the training set, which also explains the model's weakness in lip-motion/talking-head scenarios.
· "Text Dominated" content (large amounts of visible text or content dominated by text) is removed as an entire category → image-text content, subtitle-heavy content, and presentation-slide content is excluded.
· "Noisy Screenshots" content (direct screen recordings of phones/computers) is removed as an entire category → screen recordings, gameplay footage, and UI demo content is excluded.
· "Editing" content (containing obvious manual editing and effects) is removed → heavily edited music videos and trailer-style content are excluded.
As a result, the dataset in practice skews toward "unedited, single-shot live-action footage with genuine continuous motion."
· On the caption side, the GPT-4 summarization prompt explicitly requires coverage of five elements — "objects, scenery, animals, characters, and camera movements" — which can be seen as the content dimensions the team cared about, but no mixture control was done based on this.
· On the evaluation side, VBench emphasizes dimensions such as Human Action, Scene, Dynamic Degree, Multiple Objects, and Appearance Style, which is a matter of evaluation choice rather than data mixture.

### [Cosmos-Predict2.5](../models/Cosmos-Predict25.md) ⚠️

The domain system is one of the core designs of this pipeline, split into two layers: a "general 26-category taxonomy" and "five specialized Physical AI domains":
[Layer 1: general content-type classification] At the end of the filtering stage, an internally trained content type classifier assigns each clip one semantic label, drawn from a custom-built taxonomy of 26 video types. This label is the primary axis for sharding, used for "fine-grained domain balancing" and curriculum-based training. Neither the specific category names of the 26 classes nor their respective proportions are published.
[Distribution-alignment constraint] At this stage, "physically unrealistic phenomena" are explicitly removed — video games, synthetic visual patterns, animations, cartoons — on the grounds of "maintain alignment with the physical world distributions." This is a distribution-trimming principle governed by physical realism, distinct from the general video-generation-model approach of pursuing stylistic diversity.
[Layer 2: five specialized Physical AI domains] To strengthen Physical AI capability, five additional domain-specific pipelines are set up, whose output is merged into the general pretraining data: Robotics, Autonomous Driving, Smart Spaces (warehouses/factories/construction sites), Human Dynamics, and Physics. These pipelines share the same structure as the general pipeline, but with two key differences: for filtering, the expensive VLM filter is omitted in favor of "a domain-specific subset of filters plus adjusted hyperparameters"; for tagging, a larger-parameter VLM with domain-customized prompts is used instead.
[Target-distribution sampling for autonomous driving] Driving data is not randomly sampled but rather "sampled from a large-scale corpus to align with a target distribution of diverse driving attributes" — balanced sampling along nine attribute axes according to a preset target distribution: geographic region (US, Europe), traffic density (sparse/congested), ego-vehicle speed (urban roads/highway), ego-vehicle acceleration (constant speed/hard acceleration), ego-vehicle maneuvers (gentle curves/sharp turns), road type (urban/rural), rare road structures (tunnels, toll booths), visibility (clear/foggy), weather (dry/snowy), and lighting (day/night). This is the most concrete mixture control strategy in the entire document.
[Taxonomy for the physics domain] A classification system of "visually observable physical phenomena" is first defined, covering core areas such as classical mechanics and fluid dynamics (e.g., shattering glass, colliding rolling balls, flowing water), and videos that highlight these dynamic properties are then collected in a targeted manner based on it.
[Quantitative admission criteria for Human Dynamics] The human domain controls frame composition with hard numerical rules: people must appear in more than 40% of frames; the number of visible people in any given frame must not exceed 8; and at least one person must occupy more than 3% of the frame area.
[Five post-training domains] The SFT side has a separate, independent partition (object permanence / high motion / complex scenes / driving / robotic manipulation), produced by a multi-head classifier on InternVideo2 embeddings; see post_training_data.
[Uncertain: the specific categories of the 26-category taxonomy and their respective proportion figures]

### [Data-Juicer 2.0](../models/Data-Juicer.md) ⚠️

Data-Juicer's approach to category/domain mixture is fundamentally different from that of generative-model teams: rather than presetting a domain taxonomy, it provides mechanisms that make domain distribution measurable, splittable, and searchable, leaving the mixture decision to be determined empirically by the Sandbox's feedback loop.
[Domain annotation capabilities]
  · video_tagging_from_frames_mapper — generates video content tags from extracted frames (based on image-tagging models such as RAM), producing open-vocabulary semantic labels; the primary means of building a domain-distribution profile.
  · video_tagging_from_audio_mapper — generates tags from the audio track based on the Audio Spectrogram Transformer, providing domain information on the auditory side (complementary to the visual tags).
  · video_tagging_from_frames_filter — retains/removes samples by tag, i.e., targeted domain-based filtering.
  · video_object_segmenting_mapper — text-guided semantic segmentation based on YOLOE + SAM2, which can provide object-level domain evidence.
[Specialized support for the person-centric domain] v1.5.4 added nine person-centric video understanding operators in one release, forming a fairly complete processing chain for the person domain: video_human_tracks_extraction_mapper (extracts face and body-box tracks), video_face_ratio_filter (filters by the fraction of the frame occupied by faces, usable for selecting close-up/medium/long shots), video_active_speaker_detect_mapper (active speaker detection), video_captioning_from_human_tracks_mapper (generates descriptions based on person tracks), video_captioning_face_attribute_emotion_mapper (face attribute and emotion description), video_human_tracks_face_demographic_mapper (demographic attributes), and video_whole_body_pose_estimation_mapper (2D whole-body keypoints for body/hands/feet/face). This chain directly corresponds to the most important domain in video generation — "people, actions."
[Mixture strategy — empirical search rather than a priori design] The Sandbox's Probe-Analyze-Refine workflow is, in essence, an automatic search for data mixture: for each candidate operator i, the data pool is split by its statistic into three equal-sized sub-pools — P_i,low / P_i,middle / P_i,high — plus a randomly sampled control pool; reference models are trained on each and evaluated with VBench, and the operators and their value ranges are ranked accordingly. The top-ranked operators are then combined into 2^n−1 cross-product data pools (a pyramid structure, where higher tiers have higher quality but fewer samples), and the search for the optimal combination continues. This process turns "which segment of the distribution should be kept along which dimension" into an experimental question that can be answered directly by model feedback, rather than by manually setting proportions.
[Mixture-related recipes in DJ-Cookbook] Maintained recipes include "curriculum learning based on data difficulty" and contrastive-learning recipes, which serve as ready-made templates for domain/difficulty mixture scheduling.
[Uncertain] The official DJ team has not published any recommended domain taxonomy for video generation (e.g., target proportions for people/actions/scenes/styles), nor has it reported category-distribution statistics for the final data pool in its T2V case study; strategies such as concept balancing and long-tail category resampling are not seen supported by any dedicated operators in the documentation.

### [Foley-Omni](../models/Foley-Omni.md) ⚠️

The paper does not control mixture along the "visual domain/category" dimension; instead, the "task group" is the primary mixture unit — this is the fundamental difference between Foley-Omni's data organization and that of text-to-video models.
The hourly mixture across the six task groups (4.9k hours total): VisualTTS 1,980h (40.4%) > TTS 1,253h (25.6%) > TTA 912h (18.6%) > V2A 403h (8.2%) > V2ST 216h (4.4%) > TTM 139h (2.8%). Speech-related groups (VisualTTS + TTS) together account for 66%, sound-effect-related groups (TTA + V2A) account for 26.8%, and music-related content (TTM) is only 2.8% — a severe skew toward speech, which explains the model's standout advantage on the WER metric (7.59, lower than the two cascaded baselines' 10.57 and 37.84), while also potentially being a latent concern for relatively weak music-generation capability.
The visual content domain can be inferred indirectly from the dataset sources: GRID, LRS2, and Chem are controlled/semi-controlled frontal speaker videos (lab readings, BBC news, chemistry lectures); SpeakerVid and TalkVid are large-scale in-the-wild speaker videos; VGGSound consists of 10-second in-the-wild event videos (310 sound-source classes); and Kling-Foley is dedicated Foley footage. Overall, the domain is heavily concentrated in two categories — "scenes with people speaking" and "scenes with clearly identifiable sound-source events" — lacking visual domains such as pure scenery, abstract art, or animation.
[Motion intensity as an implicit domain control] The motion score is bounded to the [0.1, 3.2] range — the lower bound removes near-static frames (no visual event to align with, leaving the V2A supervision signal empty), while the upper bound removes intense motion/rapid cuts (unreliable optical-flow estimation, confused audio-visual correspondence). In effect, motion intensity is used to narrow the data domain down to "moderate dynamics, single continuous scene."
[Uncertain] No explicit visual category-label distribution statistics are given; no mention of concept balancing or long-tail category resampling strategy; no proportion control given for people/actions/scenes/styles.

### [Goku](../models/Goku.md)

★This is Goku's most distinctive section, and also the highest-value part of this survey: Data Distribution Balancing is explicitly listed as the fifth independent stage of the five-stage pipeline, rather than being subordinate to the filtering step. Most contemporaneous works (HunyuanVideo, Open-Sora, CogVideoX, etc.) treat category mixture as merely a byproduct of filtering, or don't mention it at all; Goku elevates it to a first-class stage alongside "collection/segmentation/filtering/tagging," making it one of the most explicit modelings of the "data distribution" dimension.
[Taxonomy] An internal, self-developed video classification model is used, uniformly sampling 4 keyframes per clip to assign labels, producing a two-level taxonomy: 9 top-level categories + 86 second-level subcategories.
  - Examples of top-level categories: human, scenery, animals, food, urban life, etc., of which human, scenery, food, urban life, and animals are the highest-share categories.
  - Examples of second-level subcategories: half-selfie, kid, dinner, wedding, etc. — fairly fine-grained.
[Mixture strategy] The paper explicitly states it "emphasizes human-related content while ensuring equitable representation across subcategories" — i.e., overall, human-related content is deliberately upweighted (since people are the highest-frequency, most closely watched need in video generation, and also the area most prone to visible flaws), while relative balance in representation is maintained across the 86 subcategories. There are two concrete means:
  (1) Down-sampling overrepresented categories;
  (2) Augmentation and oversampling for underrepresented categories.
[Visualization] Figure 3 of the paper shows the balanced semantic distribution with two distribution charts: (a) top-level categories and (b) second-level subcategories.
[Assessment] This stage embodies the "concept balancing" idea: filtering only guarantees the quality of individual samples, while distribution balancing guarantees the entire dataset's coverage and lack of skew in concept space, directly corresponding to the performance on multiple category metrics in VBench such as human action, object class, and scene (Goku scores 79.48 on human action and 85.72 on scene, both significantly higher than contemporaneous open-source models).
[Not disclosed] The specific percentage figures for each category (only charts, no tables), the specific down-sampling/oversampling ratios, the architecture and accuracy of the classification model, and whether a VLM was used instead of a dedicated classifier.

### [Hailuo / MiniMax Video](../models/Hailuo.md) ⚠️

[Uncertain] (no quantitative mixture disclosure whatsoever). The official team has never published proportions for categories such as people/actions/scenes/styles, nor mentioned concept balancing or long-tail category supplementation strategies. Qualitative domain coverage that can be inferred backward from product capabilities (not an official data disclosure, for reference only):
(1) Realistic live-action people: Hailuo 2.3 emphasizes "live-action facial performances" and the naturalness of micro-expressions; on Replicate, 2.3 is described as "optimized for realistic human motion, cinematic VFX, and expressive characters," indicating that live-action + cinematic footage is a core domain;
(2) Complex human motion and physics: Hailuo 02 emphasizes rendering physical laws (e.g., widely circulated demonstrations of high-difficulty moves like gymnastics and diving), and 2.3 emphasizes "more complex character body movements," indicating that highly dynamic human motion is a domain deliberately strengthened;
(3) Anime/2D style: as early as video-01-live, there was specialized training for Live2D and general animation; 2.3 explicitly expands into anime and illustration;
(4) Chinese art styles: 2.3 specifically names ink wash painting and game CG, a targeted domain reinforcement aimed at the Chinese market;
(5) Cinematography: video-01-director was trained separately for specific camera movements, indicating the existence of a subset annotated with camera-movement labels.
There is no disclosure whatsoever of the relative proportions of the above five categories, or of whether there is an explicit mixture-control mechanism.

### [HunyuanVideo-Foley](../models/HunyuanVideo-Foley.md) ⚠️

[Core mechanism: using classification tags for distribution management, without publishing distribution numbers] This is the most noteworthy — and also the vaguest — link on the data side of this work. The paper explicitly states: a "speech-music detection and audio classification model" is used to assign categorical tags to each retained clip, with the purpose of "enabling effective management of category distribution," which is said to thereby "ensure balanced representation in the training dataset."
[Key gap] The paper declares that distribution management was done, yet gives absolutely none of the following: what the taxonomy is (how many categories? category names? based on the AudioSet 527-class ontology, or a custom scheme?), the actual share of each category, what the target mixture is, or by what means balance is achieved (upsampling low-frequency categories? downsampling high-frequency categories? or simply truncating over-represented categories by threshold?). This is therefore a "mechanism exists, data doesn't" field — the reader can only confirm that the team was aware of the long-tail and category-imbalance problem and took some measure, but cannot know the specific form or effect of that measure. [Uncertain]
[Why this link matters] The natural distribution of sound-effect data is extremely long-tailed: speech, background music, and traffic noise occupy the vast majority of the duration, while genuine Foley event sounds (footsteps, page-turning, pouring water, knife chopping) make up a very small share. Without active intervention, a model trained on 100,000 hours of data would strongly skew toward generating ambient background noise and music rather than precise event sounds. The paper lists "category distribution management" as the final link in the pipeline (after all quality filtering), indicating it is the final gate that determines data composition — its importance is disproportionate to the scarcity of information disclosed about it.
[Indirect evidence: generalization coverage] Both official and third-party reports emphasize the model's generalization capability across various video types such as "people, animals, natural scenery, and cartoons/animation" — this can be seen as circumstantial confirmation of the breadth of category coverage, but it remains a qualitative description.
[Relationship to evaluation categories] The Kling-Audio-Eval benchmark used for evaluation itself has a built-in taxonomy (built by the Kuaishou Kling team), but the paper does not state whether the training data's category management was designed with reference to that scheme.

### [HunyuanVideo](../models/HunyuanVideo.md) ⚠️

This is the most methodologically valuable point of the original HunyuanVideo, and one of the few contemporaneous open-source works to explicitly perform "concept balancing":
[Original version] (1) The original data pool is divided by domain, explicitly enumerating categories such as people, animals, plants, landscapes, vehicles, objects, buildings, and animation, with coverage breadth as a data-construction goal; (2) an in-house VideoCLIP model is used to extract embeddings for clips, first performing semantic deduplication by cosine distance, then running k-means on the embeddings to obtain roughly 10K concept centroids, based on which concept resampling and balancing is performed — i.e., cluster centers are used as concept proxies, downsampling over-dense concepts and upsampling sparse ones to suppress long-tail imbalance. This "VideoCLIP embedding + 10K cluster centers + resampling" combination is the signature design of the original data pipeline. (3) The final proportion figures for each domain are not published.
[1.5] The report does not repeat the description of the concept-balancing mechanism, only emphasizing that channel diversity covers "content, filming technique, camera movement, style, scene"; whether the 10K concept-center resampling is continued is not stated. 1.5 shows traces of categorization at the RLHF stage: the I2V RLHF prompt set covers "100+ categories," and T2V DPO uses a prompt set balanced across the three dimensions of motion/scene/subject. [Uncertain]

### [InstructAV2AV](../models/InstructAV2AV.md) ⚠️

This work's domain-organization dimension is not "visual content category" (people/actions/scenes/styles) but rather "type of editing operation" — this is the fundamental difference between editing-type datasets and generation-type datasets: the object of mixture control shifts from "what was filmed" to "what is being changed."
[The paper-level four-category task taxonomy]
  (a) Identity-preserving speech modification: changing the spoken lines while preserving the speaker's identity/timbre/appearance;
  (b) AV instance editing: changing the appearance of a person or object in the frame while also changing the sound it produces;
  (c) AV instance insertion: adding a new visual element along with its corresponding sound;
  (d) AV instance removal: removing a visual element along with its corresponding sound.
  The common structure across the four categories is that the "visual target + its bound sound" changes in sync, while non-target regions and ambient sound remain unchanged.
[The five-category subset naming actually released on HuggingFace, finer-grained than the paper] add_and_remove (addition and removal, corresponding to c+d merged), clone_id (identity cloning, changing only the person, not the voice), clone_voice (voice cloning, changing only the voice, not the person), clone_id_voice (simultaneous identity + voice cloning), general_editing (general editing, corresponding to b). This five-way split reveals an important design not elaborated in the paper: speech-type editing is further orthogonally decomposed into two independently controllable factors — "visual identity" and "auditory timbre" — and data is separately constructed for three combinations (changing only one / changing only the other / changing both). This orthogonal-decomposition style of data construction is a unique advantage of synthetic data over real-world collection — in reality it is virtually impossible to collect genuine footage of "the same video with the face swapped but the original voice kept"; this can only be manufactured through controlled synthesis, which also allows precise control over which factors change and which don't, thereby giving the model a clean, disentangled supervisory signal.
[Indirect inference of the visual-content domain] Inferred backward from the sources: MovieBench + Condensed Movies + Short-Films-20K together contribute narrative film/TV scenes (character dialogue, indoor/outdoor scenes, diverse lighting and cinematography); VGGSound contributes in-the-wild sound-source events (310 classes covering animals, instruments, machinery, natural phenomena, etc.); YouTube contributes open-domain footage. Overall it skews toward two categories — "film/TV scenes with people talking" and "event scenes with clearly sounding objects" — similar in domain concentration direction to Foley-Omni.
[Implicit domain-narrowing mechanism] CoTracker3 (Karaev et al.) performs grid-based point tracking and removes clips whose average motion magnitude falls below a threshold — this step excludes purely static, near-static scenes, narrowing the domain to clips with "visible motion." The reason is that editing tasks require targets that can be mask-tracked and that exhibit deformation/displacement; static footage both lacks editing value and offers no way to test temporal consistency.
[Uncertain] No quantitative distribution table is given for any category label: the number of samples in each of the five subsets, the mixture of visual content categories (people/animals/objects/scenes), the proportion of speech vs. non-speech samples, or any concept balancing or long-tail resampling strategy — none of this is disclosed in either the paper or the data card. The specific value of the CoTracker3 motion threshold and the LAION aesthetic threshold are likewise not given.

### [Other Joint Audio-Video Generation Works, 2026](../models/JAVG_2026_misc.md) ⚠️

On this dimension, ALIVE and NAVA each establish an explicit taxonomy, making them the most valuable of the seven items; the remaining five have essentially no mixture control.
[ALIVE — a three-level label system + active mixture adjustment (the most complete domain-control design among this batch of works)]
(1) Top-level split: a division into the two core scenarios "speaking" and "non-speaking" is made first — the original text states, "First, we make a top-level distinction between core scenarios: 'speaking' and 'non-speaking' scenario." This split directly determines the model's mixture across "dialogue-driven" and "sound-effect-driven" capabilities.
(2) A three-level hierarchical label system: on top of this, a three-level hierarchy is built, where Level 1 consists of nine major domains: Animals, Home Sounds, Entertainment, Environment, Food, Nature, Sound Effects, Vehicles, Sports. Note that these nine categories are "joint audio-visual categories" rather than purely visual categories — Home Sounds and Sound Effects are themselves acoustic categories, indicating that this system was designed from the outset to serve joint audio-video modeling.
(3) Audio-visual keyword binding: every Level 3 visual label is paired with a corresponding audio keyword via "joint audio-visual keyword retrieval," forming a paired index of visual concepts and sound concepts — this is the key mechanism for controlling domain distribution and audio distribution jointly rather than independently.
(4) Active mixture adjustment: the original text states, "Guided by prior knowledge, such as concept frequency and projected application scenarios, we then adjust the data proportions for each category." — the proportion of each category is actively adjusted based on two priors: concept frequency and anticipated application scenarios. The 4.3M "balanced samples" used at the continue-training stage are the product of this mixture adjustment. Specific percentages are not disclosed [Uncertain].
[NAVA — five audio-driven content categories] YAMNet audio classification plus an omni-modal tagger divides clips into five categories: single-speaker speech, multi-speaker speech, ambient sound, music, singing. Note that its classification axis is acoustic rather than visual, differing from ALIVE's dual audio-visual axes. The proportion of each category is not disclosed [Uncertain]. There is also VLM-based filtering and tagging to retain clips with clear visual quality, though the label scheme is not detailed.
[CCL — implicit domain profile] No explicit mixture is done, but its data sources — "interviews, short dramas, and films" — themselves constitute a narrow-but-deep domain: person-centric, dialogue-dominated narrative content. This explains how it achieves SOTA with only 4M of data — trading breadth for depth. The high share of "joint generation 0.6" in its multi-task training probability distribution (see stage_data_mixture) likewise indicates its target domain is highly focused.
[OmniCustom — single domain] Fully focused on "single-person video clips," with no demand for domain diversity — all 1 million clips belong to the same category. The evaluation set deliberately controls the gender ratio at "1:1" — this is the only explicit demographic mixture control among the seven items.
[StreamChar — single domain] Person-centric character speaking videos; all three upstream datasets are human-centric, with no cross-domain mixture.
[Baton] No domain mixture strategy is mentioned; the mixing ratio of its data sources (OpenHuman-Vid human data + AudioCaps/WavCaps general audio + internet video) is not disclosed [Uncertain].
[ITS-JAVG] No training data; but its evaluation benchmark, JavisBench-mini, itself has a multi-category structure (inherited from JavisBench's scene/event classification scheme), used to examine performance differences of various validators across categories [category details uncertain].

### [Collection of Joint Audio-Video Generation Baselines](../models/JavisDiT_baselines.md) ⚠️

Domain coverage evolving from "single-domain" to "dual-domain mixture" to "general-purpose" is one of the clearest dimensions of progression in this collection:
[MM-Diffusion — single domain with clearly defined categories] Landscape covers 9 categories of natural scenes, fully disclosed: explosion, fire cracking, raining, splashing water, squishing water, thunder, underwater burbling, waterfall burbling, wind noise — all natural phenomena with strong causal binding between visual event and sound, a domain deliberately chosen to validate audio-visual alignment. AIST++ is a single street-dance domain (body dance movement ↔ music beat). The two datasets respectively represent two types of synchronization — "ambient-sound alignment" and "rhythm/motion alignment" — and this "complementary dual-dataset" setup was fully carried forward as a standard evaluation configuration by later works such as AV-DiT.
[AV-DiT] Fully carries over the above two domains, with no extension.
[JavisDiT / JavisDiT++ — explicitly built a category taxonomy] One of its biggest contributions is making the domain-distribution problem explicit: JavisBench establishes a taxonomy with five evaluation dimensions and 19 scene categories in total; the five dimensions are: Event Scenario, Video Style, Sound Type, Spatial Composition, Temporal Composition. The authors emphasize that "over 50% of videos belong to highly complex and challenging scenarios" and "75% of samples contain multiple sounding events" — a direct critique of, and improvement over, early baselines' "single sound source, single event" setup. On the training side, domain breadth is obtained through TAVGBench's general YouTube distribution, but the category proportions of the training data itself are not disclosed [Uncertain].
[Harmony — explicit 1:1 dual-domain mixture] Data is explicitly split into two domains, "human speech" and "ambient sound," with a strict 1:1 mixing ratio used in both Stage 1 and Stage 3 training. Harmony-Bench is likewise split along the same lines into three tiers — ambient-sound video, speech video, and complex scenes (ambient sound + speech co-occurring) — forming a direct correspondence between "training mixture ↔ evaluation category." This is the only work in this collection to give explicit domain mixture figures.
[UniAVGen — single domain focused on real people] The training data is "real-person audio-video," with the domain heavily concentrated in scenes of people talking/performing, paired with a Face-Aware Modulation module — a deliberately narrow-domain design; the 100-clip evaluation test set is split "50% real images / 50% AIGC and anime-style images," indicating explicit mixture control on the image-style dimension (real vs. anime, half each), but no breakdown is done at the content-domain level [Uncertain].
[Common gap] Aside from Harmony's 1:1 and UniAVGen's 50/50, none disclose fine-grained proportion control or concept balancing strategies for people/actions/scenes/styles [Uncertain].

### [Kling 3.0 / Kling Video 3.0 Omni](../models/Kling_30_Omni.md) ⚠️

[Uncertain] Category/domain mixture figures are not published. Qualitatively, the Kling-Omni report states that data collection pursues "broad scenario coverage," spanning diverse subjects and style variants; and in the multi-modal alignment step, dedicated character-identity consistency checks are performed for "person-centric" tasks, indicating that person-category data is a key subdomain modeled and mixed separately. The emphasis of product capabilities (AI-director storyboarding, multi-character dialogue, cinematic-grade texture, physical laws and inertia/collision) reflects a training mixture clearly skewed toward: character performance and dialogue, cinematic camera work, and physical-interaction scenes. KlingAvatar 2.0 explicitly constructs its evaluation subset into three categories — Chinese speech / English speech / singing — suggesting speech-category data is broken down and mixed by use case. [Uncertain: the specific proportion-control and concept-balancing methods]

### [LTX-2](../models/LTX-2.md) ⚠️

No category/domain mixture figures are disclosed at all, nor is any concept-balancing strategy described. There is only one qualitative statement: the selected subset "provides a balanced distribution of visual and auditory content," allowing captions to adequately cover both image-domain and auditory-domain information at once. Indirect clues: (1) Fig. 13 of the LTX-Video paper gives a caption word cloud, which roughly reflects concept distribution but without numbers; (2) image datasets are mixed into training specifically to supplement "concepts uncommon in video data," indicating the team pays attention to concept coverage; (3) the licensed sources are the Shutterstock/Getty commercial stock libraries, whose distribution naturally skews toward professionally shot general-purpose footage rather than UGC. The limitations section acknowledges that "languages and dialects underrepresented in the training data" lead to degraded performance, indirectly indicating a long-tail distribution problem that has not been specifically balanced. [Uncertain]

### [LongCat-Video](../models/LongCat-Video.md) ⚠️

This is the relatively most distinctive part of LongCat-Video's data work, employing an unsupervised taxonomy of "caption text-embedding clustering + LLM semantic naming" rather than a manually predefined label scheme. The specific approach: text embeddings are computed for all video captions, clustering is performed in embedding space, and an LLM then infers and names a category for each cluster, yielding content-type labels (the report gives examples such as personal interactions, artistic performances, natural landscapes, etc.). This taxonomy serves three purposes: (1) observing the data distribution to identify over-dense/over-sparse categories; (2) performing "targeted data supplementation or rebalancing"; (3) supporting "dynamic and precise allocation of data subsets tailored to specific requirements and objectives of different training phases," i.e., dynamically and precisely allocating data subsets for different training stages.
Going further, the SFT stage uses this embedding space directly as sampling weights: samples are "selected inversely proportional to their density in the caption embedding space" — this is an explicit concept-balancing mechanism, where high-density common concepts are downsampled and long-tail rare concepts are relatively boosted, making the semantic coverage of the curated set more uniform.
In addition, two domains are separately reinforced: camera motion and visual style, with dedicated specialized datasets introduced at the SFT stage to strengthen instruction-following on these two dimensions.
What is not disclosed: the number of clusters, the specific proportion figures for each category, and the target mixture after rebalancing. [Uncertain: the specific proportion figures for each category]

### [MOVA](../models/MOVA.md) ⚠️

The paper gives only a qualitative enumeration of the training data's domain coverage, with no proportion figures and no description of a concept-balancing mechanism:
- Video forms: movies, vlogs, animations, Chinese drama, cartoons, YouTube content.
- Subject-matter domains: education, sports, beauty, news, interviews, etc.; the paper states these provide "the distributional diversity needed to generalize to complex real-world scenarios."
- The only strong mixture skew that can be confirmed is being "person-speaking-centric": among the Phase 1 data sources, both SpeakerVid-5M and OpenHumanVid are person/speaker-centric datasets, and only speech clips are retained for final training (see audio_category_distribution), indicating that person-dialogue-category data absolutely dominates the mixture — consistent with the model's positioning as focused on multilingual lip-sync.
- The only category pie chart in the paper with percentages (Figure 6a, including "others 2.3%") describes the sample category distribution of the self-built evaluation benchmark, not the training-data distribution, and should not be conflated with it.
The domain mixture figures for the training data are an information gap. [Uncertain]

### [Merged entry: ① Mochi 1](../models/Mochi_MAGI_Motif.md) ⚠️

This is the dimension where the three differ the most, forming a progression line of "none → dynamic adaptive → explicit category balancing":
① Mochi 1: No disclosure whatsoever. Neither category statistics nor any mention of concept balancing or mixture control. [Uncertain]
② MAGI-1: Rather than producing a static mixture table, it proposes "Dynamic Distribution Adjustment" — the paper explicitly states that "an appropriate data distribution is crucial for training a high-performance model, but determining the optimal distribution in advance is extremely difficult," and gives an actual observation: "during training we found that landscape-type scenes are relatively easy for the model to learn, while human facial expressions are significantly harder" — insights of this kind cannot be anticipated beforehand. Its approach is therefore to continuously monitor the model's performance on various semantic concepts throughout training, and adaptively raise the sampling ratio of underfit subsets based on evaluation results, to specifically reinforce the model's weak points. This is a form of "online data mixture driven by an evaluation-feedback loop," closer to the original spirit of curriculum learning than a static mixture table, but the paper does not disclose any category list, initial mixture, or adjustment magnitude. MAGI-1 also mentions one task-level mixture: since the product scenario is primarily image-to-video, "a larger proportion of I2V tasks was allocated during training." [Partially uncertain: category list and mixture figures]
③ Motif-Video 2B: Performs explicit category balancing, and closes the loop by attaching it to caption metadata. The VLM produces structured subject and style labels in the same forward pass, and these two labels "drive domain balancing for the 720p stage and SFT." More importantly, the SFT corpus is assembled through an iterative weak-point-filling approach: "we run intermediate evaluations on the latest checkpoint, identify the subject categories with the weakest generation quality, and then specifically supplement clips for those categories." Fig. 8 gives the final result — on the image side, People dominates, reflecting a character-centric use case; on the video side, it skews toward Transportation, Sports, and Animals, because these three categories involve intense dynamics and were identified as weak points in intermediate evaluations. This is structurally isomorphic to MAGI-1's dynamic-adjustment idea, but Motif renders it into a readable category-distribution chart, and additionally uses the action=Dynamic label as the admission criterion for dynamic motion in 720p SFT.

### [Movie Gen](../models/Movie_Gen.md)

The raw pretraining pool covers domains such as humans, nature, animals, and objects. Concept balancing takes two steps: ① a joint video-text embedding model extracts semantic embeddings, which are clustered into fine-grained concept clusters; ② after merging duplicate clusters, sampling is done from each cluster in inverse proportion to the square root of cluster size (1/sqrt(cluster size), following the approach of Mahajan et al. 2018), thereby suppressing dominant head concepts and lifting the long tail — this step reduces the data from 1.15% to 0.94% in the funnel.
People are a domain that is deliberately weighted: at least 60% of videos in the final high-resolution training set contain a person. To this end, a dedicated taxonomy of 600 human verbs and expressions was built, and this taxonomy is used to do zero-shot text-to-video retrieval to specifically select videos containing people; during the concept-resampling stage, the frequency of these person videos is deliberately preserved to prevent it from being diluted by the balancing strategy.
Concept balancing at the SFT stage is more fine-grained: the same 600-verb taxonomy is first used to perform text k-NN, retrieving videos for each concept from the candidate pool; then, for each concept, a few visually striking seed videos are manually selected, and video k-NN is performed using these seeds, ultimately yielding a subset that is concept-balanced and small enough in scale to be fully manually reviewed; the k-NN uses video and text embeddings from the joint video-text embedding model.
The corresponding concept distribution on the evaluation side consists of five categories: human activity, animals, nature and scenery, physical phenomena, and unusual subjects and unusual actions.

### [NVIDIA NeMo Curator](../models/NeMo_Curator.md)

This is one of the most valuable reference fields in this entry — Cosmos WFM implements explicit, classifier-based domain mixture control via NeMo Curator/Cosmos-Curate, a rare industrial case in public materials where "data mixture is quantifiable."
[Implementation mechanism] A lightweight MLP classifier is trained on top of InternVideo2 video embeddings, assigning a category label to each clip according to a predefined video-type taxonomy, which is then used for filtering and resampling.
[Excluded categories] Abstract patterns, video game footage, animation — these three categories are judged unfavorable for learning real-world physical laws and are removed entirely.
[Resampling strategy] Upsample the human-motion and human-object-interaction categories; downsample the scenery category — because while scenery video has high image quality, it is sparse in motion information and physical-interaction information, and is naturally over-represented.
[Final nine-category mixture (published in the Cosmos WFM paper)] nature dynamics 20%, hand motion & object manipulation 16%, spatial awareness & navigation 16%, driving 11%, human motion & activity 10%, first-person POV 8%, dynamic camera 8%, other 7%, synthetically rendered 4%.
[Note] This taxonomy serves the goal of Physical AI / world models (learning physical laws), which is not the same mixture objective as general-purpose text-to-video models (which emphasize people, cinematic feel, and stylistic diversity) — the taxonomy needs to be redefined when transferring to other use cases. The open-source release of NeMo Curator does not bundle the weights of this taxonomy classifier; users must train it themselves.

### [OmniHuman Dataset + OHBench](../models/OmniHuman.md) ⚠️

Domain diversity is the primary intent of this dataset — the paper lists "limited global scene and camera diversity" as the first structural flaw of existing datasets, and the entire dataset is designed to address it:
[Scene categories (8)] Eight scene types: home spaces, offices, natural environments, urban/rural landscapes, performance venues, retail, industrial areas, etc.
[Content categories (8)] Eight content types: talk shows, educational, cooking/food, sports, music, gaming, reviews, film/TV.
[Person-configuration dimension (3 categories)] single-person, two-person, person-object interaction. This is the core dimension distinguishing this dataset from all prior work — the paper lists "sparse interaction modeling (both person-person and person-object)" as the second structural flaw, and specifically annotates interaction labels, speaker IDs, and emotion for two-person interactions, and object information for person-object interactions. Table 1 shows that all prior comparison datasets (ActivityNet, TikTok-v4, OpenHumanVid, CelebV-HQ, VoxCeleb2, HDTF, SpeakerVid-5M) are entirely empty in the "person-object interaction" and "object annotation" columns; OmniHuman is the only one covering both.
[Camera and shot dimension] Annotation covers shot scale and camera motion; in Table 1, only OpenHumanVid and SpeakerVid-5M have both of these. Evaluation found that "model performance degrades on shot transitions from close-up to long shot," indicating that shot-scale annotation does support fine-grained diagnosis.
[Mixture strategy: not disclosed] Aside from the single-person/two-person ratio found in Figure 3(b), there are no numbers for the specific share of the above 8+8+3 categories within the 1 million clips. Nor is any concept balancing, resampling, long-tail suppression, or quota-control mechanism described — i.e., the paper demonstrates that "these categories are covered" but does not demonstrate that "the distribution across categories is balanced." Given that talk-show/review content is far more abundant than industrial-area scenes on YouTube, the actual distribution is very likely severely long-tailed. [Uncertain]
[Correspondence with the evaluation system] OHBench's 509 evaluation videos are composed in the ratio 331 single-person / 128 two-person / 50 person-object, corresponding one-to-one with the dataset's three person-configuration categories, forming an explicit alignment between the training distribution and the evaluation categories (see benchmark_taxonomy_alignment for details).

### [Open-Sora series](../models/Open-Sora.md) ⚠️

**Neither project does explicit category/domain mixture control or concept balancing** — this is the most obvious shortcoming of their data strategy relative to industry models (Seedance, Movie Gen, etc., which have fine-grained domain taxonomies and mixture tables).
Identifiable implicit domain structure:
- Open-Sora 1.2 achieves domain diversity indirectly by mixing data from different sources: Webvid/Panda-70M provides general web video (YouTube long tail), MiraData specifically supplements game footage and long urban-fly-through shots, Vript supplements densely annotated cinematic content, Inter4K supplements 4K high-definition footage, and LAION/Unsplash supplements static high-aesthetic images. This is an approach based on "division of labor by dataset function" rather than "mixture by semantic category."
- Open-Sora Plan specifically filters out 160,000 high-quality portrait images from LAION-5B to strengthen person-generation capability, which is the only explicit, targeted category supplementation; the VIDAL 3M vertical-screen Shorts implicitly contain a large amount of person talking-head/lifestyle content.
- Neither publishes proportion figures for people/actions/scenes/styles, does long-tail concept balancing, or does resampling after semantic clustering. The Open-Sora 2.0 report only gives a statistics chart (Figure 3) along four low-level dimensions — aesthetic score/duration/aspect ratio/caption length — with no statistics along a semantic-category dimension at all.
- The style dimension exists only as a descriptive field within captions (one of the six elements of Open-Sora 2.0, "video style"), but no training-set mixture is done based on it.
Conclusion: domain distribution is essentially a passive inheritance of the source datasets' distribution, rather than an active design. [Uncertain]

### [Ovi](../models/Ovi.md) ⚠️

The paper discloses a clear and unusual "person-composition mixture control" mechanism, but does not give specific proportion figures.
[Explicit mixture control] An internal face detection model is used to divide clips into three categories and "ensure an adequate mix": single-person video, multi-person video, and person-free video. The motivation given in the paper is "to let the model learn to generate video across diverse contexts, without overfitting to a specific subtask" — i.e., avoiding degeneration into a pure talking-head model as most A2V models do. The specific percentages of these three categories are not disclosed [Uncertain].
[Qualitative composition of the corpus] The internal audio-video corpus is described as "human and nonhuman data from diverse contexts."
[Indirect evidence of domain coverage] The cross-modal attention visualization in Section 5.1 of the paper is organized by content category, covering instrument playing, bird calls, rockets, animals, speech (multiple examples), helicopters, sports, and other categories, from which the training distribution can be inferred to cover at least: person dialogue, instruments/music, animals, vehicles/machinery, sports, and natural ambient sound.
[Domain on the sound-effect side] By incorporating VGGSound / AudioSet / WavCaps, sound-effect domain coverage is indirectly determined by the taxonomies of these three public datasets (the AudioSet 632-class ontology, etc.).
[Not addressed] The paper does not mention proportions for style (realistic/animation/CG), scene category proportions, action category proportions, or concept-balancing strategies [Uncertain].

### [Script-a-Video](../models/Script-a-Video.md) ⚠️

Disclosure at the domain level stops at category enumeration, with no mixture figures and no description of a concept-balancing mechanism:
[Domain composition of the 500K caption side] Three qualitative categories are listed: film, television, lifestyle. The share of each is not disclosed, nor is it stated whether any mixture adjustment was done. This choice clearly skews toward content that is "narrative, has dialogue, has multiple shots, and has people" — highly coupled with the design goals of the MTSS schema: the Reference stream needs recurring characters, the Shot stream needs real shot transitions, and the Event stream needs dialogue and sound effects. In other words, the data domain was selected in reverse from the schema's requirements, rather than in pursuit of general-purpose coverage.
[Implicit domain division on the generation side] The four sets of data are actually divided by "capability dimension" rather than "subject-matter dimension": identity-centric (character-identity capability), multi-shot (shot-structure capability), cinematic AV pairs (audio-visual coordination capability), cinematic alignment pairs (high-fidelity alignment capability). This is an approach of "organizing data by target capability," where each dataset serves the specific learning objective of one training stage, rather than balanced sampling by content subject matter.
[Domain coverage on the evaluation side] The 225 evaluation samples cover four categories — film/TV clips, short video, indoor scenes, outdoor scenes; the paper states this "covers a diverse range of categories and scenarios," but no per-category proportions are given.
[Concept balancing] The paper does not describe any concept balancing, long-tail supplementation, category quotas, or resampling mechanism.
[Taxonomy internal to MTSS] It's worth noting that the MTSS schema itself has a built-in entity classification system: the Reference stream divides entities into four categories — person, object, animal, scene — and only retains entities that are integral to the main plot, with peripheral elements uniformly downgraded to the global scene description. This is a form of "narrative-importance-driven entity filtering" that constitutes an implicit domain structure at the annotation level, but is not a sampling mixture for the training data.
Overall, the subject-matter mixture figures for the training data are an information gap. [Uncertain]

### [Seedance 2.0 and Seedance 1.5 pro](../models/Seedance_20_Seedance_15_pro.md) ⚠️

The category distribution of the 1.5 and 2.0 training data is not disclosed, but its mixture intent can be reconstructed from two pieces of circumstantial evidence: (1) the "diversity-oriented data collection" reported for Seedance 1.0 explicitly lists the dimensions to be maximally covered: clip duration, resolution, subject (people/animals/objects), scene type (natural scenery/urban environment), subject action, genre (documentary/animation), artistic style, camera kinematics, and cinematographic technique; and in the "distribution rebalancing" step, frequencies are tallied along dimensions such as subject category, scene type, dominant action, genre, visual style, clip duration, resolution, and motion characteristics, with downsampling applied to overrepresented head categories and increased sampling probability plus targeted data acquisition applied to long-tail categories, aiming to form "a more equitable, more comprehensive representation" of the visual world; the SFT stage defines "several hundred categories" (divided by key attributes such as visual style, motion type) for targeted collection. (2) The Seedance 1.5 pro report states that its data pipeline prioritizes "video-audio consistency, motion expressiveness, and curriculum-based data scheduling," indicating the mixture explicitly skews toward samples with high motion expressiveness — echoed by its addition of a "Video Vividness" metric in SeedVideoBench 1.5 (split into four dimensions: action, camera work, atmosphere, emotion), with the report also criticizing the common industry practice of trading motion for stability via slow motion. On the Seedance 2.0 side, categories such as complex interaction and human motion, multi-subject interaction, multi-shot narrative, and Chinese dialect/opera/singing performance are strengthened. [Uncertain: all specific proportion figures]

### [SkyReels series](../models/SkyReels.md) ⚠️

This is the most prominent methodology on the data side of the SkyReels series — "concept balance" runs through both generations.
[SkyReels-V2] (1) Tagging comes first: the "subject category" field in the structured output of SkyCaptioner-V1 serves as the classification basis for balancing; (2) balancing is executed at the post-training stage — "detailed concept balancing is performed based on the caption generator's subject categories, which reduces the data volume by 50%," i.e., half the data is cut to eliminate head-category bias, a direct expression of "quality over quantity"; (3) concept-balanced image data is also used jointly with video during the pretraining stage; (4) blacklist-style removal at the video-type level: surveillance footage, game recordings, animation, and meaningless content are filtered out as entire categories, indicating the target domain is locked onto genuinely filmed film/TV and lifestyle content; (5) SkyCaptioner-V1's own training set is likewise "2 million concept-balanced videos curated from 10 million." The paper does not publish the specific taxonomy or per-category proportion figures.
[SkyReels-V4] Continues and extends this into two orthogonal dimensions: (1) conceptual diversity — matching-based balancing based on the taxonomy; (2) motion diversity — key motion patterns are defined for each subject category or scene category, and balancing is then done by motion pattern, avoiding a single category of subject appearing with only one type of action. This "category × motion pattern" two-dimensional balancing is V4's main increment over V2. The specific category tree and mixture figures are not disclosed. [Uncertain]

### [Sora 2](../models/Sora_2.md) ⚠️

Nothing is disclosed at all. The System Card gives no mixture figures for any category such as people/actions/scenes/styles, does not describe any concept-balancing strategy, and does not disclose how long-tail concepts are handled. The only indirectly inferable clue comes from capability descriptions: the model is emphasized as performing better on physical laws (gravity, momentum, buoyancy, material deformation, collision dynamics, object permanence); third-party interpretations claim the training data carries "physical annotations" covering these concepts, hinting at a possible targeted data mixture or annotation scheme aimed at physical interaction — but this claim comes from secondhand technical interpretation, not an official OpenAI statement, and carries no proportion figures whatsoever. In addition, the product form factor (cameo real-person appearances, social feed) suggests the share of person/face-category data is not low, but again there is no official basis for this. [Uncertain]

### [SpeakerVid-5M](../models/SpeakerVid-5M.md) ⚠️

The paper gives a qualitative taxonomy and distribution charts, but provides no proportion figures whatsoever:
[Interaction-form dimension (the dataset's primary organizational axis)] This is SpeakerVid-5M's most distinctive way of dividing domains — dividing into four major branches by interaction role rather than by visual content: (1) the dialogue branch, two-person conversation; (2) the single branch, monadic talking; (3) the listening branch (non-speaking-side reaction behavior within a conversation); (4) the multi-turn branch. The four branches derive from the same batch of raw video, via different organization and pairing methods. This "speaking / listening / one-directional / multi-turn" classification scheme directly corresponds to the capability breakdown of downstream tasks.
[Content genre dimension] Interviews, news reports, seminars, TV programs, variety shows, debates, educational videos.
[YouTube channel category dimension] Entertainment, People & Blogs, Comedy, News & Politics, Education, Science. Figure 3(c) graphically shows the topic distribution and year distribution (2006–2025), but the body text does not list percentages.
[Concept-balancing mechanism] The paper does not describe any explicit category mixture control or concept-balancing strategy; on the collection side, there is only the qualitative criterion of "manually screening high-quality two-person dialogue video."
[Strong person-centric bias] All data consists of real-person speaking/listening scenes, with no non-person domains such as object motion, natural scenery, or animation — this is a highly vertical person-dialogue dataset rather than a general-purpose video-generation dataset. [Uncertain]

### [Step-Video-T2V](../models/Step-Video-T2V.md) ⚠️

The report gives no domain proportion figures, but provides a concept-balancing mechanism that is fairly distinctive among contemporaneous works — "Video Concept Balancing":
· An in-house VideoCLIP model is used to compute a video embedding for each clip;
· K-means clustering is done in the high-dimensional embedding space, forming over 120,000 clusters — this number of clusters is far higher than contemporaneous comparable works (e.g., HunyuanVideo's roughly 10,000 concept centers), meaning the concept division granularity is extremely fine;
· Two derived labels are assigned to each clip: Cluster_Cnt (the number of samples in its cluster, used to identify over-dense/long-tail concepts) and Center_Sim (cosine distance to the cluster center, used to identify outlier samples within a cluster);
· Based on these two labels, two types of operations are implemented: first, resampling by cluster size, ensuring broad category coverage and suppressing excessive concentration of head concepts; second, at the post-training stage, removing outlier clips far from their cluster center by Center_Sim (using clustering for both "concept balancing" and "outlier quality control" simultaneously is a point of considerable methodological value in this entry).
The final mixture for each specific category (people/actions/scenes/styles) and the specific resampling ratios are not published. On the Step-Video-TI2V side, however, there is a clear disclosure of style imbalance: anime-style video accounts for over 80% of the training data, and only anime data was used in the early stage, causing this model to have strong anime performance but constrained real-scene performance — this is a rare, officially self-reported case of domain-mixture imbalance and its consequences. [Uncertain]

### [UniTalking](../models/UniTalking.md) ⚠️

UniTalking is a highly vertical, single-domain dataset — the target domain is extremely narrow, so there is no multi-category mixture problem in the traditional sense:
[The sole domain: person-centric speaking scenes] The entire pipeline's design goal is explicitly stated as "isolate high-quality, human-centric speech content." Every level of the three-tier filtering converges on this single objective: the video-level tier removes static frames (ensuring the person has motion), the audio-level tier removes samples without speech (ensuring there is speaking), and the cross-modal-level tier removes samples where lips and audio are unsynchronized (ensuring the speaking person is actually in frame). The final product is referred to as a human-centric dataset.
[Comparison with UniVerse-1's mixture orientation] UniVerse-1 is "general soundscape-first," with speech accounting for only 15.4% and non-speech soundscapes 84.6%; UniTalking, in contrast, is close to 100% speech/speaker content — the exact opposite extreme. This directly corresponds to the difference in the two models' capability positioning: UniVerse-1 pursues general-purpose audio-video generation, while UniTalking specializes in talking portraits. The paper's conclusion admits that "although validated only on speaker generation, we believe this framework can be generalized to general audio-video synthesis, including sound effects and music" — i.e., generality is merely a claim, unsupported by data or experiments.
[Undisclosed mixture dimensions] None of the following breakdown statistics exist for the 2.3 million internal clips: no person-attribute distribution (gender/age/ethnicity), no scene distribution (studio/outdoor/conference/vlog), no shot-scale distribution (close-up/half-body/full-body), no single-person/multi-person ratio, no frontal/profile ratio, no source mixture ratio between OpenHumanVid and internal data. Nor is there any description of a concept-balancing or resampling mechanism.
[An inferable skew risk] OpenHumanVid's original source is a collection of public datasets, while the source of Huawei's internal data is unspecified; the difference in person and scene distribution between the two cannot be judged, and the paper gives no explanation of cross-source distribution alignment. [Uncertain]

### [UniVerse-1](../models/UniVerse-1.md) ⚠️

Domain coverage of the training data has only qualitative enumeration, no proportion figures whatsoever, and no description of a concept-balancing mechanism:
[Content categories] Music variety shows, classical music performance, cooking tutorials, public speeches, interviews, vlogs, tool-use demonstrations, movie clips, Pexels stock footage.
[The only quantifiable mixture split] A three-way split by data "nature" rather than "subject matter": speech-centric 1,187 hours (15.4%), general audio-video 3,074 hours (40.0%), VGGSound+AudioSet 3,422 hours (44.5%). It's clear that speech data accounts for only 15%, while sound-effect/ambient-sound-category data holds an absolute majority — the exact opposite of MOVA's extreme orientation of "retaining only speech clips." UniVerse-1's mixture is general-soundscape-first, with speech as a secondary component.
[Indirect evidence of subject-matter mixture intent] The chosen subject matter corresponds one-to-one with three target capabilities (music category → instrument sound, speech/interview category → speech lip-sync, cooking/tool category → Foley and ambient sound), indicating the mixture was collected by reverse-engineering from "target sound type," rather than balanced by visual subject matter.
[The only percentage category table] The nine-category audio proportions given in the paper's appendix (natural environment 36.1%, etc.) describe the sample distribution of the Verse-Bench evaluation benchmark, not the training-data distribution, and should not be conflated with it.
The subject-matter domain mixture figures for the training data are an information gap. [Uncertain]

### [Unison](../models/Unison.md) ⚠️

The paper gives no domain mixture figures whatsoever, and does not describe a concept-balancing mechanism. The following is a qualitative reconstruction based on the composition of the data sources:
[Positioning: human-centric] This is the positioning declared right in the paper's title. Of the five audio-video data sources, four are face/person datasets (OpenHumanVid, HDTF, VFHQ, CelebV-Text), and only VGGSound is a general audio-visual event dataset — so the absolute bulk of the corpus is footage containing people.
[Subject-matter coverage inferable from each source dataset]
- OpenHumanVid → person motion, daily activities, person scenes from film/TV clips; the primary source of person-motion diversity;
- HDTF → frontal high-definition head shots from speeches and news broadcasts, contributing standardized lip-sync samples;
- VFHQ → interview-scene faces, contributing identity and expression diversity;
- CelebV-Text → in-the-wild face video with attribute/action/emotion text annotations, contributing face-attribute diversity;
- VGGSound → 310 classes of audio-visual events (instrument playing, animals, traffic, machinery, nature, etc.), the sole non-person domain source, and the visual anchor for the model's ambient-sound capability.
[The "domain" dimension the paper actually cares about is not subject matter but acoustic scene type] From the qualitative discussion, it can be seen that the domain division Unison genuinely cares about is a three-way split by "the relative relationship between speech and sound effects":
1) Narration-dominant scenes (e.g., dense voiceover) — the SCG gate suppresses the sound-effect stream to protect speech purity;
2) Complex soundscape scenes (e.g., music performance) — the SCG gate amplifies cross-stream influence to enrich non-speech acoustics;
3) Difficult scenes where both matter equally — the paper repeatedly gives examples: "singing while playing an instrument," sports commentary (the commentary should not drown out crowd cheers and impact sounds), beach scenes (voice should not overpower ocean waves), piano performance (note onsets must correspond to finger movements), and motorcycle riding (engine sound should not be overpowered by voice).
This scene classification "by speech-to-sound-effect ratio rather than by visual subject matter" is a distinguishing feature of Unison, and directly corresponds to the instance-level analysis of its SCG gate (Fig. 8c gives average gate values across semantic categories, but the paper does not list the category roster or their proportions).
[Information entirely missing] The hour/clip-count share of each data source, a subject-matter category table with proportions, person-attribute (gender/age/ethnicity) distribution, motion-type distribution, scene-type distribution, and any form of concept balancing or reweighting mechanism. The paper has neither a domain mixture table nor any statement of whether certain subsets were up- or down-sampled during training. [Uncertain]

### [Veo 3 / Veo 3.1](../models/Veo_3_Veo_31.md) ⚠️

[Uncertain] The category/domain mixture strategy is not disclosed. Official clues that can be inferred backward: (1) the Model Card explicitly states that "synthetic captions [are used] to improve the variety and diversity of concepts" associated with videos in the training data — a concept-balancing strategy achieved via caption-side rewriting rather than direct sample resampling; (2) in the "Dangerous Capabilities" section, the technical report notes that Veo 3 has "a bias for generating cinematic footage, with frequent camera cuts and dramatic camera angles," and consequently has difficulty generating low-production-value realistic coercive-type video — this strongly suggests the training data is significantly skewed toward film/professional cinematic and high-production-value footage, with low-production-value domains such as UGC/surveillance/casual phone footage underrepresented; (3) fairness evaluation shows the model clearly "skews towards lighter skin tones" when ethnicity is unspecified, and exhibits semantic bias associated with specific vocabulary and specific demographic groups, reflecting that the person-representation distribution of the training data is itself imbalanced; (4) the official team admits text rendering remains relatively weak, hinting at insufficient data coverage or annotation for scenes containing text/OCR. All of the above are inferred backward from public reports, not an official mixture disclosure.

### [Vidu S1](../models/Vidu_S1.md) ⚠️

No quantitative mixture is given, but there are strong qualitative domain constraints, making this in essence a highly vertical data distribution:
(1) Subject constraint: each clip must contain exactly one subject, and the subject must occupy a reasonable proportion of the frame — i.e., a strictly "single-subject" distribution;
(2) Shot constraint: only static shots or shots with slow motion are retained, to reduce the risk of camera drift in long-duration generation — deliberately suppressing the proportion of large camera movements;
(3) Interactivity constraint: the subject in frame is required to display clear actions or behaviors, to ensure the model learns meaningful motion information;
(4) Style coverage: multiple angles, multiple scenes, and multiple visual styles are supplemented via film/TV footage; on the product side, support for a variety of custom character images — real people, anime, pets, etc. — suggests the training data covers anime/2D and non-human subjects (the paper mentions that the face-detection expert model generalizes poorly to exaggerated/highly stylized 2D animated subjects, hence the introduction of an omni model to supplement);
(5) Semantic label dimension: the omni model assigns labels along nine quality dimensions — editing, subject, action, emotion, face, speech, scene, shot, tone — and these nine dimensions constitute a de facto domain-description scheme, though the paper does not publish mixture figures for each dimension.
The ratio between the two major sources (livestream/talking-head vs. film/TV) is likewise not disclosed [Uncertain].

### [Wan 2.5 / 2.6 / 2.7](../models/Wan.md) ⚠️

This is one of the most valuable disclosed parts of Wan 2.1; 2.5+ discloses nothing.
[Clustering to preserve the long tail (core design)] Visual-quality screening is split into two steps: "clustering + scoring." The full dataset is first divided into 100 clusters, and a certain amount of data is then taken from each cluster to proceed to the next stage. The report explicitly explains the purpose of doing this: to prevent the long-tail distribution from causing "small but important data segments" to be entirely wiped out by global-threshold filtering, thereby preserving the original data distribution. This is a typical approach of "quota sampling by cluster unit" to counteract the distribution drift that quality filtering introduces — sharing the same motivation as, but a different mechanism from, LTX-2's use of a multi-label network to constrain paired sampling.
[Six-tier motion-quality grading with differentiated sampling rates] Video is divided into six categories by motion quality, each with its own sampling priority:
1) Optimal motion (significant motion in layout/perspective/amplitude, clean and smooth) — highest priority;
2) Medium-quality motion (has clear motion but with multiple subjects or partial occlusion) — retained to preserve motion diversity, helping the model understand spatiotemporal relationships;
3) Static video (mainly chat/interview-type content, low motion information but high image quality) — separately identified and given a "reduced sampling ratio";
4) Camera-dominant motion (aerial footage, etc., where the subject barely moves) — given a "greatly reduced sampling priority" since it approximates a static image;
5) Low-quality motion (too many subjects, severe occlusion, unclear subject, e.g., crowded street scenes) — excluded outright;
6) Shaky footage (amateur handheld, motion blur, foreground/background hard to distinguish) — systematically excluded.
[Post-training category balancing] Video post-training explicitly "emphasizes category balance and high diversity," selecting data from 12 major categories; the categories named in the report include technology, animals, arts, humans, vehicles, etc. In addition to taking the top 20% by score, image post-training additionally "considers style and category factors to ensure data distribution diversity," and manually fills in concepts missing from the dataset to enhance generalization.
[Staged three-dimensional mixture] Fig. 3 of the report, "Data provisioning across different training phases," makes clear: for each training stage, the data proportions along the three dimensions of motion, quality, and category are dynamically adjusted based on data throughput. This is the most transferable point in Wan's data methodology — the mixture is not static but scheduled on a rolling basis by stage.
[Corroborating evidence from the same family] Wan-Dancer covers five dance genres — Chinese classical dance, K-pop, Latin, tap, and street dance — and "ensures an approximately uniform duration distribution to mitigate category imbalance."
The specific percentages for each category, the specific sampling weights for the six motion tiers, and the quota figures for the 100 clusters are all unpublished. [Uncertain]

### [Collection of Audio-Video Generation Evaluation Benchmarks](../models/av_benchmarks.md) ⚠️

This item has the most reverse-guidance value for the training-data side in this survey — the taxonomies of the five benchmarks can directly serve as a standard coordinate system for training-data domain mixture.

[VABench's seven-category system (the most complete content-side taxonomy)]
1. Animals: species-specific vocalization and audio-visual behavioral consistency;
2. Human Sounds: subdivided into linguistic (containing semantic content, involving lip sync) and non-linguistic (physiological/action-type, such as coughing, clapping, footsteps);
3. Music: structured audio across genres, examining melodic and rhythmic coherence;
4. Environmental Sounds: further split into three soundscapes — natural, urban, indoor;
5. Synchronous Physical Sounds: instantaneous/rhythmic physical interaction sounds, requiring strict adherence to material properties;
6. Complex Scenes: higher-order scenes, containing five sub-dimensions — complex soundscapes, subjective feeling, world knowledge, symbolic association, off-screen (invisible) sound sources;
7. Virtual Worlds: unrealistic scenes that transcend physical laws, used only for the T2AV task.
Data volume allocation: 778 T2AV clips + 521 I2AV clips are distributed according to the above category hierarchy (the paper presents this as a sunburst chart, without giving exact per-category counts [Uncertain]). Note that category 7, "Virtual Worlds," does not participate in the two physical-plausibility scores of Audio Realism / Visual Realism — this design of "scoring realistic and stylized categories separately" is likewise applicable to setting domain-specific quality thresholds for training data.

[AV-SyncBench's ten-scene system (an orthogonal split from the synchronization perspective)]
Action, Animal Sound, Object Sound, Ambient Sound, Group Vocalization, Single Speaker, Dialogue, Singing, Single Instrument, Ensemble. At the top level these are merged into three major audio classes: Voice / Music / Sound. The value of this scheme lies in splitting by "synchronization difficulty" rather than "content theme": the distinction between Single Speaker vs. Dialogue, and Single Instrument vs. Ensemble, directly corresponds to the mixture requirements for multi-source-overlap scenes in training data.

[PhyAVBench's six physical dimensions + 41 fine-grained test points (the physical-correctness perspective)]
① Sound-source mechanics (material hardness/damping, density, surface texture, object geometric size/shape/thickness, contact dynamics such as impact velocity/contact area/excitation continuity); ② Fluid and aerodynamics (flow velocity, liquid impact/splashing, Helmholtz resonance, container material, viscosity, bubbles, aerodynamic whistling); ③ Sound propagation environment (reverberation's spatial volume and surface absorption, echo, spatial transitions, diffraction, scattering, underwater acoustics, solid-borne sound transmission, vacuum, occlusion and sound insulation); ④ Observer physics (inverse-square distance law, air absorption, the Doppler effect for approaching/receding and rotating sound sources, binaural horizontal/vertical localization); ⑤ Time and causality (far-field delay and near-field synchronization caused by the difference between light and sound speed, transient synchronization of onset and offset, rhythmic consistency of periodic/aperiodic motion); ⑥ Complex coupling and extreme physics (phase changes such as boiling/freeze-shattering, supersonic and nonlinear distortion in explosions and shock waves). This system can directly serve as a checklist for the "physical-acoustic coverage" of training data.

[AVBench's scene stratification] Centered on real-person scenes, stratified by number of speakers and acoustic difficulty: Normal (1–2 speakers) 350 clips / Hard (3–4 speakers + speech overlap + noisy background) 120 clips, with a Hard Quota-Based Greedy Sampling constraint applied, forcing any single attribute's share to be ≤50% — this is an explicit concept-balancing mechanism that can be directly transplanted as an attribute-quota control strategy for training-data sampling.

[Omni-Judge] Sets no content categories; its 300 prompts are taken directly from the real-user distribution of VidProM, representing an alternative domain benchmark of "real user intent distribution."

### [Video Captioning Model Ecosystem](../models/caption_models.md) ⚠️

[Concept balancing is an explicit design goal for tagger training (one of the few works that handle it explicitly)]
· SkyCaptioner-V1: trained on "roughly 2 million concept-balanced videos, curated from 10 million" — the clearest category-balancing statement in this ecosystem; its camera-motion sub-expert further uses "16,000 pieces of motion-axis-balanced synthetic data" to fill in long-tail camera-movement types.
· The six-source mix of AVoCaDO-SFT-107K is itself a domain mixture design: TikTok-10M 24K (UGC short video) + ShortVideo 18K + Shot2Story 20K (multi-shot narrative) + FineVideo 29K (the largest portion, educational/lifestyle) + YouTube-Commons 11K (long-tail general) + CinePile 5K (film/TV) — i.e., UGC as the main body, with film/TV as a small, high-quality supplement.
· AVSCap-130K's sources are AVoCaDO-107K + ASID-1M + FineVideo + TimeChatCap-40K + Movie101; the inclusion of Movie101 significantly raises the share of film/TV narrative content.
· Panda-70M's greedy multi-teacher coverage set is, in essence, an implicit domain-handling approach of "dispatching taggers by content type": the single best tagger covers only 30.8% of samples, using all 31 covers 84.7%, and the 8 greedily selected cover 76.8% — indicating that videos of different domains require different taggers, and no single model can cover everything. This is the most important domain-distribution insight in this ecosystem.
[Domain taxonomy on the evaluation side] AVSCapBench videos come from three sources: YouTube / TikTok / Video-MME; Omni-Cloze covers 9 major domains and 47 subcategories (the only Alibaba work to publish a complete taxonomy); UGC-VideoCap focuses on TikTok UGC (1,000 clips); VDC's structured captions are split into five categories — camera / short / background / main object / detailed (this is a field dimension, not a domain dimension).
[Domain requirements the generation side places on captioners] Allegro uses Tag2Text's label output directly as the basis for its three-category (people / objects / landscapes) distribution statistics — i.e., the tagger doubles as a data-distribution statistics tool; Goku's distribution-balancing design relies on caption semantic clustering; LongCat-Video uses an LLM to name categories for the clustering results of caption embeddings.
[Gap] Aside from SkyCaptioner-V1, no captioner publishes a domain histogram of its training data or quantified gains from its balancing strategy. [Partially uncertain]

### [Collection of Geometric / Structured Annotation Datasets](../models/geometric_datasets.md)

SceneScribe-1M: derived from open-domain web video from HD-VILA-100M/Panda-70M/Koala-36M, covering multiple categories of everyday scenes; no explicit category mixture control is done, and selection preference is driven by "motion diversity" (the initial screening substantially shrinks the source pool due to motion-diversity requirements). SpatialVID: domains are actively constructed via motion keywords (walking first-person view, urban/nature touring, drone aerial photography), with explicit balanced sampling of category distribution and motion diversity done for SpatialVID-HQ; the structured label scheme covers five dimensions — weather, time of day, crowd density, lighting, scene type — enabling queryable domain control; compared with Panda-70M, SpatialVID-HQ's distribution is shown to be more concentrated across the three metrics of aesthetics, brightness, and motion, with 80% of clips having curved or turning trajectories. WildWorld: the domain is highly controlled but narrow, being entirely combat/exploration scenes from a single game world; the test set WildBench has an explicit mixture by scene type — 100 cooperative-scene clips (player + 3 NPCs fighting a monster) and 100 one-on-one-scene clips, covering multiple characters, multiple weapons, multiple monster species, multiple difficulties, and multiple events (skill activation, knockdown, death, critical hit). Action100M: the instructional-video domain (HowTo100M), with an open-vocabulary, extremely long-tailed action taxonomy; the authors use k-means semantic resampling (k=10³/10⁴/10⁵) to mitigate the long tail, and tally 7.58M duplicate groups totaling 141.8 million duplicate instances that need deduplication before resampling.

### [Post-Training Data Strategy for Video Generation](../models/post_training_data.md) ⚠️

This is the most differentiating dimension in post-training data strategy, and the one that best reflects a team's engineering maturity. The anchor paper is entirely blank on this (it only says "curated the prompt set") [Uncertain], but across the field there are four clearly distinguishable mixture paradigms:
[Paradigm 1: inverse-density sampling for long-tail lift (the most elegant)] LongCat-Video's SFT curated set, on top of multi-metric filtering (aesthetic score/video quality/motion quality), has a second layer that samples "inversely proportional to their density in the caption embedding space," directly achieving a relative lift for long-tail concepts. This is a representative approach that formalizes "concept balancing" as a computable criterion.
[Paradigm 2: k-NN concept balancing + manual selection for "cinematic feel"] Movie Gen's video SFT set is produced through four stages: automated strict-threshold filtering → k-NN concept balancing → manual selection for "cinematic feel" → manual caption rewriting; the paper explicitly states that after the first stage there remain "several million clips but concept-imbalanced," i.e., concept balancing is a second process independent of quality filtering.
[Paradigm 3: classifier-based domain split + independent per-domain SFT (the Physical AI route)] Cosmos-Predict 2.5 trains a multi-head classifier on InternVideo2 embeddings to split samples into five domains, trains an independent SFT model per domain (30k iterations, batch 256, each), and then performs model merging. The scale across domains differs enormously (object permanence 10.4M vs. robotic manipulation 730K), reflecting the priority given to the basic physical common sense that "objects don't vanish due to occlusion." Per-domain human win rates are all significantly better than the pretraining baseline: robotic manipulation 72.6% vs. 8.3%, object permanence 50.9% vs. 27.7%, high motion 44.0% vs. 34.7%, complex scenes 42.6% vs. 35.4%, driving 47.9% vs. 28.8%.
[Paradigm 4: targeted collection across hundreds of categories + sub-model fusion] Seedance 1.0 defines "several hundred categories" by attributes such as visual style and motion type for targeted collection, trains multiple sub-models each covering different styles/motions/scenes, and then does model merging, using a smaller learning rate than pretraining and a limited number of GPUs combined with early stopping to prevent overfitting and preserve text controllability. This is the idea of "replacing a single mixed SFT set with the weight-averaging of multiple specialized SFT models," sharing the same lineage as Movie Gen's model averaging.
[Mixture on the preference-data side] HunyuanVideo 1.5's T2V RLHF prompt set is category-balanced across the three dimensions of motion/scene/subject, sourced from a mix of LLM-generated prompts and training captions; on the I2V side, a prompt set covering 100+ categories is constructed, with accompanying images curated from high-aesthetic images and manually verified for image-text consistency. Seedance 1.0's RLHF prompts undergo "data balancing and information filtering" to eliminate duplicate and vague prompts.
[Motif-Video 2B's iterative weak-point filling] The SFT corpus is assembled through an iterative weak-point-filling approach: on top of the regular filtering, a stricter aesthetic cutoff is layered on, along with domain balancing driven by style/subject labels, and the video-side action=Dynamic dynamic-motion admission criterion.

### [Combined Survey of Mainstream Video Pretraining Datasets: Panda-70M, InternVid, Koala…](../models/pretraining_datasets.md) ⚠️

**The level of category governance among the seven varies enormously, ranging from "none at all" to an "explicit taxonomy of 7 themes and 108 subcategories"**:
- **Panda-70M (no taxonomy)**: the paper has **no category pie chart or percentages**; Appendix E only lists 8 categories used as illustrative examples for visualization (animals, scenery, food, sports activities, vehicles, tutorials and narratives, news and TV programs, gaming and 3D rendering), with no numbers at all; Figure 13 is a word cloud of 100K captions. The authors proactively acknowledge in the limitations section that, because HD-VILA-100M skews "talking-head-dense," **the main categories are actually news, TV programs, documentaries, first-person video, and instructional/narrative video**. If a category distribution is needed, one can only cite the 15-category balanced sampling of the upstream HD-VILA-100M.
- **Koala-36M (no taxonomy)**: also sourced from HD-VILA-100M; the paper likewise gives no category distribution.
- **InternVid (16 themes, but only a pie chart)**: category distribution is presented only as a pie chart in Figure 3; **neither the body text nor the appendix has a numeric table**, so precise percentages cannot be cited. The citable distribution is source-video duration (≤5 min 49%, 5–10 min 26%, >20 min only 8%) and country of origin (UK, US, Australia, Japan, South Korea, China, Russia, France, etc.). The authors proactively state that, for copyright reasons, four categories — **surveillance footage, sports events, movies, documentaries** — were deliberately made scarce or excluded.
- **MiraData (a 7-category YouTube taxonomy, but only a bar chart)**: (1) 3D-engine-rendered scenes, (2) urban/scenic fly-throughs, (3) movies, (4) first-person view, (5) object creation/physical-law demonstrations, (6) time-lapse photography, (7) human motion demonstrations. **Categories (1) and (3) are deliberately oversampled**, on the grounds of "better diversity, higher image quality," and the paper argues that 3D-engine footage helps with learning physical-law consistency. **The count for each category exists only in the unlabeled bar chart of Figure 2** (y-axis 0–12K videos / 0–78K clips), with no numeric table; however, the `source` column of the metadata CSV actually encodes platform + category (e.g., `youtube, 3D engine-rendered scenes`, `videvo, nature`, etc.), **allowing precise composition to be recovered via one's own value_counts** — this is the only usable path for category recovery in this survey. Note: **there is no dedicated GTA-V source**; game content is classified under "3D-engine-rendered scenes," and the paper deliberately does not name specific games (presumably for copyright reasons).
- **LVD-2M (8 categories, with precise percentages, the most practically useful)**: **BART** is used to classify captions — scenery 24%, people 23%, transportation 13%, sports 11%, animals 11%, food 9%, gaming 5%, other 4%. This is the **only dataset among the seven to give a complete numeric category distribution**.
- **UltraVideo (7 major themes × 108 subcategories, the most complete scheme but missing numbers)**: the taxonomy is obtained via "noun statistics on Koala-36M captions → LLM induction → manual revision and confirmation." The seven major themes are: **video scene, subject, action, temporal event, camera movement, video type, emotion**. This is the only scheme among the seven to incorporate "camera movement" and "emotion" as data-mixture dimensions, and **uniform sampling across categories** is done based on it (explicitly stated as uniform per-category sampling by theme similarity). **However, the 108 subcategories are never listed in the body text, appearing only as a proportion chart in Figure 4(a)**; GitHub issue #5 specifically asks about the theme↔video mapping relationship, and remains unanswered to this day.
- **OpenVid-1M**: no taxonomy disclosed. [Uncertain]
**Overall assessment**: Only UltraVideo did the active mixture design of "define the taxonomy first, then sample uniformly by category"; MiraData did coarse-grained, targeted oversampling of categories; LVD-2M only did after-the-fact statistics; the category distribution of the remaining four is entirely a passive inheritance of the upstream corpus distribution. **None of the seven did long-tail concept balancing or resampling after semantic clustering.**
## Audio Category Distribution and Mixture (Ratios and Control Strategies for Speech/Sound-Effects (Foley)/Music/Ambient Sound/Silence) — A Dimension Unique to AV Models

`audio_category_distribution` · Detail level: detailed

### [Allegro](../models/Allegro.md)

Not applicable. Allegro is a purely visual video generation model; its training data neither retains nor processes audio tracks, and the paper makes no mention anywhere of categorization or mixture ratios for speech/sound effects/music/ambient sound/silence.

### [Apollo](../models/Apollo.md) ⚠️

This is the core dimension of Apollo's data design, and the key point that distinguishes it from vision-first peer works — the entire dataset is split hierarchically (tree-structured) by audio type (Section 4.2, Audio-Guided Data Splitting):
【Level 1: Vocal vs. non-vocal】 The data is first split into two branches — vocal (containing human voice) and non-vocal — with the non-vocal branch forming the sound split (the natural-sound/sound-effects subset).
【Level 2: Three-way split within vocal】 The vocal subset is further divided into three categories — singing, single-speaker speech, and multi-speaker speech.
【Final four categories】 single-speaker speech / multi-speaker speech / singing / natural sound. Original text from the paper: "The dataset contains single-speaker speech, multi-speaker speech, singing, and natural sound clips."
【Differentiated annotation】 Different subsets go through different annotation pipelines — the speech and singing subsets additionally extract speaker attributes (gender, age) and undergo word-for-word transcription; the sound split only receives audio captions, without transcription. This is a typical design of "routing by audio category first, then annotating each branch separately."
【Comparison with peer works】 Notably, Apollo explicitly retains singing as a category, whereas works like MOVA, constrained by the capacity of their audio tower, degrade in performance on singing and either exclude or downweight it; Apollo makes singing a first-level subset, indicating that on the data side it deliberately covers the high-difficulty scenario of singing-related lip motion. At the same time, unlike MOVA's "speech-only" approach, it lets natural sound (foley/ambient) coexist with speech in the same training set, and combines this with multi-task masked training to simultaneously acquire T2A and lip-sync capabilities.
【Gaps】 The sample counts or proportions for each of the four categories, whether music (instrumental music) is broken out as its own category, and the handling of silent samples (all that is known is that samples with a silence ratio >20% are removed) are all undisclosed. [Uncertain] (Only the category taxonomy is clear; the proportion figures are missing.)

### [CineDance / CineDance-1M](../models/CineDance.md) ⚠️

Audio is explicitly decomposed into three categories and annotated separately, but the proportion of each category is not disclosed:
【Three-way split】 In shot-level audio prompts, audio content is decomposed and described separately across three categories — music, ambient sound, and effects; dialogue/speech is instead handled via an independent ASR channel, forming a de facto four-category (speech / music / ambient sound / effects) parallel annotation structure.
【Relationship to speakers】 There is also a separate character voice description field that characterizes each character's vocal timbre.
【Quality-side characterization】 Audio quality is quantified via two metrics — DNSMOS (signal fidelity) and the temporal variance of the CLAP embedding (measuring how richly audio content varies over time, which can indirectly reflect silent/monotonous tracks) — both of which are retained as metadata.
【Proportions and mixture strategy】 The paper does not give the duration or sample proportion of speech / music / effects / ambient sound / silence, nor does it describe any active control strategy for audio-category mixture ratios. This is a point where the dataset's disclosure falls short on the AV dimension. [Uncertain]

### [CogVideoX](../models/CogVideoX.md) ⚠️

Not applicable to CogVideoX itself: the video model's training makes no use of audio tracks at all, the data pipeline contains no audio dimension whatsoever, and there is no categorization or mixture of speech/sound effects/music/ambient sound/silence.
The cascaded CogSound component does involve audio data, but its training-data composition, audio-category mixture ratio, and silence-handling strategy are all undisclosed [Uncertain]. Public information only states that its generation targets cover complex sound effects such as "explosions, flowing water, musical instruments, animal calls, and vehicle sounds" as well as rhythmic/musical elements, suggesting the training data is dominated by foley sound effects with some musical component, but no quantified mixture ratio is given. There is also no statement on whether it generates or models spoken dialogue.

### [Cosmos-Predict2.5](../models/Cosmos-Predict25.md)

Not applicable. Cosmos-Predict2.5 has no audio modality — it does not generate audio, is not conditioned on audio, and its data pipeline contains no audio-track processing step whatsoever (no audio-track extraction, no speech/effects/music classification, no silence detection). No design for audio-category mixture appears anywhere in the paper. If sound-related coverage of Physical AI scenarios is needed, an external audio model would have to be attached separately, which the paper does not discuss.

### [Data-Juicer 2.0](../models/Data-Juicer.md) ⚠️

[Uncertain] Data-Juicer does not provide classification or mixture-control capability for audio categories (speech/sound-effects-foley/music/ambient/silence) — a clear gap relative to AV generation data pipelines.
【Existing audio-side capabilities are limited to】
  · Quality dimension: audio_nmf_snr_filter (estimates SNR via non-negative matrix factorization and filters by interval), audio_duration_filter (duration), audio_size_filter (file size).
  · Semantic annotation dimension: video_tagging_from_audio_mapper and video_audio_ASR_mapper both produce audio-event labels based on the Audio Spectrogram Transformer (AST, trained on AudioSet) — the AudioSet ontology contains 527 classes including speech/music/ambient sound, so in principle this could approximate a three-way audio distinction, but DJ has not packaged it into an explicit three-class field or a mixture-control operator.
  · Speaker-attribute dimension: video_audio_detect_age_gender_mapper (wav2vec2-based age/gender detection), video_audio_speech_emotion_mapper (speech emotion).
【Missing key capabilities】 There is no audio-track source-separation operator (e.g. a Demucs/Bandit-style speech/effects/music three-way separation), so field-level energy gating and track-category mixture control such as Foley-Omni's cannot be done; there is no silence-detection or silence-ratio-threshold operator; there is no dedicated operator for "removing audio-track-less samples" (this must be achieved indirectly via the audio_duration/size filters); and there is no music/vocal separation for stripping background music.
【Practical corroboration】 Its only official video use case (VBench T2V) is a purely visual task, and no audio operator is enabled throughout — indicating the audio-side capabilities have not yet been battle-tested at scale in video-generation scenarios. To build training data for joint audio-video generation with DJ, the audio-category taxonomy would need to be extended with custom operators — fortunately DJ's operator interface design (Mapper/Filter base classes plus YAML registration) keeps the extension cost low, which is also one of the programmability selling points the team claims.

### [Foley-Omni](../models/Foley-Omni.md) ⚠️

This is Foley-Omni's most distinctive dimension, and its core contribution relative to contemporaneous AV works — it elevates "audio category" from an implicit attribute to an explicit, first-class structured field.
【Three-category system】 Speech [WORDS] / Sound Effects [AUDIO] / Music [MUSIC] form three parallel fields of the structured annotation Ŝ. The key design is that "fields can be left empty" (each field can be left empty when the corresponding audio component is absent) — an empty field itself carries information, equivalent to an explicit negative annotation of "this clip contains no audio of this type." This lets the same conditioning interface express both single tasks (filling only one field = TTS/TTA/TTM) and full soundtracks (multiple fields co-present = V2ST), achieving a unified formalization of the task.
【Proportion-control strategy — two-stage determination】
Stage 1 (joint visual + auditory detection): Gemini 2.5 Pro simultaneously receives video frames and the audio track, and is explicitly instructed to "first determine whether a given audio component (Speech/Sound Effects/Music) is physically present, and only if it is present generate the corresponding description" — i.e. it first performs binary presence classification, then generates the description, rather than generating all three descriptions indiscriminately.
Stage 2 (acoustic energy-gating correction): the Bandit source-separation model (Watcharasupat et al., 2024, a speech/effects/music three-way separation model designed specifically for cinematic audio source separation) splits the original track into three stems, and a field's annotation is retained only when that stem's RMS energy E(a_c) > −35 dB. This step is specifically designed to eliminate "visual hallucination" — i.e., cases where Gemini sees a piano in the frame, a person's mouth moving, or a car on screen and assumes the presence of music/speech/sound effects even though nothing is actually audible on the track. This is a methodological innovation the paper explicitly calls out: two-path validation combining visual-path annotation with acoustic-path verification, which the paper states is missing from typical audio-video dataset construction. The −35 dB threshold was determined through "manual inspection of a small validation subset."
【Silence handling】 The very first rule of the filtering stage removes clips containing silence; audio-track-less/silent samples never enter the subsequent pipeline.
【Actual distribution】 The final proportions of the three audio categories in the training set are not given directly, but can be approximated from the task-group hour counts: speech 66%, sound effects 27%, music 3%.
【Explicit mixture in the evaluation set】 The 300 items of V2ST-Bench strictly require "≥2 co-present audio components," with a mixture of speech+effects 150 items (50%), speech+music 120 items (40%), and speech+effects+music 30 items (10%). Speech appears in 100% of the combinations, showing that the benchmark design is likewise centered on speech; the hardest scenario, with all three components present, accounts for only 10%.
[Uncertain] The actual non-empty rate of each of the three fields in the training set, and the sample-count distribution across combination patterns (single-field/double-field/triple-field), are not disclosed.

### [Goku](../models/Goku.md)

Not applicable. Goku is a purely visual image+video generation model whose training data consists of "image-text pairs" and "video-text pairs" and contains no audio track; the paper makes no mention anywhere of any categorization, proportion, or control strategy for speech/sound-effects-foley/music/ambient/silence, and there is no audio channel in the data pipeline. If the original videos came with an audio track, the paper does not state whether it was kept or discarded (judging from the video-text-pair data format, the audio track was likely simply discarded).

### [Hailuo / MiniMax Video](../models/Hailuo.md) ⚠️

Not applicable + [Uncertain]. The Hailuo video model does not generate audio; the official material does not mention at all whether native audio tracks are retained in the training data or how the proportions of speech/sound effects/music/ambient/silence are controlled, and judging from the modeling objective, the audio track was likely simply discarded during data processing. MiniMax's audio-data capability manifests in the separate MiniMax Speech (speech) and MiniMax Music (music) product lines, but the training data for these two lines is likewise undisclosed, and there is no evidence that they share a data-processing pipeline with the video line. This field therefore has no substantive content for this subject.

### [HunyuanVideo-Foley](../models/HunyuanVideo-Foley.md) ⚠️

Audio category is the core organizing dimension of this work's data processing (rather than video category), but again only the mechanism is given, with no numbers:
【Classification tools】 Two models are chained together: (1) a speech-music detection model, performing a coarse-grained speech / music / other distinction; (2) an audio classification model, performing finer-grained acoustic-event classification. Neither is named as a specific model (it is not stated whether it is PANNs, AST, BEATs, EAT, or an in-house model), and no category count is given. [Uncertain]
【Silence handling: a proportional threshold rather than a binary judgment】 Clips whose silence ratio exceeds 80% are discarded. A design detail worth noting here is that a continuous proportional metric — "silence ratio" — is used rather than a binary criterion of "does it contain silence," and the threshold is set fairly loosely (allowing up to 80% silence). The reason for this looseness can be inferred: a large number of samples in Foley scenarios naturally take the form of "long quiet stretches + a brief event sound" (e.g. a single door-closing sound in an otherwise empty room), and tightening the threshold would wrongly kill the most typical Foley training samples. The 80% figure is in fact only used to remove clips that are nearly completely silent — a fallback rule rather than a quality filter. Compared with UniVerse-1's binary approach of "directly removing silent clips using volume/energy/zero-crossing rate," this is better suited to the characteristics of the Foley task. [Uncertain: the specific computation of the threshold, such as the energy threshold or time resolution used for silence determination, is not described]
【Speech and music: detected but not necessarily removed】 The paper only says speech-music detection and labeling are performed, not that speech or music, once detected, is discarded. This means speech and music data are likely retained in the training set, participating in mixture control only through their category labels — a third path different from both MOVA (speech-only retention) and UniVerse-1 (routing and retention by speech presence): retain all categories + label-based mixture. For the Foley task, retaining a small amount of speech and music helps the model learn "when it should not generate sound effects" as well as acoustic layering in mixed scenes, but too much would dilute the learning signal for Foley event sounds — getting the mixture right is exactly what "category-distribution management" is meant to solve, but unfortunately this is undisclosed. [Uncertain]
【Ambient/event sound】 No proportion is given separately.
【Orientation comparison with UniVerse-1】 UniVerse-1 gives an explicit three-way mixture (speech 15.4% / general 40.0% / public datasets 44.5%), whereas HunyuanVideo-Foley, despite a dataset 13 times larger, gives not a single proportion figure. This is a point on which this work is clearly inferior to its peers in terms of data disclosure.

### [HunyuanVideo](../models/HunyuanVideo.md) ⚠️

Not applicable. Neither HunyuanVideo nor HunyuanVideo 1.5 generates audio; the training data involves no audio-track processing, and neither report contains any content on audio-category mixture. For an audio-data methodology within the Tencent ecosystem, one should instead look to the separate work HunyuanVideo-Foley (video-to-audio, a roughly 100,000-hour TV2A dataset, with a multi-label balancing strategy for speech/effects/music). [Uncertain]

### [InstructAV2AV](../models/InstructAV2AV.md) ⚠️

In this work, audio category is organized as a binary split between "speech" and "non-speech sound events," with the two branches going through entirely different processing chains — this is the clearest branch point in the pipeline design.
【The binary system and each branch's processing chain】
  · Speech-dominated content: TalkNet (Tao et al., 2021) performs active speaker detection, and ElevenLabs Scribe (2025) performs ASR to extract precise speech timestamps. The key constraint is that clips retain speech only when temporally aligned with a visible on-screen speaker — this single rule accomplishes three things at once: removing voiceover/narration/commentary, removing post-production dubbing, and guaranteeing that retained samples inherently carry a lip-sync supervision signal. This makes "the sound must come from a visible on-screen source" a hard admission criterion for the data, which is more thorough than filtering after the fact with a sync score.
  · Non-speech content: clips with ambiguous sound sources are filtered out to guarantee source clarity, and each retained clip is then assigned a distinct semantic sound-event label (e.g., "dog barking"). "Distinct/unique" is the key constraint — it requires exactly one dominant sound source within the clip, and clips with multiple mixed sound sources are excluded. The reason is direct: the editing task requires "changing this object's sound while leaving other sounds untouched," and if sound sources within a clip are mixed together, a clean source-target pair cannot be constructed, nor can SAM-Audio separate them reliably.
【Silence and audio quality】 PyDub is used to remove silent clips below −45 dBFS (this is the only threshold in the paper given as an exact figure); Audiobox-Aesthetics (Tjandra et al., 2025) is used for audio-quality assessment, discarding low-quality audio (threshold not given).
【The special status of background/ambient sound】 Ambient sound in this work is not an object to be filtered but an object to be protected — SAM-Audio (Shi et al., 2025) separates the target entity's sound from the original audio track, and the edited new sound is then remixed with the retained background sound. This means the background/ambient sound remains strictly identical between source and target on a per-sample basis, constituting the auditory-side supervision signal for the evaluation dimension of "content preservation." This design serves a different purpose than Foley-Omni's use of Bandit for three-way separation: Foley-Omni's separation is for verifying annotations, while this work's separation is for constructing controlled pairs where "only the foreground sound changes, the background sound stays the same."
【Music】 [Uncertain] The paper does not break out music as a separate processing category and does not mention any detection, retention, or removal strategy for background music. Given that the sources include a large number of movie clips (which commonly carry a score), the score is very likely classified as "background sound," separated by SAM-Audio, and retained as-is, but there is no explicit statement to this effect.
[Uncertain] No actual counts or proportions are given for speech-class vs. non-speech-class samples; no category distribution is given for the sound-event labels (nor is it stated whether the 310-class VGGSound label taxonomy is reused); and no duration-proportion statistics are given for each audio category.

### [Other Joint Audio-Video Generation Works of 2026](../models/JAVG_2026_misc.md) ⚠️

This is a dimension unique to AV models; among this batch of works, ALIVE and NAVA have substantive designs, while the rest are weaker on this front:
【ALIVE — a staged speech/effects two-part scheme + active BGM-proportion management】
(1) The two-stage mixture switch of the audio tower is extremely clear: T2A Stage I is pure speech pretraining, using 700k hours of transcribed speech (384M samples, 1 epoch, lr 5e-5), during which almost everything is speech; T2A Stage II switches to a mix, "approx. 5k hours" of high-quality speech plus "111k hours" of video data with sound annotations (19M samples, 10 epochs) — i.e. a drop from "700,000 hours of pure speech" to "5,000 hours of curated speech," with a large infusion of native video audio tracks. This is a clear scheduling of "large-scale speech foundation → small-scale high-quality speech kept alive + large-scale real soundscape supplementation." Note that the epoch count of Stage II (10) is far higher than that of Stage I (1), indicating the high-quality mixed data is reused repeatedly.
(2) Active management of BGM proportion: original text — "we assess the correlation and separate samples where the audio is highly correlated with the visual content, while also managing the proportion of weakly correlated data, such as background music (BGM), to optimize the dataset's composition." This splits audio into two groups by strength of correlation with the picture: strongly correlated (diegetic, on-screen audio) samples are collected separately, while weakly correlated samples (such as BGM, non-diegetic) have their proportion controlled rather than being removed entirely. This is a more nuanced strategy than "blanket removal of BGM": retaining a certain proportion of weakly correlated audio lets the model learn to generate a score, but too high a proportion would break the audio-visual causal relationship. The specific proportion is not disclosed [Uncertain].
(3) A three-way split at the caption level: <W> marks speech (verbatim speech content), <I> marks non-speech acoustic events, plus the acoustic profiles field within Subjects — together forming three annotation tracks for speech / sound effects / timbre.
(4) Among the nine Level-1 domains, the two categories Home Sounds and Sound Effects correspond directly to non-speech audio categories, and every Level-3 visual label is bound to audio keywords — audio-category mixture is embedded into, and uniformly scheduled by, the overall domain-mixture system.
【NAVA — corpus organization driven by five audio-label categories】 A YAMNet audio classifier combined with an omni-modal tagger labels every clip with one of five categories: single-speaker speech / multi-speaker speech / ambient sound / music / singing. This five-way scheme is finer-grained than the common three-way "speech/effects/music" split, notably separating single-speaker from multi-speaker speech (corresponding to its multi-speaker timbre-control capability) and separating music from singing (singing carries both speech and music attributes, so making it its own class is reasonable). The audio mixture during training is reflected through the "audio-only : audio-visual" ratio schedule: Stage 1 is 3:1 (audio-side dominant, first training up audio-generation capability strongly), and Stage 2 reverses to 1:2 (audio-visual side dominant). The absolute proportion of each audio category is not disclosed [Uncertain].
【CCL】 Audio pretraining brings in two sound-effects/general-audio datasets, WavCaps and VGGSound, while the joint-training data (OpenHumanVid + interviews/short dramas/movies) is dominated by spoken dialogue — forming a structure of "general audio pretraining + speech-dominant joint training." Proportion not disclosed [Uncertain].
【Baton】 Among the data sources, AudioCaps and WavCaps supply general sound-effect/soundscape descriptions, and OpenHuman-Vid supplies human voice; the mixture ratio is not disclosed [Uncertain].
【OmniCustom】 Audio is almost entirely single-speaker speech (a vocal-timbre customization task), with no sound-effects/music dimension; audio is uniformly resampled to 16 kHz. This is the item with the most homogeneous audio category among the batch.
【StreamChar】 Character speech is the absolute majority; Emilia is a pure-speech dataset. No sound-effects/music category design [Uncertain].
【ITS-JAVG】 No training data; but its combination of verifiers covers two categories — audio semantics (ImageBind-TA, AVHScore) and synchrony (JavisScore). One of the evaluation backbones, MMDisCo, is based on VGGSound (predominantly sound effects), while the other, JavisDiT, covers a broader range of categories.

### [A Collection of Joint Audio-Video Generation Baselines](../models/JavisDiT_baselines.md) ⚠️

On this dimension the five works diverge sharply, forming a complete spectrum precisely from "sound-effects-focused" through "speech-focused" to "both together":
【MM-Diffusion / AV-DiT — pure ambient sound and music, zero speech】 Landscape consists entirely of natural ambient sounds (rain, thunder, wind, fire, water), and AIST++ consists entirely of music (60 dance tracks). Neither dataset contains human speech or dialogue, so these two early baselines have absolutely no lip-sync or TTS capability — this is the most fundamental capability gap between them and post-2025 models.
【JavisDiT — audio pretraining takes in all categories, video stage deliberately excludes speech】 The audio side has 780,000 clips drawn from 10 datasets; JavisDiT++ states explicitly that "no data filtering strategy is adopted, to ensure maximal text-to-audio generation capability, covering the three categories general sound, music, and speech" — i.e., the audio pretraining stage deliberately maintains full category coverage with zero filtering. But the strategy reverses by the audio-video SFT stage: "the FunASR detection tool is used to remove most videos containing human speech." This is a very clear category-mixture decision — the JavisDiT series deliberately gives up lip-sync/dialogue-generation capability, concentrating the model's capability on event-level alignment for ambient sound and sound effects, thereby sidestepping high-difficulty lip-shape modeling. In JavisBench's taxonomy, "Sound Type" is one of five evaluation dimensions, showing that the evaluation side explicitly stratifies by audio category.
【Harmony — a strict 1:1 speech/ambient-sound mixture】 The clearest mixture strategy of the batch: both Stage 1 (audio pretraining) and Stage 3 (cross-task joint training) use a "1:1 mix of human speech datasets and ambient-sound datasets." The speech side comprises 2 million clips (Emilia + OpenHumanVid + SpeakerVid, filtered for consistency); the ambient-sound side comprises 2 million clips from AudioCaps + Clotho + WavCaps + in-house collection. On the evaluation end, Harmony-Bench further sets aside "complex scenes where speech and ambient sound co-occur" as a separate 50-item tier — pointing directly at the hardest case of speech and ambient sound coexisting in real video. It is fair to say Harmony is the most systematic work in this collection in its thinking about audio-category distribution.
【UniAVGen — speech as the absolute majority】 Stage 1 performs pure audio pretraining on the English subset of Emilia (a TTS corpus); Stages 2 and 3 use real human audio-video data, with human voice/dialogue at the core throughout the pipeline. The evaluation metrics (SyncNet, Whisper WER, timbre consistency, emotion consistency) are likewise all centered on speech; ambient-sound and music capability are not reported [Uncertain].
【Silence handling】 None of the five works describes a silence-detection threshold or silence-proportion control [Uncertain].

### [Kling 3.0 / Kling Video 3.0 Omni](../models/Kling_30_Omni.md) ⚠️

[Uncertain] Kling 3.0 Omni has not published the mixture ratio for speech/effects/music/ambient/silence. On the capability side, all three categories are covered: dialogue speech (5 languages + accents), ambient sound, and foley effects; official materials do not emphasize music generation. The approach of Kling-Foley, from the same team, can serve as methodological corroboration: an audio classification model first sorts material into four categories — effects/music/speech/singing — after which an audio-understanding LLM is called separately for each category to generate descriptions; training material is retrieved and mixed by keyword against the three-level AudioSet ontology (roughly 1,000 third-level categories, merged into 1,919 tags across nine major sound scenes — traffic, human voice, animal sounds, etc.); and VAD is used to filter out samples with a silence ratio ≥0.2, i.e. an explicit upper bound is set on the "silence proportion." Whether Kling 3.0 Omni reuses this taxonomy and silence threshold is not disclosed.

### [LTX-2](../models/LTX-2.md) ⚠️

This is the most valuable, and also the only clearly stated, filtering criterion on LTX-2's data side, though again there is no proportion figure.
【Filtering strategy】 Rather than taking in the full volume of LTX-Video's videos wholesale, the approach is to "focus on video clips that contained significant and informative audio components" — i.e., using "audio information content" as the core threshold for constructing the AV training subset, removing clips that are silent, have too high a silence ratio, or whose audio track bears no informative relation to the picture. This "audio information-content filtering" is LTX-2's key data-side increment relative to pure-video models.
【Coverage categories】 The caption system and model capability cover four categories: dialogue/speech (including precise transcription), music, ambient sounds/background, and foley/sound effects. The paper emphasizes that LTX-2 "goes beyond just generating speech," instead producing a full soundscape that follows character, environment, style, and mood.
【Undisclosed】 The proportion of each of the four audio categories, the retention ratio for silent clips, the method for controlling category mixture, and the specific quantified criterion behind "significant and informative" (no SNR/loudness/silence-ratio thresholds, etc., are given). The model card also admits that "audio quality is lower when generating non-speech content," hinting that the proportion of speech-category data may be significantly higher than that of effects/music. [Uncertain]

### [LongCat-Video](../models/LongCat-Video.md) ⚠️

The base LongCat-Video has no audio modality; this dimension is not applicable.
The same-series LongCat-Video-Avatar 1.5 does involve audio, but its audio processing serves "as a driving condition" rather than a generation target, so there is no generation-side mixture design for speech/effects/music/ambient/silence. The audio-related data stratification that can be summarized is: (1) speech is the absolute majority (both the close-up-face and interview source categories are centered on clear speech); (2) singing/music is broken out as its own category (from music-video sources, used for singing lip motion and rhythmic movement); (3) silent (non-speaking) data is specially constructed as an independent subset — using a dual-model consistency mechanism, an initial judgment from Qwen3-Omni followed by a re-check from Qwen3-VL, retaining a clip only when both models judge the subject to not be speaking, used to teach the model the case of "audio present but the subject is not speaking"; (4) intervals of concurrent multi-speaker activity are explicitly excluded (excluding intervals with concurrent speaker activity). The proportion of each category is not disclosed. [Uncertain: the specific mixture figures for audio categories]

### [MOVA](../models/MOVA.md)

This is the most decisive part of MOVA's data design — the final training set **retains only speech clips**, an extreme choice of audio-category mixture:
【Preprocessing-stage classification】 Silero VAD splits the audio track into speech / non-speech intervals, and combined with scene-cut points from PySceneDetect this produces four types of fixed-length 8.05-second clips: single-scene speech, single-scene non-speech, multi-scene speech, and multi-scene non-speech.
【Key mixture decision】 "Ultimately, only speech segments are selected for training, accounting for 69.47% of all preprocessed segments." That is, speech clips make up 69.47% of all preprocessed clips, and only this portion is ultimately used to train the joint model. Mapped onto the total-duration retention rate (Table 1): original 100% → Stage 1 (speech + non-speech) 84.57% → Stage 1 (speech only) 58.75%. In other words, the single rule of "speech only" alone cuts about 26 percentage points of data.
【Audio-type classifier】 The EAT self-supervised audio Transformer classification model labels the audio, constructing speech / non-speech subsets and routing them by target capability (lip sync vs. general foley/ambient modeling). The construction condition for the speech subset is that both the EAT-contained-Speech and EAT-contained-Singing labels are judged True (or the model's positive-class confidence is satisfied).
【Category mixture in audio-tower pretraining】 Unlike joint training, the 1.3B audio tower's pretraining deliberately covers three major categories: general sound effects (WavCaps + VGGSound), music (JamendoMaxCaps), and speech (in-house TTS). This is a two-stage division of labor: "effects/music capability is injected during audio-tower pretraining, and speech-lip-sync capability is reinforced during joint training."
【Cost】 The paper's Limitations section explicitly acknowledges that, because the audio tower is only 1.3B parameters and joint training is speech-skewed, the model's performance degrades on singing, complex timbral texture, and music/instrumental content.
【Undisclosed】 The fine-grained proportions within the non-speech clips (effects / music / ambient / silence) are not given.

### [Merged entry: ① Mochi 1](../models/Mochi_MAGI_Motif.md)

Not applicable. All three models are purely visual video generation models; their training data does not retain audio tracks, and the technical reports/blog posts make no mention anywhere of classification or mixture control for speech, sound-effects foley, music, ambient sound, or silence.

### [Movie Gen](../models/Movie_Gen.md)

Movie Gen Audio establishes a two-dimensional classification system of "audio type × diegetic attribute" (Table 23), which is the core innovation of its data engineering.
The first axis is audio type: voice (speech + singing), non-vocal music (pure instrumental music), and general sound (general sound effects), automatically labeled by an AED (audio event detection) model based on the 527-class AudioSet ontology, where one sample can hit multiple classes at once (AudioSet's speech/singing subclasses → voice; music subclass → music; everything else → sound).
The second axis is diegetic (on-screen, causally related to the picture — e.g. someone speaking on camera, a live band, waves, an off-frame bird call) / non-diegetic (off-screen — e.g. documentary narration, a background score, a canned laugh track, a riser). The determination uses the cosine similarity between the audio and video embeddings from CAVTP (a contrastive audio-video-text pretrained model) — because this model is trained predominantly on diegetic data, the audio and video embeddings of on-screen audio are closer together.
Bucketing thresholds (Table 24, determined by manual inspection): when AED is sound / voice / voice+sound, CAVTP>0.2 is judged diegetic; when AED is music / music+sound / voice+music+sound, CAVTP>0.3 is judged diegetic; when AED is music and CAVTP<0.1, it is judged non-diegetic; when AED is sound+music and 0.1<CAVTP<0.25, it is judged non-diegetic; when AED is sound+voice+music and 0.1<CAVTP<0.25, it is judged mixed.
Mixture and trade-offs: all videos where silence is the dominant class are discarded first; pretraining uses only data that is diegetic or a diegetic/non-diegetic mix, with a small additional amount of non-diegetic background music added; general sound is explicitly prioritized (because low-level physical regularities are the hardest to learn and errors are the easiest to notice), and in the actual distribution the Sound class is overwhelmingly dominant at O(100)M samples / O(1,000)K hours, while each of the other four classes is only O(10)M / O(100)K — i.e. sound effects outnumber music/voice-containing classes by roughly an order of magnitude (about 10:1).
Deliberate exclusions: diegetic speech is not generated (hard without a script, and the generated video has artifacts) nor is non-diegetic speech (can be substituted with TTS); within the fine-tuning cinematic split, clips containing human voice are excluded wholesale.
Fine-tuning-stage mixture: cinematic-grade audio-video (Cin-AV) and high-quality audio-only data (HQ-A) are mixed at a ratio of 10 batches : 1 batch.

### [NVIDIA NeMo Curator](../models/NeMo_Curator.md)

None. This is the most critical missing dimension of this toolchain in the context of audio-video generation data.
The video pipeline does not process audio tracks at all — there is no classification stage for speech/sound-effects-foley/music/ambient/silence, no proportion statistics or mixture-control strategy for any audio category, and no built-in stage for extracting audio tracks from video. An audio pipeline does exist, but its data model is "a standalone audio file + transcript" (aimed at ASR/TTS speech data), covering only the speech category, and it is not linked to video. The audio-enhancement stage added in version 26.07 (tagging, SQUIM quality metrics, bandwidth estimation, punctuation preparation, optional second-pass ASR scoring) is likewise entirely centered on speech quality and does not involve foley/music/ambient classification.
Therefore, to build training data for joint audio-video generation with NeMo Curator, the audio-category-distribution layer would have to be built entirely from scratch.

### [OmniHuman Dataset + OHBench](../models/OmniHuman.md) ⚠️

On the audio side, a strategy of "separation rather than filtering" is adopted — this is the most valuable design choice this dataset makes on the audio dimension:
【Core approach: Demucs four-source separation】 Demucs performs four-source separation on every audio track (the standard Demucs output being vocals / drums / bass / other), after which "vocals are used as the target track, with the remaining tracks mixed together as the background track" — i.e. the final output is a two-track structure of "vocal track + background track." This differs from both UniTalking's "remove samples containing background sound" and UniVerse-1's "bucket by speech/non-speech": OmniHuman neither removes nor buckets, but instead decomposes mixed audio into two tracks that can be annotated and used separately, leaving the decision on mixture ratio to downstream users.
【The practical value of this design】 (1) It trains controllable generation: the model can learn conditional control over "with/without background music"; (2) it supports the video-dubbing task, which needs a clean vocal track for supervision; (3) it avoids the data waste of "discarding an entire high-quality video just because it has background music" — given the extremely high proportion of YouTube content that carries BGM, adopting a UniTalking-style removal strategy would cause the retention rate to collapse catastrophically.
【Music-attribute annotation】 Music in the background is annotated separately with three attributes: type, mood/atmosphere, and relative volume. The "relative volume" attribute is especially practical — it quantifies the energy ratio between music and voice, allowing downstream users to filter "music-dominant" versus "voice-dominant" samples as needed. This is a rare instance of fine-grained music annotation among the samples surveyed.
【Background-sound labels】 The video-level annotation includes two fields, background sound labels and music attributes, both listed at the global level.
【Speech】 3D-Speaker performs speaker diarization, FunASR-Nano performs transcription, and emotion and timestamps are annotated (see dialogue_transcription_attributes for details).
【Undisclosed mixture】 The duration proportion of speech/music/effects/ambient/silence within the 1,800 hours is not given at all; the proportion of samples with vs. without BGM is not given; the handling of purely silent segments is only vaguely covered under "low-quality removal." [Uncertain]
【Audio normalization】 Unified to a 44.1 kHz sample rate.

### [Open-Sora Series](../models/Open-Sora.md)

Not applicable. Neither Open-Sora nor the entire Open-Sora Plan series generates audio; the training-data pipeline does not process audio tracks, and there is no classification, mixture, or statistics for speech/effects/music/ambient.

### [Ovi](../models/Ovi.md) ⚠️

Audio-category mixture is the core point of tension in Ovi's data design; what the paper gives is a "staged bias" rather than a static ratio.
【Pretraining stage】 "predominantly human speech," drawn from internal corpora, emphasizing linguistic diversity, prosody, and timbral variation. This stage contains almost no sound effects; the goal is to first firmly establish TTS/speaker-modeling capability.
【Audio fine-tuning stage】 An explicit shift toward sound effects: "we emphasize modeling sound effects," bringing in VGGSound / AudioSet / WavCaps to fill out SFX; at the same time, "to preserve TTS capability and better align with downstream objectives," audio tracks extracted from internal audio-video corpora are additionally mixed in. This forms a two-stage mixture schedule of "speech-heavy → effects supplementation + speech kept alive." The mixture ratio between the three public sound-effect sets and the internal speech tracks is not disclosed [Uncertain].
【Audio-video fusion stage】 The audio side comes entirely from the native audio tracks of paired videos; the natural proportions of speech/effects/music/ambient are determined by the internal corpus, with no description of any manual rebalancing [Uncertain].
【Category routing at the annotation level】 The captioning stage handles clips via a binary "has speech / no speech" split: for clips with speech, the audio description emphasizes the speaker's acoustic attributes; for non-speech clips, the audio description instead covers sound effects, background audio, and musical elements — this is in effect a three-category (speech/effects/music) annotation taxonomy.
【Silence handling】 Clips near-silent are removed via an average-volume threshold of ≥ −60 dB (see audio_quality_filtering).
【Justification for the necessity of a unified audio model】 In the results section, the paper emphasizes that real-world videos often contain complex sound effects and coherent speech at the same time, which specialized models (pure T2A or pure TTS) cannot support — hence a unified T2A+TTS audio tower must be trained. This is the core motivation behind its mixed-category audio-training strategy.
【Music】 BGM-generation capability is listed as a feature, but it is not stated whether music data is introduced separately [Uncertain].

### [Script-a-Video](../models/Script-a-Video.md) ⚠️

Audio category has a clearly defined three-way classification at the MTSS schema level, but the proportion of each category within the training data is entirely undisclosed:
【The three audio-event types defined by the schema】 The Event stream strictly classifies all audio events into three types:
1) dialogue — carrying a speaker (bound to a Reference ID) and a line (verbatim dialogue text);
2) sfx (sound effect) — required to be produced by a subject visible in the frame ("sound effects must be generated by a visible subject");
3) music.
【A fourth category: global audio】 Sounds that do not constitute an independent event (ambient background noise, background music/atmosphere) do not enter the Event stream; instead they are placed into the "global_audio" field of the Global stream. Original text from the paper: "Irrelevant background noise is filtered into the global audio metadata" — i.e. irrelevant background noise is "filtered into" global audio metadata rather than discarded. This is a four-tier routing design: sound effects with a visible source / dialogue with a speaker / music / a fallback global atmosphere track.
【Filtering principle: strict audio-visual coupling】 The Event stream's admission criterion is the "strict audio-visual coupling principle" — only audio events with a direct visual counterpart or thematic relevance are extracted. This principle effectively completes the removal/demotion of "off-screen sound sources" already at the annotation stage: sounds with no visual counterpart never become an independent event, thereby guaranteeing that every entry in the Event stream is audio with a visual cue that the generative model can learn from. This is the design of greatest methodological value on the audio side of this work.
【Handling of concurrent sound sources】 Multiple sound sources occurring simultaneously are "factorized into parallel event entries" rather than merged into a single mixed description. This ensures that in multi-source scenes, every sound has its own independent time_range, speaker, and description, and can be edited and controlled independently.
【Proportion figures】 The paper discloses none of the following: what proportion each of the three event types makes up within the 500K dataset, how many events on average each clip contains, or what proportion of events are of type dialogue. [Uncertain]
【Sampling weight】 No mechanism for adjusting training sampling weights by audio category is mentioned.

### [Seedance 2.0 and Seedance 1.5 pro](../models/Seedance_20_Seedance_15_pro.md) ⚠️

[Uncertain] The audio-category mixture on the training side is not disclosed. However, the evaluation-annotation taxonomy for Seedance 1.5 pro/2.0 is highly detailed and can be viewed as a mirror of its data-category system: the main audio-label classes of SeedVideoBench 1.5 are — Human Voice Types (speech, singing, non-verbal vocalizations such as laughter, plus fine-grained sub-dimensions); Human Voice Attributes (timbre, accent, emotional tone); and Non-Speech Audio (covering sound effects and music, classified by sound source such as animal/mechanical tools, acoustic attributes, music genre, technical parameters). SeedVideoBench 2.0 expands the fine-grained audio categories to 17: Chinese dialects/accents, Chinese multi-person dialogue, Chinese variety-show voice, Chinese opera, English, minority languages, singing/rap, spatial scene sound, off-screen voice, non-verbal voice, voice + action interaction sound, object interaction sound, animal calls, ambient/background sound, special sound effects (including ASMR), instruments and audio, and dual-channel audio. Seedance 2.0 explicitly supports multi-track parallel output across three audio-track types: background music / ambient sound effects / character dubbing. The proportion of silent samples and the specific speech:effects:music mixture are both undisclosed.

### [SkyReels Series](../models/SkyReels.md) ⚠️

Only SkyReels-V4 is relevant here; the strategy is clear but there are no proportion figures.
【Classification system】 Audio is explicitly split into four categories: sound effects (SFX), music, speech, and singing — "singing" being broken out as an independent category is a refinement relative to most audio-video models (which typically only distinguish speech/music/effects), directly serving lip-sync for short-drama and music scenarios.
【Classification tool】 The Qwen3-Omni omni-modal LLM is used uniformly for category determination on audio, and it also uniformly generates the audio captions.
【Processing differences driven by category】 (1) When padding out duration, clips are "concatenated by category" — only same-category audio is allowed to be concatenated, avoiding cross-category mixing; (2) the speech and singing categories additionally go through Whisper transcription, while effects and music are not transcribed; (3) in the caption, four special tokens — <sfx>, <dialogue>, <singing>, <bgm> — each carry one category, in one-to-one correspondence with the four-way classification.
【Undisclosed】 The sample count and proportion of each of the four categories, the retention ratio for silent samples, and any target mixture value for category proportions. It is known that the audio backbone's pretraining uses hundreds of thousands of hours of data "primarily speech data," indicating that the speech category accounts for the overwhelming majority, with effects/music/singing being minority categories. [Uncertain]

### [Sora 2](../models/Sora_2.md) ⚠️

Completely undisclosed. As a native audio-video model, Sora 2's capability explicitly covers four audio categories — dialogue, sound effects/foley (sound effects tied to on-screen actions), background music (matched to scene tone), and ambient/atmospheric sound (context-aware ambient sounds / background soundscapes) — and it is claimed that sound volume and spatial positioning change with an object's and the camera's distance. But OpenAI gives no information at all on what proportion of the training data each of these four audio categories occupies, how audio-track-less/silent clips are handled and retained, or whether explicit mixture control is applied across speech-effects-music. This is one of the dimensions with the largest information gap in this survey: the model clearly has this capability, but the data-side construction method is disclosed not at all. [Uncertain]

### [SpeakerVid-5M](../models/SpeakerVid-5M.md) ⚠️

SpeakerVid-5M is a purely speech-oriented (speech-only) dataset, and its audio-category mixture is implicit and extreme:
【Design orientation】 All branches are built around humans speaking and listening; the content of the audio tracks is overwhelmingly dominated by spoken human dialogue, and no separate subset or mixture is set aside for sound effects (foley), music, or ambient sound.
【Implicit non-speech-removal mechanism】 The audio-quality filter has a rule that removes clips with a no-speech probability > 0.8 (from Whisper's no-speech probability), which directly excludes pure sound-effect, pure music, pure ambient, and silent clips from the dataset; samples with an ASR confidence below −1.5 are likewise removed. As a result, the final corpus is, in terms of audio category, close to 100% speech-dominant clips.
【The audio peculiarity of the listening branch】 The visual subject of the listening branch is the party who is not speaking, but its paired audio track is still the other party's speech — so on the audio side it is still speech, just with no correspondence to the on-screen subject's lip motion. This is the only "audio-visual character mismatch" design within the dataset, and it is deliberate (used to train the listener's reaction generation).
【Processing not done】 The paper describes none of: background-music separation (BGM separation), speech enhancement, SNR estimation, or an effects/music classifier (such as EAT) — any fine-grained audio-category processing — nor does it give proportion statistics for speech/non-speech/silence. Compared with MOVA, which explicitly states "speech clips account for 69.47% of preprocessed clips," SpeakerVid-5M is an information blank on this dimension.
【Downstream impact】 Precisely because of its pure-speech nature, downstream models such as MOVA position it as a dedicated data source for lip-sync capability, while general sound-effects and music capability must be obtained from other datasets such as VGGSound and WavCaps. [Uncertain]

### [Step-Video-T2V](../models/Step-Video-T2V.md) ⚠️

Not applicable. Step-Video-T2V does not generate audio; its training-data pipeline is not involved at all in audio-track extraction, classification, or mixture, and the report contains no proportion-control content whatsoever for speech/effects/music/ambient/silence. For an audio-data methodology within the StepFun ecosystem, one should look instead to the separate Step-Audio series (a speech-interaction model, whose data construction covers dimensions such as emotion, dialect, language, and singing), though it shares no data with the video model. [Uncertain]

### [UniTalking](../models/UniTalking.md) ⚠️

In UniTalking, audio-category mixture is achieved by "filtering down to a single category" rather than "configuring by proportion" — this is the most fundamental data difference between it and general-purpose AV models:
【Target category: speech only】 The audio-filtering step explicitly removes three types of samples: muted, lacking speech, and low SNR. PANNs (a large-scale pretrained audio neural network, a 527-class AudioSet event classifier) and the SentenceASD model perform "speech event detection." As a result, all 2.3 million final samples in theory contain valid human speech, and sound effects and music are not retained as independent categories.
【Handling of non-speech components: removal rather than separation】 The cross-modal filtering stage further "removes videos containing audio whose sound source is not on-screen (purely diegetic audio)" [sic], determined using LightASD (a lightweight active-speaker-detection model). Note that the paper's original wording is "diegetic" (in-narrative sound), but in context it is actually used to mean removing samples where "the sound does not come from the person in frame" — i.e. clips dominated by voiceover, narration, or score are eliminated. This stands in sharp contrast to UniVerse-1's "no removal of off-screen sound sources," and is the most targeted filter UniTalking applies on the audio side.
【Background music】 No source separation is performed, nor is any separate threshold set. But two pieces of indirect evidence indicate background music remains partly present in the training data: first, the caption includes an audio-description field capable of describing the background acoustic environment; second, the experiments section demonstrates the ability to control the generated result via the text prompt "without any background music" — this kind of controllability could only be learned if the training data contains both "with background music" and "without background music" samples, and the caption faithfully annotates this difference. This is a "do not filter, instead annotate for controllability" approach (of the same lineage as MOVA's approach of marking subtitles with "This video has no subtitles.").
【Non-speech sound effects】 In the example shown in Figure 4, the prompt "a short laugh" appears and is correctly generated, showing that paralinguistic sound effects such as laughter are retained in the data and covered by the caption.
【Undisclosed mixture】 The proportion of speech within total duration, the proportion of background-music samples, the proportion of silent segments, the distribution of speaker counts — all of these are blank. None of the filters have numeric thresholds given. [Uncertain]

### [UniVerse-1](../models/UniVerse-1.md)

Audio-category mixture is the key dimension on which UniVerse-1 and MOVA part ways — UniVerse-1 deliberately preserves the dominance of non-speech data:
【Three-part mixture (by hour count)】
- Speech (speech-centric, verified through a triple check of Whisper speech detection + RetinaFace face detection + SyncNet conf>2.0): 1,187 hours, 15.4%;
- General audio-video (general, i.e. clips Whisper judges to contain no speech, covering ambient sound, sound effects, instrumental music, etc.): 3,074 hours, 40.0%;
- VGGSound + AudioSet (dedicated reinforcement for sound effects/event sounds): 3,422 hours, 44.5%.
That is, non-speech data together account for 84.6%, with speech accounting for only 15.4%.
【Classification mechanism】 Whisper ASR serves as a binary-classification gate: clips from which valid speech content can be detected enter the speech subset and undergo further face and lip-sync verification; clips from which no speech can be detected are not discarded but downgraded and retained as "general audio-video data." This is a "nothing wasted" design — the same filtering rule accomplishes both routing and retention at once, rather than outright elimination as in MOVA.
【Silence handling】 Audio-activity detection is performed via three signal-level metrics — volume, energy, and zero-crossing rate — removing silent clips. The training set therefore contains no silent samples.
【Music category】 Music capability is mainly carried by the backbone Ace-step (itself a music-generation model); on the training-data side, this is reinforced with music-variety-show and classical-performance material, but no music-hour figure is given separately.
【Undisclosed】 The fine-grained proportions within the general audio-video subset (ambient/effects/instrumental/BGM) are not given; nor is it stated whether the sampling weight of each category is adjusted during training.
【Comparison with Verse-Bench】 The evaluation side gives explicit proportions for nine audio categories: natural environment 36.1%, music and instruments 19.3%, daily life 10.2%, human voice 7.6%, traffic 4.5%, industrial and urban 3.9%, weapons and explosions 2.5%, special-effect sounds 2.3%, uncategorized 15.8%. The evaluation set is likewise dominated by natural ambient sound, with human voice at only 7.6% — consistent with the orientation of the training mixture.

### [Unison](../models/Unison.md) ⚠️

Audio category is the one distributional dimension in Unison's entire data design that is treated seriously — because the whole architecture is built around the binary split of "speech vs. sound effects." But even so, the paper still gives no proportion figures at all.
【Architecture-level mandatory binary classification: speech / sound effects】 All audio (whether from audio-video data or audio-only data) is invariably separated by Mel-RoFormer into two streams, speech and sound-effect (SFX), each encoded as a ground-truth latent and then each supervised by its own independent flow-matching loss. Original text from the paper: "we leverage Mel-Roformer to disentangle mixed audio into high-fidelity speech and sound-effect (SFX) components." This is not "classify then process separately," but rather "every sample is split into two streams and processed simultaneously" — even if a given sample has no speech, its speech stream still exists (with empty/near-silent content). This is a completely different processing paradigm from UniVerse-1 (Whisper binary classification as a routing gate, with speech/non-speech taking different paths) and MOVA (an EAT classifier making a speech/non-speech determination): Unison does not route, but runs parallel dual tracks.
【The four-way division of labor for the audio-only corpus (category membership can be determined by data source, but no proportion figures are given)】
- Speech: internal speech data;
- Sound effects: YouTube-8M, AudioSet, WavCaps;
- Music: VidMuse;
- Singing: the YuE collection.
The paper explicitly breaks out singing as an independent category, placed alongside speech and music — this is relatively rare among peer works, and directly corresponds to the named failure mode in its qualitative results, that "Universe-1 and UniAVGen struggle to distinguish singing from ordinary speech." Unison introduces the YuE singing corpus specifically to address this problem.
【Silence handling】 No mention of silence detection, a silence-ratio threshold, or removal of audio-track-less clips. [Uncertain]
【Proportion-control strategy】 Entirely undisclosed. There is no statement of the hour count, clip count, or sampling weight for each of the four audio categories, nor whether they are reweighted during training. How the total of 50 million clips / 130,000 hours is distributed across the four categories — speech/effects/music/singing — is the single most critical information gap in this field. [Uncertain]
【A runtime dynamic-balancing mechanism (not on the data side, but functionally equivalent)】 Worth recording: Unison turns "controlling the speech-to-effects ratio" from a data-mixture problem into a learnable gating problem at inference time — the SCG predicts two gating coefficients (sigmoid-constrained) from the global semantic vectors of the caption and transcription, suppressing the sound-effect stream from encroaching on the speech stream in narration-dominant scenes, and amplifying cross-stream influence in complex soundscape scenes. The analysis in Fig. 8 shows this gating exhibits dynamism along three dimensions: layer depth (polarization intensifies in deeper layers), timestep (gating divergence intensifies as denoising progresses), and instance (adapting across semantic categories). This is a technical route where "the mixture ratio does not rely on data, but on model self-adaptation" — an interesting counterpoint to traditional data-mixture control.

### [Veo 3 / Veo 3.1](../models/Veo_3_Veo_31.md) ⚠️

[Uncertain] Audio-category mixture and control strategy are completely undisclosed — one of the biggest gaps in Veo 3's data disclosure. Clues that can be inferred backward: (1) the model's capability covers four major categories — dialogue, sound effects/foley, ambient noise, and background music — indicating the paired audio-video training data has substantial coverage across all four; (2) official materials emphasize that Veo 3 solves the "silent film" problem of earlier video models, meaning samples with no audio track or with an audio track unrelated to the picture must have been explicitly excluded during training, though the exclusion criteria and the proportion of silent samples are not disclosed; (3) the technical report, in its deepfake assessment, notes that deepfakes generated by Veo 3 are "much less controllable — particularly with respect to speech," hinting that speech data has not received fine-grained annotation or conditioning along the speaker-identity/timbre dimension; (4) the model card makes no mention of any audio-source separation (such as separating BGM from voice), SNR threshold, or silence-ratio threshold. There is no public basis whatsoever for the specific mixture figures of speech/effects/music/ambient.

### [Vidu S1](../models/Vidu_S1.md) ⚠️

No quantitative proportions are given [Uncertain], but there is a clear audio-category processing and filtering strategy, oriented toward "purifying speech, removing music":
(1) The speech component is first extracted from the raw audio;
(2) VAD (voice activity detection) + ASD (active speaker detection) are used to annotate the timestamp and associated speaker of each speech segment;
(3) Each speech segment is classified into one of three categories — onscreen (the speaker is the person in frame), offscreen (voiceover, the speaker is not in frame), or overlap (multiple overlapping voices); clips containing an overlap segment are removed in their entirety;
(4) To address the instability of the diarization model in singing and strong-background-music scenes (which is prone to misclassifying voice into the music stem, producing synthetic timbre and distortion artifacts), a heuristic rule is introduced: if the speaker is vocalizing but the speech-energy proportion is too low, the segment is discarded — the practical effect being systematic removal of singing- and music-dominant clips.
(5) At the caption level, sound effects and background music fields are still annotated, showing that these two types of audio information are retained as describable attributes, though speech remains the overwhelming majority of the training distribution.

### [Wan 2.5 / 2.6 / 2.7](../models/Wan.md) ⚠️

Wan 2.1's V2A subset provides the only clearly stated audio-category filtering criterion in the Wan lineage, while the strategy from 2.5 onward is completely reversed yet undisclosed, forming the largest information gap.
【Wan 2.1 V2A (2025.02)】 The training data comes from the video-generation dataset via "strict filtering," with the rule being: systematically remove (1) videos with no audio track and (2) videos containing speech or vocal music, ultimately yielding a refined subset of O(1) thousand hours. That is: retained categories = ambient sound + background music; excluded categories = speech, human voice. The report states plainly in its limitations that the model cannot generate laughter, crying, speaking, or other human vocalizations, "mainly due to the deliberate exclusion of speech-related data during our data preparation process," and explicitly lists "adding speech-generation capability in the future" as follow-on work.
【From Wan 2.5 onward (2025.09)】 "Audio-visual sync/lip sync" becomes a core selling point, and 2.6 further supports the simultaneous matching of the three categories "voice, sound effects, BGM" as well as audio-driven lip motion and performance. This means the speech data explicitly excluded in the 2.1 era has been reintroduced at scale, and a new dialogue/speaker-related data pipeline must have been introduced. But the proportion of each of the four audio categories (speech/effects-foley/music/ambient), the retention strategy for silent clips, and the mixture-control method are all undisclosed.
【Inference】 The shift from 2.1's "exclude speech" to 2.5's "lip sync as the headline feature" is the biggest strategic pivot on the data side of the Wan lineage, and it happens to line up with the technical build-up window of Wan2.2-S2V (2025.08, dedicated to speaker-video processing) — the human-centric + active-speaker-detection pipeline of S2V is quite likely the foundation on which the 2.5 speech data was built. [Uncertain]

### [A Collection of Audio-Video Generation Evaluation Benchmarks](../models/av_benchmarks.md) ⚠️

On this dimension unique to AV, the five benchmarks provide three complementary audio-category taxonomies:

【AV-SyncBench's three-way split】 Voice / Music / Sound is the simplest, most operable top-level audio classification. Its mapping to ten scenarios is: Voice ← single-person speaking, dialogue, group vocalization, singing; Music ← solo instrument, ensemble, singing (cross-category); Sound ← actions, animal sounds, object sounds, ambient sounds. The semantic-challenge task also routes along this split: voice goes through timbre substitution (OpenVoice V2), and instruments go through timbre transfer (a pretrained DDSP) — showing that the semantic-perturbation mechanisms for the two categories are fundamentally different, and semantic-alignment filtering of training data should likewise be routed separately. In terms of sample count, the temporal challenge (37,569) far exceeds the semantic challenge (821), reflecting that semantically mismatched samples are harder to construct from real data.

【VABench's content-driven split】 Animal sounds, human voice (verbal/non-verbal), music, ambient sound (natural/urban/indoor), synchronous physical sound, complex soundscapes (including off-screen sound sources), and virtual sound effects. This is finer-grained than the three-way split, and explicitly introduces the category "off-screen/invisible sound source" — such samples are typically wrongly killed by on-screen filters in training data, but the evaluation system acknowledges their existence, suggesting the training side should retain a certain proportion of off-screen-sound samples to support complex-soundscape generation.

【PhyAVBench's physics-oriented Foley split】 Classification by sound-production mechanism rather than content topic (solid impact / mechanical structure / fluid aerodynamics / propagation environment / observer position / temporal causality) — essentially an in-depth expansion of the Foley sound-effect category.

【AVBench】 Speech is the overwhelming focus (three of its 10 dimensions — Speech Content Accuracy, Speech Realism, and Lip Sync — target speech directly), corresponding to the OpenHumanVid person-video data source; music and sound-effect categories are essentially not covered.

None of the five benchmarks give an exact percentage mixture for their audio categories [Uncertain], but the union of their categories (speech / music / physical sound effects (Foley) / ambient soundscape / animal sound / off-screen sound / silence and virtual sound) can directly serve as row labels for a training-data audio-domain mixture table.

### [The Video Captioning Model Ecosystem](../models/caption_models.md) ⚠️

This is the most differentiating dimension in the 2025Q4–2026 audio-video captioner ecosystem, and AVSCapBench is the first to provide complete, quantified, side-by-side evidence:
【Explicit modeling of three audio categories】 AVSCap's Acoustic Completeness criterion explicitly requires captions to cover all three categories — Speech / SFX / Music — at once. This is fully isomorphic to Foley-Omni's three-field schema and to Bandit (cinematic audio source separation, which naturally separates into speech / effects / music). MOVA adopts a dual-model division of labor: Qwen3-Omni-Instruct handles speech (ASR), and Qwen3-Omni-Captioner handles non-speech sound and music.
【AVSCapBench event-recall (%) side-by-side comparison table — the single most valuable data point in this ecosystem】
Model / Visual / Speech / Music / SFX / Synergy / Overall:
· AVSCap-7B: 59.33 / 69.45 / 40.36 / 30.82 / 57.70 / 60.44
· AVoCaDO-7B(9B): 50.59 / 70.42 / 38.71 / 19.25 / 29.13 / 49.31
· video-SALMONN-2-7B: 39.05 / 46.76 / 13.76 / 8.71 / 12.43 / 32.02
· Qwen3-Omni-30B: 41.85 / 49.08 / 9.34 / 8.68 / 16.19 / 35.29
· Qwen2.5-Omni-7B (bare backbone): 34.78 / 13.92 / 4.02 / 7.22 / 7.00 / 21.53
· Gemini-3-Pro: 60.43 / 79.81 / 39.52 / 27.77 / 48.88 / 60.97
· Gemini-3-Flash: 58.14 / 79.78 / 39.46 / 32.34 / 48.94 / 60.54
【Conclusive observations】 (1) Speech is the strongest audio dimension for every model (40–80 points), music is second (4–40 points), and SFX is weakest (7–32 points) — the capability gap between audio categories is as large as 8x, which directly limits the training-data quality of Foley/ambient-generation models; (2) the 7B open-source model (AVSCap, 60.44) has already matched Gemini-3-Pro (60.97) overall, and even surpasses it on Synergy (57.70 vs. 48.88), yet still trails by 10 points on Speech; (3) the bare Qwen2.5-Omni scores only 13.92 on Speech, proving that an omni backbone without dedicated caption training is completely unusable.
【Handling of audio-category mixture on the generation side】 Movie Gen retains both signals at once — AED's music posterior probability and the music caption (because the music-captioning model is prone to hallucination when there is no music) — and empirically this redundant combination gives the best controllability. This is direct engineering evidence that "audio-category distribution" shapes captioner design. LTX-2's data-filtering criterion is "contains significant and informative audio components," so as to keep the distribution of visual and auditory content balanced (specific proportions are not disclosed [Uncertain]).

### [A Collection of Geometric / Structured-Annotation Datasets](../models/geometric_datasets.md)

Not applicable. None of the four datasets involves audio-modality annotation, and there is no category mixture for speech/effects/music/ambient. Action100M is the only one that touches audio indirectly — via narration ASR transcripts inherited from HowTo100M (72% of videos have usable ASR, covering 10.6 years of content) — but this only participates as a weak textual-supervision cue, with no distribution statistics or mixture control performed by audio category.

### [Post-Training Data Strategies for Video Generation](../models/post_training_data.md) ⚠️

None of the anchor paper's four reward models (video aesthetics, image aesthetics, motion quality, text-video alignment) touches audio, and there is no statement whatsoever about the audio-category mixture of SFT data — a notable gap of this framework relative to the 2026 AV mainstream [Uncertain].
【Side by side, only three examples give an audio-side post-training mixture】
① Movie Gen's audio SFT set is the most finely disclosed on this topic: split into two — the cinematic split (professionally produced, containing on-screen audio and off-screen ambient/thematic music, explicitly excluding clips with human voice, automatically screened by a cinematic-feel classifier + AED sound-event detection and then manually selected), O(100)K samples / O(1)K hours; and the high-quality audio split (video-less high-quality music, O(10)K hours, + sound effects, O(10)K hours), O(1,000)K samples / O(10)K hours. "Excluding clips containing human voice" is a key decision: Movie Gen Audio is positioned as an effects+music generator rather than a dialogue generator, so speech-category samples are actively removed at the SFT stage.
② Foley-Omni's Stage 3 (full-soundtrack V2ST fine-tuning) does light 2-epoch fine-tuning on a minimally sized 216-hour dataset, paired with 100 hours of per-domain replay to prevent forgetting, with the admission criterion being "contains multiple audio components";
③ JavisDiT++'s AV-DPO uses AudioBox-Aesthetics to assess audio quality separately and Synchformer to assess temporal synchronization separately — i.e., at the preference-ranking level audio is treated as an independent modality — but the proportion of speech/effects/music is not disclosed.
【Seedance 1.5 pro】 It states its multi-dimensional reward model covers "audio fidelity," but whether speech/effects/music are scored along separate dimensions is not disclosed [Uncertain]. Kling 3.0 Omni's audio-dimension preference annotation (whether lip-sync degree and timbre consistency are used as independent scoring items) is likewise undisclosed [Uncertain].

### [A Merged Survey of Mainstream Video Pretraining Datasets: Panda-70M, InternVid, Koala…](../models/pretraining_datasets.md)

Not applicable. All seven datasets are purely visual+text datasets that neither generate nor process audio, and there is no classification, mixture, or statistics for speech/effects/music/ambient. Only the video files of Panda-70M and UltraVideo passively retain the original audio track (the former via download_audio:True, the latter declaring that native audio is preserved), but neither has any audio-category annotation or mixture control; the re-splitting code of Koala-36M, by contrast, discards the audio track outright.
## Narrative Structure Distribution (Single-shot vs. Multi-shot, Average Clip Duration, Shot-count Distribution, Native Audio Track)

`narrative_structure_distribution` · Detail level: brief

### [Allegro](../models/Allegro.md)

All clips are single-shot, with no multi-shot narrative structure: the original long videos are split by PySceneDetect into "single-scene clips" before entering the training set, and the paper explicitly states that the training material is single-shot camera footage.
Average clip duration falls in the 2–16 second range (6–16 seconds during the fine-tuning stage); training uniformly samples 40 or 88 frames (88 frames @15FPS ≈ 6 seconds).
No native audio track (audio is discarded).
Shot-count distribution is not applicable (always 1); the paper does not construct any multi-shot/storyboard-type data.

### [Apollo](../models/Apollo.md) ⚠️

[Single-shot vs. multi-shot] Explicitly adopts a pure single-shot strategy: the paper states verbatim, "We then apply scene splitting to ensure each sample contains only one scene." That is, all training data are single-shot clips, containing no cross-shot transition samples. This contrasts with MOVA's deliberate construction of a "single-scene/multi-scene" 2×2 data dimension, and also means Apollo does not model shot-transition capability on the data side.
[Native audio track] Training data must carry native, synchronized audio tracks — both the audio filtering and audio-visual consistency detection stages throughout the entire pipeline operate on the original audio track; samples with no audio track or abnormal audio tracks are removed during filtering.
[Average clip duration / shot-count distribution] Not disclosed. [Uncertain] (the duration figures are missing, but the single-shot conclusion is certain)

### [CineDance / CineDance-1M](../models/CineDance.md) ⚠️

Narrative structure is the foundation of this dataset and the part the paper devotes the most ink to:
[Multi-shot] Average of 24.2 consecutive shots/sequence, with all samples being multi-shot — a generational gap compared to datasets dominated by single shots such as LVD-2M (1.86 shots) and SpeakerVid-5M (1.27 shots).
[Average sequence duration] 92.8 seconds; compared with MiraData's 72.1s / 7.15 shots and LVD-2M's 20.2s / 1.86 shots.
[Native audio track] Fully retains the original synchronized audio track — the only dataset in the comparison table to simultaneously possess 1080p + native audio track + shot-level dual-modality dense annotation.
[Definition of narrative sequence] "A continuous flow of diegetic time and causality" — i.e., a combination of shots that is continuous in dramatic time and causal chain, and in which the state of characters and environment persists, allowing spatial jumps but not narrative breaks.
[Four film-theory parsing rules] ① Multi-angle shots: only the camera angle changes while the event remains unified; ② Cross-cutting: rapid alternation between different spaces bound together by a unified causal tension; ③ Causal action / ellipsis: there is a spatiotemporal jump, but the subsequent event can directly and interpretably follow from the previous event; ④ Montage: shots are mutually disjointed but unified by an overarching theme or emotional arc.
[Shot-level attributes] Each shot is annotated with five attribute dimensions: scale, angle, movement, narrative function, and duration category, plus shot transition type.
[Distribution details] Fig. 5 gives the joint distribution of duration and shot count, but the specific quantiles of shot count are not stated in text. [Uncertain]

### [CogVideoX](../models/CogVideoX.md) ⚠️

· All training clips are single-shot: the paper explicitly states "approximately 35M single-shot clips," averaging 6 seconds. Multi-shot narrative is outside the modeling scope; the model's output is a single continuous single-shot video.
· The mechanism guaranteeing single-shot is not a scene-splitting tool but a negative-label classifier: "Lack of Motion Connectivity" specifically identifies "motion discontinuity at transitions, commonly seen in manually spliced videos or videos cut from static images," and rejects clips containing shot switches/splices wholesale; the "Editing" label further removes material with transition effects.
· Shot-count distribution and average shot count statistics are not provided [Uncertain].
· Whether it contains native audio track: not applicable, the video side of training discards the audio track entirely.
· At the caption level, camera movements are described, but there is no structured multi-shot field.

### [Cosmos-Predict2.5](../models/Cosmos-Predict25.md)

Training clips are entirely single-shot: the first stage of the pipeline uses a high-precision shot-boundary detection model to cut long videos into shots, and explicitly states "ensuring that raw shot transitions are excluded"; a subsequent "semantic artifacts filter" (similar to VTSS) further removes structural anomalies such as poor transitions and video-in-video. Consequently the dataset contains no multi-shot narrative samples, and the concept of a shot-count distribution does not apply.
Single-clip duration is 5–60 seconds, with captions segmented and annotated in 5-second windows; the model generates 93 frames / about 5.8 seconds per pass. Long-horizon capability is achieved through Video2World conditioned continuation and Cosmos-Transfer2.5's long-horizon video translation, rather than by training on long clips.
Multi-view is a "non-narrative structural dimension" unique to this work: autonomous-driving data are surround-view clips from 7 synchronized cameras (30 FPS, 20 seconds); the multi-view model concatenates the 7 views along the latent time dimension (the latent time dimension is compressed to 8 to accommodate 7 views), while the robotics AgiBot variant uses 3 camera views.
Native audio track: not applicable (no audio processing).

### [Data-Juicer 2.0](../models/Data-Juicer.md) ⚠️

[Capability for handling single-shot vs. multi-shot] DJ provides video_split_by_scene_mapper (scene-change detection and splitting) as a standard means of decomposing multi-shot long videos into single-shot clips — this is the most direct lever for controlling narrative structure: enabling this operator means the training data is entirely single-shot; not enabling it preserves the original multi-shot structure. DJ leaves this choice to the user and does not take a default stance.
[Actual structure of the official case study] The three source datasets used in the T2V case study (InternVid, Panda-70M, MSR-VTT) are themselves already-cut single-shot or near-single-shot short clips; the case study does not perform any additional scene splitting, so the final data pool consists predominantly of single-shot short clips.
[Whether native audio track is included] The case study neither requires nor tabulates this; DJ supports processing videos with and without audio tracks, and splitting operators such as video_split_by_scene_mapper process the audio track in sync.
[Uncertain] No statistics on shot-count distribution, average clip duration, or the proportion of samples with audio tracks are reported; DJ also provides no "shot-count statistics" or "narrative-structure profiling" type of analysis operator (its data-analysis reports mainly consist of histograms of single-value statistics rather than structured narrative attributes).

### [Foley-Omni](../models/Foley-Omni.md) ⚠️

Overall it follows a single-shot, short-clip paradigm. Judging comprehensively from the 5–10 second clip duration, the motion-score upper bound of 3.2 that suppresses vigorous motion/fast cuts, and the data sources (GRID/LRS2/Chem/SpeakerVid/TalkVid are continuous talking-head shots, while VGGSound consists of single-event 10-second clips), the training and evaluation data are essentially single-shot, continuous clips with no cut points. This matches the nature of the task: modeling cross-shot scoring coherence is not the goal of this paper.
[Whether native audio track is included] All clips include a native audio track, and the native audio track is the sole supervision target (the generation target is to restore/reconstruct three constituent components of that track). This contrasts with text-to-video models, where "audio track is optional" — for Foley-Omni, samples without an audio track are directly filtered out, and native audio-track quality (AudioBox Aesthetics ≥0.6) is a hard gate.
[Uncertain] No shot-count distribution statistics are given, no precise average clip-duration value is given, and it is not stated whether cross-shot detection and removal was performed.

### [Goku](../models/Goku.md)

Single-shot clips constitute the absolute majority. The design goal of the splitting stage is precisely to "eliminate shot switches": PySceneDetect is first used for shot-boundary detection, followed by a secondary check using DINOv2 feature cosine similarity (the 480×864 tier requires adjacent-frame DINO similarity ≥ 0.85, the 720×1280 tier requires ≥ 0.90); this dual mechanism ensures each training clip is internally visually continuous and contains no transitions. Hence the shot-count distribution can be regarded as "almost entirely 1 shot," and there is no modeling of multi-shot narrative structure.
Average clip duration falls in the 4–10 second range (the specific mean is not disclosed).
Whether native audio track is included: no (the data form is pure video-text pairs).
The paper does not address multi-shot narrative, storyboards, long-video continuity, or other structured narrative data design.

### [Hailuo / MiniMax Video](../models/Hailuo.md) ⚠️

[Uncertain]. Not disclosed: the ratio of single-shot vs. multi-shot, average clip duration, shot-count distribution, or whether training clips retain native audio tracks (given that the model does not generate audio, it is likely that they are not retained). Indirect clues: the output is a single 6–10 second video segment, and video-01-director specifically handles camera-movement control (referring to movement within a single shot, not multi-shot editing), suggesting that the training data is overwhelmingly dominated by single-shot clips, with no observed modeling or data design for multi-shot narrative/storyboard coherence. Multi-shot long-video capability at the product level is achieved by the Media Agent stitching things together at the orchestration layer, rather than being a capability native to the model.

### [HunyuanVideo-Foley](../models/HunyuanVideo-Foley.md) ⚠️

[Predominantly single-shot, determined by the splitting strategy] The pipeline first splits shots via scene detection, then cuts into 8-second blocks, so the vast majority of training clips fall within a single shot. However, unlike UniVerse-1's approach of "discarding clips <5 seconds after splitting," this work regularizes shot segments into 8-second blocks — if a given shot is longer than 8 seconds, it will produce multiple blocks from the same shot; if shorter than 8 seconds, the handling is unspecified (it might be discarded, or it might be concatenated with an adjacent shot, thereby producing a cross-shot sample). The paper does not clarify this. [Uncertain]
[Average clip duration] A strict fixed 8 seconds, so there is no distribution to speak of.
[Shot-count distribution] Not applicable (single-shot by design).
[Whether native audio track is included] Mandatory. The very first step of the pipeline removes videos lacking audio tracks ("eliminate videos lacking audio streams"), a one-vote-veto condition, so all training data carry native, synchronized audio tracks. This is a fundamental requirement for the V2A task — the model is learning exactly the mapping from "image → its true accompanying sound," and samples with post-dubbed or replaced audio tracks would introduce incorrect supervision signals.
[Narrative structure is not a training dimension] 8-second single-shot clips do not carry narrative structure, and this work does not address shot-transition sound effects or acoustic transitions across scene changes at the narrative level — this marks the capability boundary between a Foley model and full cinematic sound design. The paper does not discuss sound-effect coherence for long videos or multi-shot scenes.

### [HunyuanVideo](../models/HunyuanVideo.md)

Both generations explicitly adopt a "single-shot" data paradigm and do not train on multi-shot narrative:
[Original] Uses PySceneDetect to split into single-shot clips; additionally uses Transnet v2 + PySceneDetect in dual paths to provide scene-boundary information for more reliable splitting; the dense description field records scene transitions and camera movement.
[1.5] Goes further: after splitting with PySceneDetect and an in-house operator, an additional dedicated "transition classifier" is applied to entirely remove clips that still retain transition effects (fades, dissolves, etc.), ensuring every training clip is a clean single shot. Average clip duration is in the 2–10 second range; shot-count distribution is meaningless (always 1). Native audio track: none (the audio track is not retained on the data side).

### [InstructAV2AV](../models/InstructAV2AV.md)

All samples are single-shot structure, with no multi-shot samples. PySceneDetect's shot splitting is the first step of the pipeline and explicitly produces single-shot clips; the fixed 5-second length further ensures clips contain no cut points.
[Average clip duration] Strictly 5.0 seconds, with no variance.
[Shot-count distribution] Always 1, no distribution.
[Whether native audio track is included] On the source side, all clips include a native audio track and it is required — clips with no audio track or that are silent (<-45 dBFS) are removed during filtering. On the target side, the audio track is a synthesized product (separated background sound + newly synthesized target sound). This constitutes a special property of the dataset: the two sides of each pair share the same background-sound component and differ only in the foreground sound, forming a precisely controlled differential pair.
[Why single-shot is mandatory] This is not an aesthetic choice but a technical necessity: Grounded-SAM-2's instance masks need to propagate continuously within a clip, CoTracker3's point tracking needs a continuous field of view, and the mask-guided video-editing model needs to perform local replacement within a stable spatiotemporal context — any shot switch would cause all three to fail simultaneously.
[Limitation] The resulting capability boundary is clear: the model cannot handle cross-shot editing consistency (such as "replace this person throughout the entire film"), nor can it edit narrative segments that contain cuts. This is the same issue viewed from another angle as the limitation the paper acknowledges: "difficulty maintaining 3D spatial consistency and object permanence under large camera motion."

### [Other Audio-Video Joint Generation Works of 2026](../models/JAVG_2026_misc.md) ⚠️

[Predominantly single-shot; only StreamChar addresses long-form narrative]
(1) Single-shot shorts: OmniCustom (fixed 5-second single-person speech, single-shot), CCL (352×640 short clips), Baton (short clips, evaluated at 10 seconds), NAVA (average 7 seconds, processed through a Hadoop pipeline, presumably single-shot clips), and ALIVE (3–10 second clips, output 5–10 seconds) are all single-shot structures with no shot-switching or transition samples.
(2) Short extractions from long sources: ALIVE's character-driven pipeline explicitly extracts N 3–10 second clips from "10–30 minute long videos" — i.e., a long video is treated as "a multi-shot material pool for the same identity," used to construct cross-clip identity pairing (cross-pair). This indirectly introduces a supervision signal for "cross-shot consistency of the same character," even though each individual training sample remains single-shot.
(3) The sole explorer of long-form narrative: StreamChar is the only work in this batch to directly address long-horizon structure, but its solution is not to train on long-video data — on the contrary, training data is capped at 20 seconds ("training data contains no videos/transcripts longer than 20 seconds"); long-horizon capability is obtained entirely at inference time through chunk-wise autoregressive stitching (each chunk is 33 frames @24fps), plus two explicit anti-drift mechanisms: a progress-aware pointer (using ground-truth end indices from ASR timestamps for smooth-L1 supervision, so the model knows how far along the dialogue is) and a persistent visual anchor / sink chunk (a persistent visual anchor suppressing identity drift). Ablation shows that removing the sink chunk worsens drift from 0.0067 to 0.0304 (about a 4.5x degradation), quantitatively proving the critical role of the visual anchor for long-horizon consistency. Evaluation involves generating a 5-minute continuous stream (using randomly selected dialogue of >300 words).
(4) Proportion with native audio track: all seven items use paired data with native synchronized audio tracks — ALIVE explicitly states it "begins by filtering videos with audio from the raw data pool" (the very first step is to filter the raw pool for videos with audio tracks); OmniCustom/StreamChar's upstream datasets are themselves audio-video paired, and NAVA's audio-video subset is the same. None adopts the route of "strip the audio track first, then dub it later."
(5) None of the seven items disclose statistics such as shot-count distribution or average shot count [Uncertain].

### [Collection of Audio-Video Joint Generation Baselines](../models/JavisDiT_baselines.md) ⚠️

All are the simplest configuration of "single-shot, short clip, with native audio track," with none addressing multi-shot narrative:
[Shot count] All five works' training samples are single-shot clips. MM-Diffusion mechanically cuts 928 source videos into 1,000 non-overlapping 10-second clips (no shot detection is mentioned, so cut points may cross shots — a rough spot in this early data-processing work); JavisDiT relies on TAVGBench's existing clip divisions; Harmony's 3–10 second clips and UniAVGen's real-person clips are likewise single-shot.
[Average clip duration] The MM-Diffusion dataset is 10 seconds, with the actual training clips being shorter; AV-DiT uses 16 frames + 1.6 seconds of audio; JavisDiT/JavisDiT++ are fixed at 4 seconds; Harmony is 3–10 seconds (a range distribution, the only non-fixed-length one in this collection); UniAVGen is not disclosed [Uncertain].
[Native audio track] Training data for all five is 100% native synchronized audio — this is a prerequisite for joint audio-video generation. MM-Diffusion's AIST++ is a special case: its music is accompaniment paired with the dance rather than a live recording, which is "aligned at production time" rather than "synchronously captured at shooting time."
[Narrative capability] None of the five have any design for shot transitions, multi-shot consistency, or story structure; the "complex scenes" subset of Harmony-Bench is the closest attempt to narrative complexity, but the complexity lies at the acoustic level of "co-occurrence of speech and ambient sound" rather than at the shot-narrative level.
[Gap versus industrial models] Compared to Veo 3 / Sora 2 / LTX-2, which have already begun handling multi-shot and minute-scale narrative, this entire collection remains at the "single-shot ≤10 seconds" stage. This is both a result of academic compute constraints, and it means the data-processing experience of these baselines is difficult to transfer directly to long-video scenarios.

### [Kling 3.0 / Kling Video 3.0 Omni](../models/Kling_30_Omni.md) ⚠️

[Uncertain] The proportion of single-shot vs. multi-shot samples and the shot-count distribution are not published. Structural facts that can be inferred: (1) the basic cleaning stage removes clips with abrupt transitions/incoherent shot switches, indicating the pretraining corpus is predominantly single-shot continuous clips; (2) the product supports "intelligent storyboarding" with up to 6 shots per submission and cross-shot character/timbre consistency (Director Memory), implying the post-training stage must have introduced multi-shot/storyboard-level long-context samples (including shot-boundary annotation and cross-shot identity/timbre correspondence); (3) the requirement for native synchronized audio-visual output means training samples must carry native audio tracks, so the high-quality subset should be predominantly clips "with native synchronous sound," with audio-less samples either removed or used only in a pure-visual stage. Points (2) and (3) above are inferences based on product capability; the report does not explicitly state them.

### [LTX-2](../models/LTX-2.md) ⚠️

Not disclosed: the single-shot vs. multi-shot ratio and the average shot-count distribution. What is confirmed is that the data-organization granularity is single-shot: the pipeline operates in shot units throughout, from "Input Shots" to "Final Shots," so training samples are predominantly single-shot clips, and the model does not tout multi-shot narrative as a selling point (the paper explicitly lists "deeper narrative coherence" as a limitation requiring external LLM-generated conditioning text to compensate). Whether native audio track is included: the LTX-2 subset, by definition, consists entirely of samples with native, synchronized audio tracks that carry significant audio information — this is the key point distinguishing it from the parent collection. See the LTX-Video duration histogram (0–30 seconds) for average clip duration. [Uncertain]

### [LongCat-Video](../models/LongCat-Video.md)

After scene-detection splitting, training data consists of single-shot clips — the role of PySceneDetect + TransNetV2 is precisely to eliminate shot switches, ensuring each training clip is internally continuous and consistent; hence the training set contains no multi-shot narrative samples, and there is no concept of a shot-count distribution. Average clip duration is not disclosed, but training uniformly samples 93 frames (at 30fps, about 3.1 seconds). Native audio track: the base version's training discards the audio track entirely (pure vision); the Avatar series, however, relies on native audio tracks (audio and video are paired and must pass lip-sync verification).
Multi-shot/long-narrative capability is not solved at the data layer but rather at the inference layer via Video-Continuation recursive continuation: conditioning on several preceding frames to generate continuously, combined with optimization targeting cross-frame temporal consistency and physical-motion plausibility, suppressing color drift, quality degradation, and motion breaks, thereby stably producing minute-scale long videos. Avatar 1.5 supports multi-segment rollout of up to 5 clips during the GRPO stage (only the last clip participates in optimization), which reflects that its long videos follow a multi-segment concatenation paradigm.

### [MOVA](../models/MOVA.md)

MOVA explicitly makes "single-shot vs. multi-shot" an orthogonal data dimension, which is relatively rare among comparable works:
- Through the cross-combination of VAD timing information and PySceneDetect scene cut-points, four categories of clips are generated: single-scene speech, single-scene non-speech, multi-scene speech, and multi-scene non-speech (i.e., a 2×2 division of {single/multi-shot} × {speech/non-speech}).
- The criterion for judging a multi-shot clip is hard: the window's time span must contain at least one scene cut-point, otherwise it is discarded (Appendix A.3, Algorithm 1, line 11).
- The criterion for judging a single-shot clip is likewise hard: the window must fall entirely between two adjacent scene cut-points (Algorithm 2, lines 5 and 19).
- Average clip duration is fixed at 8.05 seconds (193 frames @24fps), so there is no length distribution to speak of.
- Whether native audio track is included: training data must carry a native, synchronized audio track — the very first step of preprocessing is removing samples that "fail to decode" or "lack a valid audio channel," so no audio-less samples exist.
- The annotation stage also imposes specific requirements for multi-shot content: the video-caption prompt explicitly instructs MiMo-VL to "focus on video scene transitions," i.e., shot transitions are explicitly annotated.
- The specific ratio of single-shot to multi-shot clips is not disclosed.

### [Merged entry: ① Mochi 1](../models/Mochi_MAGI_Motif.md) ⚠️

① Mochi 1: not disclosed. On the generation side it outputs a fixed single 5.4-second video, suggesting the training material is inferred to be single-shot short clips, but there is no official statement. [Uncertain]
② MAGI-1: training clips are strictly single-shot — PySceneDetect splits "ensuring that each clip contains only a single shot," with an additional Transition Detection actor as a backstop (PySceneDetect struggles with complex transitions, so a cut clip may still contain multiple shots; therefore keyframes are sparsely sampled, CLIP is used to compute semantic similarity between adjacent keyframes, and if it falls below a threshold the clip is judged multi-shot and discarded in its entirety). Average clip duration by stage is ≤8s / ≤8s / ≤16s. No native audio track.
Worth calling out separately: although MAGI-1's training material is single-shot, it obtains multi-shot narrative capability at inference time through autoregressive captioning (see caption_structure) and chunk-wise text conditioning — the technical report's Fig. 10 shows a nearly 30-second example of complex action and narrative structure generation driven segment-by-segment by seven different prompts (a)–(g), and Fig. 12 also shows two types of shot transitions achieved by modulating the KV range at different denoising stages: "identity-preserving shot switches" and "transitions that preserve scene layout while changing object details." In other words, multi-shot narrative in MAGI-1 is realized by "architecture + caption schema" rather than by "multi-shot training data" — an interesting contrast on the data side.
③ Motif-Video 2B: likewise strictly single-shot, with an even finer splitting strategy: scene boundaries are first detected with a conservative threshold "biased toward over-splitting (better a false positive than a missed transition)," then SigLIP embedding similarity is used for stitch detection to re-merge consecutive shots that were mistakenly split due to momentary motion or exposure changes, and finally merged clips shorter than 2 seconds are discarded. In addition, the multi_scene label produced by the VLM is used as a "secondary check on scene splitting" — a hit results in discarding. No native audio track. Shot count is always 1, with no multi-shot narrative data construction.

### [Movie Gen](../models/Movie_Gen.md)

The video training data is entirely single-shot: FFmpeg scene-boundary detection + PySceneDetect jitter detection ensures no transitions or jump cuts within a clip, and requires the presence of non-trivial motion; only 1–2 scenes are taken per original video, 1 clip per scene, with average clip length falling in the 15–16 second range (>50%). Hence the model is a single-shot generator and does not model multi-shot narrative.
Whether native audio track is included: Movie Gen Video's training does not use the audio track at all (pure vision + text); Movie Gen Audio's pretraining, by contrast, relies on the video's native audio track as the supervision signal. The two data flows are separate.
Multi-shot appears only on the evaluation side: Movie Gen Audio Bench divides evaluation videos into single-shot (large in number, with a broad sound-effect lineage, testing robustness and generalization) and multi-shot (taken from short films, containing scene transitions, with stronger emotion and narrative, used to evaluate video-music alignment, such as when the music enters, how it evolves with the plot, whether it aligns with cut points, and whether the mix of sound effects and music is harmonious).

### [NVIDIA NeMo Curator](../models/NeMo_Curator.md) ⚠️

[Single-shot vs. multi-shot] The framework's default output is single-shot clips: TransNetV2 performs shot-aware splitting, and each resulting clip in principle does not cross shots. But there is a noteworthy reverse-direction design in the pipeline — the stitching stage: after splitting, image-embedding similarity is used to judge whether adjacent clips are content-continuous, and if similarity is high they are re-merged, to avoid over-fragmenting the same continuous scene (e.g., due to a brief camera occlusion or flash causing a false cut). This is a two-stage "split — then re-stitch" design, more robust than relying on scene detection alone.
[Shot-count distribution] The final dataset is predominantly single-shot clips; multi-shot narrative samples are not a construction goal; there is no statistical figure for shot-count distribution.
[Average clip duration] Range of 2–60 seconds; the specific mean is not published.
[Whether native audio track is included] The pipeline neither retains nor processes the audio track; transcoded output is H.264 video, so the final WebDataset contains no audio-track information. [Uncertain]

### [OmniHuman Dataset + OHBench](../models/OmniHuman.md) ⚠️

[Predominantly single-shot, guaranteed by splitting] Using TransNetV2 for shot splitting, each clip corresponds to one continuous shot, so training samples are essentially single-shot. Combined with an average clip duration of about 6.5 seconds, the data does not carry cross-shot narrative.
[But shot-level metadata is retained] Although samples themselves are single-shot, the annotation includes shot scale (e.g., close-up/medium/full/wide) and camera-motion fields — this turns "shot language" into a controllable condition rather than information that is erased. The finding that "model performance degrades on close-up→wide shot transitions" during evaluation was only diagnosable because of this scale annotation.
[Average clip duration] About 6.5 seconds (back-calculated from 1800h / 1M); the distribution is shown in Figure 3(d), but no value is given in the main text. [Uncertain]
[Shot-count distribution] Not applicable (each sample is single-shot).
[Whether native audio track is included] Mandatory. The audio-governance stage removes samples with "missing tracks," and SyncNet attribution verification requires the audio track to correspond to the person in the picture; hence all samples carry a native audio track that is co-sourced and synchronized with the picture.
[Interaction structure as a substitute narrative dimension] This dataset does not pursue cross-shot narrative, but instead pursues richness of "relational structure" within a single shot — two-person dialogue (including speaker turn-taking and listener reactions) and human-object interaction constitute its distinctive structural content. This effectively shifts narrative complexity from "shot assembly along the time axis" to "multi-subject relationships within space" — a route orthogonal to multi-shot datasets such as MuSS.

### [Open-Sora series](../models/Open-Sora.md)

The data for both projects are **strictly single-shot** clips — this is the core design orientation of their pipelines:
- Open-Sora 2.0 uses FFmpeg libavfilter's scene score for shot-boundary detection and splitting, then overlays a secondary removal step using PySceneDetect-based camera-shake/jump detection to ensure no shot switches within a clip; long clips are mechanically cut into 8-second segments.
- Open-Sora 1.x uses PySceneDetect to detect and split scenes, with output named `{video_id}_scene-{scene_id}.mp4`, likewise single-scene clips.
- Open-Sora Plan v1.3, after a 16-second split, specifically adds a level of **LPIPS jump-cut detection**, using sudden shifts in perceptual distance to identify cut points, with a 97% retention rate, explicitly targeting the removal of clips containing shot switches.
Therefore: multi-shot narrative samples are actively removed rather than retained; the average shot count in training data is 1; there is no cross-shot consistency supervision signal, and the model does not have multi-shot narrative generation capability. Average clip duration: Open-Sora 2.0 leans toward 6–8 seconds; Open-Sora Plan is about 1.3–21 seconds (retained by frame-count range after the 16-second split). Whether native audio track is included: the data itself has an audio track, but it is completely ignored by the pipeline and dropped during transcoding.

### [Ovi](../models/Ovi.md) ⚠️

[Predominantly single-shot] Splitting is done via scene detection, and the splitting granularity itself ensures each 121-frame clip lies within a single shot; hence training samples are nearly all single-shot clips containing no shot switches. This corroborates the paper's Limitations section, which states "inter-shot transitions and global story consistency are not in scope."
[Average clip duration] Fixed at 5.04 seconds (10 seconds for Ovi 1.1), with no length distribution — all samples are equal length.
[Shot-count distribution] Always 1 shot per sample.
[Native audio track] 100% include native audio track — the entire value of the audio-video paired corpus lies in the native synchronized audio track, which has been filtered by SyncNet and volume-threshold checks, removing audio-less or near-silent clips. This is the opposite of the approach some models take of "strip the audio track first, then dub it in post."
[Source of narrative capability] The paper claims the model can do "cinematic storytelling," but the narrative quality comes from visual events and dialogue annotations interwoven chronologically in the caption (chronology is explicitly required), not from multi-shot data.

### [Script-a-Video](../models/Script-a-Video.md) ⚠️

Narrative structure is a first-class citizen of MTSS, thoroughly handled at the schema level, but distribution statistics are missing:
[Single-shot vs. multi-shot] Explicitly supported and explicitly modeled simultaneously. The Shot stream itself is "a decomposition of the visual presentation into a sequence of cinematic segments," with each shot carrying an independent time_range. The evaluation set is divided into 125 single-shot / 100 multi-shot clips (about 5.6:4.4), showing that multi-shot is a first-class evaluation scenario rather than an afterthought.
[Shot-count distribution] The average number of shots per clip within the 500K dataset and the distribution range of shot counts are not disclosed by the paper. [Uncertain]
[Average clip duration] Not disclosed.
[Whether native audio track is included] Necessarily included — the construction of the Event stream depends on real dialogue and sound effects from the actual audio track, and the Global stream's global_audio depends on real ambient sound; without a native audio track, most fields of the entire schema would be empty. However, the paper does not explicitly state a pre-filtering rule such as "videos without audio tracks are removed."
[Explicit modeling of shot language] The Shot stream contains a camera field, specifically recording professional cinematic language: movements, perspectives, and scales. This turns shot language into a field that can be structurally retrieved and used for controllable generation, rather than being scattered across free text.
[Mechanism guaranteeing cross-shot narrative continuity] Cross-shot narrative coherence is guaranteed by Relational Grounding: the references_in_shot array maps subjects appearing in each shot to persistent Reference IDs, and active_events links shots to concurrent auditory events. As a result, the same character appearing in multiple shots does not need to be re-described in appearance — only the same ID needs to be referenced. This directly solves the "identity drift caused by repeated description" problem seen in monolithic captions.
[Quantified generation results] Shot Boundary Deviation drops from the baseline LTX-2-AV's 3.79 frames to 0.38 frames (Ours w/o ID configuration); human-rated MSC (multi-shot consistency) rises from 1.00 to 2.49–2.62, proving that the temporal segmentation prior provided by the Shot stream does indeed make multi-shot controllability feasible.

### [Seedance 2.0 and Seedance 1.5 pro](../models/Seedance_20_Seedance_15_pro.md) ⚠️

Seedance 1.0's splitting strategy explicitly allows "each clip to contain one or multiple temporally coherent shots" in order to preserve local narrative flow, which is the data foundation for its multi-shot generation capability. Seedance 1.5 pro emphasizes "narrative coherence" and the application potential of multi-shot video-generation workflows, and possesses capabilities such as continuous long takes, dolly-zoom-style camera moves, and cinema-grade transitions. Seedance 2.0's main selling point is "native professional-grade multi-shot narrative capability," able to autonomously plan shot sequencing and design visual-presentation templates; SeedVideoBench 2.0 adds a narrative-quality metric for this purpose, comprising three sub-dimensions: cinematography (shot logic and expressiveness, checking for redundant coverage, 180-degree-rule/crossing-the-line violations, mismatched shot scales, and uneven pacing), plot design, and stylized aesthetics. Specific statistics such as the proportion of training data with a native audio track, average clip duration, and shot-count distribution are not disclosed. [Uncertain: quantitative distribution]

### [SkyReels series](../models/SkyReels.md) ⚠️

[SkyReels-V2] Data-processing granularity is single-shot: all raw videos are first cut into single-shot clips via shot-boundary detection before entering downstream processing, so training samples are predominantly single-shot; multi-shot/long-narrative capability is achieved on the generation side by the Diffusion Forcing framework via autoregressive continuation (a non-decreasing noise schedule supports indefinite extension), rather than through multi-shot training samples. One of the paper's core selling points is "shot-aware generation," but this is reflected in the caption's shot-type/camera-position/movement fields, not in the multi-shot structure of the samples.
[SkyReels-V4] Explicitly supports and evaluates "multi-shot cinematic-grade narrative": SkyReels-VABench's prompt set covers "a complexity gradient from single-shot to multi-shot," and provides video extension and shot-transition capability. Regarding native audio track: V4's audio-video joint training data must carry a synchronized audio track and pass SyncNet verification, but the paper does not give distribution figures such as "proportion of samples with native audio track," "single-shot vs. multi-shot sample ratio," or "average shot count / average clip duration." [Uncertain]

### [Sora 2](../models/Sora_2.md) ⚠️

The shot-count distribution and single/multi-shot ratio of the training data are not disclosed. On the capability side, it is disclosed that Sora 2 can "follow intricate instructions spanning multiple shots while accurately persisting world state," i.e., it supports instruction-following across multiple shots while keeping characters, environment, and lighting consistent across shot switches — indicating that the training data must contain multi-shot narrative samples with cross-shot consistency supervision. The Storyboard tool, launched in October 2025, further allows users to plan multi-scene videos segment by segment. Whether native audio track is included: from the native audio-video joint-generation capability, it can be inferred that the training data is predominantly "videos with native synchronized sound," but the average clip duration, shot-count histogram, and the ratio of audio-present/audio-absent samples have no figures given. [Uncertain]

### [SpeakerVid-5M](../models/SpeakerVid-5M.md) ⚠️

[Single-shot vs. multi-shot] The dataset is a strictly single-shot corpus. The first splitting step uses PySceneDetect to detect significant changes in color and brightness to locate transitions, and cuts at those transitions, ensuring no shot switches within each clip; YOLO tracking-based spatial cropping is then applied, further ensuring subject continuity within a clip. Hence there are no multi-shot samples, and no shot-count distribution exists — this stands in clear contrast to MOVA's deliberate retention of a "multi-scene clip" category.
[Average clip duration] About 6.0 seconds (within the 3–14 second range); see duration_distribution for details.
[Whether native audio track is included] All clips include native synchronized audio track. The dataset was audio-visual paired from the point of collection (the original collection scope was "153K audio-visual videos"), and has passed SyncNet lip-sync verification and Whisper audio-quality filtering, removing samples without audio tracks or with audio-visual desync.
[A higher-level narrative structure: dialogue turns] SpeakerVid-5M introduces a "turn" structural layer that is rare in video datasets: the dialogue branch pairs same-period clips of two speakers; the multi-turn branch further chains multiple turns along the time axis — the contextual type aggregates the ASR transcripts of preceding turns as multi-turn dialogue context, while the sequential type judges adjacent clips with a time gap smaller than a threshold δt as a continuous dialogue and concatenates them. This means the dataset's narrative structure is not a "shot sequence" but a "dialogue turn sequence" — a unique narrative organization geared toward interactive-generation tasks.
[Turn-count distribution across branches] The paper does not give the distribution statistics of the number of turns for the multi-turn branch. [Uncertain]

### [Step-Video-T2V](../models/Step-Video-T2V.md)

Explicitly adopts a "single-shot" data paradigm and does not train on multi-shot narrative: PySceneDetect's AdaptiveDetector detects scene changes, then FFmpeg splits accordingly, with each segment containing only one shot, and 3 frames are removed from both ends to further eliminate residual transition artifacts. Hence shot-count distribution is always 1, and average clip length falls into one of three tiers: 68/136/204 frames (plus a 1-frame image tier).
Notably, dense captions explicitly describe "camera movements," and the manual review criteria in the SFT stage include an item on "whether scene transitions are smooth," showing the team pays attention to shot language — but this is at the level of camera movement within a single shot, not the structure of cross-shot multi-shot narrative.
Native audio track: none (the audio track is neither retained nor processed on the data side).

### [UniTalking](../models/UniTalking.md) ⚠️

[Single-shot vs. multi-shot] The paper does not discuss shot structure and does not describe any shot-splitting step (see shot_segmentation). However, the training data can be indirectly inferred to be effectively single-shot from two constraints: first, cross-modal lip-sync filtering (LipSync) requires a continuously detectable face synchronized with the audio track throughout the frame; a shot switch would break this continuity and would likely cause the sample to be eliminated; second, LightASD's active-speaker detection similarly relies on a continuous face trajectory. Hence multi-shot samples are very likely implicitly excluded by the filtering pipeline, but this is a side effect of filtering rather than an explicit design. [Uncertain]
[Shot attributes of upstream data] OpenHumanVid has already completed "decoding, cropping, segmentation" preprocessing in its own pipeline, so the clips UniTalking draws from are already segmented short-shot units — this may be exactly why UniTalking does not need to build its own shot-splitting module. Whether the Huawei in-house data underwent equivalent processing is not stated. [Uncertain]
[Average clip duration and shot-count distribution] Entirely undisclosed. [Uncertain]
[Whether native audio track is included] Mandatory, and it must be an on-screen sound source. This is the strictest audio requirement in this work: not only must there be an audio track (excluding silent videos), not only must there be speech (excluding pure ambient-sound tracks), but it must also be synchronized speech emitted by the on-screen person (LightASD excludes off-screen sound sources + LipSync excludes lip-audio desynchronization). These three conditions chained together are considerably stricter than UniVerse-1's "audio track suffices + speech subset additionally checked with SyncNet."
[Narrative complexity] The data consists of short single-person talking clips and does not carry narrative structure. The model also explicitly does not support multi-reference generation (Future Work acknowledges it lacks Sora2's "Cameo"-style multi-clip input capability).

### [UniVerse-1](../models/UniVerse-1.md)

[Single-shot vs. multi-shot] UniVerse-1 adopts a "pure single-shot" strategy but does not discuss it explicitly: after PySceneDetect detects scene cut-points, splitting occurs at those points, and only splitting results with length ≥5 seconds are retained, so each training clip naturally falls within a single shot, with no cross-shot samples existing. The paper does not treat "multi-shot" as an independent data dimension and does not generate multi-shot training samples — this contrasts sharply with MOVA's explicit construction of a 2×2 four-category clip scheme of "single/multi-shot × speech/non-speech," and also means UniVerse-1 lacks shot-transition generation capability.
[Average clip duration] Only the lower bound of 5 seconds is known; the training consumption window is about 5 seconds; the mean and distribution are not disclosed.
[Whether native audio track is included] Mandatory. The very first step of the cleaning pipeline is "videos lacking an audio track were immediately discarded," so all training data carry a native, synchronized audio track — there are no post-dubbed or audio-less samples.
[Shot-count distribution] Not applicable (all single-shot).
[Narrative complexity] Constrained by a window length of around 5 seconds, the training data is short single-shot clips that do not carry narrative-structure information.

### [Unison](../models/Unison.md) ⚠️

The paper does not treat narrative structure as an independent dimension and provides no related statistics. The following is indirect inference:
[Single-shot vs. multi-shot] Almost certainly purely single-shot. Among the five audio-video data sources, HDTF, VFHQ, and CelebV-Text are all already-cropped single-face, single-shot clips; VGGSound consists of fixed short clips; OpenHumanVid is also published in already-segmented clip form. The average duration of about 5.4 seconds also essentially rules out the presence of multi-shot samples. The paper does not mention any multi-shot sample construction or shot-transition processing; the model has no shot-switch generation capability. [Uncertain]
[Average clip duration] Estimated at about 5.4 seconds (3,000 hours ÷ 2 million clips), not directly given in the paper.
[Shot-count distribution] Not applicable (inferred to be entirely single-shot).
[Whether native audio track is included] On the audio-video side, all clips have a native synchronized audio track — all five data sources are videos with sound, and the lip-filtering operator explicitly removes off-screen voice-overs, meaning the speech that is retained must correspond to the on-screen person's lip movement. This is a stricter requirement than "audio track suffices."
[Pure-audio side does not carry narrative structure] The 130,000 hours of audio data have no visual accompaniment and are used only for single-modality training of the Stage 1 audio branch.
[Assessment of narrative complexity] Single-shot short clips of about 5 seconds do not carry narrative-structure information; Unison's target capability is "precise multimodal alignment within a short time window" rather than "long-form narrative." This is consistent with its architectural choice — cross-attention over a three-frame window (stride=1, only the middle frame retained) is an extremely local alignment design, naturally geared toward short clips.

### [Veo 3 / Veo 3.1](../models/Veo_3_Veo_31.md) ⚠️

[Uncertain] The proportion of single-shot vs. multi-shot in the training data, average clip duration, and shot-count distribution are not disclosed. Inferred clues: the technical report explicitly records that the model exhibits a cinematic preference for "frequent shot switching and dramatic camera positions," indicating the training data contains a large amount of multi-shot edited footage that was not entirely cut into single-shot clips — otherwise the model would not spontaneously produce cut points within 8 seconds. This is an important piece of indirect evidence about the composition of Veo 3's data. Another confirmed point: the training data must be predominantly "with native audio track," because audio-video latents need to enter the joint diffusion process as pairs.

### [Vidu S1](../models/Vidu_S1.md) ⚠️

All clips are single-shot, explicitly excluding multi-shot narrative: original videos are split into single-shot clips along shot boundaries, with long shots further subdivided; the final clips are all 3–60 second single-shot, single-person clips. All training data include native audio tracks (the pre-filtering stage removes samples lacking an audio track or with incomplete audio-video content based on "audio-visual integrity"). No values are given for average clip duration and shot-count distribution [Uncertain].

### [Wan 2.5 / 2.6 / 2.7](../models/Wan.md) ⚠️

No ratio figures at all are given on the training side, but the evolution of capabilities across versions allows a reverse inference of major changes in data structure.
- Wan 2.1/2.2 era: training samples were fixed 5-second clips; both model capability and evaluation (Wan-Bench) were organized around single-shot content, with no concept of multi-shot narrative.
- Starting with Wan 2.6: the official release added "multi-shot narrative / storyboard control," explicitly enabled via the API through shot_type:"multi" + prompt_extend:true; the official description states it "constructs the raw input into a professional-grade multi-shot passage with a complete storyline and narrative tension through high-level semantic understanding, unified modeling of core subjects, scene layout, and environmental atmosphere across seamless multi-shot switches."
- Wan 2.7: removes the shot_type parameter (the configuration no longer takes effect), switching entirely to natural-language control of shot structure — users write "generate a single-shot video" / "generate a multi-shot video," or directly use a timestamped storyboard script (official example: "Shot 1 [0-3 seconds] wide shot: a New York street on a rainy night... Shot 5 [12-15 seconds] close-up: ..."); when unspecified, the model decides for itself. The prompt-length limit also jumps from 1500 characters in 2.5/2.6 to 5000 characters for wan2.7-t2v.
[Implications on the data side] This 2.7 interface design strongly suggests the training captions contain "timestamped, storyboard-level structured descriptions" — i.e., training samples have shifted from single-shot clips to "multi-shot clips + shot-by-shot timeline annotation," requiring cross-shot subject/scene consistency annotation as well. This is structurally analogous to LTX-2's "training caption as inference interface."
[Native audio track] Starting with 2.5, the entire lineup is labeled as "video with sound" (wan2.6-i2v-flash supports both with-sound and silent), suggesting training samples predominantly carry native synchronized audio tracks.
The single-shot vs. multi-shot ratio, average shot-count distribution, and average clip-duration distribution are all undisclosed. [Uncertain]

### [Collection of Audio-Video Generation Evaluation Benchmarks](../models/av_benchmarks.md)

All five focus on evaluating single-shot short clips, with no multi-shot narrative evaluation constructed:
[VABench] 5–10 second single-shot generation, but the "complex scene" category indirectly examines narrative expressiveness through sub-dimensions such as complex soundscapes, world knowledge, and symbolic association; the two Module 2 items Expressiveness (narrative effectiveness and emotional alignment) and Artistry (aesthetic expressiveness of audio-visual fusion) are the only narrative-level metrics.
[AV-SyncBench] 3–13 second in-the-wild clips, all with native audio track (this is a prerequisite for inclusion), predominantly single-shot; dialogue and ensemble scenes implicitly involve multiple sound sources but not multiple shots.
[PhyAVBench] Controlled recordings of single-shot short clips, all with native synchronous sound.
[AVBench] Single-shot scenes of real people; the Hard subset's 3–4 overlapping-speaker speech is its setting closest to multi-subject narrative.
[Omni-Judge] Sora 2 / Veo 3 default single-prompt generation, not involving multi-shot.
Overall, long-horizon, multi-shot, and cross-shot audio-track continuity remain blank areas in the current AV evaluation landscape; even if training data is already working on multi-shot narrative data, there is currently no corresponding benchmark to verify it.

### [Video Captioning Model Ecosystem](../models/caption_models.md) ⚠️

[Single-shot vs. multi-shot labeling divide] Clips used to train mainstream generative models are single-shot (after splitting via tools such as PySceneDetect), so the vast majority of captioners are optimized for single-shot. A few exceptions target multi-shot/narrative content:
· ShareCaptioner-Video's DiffSW explicitly models "scene transitions," making it one of the few open-source captioners that natively support cross-shot description.
· AVoCaDO's five-dimensional keypoints include "Spatio-temporal & Cinematography," which explicitly covers scene switching, temporal progression, and camera movement.
· Shot2Story (20K clips within AVoCaDO-SFT) and Movie101 (one source of AVSCap-130K) are themselves multi-shot narrative datasets.
· CineDance uses Qwen3.5-27B for shot grouping and narrative-boundary determination; the bottom-up strategy achieves F1 = 88.4%, with only 3.1% of sequences shorter than the 20-second soft threshold — the only work in this ecosystem to give quantitative metrics for narrative-structure parsing.
· MOVA explicitly requires "focus on video scene transitions" in its visual-annotation instructions.
[Whether native audio track is included] Pure-vision captioners completely ignore the audio track (Panda-70M even explicitly disables Video-LLaMA's audio branch); audio-video captioners require samples to have a valid audio track — LTX-2's data-subset filtering criterion is precisely "contains a significant and information-rich audio component."
[Average clip duration] Koala-36M averages 13.75 seconds; AVSCapBench 30–120 seconds; Foley-Omni fixed at 8 seconds.
[Uncertain] No captioner publicly discloses a shot-count distribution histogram for its training data.

### [Collection of Geometric/Structured Annotation Datasets](../models/geometric_datasets.md)

SceneScribe-1M: explicitly pursues single-shot continuity — TransNetV2 first detects hard cuts and dissolve transitions, splitting "non-continuous" videos into semantically coherent single-shot clips and re-filtering; the final library is purely single-shot. SpatialVID: modifies PySceneDetect (adjusting the sensitivity threshold + an interval-based multi-frame comparison strategy) to ensure no transitions within 3–15 second clips — likewise a single-shot library, but with an emphasis on the complexity of camera trajectories within a shot (80% contain curved/turning trajectories). WildWorld: continuous in-game recording, naturally single-shot, organized at two levels — action-sequence and sample — with the narrative unit being a combat round. Action100M: does not pursue single-shot; instead it constructs a hierarchical narrative tree "above the shot" — using Ward-linkage hierarchical clustering to organize long videos into a multi-level temporal tree (Tree-of-Captions), preserving semantics at three levels simultaneously: frame-level, clip-level, and aggregate-level. None of the four have narrative annotation related to native audio tracks.

### [Video-Generation Post-Training Data Strategies](../models/post_training_data.md) ⚠️

The anchor paper does not address this [Uncertain]. Cross-comparison notes: Movie Gen's video SFT set is 50% long clips of 16 seconds, and its manual-selection criterion is "cinematic," hinting at a predominance of complete single-shot narrative;
· The post-training stage generally favors single-shot, transition-free material — one of Step-Video-T2V's manual review criteria is precisely "whether scene transitions are smooth," and Motif's SFT admission criteria include action=Dynamic;
· Multi-shot capability is primarily handled by pretraining and by prompt structure on the inference side; no work has been observed to explicitly control shot-count distribution at the SFT/preference-data layer [Uncertain];
· Autoregressive distillation (the anchor paper's Phase 4, StreamChar, LongLive, OmniForcing, Causal Forcing) converts the narrative-structure problem into a "long-horizon rollout error accumulation" problem; its training data is online rollouts self-generated by the student model rather than collected material — this is a technical route on the narrative-structure dimension that is entirely different from data collection.

### [Combined Survey of Mainstream Video Pretraining Datasets: Panda-70M, InternVid, Koala…](../models/pretraining_datasets.md)

**The seven datasets' treatment of "shot count" reveals two diametrically opposed routes — this is one of the most valuable comparative dimensions of this survey**:
[Route A: strict single-shot, treating multi-shot as noise to be removed — six datasets]
- Panda-70M: PySceneDetect shot-splitting + ImageBind feature constraints, aiming for single-shot with semantic consistency; however, its **TransNetV2 shot-boundary annotations** released additionally in October 2024 (a list of intervals per clip, with a length of 1 meaning single-shot) — **this itself is an admission that the original pipeline still leaked multi-shot clips**.
- InternVid: PySceneDetect (threshold 27) shot-splitting.
- Koala-36M: a self-developed Color-Struct SVM + 3σ probability threshold, with a **recall of 0.9395** (deliberately biased toward recall, "rarely misses transitions"), the most thorough of the seven at removing transitions.
- OpenVid-1M: a cascaded shot-splitting detector cuts multi-scene source videos into single-scene clips.
- UltraVideo: PySceneDetect AdaptiveDetector run twice + DINOv2 first/last-5-frame similarity to catch dissolve transitions.
- LVD-2M: **"long takes with no shot switches" is one of its four criteria**; a low-sample-rate PySceneDetect is used to filter out gradual transitions as well as hard cuts. Human evaluation shows its long-take purity rate is **77.5%** (excluding the WebVid portion) / **86.8%** (including the full WebVid portion), significantly higher than HD-VG's 55.0%, Panda-70M's 50.0%, and InternVid's 47.5% — **these figures are the most direct third-party evidence that "nearly half the clips in mainstream datasets contain shot switches."** The authors admit the remaining failure mode is "subtle jump cuts," which neither PySceneDetect nor MLLMs can catch.
[Route B: over-split then re-stitch, actively retaining long continuous shots — one dataset]
- MiraData: deliberately uses PySceneDetect with a **low threshold of 26** to over-split ("ensuring that all distinct clips are extracted"), then uses a **four-model voting scheme** to stitch adjacent clips back together (see the shot_segmentation field for details). This is the only one of the seven to treat "scarcity of long takes" as a core asset to cultivate. The authors openly acknowledge a limitation in the v0 README: "clips depicting the same scene from multiple camera angles are not abundant in this release."
[Average shot count and native audio track] The average shot count of training clips across all seven is 1 (a design goal), so **none contain cross-shot consistency supervision signals** and none can directly support multi-shot narrative generation. Native audio track: retained only by Panda-70M (optional) and UltraVideo, but neither makes any use of it.
[Average clip duration] See duration_distribution: 8.5s (Panda) / 11.7s (InternVid) / 13.75s (Koala) / 7.2s (OpenVid) / 20.2s (LVD-2M) / 5.3s and 30.9s (UltraVideo dual-split) / 72.1s (MiraData).

## Language/Accent Distribution (Data Foundation for Multilingual Lip-Sync Capability)

`language_accent_distribution` · Detail level: brief

### [Allegro](../models/Allegro.md) ⚠️

Not applicable to lip-sync (no audio). On the text side, T5 (not mT5) is used as the text encoder, with a maximum of 512 tokens; the paper shows through qualitative comparison (Fig. 7–8) that T5 outperforms mT5, so the training captions are effectively monolingual English; no multilingual caption or language-distribution statistics are disclosed. [Uncertain] (quantitative distribution of caption language composition)

### [Apollo](../models/Apollo.md) ⚠️

The paper does not discuss language and accent distribution at all: no supported languages are listed, no per-language proportions are given, and no accent annotation or data foundation for multilingual lip-sync is mentioned. An indirect inference that can be made: the transcription stage uses Whisper-Large-v3 (multilingual ASR), SenseVoice (Alibaba's open-source multilingual speech-understanding model covering Chinese/English/Japanese/Korean/Cantonese, with particularly strong support for Chinese and Chinese dialects), and Qwen2.5-Omni (strong in both Chinese and English) together — the combined use of all three strongly suggests the corpus covers at least Chinese and English, and the inclusion of SenseVoice points to a substantial proportion of Chinese/dialect content; the caption text encoder Qwen2.5-7B (a Chinese-English bilingual model) also supports this inference. But these are all tool-chain inferences rather than statements made in the paper. The evaluation metrics include WER (word error rate), indicating there is a transcription-accuracy evaluation, but the evaluation language is not stated. [Uncertain]

### [CineDance / CineDance-1M](../models/CineDance.md) ⚠️

[Taxonomy level] The eight-dimensional taxonomy includes a Region dimension, indirectly covering the demand for diversity in language and cultural region.
[Technical pipeline] ASR uses Qwen3-Omni-30B-A3B, which itself has multilingual capability; on the CineBench evaluation side, WER/CER is computed using Whisper-large-v3, also a multilingual model, indicating that neither the data nor the evaluation is a monolingual setup.
[Speaker attributes] Provides character voice description and ASR-to-Character binding (mapping speech sentences to specific character anchor tokens), but the annotation schema shows no explicit language-label, accent-label, or emotion-label fields.
[Quantitative distribution] The paper gives no statistics whatsoever on language proportion, accent distribution, or Chinese-English ratio. [Uncertain]

### [CogVideoX](../models/CogVideoX.md) ⚠️

Not applicable and not modeled. CogVideoX does not generate speech; the data pipeline has no ASR, no speaker attributes, and no language/accent annotation, and it lacks lip-sync capability (on the contrary, the "Lecture Type" label removes talking-head-type videos wholesale, actively weakening the model's data foundation for scenes of people speaking).
On the text-conditioning side: the caption and T5 text encoder mainly target English (both the paper and the caption-generation prompts are in English, with a text-length cap of 226 tokens); the official repository recommends using English prompts, with Chinese prompts needing to be rewritten first. Whether the CogSound side involves multilingual speech is not disclosed [Uncertain].

### [Cosmos-Predict2.5](../models/Cosmos-Predict25.md) ⚠️

Not applicable to the speech dimension (no audio, no speech, no concept of accent). On the text dimension, captions are generated by Qwen2.5-VL-7B; the paper does not state the caption language, but judging from all example prompts in the text (such as robot-welding scene descriptions, driving-scene prompt templates, and Transfer2.5's kitchen-scene prompt template), all of which are in English, and from the fact that the text encoder Cosmos-Reason1 is a predominantly English Physical AI VLM, the training prompts are essentially pure English monolingual, with no multilingual captions or translation augmentation step observed. [Uncertain: whether undisclosed non-English captions exist]

### [Data-Juicer 2.0](../models/Data-Juicer.md) ⚠️

[Uncertain] Data-Juicer does not provide language/accent distribution analysis or control capability for video/audio, and the official T2V case study does not address the language dimension either.
[Indirectly relevant capabilities]
  · Text side: DJ has included language_id_score_filter (fastText-based language identification and confidence filtering) since 1.0, which can filter caption text by text language, but this is text language rather than speech language.
  · Speech side: video_audio_ASR_mapper can transcribe the audio track; the multilingual capability of its underlying ASR model determines the range of languages that can be transcribed, but DJ does not encapsulate "the identified language" as a filterable structured field, and there is no accent-annotation operator either.
  · Speaker attributes: existing operators cover age, gender, and emotion, but not language and accent.
[Assessment] For the key AV-generation capability of multilingual lip-sync, DJ currently provides no data-side support and requires custom extension. This is related to its operator-evolution roadmap — the new operators added in 2026 concentrate on embodied intelligence (camera calibration, pose, hand reconstruction) and person-centric video understanding, and the speech/language dimension has not yet entered the priority list.

### [Foley-Omni](../models/Foley-Omni.md) ⚠️

Disclosure is missing, but it can be strongly inferred from data sources to be predominantly English. On the TTS side: LJSpeech (American-English single speaker), LibriTTS (English audiobooks, multiple speakers); on the VisualTTS side: GRID (controlled English corpus, 34 British speakers), LRS2 (BBC British English), Chem (English lectures) — all are pure English datasets; SpeakerVid and TalkVid are larger, newer talking-head video collections that may contain a small proportion of multilingual content, but the proportion is unclear.
The evaluation side uses WER as a speech-intelligibility metric (V2ST-Bench WER 7.59, GRID subset WER 15.3); WER computation typically relies on English ASR models, further corroborating that the evaluation scenario is English.
[Uncertain] The paper gives no language-distribution table, no mention of accent annotation, no mention of multilingual lip-sync capability, and no statement of the language composition of internal data. This is a clear shortfall relative to industrial-grade AV models (such as Veo 3, Sora 2, Kling Omni) on the multilingual lip-sync dimension.

### [Goku](../models/Goku.md) ⚠️

Not applicable/not addressed. Goku does not generate speech or lip shapes and has no multilingual lip-sync capability; the data pipeline has no ASR, no language identification, and no accent annotation. The only language-related aspect is the caption text language: the paper uses InternVL2.0, Tarsier2, and Qwen2 to generate and merge English captions, and the evaluation benchmarks (GenEval, DPG-Bench, VBench) are also English prompt-based systems, from which it can be inferred that training captions are predominantly English. [Uncertain] (whether captions contain Chinese or other languages, and the proportion of each language, is not stated in the paper)

### [Hailuo / MiniMax Video](../models/Hailuo.md) ⚠️

Not applicable + [Uncertain]. The model does not generate speech and has no lip-sync task, so the question of a data foundation for language/accent distribution does not arise. On the text side, prompts support both Chinese and English, but the official party has not disclosed the language composition of training captions or the Chinese-English ratio. The product targets both the Chinese and overseas markets (hailuoai.com and hailuoai.video / minimax.io, dual sites), suggesting captions may be Chinese-English bilingual, but there is no first-hand basis for this.

### [HunyuanVideo-Foley](../models/HunyuanVideo-Foley.md) ⚠️

The paper does not discuss language/accent distribution at all, and this basically does not constitute a relevant dimension for this work:
[Determined by task positioning] HunyuanVideo-Foley's goal is to generate Foley sound effects, not dialogue speech. The model does not do TTS, does not do lip-sync, and does not accept dialogue-text input, so there is no data requirement for multilingual lip-sync.
[Speech's role is to be recognized, not generated] The speech-detection step in the pipeline serves to tag speech-containing clips for category-ratio control, not to train speech-generation capability. Ideally, the model should learn to generate ambient sound and action sound in scenes where someone is talking, rather than synthesizing dialogue.
[Language of text conditioning] The text prompt is processed by the CLAP text encoder, whose training corpus is predominantly English, so the model's text-condition understanding capability should be predominantly English-based. The paper does not state whether Chinese prompts are supported, nor does it evaluate multilingual prompts. Open-source community practice generally recommends using English descriptions. [Uncertain]
[Conclusion] Language/accent distribution is not applicable to this work; the lack of disclosed information does not constitute a substantive disclosure gap.

### [HunyuanVideo](../models/HunyuanVideo.md) ⚠️

Not applicable to lip-sync (no audio generation). On the text side: the original version uses an MLLM as its text encoder and considers multiple languages during caption generation and prompt rewriting — the Hunyuan-Large LLM is used to rewrite user prompts (prompt rewrite), with functions including "prompt structure standardization, complex-term simplification, and multilingual adaptation" to align with the training-caption distribution; in practice it supports both Chinese and English prompts. The language-composition ratio of the training captions is not published. Version 1.5 does not describe prompt rewriting or language distribution. [Uncertain]

### [InstructAV2AV](../models/InstructAV2AV.md) ⚠️

[Uncertain] The paper does not disclose language/accent distribution at all — a notable gap in the data description of this work, especially considering speech editing is one of its four core tasks.
[Strong indirect evidence points to English as the primary language]
  · The ASR tool is ElevenLabs Scribe, and the speech-synthesis tool is ElevenLabs — although both support multiple languages, the paper does not mention any multilingual configuration.
  · The three film-related sources — MovieBench, Condensed Movies, and Short-Films-20K — are predominantly English film content; VGGSound is YouTube material where the speech component is inherently sparse.
  · Evaluation uses Sync-C / Sync-D (based on SyncNet, vshift=15 frames) to measure lip-sync; SyncNet itself is trained on English data and has lower discriminative accuracy for non-English phonemes.
  · Instructions in the dataset CSV wrap speech content boundaries with <S> and <E> markers, and the instructions themselves are in English.
[Missing items] No language labels, language-distribution tables, accent annotation, or evaluation/discussion of multilingual lip-sync capability are seen. Relative to industrial models such as Veo 3, Sora 2, and Kling Omni that emphasize multilingual lip-sync, this work makes no statement whatsoever on this dimension; actual capability should be regarded as validated only on English.

### [Other Audio-Video Joint Generation Works of 2026](../models/JAVG_2026_misc.md) ⚠️

This batch of works is generally weak on the language dimension — a common shortcoming:
[ALIVE] The language distribution of the training data is not disclosed [Uncertain]; only in the evaluation section is it mentioned that the evaluation prompts cover "multiple languages (Chinese and English)" — given ByteDance's product positioning and the 700k-hour transcribed-speech corpus, Chinese-and-English-dominant can be inferred but there is no positive evidence [Uncertain]. Content within the <W> tag in the caption is verbatim speech content, indicating the transcription preserves the original language.
[NAVA] The repository explicitly states "English speech generation supported" and notes "limited other languages' support" — i.e., predominantly English with weak multilingual capability. This is consistent with its data sources containing a large amount of TED-style speech videos (the vast majority of TED talks are in English). The language distribution of the 15M-clip corpus is not published [Uncertain]. Architecturally, ReDimNet speaker embeddings are introduced for timbre control, decoupling timbre from language, but language coverage is still limited by the data.
[StreamChar] Uses Emilia for orchestrator speech pretraining (80k steps) — Emilia itself is a large-scale multilingual speech dataset (covering English/Chinese/German/French/Japanese/Korean, six languages), which in principle provides a foundation for multilingual capability, but the paper does not report multilingual evaluation, so actual coverage is uncertain [Uncertain].
[OmniCustom] Uses GLM-ASR to generate transcriptions for each 5-second training audio clip ("we used GLM-ASR to generate transcriptions for each 5s training audio clip") — GLM-ASR is a Chinese-strength model, hinting the corpus contains a considerable proportion of Chinese [inference, uncertain]. The caption explicitly annotates an accent attribute (following Ovi's attribute set). The evaluation set controls a 1:1 gender ratio, but does not control the language ratio.
[CCL / Baton] Language distribution not mentioned [Uncertain]. CCL's in-house data (interviews, short dramas, movies) is mixed with OpenHumanVid, and the language composition is unclear; CCL reports a WER metric indicating it evaluated speech intelligibility, but the language is unspecified.
[ITS-JAVG] No training data; the evaluation benchmarks VGGSound and JavisBench-mini are predominantly English-environment.
[Overall assessment] None of the seven items give a quantitative language/accent-distribution table, and none report per-language evaluation of multilingual lip-sync. OmniCustom is the only work to make accent an explicit annotation field; NAVA is the only work to honestly declare "limited multilingual support."

### [Collection of Audio-Video Joint Generation Baselines](../models/JavisDiT_baselines.md) ⚠️

Disclosure is minimal, and overall shows a tendency toward "English monolingualism":
[MM-Diffusion / AV-DiT] Training data does not contain human speech (natural ambient sounds and instrumental music), so the language/accent dimension is not applicable (N/A).
[JavisDiT / JavisDiT++] Although the audio-pretraining data includes speech-type datasets (AudioSet and WavCaps contain speech components), the language distribution is not disclosed [Uncertain]; the audio-video SFT stage actively uses FunASR to remove videos containing speech, so the model basically does not have language-related lip-sync capability, and this dimension does not actually constitute a bottleneck for it.
[UniAVGen] Explicitly uses only the "English subset" of Emilia (a multilingual audio dataset) for audio pretraining — i.e., it deliberately forgoes multilingualism and focuses on English; evaluation uses the GRID benchmark (English corpus) and Whisper WER, likewise validating only English. The language composition of the internal real audio-visual data is not disclosed [Uncertain].
[Harmony] Uses Emilia (itself a multilingual TTS corpus) but does not state whether it is restricted to the English subset [Uncertain]; the paper mentions that Harmony-Bench's speech subset is used to examine "lip-sync fidelity and multilingual speech robustness" — the only work in this collection to explicitly claim multilingual coverage, but the specific language list, per-language proportions, and per-language lip-sync metrics are all not given [Uncertain].
[Common gap] None of the five annotate an accent attribute, perform language balancing, or report per-language sync metrics — a stark contrast to industrial models (such as Vidu, Kling, etc.) that emphasize multilingual lip-sync.

### [Kling 3.0 / Kling Video 3.0 Omni](../models/Kling_30_Omni.md) ⚠️

[Uncertain] Language/accent ratios are not published. Capability-side claims are explicit: five languages — Chinese, English, Japanese, and Korean, Spanish — and support for multiple regional accents/dialects and accent control (regional accent control), with lip shapes synced to the corresponding language. KlingAvatar 2.0 has expanded its data for multilingual, multi-character dialogue scenes; the evaluation set is built with 100 Chinese-speech examples, 100 English-speech examples, and 100 singing examples, hinting that Chinese and English are the dominant languages with the other three languages having smaller data volumes. [Uncertain: hours per language, dialect-coverage list, method used to annotate accent labels]

### [LTX-2](../models/LTX-2.md) ⚠️

No list of languages or proportions is given, but language/accent are first-class citizens on both the annotation and architecture sides.
[Annotation side] The caption system performs "precise transcription + speaker, language, and accent identification annotation" for dialogue — one of the most valuable annotation designs in this entry, directly constituting the data foundation for multilingual lip-sync and accent controllability.
[Architecture side] Uses the multilingual LLM Gemma3-12B as its text encoder with multi-layer feature extraction (early layers contain raw phoneme information, later layers contain complex semantics); the team explicitly states that "deep textual understanding not only serves global language support but also directly determines the phoneme accuracy of the generated speech." At inference time, users can enclose dialogue in quotes within the prompt and specify the desired language and accent.
[Limitation] The paper acknowledges: for languages or dialects underrepresented in the training data, speech-synthesis accuracy and audio-visual alignment noticeably deteriorate. How many languages are covered in total and the per-language/per-accent sample proportions are completely undisclosed. [Uncertain]

### [LongCat-Video](../models/LongCat-Video.md) ⚠️

[Uncertain]. The base version has no speech modality, but the text side explicitly supports Chinese and English: the text encoder uses umT5 (a multilingual encoder; the report explicitly states it supports both English and Chinese captions simultaneously), and the caption-augmentation stage includes translating captions between Chinese and English, so the training prompt distribution covers both Chinese and English, though the specific ratio is not disclosed.
The Avatar series involves real speech-driven generation, but the technical report discloses no language/accent distribution statistics and does not describe the data composition underlying multilingual lip-sync; the ASR transcription model used is not specified.

### [MOVA](../models/MOVA.md) ⚠️

The paper positions MOVA as "multilingual speech with high-quality lip synchronization," but gives only qualitative evidence for language/accent distribution, with no ratio figures whatsoever:
[Confirmed language coverage] Chinese and English are the two languages explicitly demonstrated and evaluated. Figure 1 shows two groups of precise lip-sync examples — English multi-speaker and Chinese multi-speaker (the Chinese example includes an adult-child two-speaker dialogue) — as well as an example of Chinese on-screen text generation. The subjective Arena evaluation set is deliberately constructed as bilingual: half of the originally pure-English Verse-Bench speech data was manually translated, forming a 732-sample Chinese-English bilingual evaluation set (600 from Verse-Bench + 132 from a self-built benchmark).
[Language basis on the data side] Chinese capability mainly comes from an in-house corpus of Chinese dramas; English capability mainly comes from SpeakerVid-5M, OpenHumanVid, and YouTube content.
[Accent annotation] The ASR-transcription prompt explicitly requires "LAW OF LANGUAGE FIDELITY: Preserve the original language. No translation," so the captions naturally preserve the original multilingual text; and audio captions describe the speaker's accent — the full caption example given in the paper includes "with a General American accent," indicating accent is one of the attributes naturally described in free text by Qwen3-Omni-Captioner, though it is not a structured enumerated field.
[Gaps] The list of supported languages, per-language hour proportions, and the distribution of accent categories are all undisclosed; the Discussion section mentions that differences in phoneme-viseme mapping across languages are a difficulty, but this is not quantified. [Uncertain]

### [Merged entry: ① Mochi 1](../models/Mochi_MAGI_Motif.md) ⚠️

① Mochi 1: the text encoder is a single T5-XXL (predominantly English); the language composition of captions is not disclosed. [Uncertain]
② MAGI-1: captions are generated by an MLLM, with examples (Table 4) all in English; language distribution is not disclosed. There is one indirect clue: to mitigate the distributional mismatch between training captions and real user input, MAGI-1 designed a Prompt Enhancement (PE) strategy on the inference side and distilled the rewriting ability of a large MLLM into a smaller ~7B model, constructing a training corpus of about 2 million entries — but the language composition of that corpus is not stated. [Uncertain]
③ Motif-Video 2B: captions are generated by Qwen3-VL-30B-A3B; the schema and examples are all in English, with caption_long defined as 150–250 words and caption_short as 15–25 words, explicitly measured in words — effectively monolingual English; the text encoder is T5Gemma2 (an early-stage sentence-level embedding model). No multilingual captions or language-distribution statistics are disclosed.
The lip-sync/accent dimension is not applicable to any of the three (no audio). [Uncertain]

### [Movie Gen](../models/Movie_Gen.md)

Not applicable and not modeled: Movie Gen Audio deliberately does not generate any human voice/dialogue (both diegetic speech and non-diegetic narration are excluded from the generation target), so there is no language, accent, or lip-sync data dimension, and no multilingual lip-sync capability. On the data side, speech is handled only to the extent of using AED to determine whether a sample contains speech/singing, used as a binary control label in captions and as a bucketing basis. On the text-conditioning side, the paper explicitly states the study is "limited to English text input."

### [NVIDIA NeMo Curator](../models/NeMo_Curator.md) ⚠️

Not addressed at all on the video side (no language dimension since audio tracks are not processed).
There is an indirectly relevant capability on the audio side, but no distribution statistics: the audio pipeline uses NVIDIA NeMo ASR models for transcription; the example dataset is FLEURS (a multilingual speech benchmark covering 100+ languages), and there is a long-form-audio-cutting tutorial involving speaker diarization. Starting with 26.02, streaming Sortformer is supported for speaker diarization, VAD, and speaker segmentation. However, the framework provides no language-identification (LID) stage or accent-annotation capability, and gives no control mechanism or statistics for language/accent ratios. The data foundation required for multilingual lip-sync cannot be directly produced by this framework. [Uncertain]

### [OmniHuman Dataset + OHBench](../models/OmniHuman.md) ⚠️

[Language] Predominantly English, accounting for up to 80% ("primarily English, accounting for up to 80%"). This is the only language figure given in the paper. The composition of the remaining 20% is entirely unspecified — whether it is concentrated in a few mainstream languages or spread across a long tail cannot be determined. [Uncertain]
[Accent] No accent annotation or statistics whatsoever. Given that the data source is globally sourced YouTube content with English content at 80%, it necessarily contains American, British, Indian-English, and other accents, but neither annotation nor distribution statistics exist. [Uncertain]
[Existence of language annotation] In the HuggingFace dataset's sample_json, the speech field contains "emotion, language, timing" — i.e., individual samples do carry language labels. Hence a complete language distribution is empirically obtainable by users, only the paper does not aggregate it. This is the same situation as with the duration/resolution distributions — the annotation is complete, but the statistical summary is missing.
[Caption language] Captions are provided in both English and Chinese variants, i.e., the annotation text itself is bilingual — but this is a separate matter from the language of the audio content (80% English audio + bilingual Chinese-English captions). Bilingual captions are a practical convenience for the Chinese-speaking community.
[Language capability of the ASR model] FunASR-Nano (from the DAMO Academy's FunASR series) performs best on Chinese while also supporting multiple languages; the evaluation side's WER computation uses SenseVoice (also multilingual); neither constitutes a language limitation.
[Implication for lip-sync capability] The 80% English skew means that a model trained on this data will have its phoneme-viseme mapping predominantly based on the English phoneme system, and its lip-shape accuracy may be weaker for tonal languages (Chinese) and other phoneme systems — the paper does not discuss this potential bias. [Uncertain]

### [Open-Sora series](../models/Open-Sora.md)

Not applicable at the speech level (no audio generation). On the text side: Open-Sora Plan explicitly uses only the **English subset** of Anytext-3M (about 50% of that dataset, 1.8M), with captions and prompts entirely in English; the Open-Sora series' captions are also entirely in English (PLLaVA, LLaVA-Video, and Qwen2.5-Max all output English descriptions). Neither project builds data support for multilingual prompts; Chinese prompts must rely on external translation. Data foundation related to accent/lip-sync: none.

### [Ovi](../models/Ovi.md) ⚠️

No quantitative distribution is given. Confirmed qualitative facts:
(1) The audio-pretraining data "emphasize linguistic diversity, prosody, and timbral variation," i.e., the corpus was intentionally designed for diversity at the language/accent level, but which languages are covered and in what proportions is not disclosed [Uncertain].
(2) At the annotation stage, an accent attribute is explicitly annotated for speech-containing clips, listed alongside age, gender, pitch, prosody, emotion, and speaking rate in the audio description — meaning accent is a controllable condition rather than an implicit variable.
(3) At the evaluation level, only English WER is reported: WER=0.035 on Seed-TTS test-en, with no non-English metrics reported, indicating its verifiable speech capability is predominantly English; community practice and demos are likewise predominantly English dialogue [Uncertain].
(4) Lip-sync capability is entirely data-driven (no face bounding box, no mask), so multilingual lip-sync capability is directly limited by the language coverage of the internal corpus.

### [Script-a-Video](../models/Script-a-Video.md) ⚠️

The paper gives no language/accent distribution data whatsoever: no supported languages are listed, no per-language proportions are given, no accent is annotated, and multilingual lip-sync is not discussed.
Indirectly inferable clues:
1) During evaluation, WER computation explicitly uses "jieba-based tokenization for CJK text," indicating that generated and evaluated content does indeed include Chinese (or CJK more broadly) speech content, so the training data is not purely English;
2) The data sources are predominantly film / television / TV drama and are Tencent in-house data, so a high likelihood exists of Chinese film/TV content dominating, but the paper offers no corroboration;
3) The Event stream of MTSS sets a "line" field for dialogue events to record verbatim speech, and the schema places no language restriction, but no language or accent field is defined;
4) The Event stream's "description" field is used to capture nuanced semantics such as emotional shifts and vocal techniques — the design closest to "speaker attributes," but it remains free text rather than a structured accent/language label.
Language and accent constitute a complete information blank. [Uncertain]

### [Seedance 2.0 and Seedance 1.5 pro](../models/Seedance_20_Seedance_15_pro.md) ⚠️

The training data's language/accent ratio is not disclosed, but capability and evaluation coverage allow reverse inference: Seedance 1.5 pro natively supports lip-shape synchronization for multiple languages and multiple regional dialects, capturing each language's distinctive speech prosody and emotional tension; Seedance 2.0 shows markedly improved instruction-following accuracy for Chinese dialects, traditional opera, and singing scenarios. SeedVideoBench 2.0's audio evaluation explicitly distinguishes categories such as Chinese dialects/accents, Chinese multi-person dialogue, Chinese variety-show voice, Chinese opera, English, and minority-ethnic languages (in version 2.0, audio-quality scores are 4.17 for English, 3.82 for minority-ethnic languages, and 2.82 for Chinese dialects; audio-visual sync scores are 4.17 for English, 3.88 for minority-ethnic languages, and 3.64 for Chinese dialects), indicating the training corpus is Chinese-English-dominant while also covering multiple Chinese dialects and minority-ethnic languages. Additionally, Seedance 1.0's caption model was trained on Chinese-English bilingual data. [Uncertain: specific proportions per language]

### [SkyReels series](../models/SkyReels.md) ⚠️

[SkyReels-V2] A pure video model with no language dimension; annotation and prompts are predominantly Chinese-English.
[SkyReels-V4] Language is an explicit object of data construction, but with no proportion figures: (1) synthetic data is specifically used for "multilingual on-screen text generation," covering Chinese, English, Japanese, Korean, German, French, and more; (2) synthetic data is also used for "multilingual speech synthesis," i.e., TTS is used to fill gaps in language coverage — a direct means of addressing the shortage of long-tail-language lip-sync data; (3) the audio backbone is pretrained on hundreds of thousands of hours of predominantly speech data, with language coverage extended via TTS; (4) the evaluation benchmark SkyReels-VABench's prompts cover multiple languages, "with particular emphasis on Chinese and English." No per-language sample-proportion figures or accent-annotation fields are given, and there is no dialect/accent dimension distribution control — accent is not an independent field in the caption schema (unlike LTX-2's explicit accent annotation). [Uncertain]

### [Sora 2](../models/Sora_2.md) ⚠️

Completely undisclosed. The model has dialogue-generation and lip-sync capability, and in practice can generate speech in multiple languages, but OpenAI has not published a list of supported languages, per-language data proportions, or accent distribution, nor has it explained the data foundation for multilingual lip-sync. On the safety side, it is only mentioned that audio transcripts pass through a safety classifier, indirectly indicating ASR capability exists, but without addressing language coverage. [Uncertain]

### [SpeakerVid-5M](../models/SpeakerVid-5M.md) ⚠️

Disclosure is minimal — one of the most obvious statistical blanks in this dataset:
[Known information] All data sources are public YouTube videos, collected under the scope of "high-quality two-person dialogue videos," spanning genres such as interviews/news/panel discussions/variety shows/debates/education, with no language restriction. Whisper is used for ASR transcription; Whisper itself has language-identification capability, and the cleaning pipeline mentions filtering based on "detected language mismatches," indicating the pipeline does internally record language labels.
[Undisclosed] Neither the paper nor the data card gives a list of supported languages, per-language clip counts or hour proportions, or any annotation or statistics for accent categories. The structured caption fields do not include a language or accent item; ASR annotation contains only the transcript text and a confidence score.
[Indirect inference] Given the interview/news/debate/education content sourced from YouTube, and the proportion of entertainment and news/political channels, the corpus is very likely predominantly English, but the paper makes no such statement explicitly, so this should not be cited as fact.
[Downstream usage] MOVA uses SpeakerVid-5M together with OpenHumanVid and YouTube content as the data foundation for its English lip-sync capability, while its Chinese capability is provided separately by an in-house Chinese-drama corpus — this indirectly corroborates that SpeakerVid-5M's language center of gravity leans English, but this is circumstantial evidence rather than a statement from the original paper. [Uncertain]

### [Step-Video-T2V](../models/Step-Video-T2V.md) ⚠️

Not applicable to lip-sync (no audio generation). The text side is a distinctive feature of this entry: the model uses two bilingual (Chinese-English) text encoders (a dual-encoder combination of Hunyuan-CLIP and the in-house Step-LLM), natively supporting both Chinese and English prompt input; the official party lists "native Chinese-English bilingual input" as a core capability. The accompanying evaluation benchmark Step-Video-T2V-Eval consists entirely of 128 Chinese prompts, which also corroborates a Chinese-leaning emphasis in both the data and evaluation. However, the report does not disclose the Chinese-English language-composition ratio of the training captions, nor whether special augmentation was performed for Chinese captions. [Uncertain]

### [UniTalking](../models/UniTalking.md) ⚠️

The paper gives no statistics of any kind on the training data's language or accent distribution: no language list, no per-language proportions, no accent annotation, and no discussion of multilingual phoneme-viseme mapping. Confirmed indirect evidence shows at least Chinese and English coverage:
[Bilingual evidence from evaluation] The TR2AV task's timbre-similarity evaluation uses the MiniMax Multilingual Test Set, with results reported separately for English (0.703) and Chinese (0.662) — the model can produce evaluable cloned speech in both languages, indicating both Chinese and English have substantial proportions in the training data. Notably the Chinese score is lower than the English score (0.662 vs. 0.703), whereas two of the three comparison methods (ElevenLabs 0.613/0.677, MiniMax 0.756/0.780, Qwen3-Omni 0.773/0.772) score higher in Chinese than in English — UniTalking is the only method exhibiting a clear "stronger-in-English, weaker-in-Chinese" tendency, indirectly suggesting its training data may be predominantly English content. This inference is not confirmed by the paper. [Uncertain]
[Monolinguality of the TTS evaluation] The Stage 1 audio model's WER evaluation is conducted only on Seed-TTS test-en (English), with no test-zh results reported, further pointing to English as the primary optimization target.
[Text encoder] Uses UMT5 (a multilingual T5), which architecturally does not constitute a language limitation.
[Transcription model] Whisper-V3 is a multilingual ASR, likewise not constituting a limitation.
[Internal TTS data] The language composition of the internal TTS data used in Stage 1 is entirely undisclosed, and it is precisely in this stage that the audio branch's pronunciation capability is primarily established — this is the true decisive factor for language capability, yet it is a complete black box. [Uncertain]

### [UniVerse-1](../models/UniVerse-1.md) ⚠️

The paper does not discuss language and accent distribution at all: no list of supported languages, no per-language hours or proportions, no accent annotation, and no discussion of multilingual phoneme-viseme mapping issues. Indirectly inferable clues:
- The training side uses Whisper for ASR, which is inherently multilingual, but the paper does not state whether the language is restricted or whether a language distribution was tabulated;
- The data sources are YouTube speeches/interviews/vlogs, where English content is most likely the largest proportion;
- On the evaluation side, Verse-Bench's Set2-V is explicitly collected from both YouTube and Bilibili — the inclusion of Bilibili indicates evaluation coverage of Chinese content; Set3-Ted is drawn from TED Talks in September 2025, which are English speeches. Hence Chinese and English are both covered, at least at the evaluation level;
- Architecturally, the speaker encoder from Ace-step was removed, with the aim of not tying the model to a specific speaker — this does not concern the language dimension.
Language and accent distribution constitute a complete information blank. [Uncertain]

### [Unison](../models/Unison.md) ⚠️

The paper does not discuss language and accent at all: no list of supported languages, no per-language proportions, no accent/dialect annotation, no discussion of multilingual phoneme-viseme mapping, and no statement of whether language is restricted.
[Indirectly inferable clues]
- HDTF is sourced from YouTube English speeches/news, monolingual English; VFHQ is sourced from interview scenes, predominantly English; CelebV-Text is in-the-wild face video, predominantly English; OpenHumanVid contains Chinese and English film/TV material with an unclear language distribution; hence the audio-visual side of the corpus is very likely predominantly English overall;
- The singing corpus is drawn from the YuE collection; the YuE series is known for Chinese-English bilingual singing generation, which may introduce a Chinese component;
- The "internal speech data" is provided by Chinese institutions (ByteDance, China Telecom's TeleAI), suggesting a higher likelihood of containing Chinese speech, but the paper says nothing about this;
- Evaluation uses Whisper-large-v3 to compute WER; Whisper itself is multilingual, but the paper does not state the language composition of the evaluation set, nor does it report WER broken down by language.
[Comparison with similar works] MOVA built a 732-sample Chinese-English bilingual Arena evaluation set and reports cpCER; UniVerse-1's Verse-Bench covers both YouTube and Bilibili; Unison has no corresponding facility on the language dimension, making its evaluation system relatively thin in this respect.
Language and accent distribution constitute a complete information blank. [Uncertain]

### [Veo 3 / Veo 3.1](../models/Veo_3_Veo_31.md) ⚠️

[Uncertain] Language and accent distribution are entirely undisclosed. On the product side, the model can generate multilingual dialogue and automatically match accent and lip shape (English, Spanish, Mandarin, etc.), but the official documentation does not list supported languages, does not give per-language data proportions, and does not describe an accent-annotation system. Third-party evaluations generally report that lip-sync and pronunciation quality for non-English dialogue are noticeably weaker than for English, indirectly indicating the training data is predominantly English. The official party also acknowledges the model "does not provide fine-grained voice control such as accent or timbre," indicating speaker attributes were not incorporated as a conditioning dimension in the training data schema.

### [Vidu S1](../models/Vidu_S1.md) ⚠️

[Uncertain]. The paper does not disclose the language and accent distribution of the training data, nor does it describe the data foundation for multilingual lip-sync capability. The caption structure only includes a dialogue field, and the Vidu-StreamBench benchmark mentions covering diverse "speaker attributes" and emotion, but does not elaborate on the language dimension. The product targets the Chinese market, suggesting a Chinese-dominant corpus, but there is no first-hand basis for this.

### [Wan 2.5 / 2.6 / 2.7](../models/Wan.md) ⚠️

No language list or proportions are disclosed at all — a notably weak link for the Wan series relative to LTX-2.
Confirmed indirect clues:
- Text encoder: after ablation, Wan 2.1 selected umT5 (5.3B, multilingual, bidirectional attention); the reported ablation explicitly shows umT5 outperforming Qwen2.5-7B-Instruct and GLM-4-9B (even when the latter is given a HunyuanVideo-style bidirectional token refiner) — umT5's multilingual capability is one reason for this choice.
- OCR and visual text: Wan 2.1's OCR-augmented caption dataset "currently only contains English and Chinese text"; synthetic text images specifically render Chinese characters to improve Chinese glyph generation.
- Prompt language: the entire API lineup "supports Chinese and English," with negative prompts likewise bilingual.
- The official examples for 2.6/2.7 include both Chinese dialogue (recitation of classical poetry) and English rap singing, indicating at least Chinese-English bilingual lip-sync coverage.
The accent dimension never appears as an annotation field or capability dimension in any public material of the Wan series — a sharp contrast with LTX-2's writing of speaker/language/accent as three attributes into its caption schema. The data foundation for multilingual lip-sync is entirely invisible. [Uncertain]

### [Collection of Audio-Video Generation Evaluation Benchmarks](../models/av_benchmarks.md) ⚠️

Disclosure is generally weak, a common shortcoming across all five benchmarks:
[AVBench] Lip-sync and Speech Content Accuracy are core dimensions, but the paper does not disclose the language and accent composition of its evaluation prompts and the OpenHumanVid training clips [Uncertain].
[VABench] Lip-sync metrics are applied only to the subset of "vocal-linguistic" samples in which a speaker's head is detected; multilingual coverage is not disclosed [Uncertain].
[AV-SyncBench] For human-voice scenes (single-person speech / dialogue / group vocalization / singing), semantic perturbation uses OpenVoice V2 for timbre replacement (a tool that itself supports cross-lingual timbre cloning), but the benchmark does not provide statistics stratified by language/accent [Uncertain].
[PhyAVBench] Predominantly non-speech physical sound effects; speech is only indirectly involved via the WER metric (Whisper-Large V3); the language background of the 184 participants is not disclosed [Uncertain].
[Omni-Judge] Does not address the language dimension [Uncertain].
Conclusion: there is currently no public benchmark against which multilingual/multi-accent lip-sync can be measured, so the language ratio of training data cannot be reverse-verified using existing benchmarks.

### [Video Captioning Model Ecosystem](../models/caption_models.md) ⚠️

[Bilingual/multilingual annotation capability (a few works explicitly address this)]
· Seedance 1.0's captioner (built on Tarsier2) is explicitly trained on Chinese-English bilingual data to acquire bilingual annotation capability, with the visual encoder frozen and full-parameter fine-tuning applied to the language model during training — the clearest first-hand requirement placed on a captioner's language capability on the generation side.
· LTX-2's in-house captioner is the most fine-grained public solution for handling language/accent: dialogue transcription not only provides text but also simultaneously annotates the three attributes of speaker, language, and accent. This schema design is critical to the data foundation for multilingual lip-sync capability.
· SkyCaptioner-V1 makes no explicit bilingual claim; SkyReels-V4's speech and singing content is transcribed by Whisper (multilingual).
[Multilingual support on the ASR side] Whisper-large-v3 (multilingual, ~1.55B) is the default choice across the ecosystem; Alibaba's SenseVoice (~234M, supporting Chinese/English/Cantonese/Japanese/Korean + emotion/event recognition) is used alongside Whisper and Qwen2.5-Omni by Apollo/Klear; ElevenLabs Scribe is used by InstructAV2AV for precise timestamps; the Qwen3-Omni series is natively multilingual.
[Widespread gap in accent handling] Aside from LTX-2, no captioner is seen to explicitly annotate accent; AVoCaDO's dialogue reward uses only a (speaker, spoken content) pair, with no language/accent field.
[Uncertain] No captioner or its training data publicly discloses language-distribution proportions.

### [Collection of Geometric/Structured Annotation Datasets](../models/geometric_datasets.md)

SceneScribe-1M / SpatialVID / WildWorld: captions are all purely English, with no multilingual or accent dimension and no speakers; Action100M: the total annotation volume of 21.3 billion words is entirely in English, sourced from the English instructional-video subset of HowTo100M (ASR is in English), with no language/accent distribution statistics performed. None of the four have a data foundation for multilingual lip-sync.

### [Video-Generation Post-Training Data Strategies](../models/post_training_data.md) ⚠️

Neither the anchor paper nor the great majority of cross-referenced subjects disclose language/accent ratios at the post-training stage [Uncertain]. Indirect facts worth recording:
· The anchor paper's prompt enhancer (Phase 3) is an LLM trained with GRPO; the language composition of its training prompts is not stated [Uncertain];
· MOVA built a 732-sample Chinese-English bilingual Arena evaluation set (for evaluation, not training); UniTalking's 20-person × 50-sample blind test and Unison's 40-sample × 25-person ranking vote are likewise for evaluation;
· The two public preference datasets HPDv3 and VideoReward are both predominantly English-prompt [Uncertain]; preference data for Chinese scenarios is a clear gap in the public ecosystem;
· The source of HunyuanVideo 1.5's RLHF prompt set is "a mix of LLM-generated prompts and training captions," with the language not stated [Uncertain].

### [Combined Survey of Mainstream Video Pretraining Datasets: Panda-70M, InternVid, Koala…](../models/pretraining_datasets.md)

Not applicable at the speech level (no audio modeling) for all. At the text level:
- The captions of all seven datasets are **entirely monolingual English**; none provide Chinese or multilingual captions, and none build data support for multilingual prompts.
- **InternVid is the only one involving multilingual data**: its internal collection includes **ASR subtitles in 11 languages** (the paper's Figure 16 shows examples of English, Chinese, Korean, and German), with source-video countries of origin covering the UK, US, Australia, Japan, Korea, China, Russia, France, and more. But **these ASR transcripts were neither used to generate captions nor released with the dataset** — the published jsonlines contain no audio, no ASR, and no subtitle field. This is a multilingual resource that was collected but then discarded.
- **Panda-70M uses subtitles but only as input**: English subtitles (including YouTube auto-generated subtitles) are fed into the teacher model as textual context (the video2dataset configuration is subtitleslangs:['en'], writeautomaticsub:True); the subtitles themselves are not released as a Panda-70M field, only landing in each clip's JSON metadata.
- Data foundation related to accent, speaker identity, and lip-sync: **all seven are zero**.
- One knock-on effect: since captions are all in English and generally lengthy (Koala 202 words, MiraData 318 words, UltraVideo 824 words), training with this data **requires pairing with a long-context text encoder** — MiraData therefore explicitly abandons CLIP's 77-token encoder in favor of **Flan-T5-XXL (512 tokens)**; LVD-2M, by contrast, suffered a setback in experiments — its 88.7-word captions were truncated at 77 tokens by a frozen CLIP text encoder, and the authors attribute the unclear improvement in I2V text matching directly to this.
